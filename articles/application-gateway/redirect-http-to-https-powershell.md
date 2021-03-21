---
title: Перенаправление HTTP в HTTPS с помощью PowerShell — шлюз приложений Azure
description: Узнайте, как создать шлюз приложений с перенаправлением трафика HTTP в HTTPS с помощью Azure PowerShell.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 09/28/2020
ms.author: victorh
ms.openlocfilehash: 86eaa645cd6a81b9180d1241695240a71aa8202d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93397269"
---
# <a name="create-an-application-gateway-with-http-to-https-redirection-using-azure-powershell"></a>Создание шлюза приложений с перенаправлением трафика HTTP в HTTPS с помощью Azure PowerShell

Вы можете использовать Azure PowerShell для создания [шлюза приложений](overview.md) с сертификатом для завершения TLS/SSL. Правило маршрутизации используется для перенаправления трафика HTTP в HTTPS-порт в шлюзе приложений. В этом примере также создается [масштабируемый набор виртуальных машин](../virtual-machine-scale-sets/overview.md) с двумя экземплярами виртуальных машин, предназначенный для внутреннего пула шлюза приложений. 

Вы узнаете, как выполнять следующие задачи:

* Создание самозаверяющего сертификата
* настройка сети;
* создание шлюза приложений с сертификатом;
* добавление прослушивателя и правила перенаправления;
* создание масштабируемого набора виртуальных машин с серверным пулом, используемым по умолчанию.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Для работы с этим руководством требуется модуль Azure PowerShell версии 1.0.0 и выше. Чтобы узнать версию, выполните команду `Get-Module -ListAvailable Az`. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Для выполнения команд в этом руководстве необходимо также выполнить командлет `Login-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-a-self-signed-certificate"></a>Создание самозаверяющего сертификата

Для использования в рабочей среде следует импортировать действительный сертификат, подписанный доверенным поставщиком. В этом руководстве мы создадим самозаверяющий сертификат с помощью [New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate). Вы можете использовать [Export-PfxCertificate](/powershell/module/pkiclient/export-pfxcertificate) с возвращенным отпечатком, чтобы экспортировать PFX-файл из сертификата.

```powershell
New-SelfSignedCertificate `
  -certstorelocation cert:\localmachine\my `
  -dnsname www.contoso.com
```

Отобразится примерно такой результат:

```
PSParentPath: Microsoft.PowerShell.Security\Certificate::LocalMachine\my

Thumbprint                                Subject
----------                                -------
E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630  CN=www.contoso.com
```

Воспользуйтесь отпечатком, чтобы создать PFX-файл:

```powershell
$pwd = ConvertTo-SecureString -String "Azure123456!" -Force -AsPlainText
Export-PfxCertificate `
  -cert cert:\localMachine\my\E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630 `
  -FilePath c:\appgwcert.pfx `
  -Password $pwd
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. Создайте группу ресурсов Azure с именем *myResourceGroupAG* , используя [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup). 

```powershell
New-AzResourceGroup -Name myResourceGroupAG -Location eastus
```

## <a name="create-network-resources"></a>Создание сетевых ресурсов

Создайте конфигурации подсетей с именами *myBackendSubnet* и *myAGSubnet*, выполнив командлет [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). Создайте виртуальную сеть с именем *myVNet*, используя командлет [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) с конфигурациями подсетей. Наконец, создайте общедоступный IP-адрес с именем *myAGPublicIPAddress*, выполнив командлет [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress). Эти ресурсы используются для обеспечения сетевого подключения к шлюзу приложений и связанным с ним ресурсам.

```powershell
$backendSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myBackendSubnet `
  -AddressPrefix 10.0.1.0/24
$agSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myAGSubnet `
  -AddressPrefix 10.0.2.0/24
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myVNet `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $backendSubnetConfig, $agSubnetConfig
$pip = New-AzPublicIpAddress `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myAGPublicIPAddress `
  -AllocationMethod Dynamic
```

## <a name="create-an-application-gateway"></a>Создание шлюза приложений

### <a name="create-the-ip-configurations-and-frontend-port"></a>Создание IP-конфигураций и интерфейсного порта

Свяжите созданную ранее подсеть *myAGSubnet* со шлюзом приложений, используя командлет [New-AzApplicationGatewayIPConfiguration](/powershell/module/az.network/new-azapplicationgatewayipconfiguration). Назначьте шлюзу приложений адрес *myAGPublicIPAddress* с помощью командлета [New-AzApplicationGatewayFrontendIPConfig](/powershell/module/az.network/new-azapplicationgatewayfrontendipconfig). Затем можно создать HTTPS-порт с помощью команды [New азаппликатионгатевайфронтендпорт](/powershell/module/az.network/new-azapplicationgatewayfrontendport).

```powershell
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Name myVNet
$subnet=$vnet.Subnets[0]
$gipconfig = New-AzApplicationGatewayIPConfiguration `
  -Name myAGIPConfig `
  -Subnet $subnet
$fipconfig = New-AzApplicationGatewayFrontendIPConfig `
  -Name myAGFrontendIPConfig `
  -PublicIPAddress $pip
$frontendPort = New-AzApplicationGatewayFrontendPort `
  -Name myFrontendPort `
  -Port 443
```

### <a name="create-the-backend-pool-and-settings"></a>Создание серверного пула и настройка параметров

Создайте серверный пул с именем *appGatewayBackendPool* для шлюза приложений с помощью командлета [New-AzApplicationGatewayBackendAddressPool](/powershell/module/az.network/new-azapplicationgatewaybackendaddresspool). Настройте параметры для серверного пула, используя командлет [New-AzApplicationGatewayBackendHttpSettings](/powershell/module/az.network/new-azapplicationgatewaybackendhttpsetting).

```powershell
$defaultPool = New-AzApplicationGatewayBackendAddressPool `
  -Name appGatewayBackendPool 
$poolSettings = New-AzApplicationGatewayBackendHttpSettings `
  -Name myPoolSettings `
  -Port 80 `
  -Protocol Http `
  -CookieBasedAffinity Enabled `
  -RequestTimeout 120
```

### <a name="create-the-default-listener-and-rule"></a>Создание прослушивателя по умолчанию и правила

Прослушиватель требуется для того, чтобы шлюз приложений правильно маршрутизировал трафик на внутренние пулы. В этом примере создается базовый прослушиватель, который ожидает передачи трафика HTTPS по корневому URL-адресу. 

Создайте объект сертификата с помощью команды [New-азаппликатионгатевайсслцертификате](/powershell/module/az.network/new-azapplicationgatewaysslcertificate) , а затем создайте прослушиватель с именем *AppGatewayHttpListener* , используя [New-азаппликатионгатевайхттплистенер](/powershell/module/az.network/new-azapplicationgatewayhttplistener) с интерфейсной конфигурацией, интерфейсным портом и ранее созданным сертификатом. Правило требуется для того, чтобы указать прослушивателю, какой внутренний пул использовать для входящего трафика. Создайте базовое правило *rule1* с помощью командлета [New-AzApplicationGatewayRequestRoutingRule](/powershell/module/az.network/new-azapplicationgatewayrequestroutingrule).

```powershell
$pwd = ConvertTo-SecureString `
  -String "Azure123456!" `
  -Force `
  -AsPlainText
$cert = New-AzApplicationGatewaySslCertificate `
  -Name "appgwcert" `
  -CertificateFile "c:\appgwcert.pfx" `
  -Password $pwd
$defaultListener = New-AzApplicationGatewayHttpListener `
  -Name appGatewayHttpListener `
  -Protocol Https `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendPort `
  -SslCertificate $cert
$frontendRule = New-AzApplicationGatewayRequestRoutingRule `
  -Name rule1 `
  -RuleType Basic `
  -HttpListener $defaultListener `
  -BackendAddressPool $defaultPool `
  -BackendHttpSettings $poolSettings
```

### <a name="create-the-application-gateway"></a>Создание шлюза приложений

Теперь, когда вы создали необходимые вспомогательные ресурсы, укажите параметры для шлюза приложений *myAppGateway* с помощью командлета [New-AzApplicationGatewaySku](/powershell/module/az.network/new-azapplicationgatewaysku), а затем создайте шлюз с сертификатом с помощью командлета [New-AzApplicationGateway](/powershell/module/az.network/new-azapplicationgateway).

```powershell
$sku = New-AzApplicationGatewaySku `
  -Name Standard_Medium `
  -Tier Standard `
  -Capacity 2
$appgw = New-AzApplicationGateway `
  -Name myAppGateway `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -BackendAddressPools $defaultPool `
  -BackendHttpSettingsCollection $poolSettings `
  -FrontendIpConfigurations $fipconfig `
  -GatewayIpConfigurations $gipconfig `
  -FrontendPorts $frontendPort `
  -HttpListeners $defaultListener `
  -RequestRoutingRules $frontendRule `
  -Sku $sku `
  -SslCertificates $cert
```

## <a name="add-a-listener-and-redirection-rule"></a>добавление прослушивателя и правила перенаправления;

### <a name="add-the-http-port"></a>Добавление HTTP-порта

Добавьте HTTP-порт в шлюз приложений с помощью [Add-азаппликатионгатевайфронтендпорт](/powershell/module/az.network/add-azapplicationgatewayfrontendport).

```powershell
$appgw = Get-AzApplicationGateway `
  -Name myAppGateway `
  -ResourceGroupName myResourceGroupAG
Add-AzApplicationGatewayFrontendPort `
  -Name httpPort  `
  -Port 80 `
  -ApplicationGateway $appgw
```

### <a name="add-the-http-listener"></a>Добавление прослушивателя HTTP

Добавьте прослушиватель HTTP с именем *myListener* в шлюз приложений с помощью [Add-азаппликатионгатевайхттплистенер](/powershell/module/az.network/add-azapplicationgatewayhttplistener).

```powershell
$fipconfig = Get-AzApplicationGatewayFrontendIPConfig `
  -Name myAGFrontendIPConfig `
  -ApplicationGateway $appgw
$fp = Get-AzApplicationGatewayFrontendPort `
  -Name httpPort `
  -ApplicationGateway $appgw
Add-AzApplicationGatewayHttpListener `
  -Name myListener `
  -Protocol Http `
  -FrontendPort $fp `
  -FrontendIPConfiguration $fipconfig `
  -ApplicationGateway $appgw
```

### <a name="add-the-redirection-configuration"></a>Добавление конфигурации перенаправления

Добавьте конфигурацию перенаправления HTTP в HTTPS в шлюз приложений с помощью [Add-азаппликатионгатевайредиректконфигуратион](/powershell/module/az.network/add-azapplicationgatewayredirectconfiguration).

```powershell
$defaultListener = Get-AzApplicationGatewayHttpListener `
  -Name appGatewayHttpListener `
  -ApplicationGateway $appgw
Add-AzApplicationGatewayRedirectConfiguration -Name httpToHttps `
  -RedirectType Permanent `
  -TargetListener $defaultListener `
  -IncludePath $true `
  -IncludeQueryString $true `
  -ApplicationGateway $appgw
```

### <a name="add-the-routing-rule"></a>Добавление правила маршрутизации

Добавьте правило маршрутизации с конфигурацией перенаправления в шлюз приложений с помощью [Add-азаппликатионгатевайрекуестраутингруле](/powershell/module/az.network/add-azapplicationgatewayrequestroutingrule).

```powershell
$myListener = Get-AzApplicationGatewayHttpListener `
  -Name myListener `
  -ApplicationGateway $appgw
$redirectConfig = Get-AzApplicationGatewayRedirectConfiguration `
  -Name httpToHttps `
  -ApplicationGateway $appgw
Add-AzApplicationGatewayRequestRoutingRule `
  -Name rule2 `
  -RuleType Basic `
  -HttpListener $myListener `
  -RedirectConfiguration $redirectConfig `
  -ApplicationGateway $appgw
Set-AzApplicationGateway -ApplicationGateway $appgw
```

## <a name="create-a-virtual-machine-scale-set"></a>создавать масштабируемый набор виртуальных машин;

В этом примере создается масштабируемый набор виртуальных машин, чтобы предоставить серверы для серверного пула в шлюзе приложений. Масштабируемый набор назначается серверному пулу при настройке параметров IP-адреса.

```powershell
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Name myVNet
$appgw = Get-AzApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway
$backendPool = Get-AzApplicationGatewayBackendAddressPool `
  -Name appGatewayBackendPool `
  -ApplicationGateway $appgw
$ipConfig = New-AzVmssIpConfig `
  -Name myVmssIPConfig `
  -SubnetId $vnet.Subnets[1].Id `
  -ApplicationGatewayBackendAddressPoolsId $backendPool.Id
$vmssConfig = New-AzVmssConfig `
  -Location eastus `
  -SkuCapacity 2 `
  -SkuName Standard_DS2 `
  -UpgradePolicyMode Automatic
Set-AzVmssStorageProfile $vmssConfig `
  -ImageReferencePublisher MicrosoftWindowsServer `
  -ImageReferenceOffer WindowsServer `
  -ImageReferenceSku 2016-Datacenter `
  -ImageReferenceVersion latest `
  -OsDiskCreateOption FromImage
Set-AzVmssOsProfile $vmssConfig `
  -AdminUsername azureuser `
  -AdminPassword "Azure123456!" `
  -ComputerNamePrefix myvmss
Add-AzVmssNetworkInterfaceConfiguration `
  -VirtualMachineScaleSet $vmssConfig `
  -Name myVmssNetConfig `
  -Primary $true `
  -IPConfiguration $ipConfig
New-AzVmss `
  -ResourceGroupName myResourceGroupAG `
  -Name myvmss `
  -VirtualMachineScaleSet $vmssConfig
```

### <a name="install-iis"></a>Установка служб IIS

```powershell
$publicSettings = @{ "fileUris" = (,"https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/appgatewayurl.ps1"); 
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File appgatewayurl.ps1" }
$vmss = Get-AzVmss -ResourceGroupName myResourceGroupAG -VMScaleSetName myvmss
Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $publicSettings
Update-AzVmss `
  -ResourceGroupName myResourceGroupAG `
  -Name myvmss `
  -VirtualMachineScaleSet $vmss
```

## <a name="test-the-application-gateway"></a>Тестирование шлюза приложений

Вы можете использовать командлет [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress), чтобы получить общедоступный IP-адрес шлюза приложений. Скопируйте общедоступный IP-адрес и вставьте его в адресную строку браузера. Например: http://52.170.203.149

```powershell
Get-AzPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress
```

![Предупреждение системы безопасности](./media/redirect-http-to-https-powershell/application-gateway-secure.png)

Чтобы принять предупреждение безопасности, если вы использовали самозаверяющий сертификат, выберите **сведения** , а затем перейдите на **веб-страницу**. На экране отобразится защищенный веб-сайт IIS, как в показано следующем примере:

![Тестирование базового URL-адреса в шлюзе приложений](./media/redirect-http-to-https-powershell/application-gateway-iistest.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Перезапись заголовков HTTP и URL-адреса с помощью шлюза приложений](rewrite-http-headers-url.md)