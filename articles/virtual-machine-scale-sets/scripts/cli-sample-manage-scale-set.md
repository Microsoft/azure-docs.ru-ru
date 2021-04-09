---
title: Пример Azure CLI для управления масштабируемым набором виртуальных машин
description: В этом примере показано, как добавить диски в масштабируемый набор виртуальных машин. Вы можете обновить диски и добавить виртуальные машины в службу проверки подлинности Azure AD.
author: mimckitt
ms.author: mimckitt
ms.date: 02/04/2021
ms.topic: sample
ms.service: virtual-machine-scale-sets
ms.devlang: azurecli
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1abdf7ae15753d78ac8728f57e9b0cd5dcd9165e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101672602"
---
# <a name="create-and-manage-virtual-machine-scale-set"></a>Создание масштабируемого набора виртуальных машин и управление им

Используйте приведенные здесь примеры команд, чтобы создать прототип масштабируемого набора виртуальных машин с помощью Azure CLI.

В этих примерах команд демонстрируются следующие операции:

* Создает масштабируемый набор виртуальных машин.
* Добавление и обновление новых или существующих дисков в масштабируемом наборе или в отдельном экземпляре набора.
* Добавление масштабируемого набора в службу проверки подлинности Azure Active Directory (Azure AD).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

## <a name="sample-commands"></a>Примеры команд

```azurecli
# Create a resource group
az group create --name MyResourceGroup --location eastus

# Create virtual machine scale set
az vmss create --resource-group MyResourceGroup --name myScaleSet --instance-count 2 \
    --image UbuntuLTS --upgrade-policy-mode automatic --admin-username azureuser \
    --generate-ssh-keys

# Attach a new managed disk to your scale set
az vmss disk attach --resource-group MyResourceGroup --vmss-name myScaleSet --size-gb 50
```

После добавления нового диска данных отформатируйте и подключите этот диск. Сведения о виртуальных машинах Windows см. в статье [Как подключить управляемый диск данных к виртуальной машине Windows с помощью портала Azure](../../virtual-machines/windows/attach-managed-disk-portal.md). Сведения о виртуальных машинах Linux см. в статье [Добавление диска к виртуальной машине Linux](../../virtual-machines/linux/add-disk.md).

```azurecli
# Attach an existing managed disk to a virtual machine instance in your scale set
az vmss disk attach --resource-group MyResourceGroup --disk myDataDisk \
    --vmss-name myScaleSet --instance-id 0

# See the instances in your virtual machine scale set
az vmss list-instances --resource-group MyResourceGroup --name myScaleSet --output table

# See the disks for your virtual machine
az disk list --resource-group MyResourceGroup \
    --query "[*].{Name:name,Gb:diskSizeGb,Tier:accountType}" --output table

# Deallocate the virtual machine
az vmss deallocate --resource-group MyResourceGroup --name myScaleSet --instance-ids 0 

# Resize the disk
az disk update --resource-group MyResourceGroup --name myDataDisk --size-gb 200

# Restart the disk
az vmss restart --resource-group MyResourceGroup --name myScaleSet --instance-ids 0
```

Чтобы использовать расширенный диск, расширьте его основной раздел. Дополнительные сведения см. разделе [Расширение раздела диска и файловой системы](../../virtual-machines/linux/expand-disks.md#expand-a-disk-partition-and-filesystem).

В этом примере изменяется размер диска данных. Эту же процедуру можно использовать и для обновления диска ОС. Дополнительные сведения о виртуальной машине Windows см. в статье [Расширение диска ОС виртуальной машины](../../virtual-machines/windows/expand-os-disk.md). Дополнительные сведения о виртуальных машинах Linux см. в статье [Расширение виртуальных жестких дисков на виртуальной машине Linux с помощью Azure CLI](../../virtual-machines/linux/expand-disks.md).

```azurecli
# Enable managed service identity on your scale set. This is required to authenticate and interact with other Azure services using bearer tokens.
az vmss identity assign --resource-group MyResourceGroup --name myScaleSet --role Owner \
    --scope /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/MyResourceGroup

# Connect to Azure AD authentication
az vmss extension set --resource-group MyResourceGroup --name AADLoginForWindows \
    --publisher Microsoft.Azure.ActiveDirectory --vmss-name myScaleSet

# Upgrade one instance of a scale set virtual machine
az vmss update-instances --resource-group MyResourceGroup --name myScaleSet --instance-ids 0 

# Remove a managed disk from the scale set
az vmss disk detach --resource-group MyResourceGroup --vmss-name myScaleSet --lun 0

# Remove a managed disk from an instance
az vmss disk detach --resource-group MyResourceGroup --vmss-name myScaleSet --instance-id 1 --lun 0

# Delete the pre-existing disk
az disk delete --resource-group MyResourceGroup --disk myDataDisk
```

## <a name="clean-up-resources"></a>Очистка ресурсов

После выполнения этих команд запустите следующую команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```azurecli
az group delete --name MyResourceGroup
```

## <a name="azure-cli-references-used-in-this-article"></a>Ссылки на команды Azure CLI, используемые в этой статье

* [az disk delete](/cli/azure/disk#az_disk_delete)
* [az disk list](/cli/azure/disk#az_disk_list)
* [az disk update](/cli/azure/disk#az_disk_update)
* [az group create](/cli/azure/group#az_group_create)
* [az vmss create](/cli/azure/vmss#az_vmss_create)
* [az virtual machine scale set deallocate](/cli/azure/vmss#az_vmss_deallocate)
* [az vmss disk attach](/cli/azure/vmss/disk#az_vmss_disk_attach)
* [az vmss disk detach](/cli/azure/vmss/disk#az_vmss_disk_detach)
* [az vmss extension set](/cli/azure/vmss/extension#az_vmss_extension_set)
* [az vmss identity assign](/cli/azure/vmss/identity#az_vmss_identity_assign)
* [az vmss list-instances](/cli/azure/vmss#az_vmss_list_instances)
* [az vmss restart](/cli/azure/vmss#az_vmss_restart)
* [az vmss update-instances](/cli/azure/vmss#az_vmss_update_instances)