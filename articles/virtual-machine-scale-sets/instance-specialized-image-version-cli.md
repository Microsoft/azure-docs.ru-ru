---
title: Создание масштабируемого набора из специализированной версии образа с помощью Azure CLI
description: Создайте масштабируемый набор с помощью специализированной версии образа в коллекции общих образов с помощью Azure CLI.
author: cynthn
ms.service: virtual-machine-scale-sets
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/01/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: devx-track-azurecli
ms.openlocfilehash: 2ecc2bfe6bdc06ede61e6c4d1e6eccfc9ef6323a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98878008"
---
# <a name="create-a-scale-set-using-a-specialized-image-version-with-the-azure-cli"></a>Создание масштабируемого набора с помощью специализированной версии образа с Azure CLI

Создание масштабируемого набора из [специализированной версии образа](../virtual-machines/shared-image-galleries.md#generalized-and-specialized-images) , хранящейся в коллекции общих образов. Если вы хотите создать масштабируемый набор с помощью обобщенной версии образа, см. раздел [Создание масштабируемого набора из обобщенного образа](instance-generalized-image-version-cli.md).

Если вы решили установить и использовать интерфейс командной строки локально, то для работы с этим руководством вам понадобится Azure CLI 2.4.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0]( /cli/azure/install-azure-cli).

При необходимости замените имена ресурсов в этом примере. 

Перечислите определения образов в коллекции, выполнив команду [AZ SIG Image-Definition List](/cli/azure/sig/image-definition#az-sig-image-definition-list) , чтобы просмотреть имя и идентификатор определений.

```azurecli-interactive 
resourceGroup=myGalleryRG
gallery=myGallery
az sig image-definition list \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --query "[].[name, id]" \
   --output tsv
```

Создайте масштабируемый набор с помощью [`az vmss create`](/cli/azure/vmss#az-vmss-create) `--specialized` параметра, чтобы указать, что изображение является специализированным изображением.

Используйте идентификатор определения образа в качестве значения параметра `--image`, чтобы создать экземпляры масштабируемого набора на основе последней доступной версии образа. Вы также можете создать экземпляры масштабируемого набора на основе определенной версии, указав идентификатор версии образа в параметре `--image`. Имейте в виду, что использование конкретной версии образа означает, что автоматизация может завершиться ошибкой, если эта версия изображения недоступна, так как она была удалена или удалена из региона. Рекомендуется использовать идентификатор определения образа для создания новой виртуальной машины, если только не требуется определенная версия образа.

В этом примере мы создаем экземпляры из последней версии образа *мимажедефинитион* .

```azurecli
az group create --name myResourceGroup --location eastus
az vmss create \
   --resource-group myResourceGroup \
   --name myScaleSet \
   --image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition" \
   --specialized
```


## <a name="next-steps"></a>Дальнейшие действия
С помощью [построителя образов Azure (Предварительная версия)](../virtual-machines/image-builder-overview.md) можно автоматизировать создание версий изображений, а также использовать его для обновления и [создания новой версии образа из существующей версии образа](../virtual-machines/linux/image-builder-gallery-update-image-version.md). 

Вы также можете создать ресурс коллекции общих образов с помощью шаблонов. Существует несколько шаблонов быстрого запуска Azure: 

- [Создание коллекции общих образов](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [Создание определения образа в коллекции общих образов](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [Создание версии образа в коллекции общих образов](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)