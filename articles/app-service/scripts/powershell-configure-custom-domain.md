---
title: PowerShell. Назначение личного домена
description: Сведения об использовании Azure PowerShell для автоматизации процессов развертывания и управления в Службе приложений. В этом примере показано, как назначить личный домен приложению.
author: msangapu-msft
tags: azure-service-management
ms.assetid: 356f5af9-f62e-411c-8b24-deba05214103
ms.topic: sample
ms.date: 03/20/2017
ms.author: msangapu
ms.custom: mvc, seodec18, devx-track-azurepowershell
ms.openlocfilehash: 749250f4ee5e1a0a39c9dc36795fa07e9da743a0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89072236"
---
# <a name="assign-a-custom-domain-to-a-web-app-using-powershell"></a>Назначение пользовательского домена веб-приложению с помощью PowerShell

Этот пример скрипта создает веб-приложение в службе приложений со связанными ресурсами, а затем сопоставляет с ним `www.<yourdomain>`. 

При необходимости установите Azure PowerShell с помощью инструкции, приведенной в [руководстве Azure PowerShell](/powershell/azure/), а затем выполните команду `Connect-AzAccount`, чтобы создать подключение к Azure. Кроме того, нужен доступ к странице конфигурации DNS вашего регистратора доменных имен.

## <a name="sample-script"></a>Пример скрипта

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/app-service/map-custom-domain/map-custom-domain.ps1?highlight=1 "Assign a custom domain to a web app")]

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
| [New-AzAppServicePlan](/powershell/module/az.websites/new-azappserviceplan) | Создает план службы приложений. |
| [New-AzWebApp](/powershell/module/az.websites/new-azwebapp) | Создает веб-приложение. |
| [Set-AzAppServicePlan](/powershell/module/az.websites/set-azappserviceplan) | Обновляет план службы приложений для изменения ценовой категории. |
| [Set-AzWebApp](/powershell/module/az.websites/set-azwebapp) | Изменяет конфигурацию веб-приложения. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры сценариев Azure PowerShell для веб-приложений службы приложений Azure доступны [здесь](../samples-powershell.md).
