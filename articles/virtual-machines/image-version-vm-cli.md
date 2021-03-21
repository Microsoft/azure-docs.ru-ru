---
title: Создание образа из виртуальной машины с помощью Azure CLI
description: Узнайте, как создать образ в общей коллекции образов на основе виртуальной машины в Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/01/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7c35be8821b6763531b43ec85b10325e91f8bc5f
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102556866"
---
# <a name="create-an-image-version-from-a-vm-in-azure-using-the-azure-cli"></a>Создание версии образа из виртуальной машины в Azure с помощью Azure CLI

Если у вас есть виртуальная машина, которую вы хотите использовать для создания нескольких идентичных виртуальных машин, можно использовать эту виртуальную машину, чтобы создать образ в общей коллекции образов с помощью Azure CLI. Вы также можете создать образ из виртуальной машины с помощью [Azure PowerShell](image-version-vm-powershell.md).

**Версия образа** — это то, что используется для создания виртуальной машины при использовании общей коллекции образов. В зависимости от требований для вашей среды, у вас может быть несколько версий образа. При использовании версии образа для создания виртуальной машины используется версия образа для создания дисков для новой виртуальной машины. Версии образов можно использовать несколько раз.


## <a name="before-you-begin"></a>Перед началом

Для работы с этой статьей необходимо иметь существующую коллекцию общих образов. 

У вас также должна быть существующая виртуальная машина в Azure в том же регионе, что и коллекция. 

Если к виртуальной машине подключен диск данных, размер диска данных не может превышать 1 ТБ.

При работе с этой статьей при необходимости замените имена ресурсов.

## <a name="get-information-about-the-vm"></a>Получение информации о виртуальной машине

Список доступных виртуальных машин можно просмотреть с помощью команды [az vm list](/cli/azure/vm#az-vm-list). 

```azurecli-interactive
az vm list --output table
```

Когда вы узнаете имя виртуальной машины и группу ресурсов, в которой она находится, получите идентификатор виртуальной машины с помощью команды [az vm get-instance-view](/cli/azure/vm#az-vm-get-instance-view). 

```azurecli-interactive
az vm get-instance-view -g MyResourceGroup -n MyVm --query id
```


## <a name="create-an-image-definition"></a>Создание определения образа

Образы можно объединять в логические группы с помощью определений образов. Определения образов используются для управления сведениями о версиях созданных в них образов. 

В имени определения образа можно использовать прописные и строчные буквы, цифры, точки и дефисы. 

Убедитесь, что используется определение образа подходящего типа. Если вы сделали виртуальную машину универсальной (с помощью Sysprep для Windows или waagent -deprovision для Linux), создайте универсальное определение образа с помощью `--os-state generalized`. Если вы хотите использовать виртуальную машину без удаления существующих учетных записей пользователей, создайте специализированное определение образа с помощью `--os-state specialized`.

Дополнительные сведения о значениях, которые можно указать для определения образа, см. в разделе [Определения образов](./shared-image-galleries.md#image-definitions).

Создайте в коллекции определение образа, используя команду [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create).

В этом примере определение образа имеет имя *myImageDefinition* и предназначено для [специализированного](./shared-image-galleries.md#generalized-and-specialized-images) образа ОС Linux. Чтобы создать определение для образов с помощью ОС Windows, используйте `--os-type Windows`. 

```azurecli-interactive 
az sig image-definition create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --publisher myPublisher \
   --offer myOffer \
   --sku mySKU \
   --os-type Linux \
   --os-state specialized
```


## <a name="create-the-image-version"></a>Создание версии образа

Создайте версию образа на основе виртуальной машины, используя команду [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create).  

Допустимыми знаками для имени версии образа являются цифры и точки. Числа должны быть в диапазоне 32-битного целого числа. Формат: *основной номер версии*.*дополнительный номер версии*.*исправление*.

В нашем примере для образа указана версия *1.0.0*, и мы собираемся создать 2 реплики в регионе *центрально-западная часть США*, 1 реплику в регионе *центрально-южная часть США* и 1 реплику в регионе *восточная часть США 2* для создания хранилища, избыточного между зонами. В числе регионов репликации должен быть регион, в котором находится исходная виртуальная машина.

Замените значение параметра `--managed-image` в этом примере на идентификатор вашей виртуальной машины из предыдущего шага.

```azurecli-interactive 
az sig image-version create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --target-regions "westcentralus" "southcentralus=1" "eastus=1=standard_zrs" \
   --replica-count 2 \
   --managed-image "/subscriptions/<Subscription ID>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM"
```

> [!NOTE]
> Прежде чем использовать тот же управляемый образ для создания другой версии образа, необходимо дождаться завершения сборки и репликации версии образа.
>
> Вы также можете сохранить образ в хранилище класса Premium, добавив `--storage-account-type  premium_lrs` [хранилище или избыточное в зону](../storage/common/storage-redundancy.md) , добавив `--storage-account-type  standard_zrs` его при создании версии образа.
>

## <a name="next-steps"></a>Дальнейшие действия

Создайте виртуальную машину из [обобщенного образа](vm-generalized-image-version-cli.md) , используя Azure CLI.

Сведения о том, как предоставить сведения о плане покупки, см. в разделе [предоставление сведений о плане покупки Azure Marketplace при создании образов](marketplace-images.md).