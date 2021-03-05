---
title: Виртуальные машины Azure серии HBv2
description: Спецификации виртуальных машин серии HBv2.
author: vermagit
ms.service: virtual-machines
ms.subservice: hpc
ms.topic: conceptual
ms.date: 10/09/2020
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: d97857e2a1ffdcb4701a7ab17866a2ab1f699d4d
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102203510"
---
# <a name="hbv2-series"></a>Серия HBv2

Виртуальные машины серии HBv2 оптимизированы для приложений, управляемых с помощью пропускной способности памяти, например "жидкие Dynamics", "анализ с конечными элементами" и "моделирование молекулярного". HBv2 VMS 120 ядра процессора AMD ЕПИК 7742, 4 ГБ ОЗУ на ядро ЦП и без одновременной многопоточности. Каждая виртуальная машина HBv2 предоставляет до 340 ГБ/с для пропускной способности памяти и до 4 операций FP64 вычислений.

Виртуальные машины серии HBv2 200 ГБ/с Mellanox HDR InfiniBand. Эти виртуальные машины подключены в неблокирующем дереве FAT для обеспечения оптимальной и стабильной производительности RDMA. Эти виртуальные машины поддерживают адаптивную маршрутизацию и динамический подключенный транспорт (ДКТ, дополнительно к стандартным транспортам RC и обновления). Эти функции улучшают производительность, масштабируемость и согласованность приложений, а их использование настоятельно рекомендуется.

[Хранилище класса Premium](premium-storage-performance.md): поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): не поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): не поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколение 1<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): поддерживается (дополнительные[сведения](https://techcommunity.microsoft.com/t5/azure-compute/accelerated-networking-on-hb-hc-hbv2-and-ndv2/ba-p/2067965) о производительности и потенциальных проблемах). <br>
[Временные диски ОС](ephemeral-os-disks.md): не поддерживаются <br>
<br>

| Размер | vCPU | Процессор | Память, ГиБ | Пропускная способность памяти ГБ/с | Базовая частота ЦП (ГГц) | Частота ядер (ГГц, пик) | Частота с одним ядром (ГГц, пик) | Производительность RDMA (ГБ/с) | Поддержка MPI | Временное хранилище, Гиб | Максимальное число дисков данных | Макс. vNIC Ethernet |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HB120rs_v2 | 120 | AMD ЕПИК 7V12 | 480 | 350 | 2.45 | 3.1 | 3.3 | 200 | Все | 480 + 960 | 8 | 8 |


[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

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
