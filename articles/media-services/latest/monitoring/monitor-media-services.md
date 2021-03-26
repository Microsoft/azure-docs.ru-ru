---
title: Мониторинг служб мультимедиа
description: Начните с этого руководства, чтобы узнать, как отслеживать службы мультимедиа
author: IngridAtMicrosoft
ms.author: inhenkel
manager: femilia
ms.topic: how-to
ms.service: media-services
ms.custom: subject-monitoring
ms.date: 03/17/2021
ms.openlocfilehash: 90ca92dc19c588d0b19adf009301cf844e0cdbde
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105609062"
---
# <a name="monitor-media-services"></a>Мониторинг служб мультимедиа

При наличии критически важных приложений и бизнес-процессов, использующих ресурсы Azure, необходимо отслеживать эти ресурсы на предмет их доступности, производительности и работы. В этой статье описываются данные мониторинга, созданные службами мультимедиа, и способы использования функций Azure Monitor для анализа и оповещения об этих данных.

## <a name="metrics-are-useful"></a>Метрики полезны

Ниже приведены примеры того, как мониторинг метрик служб мультимедиа поможет понять, как работают ваши приложения. Ниже приведены некоторые вопросы, которые можно решить с помощью метрик служб мультимедиа.

- Разделы справки отслеживать мои стандартные конечные точки потоковой передачи, чтобы узнать, когда превышены ограничения?
- Разделы справки узнать, достаточно ли единиц масштабирования для конечных точек потоковой передачи уровня "Премиум"?
- Как настроить оповещение о том, когда следует масштабировать конечные точки потоковой передачи?
- Разделы справки настроить оповещение о достижении максимального исходящего исходящего трафика для учетной записи?
- Как увидеть разбивку запросов на ошибки и причину сбоя?
- Как узнать, сколько запросов HLS или ТИРЕ извлекается из упаковщика?
- Разделы справки установить оповещение о том, что достигнуто пороговое значение неудачных запросов?

<!--THIS DOESN'T BELONG HERE Concurrency becomes a concern for the number of Streaming Endpoints used in a single account over time. You need to keep in mind the relationship between the number of concurrent streams with complex publishing parameters like dynamic packaging to multiple protocols, multiple DRM encryptions etc. Each additional published live stream adds to the CPU and output bandwidth on the Streaming Endpoint. With that in mind, you should use Azure Monitor to closely watch the Streaming Endpoint's utilization (CPU and Egress capacity) to make certain that you are scaling it appropriately (or split traffic out between multiple Streaming Endpoints if you are getting into very high concurrency).-->

<!-- Optional diagram showing monitoring for your service. If you need help creating one, contact 
robb@microsoft.com -->

## <a name="what-is-azure-monitor"></a>Общие сведения об Azure Monitor

Службы мультимедиа создают данные мониторинга с помощью [Azure Monitor](../../../azure-monitor/overview.md). это полная служба мониторинга стека в Azure, которая предоставляет полный набор функций для мониторинга ресурсов Azure в дополнение к ресурсам в других облаках и локальных средах.

Начните с чтения статьи [мониторинг ресурсов Azure с помощью Azure Monitor](../../../azure-monitor/essentials/monitor-azure-resource.md), в котором описаны следующие понятия.

- Общие сведения об Azure Monitor
- затраты, связанные с мониторингом;
- данные мониторинга, собираемые в Azure;
- настройка сбора данных;
- стандартные средства Azure для анализа данных мониторинга и оповещения.

## <a name="monitoring-data"></a>Данные мониторинга

Службы мультимедиа собирают те же данные мониторинга, что и другие ресурсы Azure, описанные в разделе [мониторинг данных из ресурсов Azure](../../../azure-monitor/essentials/monitor-azure-resource.md#monitoring-data).

Все данные, собираемые службой Azure Monitor, соответствуют одному из двух основных типов, то есть представляют собой метрики или журналы. С помощью этих двух типов можно:

- Визуализируйте и анализируйте данные метрик с помощью обозревателя метрик.
- Мониторинг журналов диагностики служб мультимедиа и создание оповещений и уведомлений для них.
- С помощью журналов можно:
  - Отправка их в службу хранилища Azure
  - Потоковая передача в концентраторы событий Azure
  - Экспортируйте их в Log Analytics
  - Использование сторонних служб

Подробные сведения о метриках и журналах, созданных службами мультимедиа, см. в статье [мониторинг справочника по данным служб мультимедиа](monitor-media-services-data-reference.md) .

## <a name="collection-and-routing"></a>Сбор и маршрутизация

*Метрики платформы* и *Журнал действий* собираются и сохраняются автоматически, но их можно направить в другие расположения с помощью параметра диагностики.  

*Журналы ресурсов* не собираются и не сохраняются, пока вы не создадите параметр диагностики и **не** направите их в одно или несколько расположений.

Подробный процесс создания параметров диагностики с помощью портал Azure, CLI или PowerShell см. в статье [Создание параметра диагностики для сбора журналов и метрик платформы в Azure](../../../azure-monitor/essentials/diagnostic-settings.md) .

Создавая параметр диагностики, нужно указать, какие категории журналов должны собираться. Категории для служб мультимедиа перечислены в [справочнике по данным мониторинга служб мультимедиа](monitor-media-services-data-reference.md).

## <a name="analyzing-metrics"></a>Анализ метрик

Вы можете анализировать метрики для служб мультимедиа с метриками из других служб Azure, используя обозреватель метрик, открыв **метрики** из меню **Azure Monitor** . Подробные сведения об использовании этого средства см. в статье [Начало работы с обозревателем метрик Azure](../../../azure-monitor/essentials/metrics-getting-started.md).

Список метрик, собранных для служб мультимедиа, см. в разделе [мониторинг справочника по данным служб мультимедиа](monitor-media-services-data-reference.md).

## <a name="analyzing-logs"></a>анализ журналов;

Данные в журналах Azure Monitor хранятся в таблицах, и каждая таблица имеет собственный набор уникальных свойств.  

Все журналы ресурсов в Azure Monitor имеют те же поля, за которыми следуют поля, связанные со службой. Общая схема описана в [Azure Monitor схеме журнала ресурсов](../../../azure-monitor/essentials/resource-logs-schema.md#top-level-common-schema).

Схема журналов ресурсов служб мультимедиа находится в [справочнике по данным служб мультимедиа](monitor-media-services-data-reference.md).

[Журнал действий](../../../azure-monitor/essentials/activity-log.md) — это журнал платформы в Azure, который предоставляет подробные сведения о событиях уровня подписки. Вы можете просмотреть их независимо или направить в журналы Azure Monitor, где можно выполнять гораздо более сложные запросы с помощью Log Analytics.

Список типов журналов ресурсов, собранных для служб мультимедиа, см. в разделе [мониторинг справочника по данным служб мультимедиа](monitor-media-services-data-reference.md).

### <a name="why-would-i-want-to-use-diagnostic-logs"></a>Зачем нужно использовать журналы диагностики?

Ниже перечислены некоторые вещи, которые можно проверить с помощью журналов диагностики.

- Число лицензий, доставляемых типом DRM.
- Число лицензий, доставленных политикой.
- Ошибки по DRM или типу политики.
- Число неавторизованных запросов лицензий от клиентов.

## <a name="alerts"></a>видны узлы

Оповещения Azure Monitor заблаговременно уведомляют вас при обнаружении важных условий в данных мониторинга. Они позволяют выявлять и устранять проблемы в системе до того, как ваши клиенты заметят их. Оповещения можно настроить для [метрик](../../../azure-monitor/alerts/alerts-metric-overview.md), [журналов](../../../azure-monitor/alerts/alerts-unified-log.md) и [журнала действий](../../../azure-monitor/alerts/activity-log-alerts.md).

Метрики служб мультимедиа собираются с регулярными интервалами независимо от того, изменяется ли значение. Они полезны для оповещений, так как их можно часто выдавать. Предупреждение может быть быстро запущено с относительно простой логикой.

<!--
The following table lists common and recommended alert rules for Media Services.

<!-- Fill in the table with metric and log alerts that would be valuable for your service. Change the format as necessary to make it more readable
**PLACEHOLDER** table

| Alert type | Condition | Description  |
|:---|:---|:---|
| | | |
| | | |
-->

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [monitoring-next-steps](../includes/monitoring-next-steps.md)]
