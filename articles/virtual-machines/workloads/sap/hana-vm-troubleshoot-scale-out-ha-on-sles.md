---
title: SAP HANA горизонтального масштабирования HSR-Pacemaker с помощью SLES на виртуальных машинах Azure | Документация Майкрософт
description: Руководство по проверке и устранению неполадок сложной конфигурации горизонтального масштабирования SAP HANA с высоким уровнем доступности на основе репликации системы SAP HANA (HSR) и Pacemaker в SLES 12 с пакетом обновления 3 (SP3) на виртуальных машинах Azure
services: virtual-machines-linux
documentationcenter: ''
author: hermanndms
manager: juergent
editor: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/24/2018
ms.author: hermannd
ms.openlocfilehash: e64abc008433c895e21690ccab8cc532ec1b34b1
ms.sourcegitcommit: 44edde1ae2ff6c157432eee85829e28740c6950d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105543862"
---
# <a name="verify-and-troubleshoot-sap-hana-scale-out-high-availability-setup-on-sles-12-sp3"></a>Проверка и устранение неполадок при настройке горизонтального масштабирования SAP HANA с высоким уровнем доступности в SLES 12 SP3 

[sles-pacemaker-ha-guide]:high-availability-guide-suse-pacemaker.md
[sles-hana-scale-out-ha-paper]:https://www.suse.com/documentation/suse-best-practices/singlehtml/SLES4SAP-hana-scaleOut-PerfOpt-12/SLES4SAP-hana-scaleOut-PerfOpt-12.html
[sap-hana-iaas-list]:https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html
[suse-pacemaker-support-log-files]:https://www.suse.com/support/kb/doc/?id=7022702
[azure-linux-multiple-nics]:https://docs.microsoft.com/azure/virtual-machines/linux/multiple-nics
[suse-cloud-netconfig]:https://www.suse.com/c/multi-nic-cloud-netconfig-ec2-azure/
[sap-list-port-numbers]:https://help.sap.com/viewer/ports
[sles-12-ha-paper]:https://www.suse.com/documentation/sle-ha-12/pdfdoc/book_sleha/book_sleha.pdf
[sles-zero-downtime-paper]:https://www.youtube.com/embed/0FW3J6GbxOk
[sap-nw-ha-guide-sles]:high-availability-guide-suse.md
[sles-12-for-sap]:https://www.scribd.com/document/377847444/Suse-Linux-Enterprise-Server-for-Sap-Applications-12-Sp1


Эта статья поможет вам проверить конфигурацию кластера Pacemaker для горизонтального масштабирования SAP HANA на виртуальных машинах Azure. Настройка кластера выполнялась с репликацией системы SAP HANA (HSR) и пакета SUSE RPM SAPHanaSR-ScaleOut. Все тесты проводились только на SUSE SLES 12 с пакетом обновления 3 (SP3). Разделы этой статьи охватывают разные области и содержат примеры команд и выдержки из файлов конфигурации. Рекомендуем использовать эти примеры для полной проверки конфигурации кластера.



## <a name="important-notes"></a>Важные примечания

Все тесты для горизонтального масштабирования SAP HANA в сочетании с репликацией системы SAP HANA и Pacemaker выполнялись только с SAP HANA 2.0. Для приложений SAP использовалась операционная система SUSE Linux Enterprise Server 12 с пакетом обновления 3 (SP3). Для настройки кластера Pacemaker использовалась последняя версия пакета RPM SAPHanaSR-ScaleOut от SUSE.
Компания SUSE опубликовала [подробное описание этой конфигурации, оптимизированной для высокой производительности][sles-hana-scale-out-ha-paper].

Типы виртуальных машин, которые поддерживаются для горизонтального масштабирования SAP HANA, указаны в [каталоге систем IaaS, сертифицированных для SAP HANA][sap-hana-iaas-list].

> [!NOTE]
> Эта статья содержит ссылки на *основные* и *подчиненные* термины, которые корпорация Майкрософт больше не использует. При удалении этих терминов из программного обеспечения мы удалим их из этой статьи.

Существовала техническая проблема с горизонтальным масштабированием SAP HANA в сочетании с несколькими подсетями и виртуальными сетевыми картами, а также настройкой HSR. Обязательно используйте последние исправления SAP HANA 2.0, где исправлена ​​эта проблема. Поддерживаются следующие версии SAP HANA: 

* выпуск 2.00.024.04 или выше; 
* выпуск 2.00.032 или выше.

Если вам нужна поддержка от SUSE, воспользуйтесь этим [руководством][suse-pacemaker-support-log-files]. Соберите всю информацию о кластере SAP HANA с высоким уровнем доступности (HA), как описано в этой статье. Службе поддержке SUSE нужна эта информация для дальнейшего анализа.

Во время внутреннего тестирования конфигурация кластера иногда зависала при нормальном завершении работы виртуальной машины на портале Azure. Поэтому рекомендуем проверять отказоустойчивость кластера другими способами. Используйте такие способы, как принудительное инициирование критического состояния ядра, выключение сетей или перенос ресурса **msl**. Дополнительные сведения приведены в следующих разделах. Предполагается, что стандартное завершение работы происходит намеренно. Лучший пример преднамеренного завершения работы — техническое обслуживание. Дополнительные сведения см. в разделе [Плановое техническое обслуживание](#planned-maintenance).

Кроме того, во время внутреннего тестирования конфигурация кластера зависала после перехвата SAP HANA в ручном режиме, когда кластер находился в режиме обслуживания. Рекомендуем заново включить его вручную, прежде чем выходить из режима обслуживания кластера. Еще один вариант — вызвать отработку отказа, прежде чем переводить кластер в режим обслуживания. Дополнительные сведения см. в разделе [Плановое техническое обслуживание](#planned-maintenance). В документации SUSE описано, как можно выполнить сброс кластера в таком случае при помощи команды **crm**. Однако упомянутый выше подход показал свою надежность во время внутреннего тестирования и никогда не вызывал каких-либо неожиданных побочных эффектов.

При использовании команды **crm migrate** обязательно очищайте конфигурацию кластера. При этом создаются дополнительные ограничения расположения, о которых вы можете не знать. Эти ограничения влияют на поведение кластера. Дополнительные сведения см. в разделе [Плановое техническое обслуживание](#planned-maintenance).



## <a name="test-system-description"></a>Описание системы тестирования

 Для проверки и сертификации высокого уровня доступности при горизонтальном масштабировании SAP HANA использовалась тестовая конфигурация. Она состояла из двух систем, каждая из которых включала три узла SAP HANA: один главный и два рабочих. В приведенной ниже таблице перечислены имена и внутренние IP-адреса виртуальных машин. Все приведенные ниже примеры проверки реализовывались на этих виртуальных машинах. Используя эти имена и IP-адреса виртуальных машин в примерах команд, вы можете лучше понять команды и их выходные данные.


| Тип узла | имя виртуальной машины; | IP-адрес |
| --- | --- | --- |
| Главный узел на сайте 1 | hso-hana-vm-s1-0 | 10.0.0.30 |
| Рабочий узел 1 на сайте 1 | hso-hana-vm-s1-1 | 10.0.0.31 |
| Рабочий узел 2 на сайте 1 | hso-hana-vm-s1-2 | 10.0.0.32 |
| | | |
| Главный узел на сайте 2 | hso-hana-vm-s2-0 | 10.0.0.40 |
| Рабочий узел 1 на сайте 2 | hso-hana-vm-s2-1 | 10.0.0.41 |
| Рабочий узел 2 на сайте 2 | hso-hana-vm-s2-2  | 10.0.0.42 |
| | | |
| Узел создания большинства | hso-hana-dm | 10.0.0.13 |
| Сервер устройств SBD | hso-hana-sbd | 10.0.0.19 |
| | | |
| Сервер NFS 1 | hso-nfs-vm-0 | 10.0.0.15 |
| Сервер NFS 2 | hso-nfs-vm-1 | 10.0.0.14 |



## <a name="multiple-subnets-and-vnics"></a>Несколько подсетей и виртуальных сетевых адаптеров

В соответствии с рекомендациями по сети SAP HANA в одной виртуальной сети Azure были созданы три подсети. Горизонтальное масштабирование SAP HANA в Azure необходимо выполнять в необщем режиме. Это означает, что каждый узел использует тома локальных дисков для папок **/hana/data** и **/hana/log**. Так как узлы используют только тома локальных дисков, нет необходимости определять отдельную подсеть для хранения:

- 10.0.2.0/24 для обмена данными между узлами SAP HANA;
- 10.0.1.0/24 для репликации системы SAP HANA (HSR);
- 10.0.0.0/24 для всего остального.

Сведения о конфигурации SAP HANA, связанные с использованием нескольких сетей, см. в разделе [SAP HANA global.ini](#sap-hana-globalini).

Каждая виртуальная машина содержит три виртуальных сетевых адаптера в соответствии с количеством подсетей. В статье [Как создать виртуальную машину Linux в Azure с несколькими сетевыми адаптерами][azure-linux-multiple-nics] описывается потенциальная проблема с маршрутизацией в Azure при развертывании виртуальной машины Linux. Она касается только использования нескольких виртуальных сетевых адаптеров. Проблема разрешена SUSE по умолчанию в SLES 12 SP3. Дополнительные сведения см. в статье [Использование нескольких сетевых адаптером с пакетом cloud-netconfig в EC2 и Azure][suse-cloud-netconfig].


Чтобы убедиться, что платформа SAP HANA правильно настроена на использование нескольких сетей, выполните приведенные ниже команды. Для начала убедитесь, что все три внутренних IP-адреса подсетей активны на уровне ОС. Если вы определили подсети с разными диапазонами IP-адресов, вам необходимо адаптировать команды:

<pre><code>
ifconfig | grep "inet addr:10\."
</code></pre>

Ниже приведен пример выходных данных второго рабочего узла на сайте 2. В нем видно три разных внутренних IP-адреса от eth0, eth1 и eth2:

<pre><code>
inet addr:10.0.0.42  Bcast:10.0.0.255  Mask:255.255.255.0
inet addr:10.0.1.42  Bcast:10.0.1.255  Mask:255.255.255.0
inet addr:10.0.2.42  Bcast:10.0.2.255  Mask:255.255.255.0
</code></pre>


Затем проверьте порты SAP HANA для сервера доменных имен и HSR. SAP HANA должен прослушивать соответствующие подсети. В зависимости от номера экземпляра SAP HANA вам необходимо адаптировать команды. Для тестовой системы номером экземпляра был **00**. Определять используемые порты можно различными способами. 

Следующая инструкция SQL возвращает ИД и номер экземпляра, а также другие сведения:

<pre><code>
select * from "SYS"."M_SYSTEM_OVERVIEW"
</code></pre>

Чтобы найти правильные номера портов, можно просмотреть, например, в HANA Studio в **конфигурации** или с помощью инструкции SQL:

<pre><code>
select * from M_INIFILE_CONTENTS WHERE KEY LIKE 'listen%'
</code></pre>

Каждый порт, который используется в стеке программного обеспечения SAP, включая SAP HANA, можно найти в списке [портов TCP/IP для всех продуктов SAP][sap-list-port-numbers].

Если номер экземпляра в тестовой системе SAP HANA 2.0 — **00**, то номер порта для сервера имен — **30001**. Номер порта для обмена метаданными HSR — **40002**. Один из вариантов — войти в рабочий узел, а затем проверить службы главного узла. Для этой статьи мы проверили рабочий узел 2 на сайте 2, пытающемся подключиться к главному узлу на сайте 2.

Проверьте порт сервера доменных имен:

<pre><code>
nc -vz 10.0.0.40 30001
nc -vz 10.0.1.40 30001
nc -vz 10.0.2.40 30001
</code></pre>

Чтобы доказать, что для взаимодействия между узлами используется подсеть **10.0.2.0/24**, результат должен выглядеть так, как в приведенном ниже примере.
Только подключение через подсеть **10.0.2.0/24** должно установиться успешно:

<pre><code>
nc: connect to 10.0.0.40 port 30001 (tcp) failed: Connection refused
nc: connect to 10.0.1.40 port 30001 (tcp) failed: Connection refused
Connection to 10.0.2.40 30001 port [tcp/pago-services1] succeeded!
</code></pre>

Теперь проверьте порт HSR **40002**:

<pre><code>
nc -vz 10.0.0.40 40002
nc -vz 10.0.1.40 40002
nc -vz 10.0.2.40 40002
</code></pre>

Чтобы доказать, что для взаимодействия HSR используется подсеть **10.0.1.0/24**, результат должен выглядеть так, как в приведенном ниже примере.
Только подключение через подсеть **10.0.1.0/24** должно установиться успешно:

<pre><code>
nc: connect to 10.0.0.40 port 40002 (tcp) failed: Connection refused
Connection to 10.0.1.40 40002 port [tcp/*] succeeded!
nc: connect to 10.0.2.40 port 40002 (tcp) failed: Connection refused
</code></pre>



## <a name="corosync"></a>Corosync


Файл конфигурации **corosync** должен быть правильным на каждом узле кластера, включая узел, образующий большинство. Если присоединение узла к кластеру не работает должным образом, создайте или скопируйте файл **/etc/corosync/corosync.conf** вручную на все узлы и перезапустите службу. 

В качестве примера приведено содержимое файла **corosync.conf** из тестовой системы.

Первый раздел — это **totem**, как описано в разделе [Установка кластера](./high-availability-guide-suse-pacemaker.md#cluster-installation) (шаг 11). Вы можете игнорировать значение для **mcastaddr**. Просто сохраните существующую запись. Записи для **token** и **consensus** должны быть настроены в соответствии с [документацией по SAP HANA для Microsoft Azure][sles-pacemaker-ha-guide].

<pre><code>
totem {
    version:    2
    secauth:    on
    crypto_hash:    sha1
    crypto_cipher:  aes256
    cluster_name:   hacluster
    clear_node_high_bit: yes

    token:      30000
    token_retransmits_before_loss_const: 10
    join:       60
    consensus:  36000
    max_messages:   20

    interface {
        ringnumber:     0
        bindnetaddr: 10.0.0.0
        mcastaddr:  239.170.19.232
        mcastport:  5405

        ttl:        1
    }
    transport:      udpu

}
</code></pre>

Для второго раздела (**logging**) используются значения по умолчанию:

<pre><code>
logging {
    fileline:   off
    to_stderr:  no
    to_logfile:     no
    logfile:    /var/log/cluster/corosync.log
    to_syslog:  yes
    debug:      off
    timestamp:  on
    logger_subsys {
        subsys:     QUORUM
        debug:  off
    }
}
</code></pre>

В третьем разделе показан элемент **nodelist**. Все узлы кластера должны отображаться с идентификаторами **nodeid**:

<pre><code>
nodelist {
  node {
   ring0_addr:hso-hana-vm-s1-0
   nodeid: 1
   }
  node {
   ring0_addr:hso-hana-vm-s1-1
   nodeid: 2
   }
  node {
   ring0_addr:hso-hana-vm-s1-2
   nodeid: 3
   }
  node {
   ring0_addr:hso-hana-vm-s2-0
   nodeid: 4
   }
  node {
   ring0_addr:hso-hana-vm-s2-1
   nodeid: 5
   }
  node {
   ring0_addr:hso-hana-vm-s2-2
   nodeid: 6
   }
  node {
   ring0_addr:hso-hana-dm
   nodeid: 7
   }
}
</code></pre>

В последнем разделе (**quorum**) важно правильно задать значение для параметра **expected_votes**. Это должно быть количество узлов, включая узел создания большинства. Для свойства **two_node** должно быть задано значение **0**. Не удаляйте запись полностью. Просто установите значение **0**.

<pre><code>
quorum {
    # Enable and configure quorum subsystem (default: off)
    # see also corosync.conf.5 and votequorum.5
    provider: corosync_votequorum
    expected_votes: 7
    two_node: 0
}
</code></pre>


Перезапустите службу с помощью **systemctl**:

<pre><code>
systemctl restart corosync
</code></pre>




## <a name="sbd-device"></a>Устройство SBD

Сведения о том, как настроить устройство SBD на виртуальной машине Azure, представлены в разделе [Ограждение SBD](./high-availability-guide-suse-pacemaker.md#sbd-fencing).

Для начала убедитесь, что на сервере SBD имеются записи ACL для каждого узла кластера. Выполните следующую команду на виртуальной машине сервера SBD:


<pre><code>
targetcli ls
</code></pre>


В тестовой системе выходные данные команды выглядят, как в приведенном ниже примере. Имена ACL наподобие **iqn.2006-04.hso-db-0.local:hso-db-0** необходимо указать в качестве соответствующих имен инициаторов на виртуальных машинах. Для всех виртуальных машин нужны разные имена.

<pre><code>
 | | o- sbddbhso ................................................................... [/sbd/sbddbhso (50.0MiB) write-thru activated]
  | |   o- alua ................................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 1]
  | o- iqn.2006-04.dbhso.local:dbhso ..................................................................................... [TPGs: 1]
  |   o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  |     o- acls .......................................................................................................... [ACLs: 7]
  |     | o- iqn.2006-04.hso-db-0.local:hso-db-0 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-1.local:hso-db-1 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-2.local:hso-db-2 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-3.local:hso-db-3 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-4.local:hso-db-4 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-5.local:hso-db-5 .................................................................. [Mapped LUNs: 1]
  |     | | o- mapped_lun0 ............................................................................. [lun0 fileio/sbddbhso (rw)]
  |     | o- iqn.2006-04.hso-db-6.local:hso-db-6 .................................................................. [Mapped LUNs: 1]
</code></pre>

Затем убедитесь, что имена инициаторов на всех виртуальных машинах отличаются и соответствуют представленным выше записям. Этот пример взят из рабочего узла 1 на сайте 1:

<pre><code>
cat /etc/iscsi/initiatorname.iscsi
</code></pre>

Выходные данные выглядят так:

<pre><code>
##
## /etc/iscsi/iscsi.initiatorname
##
## Default iSCSI Initiatorname.
##
## DO NOT EDIT OR REMOVE THIS FILE!
## If you remove this file, the iSCSI daemon will not start.
## If you change the InitiatorName, existing access control lists
## may reject this initiator.  The InitiatorName must be unique
## for each iSCSI initiator.  Do NOT duplicate iSCSI InitiatorNames.
InitiatorName=iqn.2006-04.hso-db-1.local:hso-db-1
</code></pre>

Затем убедитесь, что **обнаружение** работает должным образом. Выполните следующую команду на каждом узле кластера с использованием IP-адреса виртуальной машины сервера SBD:

<pre><code>
iscsiadm -m discovery --type=st --portal=10.0.0.19:3260
</code></pre>

Выходные данные должны выглядеть так, как в следующем примере:

<pre><code>
10.0.0.19:3260,1 iqn.2006-04.dbhso.local:dbhso
</code></pre>

Далее необходимо убедиться, что узел видит устройство SDB. Проверьте видимость на каждом узле, включая узел создания большинства:

<pre><code>
lsscsi | grep dbhso
</code></pre>

Полученный результат должен выглядеть примерно так: Однако имена могут быть другими. Имя устройства также может измениться после перезагрузки виртуальной машины:

<pre><code>
[6:0:0:0]    disk    LIO-ORG  sbddbhso         4.0   /dev/sdm
</code></pre>

В зависимости от состояния системы иногда решить проблемы можно путем перезапуска служб iSCSI. Затем выполните следующие команды:

<pre><code>
systemctl restart iscsi
systemctl restart iscsid
</code></pre>


На любом узле можно проверить, **очищены** ли все узлы. Убедитесь, что используется правильное имя устройства на определенном узле:

<pre><code>
sbd -d /dev/sdm list
</code></pre>

В выходных данных для каждого узла в кластере должно отображаться значение **clear**:

<pre><code>
0       hso-hana-vm-s1-0        clear
1       hso-hana-vm-s2-2        clear
2       hso-hana-vm-s2-1        clear
3       hso-hana-dm     clear
4       hso-hana-vm-s1-1        clear
5       hso-hana-vm-s2-0        clear
6       hso-hana-vm-s1-2        clear
</code></pre>


Другая проверка SBD — это использование параметра **dump** команды **sbd**. В этом примере команды и выходных данных с узла, образующего большинство, указано имя устройства **sdd**, а не **sdm**:

<pre><code>
sbd -d /dev/sdd dump
</code></pre>

Выходные данные (кроме имени устройства) должны выглядеть одинаково на всех узлах:

<pre><code>
==Dumping header on disk /dev/sdd
Header version     : 2.1
UUID               : 9fa6cc49-c294-4f1e-9527-c973f4d5a5b0
Number of slots    : 255
Sector size        : 512
Timeout (watchdog) : 60
Timeout (allocate) : 2
Timeout (loop)     : 1
Timeout (msgwait)  : 120
==Header on disk /dev/sdd is dumped
</code></pre>

Еще одна проверка для SBD — это возможность отправить сообщение на другой узел. Чтобы отправить сообщение на рабочий узел 2 на сайте 2, выполните следующую команду на рабочем узле 1 на сайте 2:

<pre><code>
sbd -d /dev/sdm message hso-hana-vm-s2-2 test
</code></pre>

На стороне целевой виртуальной машины (в этом примере — **hso-hana-vm-s2-2**) вы найдете следующую запись в **/var/log/messages**:

<pre><code>
/dev/disk/by-id/scsi-36001405e614138d4ec64da09e91aea68:   notice: servant: Received command test from hso-hana-vm-s2-1 on disk /dev/disk/by-id/scsi-36001405e614138d4ec64da09e91aea68
</code></pre>

Убедитесь, что записи в папке **/etc/sysconfig/sbd** соответствуют описанию из раздела [Настройка кластера Pacemaker в SUSE Linux Enterprise Server в Azure](./high-availability-guide-suse-pacemaker.md#sbd-fencing). Убедитесь, что в **/etc/iscsi/iscsid.conf** установлен параметр автоматического запуска.

Ниже перечислены важные записи из папки **/etc/sysconfig/sbd**. По мере необходимости адаптируйте значение **id**:

<pre><code>
SBD_DEVICE="/dev/disk/by-id/scsi-36001405e614138d4ec64da09e91aea68;"
SBD_PACEMAKER=yes
SBD_STARTMODE=always
SBD_WATCHDOG=yes
</code></pre>


Проверьте параметр запуска в файле **/etc/iscsi/iscsid.conf**. Нужный параметр должен был настроиться с помощью приведенной ниже команды **iscsiadm**, которая описана в документации. Проверьте и адаптируйте его вручную с помощью команды **vi**, если он отличается.

Эта команда задает поведение при запуске:

<pre><code>
iscsiadm -m node --op=update --name=node.startup --value=automatic
</code></pre>

Создайте следующую запись в файле **/etc/iscsi/iscsid.conf**:

<pre><code>
node.startup = automatic
</code></pre>

Во время тестирования и проверок возникали случаи, когда после перезагрузки виртуальной машины устройство SBD больше не обнаруживалось. Возникало несоответствие между настройкой запуска и показателем YaST2. Чтобы проверить параметры, сделайте следующее:

1. Запустите YaST2.
2. Выберите **Network Services** (Сетевые службы) слева.
3. Прокрутите вниз с правой стороны до элемента **iSCSI Initiator** (Инициатор iSCSI) и выберите его.
4. На следующем экране на вкладке **Service** (Служба) вы увидите уникальное имя инициатора для узла.
5. Убедитесь, что над именем инициатора для параметра **Service Start** (Запустить службу) установлено значение **When Booting** (При загрузке).
6. Если это не так, установите значение **When Booting** (При загрузке) вместо **Manually** (Вручную).
7. Затем переключите верхнюю вкладку на **Connected Targets** (Подключенные целевые объекты).
8. На экране **подключенные целевые объекты** должна отобразиться запись для устройства SBD, как в следующем примере: **10.0.0.19:3260 IQN. 2006-04. дбхсо. local: дбхсо**.
9. Проверьте, задано ли для параметра **Start-Up** (Запуск) значение **On boot** (При загрузке).
10. Если это не так, нажмите **Edit** (Изменить) и измените его.
11. Сохраните изменения и выйдите из YaST2.



## <a name="pacemaker"></a>Pacemaker

Когда все будет настроено должным образом, вы можете запустить следующую команду на каждом узле, чтобы проверить состояние службы Pacemaker:

<pre><code>
systemctl status pacemaker
</code></pre>

Начало полученного результата должно выглядеть примерно так: Важно, чтобы состояние после **Active** (Активное) отображалось как **loaded** (загружено) и **active (running)** (активно (работает)). После надписи **Loaded** (Загружено) должно отображаться состояние **enabled** (включено).

<pre><code>
  pacemaker.service - Pacemaker High Availability Cluster Manager
   Loaded: loaded (/usr/lib/systemd/system/pacemaker.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2018-09-07 05:56:27 UTC; 4 days ago
     Docs: man:pacemakerd
           http://clusterlabs.org/doc/en-US/Pacemaker/1.1-pcs/html/Pacemaker_Explained/index.html
 Main PID: 4496 (pacemakerd)
    Tasks: 7 (limit: 4915)
   CGroup: /system.slice/pacemaker.service
           ├─4496 /usr/sbin/pacemakerd -f
           ├─4499 /usr/lib/pacemaker/cib
           ├─4500 /usr/lib/pacemaker/stonithd
           ├─4501 /usr/lib/pacemaker/lrmd
           ├─4502 /usr/lib/pacemaker/attrd
           ├─4503 /usr/lib/pacemaker/pengine
           └─4504 /usr/lib/pacemaker/crmd
</code></pre>

Если по-прежнему отображается состояние **disabled** (отключено), выполните следующую команду:

<pre><code>
systemctl enable pacemaker
</code></pre>

Чтобы просмотреть все настроенные ресурсы в Pacemaker, выполните следующую команду:

<pre><code>
crm status
</code></pre>

Полученный результат должен выглядеть примерно так: Если ресурсы **cln** и **msl** показаны как остановленные на виртуальной машине, образующей большинство (**hso-hana-dm**), это нормально. На узле, образующем большинство, нет установки SAP HANA. Поэтому ресурсы **cln** и **msl** показаны как остановленные. Важно, чтобы на нем было указано правильное общее число виртуальных машин, **7**. Все виртуальные машины, входящие в состав кластера, должны находиться в состоянии **Online** (В сети). Текущий основной главный узел должен быть правильно распознан. В данном примере это **hso-hana-vm-s1-0**:

<pre><code>
Stack: corosync
Current DC: hso-hana-dm (version 1.1.16-4.8-77ea74d) - partition with quorum
Last updated: Tue Sep 11 15:56:40 2018
Last change: Tue Sep 11 15:56:23 2018 by root via crm_attribute on hso-hana-vm-s1-0

7 nodes configured
17 resources configured

Online: [ hso-hana-dm hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started hso-hana-dm
 Clone Set: cln_SAPHanaTop_HSO_HDB00 [rsc_SAPHanaTop_HSO_HDB00]
     Started: [ hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm ]
 Master/Slave Set: msl_SAPHanaCon_HSO_HDB00 [rsc_SAPHanaCon_HSO_HDB00]
     Masters: [ hso-hana-vm-s1-0 ]
     Slaves: [ hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm ]
 Resource Group: g_ip_HSO_HDB00
     rsc_ip_HSO_HDB00   (ocf::heartbeat:IPaddr2):       Started hso-hana-vm-s1-0
     rsc_nc_HSO_HDB00   (ocf::heartbeat:anything):      Started hso-hana-vm-s1-0
</code></pre>

В Pacemaker есть важная функция — режим обслуживания. В этом режиме можно вносить изменения, не приводящие к немедленным действиям кластера. Пример — перезагрузка виртуальной машины. Типичный вариант использования — плановое обслуживание инфраструктуры Azure или ОС. См. раздел [Плановое техническое обслуживание](#planned-maintenance). Используйте следующую команду, чтобы переключить Pacemaker в режим обслуживания:

<pre><code>
crm configure property maintenance-mode=true
</code></pre>

При проверке с помощью команды **crm status** в выходных данных вы заметите, что все ресурсы отмечены как **unmanaged** (неуправляемые). В этом состоянии кластер не реагирует на любые изменения, такие как запуск или остановка SAP HANA.
Ниже приведен пример выходных данных команды **crm status**, когда кластер находится в режиме обслуживания:

<pre><code>
Stack: corosync
Current DC: hso-hana-dm (version 1.1.16-4.8-77ea74d) - partition with quorum
Last updated: Wed Sep 12 07:48:10 2018
Last change: Wed Sep 12 07:46:54 2018 by root via cibadmin on hso-hana-vm-s2-1

7 nodes configured
17 resources configured

              *** Resource management is DISABLED ***
  The cluster will not attempt to start, stop or recover services

Online: [ hso-hana-dm hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started hso-hana-dm (unmanaged)
 Clone Set: cln_SAPHanaTop_HSO_HDB00 [rsc_SAPHanaTop_HSO_HDB00] (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s1-1 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s1-0 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s1-2 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s2-1 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s2-2 (unmanaged)
     rsc_SAPHanaTop_HSO_HDB00   (ocf::suse:SAPHanaTopology):    Started hso-hana-vm-s2-0 (unmanaged)
     Stopped: [ hso-hana-dm ]
 Master/Slave Set: msl_SAPHanaCon_HSO_HDB00 [rsc_SAPHanaCon_HSO_HDB00] (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s1-1 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s1-2 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s2-1 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Slave hso-hana-vm-s2-2 (unmanaged)
     rsc_SAPHanaCon_HSO_HDB00   (ocf::suse:SAPHanaController):  Master hso-hana-vm-s2-0 (unmanaged)
     Stopped: [ hso-hana-dm hso-hana-vm-s1-0 ]
 Resource Group: g_ip_HSO_HDB00
     rsc_ip_HSO_HDB00   (ocf::heartbeat:IPaddr2):       Started hso-hana-vm-s2-0 (unmanaged)
     rsc_nc_HSO_HDB00   (ocf::heartbeat:anything):      Started hso-hana-vm-s2-0 (unmanaged)
</code></pre>


В этом примере команды показано, как выйти из режима обслуживания кластера:

<pre><code>
crm configure property maintenance-mode=false
</code></pre>


Другая команда **crm** получает полную конфигурацию кластера и открывает ее в редакторе, где вы можете ее изменить. После сохранения изменений кластер запускает соответствующие действия:

<pre><code>
crm configure edit
</code></pre>

Чтобы просмотреть полную конфигурацию кластера, используйте команду **crm show**:

<pre><code>
crm configure show
</code></pre>



После сбоев ресурсов кластера команда **crm status** может возвращать список **неудачных действий**. Вот пример таких выходных данных:


<pre><code>
Stack: corosync
Current DC: hso-hana-dm (version 1.1.16-4.8-77ea74d) - partition with quorum
Last updated: Thu Sep 13 07:30:44 2018
Last change: Thu Sep 13 07:30:20 2018 by root via crm_attribute on hso-hana-vm-s1-0

7 nodes configured
17 resources configured

Online: [ hso-hana-dm hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started hso-hana-dm
 Clone Set: cln_SAPHanaTop_HSO_HDB00 [rsc_SAPHanaTop_HSO_HDB00]
     Started: [ hso-hana-vm-s1-0 hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-0 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm ]
 Master/Slave Set: msl_SAPHanaCon_HSO_HDB00 [rsc_SAPHanaCon_HSO_HDB00]
     Masters: [ hso-hana-vm-s1-0 ]
     Slaves: [ hso-hana-vm-s1-1 hso-hana-vm-s1-2 hso-hana-vm-s2-1 hso-hana-vm-s2-2 ]
     Stopped: [ hso-hana-dm hso-hana-vm-s2-0 ]
 Resource Group: g_ip_HSO_HDB00
     rsc_ip_HSO_HDB00   (ocf::heartbeat:IPaddr2):       Started hso-hana-vm-s1-0
     rsc_nc_HSO_HDB00   (ocf::heartbeat:anything):      Started hso-hana-vm-s1-0

Failed Actions:
* rsc_SAPHanaCon_HSO_HDB00_monitor_60000 on hso-hana-vm-s2-0 'unknown error' (1): call=86, status=complete, exitreason='none',
    last-rc-change='Wed Sep 12 17:01:28 2018', queued=0ms, exec=277663ms
</code></pre>

После сбоев необходимо очищать кластер. Снова выполните команду **crm**, используя параметр **cleanup**, чтобы избавиться от записей неудачных действий. Назовите соответствующий кластерный ресурс так:

<pre><code>
crm resource cleanup rsc_SAPHanaCon_HSO_HDB00
</code></pre>

Команда должна возвращать примерно такой результат:

<pre><code>
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-dm
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s1-0
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s1-1
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s1-2
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s2-0
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s2-1
Cleaned up rsc_SAPHanaCon_HSO_HDB00:0 on hso-hana-vm-s2-2
Waiting for 7 replies from the CRMd....... OK
</code></pre>



## <a name="failover-or-takeover"></a>Отработка отказа или перехват

Как уже упоминалось в разделе [Важные примечания](#important-notes), для проверки отработки отказа кластера или перенаправления SAP HANA HSR не следует использовать стандартное завершение работы. Вместо этого рекомендуем вызвать тревогу ядра, выполнить принудительный перенос ресурсов или завершить работу всех сетей на уровне ОС виртуальной машины. Другой способ — команда **CRM \<node\> Standby** . См. [документ SUSE][sles-12-ha-paper]. 

Следующие три команды инициируют принудительную отработку отказа:

<pre><code>
echo c &gt /proc/sysrq-trigger

crm resource migrate msl_SAPHanaCon_HSO_HDB00 hso-hana-vm-s2-0 force

wicked ifdown eth0
wicked ifdown eth1
wicked ifdown eth2
......
wicked ifdown eth&ltn&gt
</code></pre>

Как упоминалось в разделе [Плановое техническое обслуживание](#planned-maintenance), хороший способ мониторинга действий кластера — выполнение команды **SAPHanaSR-showAttr** с параметром **watch**:

<pre><code>
watch SAPHanaSR-showAttr
</code></pre>

Кроме того, можно просмотреть состояние ландшафта SAP HANA, содержащееся в сценарии SAP Python. Конфигурация кластера ищет это значение состояния. Это становится очевидным в контексте отказа рабочего узла. Если рабочий узел завершает работу, SAP HANA не сразу возвращает ошибку работоспособности всей системы горизонтального масштабирования. 

Существует несколько повторных попыток, чтобы избежать ненужных отработок отказа. Кластер реагирует только при изменении состояния с **Ok** (возвращаемое значение **4**) на **error** (возвращаемое значение **1**). Таким образом, если в выходных данных команды **SAPHanaSR-showAttr** отображается виртуальная машина с состоянием **offline**, то все в порядке. Однако пока что нет действий для переключения между основным и второстепенным ресурсами. Действия кластера не запускаются, пока SAP HANA не возвращает ошибку.

Вы можете отслеживать состояние работоспособности SAP HANA ландшафт в качестве пользователя **\<HANA SID\> ADM** , вызвав сценарий SAP Python следующим образом. Возможно, потребуется адаптировать путь:

<pre><code>
watch python /hana/shared/HSO/exe/linuxx86_64/HDB_2.00.032.00.1533114046_eeaf4723ec52ed3935ae0dc9769c9411ed73fec5/python_support/landscapeHostConfiguration.py
</code></pre>

Выходные данные этой команды должны выглядеть, как в приведенном ниже примере. Нас интересуют столбец **Host Status** (Состояние узла) и **общее состояние узла**. В фактических выходных данных есть дополнительные столбцы.
Чтобы сделать таблицу выходных данных более читаемой в этом документе, большинство столбцов с правой стороны были обрезаны:

<pre><code>
| Host             | Host   | Host   | Failover | Remove | 
|                  | Active | Status | Status   | Status | 
|                  |        |        |          |        | 
| ---------------- | ------ | ------ | -------- | ------ |    .......
| hso-hana-vm-s2-0 | yes    | ok     |          |        |        
| hso-hana-vm-s2-1 | yes    | ok     |          |        |         
| hso-hana-vm-s2-2 | yes    | ok     |          |        |        

overall host status: ok
</code></pre>


Существует еще одна команда для проверки текущих действий кластера. Ниже приведены команда и окончание выходных данных после того, как главный узел основного сайта был остановлен. Вы можете просмотреть список действий перехода, например **повышение** прежнего дополнительного главного узла (**hso-hana-vm-s2-0**) до нового основного главного узла. Если все в порядке и все действия завершены, список **Transition Summary** (Сводные данные перехода) должен быть пустым.

<pre><code>
 crm_simulate -Ls

...........

Transition Summary:
 * Fence hso-hana-vm-s1-0
 * Stop    rsc_SAPHanaTop_HSO_HDB00:1   (hso-hana-vm-s1-0)
 * Demote  rsc_SAPHanaCon_HSO_HDB00:1   (Master -> Stopped hso-hana-vm-s1-0)
 * Promote rsc_SAPHanaCon_HSO_HDB00:5   (Slave -> Master hso-hana-vm-s2-0)
 * Move    rsc_ip_HSO_HDB00     (Started hso-hana-vm-s1-0 -> hso-hana-vm-s2-0)
 * Move    rsc_nc_HSO_HDB00     (Started hso-hana-vm-s1-0 -> hso-hana-vm-s2-0)
</code></pre>



## <a name="planned-maintenance"></a>Плановое техническое обслуживание 

Когда речь идет о плановом обслуживании, существуют различные варианты использования. Важно отличать простое обслуживание инфраструктуры, например изменения на уровне ОС и настройку диска, от обновления HANA.
Дополнительные сведения можно найти в документах от SUSE, например [Обеспечение работы без простоев][sles-zero-downtime-paper] и [Сценарий SAP HANA SR, оптимизированный для высокой производительности][sles-12-for-sap]. В этих документах также приводятся примеры того, как вручную перенести основной сайт.

Для проверки варианта использования с обслуживанием инфраструктуры было проведено интенсивное внутреннее тестирование. Чтобы избежать каких-либо проблем, связанных с миграцией основного сайта, мы решили всегда переносить основной сайт, прежде чем переключать кластер в режим обслуживания. Таким образом, нет необходимости заставлять кластер забывать о прежней ситуации (какая сторона была основной, а какая — дополнительной).

В этом отношении есть две разные ситуации:

- **Плановое обслуживание текущего дополнительного сайта**. В этом случае вы можете просто переключить кластер в режим обслуживания и выполнить работу на дополнительном сайте, не затрагивая кластер.

- **Плановое обслуживание текущего основного сайта**. Чтобы пользователи могли продолжать работать во время обслуживания, необходима принудительная отработка отказа. При таком подходе вам необходимо инициировать отработку отказа кластера с помощью Pacemaker, а не только на уровне SAP HANA HSR. При настройке Pacemaker автоматически запускается перехват SAP HANA. Вам также необходимо выполнить отработку отказа, прежде чем переводить кластер в режим обслуживания.

Ниже описана процедура обслуживания текущего дополнительного сайта.

1. Переключите кластер в режим обслуживания.
2. Выполните работу на дополнительном сайте. 
3. Отключите режим обслуживания кластера.

Процедура обслуживания текущего основного сайта более сложна:

1. Активируйте отработку отказа или перенаправление SAP HANA вручную с помощью миграции ресурсов Pacemaker. Ознакомьтесь с приведенными ниже подробностями.
2. SAP HANA на прежнем основном сайте отключается во время настройки кластера.
3. Переключите кластер в режим обслуживания.
4. После завершения обслуживания зарегистрируйте прежний основной сайт как новый дополнительный сайт.
5. Очистите конфигурацию кластера. Ознакомьтесь с приведенными ниже подробностями.
6. Отключите режим обслуживания кластера.


Во время переноса ресурса в конфигурацию кластера добавляется запись. Пример — принудительная отработка отказа. Эти записи нужно очистить до выхода из режима обслуживания. Ознакомьтесь с приведенным ниже примером.

Для начала инициируйте принудительное восстановление кластера путем переноса ресурса **msl** на текущий дополнительный главный узел. В этой команде выдается предупреждение о создании **ограничения перемещения**.

<pre><code>
crm resource migrate msl_SAPHanaCon_HSO_HDB00 force

INFO: Move constraint created for msl_SAPHanaCon_HSO_HDB00
</code></pre>


Проверьте процесс отработки отказа с помощью команды **SAPHanaSR-showAttr**. Чтобы отслеживать состояние кластера, откройте окно выделенной оболочки и запустите команду **watch**:

<pre><code>
watch SAPHanaSR-showAttr
</code></pre>

В выходных данных должно быть указано, что был выполнен переход на другой ресурс вручную. Предыдущий дополнительный главный узел был **повышен** (в данном примере — **hso-hana-vm-s2-0**). Бывший основной сайт был остановлен (параметр **lss** со значением **1** для бывшего основного главного узла **hso-hana-vm-s1-0**): 

<pre><code>
Global cib-time                 prim  sec srHook sync_state
------------------------------------------------------------
global Wed Sep 12 07:40:02 2018 HSOS2 -   SFAIL  SFAIL


Sites lpt        lss mns              srr
------------------------------------------
HSOS1 10         1   hso-hana-vm-s1-0 P
HSOS2 1536738002 4   hso-hana-vm-s2-0 P


Hosts            clone_state node_state roles                        score  site
----------------------------------------------------------------------------------
hso-hana-dm                  online
hso-hana-vm-s1-0 UNDEFINED   online     master1::worker:             150    HSOS1
hso-hana-vm-s1-1 DEMOTED     online     slave::worker:               -10000 HSOS1
hso-hana-vm-s1-2 DEMOTED     online     slave::worker:               -10000 HSOS1
hso-hana-vm-s2-0 PROMOTED    online     master1:master:worker:master 150    HSOS2
hso-hana-vm-s2-1 DEMOTED     online     slave:slave:worker:slave     -10000 HSOS2
hso-hana-vm-s2-2 DEMOTED     online     slave:slave:worker:slave     -10000 HSOS2
</code></pre>

После отработки отказа кластера и перенаправления SAP HANA кластер переходит в режим обслуживания, как описано в разделе [Pacemaker](#pacemaker).

Команды **SAPHanaSR-showAttr** и **crm status** ничего не сообщают об ограничениях, создаваемых миграцией ресурсов. Один из способов сделать эти ограничения видимыми — показать полную конфигурацию кластерного ресурса с помощью следующей команды:

<pre><code>
crm configure show
</code></pre>

В конфигурации кластера вы обнаружите новое ограничение расположения, вызванное предыдущей миграцией ресурсов вручную. Этот пример записи начинается со строки **location cli-**:

<pre><code>
location cli-ban-msl_SAPHanaCon_HSO_HDB00-on-hso-hana-vm-s1-0 msl_SAPHanaCon_HSO_HDB00 role=Started -inf: hso-hana-vm-s1-0
</code></pre>

К сожалению, такие ограничения могут повлиять на общее поведение кластера. Поэтому их необходимо снова удалить, прежде чем создавать резервную копию системы. С помощью команды **unmigrate** можно удалить ограничения расположения, которые были созданы ранее. Именование может быть немного запутанным. При этом не выполняется попытка перенести ресурс обратно на виртуальную машину, с которой он был перенесен. Команда просто удаляет ограничения расположения, а также возвращает соответствующую информацию:


<pre><code>
crm resource unmigrate msl_SAPHanaCon_HSO_HDB00

INFO: Removed migration constraints for msl_SAPHanaCon_HSO_HDB00
</code></pre>

По завершении обслуживания остановите режим обслуживания кластера, как показано в разделе [Pacemaker](#pacemaker).



## <a name="hb_report-to-collect-log-files"></a>hb_report для получения файлов журнала

Чтобы проанализировать проблемы кластера Pacemaker, полезно запустить служебную программу **hb_report** (это также запрашивает служба поддержки SUSE). Она собирает все важные файлы журналов, необходимые для анализа происходящего. В этом примере вызова используется время начала и конца того или иного инцидента. Кроме того, см. раздел [Важные примечания](#important-notes):

<pre><code>
hb_report -f "2018/09/13 07:36" -t "2018/09/13 08:00" /tmp/hb_report_log
</code></pre>

Команда указывает, куда помещаются сжатые файлы журнала:

<pre><code>
The report is saved in /tmp/hb_report_log.tar.bz2
Report timespan: 09/13/18 07:36:00 - 09/13/18 08:00:00
</code></pre>

Затем вы можете извлечь отдельные файлы с помощью стандартной команды **tar**:

<pre><code>
tar -xvf hb_report_log.tar.bz2
</code></pre>

В извлеченных файлах вы найдете все файлы журнала. Большинство из них были помещены в отдельные каталоги для каждого узла кластера:

<pre><code>
-rw-r--r-- 1 root root  13655 Sep 13 09:01 analysis.txt
-rw-r--r-- 1 root root  14422 Sep 13 09:01 description.txt
-rw-r--r-- 1 root root      0 Sep 13 09:01 events.txt
-rw-r--r-- 1 root root 275560 Sep 13 09:00 ha-log.txt
-rw-r--r-- 1 root root     26 Sep 13 09:00 ha-log.txt.info
drwxr-xr-x 4 root root   4096 Sep 13 09:01 hso-hana-dm
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s1-0
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s1-1
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s1-2
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s2-0
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s2-1
drwxr-xr-x 3 root root   4096 Sep 13 09:01 hso-hana-vm-s2-2
-rw-r--r-- 1 root root 264726 Sep 13 09:00 journal.log
</code></pre>


В пределах указанного временного диапазона текущий главный узел **hso-hana-vm-s1-0** был остановлен. Записи, связанные с этим событием, можно найти в файле **journal.log**:

<pre><code>
2018-09-13T07:38:01+0000 hso-hana-vm-s2-1 su[93494]: (to hsoadm) root on none
2018-09-13T07:38:01+0000 hso-hana-vm-s2-1 su[93494]: pam_unix(su-l:session): session opened for user hsoadm by (uid=0)
2018-09-13T07:38:01+0000 hso-hana-vm-s2-1 systemd[1]: Started Session c44290 of user hsoadm.
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [TOTEM ] A new membership (10.0.0.13:120996) was formed. Members left: 1
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [TOTEM ] Failed to receive the leave message. failed: 1
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 attrd[28313]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 attrd[28313]:   notice: Removing all hso-hana-vm-s1-0 attributes for peer loss
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 attrd[28313]:   notice: Purged 1 peer with id=1 and/or uname=hso-hana-vm-s1-0 from the membership cache
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 stonith-ng[28311]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 stonith-ng[28311]:   notice: Purged 1 peer with id=1 and/or uname=hso-hana-vm-s1-0 from the membership cache
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 cib[28310]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [QUORUM] Members[6]: 7 2 3 4 5 6
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 corosync[28302]:   [MAIN  ] Completed service synchronization, ready to provide service.
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 crmd[28315]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 pacemakerd[28308]:   notice: Node hso-hana-vm-s1-0 state is now lost
2018-09-13T07:38:02+0000 hso-hana-vm-s2-1 cib[28310]:   notice: Purged 1 peer with id=1 and/or uname=hso-hana-vm-s1-0 from the membership cache
2018-09-13T07:38:03+0000 hso-hana-vm-s2-1 su[93494]: pam_unix(su-l:session): session closed for user hsoadm
</code></pre>

Другим примером является файл журнала Pacemaker на дополнительном главном сайте, который стал новым основным главным сайтом. В этом фрагменте показано, что для остановленного основного главного узла задано состояние **offline** (не в сети).

<pre><code>
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 3 still member of group stonith-ng (peer=hso-hana-vm-s1-2, counter=5.1)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 4 still member of group stonith-ng (peer=hso-hana-vm-s2-0, counter=5.2)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 5 still member of group stonith-ng (peer=hso-hana-vm-s2-1, counter=5.3)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 6 still member of group stonith-ng (peer=hso-hana-vm-s2-2, counter=5.4)
Sep 13 07:38:02 [4178] hso-hana-vm-s2-0 stonith-ng:     info: pcmk_cpg_membership:      Node 7 still member of group stonith-ng (peer=hso-hana-dm, counter=5.5)
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: pcmk_cpg_membership:      Node 1 left group crmd (peer=hso-hana-vm-s1-0, counter=5.0)
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: crm_update_peer_proc:     pcmk_cpg_membership: Node hso-hana-vm-s1-0[1] - corosync-cpg is now offline
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: peer_update_callback:     Client hso-hana-vm-s1-0/peer now has status [offline] (DC=hso-hana-dm, changed=4000000)
Sep 13 07:38:02 [4184] hso-hana-vm-s2-0       crmd:     info: pcmk_cpg_membership:      Node 2 still member of group crmd (peer=hso-hana-vm-s1-1, counter=5.0)
</code></pre>





## <a name="sap-hana-globalini"></a>SAP HANA global.ini


Приведенные ниже фрагменты взяты из файла **global.ini** для SAP HANA на сайте кластера 2. В этом примере показаны записи разрешения имен узлов при использовании различных сетей для HSR и взаимодействия между узлами SAP HANA:

<pre><code>
[communication]
tcp_keepalive_interval = 20
internal_network = 10.0.2/24
listeninterface = .internal
</code></pre>


<pre><code>
[internal_hostname_resolution]
10.0.2.40 = hso-hana-vm-s2-0
10.0.2.42 = hso-hana-vm-s2-2
10.0.2.41 = hso-hana-vm-s2-1
</code></pre>

<pre><code>
[ha_dr_provider_SAPHanaSR]
provider = SAPHanaSR
path = /hana/shared/myHooks
execution_order = 1
</code></pre>

<pre><code>
[system_replication_communication]
listeninterface = .internal

[system_replication_hostname_resolution]
10.0.1.30 = hso-hana-vm-s1-0
10.0.1.31 = hso-hana-vm-s1-1
10.0.1.32 = hso-hana-vm-s1-2
10.0.1.40 = hso-hana-vm-s2-0
10.0.1.41 = hso-hana-vm-s2-1
10.0.1.42 = hso-hana-vm-s2-2
</code></pre>



## <a name="hawk"></a>Hawk

Решение кластера предоставляет графический интерфейс браузера для тех пользователей, которые предпочитают меню и графику вместо команд на уровне оболочки.
Чтобы использовать интерфейс браузера, замените на **\<node\>** фактический узел SAP HANA в следующем URL-адресе. Затем введите учетные данные для кластера (пользователь **cluster**):

<pre><code>
https://&ltnode&gt:7630
</code></pre>

На этом снимке экрана показана панель мониторинга кластера:


![Панель мониторинга кластера Hawk](media/hana-vm-scale-out-HA-troubleshooting/hawk-1.png)


В этом примере показаны ограничения расположения, вызванные переносом кластерного ресурса, как описано в разделе [Плановое техническое обслуживание](#planned-maintenance):


![Ограничения списков Hawk](media/hana-vm-scale-out-HA-troubleshooting/hawk-2.png)


Вы также можете отправить выходные данные **hb_report** в Hawk в разделе **History** (Журнал), как показано ниже. См. раздел hb_report для сбора файлов журналов: 

![Выходные данные hb_report передачи Hawk](media/hana-vm-scale-out-HA-troubleshooting/hawk-3.png)

Затем с помощью **обозревателя журнала** можно просмотреть все переходы между кластерами, включенные в выходные данные **hb_report**:

![Переходы Hawk в выходных данных hb_report](media/hana-vm-scale-out-HA-troubleshooting/hawk-4.png)

На последнем снимке экрана показан раздел **Details** одного перехода. Кластер отреагировал на отказ основного главного узла — **hso-hana-vm-s1-0**. Теперь дополнительный узел становится главным — **hso-hana-vm-s2-0**:

![Один переход Hawk](media/hana-vm-scale-out-HA-troubleshooting/hawk-5.png)


## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве по устранению неполадок описываются вопросы высокой доступности для SAP HANA в конфигурации горизонтального масштабирования. Еще один важный компонент ландшафта SAP, помимо базы данных, — стек SAP NetWeaver. Узнайте о [высоком уровне доступности для SAP NetWeaver на виртуальных машинах Azure, использующих SUSE Enterprise Linux Server][sap-nw-ha-guide-sles].
