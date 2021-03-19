---
title: Учебник по загрузке данных из Azure Data Lake Storage
description: Используйте инструкцию COPY для загрузки данных из Azure Data Lake Storage для выделенных пулов SQL.
services: synapse-analytics
author: gaursa
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/20/2020
ms.author: gaursa
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: ca57c6200cf7006a89be4b1fd621974559e5b514
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104606129"
---
# <a name="load-data-from-azure-data-lake-storage-into-dedicated-sql-pools-in-azure-synapse-analytics"></a>Загрузка данных из Azure Data Lake Storage в выделенные пулы SQL в Azure синапсе Analytics

В этом руководство описано, как использовать [инструкцию Copy](/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest&preserve-view=true) для загрузки данных из Azure Data Lake Storage. Краткие примеры использования инструкции COPY во всех методах проверки подлинности см. в следующей документации: [Безопасная загрузка данных с помощью выделенных ПУЛОВ SQL](./quickstart-bulk-load-copy-tsql-examples.md).

> [!NOTE]  
> Чтобы отправить отзыв или сообщить о проблемах в инструкции COPY, отправьте сообщение электронной почты по следующему списку рассылки: sqldwcopypreview@service.microsoft.com .
>
> [!div class="checklist"]
>
> * Создайте целевую таблицу для загрузки данных из Azure Data Lake Storage.
> * Создайте инструкцию COPY для загрузки данных в хранилище данных.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="before-you-begin"></a>Перед началом

Перед началом работы с этим руководством скачайте и установите последнюю версию [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) (SSMS).

Для работы с этим руководством необходимы указанные ниже компоненты.

* Выделенный пул SQL. См. раздел [Создание выделенного пула SQL и данных запросов](create-data-warehouse-portal.md).
* Учетная запись Data Lake Storage. Узнайте, как [приступить к работе с Azure Data Lake Storage](../../data-lake-store/data-lake-store-get-started-portal.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Для этой учетной записи хранения необходимо настроить или указать один из следующих учетных данных для загрузки: ключ учетной записи хранения, ключ подписанного URL-адрес (SAS), пользователь приложения Azure Directory или пользователь AAD с соответствующей ролью Azure для учетной записи хранения.

## <a name="create-the-target-table"></a>Создание целевой таблицы

Подключитесь к выделенному пулу SQL и создайте целевую таблицу, в которую будет загружена таблица. В этом примере мы создаем таблицу измерения Product.

```sql
-- A: Create the target table
-- DimProduct
CREATE TABLE [dbo].[DimProduct]
(
    [ProductKey] [int] NOT NULL,
    [ProductLabel] [nvarchar](255) NULL,
    [ProductName] [nvarchar](500) NULL
)
WITH
(
    DISTRIBUTION = HASH([ProductKey]),
    CLUSTERED COLUMNSTORE INDEX
    --HEAP
);
```


## <a name="create-the-copy-statement"></a>Создание инструкции COPY

Подключитесь к выделенному пулу SQL и выполните инструкцию COPY. Полный список примеров см. в следующей документации: [Безопасная загрузка данных с помощью выделенных ПУЛОВ SQL](./quickstart-bulk-load-copy-tsql-examples.md).

```sql
-- B: Create and execute the COPY statement

COPY INTO [dbo].[DimProduct] 
--The column list allows you map, omit, or reorder input file columns to target table columns. 
--You can also specify the default value when there is a NULL value in the file.
--When the column list is not specified, columns will be mapped based on source and target ordinality
(
    ProductKey default -1 1,
    ProductLabel default 'myStringDefaultWhenNull' 2,
    ProductName default 'myStringDefaultWhenNull' 3
)
--The storage account location where you data is staged
FROM 'https://storageaccount.blob.core.windows.net/container/directory/'
WITH 
(
   --CREDENTIAL: Specifies the authentication method and credential access your storage account
   CREDENTIAL = (IDENTITY = '', SECRET = ''),
   --FILE_TYPE: Specifies the file type in your storage account location
   FILE_TYPE = 'CSV',
   --FIELD_TERMINATOR: Marks the end of each field (column) in a delimited text (CSV) file
   FIELDTERMINATOR = '|',
   --ROWTERMINATOR: Marks the end of a record in the file
   ROWTERMINATOR = '0x0A',
   --FIELDQUOTE: Specifies the delimiter for data of type string in a delimited text (CSV) file
   FIELDQUOTE = '',
   ENCODING = 'UTF8',
   DATEFORMAT = 'ymd',
   --MAXERRORS: Maximum number of reject rows allowed in the load before the COPY operation is canceled
   MAXERRORS = 10,
   --ERRORFILE: Specifies the directory where the rejected rows and the corresponding error reason should be written
   ERRORFILE = '/errorsfolder',
) OPTION (LABEL = 'COPY: ADLS tutorial');
```

## <a name="optimize-columnstore-compression"></a>Оптимизация сжатия columnstore

По умолчанию таблицы определяются как кластеризованный индекс columnstore. После завершения загрузки для некоторых строк данных может не выполняться сжатие в индекс columnstore.  Это может происходить по ряду причин. Чтобы узнать больше, ознакомьтесь с [управлением индексами columnstore](sql-data-warehouse-tables-index.md).

Чтобы оптимизировать производительность запросов и сжатие columnstore после загрузки, перестройте таблицу, чтобы настроить принудительное сжатие всех строк таблиц индексом columnstore.

```sql

ALTER INDEX ALL ON [dbo].[DimProduct] REBUILD;

```

## <a name="optimize-statistics"></a>Оптимизация статистики

Одностолбцовую статистику рекомендуется создавать сразу после загрузки. Существует несколько вариантов статистики. Например, при создании одностолбцовой статистики для каждого столбца перестройка всех статистических данных может занять длительное время. Если вам известно, что некоторые столбцы не будут входить в предикаты запросов, можно пропустить создание статистики для этих столбцов.

Если вам потребуется создать одностолбцовую статистику для каждого столбца в каждой таблице, можно использовать пример кода хранимой процедуры `prc_sqldw_create_stats` из статьи, посвященной [статистике](sql-data-warehouse-tables-statistics.md).

Следующий пример кода — хорошая отправная точка для создания статистики. Он создает одностолбцовую статистику для каждого столбца в таблице измерения и для каждого соответствующего столбца в таблицах фактов. Одно- или многостолбцовую статистику в другие столбцы таблицы фактов можно добавить позже.

## <a name="achievement-unlocked"></a>Победа!

Вы успешно загрузили данные в хранилище данных. Отличная работа!

## <a name="next-steps"></a>Дальнейшие действия
Загрузка данных является первым шагом к разработке решения для хранения данных на основе Azure Synapse Analytics. Ознакомьтесь с нашими ресурсами разработки.

> [!div class="nextstepaction"]
> [Узнайте, как проектировать таблицы для хранения данных](sql-data-warehouse-tables-overview.md).

Дополнительные примеры загрузки и ссылки см. в следующей документации:
- [Справочная документация по инструкции COPY](/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest&preserve-view=true#syntax)
- [Примеры копирования для каждого метода проверки подлинности](./quickstart-bulk-load-copy-tsql-examples.md)
- [Быстрый Запуск копирования для одной таблицы](./quickstart-bulk-load-copy-tsql.md)