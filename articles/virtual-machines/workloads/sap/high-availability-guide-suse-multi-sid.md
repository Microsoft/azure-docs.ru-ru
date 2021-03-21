---
title: Высокий уровень доступности виртуальных машин Azure для SAP NetWeaver в SLES Multi-SID | Документация Майкрософт
description: Рекомендации по обеспечению высокого уровня доступности с несколькими ИД безопасности для SAP NetWeaver на SUSE Linux Enterprise Server для приложений SAP
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 5e514964-c907-4324-b659-16dd825f6f87
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 10/16/2020
ms.author: radeltch
ms.openlocfilehash: 74606909a0bc87caa6acfb0eaf35c05cf35b1858
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101676929"
---
# <a name="high-availability-for-sap-netweaver-on-azure-vms-on-suse-linux-enterprise-server-for-sap-applications-multi-sid-guide"></a>Высокий уровень доступности SAP NetWeaver на виртуальных машинах Azure на SUSE Linux Enterprise Server для многоид безопасности приложений SAP

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-sap-applications-azure]:https://www.netapp.com/us/media/tr-4746.pdf

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

[suse-ha-guide]:https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
[suse-drbd-guide]:https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha_techguides/book_sleha_techguides.html
[suse-ha-12sp3-relnotes]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP3/

[template-multisid-xscs]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[template-converged]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-converged-md%2Fazuredeploy.json
[template-file-server]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-file-server-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability.md
[nfs-ha]:high-availability-guide-suse-nfs.md

В этой статье описывается, как развернуть несколько высокодоступных систем SAP NetWeaver или S4HANA (то есть несколько ИД безопасности) в кластере с двумя узлами на виртуальных машинах Azure с SUSE Linux Enterprise Server для приложений SAP.  

В примерах конфигураций, команд установки и т. д. три системы SAP NetWeaver 7,50 развертываются в одном кластере с высоким уровнем доступности в двух узлах. Идентификаторы безопасности систем SAP:
* **NW1**: экземпляр ASCS с номером **00** и именем виртуального узла **msnw1ascs**; ERS номер экземпляра **02** и имя виртуального узла **msnw1ers**.  
* **NW2.**: экземпляр ASCS с номером **10** и виртуальным именем узла **msnw2ascs**; Экземпляр ERS номер **12** и имя виртуального узла **msnw2ers**.  
* **NW3**: экземпляр ASCS с номером **20** и виртуальным именем узла **msnw3ascs**; Экземпляр ERS с номером **22** и именем виртуального узла **msnw3ers**.  

В этой статье не рассматривается уровень базы данных и развертывание общих папок SAP NFS. В примерах, приведенных в этой статье, мы используем виртуальные имена NW2.-NFS для общих папок NW2. NFS и NW3-NFS для общих ресурсов NW3 NFS при условии, что кластер NFS был развернут.  

Прежде чем начать, ознакомьтесь со следующими примечаниями и документацией по SAP:

* примечание к SAP [1928533][1928533], которое содержит:
  * список размеров виртуальных машин Azure, поддерживаемых для развертывания ПО SAP;
  * важные сведения о доступных ресурсах для каждого размера виртуальной машины Azure;
  * сведения о поддерживаемом программном обеспечении SAP и сочетаниях операционных систем и баз данных;
  * сведения о требуемой версии ядра SAP для Windows и Linux в Microsoft Azure.

* примечание к SAP [2015553][2015553], в котором описываются предварительные требования к SAP при развертывании программного обеспечения SAP в Azure;
* Примечание SAP [2205917][2205917] содержит рекомендуемые параметры ОС для SUSE Linux Enterprise Server для приложений SAP.
* Примечание SAP [1944799][1944799] содержит рекомендации для SAP HANA в SUSE Linux Enterprise Server для приложений SAP.
* примечание к SAP [2178632][2178632], содержащее подробные сведения обо всех доступных метриках мониторинга для SAP в Azure;
* примечание к SAP [2191498][2191498], содержащее сведения о необходимой версии агента SAP Host Agent для Linux в Azure;
* примечание к SAP [2243692][2243692], содержащее сведения о лицензировании SAP в Linux в Azure;
* примечание к SAP [1984787][1984787], содержащее общие сведения о SUSE Linux Enterprise Server 12;
* примечание к SAP [1999351][1999351], содержащее дополнительные сведения об устранении неполадок, связанных с расширением для расширенного мониторинга Azure для SAP;
* [вики-сайт сообщества SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes), содержащий все необходимые примечания к SAP для Linux;
* [SAP NetWeaver на виртуальных машинах Linux. Руководство по планированию и внедрению][planning-guide]
* [Развертывание программного обеспечения SAP на виртуальных машинах Linux в Azure][deployment-guide]
* [Общие сведения о развертывании СУБД на виртуальных машинах Azure для рабочих нагрузок SAP][dbms-guide]
* [Рекомендации по обеспечению высокого уровня доступности SUSE SAP][suse-ha-guide]. Эти руководства содержат всю необходимую информацию для настройки высокого уровня доступности NetWeaver и репликации системы SAP HANA в локальной среде. Придерживайтесь этих общих рекомендаций. Они содержат намного более подробные сведения.
* [Заметки о выпуске расширения SUSE высокого уровня доступности 12 SP3][suse-ha-12sp3-relnotes]
* [Путеводитель по кластеру SUSE Multi-SID для SLES 12 и SLES 15](https://documentation.suse.com/sbp/all/html/SBP-SAP-MULTI-SID/index.html)
* [Приложения NetApp SAP в Microsoft Azure. Использование Azure NetApp Files][anf-sap-applications-azure]
## <a name="overview"></a>Обзор

Для виртуальных машин, участвующих в кластере, необходимо иметь размер, чтобы они могли выполнять все ресурсы, в случае отработки отказа. Каждый идентификатор безопасности SAP может отключаться при сбое независимо друг от друга в высокодоступном кластере с несколькими ИД безопасности.  При использовании ограждения SBD устройства SBD могут совместно использоваться несколькими кластерами.  

Для обеспечения высокого уровня доступности SAP NetWeaver требует высокой доступности общих папок NFS. В этом примере предполагается, что общие папки SAP NFS размещены на [файловом сервере NFS](./high-availability-guide-suse-nfs.md)высокой доступности, который может использоваться несколькими системами SAP. Или общие ресурсы развертываются на [томах Azure NETAPP Files NFS](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).  

![В кластере Pacemaker отображаются подробные сведения о двух кластерах с несколькими идентификаторами безопасности: msidcl1 и msidcl2.](./media/high-availability-guide-suse/ha-suse-multi-sid.png)

> [!IMPORTANT]
> Поддержка кластеризации SAP ASCS/ERS с несколькими ИД безопасности в SUSE Linux в качестве гостевой операционной системы на виртуальных машинах Azure ограничена **пятью** идентификаторами безопасности SAP в одном кластере. Каждый новый идентификатор безопасности повышает сложность. **Не поддерживается** сочетание сервера репликации очереди SAP 1 и постановки в очередь сервера репликации 2 в одном кластере. Кластеризация с несколькими ИД безопасности описывает установку нескольких экземпляров SAP ASCS/ERS с разными идентификаторами безопасности в одном кластере Pacemaker. В настоящее время кластеризация с несколькими ИД безопасности поддерживается только для ASCS/ERS.  

> [!TIP]
> Кластеризация SAP ASCS/ERS с несколькими ИД безопасности — это решение с более высокой степенью сложности. Реализация реализуется более сложно. Он также включает в себя более высокие усилия по администрированию при выполнении действий по обслуживанию (например, по исправлению ОС). Прежде чем начать фактическую реализацию, уделите время тщательному планированию развертывания и всех связанных компонентов, например виртуальных машин, подключений NFS, виртуальных IP-адресов, конфигураций подсистемы балансировки нагрузки и т. д.  

NFS-сервер, SAP NetWeaver ASCS, SAP NetWeaver SCS, SAP NetWeaver ERS и база данных SAP HANA используют виртуальное имя узла и виртуальные IP-адреса. Балансировщику нагрузки в Azure нужен виртуальный IP-адрес. Мы рекомендуем [Load Balancer (цен. категория "Стандартный")](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md).  

В следующем списке показана конфигурация подсистемы балансировки нагрузки (A) SCS и ERS для этого кластера с несколькими идентификаторами безопасности с тремя системами SAP. Вам потребуется отдельный интерфейсный IP-адрес, зонды работоспособности и правила балансировки нагрузки для каждого экземпляра ASCS и ERS для каждого из идентификаторов безопасности. Назначьте всем виртуальным машинам, которые являются частью кластера ASCS/ASCS, один серверный пул.  

### <a name="ascs"></a>(A)SCS

* Конфигурация внешнего интерфейса:
  * IP-адрес для NW1:10.3.1.14
  * IP-адрес для NW2.: 10.3.1.16
  * IP-адрес для NW3:10.3.1.13
* Порты пробы
  * Порт 620 <strong> &lt; Nr &gt;</strong>, поэтому для пробных портов NW1, NW2. и NW3 620 **00**, 620 **10** и 620 **20**
* Правила балансировки нагрузки- 
* Создайте по одному для каждого экземпляра, то есть NW1/ASCS, NW2./ASCS и NW3/ASCS.
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
  * IP-адрес для NW1 10.3.1.15
  * IP-адрес для NW2.10.3.1.17
  * IP-адрес для NW3 10.3.1.19
* Порт пробы:
  * Порт 621 <strong> &lt; Nr &gt;</strong>, поэтому для NW1 #, NW2. и N # пробных портов 621 **02**, 621 **12** и 621 **22**
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

## <a name="sap-nfs-shares"></a>Общие папки SAP NFS

SAP NetWeaver требует наличия общего хранилища для транспорта, каталога профиля и т. д. Для высокодоступной системы SAP важно иметь высокодоступные общие папки NFS. Вам потребуется выбрать архитектуру для общих ресурсов SAP NFS. Один из вариантов — создать [высокодоступный кластер NFS на виртуальных машинах Azure на SUSE Linux Enterprise Server][nfs-ha], которые могут совместно использоваться несколькими системами SAP. 

Другой вариант — развернуть общие папки на [томах Azure NETAPP Files NFS](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).  С Azure NetApp Files вы получите встроенную высокую доступность общих ресурсов SAP NFS.

## <a name="deploy-the-first-sap-system-in-the-cluster"></a>Развертывание первой системы SAP в кластере

Теперь, когда вы решили архитектуру для общих ресурсов SAP NFS, разверните первую систему SAP в кластере, следуя соответствующей документации.

* Если используется высокодоступный NFS Server, следуйте [высокой доступности SAP NetWeaver на виртуальных машинах Azure на SUSE Linux Enterprise Server для приложений SAP](./high-availability-guide-suse.md).  
* При использовании томов Azure NetApp Files NFS используйте [высокий уровень доступности для SAP NetWeaver на виртуальных машинах Azure на SUSE Linux Enterprise Server с Azure NetApp Files для приложений SAP](./high-availability-guide-suse-netapp-files.md) .

Приведенные выше документы помогут вам выполнить необходимые действия по подготовке необходимых инфраструктур, созданию кластера, подготовке ОС к запуску приложения SAP.  

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
* Тестирование функций отказоустойчивости кластера проверено.  
* Разворачиваются общие папки NFS для всех систем SAP.  

### <a name="prepare-for-sap-netweaver-installation"></a>Подготовка к установке SAP NetWeaver

1. Добавьте конфигурацию для новой развернутой системы (т. е. **NW2.**, **NW3**) в существующую Azure Load Balancer, следуя инструкциям по [развертыванию Azure Load Balancer вручную с помощью портал Azure](./high-availability-guide-suse-netapp-files.md#deploy-azure-load-balancer-manually-via-azure-portal). Настройте IP-адреса, порты пробы работоспособности, правила балансировки нагрузки для вашей конфигурации.  

2. **[A]** настройте разрешение имен для дополнительных систем SAP. Можно либо использовать DNS-сервер, либо изменить `/etc/hosts` на всех узлах. В этом примере показано, как использовать `/etc/hosts` файл.  Адаптируйте IP-адреса и имена узлов к вашей среде. 

    ```
    sudo vi /etc/hosts
    # IP address of the load balancer frontend configuration for NW2 ASCS
    10.3.1.16 msnw2ascs
    # IP address of the load balancer frontend configuration for NW3 ASCS
    10.3.1.13 msnw3ascs
    # IP address of the load balancer frontend configuration for NW2 ERS
    10.3.1.17 msnw2ers
    # IP address of the load balancer frontend configuration for NW3 ERS
    10.3.1.19 msnw3ers
    # IP address for virtual host name for the NFS server for NW2
    10.3.1.31 nw2-nfs
    # IP address for virtual host name for the NFS server for NW3
    10.3.1.32 nw3-nfs
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

4. **[A]** настройте `autofs` для подключения файловых систем/сапмнт/СИД и/УСР/САП/СИД/СИС к дополнительным системам SAP, которые развертываются в кластере. В этом примере **NW2.** и **NW3**.  

   Обновите файл `/etc/auto.direct` с помощью файловых систем для дополнительных систем SAP, которые развертываются в кластере.  

   * Если используется файловый сервер NFS, следуйте приведенным [здесь](./high-availability-guide-suse.md#prepare-for-sap-netweaver-installation) инструкциям.
   * Если используется Azure NetApp Files, следуйте приведенным [здесь](./high-availability-guide-suse-netapp-files.md#prepare-for-sap-netweaver-installation) инструкциям. 

   Необходимо перезапустить `autofs` службу, чтобы подключить только что добавленные общие папки.  

### <a name="install-ascs--ers"></a>Установка ASCS и ERS

1. Создайте виртуальные IP-адреса и ресурсы кластера пробы работоспособности для экземпляра ASCS дополнительной системы SAP, развертываемой в кластере. Ниже приведен пример для **NW2.** и **NW3** ASCS с использованием NFS-сервера высокой доступности.  

   > [!IMPORTANT]
   > В ходе последних тестов были выявлены ситуации, когда netcat перестает отвечать на запросы из-за невыполненной работы и ограничения на обработку только одного подключения. Ресурс netcat прекращает прослушивать запросы Azure Load Balancer, и плавающий IP-адрес становится недоступным.  
   > Для существующих кластеров Pacemaker ранее рекомендовалось заменять netcat на socat. Сейчас мы рекомендуем использовать агент ресурса azure-lb, который входит в состав пакета resource-agents со следующими требованиями к версии пакета:
   > - Минимальная версия для SLES 12 с пактом обновления 4 или 5 (SP4/SP5) — resource-agents-4.3.018.a7fb5035-3.30.1.  
   > - Минимальная версия для SLES 15/15 с пакетом обновления 1 (SP1) — resource-agents-4.3.0184.6ee15eb2-4.13.1.  
   >
   > Обратите внимание, что для этого изменения потребуется незначительное время простоя.  
   > Если конфигурация существующих кластеров Pacemaker уже была изменена для использования socat, как описано в [этой статье](https://www.suse.com/support/kb/doc/?id=7024128), немедленно переключаться на агент ресурса azure-lb не нужно.

    ```
      sudo crm configure primitive fs_NW2_ASCS Filesystem device='nw2-nfs:/NW2/ASCS' directory='/usr/sap/NW2/ASCS10' fstype='nfs4' \
       op start timeout=60s interval=0 \
       op stop timeout=60s interval=0 \
       op monitor interval=20s timeout=40s
   
      sudo crm configure primitive vip_NW2_ASCS IPaddr2 \
        params ip=10.3.1.16 cidr_netmask=24 \
        op monitor interval=10 timeout=20
   
      sudo crm configure primitive nc_NW2_ASCS azure-lb port=62010
   
      sudo crm configure group g-NW2_ASCS fs_NW2_ASCS nc_NW2_ASCS vip_NW2_ASCS \
         meta resource-stickiness=3000

      sudo crm configure primitive fs_NW3_ASCS Filesystem device='nw3-nfs:/NW3/ASCS' directory='/usr/sap/NW3/ASCS20' fstype='nfs4' \
        op start timeout=60s interval=0 \
        op stop timeout=60s interval=0 \
        op monitor interval=20s timeout=40s
   
      sudo crm configure primitive vip_NW3_ASCS IPaddr2 \
       params ip=10.3.1.13 cidr_netmask=24 \
       op monitor interval=10 timeout=20
   
      sudo crm configure primitive nc_NW3_ASCS azure-lb port=62020
   
      sudo crm configure group g-NW3_ASCS fs_NW3_ASCS nc_NW3_ASCS vip_NW3_ASCS \
        meta resource-stickiness=3000
    ```

   При создании ресурсов они могут быть назначены разным ресурсам кластера. При их группировании они будут перенесены на один из узлов кластера. Убедитесь, что состояние кластера — ОК, и все ресурсы запущены. Не важно, на каком узле выполняются ресурсы.

2. **[1]** Установите SAP NetWeaver ASCS.  

   Установите SAP NetWeaver ASCS в качестве привилегированного, используя виртуальное имя узла, которое сопоставляется с IP-адресом внешней конфигурации балансировщика нагрузки для ASCS. Например, для System **NW2.** виртуальное имя узла — <b>msnw2ascs</b>, <b>10.3.1.16</b> и номер экземпляра, который использовался для пробы балансировщика нагрузки, например <b>10</b>. для System **NW3** виртуальное имя узла — <b>msnw3ascs</b>, <b>10.3.1.13</b> и номер экземпляра, который использовался для пробы подсистемы балансировки нагрузки, например <b>20</b>.

   Чтобы разрешить непривилегированному пользователю подключаться к SAPinst, можно использовать параметр SAPINST_REMOTE_ACCESS_USER. Можно использовать параметр SAPINST_USE_HOSTNAME для установки SAP с использованием имени виртуального узла.  

     ```
      sudo swpm/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=virtual_hostname
     ```

   Если при установке не удается создать вложенную папку в/usr/SAP/**SID**/АСКС **instance #**, попробуйте задать для владельца значение **SID** ADM и Group to sapsys экземпляра ASCS **#** и повторите попытку.

3. **[1]** создайте виртуальный IP-адрес и ресурсы кластера проверки работоспособности для экземпляра ERS дополнительной системы SAP, которая развертывается в кластере. Ниже приведен пример для **NW2.** и **NW3** ERS с использованием NFS-сервера высокой доступности. 

   ```
    sudo crm configure primitive fs_NW2_ERS Filesystem device='nw2-nfs:/NW2/ASCSERS' directory='/usr/sap/NW2/ERS12' fstype='nfs4' \
      op start timeout=60s interval=0 \
      op stop timeout=60s interval=0 \
      op monitor interval=20s timeout=40s
   
    sudo crm configure primitive vip_NW2_ERS IPaddr2 \
      params ip=10.3.1.17 cidr_netmask=24 \
      op monitor interval=10 timeout=20
   
    sudo crm configure primitive nc_NW2_ERS azure-lb port=62112
   
    sudo crm configure group g-NW2_ERS fs_NW2_ERS nc_NW2_ERS vip_NW2_ERS

    sudo crm configure primitive fs_NW3_ERS Filesystem device='nw3-nfs:/NW3/ASCSERS' directory='/usr/sap/NW3/ERS22' fstype='nfs4' \
      op start timeout=60s interval=0 \
      op stop timeout=60s interval=0 \
      op monitor interval=20s timeout=40s
   
    sudo crm configure primitive vip_NW3_ERS IPaddr2 \
      params ip=10.3.1.19 cidr_netmask=24 \
      op monitor interval=10 timeout=20
   
    sudo crm configure primitive nc_NW3_ERS azure-lb port=62122
   
    sudo crm configure group g-NW3_ERS fs_NW3_ERS nc_NW3_ERS vip_NW3_ERS
   ```

   При создании ресурсов они могут быть назначены разным узлам кластера. При их группировании они будут перенесены на один из узлов кластера. Убедитесь, что состояние кластера — ОК, и все ресурсы запущены.  

   Затем убедитесь, что ресурсы только что созданной группы ERS выполняются на узле кластера, а не на узле кластера, где установлен экземпляр ASCS для одной и той же системы SAP.  Например, если NW2. ASCS был установлен на, убедитесь, `slesmsscl1` что группа ERS NW2. запущена `slesmsscl2` .  Вы можете перенести группу ERS NW2. в `slesmsscl2` , выполнив следующую команду: 

    ```
      crm resource migrate g-NW2_ERS slesmsscl2 force
    ```

4. **[2]** Установите SAP NetWeaver ERS.

   Установите SAP NetWeaver ERS в качестве корневого узла на другом узле, используя виртуальное имя узла, которое сопоставляется с IP-адресом внешней конфигурации балансировщика нагрузки для ERS. Например, для System **NW2.** именем виртуального узла будет <b>msnw2ers</b>, <b>10.3.1.17</b> и номер экземпляра, который использовался для пробы подсистемы балансировки нагрузки, например <b>12</b>. Для System **NW3**— имя виртуального узла <b>msnw3ers</b>, <b>10.3.1.19</b> и номер экземпляра, который использовался для пробы подсистемы балансировки нагрузки, например <b>22</b>. 

   Чтобы разрешить непривилегированному пользователю подключаться к SAPinst, можно использовать параметр SAPINST_REMOTE_ACCESS_USER. Можно использовать параметр SAPINST_USE_HOSTNAME для установки SAP с использованием имени виртуального узла.  

    ```
     sudo swpm/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=virtual_hostname
    ```

   > [!NOTE]
   > Используйте SWPM SP 20 PL 05 или более поздней версии. Более ранние версии задают разрешения неправильно, и установка завершится с ошибкой.

   Если при установке не удается создать вложенную папку в/usr/SAP/**Nw2.**/ЕРС **instance #**, попробуйте задать для владельца **идентификатор SID** ADM, а для группы — sapsys папки ERS **instance #** и повторите попытку.

   Если необходимо перенести группу ERS только что развернутой системы SAP на другой узел кластера, не забудьте удалить ограничение расположения для группы ERS. Ограничение можно удалить, выполнив следующую команду (пример приведен для SAP Systems **NW2.** and **NW3**).  

    ```
      crm resource unmigrate g-NW2_ERS
      crm resource unmigrate g-NW3_ERS
    ```

5. **[1]** адаптируйте профили экземпляров ASCS/SCS и ERS для вновь установленных систем SAP. Ниже приведен пример для NW2.. Вам потребуется адаптировать профили ASCS/SCS и ERS для всех экземпляров SAP, добавленных в кластер.  
 
 * Профиль ASCS/SCS

   ```
   sudo vi /sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs
   
   # Change the restart command to a start command
   #Restart_Program_01 = local $(_EN) pf=$(_PF)
   Start_Program_01 = local $(_EN) pf=$(_PF)
   
   # Add the following lines
   service/halib = $(DIR_CT_RUN)/saphascriptco.so
   service/halib_cluster_connector = /usr/bin/sap_suse_cluster_connector
   
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
   
   # Add the following lines
   service/halib = $(DIR_CT_RUN)/saphascriptco.so
   service/halib_cluster_connector = /usr/bin/sap_suse_cluster_connector
   
   # remove Autostart from ERS profile
   # Autostart = 1
   ```

6. **[A]** Настройте пользователей SAP для вновь развернутой системы SAP, в этом примере это **NW2.** и **NW3**. 

   ```
   # Add sidadm to the haclient group
   sudo usermod -aG haclient nw2adm
   sudo usermod -aG haclient nw3adm
   ```

7. Добавьте в файл службы SAP ASCS и ERS для вновь установленной системы SAP `sapservice` . Ниже приведен пример для SAP Systems **NW2.** и **NW3**.  

   Добавьте запись службы ASCS во второй узел и скопируйте запись службы ERS в первый узел. Выполните команды для каждой системы SAP на узле, где установлен экземпляр ASCS для системы SAP.  

    ```
     # Execute the following commands on slesmsscl1,assuming the NW2 ASCS instance was installed on slesmsscl1
     cat /usr/sap/sapservices | grep ASCS10 | sudo ssh slesmsscl2 "cat >>/usr/sap/sapservices"
     sudo ssh slesmsscl2 "cat /usr/sap/sapservices" | grep ERS12 | sudo tee -a /usr/sap/sapservices
     # Execute the following commands on slesmsscl2, assuming the NW3 ASCS instance was installed on slesmsscl2
     cat /usr/sap/sapservices | grep ASCS20 | sudo ssh slesmsscl1 "cat >>/usr/sap/sapservices"
     sudo ssh slesmsscl1 "cat /usr/sap/sapservices" | grep ERS22 | sudo tee -a /usr/sap/sapservices
    ```

8. **[1]** создайте кластерные ресурсы SAP для вновь установленной системы SAP. 

   Если используется архитектура серверной очереди 1 (ENSA1), определите ресурсы для SAP Systems **NW2.** и **NW3** следующим образом:

    ```
     sudo crm configure property maintenance-mode="true"
    
     sudo crm configure primitive rsc_sap_NW2_ASCS10 SAPInstance \
      operations \$id=rsc_sap_NW2_ASCS10-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW2_ASCS10_msnw2ascs START_PROFILE="/sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs" \
      AUTOMATIC_RECOVER=false \
      meta resource-stickiness=5000 failure-timeout=60 migration-threshold=1 priority=10
    
     sudo crm configure primitive rsc_sap_NW2_ERS12 SAPInstance \
      operations \$id=rsc_sap_NW2_ERS12-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW2_ERS12_msnw2ers START_PROFILE="/sapmnt/NW2/profile/NW2_ERS12_msnw2ers" AUTOMATIC_RECOVER=false IS_ERS=true \
      meta priority=1000
    
     sudo crm configure modgroup g-NW2_ASCS add rsc_sap_NW2_ASCS10
     sudo crm configure modgroup g-NW2_ERS add rsc_sap_NW2_ERS12
    
     sudo crm configure colocation col_sap_NW2_no_both -5000: g-NW2_ERS g-NW2_ASCS
     sudo crm configure location loc_sap_NW2_failover_to_ers rsc_sap_NW2_ASCS10 rule 2000: runs_ers_NW2 eq 1
     sudo crm configure order ord_sap_NW2_first_start_ascs Optional: rsc_sap_NW2_ASCS10:start rsc_sap_NW2_ERS12:stop symmetrical=false
   
     sudo crm configure primitive rsc_sap_NW3_ASCS20 SAPInstance \
      operations \$id=rsc_sap_NW3_ASCS20-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW3_ASCS10_msnw3ascs START_PROFILE="/sapmnt/NW3/profile/NW3_ASCS20_msnw3ascs" \
      AUTOMATIC_RECOVER=false \
      meta resource-stickiness=5000 failure-timeout=60 migration-threshold=1 priority=10
    
     sudo crm configure primitive rsc_sap_NW3_ERS22 SAPInstance \
      operations \$id=rsc_sap_NW3_ERS22-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW3_ERS22_msnw3ers START_PROFILE="/sapmnt/NW3/profile/NW3_ERS22_msnw2ers" AUTOMATIC_RECOVER=false IS_ERS=true \
      meta priority=1000
    
     sudo crm configure modgroup g-NW3_ASCS add rsc_sap_NW3_ASCS20
     sudo crm configure modgroup g-NW3_ERS add rsc_sap_NW3_ERS22
    
     sudo crm configure colocation col_sap_NW3_no_both -5000: g-NW3_ERS g-NW3_ASCS
     sudo crm configure location loc_sap_NW3_failover_to_ers rsc_sap_NW3_ASCS10 rule 2000: runs_ers_NW3 eq 1
     sudo crm configure order ord_sap_NW3_first_start_ascs Optional: rsc_sap_NW3_ASCS20:start rsc_sap_NW3_ERS22:stop symmetrical=false
     sudo crm configure property maintenance-mode="false"
    ```

   Начиная с SAP NW 7.52 появилась поддержка сервера постановки в очередь 2, включая репликацию. Начиная с ABAP Platform 1809 сервер постановки в очередь 2 устанавливается по умолчанию. Сведения о поддержке сервера постановки в очередь 2 см. в примечании к SAP [2630416](https://launchpad.support.sap.com/#/notes/2630416).
   При использовании архитектуры Server 2 для очереди ([ENSA2](https://help.sap.com/viewer/cff8531bc1d9416d91bb6781e628d4e0/1709%20001/en-US/6d655c383abf4c129b0e5c8683e7ecd8.html)) Определите ресурсы для SAP Systems **NW2.** и **NW3** следующим образом:

    ```
     sudo crm configure property maintenance-mode="true"
    
     sudo crm configure primitive rsc_sap_NW2_ASCS10 SAPInstance \
      operations \$id=rsc_sap_NW2_ASCS10-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW2_ASCS10_msnw2ascs START_PROFILE="/sapmnt/NW2/profile/NW2_ASCS10_msnw2ascs" \
      AUTOMATIC_RECOVER=false \
      meta resource-stickiness=5000 
    
     sudo crm configure primitive rsc_sap_NW2_ERS12 SAPInstance \
      operations \$id=rsc_sap_NW2_ERS12-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW2_ERS12_msnw2ers START_PROFILE="/sapmnt/NW2/profile/NW2_ERS12_msnw2ers" AUTOMATIC_RECOVER=false IS_ERS=true 
    
     sudo crm configure modgroup g-NW2_ASCS add rsc_sap_NW2_ASCS10
     sudo crm configure modgroup g-NW2_ERS add rsc_sap_NW2_ERS12
    
     sudo crm configure colocation col_sap_NW2_no_both -5000: g-NW2_ERS g-NW2_ASCS
     sudo crm configure order ord_sap_NW2_first_start_ascs Optional: rsc_sap_NW2_ASCS10:start rsc_sap_NW2_ERS12:stop symmetrical=false
   
     sudo crm configure primitive rsc_sap_NW3_ASCS20 SAPInstance \
      operations \$id=rsc_sap_NW3_ASCS20-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW3_ASCS10_msnw3ascs START_PROFILE="/sapmnt/NW3/profile/NW3_ASCS20_msnw3ascs" \
      AUTOMATIC_RECOVER=false \
      meta resource-stickiness=5000
    
     sudo crm configure primitive rsc_sap_NW3_ERS22 SAPInstance \
      operations \$id=rsc_sap_NW3_ERS22-operations \
      op monitor interval=11 timeout=60 on-fail=restart \
      params InstanceName=NW3_ERS22_msnw3ers START_PROFILE="/sapmnt/NW3/profile/NW3_ERS22_msnw2ers" AUTOMATIC_RECOVER=false IS_ERS=true
    
     sudo crm configure modgroup g-NW3_ASCS add rsc_sap_NW3_ASCS20
     sudo crm configure modgroup g-NW3_ERS add rsc_sap_NW3_ERS22
    
     sudo crm configure colocation col_sap_NW3_no_both -5000: g-NW3_ERS g-NW3_ASCS
     sudo crm configure order ord_sap_NW3_first_start_ascs Optional: rsc_sap_NW3_ASCS20:start rsc_sap_NW3_ERS22:stop symmetrical=false
     sudo crm configure property maintenance-mode="false"
    ```

   При переходе на более новую версию сервера постановки в очередь 2 см. примечание к SAP [2641019](https://launchpad.support.sap.com/#/notes/2641019). 

   Убедитесь, что состояние кластера — "ОК" и что запущены все ресурсы. Не важно, на каком узле выполняются ресурсы.
   В следующем примере показано состояние ресурсов кластера после добавления SAP Systems **NW2.** и **NW3** в кластер. 

    ```
     sudo crm_mon -r
    
    # Online: [ slesmsscl1 slesmsscl2 ]
    
    #Full list of resources:
    
    #stonith-sbd     (stonith:external/sbd): Started slesmsscl1
    # Resource Group: g-NW1_ASCS
    #     fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
    #     nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
    #     vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
    #     rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
    # Resource Group: g-NW1_ERS
    #     fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
    #     nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
    #     vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
    #     rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
    # Resource Group: g-NW2_ASCS
    #     fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
    #     nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
    #     vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
    #     rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
    # Resource Group: g-NW2_ERS
    #     fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
    #     nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
    #     vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
    #     rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
    # Resource Group: g-NW3_ASCS
    #     fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
    #     nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
    #     vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
    #     rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
    # Resource Group: g-NW3_ERS
    #     fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
    #     nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
    #     vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
    #     rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
    ```

   На следующем рисунке показано, как будут выглядеть ресурсы в веб-консоле с высоким уровнем доступности (Hawk) с ресурсами для системы SAP, **NW2.** развернутыми.  

   [![Общие сведения о высоком уровне доступности SAP NetWeaver](./media/high-availability-guide-suse/ha-suse-multi-sid-hawk.png)](./media/high-availability-guide-suse/ha-suse-multi-sid-hawk-detail.png#lightbox)

### <a name="proceed-with-the-sap-installation"></a>Продолжение установки SAP 

Завершите установку SAP, выполнив следующие действия.

* [Подготовка серверов приложений SAP NetWeaver](./high-availability-guide-suse.md#2d6008b0-685d-426c-b59e-6cd281fd45d7)
* [Установка экземпляра СУБД](./high-availability-guide-suse.md#install-database)
* [Установка основного сервера приложений SAP](./high-availability-guide-suse.md#sap-netweaver-application-server-installation)
* Установка одного или нескольких дополнительных экземпляров приложений SAP

## <a name="test-the-multi-sid-cluster-setup"></a>Тестирование установки кластера с несколькими ИД безопасности

Следующие тесты являются подмножеством тестовых случаев в практических руководствах SUSE. Они включены для вашего удобства. Полный список тестов кластера см. в следующей документации:

* Если используется высокодоступный NFS Server, следуйте [высокой доступности SAP NetWeaver на виртуальных машинах Azure на SUSE Linux Enterprise Server для приложений SAP](./high-availability-guide-suse.md).  
* При использовании томов Azure NetApp Files NFS используйте [высокий уровень доступности для SAP NetWeaver на виртуальных машинах Azure на SUSE Linux Enterprise Server с Azure NetApp Files для приложений SAP](./high-availability-guide-suse-netapp-files.md) .

Всегда прочтите руководства и ознакомьтесь с рекомендациями SUSE и выполните все дополнительные тесты, которые могли быть добавлены.  
Представленные тесты находятся в двух узлах, кластер с несколькими ИД безопасности и тремя системами SAP.  

1. Тестирование Хажетфаиловерконфиг и Хачеккфаиловерконфиг

   Выполните следующие команды как <sapsid>adm на узле, где в настоящий момент выполняется экземпляр ASCS. Если выполнение команд завершается сбоем "Сбой: Недостаточно памяти", возможно, это связано с тире в имени вашего узла. Это известная проблема, и SUSE устранит ее в пакете sap-suse-cluster-connector.

   ```
    slesmsscl1:nw1adm 57> sapcontrol -nr 00 -function HAGetFailoverConfig

   # 10.12.2019 21:33:08
   # HAGetFailoverConfig
   # OK
   # HAActive: TRUE
   # HAProductVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4
   # HASAPInterfaceVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4 (sap_suse_cluster_connector 3.1.0)
   # HADocumentation: https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
   # HAActiveNode: slesmsscl1
   # HANodes: slesmsscl1, slesmsscl2

    slesmsscl1:nw1adm 53> sapcontrol -nr 00 -function HACheckFailoverConfig

    # 19.12.2019 21:19:58
    # HACheckFailoverConfig
    # OK
    # state, category, description, comment
    # SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version, SAPInstance includes is-ers patch

    slesmsscl2:nw2adm 35> sapcontrol -nr 10 -function HAGetFailoverConfig

   # 10.12.2019 21:37:09
   # HAGetFailoverConfig
   # OK
   # HAActive: TRUE
   # HAProductVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4
   # HASAPInterfaceVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4 (sap_suse_cluster_connector 3.1.0)
   # HADocumentation: https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
   # HAActiveNode: slesmsscl2
   # HANodes: slesmsscl2, slesmsscl1

    slesmsscl2:nw2adm 52> sapcontrol -nr 10 -function HACheckFailoverConfig

    # 19.12.2019 21:17:39
    # HACheckFailoverConfig
    # OK
    # state, category, description, comment
    # SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version, SAPInstance includes is-ers patch

    slesmsscl1:nw3adm 49> sapcontrol -nr 20 -function HAGetFailoverConfig

   # 10.12.2019 23:35:36
   # HAGetFailoverConfig
   # OK
   # HAActive: TRUE
   # HAProductVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4
   # HASAPInterfaceVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP4 (sap_suse_cluster_connector 3.1.0)
   # HADocumentation: https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
   # HAActiveNode: slesmsscl1
   # HANodes: slesmsscl1, slesmsscl2

    slesmsscl1:nw3adm 52> sapcontrol -nr 20 -function HACheckFailoverConfig

    # 19.12.2019 21:10:42
    # HACheckFailoverConfig
    # OK
    # state, category, description, comment
    # SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version, SAPInstance includes is-ers patch
   ```

2. Перенесите экземпляр ASCS вручную. В примере показано, как перенести экземпляр ASCS для SAP System NW2..  
   Состояние ресурса перед запуском теста:
   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

   Выполните следующие команды в качестве корневого, чтобы перенести экземпляр NW2. ASCS.

   ```
    crm resource migrate rsc_sap_NW2_ASCS10 force
    # INFO: Move constraint created for rsc_sap_NW2_ASCS10
    
    crm resource unmigrate rsc_sap_NW2_ASCS10
   # INFO: Removed migration constraints for rsc_sap_NW2_ASCS10
   
   # Remove failed actions for the ERS that occurred as part of the migration
    crm resource cleanup rsc_sap_NW2_ERS12
   ```

   Состояние ресурсов после теста:

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

1. Проверьте Хафаиловертоноде. Тест, представленный здесь, демонстрирует миграцию экземпляра ASCS для SAP System NW2..  

   Состояние ресурсов перед запуском теста:

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

   Выполните следующие команды в качестве **NW2.** ADM для переноса экземпляра NW2. ASCS.

   ```
    slesmsscl2:nw2adm 53> sapcontrol -nr 10 -host msnw2ascs -user nw2adm password -function HAFailoverToNode ""
   
   # run as root
   # Remove failed actions for the ERS that occurred as part of the migration
   crm resource cleanup rsc_sap_NW2_ERS12
   # Remove migration constraints
   crm resource clear rsc_sap_NW2_ASCS10
   #INFO: Removed migration constraints for rsc_sap_NW2_ASCS10
   ```

   Состояние ресурсов после теста:

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

1. Имитирование сбоя узла

   Состояние ресурсов перед запуском теста:

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
   ```

   Выполните следующую команду в качестве корневого узла на узле, где запущен хотя бы один экземпляр ASCS. В этом примере мы выполнили команду в `slesmsscl2` , где запущены экземпляры ASCS для NW1 и NW3.  

   ```
    slesmsscl2:~ # echo b > /proc/sysrq-trigger
   ```

   Если вы используете SBD, то Pacemaker не должен автоматически запускаться на завершившем работу узле. Состояние после повторного запуска узла должно выглядеть так:

   ```
    Online: [ slesmsscl1 ]
    OFFLINE: [ slesmsscl2 ]
    Full list of resources:

    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl1
    
    Failed Resource Actions:
    * rsc_sap_NW1_ERS02_monitor_11000 on slesmsscl1 'not running' (7): call=125, status=complete, exitreason='',
        last-rc-change='Fri Dec 13 19:32:10 2019', queued=0ms, exec=0ms
    * rsc_sap_NW2_ERS12_monitor_11000 on slesmsscl1 'not running' (7): call=126, status=complete, exitreason='',
        last-rc-change='Fri Dec 13 19:32:10 2019', queued=0ms, exec=0ms
    * rsc_sap_NW3_ERS22_monitor_11000 on slesmsscl1 'not running' (7): call=127, status=complete, exitreason='',
        last-rc-change='Fri Dec 13 19:32:10 2019', queued=0ms, exec=0ms
   ```

   Выполните следующие команды, чтобы запустить Pacemaker на отключенном узле, очистить сообщения SBD и очистить ресурсы, в которых произошел сбой.

   ```bash
   # run as root
   # list the SBD device(s)
   cat /etc/sysconfig/sbd | grep SBD_DEVICE=

   # output is like:
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116;/dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1;/dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3"
   
   sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e116 -d /dev/disk/by-id/scsi-36001405034a84428af24ddd8c3a3e9e1 -d /dev/disk/by-id/scsi-36001405cdd5ac8d40e548449318510c3 message slesmsscl2 clear
   
   systemctl start pacemaker
   crm resource cleanup rsc_sap_NW1_ERS02
   crm resource cleanup rsc_sap_NW2_ERS12
   crm resource cleanup rsc_sap_NW3_ERS22
   ```

   Состояние ресурсов после теста:

   ```
    Full list of resources:
    stonith-sbd     (stonith:external/sbd): Started slesmsscl1
     Resource Group: g-NW1_ASCS
         fs_NW1_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW1_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW1_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW1_ASCS00 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW1_ERS
         fs_NW1_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW1_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW1_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW1_ERS02  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW2_ASCS
         fs_NW2_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW2_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW2_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW2_ASCS10 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW2_ERS
         fs_NW2_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW2_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW2_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW2_ERS12  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
     Resource Group: g-NW3_ASCS
         fs_NW3_ASCS        (ocf::heartbeat:Filesystem):    Started slesmsscl1
         nc_NW3_ASCS        (ocf::heartbeat:azure-lb):      Started slesmsscl1
         vip_NW3_ASCS       (ocf::heartbeat:IPaddr2):       Started slesmsscl1
         rsc_sap_NW3_ASCS20 (ocf::heartbeat:SAPInstance):   Started slesmsscl1
     Resource Group: g-NW3_ERS
         fs_NW3_ERS (ocf::heartbeat:Filesystem):    Started slesmsscl2
         nc_NW3_ERS (ocf::heartbeat:azure-lb):      Started slesmsscl2
         vip_NW3_ERS        (ocf::heartbeat:IPaddr2):       Started slesmsscl2
         rsc_sap_NW3_ERS22  (ocf::heartbeat:SAPInstance):   Started slesmsscl2
   ```

## <a name="next-steps"></a>Дальнейшие действия

* [Планирование и реализация виртуальных машин Azure для SAP][planning-guide]
* [Развертывание виртуальных машин Azure для SAP NetWeaver][deployment-guide]
* [SAP NetWeaver на виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server][dbms-guide]
* Дополнительные сведения об установке высокого уровня доступности и плана для аварийного восстановления SAP HANA на виртуальных машинах Azure см. в статье [Высокий уровень доступности SAP HANA на виртуальных машинах Azure][sap-hana-ha].