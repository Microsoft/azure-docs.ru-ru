---
title: Управление свойствами и метаданными для большого двоичного объекта с помощью .NET — служба хранилища Azure
description: Узнайте, как задавать и получать системные свойства, а также хранить пользовательские метаданные в больших двоичных объектах в учетной записи хранения Azure с помощью клиентской библиотеки .NET.
services: storage
author: mhopkins-msft
ms.author: mhopkins
ms.date: 09/25/2020
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.custom: devx-track-csharp
ms.openlocfilehash: 984fb00e163a090534da1fb41850dcfef6c5d516
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95521535"
---
# <a name="manage-blob-properties-and-metadata-with-net"></a>Управление свойствами и метаданными больших двоичных объектов с помощью .NET

В дополнение к содержащимся в них данным, большие двоичные объекты поддерживают системные свойства и определяемые пользователем метаданные. В этой статье показано, как управлять свойствами системы и определяемыми пользователем метаданными с помощью [клиентской библиотеки службы хранилища Azure для .NET](/dotnet/api/overview/azure/storage).

## <a name="about-properties-and-metadata"></a>Сведения о свойствах и метаданных

- **Свойства системы**: свойства системы существуют в каждом ресурсе хранилища BLOB-объектов. Некоторые из них можно считать или задать, некоторые — только считать. На самом деле, некоторые свойства системы соответствуют определенным стандартным заголовкам HTTP. Клиентская библиотека службы хранилища Azure для .NET сохраняет эти свойства.

- **Определяемые пользователем метаданные**. определяемые пользователем метаданные состоят из одной или нескольких пар "имя-значение", указанных для ресурса хранилища BLOB-объектов. Метаданные можно использовать для хранения дополнительных значений в ресурсе. Значения метаданных предназначены только для собственных целей и не влияют на работу ресурса.

> [!NOTE]
> Теги индекса больших двоичных объектов также предоставляют возможность хранения произвольных пользовательских атрибутов ключа и значения наряду с ресурсом хранилища BLOB-объектов Azure. Аналогично метаданным, только теги индекса больших двоичных объектов автоматически индексируются и делаются доступными для поиска в собственной службе BLOB-объектов. Метаданные не могут индексироваться и запрашиваться, если не используется отдельная служба, например поиск Azure.
>
> Дополнительные сведения об этой функции см. в статье [Управление и поиск данных в хранилище BLOB-объектов Azure с помощью индекса больших двоичных объектов (Предварительная версия)](storage-manage-find-blobs.md).

## <a name="set-and-retrieve-properties"></a>Задание и получение свойств

В следующем примере кода задаются `ContentType` `ContentLanguage` системные свойства и для большого двоичного объекта.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Чтобы задать свойства для большого двоичного объекта, вызовите [сесттфеадерс](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.sethttpheaders) или [сесттфеадерсасинк](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.sethttpheadersasync). Все свойства, не заданные явно, очищаются. В следующем примере кода сначала возвращаются существующие свойства большого двоичного объекта, а затем они используются для заполнения заголовков, которые не обновляются.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Metadata.cs" id="Snippet_SetBlobProperties":::

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

```csharp
public static async Task SetBlobPropertiesAsync(CloudBlob blob)
{
    try
    {
        Console.WriteLine("Setting blob properties.");

        // You must explicitly set the MIME ContentType every time
        // the properties are updated or the field will be cleared.
        blob.Properties.ContentType = "text/plain";
        blob.Properties.ContentLanguage = "en-us";

        // Set the blob's properties.
        await blob.SetPropertiesAsync();
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```
---

Следующий пример кода получает системные свойства большого двоичного объекта и отображает некоторые из значений.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Metadata.cs" id="Snippet_ReadBlobProperties":::

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

Получение метаданных и значений свойств для ресурса хранилища BLOB-объектов — это двухэтапный процесс. Прежде чем можно будет прочитать эти значения, необходимо явно получить их, вызвав `FetchAttributes` `FetchAttributesAsync` метод или. Исключением из этого правила является то, `Exists` что `ExistsAsync` методы и вызывают соответствующий `FetchAttributes` метод в рамках. При вызове одного из этих методов также не требуется вызывать `FetchAttributes` .

> [!IMPORTANT]
> Если вы обнаружите, что значения свойств или метаданных для ресурса хранилища не заполнены, убедитесь, что код вызывает `FetchAttributes` `FetchAttributesAsync` метод или.

Чтобы получить свойства большого двоичного объекта, вызовите `FetchAttributes` метод или для `FetchAttributesAsync` большого двоичного объекта, чтобы заполнить `Properties` свойство.

```csharp
private static async Task GetBlobPropertiesAsync(CloudBlob blob)
{
    try
    {
        // Fetch the blob properties.
        await blob.FetchAttributesAsync();

        // Display some of the blob's property values.
        Console.WriteLine(" ContentLanguage: {0}", blob.Properties.ContentLanguage);
        Console.WriteLine(" ContentType: {0}", blob.Properties.ContentType);
        Console.WriteLine(" Created: {0}", blob.Properties.Created);
        Console.WriteLine(" LastModified: {0}", blob.Properties.LastModified);
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```
---

## <a name="set-and-retrieve-metadata"></a>Установка и получение метаданных

Метаданные можно указать как одну или несколько пар "имя-значение" для BLOB-ресурса или ресурса контейнера. Чтобы задать метаданные, добавьте пары "имя-значение" в `Metadata` коллекцию ресурса. Затем вызовите один из следующих методов для записи значений:

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

- [сетметадата](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.setmetadata)
- [сетметадатаасинк](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.setmetadataasync)

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

- [сетметадата](/dotnet/api/microsoft.azure.storage.blob.cloudblob.setmetadata)
- [сетметадатаасинк](/dotnet/api/microsoft.azure.storage.blob.cloudblob.setmetadataasync)
---

Пары "имя-значение" метаданных являются допустимыми заголовками HTTP и должны соответствовать всем ограничениям, регулирующим заголовки HTTP. Имена метаданных должны представлять собой допустимые имена HTTP-заголовков и допустимые идентификаторы C#, могут содержать только символы ASCII и должны обрабатываться без учета регистра. Кодировка [Base64](/dotnet/api/system.convert.tobase64string) или [URL-кодирование](/dotnet/api/system.web.httputility.urlencode) значений метаданных, содержащих символы, не входящие в набор ASCII.

Имя метаданных должно соответствовать соглашениям об именовании идентификаторов C#. Имена метаданных поддерживают регистр, используемый при создании, но не учитывают регистр при установке или чтении. Если для ресурса отправляется несколько заголовков метаданных с одним и тем же именем, хранилище BLOB-объектов Azure возвращает код ошибки HTTP 400 (недопустимый запрос).

В следующем примере кода задаются метаданные для большого двоичного объекта. Одно значение задается с помощью `Add` метода коллекции. Другое значение задается с помощью неявного синтаксиса «ключ/значение».

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Metadata.cs" id="Snippet_AddBlobMetadata":::

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

```csharp
public static async Task AddBlobMetadataAsync(CloudBlob blob)
{
    try
    {
        // Add metadata to the blob by calling the Add method.
        blob.Metadata.Add("docType", "textDocuments");

        // Add metadata to the blob by using key/value syntax.
        blob.Metadata["category"] = "guidance";

        // Set the blob's metadata.
        await blob.SetMetadataAsync();
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```
---

В следующем примере кода считываются метаданные в большом двоичном объекте.

# <a name="net-v12"></a>[.NET (версии 12)](#tab/dotnet)

Чтобы получить метаданные, вызовите метод [жетпропертиесасинк](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.getpropertiesasync) или для большого двоичного [объекта или контейнера](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.getproperties) , чтобы заполнить коллекцию [метаданных](/dotnet/api/azure.storage.blobs.models.blobproperties.metadata) , а затем считайте значения, как показано в примере ниже. Методы- **Свойства** извлекают свойства и метаданные большого двоичного объекта в одном вызове. Это отличается от интерфейсов API, для которых требуются отдельные вызовы для [получения свойств большого двоичного объекта](/rest/api/storageservices/get-blob-properties) и [получения метаданных большого двоичного объекта](/rest/api/storageservices/get-blob-metadata).

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Metadata.cs" id="Snippet_ReadBlobMetadata":::

# <a name="net-v11"></a>[.NET (версии 11)](#tab/dotnet11)

Чтобы получить метаданные, вызовите `FetchAttributes` метод или для `FetchAttributesAsync` большого двоичного объекта или контейнера, чтобы заполнить `Metadata` коллекцию, а затем считайте значения, как показано в примере ниже.

```csharp
public static async Task ReadBlobMetadataAsync(CloudBlob blob)
{
    try
    {
        // Fetch blob attributes in order to populate 
        // the blob's properties and metadata.
        await blob.FetchAttributesAsync();

        Console.WriteLine("Blob metadata:");

        // Enumerate the blob's metadata.
        foreach (var metadataItem in blob.Metadata)
        {
            Console.WriteLine("\tKey: {0}", metadataItem.Key);
            Console.WriteLine("\tValue: {0}", metadataItem.Value);
        }
    }
    catch (StorageException e)
    {
        Console.WriteLine("HTTP error code {0}: {1}",
                            e.RequestInformation.HttpStatusCode,
                            e.RequestInformation.ErrorCode);
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```
---

[!INCLUDE [storage-blob-dotnet-resources-include](../../../includes/storage-blob-dotnet-resources-include.md)]

## <a name="see-also"></a>См. также раздел

- [Операция задания свойств BLOB-объекта](/rest/api/storageservices/set-blob-properties)
- [Операция получения свойств BLOB-объекта](/rest/api/storageservices/get-blob-properties)
- [Операция задания метаданных BLOB-объекта](/rest/api/storageservices/set-blob-metadata)
- [Операция получения метаданных BLOB-объекта](/rest/api/storageservices/get-blob-metadata)