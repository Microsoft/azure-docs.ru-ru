---
title: Анализ тональности — LUIS
titleSuffix: Azure Cognitive Services
description: Если настроен анализ тональности, он входит в ответ JSON LUIS.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 04/06/2021
ms.openlocfilehash: 7524644b34a6fd479c08b9ce6418c547c836add5
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "106554031"
---
# <a name="sentiment-analysis"></a>Анализ мнений
Если настроен анализ тональности, он входит в ответ JSON LUIS. Дополнительные сведения об анализе тональности см. в документации по [анализу текста](../text-analytics/index.yml).

LUIS использует Анализ текста v2. 

Анализ тональности настраивается при публикации приложения. Дополнительные сведения см. в статье [публикация приложения](./luis-how-to-publish-app.md).

## <a name="resolution-for-sentiment"></a>Разрешение для тональности

Данные тональности представляют собой оценку между 1 и 0, означающую положительную (ближе к 1) или отрицательную (ближе к 0) тональность данных.

#### <a name="english-language"></a>[Английский язык](#tab/english)

При использовании языка и региональных параметров `en-us` выводится следующий ответ:

```JSON
"sentimentAnalysis": {
  "label": "positive",
  "score": 0.9163064
}
```

#### <a name="other-languages"></a>[Другие языки](#tab/other-languages)

Для всех других языков и региональных параметров выводится ответ:

```JSON
"sentimentAnalysis": {
  "score": 0.9163064
}
```
* * *

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [конечной точке прогнозирования V3](luis-migration-api-v3.md).
