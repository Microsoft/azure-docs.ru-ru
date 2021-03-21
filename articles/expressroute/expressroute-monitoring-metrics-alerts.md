---
title: 'Azure ExpressRoute: мониторинг, метрики и оповещения'
description: Сведения о мониторинге, метриках и оповещениях Azure ExpressRoute с помощью Azure Monitor. это средство останавливает магазин для всех метрик, предупреждений, журналов диагностики в Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 01/11/2020
ms.author: duau
ms.openlocfilehash: 7a5da35da35b2f447256bc742681ccd7a7d403da
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99091572"
---
# <a name="expressroute-monitoring-metrics-and-alerts"></a>Мониторинг, метрики и оповещения в ExpressRoute

Эта статья даст вам представление о том, как с помощью Azure Monitor работают мониторинг, метрики и оповещения в ExpressRoute. Azure Monitor— это централизованная служба метрик, оповещений, журналов диагностики для всех продуктов Azure.
 
>[!NOTE]
>Не рекомендуется использовать **классические метрики**.
>

## <a name="expressroute-metrics"></a>Метрики ExpressRoute

Чтобы просмотреть **метрики**, перейдите на страницу *Azure Monitor* и щелкните *метрики*. Чтобы просмотреть метрики **expressroute** , отфильтруйте по типу ресурса *каналы ExpressRoute*. Чтобы просмотреть метрики **Global REACH** , отфильтруйте по типу ресурса *каналы expressroute* и выберите ресурс канала expressroute, в котором включено Global REACH. Чтобы просмотреть метрики **Direct expressroute** , отфильтруйте тип ресурса по *портам ExpressRoute*. 

После выбора метрики будет применено агрегирование по умолчанию. При необходимости можно применить разделение, которое будет показывать метрику с разными измерениями.

### <a name="available-metrics"></a>Доступные показатели

|**Метрика**|**Категория**|**Измерения (s)**|**Функции**|
| --- | --- | --- | --- |
|Доступность ARP|Доступность|<ui><li>Одноранговый (основной или дополнительный маршрутизатор ExpressRoute)</ui></li><ui><li> Тип пиринга (частный, общедоступный/Майкрософт)</ui></li>|ExpressRoute|
|Доступность BGP|Доступность|<ui><li> Одноранговый (основной или дополнительный маршрутизатор ExpressRoute)</ui></li><ui><li> Тип пиринга</ui></li>|ExpressRoute|
|BitsInPerSecond|Трафик|<ui><li> Тип пиринга (ExpressRoute)</ui></li><ui><li>Ссылка (с ExpressRoute Direct)</ui></li>|<li>ExpressRoute</li><li>ExpressRoute Direct</li><ui><li>Подключение к шлюзу ExpressRoute</ui></li>|
|BitsOutPerSecond|Трафик| <ui><li>Тип пиринга (ExpressRoute)</ui></li><ui><li> Ссылка (с ExpressRoute Direct) |<ui><li>ExpressRoute<ui><li>Непосредственный ExpressRoute</ui></li><ui><li>Подключение к шлюзу ExpressRoute</ui></li>|
|Загрузка ЦП|Производительность| <ui><li>Экземпляр</ui></li>|Шлюз виртуальной сети ExpressRoute|
|Пакетов в секунду|Производительность| <ui><li>Экземпляр</ui></li>|Шлюз виртуальной сети ExpressRoute|
|Число маршрутов, объявленных для однорангового узла |Доступность| <ui><li>Экземпляр</ui></li>|Шлюз виртуальной сети ExpressRoute|
|Число маршрутов, полученных от однорангового узла |Доступность| <ui><li>Экземпляр</ui></li>|Шлюз виртуальной сети ExpressRoute|
|Частота изменения маршрутов |Доступность| <ui><li>Экземпляр</ui></li>|Шлюз виртуальной сети ExpressRoute|
|Число виртуальных машин в виртуальной сети |Доступность| Н/Д |Шлюз виртуальной сети ExpressRoute|
|GlobalReachBitsInPerSecond|Трафик|<ui><li>Скэйная цепь (ключ службы)</ui></li>|Global Reach|
|GlobalReachBitsOutPerSecond|Трафик|<ui><li>Скэйная цепь (ключ службы)</ui></li>|Global Reach|
|AdminState|Физическое подключение|Ссылка|ExpressRoute Direct|
|LineProtocol|Физическое подключение|Ссылка|ExpressRoute Direct|
|RxLightLevel|Физическое подключение|<ui><li>Ссылка</ui></li><ui><li>Lane</ui></li>|ExpressRoute Direct|
|TxLightLevel|Физическое подключение|<ui><li>Ссылка</ui></li><ui><li>Lane</ui></li>|ExpressRoute Direct|
>[!NOTE]
>Использование *глобалглобалреачбитсинперсеконд* и *глобалглобалреачбитсаутперсеконд* будет видимым, только если установлено хотя бы одно Global REACH соединение.
>

## <a name="circuits-metrics"></a>Метрики каналов

### <a name="bits-in-and-out---metrics-across-all-peerings"></a>Входные и выходные метрики во всех пиринга

Вы можете просматривать метрики для всех узлов в заданном канале ExpressRoute.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/ermetricspeering.jpg" alt-text="метрики канала":::

### <a name="bits-in-and-out---metrics-per-peering"></a>Количество входных и исходящих битов в метрики на пиринг

Доступны метрики по частному и общедоступному пирингу, а также пирингу Майкрософт в бит/с.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/erpeeringmetrics.jpg" alt-text="метрики на пиринг":::

### <a name="bgp-availability---split-by-peer"></a>Доступность BGP — разделение по узлу  

Вы можете просмотреть сведения о доступности BGP в режиме реального времени для пиринга и одноранговых узлов (основной и дополнительный маршрутизаторы ExpressRoute). На этой панели мониторинга показан основной сеанс BGP для частного пиринга и второй сеанс BGP для частного пиринга. 

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/erBgpAvailabilityMetrics.jpg" alt-text="Доступность BGP на узел":::

### <a name="arp-availability---split-by-peering"></a>Доступность ARP — разделение по пиринга  

Вы можете просмотреть сведения о доступности [ARP](./expressroute-troubleshooting-arp-resource-manager.md) в режиме реального времени для пиринга и одноранговых узлов (основной и дополнительный маршрутизаторы ExpressRoute). На этой панели мониторинга показан сеанс ARP частного пиринга по обоим одноранговым узлам, но для пиринга Майкрософт между одноранговыми узлами завершается. По умолчанию для обоих узлов использовалась статистическая обработка (среднее значение).  

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/erArpAvailabilityMetrics.jpg" alt-text="Доступность ARP на одноранговый узел":::

## <a name="expressroute-direct-metrics"></a>Непосредственные метрики ExpressRoute

### <a name="admin-state---split-by-link"></a>Состояние администратора — разбиение по каналу

Вы можете просмотреть состояние администратора для каждой ссылки на пару прямых портов ExpressRoute.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/adminstate-per-link.jpg" alt-text="Состояние прямого администратора ER":::

### <a name="bits-in-per-second---split-by-link"></a>Число битов в секунду — разбиение по каналу

Можно просмотреть биты в секунду для обеих ссылок в паре прямых портов ExpressRoute.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/bits-in-per-second-per-link.jpg" alt-text="Число прямых бит в секунду для ER":::

### <a name="bits-out-per-second---split-by-link"></a>Биты за секунду — разбиение по каналу

Кроме того, можно просмотреть биты в секунду для обеих ссылок в паре прямых портов ExpressRoute.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/bits-out-per-second-per-link.jpg" alt-text="Непосредственных бит ER в секунду":::

### <a name="line-protocol---split-by-link"></a>Протокол строки — разбиение по каналу

Протокол Line можно просмотреть по каждой ссылке на пару прямых портов ExpressRoute.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/line-protocol-per-link.jpg" alt-text="Протокол прямой строки ER":::

### <a name="rx-light-level---split-by-link"></a>Уровень источника RX — разбиение по каналу

Для каждого порта можно просмотреть уровень падения на RX (уровень освещения, который **получает** прямой порт ExpressRoute). Работоспособные уровни освещения обычно находятся в диапазоне от-10 до 0 dBm.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/rxlight-level-per-link.jpg" alt-text="Непосредственный уровень RX прямой строки ER":::

### <a name="tx-light-level---split-by-link"></a>Уровень источника TX — разбиение по каналу

Для каждого порта можно просмотреть уровень источника TX (уровень падения, который **передает** прямой порт ExpressRoute). Работоспособные уровни интенсивности TX обычно находятся в диапазоне от-10 до 0 dBm.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/txlight-level-per-link.jpg" alt-text="Неяркий уровень TX прямой строки ER":::

## <a name="expressroute-virtual-network-gateway-metrics"></a>Метрики шлюза виртуальной сети ExpressRoute

### <a name="cpu-utilization---split-instance"></a>Использование ЦП — разделение экземпляра

Можно просмотреть использование ЦП экземплярами шлюза.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/cpu-split.jpg" alt-text="Разделение ЦП":::

### <a name="packets-per-second---split-by-instance"></a>Пакетов в секунду — разделение по экземпляру

Можно просмотреть пакеты в секунду, проходящие через шлюз.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/pps-split.jpg" alt-text="Пакетов в секунду — разделение":::

### <a name="count-of-routes-advertised-to-peer---split-by-instance"></a>Число маршрутов, объявленных для однорангового разделения по экземпляру

Можно просмотреть число маршрутов, объявленных для канала ExpressRoute.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/count-of-routes-advertised-to-peer.png" alt-text="Число маршрутов, объявленных для однорангового узла":::

### <a name="count-of-routes-learned-from-peer---split-by-instance"></a>Число маршрутов, полученных от однорангового разделения по экземпляру

Можно просмотреть число маршрутов, полученных из канала ExpressRoute.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/count-of-routes-learned-from-peer.png" alt-text="Число маршрутов, полученных от однорангового узла":::

### <a name="frequency-of-routes-change---split-by-instance"></a>Частота изменения маршрутов — разбиение по экземпляру

Можно просмотреть частоту изменения маршрута на шлюзе.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/frequency-of-routes-changed.png" alt-text="Частота изменения маршрутов":::

### <a name="number-of-vms-in-the-virtual-network"></a>Число виртуальных машин в виртуальной сети

Вы можете просмотреть число виртуальных машин в виртуальной сети.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/number-of-virtual-machines-virtual-network.png" alt-text="Число виртуальных машин в виртуальной сети":::

## <a name="expressroute-gateway-connections-in-bitsseconds"></a>Подключения шлюза ExpressRoute в бит/с

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/erconnections.jpg" alt-text="подключения шлюза":::

## <a name="alerts-for-expressroute-gateway-connections"></a>Оповещения для подключений шлюза ExpressRoute

1. Чтобы настроить оповещения, перейдите в раздел **Azure Monitor**, а затем выберите **оповещения**.

   :::image type="content" source="./media/expressroute-monitoring-metrics-alerts/eralertshowto.jpg" alt-text="Предупреждения":::
2. Щелкните **+Select Target** (Выбрать целевой объект) и выберите ресурс подключения шлюза ExpressRoute.

   :::image type="content" source="./media/expressroute-monitoring-metrics-alerts/alerthowto2.jpg" alt-text="target":::
3. Указание сведений для оповещения.

   :::image type="content" source="./media/expressroute-monitoring-metrics-alerts/alerthowto3.jpg" alt-text="Группа действий":::
4. Указание сведений для группы действий, добавление группы действий.

   :::image type="content" source="./media/expressroute-monitoring-metrics-alerts/actiongroup.png" alt-text="Добавить группу действий":::

## <a name="alerts-based-on-each-peering"></a>Оповещения для каждого пиринга

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/basedpeering.jpg" alt-text="Каждый пиринг":::

## <a name="configure-alerts-for-activity-logs-on-circuits"></a>Настройка оповещений для журналов действий каналов

В раскрывающемся списке **Критерии оповещения** можно выбрать **Журнал действий** для определенного типа сигнала и указать сигнал.

:::image type="content" source="./media/expressroute-monitoring-metrics-alerts/alertshowto6activitylog.jpg" alt-text="журналы действий":::

## <a name="additional-metrics-in-log-analytics"></a>Дополнительные метрики в Log Analytics

Метрики ExpressRoute также можно просмотреть, перейдя к ресурсу канала ExpressRoute и выбрав вкладку *журналы* . Для всех метрик, которые вы запрашиваете, выходные данные будут содержать следующие столбцы.

|**Столбец**|**Тип**|**Описание**|
| --- | --- | --- |
|TimeGrain|строка|PT1M (значения метрик отправляются каждую минуту)|
|Count|real|Обычно равно 2 (каждый MSEE отправляет одно значение метрики каждую минуту).|
|Минимальные|real|Минимальное из двух значений метрик, помещаемых двумя MSEE|
|Максимум|real|Максимальное из двух значений метрик, помещаемых двумя MSEE|
|Среднее значение|real|Равно (минимум + максимум)/2|
|Итог|real|Сумма двух значений метрики из обоих MSEE (основное значение, на которое нужно сосредоточиться для запрошенной метрики)|
  
## <a name="next-steps"></a>Дальнейшие действия

Настройте подключение ExpressRoute.
  
* [Создание и изменение канала](expressroute-howto-circuit-arm.md)
* [Создание и изменение конфигурации пиринга](expressroute-howto-routing-arm.md)
* [Связывание виртуальной сети с каналом ExpressRoute](expressroute-howto-linkvnet-arm.md)
