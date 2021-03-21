---
title: Производительность размера виртуальной машины серии HBv3
description: Узнайте о результатах тестирования производительности для размеров виртуальных машин серии HBv3 в Azure.
services: virtual-machines
author: vermagit
ms.service: virtual-machines
ms.subservice: workloads
ms.workload: infrastructure-services
ms.topic: article
ms.date: 03/12/2021
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 87c3e4e9b509589624a228ea2e1f4b68e86e3fa8
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104721130"
---
# <a name="hbv3-series-virtual-machine-performance"></a>Производительность виртуальных машин серии HBv3

Пользователи с ранним доступом к виртуальным машинам HBv3 могут рассчитывать на следующие показатели производительности для стандартных микротестов HPC.

| Рабочая нагрузка                                        | HBv3                                                              |
|-------------------------------------------------|-------------------------------------------------------------------|
| STREAM триады                                    | 330-350 ГБ/с (~ 82-86 ГБ/с на NUMA)                                     |
| High-Performance Linpack (ХПЛ)                  | 4 TF (Рпеак, FP64), 8 TF (Рпеак, FP32) для размера виртуальной машины 120-Core               |
| Задержка & RDMA, пропускная способность                        | 1,2 микросекунд (1 байт), 192 ГБ/с (односторонний)                                        |
| FIO для локальных твердотельных накопителей NVMe (конфигурацию RAID0)                  | 7 ГБ/с: операций чтения, 3 ГБ/с; операций чтения 186k операций ввода-вывода, 201k операций ввода-вывода |

## <a name="process-pinning"></a>Закрепление процессов

Закрепление процессов хорошо работает на виртуальных машинах серии HBv3, так как в гостевой виртуальной машине мы предоставляем базовый Кристалл "как есть". Мы настоятельно рекомендуем закреплять процесс для обеспечения оптимальной производительности и согласованности.

## <a name="mpi-latency"></a>Задержка MPI

Тестирование задержки MPI из набора микропроизводительности осу можно выполнить в соответствии с приведенной ниже статьей. Примеры сценариев находятся на [GitHub](https://github.com/Azure/azhpc-images/blob/04ddb645314a6b2b02e9edb1ea52f079241f1297/tests/run-tests.sh).

```bash 
./bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=[INSERT CORE #] ./osu_latency
``` 
## <a name="mpi-bandwidth"></a>Пропускная способность MPI
Тест пропускной способности MPI из набора микропроизводительности осу можно выполнить в соответствии с приведенной ниже статьей. Примеры сценариев находятся на [GitHub](https://github.com/Azure/azhpc-images/blob/04ddb645314a6b2b02e9edb1ea52f079241f1297/tests/run-tests.sh).
```bash
./mvapich2-2.3.install/bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=[INSERT CORE #] ./mvapich2-2.3/osu_benchmarks/mpi/pt2pt/osu_bw
```
## <a name="mellanox-perftest"></a>Mellanox Perftest
[Пакет Mellanox Perftest](https://community.mellanox.com/s/article/perftest-package) имеет много тестов InfiniBand, таких как задержка (ib_send_lat) и пропускная способность (ib_send_bw). Ниже приведен пример команды. 
```console
numactl --physcpubind=[INSERT CORE #]  ib_send_lat -a
```
## <a name="next-steps"></a>Следующие шаги
- Дополнительные сведения о [масштабировании приложений MPI](compiling-scaling-applications.md).
- Ознакомьтесь с последними объявлениями, примерами рабочей нагрузки HPC и результатами производительности в [блогах сообщества разработчиков Azure](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Более высокоуровневое архитектурное представление выполнения рабочих нагрузок HPC см. в статье [высокопроизводительные вычисления (HPC) в Azure](/azure/architecture/topics/high-performance-computing/).