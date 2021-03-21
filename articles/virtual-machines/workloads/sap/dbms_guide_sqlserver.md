---
title: Рабочие нагрузки SAP на виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server | Документы Майкрософт
description: Рабочие нагрузки SAP на Виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: Azure, SQL Server, SAP, AlwaysOn
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/20/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 4eb7e64065e311dc18f33dffb169d5c27a34008d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101673043"
---
# <a name="sql-server-azure-virtual-machines-dbms-deployment-for-sap-netweaver"></a>SAP NetWeaver на виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server

[767598]:https://launchpad.support.sap.com/#/notes/767598
[773830]:https://launchpad.support.sap.com/#/notes/773830
[826037]:https://launchpad.support.sap.com/#/notes/826037
[965908]:https://launchpad.support.sap.com/#/notes/965908
[1031096]:https://launchpad.support.sap.com/#/notes/1031096
[1114181]:https://launchpad.support.sap.com/#/notes/1114181
[1139904]:https://launchpad.support.sap.com/#/notes/1139904
[1173395]:https://launchpad.support.sap.com/#/notes/1173395
[1245200]:https://launchpad.support.sap.com/#/notes/1245200
[1409604]:https://launchpad.support.sap.com/#/notes/1409604
[1558958]:https://launchpad.support.sap.com/#/notes/1558958
[1585981]:https://launchpad.support.sap.com/#/notes/1585981
[1588316]:https://launchpad.support.sap.com/#/notes/1588316
[1590719]:https://launchpad.support.sap.com/#/notes/1590719
[1597355]:https://launchpad.support.sap.com/#/notes/1597355
[1605680]:https://launchpad.support.sap.com/#/notes/1605680
[1619720]:https://launchpad.support.sap.com/#/notes/1619720
[1619726]:https://launchpad.support.sap.com/#/notes/1619726
[1619967]:https://launchpad.support.sap.com/#/notes/1619967
[1750510]:https://launchpad.support.sap.com/#/notes/1750510
[1752266]:https://launchpad.support.sap.com/#/notes/1752266
[1757924]:https://launchpad.support.sap.com/#/notes/1757924
[1757928]:https://launchpad.support.sap.com/#/notes/1757928
[1758182]:https://launchpad.support.sap.com/#/notes/1758182
[1758496]:https://launchpad.support.sap.com/#/notes/1758496
[1772688]:https://launchpad.support.sap.com/#/notes/1772688
[1814258]:https://launchpad.support.sap.com/#/notes/1814258
[1882376]:https://launchpad.support.sap.com/#/notes/1882376
[1909114]:https://launchpad.support.sap.com/#/notes/1909114
[1922555]:https://launchpad.support.sap.com/#/notes/1922555
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1941500]:https://launchpad.support.sap.com/#/notes/1941500
[1956005]:https://launchpad.support.sap.com/#/notes/1956005
[1973241]:https://launchpad.support.sap.com/#/notes/1973241
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2039619]:https://launchpad.support.sap.com/#/notes/2039619
[2069760]:https://launchpad.support.sap.com/#/notes/2069760
[2121797]:https://launchpad.support.sap.com/#/notes/2121797
[2134316]:https://launchpad.support.sap.com/#/notes/2134316
[2171857]:https://launchpad.support.sap.com/#/notes/2171857
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[azure-cli]:../../../cli-install-nodejs.md
[azure-portal]:https://portal.azure.com
[azure-ps]:/powershell/azure/
[azure-quickstart-templates-github]:https://github.com/Azure/azure-quickstart-templates
[azure-script-ps]:https://go.microsoft.com/fwlink/p/?LinkID=395017
[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md#subscription-limits

[dbms-guide]:dbms-guide_general.md 
[dbms-guide-2.1]:dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f 
[dbms-guide-2.2]:dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91 
[dbms-guide-2.3]:dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152 
[dbms-guide-2]:dbms-guide.md#65fa79d6-a85f-47ee-890b-22e794f51a64 
[dbms-guide-3]:dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3 
[dbms-guide-5.5.1]:dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268 
[dbms-guide-5.5.2]:dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b 
[dbms-guide-5.6]:dbms_guide_sqlserver.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8 
[dbms-guide-5.8]:dbms-guide.md#9053f720-6f3b-4483-904d-15dc54141e30 
[dbms-guide-5]:dbms-guide.md#3264829e-075e-4d25-966e-a49dad878737 
[dbms-guide-8.4.1]:dbms-guide.md#b48cfe3b-48e9-4f5b-a783-1d29155bd573 
[dbms-guide-8.4.2]:dbms-guide.md#23c78d3b-ca5a-4e72-8a24-645d141a3f5d 
[dbms-guide-8.4.3]:dbms-guide.md#77cd2fbb-307e-4cbf-a65f-745553f72d2c 
[dbms-guide-8.4.4]:dbms-guide.md#f77c1436-9ad8-44fb-a331-8671342de818 
[dbms-guide-900-sap-cache-server-on-premises]:dbms-guide.md#642f746c-e4d4-489d-bf63-73e80177a0a8
[dbms-guide-managed-disks]:dbms-guide.md#f42c6cb5-d563-484d-9667-b07ae51bce29

[dbms-guide-figure-100]:media/virtual-machines-shared-sap-dbms-guide/100_storage_account_types.png
[dbms-guide-figure-200]:media/virtual-machines-shared-sap-dbms-guide/200-ha-set-for-dbms-ha.png
[dbms-guide-figure-300]:media/virtual-machines-shared-sap-dbms-guide/300-reference-config-iaas.png
[dbms-guide-figure-400]:media/virtual-machines-shared-sap-dbms-guide/400-sql-2012-backup-to-blob-storage.png
[dbms-guide-figure-500]:media/virtual-machines-shared-sap-dbms-guide/500-sql-2012-backup-to-blob-storage-different-containers.png
[dbms-guide-figure-600]:media/virtual-machines-shared-sap-dbms-guide/600-iaas-maxdb.png
[dbms-guide-figure-700]:media/virtual-machines-shared-sap-dbms-guide/700-livecach-prod.png
[dbms-guide-figure-800]:media/virtual-machines-shared-sap-dbms-guide/800-azure-vm-sap-content-server.png
[dbms-guide-figure-900]:media/virtual-machines-shared-sap-dbms-guide/900-sap-cache-server-on-premises.png

[deployment-guide]:deployment-guide.md 
[deployment-guide-2.2]:deployment-guide.md#42ee2bdb-1efc-4ec7-ab31-fe4c22769b94 
[deployment-guide-3.1.2]:deployment-guide.md#3688666f-281f-425b-a312-a77e7db2dfab 
[deployment-guide-3.2]:deployment-guide.md#db477013-9060-4602-9ad4-b0316f8bb281 
[deployment-guide-3.3]:deployment-guide.md#54a1fc6d-24fd-4feb-9c57-ac588a55dff2 
[deployment-guide-3.4]:deployment-guide.md#a9a60133-a763-4de8-8986-ac0fa33aa8c1 
[deployment-guide-3]:deployment-guide.md#b3253ee3-d63b-4d74-a49b-185e76c4088e 
[deployment-guide-4.1]:deployment-guide.md#604bcec2-8b6e-48d2-a944-61b0f5dee2f7 
[deployment-guide-4.2]:deployment-guide.md#7ccf6c3e-97ae-4a7a-9c75-e82c37beb18e 
[deployment-guide-4.3]:deployment-guide.md#31d9ecd6-b136-4c73-b61e-da4a29bbc9cc 
[deployment-guide-4.4.2]:deployment-guide.md#6889ff12-eaaf-4f3c-97e1-7c9edc7f7542 
[deployment-guide-4.4]:deployment-guide.md#c7cbb0dc-52a4-49db-8e03-83e7edc2927d 
[deployment-guide-4.5.1]:deployment-guide.md#987cf279-d713-4b4c-8143-6b11589bb9d4 
[deployment-guide-4.5.2]:deployment-guide.md#408f3779-f422-4413-82f8-c57a23b4fc2f 
[deployment-guide-4.5]:deployment-guide.md#d98edcd3-f2a1-49f7-b26a-07448ceb60ca 
[deployment-guide-5.1]:deployment-guide.md#bb61ce92-8c5c-461f-8c53-39f5e5ed91f2 
[deployment-guide-5.2]:deployment-guide.md#e2d592ff-b4ea-4a53-a91a-e5521edb6cd1
[deployment-guide-5.3]:deployment-guide.md#fe25a7da-4e4e-4388-8907-8abc2d33cfd8 

[deployment-guide-configure-monitoring-scenario-1]:deployment-guide.md#ec323ac3-1de9-4c3a-b770-4ff701def65b 
[deployment-guide-configure-proxy]:deployment-guide.md#baccae00-6f79-4307-ade4-40292ce4e02d 
[deployment-guide-figure-100]:media/virtual-machines-shared-sap-deployment-guide/100-deploy-vm-image.png
[deployment-guide-figure-1000]:media/virtual-machines-shared-sap-deployment-guide/1000-service-properties.png
[deployment-guide-figure-11]:deployment-guide.md#figure-11
[deployment-guide-figure-1100]:media/virtual-machines-shared-sap-deployment-guide/1100-azperflib.png
[deployment-guide-figure-1200]:media/virtual-machines-shared-sap-deployment-guide/1200-cmd-test-login.png
[deployment-guide-figure-1300]:media/virtual-machines-shared-sap-deployment-guide/1300-cmd-test-executed.png
[deployment-guide-figure-14]:deployment-guide.md#figure-14
[deployment-guide-figure-1400]:media/virtual-machines-shared-sap-deployment-guide/1400-azperflib-error-servicenotstarted.png
[deployment-guide-figure-300]:media/virtual-machines-shared-sap-deployment-guide/300-deploy-private-image.png
[deployment-guide-figure-400]:media/virtual-machines-shared-sap-deployment-guide/400-deploy-using-disk.png
[deployment-guide-figure-5]:deployment-guide.md#figure-5
[deployment-guide-figure-50]:media/virtual-machines-shared-sap-deployment-guide/50-forced-tunneling-suse.png
[deployment-guide-figure-500]:media/virtual-machines-shared-sap-deployment-guide/500-install-powershell.png
[deployment-guide-figure-6]:deployment-guide.md#figure-6
[deployment-guide-figure-600]:media/virtual-machines-shared-sap-deployment-guide/600-powershell-version.png
[deployment-guide-figure-7]:deployment-guide.md#figure-7
[deployment-guide-figure-700]:media/virtual-machines-shared-sap-deployment-guide/700-install-powershell-installed.png
[deployment-guide-figure-760]:media/virtual-machines-shared-sap-deployment-guide/760-azure-cli-version.png
[deployment-guide-figure-900]:media/virtual-machines-shared-sap-deployment-guide/900-cmd-update-executed.png
[deployment-guide-figure-azure-cli-installed]:deployment-guide.md#402488e5-f9bb-4b29-8063-1c5f52a892d0
[deployment-guide-figure-azure-cli-version]:deployment-guide.md#0ad010e6-f9b5-4c21-9c09-bb2e5efb3fda
[deployment-guide-install-vm-agent-windows]:deployment-guide.md#b2db5c9a-a076-42c6-9835-16945868e866
[deployment-guide-troubleshooting-chapter]:deployment-guide.md#564adb4f-5c95-4041-9616-6635e83a810b

[deploy-template-cli]:../../../resource-group-template-deploy-cli.md
[deploy-template-portal]:../../../resource-group-template-deploy-portal.md
[deploy-template-powershell]:../../../resource-group-template-deploy.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md
[getting-started-dbms]:get-started.md#1343ffe1-8021-4ce6-a08d-3a1553a4db82
[getting-started-deployment]:get-started.md#6aadadd2-76b5-46d8-8713-e8d63630e955
[getting-started-planning]:get-started.md#3da0389e-708b-4e82-b2a2-e92f132df89c

[getting-started-windows-classic]:../../virtual-machines-windows-classic-sap-get-started.md
[getting-started-windows-classic-dbms]:../../virtual-machines-windows-classic-sap-get-started.md#c5b77a14-f6b4-44e9-acab-4d28ff72a930
[getting-started-windows-classic-deployment]:../../virtual-machines-windows-classic-sap-get-started.md#f84ea6ce-bbb4-41f7-9965-34d31b0098ea
[getting-started-windows-classic-dr]:../../virtual-machines-windows-classic-sap-get-started.md#cff10b4a-01a5-4dc3-94b6-afb8e55757d3
[getting-started-windows-classic-ha-sios]:../../virtual-machines-windows-classic-sap-get-started.md#4bb7512c-0fa0-4227-9853-4004281b1037
[getting-started-windows-classic-planning]:../../virtual-machines-windows-classic-sap-get-started.md#f2a5e9d8-49e4-419e-9900-af783173481c

[ha-guide-classic]:https://go.microsoft.com/fwlink/?LinkId=613056

[install-extension-cli]:virtual-machines-linux-enable-aem.md

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png

[msdn-set-azurermvmaemextension]:https://msdn.microsoft.com/library/azure/mt670598.aspx

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
[planning-guide-figure-2500]:media/virtual-machines-shared-sap-planning-guide/2500-vm-extension-details.png
[planning-guide-figure-2600]:media/virtual-machines-shared-sap-planning-guide/2600-sap-router-connection.png
[planning-guide-figure-2700]:media/virtual-machines-shared-sap-planning-guide/2700-exposed-sap-portal.png
[planning-guide-figure-2800]:media/virtual-machines-shared-sap-planning-guide/2800-endpoint-config.png
[planning-guide-figure-2900]:media/virtual-machines-shared-sap-planning-guide/2900-azure-ha-sap-ha.png
[planning-guide-figure-300]:media/virtual-machines-shared-sap-planning-guide/300-vpn-s2s.png
[planning-guide-figure-3000]:media/virtual-machines-shared-sap-planning-guide/3000-sap-ha-on-azure.png
[planning-guide-figure-3200]:media/virtual-machines-shared-sap-planning-guide/3200-sap-ha-with-sql.png
[planning-guide-figure-400]:media/virtual-machines-shared-sap-planning-guide/400-vm-services.png
[planning-guide-figure-600]:media/virtual-machines-shared-sap-planning-guide/600-s2s-details.png
[planning-guide-figure-700]:media/virtual-machines-shared-sap-planning-guide/700-decision-tree-deploy-to-azure.png
[planning-guide-figure-800]:media/virtual-machines-shared-sap-planning-guide/800-portal-vm-overview.png
[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd 
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f 

[resource-group-authoring-templates]:../../../resource-group-authoring-templates.md
[resource-group-overview]:../../../azure-resource-manager/management/overview.md
[resource-groups-networking]:../../../networking/networking-overview.md
[sap-pam]:https://support.sap.com/pam 
[sap-templates-2-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-2-tier-os-disk]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk%2Fazuredeploy.json
[sap-templates-2-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-3-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image%2Fazuredeploy.json
[storage-azure-cli]:../../../storage/common/storage-azure-cli.md
[storage-azure-cli-copy-blobs]:../../../storage/common/storage-azure-cli.md#copy-blobs
[storage-introduction]:../../../storage/common/storage-introduction.md
[storage-powershell-guide-full-copy-vhd]:../../../storage/common/storage-powershell-guide-full.md#how-to-copy-blobs-from-one-storage-container-to-another
[storage-premium-storage-preview-portal]:../../disks-types.md
[storage-redundancy]:../../../storage/common/storage-redundancy.md
[storage-scalability-targets]:../../../storage/common/scalability-targets-standard-accounts.md
[storage-use-azcopy]:../../../storage/common/storage-use-azcopy.md
[template-201-vm-from-specialized-vhd]:https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-from-specialized-vhd
[templates-101-simple-windows-vm]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-simple-windows-vm
[templates-101-vm-from-user-image]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image
[virtual-machines-linux-attach-disk-portal]:../../linux/attach-disk-portal.md
[virtual-machines-azure-resource-manager-architecture]:../../../resource-manager-deployment-model.md
[virtual-machines-azurerm-versus-azuresm]:../../../resource-manager-deployment-model.md
[virtual-machines-windows-classic-configure-oracle-data-guard]:../../virtual-machines-windows-classic-configure-oracle-data-guard.md
[virtual-machines-linux-cli-deploy-templates]:../../linux/cli-deploy-templates.md 
[virtual-machines-deploy-rmtemplates-powershell]:../../virtual-machines-windows-ps-manage.md 
[virtual-machines-linux-agent-user-guide]:../../linux/agent-user-guide.md
[virtual-machines-linux-agent-user-guide-command-line-options]:../../linux/agent-user-guide.md#command-line-options
[virtual-machines-linux-capture-image]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager-capture]:../../linux/capture-image.md#step-2-capture-the-vm
[virtual-machines-linux-configure-raid]:../../linux/configure-raid.md
[virtual-machines-linux-configure-lvm]:../../linux/configure-lvm.md
[virtual-machines-linux-classic-create-upload-vhd-step-1]:../../virtual-machines-linux-classic-create-upload-vhd.md#step-1-prepare-the-image-to-be-uploaded
[virtual-machines-linux-create-upload-vhd-suse]:../../linux/suse-create-upload-vhd.md
[virtual-machines-linux-redhat-create-upload-vhd]:../../linux/redhat-create-upload-vhd.md
[virtual-machines-linux-how-to-attach-disk]:../../linux/add-disk.md
[virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux]:../../linux/add-disk.md#connect-to-the-linux-vm-to-mount-the-new-disk
[virtual-machines-linux-tutorial]:../../linux/quick-create-cli.md
[virtual-machines-linux-update-agent]:../../linux/update-agent.md
[virtual-machines-manage-availability-linux]:../../linux/manage-availability.md
[virtual-machines-manage-availability-windows]:../../windows/manage-availability.md
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]:virtual-machines-windows-create-powershell.md
[virtual-machines-sizes-linux]:../../linux/sizes.md
[virtual-machines-sizes-windows]:../../windows/sizes.md
[virtual-machines-windows-classic-ps-sql-alwayson-availability-groups]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups.md
[virtual-machines-windows-classic-ps-sql-int-listener]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener.md
[virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions]:../../../azure-sql/virtual-machines/windows/business-continuity-high-availability-disaster-recovery-hadr-overview.md
[virtual-machines-sql-server-infrastructure-services]:../../../azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md
[virtual-machines-sql-server-performance-best-practices]:../../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md
[virtual-machines-upload-image-windows-resource-manager]:../../virtual-machines-windows-upload-image.md
[virtual-machines-windows-tutorial]:../../virtual-machines-windows-hero-tutorial.md
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/eresources/templates/sql-server-2014-alwayson-existing-vnet-and-ad/
[virtual-network-deploy-multinic-arm-cli]:../linux/multiple-nics.md
[virtual-network-deploy-multinic-arm-ps]:../windows/multiple-nics.md
[virtual-network-deploy-multinic-arm-template]:../../../virtual-network/template-samples.md
[virtual-networks-configure-vnet-to-vnet-connection]:../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md
[virtual-networks-create-vnet-arm-pportal]:../../../virtual-network/manage-virtual-network.md#create-a-virtual-network
[virtual-networks-manage-dns-in-vnet]:../../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md
[virtual-networks-multiple-nics]:../../../virtual-network/virtual-network-deploy-multinic-classic-ps.md
[virtual-networks-nsg]:../../../virtual-network/security-overview.md
[virtual-networks-reserved-private-ip]:../../../virtual-network/virtual-networks-static-private-ip-arm-ps.md
[virtual-networks-static-private-ip-arm-pportal]:../../../virtual-network/virtual-networks-static-private-ip-arm-pportal.md
[virtual-networks-udr-overview]:../../../virtual-network/virtual-networks-udr-overview.md
[vpn-gateway-about-vpn-devices]:../../../vpn-gateway/vpn-gateway-about-vpn-devices.md
[vpn-gateway-create-site-to-site-rm-powershell]:../../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md
[vpn-gateway-cross-premises-options]:../../../vpn-gateway/vpn-gateway-plan-design.md
[vpn-gateway-site-to-site-create]:../../../vpn-gateway/vpn-gateway-site-to-site-create.md
[vpn-gateway-vpn-faq]:../../../vpn-gateway/vpn-gateway-vpn-faq.md
[xplat-cli]:../../../cli-install-nodejs.md
[xplat-cli-azure-resource-manager]:../../../xplat-cli-azure-resource-manager.md




В этом документе рассматривается несколько аспектов, которые следует учитывать при развертывании SQL Server для рабочей нагрузки SAP в Azure IaaS. Перед чтением этого документа следует ознакомиться с документом [Вопросы развертывания СУБД для рабочей нагрузки SAP на виртуальных машинах Azure](./dbms_guide_general.md), а также с другими руководствами в [документации по рабочей нагрузке SAP в Azure](./get-started.md). 



> [!IMPORTANT]
> Этот документ относится к версии SQL Server на Windows. SAP не поддерживает версию SQL Server на Linux с программным обеспечением SAP. В этом документе не обсуждается База данных SQL Microsoft Azure, которая представляет собой предложение в формате "платформа как услуга" на платформе Microsoft Azure. В этом документе рассматривается запуск продукта SQL Server в том виде, в каком он традиционно используется для локальных развертываний на виртуальных машинах Azure, с использованием функций инфраструктуры как услуги в Azure. Возможности и функции базы данных между этими двумя предложениями различаются и не должны смешиваться друг с другом. Дополнительные сведения см. по ссылке <https://azure.microsoft.com/services/sql-database/>
> 
>

Как правило, рекомендуется использовать последние выпуски SQL Server для выполнения рабочей нагрузки SAP в Azure IaaS. Последние выпуски SQL Server обеспечивают лучшую интеграцию с некоторыми службами и функциями Azure. Или содержат изменения для оптимизации работы в инфраструктуре Azure IaaS.

Перед продолжением рекомендуется ознакомиться с статьей [что такое SQL Server на виртуальных машинах Azure (Windows)] [ https://docs.microsoft.com/azure/azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview ].

В следующих разделах упоминаются различные фрагменты документации по указанной выше ссылке. Также упоминаются особенности, связанные с SAP. Некоторые основные понятия описаны более подробно. Однако настоятельно рекомендуем сначала поработать с упомянутой выше документацией, прежде чем приступать к чтению документации, касающейся SQL Server.

Существуют некоторые специальные сведения об SQL Server в IaaS, которые следует знать перед продолжением:

* **Поддержка версий SQL**. Для клиентов SAP на виртуальной машине Microsoft Azure поддерживается SQL Server 2008 R2 и более поздних версий. Более ранние выпуски не поддерживаются. Дополнительные сведения см. в этом общем [заявлении о поддержке](https://support.microsoft.com/kb/956893). В целом SQL Server 2008 также поддерживается корпорацией Майкрософт. Однако в SQL Server 2008 R2 были представлены важные функциональные возможности для SAP, поэтому SQL Server 2008 R2 является минимальной поддерживаемой версией для SAP. Как правило, рекомендуется использовать последние выпуски SQL Server для выполнения рабочей нагрузки SAP в Azure IaaS. Последние выпуски SQL Server обеспечивают лучшую интеграцию с некоторыми службами и функциями Azure. Или содержат изменения для оптимизации работы в инфраструктуре Azure IaaS. Таким образом, этот документ относится только к SQL Server 2016 и SQL Server 2017.
* **Производительность SQL**. Размещенные в Microsoft Azure виртуальные машины работают хорошо по сравнению с другими предложениями виртуализации в общедоступном облаке, но некоторые результаты могут отличаться. Ознакомьтесь с нашей статьей [Рекомендации по оптимизации производительности SQL Server на виртуальных машинах Azure](../../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md).
* **Использование образов из Azure Marketplace**. Самый быстрый способ развернуть новую виртуальную машину Microsoft Azure — это использовать образ из Azure Marketplace. В Azure Marketplace имеются образы, содержащие последние выпуски SQL Server. Образы, в которых уже установлен SQL Server, нельзя сразу же использовать для приложений SAP NetWeaver. Причина в том, что в таких образах заданы параметры сортировки по умолчанию для SQL Server, которые отличаются от параметров, требующихся для систем SAP NetWeaver. Чтобы использовать такие образы, выполните действия, описанные в разделе [Использование образов SQL Server из Microsoft Azure Marketplace][dbms-guide-5.6]. 


## <a name="recommendations-on-vmvhd-structure-for-sap-related-sql-server-deployments"></a>Рекомендации по структуре виртуальных машин и виртуальных жестких дисков для связанных с SAP развертываний SQL Server
В соответствии с общим описанием, операционной системой, SQL Server исполняемыми файлами и в случае систем SAP 2-уровня, исполняемые файлы SAP должны быть расположены или установлены на отдельных дисках Azure. Как правило, большая часть системных баз данных SQL Server не используется на высоком уровне рабочей нагрузкой SAP NetWeaver. Однако системные базы данных SQL Server (Master, msdb и Model) должны быть вместе с другими каталогами SQL Server на отдельном диске Azure. SQL Server база данных tempdb должна находиться на нонперисистед D:\ диск или на отдельном диске.


* Для всех типов виртуальных машин с сертификацией SAP (см. примечание о SAP [1928533]), за исключением виртуальных машин серии A, данные tempdb и файлы журнала можно поместить на несохраняемом диске D:\. 
* Для более старых выпусков SQL Server, где SQL Server устанавливает базу данных tempdb по умолчанию, рекомендуется использовать несколько файлов данных tempdb. Учтите, что тома диска D:\ различаются в зависимости от типа виртуальной машины. Точный размер диска D:\ разных виртуальных машин см. в статье [Размеры виртуальных машин Windows в Azure](../../sizes.md).

Эти конфигурации позволяют базе данных tempdb потреблять больше пространства и более важных операций ввода-вывода и пропускной способности хранилища, чем может предоставить системный диск. Непостоянная D:\ диск также обеспечивает лучшую задержку операций ввода-вывода и пропускную способность (за исключением виртуальных машин серии A). Чтобы определить необходимый размер tempdb, можно проверить размеры tempdb в существующих системах. 

>[!NOTE]
> Если вы помещаете файлы журнала и файлы данных tempdb в созданную вами папку на диске D:\, убедитесь, что эта папка существует после перезагрузки виртуальной машины. Так как диск D:\ инициализируется заново после перезагрузки виртуальной машины, все структуры каталогов и файлов стираются. Возможность воссоздавать итоговые структуры каталогов на диске D:\ до запуска службы SQL Server описана в [этой статье](https://cloudblogs.microsoft.com/sqlserver/2014/09/25/using-ssds-in-azure-vms-to-store-sql-server-tempdb-and-buffer-pool-extensions/).

Конфигурация виртуальной машины, на которой выполняется SQL Server с базой данных SAP, а данные tempdb и файл журнала tempdb располагаются на диске D:\, выглядит следующим образом.

![Схема простой конфигурация дисков виртуальной машины для SQL Server](./media/dbms_sqlserver_deployment_guide/Simple_disk_structure.PNG)

Эта схема описывает простой сценарий. Как ускользнувшим в статье [рекомендации по развертыванию СУБД Azure для рабочей нагрузки SAP](dbms_guide_general.md), тип хранилища Azure, число и размер дисков зависят от разных факторов. Но, как правило, мы рекомендуем:

- Использование одного большого тома, содержащего файлы данных SQL Server. Причина такой конфигурации в том, что в реальной жизни существует большое число баз данных SAP с файлами баз данных разного размера и разными рабочими нагрузками ввода-вывода.
- Использовать диск D:\ для базы данных tempdb, если производительность достаточно высокая. Если общая рабочая нагрузка ограничена производительностью базы данных tempdb, расположенной на D:\ диск, возможно, потребуется переместить базу данных tempdb в отдельные диски хранилища Azure класса Premium или Ultra Disk, как [описано в этой статье](../../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md).


### <a name="special-for-m-series-vms"></a>Особенности виртуальных машин серии M
Для виртуальных машин Azure серии M задержку записи в журналы транзакций можно уменьшить с помощью ряда возможностей (по сравнению с производительностью хранилища Azure класса Premium), доступных при использовании ускорителя записи Azure. Таким образом, ускоритель записи Azure следует развертывать для виртуальных жестких дисков, образующих том для журналов транзакций SQL Server. Дополнительные сведения см. в документе об [ускорителе записи](../../how-to-enable-write-accelerator.md).
  

### <a name="formatting-the-disks"></a>Форматирование дисков
Для SQL Server размер блока NTFS для дисков, содержащих файлы данных и журналов SQL Server, должен составлять 64 КБ. Форматировать диск D:\ нет необходимости. Он предварительно отформатирован.

Чтобы при восстановлении или создании баз данных предотвратить инициализацию файлов данных путем обнуления их содержимого, следует убедиться, что контекст пользователя, в котором выполняется служба SQL Server, имеет определенное разрешение. Обычно эти разрешения имеют пользователи из группы администраторов Windows. Если служба SQL Server выполняется в контексте пользователя без прав администратора Windows, необходимо назначить этому пользователю право **Выполнение задач по обслуживанию томов**.  Дополнительные сведения см. в следующей статье базы знаний Майкрософт: <https://support.microsoft.com/kb/2574695>.

### <a name="impact-of-database-compression"></a>Влияние сжатия базы данных
В конфигурациях, в которых пропускная способность ввода-вывода может стать ограничивающим фактором, каждая мера, позволяющая сократить объем операций ввода-вывода, помогает растянуть рабочую нагрузку, которую можно запускать в сценарии IaaS, например Azure. Таким образом, если это еще не сделано, применение сжатия PAGE для SQL Server рекомендуется как SAP, так и корпорацией Майкрософт перед отправкой имеющихся баз данных SAP в Azure.

Выполнять сжатие базы данных перед отправкой в Azure рекомендуется по следующим двум причинам:

* Объем отправляемых данных уменьшается.
* Длительность выполнения сжатия сокращается, при условии, что локально можно использовать более мощное оборудование с большим числом ЦП, более высокой пропускной способностью ввода-вывода или меньшими задержками ввода-вывода.
* Уменьшение размеров баз данных может сократить затраты на выделение места на дисках.

Сжатие баз данных работает на виртуальных машинах Azure так же, как и в локальной среде. Дополнительные сведения о том, как сжать существующие базы данных SQL Server для SAP NetWeaver, см. в статье [Улучшенное средство сжатия SAP MSSCOMPRESS](/archive/blogs/saponsqlserver/improved-sap-compression-tool-msscompress). 

## <a name="sql-server-2014-and-more-recent---storing-database-files-directly-on-azure-blob-storage"></a>SQL Server 2014 и более поздних версий: хранение файлов базы данных непосредственно в хранилище BLOB-объектов Azure
SQL Server 2014 и более поздних версий открывает возможность хранить файлы базы данных непосредственно в хранилище BLOB-объектов Azure без создания вокруг них "оболочки" виртуального жесткого диска. В частности, при использовании виртуальных машин хранилища Azure класса "Стандартный" или меньшего размера этот тип развертывания позволяет реализовать сценарии, где можно обойти ограничения операций ввода-вывода в секунду, устанавливаемые ограниченным числом дисков, которые можно подключить к некоторым типам небольших виртуальных машин. Такое развертывание возможно в случае с пользовательскими базами данных, но не с системными базами данных SQL Server. Также это возможно для файлов данных и журналов SQL Server. Если вы предпочитаете развернуть базу данных SQL Server SAP таким образом, не заключая ее в "оболочку" виртуальных жестких дисков, помните следующее:

* Используемая учетная запись хранения должна располагаться в том же регионе Azure, что и учетная запись, которая использовалась для развертывания виртуальной машины, на которой работает SQL Server.
* К этому методу развертывания также применяются перечисленные выше рекомендации, касающиеся распределения виртуальных жестких дисков по разным учетным записям хранения Azure. Это означает количество операций ввода-вывода относительно ограничений учетной записи хранения Azure.
* Вместо квоты по операциям ввода-вывода в хранилище виртуальной машины для пропускной способности сети виртуальной машины в зависимости от ее типа будет учитываться трафик для BLOB-объектов хранилища, представляющий файлы данных и журналов SQL Server. Чтобы узнать пропускную способность сети и хранилища определенного типа виртуальной машины, см. статью [Размеры виртуальных машин Windows в Azure](../../sizes.md).
* В результате принудительной отправки определенных квотой сети операций файлового ввода-вывода вы в основном используете квоту хранилища, а также частично используете общую пропускную способность виртуальной машины.
* Показатели операций ввода-вывода в секунду и пропускной способности для дисков разного размера в хранилище Azure класса Premium больше не учитываются. Даже если созданные вами большие двоичные объекты находятся в хранилище Azure класса Premium. Целевые объекты описаны в статье [Высокопроизводительное хранилище класса Premium и управляемые диски для виртуальных машин](../../disks-types.md#premium-ssd). В результате размещения файлов данных и журналов SQL Server непосредственно в большие двоичные объекты, которые хранятся в хранилище Azure класса Premium, характеристики производительности могут отличаться по сравнению с виртуальными жесткими дисками в хранилище Azure класса Premium.
* Кэширование на основе узла, доступное на дисках хранилища Azure класса Premium, не применяется при размещении файлов данных SQL Server непосредственно в большие двоичные объекты Azure.
* На виртуальных машинах серии M ускоритель записи Azure не может использоваться для поддержки операций записи длиной менее миллисекунды в файл журнала транзакций SQL Server. 

Сведения об этих функциональных возможностях можно найти в статье [Файлы данных SQL Server в Microsoft Azure](/sql/relational-databases/databases/sql-server-data-files-in-microsoft-azure)

Для производственных систем рекомендуется избегать этой конфигурации и размещать файлы журналов и данных SQL Server на виртуальных жестких дисках хранилища Azure класса Premium, а не в больших двоичных объектах Azure напрямую.


## <a name="sql-server-2014-buffer-pool-extension"></a>Расширение буферного пула SQL Server 2014
В SQL Server 2014 появилась новая функция, которая называется [расширением буферного пула](/sql/database-engine/configure-windows/buffer-pool-extension). Эта функция расширяет буферный пул SQL Server, который хранится в памяти, с помощью кэша второго уровня, поддерживаемого локальными твердотельными накопителями сервера или виртуальной машины. Расширение буферного пула позволяет держать в памяти рабочий набор данных большего размера. По сравнению с доступом к хранилищу Azure класса "Стандартный" доступ к расширению буферного пула, который хранится на локальных твердотельных накопителях виртуальной машины Azure, предоставляется во много раз быстрее. Расширение буферного пула не обладает значительными преимуществами по сравнению с кэшем чтения хранилища Azure класса Premium, рекомендованного для файлов данных SQL Server. Причина в том, что оба кэша (расширение буферного пула для SQL Server и кэш чтения хранилища класса Premium) используют локальные диски вычислительных узлов Azure.

Испытание расширения буферного пула SQL Server с рабочей нагрузкой SAP дает смешанные результаты и не позволяет четко определиться, стоит ли использовать его во всех случаях. В идеальном случае рабочий набор, необходимый для приложения SAP, вписывается в основную память. А поскольку Azure предлагает виртуальные машины с памятью до 4 ТБ, рабочий набор может поместиться в память. Таким образом, расширение буферного пула можно применять в редких случаях, а не постоянно.  

## <a name="backuprecovery-considerations-for-sql-server"></a>Рекомендации по резервному копированию и восстановлению для SQL Server
При развертывании SQL Server в Azure методику резервного копирования необходимо пересмотреть. Даже если система не используется для производственных задач, необходимо периодически производить резервное копирование базы данных SAP, размещенной на сервере SQL Server. Так как в службе хранилища Azure хранятся три образа, резервное копирование становится менее важным с точки зрения восстановления хранилища после сбоя. Основная причина соблюдения надлежащего плана резервного копирования и восстановления состоит в том, что, имея возможность восстановить состояние на определенный момент времени, вы можете исправлять логические или пользовательские ошибки. Следовательно, резервные копии используются для восстановления состояния базы данных на определенный момент времени или для создания другой системы в Azure путем копирования существующей базы данных. 

Возможности резервного копирования SQL Server в Azure описаны в статье [Резервное копирование и восстановление SQL Server в виртуальных машинах Azure](../../../azure-sql/virtual-machines/windows/backup-restore.md). В статье описано несколько возможностей.

### <a name="manual-backups"></a>Резервное копирование вручную
Вы можете создавать резервные копии вручную:

1. Создавайте обычные резервные копии SQL Server на дисках Azure, подключенных напрямую. Преимущество этого метода в том, что резервные копии легкодоступны для обновлений системы и создания новых систем путем копирования существующих систем SAP
2.  SQL Server 2012 CU4 и более поздних версий может выполнять резервное копирование баз данных на URL-адрес хранилища Azure.
3.  Резервные копии моментальных снимков файлов базы данных в хранилище BLOB-объектов Azure. Этот метод работает только в том случае, если файлы данных и журналов SQL Server находятся в хранилище BLOB-объектов Azure

Первый метод хорошо известен и часто применяется в локальной среде. Но вам необходимо решить вопрос с долговременным хранением резервной копии. Не следует больше 30 дней хранить резервные копии в локальном хранилище Azure, поэтому нужно использовать резервные службы Azure Backup или стороннее средство резервного копирования и восстановления с функцией управления доступом к резервным копиям и периодом их хранения. Или можно создать в Azure большой файловый сервер с помощью дисковых пространств Windows.

Второй способ подробно описано в статье [Резервное копирование SQL Server на URL-адрес](../../../azure-sql/virtual-machines/windows/backup-restore.md). Эта функция имеет некоторые отличия в разных выпусках SQL Server. Поэтому ознакомьтесь с документацией для вашего выпуска SQL Server. Обратите внимание, что в этой статье перечислено много ограничений. У вас есть две взаимоисключающие возможности:

- Один страничный BLOB-объект, который ограничивает размер резервной копии до 1000 ГБ. Это ограничение также ограничивает пропускную способность, которую можно достичь.
- Несколько (до 64) блочных BLOB-объектов Azure, с которыми размер резервной копии теоретически увеличивается до 12 ТБ. Но тесты в базах данных клиента показали, что максимальный размер резервной копии может быть меньше теоретического ограничения. В этом случае вы несете ответственность за управление периодом удержания резервных копий и доступ к ним.


### <a name="automated-backup-for-sql-server"></a>Автоматизированное резервное копирование для SQL Server
Автоматическое резервное копирование реализует службу автоматического резервного копирования для экземпляров SQL Server Standard и Enterprise, работающих на виртуальной машине Windows в Azure. Эта служба работает на базе [расширения агента SQL Server IaaS](../../../azure-sql/virtual-machines/windows/sql-server-iaas-agent-extension-automate-management.md), которое автоматически устанавливается на все образы виртуальных машин Windows для SQL Server, представленные на портале Azure. Если вы развертываете собственные образы операционной системы с установленным SQL Server, необходимо отдельно установить расширения виртуальной машины. Необходимые шаги описаны в этой [статье](../../../azure-sql/virtual-machines/windows/sql-server-iaas-agent-extension-automate-management.md).

Дополнительные сведения о возможностях этого метода можно найти в следующих статьях:

- Для SQL Server 2014: [Автоматическая архивация для виртуальных машин SQL Server 2014 (Resource Manager)](../../../azure-sql/virtual-machines/windows/automated-backup-sql-2014.md)
- Для SQL Server 2016/2017: [Автоматическая архивация версии 2 для виртуальных машин Azure (Resource Manager)](../../../azure-sql/virtual-machines/windows/automated-backup.md)

В документации вы увидите, что в последних выпусках SQL Server эти функции усовершенствованы. Дополнительные сведения об автоматизированном резервном копировании SQL Server приводятся в статье [Управляемое резервное копирование SQL Server в Microsoft Azure](/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure). Теоретический лимит размера резервный копий составляет 12 ТБ.  Автоматическое резервное копирование — это удобный метод с размером резервных копий до 12 ТБ. Так как несколько больших двоичных объектов записываются в параллельном режиме, можно ожидать пропускную способность более 100 МБ в секунду. 
 

### <a name="azure-backup-for-sql-server-vms"></a>Azure Backup для виртуальных машин SQL Server
Этот новый метод резервного копирования SQL Server начиная с июня 2018 г. предоставляется как общедоступная предварительная версия в службах Azure Backup. Метод резервного копирования SQL Server используется такой же, как в других сторонних средствах, а именно интерфейс SQL Server VSS/VDI для потоковой передачи резервных копий в целевое расположение. В этом случае целевое расположение — хранилище службы восстановления Azure.

Более подробное описание этого метода резервного копирования, который добавляет многочисленные преимущества центральной настройки, мониторинга и администрирования резервного копирования, доступно [здесь](../../../backup/backup-azure-sql-database.md). 


### <a name="third-party-backup-solutions"></a>Сторонние решения для резервного копирования
Для множества клиентов SAP невозможно было начать заново и внедрить совершенно новые решения резервного копирования для той части архитектуры SAP, которая выполняется в Azure. Поэтому нужно было использовать в Azure существующие решения. Перенос существующих решений резервного копирования в Azure обычно удавался для большинства поставщиков в этой сфере. 


## <a name="using-a-sql-server-image-out-of-the-microsoft-azure-marketplace"></a><a name="1b353e38-21b3-4310-aeb6-a77e7c8e81c8"></a>Использование образов SQL Server из Microsoft Azure Marketplace
Корпорация Майкрософт размещает в Azure Marketplace виртуальные машины, которые уже содержат версии SQL Server. Следовательно, клиенты SAP, которым нужны лицензии на SQL Server и Windows, с помощью этих образов могут удовлетворить потребность в лицензиях, развертывая виртуальные машины с уже установленной средой SQL Server. Чтобы использовать такие образы для SAP, необходимо учитывать следующие факторы.

* Неознакомительные версии SQL Server стоят дороже, чем виртуальная машина "только для Windows", развернутая из Azure Marketplace. Сравнение цен см. в следующих статьях: <https://azure.microsoft.com/pricing/details/virtual-machines/windows/> и <https://azure.microsoft.com/pricing/details/virtual-machines/sql-server-enterprise/>. 
* Использовать можно только те выпуски SQL Server, которые поддерживаются SAP.
* Параметры сортировки экземпляра SQL Server, который устанавливается на виртуальных машинах из Azure Marketplace, не соответствуют параметрам, необходимым SAP NetWeaver для запуска экземпляра SQL Server. Параметры сортировки можно изменить, выполнив инструкции из следующего раздела.

### <a name="changing-the-sql-server-collation-of-a-microsoft-windowssql-server-vm"></a>Изменение параметров сортировки SQL Server на виртуальной машине с Microsoft Windows и SQL Server
Так как в образах SQL Server в Azure Marketplace не настроено использование параметров сортировки, необходимых для приложений SAP NetWeaver, их необходимо изменить сразу же после развертывания. Чтобы изменить параметры сортировки в SQL Server, выполните следующие действия сразу после того, как виртуальная машина будет развернута и администратор сможет выполнить в нее вход.

* Откройте окно командной строки Windows с правами администратора.
* Перейдите в каталог C:\Program Files\Microsoft SQL Server\110\Setup Bootstrap\SQLServer2012.
* Выполните команду: Setup.exe /QUIET /ACTION=REBUILDDATABASE /INSTANCENAME=MSSQLSERVER /SQLSYSADMINACCOUNTS=`<local_admin_account_name`> /SQLCOLLATION=SQL_Latin1_General_Cp850_BIN2   
  * `<local_admin_account_name`> — это учетная запись, которая была определена как учетная запись администратора при развертывании виртуальной машины в первый раз с помощью коллекции.

Процесс должен занять только несколько минут. Чтобы проверить, завершился ли этот шаг правильным результатом, выполните следующие действия.

* Откройте среду SQL Server Management Studio.
* Откройте окно запроса.
* Выполните команду sp_helpsort в базе данных master SQL Server.

Правильный результат должен выглядеть следующим образом:

```output
Latin1-General, binary code point comparison sort for Unicode Data, SQL Server Sort Order 40 on Code Page 850 for non-Unicode Data
```

Если результат выглядит иначе, ОСТАНОВИТЕ развертывание SAP и выясните, почему команда установки сработала не так, как ожидалось. Развертывание приложений SAP NetWeaver в экземпляре SQL Server с кодовыми страницами SQL Server, отличающимися от упомянутой выше, **НЕ** поддерживается.

## <a name="sql-server-high-availability-for-sap-in-azure"></a>Высокая доступность SQL Server для SAP в Azure
Если вы используете SQL Server в развертываниях Azure IaaS для SAP, у вас есть несколько возможностей для развертывания уровня СУБД высокой доступности. Как уже говорилось в разделе [рекомендации по развертыванию СУБД Azure для рабочей нагрузки SAP](dbms_guide_general.md) , Azure предоставляет разные соглашения об уровне обслуживания для одной виртуальной машины и пару виртуальных машин, развернутых в группе доступности Azure. Предполагается, что для развертываний в рабочих средах вы склоняетесь к соглашению об уровне обслуживания с указанием времени бесперебойной работы, которое требует развертывания в группах доступности Azure. В этом случае необходимо развернуть как минимум две виртуальные машины в такой группе доступности. На одной виртуальной машине будет выполняться активный экземпляр SQL Server. На второй виртуальной машине будет запущен пассивный экземпляр

### <a name="sql-server-clustering-using-windows-scale-out-file-server-or-azure-shared-disk"></a>SQL Server кластеризации с помощью масштабируемого файлового сервера Windows или общего диска Azure
В Windows Server 2016 корпорация Майкрософт представила [локальные дисковые пространства](/windows-server/storage/storage-spaces/storage-spaces-direct-overview). В зависимости от развертывания Локальные дисковые пространства в целом поддерживается кластеризация SQL Server FCI. Azure также предлагает [Общие диски Azure](../../disks-shared-enable.md?tabs=azure-cli) , которые можно использовать для кластеризации Windows. Для рабочей нагрузки SAP мы не поддерживаем эти параметры высокой доступности. 

### <a name="sql-server-log-shipping"></a>Доставка журналов SQL Server
Одним из способов обеспечения высокого уровня доступности (HA) является доставка журналов SQL Server. Если в конфигурации высокой доступности виртуальные машины имеют работающее разрешение имен, проблемы нет и настройка в Azure не будет отличаться от любой настройки, которая выполняется локально. Настройка доставки журналов и принципы их доставки. Сведения о доставке журналов SQL Server можно найти в статье [Сведения о доставке журналов (SQL Server)](/sql/database-engine/log-shipping/about-log-shipping-sql-server).

Функциональные возможности доставки журналов SQL Server почти не использовались в Azure для обеспечения высокой доступности в пределах одного региона Azure. Тем не менее в следующих сценариях клиенты SAP успешно использовали доставку журналов в сочетании с Azure:

- Сценарии аварийного восстановления из одного региона Azure в другой
- Конфигурация аварийного восстановления из локальной среды в регион Azure
- Сценарии перехода из локальной среды в Azure. В таких случаях доставка журналов используется для синхронизации нового развертывания СУБД в Azure с непрерывно работающей локальной рабочей системой. В момент перехода рабочая среда завершает работу, и выполняется проверка передачи последних резервных копий журналов в развертывание СУБД Azure. Затем развертывание СУБД Azure открывается для рабочей среды.  



### <a name="database-mirroring"></a>Зеркальное отображение базы данных
Зеркальное отображение базы данных в поддерживаемом SAP виде (см. примечание к SAP [965908]) полагается на определение партнера по отработке отказа в строке подключения SAP. При подключении между организациями предполагается, что две виртуальные машины находятся в одном и том же домене, а пользовательский контекст, в котором запускаются два экземпляра SQL Server, является также пользователями домена, которые имеют достаточно прав в двух задействованных экземплярах SQL Server. Таким образом, настройка зеркального отображения базы данных в Azure не отличается от обычной локальной установки и конфигурации.

Самый простой способ полностью облачного развертывания — настроить другой домен в Azure так, чтобы эти виртуальные машины СУБД (а в идеале выделенные виртуальные машины SAP) находились в пределах одного домена.

Если использовать домен невозможно, можно также применять сертификаты для конечных точек зеркального отображения базы данных, как описано здесь: <https://docs.microsoft.com/sql/database-engine/database-mirroring/use-certificates-for-a-database-mirroring-endpoint-transact-sql>

Руководство по настройке зеркального отображения базы данных в Azure см. по ссылке: <https://docs.microsoft.com/sql/database-engine/database-mirroring/database-mirroring-sql-server> 

### <a name="sql-server-always-on"></a>AlwaysOn в SQL Server
Так как функция AlwaysOn поддерживается для SAP в локальной среде (см. примечание к SAP [1772688]), то ее можно использовать вместе с SAP в Azure. Существуют особые рекомендации по развертыванию прослушивателя группы доступности SQL Server (не путать с группой доступности Azure), так как Azure на данный момент не разрешает создавать объект AD/DNS, как это возможно в локальной среде. Таким образом, необходимо выполнить ряд отдельных шагов по установке для преодоления характерного поведения Azure.

Ниже приведены некоторые рекомендации по использованию прослушивателя группы доступности.

* Использование прослушивателя группы доступности возможно, только если в качестве гостевой ОС виртуальной машины используется Windows Server 2012 или более поздней версии. Если вы используете Windows Server 2012, установите следующее исправление: <https://support.microsoft.com/kb/2854082>. 
* Для Windows Server 2008 R2 такое исправление отсутствует, и AlwaysOn необходимо использовать таким же образом, как зеркальное отображение базы данных, указав партнера по обеспечению отработки отказа в строке подключения (для этого используется параметр dbs/mss/server в SAP default.pfl; см. примечание к SAP [965908]).
* При использовании прослушивателя группы доступности виртуальные машины баз данных необходимо подключить к выделенному Load Balancer. Чтобы в Azure не назначались новые IP-адреса при случайном завершении работы обеих виртуальных машин, следует назначить статические IP-адреса сетевым интерфейсам этих виртуальных машин в конфигурации AlwaysOn (определение статического IP-адреса описано в [этой статье][virtual-networks-reserved-private-ip]).
* Особые действия требуются при построении конфигурации кластера WSFC, где кластеру требуется назначить специальный IP-адрес, так как Azure с текущими функциональными возможностями назначит имени кластера тот же IP-адрес, что и у узла, на котором создается кластер. Такое поведение означает, что для назначения кластеру другого IP-адреса необходимо выполнить действие вручную.
* Прослушиватель группы доступности будет создан в Azure с конечными точками TCP/IP, назначенными виртуальным машинам, на которых запускаются первичная и вторичная реплики группы доступности.
* Может возникнуть необходимость обеспечить безопасность этих конечных точек с помощью списков управления доступом.

Подробная документация по развертыванию AlwaysOn с SQL Server на виртуальных машинах Azure:

- [Введение в группы доступности AlwaysOn SQL Server на виртуальных машинах Azure](../../../azure-sql/virtual-machines/windows/availability-group-overview.md).
- [Настройка группы доступности AlwaysOn на виртуальных машинах Azure в разных регионах](../../../azure-sql/virtual-machines/windows/availability-group-manually-configure-multiple-regions.md).
- [Настройка подсистемы балансировки нагрузки для группы доступности AlwaysOn в Azure](../../../azure-sql/virtual-machines/windows/availability-group-load-balancer-portal-configure.md).

>[!NOTE]
> Если вы настраиваете подсистему балансировки нагрузки Azure для виртуального IP-адреса прослушивателя группы доступности, убедитесь, что DirectServerReturn настроен. Настройка этого параметра позволит снизить задержку кругового пути между уровнем приложений SAP и уровнем СУБД. 

SQL Server AlwaysOn — это самая популярная функция высокой доступности и аварийного восстановления, которая используется в Azure для развертываний рабочей нагрузки SAP. Большинство клиентов применяет AlwaysOn для обеспечения высокой доступности в одном регионе Azure. Если развертывание будет ограничено только двумя узлами, у вас есть два варианта подключения:

- Использование прослушивателя группы доступности. При использовании прослушивателя группы доступности необходимо развернуть подсистему балансировки нагрузки Azure. Это способ развертывания по умолчанию. Приложения SAP будут настроены для подключения к прослушивателю группы доступности, а не к одному узлу
- Использование параметров подключения зеркального отображения базы данных SQL Server. В этом случае необходимо настроить подключение приложений SAP так, чтобы были указаны имена обоих узлов. Точные сведения о такой конфигурации на стороне SAP задокументированы в примечании к SAP [965908](https://launchpad.support.sap.com/#/notes/965908). При использовании этого параметра не придется настраивать прослушиватель группы доступности. А значит, никакой подсистемы балансировки нагрузки Azure для обеспечения высокой доступности SQL Server. В результате задержка сети между уровнем приложений SAP и уровнем СУБД меньше, так как входящий трафик к экземпляру SQL Server не направляется через подсистему балансировки нагрузки Azure. Но помните, что этот параметр работает, только если вы ограничите группу доступности двумя экземплярами. 

Довольно много клиентов используют функции SQL Server AlwaysOn для дополнительных возможностей аварийного восстановления между регионами Azure. Некоторые клиенты также используют возможность создавать резервные копии из вторичной реплики. 

## <a name="sql-server-transparent-data-encryption"></a>Прозрачное шифрование данных SQL Server
Некоторые клиенты используют [прозрачное шифрование данных (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption) SQL Server при развертывании баз данных SAP SQL Server в Azure. Функция прозрачного шифрования данных для SQL Server полностью поддерживается системой SAP (см. примечание к SAP [1380493](https://launchpad.support.sap.com/#/notes/1380493)). 

### <a name="applying-sql-server-tde"></a>Применение прозрачного шифрования данных SQL Server
Если вы выполняете разнородный переход с другой СУБД, работающей локально, в Windows или SQL Server в Azure, необходимо заранее создать пустую целевую базу данных в SQL Server. На следующем этапе вы примените функцию прозрачного шифрования данных для SQL Server, пока ваша рабочая система все еще работает локально. Причина такой последовательности в том, что процесс шифрования пустой базы данных может занять немало времени. Процессы импорта SAP импортируют данные в зашифрованную базу данных во время фазы простоя. Импорт зашифрованной базы данных требует гораздо меньше ресурсов, чем шифрование базы данных после завершения этапа экспорта в фазе простоя. При попытке применить TDE с рабочей нагрузкой SAP, запущенной на базе базы данных, были сделаны отрицательные впечатления. Поэтому мы рекомендуем выполнять прозрачное шифрование данных без рабочей нагрузки SAP в определенной базе данных.

Если вы перемещаете базы данных SAP SQL Server из локальной среды в Azure, мы рекомендуем протестировать, в какой инфраструктуре шифрование будет выполнено быстрее. Для этого учитывайте следующие факты:

- Невозможно определить, сколько потоков используется для шифрования данных в базе данных. Число потоков во многом зависит от числа томов диска, по которым распределены файлы данных и журналов SQL Server. Чем больше отдельных томов (букв дисков), тем больше будет параллельных потоков при шифровании. Такая конфигурация немного противоречит предыдущим рекомендациям по конфигурации диска, когда мы советовали использовать одно или всего несколько дисковых пространств для файлов базы данных SQL Server на виртуальных машинах Azure. В конфигурации с небольшим числом томов во время шифрования будет задействовано небольшое число потоков. Шифрование одного потока — это чтение экстентов размером 64 КБ, шифрует его, а затем записывает запись в файл журнала транзакций, указывая, что экстент зашифрован. В результате журнал транзакций испытывает умеренную нагрузку.
- В предыдущих выпусках SQL Server сжатие резервных копий уже не позволяло повысить эффективность при шифровании базы данных SQL Server. Такое поведение могло превратиться в проблему, если вы планировали зашифровать базу данных SQL Server локально, а затем скопировать резервную копию в Azure, чтобы восстановить базу данных в Azure. Сжатие резервных копий SQL Server обычно имеет коэффициент 4.
- В SQL Server 2016 представлены новые возможности, позволяющие эффективно сжимать зашифрованные базы данных. Дополнительные сведения см. в [этом блоге](/archive/blogs/sqlcat/sqlsweet16-episode-1-backup-compression-for-tde-enabled-databases) .
 
Рассматривая применение шифрования TDE только с рабочей нагрузкой SAP или без него, следует протестировать конкретную конфигурацию, чтобы лучше применять TDE к локальной базе данных SAP или сделать это в Azure. В Azure у вас определенно будет больше возможностей с точки зрения излишней подготовки инфраструктуры и сжатия инфраструктуры после применения прозрачного шифрования данных.

### <a name="using-azure-key-vault"></a>Использование Azure Key Vault
Azure предлагает службу [Key Vault](https://azure.microsoft.com/services/key-vault/) для хранения ключей шифрования. SQL Server, с другой стороны, предлагает соединитель, чтобы использовать Azure Key Vault для хранения сертификатов прозрачного шифрования данных.

Дополнительные сведения об использовании Azure Key Vault для прозрачного шифрования данных в SQL Server см. в следующих разделах:

- [Расширенное управление ключами с помощью службы Azure Key Vault (SQL Server)](/sql/relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server).
- [Расширенное управление ключами для прозрачного шифрования данных в SQL Server с помощью службы Azure Key Vault — настройка](/sql/relational-databases/security/encryption/setup-steps-for-extensible-key-management-using-the-azure-key-vault).
- [Обслуживание и устранение неполадок соединителя SQL Server](/sql/relational-databases/security/encryption/sql-server-connector-maintenance-troubleshooting).
- [Дополнительные вопросы клиентов о прозрачном шифровании данных в SQL Server — TDE + Azure Key Vault](/archive/blogs/saponsqlserver/more-questions-from-customers-about-sql-server-transparent-data-encryption-tde-azure-key-vault).


>[!IMPORTANT]
>При использовании прозрачного шифрования данных в SQL Server, особенно с Azure Key Vault, рекомендуется использовать последние пакеты обновления SQL Server 2014, SQL Server 2016 и SQL Server 2017. Это связано с отзывами клиентов, оптимизацией и исправлениями, примененными к коду. Например, см. [KBA 4058175](https://support.microsoft.com/help/4058175/tde-enabled-backup-and-restore-slow-if-encryption-key-is-stored-in-ekm).
>  

## <a name="general-sql-server-for-sap-on-azure-summary"></a><a name="9053f720-6f3b-4483-904d-15dc54141e30"></a>Общие сведения об SQL Server для SAP в Azure
Это руководство содержит много рекомендаций, и мы советуем прочитать его несколько раз перед планированием развертывания в Azure. В целом необходимо учитывать основные рекомендации, касающиеся СУБД в Azure:

1. Используйте новейшую версию СУБД, например SQL Server 2017, которая обладает максимумом преимуществ в Azure. 
2. Тщательно планируйте ландшафт системы SAP в Azure, чтобы сбалансировать структуру файлов данных и ограничения Azure.
   * Не устанавливайте слишком много дисков, но обеспечьте достаточное количество для достижения необходимого числа операций ввода-вывода.
   * Если вы не используете Управляемые диски, помните, что количество операций ввода-вывода также ограничено для каждой учетной записи хранения Azure, а количество учетных записей хранения ограничено для каждой подписки Azure (см. [дополнительные сведения][azure-resource-manager/management/azure-subscription-service-limits]). 
   * Создавайте stripe-массив из дисков только при необходимости обеспечить более высокую пропускную способность.
3. Никогда не устанавливайте программное обеспечение и не помещайте файлы, которые требуется сохранять на постоянной основе, на диск D:\, так как он является непостоянным, и любое содержимое этого диска будет потеряно при перезагрузке Windows.
4. Не используйте кэширование дисков для хранилища Azure класса Standard.
5. Не используйте стандартные геореплицированные учетные записи хранения Azure.  Используйте локальную избыточность для рабочих нагрузок СУБД.
6. Используйте решение HA/DR поставщика СУБД для репликации данных из базы данных.
7. Всегда используйте разрешение имен, не полагайтесь на IP-адреса.
8. Если вы используете прозрачное шифрование данных в SQL Server, применяйте последние исправления для SQL Server.
9. Используйте наибольшую возможную степень сжатия базы данных. Для SQL Server это сжатие страниц.
10. Соблюдайте осторожность при использовании образов SQL Server из Azure Marketplace. При использовании образа SQL Server необходимо изменить параметры сортировки экземпляра, прежде чем устанавливать на него систему SAP NetWeaver.
11. Установите и настройте мониторинг узла SAP для Azure, как описано в [руководстве по развертыванию][deployment-guide].


## <a name="next-steps"></a>Дальнейшие действия
Читать статью 

- [Вопросы развертывания СУБД для рабочей нагрузки SAP на виртуальных машинах Azure](dbms_guide_general.md)