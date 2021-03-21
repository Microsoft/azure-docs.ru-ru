---
title: Создание группы размещения с учетом расположения с помощью Azure CLI
description: Узнайте, как создавать и использовать группы размещения близкого взаимодействия для виртуальных машин в Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: proximity-placement-groups
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 3/8/2021
ms.author: cynthn
ms.openlocfilehash: a347c9284608340811f9c2388df26129baeb8837
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102505653"
---
# <a name="deploy-vms-to-proximity-placement-groups-using-azure-cli"></a>Развертывание виртуальных машин в группах размещения близкого взаимодействия с помощью Azure CLI

Чтобы виртуальные машины максимально близки к максимально возможной задержке, следует развернуть их в [группе размещения](../co-location.md#proximity-placement-groups)с учетом расположения.

Группа размещения близкого взаимодействия — это логическая группировка, используемая для того, чтобы ресурсы вычислений Azure физически размещались близко друг к другу. Группа размещения близкого взаимодействия удобны для рабочих нагрузок, где требуется низкая задержка.


## <a name="create-the-proximity-placement-group"></a>Создание группы размещения с учетом расположения
Создайте группу размещения с учетом расположения с помощью [`az ppg create`](/cli/azure/ppg#az-ppg-create) . 

```azurecli-interactive
az group create --name myPPGGroup --location westus
az ppg create \
   -n myPPG \
   -g myPPGGroup \
   -l westus \
   -t standard 
```

## <a name="list-proximity-placement-groups"></a>Перечисление групп размещения близкого взаимодействия

Список всех групп размещения с учетом расположения можно вывести с помощью команды [AZ ППГ List](/cli/azure/ppg#az-ppg-list).

```azurecli-interactive
az ppg list -o table
```

## <a name="create-a-vm"></a>Создание виртуальной машины

Создайте виртуальную машину в группе размещения с учетом расположения, используя [новую виртуальную машину AZ](/cli/azure/vm#az-vm-create).

```azurecli-interactive
az vm create \
   -n myVM \
   -g myPPGGroup \
   --image UbuntuLTS \
   --ppg myPPG  \
   --generate-ssh-keys \
   --size Standard_D1_v2  \
   -l westus
```

Виртуальную машину можно просмотреть в группе размещения с помощью команды [AZ ППГ Показать](/cli/azure/ppg#az-ppg-show).

```azurecli-interactive
az ppg show --name myppg --resource-group myppggroup --query "virtualMachines"
```

## <a name="availability-sets"></a>Группы доступности
Вы также можете создать группу доступности в группе размещения с близкой назначением. Используйте тот же параметр, выполнив команду `--ppg` [AZ VM Availability-Set Create](/cli/azure/vm/availability-set#az-vm-availability-set-create) , чтобы создать группу доступности, а все виртуальные машины в группе доступности также будут созданы в той же группы размещения близости.

## <a name="scale-sets"></a>Масштабируемые наборы

Вы также можете создать масштабируемый набор в группе размещения с учетом расположения. Используйте тот же `--ppg` параметр команды [AZ vmss Create](/cli/azure/vmss#az_vmss_create) , чтобы создать масштабируемый набор, и все экземпляры будут созданы в той же группе размещения близости.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о командах [Azure CLI](/cli/azure/ppg) для групп размещения с учетом расположения.