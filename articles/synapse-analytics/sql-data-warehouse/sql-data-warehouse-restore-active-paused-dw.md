---
title: Восстановление существующего выделенного пула SQL (ранее — хранилища данных SQL)
description: Руководство по восстановлению существующего выделенного пула SQL в Azure синапсе Analytics.
services: synapse-analytics
author: anumjs
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/13/2020
ms.author: joanpo
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 2ce552a13592c9d26ef70575f98b0b76ecc454ff
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97591998"
---
# <a name="restore-an-existing-dedicated-sql-pool-formerly-sql-dw"></a>Восстановление существующего выделенного пула SQL (ранее — хранилища данных SQL)

Из этой статьи вы узнаете, как восстановить существующий выделенный пул SQL (ранее — хранилище данных SQL) с помощью портал Azure и PowerShell.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

**Проверьте ресурсы DTU.** Каждый пул размещается на [логическом сервере SQL](../../azure-sql/database/logical-servers.md) Server (например, MyServer.Database.Windows.NET), который имеет квоту DTU по умолчанию. Убедитесь, что на сервере достаточное количество оставшихся квот DTU для восстанавливаемой базы данных. Чтобы узнать, как вычислить необходимое количество DTU или запросить дополнительные единицы DTU, ознакомьтесь с разделом [Создание запроса в службу поддержки для хранилища данных SQL](sql-data-warehouse-get-started-create-support-ticket.md).

## <a name="before-you-begin"></a>Перед началом

1. Не забудьте [установить Azure PowerShell](/powershell/azure/?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).
2. Наличие существующей точки восстановления, из которой необходимо выполнить восстановление. Если вы хотите создать новое восстановление, см. [руководство по созданию новой точки восстановления, определенной пользователем](sql-data-warehouse-restore-points.md).

## <a name="restore-an-existing-dedicated-sql-pool-formerly-sql-dw-through-powershell"></a>Восстановление существующего выделенного пула SQL (ранее — хранилища данных SQL) с помощью PowerShell

Чтобы восстановить существующий выделенный пул SQL (ранее — хранилище данных SQL) из точки восстановления, используйте командлет PowerShell [RESTORE-азсклдатабасе](/powershell/module/az.sql/restore-azsqldatabase?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) .

1. Откройте средство PowerShell.

2. Подключитесь к своей учетной записи Azure и выведите список всех подписок, связанных с ней.

3. Выберите подписку, содержащую базу данных, которую нужно восстановить.

4. Перечислите точки восстановления для выделенного пула SQL (ранее — хранилища данных SQL).

5. Выберите нужные точки восстановления с помощью свойства RestorePointCreationDate.

6. Восстановите выделенный пул SQL (ранее — хранилище данных SQL) в нужную точку восстановления с помощью командлета PowerShell [RESTORE-азсклдатабасе](/powershell/module/az.sql/restore-azsqldatabase?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) .

    1. Чтобы восстановить выделенный пул SQL (ранее — хранилище данных SQL) на другой сервер, убедитесь, что указано другое имя сервера.  Этот сервер также может находиться в другой группе ресурсов и регионе.
    2. Чтобы выполнить восстановление в другую подписку, используйте кнопку "Переместить", чтобы переместить сервер в другую подписку.

7. Убедитесь, что восстановленный выделенный пул SQL (ранее — хранилище данных SQL) находится в сети.

8. После завершения восстановления можно настроить восстановленный выделенный пул SQL (прежнее название — хранилище данных SQL), выполнив инструкцию [Configure a Database после восстановления](../../azure-sql/database/disaster-recovery-guidance.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#configure-your-database-after-recovery).

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

# Or list all restore points
Get-AzSqlDatabaseRestorePoint -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName

# Get the specific database to restore
$Database = Get-AzSqlDatabase -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName

# Pick desired restore point using RestorePointCreationDate "xx/xx/xxxx xx:xx:xx xx"
$PointInTime="<RestorePointCreationDate>"  

# Restore database from a restore point
$RestoredDatabase = Restore-AzSqlDatabase –FromPointInTimeBackup –PointInTime $PointInTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName $NewDatabaseName –ResourceId $Database.ResourceID

# Use the following command to restore to a different server
#$TargetResourceGroupName = $Database.ResourceGroupName # for restoring to different server in same resourcegroup 
#$RestoredDatabase = Restore-AzSqlDatabase –FromPointInTimeBackup –PointInTime $PointInTime -ResourceGroupName $TargetResourceGroupName -ServerName $TargetServerName -TargetDatabaseName $NewDatabaseName –ResourceId $Database.ResourceID

# Verify the status of restored database
$RestoredDatabase.status

```

## <a name="restore-an-existing-dedicated-sql-pool-formerly-sql-dw-through-the-azure-portal"></a>Восстановление существующего выделенного пула SQL (ранее — хранилища данных SQL) с помощью портал Azure

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Перейдите к выделенной области, из которой нужно выполнить восстановление.
3. В верхней области колонки обзора выберите **Восстановить**.

    ![ Обзор восстановления](./media/sql-data-warehouse-restore-active-paused-dw/restoring-01.png)

4. Выберите **Точки автоматического восстановления** или **Определенные пользователем точки восстановления**. Если выделенный пул SQL (ранее — хранилище данных SQL) не имеет точек автоматического восстановления, подождите несколько часов или создайте точку восстановления, определенную пользователем, перед восстановлением. Для User-Defined точек восстановления выберите существующую или создайте новую. Для **сервера** можно выбрать сервер в другой группе ресурсов и регионе или создать новый. После ввода всех параметров щелкните **Проверка и восстановление**.

    ![Точки автоматического восстановления](./media/sql-data-warehouse-restore-active-paused-dw/restoring-11.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Восстановление удаленного выделенного пула SQL (ранее — хранилища данных SQL)](sql-data-warehouse-restore-deleted-dw.md)
- [Восстановление из выделенного пула SQL для георезервного копирования (ранее — хранилище данных SQL)](sql-data-warehouse-restore-from-geo-backup.md)
