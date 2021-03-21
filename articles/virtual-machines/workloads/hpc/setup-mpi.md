---
title: Настройка интерфейса передачи сообщений для HPC — виртуальные машины Azure | Документация Майкрософт
description: Узнайте, как настроить MPI для HPC в Azure.
author: vermagit
ms.service: virtual-machines
ms.subservice: hpc
ms.topic: article
ms.date: 03/18/2021
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 8f071dfe817d15b745575fbfb70ff662a643db70
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104721359"
---
# <a name="set-up-message-passing-interface-for-hpc"></a>Настройка интерфейса передачи сообщений для HPC

[Интерфейс передачи сообщений (MPI)](https://en.wikipedia.org/wiki/Message_Passing_Interface) — это открытая библиотека и де-факто Standard для параллелизации распределенной памяти. Обычно он используется во многих рабочих нагрузках HPC. Рабочие нагрузки HPC на виртуальных машинах серии [H](../../sizes-hpc.md) и серии [N](../../sizes-gpu.md) с [поддержкой RDMA](../../sizes-hpc.md#rdma-capable-instances) могут использовать MPI для обмена данными через сеть InfiniBand с низкой задержкой и с высокой пропускной способностью.
- Размеры виртуальных машин с поддержкой SR-IOV в Azure поддерживают почти любую версию MPI для использования с Mellanox ОФЕД.
- На виртуальных машинах, не использующих SR-IOV, поддерживаемые реализации MPI используют интерфейс Microsoft Network Direct (ND) для обмена данными между виртуальными машинами. Таким образом, поддерживаются только версии Microsoft MPI (MS-MPI) 2012 R2 и более поздних версий и Intel MPI 5. x. Более поздние версии (2017, 2018) библиотеки Intel MPI Runtime могут быть несовместимыми с драйверами Azure RDMA.

Для [виртуальных машин с поддержкой RDMA](../../sizes-hpc.md#rdma-capable-instances), поддерживающих SR-IOV, подходят [образы виртуальных машин CentOS-HPC](configure.md#centos-hpc-vm-images) версии 7,6 и более поздних. Эти образы виртуальных машин оптимизированы и предварительно загружены с драйверами ОФЕД для RDMA и различными часто используемыми библиотеками MPI и инженерными пакетами для научных вычислений и являются самым простым способом приступить к работе.

Хотя здесь приведены примеры для RHEL/CentOS, но эти действия являются общими и могут использоваться для любой совместимой операционной системы Linux, такой как Ubuntu (16,04, 18,04 19,04, 20,04) и SLES (12 SP4 и 15). Дополнительные примеры настройки других реализаций MPI на других реализациях дистрибутивов см. в [репозитории азпк-Images](https://github.com/Azure/azhpc-images/blob/master/ubuntu/ubuntu-18.x/ubuntu-18.04-hpc/install_mpis.sh).

> [!NOTE]
> Выполнение заданий MPI на виртуальных машинах с поддержкой SR-IOV с определенными библиотеками MPI (например, платформой MPI) может потребовать настройки ключей разделов (p-Keys) в клиенте для изоляции и безопасности. Выполните действия, описанные в разделе [Обнаружение ключей разделов](#discover-partition-keys) , чтобы узнать о том, как определить значения p-ключа и правильно настроить их для задания MPI с помощью этой библиотеки MPI.

> [!NOTE]
> Ниже приведены примеры фрагментов кода. Рекомендуется использовать последние стабильные версии пакетов или ссылаться на [репозиторий азпк-Images](https://github.com/Azure/azhpc-images/blob/master/ubuntu/ubuntu-18.x/ubuntu-18.04-hpc/install_mpis.sh).

## <a name="ucx"></a>уккс

[Единая связь X (уккс)](https://github.com/openucx/ucx) — это платформа интерфейсов API взаимодействия для HPC. Она оптимизирована для связи MPI через InfiniBand и работает со многими реализациями MPI, такими как Опенмпи и МПИЧ.

```bash
wget https://github.com/openucx/ucx/releases/download/v1.4.0/ucx-1.4.0.tar.gz
tar -xvf ucx-1.4.0.tar.gz
cd ucx-1.4.0
./configure --prefix=<ucx-install-path>
make -j 8 && make install
```

> [!NOTE]
> В последних сборках УККС устранена [проблема](https://github.com/openucx/ucx/pull/5965) , при которой в наличии нескольких ИНТЕРФЕЙСОВ сетевой карты выбран правильный интерфейс InfiniBand. Дополнительные сведения о работе MPI через InfiniBand при включении ускоренной [сети на виртуальной](hb-hc-known-issues.md#accelerated-networking-on-hb-hc-hbv2-and-ndv2) машине.

## <a name="hpc-x"></a>HPC-X

[Набор программного обеспечения HPC-X](https://www.mellanox.com/products/hpc-x-toolkit) содержит УККС и хколл и может быть построен для уккс.

```bash
HPCX_VERSION="v2.6.0"
HPCX_DOWNLOAD_URL=https://azhpcstor.blob.core.windows.net/azhpc-images-store/hpcx-v2.6.0-gcc-MLNX_OFED_LINUX-5.0-1.0.0.0-redhat7.7-x86_64.tbz
get --retry-connrefused --tries=3 --waitretry=5 $HPCX_DOWNLOAD_URL
tar -xvf hpcx-${HPCX_VERSION}-gcc-MLNX_OFED_LINUX-5.0-1.0.0.0-redhat7.7-x86_64.tbz
mv hpcx-${HPCX_VERSION}-gcc-MLNX_OFED_LINUX-5.0-1.0.0.0-redhat7.7-x86_64 ${INSTALL_PREFIX}
HPCX_PATH=${INSTALL_PREFIX}/hpcx-${HPCX_VERSION}-gcc-MLNX_OFED_LINUX-5.0-1.0.0.0-redhat7.7-x86_64
```

Запуск HPC-X

```bash
${HPCX_PATH}mpirun -np 2 --map-by ppr:2:node -x UCX_TLS=rc ${HPCX_PATH}/ompi/tests/osu-micro-benchmarks-5.3.2/osu_latency
```
> [!NOTE] 
> При использовании HPC-X 2.7.4 + может потребоваться явно передавать LD_LIBRARY_PATH, если версия УККС в МОФЕД VS, а в HPC-X отличается.

## <a name="openmpi"></a>опенмпи

Установите УККС, как описано выше. ХКОЛЛ является частью [набора программного обеспечения HPC-X](https://www.mellanox.com/products/hpc-x-toolkit) и не требует специальной установки.

Опенмпи можно установить из пакетов, доступных в репозитории.

```bash
sudo yum install –y openmpi
```

Мы советуем создать новейший, стабильный выпуск Опенмпи с УККС.

```bash
OMPI_VERSION="4.0.3"
OMPI_DOWNLOAD_URL=https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-${OMPI_VERSION}.tar.gz
wget --retry-connrefused --tries=3 --waitretry=5 $OMPI_DOWNLOAD_URL
tar -xvf openmpi-${OMPI_VERSION}.tar.gz
cd openmpi-${OMPI_VERSION}
./configure --prefix=${INSTALL_PREFIX}/openmpi-${OMPI_VERSION} --with-ucx=${UCX_PATH} --with-hcoll=${HCOLL_PATH} --enable-mpirun-prefix-by-default --with-platform=contrib/platform/mellanox/optimized && make -j$(nproc) && make install
```

Для оптимальной производительности запустите Опенмпи с помощью `ucx` и `hcoll` .

```bash
${INSTALL_PREFIX}/bin/mpirun -np 2 --map-by node --hostfile ~/hostfile -mca pml ucx --mca btl ^vader,tcp,openib -x UCX_NET_DEVICES=mlx5_0:1  -x UCX_IB_PKEY=0x0003  ./osu_latency
```

Проверьте ключ раздела, как упоминалось выше.

## <a name="intel-mpi"></a>Intel MPI

Скачайте версию [Intel MPI](https://software.intel.com/mpi-library/choose-download). Измените переменную среды I_MPI_FABRICS в зависимости от версии.
- Intel MPI 2019 и 2021: используйте `I_MPI_FABRICS=shm:ofi` , `I_MPI_OFI_PROVIDER=mlx` . `mlx`Поставщик использует уккс. Использование команд было обнаружено как нестабильное и менее производительное. Дополнительные сведения см. в [статье течкоммунити](https://techcommunity.microsoft.com/t5/azure-compute/intelmpi-2019-on-azure-hpc-clusters/ba-p/1403149) .
- Intel MPI 2018: использование `I_MPI_FABRICS=shm:ofa`
- Intel MPI 2016: использование `I_MPI_DAPL_PROVIDER=ofa-v2-ib0`

### <a name="non-sr-iov-vms"></a>Виртуальные машины без SR-IOV
Для виртуальных машин, не использующих SR-IOV, пример загрузки [бесплатной оценочной версии](https://registrationcenter.intel.com/en/forms/?productid=1740) среды выполнения 5. x выглядит следующим образом:
```bash
wget http://registrationcenter-download.intel.com/akdlm/irc_nas/tec/9278/l_mpi_p_5.1.3.223.tgz
```
Шаги установки см. в [руководстве по установке библиотеки Intel MPI](https://registrationcenter-download.intel.com/akdlm/irc_nas/1718/INSTALL.html?lang=en&fileExt=.html).
При необходимости вы можете включить птраце для некорневых процессов, не использующих отладчик (требуется для самых последних версий Intel MPI).
```bash
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
```

### <a name="suse-linux"></a>SUSE Linux
Для SUSE Linux Enterprise Server версии образа виртуальной машины — SLES 12 SP3 для HPC, SLES 12 SP3 для HPC (Premium), SLES 12 SP1 для HPC, SLES 12 SP1 для HPC (Premium), SLES 12 SP4 и SLES 15, установлены драйверы RDMA и пакеты Intel MPI распределяются на виртуальной машине. Установите Intel MPI, выполнив следующую команду:
```bash
sudo rpm -v -i --nodeps /opt/intelMPI/intel_mpi_packages/*.rpm
```

## <a name="mvapich2"></a>MVAPICH2

Сборка MVAPICH2.

```bash
wget http://mvapich.cse.ohio-state.edu/download/mvapich/mv2/mvapich2-2.3.tar.gz
tar -xv mvapich2-2.3.tar.gz
cd mvapich2-2.3
./configure --prefix=${INSTALL_PREFIX}
make -j 8 && make install
```

Выполнение MVAPICH2.

```bash
${INSTALL_PREFIX}/bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=48 ./osu_latency
```

## <a name="platform-mpi"></a>Платформа MPI

Установите необходимые пакеты для платформы MPI Community Edition.

```bash
sudo yum install libstdc++.i686
sudo yum install glibc.i686
Download platform MPI at https://www.ibm.com/developerworks/downloads/im/mpi/index.html 
sudo ./platform_mpi-09.01.04.03r-ce.bin
```

Выполните процедуру установки.

Следующие команды являются примерами запуска MPI пингпонг и онлайн-обучение с использованием платформы MPI на виртуальных машинах HBv3 с помощью образов виртуальных машин CentOS-HPC 7,6, 7,8 и 8,1.

```bash
/opt/ibm/platform_mpi/bin/mpirun -hostlist 10.0.0.8:1,10.0.0.9:1 -np 2 -e MPI_IB_PKEY=0x800a  -ibv  /home/jijos/mpi-benchmarks/IMB-MPI1 pingpong
/opt/ibm/platform_mpi/bin/mpirun -hostlist 10.0.0.8:120,10.0.0.9:120 -np 240 -e MPI_IB_PKEY=0x800a  -ibv  /home/jijos/mpi-benchmarks/IMB-MPI1 allreduce -npmin 240
```


## <a name="mpich"></a>MPICH

Установите УККС, как описано выше. Сборка МПИЧ.

```bash
wget https://www.mpich.org/static/downloads/3.3/mpich-3.3.tar.gz
tar -xvf mpich-3.3.tar.gz
cd mpich-3.3
./configure --with-ucx=${UCX_PATH} --prefix=${INSTALL_PREFIX} --with-device=ch4:ucx
make -j 8 && make install
```

Выполнение МПИЧ.

```bash
${INSTALL_PREFIX}/bin/mpiexec -n 2 -hostfile ~/hostfile -env UCX_IB_PKEY=0x0003 -bind-to hwthread ./osu_latency
```

Проверьте ключ раздела, как упоминалось выше.

## <a name="osu-mpi-benchmarks"></a>Тесты производительности осу MPI

Скачайте [тесты производительности осу MPI](http://mvapich.cse.ohio-state.edu/benchmarks/) и распаковать.

```bash
wget http://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-5.5.tar.gz
tar –xvf osu-micro-benchmarks-5.5.tar.gz
cd osu-micro-benchmarks-5.5
```

Создавайте тесты производительности с помощью определенной библиотеки MPI:

```bash
CC=<mpi-install-path/bin/mpicc>CXX=<mpi-install-path/bin/mpicxx> ./configure 
make
```

Тесты MPI находятся в `mpi/` папке.


## <a name="discover-partition-keys"></a>Обнаружение ключей разделов

Обнаружение ключей разделов (p-Keys) для взаимодействия с другими виртуальными машинами в пределах одного клиента (группы доступности или масштабируемого набора виртуальных машин).

```bash
/sys/class/infiniband/mlx5_0/ports/1/pkeys/0
/sys/class/infiniband/mlx5_0/ports/1/pkeys/1
```

Чем больше из двух, тем ключ клиента следует использовать с MPI. Пример. Если используются следующие ключи p-Keys, 0x800b следует использовать с MPI.

```bash
cat /sys/class/infiniband/mlx5_0/ports/1/pkeys/0
0x800b
cat /sys/class/infiniband/mlx5_0/ports/1/pkeys/1
0x7fff
```

Используйте раздел, отличный от ключа раздела по умолчанию (0x7FFF). УККС требует, чтобы был сброшен параметр p-Key. Например, задайте UCX_IB_PKEY как 0x000b для 0x800b.

Кроме того, обратите внимание, что если клиент (Группа доступности или масштабируемый набор виртуальных машин) существует, то PKEY не изменится. Это справедливо даже при добавлении или удалении узлов. Новые клиенты получают разные PKEY.


## <a name="set-up-user-limits-for-mpi"></a>Настройка ограничений пользователей для MPI

Настройте ограничения пользователей для MPI.

```bash
cat << EOF | sudo tee -a /etc/security/limits.conf
*               hard    memlock         unlimited
*               soft    memlock         unlimited
*               hard    nofile          65535
*               soft    nofile          65535
EOF
```

## <a name="set-up-ssh-keys-for-mpi"></a>Настройка ключей SSH для MPI

Настройте ключи SSH для требуемых типов MPI.

```bash
ssh-keygen -f /home/$USER/.ssh/id_rsa -t rsa -N ''
cat << EOF > /home/$USER/.ssh/config
Host *
    StrictHostKeyChecking no
EOF
cat /home/$USER/.ssh/id_rsa.pub >> /home/$USER/.ssh/authorized_keys
chmod 600 /home/$USER/.ssh/authorized_keys
chmod 644 /home/$USER/.ssh/config
```

В приведенном выше синтаксисе предполагается наличие общего домашнего каталога, иначе каталог. ssh должен быть скопирован на каждый узел.

## <a name="next-steps"></a>Следующие шаги

- Сведения о виртуальных машинах серии [H](../../sizes-hpc.md) и [N](../../sizes-gpu.md) с [поддержкой InfiniBand](../../sizes-hpc.md#rdma-capable-instances)
- Ознакомьтесь с обзором [HBv3-Series](hbv3-series-overview.md) и [описанием серии HC](hc-series-overview.md).
- Ознакомьтесь с последними объявлениями, примерами рабочей нагрузки HPC и результатами производительности в [блогах сообщества разработчиков Azure](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Сведения о более высоком уровне архитектурного представления выполнения рабочих нагрузок HPC см. в статье [Высокопроизводительные вычисления (HPC) в Azure](/azure/architecture/topics/high-performance-computing/).
