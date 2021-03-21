---
title: Копирование большого двоичного объекта с помощью API службы хранилища Azure
description: Узнайте, как скопировать большой двоичный объект с помощью клиентских библиотек службы хранилища Azure.
services: storage
author: mhopkins-msft
ms.author: mhopkins
ms.date: 01/08/2021
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 28b23555f97e58dde038c5e3e12a4b741b81776d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98059047"
---
# <a name="copy-a-blob-with-azure-storage-client-libraries"></a>Копирование большого двоичного объекта с помощью клиентских библиотек службы хранилища Azure

В этой статье показано, как скопировать большой двоичный объект в учетную запись хранения Azure. В нем также показано, как прервать асинхронную операцию копирования. В примере кода используются клиентские библиотеки службы хранилища Azure.

## <a name="about-copying-blobs"></a>Сведения о копировании больших двоичных объектов

При копировании большого двоичного объекта в той же учетной записи хранения это синхронная операция. При копировании между учетными записями это асинхронная операция.

Источником для операции копирования может быть блочный BLOB-объект, добавочный большой двоичный объект, страничный BLOB-объект или моментальный снимок. Если BLOB-объект назначения уже существует, он должен иметь тот же тип BLOB-объекта, что и BLOB-объект источника. Существующий целевой BLOB-объект будет перезаписан.

Целевой BLOB-объект нельзя изменить во время выполнения операции копирования. Целевой BLOB-объект может иметь только одну незавершенную операцию копирования. Иными словами, большой двоичный объект не может быть местом назначения для нескольких ожидающих операций копирования.

Всегда копируется весь исходный BLOB-объект или файл. Копирование диапазона байтов или набора блоков не поддерживается.

При копировании большого двоичного объекта его системные свойства копируются в целевой большой двоичный объект с теми же значениями.

Операция копирования может принимать любое из следующих форм:

- Скопируйте исходный BLOB-объект в целевой большой двоичный объект с другим именем. Целевой BLOB-объект может быть существующим большим двоичным объектом того же типа большого двоичного объекта (блок, добавление или страница) или новым большим двоичным объектом, созданным операцией копирования.
- Скопируйте исходный BLOB-объект в целевой большой двоичный объект с тем же именем, фактически заменив целевой BLOB-объект. Такая операция копирования удаляет все незафиксированные блоки и перезаписывает метаданные целевого BLOB-объекта.
- Скопируйте исходный файл в службу файлов Azure в целевой BLOB-объект. Целевым объектом BLOB может быть существующий блочный BLOB-объект или новый блочный большой двоичный объект, созданный операцией копирования. Копирование из файлов в страничные BLOB-объекты или добавочные большие двоичные объекты не поддерживается.
- Копирование моментального снимка из базового большого двоичного объекта. Повысив уровень моментального снимка до базового большого двоичного объекта, можно восстановить более раннюю версию большого двоичного объекта.
- Скопируйте моментальный снимок в целевой BLOB-объект с другим именем. Результирующий целевой большой двоичный объект не станет моментальным снимком, а будет большим двоичным объектом, доступным для записи.

## <a name="copy-a-blob"></a>Копирование большого двоичного объекта

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Чтобы скопировать большой двоичный объект, вызовите один из следующих методов:

- [старткопифромури](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.startcopyfromuri)
- [старткопифромуриасинк](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.startcopyfromuriasync)

`StartCopyFromUri`Методы и `StartCopyFromUriAsync` возвращают объект [копифромуриоператион](/dotnet/api/azure.storage.blobs.models.copyfromurioperation) , содержащий сведения об операции копирования.

В следующем примере кода возвращается [блобклиент](/dotnet/api/azure.storage.blobs.blobclient) , представляющий ранее созданный большой двоичный объект, и он копируется в новый большой двоичный объект в том же контейнере:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CopyBlob.cs" id="Snippet_CopyBlob":::

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

Чтобы скопировать большой двоичный объект, вызовите один из следующих методов:

- [старткопи](/dotnet/api/microsoft.azure.storage.blob.cloudblob.startcopy)
- [старткопясинк](/dotnet/api/microsoft.azure.storage.blob.cloudblob.startcopyasync)

`StartCopy`Методы и `StartCopyAsync` возвращают значение идентификатора копии, которое используется для проверки состояния или отмены операции копирования.

В следующем примере кода возвращается ссылка на ранее созданный большой двоичный объект и копируется в новый большой двоичный объект в том же контейнере:

```csharp
private static async Task CopyBlockBlobAsync(CloudBlobContainer container)
{
    CloudBlockBlob sourceBlob = null;
    CloudBlockBlob destBlob = null;
    string leaseId = null;

    try
    {
        // Get a block blob from the container to use as the source.
        sourceBlob = container.ListBlobs().OfType<CloudBlockBlob>().FirstOrDefault();

        // Lease the source blob for the copy operation 
        // to prevent another client from modifying it.
        // Specifying null for the lease interval creates an infinite lease.
        leaseId = await sourceBlob.AcquireLeaseAsync(null);

        // Get a reference to a destination blob (in this case, a new blob).
        destBlob = container.GetBlockBlobReference("copy of " + sourceBlob.Name);

        // Ensure that the source blob exists.
        if (await sourceBlob.ExistsAsync())
        {
            // Get the ID of the copy operation.
            string copyId = await destBlob.StartCopyAsync(sourceBlob);

            // Fetch the destination blob's properties before checking the copy state.
            await destBlob.FetchAttributesAsync();

            Console.WriteLine("Status of copy operation: {0}", destBlob.CopyState.Status);
            Console.WriteLine("Completion time: {0}", destBlob.CopyState.CompletionTime);
            Console.WriteLine("Bytes copied: {0}", destBlob.CopyState.BytesCopied.ToString());
            Console.WriteLine("Total bytes: {0}", destBlob.CopyState.TotalBytes.ToString());
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
    finally
    {
        // Break the lease on the source blob.
        if (sourceBlob != null)
        {
            await sourceBlob.FetchAttributesAsync();

            if (sourceBlob.Properties.LeaseState != LeaseState.Available)
            {
                await sourceBlob.BreakLeaseAsync(new TimeSpan(0));
            }
        }
    }
}
```

# <a name="python-v12"></a>[Python версии 12](#tab/python)

Чтобы скопировать большой двоичный объект, вызовите метод [start_copy_from_url](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient#start-copy-from-url-source-url--metadata-none--incremental-copy-false----kwargs-) . `start_copy_from_url`Метод возвращает словарь, содержащий сведения об операции копирования.

В следующем примере кода возвращается [блобклиент](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient) , представляющий ранее созданный большой двоичный объект, и он копируется в новый большой двоичный объект в том же контейнере:

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/copy_blob.py" id="Snippet_BlobCopy":::

---

## <a name="abort-a-copy-operation"></a>Прервать операцию копирования

Прерывание операции копирования приводит к получению целевого большого двоичного объекта нулевой длины. Однако метаданные для целевого BLOB-объекта будут иметь новые значения, скопированные из исходного большого двоичного объекта или явно заданы во время операции копирования. Для сохранения исходных метаданных перед копированием Создайте моментальный снимок целевого большого двоичного объекта перед вызовом одного из методов копирования.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Проверьте свойство [блобпропертиес. копистатус](/dotnet/api/azure.storage.blobs.models.blobproperties.copystatus) в целевом большом двоичном объекте, чтобы получить состояние операции копирования. После завершения копирования конечный BLOB-объект будет зафиксирован.

При прерывании операции копирования состояние копирования большого двоичного объекта задается равным [копистатус. прервано](/dotnet/api/microsoft.azure.storage.blob.copystatus).

Методы [аборткопифромури](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.abortcopyfromuri) и [аборткопифромуриасинк](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.abortcopyfromuriasync) отменяют текущую операцию копирования.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CopyBlob.cs" id="Snippet_StopBlobCopy":::

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

Проверьте свойство [копистате. status](/dotnet/api/microsoft.azure.storage.blob.copystate.status) в целевом большом двоичном объекте, чтобы получить состояние операции копирования. После завершения копирования конечный BLOB-объект будет зафиксирован.

При прерывании операции копирования состояние копирования большого двоичного объекта задается равным [копистатус. прервано](/dotnet/api/microsoft.azure.storage.blob.copystatus).

Методы [аборткопи](/dotnet/api/microsoft.azure.storage.blob.cloudblob.abortcopy) и [аборткопясинк](/dotnet/api/microsoft.azure.storage.blob.cloudblob.abortcopyasync) отменяют текущую операцию копирования.

```csharp
// Fetch the destination blob's properties before checking the copy state.
await destBlob.FetchAttributesAsync();

// Check the copy status. If it is still pending, abort the copy operation.
if (destBlob.CopyState.Status == CopyStatus.Pending)
{
    await destBlob.AbortCopyAsync(copyId);
    Console.WriteLine("Copy operation {0} has been aborted.", copyId);
}
```

# <a name="python-v12"></a>[Python версии 12](#tab/python)

Проверьте запись Status в словаре [копипропертиес](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.copyproperties) , возвращаемом методом [get_blob_properties](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient#get-blob-properties---kwargs-) , чтобы получить состояние операции копирования. После завершения копирования конечный BLOB-объект будет зафиксирован.

При прерывании операции копирования [состояние](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.copyproperties) имеет значение "прервано".

Метод [abort_copy](/azure/developer/python/sdk/storage/azure-storage-blob/azure.storage.blob.blobclient#abort-copy-copy-id----kwargs-) отменяет текущую операцию копирования.

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/copy_blob.py" id="Snippet_StopBlobCopy":::

---

## <a name="azure-sdks"></a>Пакеты Azure SDK

Получите дополнительные сведения о пакетах SDK для Azure.

 - [Пакет Azure SDK для .NET](https://github.com/azure/azure-sdk-for-net)
 - [Пакет Azure SDK для Java](https://github.com/azure/azure-sdk-for-java)
 - [SDK Azure для Python](https://github.com/azure/azure-sdk-for-python)
 - [Пакет Azure SDK для JavaScript](https://github.com/azure/azure-sdk-for-js)

## <a name="next-steps"></a>Дальнейшие действия

В следующих разделах содержатся сведения о копировании больших двоичных объектов и прерывании текущих операций копирования с помощью интерфейсов API службы Azure.

- [Копирование BLOB-объекта](/rest/api/storageservices/copy-blob)
- [Прерывание копирования большого двоичного объекта](/rest/api/storageservices/abort-copy-blob)
