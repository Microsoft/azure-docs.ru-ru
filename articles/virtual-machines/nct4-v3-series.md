---
title: НКАС T4 серии v3
description: Спецификации для виртуальных машин серии НКАС T4 v3.
ms.service: virtual-machines
ms.subservice: vm-sizes-gpu
author: vikancha-MSFT
ms.topic: conceptual
ms.date: 01/12/2021
ms.author: vikancha
ms.openlocfilehash: d73bd81f15263c79e16b574eb961d4ae0ac61175
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103417813"
---
# <a name="ncast4_v3-series"></a>Серия NCasT4_v3 

Виртуальные машины серии NCasT4_v3 на базе процессоров [NVIDIA Tesla T4](https://www.nvidia.com/en-us/data-center/tesla-t4/) GPU и AMD ЕПИК 7V12 (Рим). Компоненты виртуальных машин до 4 видеопроцессоров NVIDIA T4 с 16 ГБ памяти каждый, до 64 процессоров AMD ЕПИК 7V12 (Рим) и 440 гиб системной памяти. Эти виртуальные машины идеально подходят для развертывания служб искусственного интеллекта, например для обработки запросов, созданных пользователем, в реальном времени, а также для интерактивных рабочих нагрузок графики и визуализации с помощью драйвера сетки NVIDIA и технологии виртуальных GPU. Стандартные расчетные рабочие нагрузки GPU, основанные на CUDA, Тенсоррт, Caffe, ONNX и других платформах или графических приложениях с ускорением GPU и DirectX, могут быть развернуты появился, с близкой точностью к пользователям, в серии NCasT4_v3.

<br>

[ACU](acu.md): 230-260<br>
[Хранилище класса Premium](premium-storage-performance.md): поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): не поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): не поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколение 1 и 2<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): поддерживается<br>
[Временные диски ОС](ephemeral-os-disks.md): не поддерживаются <br>
NVIDIA Нвлинк Interconnect: поддерживается<br>
<br>

| Размер | vCPU | Память: ГиБ | Временное хранилище (SSD): ГиБ | Графический процессор | Память GPU: ГиБ | Максимальное число дисков данных | Максимальное количество сетевых адаптеров / ожидаемая пропускная способность сети (Мбит/с) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_NC4as_T4_v3 |4 |28 |180 | 1 | 16 | 8 | 2 / 8000 |
| Standard_NC8as_T4_v3 |8 |56 |360 | 1 | 16 | 16 | 4 / 8000  |
| Standard_NC16as_T4_v3 |16 |110 |360 | 1 | 16 | 32 | 8/8000  |
| Standard_NC64as_T4_v3 |64 |440 |2880 | 4 | 64 | 32 | 8 / 32 000  |


[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="supported-operating-systems-and-drivers"></a>Поддерживаемые операционные системы и драйверы

Чтобы воспользоваться возможностями GPU виртуальных машин серии NCasT4_v3 Azure под управлением Windows или Linux, необходимо установить драйверы GPU NVIDIA.

Чтобы установить драйверы NVIDIA GPU вручную, см. раздел [Установка драйвера GPU серии N для Windows](./windows/n-series-driver-setup.md) для поддерживаемых операционных систем, драйверов, установки и проверки.

Расширение драйвера GPU для Azure NVIDIA будет развертывать драйверы CUDA на виртуальных машинах серии NCasT4_v3. Для рабочих нагрузок графики и визуализации вручную установите драйверы сетки, поддерживаемые Azure.

## <a name="other-sizes"></a>Остальные размеры

- [Универсальные](sizes-general.md)
- [Оптимизированные для памяти](sizes-memory.md)
- [Оптимизированные для хранилища](sizes-storage.md)
- [Оптимизированные для GPU](sizes-gpu.md)
- [Для высокопроизводительных вычислений](sizes-hpc.md)
- [Предыдущие поколения](sizes-previous-gen.md)

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о том, как с помощью [единиц вычислений Azure (ACU)](acu.md) сравнить производительность вычислений для различных номеров SKU Azure.
