---
title: PowerShell. Подключение к учетной записи хранения
description: Сведения об использовании Azure PowerShell для автоматизации процессов развертывания и управления в Службе приложений. В этом примере показано, как подключить приложение к учетной записи хранения.
tags: azure-service-management
ms.assetid: e4831bdc-2068-4883-9474-0b34c2e3e255
ms.topic: sample
ms.date: 03/20/2017
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 9c182c797515288038ae6a23d6d939d5ec855011
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89073930"
---
# <a name="connect-an-app-service-app-to-a-storage-account"></a>Подключение приложения Службы приложений к учетной записи хранения

В этой статье вы узнаете, как создать учетную запись хранения и приложение Службы приложений. Затем вы свяжете учетную запись хранения с приложением, используя параметры приложения.

При необходимости установите Azure PowerShell с помощью инструкции, приведенной в [руководстве Azure PowerShell](/powershell/azure/), а затем выполните команду `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="sample-script"></a>Пример скрипта

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/app-service/connect-to-storage/connect-to-storage.ps1 "Connect an app to a storage account")]

## <a name="clean-up-deployment"></a>Очистка развертывания 

Выполнив пример сценария, вы можете удалить группу ресурсов, приложение Службы приложений и все связанные ресурсы с помощью следующей команды.

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [New-AzAppServicePlan](/powershell/module/az.websites/new-azappserviceplan) | Создает план службы приложений. |
| [New-AzWebApp](/powershell/module/az.websites/new-azwebapp) | Создает приложение Службы приложений. |
| [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) | Создает учетную запись хранения. |
| [Get-AzStorageAccountKey](/powershell/module/az.storage/get-azstorageaccountkey) | Выводит список ключей доступа для учетной записи хранения Azure. |
| [Set-AzWebApp](/powershell/module/az.websites/set-azwebapp) | Изменяет конфигурацию приложения Службы приложений. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры сценариев Azure PowerShell для Службы приложений Azure доступны [здесь](../samples-powershell.md).
