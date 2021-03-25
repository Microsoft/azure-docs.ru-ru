---
title: Создание и использование внешних таблиц в бессерверном пуле SQL
description: В этом разделе показано, как создавать и использовать внешние таблицы в бессерверном пуле SQL.
services: synapse-analytics
author: vvasic-msft
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: vvasic
ms.reviewer: jrasnick
ms.openlocfilehash: 5dcd4b7b76752ca5396fc68afc8d4c8e4e1edca5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96462306"
---
# <a name="create-and-use-external-tables-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Создание и использование внешних таблиц в бессерверном пуле SQL в Azure Synapse Analytics

В этом разделе показано, как создавать и использовать [внешние таблицы](develop-tables-external-tables.md) в бессерверном пуле SQL. Внешние таблицы полезны, если требуется управлять доступом к внешним данным в бессерверном пуле SQL и использовать такие средства, как Power BI, в сочетании с бессерверным пулом SQL. Внешние таблицы могут обращаться к хранилищам двух типов:
- общедоступное хранилище, в котором пользователи получают доступ к общедоступным файлам;
- защищенное хранилище, в котором пользователи получают доступ к файлам с помощью учетных данных SAS, удостоверения Azure AD или управляемого удостоверения рабочей области Synapse.

## <a name="prerequisites"></a>Предварительные требования

Для начала создайте базу данных, в которой будут созданы таблицы. Затем инициализируйте объекты, выполнив [скрипт настройки](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) для этой базы данных. Этот скрипт настройки создаст следующие объекты, используемые в этом примере:
- DATABASE SCOPED CREDENTIAL `sqlondemand` — предоставляет доступ к учетной записи хранения Azure `https://sqlondemandstorage.blob.core.windows.net`, защищенной с использованием SAS.

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL [sqlondemand]
    WITH IDENTITY='SHARED ACCESS SIGNATURE',  
    SECRET = 'sv=2018-03-28&ss=bf&srt=sco&sp=rl&st=2019-10-14T12%3A10%3A25Z&se=2061-12-31T12%3A10%3A00Z&sig=KlSU2ullCscyTS0An0nozEpo4tO5JAgGBvw%2FJX2lguw%3D'
    ```

- EXTERNAL DATA SOURCE `sqlondemanddemo` — ссылается на демонстрационную учетную запись хранения, защищенную ключом SAS. EXTERNAL DATA SOURCE `YellowTaxi` — ссылается на общедоступную учетную запись хранения Azure в расположении `https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/`.

    ```sql
    CREATE EXTERNAL DATA SOURCE SqlOnDemandDemo WITH (
        LOCATION = 'https://sqlondemandstorage.blob.core.windows.net',
        CREDENTIAL = sqlondemand
    );
    GO
    CREATE EXTERNAL DATA SOURCE YellowTaxi
    WITH ( LOCATION = 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/')
    ```

- Форматы файлов `QuotedCSVWithHeaderFormat` и `ParquetFormat` — описывают типы файлов CSV и Parquet.

    ```sql
    CREATE EXTERNAL FILE FORMAT QuotedCsvWithHeaderFormat
    WITH (  
        FORMAT_TYPE = DELIMITEDTEXT,
        FORMAT_OPTIONS ( FIELD_TERMINATOR = ',', STRING_DELIMITER = '"', FIRST_ROW = 2   )
    );
    GO
    CREATE EXTERNAL FILE FORMAT ParquetFormat WITH (  FORMAT_TYPE = PARQUET );
    ```

Запросы в этой статье будут выполняться к примеру базы данных и использовать эти объекты. 

## <a name="create-an-external-table-on-protected-data"></a>Создание внешней таблицы для защищенных данных

Вы можете создавать внешние таблицы для обращения к данным в учетной записи хранения Azure, которая предоставляет доступ пользователям с определенным удостоверением Azure AD или ключом SAS. Внешние таблицы можно создавать так же, как и обычные внешние таблицы SQL Server. 

В следующем запросе создается внешняя таблица, считывающая файл *population.csv* из демонстрационной учетной записи хранения Azure SynapseSQL, на которую ссылается источник данных `sqlondemanddemo` и которая защищена с помощью учетных данных уровня базы данных `sqlondemand`. 

Источник данных и учетные данные уровня базы данных создаются в [скриптах настройки](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql).

> [!NOTE]
> Измените первую строку в запросе ([mydbname]), чтобы использовать созданную вами базу данных. 

```sql
USE [mydbname];
GO
CREATE EXTERNAL TABLE populationExternalTable
(
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
    [year] smallint,
    [population] bigint
)
WITH (
    LOCATION = 'csv/population/population.csv',
    DATA_SOURCE = sqlondemanddemo,
    FILE_FORMAT = QuotedCSVWithHeaderFormat
);
```

## <a name="create-an-external-table-on-public-data"></a>Создание внешней таблицы для общедоступных данных

Вы можете создавать внешние таблицы, считывающие данные из файлов в общедоступном хранилище Azure. Этот [скрипт настройки](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) создаст общедоступный внешний источник данных и определение формата файла Parquet, которое будет использоваться в следующем запросе.

```sql
CREATE EXTERNAL TABLE Taxi (
     vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
     pickup_datetime DATETIME2, 
     dropoff_datetime DATETIME2,
     passenger_count INT,
     trip_distance FLOAT,
     fare_amount FLOAT,
     tip_amount FLOAT,
     tolls_amount FLOAT,
     total_amount FLOAT
) WITH (
         LOCATION = 'puYear=*/puMonth=*/*.parquet',
         DATA_SOURCE = YellowTaxi,
         FILE_FORMAT = ParquetFormat
);
```
## <a name="use-an-external-table"></a>Использование внешней таблицы

[Внешние таблицы](develop-tables-external-tables.md) в запросах можно использовать так же, как и в запросах SQL Server.

Следующий запрос демонстрирует использование внешней таблицы *population*, созданной ранее. Он возвращает названия стран или регионов с численностью населения по состоянию на 2019 г. в порядке убывания.

> [!NOTE]
> Измените первую строку в запросе ([mydbname]), чтобы использовать созданную вами базу данных.

```sql
USE [mydbname];
GO

SELECT
    country_name, population
FROM populationExternalTable
WHERE
    [year] = 2019
ORDER BY
    [population] DESC;
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье [Сохранение результатов запросов в хранилище с помощью решения "SQL по запросу" (предварительная версия) с помощью Azure Synapse Analytics](../sql/create-external-table-as-select.md).
