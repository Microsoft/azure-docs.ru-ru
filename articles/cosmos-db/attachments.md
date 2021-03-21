---
title: Azure Cosmos DB вложения
description: В этой статье представлен обзор Azure Cosmos DBных вложений.
author: aliuy
ms.author: andrl
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 08/07/2020
ms.reviewer: sngun
ms.openlocfilehash: a8e968d05a1f844a79d2e42d10c323ed4c392424
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102521226"
---
# <a name="azure-cosmos-db-attachments"></a>Azure Cosmos DB вложения
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-mongodb-api.md)]

Azure Cosmos DB вложения — это специальные элементы, содержащие ссылки на связанные метаданные с внешним BLOB-объектом или файлом мультимедиа.

Azure Cosmos DB поддерживает два типа вложений:

* **Неуправляемые вложения** — это обертка ссылки URI на большой двоичный объект, хранящийся во внешней службе (например, служба хранилища Azure, OneDrive и т. д.). Этот подход похож на хранение свойства URI в стандартном Azure Cosmos DB элементе.
* **Управляемые вложения** — это большие двоичные объекты, которые хранятся на внутреннем уровне Azure Cosmos DB и предоставляются через формируемый системой mediaLink.


> [!NOTE]
> Вложение является устаревшей функцией. Их поддержка ограничена, если вы уже используете эту функцию.
> 
> Вместо использования вложений рекомендуется использовать хранилище BLOB-объектов Azure в качестве службы хранилища больших двоичных объектов для хранения данных большого двоичного объекта. Вы можете продолжать хранить метаданные, связанные с большими двоичными объектами, а также ссылочные ссылки URI в Azure Cosmos DB как свойства элемента. Хранение этих данных в Azure Cosmos DB предоставляет возможность запрашивать метаданные и ссылки на большие двоичные объекты, хранящиеся в хранилище BLOB-объектов Azure.
> 
> Корпорация Майкрософт стремится предоставить минимальное 36-месячное уведомление до полного использования вложений, которое будет объявлено позже.

## <a name="known-limitations"></a>Известные ограничения

Управляемые вложения Azure Cosmos DB отличаются от поддержки стандартных элементов, для которых она предлагает неограниченную масштабируемость, глобальное распределение и интеграцию с другими службами Azure.

- Вложения не поддерживаются во всех версиях пакетов SDK для Azure Cosmos DB.
- Объем управляемых вложений ограничен 2 ГБ на учетную запись базы данных.
- Управляемые вложения несовместимы с глобальным распределением Azure Cosmos DB, и они не реплицируются в разных регионах.

> [!NOTE]
> Azure Cosmos DB API для MongoDB версии 3,2 использует управляемые вложения для Гридфс и подвержены тем же ограничениям.
>
> Мы рекомендуем разработчикам использовать набор функций MongoDB Гридфс для обновления до Azure Cosmos DB API для MongoDB версии 3,6 или более поздней, который отделяется от вложений и обеспечивает лучшую работу. Кроме того, разработчики, использующие набор функций MongoDB Гридфс, также должны подумать об использовании хранилища BLOB-объектов Azure, которое предназначено для хранения содержимого большого двоичного объекта и предлагает расширенные функциональные возможности с меньшими затратами по сравнению с Гридфс.

## <a name="migrating-attachments-to-azure-blob-storage"></a>Миграция вложений в хранилище BLOB-объектов Azure

Мы рекомендуем перенести Azure Cosmos DB вложения в хранилище BLOB-объектов Azure, выполнив следующие действия:

1. Скопируйте данные вложений из исходного контейнера Azure Cosmos DB в целевой контейнер хранилища BLOB-объектов Azure.
2. Проверьте отправленные данные большого двоичного объекта в целевом контейнере хранилища BLOB-объектов Azure.
3. Если это применимо, добавьте ссылки URI в большие двоичные объекты, содержащиеся в хранилище BLOB-объектов Azure, в качестве строковых свойств в наборе данных Azure Cosmos DB.
4. Рефакторинг кода приложения для чтения и записи больших двоичных объектов из нового контейнера хранилища BLOB-объектов Azure.

В следующем примере кода DotNet показано, как копировать вложения из Azure Cosmos DB в хранилище BLOB-объектов Azure в рамках процесса миграции с помощью пакета SDK .NET версии 2 Azure Cosmos DB и пакета SDK .NET для хранилища BLOB-объектов Azure 12. Обязательно замените `<placeholder values>` для учетной записи источника Azure Cosmos DB и целевого контейнера хранилища BLOB-объектов Azure.

```csharp

using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.Azure.Documents;
using Microsoft.Azure.Documents.Client;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

namespace attachments
{
    class Program
    {
        private static string cosmosAccount = "<Your_Azure_Cosmos_account_URI>";
        private static string cosmosKey = "<Your_Azure_Cosmos_account_PRIMARY_KEY>";
        private static string cosmosDatabaseName = "<Your_Azure_Cosmos_database>";
        private static string cosmosCollectionName = "<Your_Azure_Cosmos_collection>";
        private static string storageConnectionString = "<Your_Azure_Storage_connection_string>";
        private static string storageContainerName = "<Your_Azure_Storage_container_name>";
        private static DocumentClient cosmosClient = new DocumentClient(new Uri(cosmosAccount), cosmosKey);
        private static BlobServiceClient storageClient = new BlobServiceClient(storageConnectionString);
        private static BlobContainerClient storageContainerClient = storageClient.GetBlobContainerClient(storageContainerName);

        static void Main(string[] args)
        {
            CopyAttachmentsToBlobsAsync().Wait();
        }

        private async static Task CopyAttachmentsToBlobsAsync()
        {
            Console.WriteLine("Copying Azure Cosmos DB Attachments to Azure Blob Storage ...");

            int totalCount = 0;
            string docContinuation = null;

            // Iterate through each item (document in v2) in the Azure Cosmos DB container (collection in v2) to look for attachments.
            do
            {
                FeedResponse<dynamic> response = await cosmosClient.ReadDocumentFeedAsync(
                    UriFactory.CreateDocumentCollectionUri(cosmosDatabaseName, cosmosCollectionName),
                    new FeedOptions
                    {
                        MaxItemCount = -1,
                        RequestContinuation = docContinuation
                    });
                docContinuation = response.ResponseContinuation;

                foreach (Document document in response)
                {
                    string attachmentContinuation = null;
                    PartitionKey docPartitionKey = new PartitionKey(document.Id);

                    // Iterate through each attachment within the item (if any).
                    do
                    {
                        FeedResponse<Attachment> attachments = await cosmosClient.ReadAttachmentFeedAsync(
                            document.SelfLink,
                            new FeedOptions
                            {
                                PartitionKey = docPartitionKey,
                                RequestContinuation = attachmentContinuation
                            }
                        );
                        attachmentContinuation = attachments.ResponseContinuation;

                        foreach (var attachment in attachments)
                        {
                            // Download the attachment in to local memory.
                            MediaResponse content = await cosmosClient.ReadMediaAsync(attachment.MediaLink);

                            byte[] buffer = new byte[content.ContentLength];
                            await content.Media.ReadAsync(buffer, 0, buffer.Length);

                            // Upload the locally buffered attachment to blob storage
                            string blobId = String.Concat(document.Id, "-", attachment.Id);

                            Azure.Response<BlobContentInfo> uploadedBob = await storageContainerClient.GetBlobClient(blobId).UploadAsync(
                                new MemoryStream(buffer, writable: false),
                                true
                            );

                            Console.WriteLine("Copied attachment ... Item Id: {0} , Attachment Id: {1}, Blob Id: {2}", document.Id, attachment.Id, blobId);
                            totalCount++;

                            // Clean up attachment from Azure Cosmos DB.
                            // Warning: please verify you've succesfully migrated attachments to blog storage prior to cleaning up Azure Cosmos DB.
                            // await cosmosClient.DeleteAttachmentAsync(
                            //     attachment.SelfLink,
                            //     new RequestOptions { PartitionKey = docPartitionKey }
                            // );

                            // Console.WriteLine("Cleaned up attachment ... Document Id: {0} , Attachment Id: {1}", document.Id, attachment.Id);
                        }

                    } while (!string.IsNullOrEmpty(attachmentContinuation));
                }
            }
            while (!string.IsNullOrEmpty(docContinuation));

            Console.WriteLine("Finished copying {0} attachments to blob storage", totalCount);
        }
    }
}

```

## <a name="next-steps"></a>Дальнейшие действия

- Приступая к работе с [хранилищем BLOB-объектов Azure](../storage/blobs/storage-quickstart-blobs-dotnet.md)
- Получение ссылок на использование вложений с помощью [пакета SDK .NET версии 2 для Azure Cosmos DB](/dotnet/api/microsoft.azure.documents.attachment)
- Получение ссылок на использование вложений с помощью [пакета SDK для Java для Azure Cosmos DB версии 2](/java/api/com.microsoft.azure.documentdb.attachment)
- Получение ссылок на использование вложений с помощью [REST API Azure Cosmos DB](/rest/api/cosmos-db/attachments)
