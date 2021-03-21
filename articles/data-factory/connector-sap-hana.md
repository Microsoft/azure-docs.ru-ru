---
title: Копирование данных из SAP HANA
description: Узнайте, как копировать данные из SAP HANA на поддерживаемые приемники хранилища данных с помощью действия копирования в конвейере фабрики данных Azure.
ms.author: jingwang
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 04/22/2020
ms.openlocfilehash: ce3c1e22dd030c0730bf4d9859591c00860908a7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100382282"
---
# <a name="copy-data-from-sap-hana-using-azure-data-factory"></a>Копирование данных из SAP HANA с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-sap-hana-connector.md)
> * [Текущая версия](connector-sap-hana.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure копировать данные из базы данных SAP HANA. Это продолжение [статьи об обзоре действия копирования](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

>[!TIP]
>Сведения о общей поддержке ADF в сценарии интеграции данных SAP см. в статье [Интеграция данных SAP с помощью фабрики данных Azure](https://github.com/Azure/Azure-DataFactory/blob/master/whitepaper/SAP%20Data%20Integration%20using%20Azure%20Data%20Factory.pdf) с подробным введением в каждый соединитель SAP, компарсион и рекомендации.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель SAP HANA поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с использованием [матрицы поддерживаемых источников и приемников](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

Данные из базы данных SAP HANA можно скопировать в любое хранилище данных, поддерживаемое в качестве приемника. Список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования, см. в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

В частности, этот соединитель SAP HANA поддерживает:

- Копирование данных из всех версий базы данных SAP HANA.
- Копирование данных из **информационных моделей Hana** (например, представлений аналитики и вычислений) и **таблиц строк и столбцов**.
- копирование данных с использованием **базовой** проверки подлинности или проверки подлинности **Windows**.
- Параллельное копирование из источника SAP HANA. Дополнительные сведения см. в разделе [Параллельное копирование из SAP HANA](#parallel-copy-from-sap-hana) .

> [!TIP]
> Чтобы скопировать данные **в** хранилище данных SAP HANA, используйте универсальный соединитель ODBC. Дополнительные сведения см. в разделе о [приемнике SAP HANA](#sap-hana-sink) . Обратите внимание, что связанные службы для соединителя SAP HANA и соединитель ODBC нельзя использовать повторно, так как они имеют разные типы.

## <a name="prerequisites"></a>Предварительные условия

Чтобы использовать этот соединитель SAP HANA, сделайте следующее:

- Настроить локальную среду выполнения интеграции. Дополнительные сведения см. в статье [Создание и настройка локальной среды выполнения интеграции](create-self-hosted-integration-runtime.md).
- Установите на компьютере среды выполнения интеграции драйвер ODBC SAP HANA. Драйвер ODBC для SAP HANA можно скачать на странице [SAP Software Download Center](https://support.sap.com/swdc) (Центр загрузки программного обеспечения SAP). Выполните поиск по ключевой фразе **SAP HANA CLIENT for Windows**.

## <a name="getting-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

В следующих разделах содержатся сведения о свойствах, которые используются для определения сущностей фабрики данных, относящихся к соединителю SAP HANA.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы SAP HANA поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства type необходимо задать значение **SapHana** | Да |
| connectionString | Укажите сведения, необходимые для подключения к SAP HANA с помощью **обычной проверки подлинности** или **проверки подлинности Windows**. Ознакомьтесь с приведенными ниже примерами.<br>В строке подключения параметр "сервер/порт" является обязательным (порт по умолчанию — 30015), а имя пользователя и пароль являются обязательными при использовании обычной проверки подлинности. Дополнительные дополнительные параметры см. в разделе [SAP HANA свойства подключения ODBC](<https://help.sap.com/viewer/0eec0d68141541d1b07893a39944924e/2.0.02/en-US/7cab593774474f2f8db335710b2f5c50.html>) .<br/>Можно также поместить пароль в Azure Key Vault и извлечь конфигурацию пароля из строки подключения. Дополнительные сведения см. в разделе [хранение учетных данных в Azure Key Vault](store-credentials-in-key-vault.md) статье. | Да |
| userName | Укажите имя пользователя при использовании проверки подлинности Windows. Например, `user@domain.com`. | Нет |
| password | Укажите пароль для учетной записи пользователя. Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [добавьте ссылку на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | нет |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Требуется локальная среда IR, как упоминалось в разделе [Предварительные требования](#prerequisites). |Да |

**Пример. использование обычной проверки подлинности**

```json
{
    "name": "SapHanaLinkedService",
    "properties": {
        "type": "SapHana",
        "typeProperties": {
            "connectionString": "SERVERNODE=<server>:<port (optional)>;UID=<userName>;PWD=<Password>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Пример. Использование проверки подлинности Windows**

```json
{
    "name": "SapHanaLinkedService",
    "properties": {
        "type": "SapHana",
        "typeProperties": {
            "connectionString": "SERVERNODE=<server>:<port (optional)>;",
            "userName": "<username>", 
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

Если вы использовали SAP HANA связанную службу со следующими полезными данными, она по-прежнему поддерживается "как есть", хотя вы предлагаете использовать новую пересылку.

**Пример**.

```json
{
    "name": "SapHanaLinkedService",
    "properties": {
        "type": "SapHana",
        "typeProperties": {
            "server": "<server>:<port (optional)>",
            "authenticationType": "Basic",
            "userName": "<username>",
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

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). В этом разделе содержится список свойств, поддерживаемых набором данных SAP HANA.

Чтобы скопировать данные из SAP HANA, поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение **сафанатабле** . | Да |
| схема | Имя схемы в базе данных SAP HANA. | Нет (если свойство query указано в источнике действия) |
| table | Имя таблицы в базе данных SAP HANA. | Нет (если свойство query указано в источнике действия) |

**Пример**.

```json
{
    "name": "SAPHANADataset",
    "properties": {
        "type": "SapHanaTable",
        "typeProperties": {
            "schema": "<schema name>",
            "table": "<table name>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<SAP HANA linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

Если вы ранее использовали типизированный набор данных `RelationalTable`, он пока поддерживается и не требует изменений, но мы рекомендуем при любом удобном случае перейти на новую версию.

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). Этот раздел содержит список свойств, поддерживаемых источником SAP HANA.

### <a name="sap-hana-as-source"></a>SAP HANA в качестве источника

>[!TIP]
>Чтобы эффективно принимать данные из SAP HANA с помощью секционирования данных, дополнительные сведения см. в разделе " [Параллельное копирование из SAP HANA](#parallel-copy-from-sap-hana) ".

Чтобы скопировать данные из SAP HANA, в разделе **источник** действия копирования поддерживаются следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type источника действия копирования должно иметь значение **сафанасаурце** . | Да |
| query | Указывает SQL-запрос для чтения данных из экземпляра SAP HANA. | Да |
| partitionOptions | Задает параметры секционирования данных, используемые для приема данных из SAP HANA. Дополнительные сведения см. в статье  [Параллельное копирование из SAP HANA](#parallel-copy-from-sap-hana) .<br>Допустимые значения: **None**   (по умолчанию), **фисикалпартитионсофтабле**, **сафанадинамикранже**. Дополнительные сведения см. в статье  [Параллельное копирование из SAP HANA](#parallel-copy-from-sap-hana) . `PhysicalPartitionsOfTable` может использоваться только при копировании данных из таблицы, но не с запроса. <br>Если параметр секции включен (то есть не `None` ), степень параллелизма для параллельной загрузки данных из SAP HANA управляется [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) параметром действия копирования. | Неверно |
| partitionSettings | Позволяет указать группу параметров для секционирования данных.<br>Применяется, если параметр секционирования имеет значение `SapHanaDynamicRange`. | Неверно |
| partitionColumnName | Укажите имя исходного столбца, который будет использоваться секцией для параллельного копирования. Если значение не указано, то индекс или первичный ключ таблицы будет обнаружен и использован в качестве столбца секционирования.<br>Применяется, если параметр Partition имеет значение  `SapHanaDynamicRange` . При использовании запроса для получения исходных данных присоединитесь к  `?AdfHanaDynamicRangePartitionCondition` предложению WHERE. См. пример в разделе [Параллельное копирование из SAP HANA](#parallel-copy-from-sap-hana) . | Да при использовании `SapHanaDynamicRange` секции. |
| packetSize | Указывает размер сетевого пакета (в килобайтах) для разбиения данных на несколько блоков. При наличии большого объема данных для копирования увеличение размера пакета может увеличить скорость чтения с SAP HANA в большинстве случаев. При изменении размера пакета рекомендуется использовать тестирование производительности. | Нет.<br>Значение по умолчанию — 2048 (2 МБ). |

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromSAPHANA",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SAP HANA input dataset name>",
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
                "type": "SapHanaSource",
                "query": "<SQL query for SAP HANA>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

Если вы использовали `RelationalSource` типизированный источник копирования, он по-прежнему поддерживается "как есть", хотя вы можете использовать новый вариант.

## <a name="parallel-copy-from-sap-hana"></a>Параллельное копирование из SAP HANA

Соединитель SAP HANA фабрики данных предоставляет встроенное секционирование данных для одновременного копирования данных из SAP HANA. Параметры секционирования данных можно найти в **исходной** таблице для действия копирования.

![Снимок экрана с параметрами секционирования](./media/connector-sap-hana/connector-sap-hana-partition-options.png)

При включении секционированной копии фабрика данных выполняет параллельные запросы к источнику SAP HANA для получения данных по секциям. Степень параллелизма определяется с помощью параметра [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) для действия копирования. Например, если установлено значение `parallelCopies` 4, фабрика данных одновременно создает и выполняет четыре запроса на основе указанного параметра секции и параметров, и каждый запрос извлекает часть данных из SAP HANA.

Рекомендуется включить параллельное копирование с секционированием данных, особенно при получении большого объема данных из SAP HANA. Ниже приведены рекомендуемые конфигурации для разных сценариев. При копировании данных в хранилище данных, основанное на файлах, рекомендуется записывать в папку несколько файлов (указывать только имя папки). в этом случае производительность лучше, чем при записи в один файл.

| Сценарий                                           | Предлагаемые параметры                                           |
| -------------------------------------------------- | ------------------------------------------------------------ |
| Полная загрузка из большой таблицы.                        | **Параметр секции**: физические разделы таблицы. <br><br/>Во время выполнения фабрика данных автоматически обнаруживает физический тип секции указанной SAP HANA таблицы и выбирает соответствующую стратегию секционирования:<br>- **Секционирование по диапазону**: Получение столбцов секционирования и диапазонов секций, определенных для таблицы, а затем копирование данных по диапазону. <br>- **Хэш-секционирование**: используйте ключ хэш-секции в качестве столбца секционирования, а затем разбейте и скопируйте данные на основе вычисленных диапазонов ADF. <br>- **Циклическое секционирование** или **без секции**: Используйте первичный ключ как столбец секционирования, затем разбейте и скопируйте данные на основе вычисленных диапазонов ADF. |
| Для загрузки больших объемов данных используйте пользовательский запрос. | **Параметр секции**: Динамическая секция диапазона.<br>**Запрос**: `SELECT * FROM <TABLENAME> WHERE ?AdfHanaDynamicRangePartitionCondition AND <your_additional_where_clause>`.<br>**Столбец секционирования**: укажите столбец, используемый для применения динамической секции диапазона. <br><br>Во время выполнения фабрика данных сначала вычисляет диапазоны значений указанного столбца секционирования, равномерно распределяя строки в ряде контейнеров в соответствии с количеством различных значений столбцов секционирования и параметром копирования в режиме параллелизма, а затем заменяя `?AdfHanaDynamicRangePartitionCondition` фильтром диапазон значений столбца секционирования для каждой секции и отправляет его в SAP HANA.<br><br>Если вы хотите использовать несколько столбцов в качестве столбца секционирования, можно объединить значения каждого столбца в один столбец в запросе и указать его в качестве столбца секционирования в ADF, например `SELECT * FROM (SELECT *, CONCAT(<KeyColumn1>, <KeyColumn2>) AS PARTITIONCOLUMN FROM <TABLENAME>) WHERE ?AdfHanaDynamicRangePartitionCondition` . |

**Пример: запрос с физическими секциями таблицы**

```json
"source": {
    "type": "SapHanaSource",
    "partitionOption": "PhysicalPartitionsOfTable"
}
```

**Пример: запрос с секционированием по динамическому диапазону**

```json
"source": {
    "type": "SapHanaSource",
    "query": "SELECT * FROM <TABLENAME> WHERE ?AdfHanaDynamicRangePartitionCondition AND <your_additional_where_clause>",
    "partitionOption": "SapHanaDynamicRange",
    "partitionSettings": {
        "partitionColumnName": "<Partition_column_name>"
    }
}
```

## <a name="data-type-mapping-for-sap-hana"></a>Сопоставление типов данных для SAP HANA

При копировании данных из SAP HANA используются следующие сопоставления типов данных SAP HANA с промежуточными типами данных фабрики данных Azure. Дополнительные сведения о том, как действие копирования сопоставляет исходную схему и типы данных для приемника, см. в статье [Сопоставление схем в действии копирования](copy-activity-schema-and-type-mapping.md).

| Тип данных SAP HANA | Тип промежуточных данных фабрики данных |
| ------------------ | ------------------------------ |
| ALPHANUM           | Строка                         |
| bigint             | Int64                          |
| BINARY             | Byte[]                         |
| бинтекст            | Строка                         |
| BLOB               | Byte[]                         |
| BOOL               | Byte                           |
| CLOB               | Строка                         |
| DATE               | Дата/время                       |
| DECIMAL            | Decimal                        |
| DOUBLE             | Double                         |
| FLOAT              | Double                         |
| INTEGER            | Int32                          |
| NCLOB              | Строка                         |
| NVARCHAR           | Строка                         |
| real               | Single                         |
| SECONDDATE         | Дата/время                       |
| шорттекст          | Строка                         |
| SMALLDECIMAL       | Decimal                        |
| SMALLINT           | Int16                          |
| STGEOMETRYTYPE     | Byte[]                         |
| стпоинттипе        | Byte[]                         |
| TEXT               | Строка                         |
| TIME               | TimeSpan                       |
| TINYINT            | Byte                           |
| VARCHAR            | Строка                         |
| timestamp          | Дата/время                       |
| VARBINARY          | Byte[]                         |

## <a name="sap-hana-sink"></a>Приемник SAP HANA

В настоящее время соединитель SAP HANA не поддерживается в качестве приемника, тогда как для записи данных в SAP HANA можно использовать универсальный соединитель ODBC с драйвером SAP HANA. 

Выполните [Предварительные требования](#prerequisites) для настройки автономного Integration Runtime и сначала установите SAP HANA драйвер ODBC. Создайте связанную службу ODBC для подключения к хранилищу данных SAP HANA, как показано в следующем примере, а затем создайте набор данных и приемник действия копирования с соответствующим типом ODBC. Дополнительные сведения см. в статье о [соединителе ODBC](connector-odbc.md) .

```json
{
    "name": "SAPHANAViaODBCLinkedService",
    "properties": {
        "type": "Odbc",
        "typeProperties": {
            "connectionString": "Driver={HDBODBC};servernode=<HANA server>.clouddatahub-int.net:30015",
            "authenticationType": "Basic",
            "userName": "<username>",
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

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Дальнейшие действия
В таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в фабрике данных Azure.
