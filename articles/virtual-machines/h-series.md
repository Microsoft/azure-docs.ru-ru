---
title: Виртуальные машины Azure серии H
description: Спецификации для виртуальных машин серии H.
author: ju-shim
ms.service: virtual-machines
ms.subservice: vm-sizes-hpc
ms.topic: conceptual
ms.date: 10/09/2020
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: 7a5ff47bb5de6d54564a86c0700ad69aa450887d
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102566147"
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

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

> [!NOTE]
> Среди [виртуальных машин, поддерживающих RDMA](sizes-hpc.md#rdma-capable-instances), Серия H не поддерживает SR-IOV. Поэтому поддерживаемые [образы виртуальных машин](./workloads/hpc/configure.md#vm-images), требования к [драйверу InfiniBand](./workloads/hpc/enable-infiniband.md) и поддерживаемые [библиотеки MPI](./workloads/hpc/setup-mpi.md) ОТЛИЧАЮТСЯ от ВИРТУАЛЬНЫХ машин с поддержкой SR-IOV.

## <a name="other-sizes"></a>Остальные размеры

- [Универсальные](sizes-general.md)
- [Оптимизированные для памяти](sizes-memory.md)
- [Оптимизированные для хранилища](sizes-storage.md)
- [Оптимизированные для GPU](sizes-gpu.md)
- [Для высокопроизводительных вычислений](sizes-hpc.md)
- [Предыдущие поколения](sizes-previous-gen.md)

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о [настройке виртуальных машин](./workloads/hpc/configure.md), [включении INFINIBAND](./workloads/hpc/enable-infiniband.md), [настройке MPI](./workloads/hpc/setup-mpi.md) и оптимизации приложений HPC для [рабочих нагрузок](./workloads/hpc/overview.md)Azure в HPC.
- Ознакомьтесь с последними объявлениями и некоторыми примерами HPC, а также результатами в [блогах технического сообщества службы вычислений](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Сведения о более высоком уровне архитектурного представления выполнения рабочих нагрузок HPC см. в статье [Высокопроизводительные вычисления (HPC) в Azure](/azure/architecture/topics/high-performance-computing/).
- Узнайте больше о том, как с помощью [единиц вычислений Azure (ACU)](acu.md) сравнить производительность вычислений для различных номеров SKU Azure.
