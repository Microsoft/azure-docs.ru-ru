---
title: Получение данных из кэша Azure HPC — копирование вручную
description: Как использовать команды CP для перемещения данных в целевое хранилище BLOB-объектов в кэше HPC Azure
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 10/30/2019
ms.author: v-erkel
ms.openlocfilehash: f96a0fa264124f9d050e667b003d98579da63b77
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87092343"
---
# <a name="azure-hpc-cache-data-ingest---manual-copy-method"></a>Прием данных из кэша Azure HPC — метод копирования вручную

В этой статье приведены подробные инструкции по копированию данных вручную в контейнер хранилища BLOB-объектов для использования с кэшем HPC Azure. Для оптимизации скорости копирования используются многопоточные параллельные операции.

Дополнительные сведения о перемещении данных в хранилище BLOB-объектов для кэша HPC Azure см. в статье [Перемещение данных в хранилище BLOB-объектов Azure](hpc-cache-ingest.md).

## <a name="simple-copy-example"></a>Пример простого копирования

Можно вручную создать многопоточные операции копирования на клиентском компьютере, запустив более одной команды копирования одновременно в фоновом режиме для предопределенных наборов файлов или путей.

Команда Linux/UNIX ``cp`` содержит аргумент ``-p`` для сохранения прав владения и метаданных времени. Добавление этого аргумента в приведенные ниже команды является необязательным. (При добавлении аргумента увеличивается число вызовов файловой системы, отправленных клиентом в целевую файловую систему для изменения метаданных.)

В этом простом примере два файла копируются в параллельном режиме:

```bash
cp /mnt/source/file1 /mnt/destination1/ & cp /mnt/source/file2 /mnt/destination1/ &
```

После запуска этой команды команда `jobs` покажет, что выполняются два потока.

## <a name="copy-data-with-predictable-file-names"></a>Копирование данных с предсказуемыми именами файлов

Если имена файлов являются прогнозируемыми, можно использовать выражения для создания параллельных потоков копирования.

Например, если каталог содержит 1000 файлов, последовательно пронумерованных от `0001` до `1000` , можно использовать следующие выражения для создания 10 параллельных потоков, каждый из которых копирует файлы 100:

```bash
cp /mnt/source/file0* /mnt/destination1/ & \
cp /mnt/source/file1* /mnt/destination1/ & \
cp /mnt/source/file2* /mnt/destination1/ & \
cp /mnt/source/file3* /mnt/destination1/ & \
cp /mnt/source/file4* /mnt/destination1/ & \
cp /mnt/source/file5* /mnt/destination1/ & \
cp /mnt/source/file6* /mnt/destination1/ & \
cp /mnt/source/file7* /mnt/destination1/ & \
cp /mnt/source/file8* /mnt/destination1/ & \
cp /mnt/source/file9* /mnt/destination1/
```

## <a name="copy-data-with-unstructured-file-names"></a>Копировать данные с неструктурированными именами файлов

Если структура именования файлов не является прогнозируемой, можно сгруппировать файлы по именам каталогов.

В этом примере осуществляется сбор целых каталогов для отправки в команды ``cp``, выполняемые в качестве фоновых задач.

```bash
/root
|-/dir1
| |-/dir1a
| |-/dir1b
| |-/dir1c
   |-/dir1c1
|-/dir1d
```

После сбора файлов можно выполнить команды параллельного копирования, чтобы рекурсивно скопировать подкаталоги и все их содержимое:

```bash
cp /mnt/source/* /mnt/destination/
mkdir -p /mnt/destination/dir1 && cp /mnt/source/dir1/* mnt/destination/dir1/ &
cp -R /mnt/source/dir1/dir1a /mnt/destination/dir1/ &
cp -R /mnt/source/dir1/dir1b /mnt/destination/dir1/ &
cp -R /mnt/source/dir1/dir1c /mnt/destination/dir1/ & # this command copies dir1c1 via recursion
cp -R /mnt/source/dir1/dir1d /mnt/destination/dir1/ &
```

## <a name="when-to-add-mount-points"></a>Когда добавлять точки подключения

При наличии достаточного количества параллельных потоков для одной конечной точки монтирования файловой системы будет существовать точка, где Добавление дополнительных потоков не даст больше пропускной способности. (Пропускная способность будет измеряться в файлах/сек или байт/с в зависимости от типа данных.) Или, что еще хуже, иногда может вызвать снижение пропускной способности.

В этом случае можно добавить точки подключения на стороне клиента к другим адресам подключения кэша HPC Azure, используя тот же путь подключения удаленной файловой системы:

```bash
10.1.0.100:/nfs on /mnt/sourcetype nfs (rw,vers=3,proto=tcp,addr=10.1.0.100)
10.1.1.101:/nfs on /mnt/destination1type nfs (rw,vers=3,proto=tcp,addr=10.1.1.101)
10.1.1.102:/nfs on /mnt/destination2type nfs (rw,vers=3,proto=tcp,addr=10.1.1.102)
10.1.1.103:/nfs on /mnt/destination3type nfs (rw,vers=3,proto=tcp,addr=10.1.1.103)
```

Добавление точек подключения на стороне клиента позволяет разделить дополнительные команды копирования на дополнительные точки подключения `/mnt/destination[1-3]`, что позволяет повысить уровень параллелизма.

Например, если файлы очень велики, можно определить для команд копирования использование различных путей назначения, параллельно отправив дополнительные команды из клиента, выполняющего копирование.

```bash
cp /mnt/source/file0* /mnt/destination1/ & \
cp /mnt/source/file1* /mnt/destination2/ & \
cp /mnt/source/file2* /mnt/destination3/ & \
cp /mnt/source/file3* /mnt/destination1/ & \
cp /mnt/source/file4* /mnt/destination2/ & \
cp /mnt/source/file5* /mnt/destination3/ & \
cp /mnt/source/file6* /mnt/destination1/ & \
cp /mnt/source/file7* /mnt/destination2/ & \
cp /mnt/source/file8* /mnt/destination3/ & \
```

В приведенном выше примере процессы копирования файлов клиента направлены на все три целевые точки подключения.

## <a name="when-to-add-clients"></a>Когда следует добавлять клиенты

Наконец, когда вы достигли предела возможностей клиента, добавление потоков копирования или точек подключения не даст дополнительного увеличения числа файлов в секунду или байт в секунду. В этой ситуации можно развернуть другой клиент с тем же набором точек подключения, для которых будут выполняться собственные наборы процессов копирования файлов.

Пример

```bash
Client1: cp -R /mnt/source/dir1/dir1a /mnt/destination/dir1/ &
Client1: cp -R /mnt/source/dir2/dir2a /mnt/destination/dir2/ &
Client1: cp -R /mnt/source/dir3/dir3a /mnt/destination/dir3/ &

Client2: cp -R /mnt/source/dir1/dir1b /mnt/destination/dir1/ &
Client2: cp -R /mnt/source/dir2/dir2b /mnt/destination/dir2/ &
Client2: cp -R /mnt/source/dir3/dir3b /mnt/destination/dir3/ &

Client3: cp -R /mnt/source/dir1/dir1c /mnt/destination/dir1/ &
Client3: cp -R /mnt/source/dir2/dir2c /mnt/destination/dir2/ &
Client3: cp -R /mnt/source/dir3/dir3c /mnt/destination/dir3/ &

Client4: cp -R /mnt/source/dir1/dir1d /mnt/destination/dir1/ &
Client4: cp -R /mnt/source/dir2/dir2d /mnt/destination/dir2/ &
Client4: cp -R /mnt/source/dir3/dir3d /mnt/destination/dir3/ &
```

## <a name="create-file-manifests"></a>Создание манифестов файлов

После того как вы понимаете описанные выше подходы (несколько копий-потоков на каждый клиент, несколько клиентов на исходную файловую систему в сети), рассмотрите следующие рекомендации: сборка манифестов файлов и их использование с командами копирования на нескольких клиентах.

В этом сценарии для создания манифестов файлов или каталогов используется команда UNIX ``find``:

```bash
user@build:/mnt/source > find . -mindepth 4 -maxdepth 4 -type d
./atj5b55c53be6-01/support/gsi/2018-07-22T21:12:06EDT
./atj5b55c53be6-01/support/pcap/2018-07-23T01:34:57UTC
./atj5b55c53be6-01/support/trace/rolling
./atj5b55c53be6-03/support/gsi/2018-07-22T21:12:06EDT
./atj5b55c53be6-03/support/pcap/2018-07-23T01:34:57UTC
./atj5b55c53be6-03/support/trace/rolling
./atj5b55c53be6-02/support/gsi/2018-07-22T21:12:06EDT
./atj5b55c53be6-02/support/pcap/2018-07-23T01:34:57UTC
./atj5b55c53be6-02/support/trace/rolling
```

Перенаправьте этот результат в файл: `find . -mindepth 4 -maxdepth 4 -type d > /tmp/foo`

Затем можно выполнить итерацию по манифесту, используя команды BASH для подсчета файлов и определения размеров подкаталогов:

```bash
ben@xlcycl1:/sps/internal/atj5b5ab44b7f > for i in $(cat /tmp/foo); do echo " `find ${i} |wc -l` `du -sh ${i}`"; done
244    3.5M    ./atj5b5ab44b7f-02/support/gsi/2018-07-18T00:07:03EDT
9      172K    ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-18T05:01:00UTC
124    5.8M    ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-19T01:01:01UTC
152    15M     ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-20T01:01:00UTC
131    13M     ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-20T21:59:41UTC_partial
789    6.2M    ./atj5b5ab44b7f-02/support/gsi/2018-07-20T21:59:41UTC
134    12M     ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-20T22:22:55UTC_hpccache_catchup
7      16K     ./atj5b5ab44b7f-02/support/pcap/2018-07-18T17:12:19UTC
8      83K     ./atj5b5ab44b7f-02/support/pcap/2018-07-18T17:17:17UTC
575    7.7M    ./atj5b5ab44b7f-02/support/cores/armada_main.2000.1531980253.gsi
33     4.4G    ./atj5b5ab44b7f-02/support/trace/rolling
281    6.6M    ./atj5b5ab44b7f-01/support/gsi/2018-07-18T00:07:03EDT
15     182K    ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-18T05:01:00UTC
244    17M     ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-19T01:01:01UTC
299    31M     ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-20T01:01:00UTC
256    29M     ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-20T21:59:41UTC_partial
889    7.7M    ./atj5b5ab44b7f-01/support/gsi/2018-07-20T21:59:41UTC
262    29M     ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-20T22:22:55UTC_hpccache_catchup
11     248K    ./atj5b5ab44b7f-01/support/pcap/2018-07-18T17:12:19UTC
11     88K     ./atj5b5ab44b7f-01/support/pcap/2018-07-18T17:17:17UTC
645    11M     ./atj5b5ab44b7f-01/support/cores/armada_main.2019.1531980253.gsi
33     4.0G    ./atj5b5ab44b7f-01/support/trace/rolling
244    2.1M    ./atj5b5ab44b7f-03/support/gsi/2018-07-18T00:07:03EDT
9      158K    ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-18T05:01:00UTC
124    5.3M    ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-19T01:01:01UTC
152    15M     ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-20T01:01:00UTC
131    12M     ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-20T21:59:41UTC_partial
789    8.4M    ./atj5b5ab44b7f-03/support/gsi/2018-07-20T21:59:41UTC
134    14M     ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-20T22:25:58UTC_hpccache_catchup
7      159K    ./atj5b5ab44b7f-03/support/pcap/2018-07-18T17:12:19UTC
7      157K    ./atj5b5ab44b7f-03/support/pcap/2018-07-18T17:17:17UTC
576    12M     ./atj5b5ab44b7f-03/support/cores/armada_main.2013.1531980253.gsi
33     2.8G    ./atj5b5ab44b7f-03/support/trace/rolling
```

Наконец, вы должны создавать фактические команды копирования файлов для клиентов.

При наличии четырех клиентов используйте следующую команду:

```bash
for i in 1 2 3 4 ; do sed -n ${i}~4p /tmp/foo > /tmp/client${i}; done
```

Если у вас есть пять клиентов, используйте примерно следующее:

```bash
for i in 1 2 3 4 5; do sed -n ${i}~5p /tmp/foo > /tmp/client${i}; done
```

И для шести.... При необходимости экстраполяция.

```bash
for i in 1 2 3 4 5 6; do sed -n ${i}~6p /tmp/foo > /tmp/client${i}; done
```

Вы получите *N* файлов (по одному для каждого из ваших *N* клиентов), у которых есть пути к каталогам 4-го уровня, полученные как часть выходных данных команды `find`.

Используйте каждый файл для создания команды копирования:

```bash
for i in 1 2 3 4 5 6; do for j in $(cat /tmp/client${i}); do echo "cp -p -R /mnt/source/${j} /mnt/destination/${j}" >> /tmp/client${i}_copy_commands ; done; done
```

Вы получите *N* файлов (каждый с командой копирования в строке), которые могут быть запущены в качестве сценария BASH в клиенте.

Целью является параллельное выполнение нескольких потоков этих сценариев в каждом клиенте (одновременно в нескольких клиентах).
