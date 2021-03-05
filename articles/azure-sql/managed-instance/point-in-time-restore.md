---
title: Восстановление до точки во времени (PITR)
titleSuffix: Azure SQL Managed Instance
description: Восстановление базы данных SQL Azure Управляемый экземпляр до предыдущей точки во времени.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein, mathoma
ms.date: 08/25/2019
ms.openlocfilehash: 0a56cfc147d4fb5cbdccf13363ad28bc602d8216
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102182763"
---
# <a name="restore-a-database-in-azure-sql-managed-instance-to-a-previous-point-in-time"></a>Восстановление базы данных в Azure SQL Управляемый экземпляр до предыдущей точки во времени
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Используйте восстановление до точки во времени, чтобы создать базу данных в качестве копии другой базы данных из прошлого. В этой статье описывается, как выполнить восстановление базы данных на момент времени в Azure SQL Управляемый экземпляр.

Восстановление до точки во времени удобно использовать в таких сценариях восстановления, как инциденты, вызванные ошибками, неправильно загруженные данные или удаление важных данных. Его также можно использовать для тестирования или аудита. Файлы резервных копий хранятся в течение 7–35 дней, в зависимости от параметров базы данных.

Восстановление до точки во времени позволяет восстановить базу данных:

- из существующей базы данных;
- из удаленной базы данных;
- для того же Управляемый экземпляр SQL или для другого Управляемый экземпляр SQL. 

## <a name="limitations"></a>Ограничения

Восстановление на момент времени в SQL Управляемый экземпляр имеет следующие ограничения.

- При восстановлении из одного экземпляра SQL Управляемый экземпляр в другой оба экземпляра должны находиться в одной подписке и регионе. Восстановление между регионами и между подписками в настоящее время не поддерживается.
- Восстановление всего Управляемый экземпляр SQL на момент времени невозможно. В этой статье объясняется только то, что возможно: восстановление базы данных на момент времени, размещенной на Управляемый экземпляр SQL.

> [!WARNING]
> Учитывайте размер хранилища Управляемый экземпляр SQL. В зависимости от размера восстанавливаемых данных в хранилище экземпляра может не хватить места. Если для восстанавливаемых данных недостаточно места, используйте другой подход.

В следующей таблице приведены сценарии восстановления на момент времени для SQL Управляемый экземпляр.

|           |Восстановление существующей базы данных в том же экземпляре SQL Управляемый экземпляр| Восстановление существующей базы данных в другой Управляемый экземпляр SQL|Восстановить удаленную базу данных в том же SQL Управляемый экземпляр|Восстановление удаленной базы данных в другой Управляемый экземпляр SQL|
|:----------|:----------|:----------|:----------|:----------|
|**Портал Azure**| Да|Нет |Да|Нет|
|**Azure CLI**|Да |Да |Нет|Нет|
|**PowerShell**| Да|Да |Да|Да|

## <a name="restore-an-existing-database"></a>Восстановление существующей базы данных

Восстановите существующую базу данных на том же Управляемый экземпляр SQL с помощью портал Azure, PowerShell или Azure CLI. Чтобы восстановить базу данных в другой Управляемый экземпляр SQL, используйте PowerShell или Azure CLI, чтобы указать свойства целевого Управляемый экземпляр SQL и группы ресурсов. Если эти параметры не заданы, база данных по умолчанию будет восстановлена в существующем Управляемый экземпляр SQL. Портал Azure в настоящее время не поддерживает восстановление в другой Управляемый экземпляр SQL.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Войдите на [портал Azure](https://portal.azure.com). 
2. Перейдите к Управляемый экземпляр SQL и выберите базу данных, которую требуется восстановить.
3. На странице базы данных выберите **Восстановить**:

    ![Восстановление базы данных на портале Azure](./media/point-in-time-restore/restore-database-to-mi.png)

4. На странице **Восстановление** выберите дату и время для восстановления базы данных.
5. Чтобы восстановить базу данных, нажмите **Подтвердить**. Это действие запускает процесс восстановления, который создает новую базу данных и заполняет ее данными из исходной базы данных на указанный момент времени. Дополнительные сведения о процессе восстановления см. в статье [Время восстановления](../database/recovery-using-backups.md#recovery-time).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Если у вас еще не установлен Azure PowerShell, см. раздел [Установка модуля Azure PowerShell](/powershell/azure/install-az-ps).

Чтобы восстановить базу данных с помощью PowerShell, укажите значения параметров в следующей команде. Затем выполните команду:

```powershell-interactive
$subscriptionId = "<Subscription ID>"
$resourceGroupName = "<Resource group name>"
$managedInstanceName = "<SQL Managed Instance name>"
$databaseName = "<Source-database>"
$pointInTime = "2018-06-27T08:51:39.3882806Z"
$targetDatabase = "<Name of new database to be created>"

Get-AzSubscription -SubscriptionId $subscriptionId
Select-AzSubscription -SubscriptionId $subscriptionId

Restore-AzSqlInstanceDatabase -FromPointInTimeBackup `
                              -ResourceGroupName $resourceGroupName `
                              -InstanceName $managedInstanceName `
                              -Name $databaseName `
                              -PointInTime $pointInTime `
                              -TargetInstanceDatabaseName $targetDatabase `
```

Чтобы восстановить базу данных в другой Управляемый экземпляр SQL, также укажите имена целевой группы ресурсов и целевого Управляемый экземпляр SQL:  

```powershell-interactive
$targetResourceGroupName = "<Resource group of target SQL Managed Instance>"
$targetInstanceName = "<Target SQL Managed Instance name>"

Restore-AzSqlInstanceDatabase -FromPointInTimeBackup `
                              -ResourceGroupName $resourceGroupName `
                              -InstanceName $managedInstanceName `
                              -Name $databaseName `
                              -PointInTime $pointInTime `
                              -TargetInstanceDatabaseName $targetDatabase `
                              -TargetResourceGroupName $targetResourceGroupName `
                              -TargetInstanceName $targetInstanceName 
```

Дополнительные сведения см. в разделе [Restore-AzSqlInstanceDatabase](/powershell/module/az.sql/restore-azsqlinstancedatabase).

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Если вы еще не установили Azure CLI, обратитесь к разделу [Установка Azure CLI](/cli/azure/install-azure-cli).

Чтобы восстановить базу данных с Azure CLI, укажите значения параметров в следующей команде. Затем выполните команду:

```azurecli-interactive
az sql midb restore -g mygroupname --mi myinstancename |
-n mymanageddbname --dest-name targetmidbname --time "2018-05-20T05:34:22"
```

Чтобы восстановить базу данных в другой Управляемый экземпляр SQL, также укажите имена целевой группы ресурсов и SQL Управляемый экземпляр:  

```azurecli-interactive
az sql midb restore -g mygroupname --mi myinstancename -n mymanageddbname |
       --dest-name targetmidbname --time "2018-05-20T05:34:22" |
       --dest-resource-group mytargetinstancegroupname |
       --dest-mi mytargetinstancename
```

Подробное описание доступных параметров см. в [документации CLI по восстановлению базы данных в управляемый экземпляр SQL](/cli/azure/sql/midb#az-sql-midb-restore).

---

## <a name="restore-a-deleted-database"></a>восстановлением удаленной базы данных;

Восстановление удаленной базы данных можно выполнить с помощью PowerShell или портала Azure. Чтобы восстановить удаленную базу данных в том же экземпляре, используйте портал Azure или PowerShell. Чтобы восстановить удаленную базу данных в другом экземпляре, используйте PowerShell. 

### <a name="portal"></a>Портал 


Чтобы восстановить управляемую базу данных с помощью портал Azure, откройте страницу Обзор Управляемый экземпляр SQL и выберите **Удаленные базы данных**. Выберите удаленную базу данных, которую требуется восстановить, и введите имя новой базы данных, которая будет создана из данных, восстановленных из резервной копии.

  ![Снимок экрана: восстановление удаленной базы данных экземпляра SQL Azure](./media/point-in-time-restore/restore-deleted-sql-managed-instance-annotated.png)

.. /.. /скл-датабасе

### <a name="powershell"></a>PowerShell

Чтобы восстановить базу данных в том же экземпляре, обновите значения параметров, а затем выполните следующую команду PowerShell: 

```powershell-interactive
$subscriptionId = "<Subscription ID>"
Get-AzSubscription -SubscriptionId $subscriptionId
Select-AzSubscription -SubscriptionId $subscriptionId

$resourceGroupName = "<Resource group name>"
$managedInstanceName = "<SQL Managed Instance name>"
$deletedDatabaseName = "<Source database name>"
$targetDatabaseName = "<target database name>"

$deletedDatabase = Get-AzSqlDeletedInstanceDatabaseBackup -ResourceGroupName $resourceGroupName `
-InstanceName $managedInstanceName -DatabaseName $deletedDatabaseName

Restore-AzSqlinstanceDatabase -FromPointInTimeBackup -Name $deletedDatabase.Name `
   -InstanceName $deletedDatabase.ManagedInstanceName `
   -ResourceGroupName $deletedDatabase.ResourceGroupName `
   -DeletionDate $deletedDatabase.DeletionDate `
   -PointInTime UTCDateTime `
   -TargetInstanceDatabaseName $targetDatabaseName
```

Чтобы восстановить базу данных в другой Управляемый экземпляр SQL, также укажите имена целевой группы ресурсов и целевого Управляемый экземпляр SQL:

```powershell-interactive
$targetResourceGroupName = "<Resource group of target SQL Managed Instance>"
$targetInstanceName = "<Target SQL Managed Instance name>"

Restore-AzSqlinstanceDatabase -FromPointInTimeBackup -Name $deletedDatabase.Name `
   -InstanceName $deletedDatabase.ManagedInstanceName `
   -ResourceGroupName $deletedDatabase.ResourceGroupName `
   -DeletionDate $deletedDatabase.DeletionDate `
   -PointInTime UTCDateTime `
   -TargetInstanceDatabaseName $targetDatabaseName `
   -TargetResourceGroupName $targetResourceGroupName `
   -TargetInstanceName $targetInstanceName 
```

## <a name="overwrite-an-existing-database"></a>Перезапись существующей базы данных

Чтобы перезаписать существующую базу данных, выполните следующие действия.

1. Удалите существующую базу данных, которую требуется перезаписать.
2. Переименуйте базу данных, восстановленную до точки во времени, присвоив ей имя удаленной базы данных.

### <a name="drop-the-original-database"></a>Удаление исходной базы данных

Вы можете удалить базу данных с помощью портала Azure, PowerShell или Azure CLI.

Можно также удалить базу данных, подключившись к SQL Управляемый экземпляр напрямую, запустив SQL Server Management Studio (SSMS), а затем выполнив следующую команду Transact-SQL (T-SQL):

```sql
DROP DATABASE WorldWideImporters;
```

Для подключения к базе данных в Управляемый экземпляр SQL используйте один из следующих методов:

- [SSMS/Azure Data Studio через виртуальную машину Azure](./connect-vm-instance-configure.md)
- [Точка-сеть](./point-to-site-p2s-configure.md)
- [Общедоступная конечная точка](./public-endpoint-configure.md)

# <a name="portal"></a>[Портал](#tab/azure-portal)

В портал Azure выберите базу данных из Управляемый экземпляр SQL и нажмите кнопку **Удалить**.

   ![Удаление базы данных на портале Azure](./media/point-in-time-restore/delete-database-from-mi.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Используйте следующую команду PowerShell для удаления существующей базы данных из Управляемый экземпляр SQL:

```powershell
$resourceGroupName = "<Resource group name>"
$managedInstanceName = "<SQL Managed Instance name>"
$databaseName = "<Source database>"

Remove-AzSqlInstanceDatabase -Name $databaseName -InstanceName $managedInstanceName -ResourceGroupName $resourceGroupName
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Используйте следующую команду Azure CLI, чтобы удалить существующую базу данных из Управляемый экземпляр SQL:

```azurecli-interactive
az sql midb delete -g mygroupname --mi myinstancename -n mymanageddbname
```

---

### <a name="alter-the-new-database-name-to-match-the-original-database-name"></a>Изменение имени новой базы данных на имя исходной базы данных

Подключитесь непосредственно к Управляемый экземпляр SQL и запустите SQL Server Management Studio. Затем выполните следующий запрос Transact-SQL (T-SQL). Запрос изменит имя восстановленной базы данных на имя удаленной базы данных, которую предполагается перезаписать.

```sql
ALTER DATABASE WorldWideImportersPITR MODIFY NAME = WorldWideImporters;
```

Используйте один из следующих методов для подключения к базе данных в Управляемый экземпляр SQL:

- [Виртуальная машина Azure](./connect-vm-instance-configure.md)
- [Точка-сеть](./point-to-site-p2s-configure.md)
- [Общедоступная конечная точка](./public-endpoint-configure.md)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об [автоматических резервных копиях](../database/automated-backups-overview.md).
