---
title: Ускоритель записи Azure
description: Документация о том, как включить и использовать ускоритель записи.
author: raiye
manager: markkie
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure
ms.date: 2/20/2019
ms.author: raiye
ms.subservice: disks
ms.openlocfilehash: 827643866c23583051bc290c2c50bed3f1bdd421
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98737920"
---
# <a name="enable-write-accelerator"></a>Включение ускорителя записи

Ускоритель записи — это функция диска для виртуальных машин серии M в хранилище уровня "Премиум", использующих исключительно управляемые диски Azure. Как очевидно из названия, целью функции является уменьшение задержки операций ввода-вывода при записи в службу хранилища Azure уровня "Премиум". Ускоритель записи идеально подходит для случаев, когда требуется высокопроизводительное сохранение на диск обновлений файлов журнала для современных баз данных.

В общедоступном облаке выпущена общедоступная версия ускорителя записи для виртуальных машин серии M.

## <a name="planning-for-using-write-accelerator"></a>Планирование использования ускорителя записи

Эту функцию нужно использовать для томов, которые содержат журналы транзакций или повторяемых операций СУБД. Не рекомендуется использовать ускоритель записи для томов данных СУБД, так как эта функция оптимизирована для дисков журнала.

Ускоритель записи работает только с [управляемыми дисками Azure](https://azure.microsoft.com/services/managed-disks/).

> [!IMPORTANT]
> Включение ускорителя записи для диска операционной системы виртуальной машины приведет к перезагрузке виртуальной машины.
>
> Чтобы включить ускоритель записи в имеющемся диске Azure, который не является частью тома, состоящего из нескольких дисков, включая диспетчеры дисков или томов Windows, дисковые пространства Windows, масштабируемый файловый сервер Windows (SOFS), диспетчер логических томов Linux или MDADM, нужно завершить работу рабочей нагрузки с доступом к диску Azure. Нужно завершить работу приложений баз данных, использующих диск Azure.
>
> Если вы хотите включить или отключить ускоритель записи для имеющегося тома, который состоит из нескольких дисков службы хранилища Azure уровня "Премиум" и чередуется с помощью диспетчеров дисков или томов Windows, дисковых пространств Windows, масштабируемого файлового сервера Windows (SOFS), диспетчера логических томов Linux или MDADM, то все диски, из которых состоит том, нужно включить или отключить для ускорителя записи с помощью отдельных действий. **Перед включением или выключением ускорителя записи в такой конфигурации завершите работу виртуальной машины Azure**.

Включение ускорителя записи для дисков ОС не является необходимым шагом для конфигураций виртуальных машин, связанных с SAP.

### <a name="restrictions-when-using-write-accelerator"></a>Ограничения при использовании ускорителя записи

При использовании ускорителя записи для диска Azure или виртуального жесткого диска применяются приведенные ниже ограничения.

- Для кэширования диска уровня "Премиум" необходимо установить значение "Нет" или "Только для чтения". Все другие режимы кэширования не поддерживаются.
- Моментальные снимки для дисков с включенным ускорителем записи пока не поддерживаются. Во время резервного копирования Azure Backup автоматически исключает диски с включенным ускорителем записи, подключенные к виртуальной машине.
- Ускоренный путь используется только для записи на диск операций ввода-вывода меньшего размера (менее 512 КиБ). При использовании рабочих нагрузок, где данные поступают в ходе массовой загрузки или где буферы журнала транзакций разных DBMS заполняются перед сохранением в хранилище в большей степени, ускоренный путь, скорее всего, не будет использоваться.

Есть ограничения на число виртуальных жестких дисков службы хранилища Azure уровня "Премиум" для каждой виртуальной машины, которые могут поддерживаться ускорителем записи. Сейчас действуют такие ограничения:

| SKU виртуальной машины | Число дисков с ускорителем записи | Число дисковых операций ввода-вывода в секунду на виртуальную машину при использовании ускорителя записи |
| --- | --- | --- |
| M416ms_v2, M416s_v2| 16 | 20 000 |
| M208ms_v2, M208s_v2| 8 | 10000 |
| M128ms, M128s | 16 | 20 000 |
| M64ms, M64ls, M64s | 8 | 10000 |
| M32ms, M32ls, M32ts, M32s | 4 | 5000 |
| M16ms, M16s | 2 | 2500 |
| M8ms, M8s | 1 | 1250 |

Ограничения числа операций ввода-вывода в секунду указаны для виртуальной машины, а *не* для диска. Для всех дисков ускорителя записи действует общее ограничение числа операций ввода-вывода в секунду на виртуальную машину. Подключенные диски не могут превышать ограничение на число операций ввода-вывода для виртуальной машины. Например, несмотря на то, что подключенные диски могут выполнять 30 000 операций ввода-вывода в секунду, система не допускает, чтобы диски перейдут выше 20 000 операций ввода-вывода для M416ms_v2.

## <a name="enabling-write-accelerator-on-a-specific-disk"></a>Включение ускорителя записи на определенном диске

В нескольких разделах ниже описано, как включить ускоритель записи на виртуальных жестких дисках службы хранилища Azure уровня "Премиум".

### <a name="prerequisites"></a>Предварительные условия

На данный момент к использованию ускорителя записи применяются предварительные требования, приведенные ниже.

- Диски, к которым вы хотите применить ускоритель записи Azure, должны быть [управляемыми дисками Azure](https://azure.microsoft.com/services/managed-disks/) в хранилище уровня "Премиум".
- Необходимо использовать виртуальную машину серии M.

## <a name="enabling-azure-write-accelerator-using-azure-powershell"></a>Включение ускорителя записи Azure с помощью Azure PowerShell

Модуль Azure PowerShell не ниже версии 5.5.0 включает в себя изменения соответствующих командлетов для включения или отключения ускорителя записи для определенных дисков службы хранилища Azure уровня "Премиум".
Чтобы включить или развернуть диски, поддерживаемые ускорителем записи, команды PowerShell ниже были изменены и расширены и теперь принимают параметр для ускорителя.

В командлеты ниже был добавлен новый параметр переключения **-WriteAccelerator**.

- [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk)
- [Add-AzVMDataDisk](/powershell/module/az.compute/Add-AzVMDataDisk)
- [Set-AzVMDataDisk](/powershell/module/az.compute/Set-AzVMDataDisk)
- [Add-AzVmssDataDisk](/powershell/module/az.compute/Add-AzVmssDataDisk)

Если не задать параметр, свойству присваивается значение false и развертываются диски, которые не поддерживаются ускорителем записи.

В командлеты ниже был добавлен новый параметр переключения **-OsDiskWriteAccelerator**.

- [Set-AzVmssStorageProfile](/powershell/module/az.compute/Set-AzVmssStorageProfile)

Отсутствие заданного значения параметра присваивает свойству значение false по умолчанию, возвращая диски, не использующие ускоритель записи.

В командлеты ниже был добавлен новый необязательный логический (не допускающий значения NULL) параметр — **-OsDiskWriteAccelerator**.

- [Update-AzVM](/powershell/module/az.compute/Update-AzVM)
- [Update-AzVmss](/powershell/module/az.compute/Update-AzVmss)

Укажите значения $true или $false, чтобы управлять поддержкой ускорителя записи Azure с дисками.

Примеры команд могут выглядеть так:

```powershell
New-AzVMConfig | Set-AzVMOsDisk | Add-AzVMDataDisk -Name "datadisk1" | Add-AzVMDataDisk -Name "logdisk1" -WriteAccelerator | New-AzVM

Get-AzVM | Update-AzVM -OsDiskWriteAccelerator $true

New-AzVmssConfig | Set-AzVmssStorageProfile -OsDiskWriteAccelerator | Add-AzVmssDataDisk -Name "datadisk1" -WriteAccelerator:$false | Add-AzVmssDataDisk -Name "logdisk1" -WriteAccelerator | New-AzVmss

Get-AzVmss | Update-AzVmss -OsDiskWriteAccelerator:$false
```

Для двух основных сценариев можно написать скрипты, как показано в следующих разделах.

### <a name="adding-a-new-disk-supported-by-write-accelerator-using-powershell"></a>Добавление нового диска, поддерживаемого ускорителем записи с помощью PowerShell

Этот скрипт можно использовать, чтобы добавить новый диск на виртуальную машину. Диск, созданный с помощью этого сценария, будет использовать ускоритель записи.

Замените `myVM`, `myWAVMs`, `log001`, размер диска и LunID диска значениями, соответствующими вашему конкретному развертыванию.

```powershell
# Specify your VM Name
$vmName="myVM"
#Specify your Resource Group
$rgName = "myWAVMs"
#data disk name
$datadiskname = "log001"
#LUN Id
$lunid=8
#size
$size=1023
#Pulls the VM info for later
$vm=Get-AzVM -ResourceGroupName $rgname -Name $vmname
#add a new VM data disk
Add-AzVMDataDisk -CreateOption empty -DiskSizeInGB $size -Name $vmname-$datadiskname -VM $vm -Caching None -WriteAccelerator:$true -lun $lunid
#Updates the VM with the disk config - does not require a reboot
Update-AzVM -ResourceGroupName $rgname -VM $vm
```

### <a name="enabling-write-accelerator-on-an-existing-azure-disk-using-powershell"></a>Включение ускорителя записи на имеющемся диске Azure с помощью PowerShell

Этот сценарий позволяет включить ускоритель записи на существующем диске. Замените `myVM`, `myWAVMs` и `test-log001` на значения, соответствующие вашему конкретному развертыванию. Сценарий добавляет ускоритель записи на имеющийся диск. Для параметра **$newstatus** установлено значение $true. При использовании значения "$false" ускоритель записи на имеющемся диске будет отключен.

```powershell
#Specify your VM Name
$vmName="myVM"
#Specify your Resource Group
$rgName = "myWAVMs"
#data disk name
$datadiskname = "test-log001" 
#new Write Accelerator status ($true for enabled, $false for disabled) 
$newstatus = $true
#Pulls the VM info for later
$vm=Get-AzVM -ResourceGroupName $rgname -Name $vmname
#add a new VM data disk
Set-AzVMDataDisk -VM $vm -Name $datadiskname -Caching None -WriteAccelerator:$newstatus
#Updates the VM with the disk config - does not require a reboot
Update-AzVM -ResourceGroupName $rgname -VM $vm
```

> [!Note]
> Выполнение скрипта выше приведет к отключению указанного диска, включению на нем ускорителя записи и последующему подключению диска.

## <a name="enabling-write-accelerator-using-the-azure-portal"></a>Включение ускорителя записи с помощью портала Azure

Вы можете включить ускоритель записи на портале, указав параметры кэширования дисков.

![Ускоритель записи на портале Azure](./media/virtual-machines-common-how-to-enable-write-accelerator/wa_scrnsht.png)

## <a name="enabling-write-accelerator-using-the-azure-cli"></a>Включение ускорителя записи с помощью Azure CLI

Для включения ускорителя записи можно использовать [Azure CLI](/cli/azure/).

Чтобы включить ускоритель записи на существующем диске, выполните команду [az vm update](/cli/azure/vm#az_vm_update). Вы можете использовать следующие примеры, заменив diskName, VMName и ResourceGroup собственными значениями: `az vm update -g group1 -n vm1 -write-accelerator 1=true`.

Чтобы подключить диск с включенным ускорителем записи, выполните команду [az vm disk attach](/cli/azure/vm/disk#az_vm_disk_attach). Вы можете использовать следующий пример, если подставите собственные значения: `az vm disk attach -g group1 -vm-name vm1 -disk d1 --enable-write-accelerator`.

Чтобы отключить Ускоритель записи, используйте команду [AZ VM Update](/cli/azure/vm#az_vm_update), задав для свойств значение false: `az vm update -g group1 -n vm1 -write-accelerator 0=false 1=false`

## <a name="enabling-write-accelerator-using-rest-apis"></a>Включение ускорителя записи через интерфейсы REST API

Чтобы выполнить развертывание с помощью Azure REST API, необходимо установить клиент ARMClient Azure.

### <a name="install-armclient"></a>Установка клиента ARMClient

Чтобы запустить клиент ARMClient, нужно установить его с помощью Chocolatey. Его можно установить с помощью cmd.exe или PowerShell. Чтобы выполнить эти команды, используйте запуск от имени администратора.

При использовании cmd.exe выполните следующую команду: `@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"`

При использовании PowerShell выполните следующую команду: `Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))`

Теперь вы можете установить клиент ARMClient с помощью следующей команды как в cmd.exe, так и в PowerShell.`choco install armclient`

### <a name="getting-your-current-vm-configuration"></a>Получение текущей конфигурации виртуальной машины

Чтобы изменить атрибуты конфигурации диска, вам сначала нужно получить текущую конфигурацию в JSON-файле. Выполните следующую команду, чтобы получить текущую конфигурацию: `armclient GET /subscriptions/<<subscription-ID<</resourceGroups/<<ResourceGroup>>/providers/Microsoft.Compute/virtualMachines/<<virtualmachinename>>?api-version=2017-12-01 > <<filename.json>>`

Замените термины в скобках <<   >> своими данными, включая имя файла, которое должно быть присвоено JSON-файлу.

Выходные данные будут выглядеть примерно так, как показано ниже:

```JSON
{
  "properties": {
    "vmId": "2444c93e-f8bb-4a20-af2d-1658d9dbbbcb",
    "hardwareProfile": {
      "vmSize": "Standard_M64s"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "SUSE",
        "offer": "SLES-SAP",
        "sku": "12-SP3",
        "version": "latest"
      },
      "osDisk": {
        "osType": "Linux",
        "name": "mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a",
        "createOption": "FromImage",
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Premium_LRS",
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a"
        },
        "diskSizeGB": 30
      },
      "dataDisks": [
        {
          "lun": 0,
          "name": "data1",
          "createOption": "Attach",
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data1"
          },
          "diskSizeGB": 1023
        },
        {
          "lun": 1,
          "name": "log1",
          "createOption": "Attach",
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data2"
          },
          "diskSizeGB": 1023
        }
      ]
    },
    "osProfile": {
      "computerName": "mylittlesapVM",
      "adminUsername": "pl",
      "linuxConfiguration": {
        "disablePasswordAuthentication": false
      },
      "secrets": []
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Network/networkInterfaces/mylittlesap518"
        }
      ]
    },
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "https://mylittlesapdiag895.blob.core.windows.net/"
      }
    },
    "provisioningState": "Succeeded"
  },
  "type": "Microsoft.Compute/virtualMachines",
  "location": "westeurope",
  "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/virtualMachines/mylittlesapVM",
  "name": "mylittlesapVM"

```

Далее нужно обновить JSON-файл и включить ускоритель записи на диске с именем "log1". Это можно сделать, добавив этот атрибут в JSON-файл после записи кэша диска.

```JSON
        {
          "lun": 1,
          "name": "log1",
          "createOption": "Attach",
          "caching": "None",
          "writeAcceleratorEnabled": true,
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data2"
          },
          "diskSizeGB": 1023
        }
```

Затем обновите имеющееся развертывание с помощью следующей команды: `armclient PUT /subscriptions/<<subscription-ID<</resourceGroups/<<ResourceGroup>>/providers/Microsoft.Compute/virtualMachines/<<virtualmachinename>>?api-version=2017-12-01 @<<filename.json>>`

Выходные данные должны выглядеть, как показано ниже: Как видно, для одного диска включен ускоритель записи.

```JSON
{
  "properties": {
    "vmId": "2444c93e-f8bb-4a20-af2d-1658d9dbbbcb",
    "hardwareProfile": {
      "vmSize": "Standard_M64s"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "SUSE",
        "offer": "SLES-SAP",
        "sku": "12-SP3",
        "version": "latest"
      },
      "osDisk": {
        "osType": "Linux",
        "name": "mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a",
        "createOption": "FromImage",
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Premium_LRS",
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a"
        },
        "diskSizeGB": 30
      },
      "dataDisks": [
        {
          "lun": 0,
          "name": "data1",
          "createOption": "Attach",
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data1"
          },
          "diskSizeGB": 1023
        },
        {
          "lun": 1,
          "name": "log1",
          "createOption": "Attach",
          "caching": "None",
          "writeAcceleratorEnabled": true,
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data2"
          },
          "diskSizeGB": 1023
        }
      ]
    },
    "osProfile": {
      "computerName": "mylittlesapVM",
      "adminUsername": "pl",
      "linuxConfiguration": {
        "disablePasswordAuthentication": false
      },
      "secrets": []
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Network/networkInterfaces/mylittlesap518"
        }
      ]
    },
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "https://mylittlesapdiag895.blob.core.windows.net/"
      }
    },
    "provisioningState": "Succeeded"
  },
  "type": "Microsoft.Compute/virtualMachines",
  "location": "westeurope",
  "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/virtualMachines/mylittlesapVM",
  "name": "mylittlesapVM"
```

После реализации изменения ускоритель записи должен поддерживать диск.
