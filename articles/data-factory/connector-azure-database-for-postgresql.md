---
title: Копирование и преобразование данных в базе данных Azure для PostgreSQL
description: Узнайте, как копировать и преобразовывать данные в базе данных Azure для PostgreSQL с помощью фабрики данных Azure.
ms.author: jingwang
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 02/25/2021
ms.openlocfilehash: ec4ea645e325ef48d4cb5951cd39fd4e9cbe1617
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101738061"
---
# <a name="copy-and-transform-data-in-azure-database-for-postgresql-by-using-azure-data-factory"></a>Копирование и преобразование данных в базе данных Azure для PostgreSQL с помощью фабрики данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описано, как с помощью действия копирования в фабрике данных Azure копировать данные из базы данных Azure для PostgreSQL и в нее, а затем использовать поток данных для преобразования данных в базу данных Azure для PostgreSQL. Дополнительные сведения о Фабрике данных Azure см. во [вводной статье](introduction.md).

Этот соединитель является специализированным для [службы "база данных Azure для PostgreSQL](../postgresql/overview.md)". Чтобы скопировать данные из универсальной базы данных PostgreSQL, расположенной в локальной среде или в облаке, используйте [соединитель PostgreSQL](connector-postgresql.md).

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель базы данных Azure для PostgreSQL поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с [поддерживаемой матрицей источника и приемника](copy-activity-overview.md)
- [Поток данных для сопоставления](concepts-data-flow-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

В настоящее время поток данных поддерживает один сервер базы данных Azure для PostgreSQL, но не гибкий сервер или масштабирование (Цитус).

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, относящихся к соединителю базы данных Azure для PostgreSQL.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы базы данных Azure для PostgreSQL поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства Type необходимо задать значение **азурепостгрескл**. | Да |
| connectionString | Строка подключения к базе данных Azure для PostgreSQL через интерфейс ODBC.<br/>Можно также добавить пароль в Azure Key Vault и извлечь `password` конфигурацию из строки подключения. Для получения дополнительных сведений см. следующие примеры и [Сохраните учетные данные в Azure Key Vault](store-credentials-in-key-vault.md) . | Да |
| connectVia | Это свойство представляет [среду выполнения интеграции](concepts-integration-runtime.md) , используемую для подключения к хранилищу данных. Вы можете использовать среду выполнения интеграции Azure или локальную среду IR (если хранилище данных расположено в частной сети). Если не указано другое, по умолчанию используется интегрированная среда выполнения Azure. |нет |

Типичная строка подключения — `Server=<server>.postgres.database.azure.com;Database=<database>;Port=<port>;UID=<username>;Password=<Password>`. Ниже приведены дополнительные свойства, которые можно задать в любом случае.

| Свойство. | Описание | Параметры | Обязательно |
|:--- |:--- |:--- |:--- |
| EncryptionMethod (EM)| Метод, используемый драйвером для шифрования данных, отправленных между драйвером и сервером базы данных. Например,  `EncryptionMethod=<0/1/6>;`| 0 (без шифрования) **(по умолчанию)** -1 (SSL) или 6 (RequestSSL) | Нет |
| ValidateServerCertificate (VSC) | Определяет, проверяет ли драйвер сертификат, отправленный сервером базы данных при включенном шифровании SSL (метод шифрования = 1). Например,  `ValidateServerCertificate=<0/1>;`| 0 (отключено) **(по умолчанию)** -1 (включено) | Нет |

**Пример**:

```json
{
    "name": "AzurePostgreSqlLinkedService",
    "properties": {
        "type": "AzurePostgreSql",
        "typeProperties": {
            "connectionString": "Server=<server>.postgres.database.azure.com;Database=<database>;Port=<port>;UID=<username>;Password=<Password>"
        }
    }
}
```

**Пример**:

***Хранение пароля в Azure Key Vault***

```json
{
    "name": "AzurePostgreSqlLinkedService",
    "properties": {
        "type": "AzurePostgreSql",
        "typeProperties": {
            "connectionString": "Server=<server>.postgres.database.azure.com;Database=<database>;Port=<port>;UID=<username>;",
            "password": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName>" 
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. [в статье DataSets in Azure Data фабрика](concepts-datasets-linked-services.md). В этом разделе содержится список свойств, которые служба "база данных Azure для PostgreSQL" поддерживает в DataSets.

Чтобы скопировать данные из базы данных Azure для PostgreSQL, задайте для свойства type набора данных значение **AzurePostgreSqlTable**. Поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение **азурепостгресклтабле** | Да |
| tableName | Имя таблицы | Нет (если свойство query указано в источнике действия) |

**Пример**:

```json
{
    "name": "AzurePostgreSqlDataset",
    "properties": {
        "type": "AzurePostgreSqlTable",
        "linkedServiceName": {
            "referenceName": "<AzurePostgreSql linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, доступных для определения действий, см. [в статье конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых источником базы данных Azure для PostgreSQL.

### <a name="azure-database-for-postgresql-as-source"></a>База данных Azure для PostgreSQL в качестве источника

Чтобы скопировать данные из базы данных Azure для PostgreSQL, задайте для типа источника в действии копирования значение **AzurePostgreSqlSource**. В разделе **source** действия копирования поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type источника действия копирования должно иметь значение **AzurePostgreSqlSource** . | Да |
| query | Используйте пользовательский SQL-запрос для чтения данных. Пример: `SELECT * FROM mytable` или `SELECT * FROM "MyTable"`. Примечание. в PostgreSQL имя сущности считается нечувствительным к регистру, если оно не заключено в кавычки. | Нет (если указано свойство tableName в наборе данных) |

**Пример**:

```json
"activities":[
    {
        "name": "CopyFromAzurePostgreSql",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<AzurePostgreSql input dataset name>",
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
                "type": "AzurePostgreSqlSource",
                "query": "<custom query e.g. SELECT * FROM mytable>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="azure-database-for-postgresql-as-sink"></a>База данных Azure для PostgreSQL в качестве приемника

Чтобы скопировать данные в базу данных Azure для PostgreSQL, в разделе **приемника** действия копирования поддерживаются следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type приемника действия копирования должно иметь значение **азурепостгресклсинк**. | Да |
| preCopyScript | Укажите SQL-запрос для действия копирования, который необходимо выполнить, прежде чем записывать данные в базу данных Azure для PostgreSQL в каждом запуске. Это свойство можно использовать для очистки предварительно загруженных данных. | Нет |
| вритемесод | Метод, используемый для записи данных в базу данных Azure для PostgreSQL.<br>Допустимые значения: **копикомманд** (Предварительная версия, которая более производительна), **BulkInsert** (по умолчанию). | Нет |
| writeBatchSize | Число строк, загруженных в базу данных Azure для PostgreSQL на пакет.<br>Допустимое значение — это целое число, представляющее количество строк. | Нет (значение по умолчанию — 1 000 000) |
| writeBatchTimeout | Время ожидания до выполнения операции пакетной вставки, пока не завершится срок ее действия.<br>Допустимые значения — строки TimeSpan. Например, 00:30:00 (30 минут). | Нет (значение по умолчанию — 00:30:00) |

**Пример**:

```json
"activities":[
    {
        "name": "CopyToAzureDatabaseForPostgreSQL",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Azure PostgreSQL output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzurePostgreSQLSink",
                "preCopyScript": "<custom SQL script>",
                "writeMethod": "CopyCommand",
                "writeBatchSize": 1000000
            }
        }
    }
]
```

## <a name="mapping-data-flow-properties"></a>Свойства потока данных для сопоставления

При преобразовании данных в потоке сопоставления данных можно выполнять чтение и запись в таблицы из базы данных Azure для PostgreSQL. Дополнительные сведения см. в описаниях [преобразования источника](data-flow-source.md) и [преобразования приемника](data-flow-sink.md) в разделе, посвященном потокам данных для сопоставления. Можно выбрать использование базы данных Azure для набора данных PostgreSQL или [встроенного набора данных](data-flow-source.md#inline-datasets) в качестве типа источника и приемника.

### <a name="source-transformation"></a>Преобразование источника

В таблице ниже перечислены свойства, поддерживаемые источником "база данных Azure для PostgreSQL". Эти свойства можно изменить на вкладке **Параметры источника** .

| Имя | Описание | Обязательно | Допустимые значения | Свойство сценария потока данных |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Таблица | Если выбрать вариант Таблица как входные данные, поток данных извлекает все данные из таблицы, указанной в наборе данных. | Нет | - |*(только для встроенного набора данных)*<br>tableName |
| Запрос | При выборе запроса в качестве входных данных укажите SQL-запрос для выборки данных из источника, переопределяющий любую таблицу, указанную в наборе данных. Использование запросов — отличный способ сократить количество строк для тестирования или поиска.<br><br>Предложение **ORDER BY** не поддерживается, но можно задать полную инструкцию SELECT FROM. Кроме того, можно использовать табличные функции, определяемые пользователем. **SELECT * FROM удфжетдата ()** — это определяемая пользователем функция в SQL, которая возвращает таблицу, которую можно использовать в потоке данных.<br>Пример запроса: `select * from mytable where customerId > 1000 and customerId < 2000` или `select * from "MyTable"` . Примечание. в PostgreSQL имя сущности считается нечувствительным к регистру, если оно не заключено в кавычки.| нет | Строка | query |
| Размер пакета | Укажите размер пакета для фрагментов больших данных в пакетах. | Нет | Целое число | batchSize |
| Уровень изоляции | Выберите один из следующих уровней изоляции:<br>— Зафиксировано чтение<br>-READ UNCOMMITTED (по умолчанию)<br>— Повторяемое чтение<br>— Сериализуемый<br>-None (пропускать уровень изоляции) | Нет | <small>READ_COMMITTED<br/>READ_UNCOMMITTED<br/>REPEATABLE_READ<br/>SERIALIZABLE<br/>NONE</small> |isolationLevel |

#### <a name="azure-database-for-postgresql-source-script-example"></a>Пример скрипта источника базы данных Azure для PostgreSQL

При использовании базы данных Azure для PostgreSQL в качестве типа источника связанным сценарием потока данных будет:

```
source(allowSchemaDrift: true,
    validateSchema: false,
    isolationLevel: 'READ_UNCOMMITTED',
    query: 'select * from mytable',
    format: 'query') ~> AzurePostgreSQLSource
```

### <a name="sink-transformation"></a>Преобразование приемника

В таблице ниже перечислены свойства, поддерживаемые приемником базы данных Azure для PostgreSQL. Эти свойства можно изменить на вкладке **параметры приемника** .

| Имя | Описание | Обязательно | Допустимые значения | Свойство сценария потока данных |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Update - метод | Укажите, какие операции разрешены в назначении базы данных. По умолчанию разрешены только операции вставки.<br>Для обновления, Upsert или удаления строк необходимо [Преобразование «изменение строки](data-flow-alter-row.md) » для добавления тегов в строки для этих действий. | Да | `true` или `false` | удаляемые <br/>Insertable <br/>обновляемые <br/>упсертабле |
| Ключевые столбцы | Для обновлений, операции Upsert и DELETE необходимо задать ключевые столбцы, чтобы определить, какая строка будет изменена.<br>Имя столбца, которое вы выбираете в качестве ключа, будет использоваться как часть последующего обновления, Upsert, DELETE. Поэтому необходимо выбрать столбец, существующий в сопоставлении приемника. | Нет | Array | ключи |
| Пропустить запись ключевых столбцов | Если вы не хотите записывать значение в ключевой столбец, выберите "пропустить запись ключевых столбцов". | Нет | `true` или `false` | скипкэйвритес |
| Действие таблицы |Определяет, следует ли повторно создавать или удалять все строки целевой таблицы перед записью.<br>- **Нет**: никаких действий над таблицей не выполняется.<br>- **Повторное создание**: таблица будет удалена и создана повторно. Это действие необходимо, если новая таблица создается динамически.<br>- **Усечение**: все строки из целевой таблицы будут удалены. | Нет | `true` или `false` | повторно создать<br/>truncate |
| Размер пакета | Укажите, сколько строк записывается в каждом пакете. Более крупные размеры пакетов улучшают сжатие и оптимизацию памяти, но при кэшировании данных возникает риск нехватки памяти. | Нет | Целое число | batchSize |
| Сценарии SQL pre и POST | Укажите многострочные скрипты SQL, которые будут выполняться до (Предварительная обработка) и после чего (после обработки) записываются в базу данных приемника. | нет | Строка | пресклс<br>постсклс |

#### <a name="azure-database-for-postgresql-sink-script-example"></a>Пример скрипта приемника базы данных Azure для PostgreSQL

При использовании базы данных Azure для PostgreSQL в качестве типа приемника связанным сценарием потока данных является:

```
IncomingStream sink(allowSchemaDrift: true,
    validateSchema: false,
    deletable:false,
    insertable:true,
    updateable:true,
    upsertable:true,
    keys:['keyColumn'],
    format: 'table',
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> AzurePostgreSQLSink
```

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Дополнительные сведения о свойствах см. [в разделе действие поиска в фабрике данных Azure](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Дальнейшие действия
Список хранилищ данных, поддерживаемых действием копирования в фабрике данных Azure в качестве источников и приемников, см. в разделе [Supported Data Stores (поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats)).
