---
title: включить файл
description: включить файл
services: functions
author: craigshoemaker
manager: gwallace
ms.service: azure-functions
ms.topic: include
ms.date: 08/02/2019
ms.author: cshoe
ms.custom: include file
ms.openlocfilehash: e375a12be73c280f2778e6e28efb709b9116a4cf
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "100381666"
---
### <a name="default"></a>По умолчанию

Для записи больших двоичных объектов можно выполнить привязку к следующим типам:

* `TextWriter`
* `out string`
* `out Byte[]`
* `CloudBlobStream`
* `Stream`
* `CloudBlobContainer`<sup>1</sup>
* `CloudBlobDirectory`
* `ICloudBlob`<sup>2</sup>
* `CloudBlockBlob`<sup>2</sup>
* `CloudPageBlob`<sup>2</sup>
* `CloudAppendBlob`<sup>2</sup>

<sup>1</sup> Требует привязки in `direction` в файле *function.json* или `FileAccess.Read` в библиотеке классов C#. Однако вы можете использовать объект контейнера, предоставляемый средой выполнения для записи таких операций, как отправка больших двоичных объектов в контейнер.

<sup>2</sup> Требует привязки inout `direction` в файле *function.json* или `FileAccess.ReadWrite` в библиотеке классов C#.

Если при попытке привязаться к одному из типов SDK для службы хранилища получено сообщение об ошибке, убедитесь, что у вас есть ссылка на [правильную версию пакета SDK для службы хранилища](../articles/azure-functions/functions-bindings-storage-blob.md#azure-storage-sdk-version-in-functions-1x).

Привязку к `string` или `Byte[]` рекомендуется использовать, только если большой двоичный объект имеет небольшой размер, так как в память загружается все содержимое большого двоичного объекта. Как правило, предпочтительнее использовать тип `Stream` или `CloudBlockBlob`. Дополнительные сведения см. в разделе о [параллелизме и использовании памяти](../articles/azure-functions/functions-bindings-storage-blob-trigger.md#concurrency-and-memory-usage) выше в этой статье.

### <a name="additional-types"></a>Дополнительные типы

В приложениях, в которых используется[расширение службы хранилища версии 5.0.0 или выше](../articles/azure-functions/functions-bindings-storage-blob.md#storage-extension-5x-and-higher), также могут использоваться типы из [пакета Azure SDK для .NET](/dotnet/api/overview/azure/storage.blobs-readme). В этой версии прекращена поддержка устаревших типов `CloudBlobContainer`, `CloudBlobDirectory`, `ICloudBlob`, `CloudBlockBlob`, `CloudPageBlob` и `CloudAppendBlob`. Вместо них теперь поддерживаются следующие типы:

- [BlobContainerClient](/dotnet/api/azure.storage.blobs.blobcontainerclient)<sup>1</sup>
- [BlobClient](/dotnet/api/azure.storage.blobs.blobclient)<sup>2</sup>
- [BlockBlobClient](/dotnet/api/azure.storage.blobs.specialized.blockblobclient)<sup>2</sup>
- [PageBlobClient](/dotnet/api/azure.storage.blobs.specialized.pageblobclient)<sup>2</sup>
- [AppendBlobClient](/dotnet/api/azure.storage.blobs.specialized.appendblobclient)<sup>2</sup>
- [BlobBaseClient](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient)<sup>2</sup>

<sup>1</sup> Требует привязки in `direction` в файле *function.json* или `FileAccess.Read` в библиотеке классов C#. Однако вы можете использовать объект контейнера, предоставляемый средой выполнения для записи таких операций, как отправка больших двоичных объектов в контейнер.

<sup>2</sup> Требует привязки inout `direction` в файле *function.json* или `FileAccess.ReadWrite` в библиотеке классов C#.

Примеры использования этих типов см. в разделе [репозитория GitHub для расширения](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Microsoft.Azure.WebJobs.Extensions.Storage.Blobs#examples).
