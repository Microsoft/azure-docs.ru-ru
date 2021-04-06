---
title: Создание настраиваемой роли в лаборатории в Azure DevTest Labs — PowerShell
description: В этой статье представлен скрипт Azure PowerShell, который создает настраиваемую роль в лаборатории в Azure DevTest Labs.
ms.devlang: azurecli
ms.topic: sample
ms.date: 08/11/2020
ms.openlocfilehash: 3757d6c16a762a0f76f4718f117da93322af074c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96021203"
---
# <a name="use-powershell-to-create-a-custom-role-in-a-lab-in-azure-devtest-labs"></a>Создание настраиваемой роли в лаборатории Azure DevTest Labs с помощью PowerShell

Этот пример сценария PowerShell создает настраиваемую роль для использования в лаборатории Azure DevTest Labs. 

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

## <a name="prerequisites"></a>Предварительные требования
* **Лаборатория.** Этот сценарий требует наличия лаборатории. 

## <a name="sample-script"></a>Пример скрипта

[!code-powershell[main](../../../powershell_scripts/devtest-lab/create-custom-role-in-lab/create-custom-role-in-lab.ps1 "Add external user to a lab")]

## <a name="script-explanation"></a>Описание скрипта

Этот сценарий использует следующие команды: 

| Get-Help | Примечания |
|---|---|
| [Get-AzProviderOperation](/powershell/module/az.resources/get-azprovideroperation) | Возвращает операции для поставщика ресурсов Azure, защищаемые с помощью Azure RBAC. |
| [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition) | Выводит список всех ролей Azure, доступных для назначения. |
| [New-AzRoleDefinition](/powershell/module/az.resources/new-azroledefinition) | Создает настраиваемую роль. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).

Дополнительные примеры сценариев PowerShell для службы "Службы лабораторий Azure" приведены [здесь](../samples-powershell.md).
