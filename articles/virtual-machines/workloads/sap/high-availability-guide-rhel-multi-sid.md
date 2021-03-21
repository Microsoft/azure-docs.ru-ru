---
title: Высокий уровень доступности виртуальных машин Azure для SAP NW в RHEL Multi-SID | Документация Майкрософт
description: Установите высокий уровень доступности для SAP NW на виртуальных машинах Azure (VM) RHEL Multi-SID.
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
ms.date: 01/11/2021
ms.author: radeltch
ms.openlocfilehash: ec2121754a24a44288c158e630a4e84219c744e3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101676915"
---
# <a name="high-availability-for-sap-netweaver-on-azure-vms-on-red-hat-enterprise-linux-for-sap-applications-multi-sid-guide"></a>Высокий уровень доступности SAP NetWeaver на виртуальных машинах Azure на Red Hat Enterprise Linux для многоид безопасности приложений SAP

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-azure-doc]:https://docs.microsoft.com/azure/azure-netapp-files/
[anf-avail-matrix]:https://azure.microsoft.com/global-infrastructure/services/?products=storage&regions=all
[anf-register]:https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register
[anf-sap-applications-azure]:https://www.netapp.com/us/media/tr-4746.pdf

[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2009879]:https://launchpad.support.sap.com/#/notes/2009879
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[1410736]:https://launchpad.support.sap.com/#/notes/1410736

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[template-multisid-xscs]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability-rhel.md
[glusterfs-ha]:high-availability-guide-rhel-glusterfs.md

В этой статье описывается, как развернуть несколько высокодоступных систем SAP NetWeaver (то есть несколько идентификаторов безопасности) в кластере с двумя узлами на виртуальных машинах Azure с Red Hat Enterprise Linux для приложений SAP.  

В примерах конфигураций, команд установки и т. д. три системы SAP NetWeaver 7,50 развертываются в одном кластере с высоким уровнем доступности в двух узлах. Идентификаторы безопасности систем SAP:
* **NW1**: экземпляр ASCS с номером **00** и именем виртуального узла **msnw1ascs**; ERS номер экземпляра **02** и имя виртуального узла **msnw1ers**.  
* **NW2.**: экземпляр ASCS с номером **10** и виртуальным именем узла **msnw2ascs**; Экземпляр ERS номер **12** и имя виртуального узла **msnw2ers**.  
* **NW3**: экземпляр ASCS с номером **20** и виртуальным именем узла **msnw3ascs**; Экземпляр ERS с номером **22** и именем виртуального узла **msnw3ers**.  

В этой статье не рассматривается уровень базы данных и развертывание общих папок SAP NFS. В примерах, приведенных в этой статье, мы используем [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md)  Volume **сапмсид** для общих папок NFS, предполагая, что том уже развернут. Мы также предполагаю, что Azure NetApp Files том развертывается с помощью протокола NFSv3 и что существуют следующие пути к ресурсам кластера для экземпляров ASCS и ERS SAP Systems NW1, NW2. и NW3:  

* Volume Сапмсид (nfs://10.42.0.4/sapmnt<b>NW1</b>)
* Volume Сапмсид (nfs://10.42.0.4/usrsap<b>NW1</b>ASCS)
* Volume Сапмсид (nfs://10.42.0.4/usrsap<b>NW1</b>sys)
* Volume Сапмсид (nfs://10.42.0.4/usrsap<b>NW1</b>ERS)
* Volume Сапмсид (nfs://10.42.0.4/sapmnt<b>NW2.</b>)
* Volume Сапмсид (nfs://10.42.0.4/usrsap<b>NW2.</b>ASCS)
* Volume Сапмсид (nfs://10.42.0.4/usrsap<b>NW2.</b>sys)
* Volume Сапмсид (nfs://10.42.0.4/usrsap<b>NW2.</b>ERS)
* Volume Сапмсид (nfs://10.42.0.4/sapmnt<b>NW3</b>)
* Volume Сапмсид (nfs://10.42.0.4/usrsap<b>NW3</b>ASCS)
* Volume Сапмсид (nfs://10.42.0.4/usrsap<b>NW3</b>sys)
* Volume Сапмсид (nfs://10.42.0.4/usrsap<b>NW3</b>ERS)

Прежде чем начать, ознакомьтесь со следующими примечаниями и документацией по SAP:

* примечание к SAP [1928533], которое содержит:
  * список размеров виртуальных машин Azure, поддерживаемых для развертывания ПО SAP;
  * важные сведения о доступных ресурсах для каждого размера виртуальной машины Azure;
  * сведения о поддерживаемом программном обеспечении SAP и сочетаниях операционных систем и баз данных;
  * сведения о требуемой версии ядра SAP для Windows и Linux в Microsoft Azure.
* [Документация по Azure NetApp Files][anf-azure-doc]
* примечание к SAP [2015553], в котором описываются предварительные требования к SAP при развертывании программного обеспечения SAP в Azure;
* примечание к SAP [2002167], содержащее рекомендуемые параметры ОС для Red Hat Enterprise Linux;
* примечание к SAP [2009879], содержащее рекомендации по SAP HANA для Red Hat Enterprise Linux;
* примечание к SAP [2178632], содержащее подробные сведения обо всех доступных метриках мониторинга для SAP в Azure;
* примечание к SAP [2191498], содержащее сведения о необходимой версии агента SAP Host Agent для Linux в Azure;
* примечание к SAP [2243692], содержащее сведения о лицензировании SAP в Linux в Azure;
* примечание к SAP [1999351], содержащее дополнительные сведения об устранении неполадок, связанных с расширением для расширенного мониторинга Azure для SAP;
* [вики-сайт сообщества SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes), содержащий все необходимые примечания к SAP для Linux;
* [SAP NetWeaver на виртуальных машинах Linux. Руководство по планированию и внедрению][planning-guide]
* [Развертывание программного обеспечения SAP на виртуальных машинах Linux в Azure][deployment-guide]
* [Общие сведения о развертывании СУБД на виртуальных машинах Azure для рабочих нагрузок SAP][dbms-guide]
* [SAP Netweaver в кластере pacemaker](https://access.redhat.com/articles/3150081)
* Общая документация по RHEL
  * [Общие сведения о надстройке для обеспечения высокой доступности](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [Администрирование надстройки для обеспечения высокой доступности](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [Справочник по надстройке для обеспечения высокой доступности](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
  * [Настройка ASCS/ERS для SAP Netweaver с автономными ресурсами в RHEL 7.5](https://access.redhat.com/articles/3569681)
  * [Настройка SAP S/4HANA ASCS/ERS с автономным сервером очереди 2 (ENSA2) в Pacemaker на RHEL](https://access.redhat.com/articles/3974941)
* Документация по RHEL для Azure:
  * [Политики поддержки для кластеров высокой доступности RHEL — виртуальные машины Microsoft Azure как члены кластера](https://access.redhat.com/articles/3131341)
  * [Установка и настройка кластера высокой доступности Red Hat Enterprise Linux 7.4 (и более поздних версий) в Microsoft Azure](https://access.redhat.com/articles/3252491)
* [Приложения NetApp SAP в Microsoft Azure. Использование Azure NetApp Files][anf-sap-applications-azure]

## <a name="overview"></a>Обзор

Виртуальные машины, участвующие в кластере, должны иметь размер, чтобы иметь возможность выполнять все ресурсы, в случае отработки отказа. Каждый идентификатор безопасности SAP может отключаться при сбое независимо друг от друга в высокодоступном кластере с несколькими ИД безопасности.  

Для обеспечения высокого уровня доступности SAP NetWeaver требует высокой доступности общих ресурсов. В этой документации представлены примеры с общими ресурсами SAP, развернутыми на [томах Azure NETAPP Files NFS](../../../azure-netapp-files/azure-netapp-files-create-volumes.md). Также можно разместить общие папки в [кластере глустерфс](./high-availability-guide-rhel-glusterfs.md)высокой доступности, который может использоваться несколькими системами SAP.  

![Общие сведения о высоком уровне доступности SAP NetWeaver](./media/high-availability-guide-rhel/ha-rhel-multi-sid.png)

> [!IMPORTANT]
> Поддержка кластеризации SAP ASCS/ERS с несколькими ИД безопасности с помощью Red Hat Linux в качестве гостевой операционной системы на виртуальных машинах Azure ограничена **пятью** идентификаторами безопасности SAP в одном кластере. Каждый новый идентификатор безопасности повышает сложность. **Не поддерживается** сочетание сервера репликации очереди SAP 1 и постановки в очередь сервера репликации 2 в одном кластере. Кластеризация с несколькими ИД безопасности описывает установку нескольких экземпляров SAP ASCS/ERS с разными идентификаторами безопасности в одном кластере Pacemaker. В настоящее время кластеризация с несколькими ИД безопасности поддерживается только для ASCS/ERS.  

> [!TIP]
> Кластеризация SAP ASCS/ERS с несколькими ИД безопасности — это решение с более высокой степенью сложности. Реализация реализуется более сложно. Он также включает в себя более высокие усилия по администрированию при выполнении действий по обслуживанию (например, по исправлению ОС). Прежде чем начать фактическую реализацию, уделите время тщательному планированию развертывания и всех связанных компонентов, например виртуальных машин, подключений NFS, виртуальных IP-адресов, конфигураций подсистемы балансировки нагрузки и т. д.  

SAP NetWeaver ASCS, SAP NetWeaver SCS и SAP NetWeaver ERS используют виртуальные имена узлов и виртуальные IP-адреса. Балансировщику нагрузки в Azure нужен виртуальный IP-адрес. Мы рекомендуем [Load Balancer (цен. категория "Стандартный")](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md).  

В следующем списке показана конфигурация подсистемы балансировки нагрузки (A) SCS и ERS для этого кластера с несколькими идентификаторами безопасности с тремя системами SAP. Вам потребуется отдельный интерфейсный IP-адрес, зонды работоспособности и правила балансировки нагрузки для каждого экземпляра ASCS и ERS для каждого из идентификаторов безопасности. Назначьте всем виртуальным машинам, которые являются частью кластера ASCS/ASCS, один серверный пул одного ILB.  

### <a name="ascs"></a>(A)SCS

* Конфигурация внешнего интерфейса:
  * IP-адрес для NW1:10.3.1.50
  * IP-адрес для NW2.: 10.3.1.52
  * IP-адрес для NW3:10.3.1.54

* Порты пробы
  * Порт 620 <strong> &lt; Nr &gt;</strong>, поэтому для пробных портов NW1, NW2. и NW3 620 **00**, 620 **10** и 620 **20**
* Правила балансировки нагрузки — создайте по одному для каждого экземпляра, то есть NW1/ASCS, NW2./ASCS и NW3/ASCS.
  * Если используется Load Balancer (цен. категория "Стандартный"), выберите **порты высокой доступности**.
  * Если используется Load Balancer (цен. категория "Базовый"), создайте правила балансировки нагрузки для следующих портов.
    * 32<strong>&lt;nr&gt;</strong> TCP;
    * 36<strong>&lt;nr&gt;</strong> TCP;
    * 39<strong>&lt;nr&gt;</strong> TCP;
    * 81<strong>&lt;nr&gt;</strong> TCP;
    * 5<strong>&lt;nr&gt;</strong>13 TCP;
    * 5<strong>&lt;nr&gt;</strong>14 TCP;
    * 5<strong>&lt;nr&gt;</strong>16 TCP.

### <a name="ers"></a>ERS

* Конфигурация внешнего интерфейса:
  * IP-адрес для NW1 10.3.1.51
  * IP-адрес для NW2.10.3.1.53
  * IP-адрес для NW3 10.3.1.55

* Порт пробы:
  * Порт 621 <strong> &lt; Nr &gt;</strong>, поэтому для портов пробы NW1, NW2. и N3 621 **02**, 621 **12** и 621 **22**
* Правила балансировки нагрузки — создайте по одному для каждого экземпляра, то есть NW1/ERS, NW2./ERS и NW3/ERS.
  * Если используется Load Balancer (цен. категория "Стандартный"), выберите **порты высокой доступности**.
  * Если используется Load Balancer (цен. категория "Базовый"), создайте правила балансировки нагрузки для следующих портов.
    * 32<strong>&lt;nr&gt;</strong> TCP;
    * 33<strong>&lt;nr&gt;</strong> TCP;
    * 5<strong>&lt;nr&gt;</strong>13 TCP;
    * 5<strong>&lt;nr&gt;</strong>14 TCP;
    * 5<strong>&lt;nr&gt;</strong>16 TCP.

* Конфигурация серверной части:
  * подключена к основным сетевым интерфейсам всех виртуальных машин, которые должны быть частью кластера (A)SCS/ERS.

> [!IMPORTANT]
> Плавающий IP-адрес не поддерживается для вторичной IP-конфигурации NIC в сценариях балансировки нагрузки. Дополнительные сведения см. в статье [ограничения балансировщика нагрузки Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Если для виртуальной машины требуется дополнительный IP-адрес, разверните вторую сетевую карту.  

> [!Note]
> Если в серверный пул внутреннего (без общедоступного IP-адреса) Azure Load Balancer ценовой категории "Стандартный" помещаются виртуальные машины без общедоступных IP-адресов, у них не будет исходящего подключения к Интернету без дополнительной настройки, разрешающей маршрутизацию к общедоступным конечным точкам. Подробные сведения о такой настройке см. в статье [Подключение к общедоступной конечной точке для виртуальных машин с помощью Azure Load Balancer (цен. категория "Стандартный") в сценариях обеспечения высокого уровня доступности SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

> [!IMPORTANT]
> Не включайте метки времени TCP на виртуальных машинах Azure, размещенных за Azure Load Balancer. Включение меток времени TCP помешает работе проб работоспособности. Установите для параметра **net.ipv4.tcp_timestamps** значение **0**. Дополнительные сведения см. в статье [Пробы работоспособности Load Balancer](../../../load-balancer/load-balancer-custom-probe-overview.md).

## <a name="sap-shares"></a>Общие папки SAP

SAP NetWeaver требует наличия общего хранилища для транспорта, каталога профиля и т. д. Для высокодоступной системы SAP важно иметь высокодоступные общие папки. Вам потребуется выбрать архитектуру для общих ресурсов SAP. Один из вариантов — развернуть общие папки на [томах Azure NETAPP Files NFS](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).  С Azure NetApp Files вы получите встроенную высокую доступность общих ресурсов SAP NFS.

Кроме того, можно создать [глустерфс на виртуальных машинах Azure на Red Hat Enterprise Linux для SAP NetWeaver](./high-availability-guide-rhel-glusterfs.md), которые могут совместно использоваться несколькими системами SAP. 

## <a name="deploy-the-first-sap-system-in-the-cluster"></a>Развертывание первой системы SAP в кластере

Теперь, когда вы решили архитектуру общих ресурсов SAP, разверните первую систему SAP в кластере, следуя соответствующей документации.

* При использовании томов Azure NetApp Files NFS используйте [высокий уровень доступности виртуальных машин Azure для SAP NetWeaver на Red Hat Enterprise Linux с Azure NetApp Files для приложений SAP](./high-availability-guide-rhel-netapp-files.md) .  
* При использовании кластера Глустерфс выполните [глустерфс на виртуальных машинах Azure на Red Hat Enterprise Linux для SAP NetWeaver](./high-availability-guide-rhel-glusterfs.md).  

Перечисленные выше документы помогут вам выполнить необходимые действия по подготовке необходимой инфраструктуры, сборке кластера и подготовке ОС к запуску приложения SAP.  

> [!TIP]
> Прежде чем добавлять в кластер дополнительные идентификаторы безопасности SAP, всегда проверяйте функции отработки отказа кластера после развертывания первой системы. Таким образом вы узнаете, что функциональность кластера работает, прежде чем добавлять в кластер сложность дополнительных систем SAP.   

## <a name="deploy-additional-sap-systems-in-the-cluster"></a>Развертывание дополнительных систем SAP в кластере

В этом примере предполагается, что система **NW1** уже развернута в кластере. Мы покажем, как развертывать в кластере SAP Systems **NW2.** and **NW3**. 

Ниже приведены элементы с префиксами: **[A]**  — применяется ко всем узлам, **[1**] — применяется только к узлу 1, **[2]**  — применяется только к узлу 2.

### <a name="prerequisites"></a>Предварительные условия 

> [!IMPORTANT]
> Прежде чем следовать инструкциям по развертыванию дополнительных систем SAP в кластере, следуйте инструкциям по развертыванию первой системы SAP в кластере, так как существуют шаги, необходимые только при первом развертывании системы.  

В этой документации предполагается, что:
* Кластер Pacemaker уже настроен и запущен.  
* По крайней мере одна система SAP (экземпляр ASCS/ERS) уже развернута и выполняется в кластере.  
* Функции отработки отказа кластера были протестированы.  
* Разворачиваются общие папки NFS для всех систем SAP.  

### <a name="prepare-for-sap-netweaver-installation"></a>Подготовка к установке SAP NetWeaver

1. Добавьте конфигурацию для новой развернутой системы (т. е. **NW2.**, **NW3**) в существующую Azure Load Balancer, следуя инструкциям по [развертыванию Azure Load Balancer вручную с помощью портал Azure](./high-availability-guide-rhel-netapp-files.md#deploy-linux-manually-via-azure-portal). Настройте IP-адреса, порты пробы работоспособности, правила балансировки нагрузки для вашей конфигурации.  

2. **[A]** Настройка разрешения имен для дополнительных систем SAP. Можно либо использовать DNS-сервер, либо изменить `/etc/hosts` на всех узлах. В этом примере показано, как использовать `/etc/hosts` файл.  Адаптируйте IP-адреса и имена узлов к вашей среде. 

    ```
    sudo vi /etc/hosts
    # IP address of the load balancer frontend configuration for NW2 ASCS
    10.3.1.52 msnw2ascs
    # IP address of the load balancer frontend configuration for NW3 ASCS
    10.3.1.54 msnw3ascs
    # IP address of the load balancer frontend configuration for NW2 ERS
    10.3.1.53 msnw2ers
    # IP address of the load balancer frontend configuration for NW3 ERS
    10.3.1.55 msnw3ers
   ```

3. **[A]** Создайте общие каталоги для дополнительных **Nw2.** и **NW3** SAP Systems, которые развертываются в кластере. 

    ```
    sudo mkdir -p /sapmnt/NW2
    sudo mkdir -p /usr/sap/NW2/SYS
    sudo mkdir -p /usr/sap/NW2/ASCS10
    sudo mkdir -p /usr/sap/NW2/ERS12
    sudo mkdir -p /sapmnt/NW3
    sudo mkdir -p /usr/sap/NW3/SYS
    sudo mkdir -p /usr/sap/NW3/ASCS20
    sudo mkdir -p /usr/sap/NW3/ERS22

    
    sudo chattr +i /sapmnt/NW2
    sudo chattr +i /usr/sap/NW2/SYS
    sudo chattr +i /usr/sap/NW2/ASCS10
    sudo chattr +i /usr/sap/NW2/ERS12
    sudo chattr +i /sapmnt/NW3
    sudo chattr +i /usr/sap/NW3/SYS
    sudo chattr +i /usr/sap/NW3/ASCS20
    sudo chattr +i /usr/sap/NW3/ERS22
   ```

4. **[A]** добавьте записи подключения для файловых систем/Сапмнт/СИД и/УСР/САП/СИД/СИС для дополнительных систем SAP, которые развертываются в кластере. В этом примере **NW2.** и **NW3**.  

   Обновите файл `/etc/fstab` с помощью файловых систем для дополнительных систем SAP, которые развертываются в кластере.  

   * Если используется Azure NetApp Files, следуйте приведенным [здесь](./high-availability-guide-rhel-netapp-files.md#prepare-for-sap-netweaver-installation) инструкциям.  
   * Если используется кластер Глустерфс, следуйте приведенным [здесь](./high-availability-guide-rhel.md#prepare-for-sap-netweaver-installation) инструкциям.  

### <a name="install-ascs--ers"></a>Установка ASCS и ERS

1. Создайте виртуальные IP-адреса и ресурсы кластера пробы работоспособности для экземпляров ASCS дополнительных систем SAP, которые развертываются в кластере. Приведенный ниже пример предназначен для **NW2.** и **NW3** ASCS с использованием NFS на Azure NetApp Files томах с протоколом NFSv3.  

    ```
    sudo pcs resource create fs_NW2_ASCS Filesystem device='10.42.0.4:/sapMSIDR/usrsapNW2ascs' \
    directory='/usr/sap/NW2/ASCS10' fstype='nfs' force_unmount=safe \
    op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-NW2_ASCS

    sudo pcs resource create vip_NW2_ASCS IPaddr2 \
    ip=10.3.1.52 cidr_netmask=24 \
     --group g-NW2_ASCS
    
    sudo pcs resource create nc_NW2_ASCS azure-lb port=62010 \
     --group g-NW2_ASCS

    sudo pcs resource create fs_NW3_ASCS Filesystem device='10.42.0.4:/sapMSIDR/usrsapNW3ascs' \
    directory='/usr/sap/NW3/ASCS20' fstype='nfs' force_unmount=safe \
    op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
    --group g-NW3_ASCS

    sudo pcs resource create vip_NW3_ASCS IPaddr2 \
    ip=10.3.1.54 cidr_netmask=24 \
    --group g-NW3_ASCS

    sudo pcs resource create nc_NW3_ASCS azure-lb port=62020 \
    --group g-NW3_ASCS
    ```

   Убедитесь, что состояние кластера — ОК, и все ресурсы запущены. Не важно, на каком узле выполняются ресурсы.  

2. **[1]** Установите SAP NetWeaver ASCS.  

   Установите SAP NetWeaver ASCS в качестве привилегированного, используя виртуальное имя узла, которое сопоставляется с IP-адресом внешней конфигурации балансировщика нагрузки для ASCS. Например, для System **NW2.** виртуальное имя узла — <b>msnw2ascs</b>, <b>10.3.1.52</b> и номер экземпляра, который использовался для пробы балансировщика нагрузки, например <b>10</b>. Для System **NW3** виртуальное имя узла — <b>msnw3ascs</b>, <b>10.3.1.54</b> и номер экземпляра, который использовался для пробы подсистемы балансировки нагрузки, например <b>20</b>. Запишите, на какой узел кластера установлен ASCS для каждого идентификатора безопасности SAP.  

   Чтобы разрешить непривилегированному пользователю подключаться к SAPinst, можно использовать параметр SAPINST_REMOTE_ACCESS_USER. Можно использовать параметр SAPINST_USE_HOSTNAME для установки SAP с использованием имени виртуального узла.  

    ```
    # Allow access to SWPM. This rule is not permanent. If you reboot the machine, you have to run the command again
    sudo firewall-cmd --zone=public --add-port=4237/tcp
    sudo swpm/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=virtual_hostname
    ```

   Если при установке не удается создать вложенную папку в/usr/SAP/**SID**/АСКС **instance #**, попробуйте задать для владельца значение **SID** ADM и Group to sapsys экземпляра ASCS **#** и повторите попытку.

3. **[1]** создайте виртуальный IP-адрес и ресурсы кластера проверки работоспособности для экземпляра ERS дополнительной системы SAP, которая развертывается в кластере. Приведенный ниже пример предназначен для **NW2.** и **NW3** ERS с использованием NFS на Azure NetApp Files томах с протоколом NFSv3.  

    ```
    sudo pcs resource create fs_NW2_AERS Filesystem device='10.42.0.4:/sapMSIDR/usrsapNW2ers' \
    directory='/usr/sap/NW2/ERS12' fstype='nfs' force_unmount=safe \
    op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-NW2_AERS

    sudo pcs resource create vip_NW2_AERS IPaddr2 \
    ip=10.3.1.53 cidr_netmask=24 \
     --group g-NW2_AERS

    sudo pcs resource create nc_NW2_AERS azure-lb port=62112 \
     --group g-NW2_AERS

    sudo pcs resource create fs_NW3_AERS Filesystem device='10.42.0.4:/sapMSIDR/usrsapNW3ers' \
    directory='/usr/sap/NW3/ERS22' fstype='nfs' force_unmount=safe \
    op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-NW3_AERS

    sudo pcs resource create vip_NW3_AERS IPaddr2 \
    ip=10.3.1.55 cidr_netmask=24 \
     --group g-NW3_AERS

    sudo pcs resource create nc_NW3_AERS azure-lb port=62122 \
     --group g-NW3_AERS
   ```

   Убедитесь, что состояние кластера — ОК, и все ресурсы запущены.  

   Затем убедитесь, что ресурсы только что созданной группы ERS выполняются на узле кластера, а не на узле кластера, где установлен экземпляр ASCS для одной и той же системы SAP.  Например, если NW2. ASCS был установлен на, убедитесь, `rhelmsscl1` что группа ERS NW2. запущена `rhelmsscl2` .  Вы можете перенести группу ERS NW2. в `rhelmsscl2` , выполнив следующую команду для одного из ресурсов кластера в группе: 

    ```
      pcs resource move fs_NW2_AERS rhelmsscl2
    ```

4. **[2]** Установите SAP NetWeaver ERS.

   Установите SAP NetWeaver ERS в качестве корневого узла на другом узле, используя виртуальное имя узла, которое сопоставляется с IP-адресом внешней конфигурации балансировщика нагрузки для ERS. Например, для System **NW2.** именем виртуального узла будет <b>msnw2ers</b>, <b>10.3.1.53</b> и номер экземпляра, который использовался для пробы подсистемы балансировки нагрузки, например <b>12</b>. Для System **NW3**— имя виртуального узла <b>msnw3ers</b>, <b>10.3.1.55</b> и номер экземпляра, который использовался для пробы подсистемы балансировки нагрузки, например <b>22</b>. 

   Чтобы разрешить непривилегированному пользователю подключаться к SAPinst, можно использовать параметр SAPINST_REMOTE_ACCESS_USER. Можно использовать параметр SAPINST_USE_HOSTNAME для установки SAP с использованием имени виртуального узла.  

    ```
    # Allow access to SWPM. This rule is not permanent. If you reboot the machine, you have to run the command again
    sudo firewall-cmd --zone=public --add-port=4237/tcp
    sudo swpm/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=virtual_hostname
    ```

   > [!NOTE]
   > Используйте SWPM SP 20 PL 05 или более поздней версии. Более ранние версии задают разрешения неправильно, и установка завершится с ошибкой.

   Если при установке не удается создать вложенную папку в/usr/SAP/**Nw2.**/ЕРС **instance #**, попробуйте задать для владельца **идентификатор SID** ADM, а для группы — sapsys папки ERS **instance #** и повторите попытку.

   Если необходимо перенести группу ERS только что развернутой системы SAP на другой узел кластера, не забудьте удалить ограничение расположения для группы ERS. Ограничение можно удалить, выполнив следующую команду (пример приведен для SAP Systems **NW2.** and **NW3**). Не забудьте удалить временные ограничения для того же ресурса, который использовался в команде для перемещения кластерной группы ERS.

    ```
      pcs resource clear fs_NW2_AERS
      pcs resource clear fs_NW3_AERS
    ```

5. **[1]** адаптируйте профили экземпляров ASCS/SCS и ERS для вновь установленных систем SAP. Ниже приведен пример для NW2.. Вам потребуется адаптировать профили ASCS/SCS и ERS для всех экземпляров SAP, добавленных в кластер.  
 
   * Профиль ASCS/SCS

      ```
      sudo vi /sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs
   
      # Change the restart command to a start command
      #Restart_Program_01 = local $(_EN) pf=$(_PF)
      Start_Program_01 = local $(_EN) pf=$(_PF)

      # Add the keep alive parameter, if using ENSA1
      enque/encni/set_so_keepalive = true
      ```

     Для ENSA1 и ENSA2 убедитесь, что `keepalive` Параметры ОС заданы, как описано в примечании к SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).    

   * Профиль ERS

      ```
      sudo vi /sapmnt/NW2/profile/NW2_ERS12_msnw2ers
   
      # Change the restart command to a start command
      #Restart_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
      Start_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   
      # remove Autostart from ERS profile
      # Autostart = 1
      ```

6. **[A]** Изменение файла /usr/sap/sapservices

   Чтобы предотвратить запуск экземпляров сценарием запуска сапинит, все экземпляры, управляемые Pacemaker, должны быть снабжены комментариями из `/usr/sap/sapservices` файла.  Ниже приведен пример для SAP Systems **NW2.** и **NW3**.  

   ```
    # On the node where ASCS was installed, comment out the line for the ASCS instacnes
    #LD_LIBRARY_PATH=/usr/sap/NW2/ASCS10/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/NW2/ASCS10/exe/sapstartsrv pf=/usr/sap/NW2/SYS/profile/NW2_ASCS10_msnw2ascs -D -u nw2adm
    #LD_LIBRARY_PATH=/usr/sap/NW3/ASCS20/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/NW3/ASCS20/exe/sapstartsrv pf=/usr/sap/NW3/SYS/profile/NW3_ASCS20_msnw3ascs -D -u nw3adm

    # On the node where ERS was installed, comment out the line for the ERS instacnes
    #LD_LIBRARY_PATH=/usr/sap/NW2/ERS12/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/NW2/ERS12/exe/sapstartsrv pf=/usr/sap/NW2/ERS12/profile/NW2_ERS12_msnw2ers -D -u nw2adm
    #LD_LIBRARY_PATH=/usr/sap/NW3/ERS22/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/NW3/ERS22/exe/sapstartsrv pf=/usr/sap/NW3/ERS22/profile/NW3_ERS22_msnw3ers -D -u nw3adm
   ```

7. **[1]** создайте кластерные ресурсы SAP для вновь установленной системы SAP.  

   Если используется архитектура серверной очереди 1 (ENSA1), определите ресурсы для SAP Systems **NW2.** и **NW3** следующим образом:

    ```
     sudo pcs property set maintenance-mode=true

    sudo pcs resource create rsc_sap_NW2_ASCS10 SAPInstance \
    InstanceName=NW2_ASCS10_msnw2ascs START_PROFILE="/sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW2_ASCS

    sudo pcs resource create rsc_sap_NW2_ERS12 SAPInstance \
    InstanceName=NW2_ERS12_msnw2ers START_PROFILE="/sapmnt/NW2/profile/NW2_ERS12_msnw2ers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW2_AERS

    sudo pcs constraint colocation add g-NW2_AERS with g-NW2_ASCS -5000
    sudo pcs constraint location rsc_sap_NW2_ASCS10 rule score=2000 runs_ers_NW2 eq 1
    sudo pcs constraint order g-NW2_ASCS then g-NW2_AERS kind=Optional symmetrical=false

    sudo pcs resource create rsc_sap_NW3_ASCS20 SAPInstance \
    InstanceName=NW3_ASCS20_msnw3ascs START_PROFILE="/sapmnt/NW3/profile/NW3_ASCS20_msnw3ascs" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW3_ASCS

    sudo pcs resource create rsc_sap_NW3_ERS22 SAPInstance \
    InstanceName=NW3_ERS22_msnw3ers START_PROFILE="/sapmnt/NW3/profile/NW2_ERS22_msnw3ers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW3_AERS

    sudo pcs constraint colocation add g-NW3_AERS with g-NW3_ASCS -5000
    sudo pcs constraint location rsc_sap_NW3_ASCS20 rule score=2000 runs_ers_NW3 eq 1
    sudo pcs constraint order g-NW3_ASCS then g-NW3_AERS kind=Optional symmetrical=false

    sudo pcs property set maintenance-mode=false
    ```

   Начиная с SAP NW 7.52 появилась поддержка сервера постановки в очередь 2, включая репликацию. Начиная с ABAP Platform 1809 сервер постановки в очередь 2 устанавливается по умолчанию. Сведения о поддержке сервера постановки в очередь 2 см. в примечании к SAP [2630416](https://launchpad.support.sap.com/#/notes/2630416).
   При использовании архитектуры Server 2 для очереди ([ENSA2](https://help.sap.com/viewer/cff8531bc1d9416d91bb6781e628d4e0/1709%20001/en-US/6d655c383abf4c129b0e5c8683e7ecd8.html)) Определите ресурсы для SAP Systems **NW2.** и **NW3** следующим образом:

    ```
     sudo pcs property set maintenance-mode=true

    sudo pcs resource create rsc_sap_NW2_ASCS10 SAPInstance \
    InstanceName=NW2_ASCS10_msnw2ascs START_PROFILE="/sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW2_ASCS

    sudo pcs resource create rsc_sap_NW2_ERS12 SAPInstance \
    InstanceName=NW2_ERS12_msnw2ers START_PROFILE="/sapmnt/NW2/profile/NW2_ERS12_msnw2ers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW2_AERS

    sudo pcs constraint colocation add g-NW2_AERS with g-NW2_ASCS -5000
    sudo pcs constraint order g-NW2_ASCS then g-NW2_AERS kind=Optional symmetrical=false
    sudo pcs constraint order start g-NW2_ASCS then stop g-NW2_AERS symmetrical=false

    sudo pcs resource create rsc_sap_NW3_ASCS20 SAPInstance \
    InstanceName=NW3_ASCS20_msnw3ascs START_PROFILE="/sapmnt/NW3/profile/NW3_ASCS20_msnw3ascs" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW3_ASCS

    sudo pcs resource create rsc_sap_NW3_ERS22 SAPInstance \
    InstanceName=NW3_ERS22_msnw3ers START_PROFILE="/sapmnt/NW3/profile/NW2_ERS22_msnw3ers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
     --group g-NW3_AERS

    sudo pcs constraint colocation add g-NW3_AERS with g-NW3_ASCS -5000
    sudo pcs constraint order g-NW3_ASCS then g-NW3_AERS kind=Optional symmetrical=false
    sudo pcs constraint order start g-NW3_ASCS then stop g-NW3_AERS symmetrical=false

    sudo pcs property set maintenance-mode=false
    ```

   При переходе на более новую версию сервера постановки в очередь 2 см. примечание к SAP [2641019](https://launchpad.support.sap.com/#/notes/2641019). 

   > [!NOTE]
   > Время ожидания в приведенной выше конфигурации является просто примерами и может потребоваться адаптироваться к конкретной установке SAP. 

   Убедитесь, что состояние кластера — "ОК" и что запущены все ресурсы. Не важно, на каком узле выполняются ресурсы.
   В следующем примере показано состояние ресурсов кластера после добавления SAP Systems **NW2.** и **NW3** в кластер. 

    ```
     sudo pcs status

    Online: [ rhelmsscl1 rhelmsscl2 ]

    Full list of resources:

    rsc_st_azure   (stonith:fence_azure_arm):      Started rhelmsscl1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    ```

8. **[A]** добавьте правила брандмауэра для ASCS и ERS на обоих узлах.  В приведенном ниже примере показаны правила брандмауэра для SAP Systems **NW2.** и **NW3**.  

   ```
    # NW2 - ASCS
    sudo firewall-cmd --zone=public --add-port=62010/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=62010/tcp
    sudo firewall-cmd --zone=public --add-port=3210/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3210/tcp
    sudo firewall-cmd --zone=public --add-port=3610/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3610/tcp
    sudo firewall-cmd --zone=public --add-port=3910/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3910/tcp
    sudo firewall-cmd --zone=public --add-port=8110/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=8110/tcp
    sudo firewall-cmd --zone=public --add-port=51013/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51013/tcp
    sudo firewall-cmd --zone=public --add-port=51014/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51014/tcp
    sudo firewall-cmd --zone=public --add-port=51016/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51016/tcp
    # NW2 - ERS
    sudo firewall-cmd --zone=public --add-port=62112/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=62112/tcp
    sudo firewall-cmd --zone=public --add-port=3212/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3212/tcp
    sudo firewall-cmd --zone=public --add-port=3312/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3312/tcp
    sudo firewall-cmd --zone=public --add-port=51213/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51213/tcp
    sudo firewall-cmd --zone=public --add-port=51214/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51214/tcp
    sudo firewall-cmd --zone=public --add-port=51216/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=51216/tcp
    # NW3 - ASCS
    sudo firewall-cmd --zone=public --add-port=62020/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=62020/tcp
    sudo firewall-cmd --zone=public --add-port=3220/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3220/tcp
    sudo firewall-cmd --zone=public --add-port=3620/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3620/tcp
    sudo firewall-cmd --zone=public --add-port=3920/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3920/tcp
    sudo firewall-cmd --zone=public --add-port=8120/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=8120/tcp
    sudo firewall-cmd --zone=public --add-port=52013/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52013/tcp
    sudo firewall-cmd --zone=public --add-port=52014/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52014/tcp
    sudo firewall-cmd --zone=public --add-port=52016/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52016/tcp
    # NW3 - ERS
    sudo firewall-cmd --zone=public --add-port=62122/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=62122/tcp
    sudo firewall-cmd --zone=public --add-port=3222/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3222/tcp
    sudo firewall-cmd --zone=public --add-port=3322/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=3322/tcp
    sudo firewall-cmd --zone=public --add-port=52213/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52213/tcp
    sudo firewall-cmd --zone=public --add-port=52214/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52214/tcp
    sudo firewall-cmd --zone=public --add-port=52216/tcp --permanent
    sudo firewall-cmd --zone=public --add-port=52216/tcp
   ```

### <a name="proceed-with-the-sap-installation"></a>Продолжение установки SAP 

Завершите установку SAP, выполнив следующие действия.

* [Подготовка серверов приложений SAP NetWeaver](./high-availability-guide-rhel-netapp-files.md#2d6008b0-685d-426c-b59e-6cd281fd45d7)
* [Установка экземпляра СУБД](./high-availability-guide-rhel-netapp-files.md#install-database)
* [Установка основного сервера приложений SAP](./high-availability-guide-rhel-netapp-files.md#sap-netweaver-application-server-installation)
* Установка одного или нескольких дополнительных экземпляров приложений SAP

## <a name="test-the-multi-sid-cluster-setup"></a>Тестирование установки кластера с несколькими ИД безопасности

Следующие тесты являются подмножеством тестовых случаев в руководствах по использованию Red Hat. Они включены для вашего удобства. Полный список тестов кластера см. в следующей документации:

* При использовании томов Azure NetApp Files NFS используйте [высокий уровень доступности виртуальных машин Azure для SAP NetWeaver в RHEL с Azure NetApp Files для приложений SAP](./high-availability-guide-rhel-netapp-files.md) .
* Если вы используете высокий `GlusterFS` уровень доступности, выполните следующие действия [для SAP NetWeaver на виртуальных машинах Azure в RHEL для приложений SAP](./high-availability-guide-rhel.md).  

Всегда Прочтите рекомендации по Red Hat и выполните все дополнительные тесты, которые могли быть добавлены.  
Представленные тесты находятся в двух узлах, кластер с несколькими ИД безопасности и тремя системами SAP.  

1. Перенесите экземпляр ASCS вручную. В примере показано, как перенести экземпляр ASCS для SAP System NW3.

   Состояние ресурсов перед запуском теста:

   ```
    Online: [ rhelmsscl1 rhelmsscl2 ]

    Full list of resources:

    rsc_st_azure   (stonith:fence_azure_arm):      Started rhelmsscl1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
   ```

   Выполните следующие команды в качестве корневого, чтобы перенести экземпляр NW3 ASCS.

   ```
    pcs resource move rsc_sap_NW3_ASCS200
    # Clear temporary migration constraints
    pcs resource clear rsc_sap_NW3_ASCS20

    # Remove failed actions for the ERS that occurred as part of the migration
    pcs resource cleanup rsc_sap_NW3_ERS22
   ```

   Состояние ресурсов после теста:

   ```
    Online: [ rhelmsscl1 rhelmsscl2 ]

    Full list of resources:

    rsc_st_azure   (stonith:fence_azure_arm):      Started rhelmsscl1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
   ```

1. Имитирование сбоя узла

   Состояние ресурсов перед запуском теста:

   ```
    Online: [ rhelmsscl1 rhelmsscl2 ]

    Full list of resources:

    rsc_st_azure   (stonith:fence_azure_arm):      Started rhelmsscl1
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
   ```

   Выполните следующую команду в качестве корневого на узле, где выполняется хотя бы один экземпляр ASCS. В этом примере мы выполнили команду в `rhelmsscl1` , где запущены экземпляры ASCS для NW1, NW2. и NW3.  

   ```
   echo c > /proc/sysrq-trigger
   ```

   Состояние после теста и после сбоя узла, который был успешно запущен, должен выглядеть следующим образом.

   ```
    Full list of resources:

    rsc_st_azure    (stonith:fence_azure_arm):      Started rhelmsscl2
    Resource Group: g-NW1_ASCS
        fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW1_AERS
        fs_NW1_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW1_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW1_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW2_ASCS
        fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW2_AERS
        fs_NW2_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW2_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW2_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
    Resource Group: g-NW3_ASCS
        fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started rhelmsscl2
        vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl2
        nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started rhelmsscl2
        rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started rhelmsscl2
    Resource Group: g-NW3_AERS
        fs_NW3_AERS        (ocf::heartbeat:Filesystem):    Started rhelmsscl1
        vip_NW3_AERS       (ocf::heartbeat:IPaddr2):       Started rhelmsscl1
        nc_NW3_AERS        (ocf::heartbeat:azure-lb):      Started rhelmsscl1
        rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started rhelmsscl1
   ```

   Если имеются сообщения для неисправных ресурсов, очистите состояние неисправных ресурсов. Пример:

   ```
   pcs resource cleanup rsc_sap_NW1_ERS02
   ```

## <a name="next-steps"></a>Дальнейшие действия

* [Планирование и реализация виртуальных машин Azure для SAP][planning-guide]
* [Развертывание виртуальных машин Azure для SAP NetWeaver][deployment-guide]
* [SAP NetWeaver на виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server][dbms-guide]
* Дополнительные сведения об установке высокого уровня доступности и плана для аварийного восстановления SAP HANA на виртуальных машинах Azure см. в статье [Высокий уровень доступности SAP HANA на виртуальных машинах Azure][sap-hana-ha].