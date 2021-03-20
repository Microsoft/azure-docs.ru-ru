---
title: Изменение данных — LUIS
description: Узнайте, как изменить данные перед прогнозированием в службе "Распознавание речи" (LUIS).
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 05/06/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 621a41f743b751a8c24bf6f6ad8497fb5c79775c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95026016"
---
# <a name="alter-utterance-data-before-or-during-prediction"></a>Изменение данных высказываний до или во время прогнозирования
LUIS предоставляет способы управления высказыванием до или во время прогнозирования. Сюда входит [исправление орфографических](luis-tutorial-bing-spellcheck.md)ошибок и устранение проблем с часовым поясом для предварительно созданной [datetimeV2](luis-reference-prebuilt-datetimev2.md).

## <a name="correct-spelling-errors-in-utterance"></a>Исправление орфографических ошибок в высказывании


### <a name="v3-runtime"></a>Среда выполнения v3

Предварительная обработка текста для исправления правописания перед отправкой utterance в LUIS. Используйте пример фразы продолжительностью с правильным написанием, чтобы убедиться, что вы получаете правильные прогнозы.

Используйте [Проверка орфографии Bing](../bing-spell-check/overview.md) , чтобы исправить текст перед отправкой в Luis.

### <a name="prior-to-v3-runtime"></a>До версии 3 среды выполнения

Для исправления орфографических ошибок в высказывании LUIS использует [API проверки орфографии Bing версии 7](../Bing-Spell-Check/overview.md). LUIS требуется ключ, связанный с этой службой. Создайте ключ, а затем добавьте его в качестве параметра QueryString в [конечную точку](https://go.microsoft.com/fwlink/?linkid=2092356).

Чтобы исправления орфографических ошибок вступили в силу, конечной точке требуется два параметра.

|Параметр|Значение|
|--|--|
|`spellCheck`|Логическое|
|`bing-spell-check-subscription-key`|Ключ конечной точки [API проверки орфографии Bing версии 7](https://azure.microsoft.com/services/cognitive-services/spell-check/)|

Когда [API проверки орфографии Bing версии 7](https://azure.microsoft.com/services/cognitive-services/spell-check/) обнаруживает ошибку, из конечной точки возвращаются исходное выражение, исправленное выражение и прогнозирование.

#### <a name="v2-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V2](#tab/V2)

```JSON
{
  "query": "Book a flite to London?",
  "alteredQuery": "Book a flight to London?",
  "topScoringIntent": {
    "intent": "BookFlight",
    "score": 0.780123
  },
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V3](#tab/V3)

```JSON
{
    "query": "Book a flite to London?",
    "prediction": {
        "normalizedQuery": "book a flight to london?",
        "topIntent": "BookFlight",
        "intents": {
            "BookFlight": {
                "score": 0.780123
            }
        },
        "entities": {},
    }
}
```

* * *

### <a name="list-of-allowed-words"></a>Список разрешенных слов
API проверки орфографии Bing, используемый в LUIS, не поддерживает список игнорируемых слов во время изменений проверки орфографии. Если необходимо разрешить список слов или акронимов, обработайте utterance в клиентском приложении перед отправкой utterance в LUIS для прогнозирования намерения.

## <a name="change-time-zone-of-prebuilt-datetimev2-entity"></a>Изменение часового пояса предварительно созданной сущности datetimeV2
Когда приложение LUIS использует предварительно созданную сущность [datetimeV2](luis-reference-prebuilt-datetimev2.md) , в ответе прогноза может возвращаться значение DateTime. Часовой пояс запроса используется для определения правильного возвращаемого значения даты и времени. Если запрос поступает от бота или другого централизованного приложения до передачи в LUIS, исправьте часовой пояс, используемый LUIS.

### <a name="v3-prediction-api-to-alter-timezone"></a>V3 прогнозирование API для изменения часового пояса

В версии 3 `datetimeReference` определяет смещение часового пояса. Дополнительные сведения о [прогнозах v3](luis-migration-api-v3.md#v3-post-body).

### <a name="v2-prediction-api-to-alter-timezone"></a>API прогнозирования версии 2 для изменения часового пояса
Часовой пояс исправляется путем добавления часового пояса пользователя к конечной точке с помощью `timezoneOffset` параметра, основанного на версии API. Значение параметра должно быть положительным или отрицательным числом в минутах для изменения времени.

#### <a name="v2-prediction-daylight-savings-example"></a>Пример прогнозирования летнего времени на версии 2
Если вам нужен возвращенный datetimeV2, чтобы настроить переход на летнее время, используйте параметр QueryString со значением +/-в минутах для запроса к [конечной точке](https://go.microsoft.com/fwlink/?linkid=2092356) .

Добавьте 60 минут:

`https://{region}.api.cognitive.microsoft.com/luis/v2.0/apps/{appId}?q=Turn the lights on?timezoneOffset=60&verbose={boolean}&spellCheck={boolean}&staging={boolean}&bing-spell-check-subscription-key={string}&log={boolean}`

Удалите 60 минут:

`https://{region}.api.cognitive.microsoft.com/luis/v2.0/apps/{appId}?q=Turn the lights on?timezoneOffset=-60&verbose={boolean}&spellCheck={boolean}&staging={boolean}&bing-spell-check-subscription-key={string}&log={boolean}`

#### <a name="v2-prediction-c-code-determines-correct-value-of-parameter"></a>Код C# прогнозирования версии 2 определяет правильное значение параметра

В следующем коде C# используется метод [FindSystemTimeZoneById](/dotnet/api/system.timezoneinfo.findsystemtimezonebyid#examples) класса [TimeZoneInfo](/dotnet/api/system.timezoneinfo) для определения правильного значения смещения на основе системного времени:

```csharp
// Get CST zone id
TimeZoneInfo targetZone = TimeZoneInfo.FindSystemTimeZoneById("Central Standard Time");

// Get local machine's value of Now
DateTime utcDatetime = DateTime.UtcNow;

// Get Central Standard Time value of Now
DateTime cstDatetime = TimeZoneInfo.ConvertTimeFromUtc(utcDatetime, targetZone);

// Find timezoneOffset/datetimeReference
int offset = (int)((cstDatetime - utcDatetime).TotalMinutes);
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Исправление орфографических ошибок с помощью этого руководства](luis-tutorial-bing-spellcheck.md)