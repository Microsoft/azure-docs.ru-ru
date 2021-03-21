---
title: Изменения конечной точки прогнозирования в API V3
description: Интерфейсы API для конечной точки прогнозирования запросов изменились. В этом руководство вы узнаете, как выполнить миграцию на конечные точки API версии 3.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 06/30/2020
ms.openlocfilehash: 59dfa439f6428f2db972a8f848887e1a74bc2622
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98624309"
---
# <a name="prediction-endpoint-changes-for-v3"></a>Изменения конечной точки прогнозирования для v3

Интерфейсы API для конечной точки прогнозирования запросов изменились. В этом руководство вы узнаете, как выполнить миграцию на конечные точки API версии 3.

**Общедоступное состояние** — этот API V3 включает значительные изменения запросов и ответов JSON из API v2.

API V3 предоставляет следующие новые возможности:

* [Внешние сущности](schema-change-prediction-runtime.md#external-entities-passed-in-at-prediction-time)
* [Динамические списки](schema-change-prediction-runtime.md#dynamic-lists-passed-in-at-prediction-time)
* [Изменения предварительно созданных сущностей JSON](#prebuilt-entity-changes)

[Запрос](#request-changes) и [ответ](#response-changes) конечной точки прогнозирования имеют значительные изменения для поддержки новых функций, перечисленных выше, включая следующие:

* [Изменения объекта ответа](#top-level-json-changes)
* [Имя роли сущности ссылки вместо имени сущности](#entity-role-name-instead-of-entity-name)
* [Свойства для пометки сущностей в фразы продолжительностью](#marking-placement-of-entities-in-utterances)

[Справочная документация](https://aka.ms/luis-api-v3) доступна для версии 3.

## <a name="v3-changes-from-preview-to-ga"></a>V3 переход с режима просмотра на общедоступный

В рамках перехода к общедоступному версии 3 были внесены следующие изменения:

* Следующие предварительно созданные сущности имеют разные ответы JSON:
    * [OrdinalV1](luis-reference-prebuilt-ordinal.md)
    * [GeographyV2](luis-reference-prebuilt-geographyv2.md)
    * [DatetimeV2](luis-reference-prebuilt-datetimev2.md)
    * Имя ключа измеряемого блока из `units` в `unit`

* Изменение текста запроса JSON:
    * от `preferExternalEntities` до `preferExternalEntities`
    * Необязательный `score` параметр для внешних сущностей

* Изменения текста ответа JSON:
    * `normalizedQuery` удален

## <a name="suggested-adoption-strategy"></a>Предлагаемая стратегия внедрения

Если вы используете платформу Bot, Проверка орфографии Bing версии 7 или хотите перенести только создание приложения LUIS, продолжайте использовать конечную точку версии 2.

Если вы знаете, что ни одно из клиентских приложений или интеграции (Bot Framework и Проверка орфографии Bing версии 7) не затрагивается, и вы сможете одновременно перенести создание приложений LUIS и конечную точку прогнозирования, начните использовать конечную точку прогнозирования v3. Конечная точка прогнозирования версии 2 по-прежнему будет доступна и является хорошей стратегией отката.


## <a name="not-supported"></a>Не поддерживается

### <a name="bing-spell-check"></a>API Проверки орфографии Bing

Этот API не поддерживается в конечной точке прогнозирования v3. Продолжайте использовать конечную точку прогнозирования API v2 для исправления орфографии. Если при использовании API V3 требуется исправление орфографических ошибок, то клиентское приложение вызывает [Проверка орфографии Bing](../bing-spell-check/overview.md) API и изменяет текст на правильное написание перед отправкой текста в API Luis.

## <a name="bot-framework-and-azure-bot-service-client-applications"></a>Платформа Bot и клиентские приложения службы Azure Bot

Продолжайте использовать конечную точку прогнозирования API v2 до выпуска версии 4.7 платформы Bot.


## <a name="endpoint-url-changes"></a>Изменения URL-адреса конечной точки

### <a name="changes-by-slot-name-and-version-name"></a>Изменения по имени слота и имени версии

Изменился [Формат HTTP-вызова конечной точки версии v3](developer-reference-resource.md#rest-endpoints) .

Если вы хотите выполнить запрос по версии, сначала необходимо [ОПУБЛИКОВАТЬ API](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c3b) с помощью `"directVersionPublish":true` . Запросите конечную точку, ссылающуюся на идентификатор версии, а не на имя слота.

|Допустимые значения для `SLOT-NAME`|
|--|
|`production`|
|`staging`|

## <a name="request-changes"></a>Запрос изменений

### <a name="query-string-changes"></a>Изменения строки запроса

[!INCLUDE [V3 query params](./includes/v3-prediction-query-params.md)]

### <a name="v3-post-body"></a>V3 тело сообщения

```JSON
{
    "query":"your utterance here",
    "options":{
        "datetimeReference": "2019-05-05T12:00:00",
        "preferExternalEntities": true
    },
    "externalEntities":[],
    "dynamicLists":[]
}
```

|Свойство.|Type|Версия|По умолчанию|Назначение|
|--|--|--|--|--|
|`dynamicLists`|array|Только версия 3|Не требуется.|[Динамические списки](schema-change-prediction-runtime.md#dynamic-lists-passed-in-at-prediction-time) позволяют расширить существующую подготовленную и опубликованную сущность списка, уже находящиеся в приложении Luis.|
|`externalEntities`|array|Только версия 3|Не требуется.|[Внешние сущности](schema-change-prediction-runtime.md#external-entities-passed-in-at-prediction-time) предоставляют приложению Luis возможность определять и отмечать сущности во время выполнения, которые можно использовать в качестве функций для существующих сущностей. |
|`options.datetimeReference`|строка|Только версия 3|Нет значения по умолчанию|Используется для определения [смещения datetimeV2](luis-concept-data-alteration.md#change-time-zone-of-prebuilt-datetimev2-entity). Формат для Датетимереференце — [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601).|
|`options.preferExternalEntities`|Логическое|Только версия 3|false|Указывает, используется ли [внешняя сущность пользователя (с тем же именем, что и существующая сущность)](schema-change-prediction-runtime.md#override-existing-model-predictions) , или существующая сущность в модели используется для прогнозирования. |
|`query`|строка|Только версия 3|Обязательный.|**В версии 2** utterance для прогнозирования находится в `q` параметре. <br><br>**В версии 3** функции передаются в `query` параметре.|

## <a name="response-changes"></a>Изменения ответа

Формат JSON ответа на запрос изменен, чтобы обеспечить более частое программный доступ к данным, используемым чаще всего.

### <a name="top-level-json-changes"></a>Изменения верхнего уровня JSON



Верхние свойства JSON для v2 —, если для `verbose` свойства задано значение true, которое возвращает все способы и их оценки в `intents` свойстве:

```JSON
{
    "query":"this is your utterance you want predicted",
    "topScoringIntent":{},
    "intents":[],
    "entities":[],
    "compositeEntities":[]
}
```

Верхние свойства JSON для V3:

```JSON
{
    "query": "this is your utterance you want predicted",
    "prediction":{
        "topIntent": "intent-name-1",
        "intents": {},
        "entities":{}
    }
}
```

`intents`Объект является неупорядоченным списком. Не считайте, что первый дочерний элемент в `intents` соответствует `topIntent` . Вместо этого используйте `topIntent` значение, чтобы найти оценку:

```nodejs
const topIntentName = response.prediction.topIntent;
const score = intents[topIntentName];
```

Изменения схемы JSON ответа позволяют:

* Снимите различие между исходным utterance, `query` и возвращенным прогнозом, `prediction` .
* Упрощенный программный доступ к прогнозируемым данным. Вместо перечисления по массиву в v2 можно получить доступ к значениям по **имени** для целей и сущностей. Для прогнозируемых ролей сущности возвращается имя роли, так как оно уникально для всего приложения.
* Типы данных, если они определены, учитываются. Числовые значения больше не возвращаются в виде строк.
* Различие между данными прогноза первого приоритета и дополнительными метаданными, возвращаемыми в `$instance` объекте.

### <a name="entity-response-changes"></a>Изменения реакции на сущность

#### <a name="marking-placement-of-entities-in-utterances"></a>Маркировка размещения сущностей в фразы продолжительностью

**В версии 2** сущность была помечена в utterance с помощью `startIndex` и `endIndex` .

**В версии 3** сущность помечена `startIndex` и `entityLength` .

#### <a name="access-instance-for-entity-metadata"></a>Доступ `$instance` для метаданных сущности

Если требуются метаданные сущности, то в строке запроса необходимо использовать флаг, `verbose=true` а ответ содержит метаданные в `$instance` объекте. Примеры показаны в ответах JSON в следующих разделах.

#### <a name="each-predicted-entity-is-represented-as-an-array"></a>Каждая прогнозируемая сущность представлена в виде массива

`prediction.entities.<entity-name>`Объект содержит массив, так как каждая сущность может быть предсказана более одного раза в utterance.

<a name="prebuilt-entities-with-new-json"></a>

#### <a name="prebuilt-entity-changes"></a>Предварительно созданные изменения сущностей

Объект отклика v3 включает изменения предварительно созданных сущностей. Чтобы узнать больше, ознакомьтесь с [конкретными предварительно созданными сущностями](luis-reference-prebuilt-entities.md) .

#### <a name="list-entity-prediction-changes"></a>Перечисление изменений прогнозов сущностей

JSON для прогнозирования сущностей списка был изменен на массив массивов:

```JSON
"entities":{
    "my_list_entity":[
        ["canonical-form-1","canonical-form-2"],
        ["canonical-form-2"]
    ]
}
```
Каждый внутренний массив соответствует тексту внутри utterance. Внутренний объект является массивом, поскольку один и тот же текст может присутствовать в нескольких подсписках сущности списка.

При сопоставлении `entities` объекта с `$instance` объектом порядок объектов сохраняется для прогнозов объекта List.

```nodejs
const item = 0; // order preserved, use same enumeration for both
const predictedCanonicalForm = entities.my_list_entity[item];
const associatedMetadata = entities.$instance.my_list_entity[item];
```

#### <a name="entity-role-name-instead-of-entity-name"></a>Имя роли сущности вместо имени сущности

В версии 2 `entities` массив вернул все прогнозируемые сущности с именем сущности, которое является уникальным идентификатором. В версии 3, если сущность использует роли, а прогноз предназначен для роли сущности, первичный идентификатор — это имя роли. Это возможно, поскольку имена ролей сущности должны быть уникальными во всем приложении, включая имена других моделей (намерения, сущности).

В следующем примере: рассмотрим utterance, включающий текст, `Yellow Bird Lane` . Этот текст прогнозируется как `Location` роль пользовательской сущности `Destination` .

|Utterance текст|Имя сущности|Имя роли|
|--|--|--|
|`Yellow Bird Lane`|`Location`|`Destination`|

В версии 2 сущность определяется _именем сущности_ и ролью в качестве свойства объекта:

```JSON
"entities":[
    {
        "entity": "Yellow Bird Lane",
        "type": "Location",
        "startIndex": 13,
        "endIndex": 20,
        "score": 0.786378264,
        "role": "Destination"
    }
]
```

В v3 сущность ссылается _ролью сущности_, если прогноз предназначен для роли:

```JSON
"entities":{
    "Destination":[
        "Yellow Bird Lane"
    ]
}
```

В v3 тот же результат с `verbose` флагом для возврата метаданных сущности:

```JSON
"entities":{
    "Destination":[
        "Yellow Bird Lane"
    ],
    "$instance":{
        "Destination": [
            {
                "role": "Destination",
                "type": "Location",
                "text": "Yellow Bird Lane",
                "startIndex": 25,
                "length":16,
                "score": 0.9837309,
                "modelTypeId": 1,
                "modelType": "Entity Extractor"
            }
        ]
    }
}
```

<a name="external-entities-passed-in-at-prediction-time"></a>
<a name="override-existing-model-predictions"></a>

## <a name="extend-the-app-at-prediction-time"></a>Расширение приложения во время прогнозирования

Ознакомьтесь с [основными понятиями](schema-change-prediction-runtime.md) о том, как расширить приложение в среде выполнения прогнозирования.


## <a name="next-steps"></a>Дальнейшие действия

Используйте документацию по API V3 для обновления существующих вызовов RESTFUL к API-интерфейсам [конечной точки](https://westcentralus.dev.cognitive.microsoft.com/docs/services/luis-endpoint-api-v3-0/operations/5cb0a9459a1fe8fa44c28dd8) Luis.
