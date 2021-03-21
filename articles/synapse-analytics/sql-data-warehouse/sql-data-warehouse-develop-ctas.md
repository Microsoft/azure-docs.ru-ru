---
title: CREATE TABLE AS SELECT (CTAS)
description: Пояснение и примеры инструкции CREATE TABLE как SELECT (CTAS) в синапсе SQL для разработки решений.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/26/2019
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seoapril2019, azure-synapse
ms.openlocfilehash: 68bab754142538fc6067cf2593ae6244a03a48d1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98734820"
---
# <a name="create-table-as-select-ctas"></a>CREATE TABLE AS SELECT (CTAS)

В этой статье объясняется, CREATE TABLE как инструкция T-SQL в синапсе SQL для разработки решений. Здесь также приведены примеры кодов.

## <a name="create-table-as-select"></a>CREATE TABLE AS SELECT

Инструкция [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) (CTAS) является одной из наиболее важных доступных функций T-SQL. CTAS — это параллельная операция, которая создает новую таблицу на основе выходных данных инструкции SELECT. CTAS — это самый простой и быстрый способ создания и вставки данных в таблицу с помощью одной команды.

## <a name="selectinto-vs-ctas"></a>Выберите... В VS. CTAS

CTAS — это более настраиваемая версия [SELECT... Оператор INTO](/sql/t-sql/queries/select-into-clause-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) .

Ниже приведен пример простого SELECT... БИТЬ

```sql
SELECT *
INTO    [dbo].[FactInternetSales_new]
FROM    [dbo].[FactInternetSales]
```

Выберите... В не позволяет изменять метод распределения или тип индекса как часть операции. Для создания используется `[dbo].[FactInternetSales_new]` тип распределения по умолчанию ROUND_ROBIN и табличная структура кластеризованного индекса COLUMNSTORE по умолчанию.

С другой стороны, с CTAS можно указать как распределение данных таблицы, так и тип структуры таблицы. Преобразование предыдущего примера в CTAS:

```sql
CREATE TABLE [dbo].[FactInternetSales_new]
WITH
(
 DISTRIBUTION = ROUND_ROBIN
 ,CLUSTERED COLUMNSTORE INDEX
)
AS
SELECT  *
FROM    [dbo].[FactInternetSales];
```

> [!NOTE]
> Если вы пытаетесь изменить индекс в операции CTAS, а исходная таблица распределена по хэш-коду, сохраните один и тот же столбец распределения и тип данных. Это позволяет избежать перемещения данных между распределениями во время операции, что более эффективно.

## <a name="use-ctas-to-copy-a-table"></a>Использование инструкции CTAS для копирования таблицы

Возможно, одно из наиболее распространенных применений CTAS — создание копии таблицы для изменения DDL. Предположим, вы изначально создали таблицу как `ROUND_ROBIN` , и теперь хотите изменить ее на таблицу, распределенную по столбцу. CTAS — это то, как можно изменить столбец распределения. CTAS также можно использовать для изменения секционирования, индексирования или типов столбцов.

Предположим, что вы создали эту таблицу с использованием типа распределения по умолчанию `ROUND_ROBIN` , но не указали столбец распределения в `CREATE TABLE` .

```sql
CREATE TABLE FactInternetSales
(
    ProductKey int NOT NULL,
    OrderDateKey int NOT NULL,
    DueDateKey int NOT NULL,
    ShipDateKey int NOT NULL,
    CustomerKey int NOT NULL,
    PromotionKey int NOT NULL,
    CurrencyKey int NOT NULL,
    SalesTerritoryKey int NOT NULL,
    SalesOrderNumber nvarchar(20) NOT NULL,
    SalesOrderLineNumber tinyint NOT NULL,
    RevisionNumber tinyint NOT NULL,
    OrderQuantity smallint NOT NULL,
    UnitPrice money NOT NULL,
    ExtendedAmount money NOT NULL,
    UnitPriceDiscountPct float NOT NULL,
    DiscountAmount float NOT NULL,
    ProductStandardCost money NOT NULL,
    TotalProductCost money NOT NULL,
    SalesAmount money NOT NULL,
    TaxAmt money NOT NULL,
    Freight money NOT NULL,
    CarrierTrackingNumber nvarchar(25),
    CustomerPONumber nvarchar(25));
```

Теперь нужно создать новую копию этой таблицы с помощью `Clustered Columnstore Index` , чтобы можно было воспользоваться преимуществами производительности кластеризованных таблиц columnstore. Также необходимо распространить эту таблицу `ProductKey` , так как вы ожидаете объединения в этой статье и хотите избежать перемещения данных во время соединений `ProductKey` . Наконец, необходимо также добавить секционирование в `OrderDateKey` , чтобы можно было быстро удалить старые данные, удалив старые секции. Ниже приведена инструкция CTAS, которая копирует старую таблицу в новую таблицу.

```sql
CREATE TABLE FactInternetSales_new
WITH
(
    CLUSTERED COLUMNSTORE INDEX,
    DISTRIBUTION = HASH(ProductKey),
    PARTITION
    (
        OrderDateKey RANGE RIGHT FOR VALUES
        (
        20000101,20010101,20020101,20030101,20040101,20050101,20060101,20070101,20080101,20090101,
        20100101,20110101,20120101,20130101,20140101,20150101,20160101,20170101,20180101,20190101,
        20200101,20210101,20220101,20230101,20240101,20250101,20260101,20270101,20280101,20290101
        )
    )
)
AS SELECT * FROM FactInternetSales;
```

Наконец, можно переименовать таблицы, чтобы заменить их новой таблицей, а затем удалить старую таблицу.

```sql
RENAME OBJECT FactInternetSales TO FactInternetSales_old;
RENAME OBJECT FactInternetSales_new TO FactInternetSales;

DROP TABLE FactInternetSales_old;
```

## <a name="use-ctas-to-work-around-unsupported-features"></a>Использование CTAS для обхода неподдерживаемых функций

Вы также можете использовать CTAS для обхода ряда неподдерживаемых функций, перечисленных ниже. Этот метод часто может оказаться полезным, так как не только будет соответствовать коду, но и часто будет работать быстрее в синапсе SQL. Эта производительность является результатом полностью параллелизации проекта. Сценарии включают:

* Синтаксис соединения ANSI с операторами обновления
* Синтаксис соединения ANSI с операторами удаления
* Оператор MERGE

> [!TIP]
> Постарайтесь подумать «CTAS First». Решение проблемы с помощью CTAS обычно является хорошим подходом, даже если в результате вы пишете больше данных.

## <a name="ansi-join-replacement-for-update-statements"></a>Замена соединения ANSI для инструкций обновления

Может оказаться, что имеется сложное обновление. Обновление соединяет более двух таблиц с помощью синтаксиса соединений ANSI для выполнения обновления или удаления.

Представьте, что вам необходимо обновить следующую таблицу:

```sql
CREATE TABLE [dbo].[AnnualCategorySales]
( [EnglishProductCategoryName]    NVARCHAR(50)    NOT NULL
, [CalendarYear]                    SMALLINT        NOT NULL
, [TotalSalesAmount]                MONEY            NOT NULL
)
WITH
(
    DISTRIBUTION = ROUND_ROBIN
);
```

Исходный запрос мог бы выглядеть примерно так, как показано в следующем примере:

```sql
UPDATE    acs
SET        [TotalSalesAmount] = [fis].[TotalSalesAmount]
FROM    [dbo].[AnnualCategorySales]     AS acs
JOIN    (
        SELECT [EnglishProductCategoryName]
        , [CalendarYear]
        , SUM([SalesAmount])                AS [TotalSalesAmount]
        FROM    [dbo].[FactInternetSales]        AS s
        JOIN    [dbo].[DimDate]                    AS d    ON s.[OrderDateKey]                = d.[DateKey]
        JOIN    [dbo].[DimProduct]                AS p    ON s.[ProductKey]                = p.[ProductKey]
        JOIN    [dbo].[DimProductSubCategory]    AS u    ON p.[ProductSubcategoryKey]    = u.[ProductSubcategoryKey]
        JOIN    [dbo].[DimProductCategory]        AS c    ON u.[ProductCategoryKey]        = c.[ProductCategoryKey]
        WHERE     [CalendarYear] = 2004
        GROUP BY
                [EnglishProductCategoryName]
        ,        [CalendarYear]
        ) AS fis
ON    [acs].[EnglishProductCategoryName]    = [fis].[EnglishProductCategoryName]
AND    [acs].[CalendarYear]                = [fis].[CalendarYear];
```

Синапсе SQL не поддерживает соединение ANSI в `FROM` предложении `UPDATE` инструкции, поэтому вы не можете использовать предыдущий пример, не изменяя его.

Для замены предыдущего примера можно использовать сочетание CTAS и неявного объединения.

```sql
-- Create an interim table
CREATE TABLE CTAS_acs
WITH (DISTRIBUTION = ROUND_ROBIN)
AS
SELECT    ISNULL(CAST([EnglishProductCategoryName] AS NVARCHAR(50)),0) AS [EnglishProductCategoryName]
, ISNULL(CAST([CalendarYear] AS SMALLINT),0)  AS [CalendarYear]
, ISNULL(CAST(SUM([SalesAmount]) AS MONEY),0)  AS [TotalSalesAmount]
FROM    [dbo].[FactInternetSales]        AS s
JOIN    [dbo].[DimDate]                    AS d    ON s.[OrderDateKey]                = d.[DateKey]
JOIN    [dbo].[DimProduct]                AS p    ON s.[ProductKey]                = p.[ProductKey]
JOIN    [dbo].[DimProductSubCategory]    AS u    ON p.[ProductSubcategoryKey]    = u.[ProductSubcategoryKey]
JOIN    [dbo].[DimProductCategory]        AS c    ON u.[ProductCategoryKey]        = c.[ProductCategoryKey]
WHERE     [CalendarYear] = 2004
GROUP BY [EnglishProductCategoryName]
, [CalendarYear];

-- Use an implicit join to perform the update
UPDATE  AnnualCategorySales
SET     AnnualCategorySales.TotalSalesAmount = CTAS_ACS.TotalSalesAmount
FROM    CTAS_acs
WHERE   CTAS_acs.[EnglishProductCategoryName] = AnnualCategorySales.[EnglishProductCategoryName]
AND     CTAS_acs.[CalendarYear]  = AnnualCategorySales.[CalendarYear] ;

--Drop the interim table
DROP TABLE CTAS_acs;
```

## <a name="ansi-join-replacement-for-merge"></a>Замена присоединением ANSI для слияния 

В Azure синапсе Analytics [Слияние](/sql/t-sql/statements/merge-transact-sql?view=azure-sqldw-latest&preserve-view=true) (Предварительная версия) с несоответствием целевым объектом требует, чтобы целевой объект был распределенной хэш-таблицей.  Пользователи могут использовать соединение ANSI с [Update](/sql/t-sql/queries/update-transact-sql?view=azure-sqldw-latest&preserve-view=true) или [Delete](/sql/t-sql/statements/delete-transact-sql?view=azure-sqldw-latest&preserve-view=true) в качестве обходного пути для изменения данных целевой таблицы на основе результата объединения с другой таблицей.  Ниже приведен пример.

```sql
CREATE TABLE dbo.Table1   
    (ColA INT NOT NULL, ColB DECIMAL(10,3) NOT NULL);  
GO  
CREATE TABLE dbo.Table2   
    (ColA INT NOT NULL, ColB DECIMAL(10,3) NOT NULL);  
GO  
INSERT INTO dbo.Table1 VALUES(1, 10.0);  
INSERT INTO dbo.Table2 VALUES(1, 0.0);  
GO  
UPDATE dbo.Table2   
SET dbo.Table2.ColB = dbo.Table2.ColB + dbo.Table1.ColB  
FROM dbo.Table2   
    INNER JOIN dbo.Table1   
    ON (dbo.Table2.ColA = dbo.Table1.ColA);  
GO  
SELECT ColA, ColB   
FROM dbo.Table2;

```

## <a name="explicitly-state-data-type-and-nullability-of-output"></a>явно указывайте тип данных и допустимость нулевого результата.

При переносе кода может оказаться, что вы запустили этот тип шаблона кодирования:

```sql
DECLARE @d decimal(7,2) = 85.455
,       @f float(24)    = 85.455

CREATE TABLE result
(result DECIMAL(7,2) NOT NULL
)
WITH (DISTRIBUTION = ROUND_ROBIN)

INSERT INTO result
SELECT @d*@f;
```

Вы можете подумать, что следует перенести этот код в CTAS, и вы должны быть правильными. Однако здесь есть скрытая ошибка.

Следующий код не дает такого же результата:

```sql
DECLARE @d decimal(7,2) = 85.455
, @f float(24)    = 85.455;

CREATE TABLE ctas_r
WITH (DISTRIBUTION = ROUND_ROBIN)
AS
SELECT @d*@f as result;
```

Обратите внимание, что столбец "result" переносит тип данных и допустимость значений NULL для выражения. Перенос типа данных вперед может привести к незначительным дисперсиям в значениях, если это не так осторожно.

Попробуйте следующий пример:

```sql
SELECT result,result*@d
from result;

SELECT result,result*@d
from ctas_r;
```

Значение, сохраненное для результата, отличается. Так как сохраненное значение в столбце Result используется в других выражениях, эта ошибка становится еще более значительной.

![Снимок экрана результатов CTAS](./media/sql-data-warehouse-develop-ctas/ctas-results.png)

Это важно для переноса данных. Несмотря на то, что второй запрос является более точным, существует проблема. Данные будут отличаться по сравнению с исходной системой, что ведет к вопросам целостности при миграции. Это один из тех редких случаев, когда "неправильный" ответ на самом деле является правильным!

Причина, по которой происходит различие между двумя результатами, обусловлена неявной приведением типов. В первом примере таблица определяет определение столбца. При вставке строки происходит неявное преобразование типа. Во втором примере отсутствует неявное преобразование типа, так как выражение определяет тип данных столбца.

Обратите внимание, что столбец во втором примере определен как столбец, допускающий значение null, тогда как в первом примере он не имеет значения. Когда таблица была создана в первом примере, допустимость значений NULL в столбце была определена явно. Во втором примере он оставил выражение, а по умолчанию — определение NULL.

Чтобы устранить эти проблемы, необходимо явным образом задать преобразование типа и допустимость значений NULL в части SELECT инструкции CTAS. Эти свойства нельзя задать в "CREATE TABLE".
В следующем примере показано, как исправить код:

```sql
DECLARE @d decimal(7,2) = 85.455
, @f float(24)    = 85.455

CREATE TABLE ctas_r
WITH (DISTRIBUTION = ROUND_ROBIN)
AS
SELECT ISNULL(CAST(@d*@f AS DECIMAL(7,2)),0) as result
```

Следует отметить следующее.

* Можно использовать функции CAST или CONVERT.
* Используйте ISNULL, а не объединение, чтобы принудительно реализовать допустимость значений NULL. См. следующее примечание.
* Функция ISNULL является самой внешней функцией.
* Вторая часть функции ISNULL — константа 0.

> [!NOTE]
> Чтобы правильно задать допустимость значений NULL, необходимо использовать ISNULL, а не объединение. Функция объединения не является детерминированной, поэтому результат выражения всегда будет допускать значение null. ISNULL работает иначе. Он детерминирован. Поэтому, если вторая часть функции ISNULL является константой или литералом, результирующее значение не будет равно NULL.

Проверка целостности вычислений также важна для переключения секций таблиц. Представьте, что эта таблица определена как таблица фактов:

```sql
CREATE TABLE [dbo].[Sales]
(
    [date]      INT     NOT NULL
, [product]   INT     NOT NULL
, [store]     INT     NOT NULL
, [quantity]  INT     NOT NULL
, [price]     MONEY   NOT NULL
, [amount]    MONEY   NOT NULL
)
WITH
(   DISTRIBUTION = HASH([product])
,   PARTITION   (   [date] RANGE RIGHT FOR VALUES
                    (20000101,20010101,20020101
                    ,20030101,20040101,20050101
                    )
                )
);
```

Однако поле Amount является вычисляемым выражением. Он не является частью исходных данных.

Для создания секционированного набора данных может потребоваться следующий код:

```sql
CREATE TABLE [dbo].[Sales_in]
WITH
( DISTRIBUTION = HASH([product])
, PARTITION   (   [date] RANGE RIGHT FOR VALUES
                    (20000101,20010101
                    )
                )
)
AS
SELECT
    [date]
,   [product]
,   [store]
,   [quantity]
,   [price]
,   [quantity]*[price]  AS [amount]
FROM [stg].[source]
OPTION (LABEL = 'CTAS : Partition IN table : Create');
```

Запрос будет работать отлично. Проблема возникает при попытке переключения секций. Определения таблиц не совпадают. Чтобы определения таблиц совпадали, измените CTAS, чтобы добавить `ISNULL` функцию, чтобы сохранить атрибут допустимости значений NULL для столбца.

```sql
CREATE TABLE [dbo].[Sales_in]
WITH
( DISTRIBUTION = HASH([product])
, PARTITION   (   [date] RANGE RIGHT FOR VALUES
                    (20000101,20010101
                    )
                )
)
AS
SELECT
  [date]
, [product]
, [store]
, [quantity]
, [price]
, ISNULL(CAST([quantity]*[price] AS MONEY),0) AS [amount]
FROM [stg].[source]
OPTION (LABEL = 'CTAS : Partition IN table : Create');
```

Вы можете увидеть, что согласованность типов и поддерживать свойства значений NULL в CTAS — это лучшая в проектировании методика. Это помогает поддерживать целостность вычислений, а также обеспечивает возможность переключения секций.

CTAS является одной из самых важных инструкций в синапсе SQL. Научитесь его применять. См. [документацию по CTAS](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные советы по разработке см. в статье [Проектные решения и методики программирования для хранилища данных SQL](sql-data-warehouse-overview-develop.md).