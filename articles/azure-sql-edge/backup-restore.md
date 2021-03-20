---
title: Резервное копирование и восстановление баз данных в Azure SQL ребро
description: Узнайте о возможностях резервного копирования и восстановления в Azure SQL.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: 114be810ea50f984c3211291691b4c4dd45ac2c7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93395246"
---
# <a name="back-up-and-restore-databases-in-azure-sql-edge"></a>Резервное копирование и восстановление баз данных в Azure SQL ребро 

Azure SQL Server строится на основе последних версий ядро СУБД Microsoft SQL. Она предоставляет аналогичные возможности резервного копирования и восстановления базы данных, доступные в SQL Server на Linux и SQL Server, выполняемых в контейнерах. Компонент резервного копирования и восстановления обеспечивает необходимую защиту данных, хранящихся в пограничных базах данных SQL Azure. 

Чтобы снизить риск необратимой потери данных, следует периодически создавать резервные копии баз данных, чтобы постоянно сохранять изменения в данных. Хорошо спланированная стратегия резервного копирования и восстановления защищает базы от потери данных, вызванной разными сбоями. Протестируйте стратегию, восстановив набор резервных копий, а затем восстанавливая базу данных, чтобы подготовиться к аварийному реагированию.

Дополнительные сведения о том, почему резервное копирование важно, см. в статье [резервное копирование и восстановление баз данных SQL Server](/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases/).

Azure SQL ребро позволяет выполнять резервное копирование и восстановление из локального хранилища и больших двоичных объектов Azure. Дополнительные сведения см. в статьях [SQL Server резервное копирование и восстановление с помощью хранилища BLOB-объектов Azure](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service/) и [SQL Server резервное копирование на URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url).

## <a name="back-up-a-database-in-azure-sql-edge"></a>Резервное копирование базы данных в Azure SQL ребро

Azure SQL ребр поддерживает те же типы резервных копий, что и SQL Server. Полный список см. в разделе [Общие сведения о резервном копировании](/sql/relational-databases/backup-restore/backup-overview-sql-server/).

> [!IMPORTANT] 
> Базы данных, созданные в Azure SQL ребро, по умолчанию используют простую модель восстановления. Таким образом, нельзя выполнять резервное копирование журналов в этих базах данных. Если вам нужно это сделать, вам потребуется администратор, чтобы изменить модель восстановления базы данных на модель полного восстановления. Полный список моделей восстановления, поддерживаемых SQL Server, см. в разделе [Общие сведения о модели восстановления](/sql/relational-databases/backup-restore/recovery-models-sql-server#RMov).

### <a name="back-up-to-local-disk"></a>Резервное копирование на локальный диск

В следующем примере используется `BACKUP DATABASE` команда Transact-SQL для создания резервной копии базы данных в контейнере. В этом примере создается новая папка с именем *BACKUP* для хранения файлов резервной копии.

1. Создайте папку для файлов резервных копий. Выполните эту команду на узле, где выполняется контейнер Azure SQL. В следующей команде замените **<AzureSQLEdge_Container_Name>** именем контейнера Azure SQL в развертывании.

    ```bash
    sudo docker exec -it <AzureSQLEdge_Container_Name> mkdir /var/opt/mssql/backup
    ```

2. Подключитесь к экземпляру SQL Azure с помощью SQL Server Management Studio (SSMS) или с помощью Azure Data Studio. Выполните `BACKUP DATABASE` команду, чтобы создать резервную копию пользовательской базы данных. В следующем примере выполняется резервное копирование базы данных *ироноресиликапредиктион* .

    ```sql
    BACKUP DATABASE [IronOreSilicaPrediction] 
    TO DISK = N'/var/opt/mssql/backup/IronOrePredictDB.bak' 
    WITH NOFORMAT, NOINIT,  NAME = N'IronOreSilicaPrediction-Full Database Backup', 
    SKIP, NOREWIND, NOUNLOAD, COMPRESSION,  STATS = 10
    GO
    ```

3. После выполнения команды в случае успешного резервного копирования базы данных в разделе результатов SSMS или Azure Data Studio будут отображаться сообщения, аналогичные приведенным ниже.

    ```txt
    10 percent processed.
    20 percent processed.
    30 percent processed.
    40 percent processed.
    50 percent processed.
    60 percent processed.
    70 percent processed.
    80 percent processed.
    90 percent processed.
    100 percent processed.
    Processed 51648 pages for database 'IronOreSilicaPrediction', file 'IronOreSilicaPrediction' on file 1.
    Processed 2 pages for database 'IronOreSilicaPrediction', file 'IronOreSilicaPrediction_log' on file 1.
    BACKUP DATABASE successfully processed 51650 pages in 3.588 seconds (112.461 MB/sec).

    Completion time: 2020-04-09T23:54:48.4957691-07:00
    ```

### <a name="back-up-to-url"></a>Резервное копирование по URL-адресу

SQL Azure для пограничных вычислений поддерживает резервное копирование как в страничные, так и в блочные BLOB-объекты. Дополнительные сведения см. [в статье резервное копирование в блочные BLOB-объекты и страничные](/sql/relational-databases/backup-restore/sql-server-backup-to-url#blockbloborpageblob)большие двоичные объекты. В следующем примере выполняется резервное копирование базы данных *ироноресиликапредиктион* в блочный BLOB-объект. 

1. Чтобы настроить резервное копирование для блокировки больших двоичных объектов, сначала создайте маркер подписанного URL-адрес (SAS), который можно использовать для создания учетных данных SQL Server на границе Azure SQL. Скрипт создает SAS, связанный с хранимой политикой доступа. Дополнительные сведения см. [в разделе подписанные URL — часть 1. Основные сведения о модели SAS](../storage/common/storage-sas-overview.md). Кроме того, скрипт записывает команду T-SQL, необходимую для создания учетных данных в SQL Server. В следующем сценарии предполагается, что у вас уже есть подписка Azure с учетной записью хранения и контейнер хранилища для резервных копий.

    ```PowerShell
    # Define global variables for the script  
    $subscriptionName='<your subscription name>'   # the name of subscription name you will use
    $resourcegroupName = '<your resource group name>' # the name of resource group you will use
    $storageAccountName= '<your storage account name>' # the storage account name you will use for backups
    $containerName= '<your storage container name>'  # the storage container name to which you will attach the SAS policy with its SAS token  
    $policyName = 'SASPolicy' # the name of the SAS policy  

    # adds an authenticated Azure account for use in the session
    Login-AzAccount

    # set the tenant, subscription and environment for use in the rest of
    Select-AzSubscription -Subscription $subscriptionName

    # Generate the SAS token
    $sa = Get-AzStorageAccount -ResourceGroupName $resourcegroupName -Name $storageAccountName 
    $storagekey = Get-AzStorageAccountKey -ResourceGroupName $resourcegroupName -Name $storageAccountName 
    $storageContext = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storagekey[0].Value
    $cbc = Get-AzStorageContainer -Name $containerName -Context $storageContext
    $policy = New-AzStorageContainerStoredAccessPolicy -Container $containerName -Policy $policyName -Context $storageContext -ExpiryTime $(Get-Date).ToUniversalTime().AddYears(10) -Permission "rwld"
    $sas = New-AzStorageContainerSASToken -Policy $policyName -Context $storageContext -Container $containerName
    Write-Host 'Shared Access Signature= '$($sas.Substring(1))''

    # Outputs the Transact SQL to the clipboard and to the screen to create the credential using the Shared Access Signature  
    Write-Host 'Credential T-SQL'  
    $tSql = "CREATE CREDENTIAL [{0}] WITH IDENTITY='Shared Access Signature', SECRET='{1}'" -f $cbc.CloudBlobContainer.Uri.AbsoluteUri,$sas.Substring(1)
    $tSql | clip  
    Write-Host $tSql
    ```

    После успешного выполнения скрипта скопируйте `CREATE CREDENTIAL` команду в инструмент запроса. Затем подключитесь к экземпляру SQL Server и выполните команду, чтобы создать учетные данные с помощью SAS.

2. Подключитесь к экземпляру SQL Azure с помощью SSMS или Azure Data Studio и создайте учетные данные с помощью команды из предыдущего шага. Обязательно замените `CREATE CREDENTIAL` команду на фактические выходные данные предыдущего шага.

    ```sql
    IF NOT EXISTS  
    (SELECT * FROM sys.credentials
    WHERE name = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>')  
    CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>]
       WITH IDENTITY = 'SHARED ACCESS SIGNATURE',  
       SECRET = '<SAS_TOKEN>';
    ```

3. Следующая команда выполняет резервное копирование базы данных *IronOreSilicaPrediction* в контейнер службы хранилища Azure.

    ```sql
    BACKUP DATABASE IronOreSilicaPrediction
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mycontainername>/IronOreSilicaPrediction.bak'
    With MAXTRANSFERSIZE = 4194304,BLOCKSIZE=65536;  
    GO
    ```

## <a name="restore-a-database-in-azure-sql-edge"></a>Восстановление базы данных в Azure SQL ребро

В Azure SQL Server можно выполнить восстановление с локального диска, из сетевой папки или учетной записи хранилища BLOB-объектов Azure. Дополнительные сведения о восстановлении и восстановлении в SQL Server см. в статье [Общие сведения](/sql/relational-databases/backup-restore/restore-and-recovery-overview-sql-server)о восстановлении и восстановлении. Общие сведения о простой модели восстановления в SQL Server см. в разделе [полное восстановление базы данных (простая модель восстановления)](/sql/relational-databases/backup-restore/complete-database-restores-simple-recovery-model).

> [!IMPORTANT] 
> Базы данных, созданные в Azure SQL Server, не могут быть восстановлены в экземпляре Microsoft SQL Server или Azure SQL. Кроме того, база данных, созданная на Microsoft SQL Server или Azure SQL, может быть восстановлена в Azure SQL Server, если база данных не содержит функций, которые не поддерживаются Azure SQL Server. 

### <a name="restore-from-a-local-disk"></a>Восстановление с локального диска

В этом примере используется резервная копия *ироноресиликапредиктион* , созданная в предыдущем примере. Теперь вы восстановите ее как новую базу данных с другим именем.

1. Если файл резервной копии базы данных еще не существует в контейнере, можно использовать следующую команду, чтобы скопировать файл в контейнер. В следующем примере предполагается, что файл резервной копии находится на локальном узле и копируется в папку/Вар/ОПТ/мсскл/баккуп в контейнер пограничных данных SQL Azure с именем *SQL1*.

    ```bash
    sudo docker cp IronOrePredictDB.bak sql1:/var/opt/mssql/backup
    ```

2. Подключитесь к экземпляру Azure SQL с помощью среды SSMS или Azure Data Studio, чтобы выполнить команду RESTORE. В следующем примере восстанавливается **иронорепредиктдб. bak** для создания новой базы данных, **IronOreSilicaPrediction_2**.

    ```sql
    Restore FilelistOnly from disk = N'/var/opt/mssql/backup/IronOrePredictDB.bak'

    Restore Database IronOreSilicaPrediction_2
    From disk = N'/var/opt/mssql/backup/IronOrePredictDB.bak'
    WITH MOVE 'IronOreSilicaPrediction' TO '/var/opt/mssql/data/IronOreSilicaPrediction_Primary_2.mdf',
    MOVE 'IronOreSilicaPrediction_log' TO '/var/opt/mssql/data/IronOreSilicaPrediction_Primary_2.ldf'
    ```

3. После выполнения команды Restore, если операция восстановления прошла успешно, в окне Вывод будут отображаться сообщения, аналогичные приведенным ниже. 

    ```txt
    Processed 51648 pages for database 'IronOreSilicaPrediction_2', file 'IronOreSilicaPrediction' on file 1.
    Processed 2 pages for database 'IronOreSilicaPrediction_2', file 'IronOreSilicaPrediction_log' on file 1.
    RESTORE DATABASE successfully processed 51650 pages in 6.543 seconds (61.670 MB/sec).

    Completion time: 2020-04-13T23:49:21.1600986-07:00
    ```

### <a name="restore-from-url"></a>Восстановление из URL-адреса

SQL Azure для пограничных вычислений также поддерживает восстановление базы данных из учетной записи службы хранилища Azure. Вы можете выполнить восстановление из блочных BLOB-объектов или страничных резервных копий. В следующем примере файл резервной копии базы данных *IronOreSilicaPrediction_2020_04_16. bak* в блочном BLOB-объекте восстанавливается для создания базы данных, *IronOreSilicaPrediction_3*.

```sql
RESTORE DATABASE IronOreSilicaPrediction_3
FROM URL = 'https://mystorageaccount.blob.core.windows.net/mysecondcontainer/IronOreSilicaPrediction_2020_04_16.bak'
WITH MOVE 'IronOreSilicaPrediction' TO '/var/opt/mssql/data/IronOreSilicaPrediction_Primary_3.mdf', 
MOVE 'IronOreSilicaPrediction_log' TO '/var/opt/mssql/data/IronOreSilicaPrediction_Primary_3.ldf',
STATS = 10;
```