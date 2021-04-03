---
title: Руководство по Создание масштабируемого набора виртуальных машин Windows
description: Узнайте, как с помощью Azure PowerShell создать и развернуть высокодоступное приложение на виртуальных машинах Windows с помощью масштабируемого набора виртуальных машин
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: windows
ms.date: 11/30/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: b3853ddc71d1a9be26b2492764a9b341446e0eeb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "89078747"
---
# <a name="tutorial-create-a-virtual-machine-scale-set-and-deploy-a-highly-available-app-on-windows-with-azure-powershell"></a>Руководство по Создание масштабируемого набора виртуальных машин и развертывание в Windows приложения высокого уровня доступности с помощью Azure PowerShell
Масштабируемый набор виртуальных машин позволяет развернуть набор одинаковых виртуальных машин с возможностью автомасштабирования и управлять этим набором. Количество виртуальных машин в масштабируемом наборе можно изменять вручную. Также можно задавать правила автомасштабирования на основе использования ресурсов, например ЦП, памяти или сетевого трафика. В этом руководстве вы развернете масштабируемый набор виртуальных машин в Azure и научитесь выполнять следующие операции:

> [!div class="checklist"]
> * Использование расширения пользовательских скриптов для определения масштабируемого сайта IIS
> * Создание балансировщика нагрузки для масштабируемого набора
> * создавать масштабируемый набор виртуальных машин;
> * увеличивать или уменьшать количество экземпляров в масштабируемом наборе;
> * Создание правил автомасштабирования

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.

## <a name="scale-set-overview"></a>Обзор масштабируемого набора
Масштабируемый набор виртуальных машин позволяет развернуть набор одинаковых виртуальных машин с возможностью автомасштабирования и управлять этим набором. Виртуальные машины распределяются по логическим доменам сбоя и обновления в одной или нескольких *группах размещения*. Группы размещения — это группы одинаковым образом настроенных виртуальных машин, аналогичные [группам доступности](tutorial-availability-sets.md).

Виртуальные машины создаются в масштабируемом наборе по мере необходимости. Можно определить правила автомасштабирования, чтобы управлять добавлением и удалением виртуальных машин в масштабируемом наборе. Эти правила могут активироваться на основе метрик, таких как использование ЦП, использование памяти или сетевой трафик.

При использовании образа платформы Azure масштабируемые наборы поддерживают до 1000 виртуальных машин. Для рабочих нагрузок, требующих установки значительного числа компонентов или сложной настройки виртуальных машин, вы можете [создать образ виртуальной машины](tutorial-custom-images.md). При использовании пользовательского образа в масштабируемом наборе можно создать до 600 виртуальных машин.


## <a name="create-a-scale-set"></a>Создание масштабируемого набора
Создайте масштабируемый набор виртуальных машин с помощью командлета [New-AzVmss](/powershell/module/az.compute/new-azvmss). В следующем примере создается масштабируемый набор с именем *myScaleSet*, использующий образ платформы *Windows Server 2016 Datacenter*. Сетевые ресурсы Azure для виртуальной сети, общедоступный IP-адрес и подсистема балансировки нагрузки создаются автоматически. При появлении запроса можно задать собственные административные учетные данные для экземпляров виртуальных машин в масштабируемом наборе:

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic"
```

Создание и настройка всех ресурсов и виртуальных машин масштабируемого набора занимает несколько минут.


## <a name="deploy-sample-application"></a>Разверните пример приложения
Для проверки масштабируемого набора установите базовое веб-приложение. Расширение пользовательского скрипта Azure используется для скачивания и запуска скрипта, который устанавливает IIS на экземплярах виртуальных машин. Это расширение можно использовать для настройки после развертывания, установки программного обеспечения и других задач настройки или управления. Дополнительные сведения см. в статье [Расширение Custom Script в ОС Windows](../extensions/custom-script-windows.md).

Использование расширения пользовательских скриптов для установки базового веб-сервера IIS Примените расширение пользовательского скрипта, которое устанавливает IIS, как показано ниже:

```azurepowershell-interactive
# Define the script for your Custom Script Extension to run
$publicSettings = @{
    "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis.ps1");
    "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis.ps1"
}

# Get information about the scale set
$vmss = Get-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet"

# Use Custom Script Extension to install IIS and configure basic website
Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $publicSettings

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

## <a name="allow-traffic-to-application"></a>Разрешение передачи трафика в приложение

Чтобы разрешить доступ к базовому веб-приложению, создайте сетевую группу безопасности с помощью командлетов [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) и [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). Дополнительные сведения см. в статье [Сеть для масштабируемых наборов виртуальных машин Azure](../../virtual-machine-scale-sets/virtual-machine-scale-sets-networking.md).

```azurepowershell-interactive
# Get information about the scale set
$vmss = Get-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet"

#Create a rule to allow traffic over port 80
$nsgFrontendRule = New-AzNetworkSecurityRuleConfig `
  -Name myFrontendNSGRule `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 200 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access Allow

#Create a network security group and associate it with the rule
$nsgFrontend = New-AzNetworkSecurityGroup `
  -ResourceGroupName  "myResourceGroupScaleSet" `
  -Location EastUS `
  -Name myFrontendNSG `
  -SecurityRules $nsgFrontendRule

$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName  "myResourceGroupScaleSet" `
  -Name myVnet

$frontendSubnet = $vnet.Subnets[0]

$frontendSubnetConfig = Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name mySubnet `
  -AddressPrefix $frontendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgFrontend

Set-AzVirtualNetwork -VirtualNetwork $vnet

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

## <a name="test-your-scale-set"></a>Проверка масштабируемого набора
Чтобы проверить работу масштабируемого набора, получите общедоступный IP-адрес своей подсистемы балансировки нагрузки с помощью командлета [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress). Следующий пример отображает IP-адрес элемента *myPublicIP*, созданного ранее вместе с масштабируемым набором:

```azurepowershell-interactive
Get-AzPublicIPAddress `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Name "myPublicIPAddress" | select IpAddress
```

Введите общедоступный IP-адрес в веб-браузер. Отобразится веб-приложение, а также имя узла виртуальной машины, на которую подсистема балансировки нагрузки направила трафик.

![Выполнение сайта IIS](./media/tutorial-create-vmss/running-iis-site.png)

Чтобы познакомиться с работой масштабируемого набора, принудительно обновите окно веб-браузера, чтобы увидеть, как балансировщик нагрузки распределяет трафик между всеми виртуальными машинами, где выполняется приложение.


## <a name="management-tasks"></a>Задачи управления
На протяжении жизненного цикла масштабируемого набора может возникнуть необходимость выполнить одну или несколько задач управления. Кроме того, можно создавать сценарии для автоматизации различных задач жизненного цикла. Azure PowerShell позволяет быстро выполнять эти задачи. Ниже приведено несколько распространенных задач.

### <a name="view-vms-in-a-scale-set"></a>Просмотр виртуальных машин в масштабируемом наборе
Чтобы просмотреть список экземпляров виртуальных машин в масштабируемом наборе, выполните командлет [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm), как показано ниже.

```azurepowershell-interactive
Get-AzVmssVM `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet"
```

В следующем примере выходных данных показано два экземпляра виртуальной машины в масштабируемом наборе.

```powershell
ResourceGroupName                 Name Location             Sku InstanceID ProvisioningState
-----------------                 ---- --------             --- ---------- -----------------
MYRESOURCEGROUPSCALESET   myScaleSet_0   eastus Standard_DS1_v2          0         Succeeded
MYRESOURCEGROUPSCALESET   myScaleSet_1   eastus Standard_DS1_v2          1         Succeeded
```

Чтобы просмотреть дополнительные сведения о конкретном экземпляре виртуальной машины, добавьте параметр `-InstanceId` в командлет [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm). Следующий пример возвращает сведения об экземпляре виртуальной машины *1*.

```azurepowershell-interactive
Get-AzVmssVM `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet" `
  -InstanceId "1"
```


### <a name="increase-or-decrease-vm-instances"></a>Увеличение или уменьшение числа экземпляров виртуальной машины
Чтобы просмотреть количество экземпляров, присутствующих в масштабируемом наборе, выполните командлет [Get-AzVmss](/powershell/module/az.compute/get-azvmss) и отправьте запрос для *sku.capacity*.

```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet" | `
  Select -ExpandProperty Sku
```

После этого можно вручную увеличить или уменьшить число виртуальных машин в масштабируемом наборе, выполнив командлет [Update-AzVmss](/powershell/module/az.compute/update-azvmss). В следующем примере число виртуальных машин в масштабируемом наборе определяется равным *3*:

```azurepowershell-interactive
# Get current scale set
$scaleset = Get-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet"

# Set and update the capacity of your scale set
$scaleset.sku.capacity = 3
Update-AzVmss -ResourceGroupName "myResourceGroupScaleSet" `
    -Name "myScaleSet" `
    -VirtualMachineScaleSet $scaleset
```

Обновление числа экземпляров в масштабируемом наборе занимает несколько минут.


### <a name="configure-autoscale-rules"></a>Настройка правил автомасштабирования
Вместо установки числа экземпляров в масштабируемом наборе вручную можно определить правила автомасштабирования. С помощью правил среда отслеживает экземпляры в масштабируемом наборе и реагирует соответствующим образом на основе определенных метрик и пороговых значений. Приведенный ниже пример увеличивает число экземпляров на один, когда средняя загрузка ЦП превышает 60 % в течение 5 минут. Если средняя загрузка ЦП не превышает 30 % в течение 5 минут, число экземпляров уменьшается на один.

```azurepowershell-interactive
# Define your scale set information
$mySubscriptionId = (Get-AzSubscription)[0].Id
$myResourceGroup = "myResourceGroupScaleSet"
$myScaleSet = "myScaleSet"
$myLocation = "East US"
$myScaleSetId = (Get-AzVmss -ResourceGroupName $myResourceGroup -VMScaleSetName $myScaleSet).Id 

# Create a scale up rule to increase the number instances after 60% average CPU usage exceeded for a 5-minute period
$myRuleScaleUp = New-AzAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId $myScaleSetId `
  -Operator GreaterThan `
  -MetricStatistic Average `
  -Threshold 60 `
  -TimeGrain 00:01:00 `
  -TimeWindow 00:05:00 `
  -ScaleActionCooldown 00:05:00 `
  -ScaleActionDirection Increase `
  -ScaleActionValue 1

# Create a scale down rule to decrease the number of instances after 30% average CPU usage over a 5-minute period
$myRuleScaleDown = New-AzAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId $myScaleSetId `
  -Operator LessThan `
  -MetricStatistic Average `
  -Threshold 30 `
  -TimeGrain 00:01:00 `
  -TimeWindow 00:05:00 `
  -ScaleActionCooldown 00:05:00 `
  -ScaleActionDirection Decrease `
  -ScaleActionValue 1

# Create a scale profile with your scale up and scale down rules
$myScaleProfile = New-AzAutoscaleProfile `
  -DefaultCapacity 2  `
  -MaximumCapacity 10 `
  -MinimumCapacity 2 `
  -Rule $myRuleScaleUp,$myRuleScaleDown `
  -Name "autoprofile"

# Apply the autoscale rules
Add-AzAutoscaleSetting `
  -Location $myLocation `
  -Name "autosetting" `
  -ResourceGroup $myResourceGroup `
  -TargetResourceId $myScaleSetId `
  -AutoscaleProfile $myScaleProfile
```

Дополнительные сведения см. в [рекомендациях по автомасштабированию](/azure/architecture/best-practices/auto-scaling).


## <a name="next-steps"></a>Дальнейшие действия
В рамках этого руководства вы создали набор масштабирования виртуальных машин. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Использование расширения пользовательских скриптов для определения масштабируемого сайта IIS
> * Создание балансировщика нагрузки для масштабируемого набора
> * создавать масштабируемый набор виртуальных машин;
> * увеличивать или уменьшать количество экземпляров в масштабируемом наборе;
> * Создание правил автомасштабирования

Перейдите к следующему руководству, чтобы узнать о принципах балансировки нагрузки для виртуальных машин.

> [!div class="nextstepaction"]
> [Балансировка нагрузки между виртуальными машинами](tutorial-load-balancer.md)
