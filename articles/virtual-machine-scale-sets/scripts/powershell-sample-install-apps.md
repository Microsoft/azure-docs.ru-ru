---
title: Примеры для Azure PowerShell. Установка приложений
description: Этот скрипт создает виртуальную машину Azure под управлением Windows Server 2016 и использует расширение пользовательских скриптов для установки базовых веб-приложений.
author: mimckitt
ms.author: mimckitt
ms.topic: sample
ms.service: virtual-machine-scale-sets
ms.subservice: powershell
ms.date: 06/25/2020
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: ea4dbf06bd1c129131d57748986e22feca80d808
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89079614"
---
# <a name="install-applications-into-a-virtual-machine-scale-set-with-powershell"></a>Установка приложений в масштабируемый набор виртуальных машин с помощью PowerShell
Этот скрипт создает виртуальную машину Azure под управлением Windows Server 2016 и использует расширение пользовательских скриптов для установки базовых веб-приложений. После выполнения скрипта можно получить доступ к веб-приложению с помощью веб-браузера.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="sample-script"></a>Пример скрипта

[!code-powershell[main](../../../powershell_scripts/virtual-machine-scale-sets/install-apps/install-apps.ps1 "Install apps into a scale set")]

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
| [Get-AzVmss](/powershell/module/az.compute/get-azvmss) | Получение информации о масштабируемом наборе виртуальных машин. |
| [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension) | Добавление расширения виртуальной машины для настраиваемого скрипта установки базового веб-приложения. |
| [Update-AzVmss](/powershell/module/az.compute/update-azvmss) | Обновление модели масштабируемого набора виртуальных машин для применения расширения виртуальной машины. |
| [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) | Позволяет получить сведения об общедоступном IP-адресе, который использует подсистема балансировки нагрузки. |
|  [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов и все ресурсы, содержащиеся в ней. |

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о модуле Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).
