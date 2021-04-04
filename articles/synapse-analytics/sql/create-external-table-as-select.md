---
title: Сохранение результатов запроса из бессерверного пула SQL
description: Из этой статьи вы узнаете, как сохранять результаты запросов в хранилище, используя бессерверный пул SQL.
services: synapse-analytics
author: vvasic-msft
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: vvasic
ms.reviewer: jrasnick
ms.openlocfilehash: 12841c747116cc9e14f348dfcf81acaa5da5e8c9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98165371"
---
# <a name="store-query-results-to-storage-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Сохранение результатов запросов в хранилище с помощью бессерверного пула SQL в Azure Synapse Analytics

Из этой статьи вы узнаете, как сохранять результаты запросов в хранилище, используя бессерверный пул SQL.

## <a name="prerequisites"></a>Предварительные требования

Для начала **создайте базу данных**, в которой будут выполняться запросы. Затем инициализируйте объекты, выполнив [скрипт настройки](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) для этой базы данных. Этот скрипт создает источники данных, учетные данные области базы данных и форматы внешних файлов, которые используются для чтения данных в этих примерах.

Придерживаясь инструкций в этой статье, создайте источники данных, учетные данные области базы данных и форматы внешних файлов, которые будут использоваться для записи данных в выходное хранилище.

## <a name="create-external-table-as-select"></a>CREATE EXTERNAL TABLE AS SELECT

Чтобы сохранить результаты запроса в хранилище, можно использовать инструкцию CREATE EXTERNAL TABLE AS SELECT (CETAS).

> [!NOTE]
> Измените первую строку в запросе ([mydbname]), чтобы использовать созданную вами базу данных.

```sql
USE [mydbname];
GO

CREATE DATABASE SCOPED CREDENTIAL [SasTokenWrite]
WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
     SECRET = 'sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-04-18T20:42:12Z&st=2019-04-18T12:42:12Z&spr=https&sig=lQHczNvrk1KoYLCpFdSsMANd0ef9BrIPBNJ3VYEIq78%3D';
GO

CREATE EXTERNAL DATA SOURCE [MyDataSource] WITH (
    LOCATION = 'https://<storage account name>.blob.core.windows.net/csv', CREDENTIAL = [SasTokenWrite]
);
GO

CREATE EXTERNAL FILE FORMAT [ParquetFF] WITH (
    FORMAT_TYPE = PARQUET,
    DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'
);
GO

CREATE EXTERNAL TABLE [dbo].[PopulationCETAS] WITH (
        LOCATION = 'populationParquet/',
        DATA_SOURCE = [MyDataSource],
        FILE_FORMAT = [ParquetFF]
) AS
SELECT
    *
FROM
    OPENROWSET(
        BULK 'csv/population-unix/population.csv',
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0'
    ) WITH (
        CountryCode varchar(4),
        CountryName varchar(64),
        Year int,
        PopulationCount int
    ) AS r;

```

> [!NOTE]
> Вам необходимо изменить этот скрипт и задать новое целевое расположение для его выполнения. Внешние таблицы не могут быть созданы в расположении, где уже присутствуют данные.

## <a name="use-the-external-table"></a>Использование внешней таблицы

Вы можете использовать внешнюю таблицу, созданную с помощью CETAS, как обычную внешнюю таблицу.

> [!NOTE]
> Измените первую строку в запросе ([mydbname]), чтобы использовать созданную вами базу данных.

```sql
USE [mydbname];
GO

SELECT
    country_name, population
FROM PopulationCETAS
WHERE
    [year] = 2019
ORDER BY
    [population] DESC;
```

## <a name="remarks"></a>Примечания

После сохранения результатов данные во внешней таблице нельзя будет изменить. Этот скрипт нельзя повторять, так как CETAS не перезаписывает базовые данные, созданные при предыдущем выполнении. Проголосуйте за следующие элементы системы отзывов, если некоторые из них требуются в ваших сценариях, или предложите новые на сайте отзывов Azure:
- [Включение вставки новых данных во внешнюю таблицу](https://feedback.azure.com/forums/307516-azure-synapse-analytics/suggestions/32981347-polybase-allow-insert-new-data-to-existing-exteran)
- [Включение удаления данных из внешней таблицы](https://feedback.azure.com/forums/307516-azure-synapse-analytics/suggestions/15158034-polybase-delete-from-external-tables)
- [Указание разделов в CETAS](https://feedback.azure.com/forums/307516-azure-synapse-analytics/suggestions/19520860-polybase-partitioned-by-functionality-when-creati)
- [Указание размеров и количества файлов](https://feedback.azure.com/forums/307516-azure-synapse-analytics/suggestions/42263617-cetas-specify-number-of-parquet-files-file-size)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о том, как запрашивать различные типы файлов, см. в статьях [Запрашивание одного CSV-файла](query-single-csv-file.md), [Запрашивание файлов Parquet](query-parquet-files.md) и [Запрашивание файлов JSON](query-json-files.md).
