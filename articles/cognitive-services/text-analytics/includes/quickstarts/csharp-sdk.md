---
title: Краткое руководство. Клиентская библиотека Анализа текста для C# | Документация Майкрософт
description: Начало работы с клиентской библиотекой Анализа текста для C#.
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 03/11/2021
ms.author: aahi
ms.reviewer: assafi
ms.openlocfilehash: d36efaf64edd51fd1c2b46b959af1a994f6ec956
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105582735"
---
<a name="HOLTop"></a>

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

[Справочная документация по версии 3.1](/dotnet/api/azure.ai.textanalytics?preserve-view=true&view=azure-dotnet-preview) | [Исходный код библиотеки версии 3.1](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics) | [Пакет для разработки (NuGet) версии 3.1](https://www.nuget.org/packages/Azure.AI.TextAnalytics/5.1.0-beta.5) | [Примеры кода версии 3.1](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics/samples)

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

[Справочная документация версии 3](/dotnet/api/azure.ai.textanalytics) | [Исходный код библиотеки версии 3](https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.0.0/sdk/textanalytics/Azure.AI.TextAnalytics) | [Пакет для разработки (NuGet) версии 3](https://www.nuget.org/packages/Azure.AI.TextAnalytics) | [Образцы кода версии 3](https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.TextAnalytics_5.0.0/sdk/textanalytics/Azure.AI.TextAnalytics/samples)

---

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* [Интегрированная среда разработки Visual Studio](https://visualstudio.microsoft.com/vs/).
* Получив подписку Azure, перейдите к <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics"  title="созданию ресурса Анализа текста"  target="_blank">созданию ресурса Анализа текста</a> на портале Azure, чтобы получить ключ и конечную точку.  После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Анализа текста потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.
* Чтобы использовать функцию анализа, вам потребуется ресурс Анализа текста ценовой категории "Стандартный" (S).

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-net-core-application"></a>Создание приложения .NET Core

С помощью интегрированной среды разработки Visual Studio создайте консольное приложение .NET Core. Будет создан проект Hello World на языке C# с одним файлом исходного кода: *program.cs*.

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Установите клиентскую библиотеку, щелкнув правой кнопкой мыши решение в **обозревателе решений** и выбрав **Управление пакетами NuGet**. В открывшемся диспетчере пакетов выберите **Просмотр** и выполните поиск по запросу `Azure.AI.TextAnalytics`. Установите флажок **Включить предварительные версии**, выберите версию `5.1.0-beta.5` и нажмите кнопку **Установить**. Вы также можете использовать [консоль диспетчера пакетов](/nuget/consume-packages/install-use-packages-powershell#find-and-install-a-package).

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Установите клиентскую библиотеку, щелкнув правой кнопкой мыши решение в **обозревателе решений** и выбрав **Управление пакетами NuGet**. В открывшемся диспетчере пакетов выберите **Просмотр** и выполните поиск по запросу `Azure.AI.TextAnalytics`. Выберите версию `5.0.0`, а затем **Установить**. Вы также можете использовать [консоль диспетчера пакетов](/nuget/consume-packages/install-use-packages-powershell#find-and-install-a-package).


> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/TextAnalytics/program.cs), где размещены примеры кода для этого краткого руководства. 

---

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Откройте файл *program.cs* и добавьте следующие директивы `using`:

```csharp
using Azure;
using System;
using System.Globalization;
using Azure.AI.TextAnalytics;
```

В классе приложения `Program` создайте переменные для ключа и конечной точки вашего ресурса.

[!INCLUDE [text-analytics-find-resource-information](../find-azure-resource-info.md)]

```csharp
private static readonly AzureKeyCredential credentials = new AzureKeyCredential("<replace-with-your-text-analytics-key-here>");
private static readonly Uri endpoint = new Uri("<replace-with-your-text-analytics-endpoint-here>");
```

Замените метод приложения `Main`. Вызванные методы будут определены здесь позже.

```csharp
static void Main(string[] args)
{
    var client = new TextAnalyticsClient(endpoint, credentials);
    // You will implement these methods later in the quickstart.
    SentimentAnalysisExample(client);
    SentimentAnalysisWithOpinionMiningExample(client);
    LanguageDetectionExample(client);
    EntityRecognitionExample(client);
    EntityLinkingExample(client);
    RecognizePIIExample(client);
    KeyPhraseExtractionExample(client);

    Console.Write("Press any key to exit.");
    Console.ReadKey();
}
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Откройте файл *program.cs* и добавьте следующие директивы `using`:

```csharp
using Azure;
using System;
using System.Globalization;
using Azure.AI.TextAnalytics;
```

В классе приложения `Program` создайте переменные для ключа и конечной точки вашего ресурса.

[!INCLUDE [text-analytics-find-resource-information](../find-azure-resource-info.md)]

```csharp
private static readonly AzureKeyCredential credentials = new AzureKeyCredential("<replace-with-your-text-analytics-key-here>");
private static readonly Uri endpoint = new Uri("<replace-with-your-text-analytics-endpoint-here>");
```

Замените метод приложения `Main`. Вызванные методы будут определены здесь позже.

```csharp
static void Main(string[] args)
{
    var client = new TextAnalyticsClient(endpoint, credentials);
    // You will implement these methods later in the quickstart.
    SentimentAnalysisExample(client);
    LanguageDetectionExample(client);
    EntityRecognitionExample(client);
    EntityLinkingExample(client);
    KeyPhraseExtractionExample(client);

    Console.Write("Press any key to exit.");
    Console.ReadKey();
}
```

---

## <a name="object-model"></a>Объектная модель

Клиент Анализа текста является объектом `TextAnalyticsClient`, который выполняет проверку подлинности в Azure с помощью вашего ключа и предоставляет функции для приема текста в виде отдельных строк или пакета. Текст в API можно отправлять как синхронно, так и асинхронно. Объект в ответе будет содержать аналитическую информацию по каждому отправленному вами документу. 

Если вы используете версию `3.x` этой службы, можно использовать необязательный экземпляр `TextAnalyticsClientOptions` для инициализации клиента с разными параметрами по умолчанию (например, языком по умолчанию или подсказкой для страны или региона). Проверку подлинности можно также выполнить с помощью токена Azure Active Directory. 

## <a name="code-examples"></a>Примеры кода

* [Пример. Как определить тональность с помощью Анализа текста](#sentiment-analysis)
* [Интеллектуальный анализ мнений](#opinion-mining)
* [Пример. Как определить язык с помощью Анализа текста](#language-detection)
* [Распознавание именованных сущностей](#named-entity-recognition-ner)
* [Связывание сущностей](#entity-linking)
* [Пример. Как извлечь ключевые фразы с помощью Анализа текста](#key-phrase-extraction)

## <a name="authenticate-the-client"></a>Аутентификация клиента

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Убедитесь, что основной метод из более ранней версии создает новый объект клиента с конечной точкой и учетными данными.

```csharp
var client = new TextAnalyticsClient(endpoint, credentials);
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Убедитесь, что основной метод из более ранней версии создает новый объект клиента с конечной точкой и учетными данными.

```csharp
var client = new TextAnalyticsClient(endpoint, credentials);
```

---

## <a name="sentiment-analysis"></a>Анализ мнений

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Создайте функцию с именем `SentimentAnalysisExample()`, которая принимает созданный ранее клиент, и вызовите вложенную в нее функцию `AnalyzeSentiment()`. Возвращаемый объект `Response<DocumentSentiment>` будет содержать метку тональности и оценку всего входного документа, а также анализ тональности для каждого предложения, если выполнение успешное. При появлении ошибки возникает исключение `RequestFailedException`.

```csharp
static void SentimentAnalysisExample(TextAnalyticsClient client)
{
    string inputText = "I had the best day of my life. I wish you were there with me.";
    DocumentSentiment documentSentiment = client.AnalyzeSentiment(inputText);
    Console.WriteLine($"Document sentiment: {documentSentiment.Sentiment}\n");

    foreach (var sentence in documentSentiment.Sentences)
    {
        Console.WriteLine($"\tText: \"{sentence.Text}\"");
        Console.WriteLine($"\tSentence sentiment: {sentence.Sentiment}");
        Console.WriteLine($"\tPositive score: {sentence.ConfidenceScores.Positive:0.00}");
        Console.WriteLine($"\tNegative score: {sentence.ConfidenceScores.Negative:0.00}");
        Console.WriteLine($"\tNeutral score: {sentence.ConfidenceScores.Neutral:0.00}\n");
    }
}
```

### <a name="output"></a>Выходные данные

```console
Document sentiment: Positive

        Text: "I had the best day of my life."
        Sentence sentiment: Positive
        Positive score: 1.00
        Negative score: 0.00
        Neutral score: 0.00

        Text: "I wish you were there with me."
        Sentence sentiment: Neutral
        Positive score: 0.21
        Negative score: 0.02
        Neutral score: 0.77
```

### <a name="opinion-mining"></a>Интеллектуальный анализ данных

Создайте функцию с именем `SentimentAnalysisWithOpinionMiningExample()`, которая принимает созданный ранее клиент, и вызовите вложенную в нее функцию `AnalyzeSentimentBatch()` с параметром `IncludeOpinionMining` в контейнере `AnalyzeSentimentOptions`. Возвращаемый объект `AnalyzeSentimentResultCollection` будет содержать коллекцию `AnalyzeSentimentResult`, в которой представлен объект `Response<DocumentSentiment>`. Разница между `SentimentAnalysis()` и `SentimentAnalysisWithOpinionMiningExample()` заключается в том, что последний метод будет содержать `SentenceOpinion` в каждом предложении, где показан проанализированный целевой объект и связанные с ним оценки. При появлении ошибки возникает исключение `RequestFailedException`.

```csharp
static void SentimentAnalysisWithOpinionMiningExample(TextAnalyticsClient client)
{
    var documents = new List<string>
    {
        "The food and service were unacceptable, but the concierge were nice."
    };

    AnalyzeSentimentResultCollection reviews = client.AnalyzeSentimentBatch(documents, options: new AnalyzeSentimentOptions()
    {
        IncludeOpinionMining = true
    });

    foreach (AnalyzeSentimentResult review in reviews)
    {
        Console.WriteLine($"Document sentiment: {review.DocumentSentiment.Sentiment}\n");
        Console.WriteLine($"\tPositive score: {review.DocumentSentiment.ConfidenceScores.Positive:0.00}");
        Console.WriteLine($"\tNegative score: {review.DocumentSentiment.ConfidenceScores.Negative:0.00}");
        Console.WriteLine($"\tNeutral score: {review.DocumentSentiment.ConfidenceScores.Neutral:0.00}\n");
        foreach (SentenceSentiment sentence in review.DocumentSentiment.Sentences)
        {
            Console.WriteLine($"\tText: \"{sentence.Text}\"");
            Console.WriteLine($"\tSentence sentiment: {sentence.Sentiment}");
            Console.WriteLine($"\tSentence positive score: {sentence.ConfidenceScores.Positive:0.00}");
            Console.WriteLine($"\tSentence negative score: {sentence.ConfidenceScores.Negative:0.00}");
            Console.WriteLine($"\tSentence neutral score: {sentence.ConfidenceScores.Neutral:0.00}\n");

            foreach (SentenceOpinion sentenceOpinion in sentence.Opinions)
            {
                Console.WriteLine($"\tTarget: {sentenceOpinion.Target.Text}, Value: {sentenceOpinion.Target.Sentiment}");
                Console.WriteLine($"\tTarget positive score: {sentenceOpinion.Target.ConfidenceScores.Positive:0.00}");
                Console.WriteLine($"\tTarget negative score: {sentenceOpinion.Target.ConfidenceScores.Negative:0.00}");
                foreach (AssessmentSentiment assessment in sentenceOpinion.Assessments)
                {
                    Console.WriteLine($"\t\tRelated Assessment: {assessment.Text}, Value: {assessment.Sentiment}");
                    Console.WriteLine($"\t\tRelated Assessment positive score: {assessment.ConfidenceScores.Positive:0.00}");
                    Console.WriteLine($"\t\tRelated Assessment negative score: {assessment.ConfidenceScores.Negative:0.00}");
                }
            }
        }
        Console.WriteLine($"\n");
    }
}
```

### <a name="output"></a>Выходные данные

```console
Document sentiment: Positive

        Positive score: 0.84
        Negative score: 0.16
        Neutral score: 0.00

        Text: "The food and service were unacceptable, but the concierge were nice."
        Sentence sentiment: Positive
        Sentence positive score: 0.84
        Sentence negative score: 0.16
        Sentence neutral score: 0.00

        Target: food, Value: Negative
        Target positive score: 0.01
        Target negative score: 0.99
                Related Assessment: unacceptable, Value: Negative
                Related Assessment positive score: 0.01
                Related Assessment negative score: 0.99
        Target: service, Value: Negative
        Target positive score: 0.01
        Target negative score: 0.99
                Related Assessment: unacceptable, Value: Negative
                Related Assessment positive score: 0.01
                Related Assessment negative score: 0.99
        Target: concierge, Value: Positive
        Target positive score: 1.00
        Target negative score: 0.00
                Related Assessment: nice, Value: Positive
                Related Assessment positive score: 1.00
                Related Assessment negative score: 0.00

Press any key to exit.
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Создайте функцию с именем `SentimentAnalysisExample()`, которая принимает созданный ранее клиент, и вызовите вложенную в нее функцию `AnalyzeSentiment()`. Возвращаемый объект `Response<DocumentSentiment>` будет содержать метку тональности и оценку всего входного документа, а также анализ тональности для каждого предложения, если выполнение успешное. При появлении ошибки возникает исключение `RequestFailedException`.

```csharp
static void SentimentAnalysisExample(TextAnalyticsClient client)
{
    string inputText = "I had the best day of my life. I wish you were there with me.";
    DocumentSentiment documentSentiment = client.AnalyzeSentiment(inputText);
    Console.WriteLine($"Document sentiment: {documentSentiment.Sentiment}\n");

    foreach (var sentence in documentSentiment.Sentences)
    {
        Console.WriteLine($"\tText: \"{sentence.Text}\"");
        Console.WriteLine($"\tSentence sentiment: {sentence.Sentiment}");
        Console.WriteLine($"\tPositive score: {sentence.ConfidenceScores.Positive:0.00}");
        Console.WriteLine($"\tNegative score: {sentence.ConfidenceScores.Negative:0.00}");
        Console.WriteLine($"\tNeutral score: {sentence.ConfidenceScores.Neutral:0.00}\n");
    }
}
```

### <a name="output"></a>Выходные данные

```console
Document sentiment: Positive

        Text: "I had the best day of my life."
        Sentence sentiment: Positive
        Positive score: 1.00
        Negative score: 0.00
        Neutral score: 0.00

        Text: "I wish you were there with me."
        Sentence sentiment: Neutral
        Positive score: 0.21
        Negative score: 0.02
        Neutral score: 0.77
```

---

## <a name="language-detection"></a>Определение языка

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)


Создайте функцию с именем `LanguageDetectionExample()`, которая принимает созданный ранее клиент, и вызовите вложенную в нее функцию `DetectLanguage()`. Возвращенный объект `Response<DetectedLanguage>` будет содержать обнаруженный язык вместе с его именем и кодом ISO-6391. При появлении ошибки возникает исключение `RequestFailedException`.

> [!Tip]
> В некоторых случаях неоднозначность языков на основе входных данных может быть трудно устранить. Используйте параметр `countryHint`, чтобы указать двухбуквенный код страны или региона. По умолчанию API использует US в качестве значения countryHint. Чтобы отменить это поведение, вы можете сбросить этот параметр, установив для этого значения пустую строку `countryHint = ""`. Чтобы установить другое значение по умолчанию, задайте свойство `TextAnalyticsClientOptions.DefaultCountryHint` и передайте его во время инициализации клиента.

```csharp
static void LanguageDetectionExample(TextAnalyticsClient client)
{
    DetectedLanguage detectedLanguage = client.DetectLanguage("Ce document est rédigé en Français.");
    Console.WriteLine("Language:");
    Console.WriteLine($"\t{detectedLanguage.Name},\tISO-6391: {detectedLanguage.Iso6391Name}\n");
}
```

### <a name="output"></a>Выходные данные

```console
Language:
        French, ISO-6391: fr
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)


Создайте функцию с именем `LanguageDetectionExample()`, которая принимает созданный ранее клиент, и вызовите вложенную в нее функцию `DetectLanguage()`. Возвращенный объект `Response<DetectedLanguage>` будет содержать обнаруженный язык вместе с его именем и кодом ISO-6391. При появлении ошибки возникает исключение `RequestFailedException`.

> [!Tip]
> В некоторых случаях неоднозначность языков на основе входных данных может быть трудно устранить. Используйте параметр `countryHint`, чтобы указать двухбуквенный код страны или региона. По умолчанию API использует US в качестве значения countryHint. Чтобы отменить это поведение, вы можете сбросить этот параметр, установив для этого значения пустую строку `countryHint = ""`. Чтобы установить другое значение по умолчанию, задайте свойство `TextAnalyticsClientOptions.DefaultCountryHint` и передайте его во время инициализации клиента.

```csharp
static void LanguageDetectionExample(TextAnalyticsClient client)
{
    DetectedLanguage detectedLanguage = client.DetectLanguage("Ce document est rédigé en Français.");
    Console.WriteLine("Language:");
    Console.WriteLine($"\t{detectedLanguage.Name},\tISO-6391: {detectedLanguage.Iso6391Name}\n");
}
```

### <a name="output"></a>Выходные данные

```console
Language:
        French, ISO-6391: fr
```


---

## <a name="named-entity-recognition-ner"></a>Распознавание именованных сущностей (NER)

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)


Создайте функцию с именем `EntityRecognitionExample()`, которая принимает созданный ранее клиент, вызовите вложенную в нее функцию `RecognizeEntities()` и просмотрите все результаты. Возвращенный объект `Response<CategorizedEntityCollection>` будет содержать коллекцию обнаруженных сущностей `CategorizedEntity`. При появлении ошибки возникает исключение `RequestFailedException`.

```csharp
static void EntityRecognitionExample(TextAnalyticsClient client)
{
    var response = client.RecognizeEntities("I had a wonderful trip to Seattle last week.");
    Console.WriteLine("Named Entities:");
    foreach (var entity in response.Value)
    {
        Console.WriteLine($"\tText: {entity.Text},\tCategory: {entity.Category},\tSub-Category: {entity.SubCategory}");
        Console.WriteLine($"\t\tScore: {entity.ConfidenceScore:F2},\tLength: {entity.Length},\tOffset: {entity.Offset}\n");
    }
}
```

### <a name="output"></a>Выходные данные

```console
Named Entities:
        Text: trip,     Category: Event,        Sub-Category:
                Score: 0.61,    Length: 4,      Offset: 18

        Text: Seattle,  Category: Location,     Sub-Category: GPE
                Score: 0.82,    Length: 7,      Offset: 26

        Text: last week,        Category: DateTime,     Sub-Category: DateRange
                Score: 0.80,    Length: 9,      Offset: 34
```

### <a name="entity-linking"></a>Связывание сущностей

Создайте функцию с именем `EntityLinkingExample()`, которая принимает созданный ранее клиент, вызовите вложенную в нее функцию `RecognizeLinkedEntities()` и просмотрите все результаты. Возвращенный объект `Response<LinkedEntityCollection>` будет содержать коллекцию обнаруженных сущностей `LinkedEntity`. При появлении ошибки возникает исключение `RequestFailedException`. Так как связанные сущности уникально идентифицируются, вхождения одной и той же сущности группируются в объекте `LinkedEntity` как список объектов `LinkedEntityMatch`.

```csharp
static void EntityLinkingExample(TextAnalyticsClient client)
{
    var response = client.RecognizeLinkedEntities(
        "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, " +
        "to develop and sell BASIC interpreters for the Altair 8800. " +
        "During his career at Microsoft, Gates held the positions of chairman, " +
        "chief executive officer, president and chief software architect, " +
        "while also being the largest individual shareholder until May 2014.");
    Console.WriteLine("Linked Entities:");
    foreach (var entity in response.Value)
    {
        Console.WriteLine($"\tName: {entity.Name},\tID: {entity.DataSourceEntityId},\tURL: {entity.Url}\tData Source: {entity.DataSource}");
        Console.WriteLine("\tMatches:");
        foreach (var match in entity.Matches)
        {
            Console.WriteLine($"\t\tText: {match.Text}");
            Console.WriteLine($"\t\tScore: {match.ConfidenceScore:F2}");
            Console.WriteLine($"\t\tLength: {match.Length}");
            Console.WriteLine($"\t\tOffset: {match.Offset}\n");
        }
    }
}
```

### <a name="output"></a>Выходные данные

```console
Linked Entities:
        Name: Microsoft,        ID: Microsoft,  URL: https://en.wikipedia.org/wiki/Microsoft    Data Source: Wikipedia
        Matches:
                Text: Microsoft
                Score: 0.55
                Length: 9
                Offset: 0

                Text: Microsoft
                Score: 0.55
                Length: 9
                Offset: 150

        Name: Bill Gates,       ID: Bill Gates, URL: https://en.wikipedia.org/wiki/Bill_Gates   Data Source: Wikipedia
        Matches:
                Text: Bill Gates
                Score: 0.63
                Length: 10
                Offset: 25

                Text: Gates
                Score: 0.63
                Length: 5
                Offset: 161

        Name: Paul Allen,       ID: Paul Allen, URL: https://en.wikipedia.org/wiki/Paul_Allen   Data Source: Wikipedia
        Matches:
                Text: Paul Allen
                Score: 0.60
                Length: 10
                Offset: 40

        Name: April 4,  ID: April 4,    URL: https://en.wikipedia.org/wiki/April_4      Data Source: Wikipedia
        Matches:
                Text: April 4
                Score: 0.32
                Length: 7
                Offset: 54

        Name: BASIC,    ID: BASIC,      URL: https://en.wikipedia.org/wiki/BASIC        Data Source: Wikipedia
        Matches:
                Text: BASIC
                Score: 0.33
                Length: 5
                Offset: 89

        Name: Altair 8800,      ID: Altair 8800,        URL: https://en.wikipedia.org/wiki/Altair_8800  Data Source: Wikipedia
        Matches:
                Text: Altair 8800
                Score: 0.88
                Length: 11
                Offset: 116
```

### <a name="personally-identifiable-information-recognition"></a>Распознавание личных сведений

Создайте функцию с именем `RecognizePIIExample()`, которая принимает созданный ранее клиент, вызовите вложенную в нее функцию `RecognizePiiEntities()` и просмотрите все результаты. Возвращаемый объект `PiiEntityCollection` представляет список обнаруженных сущностей личных сведений. При появлении ошибки возникает исключение `RequestFailedException`.

```csharp
static void RecognizePIIExample(TextAnalyticsClient client)
{
    string document = "A developer with SSN 859-98-0987 whose phone number is 800-102-1100 is building tools with our APIs.";

    PiiEntityCollection entities = client.RecognizePiiEntities(document).Value;

    Console.WriteLine($"Redacted Text: {entities.RedactedText}");
    if (entities.Count > 0)
    {
        Console.WriteLine($"Recognized {entities.Count} PII entit{(entities.Count > 1 ? "ies" : "y")}:");
        foreach (PiiEntity entity in entities)
        {
            Console.WriteLine($"Text: {entity.Text}, Category: {entity.Category}, SubCategory: {entity.SubCategory}, Confidence score: {entity.ConfidenceScore}");
        }
    }
    else
    {
        Console.WriteLine("No entities were found.");
    }
}
```

### <a name="output"></a>Выходные данные

```console
Redacted Text: A developer with SSN *********** whose phone number is ************ is building tools with our APIs.
Recognized 2 PII entities:
Text: 859-98-0987, Category: U.S. Social Security Number (SSN), SubCategory: , Confidence score: 0.65
Text: 800-102-1100, Category: Phone Number, SubCategory: , Confidence score: 0.8
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)


> [!NOTE]
> Новое в версии `3.0`.
> * Связывание сущностей теперь отделено от их распознавания.


Создайте функцию с именем `EntityRecognitionExample()`, которая принимает созданный ранее клиент, вызовите вложенную в нее функцию `RecognizeEntities()` и просмотрите все результаты. Возвращенный объект `Response<IReadOnlyCollection<CategorizedEntity>>` будет содержать список обнаруженных сущностей. При появлении ошибки возникает исключение `RequestFailedException`.

```csharp
static void EntityRecognitionExample(TextAnalyticsClient client)
{
    var response = client.RecognizeEntities("I had a wonderful trip to Seattle last week.");
    Console.WriteLine("Named Entities:");
    foreach (var entity in response.Value)
    {
        Console.WriteLine($"\tText: {entity.Text},\tCategory: {entity.Category},\tSub-Category: {entity.SubCategory}");
        Console.WriteLine($"\t\tScore: {entity.ConfidenceScore:F2}\n");
    }
}
```

### <a name="output"></a>Выходные данные

```console
Named Entities:
        Text: trip,     Category: Event,        Sub-Category:
                Score: 0.61

        Text: Seattle,  Category: Location,     Sub-Category: GPE
                Score: 0.82

        Text: last week,        Category: DateTime,     Sub-Category: DateRange
                Score: 0.80
```

### <a name="entity-linking"></a>Связывание сущностей

Создайте функцию с именем `EntityLinkingExample()`, которая принимает созданный ранее клиент, вызовите вложенную в нее функцию `RecognizeLinkedEntities()` и просмотрите все результаты. Возвращаемый `Response<IReadOnlyCollection<LinkedEntity>>` представляет список обнаруженных сущностей. При появлении ошибки возникает исключение `RequestFailedException`. Так как связанные сущности уникально идентифицируются, вхождения одной и той же сущности группируются в объекте `LinkedEntity` как список объектов `LinkedEntityMatch`.

```csharp
static void EntityLinkingExample(TextAnalyticsClient client)
{
    var response = client.RecognizeLinkedEntities(
        "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, " +
        "to develop and sell BASIC interpreters for the Altair 8800. " +
        "During his career at Microsoft, Gates held the positions of chairman, " +
        "chief executive officer, president and chief software architect, " +
        "while also being the largest individual shareholder until May 2014.");
    Console.WriteLine("Linked Entities:");
    foreach (var entity in response.Value)
    {
        Console.WriteLine($"\tName: {entity.Name},\tID: {entity.DataSourceEntityId},\tURL: {entity.Url}\tData Source: {entity.DataSource}");
        Console.WriteLine("\tMatches:");
        foreach (var match in entity.Matches)
        {
            Console.WriteLine($"\t\tText: {match.Text}");
            Console.WriteLine($"\t\tScore: {match.ConfidenceScore:F2}\n");
        }
    }
}
```

### <a name="output"></a>Выходные данные

```console
Linked Entities:
        Name: Altair 8800,      ID: Altair 8800,        URL: https://en.wikipedia.org/wiki/Altair_8800  Data Source: Wikipedia
        Matches:
                Text: Altair 8800
                Score: 0.88

        Name: Bill Gates,       ID: Bill Gates, URL: https://en.wikipedia.org/wiki/Bill_Gates   Data Source: Wikipedia
        Matches:
                Text: Bill Gates
                Score: 0.63

                Text: Gates
                Score: 0.63

        Name: Paul Allen,       ID: Paul Allen, URL: https://en.wikipedia.org/wiki/Paul_Allen   Data Source: Wikipedia
        Matches:
                Text: Paul Allen
                Score: 0.60

        Name: Microsoft,        ID: Microsoft,  URL: https://en.wikipedia.org/wiki/Microsoft    Data Source: Wikipedia
        Matches:
                Text: Microsoft
                Score: 0.55

                Text: Microsoft
                Score: 0.55

        Name: April 4,  ID: April 4,    URL: https://en.wikipedia.org/wiki/April_4      Data Source: Wikipedia
        Matches:
                Text: April 4
                Score: 0.32

        Name: BASIC,    ID: BASIC,      URL: https://en.wikipedia.org/wiki/BASIC        Data Source: Wikipedia
        Matches:
                Text: BASIC
                Score: 0.33
```

--- 


### <a name="key-phrase-extraction"></a>Извлечение ключевой фразы

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Создайте функцию с именем `KeyPhraseExtractionExample()`, которая принимает созданный ранее клиент, и вызовите вложенную в нее функцию `ExtractKeyPhrases()`. Возвращенный объект `<Response<KeyPhraseCollection>` будет содержать список обнаруженных ключевых фраз. При появлении ошибки возникает исключение `RequestFailedException`.

```csharp
static void KeyPhraseExtractionExample(TextAnalyticsClient client)
{
    var response = client.ExtractKeyPhrases("My cat might need to see a veterinarian.");

    // Printing key phrases
    Console.WriteLine("Key phrases:");

    foreach (string keyphrase in response.Value)
    {
        Console.WriteLine($"\t{keyphrase}");
    }
}
```

### <a name="output"></a>Выходные данные

```console
Key phrases:
    cat
    veterinarian
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Создайте функцию с именем `KeyPhraseExtractionExample()`, которая принимает созданный ранее клиент, и вызовите вложенную в нее функцию `ExtractKeyPhrases()`. Возвращенный объект `<Response<IReadOnlyCollection<string>>` будет содержать список обнаруженных ключевых фраз. При появлении ошибки возникает исключение `RequestFailedException`.

```csharp
static void KeyPhraseExtractionExample(TextAnalyticsClient client)
{
    var response = client.ExtractKeyPhrases("My cat might need to see a veterinarian.");

    // Printing key phrases
    Console.WriteLine("Key phrases:");

    foreach (string keyphrase in response.Value)
    {
        Console.WriteLine($"\t{keyphrase}");
    }
}
```

### <a name="output"></a>Выходные данные

```console
Key phrases:
    cat
    veterinarian
```

---

## <a name="use-the-api-asynchronously-with-the-analyze-operation"></a>Асинхронное использование API с операцией анализа

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

[!INCLUDE [Analyze operation pricing](../analyze-operation-pricing-caution.md)]

Создайте функцию с именем `AnalyzeOperationExample()`, которая принимает созданный ранее клиент, и вызовите вложенную в нее функцию `StartAnalyzeBatchActionsAsync()`. Возвращаемый объект `AnalyzeBatchActionsOperation` будет содержать объект интерфейса `Operation`. Так как это длительная операция, для `operation.WaitForCompletionAsync()` нужно использовать `await`, чтобы значение обновилось. По завершении `WaitForCompletionAsync()` коллекция должна быть обновлена в `operation.Value`. При появлении ошибки возникает исключение `RequestFailedException`.


```csharp
static async Task AnalyzeOperationExample(TextAnalyticsClient client)
{
    string inputText = "Microsoft was founded by Bill Gates and Paul Allen.";

    var batchDocuments = new List<string> { inputText };


    TextAnalyticsActions actions = new TextAnalyticsActions()
    {
        RecognizeEntitiesOptions = new List<RecognizeEntitiesOptions>() { new RecognizeEntitiesOptions() },
        DisplayName = "Analyze Operation Quick Start Example"
    };

    AnalyzeBatchActionsOperation operation = await client.StartAnalyzeBatchActionsAsync(batchDocuments, actions);

    await operation.WaitForCompletionAsync();

    Console.WriteLine($"Status: {operation.Status}");
    Console.WriteLine($"Created On: {operation.CreatedOn}");
    Console.WriteLine($"Expires On: {operation.ExpiresOn}");
    Console.WriteLine($"Last modified: {operation.LastModified}");
    if (!string.IsNullOrEmpty(operation.DisplayName))
        Console.WriteLine($"Display name: {operation.DisplayName}");
    Console.WriteLine($"Total actions: {operation.TotalActions}");
    Console.WriteLine($"  Succeeded actions: {operation.ActionsSucceeded}");
    Console.WriteLine($"  Failed actions: {operation.ActionsFailed}");
    Console.WriteLine($"  In progress actions: {operation.ActionsInProgress}");

    await foreach (AnalyzeBatchActionsResult documentsInPage in operation.Value)
    {
        RecognizeEntitiesResultCollection entitiesResult = documentsInPage.RecognizeEntitiesActionsResults.FirstOrDefault().Result;

        Console.WriteLine("Recognized Entities");

        foreach (RecognizeEntitiesResult result in entitiesResult)
        {
            Console.WriteLine($"  Recognized the following {result.Entities.Count} entities:");

            foreach (CategorizedEntity entity in result.Entities)
            {
                Console.WriteLine($"  Entity: {entity.Text}");
                Console.WriteLine($"  Category: {entity.Category}");
                Console.WriteLine($"  Offset: {entity.Offset}");
                Console.WriteLine($"  Length: {entity.Length}");
                Console.WriteLine($"  ConfidenceScore: {entity.ConfidenceScore}");
                Console.WriteLine($"  SubCategory: {entity.SubCategory}");
            }
            Console.WriteLine("");
        }
    }
}
```

После добавления этого примера в приложение вызовите метод `main()` с помощью `await`.

```csharp
await AnalyzeOperationExample(client).ConfigureAwait(false);
```
### <a name="output"></a>Выходные данные

```console
Status: succeeded
Created On: 3/10/2021 2:25:01 AM +00:00
Expires On: 3/11/2021 2:25:01 AM +00:00
Last modified: 3/10/2021 2:25:05 AM +00:00
Display name: Analyze Operation Quick Start Example
Total actions: 1
  Succeeded actions: 1
  Failed actions: 0
  In progress actions: 0
Recognized Entities
    Recognized the following 3 entities:
    Entity: Microsoft
    Category: Organization
    Offset: 0
    ConfidenceScore: 0.83
    SubCategory: 
    Entity: Bill Gates
    Category: Person
    Offset: 25
    ConfidenceScore: 0.85
    SubCategory: 
    Entity: Paul Allen
    Category: Person
    Offset: 40
    ConfidenceScore: 0.9
    SubCategory: 
```

Кроме того, вы можете использовать операцию анализа для обнаружения персональных данных и извлечения ключевых фраз. [Пример анализа](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics/samples) см. на сайте GitHub.

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Эта функция недоступна в версии 3.0.

---
