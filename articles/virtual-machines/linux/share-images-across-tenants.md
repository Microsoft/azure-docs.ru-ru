---
title: Совместное использование образов коллекций в клиентах
description: Узнайте, как совместно использовать образы виртуальных машин в клиентах Azure с помощью общих коллекций образов, использующих Azure CLI.
author: axayjo
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/04/2019
ms.author: akjosh
ms.reviewer: cynthn
ms.custom: devx-track-azurecli
ms.openlocfilehash: 2ef788659c871a9bcef6f519664689eacda94daa
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102552887"
---
# <a name="share-gallery-vm-images-across-azure-tenants-using-the-azure-cli"></a>Совместное использование образов виртуальных машин коллекции в клиентах Azure с помощью Azure CLI

Коллекции общих образов позволяют предоставлять общий доступ к образам с помощью Azure RBAC. Вы можете использовать Azure RBAC для совместного использования образов в вашем клиенте и даже для пользователей за пределами клиента. Дополнительные сведения об этом простом параметре общего доступа см. в разделе [общий доступ к коллекции](./shared-images-portal.md#share-the-gallery).

[!INCLUDE [virtual-machines-share-images-across-tenants](../../../includes/virtual-machines-share-images-across-tenants.md)]

> [!IMPORTANT]
> Вы не можете использовать портал для развертывания виртуальной машины из образа в другом клиенте Azure. Чтобы создать виртуальную машину из образа, совместно используемого клиентами, необходимо использовать Azure CLI или [PowerShell](../windows/share-images-across-tenants.md).

## <a name="create-a-vm-using-azure-cli"></a>Создание виртуальной машины с помощью Azure CLI

Войдите в субъект-службу для клиента 1, используя appID, ключ приложения и идентификатор клиента 1. При необходимости можно использовать `az account show --query "tenantId"` для получения идентификаторов клиентов.

```azurecli-interactive
az account clear
az login --service-principal -u '<app ID>' -p '<Secret>' --tenant '<tenant 1 ID>'
az account get-access-token 
```
 
Войдите в субъект-службу для клиента 2, используя appID, ключ приложения и идентификатор клиента 2:

```azurecli-interactive
az login --service-principal -u '<app ID>' -p '<Secret>' --tenant '<tenant 2 ID>'
az account get-access-token
```

Создайте виртуальную машину. Замените сведения в примере собственными.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image "/subscriptions/<Tenant 1 subscription>/resourceGroups/<Resource group>/providers/Microsoft.Compute/galleries/<Gallery>/images/<Image definition>/versions/<version>" \
  --admin-username azureuser \
  --generate-ssh-keys
```

## <a name="next-steps"></a>Дальнейшие действия

Если вы столкнетесь с проблемами, обратитесь к статье об [устранении неполадок с коллекциями общих образов](../troubleshooting-shared-images.md).
