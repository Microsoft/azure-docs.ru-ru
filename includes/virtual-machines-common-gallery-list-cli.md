---
title: Включить файл
description: Включить файл
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 01/28/2020
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 1c06f5ab8995e7285365fa2d0ee77c327be2b1bd
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107386805"
---
## <a name="list-information"></a>Просмотр сведений

Чтобы просмотреть сведения о расположении, состоянии и другие данные о доступных коллекциях образов, выполните команду [az sig list](/cli/azure/sig#az-sig-list).

```azurecli-interactive 
az sig list -o table
```

Чтобы просмотреть определения образов в коллекции, в том числе сведения о состоянии и типе ОС, выполните команду [az sig image-definition list](/cli/azure/sig/image-definition#az-sig-image-definition-list).

```azurecli-interactive 
az sig image-definition list --resource-group myGalleryRG --gallery-name myGallery -o table
```

Чтобы просмотреть версии общих образов в коллекции, выполните команду [az sig image-version list](/cli/azure/sig/image-version#az-sig-image-version-list).

```azurecli-interactive
az sig image-version list --resource-group myGalleryRG --gallery-name myGallery --gallery-image-definition myImageDefinition -o table
```

Чтобы получить идентификатор версии образа, выполните команду [az sig image-version show](/cli/azure/sig/image-version#az-sig-image-version-show).

```azurecli-interactive
az sig image-version show \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --query "id"
```
