---
title: Тип сущности машинного обучения — LUIS
titleSuffix: Azure Cognitive Services
description: Сущность машинного обучения — это предпочтительная сущность для создания LUIS приложений.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 04/30/2020
ms.openlocfilehash: f16f3ec445af777c3bde1cb2ab0735df4b6614f8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91542246"
---
# <a name="machine-learning-entity"></a>Сущность машинного обучения

Сущность машинного обучения — это предпочтительная сущность для создания LUIS приложений.


## <a name="example-json"></a>Пример JSON

Предположим, что приложение принимает заказы на пиццу, например [учебник по сущностям делимыми](tutorial-machine-learned-entity.md). Каждый заказ может включать несколько различных пицц, включая различные размеры.

Ниже приведены примеры фраз.

|Пример фразы продолжительностью для приложения пиццы|
|--|
|`Can I get a pepperoni pizza and a can of coke please`|
|`can I get a small pizza with onions peppers and olives`|
|`pickup an extra large meat lovers pizza`|



#### <a name="v3-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V3](#tab/V3)

Поскольку сущность машинного обучения может иметь множество вложенных сущностей с необходимыми функциями, это только пример. Он должен рассматриваться как пошаговое описание того, что будет возвращено сущностью.

Рассмотрим запрос:

`deliver 1 large cheese pizza on thin crust and 2 medium pepperoni pizzas on deep dish crust`

Это JSON, если `verbose=false` в строке запроса задано:

```json
"entities": {
    "Order": [
        {
            "FullPizzaWithModifiers": [
                {
                    "PizzaType": [
                        "cheese pizza"
                    ],
                    "Size": [
                        [
                            "Large"
                        ]
                    ],
                    "Quantity": [
                        1
                    ]
                },
                {
                    "PizzaType": [
                        "pepperoni pizzas"
                    ],
                    "Size": [
                        [
                            "Medium"
                        ]
                    ],
                    "Quantity": [
                        2
                    ],
                    "Crust": [
                        [
                            "Deep Dish"
                        ]
                    ]
                }
            ]
        }
    ],
    "ToppingList": [
        [
            "Cheese"
        ],
        [
            "Pepperoni"
        ]
    ],
    "CrustList": [
        [
            "Thin"
        ]
    ]
}

```

Это JSON, если `verbose=true` в строке запроса задано:

```json
"entities": {
    "Order": [
        {
            "FullPizzaWithModifiers": [
                {
                    "PizzaType": [
                        "cheese pizza"
                    ],
                    "Size": [
                        [
                            "Large"
                        ]
                    ],
                    "Quantity": [
                        1
                    ],
                    "$instance": {
                        "PizzaType": [
                            {
                                "type": "PizzaType",
                                "text": "cheese pizza",
                                "startIndex": 16,
                                "length": 12,
                                "score": 0.999998868,
                                "modelTypeId": 1,
                                "modelType": "Entity Extractor",
                                "recognitionSources": [
                                    "model"
                                ]
                            }
                        ],
                        "Size": [
                            {
                                "type": "SizeList",
                                "text": "large",
                                "startIndex": 10,
                                "length": 5,
                                "score": 0.998720646,
                                "modelTypeId": 1,
                                "modelType": "Entity Extractor",
                                "recognitionSources": [
                                    "model"
                                ]
                            }
                        ],
                        "Quantity": [
                            {
                                "type": "builtin.number",
                                "text": "1",
                                "startIndex": 8,
                                "length": 1,
                                "score": 0.999878645,
                                "modelTypeId": 1,
                                "modelType": "Entity Extractor",
                                "recognitionSources": [
                                    "model"
                                ]
                            }
                        ]
                    }
                },
                {
                    "PizzaType": [
                        "pepperoni pizzas"
                    ],
                    "Size": [
                        [
                            "Medium"
                        ]
                    ],
                    "Quantity": [
                        2
                    ],
                    "Crust": [
                        [
                            "Deep Dish"
                        ]
                    ],
                    "$instance": {
                        "PizzaType": [
                            {
                                "type": "PizzaType",
                                "text": "pepperoni pizzas",
                                "startIndex": 56,
                                "length": 16,
                                "score": 0.999987066,
                                "modelTypeId": 1,
                                "modelType": "Entity Extractor",
                                "recognitionSources": [
                                    "model"
                                ]
                            }
                        ],
                        "Size": [
                            {
                                "type": "SizeList",
                                "text": "medium",
                                "startIndex": 49,
                                "length": 6,
                                "score": 0.999841452,
                                "modelTypeId": 1,
                                "modelType": "Entity Extractor",
                                "recognitionSources": [
                                    "model"
                                ]
                            }
                        ],
                        "Quantity": [
                            {
                                "type": "builtin.number",
                                "text": "2",
                                "startIndex": 47,
                                "length": 1,
                                "score": 0.9996054,
                                "modelTypeId": 1,
                                "modelType": "Entity Extractor",
                                "recognitionSources": [
                                    "model"
                                ]
                            }
                        ],
                        "Crust": [
                            {
                                "type": "CrustList",
                                "text": "deep dish crust",
                                "startIndex": 76,
                                "length": 15,
                                "score": 0.761551,
                                "modelTypeId": 1,
                                "modelType": "Entity Extractor",
                                "recognitionSources": [
                                    "model"
                                ]
                            }
                        ]
                    }
                }
            ],
            "$instance": {
                "FullPizzaWithModifiers": [
                    {
                        "type": "FullPizzaWithModifiers",
                        "text": "1 large cheese pizza on thin crust",
                        "startIndex": 8,
                        "length": 34,
                        "score": 0.616001546,
                        "modelTypeId": 1,
                        "modelType": "Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    },
                    {
                        "type": "FullPizzaWithModifiers",
                        "text": "2 medium pepperoni pizzas on deep dish crust",
                        "startIndex": 47,
                        "length": 44,
                        "score": 0.7395033,
                        "modelTypeId": 1,
                        "modelType": "Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    }
                ]
            }
        }
    ],
    "ToppingList": [
        [
            "Cheese"
        ],
        [
            "Pepperoni"
        ]
    ],
    "CrustList": [
        [
            "Thin"
        ]
    ],
    "$instance": {
        "Order": [
            {
                "type": "Order",
                "text": "1 large cheese pizza on thin crust and 2 medium pepperoni pizzas on deep dish crust",
                "startIndex": 8,
                "length": 83,
                "score": 0.6881274,
                "modelTypeId": 1,
                "modelType": "Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ],
        "ToppingList": [
            {
                "type": "ToppingList",
                "text": "cheese",
                "startIndex": 16,
                "length": 6,
                "modelTypeId": 5,
                "modelType": "List Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            },
            {
                "type": "ToppingList",
                "text": "pepperoni",
                "startIndex": 56,
                "length": 9,
                "modelTypeId": 5,
                "modelType": "List Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ],
        "CrustList": [
            {
                "type": "CrustList",
                "text": "thin crust",
                "startIndex": 32,
                "length": 10,
                "modelTypeId": 5,
                "modelType": "List Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```
#### <a name="v2-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V2](#tab/V2)

Эта сущность недоступна в среде выполнения прогнозирования версии 2.
* * *

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о сущности машинного обучения, включая [учебник](tutorial-machine-learned-entity.md), [Основные понятия](luis-concept-entity-types.md#design-entities-for-decomposition)и [руководство](luis-how-to-add-entities.md#create-a-machine-learned-entity).

Сведения о сущности [списка](reference-entity-list.md) и сущности [регулярного выражения](reference-entity-regular-expression.md) .