---
title: Виртуальные машины Azure серии H
description: Спецификации для виртуальных машин серии H.
author: vermagit
ms.service: virtual-machines
ms.subservice: vm-sizes-hpc
ms.topic: conceptual
ms.date: 10/09/2020
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: b3e3beb51256bbf22d29d74b51c52ca3a6bee0c4
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104774785"
---
# <a name="h-series"></a>Серия H

Виртуальные машины серии H оптимизированы для приложений, которые управляются высокой частотой ЦП или большими объемами памяти на базовые требования. Виртуальные машины серии H, компоненты 8 или 16 процессоров Intel Xeon 3 2667 v3, до 14 ГБ ОЗУ на ядро ЦП и без технологии Hyper-Threading. Функции серии H 56 ГБ/с Mellanox FDR InfiniBand в неблокирующей конфигурации дерева FAT для обеспечения постоянной производительности RDMA. В настоящее время на виртуальных машинах серии H не поддерживается SR-IOV, поддерживаются Intel MPI 5. x и MS-MPI.

[ACU](acu.md): 290-300<br>
[Хранилище класса Premium](premium-storage-performance.md): не поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): не поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): не поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): не поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколение 1<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): не поддерживается<br>
[Временные диски ОС](ephemeral-os-disks.md): не поддерживаются <br>
<br>

| Размер | vCPU | Процессор | Память, ГиБ | Пропускная способность памяти ГБ/с | Базовая частота ЦП (ГГц) | Частота ядер (ГГц, пик) | Частота с одним ядром (ГГц, пик) | Производительность RDMA (ГБ/с) | Поддержка MPI | Временное хранилище, Гиб | Максимальное число дисков данных | Максимальная пропускная способность дисков, операций ввода-вывода в секунду | Макс. vNIC Ethernet |
| --- | --- |--- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_H8   | 8  | Intel Xeon, версия 3, 2667 v3 | 56 | 40 | 3.2 | 3.3 | 3,6 | - | Intel 5. x, MS-MPI | 1000 | 32 | 32x500 | 2 |
| Standard_H16  | 16 | Intel Xeon, версия 3, 2667 v3 | 112 | 80 | 3.2 | 3.3 | 3,6 | - | Intel 5. x, MS-MPI | 2000 | 64 | 64x500 | 4 |
| Standard_H8m  | 8  | Intel Xeon, версия 3, 2667 v3 | 112 | 40 | 3.2 | 3.3 | 3,6 | - | Intel 5. x, MS-MPI | 1000 | 32 | 32x500 | 2 |
| Standard_H16m | 16 | Intel Xeon, версия 3, 2667 v3 | 224 | 80 | 3.2 | 3.3 | 3,6 | - | Intel 5. x, MS-MPI | 2000 | 64 | 64x500 | 4 |
| Standard_H16r <sup>1</sup>  | 16 | Intel Xeon, версия 3, 2667 v3 | 112 | 80 | 3.2 | 3.3 | 3,6 | 56 | Intel 5. x, MS-MPI | 2000 | 64 | 64x500 | 4 |
| Standard_H16mr <sup>1</sup> | 16 | Intel Xeon, версия 3, 2667 v3 | 224 | 80 | 3.2 | 3.3 | 3,6 | 56 | Intel 5. x, MS-MPI | 2000 | 64 | 64x500 | 4 |

<sup>1</sup> для приложений MPI сеть с выделенной внутренней сетью RDMA включена в сети FDR InfiniBand.

> [!NOTE]
> Среди [виртуальных машин с поддержкой RDMA](sizes-hpc.md#rdma-capable-instances)для серии H не включена поддержка SR-IOV. Поэтому поддерживаемые [образы виртуальных машин](./workloads/hpc/configure.md#vm-images), требования к [драйверу InfiniBand](./workloads/hpc/enable-infiniband.md) и поддерживаемые [библиотеки MPI](./workloads/hpc/setup-mpi.md) ОТЛИЧАЮТСЯ от ВИРТУАЛЬНЫХ машин с поддержкой SR-IOV.

## <a name="software-specifications"></a>Спецификации программного обеспечения

| Спецификации программного обеспечения     |Виртуальная машина серии HC           |
|-----------------------------|-----------------------|
| Максимальный размер задания MPI            | 4800 ядер (300 ВМ в одном масштабируемом наборе виртуальных машин с singlePlacementGroup = true)  |
| Поддержка MPI                 | Intel MPI 5. x, MS-MPI  |
| Поддержка операционных систем для SRIOV RDMA   | CentOS/RHEL 6,5-7,4, SLES 12 SP4 +, WinServer 2012 — 2016  |
| Поддержка Orchestrator        | Циклеклауд, Batch, AKS  |

[!INCLUDE [hpc-include](./workloads/hpc/includes/hpc-include.md)]

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes"></a>Остальные размеры

- [Универсальные](sizes-general.md)
- [Оптимизированные для памяти](sizes-memory.md)
- [Оптимизированные для хранилища](sizes-storage.md)
- [Оптимизированные для GPU](sizes-gpu.md)
- [Для высокопроизводительных вычислений](sizes-hpc.md)
- [Предыдущие поколения](sizes-previous-gen.md)

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с последними объявлениями, примерами рабочей нагрузки HPC и результатами производительности в [блогах сообщества разработчиков Azure](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Сведения о более высоком уровне архитектурного представления выполнения рабочих нагрузок HPC см. в статье [Высокопроизводительные вычисления (HPC) в Azure](/azure/architecture/topics/high-performance-computing/).
- Узнайте больше о том, как с помощью [единиц вычислений Azure (ACU)](acu.md) сравнить производительность вычислений для различных номеров SKU Azure.
