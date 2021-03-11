---
title: Перечисление, обновление и удаление ресурсов изображений с помощью PowerShell
description: Перечисление, обновление и удаление ресурсов изображений в коллекции общих образов с помощью Azure PowerShell.
author: cynthn
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 04/27/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: bde11f57152b7fd72ce08be54b616bbe428fa167
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102553601"
---
# <a name="list-update-and-delete-image-resources-using-powershell"></a>Перечисление, обновление и удаление ресурсов изображений с помощью PowerShell 

Вы можете управлять ресурсами коллекции общих образов с помощью Azure PowerShell.

[!INCLUDE [virtual-machines-common-gallery-list-ps.md](../../includes/virtual-machines-common-gallery-list-ps.md)]

[!INCLUDE [virtual-machines-common-shared-images-update-delete-ps](../../includes/virtual-machines-common-shared-images-update-delete-ps.md)]

## <a name="next-steps"></a>Дальнейшие действия

С помощью [построителя образов Azure (Предварительная версия)](./image-builder-overview.md) можно автоматизировать создание версий изображений, а также использовать его для обновления и [создания новой версии образа из существующей версии образа](./linux/image-builder-gallery-update-image-version.md).