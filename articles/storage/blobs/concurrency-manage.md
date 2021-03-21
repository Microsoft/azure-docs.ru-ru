---
title: Управление параллелизмом в хранилище BLOB-объектов
titleSuffix: Azure Storage
description: Узнайте, как управлять несколькими потоками записи в большой двоичный объект, реализовав оптимистичный или пессимистичный параллелизм в приложении. Оптимистическая блокировка проверяет значение ETag для большого двоичного объекта и сравнивает его с предоставленным тегом ETag. Пессимистичный параллелизм использует эксклюзивную аренду для блокировки большого двоичного объекта на другие модули записи.
services: storage
author: tamram
ms.service: storage
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 12/01/2020
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: ea0bed0884a3a03e2cd15b274b2afb0f054b0cbd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96523637"
---
# <a name="managing-concurrency-in-blob-storage"></a>Управление параллелизмом в хранилище BLOB-объектов

Современные приложения часто используют несколько пользователей для одновременного просмотра и обновления данных. Разработчикам приложений необходимо тщательно подумать о том, как обеспечить предсказуемый опыт работы конечных пользователей, особенно для сценариев, в которых несколько пользователей могут обновлять одни и те же данные. Обычно разработчики рассматривают три следующие основные стратегии конфликтов данных:  

- **Оптимистическая блокировка**. приложение, выполняющее обновление, в рамках его обновления определяет, изменились ли данные с момента последнего считывания этих данных приложением. Например, если два пользователя, просматривающие вики-страницу, обновляют эту страницу, то платформа wiki должна гарантировать, что второе обновление не перезапишет первое обновление. Кроме того, необходимо убедиться, что оба пользователя понимают, успешно ли их обновление. Стратегия наиболее часто применяется в веб-приложениях.

- **Пессимистичный параллелизм**. приложение, которое планируется выполнить обновление, заблокирует объект, запрещающий другим пользователям обновлять данные до тех пор, пока блокировка не будет снята. Например, в сценарии репликации с первичным или вторичным данными, в котором только первично выполняет обновления, первичная, как правило, удерживает монопольную блокировку данных в течение продолжительного периода времени, чтобы никто не мог обновить их.

- **Побеждает последний модуль записи**: подход, который позволяет выполнять операции обновления, не определяя сначала, обновлялись ли данные другим приложением с момента считывания. Такой подход обычно используется, когда данные секционированы таким образом, что несколько пользователей не будут обращаться к одним и тем же данным одновременно. Она также применяется при обработке кратковременных потоков данных.

Служба хранилища Azure поддерживает все три стратегии, хотя она позволяет обеспечить полную поддержку оптимистичного и пессимистического параллелизма. Служба хранилища Azure разработана так, чтобы обеспечить надежную модель согласованности, гарантирующую, что после выполнения службой операции вставки или обновления последующие операции чтения возвращают Последнее обновление.

Помимо выбора соответствующей стратегии параллелизма, разработчики также должны знать, как платформа хранения изолирует изменения, в особенности изменения одного и того же объекта в транзакциях. Служба хранилища Azure использует изоляцию моментального снимка, чтобы разрешить операции чтения одновременно с операциями записи в пределах одной секции. Изоляция моментальных снимков гарантирует, что все операции чтения возвращают согласованный моментальный снимок данных даже во время обновления.

Для управления доступом к BLOB-объектам и контейнерам можно использовать модели оптимистичного и пессимистического параллелизма. Если стратегия не указана явно, по умолчанию используется последний модуль записи.  

## <a name="optimistic-concurrency"></a>Оптимистическая блокировка

Служба хранилища Azure назначает идентификатор каждому сохраненному объекту. Этот идентификатор обновляется каждый раз при выполнении операции записи над объектом. Идентификатор возвращается клиенту как часть ответа HTTP GET в заголовке ETag, который определяется протоколом HTTP.

Клиент, выполняющий обновление, может отправить исходный ETag вместе с условным заголовком, чтобы гарантировать, что обновление будет выполнено только при соблюдении определенного условия. Например, если указан заголовок **If-Match** , служба хранилища Azure проверяет, совпадает ли значение ETag, указанное в запросе на обновление, с ETag для обновляемого объекта. Дополнительные сведения о условных заголовках см. в разделе [Указание условных заголовков для операций службы BLOB-объектов](/rest/api/storageservices/specifying-conditional-headers-for-blob-service-operations).

Для этой команды используется следующая структура:  

1. Получите большой двоичный объект из службы хранилища Azure. Ответ содержит значение заголовка HTTP ETag, определяющее текущую версию объекта.
1. При обновлении большого двоичного объекта включите значение ETag, полученное на шаге 1, в условном заголовке **If-Match** запроса на запись. Служба хранилища Azure сравнивает значение ETag в запросе с текущим значением ETag большого двоичного объекта.
1. Если текущее значение ETag большого двоичного объекта отличается от указанного в условном заголовке **If-Match** , предоставленном в запросе, служба хранилища Azure возвращает код состояния HTTP 412 (предусловие не выполнено). Эта ошибка указывает клиенту, что другой процесс обновил большой двоичный объект с момента его первого извлечения клиентом.
1. Если текущее значение ETag большого двоичного объекта совпадает с версией ETag в условном заголовке **If-Match** в запросе, служба хранилища Azure выполняет запрошенную операцию и обновляет текущее значение ETag большого двоичного объекта.  

В следующих примерах кода показано, как создать условие " **If-Match** " для запроса на запись, который проверяет значение ETag для большого двоичного объекта. Служба хранилища Azure оценивает, совпадает ли текущий ETag BLOB-объекта с тегом ETag, указанным в запросе, и выполняет операцию записи, только если два значения ETag совпадают. Если другой процесс обновил большой двоичный объект в промежуточном состоянии, служба хранилища Azure выводит сообщение об изменении состояния HTTP 412 (предусловие не выполнено).  

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Concurrency.cs" id="Snippet_DemonstrateOptimisticConcurrencyBlob":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

```csharp
public void DemonstrateOptimisticConcurrencyBlob(string containerName, string blobName)
{
    Console.WriteLine("Demonstrate optimistic concurrency");

    // Parse connection string and create container.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExists();

    // Create test blob. The default strategy is last writer wins, so
    // write operation will overwrite existing blob if present.
    CloudBlockBlob blockBlob = container.GetBlockBlobReference(blobName);
    blockBlob.UploadText("Hello World!");

    // Retrieve the ETag from the newly created blob.
    string originalETag = blockBlob.Properties.ETag;
    Console.WriteLine("Blob added. Original ETag = {0}", originalETag);

    /// This code simulates an update by another client.
    string helloText = "Blob updated by another client.";
    // No ETag was provided, so original blob is overwritten and ETag updated.
    blockBlob.UploadText(helloText);
    Console.WriteLine("Blob updated. Updated ETag = {0}", blockBlob.Properties.ETag);

    // Now try to update the blob using the original ETag value.
    try
    {
        Console.WriteLine(@"Attempt to update blob using original ETag
                            to generate if-match access condition");
        blockBlob.UploadText(helloText, accessCondition: AccessCondition.GenerateIfMatchCondition(originalETag));
    }
    catch (StorageException ex)
    {
        if (ex.RequestInformation.HttpStatusCode == (int)HttpStatusCode.PreconditionFailed)
        {
            Console.WriteLine(@"Precondition failure as expected.
                                Blob's ETag does not match.");
        }
        else
        {
            throw;
        }
    }
    Console.WriteLine();
}
```

---

Служба хранилища Azure также поддерживает другие условные заголовки, включая, например, **If-Modified-** with, **if-без изменений-с** и **If-None-Match**. Дополнительные сведения см. в статье [Указание условных заголовков для операций службы BLOB-объектов](/rest/api/storageservices/specifying-conditional-headers-for-blob-service-operations).  

## <a name="pessimistic-concurrency-for-blobs"></a>Пессимистичный параллелизм для BLOB-объектов

Для блокировки BLOB-объекта в целях монопольного использования вы можете получить его в аренду . При получении аренды указывается длительность аренды. Конечная Аренда может быть действительной в диапазоне от 15 до 60 секунд. Аренда также может быть бесконечной, что составляет эксклюзивную блокировку. Вы можете продлить конечную аренду, чтобы продлить ее, и вы можете освободить аренду по завершении работы с ней. Служба хранилища Azure автоматически освобождает конечные аренды по истечении срока их действия.  

Аренда обеспечивает поддержку различных стратегий синхронизации, включая монопольные операции записи и чтения, монопольные операции записи и монопольного чтения, а также общие операции записи и монопольного чтения. Если Аренда существует, служба хранилища Azure обеспечивает монопольный доступ к операциям записи для держателя аренды. Тем не менее, чтобы обеспечить исключительные права для операций чтения, разработчик должен убедиться, что все клиентские приложения используют идентификатор аренды и что только один клиент в каждый момент времени имеет действительный идентификатор аренды. Операции чтения, не имеющие идентификатора аренды, окажутся в общем чтении.  

В следующих примерах кода показано, как получить эксклюзивную аренду для большого двоичного объекта, обновить содержимое большого двоичного объекта, указав идентификатор аренды, а затем освободить аренду. Если Аренда активна и идентификатор аренды не указан в запросе на запись, операция записи завершается ошибкой с кодом 412 (предусловие не выполнено).  

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Concurrency.cs" id="Snippet_DemonstratePessimisticConcurrencyBlob":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

```csharp
public void DemonstratePessimisticConcurrencyBlob(string containerName, string blobName)
{
    Console.WriteLine("Demonstrate pessimistic concurrency");

    // Parse connection string and create container.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExists();

    CloudBlockBlob blockBlob = container.GetBlockBlobReference(blobName);
    blockBlob.UploadText("Hello World!");
    Console.WriteLine("Blob added.");

    // Acquire lease for 15 seconds.
    string lease = blockBlob.AcquireLease(TimeSpan.FromSeconds(15), null);
    Console.WriteLine("Blob lease acquired. Lease = {0}", lease);

    // Update blob using lease. This operation should succeed.
    const string helloText = "Blob updated";
    var accessCondition = AccessCondition.GenerateLeaseCondition(lease);
    blockBlob.UploadText(helloText, accessCondition: accessCondition);
    Console.WriteLine("Blob updated using an exclusive lease");

    // Simulate another client attempting to update to blob without providing lease.
    try
    {
        // Operation will fail as no valid lease was provided.
        Console.WriteLine("Now try to update blob without valid lease.");
        blockBlob.UploadText("Update operation will fail without lease.");
    }
    catch (StorageException ex)
    {
        if (ex.RequestInformation.HttpStatusCode == (int)HttpStatusCode.PreconditionFailed)
        {
            Console.WriteLine(@"Precondition failure error as expected.
                                Blob lease not provided.");
        }
        else
        {
            throw;
        }
    }

    // Release lease proactively.
    blockBlob.ReleaseLease(accessCondition);
    Console.WriteLine();
}
```

---

## <a name="pessimistic-concurrency-for-containers"></a>Пессимистичный параллелизм для контейнеров

Аренда в контейнерах включает те же стратегии синхронизации, которые поддерживаются для больших двоичных объектов, включая монопольный доступ на запись и чтение, монопольную запись и монопольное чтение, а также общие операции записи и монопольного чтения. Однако для контейнеров монопольная блокировка применяется только к операциям удаления. Для удаления контейнера с активной арендой клиент наряду с запросом на удаление должен ввести активный идентификатор аренды. Все остальные операции с контейнером будут выполнены в арендованном контейнере без идентификатора аренды.  

## <a name="next-steps"></a>Дальнейшие действия

* [Указание условных заголовков для операций службы BLOB-объектов](/rest/api/storageservices/specifying-conditional-headers-for-blob-service-operations)
* [Аренда контейнера](/rest/api/storageservices/lease-container)
* [Аренда BLOB-объекта](/rest/api/storageservices/lease-blob)
