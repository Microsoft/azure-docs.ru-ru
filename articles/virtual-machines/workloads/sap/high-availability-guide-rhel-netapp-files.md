---
title: Высокая доступность виртуальных машин Azure для SAP NW в RHEL с Azure NetApp Files | Документация Майкрософт
description: Установите высокий уровень доступности для SAP NW на виртуальных машинах Azure (VM), RHEL с Azure NetApp Files.
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
ms.openlocfilehash: e652d1374db12d797dc4505f07350e6e110d6408
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101674443"
---
# <a name="azure-virtual-machines-high-availability-for-sap-netweaver-on-red-hat-enterprise-linux-with-azure-netapp-files-for-sap-applications"></a>Высокая доступность виртуальных машин Azure для SAP NetWeaver на Red Hat Enterprise Linux с Azure NetApp Files для приложений SAP

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

В этой статье описывается развертывание виртуальных машин, их настройка, установка платформы кластера, а также установка высокодоступной системы SAP NetWeaver 7.50 с помощью [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md).
В примерах конфигураций, команд установки и т. д. Экземпляр ASCS имеет номер 00, экземпляр ERS — номер 01, основной экземпляр приложения (PAS) — 02, а экземпляр приложения (AAS) — 03. Используется QAS с идентификатором системы SAP. 

В этой статье нет подробных сведений об уровне базы данных.  

Прежде всего прочитайте следующие примечания и документы SAP:

* [Документация по Azure NetApp Files][anf-azure-doc] 
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
* [SAP NetWeaver на виртуальных машинах Linux. Руководство по планированию и внедрению][planning-guide]
* [Развертывание программного обеспечения SAP на виртуальных машинах Linux в Azure][deployment-guide]
* [Общие сведения о развертывании СУБД на виртуальных машинах Azure для рабочих нагрузок SAP][dbms-guide]
* [SAP Netweaver в кластере pacemaker](https://access.redhat.com/articles/3150081)
* Общая документация по RHEL
  * [Общие сведения о надстройке для обеспечения высокой доступности](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [Администрирование надстройки для обеспечения высокой доступности](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [Справочник по надстройке для обеспечения высокой доступности](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
  * [Настройка ASCS/ERS для SAP Netweaver с автономными ресурсами в RHEL 7.5](https://access.redhat.com/articles/3569681)
  * [Настройка SAP S/4HANA ASCS/ERS с автономным сервером очереди 2 (ENSA2) в Pacemaker на RHEL ](https://access.redhat.com/articles/3974941)
* Документация по RHEL для Azure:
  * [Политики поддержки для кластеров высокой доступности RHEL — виртуальные машины Microsoft Azure как члены кластера](https://access.redhat.com/articles/3131341)
  * [Установка и настройка кластера высокой доступности Red Hat Enterprise Linux 7.4 (и более поздних версий) в Microsoft Azure](https://access.redhat.com/articles/3252491)
* [Приложения NetApp SAP в Microsoft Azure. Использование Azure NetApp Files][anf-sap-applications-azure]

## <a name="overview"></a>Обзор

Для обеспечения высокого уровня доступности (HA) центральных служб SAP NetWeaver требуется общее хранилище.
Для этого в Red Hat Linux было необходимо создать отдельный кластер Глустерфс высокой доступности. 

Теперь обеспечить высокий уровень доступности SAP NetWeaver можно с помощью общего хранилища, развернутого в службе Azure NetApp Files. Использование Azure NetApp Files для общего хранилища устраняет необходимость в дополнительном [кластере глустерфс](./high-availability-guide-rhel-glusterfs.md). Для обеспечения высокого уровня доступности центральных служб SAP NetWeaver (ASCS/SCS) по-прежнему требуется Pacemaker.

![Общие сведения о высоком уровне доступности SAP NetWeaver](./media/high-availability-guide-rhel/high-availability-guide-rhel-anf.png)

SAP NetWeaver ASCS, SAP NetWeaver SCS, SAP NetWeaver ERS и база данных SAP HANA используют виртуальное имя узла и виртуальные IP-адреса. Балансировщику нагрузки в Azure нужен виртуальный IP-адрес. Мы рекомендуем [Load Balancer (цен. категория "Стандартный")](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md). В следующем списке показана конфигурация балансировщика нагрузки с отдельными интерфейсными IP-адресами (A) SCS и ERS.

### <a name="ascs"></a>(A)SCS

* Конфигурация внешнего интерфейса:
  * 192.168.14.9 IP-адреса
* Порт пробы:
  * порт 620<strong>&lt;nr&gt;</strong>.
* Правила балансировки нагрузки
  * Если используется Load Balancer (цен. категория "Стандартный"), выберите **порты высокой доступности**.
  * 32<strong>&lt;nr&gt;</strong> TCP;
  * 36<strong>&lt;nr&gt;</strong> TCP;
  * 39<strong>&lt;nr&gt;</strong> TCP;
  * 81<strong>&lt;nr&gt;</strong> TCP;
  * 5<strong>&lt;nr&gt;</strong>13 TCP;
  * 5<strong>&lt;nr&gt;</strong>14 TCP;
  * 5<strong>&lt;nr&gt;</strong>16 TCP.

### <a name="ers"></a>ERS

* Конфигурация внешнего интерфейса:
  * 192.168.14.10 IP-адреса
* Порт пробы:
  * Порт 621<strong>&lt;nr&gt;</strong>.
* Правила балансировки нагрузки
  * Если используется Load Balancer (цен. категория "Стандартный"), выберите **порты высокой доступности**.
  * 32<strong>&lt;nr&gt;</strong> TCP;
  * 33<strong>&lt;nr&gt;</strong> TCP;
  * 5<strong>&lt;nr&gt;</strong>13 TCP;
  * 5<strong>&lt;nr&gt;</strong>14 TCP;
  * 5<strong>&lt;nr&gt;</strong>16 TCP.

* Конфигурация серверной части:
  * подключена к основным сетевым интерфейсам всех виртуальных машин, которые должны быть частью кластера (A)SCS/ERS.

## <a name="setting-up-the-azure-netapp-files-infrastructure"></a>Настройка инфраструктуры Azure NetApp Files 

SAP NetWeaver требует общее хранилище для каталога профиля и транспорта.  Прежде чем продолжить настройку инфраструктуры Azure NetApp Files, ознакомьтесь с документацией по [Azure NetApp Files][anf-azure-doc]. Проверьте, предлагает ли выбранный регион Azure службу Azure NetApp Files. Перейдите по приведенной ниже ссылке, чтобы узнать о доступности службы Azure NetApp Files по регионам Azure: [Доступность Azure NetApp Files по региону Azure][anf-avail-matrix].

Файлы Azure NetApp доступны в нескольких [регионах Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Перед развертыванием Azure NetApp Files запросите подключение к Azure NetApp Files, следуя инструкциям в статье [Регистрация в службе Azure NetApp Files][anf-register]. 

### <a name="deploy-azure-netapp-files-resources"></a>Развертывание ресурсов Azure NetApp Files  

Предполагается, что вы уже развернули [виртуальную сеть Azure](../../../virtual-network/virtual-networks-overview.md). Ресурсы Azure NetApp Files и виртуальные машины, к которым будут подключаться ресурсы Azure NetApp Files, должны быть развернуты в одной виртуальной сети Azure или в одноранговых виртуальных сетях Azure.  

1. Если вы еще не сделали этого, запросите [подключение к Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).  
2. Создайте учетную запись NetApp в выбранном регионе Azure, следуя [этим инструкциям](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md).  
3. Настройте пул емкости Azure NetApp Files, следуя [инструкциям по настройке пула емкости Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).  
В архитектуре SAP NetWeaver, представленной в этой статье, используется один пул емкости Azure NetApp Files с номером SKU категории "Премиум". Он является рекомендуемым для рабочей нагрузки приложения SAP NetWeaver в Azure.  
4. Делегируйте подсеть в Azure NetApp Files согласно [инструкциям по делегированию подсети в Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).  

5. Разверните тома Azure NetApp Files, следуя [инструкциям по созданию тома для Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md). Разверните тома в выделенной [ подсети](/rest/api/virtualnetwork/subnets) Azure NetApp Files. IP-адреса томов Azure NetApp Files назначаются автоматически. Учтите, что ресурсы Azure NetApp Files и виртуальные машины Azure должны находиться в одной виртуальной сети Azure или в одноранговых виртуальных сетях Azure. В этом примере используются два Azure NetApp Files томов: SAP<b>Оценка качества</b> и транссап. Пути к файлам, подключенные к соответствующим точкам подключения, —/усрсап<b>Оценка качества</b>/sapmnt<b>Оценка качества</b>,/усрсап<b>Оценка качества</b>/усрсап<b>Оценка качества</b>sys и т. д.  

   1. Volume SAP<b>Оценка качества</b> (NFS://192.168.24.5/usrsap<b>Оценка качества</b>/sapmnt<b>Оценка качества</b>)
   2. Volume SAP<b>Оценка качества</b> (NFS://192.168.24.5/usrsap<b>Оценка качества</b>/усрсап<b>Оценка качества</b>ASCS)
   3. Volume SAP<b>Оценка качества</b> (NFS://192.168.24.5/usrsap<b>Оценка качества</b>/усрсап<b>Оценка качества</b>sys)
   4. Volume SAP<b>Оценка качества</b> (NFS://192.168.24.5/usrsap<b>Оценка качества</b>/усрсап<b>Оценка качества</b>ERS)
   5. Volume Транссап (nfs://192.168.24.4/transSAP)
   6. Volume SAP<b>Оценка качества</b> (NFS://192.168.24.5/usrsap<b>Оценка качества</b>/усрсап<b>Оценка качества</b>PAS)
   7. Volume SAP<b>Оценка качества</b> (NFS://192.168.24.5/usrsap<b>Оценка качества</b>/усрсап<b>Оценка качества</b>AAS)
  
В этом примере мы использовали Azure NetApp Files для всех файловых систем SAP NetWeaver, чтобы продемонстрировать использование этой службы. Файловые системы SAP, которые не нужно подключать через NFS, можно также развернуть как [хранилище дисков Azure](../../disks-types.md#premium-ssd). В этом примере <b>a-e</b> должен быть в Azure NetApp Files, а <b>f-g</b> (т. е. /usr/sap/<b>QAS</b>/D<b>02</b>, /usr/sap/<b>QAS</b>/D<b>03</b>) можно развернуть в качестве хранилища дисков Azure. 

### <a name="important-considerations"></a>Важные сведения

При выборе Azure NetApp Files для архитектуры с высоким уровнем доступности SAP NetWeaver в SUSE учитывайте следующие важные моменты.

- Минимальный размер пула емкости равен 4 ТиБ. Размер пула емкости можно увеличить с шагом в 1 тиб.
- Минимальный размер том — 100 ГиБ.
- Azure NetApp Files и все виртуальные машины, к которым будут подключаться тома Azure NetApp Files, должны быть развернуты в одной виртуальной сети Azure или в [одноранговых виртуальных сетях](../../../virtual-network/virtual-network-peering-overview.md) в одном регионе. В настоящее время поддерживается доступ к Azure NetApp Files через пиринг виртуальных сетей в том же регионе. Доступ к NetApp Azure через глобальный пиринг пока не поддерживается.
- В выбранной виртуальной сети должна быть подсеть, делегированная службе Azure NetApp Files.
- Azure NetApp Files предлагает [политику экспорта](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md), которая позволяет управлять разрешенными клиентами, типом доступа (чтение и запись, доступ только для чтения и т. д.). 
- Функция Azure NetApp Files пока еще не поддерживает зоны. В настоящее время Azure NetApp Files не развернута во всех зонах доступности региона Azure. Следует учитывать возможную задержку в некоторых регионах Azure. 
- Тома Azure NetApp Files можно развернуть в виде томов NFSv3 или NFSv4.1. Оба протокола поддерживаются на уровне приложений SAP (ASCS/ERS, серверы приложений SAP). 

## <a name="setting-up-ascs"></a>Настройка (A)SCS

В этом примере ресурсы были развернуты вручную с помощью [портал Azure](https://portal.azure.com/#home).

### <a name="deploy-linux-manually-via-azure-portal"></a>Развертывание Linux вручную с помощью портала Azure

Сначала необходимо создать тома Azure NetApp Files. Разверните виртуальные машины. После этого следует создать подсистему балансировки нагрузки и использовать виртуальные машины во внутренних пулах.

1. Создайте подсистему балансировки нагрузки (внутреннюю, ценовой категории "Стандартный").  
   1. Создайте IP-адреса внешнего интерфейса.
      1. IP-адрес 192.168.14.9 для ASCS
         1. Откройте подсистему балансировки нагрузки, выберите пул внешних IP-адресов и щелкните "Добавить".
         1. Введите имя нового внешнего пула IP-адресов (например, **frontend.QAS.ASCS**).
         1. Задайте для назначения значение static и введите IP-адрес (например, **192.168.14.9**).
         1. Нажмите кнопку "ОК".
      1. IP-адрес 192.168.14.10 для ASCS ERS
         * Повторите описанные выше шаги в разделе "a", чтобы создать IP-адрес для ERS (например, **192.168.14.10** и **интерфейсного интерфейса). Оценка качества. ERS**)
   1. Создание внутреннего пула
      1. Выберите подсистему балансировки нагрузки, щелкните "Серверные пулы" и нажмите кнопку "Добавить".
      1. Введите имя нового внутреннего пула (например, **backend.QAS**).
      1. Щелкните "Добавить виртуальную машину".
      1. Выберите виртуальную машину. 
      1. Выберите виртуальные машины кластера (A)SCS и IP-адреса для них.
      1. Нажмите "Добавить"
   1. Создайте пробы работоспособности.
      1. Порт 620 **00** для ASCS
         1. Выберите подсистему балансировки нагрузки, щелкните "Зонды работоспособности" и нажмите кнопку "Добавить".
         1. Введите имя новой пробы работоспособности (например, **health.QAS.ASCS**).
         1. Выберите протокол TCP, порт 620 **00**, интервал, равный 5, и порог состояния неработоспособности, равный 2.
         1. Нажмите кнопку "ОК"
      1. Порт 621 **01** для ASCS ERS
            * Чтобы создать пробу работоспособности для ERS, повторите предыдущие шаги в разделе "в" (например, 621 **01** и **health.QAS.ERS**).
   1. Правила балансировки нагрузки
      1. Правила балансировки нагрузки для ASCS
         1. Откройте подсистему балансировки нагрузки, выберите правила балансировки нагрузки и нажмите кнопку Добавить.
         1. Введите имя нового правила балансировки нагрузки (например, **lb.QAS.ASCS**).
         1. Выберите внешний IP-адрес для ASCS, внутренний пул и проверку работоспособности, созданные ранее (например, **frontend.QAS.ASCS**, **backend.QAS** и **health.QAS.ASCS**).
         1. Выберите **Порты высокой доступности**.
         1. Увеличьте время ожидания до 30 минут.
         1. **Не забудьте включить плавающий IP-адрес**.
         1. Нажмите кнопку "ОК"
         * Повторите описанные выше действия, чтобы создать правила балансировки нагрузки для ERS (например, **lb.QAS.ERS**).
1. Кроме того, если в сценарии требуется базовая подсистема балансировки нагрузки (внутренняя), выполните приведенные ниже действия.  
   1. Создайте IP-адреса внешнего интерфейса.
      1. IP-адрес 192.168.14.9 для ASCS
         1. Откройте подсистему балансировки нагрузки, выберите пул внешних IP-адресов и щелкните "Добавить".
         1. Введите имя нового внешнего пула IP-адресов (например, **frontend.QAS.ASCS**).
         1. Задайте для назначения значение static и введите IP-адрес (например, **192.168.14.9**).
         1. Нажмите кнопку "ОК".
      1. IP-адрес 192.168.14.10 для ASCS ERS
         * Повторите описанные выше шаги в разделе "a", чтобы создать IP-адрес для ERS (например, **192.168.14.10** и **интерфейсного интерфейса). Оценка качества. ERS**)
   1. Создание внутреннего пула
      1. Выберите подсистему балансировки нагрузки, щелкните "Серверные пулы" и нажмите кнопку "Добавить".
      1. Введите имя нового внутреннего пула (например, **backend.QAS**).
      1. Щелкните "Добавить виртуальную машину".
      1. Выберите ранее созданную группу доступности для ASCS. 
      1. Выберите виртуальные машины кластера (A)SCS.
      1. Нажмите кнопку "ОК"
   1. Создайте пробы работоспособности.
      1. Порт 620 **00** для ASCS
         1. Выберите подсистему балансировки нагрузки, щелкните "Зонды работоспособности" и нажмите кнопку "Добавить".
         1. Введите имя новой пробы работоспособности (например, **health.QAS.ASCS**).
         1. Выберите протокол TCP, порт 620 **00**, интервал, равный 5, и порог состояния неработоспособности, равный 2.
         1. Нажмите кнопку "ОК"
      1. Порт 621 **01** для ASCS ERS
            * Чтобы создать пробу работоспособности для ERS, повторите предыдущие шаги в разделе "в" (например, 621 **01** и **health.QAS.ERS**).
   1. Правила балансировки нагрузки
      1. TCP 32 **00** для ASCS
         1. Откройте подсистему балансировки нагрузки, выберите правила балансировки нагрузки и нажмите кнопку Добавить.
         1. Введите имя нового правила балансировки нагрузки (например, **lb.QAS.ASCS.3200**).
         1. Выберите внешний IP-адрес для ASCS, внутренний пул и пробу работоспособности, созданные ранее (например, **frontend.QAS.ASCS**).
         1. Оставьте выбранным протокол **TCP** и введите порт **3200**.
         1. Увеличьте время ожидания до 30 минут.
         1. **Не забудьте включить плавающий IP-адрес**.
         1. Нажмите кнопку "ОК"
      1. Дополнительные порты для ASCS
         * Повторите предыдущие шаги в разделе "г", указав порты 36 **00**, 39 **00**, 81 **00**, 5 **00** 13, 5 **00** 14, 5 **00** 16 и TCP в качестве протокола для ASCS.
      1. Дополнительные порты для ASCS ERS
         * Повторите предыдущие шаги, указав порты 32 **01**, 33 **01**, 5 **01** 13, 5 **01** 14, 5 **01** 16 и TCP в качестве протокола для ASCS ERS.

      > [!IMPORTANT]
      > Плавающий IP-адрес не поддерживается для вторичной IP-конфигурации NIC в сценариях балансировки нагрузки. Дополнительные сведения см. в статье [ограничения балансировщика нагрузки Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Если для виртуальной машины требуется дополнительный IP-адрес, разверните вторую сетевую карту.  

      > [!Note]
      > Если в серверный пул внутреннего (без общедоступного IP-адреса) Azure Load Balancer ценовой категории "Стандартный" помещаются виртуальные машины без общедоступных IP-адресов, у них не будет исходящего подключения к Интернету без дополнительной настройки, разрешающей маршрутизацию к общедоступным конечным точкам. Подробные сведения о такой настройке см. в статье [Подключение к общедоступной конечной точке для виртуальных машин с помощью Azure Load Balancer (цен. категория "Стандартный") в сценариях обеспечения высокого уровня доступности SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

      > [!IMPORTANT]
      > Не включайте метки времени TCP на виртуальных машинах Azure, размещенных за Azure Load Balancer. Включение меток времени TCP помешает работе проб работоспособности. Установите для параметра **net.ipv4.tcp_timestamps** значение **0**. Дополнительные сведения см. в статье [Пробы работоспособности Load Balancer](../../../load-balancer/load-balancer-custom-probe-overview.md).

## <a name="disable-id-mapping-if-using-nfsv41"></a>Отключите сопоставление идентификаторов (если используется NFSv4.1).

Инструкции в этом разделе применимы, только если используются тома Azure NetApp Files с протоколом NFSv4.1. Выполните настройку на всех виртуальных машинах, к которым будут подключены тома Azure NetApp Files NFSv4.1.  

1. Проверьте параметр домена NFS. Убедитесь, что домен настроен в качестве домена Azure NetApp Files по умолчанию, т. е. **`defaultv4iddomain.com`** , а для сопоставления установлено значение **Никто**.  

    > [!IMPORTANT]
    > Обязательно укажите домен NFS в `/etc/idmapd.conf` виртуальной машине в соответствии с конфигурацией домена по умолчанию в Azure NetApp Files: **`defaultv4iddomain.com`** . В случае несоответствия между конфигурацией домена на клиенте NFS (т. е. виртуальной машине) и NFS-сервере (т. е. конфигурацией Azure NetApp) разрешения для файлов на томах NetApp в Azure, подключенных к виртуальным машинам, будут отображаться как `nobody`.  

    <pre><code>
    sudo cat /etc/idmapd.conf
    # Example
    [General]
    Domain = <b>defaultv4iddomain.com</b>
    [Mapping]
    Nobody-User = <b>nobody</b>
    Nobody-Group = <b>nobody</b>
    </code></pre>

4. **[A]** Проверьте параметр `nfs4_disable_idmapping`. Ему должно быть задано значение **Y**. Чтобы создать структуру каталогов, в которой находится параметр `nfs4_disable_idmapping`, выполните команду mount. Вы не сможете вручную создать каталог в /sys/modules, так как доступ зарезервирован для ядра или драйверов.  

    <pre><code>
    # Check nfs4_disable_idmapping 
    cat /sys/module/nfs/parameters/nfs4_disable_idmapping
    # If you need to set nfs4_disable_idmapping to Y
    mkdir /mnt/tmp
    mount 192.168.24.5:/sap<b>QAS</b>
    umount  /mnt/tmp
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    </code></pre>

   Дополнительные сведения о том, как изменить `nfs4_disable_idmapping` параметр, см. в разделе https://access.redhat.com/solutions/1749883 .

### <a name="create-pacemaker-cluster"></a>Создание кластера Pacemaker

Следуйте указаниям в статье [Настройка кластера Pacemaker в Red Hat Enterprise Linux в Azure](high-availability-guide-rhel-pacemaker.md), чтобы создать базовый кластер Pacemaker для этого (A)SCS-сервера.

### <a name="prepare-for-sap-netweaver-installation"></a>Подготовка к установке SAP NetWeaver

Ниже приведены элементы с префиксами: **[A]**  — применяется ко всем узлам, **[1**] — применяется только к узлу 1, **[2]**  — применяется только к узлу 2.

1. **[A]** Установите разрешения имен.

   Можно использовать DNS-сервер или внести изменения в файл /etc/hosts на всех узлах. В этом примере показано, как использовать файл /etc/hosts.
   Замените IP-адрес и имя узла в следующих командах.

    ```
    sudo vi /etc/hosts
    ```

   Вставьте следующие строки в /etc/hosts. Измените IP-адрес и имя узла в соответствии со своей средой.

    ```
    # IP address of cluster node 1
    192.168.14.5    anftstsapcl1
    # IP address of cluster node 2
    192.168.14.6     anftstsapcl2
    # IP address of the load balancer frontend configuration for SAP Netweaver ASCS
    192.168.14.9    anftstsapvh
    # IP address of the load balancer frontend configuration for SAP Netweaver ERS
    192.168.14.10    anftstsapers
    ```

1. **[1]** Создайте каталоги SAP в томе Azure NetApp Files.  
   Временно подключите том Azure NetApp Files к одной из виртуальных машин и создайте каталоги SAP (пути к файлам).  

    ```
     # mount temporarily the volume
     sudo mkdir -p /saptmp
     # If using NFSv3
     sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 192.168.24.5:/sapQAS /saptmp
     # If using NFSv4.1
     sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys,tcp 192.168.24.5:/sapQAS /saptmp
     # create the SAP directories
     sudo cd /saptmp
     sudo mkdir -p sapmntQAS
     sudo mkdir -p usrsapQASascs
     sudo mkdir -p usrsapQASers
     sudo mkdir -p usrsapQASsys
     sudo mkdir -p usrsapQASpas
     sudo mkdir -p usrsapQASaas
     # unmount the volume and delete the temporary directory
     sudo cd ..
     sudo umount /saptmp
     sudo rmdir /saptmp
    ``` 

1. **[A]** Создайте общие папки.

   ```
   sudo mkdir -p /sapmnt/QAS
   sudo mkdir -p /usr/sap/trans
   sudo mkdir -p /usr/sap/QAS/SYS
   sudo mkdir -p /usr/sap/QAS/ASCS00
   sudo mkdir -p /usr/sap/QAS/ERS01
   
   sudo chattr +i /sapmnt/QAS
   sudo chattr +i /usr/sap/trans
   sudo chattr +i /usr/sap/QAS/SYS
   sudo chattr +i /usr/sap/QAS/ASCS00
   sudo chattr +i /usr/sap/QAS/ERS01
   ```

1. **[A]** установите клиент NFS и другие требования

   ```
   sudo yum -y install nfs-utils resource-agents resource-agents-sap
   ```

1. **[A]** Проверка версии resource-agents-sap

   Убедитесь, что версия пакета resource-agents-sap не ниже 3.9.5-124.el7.
   ```
   sudo yum info resource-agents-sap
   
   # Loaded plugins: langpacks, product-id, search-disabled-repos
   # Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
   # Installed Packages
   # Name        : resource-agents-sap
   # Arch        : x86_64
   # Version     : 3.9.5
   # Release     : 124.el7
   # Size        : 100 k
   # Repo        : installed
   # From repo   : rhel-sap-for-rhel-7-server-rpms
   # Summary     : SAP cluster resource agents and connector script
   # URL         : https://github.com/ClusterLabs/resource-agents
   # License     : GPLv2+
   # Description : The SAP resource agents and connector script interface with
   #          : Pacemaker to allow SAP instances to be managed in a cluster
   #          : environment.
   ```


1. **[A]** Добавление записей подключения

   При использовании NFSv3:
   ```
   sudo vi /etc/fstab
   
   # Add the following lines to fstab, save and exit
    192.168.24.5:/sapQAS/sapmntQAS /sapmnt/QAS nfs rw,hard,rsize=65536,wsize=65536,vers=3
    192.168.24.5:/sapQAS/usrsapQASsys /usr/sap/QAS/SYS nfs rw,hard,rsize=65536,wsize=65536,vers=3
    192.168.24.4:/transSAP /usr/sap/trans nfs rw,hard,rsize=65536,wsize=65536,vers=3
   ```

   При использовании Нфсв 4.1:
   ```
   sudo vi /etc/fstab
   
   # Add the following lines to fstab, save and exit
    192.168.24.5:/sapQAS/sapmntQAS /sapmnt/QAS nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
    192.168.24.5:/sapQAS/usrsapQASsys /usr/sap/QAS/SYS nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
    192.168.24.4:/transSAP /usr/sap/trans nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   ```

   > [!NOTE]
   > При подключении томов Azure NetApp Files следует использовать соответствующую версию протокола NFS. Если тома Azure NetApp Files созданы как тома NFSv3, применяйте соответствующую конфигурацию NFSv3. Если тома Azure NetApp Files созданы как тома NFSv4.1, следуйте инструкциям по отключению сопоставления идентификаторов и обязательно используйте соответствующую конфигурацию NFSv4.1. В этом примере тома Azure NetApp Files были созданы как тома NFSv3.  

   Подключение новых общих папок

   ```
   sudo mount -a  
   ```

1. **[A]** Настройте файл подкачки.

   ```
   sudo vi /etc/waagent.conf
   
   # Set the property ResourceDisk.EnableSwap to y
   # Create and use swapfile on resource disk.
   ResourceDisk.EnableSwap=y
   
   # Set the size of the SWAP file with property ResourceDisk.SwapSizeMB
   # The free space of resource disk varies by virtual machine size. Make sure that you do not set a value that is too big. You can check the SWAP space with command swapon
   # Size of the swapfile.
   ResourceDisk.SwapSizeMB=2000
   ```

   Перезапустите агент, чтобы активировать изменения:

   ```
   sudo service waagent restart
   ```

1. **[A]** Настройка RHEL

   Настройте RHEL, как описано в примечании к SAP [2002167]

### <a name="installing-sap-netweaver-ascsers"></a>Установка SAP NetWeaver ASCS/ERS

1. **[1]** Создайте виртуальный IP-адрес и проверку работоспособности для экземпляра ASCS.

   ```
   sudo pcs node standby anftstsapcl2
   # If using NFSv3
   sudo pcs resource create fs_QAS_ASCS Filesystem device='192.168.24.5:/sapQAS/usrsapQASascs' \
     directory='/usr/sap/QAS/ASCS00' fstype='nfs' force_unmount=safe \
     op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-QAS_ASCS
   
   # If using NFSv4.1
   sudo pcs resource create fs_QAS_ASCS Filesystem device='192.168.24.5:/sapQAS/usrsapQASascs' \
     directory='/usr/sap/QAS/ASCS00' fstype='nfs' force_unmount=safe options='sec=sys,vers=4.1' \
     op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
     --group g-QAS_ASCS
   
   sudo pcs resource create vip_QAS_ASCS IPaddr2 \
     ip=192.168.14.9 cidr_netmask=24 \
     --group g-QAS_ASCS
   
   sudo pcs resource create nc_QAS_ASCS azure-lb port=62000 \
     --group g-QAS_ASCS
   ```

   Убедитесь, что состояние кластера — "ОК" и что запущены все ресурсы. Не важно, на каком узле выполняются ресурсы.

   ```
   sudo pcs status
   
   # Node anftstsapcl2: standby
   # Online: [ anftstsapcl1 ]
   #
   # Full list of resources:
   #
   # rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
   #  Resource Group: g-QAS_ASCS
   #      fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
   #      nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
   #      vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
   ```

1. **[1]** Установите SAP NetWeaver ASCS.  

   Установите SAP NetWeaver ASCS в качестве корневого узла на первом узле, используя виртуальное имя узла, которое сопоставляется с IP-адресом внешней конфигурации балансировщика нагрузки для ASCS, например <b>анфтстсапвх</b>, <b>192.168.14.9</b> и номером экземпляра, который использовался для пробы подсистемы балансировки нагрузки, например <b>00</b>.

   Чтобы разрешить непривилегированному пользователю подключаться к SAPinst, можно использовать параметр SAPINST_REMOTE_ACCESS_USER.

   ```
   # Allow access to SWPM. This rule is not permanent. If you reboot the machine, you have to run the command again.
   sudo firewall-cmd --zone=public  --add-port=4237/tcp
   
   sudo <swpm>/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=<virtual_hostname>
   ```

   Если при установке не удается создать вложенную папку в/usr/SAP/**Оценка качества**/АСКС **00**, попробуйте задать владельца и группу в папке ASCS **00** и повторите попытку.

   ```
   sudo chown qasadm /usr/sap/QAS/ASCS00
   sudo chgrp sapsys /usr/sap/QAS/ASCS00
   ```

1. **[1]** Создайте виртуальный IP-адрес и проверку работоспособности для экземпляра ERS.

   ```
   sudo pcs node unstandby anftstsapcl2
   sudo pcs node standby anftstsapcl1
   
   # If using NFSv3
   sudo pcs resource create fs_QAS_AERS Filesystem device='192.168.24.5:/sapQAS/usrsapQASers' \
     directory='/usr/sap/QAS/ERS01' fstype='nfs' force_unmount=safe \
     op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
    --group g-QAS_AERS
   
   # If using NFSv4.1
   sudo pcs resource create fs_QAS_AERS Filesystem device='192.168.24.5:/sapQAS/usrsapQASers' \
     directory='/usr/sap/QAS/ERS01' fstype='nfs' force_unmount=safe options='sec=sys,vers=4.1' \
     op start interval=0 timeout=60 op stop interval=0 timeout=120 op monitor interval=200 timeout=40 \
    --group g-QAS_AERS
   
   sudo pcs resource create vip_QAS_AERS IPaddr2 \
     ip=192.168.14.10 cidr_netmask=24 \
    --group g-QAS_AERS
   
   sudo pcs resource create nc_QAS_AERS azure-lb port=62101 \
    --group g-QAS_AERS
   ```
 
   Убедитесь, что состояние кластера — "ОК" и что запущены все ресурсы. Не важно, на каком узле выполняются ресурсы.

   ```
   sudo pcs status
   
   # Node anftstsapcl1: standby
   # Online: [ anftstsapcl2 ]
   #
   # Full list of resources:
   #
   # rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl2
   #  Resource Group: g-QAS_ASCS
   #      fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
   #      nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2<
   #      vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
   #  Resource Group: g-QAS_AERS
   #      fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
   #      nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
   #      vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
   ```

1. **[2]** Установите SAP NetWeaver ERS.  

   Установите SAP NetWeaver ERS в качестве корневого узла на втором узле, используя виртуальное имя узла, которое сопоставляется с IP-адресом внешней конфигурации балансировщика нагрузки для ERS, например <b>анфтстсаперс</b>, <b>192.168.14.10</b> и номером экземпляра, который использовался для пробы подсистемы балансировки нагрузки, например <b>01</b>.

   Чтобы разрешить непривилегированному пользователю подключаться к SAPinst, можно использовать параметр SAPINST_REMOTE_ACCESS_USER.

   ```
   # Allow access to SWPM. This rule is not permanent. If you reboot the machine, you have to run the command again.
   sudo firewall-cmd --zone=public  --add-port=4237/tcp

   sudo <swpm>/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin SAPINST_USE_HOSTNAME=<virtual_hostname>
   ```

   Если установка завершилась ошибкой при создании вложенной папки в /usr/sap/**QAS**/ERS **01**, попробуйте задать владельца и группу ERS папки **01** и повторите заново.

   ```
   sudo chown qaadm /usr/sap/QAS/ERS01
   sudo chgrp sapsys /usr/sap/QAS/ERS01
   ```

1. **[1]** Адаптируйте профили экземпляров ASCS/SCS и ERS.

   * Профиль ASCS/SCS

   ```
   sudo vi /sapmnt/QAS/profile/QAS_ASCS00_anftstsapvh
   
   # Change the restart command to a start command
   #Restart_Program_01 = local $(_EN) pf=$(_PF)
   Start_Program_01 = local $(_EN) pf=$(_PF)
   
   # Add the keep alive parameter, if using ENSA1
   enque/encni/set_so_keepalive = true
   ```

   Для ENSA1 и ENSA2 убедитесь, что `keepalive` Параметры ОС заданы, как описано в примечании к SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).  

   * Профиль ERS

   ```
   sudo vi /sapmnt/QAS/profile/QAS_ERS01_anftstsapers
   
   # Change the restart command to a start command
   #Restart_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   Start_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   
   # remove Autostart from ERS profile
   # Autostart = 1
   ```


1. **[A]** Настройте активность.

   Обмен данными между сервером приложений SAP NetWeaver и ASCS/SCS происходит через программный балансировщик нагрузки. Балансировщик нагрузки отключает неактивные подключения по истечении времени ожидания, которое можно настроить. Чтобы избежать этого, необходимо задать параметр в профиле SAP NetWeaver ASCS/SCS, если используется ENSA1, и изменить системные `keepalive` Параметры Linux на всех серверах SAP как для ENSA1, так и для ENSA2. Дополнительные сведения см. в [примечании к SAP 1410736][1410736].

   ```
   # Change the Linux system configuration
   sudo sysctl net.ipv4.tcp_keepalive_time=300
   ```

1. **[A]** Изменение файла /usr/sap/sapservices

   Чтобы предотвратить запуск экземпляров скриптом запуска sapinit, необходимо закомментировать все экземпляры, управляемые Pacemaker, в файле /usr/sap/sapservices. Не следует закомментировать экземпляр SAP HANA, если он будет использоваться с HANA SR.

   ```
   sudo vi /usr/sap/sapservices
   
   # On the node where you installed the ASCS, comment out the following line
   # LD_LIBRARY_PATH=/usr/sap/QAS/ASCS00/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/QAS/ASCS00/exe/sapstartsrv pf=/usr/sap/QAS/SYS/profile/QAS_ASCS00_anftstsapvh -D -u qasadm
   
   # On the node where you installed the ERS, comment out the following line
   # LD_LIBRARY_PATH=/usr/sap/QAS/ERS01/exe:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; /usr/sap/QAS/ERS01/exe/sapstartsrv pf=/usr/sap/QAS/ERS01/profile/QAS_ERS01_anftstsapers -D -u qasadm
   ```

1. **[1]** Создайте кластерные ресурсы SAP.

   Если используется архитектура сервера постановки в очередь 1 (ENSA1), определите ресурсы следующим образом:

   ```
   sudo pcs property set maintenance-mode=true
   
    sudo pcs resource create rsc_sap_QAS_ASCS00 SAPInstance \
    InstanceName=QAS_ASCS00_anftstsapvh START_PROFILE="/sapmnt/QAS/profile/QAS_ASCS00_anftstsapvh" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
    --group g-QAS_ASCS
   
    sudo pcs resource create rsc_sap_QAS_ERS01 SAPInstance \
    InstanceName=QAS_ERS01_anftstsapers START_PROFILE="/sapmnt/QAS/profile/QAS_ERS01_anftstsapers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
    --group g-QAS_AERS
      
    sudo pcs constraint colocation add g-QAS_AERS with g-QAS_ASCS -5000
    sudo pcs constraint location rsc_sap_QAS_ASCS00 rule score=2000 runs_ers_QAS eq 1
    sudo pcs constraint order g-QAS_ASCS then g-QAS_AERS kind=Optional symmetrical=false
    
    sudo pcs node unstandby anftstsapcl1
    sudo pcs property set maintenance-mode=false
    ```

   Начиная с SAP NW 7.52 появилась поддержка сервера постановки в очередь 2, включая репликацию. Начиная с ABAP Platform 1809 сервер постановки в очередь 2 устанавливается по умолчанию. Сведения о поддержке сервера постановки в очередь 2 см. в примечании к SAP [2630416](https://launchpad.support.sap.com/#/notes/2630416).
   При использовании архитектуры Server 2 для постановки в очередь ([ENSA2](https://help.sap.com/viewer/cff8531bc1d9416d91bb6781e628d4e0/1709%20001/en-US/6d655c383abf4c129b0e5c8683e7ecd8.html)) установите агент ресурсов Resource-Agents-SAP-4.1.1-12.el7.x86_64 или более поздней версии и определите ресурсы следующим образом:

    ```
    sudo pcs property set maintenance-mode=true
    
    sudo pcs resource create rsc_sap_QAS_ASCS00 SAPInstance \
    InstanceName=QAS_ASCS00_anftstsapvh START_PROFILE="/sapmnt/QAS/profile/QAS_ASCS00_anftstsapvh" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 migration-threshold=1 failure-timeout=60 \
    op monitor interval=20 on-fail=restart timeout=60 \
    op start interval=0 timeout=600 op stop interval=0 timeout=600 \
    --group g-QAS_ASCS
   
    sudo pcs resource create rsc_sap_QAS_ERS01 SAPInstance \
    InstanceName=QAS_ERS01_anftstsapers START_PROFILE="/sapmnt/QAS/profile/QAS_ERS01_anftstsapers" \
    AUTOMATIC_RECOVER=false IS_ERS=true \
    op monitor interval=20 on-fail=restart timeout=60 op start interval=0 timeout=600 op stop interval=0 timeout=600 \
    --group g-QAS_AERS
      
    sudo pcs constraint colocation add g-QAS_AERS with g-QAS_ASCS -5000
    sudo pcs constraint order g-QAS_ASCS then g-QAS_AERS kind=Optional symmetrical=false
    sudo pcs constraint order start g-QAS_ASCS then stop g-QAS_AERS symmetrical=false
   
    sudo pcs node unstandby anftstsapcl1
    sudo pcs property set maintenance-mode=false
    ```

   Если вы обновляете старую версию и переходите на сервер очереди 2, см. Примечание SAP [2641322](https://launchpad.support.sap.com/#/notes/2641322). 

   > [!NOTE]
   > Время ожидания в приведенной выше конфигурации является просто примерами и может потребоваться адаптироваться к конкретной установке SAP. 

   Убедитесь, что состояние кластера — "ОК" и что запущены все ресурсы. Не важно, на каком узле выполняются ресурсы.

    ```
    sudo pcs status
    
    # Online: [ anftstsapcl1 anftstsapcl2 ]
    #
    # Full list of resources:
    #
    # rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl2
    #  Resource Group: g-QAS_ASCS
    #      fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
    #      nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
    #      vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
    #      rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    #  Resource Group: g-QAS_AERS
    #      fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
    #      nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
    #      vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
    #      rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

1. **[A]** добавьте правила брандмауэра для ASCS и ERS на обоих узлах. Добавьте правила брандмауэра для ASCS и ERS на обоих узлах.
   ```
   # Probe Port of ASCS
   sudo firewall-cmd --zone=public --add-port=62000/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=62000/tcp
   sudo firewall-cmd --zone=public --add-port=3200/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=3200/tcp
   sudo firewall-cmd --zone=public --add-port=3600/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=3600/tcp
   sudo firewall-cmd --zone=public --add-port=3900/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=3900/tcp
   sudo firewall-cmd --zone=public --add-port=8100/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=8100/tcp
   sudo firewall-cmd --zone=public --add-port=50013/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50013/tcp
   sudo firewall-cmd --zone=public --add-port=50014/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50014/tcp
   sudo firewall-cmd --zone=public --add-port=50016/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50016/tcp
   # Probe Port of ERS
   sudo firewall-cmd --zone=public --add-port=62101/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=62101/tcp
   sudo firewall-cmd --zone=public --add-port=3201/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=3201/tcp
   sudo firewall-cmd --zone=public --add-port=3301/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=3301/tcp
   sudo firewall-cmd --zone=public --add-port=50113/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50113/tcp
   sudo firewall-cmd --zone=public --add-port=50114/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50114/tcp
   sudo firewall-cmd --zone=public --add-port=50116/tcp --permanent
   sudo firewall-cmd --zone=public --add-port=50116/tcp
   ```

## <a name="sap-netweaver-application-server-preparation"></a><a name="2d6008b0-685d-426c-b59e-6cd281fd45d7"></a>Подготовка сервера приложений SAP NetWeaver

   Для некоторых баз данных требуется установить экземпляр базы данных на сервере приложений. Для этих случаев подготовьте виртуальные машины сервера приложений к их использованию.  

   В шагах ниже предполагается, что сервер приложений установлен на сервере, отличном от серверов ASCS/SCS и HANA. В противном случае некоторые описанные ниже шаги (например, настройка разрешения имени узла) не требуются.  

   Ниже приведены элементы с префиксами: **[A]** — применяется к PAS и AAS, **[P]**  — применяется только к PAS, **[S]**  — применяется только к узлу AAS.  

1. **[A]** настройте разрешение имен узлов. можно либо использовать DNS-сервер, либо изменить/etc/hosts на всех узлах. В этом примере показано, как использовать файл /etc/hosts.
   Замените IP-адрес и имя узла в следующих командах:  

   ```
   sudo vi /etc/hosts
   ```

   Вставьте следующие строки в /etc/hosts. Измените IP-адрес и имя узла в соответствии с вашей средой.

   ```
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS
   192.168.14.9 anftstsapvh
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS ERS
   192.168.14.10 anftstsapers
   192.168.14.7 anftstsapa01
   192.168.14.8 anftstsapa02
   ```

1. **[A]** создайте каталог sapmnt, чтобы создать каталог sapmnt.
   ```
   sudo mkdir -p /sapmnt/QAS
   sudo mkdir -p /usr/sap/trans

   sudo chattr +i /sapmnt/QAS
   sudo chattr +i /usr/sap/trans
   ```

1. **[A]** установите клиент NFS и другие требования  

   ```
   sudo yum -y install nfs-utils uuidd
   ```

1. **[A]** Добавление записей подключения  
   При использовании NFSv3:
   ```
   sudo vi /etc/fstab
   
   # Add the following lines to fstab, save and exit
   192.168.24.5:/sapQAS/sapmntQAS /sapmnt/QAS nfs rw,hard,rsize=65536,wsize=65536,vers=3
   192.168.24.4:/transSAP /usr/sap/trans nfs rw,hard,rsize=65536,wsize=65536,vers=3
   ```

   При использовании Нфсв 4.1:
   ```
   sudo vi /etc/fstab
   
   # Add the following lines to fstab, save and exit
   192.168.24.5:/sapQAS/sapmntQAS /sapmnt/QAS nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   192.168.24.4:/transSAP /usr/sap/trans nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   ```

   Подключение новых общих папок

   ```
   sudo mount -a
   ```

1. **[P]** создание и подключение каталога PAS  
   При использовании NFSv3:
   ```
   sudo mkdir -p /usr/sap/QAS/D02
   sudo chattr +i /usr/sap/QAS/D02
   
   sudo vi /etc/fstab
   # Add the following line to fstab
   92.168.24.5:/sapQAS/usrsapQASpas /usr/sap/QAS/D02 nfs rw,hard,rsize=65536,wsize=65536,vers=3
   
   # Mount
   sudo mount -a
   ```

   При использовании Нфсв 4.1:
   ```
   sudo mkdir -p /usr/sap/QAS/D02
   sudo chattr +i /usr/sap/QAS/D02
   
   sudo vi /etc/fstab
   # Add the following line to fstab
   92.168.24.5:/sapQAS/usrsapQASpas /usr/sap/QAS/D02 nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   
   # Mount
   sudo mount -a
   ```

1. **[S]** создание и подключение каталога консультантов  
   При использовании NFSv3:
   ```
   sudo mkdir -p /usr/sap/QAS/D03
   sudo chattr +i /usr/sap/QAS/D03
   
   sudo vi /etc/fstab
   # Add the following line to fstab
   92.168.24.5:/sapQAS/usrsapQASaas /usr/sap/QAS/D03 nfs rw,hard,rsize=65536,wsize=65536,vers=3
   
   # Mount
   sudo mount -a
   ```

   При использовании Нфсв 4.1:
   ```
   sudo mkdir -p /usr/sap/QAS/D03
   sudo chattr +i /usr/sap/QAS/D03
   
   sudo vi /etc/fstab
   # Add the following line to fstab
   92.168.24.5:/sapQAS/usrsapQASaas /usr/sap/QAS/D03 nfs rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys
   
   # Mount
   sudo mount -a
   ```

1. **[A]** Настройте файл подкачки.
 
   ```
   sudo vi /etc/waagent.conf
   
   # Set the property ResourceDisk.EnableSwap to y
   # Create and use swapfile on resource disk.
   ResourceDisk.EnableSwap=y
   
   # Set the size of the SWAP file with property ResourceDisk.SwapSizeMB
   # The free space of resource disk varies by virtual machine size. Make sure that you do not set a value that is too big. You can check the SWAP space with command swapon
   # Size of the swapfile.
   ResourceDisk.SwapSizeMB=2000
   ```

   Перезапустите агент, чтобы активировать изменения:

   ```
   sudo service waagent restart
   ```

## <a name="install-database"></a>Установка базы данных

В данном примере SAP NetWeaver уже установлен на SAP HANA. Для этой установки можно использовать любую поддерживаемую базу данных. Дополнительные сведения об установке SAP HANA в Azure см. в статье [высокий уровень доступности SAP HANA на виртуальных машинах Azure на Red Hat Enterprise Linux][sap-hana-ha] . For a list of supported databases, see [SAP Note 1928533][1928533] .

1. Запустите установку экземпляра базы данных SAP.

   Установите экземпляр базы данных SAP NetWeaver в качестве корневого, используя имя виртуального узла, которое сопоставляется с внешним IP-адресом подсистемы балансировки нагрузки для базы данных

   Чтобы разрешить непривилегированному пользователю подключаться к SAPinst, можно использовать параметр SAPINST_REMOTE_ACCESS_USER.

   ```
   sudo <swpm>/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin
   ```

## <a name="sap-netweaver-application-server-installation"></a>Установка сервера приложений SAP NetWeaver

Для установки сервера приложений SAP выполните следующие действия.

1. Подготовка приложения сервера

   Чтобы подготовить сервер приложений, следуйте инструкциям из раздела [Подготовка сервера приложений SAP NetWeaver](high-availability-guide-rhel.md#2d6008b0-685d-426c-b59e-6cd281fd45d7).

1. Установите сервер приложений SAP NetWeaver:

   Установите основной и дополнительный сервер приложений SAP NetWeaver:

   Чтобы разрешить непривилегированному пользователю подключаться к SAPinst, можно использовать параметр SAPINST_REMOTE_ACCESS_USER.

   ```
   sudo <swpm>/sapinst SAPINST_REMOTE_ACCESS_USER=sapadmin
   ```

1. Обновите безопасное хранилище SAP HANA.

   Обновите безопасное хранилище SAP HANA, чтобы оно указывало на виртуальное имя настройки репликации системы SAP HANA.

   Выполните следующую команду, чтобы вывести список записей в формате \<sapsid> ADM

   ```
   hdbuserstore List
   ```

   Это позволит вывести список всех записей. Он будет выглядеть так:
   ```
   DATA FILE       : /home/qasadm/.hdb/anftstsapa01/SSFS_HDB.DAT
   KEY FILE        : /home/qasadm/.hdb/anftstsapa01/SSFS_HDB.KEY
   
   KEY DEFAULT
     ENV : 192.168.14.4:30313
     USER: SAPABAP1
     DATABASE: QAS
   ```

   Выходные данные будут содержать IP-адрес записи по умолчанию, которая указывает на виртуальную машину, а не на IP-адрес балансировщика нагрузки. Эту запись необходимо изменить так, чтобы она указывала на имя виртуального узла балансировщика нагрузки. Убедитесь, что вы используете тот же порт (**30313** в выходных данных выше) и имя базы данных (**QAS**)!

   ```
   su - qasadm
   hdbuserstore SET DEFAULT qasdb:30313@QAS SAPABAP1 <password of ABAP schema>
   ```

## <a name="test-the-cluster-setup"></a>Проверка настройки кластера

1. Миграция экземпляра ASCS вручную

   Состояние ресурсов перед запуском теста:

   ```
    rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

   Выполните следующие команды от имени привилегированного пользователя, чтобы перенести экземпляр ASCS.

   ```
   [root@anftstsapcl1 ~]# pcs resource move rsc_sap_QAS_ASCS00
   
   [root@anftstsapcl1 ~]# pcs resource clear rsc_sap_QAS_ASCS00
   
   # Remove failed actions for the ERS that occurred as part of the migration
   [root@anftstsapcl1 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   Состояние ресурсов после теста:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

1. Имитирование сбоя узла

   Состояние ресурсов перед запуском теста:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

   Выполните следующую команду в качестве привилегированного пользователя на узле, где выполняется экземпляр ASCS.

   ```
   [root@anftstsapcl2 ~]# echo b > /proc/sysrq-trigger
   ```

   Состояние после повторного запуска узла должно выглядеть так:

   ```
   Online: [ anftstsapcl1 anftstsapcl2 ]
   
   Full list of resources:
   
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   
   Failed Actions:
   * rsc_sap_QAS_ERS01_monitor_11000 on anftstsapcl1 'not running' (7): call=45, status=complete, exitreason='',
   ```

   Выполните следующую команду, чтобы очистить ресурсы со сбоями.

   ```
   [root@anftstsapcl1 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   Состояние ресурсов после теста:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

1. Завершение процесса сервера сообщений

   Состояние ресурсов перед запуском теста:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```
   
   Выполните следующие команды от имени привилегированного пользователя, чтобы определить процесс сервера сообщений и завершить его.

   ```
   [root@anftstsapcl1 ~]# pgrep -f ms.sapQAS | xargs kill -9
   ```

   Если вы завершите работу сервера сообщений только раз, его перезапустит `sapstart`. Если вы завершите работу сервера достаточно много раз, Pacemaker в конечном итоге переместит экземпляр ASCS на другой узел. Выполните следующие команды от имени привилегированного пользователя для очистки состояния ресурсов экземпляра ERS и ASCS после теста.

   ```
   [root@anftstsapcl1 ~]# pcs resource cleanup rsc_sap_QAS_ASCS00
   [root@anftstsapcl1 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   Состояние ресурсов после теста:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

1. Завершение процесса сервера постановки в очередь

   Состояние ресурсов перед запуском теста:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   ```

   Выполните следующие команды от имени привилегированного пользователя на узле, где выполняется экземпляр ASCS, чтобы завершить работу сервера постановки в очередь.

   ```
   #If using ENSA1
   [root@anftstsapcl2 ~]# pgrep -f en.sapQAS | xargs kill -9
   #If using ENSA2
   [root@anftstsapcl2 ~]# pgrep -f enq.sapQAS | xargs kill -9
   ```

   Для экземпляра ASCS должна быть немедленно выполнена отработка отказа с переносом на другой узел. После запуска экземпляра ASCS экземпляр ERS должен также выполнить отработку отказа. Выполните следующие команды от имени привилегированного пользователя для очистки состояния ресурсов экземпляра ERS и ASCS после теста.

   ```
   [root@anftstsapcl2 ~]# pcs resource cleanup rsc_sap_QAS_ASCS00
   [root@anftstsapcl2 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   Состояние ресурсов после теста:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

1. Завершение процесса сервера постановки в очередь для репликации

   Состояние ресурсов перед запуском теста:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

   Выполните следующую команду от имени привилегированного пользователя на узле, где выполняется экземпляр ERS, чтобы завершить работу процесса сервера постановки в очередь для репликации.

   ```
   #If using ENSA1
   [root@anftstsapcl2 ~]# pgrep -f er.sapQAS | xargs kill -9
   #If using ENSA2
   [root@anftstsapcl2 ~]# pgrep -f enqr.sapQAS | xargs kill -9
   ```

   Если вы выполните команду только раз, `sapstart` перезапустит процесс. Если вы выполните ее достаточно часто, `sapstart` не перезапустит процесс и ресурс будет находиться в остановленном состоянии. Выполните следующие команды от имени привилегированного пользователя для очистки состояния ресурсов экземпляра ERS после теста.

   ```
   [root@anftstsapcl2 ~]# pcs resource cleanup rsc_sap_QAS_ERS01
   ```

   Состояние ресурсов после теста:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

1. Завершение процесса постановки sapstartsrv в очередь

   Состояние ресурсов перед запуском теста:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

   Выполните следующие команды в качестве привилегированного пользователя на узле, где выполняется ASCS.

   ```
   [root@anftstsapcl1 ~]# pgrep -fl ASCS00.*sapstartsrv
   # 59545 sapstartsrv
   
   [root@anftstsapcl1 ~]# kill -9 59545
   ```

   Процесс sapstartsrv всегда должен перезапускаться с помощью агента ресурсов Pacemaker в рамках мониторинга. Состояние ресурсов после теста:

   ```
   rsc_st_azure    (stonith:fence_azure_arm):      Started anftstsapcl1
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
    Resource Group: g-QAS_AERS
        fs_QAS_AERS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_AERS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_AERS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   ```

## <a name="next-steps"></a>Дальнейшие действия

* [Обеспечение высокого уровня доступности SAP NW на виртуальных машинах Azure в RHEL с несколькими ИД безопасности для приложений SAP](./high-availability-guide-rhel-multi-sid.md)
* [SAP NetWeaver на виртуальных машинах Windows. Руководство по планированию и внедрению][planning-guide]
* [Развертывание виртуальных машин Azure для SAP NetWeaver][deployment-guide]
* [SAP NetWeaver на виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server][dbms-guide]
* Дополнительные сведения об обеспечении высокого уровня доступности и планировании аварийного восстановления SAP HANA в Azure (крупные экземпляры) см. в [этой статье](hana-overview-high-availability-disaster-recovery.md).
* Дополнительные сведения об установке высокого уровня доступности и плана для аварийного восстановления SAP HANA на виртуальных машинах Azure см. в статье [Высокий уровень доступности SAP HANA на виртуальных машинах Azure][sap-hana-ha].