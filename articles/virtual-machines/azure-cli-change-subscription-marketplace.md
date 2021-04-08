---
title: Пример для Azure CLI, позволяющий переместить виртуальную машину Azure Marketplace в другую подписку
description: Пример для Azure CLI, предназначенный для перемещения виртуальной машины Azure Marketplace в другую подписку.
author: cynthn
ms.author: cynthn
manager: ''
ms.date: 01/29/2021
ms.topic: sample
ms.service: virtual-machines
ms.devlang: azurecli
ms.custom: devx-track-azurecli
ms.openlocfilehash: e70284a3a89c30d0f1d1363a15652449a6a1d4f5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100558100"
---
# <a name="move-a-marketplace-azure-virtual-machine-to-another-subscription"></a>Перемещение виртуальной машины Azure Marketplace в другую подписку

Чтобы переместить виртуальную машину Marketplace в другую подписку, нужно переместить в новую подписку диск операционной системы и повторно создать виртуальную машину.

Эта процедура не требуется, если вы просто перемещаете диск данных в новую подписку. Вместо этого создайте виртуальную машину из Marketplace в новой подписке, а затем переместите и подключите диск данных.

В этом скрипте показано три операции:

- создание моментального снимка диска ОС;
- перемещение моментального снимка в другую подписку;
- создание виртуальной машины на основе этого моментального снимка.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="sample-script"></a>Пример скрипта

Чтобы переместить виртуальную машину Marketplace в другую подписку, нужно создать новую виртуальную машину на основе того же предложения Marketplace и перемещенного диска ОС.

> [!NOTE]
> Если план старой виртуальной машины уже не доступен в Marketplace, эту процедуру использовать невозможно.

```azurecli
#!/bin/bash
# Set variable values before proceeding. 

# Variables
sourceResourceGroup= Resource group for the current virtual machine
sourceSubscription= Subscription for the current virtual machine
vmName= Name of the current virtual machine

destinationResourceGroup= Resource group for the new virtual machine, create if necessary
destinationSubscription= Subscription for the new virtual machine

# Set your current subscription for the source virtual machine
az account set --subscription $sourceSubscription

# Load variables about your virtual machine
# osType = windows or linux
osType=$(az vm get-instance-view --resource-group $sourceResourceGroup \
    --name $vmName --subscription $sourceSubscription \
    --query 'storageProfile.osDisk.osType' --output tsv)

# offer = Your offer in Marketplace
offer=$(az vm get-instance-view --resource-group $sourceResourceGroup \
    --name $vmName --query 'storageProfile.imageReference.offer' --output tsv)

# plan = Your plan in Marketplace
plan=$(az vm get-instance-view --resource-group $sourceResourceGroup \
    --name $vmName --query 'plan' --output tsv)

# publisher = Your publisher in Marketplace
publisher=$(az vm get-instance-view --resource-group $sourceResourceGroup \
    --name $vmName --query 'storageProfile.imageReference.publisher' --output tsv)

# Get information to create new virtual machine
planName=$(az vm get-instance-view --resource-group $sourceResourceGroup \
    --subscription $sourceSubscription --query 'plan.name' --name $vmName)
planProduct=$(az vm get-instance-view --resource-group $sourceResourceGroup \
    --subscription $sourceSubscription --query 'plan.product' --name $vmName)
planPublisher=$(az vm get-instance-view --resource-group $sourceResourceGroup \
    --subscription $sourceSubscription --query 'plan.publisher' --name $vmName)

# Get the name of the OS disk
osDiskName=$(az vm show --resource-group $sourceResourceGroup --name $vmName \
    --query 'storageProfile.osDisk.name' --output tsv)

# Verify the terms for your market virtual machine
az vm image terms show --offer $offer --plan '$plan' --publisher $publisher \
    --subscription $sourceSubscription

# Deallocate the virtual machine
az vm deallocate --resource-group $sourceResourceGroup --name $vmName

# Create a snapshot of the OS disk
az snapshot create --resource-group $sourceResourceGroup --name MigrationSnapshot \
    --source "/subscriptions/$sourceSubscription/resourceGroups/$sourceResourceGroup/providers/Microsoft.Compute/disks/$osDiskName"

# Move the snapshot to your destination resource group
az resource move --destination-group $destinationResourceGroup \
    --destination-subscription-id $destinationSubscription \
    --ids "/subscriptions/$sourceSubscription/resourceGroups/$sourceResourceGroup/providers/Microsoft.Compute/snapshots/MigrationSnapshot"

# Set your subscription to the destination value
az account set --subscription $destinationSubscription

# Accept the terms from the Marketplace
az vm image terms accept --offer $offer --plan '$plan' --publisher $publisher \
    --subscription $destinationSubscription

# Create disk from the snapshot 
az disk create --resource-group $destinationResourceGroup --name DestinationDisk \
    --source "/subscriptions/$destinationSubscription/resourceGroups/$destinationResourceGroup/providers/Microsoft.Compute/snapshots/MigrationSnapshot" \
    --os-type $osType

# Create virtual machine from disk
az vm create --resource-group $destinationResourceGroup --name $vmName \
    --plan-name $planName --plan-product $planProduct  --plan-publisher $planPublisher \
    --attach-os-disk "/subscriptions/$destinationSubscription/resourceGroups/$destinationResourceGroup/providers/Microsoft.Compute/disks/DestinationDisk" \
    --os-type $osType
```

## <a name="clean-up-resources"></a>Очистка ресурсов

После выполнения этого примера выполните указанные ниже команды, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```azurecli
az group delete --name $sourceResourceGroup --subscription $sourceSubscription
az group delete --name $destinationResourceGroup --subscription $destinationSubscription
```

## <a name="azure-cli-references-used-in-this-article"></a>Ссылки на команды Azure CLI, используемые в этой статье

- [az account set](/cli/azure/account#az_account_set)
- [az disk create](/cli/azure/disk#az_disk_create)
- [az group delete](/cli/azure/group#az_group_delete)
- [az resource move](/cli/azure/resource#az_resource_move)
- [az snapshot create](/cli/azure/snapshot#az_snapshot_create)
- [az vm create](/cli/azure/vm#az_vm_create)
- [az vm deallocate](/cli/azure/vm#az_vm_deallocate)
- [az vm delete](/cli/azure/vm#az_vm_delete)
- [az vm get-instance-view](/cli/azure/vm#az_vm_get_instance_view)
- [az vm image terms accept](/cli/azure/vm/image/terms#az_vm_image_terms_accept)
- [az vm image terms show](/cli/azure/vm/image/terms#az_vm_image_terms_show)
- [az vm show](/cli/azure/vm#az_vm_show)

## <a name="next-steps"></a>Дальнейшие действия

- [Перемещение виртуальных машин в другой регион Azure](../site-recovery/azure-to-azure-tutorial-migrate.md)
- [Перемещение виртуальной машины в другую подписку или группу ресурсов](./linux/move-vm.md)
