---
title: Опрос долговременных операций | Документация Майкрософт
description: Службы мультимедиа Azure предлагают интерфейсы API, которые отправляют запросы к службам мультимедиа для запуска операций (например, создание, запуск, завершение или удаление канала). Эти операции выполняются долго. В этом разделе содержатся сведения об опросе долговременных операций.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
writer: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 7926f7aaa427d49d13cab5e13f5153bcd22e5898
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103013828"
---
# <a name="delivering-live-streaming-with-azure-media-services"></a>Доставка динамической потоковой передачи с помощью служб мультимедиа Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Обзор

Службы Microsoft Azure Media Services предоставляют интерфейсы API, которые отправляют запросы в службы мультимедиа на запуск операций (например, создание, запуск, остановка или удаление канала). Эти операции являются долговременными.

Пакет SDK .NET служб мультимедиа предоставляет интерфейсы API, которые отправляют запрос и ожидают завершения операции (на внутреннем уровне API выполняют опрос о ходе операции через определенные промежутки времени). Например, при вызове channel.Start() метод возвращается после запуска канала. Кроме того, вы можете использовать асинхронную версию: await channel.StartAsync() (сведения об асинхронном шаблоне на основе задач см. в статье [TAP](./media-services-mes-schema.md)). API, которые отправляют запрос на операцию, а затем запрашивают состояние до завершения операции, называются методами опроса. Такие методы (особенно асинхронная версия) рекомендуются для полнофункциональных клиентских приложений и (или) служб с отслеживанием состояния.

Существуют сценарии, когда приложение не может ждать долговременного HTTP-запроса и хочет запросить ход выполнения операции вручную. Типичным примером может служить браузер, взаимодействующий со службой без учета состояния: когда браузер запрашивает создание канала, веб-служба инициирует долговременную операцию и возвращает идентификатор операции в браузер. Затем браузер может запросить в веб-службе состояние операции на основе идентификатора Пакет SDK .NET служб мультимедиа предоставляет интерфейсы API, полезные для этого сценария. Пакет SDK .NET служб мультимедиа предоставляет интерфейсы API, полезные для этого сценария. Такие API называются неопросными методами.
"Неопросные методы" именуются по такому шаблону: Send *имя_операции* Operation (например, SendCreateOperation). Методы Send *имя_операции* Operation возвращают объект **IOperation**. Полученный объект содержит сведения, которые можно использовать для отслеживания операции. Методы Send *имя_операции* OperationAsync возвращают **Task\<IOperation>**.

В настоящее время неопросные методы поддерживаются такими классами: **Channel**, **StreamingEndpoint** и **Program**.

Чтобы опросить состояние операции, используйте метод **GetOperation** в классе **OperationBaseCollection**. Для проверки состояния операции используйте такие интервалы: для операций **Channel** и **StreamingEndpoint** — 30 секунд, для операций **Program** — 10 секунд.

## <a name="create-and-configure-a-visual-studio-project"></a>Создание и настройка проекта Visual Studio

Настройте среду разработки и заполните файл app.config данными о соединении, как описано в разделе [Разработка служб мультимедиа с помощью .NET](media-services-dotnet-how-to-use.md).

## <a name="example"></a>Пример

Пример ниже определяет класс, который называется **ChannelOperations**. Это определение класса может быть стартовой точкой для определения класса веб-службы. Для простоты в примерах ниже используются синхронные версии методов.

В примере также показано, как клиент может использовать этот класс.

### <a name="channeloperations-class-definition"></a>Определение класса ChannelOperations

```csharp
using Microsoft.WindowsAzure.MediaServices.Client;
using System;
using System.Collections.Generic;
using System.Configuration;
using System.Net;

/// <summary> 
/// The ChannelOperations class only implements 
/// the Channel’s creation operation. 
/// </summary> 
public class ChannelOperations
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

    // Field for service context.
    private static CloudMediaContext _context = null;

    public ChannelOperations()
    {
        AzureAdTokenCredentials tokenCredentials = 
            new AzureAdTokenCredentials(_AADTenantDomain,
                new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                AzureEnvironments.AzureCloudEnvironment);

        var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

        _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);
    }

    /// <summary>  
    /// Initiates the creation of a new channel.  
    /// </summary>  
    /// <param name="channelName">Name to be given to the new channel</param>  
    /// <returns>  
    /// Operation Id for the long running operation being executed by Media Services. 
    /// Use this operation Id to poll for the channel creation status. 
    /// </returns> 
    public string StartChannelCreation(string channelName)
    {
        var operation = _context.Channels.SendCreateOperation(
            new ChannelCreationOptions
            {
                Name = channelName,
                Input = CreateChannelInput(),
                Preview = CreateChannelPreview(),
                Output = CreateChannelOutput()
            });

        return operation.Id;
    }

    /// <summary> 
    /// Checks if the operation has been completed. 
    /// If the operation succeeded, the created channel Id is returned in the out parameter.
    /// </summary> 
    /// <param name="operationId">The operation Id.</param> 
    /// <param name="channel">
    /// If the operation succeeded, 
    /// the created channel Id is returned in the out parameter.</param>
    /// <returns>Returns false if the operation is still in progress; otherwise, true.</returns> 
    public bool IsCompleted(string operationId, out string channelId)
    {
        IOperation operation = _context.Operations.GetOperation(operationId);
        bool completed = false;

        channelId = null;

        switch (operation.State)
        {
            case OperationState.Failed:
                // Handle the failure. 
                // For example, throw an exception. 
                // Use the following information in the exception: operationId, operation.ErrorMessage.
                break;
            case OperationState.Succeeded:
                completed = true;
                channelId = operation.TargetEntityId;
                break;
            case OperationState.InProgress:
                completed = false;
                break;
        }
        return completed;
    }

    private static ChannelInput CreateChannelInput()
    {
        return new ChannelInput
        {
            StreamingProtocol = StreamingProtocol.RTMP,
            AccessControl = new ChannelAccessControl
            {
                IPAllowList = new List<IPRange>
                    {
                        new IPRange
                        {
                            Name = "TestChannelInput001",
                            Address = IPAddress.Parse("0.0.0.0"),
                            SubnetPrefixLength = 0
                        }
                    }
            }
        };
    }

    private static ChannelPreview CreateChannelPreview()
    {
        return new ChannelPreview
        {
            AccessControl = new ChannelAccessControl
            {
                IPAllowList = new List<IPRange>
                    {
                        new IPRange
                        {
                            Name = "TestChannelPreview001",
                            Address = IPAddress.Parse("0.0.0.0"),
                            SubnetPrefixLength = 0
                        }
                    }
            }
        };
    }

    private static ChannelOutput CreateChannelOutput()
    {
        return new ChannelOutput
        {
            Hls = new ChannelOutputHls { FragmentsPerSegment = 1 }
        };
    }
}
```

### <a name="the-client-code"></a>Код клиента

```csharp
ChannelOperations channelOperations = new ChannelOperations();
string opId = channelOperations.StartChannelCreation("MyChannel001");

string channelId = null;
bool isCompleted = false;

while (isCompleted == false)
{
    System.Threading.Thread.Sleep(TimeSpan.FromSeconds(30));
    isCompleted = channelOperations.IsCompleted(opId, out channelId);
}

// If we got here, we should have the newly created channel id.
Console.WriteLine(channelId);
```

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]