---
title: Перемещение данных из Amazon Simple Storage Service с помощью фабрики данных
description: Вы можете узнать, как переместить данные из Amazon Simple Storage Service (S3) с помощью фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 6d254a535b1db53478772b481bd029a8c4db6f3c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100361352"
---
# <a name="move-data-from-amazon-simple-storage-service-by-using-azure-data-factory"></a>Перемещение данных из Amazon Simple Storage Service с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-amazon-simple-storage-service-connector.md)
> * [Версия 2 (текущая)](../connector-amazon-simple-storage-service.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [соединителе Amazon S3 в службе "Фабрика данных Azure" версии 2](../connector-amazon-simple-storage-service.md).

В этой статье описано, как с помощью действия копирования в фабрике данных Azure перемещать данные из службы хранения Amazon Simple Storage Service (S3). Это продолжение статьи о [действиях перемещения данных](data-factory-data-movement-activities.md), в которой приведены общие сведения о перемещении данных с помощью действия копирования.

Данные из Amazon S3 можно скопировать в любое хранилище данных, поддерживаемое в качестве приемника. Список хранилищ данных, поддерживаемых действием копирования в качестве приемников, см. в таблице [Поддерживаемые хранилища данных](data-factory-data-movement-activities.md#supported-data-stores-and-formats) . Сейчас фабрика данных поддерживает только перемещение данных из Amazon S3 в другие хранилища данных, но не наоборот.

## <a name="required-permissions"></a>Необходимые разрешения
Чтобы скопировать данные из Amazon S3, убедитесь, что вы предоставили следующие разрешения:

* `s3:GetObject` и `s3:GetObjectVersion` для операций с объектами Amazon S3.
* `s3:ListBucket` для операций в контейнере Amazon S3. Если вы используете мастер копирования фабрики данных, также требуется разрешение `s3:ListAllMyBuckets`.

Полный список разрешений Amazon S3 см. в статье, посвященной [назначению разрешений в политике](https://docs.aws.amazon.com/AmazonS3/latest/dev/using-with-s3-actions.html).

## <a name="getting-started"></a>Начало работы
Вы можете создать конвейер с действием копирования, которое перемещает данные из источника Amazon S3 с помощью различных средств или API-интерфейсов.

Самый простой способ создать конвейер — использовать **Мастер копирования**. Краткое пошаговое руководство представлено в разделе [Руководство. Создание конвейера с действием копирования с помощью мастера копирования фабрики данных](data-factory-copy-data-wizard-tutorial.md).

Для создания конвейера можно также использовать следующие средства: **Visual Studio**, **Azure PowerShell**, **Azure Resource Manager шаблон**, **API .NET** и **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в [руководстве по действию копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Независимо от используемого средства или API-интерфейса, для создания конвейера, который перемещает данные из источника данных в приемник, выполняются следующие шаги:

1. Создайте **связанные службы** , чтобы связать входные и выходные данные хранилища с фабрикой данных.
2. Создайте **наборы данных**, которые представляют входные и выходные данные для операции копирования.
3. Создайте **конвейер** с действием копирования, который принимает входной набор данных и возвращает выходной набор данных.

Если вы используете мастер, то он автоматически создает определения JSON для сущностей фабрики данных (связанных служб, наборов данных и конвейера). При использовании средств или интерфейсов API (за исключением API .NET) вы самостоятельно определяете эти сущности фабрики данных в формате JSON. Пример определений JSON для сущностей фабрики данных, которые используются для копирования данных из хранилища данных Amazon S3, вы найдете в разделе [Пример JSON. Копирование данных из Amazon S3 в хранилище BLOB-объектов Azure](#json-example-copy-data-from-amazon-s3-to-azure-blob-storage) этой статьи.

> [!NOTE]
> Подробные сведения о поддерживаемых форматах файлов и сжатия для действия копирования см. в разделе [Форматы файлов и сжатия данных, поддерживаемые фабрикой данных Azure](data-factory-supported-file-and-compression-formats.md).

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, характерных для Amazon S3.

## <a name="linked-service-properties"></a>Свойства связанной службы
Связанная служба привязывает хранилище данных к фабрике данных. Для связи хранилища данных Amazon S3 с фабрикой данных следует создать связанную службу типа **AwsAccessKey**. В таблице ниже приведено описание элементов JSON, которые относятся к связанной службе Amazon S3 (AwsAccessKey).

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| accessKeyID |Идентификатор секретного ключа доступа. |строка |Да |
| secretAccessKey |Сам секретный ключ доступа. |Зашифрованная строка секрета |Да |

>[!NOTE]
>Для этого соединителя требуются ключи доступа к учетной записи IAM для копирования данных из Amazon S3. [Временные учетные данные безопасности](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html) не поддерживаются.
>

Например:

```json
{
    "name": "AmazonS3LinkedService",
    "properties": {
        "type": "AwsAccessKey",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": "<secret access key>"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных
Чтобы указать набор данных, представляющий входные данные в хранилище BLOB-объектов Azure, присвойте свойству типа набора данных значение **AmazonS3**. Для свойства **linkedServiceName** набора данных задайте имя связанной службы Amazon S3. Полный список разделов и свойств, доступных для определения наборов данных, см. в разделе [Наборы данных в фабрике данных Azure](data-factory-create-datasets.md). 

Разделы structure, availability и policy одинаковы для всех типов наборов данных (таких как базы данных SQL, большие двоичные объекты Azure и таблицы Azure). Раздел **typeProperties** отличается для каждого типа набора данных и предоставляет сведения о расположении данных в хранилище данных. Раздел **typeProperties** набора данных типа **AmazonS3** (который включает в себя набор данных Amazon S3) содержит следующие свойства:

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| bucketName |Имя контейнера S3. |Строка |Да |
| ключ |Ключ объекта S3. |Строка |нет |
| prefix |Префикс для ключа объекта S3. Выбираются объекты, ключи которых начинаются с этого префикса. Применяется, только если ключ пустой. |Строка |Нет |
| version |Версия объекта S3, если включено управление версиями S3. |Строка |Нет |
| format | Поддерживаются следующие типы формата: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Свойству **type** в разделе format необходимо присвоить одно из этих значений. Дополнительные сведения см. в разделах о [текстовом формате](data-factory-supported-file-and-compression-formats.md#text-format), [формате JSON](data-factory-supported-file-and-compression-formats.md#json-format), [формате Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [формате Orc](data-factory-supported-file-and-compression-formats.md#orc-format) и [формате Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Если требуется скопировать файлы между файловыми хранилищами как есть (двоичное копирование), можно пропустить раздел форматирования в определениях входного и выходного наборов данных. | |Нет |
| compression | Укажите тип и уровень сжатия данных. Поддерживаемые типы: **GZip**, **Deflate**, **BZip2** и **ZipDeflate**. Поддерживаемые уровни: **Optimal** (Оптимальный) и **Fastest** (Самый быстрый). Узнайте больше о [форматах файлов и сжатия данных в фабрике данных Azure](data-factory-supported-file-and-compression-formats.md#compression-support). | |Нет |


> [!NOTE]
> **Свойства bucketName и key** указывают расположение объекта S3, где контейнер — это корневой контейнер для объектов S3, а ключ — полный путь к объекту S3.

### <a name="sample-dataset-with-prefix"></a>Пример набора данных с префиксом

```json
{
    "name": "dataset-s3",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": "link- testS3",
        "typeProperties": {
            "prefix": "testFolder/test",
            "bucketName": "testbucket",
            "format": {
                "type": "OrcFormat"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
### <a name="sample-dataset-with-version"></a>Пример набора данных (с версией)

```json
{
    "name": "dataset-s3",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": "link- testS3",
        "typeProperties": {
            "key": "testFolder/test.orc",
            "bucketName": "testbucket",
            "version": "XXXXXXXXXczm0CJajYkHf0_k6LhBmkcL",
            "format": {
                "type": "OrcFormat"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

### <a name="dynamic-paths-for-s3"></a>Динамические пути для S3
В предыдущем примере используются фиксированные значения для свойств **key** и **bucketName** в наборе данных Amazon S3.

```json
"key": "testFolder/test.orc",
"bucketName": "testbucket",
```

Можно настроить фабрику данных для динамического вычисления значений этих свойств во время выполнения с помощью системных переменных, например SliceStart.

```json
"key": "$$Text.Format('{0:MM}/{0:dd}/test.orc', SliceStart)"
"bucketName": "$$Text.Format('{0:yyyy}', SliceStart)"
```

То же самое можно сделать и для свойства **prefix** набора данных Amazon S3. Список поддерживаемых функций и переменных см. в статье [Функции и системные переменные фабрики данных](data-factory-functions-variables.md).

## <a name="copy-activity-properties"></a>Свойства действия копирования
Полный список разделов и свойств, доступных для определения действий, см. в разделе [Конвейеры и действия в фабрике данных Azure](data-factory-create-pipelines.md). Свойства (такие как имя, описание, входные и выходные таблицы, политики и т. д.) доступны для всех типов действий. Свойства, доступные в разделе **typeProperties** действия, зависят от конкретного типа действия. Для действия копирования свойства различаются в зависимости от типов источников и приемников. Когда источник действия копирования имеет тип **FileSystemSource** (который включает в себя Amazon S3), в разделе **typeProperties** доступно приведенное ниже свойство.

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| recursive |Указывает, следует ли отображать рекурсивный список объектов S3 в каталоге. |true/false |Нет |

## <a name="json-example-copy-data-from-amazon-s3-to-azure-blob-storage"></a>Пример JSON. Копирование данных из Amazon S3 в хранилище BLOB-объектов Azure
В этом примере показано, как скопировать данные из Amazon S3 в хранилище BLOB-объектов Azure. Тем не менее данные можно копировать непосредственно в [любой из поддерживаемых приемников](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Для этого применяется действие копирования в фабрике данных.

Пример содержит определения JSON для следующих сущностей фабрики данных. Эти определения можно использовать для создания конвейера для копирования данных из Amazon S3 в хранилище BLOB-объектов с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) или [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md).   

* Связанная служба типа [AwsAccessKey](#linked-service-properties).
* Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Входной [набор данных](data-factory-create-datasets.md) типа [AmazonS3](#dataset-properties).
* Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [FileSystemSource](#copy-activity-properties) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

В этом примере данные из Amazon S3 каждый час копируются в большой двоичный объект Azure. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

### <a name="amazon-s3-linked-service"></a>Связанная служба Amazon S3

```json
{
    "name": "AmazonS3LinkedService",
    "properties": {
        "type": "AwsAccessKey",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": "<secret access key>"
        }
    }
}
```

### <a name="azure-storage-linked-service"></a>Связанная служба хранения Azure

```json
{
  "name": "AzureStorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

### <a name="amazon-s3-input-dataset"></a>Входной набор данных Amazon S3

Если параметру **external** присвоить значение true, то фабрика данных воспримет этот набор данных как внешний. Присвойте значение true этому параметру входного набора данных, который не является результатом какого-либо действия в конвейере.

```json
    {
        "name": "AmazonS3InputDataset",
        "properties": {
            "type": "AmazonS3",
            "linkedServiceName": "AmazonS3LinkedService",
            "typeProperties": {
                "key": "testFolder/test.orc",
                "bucketName": "testbucket",
                "format": {
                    "type": "OrcFormat"
                }
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            },
            "external": true
        }
    }
```


### <a name="azure-blob-output-dataset"></a>Выходной набор данных BLOB-объекта Azure

Данные записываются в новый BLOB-объект каждый час (frequency: hour, interval: 1). Путь к папке BLOB-объекта вычисляется динамически на основе времени начала обрабатываемого среза. В пути к папке используются год, месяц, день и час времени начала.

```json
{
    "name": "AzureBlobOutputDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/fromamazons3/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            },
            "partitionedBy": [
                {
                    "name": "Year",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "yyyy"
                    }
                },
                {
                    "name": "Month",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "MM"
                    }
                },
                {
                    "name": "Day",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "dd"
                    }
                },
                {
                    "name": "Hour",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "HH"
                    }
                }
            ]
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```


### <a name="copy-activity-in-a-pipeline-with-an-amazon-s3-source-and-a-blob-sink"></a>Действие копирования в конвейере с Amazon S3 в качестве источника и BLOB-объектом в качестве приемника

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **Source** установлено значение **FileSystemSource**, а для типа **sink** — значение **BlobSink**.

```json
{
    "name": "CopyAmazonS3ToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "FileSystemSource",
                        "recursive": true
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "AmazonS3InputDataset"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutputDataSet"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "AmazonS3ToBlob"
            }
        ],
        "start": "2014-08-08T18:00:00Z",
        "end": "2014-08-08T19:00:00Z"
    }
}
```
> [!NOTE]
> Сведения о сопоставлении столбцов в наборе данных, используемом в качестве источника, со столбцами в приемнике см. в статье [Сопоставление столбцов набора данных в фабрике данных Azure](data-factory-map-columns.md).


## <a name="next-steps"></a>Дальнейшие действия
См. следующие статьи:

* Сведения о ключевых факторах, влияющих на производительность перемещения данных (действие копирования) в фабрике данных, и различных способах оптимизации этого процесса см. в [руководстве по настройке производительности действия копирования](data-factory-copy-activity-performance.md).

* Пошаговые инструкции по созданию конвейера с действием копирования см. в [руководстве по действию копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
