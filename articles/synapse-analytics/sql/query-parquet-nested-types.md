---
title: Запрос Parquet вложенных типов с помощью бессерверного пула SQL
description: Из этой статьи вы узнаете, как выполнять запросы к вложенным типам Parquet с помощью бессерверного пула SQL.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 2ef547bea9ba84665ec7187a5adf2f7a7d87e0a4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101666323"
---
# <a name="query-nested-types-in-parquet-and-json-files-by-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Запрос вложенных типов в файлах Parquet и JSON с помощью бессерверного пула SQL в Azure синапсе Analytics

В этой статье вы узнаете, как написать запрос с помощью бессерверного пула SQL в Azure синапсе Analytics. Запрос будет считывать вложенные типы Parquet.
Вложенные типы являются сложными структурами, представляющими объекты или массивы. Вложенные типы могут храниться в: 
- [Parquet](query-parquet-files.md), где можно иметь несколько сложных столбцов, содержащих массивы и объекты.
- Иерархические [JSON-файлы](query-json-files.md), в которых можно читать СЛОЖНЫЕ документы JSON в виде одного столбца.
- Azure Cosmos DB коллекции (в настоящее время находится в общедоступной предварительной версии), где каждый документ может содержать сложные вложенные свойства.

Несерверный пул SQL форматирует все вложенные типы как объекты JSON и массивы. Таким образом, можно [извлекать или изменять сложные объекты с помощью функций JSON](/sql/relational-databases/json/validate-query-and-change-json-data-with-built-in-functions-sql-server) или [анализировать данные JSON с помощью функции OPENJSON](/sql/relational-databases/json/convert-json-data-to-rows-and-columns-with-openjson-sql-server). 

Ниже приведен пример запроса, который извлекает скалярные значения и из [ковид-19 Open Research DataSet](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/) JSON, который содержит вложенные объекты: 

```sql
SELECT
    title = JSON_VALUE(doc, '$.metadata.title'),
    first_author = JSON_QUERY(doc, '$.metadata.authors[0]'),
    first_author_name = JSON_VALUE(doc, '$.metadata.authors[0].first'),
    complex_object = doc
FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/covid19temp/comm_use_subset/pdf_json/000b7d1517ceebb34e1e3e817695b6de03e2fa78.json',
        FORMAT='CSV', FIELDTERMINATOR ='0x0b', FIELDQUOTE = '0x0b', ROWTERMINATOR = '0x0b'
    )
    WITH ( doc varchar(MAX) ) AS docs;
```

`JSON_VALUE`Функция возвращает скалярное значение из поля по указанному пути. `JSON_QUERY`Функция возвращает объект, отформатированный в формате JSON, из поля по указанному пути.

> [!IMPORTANT]
> В этом примере используется файл из набора данных КОВИД-19 Open Research. [Ознакомьтесь с лицензиями и структурой данных здесь](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/).

## <a name="prerequisites"></a>Предварительные условия

Первым шагом является создание базы данных, в которой будет создан источник данных. Затем вы инициализируйте объекты, запустив [сценарий установки](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) в базе данных. Сценарий установки создаст источники данных, учетные данные области базы данных и форматы внешних файлов, используемые в примерах.

## <a name="project-nested-or-repeated-data"></a>Вложенные или повторяющиеся данные в проекте

Файл Parquet может иметь несколько столбцов со сложными типами. Значения из этих столбцов форматируются в виде текста JSON и возвращаются в виде столбцов типа VARCHAR. Следующий запрос считывает файл Структексампле. Parquet и показывает, как считывать значения вложенных столбцов: 

```sql
SELECT
    DateStruct, TimeStruct, TimestampStruct, DecimalStruct, FloatStruct
FROM
    OPENROWSET(
        BULK 'parquet/nested/structExample.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    )
    WITH (
        DateStruct VARCHAR(8000),
        TimeStruct VARCHAR(8000),
        TimestampStruct VARCHAR(8000),
        DecimalStruct VARCHAR(8000),
        FloatStruct VARCHAR(8000)
    ) AS [r];
```

Этот запрос возвращает следующий результат. Содержимое каждого вложенного объекта возвращается в виде текста JSON.

| датеструкт    | тиместрукт    | тиместампструкт   | деЦималструкт | флоатструкт |
| --- | --- | --- | --- | --- |
|{"Date": "2009-04-25"}| {"Time": "20:51:54.3598000"}|    {"Timestamp": "5501-04-08 12:13:57.4821000"}|    {"Decimal": 11143412.25350}| {"Float": 0,5}|
|{"Date": "1916-04-29"}| {"Time": "00:16:04.6778000"}|    {"Timestamp": "1990-06-30 20:50:52.6828000"}|    {"Decimal": 1963545.62800}|  {"Float":-2,125}|

Приведенный ниже запрос считывает файл justSimpleArray.parquet. Он проецирует все столбцы из файла Parquet, включая вложенные и повторяющиеся данные.

```sql
SELECT
    SimpleArray
FROM
    OPENROWSET(
        BULK 'parquet/nested/justSimpleArray.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) AS [r];
```

Этот запрос возвратит следующий результат:

|симплеаррай|
| --- |
|[11, 12, 13]|
|[21, 22, 23]|

## <a name="read-properties-from-nested-object-columns"></a>Чтение свойств из столбцов вложенных объектов

`JSON_VALUE`Функция позволяет возвращать значения из столбцов, отформатированных в виде текста JSON:

```sql
SELECT
    title = JSON_VALUE(complex_column, '$.metadata.title'),
    first_author_name = JSON_VALUE(complex_column, '$.metadata.authors[0].first'),
    body_text = JSON_VALUE(complex_column, '$.body_text.text'),
    complex_column
FROM
    OPENROWSET( BULK 'https://azureopendatastorage.blob.core.windows.net/covid19temp/comm_use_subset/pdf_json/000b7d1517ceebb34e1e3e817695b6de03e2fa78.json',
                FORMAT='CSV', FIELDTERMINATOR ='0x0b', FIELDQUOTE = '0x0b', ROWTERMINATOR = '0x0b' ) WITH ( complex_column varchar(MAX) ) AS docs;
```

Результат показан в следующей таблице:

|title  | first_author_name | body_text | complex_column |
| --- | --- | --- | --- |
| Дополнительная информация — епидемиоло... | Жюльен   | -Рис. S1: Филожени из... | `{    "paper_id": "000b7d1517ceebb34e1e3e817695b6de03e2fa78",    "metadata": {        "title": "Supplementary Information An eco-epidemiological study of Morbilli-related paramyxovirus infection in Madagascar bats reveals host-switching as the dominant macro-evolutionary mechanism",        "authors": [            {                "first": "Julien"` |

В отличие от JSON-файлов, которые в большинстве случаев возвращают один столбец, содержащий сложный объект JSON, Parquet-файлы могут иметь несколько сложных столбцов. Свойства вложенных столбцов можно считывать с помощью `JSON_VALUE` функции для каждого столбца. `OPENROWSET` позволяет непосредственно указывать пути вложенных свойств в `WITH` предложении. Можно задать пути в качестве имени столбца или добавить [выражение пути JSON](/sql/relational-databases/json/json-path-expressions-sql-server) после типа столбца.

Следующий запрос считывает файл Структексампле. Parquet и показывает, как Surface Elements вложенного столбца. Существует два способа ссылки на вложенное значение:
- Путем указания выражения пути вложенного значения после спецификации типа.
- Путем форматирования имени столбца как вложенного пути с помощью команды "." для ссылки на поля.

```sql
SELECT
    *
FROM
    OPENROWSET(
        BULK 'parquet/nested/structExample.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    )
    WITH (
        [DateValue] DATE '$.DateStruct.Date',
        [TimeStruct.Time] TIME,
        [TimestampStruct.Timestamp] DATETIME2,
        DecimalValue DECIMAL(18, 5) '$.DecimalStruct.Decimal',
        [FloatStruct.Float] FLOAT
    ) AS [r];
```

## <a name="access-elements-from-repeated-columns"></a>Доступ к элементам из повторяющихся столбцов

Следующий запрос считывает файл Жустсимплеаррай. Parquet и использует [JSON_VALUE](/sql/t-sql/functions/json-value-transact-sql?view=azure-sqldw-latest&preserve-view=true) для получения скалярного элемента из повторяющегося столбца, например массива или Map:

```sql
SELECT
    *,
    JSON_VALUE(SimpleArray, '$[0]') AS FirstElement,
    JSON_VALUE(SimpleArray, '$[1]') AS SecondElement,
    JSON_VALUE(SimpleArray, '$[2]') AS ThirdElement
FROM
    OPENROWSET(
        BULK 'parquet/nested/justSimpleArray.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) AS [r];
```

Ниже приведен результат:

|симплеаррай    | фирстелемент  | секонделемент | сирделемент |
| --- | --- | --- | --- |
| [11, 12, 13] | 11   | 12 | 13 |
| [21, 22, 23] | 21   | 22 | 23 |

## <a name="access-sub-objects-from-complex-columns"></a>Доступ к подобъектам из сложных столбцов

Следующий запрос считывает файл Мапексампле. Parquet и использует [JSON_QUERY](/sql/t-sql/functions/json-query-transact-sql?view=azure-sqldw-latest&preserve-view=true) для получения нескалярного элемента из повторяющегося столбца, например массива или Map:

```sql
SELECT
    MapOfPersons,
    JSON_QUERY(MapOfPersons, '$."John Doe"') AS [John]
FROM
    OPENROWSET(
        BULK 'parquet/nested/mapExample.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) AS [r];
```

Также можно явно ссылаться на столбцы, возвращаемые в `WITH` предложении:

```sql
SELECT DocId,
    MapOfPersons,
    JSON_QUERY(MapOfPersons, '$."John Doe"') AS [John]
FROM
    OPENROWSET(
        BULK 'parquet/nested/mapExample.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) 
    WITH (DocId bigint, MapOfPersons VARCHAR(max)) AS [r];
```

Структура `MapOfPersons` возвращается в виде СТОЛБЦА varchar и форматируется как строка JSON.

## <a name="project-values-from-repeated-columns"></a>Значения проекта из повторяющихся столбцов

Если имеется массив скалярных значений (например `[1,2,3]` ,) в некоторых столбцах, их можно легко развернуть и соединить с главной строкой с помощью этого скрипта:

```sql
SELECT
    SimpleArray, Element
FROM
    OPENROWSET(
        BULK 'parquet/nested/justSimpleArray.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) AS arrays
    CROSS APPLY OPENJSON (SimpleArray) WITH (Element int '$') as array_values
```

## <a name="next-steps"></a>Дальнейшие действия

В следующей статье будет показано, как [запрашивать файлы JSON](query-json-files.md).