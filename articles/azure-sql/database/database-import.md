---
title: Импорт BACPAC-файла для создания базы данных в Базе данных SQL Azure
description: Создайте базу данных в службе "База данных SQL Azure" или "Управляемый экземпляр SQL Azure" на основе BACPAC-файла.
services: sql-database
ms.service: sql-db-mi
ms.subservice: migrate
ms.custom: sqldbrb=1, devx-track-azurepowershell
ms.devlang: ''
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 10/29/2020
ms.openlocfilehash: 2412d3d2851d1b4d251b50ff3068b7dcafcccee8
ms.sourcegitcommit: c8b50a8aa8d9596ee3d4f3905bde94c984fc8aa2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/28/2021
ms.locfileid: "105642043"
---
# <a name="quickstart-import-a-bacpac-file-to-a-database-in-azure-sql-database-or-azure-sql-managed-instance"></a>Краткое руководство. Импорт BACPAC-файла в базу данных в службе "База данных SQL Azure" или "Управляемый экземпляр SQL Azure"
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

С помощью [BACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac)-файла вы можете импортировать базу данных SQL Server в Базу данных SQL Azure или Управляемый экземпляр SQL. Вы можете импортировать данные из BACPAC-файла, который хранится в хранилище BLOB-объектов Azure (только цен. категория "Стандартный") или локальном хранилище, в локальное расположение. Чтобы развернуть скорость импорта, предоставив дополнительные ресурсы и ускоренную скорость доступа к ним, масштабируйте базу данных до более высокого уровня службы и объема вычислительных ресурсов во время процесса импорта. После завершения импорта эти значения можно уменьшить.

> [!NOTE]
> Уровень совместимости импортированной базы данных основан на уровне совместимости базы данных-источника.

> [!IMPORTANT]
> После импорта базы данных вы можете работать с ней при текущем уровне совместимости (уровень 100 для базы данных AdventureWorks2008R2) или при более высоком уровне. Дополнительные сведения о последствиях и параметрах работы базы данных с определенным уровнем совместимости см. в разделе [ALTER DATABASE (Transact-SQL) Compatibility Level](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level) (Уровень совместимости ALTER DATABASE (Transact-SQL)). Кроме того, в статье [ALTER DATABASE SCOPED CONFIGURATION (Transact-SQL)](/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql) вы можете получить сведения о дополнительных настройках уровня базы данных, связанных с уровнем совместимости.

## <a name="using-azure-portal"></a>Использование портала Azure

Посмотрите это видео, чтобы узнать, как импортировать данные из BACPAC-файла с помощью портала Azure, или продолжайте читать эту статью.

> [!VIDEO https://channel9.msdn.com/Shows/Data-Exposed/Its-just-SQL-Restoring-a-database-to-Azure-SQL-DB-from-backup/player?WT.mc_id=dataexposed-c9-niner]

[Портал Azure поддерживает создание](https://portal.azure.com) *только* отдельной базы данных в Базе данных SQL Azure и *только* из файла BACPAC, который хранится в хранилище BLOB-объектов Azure.

Чтобы перенести базу данных в [Управляемый экземпляр SQL Azure](../managed-instance/sql-managed-instance-paas-overview.md) из BACPAC-файла, используйте SQL Server Management Studio или SQLPackage, так как эта операция пока не поддерживается на портале Azure и в Azure PowerShell.

> [!NOTE]
> Компьютеры, обрабатывающие запросы на импорт и экспорт, отправленные через портал Azure или PowerShell, должны хранить BACPAC-файл и временные файлы, созданные платформой приложения уровня данных (DacFX). Требуемое дисковое пространство может быть существенно разным для баз данных одного размера и в три раза превышать размер самой базы данных. Компьютеры, на которых выполняется запрос на импорт или экспорт, располагают только 450 ГБ пространства на локальном диске. Это означает, что некоторые запросы могут завершиться ошибкой `There is not enough space on the disk`. В качестве обходного решения можно запустить sqlpackage.exe на другом компьютере, где есть достаточно места на локальном диске. Мы советуем использовать SqlPackage для импорта и экспорта баз данных, превышающих 150 ГБ, чтобы избежать этой проблемы.

1. Для импорта из BACPAC-файла в новую отдельную базу данных с помощью портала Azure откройте страницу нужного сервера и затем выберите **Импорт базы данных** на панели инструментов.  

   ![База данных import1](./media/database-import/sql-server-import-database.png)

1. Сначала выберите учетную запись хранения и контейнер для файла BACPAC, а затем — сам файл BACPAC из которого необходимо выполнить импорт.

1. Укажите размер новой базы данных (обычно он такой же, как и у источника) и учетные данные для целевого сервера SQL Server. Список значений, доступных для новой базы данных в Базе данных SQL Azure, см. в статье [CREATE DATABASE](/sql/t-sql/statements/create-database-transact-sql?view=azuresqldb-current&preserve-view=true).

   ![База данных import2](./media/database-import/sql-server-import-database-settings.png)

1. Нажмите кнопку **ОК**.

1. Чтобы отслеживать ход выполнения импорта, откройте страницу сервера базы данных. В разделе **Параметры** выберите **Журнал импорта и экспорта**. При успешном выполнении импорта отображается состояние **Завершено**.

   ![Состояние импорта базы данных](./media/database-import/sql-server-import-database-history.png)

1. Выберите **Базы данных SQL**, чтобы проверить активность базы данных на сервере, и убедитесь, что для новой базы данных задано состояние **В сети**.

## <a name="using-sqlpackage"></a>Использование SqlPackage

Импорт базы данных SQL Server с помощью программы командной строки [SqlPackage](/sql/tools/sqlpackage) описывается в разделе [Параметры и свойства импорта](/sql/tools/sqlpackage#import-parameters-and-properties). SqlPackage входит в состав [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) и [SQL Server Data Tools для Visual Studio](/sql/ssdt/download-sql-server-data-tools-ssdt). Кроме того, новую версию [SqlPackage](https://www.microsoft.com/download/details.aspx?id=53876) можно скачать в Центре загрузки Майкрософт. 

Вместо портала Azure рекомендуется использовать SqlPackage для масштабирования и обеспечения производительности в большинстве рабочих сред. Сведения о миграции из SQL Server в Базу данных SQL Azure с помощью файлов `BACPAC` см. в блоге команды Помощника по Azure для пользователей SQL Server [здесь](/archive/blogs/sqlcat/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files).

Модель подготовки на основе DTU поддерживает выбор значений максимального размера базы данных для каждого уровня. При импорте базы данных [используйте одно из этих поддерживаемых значений](/sql/t-sql/statements/create-database-transact-sql). 

Приведенная ниже команда SqlPackage импортирует базу данных **AdventureWorks2008R2** из локального хранилища на логический сервер SQL с именем **mynewserver20170403**. Эта команда создает базу данных **myMigratedDatabase** с уровнем служб **Премиум** и целевым уровнем служб **P6**. Подставьте соответствующие значения для своей среды.

```cmd
sqlpackage.exe /a:import /tcs:"Data Source=<serverName>.database.windows.net;Initial Catalog=<migratedDatabase>;User Id=<userId>;Password=<password>" /sf:AdventureWorks2008R2.bacpac /p:DatabaseEdition=Premium /p:DatabaseServiceObjective=P6
```

> [!IMPORTANT]
> Чтобы подключиться к серверу Базы данных SQL Azure через корпоративный брандмауэр, следует открыть порт 1433 в этом брандмауэре. Чтобы подключиться к Управляемому экземпляру SQL, требуется [подключение типа "точка — сеть"](../managed-instance/point-to-site-p2s-configure.md) или подключение Express Route.

В этом примере показано, как импортировать базу данных с помощью SqlPackage, используя универсальную проверку подлинности Active Directory.

```cmd
sqlpackage.exe /a:Import /sf:testExport.bacpac /tdn:NewDacFX /tsn:apptestserver.database.windows.net /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="using-powershell"></a>Использование PowerShell

> [!NOTE]
> Сейчас [Управляемый экземпляр SQL](../managed-instance/sql-managed-instance-paas-overview.md) не поддерживает миграцию базы данных в базу данных экземпляра из BACPAC-файла с помощью Azure PowerShell. Чтобы выполнить импорт в Управляемый экземпляр SQL, используйте SQL Server Management Studio или SQLPackage.

> [!NOTE]
> Компьютеры, обрабатывающие запросы на импорт и экспорт, отправленные через портал или PowerShell, должны хранить не только BACPAC-файл, но и временные файлы, созданные платформой приложения уровня данных (DacFX). Требуемое дисковое пространство может быть существенно разным для баз данных одного размера и в три раза превышать размер самой базы данных. Компьютеры, на которых выполняется запрос на импорт или экспорт, располагают только 450 ГБ пространства на локальном диске. В результате некоторые запросы могут завершиться ошибкой "Недостаточно места на диске". В качестве обходного решения можно запустить sqlpackage.exe на другом компьютере, где есть достаточно места на локальном диске. При импорте или экспорте баз данных, размер которых превышает 150 ГБ, используйте SqlPackage, чтобы избежать этой проблемы.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

> [!IMPORTANT]
> Модуль PowerShell Azure Resource Manager по-прежнему поддерживается, но вся будущая разработка сосредоточена на модуле Az.Sql. Исправления ошибок для модуля AzureRM будут продолжать выпускаться как минимум до декабря 2020 г.  Аргументы команд в модулях Az и AzureRm практически идентичны. Дополнительные сведения о совместимости см. в статье [Знакомство с новым модулем Az для Azure PowerShell](/powershell/azure/new-azureps-module-az).

Используйте командлет [New-AzSqlDatabaseImport](/powershell/module/az.sql/new-azsqldatabaseimport), чтобы отправить запрос на импорт базы данных в Azure. Операция импорта может занять некоторое время в зависимости от размера базы данных. Модель подготовки на основе DTU поддерживает выбор значений максимального размера базы данных для каждого уровня. При импорте базы данных [используйте одно из этих поддерживаемых значений](/sql/t-sql/statements/create-database-transact-sql). 

```powershell
$importRequest = New-AzSqlDatabaseImport -ResourceGroupName "<resourceGroupName>" `
    -ServerName "<serverName>" -DatabaseName "<databaseName>" `
    -DatabaseMaxSizeBytes "<databaseSizeInBytes>" -StorageKeyType "StorageAccessKey" `
    -StorageKey $(Get-AzStorageAccountKey `
        -ResourceGroupName "<resourceGroupName>" -StorageAccountName "<storageAccountName>").Value[0] `
        -StorageUri "https://myStorageAccount.blob.core.windows.net/importsample/sample.bacpac" `
        -Edition "Standard" -ServiceObjectiveName "P6" `
        -AdministratorLogin "<userId>" `
        -AdministratorLoginPassword $(ConvertTo-SecureString -String "<password>" -AsPlainText -Force)
```

Чтобы проверить ход выполнения импорта, используйте командлет [Get-AzSqlDatabaseImportExportStatus](/powershell/module/az.sql/get-azsqldatabaseimportexportstatus). Если выполнить его немедленно после запроса, то обычно возвращается сообщение `Status: InProgress`. Импорт считается завершенным, когда вы увидите состояние `Status: Succeeded`.

```powershell
$importStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink

[Console]::Write("Importing")
while ($importStatus.Status -eq "InProgress") {
    $importStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink
    [Console]::Write(".")
    Start-Sleep -s 10
}

[Console]::WriteLine("")
$importStatus
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы отправить запрос на импорт базы данных в Azure, используйте команду [az-sql-db-import](/cli/azure/sql/db#az-sql-db-import). Операция импорта может занять некоторое время в зависимости от размера базы данных. Модель подготовки на основе DTU поддерживает выбор значений максимального размера базы данных для каждого уровня. При импорте базы данных [используйте одно из этих поддерживаемых значений](/sql/t-sql/statements/create-database-transact-sql). 

```azurecli
# get the storage account key
az storage account keys list --resource-group "<resourceGroup>" --account-name "<storageAccount>"

az sql db import --resource-group "<resourceGroup>" --server "<server>" --name "<database>" `
    --storage-key-type "StorageAccessKey" --storage-key "<storageAccountKey>" `
    --storage-uri "https://myStorageAccount.blob.core.windows.net/importsample/sample.bacpac" `
    -u "<userId>" -p "<password>"
```

* * *

> [!TIP]
> Еще один пример сценария приведен в разделе [Импорт базы данных из BACPAC-файла](scripts/import-from-bacpac-powershell.md).

## <a name="cancel-the-import-request"></a>Отмена запроса на импорт

Используйте API из раздела [Операции с базой данных — отмена](/rest/api/sql/databaseoperations/cancel) или команду [Stop-AzSqlDatabaseActivity](/powershell/module/az.sql/Stop-AzSqlDatabaseActivity) в PowerShell. Ниже приведен пример команды PowerShell.

```cmd
Stop-AzSqlDatabaseActivity -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName -OperationId $Operation.OperationId
```


## <a name="limitations"></a>Ограничения

- Импорт базы данных в эластичный пул не поддерживается. Однако вы можете сначала выполнить импорт данных в отдельную базу данных, а затем переместить эту базу данных в эластичный пул.
- Служба импорта и экспорта не работает, если параметр "Разрешить доступ к службам Azure" отключен. Но эту проблему можно обойти, вручную запустив sqlpackage.exe с виртуальной машины Azure или выполнив экспорт непосредственно в коде с помощью API DACFx.
- При импорте нельзя указать уровень избыточности для хранилища резервных копий, если вы создаете новую базу данных. Она всегда создается с избыточностью по умолчанию как геоизбыточное хранилище резервных копий. Если это вам не подходит, сначала создайте на портале Azure или с помощью PowerShell пустую базу данных с нужной избыточностью хранилища резервных копий, а затем импортируйте BACPAC-файл в эту пустую базу данных. 

> [!NOTE]
> Настраиваемая избыточность хранилища резервных копий для Базы данных SQL Azure сейчас предоставляется только в регионе Azure Юго-Восточной Азии в режиме общедоступной предварительной версии.

## <a name="import-using-wizards"></a>Импорт с помощью мастера

Импорт также можно выполнить с помощью следующих мастеров:

- [мастер импорта приложения уровня данных в SQL Server Management Studio](/sql/relational-databases/data-tier-applications/import-a-bacpac-file-to-create-a-new-user-database#using-the-import-data-tier-application-wizard);
- [мастер импорта и экспорта SQL Server](/sql/integration-services/import-export-data/start-the-sql-server-import-and-export-wizard).

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы научиться подключаться к базе данных в Базе данных SQL и отправлять к ней запросы, ознакомьтесь со статьей [Краткое руководство. Подключение к базе данных SQL Azure и Подключение к базе данных SQL Azure и создание запросов к ней с помощью SQL Server Management Studio](connect-query-ssms.md).
- Сведения о миграции из SQL Server в Базу данных SQL Azure с использованием BACPAC-файлов см. в [блоге группы консультирования клиентов SQL Server](https://techcommunity.microsoft.com/t5/DataCAT/Migrating-from-SQL-Server-to-Azure-SQL-Database-using-Bacpac/ba-p/305407).
- Описание процесса миграции базы данных SQL Server в базу данных SQL Azure, включая рекомендации по его использованию, см. в [этой статье](migrate-to-database-from-sql-server.md).
- Чтобы узнать, как безопасно управлять ключами к хранилищу данных и подписанными URL-адресами и совместно их использовать, ознакомьтесь с [руководством по безопасности службы хранилища Azure](../../storage/blobs/security-recommendations.md).