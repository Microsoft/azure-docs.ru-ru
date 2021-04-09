---
title: Таблицы общих метаданных
description: Azure Synapse Analytics предоставляет общую модель метаданных, в которой к таблице, созданной в бессерверном пуле Apache Spark, можно будет получить доступ из бессерверного пула SQL и выделенного пула SQL без необходимости дублировать данные.
services: sql-data-warehouse
author: MikeRys
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: metadata
ms.date: 05/01/2020
ms.author: mrys
ms.reviewer: jrasnick
ms.custom: devx-track-csharp
ms.openlocfilehash: a8080720480beaeb7bc8692f2dcddddad5da0e3c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102548467"
---
# <a name="azure-synapse-analytics-shared-metadata-tables"></a>Таблицы общих метаданных Azure Synapse Analytics


Azure Synapse Analytics позволяет различным вычислительным системам рабочих областей использовать базы данных и таблицы на основе Parquet совместно с пулами Apache Spark и бессерверным пулом SQL.

После того как задание Spark создаст базу данных, в ней можно с помощью Spark создать таблицы с форматом хранения Parquet. Имена таблиц будут преобразованы в нижний регистр, и их нужно будет запрашивать с использованием нижнего регистра. Эти таблицы немедленно становятся доступными для выполнения запросов через любой пул Spark в рабочей области Azure Synapse. Их также можно использовать из любого задания Spark при наличии соответствующих разрешений.

Таблицы, создаваемые и управляемые в Spark, а также внешние таблицы Spark доступны как внешние таблицы с тем же именем в соответствующей синхронизированной базе данных в бессерверном пуле SQL. Дополнительные сведения можно найти в разделе [Exposing a Spark table in SQL](#expose-a-spark-table-in-sql) (Предоставление таблицы Spark в SQL).

Так как таблицы синхронизируются с бессерверным пулом SQL асинхронно, они появляются с некоторой задержкой.

## <a name="manage-a-spark-created-table"></a>Управление созданной в Spark таблицей

Используйте Spark для управления созданными в Spark базами данных. Например, можно удалить таблицу из задания бессерверного пула Apache Spark или создать в ней таблицы средствами Spark.

Если вы создадите объекты в такой базе данных из бессерверного пула SQL или попытаетесь удалить базу данных, операция завершится ошибкой. Исходную базу данных Spark нельзя изменить с помощью бессерверного пула SQL.

## <a name="expose-a-spark-table-in-sql"></a>Предоставление таблицы Spark в SQL

### <a name="shared-spark-tables"></a>Общие таблицы Spark

Spark предоставляет два типа таблиц, которые Azure Synapse автоматически делает доступными в SQL.

- Управляемые таблицы

  Spark предоставляет множество форматов для хранения данных в управляемых таблицах, в том числе TEXT, CSV, JSON, JDBC, PARQUET, ORC, HIVE, DELTA и LIBSVM. Обычно эти файлы размещаются в каталоге `warehouse`, где хранятся данные управляемой таблицы.

- Внешние таблицы

  Spark также предлагает возможности для создания внешних таблиц на основе существующих данных: вы можете указать параметр `LOCATION` или использовать формат Hive. Такие внешние таблицы могут иметь разные форматы данных, включая Parquet.

В настоящее время Azure Synapse предоставляет в совместное использование для подсистем SQL только те управляемые и внешние таблицы Spark, данные в которых хранятся в формате Parquet. Таблицы на основе других форматов не синхронизируются автоматически. Но вы можете явным образом включить для базы данных SQL синхронизацию таких таблиц во внешнюю таблицу, если подсистема SQL поддерживает базовый формат таблицы.

### <a name="share-spark-tables"></a>Совместное использование таблиц Spark

Управляемые и внешние таблицы Spark предоставляются в ядре SQL для совместного использования как внешние таблицы со следующими свойствами:

- источником данных внешней таблицы SQL считается тот источник данных, который представляет папку с этой таблицей Spark;
- для внешней таблицы SQL используется формат файла Parquet;
- для учетных данных внешней таблицы SQL используется сквозная аутентификация.

Так как все имена таблиц Spark являются допустимыми именами таблиц в SQL, а все имена столбцов Spark являются допустимыми именами столбцов в SQL, для внешней таблицы SQL сохраняются все имена таблиц и столбцов Spark.

Типы данных в таблицах Spark отличаются от типов данных в подсистемах Synapse SQL. В следующей таблице указаны сопоставления между типами данных в таблице Spark и в SQL.

| Тип данных Spark | Тип данных SQL | Комментарии |
|---|---|---|
| `byte`      | `smallint`       ||
| `short`     | `smallint`       ||
| `integer`   |    `int`            ||
| `long`      |    `bigint`         ||
| `float`     | `real`           |<!-- need precision and scale-->|
| `double`    | `float`          |<!-- need precision and scale-->|
| `decimal`      | `decimal`        |<!-- need precision and scale-->|
| `timestamp` |    `datetime2`      |<!-- need precision and scale-->|
| `date`      | `date`           ||
| `string`    |    `varchar(max)`   | С параметрами сортировки `Latin1_General_100_BIN2_UTF8` |
| `binary`    |    `varbinary(max)` ||
| `boolean`   |    `bit`            ||
| `array`     |    `varchar(max)`   | Сериализуется в JSON с параметрами сортировки `Latin1_General_100_BIN2_UTF8` |
| `map`       |    `varchar(max)`   | Сериализуется в JSON с параметрами сортировки `Latin1_General_100_BIN2_UTF8` |
| `struct`    |    `varchar(max)`   | Сериализуется в JSON с параметрами сортировки `Latin1_General_100_BIN2_UTF8` |

<!-- TODO: Add precision and scale to the types mentioned above -->

## <a name="security-model"></a>Модель безопасности

Базы данных и таблицы Spark, а также их синхронизированные представления в ядре SQL защищаются на уровне базового хранилища. Так как в настоящее время для них не поддерживаются разрешения на уровне объекта, все объекты будут доступны для просмотра в обозревателе объектов.

Субъект безопасности, который создает управляемую таблицу, становится ее владельцем и получает все права на такую таблицу, а также на ее базовые папки и файлы. Кроме того, владелец базы данных автоматически становится совладельцем таблицы.

Если вы создадите внешнюю таблицу Spark или SQL со сквозной аутентификацией, данные будут защищаться только на уровнях папок и файлов. При любом запросе к внешней таблице такого типа идентификатор безопасности отправителя запроса передается в файловую систему, которая проверяет наличие прав доступа.

Дополнительные сведения о настройке разрешений для папок и файлов общей базы данных в Azure Synapse Analytics см. в [этой статье](database.md).

## <a name="examples"></a>Примеры

### <a name="create-a-managed-table-backed-by-parquet-in-spark-and-query-from-serverless-sql-pool"></a>Создание управляемой таблицы на основе Parquet в Spark и обращение к ней из бессерверного пула SQL

В этом сценарии существует база данных Spark с именем `mytestdb`. См. раздел [Создание базы данных Spark и подключение к ней с помощью бессерверного пула SQL](database.md#create-and-connect-to-spark-database-with-serverless-sql-pool).

Создайте управляемую таблицу Spark с помощью SparkSQL, выполнив следующую команду:

```sql
    CREATE TABLE mytestdb.myparquettable(id int, name string, birthdate date) USING Parquet
```

Эта команда создает таблицу `myparquettable` в базе данных `mytestdb`. Имена таблиц будут преобразованы в нижний регистр. После короткой задержки вы увидите таблицу в бессерверном пуле SQL. Например, выполните следующую инструкцию из бессерверного пула SQL.

```sql
    USE mytestdb;
    SELECT * FROM sys.tables;
```

Убедитесь, что `myparquettable` есть в результатах.

>[!NOTE]
>Если таблица использует формат хранения, отличный от Parquet, она не будет синхронизирована.

Теперь вставьте в таблицу некоторые значения из Spark, например выполнив следующие инструкции Spark на языке C# в записной книжке C#:

```csharp
using Microsoft.Spark.Sql.Types;

var data = new List<GenericRow>();

data.Add(new GenericRow(new object[] { 1, "Alice", new Date(2010, 1, 1)}));
data.Add(new GenericRow(new object[] { 2, "Bob", new Date(1990, 1, 1)}));

var schema = new StructType
    (new List<StructField>()
        {
            new StructField("id", new IntegerType()),
            new StructField("name", new StringType()),
            new StructField("birthdate", new DateType())
        }
    );

var df = spark.CreateDataFrame(data, schema);
df.Write().Mode(SaveMode.Append).InsertInto("mytestdb.myparquettable");
```

Теперь вы можете считывать данные из бессерверного пула SQL следующим образом:

```sql
SELECT * FROM mytestdb.dbo.myparquettable WHERE name = 'Alice';
```

В качестве результата возвращается следующая строка:

```
id | name | birthdate
---+-------+-----------
1 | Alice | 2010-01-01
```

### <a name="create-an-external-table-backed-by-parquet-in-spark-and-query-from-serverless-sql-pool"></a>Создание внешней таблицы на основе Parquet в Spark и обращение к ней из бессерверного пула SQL

В этом примере вы создаете внешнюю таблицу Spark на основе файлов данных Parquet, которые мы получили в предыдущем примере для управляемой таблицы.

Например, выполните такой код SparkSQL:

```sql
CREATE TABLE mytestdb.myexternalparquettable
    USING Parquet
    LOCATION "abfss://<fs>@arcadialake.dfs.core.windows.net/synapse/workspaces/<synapse_ws>/warehouse/mytestdb.db/myparquettable/"
```

Замените заполнитель `<fs>` именем файловой системы, которое используется по умолчанию для рабочей области, а заполнитель `<synapse_ws>` — именем рабочей области Synapse, в которой вы выполняете этот пример.

Приведенный выше пример создает таблицу `myextneralparquettable` в базе данных `mytestdb`. После короткой задержки вы увидите таблицу в бессерверном пуле SQL. Например, выполните следующую инструкцию из бессерверного пула SQL.

```sql
USE mytestdb;
SELECT * FROM sys.tables;
```

Убедитесь, что `myexternalparquettable` есть в результатах.

Теперь вы можете считывать данные из бессерверного пула SQL следующим образом:

```sql
SELECT * FROM mytestdb.dbo.myexternalparquettable WHERE name = 'Alice';
```

В качестве результата возвращается следующая строка:

```
id | name | birthdate
---+-------+-----------
1 | Alice | 2010-01-01
```

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь со статьей [Общие метаданные Azure Synapse Analytics](overview.md)
- Ознакомьтесь со статьей [Общие базы метаданных Azure Synapse Analytics](database.md)


