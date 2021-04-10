---
title: Примеры Azure PowerShell. Создание полного масштабируемого набора виртуальных машин
description: Этот скрипт создает масштабируемый набор виртуальных машин Windows Server 2016, где настраиваются и создаются индивидуальные ресурсы.
author: mimckitt
ms.author: mimckitt
ms.topic: sample
ms.service: virtual-machine-scale-sets
ms.date: 05/29/2018
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: 31d779adc0ce08331486d615a6af2095323189bd
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105935487"
---
# <a name="create-a-complete-virtual-machine-scale-set-with-powershell"></a>Создание полного масштабируемого набора виртуальных машин с помощью PowerShell

Этот скрипт позволяет создать масштабируемый набор виртуальных машин под управлением Windows Server 2016. Индивидуальные ресурсы настраиваются и создаются чаще, чем используются [встроенные возможности создания ресурсов, доступные в New-AzVmss](powershell-sample-create-simple-scale-set.md). После выполнения сценария можно получить доступ к экземплярам виртуальных машин по протоколу RDP.


[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="sample-script"></a>Пример скрипта

[!code-powershell[main](../../../powershell_scripts/virtual-machine-scale-sets/complete-scale-set/complete-scale-set.ps1 "Create a complete virtual machine scale set")]

## <a name="clean-up-deployment"></a>Очистка развертывания
Выполните следующую команду, чтобы удалить группу ресурсов, масштабируемый набор и все связанные с ними ресурсы.

```powershell
Remove-AzResourceGroup -Name $resourceGroupName
```

## <a name="script-explanation"></a>Описание скрипта
Чтобы создать развертывание, скрипт использует следующие команды. Для каждого элемента в таблице приведены ссылки на документацию по команде.

| Get-Help | Примечания |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Создает конфигурацию подсети. Эта конфигурация используется в процессе создания виртуальной сети. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Создает виртуальную сеть. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Создает общедоступный IP-адрес. |
| [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) | Создает конфигурацию внешних IP-адресов для балансировщика нагрузки. |
| [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) | Создает конфигурацию внутреннего пула адресов для подсистемы балансировки нагрузки. |
| [New-AzLoadBalancerInboundNatRuleConfig](/powershell/module/az.network/new-azloadbalancerinboundnatruleconfig) | Создает конфигурацию правил NAT для балансировщика нагрузки. |
| [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer) | Создает подсистему балансировки нагрузки. |
| [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/new-azloadbalancerprobeconfig) | Создает конфигурацию пробы для балансировщика нагрузки. |
| [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/new-azloadbalancerruleconfig) | Создает конфигурацию правил для балансировщика нагрузки. |
| [Set-AzLoadBalancer](/powershell/module/az.Network/Set-azLoadBalancer) | Обновляет подсистему балансировки нагрузки предоставленной информацией. |
| [New-AzVmssIpConfig](/powershell/module/az.Compute/New-azVmssIpConfig) | Создает конфигурацию протокола IP для экземпляров масштабируемого набора ВМ. Экземпляры ВМ подключены к подсистеме балансировки нагрузки, серверному пулу, пулу NAT и подсети виртуальной сети. |
| [New-AzVmssConfig](/powershell/module/az.Compute/New-azVmssConfig) | Создает конфигурацию масштабируемого набора. Такая конфигурация содержит сведения, такие как количество экземпляров создаваемых виртуальных машин, номер SKU ВМ (размер) и режим политики обновления. Конфигурация добавляется дополнительными командлетами и используется во время создания масштабируемого набора. |
| [Set-AzVmssStorageProfile](/powershell/module/az.Compute/Set-azVmssStorageProfile) | Определяет образ, используемый для экземпляров виртуальной машины, и добавляет его в конфигурацию масштабируемого набора. |
| [Set-AzVmssOsProfile](/powershell/module/az.Compute/Set-azVmssStorageProfile) | Определяет учетные данные администратора и префикс имени виртуальной машины. Добавьте эти значения в конфигурацию масштабируемого набора. |
| [Add-AzVmssNetworkInterfaceConfiguration](/powershell/module/az.Compute/Add-azVmssNetworkInterfaceConfiguration) | Добавляет виртуальный сетевой интерфейс для экземпляров виртуальных машин на основе конфигурации IP. Добавьте эти значения в конфигурацию масштабируемого набора. |
| [New-AzVmss](/powershell/module/az.Compute/New-azVmss) | Создает масштабируемый набор на основе данных, указанных в конфигурации масштабируемого набора. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов и все ресурсы, содержащиеся в ней. |

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о модуле Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).
