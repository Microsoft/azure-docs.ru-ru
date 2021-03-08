---
title: Экспорт и импорт регистраций центров уведомлений Azure в групповой операции | Документация Майкрософт
description: Узнайте, как использовать небольшую поддержку центров уведомлений для выполнения большого количества операций в концентраторе уведомлений или экспорта всех регистраций.
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 08/04/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 03/18/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 73c19b72c75d9638213f3c813a708a272cbe2a1b
ms.sourcegitcommit: 6386854467e74d0745c281cc53621af3bb201920
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102452345"
---
# <a name="export-and-import-azure-notification-hubs-registrations-in-bulk"></a>Экспорт и импорт регистраций центров уведомлений Azure в групповой операции

Существуют сценарии, в которых требуется создание или изменение большого числа регистраций в концентраторе уведомлений. Некоторые из этих сценариев представляют собой обновления тегов, которые выполняются с помощью пакетных вычислений, или миграцию существующей реализации push-уведомлений для использования концентраторов уведомления Azure.

В этой статье объясняется, как выполнить большое количество операций в концентраторе уведомлений или экспортировать все регистрации.

## <a name="high-level-flow"></a>Поток высокого уровня

Пакетная поддержка обеспечивает долговременные задания, связанные с миллионами регистраций. Для достижения этого масштаба Пакетная поддержка использует службу хранилища Azure для хранения сведений о заданиях и выходных данных. Для операций пакетного обновления пользователю требуется создать файл в контейнере больших двоичных объектов, содержимое которого является списком регистрации операций обновления. При запуске задания пользователь предоставляет URL-адрес для входного большого двоичного объекта вместе с URL-адресом для выходного каталога (а также для контейнера больших двоичных объектов). После запуска задания пользователь может проверить состояние, запросив URL-адрес, указанный при запуске задания. Конкретное задание может выполнять только операции определенного типа (создание, обновление или удаление). Операции экспорта выполняются аналогично.

## <a name="import"></a>Импорт

### <a name="set-up"></a>Настройка

В этом разделе предполагается, что у вас есть следующие сущности:

- Провизионированный концентратор уведомлений.
- Контейнер больших двоичных объектов хранилища Azur.
- Ссылки на пакет NuGet для пакета NuGet службы [хранилища Azure](https://www.nuget.org/packages/windowsazure.storage/) и для [концентраторов уведомлений](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/).

### <a name="create-input-file-and-store-it-in-a-blob"></a>Создание входного файла и сохранение его в контейнере больших двоичных объектов

Входной файл содержит список регистраций, сериализованных в XML-коде, по одной на строку. С помощью пакета Azure SDK в следующем примере кода показано, как выполнить сериализацию регистраций и передать их в контейнер больших двоичных объектов:

```csharp
private static void SerializeToBlob(CloudBlobContainer container, RegistrationDescription[] descriptions)
{
    StringBuilder builder = new StringBuilder();
    foreach (var registrationDescription in descriptions)
    {
        builder.AppendLine(registrationDescription.Serialize());
    }

    var inputBlob = container.GetBlockBlobReference(INPUT_FILE_NAME);
    using (MemoryStream stream = new MemoryStream(Encoding.UTF8.GetBytes(builder.ToString())))
    {
        inputBlob.UploadFromStream(stream);
    }
}
```

> [!IMPORTANT]
> Предыдущий код сериализует регистрации в памяти и затем передает весь поток в контейнер больших двоичных объектов. Если вы отправили файл более чем через несколько мегабайт, ознакомьтесь с рекомендациями по использованию BLOB-объектов Azure, чтобы выполнить эти действия. Например, [блочные BLOB-объекты](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs).

### <a name="create-url-tokens"></a>Создание токенов URL-адресов

После отправки входного файла создайте URL-адреса для предоставления концентратору уведомлений как для входного файла, так и для выходного каталога. Для ввода и вывода можно использовать два разных контейнера больших двоичных объектов.

```csharp
static Uri GetOutputDirectoryUrl(CloudBlobContainer container)
{
    SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
    {
        SharedAccessExpiryTime = DateTime.UtcNow.AddDays(1),
        Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List | SharedAccessBlobPermissions.Read
    };

    string sasContainerToken = container.GetSharedAccessSignature(sasConstraints);
    return new Uri(container.Uri + sasContainerToken);
}

static Uri GetInputFileUrl(CloudBlobContainer container, string filePath)
{
    SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
    {
        SharedAccessExpiryTime = DateTime.UtcNow.AddDays(1),
        Permissions = SharedAccessBlobPermissions.Read
    };
    string sasToken = container.GetBlockBlobReference(filePath).GetSharedAccessSignature(sasConstraints);
    return new Uri(container.Uri + "/" + filePath + sasToken);
}
```

### <a name="submit-the-job"></a>отправить задание.

Теперь с двумя входными и выходными URL-адресами можно запустить пакетное задание.

```csharp
NotificationHubClient client = NotificationHubClient.CreateClientFromConnectionString(CONNECTION_STRING, HUB_NAME);
var createTask = client.SubmitNotificationHubJobAsync(
new NotificationHubJob {
        JobType = NotificationHubJobType.ImportCreateRegistrations,
        OutputContainerUri = outputContainerSasUri,
        ImportFileUri = inputFileSasUri
    }
);
createTask.Wait();

var job = createTask.Result;
long i = 10;
while (i > 0 && job.Status != NotificationHubJobStatus.Completed)
{
    var getJobTask = client.GetNotificationHubJobAsync(job.JobId);
    getJobTask.Wait();
    job = getJobTask.Result;
    Thread.Sleep(1000);
    i--;
}
```

В дополнение к входным и выходным URL-адресам в этом примере создается `NotificationHubJob` объект, содержащий `JobType` объект, который может иметь один из следующих типов:

- `ImportCreateRegistrations`
- `ImportUpdateRegistrations`
- `ImportDeleteRegistrations`

После завершения вызова задание продолжится концентратором уведомлений, и его состояние можно проверить с помощью вызова [жетнотификатионхубжобасинк](/dotnet/api/microsoft.azure.notificationhubs.notificationhubclient.getnotificationhubjobasync).

По завершении выполнения задания можно проверить результаты, просмотрев следующие файлы в выходном каталоге:

- `/<hub>/<jobid>/Failed.txt`
- `/<hub>/<jobid>/Output.txt`

Эти файлы содержат список успешных и неудачных операций из пакета. Формат файла `.cvs` , в котором каждая строка имеет номер строки исходного входного файла, и выходные данные операции (обычно это созданное или обновленное описание регистрации).

### <a name="full-sample-code"></a>Полный пример кода

В следующем примере кода выполняется импорт регистраций в концентратор уведомлений.

```csharp
using Microsoft.Azure.NotificationHubs;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using System;
using System.Collections.Generic;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Runtime.Serialization;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using System.Xml;

namespace ConsoleApplication1
{
    class Program
    {
        private static string CONNECTION_STRING = "---";
        private static string HUB_NAME = "---";
        private static string INPUT_FILE_NAME = "CreateFile.txt";
        private static string STORAGE_ACCOUNT = "---";
        private static string STORAGE_PASSWORD = "---";
        private static StorageUri STORAGE_ENDPOINT = new StorageUri(new Uri("---"));

        static void Main(string[] args)
        {
            var descriptions = new[]
            {
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMkUxREQFBlVTTkMwMQ"),
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMjUxREQFBlVTTkMwMQ"),
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMhUxREQFBlVTTkMwMQ"),
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMdUxREQFBlVTTkMwMQ"),
            };

            // Write to blob store to create an input file
            var blobClient = new CloudBlobClient(STORAGE_ENDPOINT, new Microsoft.WindowsAzure.Storage.Auth.StorageCredentials(STORAGE_ACCOUNT, STORAGE_PASSWORD));
            var container = blobClient.GetContainerReference("testjobs");
            container.CreateIfNotExists();

            SerializeToBlob(container, descriptions);

            // TODO then create Sas
            var outputContainerSasUri = GetOutputDirectoryUrl(container);
            var inputFileSasUri = GetInputFileUrl(container, INPUT_FILE_NAME);


            // Import this file
            NotificationHubClient client = NotificationHubClient.CreateClientFromConnectionString(CONNECTION_STRING, HUB_NAME);
            var createTask = client.SubmitNotificationHubJobAsync(
                new NotificationHubJob {
                    JobType = NotificationHubJobType.ImportCreateRegistrations,
                    OutputContainerUri = outputContainerSasUri,
                    ImportFileUri = inputFileSasUri
                }
            );
            createTask.Wait();

            var job = createTask.Result;
            long i = 10;
            while (i > 0 && job.Status != NotificationHubJobStatus.Completed)
            {
                var getJobTask = client.GetNotificationHubJobAsync(job.JobId);
                getJobTask.Wait();
                job = getJobTask.Result;
                Thread.Sleep(1000);
                i--;
            }
        }

        private static void SerializeToBlob(CloudBlobContainer container, RegistrationDescription[] descriptions)
        {
            StringBuilder builder = new StringBuilder();
            foreach (var registrationDescription in descriptions)
            {
                builder.AppendLine(registrationDescription.Serialize());
            }

            var inputBlob = container.GetBlockBlobReference(INPUT_FILE_NAME);
            using (MemoryStream stream = new MemoryStream(Encoding.UTF8.GetBytes(builder.ToString())))
            {
                inputBlob.UploadFromStream(stream);
            }
        }

        static Uri GetOutputDirectoryUrl(CloudBlobContainer container)
        {
            // Set the expiry time and permissions for the container.
            // In this case no start time is specified, so the shared access signature becomes valid immediately.
            SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
            {
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(4),
                Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List | SharedAccessBlobPermissions.Read
            };

            // Generate the shared access signature on the container, setting the constraints directly on the signature.
            string sasContainerToken = container.GetSharedAccessSignature(sasConstraints);

            // Return the URI string for the container, including the SAS token.
            return new Uri(container.Uri + sasContainerToken);
        }

        static Uri GetInputFileUrl(CloudBlobContainer container, string filePath)
        {
            // Set the expiry time and permissions for the container.
            // In this case no start time is specified, so the shared access signature becomes valid immediately.
            SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
            {
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(4),
                Permissions = SharedAccessBlobPermissions.Read
            };

            // Generate the shared access signature on the container, setting the constraints directly on the signature.
            string sasToken = container.GetBlockBlobReference(filePath).GetSharedAccessSignature(sasConstraints);

            // Return the URI string for the container, including the SAS token.
            return new Uri(container.Uri + "/" + filePath + sasToken);
        }

        static string GetJobPath(string namespaceName, string notificationHubPath, string jobId)
        {
            return string.Format(CultureInfo.InvariantCulture, @"{0}//{1}/{2}/", namespaceName, notificationHubPath, jobId);
        }
    }
}
```

## <a name="export"></a>Экспорт

Регистрация экспорта похожа на регистрацию импорта, но имеет следующие отличия:

- Необходим только выходной URL-адрес.
- Вы создаете Нотификатионхубжоб типа Експортрегистратионс.

### <a name="sample-code-snippet"></a>Пример фрагмента кода

Ниже приведен пример фрагмента кода для экспорта регистраций в Java.

```java
// Submit an export job
NotificationHubJob job = new NotificationHubJob();
job.setJobType(NotificationHubJobType.ExportRegistrations);
job.setOutputContainerUri("container uri with SAS signature");
job = hub.submitNotificationHubJob(job);

// Wait until the job is done
while(true){
    Thread.sleep(1000);
    job = hub.getNotificationHubJob(job.getJobId());
    if(job.getJobStatus() == NotificationHubJobStatus.Completed)
        break;
}

```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о регистрациях см. в следующих статьях:

- [Управление регистрацией](notification-hubs-push-notification-registration-management.md)
- [Теги для регистрации](notification-hubs-tags-segment-push-message.md)
- [Регистрация шаблонов](notification-hubs-templates-cross-platform-push-messages.md)
