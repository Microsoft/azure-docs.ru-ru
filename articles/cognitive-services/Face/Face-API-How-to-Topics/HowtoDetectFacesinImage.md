---
title: Обнаружение лиц на изображении
titleSuffix: Azure Cognitive Services
description: В этом руководстве показано, как использовать обнаружение лиц для извлечения таких атрибутов, как Gender, Age или of a, из определенного изображения.
services: cognitive-services
author: SteveMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 02/23/2021
ms.author: sbowles
ms.custom: devx-track-csharp
ms.openlocfilehash: 3a15cce45c527a92c99e0488661e0b67bb8e2371
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101713071"
---
# <a name="get-face-detection-data"></a>Получение данных обнаружения лиц

В этом руководстве показано, как использовать обнаружение лиц для извлечения таких атрибутов, как Gender, Age или of a, из определенного изображения. Фрагменты кода в этом руководство написаны на языке C# с помощью клиентской библиотеки Azure Cognitive Services Face. Те же функциональные возможности доступны в [REST API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).

В этом учебнике показано, как:

- Получение расположений и размеров граней в изображении.
- Получите расположения различных ориентиров, таких как пупилс, нос и рот, в образе.
- Догадаться пол, возраст, распознавания эмоций и другие атрибуты обнаруженного лица.

## <a name="setup"></a>Настройка

В этом учебнике предполагается, что вы уже создавали объект [фацеклиент](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient) `faceClient` с именем с ключом подписки лица и URL-адресом конечной точки. Здесь можно использовать функцию обнаружения лиц, вызвав либо [детектвисурласинк](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithurlasync), которая используется в этом руководством, либо [детектвисстреамасинк](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithstreamasync). Инструкции по настройке этой функции см. в одном из кратких руководств.

В этом разделе рассматриваются особенности вызова метода обнаружения, например, какие аргументы можно передавать и что можно делать с возвращаемыми данными. Рекомендуется запрашивать только необходимые компоненты. Для выполнения каждой операции требуется дополнительное время.

## <a name="get-basic-face-data"></a>Получение основных данных о грани

Чтобы найти лица и получить их расположения в изображении, вызовите метод [детектвисурласинк](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithurlasync) или [детектвисстреамасинк](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithstreamasync) с параметром _ретурнфацеид_ , имеющим значение **true**. Это значение по умолчанию.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="basic1":::

Вы можете запросить возвращенные объекты [детектедфаце](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.detectedface) для их уникальных идентификаторов и прямоугольника, который дает координаты точки.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="basic2":::

Сведения о том, как анализировать расположение и размеры лица, см. в разделе [фацеректангле](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.facerectangle). Обычно этот прямоугольник содержит глаза, бровей, нос и рот. Верхняя часть Head, ушки и Чин не обязательно включена. Чтобы использовать прямоугольник лицевой стороны для обрезки полного заголовка или получения середины рисунка книжной ориентации, возможно, для изображения типа "идентификатор фотографии", можно развернуть прямоугольник в каждом направлении.

## <a name="get-face-landmarks"></a>Получение ориентиров для лиц

[Ориентиры](../concepts/face-detection.md#face-landmarks) — это набор удобных для поиска точек на лицевой стороне, например пупилс или Совет нос. Чтобы получить данные ориентиров, установите для параметра _детектионмодел_ значение **детектионмодел. Detection01** , а для параметра _ретурнфацеландмаркс_ — **значение true**.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="landmarks1":::

В следующем коде показано, как можно извлечь расположения нос и пупилс:

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="landmarks2":::

Можно также использовать данные ориентиров для точного вычисления направления лица. Например, можно определить поворот грани в виде вектора от центра рот до центра глаз. Следующий код вычисляет этот вектор:

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="direction":::

Если вы знакомы с направлением лица, вы можете повернуть прямоугольную рамку, чтобы правильно выстроить ее. Чтобы обрезать фрагменты изображения, можно программно повернуть изображение таким образом, чтобы они всегда отображались вертикально.

## <a name="get-face-attributes"></a>Получение атрибутов лиц

Помимо прямоугольников и ориентиров, API обнаружения лиц может анализировать несколько концептуальных атрибутов лица. Полный список см. в разделе Общие сведения о [атрибутах лиц](../concepts/face-detection.md#attributes) .

Чтобы проанализировать атрибуты лиц, задайте для параметра _детектионмодел_ значение **детектионмодел. Detection01** , а параметр _Ретурнфацеаттрибутес_ — список значений [перечисления фацеаттрибутетипе](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.faceattributetype) .

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="attributes1":::

Затем получите ссылки на возвращенные данные и выполните больше операций в соответствии с вашими потребностями.

:::code language="csharp" source="~/cognitive-services-quickstart-code/dotnet/Face/sdk/detect.cs" id="attributes2":::

Дополнительные сведения о каждом из атрибутов см. в разделе Общие сведения об [обнаружении и атрибутах лиц](../concepts/face-detection.md) .

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство вы узнали, как использовать различные функции обнаружения лиц. Затем интегрируйте эти функции в приложение, выполнив подробное руководство.

- [Руководство. Создание приложения WPF для отображения данных о лицах на изображении](../Tutorials/FaceAPIinCSharpTutorial.md)

## <a name="related-topics"></a>Связанные темы

- [Справочная документация (ОСТАВШАЯся)](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)
- [Справочная документация (пакет SDK для .NET)](/dotnet/api/overview/azure/cognitiveservices/client/faceapi)
