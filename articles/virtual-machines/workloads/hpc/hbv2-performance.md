---
title: Производительность размера виртуальной машины серии HBv2
description: Узнайте о результатах тестирования производительности для размеров виртуальных машин серии HBv2 в Azure.
services: virtual-machines
author: vermagit
ms.service: virtual-machines
ms.subservice: hpc
ms.workload: infrastructure-services
ms.topic: article
ms.date: 09/28/2020
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 28e7066c166a4c61c2f6436e9bd126f712f89f78
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101670071"
---
# <a name="hbv2-series-virtual-machine-sizes"></a>Размеры виртуальных машин серии HBv2

На виртуальных машинах с размером [серии HBv2](../../hbv2-series.md) были выполнены несколько тестов производительности. Ниже приведены некоторые результаты этого тестирования производительности.


| Рабочая нагрузка                                        | HBv2                                                              |
|-------------------------------------------------|-------------------------------------------------------------------|
| STREAM триады                                    | 350 ГБ/с (21-23 ГБ/с на КККС)                                     |
| High-Performance Linpack (ХПЛ)                  | 4 операций (Рпеак, FP64), 8 операций (Рмакс, FP32)               |
| Задержка & RDMA, пропускная способность                        | 1,2 микросекунд, 190 ГБ/с                                        |
| FIO на локальном SSD NVMe                           | 2,7 ГБ/с, операций записи, 1,1 ГБ/с; операций чтения 102 тыс операций ввода-вывода, 115 операций ввода-вывода |
| ИОР на 8 * SSD (цен. категория "Премиум") Azure (управляемые диски P40, конфигурацию RAID0) * *  | 1,3 ГБ/с, 2,5 ГБ/запись; операций чтения 101k операций ввода-вывода, 105k операций ввода-вывода |


## <a name="mpi-latency"></a>Задержка MPI

Выполняется тестирование задержки MPI из набора микротестирования осу. Примеры сценариев находятся на [GitHub](https://github.com/Azure/azhpc-images/blob/04ddb645314a6b2b02e9edb1ea52f079241f1297/tests/run-tests.sh).


```bash 
./bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=[INSERT CORE #] ./osu_latency
``` 
 
:::image type="content" source="./media/latency-hbv2.png" alt-text="MPI с задержкой в Azure ХБ.":::


## <a name="mpi-bandwidth"></a>Пропускная способность MPI

Выполняется тестирование пропускной способности MPI из набора микротестирования осу. Примеры сценариев находятся на [GitHub](https://github.com/Azure/azhpc-images/blob/04ddb645314a6b2b02e9edb1ea52f079241f1297/tests/run-tests.sh).


```bash
./mvapich2-2.3.install/bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=[INSERT CORE #] ./mvapich2-2.3/osu_benchmarks/mpi/pt2pt/osu_bw
``` 

:::image type="content" source="./media/bandwidth-hbv2.png" alt-text="Пропускная способность MPI в Azure ХБ.":::


## <a name="mellanox-perftest"></a>Mellanox Perftest

[Пакет Mellanox Perftest](https://community.mellanox.com/s/article/perftest-package) имеет много тестов InfiniBand, таких как задержка (ib_send_lat) и пропускная способность (ib_send_bw). Ниже приведен пример команды. 


```console
numactl --physcpubind=[INSERT CORE #]  ib_send_lat -a
```


## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с последними объявлениями и некоторыми примерами высокопроизводительных вычислений (HPC) и результатами в [блогах сообщества разработчиков Azure](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Более высокоуровневое архитектурное представление выполнения рабочих нагрузок HPC см. в статье [высокопроизводительные вычисления (HPC) в Azure](/azure/architecture/topics/high-performance-computing/).