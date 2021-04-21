---
title: Серия NCas T4 v3
description: Спецификации для виртуальных машин серии NCas T4 v3.
ms.service: virtual-machines
ms.subservice: vm-sizes-gpu
author: vikancha-MSFT
ms.topic: conceptual
ms.date: 01/12/2021
ms.author: vikancha
ms.openlocfilehash: d73bd81f15263c79e16b574eb961d4ae0ac61175
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103417813"
---
# <a name="ncast4_v3-series"></a>Серия NCasT4_v3 

Виртуальные машины серии NCasT4_v3 на базе процессоров [Nvidia Tesla T4](https://www.nvidia.com/en-us/data-center/tesla-t4/) и AMD EPYC 7V12(Rome). Виртуальные машины включают до 4 процессоров NVIDIA T4, по 16 ГБ памяти на каждом, и до 64 не многопоточных ядер AMD EPYC 7V12 (Rome) и 440 ГиБ системной памяти. Эти виртуальные машины идеально подходят для развертывания служб искусственного интеллекта, например для вывода запросов, созданных пользователем, в реальном времени, а также для интерактивных графических рабочих нагрузок с помощью драйвера NVIDIA GRID и технологии виртуальных GPU. Стандартные вычислительные рабочие нагрузки GPU, основанные на CUDA, TensorRT, Caffe, ONNX и других платформах, или графические приложения на базе GPU, основанные на OpenGL и DirectX, могут быть развернуты без лишних затрат и рядом с пользователями с помощью серии NCasT4_v3.

<br>

[ACU](acu.md): 230-260<br>
[Хранилище класса Premium](premium-storage-performance.md): поддерживается<br>
[Кэширование в хранилище класса Premium](premium-storage-performance.md): поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): не поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): не поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколения 1 и 2<br>
[Ускорение сети](../virtual-network/create-vm-accelerated-networking-cli.md): поддерживается<br>
[Временные диски ОС](ephemeral-os-disks.md): не поддерживаются <br>
Nvidia NVLink Interconnect: поддерживается<br>
<br>

| Размер | vCPU | Память: ГиБ | Временное хранилище (SSD): ГиБ | Графический процессор | Память GPU: ГиБ | Максимальное число дисков данных | Максимальное количество сетевых адаптеров / ожидаемая пропускная способность сети (Мбит/с) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_NC4as_T4_v3 |4 |28 |180 | 1 | 16 | 8 | 2 / 8000 |
| Standard_NC8as_T4_v3 |8 |56 |360 | 1 | 16 | 16 | 4 / 8000  |
| Standard_NC16as_T4_v3 |16 |110 |360 | 1 | 16 | 32 | 8/8000  |
| Standard_NC64as_T4_v3 |64 |440 |2880 | 4 | 64 | 32 | 8 / 32 000  |


[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="supported-operating-systems-and-drivers"></a>Поддерживаемые операционные системы и драйверы

Чтобы воспользоваться преимуществами GPU виртуальных машин Azure серии NCasT4_v3 под управлением Windows или Linux, необходимо установить графические драйверы Nvidia.

Чтобы установить драйверы NVIDIA GPU вручную, см. сведения о поддерживаемых операционных системах и действиях по установке и проверке в разделе [Установка драйвера GPU для виртуальных машин серии N под управлением Windows](./windows/n-series-driver-setup.md).

Расширение драйвера GPU для Azure Nvidia будет развертывать драйверы CUDA на виртуальных машинах серии NCasT4_v3. Для рабочих нагрузок графики и визуализации вручную установите драйверы GRID, поддерживаемые Azure.

## <a name="other-sizes"></a>Остальные размеры

- [Универсальные](sizes-general.md)
- [Оптимизированные для памяти](sizes-memory.md)
- [Оптимизированные для хранилища](sizes-storage.md)
- [Оптимизированные для GPU](sizes-gpu.md)
- [Для высокопроизводительных вычислений](sizes-hpc.md)
- [Предыдущие поколения](sizes-previous-gen.md)

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о том, как с помощью [единиц вычислений Azure (ACU)](acu.md) сравнить производительность вычислений для различных номеров SKU Azure.
