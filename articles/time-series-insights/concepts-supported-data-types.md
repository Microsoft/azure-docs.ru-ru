---
title: Поддерживаемые типы данных — Azure Time Series Insights Gen2 | Документация Майкрософт
description: Узнайте о поддерживаемых типах данных в Azure Time Series Insights Gen2.
author: lyrana
ms.author: lyhughes
manager: deepakpalled
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 01/19/2021
ms.openlocfilehash: b0536a152797d17cba0930b3a142a7fb92eaf5ea
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98685163"
---
# <a name="supported-data-types"></a>Поддерживаемые типы данных

В следующей таблице перечислены типы данных, поддерживаемые службой "аналитика временных рядов Azure" Gen2

| Тип данных | Описание | Пример | [Синтаксис выражений временных рядов](/rest/api/time-series-insights/reference-time-series-expression-syntax) | Имя столбца свойств в Parquet
|---|---|---|---|---|
| **bool**; | Тип данных, имеющий одно из двух состояний: `true` или `false`. | `"isQuestionable" : true` | `$event.isQuestionable.Bool` или `$event['isQuestionable'].Bool` | `isQuestionable_bool`
| **datetime** | Представляет текущее время, обычно выраженное как дата и время суток. Выражено в формате [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html). Свойства DateTime всегда хранятся в формате UTC. Смещения часового пояса, если они правильно отформатированы, будут применены, а затем получится значение, хранящееся в формате UTC. Дополнительные сведения о свойстве timestamp среды и смещениях DateTime см. в [этом](concepts-streaming-ingestion-event-sources.md#event-source-timestamp) разделе. | `"eventProcessedLocalTime": "2020-03-20T09:03:32.8301668Z"` |  Если "Евентпроцесседлокалтиме" является меткой времени источника события: `$event.$ts` . Если это другое свойство JSON: `$event.eventProcessedLocalTime.DateTime` или `$event['eventProcessedLocalTime'].DateTime` | `eventProcessedLocalTime_datetime`
| **double** | 64-разрядное число двойной точности  | `"value": 31.0482941` | `$event.value.Double` или `$event['value'].Double` |  `value_double`
| **long** | 64-разрядное целое число со знаком  | `"value" : 31` | `$event.value.Long` или `$event['value'].Long` |  `value_long`
| **string** | Текстовые значения должны состоять из допустимых UTF-8. Значения NULL и пустые строки считаются одинаковыми. |  `"site": "DIM_MLGGG"`| `$event.site.String` или `$event['site'].String`| `site_string`
| **dynamic** | Сложный (не являющийся примитивом) тип, состоящий из массива или контейнера свойств (Dictionary). Сейчас только переведенные массивы JSON примитивов или массивов объектов, не содержащих идентификатор TS или свойство timestamp, будут храниться как динамические. Ознакомьтесь с этой [статьей](./concepts-json-flattening-escaping-rules.md) , чтобы понять, как объекты будут сведены, а массивы можно будет отменить. Свойства полезных данных, хранящиеся в этом типе, доступны только при выборе `Explore Events` в обозревателе "аналитика временных рядов" для просмотра необработанных событий или через [`GetEvents`](/rest/api/time-series-insights/dataaccessgen2/query/execute#getevents) API запросов для синтаксического анализа на стороне клиента. |  `"values": "[197, 194, 189, 188]"` | Ссылки на динамические типы в выражении временных рядов пока не поддерживаются | `values_dynamic`

> [!NOTE]
> 64 разрядов поддерживаются, но максимальное число, которое обозреватель службы "аналитика временных рядов Azure" может безопасно выразить, равно 9 007 199 254 740 991 (2 ^ 53-1) из-за ограничений JavaScript. При работе с числами в модели данных выше можно уменьшить размер, создав [переменную модели временных рядов](./concepts-variables.md#numeric-variables) и [преобразуя](/rest/api/time-series-insights/reference-time-series-expression-syntax#conversion-functions) значение.

> [!NOTE]
> Тип **строки** не допускает значения NULL:
>
> * [Выражение временного ряда](/rest/api/time-series-insights/reference-time-series-expression-syntax) , выраженное в [запросе временных рядов](/rest/api/time-series-insights/reference-query-apis) , которое сравнивает значение пустой строки (**""**) со **значением NULL** , будет вести себя так: `$event.siteid.String = NULL` эквивалентно `$event.siteid.String = ''` .
> * API может возвращать значения **null** , даже если исходные события содержат пустые строки.
> * Не задавайте зависимости от значений **null** в **строковых** столбцах для сравнения или вычисления, обрабатывают их так же, как пустые строки.

## <a name="sending-mixed-data-types"></a>Отправка смешанных типов данных

Среда Gen2 "аналитика временных рядов Azure" является строго типизированной. Если устройства или теги отправляют данные различных типов для свойства устройства, значения будут храниться в двух отдельных столбцах, а [функция объединения ()](/rest/api/time-series-insights/reference-time-series-expression-syntax#other-functions) должна использоваться при определении выражений переменных временных рядов в вызовах API.

Обозреватель службы "аналитика временных рядов Azure" предлагает способ автоматического объединения отдельных столбцов одного и того же свойства устройства. В приведенном ниже примере датчик отправляет `PresentValue` свойство, которое может быть одновременно длинным или двойным. Чтобы запросить все сохраненные значения (независимо от типа данных) `PresentValue` свойства, выберите, `PresentValue (Double | Long)` и столбцы будут объединены.

[![Автоматическое объединение в обозревателе](media\concepts-supported-data-types/explorer-auto-coalesce-sample.png)](media\concepts-supported-data-types/explorer-auto-coalesce-sample.png#lightbox)

## <a name="objects-and-arrays"></a>Объекты и массивы

В качестве части полезных данных события можно передавать сложные типы, такие как объекты и массивы. Вложенные объекты будут плоскими, а массивы будут храниться как `dynamic` или сведены для создания нескольких событий в зависимости от конфигурации среды и фигуры JSON. Дополнительные сведения о [правилах плоской обработки и экранирования JSON](./concepts-json-flattening-escaping-rules.md)

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь с [правилами преобразования и экранирования JSON](./concepts-json-flattening-escaping-rules.md) , чтобы понять, как будут храниться события.

* Общие сведения об [ограничениях пропускной способности](./concepts-streaming-ingress-throughput-limits.md) среды

* Сведения об [источниках событий](concepts-streaming-ingestion-event-sources.md) для приема потоковых данных.
