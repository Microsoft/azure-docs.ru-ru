---
title: Копирование данных из Teradata разных с помощью фабрики данных Azure
description: Соединитель Teradata службы фабрики данных позволяет копировать данные из Teradata разных в хранилища данных, поддерживаемые фабрикой данных в качестве приемников.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/22/2021
ms.author: jingwang
ms.openlocfilehash: c65b295dca16a3453b2ee5472b9a3ee5e8d62df7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100381772"
---
# <a name="copy-data-from-teradata-vantage-by-using-azure-data-factory"></a>Копирование данных из Teradata разных с помощью фабрики данных Azure

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
>
> * [Версия 1](v1/data-factory-onprem-teradata-connector.md)
> * [Текущая версия](connector-teradata.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описано, как с помощью действия копирования в фабрике данных Azure копировать данные из Teradata разных. Она основана на [обзоре действия копирования](copy-activity-overview.md).

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель Teradata поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

Данные из Teradata разных можно скопировать в любое поддерживаемое хранилище данных, которое является приемником. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, приведен в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

Этот соединитель Teradata поддерживает:

- Teradata **версии 14,10, 15,0, 15,10, 16,0, 16,10 и 16,20**.
- Копирование данных с помощью проверки подлинности **Basic**, **Windows** или **LDAP** .
- Параллельное копирование из источника Teradata. Дополнительные сведения см. в разделе [Параллельное копирование из Teradata](#parallel-copy-from-teradata) .

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

Если вы используете автономную Integration Runtime, обратите внимание на то, что она предоставляет встроенный драйвер Teradata, начиная с версии 3,18. Устанавливать драйверы вручную не требуется. Для драйвера требуется "Visual C++ распространяемого пакета обновления 4 для 2012" на компьютере с локальной средой выполнения интеграции. Если этот компонент не установлен, скачайте его [на этой странице](https://www.microsoft.com/en-sg/download/details.aspx?id=30679).

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, относящихся к соединителю Teradata.

## <a name="linked-service-properties"></a>Свойства связанной службы

Связанная служба Teradata поддерживает следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства Type необходимо задать значение **Teradata**. | Да |
| connectionString | Задает сведения, необходимые для подключения к экземпляру Teradata. Ознакомьтесь с приведенными ниже примерами.<br/>Можно также добавить пароль в Azure Key Vault и извлечь `password` конфигурацию из строки подключения. Дополнительные сведения см. в разделе [хранение учетных данных в Azure Key Vault](store-credentials-in-key-vault.md) . | Да |
| username | Укажите имя пользователя для подключения к Teradata. Применяется при использовании проверки подлинности Windows. | Нет |
| password | Укажите пароль для учетной записи пользователя, указанной для имени пользователя. Можно также указать ссылку на [секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). <br>Применяется при использовании проверки подлинности Windows или при создании ссылки на пароль в Key Vault для обычной проверки подлинности. | Нет |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Дополнительные сведения см. в разделе [Предварительные требования](#prerequisites). Если не указано другое, по умолчанию используется интегрированная среда выполнения Azure. |Нет |

Дополнительные свойства подключения, которые можно задать в строке подключения, в любом случае:

| Свойство. | Описание | Значение по умолчанию |
|:--- |:--- |:--- |
| тдмстпортнумбер | Номер порта, используемого для доступа к базе данных Teradata.<br>Не изменяйте это значение, если оно не указано в технической поддержке. | 1025 |
| уседатаенкриптион | Указывает, следует ли шифровать все связи с базой данных Teradata. Допустимые значения: 0 или 1.<br><br/>- **0 (отключено, по умолчанию)**: шифрует только сведения о проверке подлинности.<br/>- **1 (включено)**: шифрует все данные, передаваемые между драйвером и базой данных. | `0` |
| CharacterSet | Кодировка, используемая для сеанса. Например, `CharacterSet=UTF16` .<br><br/>Это значение может быть пользовательской кодировкой или одной из следующих предварительно определенных наборов символов: <br/>-ASCII<br/>-UTF8<br/>-UTF16<br/>— LATIN1252_0A<br/>— LATIN9_0A<br/>— LATIN1_0A<br/>-Shift-JIS (Windows, совместимые с DOS, KANJISJIS_0S)<br/>-EUC (совместимый с UNIX, KANJIEC_0U)<br/>— Мэйнфрейм IBM (KANJIEBCDIC5035_0I)<br/>— KANJI932_1S0<br/>-BIG5 (TCHBIG5_1R0)<br/>-GB (SCHGB2312_1T0)<br/>— SCHINESE936_6R0<br/>— TCHINESE950_8R0<br/>-Нетворккореан (HANGULKSC5601_2R4)<br/>— HANGUL949_7R0<br/>— ARABIC1256_6A0<br/>— CYRILLIC1251_2A0<br/>— HEBREW1255_5A0<br/>— LATIN1250_1A0<br/>— LATIN1254_7A0<br/>— LATIN1258_8A0<br/>— THAI874_4A0 | `ASCII` |
| максреспсизе |Максимальный размер буфера ответов для запросов SQL в килобайтах (KBs). Например, `MaxRespSize=‭10485760‬` .<br/><br/>Для базы данных Teradata версии 16,00 или более поздней максимальное значение равно 7361536. Для соединений, использующих более ранние версии, максимальное значение — 1048576. | `65536` |
| мечанисмнаме | Чтобы использовать протокол LDAP для проверки подлинности подключения, укажите `MechanismName=LDAP` . | Н/Д |

**Пример использования обычной проверки подлинности**

```json
{
    "name": "TeradataLinkedService",
    "properties": {
        "type": "Teradata",
        "typeProperties": {
            "connectionString": "DBCName=<server>;Uid=<username>;Pwd=<password>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Пример использования проверки подлинности Windows**

```json
{
    "name": "TeradataLinkedService",
    "properties": {
        "type": "Teradata",
        "typeProperties": {
            "connectionString": "DBCName=<server>",
            "username": "<username>",
            "password": "<password>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Пример использования проверки подлинности LDAP**

```json
{
    "name": "TeradataLinkedService",
    "properties": {
        "type": "Teradata",
        "typeProperties": {
            "connectionString": "DBCName=<server>;MechanismName=LDAP;Uid=<username>;Pwd=<password>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

> [!NOTE]
>
> Следующие полезные данные по-прежнему поддерживаются. Однако в дальнейшем следует использовать новый.

**Предыдущие полезные данные:**

```json
{
    "name": "TeradataLinkedService",
    "properties": {
        "type": "Teradata",
        "typeProperties": {
            "server": "<server>",
            "authenticationType": "<Basic/Windows>",
            "username": "<username>",
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

В этом разделе содержится список свойств, поддерживаемых набором данных Teradata. Полный список разделов и свойств, доступных для определения наборов данных, см. в разделе [Наборы данных в фабрике данных Azure](concepts-datasets-linked-services.md).

Для копирования данных из Teradata поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение `TeradataTable` . | Да |
| База данных | Имя экземпляра Teradata. | Нет (если свойство query указано в источнике действия) |
| table | Имя таблицы в экземпляре Teradata. | Нет (если свойство query указано в источнике действия) |

**Пример**.

```json
{
    "name": "TeradataDataset",
    "properties": {
        "type": "TeradataTable",
        "typeProperties": {},
        "schema": [],        
        "linkedServiceName": {
            "referenceName": "<Teradata linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

> [!NOTE]
>
> `RelationalTable` набор данных типа по-прежнему поддерживается. Однако рекомендуется использовать новый набор данных.

**Предыдущие полезные данные:**

```json
{
    "name": "TeradataDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<Teradata linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

В этом разделе содержится список свойств, поддерживаемых Teradata в качестве источника. Полный список разделов и свойств, доступных для определения действий, см. в разделе [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). 

### <a name="teradata-as-source"></a>Teradata в качестве источника

>[!TIP]
>Чтобы эффективно загружать данные из Teradata с помощью секционирования данных, дополнительные сведения см. в разделе " [Параллельное копирование из Teradata](#parallel-copy-from-teradata) ".

Чтобы скопировать данные из Teradata, в разделе **источник** действия копирования поддерживаются следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type источника действия копирования должно иметь значение `TeradataSource` . | Да |
| query | Используйте пользовательский SQL-запрос для чтения данных. Например, `"SELECT * FROM MyTable"`.<br>При включении секционированной нагрузки необходимо привязать все соответствующие встроенные параметры секции в запросе. Примеры см. в разделе [Параллельное копирование из Teradata](#parallel-copy-from-teradata) . | Нет (если таблица в наборе данных указана) |
| partitionOptions | Задает параметры секционирования данных, используемые для загрузки данных из Teradata. <br>Допустимые значения: **None** (по умолчанию), **hash** и **динамикранже**.<br>Если параметр секции включен (то есть не `None` ), степень параллелизма для параллельной загрузки данных из Teradata управляется [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) параметром действия копирования. | нет |
| partitionSettings | Позволяет указать группу параметров для секционирования данных. <br>Применяется, если параметр секционирования имеет значение, отличное от `None`. | нет |
| partitionColumnName | Укажите имя исходного столбца, который будет использоваться секцией диапазона или хэш-секцией для параллельного копирования. Если не указано, первичный индекс таблицы автоматически определяется и используется в качестве столбца секционирования. <br>Применяется, если параметр Partition имеет значение `Hash` или `DynamicRange` . При использовании запроса для получения исходных данных, обработчика `?AdfHashPartitionCondition` или  `?AdfRangePartitionColumnName` в предложении WHERE. См. пример в разделе [Параллельное копирование из Teradata](#parallel-copy-from-teradata) . | нет |
| partitionUpperBound | Максимальное значение столбца секционирования для копирования данных. <br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Если для получения исходных данных используется запрос, подключите `?AdfRangePartitionUpbound` в предложении WHERE. Пример см. в разделе " [Параллельное копирование из Teradata](#parallel-copy-from-teradata) ". | нет |
| partitionLowerBound | Минимальное значение столбца секционирования для копирования данных. <br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Если для получения исходных данных используется запрос, подключите `?AdfRangePartitionLowbound` в предложении WHERE. Пример см. в разделе " [Параллельное копирование из Teradata](#parallel-copy-from-teradata) ". | Нет |

> [!NOTE]
>
> `RelationalSource` Тип источника копирования по-прежнему поддерживается, но не поддерживает новую встроенную параллельную загрузку из Teradata (параметры секционирования). Однако рекомендуется использовать новый набор данных.

**Пример. копирование данных с помощью простого запроса без секции**

```json
"activities":[
    {
        "name": "CopyFromTeradata",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Teradata input dataset name>",
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
                "type": "TeradataSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="parallel-copy-from-teradata"></a>Параллельная копия из Teradata

Соединитель Teradata фабрики данных предоставляет встроенное секционирование данных для копирования данных из Teradata в параллельном режиме. Параметры секционирования данных можно найти в **исходной** таблице для действия копирования.

![Снимок экрана с параметрами секционирования](./media/connector-teradata/connector-teradata-partition-options.png)

При включении секционированной копии фабрика данных выполняет параллельные запросы к источнику Teradata для загрузки данных по секциям. Степень параллелизма определяется с помощью параметра [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) для действия копирования. Например, если установлено значение `parallelCopies` 4, фабрика данных одновременно создает и выполняет четыре запроса на основе указанного параметра секции и параметров, и каждый запрос извлекает часть данных из Teradata.

Рекомендуется включить параллельное копирование с секционированием данных, особенно при загрузке большого объема данных из Teradata. Ниже приведены рекомендуемые конфигурации для разных сценариев. Если копирование данных выполняется в файловое хранилище данных, мы рекомендуем сохранять данные в папку несколькими файлами (указывая только имя папки), так как производительность в таком случае будет выше, чем при записи в один файл.

| Сценарий                                                     | Предлагаемые параметры                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Полная загрузка из большой таблицы.                                   | **Параметр секции**: hash. <br><br/>Во время выполнения фабрика данных автоматически обнаруживает первичный Индексный столбец, применяет к нему хэш и копирует данные по секциям. |
| Для загрузки больших объемов данных используйте пользовательский запрос.                 | **Параметр секции**: hash.<br>**Запрос**: `SELECT * FROM <TABLENAME> WHERE ?AdfHashPartitionCondition AND <your_additional_where_clause>`.<br>**Столбец секционирования**: укажите столбец, используемый для применения хэш-секции. Если не указано, фабрика данных автоматически обнаруживает столбец PK таблицы, указанной в наборе данных Teradata.<br><br>Во время выполнения фабрика данных заменяет `?AdfHashPartitionCondition` логику хэш-секции и отправляется в Teradata. |
| Для загрузки большого объема данных используйте пользовательский запрос с целочисленным столбцом, значения которого распределены равномерно для поддержки секционирования по диапазонам. | **Partition options** (Параметры секционирования): динамический диапазон.<br>**Запрос**: `SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>`.<br>**Столбец секционирования**: укажите столбец для секционирования данных. Секционирование можно выполнять по столбцу с целочисленным типом данных.<br>**Partition upper bound** (Верхняя граница секции) и **Partition lower bound** (Нижняя граница секции): укажите эти значения, если нужно добавить фильтрацию по столбцу секционирования, чтобы получить данные только в пределах между нижним и верхним значениями.<br><br>Во время выполнения фабрика данных заменяет `?AdfRangePartitionColumnName` , `?AdfRangePartitionUpbound` и `?AdfRangePartitionLowbound` фактическим именем столбца и диапазонами значений для каждой секции и отправляется в Teradata. <br>Например, если указан столбец секционирования "ID" с нижней границей 1 и верхней границей 80 при этом для параллельного копирования указано значение 4, Фабрика данных будет извлекать данные по 4 секциям. Для них будут применены следующие диапазоны значений идентификаторов: [1, 20], [21, 40], [41, 60] и [61, 80]. |

**Пример: запрос с хэш-секцией**

```json
"source": {
    "type": "TeradataSource",
    "query": "SELECT * FROM <TABLENAME> WHERE ?AdfHashPartitionCondition AND <your_additional_where_clause>",
    "partitionOption": "Hash",
    "partitionSettings": {
        "partitionColumnName": "<hash_partition_column_name>"
    }
}
```

**Пример: запрос с секционированием по динамическому диапазону**

```json
"source": {
    "type": "TeradataSource",
    "query": "SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>",
    "partitionOption": "DynamicRange",
    "partitionSettings": {
        "partitionColumnName": "<dynamic_range_partition_column_name>",
        "partitionUpperBound": "<upper_value_of_partition_column>",
        "partitionLowerBound": "<lower_value_of_partition_column>"
    }
}
```

## <a name="data-type-mapping-for-teradata"></a>Сопоставление типов для Teradata

При копировании данных из Teradata применяются следующие сопоставления. Дополнительные сведения о том, как действие копирования сопоставляет исходную схему и типы данных для приемника, см. в статье [Сопоставление схем в действии копирования](copy-activity-schema-and-type-mapping.md).

| Тип данных Teradata | Тип промежуточных данных фабрики данных |
|:--- |:--- |
| BigInt |Int64 |
| BLOB-объект |Byte[] |
| Byte |Byte[] |
| Тип ByteInt |Int16 |
| CHAR |Строка |
| Clob |Строка |
| Дата |Дата и время |
| Decimal |Decimal |
| Double |Double |
| GRAPHIC |Не поддерживается. Применить явное приведение в исходном запросе. |
| Целое число |Int32 |
| Interval Day |Не поддерживается. Применить явное приведение в исходном запросе. |
| Interval Day To Hour |Не поддерживается. Применить явное приведение в исходном запросе. |
| Interval Day To Minute |Не поддерживается. Применить явное приведение в исходном запросе. |
| Interval Day To Second |Не поддерживается. Применить явное приведение в исходном запросе. |
| Interval Hour |Не поддерживается. Применить явное приведение в исходном запросе. |
| Interval Hour To Minute |Не поддерживается. Применить явное приведение в исходном запросе. |
| Interval Hour To Second |Не поддерживается. Применить явное приведение в исходном запросе. |
| Interval Minute |Не поддерживается. Применить явное приведение в исходном запросе. |
| Interval Minute To Second |Не поддерживается. Применить явное приведение в исходном запросе. |
| Interval Month |Не поддерживается. Применить явное приведение в исходном запросе. |
| Interval Second |Не поддерживается. Применить явное приведение в исходном запросе. |
| Interval Year |Не поддерживается. Применить явное приведение в исходном запросе. |
| Interval Year To Month |Не поддерживается. Применить явное приведение в исходном запросе. |
| Число |Double |
| Period (дата) |Не поддерживается. Применить явное приведение в исходном запросе. |
| Period (время) |Не поддерживается. Применить явное приведение в исходном запросе. |
| Period (время с часовым поясом) |Не поддерживается. Применить явное приведение в исходном запросе. |
| Period (Timestamp) |Не поддерживается. Применить явное приведение в исходном запросе. |
| Период (метка времени с часовым поясом) |Не поддерживается. Применить явное приведение в исходном запросе. |
| SmallInt |Int16 |
| Time |TimeSpan |
| Time With Time Zone |TimeSpan |
| Отметка времени |Дата и время |
| Timestamp With Time Zone |Дата/время |
| VarByte |Byte[] |
| VarChar |Строка |
| VarGraphic |Не поддерживается. Применить явное приведение в исходном запросе. |
| Xml |Не поддерживается. Применить явное приведение в исходном запросе. |


## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных.
