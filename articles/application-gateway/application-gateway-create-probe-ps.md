---
title: Создание пользовательской пробы с помощью PowerShell
titleSuffix: Azure Application Gateway
description: Узнайте, как создать пользовательскую проверку для шлюза приложений с помощью PowerShell в диспетчере ресурсов.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 07/09/2020
ms.author: victorh
ms.openlocfilehash: 63f4b28915a932398e5417f8ea9be3fe4d68e705
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93397847"
---
# <a name="create-a-custom-probe-for-azure-application-gateway-by-using-powershell-for-azure-resource-manager"></a>Создание пользовательской проверки для шлюза приложений с помощью PowerShell для диспетчера ресурсов Azure

> [!div class="op_single_selector"]
> * [Портал Azure](application-gateway-create-probe-portal.md)
> * [PowerShell и диспетчер ресурсов Azure](application-gateway-create-probe-ps.md)
> * [Классическая оболочка Azure](application-gateway-create-probe-classic-ps.md)

Следуя инструкциям этой статьи вы добавите пользовательскую пробу в имеющийся шлюз приложений с помощью PowerShell. Пользовательские пробы полезны в приложениях с конкретной страницей проверки работоспособности или приложениях, не предоставляющих успешный ответ веб-приложению по умолчанию.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## <a name="create-an-application-gateway-with-a-custom-probe"></a>Создание шлюза приложений с пользовательской пробой

### <a name="sign-in-and-create-resource-group"></a>Вход и создание группы ресурсов

1. Используйте `Connect-AzAccount` для аутентификации.

   ```powershell
   Connect-AzAccount
   ```

1. Получите подписку для учетной записи.

   ```powershell
   Get-AzSubscription
   ```

1. Выберите, какие подписки Azure будут использоваться.

   ```powershell
   Select-AzSubscription -Subscriptionid '{subscriptionGuid}'
   ```

1. Создайте группу ресурсов. Если у вас есть группа ресурсов, можно пропустить этот шаг.

   ```powershell
   New-AzResourceGroup -Name appgw-rg -Location 'West US'
   ```

Диспетчер ресурсов Azure требует, чтобы все группы ресурсов указывали расположение. Это расположение используется в качестве расположения по умолчанию для всех ресурсов данной группы. Убедитесь, что во всех командах для создания шлюза приложений используется одна группа ресурсов.

В примере выше мы создали группу ресурсов **appgw-RG** в **западной части США**.

### <a name="create-a-virtual-network-and-a-subnet"></a>Создание виртуальной сети и подсети

Следующий пример кода создает виртуальную сеть и подсеть шлюза приложений. Шлюзу приложений требуется собственная подсеть. По этой причине созданная для него подсеть должна быть меньше адресного пространства виртуальной сети. Это позволит создавать и использовать другие подсети.

```powershell
# Assign the address range 10.0.0.0/24 to a subnet variable to be used to create a virtual network.
$subnet = New-AzVirtualNetworkSubnetConfig -Name subnet01 -AddressPrefix 10.0.0.0/24

# Create a virtual network named appgwvnet in resource group appgw-rg for the West US region using the prefix 10.0.0.0/16 with subnet 10.0.0.0/24.
$vnet = New-AzVirtualNetwork -Name appgwvnet -ResourceGroupName appgw-rg -Location 'West US' -AddressPrefix 10.0.0.0/16 -Subnet $subnet

# Assign a subnet variable for the next steps, which create an application gateway.
$subnet = $vnet.Subnets[0]
```

### <a name="create-a-public-ip-address-for-the-front-end-configuration"></a>Создание общедоступного IP-адреса для конфигурации интерфейсной части

Создайте ресурс общедоступного IP-адреса **publicIP01** в группе ресурсов **appgw-RG** для региона "Западная часть США". В этом примере для интерфейсных IP-адресов шлюза приложений используются общедоступные IP-адреса.  Шлюзу приложений требуется общедоступный IP-адрес с динамически созданным DNS-именем, поэтому при создании общедоступных IP-адресов нельзя указывать `-DomainNameLabel`.

```powershell
$publicip = New-AzPublicIpAddress -ResourceGroupName appgw-rg -Name publicIP01 -Location 'West US' -AllocationMethod Dynamic
```

### <a name="create-an-application-gateway"></a>Создание шлюза приложений

Перед созданием шлюза приложений необходимо настроить все элементы конфигурации. Пример кода ниже создает элементы конфигурации, необходимые для ресурса шлюза приложений.

| **Компонент** | **Описание** |
|---|---|
| **Конфигурация IP-адреса шлюза** | Конфигурация IP-адреса шлюза приложений.|
| **Серверный пул** | Пул IP-адресов, полное доменное имя или сетевые адаптеры серверов приложений, на которых размещается веб-приложение.|
| **Зонд работоспособности** | Пользовательская проба, используемая для мониторинга работоспособности участников серверного пула.|
| **Параметры HTTP** | Коллекция параметров, в том числе порта, протокола, сходства на основе файлов cookie, пробы и времени ожидания.  Эти параметры определяют передачу трафика участникам серверного пула.|
| **Интерфейсный порт** | Порт, на котором шлюз приложений прослушивает трафик.|
| **Средство прослушивания** | Сочетание протокола, конфигурации интерфейсного IP-адреса и интерфейсного порта. Это компонент, который прослушивает входящие запросы.
|**Правило**| Направляет трафик в соответствующую серверную часть на основе параметров HTTP.|

```powershell
# Creates an application gateway Frontend IP configuration named gatewayIP01
$gipconfig = New-AzApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet

#Creates a back-end IP address pool named pool01 with IP addresses 134.170.185.46, 134.170.188.221, 134.170.185.50.
$pool = New-AzApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 134.170.185.46, 134.170.188.221, 134.170.185.50

# Creates a probe that will check health at http://contoso.com/path/path.htm
$probe = New-AzApplicationGatewayProbeConfig -Name probe01 -Protocol Http -HostName 'contoso.com' -Path '/path/path.htm' -Interval 30 -Timeout 120 -UnhealthyThreshold 8

# Creates the backend http settings to be used. This component references the $probe created in the previous command.
$poolSetting = New-AzApplicationGatewayBackendHttpSettings -Name poolsetting01 -Port 80 -Protocol Http -CookieBasedAffinity Disabled -Probe $probe -RequestTimeout 80

# Creates a frontend port for the application gateway to listen on port 80 that will be used by the listener.
$fp = New-AzApplicationGatewayFrontendPort -Name frontendport01 -Port 80

# Creates a frontend IP configuration. This associates the $publicip variable defined previously with the front-end IP that will be used by the listener.
$fipconfig = New-AzApplicationGatewayFrontendIPConfig -Name fipconfig01 -PublicIPAddress $publicip

# Creates the listener. The listener is a combination of protocol and the frontend IP configuration $fipconfig and frontend port $fp created in previous steps.
$listener = New-AzApplicationGatewayHttpListener -Name listener01  -Protocol Http -FrontendIPConfiguration $fipconfig -FrontendPort $fp

# Creates the rule that routes traffic to the backend pools.  In this example we create a basic rule that uses the previous defined http settings and backend address pool.  It also associates the listener to the rule
$rule = New-AzApplicationGatewayRequestRoutingRule -Name rule01 -RuleType Basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool

# Sets the SKU of the application gateway, in this example we create a small standard application gateway with 2 instances.
$sku = New-AzApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2

# The final step creates the application gateway with all the previously defined components.
$appgw = New-AzApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg -Location 'West US' -BackendAddressPools $pool -Probes $probe -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig  -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku
```

## <a name="add-a-probe-to-an-existing-application-gateway"></a>Добавление проверки для существующего шлюза приложений

Следующий фрагмент кода добавляет пробу в имеющийся шлюз приложений.

```powershell
# Load the application gateway resource into a PowerShell variable by using Get-AzApplicationGateway.
$getgw =  Get-AzApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg

# Create the probe object that will check health at http://contoso.com/path/path.htm
$getgw = Add-AzApplicationGatewayProbeConfig -ApplicationGateway $getgw -Name probe01 -Protocol Http -HostName 'contoso.com' -Path '/path/custompath.htm' -Interval 30 -Timeout 120 -UnhealthyThreshold 8

# Set the backend HTTP settings to use the new probe
$getgw = Set-AzApplicationGatewayBackendHttpSettings -ApplicationGateway $getgw -Name $getgw.BackendHttpSettingsCollection.name -Port 80 -Protocol Http -CookieBasedAffinity Disabled -Probe $probe -RequestTimeout 120

# Save the application gateway with the configuration changes
Set-AzApplicationGateway -ApplicationGateway $getgw
```

## <a name="remove-a-probe-from-an-existing-application-gateway"></a>Удаление проверки из существующего шлюза приложений

Следующий фрагмент кода удаляет пробу из имеющегося шлюза приложений.

```powershell
# Load the application gateway resource into a PowerShell variable by using Get-AzApplicationGateway.
$getgw =  Get-AzApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg

# Remove the probe from the application gateway configuration object
$getgw = Remove-AzApplicationGatewayProbeConfig -ApplicationGateway $getgw -Name $getgw.Probes.name

# Set the backend HTTP settings to remove the reference to the probe. The backend http settings now use the default probe
$getgw = Set-AzApplicationGatewayBackendHttpSettings -ApplicationGateway $getgw -Name $getgw.BackendHttpSettingsCollection.name -Port 80 -Protocol http -CookieBasedAffinity Disabled

# Save the application gateway with the configuration changes
Set-AzApplicationGateway -ApplicationGateway $getgw
```

## <a name="get-application-gateway-dns-name"></a>Получение DNS-имени шлюза приложений

После создания шлюза следует настроить внешний интерфейс для обмена данными. Если вы используете общедоступный IP-адрес, шлюзу приложений требуется динамически назначаемое непонятное имя DNS. Чтобы гарантировать попадание пользователей на шлюз приложений, можно использовать запись CNAME, чтобы указать общедоступную конечную точку шлюза приложения. [Настройка пользовательского имени домена в Azure](../cloud-services/cloud-services-custom-domain-name-portal.md). Получите информацию о шлюзе приложений и соответствующее IP- или DNS-имя с помощью элемента PublicIPAddress, связанного со шлюзом приложений. DNS-имя шлюза приложений должно использоваться для создания записи CNAME, указывающей двум веб-приложениям на это DNS-имя. Использование записи A не рекомендуется, так как виртуальный IP-адрес может измениться после перезапуска приложения шлюза.

```powershell
Get-AzPublicIpAddress -ResourceGroupName appgw-RG -Name publicIP01
```

```
Name                     : publicIP01
ResourceGroupName        : appgw-RG
Location                 : westus
Id                       : /subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/publicIPAddresses/publicIP01
Etag                     : W/"00000d5b-54ed-4907-bae8-99bd5766d0e5"
ResourceGuid             : 00000000-0000-0000-0000-000000000000
ProvisioningState        : Succeeded
Tags                     : 
PublicIpAllocationMethod : Dynamic
IpAddress                : xx.xx.xxx.xx
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : {
                                "Id": "/subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/applicationGateways/appgwtest/frontendIP
                            Configurations/frontend1"
                            }
DnsSettings              : {
                                "Fqdn": "00000000-0000-xxxx-xxxx-xxxxxxxxxxxx.cloudapp.net"
                            }
```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как настроить разгрузку TLS, посетив страницу [Настройка разгрузки TLS](./tutorial-ssl-powershell.md) .