---
title: Краткое руководство. Клиентская библиотека Анализа текста версии 3 для Node.js | Документация Майкрософт
description: Начало работы с клиентской библиотекой Анализа текста версии 3 для Node.js.
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 02/09/2021
ms.author: aahi
ms.reviewer: sumeh, assafi
ms.custom: devx-track-js
ms.openlocfilehash: a71559c5f75694d314547220f04fc2d7d7e1dbc6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104599087"
---
<a name="HOLTop"></a>

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

[Справочная документация по версии 3](/javascript/api/overview/azure/ai-text-analytics-readme?preserve-view=true&view=azure-node-preview) | [Исходный код библиотеки версии 3](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics) | [Пакет (NPM) для версии 3](https://www.npmjs.com/package/@azure/ai-text-analytics) | [Примеры для версии 3](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics/samples)


# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

[Справочная документация по версии 3](/javascript/api/overview/azure/ai-text-analytics-readme) | [Исходный код библиотеки версии 3](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics) | [Пакет (NPM) для версии 3](https://www.npmjs.com/package/@azure/ai-text-analytics) | [Примеры для версии 3](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics/samples)


---

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* Текущая версия [Node.js](https://nodejs.org/).
* Получив подписку Azure, перейдите к <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics"  title="созданию ресурса Анализа текста"  target="_blank">созданию ресурса Анализа текста</a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Анализа текста потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.
* Чтобы использовать функцию анализа, вам потребуется ресурс Анализа текста ценовой категории "Стандартный" (S).

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-nodejs-application"></a>создание приложения Node.js;

В окне консоли (например, cmd, PowerShell или Bash) создайте новый каталог для приложения и перейдите в него. 

```console
mkdir myapp 

cd myapp
```

Выполните команду `npm init`, чтобы создать приложение узла с помощью файла `package.json`. 

```console
npm init
```
### <a name="install-the-client-library"></a>Установка клиентской библиотеки

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Установите пакеты NPM `@azure/ai-text-analytics`:

```console
npm install --save @azure/ai-text-analytics@5.1.0-beta.5
```

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/TextAnalytics/text-analytics-v3-client-library.js), где размещены примеры кода для этого краткого руководства. 


# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Установите пакеты NPM `@azure/ai-text-analytics`:

```console
npm install --save @azure/ai-text-analytics@5.0.0
```

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/TextAnalytics/text-analytics-v3-client-library.js), где размещены примеры кода для этого краткого руководства. 

---

Файл `package.json` этого приложения будет дополнен зависимостями.
Создайте файл с именем `index.js` и добавьте следующее:

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

```javascript
"use strict";

const { TextAnalyticsClient, AzureKeyCredential } = require("@azure/ai-text-analytics");
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

```javascript
"use strict";

const { TextAnalyticsClient, AzureKeyCredential } = require("@azure/ai-text-analytics");
```

---

Создайте переменные для конечной точки Azure и ключа ресурса.

[!INCLUDE [text-analytics-find-resource-information](../find-azure-resource-info.md)]

```javascript
const key = '<paste-your-text-analytics-key-here>';
const endpoint = '<paste-your-text-analytics-endpoint-here>';
```

## <a name="object-model"></a>Объектная модель

Клиент Анализа текста представляет собой объект `TextAnalyticsClient`, который выполняет проверку подлинности в Azure с использованием ключа. Этот клиент предоставляет несколько методов для анализа текста по отдельным строкам или пакетам строк.

Текст отправляется в API в формате списка `documents` объектов `dictionary`, содержащих комбинации атрибутов `id`, `text` и `language` в зависимости от используемого метода. Атрибут `text` содержит текст для анализа в источнике `language`, а `id` может иметь любое значение. 

Объект в ответе — это список, содержащий аналитику по каждому документу. 

## <a name="code-examples"></a>Примеры кода

* [Аутентификация клиента](#client-authentication)
* [Анализ тональности](#sentiment-analysis). 
* [Интеллектуальный анализ мнений](#opinion-mining)
* [Пример. Как определить язык с помощью Анализа текста](#language-detection)
* [Распознавание именованных сущностей](#named-entity-recognition-ner)
* [Связывание сущностей](#entity-linking)
* Персональные данные
* [Пример. Как извлечь ключевые фразы с помощью Анализа текста](#key-phrase-extraction)

## <a name="client-authentication"></a>Аутентификация клиента

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Создайте новый объект `TextAnalyticsClient`, используя в качестве параметров ключ и конечную точку.

```javascript
const textAnalyticsClient = new TextAnalyticsClient(endpoint,  new AzureKeyCredential(key));
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Создайте новый объект `TextAnalyticsClient`, используя в качестве параметров ключ и конечную точку.

```javascript
const textAnalyticsClient = new TextAnalyticsClient(endpoint,  new AzureKeyCredential(key));
```

---

## <a name="sentiment-analysis"></a>Анализ мнений

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Создайте массив строк, содержащий документ, который следует проанализировать. Вызовите метод клиента `analyzeSentiment()` и получите возвращенный объект `SentimentBatchResult`. Выполните итерацию результатов и выведите идентификатор каждого документа, тональность на уровне документа со степенями уверенности. Результат содержит тональность на уровне предложений, а также смещения, длину и показатели уверенности для каждого документа.

```javascript
async function sentimentAnalysis(client){

    const sentimentInput = [
        "I had the best day of my life. I wish you were there with me."
    ];
    const sentimentResult = await client.analyzeSentiment(sentimentInput);

    sentimentResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tDocument Sentiment: ${document.sentiment}`);
        console.log(`\tDocument Scores:`);
        console.log(`\t\tPositive: ${document.confidenceScores.positive.toFixed(2)} \tNegative: ${document.confidenceScores.negative.toFixed(2)} \tNeutral: ${document.confidenceScores.neutral.toFixed(2)}`);
        console.log(`\tSentences Sentiment(${document.sentences.length}):`);
        document.sentences.forEach(sentence => {
            console.log(`\t\tSentence sentiment: ${sentence.sentiment}`)
            console.log(`\t\tSentences Scores:`);
            console.log(`\t\tPositive: ${sentence.confidenceScores.positive.toFixed(2)} \tNegative: ${sentence.confidenceScores.negative.toFixed(2)} \tNeutral: ${sentence.confidenceScores.neutral.toFixed(2)}`);
        });
    });
}
sentimentAnalysis(textAnalyticsClient)
```

Выполните код с помощью `node index.js` в окне консоли.

### <a name="output"></a>Выходные данные

```console
ID: 0
        Document Sentiment: positive
        Document Scores:
                Positive: 1.00  Negative: 0.00  Neutral: 0.00
        Sentences Sentiment(2):
                Sentence sentiment: positive
                Sentences Scores:
                Positive: 1.00  Negative: 0.00  Neutral: 0.00
                Sentence sentiment: neutral
                Sentences Scores:
                Positive: 0.21  Negative: 0.02  Neutral: 0.77
```

### <a name="opinion-mining"></a>Интеллектуальный анализ данных

Чтобы выполнить анализ тональности с интеллектуальным анализом мнений, создайте массив строк, содержащий документ, который необходимо проанализировать. Вызовите метод клиента `analyzeSentiment()`, добавив флаг параметра `includeOpinionMining: true`, и получите возвращенный объект `SentimentBatchResult`. Выполните итерацию результатов и выведите идентификатор каждого документа, тональность на уровне документа со степенями уверенности. Для каждого документа в результате указана тональность уровня предложения, как показано выше, а также тональность на уровне мнения и аспекта.

```javascript
async function sentimentAnalysisWithOpinionMining(client){

  const sentimentInput = [
    {
      text: "The food and service were unacceptable, but the concierge were nice",
      id: "0",
      language: "en"
    }
  ];
  const results = await client.analyzeSentiment(sentimentInput, { includeOpinionMining: true });

  for (let i = 0; i < results.length; i++) {
    const result = results[i];
    console.log(`- Document ${result.id}`);
    if (!result.error) {
      console.log(`\tDocument text: ${sentimentInput[i].text}`);
      console.log(`\tOverall Sentiment: ${result.sentiment}`);
      console.log("\tSentiment confidence scores:", result.confidenceScores);
      console.log("\tSentences");
      for (const { sentiment, confidenceScores, opinions } of result.sentences) {
        console.log(`\t- Sentence sentiment: ${sentiment}`);
        console.log("\t  Confidence scores:", confidenceScores);
        console.log("\t  Mined opinions");
        for (const { target, assessments } of opinions) {
          console.log(`\t\t- Target text: ${target.text}`);
          console.log(`\t\t  Target sentiment: ${target.sentiment}`);
          console.log("\t\t  Target confidence scores:", target.confidenceScores);
          console.log("\t\t  Target assessments");
          for (const { text, sentiment } of assessments) {
            console.log(`\t\t\t- Text: ${text}`);
            console.log(`\t\t\t  Sentiment: ${sentiment}`);
          }
        }
      }
    } else {
      console.error(`\tError: ${result.error}`);
    }
  }
}
sentimentAnalysisWithOpinionMining(textAnalyticsClient)
```

Выполните код с помощью `node index.js` в окне консоли.

### <a name="output"></a>Выходные данные

```console
- Document 0
        Document text: The food and service were unacceptable, but the concierge were nice
        Overall Sentiment: positive
        Sentiment confidence scores: { positive: 0.84, neutral: 0, negative: 0.16 }
        Sentences
        - Sentence sentiment: positive
          Confidence scores: { positive: 0.84, neutral: 0, negative: 0.16 }
          Mined opinions
                - Target text: food
                  Target sentiment: negative
                  Target confidence scores: { positive: 0.01, negative: 0.99 }
                  Target assessments
                        - Text: unacceptable
                          Sentiment: negative
                - Target text: service
                  Target sentiment: negative
                  Target confidence scores: { positive: 0.01, negative: 0.99 }
                  Target assessments
                        - Text: unacceptable
                          Sentiment: negative
                - Target text: concierge
                  Target sentiment: positive
                  Target confidence scores: { positive: 1, negative: 0 }
                  Target assessments
                        - Text: nice
                          Sentiment: positive
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Создайте массив строк, содержащий документ, который следует проанализировать. Вызовите метод клиента `analyzeSentiment()` и получите возвращенный объект `SentimentBatchResult`. Выполните итерацию результатов и выведите идентификатор каждого документа, тональность на уровне документа со степенями уверенности. Результат содержит тональность на уровне предложений, а также смещения, длину и показатели уверенности для каждого документа.

```javascript
async function sentimentAnalysis(client){

    const sentimentInput = [
        "I had the best day of my life. I wish you were there with me."
    ];
    const sentimentResult = await client.analyzeSentiment(sentimentInput);

    sentimentResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tDocument Sentiment: ${document.sentiment}`);
        console.log(`\tDocument Scores:`);
        console.log(`\t\tPositive: ${document.confidenceScores.positive.toFixed(2)} \tNegative: ${document.confidenceScores.negative.toFixed(2)} \tNeutral: ${document.confidenceScores.neutral.toFixed(2)}`);
        console.log(`\tSentences Sentiment(${document.sentences.length}):`);
        document.sentences.forEach(sentence => {
            console.log(`\t\tSentence sentiment: ${sentence.sentiment}`)
            console.log(`\t\tSentences Scores:`);
            console.log(`\t\tPositive: ${sentence.confidenceScores.positive.toFixed(2)} \tNegative: ${sentence.confidenceScores.negative.toFixed(2)} \tNeutral: ${sentence.confidenceScores.neutral.toFixed(2)}`);
        });
    });
}
sentimentAnalysis(textAnalyticsClient)
```

Выполните код с помощью `node index.js` в окне консоли.

### <a name="output"></a>Выходные данные

```console
ID: 0
        Document Sentiment: positive
        Document Scores:
                Positive: 1.00  Negative: 0.00  Neutral: 0.00
        Sentences Sentiment(2):
                Sentence sentiment: positive
                Sentences Scores:
                Positive: 1.00  Negative: 0.00  Neutral: 0.00
                Sentence sentiment: neutral
                Sentences Scores:
                Positive: 0.21  Negative: 0.02  Neutral: 0.77
```

---

## <a name="language-detection"></a>Определение языка

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Создайте массив строк, содержащий документ, который следует проанализировать. Вызовите метод клиента `detectLanguage()` и получите возвращенный `DetectLanguageResultCollection`. Затем выполните итерацию результатов и выведите идентификатор каждого документа с указанием соответствующего основного языка.

```javascript
async function languageDetection(client) {

    const languageInputArray = [
        "Ce document est rédigé en Français."
    ];
    const languageResult = await client.detectLanguage(languageInputArray);

    languageResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tPrimary Language ${document.primaryLanguage.name}`)
    });
}
languageDetection(textAnalyticsClient);
```

Выполните код с помощью `node index.js` в окне консоли.

### <a name="output"></a>Выходные данные

```console
ID: 0
        Primary Language French
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Создайте массив строк, содержащий документ, который следует проанализировать. Вызовите метод клиента `detectLanguage()` и получите возвращенный `DetectLanguageResultCollection`. Затем выполните итерацию результатов и выведите идентификатор каждого документа с указанием соответствующего основного языка.

```javascript
async function languageDetection(client) {

    const languageInputArray = [
        "Ce document est rédigé en Français."
    ];
    const languageResult = await client.detectLanguage(languageInputArray);

    languageResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tPrimary Language ${document.primaryLanguage.name}`)
    });
}
languageDetection(textAnalyticsClient);
```

Выполните код с помощью `node index.js` в окне консоли.

### <a name="output"></a>Выходные данные

```console
ID: 0
        Primary Language French
```

---

## <a name="named-entity-recognition-ner"></a>Распознавание именованных сущностей (NER)

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

> [!NOTE]
> В версии `3.1`:
> * Связывание сущностей — это отдельный запрос, отличный от NER.

Создайте массив строк, содержащий документ, который следует проанализировать. Вызовите метод клиента `recognizeEntities()` и получите объект `RecognizeEntitiesResult`. Выполните итерацию по списку результатов и выведите имя сущности, тип, подтип, смещение, длину и оценку.

```javascript
async function entityRecognition(client){

    const entityInputs = [
        "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, to develop and sell BASIC interpreters for the Altair 8800",
        "La sede principal de Microsoft se encuentra en la ciudad de Redmond, a 21 kilómetros de Seattle."
    ];
    const entityResults = await client.recognizeEntities(entityInputs);

    entityResults.forEach(document => {
        console.log(`Document ID: ${document.id}`);
        document.entities.forEach(entity => {
            console.log(`\tName: ${entity.text} \tCategory: ${entity.category} \tSubcategory: ${entity.subCategory ? entity.subCategory : "N/A"}`);
            console.log(`\tScore: ${entity.confidenceScore}`);
        });
    });
}
entityRecognition(textAnalyticsClient);
```

Выполните код с помощью `node index.js` в окне консоли.

### <a name="output"></a>Выходные данные

```console
Document ID: 0
        Name: Microsoft         Category: Organization  Subcategory: N/A
        Score: 0.29
        Name: Bill Gates        Category: Person        Subcategory: N/A
        Score: 0.78
        Name: Paul Allen        Category: Person        Subcategory: N/A
        Score: 0.82
        Name: April 4, 1975     Category: DateTime      Subcategory: Date
        Score: 0.8
        Name: 8800      Category: Quantity      Subcategory: Number
        Score: 0.8
Document ID: 1
        Name: 21        Category: Quantity      Subcategory: Number
        Score: 0.8
        Name: Seattle   Category: Location      Subcategory: GPE
        Score: 0.25
```

### <a name="entity-linking"></a>Связывание сущностей

Создайте массив строк, содержащий документ, который следует проанализировать. Вызовите метод клиента `recognizeLinkedEntities()` и получите объект `RecognizeLinkedEntitiesResult`. Выполните итерацию по списку результатов и выведите имя сущности, идентификатор, источник данных, URL-адрес, совпадения. Каждый объект в массиве `matches` будет содержать смещение, длину и оценку для этого совпадения.

```javascript
async function linkedEntityRecognition(client){

    const linkedEntityInput = [
        "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, to develop and sell BASIC interpreters for the Altair 8800. During his career at Microsoft, Gates held the positions of chairman, chief executive officer, president and chief software architect, while also being the largest individual shareholder until May 2014."
    ];
    const entityResults = await client.recognizeLinkedEntities(linkedEntityInput);

    entityResults.forEach(document => {
        console.log(`Document ID: ${document.id}`);
        document.entities.forEach(entity => {
            console.log(`\tName: ${entity.name} \tID: ${entity.dataSourceEntityId} \tURL: ${entity.url} \tData Source: ${entity.dataSource}`);
            console.log(`\tMatches:`)
            entity.matches.forEach(match => {
                console.log(`\t\tText: ${match.text} \tScore: ${match.confidenceScore.toFixed(2)}`);
        })
        });
    });
}
linkedEntityRecognition(textAnalyticsClient);
```

Выполните код с помощью `node index.js` в окне консоли.

### <a name="output"></a>Выходные данные

```console
Document ID: 0
        Name: Altair 8800       ID: Altair 8800         URL: https://en.wikipedia.org/wiki/Altair_8800  Data Source: Wikipedia
        Matches:
                Text: Altair 8800       Score: 0.88
        Name: Bill Gates        ID: Bill Gates  URL: https://en.wikipedia.org/wiki/Bill_Gates   Data Source: Wikipedia
        Matches:
                Text: Bill Gates        Score: 0.63
                Text: Gates     Score: 0.63
        Name: Paul Allen        ID: Paul Allen  URL: https://en.wikipedia.org/wiki/Paul_Allen   Data Source: Wikipedia
        Matches:
                Text: Paul Allen        Score: 0.60
        Name: Microsoft         ID: Microsoft   URL: https://en.wikipedia.org/wiki/Microsoft    Data Source: Wikipedia
        Matches:
                Text: Microsoft         Score: 0.55
                Text: Microsoft         Score: 0.55
        Name: April 4   ID: April 4     URL: https://en.wikipedia.org/wiki/April_4      Data Source: Wikipedia
        Matches:
                Text: April 4   Score: 0.32
        Name: BASIC     ID: BASIC       URL: https://en.wikipedia.org/wiki/BASIC        Data Source: Wikipedia
        Matches:
                Text: BASIC     Score: 0.33
```

### <a name="personally-identifying-information-pii-recognition"></a>Распознавание личных сведений (PII)

Создайте массив строк, содержащий документ, который следует проанализировать. Вызовите метод клиента `recognizePiiEntities()` и получите объект `RecognizePIIEntitiesResult`. Выполните итерацию по списку результатов и выведите имя сущности, тип и оценку.

```javascript
async function piiRecognition(client) {

    const documents = [
        "The employee's phone number is (555) 555-5555."
    ];

    const results = await client.recognizePiiEntities(documents, "en");
    for (const result of results) {
        if (result.error === undefined) {
            console.log("Redacted Text: ", result.redactedText);
            console.log(" -- Recognized PII entities for input", result.id, "--");
            for (const entity of result.entities) {
                console.log(entity.text, ":", entity.category, "(Score:", entity.confidenceScore, ")");
            }
        } else {
            console.error("Encountered an error:", result.error);
        }
    }
}
piiRecognition(textAnalyticsClient)
```

Выполните код с помощью `node index.js` в окне консоли.

### <a name="output"></a>Выходные данные

```console
Redacted Text:  The employee's phone number is **************.
 -- Recognized PII entities for input 0 --
(555) 555-5555 : Phone Number (Score: 0.8 )
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

> [!NOTE]
> В версии `3.0`:
> * Связывание сущностей — это отдельный запрос, отличный от NER.

Создайте массив строк, содержащий документ, который следует проанализировать. Вызовите метод клиента `recognizeEntities()` и получите объект `RecognizeEntitiesResult`. Выполните итерацию по списку результатов и выведите имя сущности, тип, подтип, смещение, длину и оценку.

```javascript
async function entityRecognition(client){

    const entityInputs = [
        "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, to develop and sell BASIC interpreters for the Altair 8800",
        "La sede principal de Microsoft se encuentra en la ciudad de Redmond, a 21 kilómetros de Seattle."
    ];
    const entityResults = await client.recognizeEntities(entityInputs);

    entityResults.forEach(document => {
        console.log(`Document ID: ${document.id}`);
        document.entities.forEach(entity => {
            console.log(`\tName: ${entity.text} \tCategory: ${entity.category} \tSubcategory: ${entity.subCategory ? entity.subCategory : "N/A"}`);
            console.log(`\tScore: ${entity.confidenceScore}`);
        });
    });
}
entityRecognition(textAnalyticsClient);
```

Выполните код с помощью `node index.js` в окне консоли.

### <a name="output"></a>Выходные данные

```console
Document ID: 0
        Name: Microsoft         Category: Organization  Subcategory: N/A
        Score: 0.29
        Name: Bill Gates        Category: Person        Subcategory: N/A
        Score: 0.78
        Name: Paul Allen        Category: Person        Subcategory: N/A
        Score: 0.82
        Name: April 4, 1975     Category: DateTime      Subcategory: Date
        Score: 0.8
        Name: 8800      Category: Quantity      Subcategory: Number
        Score: 0.8
Document ID: 1
        Name: 21        Category: Quantity      Subcategory: Number
        Score: 0.8
        Name: Seattle   Category: Location      Subcategory: GPE
        Score: 0.25
```

### <a name="entity-linking"></a>Связывание сущностей

Создайте массив строк, содержащий документ, который следует проанализировать. Вызовите метод клиента `recognizeLinkedEntities()` и получите объект `RecognizeLinkedEntitiesResult`. Выполните итерацию по списку результатов и выведите имя сущности, идентификатор, источник данных, URL-адрес, совпадения. Каждый объект в массиве `matches` будет содержать смещение, длину и оценку для этого совпадения.

```javascript
async function linkedEntityRecognition(client){

    const linkedEntityInput = [
        "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, to develop and sell BASIC interpreters for the Altair 8800. During his career at Microsoft, Gates held the positions of chairman, chief executive officer, president and chief software architect, while also being the largest individual shareholder until May 2014."
    ];
    const entityResults = await client.recognizeLinkedEntities(linkedEntityInput);

    entityResults.forEach(document => {
        console.log(`Document ID: ${document.id}`);
        document.entities.forEach(entity => {
            console.log(`\tName: ${entity.name} \tID: ${entity.dataSourceEntityId} \tURL: ${entity.url} \tData Source: ${entity.dataSource}`);
            console.log(`\tMatches:`)
            entity.matches.forEach(match => {
                console.log(`\t\tText: ${match.text} \tScore: ${match.confidenceScore.toFixed(2)}`);
        })
        });
    });
}
linkedEntityRecognition(textAnalyticsClient);
```

Выполните код с помощью `node index.js` в окне консоли.

### <a name="output"></a>Выходные данные

```console
Document ID: 0
        Name: Altair 8800       ID: Altair 8800         URL: https://en.wikipedia.org/wiki/Altair_8800  Data Source: Wikipedia
        Matches:
                Text: Altair 8800       Score: 0.88
        Name: Bill Gates        ID: Bill Gates  URL: https://en.wikipedia.org/wiki/Bill_Gates   Data Source: Wikipedia
        Matches:
                Text: Bill Gates        Score: 0.63
                Text: Gates     Score: 0.63
        Name: Paul Allen        ID: Paul Allen  URL: https://en.wikipedia.org/wiki/Paul_Allen   Data Source: Wikipedia
        Matches:
                Text: Paul Allen        Score: 0.60
        Name: Microsoft         ID: Microsoft   URL: https://en.wikipedia.org/wiki/Microsoft    Data Source: Wikipedia
        Matches:
                Text: Microsoft         Score: 0.55
                Text: Microsoft         Score: 0.55
        Name: April 4   ID: April 4     URL: https://en.wikipedia.org/wiki/April_4      Data Source: Wikipedia
        Matches:
                Text: April 4   Score: 0.32
        Name: BASIC     ID: BASIC       URL: https://en.wikipedia.org/wiki/BASIC        Data Source: Wikipedia
        Matches:
                Text: BASIC     Score: 0.33
```


---

## <a name="key-phrase-extraction"></a>Извлечение ключевой фразы

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Создайте массив строк, содержащий документ, который следует проанализировать. Вызовите метод клиента `extractKeyPhrases()` и получите возвращенный объект `ExtractKeyPhrasesResult`. Выполните итерацию результатов и выведите идентификатор каждого документа и все обнаруженные ключевые фразы.

```javascript
async function keyPhraseExtraction(client){

    const keyPhrasesInput = [
        "My cat might need to see a veterinarian.",
    ];
    const keyPhraseResult = await client.extractKeyPhrases(keyPhrasesInput);
    
    keyPhraseResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tDocument Key Phrases: ${document.keyPhrases}`);
    });
}
keyPhraseExtraction(textAnalyticsClient);
```

Выполните код с помощью `node index.js` в окне консоли.

### <a name="output"></a>Выходные данные

```console
ID: 0
        Document Key Phrases: cat,veterinarian
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Создайте массив строк, содержащий документ, который следует проанализировать. Вызовите метод клиента `extractKeyPhrases()` и получите возвращенный объект `ExtractKeyPhrasesResult`. Выполните итерацию результатов и выведите идентификатор каждого документа и все обнаруженные ключевые фразы.

```javascript
async function keyPhraseExtraction(client){

    const keyPhrasesInput = [
        "My cat might need to see a veterinarian.",
    ];
    const keyPhraseResult = await client.extractKeyPhrases(keyPhrasesInput);
    
    keyPhraseResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tDocument Key Phrases: ${document.keyPhrases}`);
    });
}
keyPhraseExtraction(textAnalyticsClient);
```

Выполните код с помощью `node index.js` в окне консоли.

### <a name="output"></a>Выходные данные

```console
ID: 0
        Document Key Phrases: cat,veterinarian
```


---

## <a name="use-the-api-asynchronously-with-the-analyze-operation"></a>Асинхронное использование API с операцией анализа

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

> [!CAUTION]
> Чтобы использовать операции анализа, вам нужно использовать ресурс Анализа текста ценовой категории "Стандартный" (S).  

Создайте новую функцию с именем `analyze_example()`, которая вызывает функцию `beginAnalyze()`. В результате будет выполняться длительная операция, опрашиваемая на предмет результатов.

```javascript
async function analyze_example(client) {
  const documents = [
    "Microsoft was founded by Bill Gates and Paul Allen.",
  ];

  const actions = {
    recognizeEntitiesActions: [{ modelVersion: "latest" }],
  };
  const poller = await client.beginAnalyzeBatchActions(documents, actions, "en");

  console.log(
    `The analyze batch actions operation was created on ${poller.getOperationState().createdOn}`
  );
  console.log(
    `The analyze batch actions operation results will expire on ${
      poller.getOperationState().expiresOn
    }`
  );
  const resultPages = await poller.pollUntilDone();
  for await (const page of resultPages) {
    const entitiesAction = page.recognizeEntitiesResults[0];
    if (!entitiesAction.error) {
      for (const doc of entitiesAction.results) {
        console.log(`- Document ${doc.id}`);
        if (!doc.error) {
          console.log("\tEntities:");
          for (const entity of doc.entities) {
            console.log(`\t- Entity ${entity.text} of type ${entity.category}`);
          }
        } else {
          console.error("\tError:", doc.error);
        }
      }
    }
  }
}

analyze_example(textAnalyticsClient);
```

### <a name="output"></a>Выходные данные

```console
The analyze batch actions operation was created on Fri Mar 12 2021 09:53:49 GMT-0800 (Pacific Standard Time)
The analyze batch actions operation results will expire on Sat Mar 13 2021 09:53:49 GMT-0800 (Pacific Standard Time)
- Document 0
        Entities:
        - Entity Microsoft of type Organization
        - Entity Bill Gates of type Person
        - Entity Paul Allen of type Person
```

Кроме того, вы можете использовать операцию анализа для обнаружения персональных данных, распознавания связанных сущностей и извлечения ключевых фраз. Примеры анализа для [JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics/samples/javascript) и [TypeScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics/samples/typescript/src) см. на сайте GitHub.

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Эта функция недоступна в версии 3.0.

---

## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение, выполнив команду `node` для файла quickstart.

```console
node index.js
```
