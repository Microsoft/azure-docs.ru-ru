---
title: Управление ресурсами, созданными во время процесса перемещения виртуальной машины в службе "перемещение ресурсов Azure"
description: Узнайте, как управлять ресурсами, созданными во время процесса перемещения виртуальной машины в службе "перемещение ресурсов Azure".
manager: evansma
author: rayne-wiselman
ms.service: resource-move
ms.topic: how-to
ms.date: 09/10/2020
ms.author: raynew
ms.openlocfilehash: d3c4c4e86e2461ea1d05af284e724a5a2991f040
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101727045"
---
# <a name="manage-resources-created-for-the-vm-move"></a>Управление ресурсами, созданными для перемещения виртуальной машины

В этой статье описывается, как управлять ресурсами, созданными явно с помощью [перемещения ресурсов Azure](overview.md) , чтобы упростить процесс перемещения виртуальной машины. 

После перемещения виртуальных машин в регионах существует ряд ресурсов, созданных с помощью перемещения ресурсов, которые следует очищать вручную.

## <a name="delete-resources-created-for-vm-move"></a>Удаление ресурсов, созданных для перемещения виртуальной машины

Вручную удалите коллекцию перемещения и Site Recovery ресурсы, созданные для перемещения виртуальной машины.

1. Изучите ресурсы в группе ресурсов ```ResourceMoverRG-<sourceregion>-<target-region>-<metadataRegionShortName>``` .
2. Убедитесь, что виртуальная машина и все другие исходные ресурсы в коллекции перемещения перемещены или удалены. Это гарантирует, что у вас отсутствуют ресурсы, ожидающие использования.
2. Удалите эти ресурсы.

    - Имя перемещаемой коллекции — ```movecollection-<sourceregion>-<target-region>-<metadata-region>```.
    - Имя учетной записи хранения — ```resmovecache<guid>```.
    - Имя хранилища — ```ResourceMove-<sourceregion>-<target-region>-GUID```.

## <a name="next-steps"></a>Дальнейшие действия

Попробуйте [переместить виртуальную машину](tutorial-move-region-virtual-machines.md) в другой регион с помощью перемещения ресурсов.
