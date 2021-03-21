---
title: Расширение диска операционной системы виртуальной машины Windows в Azure
description: Увеличьте размер диска операционной системы виртуальной машины, используя Azure PowerShell в модели развертывания диспетчер ресурсов.
services: virtual-machines
documentationcenter: ''
author: kirpasingh
manager: roshar
editor: ''
tags: azure-resource-manager
ms.assetid: d9edfd9f-482f-4c0b-956c-0d2c2c30026c
ms.service: virtual-machines
ms.collection: windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 09/02/2020
ms.author: kirpas
ms.subservice: disks
ms.openlocfilehash: af57bc396349286c3edcaefc8385ddbaec066226
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103199474"
---
# <a name="how-to-expand-the-os-drive-of-a-virtual-machine"></a>Расширение диска ОС виртуальной машины

При создании новой виртуальной машины в группе ресурсов путем развертывания образа из [Azure Marketplace](https://azure.microsoft.com/marketplace/)диск ОС по умолчанию часто 127 Гб (по умолчанию некоторые образы имеют меньшие размеры диска ОС). Несмотря на то, что можно добавить диски данных на виртуальную машину (число зависит от выбранного номера SKU) и мы рекомендуем устанавливать приложения и рабочие нагрузки с интенсивным использованием ЦП на этих дополнительных дисках, часто клиенты должны расширить диск операционной системы для поддержки конкретных сценариев:

- Для поддержки устаревших приложений, устанавливающих компоненты на диск операционной системы.
- Перенос физического компьютера или виртуальной машины из локальной среды с более крупным диском операционной системы.

> [!IMPORTANT]
> Для изменения размера операционной системы или диска данных виртуальной машины Azure требуется освободить виртуальную машину.
>
> Сжатие существующего диска не поддерживается и потенциально может привести к потере данных.
> 
> После расширения диска необходимо [расширить том ОС](#expand-the-volume-within-the-os), чтобы использовать преимущества диска большего размера.

## <a name="resize-a-managed-disk-in-the-azure-portal"></a>Изменение размера управляемого диска в портал Azure

1. В [портал Azure](https://portal.azure.com)перейдите к виртуальной машине, в которой нужно расширить диск. Выберите пункт " **прерывать** ", чтобы освободить виртуальную машину.
2. Когда виртуальная машина будет остановлена, в меню слева в разделе **Параметры** выберите **диски**.

    :::image type="content" source="./media/expand-os-disk/select-disks.png" alt-text="Снимок экрана, показывающий параметр &quot;диски&quot;, выбранный в разделе &quot;Параметры&quot; меню.":::

 
3. В разделе **имя диска** выберите диск, размер которого нужно изменить.

    :::image type="content" source="./media/expand-os-disk/disk-name.png" alt-text="Снимок экрана, на котором отображается панель диски с выбранным именем диска.":::

4. В меню слева в разделе **Параметры** выберите **размер и производительность**.

    :::image type="content" source="./media/expand-os-disk/configuration.png" alt-text="Снимок экрана, на котором показан параметр &quot;размер и производительность&quot;, выбранный в разделе &quot;Параметры&quot; меню.":::

5. В поле **размер и производительность** выберите нужный размер диска.
   
   > [!WARNING]
   > Новый размер должен быть больше, чем размер существующего диска. Максимально допустимый объем — 2 048 ГБ для дисков ОС. (Вы можете расширить большой двоичный объект VHD, не выполнив его, но операционная система будет работать только с первыми 2 048 ГБ пространства.)
   > 

    :::image type="content" source="./media/expand-os-disk/size.png" alt-text="Снимок экрана, на котором показана область размера и производительности с выбранным размером диска.":::

6. Выберите **изменить размер** в нижней части страницы.

    :::image type="content" source="./media/expand-os-disk/save.png" alt-text="Снимок экрана, на котором показана панель &quot;размер и производительность&quot; с выбранной кнопкой &quot;изменить размер&quot;.":::


## <a name="resize-a-managed-disk-by-using-powershell"></a>Изменение размера управляемого диска с помощью PowerShell

Откройте окно интегрированной среды сценариев PowerShell или PowerShell в режиме администрирования и выполните следующие действия:

1. Войдите в учетную запись Microsoft Azure в режиме управления ресурсами и выберите свою подписку:
   
    ```powershell
    Connect-AzAccount
    Select-AzSubscription –SubscriptionName 'my-subscription-name'
    ```

2. Задайте имя группы ресурсов и имя виртуальной машины:
   
    ```powershell
    $rgName = 'my-resource-group-name'
    $vmName = 'my-vm-name'
    ```

3. Получите ссылку на виртуальную машину:
   
    ```powershell
    $vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

4. Перед изменением размера диска завершите работу виртуальной машины:
   
    ```powershell
    Stop-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

5. Получите ссылку на управляемый диск ОС. Задайте требуемое значение размера управляемого диска ОС и обновите диск:
   
    ```powershell
    $disk= Get-AzDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.OsDisk.Name
    $disk.DiskSizeGB = 1023
    Update-AzDisk -ResourceGroupName $rgName -Disk $disk -DiskName $disk.Name
    ```   
    > [!WARNING]
    > Новый размер должен быть больше, чем размер существующего диска. Максимально допустимый объем — 2 048 ГБ для дисков ОС. (Большой двоичный объект VHD можно расширить сверх этого размера, но операционная система работает только с первыми 2 048 ГБ пространства.)
    > 
         
6. Обновление виртуальной машины может занять несколько секунд. После завершения выполнения команды перезапустите виртуальную машину.
   
    ```powershell
    Start-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

Вот и все! Теперь подключитесь по RDP к виртуальной машине, откройте раздел "Управление компьютером" (или "Управление дисками") и увеличьте диск, используя только что выделенное для этого место.

## <a name="resize-an-unmanaged-disk-by-using-powershell"></a>Изменение размера неуправляемого диска с помощью PowerShell

Откройте окно интегрированной среды сценариев PowerShell или PowerShell в режиме администрирования и выполните следующие действия:

1. Войдите в учетную запись Microsoft Azure в режиме управления ресурсами и выберите свою подписку:
   
    ```powershell
    Connect-AzAccount
    Select-AzSubscription –SubscriptionName 'my-subscription-name'
    ```

2. Задайте имя группы ресурсов и имена виртуальных машин:
   
    ```powershell
    $rgName = 'my-resource-group-name'
    $vmName = 'my-vm-name'
    ```

3. Получите ссылку на виртуальную машину:
   
    ```powershell
    $vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

4. Перед изменением размера диска завершите работу виртуальной машины:
   
    ```powershell
    Stop-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

5. Задайте требуемое значение размера неуправляемого диска ОС и обновите виртуальную машину:
   
    ```powershell
    $vm.StorageProfile.OSDisk.DiskSizeGB = 1023
    Update-AzVM -ResourceGroupName $rgName -VM $vm
    ```
   
    > [!WARNING]
    > Новый размер должен быть больше, чем размер существующего диска. Максимально допустимый объем — 2 048 ГБ для дисков ОС. (Вы можете расширить большой двоичный объект VHD, если он больше этого размера, но операционная система сможет работать только с первыми 2 048 ГБ пространства.)
    > 
    > 
   
6. Обновление виртуальной машины может занять несколько секунд. После завершения выполнения команды перезапустите виртуальную машину.
   
    ```powershell
    Start-AzVM -ResourceGroupName $rgName -Name $vmName
    ```


## <a name="scripts-for-os-disk"></a>Сценарии для диска операционной системы

Ниже приведен полный код сценария для ссылки на управляемые и неуправляемые диски:


**Управляемые диски**

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName 'my-subscription-name'
$rgName = 'my-resource-group-name'
$vmName = 'my-vm-name'
$vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
Stop-AzVM -ResourceGroupName $rgName -Name $vmName
$disk= Get-AzDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.OsDisk.Name
$disk.DiskSizeGB = 1023
Update-AzDisk -ResourceGroupName $rgName -Disk $disk -DiskName $disk.Name
Start-AzVM -ResourceGroupName $rgName -Name $vmName
```

**Неуправляемые диски**

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName 'my-subscription-name'
$rgName = 'my-resource-group-name'
$vmName = 'my-vm-name'
$vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
Stop-AzVM -ResourceGroupName $rgName -Name $vmName
$vm.StorageProfile.OSDisk.DiskSizeGB = 1023
Update-AzVM -ResourceGroupName $rgName -VM $vm
Start-AzVM -ResourceGroupName $rgName -Name $vmName
```

## <a name="resizing-data-disks"></a>Изменение размера дисков данных

В этой статье основное внимание уделено расширению диска ОС виртуальной машины. Но разработанный сценарий также можно использовать для увеличения дисков данных, подключенных к виртуальной машине. Например, чтобы увеличить первый диск с данными, подключенный к виртуальной машине, замените объект `OSDisk` в `StorageProfile` на массив `DataDisks` и используйте числовой индекс для получения ссылки на первый подключенный диск с данными, как показано ниже:

**Управляемый диск**

```powershell
$disk= Get-AzDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.DataDisks[0].Name
$disk.DiskSizeGB = 1023
```

**Неуправляемый диск**

```powershell
$vm.StorageProfile.DataDisks[0].DiskSizeGB = 1023
```

Аналогичным образом можно ссылаться на другие диски данных, подключенные к виртуальной машине, с помощью индекса, как показано выше, или свойства **имени** диска:


**Управляемый диск**

```powershell
(Get-AzDisk -ResourceGroupName $rgName -DiskName ($vm.StorageProfile.DataDisks | Where ({$_.Name -eq 'my-second-data-disk'})).Name).DiskSizeGB = 1023
```

**Неуправляемый диск**

```powershell
($vm.StorageProfile.DataDisks | Where ({$_.Name -eq 'my-second-data-disk'})).DiskSizeGB = 1023
```

## <a name="expand-the-volume-within-the-os"></a>Расширение тома в ОС

Когда вы развернете диск для виртуальной машины, необходимо переключиться в операционную систему и расширить том, чтобы охватить новое пространство. Существует несколько способов расширения раздела. В этом разделе описано подключение виртуальной машины с использованием протокола удаленного рабочего стола для расширения раздела с помощью **DiskPart**.

1. Откройте подключение к виртуальной машине по протоколу удаленного рабочего стола.

2. Откройте командную строку и введите **diskpart**.

3. В командной строке **DISKPART** введите `list volume`. Найдите том, который требуется расширить.

4. В командной строке **DISKPART** введите `select volume <volumenumber>`. При этом будет выбран том с указанным *номером тома*, который вы хотите расширить в смежную пустую область на том же диске.

5. В командной строке **DISKPART** введите `extend [size=<size>]`. Эта команда расширяет выбранный том на указанный *размер* в МБ.


## <a name="next-steps"></a>Дальнейшие действия

Диски также можно подключить на [портале Azure](attach-managed-disk-portal.md).
