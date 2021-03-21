---
title: Предварительно созданные сущности измерения — LUIS
titleSuffix: Azure Cognitive Services
description: В этой статье описана готовая сущность dimension в службе "Распознавание речи" (LUIS).
services: cognitive-services
ms.custom: seodec18
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 10/14/2019
ms.openlocfilehash: 0b5f081dc8c0bc203ccafde4a513371d11e36879
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91538353"
---
# <a name="dimension-prebuilt-entity-for-a-luis-app"></a>Предварительно созданная сущность dimension для приложения LUIS
Предварительно созданная сущность dimension обнаруживает различные типы измерений, независимо от языка и региональных параметров LUIS. Так как эта сущность уже обучена, добавлять в назначения приложения примеры фраз, содержащие сущности dimension, не нужно. Сущность dimension поддерживается во [многих языках и региональных параметрах](luis-reference-prebuilt-entities.md).

## <a name="types-of-dimension"></a>Типы сущностей dimension

Управление измерением осуществляется из репозитория [распознавания текста](https://github.com/Microsoft/Recognizers-Text/blob/master/Patterns/English/English-NumbersWithUnit.yaml) GitHub.

## <a name="resolution-for-dimension-entity"></a>Разрешение для сущности dimension

Для запроса возвращаются следующие объекты сущности:

`10 1/2 miles of cable`

#### <a name="v3-response"></a>[V3 ответ](#tab/V3)

Следующий код JSON имеет параметр со `verbose` значением `false` :

```json
"entities": {
    "dimension": [
        {
            "number": 10.5,
            "units": "Mile"
        }
    ]
}
```
#### <a name="v3-verbose-response"></a>[V3 подробный ответ](#tab/V3-verbose)
Следующий код JSON имеет параметр со `verbose` значением `true` :

```json
"entities": {
    "dimension": [
        {
            "number": 10.5,
            "units": "Mile"
        }
    ],
    "$instance": {
        "dimension": [
            {
                "type": "builtin.dimension",
                "text": "10 1/2 miles",
                "startIndex": 0,
                "length": 12,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```

#### <a name="v2-response"></a>[Ответ v2](#tab/V2)

В следующем примере показано разрешение сущности **builtin.dimension**.

```json
{
    "entity": "10 1/2 miles",
    "type": "builtin.dimension",
    "startIndex": 0,
    "endIndex": 11,
    "resolution": {
    "unit": "Mile",
    "value": "10.5"
    }
}
```
* * *

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [конечной точке прогнозирования V3](luis-migration-api-v3.md).

См. дополнительные сведения о сущностях [email](luis-reference-prebuilt-email.md), [number](luis-reference-prebuilt-number.md) и [ordinal](luis-reference-prebuilt-ordinal.md).
