---
title: Получение списка больших двоичных объектов с помощью .NET — служба хранилища Azure
description: Узнайте, как получить список больших двоичных объектов в контейнере в учетной записи хранения Azure с помощью клиентской библиотеки .NET. В примерах кода показано, как получить неструктурированный список больших двоичных объектов или как создать структурированный список больших двоичных объектов, как если бы они были упорядочены по каталогам или папкам.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/16/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: ddd19c90c8c47016497e2c3b00e04595a94e7715
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95543074"
---
# <a name="list-blobs-with-net"></a>Получение списка больших двоичных объектов с помощью .NET

При перечислении больших двоичных объектов из кода можно указать ряд параметров для управления способом возврата результатов из службы хранилища Azure. Можно указать число возвращаемых результатов в каждом наборе результатов, а затем извлечь последующие наборы. Можно указать префикс для возврата больших двоичных объектов, имена которых начинаются с указанного символа или строки. Кроме того, можно создать неструктурированный или структурированный список больших двоичных объектов. В структурированном списке большие двоичные объекты представлены так, будто они организованы по папкам.

В этой статье показано, как получить список больших двоичных объектов с помощью [клиентской библиотеки службы хранилища Azure для .NET](/dotnet/api/overview/azure/storage).  

## <a name="understand-blob-listing-options"></a>Общие сведения о параметрах получения списка больших двоичных объектов

Чтобы получить список больших двоичных объектов в учетной записи хранения, вызовите один из следующих методов:

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

- [Блобконтаинерклиент. BLOB](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobs)
- [Блобконтаинерклиент. Жетблобсасинк](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsasync)
- [Блобконтаинерклиент. Жетблобсбихиерарчи](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsbyhierarchy)
- [Блобконтаинерклиент. Жетблобсбихиерарчясинк](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsbyhierarchyasync)

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

- [CloudBlobClient.ListBlobs](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listblobs)
- [CloudBlobClient.ListBlobsSegmented](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listblobssegmented)
- [CloudBlobClient.ListBlobsSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient.listblobssegmentedasync)

Чтобы получить список больших двоичных объектов в контейнере, вызовите один из следующих методов:

- [CloudBlobContainer.ListBlobs](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobs)
- [CloudBlobContainer.ListBlobsSegmented](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobssegmented)
- [CloudBlobContainer.ListBlobsSegmentedAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobssegmentedasync)

Перегрузки для этих методов предоставляют дополнительные параметры для управления возвратом больших двоичных объектов операцией перечисления. Описание этих параметров приводится в следующих разделах.

---

### <a name="manage-how-many-results-are-returned"></a>Управление количеством возвращаемых результатов

По умолчанию операция перечисления возвращает до 5000 результатов за раз, но можно указать количество результатов, которое будет возвращено каждой операции перечисления. В примерах, приведенных в этой статье, показано, как вернуть результаты на страницы.

### <a name="filter-results-with-a-prefix"></a>Фильтрация результатов с помощью префикса

Чтобы отфильтровать список больших двоичных объектов, укажите строку для `prefix` параметра. Строка префикса может содержать один или несколько символов. Служба хранилища Azure возвращает только те большие двоичные объекты, имена которых начинаются с этого префикса.

### <a name="return-metadata"></a>Возврат метаданных

Метаданные большого двоичного объекта можно вернуть с результатами.

- Если вы используете пакет SDK для .NET версии 12, укажите значение **метаданных** для перечисления [блобтраитс](/dotnet/api/azure.storage.blobs.models.blobtraits) .

- Если вы используете пакет SDK для .NET версии 11, укажите значение **метаданных** для перечисления [блоблистингдетаилс](/dotnet/api/microsoft.azure.storage.blob.bloblistingdetails) . Служба хранилища Azure включает метаданные для каждого возвращенного большого двоичного объекта, поэтому вам не нужно вызывать один из методов **FetchAttributes** в этом контексте, чтобы получить метаданные большого двоичного объекта.

### <a name="list-blob-versions-or-snapshots"></a>Вывод списка версий BLOB-объектов или моментальных снимков

Чтобы получить список версий BLOB-объектов или моментальных снимков с клиентской библиотекой .NET 12, укажите параметр [блобстатес](/dotnet/api/azure.storage.blobs.models.blobstates) в поле **версия** или **моментальный снимок** . Версии и моментальные снимки перечислены от старых к новым. Дополнительные сведения о листинге версий см. в разделе [список версий BLOB-объектов](versioning-enable.md#list-blob-versions).

### <a name="flat-listing-versus-hierarchical-listing"></a>Неструктурированный список и структурированный список

Большие двоичные объекты находятся в службе хранилища Azure в неструктурированном виде, без использования какой-либо иерархии (например, как в классической файловой системе). Однако большие двоичные объекты можно упорядочить по *виртуальным каталогам*, чтобы имитировать структуру папок. Виртуальный каталог образует часть имени большого двоичного объекта и обозначается символом-разделителем.

Чтобы упорядочить большие двоичные объекты по виртуальным каталогам, используйте символ-разделитель в имени большого двоичного объекта. Символом-разделителем по умолчанию является косая черта (/), однако в качестве разделителя можно указать любой символ.

Если присвоить большим двоичным объектам имена с использованием разделителя, можно выбрать получение структурированного списка больших двоичных объектов. Для этого служба хранилища Azure возвращает все виртуальные каталоги и большие двоичные объекты под родительским объектом. Операцию перечисления можно вызвать рекурсивно для прохода по иерархии, подобно тому, как осуществляется программный обход классической файловой системы.

## <a name="use-a-flat-listing"></a>Использование неструктурированного списка

По умолчанию операция перечисления возвращает большие двоичные объекты в виде неструктурированного списка. В неструктурированном списке большие двоичные объекты не упорядочиваются по виртуальным каталогам.

В следующем примере выводится неструктурированный список больших двоичных объектов в указанном контейнере с указанным необязательным размером сегмента и записывается имя большого двоичного объекта в окно консоли.

Если вы включили функцию иерархического пространства имен в учетной записи, каталоги не являются виртуальными. Вместо этого они являются конкретными, независимыми объектами. Поэтому каталоги отображаются в списке как большие двоичные объекты нулевой длины.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ListBlobsFlatListing":::

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

Если операция перечисления возвращает более 5000 больших двоичных объектов или число доступных больших двоичных объектов превышает указанное число, служба хранилища Azure возвращает *токен продолжения* со списком больших двоичных объектов. Маркер продолжения — это непрозрачное значение, которое можно использовать для получения следующего набора результатов из службы хранилища Azure.

В коде проверьте значение маркера продолжения, чтобы определить, имеет ли он значение NULL. Если маркер продолжения имеет значение NULL, набор результатов будет полным. Если токен продолжения не равен NULL, вызовите операцию перечисления, передав маркер продолжения для получения следующего набора результатов, пока маркер продолжения не будет иметь значение NULL.

```csharp
private static async Task ListBlobsFlatListingAsync(CloudBlobContainer container, int? segmentSize)
{
    BlobContinuationToken continuationToken = null;
    CloudBlob blob;

    try
    {
        // Call the listing operation and enumerate the result segment.
        // When the continuation token is null, the last segment has been returned
        // and execution can exit the loop.
        do
        {
            BlobResultSegment resultSegment = await container.ListBlobsSegmentedAsync(string.Empty,
                true, BlobListingDetails.Metadata, segmentSize, continuationToken, null, null);

            foreach (var blobItem in resultSegment.Results)
            {
                blob = (CloudBlob)blobItem;

                // Write out some blob properties.
                Console.WriteLine("Blob name: {0}", blob.Name);
            }

            Console.WriteLine();

           // Get the continuation token and loop until it is null.
           continuationToken = resultSegment.ContinuationToken;

        } while (continuationToken != null);
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

Выходные данные аналогичны следующим:

```
Blob name: FolderA/blob1.txt
Blob name: FolderA/blob2.txt
Blob name: FolderA/blob3.txt
Blob name: FolderA/FolderB/blob1.txt
Blob name: FolderA/FolderB/blob2.txt
Blob name: FolderA/FolderB/blob3.txt
Blob name: FolderA/FolderB/FolderC/blob1.txt
Blob name: FolderA/FolderB/FolderC/blob2.txt
Blob name: FolderA/FolderB/FolderC/blob3.txt
```

## <a name="use-a-hierarchical-listing"></a>Использование иерархического списка

При вызове операции иерархического перечисления служба хранилища Azure возвращает виртуальные каталоги и большие двоичные объекты на первом уровне иерархии. Для каждого виртуального каталога задается свойство [Prefix](/dotnet/api/microsoft.azure.storage.blob.cloudblobdirectory.prefix), чтобы можно было передать префикс в рекурсивном вызове для получения следующего каталога.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Чтобы составить список больших двоичных объектов в иерархическом порядке, вызовите метод [блобконтаинерклиент. жетблобсбихиерарчи](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsbyhierarchy)или [блобконтаинерклиент. жетблобсбихиерарчясинк](/dotnet/api/azure.storage.blobs.blobcontainerclient.getblobsbyhierarchyasync) .

В следующем примере перечисляются большие двоичные объекты в указанном контейнере с помощью иерархического перечисления с указанным необязательным размером сегмента и записывает имя большого двоичного объекта в окно консоли.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ListBlobsHierarchicalListing":::

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

Чтобы получить иерархический список больших двоичных объектов, задайте для параметра `useFlatBlobListing` метода перечисления значение **false**.

В следующем примере выводится иерархический список больших двоичных объектов в указанном контейнере с указанным необязательным размером сегмента и записывается имя большого двоичного объекта в окно консоли.

```csharp
private static async Task ListBlobsHierarchicalListingAsync(CloudBlobContainer container, string prefix)
{
    CloudBlobDirectory dir;
    CloudBlob blob;
    BlobContinuationToken continuationToken = null;

    try
    {
        // Call the listing operation and enumerate the result segment.
        // When the continuation token is null, the last segment has been returned and
        // execution can exit the loop.
        do
        {
            BlobResultSegment resultSegment = await container.ListBlobsSegmentedAsync(prefix,
                false, BlobListingDetails.Metadata, null, continuationToken, null, null);
            foreach (var blobItem in resultSegment.Results)
            {
                // A hierarchical listing may return both virtual directories and blobs.
                if (blobItem is CloudBlobDirectory)
                {
                    dir = (CloudBlobDirectory)blobItem;

                    // Write out the prefix of the virtual directory.
                    Console.WriteLine("Virtual directory prefix: {0}", dir.Prefix);

                    // Call recursively with the prefix to traverse the virtual directory.
                    await ListBlobsHierarchicalListingAsync(container, dir.Prefix);
                }
                else
                {
                    // Write out the name of the blob.
                    blob = (CloudBlob)blobItem;

                    Console.WriteLine("Blob name: {0}", blob.Name);
                }
                Console.WriteLine();
            }

            // Get the continuation token and loop until it is null.
            continuationToken = resultSegment.ContinuationToken;

        } while (continuationToken != null);
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

Выходные данные аналогичны следующим:

```
Virtual directory prefix: FolderA/
Blob name: FolderA/blob1.txt
Blob name: FolderA/blob2.txt
Blob name: FolderA/blob3.txt

Virtual directory prefix: FolderA/FolderB/
Blob name: FolderA/FolderB/blob1.txt
Blob name: FolderA/FolderB/blob2.txt
Blob name: FolderA/FolderB/blob3.txt

Virtual directory prefix: FolderA/FolderB/FolderC/
Blob name: FolderA/FolderB/FolderC/blob1.txt
Blob name: FolderA/FolderB/FolderC/blob2.txt
Blob name: FolderA/FolderB/FolderC/blob3.txt
```

> [!NOTE]
> При выполнении операции иерархического перечисления не выводятся моментальные снимки больших двоичных объектов.

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

- [Перечисление больших двоичных объектов](/rest/api/storageservices/list-blobs)
- [Перечисление ресурсов больших двоичных объектов](/rest/api/storageservices/enumerating-blob-resources)