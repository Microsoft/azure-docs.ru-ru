---
title: Виртуальные машины Azure серии HBv3
description: Спецификации виртуальных машин серии HBv3.
author: vermagit
ms.service: virtual-machines
ms.subservice: vm-sizes-hpc
ms.topic: conceptual
ms.date: 03/12/2021
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: a03eac3969e8918c8da20b90d0dcf8b60b39b8ee
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104800845"
---
# <a name="hbv3-series"></a>Серия HBv3

Виртуальные машины серии HBv3 оптимизированы для приложений HPC, таких как жидкие Dynamics, явный и неявный анализ конечных элементов, моделирование погоды, обработка пластового, моделирование молекулярного и имитация RTL. HBv3 VMS до 120 AMD ЕПИК™ 7003-Series (Милан) ядер ЦП, 448 ГБ ОЗУ и без технологии Hyper-Threading. Виртуальные машины серии HBv3 также предоставляют 350 ГБ/с для пропускной способности памяти, до 32 МБ кэш-памяти третьего уровня на ядро, до 7 ГБ/с для процессора блочного устройства и тактовые частоты до 3,675 ГГц. 

Все виртуальные машины серии HBv3 200 Гбит/с в HDR InfiniBand из сети NVIDIA позволяют выполнять крупномасштабные рабочие нагрузки MPI. Эти виртуальные машины подключены в неблокирующем дереве FAT для обеспечения оптимальной и стабильной производительности RDMA. Структура HDR InfiniBand также поддерживает адаптивную маршрутизацию и динамический подключенный транспорт (ДКТ, дополнительно к стандартным транспортам RC и обновления). Эти функции улучшают производительность, масштабируемость и согласованность приложений, и их использование настоятельно рекомендуется.

[Хранилище класса Premium](premium-storage-performance.md): поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): не поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): не поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколение 1 и 2<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): ожидается в ближайшее время<br>
[Временные диски ОС](ephemeral-os-disks.md): не поддерживаются <br>
<br>

|Размер |vCPU |Процессор |Память, ГиБ |Пропускная способность памяти ГБ/с |Базовая частота ЦП (ГГц) |Частота ядер (ГГц, пик) |Частота с одним ядром (ГГц, пик) |Производительность RDMA (ГБ/с) |Поддержка MPI |Временное хранилище, Гиб |Максимальное число дисков данных |Макс. vNIC Ethernet |
|----|----|----|----|----|----|----|----|----|----|----|----|----|
|Standard_HB120rs_v3    |120 |AMD ЕПИК 7V13 |448 |350 |2.45 |3.1 |3,675 |200 |Все |2 * 960 |32 |8 |
|Standard_HB120 96rs_v3 |96  |AMD ЕПИК 7V13 |448 |350 |2.45 |3.1 |3,675 |200 |Все |2 * 960 |32 |8 |
|Standard_HB120 64rs_v3 |64  |AMD ЕПИК 7V13 |448 |350 |2.45 |3.1 |3,675 |200 |Все |2 * 960 |32 |8 |
|Standard_HB120 32rs_v3 |32  |AMD ЕПИК 7V13 |448 |350 |2.45 |3.1 |3,675 |200 |Все |2 * 960 |32 |8 |
|Standard_HB120 16rs_v3 |16  |AMD ЕПИК 7V13 |448 |350 |2.45 |3.1 |3,675 |200 |Все |2 * 960 |32 |8 |

Дополнительные сведения о:
- [Архитектура и топология виртуальных машин](./workloads/hpc/hbv3-series-overview.md)
- Поддерживаемый [Стек программного обеспечения](./workloads/hpc/hbv3-series-overview.md#software-specifications) , включая ПОДДЕРЖИВАЕМую ОС
- Ожидаемая [производительность](./workloads/hpc/hbv3-performance.md) виртуальной машины серии HBv3

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
