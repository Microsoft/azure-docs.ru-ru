---
title: Учебник по созданию виртуальных машин Windows и управлению ими с помощью Azure PowerShell
description: В этом руководстве описано, как с помощью Azure PowerShell создать и администрировать виртуальные машины Windows в Azure.
author: cynthn
ms.service: virtual-machines
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 06/06/2019
ms.author: cynthn
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: ae298562b5d3306cf6c20b4c2fb7b378ba99c986
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102561625"
---
# <a name="tutorial-create-and-manage-windows-vms-with-azure-powershell"></a>Руководство по Создание виртуальных машин Windows и управление ими с помощью Azure PowerShell

Виртуальные машины Azure предоставляют полностью настраиваемую и гибкую вычислительную среду. В этом руководстве рассматриваются основные элементы развертывания виртуальной машины Azure, например выбор ее размера, образа и ее развертывание. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание виртуальной машины и подключение к ней
> * Выбор и использование образов виртуальных машин
> * Просмотр и использование определенных размеров виртуальных машин
> * Изменение размера виртуальной машины
> * Просмотр виртуальной машины и оценка ее состояния

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.

## <a name="create-resource-group"></a>Создать группу ресурсов

Создайте группу ресурсов с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. Группу ресурсов следует создавать до виртуальной машины. В следующем примере создается группа ресурсов с именем *myResourceGroupVM* в регионе *EastUS*:

```azurepowershell-interactive
New-AzResourceGroup `
   -ResourceGroupName "myResourceGroupVM" `
   -Location "EastUS"
```

Группа ресурсов указывается при создании или изменении виртуальной машины, что показывается в этом руководстве.

## <a name="create-a-vm"></a>Создание виртуальной машины

При создании виртуальной машины можно настроить несколько параметров, включая образ операционной системы, конфигурацию сети и учетные данные администратора. В этом примере создается виртуальная машина *myVM*, на которой выполняется последняя стандартная версия Windows Server 2016 Datacenter.

Настройте на виртуальной машине имя пользователя и пароль для учетной записи администратора с помощью командлета [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential):

```azurepowershell-interactive
$cred = Get-Credential
```

Создайте виртуальную машину с помощью команды [New-AzVM](/powershell/module/az.compute/new-azvm).

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupVM" `
    -Name "myVM" `
    -Location "EastUS" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -Credential $cred
```

## <a name="connect-to-vm"></a>Подключение к виртуальной машине

После завершения развертывания создайте подключение к удаленному рабочему столу виртуальной машины.

Выполните приведенные ниже команды, чтобы получить общедоступный IP-адрес виртуальной машины. Запишите этот IP-адрес. Он потребуется, чтобы подключиться к виртуальной машине в браузере и проверить возможность подключения к Интернету на следующем этапе.

```azurepowershell-interactive
Get-AzPublicIpAddress `
   -ResourceGroupName "myResourceGroupVM"  | Select IpAddress
```

Для создания сеанса удаленного рабочего стола с виртуальной машиной выполните команду ниже на своем локальном компьютере. Замените указанный IP-адрес *общедоступным IP-адресом* своей виртуальной машины. При появлении запроса введите учетные данные, использованные при создании виртуальной машины.

```powershell
mstsc /v:<publicIpAddress>
```

В окне **Безопасность Windows** выберите **Варианты выбора** и нажмите **Использовать другую учетную запись**. Введите имя пользователя и пароль, созданные для виртуальной машины, и нажмите кнопку **ОК**.

## <a name="understand-marketplace-images"></a>Основные сведения о платежах в Marketplace

Azure Marketplace содержит множество образов, которые можно использовать для создания виртуальных машин. На предыдущих шагах виртуальная машина создавалась с помощью образа Windows Server 2016 Datacenter. На этом шаге модуль PowerShell используется для поиска других образов Windows на сайте Marketplace, которые можно также использовать для создания виртуальных машин. Этот процесс заключается в поиске сведений об издателе, предложении, номера SKU и (необязательно) номера версии для [идентификации](cli-ps-findimage.md#terminology) образа.

Получите список издателей образов, используя команду [Get-AzVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher):

```azurepowershell-interactive
Get-AzVMImagePublisher -Location "EastUS"
```

Получите список предложений образов, используя команду [Get-AzVMImageOffer](/powershell/module/az.compute/get-azvmimageoffer). Эта команда возвращает список, отфильтрованный по указанному издателю `MicrosoftWindowsServer`:

```azurepowershell-interactive
Get-AzVMImageOffer `
   -Location "EastUS" `
   -PublisherName "MicrosoftWindowsServer"
```

Должно отобразиться примерно следующее: 

```powershell
Offer             PublisherName          Location
-----             -------------          --------
Windows-HUB       MicrosoftWindowsServer EastUS
WindowsServer     MicrosoftWindowsServer EastUS
WindowsServer-HUB MicrosoftWindowsServer EastUS
```

Команда [Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku) отфильтрует список по имени издателя и названию предложения, отобразив список имен образов.

```azurepowershell-interactive
Get-AzVMImageSku `
   -Location "EastUS" `
   -PublisherName "MicrosoftWindowsServer" `
   -Offer "WindowsServer"
```

Должно отобразиться примерно следующее: 

```powershell
Skus                                      Offer         PublisherName          Location
----                                      -----         -------------          --------
2008-R2-SP1                               WindowsServer MicrosoftWindowsServer EastUS  
2008-R2-SP1-smalldisk                     WindowsServer MicrosoftWindowsServer EastUS  
2012-Datacenter                           WindowsServer MicrosoftWindowsServer EastUS  
2012-Datacenter-smalldisk                 WindowsServer MicrosoftWindowsServer EastUS  
2012-R2-Datacenter                        WindowsServer MicrosoftWindowsServer EastUS  
2012-R2-Datacenter-smalldisk              WindowsServer MicrosoftWindowsServer EastUS  
2016-Datacenter                           WindowsServer MicrosoftWindowsServer EastUS  
2016-Datacenter-Server-Core               WindowsServer MicrosoftWindowsServer EastUS  
2016-Datacenter-Server-Core-smalldisk     WindowsServer MicrosoftWindowsServer EastUS
2016-Datacenter-smalldisk                 WindowsServer MicrosoftWindowsServer EastUS
2016-Datacenter-with-Containers           WindowsServer MicrosoftWindowsServer EastUS
2016-Datacenter-with-Containers-smalldisk WindowsServer MicrosoftWindowsServer EastUS
2016-Datacenter-with-RDSH                 WindowsServer MicrosoftWindowsServer EastUS
2016-Nano-Server                          WindowsServer MicrosoftWindowsServer EastUS
```

Эти сведения можно использовать для развертывания виртуальной машины на основе конкретного образа. В этом примере развертывается виртуальная машина с помощью последней версии Windows Server 2016 и образа контейнеров.

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupVM" `
    -Name "myVM2" `
    -Location "EastUS" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress2" `
    -ImageName "MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest" `
    -Credential $cred `
    -AsJob
```

Параметр `-AsJob` создает виртуальную машину как фоновую задачу, поэтому PowerShell отображает запрос о возврате. Подробные сведения о фоновых заданиях можно просмотреть с помощью командлета `Get-Job`.

## <a name="understand-vm-sizes"></a>Описание размеров виртуальных машин

Размер виртуальной машины определяет количество выделяемых ей вычислительных ресурсов, таких как ЦП, GPU и память. Размер создаваемых виртуальных машин должен соответствовать ожидаемой рабочей нагрузке. При увеличении рабочей нагрузки размер существующей виртуальной машины может быть изменен.

### <a name="vm-sizes"></a>Размеры виртуальных машин

В приведенной ниже таблицы указаны категории размеров и примеры использования.  

| Тип                     | Распространенные размеры           |    Описание       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [Универсальные](../sizes-general.md)         |B, Dsv3, Dv3, DSv2, Dv2, Av2, DC| Сбалансированное соотношение ресурсов ЦП и памяти. Идеально подходят для разработки и тестирования малых и средних приложений и решений для обработки данных.  |
| [Оптимизированные для вычислений](../sizes-compute.md)   | Fsv2          | Высокое соотношение ресурсов ЦП и памяти. Подходят для приложений со средним объемом трафика, сетевых устройств и пакетных процессов.        |
| [Оптимизированные для памяти](../sizes-memory.md)    | Esv3, Ev3, M, DSv2, Dv2  | Высокое соотношение ресурсов памяти и числа ядер. Отлично подходят для реляционных баз данных, кэша среднего и большого объема, а также выполняющейся в памяти аналитики.                 |
| [Оптимизированные для хранилища](../sizes-storage.md)      | Lsv2, Ls              | Высокая пропускная способность дисков и количество операций ввода-вывода. Идеальный вариант для работы с большими данными, а также с базами данных SQL и NoSQL.                                                         |
| [GPU](../sizes-gpu.md)          | NV, NVv2, NC, NCv2, NCv3, ND            | Специализированные виртуальные машины, предназначенные для ресурсоемкой отрисовки изображений и редактирования видео.       |
| [Высокопроизводительные](../sizes-hpc.md) | H        | Виртуальные машины с самыми мощными ЦП, для которых можно настроить сетевые интерфейсы с высокой пропускной способностью (RDMA). |

### <a name="find-available-vm-sizes"></a>Поиск всех доступных размеров виртуальных машин

Чтобы просмотреть список доступных размеров виртуальных машин в определенном регионе, используйте команду [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize).

```azurepowershell-interactive
Get-AzVMSize -Location "EastUS"
```

## <a name="resize-a-vm"></a>Изменение размера виртуальной машины

После развертывания виртуальной машины ее размер можно изменить, чтобы увеличить или уменьшить выделенные ей ресурсы.

Перед изменением размера виртуальной машины проверьте, доступен ли нужный размер в текущем кластере виртуальных машин. Команда [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize) возвращает список размеров.

```azurepowershell-interactive
Get-AzVMSize -ResourceGroupName "myResourceGroupVM" -VMName "myVM"
```

Если нужный размер доступен, то размер виртуальной машины можно изменить во включенном состоянии, однако виртуальную машину нужно будет перезагрузить.

```azurepowershell-interactive
$vm = Get-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -VMName "myVM"
$vm.HardwareProfile.VmSize = "Standard_DS3_v2"
Update-AzVM `
   -VM $vm `
   -ResourceGroupName "myResourceGroupVM"
```

Если желаемый размер в текущем кластере недоступен, то перед изменением размера виртуальную машину нужно освободить. Обратите внимание на то, что после повторного включения виртуальной машины все данные на временном диске будут удалены, а общедоступный IP-адрес изменится, если только не используется статический IP-адрес.

```azurepowershell-interactive
Stop-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM" -Force
$vm = Get-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -VMName "myVM"
$vm.HardwareProfile.VmSize = "Standard_E2s_v3"
Update-AzVM -VM $vm `
   -ResourceGroupName "myResourceGroupVM"
Start-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -Name $vm.name
```

## <a name="vm-power-states"></a>Состояния включенной виртуальной машины

Включенная виртуальная машина Azure может находиться в одном из многих состояний. 


| Состояние включения | Описание
|----|----|
| Запуск | Виртуальная машина запускается. |
| Запущен | Виртуальная машина запущена. |
| Остановка | Виртуальная машина останавливается. |
| Остановлена | Виртуальная машина остановилась. За виртуальные машины в остановленном состоянии по-прежнему взимается плата за вычислительные операции.  |
| Отмена выделения | Виртуальная машина освобождается. |
| Освобождено | Указывает, что виртуальная машина удалена из гипервизора, но по-прежнему доступна в плоскости управления. За виртуальные машины в состоянии `Deallocated` не взимается плата за вычислительные операции. |
| - | Состояние включенной виртуальной машины неизвестно. |


Чтобы получить состояние конкретной виртуальной машины, используйте команду [Get-AzVM](/powershell/module/az.compute/get-azvm). Необходимо указать допустимое имя виртуальной машины и группы ресурсов.

```azurepowershell-interactive
Get-AzVM `
    -ResourceGroupName "myResourceGroupVM" `
    -Name "myVM" `
    -Status | Select @{n="Status"; e={$_.Statuses[1].Code}}
```

Результат должен выглядеть следующим образом.

```powershell
Status
------
PowerState/running
```

Чтобы узнать рабочее состояние всех виртуальных машин в подписке, используйте [API "Виртуальные машины — показать все"](/rest/api/compute/virtualmachines/listall) с параметром **statusOnly**, которому задано значение *true*.

## <a name="management-tasks"></a>Задачи управления

В течение жизненного цикла виртуальной машины можно выполнять задачи управления, такие как запуск, остановка или удаление виртуальной машины. Кроме того, можно создавать скрипты для автоматизации повторяющихся или сложных задач. С помощью Azure PowerShell можно выполнять множество распространенных задач управления в командной строке или в скриптах.

### <a name="stop-a-vm"></a>Остановка виртуальной машины

Остановите и освободите виртуальную машину с помощью команды [Stop-AzVM](/powershell/module/az.compute/stop-azvm):

```azurepowershell-interactive
Stop-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM" -Force
```

Если вы хотите сохранить виртуальную машину в подготовленном состоянии, задайте параметр -StayProvisioned.

### <a name="start-a-vm"></a>Запуск виртуальной машины

```azurepowershell-interactive
Start-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM"
```

### <a name="delete-resource-group"></a>Удалить группу ресурсов

При удалении группы ресурсов удаляются и все содержащиеся в ней элементы.

```azurepowershell-interactive
Remove-AzResourceGroup `
   -Name "myResourceGroupVM" `
   -Force
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства вы изучили основы создания виртуальной машины и управления ею. Вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание виртуальной машины и подключение к ней
> * Выбор и использование образов виртуальных машин
> * Просмотр и использование определенных размеров виртуальных машин
> * Изменение размера виртуальной машины
> * Просмотр виртуальной машины и оценка ее состояния

Перейдите к следующему руководству, чтобы узнать о дисках виртуальных машин.  

> [!div class="nextstepaction"]
> [Управление дисками Azure с помощью Azure CLI](./tutorial-manage-data-disk.md)