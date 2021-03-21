---
title: Руководство по обеспечению высокого уровня доступности SAP NetWeaver на виртуальных машинах Azure
description: В этой статье вы узнаете о высокой доступности виртуальных машин Azure для SAP NetWeaver.
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
ms.date: 05/05/2017
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 67b472d1629c2b59f80d36aaf43df1e783d2ffa4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101666810"
---
# <a name="high-availability-azure-virtual-machines-for-sap-netweaver"></a>Виртуальные машины Azure с высоким уровнем доступности для SAP NetWeaver

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides

[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md

[dbms-guide]:dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md

[ha-guide]:sap-high-availability-guide.md

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



Виртуальные машины Azure позволяют организациям быстро получать вычислительные ресурсы, ресурсы хранилища и сетевые ресурсы без длительных циклов закупки. С помощью виртуальных машин Azure можно развернуть в Azure классические приложения, например приложения на основе SAP NetWeaver (ABAP, Java и ABAP со стеком Java), и повысить их надежность и доступность без локального развертывания ресурсов. Повысьте уровень надежности и доступности без привлечения дополнительных локальных ресурсов. Виртуальные машины Azure поддерживают распределенные подключения. Это позволяет интегрировать их в локальные домены, частные облака и системный ландшафт SAP вашей организации.

В этой статье описаны действия, необходимые для развертывания в Azure систем SAP с высоким уровнем доступности, используя модель развертывания с помощью Azure Resource Manager. Здесь рассматриваются следующие основные задачи:

* Поиск соответствующих руководств по установке SAP и примечаний, приведенных в разделе [Материалы][sap-ha-guide-2]. Это руководство дополняет документацию по установке SAP и примечания к SAP, которые являются основными ресурсами по установке и развертыванию ПО SAP на определенных платформах.
* Изучение различий между классической моделью развертывания Azure и моделью развертывания с помощью Azure Resource Manager.
* Понимание режимов кворума отказоустойчивой кластеризации Windows Server, которое позволит выбрать модель, подходящую для развертывания Azure.
* Понимание принципов работы общего хранилища отказоустойчивого кластера Windows Server в Azure.
* Понимание методов защиты в Azure компонентов SAP, представляющих собой единственную точку отказа (например, SAP Central Services (SCS) / Advanced Business Application Programming SAP Central Services (ASCS) и системы управления базами данных (СУБД )), и избыточных компонентов (например, серверы приложений SAP).
* Выполнение пошаговых инструкций по установке и настройке системы SAP с высоким уровнем доступности в отказоустойчивом кластере Windows Server в Azure с помощью Azure Resource Manager.
* Дополнительные шаги, необходимые для использования WSFC в Azure, которые не требуется выполнять в локальных развертываниях.

Чтобы упростить развертывание и настройку, в этой статье используются шаблоны Resource Manager для трехуровневой конфигурации SAP с высоким уровнем доступности. Шаблоны автоматизируют развертывание полной инфраструктуры, необходимой для высокодоступной системы SAP. Эта инфраструктура также поддерживает метрику производительности SAP Application Performance Standard (SAPS) для вашей системы SAP.

## <a name="prerequisites"></a><a name="217c5479-5595-4cd8-870d-15ab00d4f84c"></a> Предварительные требования
Перед началом работы необходимо выполнить все предварительные условия, описанные в следующих разделах. Кроме того, ознакомьтесь со всеми материалами, указанными в разделе [Материалы][sap-ha-guide-2].

В этой статье мы используем шаблоны Azure Resource Manager для [трехуровневой платформы SAP NetWeaver, использующей управляемые диски](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md/). Обзор шаблонов представлен в статье о [шаблонах Azure Resource Manager для SAP](/archive/blogs/saponsqlserver/azure-quickstart-templates-for-sap).

## <a name="resources"></a><a name="42b8f600-7ba3-4606-b8a5-53c4f026da08"></a> Ресурсы
Дополнительные сведения о развертывании SAP в Azure см. в следующих статьях:

* [SAP NetWeaver на виртуальных машинах Windows. Руководство по планированию и внедрению][planning-guide]
* [Развертывание виртуальных машин Azure для SAP NetWeaver][deployment-guide]
* [SAP NetWeaver на виртуальных машинах Windows. Руководство по развертыванию СУБД][dbms-guide]
* [Руководство по обеспечению высокого уровня доступности SAP NetWeaver на виртуальных машинах Azure (это руководство)][sap-ha-guide]

> [!NOTE]
> При возможности мы предоставляем ссылки на соответствующее руководство по установке SAP (см. [руководства по установке SAP][sap-installation-guides]). Что касается необходимых компонентов и процесса установки, внимательно изучите руководства по установке SAP NetWeaver. В этой статье рассматриваются только отдельные задачи для систем на основе SAP NetWeaver, устанавливаемых на виртуальные машины Microsoft Azure.
>
>

Следующие примечания по SAP актуальны для развертывания SAP в Azure.

| Номер примечания | Title |
| --- | --- |
| [1928533] |Приложения SAP в Azure: Поддерживаемые продукты и размеры |
| [2015553] |SAP в Microsoft Azure: необходимые компоненты для поддержки |
| [1999351] |Расширенный мониторинг Azure для SAP |
| [2178632] |Ключевые метрики мониторинга для SAP в Microsoft Azure |
| [1999351] |Виртуализация в Windows: расширенный мониторинг |
| [2243692] |Использование хранилища SSD класса Premium Azure для экземпляра СУБД SAP |

Узнайте больше об [ограничениях для подписок Azure][azure-resource-manager/management/azure-subscription-service-limits-subscription], включая общие ограничения по умолчанию и максимальные ограничения.

## <a name="high-availability-sap-with-azure-resource-manager-vs-the-azure-classic-deployment-model"></a><a name="42156640c6-01cf-45a9-b225-4baa678b24f1"></a>Сравнение SAP с высоким уровнем доступности в модели развертывания с помощью Azure Resource Manager и классической модели развертывания Azure
Модель развертывания с помощью Azure Resource Manager и классическая модель развертывания Azure отличаются в следующих областях:

- Группы ресурсов
- зависимость внутреннего балансировщика нагрузки Azure от группы ресурсов Azure;
- поддержка сценариев с несколькими идентификаторами безопасности SAP.

### <a name="resource-groups"></a><a name="f76af273-1993-4d83-b12d-65deeae23686"></a> Группы ресурсов
Группы ресурсов в Azure Resource Manager можно использовать для управления всеми ресурсами приложения в подписке Azure. В рамках интегрированного подхода в группе ресурсов все ресурсы имеют одинаковый цикл жизни. Например, все они создаются одновременно и удаляются тоже одновременно. Дополнительная информация о [группах ресурсов](../../../azure-resource-manager/management/overview.md#resource-groups).

### <a name="azure-internal-load-balancer-dependency-on-the-azure-resource-group"></a><a name="3e85fbe0-84b1-4892-87af-d9b65ff91860"></a> Зависимость внутреннего балансировщика нагрузки Azure от группы ресурсов Azure

В классической модели развертывания Azure между внутренним балансировщиком нагрузки (службой Azure Load Balancer) и облачной службой существует зависимость. Для каждого внутреннего балансировщика нагрузки требуется одна облачная служба.

В Azure Resource Manager каждый ресурс Azure должен быть помещен в группу ресурсов Azure, и это правило также действует для Azure Load Balancer. При этом нет необходимости создавать по группе ресурсов Azure для каждого Azure Load Balancer. То есть в одной группе ресурсов Azure может быть несколько Azure Load Balancer. Среда в этом случае не такая сложная и более гибкая. 

### <a name="support-for-sap-multi-sid-scenarios"></a>поддержка сценариев с несколькими идентификаторами безопасности SAP.

В модели Azure Resource Manager в одном кластере можно установить несколько экземпляров SAP ASCS/SCS с системными идентификаторами (идентификаторами безопасности). Это возможно за счет поддержки каждым внутренним балансировщиком нагрузки Azure нескольких IP-адресов.

Чтобы использовать классическую модель развертывания Azure, выполните процедуру из [документа по SAP NetWeaver в Azure, посвященного кластеризации в Azure экземпляров SAP ASCS/SCS с помощью отказоустойчивых кластеров Windows Server с использованием SIOS DataKeeper](https://go.microsoft.com/fwlink/?LinkId=613056).

> [!IMPORTANT]
> Для установки приложений SAP настоятельно рекомендуется использовать модель развертывания с помощью Azure Resource Manager. Она предлагает преимущества, недоступные в классической модели развертывания. См. дополнительные сведения о [моделях развертывания Azure][virtual-machines-azure-resource-manager-architecture-benefits-arm].   
>
>

## <a name="windows-server-failover-clustering"></a><a name="8ecf3ba0-67c0-4495-9c14-feec1a2255b7"></a> Отказоустойчивый кластер Windows Server (WSFC)
Отказоустойчивая кластеризация Windows Server является основой для установки высокодоступных приложений SAP ASCS/SCS установки и СУБД в Windows.

Отказоустойчивый кластер представляет собой группу из 1 + n независимых серверов (узлов), работающих совместно для повышения доступности приложений и служб. В случае отказа узла отказоустойчивый кластер Windows Server определяет количество возможных сбоев, которые могут произойти во время восстановления работоспособности кластера для предоставления приложений и служб. Возможность отказоустойчивой кластеризации можно добавить, используя разные режимы кворума.

### <a name="quorum-modes"></a><a name="1a3c5408-b168-46d6-99f5-4219ad1b1ff2"></a> Режимы кворума
При использовании WSFC доступно четыре режима кворума.

* **Большинство узлов**. Голосовать может каждый узел кластера. Кластер работает только при большинстве голосов (более половины). Этот вариант рекомендуется при наличии нечетного числа узлов. Пример: в кластере из 7 узлов могут отказать 3 узла, при этом в нем сохранится большинство голосов и кластер продолжит работу.  
* **Большинство узлов и диска**. Голосовать может каждый узел и указанный диск в хранилище кластера (диск-свидетель), когда они доступны и могут обмениваться данными. Кластер работает только при большинстве голосов (более половины). Этот режим целесообразно использовать в среде кластера с четным числом узлов. Пока половина узлов и диск находятся в сети, кластер находится в работоспособном состоянии.
* **Большинство узлов и файлового ресурса**. Голосовать может каждый узел, а также указанный файловый ресурс, созданный администратором (файловый ресурс-свидетель), независимо от того, доступны ли они и могут ли обмениваться данными. Кластер работает только при большинстве голосов (более половины). Этот режим целесообразно использовать в среде кластера с четным числом узлов. Он похож на режим большинства узлов и дисков, только вместо диска-свидетеля используется файловый ресурс-свидетель. Данный режим прост в реализации, но если файловый ресурс не является высокодоступным, то он может оказаться единственной точкой отказа.
* **Нет большинства. Только диск**. В кластере есть кворум, если доступен один узел, подключенный к определенному диску в хранилище кластера. Только узлы, которые также подключены к этому диску, могут присоединиться к кластеру. Использовать этот режим не рекомендуется.


## <a name="windows-server-failover-clustering-on-premises"></a><a name="fdfee875-6e66-483a-a343-14bbaee33275"></a> Отказоустойчивый кластер Windows Server в локальной среде
На рис. 1 показан кластер, состоящий из двух узлов. В случае отказа сетевого подключения между узлами, если оба из них работоспособны, необходимо выяснить, какой узел должен по-прежнему предоставлять приложения и службы кластера. Узел, у которого есть доступ к диску или файловому ресурсу кворума, должен обеспечить работоспособность служб.

Так как в этом примере используется кластер из двух узлов, был выбран режим кворума "Большинство узлов и файлового ресурса". Можно также использовать большинство узлов и дисков. В рабочей среде рекомендуется использовать диск кворума. Можно применить сетевую систему хранения, чтобы обеспечить его высокую доступность.

![Рисунок 1. Пример конфигурации отказоустойчивого кластера Windows Server для SAP ASCS/SCS в Azure][sap-ha-guide-figure-1000]

_**Рис. 1.** Пример конфигурации отказоустойчивого кластера Windows Server для SAP ASCS/SCS в Azure_

### <a name="shared-storage"></a><a name="be21cf3e-fb01-402b-9955-54fbecf66592"></a> Общее хранилище
На рис. 1 также показано общее хранилище кластера с двумя узлами. В кластере с общим хранилищем в локальной среде имеется общее хранилище, видимое для всех узлов в кластере. Механизм блокировки защищает данные от повреждения. Каждый узел может обнаружить отказ другого узла. При сбое одного узла второй узел получает право на владение ресурсами хранилища и обеспечивает доступность служб.

> [!NOTE]
> В некоторых СУБД, например SQL Server, общие диски не требуются для достижения высокого уровня доступности. SQL Server AlwaysOn выполняет репликацию файлов данных и журналов СУБД с локального диска одного узла кластера на локальный диск другого узла кластера. Поэтому в конфигурации кластера Windows не нужен общий диск.
>
>

### <a name="networking-and-name-resolution"></a><a name="ff7a9a06-2bc5-4b20-860a-46cdb44669cd"></a> Сетевые подключения и разрешение имен
Клиентские компьютеры обращаются к кластеру посредством виртуального IP-адреса и имени виртуального узла, которое предоставляет DNS-сервер. Узлы находятся в локальной среде, и DNS-сервер может обрабатывать несколько IP-адресов.

В стандартной конфигурации используется от двух сетевых подключений:

* выделенное подключение к хранилищу;
* внутреннее сетевое подключение в кластере для передачи пульса;
* общедоступная сеть, используемая клиентами для подключения к кластеру.

## <a name="windows-server-failover-clustering-in-azure"></a><a name="2ddba413-a7f5-4e4e-9a51-87908879c10a"></a> Отказоустойчивый кластер Windows Server в Azure
По сравнению с развертываниями без операционной системы или развертываниями частных облаков виртуальные машины Azure требуют дополнительной настройки WSFC. При создании общего диска кластера экземпляру SAP ASCS/SCS потребуется задать несколько IP-адресов и имен виртуальных узлов.

В этой статье мы обсудим основные понятия и дополнительные действия, необходимые при создании в Azure кластера SAP Central Services с высоким уровнем доступности. Мы покажем, как настроить сторонний инструмент SIOS DataKeeper и внутренний балансировщик нагрузки Azure. С помощью этих инструментов можно создать отказоустойчивый кластер Windows с файловым ресурсом-свидетелем в Azure.

![Рис. 2. Конфигурация отказоустойчивого кластера Windows Server без общего диска в Azure][sap-ha-guide-figure-1001]

_**Рис. 2.** Конфигурация отказоустойчивого кластера Windows Server без общего диска в Azure_

### <a name="shared-disk-in-azure-with-sios-datakeeper"></a><a name="1a464091-922b-48d7-9d08-7cecf757f341"></a> Создание общего диска в Azure с помощью SIOS DataKeeper
Общее хранилище кластера необходимо для экземпляра SAP ASCS/SCS с высоким уровнем доступности. По состоянию на сентябрь 2016 г. Azure не предоставляет общее хранилище для создания кластера с общим хранилищем. Вы можете использовать стороннее ПО SIOS DataKeeper Cluster Edition, чтобы создать зеркальное хранилище, имитирующее общее хранилище кластера. Решение SIOS обеспечивает синхронную репликацию данных в реальном времени. Вот как можно создать общий дисковый ресурс для кластера.

1. Подключите дополнительный диск к каждой виртуальной машине в конфигурации кластера Windows.
2. Запустите SIOS DataKeeper Cluster Edition на обоих узлах виртуальной машины.
3. Настройте ПО SIOS DataKeeper Cluster Edition таким образом, чтобы оно зеркально отображало содержимое тома дополнительного подключенного диска исходной виртуальной машины в том дополнительного подключенного диска целевой виртуальной машины. SIOS DataKeeper абстрагирует исходные и конечные локальные тома, а затем предоставляет их отказоустойчивому кластеру Windows как один общий диск.

Дополнительные сведения о SIOS DataKeeper см. [здесь](https://us.sios.com/products/datakeeper-cluster/).

![Рис. 3. Конфигурация отказоустойчивой кластеризации Windows Server в Azure с использованием SIOS DataKeeper][sap-ha-guide-figure-1002]

_**Рис. 3.** Конфигурация отказоустойчивой кластеризации Windows Server в Azure с использованием SIOS DataKeeper_

> [!NOTE]
> В некоторых СУБД, например SQL Server, общие диски не требуются для достижения высокого уровня доступности. SQL Server AlwaysOn выполняет репликацию файлов данных и журналов СУБД с локального диска одного узла кластера на локальный диск другого узла кластера. Поэтому в конфигурации кластера Windows не нужен общий диск.
>
>

### <a name="name-resolution-in-azure"></a><a name="44641e18-a94e-431f-95ff-303ab65e0bcb"></a> Разрешение имен в Azure
Облачная платформа Azure не предоставляет возможность настройки виртуальных IP-адресов, т. е. плавающих IP-адресов. Для настройки виртуального IP-адреса, используемого для доступа к кластерным ресурсам в облаке, вам потребуется альтернативное решение.
Azure предоставляет внутренний балансировщик нагрузки в службе балансировки нагрузки Azure. С его помощью клиенты могут обращаться к кластеру через виртуальный IP-адрес кластера.
Внутренний балансировщик нагрузки необходимо развернуть в группе ресурсов, содержащей узлы кластера. Затем нужно настроить все необходимые правила перенаправления портов для портов проб внутреннего балансировщика нагрузки.
Клиенты могут подключаться через имя виртуального узла. DNS-сервер разрешает IP-адрес кластера, и внутренний балансировщик нагрузки выполняет перенаправление на активный узел кластера.

## <a name="sap-netweaver-high-availability-in-azure-infrastructure-as-a-service-iaas"></a><a name="2e3fec50-241e-441b-8708-0b1864f66dfa"></a> Высокий уровень доступности SAP NetWeaver в Azure IaaS
Чтобы обеспечить высокий уровень доступности приложения SAP, т. е. компонентов ПО SAP, необходимо защитить следующие компоненты:

* экземпляр сервера приложений SAP;
* экземпляров ASCS/SCS SAP,
* сервер СУБД.

Дополнительные сведения о защите компонентов SAP в сценариях обеспечения высокого уровня доступности см. в статье [SAP NetWeaver на виртуальных машинах Windows. Руководство по планированию и внедрению][planning-guide-11].

### <a name="high-availability-sap-application-server"></a><a name="93faa747-907e-440a-b00a-1ae0a89b1c0e"></a> Высокая доступность серверов приложений SAP
Как правило, для экземпляров серверов приложений или диалоговых экземпляров SAP специальное решение для обеспечения высокой доступности не требуется. Высокий уровень доступности достигается за счет избыточности, то есть на разных экземплярах виртуальных машин Azure настраивается несколько диалоговых экземпляров. Необходимо установить как минимум два экземпляра приложения SAP на два экземпляра виртуальных машин Azure.

![Рис. 4. Сервер приложений SAP с высоким уровнем доступности][sap-ha-guide-figure-2000]

_**Рис. 4.** Сервер приложений SAP с высоким уровнем доступности_

Все виртуальные машины, на которых размещены экземпляры серверов приложений SAP, должны находиться в одной группе доступности Azure. Группа доступности Azure гарантирует следующее.

* Все виртуальные машины входят в одни и те же домены обновления. Домен обновления, например, позволяет избежать одновременного обновления виртуальных машин во время планового обслуживания.
* Все виртуальные машины входят в одни и те же домены сбоя. Домен сбоя, например, позволяет избежать развертывания виртуальных машин таким образом, что единая точка отказа сможет влиять на доступность всех виртуальных машин.

Дополнительные сведения о том, как [управлять доступностью виртуальных машин] [.. /manage-availability.md].

Только для неуправляемого диска: так как учетная запись хранения Azure является потенциальной единственной точкой отказа, важно иметь по меньшей мере две учетные записи хранения Azure, в которых будут распределены как минимум две виртуальные машины. В идеале все диски виртуальных машин, на которых выполняются диалоговые экземпляры SAP, следует развертывать в разных учетных записях хранения.

### <a name="high-availability-sap-ascsscs-instance"></a><a name="f559c285-ee68-4eec-add1-f60fe7b978db"></a> Высокая доступность экземпляра SAP ASCS/SCS
На рис. 5 приведен пример высокой доступности экземпляра SAP ASCS/SCS

![Рис. 5. Высокодоступный экземпляр SAP ASCS/SCS][sap-ha-guide-figure-2001]

_**Рис. 5.** Высокодоступный экземпляр SAP ASCS/SCS_

#### <a name="sap-ascsscs-instance-high-availability-with-windows-server-failover-clustering-in-azure"></a><a name="b5b1fd0b-1db4-4d49-9162-de07a0132a51"></a> Обеспечение высокого уровня доступности экземпляра SAP ASCS/SCS с помощью отказоустойчивой кластеризации Windows Server в Azure
По сравнению с развертываниями без операционной системы или развертываниями частных облаков виртуальные машины Azure требуют дополнительной настройки WSFC. Чтобы создать отказоустойчивый кластер Windows для кластеризации экземпляра SAP ASCS/SCS, требуется общий диск кластера, несколько IP-адресов и имен виртуальных узлов, а также внутренний балансировщик нагрузки Azure. Этот вопрос рассматривается более подробно далее в этом документе.

![Рисунок 6. Конфигурация отказоустойчивого кластера Windows Server для SAP ASCS/SCS в Azure с использованием SIOS DataKeeper][sap-ha-guide-figure-1002]

_**Рисунок 6.** Конфигурация отказоустойчивого кластера Windows Server для SAP ASCS/SCS в Azure с использованием SIOS DataKeeper_

### <a name="high-availability-dbms-instance"></a><a name="ddd878a0-9c2f-4b8e-8968-26ce60be1027"></a> Высокий уровень доступности экземпляра СУБД
СУБД также является единой точкой отказа системы SAP. Она должна быть защищена с помощью решения для обеспечения высокого уровня доступности. На рис. 7 показан пример решения для обеспечения высокого уровня доступности SQL Server AlwaysOn в Azure, в котором используется отказоустойчивый кластер Windows Server и внутренний балансировщик нагрузки. SQL Server AlwaysOn реплицирует файлы данных и журналов СУБД с помощью собственной функции репликации СУБД. Поэтому вам не требуются общие диски кластера, что упрощает всю конфигурацию.

![Рисунок 7. Пример высокодоступного сервера СУБД SAP с SQL Server Always On][sap-ha-guide-figure-2003]

_**Рисунок 7.** Пример высокодоступного сервера СУБД SAP с SQL Server Always On_

Дополнительные сведения о кластеризации SQL Server в Azure посредством модели развертывания с помощью Azure Resource Manager можно найти в следующих статьях:

* [Настройка группы доступности AlwaysOn на виртуальных машинах Azure с использованием Resource Manager][virtual-machines-windows-portal-sql-alwayson-availability-groups-manual]
* [Настройка внутреннего балансировщика нагрузки для группы доступности AlwaysOn в Azure][virtual-machines-windows-portal-sql-alwayson-int-listener]

## <a name="end-to-end-high-availability-deployment-scenarios"></a><a name="045252ed-0277-4fc8-8f46-c5a29694a816"></a> Сценарии комплексного развертывания с высоким уровнем доступности

### <a name="deployment-scenario-using-architectural-template-1"></a>Сценарий развертывания с помощью шаблона 1 архитектуры

На рис. 8 показан пример архитектуры SAP NetWeaver с высоким уровнем доступности в Azure для **одной** системы SAP. В этом сценарии используется следующая конфигурация:

- один выделенный кластер для экземпляра SAP ASCS/SCS;
- один выделенный кластер для экземпляра СУБД;
- экземпляры сервера приложений SAP развернуты на собственных выделенных виртуальных машинах.

![Рисунок 8. Шаблон 1 архитектуры SAP с высоким уровнем доступности с выделенным кластером для ASCS/SCS и СУБД][sap-ha-guide-figure-2004]

_**Рисунок 8.** Шаблон 1 архитектуры SAP с высоким уровнем доступности, выделенным кластером для ASCS/SCS и СУБД_

### <a name="deployment-scenario-using-architectural-template-2"></a>Сценарий развертывания с помощью шаблона 2 архитектуры

На рис. 9 показан пример архитектуры SAP NetWeaver с высоким уровнем доступности в Azure для **одной** системы SAP. В этом сценарии используется следующая конфигурация:

- **один** выделенный кластер для экземпляра SAP ASCS/SCS и СУБД;
- экземпляры сервера приложений SAP развернуты на собственных выделенных виртуальных машинах.

![Рисунок 9. Шаблон 2 архитектуры SAP с высоким уровнем доступности с выделенным кластером для ASCS/SCS и СУБД][sap-ha-guide-figure-2005]

_**Рисунок 9.** Шаблон 2 архитектуры SAP с высоким уровнем доступности с выделенным кластером для ASCS/SCS и СУБД_

### <a name="deployment-scenario-using-architectural-template-3"></a>Сценарий развертывания с помощью шаблона 3 архитектуры

На рис. 10 показан пример архитектуры SAP NetWeaver с высоким уровнем доступности в Azure для **двух** систем SAP с &lt;SID1&gt; и &lt;SID2&gt;. В этом сценарии используется следующая конфигурация:

- один выделенный кластер для **обоих** экземпляров SID1 *и* SID2 SAP ASCS/SCS (один кластер);
- один выделенный кластер для SID1 СУБД и другой выделенный кластер для SID2 СУБД (два кластера);
- экземпляры сервера приложений SAP с SID1 системы SAP размещаются на собственных выделенных виртуальных машинах;
- экземпляры сервера приложений SAP с SID2 системы SAP размещаются на собственных выделенных виртуальных машинах.

![Рисунок 10. Шаблон 3 архитектуры SAP с высоким уровнем доступности с выделенным кластером для разных экземпляров ASCS/SCS][sap-ha-guide-figure-6003]

_**Рисунок 10.** Шаблон 3 архитектуры SAP с высоким уровнем доступности с выделенным кластером для разных экземпляров ASCS/SCS_

## <a name="prepare-the-infrastructure"></a><a name="78092dbe-165b-454c-92f5-4972bdbef9bf"></a> Подготовка инфраструктуры

### <a name="prepare-the-infrastructure-for-architectural-template-1"></a>Подготовка инфраструктуры для шаблона 1 архитектуры
Шаблоны Azure Resource Manager для SAP помогают упростить развертывание необходимых ресурсов.

Эти трехуровневые шаблоны в Azure Resource Manager также поддерживают сценарии обеспечения высокого уровня доступности, например шаблон 1, содержащий два кластера. Каждый кластер является единой точкой отказа SAP для SAP ASCS/SCS и СУБД.

Шаблоны Azure Resource Manager, описанные в этой статье, доступны здесь:

* [Образ Azure Marketplace](https://github.com/Azure/azure-quickstart-templates/)  
* [Образ Azure Marketplace, использующий управляемые диски](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md)  
* [Пользовательский образ](https://github.com/Azure/azure-quickstart-templates/)
* [Пользовательский образ, использующий управляемые диски](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-md)

Чтобы подготовить инфраструктуру для использования шаблона 1 архитектуры, сделайте следующее:

- На портале Azure в колонке **Параметры** в поле **SYSTEMAVAILABILITY** выберите **HA**.

  ![Рисунок 11. Настройка параметров Azure Resource Manager для SAP с высоким уровнем доступности][sap-ha-guide-figure-3000]

_**Рисунок 11.** Настройка параметров Azure Resource Manager для SAP с высоким уровнем доступности_


  Этот шаблон создаст:

  * **Виртуальные машины**:
    * виртуальные машины сервера приложений SAP: <*SAPSystemSID*>-di-<*Number*>;
    * виртуальные машины кластера SCS/SCS <*SAPSystemSID*>-ascs-<*Number*>;
    * виртуальные машины кластера СУБД: <*SID_системы_SAP*>-db-<*номер*>.

  * **Сетевые карты для всех виртуальных машин с соответствующими IP-адресами**:
    * <*SAPSystemSID*>-nic-di-<*Number*>;
    * <*SAPSystemSID*>-nic-ascs-<*Number*>;
    * <*SAPSystemSID*>-nic-db-<*Number*>.

  * **Учетные записи хранения (только неуправляемые диски)**

  * **Группы доступности** для:
    * виртуальных машин сервера приложений SAP: <*SAPSystemSID*>-avset-di;
    * виртуальных машин кластера SCS/SCS: <*SAPSystemSID*>-avset-ascs;
    * виртуальных машин кластера СУБД: <*SAPSystemSID*>-avset-db.

  * **Внутренний балансировщик нагрузки Azure**:
    * со всеми портами для экземпляра ASCS/SCS и IP-адресом: <*SAPSystemSID*>-lb-ascs;
    * со всеми портами для СУБД SQL Server и IP-адресом: <*SAPSystemSID*>-lb-db.

  * **Группа безопасности сети**: <*SAPSystemSID*>-nsg-ascs-0;  
    * с открытым внешним портом RDP для виртуальной машины: <*SAPSystemSID*>-ascs-0.

> [!NOTE]
> Все IP-адреса сетевых карт и внутренних балансировщиков нагрузки Azure изначально создаются как **динамические**. Их необходимо изменить на **статические** IP-адреса. Процедура будет описана далее.
>
>

### <a name="deploy-virtual-machines-with-corporate-network-connectivity-cross-premises-to-use-in-production"></a><a name="c87a8d3f-b1dc-4d2f-b23c-da4b72977489"></a> Развертывание виртуальных машин с подключением к корпоративной сети (распределенное развертывание) для использования в рабочей среде
Для рабочих систем SAP виртуальные машины Azure развертываются с подключением к корпоративной сети с помощью VPN типа "сеть — сеть" Azure или канала Azure ExpressRoute.

> [!NOTE]
> Можно использовать имеющийся экземпляр виртуальной сети Azure. Виртуальная сеть и подсеть уже созданы и подготовлены.
>
>

1. На портале Azure в колонке **Параметры** в поле **NEWOREXISTINGSUBNET** выберите **existing**.
2. В поле **SUBNETID** введите полную строку SubnetID подготовленной сети Azure, в которой планируется развертывание виртуальных машин Azure.
3. Чтобы получить список всех подсетей в сети Azure, выполните эту команду PowerShell:

   ```powershell
   (Get-AzVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets
   ```

   В поле **ID** отображается **SUBNETID**.
4. Чтобы получить список всех значений **SUBNETID**, выполните эту команду PowerShell:

   ```powershell
   (Get-AzVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets.Id
   ```

   **SUBNETID** выглядит следующим образом:

   ```
   /subscriptions/<SubscriptionId>/resourceGroups/<VPNName>/providers/Microsoft.Network/virtualNetworks/azureVnet/subnets/<SubnetName>
   ```

### <a name="deploy-cloud-only-sap-instances-for-test-and-demo"></a><a name="7fe9af0e-3cce-495b-a5ec-dcb4d8e0a310"></a> Развертывание облачных экземпляров SAP для тестирования и демонстрации
Систему SAP с высоким уровнем доступности можно развернуть в модели полностью облачного развертывания. Этот тип развертывания больше подходит для демонстрации и тестирования использования. Он не поддерживается для использования в рабочей среде.

- На портале Azure в колонке **Параметры** в поле **NEWOREXISTINGSUBNET** выберите **new**. Оставьте поле **SUBNETID** пустым.

  Виртуальная сеть и подсеть Azure создадутся автоматически с помощью шаблона Azure Resource Manager для SAP.

> [!NOTE]
> Кроме того, необходимо развернуть по меньшей мере одну выделенную виртуальную машину для служб AD и DNS в этой же виртуальной сети. Эти виртуальные машины не создаются с помощью шаблона.
>
>


### <a name="prepare-the-infrastructure-for-architectural-template-2"></a>Подготовка инфраструктуры для шаблона 2 архитектуры

Эти шаблоны Azure Resource Manager для SAP можно использовать для упрощения развертывания ресурсов инфраструктуры, необходимых для шаблона 2 архитектуры SAP.

Шаблоны Azure Resource Manager для сценария развертывания доступны здесь:

* [Образ Azure Marketplace](https://github.com/Azure/azure-quickstart-templates/)  
* [Образ Azure Marketplace, использующий управляемые диски](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-converged-md)  
* [Пользовательский образ](https://github.com/Azure/azure-quickstart-templates/)
* [Пользовательский образ, использующий управляемые диски](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-converged-md)


### <a name="prepare-the-infrastructure-for-architectural-template-3"></a>Подготовка инфраструктуры для шаблона 3 архитектуры

Вы можете подготовить инфраструктуру и настроить SAP для поддержки сценария с **несколькими идентификаторами безопасности**. Например, вы можете добавить дополнительный экземпляр SAP ASCS/SCS в *имеющуюся* конфигурацию кластера. Дополнительные сведения см. в статье [Create an SAP NetWeaver multi-SID configuration][sap-ha-multi-sid-guide] (Создание конфигурации SAP NetWeaver с несколькими идентификаторами безопасности).

Если вы хотите создать кластер с несколькими идентификаторами безопасности, можно воспользоваться [шаблонами быстрого запуска в GitHub](https://github.com/Azure/azure-quickstart-templates).
Чтобы создать кластер с несколькими идентификаторами безопасности, разверните следующие три шаблона:

* [шаблон ASCS/SCS](#ASCS-SCS-template);
* [шаблон базы данных](#database-template);
* [шаблон серверов приложений](#application-servers-template).

В следующих разделах приведены более подробные сведения о шаблонах и параметрах, которые нужно указать в них.

#### <a name="ascsscs-template"></a><a name="ASCS-SCS-template"></a> Шаблон ASCS/SCS

Шаблон ASCS/SCS развертывает две виртуальные машины, используемые для создания отказоустойчивого кластера Windows Server, содержащего несколько экземпляров ASCS/SCS.

Чтобы настроить [шаблон ASCS/SCS с несколькими ИД безопасности][sap-templates-3-tier-multisid-xscs-marketplace-image] или [шаблон ASCS/SCS с несколькими ИД безопасности, использующий управляемые диски][sap-templates-3-tier-multisid-xscs-marketplace-image-md], введите в нем следующие значения параметров:

  - **Префикс ресурса.**  Префикс ресурса используется в качестве префикса для всех ресурсов, созданных во время развертывания. Так как ресурсы принадлежат к нескольким системам SAP, префикс ресурса не является идентификатором безопасности одной системы SAP.  Длина префикса должна составлять **от 3 до 6 символов**.
  - **Тип стека.** Выберите тип стека системы SAP. В зависимости от типа стека в службе Azure Load Balancer предоставляется один (только ABAP или Java) или два (ABAP и Java) частных IP-адреса на систему SAP.
  -  **Тип ОС.** Выберите операционную систему виртуальных машин.
  -  **Количество систем SAP.** Выберите количество систем SAP, которое требуется установить в этом кластере.
  -  **Доступность системы.** Выберите **высокую доступность**.
  -  **Имя пользователя и пароль администратора.** Создайте пользователя, которого можно использовать для входа на компьютер.
  -  **Новая или имеющаяся подсеть.** Определите, следует ли создать виртуальную сеть и подсеть или использовать имеющуюся подсеть. При наличии виртуальной сети, подключенной к локальной сети, выберите **ее**.
  -  **Идентификатор подсети.** Чтобы развернуть виртуальную машину в имеющейся виртуальной сети с определенной подсетью, необходимо указать идентификатор этой определенной подсети. Идентификатор обычно выглядит следующим образом: /subscriptions/<*идентификатор_подписки*>/resourceGroups/<*имя_группы_ресурсов*>/providers/Microsoft.Network/virtualNetworks/<*имя_виртуальной_сети*>/subnets/<*имя_подсети*>

Шаблон развертывает один экземпляр Azure Load Balancer, который поддерживает несколько систем SAP:

- экземпляры ASCS настроены для номеров экземпляров 00, 10, 20...;
- экземпляры ASCS настроены для номеров экземпляров 01, 11, 21...;
- экземпляры ASCS ERS (только Linux) настроены для номеров экземпляров 02, 12, 22...;
- экземпляры SCS ERS (только Linux) настроены для номеров экземпляров 03, 13, 23...

Балансировщик нагрузки содержит 1 виртуальный IP-адрес (2 для Linux): 1 виртуальный IP-адрес для ASCS/SCS и 1 один для ERS (только Linux).

В следующем списке содержатся все правила балансировки нагрузки (где x — номер системы SAP, например 1, 2, 3 и т. д.):
- порты для любой системы SAP в среде Windows: 445, 5985;
- порты ASCS (номер экземпляра x0): 32x0, 36x0, 39x0, 81x0, 5x013, 5x014, 5x016;
- порты SCS (номер экземпляра x1): 32x1, 33x1, 39x1, 81x1, 5x113, 5x114, 5x116;
- порты ASCS ERS в Linux (номер экземпляра x2): 33x2, 5x213, 5x214, 5x216;
- порты ASCS ERS в Linux (номер экземпляра x3): 33x3, 5x313, 5x314, 5x316.

Балансировщик нагрузки настроен для использования следующих портов пробы (где x — номер системы SAP, например 1, 2, 3 и т. д.):
- порт пробы внутренней подсистемы балансировки нагрузки ASCS/SCS — 620x0;
- порт пробы внутренней подсистемы балансировки нагрузки ERS (только Linux) — 621x2.

#### <a name="database-template"></a><a name="database-template"></a> Шаблон базы данных

Шаблон базы данных развертывает одну или две виртуальные машины, используемые для установки системы управления реляционной базой данных (реляционной СУБД) для одной системы SAP. Например, при развертывании шаблона ASCS/SCS для 5 систем SAP его необходимо развернуть пять раз.

Чтобы настроить [шаблон базы данных с несколькими ИД безопасности][sap-templates-3-tier-multisid-db-marketplace-image] или [шаблон базы данных с несколькими ИД безопасности, использующий управляемые диски][sap-templates-3-tier-multisid-db-marketplace-image-md], введите в нем следующие значения параметров:

- **Идентификатор системы SAP.** Введите идентификатор системы SAP, которую требуется установить. Идентификатор будет использоваться в качестве префикса для развертываемых ресурсов.
- **Тип ОС.** Выберите операционную систему виртуальных машин.
- **Тип базы данных.** Выберите тип базы данных, которую необходимо установить в кластере. Если вы хотите установить Microsoft SQL Server, выберите **SQL**. Чтобы установить SAP HANA на виртуальных машинах, выберите **HANA**. Выберите соответствующий тип операционной системы: **Windows** для SQL и Linux для HANA. Балансировщик нагрузки Azure, подключенный к виртуальным машинам, будет настроен для поддержки выбранного типа базы данных:
  * **SQL.** Балансировщик нагрузки будет выполнять балансировку нагрузки порта 1433. Используйте этот порт для настройки SQL Server AlwaysOn.
  * **HANA.** Балансировщик нагрузки будет выполнять балансировку нагрузки порта 35015 и 35017. Установите SAP HANA с номером экземпляра **50**.
  Балансировщик нагрузки будет использовать порт пробы 62550.
- **Размер системы SAP.** Задайте количество систем SAP, которые будет содержать новая система. Если вы не знаете, сколько систем SAP потребуется для системы, обратитесь к партнеру по технологиям SAP или к системному интегратору SAP.
- **Доступность системы.** Выберите **высокую доступность**.
- **Имя пользователя и пароль администратора.** Создайте пользователя, которого можно использовать для входа на компьютер.
- **Идентификатор подсети.** Введите идентификатор подсети, используемый во время развертывания шаблона ASCS/SCS, или идентификатор подсети, созданный в процессе развертывания шаблона ASCS/SCS.

#### <a name="application-servers-template"></a><a name="application-servers-template"></a> Шаблон серверов приложений

Этот шаблон серверов приложений развертывает две или более виртуальные машины, используемые в качестве экземпляров сервера приложений SAP для одной системы SAP. Например, при развертывании шаблона ASCS/SCS для 5 систем SAP его необходимо развернуть пять раз.

Чтобы настроить [шаблон серверов приложений с несколькими ИД безопасности][sap-templates-3-tier-multisid-apps-marketplace-image] или [шаблон серверов приложений с несколькими ИД безопасности, использующий управляемые диски][sap-templates-3-tier-multisid-apps-marketplace-image-md], введите в нем следующие значения параметров:

  -  **Идентификатор системы SAP.** Введите идентификатор системы SAP, которую требуется установить. Идентификатор будет использоваться в качестве префикса для развертываемых ресурсов.
  -  **Тип ОС.** Выберите операционную систему виртуальных машин.
  -  **Размер системы SAP.** Количество систем SAP, которые будет содержать новая система. Если вы не знаете, сколько систем SAP потребуется для системы, обратитесь к партнеру по технологиям SAP или к системному интегратору SAP.
  -  **Доступность системы.** Выберите **высокую доступность**.
  -  **Имя пользователя и пароль администратора.** Создайте пользователя, которого можно использовать для входа на компьютер.
  -  **Идентификатор подсети.** Введите идентификатор подсети, используемый во время развертывания шаблона ASCS/SCS, или идентификатор подсети, созданный в процессе развертывания шаблона ASCS/SCS.


### <a name="azure-virtual-network"></a><a name="47d5300a-a830-41d4-83dd-1a0d1ffdbe6a"></a> Виртуальная сеть Azure
В нашем примере адресное пространство виртуальной сети Azure — 10.0.0.0/16. Существует одна подсеть **Subnet** с диапазоном адресов 10.0.0.0/24. Все виртуальные машины и внутренние балансировщики нагрузки развертываются в этой виртуальной сети.

> [!IMPORTANT]
> Не изменяйте параметры сети в гостевой ОС. Например, IP-адреса, DNS-серверы и подсети. Все параметры сети настраиваются с помощью Azure. Они распространяются через службу DHCP.
>
>

### <a name="dns-ip-addresses"></a><a name="b22d7b3b-4343-40ff-a319-097e13f62f9e"></a> IP-адреса DNS

Чтобы задать необходимые IP-адреса DNS, выполните следующие действия.

1. На портале Azure в колонке **DNS-серверы** для параметра **DNS-серверы** виртуальной сети задайте значение **Пользовательская служба DNS**.
2. Выберите параметры в зависимости от типа сети. Для получения дополнительных сведений см. следующие ресурсы:
   * Добавьте IP-адреса локальных DNS-серверов.  
   Локальные DNS-серверы могут быть расширены до виртуальных машин в Azure. В этом случае можно добавить IP-адреса этих виртуальных машин Azure, на которых запущена служба DNS.
   * Для развертываний виртуальных машин, изолированных в Azure: разверните дополнительную виртуальную машину в том же экземпляре виртуальной сети, который служит DNS-сервером. Добавьте IP-адреса этих виртуальных машин Azure, настроенных для запуска службы DNS.

   ![Рисунок 12. Настройка DNS-серверов для виртуальной сети Azure][sap-ha-guide-figure-3001]

   _**Рисунок 12.** Настройка DNS-серверов для виртуальной сети Azure_

   > [!NOTE]
   > В случае изменения IP-адресов DNS-серверов необходимо перезагрузить виртуальные машины Azure, чтобы применить изменения и распространить новые адреса DNS-серверов.
   >
   >

В нашем примере служба DNS установлена и настроена на следующих виртуальных машинах Windows.

| Роль виртуальной машины | Имя узла виртуальной машины | Имя сетевой карты | Статический IP-адрес |
| --- | --- | --- | --- |
| 1-й DNS-сервер |domcontr-0 |pr1-nic-domcontr-0 |10.0.0.10 |
| 2-й DNS-сервер |domcontr-1 |pr1-nic-domcontr-1 |10.0.0.11 |

### <a name="host-names-and-static-ip-addresses-for-the-sap-ascsscs-clustered-instance-and-dbms-clustered-instance"></a><a name="9fbd43c0-5850-4965-9726-2a921d85d73f"></a> Имена узлов и статические IP-адреса для кластеризованного экземпляра SAP ASCS/SCS и кластеризованного экземпляра СУБД

Для локального развертывания необходимы следующие зарезервированные имена узлов и IP-адреса.

| Роль имени виртуального узла | Имя виртуального узла | Виртуальный статический IP-адрес |
| --- | --- | --- |
| Имя 1-го виртуального узла кластера SAP ASCS/SCS (используется для управления кластером) |pr1-ascs-vir |10.0.0.42 |
| Имя виртуального узла экземпляра SAP ASCS/SCS |pr1-ascs-sap |10.0.0.43 |
| Имя 2-го виртуального узла кластера SAP ASCS/SCS (используется для управления кластером) |pr1-dbms-vir |10.0.0.32 |

Имена виртуальных узлов **pr1-ascs-vir** и **pr1-dbms-vir** и связанные IP-адреса, используемые для управления кластером, создаются вместе с кластером. Дополнительные сведения о том, как это сделать, см. в разделе [Сбор узлов кластера в конфигурации кластера][sap-ha-guide-8.12.1].

Два других имени виртуальных узлов — **pr1-ascs-sap** и **pr1-dbms-sap** — и связанные IP-адреса можно вручную создать на DNS-сервере. Эти ресурсы используются кластеризованным экземпляром SAP ASCS/SCS и кластеризованным экземпляром СУБД. Дополнительные сведения о том, как это сделать, см. в разделе [Создание имени виртуального узла для кластеризованного экземпляра SAP ASCS/SCS][sap-ha-guide-9.1.1].

### <a name="set-static-ip-addresses-for-the-sap-virtual-machines"></a><a name="84c019fe-8c58-4dac-9e54-173efd4b2c30"></a> Настройка статических IP-адресов для виртуальных машин SAP
После развертывания виртуальных машин для использования в кластере необходимо настроить статические IP-адреса для всех виртуальных машин. Это невозможно сделать в операционной системе на виртуальной машине. Задача выполняется в конфигурации виртуальной сети Azure.

1. На портале Azure выберите **Группа ресурсов** > **Сетевая карта** > **Параметры** > **IP-адрес**.
2. В колонке **IP-адреса** в разделе **Назначение** выберите **Статический**. В поле **IP-адрес** введите нужный IP-адрес.

   > [!NOTE]
   > В случае изменения IP-адреса сетевой карты необходимо перезапустить виртуальные машины Azure, чтобы применить изменение.  
   >
   >

   ![Рисунок 13. Настройка статического IP-адреса для сетевой карты каждой виртуальной машины][sap-ha-guide-figure-3002]

   _**Рисунок 13.** Настройка статического IP-адреса для сетевой карты каждой виртуальной машины_

   Повторите этот шаг для всех сетевых интерфейсов, т. е. для всех виртуальных машин, включая те, которые вы хотите использовать для служб Active Directory и DNS.

В нашем примере используются следующие виртуальные машины и статические IP-адреса.

| Роль виртуальной машины | Имя узла виртуальной машины | Имя сетевой карты | Статический IP-адрес |
| --- | --- | --- | --- |
| Первый экземпляр сервера приложений SAP |pr1-di-0 |pr1-nic-di-0 |10.0.0.50 |
| Второй экземпляр сервера приложений SAP |pr1-di-1 |pr1-nic-di-1 |10.0.0.51 |
| ... |... |... |... |
| Последний экземпляр сервера приложений SAP |pr1-di-5 |pr1-nic-di-5 |10.0.0.55 |
| 1-й узел кластера для экземпляра ASCS/SCS |pr1-ascs-0 |pr1-nic-ascs-0 |10.0.0.40 |
| 2-й узел кластера для экземпляра ASCS/SCS |pr1-ascs-1 |pr1-nic-ascs-1 |10.0.0.41 |
| 1-й узел кластера для экземпляра СУБД |pr1-db-0 |pr1-nic-db-0 |10.0.0.30 |
| 2-й узел кластера для экземпляра СУБД |pr1-db-1 |pr1-nic-db-1 |10.0.0.31 |

### <a name="set-a-static-ip-address-for-the-azure-internal-load-balancer"></a><a name="7a8f3e9b-0624-4051-9e41-b73fff816a9e"></a> Настройка статического IP-адреса для внутреннего балансировщика нагрузки Azure

Шаблон Azure Resource Manager для SAP создает внутренний балансировщик нагрузки Azure, используемый для кластера экземпляров SAP ASCS/SCS и кластера СУБД.

> [!IMPORTANT]
> IP-адрес имени виртуального узла для SAP ASCS/SCS совпадает с IP-адресом внутреннего балансировщика нагрузки SAP ASCS/SCS: **pr1-lb-ascs**.
> IP-адрес виртуального имени СУБД совпадает с IP-адресом внутреннего балансировщика нагрузки СУБД: **pr1-lb-dbms**.
>
>

Чтобы настроить статические IP-адреса для внутреннего балансировщика нагрузки Azure, выполните следующие действия:

1. При первоначальном развертывании для внутреннего балансировщика нагрузки задается **динамический** IP-адрес. На портале Azure в колонке **IP-адреса** в разделе **Назначение** выберите **Статический**.
2. Задайте для внутреннего балансировщика нагрузки **pr1-lb-ascs** IP-адрес имени виртуального узла экземпляра SAP ASCS/SCS.
3. Задайте для внутреннего балансировщика нагрузки **pr1-lb-dbms** IP-адрес имени виртуального узла экземпляра СУБД.

   ![Рисунок 14. Настройка статических IP-адресов внутренней подсистемы балансировки нагрузки для экземпляра SAP ASCS/SCS][sap-ha-guide-figure-3003]

   _**Рисунок 14.** Настройка статических IP-адресов внутренней подсистемы балансировки нагрузки для экземпляра SAP ASCS/SCS_

В нашем примере есть два внутренних балансировщика нагрузки Azure со следующими статическими IP-адресами.

| Роль внутреннего балансировщика нагрузки Azure | Имя внутреннего балансировщика нагрузки Azure | Статический IP-адрес |
| --- | --- | --- |
| Внутренний балансировщик нагрузки экземпляра SAP ASCS/SCS |pr1-lb-ascs |10.0.0.43 |
| Внутренний балансировщик нагрузки экземпляра СУБД SAP |pr1-lb-dbms |10.0.0.33 |


### <a name="default-ascsscs-load-balancing-rules-for-the-azure-internal-load-balancer"></a><a name="f19bd997-154d-4583-a46e-7f5a69d0153c"></a> Правила балансировки нагрузки ASCS/SCS по умолчанию для внутреннего балансировщика нагрузки Azure

По умолчанию шаблон Azure Resource Manager для SAP создает все необходимые порты для следующих экземпляров:
* экземпляра ABAP ASCS с номером экземпляра по умолчанию **00**;
* экземпляра Java SCS с номером экземпляра по умолчанию **01**.

При установке экземпляра SAP ASCS/SCS необходимо использовать номер экземпляра по умолчанию **00** для экземпляра ABAP ASCS и номер экземпляра по умолчанию **01** для экземпляра Java SCS.

Затем создайте необходимые конечные точки внутренней балансировки нагрузки для портов SAP NetWeaver.

Чтобы создать необходимые конечные точки внутренней балансировки нагрузки, сначала их необходимо создать для портов SAP NetWeaver ABAP ASCS.

| Служба, имя правила балансировки нагрузки | Номера портов по умолчанию | Определенные порты для (экземпляра ASCS с номером экземпляра 00) (ERS с номером 10) |
| --- | --- | --- |
| Сервер постановки в очередь, *lbrule3200* |32<*InstanceNumber*> |3200 |
| Сервер сообщений ABAP, *lbrule3600* |36<*InstanceNumber*> |3600 |
| Внутренняя служба сообщений ABAP, *lbrule3900* |39<*InstanceNumber*> |3900 |
| Сервер сообщений (HTTP), *Lbrule8100* |81<*InstanceNumber*> |8100 |
| Служба запуска SAP ASCS (HTTP), *Lbrule50013* |5<*InstanceNumber*>13 |50013 |
| Служба запуска SAP ASCS (HTTPS), *Lbrule50014* |5<*InstanceNumber*>14 |50014 |
| Служба постановки в очередь для репликации, *Lbrule50016* |5<*InstanceNumber*>16 |50016 |
| Служба запуска SAP ERS (HTTP), *Lbrule51013* |5<*InstanceNumber*>13 |51013 |
| Служба запуска SAP ERS (HTTPS), *Lbrule51014* |5<*InstanceNumber*>14 |51014 |
| Win RM, *Lbrule5985* | |5985 |
| Общий доступ к файлам, *Lbrule445* | |445 |

_**Таблица 1.** Номера портов экземпляров SAP NetWeaver ABAP ASCS_

Затем создайте эти конечные точки балансировки нагрузки для портов SAP NetWeaver Java SCS.

| Служба, имя правила балансировки нагрузки | Номера портов по умолчанию | Определенные порты для (экземпляра SCS с номером экземпляра 01) (ERS с номером 11) |
| --- | --- | --- |
| Сервер постановки в очередь, *lbrule3201* |32<*InstanceNumber*> |3201 |
| Сервер шлюза, *lbrule3301* |33<*InstanceNumber*> |3301 |
| Сервер сообщений Java, *lbrule3900* |39<*InstanceNumber*> |3901 |
| Сервер сообщений (HTTP), *Lbrule8101* |81<*InstanceNumber*> |8101 |
| Служба запуска SAP SCS (HTTP), *Lbrule50113* |5<*InstanceNumber*>13 |50113 |
| Служба запуска SAP SCS (HTTPS), *Lbrule50114* |5<*InstanceNumber*>14 |50114 |
| Служба постановки в очередь для репликации, *Lbrule50116* |5<*InstanceNumber*>16 |50116 |
| Служба запуска SAP ERS (HTTP), *Lbrule51113* |5<*InstanceNumber*>13 |51113 |
| Служба запуска SAP ERS (HTTPS), *Lbrule51114* |5<*InstanceNumber*>14 |51114 |
| Win RM, *Lbrule5985* | |5985 |
| Общий доступ к файлам, *Lbrule445* | |445 |

_**Таблица 2.** Номера портов экземпляров SAP NetWeaver Java SCS_

![Рисунок 15. Правила балансировки нагрузки ASCS/SCS по умолчанию для внутренней подсистемы балансировки нагрузки Azure][sap-ha-guide-figure-3004]

_**Рисунок 15.** Правила балансировки нагрузки ASCS/SCS по умолчанию для внутренней подсистемы балансировки нагрузки Azure_

Задайте для балансировщика нагрузки **pr1-lb-dbms** IP-адрес имени виртуального узла экземпляра СУБД.

### <a name="change-the-ascsscs-default-load-balancing-rules-for-the-azure-internal-load-balancer"></a><a name="fe0bd8b5-2b43-45e3-8295-80bee5415716"></a> Изменение правил балансировки нагрузки ASCS/SCS по умолчанию для внутреннего балансировщика нагрузки Azure

Если вы хотите использовать другие номера для экземпляров SAP ASCS или SCS, необходимо изменить имена и значения по умолчанию их портов.

1. На портале Azure выберите **<*SID*>-балансировщик нагрузки -lb-ascs** > **Правила балансировки нагрузки**.
2. Для всех правил балансировки нагрузки, относящихся к экземпляру SAP ASCS или SCS, измените следующие значения.

   * Имя
   * Порт
   * Внутренний порт

   Например, если требуется изменить номер экземпляра ASCS по умолчанию с 00 на 31, то необходимо внести изменения для всех портов, указанных в таблице 1.

   Ниже приведен пример обновления *lbrule3200* для порта.

   ![Рисунок 16. Изменение правил балансировки нагрузки ASCS/SCS по умолчанию для внутренней подсистемы балансировки нагрузки Azure.][sap-ha-guide-figure-3005]

   _**Рисунок 16.** Изменение правил балансировки нагрузки ASCS/SCS по умолчанию для внутренней подсистемы балансировки нагрузки Azure_

### <a name="add-windows-virtual-machines-to-the-domain"></a><a name="e69e9a34-4601-47a3-a41c-d2e11c626c0c"></a> Присоединение виртуальных машин Windows к домену

После назначения виртуальным машинам статических IP-адресов добавьте эти машины в домен.

![Рисунок 17. Добавление виртуальной машины в домен][sap-ha-guide-figure-3006]

_**Рисунок 17.** Добавление виртуальной машины в домен_

### <a name="add-registry-entries-on-both-cluster-nodes-of-the-sap-ascsscs-instance"></a><a name="661035b2-4d0f-4d31-86f8-dc0a50d78158"></a> Добавление записей реестра для обоих узлов кластера экземпляра SAP ASCS/SCS

Azure Load Balancer имеет внутренний балансировщик нагрузки, который закрывает подключения, если они простаивают определенный период времени (время простоя). Рабочие процессы SAP в диалоговых экземплярах открывают подключения к процессу постановки в очередь SAP при первой же необходимости отправить первый запрос на постановку в очередь или вывод из очереди. Эти подключения обычно сохраняются до перезапуска рабочего процесса или процесса постановки в очередь. Однако если подключение бездействует в течение некоторого времени, внутренний балансировщик нагрузки Azure закроет его. Это не является проблемой, так как рабочие процессы SAP повторно установят подключение к процессу постановки в очередь, если обнаружат его отсутствие. Эти действия будут описаны в трассировках разработчика для процессов SAP, но это приведет к созданию большого объема содержимого в этих трассировках. Поэтому рекомендуется изменить параметры TCP/IP `KeepAliveTime` и `KeepAliveInterval` на обоих узлах кластера. Необходимо также изменить параметры профиля SAP, которые будут описаны далее в этом документе.

Чтобы добавить записи реестра на обоих узлах кластера экземпляра SAP ASCS/SCS, сначала добавьте эти записи реестра Windows на обоих узлах кластера Windows для SAP ASCS/SCS.

| путь | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| Имя переменной |`KeepAliveTime` |
| Тип переменной |REG_DWORD (десятичное) |
| Значение |120000 |
| Ссылка на документацию |[https://technet.microsoft.com/library/cc957549.aspx](/previous-versions/windows/it-pro/windows-2000-server/cc957549(v=technet.10)) |

_**Таблица 3.** Изменение первого параметра TCP/IP_

Затем добавьте следующие записи реестра Windows на обоих узлах кластера Windows для SAP ASCS/SCS.

| путь | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| Имя переменной |`KeepAliveInterval` |
| Тип переменной |REG_DWORD (десятичное) |
| Значение |120000 |
| Ссылка на документацию |[https://technet.microsoft.com/library/cc957548.aspx](/previous-versions/windows/it-pro/windows-2000-server/cc957548(v=technet.10)) |

_**Таблица 4.** Изменение второго параметра TCP/IP_

**Чтобы применить изменения, перезапустите оба узла кластера.**

### <a name="set-up-a-windows-server-failover-clustering-cluster-for-an-sap-ascsscs-instance"></a><a name="0d67f090-7928-43e0-8772-5ccbf8f59aab"></a> Установка отказоустойчивого кластера Windows Server для экземпляра SAP ASCS/SCS

Настройка отказоустойчивого кластера Windows Server для экземпляра SAP ASCS/SCS состоит из следующих заданий:

- сбор узлов кластера в конфигурации кластера;
- настройка файлового ресурса-свидетеля кластера.

#### <a name="collect-the-cluster-nodes-in-a-cluster-configuration"></a><a name="5eecb071-c703-4ccc-ba6d-fe9c6ded9d79"></a> Сбор узлов кластера в конфигурации кластера

1. В мастере добавления ролей и компонентов добавьте компонент отказоустойчивой кластеризации на оба узла кластера.
2. Настройте отказоустойчивый кластер с помощью диспетчера отказоустойчивости кластеров. В диспетчере отказоустойчивости кластеров выберите **Создать кластер** и добавьте только имя первого узла A кластера. Пока не добавляйте второй узел. Он будет добавлен позже.

   ![Рисунок 18. Добавление имени сервера или виртуальной машины первого узла кластера][sap-ha-guide-figure-3007]

   _**Рисунок 18.** Добавление имени сервера или виртуальной машины первого узла кластера_

3. Введите сетевое имя кластера (имя виртуального узла).

   ![Рисунок 19. Ввод имени кластера][sap-ha-guide-figure-3008]

   _**Рисунок 19.** Ввод имени кластера_

4. После создания кластера запустите проверочный тест.

   ![Рисунок 20. Проверка кластера][sap-ha-guide-figure-3009]

   _**Рисунок 20.** Проверка кластера_

   На этом этапе можно игнорировать любые предупреждения о дисках. Файловый ресурс-свидетель и общие диски SIOS будут добавлены позже. На этом этапе кворум нам не важен.

   ![Рисунок 21. Предупреждение об отсутствии диска кворума][sap-ha-guide-figure-3010]

   _**Рисунок 21.** Предупреждение об отсутствии диска кворума_

   ![Рисунок 22. Определение основного ресурса кластера с IP-адресом (требуется новый IP-адрес)][sap-ha-guide-figure-3011]

   _**Рисунок 22.** Определение основного ресурса кластера с IP-адресом (требуется новый IP-адрес)_

5. Измените IP-адрес основной службы кластера. Кластер не может запуститься, пока не будет изменен IP-адрес основной службы кластера. Это связано с тем, что IP-адрес сервера указывает на один из узлов виртуальных машин. Его можно изменить на странице **Свойства** ресурса IP-адреса основной службы кластера.

   Например, нужно назначить IP-адрес (в нашем примере — **10.0.0.42**) для имени виртуального узла кластера **pr1-ascs-vir**.

   ![Рисунок 23. Использование окна "Свойства" для изменения IP-адреса][sap-ha-guide-figure-3012]

   _**Рисунок 23.** Использование окна **Свойства** для изменения IP-адреса_

   ![Рисунок 24. Назначение IP-адреса, зарезервированного для кластера][sap-ha-guide-figure-3013]

   _**Рисунок 24.** Назначение IP-адреса, зарезервированного для кластера_

6. Подключите имя виртуального узла кластера к сети.

   ![Рисунок 25. Основная служба кластера запущена и работает с правильным IP-адресом][sap-ha-guide-figure-3014]

   _**Рисунок 25.** Основная служба кластера запущена и работает с правильным IP-адресом_

7. Добавьте второй узел кластера.

   Теперь, когда основная служба кластера запущена и работает, можно добавить второй узел кластера.

   ![Рисунок 26. Добавление второго узла кластера][sap-ha-guide-figure-3015]

   _**Рисунок 26.** Добавление второго узла кластера_

8. Введите имя второго узла кластера.

   ![Рисунок 27. Ввод имени второго узла кластера][sap-ha-guide-figure-3016]

   _**Рисунок 27.** Ввод имени второго узла кластера_

   > [!IMPORTANT]
   > **Снимите** флажок **Добавление всех допустимых хранилищ в кластер**.  
   >
   >

   ![Рисунок 28. Этот флажок не должен быть установлен][sap-ha-guide-figure-3017]

   _**Рисунок 28.** Этот флажок **не должен** быть установлен_

   Предупреждения о кворуме и дисках можно игнорировать. Настройка диска кворума и общих дисков будет выполнена позже, как описано в разделе [Установка SIOS DataKeeper Cluster Edition для общего диска кластера SAP ASCS/SCS][sap-ha-guide-8.12.3].

   ![Рисунок 29. Предупреждение о диске кворума можно игнорировать][sap-ha-guide-figure-3018]

   _**Рисунок 29.** Предупреждение о диске кворума можно игнорировать_


#### <a name="configure-a-cluster-file-share-witness"></a><a name="e49a4529-50c9-4dcf-bde7-15a0c21d21ca"></a> Настройка файлового ресурса-свидетеля кластера

Настройка файлового ресурса-свидетеля кластера состоит из следующих заданий:

- создание файлового ресурса;
- настройка кворума на основе файлового ресурса-свидетеля в диспетчере отказоустойчивости кластеров.

##### <a name="create-a-file-share"></a><a name="06260b30-d697-4c4d-b1c9-d22c0bd64855"></a> Создание файлового ресурса

1. Вместо диска кворума нужно выбрать файловый ресурс-свидетель. SIOS DataKeeper поддерживает такой вариант.

   В примерах в этой статье файловый ресурс-свидетель настроен на сервере Active Directory или DNS-сервере, работающем в Azure. Этот файловый ресурс-свидетель называется **domcontr 0**. Так как вам пришлось бы настроить VPN-подключение к Azure (подключение типа "сеть — сеть" или канал ExpressRoute), то службы Active Directory и DNS находились бы в локальной среде и поэтому не подошли бы для работы файлового ресурса-свидетеля.

   > [!NOTE]
   > Если службы Active Directory и DNS работают только в локальной среде, не следует настраивать файловый ресурс-свидетель в операционной системе Windows для служб Active Directory и DNS, запущенной локально. Задержка сети между узлами кластера, работающими в Azure, и локальной средой служб Active Directory и DNS, может быть слишком большой и вызвать проблемы с подключением. Файловый ресурс-свидетель следует настроить на виртуальной машине Azure, размещенной близко от узла кластера.  
   >
   >

   На диске кворума должно не менее 1024 МБ свободного места. Мы рекомендуем выделить 2048 МБ свободного места для диска кворума.

2. Добавьте объект имени кластера.

   ![Рисунок 30. Назначение разрешений для общего ресурса для объекта имени кластера][sap-ha-guide-figure-3019]

   _**Рисунок 30.** Назначение разрешений для общего ресурса для объекта имени кластера_

   Убедитесь, что разрешения позволяют изменять данные в общем ресурсе для объекта имени кластера (в нашем примере это **pr1-ascs-vir$** ).

3. Чтобы добавить объект имени кластера в список, нажмите кнопку **Добавить**. Измените фильтр для добавления дополнительных объектов-компьютеров к показанным на рис. 31.

   ![Рисунок 31. Изменение типов объектов для добавления компьютеров][sap-ha-guide-figure-3020]

   _**Рисунок 31.** Изменение типов объектов для добавления компьютеров_

   ![Рисунок 32. Установка флажка "Компьютеры"][sap-ha-guide-figure-3021]

   _**Рисунок 32.** Установка флажка **Компьютеры**_

4. Введите объект имени кластера, как показано на рис. 31. Так как запись уже создана, можно изменить разрешения, как показано на рис. 30.

5. Откройте вкладку **Безопасность** общего ресурса и определите более детальные разрешения для объекта имени кластера.

   ![Рисунок 33. Настройка атрибутов безопасности для объекта имени кластера в кворуме на основе файлового ресурса][sap-ha-guide-figure-3022]

   _**Рисунок 33.** Настройка атрибутов безопасности для объекта имени кластера в кворуме на основе файлового ресурса_

##### <a name="set-the-file-share-witness-quorum-in-failover-cluster-manager"></a><a name="4c08c387-78a0-46b1-9d27-b497b08cac3d"></a> Настройка кворума на основе файлового ресурса-свидетеля в диспетчере отказоустойчивости кластеров

1. Откройте мастер настройки параметров кворума.

   ![Рисунок 34. Запуск мастера настройки кворума кластера][sap-ha-guide-figure-3023]

   _**Рисунок 34.** Запуск мастера настройки кворума кластера_

2. На странице **Выберите конфигурацию кворума** установите флажок **Выбрать свидетель кворума**.

   ![Рисунок 35. Выбор конфигураций кворума][sap-ha-guide-figure-3024]

   _**Рисунок 35:** Выбор конфигураций кворума_

3. На странице **Выбрать свидетель кворума** установите флажок **Настроить файловый ресурс-свидетель**.

   ![Рисунок 36. Выбор файлового ресурса-свидетеля][sap-ha-guide-figure-3025]

   _**Рисунок 36.** Выбор файлового ресурса-свидетеля_

4. Введите путь UNC к файловому ресурсу (в нашем примере это \\domcontr-0\FSW). Чтобы просмотреть список изменений, которые можно внести, нажмите кнопку **Далее**.

   ![Рисунок 37. Определение расположения файлового ресурса-свидетеля][sap-ha-guide-figure-3026]

   _**Рисунок 37.** Определение расположения файлового ресурса-свидетеля_

5. Выберите нужные изменения и нажмите кнопку **Далее**. Конфигурацию кластера необходимо перенастроить, как показано на рис. 38.  

   ![Рисунок 38. Подтверждение изменения настройки кластера][sap-ha-guide-figure-3027]

   _**Рисунок 38.** Подтверждение изменения настройки кластера_

После успешной установки отказоустойчивого кластера Windows необходимо внести изменения в некоторые пороговые значения, чтобы адаптировать обнаружение отработки отказа к условиям в Azure. Значения параметров, которые нужно изменить, указаны в следующей статье блога: https://blogs.msdn.microsoft.com/clustering/2012/11/21/tuning-failover-cluster-network-thresholds/. Предположим, что две виртуальные машины, образующие конфигурацию кластера Windows для ASCS/SCS, находятся в одной подсети. Тогда нужно изменить значения приведенных ниже параметров следующими значениями.
- SameSubNetDelay = 2
- SameSubNetThreshold = 15

Эти параметры были протестированы у клиентов, и, с одной стороны, они обеспечивали удачный компромисс с точки зрения устойчивости. С другой стороны, эти параметры обеспечивали достаточно быструю отработку отказа в реальных условиях сбоя программного обеспечения SAP либо узла или виртуальной машины. 

### <a name="install-sios-datakeeper-cluster-edition-for-the-sap-ascsscs-cluster-share-disk"></a><a name="5c8e5482-841e-45e1-a89d-a05c0907c868"></a> Установка SIOS DataKeeper Cluster Edition для создания общего диска кластера SAP ASCS/SCS

Теперь у вас есть рабочая конфигурация отказоустойчивого кластера Windows Server в Azure. Однако чтобы установить экземпляр SAP ASCS/SCS, требуется общий дисковый ресурс. Azure не позволяет создавать общие дисковые ресурсы. Поэтому для выполнения этой задачи придется воспользоваться SIOS DataKeeper Cluster Edition — сторонним решением для создания общих дисковых ресурсов.

Установка SIOS DataKeeper Cluster Edition для общего диска кластера SAP ASCS/SCS состоит из следующих заданий:

- добавление .NET Framework 3.5;
- Установка SIOS DataKeeper
- настройка SIOS DataKeeper.

#### <a name="add-the-net-framework-35"></a><a name="1c2788c3-3648-4e82-9e0d-e058e475e2a3"></a> Добавление .NET Framework 3.5
Компонент Microsoft .NET Framework 3.5 не установлен и не включен автоматически в Windows Server 2012 R2. Для работы SIOS DataKeeper требуется установить .NET Framework на всех узлах, где устанавливается этот продукт. Поэтому необходимо установить компонент .NET Framework 3.5 в операционной системе на всех виртуальных машинах в кластере.

.NET Framework 3.5 можно добавить двумя способами.

- Первый способ — с помощью мастера добавления ролей и компонентов в Windows, как показано на рис. 39.

  ![Рисунок 39. Установка .NET Framework 3.5 с помощью мастера добавления ролей и компонентов][sap-ha-guide-figure-3028]

  _**Рисунок 39.** Установка .NET Framework 3.5 с помощью мастера добавления ролей и компонентов_

  ![Рисунок 40. Ход установки .NET Framework 3.5 при установке с помощью мастера добавления ролей и компонентов][sap-ha-guide-figure-3029]

  _**Рисунок 40.** Ход установки .NET Framework 3.5 при установке с помощью мастера добавления ролей и компонентов_

- Второй способ — с помощью средства командной строки dism.exe. Для этого необходим доступ к каталогу SxS на установочном носителе Windows. В командной строке с повышенными привилегиями выполните следующую команду:

  ```
  Dism /online /enable-feature /featurename:NetFx3 /All /Source:installation_media_drive:\sources\sxs /LimitAccess
  ```

#### <a name="install-sios-datakeeper"></a><a name="dd41d5a2-8083-415b-9878-839652812102"></a> Установка SIOS DataKeeper

SIOS DataKeeper Cluster Edition нужно установить на обоих узлах в кластере. Чтобы создать виртуальное общее хранилище с помощью SIOS DataKeeper, создайте синхронизированное зеркало и сымитируйте общее хранилище кластера.

Перед установкой программного обеспечения SIOS необходимо создать пользователя домена **DataKeeperSvc**.

> [!NOTE]
> Добавьте пользователя **DataKeeperSvc** в группу **локальных администраторов** на обоих узлах кластера.
>
>

Чтобы установить SIOS DataKeeper, сделайте следующее:

1. Установите программное обеспечение SIOS на обоих узлах кластера.

   ![Установщик SIOS][sap-ha-guide-figure-3030]

   ![Рисунок 41. Первая страница установки SIOS DataKeeper][sap-ha-guide-figure-3031]

   _**Рисунок 41.** Первая страница установки SIOS DataKeeper_

2. В диалоговом окне, показанном на рис. 42, выберите **Да**.

   ![Рисунок 42. Предупреждение DataKeeper об отключении службы][sap-ha-guide-figure-3032]

   _**Рисунок 42.** Предупреждение DataKeeper об отключении службы_

3. В диалоговом окне, показанном на рис. 43, мы рекомендуем установить флажок **Domain or Server account** (Учетная запись домена или сервера).

   ![Рисунок 43. Выбор пользователя для SIOS DataKeeper][sap-ha-guide-figure-3033]

   _**Рисунок 43.** Выбор пользователя для SIOS DataKeeper_

4. Укажите имя учетной записи пользователя домена и пароли, созданные для SIOS DataKeeper.

   ![Рисунок 44. Ввод имени пользователя и пароля для установки SIOS DataKeeper][sap-ha-guide-figure-3034]

   _**Рисунок 44.** Ввод имени пользователя и пароля для установки SIOS DataKeeper_

5. Установите лицензионный ключ для своего экземпляра SIOS DataKeeper, как показано на рис. 45.

   ![Рисунок 45. Ввод ключа лицензии на использование SIOS DataKeeper][sap-ha-guide-figure-3035]

   _**Рисунок 45.** Ввод ключа лицензии на использование SIOS DataKeeper_

6. При появлении запроса перезапустите виртуальную машину.

#### <a name="set-up-sios-datakeeper"></a><a name="d9c1fc8e-8710-4dff-bec2-1f535db7b006"></a> Настройка SIOS DataKeeper

После установки SIOS DataKeeper на обоих узлах нужно начать настройку параметров. Наша цель — настроить синхронную репликацию данных между дополнительными дисками, подключенными к каждой виртуальной машине.

1. Запустите инструмент управления и настройки DataKeeper, а затем щелкните **Подключение к серверу**. (На рис. 46 эта ссылка обведена красным.)

   ![Рисунок 46. Инструмент управления и настройки SIOS DataKeeper][sap-ha-guide-figure-3036]

   _**Рисунок 46.** Инструмент управления и настройки SIOS DataKeeper_

2. Введите имя или TCP/IP-адрес первого узла, к которому должен подключиться инструмент управления и настройки, а на втором шаге — для второго узла.

   ![Рисунок 47. Ввод имени или TCP/IP-адреса первого узла, к которому должен подключаться инструмент управления и настройки, а также второго узла на втором шаге][sap-ha-guide-figure-3037]

   _**Рисунок 47.** Ввод имени или TCP/IP-адреса первого узла, к которому должен подключаться инструмент управления и настройки, а также второго узла на втором шаге_

3. Создайте задание репликации между двумя узлами.

   ![Рисунок 48. Создание задания репликации][sap-ha-guide-figure-3038]

   _**Рисунок 48.** Создание задания репликации_

   Мастер поможет выполнить эту процедуру.
4. Определите имя, TCP/IP-адрес и дисковый том исходного узла.

   ![Рисунок 49. Определение имени для задания репликации][sap-ha-guide-figure-3039]

   _**Рисунок 49.** Определение имени для задания репликации_

   ![Рисунок 50. Определение базовых данных для узла, который должен быть текущим исходным узлом][sap-ha-guide-figure-3040]

   _**Рисунок 50.** Определение базовых данных для узла, который должен быть текущим исходным узлом_

5. Определите имя, TCP/IP-адрес и дисковый том целевого узла.

   ![Рисунок 51. Определение базовых данных для узла, который должен быть текущим целевым узлом][sap-ha-guide-figure-3041]

   _**Рисунок 51.** Определение базовых данных для узла, который должен быть текущим целевым узлом_

6. Определите алгоритм сжатия. Для наших целей рекомендуется сжатие потока репликации. Сжатие потока репликации значительно сокращает время повторной синхронизации (особенно в случаях повторной синхронизации). Имейте в виду, что сжатие использует ресурсы ЦП и памяти виртуальной машины. Чем выше степень сжатия, тем больше ресурсов ЦП будет использоваться. Этот параметр можно изменить позже.

7. Кроме того, нужно проверить параметр, который позволяет включить синхронный или асинхронный режим репликации. *Для защиты конфигураций SAP ASCS/SCS требуется синхронная репликация*.  

   ![Рисунок 52. Определение сведений о репликации][sap-ha-guide-figure-3042]

   _**Рисунок 52.** Определение сведений о репликации_

8. Определите, следует ли том, реплицируемый с помощью задания репликации, представить в конфигурации кластера WSFC как общий диск. Для конфигурации SAP ASCS/SCS выберите **Да**, чтобы кластер Windows воспринимал реплицируемый том как общий диск, который можно использовать в качестве тома кластера.

   ![Рисунок 53. Подтверждение согласия на выбор реплицируемого тома в качестве тома кластера][sap-ha-guide-figure-3043]

   _**Рисунок 53.** **Подтверждение** согласия на выбор реплицируемого тома в качестве тома кластера_

   После создания тома инструмент управления и настройки DataKeeper отобразят задание репликации как активное.

   ![Рисунок 54. Синхронное зеркальное отображение DataKeeper для общего диска SAP ASCS/SCS активно][sap-ha-guide-figure-3044]

   _**Рисунок 54.** Синхронное зеркальное отображение DataKeeper для общего диска SAP ASCS/SCS активно_

   Теперь диск отображается в диспетчере отказоустойчивости кластеров как диск DataKeeper, как показано на рис. 55.

   ![Рисунок 55. Реплицируемый диск DataKeeper отображается в диспетчере отказоустойчивости кластеров][sap-ha-guide-figure-3045]

   _**Рисунок 55.** Реплицируемый диск DataKeeper отображается в диспетчере отказоустойчивости кластеров_

## <a name="install-the-sap-netweaver-system"></a><a name="a06f0b49-8a7a-42bf-8b0d-c12026c5746b"></a> Установка системы SAP NetWeaver

Мы не описываем настройку СУБД, так как она зависит от используемой СУБД. Тем не менее мы предполагаем, что задачи обеспечения высокого уровня доступности СУБД решены благодаря возможностям различных поставщиков СУБД, поддерживающих Azure. Например, AlwaysOn или зеркальное отображение базы данных для SQL Server и Oracle Data Guard для баз данных Oracle. В нашем примере сценария мы не реализуем дополнительную защиту СУБД.

Особые рекомендации по взаимодействию различных СУБД с такой кластеризованной конфигурацией SAP ASCS/SCS в Azure отсутствуют.

> [!NOTE]
> Процедуры установки систем SAP NetWeaver ABAP, SAP Java и SAP ABAP с Java практически идентичны. Главное отличие состоит в том, что в системе SAP ABAP всего один экземпляр ASCS. В системе SAP Java имеется один экземпляр SCS. В системе SAP ABAP с Java — один экземпляр ASCS и один экземпляр SCS, которые работают в одной группе отказоустойчивого кластера Майкрософт. Любые отличия в установке для каждого стека установки SAP NetWeaver будут указаны явным образом. Все остальные составляющие считаются одинаковыми.  
>
>

### <a name="install-sap-with-a-high-availability-ascsscs-instance"></a><a name="31c6bd4f-51df-4057-9fdf-3fcbc619c170"></a> Установка SAP с высокодоступным экземпляром ASCS/SCS

> [!IMPORTANT]
> Не размещайте файл подкачки на зеркальных томах DataKeeper. DataKeeper не поддерживает зеркальные тома. Можно оставить файл подкачки на временном диске D виртуальной машины Azure, который задан по умолчанию. Если его там нет, то поместите файл подкачки Windows на диск D виртуальной машины Azure.
>
>

Установка SAP с высокодоступным экземпляром ASCS/SCS состоит из следующих заданий:

- создание имени виртуального узла для кластеризованного экземпляра SAP ASCS/SCS;
- установка первого узла кластера SAP;
- изменение профиля SAP экземпляра ASCS/SCS;
- добавление порта пробы;
- открытие порта пробы в брандмауэре Windows.

#### <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance"></a><a name="a97ad604-9094-44fe-a364-f89cb39bf097"></a> Создание имени виртуального узла для кластеризованного экземпляра SAP ASCS/SCS

1. В диспетчере DNS Windows создайте запись DNS для имени виртуального узла экземпляра ASCS/SCS.

   > [!IMPORTANT]
   > IP-адрес, который вы назначите имени виртуального узла для экземпляра ASCS/SCS, должен совпадать с IP-адресом, назначенным для Azure Load Balancer ( **<*SID*>-lb-ascs**).  
   >
   >

   IP-адрес имени виртуального узла SAP ASCS/SCS (**pr1-ascs-sap**) совпадает с IP-адресом Azure Load Balancer (**pr1-lb-ascs**).

   ![Рисунок 56. Настройка записи DNS для виртуального имени и TCP/IP-адреса кластера SAP ASCS/SCS][sap-ha-guide-figure-3046]

   _**Рисунок 56.** Настройка записи DNS для виртуального имени и TCP/IP-адреса кластера SAP ASCS/SCS_

2. Чтобы определить IP-адрес виртуального имени узла, выберите **Диспетчер DNS** > **Домен**.

   ![Рисунок 57. Новое виртуальное имя и TCP/IP-адрес для конфигурации кластера SAP ASCS/SCS][sap-ha-guide-figure-3047]

   _**Рисунок 57.** Новое виртуальное имя и TCP/IP-адрес для конфигурации кластера SAP ASCS/SCS_

#### <a name="install-the-sap-first-cluster-node"></a><a name="eb5af918-b42f-4803-bb50-eff41f84b0b0"></a> Установка первого узла кластера SAP

1. Выполните параметр первого узла кластера на узле А кластера, например на узле **pr1-ascs-0**.
2. Чтобы сохранить порты по умолчанию для внутреннего балансировщика нагрузки Azure, выберите:

   * **Для системы ABAP**: экземпляр **ASCS** с номером **00**;
   * **Для системы Java**: экземпляр **SCS** с номером **01**;
   * **Для системы ABAP+Java**: экземпляр **ASCS** с номером **00** и экземпляр **SCS** с номером **01**.

   Чтобы использовать номера экземпляров, отличные от 00 для ABAP ASCS и от 01 для Java SCS, сначала необходимо изменить правила балансировки нагрузки по умолчанию для внутреннего балансировщика нагрузки Azure, как описано в разделе [Изменение правил балансировки нагрузки ASCS/SCS по умолчанию для внутреннего балансировщика нагрузки Azure][sap-ha-guide-8.9].

Следующие несколько заданий не описаны в обычной документации по установке SAP.

> [!NOTE]
> В документации по установке SAP содержатся сведения по установке первого узла кластера ASCS/SCS.
>
>

#### <a name="modify-the-sap-profile-of-the-ascsscs-instance"></a><a name="e4caaab2-e90f-4f2c-bc84-2cd2e12a9556"></a> Изменение профиля SAP экземпляра ASCS/SCS

Необходимо добавить новый параметр профиля. Этот параметр профиля запрещает закрытие подключений между рабочими процессами SAP и сервером постановки в очередь, если они простаивают слишком долго. Мы уже упоминали проблему в разделе [Добавление записей реестра для обоих узлов кластера экземпляра SAP ASCS/SCS][sap-ha-guide-8.11]. В этом разделе мы также внесли два изменения в некоторые основные параметры подключения по TCP/IP. На втором шаге необходимо настроить сервер постановки в очередь для отправки сигнала `keep_alive`, чтобы простой подключений не превышал пороговое значение, заданное во внутреннем балансировщике нагрузки Azure.

Чтобы изменить профиль SAP экземпляра ASCS/SCS, сделайте следующее:

1. Добавьте этот параметр профиля в профиль экземпляра SAP ASCS/SCS:

   ```
   enque/encni/set_so_keepalive = true
   ```
   В нашем примере путь следующий:

   `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_ASCS00_pr1-ascs-sap`

   Например, ниже приведен профиль экземпляра SAP SCS и соответствующий путь.

   `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_SCS01_pr1-ascs-sap`

2. Чтобы применить изменения, перезапустите экземпляр SAP ASCS /SCS.

#### <a name="add-a-probe-port"></a><a name="10822f4f-32e7-4871-b63a-9b86c76ce761"></a> Добавление порта пробы

Чтобы вся конфигурация кластера работала с Azure Load Balancer, нужно использовать функцию проб внутреннего балансировщика нагрузки. Обычно внутренний балансировщик нагрузки Azure распределяет входящую рабочую нагрузку поровну между участвующими виртуальными машинами. Однако это не будет работать в некоторых конфигурациях кластера, так как активен только один экземпляр. Другой экземпляр пассивен и не может принимать рабочую нагрузку. Функция проб полезна в случаях, когда внутренний балансировщик нагрузки Azure назначает работу только активным экземплярам. С помощью этой функции внутренний балансировщик нагрузки может определить, какой из экземпляров активен, и затем направлять рабочую нагрузку только на него.

Чтобы добавить порт пробы, сделайте следующее:

1. Проверьте текущий параметр **ProbePort**, выполнив приведенную ниже команду PowerShell. Выполните ее на одной из виртуальных машин в конфигурации кластера.

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPNetworkIPClusterName = "SAP $SAPSID IP"
   Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter
   ```

2. Укажите порт пробы. По умолчанию для проб задан порт **0**. В нашем примере мы используем порт пробы **62000**.

   ![Рисунок 58. По умолчанию в конфигурации кластера для проб указан порт 0][sap-ha-guide-figure-3048]

   _**Рисунок 58.** По умолчанию в конфигурации кластера для проб указан порт 0_

   Номер порта определен в шаблонах Azure Resource Manager для SAP. Назначить номер порта можно в PowerShell.

   Чтобы задать новое значение ProbePort для кластерного ресурса **SAP <*SID*> IP**, выполните следующий скрипт PowerShell. Обновите переменные PowerShell для своей среды. После выполнения сценария вам будет предложено перезапустить кластерную группу SAP, чтобы активировать изменения.

   ```powershell
   $SAPSID = "PR1"      # SAP <SID>
   $ProbePort = 62000   # ProbePort of the Azure Internal Load Balancer

   Clear-Host
   $SAPClusterRoleName = "SAP $SAPSID"
   $SAPIPresourceName = "SAP $SAPSID IP"
   $SAPIPResourceClusterParameters =  Get-ClusterResource $SAPIPresourceName | Get-ClusterParameter
   $IPAddress = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Address" }).Value
   $NetworkName = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Network" }).Value
   $SubnetMask = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "SubnetMask" }).Value
   $OverrideAddressMatch = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "OverrideAddressMatch" }).Value
   $EnableDhcp = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "EnableDhcp" }).Value
   $OldProbePort = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "ProbePort" }).Value

   $var = Get-ClusterResource | Where-Object {  $_.name -eq $SAPIPresourceName  }

   Write-Host "Current configuration parameters for SAP IP cluster resource '$SAPIPresourceName' are:" -ForegroundColor Cyan
   Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter

   Write-Host
   Write-Host "Current probe port property of the SAP cluster resource '$SAPIPresourceName' is '$OldProbePort'." -ForegroundColor Cyan
   Write-Host
   Write-Host "Setting the new probe port property of the SAP cluster resource '$SAPIPresourceName' to '$ProbePort' ..." -ForegroundColor Cyan
   Write-Host

   $var | Set-ClusterParameter -Multiple @{"Address"=$IPAddress;"ProbePort"=$ProbePort;"Subnetmask"=$SubnetMask;"Network"=$NetworkName;"OverrideAddressMatch"=$OverrideAddressMatch;"EnableDhcp"=$EnableDhcp}

   Write-Host

   $ActivateChanges = Read-Host "Do you want to take restart SAP cluster role '$SAPClusterRoleName', to activate the changes (yes/no)?"

   if($ActivateChanges -eq "yes"){
   Write-Host
   Write-Host "Activating changes..." -ForegroundColor Cyan

   Write-Host
   write-host "Taking SAP cluster IP resource '$SAPIPresourceName' offline ..." -ForegroundColor Cyan
   Stop-ClusterResource -Name $SAPIPresourceName
   sleep 5

   Write-Host "Starting SAP cluster role '$SAPClusterRoleName' ..." -ForegroundColor Cyan
   Start-ClusterGroup -Name $SAPClusterRoleName

   Write-Host "New ProbePort parameter is active." -ForegroundColor Green
   Write-Host

   Write-Host "New configuration parameters for SAP IP cluster resource '$SAPIPresourceName':" -ForegroundColor Cyan
   Write-Host
   Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter
   }else
   {
   Write-Host "Changes are not activated."
   }
   ```

   Подключив роль кластера **SAP <*SID*>** , убедитесь, что используется новое значение параметра **ProbePort**.

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPNetworkIPClusterName = "SAP $SAPSID IP"
   Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter

   ```

   ![Рисунок 59. Проверка порта пробы кластера после изменения значения][sap-ha-guide-figure-3049]

   _**Рисунок 59.** Проверка порта пробы кластера после изменения значения_

#### <a name="open-the-windows-firewall-probe-port"></a><a name="4498c707-86c0-4cde-9c69-058a7ab8c3ac"></a> Открытие порта пробы в брандмауэре Windows

Порт пробы брандмауэра Windows необходимо открыть на обоих узлах кластера. Чтобы открыть порт пробы брандмауэра Windows, используйте следующий скрипт. Обновите переменные PowerShell для своей среды.

  ```powershell
  $ProbePort = 62000   # ProbePort of the Azure Internal Load Balancer

  New-NetFirewallRule -Name AzureProbePort -DisplayName "Rule for Azure Probe Port" -Direction Inbound -Action Allow -Protocol TCP -LocalPort $ProbePort
  ```

Для **ProbePort** установлено значение **62000**. Теперь доступ к файловому ресурсу **\\\ascsha-clsap\sapmnt** можно получить из других узлов, например **ascsha-dbas**.

### <a name="install-the-database-instance"></a><a name="85d78414-b21d-4097-92b6-34d8bcb724b7"></a> Установка экземпляра базы данных

Чтобы установить экземпляр базы данных, следуйте инструкциям в документации по установке SAP.

### <a name="install-the-second-cluster-node"></a><a name="8a276e16-f507-4071-b829-cdc0a4d36748"></a> Установка второго узла кластера

Чтобы установить второй кластер, выполните действия, описанные в руководстве по установке SAP.

### <a name="change-the-start-type-of-the-sap-ers-windows-service-instance"></a><a name="094bc895-31d4-4471-91cc-1513b64e406a"></a> Изменение типа запуска службы Windows для экземпляра SAP ERS

На обоих узлах кластера измените тип запуска служб Windows SAP ERS на **Автоматически (отложенный запуск)** .

![Рисунок 60. Изменение типа службы для экземпляра SAP ERS на "Автоматически (отложенный запуск)"][sap-ha-guide-figure-3050]

_**Рисунок 60.** Изменение типа службы для экземпляра SAP ERS на "Автоматически (отложенный запуск)"_

### <a name="install-the-sap-primary-application-server"></a><a name="2477e58f-c5a7-4a5d-9ae3-7b91022cafb5"></a> Установка основного сервера приложений SAP

Установите экземпляр основного сервера приложений (PAS) <*SID*>-di-0> на виртуальную машину, предназначенную для размещения PAS. Какие-либо специальные зависимости от параметров Azure или DataKeeper отсутствуют.

### <a name="install-the-sap-additional-application-server"></a><a name="0ba4a6c1-cc37-4bcf-a8dc-025de4263772"></a> Установка дополнительного сервера приложений SAP

Установите дополнительный сервер приложений SAP (AAS) на все виртуальные машины, предназначенные для размещения экземпляра сервера приложений SAP. Например, с <*SID*>-di-1 по <*SID*>-di-&lt;n&gt;.

> [!NOTE]
> После этого установка системы SAP NetWeaver с высоким уровнем доступности будет завершена. Затем перейдите к тестированию отработки отказа.
>


## <a name="test-the-sap-ascsscs-instance-failover-and-sios-replication"></a><a name="18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9"></a> Тестирование отработки отказа экземпляра SAP ASCS/SCS и репликации SIOS
С помощью диспетчера отказоустойчивости кластеров и инструмента управления и настройки SIOS DataKeeper можно легко выполнять тестирование и мониторинг отработки отказа экземпляров SAP ASCS/SCS и репликации дисков SIOS.

### <a name="sap-ascsscs-instance-is-running-on-cluster-node-a"></a><a name="65fdef0f-9f94-41f9-b314-ea45bbfea445"></a> Экземпляр SAP ASCS/SCS выполняется на узле A кластера

Кластерная группа **SAP PR1** работает на узле A кластера, например **pr1-ascs-0**. Назначьте узлу A кластера общий диск S:, принадлежащий кластерной группе **SAP PR1** и используемый экземпляром ASCS/SCS.

![Рисунок 61. Диспетчер отказоустойчивости кластеров: кластерная группа SAP <SID> работает в узле A кластера][sap-ha-guide-figure-5000]

_**Рисунок 61.** Диспетчер отказоустойчивости кластеров: кластерная группа SAP <*SID*> работает в узле A кластера_

С помощью инструмента управления и настройки SIOS DataKeeper можно увидеть, что данные общего диска синхронно реплицируются с исходного дискового тома S: на узле А кластера на целевой дисковый том S: на узле B кластера (например, с **pr1-ascs-0 [10.0.0.40]** на **pr1-ascs-1 [10.0.0.41]** ).

![Рисунок 62. SIOS DataKeeper: репликация локального тома с узла A в узел B кластера][sap-ha-guide-figure-5001]

_**Рисунок 62.** SIOS DataKeeper: репликация локального тома с узла A в узел B кластера_

### <a name="failover-from-node-a-to-node-b"></a><a name="5e959fa9-8fcd-49e5-a12c-37f6ba07b916"></a> Отработка отказа с узла A на узел B

1. Инициируйте отработку отказа кластерной группы SAP <*SID*> с узла A на узел B кластера одним из следующих способов:
   - воспользовавшись диспетчером отказоустойчивости кластеров;  
   - воспользовавшись командами PowerShell отказоустойчивого кластера;

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPClusterGroup = "SAP $SAPSID"
   Move-ClusterGroup -Name $SAPClusterGroup

   ```
2. перезапустив узел A кластера в гостевой ОС на виртуальной машине Windows (это инициирует автоматическую отработку отказа кластерной группы SAP <*SID*> с узла A на узел B кластера);  
3. перезапустив узел A кластера на портале Azure (это инициирует автоматическую отработку отказа кластерной группы SAP <*SID*> с узла A на узел B кластера);  
4. перезапустив узел A кластера с помощью Azure PowerShell (это инициирует автоматическую отработку отказа кластерной группы SAP <*SID*> с узла A на узел B кластера).

   После отработки отказа кластерная группа SAP <*SID*> выполняется на узле B кластера, например на узле **pr1-ascs-1**.

   ![Рисунок 63. Диспетчер отказоустойчивости кластеров: кластерная группа SAP <SID> работает в узле B кластера][sap-ha-guide-figure-5002]

   _**Рисунок 63.** Диспетчер отказоустойчивости кластеров: кластерная группа SAP <*SID*> работает в узле B кластера_

   Теперь общий диск подключен к узлу B кластера. SIOS DataKeeper реплицирует данные с исходного дискового тома S: на узле B кластера на целевой дисковый том S: на узле А кластера (например, с **pr1-ascs-1 [10.0.0.41]** на **pr1-ascs-0 [10.0.0.40]** ).

   ![Рисунок 64. SIOS DataKeeper: репликация локального тома с узла B в узел A кластера][sap-ha-guide-figure-5003]

   _**Рисунок 64.** SIOS DataKeeper: репликация локального тома с узла B в узел A кластера_
