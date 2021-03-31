---
title: PowerShell. Резервное копирование приложения
description: Сведения об использовании Azure PowerShell для автоматизации процессов развертывания и управления в Службе приложений. В этом примере показано, как выполнить резервное копирование приложения.
author: msangapu-msft
tags: azure-service-management
ms.assetid: fc755f82-ca3e-4532-b251-690b699324d6
ms.topic: sample
ms.date: 10/30/2017
ms.author: msangapu
ms.custom: mvc, seodec18, devx-track-azurepowershell
ms.openlocfilehash: 8d41d491edd5fc6acf0b3b93d92771d0a99ece90
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89079971"
---
# <a name="back-up-a-web-app-using-powershell"></a>Резервное копирование веб-приложения с помощью PowerShell

С помощью этого скрипта можно создать веб-приложение в службе приложений со связанными ресурсами, а затем однократно создать для него резервную копию. 

При необходимости установите Azure PowerShell с помощью инструкции, приведенной в [руководстве Azure PowerShell](/powershell/azure/), а затем выполните команду `Connect-AzAccount`, чтобы создать подключение к Azure. 

## <a name="sample-script"></a>Пример скрипта

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/app-service/backup-onetime/backup-onetime.ps1?highlight=1-5 "Back up a web app")]

## <a name="clean-up-deployment"></a>Очистка развертывания 

Выполнив пример сценария, вы можете удалить группу ресурсов, веб-приложение и все связанные ресурсы, используя следующую команду.

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) | Создает учетную запись хранения. |
| [New-AzStorageContainer](/powershell/module/az.storage/new-AzStoragecontainer) | Позволяет создать контейнер хранилища Azure. |
| [New-AzStorageContainerSASToken;](/powershell/module/az.storage/new-AzStoragecontainersastoken) | Создание маркера SAS для контейнера хранилища Azure.  |
| [New-AzAppServicePlan](/powershell/module/az.websites/new-azappserviceplan) | Создает план службы приложений. |
| [New-AzWebApp](/powershell/module/az.websites/new-azwebapp) | Создает веб-приложение. |
| [New-AzWebAppBackup](/powershell/module/az.websites/new-azwebappbackup) | Позволяет создать резервную копию для веб-приложения. |
| [Get-AzWebAppBackupList](/powershell/module/az.websites/get-azwebappbackuplist) | Получение списка резервных копий веб-приложения. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры сценариев Azure PowerShell для веб-приложений службы приложений Azure доступны [здесь](../samples-powershell.md).
