---
title: Производительность размера виртуальной машины серии ХБ
description: Узнайте о результатах тестирования производительности для размеров виртуальных машин серии ХБ в Azure.
author: vermagit
ms.service: virtual-machines
ms.subservice: hpc
ms.topic: article
ms.date: 09/09/2020
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 1f2d24279d3e74774da05eba42c0e916370d4f1c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101672155"
---
# <a name="hb-series-virtual-machine-sizes"></a>Размеры виртуальных машин серии ХБ

На размерах серии ХБ было запущено несколько тестов производительности. Ниже приведены некоторые результаты этого тестирования производительности.

| Рабочая нагрузка                                        | HB                    |
|-------------------------------------------------|-----------------------|
| STREAM триады                                    | 260 ГБ/с (32-33 Гб/с на КККС)  |
| High-Performance Linpack (ХПЛ)                  | 1 000 гигафлопс (Рпеак), 860 гигафлопс (Рмакс) |
| Задержка & RDMA, пропускная способность                        | 1,27 микросекунд, 99,1 ГБ/с   |
| FIO на локальном SSD NVMe                           | 1,7 ГБ/с, операций чтения, 1,0 ГБ/с      |  
| ИОР на 4 * SSD (цен. категория "Премиум") Azure (управляемые диски P30, конфигурацию RAID0) * *  | 725 МБ/с, 780 МБ/операций записи   |


## <a name="mpi-latency"></a>Задержка MPI

Выполняется тестирование задержки MPI из набора микротестирования осу. Примеры сценариев в [GitHub](https://github.com/Azure/azhpc-images/blob/04ddb645314a6b2b02e9edb1ea52f079241f1297/tests/run-tests.sh)

```bash
./bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=[INSERT CORE #] ./osu_latency 
```

:::image type="content" source="./media/latency-hb.png" alt-text="MPI с задержкой в Azure ХБ.":::

## <a name="mpi-bandwidth"></a>Пропускная способность MPI

Выполняется тестирование пропускной способности MPI из набора микротестирования осу. Примеры сценариев в [GitHub](https://github.com/Azure/azhpc-images/blob/04ddb645314a6b2b02e9edb1ea52f079241f1297/tests/run-tests.sh)

```bash
./mvapich2-2.3.install/bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=[INSERT CORE #] ./mvapich2-2.3/osu_benchmarks/mpi/pt2pt/osu_bw
```

:::image type="content" source="./media/bandwidth-hb.png" alt-text="Пропускная способность MPI в Azure ХБ.":::


## <a name="mellanox-perftest"></a>Mellanox Perftest

[Пакет Mellanox Perftest](https://community.mellanox.com/s/article/perftest-package) имеет много тестов InfiniBand, таких как задержка (ib_send_lat) и пропускная способность (ib_send_bw). Ниже приведен пример команды.

```console
numactl --physcpubind=[INSERT CORE #]  ib_send_lat -a
```

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с последними объявлениями и некоторыми примерами высокопроизводительных вычислений (HPC) и результатами в [блогах сообщества разработчиков Azure](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Более высокоуровневое архитектурное представление выполнения рабочих нагрузок HPC см. в статье [высокопроизводительные вычисления (HPC) в Azure](/azure/architecture/topics/high-performance-computing/).
