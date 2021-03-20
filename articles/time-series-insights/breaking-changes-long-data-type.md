---
title: Поддержка типа данных Long в службе "аналитика временных рядов Azure" Gen2 | Документация Майкрософт
description: Поддержка типа данных Long в службе "аналитика временных рядов Azure" Gen2.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.topic: conceptual
ms.date: 12/07/2020
ms.custom: dpalled
ms.openlocfilehash: 0f7ac9844bfe7eac7094d3b7fdf653e07f236599
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96780830"
---
# <a name="adding-support-for-long-data-type-in-azure-time-series-insights-gen2"></a>Добавление поддержки для типа данных Long в Azure Time Series Insights Gen2

Добавление поддержки для типа данных Long влияет только на то, как мы сохраняем и индексируем числовые данные в средах "аналитика временных рядов Azure" Gen2. Если у вас есть среда Gen1, эти изменения можно игнорировать.

Начиная с 29 июня или 30 июня 2020 в зависимости от региона, данные будут индексироваться как **длинные** и **Double**.  Если у вас есть вопросы или проблемы, связанные с этим изменением, отправьте запрос в службу поддержки через портал Azure и упоминание об этом взаимодействии.

Если вы затронули один из следующих вариантов, внесите Рекомендуемые изменения:

- **Вариант 1**. в настоящее время вы используете переменные модели временных рядов и отправляете в данные телеметрии только целочисленные типы данных.
- **Вариант 2**. в настоящее время вы используете переменные модели временных рядов и отправляете в данные телеметрии как целочисленные, так и Нецелочисленные типы данных.
- **Вариант 3**. используйте переменные категории, чтобы сопоставлять целочисленные значения с категориями.
- **Случай 4**. для создания пользовательского клиентского приложения используется пакет SDK для JavaScript.
- **Случай 5**. вы приближаетесь к ограничению имени свойства 1 000 в "горячем" хранении и отправляете как целочисленные, так и Нецелочисленные данные. Число свойств можно просмотреть в виде метрики в [портал Azure](https://portal.azure.com/).

Если к вам применим любой из вариантов, внесите изменения в модель. Обновите выражение временных рядов (целевой сервер) в определении переменной с рекомендованными изменениями. Обновите оба:

- Обозреватель службы "Аналитика временных рядов Azure"
- Любой пользовательский клиент, использующий наши API

В зависимости от решения и ограничений Интернета вещей вы можете не видеть данные, которые отправляются в среду Gen2 службы "аналитика временных рядов Azure". Если вы не уверены, являются ли данные целыми или как целыми, так и нецелыми, есть несколько вариантов:

- Можно подождать, пока компонент будет освобожден. Затем изучите необработанные события в пользовательском интерфейсе обозревателя, чтобы понять, какие свойства сохраняются в двух отдельных столбцах.
- Для всех числовых тегов можно использовать более приоритетные изменения.
- Можно временно направить подмножество событий в хранилище, чтобы лучше понять и исследовать схему.

Чтобы сохранить события, включите [запись событий](../event-hubs/event-hubs-capture-overview.md) для концентраторов событий Azure или [отправьте](../iot-hub/iot-hub-devguide-messages-d2c.md#azure-storage-as-a-routing-endpoint) их из центра Интернета вещей в хранилище BLOB-объектов Azure.

Данные также можно наблюдать в [обозревателе концентратора событий](https://marketplace.visualstudio.com/items?itemName=Summer.azure-event-hub-explorer)или с помощью [узла обработчика событий](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md#receive-events).

Если вы используете центр Интернета вещей, перейдите к разделу [Чтение сообщений, отправляемых с устройства в облако, из встроенной конечной точки,](../iot-hub/iot-hub-devguide-messages-read-builtin.md) чтобы получить доступ к встроенной конечной точке.

> [!NOTE]
> Если вы не вносите Рекомендуемые изменения, вы можете столкнуться с перерывом. Например, затронутые переменные "аналитика временных рядов", доступные через API запросов или обозреватель аналитики временных рядов, будут возвращать **значение NULL** (т. е. не показывать данные в обозревателе).

## <a name="recommended-changes"></a>Рекомендованные изменения

### <a name="case-1-using-time-series-model-variables-and-sending-only-integral-data-types-in-telemetry-data"></a>Вариант 1. Использование переменных модели временных рядов и отправка только целочисленных типов данных в данные телеметрии

Рекомендуемые изменения для варианта 1 те же, что и в случае 2. Следуйте инструкциям, приведенным в разделе вариант 2.

### <a name="case-2-using-time-series-model-variables-and-sending-both-integral-and-nonintegral-types-in-telemetry-data"></a>Вариант 2. Использование переменных модели временных рядов и отправка как целочисленных, так и нецелочисленных типов в данных телеметрии

Если в данный момент вы отправляете целочисленные данные телеметрии, данные будут разделены на два столбца:

- **propertyValue_double**
- **propertyValue_long**

Целочисленные данные записываются в **propertyValue_long**. Ранее принятые (и будущие) числовые данные в **propertyValue_double** не копируются.

Если необходимо запросить данные в этих двух столбцах для свойства **propertyvalue** , необходимо использовать скалярную функцию **объединения ()** в целевом серверном компьютере. Функция принимает аргументы одного и того же **типа данных** и возвращает первое значение, отличное от NULL, в списке аргументов. Дополнительные сведения см. в статье службы " [аналитика временных рядов Azure" Gen2 "Основные понятия доступа к данным](/rest/api/time-series-insights/reference-time-series-expression-syntax#other-functions)".

#### <a name="variable-definition-in-tsx---numeric"></a>Определение переменной в ЦЕЛЕВом числе

*Предыдущее определение переменной:*

[![На снимке экрана показано диалоговое окно Добавление новой переменной для переменной PropertyValue numeric.](media/time-series-insights-long-data-type/var-def-previous.png)](media/time-series-insights-long-data-type/var-def-previous.png#lightbox)

*Новое определение переменной:*

[![На снимке экрана показано диалоговое окно Добавление новой переменной для переменной PropertyValue с пользовательским значением numeric.](media/time-series-insights-long-data-type/var-def.png)](media/time-series-insights-long-data-type/var-def.png#lightbox)

В качестве пользовательского [выражения временных рядов](/rest/api/time-series-insights/reference-time-series-expression-syntax)также можно использовать **объединение ($Event. PropertyValue. Double, toDouble ($Event. PropertyValue. Long))** .

#### <a name="inline-variable-definition-using-tsx-query-apis---numeric"></a>Определение встроенных переменных с помощью API целевого запроса — числовые значения

*Предыдущее определение переменной:*

```JSON
"PropertyValueVariable": {

    "kind": "numeric",

    "value": {

        "tsx": "$event.propertyValue.Double"

    },

    "filter": null,

    "aggregation": {

        "tsx": "avg($value)"
    }
}
```

*Новое определение переменной:*

```JSON
"PropertyValueVariable ": {

    "kind": "numeric",

    "value": {

        "tsx": "coalesce($event.propertyValue.Long, toLong($event.propertyValue.Double))"

    },

    "filter": null,

    "aggregation": {

        "tsx": "avg($value)"
    }
}
```

В качестве пользовательского [выражения временных рядов](/rest/api/time-series-insights/reference-time-series-expression-syntax)также можно использовать **объединение ($Event. PropertyValue. Double, toDouble ($Event. PropertyValue. Long))** .

> [!NOTE]
> Рекомендуется обновлять эти переменные во всех местах, где они могут использоваться. К таким местам относятся модель временных рядов, сохраненные запросы и Power BI запросы соединителей.

### <a name="case-3-using-categorical-variables-to-map-integer-values-to-categories"></a>Вариант 3. Использование переменных категории для отображения целочисленных значений по категориям

Если в настоящее время используются переменные категории, которые сопоставляют целочисленные значения с категориями, то, скорее всего, вы используете функцию **толонг** для преобразования данных из типа **Double** в тип **Long** . Как и в случае 1 и 2, необходимо объединить столбцы типа **Double** и **Long**  .

#### <a name="variable-definition-in-time-series-explorer---categorical"></a>Определение переменной в обозревателе временных рядов — Категория

*Предыдущее определение переменной:*

[![На снимке экрана показано диалоговое окно Добавление новой переменной для переменной PropertyValue, Категория.](media/time-series-insights-long-data-type/var-def-cat-previous.png)](media/time-series-insights-long-data-type/var-def-cat-previous.png#lightbox)

*Новое определение переменной:*

[![На снимке экрана показано диалоговое окно Добавление новой переменной для переменной PropertyValue с пользовательским значением по категориям.](media/time-series-insights-long-data-type/var-def-cat.png)](media/time-series-insights-long-data-type/var-def-cat.png#lightbox)

В качестве пользовательского [выражения временных рядов](/rest/api/time-series-insights/preview#time-series-expression-and-syntax)также можно использовать **объединение ($Event. PropertyValue. Double, toDouble ($Event. PropertyValue. Long))** .

Переменная категории по-прежнему требует, чтобы значение было целочисленным типом. Тип **данных** всех аргументов **объединения ()** должен быть типа **Long** в пользовательском [выражении временных рядов.](/rest/api/time-series-insights/reference-time-series-expression-syntax)

#### <a name="inline-variable-definition-using-tsx-query-apis---categorical"></a>Определение встроенных переменных с помощью API целевого запроса — Категория

*Предыдущее определение переменной:*

```JSON
"PropertyValueVariable_Long": {

    "kind": "categorical",

    "value": {

        "tsx": "tolong($event.propertyValue.Double)"

    },

    "categories": [

    {
        "label": "Good",

        "values": [0, 1, 2 ]

    },

    {

        "label": "Bad",

        "values": [ 3, 4 ]

    } ],

    "defaultCategory": {

        "label": "Unknown"

    }
}
```

*Новое определение переменной:*

```JSON
"PropertyValueVariable_Long": {

    "kind": "categorical",

    "value": {

        "tsx": "coalesce($event.propertyValue.Long, tolong($event.propertyValue.Double))"

    },

    "categories": [

    {
        "label": "Good",

        "values": [0, 1, 2 ]

    },

    {

        "label": "Bad",

        "values": [ 3, 4 ]

    } ],

    "defaultCategory": {

        "label": "Unknown"

    }
}
```

Переменная категории по-прежнему требует, чтобы значение было целочисленным типом. Тип **данных** всех аргументов **объединения ()** должен быть типа **Long** в пользовательском [выражении временных рядов](/rest/api/time-series-insights/reference-time-series-expression-syntax).

> [!NOTE]
> Рекомендуется обновлять эти переменные во всех местах, где они могут использоваться. К таким местам относятся модель временных рядов, сохраненные запросы и Power BI запросы соединителей.

### <a name="case-4-using-the-javascript-sdk-to-build-a-custom-front-end-application"></a>Пример 4. Использование пакета SDK для JavaScript для создания пользовательского клиентского приложения

Если вы затронули варианты от 1 до 3 и создаете пользовательские приложения, необходимо обновить запросы для использования функции **объединения ()** , как показано в предыдущих примерах.

### <a name="case-5-nearing-warm-store-1000-property-limit"></a>Вариант 5. ближайшее ограничение на свойство для теплого хранилища 1 000

Если вы являетесь пользователем горячего магазина с большим количеством свойств и считаете, что это изменение приведет к принудительной отправке среды через ограничение имени свойства "горячий магазин 1 000", отправьте запрос в службу поддержки через портал Azure и сообщите об этом взаимодействии.

## <a name="next-steps"></a>Дальнейшие действия

- Просмотрите полный список [поддерживаемых типов данных](concepts-supported-data-types.md).
