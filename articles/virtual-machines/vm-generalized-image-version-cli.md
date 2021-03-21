---
title: Создание виртуальной машины из обобщенного образа с помощью Azure CLI
description: Создайте виртуальную машину из обобщенной версии образа, используя Azure CLI.
author: cynthn
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.custom: devx-track-azurecli
ms.openlocfilehash: 2bcaf85f61a4d8cf4d23c9c5be7f46d765d77dbb
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102551048"
---
# <a name="create-a-vm-from-a-generalized-image-version-using-the-cli"></a>Создание виртуальной машины из универсальной версии образа с помощью интерфейса командной строки

Создайте виртуальную машину из [универсальной версии образа](./shared-image-galleries.md#generalized-and-specialized-images) , хранящейся в коллекции общих образов. Если вы хотите создать виртуальную машину с помощью специализированного образа, см. раздел [Создание виртуальной машины на основе специализированного образа](vm-specialized-image-version-powershell.md). 


## <a name="get-the-image-id"></a>Получение идентификатора образа

Перечислите определения образов в коллекции, выполнив команду [AZ SIG Image-Definition List](/cli/azure/sig/image-definition#az-sig-image-definition-list) , чтобы просмотреть имя и идентификатор определений.

```azurecli-interactive 
resourceGroup=myGalleryRG
gallery=myGallery
az sig image-definition list --resource-group $resourceGroup --gallery-name $gallery --query "[].[name, id]" --output tsv
```

## <a name="create-the-vm"></a>Создание виртуальной машины

Создайте виртуальную машину, выполнив команду [az vm create](/cli/azure/vm#az-vm-create). Чтобы использовать последнюю версию образа, задайте `--image` идентификатор определения образа. 

При необходимости замените имена ресурсов в этом примере. 

```azurecli-interactive 
imgDef="/subscriptions/<subscription ID where the gallery is located>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition"
vmResourceGroup=myResourceGroup
location=eastus
vmName=myVM
adminUsername=azureuser


az group create --name $vmResourceGroup --location $location

az vm create\
   --resource-group $vmResourceGroup \
   --name $vmName \
   --image $imgDef \
   --admin-username $adminUsername \
   --generate-ssh-keys
```

Вы также можете использовать определенную версию с помощью идентификатора версии образа для `--image` параметра. Например, чтобы использовать тип Image версии *1.0.0* : `--image "/subscriptions/<subscription ID where the gallery is located>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition/versions/1.0.0"` .

## <a name="next-steps"></a>Дальнейшие действия

С помощью [построителя образов Azure (Предварительная версия)](./image-builder-overview.md) можно автоматизировать создание версий изображений, а также использовать его для обновления и [создания новой версии образа из существующей версии образа](./linux/image-builder-gallery-update-image-version.md).