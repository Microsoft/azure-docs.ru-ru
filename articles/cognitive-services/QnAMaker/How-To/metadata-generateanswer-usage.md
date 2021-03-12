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
ms.openlocfilehash: 7e8d1b13dfd802df820bea4015e411dbb85540ba
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103011431"
---
# <a name="get-an-answer-with-the-generateanswer-api"></a>Получение ответа с помощью API Женератеансвер

Чтобы получить прогнозируемый ответ на вопрос пользователя, используйте API Женератеансвер. При публикации базы знаний можно просмотреть сведения об использовании этого API на странице **Публикация** . Вы также можете настроить API для фильтрации ответов на основе тегов метаданных и проверить базу знаний из конечной точки с помощью параметра строки тестового запроса.

<a name="generateanswer-api"></a>

## <a name="get-answer-predictions-with-the-generateanswer-api"></a>Получение прогнозов ответов с помощью API Женератеансвер

Используйте [API женератеансвер](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer) в роботе или приложении для запроса базы знаний с пользовательским вопросом, чтобы получить лучшее совпадение из пар вопросов и ответов.

<a name="generateanswer-endpoint"></a>

## <a name="publish-to-get-generateanswer-endpoint"></a>Публикация для получения конечной точки Женератеансвер

После публикации базы знаний либо с [портала QnA Maker](https://www.qnamaker.ai), либо с помощью [API](/rest/api/cognitiveservices/qnamaker/knowledgebase/publish)можно получить сведения о конечной точке женератеансвер.

Вот как это можно сделать.
1. Войдите на портал [https://www.qnamaker.ai](https://www.qnamaker.ai).
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
|Только точные ответы|Да|true|
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
