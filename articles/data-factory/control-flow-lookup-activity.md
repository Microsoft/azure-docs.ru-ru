---
title: Действие поиска в фабрике данных Azure
description: Сведения об использовании действия поиска для поиска значений во внешнем источнике. На эти выходные данные можно затем ссылаться в последующих действиях.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: conceptual
ms.date: 02/25/2021
ms.openlocfilehash: 4b2fb49899b6a676520fe0912dd122dd72cce023
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101712918"
---
# <a name="lookup-activity-in-azure-data-factory"></a>Действие поиска в фабрике данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Действие поиска можно использовать для получения набора данных из любых источников данных, поддерживаемых Фабрикой данных Azure. его можно использовать для динамического определения объектов, которые должны работать в последующем действии, вместо того, чтобы жестко программировать имя объекта. Примерами объектов могут быть файлы и таблицы.

Действие поиска считывает и возвращает содержимое файла конфигурации или таблицы. Оно также возвращает результат выполнения запроса или хранимой процедуры. Выходными данными может быть одноэлементное значение или массив атрибутов, которые могут быть использованы в последующих операциях копирования, преобразования или потока управления, таких как действие ForEach.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Следует отметить следующее.

- Действие поиска может возвращать до **5000 строк**. Если результирующий набор содержит больше записей, будут возвращены первые 5000 строк.
- Выходные данные действия поиска поддерживают до **4 МБ** памяти. Если размер превышает установленный предел, действие завершится ошибкой. 
- Самая длинная длительность действия уточняющего запроса до истечения времени ожидания составляет **24 часа**.
- При использовании запроса или хранимой процедуры для поиска данных убедитесь, что возвращен один и точный результирующий набор. В противном случае действие поиска завершается ошибкой.

Для действия поиска поддерживаются следующие источники данных. 

[!INCLUDE [data-factory-v2-supported-data-stores](../../includes/data-factory-v2-supported-data-stores-for-lookup-activity.md)]

## <a name="syntax"></a>Синтаксис

```json
{
    "name":"LookupActivity",
    "type":"Lookup",
    "typeProperties":{
        "source":{
            "type":"<source type>"
        },
        "dataset":{
            "referenceName":"<source dataset name>",
            "type":"DatasetReference"
        },
        "firstRowOnly":<true or false>
    }
}
```

## <a name="type-properties"></a>Свойства типа

Имя | Описание | Тип | Обязательно?
---- | ----------- | ---- | --------
набор данных | Предоставляет ссылку на набор данных для поиска. Дополнительные сведения можно найти в разделе **Свойства набора данных** в каждой соответствующей статье о соединителе. | Пара "ключ — значение" | Да
source | Содержит свойства источника конкретного набора данных, аналогичные источнику действия копирования. Дополнительные сведения можно найти в разделе **Свойства действия копирования** в каждой соответствующей статье о соединителе. | Пара "ключ — значение" | Да
firstRowOnly | Указывает, следует ли возвращать только первую или все строки. | Логическое | Нет. Значение по умолчанию — `true`.

> [!NOTE]
> 
> * Исходные столбцы с типом **ByteArray** не поддерживаются.
> * **Структура** не поддерживается в определениях набора данных. Для текстовых файлов форматирования можно использовать строку заголовка, чтобы указать имя столбца.
> * Если ваш источник поиска — файл JSON, параметр `jsonPathDefinition` для изменения объекта JSON не поддерживается. Будут извлечены целые объекты.

## <a name="use-the-lookup-activity-result"></a>Использование результата действия уточняющего запроса

Результат поиска возвращается в раздел `output` результатов выполнения действия.

* **Когда для `firstRowOnly` задано значение `true` (по умолчанию)** , формат выходного значения выглядит, как показано в следующем коде. Результат поиска находится в предопределенном ключе `firstRow`. Чтобы использовать результат в последующем действии, используйте шаблон  `@{activity('LookupActivity').output.firstRow.table}` .

    ```json
    {
        "firstRow":
        {
            "Id": "1",
            "schema":"dbo",
            "table":"Table1"
        }
    }
    ```

* **Когда для `firstRowOnly` задано значение `false`** , формат выходного значения выглядит, как показано в следующем коде. В поле `count` указано количество возвращенных записей. Подробные значения отображаются в списке предопределенного массива `value`. В этом случае за действием поиска следует [действие Foreach](control-flow-for-each-activity.md). Укажите массив `value` в поле действия ForEach `items` с помощью шаблона `@activity('MyLookupActivity').output.value`. Для получения доступа к элементам массива `value` используйте следующий синтаксис: `@{activity('lookupActivity').output.value[zero based index].propertyname}`. Например, `@{activity('lookupActivity').output.value[0].schema}`.

    ```json
    {
        "count": "2",
        "value": [
            {
                "Id": "1",
                "schema":"dbo",
                "table":"Table1"
            },
            {
                "Id": "2",
                "schema":"dbo",
                "table":"Table2"
            }
        ]
    } 
    ```

## <a name="example"></a>Пример

В этом примере конвейер содержит два действия: **Поиск** и **копирование**. Действие копирования копирует данные из таблицы SQL в экземпляре базы данных SQL Azure в хранилище BLOB-объектов Azure. Имя таблицы SQL хранится в файле JSON в хранилище BLOB-объектов. Действие поиска ищет имя таблицы во время выполнения. Такой подход позволяет динамически изменять файл JSON. При этом нет необходимости в повторном развертывании конвейеров или наборов данных. 

В этом примере демонстрируется поиск только для первой строки. Примеры выполнения поиска всех строк и связывания результатов с действием ForEach см. в статье [Копирование нескольких таблиц в пакетном режиме с помощью фабрики данных Azure](tutorial-bulk-copy.md).


### <a name="pipeline"></a>Pipeline

- Действие поиска настроено для использования **LookupDataset**, который ссылается на расположение в хранилище BLOB-объектов Azure. Действие поиска считывает имя таблицы SQL из файла JSON в этом расположении. 
- Действие копирования использует выходные данные действия уточняющего запроса, то есть имя таблицы SQL. Свойство **tableName** в наборе данных **SourceDataset** настроено для использования выходных данных действия поиска. Действие копирования копирует данные из таблицы SQL в расположение в хранилище BLOB-объектов Azure. Расположение задается свойством **SinkDataset**. 

```json
{
    "name": "LookupPipelineDemo",
    "properties": {
        "activities": [
            {
                "name": "LookupActivity",
                "type": "Lookup",
                "dependsOn": [],
                "policy": {
                    "timeout": "7.00:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30,
                    "secureOutput": false,
                    "secureInput": false
                },
                "userProperties": [],
                "typeProperties": {
                    "source": {
                        "type": "JsonSource",
                        "storeSettings": {
                            "type": "AzureBlobStorageReadSettings",
                            "recursive": true
                        },
                        "formatSettings": {
                            "type": "JsonReadSettings"
                        }
                    },
                    "dataset": {
                        "referenceName": "LookupDataset",
                        "type": "DatasetReference"
                    },
                    "firstRowOnly": true
                }
            },
            {
                "name": "CopyActivity",
                "type": "Copy",
                "dependsOn": [
                    {
                        "activity": "LookupActivity",
                        "dependencyConditions": [
                            "Succeeded"
                        ]
                    }
                ],
                "policy": {
                    "timeout": "7.00:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30,
                    "secureOutput": false,
                    "secureInput": false
                },
                "userProperties": [],
                "typeProperties": {
                    "source": {
                        "type": "AzureSqlSource",
                        "sqlReaderQuery": {
                            "value": "select * from [@{activity('LookupActivity').output.firstRow.schema}].[@{activity('LookupActivity').output.firstRow.table}]",
                            "type": "Expression"
                        },
                        "queryTimeout": "02:00:00",
                        "partitionOption": "None"
                    },
                    "sink": {
                        "type": "DelimitedTextSink",
                        "storeSettings": {
                            "type": "AzureBlobStorageWriteSettings"
                        },
                        "formatSettings": {
                            "type": "DelimitedTextWriteSettings",
                            "quoteAllText": true,
                            "fileExtension": ".txt"
                        }
                    },
                    "enableStaging": false,
                    "translator": {
                        "type": "TabularTranslator",
                        "typeConversion": true,
                        "typeConversionSettings": {
                            "allowDataTruncation": true,
                            "treatBooleanAsNumber": false
                        }
                    }
                },
                "inputs": [
                    {
                        "referenceName": "SourceDataset",
                        "type": "DatasetReference",
                        "parameters": {
                            "schemaName": {
                                "value": "@activity('LookupActivity').output.firstRow.schema",
                                "type": "Expression"
                            },
                            "tableName": {
                                "value": "@activity('LookupActivity').output.firstRow.table",
                                "type": "Expression"
                            }
                        }
                    }
                ],
                "outputs": [
                    {
                        "referenceName": "SinkDataset",
                        "type": "DatasetReference",
                        "parameters": {
                            "schema": {
                                "value": "@activity('LookupActivity').output.firstRow.schema",
                                "type": "Expression"
                            },
                            "table": {
                                "value": "@activity('LookupActivity').output.firstRow.table",
                                "type": "Expression"
                            }
                        }
                    }
                ]
            }
        ],
        "annotations": [],
        "lastPublishTime": "2020-08-17T10:48:25Z"
    }
}
```

### <a name="lookup-dataset"></a>Набор данных поиска

Набор данных **подстановки** — это **sourcetable.js** файла в папке поиска службы хранилища Azure, заданной типом **азуреблобсторажелинкедсервице** . 

```json
{
    "name": "LookupDataset",
    "properties": {
        "linkedServiceName": {
            "referenceName": "AzureBlobStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "annotations": [],
        "type": "Json",
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "fileName": "sourcetable.json",
                "container": "lookup"
            }
        }
    }
}
```

### <a name="source-dataset-for-copy-activity"></a>**Исходный** набор данных для действия копирования

**Исходный** набор данных использует выходные данные действия поиска — имя таблицы SQL. Действие копирования копирует данные из этой таблицы SQL в расположение в хранилище BLOB-объектов Azure. Расположение задается набором данных **приемника**. 

```json
{
    "name": "SourceDataset",
    "properties": {
        "linkedServiceName": {
            "referenceName": "AzureSqlDatabase",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "schemaName": {
                "type": "string"
            },
            "tableName": {
                "type": "string"
            }
        },
        "annotations": [],
        "type": "AzureSqlTable",
        "schema": [],
        "typeProperties": {
            "schema": {
                "value": "@dataset().schemaName",
                "type": "Expression"
            },
            "table": {
                "value": "@dataset().tableName",
                "type": "Expression"
            }
        }
    }
}
```

### <a name="sink-dataset-for-copy-activity"></a>Набор данных **приемника** для действия копирования

Действие копирования копирует данные из таблицы SQL в файл **filebylookup.csv** в папке **csv** в хранилище Azure. Файл задается свойством **азуреблобсторажелинкедсервице** . 

```json
{
    "name": "SinkDataset",
    "properties": {
        "linkedServiceName": {
            "referenceName": "AzureBlobStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "schema": {
                "type": "string"
            },
            "table": {
                "type": "string"
            }
        },
        "annotations": [],
        "type": "DelimitedText",
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "fileName": {
                    "value": "@{dataset().schema}_@{dataset().table}.csv",
                    "type": "Expression"
                },
                "container": "csv"
            },
            "columnDelimiter": ",",
            "escapeChar": "\\",
            "quoteChar": "\""
        },
        "schema": []
    }
}
```

### <a name="sourcetablejson"></a>sourcetable.JSON

Для **sourcetable.jsв** файле можно использовать два вида форматов.

#### <a name="set-of-objects"></a>Набор объектов

```json
{
   "Id":"1",
   "schema":"dbo",
   "table":"Table1"
}
{
   "Id":"2",
   "schema":"dbo",
   "table":"Table2"
}
```

#### <a name="array-of-objects"></a>Массив объектов

```json
[ 
    {
        "Id": "1",
        "schema":"dbo",
        "table":"Table1"
    },
    {
        "Id": "2",
        "schema":"dbo",
        "table":"Table2"
    }
]
```

## <a name="limitations-and-workarounds"></a>Ограничения и обходные решения

Ниже описаны некоторые ограничения для действия Lookup и рекомендуемые обходные решения.

| Ограничение | Обходной путь |
|---|---|
| Действие уточняющего запроса содержит не более 5 000 строк и максимальный размер в 4 МБ. | Разработайте двухуровневый конвейер, где внешний конвейер выполняет итерацию по внутреннему конвейеру, получаемые данные которого не превышают максимальное количество строк и размер. |
| | |

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь с другими действиями потока управления, которые поддерживаются фабрикой данных: 

- [Действие выполнения конвейера в фабрике данных Azure](control-flow-execute-pipeline-activity.md)
- [Действие ForEach](control-flow-for-each-activity.md)
- [Действие получения метаданных в Фабрике данных Azure](control-flow-get-metadata-activity.md)
- [Веб-действие](control-flow-web-activity.md)
