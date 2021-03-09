---
title: Виртуальные машины Azure серии HC
description: Спецификации виртуальных машин серии HC.
author: vermagit
ms.service: virtual-machines
ms.subservice: hpc
ms.topic: conceptual
ms.date: 03/05/2021
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: 630d4ab1e20e92cc9dbe3881469df7f209078fba
ms.sourcegitcommit: 8d1b97c3777684bd98f2cfbc9d440b1299a02e8f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102485707"
---
# <a name="hc-series"></a>Серия HC

Виртуальные машины серии HC оптимизированы для приложений, управляемых несжатыми вычислениями, например неявным анализом конечных элементов, молекулярное Dynamics и вычислительными химия. HC VMS 44 процессорных ядер Intel Xeon Platinum 8168, 8 ГБ ОЗУ на ядро ЦП и без технологии hypering. Платформа Intel Xeon Platinum поддерживает обширную экосистему программных средств Intel, таких как библиотека Intel Math Kernel и расширенные возможности векторной обработки, такие как AVX-512.

Виртуальные машины серии HC 100 ГБ/с Mellanox ЕДР InfiniBand. Эти виртуальные машины подключены в неблокирующем дереве FAT для обеспечения оптимальной и стабильной производительности RDMA. Эти виртуальные машины поддерживают адаптивную маршрутизацию и динамический подключенный транспорт (ДКТ, дополнительно к стандартным транспортам RC и обновления). Эти функции улучшают производительность, масштабируемость и согласованность приложений, и их использование рекомендуется. 

[ACU](acu.md): 297-315<br>
[Хранилище класса Premium](premium-storage-performance.md): поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): не поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): не поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколение 1 и 2<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): поддерживается (дополнительные[сведения](https://techcommunity.microsoft.com/t5/azure-compute/accelerated-networking-on-hb-hc-hbv2-and-ndv2/ba-p/2067965) о производительности и потенциальных проблемах).<br>
[Временные диски ОС](ephemeral-os-disks.md): поддерживаются <br>

<br>

| Размер | vCPU | Процессор | Память, ГиБ | Пропускная способность памяти ГБ/с | Базовая частота ЦП (ГГц) | Частота ядер (ГГц, пик) | Частота с одним ядром (ГГц, пик) | Производительность RDMA (ГБ/с) | Поддержка MPI | Временное хранилище, Гиб | Максимальное число дисков данных | Макс. vNIC Ethernet |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HC44rs | 44 | Intel Xeon Platinum 8168 | 352 | 191 | 2.7 | 3.4 | 3,7 | 100 | Все | 700 | 4 | 8 |

Узнайте больше о базовой [архитектуре, топологии виртуальной машины](./workloads/hpc/hc-series-overview.md) и ожидаемой [производительности](./workloads/hpc/hc-series-performance.md) виртуальной машины серии HC.

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

- Ознакомьтесь с последними объявлениями, примерами рабочих нагрузок HPC и результатами производительности в [блогах сообщества Azure для разработчиков](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Высокоуровневое архитектурное представление выполнения рабочих нагрузок HPC см. [в статье высокопроизводительные вычисления (HPC) в Azure](/azure/architecture/topics/high-performance-computing/).
- Узнайте больше о том, как с помощью [единиц вычислений Azure (ACU)](acu.md) сравнить производительность вычислений для различных номеров SKU Azure.
