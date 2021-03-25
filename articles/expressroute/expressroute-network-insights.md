---
title: Azure ExpressRoute Insights с помощью Network Insights
description: Сведения об Azure ExpressRoute Insights с помощью Network Insights.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 03/23/2021
ms.author: duau
ms.openlocfilehash: 7033ea6a1ba6d85f9aa15e14bb9577b2439c59a8
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105050569"
---
# <a name="azure-expressroute-insights-using-network-insights"></a>Azure ExpressRoute Insights с помощью Network Insights

В этой статье объясняется, как с помощью Network Insights можно просматривать метрики и конфигурации ExpressRoute в одном месте. С помощью Network Insights вы можете просматривать карты топологическом и панели мониторинга работоспособности, содержащие важные сведения ExpressRoute, без необходимости выполнения дополнительных настроек.

:::image type="content" source="./media/expressroute-network-insights/monitor-landing-page.png" alt-text="Снимок экрана: Целевая страница монитора ExpressRoute." lightbox="./media/expressroute-network-insights/monitor-landing-page-expanded.png":::

## <a name="visualize-functional-dependencies"></a>Визуализация функциональных зависимостей

Чтобы просмотреть это решение, перейдите на страницу *Azure Monitor* , выберите *сети*, а затем выберите плату *каналов ExpressRoute* . Затем нажмите кнопку Topology (топология) для канала, который вы хотите просмотреть.

Представление функциональной зависимости представляет собой четкое изображение настройки ExpressRoute, описывающее связь между различными компонентами ExpressRoute (пирингами, подключениями, шлюзами).

:::image type="content" source="./media/expressroute-network-insights/topology-view.png" alt-text="Снимок экрана представления топологии для анализа сети." lightbox="./media/expressroute-network-insights/topology-view-expanded.png":::

Наведите указатель мыши на любой компонент на карте топологии, чтобы просмотреть сведения о конфигурации. Например, наведите указатель мыши на компонент пиринга ExpressRoute, чтобы просмотреть такие сведения, как пропускная способность канала и включение Global Reach.

:::image type="content" source="./media/expressroute-network-insights/topology-hovered.png" alt-text="Снимок экрана: Просмотр ресурсов при наведении на топологию." lightbox="./media/expressroute-network-insights/topology-hovered-expanded.png":::

## <a name="view-a-detailed-and-pre-loaded-metrics-dashboard"></a>Просмотр подробной и предварительно загруженной панели мониторинга метрик

Просматривая топологию установки ExpressRoute с помощью представления функциональная зависимость, выберите **Просмотреть подробные метрики** , чтобы перейти к подробному представлению метрик, чтобы понять производительность канала. В этом представлении представлен упорядоченный список связанных ресурсов и многофункциональной панели мониторинга важных метрик ExpressRoute.

В разделе **связанные ресурсы** перечислены подключенные шлюзы ExpressRoute и настроенные пиринга, которые можно выбрать для перехода к соответствующей странице ресурсов.

:::image type="content" source="./media/expressroute-network-insights/linked-resources.png" alt-text="Снимок экрана: связанный ресурс на странице монитора.":::


В разделе **метрики ExpressRoute** содержатся диаграммы важных метрик канала по категориям **доступность**, **пропускная способность**, удаление **пакетов** и **метрики шлюза**.

### <a name="availability"></a>Доступность

На вкладке *доступность* отслеживаются сведения о доступности ARP и BGP, в которых отображаются данные для канала как в целом, так и в отдельном соединении (основной и дополнительный). 

:::image type="content" source="./media/expressroute-network-insights/arp-bgp-availability.png" alt-text="Снимок экрана со графами метрик доступности." lightbox="./media/expressroute-network-insights/arp-bgp-availability-expanded.png":::

### <a name="throughput"></a>Пропускная способность

Аналогичным образом вкладка *пропускная способность* отображает общую пропускную способность входящего и исходящего трафика для канала в битах/сек. Можно также просмотреть пропускную способность для отдельных подключений и для каждого типа настроенного пиринга.

:::image type="content" source="./media/expressroute-network-insights/throughput.png" alt-text="Снимок экрана с диаграммами метрик пропускной способности." lightbox="./media/expressroute-network-insights/throughput-expanded.png":::

### <a name="packet-drops"></a>Пропуски пакетов

Вкладка *пакет удаляет* отброшенные биты/секунды для входящего и исходящего трафика через канал. На этой вкладке можно легко отслеживать проблемы с производительностью, которые могут возникнуть, если вам приходится постоянно требовать или превысить пропускную способность канала.

:::image type="content" source="./media/expressroute-network-insights/dropped-packets.png" alt-text="Снимок экрана с пропущенными графами пакетов." lightbox="./media/expressroute-network-insights/dropped-packets-expanded.png":::

### <a name="gateway-metrics"></a>Метрики шлюзов

Наконец, вкладка метрики шлюза заполняется диаграммами ключевых метрик для выбранного шлюза ExpressRoute (из раздела связанные ресурсы). Эта вкладка используется, когда необходимо отслеживать подключение к конкретным виртуальным сетям.

:::image type="content" source="./media/expressroute-network-insights/gateway-metrics.png" alt-text="Снимок экрана: пропускная способность и метрики ЦП шлюза." lightbox="./media/expressroute-network-insights/gateway-metrics-expanded.png":::

## <a name="next-steps"></a>Дальнейшие действия

Настройте подключение ExpressRoute.
  
* Дополнительные сведения о [Azure ExpressRoute](expressroute-introduction.md), [Network Insights](../azure-monitor/insights/network-insights-overview.md)и [наблюдателе за сетями](../network-watcher/network-watcher-monitoring-overview.md)
* [Создание и изменение канала](expressroute-howto-circuit-arm.md)
* [Создание и изменение конфигурации пиринга](expressroute-howto-routing-arm.md)
* [Связывание виртуальной сети с каналом ExpressRoute](expressroute-howto-linkvnet-arm.md)
* [Настройка метрик](expressroute-monitoring-metrics-alerts.md) и создание [монитора подключения](../network-watcher/connection-monitor-overview.md)
