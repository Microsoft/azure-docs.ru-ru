---
title: Учебник. Улучшение доступа к веб-приложениям с помощью Шлюза приложений Azure
description: В этом руководстве вы узнаете, как создать автоматически масштабируемый, избыточный в пределах зоны шлюз приложений с зарезервированным IP-адресом с помощью Azure PowerShell.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: tutorial
ms.date: 03/08/2021
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 2a756313a4659dfc531289c2c86890371f700367
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102452294"
---
# <a name="tutorial-create-an-application-gateway-that-improves-web-application-access"></a>Учебник. Создание шлюза приложений, который улучшает доступ к веб-приложениям

Если вы ИТ-администратор и желаете улучшить доступ к веб-приложениям, то можете оптимизировать шлюз приложений таким образом, чтобы он выполнял масштабирование на основе пользовательского спроса и охватывал несколько зон доступности. Это руководство поможет вам настроить компоненты Шлюза приложений Azure, которые отвечают за автоматическое масштабирование, избыточность в пределах зоны и зарезервированные виртуальные IP-адреса (статический IP-адрес). Чтобы решить проблему, вы будете использовать командлеты Azure PowerShell и модель развертывания с помощью Azure Resource Manager.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание самозаверяющего сертификата
> * Создание автоматически масштабируемой виртуальной сети.
> * Создание зарезервированного общедоступного IP-адреса
> * Настройка инфраструктуры шлюза приложений.
> * Настройка автомасштабирования
> * Создание шлюза приложений
> * Тестирование шлюза приложений

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Для работы с этим учебником требуется запустить Azure PowerShell в локальной среде с правами администратора. Необходим модуль Azure PowerShell 1.0.0 или более поздней версии. Чтобы узнать версию, выполните команду `Get-Module -ListAvailable Az`. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). После проверки версии PowerShell выполните командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="sign-in-to-azure"></a>Вход в Azure

```azurepowershell
Connect-AzAccount
Select-AzSubscription -Subscription "<sub name>"
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов
Создайте группу ресурсов в одном из доступных расположений.

```azurepowershell
$location = "East US 2"
$rg = "AppGW-rg"

#Create a new Resource Group
New-AzResourceGroup -Name $rg -Location $location
```

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

Воспользуйтесь отпечатком, чтобы создать файл PFX. Замените *\<password>* своим паролем:

```powershell
$pwd = ConvertTo-SecureString -String "<password>" -Force -AsPlainText

Export-PfxCertificate `
  -cert cert:\localMachine\my\E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630 `
  -FilePath c:\appgwcert.pfx `
  -Password $pwd
```


## <a name="create-a-virtual-network"></a>Создание виртуальной сети

Создайте виртуальную сеть с одной выделенной подсетью для автоматически масштабируемого шлюза приложений. В настоящее время в каждой выделенной подсети можно развернуть только один автоматически масштабируемый шлюз приложений.

```azurepowershell
#Create VNet with two subnets
$sub1 = New-AzVirtualNetworkSubnetConfig -Name "AppGwSubnet" -AddressPrefix "10.0.0.0/24"
$sub2 = New-AzVirtualNetworkSubnetConfig -Name "BackendSubnet" -AddressPrefix "10.0.1.0/24"
$vnet = New-AzvirtualNetwork -Name "AutoscaleVNet" -ResourceGroupName $rg `
       -Location $location -AddressPrefix "10.0.0.0/16" -Subnet $sub1, $sub2
```

## <a name="create-a-reserved-public-ip"></a>Создание зарезервированного общедоступного IP-адреса

Укажите метод распределения PublicIPAddress как **статический**. Виртуальный IP-адрес автоматически масштабируемого шлюза приложений может быть только статическим. Динамические IP-адреса не поддерживаются. Поддерживается только номер SKU "Стандартный" для PublicIpAddress.

```azurepowershell
#Create static public IP
$pip = New-AzPublicIpAddress -ResourceGroupName $rg -name "AppGwVIP" `
       -location $location -AllocationMethod Static -Sku Standard -Zone 1,2,3
```

## <a name="retrieve-details"></a>Получение сведений

Извлеките сведения о группе ресурсов, подсети и IP-адресе в локальный объект, чтобы создать сведения об IP-конфигурации шлюза приложений.

```azurepowershell
$publicip = Get-AzPublicIpAddress -ResourceGroupName $rg -name "AppGwVIP"
$vnet = Get-AzvirtualNetwork -Name "AutoscaleVNet" -ResourceGroupName $rg
$gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "AppGwSubnet" -VirtualNetwork $vnet
```

## <a name="create-web-apps"></a>Создание веб-приложений

Настройте два веб-приложения для серверного пула. Замените *\<site1-name>* и *\<site-2-name>* уникальными именами в домене `azurewebsites.net`.

```azurepowershell
New-AzAppServicePlan -ResourceGroupName $rg -Name "ASP-01"  -Location $location -Tier Basic `
   -NumberofWorkers 2 -WorkerSize Small
New-AzWebApp -ResourceGroupName $rg -Name <site1-name> -Location $location -AppServicePlan ASP-01
New-AzWebApp -ResourceGroupName $rg -Name <site2-name> -Location $location -AppServicePlan ASP-01
```

## <a name="configure-the-infrastructure"></a>Настройка инфраструктуры

Настройте IP-конфигурацию, IP-конфигурацию внешнего интерфейса, внутренний пул, параметры HTTP, сертификат, порт, прослушиватель и правило в формате, идентичном конфигурации имеющегося шлюза приложений ценовой категории "Стандартный". В новом номере SKU используется та же объектная модель, что и в номере SKU "Стандартный".

Замените FQDN двух веб-приложений (например, `mywebapp.azurewebsites.net`) в определении переменной $pool.

```azurepowershell
$ipconfig = New-AzApplicationGatewayIPConfiguration -Name "IPConfig" -Subnet $gwSubnet
$fip = New-AzApplicationGatewayFrontendIPConfig -Name "FrontendIPCOnfig" -PublicIPAddress $publicip
$pool = New-AzApplicationGatewayBackendAddressPool -Name "Pool1" `
       -BackendIPAddresses <your first web app FQDN>, <your second web app FQDN>
$fp01 = New-AzApplicationGatewayFrontendPort -Name "SSLPort" -Port 443
$fp02 = New-AzApplicationGatewayFrontendPort -Name "HTTPPort" -Port 80

$securepfxpwd = ConvertTo-SecureString -String "Azure123456!" -AsPlainText -Force
$sslCert01 = New-AzApplicationGatewaySslCertificate -Name "SSLCert" -Password $securepfxpwd `
            -CertificateFile "c:\appgwcert.pfx"
$listener01 = New-AzApplicationGatewayHttpListener -Name "SSLListener" `
             -Protocol Https -FrontendIPConfiguration $fip -FrontendPort $fp01 -SslCertificate $sslCert01
$listener02 = New-AzApplicationGatewayHttpListener -Name "HTTPListener" `
             -Protocol Http -FrontendIPConfiguration $fip -FrontendPort $fp02

$setting = New-AzApplicationGatewayBackendHttpSettings -Name "BackendHttpSetting1" `
          -Port 80 -Protocol Http -CookieBasedAffinity Disabled -PickHostNameFromBackendAddress
$rule01 = New-AzApplicationGatewayRequestRoutingRule -Name "Rule1" -RuleType basic `
         -BackendHttpSettings $setting -HttpListener $listener01 -BackendAddressPool $pool
$rule02 = New-AzApplicationGatewayRequestRoutingRule -Name "Rule2" -RuleType basic `
         -BackendHttpSettings $setting -HttpListener $listener02 -BackendAddressPool $pool
```

## <a name="specify-autoscale"></a>Настройка автомасштабирования

Теперь можно указать конфигурацию автомасштабирования для шлюза приложений. 

   ```azurepowershell
   $autoscaleConfig = New-AzApplicationGatewayAutoscaleConfiguration -MinCapacity 2
   $sku = New-AzApplicationGatewaySku -Name Standard_v2 -Tier Standard_v2
   ```
В этом режиме шлюз приложений автоматически масштабируется в соответствии с шаблоном трафика приложений.

## <a name="create-the-application-gateway"></a>Создание шлюза приложений

Создайте шлюз приложений с зонами избыточности и конфигурацией автомасштабирования.

```azurepowershell
$appgw = New-AzApplicationGateway -Name "AutoscalingAppGw" -Zone 1,2,3 `
  -ResourceGroupName $rg -Location $location -BackendAddressPools $pool `
  -BackendHttpSettingsCollection $setting -GatewayIpConfigurations $ipconfig `
  -FrontendIpConfigurations $fip -FrontendPorts $fp01, $fp02 `
  -HttpListeners $listener01, $listener02 -RequestRoutingRules $rule01, $rule02 `
  -Sku $sku -sslCertificates $sslCert01 -AutoscaleConfiguration $autoscaleConfig
```

## <a name="test-the-application-gateway"></a>Тестирование шлюза приложений

Используйте командлет Get-AzPublicIPAddress, чтобы получить общедоступный IP-адрес шлюза приложений. Скопируйте общедоступный IP-адрес или DNS-имя и вставьте его в адресную строку браузера.

```azurepowershell
$pip = Get-AzPublicIPAddress -ResourceGroupName $rg -Name AppGwVIP
$pip.IpAddress
```


## <a name="clean-up-resources"></a>Очистка ресурсов

Сначала изучите ресурсы, которые были созданы с помощью шлюза приложений. Потом выполните команду `Remove-AzResourceGroup`, чтобы удалить группу ресурсов, шлюз приложений и все связанные ресурсы, если они вам больше не нужны.

`Remove-AzResourceGroup -Name $rg`

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание шлюза приложений с правилами маршрутизации на основе URL-путей](./tutorial-url-route-powershell.md)