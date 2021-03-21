---
title: Виртуальные машины Azure серии HBv2
description: Спецификации виртуальных машин серии HBv2.
author: vermagit
ms.service: virtual-machines
ms.subservice: vm-sizes-hpc
ms.topic: conceptual
ms.date: 03/08/2021
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: 1abc05cf1486651b87094f40777f3679d234a34b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103009343"
---
# <a name="hbv2-series"></a>Серия HBv2

Виртуальные машины серии HBv2 оптимизированы для приложений, которые управляются пропускной способностью памяти, например "жидкие Dynamics", "неограниченный" анализ элементов и моделирования молекулярного. HBv2 VMS 120 ядра процессора AMD ЕПИК 7742, 4 ГБ ОЗУ на ядро ЦП и без одновременной многопоточности. Каждая виртуальная машина HBv2 предоставляет до 340 ГБ/с для пропускной способности памяти и до 4 операций FP64 вычислений.

Виртуальные машины серии HBv2 200 ГБ/с Mellanox HDR InfiniBand. Эти виртуальные машины подключены в неблокирующем дереве FAT для обеспечения оптимальной и стабильной производительности RDMA. Эти виртуальные машины поддерживают адаптивную маршрутизацию и динамический подключенный транспорт (ДКТ в дополнение к стандартным транспортам RC и обновления). Эти функции улучшают производительность, масштабируемость и согласованность приложений, и их использование рекомендуется.

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
| Standard_HB120rs_v2 | 120 | AMD ЕПИК 7V12 | 456 | 350 | 2.45 | 3.1 | 3.3 | 200 | Все | 480 + 960 | 8 | 8 |

Дополнительные сведения
- Базовая [архитектура и топология виртуальной машины](./workloads/hpc/hbv2-series-overview.md)
- [Поддерживаемый стек программного обеспечения](./workloads/hpc/hbv2-series-overview.md#software-specifications) , включая ПОДДЕРЖИВАЕМую ОС
- Ожидаемая [производительность](./workloads/hpc/hbv2-performance.md) виртуальной машины серии HBv2.

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
