---
title: Использование метаданных файлов в запросах
description: Функция OPENROWSET предоставляет сведения о файле и пути для каждого используемого в запросе файла, что позволяет фильтровать или анализировать данные по именам файлов и (или) путям к папкам.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: d24ae1f42c685589309506b2d5e0eab157b2bc42
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96299621"
---
# <a name="use-file-metadata-in-serverless-sql-pool-queries"></a>Использование метаданных файлов в несерверных запросах к пулу SQL

Несерверный пул SQL может обращаться к нескольким файлам и папкам, как описано в статье о [папках запросов и нескольких файлах](query-folders-multiple-csv-files.md) . Здесь вы узнаете, как использовать в запросах метаданные с именами файлов и папок.

Иногда бывает нужно узнать, какой исходный файл или папка соотносятся с определенной строкой результирующего набора.

Функции `filepath` и `filename` можно применить для получения имен файлов и (или) путей в результирующем наборе. Также вы можете применить их для фильтрации данных по именам файлов и (или) путям к папкам. Эти функции описаны в разделах, посвященных [функции filename](query-data-storage.md#filename-function) и [функции filepath](query-data-storage.md#filepath-function). В следующих разделах содержатся краткие описания и примеры.

## <a name="prerequisites"></a>Предварительные требования

Первым делом вам нужно **создать базу данных**, у которой источник данных ссылается на учетную запись хранения. Затем инициализируйте объекты, выполнив [скрипт настройки](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) для этой базы данных. Этот скрипт настройки создает источники данных, учетные данные области базы данных и форматы внешних файлов, которые используются в этих примерах.

## <a name="functions"></a>Функции

### <a name="filename"></a>Имя файла

Эта функция возвращает имя файла, из которого получена строка.

Следующий пример считывает файлы данных из набора NYC Yellow Taxi (данные о поездках в такси за три последних месяца 2017 года) и возвращает число поездок, внесенных в каждый файл. Сегмент OPENROWSET в запросе указывает, какие файлы будут считаны.

```sql
SELECT
    nyc.filename() AS [filename]
    ,COUNT_BIG(*) AS [rows]
FROM  
    OPENROWSET(
        BULK 'parquet/taxi/year=2017/month=9/*.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) nyc
GROUP BY nyc.filename();
```

Следующий пример демонстрирует применение функции *filename()* в предложении WHERE для фильтрации считываемых файлов. Он обращается ко всей папке в сегменте запроса OPENROWSET и фильтрует файлы в предложении WHERE.

Результаты будут такими же, как в предыдущем примере.

```sql
SELECT
    r.filename() AS [filename]
    ,COUNT_BIG(*) AS [rows]
FROM OPENROWSET(
    BULK 'csv/taxi/yellow_tripdata_2017-*.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV',
        PARSER_VERSION = '2.0',
        FIRSTROW = 2) 
        WITH (C1 varchar(200) ) AS [r]
WHERE
    r.filename() IN ('yellow_tripdata_2017-10.csv', 'yellow_tripdata_2017-11.csv', 'yellow_tripdata_2017-12.csv')
GROUP BY
    r.filename()
ORDER BY
    [filename];
```

### <a name="filepath"></a>Filepath

Функция filepath возвращает полный или частичный путь:

- При вызове без параметров она возвращает полный путь к файлу, из которого была получена строка. Если в инструкции OPENROWSET используется DATA_SOURCE, возвращается путь относительно DATA_SOURCE. 
- При вызове с параметром она возвращает ту часть пути, которая соответствует подстановочному знаку в той позиции, которую определяет этот параметр. Например, при значении параметра 1 возвращается часть пути, соответствующая первому подстановочному знаку.

Следующий пример считывает файлы данных из набора NYC Yellow Taxi (данные о поездках в такси за три последних месяца 2017 года). Он возвращает число поездок, сохраненных в каждом пути. Сегмент OPENROWSET в запросе указывает, какие файлы будут считаны.

```sql
SELECT
    r.filepath() AS filepath
    ,COUNT_BIG(*) AS [rows]
FROM OPENROWSET(
        BULK 'csv/taxi/yellow_tripdata_2017-1*.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV',
        PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        vendor_id INT
    ) AS [r]
GROUP BY
    r.filepath()
ORDER BY
    filepath;
```

Следующий пример демонстрирует применение функции *filepath()* в предложении WHERE для фильтрации считываемых файлов.

Вы можете указать подстановочные знаки в сегменте запроса OPENROWSET и отфильтровать файлы в предложении WHERE. Результаты будут такими же, как в предыдущем примере.

```sql
SELECT
    r.filepath() AS filepath
    ,r.filepath(1) AS [year]
    ,r.filepath(2) AS [month]
    ,COUNT_BIG(*) AS [rows]
FROM OPENROWSET(
        BULK 'csv/taxi/yellow_tripdata_*-*.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV',
        PARSER_VERSION = '2.0',        
        FIRSTROW = 2
    )
WITH (
    vendor_id INT
) AS [r]
WHERE
    r.filepath(1) IN ('2017')
    AND r.filepath(2) IN ('10', '11', '12')
GROUP BY
    r.filepath()
    ,r.filepath(1)
    ,r.filepath(2)
ORDER BY
    filepath;
```

## <a name="next-steps"></a>Дальнейшие действия

В следующей статье вы узнаете, как [выполнять запросы по файлам Parquet](query-parquet-files.md).
