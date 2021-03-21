---
title: DPDK в виртуальной машине Linux в Azure | Документация Майкрософт
description: Узнайте о преимуществах пакета средств разработки для плоскости данных (ДПДК) и о настройке ДПДК на виртуальной машине Linux.
services: virtual-network
documentationcenter: na
author: laxmanrb
manager: gedegrac
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/12/2020
ms.author: labattul
ms.openlocfilehash: 3b4d66525ec52ef2382dfbe97bc09278e35b31fb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102124675"
---
# <a name="set-up-dpdk-in-a-linux-virtual-machine"></a>Настройка DPDK в виртуальной машине Linux

Набор Data Plane Development Kit (DPDK) в Azure предоставляет более быструю платформу пакетной обработки в пользовательском пространстве для высокопроизводительных приложений. Эта платформа обходит сетевой стек ядра виртуальной машины.

В типичной обработке пакетов с использованием сетевого стека ядра применяется процесс на основе прерываний. Когда сетевой интерфейс принимает входящие пакеты, происходит прерывание работы ядра для обработки пакета и переключение контекста из пространства ядра в пространство пользователя. DPDK устраняет переключение контекста и метод взаимодействия на основе прерываний в пользу реализации в пользовательском пространстве с использованием драйверов режима опроса (PMD) для быстрой обработки пакетов.

DPDK состоит из наборов пользовательских библиотек, которые предоставляют доступ к ресурсам нижнего уровня. Эти ресурсы могут включать оборудование, логические ядра, управление памятью и драйверы режима опроса для сетевых карт.

DPDK может работать на виртуальных машинах Azure, поддерживающих несколько дистрибутивов операционной системы. Ключевое отличие DPDK с точки зрения производительности — реализация виртуализации сетевых функций. Эти реализации могут принять форму сетевых виртуальных модулей (NVA), таких как виртуальный маршрутизатор, брандмауэр, VPN, подсистема балансировки нагрузки, развитое пакетное ядро и приложения типа "отказ в обслуживании" (DDoS).

## <a name="benefit"></a>Преимущество

**Большее количество пакетов в секунду (PPS)** . Обход ядра и управление пакетами в пространстве пользователя уменьшают количество циклов за счет исключения переключения контекста. Благодаря этому также повышается скорость обработки пакетов в секунду на виртуальных машинах Linux в Azure.


## <a name="supported-operating-systems"></a>Поддерживаемые операционные системы

Поддерживаются следующие дистрибутивы из Azure Marketplace:

| ОС Linux     | Версия ядра               | 
|--------------|---------------------------   |
| Ubuntu 16.04 | 4.15.0-1014-azure+           | 
| Ubuntu 18.04 | 4.15.0-1014-azure+           |
| SLES 15 SP1  | 4.12.14-8.19-Azure +          | 
| RHEL 7.5     | 3.10.0-862.11.6.el7.x86_64+  | 
| CentOS 7.5   | 3.10.0-862.11.6.el7.x86_64+  | 

**Поддержка настраиваемого ядра**

Любую версию ядра Linux, не указанную в списке, можно найти в [репозитории GitHub](https://github.com/microsoft/azure-linux-kernel). Чтобы получить дополнительные сведения, обратитесь по адресу [aznetdpdk@microsoft.com](mailto:aznetdpdk@microsoft.com). 

## <a name="region-support"></a>Поддержка регионов

Все регионы Azure поддерживают DPDK.

## <a name="prerequisites"></a>Предварительные требования

На виртуальной машине Linux необходимо включить ускоренную работу в сети. Виртуальная машина должна иметь по крайней мере два сетевых интерфейса с одним интерфейсом для управления. Включение ускоренной сети в интерфейсе управления не рекомендуется. Сведения о создании виртуальной машины Linux с ускоренной сетью см. в [этой статье](create-vm-accelerated-networking-cli.md).

## <a name="install-dpdk-dependencies"></a>Установка зависимостей DPDK

### <a name="ubuntu-1604"></a>Ubuntu 16.04

```bash
sudo add-apt-repository ppa:canonical-server/dpdk-azure -y
sudo apt-get update
sudo apt-get install -y librdmacm-dev librdmacm1 build-essential libnuma-dev libmnl-dev
```

### <a name="ubuntu-1804"></a>Ubuntu 18.04

```bash
sudo add-apt-repository ppa:canonical-server/dpdk-azure -y
sudo apt-get update
sudo apt-get install -y librdmacm-dev librdmacm1 build-essential libnuma-dev libmnl-dev
```

### <a name="rhel75centos-75"></a>RHEL7.5/CentOS 7.5

```bash
yum -y groupinstall "Infiniband Support"
sudo dracut --add-drivers "mlx4_en mlx4_ib mlx5_ib" -f
yum install -y gcc kernel-devel-`uname -r` numactl-devel.x86_64 librdmacm-devel libmnl-devel
```

### <a name="sles-15-sp1"></a>SLES 15 SP1

**Ядро Azure**

```bash
zypper  \
  --no-gpg-checks \
  --non-interactive \
  --gpg-auto-import-keys install kernel-azure kernel-devel-azure gcc make libnuma-devel numactl librdmacm1 rdma-core-devel
```

**Ядро по умолчанию**

```bash
zypper \
  --no-gpg-checks \
  --non-interactive \
  --gpg-auto-import-keys install kernel-default-devel gcc make libnuma-devel numactl librdmacm1 rdma-core-devel
```

## <a name="set-up-the-virtual-machine-environment-once"></a>Настройка среды виртуальной машины (один раз)

1. [Скачайте последнюю версию DPDK](https://core.dpdk.org/download). Для Azure требуется версия 18.11 LTS или 19.11 LTS.
2. Создайте конфигурацию по умолчанию с помощью команды `make config T=x86_64-native-linuxapp-gcc`.
3. Включите драйверы PMD Mellanox в созданной конфигурации с помощью команды `sed -ri 's,(MLX._PMD=)n,\1y,' build/.config`.
4. Выполните сборку с помощью команды `make`.
5. Выполните установку с помощью команды `make install DESTDIR=<output folder>`.

## <a name="configure-the-runtime-environment"></a>Настройка среды выполнения

После перезагрузки выполните один раз следующие команды:

1. Hugepages.

   * Настройте функцию больших страниц, выполнив следующую команду (один раз для каждого узла NUMA):

     ```bash
     echo 1024 | sudo tee /sys/devices/system/node/node*/hugepages/hugepages-2048kB/nr_hugepages
     ```

   * Создайте каталог для подключения с помощью команды `mkdir /mnt/huge`.
   * Подключите большие страницы с помощью команды `mount -t hugetlbfs nodev /mnt/huge`.
   * Проверьте, зарезервированы ли большие страницы, с помощью команды `grep Huge /proc/meminfo`.

     > [ПРИМЕЧАНИЕ] Вы можете изменить файл grub, чтобы большие страницы были зарезервированы при загрузке, выполнив [эти инструкции](https://dpdk.org/doc/guides/linux_gsg/sys_reqs.html#use-of-hugepages-in-the-linux-environment) для DPDK. Инструкции находятся в нижней части страницы. При использовании виртуальной машины Linux Azure измените файлы в разделе **/etc/config/grub.d**, чтобы резервировать большие страницы при перезагрузках.

2. MAC- и IP-адреса Эти адреса сетевых интерфейсов можно просмотреть с помощью команды `ifconfig –a`. Сетевые интерфейсы *VF* и *NETVSC* имеют одинаковый MAC-адрес, но лишь сетевой интерфейс *NETVSC* имеет IP-адрес. Интерфейсы *VF* работают как подчиненные интерфейсов *NETVSC*.

3. PCI-адреса

   * Узнайте, какой PCI-адрес нужно использовать для *VF*, с помощью команды `ethtool -i <vf interface name>`.
   * Если *eth0* имеет ускоренную сеть, убедитесь, что testpmd случайно не использует устройство PCI *VF* для *eth0*. Если приложение DPDK случайно захватило сетевой интерфейс управления, что привело к потери SSH-подключения, используйте последовательную консоль для прекращения работы приложения DPDK. Последовательную консоль также можно использовать для остановки или запуска виртуальной машины.

4. Загружайте *ibuverbs* при каждой перезагрузке с помощью команды `modprobe -a ib_uverbs`. Только для SLES 15 загрузите *mlx4_ib* с помощью команды `modprobe -a mlx4_ib`.

## <a name="failsafe-pmd"></a>Отказоустойчивый драйвер PMD

Приложения DPDK должны работать через отказоустойчивый драйвер PMD, доступный в Azure. Если приложение работает непосредственно через *VF* PMD, оно не получает **все** пакеты, предназначенные для виртуальной машины, так как некоторые из них отображаются в синтетическом интерфейсе. 

Запуск приложения DPDK через отказоустойчивый драйвер PMD гарантирует, что приложение получит все пакеты, которые были ему отправлены. Это также позволяет убедиться, что приложение продолжает выполняться в режиме DPDK, даже если VF отменяется при обслуживании узла. Дополнительные сведения об отказоустойчивом PMD см. в разделе [Fail-safe poll mode driver library](https://doc.dpdk.org/guides/nics/fail_safe.html) (Библиотека отказоустойчивого драйвера режима опроса).

## <a name="run-testpmd"></a>Запуск testpmd

Чтобы запустить *testpmd* в режиме root, выполните сначала команду `sudo`.

### <a name="basic-sanity-check-failsafe-adapter-initialization"></a>Базовый уровень: проверка работоспособности, инициализация отказоустойчивого адаптера

1. Выполните следующие команды, чтобы запустить приложение testpmdс с одним портом:

   ```bash
   testpmd -w <pci address from previous step> \
     --vdev="net_vdev_netvsc0,iface=eth1" \
     -- -i \
     --port-topology=chained
    ```

2. Чтобы запустить приложение testpmdс с двумя портами, выполните следующие команды:

   ```bash
   testpmd -w <pci address nic1> \
   -w <pci address nic2> \
   --vdev="net_vdev_netvsc0,iface=eth1" \
   --vdev="net_vdev_netvsc1,iface=eth2" \
   -- -i
   ```

   При запуске testpmd с более чем двумя сетевыми адаптерами аргумент `--vdev` выглядит подобно следующему шаблону: `net_vdev_netvsc<id>,iface=<vf’s pairing eth>`.

3.  После запуска testpmd выполните команду `show port info all`, чтобы проверить сведения о порте. Вы увидите один или два порта DPDK со значением net_failsafe (не *net_mlx4*).
4.  Используйте команду `start <port> /stop <port>` для запуска трафика.

Предыдущие команды запускают *testpmd* в интерактивном режиме, что и рекомендуется, чтобы попробовать выполнить команды testpmd.

### <a name="basic-single-sendersingle-receiver"></a>Базовый уровень: одиночные отправитель и получатель

Следующие команды периодически выводят статистику пакетов в секунду:

1. Выполните приведенную ниже команду на стороне отправки:

   ```bash
   testpmd \
     -l <core-list> \
     -n <num of mem channels> \
     -w <pci address of the device you plan to use> \
     --vdev="net_vdev_netvsc<id>,iface=<the iface to attach to>" \
     -- --port-topology=chained \
     --nb-cores <number of cores to use for test pmd> \
     --forward-mode=txonly \
     --eth-peer=<port id>,<receiver peer MAC address> \
     --stats-period <display interval in seconds>
   ```

2. Выполните приведенную ниже команду на стороне приема:

   ```bash
   testpmd \
     -l <core-list> \
     -n <num of mem channels> \
     -w <pci address of the device you plan to use> \
     --vdev="net_vdev_netvsc<id>,iface=<the iface to attach to>" \
     -- --port-topology=chained \
     --nb-cores <number of cores to use for test pmd> \
     --forward-mode=rxonly \
     --eth-peer=<port id>,<sender peer MAC address> \
     --stats-period <display interval in seconds>
   ```

При выполнении предыдущих команд в виртуальной машине измените адреса *IP_SRC_ADDR* и *IP_DST_ADDR* в `app/test-pmd/txonly.c`, чтобы они соответствовали фактическому IP-адресу виртуальных машин перед сборкой. В противном случае пакеты будут удалены, прежде чем они достигнут получателя.

### <a name="advanced-single-sendersingle-forwarder"></a>Дополнительно: одиночные отправитель и сервер пересылки
Следующие команды периодически выводят статистику пакетов в секунду:

1. Выполните приведенную ниже команду на стороне отправки:

   ```bash
   testpmd \
     -l <core-list> \
     -n <num of mem channels> \
     -w <pci address of the device you plan to use> \
     --vdev="net_vdev_netvsc<id>,iface=<the iface to attach to>" \
     -- --port-topology=chained \
     --nb-cores <number of cores to use for test pmd> \
     --forward-mode=txonly \
     --eth-peer=<port id>,<receiver peer MAC address> \
     --stats-period <display interval in seconds>
    ```

2. Выполните приведенную ниже команду на стороне пересылки:

   ```bash
   testpmd \
     -l <core-list> \
     -n <num of mem channels> \
     -w <pci address NIC1> \
     -w <pci address NIC2> \
     --vdev="net_vdev_netvsc<id>,iface=<the iface to attach to>" \
     --vdev="net_vdev_netvsc<2nd id>,iface=<2nd iface to attach to>" (you need as many --vdev arguments as the number of devices used by testpmd, in this case) \
     -- --nb-cores <number of cores to use for test pmd> \
     --forward-mode=io \
     --eth-peer=<recv port id>,<sender peer MAC address> \
     --stats-period <display interval in seconds>
    ```

При выполнении предыдущих команд в виртуальной машине измените адреса *IP_SRC_ADDR* и *IP_DST_ADDR* в `app/test-pmd/txonly.c`, чтобы они соответствовали фактическому IP-адресу виртуальных машин перед сборкой. В противном случае пакеты будут удалены, прежде чем они достигнут сервера пересылки. Третий компьютер не сможет получить отправленный трафик, потому что сервер пересылки *testpmd* не изменяет адреса уровня 3, если вы не внесли изменения в код.

## <a name="references"></a>Ссылки

* [Параметры командной строки EAL](https://dpdk.org/doc/guides/testpmd_app_ug/run_app.html#eal-command-line-options)
* [Параметры командной строки testpmd](https://dpdk.org/doc/guides/testpmd_app_ug/run_app.html#testpmd-command-line-options)
