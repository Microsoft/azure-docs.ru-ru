---
title: PowerShell — создание пользовательского образа из VHD-файла в Службах лабораторий Azure
description: Этот сценарий PowerShell создает пользовательский образ из VHD-файла в службе "Службы лабораторий Azure".
ms.devlang: azurecli
ms.topic: sample
ms.date: 08/11/2020
ms.openlocfilehash: d22e6e1d226e826bf114a0fdb378879b828d9b4a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96022240"
---
# <a name="use-powershell-to-create-a-custom-image-from-a-vhd-file-in-azure-lab-services"></a>Создание пользовательского образа из VHD-файла в службе "Службы лабораторий Azure" с помощью PowerShell

Этот пример сценария PowerShell создает пользовательский образ из VHD-файла в службе "Службы лабораторий Azure".

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

## <a name="prerequisites"></a>Предварительные требования
* **Лаборатория.** Этот сценарий требует наличия лаборатории. 

## <a name="sample-script"></a>Пример скрипта

[!code-powershell[main](../../../powershell_scripts/devtest-lab/create-custom-image-from-vhd/create-custom-image-from-vhd.ps1 "Add external user to a lab")]

## <a name="script-explanation"></a>Описание скрипта

Этот сценарий использует следующие команды: 

| Get-Help | Примечания |
|---|---|
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Получает ресурсы. |
| [Get-AzStorageAccountKey](/powershell/module/az.storage/get-azstorageaccountkey) | Выводит список ключей доступа для учетной записи хранения Azure. |
| [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) | Добавляет развертывание Azure в группу ресурсов. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).

Дополнительные примеры сценариев PowerShell для службы "Службы лабораторий Azure" приведены [здесь](../samples-powershell.md).
