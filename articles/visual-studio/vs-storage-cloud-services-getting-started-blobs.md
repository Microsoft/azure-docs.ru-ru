---
title: Начало работы с хранилищем BLOB-объектов с помощью Visual Studio (облачные службы)
description: Как приступить к работе, используя хранилище больших двоичных объектов Azure в проекте облачной службы в Visual Studio после подключения к учетной записи хранения с помощью подключенных служб Visual Studio
services: storage
author: ghogen
manager: jillfra
ms.assetid: 1144a958-f75a-4466-bb21-320b7ae8f304
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure, devx-track-csharp
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 12/02/2016
ms.author: ghogen
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 89e0d6873ebfd8f8396c36185730c57a66af0dd9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96007051"
---
# <a name="get-started-with-azure-blob-storage-and-visual-studio-connected-services-cloud-services-projects"></a>Начало работы с хранилищем больших двоичных объектов Azure и подключенными службами Visual Studio (проектами облачных служб)
[!INCLUDE [storage-try-azure-tools-blobs](../../includes/storage-try-azure-tools-blobs.md)]

## <a name="overview"></a>Обзор
В этой статье описывается, как приступить к использованию хранилища больших двоичных объектов Azure после создания учетной записи хранения Azure с помощью диалогового окна **Добавление подключенных служб** в проекте облачных служб Visual Studio. Мы покажем, как получить доступ к контейнерам больших двоичных объектов и как создавать их, а также как выполнять общие задачи, такие как отправка, перечисление и скачивание больших двоичных объектов. Примеры написаны на языке C \# и используют [клиентскую библиотеку служба хранилища Microsoft Azure для .NET](/previous-versions/azure/dn261237(v=azure.100)).

Хранилище больших двоичных объектов Azure — это служба хранения большого количества неструктурированных данных, к которым можно получить доступ практически из любой точки мира по протоколам HTTP или HTTPS. Один большой двоичный объект может быть любого размера. Большими двоичными объектами могут быть изображения, аудио- и видеофайлы, необработанные данные и файлы документов.

Так же как файлы располагаются в папках, большие двоичные объекты хранилища располагаются в контейнерах. После создания хранилища вы можете создать один или несколько контейнеров в нем. Например, можно создать хранилище с именем "Альбом", в нем создать контейнеры "изображения" для хранения файлов изображений и "аудио" для хранения аудиофайлов. После создания контейнеров в них можно отправлять индивидуальные большие двоичные объекты.

* Дополнительные сведения о программных операциях с большими двоичными объектами см. в статье [Приступая к работе с хранилищем BLOB-объектов Azure с помощью .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md).
* Общие сведения о службе хранилища Azure см. в [документации по службе хранилища](https://azure.microsoft.com/documentation/services/storage/).
* Общие сведения об облачных службах Azure см. в [документации по облачным службам](https://azure.microsoft.com/documentation/services/cloud-services/).
* Дополнительные сведения о программировании для приложений ASP.NET см. в разделе [ASP.NET](https://www.asp.net).

## <a name="access-blob-containers-in-code"></a>Доступ к контейнерам больших двоичных объектов в коде
Для программного доступа к большим двоичным объектам в проектах облачных служб необходимо добавить следующие элементы, если они еще не существуют.

1. Добавьте следующие объявления пространств имен кода в начало любого файла C#,в котором вы собираетесь получать доступ к хранилищу Azure программным способом.
   
    ```csharp
    using Microsoft.Framework.Configuration;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    using System.Threading.Tasks;
    using LogLevel = Microsoft.Framework.Logging.LogLevel;
    ```
2. Получите объект **CloudStorageAccount** , представляющий данные учетной записи хранения. Используйте следующий код, чтобы получить строку подключения и сведения об учетной записи хранения из конфигурации службы Azure.
   
    ```csharp
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("<storage account name>_AzureStorageConnectionString"));
    ```
3. Получите объект **CloudBlobClient** , чтобы указать ссылку на существующий контейнер в вашей учетной записи хранения.
   
    ```csharp
    // Create a blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    ```
4. Получите объект **CloudBlobContainer** , чтобы указать ссылку на определенный контейнер больших двоичных объектов.
   
    ```csharp
    // Get a reference to a container named "mycontainer."
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");
    ```

> [!NOTE]
> Вставьте весь код, показанный в предыдущей процедуре, перед кодом, показанным в следующих разделах.
> 
> 

## <a name="create-a-container-in-code"></a>Создание контейнера в коде
> [!NOTE]
> Некоторые интерфейсы API, которые выполняют вызовы в хранилище Azure в ASP.NET, являются асинхронными. Дополнительные сведения см. в статье [Asynchronous Programming with async and await (C#)](/previous-versions/hh191443(v=vs.140)) (Асинхронное программирование с использованием ключевых слов Async и Await (C#)). Код в приведенном ниже примере предполагает, что вы используете асинхронные методы программирования.
> 
> 

Чтобы создать контейнер в учетной записи хранения, необходимо всего лишь добавить вызов **CreateIfNotExistsAsync** , как это сделано в следующем коде.

```csharp
// If "mycontainer" doesn't exist, create it.
await container.CreateIfNotExistsAsync();
```


Чтобы сделать файлы в контейнере доступными для всех пользователей, сделайте контейнер общедоступным с помощью следующего примера кода.

```csharp
await container.SetPermissionsAsync(new BlobContainerPermissions
{
    PublicAccess = BlobContainerPublicAccessType.Blob
});
```


Любой пользователь в Интернете может видеть большие двоичные объекты в открытом контейнере, но изменить или удалить их можно только при наличии ключа доступа.

## <a name="upload-a-blob-into-a-container"></a>Отправка BLOB-объекта в контейнер
Хранилище BLOB-объектов Azure поддерживает блочные и страничные BLOB-объекты. В большинстве случаев рекомендуется использовать блочные BLOB-объекты.

Для передачи файла в блочный BLOB-объект получите ссылку на контейнер и используйте ее для получения ссылки на блочный BLOB-объект. После получения ссылки на большой двоичный объект можно передать в него любой поток данных, вызвав метод **UploadFromStream** . Эта операция создает большой двоичный объект, если он не существует, или заменяет его, если он существует. В следующем примере показано, как отправить BLOB-объект в контейнер. Предполагается, что контейнер уже был создан.

```csharp
// Retrieve a reference to a blob named "myblob".
CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = System.IO.File.OpenRead(@"path\myfile"))
{
    blockBlob.UploadFromStream(fileStream);
}
```

## <a name="list-the-blobs-in-a-container"></a>Перечисление BLOB-объектов в контейнере
Для перечисления BLOB-объектов в контейнере сначала необходимо получить ссылку на контейнер. Затем можно использовать метод **ListBlobs** контейнера, чтобы извлечь большие двоичные объекты и/или каталоги в нем. Чтобы получить доступ к обширному набору свойств и методов для возвращаемого **возвращаемого ilistblobitem**, необходимо привести его к объекту **CloudBlockBlob**, **CloudPageBlob** или **CloudBlobDirectory** . Если тип неизвестен, можно использовать проверку типов, чтобы определить нужный тип. Следующий код демонстрирует, как получить и вывести универсальный код ресурса (URI) каждого элемента в контейнере **photos** .

```csharp
// Loop over items within the container and output the length and URI.
foreach (IListBlobItem item in container.ListBlobs(null, false))
{
    if (item.GetType() == typeof(CloudBlockBlob))
    {
        CloudBlockBlob blob = (CloudBlockBlob)item;

        Console.WriteLine("Block blob of length {0}: {1}", blob.Properties.Length, blob.Uri);

    }
    else if (item.GetType() == typeof(CloudPageBlob))
    {
        CloudPageBlob pageBlob = (CloudPageBlob)item;

        Console.WriteLine("Page blob of length {0}: {1}", pageBlob.Properties.Length, pageBlob.Uri);

    }
    else if (item.GetType() == typeof(CloudBlobDirectory))
    {
        CloudBlobDirectory directory = (CloudBlobDirectory)item;

        Console.WriteLine("Directory: {0}", directory.Uri);
    }
}
```

Как показано в предыдущем примере кода, служба BLOB-объектов также использует концепцию каталогов внутри контейнеров. Таким образом, можно организовать BLOB-объекты в структуре, похожей на папки. Для примера рассмотрим следующий набор блочных BLOB-объектов в контейнере **photos**.

```output
photo1.jpg
2010/architecture/description.txt
2010/architecture/photo3.jpg
2010/architecture/photo4.jpg
2011/architecture/photo5.jpg
2011/architecture/photo6.jpg
2011/architecture/description.txt
2011/photo7.jpg
```

При вызове метода **ListBlobs** контейнера (как в примере выше) возвращаемая коллекция будет содержать объекты **CloudBlobDirectory** и **CloudBlockBlob**, представляющие каталоги и большие двоичные объекты верхнего уровня. В результате получается следующее:

```output
Directory: https://<accountname>.blob.core.windows.net/photos/2010/
Directory: https://<accountname>.blob.core.windows.net/photos/2011/
Block blob of length 505623: https://<accountname>.blob.core.windows.net/photos/photo1.jpg
```


При необходимости можно установить для параметра **UseFlatBlobListing** метода **ListBlobs** значение **true**. Это приведет к возвращению каждого большого двоичного объекта в виде **CloudBlockBlob** независимо от того, какой каталог используется. Вот как выглядит вызов метода **ListBlobs**.

```csharp
// Loop over items within the container and output the length and URI.
foreach (IListBlobItem item in container.ListBlobs(null, true))
{
    ...
}
```

А это — результат:

```output
Block blob of length 4: https://<accountname>.blob.core.windows.net/photos/2010/architecture/description.txt
Block blob of length 314618: https://<accountname>.blob.core.windows.net/photos/2010/architecture/photo3.jpg
Block blob of length 522713: https://<accountname>.blob.core.windows.net/photos/2010/architecture/photo4.jpg
Block blob of length 4: https://<accountname>.blob.core.windows.net/photos/2011/architecture/description.txt
Block blob of length 419048: https://<accountname>.blob.core.windows.net/photos/2011/architecture/photo5.jpg
Block blob of length 506388: https://<accountname>.blob.core.windows.net/photos/2011/architecture/photo6.jpg
Block blob of length 399751: https://<accountname>.blob.core.windows.net/photos/2011/photo7.jpg
Block blob of length 505623: https://<accountname>.blob.core.windows.net/photos/photo1.jpg
```

Дополнительные сведения см. в описании метода [CloudBlobContainer.ListBlobs](/rest/api/storageservices/List-Blobs).

## <a name="download-blobs"></a>Скачивание больших двоичных объектов
Для загрузки BLOB-объектов сначала нужно получить ссылку на BLOB-объект, а затем вызвать метод **DownloadToStream** . В следующем примере метод **DownloadToStream** используется для переноса содержимого большого двоичного объекта в объект потока, который затем можно сохранить в локальном файле.

```csharp
// Get a reference to a blob named "photo1.jpg".
CloudBlockBlob blockBlob = container.GetBlockBlobReference("photo1.jpg");

// Save blob contents to a file.
using (var fileStream = System.IO.File.OpenWrite(@"path\myfile"))
{
    blockBlob.DownloadToStream(fileStream);
}
```

Можно также использовать метод **DownloadToStream** , чтобы загрузить содержимое BLOB-объекта как текстовую строку.

```csharp
// Get a reference to a blob named "myblob.txt"
CloudBlockBlob blockBlob2 = container.GetBlockBlobReference("myblob.txt");

string text;
using (var memoryStream = new MemoryStream())
{
    blockBlob2.DownloadToStream(memoryStream);
    text = System.Text.Encoding.UTF8.GetString(memoryStream.ToArray());
}
```

## <a name="delete-blobs"></a>Удаление blob-объектов
Чтобы удалить большой двоичный объект, сначала нужно получить ссылку на него, а затем вызвать метод **Delete** .

```csharp
// Get a reference to a blob named "myblob.txt".
CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob.txt");

// Delete the blob.
blockBlob.Delete();
```


## <a name="list-blobs-in-pages-asynchronously"></a>Асинхронное перечисление BLOB-объектов в страницах
Если вам нужно расположить большое количество BLOB-объектов или вы хотите управлять отображением количества объектов в результате запроса, вы можете задать расположение BLOB-объектов на странице. В этом примере вы узнаете, как расположить запрошенные результаты на странице асинхронно для того, чтобы не блокировать выполнение задачи ожиданием большого объема возвращаемых данных.

В этом примере показано создание плоского списка больших двоичных объектов. Но вы также можете создать иерархический список, установив для параметра **useFlatBlobListing** в методе **ListBlobsSegmentedAsync** значение **false**.

Так как метод из примера вызывает асинхронный метод, перед ним необходимо задать ключевое слово **async**. Это позволит вернуть объект **Task**. При ожидании ключевого слова для **ListBlobsSegmentedAsync** метод приостанавливает выполнение примера до тех пор, пока задача размещения результатов не завершена.

```csharp
async public static Task ListBlobsSegmentedInFlatListing(CloudBlobContainer container)
{
    // List blobs to the console window, with paging.
    Console.WriteLine("List blobs in pages:");

    int i = 0;
    BlobContinuationToken continuationToken = null;
    BlobResultSegment resultSegment = null;

    // Call ListBlobsSegmentedAsync and enumerate the result segment returned, while the continuation token is non-null.
    // When the continuation token is null, the last page has been returned and execution can exit the loop.
    do
    {
        // This overload allows control of the page size. You can return all remaining results by passing null for the maxResults parameter,
        // or by calling a different overload.
        resultSegment = await container.ListBlobsSegmentedAsync("", true, BlobListingDetails.All, 10, continuationToken, null, null);
        if (resultSegment.Results.Count<IListBlobItem>() > 0) { Console.WriteLine("Page {0}:", ++i); }
        foreach (var blobItem in resultSegment.Results)
        {
            Console.WriteLine("\t{0}", blobItem.StorageUri.PrimaryUri);
        }
        Console.WriteLine();

        //Get the continuation token.
        continuationToken = resultSegment.ContinuationToken;
    }
    while (continuationToken != null);
}
```

## <a name="next-steps"></a>Дальнейшие действия
[!INCLUDE [vs-storage-dotnet-blobs-next-steps](../../includes/vs-storage-dotnet-blobs-next-steps.md)]