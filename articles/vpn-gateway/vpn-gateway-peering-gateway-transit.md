---
title: Настройка транзита VPN-шлюзов для пиринга между виртуальными сетями
description: Настройте транзитный шлюз для пиринга виртуальной сети, чтобы легко подключать две виртуальные сети Azure к одной для обеспечения подключения.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 11/30/2020
ms.author: cherylmc
ms.openlocfilehash: 73a7d76de34d29b2d51c54569b234cd8221b08f8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98872185"
---
# <a name="configure-vpn-gateway-transit-for-virtual-network-peering"></a>Настройка транзита VPN-шлюзов для пиринга между виртуальными сетями

С помощью сведений, приведенных в этой статье, вы сможете настроить транзит шлюзов для пиринга между виртуальными сетями. [Пиринг между виртуальными сетями](../virtual-network/virtual-network-peering-overview.md) соединяет две виртуальные сети Azure, объединяя их в одну в целях подключения. [Транзитный шлюз](../virtual-network/virtual-network-peering-overview.md#gateways-and-on-premises-connectivity) — это свойство пиринга, которое позволяет одной виртуальной сети использовать VPN-шлюз в одноранговой виртуальной сети для подключения между организациями или между виртуальными сетями. На следующей схеме показано, как транзит шлюзов работает с пирингом между виртуальными сетями.

![Схема транзитного шлюза](./media/vpn-gateway-peering-gateway-transit/gatewaytransit.png)

На схеме транзит шлюзов позволяет одноранговым виртуальным сетям использовать VPN-шлюз Azure в концентраторе Resource Manager. Подключения к VPN-шлюзу, включая подключения S2S, P2S и "виртуальная сеть — виртуальная сеть", применяются ко всем трем виртуальным сетям. Параметр транзита доступен для пиринга между теми же или разными моделями развертывания. При настройке транзита между различными моделями развертывания виртуальная сеть концентратора и шлюз виртуальной сети должны находиться в модели развертывания диспетчер ресурсов, а не в классической модели развертывания.
>

В сетевой архитектуре типа "звезда" транзит шлюза позволяет периферийным виртуальным сетям совместно использовать VPN-шлюз в концентраторе вместо развертывания VPN-шлюзов в каждой периферийной виртуальной сети. Маршруты к подключенным к шлюзу виртуальным сетям или локальным сетям будут распространяться на таблицы маршрутизации для одноранговых виртуальных сетей, использующих транзит шлюзов. Вы можете отключить автоматическое распространение маршрута из VPN-шлюза. Создайте таблицу маршрутизации с параметром **Отключить распространение маршрутов BGP** и привяжите ее к подсетям, чтобы предотвратить распространение маршрута в этих подсетях. Дополнительные сведения см. в статье [Create, change, or delete a route table](../virtual-network/manage-route-table.md) (Создание, изменение или удаление таблицы маршрутизации).

В этой статье есть два сценария:

* **Та же модель развертывания**. в модели развертывания Диспетчер ресурсов созданы обе виртуальные сети.
* **Различные модели развертывания**. резервная виртуальная сеть создается в классической модели развертывания, а виртуальная сеть и шлюз концентратора находятся в модели развертывания Диспетчер ресурсов.

>[!NOTE]
> Если вы внесли изменения в топологию сети и используете VPN-клиенты Windows, необходимо повторно скачать и установить пакет VPN-клиента для клиентов Windows, чтобы изменения были применены к клиентам.
>

## <a name="prerequisites"></a>Предварительные условия

Перед началом убедитесь, что у вас есть следующие виртуальные сети и разрешения:

### <a name="virtual-networks"></a><a name="vnet"></a>Виртуальные сети

|Виртуальная сеть|Модель развертывания| Шлюз виртуальной сети|
|---|---|---|---|
| Hub-RM| [Resource Manager](./tutorial-site-to-site-portal.md)| [Да](tutorial-create-gateway-portal.md)|
| Лучевой RM | [Resource Manager](./tutorial-site-to-site-portal.md)| Нет |
| Spoke-Classic | [Классический](vpn-gateway-howto-site-to-site-classic-portal.md#CreatVNet) | Нет |

### <a name="permissions"></a><a name="permissions"></a>Permissions

У учетных записей, используемых для создания пиринга между виртуальными сетями, должны быть необходимые роли или разрешения. В приведенном ниже примере, если вы установили пиринг между двумя виртуальными сетями с именем **Hub-RM** и **лучевой-Classic**, ваша учетная запись должна иметь следующие роли или разрешения для каждой виртуальной сети:

|Виртуальная сеть|Модель развертывания|Роль|Разрешения|
|---|---|---|---|
|Hub-RM|Resource Manager|[Участник сети](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)|Microsoft.Network/virtualNetworks/virtualNetworkPeerings/write|
| |Классический|[Участник классической сети](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#classic-network-contributor)|Недоступно|
|Spoke-Classic|Resource Manager|[Участник сети](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)|Microsoft.Network/virtualNetworks/peer|
||Классический|[Участник классической сети](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#classic-network-contributor)|Microsoft.ClassicNetwork/virtualNetworks/peer|

Подробнее о [встроенных ролях](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) и присвоении разрешений, определенных для [настраиваемых ролей](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (только для Resource Manager).

## <a name="same-deployment-model"></a><a name="same"></a>Та же модель развертывания

В этом сценарии виртуальные сети находятся в модели развертывания диспетчер ресурсов. Выполните следующие действия, чтобы создать или обновить пирингы виртуальных сетей, чтобы включить транзит шлюза.

### <a name="to-add-a-peering-and-enable-transit"></a>Добавление пиринга и включение передачи

1. В [портал Azure](https://portal.azure.com)Создайте или обновите пиринг виртуальной сети из центра RM. Перейдите в виртуальную сеть **центра RM** . Выберите **пиринг**, а затем **+ Добавить** , чтобы открыть **Добавить пиринг**.
1. На странице **Добавление пиринга** настройте значения для **этой виртуальной сети**.

   * Имя ссылки пиринга: Назовите ссылку. Пример: **хубрмтоспокерм**
   * Трафик в удаленную виртуальную сеть: **Разрешить**
   * Трафик, пересылаемый из удаленной виртуальной сети: **Разрешить**
   * Шлюз виртуальной сети: **использовать шлюз виртуальной сети**

     :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/peering-vnet.png" alt-text="На снимке экрана показано добавление пиринга.":::

1. На той же странице продолжите настройку значений для **удаленной виртуальной сети**.

   * Имя ссылки пиринга: Назовите ссылку. Пример: **спокермтохубрм**
   * Модель развертывания: **Диспетчер ресурсов**
   * Виртуальная сеть: **лучевой RM**
   * Трафик в удаленную виртуальную сеть: **Разрешить**
   * Трафик, пересылаемый из удаленной виртуальной сети: **Разрешить**
   * Шлюз виртуальной сети: **использование шлюза удаленной виртуальной сети**

     :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/peering-remote.png" alt-text="На снимке экрана показаны значения для удаленной виртуальной сети.":::

1. Нажмите кнопку **Добавить** , чтобы создать пиринг.
1. Проверьте состояние пиринга, **подключенное** в обеих виртуальных сетях.

### <a name="to-modify-an-existing-peering-for-transit"></a>Изменение существующего пиринга для передачи

Если пиринг уже создан, можно изменить пиринг для передачи.

1. Перейдите к виртуальной сети. Выберите **одноранговые узлы** и выберите пиринг, который требуется изменить.

   :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/peering-modify.png" alt-text="На снимке экрана показана выбор одноранговых узлов.":::

1. Обновите пиринг виртуальной сети.

   * Трафик в удаленную виртуальную сеть: **Разрешить**
   * Трафик, перенаправленный в виртуальную сеть; **Разрешить**
   * Шлюз виртуальной сети: **использование шлюза удаленной виртуальной сети**

     :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/modify-peering-settings.png" alt-text="На снимке экрана показано изменение шлюза пиринга.":::

1. **Сохраните** параметры пиринга.

### <a name="powershell-sample"></a><a name="ps-same"></a>Пример для PowerShell

Вы также можете использовать PowerShell для создания или обновления пиринга с помощью приведенного выше примера. Замените переменные именами ваших виртуальных сетей и групп ресурсов.

```azurepowershell-interactive
$SpokeRG = "SpokeRG1"
$SpokeRM = "Spoke-RM"
$HubRG   = "HubRG1"
$HubRM   = "Hub-RM"

$spokermvnet = Get-AzVirtualNetwork -Name $SpokeRM -ResourceGroup $SpokeRG
$hubrmvnet   = Get-AzVirtualNetwork -Name $HubRM -ResourceGroup $HubRG

Add-AzVirtualNetworkPeering `
  -Name SpokeRMtoHubRM `
  -VirtualNetwork $spokermvnet `
  -RemoteVirtualNetworkId $hubrmvnet.Id `
  -UseRemoteGateways

Add-AzVirtualNetworkPeering `
  -Name HubRMToSpokeRM `
  -VirtualNetwork $hubrmvnet `
  -RemoteVirtualNetworkId $spokermvnet.Id `
  -AllowGatewayTransit
```

## <a name="different-deployment-models"></a><a name="different"></a>Различные модели развертывания

В этой конфигурации периферийная виртуальная сеть **спица-классическая** находится в классической модели развертывания, а концентратор виртуальной сети **— RM** — в модели развертывания Диспетчер ресурсов. При настройке транзита между моделями развертывания Шлюз виртуальной сети должен быть настроен для диспетчер ресурсов VNet, а не для классической виртуальной сети.

Для этой конфигурации необходимо настроить только виртуальную сеть **центра RM** . Вам не нужно настраивать что-либо в **периферийной классической** виртуальной сети.

1. В портал Azure перейдите в виртуальную сеть **центра RM** , выберите **узел пиринга**, а затем щелкните **+ добавить**.
1. На странице **Добавление пиринга** настройте следующие значения.

   * Имя ссылки пиринга: Назовите ссылку. Пример: **хубрмтоклассик**
   * Трафик в удаленную виртуальную сеть: **Разрешить**
   * Трафик, пересылаемый из удаленной виртуальной сети: **Разрешить**
   * Шлюз виртуальной сети: **использовать шлюз виртуальной сети**
   * Удаленная виртуальная сеть: **Классическая**

     :::image type="content" source="./media/vpn-gateway-peering-gateway-transit/peering-classic.png" alt-text="Добавление страницы пиринга для лучевой-классической модели":::

1. Убедитесь, что подписка правильная, а затем выберите виртуальную сеть в раскрывающемся списке.
1. Нажмите кнопку **Добавить** , чтобы добавить пиринг.
1. Проверьте состояние пиринга, **подключенное** в виртуальной сети центра RM. 

Для этой конфигурации не нужно настраивать что-либо в виртуальной сети с **периферийным классом** . Когда отобразится состояние **подключено**, периферийная виртуальная сеть может использовать подключение через VPN-шлюз в виртуальной сети концентратора.

### <a name="powershell-sample"></a><a name="ps-different"></a>Пример для PowerShell

Вы также можете использовать PowerShell для создания или обновления пиринга с помощью приведенного выше примера. Замените переменные и идентификатор подписки на значения вашей виртуальной сети, групп ресурсов и подписки. Вам нужно всего лишь создать пиринг между виртуальными сетями в виртуальной сети концентратора.

```azurepowershell-interactive
$HubRG   = "HubRG1"
$HubRM   = "Hub-RM"

$hubrmvnet   = Get-AzVirtualNetwork -Name $HubRM -ResourceGroup $HubRG

Add-AzVirtualNetworkPeering `
  -Name HubRMToClassic `
  -VirtualNetwork $hubrmvnet `
  -RemoteVirtualNetworkId "/subscriptions/<subscription Id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/Spoke-Classic" `
  -AllowGatewayTransit
```

## <a name="next-steps"></a>Дальнейшие действия

* Прежде чем создавать пиринг между виртуальными сетями для рабочей среды, ознакомьтесь с разделами [Requirements and constraints](../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints) (Требования и ограничения) и [Create a peering](../virtual-network/virtual-network-manage-peering.md#create-a-peering) (Создание пиринга).
* Сведения о создании звездообразной топологии сети с пирингом между виртуальными сетями и транзитом шлюзов см. в [этом разделе](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke#virtual-network-peering).
* [Создайте пиринг виртуальных сетей с помощью той же модели развертывания](../virtual-network/tutorial-connect-virtual-networks-portal.md).
* [Создайте пиринг виртуальных сетей с разными моделями развертывания](../virtual-network/create-peering-different-deployment-models.md).