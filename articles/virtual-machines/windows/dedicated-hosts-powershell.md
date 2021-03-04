---
title: Развертывание выделенных узлов Azure с помощью Azure PowerShell
description: Развертывание виртуальных машин на выделенных узлах с помощью Azure PowerShell.
author: cynthn
ms.service: virtual-machines
ms.subservice: dedicated-hosts
ms.topic: how-to
ms.workload: infrastructure
ms.date: 11/12/2020
ms.author: cynthn
ms.reviewer: zivr
ms.openlocfilehash: ed6319d5374db56cfe85e7ef9413480e523d9a34
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102050891"
---
# <a name="deploy-vms-to-dedicated-hosts-using-the-azure-powershell"></a>Развертывание виртуальных машин на выделенных узлах с помощью Azure PowerShell

В этой статье рассказывается, как создать [выделенный узел](../dedicated-hosts.md) Azure для размещения виртуальных машин. 

Убедитесь, что установлен Azure PowerShell версии 2.8.0 или более поздней, и вы вошли в учетную запись Azure в с помощью `Connect-AzAccount` . 

## <a name="limitations"></a>Ограничения

- Размеры и типы оборудования, доступные для выделенных узлов, зависят от региона. Дополнительные сведения см. на [странице с ценами узлов](https://aka.ms/ADHPricing).

## <a name="create-a-host-group"></a>Создание группы узлов

**Группа узлов** — это ресурс, который представляет собой коллекцию выделенных узлов. Группа узлов создается в регионе и зоне доступности, и в нее добавляются узлы. При планировании высокого уровня доступности имеются дополнительные параметры. Для выделенных узлов можно использовать один или оба следующих варианта: 
- Развертывание в нескольких зонах доступности. В этом случае группа узлов должна быть в каждой из зон, которую необходимо использовать.
- Развертывание в нескольких доменах сбоя, сопоставленных с физическими стойками. 
 
В любом случае необходимо указать количество доменов сбоя для группы узлов. Если группа не должна охватывать несколько доменов сбоя, используйте число доменов сбоя, равное 1. 

Можно также использовать как зоны доступности, так и домены сбоя. В этом примере создается группа узлов в зоне 1 с 2 доменами сбоя. 


```azurepowershell-interactive
$rgName = "myDHResourceGroup"
$location = "EastUS"

New-AzResourceGroup -Location $location -Name $rgName
$hostGroup = New-AzHostGroup `
   -Location $location `
   -Name myHostGroup `
   -PlatformFaultDomain 2 `
   -ResourceGroupName $rgName `
   -Zone 1
```


Добавьте `-SupportAutomaticPlacement true` параметр, чтобы виртуальные машины и экземпляры масштабируемых наборов автоматически размещались на узлах в группе узлов. Дополнительные сведения см. в разделе [Ручное и автоматическое размещение ](../dedicated-hosts.md#manual-vs-automatic-placement).


## <a name="create-a-host"></a>Создание узла

Теперь создадим выделенный узел в группе узлов. Помимо имени узла, необходимо указать его номер SKU. Номер SKU узла определяет поддерживаемую серию виртуальных машин, а также поколение оборудования для выделенного узла.

Дополнительные сведения о номерах SKU и ценах узлов см. на странице [цен на выделенные узлы Azure](https://aka.ms/ADHPricing).

Если для группы узлов задано число доменов сбоя, вам будет предложено указать домен сбоя для узла. В этом примере мы устанавливаем домен сбоя для узла в значение 1.


```azurepowershell-interactive
$dHost = New-AzHost `
   -HostGroupName $hostGroup.Name `
   -Location $location -Name myHost `
   -ResourceGroupName $rgName `
   -Sku DSv3-Type1 `
   -AutoReplaceOnFailure 1 `
   -PlatformFaultDomain 1
```

## <a name="create-a-vm"></a>Создание виртуальной машины

Создайте виртуальную машину на выделенном узле. 

Если при создании группы узлов была указана зона доступности, то при создании виртуальной машины необходимо использовать ту же зону. В этом примере, так как наша группа узлов находится в зоне 1, нам нужно создать виртуальную машину в зоне 1.  


```azurepowershell-interactive
$cred = Get-Credential
New-AzVM `
   -Credential $cred `
   -ResourceGroupName $rgName `
   -Location $location `
   -Name myVM `
   -HostId $dhost.Id `
   -Image Win2016Datacenter `
   -Zone 1 `
   -Size Standard_D4s_v3
```

> [!WARNING]
> Если виртуальная машина создается в узле, в котором недостаточно ресурсов, она будет создана в состоянии сбоя. 

## <a name="check-the-status-of-the-host"></a>Проверка состояния узла

Вы можете проверить состояние работоспособности узла и количество виртуальных машин, которые вы по-прежнему можете развернуть на узле с помощью [жетазост](/powershell/module/az.compute/get-azhost) с `-InstanceView` параметром.

```azurepowershell-interactive
Get-AzHost `
   -ResourceGroupName $rgName `
   -Name myHost `
   -HostGroupName $hostGroup.Name `
   -InstanceView
```

Выходные данные будут выглядеть следующим образом.

```
ResourceGroupName      : myDHResourceGroup
PlatformFaultDomain    : 1
AutoReplaceOnFailure   : True
HostId                 : 12345678-1234-1234-abcd-abc123456789
ProvisioningTime       : 7/28/2019 5:31:01 PM
ProvisioningState      : Succeeded
InstanceView           : 
  AssetId              : abc45678-abcd-1234-abcd-123456789abc
  AvailableCapacity    : 
    AllocatableVMs[0]  : 
      VmSize           : Standard_D2s_v3
      Count            : 32
    AllocatableVMs[1]  : 
      VmSize           : Standard_D4s_v3
      Count            : 16
    AllocatableVMs[2]  : 
      VmSize           : Standard_D8s_v3
      Count            : 8
    AllocatableVMs[3]  : 
      VmSize           : Standard_D16s_v3
      Count            : 4
    AllocatableVMs[4]  : 
      VmSize           : Standard_D32-8s_v3
      Count            : 2
    AllocatableVMs[5]  : 
      VmSize           : Standard_D32-16s_v3
      Count            : 2
    AllocatableVMs[6]  : 
      VmSize           : Standard_D32s_v3
      Count            : 2
    AllocatableVMs[7]  : 
      VmSize           : Standard_D64-16s_v3
      Count            : 1
    AllocatableVMs[8]  : 
      VmSize           : Standard_D64-32s_v3
      Count            : 1
    AllocatableVMs[9]  : 
      VmSize           : Standard_D64s_v3
      Count            : 1
  Statuses[0]          : 
    Code               : ProvisioningState/succeeded
    Level              : Info
    DisplayStatus      : Provisioning succeeded
    Time               : 7/28/2019 5:31:01 PM
  Statuses[1]          : 
    Code               : HealthState/available
    Level              : Info
    DisplayStatus      : Host available
Sku                    : 
  Name                 : DSv3-Type1
Id                     : /subscriptions/10101010-1010-1010-1010-101010101010/re
sourceGroups/myDHResourceGroup/providers/Microsoft.Compute/hostGroups/myHostGroup/hosts
/myHost
Name                   : myHost
Location               : eastus
Tags                   : {}
```

## <a name="create-a-scale-set"></a>Создание масштабируемого набора 

При развертывании масштабируемого набора указывается группа узлов.

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -VMScaleSetName "myDHScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic"`
  -HostGroupId $hostGroup.Id
```

Если вы хотите вручную выбрать узел для развертывания масштабируемого набора, добавьте `--host` и имя узла.



## <a name="add-an-existing-vm"></a>Добавление существующей виртуальной машины 

Существующую виртуальную машину можно добавить на выделенный узел, но сначала необходимо Стоп\деаллокатед.. Перед перемещением виртуальной машины на выделенный узел убедитесь, что конфигурация виртуальной машины поддерживается:

- Размер виртуальной машины должен быть в том же семействе размеров, что и выделенный узел. Например, если выделенный узел является DSv3, размер виртуальной машины может быть Standard_D4s_v3, но не Standard_A4_v2. 
- Виртуальная машина должна находиться в том же регионе, что и выделенный узел.
- Виртуальная машина не может входить в группу размещения с учетом расположения. Удалите виртуальную машину из группы размещения с учетом расположения, прежде чем перемещать ее на выделенный узел. Дополнительные сведения см. в разделе [перемещение виртуальной машины из группы размещения](./proximity-placement-groups.md#move-an-existing-vm-out-of-a-proximity-placement-group) с учетом расположения.
- Виртуальная машина не может находиться в группе доступности.
- Если виртуальная машина находится в зоне доступности, она должна быть той же зоной доступности, что и группа узлов. Параметры зоны доступности для виртуальной машины и группы узлов должны совпадать.

Замените значения переменных собственными сведениями.

```azurepowershell-interactive
$vmRGName = "movetohost"
$vmName = "myVMtoHost"
$dhRGName = "myDHResourceGroup"
$dhGroupName = "myHostGroup"
$dhName = "myHost"

$myDH = Get-AzHost `
   -HostGroupName $dhGroupName `
   -ResourceGroupName $dhRGName `
   -Name $dhName
   
$myVM = Get-AzVM `
   -ResourceGroupName $vmRGName `
   -Name $vmName
   
$myVM.Host = New-Object Microsoft.Azure.Management.Compute.Models.SubResource

$myVM.Host.Id = "$myDH.Id"

Stop-AzVM `
   -ResourceGroupName $vmRGName `
   -Name $vmName -Force
   
Update-AzVM `
   -ResourceGroupName $vmRGName `
   -VM $myVM -Debug
   
Start-AzVM `
   -ResourceGroupName $vmRGName `
   -Name $vmName
```


## <a name="clean-up"></a>Очистка

Вы платите за выделенные узлы, даже если виртуальные машины не развернуты. Для экономии затрат следует удалить все узлы, которые в настоящее время не используются.  

Узел можно удалить только в том случае, если он больше не используется виртуальными машинами. Удалите виртуальные машины с помощью [Remove-AzVM](/powershell/module/az.compute/remove-azvm).

```azurepowershell-interactive
Remove-AzVM -ResourceGroupName $rgName -Name myVM
```

После удаления виртуальных машин можно удалить узел с помощью команды [Remove-азост](/powershell/module/az.compute/remove-azhost).

```azurepowershell-interactive
Remove-AzHost -ResourceGroupName $rgName -Name myHost
```

После удаления всех узлов группу узлов можно удалить с помощью команды [Remove-азостграуп](/powershell/module/az.compute/remove-azhostgroup). 

```azurepowershell-interactive
Remove-AzHost -ResourceGroupName $rgName -Name myHost
```

Вы также можете удалить всю группу ресурсов в одной команде, используя [Remove-азресаурцеграуп](/powershell/module/az.resources/remove-azresourcegroup). Это приведет к удалению всех ресурсов, созданных в группе, включая все виртуальные машины, узлы и группы узлов.
 
```azurepowershell-interactive
Remove-AzResourceGroup -Name $rgName
```


## <a name="next-steps"></a>Дальнейшие действия

- [Здесь](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md) можно найти пример шаблона, в котором используются как зоны, так и домены сбоя для обеспечения максимальной устойчивости в регионе.

- Кроме того, выделенные узлы можно развернуть с помощью [портал Azure](../dedicated-hosts-portal.md).
