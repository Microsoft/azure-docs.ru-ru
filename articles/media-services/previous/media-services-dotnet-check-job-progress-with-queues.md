---
title: Использование хранилища очередей Azure для наблюдения за уведомлениями о заданиях служб мультимедиа с использованием .NET | Документация Майкрософт
description: Узнайте, как использовать хранилище очередей Azure для наблюдения за уведомлениями о заданиях служб мультимедиа. Пример кода написан на языке C# и использует пакет SDK служб мультимедиа для .NET.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: f535d0b5-f86c-465f-81c6-177f4f490987
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: dc6ab94b3685e55125032b0afa52076eae72ba5b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103016225"
---
# <a name="use-azure-queue-storage-to-monitor-media-services-job-notifications-with-net"></a>Использование хранилища очередей Azure для наблюдения за уведомлениями о заданиях служб мультимедиа с использованием .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

При выполнении заданий кодирования часто требуется способ отслеживания их хода выполнения. Можно настроить службы мультимедиа для доставки уведомлений в [хранилище очередей Azure](../../storage/queues/storage-dotnet-how-to-use-queues.md). Вы можете отслеживать ход выполнения задания, получая уведомления из хранилища очередей. 

Доступ к сообщениям, доставленным в хранилище очередей, можно получить в любой точке мира. Хранилище очередей предоставляет надежную и высокомасштабируемую архитектуру обмена сообщениями. Рекомендуется опрашивать хранилище очередей для получения сообщений, а не использовать другие методы.

Вот один из распространенных сценариев ожидания передачи уведомлений служб мультимедиа: вы разрабатываете систему управления содержимым, которая должна выполнить ряд дополнительных задач после завершения кодирования (например, активировать следующий шаг в рабочем процессе или опубликовать содержимое).

В этой статье показано, как получать уведомления из хранилища очередей.  

## <a name="considerations"></a>Рекомендации
При разработке приложений служб мультимедиа, использующих хранилище очередей, учитывайте следующее.

* Хранилище очередей не гарантирует доставку по методу FIFO (первым пришел, первым вышел). Дополнительные сведения см. в статье [Очереди Azure и очереди служебной шины: сходства и различия](/previous-versions/azure/hh767287(v=azure.100)).
* Хранилище очередей не является службой Push-уведомлений. Необходимо опрашивать очередь.
* Можно использовать любое количество очередей. Дополнительные сведения см. в статье [REST API службы очередей](/rest/api/storageservices/queue-service-rest-api).
* Хранилище очередей имеет ряд ограничений и особенностей, которые следует учитывать. Они описаны в статье [Очереди службы хранилища и очереди служебной шины: сходства и различия](../../service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted.md).

## <a name="net-code-example"></a>Пример кода .NET

Пример кода в этом разделе выполняет следующие задачи:

1. Определяет класс **EncodingJobMessage**, который сопоставляется с форматом уведомления. Код десериализует сообщения, полученные из очереди, в объекты типа **EncodingJobMessage**.
2. загружает данные учетной записи служб мультимедиа и хранилища из файла app.config; Этот пример кода использует данные сведения для создания объектов **CloudMediaContext** и **CloudQueue**.
3. Создает очередь, которая получает уведомления о задании кодирования.
4. создает конечную точку уведомлений, сопоставленную с очередью;
5. присоединяет конечную точку уведомлений к заданию и отправляет задание кодирования. К заданию может быть прикреплено несколько конечных точек уведомлений.
6. Передает **NotificationJobState.FinalStatesOnly** в метод **AddNew**. (В этом примере нас интересуют только конечные состояния обработки задания.)

    ```csharp
    job.JobNotificationSubscriptions.AddNew(NotificationJobState.FinalStatesOnly, _notificationEndPoint);
    ```

7. Если передать параметр **NotificationJobState.All**, вы получите все уведомления об изменениях следующих состояний: Queued, Scheduled, Processing, Finished. Тем не менее, как было отмечено ранее, хранилище очередей не гарантирует упорядоченной доставки. Чтобы упорядочить сообщения, можно использовать свойство **Timestamp** (определено в типе **EncodingJobMessage** в примере ниже). Возможны повторяющиеся сообщения. Используйте свойство **ETag** (определенное в типе **EncodingJobMessage**) для проверки наличия повторов. Также возможно, что некоторые уведомления об изменении состояния будут пропущены.
8. Ожидает, пока задание не перейдет в состояние завершения, проверяя очередь каждые 10 секунд. Удаляет сообщения после их обработки.
9. Удаляет очередь и конечную точку уведомлений.

> [!NOTE]
> Рекомендуемый способ наблюдения за состоянием задания — ожидание передачи уведомлений, как показано в следующем примере:
>
> Кроме того, можно проверить состояние задания с помощью свойства **IJob.State** .  Уведомление о завершении задания может поступить до того, как свойство State в **IJob** получит значение **Finished**. Свойство **IJob.State** отражает точное состояние с небольшой задержкой.
>
>

### <a name="create-and-configure-a-visual-studio-project"></a>Создание и настройка проекта Visual Studio

1. Настройте среду разработки и укажите в файле app.config сведения о подключении, как описано в статье [Разработка служб мультимедиа с помощью .NET](media-services-dotnet-how-to-use.md). 
2. Создайте папку (в любом расположении на локальном диске) и скопируйте MP4-файл, для которого нужно выполнить кодирование, потоковую передачу или поэтапное скачивание. В этом примере используется путь C:\Media.
3. Добавьте ссылку на библиотеку **System.Runtime.Serialization**.

### <a name="code"></a>Код

```csharp
using System;
using System.Linq;
using System.Configuration;
using System.IO;
using System.Threading;
using System.Collections.Generic;
using Microsoft.WindowsAzure.MediaServices.Client;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Queue;
using System.Runtime.Serialization.Json;

namespace JobNotification
{
    public class EncodingJobMessage
    {
        // MessageVersion is used for version control.
        public String MessageVersion { get; set; }

        // Type of the event. Valid values are
        // JobStateChange and NotificationEndpointRegistration.
        public String EventType { get; set; }

        // ETag is used to help the customer detect if
        // the message is a duplicate of another message previously sent.
        public String ETag { get; set; }

        // Time of occurrence of the event.
        public String TimeStamp { get; set; }

        // Collection of values specific to the event.

        // For the JobStateChange event the values are:
        //     JobId - Id of the Job that triggered the notification.
        //     NewState- The new state of the Job. Valid values are:
        //          Scheduled, Processing, Canceling, Cancelled, Error, Finished
        //     OldState- The old state of the Job. Valid values are:
        //          Scheduled, Processing, Canceling, Cancelled, Error, Finished

        // For the NotificationEndpointRegistration event the values are:
        //     NotificationEndpointId- Id of the NotificationEndpoint
        //          that triggered the notification.
        //     State- The state of the Endpoint.
        //          Valid values are: Registered and Unregistered.

        public IDictionary<string, object> Properties { get; set; }
    }

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

        private static readonly string _StorageConnectionString = 
            ConfigurationManager.AppSettings["StorageConnectionString"];

        private static CloudMediaContext _context = null;
        private static CloudQueue _queue = null;
        private static INotificationEndPoint _notificationEndPoint = null;

        private static readonly string _singleInputMp4Path =
            Path.GetFullPath(@"C:\Media\BigBuckBunny.mp4");

        static void Main(string[] args)
        {
            string endPointAddress = Guid.NewGuid().ToString();

            // Create the context.
            AzureAdTokenCredentials tokenCredentials = 
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);
            
            // Create the queue that will be receiving the notification messages.
            _queue = CreateQueue(_StorageConnectionString, endPointAddress);

            // Create the notification point that is mapped to the queue.
            _notificationEndPoint =
                    _context.NotificationEndPoints.Create(
                    Guid.NewGuid().ToString(), NotificationEndPointType.AzureQueue, endPointAddress);


            if (_notificationEndPoint != null)
            {
                IJob job = SubmitEncodingJobWithNotificationEndPoint(_singleInputMp4Path);
                WaitForJobToReachedFinishedState(job.Id);
            }

            // Clean up.
            _queue.Delete();
            _notificationEndPoint.Delete();
        }


        static public CloudQueue CreateQueue(string storageAccountConnectionString, string endPointAddress)
        {
            CloudStorageAccount storageAccount = CloudStorageAccount.Parse(storageAccountConnectionString);

            // Create the queue client
            CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

            // Retrieve a reference to a queue
            CloudQueue queue = queueClient.GetQueueReference(endPointAddress);

            // Create the queue if it doesn't already exist
            queue.CreateIfNotExists();

            return queue;
        }


        public static IJob SubmitEncodingJobWithNotificationEndPoint(string inputMediaFilePath)
        {
            // Declare a new job.
            IJob job = _context.Jobs.Create("My MP4 to Smooth Streaming encoding job");

            //Create an encrypted asset and upload the mp4.
            IAsset asset = CreateAssetAndUploadSingleFile(AssetCreationOptions.StorageEncrypted,
                inputMediaFilePath);

            // Get a media processor reference, and pass to it the name of the
            // processor to use for the specific task.
            IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

            // Create a task with the conversion details, using a configuration file.
            ITask task = job.Tasks.AddNew("My encoding Task",
                processor,
                "Adaptive Streaming",
                Microsoft.WindowsAzure.MediaServices.Client.TaskOptions.None);

            // Specify the input asset to be encoded.
            task.InputAssets.Add(asset);

            // Add an output asset to contain the results of the job.
            task.OutputAssets.AddNew("Output asset",
                AssetCreationOptions.None);

            // Add a notification point to the job. You can add multiple notification points.  
            job.JobNotificationSubscriptions.AddNew(NotificationJobState.FinalStatesOnly,
                _notificationEndPoint);

            job.Submit();

            return job;
        }

        public static void WaitForJobToReachedFinishedState(string jobId)
        {
            int expectedState = (int)JobState.Finished;
            int timeOutInSeconds = 600;

            bool jobReachedExpectedState = false;
            DateTime startTime = DateTime.Now;
            int jobState = -1;

            while (!jobReachedExpectedState)
            {
                // Specify how often you want to get messages from the queue.
                Thread.Sleep(TimeSpan.FromSeconds(10));

                foreach (var message in _queue.GetMessages(10))
                {
                    using (Stream stream = new MemoryStream(message.AsBytes))
                    {
                        DataContractJsonSerializerSettings settings = new DataContractJsonSerializerSettings();
                        settings.UseSimpleDictionaryFormat = true;
                        DataContractJsonSerializer ser = new DataContractJsonSerializer(typeof(EncodingJobMessage), settings);
                        EncodingJobMessage encodingJobMsg = (EncodingJobMessage)ser.ReadObject(stream);

                        Console.WriteLine();

                        // Display the message information.
                        Console.WriteLine("EventType: {0}", encodingJobMsg.EventType);
                        Console.WriteLine("MessageVersion: {0}", encodingJobMsg.MessageVersion);
                        Console.WriteLine("ETag: {0}", encodingJobMsg.ETag);
                        Console.WriteLine("TimeStamp: {0}", encodingJobMsg.TimeStamp);
                        foreach (var property in encodingJobMsg.Properties)
                        {
                            Console.WriteLine("    {0}: {1}", property.Key, property.Value);
                        }

                        // We are only interested in messages
                        // where EventType is "JobStateChange".
                        if (encodingJobMsg.EventType == "JobStateChange")
                        {
                            string JobId = (String)encodingJobMsg.Properties.Where(j => j.Key == "JobId").FirstOrDefault().Value;
                            if (JobId == jobId)
                            {
                                string oldJobStateStr = (String)encodingJobMsg.Properties.
                                                            Where(j => j.Key == "OldState").FirstOrDefault().Value;
                                string newJobStateStr = (String)encodingJobMsg.Properties.
                                                            Where(j => j.Key == "NewState").FirstOrDefault().Value;

                                JobState oldJobState = (JobState)Enum.Parse(typeof(JobState), oldJobStateStr);
                                JobState newJobState = (JobState)Enum.Parse(typeof(JobState), newJobStateStr);

                                if (newJobState == (JobState)expectedState)
                                {
                                    Console.WriteLine("job with Id: {0} reached expected state: {1}",
                                        jobId, newJobState);
                                    jobReachedExpectedState = true;
                                    break;
                                }
                            }
                        }
                    }
                    // Delete the message after we've read it.
                    _queue.DeleteMessage(message);
                }

                // Wait until timeout
                TimeSpan timeDiff = DateTime.Now - startTime;
                bool timedOut = (timeDiff.TotalSeconds > timeOutInSeconds);
                if (timedOut)
                {
                    Console.WriteLine(@"Timeout for checking job notification messages,
                                        latest found state ='{0}', wait time = {1} secs",
                        jobState,
                        timeDiff.TotalSeconds);

                    throw new TimeoutException();
                }
            }
        }

        static private IAsset CreateAssetAndUploadSingleFile(AssetCreationOptions assetCreationOptions, string singleFilePath)
        {
            var asset = _context.Assets.Create("UploadSingleFile_" + DateTime.UtcNow.ToString(),
                assetCreationOptions);

            var fileName = Path.GetFileName(singleFilePath);

            var assetFile = asset.AssetFiles.Create(fileName);

            Console.WriteLine("Created assetFile {0}", assetFile.Name);
            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading of {0}", assetFile.Name);

            return asset;
        }

        static private IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
        {
            var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
                ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

            if (processor == null)
                throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

            return processor;
        }
    }
}
```

Приведенный выше пример формирует указанные ниже выходные данные. Фактические значения могут отличаться.

```output
Created assetFile BigBuckBunny.mp4
Upload BigBuckBunny.mp4
Done uploading of BigBuckBunny.mp4

EventType: NotificationEndPointRegistration
MessageVersion: 1.0
ETag: e0238957a9b25bdf3351a88e57978d6a81a84527fad03bc23861dbe28ab293f6
TimeStamp: 2013-05-14T20:22:37
    NotificationEndPointId: nb:nepid:UUID:d6af9412-2488-45b2-ba1f-6e0ade6dbc27
    State: Registered
    Name: dde957b2-006e-41f2-9869-a978870ac620
    Created: 2013-05-14T20:22:35

EventType: JobStateChange
MessageVersion: 1.0
ETag: 4e381f37c2d844bde06ace650310284d6928b1e50101d82d1b56220cfcb6076c
TimeStamp: 2013-05-14T20:24:40
    JobId: nb:jid:UUID:526291de-f166-be47-b62a-11ffe6d4be54
    JobName: My MP4 to Smooth Streaming encoding job
    NewState: Finished
    OldState: Processing
    AccountName: westeuropewamsaccount
job with Id: nb:jid:UUID:526291de-f166-be47-b62a-11ffe6d4be54 reached expected
State: Finished
```

## <a name="next-step"></a>Следующий шаг
Просмотрите схемы обучения работе со службами мультимедиа.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
