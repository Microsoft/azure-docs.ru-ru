---
title: Анонимный доступ к открытым контейнерам и BLOB-объектам с помощью .NET
titleSuffix: Azure Storage
description: Используйте клиентскую библиотеку службы хранилища Azure для .NET для анонимного доступа к открытым контейнерам и BLOB-объектам.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 08/02/2020
ms.author: tamram
ms.reviewer: fryu
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: 2437c5b3272163b3931d7417c84e761c591aec85
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90088822"
---
# <a name="access-public-containers-and-blobs-anonymously-with-net"></a>Анонимный доступ к открытым контейнерам и BLOB-объектам с помощью .NET

Служба хранилища Azure поддерживает необязательный общий доступ на чтение для контейнеров и больших двоичных объектов. Клиенты могут анонимно обращаться к открытым контейнерам и BLOB-объектам с помощью клиентских библиотек службы хранилища Azure, а также с помощью других средств и служебных программ, поддерживающих доступ к данным в службе хранилища Azure.

В этой статье показано, как получить доступ к открытому контейнеру или BLOB-объекту из .NET. Сведения о настройке анонимного доступа для чтения в контейнере см. в разделе [Настройка анонимного общего доступа на чтение для контейнеров и больших двоичных объектов](anonymous-read-access-configure.md). Сведения о предотвращении анонимного доступа к учетной записи хранения см. [в разделе Предотвращение анонимного общего доступа на чтение к контейнерам и BLOB-объектам](anonymous-read-access-prevent.md).

Клиент, который обращается к контейнерам и большим двоичным объектам анонимно, может использовать конструкторы, не требующие учетных данных. В следующих примерах показаны несколько различных способов анонимного обращения к контейнерам и BLOB-объектам.

## <a name="create-an-anonymous-client-object"></a>Создание анонимного клиентского объекта

Вы можете создать новый объект клиента службы для анонимного доступа, предоставив конечную точку хранилища BLOB-объектов для учетной записи. Тем не менее необходимо также знать имя контейнера в этой учетной записи, которая доступна для анонимного доступа.

# <a name="net-v12-sdk"></a>[\.NET (пакет SDK версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Security.cs" id="Snippet_CreateAnonymousBlobClient":::

# <a name="net-v11-sdk"></a>[\..NET (пакет SDK версии 11)](#tab/dotnet11)

```csharp
public static void CreateAnonymousBlobClient()
{
    // Create the client object using the Blob storage endpoint for your account.
    CloudBlobClient blobClient = new CloudBlobClient(
        new Uri(@"https://storagesamples.blob.core.windows.net"));

    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = blobClient.GetContainerReference("sample-container");

    // Read the container's properties. 
    // Note this is only possible when the container supports full public read access.
    container.FetchAttributes();
    Console.WriteLine(container.Properties.LastModified);
    Console.WriteLine(container.Properties.ETag);
}
```

---

## <a name="reference-a-container-anonymously"></a>Анонимная ссылка на контейнер

При наличии URL-адреса контейнера, который доступен анонимно, можно воспользоваться им для прямой ссылки на контейнер.

# <a name="net-v12-sdk"></a>[\.NET (пакет SDK версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Security.cs" id="Snippet_ListBlobsAnonymously":::

# <a name="net-v11-sdk"></a>[\..NET (пакет SDK версии 11)](#tab/dotnet11)

```csharp
public static void ListBlobsAnonymously()
{
    // Get a reference to a container that's available for anonymous access.
    CloudBlobContainer container = new CloudBlobContainer(
        new Uri(@"https://storagesamples.blob.core.windows.net/sample-container"));

    // List blobs in the container.
    // Note this is only possible when the container supports full public read access.
    foreach (IListBlobItem blobItem in container.ListBlobs())
    {
        Console.WriteLine(blobItem.Uri);
    }
}
```

---

## <a name="reference-a-blob-anonymously"></a>Анонимная ссылка на большой двоичный объект

Если имеется URL-адрес большого двоичного объекта, который доступен для анонимного доступа, можно создать прямую ссылку на этот большой двоичный объект, используя URL-адрес.

# <a name="net-v12-sdk"></a>[\.NET (пакет SDK версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Security.cs" id="Snippet_DownloadBlobAnonymously":::

# <a name="net-v11-sdk"></a>[\..NET (пакет SDK версии 11)](#tab/dotnet11)

```csharp
public static void DownloadBlobAnonymously()
{
    CloudBlockBlob blob = new CloudBlockBlob(
        new Uri(@"https://storagesamples.blob.core.windows.net/sample-container/logfile.txt"));
    blob.DownloadToFile(@"C:\Temp\logfile.txt", FileMode.Create);
}
```

---

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка анонимного общего доступа на чтение для контейнеров и больших двоичных объектов](anonymous-read-access-configure.md)
- [Запретить анонимный общий доступ на чтение к контейнерам и BLOB-объектам](anonymous-read-access-prevent.md)
- [Авторизация доступа к службе хранилища Azure](../common/storage-auth.md)
