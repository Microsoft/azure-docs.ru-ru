---
title: Серия ND — виртуальные машины Azure
description: Спецификации для виртуальных машин серии ND.
author: vikancha-MSFT
ms.service: virtual-machines
ms.subservice: vm-sizes-gpu
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: 5516e17a74fffd28f432df305dfa7f592644d4a7
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102611578"
---
# <a name="nd-series"></a>Серия ND

Виртуальные машины серии ND — это новое дополнение к семейству GPU, предназначенное для рабочих нагрузок ИИ и глубокого обучения. Они обеспечивают превосходную производительность для обучения и вывода. Экземпляры ND на базе процессоров [NVIDIA Tesla P40](https://images.nvidia.com/content/pdf/tesla/184427-Tesla-P40-Datasheet-NV-Final-Letter-Web.pdf) GPU и Intel Xeon 2690 V4 (Broadwell). Эти экземпляры обеспечивают высокую производительность для операций одиночной точности с числами с плавающей запятой, а также для рабочих нагрузок ИИ, в которых используются Microsoft Cognitive Toolkit, TensorFlow, Caffe и другие платформы. В серии ND значительно увеличен объем памяти GPU (24 ГБ), что позволяет работать с моделями нейронных сетей гораздо большего размера. Как и в серии NC, конфигурация серии ND предусматривает низкую задержку (менее секунды), высокую пропускную способность сети за счет использования RDMA и подключение InfiniBand. Это позволяет выполнять масштабные задания, связанные с обучением, в которых задействованы многочисленные GPU.

[Хранилище класса Premium](premium-storage-performance.md): поддерживается<br>
[Кэширование хранилища класса Premium](premium-storage-performance.md): поддерживается<br>
[Динамическая миграция](maintenance-and-updates.md): не поддерживается<br>
[Обновления с сохранением памяти](maintenance-and-updates.md): не поддерживается<br>
[Поддержка создания виртуальных машин](generation-2.md): поколение 1 и 2<br>
[Ускоренная сеть](../virtual-network/create-vm-accelerated-networking-cli.md): не поддерживается<br>
[Временные диски ОС](ephemeral-os-disks.md): поддерживаются <br>
Нвлинкное Interconnect NVIDIA: не поддерживается<br>

> [!IMPORTANT]
> Для этой серии виртуальных машин значение квоты виртуальных ЦП (Core) для каждого региона в подписке изначально равно 0. [Запросите увеличение квоты виртуальных ЦП](../azure-portal/supportability/resource-manager-core-quotas-request.md) для этой серии в [доступном регионе](https://azure.microsoft.com/regions/services/).
>
| Размер | vCPU | Память: ГиБ | Временное хранилище (SSD): ГиБ | Графический процессор | Память GPU: ГиБ | Максимальное число дисков данных | Максимальная пропускная способность дисков без кэширования: операций ввода-вывода в секунду / МБит/с | Максимальное число сетевых адаптеров |
|---|---|---|---|---|---|---|---|---|
| Standard_ND6s    | 6  | 112 | 736  | 1 | 24 | 12 | 20000/200 | 4 |
| Standard_ND12s   | 12 | 224 | 1474 | 2 | 48 | 24 | 40000/400 | 8 |
| Standard_ND24s   | 24 | 448 | 2948 | 4 | 24 | 32 | 80000/800 | 8 |
| Standard_ND24rs* | 24 | 448 | 2948 | 4 | 96 | 32 | 80000/800 | 8 |

1 GPU = 1 карта P40.

*С поддержкой RDMA

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="supported-operating-systems-and-drivers"></a>Поддерживаемые операционные системы и драйверы

Чтобы воспользоваться преимуществами возможностей GPU виртуальных машин Azure серии N, необходимо установить драйверы GPU NVIDIA.

[Расширение драйвера GPU NVIDIA](./extensions/hpccompute-gpu-windows.md) устанавливает необходимые драйверы CUDA или GRID NVIDIA на виртуальную машину серии N. Для установки расширения и управления им можно использовать портал Azure или такие инструменты, как Azure PowerShell и шаблоны Azure Resource Manager. Сведения о поддерживаемых операционных системах и этапах развертывания см. в [документации по расширению драйвера GPU NVIDIA](./extensions/hpccompute-gpu-windows.md). Общие сведения о расширениях виртуальных машин см. в статье [Расширения и компоненты виртуальных машин Azure](./extensions/overview.md).

Если вы решили установить драйверы NVIDIA GPU вручную, см. раздел [Установка драйвера GPU серии n для Windows](./windows/n-series-driver-setup.md) или [Установка драйвера GPU для Linux](./linux/n-series-driver-setup.md) для поддерживаемых операционных систем, драйверов, установки и проверки.

## <a name="other-sizes"></a>Остальные размеры

- [Универсальные](sizes-general.md)
- [Оптимизированные для памяти](sizes-memory.md)
- [Оптимизированные для хранилища](sizes-storage.md)
- [Оптимизированные для GPU](sizes-gpu.md)
- [Для высокопроизводительных вычислений](sizes-hpc.md)
- [Предыдущие поколения](sizes-previous-gen.md)

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о том, как с помощью [единиц вычислений Azure (ACU)](acu.md) сравнить производительность вычислений для различных номеров SKU Azure.
