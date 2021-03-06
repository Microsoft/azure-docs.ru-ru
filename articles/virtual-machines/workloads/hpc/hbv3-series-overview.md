---
title: Обзор ВИРТУАЛЬНЫХ машин серии HBv3, архитектура, топология, виртуальные машины Azure | Документация Майкрософт
description: Сведения о размере виртуальной машины серии HBv3 в Azure.
services: virtual-machines
author: vermagit
tags: azure-resource-manager
ms.service: virtual-machines
ms.subservice: workloads
ms.workload: infrastructure-services
ms.topic: article
ms.date: 03/25/2021
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: f78420a65cd9c2402266eb9ba973eabe758d7ee5
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105608290"
---
# <a name="hbv3-series-virtual-machine-overview"></a>Обзор виртуальных машин серии HBv3 

Сервер [серии HBv3](../../hbv3-series.md) 2 * 64-Core ЕПИК 7V13 ЦП для всего 128 физических ядер "Zen3". Одновременная многопоточность (SMT) отключена в HBv3. Эти 128 ядер разделены на 16 разделов (8 на сокет), каждый раздел содержит 8 процессорных ядер с равномерным доступом к кэшу третьего уровня 32 МБ. На серверах Azure HBv3 также выполняются следующие параметры AMD BIOS:

```bash
Nodes per Socket (NPS) = 2
L3 as NUMA = Disabled
NUMA domains within VM OS = 4
C-states = Enabled
```

В результате сервер загружается с 4 доменами NUMA (2 для каждого сокета), размер которых составляет 32 ядер. Каждый NUMA имеет прямой доступ к 4 каналам физической памяти DRAM в 3200/с.

Чтобы создать комнату для низкоуровневой оболочки Azure, не мешая виртуальной машине, мы резервируем 8 физических ядер на каждом сервере.

## <a name="vm-topology"></a>Топология виртуальной машины

На следующей схеме показана топология сервера. Мы зарезервированы 8 основных ядер узла гипервизора (желтый) в обоих разъемах ЦП, принимая первые 2 ядра из конкретного ядра сложного аварийного резерва (Ккдс) в каждом домене NUMA, с оставшимися ядрами для виртуальной машины серии HBv3 (Green).

![Топология сервера серии HBv3](./media/architecture/hbv3/hbv3-topology-server.png)

Обратите внимание, что граница ККД не эквивалентна границе NUMA. В HBv3 группа из четырех последовательных (4) Ккдс настраивается как домен NUMA как на уровне сервера-хоста, так и на гостевой виртуальной машине. Таким образом, все размеры виртуальных машин HBv3 предоставляют 4 домена NUMA, которые будут отображаться в ОС и приложении, как показано ниже, 4 универсальных доменах NUMA, каждый из которых имеет разное количество ядер в зависимости от конкретного [размера виртуальной машины HBv3](../../hbv3-series.md).

![Топология виртуальной машины серии HBv3](./media/architecture/hbv3/hbv3-topology-vm.png)

Каждый размер виртуальной машины HBv3 аналогичен физическому макету, возможностям и производительности другого ЦП из серии AMD ЕПИК 7003, как показано ниже.

| Размер виртуальной машины серии HBv3             | Домены NUMA | Ядер на домен NUMA  | Сходство с AMD ЕПИК         |
|---------------------------------|--------------|------------------------|----------------------------------|
Standard_HB120rs_v3               | 4            | 30                     | Два разъема ЕПИК 7713            |
Standard_HB120r 96s_v3            | 4            | 24                     | Два разъема ЕПИК 7643            |
Standard_HB120r 64s_v3            | 4            | 16                     | Два разъема ЕПИК 7543            |
Standard_HB120r 32s_v3            | 4            | 8                      | Два разъема ЕПИК 7313            |
Standard_HB120r 16s_v3            | 4            | 4                      | Двойной сокет ЕПИК 72F3            |

> [!NOTE]
> Размеры виртуальных машин с ограниченным количеством ядер уменьшают только число физических ядер, предоставляемых виртуальной машине. Все глобальные общие ресурсы (ОЗУ, пропускная способность памяти, кэш третьего уровня, подключение ГМИ и Ксгми, InfiniBand, сеть Ethernet Azure, локальный SSD) остаются постоянными. Это позволяет клиенту выбрать размер виртуальной машины, наиболее подходящий для определенного набора рабочих нагрузок или лицензирования программного обеспечения.

Виртуальное NUMA-сопоставление каждого размера виртуальной машины HBv3 сопоставляется с базовой физической топологией NUMA. Не существует потенциальной неверной абстракции топологии оборудования. 

Точная топология для различных [размеров виртуальной машины HBv3](../../hbv3-series.md) выглядит следующим образом с помощью выходных данных [лстопо](https://linux.die.net/man/1/lstopo):
```bash
lstopo-no-graphics --no-io --no-legend --of txt
```
<br>
<details>
<summary>Щелкните, чтобы просмотреть выходные данные лстопо для Standard_HB120rs_v3</summary>

![выходные данные лстопо для виртуальной машины HBv3-120](./media/architecture/hbv3/hbv3-120-lstopo.png)
</details>

<details>
<summary>Щелкните, чтобы просмотреть выходные данные лстопо для Standard_HB120rs-96_v3</summary>

![выходные данные лстопо для виртуальной машины HBv3-96](./media/architecture/hbv3/hbv3-96-lstopo.png)
</details>

<details>
<summary>Щелкните, чтобы просмотреть выходные данные лстопо для Standard_HB120rs-64_v3</summary>

![выходные данные лстопо для виртуальной машины HBv3-64](./media/architecture/hbv3/hbv3-64-lstopo.png)
</details>

<details>
<summary>Щелкните, чтобы просмотреть выходные данные лстопо для Standard_HB120rs-32_v3</summary>

![выходные данные лстопо для виртуальной машины HBv3-32](./media/architecture/hbv3/hbv3-32-lstopo.png)
</details>

<details>
<summary>Щелкните, чтобы просмотреть выходные данные лстопо для Standard_HB120rs-16_v3</summary>

![выходные данные лстопо для виртуальной машины HBv3-16](./media/architecture/hbv3/hbv3-16-lstopo.png)
</details>

## <a name="infiniband-networking"></a>Сеть InfiniBand
Виртуальные машины HBv3 также могут иметь сетевые адаптеры NVIDIA Mellanox HDR InfiniBand (ConnectX-6) с частотой до 200 Гбит/с. Сетевая карта передается на виртуальную машину через SRIOV, что позволяет использовать сетевой трафик для обхода низкоуровневой оболочки. В результате клиенты загружают стандартные драйверы Mellanox ОФЕД на виртуальных машинах HBv3, так как они будут средой без операционной системы.

Виртуальные машины HBv3 поддерживают адаптивную маршрутизацию, динамический подключенный транспорт (ДКТ, дополнительный для стандартных транспортов RC и обновления) и аппаратную разгрузку совокупных ресурсов MPI на встроенный процессор адаптера ConnectX-6. Эти функции улучшают производительность, масштабируемость и согласованность приложений, а их использование настоятельно рекомендуется.

## <a name="temporary-storage"></a>Временное хранилище
HBv3 VMS Feature 3 локальные устройства SSD. Одно устройство предварительно отформатировано для использования в качестве файла подкачки и будет отображаться в виртуальной машине как универсальное устройство SSD.

Два других более крупных твердотельных накопителей предоставляются в виде неформатированных блочных устройств NVMe через Нвмедирект. Так как блочное устройство NVMe обходит гипервизор, оно будет иметь более высокую пропускную способность, большее число операций ввода-вывода и более низкую задержку на источнике.

При связывании в чередующемся массиве SSD NVMe предоставляет до 7 ГБ/с чтения и 3 ГБ/с, а также до 186 000 операций ввода-вывода в секунду (операций чтения) и 201 000 операций ввода-вывода в секунду (операций записи) для глубокой глубины очереди.

## <a name="hardware-specifications"></a>Характеристики оборудования 

| Характеристики оборудования          | Виртуальные машины серии HBv3              |
|----------------------------------|----------------------------------|
| Ядра                            | 120, 96, 64, 32 или 16 (SMT Disabled)               | 
| ЦП                              | AMD ЕПИК 7V13                   | 
| Частота ЦП (не AVX)          | 3,1 ГГц (все ядра), 3,675 ГГц (до 10 ядер)    | 
| Память                           | 448 ГБ (ОЗУ на ядро зависит от размера виртуальной машины)         | 
| Локальный диск                       | 2 * 960 Гб NVMe (Block), твердотельный накопитель 480 ГБ (файл подкачки) | 
| Infiniband;                       | 200 Гбит/с Mellanox ConnectX-6 HDR InfiniBand | 
| Сеть                          | 50 ГБ/с Ethernet (40 ГБ/с) для использования во второй Смартник Azure | 

## <a name="software-specifications"></a>Спецификации программного обеспечения 

| Спецификации программного обеспечения        | Виртуальные машины серии HBv3                                            | 
|--------------------------------|-----------------------------------------------------------|
| Максимальный размер задания MPI               | 36 000 ядер (300 ВМ в одном масштабируемом наборе виртуальных машин с singlePlacementGroup = true) |
| Поддержка MPI                    | HPC-X, Intel MPI, Опенмпи, MVAPICH2, МПИЧ  |
| Дополнительные платформы          | УККС, либфабрик, ПГАС                  |
| Поддержка хранилища Azure          | Диски уровня "Стандартный" и "Премиум" (максимум 32 дисков)              |
| Поддержка ОС для SRIOV RDMA      | CentOS/RHEL 7.6 +, Ubuntu 18.04 +, SLES 12 SP4 +, WinServer 2016 +           |
| Рекомендуемая операционная система для повышения производительности | CentOS 8,1, Windows Server 2019 +
| Поддержка Orchestrator           | Azure Циклеклауд, пакетная служба Azure, AKS; [Параметры конфигурации кластера](../../sizes-hpc.md#cluster-configuration-options)                      | 

> [!NOTE] 
> Windows Server 2012 R2 не поддерживается на HBv3 и других виртуальных машинах с более чем 64 (виртуальными или физическими) ядрами. Подробнее см. [здесь](https://docs.microsoft.com/windows-server/virtualization/hyper-v/supported-windows-guest-operating-systems-for-hyper-v-on-windows).

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с последними объявлениями, примерами рабочей нагрузки HPC и результатами производительности в [блогах сообщества разработчиков Azure](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Сведения о более высоком уровне архитектурного представления выполнения рабочих нагрузок HPC см. в статье [Высокопроизводительные вычисления (HPC) в Azure](/azure/architecture/topics/high-performance-computing/).
