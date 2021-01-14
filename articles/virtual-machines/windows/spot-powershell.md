---
title: Развертывание виртуальных машин точки Azure с помощью PowerShell
description: Узнайте, как использовать Azure PowerShell для развертывания плашечных виртуальных машин, чтобы сэкономить на затратах.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 06/26/2020
ms.author: cynthn
ms.reviewer: jagaveer
ms.openlocfilehash: 0ca3c99aed8160161c125a89da3cb176c6e745f6
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/14/2021
ms.locfileid: "98202068"
---
# <a name="deploy-spot-vms-using-azure-powershell"></a>Развертывание плашечных виртуальных машин с помощью Azure PowerShell


Использование [плашечных виртуальных машин](../spot-vms.md) позволяет использовать преимущества нашей неиспользуемой емкости с значительной экономией затрат. В любой момент времени, когда эта емкость становится нужна, инфраструктура Azure вытесняет точечные виртуальные машины. Это означает, что точечные виртуальные машины прекрасно подходят для рабочих нагрузок, для которых допустимы прерывания, например для заданий пакетной обработки, сред разработки или тестирования, больших вычислительных рабочих нагрузок и других.

Цена на точечные виртуальные машины может изменяться в зависимости от региона и ценовой категории. Дополнительные сведения см. на страницах с информацией о ценах на виртуальные машины [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) и [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). Дополнительные сведения о настройке максимальной цены см. в разделе [точки на виртуальных машинах — цены](../spot-vms.md#pricing).

Вы можете задать максимальную цену, которую вы хотите платить, в час, для виртуальной машины. Максимальная цена на плашечную виртуальную машину можно установить в долларах США (USD), используя до 5 десятичных разрядов. Например, значение `0.98765` определяет максимальную цену 0,98765 долларов США в час. Если вы укажете для максимальной цены значение `-1`, виртуальная машине не будет вытесняться по критерию цены. Цена на такую виртуальную машину будет определяться меньшим из двух значений: текущая цена точечных виртуальных машин или цена на стандартные виртуальные машины, но только при условии наличия емкости и соблюдения квоты.


## <a name="create-the-vm"></a>Создание виртуальной машины

Создайте Спотвм с помощью [New-азвмконфиг](/powershell/module/az.compute/new-azvmconfig) , чтобы создать конфигурацию. Включите `-Priority Spot` и задайте `-MaxPrice` одно из следующих действий:
- `-1` так что виртуальная машина не будет исключена на основе цены.
- сумма в долларах, до 5 цифр. Например, это `-MaxPrice .98765` означает, что виртуальная машина будет освобождена, когда цена за спотвм займет около $. 98765 в час.


В этом примере создается Спотвм, который не будет освобожден на основе ценообразования (только в том случае, если Azure требуется обратная емкость). Политика вытеснения устанавливается таким образом, чтобы освободить виртуальную машину, чтобы ее можно было перезапустить позднее. Если вы хотите удалить виртуальную машину и базовый диск при удалении виртуальной машины, присвойте параметру `-EvictionPolicy` значение `Delete` в `New-AzVMConfig` .


```azurepowershell-interactive
$resourceGroup = "mySpotRG"
$location = "eastus"
$vmName = "mySpotVM"
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."
New-AzResourceGroup -Name $resourceGroup -Location $location
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
   -Name mySubnet -AddressPrefix 192.168.1.0/24
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup `
   -Location $location -Name MYvNET -AddressPrefix 192.168.0.0/16 `
   -Subnet $subnetConfig
$pip = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Location $location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
  -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 3389 -Access Allow
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location $location `
  -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface -Name myNic -ResourceGroupName $resourceGroup -Location $location `
  -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration and set this to be a Spot VM

$vmConfig = New-AzVMConfig -VMName $vmName -VMSize Standard_D1 -Priority "Spot" -MaxPrice -1 -EvictionPolicy Deallocate | `
Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred | `
Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2016-Datacenter -Version latest | `
Add-AzVMNetworkInterface -Id $nic.Id

New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig
```

После создания виртуальной машины можно выполнить запрос, чтобы увидеть максимальную цену для всех виртуальных машин в группе ресурсов.

```azurepowershell-interactive
Get-AzVM -ResourceGroupName $resourceGroup | `
   Select-Object Name,@{Name="maxPrice"; Expression={$_.BillingProfile.MaxPrice}}
```

## <a name="simulate-an-eviction"></a>Имитация вытеснения

Вы можете [имитировать вытеснение](/rest/api/compute/virtualmachines/simulateeviction) для ПЛАШЕЧНОЙ виртуальной машины, чтобы проверить, насколько хорошо ваше приложение будет ответил внезапного вытеснения. 

Замените следующие сведения: 

- `subscriptionId`
- `resourceGroupName`
- `vmName`


```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/simulateEviction?api-version=2020-06-01
```

## <a name="next-steps"></a>Дальнейшие действия

Можно также создать плашечную виртуальную машину с помощью [Azure CLI](../linux/spot-cli.md), [портала](../spot-portal.md) или [шаблона](../linux/spot-template.md).

Запросите актуальные сведения о ценах с помощью [API розничных цен Azure](/rest/api/cost-management/retail-prices/azure-retail-prices) для получения сведений о ценах. Объект `meterName` и `skuName` будет содержать `Spot` .

Если возникает ошибка, см. раздел [коды ошибок](../error-codes-spot.md).