---
title: Перемещение данных в виртуальную машину сервера SQL Server — командный процесс обработки и анализа данных
description: Перемещение данных из неструктурированных файлов или из локальной SQL Server в SQL Server на виртуальной машине Azure.
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
ms.openlocfilehash: c80a90b07e25942e751d52cafa47f6e3e94852ab
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93320331"
---
# <a name="move-data-to-sql-server-on-an-azure-virtual-machine"></a>Перемещение данных в SQL Server на виртуальной машине Azure

В этой статье описаны варианты перемещения данных из неструктурированных файлов (в формате CSV или TSV) или с локального сервера SQL Server на сервер SQL Server на виртуальной машине Azure. Эти задачи перемещения данных в облако являются этапом процесса обработки и анализа данных группы.

Описание вариантов перемещения данных в базу данных SQL Azure для машинного обучения см. в [этой статье](move-sql-azure.md).

В следующей таблице перечислены варианты перемещения данных в SQL Server на виртуальной машине Azure.

| <b>ИСТОЧНИКА</b> | <b>НАЗНАЧЕНИЕ: SQL Server на виртуальной машине Azure</b> |
| --- | --- |
| <b>Неструктурированный файл</b> |1. <a href="#insert-tables-bcp">программа для выполнения операций с массовым копированием из командной строки (BCP) </a><br> 2. <a href="#insert-tables-bulkquery">SQL — запрос на выполнение инструкций групповой вставки </a><br> 3. <a href="#sql-builtin-utilities">графические Встроенные служебные программы в SQL Server</a> |
| <b>Локальная SQL Server</b> |1. <a href="#deploy-a-sql-server-database-to-a-microsoft-azure-vm-wizard">развертывание SQL Server базы данных в мастере Microsoft Azure виртуальной машины</a><br> 2. <a href="#export-flat-file">Экспорт в неструктурированный файл </a><br> 3. <a href="#sql-migration">Мастер миграции баз данных SQL </a> <br> 4. <a href="#sql-backup">резервное копирование и восстановление базы данных </a><br> |

В этом документе предполагается, что команды SQL выполняются из SQL Server Management Studio или Visual Studio обозреватель базы данных.

> [!TIP]
> Вы также можете воспользоваться [фабрикой данных Azure](https://azure.microsoft.com/services/data-factory/), чтобы создать и назначить конвейер, который переносит данные в виртуальную машину SQL Server в среде Azure. Дополнительные сведения см. в статье [Перемещение данных с помощью действия копирования](../../data-factory/copy-activity-overview.md).
>
>

## <a name="prerequisites"></a><a name="prereqs"></a>Предварительные требования
Для выполнения действий, описанных в этом учебнике, вам необходимо следующее.

* **Подписка Azure**. Если у вас нет подписки, вы можете зарегистрироваться для получения [бесплатной пробной версии](https://azure.microsoft.com/pricing/free-trial/).
* **Учетная запись хранения Azure**. Учетная запись хранения Azure будет использоваться для хранения данных в этом учебнике. Если у вас ее нет, см. раздел [Создание учетной записи хранения](../../storage/common/storage-account-create.md). После создания учетной записи хранения необходимо получить ключ, используемый для доступа к хранилищу. См. [раздел Управление ключами доступа учетной записи хранения](../../storage/common/storage-account-keys-manage.md).
* Подготовленный **SQL Server на виртуальной машине Azure**. Инструкции см. в статье [Настройка SQL Server на виртуальной машине Azure как сервера IPython Notebook для расширенной аналитики](../data-science-virtual-machine/overview.md).
* Установленная и настроенная локальная среда **Azure PowerShell**. Инструкции см. в статье [Приступая к работе с командлетами Azure PowerShell](/powershell/azure/).

## <a name="moving-data-from-a-flat-file-source-to-sql-server-on-an-azure-vm"></a><a name="filesource_to_sqlonazurevm"></a> Перемещение данных из неструктурированного файла на сервер SQL Server на виртуальной машине Azure
Если данные (упорядоченные по строкам и столбцам) хранятся в неструктурированном файле, этот файл можно переместить в виртуальную машину SQL Server в Azure с помощью следующих средств:

1. [Программа командной строки для выполнения операций с массовым копированием (BCP)](#insert-tables-bcp)
2. [SQL-запрос на массовую вставку](#insert-tables-bulkquery)
3. [Графические служебные программы, встроенные в SQL Server (импорт или экспорт, службы SSIS)](#sql-builtin-utilities)

### <a name="command-line-bulk-copy-utility-bcp"></a><a name="insert-tables-bcp"></a>Служебная программа командной строки для массового копирования (BCP)
BCP — это служебная программа командной строки, устанавливаемая вместе с SQL Server, которая предоставляет один из самых быстрых способов перемещения данных. Он работает во всех трех вариантах SQL Server (локальных SQL Server, SQL Azure и виртуальных машин SQL Server в Azure).

> [!NOTE]
> **Где следует размещать данные для программы BCP?**  
> Это не обязательно, но если файлы, содержащие источник данных, расположены на одном компьютере с целевым сервером SQL, это позволяет ускорить перемещение (скорость сети и скорость ввода-вывода на локальном диске). Перемещать неструктурированные файлы, содержащие данные, на компьютер с SQL Server можно с помощью различных инструментов копирования файлов, таких как [AZCopy](../../storage/common/storage-use-azcopy-v10.md), [обозреватель хранилищ Azure](https://storageexplorer.com/) или средства копирования и вставки по протоколу удаленного рабочего стола (RDP) ОС Windows.
>
>

1. Убедитесь, что базы данных и таблицы создаются в целевой базе данных SQL Server. В примере ниже показано, как выполнить эту задачу с помощью команд `Create Database` и `Create Table`.

    ```sql
    CREATE DATABASE <database_name>
    
    CREATE TABLE <tablename>
    (
        <columnname1> <datatype> <constraint>,
        <columnname2> <datatype> <constraint>,
        <columnname3> <datatype> <constraint>
    )
    ```

1. Создайте файл форматирования, описывающий схему для таблицы, выполнив следующую команду из командной строки компьютера, на котором установлена программа bcp.

    `bcp dbname..tablename format nul -c -x -f exportformatfilename.xml -S servername\sqlinstance -T -t \t -r \n`
1. Вставьте данные в базу данных с помощью команды bcp, которая должна работать из командной строки, если SQL Server установлена на одном компьютере:

    `bcp dbname..tablename in datafilename.tsv -f exportformatfilename.xml -S servername\sqlinstancename -U username -P password -b block_size_to_move_in_single_attempt -t \t -r \n`

> **Оптимизация операций вставки BCP.** Подробные сведения об оптимизации таких операций вставки см. в статье [Рекомендации по оптимизации массового импорта данных](/previous-versions/sql/sql-server-2008-r2/ms177445(v=sql.105)).
>
>

### <a name="parallelizing-inserts-for-faster-data-movement"></a><a name="insert-tables-bulkquery-parallel"></a>Распараллеливание операций вставки для ускорения перемещения данных
Если перемещаемые данные велики, можно ускорить выполнение нескольких команд BCP в параллельном режиме в сценарии PowerShell.

> [!NOTE]
> **Прием данных большого размера.** Чтобы оптимизировать загрузку больших и очень больших наборов данных, разбейте таблицы логических и физических баз данных на разделы, используя несколько групп файлов и секционированных таблиц. Дополнительные сведения о создании секционированных таблиц и загрузке в них данных см. в статье [Параллельный массовый импорт данных с использованием таблиц секционирования SQL](parallel-load-sql-partitioned-tables.md).
>
>

В следующем примере сценария PowerShell показаны параллельные операции вставки при использовании программы BCP.

```powershell
$NO_OF_PARALLEL_JOBS=2

Set-ExecutionPolicy RemoteSigned #set execution policy for the script to execute
# Define what each job does
$ScriptBlock = {
    param($partitionnumber)

    #Explicitly using SQL username password
    bcp database..tablename in datafile_path.csv -F 2 -f format_file_path.xml -U username@servername -S tcp:servername -P password -b block_size_to_move_in_single_attempt -t "," -r \n -o path_to_outputfile.$partitionnumber.txt

    #Trusted connection w.o username password (if you are using windows auth and are signed in with that credentials)
    #bcp database..tablename in datafile_path.csv -o path_to_outputfile.$partitionnumber.txt -h "TABLOCK" -F 2 -f format_file_path.xml  -T -b block_size_to_move_in_single_attempt -t "," -r \n
}


# Background processing of all partitions
for ($i=1; $i -le $NO_OF_PARALLEL_JOBS; $i++)
{
    Write-Debug "Submit loading partition # $i"
    Start-Job $ScriptBlock -Arg $i      
}


# Wait for it all to complete
While (Get-Job -State "Running")
{
    Start-Sleep 10
    Get-Job
}

# Getting the information back from the jobs
Get-Job | Receive-Job
Set-ExecutionPolicy Restricted #reset the execution policy
```

### <a name="bulk-insert-sql-query"></a><a name="insert-tables-bulkquery"></a>SQL-запрос на массовую вставку
[SQL-запрос на массовую вставку](/sql/t-sql/statements/bulk-insert-transact-sql) можно использовать для импорта в базу данных информации из файлов на основе строк или столбцов (поддерживаемые типы описаны в статье [Подготовка данных к массовому экспорту или импорту (SQL Server)](/sql/relational-databases/import-export/prepare-data-for-bulk-export-or-import-sql-server)).

Ниже приведены некоторые примеры команд для массовой вставки:  

1. Перед импортом проанализируйте данные, задайте все необходимые пользовательские настройки и убедитесь, что в базе данных SQL Server для всех специальных полей, таких как дата, задан одинаковый формат. Ниже показано, как задать формат даты "год-месяц-день" (если в данных содержатся даты в формате "год-месяц-день"):

    ```sql
    SET DATEFORMAT ymd;
    ```
2. Импорт данных с помощью операторов массового импорта:

    ```sql
    BULK INSERT <tablename>
    FROM
    '<datafilename>'
    WITH
    (
        FirstRow = 2,
        FIELDTERMINATOR = ',', --this should be column separator in your data
        ROWTERMINATOR = '\n'   --this should be the row separator in your data
    )
    ```

### <a name="built-in-utilities-in-sql-server"></a><a name="sql-builtin-utilities"></a>Служебные программы, встроенные в SQL Server
SQL Server Integration Services (SSIS) можно использовать для импорта данных в виртуальную машину SQL Server в Azure из неструктурированного файла.
Службы SSIS доступны в двух средах Studio. Дополнительные сведения см. в статье [Разработка Integration Services (SSIS) и средства управления](/sql/integration-services/integration-services-ssis-development-and-management-tools):

* сведения о SQL Server Data Tools см. в статье [Скачать SQL Server Data Tools (SSDT)](/sql/ssdt/download-sql-server-data-tools-ssdt);  
* сведения о мастере импорта и экспорта см. в статье [Импорт и экспорт источников данных, поддерживаемых SQL Server](/sql/integration-services/import-export-data/import-and-export-data-with-the-sql-server-import-and-export-wizard).

## <a name="moving-data-from-on-premises-sql-server-to-sql-server-on-an-azure-vm"></a><a name="sqlonprem_to_sqlonazurevm"></a>Перемещение данных с локального сервера SQL Server на сервер SQL Server на виртуальной машине Azure
Вы также можете использовать следующие стратегии миграции:

1. [Мастер развертывания Базы данных SQL Server на виртуальной машине Microsoft Azure](#deploy-a-sql-server-database-to-a-microsoft-azure-vm-wizard)
2. [Экспорт в неструктурированный файл](#export-flat-file)
3. [Мастер миграции баз данных SQL](#sql-migration)
4. [Резервное копирование и восстановление базы данных](#sql-backup)

Описание каждого из этих параметров приведено ниже.

### <a name="deploy-a-sql-server-database-to-a-microsoft-azure-vm-wizard"></a>Мастер развертывания Базы данных SQL Server на виртуальной машине Microsoft Azure
**Мастер развертывания базы данных SQL Server на виртуальной машине Microsoft Azure** представляет собой простой и рекомендуемый способ перемещения данных из локального экземпляра SQL Server на сервер SQL Server на виртуальной машине Azure. Подробные инструкции, а также описание других вариантов см. в статье [Миграция базы данных SQL Server в экземпляр SQL Server на виртуальной машине Azure](../../azure-sql/virtual-machines/windows/migrate-to-vm-from-sql-server.md).

### <a name="export-to-flat-file"></a><a name="export-flat-file"></a>Экспорт в неструктурированный файл
Для массового экспорта данных с локального сервера SQL Server можно использовать различные способы, которые описаны в разделе [Массовый импорт и экспорт данных (SQL Server)](/sql/relational-databases/import-export/bulk-import-and-export-of-data-sql-server) . В этом документе в качестве примера описано использование программы массового копирования (BCP). После экспорта данных в неструктурированный файл этот файл можно импортировать в другой SQL Server с помощью операции массового импорта.

1. Экспортируйте данные из локальной SQL Server в файл с помощью служебной программы bcp следующим образом.

    `bcp dbname..tablename out datafile.tsv -S    servername\sqlinstancename -T -t \t -t \n -c`
2. Создайте базу данных и таблицу в виртуальной машине SQL Server в Azure с помощью команд `create database` и `create table` для схемы таблицы, экспортированной на шаге 1.
3. Создайте файл форматирования для описания схемы таблицы экспортируемых или импортируемых данных. Информация о файле форматирования приведена в разделе [Создание файла форматирования (SQL Server)](/sql/relational-databases/import-export/create-a-format-file-sql-server).

    Создание файла форматирования при запуске программы BCP с компьютера SQL Server

    `bcp dbname..tablename format nul -c -x -f exportformatfilename.xml -S servername\sqlinstance -T -t \t -r \n`

    Создание файла форматирования при удаленном запуске BCP для SQL Server

    `bcp dbname..tablename format nul -c -x -f  exportformatfilename.xml  -U username@servername.database.windows.net -S tcp:servername -P password  --t \t -r \n`
4. С помощью любого из методов, описанных в разделе [Перемещение данных из файла источника](#filesource_to_sqlonazurevm) , переместите данные неструктурированных файлов в SQL Server.

### <a name="sql-database-migration-wizard"></a><a name="sql-migration"></a>Мастер миграции баз данных SQL
[Мастер миграции баз данных SQL Server](https://sqlazuremw.codeplex.com/) представляет собой удобный способ перемещения данных между двумя экземплярами SQL Server. Пользователь может сопоставлять схемы данных между источниками и таблицами назначения, выбирать типы столбцов и выполнять другие функции. Этот мастер использует программу массового копирования (BCP). Ниже приведен снимок экрана приветствия мастера миграции баз данных SQL.  

![Мастер миграции SQL Server][2]

### <a name="database-back-up-and-restore"></a><a name="sql-backup"></a>Архивация и восстановление базы данных
SQL Server поддерживает:

1. [Архивацию и восстановление базы данных](/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases) (экспорт в локальный файл и экспорт BACPAC-файла в большой двоичный объект) и [приложения уровня данных](/sql/relational-databases/data-tier-applications/data-tier-applications) (с использованием BACPAC-файла).
2. Возможность непосредственного создания SQL Server виртуальных машин в Azure с помощью скопированной базы данных или копирования в существующую базу данных SQL. Дополнительные сведения см. в статье [Use the Copy Database Wizard](/sql/relational-databases/databases/use-the-copy-database-wizard).

Ниже приведен снимок экрана параметров архивации и восстановления базы данных в среде SQL Server Management Studio.

![Средство импорта данных из SQL Server][1]

## <a name="resources"></a>Ресурсы
[Миграция базы данных в SQL Server на виртуальной машине Azure](../../azure-sql/virtual-machines/windows/migrate-to-vm-from-sql-server.md)

[Обзор SQL Server в виртуальных машинах Azure](../../azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md)

[1]: ./media/move-sql-server-virtual-machine/sqlserver_builtin_utilities.png
[2]: ./media/move-sql-server-virtual-machine/database_migration_wizard.png