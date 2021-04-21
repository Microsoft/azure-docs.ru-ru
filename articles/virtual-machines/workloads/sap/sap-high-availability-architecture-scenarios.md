---
title: Архитектура высокого уровня доступности (HA) и сценарии для SAP NetWeaver на виртуальных машинах Azure | Документация Майкрософт
description: Архитектура высокого уровня доступности и сценарии для SAP NetWeaver на виртуальных машинах Azure
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 887caaec-02ba-4711-bd4d-204a7d16b32b
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 02/26/2020
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: d689db4916758e13311ad181fb3b5d7ca9a85260
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101675314"
---
# <a name="high-availability-architecture-and-scenarios-for-sap-netweaver"></a>Высокодоступная архитектура и сценарии для SAP NetWeaver

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png


[sap-installation-guides]:http://service.sap.com/instguides

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
[sap-ascs-ha-multi-sid-wsfc-file-share]:sap-ascs-ha-multi-sid-wsfc-file-share.md
[sap-ascs-ha-multi-sid-wsfc-shared-disk]:sap-ascs-ha-multi-sid-wsfc-shared-disk.md
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md
[sap-suse-ascs-ha-anf]:high-availability-guide-suse-netapp-files.md
[sap-higher-availability]:sap-higher-availability-architecture-scenarios.md

[planning-guide]:planning-guide.md  
[planning-guide-1.2]:planning-guide.md#e55d1e22-c2c8-460b-9897-64622a34fdff
[planning-guide-11]:planning-guide.md#7cf991a1-badd-40a9-944e-7baae842a058
[planning-guide-11.4.1]:planning-guide.md#5d9d36f9-9058-435d-8367-5ad05f00de77
[planning-guide-11.5]:planning-guide.md#4e165b58-74ca-474f-a7f4-5e695a93204f
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10
[planning-guide-3.1]:planning-guide.md#be80d1b9-a463-4845-bd35-f4cebdb5424a
[planning-guide-3.2.1]:planning-guide.md#df49dc09-141b-4f34-a4a2-990913b30358
[planning-guide-3.2.2]:planning-guide.md#fc1ac8b2-e54a-487c-8581-d3cc6625e560
[planning-guide-3.2.3]:planning-guide.md#18810088-f9be-4c97-958a-27996255c665
[planning-guide-3.2]:planning-guide.md#8d8ad4b8-6093-4b91-ac36-ea56d80dbf77
[planning-guide-3.3.2]:planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92
[planning-guide-5.1.1]:planning-guide.md#4d175f1b-7353-4137-9d2f-817683c26e53
[planning-guide-5.1.2]:planning-guide.md#e18f7839-c0e2-4385-b1e6-4538453a285c
[planning-guide-5.2.1]:planning-guide.md#1b287330-944b-495d-9ea7-94b83aff73ef
[planning-guide-5.2.2]:planning-guide.md#57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3
[planning-guide-5.2]:planning-guide.md#6ffb9f41-a292-40bf-9e70-8204448559e7
[planning-guide-5.3.1]:planning-guide.md#6e835de8-40b1-4b71-9f18-d45b20959b79
[planning-guide-5.3.2]:planning-guide.md#a43e40e6-1acc-4633-9816-8f095d5a7b6a
[planning-guide-5.4.2]:planning-guide.md#9789b076-2011-4afa-b2fe-b07a8aba58a1
[planning-guide-5.5.1]:planning-guide.md#4efec401-91e0-40c0-8e64-f2dceadff646
[planning-guide-5.5.3]:planning-guide.md#17e0d543-7e8c-4160-a7da-dd7117a1ad9d
[planning-guide-7.1]:planning-guide.md#3e9c3690-da67-421a-bc3f-12c520d99a30
[planning-guide-7]:planning-guide.md#96a77628-a05e-475d-9df3-fb82217e8f14
[planning-guide-9.1]:planning-guide.md#6f0a47f3-a289-4090-a053-2521618a28c3
[planning-guide-azure-premium-storage]:planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92

[virtual-machines-windows-portal-sql-alwayson-availability-groups-manual]:../../windows/sql/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual.md
[virtual-machines-windows-portal-sql-alwayson-int-listener]:../../../azure-sql/virtual-machines/windows/availability-group-load-balancer-portal-configure.md

[sap-ha-bc-virtual-env-hyperv-vmware-white-paper]:https://scn.sap.com/docs/DOC-44415
[sap-ha-partner-information]:https://scn.sap.com/docs/DOC-8541
[azure-sla]:https://azure.microsoft.com/support/legal/sla/
[azure-virtual-machines-manage-availability]:../../windows/manage-availability.md
[azure-storage-redundancy]:https://azure.microsoft.com/documentation/articles/storage-redundancy/
[azure-storage-managed-disks-overview]:https://docs.microsoft.com/azure/storage/storage-managed-disks-overview

[planning-guide-figure-100]:media/virtual-machines-shared-sap-planning-guide/100-single-vm-in-azure.png
[planning-guide-figure-1300]:media/virtual-machines-shared-sap-planning-guide/1300-ref-config-iaas-for-sap.png
[planning-guide-figure-1400]:media/virtual-machines-shared-sap-planning-guide/1400-attach-detach-disks.png
[planning-guide-figure-1600]:media/virtual-machines-shared-sap-planning-guide/1600-firewall-port-rule.png
[planning-guide-figure-1700]:media/virtual-machines-shared-sap-planning-guide/1700-single-vm-demo.png
[planning-guide-figure-1900]:media/virtual-machines-shared-sap-planning-guide/1900-vm-set-vnet.png
[planning-guide-figure-200]:media/virtual-machines-shared-sap-planning-guide/200-multiple-vms-in-azure.png
[planning-guide-figure-2100]:media/virtual-machines-shared-sap-planning-guide/2100-s2s.png
[planning-guide-figure-2200]:media/virtual-machines-shared-sap-planning-guide/2200-network-printing.png
[planning-guide-figure-2300]:media/virtual-machines-shared-sap-planning-guide/2300-sapgui-stms.png
[planning-guide-figure-2400]:media/virtual-machines-shared-sap-planning-guide/2400-vm-extension-overview.png
[planning-guide-figure-2500]:media/virtual-machines-shared-sap-planning-guide/planning-monitoring-overview-2502.png
[planning-guide-figure-2600]:media/virtual-machines-shared-sap-planning-guide/2600-sap-router-connection.png
[planning-guide-figure-2700]:media/virtual-machines-shared-sap-planning-guide/2700-exposed-sap-portal.png
[planning-guide-figure-2800]:media/virtual-machines-shared-sap-planning-guide/2800-endpoint-config.png
[planning-guide-figure-2900]:media/virtual-machines-shared-sap-planning-guide/2900-azure-ha-sap-ha.png
[planning-guide-figure-2901]:media/virtual-machines-shared-sap-planning-guide/2901-azure-ha-sap-ha-md.png
[planning-guide-figure-300]:media/virtual-machines-shared-sap-planning-guide/300-vpn-s2s.png
[planning-guide-figure-3000]:media/virtual-machines-shared-sap-planning-guide/3000-sap-ha-on-azure.png
[planning-guide-figure-3200]:media/virtual-machines-shared-sap-planning-guide/3200-sap-ha-with-sql.png
[planning-guide-figure-3201]:media/virtual-machines-shared-sap-planning-guide/3201-sap-ha-with-sql-md.png
[planning-guide-figure-400]:media/virtual-machines-shared-sap-planning-guide/400-vm-services.png
[planning-guide-figure-600]:media/virtual-machines-shared-sap-planning-guide/600-s2s-details.png
[planning-guide-figure-700]:media/virtual-machines-shared-sap-planning-guide/700-decision-tree-deploy-to-azure.png
[planning-guide-figure-800]:media/virtual-machines-shared-sap-planning-guide/800-portal-vm-overview.png
[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

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

[virtual-machines-manage-availability]:../../manage-availability.md


## <a name="terminology-definitions"></a>Определения терминов

**Высокий уровень доступности**. Относится к группе технологий, которые минимизируют перебои в работе ИТ-инфраструктуры, обеспечивая непрерывность работы ИТ-служб благодаря использованию избыточных компонентов, отказоустойчивых компонентов или компонентов, защищенных с помощью отработки отказа в *одном* центре обработки данных. В нашем случае центр обработки данных находится в пределах одного региона Azure.

**Аварийное восстановление.** Относится также к минимизации перебоев в работе ИТ-служб и их восстановлению, но в *разных* центрах обработки данных, которые находятся за сотни километров друг от друга. В нашем случае центры обработки данных могут находиться в различных регионах Azure в одном геополитическом регионе или расположениях, определенных клиентом.


## <a name="overview-of-high-availability"></a>Общие сведения о высокой доступности
Высокий уровень доступности SAP в Azure можно разделить на три типа:

* **Высокая доступность инфраструктуры Azure**. 

    Например, высокая доступность вычислений (на виртуальных машинах), сети или хранилища и ее преимущества для повышения доступности приложений SAP.

* **Использование перезапуска виртуальной машины в инфраструктуре Azure для *повышения доступности* приложений SAP**: 

    Если вы решили не применять такие функции, как отказоустойчивый кластер Windows Server (WSFC) или Pacemaker в Linux, используйте перезапуск виртуальной машины Azure. Это защищает системы SAP от запланированных и незапланированных простоев инфраструктуры физического сервера Azure и всей базовой платформы Azure.

* **Высокий уровень доступности приложений SAP**. 

    Для обеспечения высокой доступности всей системы SAP необходимо защитить все ее важные компоненты. Пример:
    * избыточные серверы приложений SAP;
    * уникальные компоненты. Примером может служить компонент единой точки отказа (SPOF), такой как экземпляр ASCS/SCS или СУБД (DBMS).

Высокая доступность SAP в Azure отличается от высокой доступности SAP в локальной физической или виртуальной среде. В документе [SAP NetWeaver High Availability and Business Continuity in Virtual Environments with VMware and Hyper-V on Microsoft Windows][sap-ha-bc-virtual-env-hyperv-vmware-white-paper] (Высокая доступность и непрерывность бизнес-процессов SAP NetWeaver в виртуальных средах с VMware и Hyper-V в Microsoft Windows) описываются стандартные конфигурации высокого уровня доступности SAP в виртуализированных средах Windows.

Конфигурации высокой доступности SAP, интегрированной с SAPinst, для Linux (как и для Windows) не существует. Дополнительные сведения об обеспечении высокого уровня доступности SAP в локальной среде Linux см. в статье [с информацией о партнерских решениях для обеспечения высокой доступности][sap-ha-partner-information].

## <a name="azure-infrastructure-high-availability"></a>Высокая доступность инфраструктуры Azure

### <a name="sla-for-single-instance-virtual-machines"></a>Соглашение об уровне обслуживания для одноэкземплярных виртуальных машин

В настоящее время предоставляется соглашение SLA для одной виртуальной машины с временем бесперебойной работы на уровне 99,9 % и хранилищем класса Premium. Чтобы понять, какой может быть доступность одной виртуальной машины, можно выполнить сборку продукта, регулируемого различными доступными [соглашениями об уровне обслуживания Azure][azure-sla].

Для расчета используется период 30 дней в месяц или 43 200 минут. Например, время простоя 0,05 % соответствует 21,6 минутам. Обычно доступность различных служб вычисляется следующим образом:

(Доступность службы 1 / 100) * (доступность службы 2 / 100) * (доступность службы 3 / 100) \*…

Пример:

(99,95 / 100) * (99,9 / 100) * (99,9 / 100) = 0,9975, или общая доступность 99,75 %.

### <a name="multiple-instances-of-virtual-machines-in-the-same-availability-set"></a>Несколько экземпляров виртуальных машин в одной группе доступности
Для виртуальных машин, два и более экземпляра которых развернуты в *той же группе доступности*, мы гарантируем доступность подключения хотя бы к одному экземпляру на уровне не менее 99,95 %.

Когда две или несколько виртуальных машин являются частью одной и той же группы доступности, каждой виртуальной машине в группе доступности платформа Azure назначает *домен обновления* и *домен сбоя*.

* **Домены обновления** гарантируют, что несколько виртуальных машин не будут перезагружаться одновременно при плановом обслуживании инфраструктуры Azure. В одно время перезагружается только одна виртуальная машина.

* **Домены сбоя** гарантируют, что виртуальные машины развернуты на компонентах оборудования, которые не имеют общего источника питания и сетевого коммутатора. Когда в сервере, сетевых коммутаторах или источниках питания возникает незапланированный простой, будет задета только одна виртуальная машина.

Дополнительные сведения см. в статье [Управление доступностью виртуальных машин Windows в Azure][azure-virtual-machines-manage-availability].

Группа доступности используется для достижения высокого уровня доступности для таких компонентов:

* избыточные серверы приложений SAP;  
* кластеры с двумя или более узлами (например, виртуальные машины), которые защищают SPOF, такие как экземпляры ASCS/SCS SAP или СУБД.


### <a name="azure-availability-zones"></a>Зоны доступности Azure
Azure находится в процессе развертывания концепций [Зон доступности Azure](../../../availability-zones/az-overview.md) в разных [регионах Azure](https://azure.microsoft.com/global-infrastructure/regions/). В регионах Azure, где предлагаются Зоны доступности, есть несколько центров обработки данных с независимыми источниками питания, системами охлаждения и сетями. В одном регионе Azure предлагаются разные зоны, чтобы обеспечить возможность развертывать приложения в двух или трех предложенных Зонах доступности. Если неполадки с источниками питания или сетью затрагивают инфраструктуру только одной зоны доступности, развертывание приложения в регионе Azure по-прежнему будет работать нормально. Через время возможно некоторое снижение емкости, так как некоторые виртуальные машины в одной зоне могут быть потеряны. Виртуальные машины в двух других зонах по-прежнему будут работать. Регионы Azure, которые предлагают зоны, перечислены в статье [Что такое Зоны доступности в Azure?](../../../availability-zones/az-overview.md).

При использовании Зон доступности есть несколько моментов, которые следует учесть. Ниже приведен список рекомендаций.

- Вы не можете развертывать группы доступности Azure в Зоне доступности. Вам необходимо выбрать Зону доступности или группу доступности в качестве области развертывания для виртуальной машины.
- Вы не можете использовать [Load Balancer (цен. категория "Базовый")](../../../load-balancer/load-balancer-overview.md) для создания отказоустойчивых кластерных решений, основанных на службах отказоустойчивого кластера Windows или Linux Pacemaker. Вместо этого необходимо использовать [номер SKU Load Balancer (цен. категория "Стандартный") Azure](../../../load-balancer/load-balancer-standard-availability-zones.md).
- Зоны доступности Azure не дают никаких гарантий относительно определенного расстояния между различными зонами в одном регионе.
- Задержка сети между различными Зонами доступности Azure в разных регионах Azure может отличаться. В некоторых случаях вы, как клиент, можете запустить прикладной уровень SAP, развернутый в разных зонах, так как задержка сети от одной зоны до активной виртуальной машины СУБД все еще приемлема из-за влияния бизнес-процесса. При этом будут сценарии клиента, в которых задержка между активной виртуальной машиной СУБД в одной зоне и экземпляром приложения SAP в виртуальной машине в другой зоне может быть слишком значительной и неприемлемой для бизнес-процессов SAP. В результате архитектуры развертывания должны отличаться: архитектура "активная — активная" для приложения или "активная — пассивная", если задержка слишком высока.
- Использование [управляемых дисков Azure](https://azure.microsoft.com/services/managed-disks/) является обязательным для развертывания в Зонах доступности Azure. 


### <a name="planned-and-unplanned-maintenance-of-virtual-machines"></a>Запланированное и незапланированное обслуживание виртуальных машин

Есть два типа событий платформы Azure, которые могут повлиять на доступность виртуальных машин:

* События **планового обслуживания** — это периодические обновления, вносимые корпорацией Майкрософт в базовую платформу Azure. Обновления улучшают общую надежность, производительность и безопасность инфраструктуры платформы, на которой запущена виртуальная машина.

* События **незапланированного обслуживания** происходят в тех случаях, когда в оборудовании или физической инфраструктуре, на основе которых работает виртуальная машина, происходит какая-либо ошибка. Это могут быть сбои локальной сети или локальных жестких дисков, а также другие ошибки на уровне стойки. При выявлении такой ошибки платформа Azure автоматически выполнит перенос вашей виртуальной машины с неработоспособного физического сервера, где она размещена, на исправный. Это происходит редко, но также может быть причиной перезагрузки вашей виртуальной машины.

Дополнительные сведения см. в статье [Управление доступностью виртуальных машин Windows в Azure][azure-virtual-machines-manage-availability].

### <a name="azure-storage-redundancy"></a>Избыточность хранилища Azure
Данные в учетной записи хранения всегда реплицируются, обеспечивая устойчивость, высокий уровень доступности и соответствие соглашению об уровне обслуживания для службы хранилища Azure даже при временных сбоях оборудования.

Так как служба хранилища Azure сохраняет три образа данных по умолчанию, использование RAID 5 или RAID 1 на нескольких дисках Azure не требуется.

Дополнительные сведения см. в статье [Репликация службы хранилища Azure][azure-storage-redundancy].

### <a name="azure-managed-disks"></a>Управляемые диски Azure
Управляемые диски — это тип ресурса в Azure Resource Manager, который рекомендуется использовать вместо виртуальных жестких дисков, хранимых в учетных записях хранения Azure. Управляемые диски автоматически соответствуют группе доступности Azure виртуальной машины, к которой они присоединены. Они повышают ее доступность и доступность служб, выполняющихся на ней.

Дополнительные сведения об Управляемых дисках Azure см. в статье [Обзор компонента "Управляемые диски" Azure][azure-storage-managed-disks-overview].

Мы советуем использовать управляемые диски, так как они упрощают развертывание виртуальных машин и управление ими.



## <a name="utilizing-azure-infrastructure-high-availability-to-achieve-higher-availability-of-sap-applications"></a>Использование высокой доступности инфраструктуры Azure для повышения *уровня доступности* приложений SAP

Если вы решили не применять такие функции, как отказоустойчивый кластер Windows Server (WSFC) или Pacemaker в Linux (в настоящее время поддерживается только для SUSE Linux Enterprise Server (SLES) 12 или более поздней версии), используйте перезапуск виртуальной машины. Это защищает системы SAP от запланированных и незапланированных простоев инфраструктуры физического сервера Azure и всей базовой платформы Azure.

Дополнительные сведения об этом подходе см. в статье [Использование перезапуска виртуальной машины в инфраструктуре Azure для повышения доступности системы SAP][sap-higher-availability].

## <a name="high-availability-of-sap-applications-on-azure-iaas"></a><a name="baed0eb3-c662-4405-b114-24c10a62954e"></a>Высокая доступность приложений SAP в Azure IaaS

Для обеспечения высокой доступности всей системы SAP необходимо защитить все ее важные компоненты. Пример:
  * избыточные серверы приложений SAP;
  * уникальные компоненты. Примером может служить компонент единой точки отказа (SPOF), такой как экземпляр ASCS/SCS или СУБД (DBMS).

В следующем разделе описано, как добиться высокой доступности для всех трех важных компонентов системы SAP.

### <a name="high-availability-architecture-for-sap-application-servers"></a>Архитектура высокого уровня доступности для серверов приложений SAP

> Этот раздел касается:
>
> ![Логотип Windows.][Logo_Windows] Windows и ![Логотип Linux.][Logo_Linux] Linux
>

Как правило, для экземпляров серверов приложений или диалоговых экземпляров SAP специальное решение для обеспечения высокой доступности не требуется. Высокий уровень доступности достигается за счет избыточности, то есть на разных экземплярах виртуальных машин Azure настраиваются различные диалоговые экземпляры. Необходимо установить как минимум два экземпляра приложения SAP на два экземпляра виртуальных машин Azure.

![Рисунок 1. Сервер приложений SAP с высоким уровнем доступности][sap-ha-guide-figure-2000]

_**Рис. 1.** Сервер приложений SAP с высоким уровнем доступности_

Все виртуальные машины, на которых размещены экземпляры серверов приложений SAP, должны находиться в одной группе доступности Azure. Группа доступности Azure гарантирует следующее.

* Все виртуальные машины не входят в один и тот же домен обновления.  
    Домен обновления гарантирует, что виртуальные машины могут избежать одновременного обновления во время планового обслуживания.

    Основные функциональные возможности, основанные на разделении доменов обновления и доменов сбоя в рамках единицы масштабирования Azure, уже были описаны выше в разделе [Домены обновления][planning-guide-3.2.2].

* Все виртуальные машины не входят в один и тот же домен сбоя.  
    Домен сбоя позволяет избежать развертывания виртуальных машин таким образом, что единая точка отказа сможет влиять на доступность всех виртуальных машин.

Количество доменов сбоя и обновления, которые можно использовать в группе доступности Azure в единице масштабирования Azure, не бесконечно. Если продолжить добавлять виртуальные машины в одну группу доступности, две или несколько виртуальных машин в конечном итоге окажутся в том же домене сбоя или обновления.

Если развернуть несколько экземпляров сервера приложений SAP на выделенных виртуальных машинах и предположить, что у нас есть пять доменов обновления, в конце концов сложится следующая ситуация. Фактическое максимальное число доменов обновления и сбоя в группе доступности в будущем может измениться:

![Рис. 2. Высокий уровень доступности серверов приложений SAP в группе доступности Azure][planning-guide-figure-3000]
_**Рис. 2.** Высокий уровень доступности серверов приложений SAP в группе доступности Azure_

Дополнительные сведения см. в статье [Управление доступностью виртуальных машин Windows в Azure][azure-virtual-machines-manage-availability].

Дополнительные сведения см. в разделе [Группы доступности Azure][planning-guide-3.2.3] документа о планировании и внедрении SAP NetWeaver на виртуальных машинах Azure.

**Только неуправляемые диски.** Так как учетная запись хранения Azure является потенциальной единственной точкой отказа, важно иметь по меньшей мере две учетные записи хранения Azure, в которых будут распределены как минимум две виртуальные машины. В идеале все диски виртуальных машин, на которых выполняются диалоговые экземпляры SAP, следует развертывать в разных учетных записях хранения.

> [!IMPORTANT]
> Мы рекомендуем использовать управляемые диски Azure для установок приложений SAP высокой доступности. Так как управляемые диски автоматически соответствуют группам доступности виртуальной машины, к которой они подключены, они повышают ее доступность и доступность служб, выполняющихся на ней.  
>

### <a name="high-availability-architecture-for-an-sap-ascsscs-instance-on-windows"></a>Архитектура высокой доступности для экземпляра SAP ASCS/SCS на Windows

> ![Логотип Windows.][Logo_Windows] Windows
>

Решение WSFC можно использовать для защиты экземпляра SAP ASCS/SCS. Решение содержит два варианта:

* **Кластеризация экземпляра SAP ASCS/SCS с использованием кластерных общих дисков**. Дополнительные сведения об этой архитектуре см. в статье [Кластеризация экземпляра SAP ASCS/SCS в отказоустойчивом кластере Windows с помощью общего диска кластера в Azure][sap-high-availability-guide-wsfc-shared-disk].   

* **Кластеризация экземпляра SAP ASCS/SCS с использованием общей папки**. Дополнительные сведения об этой архитектуре см. в статье [Кластеризация экземпляра SAP ASCS/SCS в отказоустойчивом кластере Windows с помощью общей папки в Azure][sap-high-availability-guide-wsfc-file-share].

* **Кластеризация экземпляра SAP ASCS/SCS с использованием общей папки ANF SMB**. Дополнительные сведения об этой архитектуре см. в статье [Кластеризация экземпляра SAP ASCS/SCS в отказоустойчивом кластере Windows с помощью общей папки ANF SMB](./high-availability-guide-windows-netapp-files-smb.md).

### <a name="high-availability-architecture-for-an-sap-ascsscs-instance-on-linux"></a>Архитектура высокой доступности для экземпляра SAP ASCS/SCS на Linux

> ![Логотип Linux.][Logo_Linux] Linux
> 
> Дополнительные сведения о кластеризации экземпляра SAP ASCS/SCS с помощью платформы кластера SLES см. в статье [Обеспечение высокого уровня доступности SAP NetWeaver на виртуальных машинах Azure в SUSE Linux Enterprise Server для приложений SAP][sap-suse-ascs-ha]. Сведения об альтернативной высокодоступной архитектуре в SLES, для которой не требуется высокодоступная NFS, см. в статье [Обеспечение высокого уровня доступности SAP NetWeaver в SUSE Linux Enterprise Server для приложений SAP с помощью Azure NetApp Files][sap-suse-ascs-ha-anf].

Дополнительные сведения о кластеризации экземпляра SAP ASCS/SCS с помощью платформы кластера Red Hat см. в статье [Обеспечение высокого уровня доступности SAP NetWeaver в виртуальных машинах Azure с Red Hat Enterprise Linux](./high-availability-guide-rhel.md).


### <a name="sap-netweaver-multi-sid-configuration-for-a-clustered-sap-ascsscs-instance"></a>Конфигурации SAP NetWeaver с несколькими идентификаторами безопасности для кластеризованного экземпляра SAP ASCS/SCS

> ![Логотип Windows.][Logo_Windows] Windows
> 
> Несколько идентификаторов безопасности поддерживается c WSFC при использовании общей папки и общего диска.
> 
> Дополнительные сведения об использовании архитектуры высокого уровня доступности с несколькими идентификаторами безопасности в Windows см. в следующих статьях:

* [Обеспечение высокого уровня доступности экземпляра SAP ASCS/SCS с несколькими ИД безопасности с помощью отказоустойчивой кластеризации Windows Server и общей папки в Azure][sap-ascs-ha-multi-sid-wsfc-file-share]

* [Обеспечение высокого уровня доступности с несколькими ИД безопасности для экземпляра SAP ASCS/SCS с помощью отказоустойчивой кластеризации Windows Server и общего диска в Azure][sap-ascs-ha-multi-sid-wsfc-shared-disk]

> ![Логотип Linux.][Logo_Linux] Linux
> 
> Кластеризация с несколькими ИД безопасности поддерживается в кластерах Pacemaker Linux для SAP ASCS/ERS, но в одном кластере может быть только **пять** идентификаторов безопасности SAP.
> Дополнительные сведения об использовании архитектуры высокого уровня доступности с несколькими идентификаторами безопасности в Linux см. в следующих статьях:

* [Обеспечение высокого уровня доступности SAP NW на виртуальных машинах Azure в SLES с несколькими ИД безопасности для приложений SAP](./high-availability-guide-suse-multi-sid.md)
* [Обеспечение высокого уровня доступности SAP NW на виртуальных машинах Azure в RHEL с несколькими ИД безопасности для приложений SAP](./high-availability-guide-rhel-multi-sid.md)

### <a name="high-availability-dbms-instance"></a>Экземпляр СУБД высокого уровня доступности

СУБД также является единой точкой отказа системы SAP. Она должна быть защищена с помощью решения для обеспечения высокого уровня доступности. На следующем рисунке показан пример решения для обеспечения высокого уровня доступности SQL Server AlwaysOn в Azure, в котором используется отказоустойчивый кластер Windows Server и внутренний балансировщик нагрузки Azure. SQL Server AlwaysOn реплицирует файлы данных и журналов СУБД с помощью собственной функции репликации СУБД. Поэтому вам не требуется общий диск кластера, что упрощает всю конфигурацию.

![Рис. 3. Пример высокодоступной СУБД SAP с SQL Server Always On][sap-ha-guide-figure-2003]

_**Рис. 3.** Пример высокодоступной СУБД SAP с SQL Server Always On_

Дополнительные сведения о кластеризации СУБД SQL Server в Azure посредством модели развертывания с помощью Azure Resource Manager можно найти в следующих статьях:

* [Введение в группы доступности Always On SQL Server на виртуальных машинах Azure][virtual-machines-windows-portal-sql-alwayson-availability-groups-manual]

* [Настройка подсистемы балансировки нагрузки для группы доступности Always On в Azure][virtual-machines-windows-portal-sql-alwayson-int-listener]

Дополнительные сведения о кластеризации СУБД SAP HANA в Azure с использованием модели развертывания с помощью Azure Resource Manager см. в статье [Обеспечение высокого уровня доступности SAP HANA на виртуальных машинах Azure в SUSE Linux Enterprise Server][sap-hana-ha].
