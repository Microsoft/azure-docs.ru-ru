---
title: Сценарий PowerShell для задания допустимых размеров виртуальных машин в службе "Службы лабораторий Azure" | Документация Майкрософт
description: В этой статье показан пример скрипта PowerShell, который задает допустимые размеры виртуальных машин в Службах лабораторий Azure.
ms.devlang: azurecli
ms.topic: sample
ms.date: 08/11/2020
ms.openlocfilehash: 476b86b7c577db17efc39dbac64a527432c916b6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95998144"
---
# <a name="use-powershell-to-set-allowed-vm-sizes-in-azure-lab-services"></a>Задание допустимых размеров виртуальных машин в службе "Службы лабораторий Azure" с помощью PowerShell

Этот пример сценария PowerShell задает допустимые размеры виртуальных машин в службе "Службы лабораторий Azure".

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

## <a name="prerequisites"></a>Предварительные требования
* **Лаборатория.** Этот сценарий требует наличия лаборатории. 

## <a name="sample-script"></a>Пример скрипта

[!code-powershell[main](../../../powershell_scripts/devtest-lab/set-allowed-vm-sizes-in-lab/set-allowed-vm-sizes-in-lab.ps1 "Add external user to a lab")]

## <a name="script-explanation"></a>Описание скрипта

Этот сценарий использует следующие команды: 

| Get-Help | Примечания |
|---|---|
| Find-AzResource | Выполняет поиск ресурсов на основе указанных параметров. |
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Получает ресурсы. |
| [Set-AzResource](/powershell/module/az.resources/set-azresource) | Изменяет ресурс. |
| [New-AzResource](/powershell/module/az.resources/new-azresource) | Создание ресурса. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).

Дополнительные примеры сценариев PowerShell для службы "Службы лабораторий Azure" приведены [здесь](../samples-powershell.md).
