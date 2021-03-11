---
title: Изменение размера диска ОС с разделом GPT
description: В этой статье приводятся инструкции по изменению размера диска операционной системы с разделом GPT в Linux.
services: virtual-machines
ms.topic: article
author: kailashmsft
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines
ms.subservice: disks
ms.collection: linux
ms.workload: infrastructure-services
ms.devlang: azurecli
ms.date: 05/03/2020
ms.author: kaib
ms.custom: seodec18
ms.openlocfilehash: 46b6ceff74dd3a296d26cc018b380c1c3f76664c
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102552955"
---
# <a name="resize-an-os-disk-that-has-a-gpt-partition"></a>Изменение размера диска ОС с разделом GPT

> [!NOTE]
> Эта статья относится только к дискам ОС с разделом GPT.

В этой статье описывается, как увеличить размер диска ОС с разделом GPT в Linux. 

## <a name="identify-whether-the-os-disk-has-an-mbr-or-gpt-partition"></a>Необходимо определить, содержит ли диск ОС раздел MBR или GPT

Используйте `parted` команду, чтобы указать, создан ли раздел диска с помощью раздела основной загрузочной записи (MBR) или раздела GPT.

### <a name="mbr-partition"></a>Раздел MBR

В следующих выходных данных **таблица разделов** отображает значение **msdos**. Это значение указывает на раздел MBR.

```
[user@myvm ~]# parted -l /dev/sda
Model: Msft Virtual Disk (scsi)
Disk /dev/sda: 107GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Number  Start   End     Size    Type     File system  Flags
1       1049kB  525MB   524MB   primary  ext4         boot
2       525MB   34.4GB  33.8GB  primary  ext4
[user@myvm ~]#
```

### <a name="gpt-partition"></a>Раздел GPT

В следующих выходных данных **таблица разделов** отображает значение **gpt**. Это значение указывает на наличие раздела GPT.

```
[user@myvm ~]# parted -l /dev/sda
Model: Msft Virtual Disk (scsi)
Disk /dev/sda: 68.7GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name                  Flags
1       1049kB  525MB   524MB   fat16        EFI System Partition  boot
2       525MB   1050MB  524MB   xfs
3       1050MB  1052MB  2097kB                                     bios_grub
4       1052MB  68.7GB  67.7GB                                     lvm
```

Если виртуальная машина имеет раздел GPT на диске операционной системы, необходимо увеличить размер диска операционной системы.

## <a name="increase-the-size-of-the-os-disk"></a>Увеличение размера диска ОС

Приведенные ниже инструкции относятся к дистрибутивам, рекомендованным Linux.

> [!NOTE]
> Прежде чем продолжить работу, создайте резервную копию виртуальной машины или моментальный снимок диска операционной системы.

### <a name="ubuntu"></a>Ubuntu

Для увеличения размера диска операционной системы в Ubuntu 16. *x* и 18. *x*:

1. Остановите виртуальную машину.
1. Увеличьте размер диска операционной системы на портале.
1. Перезапустите виртуальную машину, а затем войдите на виртуальную машину в качестве **привилегированного** пользователя.
1. Убедитесь в том, что размер файловой системы на диске ОС увеличился.

В следующем примере размер диска ОС изменился с портала на 100 ГБ. Файловая система **/dev/sda1**, смонтированная на **/** , теперь имеет размер 97 ГБ.

```
user@myvm:~# df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  314M     0  314M   0% /dev
tmpfs          tmpfs      65M  2.3M   63M   4% /run
/dev/sda1      ext4       97G  1.8G   95G   2% /
tmpfs          tmpfs     324M     0  324M   0% /dev/shm
tmpfs          tmpfs     5.0M     0  5.0M   0% /run/lock
tmpfs          tmpfs     324M     0  324M   0% /sys/fs/cgroup
/dev/sda15     vfat      105M  3.6M  101M   4% /boot/efi
/dev/sdb1      ext4       20G   44M   19G   1% /mnt
tmpfs          tmpfs      65M     0   65M   0% /run/user/1000
user@myvm:~#
```

### <a name="suse"></a>SUSE

Увеличение размера диска ОС в SUSE 12 SP4, SUSE SLES 12 для SAP, SUSE SLES 15 и SUSE SLES 15 для SAP.

1. Остановите виртуальную машину.
1. Увеличьте размер диска операционной системы на портале.
1. Перезапустите виртуальную машину.

После перезапуска виртуальной машины выполните следующие действия.

1. Получите доступ к виртуальной машине от имени **привилегированного** пользователя с помощью следующей команды:

   ```
   # sudo -i
   ```

1. Используйте следующую команду, чтобы установить пакет **гровпарт** , который будет использоваться для изменения размера раздела:

   ```
   # zypper install growpart
   ```

1. Используйте `lsblk` команду, чтобы найти раздел, подключенный в корне файловой системы ( **/** ). В этом случае мы видим, что раздел 4 устройства **SDA** подключен к **/** :

   ```
   # lsblk
   NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda      8:0    0   48G  0 disk
   ├─sda1   8:1    0    2M  0 part
   ├─sda2   8:2    0  512M  0 part /boot/efi
   ├─sda3   8:3    0    1G  0 part /boot
   └─sda4   8:4    0 28.5G  0 part /
   sdb      8:16   0    4G  0 disk
   └─sdb1   8:17   0    4G  0 part /mnt/resource
   ```

1. Измените размер требуемой секции с помощью `growpart` команды и номера секции, определенного на предыдущем шаге:

   ```
   # growpart /dev/sda 4
   CHANGED: partition=4 start=3151872 old: size=59762655 end=62914527 new: size=97511391 end=100663263
   ```

1. Выполните `lsblk` команду еще раз, чтобы проверить, увеличилась ли секция.

   Следующие выходные данные показывают, что размер раздела **/dev/sda4** был изменен до 46,5 ГБ:
   
   ```
   linux:~ # lsblk
   NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda      8:0    0   48G  0 disk
   ├─sda1   8:1    0    2M  0 part
   ├─sda2   8:2    0  512M  0 part /boot/efi
   ├─sda3   8:3    0    1G  0 part /boot
   └─sda4   8:4    0 46.5G  0 part /
   sdb      8:16   0    4G  0 disk
   └─sdb1   8:17   0    4G  0 part /mnt/resource
   ```

1. Укажите тип файловой системы на диске ОС с помощью `lsblk` команды с `-f` флагом:

   ```
   linux:~ # lsblk -f
   NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
   sda
   ├─sda1
   ├─sda2 vfat   EFI   AC67-D22D                            /boot/efi
   ├─sda3 xfs    BOOT  5731a128-db36-4899-b3d2-eb5ae8126188 /boot
   └─sda4 xfs    ROOT  70f83359-c7f2-4409-bba5-37b07534af96 /
   sdb
   └─sdb1 ext4         8c4ca904-cd93-4939-b240-fb45401e2ec6 /mnt/resource
   ```

1. Для изменения размера файловой системы используйте команды, соответствующие типу файловой системы.
   
   Для **XFS** используйте следующую команду:
   
   ```
   #xfs_growfs /
   ```
   
   Выходные данные примера:
   
   ```
   linux:~ # xfs_growfs /
   meta-data=/dev/sda4              isize=512    agcount=4, agsize=1867583 blks
            =                       sectsz=512   attr=2, projid32bit=1
            =                       crc=1        finobt=0 spinodes=0 rmapbt=0
            =                       reflink=0
   data     =                       bsize=4096   blocks=7470331, imaxpct=25
            =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
   log      =internal               bsize=4096   blocks=3647, version=2
            =                       sectsz=512   sunit=0 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   data blocks changed from 7470331 to 12188923
   ```
   
   Для **ext4** используйте следующую команду:
   
   ```
   #resize2fs /dev/sda4
   ```
   
1. Проверьте увеличенный размер файловой системы для **DF-TH** с помощью следующей команды:
   
   ```
   #df -Thl
   ```
   
   Выходные данные примера:
   
   ```
   linux:~ # df -Thl
   Filesystem     Type      Size  Used Avail Use% Mounted on
   devtmpfs       devtmpfs  445M  4.0K  445M   1% /dev
   tmpfs          tmpfs     458M     0  458M   0% /dev/shm
   tmpfs          tmpfs     458M   14M  445M   3% /run
   tmpfs          tmpfs     458M     0  458M   0% /sys/fs/cgroup
   /dev/sda4      xfs        47G  2.2G   45G   5% /
   /dev/sda3      xfs      1014M   86M  929M   9% /boot
   /dev/sda2      vfat      512M  1.1M  511M   1% /boot/efi
   /dev/sdb1      ext4      3.9G   16M  3.7G   1% /mnt/resource
   tmpfs          tmpfs      92M     0   92M   0% /run/user/1000
   tmpfs          tmpfs      92M     0   92M   0% /run/user/490
   ```
   
   В предыдущем примере можно увидеть, что размер файловой системы для диска ОС увеличился.

### <a name="rhel-with-lvm"></a>RHEL с LVM

1. Получите доступ к виртуальной машине от имени **привилегированного** пользователя с помощью следующей команды:

   ```bash
   [root@dd-rhel7vm ~]# sudo -i
   ```

1. Используйте `lsblk` команду, чтобы определить, какой логический том (LV) подключен в корне файловой системы ( **/** ). В этом случае мы видим, что **рутвг-рутлв** подключен к **/** . Если вам нужна другая файловая система, замените ее на "Латвия" и "точка подключения" в этой статье.

   ```shell
   [root@dd-rhel7vm ~]# lsblk -f
   NAME                  FSTYPE      LABEL   UUID                                   MOUNTPOINT
   fd0
   sda
   ├─sda1                vfat                C13D-C339                              /boot/efi
   ├─sda2                xfs                 8cc4c23c-fa7b-4a4d-bba8-4108b7ac0135   /boot
   ├─sda3
   └─sda4                LVM2_member         zx0Lio-2YsN-ukmz-BvAY-LCKb-kRU0-ReRBzh
      ├─rootvg-tmplv      xfs                 174c3c3a-9e65-409a-af59-5204a5c00550   /tmp
      ├─rootvg-usrlv      xfs                 a48dbaac-75d4-4cf6-a5e6-dcd3ffed9af1   /usr
      ├─rootvg-optlv      xfs                 85fe8660-9acb-48b8-98aa-bf16f14b9587   /opt
      ├─rootvg-homelv     xfs                 b22432b1-c905-492b-a27f-199c1a6497e7   /home
      ├─rootvg-varlv      xfs                 24ad0b4e-1b6b-45e7-9605-8aca02d20d22   /var
      └─rootvg-rootlv     xfs                 4f3e6f40-61bf-4866-a7ae-5c6a94675193   /
   ```

1. Проверьте, нет ли свободного места в группе томов LVM (VG), содержащей корневой раздел. Если свободного места нет, перейдите к шагу 12.

   ```bash
   [root@dd-rhel7vm ~]# vgdisplay rootvg
   --- Volume group ---
   VG Name               rootvg
   System ID
   Format                lvm2
   Metadata Areas        1
   Metadata Sequence No  7
   VG Access             read/write
   VG Status             resizable
   MAX LV                0
   Cur LV                6
   Open LV               6
   Max PV                0
   Cur PV                1
   Act PV                1
   VG Size               <63.02 GiB
   PE Size               4.00 MiB
   Total PE              16132
   Alloc PE / Size       6400 / 25.00 GiB
   Free  PE / Size       9732 / <38.02 GiB
   VG UUID               lPUfnV-3aYT-zDJJ-JaPX-L2d7-n8sL-A9AgJb
   ```

   В этом примере **бесплатный PE и размер** строки показывают, что в группе томов свободно 38,02 ГБ. Не нужно изменять размер диска перед добавлением пространства в группу томов.

1. Чтобы увеличить размер диска операционной системы в RHEL 7. *x* с LVM:

   1. Остановите виртуальную машину.
   1. Увеличьте размер диска операционной системы на портале.
   1. Запустите виртуальную машину.

1. После перезапуска виртуальной машины выполните следующие действия.

   - Установите пакет **Cloud-utils-гровпарт** , чтобы предоставить команду **гровпарт** , которая требуется для увеличения размера диска операционной системы и обработчика гдиск для разметки дисков GPT. Эти пакеты предварительно устанавливаются в большинстве образов Marketplace.

   ```bash
   [root@dd-rhel7vm ~]# yum install cloud-utils-growpart gdisk
   ```

1. Определите, какой диск и раздел содержат физический том или тома LVM (ПС) в группе томов с именем **рутвг** , выполнив `pvscan` команду. Обратите внимание на размер и свободное пространство, перечисленное между квадратными скобками (**[** и **]**).

   ```bash
   [root@dd-rhel7vm ~]# pvscan
     PV /dev/sda4   VG rootvg          lvm2 [<63.02 GiB / <38.02 GiB free]
   ```

1. Проверьте размер секции с помощью `lsblk` . 

   ```bash
   [root@dd-rhel7vm ~]# lsblk /dev/sda4
   NAME            MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
   sda4              8:4    0  63G  0 part
   ├─rootvg-tmplv  253:1    0   2G  0 lvm  /tmp
   ├─rootvg-usrlv  253:2    0  10G  0 lvm  /usr
   ├─rootvg-optlv  253:3    0   2G  0 lvm  /opt
   ├─rootvg-homelv 253:4    0   1G  0 lvm  /home
   ├─rootvg-varlv  253:5    0   8G  0 lvm  /var
   └─rootvg-rootlv 253:6    0   2G  0 lvm  /
   ```

1. Разверните секцию, содержащую эту ПС, используя `growpart` , имя устройства и номер раздела. Это позволит расширить указанную секцию, чтобы она использовала все свободное непрерывное пространство на устройстве.

   ```bash
   [root@dd-rhel7vm ~]# growpart /dev/sda 4
   CHANGED: partition=4 start=2054144 old: size=132161536 end=134215680 new: size=199272414 end=201326558
   ```

1. Убедитесь, что размер секции изменился до ожидаемого размера, повторно с помощью `lsblk` команды. Обратите внимание, что в примере **sda4** изменился с 63 гб до 95 ГБ.

   ```bash
   [root@dd-rhel7vm ~]# lsblk /dev/sda4
   NAME            MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
   sda4              8:4    0  95G  0 part
   ├─rootvg-tmplv  253:1    0   2G  0 lvm  /tmp
   ├─rootvg-usrlv  253:2    0  10G  0 lvm  /usr
   ├─rootvg-optlv  253:3    0   2G  0 lvm  /opt
   ├─rootvg-homelv 253:4    0   1G  0 lvm  /home
   ├─rootvg-varlv  253:5    0   8G  0 lvm  /var
   └─rootvg-rootlv 253:6    0   2G  0 lvm  /
   ```

1. Разверните ПС, чтобы использовать оставшуюся часть вновь развернутой секции:

   ```bash
   [root@dd-rhel7vm ~]# pvresize /dev/sda4
   Physical volume "/dev/sda4" changed
   1 physical volume(s) resized or updated / 0 physical volume(s) not resized
   ```

1. Убедитесь, что новый размер PV соответствует ожидаемому размеру, сравнив его с исходными значениями **[size/Free]** :

   ```bash
   [root@dd-rhel7vm ~]# pvscan
   PV /dev/sda4   VG rootvg          lvm2 [<95.02 GiB / <70.02 GiB free]
   ```

1. Разверните нужный логический том (LV) на нужное количество. Объем не обязательно должен быть свободным местом в группе томов. В следующем примере размер **/Дев/маппер/рутвг-рутлв** изменяется с 2 ГБ до 12 ГБ (увеличение 10 ГБ). Эта команда также позволяет изменить размер файловой системы.

   ```bash
   [root@dd-rhel7vm ~]# lvresize -r -L +10G /dev/mapper/rootvg-rootlv
   ```

   Выходные данные примера:

   ```bash
   [root@dd-rhel7vm ~]# lvresize -r -L +10G /dev/mapper/rootvg-rootlv
   Size of logical volume rootvg/rootlv changed from 2.00 GiB (512 extents) to 12.00 GiB (3072 extents).
   Logical volume rootvg/rootlv successfully resized.
   meta-data=/dev/mapper/rootvg-rootlv isize=512    agcount=4, agsize=131072 blks
            =                       sectsz=4096  attr=2, projid32bit=1
            =                       crc=1        finobt=0 spinodes=0
   data     =                       bsize=4096   blocks=524288, imaxpct=25
            =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
   log      =internal               bsize=4096   blocks=2560, version=2
            =                       sectsz=4096  sunit=1 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   data blocks changed from 524288 to 3145728
   ```

1. `lvresize`Команда автоматически вызывает соответствующую команду изменения размера для файловой системы в lv. Проверьте, имеет ли **/Дев/маппер/рутвг-рутлв**, который подключен к **/** , повышенный размер файловой системы с помощью следующей команды:

   ```shell
   [root@dd-rhel7vm ~]# df -Th /
   ```

   Выходные данные примера:

   ```shell
   [root@dd-rhel7vm ~]# df -Th /
   Filesystem                Type  Size  Used Avail Use% Mounted on
   /dev/mapper/rootvg-rootlv xfs    12G   71M   12G   1% /
   [root@dd-rhel7vm ~]#
   ```

> [!NOTE]
> Чтобы использовать ту же процедуру для изменения размера любого другого логического тома, измените имя LV на шаге 12.

### <a name="rhel-raw"></a>RHEL RAW
>[!NOTE] 
>Прежде чем увеличивать размер диска ОС, всегда сделайте снимок виртуальной машины.

Чтобы увеличить размер диска операционной системы в необработанном разделе RHEL, выполните следующие действия.

1. Остановите виртуальную машину.
1. Увеличьте размер диска операционной системы на портале.
1. Запустите виртуальную машину.

После перезапуска виртуальной машины выполните следующие действия.

1. Войдите на виртуальную машину с учетными данными **корневого**, используя следующую команду:
 
   ```
   sudo su
   ```

1. Установите пакет **gptfdisk**, который необходим для увеличения размера диска ОС.

   ```
   yum install gdisk -y
   ```

1.  Чтобы просмотреть все доступные секторы на диске, выполните следующую команду:
    ```
    gdisk -l /dev/sda
    ```

1. Вы увидите сведения, уведомляющие о типе секции. Убедитесь, что это GPT. Определяет корневой раздел. Не изменяйте и не удаляйте загрузочный раздел (загрузочный раздел BIOS) и системный раздел ("системный раздел EFI").

1. Используйте следующую команду, чтобы начать секционирование в первый раз. 
    ```
    gdisk /dev/sda
    ```

1. Теперь вы увидите сообщение с запросом следующей команды (команда:? для справки). 

   ```
   w
   ```

1. Появится предупреждение "предупреждение! Дополнительный заголовок помещается на диске слишком рано. Вы хотите устранить эту проблему? (Y/N): ". Необходимо нажать "Y"

   ```
   Y
   ```

1. Должно отобразиться сообщение о завершении окончательных проверок и запросе подтверждения. Нажмите "Y"

   ```
   Y
   ```

1. Проверьте, правильно ли произошло все с помощью команды партпробе

   ```
   partprobe
   ```

1. Описанные выше шаги гарантируют, что дополнительный заголовок GPT помещается в конец. Следующим шагом является запуск процесса изменения размера с помощью средства гдиск. Используйте следующую команду.

   ```
   gdisk /dev/sda
   ```
1. Чтобы просмотреть список разделов, в меню Команда нажмите кнопку "p". Выявление корневого раздела (в шагах sda2 считается корневым разделом) и загрузочным разделом (в шагах sda3 считается загрузочным разделом). 

   ```
   p
   ```
    ![Корневой раздел и загрузочный раздел](./media/resize-os-disk-rhelraw/resize-os-disk-rhelraw1.png)

1. Нажмите клавишу "d", чтобы удалить секцию и выбрать номер секции, назначенный для загрузки (в этом примере это "3")
   ```
   d
   3
   ```
1. Нажмите клавишу "d", чтобы удалить секцию и выбрать номер секции, назначенный для загрузки (в этом примере — "2").
   ```
   d
   2
   ```
    ![Удалить корневой раздел и загрузочный раздел](./media/resize-os-disk-rhelraw/resize-os-disk-rhelraw2.png)

1. Чтобы повторно создать корневой раздел с увеличенным размером, нажмите "n", введите номер раздела, который вы ранее удалили для корневого ("2" в этом примере), и выберите первый сектор в качестве значения по умолчанию, последний сектор — "Последнее значение сектора — размер загрузочного сектора" ("4096" в этом случае) и шестнадцатеричный код как "8300".
   ```
   n
   2
   (Enter default)
   (Calculateed value of Last sector value - 4096)
   8300
   ```
1. Чтобы повторно создать загрузочный раздел, нажмите "n", введите номер раздела, который вы ранее удалили для загрузки ("3" для этого примера), и выберите первый сектор в качестве значения по умолчанию, последний сектор — "значение по умолчанию" и шестнадцатеричный код как "EF02"
   ```
   n
   3
   (Enter default)
   (Enter default)
   EF02
   ```

1. Запишите изменения с помощью команды "w" и нажмите клавишу "Y" для подтверждения
   ```
   w
   Y
   ```
1. Выполните команду "партпробе", чтобы проверить стабильность диска
   ```
   partprobe
   ```
1. Перезагрузите виртуальную машину, и размер корневого раздела увеличится.
   ```
   reboot
   ```

   ![Новый корневой раздел и загрузочный раздел](./media/resize-os-disk-rhelraw/resize-os-disk-rhelraw3.png)

1. Выполните команду xfs_growfs в разделе, чтобы изменить ее размер.
   ```
   xfs_growfs /dev/sda2
   ```

   ![XFS расширение FS](./media/resize-os-disk-rhelraw/resize-os-disk-rhelraw4.png)


1. Убедитесь, что новый размер отражается с помощью команды **DF** :

   ```bash
   [root@vm-dd-cent7 ~]# df -hl
   Filesystem      Size  Used Avail Use% Mounted on
   devtmpfs        452M     0  452M   0% /dev
   tmpfs           464M     0  464M   0% /dev/shm
   tmpfs           464M  6.8M  457M   2% /run
   tmpfs           464M     0  464M   0% /sys/fs/cgroup
   /dev/sda2        48G  2.1G   46G   5% /
   /dev/sda1       494M   65M  430M  13% /boot
   /dev/sda15      495M   12M  484M   3% /boot/efi
   /dev/sdb1       3.9G   16M  3.7G   1% /mnt/resource
   tmpfs            93M     0   93M   0% /run/user/1000
   ```
