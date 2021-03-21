---
title: Перечисление контейнеров больших двоичных объектов с помощью .NET в службе хранилища Azure
description: Узнайте, как перечислить контейнеры больших двоичных объектов в учетной записи хранения Azure с помощью клиентской библиотеки .NET.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 10/14/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: a12fc991734fe74e450aa14a477f3a4500ba659c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96937259"
---
# <a name="list-blob-containers-with-net"></a>Вывод списка контейнеров больших двоичных объектов с помощью .NET

При перечислении контейнеров в учетной записи хранения Azure из кода можно указать несколько параметров для управления возвратом результатов из службы хранилища Azure. В этой статье показано, как перечислить контейнеры с помощью [клиентской библиотеки службы хранилища Azure для .NET](/dotnet/api/overview/azure/storage).  

## <a name="understand-container-listing-options"></a>Общие сведения о параметрах перечисления контейнеров

Чтобы вывести список контейнеров в учетной записи хранения, вызовите один из следующих методов.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

- [жетблобконтаинерс](/dotnet/api/azure.storage.blobs.blobserviceclient.getblobcontainers)
- [жетблобконтаинерсасинк](/dotnet/api/azure.storage.blobs.blobserviceclient.getblobcontainersasync)

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

- [листконтаинерссегментед](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listcontainerssegmented)
- [листконтаинерссегментедасинк](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listcontainerssegmentedasync)

---

Перегрузки для этих методов предоставляют дополнительные параметры для управления возвратом контейнеров с помощью операции перечисления. Описание этих параметров приводится в следующих разделах.

### <a name="manage-how-many-results-are-returned"></a>Управление количеством возвращаемых результатов

По умолчанию операция перечисления возвращает до 5000 результатов за раз. Чтобы вернуть меньший набор результатов, укажите ненулевое значение размера возвращаемой страницы результатов.

Если ваша учетная запись хранения содержит более 5000 контейнеров, или если вы указали размер страницы, чтобы операция перечисления возвращала подмножество контейнеров в учетной записи хранения, служба хранилища Azure возвращает *токен продолжения* со списком контейнеров. Маркер продолжения — это непрозрачное значение, которое можно использовать для получения следующего набора результатов из службы хранилища Azure.

В коде проверьте значение маркера продолжения, чтобы определить, является ли он пустым (для .NET версии 12) или null (для .NET версии 11 и более ранних версий). Если маркер продолжения имеет значение NULL, набор результатов будет полным. Если токен продолжения не равен null, снова вызовите метод List, передав маркер продолжения для получения следующего набора результатов, пока маркер продолжения не будет иметь значение null.

### <a name="filter-results-with-a-prefix"></a>Фильтрация результатов с помощью префикса

Чтобы отфильтровать список контейнеров, укажите строку для параметра `prefix`. Строка префикса может содержать один или несколько символов. Служба хранилища Azure возвращает только те контейнеры, имена которых начинаются с этого префикса.

### <a name="return-metadata"></a>Возврат метаданных

Чтобы вернуть метаданные контейнера с результатами, укажите значение **метаданных** для перечисления [блобконтаинертраитс](/dotnet/api/azure.storage.blobs.models.blobcontainertraits) (для .NET версии 12) или перечисление [контаинерлистингдетаилс](/dotnet/api/microsoft.azure.storage.blob.containerlistingdetails) (для .NET версии 11 и более ранних версий). Служба хранилища Azure включает метаданные с каждым возвращенным контейнером, поэтому не нужно также получать метаданные контейнера.

## <a name="example-list-containers"></a>Пример. список контейнеров

В следующем примере асинхронно перечисляются контейнеры в учетной записи хранения, которые начинаются с указанного префикса. В примере перечисляются контейнеры, начинающиеся с указанного префикса, и возвращается указанное количество результатов для каждого вызова операции перечисления. Затем он использует токен продолжения для получения следующего сегмента результатов. В примере также возвращаются метаданные контейнера с результатами.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Containers.cs" id="Snippet_ListContainers":::

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

```csharp
private static async Task ListContainersWithPrefixAsync(CloudBlobClient blobClient,
                                                        string prefix,
                                                        int? segmentSize)
{
    Console.WriteLine("List containers beginning with prefix {0}, plus container metadata:", prefix);

    BlobContinuationToken continuationToken = null;
    ContainerResultSegment resultSegment;

    try
    {
        do
        {
            // List containers beginning with the specified prefix,
            // returning segments of 5 results each.
            // Passing in null for the maxResults parameter returns the maximum number of results (up to 5000).
            // Requesting the container's metadata as part of the listing operation populates the metadata,
            // so it's not necessary to call FetchAttributes() to read the metadata.
            resultSegment = await blobClient.ListContainersSegmentedAsync(
                prefix, ContainerListingDetails.Metadata, segmentSize, continuationToken, null, null);

            // Enumerate the containers returned.
            foreach (var container in resultSegment.Results)
            {
                Console.WriteLine("\tContainer:" + container.Name);

                // Write the container's metadata keys and values.
                foreach (var metadataItem in container.Metadata)
                {
                    Console.WriteLine("\t\tMetadata key: " + metadataItem.Key);
                    Console.WriteLine("\t\tMetadata value: " + metadataItem.Value);
                }
            }

            // Get the continuation token.
            continuationToken = resultSegment.ContinuationToken;

        } while (continuationToken != null);

        Console.WriteLine();
    }
    catch (StorageException e)
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}
```

---

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="see-also"></a>См. также раздел

- [Список контейнеров](/rest/api/storageservices/list-containers2)
- [Перечисление ресурсов больших двоичных объектов](/rest/api/storageservices/enumerating-blob-resources)
