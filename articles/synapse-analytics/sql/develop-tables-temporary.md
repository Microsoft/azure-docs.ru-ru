---
title: Использование временных таблиц в синапсе SQL
description: Основные рекомендации по использованию временных таблиц в синапсе SQL.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: xiaoyul
ms.reviewer: igorstan
ms.openlocfilehash: 06faa1da71331c299245a93af96166880e7732de
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96451773"
---
# <a name="temporary-tables-in-synapse-sql"></a>Временные таблицы в синапсе SQL

Эта статья содержит важные рекомендации по использованию временных таблиц и выделяет принципы временных таблиц уровня сеанса в синапсе SQL. 

Как выделенный пул SQL, так и ресурсы пула SQL, бессерверные, могут использовать временные таблицы. У бессерверного пула SQL есть ограничения, которые обсуждаются в конце этой статьи. 

## <a name="temporary-tables"></a>Временные таблицы

Временные таблицы удобны при обработке данных — особенно во время преобразования, где промежуточные результаты являются временными. При использовании синапсе SQL временные таблицы существуют на уровне сеанса.  Их можно просмотреть только в сеансе, в котором они были созданы. Поэтому после выхода из сеанса они автоматически удаляются. 

## <a name="temporary-tables-in-dedicated-sql-pool"></a>Временные таблицы в выделенном пуле SQL

В выделенном ресурсе пула SQL временные таблицы обеспечивают выигрыш в производительности, поскольку их результаты записываются в локальную, а не в удаленное хранилище.

### <a name="create-a-temporary-table"></a>Создание временной таблицы

Для создания временной таблицы к имени таблицы добавляется префикс `#`.  Пример:

```sql
CREATE TABLE #stats_ddl
(
    [schema_name]        NVARCHAR(128) NOT NULL
,    [table_name]            NVARCHAR(128) NOT NULL
,    [stats_name]            NVARCHAR(128) NOT NULL
,    [stats_is_filtered]     BIT           NOT NULL
,    [seq_nmbr]              BIGINT        NOT NULL
,    [two_part_name]         NVARCHAR(260) NOT NULL
,    [three_part_name]       NVARCHAR(400) NOT NULL
)
WITH
(
    DISTRIBUTION = HASH([seq_nmbr])
,    HEAP
)
```

Временные таблицы можно также создать с помощью `CTAS` точно таким же образом:

```sql
CREATE TABLE #stats_ddl
WITH
(
    DISTRIBUTION = HASH([seq_nmbr])
,    HEAP
)
AS
(
SELECT
        sm.[name]                                                                AS [schema_name]
,        tb.[name]                                                                AS [table_name]
,        st.[name]                                                                AS [stats_name]
,        st.[has_filter]                                                            AS [stats_is_filtered]
,       ROW_NUMBER()
        OVER(ORDER BY (SELECT NULL))                                            AS [seq_nmbr]
,                                 QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [two_part_name]
,        QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [three_part_name]
FROM    sys.objects            AS ob
JOIN    sys.stats            AS st    ON    ob.[object_id]        = st.[object_id]
JOIN    sys.stats_columns    AS sc    ON    st.[stats_id]        = sc.[stats_id]
                                    AND st.[object_id]        = sc.[object_id]
JOIN    sys.columns            AS co    ON    sc.[column_id]        = co.[column_id]
                                    AND    sc.[object_id]        = co.[object_id]
JOIN    sys.tables            AS tb    ON    co.[object_id]        = tb.[object_id]
JOIN    sys.schemas            AS sm    ON    tb.[schema_id]        = sm.[schema_id]
WHERE    1=1
AND        st.[user_created]   = 1
GROUP BY
        sm.[name]
,        tb.[name]
,        st.[name]
,        st.[filter_definition]
,        st.[has_filter]
)
;
```

> [!NOTE]
> `CTAS` — мощная команда, которая особенно эффективна при использовании пространства журнала транзакций. 
> 
> 

### <a name="drop-temporary-tables"></a>Удалить временные таблицы

При создании сеанса не должно быть ни одной временной таблицы.  Однако при вызове той же хранимой процедуры, которая создает временную с тем же именем, чтобы убедиться в `CREATE TABLE` успешном выполнении инструкций, используйте простую проверку предварительного проверки существования  `DROP` : 

```sql
IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN
    DROP TABLE #stats_ddl
END
```

Для согласованности кода целесообразно использовать этот шаблон как для обычных, так и для временных таблиц.  Также рекомендуется `DROP TABLE` удалить временные таблицы по завершении работы с ними.  

При разработке хранимой процедуры команды удаления обычно помещаются вместе в конце процедуры, чтобы гарантировать удаление объектов.

```sql
DROP TABLE #stats_ddl
```

### <a name="modularize-code"></a>Код разделения

Временные таблицы можно использовать в любом месте пользовательского сеанса. Эта возможность может быть использована для разделения кода приложения.  Чтобы продемонстрировать, следующая хранимая процедура создает DDL для обновления всей статистики в базе данных по имени статистики:

```sql
CREATE PROCEDURE    [dbo].[prc_sqldw_update_stats]
(   @update_type    tinyint -- 1 default 2 fullscan 3 sample 4 resample
    ,@sample_pct     tinyint
)
AS

IF @update_type NOT IN (1,2,3,4)
BEGIN;
    THROW 151000,'Invalid value for @update_type parameter. Valid range 1 (default), 2 (fullscan), 3 (sample) or 4 (resample).',1;
END;

IF @sample_pct IS NULL
BEGIN;
    SET @sample_pct = 20;
END;

IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN
    DROP TABLE #stats_ddl
END

CREATE TABLE #stats_ddl
WITH
(
    DISTRIBUTION = HASH([seq_nmbr])
)
AS
(
SELECT
        sm.[name]                                                                AS [schema_name]
,        tb.[name]                                                                AS [table_name]
,        st.[name]                                                                AS [stats_name]
,        st.[has_filter]                                                            AS [stats_is_filtered]
,       ROW_NUMBER()
        OVER(ORDER BY (SELECT NULL))                                            AS [seq_nmbr]
,                                 QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [two_part_name]
,        QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [three_part_name]
FROM    sys.objects            AS ob
JOIN    sys.stats            AS st    ON    ob.[object_id]        = st.[object_id]
JOIN    sys.stats_columns    AS sc    ON    st.[stats_id]        = sc.[stats_id]
                                    AND st.[object_id]        = sc.[object_id]
JOIN    sys.columns            AS co    ON    sc.[column_id]        = co.[column_id]
                                    AND    sc.[object_id]        = co.[object_id]
JOIN    sys.tables            AS tb    ON    co.[object_id]        = tb.[object_id]
JOIN    sys.schemas            AS sm    ON    tb.[schema_id]        = sm.[schema_id]
WHERE    1=1
AND        st.[user_created]   = 1
GROUP BY
        sm.[name]
,        tb.[name]
,        st.[name]
,        st.[filter_definition]
,        st.[has_filter]
)
SELECT
    CASE @update_type
    WHEN 1
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+');'
    WHEN 2
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH FULLSCAN;'
    WHEN 3
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH SAMPLE '+CAST(@sample_pct AS VARCHAR(20))+' PERCENT;'
    WHEN 4
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH RESAMPLE;'
    END AS [update_stats_ddl]
,   [seq_nmbr]
FROM    t1
;
GO
```

На этом этапе единственное произошедшее действие — создание хранимой процедуры, которая создает #stats_ddl временную таблицу.  Хранимая процедура удаляет #stats_ddl, если она уже существует. Эта инструкция DROP гарантирует, что она не завершится ошибкой, если в сеансе будет выполняться несколько раз.  

Поскольку в `DROP TABLE` конце хранимой процедуры отсутствует элемент, то при завершении хранимой процедуры созданная таблица остается и может быть прочитана за пределами хранимой процедуры.  

В отличие от других SQL Server баз данных, синапсе SQL позволяет использовать временную таблицу за пределами процедуры, которая ее создала.  Временные таблицы, созданные через выделенный пул SQL, можно использовать в **любом месте** внутри сеанса. В результате у вас будет более модульный и управляемый код, как показано в примере ниже:

```sql
EXEC [dbo].[prc_sqldw_update_stats] @update_type = 1, @sample_pct = NULL;

DECLARE @i INT              = 1
,       @t INT              = (SELECT COUNT(*) FROM #stats_ddl)
,       @s NVARCHAR(4000)   = N''

WHILE @i <= @t
BEGIN
    SET @s=(SELECT update_stats_ddl FROM #stats_ddl WHERE seq_nmbr = @i);

    PRINT @s
    EXEC sp_executesql @s
    SET @i+=1;
END

DROP TABLE #stats_ddl;
```

### <a name="temporary-table-limitations"></a>Ограничения временной таблицы

В выделенном пуле SQL есть несколько ограничений реализации для временных таблиц:

- Поддерживаются только временные таблицы с областью действия сеанса.  Глобальные временные таблицы не поддерживаются.
- Невозможно создать представления во временных таблицах.
- Временные таблицы можно создавать только с помощью хэша или циклического распределения.  Распределение реплицированных временных таблиц не поддерживается. 

## <a name="temporary-tables-in-serverless-sql-pool"></a>Временные таблицы в несерверном пуле SQL

Временные таблицы в бессерверном пуле SQL поддерживаются, но их использование ограничено. Они не могут использоваться в запросах, которые предназначены для целевых файлов. 

Например, нельзя присоединить временную таблицу к данным из файлов в хранилище. Количество временных таблиц ограничено 100, а их общий размер ограничен 100 МБ.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о разработке таблиц см. в статье [Проектирование таблиц с помощью ресурсов Synapse SQL](develop-tables-overview.md).

