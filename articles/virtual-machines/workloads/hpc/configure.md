---
title: Настройка и оптимизация виртуальных машин Azure серии H и серий N с поддержкой InfiniBand
description: Узнайте, как настроить и оптимизировать виртуальные машины серии H и N с поддержкой InfiniBand для HPC.
author: vermagit
ms.service: virtual-machines
ms.subservice: hpc
ms.topic: article
ms.date: 10/23/2020
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 94334e54865b3a3b603cbd0b3943899a375d894e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101675654"
---
# <a name="configure-and-optimize-vms"></a>Настройка и оптимизация виртуальных машин

В этой статье приведены известные методы настройки и оптимизации виртуальных машин серии [H](../../sizes-hpc.md) и [N](../../sizes-gpu.md) с поддержкой InfiniBand для HPC.

## <a name="vm-images"></a>Образы виртуальных машин
На виртуальных машинах с поддержкой InfiniBand для включения RDMA требуются соответствующие драйверы. В Linux образы виртуальных машин CentOS-HPC в Marketplace предварительно настроены с соответствующими драйверами. Образы виртуальных машин Ubuntu можно настроить с помощью правильных драйверов, выполнив приведенные [здесь инструкции](https://techcommunity.microsoft.com/t5/azure-compute/configuring-infiniband-for-ubuntu-hpc-and-gpu-vms/ba-p/1221351). Также рекомендуется создавать [пользовательские образы виртуальных машин](../../linux/tutorial-custom-images.md) с соответствующими драйверами и конфигурацией и повторно использовать их.

> [!NOTE]
> На виртуальных машинах [серии N](../../sizes-gpu.md) с поддержкой GPU также требуются драйверы GPU, которые можно добавить с помощью [расширений виртуальной машины](../../extensions/hpccompute-gpu-linux.md) или [вручную](../../linux/n-series-driver-setup.md). Некоторые образы виртуальных машин в Marketplace также устанавливаются вместе с драйверами GPU NVIDIA.

### <a name="centos-hpc-vm-images"></a>CentOS — образы виртуальных машин HPC

#### <a name="non-sr-iov-enabled-vms"></a>Виртуальные машины с поддержкой не SR-IOV
Для [виртуальных машин с поддержкой RDMA](../../sizes-hpc.md#rdma-capable-instances), не поддерживающих SR-IOV, CENTOS-HPC версии 6,5 или более поздней версии, подходит до 7,5 в Marketplace. Например, для [виртуальных машин серии H16](../../h-series.md)рекомендуется использовать версии 7,1 и 7,5. Эти образы виртуальных машин предварительно загружаются с драйверами сети Direct для RDMA и Intel MPI версии 5,1.

> [!NOTE]
> В этих образах HPC на основе CentOS для виртуальных машин, не использующих SR-IOV, обновления ядра отключены в файле конфигурации **Yum** . Это обусловлено тем, что драйверы Нетворкдирект Linux RDMA распространяются в виде пакета RPM, а обновления драйверов могут не работать при обновлении ядра.

#### <a name="sr-iov-enabled-vms"></a>Виртуальные машины с поддержкой SR-IOV
  Для [виртуальных машин с поддержкой RDMA](../../sizes-hpc.md#rdma-capable-instances), поддерживающих SR-IOV, подходят [CentOS-HPC версии 7,6 или более ПОЗДНИХ](https://techcommunity.microsoft.com/t5/Azure-Compute/CentOS-HPC-VM-Image-for-SR-IOV-enabled-Azure-HPC-VMs/ba-p/665557) версий виртуальных машин в Marketplace. Эти образы виртуальных машин оптимизированы и предварительно загружены с драйверами ОФЕД для RDMA и различными часто используемыми библиотеками MPI и инженерными пакетами для научных вычислений и являются самым простым способом приступить к работе.

  Примеры сценариев, используемых при создании образов виртуальных машин CentOS-HPC версии 7,6 и более поздних версий из базового образа CentOS Marketplace, находятся в [репозитории азпк-Images](https://github.com/Azure/azhpc-images/tree/master/centos).
  
  > [!NOTE] 
  > Новейшие образы Azure HPC Marketplace имеют Mellanox ОФЕД 5,1 и более поздних версий, которые не поддерживают карты ConnectX3-Pro InfiniBand. Размеры виртуальных машин серии N с поддержкой SR-IOV с FDR InfiniBand (например, NCv3) смогут использовать следующие версии образов виртуальных машин CentOS-HPC или более ранних версий:
  >- OpenLogic: CentOS-HPC: 7.6:7.6.2020062900
  >- OpenLogic: CentOS-HPC: 7_6gen2:7.6.2020062901
  >- OpenLogic: CentOS-HPC: 7.7:7.7.2020062600
  >- OpenLogic: CentOS-HPC: 7_7-Gen2:7.7.2020062601
  >- OpenLogic: CentOS-HPC: 8_1:8.1.2020062400
  >- OpenLogic: CentOS-HPC: 8_1-Gen2:8.1.2020062401


### <a name="rhelcentos-vm-images"></a>Образы виртуальных машин RHEL/CentOS
Образы виртуальных машин на основе RHEL или CentOS, которые не поддерживают HPC в Marketplace, можно настроить для использования на [виртуальных машинах](../../sizes-hpc.md#rdma-capable-instances)с поддержкой RDMA с поддержкой SR-IOV. Узнайте больше о [включении InfiniBand](enable-infiniband.md) и [настройке MPI](setup-mpi.md) на виртуальных машинах.

  Примеры сценариев, используемых при создании образов виртуальных машин CentOS-HPC версии 7,6 и более поздних версий из базового образа CentOS Marketplace, находятся в [репозитории азпк-Images](https://github.com/Azure/azhpc-images/tree/master/centos).
  
  > [!NOTE]
  > Mellanox ОФЕД 5,1 и более поздних версий не поддерживают карты ConnectX3-Pro InfiniBand на виртуальных машинах серии N с поддержкой SR-IOV с FDR InfiniBand (например, NCv3). Используйте LTS Mellanox ОФЕД версии 4.9-0.1.7.0 или более раннюю версию на виртуальных машинах серии N с картами ConnectX3-Pro. Дополнительные сведения см. [здесь](https://www.mellanox.com/products/infiniband-drivers/linux/mlnx_ofed).

### <a name="ubuntu-vm-images"></a>Образы виртуальных машин Ubuntu
Машины Ubuntu Server 16,04 LTS, 18,04 LTS и 20,04 LTS образы виртуальных машин в Marketplace поддерживаются для [виртуальных машин с поддержкой RDMA](../../sizes-hpc.md#rdma-capable-instances)(SR-IOV и без SR-IOV). Узнайте больше о [включении InfiniBand](enable-infiniband.md) и [настройке MPI](setup-mpi.md) на виртуальных машинах.

  Примеры сценариев, которые могут использоваться при создании образов виртуальных машин HPC на базе Ubuntu 18,04 LTS, находятся в [репозитории азпк-Images](https://github.com/Azure/azhpc-images/tree/master/ubuntu/ubuntu-18.x/ubuntu-18.04-hpc).

### <a name="suse-linux-enterprise-server-vm-images"></a>SUSE Linux Enterprise Server образы виртуальных машин
SLES 12 SP3 для HPC, SLES 12 SP3 для HPC (Premium), SLES 12 SP1 для HPC, SLES 12 SP1 для HPC (Premium), SLES 12 SP4 и SLES 15 образов виртуальных машин в Marketplace поддерживаются. Эти образы виртуальных машин предварительно загружаются с драйверами сети Direct для RDMA и Intel MPI версии 5,1. Дополнительные сведения о [настройке MPI](setup-mpi.md) на виртуальных машинах.

## <a name="optimize-vms"></a>Оптимизация виртуальных машин

Ниже приведены некоторые дополнительные параметры оптимизации для повышения производительности виртуальной машины.

### <a name="update-lis"></a>Обновить LIS

Если это необходимо для функциональности или производительности, [драйверы Linux Integration Services (LIS)](../../linux/endorsed-distros.md) могут быть установлены или обновлены на поддерживаемых ОС дистрибутивов, особенно при развертывании с помощью пользовательского образа или более старой версии ОС, например CENTOS/RHEL 6. x или более ранней версии 7. x.

```bash
wget https://aka.ms/lis
tar xzf lis
pushd LISISO
./upgrade.sh
```

### <a name="reclaim-memory"></a>Освобождение памяти

Повышение производительности за счет автоматического освобождения памяти для предотвращения удаленного доступа к памяти.

```bash
echo 1 >/proc/sys/vm/zone_reclaim_mode
```

Чтобы сохранить это после перезагрузки виртуальной машины, выполните следующие действия.

```bash
echo "vm.zone_reclaim_mode = 1" >> /etc/sysctl.conf sysctl -p
```

### <a name="disable-firewall-and-selinux"></a>Отключение брандмауэра и SELinux

```bash
systemctl stop iptables.service
systemctl disable iptables.service
systemctl mask firewalld
systemctl stop firewalld.service
systemctl disable firewalld.service
iptables -nL
sed -i -e's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

### <a name="disable-cpupower"></a>Отключить кпуповер

```bash
service cpupower status
if enabled, disable it:
service cpupower stop
sudo systemctl disable cpupower
```

### <a name="configure-walinuxagent"></a>Настройка WALinuxAgent

```bash
sed -i -e 's/# OS.EnableRDMA=y/OS.EnableRDMA=y/g' /etc/waagent.conf
```
При необходимости WALinuxAgent может быть отключен как шаг перед заданием и включен назад после задания для максимальной доступности ресурсов виртуальной машины для рабочей нагрузки HPC.


## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о [включении InfiniBand](enable-infiniband.md) на виртуальных машинах серии [H](../../sizes-hpc.md) и [N](../../sizes-gpu.md) с поддержкой InfiniBand.
- Узнайте больше об установке различных [поддерживаемых библиотек MPI](setup-mpi.md) и их оптимальной конфигурации на виртуальных машинах.
- Дополнительные сведения об оптимальной настройке рабочих нагрузок для производительности и масштабируемости см. в статье с обзором виртуальных машин серии [HB](hb-series-overview.md) и [HC](hc-series-overview.md).
- Ознакомьтесь с последними объявлениями и некоторыми примерами HPC, а также результатами в [блогах технического сообщества службы вычислений](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Сведения о более высоком уровне архитектурного представления выполнения рабочих нагрузок HPC см. в статье [Высокопроизводительные вычисления (HPC) в Azure](/azure/architecture/topics/high-performance-computing/).
