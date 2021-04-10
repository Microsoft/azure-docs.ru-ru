---
title: Передача больших объемов случайных данных в параллельном режиме в службу хранилища Azure
description: Узнайте, как использовать клиентскую библиотеку службы хранилища Azure для передачи больших объемов случайных данных в параллельном режиме в учетную запись службы хранилища Azure.
author: roygara
ms.service: storage
ms.topic: tutorial
ms.date: 02/04/2021
ms.author: rogarana
ms.subservice: blobs
ms.openlocfilehash: ed7020a58f3f15403108934bcc3fab644bd1b627
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99584470"
---
# <a name="upload-large-amounts-of-random-data-in-parallel-to-azure-storage"></a>Передача больших объемов случайных данных в параллельном режиме в службу хранилища Azure

Это руководство представляет собой вторую часть цикла. В этом руководстве рассматривается развертывание приложения, которое передает большой объем случайных данных в учетную запись хранения Azure.

Из второй части цикла вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Настройка строки подключения
> * создание приложения;
> * Выполнение приложения
> * Проверка количества подключений.

Хранилище BLOB-объектов Microsoft Azure — это масштабируемая служба для хранения данных. Чтобы обеспечить высокую производительность приложения, рекомендуется ознакомиться с работой хранилища BLOB-объектов. Важно понимание ограничений больших двоичных объектов Azure. Дополнительные сведения см. в статье [Scalability and performance targets for Blob storage](../blobs/scalability-targets.md) (Целевые показатели масштабируемости и производительности для хранилища BLOB-объектов).

[Именование разделов](../blobs/storage-performance-checklist.md#partitioning) является еще одним важным фактором при разработке приложения с высокой производительностью, использующего большие двоичные объекты. Для размера блока более 4 МиБ используются [блочные BLOB-объекты с высокой пропускной способностью](https://azure.microsoft.com/blog/high-throughput-with-azure-blob-storage/). В этом случае именование раздела не повлияет на производительность. Для блоков размером менее 4 МиБ служба хранилища Azure использует схему секционирования по диапазону для масштабирования и распределения нагрузки. Эта конфигурация означает, что файлы с одинаковыми соглашениями об именовании или префиксами располагаются в одном разделе. Эта логика включает имя контейнера, в который передаются файлы. В этом руководстве используются файлы, содержащие глобальные уникальные идентификаторы (GUID) имен, а также содержимое, создаваемое случайным образом. Затем они передаются в пять разных контейнеров со случайными именами.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством необходимо изучить предыдущее руководство по использованию хранилища: [Создание виртуальной машины и учетной записи хранения для масштабируемого приложения][previous-tutorial].

## <a name="remote-into-your-virtual-machine"></a>Удаленное подключение к виртуальной машине

Для создания сеанса удаленного рабочего стола с виртуальной машиной выполните команду ниже на своем локальном компьютере. Замените IP-адрес значением publicIPAddress виртуальной машины. При появлении запроса введите учетные данные, использованные при создании виртуальной машины.

```console
mstsc /v:<publicIpAddress>
```

## <a name="configure-the-connection-string"></a>Настройка строки подключения

Войдите в свою учетную запись хранения на портале Azure. В меню учетной записи хранения в разделе **Параметры** выберите **Ключи доступа**. Скопируйте **строку подключения** из основного или вторичного ключа. Войдите на виртуальную машину, созданную в рамках работы с предыдущим руководством. Откройте **командную строку** от имени администратора и выполните команду `setx` с параметром `/m`. Таким образом вы сохраните переменную среды параметра виртуальной машины. Переменная среды доступна только после перезагрузки **командной строки**. Замените **\<storageConnectionString\>** в следующем примере:

```console
setx storageconnectionstring "<storageConnectionString>" /m
```

По завершении откройте другую **командную строку**, перейдите в расположение `D:\git\storage-dotnet-perf-scale-app` и введите `dotnet build`, чтобы создать приложение.

## <a name="run-the-application"></a>Выполнение приложения

Перейдите на страницу `D:\git\storage-dotnet-perf-scale-app`.

Нажмите клавишу `dotnet run`, чтобы запустить приложение. При первом выполнении команды `dotnet` заполнится локальный кэш пакета для оптимизации скорости восстановления и включения автономного доступа. Выполнение этой команды займет около минуты, и она используется только один раз.

```console
dotnet run
```

Приложение создаст пять контейнеров со случайными именами и начнет отправлять файлы из промежуточного каталога в учетную запись хранения.

Метод `UploadFilesAsync` показан в следующем примере:

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Scalable.cs" id="Snippet_UploadFilesAsync":::

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

Для минимального и максимального числа потоков задано значение 100, чтобы обеспечить допустимость большого количества одновременных подключений.

```csharp
private static async Task UploadFilesAsync()
{
    // Create five randomly named containers to store the uploaded files.
    CloudBlobContainer[] containers = await GetRandomContainersAsync();

    var currentdir = System.IO.Directory.GetCurrentDirectory();

    // Path to the directory to upload
    string uploadPath = currentdir + "\\upload";

    // Start a timer to measure how long it takes to upload all the files.
    Stopwatch time = Stopwatch.StartNew();

    try
    {
        Console.WriteLine("Iterating in directory: {0}", uploadPath);

        int count = 0;
        int max_outstanding = 100;
        int completed_count = 0;

        // Define the BlobRequestOptions on the upload.
        // This includes defining an exponential retry policy to ensure that failed connections
        // are retried with a back off policy. As multiple large files are being uploaded using
        // large block sizes, this can cause an issue if an exponential retry policy is not defined.
        // Additionally, parallel operations are enabled with a thread count of 8.
        // This should be a multiple of the number of processor cores in the machine.
        // Lastly, MD5 hash validation is disabled for this example, improving the upload speed.
        BlobRequestOptions options = new BlobRequestOptions
        {
            ParallelOperationThreadCount = 8,
            DisableContentMD5Validation = true,
            StoreBlobContentMD5 = false
        };

        // Create a new instance of the SemaphoreSlim class to 
        // define the number of threads to use in the application.
        SemaphoreSlim sem = new SemaphoreSlim(max_outstanding, max_outstanding);

        List<Task> tasks = new List<Task>();
        Console.WriteLine("Found {0} file(s)", Directory.GetFiles(uploadPath).Count());

        // Iterate through the files
        foreach (string path in Directory.GetFiles(uploadPath))
        {
            var container = containers[count % 5];
            string fileName = Path.GetFileName(path);
            Console.WriteLine("Uploading {0} to container {1}", path, container.Name);
            CloudBlockBlob blockBlob = container.GetBlockBlobReference(fileName);

            // Set the block size to 100MB.
            blockBlob.StreamWriteSizeInBytes = 100 * 1024 * 1024;

            await sem.WaitAsync();

            // Create a task for each file to upload. The tasks are
            // added to a collection and all run asynchronously.
            tasks.Add(blockBlob.UploadFromFileAsync(path, null, options, null).ContinueWith((t) =>
            {
                sem.Release();
                Interlocked.Increment(ref completed_count);
            }));

            count++;
        }

        // Run all the tasks asynchronously.
        await Task.WhenAll(tasks);

        time.Stop();

        Console.WriteLine("Upload has been completed in {0} seconds. Press any key to continue", time.Elapsed.TotalSeconds.ToString());

        Console.ReadLine();
    }
    catch (DirectoryNotFoundException ex)
    {
        Console.WriteLine("Error parsing files in the directory: {0}", ex.Message);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```
Помимо параметров ограничения для количества подключений и потоков настраиваются также параметры [BlobRequestOptions](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions) метода [UploadFromStreamAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob.uploadfromstreamasync) для использования параллелизма и отключения проверки хэша MD5. Файлы передаются в блоки размером 100 МБ. Такая конфигурация обеспечивает лучшую производительность, но может быть дорогостоящей при использовании медленной сети. Это объясняется тем, что в случае сбоя будет выполнена повторная передача всего блока.

|Свойство|Значение|Описание|
|---|---|---|
|[ParallelOperationThreadCount](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.paralleloperationthreadcount)| 8| Этот параметр разделяет передаваемый большой двоичный объект на блоки. Для обеспечения максимальной производительности это значение должно в 8 раз превышать количество ядер. |
|[DisableContentMD5Validation](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.disablecontentmd5validation)| Да| Это свойство отключает проверку хэша MD5 отправляемого содержимого. При этом передача ускоряется. Но без проверки MD5 не будет подтверждения о достоверности или целостности передаваемых файлов.   |
|[StoreBlobContentMD5](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.storeblobcontentmd5)| false| Это свойство определяет, будет ли вычисляться и сохраняться в файле хэш MD5.   |
| [RetryPolicy](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.retrypolicy)| Интервал задержки — 2 секунды. Максимальное количество повторений — 10. |Определяет политику повтора запросов. При сбоях подключения выполняются повторные попытки подключения. В этом примере для политики [ExponentialRetry](/dotnet/api/microsoft.azure.batch.common.exponentialretry) настроен интервал задержки 2 секунды с максимальным количеством повторений — 10. Этот параметр важен, когда есть вероятность превышения приложением ограничений целевых показателей масштабируемости хранилища BLOB-объектов. Дополнительные сведения см. в статье [Scalability and performance targets for Blob storage](../blobs/scalability-targets.md) (Целевые показатели масштабируемости и производительности для хранилища BLOB-объектов).  |

---

В следующем примере представлены усеченные выходные данные приложения, работающего под управлением Windows.

```console
Created container 2dbb45f4-099e-49eb-880c-5b02ebac135e
Created container 0d784365-3bdf-4ef2-b2b2-c17b6480792b
Created container 42ac67f2-a316-49c9-8fdb-860fb32845d7
Created container f0357772-cb04-45c3-b6ad-ff9b7a5ee467
Created container 92480da9-f695-4a42-abe8-fb35e71eb887
Iterating in directory: C:\git\myapp\upload
Found 5 file(s)
Uploading 1d596d16-f6de-4c4c-8058-50ebd8141e4d.pdf to container 2dbb45f4-099e-49eb-880c-5b02ebac135e
Uploading 242ff392-78be-41fb-b9d4-aee8152a6279.pdf to container 0d784365-3bdf-4ef2-b2b2-c17b6480792b
Uploading 38d4d7e2-acb4-4efc-ba39-f9611d0d55ef.pdf to container 42ac67f2-a316-49c9-8fdb-860fb32845d7
Uploading 45930d63-b0d0-425f-a766-cda27ff00d32.pdf to container f0357772-cb04-45c3-b6ad-ff9b7a5ee467
Uploading 5129b385-5781-43be-8bac-e2fbb7d2bd82.pdf to container 92480da9-f695-4a42-abe8-fb35e71eb887
Uploaded 5 files in 16.9552163 seconds
```

### <a name="validate-the-connections"></a>Проверка подключений

Во время передачи файлов можно проверить количество одновременных подключений к учетной записи хранения. Откройте окно консоли и введите `netstat -a | find /c "blob:https"`. Эта команда показывает количество подключений, которые открыты в настоящее время. Как видно из следующего примера, при передаче случайных файлов в учетную запись хранения было открыто 800 подключений. Это значение меняется на протяжении выполнения передачи. При отправке в параллельных фрагментах блока значительно сокращается количество времени, необходимое для передачи содержимого.

```
C:\>netstat -a | find /c "blob:https"
800

C:\>
```

## <a name="next-steps"></a>Дальнейшие действия

Из второй части в серии вы узнали не только о передаче больших объемов случайных данных в учетную запись хранения в параллельном режиме, но и о том, как выполнять такие задачи:

> [!div class="checklist"]
> * Настройка строки подключения
> * создание приложения;
> * Выполнение приложения
> * Проверка количества подключений.

Ознакомьтесь с третьей частью в серии, чтобы узнать, как загружать большие объемы данных из учетной записи хранения.

> [!div class="nextstepaction"]
> [Скачивание больших объемов случайных данных из службы хранилища Azure](storage-blob-scalable-app-download-files.md)

[previous-tutorial]: storage-blob-scalable-app-create-vm.md
