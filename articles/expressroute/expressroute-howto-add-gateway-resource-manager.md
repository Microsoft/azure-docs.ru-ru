---
title: 'Руководство. Azure ExpressRoute: добавление шлюза в виртуальную сеть с помощью Azure PowerShell'
description: В этом руководстве показано, как добавить шлюз виртуальной сети в созданную виртуальную сеть Resource Manager для ExpressRoute с помощью Azure PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/05/2020
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 120bfe2eefae3c1721073060231c6c2a1962b7c8
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106110280"
---
# <a name="tutorial-configure-a-virtual-network-gateway-for-expressroute-using-powershell"></a>Руководство по Настройка шлюза виртуальной сети для ExpressRoute с помощью PowerShell
> [!div class="op_single_selector"]
> * [Resource Manager — портал Azure](expressroute-howto-add-gateway-portal-resource-manager.md)
> * [Resource Manager — PowerShell](expressroute-howto-add-gateway-resource-manager.md)
> * [Классическая модель: PowerShell](expressroute-howto-add-gateway-classic.md)
> * [Видео — портал Azure](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-vpn-gateway-for-your-virtual-network)
> 

Из этого руководства вы узнаете, как добавить шлюз виртуальной сети для существующей виртуальной сети, удалить его или изменить его размер. Приведенные действия по настройке применяются к виртуальным сетям, созданным с помощью модели развертывания Resource Manager для конфигурации ExpressRoute. Дополнительные сведения см. в статье [Сведения о шлюзах виртуальных сетей ExpressRoute](expressroute-about-virtual-network-gateways.md).

В этом руководстве описано следующее:
> [!div class="checklist"]
> - Создайте подсеть шлюза.
> - создание шлюза виртуальной сети.

## <a name="prerequisites"></a>Предварительные требования

### <a name="configuration-reference-list"></a>Справочный список для конфигурации

Для выполнения этой задачи используйте виртуальную сеть со значениями, приведенными в справочном списке для настройки ниже. В этом списке также приведены дополнительные параметры и имена. Мы не используем этот список непосредственно ни на каком из шагов, несмотря на то, что добавляем переменные согласно значениям в этом списке. Вы можете скопировать список для справки, заменив значения собственными.

| Параметр                   | Значение                                              |
| ---                       | ---                                                |
| имя виртуальной сети; | *TestVNet* |    
| Диапазон адресов виртуальной сети | *192.168.0.0/16* |
| Группа ресурсов | *TestRG* |
| Имя подсети Subnet1 | *FrontEnd* |   
| Диапазон адресов Subnet1 | *192.168.1.0/24* |
| Имя подсети Subnet1 | *FrontEnd* |
| Имя подсети шлюза | *GatewaySubnet* |    
| Диапазон адресов подсети шлюза | *192.168.200.0/26* |
| Регион | *Восточная часть США* |
| Имя шлюза | *GW* |   
| Имя IP-адреса шлюза | *GWIP* |
| Имя конфигурации IP-адреса шлюза | *gwipconf* |
| Тип | *ExpressRoute* |
| Имя общедоступного IP-адреса шлюза  | *gwpip* |

> [!IMPORTANT]
> Поддержка IPv6 для частного пиринга сейчас находится на этапе **общедоступной предварительной версии**. Если вы хотите подключить виртуальную сеть к каналу ExpressRoute с настроенным частным пирингом на основе IPv6, убедитесь, что ваша виртуальная сеть имеет два стека и соответствует рекомендациям, описанным [здесь](../virtual-network/ipv6-overview.md).
> 
> 

## <a name="add-a-gateway"></a>Добавление шлюза

1. Чтобы подключиться к Azure, выполните `Connect-AzAccount`.

1. Объявите переменные для этого упражнения. Не забудьте изменить пример в соответствии с параметрами, которые вы хотите использовать.

   ```azurepowershell-interactive 
   $RG = "TestRG"
   $Location = "East US"
   $GWName = "GW"
   $GWIPName = "GWIP"
   $GWIPconfName = "gwipconf"
   $VNetName = "TestVNet"
   ```
1. Сохраните объект виртуальной сети как переменную.

   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $RG
   ```
1. Создайте подсеть шлюза своей виртуальной сети. Подсеть шлюза должна иметь имя GatewaySubnet. Подсеть шлюза должна иметь блок адресов /27 или больше (/26, /25 и т. д.). Если вы планируете подключить 16 каналов ExpressRoute к шлюзу, **необходимо** создать подсеть шлюза размером /26 или больше.

   ```azurepowershell-interactive
   Add-AzVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet -AddressPrefix 192.168.200.0/26
   ```
    Если вы используете виртуальную сеть с двумя стеками и планируете использовать частный пиринг на основе IPv6 через ExpressRoute, создайте подсеть шлюза с двумя стеками.

   ```azurepowershell-interactive
   Add-AzVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet -AddressPrefix "10.0.0.0/26","ace:daa:daaa:deaa::/64"
   ```
1. Теперь нужно настроить конфигурацию.

   ```azurepowershell-interactive
   $vnet = Set-AzVirtualNetwork -VirtualNetwork $vnet
   ```
1. Сохраните подсеть шлюза как переменную.

   ```azurepowershell-interactive
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
   ```
1. Запросите общедоступный IP-адрес. IP-адрес запрашивается перед созданием шлюза. Указать необходимый IP-адрес нельзя, он назначается динамически. Этот IP-адрес будет использоваться на следующем этапе конфигурации. Параметр AllocationMethod должен иметь значение Dynamic.

   ```azurepowershell-interactive
   $pip = New-AzPublicIpAddress -Name $GWIPName  -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
   ```
1. Создайте конфигурацию для шлюза. Конфигурация шлюза определяет используемые подсеть и общедоступный IP-адрес. На этом шаге вы задаете конфигурацию, которая будет использоваться при создании шлюза. Используйте следующий пример, чтобы создать конфигурацию шлюза.

   ```azurepowershell-interactive
   $ipconf = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip
   ```
1. Создайте шлюз. На этом шаге особенно важен параметр **-GatewayType** . Необходимо использовать значение **ExpressRoute**. После выполнения этих командлетов на создание шлюза может потребоваться 45 минут или больше.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG -Location $Location -IpConfigurations $ipconf -GatewayType Expressroute -GatewaySku Standard
   ```
> [!IMPORTANT]
> Если вы планируете использовать частный пиринг на основе IPv6 через ExpressRoute, убедитесь, что выбран SKU AZ (ErGw1AZ, ErGw2AZ, ErGw3AZ) для **-GatewaySku**.
> 
> 

## <a name="verify-the-gateway-was-created"></a>Проверка создания шлюза
Используйте следующую команду, чтобы проверить, был ли создан шлюз:

```azurepowershell-interactive
Get-AzVirtualNetworkGateway -ResourceGroupName $RG
```

## <a name="resize-a-gateway"></a>Изменение размера шлюза
Существует несколько [номеров SKU шлюзов](expressroute-about-virtual-network-gateways.md). Чтобы изменить SKU шлюза в любое время, можно использовать следующую команду.

> [!IMPORTANT]
> Эта команда не работает для шлюза UltraPerformance. Чтобы заменить шлюз на UltraPerformance, сначала удалите существующий шлюз ExpressRoute, а затем создайте шлюз UltraPerformance. Чтобы заменить шлюз UltraPerformance обычным шлюзом, сначала удалите шлюз UltraPerformance, а затем создайте новый шлюз.
> 

```azurepowershell-interactive
$gw = Get-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
Resize-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -GatewaySku HighPerformance
```

## <a name="clean-up-resources"></a>Очистка ресурсов
Чтобы удалить шлюз, используйте следующую команду:

```azurepowershell-interactive
Remove-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
```

## <a name="next-steps"></a>Дальнейшие действия
После создания шлюза виртуальной сети вы можете связать виртуальную сеть с каналом ExpressRoute. 

> [!div class="nextstepaction"]
> [Связывание виртуальной сети с каналом ExpressRoute](expressroute-howto-linkvnet-arm.md)
