---
title: Копирование данных из Netezza с помощью Фабрики данных Azure
description: Узнайте, как копировать данные из Netezza в поддерживаемые хранилища данных-приемники с помощью действия копирования в конвейере фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 05/28/2020
ms.author: jingwang
ms.openlocfilehash: 4b12c1f24f389634004de3d487a693bc588a7241
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100374360"
---
# <a name="copy-data-from-netezza-by-using-azure-data-factory"></a>Копирование данных из Netezza с помощью Фабрики данных Azure
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Из этой статьи вы узнаете, как с помощью действия копирования в Фабрике данных Azure копировать данные из Netezza. Это продолжение статьи о [действии копирования в Фабрике данных Azure](copy-activity-overview.md), в которой представлены общие сведения о действии копирования.

>[!TIP]
>Дополнительные сведения о сценарии переноса данных из Netezza в Azure см. в статье [Использование Фабрики данных Azure для переноса данных с локального сервера Netezza в Azure](data-migration-guidance-netezza-azure-sqldw.md).

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Соединитель Netezza предназначен для поддержки следующих действий.

- [Действие копирования](copy-activity-overview.md) с использованием матрицы [поддерживаемых источников и приемников](copy-activity-overview.md).
- [Действие поиска](control-flow-lookup-activity.md)


Данные из Netezza можно скопировать в любое поддерживаемое хранилище данных, используемое в качестве приемника. Список хранилищ данных, поддерживаемых действием копирования в качестве источников и приемников, приведен в разделе [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

Соединитель Netezza поддерживает параллельное копирование из источника. Дополнительные сведения см. в разделе [Параллельное копирование из Netezza](#parallel-copy-from-netezza).

В Фабрике данных Azure предоставляется встроенный драйвер, который обеспечивает подключение. Не нужно вручную устанавливать драйвер для использования этого соединителя.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

## <a name="get-started"></a>Начало работы

Конвейер можно создать с помощью действия копирования и пакета SDK для .NET, пакета SDK для Python, Azure PowerShell, REST API либо шаблона Azure Resource Manager. Пошаговые инструкции по созданию конвейера с действием копирования см. в [руководстве по действию копирования](quickstart-create-data-factory-dot-net.md).

В разделах ниже приведены сведения о свойствах, которые используются для определения сущностей Фабрики данных, относящихся к соединителю Netezza.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы Netezza поддерживаются следующие свойства.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства **type** необходимо задать значение **Netezza**. | Да |
| connectionString | Строка для подключения к Netezza через интерфейс ODBC. <br/>Вы можете также поместить пароль в Azure Key Vault и извлечь конфигурацию `pwd` из строки подключения. Ознакомьтесь с приведенными ниже примерами и подробными сведениями в статье [Хранение учетных данных в Azure Key Vault](store-credentials-in-key-vault.md). | Да |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Дополнительные сведения см. в разделе [Предварительные условия](#prerequisites). Если не указано другое, по умолчанию используется интегрированная Azure Integration Runtime. |нет |

Типичная строка подключения — `Server=<server>;Port=<port>;Database=<database>;UID=<user name>;PWD=<password>`. В следующей таблице описаны дополнительные свойства, которые можно задать.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| SecurityLevel | Уровень безопасности, используемый драйвером для подключения к хранилищу данных. Драйвер поддерживает SSL-подключения с односторонней проверкой подлинности по протоколу SSL версии 3. <br>Например, `SecurityLevel=preferredSecured`. Поддерживаются значения:<br/>- **Только незащищенный** (**onlyUnSecured**): драйвер не использует SSL.<br/>- **Предпочтительно незащищенный (preferredUnSecured) (по умолчанию)**: если сервер предоставляет возможность выбора, драйвер не использует SSL. <br/>- **Предпочитаемая защита (преферредсекуред)**: Если сервер предоставляет возможность выбора, драйвер использует SSL. <br/>- **Только защищенный (onlySecured)**: драйвер не подключается, если SSL-подключение недоступно. | нет |
| CaCertFile | Полный путь к SSL-сертификату, используемому сервером. Например, `CaCertFile=<cert path>;`.| Да, если протокол SSL включен |

**Пример**

```json
{
    "name": "NetezzaLinkedService",
    "properties": {
        "type": "Netezza",
        "typeProperties": {
            "connectionString": "Server=<server>;Port=<port>;Database=<database>;UID=<user name>;PWD=<password>"
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
    "name": "NetezzaLinkedService",
    "properties": {
        "type": "Netezza",
        "typeProperties": {
            "connectionString": "Server=<server>;Port=<port>;Database=<database>;UID=<user name>;",
            "pwd": { 
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

Этот раздел содержит список свойств, поддерживаемых набором данных Netezza.

Полный список разделов и свойств, используемых для определения наборов данных, приведен в статье [Наборы данных и связанные службы в фабрике данных Azure](concepts-datasets-linked-services.md).

Чтобы скопировать данные из Netezza, установите для свойства **type** набора данных значение **NetezzaTable**. Поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства type набора данных необходимо задать следующее значение: **NetezzaTable** | Да |
| схема | Имя схемы. |Нет (если свойство query указано в источнике действия)  |
| table | Имя таблицы. |Нет (если свойство query указано в источнике действия)  |
| tableName | Имя таблицы со схемой. Это свойство поддерживается только для обеспечения обратной совместимости. Для новых рабочих нагрузок используйте `schema` и `table`. | Нет (если свойство query указано в источнике действия) |

**Пример**

```json
{
    "name": "NetezzaDataset",
    "properties": {
        "type": "NetezzaTable",
        "linkedServiceName": {
            "referenceName": "<Netezza linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Этот раздел содержит список свойств, поддерживаемых источником Netezza.

Полный список разделов и свойств, доступных для определения действий, см. в статье, посвященной [конвейерам и действиям в Фабрике данных Azure](concepts-pipelines-activities.md).

### <a name="netezza-as-source"></a>Netezza в качестве источника

>[!TIP]
>Чтобы эффективно загружать данные из Netezza с использованием секционирования данных, изучите дополнительные сведения в разделе [Параллельное копирование из Netezza](#parallel-copy-from-netezza).

Чтобы копировать данные из Netezza, установите для типа **source** в действии копирования значение **NetezzaSource**. В разделе **source** действия копирования поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство **type** источника действия копирования должно иметь значение **NetezzaSource**. | Да |
| query | Используйте пользовательский SQL-запрос для чтения данных. Например, `"SELECT * FROM MyTable"`. | Нет (если для набора данных задано свойство tableName) |
| partitionOptions | Задает параметры секционирования данных, используемые для загрузки данных из Netezza. <br>Следующие значения являются допустимыми: **None** (по умолчанию), **DataSlice** или **DynamicRange**.<br>Если параметр секционирования включен (любое значение кроме `None`), степень параллелизма для параллельной загрузки данных из базы данных Netezza управляется параметром [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) в действии копирования. | нет |
| partitionSettings | Позволяет указать группу параметров для секционирования данных. <br>Применяется, если параметр секционирования имеет значение, отличное от `None`. | нет |
| partitionColumnName | Укажите имя исходного столбца **целочисленного типа**, который будет использоваться для секционирования по диапазонам при параллельном копировании. Если значение не указано, автоматически определяется первичный ключ таблицы и используется в качестве столбца секционирования. <br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Если для получения исходных данных используется запрос, подключите `?AdfRangePartitionColumnName` в предложении WHERE. Пример см. в разделе [Параллельное копирование из Netezza](#parallel-copy-from-netezza). | нет |
| partitionUpperBound | Максимальное значение столбца секционирования для копирования данных. <br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Если для получения исходных данных используется запрос, подключите `?AdfRangePartitionUpbound` в предложении WHERE. Пример можно найти в разделе [Параллельное копирование из Netezza](#parallel-copy-from-netezza). | нет |
| partitionLowerBound | Минимальное значение столбца секционирования для копирования данных. <br>Применяется, если параметр секционирования имеет значение `DynamicRange`. Если для получения исходных данных используется запрос, подключите `?AdfRangePartitionLowbound` в предложении WHERE. Пример можно найти в разделе [Параллельное копирование из Netezza](#parallel-copy-from-netezza). | нет |

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromNetezza",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Netezza input dataset name>",
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
                "type": "NetezzaSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="parallel-copy-from-netezza"></a>Параллельное копирование из Netezza

Соединитель Netezza для Фабрики данных предоставляет встроенную функцию секционирования данных для параллельного копирования из Netezza. Параметры секционирования данных можно найти в **исходной** таблице для действия копирования.

![Снимок экрана с параметрами секционирования](./media/connector-netezza/connector-netezza-partition-options.png)

Если включено копирование с секционированием, Фабрика данных выполняет параллельные запросы к источнику Netezza для загрузки данных по секциям. Степень параллелизма определяется с помощью параметра [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) для действия копирования. Например, если `parallelCopies` имеет значение 4, Фабрика данных будет создавать и выполнять четыре запроса с учетом указанного способа и параметров секционирования, где каждый запрос извлекает часть данных из базы данных Netezza.

Рекомендуется включить параллельное копирование с секционированием данных, особенно при загрузке большого объема данных из базы данных Netezza. Ниже приведены рекомендуемые конфигурации для разных сценариев. Если копирование данных выполняется в файловое хранилище данных, мы рекомендуем сохранять данные в папку несколькими файлами (указывая только имя папки), так как производительность в таком случае будет выше, чем при записи в один файл.

| Сценарий                                                     | Предлагаемые параметры                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Полная загрузка из большой таблицы.                                   | **Partition option** (Параметр секционирования): срез данных. <br><br/>Во время выполнения Фабрика данных автоматически секционирует данные по [встроенным срезам данных Netezza](https://www.ibm.com/support/knowledgecenter/en/SSULQD_7.2.1/com.ibm.nz.adm.doc/c_sysadm_data_slices_parts_disks.html) и копирует данные отдельно для каждой секции. |
| Для загрузки больших объемов данных используйте пользовательский запрос.                 | **Partition option** (Параметр секционирования): срез данных.<br>**Запрос**: `SELECT * FROM <TABLENAME> WHERE mod(datasliceid, ?AdfPartitionCount) = ?AdfDataSliceCondition AND <your_additional_where_clause>`.<br>Во время выполнения Фабрика данных заменяет `?AdfPartitionCount` (количеством параллельных операций копирования для действия копирования) и `?AdfDataSliceCondition` значениями из логики секционирования для среза данных, а затем отправляет эти значения в Netezza. |
| Для загрузки большого объема данных используйте пользовательский запрос с целочисленным столбцом, значения которого распределены равномерно для поддержки секционирования по диапазонам. | **Partition options** (Параметры секционирования): динамический диапазон.<br>**Запрос**: `SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>`.<br>**Столбец секционирования**: укажите столбец для секционирования данных. Секционирование можно выполнять по столбцу с целочисленным типом данных.<br>**Partition upper bound** (Верхняя граница секции) и **Partition lower bound** (Нижняя граница секции): укажите эти значения, если нужно добавить фильтрацию по столбцу секционирования, чтобы получить данные только в пределах между нижним и верхним значениями.<br><br>Во время выполнения Фабрика данных заменяет `?AdfRangePartitionColumnName`, `?AdfRangePartitionUpbound` и `?AdfRangePartitionLowbound` фактическими значениями именем столбца и диапазонами значений для каждой секции, а затем отправляет их в Netezza. <br>Например, если указан столбец секционирования "ID" с нижней границей 1 и верхней границей 80 при этом для параллельного копирования указано значение 4, Фабрика данных будет извлекать данные по 4 секциям. Для них будут применены следующие диапазоны значений идентификаторов: [1, 20], [21, 40], [41, 60] и [61, 80]. |

**Пример: запрос с секционированием по срезу данных**

```json
"source": {
    "type": "NetezzaSource",
    "query": "SELECT * FROM <TABLENAME> WHERE mod(datasliceid, ?AdfPartitionCount) = ?AdfDataSliceCondition AND <your_additional_where_clause>",
    "partitionOption": "DataSlice"
}
```

**Пример: запрос с секционированием по динамическому диапазону**

```json
"source": {
    "type": "NetezzaSource",
    "query": "SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>",
    "partitionOption": "DynamicRange",
    "partitionSettings": {
        "partitionColumnName": "<dynamic_range_partition_column_name>",
        "partitionUpperBound": "<upper_value_of_partition_column>",
        "partitionLowerBound": "<lower_value_of_partition_column>"
    }
}
```

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Подробные сведения об этих свойствах см. в разделе [Действие поиска](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Дальнейшие действия

В разделе [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats) приведен список хранилищ данных, которые поддерживаются в качестве источников и приемников для действия копирования в Фабрике данных Azure.
