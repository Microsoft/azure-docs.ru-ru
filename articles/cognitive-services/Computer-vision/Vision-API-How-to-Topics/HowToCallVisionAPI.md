---
title: Вызов API компьютерного зрения
titleSuffix: Azure Cognitive Services
description: Узнайте, как вызывать API компьютерного зрения с помощью REST API в Azure Cognitive Services.
services: cognitive-services
author: KellyDF
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: sample
ms.date: 09/09/2019
ms.author: kefre
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: abb367b64da0811a1ff46efe60b60485375f809f
ms.sourcegitcommit: 8d1b97c3777684bd98f2cfbc9d440b1299a02e8f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102486069"
---
# <a name="call-the-computer-vision-api"></a>Вызов API компьютерного зрения

В этой статье показано, как вызвать API Компьютерного зрения с помощью REST API. Примеры написаны как на языке C# с помощью клиентской библиотеки API компьютерного зрения, так и в виде вызовов HTTP POST или GET. Эта статья посвящена следующим вопросам.

- Получение тегов, описания и категорий
- Получение сведений, относящихся к определенной теме ("celebrities")

В примерах, приведенных в этой статье, демонстрируются следующие возможности.

* Анализ изображения для возврата массива тегов и описания
* Анализ изображения с помощью модели предметной области (в частности, модель "знаменитости") для получения соответствующего результата в перенастраиваемом JSON

Эти функции предлагают следующие возможности.

- **Вариант 1**. Анализ с заданной областью — анализ только указанной модели
- **Вариант 2**. Расширенный анализ — анализ для предоставления дополнительных сведений с использованием [классификации по 86 категориям](../Category-Taxonomy.md)

## <a name="prerequisites"></a>Предварительные требования

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Получив подписку Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="создайте ресурс Компьютерного зрения"  target="_blank">create a Computer Vision resource </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Компьютерного зрения потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.
* URL-адрес изображения или путь к локально сохраненному изображению
* Поддерживаемые методы ввода: двоичный файл изображения в формате RAW в форме приложения или потока октета или URL-адреса изображения
* Поддерживаемые форматы файлов изображений: JPEG, PNG, GIF и BMP
* Размер файла изображения: не более 4 МБ
* Размеры изображения: 50 &times; 50 пикселей или больше
  
## <a name="authorize-the-api-call"></a>Авторизация вызова API

Каждый вызов API компьютерного зрения требует ключ подписки. Этот ключ должен быть передан через параметр строки запроса или указан в заголовке запроса.

Вы можете передать ключ подписки одним из следующих способов.

* Передать его через строку запроса, как в следующем примере API Компьютерного зрения.

  ```
  https://westus.api.cognitive.microsoft.com/vision/v2.1/analyze?visualFeatures=Description,Tags&subscription-key=<Your subscription key>
  ```

* Указать в заголовке HTTP-запроса.

  ```
  ocp-apim-subscription-key: <Your subscription key>
  ```

* При использовании клиентской библиотеки передавайте ключ через конструктор ComputerVisionClient и укажите регион в свойстве клиента:

    ```
    var visionClient = new ComputerVisionClient(new ApiKeyServiceClientCredentials("Your subscriptionKey"))
    {
        Endpoint = "https://westus.api.cognitive.microsoft.com"
    }
    ```

## <a name="upload-an-image-to-the-computer-vision-api-service"></a>Передача изображения в API службы Компьютерного зрения

Основной способ выполнения вызова API Компьютерного зрения заключается в непосредственной передаче изображения для возврата тегов, описания и имен знаменитостей. Это делается путем отправки запроса POST с двоичным изображением в теле HTTP-запроса вместе с данными, считанными из изображения. Метод передачи одинаков для всех вызовов API Компьютерного зрения. Единственное отличие заключается в указании параметров запроса. 

Для рассматриваемого изображения теги и описание можно получить одним из следующих способов.

### <a name="option-1-get-a-list-of-tags-and-a-description"></a>Вариант 1. Получение списка тегов и описания

```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/analyze?visualFeatures=Description,Tags&subscription-key=<Your subscription key>
```

```csharp
using System.IO;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;

ImageAnalysis imageAnalysis;
var features = new VisualFeatureTypes[] { VisualFeatureTypes.Tags, VisualFeatureTypes.Description };

using (var fs = new FileStream(@"C:\Vision\Sample.jpg", FileMode.Open))
{
  imageAnalysis = await visionClient.AnalyzeImageInStreamAsync(fs, features);
}
```

### <a name="option-2-get-a-list-of-tags-only-or-a-description-only"></a>Вариант 2. Получение только списка тегов или только описания

В случае получения только тегов выполните:

```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/tag?subscription-key=<Your subscription key>
var tagResults = await visionClient.TagImageAsync("http://contoso.com/example.jpg");
```

В случае получения только описания выполните:

```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/describe?subscription-key=<Your subscription key>
using (var fs = new FileStream(@"C:\Vision\Sample.jpg", FileMode.Open))
{
  imageDescription = await visionClient.DescribeImageInStreamAsync(fs);
}
```

## <a name="get-domain-specific-analysis-celebrities"></a>Анализ содержимого, связанного с определенными предметными областями (знаменитости)

### <a name="option-1-scoped-analysis---analyze-only-a-specified-model"></a>Вариант 1. Анализ с заданной областью — анализ только указанной модели
```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/models/celebrities/analyze
var celebritiesResult = await visionClient.AnalyzeImageInDomainAsync(url, "celebrities");
```

Для этого параметра все остальные параметры запроса {visualFeatures, details} недопустимы. Чтобы увидеть все поддерживаемые модели, используйте следующее.

```
GET https://westus.api.cognitive.microsoft.com/vision/v2.1/models 
var models = await visionClient.ListModelsAsync();
```

### <a name="option-2-enhanced-analysis---analyze-to-provide-additional-details-by-using-86-categories-taxonomy"></a>Вариант 2. Расширенный анализ — анализ для предоставления дополнительных сведений с использованием классификации по 86 категориям

В приложениях, где пользователи хотят получить анализ обычного изображения с дополнительными сведениями из одной или нескольких моделей, предназначенных для определенных сфер, расширьте API версии 1 с помощью параметра модели запроса.

```
POST https://westus.api.cognitive.microsoft.com/vision/v2.1/analyze?details=celebrities
```

При вызове этого метода сначала вызывается классификатор по [86 категориям](../Category-Taxonomy.md). Если какие-либо категории соответствуют известным или сопоставленным моделям, возникает второй этап вызовов классификатора. Например, если "details=all" или "details" содержит "celebrities", то модель знаменитостей вызывается после вызова классификатора по 86 категориям. Полученный результат содержит определенную категорию. В отличие от варианта 1, этот метод увеличивает задержку для пользователей, заинтересованных в знаменитостях.

В этом случае все параметры запроса из варианта 1 ведут себя таким же образом. Если не задать параметр visualFeatures=categories, он будет включен неявно.

## <a name="retrieve-and-understand-the-json-output-for-analysis"></a>Получение и интерпретация выходных данных JSON для анализа

Ниже приведен пример:

```json
{  
  "tags":[  
    {  
      "name":"outdoor",
      "score":0.976
    },
    {  
      "name":"bird",
      "score":0.95
    }
  ],
  "description":{  
    "tags":[  
      "outdoor",
      "bird"
    ],
    "captions":[  
      {  
        "text":"partridge in a pear tree",
        "confidence":0.96
      }
    ]
  }
}
```

Поле | Тип | Содержимое
------|------|------|
Теги  | `object` | Объект верхнего уровня для массива тегов.
tags[].Name | `string`    | Ключевое слово из классификатора тегов.
tags[].Score    | `number`    | Оценка достоверности находится в пределах от 0 до 1.
description     | `object`    | Объект верхнего уровня для описания.
description.tags[] |    `string`    | Список тегов.  Если нет уверенности в способности создать заголовок, теги могут быть единственной информацией, доступной для вызывающего объекта.
description.captions[].text    | `string`    | Фраза, описывающая образ.
description.captions[].confidence    | `number`    | Оценка достоверности для фразы.

## <a name="retrieve-and-understand-the-json-output-of-domain-specific-models"></a>Получение и интерпретация выходных данных JSON из моделей, предназначенных для определенных сфер

### <a name="option-1-scoped-analysis---analyze-only-a-specified-model"></a>Вариант 1. Анализ с заданной областью — анализ только указанной модели

Выходные данные представляют собой массив тегов, как показано в следующем примере.

```json
{  
  "result":[  
    {  
      "name":"golden retriever",
      "score":0.98
    },
    {  
      "name":"Labrador retriever",
      "score":0.78
    }
  ]
}
```

### <a name="option-2-enhanced-analysis---analyze-to-provide-additional-details-by-using-the-86-categories-taxonomy"></a>Вариант 2. Расширенный анализ — анализ для предоставления дополнительных сведений с использованием классификации по 86 категориям

Для моделей, зависящих от предметной области, при использовании варианта 2 (расширенный анализ) тип возвращаемого значения категорий расширен, как показано в следующем примере.

```json
{  
  "requestId":"87e44580-925a-49c8-b661-d1c54d1b83b5",
  "metadata":{  
    "width":640,
    "height":430,
    "format":"Jpeg"
  },
  "result":{  
    "celebrities":[  
      {  
        "name":"Richard Nixon",
        "faceRectangle":{  
          "left":107,
          "top":98,
          "width":165,
          "height":165
        },
        "confidence":0.9999827
      }
    ]
  }
}
```

Поле категории — список из одной или нескольких из [86 категорий](../Category-Taxonomy.md) в исходной классификации. Категории, которые заканчиваются символом подчеркивания, соответствуют категории и ее производным (например, "people_" или "people_group" для модели знаменитостей).

Поле    | Тип    | Содержимое
------|------|------|
Категории | `object`    | Объект верхнего уровня.
categories[].name     | `string`    | Имя из списка классификации по 86 категориям.
categories[].score    | `number`    | Оценка достоверности находится в пределах от 0 до 1.
categories[].detail     | `object?`      | (Необязательно) Объект Detail.

Если несколько категорий совпадают (например, классификатор по 86 категориям возвращает оценку для "people_" и "people_young", когда model=celebrities), сведения присоединяются к совпадению наиболее общего уровня (в этом примере — "people_").

## <a name="error-responses"></a>Сообщения об ошибках

Эти ошибки идентичны тем, которые есть в методе vision.analyze, но с дополнительной ошибкой NotSupportedModel (HTTP 400), которая может быть возвращена как в варианте 1, так и в варианте 2. Для варианта 2 (расширенный анализ), если какая-либо из моделей, указанных в сведениях, не распознается, API возвращает ошибку NotSupportedModel, даже если один или несколько из них являются допустимыми. Чтобы узнать, какие модели поддерживаются, пользователи могут вызывать функцию listModels.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы использовать REST API, см. [Справочник по API компьютерного зрения компьютера](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2-preview-3/operations/56f91f2e778daf14a499f21b).
