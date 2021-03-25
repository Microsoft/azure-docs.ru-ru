---
title: Создание и использование представлений в бессерверном пуле SQL
description: Из этой статьи вы узнаете, как создавать и использовать представления для "обертывания" запросов бессерверного пула SQL. Представления позволяют повторно использовать эти запросы. Представления также необходимы, если вы хотите использовать средства, такие как Power BI, в сочетании с бессерверным пулом SQL.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 05/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 0948c7c82d7577bae07057bff9d1be4d7e09f978
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96462289"
---
# <a name="create-and-use-views-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Создание и использование представлений с помощью бессерверного пула SQL в Azure Synapse Analytics

Из этой статьи вы узнаете, как создавать и использовать представления для "обертывания" запросов бессерверного пула SQL. Представления позволяют повторно использовать эти запросы. Представления также необходимы, если вы хотите использовать средства, такие как Power BI, в сочетании с бессерверным пулом SQL.

## <a name="prerequisites"></a>Предварительные требования

Первый этап — создание базы данных. В ней создается представление и инициализируются объекты, необходимые для проверки подлинности в службе хранилища Azure. Для этого в базе данных выполняется [скрипт установки](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql). Все запросы в этой статье будут выполняться в примере базы данных.

## <a name="create-a-view"></a>Создание представления

Представления можно создавать так же, как и обычные представления SQL Server. Приведенный ниже запрос позволяет создать представление, которое считывает файл *population.csv*.

> [!NOTE]
> Измените первую строку в запросе ([mydbname]), чтобы использовать созданную вами базу данных.

```sql
USE [mydbname];
GO

DROP VIEW IF EXISTS populationView;
GO

CREATE VIEW populationView AS
SELECT * 
FROM OPENROWSET(
        BULK 'csv/population/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', 
        FIELDTERMINATOR =',', 
        ROWTERMINATOR = '\n'
    )
WITH (
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
    [year] smallint,
    [population] bigint
) AS [r];
```

В представлении в этом примере применяется функция `OPENROWSET`, которая использует абсолютный путь к базовым файлам. Если есть `EXTERNAL DATA SOURCE` с корневым URL-адресом хранилища, можно использовать `OPENROWSET` с `DATA_SOURCE` и относительным путем к файлу:

```sql
CREATE VIEW TaxiView
AS SELECT *, nyc.filepath(1) AS [year], nyc.filepath(2) AS [month]
FROM
    OPENROWSET(
        BULK 'parquet/taxi/year=*/month=*/*.parquet',
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT='PARQUET'
    ) AS nyc
```

## <a name="use-a-view"></a>Использование представления

Представления в запросах можно использовать так же, как и в запросах SQL Server.

Следующий запрос демонстрирует использование представления *population_csv*, созданного в разделе [Создание представления](#create-a-view). Он возвращает названия стран и регионов с численностью населения по состоянию на 2019 г. в убывающем порядке.

> [!NOTE]
> Измените первую строку в запросе ([mydbname]), чтобы использовать созданную вами базу данных.

```sql
USE [mydbname];
GO

SELECT
    country_name, population
FROM populationView
WHERE
    [year] = 2019
ORDER BY
    [population] DESC;
```

## <a name="next-steps"></a>Дальнейшие действия

Сведения о том, как запрашивать различные типы файлов, см. в статьях [Запрашивание одного CSV-файла](query-single-csv-file.md), [Запрашивание файлов Parquet](query-parquet-files.md) и [Запрашивание файлов JSON](query-json-files.md).
