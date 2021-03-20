---
title: Используйте каталог U-SQL в Azure Data Lake Analytics
description: Сведения об использовании каталога U-SQL для совместного использования кода и данных. Создание функций, возвращающих табличное значение, создание представлений, создание таблиц и выполнение запросов к ним.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 05/09/2017
ms.openlocfilehash: f92aadc8ccf18dd91b5dd4b35285f60b174e4cf7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92220080"
---
# <a name="get-started-with-the-u-sql-catalog-in-azure-data-lake-analytics"></a>Начало работы с каталогом U-SQL в Azure Data Lake Analytics

## <a name="create-a-tvf"></a>Создание функции TVF

В предыдущем скрипте U-SQL мы повторно использовали EXTRACT для чтения из одного и того же исходного файла. В U-SQL функция с табличным значением (TVF) позволяет инкапсулировать данные для повторного использования в будущем.  

Следующий скрипт создает возвращающую табличное значение функцию с именем `Searchlog()` в базе данных и схеме по умолчанию:

```usql
DROP FUNCTION IF EXISTS Searchlog;

CREATE FUNCTION Searchlog()
RETURNS @searchlog TABLE
(
            UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
)
AS BEGIN
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM "/Samples/Data/SearchLog.tsv"
USING Extractors.Tsv();
RETURN;
END;
```

Следующий скрипт демонстрирует, как использовать функцию TVF, определенную в предыдущем скрипте:

```usql
@res =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM Searchlog() AS S
GROUP BY Region
HAVING SUM(Duration) > 200;

OUTPUT @res
    TO "/output/SearchLog-use-tvf.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

## <a name="create-views"></a>Создание представлений

Если у вас есть одно выражение запроса вместо возвращающей табличное значение функции, можно использовать представление U-SQL для инкапсуляции этого выражения.

Следующий скрипт создает представление с именем `SearchlogView` в базе данных и схеме по умолчанию:

```usql
DROP VIEW IF EXISTS SearchlogView;

CREATE VIEW SearchlogView AS  
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM "/Samples/Data/SearchLog.tsv"
USING Extractors.Tsv();
```

Следующий скрипт демонстрирует, как использовать определенное представление.

```usql
@res =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM SearchlogView
GROUP BY Region
HAVING SUM(Duration) > 200;

OUTPUT @res
    TO "/output/Searchlog-use-view.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

## <a name="create-tables"></a>Создание таблиц
Как и в случае с таблицами реляционной базы данных, U-SQL позволяет создать таблицу с предварительно определенной схемой или же создать таблицу и определить схему из запроса, который заполняет таблицу (CREATE TABLE AS SELECT или CTAS).

Следующий скрипт создает базу данных и две таблицы:

```usql
DROP DATABASE IF EXISTS SearchLogDb;
CREATE DATABASE SearchLogDb;
USE DATABASE SearchLogDb;

DROP TABLE IF EXISTS SearchLog1;
DROP TABLE IF EXISTS SearchLog2;

CREATE TABLE SearchLog1 (
            UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string,

            INDEX sl_idx CLUSTERED (UserId ASC)
                DISTRIBUTED BY HASH (UserId)
);

INSERT INTO SearchLog1 SELECT * FROM master.dbo.Searchlog() AS s;

CREATE TABLE SearchLog2(
    INDEX sl_idx CLUSTERED (UserId ASC)
            DISTRIBUTED BY HASH (UserId)
) AS SELECT * FROM master.dbo.Searchlog() AS S; // You can use EXTRACT or SELECT here
```

## <a name="query-tables"></a>Запросы к таблицам
Вы можете отправлять запросы к таблицам (созданным с помощью предыдущего скрипта) так же, как вы отправляете запросы к файлам данных. Чтобы не создавать набор строк с помощью EXTRACT, теперь можно просто ссылаться на имя таблицы.

Для чтения из таблиц нужно изменить скрипт преобразования, который использовался ранее:

```usql
@rs1 =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM SearchLogDb.dbo.SearchLog2
GROUP BY Region;

@res =
    SELECT *
    FROM @rs1
    ORDER BY TotalDuration DESC
    FETCH 5 ROWS;

OUTPUT @res
    TO "/output/Searchlog-query-table.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

 >[!NOTE]
 >Сейчас выполнять SELECT для таблицы в том же скрипте, где создается таблица, нельзя.

## <a name="next-steps"></a>Дальнейшие действия
* [Обзор аналитики озера данных Microsoft Azure](data-lake-analytics-overview.md)
* [Разработка сценариев U-SQL с помощью средств озера данных для Visual Studio.](data-lake-analytics-data-lake-tools-get-started.md)
* [Мониторинг и устранение неполадок Azure Data Lake Analytics заданий с помощью портал Azure](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)
