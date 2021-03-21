---
title: Использование удостоверения для создания суррогатных ключей
description: Рекомендации и примеры использования свойства IDENTITY для создания суррогатных ключей для таблиц в выделенном пуле SQL.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 07/20/2020
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 5c620aa60e134379614a905226caa4a66d179fae
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98673541"
---
# <a name="using-identity-to-create-surrogate-keys-using-dedicated-sql-pool-in-azuresynapse-analytics"></a>Использование удостоверения для создания суррогатных ключей с помощью выделенного пула SQL в Азуресинапсе Analytics

В этой статье вы найдете рекомендации и примеры использования свойства IDENTITY для создания суррогатных ключей для таблиц в выделенном пуле SQL.

## <a name="what-is-a-surrogate-key"></a>Что такое суррогатный ключ

Суррогатный ключ таблицы представляет собой столбец с уникальным идентификатором каждой строки. Ключ не генерируется из данных таблицы. Разработчики моделей данных создают суррогатные ключи для таблиц, когда проектируют модели хранилища данных. Чтобы просто и эффективно достичь этой цели, не оказывая влияния на производительность загрузки, можно использовать свойство IDENTITY.
> [!NOTE]
> В Azure синапсе Analytics значение идентификатора увеличивается отдельно в каждом распределении и не перекрывается со значениями ИДЕНТИФИКАТОРов в других дистрибутивах.  Значение идентификатора в синапсе не обязательно должно быть уникальным, если пользователь явно вставляет повторяющееся значение с параметром "SET IDENTITY_INSERT" или повторно заполняет УДОСТОВЕРЕНие. Дополнительные сведения см. в разделе [CREATE TABLE (Transact-SQL) Identity (свойство)](/sql/t-sql/statements/create-table-transact-sql-identity-property?view=azure-sqldw-latest&preserve-view=true). 


## <a name="creating-a-table-with-an-identity-column"></a>Создание таблицы со столбцом IDENTITY

Свойство IDENTITY предназначено для горизонтального масштабирования всех распределений в выделенном пуле SQL, не влияя на производительность нагрузки. Поэтому реализация IDENTITY ориентирована на достижение этих целей.

Свойство IDENTITY можно определить для таблицы при ее создании, используя синтаксис, как в приведенной ниже инструкции.

```sql
CREATE TABLE dbo.T1
(    C1 INT IDENTITY(1,1) NOT NULL
,    C2 INT NULL
)
WITH
(   DISTRIBUTION = HASH(C2)
,   CLUSTERED COLUMNSTORE INDEX
)
;
```

Затем можно использовать `INSERT..SELECT` для заполнения таблицы.

В оставшейся части этого раздела описываются особенности этой реализации, что поможет вам лучше понять их.  

### <a name="allocation-of-values"></a>Распределение значений

Свойство IDENTITY не гарантирует, что порядок размещения суррогатных значений будет выделен из-за распределенной архитектуры хранилища данных. Свойство IDENTITY предназначено для горизонтального масштабирования всех распределений в выделенном пуле SQL, не влияя на производительность нагрузки. 

Ниже приведен характерный пример.

```sql
CREATE TABLE dbo.T1
(    C1 INT IDENTITY(1,1)    NOT NULL
,    C2 VARCHAR(30)                NULL
)
WITH
(   DISTRIBUTION = HASH(C2)
,   CLUSTERED COLUMNSTORE INDEX
)
;

INSERT INTO dbo.T1
VALUES (NULL);

INSERT INTO dbo.T1
VALUES (NULL);

SELECT *
FROM dbo.T1;

DBCC PDW_SHOWSPACEUSED('dbo.T1');
```

В приведенном выше примере две строки попали в распределение 1. У первой строки есть суррогатное значение 1 в столбце `C1`, а у второй строки есть суррогатное значение 61. Оба эти значения были созданы свойством IDENTITY. Тем не менее распределение этих значений не является связанным. В этом весь замысел.

### <a name="skewed-data"></a>Неравномерные данные

Диапазон значений определенного типа данных равномерно размещается в распределениях. Если распределенная таблица содержит неравномерные данные, то диапазон значений, доступных для типа данных, может быть преждевременно исчерпан. Например, если все данные попадают в отдельное распределение, фактически таблица имеет доступ к только одной шестидесятой части значений этого типа данных. По этой причине свойство IDENTITY ограничено следующими типами данных: `INT` и `BIGINT`.

### <a name="selectinto"></a>SELECT..INTO

При выборке существующего столбца IDENTITY в новую таблицу новый столбец наследует свойство IDENTITY, если только не выполняется одно из следующих условий:

- Инструкция SELECT содержит соединение.
- несколько инструкций SELECT соединены при помощи UNION;
- столбец IDENTITY более одного раза указан в списке инструкции SELECT;
- столбец IDENTITY является частью выражения.

Если любое из этих условий выполняется, столбец создается как NOT NULL и не наследует свойство IDENTITY.

### <a name="create-table-as-select"></a>CREATE TABLE AS SELECT

CREATE TABLE AS SELECT (CTAS) выполняется SQL Server так же, как и SELECT..INTO. Тем не менее невозможно указать свойство IDENTITY в определении столбца в части `CREATE TABLE` инструкции. Невозможно также использовать функцию IDENTITY в части `SELECT` инструкции CTAS. Для заполнения таблицы необходимо использовать `CREATE TABLE`, чтобы определить таблицу, а затем указать `INSERT..SELECT`, чтобы ее заполнить.

## <a name="explicitly-inserting-values-into-an-identity-column"></a>Вставка значений в столбец IDENTITY явным образом

Выделенный пул SQL поддерживает `SET IDENTITY_INSERT <your table> ON|OFF` синтаксис. Этот синтаксис позволяет явно вставить значения в столбец IDENTITY.

Многие разработчики моделей данных используют в измерениях предопределенные отрицательные значения для определенных строк. Например, значение -1 или строка "неизвестный элемент".

Приведенный ниже сценарий показывает, как явным образом добавить эту строку с помощью инструкции SET IDENTITY_INSERT.

```sql
SET IDENTITY_INSERT dbo.T1 ON;

INSERT INTO dbo.T1
(   C1
,   C2
)
VALUES (-1,'UNKNOWN')
;

SET IDENTITY_INSERT dbo.T1 OFF;

SELECT     *
FROM    dbo.T1
;
```

## <a name="loading-data"></a>Загрузка данных

Наличие свойства IDENTITY определенным образом отражается на коде для загрузки данных. В этом разделе описываются некоторые основные схемы загрузки данных в таблицы с использованием свойства IDENTITY.

Чтобы загрузить данные в таблицу и создать суррогатный ключ с помощью свойства IDENTITY, создайте таблицу и выполните инструкцию INSERT..SELECT или INSERT..VALUES для загрузки данных.

В следующем примере представлена базовая схема.

```sql
--CREATE TABLE with IDENTITY
CREATE TABLE dbo.T1
(    C1 INT IDENTITY(1,1)
,    C2 VARCHAR(30)
)
WITH
(   DISTRIBUTION = HASH(C2)
,   CLUSTERED COLUMNSTORE INDEX
)
;

--Use INSERT..SELECT to populate the table from an external table
INSERT INTO dbo.T1
(C2)
SELECT     C2
FROM    ext.T1
;

SELECT *
FROM   dbo.T1
;

DBCC PDW_SHOWSPACEUSED('dbo.T1');
```

> [!NOTE]
> В настоящее время невозможно использовать `CREATE TABLE AS SELECT` при загрузке данных в таблицу со столбцом IDENTITY.
>

Дополнительные сведения о загрузке данных см. в разделе [Разработка операций извлечения, загрузки и преобразования (ELT) для выделенного пула SQL](design-elt-data-loading.md) и  [Загрузка рекомендаций](guidance-for-loading-data.md).

## <a name="system-views"></a>Системные представления

Можно использовать представление каталога [sys.identity_columns](/sql/relational-databases/system-catalog-views/sys-identity-columns-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) для поиска столбца со свойством IDENTITY.

Для лучшего понимания схемы базы данных в этом примере показано, как интегрировать sys.identity_column с остальными представлениями системных каталогов.

```sql
SELECT  sm.name
,       tb.name
,       co.name
,       CASE WHEN ic.column_id IS NOT NULL
             THEN 1
        ELSE 0
        END AS is_identity
FROM        sys.schemas AS sm
JOIN        sys.tables  AS tb           ON  sm.schema_id = tb.schema_id
JOIN        sys.columns AS co           ON  tb.object_id = co.object_id
LEFT JOIN   sys.identity_columns AS ic  ON  co.object_id = ic.object_id
                                        AND co.column_id = ic.column_id
WHERE   sm.name = 'dbo'
AND     tb.name = 'T1'
;
```

## <a name="limitations"></a>Ограничения

Свойство IDENTITY не может быть использовано:

- если тип данных столбца не является INT или BIGINT;
- если столбец также является ключом распределения;
- если таблица является внешней.

Следующие связанные функции не поддерживаются в выделенном пуле SQL:

- [IDENTITY ()](/sql/t-sql/functions/identity-function-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [@@IDENTITY](/sql/t-sql/functions/identity-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [SCOPE_IDENTITY](/sql/t-sql/functions/scope-identity-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [IDENT_CURRENT](/sql/t-sql/functions/ident-current-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [IDENT_INCR](/sql/t-sql/functions/ident-incr-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [IDENT_SEED](/sql/t-sql/functions/ident-seed-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)

## <a name="common-tasks"></a>Стандартные задачи

Этот раздел содержит пример кода, который можно использовать для выполнения распространенных задач при работе со столбцами IDENTITY.

Столбец C1 является столбцом IDENTITY во всех следующих задачах.

### <a name="find-the-highest-allocated-value-for-a-table"></a>Поиск максимального распределенного значения в таблице

Используйте функцию `MAX()`, чтобы определить максимальное значение в распределенной таблице.

```sql
SELECT MAX(C1)
FROM dbo.T1
```

### <a name="find-the-seed-and-increment-for-the-identity-property"></a>Поиск начального значения и шага приращения для свойства IDENTITY

Представления каталога можно использовать для обнаружения значения шага приращения идентификатора и начального значения конфигурации для таблицы. Для этого можно выполнить приведенный ниже запрос.

```sql
SELECT  sm.name
,       tb.name
,       co.name
,       ic.seed_value
,       ic.increment_value
FROM        sys.schemas AS sm
JOIN        sys.tables  AS tb           ON  sm.schema_id = tb.schema_id
JOIN        sys.columns AS co           ON  tb.object_id = co.object_id
JOIN        sys.identity_columns AS ic  ON  co.object_id = ic.object_id
                                        AND co.column_id = ic.column_id
WHERE   sm.name = 'dbo'
AND     tb.name = 'T1'
;
```

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о таблицах](sql-data-warehouse-tables-overview.md)
- [CREATE TABLE (Transact-SQL) IDENTITY (Свойство)](/sql/t-sql/statements/create-table-transact-sql-identity-property?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [DBCC ЧЕККИНДЕНТ](/sql/t-sql/database-console-commands/dbcc-checkident-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
