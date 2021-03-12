---
title: Создание моментального снимка виртуального жесткого диска с помощью портала или PowerShell
description: Узнайте, как создать копию виртуальной машины Azure для использования в качестве резервного копирования или для устранения проблем с помощью портала или PowerShell.
author: roygara
manager: twooley
ms.service: virtual-machines
ms.subservice: disks
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 10/08/2018
ms.author: rogarana
ms.openlocfilehash: 9070b69ac4c6b85791ff3dd4662273e75a3cd22c
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102556066"
---
# <a name="create-a-snapshot-using-the-portal-or-powershell"></a>Создание моментального снимка с помощью портала или PowerShell

Моментальный снимок — это полная копия виртуального жесткого диска, доступная только для чтения. Создайте моментальный снимок диска операционной системы или виртуального жесткого диска данных для использования в качестве резервных ресурсов или устранения неполадок виртуальной машины.

Если вы намерены создать новую виртуальную машину на основе моментального снимка, мы рекомендуем аккуратно завершить работу виртуальной машины перед созданием моментального снимка, чтобы очистить все выполняющиеся процессы.

## <a name="use-the-azure-portal"></a>Использование портала Azure 

Чтобы создать моментальный снимок, выполните следующие действия. 
1.  На [портал Azure](https://portal.azure.com)выберите **создать ресурс**.
2. Найдите и выберите **snapshot (моментальный снимок**).
3. В окне **Моментальный снимок** нажмите кнопку **Создать**. Откроется окно **Создание моментального снимка**.
4. Заполните поле **Имя** для моментального снимка.
5. Введите имя новой [группы ресурсов](../../azure-resource-manager/management/overview.md#resource-groups) или выберите имеющуюся. 
6. Выберите **расположение** центра обработки данных Azure.  
7. В поле **Исходный диск** выберите управляемый диск, моментальный снимок которого необходимо создать.
8. Выберите **тип учетной записи**, которая будет использоваться для хранения моментального снимка. Выберите **Standard_HDD**, если вам не нужно хранить моментальный снимок на высокопроизводительном диске.
9. Нажмите кнопку **создания**.

## <a name="use-powershell"></a>Использование PowerShell

В следующих шагах показано, как скопировать диск виртуального жесткого диска и создать конфигурацию моментального снимка. Затем можно создать моментальный снимок диска с помощью командлета [New-азснапшот](/powershell/module/az.compute/new-azsnapshot) . 

 

1. Задайте некоторые параметры: 

   ```azurepowershell-interactive
   $resourceGroupName = 'myResourceGroup' 
   $location = 'eastus' 
   $vmName = 'myVM'
   $snapshotName = 'mySnapshot'  
   ```

2. Получите виртуальную машину.

   ```azurepowershell-interactive
   $vm = Get-AzVM `
       -ResourceGroupName $resourceGroupName `
       -Name $vmName
   ```

3. Создайте конфигурацию моментального снимка. В этом примере моментальный снимок — это снимок диска операционной системы:

   ```azurepowershell-interactive
   $snapshot =  New-AzSnapshotConfig `
       -SourceUri $vm.StorageProfile.OsDisk.ManagedDisk.Id `
       -Location $location `
       -CreateOption copy
   ```
   
   > [!NOTE]
   > Перед сохранением моментального снимка в хранилище, отказоустойчивое в пределах зоны, его необходимо создать в регионе, который поддерживает [зоны доступности](../../availability-zones/az-overview.md) и в котором включен параметр `-SkuName Standard_ZRS`.   
   
4. Создайте моментальный снимок.

   ```azurepowershell-interactive
   New-AzSnapshot `
       -Snapshot $snapshot `
       -SnapshotName $snapshotName `
       -ResourceGroupName $resourceGroupName 
   ```


## <a name="next-steps"></a>Дальнейшие действия

Создайте виртуальную машину из моментального снимка, преобразовав его в управляемый диск, а затем подключив этот диск как диск ОС. Дополнительные сведения см. в статье [Создание виртуальной машины из моментального снимка с помощью PowerShell](/previous-versions/azure/virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm-from-snapshot).