---
title: Как скомпоновать два или более видеофайла с помощью .NET | Документация Майкрософт
description: В этой статье показано, как скомпоновать два или более видеофайла.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: how-to
ms.date: 03/24/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 5eacc366961d3101a7eaf8877a34ef2d462ea76b
ms.sourcegitcommit: bed20f85722deec33050e0d8881e465f94c79ac2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105111484"
---
# <a name="how-to-stitch-two-or-more-video-files-with-net"></a>Как скомпоновать два или более видеофайла с помощью .NET

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

## <a name="stitch-two-or-more-video-files"></a>Сшивка двух или более видеофайлов

В следующем примере показано, как можно создать предустановку для сшивания двух или более видеофайлов. Наиболее распространенный сценарий — это добавление названия или трейлера к основному видео.

> [!NOTE]
> Файлы видео, измененные вместе, должны совместно использовать свойства (разрешение видео, частота кадров, число звуковых дорожек и т. д.). Не следует комбинировать видео с разной частотой кадров или разным количеством аудиодорожек.

## <a name="prerequisites"></a>Предварительные требования

Клонирование или скачивание [примеров .NET для служб мультимедиа](https://github.com/Azure-Samples/media-services-v3-dotnet/).  Код, указанный ниже, находится в [папке енкодингвисмескустомститчтвоассетс](https://github.com/Azure-Samples/media-services-v3-dotnet/blob/main/VideoEncoding/EncodingWithMESCustomStitchTwoAssets/Program.cs)

### <a name="net-code"></a>Код .NET

[!code-csharp[Main](../../../media-services-v3-dotnet/VideoEncoding/EncodingWithMESCustomStitchTwoAssets/Program.cs)]
