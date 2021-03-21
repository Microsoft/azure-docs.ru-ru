---
title: Действие "получить метаданные" в фабрике данных Azure
description: Узнайте, как использовать действие получение метаданных в конвейере фабрики данных.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 02/25/2021
ms.author: jingwang
ms.openlocfilehash: bd8fc3383d6d9a0afb7733cb94643623e6879d23
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102178547"
---
# <a name="get-metadata-activity-in-azure-data-factory"></a>Действие "получить метаданные" в фабрике данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Действие получить метаданные можно использовать для получения метаданных любых данных в фабрике данных Azure. Выходные данные действия получить метаданные в условных выражениях можно использовать для выполнения проверки или использования метаданных в последующих действиях.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Действие получить метаданные принимает в качестве входного аргумента набор данных и возвращает сведения о метаданных в виде выходных данных. В настоящее время поддерживаются следующие соединители и соответствующие загружаемые метаданные. Максимальный размер возвращаемых метаданных — **4 МБ**.

### <a name="supported-connectors"></a>Поддерживаемые соединители

**Хранилище файлов**

| Соединитель и метаданные | itemName<br>(файл или папка) | itemType<br>(файл или папка) | size<br>(файл) | created<br>(файл или папка) | lastModified<sup>1</sup><br>(файл или папка) |childItems<br>(папка) |contentMD5<br>(файл) | Структура<sup>2</sup><br/>(файл) | число столбцов<sup>2</sup><br>(файл) | Существует<sup>3</sup><br>(файл или папка) |
|:--- |:--- |:--- |:--- |:--- |:--- |:--- |:--- |:--- |:--- |:--- |
| [Amazon S3](connector-amazon-simple-storage-service.md) | √/√ | √/√ | √ | x/x | √/√ | √ | x | √ | √ | √/√ |
| [Google Cloud Storage](connector-google-cloud-storage.md) | √/√ | √/√ | √ | x/x | √/√ | √ | x | √ | √ | √/√ |
| [Хранилище BLOB-объектов Azure](connector-azure-blob-storage.md) | √/√ | √/√ | √ | x/x | √/√ | √ | √ | √ | √ | √/√ |
| [Azure Data Lake Storage 1-го поколения](connector-azure-data-lake-store.md) | √/√ | √/√ | √ | x/x | √/√ | √ | x | √ | √ | √/√ |
| [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md) | √/√ | √/√ | √ | x/x | √/√ | √ | √ | √ | √ | √/√ |
| [Файлы Azure](connector-azure-file-storage.md) | √/√ | √/√ | √ | √/√ | √/√ | √ | x | √ | √ | √/√ |
| [Файловая система](connector-file-system.md) | √/√ | √/√ | √ | √/√ | √/√ | √ | x | √ | √ | √/√ |
| [SFTP](connector-sftp.md) | √/√ | √/√ | √ | x/x | √/√ | √ | x | √ | √ | √/√ |
| [FTP](connector-ftp.md) | √/√ | √/√ | √ | x/x | x/x | √ | x | √ | √ | √/√ |

<sup>1</sup> метаданные `lastModified` :
- Для Amazon S3 и Google Cloud Storage `lastModified` применяется к контейнеру и ключу, но не к виртуальной папке и `exists` применяется к контейнеру и ключу, но не к префиксу или виртуальной папке. 
- Для хранилища BLOB-объектов Azure `lastModified` применяется к контейнеру и большому двоичному объекту, но не к виртуальной папке.

<sup>2</sup> метаданные `structure` и `columnCount` не поддерживаются при получении метаданных из двоичных, JSON или XML-файлов.

<sup>3</sup> метаданные `exists` : для Amazon S3 и Google Cloud Storage `exists` применяются к контейнеру и ключу, но не к префиксу или виртуальной папке.

Следует отметить следующее.

- При использовании действия получить метаданные для папки убедитесь, что у вас есть разрешение LIST/EXECUTE для данной папки.
- Фильтр с подстановочными знаками для папок и файлов не поддерживается для действия "получение метаданных".
- `modifiedDatetimeStart` и `modifiedDatetimeEnd` фильтры, заданные для соединителя:

    - Эти два свойства используются для фильтрации дочерних элементов при получении метаданных из папки. Он не применяется при получении метаданных из файла.
    - При использовании такого фильтра в `childItems` выходных данных включаются только те файлы, которые были изменены в указанном диапазоне, но не являются папками.
    - Чтобы применить такой фильтр, действие with Metadata будет перечислять все файлы в указанной папке и проверять время изменения. Не следует указывать на папку с большим количеством файлов, даже если ожидаемое число файлов мало. 

**Реляционная база данных**

| Соединитель и метаданные | structure | columnCount | exists (существует) |
|:--- |:--- |:--- |:--- |
| [База данных SQL Azure](connector-azure-sql-database.md) | √ | √ | √ |
| [Управляемый экземпляр SQL Azure](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md) | √ | √ | √ |
| [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md) | √ | √ | √ |
| [SQL Server](connector-sql-server.md) | √ | √ | √ |

### <a name="metadata-options"></a>Параметры метаданных

Для получения соответствующих сведений можно указать следующие типы метаданных в списке полей действие получения метаданных.

| Тип метаданных | Описание |
|:--- |:--- |
| itemName | Имя файла или папки. |
| itemType | Тип файла или папки. Возвращаемое значение — `File` или `Folder` . |
| size | Размер файла в байтах. Применимо только к файлам. |
| created | Дата и время создания файла или папки. |
| lastModified | Дата и время последнего изменения файла или папки. |
| childItems | Список вложенных папок и файлов в указанной папке. Применимо только к папкам. Возвращаемое значение — это список имен и типов для каждого дочернего элемента. |
| contentMD5 | Сверка MD5 файла. Применимо только к файлам. |
| structure | Структура данных таблицы файла или реляционной базы данных. Возвращаемое значение — это список имен столбцов и типов столбцов. |
| columnCount | Число столбцов в файле или реляционной таблице. |
| exists (существует)| Существует ли файл, папка или таблица. Если `exists` в списке полей получить метаданные указано значение, действие не будет выполнено, даже если файл, папка или таблица не существуют. Вместо этого `exists: false` в выходных данных возвращается. |

> [!TIP]
> Если необходимо проверить, существует ли файл, папка или таблица, укажите `exists` в списке полей действие получить метаданные. Затем можно проверить `exists: true/false` результат в выходных данных действия. Если `exists` не указан в списке полей, действие получение метаданных завершится ошибкой, если объект не найден.

> [!NOTE]
> При получении метаданных из файлов и настройке `modifiedDatetimeStart` или, в `modifiedDatetimeEnd` `childItems` выходных данных включаются только файлы в указанном пути, которые имеют время последнего изменения в указанном диапазоне. Элементы во вложенных папках не включаются.

> [!NOTE]
> Чтобы в списке полей **структуры** обеспечить реальную структуру данных для наборов данных с разделителями текста и форматов Excel, необходимо включить `First Row as Header` свойство, которое поддерживается только для этих источников.

## <a name="syntax"></a>Синтаксис

**Действие "получить метаданные"**

```json
{
    "name":"MyActivity",
    "type":"GetMetadata",
    "dependsOn":[

    ],
    "policy":{
        "timeout":"7.00:00:00",
        "retry":0,
        "retryIntervalInSeconds":30,
        "secureOutput":false,
        "secureInput":false
    },
    "userProperties":[

    ],
    "typeProperties":{
        "dataset":{
            "referenceName":"MyDataset",
            "type":"DatasetReference"
        },
        "fieldList":[
            "size",
            "lastModified",
            "structure"
        ],
        "storeSettings":{
            "type":"AzureBlobStorageReadSettings"
        },
        "formatSettings":{
            "type":"JsonReadSettings"
        }
    }
}
```

**Набор данных**

```json
{
    "name":"MyDataset",
    "properties":{
        "linkedServiceName":{
            "referenceName":"AzureStorageLinkedService",
            "type":"LinkedServiceReference"
        },
        "annotations":[

        ],
        "type":"Json",
        "typeProperties":{
            "location":{
                "type":"AzureBlobStorageLocation",
                "fileName":"file.json",
                "folderPath":"folder",
                "container":"container"
            }
        }
    }
}
```

## <a name="type-properties"></a>Свойства типа

В настоящее время действие получить метаданные может возвращать следующие типы сведений о метаданных:

Свойство. | Описание | Обязательно
-------- | ----------- | --------
fieldList | Типы необходимых сведений о метаданных. Дополнительные сведения о поддерживаемых метаданных см. в разделе [Параметры метаданных](#metadata-options) этой статьи. | Да 
набор данных | Эталонный набор данных, метаданные которого будут извлечены действием "получение метаданных". Сведения о поддерживаемых соединителях см. в разделе [возможности](#supported-capabilities) . Сведения о синтаксисе набора данных см. в разделах, посвященных конкретным соединителям. | Да
форматсеттингс | Применяется при использовании набора данных типа Format. | Нет
сторесеттингс | Применяется при использовании набора данных типа Format. | Нет

## <a name="sample-output"></a>Пример выходных данных

Результаты получения метаданных отображаются в выходных данных действия. Ниже приведены два примера, в которых показаны расширенные параметры метаданных. Чтобы использовать результаты в последующем действии, используйте следующий шаблон: `@{activity('MyGetMetadataActivity').output.itemName}` .

### <a name="get-a-files-metadata"></a>Получение метаданных файла

```json
{
  "exists": true,
  "itemName": "test.csv",
  "itemType": "File",
  "size": 104857600,
  "lastModified": "2017-02-23T06:17:09Z",
  "created": "2017-02-23T06:17:09Z",
  "contentMD5": "cMauY+Kz5zDm3eWa9VpoyQ==",
  "structure": [
    {
        "name": "id",
        "type": "Int64"
    },
    {
        "name": "name",
        "type": "String"
    }
  ],
  "columnCount": 2
}
```

### <a name="get-a-folders-metadata"></a>Получение метаданных папки

```json
{
  "exists": true,
  "itemName": "testFolder",
  "itemType": "Folder",
  "lastModified": "2017-02-23T06:17:09Z",
  "created": "2017-02-23T06:17:09Z",
  "childItems": [
    {
      "name": "test.avro",
      "type": "File"
    },
    {
      "name": "folder hello",
      "type": "Folder"
    }
  ]
}
```

## <a name="next-steps"></a>Дальнейшие действия
Сведения о других действиях потока управления, поддерживаемых фабрикой данных:

- [Действие выполнения конвейера в фабрике данных Azure](control-flow-execute-pipeline-activity.md)
- [Действие ForEach](control-flow-for-each-activity.md)
- [Действие поиска](control-flow-lookup-activity.md)
- [Веб-действие](control-flow-web-activity.md)
