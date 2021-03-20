---
title: Предварительно созданная сущность Age — LUIS
titleSuffix: Azure Cognitive Services
description: В этой статье описана готовая сущность age в службе "Распознавание речи" (LUIS).
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 10/04/2019
ms.openlocfilehash: 4e1a6b1b6fb99a2786de06e89960e8480a5e1338
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91532930"
---
# <a name="age-prebuilt-entity-for-a-luis-app"></a>Предварительно созданная сущность age для приложения LUIS
Предварительно созданная сущность age фиксирует значение возраста как в цифровом выражении, так и в виде количества дней, недель, месяцев или лет. Так как эта сущность уже обучена, добавлять в назначения приложения примеры фраз, содержащие сущности age, не нужно. Сущность age поддерживается во [многих языках и региональных параметрах](luis-reference-prebuilt-entities.md).

## <a name="types-of-age"></a>Типы сущностей age
Возраст управляется из репозитория [распознаваемого текста](https://github.com/Microsoft/Recognizers-Text/blob/master/Patterns/English/English-NumbersWithUnit.yaml#L3) GitHub.

## <a name="resolution-for-prebuilt-age-entity"></a>Разрешение для предварительно созданной сущности age



#### <a name="v3-response"></a>[V3 ответ](#tab/V3)

Следующий код JSON имеет параметр со `verbose` значением `false` :

```json
"entities": {
    "age": [
        {
            "number": 90,
            "unit": "Day"
        }
    ]
}
```
#### <a name="v3-verbose-response"></a>[V3 подробный ответ](#tab/V3-verbose)
Следующий код JSON имеет параметр со `verbose` значением `true` :

```json
"entities": {
    "age": [
        {
            "number": 90,
            "unit": "Day"
        }
    ],
    "$instance": {
        "age": [
            {
                "type": "builtin.age",
                "text": "90 day old",
                "startIndex": 2,
                "length": 10,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor"
            }
        ]
    }
}
```
#### <a name="v2-response"></a>[Ответ v2](#tab/V2)

В следующем примере показано разрешение сущности **builtin.age**.

```json
  "entities": [
    {
      "entity": "90 day old",
      "type": "builtin.age",
      "startIndex": 2,
      "endIndex": 11,
      "resolution": {
        "unit": "Day",
        "value": "90"
      }
    }
```
* * *

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [конечной точке прогнозирования V3](luis-migration-api-v3.md).

См. дополнительные сведения о сущностях [currency](luis-reference-prebuilt-currency.md), [datetimeV2](luis-reference-prebuilt-datetimev2.md) и [dimension](luis-reference-prebuilt-dimension.md).
