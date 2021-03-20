---
title: Когнитивный навык объединения текста
titleSuffix: Azure Cognitive Search
description: Объедините текст из коллекции полей в одно консолидированное поле. Используйте этот опытный навык в конвейере обогащения искусственного интеллекта в Azure Когнитивный поиск.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: b364655f26c6ac29c14d387d69d7b4277d6aeb86
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88924645"
---
#   <a name="text-merge-cognitive-skill"></a>Когнитивный навык объединения текста

Когнитивный навык **объединения текста** объединяет текст из коллекции полей в одном поле. 

> [!NOTE]
> Этот навык не привязан к API Cognitive Services, и за его использование плата не взимается. Но вам следует по-прежнему [подключать ресурс Cognitive Services](cognitive-search-attach-cognitive-services.md), чтобы переопределить **бесплатную** категорию ресурса, ограниченную малым количеством обогащений в день.

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.MergeSkill

## <a name="skill-parameters"></a>Параметры навыков

Параметры зависят от регистра.

| Имя параметра     | Описание |
|--------------------|-------------|
| `insertPreTag`    | Строка, которая должна быть включена перед каждой вставкой. Значение по умолчанию — `" "`. Чтобы исключить пробел, задайте значение `""`.  |
| `insertPostTag`   | Строка, которая должна быть включена после каждой вставки. Значение по умолчанию — `" "`. Чтобы исключить пробел, задайте значение `""`.  |


##  <a name="sample-input"></a>Пример ввода
Документ JSON, предоставляющий полезные входные данные для этого навыка, может иметь следующий вид:

```json
{
  "values": [
    {
      "recordId": "1",
      "data":
      {
        "text": "The brown fox jumps over the dog",
        "itemsToInsert": ["quick", "lazy"],
        "offsets": [3, 28]
      }
    }
  ]
}
```

##  <a name="sample-output"></a>Пример выходных данных
В этом примере показаны выходные данные предыдущего ввода, при условии, что *insertPreTag* имеет значение `" "`, а *insertPostTag* — `""`. 

```json
{
  "values": [
    {
      "recordId": "1",
      "data":
      {
        "mergedText": "The quick brown fox jumps over the lazy dog"
      }
    }
  ]
}
```

## <a name="extended-sample-skillset-definition"></a>Расширенный пример определения набора навыков

Распространенным вариантом использования объединения текста является возможность объединить текстовое представление изображений (текст из навыка OCR или заголовок изображения) с полем содержимого документа. 

В следующем примере набора навыков используется умение OCR для извлечения текста из изображений, встроенных в документ. Затем создается поле *merged_text*, содержащее как исходный, так и обработанный OCR текст из каждого изображения. Дополнительные сведения о навыке OCR см. [здесь](./cognitive-search-skill-ocr.md).

```json
{
  "description": "Extract text from images and merge with content text to produce merged_text",
  "skills":
  [
    {
      "description": "Extract text (plain and structured) from image.",
      "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
      "context": "/document/normalized_images/*",
      "defaultLanguageCode": "en",
      "detectOrientation": true,
      "inputs": [
        {
          "name": "image",
          "source": "/document/normalized_images/*"
        }
      ],
      "outputs": [
        {
          "name": "text"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.MergeSkill",
      "description": "Create merged_text, which includes all the textual representation of each image inserted at the right location in the content field.",
      "context": "/document",
      "insertPreTag": " ",
      "insertPostTag": " ",
      "inputs": [
        {
          "name":"text", 
          "source": "/document/content"
        },
        {
          "name": "itemsToInsert", 
          "source": "/document/normalized_images/*/text"
        },
        {
          "name":"offsets", 
          "source": "/document/normalized_images/*/contentOffset" 
        }
      ],
      "outputs": [
        {
          "name": "mergedText", 
          "targetName" : "merged_text"
        }
      ]
    }
  ]
}
```
В приведенном выше примере предполагается наличие поля нормализованных изображений. Чтобы создать поле normalized-images, настройте конфигурацию *imageAction* в определении индексатора *generateNormalizedImages*, как показано ниже.

```json
{
  //...rest of your indexer definition goes here ...
  "parameters":{
    "configuration":{
        "dataToExtract":"contentAndMetadata",
        "imageAction":"generateNormalizedImages"
    }
  }
}
```

## <a name="see-also"></a>См. также раздел

+ [Встроенные навыки](cognitive-search-predefined-skills.md)
+ [Определение набора навыков](cognitive-search-defining-skillset.md)
+ [Создание индексатора — REST](/rest/api/searchservice/create-indexer)