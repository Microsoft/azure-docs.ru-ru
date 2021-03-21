---
title: Хранилище BLOB-объектов Azure в качестве источника службы "Сетка событий"
description: Описание свойств событий хранилища больших двоичных объектов в службе "Сетка событий Azure"
ms.topic: conceptual
ms.date: 02/11/2021
ms.openlocfilehash: 893e86ecf220ceb327eed9c6f95be4c7ed1afb1c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100363650"
---
# <a name="azure-blob-storage-as-an-event-grid-source"></a>Хранилище BLOB-объектов Azure в качестве источника службы "Сетка событий"

В этой статье описаны свойства и схема событий хранилища BLOB-объектов. Общие сведения о схемах событий см. в статье [Схема событий службы "Сетка событий Azure"](event-schema.md). Здесь также приводится список быстрых запусков и учебников по использованию хранилища BLOB-объектов Azure в качестве источника событий.


>[!NOTE]
> Интеграция событий поддерживается только для учетных записей хранения типа **StorageV2 (общее назначение версии 2)**, **блоккблобстораже** и **блобстораже** . **Хранилище (общее назначение** версии 1) *не поддерживает интеграцию со службой "* сетка событий".

## <a name="available-event-types"></a>Доступные типы событий

### <a name="list-of-events-for-blob-rest-apis"></a>Список событий для API-интерфейсов RESTFUL для BLOB-объектов

Эти события активируются, когда клиент создает, заменяет или удаляет большой двоичный объект, вызывая интерфейсы API-интерфейсов RESTFUL для больших двоичных объектов.

> [!NOTE]
> `$logs` `$blobchangefeed` Контейнеры и не интегрированы со службой "Сетка событий", поэтому действия в этих контейнерах не будут создавать события. Кроме того, использование конечной точки DFS *`(abfss://URI) `* для учетных записей с включенными неиерархическими пространствами имен не приведет к формированию событий, но конечная точка большого двоичного объекта *`(wasb:// URI)`* создаст события.

 |Имя события |Описание|
 |----------|-----------|
 |**Microsoft.Storage.BlobCreated** |Активируется при создании или замене большого двоичного объекта. <br>В частности, это событие активируется, когда клиенты используют `PutBlob` `PutBlockList` операции, или `CopyBlob` , доступные в REST API больших двоичных объектов.   |
 |**Microsoft.Storage.BlobDeleted** |Активируется при удалении большого двоичного объекта. <br>В частности, это событие активируется, когда клиенты вызывают `DeleteBlob` операцию, доступную в REST API больших двоичных объектов. |

> [!NOTE]
> Для **хранилища BLOB-объектов Azure**, если необходимо, чтобы событие **Microsoft. Storage. BlobCreated** вызывалось только при полной фиксации блочного BLOB-объекта, отфильтруйте событие для `CopyBlob` `PutBlob` вызовов, и `PutBlockList` REST API. Эти вызовы API активируют событие **Microsoft. Storage. BlobCreated** только после полной фиксации данных в блочном BLOB-объекте. Сведения о создании фильтра см. в разделе [Фильтрация событий для сетки событий](./how-to-filter-events.md).

### <a name="list-of-the-events-for-azure-data-lake-storage-gen-2-rest-apis"></a>Список событий для интерфейсов API для Azure Data Lake Storage поколения 2

Эти события активируются при включении иерархического пространства имен в учетной записи хранения, а клиенты используют Azure Data Lake Storage 2-го поколения интерфейсы API. Дополнительные сведения грамме Azure Data Lake Storage 2-го поколения см. [в статье Введение в Azure Data Lake Storage 2-го поколения](../storage/blobs/data-lake-storage-introduction.md).

|Имя события|Описание|
|----------|-----------|
|**Microsoft.Storage.BlobCreated** | Активируется при создании или замене большого двоичного объекта. <br>В частности, это событие активируется, когда клиенты используют `CreateFile` `FlushWithClose` операции и, доступные в Azure Data Lake Storage 2-го поколения REST API. |
|**Microsoft.Storage.BlobDeleted** |Активируется при удалении большого двоичного объекта. <br>В частности, это событие также активируется, когда клиенты вызывают `DeleteFile` операцию, доступную в REST API Azure Data Lake Storage 2-го поколения. |
|**Microsoft. Storage. Блобренамед**|Активируется при переименовании большого двоичного объекта. <br>В частности, это событие активируется, когда клиенты используют `RenameFile` операцию, доступную в REST API Azure Data Lake Storage 2-го поколения.|
|**Microsoft. Storage. Директорикреатед**|Активируется при создании каталога. <br>В частности, это событие активируется, когда клиенты используют `CreateDirectory` операцию, доступную в REST API Azure Data Lake Storage 2-го поколения.|
|**Microsoft. Storage. Директориренамед**|Активируется при переименовании каталога. <br>В частности, это событие активируется, когда клиенты используют `RenameDirectory` операцию, доступную в REST API Azure Data Lake Storage 2-го поколения.|
|**Microsoft. Storage. Директориделетед**|Активируется при удалении каталога. <br>В частности, это событие активируется, когда клиенты используют `DeleteDirectory` операцию, доступную в REST API Azure Data Lake Storage 2-го поколения.|

> [!NOTE]
> Для **Azure Data Lake Storage 2-го поколения**, если необходимо, чтобы событие **Microsoft. Storage. BlobCreated** вызывалось только при полной фиксации блочного BLOB-объекта, отфильтруйте событие для `FlushWithClose` вызова REST API. Этот вызов API активирует событие **Microsoft. Storage. BlobCreated** только после полной фиксации данных в блочном BLOB-объекте. Сведения о создании фильтра см. в разделе [Фильтрация событий для сетки событий](./how-to-filter-events.md).

## <a name="example-event"></a>Пример события
При активации события служба Сетки событий отправляет данные о нем на подписанную конечную точку. В этом разделе содержится пример того, как будут выглядеть данные для каждого события хранилища BLOB-объектов.

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

### <a name="microsoftstorageblobcreated-event"></a>Событие Microsoft. Storage. BlobCreated

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/test-container/blobs/new-file.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "\"0x8D4BCC2E4835CD0\"",
    "contentType": "text/plain",
    "contentLength": 524288,
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.blob.core.windows.net/testcontainer/new-file.txt",
    "sequencer": "00000000000004420000000000028963",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobcreated-event-data-lake-storage-gen2"></a>Событие Microsoft. Storage. BlobCreated (Data Lake Storage 2-го поколения)

Если учетная запись хранения BLOB-объектов имеет иерархическое пространство имен, то данные выглядят примерно так, как в предыдущем примере, за исключением следующих изменений:

* `dataVersion`Для ключа задается значение `2` .

* `data.api`Для ключа задается строка `CreateFile` или `FlushWithClose` .

* `contentOffset`Ключ включен в набор данных.

> [!NOTE]
> Если приложения используют `PutBlockList` операцию для отправки нового большого двоичного объекта в учетную запись, данные не будут содержать эти изменения.

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/new-file.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "CreateFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "\"0x8D4BCC2E4835CD0\"",
    "contentType": "text/plain",
    "contentLength": 0,
    "contentOffset": 0,
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/new-file.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "2",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobdeleted-event"></a>Событие Microsoft. Storage. BlobDeleted

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/testcontainer/blobs/file-to-delete.txt",
  "eventType": "Microsoft.Storage.BlobDeleted",
  "eventTime": "2017-11-07T20:09:22.5674003Z",
  "id": "4c2359fe-001e-00ba-0e04-58586806d298",
  "data": {
    "api": "DeleteBlob",
    "requestId": "4c2359fe-001e-00ba-0e04-585868000000",
    "contentType": "text/plain",
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.blob.core.windows.net/testcontainer/file-to-delete.txt",
    "sequencer": "0000000000000281000000000002F5CA",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobdeleted-event-data-lake-storage-gen2"></a>Событие Microsoft. Storage. BlobDeleted (Data Lake Storage 2-го поколения)

Если учетная запись хранения BLOB-объектов имеет иерархическое пространство имен, то данные выглядят примерно так, как в предыдущем примере, за исключением следующих изменений:

* `dataVersion`Для ключа задается значение `2` .

* `data.api`Для ключа задается строка `DeleteFile` .

* `url`Ключ содержит путь `dfs.core.windows.net` .

> [!NOTE]
> Если приложения используют `DeleteBlob` операцию для удаления большого двоичного объекта из учетной записи, данные не будут содержать эти изменения.

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/file-to-delete.txt",
  "eventType": "Microsoft.Storage.BlobDeleted",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
    "data": {
    "api": "DeleteFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "contentType": "text/plain",
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/file-to-delete.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "2",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobrenamed-event"></a>Событие Microsoft. Storage. Блобренамед

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-renamed-file.txt",
  "eventType": "Microsoft.Storage.BlobRenamed",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "RenameFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "destinationUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-renamed-file.txt",
    "sourceUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-original-file.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstoragedirectorycreated-event"></a>Событие Microsoft. Storage. Директорикреатед

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-new-directory",
  "eventType": "Microsoft.Storage.DirectoryCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "CreateDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-new-directory",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstoragedirectoryrenamed-event"></a>Событие Microsoft. Storage. Директориренамед

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-renamed-directory",
  "eventType": "Microsoft.Storage.DirectoryRenamed",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "RenameDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "destinationUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-renamed-directory",
    "sourceUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-original-directory",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstoragedirectorydeleted-event"></a>Событие Microsoft. Storage. Директориделетед

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/directory-to-delete",
  "eventType": "Microsoft.Storage.DirectoryDeleted",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "DeleteDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/directory-to-delete",
    "recursive": "true", 
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

### <a name="microsoftstorageblobcreated-event"></a>Событие Microsoft. Storage. BlobCreated

```json
[{
  "source": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/test-container/blobs/new-file.txt",
  "type": "Microsoft.Storage.BlobCreated",
  "time": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "\"0x8D4BCC2E4835CD0\"",
    "contentType": "text/plain",
    "contentLength": 524288,
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.blob.core.windows.net/testcontainer/new-file.txt",
    "sequencer": "00000000000004420000000000028963",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "specversion": "1.0"
}]
```

### <a name="microsoftstorageblobcreated-event-data-lake-storage-gen2"></a>Событие Microsoft. Storage. BlobCreated (Data Lake Storage 2-го поколения)

Если учетная запись хранения BLOB-объектов имеет иерархическое пространство имен, то данные выглядят примерно так, как в предыдущем примере, за исключением следующих изменений:

* `data.api`Для ключа задается строка `CreateFile` или `FlushWithClose` .
* `contentOffset`Ключ включен в набор данных.

> [!NOTE]
> Если приложения используют `PutBlockList` операцию для отправки нового большого двоичного объекта в учетную запись, данные не будут содержать эти изменения.

```json
[{
  "source": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/new-file.txt",
  "type": "Microsoft.Storage.BlobCreated",
  "time": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "CreateFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "\"0x8D4BCC2E4835CD0\"",
    "contentType": "text/plain",
    "contentLength": 0,
    "contentOffset": 0,
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/new-file.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "specversion": "1.0"
}]
```

### <a name="microsoftstorageblobdeleted-event"></a>Событие Microsoft. Storage. BlobDeleted

```json
[{
  "source": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/testcontainer/blobs/file-to-delete.txt",
  "type": "Microsoft.Storage.BlobDeleted",
  "time": "2017-11-07T20:09:22.5674003Z",
  "id": "4c2359fe-001e-00ba-0e04-58586806d298",
  "data": {
    "api": "DeleteBlob",
    "requestId": "4c2359fe-001e-00ba-0e04-585868000000",
    "contentType": "text/plain",
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.blob.core.windows.net/testcontainer/file-to-delete.txt",
    "sequencer": "0000000000000281000000000002F5CA",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "specversion": "1.0"
}]
```

### <a name="microsoftstorageblobdeleted-event-data-lake-storage-gen2"></a>Событие Microsoft. Storage. BlobDeleted (Data Lake Storage 2-го поколения)

Если учетная запись хранения BLOB-объектов имеет иерархическое пространство имен, то данные выглядят примерно так, как в предыдущем примере, за исключением следующих изменений:


* `data.api`Для ключа задается строка `DeleteFile` .
* `url`Ключ содержит путь `dfs.core.windows.net` .

> [!NOTE]
> Если приложения используют `DeleteBlob` операцию для удаления большого двоичного объекта из учетной записи, данные не будут содержать эти изменения.

```json
[{
  "source": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/file-to-delete.txt",
  "type": "Microsoft.Storage.BlobDeleted",
  "time": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
    "data": {
    "api": "DeleteFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "contentType": "text/plain",
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/file-to-delete.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "specversion": "1.0"
}]
```

### <a name="microsoftstorageblobrenamed-event"></a>Событие Microsoft. Storage. Блобренамед

```json
[{
  "source": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-renamed-file.txt",
  "type": "Microsoft.Storage.BlobRenamed",
  "time": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "RenameFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "destinationUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-renamed-file.txt",
    "sourceUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-original-file.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "specversion": "1.0"
}]
```

### <a name="microsoftstoragedirectorycreated-event"></a>Событие Microsoft. Storage. Директорикреатед

```json
[{
  "source": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-new-directory",
  "type": "Microsoft.Storage.DirectoryCreated",
  "time": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "CreateDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-new-directory",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "specversion": "1.0"
}]
```

### <a name="microsoftstoragedirectoryrenamed-event"></a>Событие Microsoft. Storage. Директориренамед

```json
[{
  "source": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-renamed-directory",
  "type": "Microsoft.Storage.DirectoryRenamed",
  "time": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "RenameDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "destinationUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-renamed-directory",
    "sourceUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-original-directory",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "specversion": "1.0"
}]
```

### <a name="microsoftstoragedirectorydeleted-event"></a>Событие Microsoft. Storage. Директориделетед

```json
[{
  "source": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/directory-to-delete",
  "type": "Microsoft.Storage.DirectoryDeleted",
  "time": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "DeleteDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/directory-to-delete",
    "recursive": "true", 
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "specversion": "1.0"
}]
```

---


## <a name="event-properties"></a>Свойства события

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Событие содержит следующие высокоуровневые данные:

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `topic` | строка | Полный путь к ресурсу для источника событий. Это поле защищено от записи. Это значение предоставляет служба "Сетка событий". |
| `subject` | строка | Определенный издателем путь к субъекту событий. |
| `eventType` | строка | Один из зарегистрированных типов событий для этого источника событий. |
| `eventTime` | строка | Время создания события с учетом времени поставщика в формате UTC. |
| `id` | строка | Уникальный идентификатор события. |
| `data` | объект | Данные события хранилища BLOB-объектов. |
| `dataVersion` | строка | Версия схемы для объекта данных. Версию схемы определяет издатель. |
| `metadataVersion` | строка | Версия схемы для метаданных события. Служба "Сетка событий" определяет схему свойств верхнего уровня. Это значение предоставляет служба "Сетка событий". |

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Событие содержит следующие высокоуровневые данные:

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `source` | строка | Полный путь к ресурсу для источника событий. Это поле защищено от записи. Это значение предоставляет служба "Сетка событий". |
| `subject` | строка | Определенный издателем путь к субъекту событий. |
| `type` | строка | Один из зарегистрированных типов событий для этого источника событий. |
| `time` | строка | Время создания события с учетом времени поставщика в формате UTC. |
| `id` | строка | Уникальный идентификатор события. |
| `data` | объект | Данные события хранилища BLOB-объектов. |
| `specversion` | строка | Версия спецификации схемы Клаудевентс. |

---

Объект данных имеет следующие свойства:

| Свойство | Type | Описание |
| -------- | ---- | ----------- |
| `api` | строка | Операция, вызвавшая событие. |
| `clientRequestId` | строка | предоставленный клиентом идентификатор запроса для операции API хранилища. Этот идентификатор можно использовать для сопоставления с журналами диагностики службы хранилища Azure с помощью поля "Client-Request-ID" в журналах и может быть предоставлено в запросах клиента с помощью заголовка "x-MS-Client-Request-ID". Ознакомьтесь со статьей [Storage Analytics Log Format](/rest/api/storageservices/storage-analytics-log-format) (Формат журналов Аналитики Службы хранилища). |
| `requestId` | строка | Идентификатор запроса, формируемый службой для операции API хранилища. Может использоваться для корреляции журналов диагностики службы хранилища Azure с помощью поля request-id-header в журналах. Возвращается при инициации вызова API в заголовке x-ms-request-id. Ознакомьтесь со статьей [Storage Analytics Log Format](/rest/api/storageservices/storage-analytics-log-format) (Формат журналов Аналитики Службы хранилища). |
| `eTag` | строка | Значение, которое можно использовать для условного выполнения операций. |
| `contentType` | строка | Тип содержимого, указанный для BLOB-объекта. |
| `contentLength` | Целое число | Размер большого двоичного объекта в байтах. |
| `blobType` | строка | Тип большого двоичного объекта. Допустимые значения: BlockBlob или PageBlob. |
| `contentOffset` | Число | Смещение в байтах операции записи, выполненной в момент, когда приложение, запускающее событие, выполнило запись в файл. <br>Отображается только для событий, запускаемых в учетных записях хранения BLOB-объектов с иерархическим пространством имен.|
| `destinationUrl` |строка | URL-адрес файла, который будет существовать после завершения операции. Например, если файл переименован, `destinationUrl` свойство содержит URL-адрес нового имени файла. <br>Отображается только для событий, запускаемых в учетных записях хранения BLOB-объектов с иерархическим пространством имен.|
| `sourceUrl` |строка | URL-адрес файла, который существует до завершения операции. Например, при переименовании файла `sourceUrl` содержит URL-адрес исходного имени файла перед операцией переименования. <br>Отображается только для событий, запускаемых в учетных записях хранения BLOB-объектов с иерархическим пространством имен. |
| `url` | строка | Путь к BLOB-объекту. <br>Если клиент использует REST API больших двоичных объектов, URL-адрес имеет следующую структуру: `<storage-account-name>.blob.core.windows.net\<container-name>\<file-name>` . <br>Если клиент использует REST API Data Lake Storage, URL-адрес имеет следующую структуру: `<storage-account-name>.dfs.core.windows.net/<file-system-name>/<file-name>` . |
| `recursive` | строка | `True` для выполнения операции со всеми дочерними каталогами; в противном случае — значение `False` . <br>Отображается только для событий, запускаемых в учетных записях хранения BLOB-объектов с иерархическим пространством имен. |
| `sequencer` | строка | Значение непрозрачной строки, представляющее логическую последовательность событий для любого отдельного имени большого двоичного объекта.  Пользователи могут использовать стандартное сравнение строк для понимания относительной последовательности двух событий в одном имени большого двоичного объекта. |
| `storageDiagnostics` | object | Диагностические данные, которые иногда включаются службой хранилища Azure. Если они присутствуют, то должны игнорироваться потребителями события. |

## <a name="tutorials-and-how-tos"></a>Руководства и инструкции
|Заголовок  |Описание  |
|---------|---------|
| [Перенаправление событий хранилища BLOB-объектов в пользовательскую конечную веб-точку с помощью Azure CLI](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Содержит сведения об отправке событий хранилища BLOB-объектов в веб-перехватчик с помощью Azure CLI. |
| [Перенаправление событий хранилища BLOB-объектов в пользовательскую конечную веб-точку с помощью PowerShell](../storage/blobs/storage-blob-event-quickstart-powershell.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Содержит сведения об отправке событий хранилища BLOB-объектов в веб-перехватчик с помощью Azure PowerShell. |
| [Создание и перенаправление событий хранилища BLOB-объектов с помощью службы "Сетка событий Azure" и портала Azure](blob-event-quickstart-portal.md) | Содержит сведения об отправке событий хранилища BLOB-объектов в веб-перехватчик с помощью портала. |
| [Создание подписки на события, связанные с учетной записью хранения больших двоичных объектов, с использованием Azure CLI](./scripts/event-grid-cli-blob.md) | Пример сценария, позволяющий подписаться на события, связанные с учетной записью хранения больших двоичных объектов. Он отправляет событие в веб-перехватчик. |
| [Создание подписки на события, связанные с учетной записью хранения больших двоичных объектов, с помощью PowerShell](./scripts/event-grid-powershell-blob.md) | Пример сценария, позволяющий подписаться на события, связанные с учетной записью хранения больших двоичных объектов. Он отправляет событие в веб-перехватчик. |
| [Шаблон Resource Manager для создания хранилища BLOB-объектов и подписки](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid-subscription-and-storage) | Развертывает учетную запись службы хранилища больших двоичных объектов Azure и создает подписку на события для этой учетной записи хранения. Он отправляет события в веб-перехватчик. |
| [Реагирование на события хранилища BLOB-объектов](../storage/blobs/storage-blob-event-overview.md) | Общие сведения об интеграции хранилища BLOB-объектов со службой "Сетка событий". |

## <a name="next-steps"></a>Дальнейшие действия

* См. общие сведения о [службе "Сетка событий Azure"](overview.md).
* Дополнительные сведения о создании подписки на Сетку событий Azure см. в статье [Схема подписки для службы "Сетка событий"](subscription-creation-schema.md).
* Основные сведения о работе с событиями хранилища BLOB-объектов см. в статье [Перенаправление событий хранилища BLOB-объектов в пользовательскую конечную веб-точку (предварительная версия)](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json). 
