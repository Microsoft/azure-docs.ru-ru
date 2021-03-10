---
title: Виртуальные машины Azure серии dv4 и Dsv4
description: Спецификации для виртуальных машин серии dv4 и Dsv4.
author: brbell
ms.author: brbell
ms.reviewer: cynthn
ms.custom: mimckitt
ms.service: virtual-machines
ms.subservice: vm-sizes-general
ms.topic: conceptual
ms.date: 06/08/2020
ms.openlocfilehash: cc6ce30855d17f45636e0df04978fed88dcecff7
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102549147"
---
# <a name="dv4-and-dsv4-series"></a>Серии Dv4 и Dsv4

Серия dv4 и Dsv4 работает на &reg; &reg; процессорах Intel Xeon Platinum 8272CL (Cascade Lake) в конфигурации с технологией Hyper-Threading, предоставляя лучшее ценное предложение для большинства рабочих нагрузок общего назначения. Он включает всю частоту ядра Turbo 3,4 ГГц. 

> [!NOTE]
> Часто задаваемые вопросы см. в статье  [размеры виртуальных машин Azure без локального временного диска](azure-vms-no-temp-disk.md).
## <a name="dv4-series"></a>Серия dv4

Размеры серии dv4 работают на Intel &reg; Xeon &reg; Platinum 8272CL (Cascade Lake). Размеры серии dv4 предлагают сочетание параметров виртуальных ЦП, памяти и удаленного хранилища для большинства рабочих нагрузок. Виртуальные машины серии dv4 имеют функцию [Intel &reg; Hyper-Threading Technology](https://www.intel.com/content/www/us/en/architecture-and-technology/hyper-threading/hyper-threading-technology.html).

Удаление хранилища данных диска оплачивается отдельно от виртуальных машин. Чтобы использовать диски хранилища класса Premium, используйте размеры Dsv4. Показатели ценообразования и выставления счетов для размеров Dsv4 совпадают с серией dv4.

[ACU](acu.md): 195-210<br>
[Хранилище класса Premium](premium-storage-performance.md): не поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): не поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколение 1<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): поддерживается (*требуется не менее 4 виртуальных ЦП*)<br>
[Временные диски ОС](ephemeral-os-disks.md): не поддерживаются <br>
<br>

| Размер | vCPU | Память: ГиБ | Временное хранилище (SSD): ГиБ | Максимальное число дисков данных | Максимальное число сетевых адаптеров|Ожидаемая пропускная способность сети (Мбит/с) |
|---|---|---|---|---|---|---|
| Standard_D2_v4 | 2 | 8 | Только удаленное хранилище | 4 | 2|1000 |
| Standard_D4_v4 | 4 | 16  | Только удаленное хранилище | 8 | 2|2000 |
| Standard_D8_v4 | 8 | 32 | Только удаленное хранилище | 16 | 4|4000 |
| Standard_D16_v4 | 16 | 64 | Только удаленное хранилище | 32 | 8|8000 |
| Standard_D32_v4 | 32 | 128 | Только удаленное хранилище | 32 | 8|16000 |
| Standard_D48_v4 | 48 | 192 | Только удаленное хранилище | 32 | 8|24 000 |
| Standard_D64_v4 | 64 | 256 | Только удаленное хранилище | 32 | 8|30 000 |

## <a name="dsv4-series"></a>Серия Dsv4

Размеры серии Dsv4 работают на Intel &reg; Xeon &reg; Platinum 8272CL (Cascade Lake). Размеры серии dv4 предлагают сочетание параметров виртуальных ЦП, памяти и удаленного хранилища для большинства рабочих нагрузок. Виртуальные машины серии Dsv4 имеют функцию [Intel &reg; Hyper-Threading Technology](https://www.intel.com/content/www/us/en/architecture-and-technology/hyper-threading/hyper-threading-technology.html). Удаление хранилища данных диска оплачивается отдельно от виртуальных машин.

[ACU](acu.md): 195-210<br>
[Хранилище класса Premium](premium-storage-performance.md): поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколение 1 и 2<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): поддерживается (*требуется не менее 4 виртуальных ЦП*)<br>
[Временные диски ОС](ephemeral-os-disks.md): не поддерживаются <br>
<br>

| Размер | vCPU | Память: ГиБ | Временное хранилище (SSD): ГиБ | Максимальное число дисков данных | Максимальная пропускная способность дисков без кэширования: операций ввода-вывода в секунду / МБит/с | Максимальное число сетевых адаптеров|Ожидаемая пропускная способность сети (Мбит/с) |
|---|---|---|---|---|---|---|---|
| Standard_D2s_v4 | 2 | 8  | Только удаленное хранилище | 4 | 3200/48 | 2|1000 |
| Standard_D4s_v4 | 4 | 16 | Только удаленное хранилище | 8 | 6400/96 | 2|2000 |
| Standard_D8s_v4 | 8 | 32 | Только удаленное хранилище | 16 | 12800/192 | 4|4000 |
| Standard_D16s_v4 | 16 | 64  | Только удаленное хранилище | 32 | 25600/384 | 8|8000 |
| Standard_D32s_v4 | 32 | 128 | Только удаленное хранилище | 32 | 51200/768 | 8|16000 |
| Standard_D48s_v4 | 48 | 192 | Только удаленное хранилище | 32 | 76800/1152 | 8|24 000 |
| Standard_D64s_v4 | 64 | 256 | Только удаленное хранилище | 32 | 80000/1200 | 8|30 000 |
