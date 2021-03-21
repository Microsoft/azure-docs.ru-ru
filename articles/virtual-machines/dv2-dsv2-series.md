---
title: Виртуальные машины Azure серии Dv2 и DSv2
description: Спецификации для виртуальных машин серии Dv2 и Dsv2.
author: joelpelley
ms.service: virtual-machines
ms.subservice: vm-sizes-general
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: a3dee22a1ee793641e48be6edb1b33a00219217e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102560316"
---
# <a name="dv2-and-dsv2-series"></a>Серии Dv2 и DSv2

Серия Dv2 и DSv2, посвященная серии D, предоставляет более мощный процессор и оптимальную конфигурацию ЦП в памяти, что делает их пригодными для большинства рабочих нагрузок. Серия Dv2 примерно на 35 % быстрее, чем серия D. Серия Dv2 работает на базе Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2.1 ГГц (Skylake), Intel® Xeon® 2673-Broadwell v4 2,3 ГГц (2673) или Intel® Xeon® Haswell v3 2,4 ГГц () с 2,0 технологией Intel Turbo Boost. Серия Dv2 имеет такие же конфигурации памяти и диска, как и серия D.

## <a name="dv2-series"></a>Серия Dv2

Размеры серии Dv2 работают на Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2.1 ГГц (Skylake) или Intel® Xeon® 2673-Broadwell версии 2,3 4 ГГц (2673) или Intel® Xeon® Haswell v3 2,4 ГГц () с 2,0 технологией Intel Turbo Boost.

[ACU](acu.md): 210-250<br>
[Хранилище класса Premium](premium-storage-performance.md): не поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): не поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколение 1<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): поддерживается (*требуется не менее 4 виртуальных ЦП*)<br>
[Временные диски ОС](ephemeral-os-disks.md): не поддерживаются <br>
<br>

| Размер | vCPU | Память: ГиБ | Временное хранилище (SSD): ГиБ | Максимальная пропускная способность хранилища: операций ввода-вывода/чтения Мбит/с | Максимальное число дисков данных | Пропускная способность: операции ввода-вывода в секунду | Максимальное число сетевых адаптеров | Ожидаемая пропускная способность сети (Мбит/с) |
|---|---|---|---|---|---|---|---|---|
| Standard_D1_v2 | 1  | 3,5 | 50  | 3000/46/23    | 4  | 4x500  | 2|750   |
| Standard_D2_v2 | 2  | 7   | 100 | 6000/93/46    | 8  | 8x500  | 2|1500  |
| Standard_D3_v2 | 4  | 14  | 200 | 12000/187/93  | 16 | 16x500 | 4|3000  |
| Standard_D4_v2 | 8  | 28  | 400 | 24000/375/187 | 32 | 32x500 | 8|6000  |
| Standard_D5_v2 | 16 | 56  | 800 | 48000/750/375 | 64 | 64x500 | 8|12000 |

## <a name="dsv2-series"></a>Серия DSv2

Размеры серии DSv2 работают на процессорах Intel® Xeon® Platinum 8272CL (Cascade Lake), Intel® Xeon® 8171M 2.1 ГГц (Skylake) или Intel® Xeon® 2673-Broadwell версии 2,3 4 ГГц (2673) или Intel® Xeon® Haswell-v3 2,4 ГГц () с 2,0 технологией Intel Turbo Boost и используют хранилище класса Premium.

[ACU](acu.md): 210-250<br>
[Хранилище класса Premium](premium-storage-performance.md): поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколение 1 и 2<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): поддерживается (*требуется не менее 4 виртуальных ЦП*)<br>
[Временные диски ОС](ephemeral-os-disks.md): поддерживаются <br>
<br>

| Размер | vCPU | Память: ГиБ | Временное хранилище (SSD): ГиБ | Максимальное число дисков данных | Максимальная пропускная способность кэшированного и временного хранилища: операций ввода-вывода в секунду (размер кэша в гиб) | Максимальная пропускная способность дисков без кэширования: операций ввода-вывода в секунду / МБит/с | Максимальное число сетевых адаптеров|Ожидаемая пропускная способность сети (Мбит/с) |
|---|---|---|---|---|---|---|---|---|
| Standard_DS1_v2 | 1  | 3,5 | 7   | 4  | 4000/32 (43)    | 3200/48   | 2|750   |
| Standard_DS2_v2 | 2  | 7   | 14  | 8  | 8000/64 (86)    | 6400/96   | 2|1500  |
| Standard_DS3_v2 | 4  | 14  | 28  | 16 | 16000/128 (172) | 12800/192 | 4|3000  |
| Standard_DS4_v2 | 8  | 28  | 56  | 32 | 32000/256 (344) | 25600/384 | 8|6000  |
| Standard_DS5_v2 | 16 | 56  | 112 | 64 | 64000/512 (688) | 51200/768 | 8|12000 |

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes-and-information"></a>Другие размеры и сведения

- [Универсальные](sizes-general.md)
- [Оптимизированные для памяти](sizes-memory.md)
- [Оптимизированные для хранилища](sizes-storage.md)
- [Оптимизированные для GPU](sizes-gpu.md)
- [Для высокопроизводительных вычислений](sizes-hpc.md)
- [Предыдущие поколения](sizes-previous-gen.md)

Калькулятор цен: [Калькулятор цен](https://azure.microsoft.com/pricing/calculator/)

Дополнительные сведения о типах дисков: [типы дисков](./disks-types.md#ultra-disk)

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о том, как с помощью [единиц вычислений Azure (ACU)](acu.md) сравнить производительность вычислений для различных номеров SKU Azure.
