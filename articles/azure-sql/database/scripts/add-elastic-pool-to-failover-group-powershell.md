---
title: PowerShell. Добавление эластичного пула в группу автоматической отработки отказа
description: Пример скрипта Azure PowerShell для создания эластичного пула в службе "База данных SQL Azure", добавления его в группу автоматической отработки отказа и тестовой отработки отказа.
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
ms.openlocfilehash: a8326249b48e98eccecb5f0c4953add26e167560
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94594119"
---
# <a name="use-powershell-to-add-an-elastic-pool-to-a-failover-group"></a>Добавление эластичного пула в группу отработки отказа с помощью PowerShell
[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Этот сценарий Azure PowerShell позволяет создать базу данных в службе "База данных SQL Azure", добавить ее в эластичный пул, создать группу отработки отказа и выполнить тестовую отработку отказа.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально, вам понадобится Az PowerShell 1.4.0 или последующей версии для работы с этим учебником. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="sample-scripts"></a>Примеры сценариев

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/failover-groups/add-elastic-pool-to-failover-group-az-ps.ps1 "Add elastic pool to a failover group")]

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
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Создает сервер, на котором размещены базы данных и эластичные пулы. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Создает для сервера правило брандмауэра на уровне сервера. |
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Создает новую базу данных. |
| [New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool) | Создает пул эластичных баз данных.|
| [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase) | Определяет свойства базы данных или перемещает ее в эластичный пул. |
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) | Создает группу отработки отказа. |
| [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) | Получает одну или несколько баз данных. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup) | Добавляет одну или несколько баз данных в группу отработки отказа. |
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Возвращает или перечисляет группы отработки отказа для базы данных. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup)| Выполняет отработку отказа для группы отработки отказа базы данных. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры сценариев PowerShell для базы данных SQL Azure можно найти [здесь](../powershell-script-content-guide.md).
