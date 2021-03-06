---
title: Размеры виртуальных машин Azure — GPU | Документация Майкрософт
description: Список различных размеров GPU, доступных для виртуальных машин в Azure. Сведения о количестве виртуальных ЦП, дисков данных и сетевых адаптеров, а также о пропускной способности хранилища и сети для размеров виртуальных машин этой серии.
author: vikancha-MSFT
ms.service: virtual-machines
ms.subservice: vm-sizes-gpu
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 02/03/2020
ms.author: jushiman
ms.openlocfilehash: 09d62ea5f2db77c14e8faff44de7fb3ce759c6fe
ms.sourcegitcommit: dae6b628a8d57540263a1f2f1cdb10721ed1470d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "105709741"
---
# <a name="gpu-optimized-virtual-machine-sizes"></a>Размеры виртуальных машин, оптимизированных для GPU

Размер ВИРТУАЛЬНЫХ машин, оптимизированных для GPU, — это специализированные виртуальные машины, доступные с помощью одного, нескольких и дробных GPU. Эти размеры предназначены для рабочих нагрузок с большим объемом вычислений, графической обработки и визуализаций. В этой статье содержатся сведения о количестве и типе GPU, виртуальных ЦП, дисков данных и сетевых адаптеров. Кроме того, для каждого размера виртуальной машины этой группы учитывается пропускная способность хранилища и сети.

- Размеры серий [NCv3](ncv3-series.md) и [T4_v3 NC](nct4-v3-series.md) оптимизированы для ресурсоемких приложений с большим объемом вычислений. Примерами могут быть CUDA и OpenCL приложения и моделирование, AI и глубокое обучение. Серия NC T4 серии v3 посвящена рабочим нагрузкам вывода, использующим процессор NVIDIA Tesla T4 GPU и AMD EPYC2 Рим. Серия NCv3 сосредоточена на высокопроизводительных вычислительных и ИСКУССТВЕНных рабочих нагрузках, в которых используется графический процессор NVIDIA Tesla V100.

- Размер [серии NDv2](ndv2-series.md) ориентирован на обучающие и масштабируемые приложения для глубокого обучения. В серии NDv2 используется процессор NVIDIA Волта V100 и Intel Xeon Platinum 8168 (Skylake).

- Размеры серий [NV](nv-series.md) и [NVv3](nvv3-series.md) оптимизированы и предназначены для удаленной визуализации, потоковой передачи, игр, кодирования и сценариев VDI с использованием таких платформ, как OpenGL и DirectX. Эти виртуальные машины работают на базе графического процессора NVIDIA Tesla M60.

- [Серия NVv4](nvv4-series.md) Размеры виртуальных машин оптимизированы и предназначены для VDI и удаленной визуализации. При использовании секционированных GPU NVv4 предлагает правильный размер для рабочих нагрузок, для которых требуются небольшие ресурсы GPU. Эти виртуальные машины поддерживаются графическим процессором AMD Radeon порывом MI25. В настоящее время виртуальные машины NVv4 поддерживают только гостевую операционную систему Windows.

## <a name="supported-operating-systems-and-drivers"></a>Поддерживаемые операционные системы и драйверы

Чтобы воспользоваться преимуществами возможностей GPU виртуальных машин Azure серии N, необходимо установить драйверы NVIDIA или AMD GPU.

- Для виртуальных машин, которые поддерживаются графическими процессорами NVIDIA, [расширение драйвера GPU NVIDIA](./extensions/hpccompute-gpu-windows.md) устанавливает соответствующие драйверы NVIDIA CUDA или Grid. Для установки расширения и управления им можно использовать портал Azure или такие инструменты, как Azure PowerShell и шаблоны Azure Resource Manager. Сведения о поддерживаемых операционных системах и этапах развертывания см. в [документации по расширению драйвера GPU NVIDIA](./extensions/hpccompute-gpu-windows.md). Общие сведения о расширениях виртуальных машин см. в статье [Расширения и компоненты виртуальных машин Azure](./extensions/overview.md).

   Кроме того, драйверы NVIDIA GPU можно установить вручную. См. статью [Установка драйверов NVIDIA GPU на виртуальных машинах серии n под управлением Windows](./windows/n-series-driver-setup.md) или [Установка драйверов NVIDIA GPU на виртуальных машинах серии n под управлением Linux](./linux/n-series-driver-setup.md) для поддерживаемых операционных систем, драйверов, установки и шагов проверки.

- Для виртуальных машин, поддерживаемых процессорами AMD GPU, [расширение драйвера GPU AMD](./extensions/hpccompute-amd-gpu-windows.md) устанавливает соответствующие драйверы AMD. Для установки расширения и управления им можно использовать портал Azure или такие инструменты, как Azure PowerShell и шаблоны Azure Resource Manager. Общие сведения о расширениях виртуальных машин см. в статье [Расширения и компоненты виртуальных машин Azure](./extensions/overview.md).

   Кроме того, драйверы AMD GPU можно установить вручную. Поддерживаемые операционные системы, драйверы, установка и проверка см. [в статье Установка драйверов AMD GPU на виртуальных машинах серии N под управлением Windows](./windows/n-series-amd-driver-setup.md) .

## <a name="deployment-considerations"></a>Рекомендации по развертыванию

- Сведения о доступности виртуальных машин серии N по регионам см. на [этой странице](https://azure.microsoft.com/regions/services/).

- Виртуальные машины серии N поддерживают только модель развертывания с помощью Resource Manager.

- Виртуальные машины серии N отличаются типом хранилища Azure, которое поддерживается для их дисков. Виртуальные машины серий NC и NV поддерживают только диски на основе хранилища дисков уровня "Стандартный" (HDD). Все остальные виртуальные машины GPU поддерживают диски виртуальных машин с поддержкой стандартных Хранилище дисков и Premium Хранилище дисков (SSD).

- Если вы хотите развернуть большое количество виртуальных машин серии N, мы рекомендуем подписку с оплатой по мере использования или другие варианты покупки. Если вы используете [бесплатную учетную запись Azure](https://azure.microsoft.com/free/), вам доступно ограниченное количество вычислительных ядер Azure.

- Возможно, вам потребуется увеличить квоту на использование ядер (для каждого региона) в подписке Azure и отдельную квоту для ядер серии NC, NCv2, NCv3, ND, NDv2, NV или NVv2. Чтобы увеличить квоту, [отправьте запрос в службу поддержки](../azure-portal/supportability/how-to-create-azure-support-request.md). Это бесплатная услуга. Ограничения по умолчанию отличаются в зависимости от категории подписки.

## <a name="other-sizes"></a>Остальные размеры

- [Универсальные](sizes-general.md)
- [Оптимизированные для вычислений](sizes-compute.md)
- [Для высокопроизводительных вычислений](sizes-hpc.md)
- [Оптимизированные для памяти](sizes-memory.md)
- [Оптимизированные для хранилища](sizes-storage.md)
- [Предыдущие поколения](sizes-previous-gen.md)

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о том, как с помощью [единиц вычислений Azure (ACU)](acu.md) сравнить производительность вычислений для различных номеров SKU Azure.
