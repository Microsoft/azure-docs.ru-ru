---
title: Предварительно построенные сущности URL-LUIS
titleSuffix: Azure Cognitive Services
description: В этой статье описана готовая сущность url в службе "Распознавание речи" (LUIS).
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 10/04/2019
ms.openlocfilehash: bc33fd1b90306a016c419f227fb9e73e83e8ea83
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91535259"
---
# <a name="url-prebuilt-entity-for-a-luis-app"></a>Предварительно созданная сущность url для приложения LUIS
Сущность URL-адреса извлекает URL-адреса с доменными именами или IP-адресами. Поскольку эта сущность уже обучена, добавлять в приложения примеры фраз, содержащие URL-адреса, не нужно. Сущность URL-адреса поддерживается только для такого языка и региональных параметров: `en-us`.

## <a name="types-of-urls"></a>Типы URL-адресов
Управление URL-адресом осуществляется из репозитория [распознаваемого текста](https://github.com/Microsoft/Recognizers-Text/blob/master/Patterns/Base-URL.yaml) GitHub

## <a name="resolution-for-prebuilt-url-entity"></a>Разрешение для предварительно созданной сущности URL-адреса

Для запроса возвращаются следующие объекты сущности:

`https://www.luis.ai is a great cognitive services example of artificial intelligence`

#### <a name="v3-response"></a>[V3 ответ](#tab/V3)

Следующий код JSON имеет параметр со `verbose` значением `false` :

```json
"entities": {
    "url": [
        "https://www.luis.ai"
    ]
}
```
#### <a name="v3-verbose-response"></a>[V3 подробный ответ](#tab/V3-verbose)

Следующий код JSON имеет параметр со `verbose` значением `true` :

```json
"entities": {
    "url": [
        "https://www.luis.ai"
    ],
    "$instance": {
        "url": [
            {
                "type": "builtin.url",
                "text": "https://www.luis.ai",
                "startIndex": 0,
                "length": 17,
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

В следующем примере показано решение проблемы, которая https://www.luis.ai представляет собой хороший пример использования искусственных служб.

```json
"entities": [
    {
        "entity": "https://www.luis.ai",
        "type": "builtin.url",
        "startIndex": 0,
        "endIndex": 17
    }
]
```

* * *

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [конечной точке прогнозирования V3](luis-migration-api-v3.md).

Дополнительные сведения о сущностях [порядковый номер](luis-reference-prebuilt-ordinal.md), [номер](luis-reference-prebuilt-number.md) и [температура](luis-reference-prebuilt-temperature.md).
