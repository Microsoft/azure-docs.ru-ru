---
title: Краткое руководство. Направление веб-трафика с помощью PowerShell
titleSuffix: Azure Application Gateway
description: В этом кратком руководстве показано, как с помощью Azure PowerShell создать Шлюз приложений Azure, который направляет веб-трафик к виртуальным машинам в серверном пуле.
services: application-gateway
author: vhorne
ms.author: victorh
ms.date: 01/19/2021
ms.topic: quickstart
ms.service: application-gateway
ms.custom:
- mvc
- mode-api
ms.openlocfilehash: a4aa16ba4334e5f1e035face9549f4c03ad3a14f
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107538557"
---
# <a name="quickstart-direct-web-traffic-with-azure-application-gateway-using-azure-powershell"></a>Краткое руководство. Направление веб-трафика с помощью Шлюза приложений Azure в Azure PowerShell

В этом кратком руководстве рассказывается о том, как создать шлюз приложений с помощью Azure PowerShell, а затем протестировать его, чтобы убедиться в его правильной работе. 

Шлюз приложений направляет веб-трафик приложений на определенные ресурсы в серверном пуле. Вы назначаете прослушиватели портам, создаете правила и добавляете ресурсы в серверный пул. Для наглядности в этой статье применяются: простая настройка с открытым интерфейсным IP-адресом, базовый прослушиватель для размещения одного сайта на шлюзе приложений, базовое правило маршрутизации запросов и две виртуальные машины, используемые для серверного пула.

Инструкции в этом кратком руководстве можно также выполнить с помощью [Azure CLI](quick-create-cli.md) или [портала Azure](quick-create-portal.md).

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [Azure PowerShell версии 1.0.0 или более поздней версии](/powershell/azure/install-az-ps) (при локальном запуске Azure PowerShell).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="connect-to-azure"></a>Подключение к Azure

Чтобы подключиться к Azure, выполните `Connect-AzAccount`.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

В Azure выделите связанные ресурсы группе ресурсов. Вы можете выбрать существующую группу ресурсов или создать новую.

Чтобы создать новую группу ресурсов, используйте командлет `New-AzResourceGroup`. 

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroupAG -Location eastus
```
## <a name="create-network-resources"></a>Создание сетевых ресурсов

В Azure для обмена между создаваемыми ресурсами необходима виртуальная сеть.  Подсеть шлюза приложений может содержать только шлюзы приложений. Другие ресурсы запрещены.  Вы можете создать новую подсеть для шлюза приложений или использовать уже существующую. В этом примере создаются две подсети: одна — для шлюза приложений, а вторая — для внутренних серверов. Вы можете настроить интерфейсный IP-адрес как общедоступный или частный в зависимости от варианта использования Шлюза приложений. В этом примере мы будем использовать общедоступный интерфейсный IP-адрес.

1. Создайте конфигурации подсети с помощью `New-AzVirtualNetworkSubnetConfig`.
2. Создайте виртуальную сеть с конфигурациями подсетей с помощью `New-AzVirtualNetwork`. 
3. Создайте общедоступный IP-адрес с помощью `New-AzPublicIpAddress`. 
> [!NOTE]
> [Политики конечных точек служб для виртуальной сети](../virtual-network/virtual-network-service-endpoint-policies-overview.md) в настоящее время не поддерживаются в подсети Шлюза приложений.

```azurepowershell-interactive
$agSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myAGSubnet `
  -AddressPrefix 10.0.1.0/24
$backendSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myBackendSubnet `
  -AddressPrefix 10.0.2.0/24
New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myVNet `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $agSubnetConfig, $backendSubnetConfig
New-AzPublicIpAddress `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myAGPublicIPAddress `
  -AllocationMethod Static `
  -Sku Standard
```
## <a name="create-an-application-gateway"></a>Создание шлюза приложений

### <a name="create-the-ip-configurations-and-frontend-port"></a>Создание IP-конфигураций и интерфейсного порта

1. Используйте `New-AzApplicationGatewayIPConfiguration`, чтобы создать конфигурацию, которая позволяет связать созданную подсеть со шлюзом приложений. 
2. Используйте командлет `New-AzApplicationGatewayFrontendIPConfig`, чтобы создать конфигурацию, которая позволяет назначить для шлюза приложений созданный вами общедоступный IP-адрес. 
3. Используйте `New-AzApplicationGatewayFrontendPort`, чтобы назначить порт 80 для доступа к шлюзу приложений.

```azurepowershell-interactive
$vnet   = Get-AzVirtualNetwork -ResourceGroupName myResourceGroupAG -Name myVNet
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name myAGSubnet
$pip    = Get-AzPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress 
$gipconfig = New-AzApplicationGatewayIPConfiguration `
  -Name myAGIPConfig `
  -Subnet $subnet
$fipconfig = New-AzApplicationGatewayFrontendIPConfig `
  -Name myAGFrontendIPConfig `
  -PublicIPAddress $pip
$frontendport = New-AzApplicationGatewayFrontendPort `
  -Name myFrontendPort `
  -Port 80
```

### <a name="create-the-backend-pool"></a>Создание внутреннего пула

1. Используйте `New-AzApplicationGatewayBackendAddressPool`, чтобы создать серверный пул для шлюза приложений. Серверный пул пока пуст. Позже вы добавите в него создаваемые сетевые адаптеры внутреннего сервера (см. следующий раздел).
2. Настройте параметры для серверного пула с помощью `New-AzApplicationGatewayBackendHttpSetting`.

```azurepowershell-interactive
$backendPool = New-AzApplicationGatewayBackendAddressPool `
  -Name myAGBackendPool
$poolSettings = New-AzApplicationGatewayBackendHttpSetting `
  -Name myPoolSettings `
  -Port 80 `
  -Protocol Http `
  -CookieBasedAffinity Enabled `
  -RequestTimeout 30
```

### <a name="create-the-listener-and-add-a-rule"></a>Создание прослушивателя и добавление правила

Прослушиватель требуется для того, чтобы шлюз приложений правильно маршрутизировал трафик на внутренние пулы. Azure требует правило для того, чтобы указать прослушивателю, какой внутренний пул использовать для входящего трафика. 

1. Создайте прослушиватель созданными ранее конфигурацией внешнего интерфейса и интерфейсным портом, используя командлет `New-AzApplicationGatewayHttpListener`. 
2. Используйте `New-AzApplicationGatewayRequestRoutingRule`, чтобы создать правило с именем *rule1*. 

```azurepowershell-interactive
$defaultlistener = New-AzApplicationGatewayHttpListener `
  -Name myAGListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendport
$frontendRule = New-AzApplicationGatewayRequestRoutingRule `
  -Name rule1 `
  -RuleType Basic `
  -HttpListener $defaultlistener `
  -BackendAddressPool $backendPool `
  -BackendHttpSettings $poolSettings
```

### <a name="create-the-application-gateway"></a>Создание шлюза приложений

Теперь, поскольку вы уже создали необходимые вспомогательные ресурсы, создайте шлюз приложений.

1. Используйте `New-AzApplicationGatewaySku`, чтобы задать параметры для шлюза приложений.
2. Используйте `New-AzApplicationGateway`, чтобы создать шлюз приложений.

```azurepowershell-interactive
$sku = New-AzApplicationGatewaySku `
  -Name Standard_v2 `
  -Tier Standard_v2 `
  -Capacity 2
New-AzApplicationGateway `
  -Name myAppGateway `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -BackendAddressPools $backendPool `
  -BackendHttpSettingsCollection $poolSettings `
  -FrontendIpConfigurations $fipconfig `
  -GatewayIpConfigurations $gipconfig `
  -FrontendPorts $frontendport `
  -HttpListeners $defaultlistener `
  -RequestRoutingRules $frontendRule `
  -Sku $sku
```

### <a name="backend-servers"></a>Внутренние серверы

Создав Шлюз приложений, создайте серверные виртуальные машины, на которых будут размещаться веб-сайты. Серверная часть может состоять из сетевых адаптеров, масштабируемых наборов виртуальных машин, общедоступного IP-адреса, внутреннего IP-адреса, полных доменных имен и таких мультитенантых серверных частей, как Служба приложений Azure. 

В этом примере вы создадите две виртуальные машины, которые будут использоваться как внутренние серверы для шлюза приложений. Вы также установите службы IIS на виртуальных машинах, чтобы убедиться, что шлюз приложений успешно создан в Azure.

#### <a name="create-two-virtual-machines"></a>Создание двух виртуальных машин

1. Получите недавно созданную конфигурацию серверного пула Шлюза приложений с помощью `Get-AzApplicationGatewayBackendAddressPool`.
2. Создайте сетевой интерфейс с помощью `New-AzNetworkInterface`.
3. Создайте конфигурацию виртуальной машины с помощью `New-AzVMConfig`.
4. Создайте виртуальную машину с помощью `New-AzVM`.

Во время запуска следующего примера кода для создания виртуальных машин Azure запрашивает ваши учетные данные. Введите имя пользователя и пароль:
    
```azurepowershell-interactive
$appgw = Get-AzApplicationGateway -ResourceGroupName myResourceGroupAG -Name myAppGateway
$backendPool = Get-AzApplicationGatewayBackendAddressPool -Name myAGBackendPool -ApplicationGateway $appgw
$vnet   = Get-AzVirtualNetwork -ResourceGroupName myResourceGroupAG -Name myVNet
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name myBackendSubnet
$cred = Get-Credential
for ($i=1; $i -le 2; $i++)
{
  $nic = New-AzNetworkInterface `
    -Name myNic$i `
    -ResourceGroupName myResourceGroupAG `
    -Location EastUS `
    -Subnet $subnet `
    -ApplicationGatewayBackendAddressPool $backendpool
  $vm = New-AzVMConfig `
    -VMName myVM$i `
    -VMSize Standard_DS2_v2
  Set-AzVMOperatingSystem `
    -VM $vm `
    -Windows `
    -ComputerName myVM$i `
    -Credential $cred
  Set-AzVMSourceImage `
    -VM $vm `
    -PublisherName MicrosoftWindowsServer `
    -Offer WindowsServer `
    -Skus 2016-Datacenter `
    -Version latest
  Add-AzVMNetworkInterface `
    -VM $vm `
    -Id $nic.Id
  Set-AzVMBootDiagnostic `
    -VM $vm `
    -Disable
  New-AzVM -ResourceGroupName myResourceGroupAG -Location EastUS -VM $vm
  Set-AzVMExtension `
    -ResourceGroupName myResourceGroupAG `
    -ExtensionName IIS `
    -VMName myVM$i `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
    -Location EastUS
}
```

## <a name="test-the-application-gateway"></a>Тестирование шлюза приложений

Для создания шлюза приложений не требуется устанавливать службы IIS. Но в рамках этого руководства они устанавливаются, чтобы проверить, создан ли он.

Используйте IIS для тестирования шлюза приложений.

1. Запустите командлет `Get-AzPublicIPAddress`, чтобы получить общедоступный IP-адрес шлюза приложений. 
2. Скопируйте и вставьте общедоступный IP-адрес в адресную строку браузера. Когда вы обновляете браузер, должно отобразиться имя виртуальной машины. Допустимый ответ подтверждает, что шлюз приложений создан и может успешно подключиться к серверу.

```azurepowershell-interactive
Get-AzPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress
```

![Тестирование шлюза приложений](./media/quick-create-powershell/application-gateway-iistest.png)


## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам уже не нужны ресурсы, созданные с помощью шлюза приложений, удалите группу ресурсов. Удалив ее, вы также удалите шлюз приложений и все связанные с ним ресурсы. 

Чтобы удалить группу ресурсов, вызовите командлет `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroupAG
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Manage web traffic with an application gateway using Azure PowerShell](./tutorial-manage-web-traffic-powershell.md) (Управление веб-трафиком с помощью шлюза приложений в Azure PowerShell)
