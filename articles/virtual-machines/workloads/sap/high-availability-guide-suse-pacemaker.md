---
title: Настройка Pacemaker в SLES в Azure | Документация Майкрософт
description: Настройка кластера Pacemaker в SUSE Linux Enterprise Server в Azure.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 02/03/2020
ms.author: radeltch
ms.openlocfilehash: ddee5edcf1d19af0fb088976c590b62866a1484e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101674422"
---
# <a name="setting-up-pacemaker-on-suse-linux-enterprise-server-in-azure"></a>Настройка кластера Pacemaker в SUSE Linux Enterprise Server в Azure.

[planning-guide]:planning-guide.md
[deployment-guide]:deployment-guide.md
[dbms-guide]:dbms-guide.md
[sap-hana-ha]:sap-hana-high-availability.md
[virtual-machines-linux-maintenance]:../../maintenance-and-updates.md#maintenance-that-doesnt-require-a-reboot
[virtual-machines-windows-maintenance]:../../maintenance-and-updates.md#maintenance-that-doesnt-require-a-reboot
[sles-nfs-guide]:high-availability-guide-suse-nfs.md
[sles-guide]:high-availability-guide-suse.md

Существуют два варианта настройки кластера Pacemaker в Azure. Вы можете использовать агент ограждения, который выполняет перезапуск неисправного узла через интерфейсы API Azure, или можете использовать устройство SBD.

Для устройства SBD требуется по крайней мере одна дополнительная виртуальная машина, выступающая в роли сервера цели iSCSI и предоставляющая устройство SBD. Эти серверы цели iSCSI могут совместно использоваться другими кластерами Pacemaker. Преимущество использования устройства SBD заключается в том, что если вы уже используете устройства SBD локально, не требует каких-либо изменений в работе кластера Pacemaker. В кластере Pacemaker можно использовать до трех устройств SBD, чтобы устройство SBD можно было сделать недоступным, например, во время установки исправлений ОС на сервере цели iSCSI. Если вы хотите использовать несколько устройств SBD в Pacemaker, разверните несколько серверов цели iSCSI и подключите одно устройство SBD из каждого из этих серверов. Мы рекомендуем использовать одно устройство SBD или три. Pacemaker не сможет автоматически ограждать узел кластера, если вы настроите только два устройства SBD и одно из них будет недоступно. Если вы хотите иметь возможность ограждения, когда один сервер цели iSCSI не работает, необходимо использовать три устройства SBD и, следовательно, три целевых сервера iSCSI, которые являются самой гибкой конфигурацией при использовании Сбдс.

Агенту ограждения Azure не требуется развертывание дополнительных виртуальных машин.   

![Обзор кластера Pacemaker на SLES](./media/high-availability-guide-suse-pacemaker/pacemaker.png)

>[!IMPORTANT]
> При планировании и развертывании Linux Pacemaker кластеризованных узлов и устройств SBD очень важно для общей надежности полной конфигурации кластера, чтобы маршрутизация между задействованными виртуальными машинами и виртуальными машинами, на которых размещены устройства SBD, не проходила через другие устройства, такие как [виртуальные сетевые устройства](https://azure.microsoft.com/solutions/network-appliances/). В противном случае проблемы и события обслуживания с виртуальным сетевым устройством могут иметь негативное влияние на стабильность и надежность общей конфигурации кластера. Во избежание таких препятствий при планировании и развертывании кластерных узлов Linux Pacemaker и устройств SBD не определяйте правила маршрутизации виртуальных сетевых устройств или [определяемые пользователем правила маршрутизации](../../../virtual-network/virtual-networks-udr-overview.md), которые маршрутизируют трафик между кластерными узлами и устройствами SBD через виртуальные сетевые устройства и аналогичные устройства. 
>

## <a name="sbd-fencing"></a>Ограждение SBD

Если вы хотите использовать устройство SBD для ограждения, выполните следующие действия.

### <a name="set-up-iscsi-target-servers"></a>Настройка серверов цели iSCSI

Сначала необходимо создать целевые виртуальные машины iSCSI. Серверы цели iSCSI могут использовать совместно несколько кластеров Pacemaker.

1. Разверните новую виртуальную машину под управлением SLES 12 с пакетом обновления 1 (SP1) или более поздней версии и подключите ее по протоколу SSH. Размер таких виртуальных машин не обязательно должен быть большим. Будет достаточно выбрать размер Standard_E2s_v3 или Standard_D2s_v3. Убедитесь в том, что для диска ОС используется хранилище класса Premium.

Выполните следующие команды на всех **целевых виртуальных машинах iSCSI**.

1. Обновите SLES.

   <pre><code>sudo zypper update
   </code></pre>

   > [!NOTE]
   > После обновления ОС может потребоваться ее перезагрузка. 

1. Удалите пакеты.

   Чтобы избежать проблем с targetcli и SLES 12 с пакетом обновления 3 (SP3), удалите указанные ниже пакеты. Можно игнорировать сообщения об ошибках со сведениями о том, что пакет не удается обнаружить.

   <pre><code>sudo zypper remove lio-utils python-rtslib python-configshell targetcli
   </code></pre>

1. Установите пакеты цели iSCSI.

   <pre><code>sudo zypper install targetcli-fb dbus-1-python
   </code></pre>

1. Включите службу цели iSCSI.

   <pre><code>sudo systemctl enable targetcli
   sudo systemctl start targetcli
   </code></pre>

### <a name="create-iscsi-device-on-iscsi-target-server"></a>Создайте устройство iSCSI на сервере цели iSCSI.

Выполните следующие команды на всех **целевых виртуальных машинах iSCSI**, чтобы создать диски iSCSI для кластеров, используемых вашими системами SAP. В следующем примере создаются устройства SBD для нескольких кластеров. В нем показано, как использовать один сервер цели iSCSI для нескольких кластеров. Устройства SBD размещаются на диске операционной системы. Убедитесь, что на диске достаточно места.

**`nfs`**  — это кластер NFS, **ascsnw1** — кластер ASCS **NW1**, **dbnw1** — кластер базы данных **NW1**, **nfs-0** и **nfs-1** — имена узлов кластера NFS, **nw1-xscs-0** и **nw1-xscs-1** — имена узлов кластера **NW1** ASCS, а **nw1-db-0** и **nw1-db-1** — имена узлов кластера базы данных. Замените их именами узлов кластера и SID системы SAP.

<pre><code># Create the root folder for all SBD devices
sudo mkdir /sbd

# Create the SBD device for the NFS server
sudo targetcli backstores/fileio create sbdnfs /sbd/sbdnfs 50M write_back=false
sudo targetcli iscsi/ create iqn.2006-04.nfs.local:nfs
sudo targetcli iscsi/iqn.2006-04.nfs.local:nfs/tpg1/luns/ create /backstores/fileio/sbdnfs
sudo targetcli iscsi/iqn.2006-04.nfs.local:nfs/tpg1/acls/ create iqn.2006-04.<b>nfs-0.local:nfs-0</b>
sudo targetcli iscsi/iqn.2006-04.nfs.local:nfs/tpg1/acls/ create iqn.2006-04.<b>nfs-1.local:nfs-1</b>

# Create the SBD device for the ASCS server of SAP System NW1
sudo targetcli backstores/fileio create sbdascs<b>nw1</b> /sbd/sbdascs<b>nw1</b> 50M write_back=false
sudo targetcli iscsi/ create iqn.2006-04.ascs<b>nw1</b>.local:ascs<b>nw1</b>
sudo targetcli iscsi/iqn.2006-04.ascs<b>nw1</b>.local:ascs<b>nw1</b>/tpg1/luns/ create /backstores/fileio/sbdascs<b>nw1</b>
sudo targetcli iscsi/iqn.2006-04.ascs<b>nw1</b>.local:ascs<b>nw1</b>/tpg1/acls/ create iqn.2006-04.<b>nw1-xscs-0.local:nw1-xscs-0</b>
sudo targetcli iscsi/iqn.2006-04.ascs<b>nw1</b>.local:ascs<b>nw1</b>/tpg1/acls/ create iqn.2006-04.<b>nw1-xscs-1.local:nw1-xscs-1</b>

# Create the SBD device for the database cluster of SAP System NW1
sudo targetcli backstores/fileio create sbddb<b>nw1</b> /sbd/sbddb<b>nw1</b> 50M write_back=false
sudo targetcli iscsi/ create iqn.2006-04.db<b>nw1</b>.local:db<b>nw1</b>
sudo targetcli iscsi/iqn.2006-04.db<b>nw1</b>.local:db<b>nw1</b>/tpg1/luns/ create /backstores/fileio/sbddb<b>nw1</b>
sudo targetcli iscsi/iqn.2006-04.db<b>nw1</b>.local:db<b>nw1</b>/tpg1/acls/ create iqn.2006-04.<b>nw1-db-0.local:nw1-db-0</b>
sudo targetcli iscsi/iqn.2006-04.db<b>nw1</b>.local:db<b>nw1</b>/tpg1/acls/ create iqn.2006-04.<b>nw1-db-1.local:nw1-db-1</b>

# save the targetcli changes
sudo targetcli saveconfig
</code></pre>

Проверить правильность настройки можно следующим образом:

<pre><code>sudo targetcli ls

o- / .......................................................................................................... [...]
  o- backstores ............................................................................................... [...]
  | o- block ................................................................................... [Storage Objects: 0]
  | o- fileio .................................................................................. [Storage Objects: 3]
  | | o- <b>sbdascsnw1</b> ................................................ [/sbd/sbdascsnw1 (50.0MiB) write-thru activated]
  | | | o- alua .................................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ........................................................ [ALUA state: Active/optimized]
  | | o- <b>sbddbnw1</b> .................................................... [/sbd/sbddbnw1 (50.0MiB) write-thru activated]
  | | | o- alua .................................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ........................................................ [ALUA state: Active/optimized]
  | | o- <b>sbdnfs</b> ........................................................ [/sbd/sbdnfs (50.0MiB) write-thru activated]
  | |   o- alua .................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ........................................................ [ALUA state: Active/optimized]
  | o- pscsi ................................................................................... [Storage Objects: 0]
  | o- ramdisk ................................................................................. [Storage Objects: 0]
  o- iscsi ............................................................................................. [Targets: 3]
  | o- <b>iqn.2006-04.ascsnw1.local:ascsnw1</b> .................................................................. [TPGs: 1]
  | | o- tpg1 ................................................................................ [no-gen-acls, no-auth]
  | |   o- acls ........................................................................................... [ACLs: 2]
  | |   | o- <b>iqn.2006-04.nw1-xscs-0.local:nw1-xscs-0</b> ............................................... [Mapped LUNs: 1]
  | |   | | o- mapped_lun0 ............................................................ [lun0 fileio/<b>sbdascsnw1</b> (rw)]
  | |   | o- <b>iqn.2006-04.nw1-xscs-1.local:nw1-xscs-1</b> ............................................... [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 ............................................................ [lun0 fileio/<b>sbdascsnw1</b> (rw)]
  | |   o- luns ........................................................................................... [LUNs: 1]
  | |   | o- lun0 .......................................... [fileio/sbdascsnw1 (/sbd/sbdascsnw1) (default_tg_pt_gp)]
  | |   o- portals ..................................................................................... [Portals: 1]
  | |     o- 0.0.0.0:3260 ...................................................................................... [OK]
  | o- <b>iqn.2006-04.dbnw1.local:dbnw1</b> ...................................................................... [TPGs: 1]
  | | o- tpg1 ................................................................................ [no-gen-acls, no-auth]
  | |   o- acls ........................................................................................... [ACLs: 2]
  | |   | o- <b>iqn.2006-04.nw1-db-0.local:nw1-db-0</b> ................................................... [Mapped LUNs: 1]
  | |   | | o- mapped_lun0 .............................................................. [lun0 fileio/<b>sbddbnw1</b> (rw)]
  | |   | o- <b>iqn.2006-04.nw1-db-1.local:nw1-db-1</b> ................................................... [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 .............................................................. [lun0 fileio/<b>sbddbnw1</b> (rw)]
  | |   o- luns ........................................................................................... [LUNs: 1]
  | |   | o- lun0 .............................................. [fileio/sbddbnw1 (/sbd/sbddbnw1) (default_tg_pt_gp)]
  | |   o- portals ..................................................................................... [Portals: 1]
  | |     o- 0.0.0.0:3260 ...................................................................................... [OK]
  | o- <b>iqn.2006-04.nfs.local:nfs</b> .......................................................................... [TPGs: 1]
  |   o- tpg1 ................................................................................ [no-gen-acls, no-auth]
  |     o- acls ........................................................................................... [ACLs: 2]
  |     | o- <b>iqn.2006-04.nfs-0.local:nfs-0</b> ......................................................... [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ................................................................ [lun0 fileio/<b>sbdnfs</b> (rw)]
  |     | o- <b>iqn.2006-04.nfs-1.local:nfs-1</b> ......................................................... [Mapped LUNs: 1]
  |     |   o- mapped_lun0 ................................................................ [lun0 fileio/<b>sbdnfs</b> (rw)]
  |     o- luns ........................................................................................... [LUNs: 1]
  |     | o- lun0 .................................................. [fileio/sbdnfs (/sbd/sbdnfs) (default_tg_pt_gp)]
  |     o- portals ..................................................................................... [Portals: 1]
  |       o- 0.0.0.0:3260 ...................................................................................... [OK]
  o- loopback .......................................................................................... [Targets: 0]
  o- vhost ............................................................................................. [Targets: 0]
  o- xen-pvscsi ........................................................................................ [Targets: 0]
</code></pre>

### <a name="set-up-sbd-device"></a>Настройка устройства SBD

Подключитесь к устройству iSCSI, созданному на предыдущем шаге, из кластера.
Выполните следующие команды для узлов нового кластера, которые нужно создать.
Ниже приведены элементы с префиксами: **[A]**  — применяется ко всем узлам, **[1**] — применяется только к узлу 1, **[2]**  — применяется только к узлу 2.

1. **[A]**  Подключитесь к устройствам iSCSI.

   Сначала включите службы iSCSI и SBD.

   <pre><code>sudo systemctl enable iscsid
   sudo systemctl enable iscsi
   sudo systemctl enable sbd
   </code></pre>

1. **[1]**  Измените имя инициатора на первом узле.

   <pre><code>sudo vi /etc/iscsi/initiatorname.iscsi
   </code></pre>

   Измените содержимое файла в соответствии со списками управления доступом, использованными при создании устройства iSCSI на сервере цели iSCSI, например, на NFS-сервере.

   <pre><code>InitiatorName=<b>iqn.2006-04.nfs-0.local:nfs-0</b>
   </code></pre>

1. **[2]**  Измените имя инициатора на втором узле.

   <pre><code>sudo vi /etc/iscsi/initiatorname.iscsi
   </code></pre>

   Измените содержимое файла в соответствии со списками управления доступом, использованными при создании устройства iSCSI на сервере цели iSCSI.

   <pre><code>InitiatorName=<b>iqn.2006-04.nfs-1.local:nfs-1</b>
   </code></pre>

1. **[A]** Запустите службу iSCSI.

   Теперь запустите службу iSCSI для применения изменений.

   <pre><code>sudo systemctl restart iscsid
   sudo systemctl restart iscsi
   </code></pre>

   Подключитесь к устройствам iSCSI. В следующем примере 10.0.0.17 является IP-адресом сервера цели iSCSI, а 3260 — это порт по умолчанию. <b>iqn.2006-04.nfs.local:nfs</b> является одним из целевых имен, которые возвращаются при выполнении первой приведенной ниже команды (iscsiadm -m discovery).

   <pre><code>sudo iscsiadm -m discovery --type=st --portal=<b>10.0.0.17:3260</b>   
   sudo iscsiadm -m node -T <b>iqn.2006-04.nfs.local:nfs</b> --login --portal=<b>10.0.0.17:3260</b>
   sudo iscsiadm -m node -p <b>10.0.0.17:3260</b> -T <b>iqn.2006-04.nfs.local:nfs</b> --op=update --name=node.startup --value=automatic
   
   # If you want to use multiple SBD devices, also connect to the second iSCSI target server
   sudo iscsiadm -m discovery --type=st --portal=<b>10.0.0.18:3260</b>   
   sudo iscsiadm -m node -T <b>iqn.2006-04.nfs.local:nfs</b> --login --portal=<b>10.0.0.18:3260</b>
   sudo iscsiadm -m node -p <b>10.0.0.18:3260</b> -T <b>iqn.2006-04.nfs.local:nfs</b> --op=update --name=node.startup --value=automatic
   
   # If you want to use multiple SBD devices, also connect to the third iSCSI target server
   sudo iscsiadm -m discovery --type=st --portal=<b>10.0.0.19:3260</b>   
   sudo iscsiadm -m node -T <b>iqn.2006-04.nfs.local:nfs</b> --login --portal=<b>10.0.0.19:3260</b>
   sudo iscsiadm -m node -p <b>10.0.0.19:3260</b> -T <b>iqn.2006-04.nfs.local:nfs</b> --op=update --name=node.startup --value=automatic
   </code></pre>

   Убедитесь, что устройства iSCSI доступны, и запишите имя устройства (в следующем примере это /dev/sde).

   <pre><code>lsscsi
   
   # [2:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sda
   # [3:0:1:0]    disk    Msft     Virtual Disk     1.0   /dev/sdb
   # [5:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sdc
   # [5:0:0:1]    disk    Msft     Virtual Disk     1.0   /dev/sdd
   # <b>[6:0:0:0]    disk    LIO-ORG  sbdnfs           4.0   /dev/sdd</b>
   # <b>[7:0:0:0]    disk    LIO-ORG  sbdnfs           4.0   /dev/sde</b>
   # <b>[8:0:0:0]    disk    LIO-ORG  sbdnfs           4.0   /dev/sdf</b>
   </code></pre>

   Теперь получите идентификаторы устройств iSCSI.

   <pre><code>ls -l /dev/disk/by-id/scsi-* | grep <b>sdd</b>
   
   # lrwxrwxrwx 1 root root  9 Aug  9 13:20 /dev/disk/by-id/scsi-1LIO-ORG_sbdnfs:afb0ba8d-3a3c-413b-8cc2-cca03e63ef42 -> ../../sdd
   # <b>lrwxrwxrwx 1 root root  9 Aug  9 13:20 /dev/disk/by-id/scsi-36001405afb0ba8d3a3c413b8cc2cca03 -> ../../sdd</b>
   # lrwxrwxrwx 1 root root  9 Aug  9 13:20 /dev/disk/by-id/scsi-SLIO-ORG_sbdnfs_afb0ba8d-3a3c-413b-8cc2-cca03e63ef42 -> ../../sdd
   
   ls -l /dev/disk/by-id/scsi-* | grep <b>sde</b>
   
   # lrwxrwxrwx 1 root root  9 Feb  7 12:39 /dev/disk/by-id/scsi-1LIO-ORG_cl1:3fe4da37-1a5a-4bb6-9a41-9a4df57770e4 -> ../../sde
   # <b>lrwxrwxrwx 1 root root  9 Feb  7 12:39 /dev/disk/by-id/scsi-360014053fe4da371a5a4bb69a419a4df -> ../../sde</b>
   # lrwxrwxrwx 1 root root  9 Feb  7 12:39 /dev/disk/by-id/scsi-SLIO-ORG_cl1_3fe4da37-1a5a-4bb6-9a41-9a4df57770e4 -> ../../sde
   
   ls -l /dev/disk/by-id/scsi-* | grep <b>sdf</b>
   
   # lrwxrwxrwx 1 root root  9 Aug  9 13:32 /dev/disk/by-id/scsi-1LIO-ORG_sbdnfs:f88f30e7-c968-4678-bc87-fe7bfcbdb625 -> ../../sdf
   # <b>lrwxrwxrwx 1 root root  9 Aug  9 13:32 /dev/disk/by-id/scsi-36001405f88f30e7c9684678bc87fe7bf -> ../../sdf</b>
   # lrwxrwxrwx 1 root root  9 Aug  9 13:32 /dev/disk/by-id/scsi-SLIO-ORG_sbdnfs_f88f30e7-c968-4678-bc87-fe7bfcbdb625 -> ../../sdf
   </code></pre>

   Команда выдает три идентификатора для каждого устройства SBD. Мы рекомендуем использовать идентификатор, начинающийся со scsi-3. В приведенном примере это

   * **/dev/disk/by-id/scsi-36001405afb0ba8d3a3c413b8cc2cca03**
   * **/dev/disk/by-id/scsi-360014053fe4da371a5a4bb69a419a4df**
   * **/dev/disk/by-id/scsi-36001405f88f30e7c9684678bc87fe7bf**

1. **[1]** Создайте устройство SBD.

   Используйте идентификаторы устройств iSCSI, чтобы создать устройства SBD на первом узле кластера.

   <pre><code>sudo sbd -d <b>/dev/disk/by-id/scsi-36001405afb0ba8d3a3c413b8cc2cca03</b> -1 60 -4 120 create

   # Also create the second and third SBD devices if you want to use more than one.
   sudo sbd -d <b>/dev/disk/by-id/scsi-360014053fe4da371a5a4bb69a419a4df</b> -1 60 -4 120 create
   sudo sbd -d <b>/dev/disk/by-id/scsi-36001405f88f30e7c9684678bc87fe7bf</b> -1 60 -4 120 create
   </code></pre>

1. **[A]** Адаптируйте конфигурацию SBD.

   Откройте файл конфигурации SBD.

   <pre><code>sudo vi /etc/sysconfig/sbd
   </code></pre>

   Измените значение свойства устройства SBD, включите интеграцию Pacemaker и измените режим запуска SBD.

   <pre><code>[...]
   <b>SBD_DEVICE="/dev/disk/by-id/scsi-36001405afb0ba8d3a3c413b8cc2cca03;/dev/disk/by-id/scsi-360014053fe4da371a5a4bb69a419a4df;/dev/disk/by-id/scsi-36001405f88f30e7c9684678bc87fe7bf"</b>
   [...]
   <b>SBD_PACEMAKER="yes"</b>
   [...]
   <b>SBD_STARTMODE="always"</b>
   [...]
   </code></pre>

   Создание файла конфигурации `softdog`

   <pre><code>echo softdog | sudo tee /etc/modules-load.d/softdog.conf
   </code></pre>

   Теперь загрузите модуль.

   <pre><code>sudo modprobe -v softdog
   </code></pre>

## <a name="cluster-installation"></a>Установка кластера

Ниже приведены элементы с префиксами: **[A]**  — применяется ко всем узлам, **[1**] — применяется только к узлу 1, **[2]**  — применяется только к узлу 2.

1. **[A]** Обновите SLES.

   <pre><code>sudo zypper update
   </code></pre>

1. **[A]** Установка компонента, требуемого для ресурсов кластера

   <pre><code>sudo zypper in socat
   </code></pre>

1. **[A]** Установка компонента azure-lb, требуемого для ресурсов кластера

   <pre><code>sudo zypper in resource-agents
   </code></pre>

   > [!NOTE]
   > Проверьте версию resource-agents пакета и убедитесь, что выполняются минимальные требования к версии.  
   > - Минимальная версия для SLES 12 SP4/SP5 — resource-agents-4.3.018.a7fb5035-3.30.1.  
   > - Минимальная версия для SLES 15/15 с пакетом обновления 1 (SP1) — resource-agents-4.3.0184.6ee15eb2-4.13.1.  

1. **[A]** Настройка операционной системы

   В некоторых случаях Pacemaker создает много процессов и тем самым исчерпывает число допустимых процессов. В таком случае пульс между узлами кластера может завершиться с ошибкой и привести к отработке отказа ресурсов. Рекомендуется увеличить максимальное число разрешенных процессов, установив следующий параметр.

   <pre><code># Edit the configuration file
   sudo vi /etc/systemd/system.conf
   
   # Change the DefaultTasksMax
   #DefaultTasksMax=512
   DefaultTasksMax=4096
   
   #and to activate this setting
   sudo systemctl daemon-reload
   
   # test if the change was successful
   sudo systemctl --no-pager show | grep DefaultTasksMax
   </code></pre>

   Уменьшите размер "грязного" кэша. Дополнительные сведения см. в статье [Low write performance on SLES 11/12 servers with large RAM](https://www.suse.com/support/kb/doc/?id=7010287) (Низкая производительность записи на серверах SLES 11/12 с большим объем ОЗУ).

   <pre><code>sudo vi /etc/sysctl.conf

   # Change/set the following settings
   vm.dirty_bytes = 629145600
   vm.dirty_background_bytes = 314572800
   </code></pre>

1. **[A]** Настройка cloud-netconfig-azure для кластера с высоким уровнем доступности

   >[!NOTE]
   > Проверьте установленную версию пакета **cloud-netconfig-azure**, выполнив **zypper info cloud-netconfig-azure**. Если в вашей среде используется версия 1.3 и выше, вам не нужно подавлять управление сетевыми интерфейсами с помощью подключаемого модуля облачной сети. Если версия ниже 1.3, мы рекомендуем обновить пакет **cloud-netconfig-azure** до последней доступной версии.  

   Измените файл конфигурации для сетевого интерфейса, как показано ниже, чтобы подключаемый модуль облачной сети не препятствовал удалению виртуального IP-адреса (при этом назначением виртуального IP-адреса управляет Pacemaker). См. сведения здесь: [SUSE KB 7023633](https://www.suse.com/support/kb/doc/?id=7023633). 

   <pre><code># Edit the configuration file
   sudo vi /etc/sysconfig/network/ifcfg-eth0 
   
   # Change CLOUD_NETCONFIG_MANAGE
   # CLOUD_NETCONFIG_MANAGE="yes"
   CLOUD_NETCONFIG_MANAGE="no"
   </code></pre>

1. **[1]** Включите доступ по протоколу SSH.

   <pre><code>sudo ssh-keygen
   
   # Enter file in which to save the key (/root/.ssh/id_rsa): -> Press ENTER
   # Enter passphrase (empty for no passphrase): -> Press ENTER
   # Enter same passphrase again: -> Press ENTER
   
   # copy the public key
   sudo cat /root/.ssh/id_rsa.pub
   </code></pre>

1. **[2]** Включите доступ по протоколу SSH.

   <pre><code>
   sudo ssh-keygen
   
   # Enter file in which to save the key (/root/.ssh/id_rsa): -> Press ENTER
   # Enter passphrase (empty for no passphrase): -> Press ENTER
   # Enter same passphrase again: -> Press ENTER
   
   # insert the public key you copied in the last step into the authorized keys file on the second server
   sudo vi /root/.ssh/authorized_keys   
   
   # copy the public key
   sudo cat /root/.ssh/id_rsa.pub
   </code></pre>

1. **[1]** Включите доступ по протоколу SSH.

   <pre><code># insert the public key you copied in the last step into the authorized keys file on the first server
   sudo vi /root/.ssh/authorized_keys
   </code></pre>

1. **[A]** установите пакет агентов ограждения, если используется устройство STONITH на основе агента ограждения Azure.  
   
   <pre><code>sudo zypper install fence-agents
   </code></pre>

   >[!IMPORTANT]
   > Установленная версия **ограждения** пакета. агенты должны быть по крайней мере **4.4.0**  , чтобы воспользоваться преимуществами более быстрого перехода на другой ресурс с помощью агента ограждения Azure, если узлы кластера должны быть ограждены. Рекомендуется обновить пакет, если он работает под управлением более ранней версии.  


1. **[A]** установите пакет SDK для Azure Python. 
   - В SLES 12 SP4 или SLES 12 SP5
   <pre><code>
    # You may need to activate the Public cloud extention first
    SUSEConnect -p sle-module-public-cloud/12/x86_64
    sudo zypper install python-azure-mgmt-compute
   </code></pre> 

   - На SLES 15 и более поздних версий 
   <pre><code>
    # You may need to activate the Public cloud extention first. In this example the SUSEConnect command is for SLES 15 SP1
    SUSEConnect -p sle-module-public-cloud/15.1/x86_64
    sudo zypper install python3-azure-mgmt-compute
   </code></pre> 
 
   >[!IMPORTANT]
   >В зависимости от версии и типа образа может потребоваться активировать расширение общедоступного облака для выпуска ОС, прежде чем можно будет установить пакет SDK для Azure Python.
   >Вы можете проверить расширение, запустив Сусеконнект---List-Extensions.  
   >Чтобы ускорить отработку отказа с помощью агента ограждения Azure, сделайте следующее:
   > - в SLES 12 SP4 или SLES 12 SP5 установка версии **4.6.2** или более поздней версии пакет Python-Azure-управление-вычисление  
   > - в SLES 15 Install версии **4.6.2** или более поздней версии пакет Python **3**— Azure-управление — вычисление 

1. **[A]** Установите разрешения имен.

   Можно использовать DNS-сервер или внести изменения в файл /etc/hosts на всех узлах. В этом примере показано, как использовать файл /etc/hosts.
   Замените IP-адрес и имя узла в следующих командах.

   >[!IMPORTANT]
   > При использовании имен узлов в конфигурации кластера крайне важно иметь надежное разрешение имен узлов. Соединение с кластером завершится ошибкой, если имена недоступны и могут привести к задержкам при отработке отказа кластера.
   > Преимущество использования /etc/hosts в том, что кластер становится независимым от службы DNS, которая также может являться единой точкой отказа.  
     
   <pre><code>sudo vi /etc/hosts

   </code></pre>

   Вставьте следующие строки в /etc/hosts. Измените IP-адрес и имя узла в соответствии со своей средой.   

   <pre><code># IP address of the first cluster node
   <b>10.0.0.6 prod-cl1-0</b>
   # IP address of the second cluster node
   <b>10.0.0.7 prod-cl1-1</b>
   </code></pre>

1. **[1]** Установите кластер.
- При использовании устройств SBD для ограждения
   <pre><code>sudo ha-cluster-init -u
   
   # ! NTP is not configured to start at system boot.
   # Do you want to continue anyway (y/n)? <b>y</b>
   # /root/.ssh/id_rsa already exists - overwrite (y/n)? <b>n</b>
   # Address for ring0 [10.0.0.6] <b>Press ENTER</b>
   # Port for ring0 [5405] <b>Press ENTER</b>
   # SBD is already configured to use /dev/disk/by-id/scsi-36001405639245768818458b930abdf69;/dev/disk/by-id/scsi-36001405afb0ba8d3a3c413b8cc2cca03;/dev/disk/by-id/scsi-36001405f88f30e7c9684678bc87fe7bf - overwrite (y/n)? <b>n</b>
   # Do you wish to configure an administration IP (y/n)? <b>n</b>
   </code></pre>

- Если *не использовать* устройства SBD для ограждения
   <pre><code>sudo ha-cluster-init -u
   
   # ! NTP is not configured to start at system boot.
   # Do you want to continue anyway (y/n)? <b>y</b>
   # /root/.ssh/id_rsa already exists - overwrite (y/n)? <b>n</b>
   # Address for ring0 [10.0.0.6] <b>Press ENTER</b>
   # Port for ring0 [5405] <b>Press ENTER</b>
   # Do you wish to use SBD (y/n)? <b>n</b>
   #WARNING: Not configuring SBD - STONITH will be disabled.
   # Do you wish to configure an administration IP (y/n)? <b>n</b>
   </code></pre>

1. **[2]** Добавьте узел в кластер.

   <pre><code>sudo ha-cluster-join
   
   # ! NTP is not configured to start at system boot.
   # Do you want to continue anyway (y/n)? <b>y</b>
   # IP address or hostname of existing node (e.g.: 192.168.1.1) []<b>10.0.0.6</b>
   # /root/.ssh/id_rsa already exists - overwrite (y/n)? <b>n</b>
   </code></pre>

1. **[A]** Измените пароль hacluster на тот же пароль.

   <pre><code>sudo passwd hacluster
   </code></pre>

1. **[A]** Настройка параметров corosync  

   <pre><code>sudo vi /etc/corosync/corosync.conf
   </code></pre>

   Добавьте следующее содержимое, выделенное полужирным шрифтом, в файл, если значения отсутствуют или отличаются. Не забудьте заменить токен на 30 000, чтобы разрешить обслуживание с сохранением памяти. См. сведения в статьях о [Linux][virtual-machines-linux-maintenance] и [Windows][virtual-machines-windows-maintenance].

   <pre><code>[...]
     <b>token:          30000
     token_retransmits_before_loss_const: 10
     join:           60
     consensus:      36000
     max_messages:   20</b>
     
     interface { 
        [...] 
     }
     transport:      udpu
   } 
   nodelist {
     node {
      ring0_addr:10.0.0.6
     }
     node {
      ring0_addr:10.0.0.7
     } 
   }
   logging {
     [...]
   }
   quorum {
        # Enable and configure quorum subsystem (default: off)
        # see also corosync.conf.5 and votequorum.5
        provider: corosync_votequorum
        <b>expected_votes: 2</b>
        <b>two_node: 1</b>
   }
   </code></pre>

   Затем перезапустите службу corosync.

   <pre><code>sudo service corosync restart
   </code></pre>

## <a name="default-pacemaker-configuration-for-sbd"></a>Конфигурация Pacemaker по умолчанию для SBD

Конфигурация в этом разделе применима только при использовании SBD STONITH.  

1. **[1]** Активируйте использование устройства STONITH и задайте задержку ограничения.

<pre><code>sudo crm configure property stonith-timeout=144
sudo crm configure property stonith-enabled=true

# List the resources to find the name of the SBD device
sudo crm resource list
sudo crm resource stop stonith-sbd
sudo crm configure delete <b>stonith-sbd</b>
sudo crm configure primitive <b>stonith-sbd</b> stonith:external/sbd \
   params pcmk_delay_max="15" \
   op monitor interval="15" timeout="15"
</code></pre>

## <a name="create-azure-fence-agent-stonith-device"></a>Создание устройства STONITH с ролью агента ограждения Azure

Этот раздел документации применим только при использовании STONITH на основе агента ограждения Azure.
Устройство STONITH использует субъект-службу для авторизации в Microsoft Azure. Чтобы создать субъект-службу, выполните следующее.

1. Перейдите на сайт <https://portal.azure.com>.
1. Откройте колонку "Azure Active Directory".  
   Перейдите в колонку "Свойства" и запишите идентификатор каталога. Это **идентификатор клиента**.
1. Щелкните "Регистрация приложений".
1. Щелкните "Новая регистрация".
1. Введите имя и выберите "Учетные записи только из этого каталога организации". 
2. Выберите тип приложения "Веб", введите URL-адрес входа (например, http:\//localhost) и нажмите кнопку "Добавить".  
   URL-адрес входа не используется и может быть любым допустимым URL-адресом.
1. Выберите "Сертификаты и секреты", а затем щелкните "Новый секрет клиента".
1. Введите описание нового ключа, выберите "Срок действия не ограничен" и нажмите кнопку "Добавить".
1. Запишите его значение. Он используется в качестве **пароля** субъекта-службы.
1. Щелкните "Обзор". Запишите идентификатор приложения. Он используется в качестве имени пользователя (**идентификатора для входа** в следующих шагах) субъекта-службы.

### <a name="1-create-a-custom-role-for-the-fence-agent"></a>**[1]** Создайте пользовательскую роль для агента ограждения.

У субъекта-службы по умолчанию нет разрешений на доступ к ресурсам Azure. Необходимо предоставить ему разрешения на запуск и остановку (освобождение) всех виртуальных машин кластера. Если вы еще не создали эту пользовательскую роль, ее можно создать с помощью [PowerShell](../../../role-based-access-control/custom-roles-powershell.md#create-a-custom-role) или [Azure CLI](../../../role-based-access-control/custom-roles-cli.md).

Используйте следующее содержимое для входного файла. Необходимо адаптировать содержимое для ваших подписок, поэтому замените c276fc76-9cd4-44c9-99a7-4fd71546436e и e91d47c4-76f3-4271-a796-21b4ecfe3624 идентификаторами своих подписок. Если у вас имеется только одна подписка, удалите вторую запись в AssignableScopes.

```json
{
    "properties": {
        "roleName": "Linux Fence Agent Role",
        "description": "Allows to power-off and start virtual machines",
        "assignableScopes": [
            "/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e",
            "/subscriptions/e91d47c4-76f3-4271-a796-21b4ecfe3624"
        ],
        "permissions": [
            {
                "actions": [
                    "Microsoft.Compute/*/read",
                    "Microsoft.Compute/virtualMachines/powerOff/action",
                    "Microsoft.Compute/virtualMachines/start/action"
                ],
                "notActions": [],
                "dataActions": [],
                "notDataActions": []
            }
        ]
    }
}
```

### <a name="a-assign-the-custom-role-to-the-service-principal"></a>**[A]** Назначьте пользовательскую роль субъекту-службе.

Назначьте субъекту-службе пользовательскую роль Linux Fence Agent Role, созданную в последней главе. Больше не используйте роль владельца!

1. Перейдите по адресу [https://portal.azure.com](https://portal.azure.com).
1. Откройте колонку "Все ресурсы".
1. Выберите виртуальную машину первого узла кластера.
1. Выберите "Управление доступом (IAM)".
1. Выберите "Добавить назначение ролей".
1. Выберите роль Linux Fence Agent Role.
1. Введите имя созданного ранее приложения.
1. Щелкните Сохранить

Повторите предыдущие шаги для второго узла кластера.

### <a name="1-create-the-stonith-devices"></a>**[1]** Создайте устройства STONITH.

После изменения разрешений для виртуальных машин можно настроить устройства STONITH в кластере.

> [!NOTE]
> Параметр "pcmk_host_map" требуется только в команде, если имена узлов и имена виртуальных машин Azure не идентичны. Укажите сопоставление в формате имя **узла: VM-Name**.
> См. раздел команды, выделенный полужирным шрифтом.

<pre><code>sudo crm configure property stonith-enabled=true
crm configure property concurrent-fencing=true
# replace the bold string with your subscription ID, resource group, tenant ID, service principal ID and password
sudo crm configure primitive rsc_st_azure stonith:fence_azure_arm \
  params subscriptionId="<b>subscription ID</b>" resourceGroup="<b>resource group</b>" tenantId="<b>tenant ID</b>" login="<b>login ID</b>" passwd="<b>password</b>" \
  pcmk_monitor_retries=4 pcmk_action_limit=3 power_timeout=240 pcmk_reboot_timeout=900 <b>pcmk_host_map="prod-cl1-0:prod-cl1-0-vm-name;prod-cl1-1:prod-cl1-1-vm-name"</b> \
  op monitor interval=3600 timeout=120

sudo crm configure property stonith-timeout=900

</code></pre>

> [!IMPORTANT]
> Операции мониторинга и ограждения десериализованы. В результате, если выполняется более длительная операция наблюдения и одновременное событие ограждения, задержка на отработку отказа кластера отсутствует из-за уже запущенной операции мониторинга.

> [!TIP]
>Для агента ограждения Azure требуется исходящее подключение к общедоступным конечным точкам. См. сведения об этом требовании и о возможных решениях в статье [Подключение к общедоступной конечной точке для виртуальных машин с помощью Azure Load Balancer (цен. категория "Стандартный") в сценариях с SAP с высоким уровнем доступности](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

## <a name="pacemaker-configuration-for-azure-scheduled-events"></a>Конфигурация Pacemaker для запланированных событий Azure

Azure предлагает [запланированные события](../../linux/scheduled-events.md). Запланированные события предоставляются через службу метаданных и позволяют приложению подготовиться к таким событиям, как завершение работы виртуальной машины, ее повторное развертывание и т. д. Агент ресурсов **[azure-events](https://github.com/ClusterLabs/resource-agents/pull/1161)** выполняет мониторинг запланированных событий Azure. Если события обнаружены, агент попытается закрыть все ресурсы на затронутой виртуальной машине и переместить их на другой узел в кластере. Для этого нужно настроить дополнительные ресурсы Pacemaker. 

1. **[A]** Убедитесь, что пакет для агента **azure-events** установлен и обновлен. 

<pre><code>sudo zypper info resource-agents
</code></pre>

2. **[1]** Настройте ресурсы в Pacemaker. 

<pre><code>
#Place the cluster in maintenance mode
sudo crm configure property maintenance-mode=true

#Create Pacemaker resources for the Azure agent
sudo crm configure primitive rsc_azure-events ocf:heartbeat:azure-events op monitor interval=10s
sudo crm configure clone cln_azure-events rsc_azure-events

#Take the cluster out of maintenance mode
sudo crm configure property maintenance-mode=false
</code></pre>

   > [!NOTE]
   > После настройки ресурсов Pacemaker для агента azure-events при включении и отключении режима обслуживания кластера могут появиться следующие сообщения с предупреждениями:  
     WARNING: cib-bootstrap-options: unknown attribute 'hostName_  <strong>hostname</strong>'  
     WARNING: cib-bootstrap-options: unknown attribute 'azure-events_globalPullState'  
     WARNING: cib-bootstrap-options: unknown attribute 'hostName_ <strong>hostname</strong>'  
   > Эти предупреждения можно игнорировать.

## <a name="next-steps"></a>Дальнейшие действия

* [Планирование и реализация виртуальных машин Azure для SAP][planning-guide]
* [Развертывание виртуальных машин Azure для SAP NetWeaver][deployment-guide]
* [Развертывание СУБД виртуальных машин Azure для SAP][dbms-guide]
* [Обеспечение высокого уровня доступности NFS на виртуальных машинах Azure в SUSE Linux Enterprise Server][sles-nfs-guide]
* [Руководство по обеспечению высокого уровня доступности виртуальных машин Azure для SAP NetWeaver на SUSE Linux Enterprise Server для приложений SAP][sles-guide]
* См. сведения в статье [Обеспечение высокого уровня доступности SAP HANA на виртуальных машинах Azure в SUSE Linux Enterprise Server][sap-hana-ha].
