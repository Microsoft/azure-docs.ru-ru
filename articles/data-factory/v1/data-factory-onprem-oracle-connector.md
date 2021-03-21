---
title: Копирование данных в Oracle или из него с помощью фабрики данных
description: Узнайте, как копировать данные в локальную базу данных Oracle или из нее с помощью Фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 05/15/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 02fc142a08176aa577250417c0e394218e832f34
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100387348"
---
# <a name="copy-data-to-or-from-oracle-on-premises-by-using-azure-data-factory"></a>Копирование данных в локальную базу данных Oracle и обратно с помощью Фабрики данных Azure

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-onprem-oracle-connector.md)
> * [Версия 2 (текущая)](../connector-oracle.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию службы "Фабрика данных", ознакомьтесь с руководством по [использованию соединителя Oracle в службе "Фабрика данных Azure" версии 2](../connector-oracle.md).


В этой статье рассказывается, как с помощью действия копирования в Фабрике данных Azure перемещать данные в локальную базу данных Oracle и обратно. Это продолжение статьи о [действиях перемещения данных](data-factory-data-movement-activities.md), в которой приведены общие сведения о перемещении данных с помощью действия копирования.

## <a name="supported-scenarios"></a>Поддерживаемые сценарии

Данные можно скопировать *из базы данных Oracle* в следующие хранилища данных:

[!INCLUDE [data-factory-supported-sink](../../../includes/data-factory-supported-sinks.md)]

Данные можно скопировать *в базу данных Oracle* из следующих хранилищ данных:

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

## <a name="prerequisites"></a>Предварительные условия

Фабрика данных поддерживает подключение к локальным источникам Oracle с помощью шлюза управления данными. Дополнительные сведения о шлюзе управления данными см. в [этой](data-factory-data-management-gateway.md) статье. Пошаговые инструкции по настройке шлюза для перемещения данных с помощью конвейера см. в статье, посвященной [перемещению данных между локальными источниками и облаком](data-factory-move-data-between-onprem-and-cloud.md).

Шлюз является обязательным, даже если база данных Oracle размещается на виртуальной машине IaaS (инфраструктура как услуга) Azure. Шлюз можно установить на той же виртуальной машине IaaS, на которой размещается хранилище данных, или на другой виртуальной машине. Важно чтобы шлюз мог подключиться к базе данных.

> [!NOTE]
> Советы по устранению неполадок, связанных со шлюзом или подключением, см. в разделе об [устранении неполадок в работе шлюза](data-factory-data-management-gateway.md#troubleshooting-gateway-issues).

## <a name="supported-versions-and-installation"></a>Поддерживаемые версии и установка

Этот соединитель Oracle поддерживает две версии драйверов.

- **Драйвер Майкрософт для Oracle (рекомендуется)**. Начиная со шлюза управления данными версии 2.7, драйвер Майкрософт для Oracle устанавливается автоматически вместе со шлюзом, поэтому не нужно устанавливать или обновлять драйвер для подключения к Oracle. Кроме того, производительность копирования при использовании этого драйвера может увеличиться. Ниже перечислены поддерживаемые версии баз данных Oracle.
  - Oracle 12c R1 (12.1)
  - Oracle 11g R1, R2 (11.1, 11.2)
  - Oracle 10g R1, R2 (10.1, 10.2)
  - Oracle 9i R1, R2 (9.0.1, 9.2)
  - Oracle 8i R3 (8.1.7)

    > [!NOTE]
    > Прокси-сервер Oracle не поддерживается.

    > [!IMPORTANT]
    > В настоящее время драйвер Майкрософт для Oracle поддерживает только копирование данных из Oracle. Драйвер не поддерживает запись в базу данных Oracle. Обратите внимание, что этот драйвер не поддерживает возможность тестирования подключения на вкладке **Диагностика** в шлюзе управления данными. Кроме того, для проверки подключения можно воспользоваться мастером копирования.
    >

- **Поставщик данных Oracle для .NET**. Для копирования данных в базу данных Oracle и из нее также можно использовать поставщик данных Oracle. Входит в состав [компонентов доступа к данным Oracle для Windows](https://www.oracle.com/technetwork/topics/dotnet/downloads/). Установите соответствующую версию (32- или 64-разрядную) на компьютере, на котором установлен шлюз. [Поставщик данных Oracle для .NET 12.1](https://docs.oracle.com/database/121/ODPNT/InstallSystemRequirements.htm#ODPNT149) может обращаться к Oracle Database 10g версии 2 или более поздней.

    Если вы выбрали **установку XCopy**, то следуйте инструкциям в файле readme.htm. Мы рекомендуем выбрать установщик, который включает пользовательский интерфейс (не установщик XCopy).

    После установки поставщика перезапустите на компьютере службу узла шлюза управления данными, используя приложение "Службы" или диспетчер конфигурации шлюза управления данными.

При использовании мастера копирования для создания конвейера копирования тип драйвера будет определен автоматически. Драйвер Майкрософт используется по умолчанию, если только вы не используете версию шлюза ниже 2.7 или выбрали Oracle в качестве приемника.

## <a name="get-started"></a>Начало работы

Вы можете создать конвейер с действием копирования. Конвейер перемещает данные в локальную базу данных Oracle или из нее с помощью разных инструментов или API.

Проще всего создать конвейер с помощью мастера копирования. В статье, посвященной [созданию конвейера с помощью мастера копирования](data-factory-copy-data-wizard-tutorial.md), приведены краткие пошаговые указания по созданию конвейера с помощью мастера копирования данных.

Для создания конвейера можно также использовать одно из следующих средств: **Visual Studio**, **Azure PowerShell**, **шаблон Azure Resource Manager**, **API .NET** или **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в [руководстве по действию копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Независимо от используемого инструмента или интерфейса API для создания конвейера, который перемещает данные из исходного хранилища данных в приемник, необходимо выполнить указанные ниже действия.

1. Создайте **фабрику данных**. Фабрика данных может содержать один или несколько конвейеров.
2. Создайте **связанные службы** , чтобы связать входные и выходные данные хранилища с фабрикой данных. Например, при копировании данных из базы данных Oracle в хранилище BLOB-объектов Azure создайте две связанные службы, чтобы связать базу данных Oracle и учетную запись хранения Azure с фабрикой данных. Сведения о свойствах связанной службы, относящихся к Oracle, см. в разделе [свойств связанной службы](#linked-service-properties).
3. Создайте **наборы данных**, которые представляют входные и выходные данные для операции копирования. В примере, упомянутом на последнем шаге, создайте набор данных, чтобы указать таблицу в базе данных Oracle, содержащую входные данные. Создайте другой набор данных, чтобы указать контейнер больших двоичных объектов и папку, содержащую данные, скопированные из базы данных Oracle. Сведения о свойствах набора данных, относящихся к Oracle, см. в [этом разделе](#dataset-properties).
4. Создайте **конвейер** с действием копирования, который принимает входной набор данных и возвращает выходной набор данных. В примере выше **OracleSource** используется как источник, а **BlobSink** — как приемник для действия копирования. Аналогично при копировании из хранилища BLOB-объектов Azure в базу данных Oracle в действии копирования используются **BlobSource** и **OracleSink**. Сведения о свойствах действия копирования, относящихся к базе данных Oracle, см. в [этом разделе](#copy-activity-properties). Для получения сведений о том, как использовать хранилище данных в качестве источника или приемника, щелкните ссылку в предыдущем разделе.

Если вы используете мастер, то он автоматически создает определения JSON для сущностей Фабрики данных (связанных служб, наборов данных и конвейера). При использовании инструментов или интерфейсов API (за исключением API .NET) вы самостоятельно определяете эти сущности Фабрики данных в формате JSON. Примеры с определениями JSON для сущностей Фабрики данных, которые используются для копирования данных в локальную базу данных Oracle и обратно, см. в разделе "Примеры JSON".

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей Фабрики данных.

## <a name="linked-service-properties"></a>Свойства связанной службы

В приведенной ниже таблице описываются элементы JSON, которые относятся к связанной службе Oracle.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| type |Для свойства **Type** необходимо задать значение **OnPremisesOracle**. |Да |
| driverType | Укажите, какой драйвер следует использовать для копирования данных в базу данных Oracle и из нее. Допустимые значения: **Майкрософт** или **ODP** (по умолчанию). Дополнительные сведения о драйверах см. в разделе [Поддерживаемые версии и установка](#supported-versions-and-installation). | Нет |
| connectionString | В свойстве **connectionString** указываются сведения, необходимые для подключения к экземпляру базы данных Oracle. | Да |
| gatewayName | Имя шлюза, который используется для подключения к локальному серверу Oracle. |Да |

**Пример: использование драйвера Майкрософт**

> [!TIP]
> Если при использовании версии Oracle 8i возникает ошибка "ORA-01025: значение параметра UPI вне допустимого диапазона", добавьте `WireProtocolMode=1` в строку подключения и повторите попытку.

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "driverType": "Microsoft",
            "connectionString":"Host=<host>;Port=<port>;Sid=<service ID>;User Id=<user name>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

**Пример: использование драйвера ODP**

Дополнительные сведения о допустимых форматах см. на странице о [поставщике данных Oracle для ODP .NET](https://www.connectionstrings.com/oracle-data-provider-for-net-odp-net/).

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "connectionString": "Data Source=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<host name>)(PORT=<port number>))(CONNECT_DATA=(SERVICE_NAME=<service ID>))); User Id=<user name>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, используемых для определения наборов данных, см. в статье [Наборы данных в фабрике данных Azure](data-factory-create-datasets.md).

Разделы файла JSON в наборе данных, такие как структура, доступность и политика, одинаковы для всех типов наборов данных (например, база данных Oracle, хранилище BLOB-объектов Azure и табличное хранилище Azure).

Раздел **typeProperties** отличается для каждого типа набора данных и предоставляет сведения о расположении данных в хранилище данных. Раздел **typeProperties** набора данных типа **OracleTable** имеет следующие свойства.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| tableName |Имя таблицы в базе данных Oracle, на которое ссылается связанная служба. |Нет (если указан параметр **oracleReaderQuery** объекта **OracleSource**) |

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, доступных для определения действий, см. в статье, посвященной [конвейерам и действиям в Фабрике данных Azure](data-factory-create-pipelines.md).

Свойства (включая имя, описание, входные и выходные таблицы, политику и т. д.) доступны для всех типов действий.

> [!NOTE]
> Действие копирования принимает только один набор входных данных и возвращает только один набор выходных.

Свойства, доступные в разделе действия **typeProperties**, зависят от типа действия. Для действия копирования свойства различаются в зависимости от типов источников и приемников.

### <a name="oraclesource"></a>OracleSource

Если источник относится к типу **OracleSource**, в разделе **typeProperties** для действия копирования доступны следующие свойства:

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| oracleReaderQuery |Используйте пользовательский запрос для чтения данных. |Строка запроса SQL. Например, "select \* from **MyTable**". <br/><br/>Если не указано другое, выполняется инструкция SQL: "select \* from **MyTable**" |Нет<br />(если для свойства **tableName** задано значение **dataset**) |

### <a name="oraclesink"></a>OracleSink

**OracleSink** поддерживает следующие свойства:

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| writeBatchTimeout |Время ожидания до выполнения операции пакетной вставки, пока не завершится срок ее действия. |**Интервал времени**<br/><br/> Пример: 00:30:00 (30 минут). |Нет |
| writeBatchSize |Вставляет данные в таблицу SQL, когда размер буфера достигает значения **writeBatchSize**. |Целое число (количество строк) |Нет (значение по умолчанию — 100) |
| sqlWriterCleanupScript |Указывает запрос на выполнение действия копирования, позволяющий убедиться в том, что данные конкретного среза очищены. |Инструкция запроса. |Нет |
| sliceIdentifierColumnName |Указывает имя столбца, в которое действие копирования добавляет автоматически созданный идентификатор среза. Значение для имени **sliceIdentifierColumnName** используется для очистки данных конкретного среза при повторном запуске. |Имя столбца с типом данных **binary(32)**. |Нет |

## <a name="json-examples-for-copying-data-to-and-from-the-oracle-database"></a>Примеры JSON для копирования данных в базу данных Oracle и обратно

Следующие примеры содержат образцы определений JSON, которые можно использовать для создания конвейера с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) или [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). В примерах показано, как копировать данные из базы данных Oracle и хранилища BLOB-объектов Azure и обратно. Однако данные можно копировать в любые приемники, указанные в статье о [поддерживаемых хранилищах данных и форматах](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Для этого применяется действие копирования в Фабрике данных.

**Пример: копирование данных из Oracle в хранилище BLOB-объектов Azure**

Пример содержит следующие сущности фабрики данных.

* Связанная служба типа [OnPremisesOracle](data-factory-onprem-oracle-connector.md#linked-service-properties).
* Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Входной [набор данных](data-factory-create-datasets.md) типа [OracleTable](data-factory-onprem-oracle-connector.md#dataset-properties).
* Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [OracleSource](data-factory-onprem-oracle-connector.md#copy-activity-properties) в качестве источника и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) в качестве приемника.

В примере данные из локальной таблицы Oracle ежечасно копируются в хранилище BLOB-объектов. Дополнительные сведения о различных свойствах, которые используются в примере, см. в разделах после примеров.

**Связанная служба Oracle**

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "driverType": "Microsoft",
            "connectionString":"Host=<host>;Port=<port>;Sid=<service ID>;User Id=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

**Связанная служба хранилища BLOB-объектов Azure**

```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>"
        }
    }
}
```

**Входной набор данных Oracle**

В примере предполагается, что вы создали в Oracle таблицу **MyTable**. Она содержит столбец с именем **timestampcolumn** для данных временных рядов.

Если параметру **external** присвоить значение **true**, то Фабрика данных воспримет этот набор данных как внешний и созданный не в результате какого-либо действия в этой фабрике данных.

```json
{
    "name": "OracleInput",
    "properties": {
        "type": "OracleTable",
        "linkedServiceName": "OnPremisesOracleLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "offset": "01:00:00",
            "interval": "1",
            "anchorDateTime": "2014-02-27T12:00:00",
            "frequency": "Hour"
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

**Выходной набор данных BLOB-объекта Azure**

Данные записываются в новый большой двоичный объект каждый час (**Частота**: **час**, **интервал**: **1**). Путь к папке с большим двоичным объектом и имя файла вычисляются динамически на основе времени начала обработки среза. В пути к папке используется год, месяц, день и час времени запуска.

```json
{
    "name": "AzureBlobOutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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
            ],
            "format": {
                "type": "TextFormat",
                "columnDelimiter": "\t",
                "rowDelimiter": "\n"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Конвейер с действием копирования**

Конвейер содержит действие копирования, которое использует входные и выходные наборы данных и выполняется ежечасно. В определении JSON конвейера для типа **source** устанавливается значение **OracleSource**, а для типа **sink** — значение **BlobSink**. SQL-запрос, указанный с помощью свойства **oracleReaderQuery**, выбирает для копирования данные за последний час.

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for a copy activity",
        "activities":[
            {
                "name": "OracletoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": " OracleInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "OracleSource",
                        "oracleReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```

**Пример: копирование данных из хранилища BLOB-объектов Azure в Oracle**

В этом примере показано, как скопировать данные из хранилища BLOB-объектов Azure в локальную базу данных Oracle. Тем не менее вы можете скопировать данные *непосредственно* из любых источников, перечисленных в статье о [поддерживаемых хранилищах данных и форматах](data-factory-data-movement-activities.md#supported-data-stores-and-formats), с помощью действия копирования в Фабрике данных Azure.

Пример содержит следующие сущности фабрики данных.

* Связанная служба типа [OnPremisesOracle](data-factory-onprem-oracle-connector.md#linked-service-properties).
* Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Входной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* Выходной [набор данных](data-factory-create-datasets.md) типа [OracleTable](data-factory-onprem-oracle-connector.md#dataset-properties).
* [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) в качестве источника и [OracleSink](data-factory-onprem-oracle-connector.md#copy-activity-properties) в качестве приемника.

В примере данные из хранилища BLOB-объектов каждый час копируются в таблицу в локальной базе данных Oracle. Дополнительные сведения о различных свойствах, которые используются в примере, см. в разделах после примеров.

**Связанная служба Oracle**

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "connectionString": "Data Source=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<host name>)(PORT=<port number>))(CONNECT_DATA=(SERVICE_NAME=<service ID>)));
            User Id=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

**Связанная служба хранилища BLOB-объектов Azure**

```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>"
        }
    }
}
```

**Входной набор данных большого двоичного объекта Azure**

Данные выбираются из нового большого двоичного объекта каждый час (**Частота**: **час**, **интервал**: **1**). Путь к папке с большим двоичным объектом и имя файла вычисляются динамически на основе времени начала обработки среза. В пути к папке используется год, месяц и день начала. В имени файла используется часть времени начала, обозначающая час. Если для параметра **external** задать значение **true**, то Фабрика данных воспримет эту таблицу как внешнюю, которая создана не в результате какого-либо действия в этой службе.

```json
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
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
                }
            ],
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            }
        },
        "external": true,
        "availability": {
            "frequency": "Day",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

**Выходной набор данных Oracle**

В примере предполагается, что вы создали в Oracle таблицу **MyTable**. Таблицу в Oracle нужно создать с таким же количеством столбцов, которое должно быть в CSV-файле большого двоичного объекта. Новые строки добавляются в таблицу каждый час.

```json
{
    "name": "OracleOutput",
    "properties": {
        "type": "OracleTable",
        "linkedServiceName": "OnPremisesOracleLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "availability": {
            "frequency": "Day",
            "interval": "1"
        }
    }
}
```

**Конвейер с действием копирования**

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** устанавливается значение **BlobSource**, а для типа **sink** — значение **OracleSink**.

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-05T19:00:00",
        "description":"pipeline with a copy activity",
        "activities":[
            {
                "name": "AzureBlobtoOracle",
                "description": "Copy Activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureBlobInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "OracleOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "OracleSink"
                    }
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```


## <a name="troubleshooting-tips"></a>Советы по устранению неполадок

### <a name="problem-1-net-framework-data-provider"></a>Проблема 1: поставщик данных .NET Framework

**Сообщение об ошибке**

```text
Copy activity met invalid parameters: 'UnknownParameterName', Detailed message: Unable to find the requested .NET Framework Data Provider. It may not be installed.
```

**Возможные причины**

* Поставщик данных .NET Framework для Oracle не был установлен.
* Поставщик данных .NET Framework для Oracle был установлен в .NET Framework 2.0 и не найден в папках .NET Framework 4.0.

**Решение**

* Если поставщик данных .NET для Oracle не установлен, [установите его](https://www.oracle.com/technetwork/topics/dotnet/downloads/) и повторите сценарий.
* Если сообщение об ошибке появляется даже после установки поставщика, выполните следующие действия:
    1. Откройте файл конфигурации .NET 2.0 из папки: <системный диск\>:\Windows\Microsoft.NET\Framework64\v2.0.50727\CONFIG\machine.config.
    2. Найдите **поставщика данных Oracle для .NET**, Вы сможете найти запись, как показано в следующем примере в разделе **System. Data**  >  **дбпровидерфакториес**:`<add name="Oracle Data Provider for .NET" invariant="Oracle.DataAccess.Client" description="Oracle Data Provider for .NET" type="Oracle.DataAccess.Client.OracleClientFactory, Oracle.DataAccess, Version=2.112.3.0, Culture=neutral, PublicKeyToken=89b483f429c47342" />`
* Скопируйте эту запись в файл machine.config в следующей папке .NET 4,0: <системного диска \>:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config. Затем измените версию на 4. xxx. x.x.
* Установите файл <путь установки ODP.NET\>\11.2.0\client_1\odp.net\bin\4\Oracle.DataAccess.dll в глобальный кэш сборок (GAC), выполнив команду **gacutil /i [путь к поставщику]**.

### <a name="problem-2-datetime-formatting"></a>Проблема 2: формат даты и времени

**Сообщение об ошибке**

```text
Message=Operation failed in Oracle Database with the following error: 'ORA-01861: literal does not match format string'.,Source=,''Type=Oracle.DataAccess.Client.OracleException,Message=ORA-01861: literal does not match format string,Source=Oracle Data Provider for .NET,'.
```

**Решение**

Необходимо настроить строку запроса в действии копирования в зависимости от того, как настроены даты в базе данных Oracle. Ниже приведен пример (с использованием функции **to_date**):

```console   
"oracleReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= to_date(\\'{0:MM-dd-yyyy HH:mm}\\',\\'MM/DD/YYYY HH24:MI\\') AND timestampcolumn < to_date(\\'{1:MM-dd-yyyy HH:mm}\\',\\'MM/DD/YYYY HH24:MI\\') ', WindowStart, WindowEnd)"
```


## <a name="type-mapping-for-oracle"></a>Сопоставление типов для Oracle

Как упоминалось в статье о [действиях перемещения данных](data-factory-data-movement-activities.md), во время копирования типы источников автоматически преобразовываются в типы приемников. Такое преобразование выполняется в два этапа:

1. Преобразование собственных типов источников в тип .NET.
2. Преобразование типа .NET в собственный тип приемника.

Когда данные перемещаются из Oracle, для преобразования типа данных Oracle в тип .NET и наоборот используются следующие сопоставления.

| Тип данных Oracle | Тип данных платформы .NET |
| --- | --- |
| BFILE |Byte[] |
| BLOB |Byte[]<br/>(поддерживается только в Oracle 10g и более поздних версиях при использовании драйвера Майкрософт) |
| CHAR |Строка |
| CLOB |Строка |
| DATE |Дата/время |
| FLOAT |десятичное число, строка (если точность больше 28) |
| INTEGER |десятичное число, строка (если точность больше 28) |
| INTERVAL YEAR TO MONTH |Int32 |
| INTERVAL DAY TO SECOND |TimeSpan |
| LONG |Строка |
| LONG RAW |Byte[] |
| NCHAR |Строка |
| NCLOB |Строка |
| NUMBER |десятичное число, строка (если точность больше 28) |
| NVARCHAR2 |Строка |
| RAW |Byte[] |
| ROWID |Строка |
| timestamp |Дата/время |
| TIMESTAMP WITH LOCAL TIME ZONE |Дата/время |
| TIMESTAMP WITH TIME ZONE |Дата/время |
| UNSIGNED INTEGER |Число |
| VARCHAR2 |Строка |
| XML |Строка |

> [!NOTE]
> При использовании драйвера Майкрософт типы данных **INTERVAL YEAR TO MONTH** и **INTERVAL DAY TO SECOND** не поддерживаются.

## <a name="map-source-to-sink-columns"></a>Сопоставление столбцов источника и приемника

Сведения о сопоставлении столбцов в наборе данных, используемом в качестве источника, со столбцами в приемнике см. в статье о [сопоставлении столбцов исходного набора данных со столбцами целевого набора данных](data-factory-map-columns.md).

## <a name="repeatable-read-from-relational-sources"></a>Повторяющиеся операции чтения из реляционных источников

При копировании данных из реляционных хранилищ важно помнить о повторяемости, чтобы избежать непредвиденных результатов. В Фабрике данных Azure можно вручную повторно выполнить срез. Вы можете также настроить для набора данных политику повтора, чтобы при сбое срез выполнялся повторно. При повторном выполнении среза в любом случае (вручную или с помощью политики повтора) необходимо убедиться в том, что считываются те же данные, независимо от того, сколько раз выполняется срез. Дополнительные сведения см. в разделе о [повторяющихся операциях чтения из реляционных источников](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="performance-and-tuning"></a>Производительность и настройка

Сведения о ключевых факторах, влияющих на производительность перемещения данных (действие копирования) в Фабрике данных Azure, см. в [руководстве по настройке производительности действия копирования](data-factory-copy-activity-performance.md). Вы также узнаете о различных способах оптимизации этого процесса.
