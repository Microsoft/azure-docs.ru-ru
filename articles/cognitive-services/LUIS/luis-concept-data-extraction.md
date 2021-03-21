---
title: Извлечение данных — LUIS
description: Извлечение данных из utterance текста с помощью целей и сущностей. Сведения о том, какие типы данных можно извлечь из Распознавание речи (LUIS).
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 05/01/2020
ms.openlocfilehash: e6f01354bb5aa2b78d3c9962bac49be39dd2c81f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95025999"
---
# <a name="extract-data-from-utterance-text-with-intents-and-entities"></a>Извлечение данных из текста речевого фрагмента с помощью намерений и сущностей
Служба LUIS дает возможность получать информацию из высказываний на естественном языке пользователя. Информация извлекается таким образом, который позволяет использовать ее программой, приложением или чат-ботом для выполнения определенных действий. В следующих разделах содержатся сведения о данных, возвращаемых из намерений и сущностей, с примерами JSON.

Самые сложные данные для извлечения — это данные машинного обучения, так как это не точное совпадение текста. Извлечение данных [сущностей](luis-concept-entity-types.md) машинного обучения должно быть частью [цикла разработки](luis-concept-app-iteration.md) до тех пор, пока вы не будете уверены, что будете получать нужные данные.

## <a name="data-location-and-key-usage"></a>Расположение данных и использование ключа
LUIS извлекает данные из utterance пользователя на опубликованной [конечной точке](luis-glossary.md#endpoint). **Запрос HTTPS** (POST или GET) содержит высказывание, а также некоторые дополнительные конфигурации, например промежуточные или рабочие среды.

**Запрос конечной точки прогнозирования V2**

`https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/<appID>?subscription-key=<subscription-key>&verbose=true&timezoneOffset=0&q=book 2 tickets to paris`

**Запрос конечной точки прогнозирования V3**

`https://westus.api.cognitive.microsoft.com/luis/v3.0-preview/apps/<appID>/slots/<slot-type>/predict?subscription-key=<subscription-key>&verbose=true&timezoneOffset=0&query=book 2 tickets to paris`

`appID` доступен на странице **Параметры** приложения LUIS, а также в составе URL-адреса (после `/apps/`) при редактировании этого приложения LUIS. `subscription-key` — это ключ конечной точки, используемый для запросов приложения. При использовании бесплатного ключа разработки или начального ключа во время обучения LUIS необходимо изменить ключ конечной точки на ключ, который поддерживает [ожидаемое использование LUIS](luis-limits.md#key-limits). Единица `timezoneOffset` в минутах.

**Ответ HTTPS** содержит все сведения о намерении и сущности, которые служба LUIS может определять на основе текущей опубликованной модели промежуточной или рабочей конечной точки. URL-адрес конечной точки указан на веб-сайте [LUIS](luis-reference-regions.md), раздел **Управление**, страница **Ключи и конечные точки**.

## <a name="data-from-intents"></a>Данные из намерений
Основными данными является **имя намерения** с самой высокой оценкой. Ответ конечной точки:

#### <a name="v2-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V2](#tab/V2)

```JSON
{
  "query": "when do you open next?",
  "topScoringIntent": {
    "intent": "GetStoreInfo",
    "score": 0.984749258
  },
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V3](#tab/V3)

```JSON
{
  "query": "when do you open next?",
  "prediction": {
    "normalizedQuery": "when do you open next?",
    "topIntent": "GetStoreInfo",
    "intents": {
        "GetStoreInfo": {
            "score": 0.984749258
        }
    }
  },
  "entities": []
}
```

Дополнительные сведения о [конечной точке прогнозирования V3](luis-migration-api-v3.md).

* * *

|Объект данных|Тип данных|Расположение данных|Значение|
|--|--|--|--|
|Блокировка с намерением|Строка|topScoringIntent.intent|"GetStoreInfo"|

Если приложение, вызывающее чат-бот или LUIS, принимает решение на основе более чем одной оценки намерения, возвратите все оценки.


#### <a name="v2-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V2](#tab/V2)

Задайте параметр QueryString, `verbose=true` . Ответ конечной точки:

```JSON
{
  "query": "when do you open next?",
  "topScoringIntent": {
    "intent": "GetStoreInfo",
    "score": 0.984749258
  },
  "intents": [
    {
      "intent": "GetStoreInfo",
      "score": 0.984749258
    },
    {
      "intent": "None",
      "score": 0.2040639
    }
  ],
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V3](#tab/V3)

Задайте параметр QueryString, `show-all-intents=true` . Ответ конечной точки:

```JSON
{
    "query": "when do you open next?",
    "prediction": {
        "normalizedQuery": "when do you open next?",
        "topIntent": "GetStoreInfo",
        "intents": {
            "GetStoreInfo": {
                "score": 0.984749258
            },
            "None": {
                 "score": 0.2040639
            }
        },
        "entities": {
        }
    }
}
```

Дополнительные сведения о [конечной точке прогнозирования V3](luis-migration-api-v3.md).

* * *

Намерения упорядочены от имеющего самую высокую оценку до имеющего самую низкую оценку.

|Объект данных|Тип данных|Расположение данных|Значение|Оценка|
|--|--|--|--|:--|
|Блокировка с намерением|Строка|intents[0].intent|"GetStoreInfo"|0,984749258|
|Блокировка с намерением|Строка|intents[1].intent|"None"|0,0168218873|

При добавлении готовых предметных областей имя намерения указывает предметную область, такую как `Utilties` или `Communication`, а также намерение:

#### <a name="v2-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V2](#tab/V2)

```JSON
{
  "query": "Turn on the lights next monday at 9am",
  "topScoringIntent": {
    "intent": "Utilities.ShowNext",
    "score": 0.07842206
  },
  "intents": [
    {
      "intent": "Utilities.ShowNext",
      "score": 0.07842206
    },
    {
      "intent": "Communication.StartOver",
      "score": 0.0239675418
    },
    {
      "intent": "None",
      "score": 0.0168218873
    }],
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V3](#tab/V3)

```JSON
{
    "query": "Turn on the lights next monday at 9am",
    "prediction": {
        "normalizedQuery": "Turn on the lights next monday at 9am",
        "topIntent": "Utilities.ShowNext",
        "intents": {
            "Utilities.ShowNext": {
                "score": 0.07842206
            },
            "Communication.StartOver": {
                "score": 0.0239675418
            },
            "None": {
                "score": 0.00085447653
            }
        },
        "entities": []
    }
}
```

Дополнительные сведения о [конечной точке прогнозирования V3](luis-migration-api-v3.md).

* * *

|Домен|Объект данных|Тип данных|Расположение данных|Значение|
|--|--|--|--|--|
|Служебные программы|Блокировка с намерением|Строка|intents[0].intent|"<b>Utilities</b>.ShowNext"|
|Связь|Блокировка с намерением|Строка|intents[1].intent|<b>Communication</b>.StartOver"|
||Блокировка с намерением|Строка|intents[2].intent|"None"|


## <a name="data-from-entities"></a>Данные из сущностей
Большинству программы-роботы и приложениям разговора требуется больше имен намерения. Эти дополнительные данные поступают из сущностей, обнаруживаемых в высказывании. Каждый тип сущности возвращает различные сведения о совпадении.

Одно слово или фраза в высказывании может соответствовать нескольким сущностям. В этом случае каждая совпадающая сущность возвращается с оценкой.

Все сущности возвращаются в массиве **сущностей** ответа от конечной точки.

## <a name="tokenized-entity-returned"></a>Возвращенная размеченная сущность

Ознакомьтесь с [поддержкой маркеров](luis-language-support.md#tokenization) в Luis.


## <a name="prebuilt-entity-data"></a>Данные предварительно созданной сущности
Обнаружение [предварительно созданных](luis-concept-entity-types.md) сущностей выполняется на основе совпадения регулярного выражения с помощью с проекта [Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) с открытым исходным кодом. Предварительно созданные сущности возвращаются в массиве сущностей и используют имя типа с префиксом `builtin::`.

## <a name="list-entity-data"></a>Данные сущности списка

[Список сущностей](reference-entity-list.md) представляет собой фиксированный, закрытый набор связанных слов вместе с их синонимами. LUIS не обнаруживает дополнительные значения для сущностей списка. Посмотреть предложения для новых слов на основе текущего списка можно с помощью функции **Рекомендовать**. Если имеется несколько сущностей списка с одним и тем же значением, в запросе конечной точки возвращается каждая сущность.

## <a name="regular-expression-entity-data"></a>Данные сущности регулярного выражения

[Сущность регулярного выражения](reference-entity-regular-expression.md) извлекает сущность на основе предоставленного регулярного выражения.

## <a name="extracting-names"></a>Извлечение имен
Получение имен из высказывания является сложной задачей, имя может быть практически любым сочетанием букв и слов. В зависимости от того, какой тип имени извлекается, вы можете использовать один из подходящих вариантов. Следующие предложения являются не правилами, а дополнительными рекомендациями.

### <a name="add-prebuilt-personname-and-geographyv2-entities"></a>Добавление предварительно созданных сущностей PersonName и GeographyV2

Сущности [PersonName](luis-reference-prebuilt-person.md) и [GeographyV2](luis-reference-prebuilt-geographyV2.md) доступны для некоторых [языков и региональных параметров](luis-reference-prebuilt-entities.md).

### <a name="names-of-people"></a>Имена людей

В зависимости от языка и региональных параметров формат имен людей может незначительно отличаться. Используйте предварительно созданную сущность **[персоннаме](luis-reference-prebuilt-person.md)** или **[простую сущность](luis-concept-entity-types.md)** с ролями с именами и фамилиями.

Если вы используете простую сущность, не забудьте предоставить примеры, в которых используются имя и фамилия в различных частях utterance, в фразы продолжительностью с различной длиной и фразы продолжительностью по всем смыслу, включая намерение None. Регулярно [просматривайте](./luis-how-to-review-endpoint-utterances.md) высказывания конечной точки, чтобы помечать имена, которые были неправильно спрогнозированы.

### <a name="names-of-places"></a>Имена мест

Имена расположений задаются и называются городами, подсчетами, Штатами, провинции, странами и регионами. Используйте предварительно созданную сущность **[geographyV2](luis-reference-prebuilt-geographyv2.md)** для извлечения сведений о расположении.

### <a name="new-and-emerging-names"></a>Новые имена

Некоторые приложения должны иметь возможность поиска новых имен, например имен продуктов и компаний. Эти типы имен являются наиболее сложным типом извлечения данных. Начните с **[простой сущности](luis-concept-entity-types.md#simple-entity)** и добавьте [список фраз](luis-concept-feature.md). Регулярно [просматривайте](./luis-how-to-review-endpoint-utterances.md) высказывания конечной точки, чтобы помечать имена, которые были неправильно спрогнозированы.

## <a name="patternany-entity-data"></a>Данные сущности Pattern.any

[Pattern. Any](reference-entity-pattern-any.md) — это заполнитель переменной длины, используемый только в шаблоне шаблона utterance для обозначения места начала и окончания сущности. Чтобы применить шаблон, необходимо найти сущность, используемую в шаблоне.

## <a name="sentiment-analysis"></a>Анализ мнений
Если во время [публикации](luis-how-to-publish-app.md#sentiment-analysis)настраивается анализ тональности, ответ JSON Luis включает в себя анализ тональности. Дополнительные сведения об анализе тональности см. в документации по [анализу текста](../text-analytics/index.yml).

## <a name="key-phrase-extraction-entity-data"></a>Данные сущности извлечения ключевой фразы
[Сущность извлечения ключевых фраз](luis-reference-prebuilt-keyphrase.md) возвращает ключевые фразы в utterance, предоставляемые [анализ текста](../text-analytics/index.yml).

## <a name="data-matching-multiple-entities"></a>Данные, соответствующие нескольким сущностям

LUIS возвращает все сущности, обнаруженные в высказывании. В результате для программы-робота Chat может потребоваться принять решение на основе результатов.

## <a name="data-matching-multiple-list-entities"></a>Данные, соответствующие нескольким сущностям списка

Если слово или фраза соответствует нескольким сущностям списка, запрос конечной точки возвращает каждую сущность списка.

Для запроса `when is the best time to go to red rock?` , и приложение содержит слово `red` в нескольких СПИСКАХ, Luis распознает все сущности и возвращает массив сущностей в качестве части ответа КОНЕЧНОЙ точки JSON.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о добавлении сущностей в приложение LUIS см. в статье [Добавление сущностей](luis-how-to-add-entities.md).
