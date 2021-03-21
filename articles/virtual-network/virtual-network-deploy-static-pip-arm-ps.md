---
title: Создание виртуальной машины со статическим общедоступным IP-адресом с помощью PowerShell | Документация Майкрософт
description: Создание виртуальной машины с статическим общедоступным IP-адресом с помощью PowerShell. Статические общедоступные IP-адреса — это адреса, которые никогда не меняются.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: ip-services
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/08/2018
ms.author: allensu
ms.openlocfilehash: 047a6db82e65c97deba5270d181f72315a67e82c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98791262"
---
# <a name="create-a-virtual-machine-with-a-static-public-ip-address-using-powershell"></a>Создание виртуальной машины со статическим общедоступным IP-адресом с помощью PowerShell


Вы можете создать виртуальную машину со статическим общедоступным IP-адресом. Общедоступный IP-адрес позволяет подключиться к виртуальной машине из Интернета. Присвойте статический общедоступный IP-адрес, а не динамический, чтобы гарантировать, что адрес никогда не изменится. Подробнее о [статических общедоступных IP-адресах](./public-ip-addresses.md#allocation-method). Чтобы изменить общедоступный IP-адрес, присвоенный существующей виртуальной машине, с динамического на статический или использовать частные IP-адреса, см. статью [Добавление, изменение и удаление IP-адресов для сетевого интерфейса Azure](virtual-network-network-interface-addresses.md). За использование общедоступных IP-адресов взимается [номинальная плата](https://azure.microsoft.com/pricing/details/ip-addresses). Кроме того, существует [ограничение](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits) на число общедоступных IP-адресов, которые можно использовать в рамках одной подписки.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]


## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Указанные ниже действия можно выполнить с помощью локального компьютера или Azure Cloud Shell. Чтобы использовать локальный компьютер, на нем должна быть установлена служба [Azure PowerShell](/powershell/azure/install-az-ps?toc=%2fazure%2fvirtual-network%2ftoc.json). Чтобы использовать Azure Cloud Shell, выберите функцию **Попробовать** в правом верхнем углу любого окна со следующими командами. Через Cloud Shell будет выполнен вход в Azure.

1. При использовании Cloud Shell перейдите к шагу 2. Откройте сеанс командной строки и войдите в Azure с помощью команды `Connect-AzAccount`.
2. Создайте группу ресурсов с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). В следующем примере создается группа ресурсов в регионе Azure "Восточная часть США".

   ```azurepowershell-interactive
   New-AzResourceGroup -Name myResourceGroup -Location EastUS
   ```

3. Создайте виртуальную машину с помощью команды [New-AzVM](/powershell/module/az.Compute/New-azVM) . Параметр `-AllocationMethod "Static"` присваивает статический общедоступный IP-адрес виртуальной машине. В следующем примере создается виртуальная машина Windows Server со статическим общедоступным IP-адресом SKU "Базовый" с именем *myPublicIpAddress*. Когда появится запрос, укажите имя пользователя и пароль в качестве учетных данных для входа на виртуальную машину.

   ```azurepowershell-interactive
   New-AzVm `
     -ResourceGroupName "myResourceGroup" `
     -Name "myVM" `
     -Location "East US" `
     -PublicIpAddressName "myPublicIpAddress" `
     -AllocationMethod "Static"
   ```

   Если у общедоступного IP-адреса должен быть SKU "Стандартный", вам необходимо выполнить отдельные действия: [создать общедоступный IP-адрес](virtual-network-public-ip-address.md#create-a-public-ip-address), [создать сетевой интерфейс](virtual-network-network-interface.md#create-a-network-interface), [назначить общедоступный IP-адрес сетевому интерфейсу](virtual-network-network-interface-addresses.md#add-ip-addresses), а затем [создать виртуальную машину с помощью сетевого интерфейса](virtual-network-network-interface-vm.md#add-existing-network-interfaces-to-a-new-vm). Дополнительные сведения о [номерах SKU общедоступных IP-адресов](./public-ip-addresses.md#sku). Если виртуальная машина добавляется в серверный пул общедоступной подсистемы Azure Load Balancer, номера SKU общедоступных IP-адресов виртуальной машины и подсистемы балансировки нагрузки должны совпадать. Подробные сведения см. в статье [Что такое Azure Load Balancer](../load-balancer/skus.md).

4. Просмотрите назначенный общедоступный IP-адрес и убедитесь, что он был создан как статический адрес с помощью [Get-азпублиЦипаддресс](/powershell/module/az.network/get-azpublicipaddress):

   ```azurepowershell-interactive
   Get-AzPublicIpAddress `
     -ResourceGroupName "myResourceGroup" `
     -Name "myPublicIpAddress" `
     | Select "IpAddress", "PublicIpAllocationMethod" `
     | Format-Table
   ```

   Платформа Azure присвоила общедоступный IP-адрес из адресов, используемых в регионе, в котором вы создали виртуальную машину. Вы можете загрузить список диапазонов (префиксов) для облаков Azure: [общедоступное](https://www.microsoft.com/download/details.aspx?id=56519), облако [правительства США](https://www.microsoft.com/download/details.aspx?id=57063), облако для [Китая](https://www.microsoft.com/download/details.aspx?id=57062) и [Германии](https://www.microsoft.com/download/details.aspx?id=57064).

> [!WARNING]
> Не изменяйте параметры IP-адресов в операционной системе виртуальной машины. Общедоступные IP-адреса Azure не поддерживаются в операционной системе. Вы можете добавлять параметры частных IP-адресов в операционную систему, но это рекомендуется делать только при необходимости и только после прочтения раздела о [добавлении частных IP-адресов в операционную систему](virtual-network-network-interface-addresses.md#private).

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы с помощью командлета [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup):

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения об [общедоступных IP-адресах](./public-ip-addresses.md#public-ip-addresses) в Azure
- Дополнительные сведения о всех [параметрах общедоступного IP-адреса](virtual-network-public-ip-address.md#create-a-public-ip-address)
- Дополнительные сведения о [частных IP-адресах](./private-ip-addresses.md) и назначении [статического частного IP-адреса](virtual-network-network-interface-addresses.md#add-ip-addresses) виртуальной машине Azure
- Дополнительные сведения о создании виртуальных машин [Linux](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json) и [Windows](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
