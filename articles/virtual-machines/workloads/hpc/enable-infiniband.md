---
title: Включение Инифинбанд на виртуальных машинах HPC с помощью виртуальных машин Azure | Документация Майкрософт
description: Узнайте, как включить InfiniBand на виртуальных машинах Azure HPC.
author: vermagit
ms.service: virtual-machines
ms.subservice: hpc
ms.topic: article
ms.date: 11/06/2020
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: cd4d928217ceba80fa5ea0252a6ed20803a812d3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101666954"
---
# <a name="enable-infiniband"></a>Включение InfiniBand

Виртуальные машины серии [H](../../sizes-hpc.md) и [N](../../sizes-gpu.md) [с поддержкой RDMA](../../sizes-hpc.md#rdma-capable-instances) обмениваются данными по сети InfiniBand с низкой задержкой и высокой пропускной способностью. Поддержка RDMA в таком взаимодействии очень важна для повышения масштабируемости и производительности рабочих нагрузок распределенного узла HPC и ИИ. Виртуальные машины серии H и N с InfiniBand соединены в неблокируемое утолщенное дерево с небольшим размером программной архитектуры для оптимизированной и согласованной производительности RDMA.

Существует несколько способов включения InfiniBand на совместимых размерах виртуальных машин.

## <a name="vm-images-with-infiniband-drivers"></a>Образы виртуальных машин с драйверами InfiniBand
В разделе [образы виртуальных](configure.md#vm-images) машин приведен список поддерживаемых образов виртуальных машин в Marketplace, которые предварительно загружены с помощью драйверов InfiniBand (для ВИРТУАЛЬНЫХ машин SR-IOV или без SR-IOV) или могут быть настроены с помощью соответствующих драйверов.
Чтобы приступить к работе с [виртуальными машинами с поддержкой RDMA](../../sizes-hpc.md#rdma-capable-instances)на основе SR-IOV, [CentOS-HPC версии 7,6 или более поздней](https://techcommunity.microsoft.com/t5/Azure-Compute/CentOS-HPC-VM-Image-for-SR-IOV-enabled-Azure-HPC-VMs/ba-p/665557) версии в Marketplace — это самый простой способ начать работу.
Образы виртуальных машин Ubuntu можно настроить с помощью правильных драйверов для виртуальных машин с поддержкой SR-IOV и без SR-IOV, используя приведенные [здесь инструкции](https://techcommunity.microsoft.com/t5/azure-compute/configuring-infiniband-for-ubuntu-hpc-and-gpu-vms/ba-p/1221351).

## <a name="infiniband-driver-vm-extensions"></a>Расширения виртуальных машин драйвера InfiniBand
В Linux [расширение виртуальной машины инфинибанддриверлинукс](../../extensions/hpc-compute-infiniband-linux.md) можно использовать для установки драйверов Mellanox офед и включения InfiniBand на виртуальных машинах серии H и N с поддержкой SR-IOV.

В Windows [расширение виртуальной машины инфинибанддривервиндовс](../../extensions/hpc-compute-infiniband-windows.md) устанавливает драйверы Windows Network Direct (на виртуальных машинах без SR-IOV) или драйверы Mellanox офед (на виртуальных машинах SR-IOV) для подключения RDMA. В некоторых развертываниях экземпляров A8 и A9 расширение HpcVmDrivers добавляется автоматически. Обратите внимание, что расширение виртуальной машины HpcVmDrivers является устаревшим. Он не будет обновлен.

Чтобы добавить в виртуальную машину расширение виртуальной машины, можно использовать командлеты [Azure PowerShell](/powershell/azure/). Дополнительные сведения см. в статье [Обзор расширений и компонентов виртуальной машины под управлением Windows](../../extensions/overview.md). Вы также можете работать с расширениями для виртуальных машин, развернутых в рамках [классической модели развертывания](/previous-versions/azure/virtual-machines/windows/classic/agents-and-extensions-classic).

## <a name="manual-installation"></a>Установка вручную
[Драйверы Mellanox опенфабрикс (офед)](https://www.mellanox.com/products/InfiniBand-VPI-Software) можно установить вручную на виртуальных машинах серии [H](../../sizes-hpc.md) и [N](../../sizes-gpu.md) , [поддерживающих SR-IOV](../../sizes-hpc.md#rdma-capable-instances) .

### <a name="linux"></a>Linux
[Драйверы офед для Linux](https://www.mellanox.com/products/infiniband-drivers/linux/mlnx_ofed) можно установить с помощью приведенного ниже примера. Хотя здесь приведен пример для RHEL/CentOS, но эти действия являются общими и могут использоваться для любой совместимой операционной системы Linux, такой как Ubuntu (16,04, 18,04 19,04, 20,04) и SLES (12 SP4 и 15). Дополнительные примеры для других дистрибутивов см. в [репозитории азпк-Images](https://github.com/Azure/azhpc-images/blob/master/ubuntu/ubuntu-18.x/ubuntu-18.04-hpc/install_mellanoxofed.sh). Кроме того, драйверы для папки "Входящие" также работают, но драйверы Mellanox ОФЕД предоставляют больше возможностей.

```bash
MLNX_OFED_DOWNLOAD_URL=http://content.mellanox.com/ofed/MLNX_OFED-5.0-2.1.8.0/MLNX_OFED_LINUX-5.0-2.1.8.0-rhel7.7-x86_64.tgz
# Optionally verify checksum
wget --retry-connrefused --tries=3 --waitretry=5 $MLNX_OFED_DOWNLOAD_URL
tar zxvf MLNX_OFED_LINUX-5.0-2.1.8.0-rhel7.7-x86_64.tgz

KERNEL=( $(rpm -q kernel | sed 's/kernel\-//g') )
KERNEL=${KERNEL[-1]}
# Uncomment the lines below if you are running this on a VM
#RELEASE=( $(cat /etc/centos-release | awk '{print $4}') )
#yum -y install http://olcentgbl.trafficmanager.net/centos/${RELEASE}/updates/x86_64/kernel-devel-${KERNEL}.rpm
yum install -y kernel-devel-${KERNEL}
./MLNX_OFED_LINUX-5.0-2.1.8.0-rhel7.7-x86_64/mlnxofedinstall --kernel $KERNEL --kernel-sources /usr/src/kernels/${KERNEL} --add-kernel-support --skip-repo
```

### <a name="windows"></a>Windows
Для Windows Скачайте и установите [драйверы MELLANOX офед для Windows](https://www.mellanox.com/products/adapter-software/ethernet/windows/winof-2).

## <a name="enable-ip-over-infiniband-ib"></a>Включить использование IP-адресов через InfiniBand (превышено)
Если вы планируете выполнять задания MPI, обычно не требуется использование IPoIB. Библиотека MPI будет использовать интерфейс команд для обмена данными (если только вы явно не используете канал TCP/IP в библиотеке MPI). Но если у вас есть приложение, использующее протокол TCP/IP для обмена данными, и вы хотите выполнить переработку, можно использовать для этого интерфейс в качестве средства. Используйте следующие команды (для RHEL/CentOS), чтобы включить IP-адрес через InfiniBand.

```bash
sudo sed -i -e 's/# OS.EnableRDMA=n/OS.EnableRDMA=y/g' /etc/waagent.conf
sudo systemctl restart waagent
```

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше об установке различных [поддерживаемых библиотек MPI](setup-mpi.md) и их оптимальной конфигурации на виртуальных машинах.
- Дополнительные сведения об оптимальной настройке рабочих нагрузок для производительности и масштабируемости см. в статье с обзором виртуальных машин серии [HB](hb-series-overview.md) и [HC](hc-series-overview.md).
- Ознакомьтесь с последними объявлениями и некоторыми примерами HPC, а также результатами в [блогах технического сообщества службы вычислений](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Сведения о более высоком уровне архитектурного представления выполнения рабочих нагрузок HPC см. в статье [Высокопроизводительные вычисления (HPC) в Azure](/azure/architecture/topics/high-performance-computing/).
