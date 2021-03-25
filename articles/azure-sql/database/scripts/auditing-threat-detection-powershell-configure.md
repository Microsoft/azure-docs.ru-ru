---
title: Использование примера скрипта PowerShell для настройки аудита и Расширенной защиты от угроз для Базы данных SQL Azure
description: Использование примера скрипта Azure PowerShell для настройки аудита и Расширенная защита от угроз для Базы данных SQL Azure.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: security, sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: DavidTrigano
ms.author: datrigan
ms.reviewer: sstein, vanto
ms.date: 04/28/2020
ms.openlocfilehash: 1dc4630e429641fad1ef9efa96bbfa920c6d8963
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91321543"
---
# <a name="use-powershell-to-configure-sql-database-auditing-and-advanced-threat-protection"></a>Узнайте, как настроить аудит и Расширенную защита от угроз для Базы данных SQL с помощью PowerShell.
[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Этот пример скрипта PowerShell настраивает аудит и Расширенную защиту от угроз для Базы данных SQL Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально, вам понадобится Az PowerShell 1.4.0 или последующей версии для работы с этим руководством. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="sample-script"></a>Пример скрипта

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/database-auditing-and-threat-detection/database-auditing-and-threat-detection.ps1?highlight=15-16 "Configure auditing and threat detection")]

## <a name="clean-up-deployment"></a>Очистка развертывания

Используйте следующую команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourcegroupname
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Создает сервер. |
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Создает базу данных или эластичный пул. |
| [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) | Создание учетной записи хранения. |
| [Set-AzSqlDatabaseAuditing](/powershell/module/az.sql/set-azsqldatabaseauditing) | Задает политику аудита для базы данных. |
| [Set-AzSqlDatabaseThreatDetectionPolicy](/powershell/module/az.sql/set-azsqldatabasethreatdetectionpolicy) | Задает политику Расширенной защиты от угроз для базы данных. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов со всеми вложенными ресурсами. |
|||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры скриптов PowerShell для базы данных SQL Azure можно найти [здесь](../powershell-script-content-guide.md).
