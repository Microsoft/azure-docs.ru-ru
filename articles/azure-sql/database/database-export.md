---
title: Экспорт базы данных SQL Azure в BACPAC-файл (портал Azure)
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: Экспорт базы данных в BACPAC-файл с помощью портал Azure.
services: sql-database
ms.service: sql-db-mi
ms.subservice: data-movement
author: stevestein
ms.custom: sqldbrb=2
ms.author: sstein
ms.reviewer: ''
ms.date: 01/11/2021
ms.topic: how-to
ms.openlocfilehash: 866500e9cd9e3fe6aac6a5bfded0dbb21ab137fc
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102614281"
---
# <a name="export-to-a-bacpac-file---azure-sql-database-and-azure-sql-managed-instance"></a>Экспорт в файл BACPAC — база данных SQL Azure и Azure SQL Управляемый экземпляр

[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Если нужно экспортировать базу данных для создания архива или для перехода на другую платформу, можно экспортировать схему и данные базы данных в [BACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications#Anchor_4)-файл. BACPAC-файл — это ZIP-файл с расширением BACPAC, содержащим метаданные и данные из базы данных. BACPAC-файл можно хранить в хранилище BLOB-объектов Azure или локально в локальном расположении, а затем импортировать обратно в базу данных SQL Azure, Управляемый экземпляр Azure SQL или экземпляр SQL Server.

## <a name="considerations"></a>Рекомендации

- Чтобы экспорт был транзакционно согласованным, необходимо убедиться в том, что во время экспорта не происходит ни одной операции записи, или что вы экспортируете из [транзакционно согласованной копии](database-copy.md) базы данных.
- Максимальный размер BACPAC-файла при экспорте в хранилище BLOB-объектов составляет 200 ГБ. Для архивации BACPAC-файла большего размера выполняйте экспорт в локальное хранилище.
- Экспорт BACPAC-файла в службу хранилища Azure уровня "Премиум" с использованием методов, описанных в этой статье, не поддерживается.
- Хранилище за брандмауэром сейчас не поддерживается.
- Имя файла хранилища или входное значение для StorageURI должно содержать менее 128 символов и не может заканчиваться символом "." и не может содержать специальные символы, такие как пробел или "<, >, *,%, &,:, \, /,?". 
- Если операция экспорта длится более 20 часов, она может быть отменена. Для повышения производительности во время экспорта можно сделать следующее.

  - Временно повысить объем вычислительных ресурсов.
  - Прекратить все операции чтения и записи во время экспорта.
  - Используйте для всех больших таблиц [кластеризованный индекс](/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described) со значениями, отличными от NULL. Без кластеризованных индексов экспорт может завершиться ошибкой, если он длится больше 6–12 часов. Это обусловлено тем, что службам экспорта требуется выполнить проверку таблицы, чтобы экспортировать всю таблицу. Хороший способ определить, оптимизированы ли таблицы для экспорта, — выполнить **DBCC SHOW_STATISTICS** и убедиться, что значение *RANGE_HI_KEY* не равно NULL и имеет хорошее распределение. Дополнительную информацию см. в разделе [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql).

> [!NOTE]
> BACPAC-файлы не предназначены для операций службы архивации и восстановления. Azure автоматически создает резервные копии для каждой пользовательской базы данных. Дополнительные сведения см. в статьях [Обзор обеспечения непрерывности бизнес-процессов с помощью базы данных SQL Azure](business-continuity-high-availability-disaster-recover-hadr-overview.md) и [Автоматическое резервное копирование](automated-backups-overview.md).

## <a name="the-azure-portal"></a>Портал Azure

Экспорт BACPAC базы данных из [Azure SQL управляемый экземпляр](../managed-instance/sql-managed-instance-paas-overview.md) с помощью портал Azure в настоящее время не поддерживается. Вместо этого используйте SQL Server Management Studio или SQLPackage.

> [!NOTE]
> Компьютеры, обрабатывающие запросы на импорт и экспорт, отправленные через портал Azure или PowerShell, должны хранить BACPAC-файл и временные файлы, созданные платформой приложения уровня данных (DacFX). Требуемое дисковое пространство может быть существенно разным для баз данных одного размера и в три раза превышать размер самой базы данных. Компьютеры, на которых работает запрос на импорт или экспорт, имеют только 450GB место на локальном диске. Это означает, что некоторые запросы могут завершиться ошибкой `There is not enough space on the disk`. В качестве обходного решения можно запустить sqlpackage.exe на другом компьютере, где есть достаточно места на локальном диске. Мы советуем использовать [SqlPackage](#sqlpackage-utility) для импорта и экспорта баз данных, превышающих 150 ГБ, чтобы избежать этой проблемы.

1. Чтобы экспортировать базу данных с помощью [портала Azure](https://portal.azure.com), откройте страницу для базы данных и щелкните **Экспорт** на панели инструментов.

   ![Снимок экрана, на котором наделена кнопка "экспорт".](./media/database-export/database-export1.png)

2. Введите имя BACPAC-файла, выберите учетную запись хранения Azure и контейнер для экспорта, а также соответствующие учетные данные для доступа к базе данных-источнику. Имя для **входа администратора SQL Server** необходимо, даже если вы являетесь администратором Azure, так как администратор Azure не имеет права администратора в базе данных SQL Azure или azure SQL управляемый экземпляр.

    ![Экспорт базы данных](./media/database-export/database-export2.png)

3. Нажмите кнопку **ОК**.

4. Чтобы отслеживать ход выполнения операции экспорта, откройте страницу для сервера, содержащего экспортируемую базу данных. В разделе **Параметры** выберите **Журнал импорта и экспорта**.

   ![журнал экспорта](./media/database-export/export-history.png)

## <a name="sqlpackage-utility"></a>Служебная программа SQLPackage

Сведения о том, как экспортировать базу данных в базу данных SQL с помощью служебной программы командной строки [SqlPackage](/sql/tools/sqlpackage) , см. в разделе [Экспорт параметров и свойств](/sql/tools/sqlpackage#export-parameters-and-properties). Служебная программа SqlPackage поставляется вместе с последними версиями [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) и [SQL Server Data Tools для Visual Studio](/sql/ssdt/download-sql-server-data-tools-ssdt). Кроме того, вы можете скачать последнюю версию [SqlPackage](https://www.microsoft.com/download/details.aspx?id=53876) непосредственно из Центра загрузки Майкрософт.

Рекомендуется использовать служебную программу SqlPackage для масштабирования и обеспечения производительности в большинстве рабочих сред. Сведения о миграции из SQL Server в Базу данных SQL Azure с использованием BACPAC-файлов см. в [блоге группы консультирования клиентов SQL Server](/archive/blogs/sqlcat/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files).

В этом примере показано, как экспортировать базу данных с помощью SqlPackage.exe, используя универсальную аутентификацию Active Directory.

```cmd
SqlPackage.exe /a:Export /tf:testExport.bacpac /scs:"Data Source=apptestserver.database.windows.net;Initial Catalog=MyDB;" /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="sql-server-management-studio-ssms"></a>SQL Server Management Studio (SSMS)

Новейшие версии SQL Server Management Studio предоставляют мастер для экспорта базы данных в базе данных SQL Azure или базы данных SQL Управляемый экземпляр в BACPAC-файл. Ознакомьтесь с разделом [Export a Data-tier Application](/sql/relational-databases/data-tier-applications/export-a-data-tier-application) (Экспорт приложения уровня данных).

## <a name="powershell"></a>PowerShell

> [!NOTE]
> В настоящее время [управляемый экземпляр Azure SQL](../managed-instance/sql-managed-instance-paas-overview.md) не поддерживает экспорт базы данных в BACPAC-файл с помощью Azure PowerShell. Чтобы экспортировать управляемый экземпляр в BACPAC-файл, используйте SQL Server Management Studio или SQLPackage.

Используйте командлет [New-азсклдатабасикспорт](/powershell/module/az.sql/new-azsqldatabaseexport) , чтобы отправить запрос на экспорт базы данных в службу базы данных SQL Azure. Операция экспорта может занять некоторое время в зависимости от размера базы данных.

```powershell
$exportRequest = New-AzSqlDatabaseExport -ResourceGroupName $ResourceGroupName -ServerName $ServerName `
  -DatabaseName $DatabaseName -StorageKeytype $StorageKeytype -StorageKey $StorageKey -StorageUri $BacpacUri `
  -AdministratorLogin $creds.UserName -AdministratorLoginPassword $creds.Password
```

Чтобы проверить состояние запроса на экспорт, используйте командлет [Get-азсклдатабасеимпортекспортстатус](/powershell/module/az.sql/get-azsqldatabaseimportexportstatus) . Если выполнить его немедленно после запроса, то обычно возвращается сообщение **Состояние: выполняется**. Отображение сообщения **Состояние: выполнен** означает, что экспорт завершен.

```powershell
$exportStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $exportRequest.OperationStatusLink
[Console]::Write("Exporting")
while ($exportStatus.Status -eq "InProgress")
{
    Start-Sleep -s 10
    $exportStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $exportRequest.OperationStatusLink
    [Console]::Write(".")
}
[Console]::WriteLine("")
$exportStatus
```
## <a name="cancel-the-export-request"></a>Отменить запрос на экспорт

Ниже приведен пример команды PowerShell для работы с [базой данных Operations-Cancel API](https://docs.microsoft.com/rest/api/sql/databaseoperations/cancel) или [командой PowerShell остановить-азсклдатабасеактивити](https://docs.microsoft.com/powershell/module/az.sql/Stop-AzSqlDatabaseActivity).

```cmd
Stop-AzSqlDatabaseActivity -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName -OperationId $Operation.OperationId
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о долгосрочном хранении резервных копий отдельной базы данных и баз данных в составе пула в качестве альтернативы экспорту базы данных для архивации см. в статье [долгосрочное хранение резервных копий](long-term-retention-overview.md). Задания агентов SQL можно использовать для планирования [резервных копий только для копирования базы данных](/sql/relational-databases/backup-restore/copy-only-backups-sql-server) как альтернативу долгосрочному хранению архивных копий.
- Сведения о миграции из SQL Server в Базу данных SQL Azure с использованием BACPAC-файлов см. в [блоге группы консультирования клиентов SQL Server](/archive/blogs/sqlcat/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files).
- Сведения об импорте BACPAC-файла в базу данных SQL Server см. в статье [Импорт файла BACPAC для создания новой пользовательской базы данных](/sql/relational-databases/data-tier-applications/import-a-bacpac-file-to-create-a-new-user-database).
- Сведения об экспорте BACPAC-файла из базы данных SQL Server см. в разделе [Export a Data-tier Application](/sql/relational-databases/data-tier-applications/export-a-data-tier-application) (Экспорт приложения уровня данных).
- Дополнительные сведения об использовании службы переноса данных для переноса базы данных см. в статье [Миграция из SQL Server в базу данных SQL Azure в автономном режиме с помощью DMS](../../dms/tutorial-sql-server-to-azure-sql.md).
- Если вы экспортируете данные из SQL Server, чтобы потом перенести их в Базу данных SQL Azure, см. в статью [Migrate a SQL Server database to Azure SQL Database](migrate-to-database-from-sql-server.md) (Миграция базы данных SQL Server в базу данных SQL).
- Чтобы узнать, как безопасно управлять ключами к хранилищу данных и подписанными URL-адресами и совместно их использовать, ознакомьтесь с [руководством по безопасности службы хранилища Azure](../../storage/blobs/security-recommendations.md).
