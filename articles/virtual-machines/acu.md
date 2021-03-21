---
title: Общие сведения о единице вычислений Azure
description: Обзор концепции единиц вычислений Azure. Единица вычислений Azure (ACU) предоставляет собой способ сравнения производительности ЦП для различных номеров SKU Azure.
author: mimckitt
ms.service: virtual-machines
ms.subservice: azure-compute-unit
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 02/03/2020
ms.author: mimckitt
ms.reviewer: davberg
ms.openlocfilehash: ae37771cd9e48f450ae538174b9bed9cfa866f9d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102565535"
---
# <a name="azure-compute-unit-acu"></a>Единицы вычислений Azure (ACU)

Концепция единицы вычислений Azure (ACU) позволяет сравнивать производительность ЦП разных номеров SKU Azure. Это поможет вам легко определить, какие SKU с наибольшей вероятностью удовлетворят ваши требования к производительности. В настоящее время ACU на небольшой виртуальной машине (Standard_A1), которая составляет 100, а все остальные номера SKU представляют приблизительно меньшее количество времени, в течение которого SKU может выполнять стандартный тест производительности.

* В единицах ACU используется технология Intel® Turbo для увеличения частоты ЦП и повышения производительности.  Степень повышения производительности может различаться в зависимости от размера виртуальной машины, рабочей нагрузки и других рабочих нагрузок, выполняющихся на том же узле.

** В единицах ACU используется технология AMD® Boost для увеличения частоты ЦП и повышения производительности.  Степень повышения производительности может различаться в зависимости от размера виртуальной машины, рабочей нагрузки и других рабочих нагрузок, выполняющихся на том же узле.

*** Поддержка технологии Hyper Threading и вложенной виртуализации.

> [!IMPORTANT]
> Единица ACU используется только для справки. Фактические результаты для конкретной рабочей нагрузки могут отличаться.
<br>

| Семейство SKU | ACU \ виртуальные ЦП | Виртуальный процессор: ядро |
| --- | --- |---|
| [A0](sizes-previous-gen.md) |50 | 1:1 |
| [A1 - A4](sizes-previous-gen.md) |100 | 1:1 |
| [A5 - A7](sizes-previous-gen.md) |100 | 1:1 |
| [A1_v2 - A8_v2](sizes-general.md) |100 | 1:1 |
| [A2m_v2 - A8m_v2](sizes-general.md) |100 | 1:1 |
| [A8 - A11](sizes-previous-gen.md) |225* | 1:1 |
| [Б](sizes-b-series-burstable.md) |Различается | Различается |
| [D1 - D14](sizes-previous-gen.md) |160-250 | 1:1 |
| [D1_v2 - D15_v2](dv2-dsv2-series.md) |210–250* | 1:1 |
| [DS1 - DS14](sizes-previous-gen.md) |160-250 | 1:1 |
| [DS1_v2 - DS15_v2](dv2-dsv2-series.md) |210–250* | 1:1 |
| [D_v3](dv3-dsv3-series.md) |160–190* | 2:1\*\*\* |
| [Ds_v3](dv3-dsv3-series.md) |160–190* | 2:1\*\*\* |
| [Dav4](dav4-dasv4-series.md) |230-260 * * | 2:1 |
| [Dasv4](dav4-dasv4-series.md) |230-260 * * | 2:1 |
| [Dv4](dv4-dsv4-series.md) | 195—210 | 2:1\*\*\* |
| [Dsv4](dv4-dsv4-series.md) | 195—210 | 2:1\*\*\* |
| [Ddv4](ddv4-ddsv4-series.md) | 195-210 * | 2:1\*\*\* |
| [Ddsv4](ddv4-ddsv4-series.md) | 195-210 * | 2:1\*\*\* |
| [E_v3](ev3-esv3-series.md) |160–190* | 2:1\*\*\*|
| [Es_v3](ev3-esv3-series.md) |160–190* | 2:1\*\*\* |
| [Eav4](eav4-easv4-series.md) |230-260 * * | 2:1 |
| [Easv4](eav4-easv4-series.md) | 230-260 * * | 2:1 |
| [Ev4](ev4-esv4-series.md) | 195—210 | 2:1\*\*\* |
| [Esv4](ev4-esv4-series.md) | 195—210 | 2:1\*\*\* |
| [Edv4](edv4-edsv4-series.md) | 195-210 * | 2:1\*\*\* |
| [Edsv4](edv4-edsv4-series.md) | 195-210 * | 2:1\*\*\* |
| [F2s_v2 - F72s_v2](fsv2-series.md) |195-210 * | 2:1\*\*\* |
| [F1 - F16](sizes-previous-gen.md) |210–250* | 1:1 |
| [F1s - F16s](sizes-previous-gen.md) |210–250* | 1:1 |
| [G1 - G5](sizes-previous-gen.md) |180–240* | 1:1 |
| [GS1 - GS5](sizes-previous-gen.md) |180–240* | 1:1 |
| [H](h-series.md) |290–300* | 1:1 |
| [хб](hb-series.md) |199-216 * * | 1:1 |
| [HC](hc-series.md) |297-315 * | 1:1 |
| [L4s - L32s](sizes-previous-gen.md) |180–240* | 1:1 |
| [L8s_v2 - L80s_v2](lsv2-series.md) |150 - 175** | 2:1 |
| [M](m-series.md) | 160-180 | 2:1\*\*\* |
| [NVv4](nvv4-series.md) |230-260 * * | 2:1 |

Ниже приведены ссылки на дополнительные сведения о различных размерах:

- [Общего назначения](sizes-general.md)
- [Оптимизированные для операций в памяти](sizes-memory.md)
- [Оптимизированные для вычислений](sizes-compute.md)
- [Оптимизированные для GPU](sizes-gpu.md)
- [Для высокопроизводительных вычислений](sizes-hpc.md)
- [Оптимизированные для операций в хранилище](sizes-storage.md)
