---
title: Просмотр сведений о состоянии и метрик BGP
titleSuffix: Azure VPN Gateway
description: Просмотрите важные сведения, связанные с BGP, для устранения неполадок.
services: vpn-gateway
author: anzaman
ms.service: vpn-gateway
ms.topic: sample
ms.date: 03/10/2021
ms.author: alzam
ms.openlocfilehash: f97bbccc980705699af822ba2730233239cdfd5f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103148941"
---
# <a name="view-bgp-metrics-and-status"></a>Просмотр метрик и состояния BGP

Вы можете изучить метрики и состояние BGP на портале Azure или с помощью Azure PowerShell.

## <a name="azure-portal"></a>Портал Azure

На портале Azure отображаются одноранговые узлы BGP, изученные маршруты и объявленные маршруты. Также здесь можно скачать CSV-файлы с этими данными.

1. На [портале Azure](https://portal.azure.com) перейдите к странице шлюза виртуальной сети.
1. В разделе **Мониторинг** выберите **Одноранговые узлы BGP**, чтобы открыть страницу узлов BGP.

   :::image type="content" source="./media/bgp-diagnostics/bgp-portal.jpg" alt-text="Снимок экрана: метрики на портале Azure.":::

### <a name="learned-routes"></a>Полученные маршруты

1. На портале отображается до 50 полученных маршрутов.

   :::image type="content" source="./media/bgp-diagnostics/learned-routes.jpg" alt-text="Снимок экрана: полученные маршруты.":::

1. Вы также можете скачать файл с полученными маршрутами. Если число изученных маршрутов превышает 50, вы сможете увидеть их только в скачанном CSV-файле. Чтобы скачать его, щелкните **Download learned routes** (Скачать изученные маршруты).

   :::image type="content" source="./media/bgp-diagnostics/download-routes.jpg" alt-text="Снимок экрана: скачивание изученных маршрутов.":::
1. Теперь просмотрите файл.

   :::image type="content" source="./media/bgp-diagnostics/learned-routes-file.jpg" alt-text="Снимок экрана: скачанные изученные маршруты.":::

### <a name="advertised-routes"></a>Объявленные маршруты

1. Чтобы просмотреть объявленные маршруты, щелкните **...** в конце строки нужной сети, а затем щелкните **Просмотреть объявленные маршруты**.

   :::image type="content" source="./media/bgp-diagnostics/select-route.png" alt-text="Снимок экрана: просмотр объявленных маршрутов." lightbox="./media/bgp-diagnostics/route-expand.png":::
1. На странице **Routes advertised to peer** (Маршруты, объявляемые для однорангового узла) отображется до 50 объявленных маршрутов.

   :::image type="content" source="./media/bgp-diagnostics/advertised-routes.jpg" alt-text="Снимок экрана: объявленные маршруты.":::
1. Также можно скачать файл объявленных маршрутов. Если число объявленных маршрутов превышает 50, вы сможете их увидеть только в скачанном CSV-файле. Для скачивания щелкните **Download advertised routes** (Скачать объявленные маршруты).

   :::image type="content" source="./media/bgp-diagnostics/download-advertised.jpg" alt-text="Снимок экрана: выбор скачанных объявленных маршрутов.":::
1. Теперь просмотрите файл.

   :::image type="content" source="./media/bgp-diagnostics/advertised-routes-file.jpg" alt-text="Снимок экрана: скачанные объявленные маршруты.":::

### <a name="bgp-peers"></a>Одноранговые узлы BGP

1. На портале отображается до 50 одноранговых узлов BGP.

   :::image type="content" source="./media/bgp-diagnostics/peers.jpg" alt-text="Снимок экрана: список одноранговых узлов BGP.":::
1. Также можно скачать файл со списком одноранговых узлов BGP. Если число одноранговых узлов BGP превышает 50, вы сможете их увидеть только в скачанном CSV-файле. Чтобы скачать его, на странице портала щелкните **Download BGP peers** (Скачать одноранговые узлы BGP).

   :::image type="content" source="./media/bgp-diagnostics/download-peers.jpg" alt-text="Снимок экрана: скачивание одноранговых узлов BGP.":::
1. Теперь просмотрите файл.

   :::image type="content" source="./media/bgp-diagnostics/peers-file.jpg" alt-text="Снимок экрана: скачанные одноранговые узлы BGP.":::

## <a name="powershell"></a>PowerShell

С помощью **Get-AzVirtualNetworkGatewayBGPPeerStatus** можно просмотреть сведения о состоянии и всех одноранговых узлах BGP.

[!INCLUDE [VPN Gateway PowerShell instructions](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayBgpPeerStatus -ResourceGroupName resourceGroup -VirtualNetworkGatewayName gatewayName

Asn               : 65515
ConnectedDuration : 9.01:04:53.5768637
LocalAddress      : 10.1.0.254
MessagesReceived  : 14893
MessagesSent      : 14900
Neighbor          : 10.0.0.254
RoutesReceived    : 1
State             : Connected
```

С помощью **Get-AzVirtualNetworkGatewayLearnedRoute** можно просмотреть все маршруты, которые шлюз изучил посредством BGP.

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayLearnedRoute -ResourceGroupName resourceGroup -VirtualNetworkGatewayname gatewayName

AsPath       :
LocalAddress : 10.1.0.254
Network      : 10.1.0.0/16
NextHop      :
Origin       : Network
SourcePeer   : 10.1.0.254
Weight       : 32768

AsPath       :
LocalAddress : 10.1.0.254
Network      : 10.0.0.254/32
NextHop      :
Origin       : Network
SourcePeer   : 10.1.0.254
Weight       : 32768

AsPath       : 65515
LocalAddress : 10.1.0.254
Network      : 10.0.0.0/16
NextHop      : 10.0.0.254
Origin       : EBgp
SourcePeer   : 10.0.0.254
Weight       : 32768
```

С помощью **Get-AzVirtualNetworkGatewayAdvertisedRoute** можно просмотреть все маршруты, которые шлюз объявляет одноранговым узлам посредством BGP.

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayAdvertisedRoute -VirtualNetworkGatewayName gatewayName -ResourceGroupName resourceGroupName -Peer 10.0.0.254
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о протоколе BGP см. в статье [Настройка BGP на VPN-шлюзах](bgp-howto.md).
