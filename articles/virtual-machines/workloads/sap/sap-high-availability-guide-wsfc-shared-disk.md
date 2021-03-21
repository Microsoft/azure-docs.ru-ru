---
title: Кластеризация экземпляра SAP ASCS/SCS в WSFC с помощью общего диска в Azure | Документация Майкрософт
description: Сведения о кластеризации экземпляра SAP ASCS/SCS в отказоустойчивом кластере Windows с помощью общего диска кластера.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: f6fb85f8-c77a-4af1-bde8-1de7e4425d2e
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 10/16/2020
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c903cf06981e1336ae30942775de11d09bb1299b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101675359"
---
# <a name="cluster-an-sap-ascsscs-instance-on-a-windows-failover-cluster-by-using-a-cluster-shared-disk-in-azure"></a>Кластеризация экземпляра SAP ASCS/SCS в отказоустойчивом кластере Windows с помощью общего диска кластера в Azure

> ![ОС Windows][Logo_Windows] Windows
>

Отказоустойчивая кластеризация Windows Server является основой для установки высокодоступных приложений SAP ASCS/SCS и СУБД в Windows.

Отказоустойчивый кластер представляет собой группу из 1 + n независимых серверов (узлов), работающих совместно для повышения доступности приложений и служб. В случае отказа узла отказоустойчивый кластер Windows Server вычисляет допустимое количество сбоев, при котором сохранится работоспособность кластера для предоставления приложений и служб. Возможность отказоустойчивой кластеризации можно добавить, используя разные режимы кворума.

## <a name="prerequisites"></a>Предварительные условия
Перед началом работы с этой статьей, ознакомьтесь со следующим документом:

* [Архитектура высокого уровня доступности и сценарии для SAP NetWeaver на виртуальных машинах Azure][sap-high-availability-architecture-scenarios]


## <a name="windows-server-failover-clustering-in-azure"></a>Отказоустойчивый кластер Windows Server в Azure

Для отказоустойчивой кластеризации Windows Server с виртуальными машинами Azure требуются дополнительные действия по настройке. При создании кластера вы должны назначить несколько IP-адресов и имен виртуальных узлов экземпляру SAP ASCS/SCS.

### <a name="name-resolution-in-azure-and-the-cluster-virtual-host-name"></a>Разрешение имен в Azure и имя виртуального узла кластера

Облачная платформа Azure не предоставляет возможность настройки виртуальных IP-адресов, т. е. плавающих IP-адресов. Для настройки виртуального IP-адреса, используемого для доступа к кластерным ресурсам в облаке, вам потребуется альтернативное решение. 

Azure Load Balancer выполняет роль *внутренней подсистемы балансировки нагрузки* для Azure. С его помощью клиенты могут обращаться к кластеру через виртуальный IP-адрес кластера. 

Разверните внутреннюю подсистему балансировки нагрузки в группе ресурсов, которая содержит узлы кластера. Затем настройте все необходимые правила перенаправления портов, используя порты проб внутренней подсистемы балансировки нагрузки. Для подключения клиенты могут использовать имя виртуального узла. DNS-сервер разрешает IP-адрес кластера, и внутренний балансировщик нагрузки выполняет перенаправление на активный узел кластера.

> [!IMPORTANT]
> Плавающий IP-адрес не поддерживается для вторичной IP-конфигурации NIC в сценариях балансировки нагрузки. Дополнительные сведения см. в статье [ограничения балансировщика нагрузки Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Если для виртуальной машины требуется дополнительный IP-адрес, разверните вторую сетевую карту.  

![Рис. 1. Конфигурация отказоустойчивого кластера Windows без общего диска в Azure][sap-ha-guide-figure-1001]

_Конфигурация отказоустойчивого кластера Windows Server в Azure без общего диска_

### <a name="sap-ascsscs-ha-with-cluster-shared-disks"></a>Высокодоступная система SAP ASCS/SCS с общими дисками кластера
Экземпляр SAP ASCS/SCS в Windows содержит центральные службы SAP, сервер сообщений SAP, серверные процессы постановки в очередь и файлы глобального узла SAP. Файлы глобального узла SAP содержат центральные файлы для всей системы SAP.

Экземпляр SAP ASCS/SCS состоит из следующих компонентов.

* Центральные службы SAP.
    * Два процесса: сообщение, сервер очереди и объект \<ASCS/SCS virtual host name> , который используется для доступа к этим двум процессам.
    * Структура файла: S:\usr\sap \\ &lt; SID &gt; \ ASCS/SCS\<instance number\>


* Файлы глобального узла SAP.
  * Структура файлов: S:\usr\sap\\&lt;SID&gt;\SYS\..
  * Файловый ресурс sapmnt, который предоставляет доступ к глобальным файлам S:\usr\sap\\&lt;SID&gt;\SYS\... по следующему UNC-пути:

    \\\\<имя виртуального узла ASCS/SCS \> \Sapmnt \\ &lt; SID &gt; \SYS \. ..


![Рисунок 2. Процессы, структура файлов и файловый ресурс sapmnt глобального узла экземпляра SAP ASCS/SCS][sap-ha-guide-figure-8001]

_Процессы, структура файлов и общая папка sapmnt общего узла для экземпляра SAP ASCS/SCS_

В системе высокого уровня доступности применяются экземпляры SAP ASCS/SCS кластера. Мы используем *кластерные общие диски* (в нашем примере это диск S:\) для размещения файлов SAP ASCS/SCS и файлов глобального узла SAP.

![Рисунок 3. Архитектура высокодоступной системы SAP ASCS/SCS с общим диском][sap-ha-guide-figure-8002]

_Архитектура высокой доступности SAP ASCS/SCS с общим диском_


Архитектура репликации сервера постановки в очередь 1:
* То же самое можно \<ASCS/SCS virtual host name> использовать для доступа к сообщениям SAP и серверным процессам постановки в очередь, а также к файлам глобального узла SAP через общую папку sapmnt.
* Общий диск кластера с именем S: используется ими совместно.  

Архитектура репликации сервера постановки в очередь 2: 
* То же самое \<ASCS/SCS virtual host name> используется для доступа к процессу сервера сообщений SAP и файлов глобального узла SAP через общую папку sapmnt.
* Общий диск кластера с именем S: используется ими совместно.
* Существует отдельный \<ERS virtual host name> доступ к процессу сервера постановки в очередь  

![Рисунок 4. Архитектура высокодоступной системы SAP ASCS/SCS с общим диском][sap-ha-guide-figure-8003]

_Архитектура высокой доступности SAP ASCS/SCS с общим диском_

#### <a name="shared-disk-and-enqueue-replication-server"></a>Общий диск и постановка в очередь сервера репликации 

1. Общий диск поддерживается в архитектуре репликации сервера очереди 1, где экземпляр сервера репликации постановки в очередь (ERS):   

   - не кластеризован
   - использует `localhost` имя
   - развернут на локальных дисках на каждом узле кластера;

2. Кроме того, общий диск поддерживается архитектурой репликации сервера постановки в очередь, где экземпляр сервера репликации постановки в очередь 2 (ERS2):  

   - кластеризованный
   - использует выделенное виртуальное или сетевое имя узла
   - в дополнение к IP-адресу (A) SCS должен быть настроен IP-адрес виртуального узла ERS для внутренних Load Balancer Azure.
   - развертывается на **локальных дисках** каждого из кластерных узлов, поэтому нет необходимости в общем диске.

   > [!TIP]
   > Дополнительные сведения о постановке в очередь серверов репликации 1 и 2 (ERS1 и ERS2) можно найти здесь:  
   > [Постановка сервера репликации в очередь в отказоустойчивом кластере Майкрософт](https://help.sap.com/viewer/3741bfe0345f4892ae190ee7cfc53d4c/CURRENT_VERSION_SWPM20/en-US/8abd4b52902d4b17a105c2fabdf5c0cf.html)  
   > [Новый репликатор очереди в средах отказоустойчивого кластера](https://blogs.sap.com/2019/03/19/new-enqueue-replicator-in-failover-cluster-environments/)  

#### <a name="options-for-shared-disk-in-azure-for-sap-workloads"></a>Параметры для общего диска в рабочих нагрузках SAP в Azure

Существует два варианта использования общего диска в отказоустойчивом кластере Windows в Azure.

- [Общие диски Azure](../../disks-shared.md) — компонент, позволяющий одновременно подключать управляемый диск Azure к нескольким виртуальным машинам. 
- Использование стороннего программного обеспечения [SIOS Cluster Edition](https://us.sios.com/products/datakeeper-cluster) для создания зеркально отображаемого хранилища, моделирующего общее хранилище кластера. 

При выборе технологии для общего диска учитывайте следующие моменты.

**Общий диск Azure для рабочих нагрузок SAP**
- Позволяет подключать управляемый диск Azure к нескольким виртуальным машинам одновременно, не требуя дополнительного программного обеспечения для обслуживания и эксплуатации 
- Вы будете работать с одним общим диском Azure в одном кластере хранилища. Это влияет на надежность решения SAP.
- Сейчас поддерживается только развертывание с общим диском Azure уровня "Премиум" в группе доступности. Общий диск Azure не поддерживается в развертывании зональные.     
- Обязательно подготавливает диск Azure уровня "Премиум" с минимальным размером диска, указанным в [SSD (цен. Категория "Премиум") диапазоны](../../disks-shared.md#disk-sizes) , чтобы иметь возможность одновременного подключения к требуемому количеству виртуальных машин (обычно 2 для отказоустойчивого кластера Windows SAP ASCS). 
- Общий Ultra диск Azure не поддерживается для рабочих нагрузок SAP, так как он не поддерживает развертывание в группе доступности или развертывании зональные.  
 
**SIOS**
- Решение SIOS обеспечивает синхронную репликацию данных в режиме реального времени между двумя дисками.
- Решение SIOS, работающее с двумя управляемыми дисками. при использовании групп доступности или зон доступности управляемые диски будут находиться на разных кластерах хранения. 
- Развертывание в зонах доступности поддерживается
- Требуется установка и эксплуатация стороннего программного обеспечения, которое потребуется приобрести дополнительно

### <a name="shared-disk-using-azure-shared-disk"></a>Общий диск с использованием общего диска Azure

Корпорация Майкрософт предлагает [Общие диски Azure](../../disks-shared.md), которые можно использовать для реализации высокой доступности SAP ASCS/SCS с помощью параметра общий диск.

#### <a name="prerequisites-and-limitations"></a>Предварительные условия и ограничения

В настоящее время вы можете использовать диски SSD (цен. категория "Премиум") Azure в качестве общего диска Azure для экземпляра SAP ASCS/SCS. В настоящее время используются следующие ограничения:

-  [Azure Ultra Disk](../../disks-types.md#ultra-disk) не поддерживается как общий диск Azure для рабочих нагрузок SAP. Сейчас невозможно разместить виртуальные машины Azure с помощью Azure Ultra Disk в группе доступности.
-  [Общий диск Azure](../../disks-shared.md) с SSD (цен. Категория "Премиум")ными дисками поддерживается только для виртуальных машин в группе доступности. Он не поддерживается в Зоны доступности развертывании. 
-  Значение общего диска Azure [максшарес](../../disks-shared-enable.md?tabs=azure-cli#disk-sizes) определяет, сколько узлов кластера может использовать общий диск. Как правило, для экземпляра SAP ASCS/SCS настраиваются два узла в отказоустойчивом кластере Windows, поэтому для параметра `maxShares` необходимо задать значение 2.
-  Все виртуальные машины кластера SAP ASCS/SCS должны быть развернуты в одной [группе размещения](../../windows/proximity-placement-groups.md)службы расположения Azure.   
   Несмотря на то, что вы можете развернуть виртуальные машины кластера Windows в группе доступности с общим диском Azure без ППГ, ППГ обеспечит близкое физическое сходство общих дисков Azure и виртуальных машин кластера, тем самым снижая задержки между виртуальными машинами и уровнем хранилища.    

Дополнительные сведения об ограничениях для общего диска Azure см. в разделе " [ограничения](../../disks-shared.md#limitations) " в документации по общим дискам Azure.

> [!IMPORTANT]
> При развертывании отказоустойчивого кластера Windows SAP ASCS/SCS с помощью общего диска Azure имейте в виду, что развертывание будет работать с одним общим диском в одном кластере хранилища. На случай возникновения проблем с кластером хранения, на котором развернут общий диск Azure, будет затронуто экземпляр SAP ASCS/SCS.    

> [!TIP]
> При планировании развертывания SAP ознакомьтесь с [руководством по планированию SAP NetWeaver в Azure](./planning-guide.md) и [руководством по службе хранилища Azure для рабочих нагрузок SAP](./planning-guide-storage.md) .

### <a name="supported-os-versions"></a>Поддерживаемые версии ОС

Поддерживаются как Windows Server 2016, так и 2019 (Используйте последние образы центра обработки данных).

Настоятельно рекомендуется использовать **Windows Server 2019 Datacenter**, как показано ниже.
- Служба отказоустойчивого кластера Windows 2019 поддерживает Azure
- Добавлена поддержка интеграции и поддержки обслуживания узла Azure и улучшена работа по планированию событий Azure.
- Можно использовать имя распределенной сети (это параметр по умолчанию). Поэтому нет необходимости иметь выделенный IP-адрес для сетевого имени кластера. Кроме того, не нужно настраивать этот IP-адрес во внутренних Load Balancerах Azure. 

### <a name="shared-disks-in-azure-with-sios-datakeeper"></a>Общие диски в Azure с использованием SIOS DataKeeper

Другим вариантом для общего диска является использование стороннего программного обеспечения SIOS Cluster Edition для создания зеркально отображаемого хранилища, моделирующего общее хранилище кластера. Решение SIOS обеспечивает синхронную репликацию данных в реальном времени.

Чтобы создать ресурс общего диска для кластера, выполните следующие действия.

1. Подключите дополнительный диск к каждой виртуальной машине, входящей в кластер Windows.
2. Запустите SIOS DataKeeper Cluster Edition на обоих узлах виртуальной машины.
3. Настройте ПО SIOS DataKeeper Cluster Edition таким образом, чтобы оно зеркально отображало содержимое тома дополнительного подключенного диска исходной виртуальной машины в том дополнительного подключенного диска целевой виртуальной машины. SIOS DataKeeper абстрагирует исходные и конечные локальные тома и предоставляет их как один общий диск отказоустойчивому кластеру Windows.

Дополнительные сведения о SIOS DataKeeper см. [здесь](https://us.sios.com/products/datakeeper-cluster/).

![Рисунок 5. Конфигурация отказоустойчивой кластеризации Windows Server в Azure с использованием SIOS DataKeeper][sap-ha-guide-figure-1002]

_Конфигурация отказоустойчивого кластера Windows в Azure с помощью SIOS_

> [!NOTE]
> В некоторых СУБД, например SQL Server, общие диски не требуются для достижения высокого уровня доступности. SQL Server AlwaysOn выполняет репликацию файлов данных и журналов СУБД с локального диска одного узла кластера на локальный диск другого узла кластера. Поэтому в конфигурации кластера Windows не нужен общий диск.
>

## <a name="next-steps"></a>Дальнейшие действия

* [Узнайте, как подготовить инфраструктуру Azure для SAP высокого уровня доступности с помощью отказоустойчивого кластера Windows и общего диска для экземпляров SAP ASCS/SCS][sap-high-availability-infrastructure-wsfc-shared-disk]

* [Установите SAP NetWeaver высокого уровня доступности в отказоустойчивом кластере Windows на общем диске для экземпляра SAP ASCS/SCS][sap-high-availability-installation-wsfc-shared-disk]


[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides

[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md

[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md
[ha-guide]:sap-high-availability-guide.md
[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

[sap-ha-guide]:sap-high-availability-guide.md
[sap-ha-guide-2]:#42b8f600-7ba3-4606-b8a5-53c4f026da08
[sap-ha-guide-4]:#8ecf3ba0-67c0-4495-9c14-feec1a2255b7
[sap-ha-guide-8]:#78092dbe-165b-454c-92f5-4972bdbef9bf
[sap-ha-guide-8.1]:#c87a8d3f-b1dc-4d2f-b23c-da4b72977489
[sap-ha-guide-8.9]:#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-ha-guide-8.11]:#661035b2-4d0f-4d31-86f8-dc0a50d78158
[sap-ha-guide-8.12]:#0d67f090-7928-43e0-8772-5ccbf8f59aab
[sap-ha-guide-8.12.1]:#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79
[sap-ha-guide-8.12.3]:#5c8e5482-841e-45e1-a89d-a05c0907c868
[sap-ha-guide-8.12.3.1]:#1c2788c3-3648-4e82-9e0d-e058e475e2a3
[sap-ha-guide-8.12.3.2]:#dd41d5a2-8083-415b-9878-839652812102
[sap-ha-guide-8.12.3.3]:#d9c1fc8e-8710-4dff-bec2-1f535db7b006
[sap-ha-guide-9]:#a06f0b49-8a7a-42bf-8b0d-c12026c5746b
[sap-ha-guide-9.1]:#31c6bd4f-51df-4057-9fdf-3fcbc619c170
[sap-ha-guide-9.1.1]:#a97ad604-9094-44fe-a364-f89cb39bf097

[sap-ha-multi-sid-guide]:sap-high-availability-multi-sid.md (Конфигурации высокой доступности SAP с несколькими SID)

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png

[sap-ha-guide-figure-1000]:./media/virtual-machines-shared-sap-high-availability-guide/1000-wsfc-for-sap-ascs-on-azure.png
[sap-ha-guide-figure-1001]:./media/virtual-machines-shared-sap-high-availability-guide/1001-wsfc-on-azure-ilb.png
[sap-ha-guide-figure-1002]:./media/virtual-machines-shared-sap-high-availability-guide/1002-wsfc-sios-on-azure-ilb.png
[sap-ha-guide-figure-2000]:./media/virtual-machines-shared-sap-high-availability-guide/2000-wsfc-sap-as-ha-on-azure.png
[sap-ha-guide-figure-2001]:./media/virtual-machines-shared-sap-high-availability-guide/2001-wsfc-sap-ascs-ha-on-azure.png
[sap-ha-guide-figure-2003]:./media/virtual-machines-shared-sap-high-availability-guide/2003-wsfc-sap-dbms-ha-on-azure.png
[sap-ha-guide-figure-2004]:./media/virtual-machines-shared-sap-high-availability-guide/2004-wsfc-sap-ha-e2e-archit-template1-on-azure.png
[sap-ha-guide-figure-2005]:./media/virtual-machines-shared-sap-high-availability-guide/2005-wsfc-sap-ha-e2e-arch-template2-on-azure.png

[sap-ha-guide-figure-3000]:./media/virtual-machines-shared-sap-high-availability-guide/3000-template-parameters-sap-ha-arm-on-azure.png
[sap-ha-guide-figure-3001]:./media/virtual-machines-shared-sap-high-availability-guide/3001-configuring-dns-servers-for-Azure-vnet.png
[sap-ha-guide-figure-3002]:./media/virtual-machines-shared-sap-high-availability-guide/3002-configuring-static-IP-address-for-network-card-of-each-vm.png
[sap-ha-guide-figure-3003]:./media/virtual-machines-shared-sap-high-availability-guide/3003-setup-static-ip-address-ilb-for-ascs-instance.png
[sap-ha-guide-figure-3004]:./media/virtual-machines-shared-sap-high-availability-guide/3004-default-ascs-scs-ilb-balancing-rules-for-azure-ilb.png
[sap-ha-guide-figure-3005]:./media/virtual-machines-shared-sap-high-availability-guide/3005-changing-ascs-scs-default-ilb-rules-for-azure-ilb.png
[sap-ha-guide-figure-3006]:./media/virtual-machines-shared-sap-high-availability-guide/3006-adding-vm-to-domain.png
[sap-ha-guide-figure-3007]:./media/virtual-machines-shared-sap-high-availability-guide/3007-config-wsfc-1.png
[sap-ha-guide-figure-3008]:./media/virtual-machines-shared-sap-high-availability-guide/3008-config-wsfc-2.png
[sap-ha-guide-figure-3009]:./media/virtual-machines-shared-sap-high-availability-guide/3009-config-wsfc-3.png
[sap-ha-guide-figure-3010]:./media/virtual-machines-shared-sap-high-availability-guide/3010-config-wsfc-4.png
[sap-ha-guide-figure-3011]:./media/virtual-machines-shared-sap-high-availability-guide/3011-config-wsfc-5.png
[sap-ha-guide-figure-3012]:./media/virtual-machines-shared-sap-high-availability-guide/3012-config-wsfc-6.png
[sap-ha-guide-figure-3013]:./media/virtual-machines-shared-sap-high-availability-guide/3013-config-wsfc-7.png
[sap-ha-guide-figure-3014]:./media/virtual-machines-shared-sap-high-availability-guide/3014-config-wsfc-8.png
[sap-ha-guide-figure-3015]:./media/virtual-machines-shared-sap-high-availability-guide/3015-config-wsfc-9.png
[sap-ha-guide-figure-3016]:./media/virtual-machines-shared-sap-high-availability-guide/3016-config-wsfc-10.png
[sap-ha-guide-figure-3017]:./media/virtual-machines-shared-sap-high-availability-guide/3017-config-wsfc-11.png
[sap-ha-guide-figure-3018]:./media/virtual-machines-shared-sap-high-availability-guide/3018-config-wsfc-12.png
[sap-ha-guide-figure-3019]:./media/virtual-machines-shared-sap-high-availability-guide/3019-assign-permissions-on-share-for-cluster-name-object.png
[sap-ha-guide-figure-3020]:./media/virtual-machines-shared-sap-high-availability-guide/3020-change-object-type-include-computer-objects.png
[sap-ha-guide-figure-3021]:./media/virtual-machines-shared-sap-high-availability-guide/3021-check-box-for-computer-objects.png
[sap-ha-guide-figure-3022]:./media/virtual-machines-shared-sap-high-availability-guide/3022-set-security-attributes-for-cluster-name-object-on-file-share-quorum.png
[sap-ha-guide-figure-3023]:./media/virtual-machines-shared-sap-high-availability-guide/3023-call-configure-cluster-quorum-setting-wizard.png
[sap-ha-guide-figure-3024]:./media/virtual-machines-shared-sap-high-availability-guide/3024-selection-screen-different-quorum-configurations.png
[sap-ha-guide-figure-3025]:./media/virtual-machines-shared-sap-high-availability-guide/3025-selection-screen-file-share-witness.png
[sap-ha-guide-figure-3026]:./media/virtual-machines-shared-sap-high-availability-guide/3026-define-file-share-location-for-witness-share.png
[sap-ha-guide-figure-3027]:./media/virtual-machines-shared-sap-high-availability-guide/3027-successful-reconfiguration-cluster-file-share-witness.png
[sap-ha-guide-figure-3028]:./media/virtual-machines-shared-sap-high-availability-guide/3028-install-dot-net-framework-35.png
[sap-ha-guide-figure-3029]:./media/virtual-machines-shared-sap-high-availability-guide/3029-install-dot-net-framework-35-progress.png
[sap-ha-guide-figure-3030]:./media/virtual-machines-shared-sap-high-availability-guide/3030-sios-installer.png
[sap-ha-guide-figure-3031]:./media/virtual-machines-shared-sap-high-availability-guide/3031-first-screen-sios-data-keeper-installation.png
[sap-ha-guide-figure-3032]:./media/virtual-machines-shared-sap-high-availability-guide/3032-data-keeper-informs-service-be-disabled.png
[sap-ha-guide-figure-3033]:./media/virtual-machines-shared-sap-high-availability-guide/3033-user-selection-sios-data-keeper.png
[sap-ha-guide-figure-3034]:./media/virtual-machines-shared-sap-high-availability-guide/3034-domain-user-sios-data-keeper.png
[sap-ha-guide-figure-3035]:./media/virtual-machines-shared-sap-high-availability-guide/3035-provide-sios-data-keeper-license.png
[sap-ha-guide-figure-3036]:./media/virtual-machines-shared-sap-high-availability-guide/3036-data-keeper-management-config-tool.png
[sap-ha-guide-figure-3037]:./media/virtual-machines-shared-sap-high-availability-guide/3037-tcp-ip-address-first-node-data-keeper.png
[sap-ha-guide-figure-3038]:./media/virtual-machines-shared-sap-high-availability-guide/3038-create-replication-sios-job.png
[sap-ha-guide-figure-3039]:./media/virtual-machines-shared-sap-high-availability-guide/3039-define-sios-replication-job-name.png
[sap-ha-guide-figure-3040]:./media/virtual-machines-shared-sap-high-availability-guide/3040-define-sios-source-node.png
[sap-ha-guide-figure-3041]:./media/virtual-machines-shared-sap-high-availability-guide/3041-define-sios-target-node.png
[sap-ha-guide-figure-3042]:./media/virtual-machines-shared-sap-high-availability-guide/3042-define-sios-synchronous-replication.png
[sap-ha-guide-figure-3043]:./media/virtual-machines-shared-sap-high-availability-guide/3043-enable-sios-replicated-volume-as-cluster-volume.png
[sap-ha-guide-figure-3044]:./media/virtual-machines-shared-sap-high-availability-guide/3044-data-keeper-synchronous-mirroring-for-SAP-gui.png
[sap-ha-guide-figure-3045]:./media/virtual-machines-shared-sap-high-availability-guide/3045-replicated-disk-by-data-keeper-in-wsfc.png
[sap-ha-guide-figure-3046]:./media/virtual-machines-shared-sap-high-availability-guide/3046-dns-entry-sap-ascs-virtual-name-ip.png
[sap-ha-guide-figure-3047]:./media/virtual-machines-shared-sap-high-availability-guide/3047-dns-manager.png
[sap-ha-guide-figure-3048]:./media/virtual-machines-shared-sap-high-availability-guide/3048-default-cluster-probe-port.png
[sap-ha-guide-figure-3049]:./media/virtual-machines-shared-sap-high-availability-guide/3049-cluster-probe-port-after.png
[sap-ha-guide-figure-3050]:./media/virtual-machines-shared-sap-high-availability-guide/3050-service-type-ers-delayed-automatic.png
[sap-ha-guide-figure-5000]:./media/virtual-machines-shared-sap-high-availability-guide/5000-wsfc-sap-sid-node-a.png
[sap-ha-guide-figure-5001]:./media/virtual-machines-shared-sap-high-availability-guide/5001-sios-replicating-local-volume.png
[sap-ha-guide-figure-5002]:./media/virtual-machines-shared-sap-high-availability-guide/5002-wsfc-sap-sid-node-b.png
[sap-ha-guide-figure-5003]:./media/virtual-machines-shared-sap-high-availability-guide/5003-sios-replicating-local-volume-b-to-a.png

[sap-ha-guide-figure-6003]:./media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png


[sap-ha-guide-figure-8001]:./media/virtual-machines-shared-sap-high-availability-guide/8001.png
[sap-ha-guide-figure-8002]:./media/virtual-machines-shared-sap-high-availability-guide/8002.png
[sap-ha-guide-figure-8003]:./media/virtual-machines-shared-sap-high-availability-guide/8003.png
[sap-ha-guide-figure-8004]:./media/virtual-machines-shared-sap-high-availability-guide/8004.png
[sap-ha-guide-figure-8005]:./media/virtual-machines-shared-sap-high-availability-guide/8005.png
[sap-ha-guide-figure-8006]:./media/virtual-machines-shared-sap-high-availability-guide/8006.png
[sap-ha-guide-figure-8007]:./media/virtual-machines-shared-sap-high-availability-guide/8007.png
[sap-ha-guide-figure-8008]:./media/virtual-machines-shared-sap-high-availability-guide/8008.png
[sap-ha-guide-figure-8009]:./media/virtual-machines-shared-sap-high-availability-guide/8009.png
[sap-ha-guide-figure-8010]:./media/virtual-machines-shared-sap-high-availability-guide/8010.png
[sap-ha-guide-figure-8011]:./media/virtual-machines-shared-sap-high-availability-guide/8011.png
[sap-ha-guide-figure-8012]:./media/virtual-machines-shared-sap-high-availability-guide/8012.png
[sap-ha-guide-figure-8013]:./media/virtual-machines-shared-sap-high-availability-guide/8013.png
[sap-ha-guide-figure-8014]:./media/virtual-machines-shared-sap-high-availability-guide/8014.png
[sap-ha-guide-figure-8015]:./media/virtual-machines-shared-sap-high-availability-guide/8015.png
[sap-ha-guide-figure-8016]:./media/virtual-machines-shared-sap-high-availability-guide/8016.png
[sap-ha-guide-figure-8017]:./media/virtual-machines-shared-sap-high-availability-guide/8017.png
[sap-ha-guide-figure-8018]:./media/virtual-machines-shared-sap-high-availability-guide/8018.png
[sap-ha-guide-figure-8019]:./media/virtual-machines-shared-sap-high-availability-guide/8019.png
[sap-ha-guide-figure-8020]:./media/virtual-machines-shared-sap-high-availability-guide/8020.png
[sap-ha-guide-figure-8021]:./media/virtual-machines-shared-sap-high-availability-guide/8021.png
[sap-ha-guide-figure-8022]:./media/virtual-machines-shared-sap-high-availability-guide/8022.png
[sap-ha-guide-figure-8023]:./media/virtual-machines-shared-sap-high-availability-guide/8023.png
[sap-ha-guide-figure-8024]:./media/virtual-machines-shared-sap-high-availability-guide/8024.png
[sap-ha-guide-figure-8025]:./media/virtual-machines-shared-sap-high-availability-guide/8025.png


[sap-templates-3-tier-multisid-xscs-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs%2Fazuredeploy.json
[sap-templates-3-tier-multisid-xscs-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps-md%2Fazuredeploy.json

[virtual-machines-azure-resource-manager-architecture-benefits-arm]:../../../azure-resource-manager/management/overview.md#the-benefits-of-using-resource-manager

[virtual-machines-manage-availability]:../../virtual-machines-windows-manage-availability.md