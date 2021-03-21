---
title: Сохранение метаданных и ACL с помощью действия копирования в фабрике данных Azure
description: Узнайте, как сохранить метаданные и списки ACL во время копирования с помощью действия копирования в фабрике данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 09/23/2020
ms.author: jingwang
ms.openlocfilehash: 396a598d143e85687f9dfbf765b3c18736627e41
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100387722"
---
#  <a name="preserve-metadata-and-acls-using-copy-activity-in-azure-data-factory"></a>Сохранение метаданных и ACL с помощью действия копирования в фабрике данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

При использовании действия копирования фабрики данных Azure для копирования данных из источника в приемник в следующих сценариях можно также сохранить метаданные и списки ACL.

## <a name="preserve-metadata-for-lake-migration"></a><a name="preserve-metadata"></a> Сохранение метаданных для Lake Migration

При переносе данных из одного Data Lake в другой, включая [Amazon S3](connector-amazon-simple-storage-service.md), [большой двоичный объект Azure](connector-azure-blob-storage.md), [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md)и [хранилище файлов Azure](connector-azure-file-storage.md), можно сохранить метаданные файла вместе с данными.

Действие копирования поддерживает сохранение следующих атрибутов во время копирования данных:

- **Все указанные клиентом метаданные** 
- И следующие **пять встроенных системных свойств хранилища данных**: `contentType` , `contentLanguage` (за исключением Amazon S3),, `contentEncoding` `contentDisposition` `cacheControl` .

**Обработку различий в метаданных:** Amazon S3 и служба хранилища Azure позволяют использовать разные наборы символов в ключах указанных пользователем метаданных. Если вы решили сохранить метаданные с помощью действия копирования, ADF автоматически заменит недопустимые символы символом "_".

При копировании файлов "как есть" из Amazon S3/Azure Data Lake Storage 2-го поколения/BLOB-объекта Azure или хранилища файлов Azure в Azure Data Lake Storage 2-го поколения/BLOB-объект Azure или хранилище файлов Azure с двоичным форматом можно найти параметр " **сохранить** " на вкладке "Параметры **действия копирования**  >   " для создания действий или на странице " **Параметры** " в копирование данных средстве.

![Сохранить метаданные действия копирования](./media/copy-activity-preserve-metadata/copy-activity-preserve-metadata.png)

Ниже приведен пример конфигурации JSON действия копирования (см. раздел `preserve` ): 

```json
"activities":[
    {
        "name": "CopyAndPreserveMetadata",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "BinarySource",
                "storeSettings": {
                    "type": "AmazonS3ReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "BinarySink",
                "storeSettings": {
                    "type": "AzureBlobFSWriteSettings"
                }
            },
            "preserve": [
                "Attributes"
            ]
        },
        "inputs": [
            {
                "referenceName": "<Binary dataset Amazon S3/Azure Blob/ADLS Gen2 source>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Binary dataset for Azure Blob/ADLS Gen2 sink>",
                "type": "DatasetReference"
            }
        ]
    }
]
```

## <a name="preserve-acls-from-data-lake-storage-gen1gen2-to-gen2"></a><a name="preserve-acls"></a> Сохранение списков управления доступом из Data Lake Storage 1-го поколения или Gen2 в Gen2

При обновлении с Azure Data Lake Storage 1-го поколения до Gen2 или копировании данных между ADLS 2-го поколения можно сохранить списки управления доступом (ACL) POSIX вместе с файлами данных. Дополнительные сведения об управлении доступом см. в разделе [Контроль доступа в Azure Data Lake Storage 1-го поколения](../data-lake-store/data-lake-store-access-control.md) и [управление доступом в Azure Data Lake Storage 2-го поколения](../storage/blobs/data-lake-storage-access-control.md).

Действие копирования поддерживает сохранение следующих типов списков ACL во время копирования данных. Можно выбрать один или несколько типов:

- **ACL**: копирование и сохранение списков управления доступом POSIX для файлов и каталогов. Он копирует все существующие списки ACL из источника в приемник. 
- **Владелец**: Скопируйте и сохраните пользователя-владельца файлов и каталогов. Требуется доступ суперпользователя к Data Lake Storage 2-го поколения приемника.
- **Группа**: копирование и сохранение группы владельцев файлов и каталогов. Необходим доступ суперпользователя к Data Lake Storage 2-го поколения приемника или пользователю-владельцу (если владелец также является членом Целевой группы).

Если вы указываете копировать из папки, фабрика данных реплицирует списки управления доступом для указанной папки, а также файлы и каталоги, если для параметра задано значение `recursive` true. Если указать копирование из одного файла, то списки ACL для этого файла будут скопированы.

>[!NOTE]
>При использовании ADF для сохранения списков ACL из Data Lake Storage 1-го поколения или Gen2 в Gen2 существующие списки ACL в приемнике Gen2's соответствующие папки и файлы будут перезаписаны.

>[!IMPORTANT]
>Если вы решили сохранить списки управления доступом, убедитесь, что вы предоставляли достаточно высокий уровень разрешений для работы фабрики данных с учетной записью Data Lake Storage 2-го поколения приемника. Например, используйте проверку подлинности с помощью ключа учетной записи или назначьте роль владельца данных BLOB-объекта хранилища субъекту-службе или управляемому удостоверению.

При настройке источника как Data Lake Storage 1-го поколения/Gen2 с двоичным форматом или параметром двоичного копирования, а также приемника в качестве Data Lake Storage 2-го поколения с двоичным форматом или параметром копирования в двоичном формате можно найти параметр **сохранить** на странице **Параметры** в копирование данных средстве или на вкладке Параметры **действия копирования**  >   для создания действий.

![Data Lake Storage 1-го поколения или Gen2 в Gen2 сохранить список ACL](./media/connector-azure-data-lake-storage/adls-gen2-preserve-acl.png)

Ниже приведен пример конфигурации JSON действия копирования (см. раздел `preserve` ): 

```json
"activities":[
    {
        "name": "CopyAndPreserveACLs",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "BinarySource",
                "storeSettings": {
                    "type": "AzureDataLakeStoreReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "BinarySink",
                "storeSettings": {
                    "type": "AzureBlobFSWriteSettings"
                }
            },
            "preserve": [
                "ACL",
                "Owner",
                "Group"
            ]
        },
        "inputs": [
            {
                "referenceName": "<Binary dataset name for Azure Data Lake Storage Gen1/Gen2 source>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Binary dataset name for Azure Data Lake Storage Gen2 sink>",
                "type": "DatasetReference"
            }
        ]
    }
]
```

## <a name="next-steps"></a>Дальнейшие действия

См. другие статьи о действиях копирования:

- [Действие копирования в фабрике данных Azure](copy-activity-overview.md)
- [Руководство по настройке производительности действия копирования](copy-activity-performance.md)
