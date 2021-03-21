---
title: Папки запросов и несколько файлов с помощью несерверного пула SQL
description: Бессерверный пул SQL поддерживает чтение нескольких файлов и папок с помощью подстановочных знаков, которые похожи на подстановочные знаки, используемые в ОС Windows.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 04/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 83c4d88e1a87f6b546e26dd55da338a36f16ebe4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96462632"
---
# <a name="query-folders-and-multiple-files"></a>Запрашивание папок и нескольких файлов  

В этой статье вы узнаете, как написать запрос с помощью бессерверного пула SQL в Azure синапсе Analytics.

Бессерверный пул SQL поддерживает чтение нескольких файлов и папок с помощью подстановочных знаков, которые похожи на подстановочные знаки, используемые в ОС Windows. Однако большая гибкость имеется, так как разрешено использование нескольких подстановочных знаков.

## <a name="prerequisites"></a>Предварительные условия

Для начала **создайте базу данных**, в которой будут выполняться запросы. Затем инициализируйте объекты, выполнив [скрипт настройки](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) для этой базы данных. Этот сценарий установки создает источники данных, учетные данные области базы данных и форматы внешних файлов, которые используются в этих примерах.

Для выполнения примеров запросов вы будете использовать *CSV-файл или каталог такси* . В нем содержится Нью такси (желтый), который записывает данные с 2016 июля по 2018 июня. Файлы в *формате CSV или такси* именуются по годам и месяцу с использованием следующего шаблона: yellow_tripdata_ <year> - <month> . csv.

## <a name="read-all-files-in-folder"></a>Чтение всех файлов в папке

В приведенном ниже примере считываются все файлы данных Нью желтого такси из папки *CSV/такси* и возвращается общее число пассажиров и задается в год. Он также показывает использование агрегатных функций.

```sql
SELECT 
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
        BULK 'csv/taxi/*.csv',
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        pickup_datetime DATETIME2 2, 
        passenger_count INT 4
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Все файлы, доступ к которым осуществляется с помощью одной инструкции OPENROWSET, должны иметь одинаковую структуру (т. е. число столбцов и их типы данных).

### <a name="read-subset-of-files-in-folder"></a>Чтение подмножества файлов в папке

В примере ниже считываются файлы данных 2017 Нью желтого такси из папки *CSV/такси* с использованием подстановочных знаков и возвращается общая сумма FARE для каждого типа платежа.

```sql
SELECT 
    payment_type,  
    SUM(fare_amount) AS fare_total
FROM OPENROWSET(
        BULK 'csv/taxi/yellow_tripdata_2017-*.csv',
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        payment_type INT 10,
        fare_amount FLOAT 11
    ) AS nyc
GROUP BY payment_type
ORDER BY payment_type;
```

> [!NOTE]
> Все файлы, доступ к которым осуществляется с помощью одной инструкции OPENROWSET, должны иметь одинаковую структуру (т. е. число столбцов и их типы данных).

## <a name="read-folders"></a>Чтение папок

Путь, который вы предоставляете в инструкции OPENROWSET, также может быть путем к папке. Эти типы запросов включают следующие разделы.

### <a name="read-all-files-from-specific-folder"></a>Чтение всех файлов из указанной папки

Вы можете читать все файлы в папке, используя подстановочный знак уровня файла, как показано в [статье чтение всех файлов в папке](#read-all-files-in-folder). Но существует способ запросить папку и использовать все файлы в этой папке.

Если путь, указанный в инструкции OPENROWSET, указывает на папку, все файлы в этой папке будут использоваться в качестве источника запроса. Следующий запрос будет считывать все файлы в папке *CSV/такси* .

> [!NOTE]
> Обратите внимание на существование/в конце пути в приведенном ниже запросе. Он обозначает папку. Если параметр/не указан, запрос будет указывать на файл с именем *такси* .

```sql
SELECT
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
        BULK 'csv/taxi/',
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
        pickup_datetime DATETIME2, 
        dropoff_datetime DATETIME2,
        passenger_count INT,
        trip_distance FLOAT,
        rate_code INT,
        store_and_fwd_flag VARCHAR(100) COLLATE Latin1_General_BIN2,
        pickup_location_id INT,
        dropoff_location_id INT,
        payment_type INT,
        fare_amount FLOAT,
        extra FLOAT,
        mta_tax FLOAT,
        tip_amount FLOAT,
        tolls_amount FLOAT,
        improvement_surcharge FLOAT,
        total_amount FLOAT
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Все файлы, доступ к которым осуществляется с помощью одной инструкции OPENROWSET, должны иметь одинаковую структуру (т. е. число столбцов и их типы данных).

### <a name="read-all-files-from-multiple-folders"></a>Чтение всех файлов из нескольких папок

Можно считывать файлы из нескольких папок с помощью подстановочных знаков. Следующий запрос будет считывать все файлы из всех папок, расположенных в папке *CSV* , имена которых начинаются с *t* и заканчиваются на *i*.

> [!NOTE]
> Обратите внимание на существование/в конце пути в приведенном ниже запросе. Он обозначает папку. Если параметр/не указан, запрос будет указывать на файлы с именем *t &ast; i* .

```sql
SELECT
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
        BULK 'csv/t*i/', 
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
        pickup_datetime DATETIME2, 
        dropoff_datetime DATETIME2,
        passenger_count INT,
        trip_distance FLOAT,
        rate_code INT,
        store_and_fwd_flag VARCHAR(100) COLLATE Latin1_General_BIN2,
        pickup_location_id INT,
        dropoff_location_id INT,
        payment_type INT,
        fare_amount FLOAT,
        extra FLOAT,
        mta_tax FLOAT,
        tip_amount FLOAT,
        tolls_amount FLOAT,
        improvement_surcharge FLOAT,
        total_amount FLOAT
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Все файлы, доступ к которым осуществляется с помощью одной инструкции OPENROWSET, должны иметь одинаковую структуру (т. е. число столбцов и их типы данных).

Так как у вас есть только одна папка, соответствующая критериям, результат запроса будет таким же, как и [чтение всех файлов в папке](#read-all-files-in-folder).

## <a name="traverse-folders-recursively"></a>Рекурсивно просматривать папки

Несерверный пул SQL может рекурсивно перемещаться по папкам, если в конце пути указан/* *. Следующий запрос будет считывать все файлы из всех папок и вложенных папок, расположенных в папке *CSV* .

```sql
SELECT
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
        BULK 'csv/taxi/**', 
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
        pickup_datetime DATETIME2, 
        dropoff_datetime DATETIME2,
        passenger_count INT,
        trip_distance FLOAT,
        rate_code INT,
        store_and_fwd_flag VARCHAR(100) COLLATE Latin1_General_BIN2,
        pickup_location_id INT,
        dropoff_location_id INT,
        payment_type INT,
        fare_amount FLOAT,
        extra FLOAT,
        mta_tax FLOAT,
        tip_amount FLOAT,
        tolls_amount FLOAT,
        improvement_surcharge FLOAT,
        total_amount FLOAT
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Все файлы, доступ к которым осуществляется с помощью одной инструкции OPENROWSET, должны иметь одинаковую структуру (т. е. число столбцов и их типы данных).

## <a name="multiple-wildcards"></a>Несколько подстановочных знаков

Можно использовать несколько подстановочных знаков на разных уровнях пути. Например, можно создать предыдущий запрос для чтения файлов только с данными 2017, из всех папок, имена которых начинаются с *t* и заканчиваются на *i*.

> [!NOTE]
> Обратите внимание на существование/в конце пути в приведенном ниже запросе. Он обозначает папку. Если параметр/не указан, запрос будет указывать на файлы с именем *t &ast; i* .
> Максимальное число подстановочных знаков для каждого запроса — 10.

```sql
SELECT
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
        BULK 'csv/t*i/yellow_tripdata_2017-*.csv',
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
        pickup_datetime DATETIME2, 
        dropoff_datetime DATETIME2,
        passenger_count INT,
        trip_distance FLOAT,
        rate_code INT,
        store_and_fwd_flag VARCHAR(100) COLLATE Latin1_General_BIN2,
        pickup_location_id INT,
        dropoff_location_id INT,
        payment_type INT,
        fare_amount FLOAT,
        extra FLOAT,
        mta_tax FLOAT,
        tip_amount FLOAT,
        tolls_amount FLOAT,
        improvement_surcharge FLOAT,
        total_amount FLOAT
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Все файлы, доступ к которым осуществляется с помощью одной инструкции OPENROWSET, должны иметь одинаковую структуру (т. е. число столбцов и их типы данных).

Так как у вас есть только одна папка, соответствующая критериям, результат запроса будет таким же, как и для [чтения подмножества файлов в папке](#read-subset-of-files-in-folder) , и для [чтения всех файлов из указанной папки](#read-all-files-from-specific-folder). Более сложные сценарии использования подстановочных знаков рассматриваются в разделе [файлы Parquet запросов](query-parquet-files.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения можно найти в статье о [файлах, связанных с запросом](query-specific-files.md) .
