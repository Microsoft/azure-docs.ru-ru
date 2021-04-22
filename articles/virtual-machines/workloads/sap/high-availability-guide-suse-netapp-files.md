---
title: Обеспечение высокого уровня доступности виртуальных машин Azure для SAP NW на SLES с помощью Azure NetApp Files | Документация Майкрософт
description: Руководство по обеспечению высокой доступности для SAP NetWeaver на SUSE Linux Enterprise Server с помощью Azure NetApp Files для приложений SAP
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
ms.date: 04/12/2021
ms.author: radeltch
ms.openlocfilehash: eed971471ad8a91f09a812830eb83ac17bc59a01
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107307577"
---
# <a name="high-availability-for-sap-netweaver-on-azure-vms-on-suse-linux-enterprise-server-with-azure-netapp-files-for-sap-applications"></a>Обеспечение высокого уровня доступности виртуальных машин Azure для SAP NetWeaver на SUSE Linux Enterprise Server с помощью Azure NetApp Files для приложений SAP

[dbms-guide]:dbms_guide_general.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-azure-doc]:https://docs.microsoft.com/azure/azure-netapp-files/
[anf-avail-matrix]:https://azure.microsoft.com/global-infrastructure/services/?products=storage&regions=all
[anf-register]:https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register
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

В этой статье описывается развертывание виртуальных машин, их настройка, установка платформы кластера, а также установка высокодоступной системы SAP NetWeaver 7.50 с помощью [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md).
В примерах конфигураций, команд установки и т. д. экземпляру ASCS задан номер 00, экземпляру ERS — номер 01, экземпляру основного приложения (PAS) — номер 02, экземпляру приложения (AAS) — номер 03. Используется QAS с идентификатором системы SAP. 

В этой статье объясняется, как обеспечить высокий уровень доступности приложений SAP NetWeaver с помощью Azure NetApp Files. В этой статье нет подробных сведений об уровне базы данных.

Прежде всего прочитайте следующие примечания и документы SAP:

* [Документация по Azure NetApp Files][anf-azure-doc] 
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
* вики-сайт сообщества SAP (https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) ), содержащий все необходимые примечания к SAP для Linux;
* [SAP NetWeaver на виртуальных машинах Linux. Руководство по планированию и внедрению][planning-guide]
* [Развертывание программного обеспечения SAP на виртуальных машинах Linux в Azure][deployment-guide]
* [Общие сведения о развертывании СУБД на виртуальных машинах Azure для рабочих нагрузок SAP][dbms-guide]
* [Рекомендации по обеспечению высокого уровня доступности SUSE SAP][suse-ha-guide]. Эти руководства содержат всю необходимую информацию для настройки высокого уровня доступности NetWeaver и репликации системы SAP HANA в локальной среде. Придерживайтесь этих общих рекомендаций. Они содержат намного более подробные сведения.
* [Заметки о выпуске расширения SUSE высокого уровня доступности 12 SP3][suse-ha-12sp3-relnotes]
* [Приложения NetApp SAP в Microsoft Azure. Использование Azure NetApp Files][anf-sap-applications-azure]

## <a name="overview"></a>Обзор

Для обеспечения высокого уровня доступности (HA) центральных служб SAP NetWeaver требуется общее хранилище.
Ранее для выполнения этой задачи в SUSE Linux было необходимо создавать отдельный высокодоступный кластер NFS. 

Теперь обеспечить высокий уровень доступности SAP NetWeaver можно с помощью общего хранилища, развернутого в службе Azure NetApp Files. Использование службы Azure NetApp Files для общего хранилища избавляет от необходимости развертывания дополнительного [кластера NFS](./high-availability-guide-suse-nfs.md). Для обеспечения высокого уровня доступности центральных служб SAP NetWeaver (ASCS/SCS) по-прежнему требуется Pacemaker.


![Общие сведения о высоком уровне доступности SAP NetWeaver](./media/high-availability-guide-suse-anf/high-availability-guide-suse-anf.png)

SAP NetWeaver ASCS, SAP NetWeaver SCS, SAP NetWeaver ERS и база данных SAP HANA используют виртуальное имя узла и виртуальные IP-адреса. [Load Balancer](../../../load-balancer/load-balancer-overview.md) в Azure должен использовать виртуальный IP-адрес. Мы рекомендуем [Load Balancer (цен. категория "Стандартный")](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md). Ниже приведен список конфигурации балансировщика нагрузки (A)SCS и ERS.

### <a name="ascs"></a>(A)SCS

* Конфигурация внешнего интерфейса:
  * IP-адрес 10.1.1.20
* Порт пробы:
  * порт 620<strong>&lt;nr&gt;</strong>.
* Правила балансировки нагрузки
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
  * IP-адрес 10.1.1.21
* Порт пробы:
  * Порт 621<strong>&lt;nr&gt;</strong>.
* Правила балансировки нагрузки
  * Если используется Load Balancer (цен. категория "Стандартный"), выберите **порты высокой доступности**.
  * Если используется Load Balancer (цен. категория "Базовый"), создайте правила балансировки нагрузки для следующих портов.
    * 32<strong>&lt;nr&gt;</strong> TCP;
    * 33<strong>&lt;nr&gt;</strong> TCP;
    * 5<strong>&lt;nr&gt;</strong>13 TCP;
    * 5<strong>&lt;nr&gt;</strong>14 TCP;
    * 5<strong>&lt;nr&gt;</strong>16 TCP.

* Конфигурация серверной части:
  * подключена к основным сетевым интерфейсам всех виртуальных машин, которые должны быть частью кластера (A)SCS/ERS.


## <a name="setting-up-the-azure-netapp-files-infrastructure"></a>Настройка инфраструктуры Azure NetApp Files 

SAP NetWeaver требует общее хранилище для каталога профиля и транспорта.  Прежде чем продолжить настройку инфраструктуры Azure NetApp Files, ознакомьтесь с документацией по [Azure NetApp Files][anf-azure-doc]. Проверьте, предлагает ли выбранный регион Azure службу Azure NetApp Files. Перейдите по приведенной ниже ссылке, чтобы узнать о доступности службы Azure NetApp Files по регионам Azure: [Доступность Azure NetApp Files по региону Azure][anf-avail-matrix].

Служба Azure NetApp Files доступна в нескольких [регионах Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Перед развертыванием Azure NetApp Files запросите подключение к Azure NetApp Files, следуя инструкциям в статье [Регистрация в службе Azure NetApp Files][anf-register]. 

### <a name="deploy-azure-netapp-files-resources"></a>Развертывание ресурсов Azure NetApp Files  

Предполагается, что вы уже развернули [виртуальную сеть Azure](../../../virtual-network/virtual-networks-overview.md). Ресурсы Azure NetApp Files и виртуальные машины, к которым будут подключаться ресурсы Azure NetApp Files, должны быть развернуты в одной виртуальной сети Azure или в одноранговых виртуальных сетях Azure.  

1. Если вы еще не сделали этого, запросите [подключение к Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).  

2. Создайте учетную запись NetApp в выбранном регионе Azure, следуя [этим инструкциям](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md).  
3. Настройте пул емкости Azure NetApp Files, следуя [инструкциям по настройке пула емкости Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).  
В архитектуре SAP NetWeaver, представленной в этой статье, используется один пул емкости Azure NetApp Files с номером SKU категории "Премиум". Он является рекомендуемым для рабочей нагрузки приложения SAP NetWeaver в Azure.  

4. Делегируйте подсеть в Azure NetApp Files согласно [инструкциям по делегированию подсети в Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).  

5. Разверните тома Azure NetApp Files, следуя [инструкциям по созданию тома для Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md). Разверните тома в выделенной [ подсети](/rest/api/virtualnetwork/subnets) Azure NetApp Files. IP-адреса томов Azure NetApp Files назначаются автоматически. Учтите, что ресурсы Azure NetApp Files и виртуальные машины Azure должны находиться в одной виртуальной сети Azure или в одноранговых виртуальных сетях Azure. В этом примере используются два тома Azure NetApp Files: sap<b>QAS</b> и trans. Пути к файлам, подключенные к соответствующим точкам подключения, — /usrsap<b>qas</b>/sapmnt<b>QAS</b>, /usrsap<b>qas</b>/usrsap<b>QAS</b>sys и т. д.  

   1. volume sap<b>QAS</b> (nfs://10.1.0.4/usrsap<b>qas</b>/sapmnt<b>QAS</b>)
   2. volume sap<b>QAS</b> (nfs://10.1.0.4/usrsap<b>qas</b>/usrsap<b>QAS</b>ascs)
   3. volume sap<b>QAS</b> (nfs://10.1.0.4/usrsap<b>qas</b>/usrsap<b>QAS</b>sys)
   4. volume sap<b>QAS</b> (nfs://10.1.0.4/usrsap<b>qas</b>/usrsap<b>QAS</b>ers)
   5. volume trans (nfs://10.1.0.4/trans)
   6. volume sap<b>QAS</b> (nfs://10.1.0.4/usrsap<b>qas</b>/usrsap<b>QAS</b>pas)
   7. volume sap<b>QAS</b> (nfs://10.1.0.4/usrsap<b>qas</b>/usrsap<b>QAS</b>aas)

   
В этом примере мы использовали Azure NetApp Files для всех файловых систем SAP NetWeaver, чтобы продемонстрировать использование этой службы. Файловые системы SAP, которые не нужно подключать через NFS, можно также развернуть как [хранилище дисков Azure](../../disks-types.md#premium-ssd). В этом примере <b>a-e</b> должен быть в Azure NetApp Files, а <b>f-g</b> (т. е. /usr/sap/<b>QAS</b>/D<b>02</b>, /usr/sap/<b>QAS</b>/D<b>03</b>) можно развернуть в качестве хранилища дисков Azure. 

### <a name="important-considerations"></a>Важные сведения

При выборе Azure NetApp Files для архитектуры с высоким уровнем доступности SAP NetWeaver в SUSE учитывайте следующие важные моменты.

- Минимальный размер пула емкости равен 4 ТиБ. Размер пула емкости можно увеличить шагами по 1 ТиБ.
- Минимальный размер том — 100 ГиБ.
- Azure NetApp Files и все виртуальные машины, к которым будут подключаться тома Azure NetApp Files, должны быть развернуты в одной виртуальной сети Azure или в [одноранговых виртуальных сетях](../../../virtual-network/virtual-network-peering-overview.md) в одном регионе. В настоящее время поддерживается доступ к Azure NetApp Files через пиринг виртуальных сетей в том же регионе. Доступ к NetApp Azure через глобальный пиринг пока не поддерживается.
- В выбранной виртуальной сети должна быть подсеть, делегированная службе Azure NetApp Files.
- Azure NetApp Files предлагает [политику экспорта](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md), которая позволяет управлять разрешенными клиентами, типом доступа (чтение и запись, доступ только для чтения и т. д.). 
- Функция Azure NetApp Files пока еще не поддерживает зоны. В настоящее время Azure NetApp Files не развернута во всех зонах доступности региона Azure. Следует учитывать возможную задержку в некоторых регионах Azure. 
- Тома Azure NetApp Files можно развернуть в виде томов NFSv3 или NFSv4.1. Оба протокола поддерживаются на уровне приложений SAP (ASCS/ERS, серверы приложений SAP). 

## <a name="deploy-linux-vms-manually-via-azure-portal"></a>Развертывание виртуальных машин Linux вручную с помощью портала Azure

Сначала необходимо создать тома Azure NetApp Files. Разверните виртуальные машины. После этого следует создать подсистему балансировки нагрузки и использовать виртуальные машины во внутренних пулах.

1. Создание группы ресурсов
1. Создайте виртуальную сеть
1. Создайте группу доступности для ASCS.  
   Настройка максимального числа доменов обновления.
1. Создание виртуальной машины 1.  
   Используйте по меньшей мере SLES4SAP 12 с пакетом обновления 3 (SP3). В этом примере используется образ SLES4SAP 12 с пакетом обновления 3 (SP3).  
   Выберите ранее созданную группу доступности для ASCS.  
1. Создание виртуальной машины 2.  
   Используйте по меньшей мере SLES4SAP 12 с пакетом обновления 3 (SP3). В этом примере используется образ SLES4SAP 12 с пакетом обновления 3 (SP3).  
   Выберите ранее созданную группу доступности для ASCS.  
1. Создайте группу доступности для экземпляров приложений SAP (PAS, AAS).    
   Настройка максимального числа доменов обновления.
1. Создайте виртуальную машину 3.  
   Используйте по меньшей мере SLES4SAP 12 с пакетом обновления 3 (SP3). В этом примере используется образ SLES4SAP 12 с пакетом обновления 3 (SP3).  
   Выберите ранее созданную группу доступности для PAS/AAS.   
1. Создайте виртуальную машину 4.  
   Используйте по меньшей мере SLES4SAP 12 с пакетом обновления 3 (SP3). В этом примере используется образ SLES4SAP 12 с пакетом обновления 3 (SP3).  
   Выберите ранее созданную группу доступности для PAS/AAS.  

## <a name="disable-id-mapping-if-using-nfsv41"></a>Отключите сопоставление идентификаторов (если используется NFSv4.1).

Инструкции в этом разделе применимы, только если используются тома Azure NetApp Files с протоколом NFSv4.1. Выполните настройку на всех виртуальных машинах, к которым будут подключены тома Azure NetApp Files NFSv4.1.  

1. Проверьте параметр домена NFS. Убедитесь, что домен настроен в качестве домена Azure NetApp Files по умолчанию, т. е. **`defaultv4iddomain.com`** , а для сопоставления установлено значение **Никто**.  

    > [!IMPORTANT]
    > Обязательно укажите домен NFS в `/etc/idmapd.conf` виртуальной машине в соответствии с конфигурацией домена по умолчанию в Azure NetApp Files: **`defaultv4iddomain.com`** . В случае несоответствия между конфигурацией домена на клиенте NFS (т. е. виртуальной машине) и NFS-сервере (т. е. конфигурацией Azure NetApp) разрешения для файлов на томах NetApp в Azure, подключенных к виртуальным машинам, будут отображаться как `nobody`.  

    <pre><code>
    sudo cat /etc/idmapd.conf
    # Example
    [General]
    Verbosity = 0
    Pipefs-Directory = /var/lib/nfs/rpc_pipefs
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
    mount 10.1.0.4:/sapmnt/<b>qas</b> /mnt/tmp
    umount  /mnt/tmp
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    </code></pre>


## <a name="setting-up-ascs"></a>Настройка (A)SCS

В этом примере ресурсы были развернуты вручную с помощью [портала Azure](https://portal.azure.com/#home).

### <a name="deploy-azure-load-balancer-manually-via-azure-portal"></a>Развертывание Azure Load Balancer вручную с помощью портала Azure

Сначала необходимо создать тома Azure NetApp Files. Разверните виртуальные машины. После этого следует создать подсистему балансировки нагрузки и использовать виртуальные машины во внутренних пулах.

1. Создайте подсистему балансировки нагрузки (внутреннюю, ценовой категории "Стандартный").  
   1. Создайте IP-адреса внешнего интерфейса.
      1. IP-адрес 10.1.1.20 для ASCS
         1. Откройте подсистему балансировки нагрузки, выберите пул внешних IP-адресов и щелкните "Добавить".
         1. Введите имя нового внешнего пула IP-адресов (например, **frontend.QAS.ASCS**).
         1. Для параметра "Назначение" выберите значение "Статическое" и введите IP-адрес (например, **10.1.1.20**).
         1. Нажмите кнопку "ОК"
      1. IP-адрес 10.1.1.21 для ASCS ERS
         * Чтобы создать IP-адреса для ERS, повторите предыдущие шаги в разделе "а" (например, **10.1.1.21** и **frontend.QAS.ERS**).
   1. Создание внутреннего пула
      1. Выберите подсистему балансировки нагрузки, щелкните "Серверные пулы" и нажмите кнопку "Добавить".
      1. Введите имя нового внутреннего пула (например, **backend.QAS**).
      1. Щелкните "Добавить виртуальную машину".
      1. Выберите "Виртуальная машина".
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
      1. Создайте внутренний пул для ASCS.
         1. Откройте подсистему балансировки нагрузки, щелкните "Правила балансировки нагрузки" и нажмите кнопку "Добавить".
         1. Введите имя нового правила балансировки нагрузки (например, **lb.QAS.ASCS**).
         1. Выберите внешний IP-адрес для ASCS, внутренний пул и проверку работоспособности, созданные ранее (например, **frontend.QAS.ASCS**, **backend.QAS** и **health.QAS.ASCS**).
         1. Выберите **Порты высокой доступности**.
         1. **Не забудьте включить плавающий IP-адрес**.
         1. Нажмите кнопку "ОК"
         * Повторите описанные выше действия, чтобы создать правила балансировки нагрузки для ERS (например, **lb.QAS.ERS**).
1. Кроме того, если в сценарии требуется базовая подсистема балансировки нагрузки (внутренняя), выполните приведенные ниже действия.  
   1. Создайте IP-адреса внешнего интерфейса.
      1. IP-адрес 10.1.1.20 для ASCS
         1. Откройте подсистему балансировки нагрузки, выберите пул внешних IP-адресов и щелкните "Добавить".
         1. Введите имя нового внешнего пула IP-адресов (например, **frontend.QAS.ASCS**).
         1. Для параметра "Назначение" выберите значение "Статическое" и введите IP-адрес (например, **10.1.1.20**).
         1. Нажмите кнопку "ОК"
      1. IP-адрес 10.1.1.21 для ASCS ERS
         * Чтобы создать IP-адреса для ERS, повторите предыдущие шаги в разделе "а" (например, **10.1.1.21** и **frontend.QAS.ERS**).
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
         1. Откройте подсистему балансировки нагрузки, щелкните "Правила балансировки нагрузки" и нажмите кнопку "Добавить".
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
      > Плавающий IP-адрес не поддерживается для вторичных IP-конфигураций сетевых карт в сценариях с балансировкой нагрузки. Подробности см. в статье [Ограничения подсистемы балансировки нагрузки Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Если для виртуальной машины требуется дополнительный IP-адрес, следует развернуть вторую сетевую карту.  

      > [!Note]
      > Если в серверный пул внутреннего (без общедоступного IP-адреса) Azure Load Balancer ценовой категории "Стандартный" помещаются виртуальные машины без общедоступных IP-адресов, у них не будет исходящего подключения к Интернету без дополнительной настройки, разрешающей маршрутизацию к общедоступным конечным точкам. Подробные сведения о такой настройке см. в статье [Подключение к общедоступной конечной точке для виртуальных машин с помощью Azure Load Balancer (цен. категория "Стандартный") в сценариях обеспечения высокого уровня доступности SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md).  

      > [!IMPORTANT]
      > Не включайте метки времени TCP на виртуальных машинах Azure, размещенных за Azure Load Balancer. Включение меток времени TCP помешает работе проб работоспособности. Установите для параметра **net.ipv4.tcp_timestamps** значение **0**. Дополнительные сведения см. в статье [Пробы работоспособности Load Balancer](../../../load-balancer/load-balancer-custom-probe-overview.md).

### <a name="create-pacemaker-cluster"></a>Создание кластера Pacemaker

Следуйте указаниям в статье [Настройка кластера Pacemaker в SUSE Linux Enterprise Server в Azure](high-availability-guide-suse-pacemaker.md), чтобы создать базовый кластер Pacemaker для этого (A)SCS-сервера.

### <a name="installation"></a>Установка

Ниже приведены элементы с префиксами: **[A]**  — применяется ко всем узлам, **[1**] — применяется только к узлу 1, **[2]**  — применяется только к узлу 2.

1. **[A]** Установите соединитель SUSE.

   <pre><code>sudo zypper install sap-suse-cluster-connector
   </code></pre>

   > [!NOTE]
   > Известная проблема с использованием тире в именах узлов устранена в версии **3.1.1** пакета **sap-suse-cluster-connector**. Если вы работаете с узлами кластеров, содержащими тире в имени узла, следует использовать как минимум версию 3.1.1 пакета sap-suse-cluster-connector. Иначе кластер не будет работать. 

   Убедитесь, что вы установили новую версию соединителя кластера SAP SUSE. Старая версия называется sap_suse_cluster_connector, а новая — **sap-suse-cluster-connector**.

   <pre><code>sudo zypper info sap-suse-cluster-connector
   
    # Information for package sap-suse-cluster-connector:
    # ---------------------------------------------------
    # Repository     : SLE-12-SP3-SAP-Updates
    # Name           : sap-suse-cluster-connector
    # Version        : 3.1.0-8.1
    # Arch           : noarch
    # Vendor         : SUSE LLC &lt;https://www.suse.com/&gt;
    # Support Level  : Level 3
    # Installed Size : 45.6 KiB
    # Installed      : Yes
    # Status         : up-to-date
    # Source package : sap-suse-cluster-connector-3.1.0-8.1.src
    # Summary        : SUSE High Availability Setup for SAP Products
   </code></pre>

2. **[A]** Обновите агенты ресурсов SAP.  
   
   Для использования новой конфигурации, описанной в этой статье, нужно обязательно установить исправления для пакета агентов ресурсов. Вы можете проверить, установлено ли исправление, с помощью следующей команды:

   <pre><code>sudo grep 'parameter name="IS_ERS"' /usr/lib/ocf/resource.d/heartbeat/SAPInstance
   </code></pre>

   Вы должны получить примерно такой результат:

   <pre><code>&lt;parameter name="IS_ERS" unique="0" required="0"&gt;
   </code></pre>

   Если команде grep не удается найти параметр IS_ERS, необходимо установить исправления, перечисленные на [странице скачивания SUSE](https://download.suse.com/patch/finder/#bu=suse&familyId=&productId=&dateRange=&startDate=&endDate=&priority=&architecture=&keywords=resource-agents).

   <pre><code># example for patch for SLES 12 SP1
   sudo zypper in -t patch SUSE-SLE-HA-12-SP1-2017-885=1
   # example for patch for SLES 12 SP2
   sudo zypper in -t patch SUSE-SLE-HA-12-SP2-2017-886=1
   </code></pre>

3. **[A]** Установите разрешения имен.

   Можно использовать DNS-сервер или внести изменения в файл /etc/hosts на всех узлах. В этом примере показано, как использовать файл /etc/hosts.
   Замените IP-адрес и имя узла в следующих командах.

   <pre><code>sudo vi /etc/hosts
   </code></pre>

   Вставьте следующие строки в /etc/hosts. Измените IP-адрес и имя узла в соответствии со своей средой.   

   <pre><code>
   # IP address of cluster node 1
   <b>10.1.1.18    anftstsapcl1</b>
   # IP address of cluster node 2
   <b>10.1.1.6     anftstsapcl2</b>
   # IP address of the load balancer frontend configuration for SAP Netweaver ASCS
   <b>10.1.1.20    anftstsapvh</b>
   # IP address of the load balancer frontend configuration for SAP Netweaver ERS
   <b>10.1.1.21    anftstsapers</b>
   </code></pre>

4. **[1]** Создайте каталоги SAP в томе Azure NetApp Files.  
   Временно подключите том Azure NetApp Files к одной из виртуальных машин и создайте каталоги SAP (пути к файлам).  

   ```
    # mount temporarily the volume
    sudo mkdir -p /saptmp
    # If using NFSv3
    sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 10.1.0.4:/sapQAS /saptmp
    # If using NFSv4.1
    sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys,tcp 10.1.0.4:/sapQAS /saptmp
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

## <a name="prepare-for-sap-netweaver-installation"></a>Подготовка к установке SAP NetWeaver

1. **[A]** Создайте общие папки.

   <pre><code>sudo mkdir -p /sapmnt/<b>QAS</b>
   sudo mkdir -p /usr/sap/trans
   sudo mkdir -p /usr/sap/<b>QAS</b>/SYS
   sudo mkdir -p /usr/sap/<b>QAS</b>/ASCS<b>00</b>
   sudo mkdir -p /usr/sap/<b>QAS</b>/ERS<b>01</b>
   
   sudo chattr +i /sapmnt/<b>QAS</b>
   sudo chattr +i /usr/sap/trans
   sudo chattr +i /usr/sap/<b>QAS</b>/SYS
   sudo chattr +i /usr/sap/<b>QAS</b>/ASCS<b>00</b>
   sudo chattr +i /usr/sap/<b>QAS</b>/ERS<b>01</b>
   </code></pre>

2. **[A]** Настройте `autofs`.

   <pre><code>
   sudo vi /etc/auto.master
   # Add the following line to the file, save and exit
   /- /etc/auto.direct
   </code></pre>

   Если используется NFSv3, создайте файл со следующим текстом:

   <pre><code>
   sudo vi /etc/auto.direct
   # Add the following lines to the file, save and exit
   /sapmnt/<b>QAS</b> -nfsvers=3,nobind 10.1.0.4:/usrsap<b>qas</b>/sapmnt<b>QAS</b>
   /usr/sap/trans -nfsvers=3,nobind 10.1.0.4:/trans
   /usr/sap/<b>QAS</b>/SYS -nfsvers=3,nobind 10.1.0.4:/usrsap<b>qas</b>/usrsap<b>QAS</b>sys
   </code></pre>
   
   Если используется NFSv4.1, создайте файл со следующим текстом:

   <pre><code>
   sudo vi /etc/auto.direct
   # Add the following lines to the file, save and exit
   /sapmnt/<b>QAS</b> -nfsvers=4.1,nobind,sec=sys 10.1.0.4:/usrsap<b>qas</b>/sapmnt<b>QAS</b>
   /usr/sap/trans -nfsvers=4.1,nobind,sec=sys 10.1.0.4:/trans
   /usr/sap/<b>QAS</b>/SYS -nfsvers=4.1,nobind,sec=sys 10.1.0.4:/usrsap<b>qas</b>/usrsap<b>QAS</b>sys
   </code></pre>
   
   > [!NOTE]
   > При подключении томов Azure NetApp Files следует использовать соответствующую версию протокола NFS. Если тома Azure NetApp Files созданы как тома NFSv3, применяйте соответствующую конфигурацию NFSv3. Если тома Azure NetApp Files созданы как тома NFSv4.1, следуйте инструкциям по отключению сопоставления идентификаторов и обязательно используйте соответствующую конфигурацию NFSv4.1. В этом примере тома Azure NetApp Files были созданы как тома NFSv3.  
   
   Перезапустите `autofs`, чтобы подключить новые общие папки.
    <pre><code>
      sudo systemctl enable autofs
      sudo service autofs restart
     </code></pre>

3. **[A]** Настройте файл подкачки.

   <pre><code>sudo vi /etc/waagent.conf
   
   # Set the property ResourceDisk.EnableSwap to y
   # Create and use swapfile on resource disk.
   ResourceDisk.EnableSwap=<b>y</b>
   
   # Set the size of the SWAP file with property ResourceDisk.SwapSizeMB
   # The free space of resource disk varies by virtual machine size. Make sure that you do not set a value that is too big. You can check the SWAP space with command swapon
   # Size of the swapfile.
   ResourceDisk.SwapSizeMB=<b>2000</b>
   </code></pre>

   Перезапустите агент, чтобы активировать изменения:

   <pre><code>sudo service waagent restart
   </code></pre>

### <a name="installing-sap-netweaver-ascsers"></a>Установка SAP NetWeaver ASCS/ERS

1. **[1]** Создайте виртуальный IP-адрес и проверку работоспособности для экземпляра ASCS.

   > [!IMPORTANT]
   > В ходе последних тестов были выявлены ситуации, когда netcat перестает отвечать на запросы из-за невыполненной работы и ограничения на обработку только одного подключения. Ресурс netcat прекращает прослушивать запросы Azure Load Balancer, и плавающий IP-адрес становится недоступным.  
   > Для существующих кластеров Pacemaker ранее рекомендовалось заменять netcat на socat. Сейчас мы рекомендуем использовать агент ресурса azure-lb, который входит в состав пакета resource-agents со следующими требованиями к версии пакета:
   > - Минимальная версия для SLES 12 с пактом обновления 4 или 5 (SP4/SP5) — resource-agents-4.3.018.a7fb5035-3.30.1.  
   > - Минимальная версия для SLES 15/15 с пакетом обновления 1 (SP1) — resource-agents-4.3.0184.6ee15eb2-4.13.1.  
   >
   > Обратите внимание, что для этого изменения потребуется незначительное время простоя.  
   > Если конфигурация существующих кластеров Pacemaker уже была изменена для использования socat, как описано в [этой статье](https://www.suse.com/support/kb/doc/?id=7024128), немедленно переключаться на агент ресурса azure-lb не нужно.

   <pre><code>sudo crm node standby <b>anftstsapcl2</b>
   # If using NFSv3
   sudo crm configure primitive fs_<b>QAS</b>_ASCS Filesystem device='<b>10.1.0.4</b>/usrsap<b>qas</b>/usrsap<b>QAS</b>ascs' directory='/usr/sap/<b>QAS</b>/ASCS<b>00</b>' fstype='nfs' \
     op start timeout=60s interval=0 \
     op stop timeout=60s interval=0 \
     op monitor interval=20s timeout=40s
   
   # If using NFSv4.1
   sudo crm configure primitive fs_<b>QAS</b>_ASCS Filesystem device='<b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>ascs' directory='/usr/sap/<b>QAS</b>/ASCS<b>00</b>' fstype='nfs' options='sec=sys,vers=4.1' \
     op start timeout=60s interval=0 \
     op stop timeout=60s interval=0 \
     op monitor interval=20s timeout=40s
   
   sudo crm configure primitive vip_<b>QAS</b>_ASCS IPaddr2 \
     params ip=<b>10.1.1.20</b> cidr_netmask=<b>24</b> \
     op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>QAS</b>_ASCS azure-lb port=620<b>00</b>
   
   sudo crm configure group g-<b>QAS</b>_ASCS fs_<b>QAS</b>_ASCS nc_<b>QAS</b>_ASCS vip_<b>QAS</b>_ASCS \
      meta resource-stickiness=3000
   </code></pre>

   Убедитесь, что состояние кластера — "ОК" и что запущены все ресурсы. Не важно, на каком узле выполняются ресурсы.

   <pre><code>sudo crm_mon -r
   
   # Node anftstsapcl2: standby
   # <b>Online: [ anftstsapcl1 ]</b>
   # 
   # Full list of resources:
   #
   # Resource Group: g-QAS_ASCS
   #     fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    <b>Started anftstsapcl1</b>
   #     nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      <b>Started anftstsapcl1</b>
   #     vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       <b>Started anftstsapcl1</b>
   # stonith-sbd     (stonith:external/sbd): <b>Started anftstsapcl2</b>
   </code></pre>
  
2. **[1]** Установите SAP NetWeaver ASCS.  

   Установите SAP NetWeaver ASCS в качестве корневого на первом узле, используя имя виртуального узла, которое сопоставляется с внешним IP-адресом подсистемы балансировки нагрузки ASCS (например, <b>anftstsapvh</b>, <b>10.1.1.20</b>) и экземпляром номера, который использовался для проверки подсистемы балансировки нагрузки, например <b>00</b>.

   Чтобы разрешить непривилегированному пользователю подключаться к SAPinst, можно использовать параметр SAPINST_REMOTE_ACCESS_USER. Для установки SAP можно использовать параметр SAPINST_USE_HOSTNAME с именем виртуального узла.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b> SAPINST_USE_HOSTNAME=<b>virtual_hostname</b>
   </code></pre>

   Если установка завершилась ошибкой при создании вложенной папки в /usr/sap/**QAS**/ASCS **00**, попробуйте задать владельца и группу ASCS папки **00** и повторите заново. 

   <pre><code>
   chown <b>qas</b>adm /usr/sap/<b>QAS</b>/ASCS<b>00</b>
   chgrp sapsys /usr/sap/<b>QAS</b>/ASCS<b>00</b>
   </code></pre>

3. **[1]** Создайте виртуальный IP-адрес и проверку работоспособности для экземпляра ERS.

   <pre><code>
   sudo crm node online <b>anftstsapcl2</b>
   sudo crm node standby <b>anftstsapcl1</b>
   # If using NFSv3
   sudo crm configure primitive fs_<b>QAS</b>_ERS Filesystem device='<b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>ers' directory='/usr/sap/<b>QAS</b>/ERS<b>01</b>' fstype='nfs' \
     op start timeout=60s interval=0 \
     op stop timeout=60s interval=0 \
     op monitor interval=20s timeout=40s
   
   # If using NFSv4.1
   sudo crm configure primitive fs_<b>QAS</b>_ERS Filesystem device='<b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>ers' directory='/usr/sap/<b>QAS</b>/ERS<b>01</b>' fstype='nfs' options='sec=sys,vers=4.1' \
     op start timeout=60s interval=0 \
     op stop timeout=60s interval=0 \
     op monitor interval=20s timeout=40s
   
   sudo crm configure primitive vip_<b>QAS</b>_ERS IPaddr2 \
     params ip=<b>10.1.1.21</b> cidr_netmask=<b>24</b> \
     op monitor interval=10 timeout=20
   
   sudo crm configure primitive nc_<b>QAS</b>_ERS azure-lb port=621<b>01</b>
   
   sudo crm configure group g-<b>QAS</b>_ERS fs_<b>QAS</b>_ERS nc_<b>QAS</b>_ERS vip_<b>QAS</b>_ERS
   </code></pre>

   Убедитесь, что состояние кластера — "ОК" и что запущены все ресурсы. Не важно, на каком узле выполняются ресурсы.

   <pre><code>sudo crm_mon -r
   
   # Node <b>anftstsapcl1: standby</b>
   # <b>Online: [ anftstsapcl2 ]</b>
   # 
   # Full list of resources:
   #
   # stonith-sbd     (stonith:external/sbd): <b>Started anftstsapcl2</b>
   #  Resource Group: g-QAS_ASCS
   #      fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    <b>Started anftstsapcl2</b>
   #      nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      <b>Started anftstsapcl2</b>
   #      vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       <b>Started anftstsapcl2</b>
   #  Resource Group: g-QAS_ERS
   #      fs_QAS_ERS (ocf::heartbeat:Filesystem):    <b>Started anftstsapcl2</b>
   #      nc_QAS_ERS (ocf::heartbeat:azure-lb):      <b>Started anftstsapcl2</b>
   #      vip_QAS_ERS  (ocf::heartbeat:IPaddr2):     <b>Started anftstsapcl2</b>
   </code></pre>

4. **[2]** Установите SAP NetWeaver ERS.

   Установите SAP NetWeaver ERS в качестве корневого на втором узле, используя имя виртуального узла, которое сопоставляется с внешним IP-адресом подсистемы балансировки нагрузки ERS (например, <b>anftstsapers</b>, <b>10.1.1.21</b>) и экземпляром номера, который использовался для проверки подсистемы балансировки нагрузки, например <b>01</b>.

   Чтобы разрешить непривилегированному пользователю подключаться к SAPinst, можно использовать параметр SAPINST_REMOTE_ACCESS_USER. Для установки SAP можно использовать параметр SAPINST_USE_HOSTNAME с именем виртуального узла.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b> SAPINST_USE_HOSTNAME=<b>virtual_hostname</b>
   </code></pre>

   > [!NOTE]
   > Используйте SWPM SP 20 PL 05 или более поздней версии. Более ранние версии задают разрешения неправильно, и установка завершится с ошибкой.

   Если установка завершилась ошибкой при создании вложенной папки в /usr/sap/**QAS**/ERS **01**, попробуйте задать владельца и группу ERS папки **01** и повторите заново.

   <pre><code>
   chown qasadm /usr/sap/<b>QAS</b>/ERS<b>01</b>
   chgrp sapsys /usr/sap/<b>QAS</b>/ERS<b>01</b>
   </code></pre>


5. **[1]** Адаптируйте профили экземпляров ASCS/SCS и ERS.
 
   * Профиль ASCS/SCS

   <pre><code>
   sudo vi /sapmnt/<b>QAS</b>/profile/<b>QAS</b>_<b>ASCS00</b>_<b>anftstsapvh</b>
   
   # Change the restart command to a start command
   #Restart_Program_01 = local $(_EN) pf=$(_PF)
   Start_Program_01 = local $(_EN) pf=$(_PF)
   
   # Add the following lines
   service/halib = $(DIR_CT_RUN)/saphascriptco.so
   service/halib_cluster_connector = /usr/bin/sap_suse_cluster_connector
   
   # Add the keep alive parameter, if using ENSA1
   enque/encni/set_so_keepalive = true
   </code></pre>

   Убедитесь, что для ENSA1 и ENSA2 параметры ОС `keepalive` заданы так, как описано в примечании к SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).  

   * Профиль ERS

   <pre><code>
   sudo vi /sapmnt/<b>QAS</b>/profile/<b>QAS</b>_ERS<b>01</b>_<b>anftstsapers</b>
   
   # Change the restart command to a start command
   #Restart_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   Start_Program_00 = local $(_ER) pf=$(_PFL) NR=$(SCSID)
   
   # Add the following lines
   service/halib = $(DIR_CT_RUN)/saphascriptco.so
   service/halib_cluster_connector = /usr/bin/sap_suse_cluster_connector
   
   # remove Autostart from ERS profile
   # Autostart = 1
   </code></pre>

6. **[A]** Настройте активность.

   Обмен данными между сервером приложений SAP NetWeaver и ASCS/SCS происходит через программный балансировщик нагрузки. Балансировщик нагрузки отключает неактивные подключения по истечении времени ожидания, которое можно настроить. Чтобы исключить такое функционирование, нужно задать соответствующий параметр в профиле SAP NetWeaver ASCS/SCS, если используется ENSA1, и изменить системные параметры Linux `keepalive` на всех серверах SAP как для ENSA1, так и для ENSA2. Дополнительные сведения см. в [примечании к SAP 1410736][1410736].

   <pre><code>
   # Change the Linux system configuration
   sudo sysctl net.ipv4.tcp_keepalive_time=300
   </code></pre>

7. **[A]** Настройте пользователей SAP после установки.

   <pre><code>
   # Add sidadm to the haclient group
   sudo usermod -aG haclient <b>qas</b>adm
   </code></pre>

8. **[1]** Добавьте службы SAP ASCS и ERS в файл `sapservice`.

   Добавьте запись службы ASCS во второй узел и скопируйте запись службы ERS в первый узел.

   <pre><code>
   cat /usr/sap/sapservices | grep ASCS<b>00</b> | sudo ssh <b>anftstsapcl2</b> "cat >>/usr/sap/sapservices"
   sudo ssh <b>anftstsapcl2</b> "cat /usr/sap/sapservices" | grep ERS<b>01</b> | sudo tee -a /usr/sap/sapservices
   </code></pre>

9. **[1]** Создайте кластерные ресурсы SAP.

Если используется архитектура сервера постановки в очередь 1 (ENSA1), определите ресурсы следующим образом:

   <pre><code>sudo crm configure property maintenance-mode="true"
   
   sudo crm configure primitive rsc_sap_<b>QAS</b>_ASCS<b>00</b> SAPInstance \
    operations \$id=rsc_sap_<b>QAS</b>_ASCS<b>00</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>QAS</b>_ASCS<b>00</b>_<b>anftstsapvh</b> START_PROFILE="/sapmnt/<b>QAS</b>/profile/<b>QAS</b>_ASCS<b>00</b>_<b>anftstsapvh</b>" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000 failure-timeout=60 migration-threshold=1 priority=10
   
   sudo crm configure primitive rsc_sap_<b>QAS</b>_ERS<b>01</b> SAPInstance \
    operations \$id=rsc_sap_<b>QAS</b>_ERS<b>01</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>QAS</b>_ERS<b>01</b>_<b>anftstsapers</b> START_PROFILE="/sapmnt/<b>QAS</b>/profile/<b>QAS</b>_ERS<b>01</b>_<b>anftstsapers</b>" AUTOMATIC_RECOVER=false IS_ERS=true \
    meta priority=1000
   
   sudo crm configure modgroup g-<b>QAS</b>_ASCS add rsc_sap_<b>QAS</b>_ASCS<b>00</b>
   sudo crm configure modgroup g-<b>QAS</b>_ERS add rsc_sap_<b>QAS</b>_ERS<b>01</b>
   
   sudo crm configure colocation col_sap_<b>QAS</b>_no_both -5000: g-<b>QAS</b>_ERS g-<b>QAS</b>_ASCS
   sudo crm configure location loc_sap_<b>QAS</b>_failover_to_ers rsc_sap_<b>QAS</b>_ASCS<b>00</b> rule 2000: runs_ers_<b>QAS</b> eq 1
   sudo crm configure order ord_sap_<b>QAS</b>_first_start_ascs Optional: rsc_sap_<b>QAS</b>_ASCS<b>00</b>:start rsc_sap_<b>QAS</b>_ERS<b>01</b>:stop symmetrical=false
   
   sudo crm node online <b>anftstsapcl1</b>
   sudo crm configure property maintenance-mode="false"
   </code></pre>

   Начиная с SAP NW 7.52 появилась поддержка сервера постановки в очередь 2, включая репликацию. Начиная с ABAP Platform 1809 сервер постановки в очередь 2 устанавливается по умолчанию. Сведения о поддержке сервера постановки в очередь 2 см. в примечании к SAP [2630416](https://launchpad.support.sap.com/#/notes/2630416).
Если используется архитектура сервера постановки в очередь 2 ([ENSA2](https://help.sap.com/viewer/cff8531bc1d9416d91bb6781e628d4e0/1709%20001/en-US/6d655c383abf4c129b0e5c8683e7ecd8.html)), определите ресурсы следующим образом.

   <pre><code>sudo crm configure property maintenance-mode="true"
   
   sudo crm configure primitive rsc_sap_<b>QAS</b>_ASCS<b>00</b> SAPInstance \
    operations \$id=rsc_sap_<b>QAS</b>_ASCS<b>00</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>QAS</b>_ASCS<b>00</b>_<b>anftstsapvh</b> START_PROFILE="/sapmnt/<b>QAS</b>/profile/<b>QAS</b>_ASCS<b>00</b>_<b>anftstsapvh</b>" \
    AUTOMATIC_RECOVER=false \
    meta resource-stickiness=5000
   
   sudo crm configure primitive rsc_sap_<b>QAS</b>_ERS<b>01</b> SAPInstance \
    operations \$id=rsc_sap_<b>QAS</b>_ERS<b>01</b>-operations \
    op monitor interval=11 timeout=60 on-fail=restart \
    params InstanceName=<b>QAS</b>_ERS<b>01</b>_<b>anftstsapers</b> START_PROFILE="/sapmnt/<b>QAS</b>/profile/<b>QAS</b>_ERS<b>01</b>_<b>anftstsapers</b>" AUTOMATIC_RECOVER=false IS_ERS=true
   
   sudo crm configure modgroup g-<b>QAS</b>_ASCS add rsc_sap_<b>QAS</b>_ASCS<b>00</b>
   sudo crm configure modgroup g-<b>QAS</b>_ERS add rsc_sap_<b>QAS</b>_ERS<b>01</b>
   
   sudo crm configure colocation col_sap_<b>QAS</b>_no_both -5000: g-<b>QAS</b>_ERS g-<b>QAS</b>_ASCS
   sudo crm configure order ord_sap_<b>QAS</b>_first_start_ascs Optional: rsc_sap_<b>QAS</b>_ASCS<b>00</b>:start rsc_sap_<b>QAS</b>_ERS<b>01</b>:stop symmetrical=false
   
   sudo crm node online <b>anftstsapcl1</b>
   sudo crm configure property maintenance-mode="false"
   </code></pre>

   При переходе на более новую версию сервера постановки в очередь 2 см. примечание к SAP [2641019](https://launchpad.support.sap.com/#/notes/2641019). 

   Убедитесь, что состояние кластера — "ОК" и что запущены все ресурсы. Не важно, на каком узле выполняются ресурсы.

   <pre><code>sudo crm_mon -r
   # Full list of resources:
   #
   # stonith-sbd     (stonith:external/sbd): <b>Started anftstsapcl2</b>
   #  Resource Group: g-QAS_ASCS
   #      fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    <b>Started anftstsapcl1</b>
   #      nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      <b>Started anftstsapcl1</b>
   #      vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       <b>Started anftstsapcl1</b>
   #      rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   <b>Started anftstsapcl1</b>
   #  Resource Group: g-QAS_ERS
   #      fs_QAS_ERS (ocf::heartbeat:Filesystem):    <b>Started anftstsapcl2</b>
   #      nc_QAS_ERS (ocf::heartbeat:azure-lb):      <b>Started anftstsapcl2</b>
   #      vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       <b>Started anftstsapcl2</b>
   #      rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   <b>Started anftstsapcl2</b>
   </code></pre>

## <a name="sap-netweaver-application-server-preparation"></a><a name="2d6008b0-685d-426c-b59e-6cd281fd45d7"></a>Подготовка сервера приложений SAP NetWeaver 

Для некоторых баз данных требуется установить экземпляр базы данных на сервере приложений. Для этих случаев подготовьте виртуальные машины сервера приложений к их использованию.

В шагах ниже предполагается, что сервер приложений установлен на сервере, отличном от серверов ASCS/SCS и HANA. В противном случае некоторые описанные ниже шаги (например, настройка разрешения имени узла) не требуются.

Ниже приведены элементы с префиксами: **[A]** — применяется к PAS и AAS, **[P]**  — применяется только к PAS, **[S]**  — применяется только к узлу AAS.


1. **[A]** Настройка операционной системы

   Уменьшите размер "грязного" кэша. Дополнительные сведения см. в статье [Low write performance on SLES 11/12 servers with large RAM](https://www.suse.com/support/kb/doc/?id=7010287) (Низкая производительность записи на серверах SLES 11/12 с большим объем ОЗУ).

   <pre><code>
   sudo vi /etc/sysctl.conf
   # Change/set the following settings
   vm.dirty_bytes = 629145600
   vm.dirty_background_bytes = 314572800
   </code></pre>

1. **[A]** Установите разрешения имен.

   Можно использовать DNS-сервер или внести изменения в файл /etc/hosts на всех узлах. В этом примере показано, как использовать файл /etc/hosts.
   Замените IP-адрес и имя узла в следующих командах.

   ```bash
   sudo vi /etc/hosts
   ```

   Вставьте следующие строки в /etc/hosts. Измените IP-адрес и имя узла в соответствии со своей средой.

   <pre><code>
   # IP address of the load balancer frontend configuration for SAP NetWeaver ASCS/SCS
   <b>10.1.1.20 anftstsapvh</b>
   # IP address of the load balancer frontend configuration for SAP NetWeaver ERS
   <b>10.1.1.21 anftstsapers</b>
   # IP address of all application servers
   <b>10.1.1.15 anftstsapa01</b>
   <b>10.1.1.16 anftstsapa02</b>
   </code></pre>

1. **[A]** Создайте каталог sapmnt.

   <pre><code>
   sudo mkdir -p /sapmnt/<b>QAS</b>
   sudo mkdir -p /usr/sap/trans

   sudo chattr +i /sapmnt/<b>QAS</b>
   sudo chattr +i /usr/sap/trans
   </code></pre>

1. **[P]** Создайте каталог PAS.

   <pre><code>
   sudo mkdir -p /usr/sap/<b>QAS</b>/D<b>02</b>
   sudo chattr +i /usr/sap/<b>QAS</b>/D<b>02</b>
   </code></pre>

1. **[S]** Создайте каталог AAS.

   <pre><code>
   sudo mkdir -p /usr/sap/<b>QAS</b>/D<b>03</b>
   sudo chattr +i /usr/sap/<b>QAS</b>/D<b>03</b>
   </code></pre>

1. **[P]** Настройте `autofs` в PAS.

   <pre><code>sudo vi /etc/auto.master
   
   # Add the following line to the file, save and exit
   /- /etc/auto.direct
   </code></pre>

   Если используется NFSv3, создайте файл со следующим текстом:

   <pre><code>
   sudo vi /etc/auto.direct
   # Add the following lines to the file, save and exit
   /sapmnt/<b>QAS</b> -nfsvers=3,nobind <b>10.1.0.4</b>:/usrsap<b>qas</b>/sapmnt<b>QAS</b>
   /usr/sap/trans -nfsvers=3,nobind <b>10.1.0.4</b>:/trans
   /usr/sap/<b>QAS</b>/D<b>02</b> -nfsvers=3,nobind <b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>pas
   </code></pre>

   Если используется NFSv4.1, создайте файл со следующим текстом:

   <pre><code>
   sudo vi /etc/auto.direct
   # Add the following lines to the file, save and exit
   /sapmnt/<b>QAS</b> -nfsvers=4.1,nobind,sec=sys <b>10.1.0.4</b>:/usrsap<b>qas</b>/sapmnt<b>QAS</b>
   /usr/sap/trans -nfsvers=4.1,nobind,sec=sys <b>10.1.0.4</b>:/trans
   /usr/sap/<b>QAS</b>/D<b>02</b> -nfsvers=4.1,nobind,sec=sys <b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>pas
   </code></pre>

   Перезапустите `autofs`, чтобы подключить новые общие папки.

   <pre><code>
   sudo systemctl enable autofs
   sudo service autofs restart
   </code></pre>

1. **[P]** Настройте `autofs` в AAS.

   <pre><code>sudo vi /etc/auto.master
   
   # Add the following line to the file, save and exit
   /- /etc/auto.direct
   </code></pre>

   Если используется NFSv3, создайте файл со следующим текстом:

   <pre><code>
   sudo vi /etc/auto.direct
   # Add the following lines to the file, save and exit
   /sapmnt/<b>QAS</b> -nfsvers=3,nobind <b>10.1.0.4</b>:/usrsap<b>qas</b>/sapmnt<b>QAS</b>
   /usr/sap/trans -nfsvers=3,nobind <b>10.1.0.4</b>:/trans
   /usr/sap/<b>QAS</b>/D<b>03</b> -nfsvers=3,nobind <b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>aas
   </code></pre>

   Если используется NFSv4.1, создайте файл со следующим текстом:

   <pre><code>
   sudo vi /etc/auto.direct
   # Add the following lines to the file, save and exit
   /sapmnt/<b>QAS</b> -nfsvers=4.1,nobind,sec=sys <b>10.1.0.4</b>:/usrsap<b>qas</b>/sapmnt<b>QAS</b>
   /usr/sap/trans -nfsvers=4.1,nobind,sec=sys <b>10.1.0.4</b>:/trans
   /usr/sap/<b>QAS</b>/D<b>03</b> -nfsvers=4.1,nobind,sec=sys <b>10.1.0.4</b>:/usrsap<b>qas</b>/usrsap<b>QAS</b>aas
   </code></pre>

   Перезапустите `autofs`, чтобы подключить новые общие папки.

   <pre><code>
   sudo systemctl enable autofs
   sudo service autofs restart
   </code></pre>

1. **[A]** Настройте файл подкачки.

   <pre><code>
   sudo vi /etc/waagent.conf
   
   # Set the property ResourceDisk.EnableSwap to y
   # Create and use swapfile on resource disk.
   ResourceDisk.EnableSwap=<b>y</b>
   
   # Set the size of the SWAP file with property ResourceDisk.SwapSizeMB
   # The free space of resource disk varies by virtual machine size. Make sure that you do not set a value that is too big. You can check the SWAP space with command swapon
   # Size of the swapfile.
   ResourceDisk.SwapSizeMB=<b>2000</b>
   </code></pre>

   Перезапустите агент, чтобы активировать изменения:

   <pre><code>sudo service waagent restart
   </code></pre>

## <a name="install-database"></a>Установка базы данных

В данном примере SAP NetWeaver уже установлен на SAP HANA. Для этой установки можно использовать любую поддерживаемую базу данных. Дополнительные сведения об установке SAP HANA в Azure см. в статье [Высокий уровень доступности SAP HANA на виртуальных машинах Azure][sap-hana-ha]. Список поддерживаемых баз данных см. в примечании к SAP [1928533][1928533].

* Запустите установку экземпляра базы данных SAP.

   Установите экземпляр базы данных SAP NetWeaver в качестве корневого, используя имя виртуального узла, которое сопоставляется с внешним IP-адресом подсистемы балансировки нагрузки для базы данных

   Чтобы разрешить непривилегированному пользователю подключаться к SAPinst, можно использовать параметр SAPINST_REMOTE_ACCESS_USER.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b>
   </code></pre>

## <a name="sap-netweaver-application-server-installation"></a>Установка сервера приложений SAP NetWeaver

Для установки сервера приложений SAP выполните следующие действия.

1. **[A]** Подготовьте сервер приложений. Следуйте инструкциям из раздела [Подготовка сервера приложений SAP NetWeaver](high-availability-guide-suse-netapp-files.md#2d6008b0-685d-426c-b59e-6cd281fd45d7) выше.

2. **[A]** Установите сервер приложений SAP NetWeaver. Установите основной или дополнительный сервер приложений SAP NetWeaver.

   Чтобы разрешить непривилегированному пользователю подключаться к SAPinst, можно использовать параметр SAPINST_REMOTE_ACCESS_USER.

   <pre><code>sudo &lt;swpm&gt;/sapinst SAPINST_REMOTE_ACCESS_USER=<b>sapadmin</b>
   </code></pre>

3. **[A]** Обновите безопасное хранилище SAP HANA.

   Обновите безопасное хранилище SAP HANA, чтобы оно указывало на виртуальное имя настройки репликации системы SAP HANA.

   Чтобы получить список записей, выполните следующую команду:
   <pre><code>
   hdbuserstore List
   </code></pre>

   Это позволит вывести список всех записей. Он будет выглядеть так:
   <pre><code>
   DATA FILE       : /home/qasadm/.hdb/anftstsapa01/SSFS_HDB.DAT
   KEY FILE        : /home/qasadm/.hdb/anftstsapa01/SSFS_HDB.KEY
   
   KEY DEFAULT
     ENV : 10.1.1.5:<b>30313</b>
     USER: <b>SAPABAP1</b>
     DATABASE: <b>QAS</b>
   </code></pre>

   Выходные данные будут содержать IP-адрес записи по умолчанию, которая указывает на виртуальную машину, а не на IP-адрес балансировщика нагрузки. Эту запись необходимо изменить так, чтобы она указывала на имя виртуального узла балансировщика нагрузки. Убедитесь, что вы используете тот же порт (**30313** в выходных данных выше) и имя базы данных (**QAS**)!

   <pre><code>
   su - <b>qas</b>adm
   hdbuserstore SET DEFAULT <b>qasdb:30313@QAS</b> <b>SAPABAP1</b> <b>&lt;password of ABAP schema&gt;</b>
   </code></pre>

## <a name="test-the-cluster-setup"></a>Проверка настройки кластера

Следующие тесты являются копией тестовых случаев из [рекомендаций по SUSE][suse-ha-guide]. Они скопированы для вашего удобства. Читайте рекомендации и выполняйте все дополнительные тесты, которые были добавлены.

1. Протестируйте HAGetFailoverConfig, HACheckConfig и HACheckFailoverConfig.

   Выполните следующие команды как \<sapsid>adm на узле, где в настоящий момент выполняется экземпляр ASCS. Если выполнение команд завершается сбоем "Сбой: Недостаточно памяти", возможно, это связано с тире в имени вашего узла. Это известная проблема, и SUSE устранит ее в пакете sap-suse-cluster-connector.

   <pre><code>
   anftstsapcl1:qasadm 52> sapcontrol -nr 00 -function HAGetFailoverConfig
   07.03.2019 20:08:59
   HAGetFailoverConfig
   OK
   HAActive: TRUE
   HAProductVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP3
   HASAPInterfaceVersion: SUSE Linux Enterprise Server for SAP Applications 12 SP3 (sap_suse_cluster_connector 3.1.0)
   HADocumentation: https://www.suse.com/products/sles-for-sap/resource-library/sap-best-practices/
   HAActiveNode: anftstsapcl1
   HANodes: anftstsapcl1, anftstsapcl2

   anftstsapcl1:qasadm 54> sapcontrol -nr 00 -function HACheckConfig
   07.03.2019 23:28:29
   HACheckConfig
   OK
   state, category, description, comment
   SUCCESS, SAP CONFIGURATION, Redundant ABAP instance configuration, 2 ABAP instances detected
   SUCCESS, SAP CONFIGURATION, Redundant Java instance configuration, 0 Java instances detected
   SUCCESS, SAP CONFIGURATION, Enqueue separation, All Enqueue server separated from application server
   SUCCESS, SAP CONFIGURATION, MessageServer separation, All MessageServer separated from application server
   SUCCESS, SAP CONFIGURATION, ABAP instances on multiple hosts, ABAP instances on multiple hosts detected
   SUCCESS, SAP CONFIGURATION, Redundant ABAP SPOOL service configuration, 2 ABAP instances with SPOOL service detected
   SUCCESS, SAP STATE, Redundant ABAP SPOOL service state, 2 ABAP instances with active SPOOL service detected
   SUCCESS, SAP STATE, ABAP instances with ABAP SPOOL service on multiple hosts, ABAP instances with active ABAP SPOOL service on multiple hosts detected
   SUCCESS, SAP CONFIGURATION, Redundant ABAP BATCH service configuration, 2 ABAP instances with BATCH service detected
   SUCCESS, SAP STATE, Redundant ABAP BATCH service state, 2 ABAP instances with active BATCH service detected
   SUCCESS, SAP STATE, ABAP instances with ABAP BATCH service on multiple hosts, ABAP instances with active ABAP BATCH service on multiple hosts detected
   SUCCESS, SAP CONFIGURATION, Redundant ABAP DIALOG service configuration, 2 ABAP instances with DIALOG service detected
   SUCCESS, SAP STATE, Redundant ABAP DIALOG service state, 2 ABAP instances with active DIALOG service detected
   SUCCESS, SAP STATE, ABAP instances with ABAP DIALOG service on multiple hosts, ABAP instances with active ABAP DIALOG service on multiple hosts detected
   SUCCESS, SAP CONFIGURATION, Redundant ABAP UPDATE service configuration, 2 ABAP instances with UPDATE service detected
   SUCCESS, SAP STATE, Redundant ABAP UPDATE service state, 2 ABAP instances with active UPDATE service detected
   SUCCESS, SAP STATE, ABAP instances with ABAP UPDATE service on multiple hosts, ABAP instances with active ABAP UPDATE service on multiple hosts detected
   SUCCESS, SAP STATE, SCS instance running, SCS instance status ok
   SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version (anftstsapvh_QAS_00), SAPInstance includes is-ers patch
   SUCCESS, SAP CONFIGURATION, Enqueue replication (anftstsapvh_QAS_00), Enqueue replication enabled
   SUCCESS, SAP STATE, Enqueue replication state (anftstsapvh_QAS_00), Enqueue replication active
   
   anftstsapcl1:qasadm 55> sapcontrol -nr 00 -function HACheckFailoverConfig
   07.03.2019 23:30:48
   HACheckFailoverConfig
   OK
   state, category, description, comment
   SUCCESS, SAP CONFIGURATION, SAPInstance RA sufficient version, SAPInstance includes is-ers patch
   </code></pre>

2. Миграция экземпляра ASCS вручную

   Состояние ресурсов перед запуском теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rscsap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Starting anftstsapcl1
   </code></pre>

   Выполните следующие команды от имени привилегированного пользователя, чтобы перенести экземпляр ASCS.

   <pre><code>
   anftstsapcl1:~ # crm resource migrate rsc_sap_QAS_ASCS00 force
   INFO: Move constraint created for rsc_sap_QAS_ASCS00
   
   anftstsapcl1:~ # crm resource unmigrate rsc_sap_QAS_ASCS00
   INFO: Removed migration constraints for rsc_sap_QAS_ASCS00
   
   # Remove failed actions for the ERS that occurred as part of the migration
   anftstsapcl1:~ # crm resource cleanup rsc_sap_QAS_ERS01
   </code></pre>

   Состояние ресурсов после теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   </code></pre>

3. Проверка HAFailoverToNode

   Состояние ресурсов перед запуском теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   </code></pre>

   Выполните следующие команды как \<sapsid>adm, чтобы перенести экземпляр ASCS.

   <pre><code>
   anftstsapcl1:qasadm 53> sapcontrol -nr 00 -host anftstsapvh -user <b>qas</b>adm &lt;password&gt; -function HAFailoverToNode ""
   
   # run as root
   # Remove failed actions for the ERS that occurred as part of the migration
   anftstsapcl1:~ # crm resource cleanup rsc_sap_QAS_ERS01
   # Remove migration constraints
   anftstsapcl1:~ # crm resource clear rsc_sap_QAS_ASCS00
   #INFO: Removed migration constraints for rsc_sap_QAS_ASCS00
   </code></pre>

   Состояние ресурсов после теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

4. Имитирование сбоя узла 

   Состояние ресурсов перед запуском теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

   Выполните следующую команду в качестве привилегированного пользователя на узле, где выполняется экземпляр ASCS.

   <pre><code>anftstsapcl2:~ # echo b > /proc/sysrq-trigger
   </code></pre>

   Если вы используете SBD, то Pacemaker не должен автоматически запускаться на завершившем работу узле. Состояние после повторного запуска узла должно выглядеть так:

   <pre><code>Online:
   Online: [ anftstsapcl1 ]
   OFFLINE: [ anftstsapcl2 ]

   Full list of resources:

    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1

   Failed Actions:
   * rsc_sap_QAS_ERS01_monitor_11000 on anftstsapcl1 'not running' (7): call=166, status=complete, exitreason='',
    last-rc-change='Fri Mar  8 18:26:10 2019', queued=0ms, exec=0ms
   </code></pre>

   Выполните следующие команды, чтобы запустить Pacemaker на отключенном узле, очистить сообщения SBD и очистить ресурсы, в которых произошел сбой.

   <pre><code>
   # run as root
   # list the SBD device(s)
   anftstsapcl2:~ # cat /etc/sysconfig/sbd | grep SBD_DEVICE=
   # SBD_DEVICE="/dev/disk/by-id/scsi-36001405b730e31e7d5a4516a2a697dcf;/dev/disk/by-id/scsi-36001405f69d7ed91ef54461a442c676e;/dev/disk/by-id/scsi-360014058e5f335f2567488882f3a2c3a"

   anftstsapcl2:~ # sbd -d /dev/disk/by-id/scsi-36001405772fe8401e6240c985857e11 -d /dev/disk/by-id/scsi-36001405f69d7ed91ef54461a442c676e -d /dev/disk/by-id/scsi-360014058e5f335f2567488882f3a2c3a message anftstsapcl2 clear

   anftstsapcl2:~ # systemctl start pacemaker
   anftstsapcl2:~ # crm resource cleanup rsc_sap_QAS_ASCS00
   anftstsapcl2:~ # crm resource cleanup rsc_sap_QAS_ERS01
   </code></pre>

   Состояние ресурсов после теста:

   <pre><code>
   Full list of resources:
   
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   </code></pre>

5. Проверка перезапуска экземпляра ASCS вручную

   Состояние ресурсов перед запуском теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

   Создайте блокировку постановки в очередь, например измените пользователя в транзакции su01. Выполните следующие команды как <sapsid\>adm на узле, где выполняется экземпляр ASCS. Команды остановят экземпляр ASCS и запустят его снова. Если используется архитектура сервера постановки в очередь 1, блокировка очереди должна быть потеряна в этом тесте. Если используется архитектура сервера постановки в очередь 2, блокировка сохранится. 

   <pre><code>anftstsapcl2:qasadm 51> sapcontrol -nr 00 -function StopWait 600 2
   </code></pre>

   Теперь экземпляр ASCS должен быть отключен в Pacemaker.

   <pre><code>  rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Stopped (disabled)
   </code></pre>

   Снова запустите экземпляр ASCS на этом же узле.

   <pre><code>anftstsapcl2:qasadm 52> sapcontrol -nr 00 -function StartWait 600 2
   </code></pre>

   Если используется архитектура репликации сервера постановки в очередь 1, блокировка постановки в очередь для транзакции su01 должна быть потеряна, а серверная часть — сброшена. Состояние ресурсов после теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

6. Завершение процесса сервера сообщений

   Состояние ресурсов перед запуском теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

   Выполните следующие команды от имени привилегированного пользователя, чтобы определить процесс сервера сообщений и завершить его.

   <pre><code>anftstsapcl2:~ # pgrep ms.sapQAS | xargs kill -9
   </code></pre>

   Если вы завершите работу сервера сообщений только раз, его перезапустит `sapstart`. Если вы завершите работу сервера достаточно много раз, Pacemaker в конечном итоге переместит экземпляр ASCS на другой узел. Выполните следующие команды от имени привилегированного пользователя для очистки состояния ресурсов экземпляра ERS и ASCS после теста.

   <pre><code>
   anftstsapcl2:~ # crm resource cleanup rsc_sap_QAS_ASCS00
   anftstsapcl2:~ # crm resource cleanup rsc_sap_QAS_ERS01
   </code></pre>

   Состояние ресурсов после теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   </code></pre>

7. Завершение процесса сервера постановки в очередь

   Состояние ресурсов перед запуском теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   </code></pre>

   Выполните следующие команды от имени привилегированного пользователя на узле, где выполняется экземпляр ASCS, чтобы завершить работу сервера постановки в очередь.

   <pre><code>anftstsapcl1:~ # pgrep en.sapQAS | xargs kill -9
   </code></pre>

   Для экземпляра ASCS должна быть немедленно выполнена отработка отказа с переносом на другой узел. После запуска экземпляра ASCS экземпляр ERS должен также выполнить отработку отказа. Выполните следующие команды от имени привилегированного пользователя для очистки состояния ресурсов экземпляра ERS и ASCS после теста.

   <pre><code>
   anftstsapcl1:~ # crm resource cleanup rsc_sap_QAS_ASCS00
   anftstsapcl1:~ # crm resource cleanup rsc_sap_QAS_ERS01
   </code></pre>

   Состояние ресурсов после теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

8. Завершение процесса сервера постановки в очередь для репликации

   Состояние ресурсов перед запуском теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

   Выполните следующую команду от имени привилегированного пользователя на узле, где выполняется экземпляр ERS, чтобы завершить работу процесса сервера постановки в очередь для репликации.

   <pre><code>anftstsapcl1:~ # pgrep er.sapQAS | xargs kill -9
   </code></pre>

   Если вы выполните команду только раз, `sapstart` перезапустит процесс. Если вы выполните ее достаточно часто, `sapstart` не перезапустит процесс и ресурс будет находиться в остановленном состоянии. Выполните следующие команды от имени привилегированного пользователя для очистки состояния ресурсов экземпляра ERS после теста.

   <pre><code>anftstsapcl1:~ # crm resource cleanup rsc_sap_QAS_ERS01
   </code></pre>

   Состояние ресурсов после теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

9. Завершение процесса постановки sapstartsrv в очередь

   Состояние ресурсов перед запуском теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

   Выполните следующие команды в качестве привилегированного пользователя на узле, где выполняется ASCS.

   <pre><code>
   anftstsapcl2:~ # pgrep -fl ASCS00.*sapstartsrv
   #67625 sapstartsrv
   
   anftstsapcl2:~ # kill -9 67625
   </code></pre>

   Процесс sapstartsrv всегда должен перезапускаться с помощью агента ресурсов Pacemaker. Состояние ресурсов после теста:

   <pre><code>
    Resource Group: g-QAS_ASCS
        fs_QAS_ASCS        (ocf::heartbeat:Filesystem):    Started anftstsapcl2
        nc_QAS_ASCS        (ocf::heartbeat:azure-lb):      Started anftstsapcl2
        vip_QAS_ASCS       (ocf::heartbeat:IPaddr2):       Started anftstsapcl2
        rsc_sap_QAS_ASCS00 (ocf::heartbeat:SAPInstance):   Started anftstsapcl2
   stonith-sbd     (stonith:external/sbd): Started anftstsapcl1
    Resource Group: g-QAS_ERS
        fs_QAS_ERS (ocf::heartbeat:Filesystem):    Started anftstsapcl1
        nc_QAS_ERS (ocf::heartbeat:azure-lb):      Started anftstsapcl1
        vip_QAS_ERS        (ocf::heartbeat:IPaddr2):       Started anftstsapcl1
        rsc_sap_QAS_ERS01  (ocf::heartbeat:SAPInstance):   Started anftstsapcl1
   </code></pre>

## <a name="next-steps"></a>Дальнейшие действия

* [Обеспечение высокого уровня доступности SAP NW на виртуальных машинах Azure в SLES с несколькими ИД безопасности для приложений SAP](./high-availability-guide-suse-multi-sid.md)
* [SAP NetWeaver на виртуальных машинах Windows. Руководство по планированию и внедрению][planning-guide]
* [Развертывание виртуальных машин Azure для SAP NetWeaver][deployment-guide]
* [SAP NetWeaver на виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server][dbms-guide]
* Дополнительные сведения об установке высокого уровня доступности и плана для аварийного восстановления SAP HANA на виртуальных машинах Azure см. в статье [Высокий уровень доступности SAP HANA на виртуальных машинах Azure][sap-hana-ha].