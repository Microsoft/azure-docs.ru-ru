---
title: 'VPN-шлюз Azure: Удаление шлюза: PowerShell'
description: Удаление шлюза виртуальной сети посредством PowerShell в модели развертывания с помощью Resource Manager.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.date: 09/03/2020
ms.author: cherylmc
ms.topic: how-to
ms.openlocfilehash: 48e0998719ea19c0b360b50d8c9171d2b448ad19
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89440768"
---
# <a name="delete-a-virtual-network-gateway-using-powershell"></a>Удаление шлюза виртуальной сети с помощью PowerShell
> [!div class="op_single_selector"]
> * [Портал Azure](vpn-gateway-delete-vnet-gateway-portal.md)
> * [PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
> * [PowerShell (классическая модель)](vpn-gateway-delete-vnet-gateway-classic-powershell.md)
>
>

Существует несколько разных подходов, которые можно применить, когда требуется удалить шлюз виртуальной сети для настройки VPN-шлюза.

- Если вы хотите все удалить и начать заново, как в случае с тестовой средой, то можете удалить группу ресурсов. При удалении группы ресурсов удаляются все ресурсы в этой группе. Этот метод рекомендуется, только если вы не хотите оставить какие-либо ресурсы из группы ресурсов. При таком подходе невозможно выборочно удалить только некоторые из ресурсов.

- Если вы хотите сохранить некоторые ресурсы в группе ресурсов, то удаление шлюза виртуальной сети немного усложняется. Прежде чем удалить шлюз виртуальной сети, необходимо удалить все ресурсы, зависящие от него. Выполняемые действия зависят от типа подключений, которые были созданы, и зависимых ресурсов для каждого подключения.

## <a name="before-beginning"></a>Подготовка



### <a name="1-download-the-latest-azure-resource-manager-powershell-cmdlets"></a>1. Скачайте последнюю версию командлетов PowerShell Azure Resource Manager.

Скачайте и установите последнюю версию командлетов PowerShell для Azure Resource Manager. Дополнительные сведения о скачивании и установке командлетов PowerShell см. в статье [Get started with Azure PowerShell cmdlets](/powershell/azure/) (Приступая к работе с командлетами Azure PowerShell).

### <a name="2-connect-to-your-azure-account"></a>2. Подключитесь к учетной записи Azure.

Откройте консоль PowerShell и подключитесь к своей учетной записи. Для подключения используйте следующий пример кода:

```powershell
Connect-AzAccount
```

Просмотрите подписки учетной записи.

```powershell
Get-AzSubscription
```

При наличии нескольких подписок укажите подписку, которую вы хотите использовать.

```powershell
Select-AzSubscription -SubscriptionName "Replace_with_your_subscription_name"
```

## <a name="delete-a-site-to-site-vpn-gateway"></a><a name="S2S"></a>Удаление VPN-шлюза типа "сеть — сеть"

Чтобы удалить шлюз виртуальной сети для конфигурации "сеть — сеть", необходимо сначала удалить каждый ресурс, относящийся к этому шлюзу виртуальной сети. Ресурсы должны быть удалены в определенном порядке из-за зависимостей. При работе с приведенными ниже примерами некоторые значения необходимо указывать, тогда как другие значения находятся в выходных данных. В примерах для демонстрационных целей мы используем следующие конкретные значения.

Имя виртуальной сети: VNet1.<br>
Имя группы ресурсов: RG1.<br>
Имя шлюза виртуальной сети: GW1.<br>

Приведенные ниже инструкции относятся к модели развертывания с помощью Resource Manager.

### <a name="1-get-the-virtual-network-gateway-that-you-want-to-delete"></a>1. получите шлюз виртуальной сети, который необходимо удалить.

```powershell
$GW=get-Azvirtualnetworkgateway -Name "GW1" -ResourceGroupName "RG1"
```

### <a name="2-check-to-see-if-the-virtual-network-gateway-has-any-connections"></a>2. Проверьте наличие подключений у шлюза виртуальной сети.

```powershell
get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG1" | where-object {$_.VirtualNetworkGateway1.Id -eq $GW.Id}
$Conns=get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG1" | where-object {$_.VirtualNetworkGateway1.Id -eq $GW.Id}
```

### <a name="3-delete-all-connections"></a>3. Удалите все подключения.

Может потребоваться подтвердить удаление каждого подключения.

```powershell
$Conns | ForEach-Object {Remove-AzVirtualNetworkGatewayConnection -Name $_.name -ResourceGroupName $_.ResourceGroupName}
```

### <a name="4-delete-the-virtual-network-gateway"></a>4. Удалите шлюз виртуальной сети.

Может потребоваться подтвердить его удаление. Если кроме конфигурации типа "сеть — сеть" в этой виртуальной сети есть конфигурация типа "точка — сеть", то удаление шлюза виртуальной сети приведет к автоматическому отключению всех клиентов P2S без предупреждения.


```powershell
Remove-AzVirtualNetworkGateway -Name "GW1" -ResourceGroupName "RG1"
```

На этом этапе шлюз виртуальной сети будет удален. Вы можете выполнить следующие шаги, чтобы удалить любые неиспользуемые ресурсы.

### <a name="5-delete-the-local-network-gateways"></a>5. Удалите шлюзы локальной сети.

Получите список соответствующих шлюзов локальной сети.

```powershell
$LNG=Get-AzLocalNetworkGateway -ResourceGroupName "RG1" | where-object {$_.Id -In $Conns.LocalNetworkGateway2.Id}
```

Удалите шлюзы локальной сети. Может потребоваться подтвердить удаление каждого из них.

```powershell
$LNG | ForEach-Object {Remove-AzLocalNetworkGateway -Name $_.Name -ResourceGroupName $_.ResourceGroupName}
```

### <a name="6-delete-the-public-ip-address-resources"></a>6. Удалите ресурсы общедоступного IP-адреса.

Получите конфигурации IP шлюза виртуальной сети.

```powershell
$GWIpConfigs = $Gateway.IpConfigurations
```

Получите список ресурсов с общедоступным IP-адресом, используемых для этого шлюза виртуальной сети. Если шлюз виртуальной сети работал в режиме "активный — активный", вы увидите два общедоступных IP-адреса.

```powershell
$PubIP=Get-AzPublicIpAddress | where-object {$_.Id -In $GWIpConfigs.PublicIpAddress.Id}
```

Удалите ресурсы с общедоступным IP-адресом.

```powershell
$PubIP | foreach-object {remove-AzpublicIpAddress -Name $_.Name -ResourceGroupName "RG1"}
```

### <a name="7-delete-the-gateway-subnet-and-set-the-configuration"></a>7. Удалите подсеть шлюза и настройте конфигурацию.

```powershell
$GWSub = Get-AzVirtualNetwork -ResourceGroupName "RG1" -Name "VNet1" | Remove-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet"
Set-AzVirtualNetwork -VirtualNetwork $GWSub
```

## <a name="delete-a-vnet-to-vnet-vpn-gateway"></a><a name="v2v"></a>Удаление VPN-шлюза типа "виртуальная сеть — виртуальная сеть"

Чтобы удалить шлюз виртуальной сети для конфигурации "виртуальная сеть — виртуальная сеть", необходимо сначала удалить каждый ресурс, относящийся к этому шлюзу виртуальной сети. Ресурсы должны быть удалены в определенном порядке из-за зависимостей. При работе с приведенными ниже примерами некоторые значения необходимо указывать, тогда как другие значения находятся в выходных данных. В примерах для демонстрационных целей мы используем следующие конкретные значения.

Имя виртуальной сети: VNet1.<br>
Имя группы ресурсов: RG1.<br>
Имя шлюза виртуальной сети: GW1.<br>

Приведенные ниже инструкции относятся к модели развертывания с помощью Resource Manager.

### <a name="1-get-the-virtual-network-gateway-that-you-want-to-delete"></a>1. получите шлюз виртуальной сети, который необходимо удалить.

```powershell
$GW=get-Azvirtualnetworkgateway -Name "GW1" -ResourceGroupName "RG1"
```

### <a name="2-check-to-see-if-the-virtual-network-gateway-has-any-connections"></a>2. Проверьте наличие подключений у шлюза виртуальной сети.

```powershell
get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG1" | where-object {$_.VirtualNetworkGateway1.Id -eq $GW.Id}
```
 
Могут существовать другие подключения к шлюзу виртуальной сети, входящие в другую группу ресурсов. Проверьте наличие дополнительных подключений в каждой дополнительной группе ресурсов. В этом примере выполняется проверка на наличие подключений из RG2. Выполните его для каждой имеющейся группы ресурсов, которая может быть подключена к шлюзу виртуальной сети.

```powershell
get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG2" | where-object {$_.VirtualNetworkGateway2.Id -eq $GW.Id}
```

### <a name="3-get-the-list-of-connections-in-both-directions"></a>3. получите список подключений в обоих направлениях.

Так как это конфигурация "виртуальная сеть — виртуальная сеть", необходим список подключений в обоих направлениях.

```powershell
$ConnsL=get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG1" | where-object {$_.VirtualNetworkGateway1.Id -eq $GW.Id}
```
 
В этом примере выполняется проверка на наличие подключений из RG2. Выполните его для каждой имеющейся группы ресурсов, которая может быть подключена к шлюзу виртуальной сети.

```powershell
 $ConnsR=get-Azvirtualnetworkgatewayconnection -ResourceGroupName "<NameOfResourceGroup2>" | where-object {$_.VirtualNetworkGateway2.Id -eq $GW.Id}
 ```

### <a name="4-delete-all-connections"></a>4. Удалите все подключения.

Может потребоваться подтвердить удаление каждого подключения.

```powershell
$ConnsL | ForEach-Object {Remove-AzVirtualNetworkGatewayConnection -Name $_.name -ResourceGroupName $_.ResourceGroupName}
$ConnsR | ForEach-Object {Remove-AzVirtualNetworkGatewayConnection -Name $_.name -ResourceGroupName $_.ResourceGroupName}
```

### <a name="5-delete-the-virtual-network-gateway"></a>5. Удалите шлюз виртуальной сети.

Может потребоваться подтвердить его удаление. Если кроме конфигурации типа "виртуальная сеть — виртуальная сеть" в ваших виртуальных сетях есть конфигурация типа "точка — сеть", то удаление шлюзов виртуальных сетей приведет к автоматическому отключению всех клиентов P2S без предупреждения.

```powershell
Remove-AzVirtualNetworkGateway -Name "GW1" -ResourceGroupName "RG1"
```

На этом этапе шлюз виртуальной сети будет удален. Вы можете выполнить следующие шаги, чтобы удалить любые неиспользуемые ресурсы.

### <a name="6-delete-the-public-ip-address-resources"></a>6. Удаление ресурсов общедоступного IP-адреса

Получите конфигурации IP шлюза виртуальной сети.

```powershell
$GWIpConfigs = $Gateway.IpConfigurations
```

Получите список ресурсов с общедоступным IP-адресом, используемых для этого шлюза виртуальной сети. Если шлюз виртуальной сети работал в режиме "активный — активный", вы увидите два общедоступных IP-адреса.

```powershell
$PubIP=Get-AzPublicIpAddress | where-object {$_.Id -In $GWIpConfigs.PublicIpAddress.Id}
```

Удалите ресурсы с общедоступным IP-адресом. Может потребоваться подтвердить их удаление.

```powershell
$PubIP | foreach-object {remove-AzpublicIpAddress -Name $_.Name -ResourceGroupName "<NameOfResourceGroup1>"}
```

### <a name="7-delete-the-gateway-subnet-and-set-the-configuration"></a>7. Удалите подсеть шлюза и настройте конфигурацию.

```powershell
$GWSub = Get-AzVirtualNetwork -ResourceGroupName "RG1" -Name "VNet1" | Remove-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet"
Set-AzVirtualNetwork -VirtualNetwork $GWSub
```

## <a name="delete-a-point-to-site-vpn-gateway"></a><a name="deletep2s"></a>Удаление VPN-шлюза типа "точка — сеть"

Чтобы удалить шлюз виртуальной сети для конфигурации "точка — сеть", необходимо сначала удалить каждый ресурс, относящийся к этому шлюзу виртуальной сети. Ресурсы должны быть удалены в определенном порядке из-за зависимостей. При работе с приведенными ниже примерами некоторые значения необходимо указывать, тогда как другие значения находятся в выходных данных. В примерах для демонстрационных целей мы используем следующие конкретные значения.

Имя виртуальной сети: VNet1.<br>
Имя группы ресурсов: RG1.<br>
Имя шлюза виртуальной сети: GW1.<br>

Приведенные ниже инструкции относятся к модели развертывания с помощью Resource Manager.


>[!NOTE]
> При удалении VPN-шлюза все подключенные клиенты будут отключены от виртуальной сети без предупреждения.
>
>

### <a name="1-get-the-virtual-network-gateway-that-you-want-to-delete"></a>1. получите шлюз виртуальной сети, который необходимо удалить.

```powershell
$GW=get-Azvirtualnetworkgateway -Name "GW1" -ResourceGroupName "RG1"
```

### <a name="2-delete-the-virtual-network-gateway"></a>2. Удалите шлюз виртуальной сети.

Может потребоваться подтвердить его удаление.

```powershell
Remove-AzVirtualNetworkGateway -Name "GW1" -ResourceGroupName "RG1"
```

На этом этапе шлюз виртуальной сети будет удален. Вы можете выполнить следующие шаги, чтобы удалить любые неиспользуемые ресурсы.

### <a name="3-delete-the-public-ip-address-resources"></a>3. Удалите ресурсы общедоступного IP-адреса.

Получите конфигурации IP шлюза виртуальной сети.

```powershell
$GWIpConfigs = $Gateway.IpConfigurations
```

Получите список общедоступных IP-адресов, используемых для этого шлюза виртуальной сети. Если шлюз виртуальной сети работал в режиме "активный — активный", вы увидите два общедоступных IP-адреса.

```powershell
$PubIP=Get-AzPublicIpAddress | where-object {$_.Id -In $GWIpConfigs.PublicIpAddress.Id}
```

Удалите эти общедоступные IP-адреса. Может потребоваться подтвердить их удаление.

```powershell
$PubIP | foreach-object {remove-AzpublicIpAddress -Name $_.Name -ResourceGroupName "<NameOfResourceGroup1>"}
```

### <a name="4-delete-the-gateway-subnet-and-set-the-configuration"></a>4. Удалите подсеть шлюза и настройте конфигурацию.

```powershell
$GWSub = Get-AzVirtualNetwork -ResourceGroupName "RG1" -Name "VNet1" | Remove-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet"
Set-AzVirtualNetwork -VirtualNetwork $GWSub
```

## <a name="delete-a-vpn-gateway-by-deleting-the-resource-group"></a><a name="delete"></a>Удаление VPN-шлюза путем удаления группы ресурсов

Если не требуется сохранить какие-либо ресурсы из группы ресурсов и вы просто хотите начать все заново, то вы можете удалить всю группу ресурсов. Это быстрый способ удалить все сразу. Приведенные ниже инструкции относятся только к модели развертывания с помощью Resource Manager.

### <a name="1-get-a-list-of-all-the-resource-groups-in-your-subscription"></a>1. получите список всех групп ресурсов в подписке.

```powershell
Get-AzResourceGroup
```

### <a name="2-locate-the-resource-group-that-you-want-to-delete"></a>2. Выберите группу ресурсов, которую необходимо удалить.

Найдите группу ресурсов, которую нужно удалить, и просмотрите список содержащихся в ней ресурсов. В этом примере группа ресурсов называется RG1. Измените пример, чтобы получить список всех ресурсов.

```powershell
Find-AzResource -ResourceGroupNameContains RG1
```

### <a name="3-verify-the-resources-in-the-list"></a>3. Проверьте ресурсы в списке.

Получив список ресурсов, просмотрите его, чтобы удостовериться, что вы хотите удалить все ресурсы в группе ресурсов, а также саму группу ресурсов. Если вы хотите сохранить какие-либо ресурсы из группы ресурсов, следуйте инструкциям в предыдущих разделах данной статьи, чтобы удалить шлюз.

### <a name="4-delete-the-resource-group-and-resources"></a>4. Удалите группу ресурсов и ресурсы.

Чтобы удалить группу ресурсов и ее ресурсы, измените приведенный пример и выполните его.

```powershell
Remove-AzResourceGroup -Name RG1
```

### <a name="5-check-the-status"></a>5. Проверьте состояние.

Для удаления всех ресурсов платформе Azure потребуется некоторое время. С помощью приведенного ниже командлета можно проверить состояние группы ресурсов.

```powershell
Get-AzResourceGroup -ResourceGroupName RG1
```

По завершении отобразится результат "Успешно".

```
ResourceGroupName : RG1
Location          : eastus
ProvisioningState : Succeeded
```
