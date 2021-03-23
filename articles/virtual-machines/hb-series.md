---
title: Серия HB
description: Спецификации виртуальных машин серии ХБ.
author: vermagit
ms.service: virtual-machines
ms.subservice: vm-sizes-hpc
ms.topic: conceptual
ms.date: 03/19/2021
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: f3e6f40833f3536bb915af74b7e0d80143fe0d13
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104774802"
---
# <a name="hb-series"></a>Серия HB

Виртуальные машины серии ХБ оптимизированы для приложений, которые управляются пропускной способностью памяти, например жидкие Dynamics, явный конечный анализ элементов и моделирование погоды. ХБ VMS 60 ядра процессора AMD ЕПИК 7551, 4 ГБ ОЗУ на ядро ЦП и без одновременной многопоточности. Виртуальная машина ХБ предоставляет до 260 ГБ/с для пропускной способности памяти.

Виртуальные машины серии ХБ 100 ГБ/с Mellanox ЕДР InfiniBand. Эти виртуальные машины подключены в неблокирующем дереве FAT для обеспечения оптимальной и стабильной производительности RDMA. Эти виртуальные машины поддерживают адаптивную маршрутизацию и динамический подключенный транспорт (ДКТ в дополнение к стандартным транспортам RC и обновления). Эти функции улучшают производительность, масштабируемость и согласованность приложений, и их использование рекомендуется.

[ACU](acu.md): 199-216<br>
[Хранилище класса Premium](premium-storage-performance.md): поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): не поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): не поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколение 1 и 2<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): поддерживается (дополнительные[сведения](https://techcommunity.microsoft.com/t5/azure-compute/accelerated-networking-on-hb-hc-hbv2-and-ndv2/ba-p/2067965) о производительности и потенциальных проблемах). <br>
[Временные диски ОС](ephemeral-os-disks.md): не поддерживаются <br>
<br>

| Размер | vCPU | Процессор | Память, ГиБ | Пропускная способность памяти ГБ/с | Базовая частота ЦП (ГГц) | Частота ядер (ГГц, пик) | Частота с одним ядром (ГГц, пик) | Производительность RDMA (ГБ/с) | Поддержка MPI | Временное хранилище, Гиб | Максимальное число дисков данных | Макс. vNIC Ethernet |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HB60rs | 60 | AMD ЕПИК 7551 | 228 | 263 | 2.0 | 2.55 | 2.55 | 100 | Все | 700 | 4 | 8 |

Дополнительные сведения о:
- [архитектура и топология виртуальных машин](./workloads/hpc/hb-series-overview.md)
- поддерживаемый [Стек программного обеспечения](./workloads/hpc/hb-series-overview.md#software-specifications) , включая поддерживаемую ОС, и
- Ожидаемая [производительность](./workloads/hpc/hb-series-performance.md) виртуальной машины серии ХБ.

[!INCLUDE [hpc-include.md](./workloads/hpc/includes/hpc-include.md)]

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
