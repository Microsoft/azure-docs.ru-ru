---
title: Определенные пользователем точки восстановления
description: Создание точки восстановления для выделенного пула SQL (ранее — хранилища данных SQL).
services: synapse-analytics
author: anumjs
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 07/03/2019
ms.author: anjangsh
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 097a3132208eee98b3f95291e414263e637bc265
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96545593"
---
# <a name="user-defined-restore-points-for-a-dedicated-sql-pool-formerly-sql-dw"></a>Пользовательские точки восстановления для выделенного пула SQL (ранее — хранилище данных SQL)

Из этой статьи вы узнаете, как создать новую определяемую пользователем точку восстановления для выделенного пула SQL (ранее — хранилище данных SQL) в Azure синапсе Analytics с помощью PowerShell и портал Azure.

## <a name="create-user-defined-restore-points-through-powershell"></a>Создание определяемых пользователем точек восстановления с помощью PowerShell

Чтобы создать определенную пользователем точку восстановления, используйте командлет PowerShell [New-азсклдатабасересторепоинт](/powershell/module/az.sql/new-azsqldatabaserestorepoint?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) .

1. Перед началом убедитесь, что [установлен Azure PowerShell](/powershell/azure/?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).
2. Откройте средство PowerShell.
3. Подключитесь к своей учетной записи Azure и выведите список всех подписок, связанных с ней.
4. Выберите подписку, содержащую базу данных, которую нужно восстановить.
5. Создайте точку восстановления для немедленной копии хранилища данных.

```Powershell

$SubscriptionName="<YourSubscriptionName>"
$ResourceGroupName="<YourResourceGroupName>"
$ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.windows.net
$DatabaseName="<YourDatabaseName>"
$Label = "<YourRestorePointLabel>"

Connect-AzAccount
Get-AzSubscription
Select-AzSubscription -SubscriptionName $SubscriptionName

# Create a restore point of the original database
New-AzSqlDatabaseRestorePoint -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName -RestorePointLabel $Label

```

6. Просмотрите список всех существующих точек восстановления.

```Powershell
# List all restore points
Get-AzSqlDatabaseRestorePoint -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName
```

## <a name="create-user-defined-restore-points-through-the-azure-portal"></a>Создание определяемых пользователем точек восстановления с помощью портал Azure

Пользовательские точки восстановления также можно создавать с помощью портал Azure.

1. Войдите в учетную запись [портал Azure](https://portal.azure.com/) .

2. Перейдите к выделенному пулу SQL (ранее SQL DW), для которого нужно создать точку восстановления.

3. На панели слева выберите **Обзор** и щелкните **+ создать точку восстановления**. Если кнопка Создать точку восстановления не включена, убедитесь, что выделенный пул SQL (ранее — хранилище данных SQL) не приостановлен.

    ![Новая точка восстановления](./media/sql-data-warehouse-restore-points/creating-restore-point-01.png)

4. Укажите имя для определенной пользователем точки восстановления и нажмите кнопку **Применить**. Срок хранения по умолчанию для определенных пользователем точек восстановления составляет семь дней.

    ![Имя точки восстановления](./media/sql-data-warehouse-restore-points/creating-restore-point-11.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Восстановление существующего выделенного пула SQL (ранее — хранилища данных SQL)](sql-data-warehouse-restore-active-paused-dw.md)
- [Восстановление удаленного выделенного пула SQL (ранее — хранилища данных SQL)](sql-data-warehouse-restore-deleted-dw.md)
- [Восстановление из выделенного пула SQL для георезервного копирования (ранее — хранилище данных SQL)](sql-data-warehouse-restore-from-geo-backup.md)
