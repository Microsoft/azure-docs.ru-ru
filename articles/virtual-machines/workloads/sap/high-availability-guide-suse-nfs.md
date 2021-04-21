---
title: Высокий уровень доступности для NFS на виртуальных машинах Azure на платформе SLES | Документация Майкрософт
description: Обеспечение высокого уровня доступности NFS на виртуальных машинах Azure в SUSE Linux Enterprise Server
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
ms.date: 10/16/2020
ms.author: radeltch
ms.openlocfilehash: 993baa521530ffa6a702f8324a1691850687c366
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101668696"
---
# <a name="high-availability-for-nfs-on-azure-vms-on-suse-linux-enterprise-server"></a>Обеспечение высокого уровня доступности NFS на виртуальных машинах Azure в SUSE Linux Enterprise Server

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[2205917]:https://launchpad.support.sap.com/#/notes/2205917
[1944799]:https://launchpad.support.sap.com/#/notes/1944799
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[1410736]:https://launchpad.support.sap.com/#/notes/1410736

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[sles-hae-guides]:https://www.suse.com/documentation/sle-ha-12/
[sles-for-sap-bp]:https://www.suse.com/documentation/sles-for-sap-12/
[suse-ha-12sp3-relnotes]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP3/

[template-multisid-xscs]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[template-converged]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-converged-md%2Fazuredeploy.json
[template-file-server]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-file-server-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability.md

В этой статье описывается развертывание виртуальных машин, их настройка, установка платформы кластера, а также установка высокодоступного NFS-сервера, который можно использовать для хранения общих данных высокодоступной системы SAP.
В этом руководстве описывается настройка высокодоступного NFS-сервера, который используется двумя системами SAP, NW1 и NW2. Имена ресурсов (например, виртуальных машин или виртуальных сетей) в примере предполагают, что вы использовали [шаблон файлового сервера SAP][template-file-server] с префиксом ресурсов **prod**.


> [!NOTE]
> В этой статье упоминаются термины *slave (ведомый)* и *master (главный)* , которые больше не используются Майкрософт. Когда эти термины будут удалены из программных продуктов, мы удалим их и из этой статьи.

Сначала прочитайте следующие примечания и документы SAP:

* примечание к SAP [1928533], которое содержит:
  * список размеров виртуальных машин Azure, поддерживаемых для развертывания ПО SAP;
  * важные сведения о доступных ресурсах для каждого размера виртуальной машины Azure;
  * сведения о поддерживаемом программном обеспечении SAP и сочетаниях операционных систем и баз данных;
  * сведения о требуемой версии ядра SAP для Windows и Linux в Microsoft Azure.

* примечание к SAP [2015553], в котором описываются предварительные требования к SAP при развертывании программного обеспечения SAP в Azure;
* Примечание SAP [2205917] содержит рекомендуемые параметры ОС для SUSE Linux Enterprise Server для приложений SAP.
* Примечание SAP [1944799] содержит рекомендации для SAP HANA в SUSE Linux Enterprise Server для приложений SAP.
* примечание к SAP [2178632], содержащее подробные сведения обо всех доступных метриках мониторинга для SAP в Azure;
* примечание к SAP [2191498], содержащее сведения о необходимой версии агента SAP Host Agent для Linux в Azure;
* примечание к SAP [2243692], содержащее сведения о лицензировании SAP в Linux в Azure;
* примечание к SAP [1984787], содержащее общие сведения о SUSE Linux Enterprise Server 12;
* примечание к SAP [1999351], содержащее дополнительные сведения об устранении неполадок, связанных с расширением для расширенного мониторинга Azure для SAP;
* [вики-сайт сообщества SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes), содержащий все необходимые примечания к SAP для Linux;
* [SAP NetWeaver на виртуальных машинах Windows. Руководство по планированию и внедрению][planning-guide];
* [Развертывание программного обеспечения SAP на виртуальных машинах Linux в Azure][deployment-guide] (эта статья);
* [SAP NetWeaver на виртуальных машинах Windows. Руководство по развертыванию СУБД][dbms-guide];
* [Рекомендации по SUSE Linux Enterprise Server for SAP Applications версии 12 с пакетом обновления 3 (SP3)][sles-hae-guides]
  * Высокодоступное хранилище NFS с DRBD и Pacemaker
* [рекомендации по SUSE Linux Enterprise Server for SAP Applications версии 12 с пакетом обновления 3 (SP3)][sles-for-sap-bp]:
* [Заметки о выпуске расширения SUSE высокого уровня доступности 12 SP3][suse-ha-12sp3-relnotes]

## <a name="overview"></a>Обзор

Чтобы добиться высокого уровня доступности, SAP NetWeaver необходим NFS-сервер. NFS-сервер настраивается в отдельном кластере и может использоваться несколькими системами SAP.

![Общие сведения о высоком уровне доступности SAP NetWeaver](./media/high-availability-guide-nfs/ha-suse-nfs.png)

NFS-сервер использует выделенное виртуальное имя узла и виртуальные IP-адреса для каждой системы SAP, использующей этот NFS-сервер. Балансировщику нагрузки в Azure нужен виртуальный IP-адрес. Ниже показана конфигурация балансировщика нагрузки.        

* Конфигурация внешнего интерфейса:
  * IP-адрес 10.0.0.4 для NW1.
  * IP-адрес 10.0.0.5 для NW2.
* Конфигурация серверной части:
  * подключена к основным сетевым интерфейсам всех виртуальных машин, которые должны быть частью кластера NFS.
* Порт пробы:
  * Порт 61000 для NW1.
  * Порт 61001 для NW2.
* Правила балансировки нагрузки (если используется базовая подсистема балансировки нагрузки)
  * TCP-порт 2049 для NW1.
  * UDP-порт 2049 для NW1.
  * TCP-порт 2049 для NW2.
  * UDP-порт 2049 для NW2.

## <a name="set-up-a-highly-available-nfs-server"></a>Настройка высокодоступного NFS-сервера

Вы можете использовать шаблон Azure с сайта GitHub, чтобы развернуть все необходимые ресурсы Azure, включая виртуальные машины, набор доступности и подсистему балансировки нагрузки. Эти ресурсы также можно развернуть вручную.

### <a name="deploy-linux-via-azure-template"></a>Развертывание Linux с помощью шаблон Azure

В Azure Marketplace доступен образ для SUSE Linux Enterprise Server для приложения SAP 12, который можно использовать для развертывания новых виртуальных машин.
Все необходимые ресурсы можно развернуть с помощью шаблонов быстрого запуска с сайта GitHub. Этот шаблон развертывает виртуальные машины, подсистему балансировки нагрузки, группу доступности и т. д. Чтобы развернуть шаблон, выполните следующие действия.

1. Откройте [шаблон файлового сервера SAP][template-file-server] на портале Azure.   
1. Задайте следующие параметры.
   1. Префикс ресурса  
      Введите префикс, который вы хотите использовать. Значение будет использоваться в качестве префикса для развертываемых ресурсов.
   2. Количество систем SAP  
      Введите количество систем SAP, которые будут использовать этот файловый сервер. Будет развернуто необходимое число интерфейсных конфигураций, правил балансировки нагрузки, портов пробы, дисков и т. д.
   3. Тип ОС  
      Выберите один из дистрибутивов Linux. Для этого примера выберите SLES 12.
   4. Имя пользователя и пароль администратора  
      Создается учетная запись пользователя, которую можно использовать для входа на компьютер.
   5. Идентификатор подсети  
      Чтобы развернуть виртуальную машину в имеющейся виртуальной сети с определенной подсетью, необходимо указать идентификатор этой определенной подсети. Идентификатор обычно выглядит так: /subscriptions/ **&lt;идентификатор_подписки&gt;** /resourceGroups/ **&lt;имя_группы_ресурсов&gt;** /providers/Microsoft.Network/virtualNetworks/ **&lt;имя_виртуальной_сети&gt;** /subnets/ **&lt;имя_подсети&gt;** .

### <a name="deploy-linux-manually-via-azure-portal"></a>Развертывание Linux вручную с помощью портала Azure

Сначала необходимо создать виртуальные машины для этого кластера NFS. После этого следует создать подсистему балансировки нагрузки и использовать виртуальные машины во внутренних пулах.

1. Создание группы ресурсов
1. Создайте виртуальную сеть
1. Создание группы доступности.  
   Настройка максимального числа доменов обновления.
1. Создание виртуальной машины 1. Используйте по крайней мере SLES4SAP 12 с пакетом обновления 3. В этом примере используется образ SLES4SAP 12 с пакетом обновления 3 для SLES For SAP Applications 12 с пакетом обновления 3 (BYOS).  
   Выберите ранее созданную группу доступности.  
1. Создание виртуальной машины 2. Используйте по крайней мере SLES4SAP 12 с пакетом обновления 3. В этом примере используется образ SLES4SAP 12 с пакетом обновления 3 (BYOS).  
   В этом примере используется SLES For SAP Applications 12 с пакетом обновления 3 (BYOS).  
   Выберите ранее созданную группу доступности.  
1. Добавьте один диск данных для каждой системы SAP в обе виртуальные машины.
1. Создайте подсистему балансировки нагрузки (внутреннюю). Мы рекомендуем [подсистему балансировки нагрузки ценовой категории "Стандартный"](../../../load-balancer/load-balancer-overview.md).  
   1. Чтобы создать стандартную подсистему балансировки нагрузки, выполните следующие инструкции:
      1. Создайте IP-адреса внешнего интерфейса.
         1. IP-адрес 10.0.0.4 для NW1.
            1. Откройте подсистему балансировки нагрузки, выберите пул внешних IP-адресов и щелкните "Добавить".
            1. Введите имя нового внешнего пула IP-адресов (например, **nw1-frontend**).
            1. Выберите для параметра "Назначение" значение "Статическое" и введите IP-адрес (например, **10.0.0.4**).
            1. Нажмите кнопку "ОК".
         1. IP-адрес 10.0.0.5 для NW2.
            * Повторите предыдущие шаги для порта NW2.
      1. Создайте внутренние пулы.
         1. подключена к основным сетевым интерфейсам всех виртуальных машин, которые должны быть частью кластера NFS.
            1. Выберите подсистему балансировки нагрузки, щелкните "Серверные пулы" и нажмите кнопку "Добавить".
            1. Введите имя нового внутреннего пула (например, **nw-backend**)
            1. Выбор виртуальной сети
            1. Щелкните "Добавить виртуальную машину".
            1. Выберите виртуальные машины кластера NFS и соответствующие им IP-адреса.
            1. Нажмите кнопку "Добавить".
      1. Создайте пробы работоспособности.
         1. Порт 61000 для NW1.
            1. Выберите подсистему балансировки нагрузки, щелкните "Зонды работоспособности" и нажмите кнопку "Добавить".
            1. Введите имя новой пробы работоспособности (например, **nw1-hp**).
            1. Выберите протокол TCP, порт 610 **00**, интервал, равный 5, и порог состояния неработоспособности, равный 2.
            1. Нажмите кнопку "ОК".
         1. Порт 61001 для NW2.
            * Повторите предыдущие действия, чтобы создать пробу работоспособности для NW2.
      1. Правила балансировки нагрузки
         1. Откройте подсистему балансировки нагрузки, щелкните "Правила балансировки нагрузки" и нажмите кнопку "Добавить"
         1. Введите имя нового правила для подсистемы балансировки нагрузки (например, **nw1-lb**)
         1. Выберите внешний IP-адрес, внутренний пул и пробу работоспособности, созданные ранее (например, **nw1-frontend**. **nw-backend** и **nw1-hp**)
         1. Выберите **Порты высокой доступности**.
         1. Увеличьте время ожидания до 30 минут.
         1. **Не забудьте включить плавающий IP-адрес**.
         1. Нажмите кнопку "ОК"
         * Повторите описанные выше действия, чтобы создать правило балансировки нагрузки для NW2
   1. Если же в сценарии требуется базовая подсистема балансировки нагрузки, выполните следующие инструкции:
      1. Создайте IP-адреса внешнего интерфейса.
         1. IP-адрес 10.0.0.4 для NW1.
            1. Откройте подсистему балансировки нагрузки, выберите пул внешних IP-адресов и щелкните "Добавить".
            1. Введите имя нового внешнего пула IP-адресов (например, **nw1-frontend**).
            1. Выберите для параметра "Назначение" значение "Статическое" и введите IP-адрес (например, **10.0.0.4**).
            1. Нажмите кнопку "ОК".
         1. IP-адрес 10.0.0.5 для NW2.
            * Повторите предыдущие шаги для порта NW2.
      1. Создайте внутренние пулы.
         1. подключена к основным сетевым интерфейсам всех виртуальных машин, которые должны быть частью кластера NFS.
            1. Выберите подсистему балансировки нагрузки, щелкните "Серверные пулы" и нажмите кнопку "Добавить".
            1. Введите имя нового внутреннего пула (например, **nw-backend**)
            1. Щелкните "Добавить виртуальную машину".
            1. Выберите ранее созданную группу доступности.
            1. Выберите виртуальные машины кластера NFS.
            1. Нажмите кнопку "ОК"
      1. Создайте пробы работоспособности.
         1. Порт 61000 для NW1.
            1. Выберите подсистему балансировки нагрузки, щелкните "Зонды работоспособности" и нажмите кнопку "Добавить".
            1. Введите имя новой пробы работоспособности (например, **nw1-hp**).
            1. Выберите протокол TCP, порт 610 **00**, интервал, равный 5, и порог состояния неработоспособности, равный 2.
            1. Нажмите кнопку "ОК".
         1. Порт 61001 для NW2.
            * Повторите предыдущие действия, чтобы создать пробу работоспособности для NW2.
      1. Правила балансировки нагрузки
         1. TCP-порт 2049 для NW1.
            1. Выберите подсистему балансировки нагрузки, щелкните "Правила балансировки нагрузки" и нажмите кнопку "Добавить".
            1. Введите имя нового правила балансировки нагрузки (например, **nw1-lb-2049**).
            1. Выберите внешний IP-адрес, внутренний пул и пробу работоспособности, созданные ранее (например, **nw1-frontend**).
            1. Оставьте выбранным протокол **TCP** и введите порт **2049**.
            1. Увеличьте время ожидания до 30 минут.
            1. **Не забудьте включить плавающий IP-адрес**.
            1. Нажмите кнопку "ОК"
         1. UDP-порт 2049 для NW1.
            * Повторите предыдущие шаги, указав UDP-порт 2049 для NW1.
         1. TCP-порт 2049 для NW2.
            * Повторите предыдущие шаги, указав TCP-порт 2049 для NW2.
         1. UDP-порт 2049 для NW2.
            * Повторите предыдущие шаги, указав UDP-порт 2049 для NW2.

> [!IMPORTANT]
> Плавающий IP-адрес не поддерживается для вторичных IP-конфигураций сетевых карт в сценариях с балансировкой нагрузки. Подробности см. в статье [Ограничения подсистемы балансировки нагрузки Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Если для виртуальной машины требуется дополнительный IP-адрес, следует развернуть вторую сетевую карту.  

> [!Note]
> Если в серверный пул внутреннего (без общедоступного IP-адреса) Azure Load Balancer ценовой категории "Стандартный" помещаются виртуальные машины без общедоступных IP-адресов, у них не будет исходящего подключения к Интернету без дополнительной настройки, разрешающей маршрутизацию к общедоступным конечным точкам. Подробные сведения о такой настройке см. в статье [Подключение к общедоступной конечной точке для виртуальных машин с помощью Azure Load Balancer (цен. категория "Стандартный") в сценариях обеспечения высокого уровня доступности SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

> [!IMPORTANT]
> Не включайте метки времени TCP на виртуальных машинах Azure, размещенных за Azure Load Balancer. Включение меток времени TCP помешает работе проб работоспособности. Установите для параметра **net.ipv4.tcp_timestamps** значение **0**. Дополнительные сведения см. в статье [Пробы работоспособности Load Balancer](../../../load-balancer/load-balancer-custom-probe-overview.md).

### <a name="create-pacemaker-cluster"></a>Создание кластера Pacemaker

Следуйте указаниям в статье [Настройка кластера Pacemaker в SUSE Linux Enterprise Server в Azure](high-availability-guide-suse-pacemaker.md), чтобы создать базовый кластер Pacemaker для этого NFS-сервера.

### <a name="configure-nfs-server"></a>Настройка NFS-сервера

Ниже приведены элементы с префиксами: **[A]**  — применяется ко всем узлам, **[1**] — применяется только к узлу 1, **[2]**  — применяется только к узлу 2.

1. **[A]** Установите разрешения имен.

   Можно использовать DNS-сервер или внести изменения в файл /etc/hosts на всех узлах. В этом примере показано, как использовать файл /etc/hosts.
   Замените IP-адрес и имя узла в следующих командах.

   <pre><code>sudo vi /etc/hosts
   </code></pre>
   
   Вставьте следующие строки в /etc/hosts. Измените IP-адрес и имя узла в соответствии со своей средой.
   
   <pre><code># IP address of the load balancer frontend configuration for NFS
   <b>10.0.0.4 nw1-nfs</b>
   <b>10.0.0.5 nw2-nfs</b>
   </code></pre>

1. **[A]** Включите NFS-сервер.

   Создайте корневую запись экспорта NFS.

   <pre><code>sudo sh -c 'echo /srv/nfs/ *\(rw,no_root_squash,fsid=0\)>/etc/exports'
   
   sudo mkdir /srv/nfs/
   </code></pre>

1. **[A]** Установите компоненты DRBD.

   <pre><code>sudo zypper install drbd drbd-kmp-default drbd-utils
   </code></pre>

1. **[A]** Создайте секцию для устройств DRBD.

   Выведите список всех доступных дисков данных.

   <pre><code>sudo ls /dev/disk/azure/scsi1/
   </code></pre>

   Пример выходных данных
   
   ```
   lun0  lun1
   ```

   Создайте секции для всех дисков данных.

   <pre><code>sudo sh -c 'echo -e "n\n\n\n\n\nw\n" | fdisk /dev/disk/azure/scsi1/lun0'
   sudo sh -c 'echo -e "n\n\n\n\n\nw\n" | fdisk /dev/disk/azure/scsi1/lun1'
   </code></pre>

1. **[A]** Создайте конфигурации LVM.

   Выведите список всех доступных секций.

   <pre><code>ls /dev/disk/azure/scsi1/lun*-part*
   </code></pre>

   Пример выходных данных
   
   ```
   /dev/disk/azure/scsi1/lun0-part1  /dev/disk/azure/scsi1/lun1-part1
   ```

   Создайте тома LVM для всех секций.

   <pre><code>sudo pvcreate /dev/disk/azure/scsi1/lun0-part1  
   sudo vgcreate vg-<b>NW1</b>-NFS /dev/disk/azure/scsi1/lun0-part1
   sudo lvcreate -l 100%FREE -n <b>NW1</b> vg-<b>NW1</b>-NFS

   sudo pvcreate /dev/disk/azure/scsi1/lun1-part1
   sudo vgcreate vg-<b>NW2</b>-NFS /dev/disk/azure/scsi1/lun1-part1
   sudo lvcreate -l 100%FREE -n <b>NW2</b> vg-<b>NW2</b>-NFS
   </code></pre>

1. **[A]** Настройте DRBD.

   <pre><code>sudo vi /etc/drbd.conf
   </code></pre>

   Убедитесь, что файл drbd.conf содержит приведенные ниже две строки.

   <pre><code>include "drbd.d/global_common.conf";
   include "drbd.d/*.res";
   </code></pre>

   Измените глобальную конфигурацию DRBD.

   <pre><code>sudo vi /etc/drbd.d/global_common.conf
   </code></pre>

   Добавьте следующие записи в разделы handlers и net.

   <pre><code>global {
        usage-count no;
   }
   common {
        handlers {
             fence-peer "/usr/lib/drbd/crm-fence-peer.sh";
             after-resync-target "/usr/lib/drbd/crm-unfence-peer.sh";
             split-brain "/usr/lib/drbd/notify-split-brain.sh root";
             pri-lost-after-sb "/usr/lib/drbd/notify-pri-lost-after-sb.sh; /usr/lib/drbd/notify-emergency-reboot.sh; echo b > /proc/sysrq-trigger ; reboot -f";
        }
        startup {
             wfc-timeout 0;
        }
        options {
        }
        disk {
             md-flushes yes;
             disk-flushes yes;
             c-plan-ahead 1;
             c-min-rate 100M;
             c-fill-target 20M;
             c-max-rate 4G;
        }
        net {
             after-sb-0pri discard-younger-primary;
             after-sb-1pri discard-secondary;
             after-sb-2pri call-pri-lost-after-sb;
             protocol     C;
             tcp-cork yes;
             max-buffers 20000;
             max-epoch-size 20000;
             sndbuf-size 0;
             rcvbuf-size 0;
        }
   }
   </code></pre>

1. **[A]** Создайте устройства NFS DRBD.

   <pre><code>sudo vi /etc/drbd.d/<b>NW1</b>-nfs.res
   </code></pre>

   Вставьте конфигурацию для нового устройства DRBD и выйдите.

   <pre><code>resource <b>NW1</b>-nfs {
        protocol     C;
        disk {
             on-io-error       detach;
        }
        on <b>prod-nfs-0</b> {
             address   <b>10.0.0.6:7790</b>;
             device    /dev/drbd<b>0</b>;
             disk      /dev/<b>vg-NW1-NFS</b>/<b>NW1</b>;
             meta-disk internal;
        }
        on <b>prod-nfs-1</b> {
             address   <b>10.0.0.7:7790</b>;
             device    /dev/drbd<b>0</b>;
             disk      /dev/<b>vg-NW1-NFS</b>/<b>NW1</b>;
             meta-disk internal;
        }
   }
   </code></pre>

   <pre><code>sudo vi /etc/drbd.d/<b>NW2</b>-nfs.res
   </code></pre>

   Вставьте конфигурацию для нового устройства DRBD и выйдите.

   <pre><code>resource <b>NW2</b>-nfs {
        protocol     C;
        disk {
             on-io-error       detach;
        }
        on <b>prod-nfs-0</b> {
             address   <b>10.0.0.6:7791</b>;
             device    /dev/drbd<b>1</b>;
             disk      /dev/<b>vg-NW2-NFS</b>/<b>NW2</b>;
             meta-disk internal;
        }
        on <b>prod-nfs-1</b> {
             address   <b>10.0.0.7:7791</b>;
             device    /dev/drbd<b>1</b>;
             disk      /dev/<b>vg-NW2-NFS</b>/<b>NW2</b>;
             meta-disk internal;
        }
   }
   </code></pre>

   Создайте устройство DRBD и запустите его.

   <pre><code>sudo drbdadm create-md <b>NW1</b>-nfs
   sudo drbdadm create-md <b>NW2</b>-nfs
   sudo drbdadm up <b>NW1</b>-nfs
   sudo drbdadm up <b>NW2</b>-nfs
   </code></pre>

1. **[1]** Пропустите начальную синхронизацию.

   <pre><code>sudo drbdadm new-current-uuid --clear-bitmap <b>NW1</b>-nfs
   sudo drbdadm new-current-uuid --clear-bitmap <b>NW2</b>-nfs
   </code></pre>

1. **[1]** Задайте первичный узел.

   <pre><code>sudo drbdadm primary --force <b>NW1</b>-nfs
   sudo drbdadm primary --force <b>NW2</b>-nfs
   </code></pre>

1. **[1]** Подождите, пока новые устройства DRBD синхронизируются.

   <pre><code>sudo drbdsetup wait-sync-resource NW1-nfs
   sudo drbdsetup wait-sync-resource NW2-nfs
   </code></pre>

1. **[1]** Создайте файловые системы на устройствах DRBD.

   <pre><code>sudo mkfs.xfs /dev/drbd0
   sudo mkdir /srv/nfs/NW1
   sudo chattr +i /srv/nfs/NW1
   sudo mount -t xfs /dev/drbd0 /srv/nfs/NW1
   sudo mkdir /srv/nfs/NW1/sidsys
   sudo mkdir /srv/nfs/NW1/sapmntsid
   sudo mkdir /srv/nfs/NW1/trans
   sudo mkdir /srv/nfs/NW1/ASCS
   sudo mkdir /srv/nfs/NW1/ASCSERS
   sudo mkdir /srv/nfs/NW1/SCS
   sudo mkdir /srv/nfs/NW1/SCSERS
   sudo umount /srv/nfs/NW1

   sudo mkfs.xfs /dev/drbd1
   sudo mkdir /srv/nfs/NW2
   sudo chattr +i /srv/nfs/NW2
   sudo mount -t xfs /dev/drbd1 /srv/nfs/NW2
   sudo mkdir /srv/nfs/NW2/sidsys
   sudo mkdir /srv/nfs/NW2/sapmntsid
   sudo mkdir /srv/nfs/NW2/trans
   sudo mkdir /srv/nfs/NW2/ASCS
   sudo mkdir /srv/nfs/NW2/ASCSERS
   sudo mkdir /srv/nfs/NW2/SCS
   sudo mkdir /srv/nfs/NW2/SCSERS
   sudo umount /srv/nfs/NW2
   </code></pre>

1. **[A]** Настройте обнаружение нескольких основных устройств DRBD.

   При использовании DRBD для синхронизации данных одного узла с другим может возникнуть нескольких основных устройств. Разделенный мозг — это сценарий, в котором оба узла кластера определили устройство DRBD в качестве основного и утратили синхронизацию. Эта ситуация может возникать редко, но вам следует обработать и устранить проблему "разделенного мозга" как можно быстрее. Поэтому важно получать уведомления о возникновении нескольких основных устройств.

   Ознакомьтесь с [официальной документацией по DRBD](https://www.linbit.com/drbd-user-guide/users-guide-drbd-8-4/#s-split-brain-notification), чтобы узнать, как настроить уведомление о возникновении нескольких основных устройств.

   Можно также осуществить автоматическое восстановление в случае возникновения нескольких основных устройств. Дополнительные сведения см. в статье [Automatic split brain recovery policies](https://www.linbit.com/drbd-user-guide/users-guide-drbd-8-4/#s-automatic-split-brain-recovery-configuration) (Политики автоматического восстановления при возникновении нескольких основных устройств).
   
### <a name="configure-cluster-framework"></a>Настройка платформы кластера

1. **[1]** Добавьте в конфигурацию кластера устройства NFS DRBD для системы SAP NW1.

   > [!IMPORTANT]
   > В ходе последних тестов были выявлены ситуации, когда netcat перестает отвечать на запросы из-за невыполненной работы и ограничения на обработку только одного подключения. Ресурс netcat прекращает прослушивать запросы Azure Load Balancer, и плавающий IP-адрес становится недоступным.  
   > Для существующих кластеров Pacemaker ранее рекомендовалось заменять netcat на socat. Сейчас мы рекомендуем использовать агент ресурса azure-lb, который входит в состав пакета resource-agents со следующими требованиями к версии пакета:
   > - Минимальная версия для SLES 12 с пактом обновления 4 или 5 (SP4/SP5) — resource-agents-4.3.018.a7fb5035-3.30.1.  
   > - Минимальная версия для SLES 15/15 с пакетом обновления 1 (SP1) — resource-agents-4.3.0184.6ee15eb2-4.13.1.  
   >
   > Обратите внимание, что для этого изменения потребуется незначительное время простоя.  
   > Если конфигурация существующих кластеров Pacemaker уже была изменена для использования socat, как описано в [этой статье](https://www.suse.com/support/kb/doc/?id=7024128), немедленно переключаться на агент ресурса azure-lb не нужно.

   <pre><code>sudo crm configure rsc_defaults resource-stickiness="200"

   # Enable maintenance mode
   sudo crm configure property maintenance-mode=true
   
   sudo crm configure primitive drbd_<b>NW1</b>_nfs \
     ocf:linbit:drbd \
     params drbd_resource="<b>NW1</b>-nfs" \
     op monitor interval="15" role="Master" \
     op monitor interval="30" role="Slave"
   
   sudo crm configure ms ms-drbd_<b>NW1</b>_nfs drbd_<b>NW1</b>_nfs \
     meta master-max="1" master-node-max="1" clone-max="2" \
     clone-node-max="1" notify="true" interleave="true"
   
   sudo crm configure primitive fs_<b>NW1</b>_sapmnt \
     ocf:heartbeat:Filesystem \
     params device=/dev/drbd0 \
     directory=/srv/nfs/<b>NW1</b>  \
     fstype=xfs \
     op monitor interval="10s"
   
   sudo crm configure primitive nfsserver systemd:nfs-server \
     op monitor interval="30s"
   sudo crm configure clone cl-nfsserver nfsserver

   sudo crm configure primitive exportfs_<b>NW1</b> \
     ocf:heartbeat:exportfs \
     params directory="/srv/nfs/<b>NW1</b>" \
     options="rw,no_root_squash,crossmnt" clientspec="*" fsid=1 wait_for_leasetime_on_stop=true op monitor interval="30s"
   
   sudo crm configure primitive vip_<b>NW1</b>_nfs \
     IPaddr2 \
     params ip=<b>10.0.0.4</b> cidr_netmask=<b>24</b> op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>NW1</b>_nfs azure-lb port=<b>61000</b>
   
   sudo crm configure group g-<b>NW1</b>_nfs \
     fs_<b>NW1</b>_sapmnt exportfs_<b>NW1</b> nc_<b>NW1</b>_nfs vip_<b>NW1</b>_nfs
   
   sudo crm configure order o-<b>NW1</b>_drbd_before_nfs inf: \
     ms-drbd_<b>NW1</b>_nfs:promote g-<b>NW1</b>_nfs:start
   
   sudo crm configure colocation col-<b>NW1</b>_nfs_on_drbd inf: \
     g-<b>NW1</b>_nfs ms-drbd_<b>NW1</b>_nfs:Master
   </code></pre>

1. **[1]** Добавьте в конфигурацию кластера устройства NFS DRBD для системы SAP NW2.

   <pre><code># Enable maintenance mode
   sudo crm configure property maintenance-mode=true
   
   sudo crm configure primitive drbd_<b>NW2</b>_nfs \
     ocf:linbit:drbd \
     params drbd_resource="<b>NW2</b>-nfs" \
     op monitor interval="15" role="Master" \
     op monitor interval="30" role="Slave"
   
   sudo crm configure ms ms-drbd_<b>NW2</b>_nfs drbd_<b>NW2</b>_nfs \
     meta master-max="1" master-node-max="1" clone-max="2" \
     clone-node-max="1" notify="true" interleave="true"
   
   sudo crm configure primitive fs_<b>NW2</b>_sapmnt \
     ocf:heartbeat:Filesystem \
     params device=/dev/drbd1 \
     directory=/srv/nfs/<b>NW2</b>  \
     fstype=xfs \
     op monitor interval="10s"
   
   sudo crm configure primitive exportfs_<b>NW2</b> \
     ocf:heartbeat:exportfs \
     params directory="/srv/nfs/<b>NW2</b>" \
     options="rw,no_root_squash,crossmnt" clientspec="*" fsid=2 wait_for_leasetime_on_stop=true op monitor interval="30s"
   
   sudo crm configure primitive vip_<b>NW2</b>_nfs \
     IPaddr2 \
     params ip=<b>10.0.0.5</b> cidr_netmask=<b>24</b> op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>NW2</b>_nfs azure-lb port=<b>61001</b>
   
   sudo crm configure group g-<b>NW2</b>_nfs \
     fs_<b>NW2</b>_sapmnt exportfs_<b>NW2</b> nc_<b>NW2</b>_nfs vip_<b>NW2</b>_nfs
   
   sudo crm configure order o-<b>NW2</b>_drbd_before_nfs inf: \
     ms-drbd_<b>NW2</b>_nfs:promote g-<b>NW2</b>_nfs:start
   
   sudo crm configure colocation col-<b>NW2</b>_nfs_on_drbd inf: \
     g-<b>NW2</b>_nfs ms-drbd_<b>NW2</b>_nfs:Master
   </code></pre>

   Параметр `crossmnt` в ресурсах кластера `exportfs` представлен в нашей документации для обеспечения обратной совместимости с более старыми версиями SLES.  

1. **[1]** Отключите режим обслуживания.
   
   <pre><code>sudo crm configure property maintenance-mode=false
   </code></pre>

## <a name="next-steps"></a>Дальнейшие действия

* [Настройка кластера Pacemaker в SUSE Linux Enterprise Server в Azure](high-availability-guide-suse.md)
* [SAP NetWeaver на виртуальных машинах Windows. Руководство по планированию и внедрению][planning-guide]
* [Развертывание виртуальных машин Azure для SAP NetWeaver][deployment-guide]
* [SAP NetWeaver на виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server][dbms-guide]
* Дополнительные сведения об установке высокого уровня доступности и плана для аварийного восстановления SAP HANA на виртуальных машинах Azure см. в статье [Высокий уровень доступности SAP HANA на виртуальных машинах Azure][sap-hana-ha].
