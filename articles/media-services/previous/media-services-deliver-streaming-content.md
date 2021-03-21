---
title: Публикация содержимого служб мультимедиа Azure с помощью .NET | Документация Майкрософт
description: Узнайте, как создать указатель для создания URL-адреса потоковой передачи. Примеры кода написаны на языке C# и используют пакет SDK служб мультимедиа для .NET.
author: IngridAtMicrosoft
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: c53b1f83-4cb1-4b09-840f-9c145b7d6f8d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: a1a06c9a5fdac1082a57bee71cccad271a6aae81
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103015800"
---
# <a name="publish-media-services-content-using-net"></a>Публикация содержимого Служб мультимедиа с помощью .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [REST](media-services-rest-deliver-streaming-content.md)
> * [.NET](media-services-deliver-streaming-content.md)
> * [Портал](media-services-portal-publish.md)
> 
> 

## <a name="overview"></a>Обзор
Вы можете осуществлять потоковую передачу набора MP4-файлов с адаптивной скоростью, создавая указатель потоковой передачи OnDemand и формируя URL-адрес потоковой передачи. В разделе, посвященном [кодированию ресурса](media-services-encode-asset.md) , описано кодирование данных в набор MP4-файлов с переменной скоростью. 

> [!NOTE]
> Если содержимое шифруется, то перед созданием указателя настройте политику доставки ресурсов-контейнеров (как описано в [этой](media-services-dotnet-configure-asset-delivery-policy.md) статье). 
> 
> 

Можно также использовать указатель потоковой передачи OnDemand, чтобы формировать URL-адреса, указывающие на MP4-файлы для последовательного скачивания.  

В этом разделе показано, как создать указатель потоковой передачи OnDemand, чтобы опубликовать файл и получить URL-адреса потоковой передачи Smooth, MPEG DASH и HLS. В нем также показывается, как создать URL-адреса последовательного скачивания. 

## <a name="create-an-ondemand-streaming-locator"></a>Создание указателя потоковой передачи OnDemand
Чтобы создать указатель потоковой передачи OnDemand и получить URL-адреса, вам нужно сделать следующее:

1. Если содержимое шифруется, установите политику доступа.
2. создать указатель потоковой передачи OnDemand.
3. Если планируется потоковая передача, получите файл манифеста потоковой передачи (ISM-файл) в ресурсе. 
   
   Если вы планируете последовательное скачивание, получите имена MP4-файлов в ресурсе.  
4. Создайте URL-адрес файла манифеста или URL-адреса MP4-файлов. 


>[!NOTE]
>Действует ограничение в 1 000 000 записей для разных политик AMS (например, для политики Locator или ContentKeyAuthorizationPolicy). Следует указывать один и тот же идентификатор политики, если вы используете те же дни, разрешения доступа и т. д. Речь идет, например, о политиках для указателей, которые должны оставаться на месте в течение длительного времени (не политики передачи). Чтобы узнать больше, ознакомьтесь с [этим](media-services-dotnet-manage-entities.md#limit-access-policies) разделом.

### <a name="use-media-services-net-sdk"></a>Использование пакета SDK служб мультимедиа для .NET
Создание URL-адресов потоковой передачи 

```csharp
    private static void BuildStreamingURLs(IAsset asset)
    {

        // Create a 30-day readonly access policy. 
          // You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.
        IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
            TimeSpan.FromDays(30),
            AccessPermissions.Read);

        // Create a locator to the streaming content on an origin. 
        ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
            policy,
            DateTime.UtcNow.AddMinutes(-5));

        // Display some useful values based on the locator.
        Console.WriteLine("Streaming asset base path on origin: ");
        Console.WriteLine(originLocator.Path);
        Console.WriteLine();

        // Get a reference to the streaming manifest file from the  
        // collection of files in the asset. 
        var manifestFile = asset.AssetFiles.ToList().Where(f => f.Name.ToLower().
                                    EndsWith(".ism")).
                                    FirstOrDefault();

        // Create a full URL to the manifest file. Use this for playback
        // in streaming media clients. 
        string urlForClientStreaming = originLocator.Path + manifestFile.Name + "/manifest";
        Console.WriteLine("URL to manifest for client streaming using Smooth Streaming protocol: ");
        Console.WriteLine(urlForClientStreaming);
        Console.WriteLine("URL to manifest for client streaming using HLS protocol: ");
        Console.WriteLine(urlForClientStreaming + "(format=m3u8-aapl)");
        Console.WriteLine("URL to manifest for client streaming using MPEG DASH protocol: ");
        Console.WriteLine(urlForClientStreaming + "(format=mpd-time-csf)"); 
        Console.WriteLine();
    }
```

Выходные данные:

- URL-адрес манифеста для потоковой передачи клиента с помощью протокола Smooth Streaming: \
  `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest`
- URL-адрес манифеста для потоковой передачи клиента с помощью протокола HLS: \
  `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=m3u8-aapl)`
- URL-адрес манифеста для потоковой передачи клиента по протоколу MPEG: \
  `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=mpd-time-csf)`

> [!NOTE]
> Вы также можете выполнять потоковую передачу содержимого через TLS-подключение. Для этого убедитесь, что URL-адреса потоковой передачи начинаются с HTTPS. В настоящее время AMS не поддерживает TLS с пользовательскими доменами.
> 
> 

Создание URL-адресов последовательного скачивания 

```csharp
    private static void BuildProgressiveDownloadURLs(IAsset asset)
    {
        // Create a 30-day readonly access policy. 
        IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
            TimeSpan.FromDays(30),
            AccessPermissions.Read);

        // Create an OnDemandOrigin locator to the asset. 
        ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
            policy,
            DateTime.UtcNow.AddMinutes(-5));

        // Display some useful values based on the locator.
        Console.WriteLine("Streaming asset base path on origin: ");
        Console.WriteLine(originLocator.Path);
        Console.WriteLine();

        // Get MP4 files.
        IEnumerable<IAssetFile> mp4AssetFiles = asset
            .AssetFiles
            .ToList()
            .Where(af => af.Name.EndsWith(".mp4", StringComparison.OrdinalIgnoreCase));

        // Create a full URL to the MP4 files. Use this to progressively download files.
        foreach (var pd in mp4AssetFiles)
            Console.WriteLine(originLocator.Path + pd.Name);
    }
```
Выходные данные:

- `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4`
- `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4`
- `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4`
- `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4`

    . . . 

### <a name="use-media-services-net-sdk-extensions"></a>Использование расширения пакета SDK служб мультимедиа для .NET
Следующий код вызывает методы расширения пакета SDK для .NET, которые создают указатель и генерируют URL-адреса Smooth Streaming, HLS и MPEG-DASH для адаптивной потоковой передачи.
```csharp
    // Create a loctor.
    _context.Locators.Create(
        LocatorType.OnDemandOrigin,
        inputAsset,
        AccessPermissions.Read,
        TimeSpan.FromDays(30));

    // Get the streaming URLs.
    Uri smoothStreamingUri = inputAsset.GetSmoothStreamingUri();
    Uri hlsUri = inputAsset.GetHlsUri();
    Uri mpegDashUri = inputAsset.GetMpegDashUri();

    Console.WriteLine(smoothStreamingUri);
    Console.WriteLine(hlsUri);
    Console.WriteLine(mpegDashUri);
```

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Следующие шаги
* [Скачивание ресурсов](media-services-deliver-asset-download.md)
* [Настройка политики доставки ресурсов](media-services-dotnet-configure-asset-delivery-policy.md)

