---
title: Пример использования технологий обработки в оперативной памяти
description: Пример использования технологий обработки в оперативной памяти в Базе данных SQL Azure с OLTP и индексами columnstore.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: sample
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: ''
ms.date: 12/18/2018
ms.openlocfilehash: b5a1035f8a213a6ce02dd3252ff7d3ddea46faf7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92786587"
---
# <a name="in-memory-sample"></a>Пример использования технологий обработки в оперативной памяти
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

Технологии обработки в оперативной памяти в Базе данных SQL Azure позволяют повысить производительность приложений и уменьшить стоимость использования базы данных. С помощью технологий обработки в оперативной памяти в Базе данных SQL Azure вы можете добиться улучшения производительности с помощью различных рабочих нагрузок.

В этой статье вы ознакомитесь с двумя примерами по использованию OLTP в памяти и индексов сolumnstore в Базе данных SQL Azure.

Дополнительные сведения можно найти в разделе

- Статья [Общие сведения и сценарии использования](/sql/relational-databases/in-memory-oltp/overview-and-usage-scenarios) содержит ссылки на примеры реальных клиентов и сведения, необходимые для начала работы.
- [In-Memory OLTP (оптимизация в памяти)](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)
- [Руководство по индексам columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview)
- Гибридные сценарии транзакционной и аналитической обработки, которые также называются [операционной аналитикой в реальном времени](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics).

<a id="install_oltp_manuallink" name="install_oltp_manuallink"></a>

&nbsp;

## <a name="1-install-the-in-memory-oltp-sample"></a>1. Установка образца In-Memory OLTP

На [портале Azure](https://portal.azure.com/) вы можете быстро и просто создать пример базы данных AdventureWorksLT. Также в этом разделе объясняется, как можно расширить базу данных AdventureWorksLT с использованием объектов выполняющейся в памяти OLTP, чтобы реализовать повышение производительности.

Упрощенная, но одновременно более наглядная демонстрация выполняющейся в памяти OLTP представлена здесь:

- Выпуск: [in-memory-oltp-demo-v1.0](https://github.com/Microsoft/sql-server-samples/releases/tag/in-memory-oltp-demo-v1.0)
- Исходный код: [in-memory-oltp-demo-source-code](https://github.com/microsoft/sql-server-samples/tree/master/samples/features/in-memory-database)

### <a name="installation-steps"></a>Шаги установки

1. На [портале Azure](https://portal.azure.com/) создайте на сервере базу данных уровня "Премиум" или "Критически важный для бизнеса". Укажите **источник** для примера базы данных AdventureWorksLT. Подробные инструкции см. в статье [Краткое руководство. Создание отдельной базы данных в Базе данных SQL Azure](database/single-database-create-quickstart.md).

2. Подключитесь к базе данных с помощью SQL Server Management Studio [(SSMS.exe)](/sql/ssms/download-sql-server-management-studio-ssms).

3. Скопируйте в буфер обмена [скрипт Transact-SQL для In-Memory OLTP](https://raw.githubusercontent.com/microsoft/sql-server-samples/master/samples/features/in-memory-database/in-memory-oltp/t-sql-scripts/sql_in-memory_oltp_sample.sql) . Скрипт T-SQL создаст необходимые объекты обработки в оперативной памяти в образце базы данных AdventureWorksLT, созданной на этапе 1.

4. Вставьте сценарий T-SQL в SSMS, а затем выполните его. `MEMORY_OPTIMIZED = ON`Инструкции CREATE TABLE являются ключевыми. Пример:

```sql
CREATE TABLE [SalesLT].[SalesOrderHeader_inmem](
    [SalesOrderID] int IDENTITY NOT NULL PRIMARY KEY NONCLUSTERED ...,
    ...
) WITH (MEMORY_OPTIMIZED = ON);
```

### <a name="error-40536"></a>Ошибка 40536

При запуске скрипта T-SQL может появиться ошибка 40536. В таком случае выполните следующий скрипт T-SQL, чтобы проверить, поддерживает ли база данных компонент In-Memory:

```sql
SELECT DatabasePropertyEx(DB_Name(), 'IsXTPSupported');
```

Значение **0** указывает на то, что обработка в оперативной памяти не поддерживается, а значение **1** — наоборот. Чтобы установить причину проблемы, убедитесь, что для базы данных выбран уровень служб "Премиум".

### <a name="about-the-created-memory-optimized-items"></a>Сведения о созданных элементах, оптимизированных для памяти

**Таблицы**— пример содержит следующие оптимизированные для памяти таблицы:

- SalesLT.Product_inmem
- SalesLT.SalesOrderHeader_inmem
- SalesLT.SalesOrderDetail_inmem
- Demo.DemoSalesOrderHeaderSeed
- Demo.DemoSalesOrderDetailSeed

С помощью **обозревателя объектов** в SSMS-файле можно проверить оптимизированные для памяти таблицы. Щелкните правой кнопкой мыши **Таблицы** > **Фильтры** > **Параметры фильтров** > **Оптимизирован для памяти**. Значение равно 1.

Можно также отправить запрос представлений каталога:

```sql
SELECT is_memory_optimized, name, type_desc, durability_desc
    FROM sys.tables
    WHERE is_memory_optimized = 1;
```

**Скомпилированная в собственном коде хранимая процедура**: процедуру SalesLT.usp_InsertSalesOrder_inmem можно проверить с помощью запроса представления каталога.

```sql
SELECT uses_native_compilation, OBJECT_NAME(object_id), definition
    FROM sys.sql_modules
    WHERE uses_native_compilation = 1;
```

&nbsp;

### <a name="run-the-sample-oltp-workload"></a>Запуск образца рабочей нагрузки OLTP

Единственное различие между двумя следующими *хранимыми процедурами* состоит в том, что первая процедура использует оптимизированные для памяти версии таблиц, а вторая — обычные таблицы на диске:

- SalesLT **.** usp_InsertSalesOrder **_inmem**
- SalesLT **.** usp_InsertSalesOrder **_ondisk**

В этом разделе описано, как с помощью удобной служебной программы **ostress.exe** можно выполнить две хранимые процедуры в режиме нагрузочного теста. При этом вы можете сравнить время выполнения этих нагрузочных тестов.

При запуске программы ostress.exe рекомендуется передавать значения параметров:

- -n100 — для выполнения большого количества одновременных подключений;
- -r500 — для многократного (сотни раз) выполнения каждого цикла подключения.

Тем не менее проверку работоспособности можно выполнить и с помощью гораздо меньших значений, например -n10 и -r50.

### <a name="script-for-ostressexe"></a>Скрипт для ostress.exe

В этом разделе приведен скрипт T-SQL, внедренный в командную строку ostress.exe. Этот скрипт использует элементы, созданные ранее с помощью установленного скрипта T-SQL.

Следующий скрипт вставляет образец заказа на продажу с пятью позициями строки в следующие оптимизированные для памяти *таблицы*:

- SalesLT.SalesOrderHeader_inmem
- SalesLT.SalesOrderDetail_inmem

```sql
DECLARE
    @i int = 0,
    @od SalesLT.SalesOrderDetailType_inmem,
    @SalesOrderID int,
    @DueDate datetime2 = sysdatetime(),
    @CustomerID int = rand() * 8000,
    @BillToAddressID int = rand() * 10000,
    @ShipToAddressID int = rand() * 10000;

INSERT INTO @od
    SELECT OrderQty, ProductID
    FROM Demo.DemoSalesOrderDetailSeed
    WHERE OrderID= cast((rand()*60) as int);

WHILE (@i < 20)
begin;
    EXECUTE SalesLT.usp_InsertSalesOrder_inmem @SalesOrderID OUTPUT,
        @DueDate, @CustomerID, @BillToAddressID, @ShipToAddressID, @od;
    SET @i = @i + 1;
end
```

Чтобы создать версию представленного выше скрипта T-SQL для ostress.exe для таблиц *на диске*, измените оба вхождения подстроки *_inmem* на *_ondisk*. Эти замены влияют на имена таблиц и хранимых процедур.

#### <a name="install-rml-utilities-and-ostress"></a>Установка служебных программ RML и `ostress`

В идеале вам следует запланировать запуск ostress.exe на виртуальной машине Azure. Вам нужно создать [виртуальную машину Azure](https://azure.microsoft.com/documentation/services/virtual-machines/) в том же географическом регионе Azure, в котором находится база данных AdventureWorksLT. Но вы также можете запустить программу ostress.exe и на переносном компьютере.

На виртуальной машине (или в другом размещении) установите служебные программы RML, которые включают в себя ostress.exe.

Дополнительные сведения можно найти в разделе

- Обсуждение программы ostress.exe представлено в статье [Пример базы данных для выполняющейся в памяти OLTP](/sql/relational-databases/in-memory-oltp/sample-database-for-in-memory-oltp).
- Или ознакомьтесь со статьей [Пример базы данных для выполняющейся в памяти OLTP](/sql/relational-databases/in-memory-oltp/sample-database-for-in-memory-oltp).
- Можно также прочитать [блог об установке ostress.exe](https://techcommunity.microsoft.com/t5/sql-server-support/cumulative-update-2-to-the-rml-utilities-for-microsoft-sql/ba-p/317910).

<!--
dn511655.aspx is for SQL 2014,
[Extensions to AdventureWorks to Demonstrate In-Memory OLTP]
(https://msdn.microsoft.com/library/dn511655&#x28;v=sql.120&#x29;.aspx)

whereas for SQL 2016+
[Sample Database for In-Memory OLTP]
(https://msdn.microsoft.com/library/mt465764.aspx)
-->

#### <a name="run-the-_inmem-stress-workload-first"></a>Запуск тестовой рабочей нагрузки *_inmem*

Вы можете использовать окно *командной строки RML* , чтобы запустить командную строку ostress.exe. Параметры командной строки указывают следующие действия, которые необходимо выполнить программе `ostress`:

- параллельно выполнять 100 подключений (-n100);
- заставлять каждое подключение запускать сценарий T-SQL 50 раз (-r50).

```
ostress.exe -n100 -r50 -S<servername>.database.windows.net -U<login> -P<password> -d<database> -q -Q"DECLARE @i int = 0, @od SalesLT.SalesOrderDetailType_inmem, @SalesOrderID int, @DueDate datetime2 = sysdatetime(), @CustomerID int = rand() * 8000, @BillToAddressID int = rand() * 10000, @ShipToAddressID int = rand()* 10000; INSERT INTO @od SELECT OrderQty, ProductID FROM Demo.DemoSalesOrderDetailSeed WHERE OrderID= cast((rand()*60) as int); WHILE (@i < 20) begin; EXECUTE SalesLT.usp_InsertSalesOrder_inmem @SalesOrderID OUTPUT, @DueDate, @CustomerID, @BillToAddressID, @ShipToAddressID, @od; set @i += 1; end"
```

Выполнить предыдущую команду ostress.exe можно так.

1. Чтобы удалить все данные, вставленные в ходе предыдущих запусков, сбросьте содержимое базы данных, выполнив следующую команду в SSMS.

    ``` tsql
    EXECUTE Demo.usp_DemoReset;
    ```

2. Скопируйте текст предыдущей командной строки ostress.exe в буфер обмена.

3. Замените `<placeholders>` для параметров -S, -U, -P, и -d правильными фактическими значениями.

4. В окне командной строки RML запустите измененную командную строку.

#### <a name="result-is-a-duration"></a>Результат — это длительность выполнения теста

При завершении программа `ostress.exe` записывает значение длительности выполнения в последней строке выходных данных в окне командной строки RML. Например, более короткий тестовый запуск длится около 1,5 минут:

`11/12/15 00:35:00.873 [0x000030A8] OSTRESS exiting normally, elapsed time: 00:01:31.867`

#### <a name="reset-edit-for-_ondisk-then-rerun"></a>Сброс базы данных, изменение значения *_ondisk* и повторный запуск

Получив результат выполнения *_inmem*, выполните следующие действия для запуска *_ondisk*:

1. Выполните сброс базы данных, запустив следующую команду в SSMS. Она удалит все данные, вставленные в ходе предыдущего запуска.

   ```sql
   EXECUTE Demo.usp_DemoReset;
   ```

2. Измените командную строку ostress.exe, заменив все вхождения *_inmem* на *_ondisk*.

3. Перезапустите ostress.exe еще раз и запишите результат (длительность выполнения).

4. Еще раз выполните сброс базы данных, чтобы корректно удалить большой объем тестовых данных.

#### <a name="expected-comparison-results"></a>Ожидаемые результаты сравнения

Выполняющиеся в памяти тесты демонстрируют **9-кратное** повышение производительности для упрощенных рабочих нагрузок. При этом программа `ostress` выполняется на виртуальной машине Azure, расположенной в том же регионе Azure, что и база данных.

<a id="install_analytics_manuallink" name="install_analytics_manuallink"></a>

&nbsp;

## <a name="2-install-the-in-memory-analytics-sample"></a>2. Установка образца In-Memory Analytics

В этом разделе вы сравните результаты ввода-вывода и статистические данные при использовании индекса columnstore и традиционного индекса сбалансированного дерева.

Для анализа в режиме реального времени с использованием рабочей нагрузки OLTP зачастую лучше использовать некластеризованный индекс columnstore. Дополнительные сведения см. в статье [Руководство по индексам columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview).

### <a name="prepare-the-columnstore-analytics-test"></a>Подготовка тестирования аналитики с помощью columnstore

1. С помощью портала Azure создайте из образца новую базу данных AdventureWorksLT.
   - Используйте такое же имя.
   - Выберите любой уровень служб категории «Премиум».

2. Скопируйте [sql_inmemory_analytics_sample](https://raw.githubusercontent.com/microsoft/sql-server-samples/master/samples/features/in-memory-database/in-memory-oltp/t-sql-scripts/sql_in-memory_analytics_sample.sql) в буфер обмена.
   - Скрипт T-SQL создаст необходимые объекты обработки в оперативной памяти в образце базы данных AdventureWorksLT, созданной на этапе 1.
   - Скрипт создаст таблицу измерений и две таблицы фактов. Таблицы фактов заполняются 3,5 млн строк.
   - Выполнение скрипта может занять до 15 минут.

3. Вставьте сценарий T-SQL в SSMS, а затем выполните его. Важным является ключевое слово **COLUMNSTORE** в инструкции **CREATE INDEX**, как и в:<br/>`CREATE NONCLUSTERED COLUMNSTORE INDEX ...;`

4. Задайте для базы данных AdventureWorksLT уровень совместимости 130:<br/>`ALTER DATABASE AdventureworksLT SET compatibility_level = 130;`

    Уровень 130 не имеет прямого отношения к компонентам In-Memory. При этом уровень 130 обычно обеспечивает более высокую скорость обработки запросов, чем уровень 120.

#### <a name="key-tables-and-columnstore-indexes"></a>Ключевые таблицы и индексы columnstore

- dbo.FactResellerSalesXL_CCI — это таблица, включающая в себя кластеризованный индекс columnstore с дополнительными возможностями сжатия на уровне *данных*.

- dbo.FactResellerSalesXL_PageCompressed — это таблица с эквивалентным обычным кластеризованным индексом, который сжат только на уровне *страницы*.

#### <a name="key-queries-to-compare-the-columnstore-index"></a>Ключевые запросы для сравнения с индексом columnstore

Существует [несколько типов запросов T-SQL, которые можно выполнить](https://raw.githubusercontent.com/microsoft/sql-server-samples/master/samples/features/in-memory-database/in-memory-oltp/t-sql-scripts/clustered_columnstore_sample_queries.sql) для анализа повышения производительности. На втором шаге скрипта T-SQL обратите внимание на следующую пару запросов. Они отличаются только одной строкой.

- `FROM FactResellerSalesXL_PageCompressed a`
- `FROM FactResellerSalesXL_CCI a`

Кластеризованный индекс columnstore включен в таблицу FactResellerSalesXL\_CCI.

Следующий фрагмент скрипта T-SQL отображает статистику для ввода-вывода, а также время выполнения запроса (TIME) для каждой таблицы.

```sql
/*********************************************************************
Step 2 -- Overview
-- Page Compressed BTree table v/s Columnstore table performance differences
-- Enable actual Query Plan in order to see Plan differences when Executing
*/
-- Ensure Database is in 130 compatibility mode
ALTER DATABASE AdventureworksLT SET compatibility_level = 130
GO

-- Execute a typical query that joins the Fact Table with dimension tables
-- Note this query will run on the Page Compressed table, Note down the time
SET STATISTICS IO ON
SET STATISTICS TIME ON
GO

SELECT c.Year
    ,e.ProductCategoryKey
    ,FirstName + ' ' + LastName AS FullName
    ,count(SalesOrderNumber) AS NumSales
    ,sum(SalesAmount) AS TotalSalesAmt
    ,Avg(SalesAmount) AS AvgSalesAmt
    ,count(DISTINCT SalesOrderNumber) AS NumOrders
    ,count(DISTINCT a.CustomerKey) AS CountCustomers
FROM FactResellerSalesXL_PageCompressed a
INNER JOIN DimProduct b ON b.ProductKey = a.ProductKey
INNER JOIN DimCustomer d ON d.CustomerKey = a.CustomerKey
Inner JOIN DimProductSubCategory e on e.ProductSubcategoryKey = b.ProductSubcategoryKey
INNER JOIN DimDate c ON c.DateKey = a.OrderDateKey
GROUP BY e.ProductCategoryKey,c.Year,d.CustomerKey,d.FirstName,d.LastName
GO
SET STATISTICS IO OFF
SET STATISTICS TIME OFF
GO


-- This is the same Prior query on a table with a clustered columnstore index CCI
-- The comparison numbers are even more dramatic the larger the table is (this is an 11 million row table only)
SET STATISTICS IO ON
SET STATISTICS TIME ON
GO
SELECT c.Year
    ,e.ProductCategoryKey
    ,FirstName + ' ' + LastName AS FullName
    ,count(SalesOrderNumber) AS NumSales
    ,sum(SalesAmount) AS TotalSalesAmt
    ,Avg(SalesAmount) AS AvgSalesAmt
    ,count(DISTINCT SalesOrderNumber) AS NumOrders
    ,count(DISTINCT a.CustomerKey) AS CountCustomers
FROM FactResellerSalesXL_CCI a
INNER JOIN DimProduct b ON b.ProductKey = a.ProductKey
INNER JOIN DimCustomer d ON d.CustomerKey = a.CustomerKey
Inner JOIN DimProductSubCategory e on e.ProductSubcategoryKey = b.ProductSubcategoryKey
INNER JOIN DimDate c ON c.DateKey = a.OrderDateKey
GROUP BY e.ProductCategoryKey,c.Year,d.CustomerKey,d.FirstName,d.LastName
GO

SET STATISTICS IO OFF
SET STATISTICS TIME OFF
GO
```

При использовании кластеризованного индекса рост производительности для этого запроса в базе данных ценовой категории P2 может увеличиться примерно в 9 раз по сравнению с традиционным индексом. При использовании кластеризованного индекса в базе данных ценовой категории P15 можно ожидать рост производительности примерно в 57 раз.

## <a name="next-steps"></a>Дальнейшие действия

- [Краткое руководство 1. Технологии выполнения OLTP в памяти для повышения производительности службы Transact-SQL](/sql/relational-databases/in-memory-oltp/survey-of-initial-areas-in-in-memory-oltp)

- [Повышение производительности приложений в базе данных SQL с помощью выполняющейся в памяти OLTP](in-memory-oltp-configure.md)

- [Мониторинг хранилища OLTP в памяти](in-memory-oltp-monitor-space.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

### <a name="deeper-information"></a>Подробные сведения

- Узнайте, как [Quorum удваивает ключевую рабочую нагрузку на базу данных при одновременном сокращении DTU на 70 % благодаря использованию In-Memory OLTP для Базы данных SQL Azure](https://customers.microsoft.com/story/quorum-doubles-key-databases-workload-while-lowering-dtu-with-sql-database).

- [In-Memory OLTP in Azure SQL Database](https://azure.microsoft.com/blog/in-memory-oltp-in-azure-sql-database/) (Выполняющаяся в памяти OLTP в Базе данных SQL Azure)

- [Learn about In-Memory OLTP](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization) (Знакомство с In-Memory OLTP)

- [Руководство по индексам columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview)

- [Начало работы с Columnstore для получения операционной аналитики в реальном времени](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)

- Технический документ с [рекомендациями по распространенным шаблонам рабочих нагрузок и миграции](/previous-versions/dn673538(v=msdn.10)) включает описание шаблонов рабочих нагрузок, для которых выполняющаяся OLTP обычно обеспечивает значительное повышение производительности.

#### <a name="application-design"></a>Проектирование приложений

- [In-Memory OLTP (оптимизация в памяти)](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)

- [Повышение производительности приложений в базе данных SQL с помощью выполняющейся в памяти OLTP](in-memory-oltp-configure.md)

#### <a name="tools"></a>Инструменты

- [Портал Azure](https://portal.azure.com/)

- [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms)

- [SQL Server Data Tools (SSDT)](/sql/ssdt/download-sql-server-data-tools-ssdt)