---
title: Как сопоставлять диски Azure с гостевыми дисками виртуальной машины Linux
description: Как определить диски Azure, ундерлай гостевые диски виртуальной машины Linux.
author: timbasham
ms.service: virtual-machines
ms.subservice: disks
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/17/2020
ms.author: tibasham
ms.collection: linux
ms.openlocfilehash: bc6c6273ab3d1a4403763e4ed0a8c491995fb2df
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102556729"
---
# <a name="how-to-map-azure-disks-to-linux-vm-guest-disks"></a>Как сопоставлять диски Azure с гостевыми дисками виртуальной машины Linux

Возможно, вам потребуется определить диски Azure, которые будут выполнять резервное копирование гостевых дисков виртуальной машины. В некоторых случаях можно сравнить размер диска или тома с размером подключенных дисков Azure. В сценариях, где к виртуальной машине подключено несколько дисков Azure одного размера, необходимо использовать логический номер устройства (LUN) дисков данных. 

## <a name="what-is-a-lun"></a>Что такое LUN?

Логический номер устройства (LUN) — это число, которое используется для обнаружения определенного устройства хранения. Каждому устройству хранения назначается уникальный числовой идентификатор, начинающийся с нуля. Полный путь к устройству представлен номером шины, ИДЕНТИФИКАТОРом целевого объекта и логическим номером устройства (LUN). 

Например: ***номер шины 0, идентификатор целевого объекта 0, LUN 3***

Для нашего упражнения необходимо использовать только LUN.

## <a name="finding-the-lun"></a>Поиск LUN

Ниже приведены два метода поиска LUN диска в Linux.

### <a name="lsscsi"></a>lsscsi

1. Подключение к виртуальной машине
1. `sudo lsscsi`

Первый столбец в списке будет содержать LUN, формат [узел: канал: Цель:**LUN**].

### <a name="listing-block-devices"></a>Перечисление блочных устройств

1. Подключение к виртуальной машине
1. `sudo ls -l /sys/block/*/device`

Последний столбец в списке будет содержать LUN, формат [узел: канал: целевой:**LUN**]

## <a name="finding-the-lun-for-the-azure-disks"></a>Поиск LUN для дисков Azure

Номер LUN для диска Azure можно узнать с помощью портал Azure Azure CLI.

### <a name="finding-an-azure-disks-lun-in-the-azure-portal"></a>Поиск LUN диска Azure в портал Azure

1. В портал Azure выберите "виртуальные машины", чтобы отобразить список виртуальных машин.
1. Выберите виртуальную машину.
1. Выберите "диски"
1. Выберите диск данных из списка подключенных дисков.
1. LUN диска будет отображаться в области сведений о диске. Показанный здесь LUN сопоставляется с номерами LUN, которые вы искали в гостевой системе с помощью lsscsi, или перечисление блочных устройств.

### <a name="finding-an-azure-disks-lun-using-azure-cli"></a>Поиск LUN диска Azure с помощью Azure CLI

```azurecli-interactive
az vm show -g myResourceGroup -n myVM --query "storageProfile.dataDisks"
```
