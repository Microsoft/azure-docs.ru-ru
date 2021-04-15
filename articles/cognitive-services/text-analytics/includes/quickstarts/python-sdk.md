---
author: aahill
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 02/09/2021
ms.author: aahi
ms.openlocfilehash: 791591f3d98f9e6902e89a880c464e6a609e3a1f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104599080"
---
<a name="HOLTop"></a>

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

[Справочная документация по версии 3.1](/python/api/azure-ai-textanalytics/azure.ai.textanalytics) | [Исходный код библиотеки версии 3.1](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics) | [Пакет для разработки (PiPy) версии 3.1](https://pypi.org/project/azure-ai-textanalytics/) | [Образцы кода версии 3.1](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/samples)

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

[Справочная документация по версии 3](/python/api/azure-ai-textanalytics/azure.ai.textanalytics) | [Исходный код библиотеки версии 3](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics) | [Пакет для разработки (PiPy) версии 3](https://pypi.org/project/azure-ai-textanalytics/) | [Образцы кода версии 3](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/samples)

---

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* [Python 3.x](https://www.python.org/)
* Получив подписку Azure, перейдите к <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics"  title="созданию ресурса Анализа текста"  target="_blank">созданию ресурса Анализа текста</a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Анализа текста потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.
* Чтобы использовать функцию анализа, вам потребуется ресурс Анализа текста ценовой категории "Стандартный" (S).

## <a name="setting-up"></a>Настройка

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

После установки Python вы можете установить клиентскую библиотеку с помощью следующей команды:

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

```console
pip install azure-ai-textanalytics --pre
```

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/TextAnalytics/python-v3-client-library.py), где размещены примеры кода для этого краткого руководства. 

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

```console
pip install --upgrade azure-ai-textanalytics
```

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/TextAnalytics/python-v3-client-library.py), где размещены примеры кода для этого краткого руководства. 


---

### <a name="create-a-new-python-application"></a>Создание приложения Python

Создайте файл Python и переменные для конечной точки ресурса Azure и ключа подписки.

[!INCLUDE [text-analytics-find-resource-information](../find-azure-resource-info.md)]

```python
key = "<paste-your-text-analytics-key-here>"
endpoint = "<paste-your-text-analytics-endpoint-here>"
```


## <a name="object-model"></a>Объектная модель

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Клиент Анализа текста представляет собой объект `TextAnalyticsClient`, который выполняет проверку подлинности в Azure. Этот клиент предоставляет несколько методов для анализа текста. 

При обработке текст отправляется в API в виде списка `documents`, который может быть либо списком строк, либо списком представления, подобного словарю, либо списком `TextDocumentInput/DetectLanguageInput`. Объект `dict-like` содержит комбинацию `id`, `text` и `language/country_hint`. Атрибут `text` содержит текст для анализа в источнике `country_hint`, а `id` может иметь любое значение. 

Объект в ответе — это список, содержащий аналитику по каждому документу. 

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Клиент Анализа текста представляет собой объект `TextAnalyticsClient`, который выполняет проверку подлинности в Azure с использованием ключа. Этот клиент предоставляет несколько методов для анализа текста по пакетам. 

Во время пакетной обработки текст отправляется в API в формате списка `documents` объектов `dictionary`, содержащих комбинации атрибутов `id`, `text` и `language` в зависимости от используемого метода. Атрибут `text` содержит текст для анализа в источнике `language`, а `id` может иметь любое значение. 

Объект в ответе — это список, содержащий анализ данных по каждому документу. 

---

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие задачи с клиентской библиотекой Анализа текста для Python:

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

* [аутентификация клиента](#authenticate-the-client);
* [Анализ тональности](#sentiment-analysis).
* [Интеллектуальный анализ мнений](#opinion-mining)
* [Пример. Как определить язык с помощью Анализа текста](#language-detection)
* [Распознавание именованных сущностей](#named-entity-recognition-ner) 
* [Распознавание личных сведений](#personally-identifiable-information-recognition) 
* [Связывание сущностей](#entity-linking)
* [Пример. Как извлечь ключевые фразы с помощью Анализа текста](#key-phrase-extraction)


# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

* [аутентификация клиента](#authenticate-the-client);
* [Анализ тональности](#sentiment-analysis).
* [Пример. Как определить язык с помощью Анализа текста](#language-detection)
* [Распознавание именованных сущностей](#named-entity-recognition-ner) 
* [Связывание сущностей](#entity-linking)
* [Пример. Как извлечь ключевые фразы с помощью Анализа текста](#key-phrase-extraction)

---

## <a name="authenticate-the-client"></a>Аутентификация клиента

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Создайте функцию для создания экземпляра объекта `TextAnalyticsClient` с использованием указанных ранее значений `key` и `endpoint`. Потом создайте класс. 

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

def authenticate_client():
    ta_credential = AzureKeyCredential(key)
    text_analytics_client = TextAnalyticsClient(
            endpoint=endpoint, 
            credential=ta_credential)
    return text_analytics_client

client = authenticate_client()
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Создайте функцию для создания экземпляра объекта `TextAnalyticsClient` с использованием указанных ранее значений `key` и `endpoint`. Потом создайте класс. Обратите внимание, что для использования версии 3.0 должен быть определен объект `api_version=TextAnalyticsApiVersion.V3_0`.

```python
# use this code if you're using SDK version is 5.0.0
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

def authenticate_client():
    ta_credential = AzureKeyCredential(key)
    text_analytics_client = TextAnalyticsClient(
            endpoint=endpoint, 
            credential=ta_credential) 
    return text_analytics_client

client = authenticate_client()
```

Если вы установили клиентскую библиотеку версии 5.1.0 с помощью `pip install azure-ai-textanalytics --pre`, вы можете указать версию 3.0 для API Анализа текста с параметром `api_version` клиента. Используйте следующий метод `authenticate_client()` только в том случае, если клиент имеет версию 5.1.0 или более позднюю.

```python
# Only use the following code sample if you're using v5.1.0 of the client library, 
# and are looking to specify v3.0 of the Text Analytics API for your client
from azure.ai.textanalytics import TextAnalyticsClient, TextAnalyticsApiVersion
from azure.core.credentials import AzureKeyCredential
def authenticate_client():
   ta_credential = AzureKeyCredential(key)
   text_analytics_client = TextAnalyticsClient(
     endpoint=endpoint,
     credential=ta_credential,
     api_version=TextAnalyticsApiVersion.V3_0
   )
   
client = authenticate_client()
```

--- 

## <a name="sentiment-analysis"></a>Анализ мнений

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Создайте функцию с именем `sentiment_analysis_example()`, которая принимает клиент в качестве аргумента, а затем вызывает функцию `analyze_sentiment()`. Возвращаемый объект ответа будет содержать метку тональности и оценку всего входного документа, а также анализ тональности для каждого предложения.


```python
def sentiment_analysis_example(client):

    documents = ["I had the best day of my life. I wish you were there with me."]
    response = client.analyze_sentiment(documents=documents)[0]
    print("Document Sentiment: {}".format(response.sentiment))
    print("Overall scores: positive={0:.2f}; neutral={1:.2f}; negative={2:.2f} \n".format(
        response.confidence_scores.positive,
        response.confidence_scores.neutral,
        response.confidence_scores.negative,
    ))
    for idx, sentence in enumerate(response.sentences):
        print("Sentence: {}".format(sentence.text))
        print("Sentence {} sentiment: {}".format(idx+1, sentence.sentiment))
        print("Sentence score:\nPositive={0:.2f}\nNeutral={1:.2f}\nNegative={2:.2f}\n".format(
            sentence.confidence_scores.positive,
            sentence.confidence_scores.neutral,
            sentence.confidence_scores.negative,
        ))
          
sentiment_analysis_example(client)
```

### <a name="output"></a>Выходные данные

```console
Document Sentiment: positive
Overall scores: positive=1.00; neutral=0.00; negative=0.00 

Sentence: I had the best day of my life.
Sentence 1 sentiment: positive
Sentence score:
Positive=1.00
Neutral=0.00
Negative=0.00

Sentence: I wish you were there with me.
Sentence 2 sentiment: neutral
Sentence score:
Positive=0.21
Neutral=0.77
Negative=0.02
```

## <a name="opinion-mining"></a>Интеллектуальный анализ данных

Чтобы провести анализ тональности с помощью интеллектуального анализа мнений, создайте функцию с именем `sentiment_analysis_with_opinion_mining_example()`, которая принимает клиент в качестве аргумента, а затем вызывает функцию `analyze_sentiment()` с флагом параметра `show_opinion_mining=True`. Возвращаемый объект ответа будет содержать не только метку тональности и оценку всего входного документа с анализом тональности каждого изречения, но также анализ тональности на уровне взглядов и мнений.


```python
def sentiment_analysis_with_opinion_mining_example(client):

    documents = [
        "The food and service were unacceptable, but the concierge were nice"
    ]

    result = client.analyze_sentiment(documents, show_opinion_mining=True)
    doc_result = [doc for doc in result if not doc.is_error]

    positive_reviews = [doc for doc in doc_result if doc.sentiment == "positive"]
    negative_reviews = [doc for doc in doc_result if doc.sentiment == "negative"]

    positive_mined_opinions = []
    mixed_mined_opinions = []
    negative_mined_opinions = []

    for document in doc_result:
        print("Document Sentiment: {}".format(document.sentiment))
        print("Overall scores: positive={0:.2f}; neutral={1:.2f}; negative={2:.2f} \n".format(
            document.confidence_scores.positive,
            document.confidence_scores.neutral,
            document.confidence_scores.negative,
        ))
        for sentence in document.sentences:
            print("Sentence: {}".format(sentence.text))
            print("Sentence sentiment: {}".format(sentence.sentiment))
            print("Sentence score:\nPositive={0:.2f}\nNeutral={1:.2f}\nNegative={2:.2f}\n".format(
                sentence.confidence_scores.positive,
                sentence.confidence_scores.neutral,
                sentence.confidence_scores.negative,
            ))
            for mined_opinion in sentence.mined_opinions:
                target = mined_opinion.target
                print("......'{}' target '{}'".format(target.sentiment, target.text))
                print("......Target score:\n......Positive={0:.2f}\n......Negative={1:.2f}\n".format(
                    target.confidence_scores.positive,
                    target.confidence_scores.negative,
                ))
                for assessment in mined_opinion.assessments:
                    print("......'{}' assessment '{}'".format(assessment.sentiment, assessment.text))
                    print("......Assessment score:\n......Positive={0:.2f}\n......Negative={1:.2f}\n".format(
                        assessment.confidence_scores.positive,
                        assessment.confidence_scores.negative,
                    ))
            print("\n")
        print("\n")
          
sentiment_analysis_with_opinion_mining_example(client)
```

### <a name="output"></a>Выходные данные

```console
Document Sentiment: positive
Overall scores: positive=0.84; neutral=0.00; negative=0.16

Sentence: The food and service were unacceptable, but the concierge were nice
Sentence sentiment: positive
Sentence score:
Positive=0.84
Neutral=0.00
Negative=0.16

......'negative' target 'food'
......Target score:
......Positive=0.01
......Negative=0.99

......'negative' assessment 'unacceptable'
......Assessment score:
......Positive=0.01
......Negative=0.99

......'negative' target 'service'
......Target score:
......Positive=0.01
......Negative=0.99

......'negative' assessment 'unacceptable'
......Assessment score:
......Positive=0.01
......Negative=0.99

......'positive' target 'concierge'
......Target score:
......Positive=1.00
......Negative=0.00

......'positive' assessment 'nice'
......Assessment score:
......Positive=1.00
......Negative=0.00





Press any key to continue . . .

```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Создайте функцию с именем `sentiment_analysis_example()`, которая принимает клиент в качестве аргумента, а затем вызывает функцию `analyze_sentiment()`. Возвращаемый объект ответа будет содержать метку тональности и оценку всего входного документа, а также анализ тональности для каждого предложения.


```python
def sentiment_analysis_example(client):

    documents = ["I had the best day of my life. I wish you were there with me."]
    response = client.analyze_sentiment(documents = documents)[0]
    print("Document Sentiment: {}".format(response.sentiment))
    print("Overall scores: positive={0:.2f}; neutral={1:.2f}; negative={2:.2f} \n".format(
        response.confidence_scores.positive,
        response.confidence_scores.neutral,
        response.confidence_scores.negative,
    ))
    for idx, sentence in enumerate(response.sentences):
        print("Sentence: {}".format(sentence.text))
        print("Sentence {} sentiment: {}".format(idx+1, sentence.sentiment))
        print("Sentence score:\nPositive={0:.2f}\nNeutral={1:.2f}\nNegative={2:.2f}\n".format(
            sentence.confidence_scores.positive,
            sentence.confidence_scores.neutral,
            sentence.confidence_scores.negative,
        ))
          
sentiment_analysis_example(client)
```

### <a name="output"></a>Выходные данные

```console
Document Sentiment: positive
Overall scores: positive=1.00; neutral=0.00; negative=0.00 

Sentence: I had the best day of my life.
Sentence 1 sentiment: positive
Sentence score:
Positive=1.00
Neutral=0.00
Negative=0.00

Sentence: I wish you were there with me.
Sentence 2 sentiment: neutral
Sentence score:
Positive=0.21
Neutral=0.77
Negative=0.02
```

---

## <a name="language-detection"></a>Определение языка

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Создайте функцию с именем `language_detection_example()`, которая принимает клиент в качестве аргумента, а затем вызывает функцию `detect_language()`. Возвращенный объект ответа будет содержать обнаруженный язык в `primary_language`, если выполнение успешное, и значение `error` в противном случае.

> [!Tip]
> В некоторых случаях неоднозначность языков на основе входных данных может быть трудно устранить. Используйте параметр `country_hint`, чтобы указать двухбуквенный код страны. По умолчанию API использует US в качестве значения countryHint. Чтобы отменить это поведение, вы можете сбросить этот параметр, установив для этого значения пустую строку `country_hint : ""`. 

```python
def language_detection_example(client):
    try:
        documents = ["Ce document est rédigé en Français."]
        response = client.detect_language(documents = documents, country_hint = 'us')[0]
        print("Language: ", response.primary_language.name)

    except Exception as err:
        print("Encountered exception. {}".format(err))
language_detection_example(client)
```


### <a name="output"></a>Выходные данные

```console
Language:  French
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Создайте функцию с именем `language_detection_example()`, которая принимает клиент в качестве аргумента, а затем вызывает функцию `detect_language()`. Возвращенный объект ответа будет содержать обнаруженный язык в `primary_language`, если выполнение успешное, и значение `error` в противном случае.

> [!Tip]
> В некоторых случаях неоднозначность языков на основе входных данных может быть трудно устранить. Используйте параметр `country_hint`, чтобы указать двухбуквенный код страны. По умолчанию API использует US в качестве значения countryHint. Чтобы отменить это поведение, вы можете сбросить этот параметр, установив для этого значения пустую строку `country_hint : ""`. 

```python
def language_detection_example(client):
    try:
        documents = ["Ce document est rédigé en Français."]
        response = client.detect_language(documents = documents, country_hint = 'us')[0]
        print("Language: ", response.primary_language.name)

    except Exception as err:
        print("Encountered exception. {}".format(err))
language_detection_example(client)
```


### <a name="output"></a>Выходные данные

```console
Language:  French
```


---

## <a name="named-entity-recognition-ner"></a>Распознавание именованных сущностей (NER)

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

> [!NOTE]
> В версии `3.1`: 
> * Связывание сущностей — это отдельный запрос, отличный от NER.

Создайте функцию с именем `entity_recognition_example`, которая принимает клиент в качестве аргумента, а затем вызывает функцию `recognize_entities()` и выполняет итерацию результатов. Возвращенный объект ответа будет содержать список обнаруженных сущностей в `entity`, если выполнение успешное, и значение `error` в противном случае. Для каждой обнаруженной сущности выведите ее категорию и подкатегорию, если они существуют.

```python
def entity_recognition_example(client):

    try:
        documents = ["I had a wonderful trip to Seattle last week."]
        result = client.recognize_entities(documents = documents)[0]

        print("Named Entities:\n")
        for entity in result.entities:
            print("\tText: \t", entity.text, "\tCategory: \t", entity.category, "\tSubCategory: \t", entity.subcategory,
                    "\n\tConfidence Score: \t", round(entity.confidence_score, 2), "\tLength: \t", entity.length, "\tOffset: \t", entity.offset, "\n")

    except Exception as err:
        print("Encountered exception. {}".format(err))
entity_recognition_example(client)
```

### <a name="output"></a>Выходные данные

```console
Named Entities:

        Text:    trip   Category:        Event  SubCategory:     None
        Confidence Score:        0.61   Length:          4      Offset:          18

        Text:    Seattle        Category:        Location       SubCategory:     GPE
        Confidence Score:        0.82   Length:          7      Offset:          26

        Text:    last week      Category:        DateTime       SubCategory:     DateRange
        Confidence Score:        0.8    Length:          9      Offset:          34
```

### <a name="entity-linking"></a>Связывание сущностей

Создайте функцию с именем `entity_linking_example()`, которая принимает клиент в качестве аргумента, а затем вызывает функцию `recognize_linked_entities()` и выполняет итерацию результатов. Возвращенный объект ответа будет содержать список обнаруженных сущностей в `entities`, если выполнение успешное, и значение `error` в противном случае. Так как связанные сущности уникально идентифицируются, вхождения одной и той же сущности группируются в объекте `entity` как список объектов `match`.

```python
def entity_linking_example(client):

    try:
        documents = ["""Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, 
        to develop and sell BASIC interpreters for the Altair 8800. 
        During his career at Microsoft, Gates held the positions of chairman,
        chief executive officer, president and chief software architect, 
        while also being the largest individual shareholder until May 2014."""]
        result = client.recognize_linked_entities(documents = documents)[0]

        print("Linked Entities:\n")
        for entity in result.entities:
            print("\tName: ", entity.name, "\tId: ", entity.data_source_entity_id, "\tUrl: ", entity.url,
            "\n\tData Source: ", entity.data_source)
            print("\tMatches:")
            for match in entity.matches:
                print("\t\tText:", match.text)
                print("\t\tConfidence Score: {0:.2f}".format(match.confidence_score))
                print("\t\tOffset: {}".format(match.offset))
                print("\t\tLength: {}".format(match.length))
            
    except Exception as err:
        print("Encountered exception. {}".format(err))
entity_linking_example(client)
```

### <a name="output"></a>Выходные данные

```console
Linked Entities:

        Name:  Microsoft        Id:  Microsoft  Url:  https://en.wikipedia.org/wiki/Microsoft
        Data Source:  Wikipedia
        Matches:
                Text: Microsoft
                Confidence Score: 0.55
                Offset: 0
                Length: 9
                Text: Microsoft
                Confidence Score: 0.55
                Offset: 168
                Length: 9
        Name:  Bill Gates       Id:  Bill Gates         Url:  https://en.wikipedia.org/wiki/Bill_Gates
        Data Source:  Wikipedia
        Matches:
                Text: Bill Gates
                Confidence Score: 0.63
                Offset: 25
                Length: 10
                Text: Gates
                Confidence Score: 0.63
                Offset: 179
                Length: 5
        Name:  Paul Allen       Id:  Paul Allen         Url:  https://en.wikipedia.org/wiki/Paul_Allen
        Data Source:  Wikipedia
        Matches:
                Text: Paul Allen
                Confidence Score: 0.60
                Offset: 40
                Length: 10
        Name:  April 4  Id:  April 4    Url:  https://en.wikipedia.org/wiki/April_4
        Data Source:  Wikipedia
        Matches:
                Text: April 4
                Confidence Score: 0.32
                Offset: 54
                Length: 7
        Name:  BASIC    Id:  BASIC      Url:  https://en.wikipedia.org/wiki/BASIC
        Data Source:  Wikipedia
        Matches:
                Text: BASIC
                Confidence Score: 0.33
                Offset: 98
                Length: 5
        Name:  Altair 8800      Id:  Altair 8800        Url:  https://en.wikipedia.org/wiki/Altair_8800
        Data Source:  Wikipedia
        Matches:
                Text: Altair 8800
                Confidence Score: 0.88
                Offset: 125
                Length: 11
```

### <a name="personally-identifiable-information-recognition"></a>Распознавание личных сведений

Создайте функцию с именем `pii_recognition_example`, которая принимает клиент в качестве аргумента, а затем вызывает функцию `recognize_pii_entities()` и выполняет итерацию результатов. Возвращенный объект ответа будет содержать список обнаруженных сущностей в `entity`, если выполнение успешное, и значение `error` в противном случае. Для каждой обнаруженной сущности выведите ее категорию и подкатегорию, если они существуют.

```python
def pii_recognition_example(client):
    documents = [
        "The employee's SSN is 859-98-0987.",
        "The employee's phone number is 555-555-5555."
    ]
    response = client.recognize_pii_entities(documents, language="en")
    result = [doc for doc in response if not doc.is_error]
    for doc in result:
        print("Redacted Text: {}".format(doc.redacted_text))
        for entity in doc.entities:
            print("Entity: {}".format(entity.text))
            print("\tCategory: {}".format(entity.category))
            print("\tConfidence Score: {}".format(entity.confidence_score))
            print("\tOffset: {}".format(entity.offset))
            print("\tLength: {}".format(entity.length))
pii_recognition_example(client)
```

### <a name="output"></a>Выходные данные

```console
Redacted Text: The employee's SSN is ***********.
Entity: 859-98-0987
        Category: U.S. Social Security Number (SSN)
        Confidence Score: 0.65
        Offset: 22
        Length: 11
Redacted Text: The employee's phone number is ************.
Entity: 555-555-5555
        Category: Phone Number
        Confidence Score: 0.8
        Offset: 31
        Length: 12
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

> [!NOTE]
> В версии `3.0`: 
> * Связывание сущностей — это отдельный запрос, отличный от NER.

Создайте функцию с именем `entity_recognition_example`, которая принимает клиент в качестве аргумента, а затем вызывает функцию `recognize_entities()` и выполняет итерацию результатов. Возвращенный объект ответа будет содержать список обнаруженных сущностей в `entity`, если выполнение успешное, и значение `error` в противном случае. Для каждой обнаруженной сущности выведите ее категорию и подкатегорию, если они существуют.

```python
def entity_recognition_example(client):

    try:
        documents = ["I had a wonderful trip to Seattle last week."]
        result = client.recognize_entities(documents = documents)[0]

        print("Named Entities:\n")
        for entity in result.entities:
            print("\tText: \t", entity.text, "\tCategory: \t", entity.category, "\tSubCategory: \t", entity.subcategory,
                    "\n\tConfidence Score: \t", round(entity.confidence_score, 2), "\n")

    except Exception as err:
        print("Encountered exception. {}".format(err))
entity_recognition_example(client)
```

### <a name="output"></a>Выходные данные

```console
Named Entities:

        Text:    trip   Category:        Event  SubCategory:     None
        Confidence Score:        0.61

        Text:    Seattle        Category:        Location       SubCategory:     GPE
        Confidence Score:        0.82

        Text:    last week      Category:        DateTime       SubCategory:     DateRange
        Confidence Score:        0.8
```

### <a name="entity-linking"></a>Связывание сущностей

Создайте функцию с именем `entity_linking_example()`, которая принимает клиент в качестве аргумента, а затем вызывает функцию `recognize_linked_entities()` и выполняет итерацию результатов. Возвращенный объект ответа будет содержать список обнаруженных сущностей в `entities`, если выполнение успешное, и значение `error` в противном случае. Так как связанные сущности уникально идентифицируются, вхождения одной и той же сущности группируются в объекте `entity` как список объектов `match`.

```python
def entity_linking_example(client):

    try:
        documents = ["""Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, 
        to develop and sell BASIC interpreters for the Altair 8800. 
        During his career at Microsoft, Gates held the positions of chairman,
        chief executive officer, president and chief software architect, 
        while also being the largest individual shareholder until May 2014."""]
        result = client.recognize_linked_entities(documents = documents)[0]

        print("Linked Entities:\n")
        for entity in result.entities:
            print("\tName: ", entity.name, "\tId: ", entity.data_source_entity_id, "\tUrl: ", entity.url,
            "\n\tData Source: ", entity.data_source)
            print("\tMatches:")
            for match in entity.matches:
                print("\t\tText:", match.text)
                print("\t\tConfidence Score: {0:.2f}".format(match.confidence_score))
            
    except Exception as err:
        print("Encountered exception. {}".format(err))
entity_linking_example(client)
```

### <a name="output"></a>Выходные данные

```console
Linked Entities:

        Name:  Altair 8800      Id:  Altair 8800        Url:  https://en.wikipedia.org/wiki/Altair_8800
        Data Source:  Wikipedia
        Matches:
                Text: Altair 8800
                Confidence Score: 0.88
        Name:  Bill Gates       Id:  Bill Gates         Url:  https://en.wikipedia.org/wiki/Bill_Gates
        Data Source:  Wikipedia
        Matches:
                Text: Bill Gates
                Confidence Score: 0.63
                Text: Gates
                Confidence Score: 0.63
        Name:  Paul Allen       Id:  Paul Allen         Url:  https://en.wikipedia.org/wiki/Paul_Allen
        Data Source:  Wikipedia
        Matches:
                Text: Paul Allen
                Confidence Score: 0.60
        Name:  Microsoft        Id:  Microsoft  Url:  https://en.wikipedia.org/wiki/Microsoft
        Data Source:  Wikipedia
        Matches:
                Text: Microsoft
                Confidence Score: 0.55
                Text: Microsoft
                Confidence Score: 0.55
        Name:  April 4  Id:  April 4    Url:  https://en.wikipedia.org/wiki/April_4
        Data Source:  Wikipedia
        Matches:
                Text: April 4
                Confidence Score: 0.32
        Name:  BASIC    Id:  BASIC      Url:  https://en.wikipedia.org/wiki/BASIC
        Data Source:  Wikipedia
        Matches:
                Text: BASIC
                Confidence Score: 0.33
```

---

### <a name="key-phrase-extraction"></a>Извлечение ключевой фразы

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

Создайте функцию с именем `key_phrase_extraction_example()`, которая принимает клиент в качестве аргумента, а затем вызывает функцию `extract_key_phrases()`. При успешном выполнении результат будет содержать список обнаруженных ключевых фраз в `key_phrases`. В противном случае вернется `error`. Затем выведите все обнаруженные ключевые фразы.

```python
def key_phrase_extraction_example(client):

    try:
        documents = ["My cat might need to see a veterinarian."]

        response = client.extract_key_phrases(documents = documents)[0]

        if not response.is_error:
            print("\tKey Phrases:")
            for phrase in response.key_phrases:
                print("\t\t", phrase)
        else:
            print(response.id, response.error)

    except Exception as err:
        print("Encountered exception. {}".format(err))
        
key_phrase_extraction_example(client)
```


### <a name="output"></a>Выходные данные

```console
    Key Phrases:
         cat
         veterinarian
```

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Создайте функцию с именем `key_phrase_extraction_example()`, которая принимает клиент в качестве аргумента, а затем вызывает функцию `extract_key_phrases()`. При успешном выполнении результат будет содержать список обнаруженных ключевых фраз в `key_phrases`. В противном случае вернется `error`. Затем выведите все обнаруженные ключевые фразы.

```python
def key_phrase_extraction_example(client):

    try:
        documents = ["My cat might need to see a veterinarian."]

        response = client.extract_key_phrases(documents = documents)[0]

        if not response.is_error:
            print("\tKey Phrases:")
            for phrase in response.key_phrases:
                print("\t\t", phrase)
        else:
            print(response.id, response.error)

    except Exception as err:
        print("Encountered exception. {}".format(err))
        
key_phrase_extraction_example(client)
```


### <a name="output"></a>Выходные данные

```console
    Key Phrases:
         cat
         veterinarian
```


---

## <a name="use-the-api-asynchronously-with-the-batch-analyze-operation"></a>Асинхронное использование API с пакетной операцией анализа

# <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-1)

[!INCLUDE [Analyze operation pricing](../analyze-operation-pricing-caution.md)]

Создайте функцию с именем `analyze_batch_actions_example()`, которая принимает клиент в качестве аргумента, а затем вызывает функцию `begin_analyze_batch_actions()`. В результате будет выполняться длительная операция, опрашиваемая на предмет результатов.

```python
    def analyze_batch_actions_example(client):
        documents = [
            "Microsoft was founded by Bill Gates and Paul Allen."
        ]

        poller = text_analytics_client.begin_analyze_batch_actions(
            documents,
            display_name="Sample Text Analysis",
            entities_recognition_tasks=[EntitiesRecognitionTask()]
        )

        result = poller.result()
        action_results = [action_result for action_result in list(result) if not action_result.is_error]

        entities_recognition_task_result = action_results[0]
        print("Results of Entities Recognition action:")
        docs = [doc for doc in first_action_result.document_results if not doc.is_error]

        for idx, doc in enumerate(docs):
            print("\nDocument text: {}".format(documents[idx]))
            for entity in doc.entities:
                print("Entity: {}".format(entity.text))
                print("...Category: {}".format(entity.category))
                print("...Confidence Score: {}".format(entity.confidence_score))
                print("...Offset: {}".format(entity.offset))
            print("------------------------------------------")

analyze_example(client)
```

### <a name="output"></a>Выходные данные

```console
Results of Entities Recognition task:
Document text: Microsoft was founded by Bill Gates and Paul Allen.
Entity: Microsoft
...Category: Organization
...Confidence Score: 0.83
...Offset: 0
Entity: Bill Gates
...Category: Person
...Confidence Score: 0.85
...Offset: 25
Entity: Paul Allen
...Category: Person
...Confidence Score: 0.9
...Offset: 40
------------------------------------------
```

Кроме того, вы можете использовать пакетную операцию анализа для обнаружения персональных данных и извлечения ключевых фраз. [Пример пакетного анализа](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/samples/sample_analyze_batch_actions.py) см. на сайте GitHub.

# <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Эта функция недоступна в версии 3.0.

---
