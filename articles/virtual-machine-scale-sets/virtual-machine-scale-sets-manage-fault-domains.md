---
title: Управление доменами сбоя в масштабируемых наборах виртуальных машин Azure
description: Узнайте, как выбрать нужное число доменов сбоя при создании масштабируемого набора виртуальных машин.
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: availability
ms.date: 12/18/2018
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: 5a71a6bce6d0e1a41201e0d7395110a6ac64db8c
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102209753"
---
# <a name="choosing-the-right-number-of-fault-domains-for-virtual-machine-scale-set"></a>Выбор нужного числа доменов сбоя для масштабируемого набора виртуальных машин
Масштабируемые наборы виртуальных машин создаются с пятью доменами сбоя по умолчанию в регионах Azure без зон. Для регионов, поддерживающих зональные развертывание масштабируемых наборов виртуальных машин и установлен этот параметр, значение по умолчанию для параметра количество доменов сбоя равно 1 для каждой из зон. FD = 1 в данном случае означает, что экземпляры виртуальных машин, принадлежащих к масштабируемому набору, будут распределены по многим стойкам по мере возможностей.

Можно также уравнять количество доменов сбоя масштабируемого набора с количеством доменов сбоя управляемых дисков. Такое выравнивание поможет предотвратить потерю кворума в том случае, если весь домен сбоя управляемых дисков выйдет из строя. Количество доменов сбоя может быть меньше количества доменов сбоя управляемых дисков, доступных в каждом регионе, или равно этому количеству. Ознакомьтесь с этим [документом](../virtual-machines/manage-availability.md), чтобы узнать больше о количестве доменов сбоя управляемых дисков по регионам.

## <a name="rest-api"></a>REST API
Можно задать для свойства значение `properties.platformFaultDomainCount` 1, 2 или 3 (значение по умолчанию — 3, если не указано). См. документацию по REST API [здесь](/rest/api/compute/virtualmachinescalesets/createorupdate).

## <a name="azure-cli"></a>Azure CLI
Параметру можно присвоить значение `--platform-fault-domain-count` 1, 2 или 3 (значение по умолчанию — 3, если не указано). См. документацию по Azure CLI [здесь](/cli/azure/vmss#az-vmss-create).

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --platform-fault-domain-count 3\
  --generate-ssh-keys
```

Создание и настройка всех ресурсов и виртуальных машин масштабируемого набора занимает несколько минут.

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о [функциях доступности и избыточности](../virtual-machines/availability.md) для сред Azure.
