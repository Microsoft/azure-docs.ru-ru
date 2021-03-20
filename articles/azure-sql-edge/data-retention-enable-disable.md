---
title: Включение и отключение политик хранения данных с помощью Azure SQL ребр
description: Узнайте, как включить и отключить политики хранения данных в Azure SQL Server.
keywords: SQL Server, хранение данных
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/04/2020
ms.openlocfilehash: 5b8dd911952a63ba8775f27a6128ff61e849e823
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96861431"
---
# <a name="enable-and-disable-data-retention-policies"></a>Включение и отключение политик хранения данных

В этом разделе описывается включение и отключение политик хранения данных для базы данных и таблицы. 

## <a name="enable-data-retention-for-a-database"></a>Включение хранения данных для базы данных

В следующем примере показано, как включить хранение данных с помощью [инструкции ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql-set-options).

```sql
ALTER DATABASE [<DatabaseName>] SET DATA_RETENTION  ON;
```

## <a name="check-if-data-retention-is-enabled-for-a-database"></a>Проверка включения хранения данных для базы данных

Следующую команду можно использовать для проверки того, включено ли хранение данных для базы данных.
```sql
SELECT is_data_retention_enabled, name
FROM sys.databases;
```

## <a name="enable-data-retention-for-a-table"></a>Включение хранения данных для таблицы

Для каждой таблицы, для которой необходимо автоматически очищать данные, необходимо включить хранение данных. Если для базы данных и таблицы включено хранение данных, то фоновая задача системы будет периодически сканировать таблицу для обнаружения и удаления устаревших (устаревших) строк. Хранение данных может быть включено для таблицы либо во время создания таблицы с помощью инструкции [CREATE TABLE](/sql/t-sql/statements/create-table-transact-sql) , либо с помощью [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql).

В следующем примере показано, как включить хранение данных для таблицы с помощью инструкции [CREATE TABLE](/sql/t-sql/statements/create-table-transact-sql). 

```sql
CREATE TABLE [dbo].[data_retention_table] 
(
[dbdatetime2] datetime2(7), 
[product_code] int, 
[value] char(10),  
CONSTRAINT [pk_current_data_retention_table] PRIMARY KEY CLUSTERED ([product_code])
) WITH (DATA_DELETION = ON ( FILTER_COLUMN = [dbdatetime2], RETENTION_PERIOD = 1 day ) )
```

`WITH (DATA_DELETION = ON ( FILTER_COLUMN = [dbdatetime2], RETENTION_PERIOD = 1 day ) )`Часть команды CREATE TABLE задает хранение данных в таблице. Команда использует следующие обязательные параметры. 

- DATA_DELETION — указывает, включено ли хранение данных.
- FILTER_COLUMN-Name для столбца в таблице, который будет использоваться для определения того, являются ли строки устаревшими. Столбец фильтра может быть только столбцом со следующими типами данных 
    - Дата
    - SmallDateTime
    - Дата и время
    - datetime2
    - DateTimeOffset
- RETENTION_PERIOD — целочисленное значение, за которым следует дескриптор единицы. Допустимые единицы: день, дни, неделя, недели, месяц, месяцы, год и годы.

В следующем примере показано, как включить хранение данных для таблицы с помощью [инструкции ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql).  

```sql
Alter Table [dbo].[data_retention_table]
SET (DATA_DELETION = On (FILTER_COLUMN = [timestamp], RETENTION_PERIOD = 1 day))
```

## <a name="check-if-data-retention-is-enabled-for-a-table"></a>Проверка включения хранения данных для таблицы

Для проверки таблиц, для которых включено хранение данных, можно использовать следующую команду:

```sql
select name, data_retention_period, data_retention_period_unit from sys.tables
```

Значение data_retention_period =-1 и data_retention_period_unit как бесконечное, указывает на то, что для таблицы не задано хранение данных.

Следующий запрос можно использовать для задания столбца, используемого в качестве filter_column для хранения данных. 

```sql
Select name from sys.columns
where is_data_deletion_filter_column =1 
and object_id = object_id(N'dbo.data_retention_table', N'U')
```

## <a name="correlating-db-and-table-data-retention-settings"></a>Сопоставление параметров хранения баз данных и таблиц

Параметр хранения данных в базе данных и таблице используется совместно, чтобы определить, будет ли выполняться автоматическая очистка для устаревших строк в таблицах или нет. 

|Параметр базы данных | Параметр таблицы | Поведение |
|----------------|--------------|----------|
| OFF | OFF | Политика хранения данных отключена, и автоматическая и ручная очистка устаревших записей отключена.|
| OFF | ON  | Политика хранения данных включена для таблицы. Автоматическая очистка устаревших записей отключена, однако для очистки устаревших записей можно использовать метод очистки вручную. |
| ON | OFF | Политика хранения данных включена на уровне базы данных. Однако, поскольку этот параметр отключен на уровне таблицы, очистка устаревших строк на основе удержания отсутствует.|
| ON | ON | Политика хранения данных включена как для базы данных, так и для таблиц. Автоматическая очистка устаревших записей включена. |

## <a name="disable-data-retention-on-a-table"></a>Отключение хранения данных в таблице 

Хранение данных можно отключить в таблице с помощью [инструкции ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql). Следующую команду можно использовать для отключения хранения данных в таблице.

```sql
Alter Table [dbo].[data_retention_table]
Set (DATA_DELETION = OFF)
```

## <a name="disable-data-retention-on-a-database"></a>Отключение хранения данных в базе данных

Хранение данных можно отключить в таблице с помощью [инструкции ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql-set-options). Следующую команду можно использовать для отключения хранения данных в базе данных.

```sql
ALTER DATABASE <DatabaseName> SET DATA_RETENTION  OFF;
```

## <a name="next-steps"></a>Дальнейшие действия
- [Хранение данных и автоматическая очистка данных](data-retention-overview.md)
- [Управление историческими данными с помощью политики хранения](data-retention-cleanup.md)
