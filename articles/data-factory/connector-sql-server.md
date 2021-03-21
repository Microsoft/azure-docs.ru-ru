---
title: Копирование данных в SQL Server и из них
description: Узнайте, как перемещать данные в SQL Server базу данных, которая находится локально или на виртуальной машине Azure, с помощью фабрики данных Azure.
ms.author: jingwang
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 03/17/2021
ms.openlocfilehash: 8187a71e89c364ef5a52d7ad4615ed03e6b24a4b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104592062"
---
# <a name="copy-data-to-and-from-sql-server-by-using-azure-data-factory"></a>Копирование данных в SQL Server и из них с помощью фабрики данных Azure

> [!div class="op_single_selector" title1="Выберите версию Фабрики данных, которую вы используете:"]
> * [Версия 1](v1/data-factory-sqlserver-connector.md)
> * [Текущая версия](connector-sql-server.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure копировать данные в SQL Server базу данных и в нее. Она основана на статье [Обзор действия копирования](copy-activity-overview.md) , в которой представлен общий обзор действия копирования.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель SQL Server поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)
- [Действие получения метаданных в Фабрике данных Azure](control-flow-get-metadata-activity.md)

Данные из базы данных SQL Server можно скопировать в любое хранилище данных, поддерживаемое в качестве приемника. Или можно скопировать данные из любого поддерживаемого исходного хранилища данных в базу данных SQL Server. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

В частности, этот соединитель SQL Server поддерживает:

- SQL Server версии 2005 и выше.
- копирование данных с проверкой подлинности SQL или Windows;
- В качестве источника, получая данные с помощью SQL-запроса или хранимой процедуры. Можно также выбрать параллельное копирование из источника SQL Server. Дополнительные сведения см. в разделе [Параллельное копирование из базы данных SQL](#parallel-copy-from-sql-database) .
- В качестве приемника автоматически создает целевую таблицу, если она не существует на основе исходной схемы. Добавление данных в таблицу или вызов хранимой процедуры с пользовательской логикой во время копирования. 

[SQL Server Express LocalDB](/sql/database-engine/configure-windows/sql-server-express-localdb) не поддерживается.

>[!NOTE]
>SQL Server [Always encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) сейчас не поддерживается этим соединителем. Для решения этой проблемы можно использовать [универсальный соединитель ODBC](connector-odbc.md) и драйвер SQL Server ODBC. Следуйте указаниям [по](/sql/connect/odbc/using-always-encrypted-with-the-odbc-driver) загрузке драйвера ODBC и настройке строки подключения.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

## <a name="get-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, относящихся к SQL Server соединителю базы данных.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы SQL Server поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства Type необходимо задать значение **SQLServer**. | Да |
| connectionString |Укажите сведения о **ConnectionString** , необходимые для подключения к SQL Server базе данных с помощью проверки подлинности SQL или Windows. Ознакомьтесь с приведенными ниже примерами.<br/>Вы также можете добавить пароль в Azure Key Vault. Если это проверка подлинности SQL, вытяните `password` конфигурацию из строки подключения. Дополнительные сведения см. в примере JSON, который следует за таблицей, и [Храните учетные данные в Azure Key Vault](store-credentials-in-key-vault.md). |Да |
| userName |При использовании проверки подлинности Windows укажите имя пользователя. Например, **domainname\\username**. |Нет |
| password |Укажите пароль для учетной записи пользователя, указанной для имени пользователя. Пометьте это поле как **SecureString** , чтобы безопасно хранить его в фабрике данных Azure. Вы можете также [указать секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). |Нет |
| connectVia | Это [среда выполнения интеграции](concepts-integration-runtime.md) для подключения к хранилищу данных. Дополнительные сведения см. в разделе [Предварительные требования](#prerequisites). Если не указано другое, используется среда выполнения интеграции Azure по умолчанию. |Нет |

>[!TIP]
>Если вы столкнулись с ошибкой с кодом "UserErrorFailedToConnectToSqlServer" и сообщением, например "ограничение сеанса для базы данных равно XXX и достигнут," добавьте `Pooling=false` в строку подключения и повторите попытку.

**Пример 1. Использование проверки подлинности SQL**

```json
{
    "name": "SqlServerLinkedService",
    "properties": {
        "type": "SqlServer",
        "typeProperties": {
            "connectionString": "Data Source=<servername>\\<instance name if using named instance>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;Password=<password>;"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Пример 2. Использование проверки подлинности SQL с паролем в Azure Key Vault**

```json
{
    "name": "SqlServerLinkedService",
    "properties": {
        "type": "SqlServer",
        "typeProperties": {
            "connectionString": "Data Source=<servername>\\<instance name if using named instance>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;",
            "password": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Пример 3. Использование проверки подлинности Windows**

```json
{
    "name": "SqlServerLinkedService",
    "properties": {
        "type": "SqlServer",
        "typeProperties": {
            "connectionString": "Data Source=<servername>\\<instance name if using named instance>;Initial Catalog=<databasename>;Integrated Security=True;",
            "userName": "<domain\\username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
     }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). В этом разделе содержится список свойств, поддерживаемых набором данных SQL Server.

Чтобы скопировать данные из SQL Server базу данных и в нее, поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение **SqlServerTable**. | Да |
| схема | Имя схемы. |"Нет" для источника, "Да" для приемника  |
| table | Имя таблицы или представления. |"Нет" для источника, "Да" для приемника  |
| tableName | Имя таблицы или представления со схемой. Это свойство поддерживается только для обеспечения обратной совместимости. Для новой рабочей нагрузки используйте `schema` и `table`. | "Нет" для источника, "Да" для приемника |

**Пример**

```json
{
    "name": "SQLServerDataset",
    "properties":
    {
        "type": "SqlServerTable",
        "linkedServiceName": {
            "referenceName": "<SQL Server linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "typeProperties": {
            "schema": "<schema_name>",
            "table": "<table_name>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, доступных для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых источником и приемником SQL Server.

### <a name="sql-server-as-a-source"></a>SQL Server в качестве источника

>[!TIP]
>Чтобы эффективно загружать данные из SQL Server с помощью секционирования данных, Узнайте больше от [параллельного копирования из базы данных SQL](#parallel-copy-from-sql-database).

Чтобы скопировать данные из SQL Server, задайте тип источника **SqlSource** в действии копирования. В разделе source для действия копирования поддерживаются следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type источника действия копирования должно иметь значение **SqlSource**. | Да |
| sqlReaderQuery |Используйте пользовательский SQL-запрос для чтения данных. Например, `select * from MyTable`. |нет |
| sqlReaderStoredProcedureName |Это свойство содержит имя хранимой процедуры, которая считывает данные из исходной таблицы. Последней инструкцией SQL должна быть инструкция SELECT в хранимой процедуре. |нет |
| storedProcedureParameters |Это параметры для хранимой процедуры.<br/>Допустимые значения: пары имен или значений. Имена и регистр параметров должны совпадать с именами и регистром параметров хранимой процедуры. |нет |
| isolationLevel | Задает режим блокировки транзакций для источника данных SQL. Допустимые значения: **ReadCommitted**, **READUNCOMMITTED**, **RepeatableRead**, **Serializable**, **snapshot**. Если значение не указано, используется уровень изоляции базы данных по умолчанию. Дополнительные сведения см. в [этом документе](/dotnet/api/system.data.isolationlevel). | нет |
| partitionOptions | Задает параметры секционирования данных, используемые для загрузки данных из SQL Server. <br>Допустимые значения: **None** (по умолчанию), **фисикалпартитионсофтабле** и **динамикранже**.<br>Если параметр секции включен (то есть не `None` ), степень параллелизма для параллельной загрузки данных из SQL Server управляется [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) параметром действия копирования. | нет |
| partitionSettings | Позволяет указать группу параметров для секционирования данных. <br>Применяется, если параметр Partition не имеет значение `None` . | Нет |
| ***В разделе `partitionSettings` :*** | | |
| partitionColumnName | Укажите имя исходного столбца **в целочисленном или в типе даты/времени** ( `int` , `smallint` , `bigint` , `date` , `smalldatetime` , `datetime` , `datetime2` или `datetimeoffset` ), которое будет использоваться секционированием по диапазонам для параллельного копирования. Если значение не указано, то индекс или первичный ключ таблицы будет обнаружен и использован в качестве столбца секционирования.<br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Если для получения исходных данных используется запрос, присоединитесь к  `?AdfDynamicRangePartitionCondition ` предложению WHERE. Пример см. в разделе [Параллельное копирование из базы данных SQL](#parallel-copy-from-sql-database) . | нет |
| partitionUpperBound | Максимальное значение столбца секционирования для разделения диапазона секций. Это значение используется для выбора шага секционирования, а не для фильтрации строк в таблице. Все строки в таблице или результатах запроса будут секционированы и скопированы. Если не указано, действие копирования автоматически определяет значение.  <br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Пример см. в разделе [Параллельное копирование из базы данных SQL](#parallel-copy-from-sql-database) . | нет |
| partitionLowerBound | Минимальное значение столбца секционирования для разделения диапазона секций. Это значение используется для выбора шага секционирования, а не для фильтрации строк в таблице. Все строки в таблице или результатах запроса будут секционированы и скопированы. Если не указано, действие копирования автоматически определяет значение.<br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Пример см. в разделе [Параллельное копирование из базы данных SQL](#parallel-copy-from-sql-database) . | Нет |

**Обратите внимание на следующие моменты.**

- Если для **SqlSource** указан **sqlReaderQuery** , действие копирования выполняет этот запрос к источнику SQL Server для получения данных. Есть и другой вариант: создать хранимую процедуру, указав ее имя в **sqlReaderStoredProcedureName** и параметры в **storedProcedureParameters**, если она принимает параметры.
- При использовании хранимой процедуры в источнике для получения данных обратите внимание на то, что хранимая процедура разработана так, чтобы она возвращала другую схему при передаче другого значения параметра, может возникнуть сбой или можно увидеть непредвиденный результат при импорте схемы из пользовательского интерфейса или при копировании данных в базу данных SQL с помощью автоматического создания таблиц.

**Пример. Использование SQL Query**

```json
"activities":[
    {
        "name": "CopyFromSQLServer",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SQL Server input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlSource",
                "sqlReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

**Пример. использование хранимой процедуры**

```json
"activities":[
    {
        "name": "CopyFromSQLServer",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SQL Server input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlSource",
                "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
                "storedProcedureParameters": {
                    "stringData": { "value": "str3" },
                    "identifier": { "value": "$$Text.Format('{0:yyyy}', <datetime parameter>)", "type": "Int"}
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

**Определение хранимой процедуры**

```sql
CREATE PROCEDURE CopyTestSrcStoredProcedureWithParameters
(
    @stringData varchar(20),
    @identifier int
)
AS
SET NOCOUNT ON;
BEGIN
    select *
    from dbo.UnitTestSrcTable
    where dbo.UnitTestSrcTable.stringData != stringData
    and dbo.UnitTestSrcTable.identifier != identifier
END
GO
```

### <a name="sql-server-as-a-sink"></a>SQL Server в качестве приемника

> [!TIP]
> Дополнительные сведения о поддерживаемых поведениях записи, конфигурациях и рекомендациях см. в статье рекомендации [по загрузке данных в SQL Server](#best-practice-for-loading-data-into-sql-server).

Чтобы скопировать данные в базу данных SQL Server, установите тип приемника **SqlSink** в действии копирования. В разделе sink для действия копирования поддерживаются следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type приемника действия копирования должно иметь значение **SqlSink**. | Да |
| preCopyScript |Это свойство задает SQL-запрос для выполнения действия копирования перед записью данных в SQL Server. Он вызывается однократно при каждом запуске копирования. Это свойство можно использовать для очистки предварительно загруженных данных. |нет |
| tableOption | Указывает, следует ли [автоматически создавать таблицу приемника, если она](copy-activity-overview.md#auto-create-sink-tables) не существует на основе исходной схемы. Автоматическое создание таблицы не поддерживается, если приемник указывает хранимую процедуру. Допустимые значения: `none` (по умолчанию), `autoCreate`. |нет |
| sqlWriterStoredProcedureName | Имя хранимой процедуры, в которой определяется, как применить исходные данные в целевой таблице. <br/>Эта хранимая процедура будет *вызываться для каждого пакета*. Для операций, выполняемых только один раз и не имеющих никаких действий с исходными данными, например Delete или TRUNCATE, используйте `preCopyScript` свойство.<br>См. пример из [вызова хранимой процедуры из приемника SQL](#invoke-a-stored-procedure-from-a-sql-sink). | Нет |
| сторедпроцедуретаблетипепараметернаме |Имя параметра табличного типа, указанного в хранимой процедуре.  |Нет |
| sqlWriterTableType |Имя типа таблицы, используемое в хранимой процедуре. Действие копирования предоставляет доступ к перемещаемым данным во временной таблице с указанным здесь типом. Это позволяет при выполнении хранимой процедуры объединить копируемые и существующие данные. |нет |
| storedProcedureParameters |Параметры для хранимой процедуры.<br/>Допустимые значения: пары "имя — значение". Имена и регистр параметров должны совпадать с именами и регистром параметров хранимой процедуры. | Нет |
| writeBatchSize |Число строк, вставляемых в таблицу SQL для *каждого пакета*.<br/>Допустимые значения: целое число (количество строк). По умолчанию фабрика данных Azure динамически определяет соответствующий размер пакета в зависимости от размера строки. |Нет |
| writeBatchTimeout |Это свойство определяет время ожидания при выполнении операции пакетной вставки, по истечении которого она считается не выполненной.<br/>Допустимые значения — для интервала времени. Например, "00:30:00" в течение 30 минут. Если значение не указано, время ожидания по умолчанию равно "02:00:00". |Нет |
| maxConcurrentConnections |Верхний предел одновременных подключений, установленных для хранилища данных во время выполнения действия. Укажите значение, только если требуется ограничить количество одновременных подключений.| Нет |

**Пример 1. Добавление данных**

```json
"activities":[
    {
        "name": "CopyToSQLServer",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<SQL Server output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SqlSink",
                "tableOption": "autoCreate",
                "writeBatchSize": 100000
            }
        }
    }
]
```

**Пример 2. вызов хранимой процедуры во время копирования**

Дополнительные сведения см. в разделе [Вызов хранимой процедуры из приемника SQL](#invoke-a-stored-procedure-from-a-sql-sink).

```json
"activities":[
    {
        "name": "CopyToSQLServer",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<SQL Server output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SqlSink",
                "sqlWriterStoredProcedureName": "CopyTestStoredProcedureWithParameters",
                "storedProcedureTableTypeParameterName": "MyTable",
                "sqlWriterTableType": "MyTableType",
                "storedProcedureParameters": {
                    "identifier": { "value": "1", "type": "Int" },
                    "stringData": { "value": "str1" }
                }
            }
        }
    }
]
```

## <a name="parallel-copy-from-sql-database"></a>Параллельная копия из базы данных SQL

Соединитель SQL Server в действии копирования предоставляет встроенное секционирование данных для параллельного копирования данных. Параметры секционирования данных можно найти на вкладке **источник** действия копирования.

![Снимок экрана с параметрами секционирования](./media/connector-sql-server/connector-sql-partition-options.png)

При включении секционированной копии действие копирования выполняет параллельные запросы к источнику SQL Server для загрузки данных по секциям. Степень параллелизма определяется с помощью параметра [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) для действия копирования. Например, если установлено значение `parallelCopies` 4, фабрика данных одновременно создает и выполняет четыре запроса на основе указанного параметра секции и параметров, и каждый запрос извлекает часть данных из SQL Server.

Рекомендуется включить параллельное копирование с секционированием данных, особенно при загрузке большого объема данных из SQL Server. Ниже приведены рекомендуемые конфигурации для разных сценариев. При копировании данных в хранилище данных, основанное на файлах, рекомендуется записывать в папку несколько файлов (указывать только имя папки). в этом случае производительность лучше, чем при записи в один файл.

| Сценарий                                                     | Предлагаемые параметры                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Полная загрузка из большой таблицы с физическими секциями.        | **Параметр секции**: физические разделы таблицы. <br><br/>Во время выполнения фабрика данных автоматически обнаруживает физические секции и копирует данные по секциям. <br><br/>Чтобы проверить, имеет ли таблица физическую секцию, можно обратиться к [этому запросу](#sample-query-to-check-physical-partition). |
| Полная загрузка из большой таблицы без физических секций, в то время как со столбцом типа Integer или DateTime для секционирования данных. | **Partition options** (Параметры секционирования): динамический диапазон.<br>**Столбец секционирования** (необязательно): укажите столбец, используемый для секционирования данных. Если не указано, используется индекс или столбец первичного ключа.<br/>**Верхняя граница секции** и **Нижняя граница секции** (необязательно): укажите, нужно ли определить шаг секционирования. Это не предназначено для фильтрации строк в таблице, все строки в таблице будут секционированы и скопированы. Если значение не указано, действие копирования автоматически определяет значения.<br><br>Например, если столбец секции "ID" имеет диапазон значений от 1 до 100, а нижняя граница равна 20, а верхняя граница — 80, то при параллельном копировании как 4 фабрика данных извлекает данные по 4 секциям в диапазоне <= 20, [21, 50], [51, 80] и >= 81 соответственно. |
| Загрузка большого объема данных с помощью пользовательского запроса без физических секций, в то время как со столбцом типа Integer или Date/DateTime для секционирования данных. | **Partition options** (Параметры секционирования): динамический диапазон.<br>**Запрос**: `SELECT * FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>`.<br>**Столбец секционирования**: укажите столбец для секционирования данных.<br>**Верхняя граница секции** и **Нижняя граница секции** (необязательно): укажите, нужно ли определить шаг секционирования. Это не предназначено для фильтрации строк в таблице, все строки в результатах запроса будут секционированы и скопированы. Если не указано, действие копирования автоматически определяет значение.<br><br>Во время выполнения фабрика данных заменяет `?AdfRangePartitionColumnName` фактические имена столбцов и диапазоны значений для каждой секции и отправляет их в SQL Server. <br>Например, если столбец секции "ID" имеет диапазон значений от 1 до 100, а нижняя граница равна 20, а верхняя граница — 80, то при параллельном копировании как 4 фабрика данных извлекает данные по 4 секциям в диапазоне <= 20, [21, 50], [51, 80] и >= 81 соответственно. <br><br>Ниже приведены дополнительные примеры запросов для различных сценариев.<br> 1. запросите всю таблицу: <br>`SELECT * FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition`<br> 2. запрос из таблицы с выделением столбцов и дополнительными фильтрами WHERE-Order: <br>`SELECT <column_list> FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>`<br> 3. запрос с вложенными запросами: <br>`SELECT <column_list> FROM (<your_sub_query>) AS T WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>`<br> 4. запрос с Секцией во вложенном запросе: <br>`SELECT <column_list> FROM (SELECT <your_sub_query_column_list> FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition) AS T`
|

Рекомендации по загрузке данных с параметром секции:

1. Чтобы избежать неравномерного распределения данных, выберите отличительный столбец в качестве столбца секционирования (например, первичный ключ или уникальный ключ). 
2. Если таблица имеет встроенную секцию, используйте параметр секционирования "физические разделы таблицы", чтобы повысить производительность.    
3. Если для копирования данных используется Azure Integration Runtime, можно задать больший размер "[единицы интеграции данных (Диу)](copy-activity-performance-features.md#data-integration-units)" (>4), чтобы использовать больше вычислительных ресурсов. Проверьте применимые сценарии.
4. "[Степень параллелизма копирования](copy-activity-performance-features.md#parallel-copy)" — Управление номерами секций, установка этого числа слишком большого размера, что значительно вредит производительности, рекомендуется задать это число как (Диу или число размещенных в нем IR-узлов) * (от 2 до 4).

**Пример. полная загрузка из большой таблицы с физическими секциями**

```json
"source": {
    "type": "SqlSource",
    "partitionOption": "PhysicalPartitionsOfTable"
}
```

**Пример: запрос с секционированием по динамическому диапазону**

```json
"source": {
    "type": "SqlSource",
    "query": "SELECT * FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>",
    "partitionOption": "DynamicRange",
    "partitionSettings": {
        "partitionColumnName": "<partition_column_name>",
        "partitionUpperBound": "<upper_value_of_partition_column (optional) to decide the partition stride, not as data filter>",
        "partitionLowerBound": "<lower_value_of_partition_column (optional) to decide the partition stride, not as data filter>"
    }
}
```

### <a name="sample-query-to-check-physical-partition"></a>Пример запроса для проверки физической секции

```sql
SELECT DISTINCT s.name AS SchemaName, t.name AS TableName, pf.name AS PartitionFunctionName, c.name AS ColumnName, iif(pf.name is null, 'no', 'yes') AS HasPartition
FROM sys.tables AS t
LEFT JOIN sys.objects AS o ON t.object_id = o.object_id
LEFT JOIN sys.schemas AS s ON o.schema_id = s.schema_id
LEFT JOIN sys.indexes AS i ON t.object_id = i.object_id 
LEFT JOIN sys.index_columns AS ic ON ic.partition_ordinal > 0 AND ic.index_id = i.index_id AND ic.object_id = t.object_id 
LEFT JOIN sys.columns AS c ON c.object_id = ic.object_id AND c.column_id = ic.column_id 
LEFT JOIN sys.partition_schemes ps ON i.data_space_id = ps.data_space_id 
LEFT JOIN sys.partition_functions pf ON pf.function_id = ps.function_id 
WHERE s.name='[your schema]' AND t.name = '[your table name]'
```

Если таблица имеет физическую секцию, вы увидите «Хаспартитион» как «Да», как показано ниже.

![Результат SQL запроса](./media/connector-azure-sql-database/sql-query-result.png)

## <a name="best-practice-for-loading-data-into-sql-server"></a>Рекомендации по загрузке данных в SQL Server

При копировании данных в SQL Server может потребоваться другое поведение при записи.

- [Append](#append-data): мои исходные данные имеют только новые записи.
- [Upsert](#upsert-data): исходные данные имеют как вставки, так и обновления.
- [Перезаписать](#overwrite-the-entire-table): я хочу каждый раз загружать всю таблицу измерения.
- [Запись с помощью пользовательской логики](#write-data-with-custom-logic): требуется дополнительные операции перед окончательным вставкой в целевую таблицу.

Сведения о настройке в фабрике данных Azure и рекомендациях см. в соответствующих разделах.

### <a name="append-data"></a>Добавление данных.

Добавление данных — это поведение по умолчанию для соединителя приемника SQL Server. Фабрика данных Azure обеспечивает эффективную запись в таблицу с помощью операций групповой вставки. Источник и приемник можно настроить соответствующим образом в действии копирования.

### <a name="upsert-data"></a>Операция upsert с данными

**Вариант 1.** При наличии большого объема данных для копирования можно выполнить загрузку всех записей в промежуточную таблицу с помощью действия копирования, а затем запустить действие хранимой процедуры, чтобы применить инструкцию [Merge](/sql/t-sql/statements/merge-transact-sql) или INSERT или Update в одном снимке. 

В настоящее время действие копирования не поддерживает загрузку данных во временную таблицу базы данных. Существует более сложный способ настройки с помощью сочетания нескольких действий. см. статью [Оптимизация Upsert для базы данных SQL](https://github.com/scoriani/azuresqlbulkupsert). Ниже показан пример использования постоянной таблицы в качестве промежуточного.

Например, в фабрике данных Azure можно создать конвейер с **действием копирования** , связанным с **действием хранимой процедуры**. Первый копирует данные из исходного хранилища в промежуточную таблицу SQL Server, например **упсертстагингтабле**, как имя таблицы в наборе данных. Затем второй вызовет хранимую процедуру для слияния исходных данных из промежуточной таблицы в целевую таблицу и очистки промежуточной таблицы.

![Upsert](./media/connector-azure-sql-database/azure-sql-database-upsert.png)

В базе данных Определите хранимую процедуру с помощью логики слияния, как показано в следующем примере, на который указывает действие из предыдущего действия хранимой процедуры. Предположим, что целевым объектом является **маркетинговая** таблица с тремя столбцами: **ProfileID**, **State** и **Category**. Выполните Upsert на основе столбца **ProfileID** .

```sql
CREATE PROCEDURE [dbo].[spMergeData]
AS
BEGIN
    MERGE TargetTable AS target
    USING UpsertStagingTable AS source
    ON (target.[ProfileID] = source.[ProfileID])
    WHEN MATCHED THEN
        UPDATE SET State = source.State
    WHEN NOT matched THEN
        INSERT ([ProfileID], [State], [Category])
      VALUES (source.ProfileID, source.State, source.Category);
    
    TRUNCATE TABLE UpsertStagingTable
END
```

**Вариант 2.** Можно выбрать [вызов хранимой процедуры в рамках действия копирования](#invoke-a-stored-procedure-from-a-sql-sink). Этот подход запускает каждый пакет (в соответствии со `writeBatchSize` свойством) в исходной таблице, а не использует инструкцию INSERT в качестве метода по умолчанию в действии копирования.

### <a name="overwrite-the-entire-table"></a>Перезаписать всю таблицу

Свойство **preCopyScript** можно настроить в приемнике действия копирования. В этом случае для каждого выполняемого действия копирования фабрика данных Azure сначала запускает сценарий. Затем выполняется копирование для вставки данных. Например, чтобы перезаписать всю таблицу последними данными, укажите скрипт, чтобы сначала удалить все записи, прежде чем выполнять пакетную загрузку новых данных из источника.

### <a name="write-data-with-custom-logic"></a>Запись данных с помощью пользовательской логики

Действия по записи данных с помощью пользовательской логики похожи на процедуры, описанные в разделе [данных Upsert](#upsert-data) . Если необходимо применить дополнительные операции перед окончательной вставкой исходных данных в целевую таблицу, можно загрузить в промежуточную таблицу, затем вызвать действие хранимой процедуры или вызвать хранимую процедуру в приемнике действия копирования, чтобы применить данные.

## <a name="invoke-a-stored-procedure-from-a-sql-sink"></a><a name="invoke-a-stored-procedure-from-a-sql-sink"></a> Вызов хранимой процедуры из приемника SQL

При копировании данных в базу данных SQL Server можно также настроить и вызвать пользовательскую хранимую процедуру с дополнительными параметрами для каждого пакета исходной таблицы. Функция хранимой процедуры использует возможности [возвращающих табличное значение параметров](/dotnet/framework/data/adonet/sql/table-valued-parameters).

Вы можете использовать хранимую процедуру, когда встроенные механизмы копирования не подходят. Примером может быть применение дополнительной обработки перед окончательной вставкой исходных данных в целевую таблицу. Некоторые дополнительные примеры обработки позволяют объединить столбцы, найти дополнительные значения и вставить их в несколько таблиц.

В следующем примере показано, как использовать хранимую процедуру для выполнения операции Upsert в таблице базы данных SQL Server. Предположим, что у входных данных и **маркетинговой** таблицы приемника есть три столбца: **ProfileID**, **State** и **Category**. Выполните Upsert на основе столбца **ProfileID** и примените его только к определенной категории с названием «Product а».

1. В базе данных определите тип таблицы с тем же именем, что и **и sqlwritertabletype**. Для типа таблицы укажите ту же схему, которая возвращается для входных данных.

    ```sql
    CREATE TYPE [dbo].[MarketingType] AS TABLE(
        [ProfileID] [varchar](256) NOT NULL,
        [State] [varchar](256) NOT NULL,
        [Category] [varchar](256) NOT NULL
    )
    ```

2. В базе данных Определите хранимую процедуру с тем же именем, что и **sqlWriterStoredProcedureName**. Она обрабатывает входные данные из указанного источника и выполняет их слияние в выходную таблицу. Имя параметра типа таблицы в хранимой процедуре совпадает с именем **TableName** , определенным в наборе данных.

    ```sql
    CREATE PROCEDURE spOverwriteMarketing @Marketing [dbo].[MarketingType] READONLY, @category varchar(256)
    AS
    BEGIN
    MERGE [dbo].[Marketing] AS target
    USING @Marketing AS source
    ON (target.ProfileID = source.ProfileID and target.Category = @category)
    WHEN MATCHED THEN
        UPDATE SET State = source.State
    WHEN NOT MATCHED THEN
        INSERT (ProfileID, State, Category)
        VALUES (source.ProfileID, source.State, source.Category);
    END
    ```

3. В фабрике данных Azure Определите раздел **SQL Sink** в действии копирования следующим образом:

    ```json
    "sink": {
        "type": "SqlSink",
        "sqlWriterStoredProcedureName": "spOverwriteMarketing",
        "storedProcedureTableTypeParameterName": "Marketing",
        "sqlWriterTableType": "MarketingType",
        "storedProcedureParameters": {
            "category": {
                "value": "ProductA"
            }
        }
    }
    ```

## <a name="data-type-mapping-for-sql-server"></a>Сопоставление типов данных для SQL Server

При копировании данных из и в SQL Server следующие сопоставления используются из SQL Server типов данных в промежуточные типы данных фабрики данных Azure. Дополнительные сведения о том, как действие копирования сопоставляет исходную схему и типы данных для приемника, см. в статье [Сопоставление схем в действии копирования](copy-activity-schema-and-type-mapping.md).

| Тип данных SQL Server | Промежуточный тип данных Фабрики данных Azure |
|:--- |:--- |
| BIGINT |Int64 |
| binary |Byte[] |
| bit |Логическое |
| char |String, Char[] |
| Дата |Дата и время |
| Datetime |Дата и время |
| datetime2 |Дата и время |
| Datetimeoffset |DateTimeOffset |
| Decimal |Decimal |
| FILESTREAM attribute (varbinary(max)) |Byte[] |
| Float |Double |
| Изображение |Byte[] |
| INT |Int32 |
| money |Decimal |
| nchar |String, Char[] |
| ntext |String, Char[] |
| NUMERIC |Decimal |
| nvarchar |String, Char[] |
| real |Один |
| rowversion |Byte[] |
| smalldatetime |Дата и время |
| smallint |Int16 |
| smallmoney |Decimal |
| sql_variant |Объект |
| текст |String, Char[] |
| time |TimeSpan |
| TIMESTAMP |Byte[] |
| tinyint |Int16 |
| UNIQUEIDENTIFIER |Guid |
| varbinary |Byte[] |
| varchar |String, Char[] |
| Xml |Строка |

>[!NOTE]
> Для типов данных, которые сопоставляются с промежуточным типом Decimal, в настоящее время действие копирования поддерживает точность до 28. Если для ваших данных требуется точность больше 28, попробуйте преобразовать их в строковые данные в SQL-запросе.

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Свойства действия GetMetadata

Подробные сведения об этих свойствах см. в разделе [Действие GetMetadata](control-flow-get-metadata-activity.md) 

## <a name="using-always-encrypted"></a>Использование Always Encrypted

При копировании данных из или в SQL Server с помощью [Always encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine)используйте [универсальный соединитель ODBC](connector-odbc.md) и SQL Server драйвер ODBC через локально размещенную Integration Runtime. Этот соединитель SQL Server не поддерживает Always Encrypted сейчас. 

В частности:

1. Настройте автономную Integration Runtime, если у вас ее нет. Дополнительные сведения см. в статье [Создание и настройка локальной среды выполнения интеграции](create-self-hosted-integration-runtime.md).

2. Скачайте 64-разрядный драйвер ODBC для SQL Server [отсюда](/sql/connect/odbc/download-odbc-driver-for-sql-server)и установите на компьютер Integration Runtime. Дополнительные сведения о том, как работает этот драйвер, см. в статье [использование Always encrypted с драйвером ODBC для SQL Server](/sql/connect/odbc/using-always-encrypted-with-the-odbc-driver#using-the-azure-key-vault-provider).

3. Создайте связанную службу с типом ODBC для подключения к базе данных SQL. Чтобы использовать проверку подлинности SQL, укажите строку подключения ODBC, как показано ниже, и выберите **Обычная** проверка подлинности, чтобы задать имя пользователя и пароль.

    ```
    Driver={ODBC Driver 17 for SQL Server};Server=<serverName>;Database=<databaseName>;ColumnEncryption=Enabled;KeyStoreAuthentication=KeyVaultClientSecret;KeyStorePrincipalId=<servicePrincipalKey>;KeyStoreSecret=<servicePrincipalKey>
    ```

4. Создайте набор данных и действие копирования с соответствующим типом ODBC. Дополнительные сведения см. в статье о [соединителе ODBC](connector-odbc.md) .

## <a name="troubleshoot-connection-issues"></a>Устранение неполадок с подключением

1. Настройте экземпляр SQL Server для приема удаленных подключений. Запустите **SQL Server Management Studio**, щелкните правой кнопкой мыши **сервер** и выберите пункт **Свойства**. В списке выберите **подключения** и установите флажок **Разрешить удаленные соединения с этим сервером** .

    ![Включение удаленных подключений](media/copy-data-to-from-sql-server/AllowRemoteConnections.png)

    Подробные инструкции см. [в разделе Настройка параметра конфигурации сервера Remote Access](/sql/database-engine/configure-windows/configure-the-remote-access-server-configuration-option).

2. Запустите **Диспетчер конфигурации SQL Server**. Разверните узел **Сетевая конфигурация SQL Server** для нужного экземпляра и выберите пункт **Protocols for MSSQLSERVER** (Протоколы для MSSQLSERVER). Протоколы отображаются на правой панели. Включите TCP/IP, щелкнув правой кнопкой мыши **TCP/IP** и выбрав **включить**.

    ![Включение TCP/IP](./media/copy-data-to-from-sql-server/EnableTCPProptocol.png)

    Дополнительные сведения и альтернативные способы включения протокола TCP/IP см. в разделе [Включение или отключение сетевого протокола сервера](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol).

3. В том же окне дважды щелкните **TCP/IP** , чтобы открыть окно **свойств TCP/IP** .
4. Перейдите на вкладку **IP-адреса** . Прокрутите вниз, чтобы увидеть раздел **IPAll** . Запишите **TCP-порт**. Значение по умолчанию — **1433**.
5. Создайте на компьютере **правило брандмауэра Windows** , чтобы разрешить входящий трафик через этот порт. 
6. **Проверить подключение**. чтобы подключиться к SQL Server с помощью полного имени, используйте SQL Server Management Studio с другого компьютера. Например, `"<machine>.<domain>.corp.<company>.com,1433"`.

## <a name="next-steps"></a>Дальнейшие действия
Список хранилищ данных, поддерживаемых действием копирования в фабрике данных Azure в качестве источников и приемников, см. в разделе [Supported Data Stores (поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats)).