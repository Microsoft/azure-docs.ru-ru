---
title: Метаданные с использованием API GenerateAnswer — QnA Maker
titleSuffix: Azure Cognitive Services
description: QnA Maker позволяет добавлять метаданные в виде пар "ключ-значение" в пары "вопрос — ответ". Вы можете фильтровать результаты по запросам пользователей и хранить дополнительные сведения, которые можно использовать в дальнейших беседах.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 1c2b608107beff2a4f34325f8a6e5be3a0551053
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102051911"
---
# <a name="get-an-answer-with-the-generateanswer-api-and-metadata"></a>Получение ответа с помощью API и метаданных Женератеансвер

Чтобы получить прогнозируемый ответ на вопрос пользователя, используйте API Женератеансвер. При публикации базы знаний можно просмотреть сведения об использовании этого API на странице **Публикация** . Вы также можете настроить API для фильтрации ответов на основе тегов метаданных и проверить базу знаний из конечной точки с помощью параметра строки тестового запроса.

QnA Maker позволяет добавлять метаданные в виде пар "ключ — значение" в свои пары вопросов и ответов. Затем эти сведения можно использовать для фильтрации результатов в пользовательских запросах, а также для хранения дополнительных сведений, которые можно использовать в дальнейших беседах. Дополнительные сведения см. в разделе [Knowledge base](../index.yml) (База знаний).

<a name="qna-entity"></a>

## <a name="store-questions-and-answers-with-a-qna-entity"></a>Хранение вопросов и ответов с помощью сущности QnA

Важно понимать, как QnA Maker хранит данные вопроса и ответов. На рисунке ниже показана сущность QnA.

![Иллюстрация сущности QnA](../media/qnamaker-how-to-metadata-usage/qna-entity.png)

У каждой сущности QnA имеется уникальный постоянный идентификатор. Идентификатор можно использовать для внесения обновлений в определенную сущность QnA.

<a name="generateanswer-api"></a>

## <a name="get-answer-predictions-with-the-generateanswer-api"></a>Получение прогнозов ответов с помощью API Женератеансвер

Используйте [API женератеансвер](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer) в роботе или приложении для запроса базы знаний с пользовательским вопросом, чтобы получить лучшее совпадение из пар вопросов и ответов.

<a name="generateanswer-endpoint"></a>

## <a name="publish-to-get-generateanswer-endpoint"></a>Публикация для получения конечной точки Женератеансвер

После публикации базы знаний либо с [портала QnA Maker](https://www.qnamaker.ai), либо с помощью [API](/rest/api/cognitiveservices/qnamaker/knowledgebase/publish)можно получить сведения о конечной точке женератеансвер.

Вот как это можно сделать.
1. Выполните вход на странице [https://www.qnamaker.ai](https://www.qnamaker.ai).
1. В окне **Мои базы знаний** выберите **Просмотреть код** для своей базы знаний.
    ![Снимок экрана с базами знаний](../media/qnamaker-how-to-metadata-usage/my-knowledge-bases.png)
1. Получите сведения о конечной точке GenerateAnswer.

    # <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/v1)

    ![Снимок экрана сведений о конечной точке](../media/qnamaker-how-to-metadata-usage/view-code.png)

    # <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/v2)

    ![Снимок экрана: Управление сведениями о конечной точке](../media/qnamaker-how-to-metadata-usage/view-code-managed.png)

    ---

Можно также получить сведения о конечной точки на вкладке **Settings** (Параметры) базы знаний.

<a name="generateanswer-request"></a>

## <a name="generateanswer-request-configuration"></a>Конфигурация запроса Женератеансвер

Для вызова GenerateAnswer используется HTTP-запрос POST. Пример кода, в котором демонстрируется вызов GenerateAnswer, доступен в этих [кратких руководствах](../quickstarts/quickstart-sdk.md#generate-an-answer-from-the-knowledge-base).

Запрос POST использует:

* Обязательные [Параметры URI](/rest/api/cognitiveservices/qnamakerruntime/runtime/train#uri-parameters)
* Обязательное свойство заголовка, `Authorization` , для безопасности
* Обязательные [Свойства текста](/rest/api/cognitiveservices/qnamakerruntime/runtime/train#feedbackrecorddto).

URL-адрес Женератеансвер имеет следующий формат:

```
https://{QnA-Maker-endpoint}/knowledgebases/{knowledge-base-ID}/generateAnswer
```

Не забудьте задать в качестве значения свойства "заголовок HTTP" `Authorization` значение строки `EndpointKey` с конечным пробелом, а затем ключ конечной точки, найденный на странице **параметров** .

Пример текста JSON выглядит следующим образом:

```json
{
    "question": "qna maker and luis",
    "top": 6,
    "isTest": true,
    "scoreThreshold": 30,
    "rankerType": "" // values: QuestionOnly
    "strictFilters": [
    {
        "name": "category",
        "value": "api"
    }],
    "userId": "sd53lsY="
}
```

Дополнительные сведения о [ранкертипе](../concepts/best-practices.md#choosing-ranker-type).

Предыдущий JSON запросил только ответы, которые имеют пороговую оценку 30% или выше.

<a name="generateanswer-response"></a>

## <a name="generateanswer-response-properties"></a>Свойства ответа Женератеансвер

[Ответ](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer#successful-query) — это объект JSON, включающий всю информацию, необходимую для вывода ответа, и следующий ход в диалоге, если он доступен.

```json
{
    "answers": [
        {
            "score": 38.54820341616869,
            "Id": 20,
            "answer": "There is no direct integration of LUIS with QnA Maker. But, in your bot code, you can use LUIS and QnA Maker together. [View a sample bot](https://github.com/Microsoft/BotBuilder-CognitiveServices/tree/master/Node/samples/QnAMaker/QnAWithLUIS)",
            "source": "Custom Editorial",
            "questions": [
                "How can I integrate LUIS with QnA Maker?"
            ],
            "metadata": [
                {
                    "name": "category",
                    "value": "api"
                }
            ]
        }
    ]
}
```

Предыдущий формат JSON ответил на ответ с показателем 38,5%.

## <a name="use-qna-maker-with-a-bot-in-c"></a>Использование QnA Maker с программой-роботом в C #

Платформа Bot предоставляет доступ к свойствам QnA Maker с помощью API- [интерфейса «ответ](/dotnet/api/microsoft.bot.builder.ai.qna.qnamaker.getanswersasync#Microsoft_Bot_Builder_AI_QnA_QnAMaker_GetAnswersAsync_Microsoft_Bot_Builder_ITurnContext_Microsoft_Bot_Builder_AI_QnA_QnAMakerOptions_System_Collections_Generic_Dictionary_System_String_System_String__System_Collections_Generic_Dictionary_System_String_System_Double__)»:

```csharp
using Microsoft.Bot.Builder.AI.QnA;
var metadata = new Microsoft.Bot.Builder.AI.QnA.Metadata();
var qnaOptions = new QnAMakerOptions();

metadata.Name = Constants.MetadataName.Intent;
metadata.Value = topIntent;
qnaOptions.StrictFilters = new Microsoft.Bot.Builder.AI.QnA.Metadata[] { metadata };
qnaOptions.Top = Constants.DefaultTop;
qnaOptions.ScoreThreshold = 0.3F;
var response = await _services.QnAServices[QnAMakerKey].GetAnswersAsync(turnContext, qnaOptions);
```

Предыдущий JSON запросил только ответы, которые имеют пороговую оценку 30% или выше.

## <a name="use-qna-maker-with-a-bot-in-nodejs"></a>Использование QnA Maker с программой-роботом в Node.js

Платформа Bot предоставляет доступ к свойствам QnA Maker с помощью API- [интерфейса «ответ](/javascript/api/botbuilder-ai/qnamaker?preserve-view=true&view=botbuilder-ts-latest#generateanswer-string---undefined--number--number-)»:

```javascript
const { QnAMaker } = require('botbuilder-ai');
this.qnaMaker = new QnAMaker(endpoint);

// Default QnAMakerOptions
var qnaMakerOptions = {
    ScoreThreshold: 0.30,
    Top: 3
};
var qnaResults = await this.qnaMaker.getAnswers(stepContext.context, qnaMakerOptions);
```

Предыдущий JSON запросил только ответы, которые имеют пороговую оценку 30% или выше.

<a name="metadata-example"></a>

## <a name="use-metadata-to-filter-answers-by-custom-metadata-tags"></a>Использование метаданных для фильтрации ответов по тегам пользовательских метаданных

Добавление метаданных позволяет фильтровать ответы по этим тегам метаданных. Добавьте столбец метаданных из меню **Параметры представления** . Добавьте метаданные в базу знаний, щелкнув значок метаданных, **+** чтобы добавить пару метаданных. Эта пара состоит из одного ключа и одного значения.

![Снимок экрана: Добавление метаданных](../media/qnamaker-how-to-metadata-usage/add-metadata.png)

<a name="filter-results-with-strictfilters-for-metadata-tags"></a>

## <a name="filter-results-with-strictfilters-for-metadata-tags"></a>Фильтрация результатов с помощью strictFilters по тегам метаданных

Примите во внимание вопрос пользователя "когда закроется это отель?", где предполагается, что цель предполагается для ресторана "сообразительного".

Поскольку результаты требуются только для ресторана "сообразительного", можно установить фильтр в вызове Женератеансвер для метаданных "имя ресторана". Это показано в следующем примере:

```json
{
    "question": "When does this hotel close?",
    "top": 1,
    "strictFilters": [ { "name": "restaurant", "value": "paradise"}]
}
```

### <a name="logical-and-by-default"></a>Логическое и по умолчанию

Чтобы объединить несколько фильтров метаданных в запросе, добавьте фильтры дополнительных метаданных в массив `strictFilters` Свойства. По умолчанию значения логически объединены (и). Логическое сочетание требует, чтобы все фильтры совпадали с парами QnA, чтобы пара возвращалась в ответе.

Это эквивалентно использованию `strictFiltersCompoundOperationType` свойства со значением `AND` .

### <a name="logical-or-using-strictfilterscompoundoperationtype-property"></a>Логическое или с использованием свойства Стриктфилтерскомпаундоператионтипе

При объединении нескольких фильтров метаданных, если важен только один или несколько соответствующих фильтров, используйте `strictFiltersCompoundOperationType` свойство со значением `OR` .

Это позволяет базе знаний возвращать ответы при совпадении фильтра, но не возвращает ответы, не имеющие метаданных.

```json
{
    "question": "When do facilities in this hotel close?",
    "top": 1,
    "strictFilters": [
      { "name": "type","value": "restaurant"},
      { "name": "type", "value": "bar"},
      { "name": "type", "value": "poolbar"}
    ],
    "strictFiltersCompoundOperationType": "OR"
}
```

### <a name="metadata-examples-in-quickstarts"></a>Примеры метаданных в кратком руководстве

Дополнительные сведения о метаданных см. в кратком руководстве по QnA Maker портале для метаданных.
* [Разработка: добавление метаданных в пару "вопрос-ответ"](../quickstarts/add-question-metadata-portal.md#add-metadata-to-filter-the-answers)
* [Прогнозирование запросов: фильтрация ответов по метаданным](../quickstarts/get-answer-from-knowledge-base-using-url-tool.md)

<a name="keep-context"></a>

## <a name="use-question-and-answer-results-to-keep-conversation-context"></a>Использование результатов вопросов и ответов для сохранения контекста беседы

Ответ на Женератеансвер содержит сведения о метаданных соответствующей пары вопросов и ответов. Эти сведения можно использовать в клиентском приложении для хранения контекста предыдущего диалога, который будет использоваться в последующих диалогах.

```json
{
    "answers": [
        {
            "questions": [
                "What is the closing time?"
            ],
            "answer": "10.30 PM",
            "score": 100,
            "id": 1,
            "source": "Editorial",
            "metadata": [
                {
                    "name": "restaurant",
                    "value": "paradise"
                },
                {
                    "name": "location",
                    "value": "secunderabad"
                }
            ]
        }
    ]
}
```

## <a name="match-questions-only-by-text"></a>Сопоставлять только вопросы по тексту

По умолчанию QnA Maker выполняет поиск по вопросам и ответам. Если вы хотите выполнять поиск только по вопросам, используйте `RankerType=QuestionOnly` в тексте сообщения запроса женератеансвер.

Можно выполнять поиск в опубликованной базе знаний, с помощью `isTest=false` или в тесте базы знаний с помощью `isTest=true` .

```json
{
  "question": "Hi",
  "top": 30,
  "isTest": true,
  "RankerType":"QuestionOnly"
}
```

## <a name="return-precise-answers"></a>Возврат точных ответов

### <a name="generate-answer-api"></a>Создать API ответов 

Пользователь может включить [точные ответы](../reference-precise-answering.md) при использовании управляемого ресурса QnA Maker. Параметр Ансверспанрекуест должен быть обновлен для того же самого.

```json
{
    "question": "How long it takes to charge surface pro 4?",
    "top": 3,
    "answerSpanRequest": {
        "enable": true,
        "topAnswersWithSpan": 1
    }
}
```

Аналогичным образом пользователи могут отключить точные ответы, не задавая параметр Ансверспанрекуест.

```json
{
    "question": "How long it takes to charge surface pro 4?",
    "top": 3
}
```
### <a name="bot-settings"></a>Параметры программы-робота

Если вы хотите настроить точные параметры ответов для службы Bot, перейдите к ресурсу службы приложений для программы-робота. Затем необходимо обновить конфигурации, добавив следующий параметр.

- енаблепреЦисеансвер
- дисплайпреЦисеансверонли

|настройка отображения;|енаблепреЦисеансвер|дисплайпреЦисеансверонли|
|:--|--|--|
|Только точные ответы|true|true|
|Только длинные ответы|false|false|
|Как длинные, так и точные ответы|true|false|

## <a name="common-http-errors"></a>Распространенные ошибки HTTP

|Код|Описание|
|:--|--|
|"2xx"|Успешное завершение|
|400|Параметры запроса указаны неправильно. Это означает, что требуемые параметры отсутствуют, имеют неправильный формат или слишком большой размер|
|400|Текст запроса указан неправильно. Это означает, что JSON отсутствует, имеет неправильный формат или слишком большой размер|
|401|Недопустимый ключ|
|403|Доступ запрещен. У вас нет необходимых разрешений|
|404|База знаний не существует|
|410|Этот API устарел и больше недоступен|

## <a name="next-steps"></a>Дальнейшие действия

На странице **Публикация** также содержатся сведения для [создания ответа](../Quickstarts/get-answer-from-knowledge-base-using-url-tool.md) с помощью POST-или перелистывания.

> [!div class="nextstepaction"]
> [Анализ базы знаний](../how-to/get-analytics-knowledge-base.md)
> [!div class="nextstepaction"]
> [Оценка достоверности](../Concepts/confidence-score.md)
