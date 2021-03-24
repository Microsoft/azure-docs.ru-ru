---
title: Аварийное восстановление с помощью средства создания моментальных снимков с использованием приложений Azure для Azure NetApp Files | Документация Майкрософт
description: В этой статье объясняется, как выполнить аварийное восстановление при использовании средства создания моментальных снимков с единым приложением Azure, которое можно использовать с Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: Phil-Jensen
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 12/14/2020
ms.author: phjensen
ms.openlocfilehash: 554fa394179e7cfc5b86a2b50eb754547d137a44
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104870400"
---
# <a name="disaster-recovery-using-azure-application-consistent-snapshot-tool-preview"></a>Аварийное восстановление с помощью средства создания моментальных снимков для приложений Azure (Предварительная версия)

В этой статье объясняется, как выполнить аварийное восстановление при использовании средства создания моментальных снимков, совместимых с приложениями Azure, которые можно использовать с Azure NetApp Files.

> [!IMPORTANT]
> Эта операция применяется только к **большому экземпляру Azure** .

## <a name="introduction"></a>Введение

Платформа крупных экземпляров Azure также может иметь сайт аварийного восстановления, настроенный там, где можно реплицировать моментальные снимки тома хранилища.  Если моментальные снимки настроены правильно с помощью такой установки, можно выполнить аварийное восстановление на этом сайте.  Этот документ предназначен для руководства по выполнению аварийного восстановления для этой настройки.

## <a name="prerequisites-for-disaster-recovery-setup"></a>Необходимые условия для настройки аварийного восстановления

Перед планированием отработки отказа аварийного восстановления должны быть выполнены следующие предварительные требования.

- У вас есть узел аварийного восстановления, подготовленный на сайте аварийного восстановления. Существует два варианта аварийного восстановления. Одна из них — обычная DR, а другая — многоцелевая DR.
- Репликация хранилища работает. Группа Microsoft Operations выполняет настройку репликации хранилища во время автоматической подготовки аварийного восстановления. Вы можете отслеживать репликацию хранилища с помощью команды `azacsnap -c details --details replication` на сайте аварийного восстановления.
- Вы настроили и настроили моментальные снимки хранилища в основном расположении.
- У вас есть экземпляр HANA, установленный на сайте аварийного восстановления для первичного сайта с тем же идентификатором безопасности, что и у основного экземпляра.
- Вы прочитаете и понимаете процедуру отработки отказа АВАРИЙного восстановления, описанную в статье [SAP HANA (крупные экземпляры) высокий уровень доступности и аварийное восстановление](../virtual-machines/workloads/sap/hana-failover-procedure.md)
- Вы настроили и настроили моментальные снимки хранилища в расположении аварийного восстановления.
- Файл конфигурации (например,) был `DR.json` создан с томами хранилища аварийного восстановления и связанными сведениями на сервере аварийного восстановления.
- Вы выполнили действия на сайте аварийного восстановления, чтобы:
  - Разрешить обмен данными с хранилищем.
  - Включить связь с SAP HANA.

## <a name="set-up-disaster-recovery"></a>Настройка аварийного восстановления

Корпорация Майкрософт поддерживает репликацию на уровне хранилища для АВАРИЙного восстановления. Существует два способа настройки аварийного восстановления.

Один из них является **нормальным** , а другой — **многоцелевым**. В **обычной** аварийном восстановлении у вас есть выделенный экземпляр на месте аварийного восстановления для отработки отказа. В сценарии с **многоцелевым** аварийным восстановлением у вас есть еще один экземпляр Hana или разработка, работающий на единице крупного экземпляра Hana на сайте аварийного восстановления. Но вы также установили предварительно установленный экземпляр HANA, который находится в неактивном расположении и имеет тот же идентификатор безопасности, что и экземпляр HANA, для которого требуется выполнить отработку отказа на единицу крупного экземпляра HANA. Microsoft Operations настраивает среду для включения репликации хранилища на основе входных данных, указанных в форме запроса на обслуживание (SRF) во время адаптации.

> [!IMPORTANT]
> Убедитесь, что выполнены все необходимые условия для установки аварийного восстановления.

## <a name="monitor-data-replication-from-primary-to-dr-site"></a>Мониторинг репликации данных с первичного сайта на сайт аварийного восстановления

Команда Microsoft Operations группа уже управляет и отслеживает ссылку аварийного восстановления с первичного сайта на сайт аварийного восстановления.
Вы можете отслеживать репликацию данных с сервера источника на сервер аварийного восстановления с помощью команды snapshot `azacsnap -c details --details replication` .

## <a name="perform-a-failover-to-dr-site"></a>Выполнение отработки отказа на сайт аварийного восстановления

Выполните команду отработки отказа на сайте аварийного восстановления ( `azacsnap -c restore --restore revertvolume` ).

> [!IMPORTANT]
> `azacsnap -c restore --restore revertvolume`Команда прерывает репликацию хранилища с рабочего сайта на сайт аварийного восстановления. Для повторной настройки репликации необходимо обратиться к Microsoft Operations. После повторного включения репликации все данные в хранилище аварийного восстановления для этого идентификатора SID будут инициализированы. Команда, выполняющая отработку отказа, предоставляет доступ к последнему реплицированному моментальному снимку хранилища. Если необходимо восстановить старую версию моментального снимка, отправьте запрос в службу поддержки, чтобы операции могли предоставить более ранний моментальный снимок, восстановленный на сайте АВАРИЙного восстановления.

На высоком уровне ниже приведены шаги, которые необходимо выполнить для отработки отказа аварийного восстановления.

- Необходимо завершить работу экземпляра HANA на **первичном** сайте. Это действие требуется только в том случае, если вы действительно выполняете отработку отказа на сайт аварийного восстановления, чтобы избежать несогласованности данных.
- Завершите работу экземпляра HANA на узле аварийного восстановления для рабочего идентификатора безопасности.
- Выполните команду `azacsnap -c restore --restore revertvolume` на узле аварийного восстановления с ИД безопасности, который требуется восстановить.
  - Команда разбивает канал репликации хранилища с основного сайта на сайт аварийного восстановления.
  - Команда восстанавливает только том/Дата и/логбаккупс,/Shared том не восстанавливается, а вместо него использует существующий/Shared для SID в расположении DR.
  - Подключение тома/Дата и/логбаккупс — обязательно добавьте его в файл fstab
- Восстановите моментальный снимок HANA СИСТЕМДБ. HANA Studio показывает только последний моментальный снимок HANA, доступный в моментальном снимке хранилища, который был восстановлен в ходе `azacsnap -c restore --restore revertvolume` выполнения команды.
- Восстановите базу данных клиента.
- Запустите экземпляр HANA на сайте аварийного восстановления для рабочего идентификатора безопасности (например, в этом случае — H80).
- Выполните тестирование.

### <a name="example-performing-disaster-recovery"></a>Пример выполнения аварийного восстановления

В этом подразделе описаны подробные инструкции по отработке отказа на сайт аварийного восстановления.

#### <a name="step-1-get-the-volume-details-of-the-dr-node"></a>Шаг 1. Получение сведений о томе для узла аварийного восстановления

Выполните команду, `df –h` чтобы получить список файловых систем и связанных томов, на которые следует ссылаться после отработки отказа.

```bash
df -h
```

```output
Filesystem Size Used Avail Use% Mounted on
devtmpfs 378G 8.0K 378G 1% /dev
tmpfs 569G 0 569G 0%
/dev/shm
tmpfs 378G 18M 378G 1% /run
tmpfs 378G 0 378G 0%
/sys/fs/cgroup
/dev/mapper/3600a098038304445622b4b584c575a66-part2 47G 20G 28G 42% /
/dev/mapper/3600a098038304445622b4b584c575a66-part1 979M 57M 856M 7% /boot
172.18.20.241:/hana_log_h80_mnt00003_t020_vol 512G 2.1G 510G 1% /hana/log/H80/mnt00003
172.18.20.241:/hana_log_h80_mnt00001_t020_vol 512G 5.5G 507G 2% /hana/log/H80/mnt00001
172.18.20.241:/hana_data_h80_mnt00003_t020_vol 1.2T 332M 1.2T 1% /hana/data/H80/mnt00003
172.18.20.241:/hana_log_h80_mnt00002_t020_vol 512G 2.1G 510G 1% /hana/log/H80/mnt00002
172.18.20.241:/hana_data_h80_mnt00002_t020_vol 1.2T 300M 1.2T 1% /hana/data/H80/mnt00002
172.18.20.241:/hana_data_h80_mnt00001_t020_vol 1.2T 6.4G 1.2T 1% /hana/data/H80/mnt00001
172.18.20.241:/hana_shared_h80_t020_vol/usr_sap_node1 2.7T 11G 2.7T 1% /usr/sap/H80
tmpfs 76G 0 76G 0% /run/user/0
172.18.20.241:/hana_shared_h80_t020_vol 2.7T 11G 2.7T 1% /hana/shared
172.18.20.241:/hana_data_h80_mnt00001_t020_xdp 1.2T 6.4G 1.2T 1% /hana/data/H80/mnt00001
172.18.20.241:/hana_data_h80_mnt00002_t020_xdp 1.2T 300M 1.2T 1% /hana/data/H80/mnt00002
172.18.20.241:/hana_data_h80_mnt00003_t020_xdp 1.2T 332M 1.2T 1% /hana/data/H80/mnt00003
172.18.20.241:/hana_log_backups_h80_t020_xdp 512G 15G 498G 3% /hana/logbackups/H80_T250
```

#### <a name="step-2-shut-down-hana-on-the-primary-site"></a>Шаг 2. Завершение работы HANA на первичном сайте

Если выполняется полная отработка отказа рабочих нагрузок рабочей нагрузки, можно подключиться к основному рабочему сайту, а затем завершить работу SAP HANA экземпляров, которые отправляются на сайт аварийного восстановления.

Например, если войти в качестве привилегированного пользователя, в следующем примере показано, как можно завершить работу SAP HANA.  Замените на <sid> идентификатор безопасности SAP HANA.

```bash
su - <sid>adm
HDB stop
```

#### <a name="step-3-shut-down-hana-on-dr-site"></a>Шаг 3. Завершение работы HANA на сайте аварийного восстановления

Прежде чем восстанавливать тома, важно завершить работу SAP HANA на сайте аварийного восстановления.

Например, если войти в качестве привилегированного пользователя, в следующем примере показано, как можно завершить работу SAP HANA.  Замените на <sid> идентификатор безопасности SAP HANA.

```bash
su - <sid>adm
HDB stop
```

> [!IMPORTANT]
> Перед восстановлением томов убедитесь, что экземпляры HANA на сайте аварийного восстановления отключены.

#### <a name="step-4-restore-the-volumes"></a>Шаг 4. Восстановление томов

```bash
azacsnap -c restore --restore revertvolume --dbsid H80
```

**_Выходные данные команды аварийного восстановления_**.

```bash
azacsnap --configfile DR.json -c restore --restore revertvolume --dbsid H80
```

```output
* This program is designed for those customers who have previously installed the
  Production HANA instance in the Disaster Recovery Location either as a
  stand-alone instance or as part of a multi-purpose environment.
* This program should be executed from the Disaster Recovery location otherwise
  unintended consequences may result.
* This program is intended to allow the customer to complete a Disaster Recovery
  failover.
* Any other restore points must be handled by Microsoft Operations.
* All volumes ('data' and 'other') are reverted to their most recent snapshot.
* The SnapMirror replication relationship between Prod and DR will be broken.

  CAUTION: a failback will be required after running this command and failback
   might not be a quick process and will require multiple steps in coordination
   with Microsoft Operations.

Do you wish to continue? (y/n) [n]: y
Checking state of HLI volumes for SID 'H80'
Configured volumes (Data and Other) are not quiesced for revert, will retry in 00:00:10 seconds
Volumes All Ok to Revert = True
Reverting volume 'hana_data_h80_mnt00001_t020_xdp' to snapshot 'H80_HANA_DATA_30MIN.2020-09-16_0330.0'
DR.json Data Volume #1 'hana_data_h80_mnt00001_t020_xdp' assigning to mountpoint 'mnt00001'
Reverting volume 'hana_log_backups_h80_t020_xdp01' to snapshot 'H80_HANA_LOGS_3MIN_X9.2020-09-16_0339.recent'
DR.json Other Volume #1 'hana_log_backups_h80_t020_xdp01' assigning to mountpoint '01'
HLI Volume revert completed for SID 'H80'
Displaying Mount Points by Volume as follows:
10.50.251.34:/hana_data_h80_mnt00001_t020_xdp  /hana/data/H80/mnt00001 nfs  rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
10.50.251.36:/hana_log_backups_h80_t020_xdp01  /hana/log_backups/H80/01 nfs rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
*********************  HANA DR Restore Steps  **********************************
* Please complete the following steps to recover your HANA database:           *
* 1. Ensure ALL the target mount points exist to mount the snapshot clones.    *
*    e.g. mkdir /hana/logbackups/H99_SOURCE                                    *
* 2. Add Mount Point Details from 'Displaying Mount Points by Volume' as       *
*    output above into /etc/fstab of DR Server.                                *
* 3. Mount newly added filesystems.                                            *
* 4. Perform HANA Snapshot Recovery using HANA Studio.                         *
********************************************************************************
```

> [!NOTE]
> Чтобы завершить подготовку хранилища для отработки отказа аварийного восстановления, необходимо выполнить действия в конце консоли.

#### <a name="step-5-unmount-unnecessary-filesystems"></a>Шаг 5. Отключение ненужных файловых систем

Выполните команду, `umount` чтобы отключить ненужные файловые системы или тома.

```bash
umount <Mount point>
```

Отключите резервное копирование данных и журналов точки подключения. В сценарии масштабирования может быть несколько точки подключения к данным.

#### <a name="step-6-configure-the-mount-points"></a>Шаг 6. Настройка точек подключения

Измените файл, `/etc/fstab` чтобы закомментировать записи и резервные копии журналов первичного идентификатора безопасности (в этом примере это SID = H80) и добавить новые записи точек подключения, созданные на основе томов аварийного восстановления первичного сайта. Новые записи точки подключения приведены в выходных данных команды.

- Закомментируйте существующие точки подключения, работающие на сайте аварийного восстановления, с помощью `#` символа:

  ```output
  #172.18.20.241:/hana_data_h80_mnt00001_t020_vol /hana/data/H80/mnt00001 nfs     rw,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
  #172.18.20.241:/hana_log_backups_h80_t020 /hana/logbackups/H80 nfs rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
  ```

- Добавьте следующие строки в `/etc/fstab`
  > Это должны быть те же выходные данные команды.

  ```output
  10.50.251.34:/hana_data_h80_mnt00001_t020_xdp  /hana/data/H80/mnt00001 nfs  rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
  10.50.251.36:/hana_log_backups_h80_t020_xdp01  /hana/log_backups/H80/01 nfs rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
  ```

#### <a name="step-7-mount-the-recovery-volumes"></a>Шаг 7. подключение томов восстановления

Выполните команду, `mount –a` чтобы подключить все точки подключения.

```bash
mount -a
```

Теперь при выполнении `df –h` вы должны увидеть `*_dp` подключенные тома.

```bash
df -h
```

```output
Filesystem Size Used Avail Use% Mounted on
devtmpfs 378G 8.0K 378G 1% /dev
tmpfs 569G 0 569G 0% /dev/shm
tmpfs 378G 18M 378G 1% /run
tmpfs 378G 0 378G 0% /sys/fs/cgroup
/dev/mapper/3600a098038304445622b4b584c575a66-part2 47G 20G 28G 42% /
/dev/mapper/3600a098038304445622b4b584c575a66-part1 979M 57M 856M 7% /boot
172.18.20.241:/hana_log_h80_mnt00003_t020_vol 512G 2.1G 510G 1% /hana/log/H80/mnt00003
172.18.20.241:/hana_log_h80_mnt00001_t020_vol 512G 5.5G 507G 2% /hana/log/H80/mnt00001
172.18.20.241:/hana_data_h80_mnt00003_t020_vol 1.2T 332M 1.2T 1% /hana/data/H80/mnt00003
172.18.20.241:/hana_log_h80_mnt00002_t020_vol 512G 2.1G 510G 1% /hana/log/H80/mnt00002
172.18.20.241:/hana_data_h80_mnt00002_t020_vol 1.2T 300M 1.2T 1% /hana/data/H80/mnt00002
172.18.20.241:/hana_data_h80_mnt00001_t020_vol 1.2T 6.4G 1.2T 1% /hana/data/H80/mnt00001
172.18.20.241:/hana_shared_h80_t020_vol/usr_sap_node1 2.7T 11G 2.7T 1% /usr/sap/H80
tmpfs 76G 0 76G 0% /run/user/0
172.18.20.241:/hana_shared_h80_t020_vol 2.7T 11G 2.7T 1% /hana/shared
172.18.20.241:/hana_data_h80_mnt00001_t020_xdp 1.2T 6.4G 1.2T 1% /hana/data/H80/mnt00001
172.18.20.241:/hana_data_h80_mnt00002_t020_xdp 1.2T 300M 1.2T 1% /hana/data/H80/mnt00002
172.18.20.241:/hana_data_h80_mnt00003_t020_xdp 1.2T 332M 1.2T 1% /hana/data/H80/mnt00003
172.18.20.241:/hana_log_backups_h80_t020_xdp 512G 15G 498G 3% /hana/logbackups/H80_T250
```

#### <a name="step-8-recover-the-systemdb"></a>Шаг 8. Восстановление СИСТЕМДБ

В HANA Studio щелкните правой кнопкой мыши экземпляр СИСТЕМДБ и выберите пункт "резервное копирование и восстановление", а затем "восстановить системную базу данных".

Ознакомьтесь с руководством по восстановлению базы данных из моментального снимка, в частности СИСТЕМДБ.

#### <a name="step-9-recover-the-tenant-database"></a>Шаг 9. Восстановление базы данных клиента

В HANA Studio щелкните правой кнопкой мыши экземпляр СИСТЕМДБ и выберите "резервное копирование и восстановление", а затем "восстановить базу данных клиента".

Ознакомьтесь с руководством по восстановлению базы данных из моментального снимка, в частности баз данных клиента.

### <a name="run-azacsnap--c-backup-at-the-dr-site"></a>Запуск `azacsnap -c backup` на сайте аварийного восстановления

Если на сайте аварийного восстановления выполняются резервные копии на основе моментальных снимков, то имя сервера HANA, настроенное в `azacsnap` файле конфигурации на сайте аварийного восстановления, должно совпадать с именем рабочего сервера.

> [!IMPORTANT]
> При запуске `azacsnap -c backup` можно создавать моментальные снимки хранилища на сайте аварийного восстановления, которые не реплицируются автоматически на другой сайт.  Работа с Microsoft Operations для лучшего понимания возврата файлов и данных на исходный рабочий сайт.

## <a name="next-steps"></a>Дальнейшие действия

- [Получение сведений о снимке](azacsnap-cmd-ref-details.md)
- [Создание резервной копии](azacsnap-cmd-ref-backup.md)
