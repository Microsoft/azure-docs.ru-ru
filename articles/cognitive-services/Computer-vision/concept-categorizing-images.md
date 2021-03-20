---
title: Классификация изображений — Компьютерное зрение
titleSuffix: Azure Cognitive Services
description: Понятия, связанные с функцией классификации изображений API компьютерного зрения.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 04/17/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: 9721ffa807c9adbeb50839113bc64fd23d8eb13f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96533729"
---
# <a name="categorize-images-by-subject-matter"></a>Категоризация изображений по темам

В дополнение к тегам и описанию, компьютерное зрение возвращает основанные на таксономии категории, обнаруженные в изображении. В отличие от тегов, категории организованы в иерархии родительского или дочернего наследования, и их меньше (86, в отличие от тысяч тегов). Все категории имен указаны на английском языке. Классификация может быть выполнена сама по себе или вместе с более новой моделью тегов.

## <a name="the-86-category-concept"></a>Концепция из 86 категорий

Компьютерное зрение может классифицировать изображение в широком или конкретном виде, используя список категорий 86 на следующей схеме. Полную таксономию в формате текста см. в статье [86-Categories Taxonomy](category-taxonomy.md) (Таксономия 86 категорий).

![Сгруппированный список всех категорий в классификации категорий](./Images/analyze_categories-v2.png)

## <a name="image-categorization-examples"></a>Примеры классификации изображений

В приведенном ниже ответе JSON показано, что именно возвращает компьютерное зрение при классификации примера изображения на основе его визуальных характеристик.

![Женщина на крыше жилого дома](./Images/woman_roof.png)

```json
{
    "categories": [
        {
            "name": "people_",
            "score": 0.81640625
        }
    ],
    "requestId": "bae7f76a-1cc7-4479-8d29-48a694974705",
    "metadata": {
        "height": 200,
        "width": 300,
        "format": "Jpeg"
    }
}
```

В приведенной ниже таблице показаны типичный набор изображений и категория, возвращаемая компьютерным зрением для каждого из них.

| Образ — | Category |
|-------|----------|
| ![Четыре человека позируют как семья](./Images/family_photo.png) | people_group |
| ![Щенок, сидящий на лужайке](./Images/cute_dog.png) | animal_dog |
| ![Человек, стоящий на скале на закате](./Images/mountain_vista.png) | outdoor_mountain |
| ![Гора булочек на столе](./Images/bread.png) | food_bread |

## <a name="use-the-api"></a>Использование API

Функция классификации является частью API [анализа изображений](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b) . Вы можете вызывать этот API с помощью собственного пакета SDK или с помощью вызовов REST. Включите `Categories` в параметр запроса **висуалфеатурес** . Затем, когда вы получаете полный ответ JSON, просто Проанализируйте строку для содержимого `"categories"` раздела.

* [Краткое руководство. REST API или клиентские библиотеки Компьютерного зрения](./quickstarts-sdk/client-library.md?pivots=programming-language-csharp)

## <a name="next-steps"></a>Дальнейшие действия

Изучите связанные понятия [маркировки изображений](concept-tagging-images.md) и [Описание изображений](concept-describing-images.md).
