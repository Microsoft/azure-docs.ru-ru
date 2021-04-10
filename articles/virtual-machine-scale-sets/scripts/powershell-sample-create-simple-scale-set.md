---
title: Примеры Azure PowerShell. Создание базового масштабируемого набора виртуальных машин
description: Этот скрипт создает масштабируемый набор виртуальных машин Azure Windows Server 2016, к которому можно получить доступ через протокол удаленного рабочего стола.
author: mimckitt
ms.author: mimckitt
ms.topic: sample
ms.service: virtual-machine-scale-sets
ms.date: 03/27/2018
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: 05aa5e2ae70df7b517ad1d8cc592141bed66b7ed
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105935743"
---
# <a name="create-a-basic-virtual-machine-scale-set-with-powershell"></a>Создание базового масштабируемого набора виртуальных машин с помощью PowerShell
Этот скрипт позволяет создать масштабируемый набор виртуальных машин под управлением Windows Server 2016. После выполнения сценария можно получить доступ к экземплярам виртуальных машин по протоколу RDP.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="sample-script"></a>Пример скрипта


[!code-powershell[main](../../../powershell_scripts/virtual-machine-scale-sets/simple-scale-set/simple-scale-set.ps1 "Create a simple virtual machine scale set")]

## <a name="clean-up-deployment"></a>Очистка развертывания
Выполните следующую команду, чтобы удалить группу ресурсов, масштабируемый набор и все связанные с ними ресурсы.

```powershell
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="script-explanation"></a>Описание скрипта
Чтобы создать развертывание, скрипт использует следующие команды. Для каждого элемента в таблице приведены ссылки на документацию по команде.

| Get-Help | Примечания |
|---|---|
| [New-AzVmss](/powershell/module/az.compute/new-azvmss) | Создание масштабируемого набора виртуальных машин и всех вспомогательных ресурсов, включая виртуальную сеть, подсистему балансировки нагрузки и правила NAT. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов и все ресурсы, содержащиеся в ней. |

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о модуле Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).
