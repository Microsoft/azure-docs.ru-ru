---
title: Описания изображений — Компьютерное зрение
titleSuffix: Azure Cognitive Services
description: Понятия, связанные с функцией описания изображений API компьютерного зрения.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 02/11/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: 4fd328dad3544697ba1f4896f7383be857b097a5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96530669"
---
# <a name="describe-images-with-human-readable-language"></a>Описание изображений на понятном для пользователя языке

API "Компьютерное зрение" позволяет анализировать изображения и создавать понятные пользователю предложения, описывающее содержимое изображений. Алгоритм фактически возвращает несколько описаний на основе различных визуальных функций, и каждому описанию предоставляется Оценка достоверности. Готовые выходные данные представлены в виде списка описаний, упорядоченных по оценке достоверности (от самой высокой до самой низкой).

## <a name="image-description-example"></a>Пример описания изображения

В приведенном ниже ответе JSON показано, что именно возвращает Компьютерное зрение при описании примера изображения на основе его визуальных характеристик.

![Черно-белая фотография здания на Манхэттене](./Images/bw_buildings.png)

```json
{
    "description": {
        "tags": ["outdoor", "building", "photo", "city", "white", "black", "large", "sitting", "old", "water", "skyscraper", "many", "boat", "river", "group", "street", "people", "field", "tall", "bird", "standing"],
        "captions": [
            {
                "text": "a black and white photo of a city",
                "confidence": 0.95301952483304808
            },
            {
                "text": "a black and white photo of a large city",
                "confidence": 0.94085190563213816
            },
            {
                "text": "a large white building in a city",
                "confidence": 0.93108362931954824
            }
        ]
    },
    "requestId": "b20bfc83-fb25-4b8d-a3f8-b2a1f084b159",
    "metadata": {
        "height": 300,
        "width": 239,
        "format": "Jpeg"
    }
}
```

## <a name="use-the-api"></a>Использование API

Функция описания образа является частью API [анализа изображений](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b) . Вы можете вызывать этот API с помощью собственного пакета SDK или с помощью вызовов REST. Включите `Description` в параметр запроса **висуалфеатурес** . Затем, когда вы получаете полный ответ JSON, просто Проанализируйте строку для содержимого `"description"` раздела.

* [Краткое руководство. REST API или клиентские библиотеки Компьютерного зрения](./quickstarts-sdk/client-library.md?pivots=programming-language-csharp)

## <a name="next-steps"></a>Дальнейшие действия

Изучите связанные понятия [маркировки изображений](concept-tagging-images.md) и [категоризации изображений](concept-categorizing-images.md).
