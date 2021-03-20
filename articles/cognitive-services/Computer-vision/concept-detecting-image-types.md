---
title: Обнаружение типа изображения — Компьютерное зрение
titleSuffix: Azure Cognitive Services
description: Понятия, связанные с функцией определения типа изображений API компьютерного зрения.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 03/11/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: 6d2ed00f3fc6f5b52a9a13a96f1e1659e30f02d5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96532607"
---
# <a name="detecting-image-types-with-computer-vision"></a>Определение типов изображений с помощью API компьютерного зрения

С помощью API [анализа изображений](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b) компьютерное зрение может анализировать тип содержимого изображений, указывая, является ли изображение картинкой или графиком.

## <a name="detecting-clip-art"></a>Определение картинки

Компьютерное зрение анализирует изображение и оценивает вероятность того, что оно является картинкой, по шкале от 0 до 3, как показано в таблице ниже.

| Значение | Значение |
|-------|---------|
| 0 | не картинка |
| 1 | Неоднозначно |
| 2 | Обычная картинка |
| 3 | Хорошая картинка |

### <a name="clip-art-detection-examples"></a>Примеры определения картинок

В приведенных ниже ответах JSON показано, что именно возвращает компьютерное зрение при оценке вероятности того, что примеры изображений являются картинками.

![Картинка с ломтиком сыра](./Images/cheese_clipart.png)

```json
{
    "imageType": {
        "clipArtType": 3,
        "lineDrawingType": 0
    },
    "requestId": "88c48d8c-80f3-449f-878f-6947f3b35a27",
    "metadata": {
        "height": 225,
        "width": 300,
        "format": "Jpeg"
    }
}
```

![Синий дом и передний двор](./Images/house_yard.png)

```json
{
    "imageType": {
        "clipArtType": 0,
        "lineDrawingType": 0
    },
    "requestId": "a9c8490a-2740-4e04-923b-e8f4830d0e47",
    "metadata": {
        "height": 200,
        "width": 300,
        "format": "Jpeg"
    }
}
```

## <a name="detecting-line-drawings"></a>Определение графических изображений

Компьютерное зрение анализирует изображение и возвращает логическое значение, указывающее, является ли оно графическим.

### <a name="line-drawing-detection-examples"></a>Примеры определения графических изображений

В приведенных ниже ответах JSON показано, что именно возвращает компьютерное зрение при определении того, являются ли примеры изображений графическими.

![Графическое изображение льва](./Images/lion_drawing.png)

```json
{
    "imageType": {
        "clipArtType": 2,
        "lineDrawingType": 1
    },
    "requestId": "6442dc22-476a-41c4-aa3d-9ceb15172f01",
    "metadata": {
        "height": 268,
        "width": 300,
        "format": "Jpeg"
    }
}
```

![Белый цветок на зеленом фоне](./Images/flower.png)

```json
{
    "imageType": {
        "clipArtType": 0,
        "lineDrawingType": 0
    },
    "requestId": "98437d65-1b05-4ab7-b439-7098b5dfdcbf",
    "metadata": {
        "height": 200,
        "width": 300,
        "format": "Jpeg"
    }
}
```

## <a name="use-the-api"></a>Использование API

Функция обнаружения типов изображений является частью API [анализа изображений](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b) . Вы можете вызывать этот API с помощью собственного пакета SDK или с помощью вызовов REST. Включите `ImageType` в параметр запроса **висуалфеатурес** . Затем, когда вы получаете полный ответ JSON, просто Проанализируйте строку для содержимого `"imageType"` раздела.

* [Краткое руководство. REST API или клиентские библиотеки Компьютерного зрения](./quickstarts-sdk/client-library.md?pivots=programming-language-csharp)
