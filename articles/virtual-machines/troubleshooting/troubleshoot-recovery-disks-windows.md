---
title: Использование виртуальной машины Windows для устранения неполадок с помощью Azure PowerShell | Документация Майкрософт
description: Узнайте, как устранять неполадки с виртуальными машинами Windows в Azure, подключив диск операционной системы к виртуальной машине восстановления с помощью Azure PowerShell.
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 08/09/2018
ms.author: genli
ms.openlocfilehash: fbdccc2efb72626d1974f4cd81be9d2ef4c522d4
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/14/2021
ms.locfileid: "98201592"
---
# <a name="troubleshoot-a-windows-vm-by-attaching-the-os-disk-to-a-recovery-vm-using-azure-powershell"></a>Устранение неполадок с виртуальной машиной Windows при подключении диска операционной системы к виртуальной машине восстановления с помощью Azure PowerShell
Если возникает проблема с загрузкой или диском на виртуальной машине Windows в Azure, возможно, вам нужно устранить неполадки, связанные с самим диском. Например, такая ситуация может возникнуть из-за сбоя обновления приложения, который мешает успешно загрузить виртуальную машину. В этой статье подробно описано, как с помощью Azure PowerShell подключить диск к другой виртуальной машине Windows для устранения ошибок, а затем исправить исходную виртуальную машину. 

> [!Important]
> Скрипты в этой статье применяются только к виртуальным машинам, которые используют [управляемый диск](../managed-disks-overview.md). 

 

## <a name="recovery-process-overview"></a>Обзор процесса восстановления
Теперь мы можем использовать Azure PowerShell, чтобы изменить диск операционной системы для виртуальной машины. Больше не нужно удалять и повторно создавать виртуальную машину.

Процесс устранения неполадок выглядит следующим образом.

1. Остановите затронутую виртуальную машину.
2. Создайте моментальный снимок, используя диск ОС виртуальной машины.
3. Создайте диск из моментального снимка диска операционной системы.
4. Подключите диск как диск данных к виртуальной машине для восстановления.
5. Подключитесь к виртуальной машине восстановления. Измените файлы или запустите средства, которые нужны для устранения неполадок на скопированном диске ОС.
6. Отключите и отсоедините диск от виртуальной машины восстановления.
7. Измените диск операционной системы для затронутой виртуальной машины.

Вы можете использовать команды восстановления виртуальной машины для автоматизации шагов 1, 2, 3, 4, 6 и 7. Дополнительную документацию и инструкции см. [в статье восстановление ВИРТУАЛЬНОЙ машины Windows с помощью команд восстановления виртуальной машины Azure](repair-windows-vm-using-azure-virtual-machine-repair-commands.md).

Убедитесь, что у вас установлена и настроена [последняя версия Azure PowerShell](/powershell/azure/) и выполнен вход в подписку.

```powershell
Connect-AzAccount
```

В следующих примерах замените имена параметров собственными значениями. 

## <a name="determine-boot-issues"></a>Выявление проблем при загрузке
Вы можете просмотреть снимок экрана виртуальной машины в Azure, чтобы устранить неполадки загрузки. Этот снимок экрана может помочь определить, почему виртуальная машина не загружается. Приведенный ниже пример получает снимок экрана виртуальной машины Windows `myVM`, расположенной в группе ресурсов `myResourceGroup`.

```powershell
Get-AzVMBootDiagnosticsData -ResourceGroupName myResourceGroup `
    -Name myVM -Windows -LocalPath C:\Users\ops\
```

Просмотрите его, чтобы определить причину сбоя загрузки виртуальной машины. Обратите внимание на соответствующие сообщения об ошибках или коды ошибок.

## <a name="stop-the-vm"></a>Остановка виртуальной машины

В следующем примере прекращается работа виртуальной машины `myVM` из группы ресурсов `myResourceGroup`:

```powershell
Stop-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"
```

Подождите завершения удаления виртуальной машины перед выполнением следующего шага.


## <a name="create-a-snapshot-from-the-os-disk-of-the-vm"></a>Создание моментального снимка на основе диска ОС виртуальной машины

В следующем примере создается моментальный снимок с именем `mySnapshot` из диска операционной системы виртуальной машины "myVM". 

```powershell
$resourceGroupName = 'myResourceGroup' 
$location = 'eastus' 
$vmName = 'myVM'
$snapshotName = 'mySnapshot'  

#Get the VM
$vm = get-azvm `
-ResourceGroupName $resourceGroupName `
-Name $vmName

#Create the snapshot configuration for the OS disk
$snapshot =  New-AzSnapshotConfig `
-SourceUri $vm.StorageProfile.OsDisk.ManagedDisk.Id `
-Location $location `
-CreateOption copy

#Take the snapshot
New-AzSnapshot `
   -Snapshot $snapshot `
   -SnapshotName $snapshotName `
   -ResourceGroupName $resourceGroupName 
```

Моментальный снимок — это полная копия виртуального жесткого диска, доступная только для чтения. Его нельзя присоединить к виртуальной машине. На следующем шаге мы создадим диск из этого моментального снимка.

## <a name="create-a-disk-from-the-snapshot"></a>Создание диска из моментального снимка

Этот скрипт создает управляемый диск с именем `newOSDisk` из моментального снимка `mysnapshot`.  

```powershell
#Set the context to the subscription Id where Managed Disk will be created
#You can skip this step if the subscription is already selected

$subscriptionId = 'yourSubscriptionId'

Select-AzSubscription -SubscriptionId $SubscriptionId

#Provide the name of your resource group
$resourceGroupName ='myResourceGroup'

#Provide the name of the snapshot that will be used to create Managed Disks
$snapshotName = 'mySnapshot' 

#Provide the name of the Managed Disk
$diskName = 'newOSDisk'

#Provide the size of the disks in GB. It should be greater than the VHD file size.
$diskSize = '128'

#Provide the storage type for Managed Disk. Premium_LRS or Standard_LRS.
$storageType = 'Standard_LRS'

#Provide the Azure region (e.g. westus) where Managed Disks will be located.
#This location should be same as the snapshot location
#Get all the Azure location using command below:
#Get-AzLocation
$location = 'eastus'

$snapshot = Get-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotName 
 
$diskConfig = New-AzDiskConfig -AccountType $storageType -Location $location -CreateOption Copy -SourceResourceId $snapshot.Id
 
New-AzDisk -Disk $diskConfig -ResourceGroupName $resourceGroupName -DiskName $diskName
```
Теперь у вас есть копия исходного диска ОС. Вы можете подключить диск к другой виртуальной машине Windows для устранения неполадок.

## <a name="attach-the-disk-to-another-windows-vm-for-troubleshooting"></a>Подключение диска к другой виртуальной Машине Windows для устранения неполадок

Теперь мы подключим копию исходного диска ОС к виртуальной машине как диск данных. Этот процесс позволяет исправить на диске ошибки конфигурации или изучить дополнительные файлы журналов приложения или системы. В следующем примере диск с именем `newOSDisk` подключается к виртуальной машине `RecoveryVM`.

> [!NOTE]
> Чтобы подключить диск, копия исходного диска ОС и виртуальная машина восстановления должны находиться в одном расположении.

```powershell
$rgName = "myResourceGroup"
$vmName = "RecoveryVM"
$location = "eastus" 
$dataDiskName = "newOSDisk"
$disk = Get-AzDisk -ResourceGroupName $rgName -DiskName $dataDiskName 

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName 

$vm = Add-AzVMDataDisk -CreateOption Attach -Lun 0 -VM $vm -ManagedDiskId $disk.Id

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

## <a name="connect-to-the-recovery-vm-and-fix-issues-on-the-attached-disk"></a>Подключение к виртуальной машине восстановления и устранение неполадок на подключенном диске

1. Подключитесь к виртуальной машине восстановления по протоколу RDP, используя соответствующие учетные данные. Следующий пример скачивает RDP-файл подключения для виртуальной машины `RecoveryVM` в группе ресурсов `myResourceGroup` и сохраняет его в паку `C:\Users\ops\Documents`.

    ```powershell
    Get-AzRemoteDesktopFile -ResourceGroupName "myResourceGroup" -Name "RecoveryVM" `
        -LocalPath "C:\Users\ops\Documents\myVMRecovery.rdp"
    ```

2. Диск данных будет автоматически обнаружен и подключен. Просмотрите список подключенных томов, чтобы определить букву диска, как показано ниже.

    ```powershell
    Get-Disk
    ```

    В следующем примере выходных данных показан диск, подключенный как диск **2**. (Для просмотра буква диска можно также использовать `Get-Volume`).

    ```powershell
    Number   Friendly Name   Serial Number   HealthStatus   OperationalStatus   Total Size   Partition
                                                                                             Style
    ------   -------------   -------------   ------------   -----------------   ----------   ----------
    0        Virtual HD                                     Healthy             Online       127 GB MBR
    1        Virtual HD                                     Healthy             Online       50 GB MBR
    2        newOSDisk                                  Healthy             Online       127 GB MBR
    ```

После подключения копии исходного диска ОС вы можете выполнить любые необходимые действия по устранению неполадок и обслуживанию. После устранения проблем выполните следующие действия.

## <a name="unmount-and-detach-original-os-disk"></a>Отключение и отсоединение исходного диска ОС
После устранения ошибок отключите и отсоедините существующий диск от виртуальной машины восстановления. Диск нельзя использовать с другой виртуальной машиной, пока вы не отмените аренду, присоединяющую диск к виртуальной машине восстановления.

1. Из сеанса RDP отключите диск данных на виртуальной машине восстановления. Для этого нужен номер диска из предыдущего командлета `Get-Disk`. Используйте `Set-Disk`, чтобы отключить диск.

    ```powershell
    Set-Disk -Number 2 -IsOffline $True
    ```

    Убедитесь, что диск отключен, еще раз выполнив `Get-Disk`. В следующем примере выходных данных показано, что теперь диск отключен.

    ```powershell
    Number   Friendly Name   Serial Number   HealthStatus   OperationalStatus   Total Size   Partition
                                                                                             Style
    ------   -------------   -------------   ------------   -----------------   ----------   ----------
    0        Virtual HD                                     Healthy             Online       127 GB MBR
    1        Virtual HD                                     Healthy             Online       50 GB MBR
    2        Msft Virtu...                                  Healthy             Offline      127 GB MBR
    ```

2. Выйдите из сеанса RDP. В рамках сеанса Azure PowerShell удалите диск с именем `newOSDisk` из виртуальной машины "RecoveryVM".

    ```powershell
    $myVM = Get-AzVM -ResourceGroupName "myResourceGroup" -Name "RecoveryVM"
    Remove-AzVMDataDisk -VM $myVM -Name "newOSDisk"
    Update-AzVM -ResourceGroup "myResourceGroup" -VM $myVM
    ```

## <a name="change-the-os-disk-for-the-affected-vm"></a>Изменение диска операционной системы для затронутой виртуальной машины

Для переключения дисков ОС можно использовать Azure PowerShell. Нет необходимости удалять и повторно создавать виртуальную машину.

Этот пример останавливает виртуальную машину `myVM`, а затем присваивает диск `newOSDisk` в качестве нового диска ОС. 

```powershell
# Get the VM 
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM 

# Make sure the VM is stopped\deallocated
Stop-AzVM -ResourceGroupName myResourceGroup -Name $vm.Name -Force

# Get the new disk that you want to swap in
$disk = Get-AzDisk -ResourceGroupName myResourceGroup -Name newDisk

# Set the VM configuration to point to the new disk  
Set-AzVMOSDisk -VM $vm -ManagedDiskId $disk.Id -Name $disk.Name  -sto

# Update the VM with the new OS disk. Possible values of StorageAccountType include: 'Standard_LRS' and 'Premium_LRS'
Update-AzVM -ResourceGroupName myResourceGroup -VM $vm -StorageAccountType <Type of the storage account >

# Start the VM
Start-AzVM -Name $vm.Name -ResourceGroupName myResourceGroup
```

## <a name="verify-and-enable-boot-diagnostics"></a>Проверка и включение диагностики загрузки

В следующем примере на виртуальной машине `myVMDeployed` в группе ресурсов `myResourceGroup` включается расширение диагностики:

```powershell
$myVM = Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVMDeployed"
Set-AzVMBootDiagnostics -ResourceGroupName myResourceGroup -VM $myVM -enable
Update-AzVM -ResourceGroup "myResourceGroup" -VM $myVM
```

## <a name="next-steps"></a>Дальнейшие действия
При возникновении проблем с подключением к виртуальной машине ознакомьтесь со статьей [Устранение неполадок с подключением к удаленному рабочему столу на виртуальной машине Azure под управлением Windows](troubleshoot-rdp-connection.md). Для устранения проблем с доступом к приложениям, выполняющимся на виртуальной машине, прочитайте статью [Устранение неполадок доступа к приложению, выполняющемуся в виртуальной машине Azure](troubleshoot-app-connection.md).

Дополнительные сведения об использовании Resource Manager вы найдете в статье [Общие сведения об Azure Resource Manager](../../azure-resource-manager/management/overview.md).
