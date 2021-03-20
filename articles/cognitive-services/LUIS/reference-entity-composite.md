---
title: Составной объект типа-LUIS
titleSuffix: Azure Cognitive Services
description: Составная сущность состоит из других сущностей, например готовых сущностей, простых, регулярных выражений и сущностей списка. Эти отдельные сущности формируют единую сущность.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 04/14/2020
ms.openlocfilehash: 3d936555a4d9a9c80718a24ba892c762f5a8354a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97608398"
---
# <a name="composite-entity"></a>Составная сущность

Составная сущность состоит из других сущностей, например готовых сущностей, простых, регулярных выражений и сущностей списка. Эти отдельные сущности формируют единую сущность.

> [!CAUTION]
> Эта сущность является **устаревшей**. Выполните миграцию в [сущность машинного обучения](reference-entity-machine-learned-entity.md).

**Эта сущность оптимальна для данных в таких случаях.**

* данные связаны между собой;
* Связаны между собой в контексте фразы.
* для них используются различные типы сущностей;
* должны быть сгруппированы и обработаны клиентским приложением в качестве единицы информации;
* имеют различные высказывания пользователей, которые требуют машинного обучения.

![составная сущность](./media/luis-concept-entities/composite-entity.png)

## <a name="example-json"></a>Пример JSON

Рассмотрим составную сущность предварительно построенного `number` и `Location::ToLocation` со следующими utterance:

`book 2 tickets to cairo`

Обратите внимание, что между `2` (числом) и `cairo` ToLocation находятся слова, которые не входят в какие-либо сущности. Зеленая линия подчеркивания, используемая в помеченном высказывании на веб-сайте [LUIS](luis-reference-regions.md), указывает на составную сущность.

![Составная сущность](./media/luis-concept-data-extraction/composite-entity.png)

#### <a name="v2-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V2](#tab/V2)

Составные сущности возвращаются в массиве `compositeEntities`, а все сущности в рамках составной возвращаются в массиве `entities`:

```JSON
  "entities": [
    {
      "entity": "2 tickets to cairo",
      "type": "ticketinfo",
      "startIndex": 5,
      "endIndex": 22,
      "score": 0.9214487
    },
    {
      "entity": "cairo",
      "type": "builtin.geographyV2.city",
      "startIndex": 18,
      "endIndex": 22
    },
    {
      "entity": "2",
      "type": "builtin.number",
      "startIndex": 5,
      "endIndex": 5,
      "resolution": {
        "subtype": "integer",
        "value": "2"
      }
    }
  ],
  "compositeEntities": [
    {
      "parentType": "ticketinfo",
      "value": "2 tickets to cairo",
      "children": [
        {
          "type": "builtin.number",
          "value": "2"
        },
        {
          "type": "builtin.geographyV2.city",
          "value": "cairo"
        }
      ]
    }
  ]
```

#### <a name="v3-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V3](#tab/V3)

Это JSON, если `verbose=false` в строке запроса задано:

```json
"entities": {
    "ticketinfo": [
        {
            "number": [
                2
            ],
            "geographyV2": [
                "cairo"
            ]
        }
    ]
}
```

Это JSON, если `verbose=true` в строке запроса задано:

```json
"entities": {
    "ticketinfo": [
        {
            "number": [
                2
            ],
            "geographyV2": [
                "cairo"
            ],
            "$instance": {
                "number": [
                    {
                        "type": "builtin.number",
                        "text": "2",
                        "startIndex": 5,
                        "length": 1,
                        "modelTypeId": 2,
                        "modelType": "Prebuilt Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    }
                ],
                "geographyV2": [
                    {
                        "type": "builtin.geographyV2.city",
                        "text": "cairo",
                        "startIndex": 18,
                        "length": 5,
                        "modelTypeId": 2,
                        "modelType": "Prebuilt Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    }
                ]
            }
        }
    ],
    "$instance": {
        "ticketinfo": [
            {
                "type": "ticketinfo",
                "text": "2 tickets to cairo",
                "startIndex": 5,
                "length": 18,
                "score": 0.9214487,
                "modelTypeId": 4,
                "modelType": "Composite Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```

* * *

|Объект данных|Имя сущности|Значение|
|--|--|--|
|Предварительно созданная сущность — число|"builtin.number"|"2"|
|Предварительно созданная сущность — GeographyV2|"Location::ToLocation"|Каиро|

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о сущностях:

* [Основные понятия](luis-concept-entity-types.md)
* [Создание](luis-how-to-add-entities.md)
