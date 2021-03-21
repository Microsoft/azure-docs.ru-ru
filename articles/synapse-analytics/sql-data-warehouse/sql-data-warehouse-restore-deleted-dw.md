---
title: Восстановление удаленного выделенного пула SQL (ранее — хранилища данных SQL)
description: Инструкции по восстановлению удаленного выделенного пула SQL в Azure синапсе Analytics.
services: synapse-analytics
author: anumjs
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 08/29/2018
ms.author: joanpo
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 7264791654bf1b646338f0d429930b63f0cc3a06
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96449927"
---
# <a name="restore-a-deleted-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Восстановление удаленного выделенного пула SQL (ранее — хранилища данных SQL) в Azure синапсе Analytics

Из этой статьи вы узнаете, как восстановить выделенный пул SQL (ранее — хранилище данных SQL) с помощью портал Azure или PowerShell.

## <a name="before-you-begin"></a>Перед началом

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

**Проверьте ресурсы DTU.** Каждый выделенный пул SQL (ранее — хранилище данных SQL) размещается на [логическом сервере SQL Server](../../azure-sql/database/logical-servers.md) (например, MyServer.Database.Windows.NET), который имеет квоту DTU по умолчанию.  Убедитесь, что на сервере достаточно оставшихся квот DTU для восстанавливаемой базы данных. Чтобы узнать, как вычислить необходимое количество DTU или запросить дополнительные единицы DTU, ознакомьтесь с разделом [Создание запроса в службу поддержки для хранилища данных SQL](sql-data-warehouse-get-started-create-support-ticket.md).

## <a name="restore-a-deleted-data-warehouse-through-powershell"></a>Восстановление удаленного хранилища данных с помощью PowerShell

Чтобы восстановить удаленный выделенный пул SQL (ранее — хранилище данных SQL), используйте командлет [RESTORE-азсклдатабасе](/powershell/module/az.sql/restore-azsqldatabase?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) . Если соответствующий сервер также был удален, вы не сможете восстановить это хранилище данных.

1. Перед началом убедитесь, что [установлен Azure PowerShell](/powershell/azure/?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).
2. Откройте средство PowerShell.
3. Подключитесь к своей учетной записи Azure и выведите список всех подписок, связанных с ней.
4. Выберите подписку, содержащую удаленный выделенный пул SQL (ранее — хранилище данных SQL), который нужно восстановить.
5. Получение конкретного удаленного хранилища данных.
6. Восстановление удаленного выделенного пула SQL (прежнее название — хранилище данных SQL)
    1. Чтобы восстановить удаленный выделенный пул SQL (ранее — хранилище данных SQL) на другой сервер, убедитесь, что указано другое имя сервера.  Этот сервер также может находиться в другой группе ресурсов и регионе.
    1. Чтобы выполнить восстановление в другую подписку, используйте кнопку [переместить](../../azure-resource-manager/management/move-resource-group-and-subscription.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#use-the-portal) , чтобы переместить сервер в другую подписку.
7. Убедитесь, что восстановленное хранилище данных подключено.
8. После завершения восстановления можно настроить восстановленное хранилище данных, следуя [настройке базы данных после восстановления](../../azure-sql/database/disaster-recovery-guidance.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#configure-your-database-after-recovery).

```Powershell
$SubscriptionName="<YourSubscriptionName>"
$ResourceGroupName="<YourResourceGroupName>"
$ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.windows.net
#$TargetResourceGroupName="<YourTargetResourceGroupName>" # uncomment to restore to a different server.
#$TargetServerName="<YourtargetServerNameWithoutURLSuffixSeeNote>"
$DatabaseName="<YourDatabaseName>"
$NewDatabaseName="<YourDatabaseName>"

Connect-AzAccount
Get-AzSubscription
Select-AzSubscription -SubscriptionName $SubscriptionName

# Get the deleted database to restore
$DeletedDatabase = Get-AzSqlDeletedDatabaseBackup -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName

# Restore deleted database
$RestoredDatabase = Restore-AzSqlDatabase –FromDeletedDatabaseBackup –DeletionDate $DeletedDatabase.DeletionDate -ResourceGroupName $DeletedDatabase.ResourceGroupName -ServerName $DeletedDatabase.ServerName -TargetDatabaseName $NewDatabaseName –ResourceId $DeletedDatabase.ResourceID

# Use the following command to restore deleted data warehouse to a different server
#$RestoredDatabase = Restore-AzSqlDatabase –FromDeletedDatabaseBackup –DeletionDate $DeletedDatabase.DeletionDate -ResourceGroupName $TargetResourceGroupName -ServerName $TargetServerName -TargetDatabaseName $NewDatabaseName –ResourceId $DeletedDatabase.ResourceID

# Verify the status of restored database
$RestoredDatabase.status
```

## <a name="restore-a-deleted-database-using-the-azure-portal"></a>Восстановление удаленной базы данных на портале Azure

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Перейдите на сервер, на котором было размещено удаленное хранилище данных.
3. Выберите значок **Удаленные базы данных** в содержании.

    ![Удаленные базы данных](./media/sql-data-warehouse-restore-deleted-dw/restoring-deleted-01.png)

4. Выберите удаленную аналитику Azure синапсе Analytics, которую необходимо восстановить.

    ![Выбор элемента "Удаленные базы данных"](./media/sql-data-warehouse-restore-deleted-dw/restoring-deleted-11.png)

5. Укажите новое **имя базы данных** и нажмите кнопку **ОК** .

    ![Указание нового имени базы данных](./media/sql-data-warehouse-restore-deleted-dw/restoring-deleted-21.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Восстановление существующего выделенного пула SQL (ранее — хранилища данных SQL)](sql-data-warehouse-restore-active-paused-dw.md)
- [Восстановление из выделенного пула SQL для георезервного копирования (ранее — хранилище данных SQL)](sql-data-warehouse-restore-from-geo-backup.md)
