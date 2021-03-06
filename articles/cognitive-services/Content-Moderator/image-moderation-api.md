---
title: Модерация изображений — Content Moderator
titleSuffix: Azure Cognitive Services
description: Для взрослых и носящих непристойный характер можно Content Moderator использовать управляемое машинное средство для контроля образов и средства проверки в виде циклических изображений.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: pafarley
ms.openlocfilehash: fe76e32bfd9b1734f3c84a400f897b7af7e3168b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85801001"
---
# <a name="learn-image-moderation-concepts"></a>Изучение концепций модерации изображений

Используйте Content Moderator с машинным сопровождением образов и [средством проверки](Review-Tool-User-Guide/human-in-the-loop.md) на умеренные изображения для взрослых и носящих непристойный характер содержимого. Вы можете проверить наличие текста на изображениях, извлечь этот текст и (или) распознать лица. Вы можете сопоставить изображения с настраиваемым списком и предпринять соответствующие действия.

## <a name="evaluating-for-adult-and-racy-content"></a>Оценка на наличие содержимого для взрослых и (или) непристойного характера

Операция **Evaluate** возвращает показатель достоверности в диапазоне от 0 до 1. Она также возвращает двоичное значение (True или False). Эти значения предсказывают, есть ли на изображении содержимое для взрослых или непристойного характера. Передав на API проверяемое изображение (файл или URL-адрес), вы получите ответ со следующими сведениями.

```json
"ImageModeration": {
    .............
    "adultClassificationScore": 0.019196987152099609,
    "isImageAdultClassified": false,
    "racyClassificationScore": 0.032390203326940536,
    "isImageRacyClassified": false,
    ............
    ],
```

> [!NOTE]
> 
> - `isImageAdultClassified` обозначает потенциальное наличие изображений, которые в некоторых обстоятельствах могут считаться сексуально откровенными или предназначенными только для взрослых.
> - `isImageRacyClassified` обозначает потенциальное наличие изображений, которые в некоторых обстоятельствах могут считаться сексуальными или не предназначенными для детей.
> - Оценка выражается числом в диапазоне от 0 до 1. Чем выше оценка, тем более подходящей модель считает соответствующую категорию. Эта предварительная версия использует статистическую модель прогнозирования, а не оценки, кодированные вручную. Корпорация Майкрософт рекомендует протестировать ее на своих данных, чтобы проверить применимость анализа по каждой категории.
> - Логические параметры принимают значения True или False в зависимости от внутренних пороговых значений оценки. Клиенты могут на выбор использовать значения по умолчанию или настраивать собственные пороги в соответствии с действующими политиками.

## <a name="detecting-text-with-optical-character-recognition-ocr"></a>Поиск текста через оптическое распознавание символов (OCR)

Операция **оптического распознавания символов (OCR)**, помимо прочего, умеет прогнозировать наличие текстового содержимого на изображении и извлекать этот текст для модерации. Вы можете указать язык для анализа. Если язык не указан, обнаружение по умолчанию использует английский.

Ответ содержит следующие данные:
- Исходный текст.
- Обнаруженные текстовые элементы с оценками достоверности.

Пример извлечения:

```json
"TextDetection": {
    "status": {
        "code": 3000.0,
        "description": "OK",
        "exception": null
    },
    .........
    "language": "eng",
    "text": "IF WE DID \r\nALL \r\nTHE THINGS \r\nWE ARE \r\nCAPABLE \r\nOF DOING, \r\nWE WOULD \r\nLITERALLY \r\nASTOUND \r\nOURSELVE \r\n",
    "candidates": []
},
```

## <a name="detecting-faces"></a>Обнаружение лиц

Обнаружение лиц помогает обнаруживать персональные данные, такие как лица в образах. Для каждого изображения возвращаются потенциальные лица и их количество на этом изображении.

Ответ включает такие сведения.

- Количество лиц.
- Список расположений, в которых обнаружены лица.

Пример извлечения:

```json
"FaceDetection": {
    ......
    "result": true,
    "count": 2,
    "advancedInfo": [
        .....
    ],
    "faces": [
        {
            "bottom": 598,
            "left": 44,
            "right": 268,
            "top": 374
        },
        {
            "bottom": 620,
            "left": 308,
            "right": 532,
            "top": 396
        }
    ]
}
```

## <a name="creating-and-managing-custom-lists"></a>Создание пользовательских списков и управление ими

В многих Интернет-сообществах загруженные изображения и другое содержимое оскорбительного характера могут многократно дублироваться другими пользователями в течение нескольких дней, недель или месяцев. Постоянное сканирование и удаление одного изображения или слегка измененных его версий из многих мест влечет большие затраты и может стать источником ошибок.

Вместо того, чтобы многократно удалять одно и то же изображение, вам достаточно лишь добавить оскорбительное изображение в пользовательский список блокируемого содержимого. После этого система модерации контента будет сравнивать все поступающие изображения с примерами в этом списке, эффективно прекращая любое распространение таких изображений.

> [!NOTE]
> Существует максимальное ограничение в **5 списков изображений**, каждый из которых может содержать **не более 10 000 изображений**.
>

Content Moderator предоставляет полнофункциональный [API управления списками изображений](try-image-list-api.md) с операциями для управления списками пользовательских изображений. Начните работу с изучения [консоли API для списков изображений](try-image-list-api.md) и примеров кода для REST API. Также изучите [краткое руководство по спискам изображений для .NET](image-lists-quickstart-dotnet.md), если вы уже знакомы с C# и Visual Studio.

## <a name="matching-against-your-custom-lists"></a>Проверка по настраиваемым спискам

Операция Match позволяет обнаруживать нечеткие соответствия входящих изображений по любому из настраиваемых списков, созданных и управляемых с помощью операций интерфейса List.

Если найдено соответствие, операция возвращает идентификатор и теги модерации для найденного изображения. Ответ включает такие сведения.

- Оценка совпадения (в диапазоне от 0 до 1).
- Изображение, для которого найдено совпадение.
- Теги изображения (присвоенные при предыдущих операциях модерации).
- Метки изображения.

Пример извлечения:

```json
{
    ..............,
    "IsMatch": true,
    "Matches": [
        {
            "Score": 1.0,
            "MatchId": 169490,
            "Source": "169642",
            "Tags": [],
            "Label": "Sports"
        }
    ],
    ....
}
```

## <a name="review-tool"></a>Средство проверки

Для более ограниченных вариантов используйте средство Content Moderatorной [проверки](Review-Tool-User-Guide/human-in-the-loop.md) и его API-интерфейс для отображения результатов и содержимого в ходе проверки для модераторов. Они смогут проверить теги, присвоенные системой и принять окончательное решение.

![Проверка изображений модераторами-пользователями](images/moderation-reviews-quickstart-dotnet.PNG)

## <a name="next-steps"></a>Дальнейшие действия

Проверьте в работе [консоль API для списков изображений](try-image-api.md) и примеры кода для REST API. Ознакомьтесь также с [обзорами, рабочими процессами и заданиями](./review-api.md) , чтобы узнать, как настроить отзывы пользователей.
