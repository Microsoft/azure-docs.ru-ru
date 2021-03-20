---
title: Содержимое, зависящее от домена — Компьютерное зрение
titleSuffix: Azure Cognitive Services
description: Узнайте, как указать область классификации изображений, чтобы получить более подробные сведения об изображении.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 02/08/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: 5cd872d66088e165bfc8356ab6d96a0a6135a0e0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94538314"
---
# <a name="detect-domain-specific-content"></a>Обнаружение содержимого, связанного с определенными предметными областями

Помимо добавления тегов и высокого уровня классификации, Компьютерное зрение также поддерживает дальнейший анализ с учетом предметной области, с использованием моделей, которые были обучены на специализированных данных.

Существует два способа использования моделей предметной области — сами по себе (анализ по областям) или в качестве расширения функции классификации.

### <a name="scoped-analysis"></a>Ограниченный анализ

Вы можете проанализировать изображение, используя только выбранную модель для конкретного домена, вызвав API [Models/ \<model\> /Analyze](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b) .

Ниже приведен пример ответа JSON, возвращаемого API **models/celebrities/analysis** для следующего изображения.

![Сатья Наделла, улыбающаяся](./images/satya.jpeg)

```json
{
  "result": {
    "celebrities": [{
      "faceRectangle": {
        "top": 391,
        "left": 318,
        "width": 184,
        "height": 184
      },
      "name": "Satya Nadella",
      "confidence": 0.99999856948852539
    }]
  },
  "requestId": "8217262a-1a90-4498-a242-68376a4b956b",
  "metadata": {
    "width": 800,
    "height": 1200,
    "format": "Jpeg"
  }
}
```

### <a name="enhanced-categorization-analysis"></a>Расширенный анализ классификации

Вы также можете использовать модели для предметной области для дополнения общего анализа образа. Вы можете сделать это как часть [высокоуровневой категоризации](concept-categorizing-images.md), определяя модели для предметной области в параметре *details* вызова API [Analyze](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b).

При вызове этого метода сначала вызывается классификатор таксономии на 86 категориях. Если какая-либо из обнаруженных категорий имеет соответствующую модель для предметной области, образ также передается с помощью этой модели и добавляются результаты.

В следующем ответе JSON показано, как предметно-ориентированный анализ можно включить в качестве узла `detail` в расширенный анализ классификации.

```json
"categories":[
  {
    "name":"abstract_",
    "score":0.00390625
  },
  {
    "name":"people_",
    "score":0.83984375,
    "detail":{
      "celebrities":[
        {
          "name":"Satya Nadella",
          "faceRectangle":{
            "left":597,
            "top":162,
            "width":248,
            "height":248
          },
          "confidence":0.999028444
        }
      ],
      "landmarks":[
        {
          "name":"Forbidden City",
          "confidence":0.9978346
        }
      ]
    }
  }
]
```

## <a name="list-the-domain-specific-models"></a>Список моделей предметных областей

Сейчас Компьютерное зрение поддерживает следующие модели для предметной области.

| Имя | Описание |
|------|-------------|
| celebrities | Распознавание знаменитостей; поддерживается для изображений, которые были классифицированы как относящиеся к категории `people_` |
| landmarks | Распознавание ориентиров; поддерживается для изображений, которые были классифицированы как относящиеся к категории `outdoor_` или `building_` |

Вызов API [Models](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f20e) вернет информацию вместе с категориями, к которым может применяться каждая модель.

```json
{
  "models":[
    {
      "name":"celebrities",
      "categories":[
        "people_",
        "人_",
        "pessoas_",
        "gente_"
      ]
    },
    {
      "name":"landmarks",
      "categories":[
        "outdoor_",
        "户外_",
        "屋外_",
        "aoarlivre_",
        "alairelibre_",
        "building_",
        "建筑_",
        "建物_",
        "edifício_"
      ]
    }
  ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

Сведения о концепциях [классификации изображений](concept-categorizing-images.md).
