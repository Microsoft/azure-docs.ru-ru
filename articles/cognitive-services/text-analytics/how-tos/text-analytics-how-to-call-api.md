---
title: Вызов API "Анализ текста"
titleSuffix: Azure Cognitive Services
description: В этой статье объясняется, как можно вызвать Анализ текста Azure Cognitive Services REST API и POST.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 03/01/2021
ms.author: aahi
ms.custom: references_regions
ms.openlocfilehash: 3c6fb1ca23bcc9c57e73bcaf960e0387611fcff3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104599220"
---
# <a name="how-to-call-the-text-analytics-rest-api"></a>Как вызвать REST API службы "Анализ текста"

В этой статье мы используем Анализ текста REST API и [POST](https://www.postman.com/downloads/) для демонстрации основных концепций. API предоставляет несколько синхронных и асинхронных конечных точек для использования функций службы. 

## <a name="create-a-text-analytics-resource"></a>Создание ресурса "Анализ текста"

> [!NOTE]
> * Если вы хотите использовать [](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/) `/analyze` `/health` конечные точки или, вам потребуется ресурс анализ текста с помощью ценовой категории Standard (S). `/analyze`Конечная точка включена в [ценовую категорию](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/).

Прежде чем использовать API анализа текста, вам потребуется создать ресурс Azure с ключом и конечной точкой для ваших приложений. 

1.  Во-первых, перейдите к [портал Azure](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) и создайте новый ресурс анализ текста, если у вас еще нет такого ресурса. Выберите [ценовую категорию](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/).

2.  Выберите регион, который вы хотите использовать для конечной точки.  Обратите внимание, что `/analyze` `/health` конечные точки и доступны только в следующих регионах: Западная часть США 2, Восточная часть США 2, Центральная часть США, Северная Европа и Западная Европа.

3.  Создайте ресурс Анализ текста и перейдите в колонку "ключи и конечная точка" в левой части страницы. Скопируйте ключ, который будет использоваться позже при вызове API. Вы добавите его позже в качестве значения для `Ocp-Apim-Subscription-Key` заголовка.

## <a name="change-your-pricing-tier"></a>Изменение ценовой категории 

Если у вас есть ресурс Анализ текста с использованием ценовой категории S0 – S4, необходимо обновить его для использования стандартной [ценовой категории](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/)(S). Ценовые категории S0 – S4 будут сняты с учета. Чтобы обновить цены на ресурс, выполните следующие действия.

1. Перейдите к ресурсу Анализ текста в [портал Azure](https://portal.azure.com/).
2. В меню навигации слева выберите **ценовая** Категория. Он будет ниже **управления ресурсами**. 
3. Выберите ценовую категорию Standard (S). Затем щелкните **Выбрать**.

Вы также можете создать новый ресурс Анализ текста с ценовой категорией Standard (S) и перенести приложения, чтобы использовать учетные данные для нового ресурса. 

## <a name="using-the-api-synchronously"></a>Синхронное использование API

Можно вызвать Анализ текста синхронно (для сценариев с низкой задержкой). При использовании синхронного API необходимо вызывать каждый API (функцию) отдельно. Если необходимо вызвать несколько функций, ознакомьтесь с приведенным ниже разделом об асинхронном вызове Анализ текста. 

## <a name="using-the-api-asynchronously"></a>Асинхронное использование API

Начиная с версии 3.1 – Preview. 3, API анализа текста предоставляет две асинхронные конечные точки: 

* `/analyze`Конечная точка для анализ текста позволяет анализировать один и тот же набор текстовых документов с несколькими функциями текстовой аналитики в одном вызове API. Ранее для использования нескольких функций потребуется создать отдельные вызовы API для каждой операции. Эту возможность следует использовать, если необходимо анализировать большие наборы документов с несколькими Анализ текстаными функциями.

* `/health`Конечная точка для анализ текста работоспособности, которая может извлекать и помечать соответствующие медицинские сведения из документов клинической практике.  

См. таблицу ниже, чтобы узнать, какие функции можно использовать асинхронно. Обратите внимание, что из конечной точки можно вызывать только несколько функций `/analyze` . 

| Компонент | Синхронная | Асинхронный |
|--|--|--|
| Определение языка | ✔ |  |
| Анализ мнений | ✔ |  |
| Интеллектуальный анализ данных | ✔ |  |
| Извлечение ключевой фразы | ✔ | ✔* |
| Распознавание именованных сущностей (включая персональные и фи) | ✔ | ✔* |
| Связывание сущностей | ✔ | ✔* |
| Анализ текста для работоспособности (контейнер) | ✔ |  |
| Анализ текста для работоспособности (API) |  | ✔  |

`*` — Вызывается асинхронно через `/analyze` конечную точку.


[!INCLUDE [text-analytics-api-references](../includes/text-analytics-api-references.md)]

<a name="json-schema"></a>

## <a name="api-request-formats"></a>Форматы запросов API

В API анализа текста можно отправить синхронные и асинхронные вызовы.

#### <a name="synchronous"></a>[Синхронная](#tab/synchronous)

### <a name="synchronous-requests"></a>Синхронные запросы

Формат запросов API одинаков для всех синхронных операций. Документы отправляются в объекте JSON как необработанный неструктурированный текст. XML не поддерживается. Схема JSON состоит из элементов, описанных ниже.

| Элемент | Допустимые значения | Необходим? | Использование |
|---------|--------------|-----------|-------|
|`id` |Типом данных выступает строка, однако обычно идентификаторы документов являются целыми числами. | Обязательно | Система использует предоставленные идентификаторы, чтобы структурировать выходные данные. Коды языков, ключевые фразы и оценка тональности создаются для каждого идентификатора в запросе.|
|`text` | Неструктурированный необработанный текст, не более 5 120 символов. | Обязательно | Для распознавания языка текст может быть выражен на любом языке. Для анализа тональности, извлечения ключевых фраз и идентификации сущности текст должен быть на [поддерживаемом языке](../language-support.md). |
|`language` | Двухзначный код [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) для [поддерживаемого языка](../language-support.md). | Различается | Требуется для анализа тональности, извлечения ключевых фраз и связывания сущностей. Необязательно для распознавания языка. Код можно исключить, и это не будет ошибкой, однако без него анализ будет менее точным. Код языка должен соответствовать предоставленному вами элементу `text`. |

Ниже приведен пример запроса API для синхронных Анализ текста конечных точек. 

```json
{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Sample text to be sent to the text analytics api."
    }
  ]
}
```

#### <a name="asynchronous"></a>[Асинхронный](#tab/asynchronous)

### <a name="asynchronous-requests-to-the-analyze-endpoint"></a>Асинхронные запросы к `/analyze` конечной точке

> [!NOTE]
> Последняя Предварительная версия клиентской библиотеки Анализ текста позволяет вызывать асинхронные операции анализа с помощью клиентского объекта. Примеры можно найти на сайте GitHub:
* [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics)
* [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/)
* [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics)

`/analyze`Конечная точка позволяет выбрать, какие из поддерживаемых функций анализ текста будут использоваться в одном вызове API. Сейчас эта конечная точка поддерживает:

* Извлечение ключевых фраз 
* Распознавание именованных сущностей (включая персональные и фи)
* Связывание сущностей

| Элемент | Допустимые значения | Необходим? | Использование |
|---------|--------------|-----------|-------|
|`displayName` | Строка | Необязательно | Используется в качестве отображаемого имени для уникального идентификатора задания.|
|`analysisInput` | Включает `documents` поле ниже | Обязательно | Содержит сведения о документах, которые необходимо отправить. |
|`documents` | Включает `id` поля и `text` ниже | Обязательно | Содержит сведения для каждого отправляемого документа и необработанный текст документа. |
|`id` | Строка | Обязательно | Предоставленные идентификаторы используются для структурирования выходных данных. |
|`text` | Неструктурированный необработанный текст, не более 125 000 символов. | Обязательно | Язык должен быть на английском языке, который в настоящее время поддерживается. |
|`tasks` | Включает следующие функции анализ текста: `entityRecognitionTasks` , `entityLinkingTasks` `keyPhraseExtractionTasks` или `entityRecognitionPiiTasks` . | Обязательно | Одна или несколько Анализ текста функций, которые вы хотите использовать. Обратите внимание, что `entityRecognitionPiiTasks` имеет необязательный `domain` параметр, для которого можно задать значение `pii` или, `phi` и `pii-categories` для обнаружения выбранных типов сущностей. Если `domain` параметр не указан, система по умолчанию принимает значение `pii` . |
|`parameters` | Включает `model-version` поля и `stringIndexType` ниже | Обязательно | Это поле включено в указанные выше задачи функции. Они содержат сведения о версии модели, которую необходимо использовать, и типе индекса. |
|`model-version` | Строка | Обязательно | Укажите версию вызываемой модели, которую необходимо использовать.  |
|`stringIndexType` | Строка | Обязательно | Укажите текстовый декодер, соответствующий используемой среде программирования.  Поддерживаются типы `textElement_v8` (по умолчанию), `unicodeCodePoint` , `utf16CodeUnit` . Дополнительные сведения см. в [статье о смещениях текста](../concepts/text-offsets.md#offsets-in-api-version-31-preview) .  |
|`domain` | Строка | Необязательно | Применяется только в качестве параметра к `entityRecognitionPiiTasks` задаче и может иметь значение `pii` или `phi` . Значение по умолчанию —, `pii` если не указано.  |

```json
{
    "displayName": "My Job",
    "analysisInput": {
        "documents": [
            {
                "id": "doc1",
                "text": "It's incredibly sunny outside! I'm so happy"
            },
            {
                "id": "doc2",
                "text": "Pike place market is my favorite Seattle attraction."
            }
        ]
    },
    "tasks": {
        "entityRecognitionTasks": [
            {
                "parameters": {
                    "model-version": "latest",
                    "stringIndexType": "TextElements_v8"
                }
            }
        ],
        "entityLinkingTasks": [
            {
                "parameters": {
                    "model-version": "latest",
                    "stringIndexType": "TextElements_v8"
                }
            }
        ],
        "keyPhraseExtractionTasks": [{
            "parameters": {
                "model-version": "latest"
            }
        }],
        "entityRecognitionPiiTasks": [{
            "parameters": {
                "model-version": "latest",
                "stringIndexType": "TextElements_v8",
                "domain": "phi",
                "pii-categories":"default"
            }
        }]
    }
}

```

### <a name="asynchronous-requests-to-the-health-endpoint"></a>Асинхронные запросы к `/health` конечной точке

Формат запросов API к Анализ текста для API, размещенного в работоспособности, аналогичен его контейнеру. Документы отправляются в объекте JSON как необработанный неструктурированный текст. XML не поддерживается. Схема JSON состоит из элементов, описанных ниже.  Заполните и отправьте [форму запроса Cognitive Services](https://aka.ms/csgate) , чтобы запросить доступ к анализ текста для общедоступной предварительной версии работоспособности. Вам не будет выставлен счет за Анализ текста для использования работоспособности. 

| Элемент | Допустимые значения | Необходим? | Использование |
|---------|--------------|-----------|-------|
|`id` |Типом данных выступает строка, однако обычно идентификаторы документов являются целыми числами. | Обязательно | Система использует предоставленные идентификаторы, чтобы структурировать выходные данные. |
|`text` | Неструктурированный необработанный текст, не более 5 120 символов. | Обязательно | Обратите внимание, что в настоящее время поддерживается только английский текст. |
|`language` | Двухзначный код [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) для [поддерживаемого языка](../language-support.md). | Обязательно | `en`Поддерживается только в настоящее время. |

Ниже приведен пример запроса API для Анализ текста конечных точек работоспособности. 

```json
example.json

{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Subject was administered 100mg remdesivir intravenously over a period of 120 min"
    }
  ]
}
```

---

>[!TIP]
> Сведения о тарифах и ограничениях размера для отправки данных в API анализа текста см. в статье [ограничения скорости и данных](../concepts/data-limits.md) .


## <a name="set-up-a-request"></a>Настройка запроса 

В POST (или другом средстве тестирования веб-API) добавьте конечную точку для компонента, который вы хотите использовать. Используйте приведенную ниже таблицу, чтобы найти соответствующий формат конечной точки и заменить `<your-text-analytics-resource>` его конечной точкой ресурса. Пример:

`https://my-resource.cognitiveservices.azure.com/text/analytics/v3.0/languages`

#### <a name="synchronous"></a>[Синхронная](#tab/synchronous)

### <a name="endpoints-for-sending-synchronous-requests"></a>Конечные точки для отправки синхронных запросов

| Компонент | Тип запроса | Конечные точки ресурсов |
|--|--|--|
| Определение языка | POST | `<your-text-analytics-resource>/text/analytics/v3.0/languages` |
| Анализ мнений | POST | `<your-text-analytics-resource>/text/analytics/v3.0/sentiment` |
| Интеллектуальный анализ мнений | POST | `<your-text-analytics-resource>/text/analytics/v3.0/sentiment?opinionMining=true` |
| Извлечение ключевой фразы | POST | `<your-text-analytics-resource>/text/analytics/v3.0/keyPhrases` |
| Распознавание именованных сущностей — общие | POST | `<your-text-analytics-resource>/text/analytics/v3.0/entities/recognition/general` |
| Распознавание именованных сущностей — PII | POST | `<your-text-analytics-resource>/text/analytics/v3.0/entities/recognition/pii` |
| Распознавание именованных сущностей — фи | POST |  `<your-text-analytics-resource>/text/analytics/v3.0/entities/recognition/pii?domain=phi` |

#### <a name="asynchronous"></a>[Асинхронный](#tab/asynchronous)

### <a name="endpoints-for-sending-asynchronous-requests-to-the-analyze-endpoint"></a>Конечные точки для отправки асинхронных запросов к `/analyze` конечной точке

| Компонент | Тип запроса | Конечные точки ресурсов |
|--|--|--|
| Отправить задание анализа | POST | `https://<your-text-analytics-resource>/text/analytics/v3.1-preview.4/analyze` |
| Получение состояния и результатов анализа | GET | `https://<your-text-analytics-resource>/text/analytics/v3.1-preview.4/analyze/jobs/<Operation-Location>` |

### <a name="endpoints-for-sending-asynchronous-requests-to-the-health-endpoint"></a>Конечные точки для отправки асинхронных запросов к `/health` конечной точке

| Компонент | Тип запроса | Конечные точки ресурсов |
|--|--|--|
| Отправка Анализ текста для задания работоспособности  | POST | `https://<your-text-analytics-resource>/text/analytics/v3.1-preview.4/entities/health/jobs` |
| Получение состояния задания и результатов | GET | `https://<your-text-analytics-resource>/text/analytics/v3.1-preview.4/entities/health/jobs/<Operation-Location>` |
| отмена задания. | DELETE | `https://<your-text-analytics-resource>/text/analytics/v3.1-preview.4/entities/health/jobs/<Operation-Location>` |

--- 

После создания конечной точки в POST (или другом средстве тестирования веб-API):

1. Выберите тип запроса для компонента, который вы хотите использовать.
2. Вставьте в конечную точку требуемой операции из приведенной выше таблицы.
3. Задайте три заголовка запроса:

   + `Ocp-Apim-Subscription-Key`: ключ доступа, полученный из портал Azure
   + `Content-Type`: application/json
   + `Accept`: application/json

    Если вы используете POST, ваш запрос должен выглядеть примерно так, как на следующем снимке экрана, предполагая, что `/keyPhrases` Конечная точка.
    
    ![Снимок экрана запроса с конечной точкой и заголовками](../media/postman-request-keyphrase-1.png)
    
4. Выберите **необработанный** формат **текста**
    
    ![Снимок экрана запроса с параметрами текста](../media/postman-request-body-raw.png)

5. Вставьте в некоторые документы JSON в допустимом формате. Используйте примеры в разделе **Формат запроса API** выше, а дополнительные сведения и примеры см. в следующих разделах:

      + [Пример. Как определить язык с помощью Анализа текста](text-analytics-how-to-language-detection.md)
      + [Пример. Как извлечь ключевые фразы с помощью Анализа текста](text-analytics-how-to-keyword-extraction.md)
      + [Пример. Как определить тональность с помощью Анализа текста](text-analytics-how-to-sentiment-analysis.md)
      + [Распознавание сущностей](text-analytics-how-to-entity-linking.md)

## <a name="send-the-request"></a>Отправка запроса

Отправьте запрос API. Если вызвать синхронную конечную точку, ответ будет отображен немедленно, как один документ JSON с элементом для каждого идентификатора документа, указанного в запросе.

Если вы выполнили вызов асинхронной `/analyze` или `/health` конечной точки, убедитесь, что получен код ответа 202. чтобы просмотреть результаты, необходимо получить ответ:

1. В ответе API найдите `Operation-Location` из заголовка, который определяет задание, отправленное в API. 
2. Создайте запрос GET для используемой конечной точки. см. приведенную [выше таблицу](#set-up-a-request) для формата конечной точки и просмотрите [справочную документацию по API](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/AnalyzeStatus). Пример:

    `https://my-resource.cognitiveservices.azure.com/text/analytics/v3.1-preview.4/analyze/jobs/<Operation-Location>`

3. Добавьте в `Operation-Location` запрос.

4. Ответ будет отдельным документом JSON с элементом для каждого идентификатора документа, указанного в запросе.

Обратите внимание, что для асинхронных `/analyze` `/health` операций или, результаты запроса GET на шаге 2 выше доступны в течение 24 часов с момента создания задания.  На этот раз указывается `expirationDateTime` значение в ответе Get.  По истечении этого периода результаты очищаются и больше не будут доступны для извлечения.    

## <a name="example-api-responses"></a>Примеры ответов API
 
# <a name="synchronous"></a>[Синхронная](#tab/synchronous)

### <a name="example-responses-for-synchronous-operation"></a>Примеры ответов для синхронной операции

Синхронные ответы конечных точек будут зависеть от используемой конечной точки. Примеры ответов см. в следующих статьях.

+ [Пример. Как определить язык с помощью Анализа текста](text-analytics-how-to-language-detection.md#step-3-view-the-results)
+ [Пример. Как извлечь ключевые фразы с помощью Анализа текста](text-analytics-how-to-keyword-extraction.md#step-3-view-results)
+ [Пример. Как определить тональность с помощью Анализа текста](text-analytics-how-to-sentiment-analysis.md#view-the-results)
+ [Распознавание сущностей](text-analytics-how-to-entity-linking.md#view-results)

# <a name="asynchronous"></a>[Асинхронный](#tab/asynchronous)

### <a name="example-responses-for-asynchronous-operations"></a>Примеры ответов для асинхронных операций

В случае успешного выполнения запрос GET к `/analyze` конечной точке возвратит объект, содержащий назначенные задачи. Например, `keyPhraseExtractionTasks`. Эти задачи содержат объект Response из соответствующей Анализ текста функции. Дополнительные сведения см. в приведенных ниже статьях.

+ [Пример. Как извлечь ключевые фразы с помощью Анализа текста](text-analytics-how-to-keyword-extraction.md#step-3-view-results)
+ [Распознавание сущностей](text-analytics-how-to-entity-linking.md#view-results)
+ [Анализ текста для сферы здравоохранения](text-analytics-for-health.md#hosted-asynchronous-web-api-response)

--- 

## <a name="see-also"></a>См. также раздел

* [Text Analytics overview](../overview.md) (Общие сведения об анализе текста)
* [Часто задаваемые вопросы](../text-analytics-resource-faq.md)</br>
* [Анализ текста](//go.microsoft.com/fwlink/?LinkID=759712)
* [Использование клиентской библиотеки Анализа текста](../quickstarts/client-libraries-rest-api.md)
* [Новые возможности](../whats-new.md)
