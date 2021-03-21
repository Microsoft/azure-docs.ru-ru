---
title: Параллельный массовый импорт данных с использованием таблиц разделов SQL — командный процесс обработки и анализа данных
description: Создание секционированных таблиц для быстрого параллельного массового импорта данных в базу данных SQL Server.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 456e881d84697f4542f972ac0798cc95a3455b3c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93322415"
---
# <a name="build-and-optimize-tables-for-fast-parallel-import-of-data-into-a-sql-server-on-an-azure-vm"></a>Создание и оптимизация таблиц для быстрого параллельного импорта данных в SQL Server на виртуальной машине Azure

В этой статье описывается создание секционированных таблиц для быстрого параллельного массового импорта данных в Базу данных SQL Server. Для загрузки и перемещения больших данных в базу данных SQL импорт данных в базу данных SQL и последующие запросы можно улучшить с помощью *секционированных таблиц и представлений*. 

## <a name="create-a-new-database-and-a-set-of-filegroups"></a>Создание новой базы данных и набора групп файлов
* [Создайте базу данных](/sql/t-sql/statements/create-database-transact-sql), если она еще не существует.
* Добавьте группы файлов базы данных в базу данных, которая содержит секционированные физические файлы. 
* Это можно сделать с помощью команды [Создать базу данных](/sql/t-sql/statements/create-database-transact-sql) (если базы данных нет) или [Изменить базу данных](/sql/t-sql/statements/alter-database-transact-sql-set-options) (если база данных существует).
* При необходимости добавьте один или несколько файлов в каждую группу файлов базы данных.
  
  > [!NOTE]
  > Укажите целевую группу файлов, которая будет содержать данные для этого секционирования, и имена физических файлов базы данных, в которых хранятся данные группы файлов.
  > 
  > 

В следующем примере создается новая база данных с тремя группами файлов в отличие от основных групп и групп журналов, содержащих по одному физическому файлу. Файлы базы данных создаются в папке данных SQL Server по умолчанию, настроенной в экземпляре SQL Server. Дополнительные сведения о расположении файлов по умолчанию см. в статье [Расположение файлов для экземпляра по умолчанию и именованных экземпляров SQL Server](/sql/sql-server/install/file-locations-for-default-and-named-instances-of-sql-server).

```sql
   DECLARE @data_path nvarchar(256);
   SET @data_path = (SELECT SUBSTRING(physical_name, 1, CHARINDEX(N'master.mdf', LOWER(physical_name)) - 1)
      FROM master.sys.master_files
      WHERE database_id = 1 AND file_id = 1);

   EXECUTE ('
      CREATE DATABASE <database_name>
         ON  PRIMARY 
        ( NAME = ''Primary'', FILENAME = ''' + @data_path + '<primary_file_name>.mdf'', SIZE = 4096KB , FILEGROWTH = 1024KB ), 
         FILEGROUP [filegroup_1] 
        ( NAME = ''FileGroup1'', FILENAME = ''' + @data_path + '<file_name_1>.ndf'' , SIZE = 4096KB , FILEGROWTH = 1024KB ), 
         FILEGROUP [filegroup_2] 
        ( NAME = ''FileGroup2'', FILENAME = ''' + @data_path + '<file_name_2>.ndf'' , SIZE = 4096KB , FILEGROWTH = 1024KB ), 
         FILEGROUP [filegroup_3] 
        ( NAME = ''FileGroup3'', FILENAME = ''' + @data_path + '<file_name_3>.ndf'' , SIZE = 102400KB , FILEGROWTH = 10240KB ) 
         LOG ON 
        ( NAME = ''LogFileGroup'', FILENAME = ''' + @data_path + '<log_file_name>.ldf'' , SIZE = 1024KB , FILEGROWTH = 10%)
    ')
```

## <a name="create-a-partitioned-table"></a>Создание секционированной таблицы
Для создания секционированных таблиц в соответствии со схемой данных, сопоставленной с группами файлов базы данных (созданы на предыдущем шаге), необходимо сначала создать схему и функцию секционирования. При массовом импорте данных в секционированные таблицы записи распределяются между группами файлов в соответствии со схемой секционирования, как описано ниже.

### <a name="1-create-a-partition-function"></a>1. Создание функции секционирования
[Создайте функцию секционирования](/sql/t-sql/statements/create-partition-function-transact-sql). Эта функция определяет диапазон значений или границ для каждой отдельной секционированной таблицы, например, чтобы ограничить секции по месяцам (поле\_datetime\_) в 2013 году:
  
```sql
   CREATE PARTITION FUNCTION <DatetimeFieldPFN>(<datetime_field>)  
      AS RANGE RIGHT FOR VALUES (
         '20130201', '20130301', '20130401',
         '20130501', '20130601', '20130701', '20130801',
         '20130901', '20131001', '20131101', '20131201' )
```

### <a name="2-create-a-partition-scheme"></a>2. Создание схемы секционирования
[Создайте схему функционирования](/sql/t-sql/statements/create-partition-scheme-transact-sql). Эта схема сопоставляет каждый диапазон секционирования в функции секционирования с физической группой файлов, например:
  
```sql
      CREATE PARTITION SCHEME <DatetimeFieldPScheme> AS  
        PARTITION <DatetimeFieldPFN> TO (
        <filegroup_1>, <filegroup_2>, <filegroup_3>, <filegroup_4>,
        <filegroup_5>, <filegroup_6>, <filegroup_7>, <filegroup_8>,
        <filegroup_9>, <filegroup_10>, <filegroup_11>, <filegroup_12> )
```
 
Чтобы проверить диапазоны на практике в каждой секции в соответствии с функцией и схемой, выполните следующий запрос:
  
```sql
   SELECT psch.name as PartitionScheme,
            prng.value AS PartitionValue,
            prng.boundary_id AS BoundaryID
   FROM sys.partition_functions AS pfun
   INNER JOIN sys.partition_schemes psch ON pfun.function_id = psch.function_id
   INNER JOIN sys.partition_range_values prng ON prng.function_id=pfun.function_id
   WHERE pfun.name = <DatetimeFieldPFN>
```

### <a name="3-create-a-partition-table"></a>3. Создание таблицы секционирования
[Создайте секционированные таблицы](/sql/t-sql/statements/create-table-transact-sql) в соответствии со схемой данных и укажите схему секционирования и поле ограничений, используемые для секционирования таблицы, например:
  
```sql
   CREATE TABLE <table_name> ( [include schema definition here] )
        ON <TablePScheme>(<partition_field>)
```

Дополнительные сведения см. в статье [Создание секционированных таблиц и индексов](/sql/relational-databases/partitions/create-partitioned-tables-and-indexes).

## <a name="bulk-import-the-data-for-each-individual-partition-table"></a>Массовый импорт данных для каждой отдельной таблицы секционирования

* Можно использовать BCP, BULK INSERT или другие средства, например [мастер миграции SQL Server](https://sqlazuremw.codeplex.com/). В приведенном ниже примере используется метод BCP.
* [Измените базу данных](/sql/t-sql/statements/alter-database-transact-sql-set-options), изменив схему ведения журнала транзакций на BULK_LOGGED, чтобы минимизовать нагрузку при ведении журнала, например:
  
   ```sql
      ALTER DATABASE <database_name> SET RECOVERY BULK_LOGGED
   ```
* Чтобы ускорить загрузку данных, запустите параллельные операции массового импорта. Советы по ускорению выполнения операций с массовым импортом больших данных в SQL Server базы данных см. [в разделе Загрузка 1 ТБ менее 1 часа](/archive/blogs/sqlcat/load-1tb-in-less-than-1-hour).

В следующем сценарии PowerShell приведен пример параллельной загрузки данных с использованием BCP.

```powershell
    # Set database name, input data directory, and output log directory
    # This example loads comma-separated input data files
    # The example assumes the partitioned data files are named as <base_file_name>_<partition_number>.csv
    # Assumes the input data files include a header line. Loading starts at line number 2.

    $dbname = "<database_name>"
    $indir  = "<path_to_data_files>"
    $logdir = "<path_to_log_directory>"

    # Select authentication mode
    $sqlauth = 0

    # For SQL authentication, set the server and user credentials
    $sqlusr = "<user@server>"
    $server = "<tcp:serverdns>"
    $pass   = "<password>"

    # Set number of partitions per table - Should match the number of input data files per table
    $numofparts = <number_of_partitions>

    # Set table name to be loaded, basename of input data files, input format file, and number of partitions
    $tbname = "<table_name>"
    $basename = "<base_input_data_filename_no_extension>"
    $fmtfile = "<full_path_to_format_file>"

    # Create log directory if it does not exist
    New-Item -ErrorAction Ignore -ItemType directory -Path $logdir

    # BCP example using Windows authentication
    $ScriptBlock1 = {
       param($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $num)
       bcp ($dbname + ".." + $tbname) in ($indir + "\" + $basename + "_" + $num + ".csv") -o ($logdir + "\" + $tbname + "_" + $num + ".txt") -h "TABLOCK" -F 2 -C "RAW" -f ($fmtfile) -T -b 2500 -t "," -r \n
    }

    # BCP example using SQL authentication
    $ScriptBlock2 = {
       param($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $num, $sqlusr, $server, $pass)
       bcp ($dbname + ".." + $tbname) in ($indir + "\" + $basename + "_" + $num + ".csv") -o ($logdir + "\" + $tbname + "_" + $num + ".txt") -h "TABLOCK" -F 2 -C "RAW" -f ($fmtfile) -U $sqlusr -S $server -P $pass -b 2500 -t "," -r \n
    }

    # Background processing of all partitions
    for ($i=1; $i -le $numofparts; $i++)
    {
       Write-Output "Submit loading trip and fare partitions # $i"
       if ($sqlauth -eq 0) {
          # Use Windows authentication
          Start-Job -ScriptBlock $ScriptBlock1 -Arg ($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $i)
       } 
       else {
          # Use SQL authentication
          Start-Job -ScriptBlock $ScriptBlock2 -Arg ($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $i, $sqlusr, $server, $pass)
       }
    }

    Get-Job

    # Optional - Wait till all jobs complete and report date and time
    date
    While (Get-Job -State "Running") { Start-Sleep 10 }
    date
```

## <a name="create-indexes-to-optimize-joins-and-query-performance"></a>Создание индексов для оптимизации производительности запросов и объединений
* Если вы извлекаете данные для моделирования из нескольких таблиц, создайте индексы для ключей объединений, чтобы повысить производительность объединений.
* [Создайте индексы](/sql/t-sql/statements/create-index-transact-sql) (кластеризованные или некластеризованные) для одной и той же целевой группы файлов каждой секции, например:
  
```sql
   CREATE CLUSTERED INDEX <table_idx> ON <table_name>( [include index columns here] )
        ON <TablePScheme>(<partition)field>)
--  or,
  
        CREATE INDEX <table_idx> ON <table_name>( [include index columns here] )
        ON <TablePScheme>(<partition)field>)
 ```
 
  > [!NOTE]
  > Вы можете создать индексы перед массовым импортом данных. Но это приведет к замедлению загрузки данных.
  > 
  > 

## <a name="advanced-analytics-process-and-technology-in-action-example"></a>Расширенный процесс аналитики и технологии в действии: пример
Полный пошаговый пример обработки и анализа данных группы с использованием общедоступного набора данных см. в статье [Процесс обработки и анализа данных группы на практике: использование SQL Server](sql-walkthrough.md).