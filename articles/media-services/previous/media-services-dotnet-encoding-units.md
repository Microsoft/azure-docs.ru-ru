---
title: Масштабирование обработки мультимедийных данных путем добавления единиц кодирования в Azure | Документация Майкрософт
description: В этой статье показано, как добавить единицы кодирования с помощью .NET служб мультимедиа Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 33f7625a-966a-4f06-bc09-bccd6e2a42b5
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.reviewer: milangada
ms.custom: devx-track-csharp
ms.openlocfilehash: 6feb28858271f11bef0933a352f86d7545ee5bd3
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103014083"
---
# <a name="how-to-scale-encoding-with-net-sdk"></a>Масштабирование кодирования с помощью пакета SDK для .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [Портал](media-services-portal-scale-media-processing.md)
> * [.NET](media-services-dotnet-encoding-units.md)
> * [REST](/rest/api/media/operations/encodingreservedunittype)
> * [Java](https://github.com/rnrneverdies/azure-sdk-for-media-services-java-samples)
> * [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)
> 
> 

## <a name="overview"></a>Обзор
> [!IMPORTANT]
> Обязательно ознакомьтесь с этим [обзором](media-services-scale-media-processing-overview.md), чтобы получить дополнительные сведения о масштабировании обработки мультимедиа.
> 
> 

Чтобы изменить тип зарезервированных единиц и число зарезервированных единиц кодирования с помощью пакета SDK для .NET, сделайте следующее:

```csharp
IEncodingReservedUnit encodingS1ReservedUnit = _context.EncodingReservedUnits.FirstOrDefault();
encodingS1ReservedUnit.ReservedUnitType = ReservedUnitType.Basic; // Corresponds to S1
encodingS1ReservedUnit.Update();
Console.WriteLine("Reserved Unit Type: {0}", encodingS1ReservedUnit.ReservedUnitType);

encodingS1ReservedUnit.CurrentReservedUnits = 2;
encodingS1ReservedUnit.Update();

Console.WriteLine("Number of reserved units: {0}", encodingS1ReservedUnit.CurrentReservedUnits);
```

## <a name="opening-a-support-ticket"></a>Открытие запроса в службу поддержки

По умолчанию каждая учетная запись Служб мультимедиа может масштабироваться до 10 зарезервированных единиц мультимедиа S2 или S3 либо 25 зарезервированных единиц мультимедиа S1 и до 5 зарезервированных единиц потоковой передачи по требованию. Вы можете запросить более высокий предел, открыв запрос в службу поддержки.

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
