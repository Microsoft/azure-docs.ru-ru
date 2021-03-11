---
title: Преобразование хранилища управляемых дисков между разными типами дисков с помощью Azure CLI
description: Как преобразовать управляемые диски Azure между различными типами дисков с помощью Azure CLI.
author: roygara
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.date: 02/13/2021
ms.author: albecker
ms.subservice: disks
ms.openlocfilehash: 35ba681ae35e60d7cd275c9e649b2463d2dc30a0
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102558463"
---
# <a name="convert-azure-managed-disks-storage-from-standard-to-premium-or-premium-to-standard"></a>Преобразование хранилища управляемых дисков Azure из уровня "Стандартный" в "Премиум" или "Премиум" в "Стандартный"

Существует четыре типа дисков, управляемых с Azure: диски Azure Ultra, SSD уровня "Премиум", "Стандартный SSD" и "Стандартный HDD". В зависимости от потребностей производительности можно переключаться между твердотельным накопителем уровня "Премиум", стандартным SSD и стандартным HDD. Вы еще не можете переключиться с Ultra Disk или на него. необходимо развернуть новый.

Эта функция не поддерживается для неуправляемых дисков. Но вы можете легко [преобразовать неуправляемый диск в управляемый](convert-unmanaged-to-managed-disks.md) , чтобы иметь возможность переключаться между типами дисков.

В этой статье показано, как преобразовать управляемые диски из одного типа диска в другой с помощью Azure CLI. Сведения об установке или обновлении средства см. в разделе [install Azure CLI](/cli/azure/install-azure-cli).

## <a name="before-you-begin"></a>Подготовка к работе

* Для преобразования диска требуется перезагрузка виртуальной машины, поэтому запланируйте перенос дискового накопителя во время работы предварительно существующего периода обслуживания.
* Для неуправляемых дисков сначала [преобразуйте их в управляемые диски](convert-unmanaged-to-managed-disks.md) , чтобы можно было переключаться между вариантами хранения.


## <a name="switch-all-managed-disks-of-a-vm-between-from-one-account-to-another"></a>Переключение всех управляемых дисков виртуальной машины с одной учетной записи на другую

В этом примере показано, как преобразовать все диски виртуальной машины в хранилище класса Premium. Однако, изменив переменную номера SKU в этом примере, можно преобразовать диски виртуальной машины в стандартный SSD или стандартный HDD. Обратите внимание, что для использования управляемых дисков уровня "Премиум" виртуальная машина должна использовать [Размер виртуальной машины](../sizes.md) , поддерживающий хранилище класса Premium. В этом примере также переключается на размер, поддерживающий хранилище класса Premium.

 ```azurecli

#resource group that contains the virtual machine
rgName='yourResourceGroup'

#Name of the virtual machine
vmName='yourVM'

#Premium capable size 
#Required only if converting from Standard to Premium
size='Standard_DS2_v2'

#Choose between Standard_LRS, StandardSSD_LRS and Premium_LRS based on your scenario
sku='Premium_LRS'

#Deallocate the VM before changing the size of the VM
az vm deallocate --name $vmName --resource-group $rgName

#Change the VM size to a size that supports Premium storage 
#Skip this step if converting storage from Premium to Standard
az vm resize --resource-group $rgName --name $vmName --size $size

#Update the SKU of all the data disks 
az vm show -n $vmName -g $rgName --query storageProfile.dataDisks[*].managedDisk -o tsv \
 | awk -v sku=$sku '{system("az disk update --sku "sku" --ids "$1)}'

#Update the SKU of the OS disk
az vm show -n $vmName -g $rgName --query storageProfile.osDisk.managedDisk -o tsv \
| awk -v sku=$sku '{system("az disk update --sku "sku" --ids "$1)}'

az vm start --name $vmName --resource-group $rgName

```
## <a name="switch-individual-managed-disks-from-one-disk-type-to-another"></a>Переключение между отдельными управляемыми дисками с одного типа диска на другой

Для рабочей нагрузки для разработки и тестирования может потребоваться сочетание дисков уровня "Стандартный" и "Премиум" для снижения затрат. Можно выбрать обновление только тех дисков, для которых требуется более высокая производительность. В этом примере показано, как преобразовать один диск виртуальной машины из уровня "Стандартный" в хранилище "Премиум". Однако, изменив переменную номера SKU в этом примере, можно преобразовать диски виртуальной машины в стандартный SSD или стандартный HDD. Чтобы использовать управляемые диски уровня "Премиум", виртуальная машина должна использовать [Размер виртуальной машины](../sizes.md) , поддерживающий хранилище класса Premium. В этом примере также переключается на размер, поддерживающий хранилище класса Premium.

 ```azurecli

#resource group that contains the managed disk
rgName='yourResourceGroup'

#Name of your managed disk
diskName='yourManagedDiskName'

#Premium capable size 
#Required only if converting from Standard to Premium
size='Standard_DS2_v2'

#Choose between Standard_LRS, StandardSSD_LRS and Premium_LRS based on your scenario
sku='Premium_LRS'

#Get the parent VM Id 
vmId=$(az disk show --name $diskName --resource-group $rgName --query managedBy --output tsv)

#Deallocate the VM before changing the size of the VM
az vm deallocate --ids $vmId 

#Change the VM size to a size that supports Premium storage 
#Skip this step if converting storage from Premium to Standard
az vm resize --ids $vmId --size $size

# Update the SKU
az disk update --sku $sku --name $diskName --resource-group $rgName 

az vm start --ids $vmId 
```

## <a name="switch-managed-disks-from-one-disk-type-to-another"></a>Переключение управляемых дисков с одного типа на другой

Выполните указанные ниже действия:

1. Войдите на [портал Microsoft Azure](https://portal.azure.com).
2. Выберите виртуальную машину из списка **виртуальных машин**.
3. Если виртуальная машина не остановлена, выберите **остановить** в верхней части панели **Обзор** виртуальной машины и дождитесь остановки виртуальной машины.
4. В области виртуальной машины в меню выберите **диски** .
5. Выберите диск, который требуется преобразовать.
6. В меню выберите пункт **Конфигурация** .
7. Измените **тип учетной записи** с диска исходного типа на нужный тип диска.
8. Нажмите кнопку **сохранить** и закройте область диск.

Обновление типа диска происходит мгновенно. После преобразования виртуальную машину можно перезапустить.

## <a name="next-steps"></a>Дальнейшие действия

Сделайте копию виртуальной машины, доступную только для чтения, с помощью [моментальных снимков](snapshot-copy-managed-disk.md).