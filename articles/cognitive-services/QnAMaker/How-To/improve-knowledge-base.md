---
title: Рекомендуемые вопросы об активном обучении — QnA Maker
description: Улучшите качество базы знаний с помощью активного обучения. Просмотр, принятие или отклонение, добавление без удаления или изменения существующих вопросов.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 04/06/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 3fe6ee8336872c04e85b732713494adf0fefa28a
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103011448"
---
# <a name="accept-active-learning-suggested-questions-in-the-knowledge-base"></a>Принять предлагаемые активные обучающие вопросы в базе знаний


<a name="accept-an-active-learning-suggestion-in-the-knowledge-base"></a>

Активное обучение изменяет базу знаний или Служба поиска после утверждения предложения, а затем сохраняет и обучить. Если вы утвердите предложение, оно будет добавлено в качестве альтернативного вопроса.

## <a name="turn-on-active-learning"></a>Включение активного обучения

Чтобы увидеть предлагаемые вопросы, необходимо [включить активное обучение](../concepts/active-learning-suggestions.md) для ресурса QnA Maker.

## <a name="view-suggested-questions"></a>Просмотреть предлагаемые вопросы

1. Чтобы просмотреть предлагаемые вопросы, на странице « **изменение** базы знаний» выберите **Просмотр параметров**, а затем выберите **Показать активные предложения по обучению**. Этот параметр будет отключен, если отсутствуют предложения для любой пары вопросов и ответов.  

    [![В разделе "Правка" на портале выберите Показать предложения, чтобы просмотреть альтернативы нового вопроса активного обучения.](../media/improve-knowledge-base/show-suggestions-button.png)](../media/improve-knowledge-base/show-suggestions-button.png#lightbox)

1. Отфильтруйте базу знаний с помощью пар вопросов и ответов, чтобы отображались только предложения, выбрав **Фильтр по предложениям**.

    [![Используйте переключатель фильтровать по предложениям для просмотра только альтернативных вариантов вопроса, предлагаемых в активном обучении.](../media/improve-knowledge-base/filter-by-suggestions.png)](../media/improve-knowledge-base/filter-by-suggestions.png#lightbox)

1. Каждая пара QnA предлагает новые альтернативные варианты вопроса с галочкой, `✔` , чтобы принять вопрос или `x` отклонить предложения. Установите флажок, чтобы добавить вопрос.

    [![Выберите или отклоните альтернативные варианты вопроса активного обучения, выбрав зеленую галочку или красную метку удаления.](../media/improve-knowledge-base/accept-active-learning-suggestions-small.png)](../media/improve-knowledge-base/accept-active-learning-suggestions.png#lightbox)

    Вы можете добавить или удалить _все предложения_ , выбрав **Добавить все** или **Отклонить все** на контекстной панели инструментов.

1. Выберите **Save and Train** (Сохранить и обучить), чтобы сохранить изменения в базе знаний.

1. Выберите **опубликовать** , чтобы разрешить доступ к изменениям из [API женератеансвер](metadata-generateanswer-usage.md#generateanswer-request-configuration).

    Если пять или более похожих запросов являются кластеризованными, каждые 30 минут QnA Maker предлагает дополнительные вопросы, которые можно принять или отклонить.


<a name="#score-proximity-between-knowledge-base-questions"></a>

## <a name="active-learning-suggestions-are-saved-in-the-exported-knowledge-base"></a>Активные предложения по обучению сохраняются в экспортированной базе знаний.

Если в приложении включено активное обучение и вы экспортируете приложение, `SuggestedQuestions` столбец в TSV файле содержит активные обучающие данные.

`SuggestedQuestions`Столбец представляет собой объект JSON сведений о неявных, `autosuggested` и явных, `usersuggested` обратных отзывах. Пример этого объекта JSON для одного отправленного пользователем вопроса `help` :

```JSON
[
    {
        "clusterHead": "help",
        "totalAutoSuggestedCount": 1,
        "totalUserSuggestedCount": 0,
        "alternateQuestionList": [
            {
                "question": "help",
                "autoSuggestedCount": 1,
                "userSuggestedCount": 0
            }
        ]
    }
]
```

При повторном импорте этого приложения активное обучение продолжит собираются сведения и рекомендации для своей базы знаний.


### <a name="architectural-flow-for-using-generateanswer-and-train-apis-from-a-bot"></a>Архитектурный поток для использования Женератеансвер и обучения API-интерфейсов из Bot

Для использования активного обучения в роботе или другом клиентском приложении следует использовать следующую архитектурную последовательность:

1. Bot [получает ответ от базы знаний](#use-the-top-property-in-the-generateanswer-request-to-get-several-matching-answers) с помощью API женератеансвер, используя `top` свойство для получения ряда ответов.

2. Bot определяет явный отзыв:
    * Используя собственную [бизнес-логику](#use-the-score-property-along-with-business-logic-to-get-list-of-answers-to-show-user), отфильтруйте низкие оценки.
    * В окне "bot" или "клиент-приложение" отобразится список возможных ответов для пользователя и получите выбранный ответ.
3. Bot [отправляет выбранный ответ обратно в QnA Maker](#bot-framework-sample-code) с помощью [API обучения](#train-api).

### <a name="use-the-top-property-in-the-generateanswer-request-to-get-several-matching-answers"></a>Использование свойства Top в запросе Женератеансвер для получения нескольких соответствующих ответов

При отправке вопроса в QnA Maker для ответа `top` свойство тела JSON задает число возвращаемых ответов.

```json
{
    "question": "wi-fi",
    "isTest": false,
    "top": 3
}
```

### <a name="use-the-score-property-along-with-business-logic-to-get-list-of-answers-to-show-user"></a>Использование свойства Score вместе с бизнес-логикой для получения списка ответов для отображения пользователя

Когда клиентское приложение (например, Bot чата) получает ответ, возвращаются 3 основных вопроса. Используйте `score` свойство, чтобы проанализировать близость между показателями. Этот диапазон сходства определяется собственной бизнес-логикой.

```json
{
    "answers": [
        {
            "questions": [
                "Wi-Fi Direct Status Indicator"
            ],
            "answer": "**Wi-Fi Direct Status Indicator**\n\nStatus bar icons indicate your current Wi-Fi Direct connection status:  \n\nWhen your device is connected to another device using Wi-Fi Direct, '$  \n\n+ *+ ' Wi-Fi Direct is displayed in the Status bar.",
            "score": 74.21,
            "id": 607,
            "source": "Bugbash KB.pdf",
            "metadata": []
        },
        {
            "questions": [
                "Wi-Fi - Connections"
            ],
            "answer": "**Wi-Fi**\n\nWi-Fi is a term used for certain types of Wireless Local Area Networks (WLAN). Wi-Fi communication requires access to a wireless Access Point (AP).",
            "score": 74.15,
            "id": 599,
            "source": "Bugbash KB.pdf",
            "metadata": []
        },
        {
            "questions": [
                "Turn Wi-Fi On or Off"
            ],
            "answer": "**Turn Wi-Fi On or Off**\n\nTurning Wi-Fi on makes your device able to discover and connect to compatible in-range wireless APs.  \n\n1.  From a Home screen, tap ::: Apps > e Settings .\n2.  Tap Connections > Wi-Fi , and then tap On/Off to turn Wi-Fi on or off.",
            "score": 69.99,
            "id": 600,
            "source": "Bugbash KB.pdf",
            "metadata": []
        }
    ]
}
```

## <a name="client-application-follow-up-when-questions-have-similar-scores"></a>Дальнейшие действия клиентского приложения, когда у нескольких вопросов сходные оценки

Клиентское приложение отображает вопросы с возможностью выбора _одного вопроса_ , который наиболее соответствует намерениям.

Когда пользователь выбирает один из существующих вопросов, клиентское приложение отправляет выбор пользователя в качестве отзыва с помощью API обучения QnA Maker. Этот отзыв завершает активный цикл обратной связи.

## <a name="train-api"></a>API обучения

Активная обратная связь отправляется в QnA Maker с запросом на обучение API POST. Сигнатура API:

```http
POST https://<QnA-Maker-resource-name>.azurewebsites.net/qnamaker/knowledgebases/<knowledge-base-ID>/train
Authorization: EndpointKey <endpoint-key>
Content-Type: application/json
{"feedbackRecords": [{"userId": "1","userQuestion": "<question-text>","qnaId": 1}]}
```

|Свойство HTTP-запроса|Имя|Тип|Назначение|
|--|--|--|--|
|Параметр URL-маршрута|Идентификатор базы знаний|строка|Идентификатор GUID для базы знаний.|
|Пользовательский поддомен|Имя ресурса QnAMaker|строка|Имя ресурса используется в качестве пользовательского поддомена для QnA Maker. Это можно найти на странице параметры после публикации базы знаний. Он указан как `host` .|
|Заголовок|Content-Type|строка|Тип носителя текста, отправляемого в API. Значение по умолчанию: `application/json`|
|Заголовок|Авторизация|строка|Ключ конечной точки (EndpointKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx).|
|Текст запроса POST|Объект JSON|JSON|Отзыв по обучению|

Тело JSON имеет несколько параметров:

|Свойство тела JSON|Тип|Назначение|
|--|--|--|--|
|`feedbackRecords`|array|Список отзывов.|
|`userId`|строка|Идентификатор пользователя, который принимает предлагаемые вопросы. Пользователь должен иметь формат идентификатора пользователя. Например, адрес электронной почты может быть допустимым ИДЕНТИФИКАТОРом пользователя в вашей архитектуре. Необязательный параметр.|
|`userQuestion`|строка|Точный текст запроса пользователя. Обязательный.|
|`qnaID`|number|Идентификатор вопроса, найденный в [ответе женератеансвер](metadata-generateanswer-usage.md#generateanswer-response-properties). |

Пример текста JSON выглядит следующим образом:

```json
{
    "feedbackRecords": [
        {
            "userId": "1",
            "userQuestion": "<question-text>",
            "qnaId": 1
        }
    ]
}
```

Успешный ответ возвращает состояние 204 и не имеет текста ответа JSON.

### <a name="batch-many-feedback-records-into-a-single-call"></a>Пакетная обработка большого количества записей отзывов в один вызов

В клиентском приложении, таком как Bot, можно сохранить данные, а затем отправить много записей в одном тексте JSON в `feedbackRecords` массиве.

Пример текста JSON выглядит следующим образом:

```json
{
    "feedbackRecords": [
        {
            "userId": "1",
            "userQuestion": "How do I ...",
            "qnaId": 1
        },
        {
            "userId": "2",
            "userQuestion": "Where is ...",
            "qnaId": 40
        },
        {
            "userId": "3",
            "userQuestion": "When do I ...",
            "qnaId": 33
        }
    ]
}
```



<a name="active-learning-is-saved-in-the-exported-apps-tsv-file"></a>

## <a name="bot-framework-sample-code"></a>Пример кода для платформы Bot

Код платформы Bot должен вызывать API обучения, если запрос пользователя должен использоваться для активного обучения. Написание кода состоит из двух частей:

* Определение необходимости использования запроса для активного обучения
* Отправить запрос в API обучения QnA Maker для активного обучения

В [примере Azure Bot](https://github.com/microsoft/BotBuilder-Samples)были запрограммированы оба действия.

### <a name="example-c-code-for-train-api-with-bot-framework-4x"></a>Пример кода C# для обучения API с помощью Bot Framework 4. x

В следующем примере кода показано, как отправить сведения обратно в QnA Maker с помощью API обучения.

```csharp
public class FeedbackRecords
{
    // <summary>
    /// List of feedback records
    /// </summary>
    [JsonProperty("feedbackRecords")]
    public FeedbackRecord[] Records { get; set; }
}

/// <summary>
/// Active learning feedback record
/// </summary>
public class FeedbackRecord
{
    /// <summary>
    /// User id
    /// </summary>
    public string UserId { get; set; }

    /// <summary>
    /// User question
    /// </summary>
    public string UserQuestion { get; set; }

    /// <summary>
    /// QnA Id
    /// </summary>
    public int QnaId { get; set; }
}

/// <summary>
/// Method to call REST-based QnAMaker Train API for Active Learning
/// </summary>
/// <param name="endpoint">Endpoint URI of the runtime</param>
/// <param name="FeedbackRecords">Feedback records train API</param>
/// <param name="kbId">Knowledgebase Id</param>
/// <param name="key">Endpoint key</param>
/// <param name="cancellationToken"> Cancellation token</param>
public async static void CallTrain(string endpoint, FeedbackRecords feedbackRecords, string kbId, string key, CancellationToken cancellationToken)
{
    var uri = endpoint + "/knowledgebases/" + kbId + "/train/";

    using (var client = new HttpClient())
    {
        using (var request = new HttpRequestMessage())
        {
            request.Method = HttpMethod.Post;
            request.RequestUri = new Uri(uri);
            request.Content = new StringContent(JsonConvert.SerializeObject(feedbackRecords), Encoding.UTF8, "application/json");
            request.Headers.Add("Authorization", "EndpointKey " + key);

            var response = await client.SendAsync(request, cancellationToken);
            await response.Content.ReadAsStringAsync();
        }
    }
}
```

### <a name="example-nodejs-code-for-train-api-with-bot-framework-4x"></a>Пример Node.js кода для обучения API с помощью Bot Framework 4. x

В следующем примере кода показано, как отправить сведения обратно в QnA Maker с помощью API обучения.

```javascript
async callTrain(stepContext){

    var trainResponses = stepContext.values[this.qnaData];
    var currentQuery = stepContext.values[this.currentQuery];

    if(trainResponses.length > 1){
        var reply = stepContext.context.activity.text;
        var qnaResults = trainResponses.filter(r => r.questions[0] == reply);

        if(qnaResults.length > 0){

            stepContext.values[this.qnaData] = qnaResults;

            var feedbackRecords = {
                FeedbackRecords:[
                    {
                        UserId:stepContext.context.activity.id,
                        UserQuestion: currentQuery,
                        QnaId: qnaResults[0].id
                    }
                ]
            };

            // Call Active Learning Train API
            this.activeLearningHelper.callTrain(this.qnaMaker.endpoint.host, feedbackRecords, this.qnaMaker.endpoint.knowledgeBaseId, this.qnaMaker.endpoint.endpointKey);

            return await stepContext.next(qnaResults);
        }
        else{

            return await stepContext.endDialog();
        }
    }

    return await stepContext.next(stepContext.result);
}
```

## <a name="best-practices"></a>Рекомендации

Рекомендации по использованию активного обучения см. в статье [Рекомендации по использованию базы знаний QnA Maker](../Concepts/best-practices.md#active-learning).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Использование метаданных с помощью API GenerateAnswer](metadata-generateanswer-usage.md)
