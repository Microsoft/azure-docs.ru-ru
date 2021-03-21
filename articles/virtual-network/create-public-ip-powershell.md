---
title: Создание общедоступного IP-адреса Azure PowerShell
description: Узнайте, как создать общедоступный IP-адрес с помощью Azure PowerShell
services: virtual-network
documentationcenter: na
author: blehr
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/28/2020
ms.author: blehr
ms.openlocfilehash: f81e76e32bc0a50b945c54cdfcac0575540484bc
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102502456"
---
# <a name="quickstart-create-a-public-ip-address-using-azure-powershell"></a>Краткое руководство. Создание общедоступного IP-адреса с помощью Azure PowerShell

В этой статье показано, как создать ресурс общедоступного IP-адреса с помощью Azure PowerShell. Дополнительные сведения о том, к каким ресурсам можно связываться, разница между SKU "базовый" и "Стандартный" и другими связанными сведениями см. в разделе [общедоступные IP-адреса](./public-ip-addresses.md).  В этом примере мы рассмотрим только IPv4-адреса. Дополнительные сведения об IPv6-адресах см. в статье [IPv6 для виртуальной сети Azure](./ipv6-overview.md).

## <a name="prerequisites"></a>Предварительные условия

- Локальная установка Azure PowerShell или Azure Cloud Shell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально, для работы с этой статьей вам понадобится модуль Azure PowerShell 5.4.1 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). При использовании PowerShell на локальном компьютере также нужно запустить `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup) с именем **myResourceGroup** в расположении **eastus2** .

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroup'
$loc = 'eastus2'

New-AzResourceGroup -Name $rg -Location $loc
```
## <a name="create-public-ip"></a>Создать общедоступный IP-адрес

---
# <a name="standard-sku---using-zones"></a>[**SKU "Стандартный" — Использование зон**](#tab/option-create-public-ip-standard-zones)

>[!NOTE]
>Следующая команда работает для AZ. Network Module версии 4.5.0 или более поздней.  Дополнительные сведения о модулях PowerShell, используемых в настоящее время, см. в [документации по PowerShellGet](/powershell/module/powershellget/).

Используйте [New-азпублиЦипаддресс](/powershell/module/az.network/new-azpublicipaddress) для создания стандартного общедоступного IP-адреса, избыточного в виде зоны, с именем **мистандардзрпублиЦип** в **myResourceGroup**.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroup'
$loc = 'eastus2'
$pubIP = 'myStandardZRPublicIP'
$sku = 'Standard'
$alloc = 'Static'
$zone = 1,2,3

New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $alloc -SKU $sku -zone $zone
```
> [!IMPORTANT]
> Для AZ. Network modules старше 4.5.0 выполните указанную выше команду без указания параметра зоны, чтобы создать IP-адрес, избыточный в виде зоны. 
>

Чтобы создать стандартный общедоступный IP-адрес зональные в зона 2 с именем **мистандардзоналпублиЦип** в **myResourceGroup**, используйте следующую команду:

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroup'
$loc = 'eastus2'
$pubIP = 'myStandardZonalPublicIP'
$sku = 'Standard'
$alloc = 'Static'
$zone = 2

New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $alloc -SKU $sku -zone $zone
```

Обратите внимание, что указанные выше параметры для зон — это допустимые варианты выбора в регионах с [зоны доступности](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#availability-zones).

# <a name="standard-sku---no-zones"></a>[**SKU "Стандартный" — без зон**](#tab/option-create-public-ip-standard)

>[!NOTE]
>Следующая команда работает для AZ. Network Module версии 4.5.0 или более поздней.  Дополнительные сведения о модулях PowerShell, используемых в настоящее время, см. в [документации по PowerShellGet](/powershell/module/powershellget/).

Используйте [New-азпублиЦипаддресс](/powershell/module/az.network/new-azpublicipaddress) для создания стандартного общедоступного IP-адреса в качестве ресурса, отличного от зональные, с именем **мистандардпублиЦип** в **myResourceGroup**.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroup'
$loc = 'eastus2'
$pubIP = 'myStandardPublicIP'
$sku = 'Standard'
$alloc = 'Static'

New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $alloc -SKU $sku
```

Этот выбор допустим во всех регионах и является выбором по умолчанию для стандартных общедоступных IP-адресов в регионах без [зоны доступности](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#availability-zones).

# <a name="basic-sku"></a>[**SKU "Базовый"**](#tab/option-create-public-ip-basic)

Используйте [New-азпублиЦипаддресс](/powershell/module/az.network/new-azpublicipaddress) , чтобы создать базовый статический общедоступный IP-адрес с именем **мибасикпублиЦип** в **myResourceGroup**.  Основные общедоступные IP-адреса не имеют концепции зон доступности.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroup'
$loc = 'eastus2'
$pubIP = 'myBasicPublicIP'
$sku = 'Basic'
$alloc = 'Static'

New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $alloc -SKU $sku
```
Если IP-адрес приемлем для изменения со временем, можно выбрать **динамическое** назначение IP-адресов, изменив значение AllocationMethod на Dynamic.

---

## <a name="additional-information"></a>Дополнительные сведения 

Дополнительные сведения об отдельных переменных, перечисленных выше, см. в разделе [Управление общедоступными IP-адресами](./virtual-network-public-ip-address.md#create-a-public-ip-address).

## <a name="next-steps"></a>Дальнейшие действия
- Связывание [общедоступного IP-адреса с виртуальной машиной](./associate-public-ip-address-vm.md#azure-portal)
- См. дополнительные сведения об [общедоступных IP-адресах в Azure](./public-ip-addresses.md#public-ip-addresses).
- См. сведения обо всех [параметрах общедоступных IP-адресов](virtual-network-public-ip-address.md#create-a-public-ip-address).