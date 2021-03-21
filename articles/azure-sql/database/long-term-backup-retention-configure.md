---
title: База данных SQL Azure. Управление долгосрочным хранением резервных копий
description: Узнайте, как хранить и восстанавливать автоматически создаваемые резервные копии базы данных SQL Azure в службе хранилища Azure (до 10 лет) с помощью портал Azure и PowerShell.
services: sql-database
ms.service: sql-db-mi
ms.subservice: backup-restore
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, sstein
ms.date: 12/16/2020
ms.openlocfilehash: fad19d360f7c476ba71a9bbe00b58387b92f8ac4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101690568"
---
# <a name="manage-azure-sql-database-long-term-backup-retention"></a>Управление долгосрочным хранением резервных копий базы данных SQL Azure
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

С помощью базы данных SQL Azure можно настроить политику [долгосрочного хранения резервных](long-term-retention-overview.md) копий (LTR) для автоматического хранения резервных копий в отдельных контейнерах хранилища BLOB-объектов Azure до 10 лет. После этого появляется возможность восстановления базы данных с помощью резервных копий на портале Azure или через PowerShell. Политики долгосрочного хранения также поддерживаются для [управляемый экземпляр Azure SQL](../managed-instance/long-term-backup-retention-configure.md).

## <a name="using-the-azure-portal"></a>Использование портала Azure

В следующих разделах показано, как использовать портал Azure для задания политик долгосрочного хранения, управления доступными резервными копиями долгосрочного хранения и восстановления из доступной резервной копии.

### <a name="configure-long-term-retention-policies"></a>Настройка политик долгосрочного хранения

Вы можете настроить базу данных SQL для [сохранения автоматически создаваемых резервных копий](long-term-retention-overview.md) на срок больший, чем срок хранения для вашего уровня служб.

1. В портал Azure перейдите к своему серверу, а затем выберите **резервные копии**. Перейдите на вкладку **политики хранения** , чтобы изменить параметры хранения резервных копий.

   ![Работа с политиками хранения](./media/long-term-backup-retention-configure/ltr-policies-tab.png)

2. На вкладке политики хранения выберите базы данных, на которых необходимо задать или изменить политики долгосрочного хранения резервных копий. Невыбранные базы данных не будут затронуты.

   ![Выбор базы данных для настройки политик хранения резервных копий](./media/long-term-backup-retention-configure/ltr-policies-tab-configure.png)

3. На панели **Настройка политик** укажите необходимый срок хранения для еженедельных, ежемесячных или ежегодных резервных копий. Выберите срок хранения "0", чтобы указать, что не следует задавать долгосрочное хранение резервных копий.

   ![панель «Настройка политик»](./media/long-term-backup-retention-configure/ltr-configure-policies.png)

4. Нажмите кнопку **Применить** , чтобы применить выбранные параметры хранения ко всем выбранным базам данных.

> [!IMPORTANT]
> При включении политики долгосрочного хранения резервных копий может потребоваться до 7 дней, чтобы первая резервная копия стала видимой и доступной для восстановления. Дополнительные сведения о резервном копировании LTR каданце см. в статье [долгосрочное хранение резервных копий](long-term-retention-overview.md).

### <a name="view-backups-and-restore-from-a-backup"></a>Просмотр резервных копий и восстановление из резервной копии

Просмотр резервных копий, которые хранятся для определенной базы данных с политикой LTR, и восстановление из этих резервных копий.

1. В портал Azure перейдите к своему серверу, а затем выберите **резервные копии**. Чтобы просмотреть доступные резервные копии на LTR для конкретной базы данных, выберите **Управление** в столбце доступные резервные копии по правому LTR. Появится панель со списком доступных резервных копий LTR для выбранной базы данных.

   ![доступные возможности резервного копирования](./media/long-term-backup-retention-configure/ltr-available-backups-tab.png)

1. На появившейся панели « **доступные резервные копии** » ознакомьтесь с доступными резервными копиями. Вы можете выбрать резервную копию для восстановления из или для удаления.

   ![Просмотреть доступные резервные копии на LTR](./media/long-term-backup-retention-configure/ltr-available-backups-manage.png)

1. Чтобы выполнить восстановление из доступной резервной копии LTR, выберите резервную копию, которую необходимо восстановить, и нажмите кнопку **восстановить**.

   ![Восстановление из доступной резервной копии слева направо](./media/long-term-backup-retention-configure/ltr-available-backups-restore.png)

1. Выберите имя для новой базы данных, а затем щелкните **проверить и создать** , чтобы проверить сведения о восстановлении. Выберите **создать** , чтобы восстановить базу данных из выбранной резервной копии.

   ![Настройка сведений о восстановлении](./media/long-term-backup-retention-configure/restore-ltr.png)

1. На панели инструментов щелкните значок уведомления, чтобы просмотреть состояние задания восстановления.

   ![Ход выполнения задания восстановления](./media/long-term-backup-retention-configure/restore-job-progress-long-term.png)

1. После завершения задания откройте страницу **Базы данных SQL**, чтобы просмотреть восстановленную базу данных.

> [!NOTE]
> Здесь вы можете подключиться к восстановленной базе данных с помощью SQL Server Management Studio и выполнить необходимые задания. Например, вы можете [извлечь часть данных из восстановленной базы данных, чтобы скопировать их в существующую базу данных, или удалить существующую базу данных и присвоить ее имя восстановленной базе данных](recovery-using-backups.md#point-in-time-restore).

## <a name="using-powershell"></a>Использование PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> Модуль PowerShell Azure Resource Manager по-прежнему поддерживается базой данных SQL Azure, но вся будущая разработка сосредоточена на модуле Az.Sql. Сведения об этих командлетах см. в разделе [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Аргументы команд в модулях Az и AzureRm практически идентичны.

В следующих разделах показано, как использовать PowerShell для настройки долгосрочного хранения резервных копий, просмотра резервных копий в службе хранилища Azure и восстановления из резервной копии в службе хранилища Azure.

### <a name="azure-roles-to-manage-long-term-retention"></a>Роли Azure для управления долгосрочным хранением

Для **Get-азсклдатабаселонгтермретентионбаккуп** и **RESTORE-азсклдатабасе** необходимо иметь одну из следующих ролей:

- Роль владельца подписки или
- Роль участника SQL Server или
- Пользовательская роль со следующими разрешениями:

   Microsoft. SQL/Locations/Лонгтермретентионбаккупс/Read Microsoft. SQL/Locations/Лонгтермретентионсерверс/Лонгтермретентионбаккупс/Read Microsoft. SQL/Locations/Лонгтермретентионсерверс/longTermRetentionDatabases/longTermRetentionBackups/Read

Для **Remove-азсклдатабаселонгтермретентионбаккуп** необходимо иметь одну из следующих ролей:

- Роль владельца подписки или
- Пользовательская роль со следующим разрешением:

   Microsoft.Sql/locations/longTermRetentionServers/longTermRetentionDatabases/longTermRetentionBackups/delete

> [!NOTE]
> Роль участника SQL Server не имеет разрешения на удаление резервных копий слева направо.

Разрешения RBAC для Azure можно предоставить в *подписке* или области *группы ресурсов* . Однако для доступа к резервным копиям LTR, принадлежащим удаленному серверу, разрешение должно быть предоставлено в области действия *подписки* этого сервера.

- Microsoft.Sql/locations/longTermRetentionServers/longTermRetentionDatabases/longTermRetentionBackups/delete

### <a name="create-an-ltr-policy"></a>Создание политики LTR

```powershell
# get the SQL server
$subId = "<subscriptionId>"
$serverName = "<serverName>"
$resourceGroup = "<resourceGroupName>"
$dbName = "<databaseName>"

Connect-AzAccount
Select-AzSubscription -SubscriptionId $subId

$server = Get-AzSqlServer -ServerName $serverName -ResourceGroupName $resourceGroup

# create LTR policy with WeeklyRetention = 12 weeks. MonthlyRetention and YearlyRetention = 0 by default.
Set-AzSqlDatabaseBackupLongTermRetentionPolicy -ServerName $serverName -DatabaseName $dbName `
    -ResourceGroupName $resourceGroup -WeeklyRetention P12W

# create LTR policy with WeeklyRetention = 12 weeks, YearlyRetention = 5 years and WeekOfYear = 16 (week of April 15). MonthlyRetention = 0 by default.
Set-AzSqlDatabaseBackupLongTermRetentionPolicy -ServerName $serverName -DatabaseName $dbName `
    -ResourceGroupName $resourceGroup -WeeklyRetention P12W -YearlyRetention P5Y -WeekOfYear 16
```

### <a name="view-ltr-policies"></a>Просмотр политик LTR

В этом примере показан процесс выведения списка политик LTR на сервере.

```powershell
# get all LTR policies within a server
$ltrPolicies = Get-AzSqlDatabase -ResourceGroupName $resourceGroup -ServerName $serverName | `
    Get-AzSqlDatabaseLongTermRetentionPolicy

# get the LTR policy of a specific database
$ltrPolicies = Get-AzSqlDatabaseBackupLongTermRetentionPolicy -ServerName $serverName -DatabaseName $dbName `
    -ResourceGroupName $resourceGroup
```

### <a name="clear-an-ltr-policy"></a>Удаление политики LTR

В этом примере показано, как удалить политику LTR из базы данных.

```powershell
Set-AzSqlDatabaseBackupLongTermRetentionPolicy -ServerName $serverName -DatabaseName $dbName `
    -ResourceGroupName $resourceGroup -RemovePolicy
```

### <a name="view-ltr-backups"></a>Просмотр резервных копий LTR

В этом примере показано, как вывести список резервных копий LTR на сервере.

```powershell
# get the list of all LTR backups in a specific Azure region
# backups are grouped by the logical database id, within each group they are ordered by the timestamp, the earliest backup first
$ltrBackups = Get-AzSqlDatabaseLongTermRetentionBackup -Location $server.Location

# get the list of LTR backups from the Azure region under the named server
$ltrBackups = Get-AzSqlDatabaseLongTermRetentionBackup -Location $server.Location -ServerName $serverName

# get the LTR backups for a specific database from the Azure region under the named server
$ltrBackups = Get-AzSqlDatabaseLongTermRetentionBackup -Location $server.Location -ServerName $serverName -DatabaseName $dbName

# list LTR backups only from live databases (you have option to choose All/Live/Deleted)
$ltrBackups = Get-AzSqlDatabaseLongTermRetentionBackup -Location $server.Location -DatabaseState Live

# only list the latest LTR backup for each database
$ltrBackups = Get-AzSqlDatabaseLongTermRetentionBackup -Location $server.Location -ServerName $serverName -OnlyLatestPerDatabase
```

### <a name="delete-ltr-backups"></a>Удаление резервных копий LTR

В этом примере показано, как удалить резервную копию LTR из списка резервных копий.

```powershell
# remove the earliest backup
$ltrBackup = $ltrBackups[0]
Remove-AzSqlDatabaseLongTermRetentionBackup -ResourceId $ltrBackup.ResourceId
```

> [!IMPORTANT]
> Удаление резервной копии LTR отменить нельзя. Чтобы удалить резервную копию LTR после удаления сервера, необходимо иметь разрешение на область подписки. Вы можете настроить уведомления о каждом удалении в Azure Monitor путем фильтрации для операции "Удаление резервной копии долгосрочного хранения". В журнале действий содержатся сведения о том, кто и когда выполнил запрос. Подробные сведения см. в статье [Создание, просмотр и управление оповещениями журнала действий с помощью Azure Monitor](../../azure-monitor/alerts/alerts-activity-log.md).

### <a name="restore-from-ltr-backups"></a>Восстановление из резервных копий LTR

В этом примере показано, как выполнить восстановление из резервной копии LTR. Обратите внимание, что этот интерфейс не изменился, но для параметра идентификатора ресурса теперь требуется идентификатор ресурса резервного копирования LTR.

```powershell
# restore a specific LTR backup as an P1 database on the server $serverName of the resource group $resourceGroup
Restore-AzSqlDatabase -FromLongTermRetentionBackup -ResourceId $ltrBackup.ResourceId -ServerName $serverName -ResourceGroupName $resourceGroup `
    -TargetDatabaseName $dbName -ServiceObjectiveName P1
```

> [!IMPORTANT]
> Чтобы выполнить восстановление из резервной копии LTR после удаления сервера или группы ресурсов, необходимо иметь разрешения, ограниченные подпиской сервера, и эта подписка должна быть активной. Необходимо также опустить необязательный параметр-ResourceGroupName.

> [!NOTE]
> Здесь вы можете подключиться к восстановленной базе данных с помощью SQL Server Management Studio и выполнить необходимые задания, например извлечь часть данных из восстановленной базы данных, чтобы скопировать их в имеющуюся базу данных или удалить имеющуюся базу данных и присвоить ее имя восстановленной базе данных. Ознакомьтесь с [восстановлением до точки во времени](recovery-using-backups.md#point-in-time-restore).

## <a name="limitations"></a>Ограничения
- При восстановлении из резервной копии LTR свойство Read Scale отключено. Чтобы включить, прочтите масштаб восстановленной базы данных, обновите базу данных после ее создания.
- Необходимо указать целевую цель уровня обслуживания при восстановлении из резервной копии LTR, которая была создана, когда база данных находилась в эластичном пуле. 

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о резервных копиях базы данных, создаваемых автоматически службой, см. в [этой статье](automated-backups-overview.md).
- Дополнительные сведения о долгосрочном хранении резервных копий см. в статье [Хранение резервных копий базы данных SQL Azure до 10 лет](long-term-retention-overview.md).
