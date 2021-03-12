---
title: Управление конечными точками потоковой передачи с помощью пакета SDK для .NET | Документы Майкрософт
description: В этой статье показано, как управлять конечными точками потоковой передачи с помощью пакета SDK для .NET.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
writer: juliako
manager: femila
editor: ''
ms.assetid: 0da34a97-f36c-48d0-8ea2-ec12584a2215
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 5c84e7c753e6d1eb1d357320857e4c159cc13cfc
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103013760"
---
# <a name="manage-streaming-endpoints-with-net-sdk"></a>Управление конечными точками потоковой передачи с помощью пакета SDK для .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]  

>[!NOTE]
>Обязательно просмотрите [обзорную статью](media-services-streaming-endpoints-overview.md). Просмотрите также статью [Конечная точка потоковой передачи](/rest/api/media/operations/streamingendpoint).

В этой статье приведен код, в котором показано, как выполнить следующие задачи с помощью пакета SDK служб мультимедиа Azure для .NET:

- изучить конечную точку потоковой передачи по умолчанию;
- создать и добавить конечную точку потоковой передачи.

    Может потребоваться наличие нескольких конечных точек потоковой передачи, если вы планируете использовать разные сети CDN (или сеть CDN) и прямой доступ.

    > [!NOTE]
    > Плата взимается, только когда конечная точка потоковой передачи используется.
    
- Обновите конечную точку потоковой передачи.
    
    Обязательно вызовите функцию Update().

- Удалите конечную точку потоковой передачи.

    >[!NOTE]
    >Конечную точку потоковой передачи по умолчанию удалить нельзя.

Сведения о том, как масштабировать конечную точку потоковой передачи, см. в [этой](media-services-portal-scale-streaming-endpoints.md) статье.

## <a name="create-and-configure-a-visual-studio-project"></a>Создание и настройка проекта Visual Studio

Настройте среду разработки и заполните файл app.config данными о соединении, как описано в разделе [Разработка служб мультимедиа с помощью .NET](media-services-dotnet-how-to-use.md). 

## <a name="add-code-that-manages-streaming-endpoints"></a>Добавление кода, который управляет конечными точками потоковой передачи
    
Замените код файла Program.cs на код, приведенный ниже.

```csharp
using System;
using System.Configuration;
using System.Linq;
using Microsoft.WindowsAzure.MediaServices.Client;
using Microsoft.WindowsAzure.MediaServices.Client.Live;

namespace AMSStreamingEndpoint
{
    class Program
    {
        // Read values from the App.config file.

        private static readonly string _AADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _RESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];

        private static CloudMediaContext _context = null;

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            var defaultStreamingEndpoint = _context.StreamingEndpoints.Where(s => s.Name.Contains("default")).FirstOrDefault();
            ExamineStreamingEndpoint(defaultStreamingEndpoint);

            IStreamingEndpoint newStreamingEndpoint = AddStreamingEndpoint();
            UpdateStreamingEndpoint(newStreamingEndpoint);
            DeleteStreamingEndpoint(newStreamingEndpoint);
        }

        static public void ExamineStreamingEndpoint(IStreamingEndpoint streamingEndpoint)
        {
            Console.WriteLine(streamingEndpoint.Name);
            Console.WriteLine(streamingEndpoint.StreamingEndpointVersion);
            Console.WriteLine(streamingEndpoint.FreeTrialEndTime);
            Console.WriteLine(streamingEndpoint.ScaleUnits);
            Console.WriteLine(streamingEndpoint.CdnProvider);
            Console.WriteLine(streamingEndpoint.CdnProfile);
            Console.WriteLine(streamingEndpoint.CdnEnabled);
        }

        static public IStreamingEndpoint AddStreamingEndpoint()
        {
            var name = "StreamingEndpoint" + DateTime.UtcNow.ToString("hhmmss");
            var option = new StreamingEndpointCreationOptions(name, 1)
            {
                StreamingEndpointVersion = new Version("2.0"),
                CdnEnabled = true,
                CdnProfile = "CdnProfile",
                CdnProvider = CdnProviderType.PremiumVerizon
            };

            var streamingEndpoint = _context.StreamingEndpoints.Create(option);

            return streamingEndpoint;
        }

        static public void UpdateStreamingEndpoint(IStreamingEndpoint streamingEndpoint)
        {
            if (streamingEndpoint.StreamingEndpointVersion == "1.0")
                streamingEndpoint.StreamingEndpointVersion = "2.0";

            streamingEndpoint.CdnEnabled = false;
            streamingEndpoint.Update();
        }

        static public void DeleteStreamingEndpoint(IStreamingEndpoint streamingEndpoint)
        {
            streamingEndpoint.Delete();
        }
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия
Просмотрите схемы обучения работе со службами мультимедиа.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
