---
title: Копирование и преобразование данных в базе данных Azure для MySQL
description: Получите сведения о копировании и преобразовании данных в базе данных Azure для MySQL с помощью фабрики данных Azure.
ms.author: jingwang
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 03/10/2021
ms.openlocfilehash: 4d13f6f435a21b467cae1b8e14211a001792787f
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103012615"
---
# <a name="copy-and-transform-data-in-azure-database-for-mysql-by-using-azure-data-factory"></a>Копирование и преобразование данных в базе данных Azure для MySQL с помощью фабрики данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure копировать данные из базы данных Azure для MySQL и в нее, а затем использовать поток данных для преобразования данных в базе данных Azure для MySQL. Дополнительные сведения о Фабрике данных Azure см. во [вводной статье](introduction.md).

Этот соединитель является специализированным для [службы базы данных Azure для MySQL](../mysql/overview.md). Чтобы скопировать данные из универсальной базы данных MySQL, расположенной в локальной среде или в облаке, используйте [соединитель MySQL](connector-mysql.md).

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель базы данных Azure для MySQL поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием матрицы [поддерживаемых источников и приемников](copy-activity-overview.md).
- [Поток данных для сопоставления](concepts-data-flow-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, характерных для соединителя базы данных Azure для MySQL.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы базы данных Azure для MySQL поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства type необходимо задать значение **AzureMySql**. | Да |
| connectionString | Укажите сведения, необходимые для подключения к экземпляру базы данных Azure для MySQL. <br/> Вы можете также поместить пароль в Azure Key Vault и извлечь конфигурацию `password` из строки подключения. Ознакомьтесь с приведенными ниже примерами и подробными сведениями в статье [Хранение учетных данных в Azure Key Vault](store-credentials-in-key-vault.md). | Да |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Вы можете использовать среду выполнения интеграции Azure или локальную среду IR (если хранилище данных расположено в частной сети). Если не указано другое, по умолчанию используется интегрированная среда выполнения Azure. |нет |

Типичная строка подключения — `Server=<server>.mysql.database.azure.com;Port=<port>;Database=<database>;UID=<username>;PWD=<password>`. Дополнительные свойства, которые вы можете установить в вашем случае:

| Свойство | Описание | Параметры | Обязательно |
|:--- |:--- |:--- |:--- |
| SSLMode | Этот параметр указывает, использует ли драйвер TLS-шифрование и проверку при подключении к MySQL. Пример: `SSLMode=<0/1/2/3/4>`| DISABLED (0) / PREFERRED (1) **(по умолчанию)** / REQUIRED (2) / VERIFY_CA (3) / VERIFY_IDENTITY (4) | нет |
| useSystemTrustStore | Этот параметр указывает, следует ли использовать сертификат ЦС из доверенного системного хранилища или из указанного PEM-файла. Пример: `UseSystemTrustStore=<0/1>;`| Enabled (1) / Disabled (0) **(по умолчанию)** | нет |

**Пример**.

```json
{
    "name": "AzureDatabaseForMySQLLinkedService",
    "properties": {
        "type": "AzureMySql",
        "typeProperties": {
            "connectionString": "Server=<server>.mysql.database.azure.com;Port=<port>;Database=<database>;UID=<username>;PWD=<password>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Пример: хранение пароля в Azure Key Vault**

```json
{
    "name": "AzureDatabaseForMySQLLinkedService",
    "properties": {
        "type": "AzureMySql",
        "typeProperties": {
            "connectionString": "Server=<server>.mysql.database.azure.com;Port=<port>;Database=<database>;UID=<username>;",
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

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). Этот раздел содержит список свойств, поддерживаемых набором данных базы данных Azure для MySQL.

Чтобы скопировать данные из базы данных Azure для MySQL, задайте для свойства type набора данных значение **AzureMySqlTable**. Поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство type набора данных должно иметь значение **AzureMySqlTable**. | Да |
| tableName | Имя таблицы в базе данных MySQL. | Нет (если свойство query указано в источнике действия) |

**Пример**

```json
{
    "name": "AzureMySQLDataset",
    "properties": {
        "type": "AzureMySqlTable",
        "linkedServiceName": {
            "referenceName": "<Azure MySQL linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "<table name>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых источником и приемником базы данных Azure для MySQL.

### <a name="azure-database-for-mysql-as-source"></a>База данных Azure для MySQL в качестве источника

Чтобы скопировать данные из базы данных Azure для MySQL, в разделе **источник** действия копирования поддерживаются следующие свойства.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство type источника действия копирования должно иметь значение **AzureMySqlSource**. | Да |
| query | Используйте пользовательский SQL-запрос для чтения данных. Например: `"SELECT * FROM MyTable"`. | Нет (если для набора данных задано свойство tableName) |
| куерикоммандтимеаут | Время ожидания до истечения времени ожидания запроса. Значение по умолчанию — 120 минут (02:00:00) | Нет |

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromAzureDatabaseForMySQL",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure MySQL input dataset name>",
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
                "type": "AzureMySqlSource",
                "query": "<custom query e.g. SELECT * FROM MyTable>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="azure-database-for-mysql-as-sink"></a>База данных Azure для MySQL в качестве приемника

Чтобы скопировать данные в базу данных Azure для MySQL, в разделе **приемника** действия копирования поддерживаются следующие свойства.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type приемника действия копирования должно иметь значение **азуремисклсинк** . | Да |
| preCopyScript | Укажите SQL-запрос для действия копирования, который будет выполнен перед записью данных в базу данных Azure для MySQL при каждом запуске. Это свойство можно использовать для очистки предварительно загруженных данных. | Нет |
| writeBatchSize | Вставляет данные в таблицу базы данных Azure для MySQL, когда размер буфера достигает writeBatchSize.<br>Допустимое значение — целое число, представляющее количество строк. | Нет (значение по умолчанию — 10 000) |
| writeBatchTimeout | Время ожидания до выполнения операции пакетной вставки, пока не завершится срок ее действия.<br>Допустимые значения: временной диапазон. Например, 00:30:00 (30 минут). | Нет (значение по умолчанию — 00:00:30) |

**Пример**.

```json
"activities":[
    {
        "name": "CopyToAzureDatabaseForMySQL",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Azure MySQL output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureMySqlSink",
                "preCopyScript": "<custom SQL script>",
                "writeBatchSize": 100000
            }
        }
    }
]
```

## <a name="mapping-data-flow-properties"></a>Свойства потока данных для сопоставления

При преобразовании данных в потоке сопоставления данных можно выполнять чтение и запись в таблицы из базы данных Azure для MySQL. Дополнительные сведения см. в описаниях [преобразования источника](data-flow-source.md) и [преобразования приемника](data-flow-sink.md) в разделе, посвященном потокам данных для сопоставления. Можно выбрать использование базы данных Azure для MySQL или [встроенного набора данных](data-flow-source.md#inline-datasets) в качестве типа источника и приемника.

### <a name="source-transformation"></a>Преобразование источника

В таблице ниже перечислены свойства, поддерживаемые источником базы данных Azure для MySQL. Эти свойства можно изменить на вкладке **Параметры источника** .

| Имя | Описание | Обязательно | Допустимые значения | Свойство сценария потока данных |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Таблица | Если выбрать вариант Таблица как входные данные, поток данных извлекает все данные из таблицы, указанной в наборе данных. | Нет | - |*(только для встроенного набора данных)*<br>tableName |
| Запрос | При выборе запроса в качестве входных данных укажите SQL-запрос для выборки данных из источника, переопределяющий любую таблицу, указанную в наборе данных. Использование запросов — отличный способ сократить количество строк для тестирования или поиска.<br><br>Предложение **ORDER BY** не поддерживается, но можно задать полную инструкцию SELECT FROM. Кроме того, можно использовать табличные функции, определяемые пользователем. **SELECT * FROM удфжетдата ()** — это определяемая пользователем функция в SQL, которая возвращает таблицу, которую можно использовать в потоке данных.<br>Пример запроса: `select * from mytable where customerId > 1000 and customerId < 2000` или `select * from "MyTable"` .| нет | Строка | query |
| Размер пакета | Укажите размер пакета для фрагментов больших данных в пакетах. | Нет | Целое число | batchSize |
| Уровень изоляции | Выберите один из следующих уровней изоляции:<br>— Зафиксировано чтение<br>-READ UNCOMMITTED (по умолчанию)<br>— Повторяемое чтение<br>— Сериализуемый<br>-None (пропускать уровень изоляции) | Нет | <small>READ_COMMITTED<br/>READ_UNCOMMITTED<br/>REPEATABLE_READ<br/>SERIALIZABLE<br/>NONE</small> |isolationLevel |

#### <a name="azure-database-for-mysql-source-script-example"></a>Пример исходного скрипта базы данных Azure для MySQL

При использовании базы данных Azure для MySQL в качестве типа источника связанным сценарием потока данных будет:

```
source(allowSchemaDrift: true,
    validateSchema: false,
    isolationLevel: 'READ_UNCOMMITTED',
    query: 'select * from mytable',
    format: 'query') ~> AzureMySQLSource
```

### <a name="sink-transformation"></a>Преобразование приемника

В таблице ниже перечислены свойства, поддерживаемые приемником базы данных Azure для MySQL. Эти свойства можно изменить на вкладке **параметры приемника** .

| Имя | Описание | Обязательно | Допустимые значения | Свойство сценария потока данных |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Update - метод | Укажите, какие операции разрешены в назначении базы данных. По умолчанию разрешены только операции вставки.<br>Для обновления, Upsert или удаления строк необходимо [Преобразование «изменение строки](data-flow-alter-row.md) » для добавления тегов в строки для этих действий. | Да | `true` или `false` | удаляемые <br/>Insertable <br/>обновляемые <br/>упсертабле |
| Ключевые столбцы | Для обновлений, операции Upsert и DELETE необходимо задать ключевые столбцы, чтобы определить, какая строка будет изменена.<br>Имя столбца, которое вы выбираете в качестве ключа, будет использоваться как часть последующего обновления, Upsert, DELETE. Поэтому необходимо выбрать столбец, существующий в сопоставлении приемника. | Нет | Array | ключи |
| Пропустить запись ключевых столбцов | Если вы не хотите записывать значение в ключевой столбец, выберите "пропустить запись ключевых столбцов". | Нет | `true` или `false` | скипкэйвритес |
| Действие таблицы |Определяет, следует ли повторно создавать или удалять все строки целевой таблицы перед записью.<br>- **Нет**: никаких действий над таблицей не выполняется.<br>- **Повторное создание**: таблица будет удалена и создана повторно. Это действие необходимо, если новая таблица создается динамически.<br>- **Усечение**: все строки из целевой таблицы будут удалены. | Нет | `true` или `false` | повторно создать<br/>truncate |
| Размер пакета | Укажите, сколько строк записывается в каждом пакете. Более крупные размеры пакетов улучшают сжатие и оптимизацию памяти, но при кэшировании данных возникает риск нехватки памяти. | Нет | Целое число | batchSize |
| Сценарии SQL pre и POST | Укажите многострочные скрипты SQL, которые будут выполняться до (Предварительная обработка) и после чего (после обработки) записываются в базу данных приемника. | нет | Строка | пресклс<br>постсклс |

#### <a name="azure-database-for-mysql-sink-script-example"></a>Пример скрипта приемника базы данных Azure для MySQL

При использовании базы данных Azure для MySQL в качестве типа приемника связанным сценарием потока данных является:

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
    skipDuplicateMapOutputs: true) ~> AzureMySQLSink
```

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).

## <a name="data-type-mapping-for-azure-database-for-mysql"></a>Сопоставление типов данных для базы данных Azure для MySQL

При копировании данных из базы данных Azure для MySQL для промежуточных типов данных фабрики данных Azure используются следующие сопоставления из типов данных MySQL. Дополнительные сведения о том, как действие копирования сопоставляет исходную схему и типы данных для приемника, см. в статье [Сопоставление схем в действии копирования](copy-activity-schema-and-type-mapping.md).

| Тип данных базы данных Azure для MySQL | Тип промежуточных данных фабрики данных |
|:--- |:--- |
| `bigint` |`Int64` |
| `bigint unsigned` |`Decimal` |
| `bit` |`Boolean` |
| `bit(M), M>1`|`Byte[]`|
| `blob` |`Byte[]` |
| `bool` |`Int16` |
| `char` |`String` |
| `date` |`Datetime` |
| `datetime` |`Datetime` |
| `decimal` |`Decimal, String` |
| `double` |`Double` |
| `double precision` |`Double` |
| `enum` |`String` |
| `float` |`Single` |
| `int` |`Int32` |
| `int unsigned` |`Int64`|
| `integer` |`Int32` |
| `integer unsigned` |`Int64` |
| `long varbinary` |`Byte[]` |
| `long varchar` |`String` |
| `longblob` |`Byte[]` |
| `longtext` |`String` |
| `mediumblob` |`Byte[]` |
| `mediumint` |`Int32` |
| `mediumint unsigned` |`Int64` |
| `mediumtext` |`String` |
| `numeric` |`Decimal` |
| `real` |`Double` |
| `set` |`String` |
| `smallint` |`Int16` |
| `smallint unsigned` |`Int32` |
| `text` |`String` |
| `time` |`TimeSpan` |
| `timestamp` |`Datetime` |
| `tinyblob` |`Byte[]` |
| `tinyint` |`Int16` |
| `tinyint unsigned` |`Int16` |
| `tinytext` |`String` |
| `varchar` |`String` |
| `year` |`Int32` |

## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.
