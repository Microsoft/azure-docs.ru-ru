---
title: PowerShell. Обновление схемы синхронизации компонента "Синхронизация данных SQL"
description: Пример скрипта Azure PowerShell для обновления схемы синхронизации компонента "Синхронизация данных SQL"
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/12/2019
ms.openlocfilehash: 7d346d1ff30c138667749822b258bab4c6a621f4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92792724"
---
# <a name="use-powershell-to-update-the-sync-schema-in-an-existing-sync-group"></a>Использование PowerShell для обновления схемы синхронизации в существующей группе синхронизации
[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Этот пример скрипта Azure PowerShell обновляет схему синхронизации в существующей группе синхронизации компонента "Синхронизация данных SQL". Если вы синхронизируете несколько таблиц, этот скрипт поможет вам эффективно обновить схему синхронизации. В этом примере показано использование скрипта **UpdateSyncSchema**, который можно найти в репозитории GitHub (файл [UpdateSyncSchema.ps1](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/sql-data-sync/UpdateSyncSchema.ps1)).

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально, вам понадобится Az PowerShell 1.4.0 или последующей версии для работы с этим учебником. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

Общие сведения о синхронизации данных SQL см. в статье [Синхронизация данных в нескольких облачных и локальных базах данных с помощью синхронизации данных SQL Azure](../sql-data-sync-data-sql-server-sql-database.md).

> [!IMPORTANT]
> Служба "Синхронизация данных SQL" пока не поддерживает Управляемый экземпляр SQL Azure.

## <a name="examples"></a>Примеры

### <a name="add-all-tables-to-the-sync-schema"></a>Добавление всех таблиц в схему синхронизации

Следующий пример обновляет схему базы данных и добавляет все допустимые таблицы в центральную базу данных для синхронизации схемы.

```powershell-interactive
UpdateSyncSchema.ps1 -SubscriptionId <subscriptionId> -ResourceGroupName <resourceGroupName> -ServerName <serverName> -DatabaseName <databaseName> `
    -SyncGroupName <syncGroupName> -RefreshDatabaseSchema $true -AddAllTables $true
```

### <a name="add-and-remove-tables-and-columns"></a>Добавление и удаление таблиц и столбцов

Следующий пример добавляет `[dbo].[Table1]` и `[dbo].[Table2].[Column1]` в схему синхронизации и удаляет `[dbo].[Table3]`.

```powershell-interactive
UpdateSyncSchema.ps1 -SubscriptionId <subscriptionId> -ResourceGroupName <resourceGroupName> -ServerName <serverName> -DatabaseName <databaseName> `
    -SyncGroupName <syncGroupName> -TablesAndColumnsToAdd "[dbo].[Table1],[dbo].[Table2].[Column1]" -TablesAndColumnsToRemove "[dbo].[Table3]"
```

## <a name="script-parameters"></a>Параметры скрипта

Скрипт **UpdateSyncSchema** содержит следующие параметры:

| Параметр | Примечания |
|---|---|
| $subscriptionId | Подписка, в которой создана группа синхронизации. |
| $resourceGroupName | Группа ресурсов, в которой создана группа синхронизации.|
| $serverName | Имя сервера центральной базы данных.|
| $databaseName | Имя центральной базы данных. |
| $syncGroupName | Имя группы синхронизации |
| $memberName | Укажите имя элемента, если вы хотите загрузить схему базы данных из элемента синхронизации, а не из центральной базы данных. Если вы хотите загрузить схему базы данных из центральной базы данных, оставьте этот параметр пустым. |
| $timeoutInSeconds | Время ожидания до того, как скрипт обновит схему базы данных. По умолчанию это 900 секунд. |
| $refreshDatabaseSchema | Укажите, должен ли скрипт обновлять схему базы данных. Если предыдущая конфигурация схемы базы данных была изменена (например, при добавлении новой таблицы или нового столбца), вам необходимо обновить схему до ее перенастройки. Значение по умолчанию — false. |
| $addAllTables | Если это значение равно true, все допустимые таблицы и столбцы добавляются в схему синхронизации. Значения $TablesAndColumnsToAdd и $TablesAndColumnsToRemove игнорируются. |
| $tablesAndColumnsToAdd | Укажите таблицы или столбцы для добавления в схему синхронизации. Имя каждой таблицы или каждого столбца должно содержать имя схемы. Например: `[dbo].[Table1]`, `[dbo].[Table2].[Column1]`. Чтобы указать несколько имен таблиц или столбцов, разделите их запятыми (,). |
| $tablesAndColumnsToRemove | Укажите таблицы или столбцы для удаления из схемы синхронизации. Имя каждой таблицы или каждого столбца должно содержать имя схемы. Например: `[dbo].[Table1]`, `[dbo].[Table2].[Column1]`. Чтобы указать несколько имен таблиц или столбцов, разделите их запятыми (,). |

## <a name="script-explanation"></a>Описание скрипта

Скрипт **UpdateSyncSchema** содержит следующие команды: Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [Get-AzSqlSyncGroup](/powershell/module/az.sql/get-azsqlsyncgroup) | Возвращает сведения о группе синхронизации. |
| [Update-AzSqlSyncGroup](/powershell/module/az.sql/update-azsqlsyncgroup) | Создает группу синхронизации. |
| [Get-AzSqlSyncMember](/powershell/module/az.sql/get-azsqlsyncmember) | Возвращает сведения о члене синхронизации. |
| [Get-AzSqlSyncSchema](/powershell/module/az.sql/get-azsqlsyncschema) | Возвращает сведения о схеме синхронизации. |
| [Update-AzSqlSyncSchema](/powershell/module/az.sql/update-azsqlsyncschema) | Обновляет схему синхронизации. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры сценариев PowerShell для Базы данных SQL Azure можно найти в разделе [Примеры Azure PowerShell для Базы данных SQL Azure](../powershell-script-content-guide.md).

Дополнительные сведения о синхронизации данных SQL см. в следующих материалах:

- Общие сведения о [синхронизации данных между Базой данных SQL Azure и SQL Server с помощью службы "Синхронизация данных SQL" в Azure](../sql-data-sync-data-sql-server-sql-database.md).
- Настройка синхронизации данных
    - Использование портала Azure: [Учебник по настройке функции синхронизации данных SQL между Базой данных SQL Azure и SQL Server](../sql-data-sync-sql-server-configure.md)
    - Использование PowerShell
        -  [Использование PowerShell для синхронизации данных между несколькими базами данных в службе "База данных SQL Azure"](sql-data-sync-sync-data-between-sql-databases.md).
        -  [Использование PowerShell для синхронизации данных между Базой данных SQL Azure и SQL Server](sql-data-sync-sync-data-between-azure-onprem.md).
- Data Sync Agent: [Data Sync Agent для службы "Синхронизация данных SQL" в Azure](../sql-data-sync-agent-overview.md).
- Рекомендации: [Рекомендации по синхронизации данных SQL в Azure](../sql-data-sync-best-practices.md)
- Мониторинг: [Мониторинг синхронизации данных SQL с помощью журналов Azure Monitor](../monitor-tune-overview.md)
- Устранение неполадок: [Устранение неполадок с синхронизацией данных SQL в Azure](../sql-data-sync-troubleshoot.md)
- Обновление схемы синхронизации
    - Использование Transact-SQL: [Автоматическая репликация изменений схемы в службе "Синхронизация данных SQL" в Azure](../sql-data-sync-update-sync-schema.md).

Дополнительные сведения о Базе данных SQL см. в разделах:

- [Функции службы базы данных SQL Azure](../sql-database-paas-overview.md)
- [Управление жизненным циклом базы данных](/previous-versions/sql/sql-server-guides/jj907294(v=sql.110))