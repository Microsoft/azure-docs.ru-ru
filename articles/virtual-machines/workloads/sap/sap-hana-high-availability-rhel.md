---
title: Обеспечение высокого уровня доступности SAP HANA на виртуальных машинах Azure под управлением RHEL | Документация Майкрософт
description: Обеспечение высокого уровня доступности SAP HANA на виртуальных машинах Azure.
services: virtual-machines-linux
documentationcenter: ''
author: rdeltcheva
manager: juergent
editor: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/16/2021
ms.author: radeltch
ms.openlocfilehash: daa0a6b15d4c187efdea96fd8067b08c89fa0e82
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104599873"
---
# <a name="high-availability-of-sap-hana-on-azure-vms-on-red-hat-enterprise-linux"></a>Обеспечение высокого уровня доступности SAP HANA в виртуальных машинах Azure в Red Hat Enterprise Linux

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
[2388694]:https://launchpad.support.sap.com/#/notes/2388694
[2292690]:https://launchpad.support.sap.com/#/notes/2292690
[2455582]:https://launchpad.support.sap.com/#/notes/2455582
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2009879]:https://launchpad.support.sap.com/#/notes/2009879

[sap-swcenter]:https://launchpad.support.sap.com/#/softwarecenter
[template-multisid-db]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json

Для разработки в локальной среде вы можете обеспечить высокий уровень доступности для SAP HANA с помощью системной репликации HANA или общего хранилища.
В настоящее время на виртуальных машинах Azure поддерживается только одна функция высокого уровня доступности — системная репликация HANA в Azure.
Для репликации SAP HANA используются один основной узел и по крайней мере один вторичный узел. Изменения данных на основном узле синхронно или асинхронно реплицируются на вторичные узлы.

В этой статье описывается развертывание и настройка виртуальных машин, установка платформы кластера, а также установка и настройка системной репликации SAP HANA.
В примерах конфигурации и командах установки используется номер экземпляра **03** и идентификатор системы HANA **HN1**.

Прежде всего прочитайте следующие примечания и документы SAP:

* примечание к SAP [1928533], которое содержит:
  * список размеров виртуальных машин Azure, поддерживаемых для развертывания ПО SAP;
  * важные сведения о доступных ресурсах для каждого размера виртуальной машины Azure;
  * сведения о поддерживаемом программном обеспечении SAP и сочетаниях операционных систем и баз данных;
  * сведения о требуемой версии ядра SAP для Windows и Linux в Microsoft Azure.
* примечание к SAP [2015553], в котором описываются предварительные требования к SAP при развертывании программного обеспечения SAP в Azure;
* примечание к SAP [2002167], содержащее рекомендуемые параметры ОС для Red Hat Enterprise Linux;
* примечание к SAP [2009879], содержащее рекомендации по SAP HANA для Red Hat Enterprise Linux;
* примечание к SAP [2178632], содержащее подробные сведения обо всех доступных метриках мониторинга для SAP в Azure;
* примечание к SAP [2191498], содержащее сведения о необходимой версии агента SAP Host Agent для Linux в Azure;
* примечание к SAP [2243692], содержащее сведения о лицензировании SAP в Linux в Azure;
* примечание к SAP [1999351], содержащее дополнительные сведения об устранении неполадок, связанных с расширением для расширенного мониторинга Azure для SAP;
* [вики-сайт сообщества SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes), содержащий все необходимые примечания к SAP для Linux;
* [SAP NetWeaver на виртуальных машинах Windows. Руководство по планированию и внедрению][planning-guide];
* [Развертывание программного обеспечения SAP на виртуальных машинах Linux в Azure][deployment-guide] (эта статья);
* [SAP NetWeaver на виртуальных машинах Windows. Руководство по развертыванию СУБД][dbms-guide];
* [Репликация системы SAP HANA в кластере pacemaker](https://access.redhat.com/articles/3004101)
* Общая документация по RHEL
  * [Общие сведения о надстройке для обеспечения высокой доступности](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [Администрирование надстройки для обеспечения высокой доступности](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [Справочник по надстройке для обеспечения высокой доступности](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
* Документация по RHEL для Azure:
  * [Политики поддержки для кластеров высокой доступности RHEL — виртуальные машины Microsoft Azure как члены кластера](https://access.redhat.com/articles/3131341)
  * [Установка и настройка кластера высокой доступности Red Hat Enterprise Linux 7.4 (и более поздних версий) в Microsoft Azure](https://access.redhat.com/articles/3252491)
  * [Установка SAP HANA в Red Hat Enterprise Linux для использования в Microsoft Azure](https://access.redhat.com/solutions/3193782)

## <a name="overview"></a>Обзор

Чтобы достичь высокого уровня доступности, SAP HANA устанавливается на двух виртуальных машинах. Данные реплицируются с использованием системной репликации HANA.

![Обзор высокого уровня доступности SAP HANA](./media/sap-hana-high-availability-rhel/ha-hana.png)

Для системной репликации SAP HANA используется выделенное виртуальное имя узла и виртуальный IP-адрес. Балансировщику нагрузки в Azure нужен виртуальный IP-адрес. Ниже показана конфигурация подсистемы балансировки нагрузки.

* Интерфейсная конфигурация: IP-адрес 10.0.0.13 для hn1-db
* Внутренняя конфигурация: подключена к основным сетевым интерфейсам всех виртуальных машин, которые должны входить в репликацию системы HANA.
* Порт пробы: Порт 62503
* Правила балансировки нагрузки: протокол TCP 30313, 30315, 30317, 30340, 30341, 30342.

## <a name="deploy-for-linux"></a>Развертывание для Linux

В Azure Marketplace доступен образ для Red Hat Enterprise Linux 7.4 для SAP HANA, который можно использовать для развертывания новых виртуальных машин.

### <a name="deploy-with-a-template"></a>Развертывание с помощью шаблона

Все необходимые ресурсы можно развернуть с помощью шаблонов быстрого запуска с сайта GitHub. Шаблон развертывает виртуальные машины, подсистему балансировки нагрузки, группу доступности и т. д.
Чтобы развернуть шаблон, сделайте следующее.

1. Откройте [шаблон базы данных][template-multisid-db] на портале Azure.
1. Задайте следующие параметры:
    * **Идентификатор системы SAP**: Введите идентификатор системы SAP, которую требуется установить. Идентификатор будет использоваться в качестве префикса для развертываемых ресурсов.
    * **Тип ОС**: Выберите один из дистрибутивов Linux. Для этого примера выберите **RHEL 7**.
    * **Тип базы данных**: выберите **HANA**.
    * **Размер системы SAP**: введите число протоколов SAP, которое должна предоставлять новая система. Если вы не знаете, сколько систем SAP потребуется, обратитесь к партнеру по технологиям или системному интегратору SAP.
    * **Доступность системы**: Выберите **высокую доступность**.
    * **Имя пользователя и пароль администратора или ключ SSH**: Будет создана учетная запись пользователя, которую можно использовать для входа на виртуальную машину.
    * **Идентификатор подсети**: Чтобы развернуть виртуальную машину в имеющейся виртуальной сети с определенной подсетью, необходимо указать идентификатор этой определенной подсети. Идентификатор обычно выглядит как **/Subscriptions/ \<subscription ID> /resourceGroups/ \<resource group name> /провидерс/Микрософт.Нетворк/виртуалнетворкс/ \<virtual network name> /субнетс/ \<subnet name>**. Оставьте пустым, если нужно создать новую виртуальную сеть

### <a name="manual-deployment"></a>Развертывание вручную

1. Создайте группу ресурсов.
1. Создайте виртуальную сеть.
1. Создайте группу доступности.  
   Настройте максимальное число доменов обновления.
1. Создайте подсистему балансировки нагрузки (внутреннюю). Мы рекомендуем [подсистему балансировки нагрузки ценовой категории "Стандартный"](../../../load-balancer/load-balancer-overview.md).
   * Выберите виртуальную сеть, созданную на шаге 2.
1. Создайте виртуальную машину 1.  
   Используйте версию не ниже Red Hat Enterprise Linux 7.4 для SAP HANA. В этом примере используется образ Red Hat Enterprise Linux 7.4 для SAP HANA <https://portal.azure.com/#create/RedHat.RedHatEnterpriseLinux75forSAP-ARM> выберите группу доступности, созданную в шаге 3.
1. Создайте виртуальную машину 2.  
   Используйте версию не ниже Red Hat Enterprise Linux 7.4 для SAP HANA. В этом примере используется образ Red Hat Enterprise Linux 7.4 для SAP HANA <https://portal.azure.com/#create/RedHat.RedHatEnterpriseLinux75forSAP-ARM> выберите группу доступности, созданную в шаге 3.
1. Добавьте диски данных.

> [!IMPORTANT]
> Плавающий IP-адрес не поддерживается для вторичной IP-конфигурации NIC в сценариях балансировки нагрузки. Дополнительные сведения см. в статье [ограничения балансировщика нагрузки Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Если для виртуальной машины требуется дополнительный IP-адрес, разверните вторую сетевую карту.    

> [!Note]
> Если в серверный пул внутреннего (без общедоступного IP-адреса) Azure Load Balancer ценовой категории "Стандартный" помещаются виртуальные машины без общедоступных IP-адресов, у них не будет исходящего подключения к Интернету без дополнительной настройки, разрешающей маршрутизацию к общедоступным конечным точкам. Подробные сведения о такой настройке см. в статье [Подключение к общедоступной конечной точке для виртуальных машин с помощью Azure Load Balancer (цен. категория "Стандартный") в сценариях обеспечения высокого уровня доступности SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

1. Если вы используете подсистему балансировки нагрузки ценовой категории "Стандартный", выполните следующие шаги по настройке.
   1. Сначала создайте пула IP-адресов для интерфейсной части.

      1. Откройте подсистему балансировки нагрузки, выберите **пул интерфейсных IP-адресов** и щелкните **Добавить**.
      1. Введите имя нового пула интерфейсных IP-адресов (например, **hana-frontend**).
      1. Для параметра **Назначение** выберите значение **Статическое** и введите IP-адрес (например, **10.0.0.13**).
      1. Щелкните **ОК**.
      1. Когда пул интерфейсных IP-адресов будет создан, запишите его IP-адрес.

   1. Теперь создайте серверный пул.

      1. Откройте подсистему балансировки нагрузки, выберите **серверные пулы** и щелкните **Добавить**.
      1. Введите имя нового серверного пула (например, **hana-backend**).
      1. Щелкните **Добавить виртуальную машину**.
      1. Выберите **Виртуальная машина**.
      1. Выберите виртуальные машины кластера SAP HANA и IP-адреса для них.
      1. Выберите **Добавить**.

   1. Создайте зонд работоспособности.

      1. Откройте подсистему балансировки нагрузки, выберите **Зонды работоспособности** и щелкните **Добавить**.
      1. Введите имя нового зонда работоспособности (например, **hana-hp**).
      1. Выберите протокол **TCP** и порт 625 **03**. Сохраните значение "5" для параметра **Интервал** и значение "2" для параметра **Порог состояния неработоспособности**.
      1. Щелкните **ОК**.

   1. Теперь создайте правила балансировки нагрузки.
   
      1. Откройте подсистему балансировки нагрузки, выберите **Правила балансировки нагрузки** щелкните **Добавить**.
      1. Введите имя нового правила балансировки нагрузки (например, **hana-lb**).
      1. Выберите интерфейсный пул IP-адресов, серверный пул и зонд работоспособности, который вы создали ранее (например, **hana-frontend**, **hana-backend** и **hana-hp**).
      1. Выберите **Порты высокой доступности**.
      1. Увеличьте **время ожидания** до 30 минут.
      1. Не забудьте **включить плавающий IP-адрес**.
      1. Щелкните **ОК**.


1. Если же ваш сценарий требует использовать подсистему балансировки нагрузки ценовой категории "Базовый", выполните следующие шаги по настройке.
   1. Настройте подсистему балансировки нагрузки. Сначала создайте пула IP-адресов для интерфейсной части.

      1. Откройте подсистему балансировки нагрузки, выберите **пул интерфейсных IP-адресов** и щелкните **Добавить**.
      1. Введите имя нового пула интерфейсных IP-адресов (например, **hana-frontend**).
      1. Для параметра **Назначение** выберите значение **Статическое** и введите IP-адрес (например, **10.0.0.13**).
      1. Щелкните **ОК**.
      1. Когда пул интерфейсных IP-адресов будет создан, запишите его IP-адрес.

   1. Теперь создайте серверный пул.

      1. Откройте подсистему балансировки нагрузки, выберите **серверные пулы** и щелкните **Добавить**.
      1. Введите имя нового серверного пула (например, **hana-backend**).
      1. Щелкните **Добавить виртуальную машину**.
      1. Выберите группу доступности, созданную на шаге 3.
      1. Выберите виртуальные машины кластера SAP HANA.
      1. Щелкните **ОК**.

   1. Создайте зонд работоспособности.

      1. Откройте подсистему балансировки нагрузки, выберите **Зонды работоспособности** и щелкните **Добавить**.
      1. Введите имя нового зонда работоспособности (например, **hana-hp**).
      1. Выберите протокол **TCP** и порт 625 **03**. Сохраните значение "5" для параметра **Интервал** и значение "2" для параметра **Порог состояния неработоспособности**.
      1. Щелкните **ОК**.

   1. Если используется SAP HANA 1.0, создайте правила балансировки нагрузки.

      1. Откройте подсистему балансировки нагрузки, выберите **Правила балансировки нагрузки** щелкните **Добавить**.
      1. Введите имя нового правила балансировки нагрузки (например, hana-lb-3 **03** 15).
      1. Выберите интерфейсный пул IP-адресов, серверный пул и зонд работоспособности, который вы создали ранее (например, **hana-frontend**).
      1. Для параметра **Протокол** сохраните значение **TCP** и введите порт 3 **03** 15.
      1. Увеличьте **время ожидания** до 30 минут.
      1. Не забудьте **включить плавающий IP-адрес**.
      1. Щелкните **ОК**.
      1. Повторите эти шаги для порта 3 **03** 17.

   1. Если используется SAP HANA 2.0, создайте правила балансировки нагрузки для системной базы данных.

      1. Откройте подсистему балансировки нагрузки, выберите **Правила балансировки нагрузки** щелкните **Добавить**.
      1. Введите имя нового правила балансировки нагрузки (например, hana-lb-3 **03** 13).
      1. Выберите интерфейсный пул IP-адресов, серверный пул и зонд работоспособности, который вы создали ранее (например, **hana-frontend**).
      1. Для параметра **Протокол** сохраните значение **TCP** и введите порт 3 **03** 13.
      1. Увеличьте **время ожидания** до 30 минут.
      1. Не забудьте **включить плавающий IP-адрес**.
      1. Щелкните **ОК**.
      1. Повторите эти шаги для порта 3 **03** 14.

   1. Если используется SAP HANA 2.0, создайте правила балансировки нагрузки для базы данных клиента.

      1. Откройте подсистему балансировки нагрузки, выберите **Правила балансировки нагрузки** щелкните **Добавить**.
      1. Введите имя нового правила балансировки нагрузки (например, hana-lb-3 **03** 40).
      1. Выберите пул внешних IP-адресов, внутренний пул и зонд работоспособности, созданные ранее (например, **hana-frontend**).
      1. Для параметра **Протокол** сохраните значение **TCP** и введите порт 3 **03** 40.
      1. Увеличьте **время ожидания** до 30 минут.
      1. Не забудьте **включить плавающий IP-адрес**.
      1. Щелкните **ОК**.
      1. Повторите эти шаги для портов 3 **03** 41 и 3 **03** 42.

Дополнительные сведения о портах для SAP HANA см. в главе о [подключениях к базам данных арендатора](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6/latest/en-US/7a9343c9f2a2436faa3cfdb5ca00c052.html) руководства по [базам данных клиентов SAP HANA](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6) или в [примечании к SAP № 2388694][2388694].

> [!IMPORTANT]
> Не включайте метки времени TCP на виртуальных машинах Azure, размещенных за Azure Load Balancer. Включение меток времени TCP помешает работе проб работоспособности. Установите для параметра **net.ipv4.tcp_timestamps** значение **0**. Дополнительные сведения см. в статье [Пробы работоспособности Load Balancer](../../../load-balancer/load-balancer-custom-probe-overview.md).
> Также см. [примечание к SAP 2382421](https://launchpad.support.sap.com/#/notes/2382421). 

## <a name="install-sap-hana"></a>Установка SAP HANA

Во всех действиях этого раздела используются следующие префиксы.

* **[A]** : шаг применяется ко всем узлам.
* **[1]** : шаг применяется только к узлу 1.
* **[2]** : шаг применяется к узлу 2 только в кластере Pacemaker.

1. **[A]** Настройте разметку диска: **диспетчер логических томов (LVM)** .

   Мы рекомендуем использовать диспетчер логических томов для всех томов, предназначенных для хранения данных и файлов журнала. В приведенном ниже примере предполагается, что к виртуальным машинам подключены четыре диска данных, на которых созданы два тома.

   Вывод списка всех доступных дисков:

   <pre><code>ls /dev/disk/azure/scsi1/lun*
   </code></pre>

   Выходные данные примера:

   <pre><code>
   /dev/disk/azure/scsi1/lun0  /dev/disk/azure/scsi1/lun1  /dev/disk/azure/scsi1/lun2  /dev/disk/azure/scsi1/lun3
   </code></pre>

   Создайте физические тома для всех дисков, которые вы хотите использовать.

   <pre><code>sudo pvcreate /dev/disk/azure/scsi1/lun0
   sudo pvcreate /dev/disk/azure/scsi1/lun1
   sudo pvcreate /dev/disk/azure/scsi1/lun2
   sudo pvcreate /dev/disk/azure/scsi1/lun3
   </code></pre>

   Создайте группу томов для файлов данных. Используйте одну группу томов для файлов журналов и другую — для общей папки SAP HANA.

   <pre><code>sudo vgcreate vg_hana_data_<b>HN1</b> /dev/disk/azure/scsi1/lun0 /dev/disk/azure/scsi1/lun1
   sudo vgcreate vg_hana_log_<b>HN1</b> /dev/disk/azure/scsi1/lun2
   sudo vgcreate vg_hana_shared_<b>HN1</b> /dev/disk/azure/scsi1/lun3
   </code></pre>

   Создайте логические тома. Линейный том создается при использовании `lvcreate` без параметра `-i`. Мы рекомендуем создать чередующийся том для повышения производительности операций ввода-вывода, определяя размеры блоков чередования по значениям из статьи [Конфигурации хранилища виртуальных машин SAP HANA в Azure](./hana-vm-operations-storage.md). Аргумент `-i` обозначает число базовых физических томов, а аргумент `-I` — размер блоков чередования. В этом документе для тома данных используется 2 физических тома, поэтому аргумент `-i` имеет значение **2**. Размер блока чередования для тома данных составляет **256 КиБ**. Журнальный том использует один физический том, поэтому параметры `-i` и `-I` не используются явным образом в командах для журнального тома.  

   > [!IMPORTANT]
   > Параметр `-i`, значение которого соответствует числу базовых физических томов, необходимо указывать для всех томов данных, журналов или общих томов, для которых используется более одного физического тома. Используйте параметр `-I`, чтобы указать размер блока чередования при создании чередующегося тома.  
   > Сведения о рекомендуемых конфигурациях хранилища, включая размеры блоков чередования и количество дисков, см. в статье [Конфигурации хранилища виртуальных машин SAP HANA в Azure](./hana-vm-operations-storage.md).  

   <pre><code>sudo lvcreate <b>-i 2</b> <b>-I 256</b> -l 100%FREE -n hana_data vg_hana_data_<b>HN1</b>
   sudo lvcreate -l 100%FREE -n hana_log vg_hana_log_<b>HN1</b>
   sudo lvcreate -l 100%FREE -n hana_shared vg_hana_shared_<b>HN1</b>
   sudo mkfs.xfs /dev/vg_hana_data_<b>HN1</b>/hana_data
   sudo mkfs.xfs /dev/vg_hana_log_<b>HN1</b>/hana_log
   sudo mkfs.xfs /dev/vg_hana_shared_<b>HN1</b>/hana_shared
   </code></pre>

   Создайте каталоги подключения и скопируйте идентификаторы UUID всех логических томов.

   <pre><code>sudo mkdir -p /hana/data/<b>HN1</b>
   sudo mkdir -p /hana/log/<b>HN1</b>
   sudo mkdir -p /hana/shared/<b>HN1</b>
   # Write down the ID of /dev/vg_hana_data_<b>HN1</b>/hana_data, /dev/vg_hana_log_<b>HN1</b>/hana_log, and /dev/vg_hana_shared_<b>HN1</b>/hana_shared
   sudo blkid
   </code></pre>

   Создайте записи `fstab` для трех логических томов.

   <pre><code>sudo vi /etc/fstab
   </code></pre>

   Вставьте следующую строку в файл `/etc/fstab`:

   <pre><code>/dev/disk/by-uuid/<b>&lt;UUID of /dev/mapper/vg_hana_data_<b>HN1</b>-hana_data&gt;</b> /hana/data/<b>HN1</b> xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<b>&lt;UUID of /dev/mapper/vg_hana_log_<b>HN1</b>-hana_log&gt;</b> /hana/log/<b>HN1</b> xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<b>&lt;UUID of /dev/mapper/vg_hana_shared_<b>HN1</b>-hana_shared&gt;</b> /hana/shared/<b>HN1</b> xfs  defaults,nofail  0  2
   </code></pre>

   Подключите новые тома:

   <pre><code>sudo mount -a
   </code></pre>

1. **[A]** Настройте разметку диска: **Обычные диски**.

   Для демонстрационных систем файлы данных и журналов HANA можно поместить на один диск. Создайте раздел в пути /dev/disk/azure/scsi1/lun0 и отформатируйте его для файловой системы XFS.

   <pre><code>sudo sh -c 'echo -e "n\n\n\n\n\nw\n" | fdisk /dev/disk/azure/scsi1/lun0'
   sudo mkfs.xfs /dev/disk/azure/scsi1/lun0-part1
   
   # Write down the ID of /dev/disk/azure/scsi1/lun0-part1
   sudo /sbin/blkid
   sudo vi /etc/fstab
   </code></pre>

   Добавьте такую строку в файл /etc/fstab:

   <pre><code>/dev/disk/by-uuid/<b>&lt;UUID&gt;</b> /hana xfs  defaults,nofail  0  2
   </code></pre>

   Создайте целевой каталог и подключите диск:

   <pre><code>sudo mkdir /hana
   sudo mount -a
   </code></pre>

1. **[A]** Настройка разрешения имен узлов для всех узлов.

   Вы можете использовать DNS-сервер или внести изменения в файл /etc/hosts на всех узлах. В этом примере используется файл /etc/hosts.
   Замените IP-адрес и имя узла в следующих командах:

   <pre><code>sudo vi /etc/hosts
   </code></pre>

   Вставьте следующие строки в файл /etc/hosts. Измените IP-адрес и имя узла в соответствии с параметрами среды.

   <pre><code><b>10.0.0.5 hn1-db-0</b>
   <b>10.0.0.6 hn1-db-1</b>
   </code></pre>

1. **[A]** Настройка RHEL для HANA

   Настройте RHEL, как описано в <https://access.redhat.com/solutions/2447641> и в следующих примечаниях SAP:  
   - [2292690 - SAP HANA DB: Recommended OS settings for RHEL 7](https://launchpad.support.sap.com/#/notes/2292690) (Примечание по поддержке SAP № 2292690. База данных SAP HANA: рекомендуемые параметры операционной системы для RHEL 7).
   - [2777782-SAP HANA DB: Рекомендуемые параметры ОС для RHEL 8](https://launchpad.support.sap.com/#/notes/2777782)
   - [2455582-Linux: запуск приложений SAP, скомпилированных с помощью GCC 6. x](https://launchpad.support.sap.com/#/notes/2455582)
   - [2593824-Linux: запуск приложений SAP, скомпилированных с помощью GCC 7. x](https://launchpad.support.sap.com/#/notes/2593824) 
   - [2886607-Linux: запуск приложений SAP, скомпилированных с помощью GCC 9. x](https://launchpad.support.sap.com/#/notes/2886607)

1. **[A]** Установка SAP HANA

   Чтобы установить репликацию системы SAP HANA, следуйте инструкциям на странице <https://access.redhat.com/articles/3004101>.

   * Запустите программу **hdblcm** с DVD-диска HANA. Введите следующие значения на запрос в командной строке:
   * Choose installation (Выберите установку). Введите **1**.
   * Select additional components for installation (Выберите дополнительные компоненты для установки). Введите **1**.
   * Enter Installation Path (Введите путь установки) [/hana/shared]. Нажмите клавишу ВВОД.
   * Enter Local Host Name (Введите имя локального узла) [..]. Нажмите клавишу ВВОД.
   * "Do you want to add additional hosts to the system? (y/n)" (Вы действительно хотите продолжить? (Да/нет)) [нет]. Нажмите клавишу ВВОД.
   * Enter SAP HANA System ID (Введите идентификатор системы SAP HANA). Введите идентификатор безопасности HANA, например: **HN1**.
   * Введите номер экземпляра [00]. Введите номер экземпляра HANA. Введите **03**, если вы использовали шаблон Azure или выполняли развертывание вручную по инструкциям из этой статьи.
   * Select Database Mode / Enter Index (Выберите режим базы данных и введите индекс) [1]. Нажмите клавишу ВВОД.
   * Выберите использование системы и введите индекс [4]. Выберите значение потребления системы.
   * Enter Location of Data Volumes (Введите расположение томов данных) [/hana/data/HN1]. Нажмите клавишу ВВОД.
   * Enter Location of Log Volumes (Введите расположение томов журналов) [/hana/log/HN1]. Нажмите клавишу ВВОД.
   * "Restrict maximum memory allocation?" [нет]. Нажмите клавишу ВВОД.
   * Enter Certificate Host Name For Host (Введите имя узла сертификатов для узла) "…" [...]. Нажмите клавишу ВВОД.
   * Введите пароль пользователя агента узла SAP (sapadm). Введите пароль пользователя агента узла.
   * Подтвердите пароль пользователя агента узла SAP (sapadm). Введите пароль пользователя агента узла еще раз для подтверждения.
   * Введите пароль системного администратора (hdbadm). Введите пароль системного администратора.
   * Подтвердите пароль системного администратора (hdbadm). Введите пароль системного администратора еще раз для подтверждения.
   * Enter System Administrator Home Directory (Введите домашний каталог системного администратора) [/usr/sap/HN1/home]. Нажмите клавишу ВВОД.
   * Enter System Administrator Login Shell (Введите оболочку для входа системного администратора) [/bin/sh]. Нажмите клавишу ВВОД.
   * Enter System Administrator User ID (Введите идентификатор пользователя системного администратора) [1001]. Нажмите клавишу ВВОД.
   * Enter ID of User Group (sapsys) (Введите идентификатор для группы пользователей (sapsys)) [79]. Нажмите клавишу ВВОД.
   * Введите пароль пользователя базы данных (SYSTEM). Введите пароль пользователя базы данных.
   * Подтвердите пароль пользователя базы данных (SYSTEM). Введите пароль пользователя базы данных еще раз для подтверждения.
   * "Restart system after machine reboot?" [нет]. Нажмите клавишу ВВОД.
   * "Do you want to continue? (y/n)" (Вы действительно хотите продолжить? (Да/нет)). Проверьте сводку. Для продолжения введите **y**.

1. **[A]** . Обновите агент узла SAP.

   Скачайте последний архив агента узла SAP на сайте [центра программного обеспечения SAP][sap-swcenter] и выполните следующую команду, чтобы обновить этот агент. Замените путь к архиву, чтобы он указывал на скачанный файл.

   <pre><code>sudo /usr/sap/hostctrl/exe/saphostexec -upgrade -archive &lt;path to SAP Host Agent SAR&gt;
   </code></pre>

1. **[A]** Настройка брандмауэра

   Создайте правило брандмауэра для порта пробы подсистемы балансировки нагрузки Azure.

   <pre><code>sudo firewall-cmd --zone=public --add-port=625<b>03</b>/tcp
   sudo firewall-cmd --zone=public --add-port=625<b>03</b>/tcp --permanent
   </code></pre>

## <a name="configure-sap-hana-20-system-replication"></a>Настройка репликации системы SAP HANA 2.0

Во всех действиях этого раздела используются следующие префиксы.

* **[A]** : шаг применяется ко всем узлам.
* **[1]** : шаг применяется только к узлу 1.
* **[2]** : шаг применяется к узлу 2 только в кластере Pacemaker.

1. **[A]** Настройка брандмауэра

   Создайте правила брандмауэра, разрешающие передачу трафика репликации системы HANA и клиентов. Требуемые порты перечислены в разделе [Порты TCP/IP для всех продуктов SAP](https://help.sap.com/viewer/ports). Приведенные ниже команды — лишь пример того, как можно разрешить передачу трафика репликации системы HANA 2.0 и клиентов в базу данных SYSTEMDB, HN1 и NW1.

   <pre><code>sudo firewall-cmd --zone=public --add-port=40302/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40302/tcp
   sudo firewall-cmd --zone=public --add-port=40301/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40301/tcp
   sudo firewall-cmd --zone=public --add-port=40307/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40307/tcp
   sudo firewall-cmd --zone=public --add-port=40303/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40303/tcp
   sudo firewall-cmd --zone=public --add-port=40340/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40340/tcp
   sudo firewall-cmd --zone=public --add-port=30340/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=30340/tcp
   sudo firewall-cmd --zone=public --add-port=30341/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=30341/tcp
   sudo firewall-cmd --zone=public --add-port=30342/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=30342/tcp
   </code></pre>

1. **[1]** Создайте базу данных клиента.

   Если вы используете SAP HANA 2.0 или MDC, создайте базу данных клиента для системы SAP NetWeaver. Замените **NW1** идентификатором SID для системы SAP.

   Выполните следующую команду от имени <hanasid\>adm.

   <pre><code>hdbsql -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> -d SYSTEMDB 'CREATE DATABASE <b>NW1</b> SYSTEM USER PASSWORD "<b>passwd</b>"'
   </code></pre>

1. **[1]** Настройте репликацию системы на первом узле.

   Выполните резервное копирование баз данных от имени <hanasid\>adm.

   <pre><code>hdbsql -d SYSTEMDB -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackupSYS</b>')"
   hdbsql -d <b>HN1</b> -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackupHN1</b>')"
   hdbsql -d <b>NW1</b> -u SYSTEM -p "<b>passwd</b>" -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackupNW1</b>')"
   </code></pre>

   Скопируйте системные файлы PKI на вторичный сайт.

   <pre><code>scp /usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/data/SSFS_<b>HN1</b>.DAT   <b>hn1-db-1</b>:/usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/data/
   scp /usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/key/SSFS_<b>HN1</b>.KEY  <b>hn1-db-1</b>:/usr/sap/<b>HN1</b>/SYS/global/security/rsecssfs/key/
   </code></pre>

   Создайте основной сайт:

   <pre><code>hdbnsutil -sr_enable --name=<b>SITE1</b>
   </code></pre>

1. **[2]** Настройте системную репликацию на втором узле
    
   Зарегистрируйте второй узел, чтобы запустить репликацию системы. Выполните следующую команду от имени <hanasid\>adm.

   <pre><code>sapcontrol -nr <b>03</b> -function StopWait 600 10
   hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b>
   </code></pre>

1. **[1]** Проверка состояния репликации

   Проверьте состояние репликации и подождите, пока все базы данных синхронизируются. Если сохраняется неизвестное состояние, проверьте параметры брандмауэра.

   <pre><code>sudo su - <b>hn1</b>adm -c "python /usr/sap/<b>HN1</b>/HDB<b>03</b>/exe/python_support/systemReplicationStatus.py"
   # | Database | Host     | Port  | Service Name | Volume ID | Site ID | Site Name | Secondary | Secondary | Secondary | Secondary | Secondary     | Replication | Replication | Replication    |
   # |          |          |       |              |           |         |           | Host      | Port      | Site ID   | Site Name | Active Status | Mode        | Status      | Status Details |
   # | -------- | -------- | ----- | ------------ | --------- | ------- | --------- | --------- | --------- | --------- | --------- | ------------- | ----------- | ----------- | -------------- |
   # | SYSTEMDB | <b>hn1-db-0</b> | 30301 | nameserver   |         1 |       1 | <b>SITE1</b>     | <b>hn1-db-1</b>  |     30301 |         2 | <b>SITE2</b>     | YES           | SYNC        | ACTIVE      |                |
   # | <b>HN1</b>      | <b>hn1-db-0</b> | 30307 | xsengine     |         2 |       1 | <b>SITE1</b>     | <b>hn1-db-1</b>  |     30307 |         2 | <b>SITE2</b>     | YES           | SYNC        | ACTIVE      |                |
   # | <b>NW1</b>      | <b>hn1-db-0</b> | 30340 | indexserver  |         2 |       1 | <b>SITE1</b>     | <b>hn1-db-1</b>  |     30340 |         2 | <b>SITE2</b>     | YES           | SYNC        | ACTIVE      |                |
   # | <b>HN1</b>      | <b>hn1-db-0</b> | 30303 | indexserver  |         3 |       1 | <b>SITE1</b>     | <b>hn1-db-1</b>  |     30303 |         2 | <b>SITE2</b>     | YES           | SYNC        | ACTIVE      |                |
   #
   # status system replication site "2": ACTIVE
   # overall system replication status: ACTIVE
   #
   # Local System Replication State
   # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   #
   # mode: PRIMARY
   # site id: 1
   # site name: <b>SITE1</b>
   </code></pre>

## <a name="configure-sap-hana-10-system-replication"></a>Настройка репликации системы SAP HANA 1.0

Во всех действиях этого раздела используются следующие префиксы.

* **[A]** : шаг применяется ко всем узлам.
* **[1]** : шаг применяется только к узлу 1.
* **[2]** : шаг применяется к узлу 2 только в кластере Pacemaker.

1. **[A]** Настройка брандмауэра

   Создайте правила брандмауэра, разрешающие передачу трафика репликации системы HANA и клиентов. Требуемые порты перечислены в разделе [Порты TCP/IP для всех продуктов SAP](https://help.sap.com/viewer/ports). Приведенные ниже команды — лишь пример того, как разрешить репликацию системы HANA 2.0. Адаптируйте его в соответствии со своей установкой SAP HANA 1.0.

   <pre><code>sudo firewall-cmd --zone=public --add-port=40302/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=40302/tcp
   </code></pre>

1. **[1]** Создайте обязательных пользователей.

   Выполните следующую команду от имени привилегированного пользователя. Обязательно замените строки, выделенные полужирным шрифтом (идентификатор системы HANA **HN1** и номер экземпляра **03**), значениями для своей системы SAP HANA.

   <pre><code>PATH="$PATH:/usr/sap/<b>HN1</b>/HDB<b>03</b>/exe"
   hdbsql -u system -i <b>03</b> 'CREATE USER <b>hdb</b>hasync PASSWORD "<b>passwd</b>"'
   hdbsql -u system -i <b>03</b> 'GRANT DATA ADMIN TO <b>hdb</b>hasync'
   hdbsql -u system -i <b>03</b> 'ALTER USER <b>hdb</b>hasync DISABLE PASSWORD LIFETIME'
   </code></pre>

1. **[A]** Создайте запись в хранилище ключей.

   Выполните следующую команду от имени привилегированного пользователя, чтобы создать новую запись в хранилище ключей.

   <pre><code>PATH="$PATH:/usr/sap/<b>HN1</b>/HDB<b>03</b>/exe"
   hdbuserstore SET <b>hdb</b>haloc localhost:3<b>03</b>15 <b>hdb</b>hasync <b>passwd</b>
   </code></pre>

1. **[1]** Создайте резервную копию базы данных.

   Выполните резервное копирование баз данных от имени привилегированного пользователя.

   <pre><code>PATH="$PATH:/usr/sap/<b>HN1</b>/HDB<b>03</b>/exe"
   hdbsql -d SYSTEMDB -u system -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackup</b>')"
   </code></pre>

   Если вы используете мультитенантную систему, создайте резервные копии и для баз данных клиентов.

   <pre><code>hdbsql -d <b>HN1</b> -u system -i <b>03</b> "BACKUP DATA USING FILE ('<b>initialbackup</b>')"
   </code></pre>

1. **[1]** Настройте репликацию системы на первом узле.

   Создайте основной сайт от имени <hanasid\>adm.

   <pre><code>su - <b>hdb</b>adm
   hdbnsutil -sr_enable –-name=<b>SITE1</b>
   </code></pre>

1. **[2]** Настройте репликацию системы на вторичном узле

   Зарегистрируйте дополнительный сайт от имени <hanasid\>adm.

   <pre><code>HDB stop
   hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b>
   HDB start
   </code></pre>

## <a name="create-a-pacemaker-cluster"></a>Создание кластера Pacemaker

Следуйте указаниям в статье [Настройка кластера Pacemaker в Red Hat Enterprise Linux в Azure](high-availability-guide-rhel-pacemaker.md), чтобы создать базовый кластер Pacemaker для этого сервера HANA.

## <a name="create-sap-hana-cluster-resources"></a>Создание ресурсов кластера SAP HANA

Установите агенты ресурсов SAP HANA во **всех узлах**. Обязательно включите репозиторий, содержащий пакет. Если используется образ с поддержкой RHEL 8. x с высоким уровнем доступности, вам не нужно включать дополнительные репозитории.  

<pre><code># Enable repository that contains SAP HANA resource agents
sudo subscription-manager repos --enable="rhel-sap-hana-for-rhel-7-server-rpms"
   
sudo yum install -y resource-agents-sap-hana
</code></pre>

Затем создайте топологию HANA. Выполните следующие команды на любом из узлов кластера Pacemaker.

<pre><code>sudo pcs property set maintenance-mode=true

# Replace the bold string with your instance number and HANA system ID
sudo pcs resource create SAPHanaTopology_<b>HN1</b>_<b>03</b> SAPHanaTopology SID=<b>HN1</b> InstanceNumber=<b>03</b> \
op start timeout=600 op stop timeout=300 op monitor interval=10 timeout=600 \
clone clone-max=2 clone-node-max=1 interleave=true
</code></pre>

Теперь создайте ресурсы HANA.

> [!NOTE]
> Эта статья содержит ссылки на термин « *Ведомый*» термин, который корпорация Майкрософт больше не использует. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.

При создании кластера на **RHEL 7. x** используйте следующие команды:  

<pre><code># Replace the bold string with your instance number, HANA system ID, and the front-end IP address of the Azure load balancer.
#
sudo pcs resource create SAPHana_<b>HN1</b>_<b>03</b> SAPHana SID=<b>HN1</b> InstanceNumber=<b>03</b> PREFER_SITE_TAKEOVER=true DUPLICATE_PRIMARY_TIMEOUT=7200 AUTOMATED_REGISTER=false \
op start timeout=3600 op stop timeout=3600 \
op monitor interval=61 role="Slave" timeout=700 \
op monitor interval=59 role="Master" timeout=700 \
op promote timeout=3600 op demote timeout=3600 \
master notify=true clone-max=2 clone-node-max=1 interleave=true

sudo pcs resource create vip_<b>HN1</b>_<b>03</b> IPaddr2 ip="<b>10.0.0.13</b>"
sudo pcs resource create nc_<b>HN1</b>_<b>03</b> azure-lb port=625<b>03</b>
sudo pcs resource group add g_ip_<b>HN1</b>_<b>03</b> nc_<b>HN1</b>_<b>03</b> vip_<b>HN1</b>_<b>03</b>

sudo pcs constraint order SAPHanaTopology_<b>HN1</b>_<b>03</b>-clone then SAPHana_<b>HN1</b>_<b>03</b>-master symmetrical=false
sudo pcs constraint colocation add g_ip_<b>HN1</b>_<b>03</b> with master SAPHana_<b>HN1</b>_<b>03</b>-master 4000

sudo pcs property set maintenance-mode=false
</code></pre>

При создании кластера на **RHEL 8. x** используйте следующие команды:  

<pre><code># Replace the bold string with your instance number, HANA system ID, and the front-end IP address of the Azure load balancer.
#
sudo pcs resource create SAPHana_<b>HN1</b>_<b>03</b> SAPHana SID=<b>HN1</b> InstanceNumber=<b>03</b> PREFER_SITE_TAKEOVER=true DUPLICATE_PRIMARY_TIMEOUT=7200 AUTOMATED_REGISTER=false \
op start timeout=3600 op stop timeout=3600 \
op monitor interval=61 role="Slave" timeout=700 \
op monitor interval=59 role="Master" timeout=700 \
op promote timeout=3600 op demote timeout=3600 \
promotable meta notify=true clone-max=2 clone-node-max=1 interleave=true

sudo pcs resource create vip_<b>HN1</b>_<b>03</b> IPaddr2 ip="<b>10.0.0.13</b>"
sudo pcs resource create nc_<b>HN1</b>_<b>03</b> azure-lb port=625<b>03</b>
sudo pcs resource group add g_ip_<b>HN1</b>_<b>03</b> nc_<b>HN1</b>_<b>03</b> vip_<b>HN1</b>_<b>03</b>

sudo pcs constraint order SAPHanaTopology_<b>HN1</b>_<b>03</b>-clone then SAPHana_<b>HN1</b>_<b>03</b>-clone symmetrical=false
sudo pcs constraint colocation add g_ip_<b>HN1</b>_<b>03</b> with master SAPHana_<b>HN1</b>_<b>03</b>-clone 4000

sudo pcs property set maintenance-mode=false
</code></pre>

Убедитесь, что состоянию кластера соответствует значение ОК и все ресурсы запущены. Не важно, на каком узле выполняются ресурсы.

> [!NOTE]
> Время ожидания в приведенной выше конфигурации указано только для примера. Его нужно настроить для конкретной установки HANA. Например, время ожидания при запуске лучше увеличить, если запуск базы данных SAP HANA выполняется долго.  

<pre><code>sudo pcs status

# Online: [ hn1-db-0 hn1-db-1 ]
#
# Full list of resources:
#
# azure_fence     (stonith:fence_azure_arm):      Started hn1-db-0
#  Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
#      Started: [ hn1-db-0 hn1-db-1 ]
#  Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
#      Masters: [ hn1-db-0 ]
#      Slaves: [ hn1-db-1 ]
#  Resource Group: g_ip_HN1_03
#      nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
#      vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>


## <a name="configure-hana-activeread-enabled-system-replication-in-pacemaker-cluster"></a>Настройка репликации системы HANA Active/Read с поддержкой в кластере Pacemaker

Начиная с SAP HANA 2,0 SPS 01 SAP позволяет выполнять активные и доступные для чтения программы установки для репликации системы SAP HANA, где вторичные системы SAP HANA репликации системы можно использовать для интенсивных рабочих нагрузок чтения. Для поддержки такой настройки в кластере требуется второй виртуальный IP-адрес, который позволяет клиентам получать доступ к дополнительной базе данных SAP HANA, доступной для чтения. Чтобы гарантировать, что вторичный сайт репликации по-прежнему будет доступен после перенаправление, кластеру необходимо переместить виртуальный IP-адрес, сопоставленный с дополнительным ресурсом SAPHana.

В этом разделе описаны дополнительные действия, которые необходимо выполнить для управления репликацией системы HANA Active/Read с поддержкой в кластере с высоким уровнем доступности Red Hat с вторым виртуальным IP-адресом.    

Прежде чем продолжить, убедитесь, что вы полностью настроили кластер с высоким уровнем доступности Red Hat, управляющий SAP HANA базой данных, как описано в приведенных выше сегментах документации.  

![SAP HANA высокий уровень доступности с помощью вторичной реплики с поддержкой чтения](./media/sap-hana-high-availability/ha-hana-read-enabled-secondary.png)

### <a name="additional-setup-in-azure-load-balancer-for-activeread-enabled-setup"></a>Дополнительная настройка в подсистеме балансировки нагрузки Azure для активных и доступных для чтения настроек

Чтобы продолжить выполнение дополнительных действий по подготовке второго виртуального IP-адреса, убедитесь, что вы настроили Azure Load Balancer, как описано в разделе [Развертывание вручную](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-high-availability-rhel#manual-deployment) .

1. Для **стандартного** балансировщика нагрузки выполните следующие дополнительные действия в той же подсистеме балансировки нагрузки, которая была создана в предыдущем разделе.

   а. Создайте второй интерфейсный пул IP-адресов: 

   - Откройте подсистему балансировки нагрузки, выберите **пул интерфейсных IP-адресов** и щелкните **Добавить**.
   - Введите имя второго внешнего интерфейса пула IP-адресов (например, **Hana-секондарип**).
   - Задайте для **назначения** значение **static** и введите IP-адрес (например, **10.0.0.14**).
   - Щелкните **ОК**.
   - Когда пул интерфейсных IP-адресов будет создан, запишите его IP-адрес.

   b. Создайте зонд работоспособности.

   - Откройте подсистему балансировки нагрузки, выберите **Зонды работоспособности** и щелкните **Добавить**.
   - Введите имя новой проверки работоспособности (например, **Hana-секондарихп**).
   - Выберите **TCP** в качестве протокола и порта **62603**. Сохраните значение "5" для параметра **Интервал** и значение "2" для параметра **Порог состояния неработоспособности**.
   - Щелкните **ОК**.

   c. Теперь создайте правила балансировки нагрузки.

   - Откройте подсистему балансировки нагрузки, выберите **Правила балансировки нагрузки** щелкните **Добавить**.
   - Введите имя нового правила балансировщика нагрузки (например, **Hana-секондарилб**).
   - Выберите интерфейсный IP-адрес, пул внутренних серверов и пробы работоспособности, созданные ранее (например, **Hana-секондарип**, **Hana-сервер** и **Hana-секондарихп**).
   - Выберите **Порты высокой доступности**.
   - Увеличьте **время ожидания** до 30 минут.
   - Не забудьте **включить плавающий IP-адрес**.
   - Щелкните **ОК**.

### <a name="configure-hana-activeread-enabled-system-replication"></a>Настройка репликации системы (Active/Read) с поддержкой HANA

Действия по настройке репликации системы HANA описаны в разделе [Настройка репликации системы в SAP HANA 2,0](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-high-availability-rhel#configure-sap-hana-20-system-replication) . При развертывании вторичного сценария с поддержкой чтения во время настройки репликации системы на втором узле выполните следующую команду в качестве **ханасид** ADM:

```
sapcontrol -nr 03 -function StopWait 600 10 

hdbnsutil -sr_register --remoteHost=hn1-db-0 --remoteInstance=03 --replicationMode=sync --name=SITE2 --operationMode=logreplay_readaccess 
```

### <a name="adding-a-secondary-virtual-ip-address-resource-for-an-activeread-enabled-setup"></a>Добавление дополнительного ресурса виртуального IP-адреса для установки с активной или доступной для чтения конфигурацией

Второй виртуальный IP-адрес и соответствующее ограничение совместного размещения можно настроить с помощью следующих команд:

```
pcs property set maintenance-mode=true

pcs resource create secvip_HN1_03 ocf:heartbeat:IPaddr2 ip="10.40.0.16"

pcs resource create secnc_HN1_03 ocf:heartbeat:azure-lb port=62603

pcs resource group add g_secip_HN1_03 secnc_HN1_03 secvip_HN1_03

RHEL 8.x: 
pcs constraint colocation add g_secip_HN1_03 with slave SAPHana_HN1_03-clone 4000
RHEL 7.x:
pcs constraint colocation add g_secip_HN1_03 with slave SAPHana_HN1_03-master 4000

pcs property set maintenance-mode=false
```
Убедитесь, что состоянию кластера соответствует значение ОК и все ресурсы запущены. Второй виртуальный IP-адрес будет выполняться на вторичном сайте вместе с дополнительным ресурсом SAPHana.

```
sudo pcs status

# Online: [ hn1-db-0 hn1-db-1 ]
#
# Full List of Resources:
#   rsc_hdb_azr_agt     (stonith:fence_azure_arm):      Started hn1-db-0
#   Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]:
#     Started: [ hn1-db-0 hn1-db-1 ]
#   Clone Set: SAPHana_HN1_03-clone [SAPHana_HN1_03] (promotable):
#     Masters: [ hn1-db-0 ]
#     Slaves: [ hn1-db-1 ]
#   Resource Group: g_ip_HN1_03:
#     nc_HN1_03         (ocf::heartbeat:azure-lb):      Started hn1-db-0
#     vip_HN1_03        (ocf::heartbeat:IPaddr2):       Started hn1-db-0
#   Resource Group: g_secip_HN1_03:
#     secnc_HN1_03      (ocf::heartbeat:azure-lb):      Started hn1-db-1
#     secvip_HN1_03     (ocf::heartbeat:IPaddr2):       Started hn1-db-1
```

В следующем разделе можно найти типичный набор тестов отработки отказа для выполнения.

Учитывайте второе поведение виртуального IP-адреса при тестировании кластера HANA, настроенного с помощью вторичного сервера с поддержкой чтения:

1. При переносе ресурса кластера **SAPHana_HN1_HDB03** в **HN1-DB-1** второй виртуальный IP-адрес перейдет на другой сервер **HN1-DB-0**. Если вы настроили AUTOMATED_REGISTER = "false" и репликация системы HANA не зарегистрирована автоматически, второй виртуальный IP-адрес будет работать на **HN1-DB-0,** так как сервер доступен и службы кластеров находятся в сети.  

2. При тестировании сбоя сервера второй виртуальный IP-ресурс (**rsc_secip_HN1_HDB03**) и ресурс порта балансировщика нагрузки Azure (**rsc_secnc_HN1_HDB03**) будут выполняться на сервере-источнике вместе с основными ресурсами виртуального IP-адреса.  Пока сервер-получатель не работает, приложения, подключенные к базе данных HANA с поддержкой чтения, будут подключаться к первичной базе данных HANA. Предполагается, что не нужно, чтобы приложения, подключенные к базе данных HANA с поддержкой чтения, были недоступны во время недоступности сервера-получателя.

3. Если сервер-получатель доступен и службы кластеров находятся в сети, второй виртуальный IP-адрес и порты будут автоматически перемещены на сервер-получатель, даже если репликация системы HANA не может быть зарегистрирована как вторичная. Перед запуском служб кластеров на этом сервере необходимо зарегистрировать вторичную базу данных HANA как доступную для чтения. Ресурс кластера экземпляра HANA можно настроить для автоматической регистрации базы-получателя с параметром AUTOMATED_REGISTER = true.
   
4. Во время отработки отказа и отката существующие подключения для приложений, использующие второй виртуальный IP-адрес для подключения к базе данных HANA, могут быть прерваны.  

## <a name="test-the-cluster-setup"></a>Проверка настройки кластера

В этом разделе описано, как проверить настроенную систему. Перед началом теста убедитесь в том, что в Pacemaker не зарегистрировано действий, завершившихся ошибкой (с помощью команды pcs status), нет непредвиденных ограничений на расположение (например, лишних элементов после теста миграции) и HANA находится в синхронизированном состоянии (например, с помощью systemReplicationStatus):

<pre><code>[root@hn1-db-0 ~]# sudo su - hn1adm -c "python /usr/sap/HN1/HDB03/exe/python_support/systemReplicationStatus.py"
</code></pre>

### <a name="test-the-migration"></a>Проверка миграции

Состояние ресурсов перед запуском теста:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-0 ]
    Slaves: [ hn1-db-1 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

Чтобы выполнить миграцию главного узла SAP HANA, выполните следующую команду.

<pre><code># On RHEL <b>7.x</b> 
[root@hn1-db-0 ~]# pcs resource move SAPHana_HN1_03-master
# On RHEL <b>8.x</b>
[root@hn1-db-0 ~]# pcs resource move SAPHana_HN1_03-clone --master
</code></pre>

Если вы указали `AUTOMATED_REGISTER="false"`, эта команда перенесет главный узел SAP HANA и группу, которая содержит виртуальный IP-адрес, в узел hn1-db-1.

По окончании миграции выходные данные команды sudo pcs status будут выглядеть так:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-1 ]
    Stopped: [ hn1-db-0 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-1
</code></pre>

Ресурс SAP HANA в узле hn1-db-0 остановлен. В этом случае настройте экземпляр HANA в качестве вторичного узла, выполнив следующую команду.

<pre><code>[root@hn1-db-0 ~]# su - hn1adm

# Stop the HANA instance just in case it is running
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> sapcontrol -nr 03 -function StopWait 600 10
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=hn1-db-1 --remoteInstance=03 --replicationMod
e=sync --name=SITE1
</code></pre>

При миграции создаются дополнительные ограничения расположения, которые следует удалить:

<pre><code># Switch back to root
exit
[root@hn1-db-0 ~]# pcs resource clear SAPHana_HN1_03-master
</code></pre>

Для отслеживания состояния ресурса HANA используйте команду pcs status. Когда HANA запущена на hn1-db-0, эта команда должна возвращать следующий результат:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-1 ]
    Slaves: [ hn1-db-0 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-1
</code></pre>

### <a name="test-the-azure-fencing-agent"></a>Проверка агента ограждения Azure

> [!NOTE]
> Эта статья содержит ссылки на термин « *Ведомый*» термин, который корпорация Майкрософт больше не использует. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.  

Состояние ресурсов перед запуском теста:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-1 ]
    Slaves: [ hn1-db-0 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-1
</code></pre>

Чтобы проверить конфигурацию агента ограждения, отключите сетевой интерфейс в главном узле SAP HANA.
Способ симуляции сетевого сбоя см. в [статье базы знаний Red Hat 79523](https://access.redhat.com/solutions/79523). В этом примере мы используем сценарий net_breaker для полной блокировки доступа к сети.

<pre><code>[root@hn1-db-1 ~]# sh ./net_breaker.sh BreakCommCmd 10.0.0.6
</code></pre>

Виртуальная машина должна быть перезагружена или остановлена, в зависимости от настройки кластера.
Если для параметра `stonith-action` задано значение Off (отключено), то виртуальная машина будет остановлена, а ресурсы перенесены на работающую виртуальную машину.

Если вы задали `AUTOMATED_REGISTER="false"`, то после повторного запуска виртуальной машины ресурс SAP HANA не будет запущен в режиме вторичного узла. В этом случае настройте экземпляр HANA в качестве вторичного узла, выполнив следующую команду.

<pre><code>su - <b>hn1</b>adm

# Stop the HANA instance just in case it is running
hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> sapcontrol -nr <b>03</b> -function StopWait 600 10
hn1adm@hn1-db-1:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=<b>hn1-db-0</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE2</b>

# Switch back to root and clean up the failed state
exit
# On RHEL <b>7.x</b>
[root@hn1-db-1 ~]# pcs resource cleanup SAPHana_HN1_03-master
# On RHEL <b>8.x</b>
[root@hn1-db-1 ~]# pcs resource cleanup SAPHana_HN1_03 node=&lt;hostname on which the resource needs to be cleaned&gt;
</code></pre>

Состояние ресурсов после теста:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-0 ]
    Slaves: [ hn1-db-1 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

### <a name="test-a-manual-failover"></a>Тестирование отработки отказа вручную

Состояние ресурсов перед запуском теста:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-0 ]
    Slaves: [ hn1-db-1 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

Чтобы проверить отработку отказа вручную, остановите кластер в узле hn1-db-0:

<pre><code>[root@hn1-db-0 ~]# pcs cluster stop
</code></pre>

После отработки отказа можно снова запустить кластер. Если вы указали `AUTOMATED_REGISTER="false"`, ресурс SAP HANA на узле hn1-db-0 не будет запущен в роли вторичного узла. В этом случае настройте экземпляр HANA в качестве вторичного узла, выполнив следующую команду.

<pre><code>[root@hn1-db-0 ~]# pcs cluster start
[root@hn1-db-0 ~]# su - hn1adm

# Stop the HANA instance just in case it is running
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> sapcontrol -nr 03 -function StopWait 600 10
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> hdbnsutil -sr_register --remoteHost=<b>hn1-db-1</b> --remoteInstance=<b>03</b> --replicationMode=sync --name=<b>SITE1</b>

# Switch back to root and clean up the failed state
hn1adm@hn1-db-0:/usr/sap/HN1/HDB03> exit
# On RHEL <b>7.x</b>
[root@hn1-db-1 ~]# pcs resource cleanup SAPHana_HN1_03-master
# On RHEL <b>8.x</b>
[root@hn1-db-1 ~]# pcs resource cleanup SAPHana_HN1_03 node=&lt;hostname on which the resource needs to be cleaned&gt;
</code></pre>

Состояние ресурсов после теста:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-1 ]
     Slaves: [ hn1-db-0 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-1
</code></pre>

### <a name="test-a-manual-failover"></a>Тестирование отработки отказа вручную

Состояние ресурсов перед запуском теста:

<pre><code>Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hn1-db-0 hn1-db-1 ]
Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hn1-db-0 ]
    Slaves: [ hn1-db-1 ]
Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hn1-db-0
    vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hn1-db-0
</code></pre>

Чтобы проверить отработку отказа вручную, остановите кластер в узле hn1-db-0:

<pre><code>[root@hn1-db-0 ~]# pcs cluster stop
</code></pre>


## <a name="next-steps"></a>Дальнейшие действия

* [Планирование и реализация виртуальных машин Azure для SAP][planning-guide]
* [Развертывание виртуальных машин Azure для SAP NetWeaver][deployment-guide]
* [Развертывание СУБД виртуальных машин Azure для SAP][dbms-guide]
* [Конфигурации хранилища виртуальных машин SAP HANA](./hana-vm-operations-storage.md)