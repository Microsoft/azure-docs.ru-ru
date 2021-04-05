---
title: PowerShell. Восстановление геоизбыточной резервной копии в управляемом экземпляре Базы данных SQL Azure
description: Пример скрипта PowerShell по восстановлению управляемого экземпляра Базы данных SQL Azure из геоизбыточной резервной копии.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: PowerShell
ms.topic: sample
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 07/03/2019
ms.openlocfilehash: f04e4b5a44dccdc3aaeabe6b4144836b0be7354c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92790786"
---
# <a name="use-powershell-to-restore-an-azure-sql-managed-instance-database-to-another-geo-region"></a>Восстановление управляемого экземпляра Базы данных SQL Azure в другом географическом регионе с помощью PowerShell

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqlmi.md)]

Этот пример скрипта PowerShell восстанавливает управляемый экземпляр Базы данных SQL Azure из удаленного географического региона (геовосстановление).  

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально, вам понадобится Azure PowerShell 1.4.0 или последующей версии для работы с этим руководством. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="sample-script"></a>Пример скрипта

```azurepowershell-interactive
# Connect-AzAccount
# The SubscriptionId in which to create these objects
$SubscriptionId = '<put subscription_id here>'
# Set the information for your managed instance
$SourceResourceGroupName = "myResourceGroup-$(Get-Random)"
$SourceInstanceName = "myManagedInstance-$(Get-Random)"
$SourceDatabaseName = "myInstanceDatabase-$(Get-Random)"

# Set the information for your destination managed instance
$TargetResourceGroupName = "myTargetResourceGroup-$(Get-Random)"
$TargetInstanceName = "myTargetManagedInstance-$(Get-Random)"
$TargetDatabaseName = "myTargetInstanceDatabase-$(Get-Random)"

Connect-AzAccount
Set-AzContext -SubscriptionId $SubscriptionId

$backup = Get-AzSqlInstanceDatabaseGeoBackup `
-ResourceGroupName $SourceResourceGroupName `
-InstanceName $SourceInstanceName `
-Name $SourceDatabaseName

$backup | Restore-AzSqlInstanceDatabase -FromGeoBackup `
-TargetInstanceDatabaseName $TargetDatabaseName `
-TargetInstanceName $TargetInstanceName `
-TargetResourceGroupName $TargetResourceGroupName

```

## <a name="clean-up-deployment"></a>Очистка развертывания

Используйте следующую команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```azurepowershell-interactive
Remove-AzResourceGroup -ResourceGroupName $TargetResourceGroupName
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/New-AzResourceGroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [Get-AzSqlInstanceDatabaseGeoBackup](/powershell/module/az.sql/Get-AzSqlInstanceDatabaseGeoBackup) | Создает геоизбыточную резервную копию управляемого экземпляра базы данных SQL. |
| [Restore-AzSqlInstanceDatabase](/powershell/module/az.sql/Restore-AzSqlInstanceDatabase) | Создает базу данных в управляемом экземпляре SQL из геоизбыточной резервной копии. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры скриптов PowerShell для Базы данных SQL Azure можно найти [здесь](../../database/powershell-script-content-guide.md).