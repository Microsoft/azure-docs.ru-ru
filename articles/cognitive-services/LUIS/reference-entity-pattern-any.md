---
title: Шаблон. любой тип сущности — LUIS
titleSuffix: Azure Cognitive Services
description: Patterns.any — это местозаполнитель переменной длины, используемый только в высказывании шаблона для пометки начала и окончания сущности.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 09/29/2019
ms.openlocfilehash: ec23be3709cebc534c059a21c52452abff683b18
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91542212"
---
# <a name="patternany-entity"></a>Сущность Pattern.any

Patterns.any — это местозаполнитель переменной длины, используемый только в высказывании шаблона для пометки начала и окончания сущности.  

Сущности Pattern.Any необходимо помечать в примерах [шаблонов](luis-how-to-model-intent-pattern.md), а не в примерах намерений пользователя.

**Сущность оптимальна для данных в таких случаях.**

* Конечную часть сущности можно спутать с оставшимся текстом высказывания.

## <a name="usage"></a>Использование

В клиентском приложении для поиска книг по названию сущность pattern.any извлекает полное название. Высказывание шаблона, использующее сущность pattern.any для поиска книги, — `Was {BookTitle} written by an American this year[?]`.

В следующей таблице каждая строка имеет две версии высказывания. Верхняя utterance — это то, как LUIS изначально видит utterance. Не ясно, где начинается и заканчивается заголовок книги. В нижней utterance используется шаблон. любая сущность, помечающая начало и конец сущности.

|Utterance с сущностью, выделенной жирным шрифтом|
|--|
|`Was The Man Who Mistook His Wife for a Hat and Other Clinical Tales written by an American this year?`<br><br>Книга **The Man Who Mistook His Wife for a Hat and Other Clinical Tales (Человек, который принял жену за шляпу, и другие истории из врачебной практики)** была написана американским автором в этом году?|
|`Was Half Asleep in Frog Pajamas written by an American this year?`<br><br>Книга **Half Asleep in Frog Pajamas (Сонные глазки и пижама в лягушечку)** была написана американским автором в этом году?|
|`Was The Particular Sadness of Lemon Cake: A Novel written by an American this year?`<br><br>Был ли **определенный грусть Лемон торт: романский** , написанный в США в этом году?|
|`Was There's A Wocket In My Pocket! written by an American this year?`<br><br>**Есть ли Воккет в моем кармане!** была написана американским автором этом году?|
||



## <a name="example-json"></a>Пример JSON

Обратите внимание на следующий запрос:

`where is the form Understand your responsibilities as a member of the community and who needs to sign it after I read it?`

С именем внедренной формы для извлечения в виде шаблона. any:

`Understand your responsibilities as a member of the community`

#### <a name="v2-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V2](#tab/V2)

```JSON
"entities": [
  {
    "entity": "understand your responsibilities as a member of the community",
    "type": "FormName",
    "startIndex": 18,
    "endIndex": 78,
    "role": ""
  }
```


#### <a name="v3-prediction-endpoint-response"></a>[Отклик конечной точки прогнозирования V3](#tab/V3)

Это JSON, если `verbose=false` в строке запроса задано:

```json
"entities": {
    "FormName": [
        "Understand your responsibilities as a member of the community"
    ]
}
```

Это JSON, если `verbose=true` в строке запроса задано:

```json
"entities": {
    "FormName": [
        "Understand your responsibilities as a member of the community"
    ],
    "$instance": {
        "FormName": [
            {
                "type": "FormName",
                "text": "Understand your responsibilities as a member of the community",
                "startIndex": 18,
                "length": 61,
                "modelTypeId": 7,
                "modelType": "Pattern.Any Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```

* * *

## <a name="next-steps"></a>Дальнейшие действия

В этом [руководстве](luis-tutorial-pattern.md)используется **шаблон. Любая** сущность для извлечения данных из фразы продолжительностью, где фразы продолжительностью хорошо отформатированы и где конец данных можно легко путать с остальными словами utterance.
