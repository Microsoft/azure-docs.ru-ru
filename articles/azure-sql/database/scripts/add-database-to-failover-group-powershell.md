---
title: PowerShell. Добавление базы данных в группу автоматической отработки отказа
description: Используйте пример сценария Azure PowerShell для создания базы данных в службе "База данных SQL Azure", добавления этой базы данных в группу автоматической отработки отказа и проверки отработки отказа.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 07/16/2019
ms.openlocfilehash: 34d296e51f25c492d7f66347a2e2b190c39e83f1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94594136"
---
# <a name="use-powershell-to-add-a-database-to-a-failover-group"></a>Добавление базы данных в группу отработки отказа с помощью PowerShell

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Этот пример сценария PowerShell создает отдельную базу данных в Базе данных SQL Azure и группу отработки отказа, добавлять в нее базу данных и выполняет тестовую отработку отказа.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Если требуется установить и использовать PowerShell локально, для работы с этим руководством вам понадобится AZ PowerShell 1.4.0 или последующей версии. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="sample-scripts"></a>Примеры сценариев

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/failover-groups/add-single-db-to-failover-group-az-ps.ps1 "Add a database to a failover group")]

## <a name="clean-up-deployment"></a>Очистка развертывания

Используйте следующую команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Создает сервер. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Создает для сервера правило брандмауэра на уровне сервера. |
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Создает новую базу данных. |
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) | Создает группу отработки отказа. |
| [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) | Получает одну или несколько баз данных. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup) | Добавляет одну или несколько баз данных в группу отработки отказа. |
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Получает или перечисляет группы отработки отказа. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup)| Выполняет отработку отказа для группы отработки отказа. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры скриптов PowerShell для базы данных SQL Azure можно найти [здесь](../powershell-script-content-guide.md).
