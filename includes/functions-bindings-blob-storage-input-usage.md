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
ms.openlocfilehash: 938f55ae0ba911ea3a97cd49e6424bf8aaefdc76
ms.sourcegitcommit: d4734bc680ea221ea80fdea67859d6d32241aefc
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/14/2021
ms.locfileid: "100381700"
---
### <a name="default"></a>По умолчанию

Для входной привязки BLOB-объектов можно использовать следующие типы параметров:

* `Stream`
* `TextReader`
* `string`
* `Byte[]`
* `CloudBlobContainer`
* `CloudBlobDirectory`
* `ICloudBlob`<sup>1</sup>
* `CloudBlockBlob`<sup>1</sup>
* `CloudPageBlob`<sup>1</sup>
* `CloudAppendBlob`<sup>1</sup>

<sup>1</sup> Требует привязки inout `direction` в файле *function.json* или `FileAccess.ReadWrite` в библиотеке классов C#.

Если при попытке привязаться к одному из типов SDK для службы хранилища получено сообщение об ошибке, убедитесь, что у вас есть ссылка на [правильную версию пакета SDK для службы хранилища](../articles/azure-functions/functions-bindings-storage-blob.md#azure-storage-sdk-version-in-functions-1x).

Привязку к `string` или `Byte[]` рекомендуется использовать, только если большой двоичный объект имеет небольшой размер, так как в память загружается все содержимое большого двоичного объекта. Как правило, предпочтительнее использовать тип `Stream` или `CloudBlockBlob`. Дополнительные сведения см. в разделе о [параллелизме и использовании памяти](../articles/azure-functions/functions-bindings-storage-blob-trigger.md#concurrency-and-memory-usage) выше в этой статье.

### <a name="additional-types"></a>Дополнительные типы

Приложения, использующие [версию 5.0.0 или более поздней версии,](../articles/azure-functions/functions-bindings-storage-blob.md#storage-extension-5x-and-higher) могут также использовать типы из [пакета Azure SDK для .NET](/dotnet/api/overview/azure/storage.blobs-readme). Эта версия отключает поддержку устаревших типов,,,, `CloudBlobContainer` `CloudBlobDirectory` `ICloudBlob` `CloudBlockBlob` `CloudPageBlob` и `CloudAppendBlob` в пользу для следующих типов:

- [BlobContainerClient](/dotnet/api/azure.storage.blobs.blobcontainerclient)
- [Блобклиент](/dotnet/api/azure.storage.blobs.blobclient)<sup>1</sup>
- [Блоккблобклиент](/dotnet/api/azure.storage.blobs.specialized.blockblobclient)<sup>1</sup>
- [Пажеблобклиент](/dotnet/api/azure.storage.blobs.specialized.pageblobclient)<sup>1</sup>
- [Аппендблобклиент](/dotnet/api/azure.storage.blobs.specialized.appendblobclient)<sup>1</sup>
- [Блоббасеклиент](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient)<sup>1</sup>

<sup>1</sup> Требует привязки inout `direction` в файле *function.json* или `FileAccess.ReadWrite` в библиотеке классов C#.

Примеры использования этих типов см. [в репозитории GitHub для расширения](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Microsoft.Azure.WebJobs.Extensions.Storage.Blobs#examples).
