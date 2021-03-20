---
title: Перемещение данных из Amazon RedShift с помощью фабрики данных Azure
description: Узнайте о том, как перемещать данные из Amazon Redshift с помощью действия копирования фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 025250f47bf0630be5ae988140a5feeecfd0eaf0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100377556"
---
# <a name="move-data-from-amazon-redshift-using-azure-data-factory"></a>Перемещение данных из Amazon Redshift с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-amazon-redshift-connector.md)
> * [Версия 2 (текущая)](../connector-amazon-redshift.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [соединителе Amazon Redshift в службе "Фабрика данных Azure" версии 2](../connector-amazon-redshift.md).

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure перемещать данные из Amazon Redshift. Эта статья является продолжением статьи о [действиях перемещения данных](data-factory-data-movement-activities.md), в которой приведены общие сведения о перемещении данных с помощью действия копирования.

Сейчас фабрика данных поддерживает только перемещение данных из Amazon Redshift в другое [поддерживаемое хранилище-приемник](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Перемещение данных из других хранилищ в Amazon Redshift не поддерживается.

> [!TIP]
> Чтобы обеспечить наилучшую производительность при копировании больших объемов данных из Amazon Redshift, рекомендуется использовать встроенную команду Redshift **UNLOAD** через Amazon Simple Storage Service (Amazon S3). Дополнительные сведения см. в разделе [Копирование данных из Amazon Redshift с помощью UNLOAD](#use-unload-to-copy-data-from-amazon-redshift).

## <a name="prerequisites"></a>Предварительные условия
* Для перемещения данных в локальное хранилище установите [шлюз управления данными](data-factory-data-management-gateway.md) на локальный компьютер. Предоставьте доступ для шлюза в кластер Amazon Redshift с помощью IP-адреса локального компьютера. Инструкции см. в статье об [авторизации доступа к кластеру](https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-authorize-cluster-access.html).
* Дополнительные сведения о перемещении в хранилище данных Azure см. в статье [Microsoft Azure Datacenter IP Ranges ](https://www.microsoft.com/download/details.aspx?id=41653) (Диапазоны IP-адресов центра обработки данных Microsoft Azure).

## <a name="getting-started"></a>Начало работы
Вы можете создать конвейер с действием копирования, который перемещает данные из источника Amazon Redshift, с помощью разных инструментов и API.

Проще всего создать конвейер с помощью мастера копирования фабрики данных Azure. Краткие пошаговые указания по созданию конвейера с помощью мастера копирования приведены в статье [Руководство. Создание конвейера с действием копирования с помощью мастера копирования фабрики данных](data-factory-copy-data-wizard-tutorial.md).

Кроме того, конвейер можно создать с помощью Visual Studio, Azure PowerShell или других средств. Шаблоны Azure Resource Manager, .NET API и REST API также можно использовать для создания конвейера. Пошаговые инструкции по созданию конвейера с действием копирования см. в [руководстве по действию копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Независимо от используемого средства или API-интерфейса, для создания конвейера, который перемещает данные из источника данных в приемник, выполняются следующие шаги:

1. Создайте связанные службы, чтобы связать входные и выходные данные с фабрикой данных.
2. Создайте наборы данных, которые представляют входные и выходные данные для операции копирования.
3. Создайте конвейер с действием копирования, который принимает входной набор данных и возвращает выходной набор данных.

При использовании мастера копирования для этих сущностей фабрики данных автоматически создаются определения JSON. При использовании инструментов или интерфейсов API (за исключением API .NET) вы самостоятельно определяете сущности фабрики данных в формате JSON. В разделе Пример JSON. Копирование данных из Amazon Redshift в хранилище больших двоичных объектов Azure показаны определения JSON для сущностей фабрики данных, которые используются для копирования данных из локального хранилища данных Amazon Redshift.

В следующих разделах описаны свойства JSON, используемые для определения сущностей фабрики данных для Amazon Redshift.

## <a name="linked-service-properties"></a>Свойства связанной службы

В таблице ниже приведены описания элементов JSON, которые относятся к связанной службе Amazon Redshift.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| **type** |Этому свойству необходимо задать значение **AmazonRedshift**. |Да |
| **server** |IP-адрес или имя узла сервера Amazon Redshift. |Да |
| **port** |Номер TCP-порта, используемого сервером Amazon Redshift для прослушивания клиентских подключений. |Нет (значение по умолчанию — 5439) |
| **database** |Имя базы данных Amazon Redshift. |Да |
| **username** |Имя пользователя, имеющего доступ к базе данных. |Да |
| **password** |Пароль для учетной записи пользователя. |Да |

## <a name="dataset-properties"></a>Свойства набора данных

Список разделов и свойств, используемых для определения наборов данных, см. в статье [Наборы данных в фабрике данных Azure](data-factory-create-datasets.md). Разделы **structure**, **availability** и **policy** одинаковые для всех типов наборов данных. Примеры типов наборов данных включают Azure SQL, хранилище BLOB-объектов Azure и хранилище таблиц Azure.

Раздел **typeProperties** во всех типах наборов данных разный. Он содержит сведения о расположении данных в хранилище. Раздел **typeProperties** набора данных типа **RelationalTable** (который включает в себя набор данных Amazon Redshift) содержит следующие свойства:

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| **tableName** |Имя таблицы в базе данных Amazon Redshift, на которое ссылается связанная служба. |Нет (если указано свойство **query** действия копирования типа **RelationalSource**). |

## <a name="copy-activity-properties"></a>Свойства действия копирования

Список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](data-factory-create-pipelines.md). Свойства **имя**, **описание**, **входные** и **выходные** таблицы и **политика** доступны для всех типов действий. Свойства, доступные в разделе действия **typeProperties**, зависят от типа действия. Для действия копирования свойства различаются в зависимости от типов источников и приемников данных.

Если действие копирования относится к типу **AmazonRedshiftSource**, в разделе **typeProperties** для него доступны следующие свойства:

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| **запрос** | Используйте пользовательский запрос для чтения данных. |Нет (если для свойства **tableName** задано значение dataset). |
| **redshiftUnloadSettings** | Содержит группу свойств при использовании команды Redshift **UNLOAD**. | Нет |
| **s3LinkedServiceName** | Используется Amazon S3 в качестве промежуточного хранилища. Связанная служба указывается с помощью имени типа фабрики данных Azure **AwsAccessKey**. | Требуется при использовании свойства **redshiftUnloadSettings**. |
| **bucketName** | Указывает контейнер Amazon S3 для хранения промежуточных данных. Если это свойство не указано, действие копирования автоматически создаст контейнер. | Требуется при использовании свойства **redshiftUnloadSettings**. |

Кроме того, можно использовать тип **RelationalSource** (включающий Amazon Redshift) со следующим свойством в разделе **typeProperties**. Обратите внимание, что этот тип источника не поддерживает Redshift команду **UNLOAD**.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| **запрос** |Используйте пользовательский запрос для чтения данных. | Нет (если для свойства **tableName** задано значение dataset). |

## <a name="use-unload-to-copy-data-from-amazon-redshift"></a>Копирование данных из Amazon Redshift с помощью UNLOAD

Команда Amazon Redshift [**UNLOAD**](https://docs.aws.amazon.com/redshift/latest/dg/r_UNLOAD.html) выгружает результаты запроса для одного или нескольких файлов в Amazon S3. Эта команда рекомендуется Amazon для копирования больших наборов данных из Redshift.

**Пример. копирование данных из Amazon RedShift в Azure синапсе Analytics**

В этом примере данные из Amazon RedShift копируются в Azure синапсе Analytics. В этом примере используется команда Redshift **UNLOAD**, промежуточные данные копирования и Microsoft PolyBase.

В этом примере варианта использования действие копирования сначала выгружает данные из Amazon Redshift Amazon S3, как указано в параметре **redshiftUnloadSettings**. Затем данные копируются из Amazon S3 в хранилище BLOB-объектов Azure, как указано в параметре **stagingSettings**. Наконец, Polybase загружает данные в Azure синапсе Analytics. Все промежуточные форматы обновляются действием копирования.

![Копирование рабочего процесса из Amazon RedShift в Azure синапсе Analytics](media/data-factory-amazon-redshift-connector/redshift-to-sql-dw-copy-workflow.png)

```json
{
    "name": "CopyFromRedshiftToSQLDW",
    "type": "Copy",
    "typeProperties": {
        "source": {
            "type": "AmazonRedshiftSource",
            "query": "select * from MyTable",
            "redshiftUnloadSettings": {
                "s3LinkedServiceName":"MyAmazonS3StorageLinkedService",
                "bucketName": "bucketForUnload"
            }
        },
        "sink": {
            "type": "SqlDWSink",
            "allowPolyBase": true
        },
        "enableStaging": true,
        "stagingSettings": {
            "linkedServiceName": "MyAzureStorageLinkedService",
            "path": "adfstagingcopydata"
        },
        "cloudDataMovementUnits": 32
        .....
    }
}
```

## <a name="json-example-copy-data-from-amazon-redshift-to-azure-blob-storage"></a>Пример JSON. Копирование данных из Amazon Redshift в хранилище BLOB-объектов Azure
В этом примере показано, как скопировать данные из базы данных Amazon Redshift в хранилище BLOB-объектов Azure. Данные можно скопировать непосредственно в любой [поддерживаемый приемник](data-factory-data-movement-activities.md#supported-data-stores-and-formats) с помощью действия копирования.

Образец состоит из следующих сущностей фабрики данных.

* Связанная служба типа [AmazonRedshift](#linked-service-properties)
* Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Входной [набор данных](data-factory-create-datasets.md) типа [RelationalTable](#dataset-properties)
* Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties)
* [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются свойства [RelationalSource](#copy-activity-properties) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

В примере данные из результата запроса к Amazon Redshift каждый час копируются в большой двоичный объект Azure. Используемые в этом примере свойства JSON описаны в разделах, следующих за определениями сущностей.

**Связанная служба Amazon Redshift**

```json
{
    "name": "AmazonRedshiftLinkedService",
    "properties":
    {
        "type": "AmazonRedshift",
        "typeProperties":
        {
            "server": "< The IP address or host name of the Amazon Redshift server >",
            "port": "<The number of the TCP port that the Amazon Redshift server uses to listen for client connections.>",
            "database": "<The database name of the Amazon Redshift database>",
            "username": "<username>",
            "password": "<password>"
        }
    }
}
```

**Связанная служба хранилища BLOB-объектов Azure**

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
**Входной набор данных Amazon Redshift**

Свойство **external** имеет значение true, чтобы сообщить службе фабрики данных о том, что набор данных является внешним по отношению к фабрике данных. Этот параметр указывает, что набор данных не был создан из действия в фабрике данных. Присвойте значение true этому параметру входного набора данных, который не является результатом какого-либо действия в конвейере.

```json
{
    "name": "AmazonRedshiftInputDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "AmazonRedshiftLinkedService",
        "typeProperties": {
            "tableName": "<Table name>"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

**Выходной набор данных BLOB-объекта Azure**

Данные записываются в новый большой двоичный объект каждый час. Для этого свойству **frequency** присваивается значение Hour, а свойству **interval** — 1. Свойство **folderPath** для большого двоичного объекта вычисляется динамически. Значение свойства основано на времени начала обрабатываемого среза. В пути к папке используются год, месяц, день и час времени начала.

```json
{
    "name": "AzureBlobOutputDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/fromamazonredshift/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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

**Действие копирования в конвейере с Azure Redshift (типа RelationalSource) в качестве источника и большим двоичным объектом Azure в качестве приемника**

Конвейер содержит действие копирования, которое использует входные и выходные наборы данных. Конвейер планируется запускать каждый час. В определении JSON для конвейера типу **source** присвоено значение **RelationalSource**, а типу **sink** — значение **BlobSink**. SQL-запрос, указанный для свойства **query**, выбирает для копирования данные за последний час.

```json
{
    "name": "CopyAmazonRedshiftToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "AmazonRedshiftSource",
                        "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)",
                        "redshiftUnloadSettings": {
                            "s3LinkedServiceName":"myS3Storage",
                            "bucketName": "bucketForUnload"
                        }
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    },
                    "cloudDataMovementUnits": 32
                },
                "inputs": [
                    {
                        "name": "AmazonRedshiftInputDataset"
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
                "name": "AmazonRedshiftToBlob"
            }
        ],
        "start": "2014-06-01T18:00:00Z",
        "end": "2014-06-01T19:00:00Z"
    }
}
```
### <a name="type-mapping-for-amazon-redshift"></a>Сопоставление типов для Amazon Redshift
Как упоминалось в статье о [действиях перемещения данных](data-factory-data-movement-activities.md), во время копирования типы источников автоматически преобразовываются в типы приемников. Такое преобразование выполняется в два этапа:

1. Преобразование собственных типов источников в тип .NET.
2. Преобразование типа .NET в собственный тип приемника.

Когда действие копирования преобразует данные из типа Amazon Redshift в тип .NET, используются следующие сопоставления:

| Тип Amazon Redshift | Тип .NET |
| --- | --- |
| SMALLINT |Int16 |
| INTEGER |Int32 |
| bigint |Int64 |
| DECIMAL |Decimal |
| real |Single |
| DOUBLE PRECISION |Double |
| BOOLEAN |Строка |
| CHAR |Строка |
| VARCHAR |Строка |
| DATE |Дата/время |
| timestamp |DateTime |
| TEXT |Строка |

## <a name="map-source-to-sink-columns"></a>Сопоставление столбцов источника и приемника
Сведения о сопоставлении столбцов в наборе данных, используемом в качестве источника, со столбцами в приемнике см. в статье [Сопоставление столбцов исходного набора данных со столбцами целевого набора данных](data-factory-map-columns.md).

## <a name="repeatable-reads-from-relational-sources"></a>Повторяющиеся операции чтения из реляционных источников
При копировании данных из реляционных хранилищ важно помнить о повторяемости, чтобы избежать непредвиденных результатов. В фабрике данных Azure можно вручную повторно выполнить срез. Вы можете также настроить для набора данных **политику** повтора, чтобы при сбое срез выполнялся повторно. Убедитесь, что при каждом запуске среза считываются одни и те же данные независимо от способа повторного запуска среза. Дополнительные сведения см. в разделе [Повторяющиеся операции чтения из реляционных источников](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="performance-and-tuning"></a>Производительность и настройка
Сведения о ключевых факторах, влияющих на производительность действия копирования, и различных способах оптимизации этого процесса см. в статье [Руководство по настройке производительности действия копирования](data-factory-copy-activity-performance.md).

## <a name="next-steps"></a>Дальнейшие действия
Пошаговые инструкции по созданию конвейера с действием копирования см. в руководстве [Копирование данных из хранилища BLOB-объектов Azure в базу данных SQL с помощью фабрики данных](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
