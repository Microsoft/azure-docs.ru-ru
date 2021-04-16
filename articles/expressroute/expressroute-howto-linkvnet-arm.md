---
title: Руководство по Связывание виртуальной сети с каналом ExpressRoute с помощью Azure PowerShell
description: В этом руководстве содержатся общие сведения о связывании виртуальных сетей с каналами ExpressRoute с помощью модели развертывания Resource Manager и Azure PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/06/2020
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: eda575e0f29abbe18750a24fbfc212a9d84819cf
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106110365"
---
# <a name="tutorial-connect-a-virtual-network-to-an-expressroute-circuit"></a>Руководство по Подключение виртуальной сети к каналу ExpressRoute
> [!div class="op_single_selector"]
> * [Портал Azure](expressroute-howto-linkvnet-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-linkvnet-arm.md)
> * [Azure CLI](howto-linkvnet-cli.md)
> * [Видео — портал Azure](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit)
> * [PowerShell (классическая модель)](expressroute-howto-linkvnet-classic.md)
>

Эта статья поможет вам связать виртуальные сети с каналами Azure ExpressRoute с помощью модели развертывания Resource Manager и PowerShell. Виртуальные сети могут входить в одну и ту же подписку или в разные подписки. В этой статье также показано, как обновить связь виртуальной сети.

* К стандартному каналу ExpressRoute можно подключить не более 10 виртуальных сетей. Если используется стандартный канал ExpressRoute, все виртуальные сети должны находиться в одном геополитическом регионе. 

* Отдельную виртуальную сеть можно связать максимум с 16 каналами ExpressRoute. Чтобы создать объект подключения для каждого канала ExpressRoute, к которому вы подключаетесь, воспользуйтесь инструкциям и из этой статьи. Каналы ExpressRoute могут быть размещены в той же подписке, в других подписках или и там, и там.

* Если вы включите надстройку ExpressRoute Premium, вы сможете подключить к каналу ExpressRoute виртуальные сети из другого геополитического региона. Надстройка Premium также позволяет подключить к каналу ExpressRoute более 10 виртуальных сетей (в зависимости от выбранной пропускной способности). Дополнительную информацию о надстройке Premium см. в разделе [Вопросы и ответы](expressroute-faqs.md).

В этом руководстве описано следующее:
> [!div class="checklist"]
> - Подключение к каналу виртуальной сети в той же подписке
> - Подключение к каналу виртуальной сети в другой подписке
> - Изменение подключения к виртуальной сети
> - Настройка ExpressRoute FastPath

## <a name="prerequisites"></a>Предварительные требования

* Прежде чем приступить к настройке, изучите [предварительные требования](expressroute-prerequisites.md), [требования к маршрутизации](expressroute-routing.md) и [рабочие процессы](expressroute-workflows.md).

* Вам потребуется активный канал ExpressRoute. 
  * Следуйте инструкциям, чтобы [создать канал ExpressRoute](expressroute-howto-circuit-arm.md) и включить его на стороне поставщика услуг подключения. 
  * Убедитесь, что для вашего канала настроен частный пиринг Azure. Инструкции по маршрутизации см. в статье [Настройка маршрутизации](expressroute-howto-routing-arm.md). 
  * Для создания сквозного подключения обязательно настройте частный пиринг Azure, а также пиринг BGP между своей сетью и сетью Майкрософт.
  * Вам необходимо создать и полностью подготовить виртуальную сеть и шлюз виртуальной сети. Следуйте инструкциям по [созданию шлюза виртуальной сети для ExpressRoute](expressroute-howto-add-gateway-resource-manager.md). Для ExpressRoute используется шлюз виртуальной сети типа ExpressRoute, а не VPN.

### <a name="working-with-azure-powershell"></a>Работа с Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

[!INCLUDE [expressroute-cloudshell](../../includes/expressroute-cloudshell-powershell-about.md)]

## <a name="connect-a-virtual-network-in-the-same-subscription-to-a-circuit"></a>Подключение к каналу виртуальной сети в той же подписке
Вы можете связать шлюз виртуальной сети с каналом ExpressRoute, используя следующий командлет. Убедитесь в наличии шлюза виртуальной сети и его готовности к связыванию, прежде чем выполнять командлет.

```azurepowershell-interactive
$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$gw = Get-AzVirtualNetworkGateway -Name "ExpressRouteGw" -ResourceGroupName "MyRG"
$connection = New-AzVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName "MyRG" -Location "East US" -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
```

## <a name="connect-a-virtual-network-in-a-different-subscription-to-a-circuit"></a>Подключение к каналу виртуальной сети в другой подписке
Канал ExpressRoute может совместно использоваться несколькими подписками. На рисунке ниже схематично показан способ совместного использования каналов ExpressRoute несколькими подписками.

Каждое маленькое облако внутри большого облака представляет подписки, принадлежащие различным подразделениям одной организации. Любое подразделение в организации может использовать свою собственную подписку для развертывания служб. Кроме того, подразделения могут совместно использовать один канал ExpressRoute для подключения к локальной сети. Владельцем канала ExpressRoute может выступать одно подразделение (в данном примере — ИТ-подразделение). Другие подписки в организации также могут использовать канал ExpressRoute.

> [!NOTE]
> Плата за подключение канала ExpressRoute и использование полосы пропускания будет взиматься с владельца подписки. Полоса пропускания распределяется между всеми виртуальными сетями.
> 

:::image type="content" source="./media/expressroute-howto-linkvnet-classic/cross-subscription.png" alt-text="Подключение между подписками":::

### <a name="administration---circuit-owners-and-circuit-users"></a>Администрирование: владельцы канала и его пользователи

Владельцем канала является уполномоченный опытный пользователь ресурса канала ExpressRoute. Владелец канала может создавать разрешения, которые могут быть активированы пользователями канала. Пользователи канала являются владельцами шлюзов виртуальных сетей, не включенных в подписку, к которой относится канал ExpressRoute. Пользователи канала могут активировать разрешения (по одному разрешению для каждой виртуальной сети).

Владелец канала имеет право изменить или отменить авторизацию в любое время. Отмена разрешения приводит к удалению всех связывающих подключений из подписки, доступ к которой был отменен.

### <a name="circuit-owner-operations"></a>Действия владельца канала

**Создание разрешения**

Владелец канала создает разрешение, в результате чего создается ключ авторизации, с помощью которого пользователь канала сможет подключить шлюзы виртуальной сети к каналу ExpressRoute. Разрешение действительно только для одного подключения.

В следующем фрагменте показано создание разрешения с помощью командлета.

```azurepowershell-interactive
$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
Add-AzExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "MyAuthorization1"
Set-AzExpressRouteCircuit -ExpressRouteCircuit $circuit

$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$auth1 = Get-AzExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "MyAuthorization1"
```

Ответ на предыдущие команды будет содержать ключ и состояние разрешения:

```azurepowershell
Name                   : MyAuthorization1
Id                     : /subscriptions/&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&/resourceGroups/ERCrossSubTestRG/providers/Microsoft.Network/expressRouteCircuits/CrossSubTest/authorizations/MyAuthorization1
Etag                   : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&& 
AuthorizationKey       : ####################################
AuthorizationUseStatus : Available
ProvisioningState      : Succeeded
```

**Просмотр разрешений**

Владелец канала может просмотреть все разрешения, выданные для определенного канала, выполнив следующий командлет.

```azurepowershell-interactive
$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$authorizations = Get-AzExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit
```

**Добавление разрешений**

Владелец канала может добавлять разрешения с помощью следующего командлета.

```azurepowershell-interactive
$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
Add-AzExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "MyAuthorization2"
Set-AzExpressRouteCircuit -ExpressRouteCircuit $circuit

$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$authorizations = Get-AzExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit
```

**Удаление разрешений**

Владелец канала может отзывать (удалять) разрешения, выданные пользователю, с помощью следующего командлета.

```azurepowershell-interactive
Remove-AzExpressRouteCircuitAuthorization -Name "MyAuthorization2" -ExpressRouteCircuit $circuit
Set-AzExpressRouteCircuit -ExpressRouteCircuit $circuit
```    

### <a name="circuit-user-operations"></a>Действия пользователя канала

Пользователь канала должен получить от владельца канала идентификатор однорангового узла и ключ разрешения. Ключ разрешения представляет собой идентификатор GUID.

Идентификатор однорангового узла можно проверить с помощью следующей команды.

```azurepowershell-interactive
Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
```

**Активация разрешения на подключение**

Пользователь канала может активировать разрешение на связь, выполнив следующий командлет.

```azurepowershell-interactive
$id = "/subscriptions/********************************/resourceGroups/ERCrossSubTestRG/providers/Microsoft.Network/expressRouteCircuits/MyCircuit"    
$gw = Get-AzVirtualNetworkGateway -Name "ExpressRouteGw" -ResourceGroupName "MyRG"
$connection = New-AzVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName "RemoteResourceGroup" -Location "East US" -VirtualNetworkGateway1 $gw -PeerId $id -ConnectionType ExpressRoute -AuthorizationKey "^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^"
```

**Освобождение разрешения на подключение**

Разрешение можно освободить, удалив подключение, связывающее канал ExpressRoute и виртуальную сеть.

## <a name="modify-a-virtual-network-connection"></a>Изменение подключения к виртуальной сети
Вы можете изменить определенные свойства подключения к виртуальной сети. 

**Изменение веса подключения**

Виртуальная сеть может подключаться к нескольким каналам ExpressRoute. Одинаковый префикс может быть получен из нескольких каналов ExpressRoute. Чтобы выбрать подключение для отправки трафика, предназначенного для этого префикса, можно изменить значение *RoutingWeight* подключения. Трафик будет отправляться через подключение с самым высоким значением *RoutingWeight*.

```azurepowershell-interactive
$connection = Get-AzVirtualNetworkGatewayConnection -Name "MyVirtualNetworkConnection" -ResourceGroupName "MyRG"
$connection.RoutingWeight = 100
Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection
```

Диапазон значений *RoutingWeight*: 0 до 32 000. Значение по умолчанию — 0.

## <a name="configure-expressroute-fastpath"></a>Настройка ExpressRoute FastPath 
Вы можете включить [ExpressRoute FastPath](expressroute-about-virtual-network-gateways.md), если используете сверхвысокопроизводительный шлюз или шлюз ErGw3AZ. FastPath повышает производительность передачи данных, то есть такие показатели, как количество пакетов в секунду и подключений в секунду между локальной и виртуальной сетями. 

**Настройка FastPath для нового подключения**

```azurepowershell-interactive 
$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG" 
$gw = Get-AzVirtualNetworkGateway -Name "MyGateway" -ResourceGroupName "MyRG" 
$connection = New-AzVirtualNetworkGatewayConnection -Name "MyConnection" -ResourceGroupName "MyRG" -ExpressRouteGatewayBypass -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute -Location "MyLocation" 
``` 

**Обновление существующего подключения для включения FastPath**

```azurepowershell-interactive 
$connection = Get-AzVirtualNetworkGatewayConnection -Name "MyConnection" -ResourceGroupName "MyRG" 
$connection.ExpressRouteGatewayBypass = $True
Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection
``` 

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вам больше не нужно подключение ExpressRoute, выполните команду `Remove-AzVirtualNetworkGatewayConnection` из подписки, в которой расположен шлюз, чтобы удалить подключение между шлюзом и каналом.

```azurepowershell-interactive
Remove-AzVirtualNetworkGatewayConnection "MyConnection" -ResourceGroupName "MyRG"
```

## <a name="next-steps"></a>Дальнейшие шаги
Дополнительные сведения об ExpressRoute см. в статье Вопросы и ответы по ExpressRoute.

> [!div class="nextstepaction"]
> [Вопросы и ответы по ExpressRoute](expressroute-faqs.md)
