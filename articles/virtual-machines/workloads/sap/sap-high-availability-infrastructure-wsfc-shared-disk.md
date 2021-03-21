---
title: Инфраструктура Azure для SAP ASCS/SCS с WSFC&общим диском | Документация Майкрософт
description: Узнайте, как подготовить инфраструктуру Azure для SAP высокого уровня доступности с помощью отказоустойчивого кластера Windows и общего диска для экземпляра SAP ASCS/SCS.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ec976257-396b-42a0-8ea1-01c97f820fa6
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 10/16/2020
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 4218b4c00b79d78965eaf6e73028e63f52b1ff17
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101673629"
---
# <a name="prepare-the-azure-infrastructure-for-sap-ha-by-using-a-windows-failover-cluster-and-shared-disk-for-sap-ascsscs"></a>Подготовка инфраструктуры Azure для SAP высокого уровня доступности с помощью отказоустойчивого кластера Windows и общего диска для SAP ASCS/SC

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides
[tuning-failover-cluster-network-thresholds]:https://techcommunity.microsoft.com/t5/Failover-Clustering/Tuning-Failover-Cluster-Network-Thresholds/ba-p/371834

[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md

[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md

[ha-guide]:sap-high-availability-guide.md
[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-ha-guide-9.1.1]:high-availability-guide.md#a97ad604-9094-44fe-a364-f89cb39bf097
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f


[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-shared-disk-vpn]:sap-high-availability-infrastructure-wsfc-shared-disk.md#c87a8d3f-b1dc-4d2f-b23c-da4b72977489
[sap-high-availability-infrastructure-wsfc-shared-disk-change-def-ilb]:sap-high-availability-infrastructure-wsfc-shared-disk.md#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-high-availability-infrastructure-wsfc-shared-disk-setup-wsfc]:sap-high-availability-infrastructure-wsfc-shared-disk.md#0d67f090-7928-43e0-8772-5ccbf8f59aab
[sap-high-availability-infrastructure-wsfc-shared-disk-collect-cluster-config]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5c8e5482-841e-45e1-a89d-a05c0907c868
[sap-high-availability-infrastructure-wsfc-shared-disk-add-dot-net]:sap-high-availability-infrastructure-wsfc-shared-disk.md#1c2788c3-3648-4e82-9e0d-e058e475e2a3
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios-both-nodes]:sap-high-availability-infrastructure-wsfc-shared-disk.md#dd41d5a2-8083-415b-9878-839652812102
[sap-high-availability-infrastructure-wsfc-shared-disk-setup-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#d9c1fc8e-8710-4dff-bec2-1f535db7b006

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

[sap-templates-3-tier-multisid-xscs-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs%2Fazuredeploy.json
[sap-templates-3-tier-multisid-xscs-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps-md%2Fazuredeploy.json

[virtual-machines-azure-resource-manager-architecture-benefits-arm]:../../../azure-resource-manager/management/overview.md#the-benefits-of-using-resource-manager

[virtual-machines-manage-availability]:../../virtual-machines-windows-manage-availability.md


> ![ОС Windows][Logo_Windows] Windows


В этой статье описаны действия по подготовке инфраструктуры Azure к установке и настройке высокодоступного экземпляра SAP ASCS/SCS в отказоустойчивом кластере Windows с помощью *общего диска кластера* в качестве варианта кластеризации экземпляра SAP ASCS.
В документации представлены два варианта *общего диска кластера* :

- [Общие диски Azure](../../disks-shared.md)
- Использование [SIOS Cluster Edition](https://us.sios.com/products/datakeeper-cluster/) для создания зеркально отображаемого хранилища, которое имитирует кластерный общий диск 

Представленная конфигурация полагается на [группы размещения Azure (ППГ)](./sap-proximity-placement-scenarios.md) , чтобы обеспечить оптимальную задержку в сети для рабочих нагрузок SAP. В документации не рассматривается уровень базы данных.  

> [!NOTE]
> Группы размещения службы "близость" Azure являются необходимым условием для использования общего диска Azure.
 

## <a name="prerequisites"></a>Предварительные условия

Перед началом установки ознакомьтесь со статьей:

* [Кластеризация экземпляра SAP (A)SCS в отказоустойчивом кластере Windows с помощью общего диска кластера в Azure][sap-high-availability-guide-wsfc-shared-disk].

## <a name="create-the-ascs-vms"></a>Создание виртуальных машин ASCS

Для кластера SAP ASCS/SCS разверните две виртуальные машины в группе доступности Azure. Разверните виртуальные машины в одной группе размещения с учетом расположения. После развертывания виртуальных машин выполните следующие действия.  
- Создание внутреннего Load Balancer Azure для экземпляра SAP ASCS/SCS 
- Добавление виртуальных машин Windows в домен AD

Имена узлов и IP-адреса для представленного сценария:

| Роль имени узла | Имя узла | Статический IP-адрес | Группа доступности | Группа размещения с учетом расположения |
| --- | --- | --- |---| ---|
| 1-й кластер узлов кластера ASCS/SCS |PR1-ASCS-10 |10.0.0.4 |PR1-ASCS-avset |PR1PPG |
| кластер ASCS/SCS второго узла кластера |PR1-ASCS-11 |10.0.0.5 |PR1-ASCS-avset |PR1PPG |
| Сетевое имя кластера | pr1clust |10.0.0.42 (**только** для кластера Win 2016) | Н/Д | Н/Д |
| Сетевое имя кластера ASCS | PR1 — аскскл |10.0.0.43 | Н/Д | Н/Д |
| Сетевое имя кластера ERS (**только** для ERS2) | PR1 — ерскл |10.0.0.44 | Н/Д | Н/Д |


## <a name="create-azure-internal-load-balancer"></a><a name="fe0bd8b5-2b43-45e3-8295-80bee5415716"></a> Создание внутренней подсистемы балансировки нагрузки Azure

SAP ASCS, SAP SCS и New SAP ERS2 используют виртуальное имя узла и виртуальные IP-адреса. В Azure для использования виртуального IP-адреса требуется [балансировщик нагрузки](../../../load-balancer/load-balancer-overview.md) . Настоятельно рекомендуется использовать [стандартный балансировщик нагрузки](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md). 

> [!IMPORTANT]
> Плавающий IP-адрес не поддерживается для вторичной IP-конфигурации NIC в сценариях балансировки нагрузки. Дополнительные сведения см. в статье [ограничения балансировщика нагрузки Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Если для виртуальной машины требуется дополнительный IP-адрес, разверните вторую сетевую карту.    


В следующем списке показана конфигурация балансировщика нагрузки (A) SCS/ERS. Конфигурация SAP ASCS и ERS2 выполняется в одной и той же подсистеме балансировки нагрузки Azure.  

**(A)SCS**
- Конфигурация внешнего интерфейса:
    - Статический IP-адрес ASCS/SCS **10.0.0.43**
- Конфигурация серверной части:  
    Добавьте все виртуальные машины, которые должны входить в состав кластера (A) SCS/ERS. В этом примере виртуальные машины **PR1-ASCS-10** и **PR1-ASCS-11**.
- Порт пробы:
    - Порт 620 **Nr** оставьте параметр по умолчанию для протокола TCP, Interval (5), порог неработоспособности (2)
- Правила балансировки нагрузки
    - Если используется Load Balancer (цен. категория "Стандартный"), выберите порты высокой доступности.
    - Если используется Load Balancer (цен. категория "Базовый"), создайте правила балансировки нагрузки для следующих портов.
        - 32 **nr** TCP;
        - 36 **nr** TCP;
        - 39 **nr** TCP;
        - 81 **nr** TCP;
        - 5 **nr** 13 TCP;
        - 5 **nr** 14 TCP;
        - 5 **nr** 16 TCP.

    - Убедитесь, что для параметра время ожидания простоя (в минутах) задано максимальное значение 30, а этот плавающий IP-адрес включен (прямой возврат сервера).

**ERS2**

Так как сервер репликации с очередью 2 (ERS2) также является кластеризованным, виртуальный IP-адрес ERS2 также должен быть настроен в Azure ILB в дополнение к выше IP-адреса SAP ASCS/SCS. Этот раздел применяется, только если используется архитектура серверной очереди репликации 2.  
- Конфигурация 2-го интерфейса
    - Статический IP-адрес SAP ERS2 **10.0.0.44**

- Конфигурация серверной части:  
  Виртуальные машины уже добавлены в серверный пул ILB.  

- второй порт пробы
    - Порт 621 **nr**.  
    Оставьте параметр по умолчанию для протокола (TCP), Interval (5), порог неработоспособности (2)

- 2 правила балансировки нагрузки
    - Если используется Load Balancer (цен. категория "Стандартный"), выберите порты высокой доступности.
    - Если используется Load Balancer (цен. категория "Базовый"), создайте правила балансировки нагрузки для следующих портов.
        - 32 **nr** TCP;
        - 33 **nr** TCP;
        - 5 **nr** 13 TCP;
        - 5 **nr** 14 TCP;
        - 5 **nr** 16 TCP.

    - Убедитесь, что для параметра время ожидания простоя (в минутах) задано максимальное значение 30, а этот плавающий IP-адрес включен (прямой возврат сервера).


> [!TIP]
> С помощью [шаблона Azure Resource Manager для экземпляра кластера WSFC для SAP ASCS/SCS с общим диском Azure](https://github.com/robotechredmond/301-shared-disk-sap)можно автоматизировать подготовку инфраструктуры, используя общий диск Azure для одного идентификатора безопасности SAP с ERS1.  
> Шаблон Azure ARM создаст две виртуальные машины Windows 2019 или 2016, создайте общий диск Azure и подключите их к виртуальным машинам. Также будут созданы и настроены внутренние Load Balancer Azure. Дополнительные сведения см. в разделе Шаблон ARM. 

## <a name="add-registry-entries-on-both-cluster-nodes-of-the-ascsscs-instance"></a><a name="661035b2-4d0f-4d31-86f8-dc0a50d78158"></a> Добавление записей реестра на обоих узлах кластера экземпляра ASCS/SCS

Azure Load Balancer может закрывать соединения, если соединения бездействуют в течение некоторого времени и превышают время ожидания простоя. Рабочие процессы SAP открывают подключения к процессу постановки в очередь SAP сразу же после отправки первого запроса в очередь или из очереди. Чтобы избежать прерывания этих подключений, измените значения параметров KeepAliveTime и KeepAliveInterval TCP/IP на обоих узлах кластера. При использовании ERS1 также необходимо добавить параметры профиля SAP, как описано далее в этой статье.
На обоих узлах кластера необходимо изменить следующие записи реестра:

- KeepAliveTime
- KeepAliveInterval

| Путь| Имя переменной | Тип переменной  | Значение | Документация |
| --- | --- | --- |---| ---|
| HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |KeepAliveTime |REG_DWORD (десятичное) |120000 |[KeepAliveTime](/previous-versions/windows/it-pro/windows-2000-server/cc957549(v=technet.10)) |
| HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |KeepAliveInterval |REG_DWORD (десятичное) |120000 |[KeepAliveInterval](/previous-versions/windows/it-pro/windows-2000-server/cc957548(v=technet.10)) |


Чтобы применить изменения, перезапустите оба узла кластера.
  
## <a name="add-the-windows-vms-to-the-domain"></a><a name="e69e9a34-4601-47a3-a41c-d2e11c626c0c"></a> Добавление виртуальных машин Windows в домен
После назначения статических IP-адресов виртуальным машинам добавьте виртуальные машины в домен. 

## <a name="install-and-configure--windows-failover-cluster"></a><a name="0d67f090-7928-43e0-8772-5ccbf8f59aab"></a> Установка и настройка отказоустойчивого кластера Windows 

### <a name="install-the-windows-failover-cluster-feature"></a>Установка компонента отказоустойчивого кластера Windows

Выполните эту команду на одном из узлов кластера:

   ```powershell
    # Hostnames of the Win cluster for SAP ASCS/SCS
    $SAPSID = "PR1"
    $ClusterNodes = ("pr1-ascs-10","pr1-ascs-11")
    $ClusterName = $SAPSID.ToLower() + "clust"
    
    # Install Windows features.
    # After the feature installs, manually reboot both nodes
    Invoke-Command $ClusterNodes {Install-WindowsFeature Failover-Clustering, FS-FileServer -IncludeAllSubFeature -IncludeManagementTools }
   ```

После завершения установки компонентов Перезагрузите оба узла кластера.  

### <a name="test-and-configure-windows-failover-cluster"></a>Тестирование и настройка отказоустойчивого кластера Windows 

В Windows 2019 кластер автоматически распознает, что он выполняется в Azure, и в качестве параметра по умолчанию для IP-адреса управления кластерами будет использоваться имя распределенной сети. Поэтому он будет использовать любые локальные IP-адреса узлов кластера. В результате нет нужды в выделенном (виртуальном) сетевом имени кластера, и нет необходимости настраивать этот IP-адрес на внутренних Load Balancerах Azure.

Дополнительные сведения см. в разделе [отказоустойчивая кластеризация Windows Server 2019 новые функции](https://techcommunity.microsoft.com/t5/failover-clustering/windows-server-2019-failover-clustering-new-features/ba-p/544029) выполните эту команду на одном из узлов кластера:

   ```powershell
    # Hostnames of the Win cluster for SAP ASCS/SCS
    $SAPSID = "PR1"
    $ClusterNodes = ("pr1-ascs-10","pr1-ascs-11")
    $ClusterName = $SAPSID.ToLower() + "clust"
    
    # IP adress for cluster network name is needed ONLY on Windows Server 2016 cluster
    $ClusterStaticIPAddress = "10.0.0.42"
        
    # Test cluster
    Test-Cluster –Node $ClusterNodes -Verbose
    
    $ComputerInfo = Get-ComputerInfo
    
    $WindowsVersion = $ComputerInfo.WindowsProductName
    
    if($WindowsVersion -eq "Windows Server 2019 Datacenter"){
        write-host "Configuring Windows Failover Cluster on Windows Server 2019 Datacenter..."
        New-Cluster –Name $ClusterName –Node  $ClusterNodes -Verbose
    }elseif($WindowsVersion -eq "Windows Server 2016 Datacenter"){
        write-host "Configuring Windows Failover Cluster on Windows Server 2016 Datacenter..."
        New-Cluster –Name $ClusterName –Node  $ClusterNodes –StaticAddress $ClusterStaticIPAddress -Verbose 
    }else{
        Write-Error "Not supported Windows version!"
    }
   ```

### <a name="configure-cluster-cloud-quorum"></a>Настройка кворума облака кластера
При использовании Windows Server 2016 или 2019 рекомендуется настроить [облачный следящий сервер Azure](/windows-server/failover-clustering/deploy-cloud-witness)в качестве кворума кластера.

Выполните эту команду на одном из узлов кластера:

   ```powershell
    $AzureStorageAccountName = "cloudquorumwitness"
    Set-ClusterQuorum –CloudWitness –AccountName $AzureStorageAccountName -AccessKey <YourAzureStorageAccessKey> -Verbose
   ```

### <a name="tuning-the-windows-failover-cluster-thresholds"></a>Настройка пороговых значений отказоустойчивого кластера Windows
 
После успешной установки отказоустойчивого кластера Windows необходимо настроить некоторые пороговые значения, подходящие для кластеров, развернутых в Azure. Параметры, которые необходимо изменить, описаны в записи блога [Tuning Failover Cluster Network Thresholds](https://techcommunity.microsoft.com/t5/Failover-Clustering/Tuning-Failover-Cluster-Network-Thresholds/ba-p/371834) (Настройка пороговых значений сети отказоустойчивого кластера). Предположим, что две виртуальные машины, образующие конфигурацию кластера Windows для ASCS/SCS, находятся в одной подсети. Тогда нужно изменить значения приведенных ниже параметров следующими значениями:
- SameSubNetDelay = 2000
- SameSubNetThreshold = 15
- RoutingHistoryLength = 30

Эти параметры были протестированы с клиентами и предлагают хороший компромисс. Они достаточно устойчивы, но они также обеспечивают отработку отказа, которая достаточно быстра для реальных условий возникновения ошибок в рабочих нагрузках SAP или сбоях виртуальной машины.  

## <a name="configure-azure-shared-disk"></a>Настройка общего диска Azure
Этот раздел применим только при использовании общего диска Azure. 

### <a name="create-and-attach-azure-shared-disk-with-powershell"></a>Создание и подключение общего диска Azure с помощью PowerShell
Выполните эту команду на одном из узлов кластера. Вам потребуется настроить значения для группы ресурсов, региона Azure, SAPSID и т. д.  

   ```powershell
    #############################
    # Create Azure Shared Disk
    #############################
    
    $ResourceGroupName = "MyResourceGroup"
    $location = "MyAzureRegion"
    $SAPSID = "PR1"
    
    $DiskSizeInGB = 512
    $DiskName = "$($SAPSID)ASCSSharedDisk"
    
    # With parameter '-MaxSharesCount', we define the maximum number of cluster nodes to attach the shared disk
    $NumberOfWindowsClusterNodes = 2
            
    $diskConfig = New-AzDiskConfig -Location $location -SkuName Premium_LRS  -CreateOption Empty  -DiskSizeGB $DiskSizeInGB -MaxSharesCount $NumberOfWindowsClusterNodes
    $dataDisk = New-AzDisk -ResourceGroupName $ResourceGroupName -DiskName $DiskName -Disk $diskConfig
    
    ##################################
    ## Attach the disk to cluster VMs
    ##################################
    # ASCS Cluster VM1
    $ASCSClusterVM1 = "$SAPSID-ascs-10"
    
    # ASCS Cluster VM2
    $ASCSClusterVM2 = "$SAPSID-ascs-11"
    
    # Add the Azure Shared Disk to Cluster Node 1
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $ASCSClusterVM1 
    $vm = Add-AzVMDataDisk -VM $vm -Name $DiskName -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 0
    Update-AzVm -VM $vm -ResourceGroupName $ResourceGroupName -Verbose
    
    # Add the Azure Shared Disk to Cluster Node 2
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $ASCSClusterVM2
    $vm = Add-AzVMDataDisk -VM $vm -Name $DiskName -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 0
    Update-AzVm -VM $vm -ResourceGroupName $ResourceGroupName -Verbose
   ```

### <a name="format-the-shared-disk-with-powershell"></a>Форматирование общего диска с помощью PowerShell
1. Получите номер диска. Выполните следующие команды PowerShell на одном из узлов кластера:

   ```powershell
    Get-Disk | Where-Object PartitionStyle -Eq "RAW"  | Format-Table -AutoSize 
    # Example output
    # Number Friendly Name     Serial Number HealthStatus OperationalStatus Total Size Partition Style
    # ------ -------------     ------------- ------------ ----------------- ---------- ---------------
    # 2      Msft Virtual Disk               Healthy      Online                512 GB RAW            

   ```
2. Отформатируйте диск. В этом примере это номер диска 2. 

   ```powershell
    # Format SAP ASCS Disk number '2', with drive letter 'S'
    $SAPSID = "PR1"
    $DiskNumber = 2
    $DriveLetter = "S"
    $DiskLabel = "$SAPSID" + "SAP"
    
    Get-Disk -Number $DiskNumber | Where-Object PartitionStyle -Eq "RAW" | Initialize-Disk -PartitionStyle GPT -PassThru |  New-Partition -DriveLetter $DriveLetter -UseMaximumSize | Format-Volume  -FileSystem ReFS -NewFileSystemLabel $DiskLabel -Force -Verbose
    # Example outout
    # DriveLetter FileSystemLabel FileSystem DriveType HealthStatus OperationalStatus SizeRemaining      Size
    # ----------- --------------- ---------- --------- ------------ ----------------- -------------      ----
    # S           PR1SAP          ReFS       Fixed     Healthy      OK                    504.98 GB 511.81 GB
   ```

3. Убедитесь, что диск теперь отображается как диск кластера.  
   ```powershell
    # List all disks
    Get-ClusterAvailableDisk -All
    # Example output
    # Cluster    : pr1clust
    # Id         : 88ff1d94-0cf1-4c70-89ae-cbbb2826a484
    # Name       : Cluster Disk 1
    # Number     : 2
    # Size       : 549755813888
    # Partitions : {\\?\GLOBALROOT\Device\Harddisk2\Partition2\}
   ```
4. Зарегистрируйте диск в кластере.  
   ```powershell
    # Add the disk to cluster 
    Get-ClusterAvailableDisk -All | Add-ClusterDisk
    # Example output     
    # Name           State  OwnerGroup        ResourceType 
    # ----           -----  ----------        ------------ 
    # Cluster Disk 1 Online Available Storage Physical Disk
   ```

## <a name="sios-datakeeper-cluster-edition-for-the-sap-ascsscs-cluster-share-disk"></a><a name="5c8e5482-841e-45e1-a89d-a05c0907c868"></a> SIOS Cluster Edition для общего диска кластера SAP ASCS/SCS
Этот раздел применим только в том случае, если вы используете стороннее программное обеспечение SIOS Cluster Edition для создания зеркально отображаемого хранилища, моделирующего общий диск кластера.  

Теперь у вас есть рабочая конфигурация отказоустойчивого кластера Windows Server в Azure. Чтобы установить экземпляр SAP ASCS/SCS, требуется общий дисковый ресурс. Один из вариантов — использовать SIOS Clustering Edition — это решение стороннего производителя, которое можно использовать для создания общих дисковых ресурсов.  

Установка SIOS DataKeeper Cluster Edition для общего диска кластера SAP ASCS/SCS состоит из следующих заданий:
- При необходимости добавьте Microsoft .NET Framework. См. [документацию по SIOS] (( https://us.sios.com/products/datakeeper-cluster/) для наиболее актуальных требований к платформе .NET Framework 
-  Установка SIOS DataKeeper
- Настройка SIOS

### <a name="install-sios-datakeeper"></a> Установка SIOS DataKeeper
SIOS DataKeeper Cluster Edition нужно установить на обоих узлах в кластере. Чтобы создать виртуальное общее хранилище с помощью SIOS DataKeeper, создайте синхронизированное зеркало и сымитируйте общее хранилище кластера.

Перед установкой программного обеспечения SIOS необходимо создать пользователя домена DataKeeperSvc.

> [!NOTE]
> Добавьте пользователя домена DataKeeperSvc в группу локальных администраторов на обоих узлах кластера.
>

1. Установите программное обеспечение SIOS на обоих узлах кластера.

   ![Установщик SIOS][sap-ha-guide-figure-3030]

   ![Рис. 31. Первая страница установки SIOS DataKeeper][sap-ha-guide-figure-3031]

   _Первая страница установки SIOS_

2. В диалоговом окне выберите **Да**.

   ![Рис. 32. DataKeeper информирует об отключении службы][sap-ha-guide-figure-3032]

   _При этом будет отображаться сообщение о том, что служба отключена._

3. В диалоговом окне мы рекомендуем установить флажок **Domain or Server account** (Учетная запись домена или сервера).

   ![Рис. 33. Выбор пользователя для SIOS DataKeeper][sap-ha-guide-figure-3033]

   _Выбор пользователя для SIOS DataKeeper_

4. Укажите имя учетной записи пользователя домена и пароль, созданные для SIOS DataKeeper.

   ![Рис. 34. Ввод имени пользователя и пароля для установки SIOS DataKeeper][sap-ha-guide-figure-3034]

   _Ввод имени пользователя и пароля для установки SIOS DataKeeper_

5. Установите лицензионный ключ для своего экземпляра SIOS DataKeeper, как показано на рис. 35.

   ![Рис. 35. Указание ключа лицензии на использование SIOS DataKeeper][sap-ha-guide-figure-3035]

   _Введите ключ лицензии SIOS._

6. При появлении запроса перезапустите виртуальную машину.

### <a name="configure-sios-datakeeper"></a>Настройка SIOS
После установки SIOS DataKeeper на обоих узлах начните настройку параметров. Наша цель — настроить синхронную репликацию данных между дополнительными дисками, подключенными к каждой виртуальной машине.

1. Запустите инструмент управления и настройки DataKeeper, а затем щелкните **Подключение к серверу**.

   ![Рис. 36. Инструмент управления и настройки DataKeeper][sap-ha-guide-figure-3036]

   _Инструмент управления и настройки SIOS DataKeeper_

2. Введите имя или TCP/IP-адрес первого узла, к которому должен подключиться инструмент управления и настройки, а на втором шаге — для второго узла.

   ![Рис. 37. Вставка имени или TCP/IP-адреса первого узла, к которому должен подключиться инструмент управления. На втором шаге — вставка данных для второго узла.][sap-ha-guide-figure-3037]

   _Вставьте имя или TCP/IP-адрес первого узла, к которому должно подключиться средство управления и настройки, а на втором шаге — второй узел._

3. Создайте задание репликации между двумя узлами.

   ![Рис. 38. Создание задания репликации][sap-ha-guide-figure-3038]

   _Создание задания репликации_

   Мастер поможет выполнить эту процедуру.

4. Определите имя задания репликации.

   ![Рис. 39. Определение имени задания репликации][sap-ha-guide-figure-3039]

   _Определение имени для задания репликации_

   ![Рис. 40. Определение базовых данных для узла, который должен быть текущим исходным узлом][sap-ha-guide-figure-3040]

   _Определите базовые данные для узла, который должен быть текущим исходным узлом_

5. Определите имя, TCP/IP-адрес и дисковый том целевого узла.

   ![Рис. 41. Определение имени, TCP/IP-адреса и дискового тома текущего целевого узла][sap-ha-guide-figure-3041]

   _Определение имени, TCP/IP-адреса и дискового тома текущего целевого узла_

6. Определите алгоритм сжатия. Для наших целей рекомендуется сжатие потока репликации. Сжатие потока репликации значительно сокращает время повторной синхронизации (особенно в случаях повторной синхронизации). Сжатие использует ресурсы ЦП и памяти виртуальной машины. Чем выше степень сжатия, тем больше том используемых ресурсов ЦП. Этот параметр можно будет откорректировать и изменить позже.

7. Кроме того, нужно проверить параметр, который позволяет включить синхронный или асинхронный режим репликации. Для защиты конфигураций SAP ASCS/SCS требуется синхронная репликация.  

   ![Рис. 42. Определение сведений о репликации][sap-ha-guide-figure-3042]

   _Определение сведений о репликации_

8. Определите, следует ли том, реплицируемый с помощью задания репликации, представить в конфигурации кластера WSFC как общий диск. Для конфигурации SAP ASCS/SCS выберите **Да**, чтобы кластер Windows воспринимал реплицируемый том как общий диск, который можно использовать в качестве тома кластера.

   ![Рис. 43. Чтобы задать реплицируемый том как том кластера, следует нажать кнопку "Да"][sap-ha-guide-figure-3043]

   _Выберите **Да** , чтобы задать реплицированный том как том кластера_

   После создания тома инструмент управления и настройки DataKeeper отобразят задание репликации как активное.

   ![Рис. 44. Синхронное зеркальное отображение DataKeeper для общего диска SAP ASCS/SCS активно][sap-ha-guide-figure-3044]

   _Синхронное зеркальное отображение DataKeeper для общего диска SAP ASCS/SCS активно_

   Теперь диск отображается в диспетчере отказоустойчивости кластеров как диск DataKeeper, как показано на рис. 45:

   ![Рис. 45. Реплицируемый диск DataKeeper отображается в диспетчере отказоустойчивости кластеров][sap-ha-guide-figure-3045]

   _Реплицируемый диск DataKeeper отображается в диспетчере отказоустойчивости кластеров_


## <a name="next-steps"></a>Дальнейшие действия

* [Установка SAP NetWeaver высокого уровня доступности в Azure с использованием отказоустойчивого кластера Windows и общего диска для экземпляра SAP (A)SCS][sap-high-availability-installation-wsfc-shared-disk]