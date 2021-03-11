---
title: Серия NP-Series — виртуальные машины Azure
description: Спецификации виртуальных машин серии NP.
author: vikancha-MSFT
ms.service: virtual-machines
ms.subservice: vm-sizes-gpu
ms.topic: conceptual
ms.date: 02/09/2021
ms.author: vikancha
ms.openlocfilehash: 09adb19623ea866091e1b949e78263661eddbb52
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102551153"
---
# <a name="np-series-preview"></a>Серии NP-Series (Предварительная версия) 
Виртуальные машины серии NP работают на базе [Ксилинкс U250 ](https://www.xilinx.com/products/boards-and-kits/alveo/u250.html) FPGAs для ускорения рабочих нагрузок, включая определение машинного обучения, кодирование видео и анализ базы данных & Analytics. Виртуальные машины серии NP также работают на базе процессоров Intel Xeon 8171M (Skylake) со всеми основными тактами скорости Turbo 3,2 ГГц.

Отправьте запрос, используя [форму предварительной версии](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR9x_QCQkJXxHl4qOI4jC9YtUOVI0VkgwVjhaTFFQMTVBTDFJVFpBMzJSSCQlQCN0PWcu) , которая является частью программы предварительной версии серии NP.


[Хранилище класса Premium](premium-storage-performance.md): поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): не поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): не поддерживается<br>
Поддержка создания виртуальных машин: поколение 1<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): поддерживается<br>
[Временные диски ОС](ephemeral-os-disks.md): не поддерживаются <br>
<br>

| Размер | vCPU | Память: ГиБ | Временное хранилище (SSD): ГиБ | ППВМ | Память FPGA: гиб | Максимальное число дисков данных | Максимальное число сетевых карт/ожидаемая пропускная способность сети (Мбит/с) | 
|---|---|---|---|---|---|---|---|
| Standard_NP10s | 10 | 168 | 736  | 1 | 64  | 8 | 1 / 7500 | 
| Standard_NP20s | 20 | 336 | 1474 | 2 | 128 | 16 | 2 / 15000 | 
| Standard_NP40s | 40 | 672 | 2948 | 4 | 256 | 32 | 4 / 30000 | 



[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="supported-operating-systems-and-drivers"></a>Поддерживаемые операционные системы и драйверы
Ознакомьтесь с [заметками о выпуске среды выполнения ксилинкс (КСРТ)](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_2/ug1451-xrt-release-notes.pdf) , чтобы получить полный список поддерживаемых операционных систем.

Во время предварительной версии программы Microsoft Azure инженерные группы будут использовать конкретные инструкции по установке драйверов.

## <a name="other-sizes"></a>Остальные размеры

- [Универсальные](sizes-general.md)
- [Оптимизированные для памяти](sizes-memory.md)
- [Оптимизированные для хранилища](sizes-storage.md)
- [Оптимизированные для GPU](sizes-gpu.md)
- [Для высокопроизводительных вычислений](sizes-hpc.md)
- [Предыдущие поколения](sizes-previous-gen.md)

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о том, как с помощью [единиц вычислений Azure (ACU)](acu.md) сравнить производительность вычислений для различных номеров SKU Azure.
