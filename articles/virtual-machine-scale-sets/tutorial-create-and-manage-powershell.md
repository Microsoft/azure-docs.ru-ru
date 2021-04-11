---
title: Руководство по созданию масштабируемого набора виртуальных машин Azure и управлению им с помощью Azure PowerShell
description: Сведения об использовании Azure PowerShell для создания масштабируемого набора виртуальных машин и сведения о некоторых стандартных задачах управления, включая запуск и остановку экземпляра или изменение емкости масштабируемого набора.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 05/18/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: 102ab619a3f341658383dc4997f9fbfc2cd12a43
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102519186"
---
# <a name="tutorial-create-and-manage-a-virtual-machine-scale-set-with-azure-powershell"></a>Руководство по Создание масштабируемого набора виртуальных машин с помощью Azure PowerShell

Масштабируемый набор виртуальных машин обеспечивает развертывание и администрирование набора идентичных автомасштабируемых виртуальных машин. На протяжении жизненного цикла масштабируемого набора виртуальных машин может возникнуть необходимость выполнить одну или несколько задач управления. Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * Создание масштабируемого набора виртуальных машин и подключение к нему.
> * Выбор и использование образов виртуальных машин
> * Просмотр и использование определенных размеров экземпляров виртуальных машин.
> * Масштабирование масштабируемого набора вручную.
> * Выполнение стандартных задач управления масштабируемым набором.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]



## <a name="create-a-resource-group"></a>Создание группы ресурсов
Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. Группу ресурсов следует создать до создания масштабируемого набора виртуальных машин. Создайте группу ресурсов с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). В примере создается группа ресурсов под названием *myResourceGroup* в регионе *EastUS*. 

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName "myResourceGroup" -Location "EastUS"
```
Имя группы ресурсов указывается при создании или изменении масштабируемого набора в этом руководстве.


## <a name="create-a-scale-set"></a>Создание масштабируемого набора
Первым делом настройте имя и пароль администратора для экземпляров виртуальных машин с помощью командлета [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential):

```azurepowershell-interactive
$cred = Get-Credential
```

Теперь создайте масштабируемый набор виртуальных машин с помощью командлета [New-AzVmss](/powershell/module/az.compute/new-azvmss). Чтобы распределить трафик между отдельными экземплярами виртуальных машин, создается еще и подсистема балансировки нагрузки. Подсистема балансировки нагрузки определяет правила передачи трафика на TCP-порт 80, а также разрешает подключение удаленного рабочего стола трафик через TCP-порт 3389 и удаленное взаимодействие PowerShell через TCP-порт 5985:

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet" `
  -Location "EastUS" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -Credential $cred
```

Создание и настройка всех ресурсов и экземпляров виртуальных машин масштабируемого набора занимает несколько минут.

> [!IMPORTANT]
> Если не удается подключиться к масштабируемому набору, может потребоваться создать группу безопасности сети, добавив параметр *[-SecurityGroupName "mySecurityGroup"](/powershell/module/az.compute/new-azvmss)* .


## <a name="view-the-vm-instances-in-a-scale-set"></a>Просмотр экземпляров виртуальных машин в масштабируемом наборе
Чтобы просмотреть список экземпляров виртуальных машин в масштабируемом наборе, выполните командлет [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm), как показано ниже.

```azurepowershell-interactive
Get-AzVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

В следующем примере выходных данных показано два экземпляра виртуальной машины в масштабируемом наборе.

```powershell
ResourceGroupName         Name Location             Sku InstanceID ProvisioningState
-----------------         ---- --------             --- ---------- -----------------
MYRESOURCEGROUP   myScaleSet_0   eastus Standard_DS1_v2          0         Succeeded
MYRESOURCEGROUP   myScaleSet_1   eastus Standard_DS1_v2          1         Succeeded
```

Чтобы просмотреть дополнительные сведения о конкретном экземпляре виртуальной машины, добавьте параметр `-InstanceId` в командлет [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm). Следующий пример возвращает сведения об экземпляре виртуальной машины *1*.

```azurepowershell-interactive
Get-AzVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```


## <a name="list-connection-information"></a>Вывод списка со сведениями о подключении
Общедоступный IP-адрес назначен подсистеме балансировки нагрузки, перенаправляющей трафик к отдельным экземплярам виртуальных машин. По умолчанию правила преобразования сетевых адресов (NAT) добавляются в подсистему балансировки нагрузки Azure для переадресации трафика удаленного подключения на каждую виртуальную машину на определенном порте. Чтобы подключиться к экземплярам виртуальных машин в масштабируемом наборе, необходимо создать удаленное подключение по назначенному общедоступному IP адресу и номеру порта.

Чтобы получить список портов NAT для подключения к экземплярам виртуальных машин в масштабируемом наборе, сначала получите объект подсистемы балансировки нагрузки с помощью командлета [Get-AzLoadBalancer](/powershell/module/az.network/Get-AzLoadBalancer). Затем просмотрите правила NAT для входящего трафика с помощью командлета [Get-AzLoadBalancerInboundNatRuleConfig](/powershell/module/az.network/Get-AzLoadBalancerInboundNatRuleConfig).


```azurepowershell-interactive
# Get the load balancer object
$lb = Get-AzLoadBalancer -ResourceGroupName "myResourceGroup" -Name "myLoadBalancer"

# View the list of inbound NAT rules
Get-AzLoadBalancerInboundNatRuleConfig -LoadBalancer $lb | Select-Object Name,Protocol,FrontEndPort,BackEndPort
```

В следующем примере выходных данных показано имя экземпляра, общедоступный IP-адрес подсистемы балансировки нагрузки и номер порта, в который правила преобразования сетевых адресов перенаправляют трафик.

```powershell
Name             Protocol FrontendPort BackendPort
----             -------- ------------ -----------
myScaleSet3389.0 Tcp             50001        3389
myScaleSet5985.0 Tcp             51001        5985
myScaleSet3389.1 Tcp             50002        3389
myScaleSet5985.1 Tcp             51002        5985
```

Как показано в результатах выполнения предыдущего командлета [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm), *имя* правила выравнивается по имени экземпляра виртуальной машины. Например, чтобы подключиться к экземпляру виртуальной машины *0*, используйте *myScaleSet3389.0* и подключитесь к порту *50001*. Для подключения к экземпляру виртуальной машины *1* используйте значение из *myScaleSet3389.1* и подключитесь к порту *50002*. Для осуществления удаленного подключения с помощью PowerShell подключитесь к правилу соответствующего экземпляра виртуальной машины по *TCP*-порту *5985*.

Получите сведения об общедоступном IP-адресе подсистемы балансировки нагрузки с помощью командлета [Get-AzPublicIpAddress](/powershell/module/az.network/Get-AzPublicIpAddress).


```azurepowershell-interactive
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" -Name "myPublicIPAddress" | Select IpAddress
```

Выходные данные примера:

```powershell
IpAddress
---------
52.168.121.216
```

Создайте удаленное подключение к первому экземпляру виртуальной машины. Укажите общедоступный IP-адрес и номер порта требуемого экземпляра виртуальной машины, как показано в предыдущих командах. При появлении запроса введите учетные данные, используемые для создания масштабируемого набора (по умолчанию в используемых командах этими значениями являются *azureuser* и *P\@ssw0rd!* ). Если вы используете Azure Cloud Shell, выполните этот шаг с помощью локальной командной строки PowerShell или клиента удаленного рабочего стола. В следующем примере устанавливается подключение к экземпляру виртуальной машины *1*.

```powershell
mstsc /v 52.168.121.216:50001
```

После входа в экземпляр виртуальной машины вы можете выполнить некоторые изменения конфигурации вручную. Теперь отключите удаленное подключение.


## <a name="understand-vm-instance-images"></a>Описание образов экземпляра виртуальной машины
Azure Marketplace содержит множество образов, которые можно использовать для создания экземпляров виртуальных машин. Для просмотра списка доступных издателей используйте командлет [Get-AzVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher).

```azurepowershell-interactive
Get-AzVMImagePublisher -Location "EastUS"
```

Для просмотра списка образов, доступных для данного издателя, воспользуйтесь командлетом [Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku). Кроме того, список образов можно отфильтровать по издателю или предложению с помощью аргумента `-PublisherName` или `-Offer` соответственно. В следующем примере список всех образов фильтруется по имени издателя *MicrosoftWindowsServer* и предложению *WindowsServer*.

```azurepowershell-interactive
Get-AzVMImageSku -Location "EastUS" -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer"
```

В следующем примере выходных данных показаны все доступные образы Windows Server.

```powershell
Skus                                  Offer         PublisherName          Location
----                                  -----         -------------          --------
2008-R2-SP1                           WindowsServer MicrosoftWindowsServer eastus
2008-R2-SP1-smalldisk                 WindowsServer MicrosoftWindowsServer eastus
2012-Datacenter                       WindowsServer MicrosoftWindowsServer eastus
2012-Datacenter-smalldisk             WindowsServer MicrosoftWindowsServer eastus
2012-R2-Datacenter                    WindowsServer MicrosoftWindowsServer eastus
2012-R2-Datacenter-smalldisk          WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter                       WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-Server-Core           WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-Server-Core-smalldisk WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-smalldisk             WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-with-Containers       WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-with-RDSH             WindowsServer MicrosoftWindowsServer eastus
2016-Nano-Server                      WindowsServer MicrosoftWindowsServer eastus
```

При создании масштабируемого набора в начале руководства для экземпляров виртуальных машин использовался предлагаемый по умолчанию образ виртуальной машины *Windows Server 2016 DataCenter*. Вы можете указать другой образ виртуальной машины на основе выходных данных командлета [Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku). В следующем примере создается масштабируемый набор с использованием параметра `-ImageName` для указания образа виртуальной машины *MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest*. Так как создание и настройка всех ресурсов и экземпляров виртуальных машин масштабируемого набора занимает несколько минут, вам не нужно развертывать следующий масштабируемый набор.

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup2" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet2" `
  -VirtualNetworkName "myVnet2" `
  -SubnetName "mySubnet2" `
  -PublicIpAddressName "myPublicIPAddress2" `
  -LoadBalancerName "myLoadBalancer2" `
  -UpgradePolicyMode "Automatic" `
  -ImageName "MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest" `
  -Credential $cred
```

> [!IMPORTANT]
> Мы советуем использовать образ *последней* версии. Укажите "latest", чтобы использовать последнюю версию образа, доступную во время развертывания. Обратите внимание: несмотря на то, что указано "latest", образ виртуальной машины не будет автоматически обновляться после развертывания, даже если станет доступной его новая версия.

## <a name="understand-vm-instance-sizes"></a>Описание размеров экземпляра виртуальной машины
Размер экземпляра виртуальной машины (или *номер SKU*) определяет количество выделяемых ему вычислительных ресурсов, таких как ЦП, GPU и память. Размеры экземпляров виртуальных машин в масштабируемом наборе должны соответствовать ожидаемой рабочей нагрузке.

### <a name="vm-instance-sizes"></a>Размеры экземпляров виртуальных машин
В приведенной ниже таблице указаны категории распространенных размеров виртуальной машины и варианты использования.

| Тип                     | Распространенные размеры           |    Описание       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [Универсальные](../virtual-machines/sizes-general.md)         |Dsv3, Dv3, DSv2, Dv2, DS, D, Av2, A0–7| Сбалансированное соотношение ресурсов ЦП и памяти. Идеально подходят для разработки и тестирования малых и средних приложений и решений для обработки данных.  |
| [Оптимизированные для вычислений](../virtual-machines/sizes-compute.md)   | Fs, F             | Высокое соотношение ресурсов ЦП и памяти. Подходят для приложений со средним объемом трафика, сетевых устройств и пакетных процессов.        |
| [Оптимизированные для памяти](../virtual-machines/sizes-memory.md)    | Esv3, Ev3, M, GS, G, DSv2, DS, Dv2, D   | Высокое соотношение ресурсов памяти и числа ядер. Отлично подходят для реляционных баз данных, кэша среднего и большого объема, а также выполняющейся в памяти аналитики.                 |
| [Оптимизированные для хранилища](../virtual-machines/sizes-storage.md)      | Ls                | Высокая пропускная способность дисков и количество операций ввода-вывода. Идеальный вариант для работы с большими данными, а также с базами данных SQL и NoSQL.                                                         |
| [GPU](../virtual-machines/sizes-gpu.md)          | NV, NC            | Специализированные виртуальные машины, предназначенные для ресурсоемкой отрисовки изображений и редактирования видео.       |
| [Высокопроизводительные](../virtual-machines/sizes-hpc.md) | H, A8–A11          | Виртуальные машины с самыми мощными ЦП, для которых можно настроить сетевые интерфейсы с высокой пропускной способностью (RDMA). 

### <a name="find-available-vm-instance-sizes"></a>Поиск доступных размеров экземпляров виртуальной машины
Чтобы просмотреть список доступных размеров экземпляров виртуальных машин в определенном регионе, используйте команду [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize). 

```azurepowershell-interactive
Get-AzVMSize -Location "EastUS"
```

Вывод команды будет примерно таким, как в следующем сокращенном примере, в котором показаны ресурсы, назначенные каждому размеру виртуальной машины:

```powershell
Name                   NumberOfCores MemoryInMB MaxDataDiskCount OSDiskSizeInMB ResourceDiskSizeInMB
----                   ------------- ---------- ---------------- -------------- --------------------
Standard_DS1_v2                    1       3584                4        1047552                 7168
Standard_DS2_v2                    2       7168                8        1047552                14336
[...]
Standard_A0                        1        768                1        1047552                20480
Standard_A1                        1       1792                2        1047552                71680
[...]
Standard_F1                        1       2048                4        1047552                16384
Standard_F2                        2       4096                8        1047552                32768
[...]
Standard_NV6                       6      57344               24        1047552               389120
Standard_NV12                     12     114688               48        1047552               696320
```

При создании масштабируемого набора в начале этого руководства для экземпляров виртуальной машины был указан стандартный номер SKU виртуальной машины *Standard_DS1_v2*. Вы можете указать другой размер экземпляра виртуальной машины на основе выходных данных командлета [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize). В указанном ниже примере будет создан масштабируемый набор с использованием параметра `-VmSize`, который позволяет указать размер экземпляра виртуальной машины *Standard_F1*. Так как создание и настройка всех ресурсов и экземпляров виртуальных машин масштабируемого набора занимает несколько минут, вам не нужно развертывать следующий масштабируемый набор.

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup3" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet3" `
  -VirtualNetworkName "myVnet3" `
  -SubnetName "mySubnet3" `
  -PublicIpAddressName "myPublicIPAddress3" `
  -LoadBalancerName "myLoadBalancer3" `
  -UpgradePolicyMode "Automatic" `
  -VmSize "Standard_F1" `
  -Credential $cred
```


## <a name="change-the-capacity-of-a-scale-set"></a>Изменение емкости масштабируемого набора
При создании масштабируемого набора вы запросили два экземпляра виртуальных машин. Чтобы увеличить или уменьшить число экземпляров виртуальных машин в масштабируемом наборе, можно изменить его емкость вручную. Масштабируемый набор создает или удаляет необходимое количество экземпляров виртуальной машины, а затем настраивает подсистему балансировки нагрузки для распределения трафика.

Сначала создайте объект масштабируемого набора с помощью командлета [Get-AzVmss](/powershell/module/az.compute/get-azvmss), затем укажите новое значение для `sku.capacity`. С помощью командлета [Update-AzVmss](/powershell/module/az.compute/update-azvmss) примените изменение емкости. В следующем примере число экземпляров виртуальных машин в масштабируемом наборе определяется равным *3*:

```azurepowershell-interactive
# Get current scale set
$vmss = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"

# Set and update the capacity of your scale set
$vmss.sku.capacity = 3
Update-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet" -VirtualMachineScaleSet $vmss 
```

На обновление емкости масштабируемого набора требуется несколько минут. Чтобы просмотреть количество экземпляров, присутствующих в масштабируемом наборе, выполните командлет [Get-AzVmss](/powershell/module/az.compute/get-azvmss).

```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

В следующем примере выходных данных показано, что емкость масштабируемого набора равна *3*.

```powershell
Sku        :
  Name     : Standard_DS2
  Tier     : Standard
  Capacity : 3
```


## <a name="common-management-tasks"></a>Общие задачи управления
Теперь вы можете создать масштабируемый набор, вывести сведения о подключениях и подключиться к экземплярам виртуальных машин. Вы узнали, как использовать разные образы операционной системы для экземпляров виртуальных машин, выбирать разные размеры виртуальной машины и изменять количество экземпляров вручную. В рамках повседневных задач по управлению вам может понадобиться остановить, запустить или перезапустить экземпляры виртуальных машин в масштабируемом наборе.

### <a name="stop-and-deallocate-vm-instances-in-a-scale-set"></a>Остановка и освобождение экземпляров виртуальных машин в масштабируемом наборе
С помощью командлета [Stop-AzVmss](/powershell/module/az.compute/stop-azvmss) остановите одну или несколько виртуальных машин в масштабируемом наборе. С помощью параметра `-InstanceId` можно указать одну или несколько виртуальных машин, которые нужно остановить. Если не указать идентификатор экземпляра, останавливаются все виртуальные машины в масштабируемом наборе. Следующий пример останавливает экземпляр *1*.

```azurepowershell-interactive
Stop-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```

По умолчанию остановленные виртуальные машины освобождаются, и за них не взимается плата за вычислительные операции. Если требуется, чтобы остановленная виртуальная машина осталась в подготовленном состоянии, добавьте параметр `-StayProvisioned` в предыдущую команду. За остановленные виртуальные машины, которые остаются в подготовленном состоянии, взимается плата за вычислительные операции по стандартной ставке.

### <a name="start-vm-instances-in-a-scale-set"></a>Запуск экземпляров виртуальных машин в масштабируемом наборе
С помощью командлета [Start-AzVmss](/powershell/module/az.compute/start-azvmss) запустите одну или несколько виртуальных машин в масштабируемом наборе. С помощью параметра `-InstanceId` можно указать одну или несколько виртуальных машин, которые нужно запустить. Если не указать идентификатор экземпляра, запускаются все виртуальные машины в масштабируемом наборе. Следующий пример запускает экземпляр *1*.

```azurepowershell-interactive
Start-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```

### <a name="restart-vm-instances-in-a-scale-set"></a>Перезапуск экземпляров виртуальных машин в масштабируемом наборе
Чтобы перезапустить одну или несколько виртуальных машин в масштабируемом наборе, используйте командлет [Retart-AzVmss](/powershell/module/az.compute/restart-azvmss). С помощью параметра `-InstanceId` можно указать одну или несколько виртуальных машин, которые нужно перезапустить. Если не указать идентификатор экземпляра, перезапускаются все виртуальные машины в масштабируемом наборе. Следующий пример перезапускает экземпляр *1*.

```azurepowershell-interactive
Restart-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```


## <a name="clean-up-resources"></a>Очистка ресурсов
При удалении группы ресурсов будут также удалены все содержащиеся в ней ресурсы: экземпляры виртуальной машины, виртуальная сеть и диски. Параметр `-Force` подтверждает, что вы хотите удалить ресурсы без дополнительного запроса. При использовании параметра `-AsJob` управление возвращается в командную строку без ожидания завершения операции.

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>Дальнейшие действия
Из этого руководства вы узнали, как создавать стандартный масштабируемый набор и выполнять задачи управления с помощью Azure PowerShell:

> [!div class="checklist"]
> * Создание масштабируемого набора виртуальных машин и подключение к нему.
> * Выбор и использование образов виртуальных машин
> * Просмотр и использование определенных размеров виртуальных машин
> * Масштабирование масштабируемого набора вручную.
> * Выполнение стандартных задач управления масштабируемым набором.

Перейдите к следующему руководству, чтобы узнать о дисках масштабируемого набора.

> [!div class="nextstepaction"]
> [Использование дисков данных с масштабируемыми наборами](tutorial-use-disks-powershell.md)
