---
title: Серии Edv4 и Edsv4
description: Спецификации для виртуальных машин серий Ev4, Edv4, Esv4 и Edsv4.
author: brbell
ms.author: brbell
ms.reviewer: jushiman
ms.custom: mimckitt
ms.service: virtual-machines
ms.subservice: vm-sizes-memory
ms.topic: conceptual
ms.date: 02/04/2020
ms.openlocfilehash: d6edee1f349b37c274f71c9db7abac35f9c88c4a
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102557120"
---
# <a name="edv4-and-edsv4-series"></a>Серии Edv4 и Edsv4

Серии Edv4 и Edsv4 работают на процессорах Intel&reg; Xeon&reg; Platinum 8272CL (Cascade Lake) с технологией Hyper-Threading и отлично подходят для любых корпоративных приложений, требующих больших объемов памяти, предоставляя до 504 ГиБ ОЗУ, технологию [Intel&reg; Turbo Boost 2.0](https://www.intel.com/content/www/us/en/architecture-and-technology/turbo-boost/turbo-boost-technology.html), [Intel&reg; Hyper-Threading](https://www.intel.com/content/www/us/en/architecture-and-technology/hyper-threading/hyper-threading-technology.html) и расширения [Intel&reg; Advanced Vector Extensions 512 (Intel&reg; AVX-512)](https://www.intel.com/content/www/us/en/architecture-and-technology/avx-512-overview.html). Они также поддерживают [ускорение Intel для &reg; глубокого обучения](https://software.intel.com/content/www/us/en/develop/topics/ai/deep-learning-boost.html). Новые размеры виртуальных машин будут иметь 50% больше локального хранилища, а также дополнительные операции ввода-вывода на локальный диск для чтения и записи по сравнению с размером [Ev3/Esv3](./ev3-esv3-series.md) с [виртуальными машинами Gen2](./generation-2.md). Он включает всю частоту ядра Turbo 3,4 ГГц. 

## <a name="edv4-series"></a>Серия Edv4

Размеры серии Edv4 выполняются на процессорах Intel&reg; Xeon&reg; Platinum 8272CL (Cascade Lake). Виртуальные машины размеров Edv4 оснащены до 504 ГиБ ОЗУ, а также быстрым локальным диском SSD большого объема (до 2400 ГиБ). Эти виртуальные машины идеально подходят для корпоративных приложений с высокими требованиями к объему памяти и приложений, для которых важна низкая задержка и высокая скорость при работе с локальным хранилищем. К виртуальным машинам Edv4 можно подключать дисковые накопители SSD (цен. категория "Стандартный") и HDD (цен. категория "Стандартный"). 

[ACU](acu.md): 195-210<br>
[Хранилище класса Premium](premium-storage-performance.md): не поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): не поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколение 1 и 2<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): поддерживается (*требуется не менее 4 виртуальных ЦП*)<br>
[Временные диски ОС](ephemeral-os-disks.md): не поддерживаются <br>
<br>

| Размер | vCPU | Память: ГиБ | Временное хранилище (SSD): ГиБ | Максимальное число дисков данных | <sup>**</sup> Максимальная пропускная способность кэшированного и временного хранилища: операций ввода-вывода в секунду | Максимальное число сетевых адаптеров|Ожидаемая пропускная способность сети (Мбит/с) |
|---|---|---|---|---|---|---|---|
| Standard_E2d_v4  | 2 | 16 | 75 | 4 | 19000/120 | 2|1000 |
| Standard_E4d_v4  | 4 | 32 | 150 | 8 | 38500/242 | 2|2000 |
| Standard_E8d_v4 | 8 | 64 | 300 | 16 | 77000/485 | 4|4000 |
| Standard_E16d_v4 | 16 | 128 | 600 | 32 | 154000/968 | 8|8000 |
| Standard_E20d_v4 | 20 | 160 | 750 | 32 | 193000/1211  | 8|10000 |
| Standard_E32d_v4 | 32 | 256 | 1200 | 32 | 308000/1936 | 8|16000 |
| Standard_E48d_v4 | 48 | 384 | 1800 | 32 | 462000/2904 | 8|24 000 |
| Standard_E64d_v4 | 64 | 504 | 2400 | 32 | 615000/3872 | 8|30 000 |


<sup>**</sup> Эти значения операций ввода-вывода можно гарантировать с помощью [виртуальных машин Gen2](generation-2.md)

## <a name="edsv4-series"></a>Серия Edsv4

Размеры серии Edsv4 выполняются на процессорах Intel&reg; Xeon&reg; Platinum 8272CL (Cascade Lake). Виртуальные машины размеров Edsv4 оснащены до 504 ГиБ ОЗУ, а также быстрым локальным диском SSD большого объема (до 2400 ГиБ). Эти виртуальные машины идеально подходят для корпоративных приложений с высокими требованиями к объему памяти и приложений, для которых важна низкая задержка и высокая скорость при работе с локальным хранилищем.

[ACU](acu.md): 195-210<br>
[Хранилище класса Premium](premium-storage-performance.md): поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколение 1 и 2<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): поддерживается (*требуется не менее 4 виртуальных ЦП*)<br>
[Временные диски ОС](ephemeral-os-disks.md): поддерживаются <br>
<br>

| Размер | vCPU | Память: ГиБ | Временное хранилище (SSD): ГиБ | Максимальное число дисков данных | <sup>**</sup> Максимальная пропускная способность кэшированного и временного хранилища: операций ввода-вывода в секунду (размер кэша в гиб) | Максимальная пропускная способность дисков без кэширования: операций ввода-вывода в секунду / МБит/с | Максимальное число сетевых адаптеров|Ожидаемая пропускная способность сети (Мбит/с) |
|---|---|---|---|---|---|---|---|---|
| Standard_E2ds_v4  | 2 | 16 | 75 | 4 | 19000/120(50) | 3200/48 | 2|1000 |
| Standard_E4ds_v4  | 4 | 32 | 150 | 8 | 38500/242(100) | 6400/96 | 2|2000 |
| Standard_E8ds_v4 | 8 | 64 | 300 | 16 | 77000/485(200) | 12800/192 | 4|4000 |
| Standard_E16ds_v4 | 16 | 128 | 600 | 32 | 154000/968(400) | 25600/384 | 8|8000 |
| Standard_E20ds_v4 | 20 | 160 | 750 | 32 | 193000/1211(500)  | 32000/480  | 8|10000 |
| Standard_E32ds_v4 | 32 | 256 | 1200 | 32 | 308000/1936(800) | 51200/768  | 8|16000 |
| Standard_E48ds_v4 | 48 | 384 | 1800 | 32 | 462000/2904(1200) | 76800/1152 | 8|24 000 |
| Standard_E64ds_v4 <sup>1</sup> | 64 | 504 | 2400 | 32 | 615000/3872(1600) | 80000/1200 | 8|30 000 |
| Standard_E80ids_v4 <sup>2</sup> | 80 | 504 | 2400 | 32 | 615000/3872(1600) | 80000/1200 | 8|30 000 |

<sup>1</sup> [Список доступных размеров ядер ограничен)](./constrained-vcpu.md).

<sup>2</sup> Экземпляр изолирован на оборудовании, выделенном единственному заказчику.

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
