---
title: 'SAP в Azure: руководство по планированию и реализации'
description: SAP NetWeaver на виртуальных машинах Windows. Руководство по планированию и внедрению
author: MSSedusch
manager: juergent
tags: azure-resource-manager
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/17/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017, devx-track-azurecli
ms.openlocfilehash: 8bc289e90470ae9bc8b1996ac08c3144ea78de35
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102504718"
---
# <a name="azure-virtual-machines-planning-and-implementation-for-sap-netweaver"></a>SAP NetWeaver на виртуальных машинах Windows. Руководство по планированию и внедрению

[767598]:https://launchpad.support.sap.com/#/notes/767598
[773830]:https://launchpad.support.sap.com/#/notes/773830
[826037]:https://launchpad.support.sap.com/#/notes/826037
[965908]:https://launchpad.support.sap.com/#/notes/965908
[1031096]:https://launchpad.support.sap.com/#/notes/1031096
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

[dbms-guide]:dbms-guide.md
[dbms-guide-2.1]:dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f
[dbms-guide-2.2]:dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91
[dbms-guide-2.3]:dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152
[dbms-guide-2]:dbms-guide.md#65fa79d6-a85f-47ee-890b-22e794f51a64
[dbms-guide-3]:dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3
[dbms-guide-5.5.1]:dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268
[dbms-guide-5.5.2]:dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b
[dbms-guide-5.6]:dbms-guide.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8
[dbms-guide-5.8]:dbms-guide.md#9053f720-6f3b-4483-904d-15dc54141e30
[dbms-guide-5]:dbms-guide.md#3264829e-075e-4d25-966e-a49dad878737
[dbms-guide-8.4.1]:dbms-guide.md#b48cfe3b-48e9-4f5b-a783-1d29155bd573
[dbms-guide-8.4.2]:dbms-guide.md#23c78d3b-ca5a-4e72-8a24-645d141a3f5d
[dbms-guide-8.4.3]:dbms-guide.md#77cd2fbb-307e-4cbf-a65f-745553f72d2c
[dbms-guide-8.4.4]:dbms-guide.md#f77c1436-9ad8-44fb-a331-8671342de818
[dbms-guide-900-sap-cache-server-on-premises]:dbms-guide.md#642f746c-e4d4-489d-bf63-73e80177a0a8

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

[msdn-set-Azvmaemextension]:https://msdn.microsoft.com/library/azure/mt670598.aspx

[planning-guide]:planning-guide.md
[planning-guide-1.2]:planning-guide.md#e55d1e22-c2c8-460b-9897-64622a34fdff
[planning-guide-11]:planning-guide.md#7cf991a1-badd-40a9-944e-7baae842a058
[planning-guide-11.4.1]:planning-guide.md#5d9d36f9-9058-435d-8367-5ad05f00de77
[planning-guide-11.5]:planning-guide.md#4e165b58-74ca-474f-a7f4-5e695a93204f
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

[powershell-install-configure]:https://docs.microsoft.com/powershell/azure/install-az-ps
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
[storage-powershell-guide-full-copy-vhd]:../../../storage/common/storage-powershell-guide-full.md
[storage-premium-storage-preview-portal]:../../disks-types.md
[storage-redundancy]:../../../storage/common/storage-redundancy.md
[storage-scalability-targets]:../../../storage/common/scalability-targets-standard-accounts.md
[storage-use-azcopy]:../../../storage/common/storage-use-azcopy.md
[template-201-vm-from-specialized-vhd]:https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-from-specialized-vhd
[templates-101-simple-windows-vm]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-simple-windows-vm
[templates-101-vm-from-user-image]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image
[virtual-machines-linux-attach-disk-portal]:../../linux/attach-disk-portal.md
[virtual-machines-azure-resource-manager-architecture]:../../../resource-manager-deployment-model.md
[virtual-machines-Az-versus-azuresm]:virtual-machines-linux-compare-deployment-models.md
[virtual-machines-windows-classic-configure-oracle-data-guard]:../../virtual-machines-windows-classic-configure-oracle-data-guard.md
[virtual-machines-linux-cli-deploy-templates]:../../linux/cli-deploy-templates.md
[virtual-machines-deploy-rmtemplates-powershell]:../../virtual-machines/windows/index.yml
[virtual-machines-linux-agent-user-guide]:../../extensions/agent-linux.md
[virtual-machines-linux-agent-user-guide-command-line-options]:../../extensions/agent-windows.md#command-line-options
[virtual-machines-linux-capture-image]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager-capture]:../../linux/capture-image.md#step-2-capture-the-vm
[virtual-machines-windows-capture-image-resource-manager]:../../windows/capture-image.md
[virtual-machines-windows-capture-image]:../../virtual-machines-windows-generalize-vhd.md
[virtual-machines-windows-capture-image-prepare-the-vm-for-image-capture]:../../virtual-machines-windows-generalize-vhd.md
[virtual-machines-linux-configure-raid]:../../linux/configure-raid.md
[virtual-machines-linux-configure-lvm]:../../linux/configure-lvm.md
[virtual-machines-linux-classic-create-upload-vhd-step-1]:../../virtual-machines-linux-classic-create-upload-vhd.md#step-1-prepare-the-image-to-be-uploaded
[virtual-machines-linux-create-upload-vhd-suse]:../../linux/suse-create-upload-vhd.md
[virtual-machines-linux-create-upload-vhd-oracle]:../../linux/oracle-create-upload-vhd.md
[virtual-machines-linux-redhat-create-upload-vhd]:../../linux/redhat-create-upload-vhd.md
[virtual-machines-linux-how-to-attach-disk]:../../linux/add-disk.md
[virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux]:../../linux/add-disk.md#format-and-mount-the-disk
[virtual-machines-linux-tutorial]:../../linux/quick-create-cli.md
[virtual-machines-linux-update-agent]:../../linux/update-agent.md
[virtual-machines-manage-availability]:../../linux/availability.md
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]:virtual-machines-windows-create-powershell.md
[virtual-machines-sizes-linux]:../../linux/sizes.md
[virtual-machines-sizes-windows]:../../windows/sizes.md
[virtual-machines-windows-classic-ps-sql-alwayson-availability-groups]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups.md
[virtual-machines-windows-classic-ps-sql-int-listener]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener.md
[virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions]:./../../windows/sql/virtual-machines-windows-sql-high-availability-dr.md
[virtual-machines-sql-server-infrastructure-services]:./../../windows/sql/virtual-machines-windows-sql-server-iaas-overview.md
[virtual-machines-sql-server-performance-best-practices]:./../../windows/sql/virtual-machines-windows-sql-performance.md
[virtual-machines-upload-image-windows-resource-manager]:../../virtual-machines-windows-upload-image.md
[virtual-machines-windows-tutorial]:../../virtual-machines-windows-hero-tutorial.md
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/documentation/templates/sql-server-2014-alwayson-dsc/
[virtual-network-deploy-multinic-arm-cli]:../../linux/multiple-nics.md
[virtual-network-deploy-multinic-arm-ps]:../../windows/multiple-nics.md
[virtual-network-deploy-multinic-arm-template]:../../../virtual-network/template-samples.md
[virtual-networks-configure-vnet-to-vnet-connection]:../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md
[virtual-networks-create-vnet-arm-pportal]:../../../virtual-network/manage-virtual-network.md#create-a-virtual-network
[virtual-networks-manage-dns-in-vnet]:../../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md
[virtual-networks-multiple-nics-windows]:../../windows/multiple-nics.md
[virtual-networks-multiple-nics-linux]:../../linux/multiple-nics.md
[virtual-networks-nsg]:../../../virtual-network/security-overview.md
[virtual-networks-reserved-private-ip]:../../../virtual-network/virtual-networks-static-private-ip-arm-ps.md
[virtual-networks-static-private-ip-arm-pportal]:../../../virtual-network/virtual-networks-static-private-ip-arm-pportal.md
[virtual-networks-udr-overview]:../../../virtual-network/virtual-networks-udr-overview.md
[vpn-gateway-about-vpn-devices]:../../../vpn-gateway/vpn-gateway-about-vpn-devices.md
[vpn-gateway-create-site-to-site-rm-powershell]:../../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md
[vpn-gateway-cross-premises-options]:../../../vpn-gateway/vpn-gateway-about-vpngateways.md
[vpn-gateway-site-to-site-create]:../../../vpn-gateway/vpn-gateway-site-to-site-create.md
[vpn-gateway-vpn-faq]:../../../vpn-gateway/vpn-gateway-vpn-faq.md
[xplat-cli]:../../../cli-install-nodejs.md
[xplat-cli-azure-resource-manager]:../../../xplat-cli-azure-resource-manager.md
[capture-image-linux-step-2-create-vm-image]:../../linux/capture-image.md#step-2-create-vm-image



Microsoft Azure позволяет компаниям максимально быстро получать вычислительные ресурсы и ресурсы хранения без длительных циклов закупки. С помощью службы виртуальных машин Azure на предприятиях можно развернуть классические приложения, например приложения на основе SAP NetWeaver в среде Azure, и повысить их надежность и доступность без хранения ресурсов локально. Службы виртуальных машин Azure также поддерживают возможность подключения между организациями, благодаря чему компании могут активно интегрировать виртуальные машины Azure в локальные домены, частные облака и системный ландшафт SAP.
В этом техническом документе приводятся основные сведения о виртуальных машинах Microsoft Azure и пошаговые инструкции по планированию и реализации для установок SAP NetWeaver в Azure. Именно этот документ следует изучить, прежде чем начинать развертывание SAP NetWeaver в Azure.
Это руководство дополняет документацию по установке SAP и комментарии по SAP, которые являются основными ресурсами по установке и развертыванию SAP на упомянутых платформах.

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

## <a name="summary"></a>Сводка
Облачные вычисления — это широко используемый термин, который приобретает все большее значение в отрасли ИТ, начиная от малых компаний и заканчивая крупными и транснациональными корпорациями.

Microsoft Azure — это созданная корпорацией Майкрософт платформа облачных служб, предоставляющая широкий спектр новых возможностей. Теперь клиенты могут быстро подготавливать и отзывать приложения в формате облачных служб без технических или бюджетных ограничений. Вместо того, чтобы тратить время и средства на создание и поддержку аппаратной инфраструктуры, компании могут сконцентрироваться на приложениях, бизнес-процессах, а также связанных преимуществах для клиентов и пользователей.

Корпорация Майкрософт предоставляет службы виртуальных машин Microsoft Azure как комплексную платформу «инфраструктура как услуга» (IaaS). Виртуальные машины Azure (IaaS) поддерживают приложения на основе SAP NetWeaver. В этом техническом документе описано, как следует планировать и внедрять приложения на основе SAP NetWeaver на платформе Microsoft Azure.

Основное внимание в этом документе уделено двум основным аспектам.

* В первой части описываются две поддерживаемые модели для развертывания в Azure приложений на основе SAP NetWeaver. Также в ней описываются общие процессы управления Azure и их специфика для развертываний SAP.
* Во второй части приводятся подробные инструкции по реализации различных сценариев, описанных в первой части.

Дополнительные ресурсы перечислены в этой статье в разделе [Ресурсы][planning-guide-1.2].

### <a name="definitions-upfront"></a>Определения
В этом документе мы будем использовать следующие термины.

* IaaS: инфраструктура как услуга
* PaaS: Платформа как услуга
* SaaS: программное обеспечение как услуга
* Компонент SAP: отдельное приложение SAP, например ECC, BW, Solution Manager или S/4HANA.  Компоненты SAP могут основываться на традиционной технологии ABAP или Java или быть приложениями не на основе NetWeaver, например бизнес-объектами.
* Среда SAP: один или несколько компонентов SAP, логически сгруппированных для выполнения бизнес-функции (разработка, оценка качества, обучение, аварийное восстановление или производство).
* Ландшафт SAP: этот термин обозначает все ресурсы SAP, доступные в клиентском ИТ-ландшафте. Ландшафт SAP включает все рабочие и нерабочие среды.
* Система SAP: сочетание уровня СУБД и уровня приложений системы разработки SAP ERP, например, тестовой системы SAP BW, рабочей системы SAP CRM и т. д. В развертываниях Azure не поддерживается разделение этих двух уровней между локальной средой и Azure. Таким образом, систему SAP необходимо развернуть полностью в локальной среде или полностью в Azure. Но при этом вы можете развернуть различные системы ландшафта SAP как в Azure, так и локально. Например, системы разработки и тестирования SAP CRM можно развернуть в Azure, а производственную систему SAP CRM — в локальной среде.
* Распределенная или гибридная среда: описывает сценарии, при которых в подписке Azure развернуты виртуальные машины с подключениями типа "сеть – сеть" или ExpressRoute между локальными центрами обработки данных и Azure. В документации Azure развертывания такого рода также называются распределенными или гибридными. Такое подключение используется для расширения в Azure локальных доменов, локальной службы Active Directory, OpenLDAP и локальной службы DNS. Локальная среда распространяется на ресурсы Azure, входящие в подписку. При таком расширении виртуальные машины могут быть частью локального домена. Пользователи локального домена имеют доступ к серверам и могут запускать службы на этих виртуальных машинах (например, службы СУБД). Возможно взаимодействие и разрешение имен между виртуальными машинами, развернутыми в локальной сети и в Azure. Это относится к наиболее распространенным и практически эксклюзивным случаям развертывания ресурсов SAP в Azure. Дополнительные сведения см. в [этой][vpn-gateway-cross-premises-options] и [этой][vpn-gateway-site-to-site-create] статьях.
* Расширение мониторинга Azure, расширенный мониторинг и расширение Azure для SAP: означают одно и то же. Здесь описывается расширение виртуальной машины, которое необходимо развернуть, чтобы предоставить основные данные об инфраструктуре Azure агенту узла SAP. В примечаниях SAP это может называться расширением мониторинга или расширенным наблюдением. В Azure мы называем это **расширением Azure для SAP**.

> [!NOTE]
> Для рабочих систем SAP поддерживаются распределенные или гибридные развертывания систем SAP, в которых системы SAP выполняются на виртуальных машинах Azure, являющихся членами локального домена. Распределенная или гибридная конфигурация предполагает развертывание в Azure полного ландшафта SAP или его частей. Даже если весь ландшафт SAP работает в Azure, эти виртуальные машины должны входить в локальный домен и каталог ADS (OpenLDAP).
>
>



### <a name="resources"></a><a name="e55d1e22-c2c8-460b-9897-64622a34fdff"></a>Ресурсы
Сведения о начале работы с рабочими нагрузками SAP можно найти в документации по Azure, доступной [здесь](./get-started.md). Начав с этой отправной точки, вы найдете множество статей, посвященных следующим темам:

- Использование SAP NetWeaver и Business One в Azure.
- Руководства по СУБД SAP для различных СУБД в Azure.
- Высокая доступность и аварийное восстановление рабочих нагрузок SAP в Azure.
- Специальные рекомендации по запуску SAP HANA в Azure.
- Рекомендации по крупным экземплярам HANA в Azure для СУБД SAP HANA.


> [!IMPORTANT]
> По возможности приводится ссылка на руководство по установке SAP или другую документацию по SAP (справочное руководство по установке 01: <http://service.sap.com/instguides>). Что касается предварительных требований, процесса установки или сведений о конкретных функциональных возможностях SAP: внимательно читайте документацию и руководства по SAP. Документация корпорации Майкрософт охватывает только отдельные задачи установки и эксплуатации программного обеспечения SAP на виртуальной машине Microsoft Azure.
>
>

Следующие примечания по SAP актуальны для развертывания SAP в Azure.

| Номер примечания | Title |
| --- | --- |
| [1928533] |Приложения SAP в Azure: Поддерживаемые продукты и размеры |
| [2015553] |SAP в Microsoft Azure: необходимые компоненты для поддержки |
| [1999351] |Устранение неполадок, связанных с расширенным мониторингом Azure для SAP |
| [2178632] |Ключевые метрики мониторинга для SAP в Microsoft Azure |
| [1409604] |Виртуализация в Windows: расширенный мониторинг |
| [2191498] |SAP на платформе Linux в Azure: расширенный мониторинг |
| [2243692] |Linux на виртуальной машине Microsoft Azure (IaaS): проблемы с лицензированием SAP |
| [1984787] |SUSE Linux Enterprise Server 12 Замечания по установке |
| [2002167] |Red Hat Enterprise Linux 7.x: установка и обновление |
| [2069760] |Установка и обновление Oracle Linux 7.x SAP |
| [1597355] |Рекомендация по области буфера для Linux |

Ознакомьтесь также с разделом [вики-сайта SCN](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes), где представлены все примечания к SAP для Linux.

Общие ограничения по умолчанию и максимальные ограничения для подписок Azure можно найти в [этой статье][azure-resource-manager/management/azure-subscription-service-limits-subscription].

## <a name="possible-scenarios"></a>Возможные сценарии
SAP часто рассматривается как одно из критически важных приложений на предприятиях. Архитектура и функционирование таких приложений чаще всего сложны, поэтому важно обеспечить выполнение требований по доступности и производительности.

Таким образом, предприятия должны тщательно думать, какого поставщика облачных служб следует выбрать для выполнения этих критически важных бизнес-процессов. Azure — это идеальная общедоступная облачная платформа для критически важных для бизнеса приложений SAP и бизнес-процессов. Учитывая широкий спектр инфраструктуры Azure, практически все существующие системы SAP NetWeaver и S/4HANA можно разместить в Azure уже сегодня. Azure предоставляет виртуальные машины с множеством терабайт памяти и более 200 ЦП. Помимо этого, Azure предлагает [крупные экземпляры Hana](./hana-overview-architecture.md), которые позволяют масштабировать крупномасштабные развертывания Hana размером до 24 тб и SAP HANA масштабных развертывания до 120 ТБ. Сегодня можно использовать практически все локальные сценарии SAP, которые также могут быть запущены в Azure.

Более общее описание сценариев и некоторых неподдерживаемых сценариев см. в документе о [рабочей нагрузке SAP в поддерживаемых сценариях виртуальных машин Azure](./sap-planning-supported-configurations.md).

Изучите эти сценарии и некоторые условия, которые указаны как неподдерживаемые в справочной документации, во время планирования и разработки архитектуры, которую вы хотите развернуть в Azure.

В целом обычно используется распределенное развертывание, как показано

![VPN-подключение типа "сеть — сеть" (распределенное подключение)][planning-guide-figure-300]

Многие клиенты применяют шаблон распределенного развертывания, так как он является наиболее прозрачным для всех приложений и позволяет расширить локальную среду в Azure с помощью Azure ExpressRoute, а также рассматривать Azure как виртуальный центр обработки данных. По мере того как все больше ресурсов перемещается в Azure, развернутая инфраструктура и сетевая инфраструктура Azure будут расти, а локальные ресурсы будут соответственно сокращаться. И все это прозрачно для пользователей и приложений.

Чтобы успешно развернуть системы SAP в Azure IaaS или любой другой службе IaaS, важно понимать, что предложения обычных аутсорсеров или хостеров существенно отличаются от предложений IaaS. Обычный хостер или аутсорсер будет адаптировать инфраструктуру (сеть, хранилище и тип сервера) для размещаемой клиентом рабочей нагрузки, а в развертывании IaaS клиент или его партнер отвечает за определение рабочей нагрузки и выбор подходящих компонентов Azure для виртуальных машин, хранилища и сети.

Чтобы собрать данные для планирования развертывания в Azure, выполните следующие действия.

- Оцените, какие продукты SAP поддерживаются на виртуальных машинах Azure.
- Оцените, какие выпуски операционной системы поддерживаются конкретными виртуальными машинами Azure для этих продуктов SAP.
- Оцените, какие выпуски СУБД поддерживаются для продуктов SAP с конкретными виртуальными машинами Azure.
- Оцените, требуется ли для некоторых обязательных выпусков ОС и СУБД выпуск SAP, обновление пакетов поддержки и обновление ядра, чтобы получить поддерживаемую конфигурацию.
- Оцените, нужно ли переходить на другие операционные системы для развертывания в Azure.

Сведения о поддерживаемых компонентах SAP в Azure, поддерживаемых единицах инфраструктуры Azure и связанных выпусках операционных систем и выпусках СУБД описаны в статье [Программное обеспечение SAP, поддерживаемое для развертываний Azure](./sap-supported-product-on-azure.md). Результаты оценки допустимых выпусков SAP, операционной системы и выпусков СУБД оказывают значительное влияние на действия по переносу систем SAP в Azure. Результаты этой оценки определяют, нужна ли масштабная подготовка в случае, когда требуется обновление выпуска SAP или изменение операционных систем.


## <a name="azure-regions"></a><a name="be80d1b9-a463-4845-bd35-f4cebdb5424a"></a>Регионы Azure
Службы Microsoft Azure собираются в регионах Azure. Регион Azure — это одна или несколько коллекций из центров обработки данных, содержащих оборудование и инфраструктуру, где размещены различные службы Azure. Эта инфраструктура включает большое количество узлов, которые работают как ресурсы вычислений или узлы хранилища, а также выполняют сетевые функции.

Список различных регионов Azure см. в статье [География Azure](https://azure.microsoft.com/global-infrastructure/geographies/). Не все регионы Azure предлагают одни и те же службы. В зависимости от продукта SAP, который вы хотите запустить, и операционной системы и СУБД, связанных с ним, вы можете столкнуться с ситуацией, когда в определенном регионе не предлагаются требуемые типы виртуальных машин. Особенно это происходит при запуске SAP HANA, где обычно требуются виртуальные машины серии M/Mv2. Эти семейства виртуальных машин развертываются только в некоторых регионах. Вы можете узнать, какие именно виртуальные машины, их типы, типы хранилищ Azure или другие службы Azure доступны в тех или иных регионах, на сайте [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/). Начиная планировать и рассматривая некоторые регионы в качестве основных и дополнительных, сначала узнайте, доступны ли необходимые службы в этих регионах.

### <a name="availability-zones"></a>зоны доступности;
Некоторые регионы Azure реализовали концепцию, именуемую Зонами доступности. Зоны доступности — это физически разделенные расположения в пределах одного региона Azure. Каждая зона доступности состоит из одного или нескольких центров обработки данных, оснащенных независимыми системами электроснабжения, охлаждения и сетевого взаимодействия. Например, развертывание двух виртуальных машин в двух Зонах доступности Azure и реализация инфраструктуры высокого уровня доступности для системы СУБД SAP или центральных служб SAP обеспечивает наилучшие показатели соглашения об уровне обслуживания в Azure. Для этого конкретного соглашения об уровне обслуживания для виртуальных машин в Azure проверьте последнюю версию [Соглашения об уровне обслуживания для виртуальных машин](https://azure.microsoft.com/support/legal/sla/virtual-machines/). Так как регионы Azure быстро развиваются и расширяются в последние годы, топология регионов Azure, количество физических центров обработки данных, расстояние между ними и расстояние между Зонами доступности Azure может отличаться. Задержки в сети также будут разными.

Принцип Зон доступности не применяется к службе HANA [крупных экземпляров HANA](./hana-overview-architecture.md). Соглашения об уровне обслуживания для крупных экземпляров HANA можно найти в статье [Соглашение об уровне обслуживания для SAP HANA на крупных экземплярах Azure](https://azure.microsoft.com/support/legal/sla/sap-hana-large/)


### <a name="fault-domains"></a><a name="df49dc09-141b-4f34-a4a2-990913b30358"></a>Домены сбоя
Домены сбоя представляют физическую единицу сбоя. Эта концепция тесно связана с физической инфраструктурой центров обработки данных, например физический блейд сервера или стойка может считаться доменом сбоя. Впрочем, между этими единицами нет однозначной прямой зависимости.

Когда вы развертываете несколько виртуальных машин в рамках одной системы SAP в службах виртуальных машин Microsoft Azure, вы можете потребовать, чтобы контроллер структуры Azure развернул приложение в нескольких доменах сбоя для выполнения требований соглашения об уровне обслуживания. Но при этом вы не можете напрямую контролировать распределение доменов сбоя между единицами масштабирования Azure (наборами из сотен вычислительных узлов, узлов хранения и сетевых узлов) или распределение виртуальных машин между доменам сбоя. Чтобы контроллер структуры Azure развернул набор виртуальных машин в нескольких доменах сбоя, вам следует назначить для виртуальных машин группу доступности Azure во время их развертывания. Дополнительные сведения о группах доступности Azure содержатся в разделе этого документа о [группах доступности Azure][planning-guide-3.2.3].


### <a name="upgrade-domains"></a><a name="fc1ac8b2-e54a-487c-8581-d3cc6625e560"></a>Домены обновления
Домены обновления представляют логическую единицу, на основании которой определяется порядок обновления виртуальных машин, входящих в систему SAP, которая состоит из экземпляров SAP на нескольких виртуальных машинах. Когда происходит обновление, Microsoft Azure выполняет необходимые процессы поочередно для каждого домена обновления. Если вы во время развертывания распределите виртуальные машины по разным доменам обновления, это частично защитит систему SAP от потенциального простоя. Чтобы система Azure развернула виртуальные машины системы SAP в нескольких доменах обновления, следует задать специальный атрибут во время развертывания каждой виртуальной машины. Как и в случае с доменами сбоя, каждая единица масштабирования Azure включает множество доменов обновления. Чтобы контроллер структуры Azure развернул набор виртуальных машин в нескольких доменах обновления, вам следует назначить для виртуальных машин группу доступности Azure во время их развертывания. Дополнительные сведения о группах доступности Azure содержатся ниже в разделе о [группах доступности Azure][planning-guide-3.2.3].


### <a name="azure-availability-sets"></a><a name="18810088-f9be-4c97-958a-27996255c665"></a>Группы доступности Azure
Виртуальные машины Azure, входящие в одну группу доступности Azure, распределяются контроллером структуры Azure по разным доменам обновления и сбоя. Распределение по разным доменам обновления и сбоя нужно для того, чтобы виртуальные машины системы SAP не завершали работу одновременно в ходе обслуживания инфраструктуры или в случае сбоя в одном домене сбоя. По умолчанию виртуальные машины не входят в группу доступности. Участие виртуальной машины в группе доступности определяется во время развертывания. Это поведение можно изменить позднее, изменив настройки и повторно развернув виртуальную машину.

Чтобы получить общее представление о группах доступности Azure и их взаимосвязи с доменами сбоя и обновления, ознакомьтесь с [этой статьей][virtual-machines-manage-availability].

При определении групп доступности и попытке смешать различные виртуальные машины из разных семейств виртуальных машин в одной группе доступности могут возникнуть проблемы, которые не позволяют включить определенный тип виртуальной машины в такую группу доступности. Причина заключается в том, что группа доступности привязана к единице масштабирования, содержащей определенный тип узлов вычислений. А определенный тип узла вычислений может выполнять только определенные типы семейств виртуальных машин. Например, если вы создаете группу доступности и развертываете первую виртуальную машину в группе доступности, а также выбираете тип виртуальной машины семейства Esv3, а затем выбираете семейство M в качестве второй виртуальной машины, вторая операция будет отклонена. Причина заключается в том, что виртуальные машины семейства Esv3 не работают на одном оборудовании с виртуальными машинами семейства M. Эта же проблема может возникнуть при попытке изменить размер виртуальных машин и переместить виртуальную машину семейства Esv3 в тип виртуальной машины семейства M. В случае изменения размера для семейства виртуальных машин, которое не может быть размещено на одном и том же оборудовании, необходимо завершить работу всех виртуальных машин в группе доступности и изменить их размер, чтобы они могли работать на другом типе компьютера размещения. Для соглашений об уровне обслуживания виртуальных машин, развернутых в группе доступности, ознакомьтесь со статьей [Соглашения об уровне обслуживания для виртуальных машин](https://azure.microsoft.com/support/legal/sla/virtual-machines/).

Принцип группы доступности и связанного с ней домена обновления и сбоя не применяется к службе HANA [крупных экземпляров HANA](./hana-overview-architecture.md). Соглашения об уровне обслуживания для крупных экземпляров HANA можно найти в статье [Соглашение об уровне обслуживания для SAP HANA на крупных экземплярах Azure](https://azure.microsoft.com/support/legal/sla/sap-hana-large/).

> [!IMPORTANT]
> Понятия Зон доступности Azure и групп доступности Azure являются взаимоисключающими. Это означает, что пару или несколько виртуальных машин можно развернуть либо в определенной Зоне доступности, либо в группе доступности Azure. Но не в обоих расположениях.

### <a name="azure-paired-regions"></a>Парные регионы Azure
Azure предлагает пары регионов Azure, в которых для этих пар фиксированного региона включена репликация определенных данных. Связывание регионов описано в статье [непрерывность бизнес-процессов и аварийное восстановление (BCDR): Парные регионы Azure](../../../best-practices-availability-paired-regions.md). Как описано в статье, репликация данных связана с типами хранилища Azure, которые можно настроить для репликации в парный регион. См. также [избыточность хранилища статей в дополнительном регионе](../../../storage/common/storage-redundancy.md#redundancy-in-a-secondary-region). Типы хранилища, допускающие такую репликацию, — это типы хранения, которые не подходят для рабочей нагрузки СУБД. Так как такое удобство использования репликации службы хранилища Azure будет ограничено хранилищем BLOB-объектов Azure (например, резервное копирование) или другими сценариями хранения с высокой задержкой. При проверке парных регионов и служб, которые вы хотите использовать в качестве основного или дополнительного региона, могут возникнуть ситуации, когда службы Azure и (или) типы виртуальных машин, которые вы планируете использовать в основном регионе, недоступны в парном регионе. Или может возникнуть ситуация, когда парный регион Azure не является приемлемым из соображений соответствия данных. В таких случаях необходимо использовать непарный регион в качестве дополнительного региона или области аварийного восстановления. В этом случае необходимо принять во внимание репликацию части данных, которые Azure будет реплицировать самостоятельно. Пример репликации Active Directory и DNS в регион аварийного восстановления описан в статье [Настройка аварийного восстановления для Active Directory и DNS](../../../site-recovery/site-recovery-active-directory.md) .


## <a name="azure-virtual-machine-services"></a>Службы виртуальных машин Azure
Azure предлагает различные типы виртуальных машин на выбор. Вам не потребуется инвестировать в технологии и инфраструктуру. Служба виртуальных машин Azure упрощает обслуживание и эксплуатацию приложений. Платформа по запросу выделяет вычислительные ресурсы и хранилище для размещения и масштабирования веб-приложений и подключенных приложений, а также управления ими. Платформа также позволяет автоматизировать управление инфраструктурой, обеспечивая высокий уровень доступности и динамическое масштабирование в соответствии с потребностями. Доступны различные модели оплаты.

![Позиционирование служб виртуальных машин Microsoft Azure][planning-guide-figure-400]

На базе виртуальных машин Azure корпорация Майкрософт позволяет развертывать в Azure пользовательские серверные образы как экземпляры IaaS. Также вам доступно множество вариантов образов операционных систем из коллекции образов Azure.

С точки зрения эксплуатации службы виртуальных машин Azure мало чем отличаются от виртуальных машин, развернутых на локальном компьютере. Вы несете ответственность за администрирование, выполнение операций, а также установку исправлений определенной операционной системы, работающей на виртуальной машине Azure, и приложений на этой виртуальной машине. Корпорация Майкрософт не предоставляет других услуг, кроме размещения этой виртуальной машины в своей инфраструктуре Azure (инфраструктура как услуга — IaaS). Для рабочей нагрузки SAP, которую вы развертываете в качестве клиента, корпорация Майкрософт предлагает только IaaS.

Платформа Microsoft Azure является мультитенантной. В результате хранилище, сеть и вычислительные ресурсы, в которых размещаются виртуальные машины Azure, используются клиентами совместно (за некоторыми исключениями). Интеллектуальное регулирование и логика квотирования не позволяют клиентам существенно влиять на производительность, доступную для другого клиента ("шумный сосед"). В частности, для сертификации платформы Azure для SAP HANA корпорации Майкрософт необходимо доказать изоляцию ресурсов в случаях, когда несколько виртуальных машин постоянно могут выполняться на одном узле в SAP. Логика системы Azure стремится к тому, чтобы отклонения показателей пропускной способности оставались минимальными. Однако на платформах с большим количеством соседей отклонения по доступности ресурсов и пропускной способности превышают тот уровень, который привычен для многих пользователей в локальных развертываниях. При планировании следует учитывать вероятность того, что система SAP в Azure будет испытывать более существенные отклонения, чем в локальной системе.

### <a name="azure-virtual-machines-for-sap-workload"></a>Виртуальные машины Azure для рабочей нагрузки SAP

Для рабочей нагрузки SAP мы предлагаем только семейства виртуальных машин, которые подходят для рабочей нагрузки SAP и SAP HANA. Чтобы узнать, как подобрать тип виртуальной машины и проверить его пригодность для рабочей нагрузки SAP, ознакомьтесь с документом [Программное обеспечение SAP, поддерживаемое для развертываний Azure](./sap-supported-product-on-azure.md).

> [!NOTE]
> Типы виртуальных машин, сертифицированные для рабочей нагрузки SAP, не имеют избыточной подготовки ресурсов ЦП и памяти.

Помимо выбора только неподдерживаемых типов виртуальных машин, необходимо также проверить, доступны ли эти типы виртуальных машин в определенном регионе на основе [продуктов сайта, доступных по регионам](https://azure.microsoft.com/global-infrastructure/services/). Что еще важнее, необходимо оценить следующее:

- ресурсы ЦП и памяти для разных типов виртуальных машин;
- пропускная способность операций ввода-вывода для разных типов виртуальных машин;
- сетевые возможности для разных типов виртуальных машин;
- количество подключаемых дисков;
- возможность использовать определенные типы хранилищ Azure;

соответствие вашим потребностям. Большую часть этих данных можно найти [здесь (Linux)][virtual-machines-sizes-linux] и [здесь (Windows)][virtual-machines-sizes-windows].

В качестве модели ценообразования можно выбрать один из нескольких вариантов, например:

- с оплатой по мере использования;
- резерв на 1 год;
- резерв на 3 года;
- точечные виртуальные машины.

Цены на каждое предложение с различными службами для операционных систем и регионов доступны на сайте [Цены на виртуальные машины Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) и [Цены на виртуальные машины Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). Дополнительные сведения и преимущества экземпляров, зарезервированных на один и на три года, приводятся в следующих статьях:

- [Основные сведения о резервировании в Azure](../../../cost-management-billing/reservations/save-compute-costs-reservations.md)
- [Гибкость размеров виртуальных машин при использовании зарезервированных экземпляров виртуальных машин](../../reserved-vm-instance-size-flexibility.md)
- [Применения скидки на резервирование Azure к виртуальным машинам](../../../cost-management-billing/manage/understand-vm-reservation-charges.md)

Дополнительные сведения о ценах на точечные виртуальные машины см. в статье [Точечные виртуальные машины Azure](https://azure.microsoft.com/pricing/spot/). Цены на один и тот же тип виртуальной машины могут отличаться в разных регионах Azure. Некоторые клиенты могут выбрать менее дорогостоящий регион Azure.

Кроме того, Azure предлагает выделенный узел. Концепция выделенного узла предоставляет больше возможностей управления циклами исправлений в Azure. Вы можете выбирать время установки исправлений в соответствии с собственными графиками. Это предложение специально предназначено для клиентов с рабочей нагрузкой, которая может не соответствовать нормальному циклу. Чтобы узнать больше о предложениях выделенного узла Azure, прочитайте статью [Выделенный узел Azure](../../dedicated-hosts.md). Использование этого предложения поддерживается для рабочей нагрузки SAP и предназначено для нескольких клиентов SAP, которым требуется больше контроля над установкой исправлений инфраструктуры и планами обслуживания корпорации Майкрософт. Дополнительные сведения о том, как корпорация Майкрософт сохраняет и обновляет инфраструктуру Azure, в которой размещены виртуальные машины, см. в статье [Обслуживание виртуальных машин в Azure](../../maintenance-and-updates.md).

#### <a name="generation-1-and-generation-2-virtual-machines"></a>Виртуальные машины поколений 1 и 2
Гипервизор Майкрософт может работать с двумя различными поколениями виртуальных машин. Эти форматы называются **Поколение 1** и **Поколение 2**. **Поколение 2** было представлено в 2012 году с гипервизором Windows Server 2012. Изначально в Azure использовались виртуальные машины поколения 1. При развертывании виртуальных машин Azure по умолчанию по-прежнему используется формат поколения 1. При этом можно развернуть виртуальные машины поколения 2. В статье [Поддержка виртуальных машин поколения 2 в Azure](../../generation-2.md) перечислены семейства виртуальных машин Azure, которые можно развернуть как виртуальную машину поколения 2. В этой статье также перечислены важные функциональные различия виртуальных машин версии 2, которые можно выполнять в частном облаке Hyper-V и Azure. Кроме того, в этой статье перечислены функциональные различия между виртуальными машинами поколения 1 и 2 в Azure.

> [!NOTE]
> Между виртуальными машинами поколения 1 и 2, работающими в Azure, существуют функциональные различия. Список этих различий см. в статье [Поддержка виртуальных машин поколения 2 в Azure](../../generation-2.md).

Перемещение существующей виртуальной машины из одного поколения в другое невозможно. Чтобы изменить поколение виртуальных машин, необходимо развернуть новую виртуальную машину необходимого поколения и повторно установить программное обеспечение, которое выполняется на виртуальной машине поколения. Это изменение влияет только на базовый образ VHD виртуальной машины и не влияет на диски данных, а также на подключенные общие папки NFS или SMB. Диски данных, NFS или общие папки SMB, которые были назначены, например, на виртуальной машине поколения 1.

> [!NOTE]
> Развертывание виртуальных машин семействаMv1 в качестве виртуальных машин поколения 2 возможно с начала мая 2020 года. Благодаря этому можно уменьшить разницу между виртуальными машинами Mv1 и Mv2.


### <a name="storage-microsoft-azure-storage-and-data-disks"></a><a name="a72afa26-4bf4-4a25-8cf7-855d6032157f"></a>Хранилище. Служба хранилища Microsoft Azure и диски данных
Виртуальные машины Microsoft Azure используют хранилища разных типов. При реализации SAP в службах виртуальных машин Azure важно понимать различия между двумя основными типами хранилищ:

* непостоянное (временное) хранилище;
* постоянное хранилище.

Виртуальные машины Azure предлагают временные диски после развертывания виртуальной машины. В случае перезагрузки виртуальной машины все содержимое временных дисков будет удалено. Поэтому на этих дисках ни в коем случае не следует замещать файлы данных, файлы журнала и файлы повторных операций баз данных. Для некоторых баз данных возможны исключения: для них разрешается размещать на временных дисках табличные пространства tempdb и temp. Однако не следует использовать эти диски для виртуальных машин серии A, так как пропускная способность временных дисков с этим семейством виртуальных машин ограничена. Дополнительные сведения см. в статье [Сведения о временных дисках на виртуальных машинах Windows в Azure](/archive/blogs/mast/understanding-the-temporary-drive-on-windows-azure-virtual-machines)

---
> ![Эмблема Windows.][Logo_Windows] Windows
>
> D:\ на виртуальной машине Azure — это несохраняемый диск, поддерживаемый некоторыми локальными дисками на вычислительном узле Azure. Несохраняемый означает, что любые изменения содержимого диска D:\ теряются при перезагрузке виртуальной машины. Под любыми изменениями подразумевается сохранение файлов, создание каталогов, установка приложений и т. д.
>
> ![Логотип Linux.][Logo_Linux] Linux
>
> Виртуальные машины Azure под управлением Linux автоматически подключают диск к каталогу /mnt/resource — несохраняемому диску, который поддерживается локальными дисками на вычислительном узле Azure. Несохраняемый обозначает, что любые изменения содержимого каталога /mnt/resource теряются при перезагрузке виртуальной машины. Под любыми изменениями подразумевается сохранение файлов, создание каталогов, установка приложений и т. д.
>
>

#### <a name="azure-storage-accounts"></a>Учетные записи хранения Azure

При развертывании служб или виртуальных машин в Azure развертывание виртуальных жестких дисков и образов виртуальных машин организуется в единицы, называемые учетными записями хранения Azure. [Учетные записи хранения Azure](../../../storage/common/storage-account-overview.md) имеют ограничения в операциях ввода-вывода, пропускной способности или размерах, которые может содержать. В прошлом эти ограничения описаны в:

- [Целевые показатели масштабируемости для учетных записей хранения уровня "Стандартный"](../../../storage/common/scalability-targets-standard-account.md)
- [Целевые показатели масштабируемости для учетных записей хранилища BLOB-объектов уровня "Премиум"](../../../storage/blobs/scalability-targets-premium-page-blobs.md)

играют важную роль в планировании развертывания SAP в Azure. Это было необходимо для управления количеством сохраненных дисков в учетной записи хранения. Вам необходимо было управлять учетными записями хранения и создавать новые учетные записи хранения для создания более материализованных дисков.

В последние годы введение [управляемых дисков Azure](../../managed-disks-overview.md) было освобождено от этих задач. Рекомендации по развертыванию SAP — это использование управляемых дисков Azure вместо управления учетными записями хранения Azure. Управляемые диски Azure распределяют диски между разными учетными записями хранения, поэтому ограничения отдельных учетных записей хранения не превышены.

В учетной записи хранения имеется тип концепции папки "контейнеры", которую можно использовать для группировки определенных дисков в определенные контейнеры.

В Azure имя диска или виртуального жесткого диска следует за следующим подключением к именованию, которое должно предоставить уникальное имя для виртуального жесткого диска в Azure:

`http(s)://<storage account name>.blob.core.windows.net/<container name>/<vhd name>`

Приведенная выше строка должна уникальным образом идентифицировать диск или виртуальный носитель, хранящийся в службе хранилища Azure.


#### <a name="azure-persisted-storage-types"></a>Типы материализованных хранилищ Azure
Azure предлагает целый ряд сохраненных вариантов хранения, которые можно использовать для рабочей нагрузки SAP и конкретных компонентов стека SAP. Дополнительные сведения см. в статье Документирование  [хранилища Azure для рабочих нагрузок SAP](./planning-guide-storage.md).


### <a name="microsoft-azure-networking"></a><a name="61678387-8868-435d-9f8c-450b2424f5bd"></a>Сеть Microsoft Azure

Microsoft Azure — это сетевая инфраструктура, которая позволяет реализовать любые сценарии, используемые для программного обеспечения SAP. Ключевые возможности:

* прямой внешний доступ к виртуальным машинам через службы терминалов Microsoft Windows или ssh/VNC;
* доступ к службам и конкретным портам, которые используются приложениями на виртуальных машинах;
* внутреннее взаимодействие и разрешение имен между группами виртуальных машин, развернутых в Azure;
* распределенное подключение локальной сети клиента к сети Azure;
* подключения между сайтами Azure в разных регионах Azure или в разных центрах обработки данных.

Дополнительные сведения см. здесь: <https://azure.microsoft.com/documentation/services/virtual-network/>.

Есть множество разных способов настроить в Azure разрешение имен и IP-адресов. Есть также возможность использовать службу Azure DNS вместо настройки собственного DNS-сервера. Дополнительные сведения см. в [этой статье][virtual-networks-manage-dns-in-vnet] и на [этой странице](https://azure.microsoft.com/services/dns/).

Для сценариев распределенного или гибридного развертывания мы предполагаем, что локальная служба AD, OpenLDAP или DNS подключена к Azure через VPN-подключение или частное подключение. Для некоторых описанных здесь сценариев может потребоваться установить в Azure реплику AD или OpenLDAP.

Так как подключение к сети и разрешение имен играют важную роль для развертывания базы данных для системы SAP, эта концепция более подробно описана в [руководстве по развертыванию СУБД][dbms-guide].

##### <a name="azure-virtual-networks"></a>Виртуальные сети Azure

Вы можете создать виртуальную сеть Azure, чтобы определить диапазон частных IP-адресов, назначаемых Azure DHCP. В сценариях распределенного подключения диапазон IP-адресов также выделяется средой Azure с использованием DHCP. Однако разрешение доменных имен будет выполняться локально (если виртуальные машины являются частью локального домена), что позволяет разрешать адреса, не входящие в облачные службы Azure.

Все виртуальные машины в Azure должны быть подключены к виртуальной сети.

Дополнительные сведения см. в [этой статье][resource-groups-networking] и на [этой странице](https://azure.microsoft.com/documentation/services/virtual-network/).


> [!NOTE]
> По умолчанию после развертывания виртуальной машины вы не можете изменять конфигурацию виртуальной сети. Параметры TCP/IP следует оставить под управлением DHCP-сервера Azure. Поведение по умолчанию — назначение динамических IP-адресов.
>
>

MAC-адрес виртуального сетевого адаптера может измениться, например после повторного изменения размера. В этом случае гостевая ОС Windows или Linux получает новый сетевой адаптер и автоматически использует DHCP для назначения адресов IP и DNS.

##### <a name="static-ip-assignment"></a>Назначение статического IP-адреса
Виртуальным машинам в виртуальной сети Azure можно назначить фиксированный или зарезервированный IP-адрес. Для виртуальных машин, включенных в виртуальную сеть Azure, при необходимости можно использовать эту удобную возможность. Назначенные IP-адреса сохраняются, пока существует виртуальная машина, независимо от того, работает она или отключена. Поэтому при определении диапазона IP-адресов для виртуальной сети следует учитывать общее число виртуальных машин (как работающих, так и остановленных). IP-адрес остается назначенным, пока не будут удалены виртуальная машина и ее сетевой интерфейс или пока не будет отменен IP-адрес. Дополнительные сведения см. в [этой статье][virtual-networks-static-private-ip-arm-pportal].

> [!NOTE]
> Отдельным виртуальным сетевым адаптерам следует назначить статические IP-адреса с помощью Azure. Не следует назначать статические IP-адреса для виртуальных сетевых адаптеров в гостевой ОС. Некоторые службы Azure, такие как служба Azure Backup, полагаются на тот факт, что по крайней мере для основного виртуального сетевого адаптера используется DHCP, а не статические IP-адреса. См. дополнительные сведения в статье [Устранение неполадок при архивации виртуальных машин Azure](../../../backup/backup-azure-vms-troubleshoot.md#networking).
>
>

##### <a name="multiple-nics-per-vm"></a>Несколько сетевых адаптеров на виртуальной машине

Для виртуальной машины Azure вы можете определить несколько виртуальных сетевых адаптеров (vNIC). Такая возможность позволяет настроить разделение сетевого трафика. Например, клиентский трафик можно направить через один виртуальный сетевой адаптер, а серверную часть трафика — через второй виртуальный сетевой адаптер. В зависимости от типа виртуальной машины есть разные ограничения на количество виртуальных сетевых адаптеров, которые можно назначить виртуальной машине. Подробные сведения о параметрах, функциональных возможностях и ограничениях см. в следующих статьях:

* [Создание виртуальной машины Windows, использующей несколько сетевых адаптеров][virtual-networks-multiple-nics-windows]
* [Создание виртуальной машины Linux, использующей несколько сетевых адаптеров][virtual-networks-multiple-nics-linux]
* [Развертывание виртуальных машин с несколькими сетевыми адаптерами с помощью шаблона][virtual-network-deploy-multinic-arm-template]
* [Создание виртуальной машины с несколькими сетевыми адаптерами с помощью PowerShell][virtual-network-deploy-multinic-arm-ps]
* [Развертывание виртуальных машин с несколькими сетевыми адаптерами с помощью Azure CLI][virtual-network-deploy-multinic-arm-cli]

#### <a name="site-to-site-connectivity"></a>Подключение типа "сеть — сеть"

Распределенное развертывание — это виртуальные машины Azure, подключенные к локальной сети с использованием прозрачного и постоянного VPN-подключения. Мы полагаем, что такая модель будет самой распространенной для развертываний SAP в Azure. Предполагается, что все рабочие процедуры и процессы с экземплярами SAP в Azure будут работать прозрачно. Это означает, что вы сможете распечатывать данные из этих систем, а также использовать систему управления транспортом SAP (TMS) для передачи изменений из системы разработки, размещенной в Azure, в локально развернутую тестовую систему. Дополнительную документацию по подключениям типа "сеть — сеть" можно найти в [этой статье][vpn-gateway-create-site-to-site-rm-powershell].

##### <a name="vpn-tunnel-device"></a>Устройство для VPN-туннеля

Чтобы создать подключение типа "сеть — сеть" (между локальным центром обработки данных и центром обработки данных Azure), следует получить и настроить VPN-устройство или использовать службу маршрутизации и удаленного доступа (RRAS), которая появилась в качестве компонента программного обеспечения в Windows Server 2012.

* [Создание виртуальной сети с VPN-подключением типа "сеть — сеть" с помощью PowerShell][vpn-gateway-create-site-to-site-rm-powershell]
* [О VPN-устройствах для подключений VPN-шлюзов типа "сеть — сеть"][vpn-gateway-about-vpn-devices]
* [VPN-шлюз: вопросы и ответы][vpn-gateway-vpn-faq]

![Подключение типа "сеть — сеть" между локальной средой и Azure][planning-guide-figure-600]

На рисунке выше показаны две подписки Azure с поддиапазонами IP-адресов, которые зарезервированы в Azure для использования в виртуальных сетях. Подключение между локальной сетью и Azure устанавливается через VPN.

#### <a name="point-to-site-vpn"></a>VPN-подключение типа "точка — сеть"

VPN-подключение типа "точка — сеть" подразумевает, что каждый клиентский компьютер подключается к Azure через собственное VPN-подключение. Для сценариев SAP, которые мы рассматриваем, подключение типа "точка — сеть" нецелесообразно. Поэтому далее мы не будем рассматривать VPN-подключения типа "точка — сеть".

Дополнительные сведения см. здесь:
* [Настройка подключения типа "точка — сеть" к виртуальной сети с помощью портала Azure](../../../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md)
* [Настройка подключения типа "точка — сеть" к виртуальной сети с помощью PowerShell](../../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)

#### <a name="multi-site-vpn"></a>Многосайтовая VPN

Также Azure теперь позволяет создать многосайтовое VPN-подключение для одной подписки Azure. Ранее для отдельной подписки можно было создавать только VPN-подключения типа "сеть — сеть". Теперь это ограничение снято — многосайтовые VPN-подключения поддерживаются и для отдельных подписок. Это позволяет использовать более одного региона Azure в конфигурации распределенного развертывания для одной подписки.

Дополнительную документацию можно найти в [этой статье][vpn-gateway-create-site-to-site-rm-powershell].

#### <a name="vnet-to-vnet-connection"></a>Подключение типа "виртуальная сеть — виртуальная сеть"

Если вы используете многосайтовое VPN-подключение, для каждого из регионов следует настроить отдельную виртуальную сеть Azure. Но при этом часто требуется, чтобы программные компоненты в различных регионах могли взаимодействовать друг с другом. В идеале такое подключение не должно маршрутизироваться из одного региона Azure в другой регион Azure через локальную среду. Чтобы упростить этот сценарий, Azure позволяет настроить подключение между двумя виртуальными сетями Azure, расположенными в разных регионах. Такая конфигурация называется подключением типа "виртуальная сеть — виртуальная сеть". Дополнительные сведения об этих функциях см. здесь: <https://azure.microsoft.com/documentation/articles/vpn-gateway-vnet-vnet-rm-ps/>.

#### <a name="private-connection-to-azure-expressroute"></a>Частное подключение к Azure с использованием ExpressRoute

Microsoft Azure ExpressRoute позволяет создавать частные подключения между центрами обработки данных Azure и локальной инфраструктурой клиента или средой совместного размещения. ExpressRoute предоставляется различными поставщиками MPLS VPN (с пакетной коммутацией) или другими поставщиками сетевых услуг. Подключения ExpressRoute не проходят через общедоступный Интернет. Подключения ExpressRoute за счет использования нескольких параллельных контуров обеспечивают более высокий уровень безопасности, надежности и быстродействия, а также меньшие задержки по сравнению с обычными подключениями через Интернет.

Дополнительные сведения об Azure ExpressRoute и доступных предложениях см. здесь:

* <https://azure.microsoft.com/documentation/services/expressroute/>
* <https://azure.microsoft.com/pricing/details/expressroute/>
* <https://azure.microsoft.com/documentation/articles/expressroute-faqs/>

ExpressRoute позволяет подключать несколько подписок Azure через один канал ExpressRoute, как описано здесь:

* <https://azure.microsoft.com/documentation/articles/expressroute-howto-linkvnet-arm/>
* <https://azure.microsoft.com/documentation/articles/expressroute-howto-circuit-arm/>

#### <a name="forced-tunneling-in-case-of-cross-premises"></a>Принудительное туннелирование для распределенного развертывания
Если вы подключаете виртуальные машины к локальному домену через подключение типа "сеть — сеть", "точка — сеть" или ExpressRoute, обязательно проверьте, чтобы параметры Интернет-прокси были развернуты и для всех пользователей этих виртуальных машин. По умолчанию программное обеспечение, работающее на этих виртуальных машинах, и пользователи, использующие браузер, подключаются к Интернету не через прокси-сервер организации, а напрямую через Azure. Но даже настройка прокси не позволяет на 100 % решить задачу маршрутизации трафика через прокси-сервер организации, так как программы и службы самостоятельно определяют, будут ли они использовать прокси. Если программное обеспечение, работающее на виртуальной машине, не использует эти настройки, или администратор изменит параметры, трафик будет направляться в Интернет непосредственно через Azure.

Чтобы избежать прямого подключения к Интернету, вы можете настроить принудительное туннелирование для подключения типа "сеть — сеть" между локальной сетью и Azure. Подробное описание функции принудительного туннелирования опубликовано здесь: <https://azure.microsoft.com/documentation/articles/vpn-gateway-forced-tunneling-rm/>.

Принудительное туннелирование для ExpressRoute применяется другим способом: клиенты объявляют маршруты по умолчанию с помощью сеансов пиринга BGP ExpressRoute.

#### <a name="summary-of-azure-networking"></a>Общие сведения о сети Azure

В этом разделе собраны ключевые замечания о сетевых возможностях Azure. Вот самые важные из них:

* Виртуальные сети Azure позволяют поместить структуру сети в развертывание Azure. Виртуальные сети можно изолировать друг от друга или контролировать трафик между ними с помощью групп безопасности сети.
* виртуальные сети Azure можно использовать, чтобы назначать виртуальным машинам диапазоны IP-адресов или статические IP-адреса;
* чтобы настроить подключение типа "сеть — сеть" или "точка — сеть", сначала следует создать виртуальную сеть Azure;
* после развертывания виртуальной машины вы уже не сможете изменить виртуальную сеть, которая для нее назначена.

### <a name="quotas-in-azure-virtual-machine-services"></a>Квоты в службах виртуальных машин Azure
Нужно четко понимать, что инфраструктура хранилища и сети совместно используется виртуальными машинами, на которых запущены разные службы в инфраструктуре Azure. Как и в собственных центрах обработки данных клиента, чрезмерная подготовка некоторых ресурсов инфраструктуры выполняется в определенной степени. Платформа Microsoft Azure использует квоты на ресурсы диска, ЦП, сети (и некоторые другие ресурсы), чтобы ограничивать потребление ресурсов и поддерживать стабильную и предсказуемую производительность.  Разные типы виртуальных машин (A5, A6 и т. д.) имеют разные квоты на количество дисков, а также использование ЦП, памяти и сети.

> [!NOTE]
> Ресурсы ЦП и памяти для типов виртуальных машин, поддерживаемых SAP, предварительно выделяются на главных узлах. Это означает, что доступность определенных ресурсов для развернутой виртуальной машины зависит от ее типа.
>
>

При планировании и определении размеров решений SAP в Azure следует учитывать квоты для каждого размера виртуальной машины. Квоты для виртуальных машин описаны [здесь (Linux)][virtual-machines-sizes-linux] и [здесь (Windows)][virtual-machines-sizes-windows].

Приведенные квоты определяют теоретические максимальные значения.  Максимальное количество операций ввода-вывода в секунду на диск может быть достигнуто при малых операциях ввода/вывода (8 КБ), но, возможно, не достигается при больших объемах операций ввода-вывода (1 МБ).  Ограничение на количество операций ввода-вывода применяется с детализацией по каждому диску.

С помощью приведенного ниже дерева принятия решений можно приблизительно оценивать возможность развертывания системы SAP в службах виртуальных машин Azure с предоставленными возможностями, а также необходимость изменения конфигурации существующей системы для развертывания в Azure.

![Дерево принятия решения о возможности развертывания SAP в Azure][planning-guide-figure-700]

1. Наиболее важные сведения для начала работы — это требования SAPS для конкретной системы SAP. Требования SAPS следует оценивать раздельно по уровням СУБД и приложения SAP, даже если система SAP уже развернута локально в двухуровневой конфигурации. Для существующих систем требования SAPS в отношении используемого оборудования обычно можно определить на основе измерений производительности SAP. Результаты можно найти [здесь](https://sap.com/about/benchmark.html). Для вновь развертываемых систем SAP следует выполнить оценку размеров, которая позволит определить требования SAP для системы.
1. Для существующих систем следует измерить объем операций ввода-вывода и их количество в секунду для сервера СУБД. Для планируемых систем общее представление о требованиях к объему операций ввода-вывода на стороне СУБД также можно получить, оценивая размеры. Если вы не уверены в результатах, выполните подтверждение концепции.
1. Сравните требования SAPS для сервера СУБД с возможностями SAPS, которые Azure предлагает для разных типов виртуальных машин. Сведения о SAPS для различных типов виртуальных машин Azure см. в примечании к SAP [1928533]. Важно сначала оценить виртуальную машину для СУБД, так как уровень базы данных в системе SAP NetWeaver для большинства развертываний не подлежит горизонтальному увеличению масштаба. В то же время уровень приложения SAP можно легко масштабировать. Если ни один из типов виртуальных машин Azure, поддерживаемых SAP, не обеспечивает необходимые возможности SAPS, рабочая нагрузка планируемой системы SAP не может выполняться в Azure. В этом случае следует развертывать систему локально или изменить объем рабочей нагрузки для системы.
1. Как описано [здесь (Linux)][virtual-machines-sizes-linux] и [здесь (Windows)][virtual-machines-sizes-windows], Azure применяет квоту на количество операций ввода-вывода в секунду для каждого диска как в Службе хранилища (цен. категория "Стандартный"), так и в хранилище класса Premium. В зависимости от типа виртуальной машины вы сможете подключить разное количество дисков данных. Таким образом, вы можете вычислить максимальное количество операций ввода-вывода в секунду, достижимое для всех типов виртуальных машин. В зависимости от структуры файлов базы данных вы можете использовать чередующиеся диски для одного тома в гостевой ОС. Но если фактическая скорость операций ввода-вывода в развернутой системе SAP превышает вычисленное максимальное значение для самого крупного типа виртуальных машин Azure, и у вас нет возможности компенсировать это значение увеличением объема памяти, рабочая нагрузка системы SAP может серьезно пострадать. Такое сочетание условий означает, что эту систему не следует развертывать в Azure.
1. Есть вероятность, особенно если система SAP развернута локально в двухуровневой конфигурации, что в Azure систему придется развертывать в трехуровневой конфигурации. На этом шаге следует проверить, существует ли такой компонент на уровне приложения SAP, который нельзя масштабировать и которому недостаточно ресурсов ЦП или памяти, предоставляемых для разных типов виртуальных машин Azure. Если такой компонент существует, систему SAP и ее рабочую нагрузку не удастся развернуть в Azure. Но если вы можете горизонтально увеличить масштаб компонентов приложения SAP на несколько виртуальных машин Azure, значит, система допускает развертывание в Azure.

Если компоненты уровня СУБД и приложения SAP могут выполняться на виртуальных машинах Azure, то следует оценить требуемую конфигурацию по следующим параметрам:

* количество виртуальных машин Azure;
* типы виртуальных машин для отдельных компонентов;
* количество виртуальных жестких дисков для виртуальной машины СУБД, которое обеспечит достаточное количество операций ввода-вывода в секунду.

## <a name="managing-azure-assets"></a>Управление ресурсами Azure

### <a name="azure-portal"></a>Портал Azure

Портал Azure — это один из трех интерфейсов для управления развертываниями виртуальных машин Azure. На портале Azure вы можете выполнять базовые задачи управления, например развертывание виртуальных машин из образов. Кроме того, портал Azure хорошо справляется с такими задачами, как создание учетных записей хранения, виртуальных сетей и других компонентов Azure. Но для таких действий, как отправка виртуального жесткого диска из локальной сети в Azure или копирование виртуального жесткого диска в пределах Azure, лучше применять сторонние инструменты, например PowerShell или интерфейс командной строки.

![Портал Microsoft Azure, обзор виртуальных машин][planning-guide-figure-800]


На портале Azure можно выполнять задачи администрирования и конфигурирования для экземпляра виртуальной машины.

Вы можете перезапустить виртуальную машину или завершить ее работу, а также присоединить, отсоединить или создать диски данных для экземпляра виртуальной машины, записать экземпляр для подготовки образа или изменить размер экземпляра виртуальной машины.

Портал Azure предоставляет базовые средства для развертывания и настройки виртуальных машин и многих других служб Azure. Но на портале Azure представлены не все доступные функции. Например, на портале Azure невозможно выполнить:

* отправку виртуального жесткого диска в Azure;
* копирование виртуальных машин.


### <a name="management-via-microsoft-azure-powershell-cmdlets"></a>Управление с помощью командлетов Microsoft Azure PowerShell

Windows PowerShell — это мощная и расширяемая платформа, широко используемая клиентами для большого количества систем, развернутых в Azure. Если вы установите командлеты PowerShell на свой компьютер, ноутбук или специальную станцию управления, вы сможете выполнять командлеты PowerShell удаленно.

Процессы подготовки локального компьютера или ноутбука для использования командлетов Azure PowerShell и настройки их использования для подписки Azure описаны в [этой статье][powershell-install-configure].

Подробные инструкции по установке, обновлению и настройке командлетов Azure PowerShell см. в [этом разделе руководства по развертыванию][deployment-guide-4.1].

Опыт клиентов подтверждает, что PowerShell остается наиболее мощным инструментом для развертывания виртуальных машин и создания настраиваемых действий для развертывания виртуальных машин. Все клиенты, использующие экземпляры SAP в Azure, применяют командлеты PowerShell в дополнение к действиям на портале, или даже управляют своими развертываниями в Azure исключительно с помощью командлетов PowerShell. Так как командлеты, применяемые для Azure, используют то же соглашение об именовании, что и свыше 2000 командлетов для Windows, их использование становится непростой задачей для администраторов систем под управлением Windows.

Пример см. в этом блоге: <https://blogs.technet.com/b/keithmayer/archive/2015/07/07/18-steps-for-end-to-end-iaas-provisioning-in-the-cloud-with-azure-resource-manager-arm-powershell-and-desired-state-configuration-dsc.aspx>.


Развернуть расширение Azure для SAP (см. раздел этого документа [Расширение Azure для SAP][planning-guide-9.1]) можно только с помощью PowerShell или интерфейса командной строки. Поэтому при развертывании или администрировании системы SAP NetWeaver в Azure нужно обязательно установить и настроить PowerShell или интерфейс командной строки.

Так как Azure расширяет функциональные возможности, будут добавляться новые командлеты PS. Следовательно, командлеты необходимо регулярно обновлять. Имеет смысл не реже раза в месяц проверять доступность новой версии командлетов на странице загрузки Azure <https://azure.microsoft.com/downloads/>. Новая версия просто устанавливается поверх старой.

Общий список команд PowerShell, связанных с Azure, см. здесь: <https://docs.microsoft.com/powershell/azure/>.

### <a name="management-via-microsoft-azure-cli-commands"></a>Управление с помощью интерфейса командной строки Microsoft Azure

Если клиент использует ОС Linux и хочет управлять ресурсами Azure, PowerShell может не подойти. Корпорация Майкрософт предлагает в качестве альтернативы интерфейс командной строки Azure.
Интерфейс командной строки Azure представляет собой набор межплатформенных команд с открытым кодом для работы с платформой Azure. Интерфейс командной строки Azure предоставляет практически те же функции, что и портал Azure.

Сведения об установке, конфигурации и использовании интерфейса командной строки для выполнения задач в Azure см. здесь:

* [Установка классического Azure CLI][xplat-cli]
* [Развертывание виртуальных машин и управление ими с помощью шаблонов Azure Resource Manager и интерфейса командной строки Azure][../../linux/create-ssh-secured-vm-from-template.md]
* [Использование классического Azure CLI для Mac, Linux и Windows с Azure Resource Manager][xplat-cli-azure-resource-manager]

Прочитайте также раздел [Azure CLI для виртуальных машин Linux][deployment-guide-4.5.2] в [руководстве по развертыванию][planning-guide], где описано использование Azure CLI для развертывания расширения Azure для SAP.


## <a name="first-steps-planning-a-deployment"></a>Первые шаги планирования развертывания
Проверка доступности виртуальных машин для запуска SAP выполняется не в первую очередь при планировании развертывания. Первый шаг может отнимать много времени, но это очень важный этап. Вам необходимо совместно с отделами безопасности и соответствия требованиям определить условия развертывания рабочей нагрузки SAP или бизнес-процесса в общедоступном облаке. Если ваша компания уже развертывала другое программное обеспечение в Azure, процесс может быть простым. Если ваша компания только начинает этот процесс, следует обсудить много вопросов, чтобы определить условия безопасности при размещении некоторых данных и бизнес-процессов SAP в общедоступном облаке.

Список вариантов см. в разделе [Предложения для соответствия требованиям от Майкрософт](/microsoft-365/compliance/offering-home).

Другие важные вопросы, например шифрование неактивных или других данных в службе Azure, описаны в статье [Общие сведения о шифровании Azure](../../../security/fundamentals/encryption-overview.md).

Не следует недооценивать этот этап проекта при планировании. Только при наличии соглашения и правил в этой сфере можно переходить к следующему шагу — планированию сетевой архитектуры, развертываемой в Azure.


## <a name="different-ways-to-deploy-vms-for-sap-in-azure"></a>Разные способы развертывания в Azure виртуальных машин для SAP

Из этого раздела вы узнаете о способах развертывания виртуальных машин в Azure. Также здесь рассматриваются дополнительные процедуры подготовки и процессы управления виртуальными жесткими дисками и виртуальными машинами в Azure.

### <a name="deployment-of-vms-for-sap"></a>Развертывание виртуальных машин для SAP

Microsoft Azure предусматривает несколько способов развертывания виртуальных машин и связанных дисков. Важно понимать различия между ними, так как подготовка виртуальной машины может отличаться в зависимости от метода развертывания. Мы рассмотрим следующие базовые сценарии.

#### <a name="moving-a-vm-from-on-premises-to-azure-with-a-non-generalized-disk"></a><a name="4d175f1b-7353-4137-9d2f-817683c26e53"></a>Перемещение виртуальной машины из локальной среды в Azure с помощью специализированного диска

Предположим, вы хотите переместить определенную систему SAP из локальной среды в Azure. Для этого можно передать в Azure виртуальный жесткий диск, который содержит ОС и двоичные файлы SAP и СУБД, а также дополнительные виртуальные жесткие диски с файлами данных и журналов СУБД. В отличие от описанного ниже [сценария 2][planning-guide-5.1.2], вы сохраните на виртуальной машине Azure те же имя узла, идентификатор безопасности SAP и учетные записи пользователей SAP, которые были настроены в локальной среде. В этом случае создание универсального образа не требуется. В разделе [Подготовка виртуальной машины к перемещению из локальной среды в Azure с помощью специализированного диска][planning-guide-5.2.1] этого документа описываются действия по подготовке и передаче в Azure специализированных виртуальных машин или виртуальных жестких дисков. Прочтите раздел [Сценарий 3. Перемещение локальной виртуальной машины с помощью специализированного виртуального жесткого диска Azure с SAP][deployment-guide-3.4] в [руководстве по развертыванию][deployment-guide], где приводятся пошаговые инструкции для развертывания такого образа в Azure.

Другой вариант, который мы не будем обсуждать подробно в этом разделе, использует Azure Site Recovery для репликации серверов приложений SAP NetWeaver и центральных служб SAP NetWeaver в Azure. Не рекомендуется использовать Azure Site Recovery для уровня базы данных и вместо этого использовать механизмы репликации, относящиеся к базам данных, например репликацию системы HANA. Дополнительные сведения см. в разделе [Защита SAP](../../../site-recovery/site-recovery-workload.md#protect-sap) в статье [об аварийном восстановлении для локальных приложений](../../../site-recovery/site-recovery-workload.md) .

#### <a name="deploying-a-vm-with-a-customer-specific-image"></a><a name="e18f7839-c0e2-4385-b1e6-4538453a285c"></a>Развертывание виртуальной машины с помощью пользовательского образа

Если для вашей версии ОС или СУБД существуют особые требования по установке исправлений, представленные в Azure Marketplace образы могут не соответствовать вашим потребностям. В этом случае следует создать виртуальную машину с помощью собственного пользовательского образа ОС и СУБД для виртуальной машины, который можно развертывать многократно. При подготовке пользовательского образа для дублирования следует учитывать следующие аспекты.

---
> ![Эмблема Windows.][Logo_Windows] Windows
>
> Дополнительные сведения см. здесь: <https://docs.microsoft.com/azure/virtual-machines/windows/upload-generalized-managed> Следует абстрагировать (обобщить) настройки Windows (например, идентификатор безопасности и имя узла Windows) на локальной виртуальной машине с помощью команды sysprep.
>
>
> ![Логотип Linux.][Logo_Linux] Linux
>
> Выполните действия, описанные в следующих статьях для [SUSE][virtual-machines-linux-create-upload-vhd-suse], [Red Hat][virtual-machines-linux-redhat-create-upload-vhd] или [Oracle Linux][virtual-machines-linux-create-upload-vhd-oracle], чтобы подготовить виртуальный жесткий диск для передачи в Azure.
>
>

---
Если вы уже установили содержимое SAP на локальную виртуальную машину (что особенно важно для двухуровневых систем), то параметры системы SAP можно изменить после развертывания виртуальной машины в Azure. Для этого переименуйте экземпляр в SAP Software Provisioning Manager (см. примечание к SAP [1619720]). Действия по подготовке и передаче в Azure подготовленной к работе виртуальной машины описаны в разделах этого документа [Подготовка виртуальной машины к развертыванию с помощью пользовательского образа для SAP][planning-guide-5.2.2] и [Передача VHD из локальной среды в Azure][planning-guide-5.3.2]. Прочтите раздел [Сценарий 2. Развертывание виртуальной машины для SAP с помощью пользовательского образа][deployment-guide-3.3] в [руководстве по развертыванию][deployment-guide], где приводятся инструкции по развертыванию этого образа в Azure.

#### <a name="deploying-a-vm-out-of-the-azure-marketplace"></a>Развертывание виртуальной машины из Azure Marketplace

Вы хотите использовать для развертывания виртуальной машины образ виртуальной машины из Azure Marketplace, предоставленный корпорацией Майкрософт или сторонним поставщиком. Когда вы развернете виртуальную машину в Azure, следуйте тем же рекомендациям и инструкциям средств для установки программного обеспечения SAP и/или СУБД, что и в локальной среде. Подробное описание развертывания см. в разделе [Сценарий 1. Развертывание виртуальной машины для SAP из Azure Marketplace][deployment-guide-3.2] в [руководстве по развертыванию][deployment-guide].

### <a name="preparing-vms-with-sap-for-azure"></a><a name="6ffb9f41-a292-40bf-9e70-8204448559e7"></a>Подготовка виртуальных машин с SAP для Azure

Перед отправкой виртуальных машин в Azure убедитесь, что виртуальные машины и виртуальные жесткие диски соответствуют определенным требованиям. Эти требования немного различаются в зависимости от используемого метода развертывания.

#### <a name="preparation-for-moving-a-vm-from-on-premises-to-azure-with-a-non-generalized-disk"></a><a name="1b287330-944b-495d-9ea7-94b83aff73ef"></a>Подготовка виртуальной машины к перемещению из локальной среды в Azure с помощью специализированного диска

Самым распространенным способом развертывания является перемещение имеющейся виртуальной машины, на которой работает система SAP, из локальной среды в Azure. Такая виртуальная машина и система SAP на ней будут работать в Azure с прежним именем узла и, возможно, с тем же идентификатором безопасности SAP. В этом случае не требуется обобщать гостевую операционную систему виртуальной машины для многочисленных развертываний. Если локальная сеть расширяется до Azure, в виртуальной машине могут использоваться даже учетные записи домена, которые использовались ранее в локальной среде.

Ниже перечислены требования для подготовки собственного диска виртуальной машины Azure.

* Изначально виртуальный жесткий диск, содержащий операционную систему, может иметь максимальный размер только 127 Гб. Это ограничение было снято в конце марта 2015 г. Теперь виртуальный жесткий диск, содержащий операционную систему, может иметь размер не более 1 ТБ, как и любой другой виртуальный жесткий диск, размещенный в службе хранилища Azure.
* Диск должен иметь формат фиксированного виртуального жесткого диска. Динамические виртуальные жесткие диски и виртуальные жесткие диски в формате VHDx пока не поддерживаются в Azure. Динамические виртуальные жесткие диски, загружаемые с помощью командлетов PowerShell или интерфейса командной строки, будут преобразованы в статические виртуальные жесткие диски.
* Для виртуальных жестких дисков, которые были подключены к виртуальной машине и должны быть снова подключены к виртуальной машине в Azure, также должен быть фиксированный формат. В [этой статье](../../managed-disks-overview.md) содержатся ограничения по размеру дисков данных. Динамические виртуальные жесткие диски, загружаемые с помощью командлетов PowerShell или интерфейса командной строки, будут преобразованы в статические виртуальные жесткие диски.
* Добавьте еще одну локальную учетную запись с правами администратора, которую сможет использовать служба поддержки корпорации Майкрософт. Кроме того, эту учетную запись можно назначить в качестве контекста выполнения для служб и приложений, пока не будет завершено развертывание виртуальной машины и не будут созданы подходящие учетные записи пользователей.
* Добавьте также и другие локальные учетные записи, если это нужно для вашего сценария развертывания.

---
> ![Эмблема Windows.][Logo_Windows] Windows
>
> В этом сценарии для отправки и развертывания виртуальной машины в Azure не требуется обобщать ее (с использованием sysprep).
> Убедитесь, что диск D:\ не используется.
> Настройте автоподключение для подключенных дисков, как описано в разделе этого документа [Настройка автоподключения для подключенных дисков][planning-guide-5.5.3].
>
> ![Логотип Linux.][Logo_Linux] Linux
>
> В этом сценарии для отправки и развертывания виртуальной машины в Azure не требуется обобщать ее (с использованием waagent -deprovision).
> Убедитесь, что /mnt/resource не используется, и что все диски подключены через uuid. Для диска ОС также убедитесь, что запись загрузчика содержит подключения на основе uuid.
>
>

---
#### <a name="preparation-for-deploying-a-vm-with-a-customer-specific-image-for-sap"></a><a name="57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3"></a>Подготовка виртуальной машины к развертыванию с помощью пользовательского образа для SAP

Файлы на виртуальном жестком диске, содержащие обобщенную ОС, также хранятся в контейнерах учетных записей хранения Azure или как образы управляемых дисков. Вы можете развернуть из такого образа новую виртуальную машину. Для этого укажите виртуальный жесткий диск или образ управляемого диска в качестве источника в файлах шаблона развертывания. Этот процесс описан в разделе [Сценарий 2. Развертывание виртуальной машины для SAP с помощью пользовательского образа][deployment-guide-3.3] в [руководстве по развертыванию][deployment-guide].

Ниже перечислены требования для подготовки собственного образа виртуальной машины Azure.

* Изначально виртуальный жесткий диск, содержащий операционную систему, может иметь максимальный размер только 127 Гб. Это ограничение было снято в конце марта 2015 г. Теперь виртуальный жесткий диск, содержащий операционную систему, может иметь размер не более 1 ТБ, как и любой другой виртуальный жесткий диск, размещенный в службе хранилища Azure.
* Диск должен иметь формат фиксированного виртуального жесткого диска. Динамические виртуальные жесткие диски и виртуальные жесткие диски в формате VHDx пока не поддерживаются в Azure. Динамические виртуальные жесткие диски, загружаемые с помощью командлетов PowerShell или интерфейса командной строки, будут преобразованы в статические виртуальные жесткие диски.
* Для виртуальных жестких дисков, которые были подключены к виртуальной машине и должны быть снова подключены к виртуальной машине в Azure, также должен быть фиксированный формат. В [этой статье](../../managed-disks-overview.md) содержатся ограничения по размеру дисков данных. Динамические виртуальные жесткие диски, загружаемые с помощью командлетов PowerShell или интерфейса командной строки, будут преобразованы в статические виртуальные жесткие диски.
* Добавьте также и другие локальные учетные записи, если это нужно для вашего сценария развертывания.
* Если образ содержит установку SAP NetWeaver, а имя узла при развертывании в Azure будет, вероятнее всего, отличаться от исходного имени, мы рекомендуем скопировать в шаблон последние версии диспетчера подготовки программного обеспечения SAP с DVD-диска. Это позволит вам использовать функцию переименования, предоставленную SAP, чтобы применить новое имя узла и/или изменить идентификатор SID системы SAP в развертываемом образе виртуальной машины при запуске новой копии.

---
> ![Эмблема Windows.][Logo_Windows] Windows
>
> Убедитесь, что диск D:\ не используется. Настройте автоподключение для подключенных дисков, как описано в разделе этого документа [Настройка автоподключения для подключенных дисков][planning-guide-5.5.3].
>
> ![Логотип Linux.][Logo_Linux] Linux
>
> Убедитесь, что /mnt/resource не используется, и что все диски подключены через uuid. Для диска ОС также убедитесь, что запись загрузчика содержит подключения на основе uuid.
>
>

---
* В такой шаблон можно также заранее установить графический интерфейс SAP (для администрирования и настройки).
* Вы можете установить и другое программное обеспечение, которое нужно для успешной работы виртуальной машины в распределенных сценариях. Но здесь важно, чтобы программное обеспечение сохранило работоспособность после переименования виртуальной машины.

Когда виртуальная машина будет достаточно подготовлена, чтобы быть универсальной и полностью независимой от учетных записей и пользователей, которые станут недоступными в целевом сценарии развертывания в Azure, выполните последнее действие для обобщения этого образа.

##### <a name="generalizing-a-vm"></a>Обобщение виртуальной машины
---
> ![Эмблема Windows.][Logo_Windows] Windows
>
> На последнем этапе войдите на виртуальную машину с помощью учетной записи администратора. Откройте окно командной строки Windows с правами *администратора*. Перейдите к папке %windir%\windows\system32\sysprep и запустите программу sysprep.exe.
> Откроется небольшое окно. Здесь следует установить флажок **Generalize** (Подготовка к использованию), который по умолчанию снят, и изменить значение параметра завершения работы с "Перезагрузка" (используется по умолчанию) на "Завершение работы". В этой процедуре предполагается, что процесс sysprep выполняется локально в гостевой ОС виртуальной машины.
> Если вы хотите выполнить процедуру на виртуальной машине, которая уже работает в Azure, то следуйте указаниям в [этой статье](../../windows/capture-image-resource.md).
>
> ![Логотип Linux.][Logo_Linux] Linux
>
> [Как записать образ виртуальной машины Linux для использования в качестве шаблона Resource Manager][capture-image-linux-step-2-create-vm-image]
>
>

---
### <a name="transferring-vms-and-vhds-between-on-premises-to-azure"></a>Передача виртуальных машин и виртуальных жестких дисков из локальной среды в Azure
Так как портал Azure не поддерживает передачу в Azure образов виртуальных машин и дисков, следует использовать командлеты Azure PowerShell или интерфейс командной строки. Также можно применить инструмент AzCopy. Эта программа может копировать виртуальные жесткие диски из локальной среды в Azure (и/или обратно). Также она позволяет копировать виртуальные жесткие диски между регионами Azure. Сведения о скачивании и использовании AzCopy см. в [этой документации][storage-use-azcopy] .

Есть и третий вариант — вы можете использовать различные инструменты сторонних производителей с графическим интерфейсом. Но перед этим убедитесь, что эти средства поддерживают страничные BLOB-объекты Azure. Для наших целей необходимо использовать хранилище BLOB-объектов Azure (различия описаны здесь: <https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs>). Средства, которые предоставляет Azure, эффективно выполняют сжатие отправляемых виртуальных машин и виртуальных жестких дисков. Это важно, поскольку эффективное сжатие сокращает время отправки (которое зависит также от скорости передачи интернет-подключения локальной инфраструктуры и целевого региона развертывания Azure). Можно уверенно предположить, что отправка виртуальной машины или виртуального жесткого диска из любой точки Европы в центр обработки данных Azure, расположенный в США, займет больше времени, чем отправка той же виртуальной машины или виртуального жесткого диска в европейский центр обработки данных Azure.

#### <a name="uploading-a-vhd-from-on-premises-to-azure"></a><a name="a43e40e6-1acc-4633-9816-8f095d5a7b6a"></a>Передача VHD из локальной среды в Azure
Чтобы передать существующую виртуальную машину или виртуальный жесткий диск из локальной сети, эта виртуальная машина или виртуальный жесткий диск должен отвечать требованиям, перечисленным в разделе этого документа [Подготовка виртуальной машины к перемещению из локальной среды в Azure с помощью специализированного диска][planning-guide-5.2.1].

Такую виртуальную машину НЕ НУЖНО обобщать. Она может быть загружена в том состоянии, в котором она хранится после завершения работы на локальном оборудовании. Это правило относится и к дополнительным виртуальным жестким дискам, которые не содержат операционную систему.

##### <a name="uploading-a-vhd-and-making-it-an-azure-disk"></a>Отправка виртуального жесткого диска и его преобразование в диск Azure
В этой ситуации мы хотим загрузить виртуальный жесткий диск с ОС или без нее, а затем подключить его к виртуальной машине как диск данных или использовать в качестве диска операционной системы. Этот процесс состоит из нескольких этапов.

**PowerShell**

* Войдите в свою подписку с помощью командлета *Connect-AzAccount*.
* Укажите подписку контекста с помощью командлета *Set-AzContext* с параметром SubscriptionId или SubscriptionName. Дополнительные сведения см. по адресу <https://docs.microsoft.com/powershell/module/az.accounts/set-Azcontext>.
* Загрузите виртуальный жесткий диск с помощью командлета *Add-AzVhd* в учетную запись хранения. Дополнительные сведения см. по адресу <https://docs.microsoft.com/powershell/module/az.compute/add-Azvhd>.
* (Необязательно.) Создайте управляемый диск из виртуального жесткого диска с помощью командлета *New-AzDisk*. Дополнительные сведения см. по адресу <https://docs.microsoft.com/powershell/module/az.compute/new-Azdisk>.
* Задайте в качестве диска ОС в конфигурации новой виртуальной машины виртуальный жесткий диск или управляемый диск с помощью командлета *Set-AzVMOSDisk*. Дополнительные сведения см. по адресу <https://docs.microsoft.com/powershell/module/az.compute/set-Azvmosdisk>.
* Создайте виртуальную машину из конфигурации виртуальной машины с помощью командлета *New-AzVM*. Дополнительные сведения см. по адресу <https://docs.microsoft.com/powershell/module/az.compute/new-Azvm>.
* Добавьте диск данных в новую виртуальную машину с помощью командлета *Add-AzVMDataDisk*. Дополнительные сведения см. по адресу <https://docs.microsoft.com/powershell/module/az.compute/add-Azvmdatadisk>.

**Azure CLI**

* Войдите в подписку с помощью команды *az login*.
* Выберите подписку с помощью команды *az account set --subscription `<subscription name or id`>* .
* Передайте виртуальный жесткий диск с помощью команды *az storage blob upload*. Дополнительные сведения см. в статье [Использование Azure CLI со службой хранилища Azure][storage-azure-cli].
* (Необязательно.) Создайте управляемый диск из виртуального жесткого диска с помощью команды *az disk create*. Дополнительные сведения см. по адресу https://docs.microsoft.com/cli/azure/disk.
* Создайте новую виртуальную машину, указав загруженный виртуальный жесткий диск или управляемый диск в качестве диска операционной системы с помощью команды *az vm create* c параметром *--attach-os-disk*.
* Добавьте диск данных в новую виртуальную машину с помощью команды *az vm disk attach* с параметром *--new*.

**Шаблон**

* Отправьте виртуальный жесткий диск с помощью PowerShell или Azure CLI.
* (Необязательно.) Создайте управляемый диск на основе виртуального жесткого диска с помощью PowerShell, Azure CLI или портала Azure.
* Разверните виртуальную машину с помощью шаблона JSON, в котором указана ссылка на виртуальный жесткий диск, как показано в [примере шаблона JSON](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-specialized-vhd-new-or-existing-vnet/azuredeploy.json), или управляемый диск, как показано в [этом примере шаблона JSON](https://github.com/Azure/azure-quickstart-templates/blob/master/sap-2-tier-user-image-md/azuredeploy.json).

#### <a name="deployment-of-a-vm-image"></a>Развертывание образа виртуальной машины
Чтобы передать имеющуюся виртуальную машину или виртуальный жесткий диск из локальной сети и использовать в качестве образа виртуальной машины Azure, эта виртуальная машина или виртуальный жесткий диск должен отвечать требованиям, перечисленным в разделе [Подготовка виртуальной машины к развертыванию с помощью пользовательского образа для SAP][planning-guide-5.2.2] этого документа.

* Используйте *sysprep* в Windows или *waagent-deprovision* в Linux, чтобы подготовить к работе виртуальную машину. Процесс для Windows подробно описан в [техническом руководстве по Sysprep](/previous-versions/windows/it-pro/windows-vista/cc766049(v=ws.10)), а для Linux — в разделе [Запись образа виртуальной машины Linux для его использования в качестве шаблона диспетчера ресурсов][capture-image-linux-step-2-create-vm-image].
* Войдите в свою подписку с помощью командлета *Connect-AzAccount*.
* Укажите подписку контекста с помощью командлета *Set-AzContext* с параметром SubscriptionId или SubscriptionName. Дополнительные сведения см. по адресу <https://docs.microsoft.com/powershell/module/az.accounts/set-Azcontext>.
* Загрузите виртуальный жесткий диск с помощью командлета *Add-AzVhd* в учетную запись хранения. Дополнительные сведения см. по адресу <https://docs.microsoft.com/powershell/module/az.compute/add-Azvhd>.
* (Необязательно.) Создайте образ управляемого диска из виртуального жесткого диска с помощью командлета *New-AzImage*. Дополнительные сведения см. по адресу <https://docs.microsoft.com/powershell/module/az.compute/new-Azimage>.
* Задайте в качестве диска ОС в конфигурации новой виртуальной машины:
  * Виртуальный жесткий диск с помощью командлета *Set-AzureRmVMOSDisk -SourceImageUri -CreateOption fromImage*. Дополнительные сведения см. по адресу <https://docs.microsoft.com/powershell/module/az.compute/set-Azvmosdisk>.
  * Образ управляемого диска с помощью командлета *Set-AzVMSourceImage*. Дополнительные сведения см. по адресу <https://docs.microsoft.com/powershell/module/az.compute/set-Azvmsourceimage>.
* Создайте виртуальную машину из конфигурации виртуальной машины с помощью командлета *New-AzVM*. Дополнительные сведения см. по адресу <https://docs.microsoft.com/powershell/module/az.compute/new-Azvm>.

**Azure CLI**

* Используйте *sysprep* в Windows или *waagent-deprovision* в Linux, чтобы подготовить к работе виртуальную машину. Процесс для Windows подробно описан в [техническом руководстве по Sysprep](/previous-versions/windows/it-pro/windows-vista/cc766049(v=ws.10)), а для Linux — в разделе [Запись образа виртуальной машины Linux для его использования в качестве шаблона диспетчера ресурсов][capture-image-linux-step-2-create-vm-image].
* Войдите в подписку с помощью команды *az login*.
* Выберите подписку с помощью команды *az account set --subscription `<subscription name or id`>* .
* Передайте виртуальный жесткий диск с помощью команды *az storage blob upload*. Дополнительные сведения см. в статье [Использование Azure CLI со службой хранилища Azure][storage-azure-cli].
* (Необязательно.) Создайте образ управляемого диска из виртуального жесткого диска с помощью команды *az image create*. Дополнительные сведения см. по адресу https://docs.microsoft.com/cli/azure/image.
* Создайте новую виртуальную машину, указав загруженный виртуальный жесткий диск или образ управляемого диска в качестве диска операционной системы с помощью команды *az vm create* c параметром *--image*.

**Шаблон**

* Используйте *sysprep* в Windows или *waagent-deprovision* в Linux, чтобы подготовить к работе виртуальную машину. Процесс для Windows подробно описан в [техническом руководстве по Sysprep](/previous-versions/windows/it-pro/windows-vista/cc766049(v=ws.10)), а для Linux — в разделе [Запись образа виртуальной машины Linux для его использования в качестве шаблона диспетчера ресурсов][capture-image-linux-step-2-create-vm-image].
* Отправьте виртуальный жесткий диск с помощью PowerShell или Azure CLI.
* (Необязательно.) Создайте образ управляемого диска на основе виртуального жесткого диска с помощью PowerShell, Azure CLI или портала Azure.
* Разверните виртуальную машину с помощью шаблона JSON, в котором указана ссылка на виртуальный жесткий диск образа, как показано в [примере шаблона JSON](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-specialized-vhd-new-or-existing-vnet/azuredeploy.json), или образ управляемого диска, как показано в [этом примере шаблона JSON](https://github.com/Azure/azure-quickstart-templates/blob/master/sap-2-tier-user-image-md/azuredeploy.json).

#### <a name="downloading-vhds-or-managed-disks-to-on-premises"></a>Загрузка виртуальных жестких дисков или управляемых дисков в локальную среду
Инфраструктура как услуга (IaaS) Azure позволяет не только отправлять в нее виртуальные жесткие диски и системы SAP. Вы можете переместить систему SAP из Azure обратно в локальную среду.

Во время загрузки виртуального жесткого диска или управляемого диска его нельзя использовать. А при загрузке диска, подключенного к виртуальной машине, следует отключить и освободить эту виртуальную машину. Если вам нужно только загрузить содержимое базы данных, которое будет использовано для настройки новой системы в локальной среде, и если при этом допустимо продолжать использование системы в Azure во время загрузки и установки новой системы, то вы можете избежать длительного периода простоя. Для этого можно создать сжатую резервную копию базы данных на виртуальном жестком диске и загрузить только этот диск, а не всю виртуальную машину с операционной системой.

#### <a name="powershell"></a>PowerShell

* При скачивании управляемого диска сначала необходимо получить доступ к базовому BLOB-объекту управляемого диска. Затем можно скопировать его в новую учетную запись хранения и загрузить его оттуда.

  ```powershell
  $access = Grant-AzDiskAccess -ResourceGroupName <resource group> -DiskName <disk name> -Access Read -DurationInSecond 3600
  $key = (Get-AzStorageAccountKey -ResourceGroupName <resource group> -Name <storage account name>)[0].Value
  $destContext = (New-AzStorageContext -StorageAccountName <storage account name -StorageAccountKey $key)
  Start-AzStorageBlobCopy -AbsoluteUri $access.AccessSAS -DestContainer <container name> -DestBlob <blob name> -DestContext $destContext
  # Wait for blob copy to finish
  Get-AzStorageBlobCopyState -Container <container name> -Blob <blob name> -Context $destContext
  Save-AzVhd -SourceUri <blob in new storage account> -LocalFilePath <local file path> -StorageKey $key
  # Wait for download to finish
  Revoke-AzDiskAccess -ResourceGroupName <resource group> -DiskName <disk name>
  ```

* Загрузка виртуального жесткого диска после остановки системы SAP и завершения работы виртуальной машины можно использовать командлет PowerShell `Save-AzVhd` на локальном целевом компьютере, чтобы загрузить диски VHD обратно в локальный мир. Для этого следует определить URL-адрес виртуального жесткого диска, который можно найти в разделе хранилища на портале Azure (перейдите к учетной записи хранения и найдите тот контейнер хранения, в котором был создан нужный виртуальный жесткий диск). Также нужно определить, куда следует копировать виртуальный жесткий диск.

  После этого вы можете запустить команду, указав в параметре SourceUri полученный URL-адрес загружаемого виртуального жесткого диска, а в параметре LocalFilePath — новое физическое расположение виртуального жесткого диска (включая имя файла). Эта команда будет выглядеть примерно так:

  ```powershell
  Save-AzVhd -ResourceGroupName <resource group name of storage account> -SourceUri http://<storage account name>.blob.core.windows.net/<container name>/sapidedata.vhd -LocalFilePath E:\Azure_downloads\sapidesdata.vhd
  ```

  Дополнительные сведения о командлете Save-AzVhd см. здесь: <https://docs.microsoft.com/powershell/module/az.compute/save-Azvhd>.

#### <a name="azure-cli"></a>Azure CLI
* При скачивании управляемого диска сначала необходимо получить доступ к базовому BLOB-объекту управляемого диска. Затем можно скопировать его в новую учетную запись хранения и загрузить его оттуда.

  ```azurecli
  az disk grant-access --ids "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>" --duration-in-seconds 3600
  az storage blob download --sas-token "<sas token>" --account-name <account name> --container-name <container name> --name <blob name> --file <local file>
  az disk revoke-access --ids "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>"
  ```

* Загрузка виртуального жесткого диска после остановки системы SAP и завершения работы виртуальной машины можно использовать команду Azure CLI `_azure storage blob download_` на локальном целевом компьютере, чтобы загрузить диски VHD обратно в локальный мир. Для этого следует определить имя и контейнер виртуального жесткого диска, который можно найти в разделе хранилища на портале Azure (перейдите к учетной записи хранения и найдите тот контейнер хранения, в котором был создан нужный виртуальный жесткий диск). Также нужно определить, куда следует копировать виртуальный жесткий диск.

  После этого вы можете запустить команду, указав в качестве параметров BLOB-объект и контейнер загружаемого виртуального жесткого диска, а также новое физическое расположение для него (включая имя файла). Эта команда будет выглядеть примерно так:

  ```azurecli
  az storage blob download --name <name of the VHD to download> --container-name <container of the VHD to download> --account-name <storage account name of the VHD to download> --account-key <storage account key> --file <destination of the VHD to download>
  ```

### <a name="transferring-vms-and-disks-within-azure"></a>Передача виртуальных машин и дисков в пределах Azure

#### <a name="copying-sap-systems-within-azure"></a>Копирование систем SAP в пределах Azure

Система SAP или выделенный сервер СУБД, поддерживающий уровень приложения SAP, обычно состоит из нескольких дисков, содержащих двоичные файлы ОС или файлы данных и журналов базы данных SAP. Функциям копирования и сохранения дисков в Azure в локальную среду не требуются механизмы синхронизации, позволяющие создать моментальный снимок нескольких дисков согласованным способом. Таким образом, состояние скопированных или сохраненных дисков окажется разным, даже если подключить их к той же виртуальной машине. Это означает, что если данные и журналы хранятся на нескольких дисках, скопированная база данных будет не согласована.

**Вывод. Чтобы скопировать или сохранить диски, которые входят в конфигурацию системы SAP, следует остановить эту систему SAP и завершить работу развернутой виртуальной машины. Только после этого вы можете копировать или загружать набор дисков, чтобы создать копию системы SAP в Azure или в локальной среде.**

Диски данных могут храниться в виде VHD-файлов в учетной записи хранения Azure. Их можно напрямую подключить к виртуальной машине или использовать в качестве образа. В этом случае виртуальный жесткий диск перед подключением к виртуальной машине копируется в новое расположение. Полное имя VHD-файла в Azure должно быть уникальным в пределах Azure. Как уже упоминалось ранее, имя имеет три компонента и выглядит примерно так:

`http(s)://<storage account name>.blob.core.windows.net/<container name>/<vhd name>`

Для дисков данных также можно использовать управляемые диски. В этом случае создается управляемый диск, который затем присоединяется к виртуальной машине. Имя управляемого диска должно быть уникальным в пределах группы ресурсов.

##### <a name="powershell"></a>PowerShell

Для копирования виртуального жесткого диска вы можете использовать командлеты Azure PowerShell, как описано в [этой статье][storage-powershell-guide-full-copy-vhd]. Для создания управляемого диска используйте команду New-AzDiskConfig и New-AzDisk, как показано в следующем примере.

```powershell
$config = New-AzDiskConfig -CreateOption Copy -SourceUri "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>" -Location <location>
New-AzDisk -ResourceGroupName <resource group name> -DiskName <disk name> -Disk $config
```

##### <a name="azure-cli"></a>Azure CLI

Для копирования виртуального жесткого диска можно использовать Azure CLI. Для создания управляемого диска используйте команду *az disk create*, как показано в следующем примере.

```azurecli
az disk create --source "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>" --name <disk name> --resource-group <resource group name> --location <location>
```

##### <a name="azure-storage-tools"></a>Средства хранилища Azure

* <https://storageexplorer.com/>

Профессиональные выпуски обозревателей службы хранилища Azure можно найти по следующим ссылкам.

* <https://www.cerebrata.com/>
* <https://clumsyleaf.com/products/cloudxplorer>

Сам процесс копирования виртуального жесткого диска в пределах учетной записи хранения занимает всего несколько секунд (сравнимо со скоростью создания моментальных снимков с отложенным копированием и копированием при записи на оборудовании SAN). Когда копирование файла виртуального жесткого диска завершится, вы можете присоединить его к виртуальной машине или использовать в качестве образа для присоединения копий диска к нескольким виртуальным машинам.

##### <a name="powershell"></a>PowerShell

```powershell
# attach a vhd to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$vm = Add-AzVMDataDisk -VM $vm -Name newdatadisk -VhdUri <path to vhd> -Caching <caching option> -DiskSizeInGB $null -Lun <lun, for example 0> -CreateOption attach
$vm | Update-AzVM

# attach a managed disk to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$vm = Add-AzVMDataDisk -VM $vm -Name newdatadisk -ManagedDiskId <managed disk id> -Caching <caching option> -DiskSizeInGB $null -Lun <lun, for example 0> -CreateOption attach
$vm | Update-AzVM

# attach a copy of the vhd to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$vm = Add-AzVMDataDisk -VM $vm -Name <disk name> -VhdUri <new path of vhd> -SourceImageUri <path to image vhd> -Caching <caching option> -DiskSizeInGB $null -Lun <lun, for example 0> -CreateOption fromImage
$vm | Update-AzVM

# attach a copy of the managed disk to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$diskConfig = New-AzDiskConfig -Location $vm.Location -CreateOption Copy -SourceUri <source managed disk id>
$disk = New-AzDisk -DiskName <disk name> -Disk $diskConfig -ResourceGroupName <resource group name>
$vm = Add-AzVMDataDisk -VM $vm -Caching <caching option> -Lun <lun, for example 0> -CreateOption attach -ManagedDiskId $disk.Id
$vm | Update-AzVM
```

##### <a name="azure-cli"></a>Azure CLI

```azurecli

# attach a vhd to a vm
az vm unmanaged-disk attach --resource-group <resource group name> --vm-name <vm name> --vhd-uri <path to vhd>

# attach a managed disk to a vm
az vm disk attach --resource-group <resource group name> --vm-name <vm name> --disk <managed disk id>

# attach a copy of the vhd to a vm
# this scenario is currently not possible with Azure CLI. A workaround is to manually copy the vhd to the destination.

# attach a copy of a managed disk to a vm
az disk create --name <new disk name> --resource-group <resource group name> --location <location of target virtual machine> --source <source managed disk id>
az vm disk attach --disk <new disk name or managed disk id> --resource-group <resource group name> --vm-name <vm name> --caching <caching option> --lun <lun, for example 0>
```

#### <a name="copying-disks-between-azure-storage-accounts"></a><a name="9789b076-2011-4afa-b2fe-b07a8aba58a1"></a>Копирование дисков между учетными записями хранения Azure
Эту задачу нельзя выполнить на портале Azure. Используйте командлеты Azure PowerShell, интерфейс командной строки Azure или обозреватель хранилища стороннего поставщика. Командлеты PowerShell и команды интерфейса командной строки позволяют создавать большие двоичные объекты и управлять ими, а также поддерживают возможность асинхронного копирования больших двоичных объектов между учетными записями хранения и между регионами в рамках подписки Azure.

##### <a name="powershell"></a>PowerShell
Вы также можете копировать виртуальные жесткие диски между подписками. Дополнительные сведения см. в [этой статье][storage-powershell-guide-full-copy-vhd].

Далее представлен базовый логический поток командлета PowerShell.

* Создайте контекст учетной записи хранения для **исходной** учетной записи хранения с помощью командлета *New-AzStorageContext*. Дополнительные сведения см. здесь: <https://docs.microsoft.com/powershell/module/az.storage/new-AzStoragecontext>.
* Создайте контекст учетной записи хранения для **целевой** учетной записи хранения с помощью командлета *New-AzStorageContext*. Дополнительные сведения см. здесь: <https://docs.microsoft.com/powershell/module/az.storage/new-AzStoragecontext>.
* Запустите копирование:

```powershell
Start-AzStorageBlobCopy -SrcBlob <source blob name> -SrcContainer <source container name> -SrcContext <variable containing context of source storage account> -DestBlob <target blob name> -DestContainer <target container name> -DestContext <variable containing context of target storage account>
```

* Циклически проверяйте состояние процесса копирования:

```powershell
Get-AzStorageBlobCopyState -Blob <target blob name> -Container <target container name> -Context <variable containing context of target storage account>
```

* Присоедините новый виртуальный жесткий диск к виртуальной машине, как описано выше.

Примеры можно найти в [этой статье][storage-powershell-guide-full-copy-vhd].

##### <a name="azure-cli"></a>Azure CLI
* Запустите копирование:

```azurecli
az storage blob copy start --source-blob <source blob name> --source-container <source container name> --source-account-name <source storage account name> --source-account-key <source storage account key> --destination-container <target container name> --destination-blob <target blob name> --account-name <target storage account name> --account-key <target storage account name>
```

* Циклически проверяйте состояние процесса копирования:

```azurecli
az storage blob show --name <target blob name> --container <target container name> --account-name <target storage account name> --account-key <target storage account name>
```

* Присоедините новый виртуальный жесткий диск к виртуальной машине, как описано выше.

### <a name="disk-handling"></a>Управление дисками

#### <a name="vmdisk-structure-for-sap-deployments"></a><a name="4efec401-91e0-40c0-8e64-f2dceadff646"></a>Структура виртуальной машины и диска для развертываний SAP

Управление структурой виртуальной машины и подключенными к ней дисками не должно вызывать никаких затруднений. Для локальных установок клиенты разработали много способов структурирования серверной установки.

* Один базовый диск, который содержит ОС и все двоичные файлы СУБД и/или SAP. Начиная с 2015 марта, этот диск может иметь размер до 1 ТБ вместо предыдущих ограничений, которые были ограничены 127 Гб.
* Один или несколько дисков, которые содержат файл журнала СУБД для базы данных SAP и файл журнала временной области хранения СУБД (если СУБД это поддерживает). Если журнал базы данных предъявляет высокие требования к скорости операций ввода-вывода, следует чередовать несколько дисков для достижения требуемого числа операций ввода-вывода в секунду.
* Несколько дисков, содержащих один или два файла базы данных для базы данных SAP и временные файлы данных СУБД (если СУБД это поддерживает).

![Эталонная конфигурация виртуальной машины Azure IaaS для SAP][planning-guide-figure-1300]


---
> ![Эмблема Windows.][Logo_Windows] Windows
>
> У многих клиентов мы наблюдали конфигурации, в которых двоичные файлы SAP и СУБД хранятся не на диске C:\, на котором установлена операционная система. Для этого были разные причины, но при подробном изучении, оказалось, что почти все они связаны с двумя факторами: 10–15 лет назад диски были маленькими, а ОС требовалось дополнительное пространство для обновлений. Оба этих условия в наши дни встречаются редко. Теперь диск C:\ можно сопоставить с большим диском или виртуальной машиной. Мы рекомендуем придерживаться указанного ниже шаблона развертывания для систем SAP NetWeaver в Azure, чтобы сохранить простоту структуры развертывания.
>
> Файл подкачки операционной системы Windows следует разместить на диске D: (временный диск).
>
> ![Логотип Linux.][Logo_Linux] Linux
>
> Файл подкачки Linux поместите в каталог /mnt/mnt/resource, как описано в [этой статье][virtual-machines-linux-agent-user-guide]. Настройки файла подкачки можно изменить в файле конфигурации /etc/waagent.conf агента Linux. Добавьте или измените следующие параметры:
>
>

```console
ResourceDisk.EnableSwap=y
ResourceDisk.SwapSizeMB=30720
```

Чтобы применить изменения, перезапустите агент Linux такой командой:

```console
sudo service waagent restart
```

Дополнительные сведения о рекомендуемом размере файла подкачки см. в примечании SAP [1597355] .

---
Количество дисков, используемых для файлов данных СУБД, и тип хранилища Azure для размещения этих дисков следует определять на основании требований к скорости операций ввода-вывода и сетевой задержке. Точные квоты описаны в [этой (Linux)][virtual-machines-sizes-linux] и [этой (Windows)][virtual-machines-sizes-windows] статье.

Опыт изучения развертываний SAP за последние два года позволил нам сформулировать несколько рекомендаций.

* Трафик ввода-вывода к разным файлам данных может быть разным, поскольку клиентские системы могут иметь разные размеры файлов данных для баз(ы) данных SAP. Поэтому оптимальной конфигурацией является RAID-массив для нескольких дисков, на котором размещаются файлы данных. В некоторых случаях, особенно при использовании стандартного хранилища Azure, скорость операций ввода-вывода для журнала транзакций СУБД достигала предела квоты для одного диска. В таких случаях мы рекомендуем использовать хранилище класса "Премиум" или объединять несколько дисков, размещенных в стандартном хранилище в полосу программного чередования.

---
> ![Эмблема Windows.][Logo_Windows] Windows
>
> * [Рекомендации по оптимизации производительности SQL Server на виртуальных машинах Azure][virtual-machines-sql-server-performance-best-practices]
>
> ![Логотип Linux.][Logo_Linux] Linux
>
> * [Настройка программного RAID-массива в Linux][virtual-machines-linux-configure-raid]
> * [Настройка диспетчера логических томов на виртуальной машине Linux в Azure][virtual-machines-linux-configure-lvm]
>
>

---
* Хранилище класса Premium демонстрирует значительно более высокую производительность, особенно по критически важному параметру записи журналов транзакций. Для сценариев SAP, в которых ожидается производительность на уровне рабочей системы, мы настоятельно рекомендуем использовать те серии виртуальных машин, которые позволяют использовать хранилище Azure Premium.

Помните, что диск, который содержит ОС, и, как мы рекомендуем, двоичные файлы SAP и базы данных (базовая виртуальная машина) также не ограничиваются 127 Гб. Теперь размер может донимать до 1 ТБ. Этого должно быть достаточно для хранения всех необходимых файлов, в том числе (к примеру) журналов пакетных заданий SAP.

Дополнительные рекомендации и сведения, относящиеся, в частности, к виртуальным машинам для СУБД, вы найдете в [руководстве по развертыванию СУБД][dbms-guide].

#### <a name="disk-handling"></a>Управление дисками

В большинстве случаев вам потребуются дополнительные диски для развертывания на виртуальной машине базы данных SAP. Мы уже упоминали ранее о выборе количества дисков в разделе [Структура виртуальной машины и диска для развертываний SAP][planning-guide-5.5.1] этого документа. Портал Azure позволяет подключать и отключать диски после развертывания базовой виртуальной машины. Диски можно подключать и отключать как во время работы виртуальной машины, так и во время ее остановки. При подключении диска портал Azure предлагает подключить пустой или имеющийся диск, который сейчас не подключен к другой виртуальной машине.

**Примечание.** Диски в конкретный момент времени могут быть подключены только к одной виртуальной машине.

![Подключение и отключение дисков: служба хранилища Azure класса "Стандартный"][planning-guide-figure-1400]

При развертывании новой виртуальной машины можно решить, что делать: использовать управляемые диски или разместить диски в учетных записях хранения Azure. Если вы хотите использовать хранилище класса Premium, мы советуем использовать управляемые диски.

Затем вам нужно решить, следует ли создавать новый пустой диск или выбрать для подключения к виртуальной машине существующий диск, который был передан ранее.

**ВАЖНО!** **НЕ** используйте кэширование узла со Службой хранилища Azure (цен. категория "Стандартный"). Для параметра настройки кэша узла следует оставить значение по умолчанию NONE ("не использовать"). Для хранилища Azure класса "Премиум" следует включить кэширование чтения, если в операциях ввода-вывода преобладает чтение, что является типичной ситуацией для файлов данных базы данных. Для файла журнала транзакций базы данных мы рекомендуем не использовать кэширование.

---
> ![Эмблема Windows.][Logo_Windows] Windows
>
> [Подключение диска данных на портале Azure][virtual-machines-linux-attach-disk-portal]
>
> Если диски уже подключены, войдите на виртуальную машину и откройте диспетчер дисков Windows. Если автоподключение не включено, как рекомендовано в разделе [Настройка автоподключения для подключенных дисков][planning-guide-5.5.3], то следует включить новый подключенный том и инициализировать его.
>
> ![Логотип Linux.][Logo_Linux] Linux
>
> После подключения дисков следует войти в виртуальную машину и инициализировать диски, как описано в [этой статье][virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux].
>
>

---
Кроме того, если подключен новый пустой диск, следует его отформатировать. В отношении форматирования, особенно для файлов данных и журналов СУБД, действуют те же рекомендации, что и для развертываний СУБД без ОС.

Учетная запись хранения Azure не предоставляет неограниченные ресурсы в отношении объема операций ввода-вывода, операций ввода-вывода в секунду и данных. Такие ограничения наиболее критичны для виртуальных машин с СУБД. Если у вас есть несколько виртуальных машин с высоким трафиком ввода-вывода, мы рекомендуем использовать для каждой из них отдельную учетную запись хранения, чтобы оставаться в пределах ограничений, установленных для учетной записи хранения Azure. Также вы можете выбрать любой разумный метод распределения этих виртуальных машин между несколькими учетными записями хранения, который позволит не превышать установленные ограничения для каждой из учетных записей хранения. Дополнительные сведения см. в [руководстве по развертыванию СУБД][dbms-guide]. Эти ограничения должны учитываться и для виртуальных машин, на которых работают только серверы приложения SAP, или других виртуальных машин, для которых позднее могут потребоваться дополнительные виртуальные жесткие диски. Эти ограничения не применяются при использовании управляемого диска. Если вы планируете использовать хранилище класса Premium, мы советуем использовать управляемый диск.

Еще один важный аспект для учетных записей хранения — применение георепликации для виртуальных жестких дисков, входящих в учетную запись хранения. Георепликация включается и отключается на уровне учетной записи хранения, а не на уровне виртуальной машины. Если георепликация включена, то все виртуальные жесткие диски этой учетной записи хранения будут реплицироваться в другой центр обработки данных Azure в том же регионе. Чтобы принять решение о включении георепликации, следует учитывать следующие ограничения.

Георепликация Azure работает отдельно для каждого виртуального жесткого диска на виртуальной машине. Она не сохраняет хронологический порядок операций ввода-вывода для нескольких виртуальных жестких дисков на одной виртуальной машине. Таким образом, виртуальный жесткий диск, представляющий базовую виртуальную машину, и все остальные подключенные к ней диски реплицируются независимо друг от друга. Синхронизация между изменениями на разных виртуальных жестких дисках не соблюдается. Операции ввода-вывода реплицируются без соблюдения порядка, в котором они записывались, поэтому георепликация не имеет смысла для серверов баз данных, у которых базы данных распределены по нескольким виртуальным жестким дискам. Кроме СУБД, у вас могут быть другие приложения, которые записывают или изменяют данные на нескольких виртуальных жестких дисках и для которых важен порядок этих изменений. Если у вас есть такие приложения, не следует включать георепликацию в Azure. Если в вашей среде нужна георепликация для одного набора виртуальных машин, но не нужна для другого набора, вы можете распределить виртуальные машины и подключенные к ним виртуальные жесткие диски между несколькими учетными записями хранения, для которых георепликация будет включена или отключена.

#### <a name="setting-automount-for-attached-disks"></a><a name="17e0d543-7e8c-4160-a7da-dd7117a1ad9d"></a>Настройка автоподключения для подключенных дисков
---
> ![Эмблема Windows.][Logo_Windows] Windows
>
> Проверьте, установлен ли параметр автоподключения для виртуальных машин, созданных из собственных образов или дисков. Если параметр не установлен, установите его. Если в виртуальной машине установлен этот параметр, она сможет автоматически повторно подключить все используемые диски после перезапуска или повторного развертывания в Azure.
> Для всех образов, предоставленных корпорацией Майкрософт в Azure Marketplace, этот параметр уже установлен.
>
> Чтобы установить параметр автоподключения, ознакомьтесь с документацией служебной программы diskpart.exe, выполняемой из командной строки.
>
> * [Параметры командной строки служебной программы DiskPart](/previous-versions/windows/it-pro/windows-xp/bb490893(v=technet.10))
> * [Автоподключение](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc753703(v=ws.11))
>
> Откройте окно командной строки Windows от имени администратора.
>
> Если диски уже подключены, войдите на виртуальную машину и откройте диспетчер дисков Windows. Если автоподключение не включено, как рекомендовано в разделе [Настройка автоподключения для подключенных дисков][planning-guide-5.5.3], то следует включить новый подключенный том и инициализировать его.
>
> ![Логотип Linux.][Logo_Linux] Linux
>
> Каждый новый пустой диск, который вы подключаете, следует инициализировать, как описано в [этой статье][virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux].
> Также все новые диски нужно добавить в каталог /etc/fstab.
>
>

---
### <a name="final-deployment"></a>Окончательное развертывание

Инструкции по окончательному развертыванию, в том числе инструкции по развертыванию расширения Azure для SAP, вы найдете в [руководстве по развертыванию][deployment-guide].

## <a name="accessing-sap-systems-running-within-azure-vms"></a>Доступ к системам SAP, работающим на виртуальных машинах Azure

Для сценариев, требующих подключения к этим системам SAP через общедоступный Интернет с помощью графического интерфейса SAP, необходимо применить следующие процедуры.

Далее в этом документе мы рассмотрим другие основные сценарии, например подключение к системам SAP в распределенном развертывании с подключением типа"сеть — сеть" (VPN-туннель) или подключение Azure ExpressRoute между локальными системами и системами Azure.

### <a name="remote-access-to-sap-systems"></a>Удаленный доступ к службам SAP

В Azure Resource Manager теперь нет конечных точек по умолчанию, которые использовались прежде в классической модели. При соблюдении следующих условий открыты все порты виртуальной машины Azure Resource Manager.

1. Для подсети или сетевого интерфейса не определена группа безопасности сети. Сетевой трафик к виртуальным машинам Azure можно защитить с помощью так называемых "групп безопасности сети". Дополнительные сведения см. в статье [Группа безопасности сети][virtual-networks-nsg].
2. Для сетевого интерфейса не определен Azure Load Balancer.

Различия в архитектуре между классической моделью и ARM описаны в [этой статье][virtual-machines-azure-resource-manager-architecture].

#### <a name="configuration-of-the-sap-system-and-sap-gui-connectivity-over-the-internet"></a>Настройка подключения к системе SAP и графическому пользовательскому интерфейсу SAP через Интернет

Дополнительные сведения см. в этой статье:

<https://docs.microsoft.com/archive/blogs/saponsqlserver/sap-gui-connection-closed-when-connecting-to-sap-system-in-azure>

#### <a name="changing-firewall-settings-within-vm"></a>Изменение параметров брандмауэра на виртуальной машине

Может потребоваться настроить брандмауэр на виртуальных машинах, чтобы разрешить входящий трафик к системе SAP.

---
> ![Эмблема Windows.][Logo_Windows] Windows
>
> По умолчанию брандмауэр Windows на виртуальной машине, развернутой в Azure, всегда включен. Следует разрешить открытие порта SAP, иначе графический пользовательский интерфейс SAP не сможет подключиться.
> Для этого:
>
> * Последовательно откройте "Панель управления", "Система и безопасность", "Брандмауэр Windows" и перейдите в раздел **Дополнительные параметры**.
> * Щелкните правой кнопкой мыши "Правила для входящих подключений" и выберите **Новое правило**.
> * Откроется мастер, в котором следует выбрать создание нового правила **Порт**.
> * На следующем шаге мастера оставьте значение TCP и введите номер порта, который хотите открыть. Наш экземпляр SAP имеет идентификатор 00, поэтому мы выбрали номер порта 3200. Если у вас экземпляр имеет другой номер, следует открыть тот порт, который вы ранее определили для этого номера экземпляра.
> * На следующем шаге мастера оставьте выбранным элемент **Разрешить подключение**.
> * Затем укажите, применяется ли это правило для домена, общедоступной и частной сетей. При необходимости измените его. Но если вы хотите разрешить подключение к графическому пользовательскому интерфейсу SAP из общедоступной сети, это правило необходимо применить к общедоступной сети.
> * На последнем шаге мастера присвойте правилу имя и сохраните изменения, нажав кнопку **Готово**.
>
> Правило вступает в силу немедленно.
>
> ![Определение правила для порта][planning-guide-figure-1600]
>
> ![Логотип Linux.][Logo_Linux] Linux
>
> Образы Linux, представленные в Azure Marketplace, не включают брандмауэр iptables по умолчанию, поэтому подключение к системе SAP будет работать. Если вы самостоятельно включили iptables или другой брандмауэр, обратитесь к документации по iptables или соответствующему брандмауэру и разрешите входящий трафик на TCP-порт 32xx (где xx — это номер вашей системы SAP).
>
>

---
#### <a name="security-recommendations"></a>Рекомендации по обеспечению безопасности

Графический пользовательский интерфейс SAP не подключается сразу к запущенным экземплярам SAP (через порт 32xx), а сначала подключается к процессу сервера сообщений SAP через открытый для него порт (порт 36xx). Ранее этот порт использовался сервером сообщений для внутреннего взаимодействия с экземплярами приложений. Чтобы предотвратить непреднамеренный обмен данными локальных серверов приложений с сервером сообщений в Azure, вы можете изменить порты внутренней связи. Мы настоятельно рекомендуем изменить номер порта для внутренней связи между сервером сообщений SAP и экземплярами приложения на системах, которые были клонированы из локальных систем. Это относится, например, к клонам системы разработки, созданным для тестирования проекта. Это можно сделать с помощью параметра профиля по умолчанию:

> rdisp/msserv_internal
>
>

Это описано в разделе [Параметры безопасности для сервера сообщений SAP](https://help.sap.com/saphelp_nwpi71/helpdata/en/47/c56a6938fb2d65e10000000a42189c/content.htm).


### <a name="single-vm-with-sap-netweaver-demotraining-scenario"></a><a name="3e9c3690-da67-421a-bc3f-12c520d99a30"></a>Обучающий сценарий с демонстрацией одной виртуальной машины с SAP NetWeaver

![Использование демонстрационных систем SAP с одной ВМ: в облачных службах Azure одинаковые имена виртуальных машин изолируются][planning-guide-figure-1700]

В этом сценарии мы реализуем типичный демонстрационный или учебный сценарий с системой, полностью развернутой на одной виртуальной машине. Мы предполагаем, что развертывание выполняется с помощью шаблонов образа виртуальной машины. Также мы предполагаем, что нужно будет развернуть несколько таких демонстрационных/обучающих виртуальных машин с одинаковыми именами. Учебные системы не подключаются к локальным ресурсам и являются противоположностью для гибридного развертывания.

Допустим, вы уже создали образ виртуальной машины, как описано в нескольких частях раздела этого документа [Подготовка виртуальных машин с SAP для Azure][planning-guide-5.2].

Последовательность событий для реализации сценария выглядит следующим образом.

##### <a name="powershell"></a>PowerShell

* Создайте группу ресурсов для каждого обучающего или демонстрационного ландшафта.

```powershell
$rgName = "SAPERPDemo1"
New-AzResourceGroup -Name $rgName -Location "North Europe"
```

* Создайте учетную запись хранилища, если вы не хотите использовать управляемые диски.

```powershell
$suffix = Get-Random -Minimum 100000 -Maximum 999999
$account = New-AzStorageAccount -ResourceGroupName $rgName -Name "saperpdemo$suffix" -SkuName Standard_LRS -Kind "Storage" -Location "North Europe"
```

* Создайте новую виртуальную сеть для каждого обучающего или демонстрационного ландшафта, чтобы сделать возможным использование одинаковых имен узлов и IP-адресов. Защитите виртуальную сеть с помощью группы безопасности сети, которая пропускает только трафик на порт 3389 для доступа к удаленному рабочему столу и на порт 22 для SSH.

```powershell
# Create a new Virtual Network
$rdpRule = New-AzNetworkSecurityRuleConfig -Name SAPERPDemoNSGRDP -Protocol * -SourcePortRange * -DestinationPortRange 3389 -Access Allow -Direction Inbound -SourceAddressPrefix * -DestinationAddressPrefix * -Priority 100
$sshRule = New-AzNetworkSecurityRuleConfig -Name SAPERPDemoNSGSSH -Protocol * -SourcePortRange * -DestinationPortRange 22 -Access Allow -Direction Inbound -SourceAddressPrefix * -DestinationAddressPrefix * -Priority 101
$nsg = New-AzNetworkSecurityGroup -Name SAPERPDemoNSG -ResourceGroupName $rgName -Location  "North Europe" -SecurityRules $rdpRule,$sshRule

$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name Subnet1 -AddressPrefix  10.0.1.0/24 -NetworkSecurityGroup $nsg
$vnet = New-AzVirtualNetwork -Name SAPERPDemoVNet -ResourceGroupName $rgName -Location "North Europe"  -AddressPrefix 10.0.1.0/24 -Subnet $subnetConfig
```

* Создайте новый общедоступный IP-адрес, который можно использовать для доступа к виртуальной машине из Интернета.

```powershell
# Create a public IP address with a DNS name
$pip = New-AzPublicIpAddress -Name SAPERPDemoPIP -ResourceGroupName $rgName -Location "North Europe" -DomainNameLabel $rgName.ToLower() -AllocationMethod Dynamic
```

* Создайте новый сетевой интерфейс для виртуальной машины.

```powershell
# Create a new Network Interface
$nic = New-AzNetworkInterface -Name SAPERPDemoNIC -ResourceGroupName $rgName -Location "North Europe" -Subnet $vnet.Subnets[0] -PublicIpAddress $pip
```

* Создайте виртуальную машину. В этом сценарии у всех виртуальных машин будет одно и то же имя. Идентификаторы безопасности SAP у экземпляров SAP NetWeaver на этих виртуальных машинах также будут одинаковыми. Имя виртуальной машины должно быть уникальным в пределах группы ресурсов Azure, но в разных группах ресурсов Azure можно запускать виртуальные машины с одинаковыми именами. Не допускаются учетные записи по умолчанию: "Администратор" для Windows или root для Linux. Поэтому следует определить учетную запись администратора с другим именем пользователя и пароль для нее. Также следует указать размер виртуальной машины.

```powershell
#####
# Create a new virtual machine with an official image from the Azure Marketplace
#####
$cred=Get-Credential -Message "Type the name and password of the local administrator account."
$vmconfig = New-AzVMConfig -VMName SAPERPDemo -VMSize Standard_D11

# select image
$vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer" -Skus "2012-R2-Datacenter" -Version "latest"
$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Windows -ComputerName "SAPERPDemo" -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
# $vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "SUSE" -Offer "SLES-SAP" -Skus "12-SP1" -Version "latest"
# $vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "RedHat" -Offer "RHEL" -Skus "7.2" -Version "latest"
# $vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "Oracle" -Offer "Oracle-Linux" -Skus "7.2" -Version "latest"
# $vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Linux -ComputerName "SAPERPDemo" -Credential $cred

$vmconfig = Add-AzVMNetworkInterface -VM $vmconfig -Id $nic.Id

$vmconfig = Set-AzVMBootDiagnostics -Disable -VM $vmconfig
$vm = New-AzVM -ResourceGroupName $rgName -Location "North Europe" -VM $vmconfig
```

```powershell
#####
# Create a new virtual machine with a VHD that contains the private image that you want to use
#####
$cred=Get-Credential -Message "Type the name and password of the local administrator account."
$vmconfig = New-AzVMConfig -VMName SAPERPDemo -VMSize Standard_D11

$vmconfig = Add-AzVMNetworkInterface -VM $vmconfig -Id $nic.Id

$diskName="osfromimage"
$osDiskUri=$account.PrimaryEndpoints.Blob.ToString() + "vhds/" + $diskName  + ".vhd"

$vmconfig = Set-AzVMOSDisk -VM $vmconfig -Name $diskName -VhdUri $osDiskUri -CreateOption fromImage -SourceImageUri <path to VHD that contains the OS image> -Windows
$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Windows -ComputerName "SAPERPDemo" -Credential $cred
#$vmconfig = Set-AzVMOSDisk -VM $vmconfig -Name $diskName -VhdUri $osDiskUri -CreateOption fromImage -SourceImageUri <path to VHD that contains the OS image> -Linux
#$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Linux -ComputerName "SAPERPDemo" -Credential $cred

$vmconfig = Set-AzVMBootDiagnostics -Disable -VM $vmconfig
$vm = New-AzVM -ResourceGroupName $rgName -Location "North Europe" -VM $vmconfig
```

```powershell
#####
# Create a new virtual machine with a Managed Disk Image
#####
$cred=Get-Credential -Message "Type the name and password of the local administrator account."
$vmconfig = New-AzVMConfig -VMName SAPERPDemo -VMSize Standard_D11

$vmconfig = Add-AzVMNetworkInterface -VM $vmconfig -Id $nic.Id

$vmconfig = Set-AzVMSourceImage -VM $vmconfig -Id <Id of Managed Disk Image>
$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Windows -ComputerName "SAPERPDemo" -Credential $cred
#$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Linux -ComputerName "SAPERPDemo" -Credential $cred

$vmconfig = Set-AzVMBootDiagnostics -Disable -VM $vmconfig
$vm = New-AzVM -ResourceGroupName $rgName -Location "North Europe" -VM $vmconfig
```

* По желанию вы можете добавить дополнительные диски и восстановить необходимое содержимое. Все имена больших двоичных объектов (URL-адреса для доступа к BLOB-объектам) должны быть уникальными в пределах Azure.

```powershell
# Optional: Attach additional VHD data disks
$vm = Get-AzVM -ResourceGroupName $rgName -Name SAPERPDemo
$dataDiskUri = $account.PrimaryEndpoints.Blob.ToString() + "vhds/datadisk.vhd"
Add-AzVMDataDisk -VM $vm -Name datadisk -VhdUri $dataDiskUri -DiskSizeInGB 1023 -CreateOption empty | Update-AzVM

# Optional: Attach additional Managed Disks
$vm = Get-AzVM -ResourceGroupName $rgName -Name SAPERPDemo
Add-AzVMDataDisk -VM $vm -Name datadisk -DiskSizeInGB 1023 -CreateOption empty -Lun 0 | Update-AzVM
```

##### <a name="cli"></a>CLI

Следующий пример кода можно использовать на платформе Linux. Для Windows вы можете использовать PowerShell, как описано выше, или отредактировать этот пример, заменив $rgName значением %rgName% и присвоив значение переменной среды с помощью команды Windows *set*.

* Создайте группу ресурсов для каждого обучающего или демонстрационного ландшафта.

```azurecli
rgName=SAPERPDemo1
rgNameLower=saperpdemo1
az group create --name $rgName --location "North Europe"
```

* Создание новой учетной записи хранения

```azurecli
az storage account create --resource-group $rgName --location "North Europe" --kind Storage --sku Standard_LRS --name $rgNameLower
```

* Создайте новую виртуальную сеть для каждого обучающего или демонстрационного ландшафта, чтобы сделать возможным использование одинаковых имен узлов и IP-адресов. Защитите виртуальную сеть с помощью группы безопасности сети, которая пропускает только трафик на порт 3389 для доступа к удаленному рабочему столу и на порт 22 для SSH.

```azurecli
az network nsg create --resource-group $rgName --location "North Europe" --name SAPERPDemoNSG
az network nsg rule create --resource-group $rgName --nsg-name SAPERPDemoNSG --name SAPERPDemoNSGRDP --protocol \* --source-address-prefix \* --source-port-range \* --destination-address-prefix \* --destination-port-range 3389 --access Allow --priority 100 --direction Inbound
az network nsg rule create --resource-group $rgName --nsg-name SAPERPDemoNSG --name SAPERPDemoNSGSSH --protocol \* --source-address-prefix \* --source-port-range \* --destination-address-prefix \* --destination-port-range 22 --access Allow --priority 101 --direction Inbound

az network vnet create --resource-group $rgName --name SAPERPDemoVNet --location "North Europe" --address-prefixes 10.0.1.0/24
az network vnet subnet create --resource-group $rgName --vnet-name SAPERPDemoVNet --name Subnet1 --address-prefix 10.0.1.0/24 --network-security-group SAPERPDemoNSG
```

* Создайте новый общедоступный IP-адрес, который можно использовать для доступа к виртуальной машине из Интернета.

```azurecli
az network public-ip create --resource-group $rgName --name SAPERPDemoPIP --location "North Europe" --dns-name $rgNameLower --allocation-method Dynamic
```

* Создайте новый сетевой интерфейс для виртуальной машины.

```azurecli
az network nic create --resource-group $rgName --location "North Europe" --name SAPERPDemoNIC --public-ip-address SAPERPDemoPIP --subnet Subnet1 --vnet-name SAPERPDemoVNet
```

* Создайте виртуальную машину. В этом сценарии у всех виртуальных машин будет одно и то же имя. Идентификаторы безопасности SAP у экземпляров SAP NetWeaver на этих виртуальных машинах также будут одинаковыми. Имя виртуальной машины должно быть уникальным в пределах группы ресурсов Azure, но в разных группах ресурсов Azure можно запускать виртуальные машины с одинаковыми именами. Не допускаются учетные записи по умолчанию: "Администратор" для Windows или root для Linux. Поэтому следует определить учетную запись администратора с другим именем пользователя и пароль для нее. Также следует указать размер виртуальной машины.

```azurecli
#####
# Create virtual machines using storage accounts
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image SUSE:SLES-SAP:12-SP1:latest --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image RedHat:RHEL:7.2:latest --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image "Oracle:Oracle-Linux:7.2:latest" --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --authentication-type password

#####
# Create virtual machines using Managed Disks
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image SUSE:SLES-SAP:12-SP1:latest --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image RedHat:RHEL:7.2:latest --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image "Oracle:Oracle-Linux:7.2:latest" --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --authentication-type password
```

```azurecli
#####
# Create a new virtual machine with a VHD that contains the private image that you want to use
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --os-type Windows --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --image <path to image vhd>
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --os-type Linux --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --image <path to image vhd> --authentication-type password

#####
# Create a new virtual machine with a Managed Disk Image
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --image <managed disk image id>
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --image <managed disk image id> --authentication-type password
```

* По желанию вы можете добавить дополнительные диски и восстановить необходимое содержимое. Все имена больших двоичных объектов (URL-адреса для доступа к BLOB-объектам) должны быть уникальными в пределах Azure.

```azurecli
# Optional: Attach additional VHD data disks
az vm unmanaged-disk attach --resource-group $rgName --vm-name SAPERPDemo --size-gb 1023 --vhd-uri https://$rgNameLower.blob.core.windows.net/vhds/data.vhd  --new

# Optional: Attach additional Managed Disks
az vm disk attach --resource-group $rgName --vm-name SAPERPDemo --size-gb 1023 --disk datadisk --new
```

##### <a name="template"></a>Шаблон

Примеры шаблонов можно использовать в репозитории Azure-QuickStart-templates на сайте GitHub.

* [Простая виртуальная машина Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-linux)
* [Простая виртуальная машина Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows)
* [Виртуальная машина из образа](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image)

### <a name="implement-a-set-of-vms-that-communicate-within-azure"></a>Реализация набора виртуальных машин, которые взаимодействуют в Azure

Негибридное развертывание является типичным сценарием для задач обучения или демонстрации. В нем программное обеспечение для обучения или демонстрации распределяется по нескольким виртуальным машинам. Разные компоненты, установленные на разных виртуальных машинах, должны взаимодействовать друг с другом. В этом сценарии не требуется взаимодействие с локальной сетью или использование распределенного подключения.

Этот сценарий является расширением установки, описанной в разделе этого документа [Обучающий сценарий с демонстрацией одной виртуальной машины с SAP NetWeaver][planning-guide-7.1]. В этом случае в существующую группу ресурсов будут добавлены дополнительные виртуальные машины. В следующем примере обучающий ландшафт состоит из виртуальных машин SAP ASCS/SCS, виртуальной машины с СУБД и виртуальной машины с экземпляром сервера приложений SAP.

Прежде чем создавать этот сценарий, следует обдумать все базовые настройки, как описано в предыдущем сценарии.

#### <a name="resource-group-and-virtual-machine-naming"></a>Имена группы ресурсов и виртуальных машин

Все группы ресурсов должны иметь уникальные имена. Создайте собственную схему именования ресурсов, например `<rg-name`>-суффикс.

Имя виртуальной машины должно быть уникальным в пределах группы ресурсов.

#### <a name="set-up-network-for-communication-between-the-different-vms"></a>Настройка сети для взаимодействия между разными виртуальными машинами

![Набор виртуальных машин в виртуальной сети Azure][planning-guide-figure-1900]

Чтобы не возникало коллизий между именами клонов одного обучающего/демонстрационного ландшафта, создайте отдельную виртуальную сеть Azure для каждого ландшафта. Разрешение имен DNS будет выполнять Azure. Также вы можете настроить собственный DNS-сервер за пределами Azure (этот процесс здесь не описывается). В нашем сценарии мы не создаем собственную службу DNS. Для всех виртуальных машин в одной виртуальной сети Azure возможно взаимодействие с использованием имен узлов.

Разделение обучающих/демонстрационных ландшафтов с помощью виртуальных сетей, а не только групп ресурсов, может иметь следующие причины.

* Настроенный ландшафт SAP должен иметь собственную службу AD/OpenLDAP, а сервер домена должен входить в состав каждого из ландшафтов.
* Настроенный ландшафт SAP содержит компоненты, для работы которых требуются фиксированные IP-адреса.

Дополнительные сведения о виртуальных сетях Azure и способах их определения вы можете найти в [этой статье][virtual-networks-create-vnet-arm-pportal].

## <a name="deploying-sap-vms-with-corporate-network-connectivity-cross-premises"></a>Развертывание виртуальных машин SAP с подключением к корпоративной сети (распределенное)

Предположим, у вас есть ландшафт SAP, развертывание которого вы хотите разделить на несколько уровней: аппаратная среда для высокопроизводительных серверов СУБД, локальные виртуализированные среды для уровней приложений и более мелкие двухуровневые системы SAP на базе Azure IaaS. Важное базовое допущение состоит в том, что системы SAP в пределах одного ландшафта SAP должны взаимодействовать друг с другом и с множеством других программных компонентов, которые развернуты в компании в разных формах развертывания. Также различия в форме развертывания не должны влиять на взаимодействие конечного пользователя с графическим пользовательским интерфейсом SAP или другим интерфейсом, к которому он подключается. Эти условия можно выполнить только при наличии локального каталога Active Directory/OpenLDAP и службы DNS, которые расширены в систему Azure с помощью подключения "сеть — сеть", многосайтового или частного подключения (например, Azure ExpressRoute).



### <a name="scenario-of-an-sap-landscape"></a>Сценарий ландшафта SAP

Сценарий распределенного или гибридного развертывания в общих чертах можно представить в виде следующего рисунка.

![Подключение типа "сеть — сеть" между ресурсами в локальной среде и Azure][planning-guide-figure-2100]

Минимальным требованием является использование протоколов защищенной коммуникации. Это может быть SSL/TLS для доступа через браузер или VPN-подключения для доступа систем к службам Azure. Мы понимаем, что компании используют разные методы управления VPN-подключениями между корпоративной сетью и Azure. Некоторые компании полностью открывают все порты. Другие компании хотят точно определять, какие порты следует открыть.

Ниже в таблице перечислены стандартные коммуникационные порты для SAP. Обычно вполне достаточно открыть только порт шлюза SAP.

<!-- sapms is prefix of a SAP service name and not a spelling error -->

| Служба | Имя порта | Пример: `<nn`> = 01 | Диапазон по умолчанию (мин — макс) | Комментарий |
| --- | --- | --- | --- | --- |
| Диспетчер |sapdp`<nn>` см. * |3201 |3200 - 3299 |Диспетчер SAP, который используется графическим пользовательским интерфейсом SAP для Windows и Java |
| Сервер сообщений |sapms`<sid`> см. ** |3600 |свободный sapms`<anySID`> |sid = идентификатор SAP-System-ID |
| Шлюз |sapgw`<nn`> см. * |3301 |свободный |Шлюз SAP, используемый для взаимодействия CPIC и RFC |
| Маршрутизатор SAP |sapdp99 |3299 |свободный |Только имена службы CI (центрального экземпляра) можно заменить в файле /etc/services произвольными значениями после установки. |

*) nn = номер экземпляра SAP

\*\*) sid = идентификатор SAP-System-ID

Дополнительные сведения о том, какие порты нужны для различных продуктов и служб SAP, сгруппированные по продуктам SAP, см. здесь: <https://scn.sap.com/docs/DOC-17124>.
С помощью этого документа вы сможете открыть на VPN-устройстве конкретные порты, необходимые для определенных продуктов и сценариев SAP.

В качестве дополнительной меры безопасности при развертывании виртуальных машин в таком сценарии вы можете создать [группу безопасности сети][virtual-networks-nsg] и определить для нее правила доступа.



#### <a name="printing-on-a-local-network-printer-from-sap-instance-in-azure"></a>Печать на принтере локальной сети из экземпляра SAP в Azure

##### <a name="printing-over-tcpip-in-cross-premises-scenario"></a>Печать по протоколу TCP/IP в распределенном сценарии

Настройка локальных принтеров в сети на основе TCP/IP выполняется на виртуальной машине Azure в целом так же, как в корпоративной сети, если у вас настроен VPN-туннель "сеть — сеть" или подключение ExpressRoute.

---
> ![Эмблема Windows.][Logo_Windows] Windows
>
> Для этого:
>
> * Некоторые сетевые принтеры поставляются с мастером настройки, который облегчает настройку принтера на виртуальной машине Azure. Если мастер не предоставлен, можно использовать ручной способ настройки принтера, то есть создать для принтера новый порт TCP/IP.
> * Откройте "Панель управления", затем "Устройства и принтеры" -> "Установка принтера".
> * Выберите "Добавить принтер с использованием TCP/IP-адреса или имени узла".
> * Введите IP-адрес принтера.
> * Стандартный порт принтера — 9100.
> * При необходимости вручную установите соответствующий драйвер принтера.
>
> ![Логотип Linux.][Logo_Linux] Linux
>
> * Как и для Windows, просто выполните стандартную процедуру установки сетевого принтера.
> * Просто следуйте указаниям о том, как добавить принтер, приведенным в общедоступных руководствах по Linux для [SUSE](https://www.suse.com/documentation/sles-12/book_sle_deployment/data/sec_y2_hw_print.html) или [Red Hat и Oracle Linux](https://access.redhat.com/documentation/red_hat_enterprise_linux/6/html/deployment_guide/sec-printer_configuration).
>
>

---
![Сетевая печать][planning-guide-figure-2200]

##### <a name="host-based-printer-over-smb-shared-printer-in-cross-premises-scenario"></a>Доступ по протоколу SMB к принтеру на основе узла (общему принтеру) в распределенном сценарии

Принтеры на основе узла нарочно не являются сетевыми. Но принтер на основе узла можно использовать совместно с разных компьютеров в сети, пока компьютер, к которому подключен такой принтер, остается включенным. Подключитесь к корпоративной сети с помощью подключения "сеть — сеть" или ExpressRoute, и вы сможете совместно использовать локальный принтер. Протокол SMB использует в качестве имени службы NetBIOS, а не DNS. Имя узла NetBIOS может отличаться от имени узла DNS. В стандартном случае имена узла NetBIOS и DNS совпадают. Домен DNS не используется в пространстве имен NetBIOS. Соответственно, в пространстве имен NetBIOS не следует использовать полное имя узла DNS, которое состоит из имени узла DNS и имени домена DNS.

Общий ресурс печати идентифицируется по уникальному имени в пределах сети.

* Имя узла для узла SMB (обязательно).
* Имя общего ресурса (обязательно).
* Имя домена, если общий ресурс печати находится в другом домене относительно системы SAP.
* Кроме того, для доступа к общему ресурсу печати может потребоваться имя пользователя и пароль.

Руководство.

---
> ![Эмблема Windows.][Logo_Windows] Windows
>
> Включите общий доступ к локальному принтеру.
> На виртуальной машине Azure откройте проводник Windows и введите сетевое имя принтера.
> Мастер установки принтера проведет вас через процесс установки.
>
> ![Логотип Linux.][Logo_Linux] Linux
>
> Ниже приведены некоторые примеры документации, посвященной настройке сетевых принтеров в Linux или содержащей разделы о печати в Linux. На виртуальной машине Linux Azure все будет работать точно так же, если виртуальная машина входит в виртуальную частную сеть.
>
> * SLES <https://en.opensuse.org/SDB:Printing_via_SMB_(Samba)_Share_or_Windows_Share>
> * RHEL или Oracle Linux <https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/system_administrators_guide/index#sec-Starting_Print_Settings_Config>
>
>

---
##### <a name="usb-printer-printer-forwarding"></a>USB-принтер (переадресация принтеров)

Службы удаленных рабочих столов предоставляют пользователям доступ к своим локальным устройствам печати в удаленном сеансе, но в Azure эта возможность недоступна.

---
> ![Эмблема Windows.][Logo_Windows] Windows
>
> Дополнительные сведения о печати в Windows можно найти здесь: <https://technet.microsoft.com/library/jj590748.aspx>.
>
>

---
#### <a name="integration-of-sap-azure-systems-into-correction-and-transport-system-tms-in-cross-premises"></a>Интеграция систем SAP Azure в систему исправлений и транспортировки (TMS) при распределенном развертывании

Система исправлений и транспортировки для SAP (TMS) должна быть настроена для экспорта и импорта запросов на перенос между системами, входящими в ландшафт. Мы предполагаем, что экземпляры разработки системы SAP (DEV) находятся в Azure, а системы контроля качества (QA) и производственные системы (PRD) располагаются локально. Кроме того, мы предполагаем, что существует централизованный транспортный каталог.

##### <a name="configuring-the-transport-domain"></a>Настройка транспортного домена

Настройте транспортный домен в системе, которую вы назначили контроллером транспортного домена, как описано в статье [Configuring the Transport Domain Controller](https://help.sap.com/erp2005_ehp_04/helpdata/en/44/b4a0b47acc11d1899e0000e829fbbd/content.htm)(Настройка контроллера транспортного домена). Будет создан пользователь системы TMSADM и обязательное назначение RFC. Подключения RFC вы можете проверить с использованием транзакции SM59. Для вашего транспортного домена должно быть включено разрешение имен узлов.

Руководство.

* В нашем сценарии мы выбрали на роль контроллера домена CTS локальную систему QAS. Выполните транзакцию STMS. Откроется диалоговое окно TMS. Откроется диалоговое окно "Настройка транспортного домена". (Это диалоговое окно отображается только в том случае, если транспортный домен еще не настроен).
* Убедитесь, что автоматически созданный пользователь TMSADM авторизован (SM59 -> ABAP Connection (Подключение ABAP) -> TMSADM@E61.DOMAIN_E61 -> Details (Сведения) -> Utilities (Средства) (М) -> Authorization Test (Проверка авторизации)). Начальный экран транзакции STMS должен подтвердить, что система SAP теперь выполняет роль контроллера транспортного домена, как показано ниже.

![Начальный экран транзакции STMS на контроллере домена][planning-guide-figure-2300]

#### <a name="including-sap-systems-in-the-transport-domain"></a>Включение систем SAP в транспортный домен

Для включения системы SAP в транспортный домен выполните следующие действия.

* В системе разработки, размещенной в Azure, перейдите к транспортной системе (Client 000) и вызовите транзакцию STMS. Выберите в диалоговом окне вариант Other Configuration (Другие настройки), а затем Include System in Domain (Включить систему в домен). Укажите контроллер домена в качестве целевого узла (см. раздел [Включение систем SAP в транспортный домен](https://help.sap.com/erp2005_ehp_04/helpdata/en/44/b4a0c17acc11d1899e0000e829fbbd/content.htm?frameset=/en/44/b4a0b47acc11d1899e0000e829fbbd/frameset.htm)). Система переходит в режим ожидания для включения в транспортный домен.
* По соображениям безопасности вам нужно подтвердить этот запрос, вернувшись к контроллеру домена. Выберите System Overview (Обзор системы) и Approve (Подтвердить) для ожидающей системы. Затем подтвердите запрос. Конфигурация будет распространена.

Эта система SAP теперь содержит необходимые сведения о всех других системах SAP в транспортном домене. В то же время сведения об адресах новой системы SAP отправляются во все остальные системы SAP, и новая система SAP включается в транспортный профиль программы управления транспортом. Проверьте, работают ли RFC и доступ к транспортному каталогу домена.

Продолжайте настройку транспортной системы в обычном режиме, как описано в документе [Change and Transport System](https://help.sap.com/saphelp_nw70ehp3/helpdata/en/48/c4300fca5d581ce10000000a42189c/content.htm?frameset=/en/44/b4a0b47acc11d1899e0000e829fbbd/frameset.htm)(Система изменений и транспортировки).

Руководство.

* Убедитесь, что локальная система STMS настроена правильно.
* Убедитесь, что имя узла контроллера транспортного домена разрешается на виртуальной машине, размещенной в Azure, и наоборот.
* Call transaction STMS (Вызов транзакции STMS) -> Other Configuration (Другие настройки) -> Include System in Domain (Включить систему в домен).
* Подтвердите подключение в локальной системе TMS.
* Настройте транспортные маршруты, группы и уровни, как обычно.

В сценарии распределенного развертывания с подключением типа "сеть — сеть" задержка между локальной сетью и Azure может оказаться значительной. Если мы выполняем транспортировку объектов из сред разработки и тестирования в рабочую среду или мы намерены применить транспорты или пакеты поддержки для различных систем, очевидно, что в зависимости от места расположения централизованного транспортного каталога некоторые системы столкнутся с большими задержками при чтении или записи данных в централизованный транспортный каталог. Ситуация аналогична таким конфигурациям ландшафта SAP, в которых разные системы распределяются по нескольким центрам обработки данных со значительным расстоянием между ними.

Чтобы обойти такую задержку и получить высокую скорость чтения или записи в транспортный каталог, вы можете настроить два транспортных домена STMS (один для локальной сети, а второй для систем в Azure), а затем связать эти транспортные домены. Ознакомьтесь с этой документацией, в которой объясняются принципы этой концепции в SAP TMS: <https://help.sap.com/saphelp_me60/helpdata/en/c4/6045377b52253de10000009b38f889/content.htm?frameset=/en/57/38dd924eb711d182bf0000e829fbfe/frameset.htm> .

Руководство.

* Настройте транспортный домен в каждом расположении (локально и в Azure) с использованием транзакции STMS <https://help.sap.com/saphelp_nw70ehp3/helpdata/en/44/b4a0b47acc11d1899e0000e829fbbd/content.htm>.
* Свяжите домены с помощью связи между ними и подтвердите ее.
  <https://help.sap.com/saphelp_nw73ehp1/helpdata/en/a3/139838280c4f18e10000009b38f8cf/content.htm>
* Распространите конфигурацию на связанную систему.

#### <a name="rfc-traffic-between-sap-instances-located-in-azure-and-on-premises-cross-premises"></a>Трафик RFC между экземплярами SAP в Azure и в локальной среде (распределенное развертывание)

Между системами, размещенными локально и в Azure, должен проходить трафик RFC. Для создания подключения вызовите транзакцию SM59 в исходной системе, определив в ней подключение RFC к целевой системе. Настройка выполняется так же, как и для обычного подключения RFC.

Мы предполагаем, что в распределенном сценарии виртуальные машины, на которых работают системы SAP и которые должны взаимодействовать друг с другом, расположены в одном домене. Поэтому настройка подключения RFC между системами SAP не отличается от обычных действий и параметров для локальных сценариев.

#### <a name="accessing-local-fileshares-from-sap-instances-located-in-azure-or-vice-versa"></a>Доступ к локальным файловым ресурсам из экземпляров SAP, размещенных в Azure, и наоборот

Экземпляры SAP, размещенные в Azure, должны иметь доступ к общим папкам, находящимся в локальной корпоративной среде. Кроме того, локальным экземплярам SAP требуется доступ к общим папкам, расположенным в Azure. Чтобы включить общие папки, следует настроить разрешения и параметры общего доступа в локальной системе. Убедитесь, что открыты нужные порты в подключении VPN или ExpressRoute, установленном между Azure и локальным центром обработки данных.

## <a name="supportability"></a>Возможности поддержки

### <a name="azure-extension-for-sap"></a><a name="6f0a47f3-a289-4090-a053-2521618a28c3"></a>Расширение Azure для SAP

Чтобы передать часть информации об инфраструктуре Azure для критически важных систем SAP в экземпляры агента узла SAP, установленные на виртуальных машинах, необходимо установить расширение Azure для SAP для развернутых виртуальных машин. Так как требования SAP относятся к приложениям SAP, корпорация Майкрософт решила не внедрять эту функциональность в Azure. Мы предлагаем клиентам самостоятельно развернуть необходимое расширение и конфигурации виртуальных машин, работающих в Azure. Но Azure может в значительной степени автоматизировать процессы развертывания и управления жизненным циклом расширения Azure для SAP.

#### <a name="solution-design"></a>Архитектура решения

Решение, разработанное для поддержки агента узла SAP, собирающего необходимую информацию, основано на архитектуре расширения и агента виртуальных машин Azure. Агент ВМ и платформа расширений используются для того, чтобы разрешить установку на виртуальной машине программных приложений, доступных в коллекции расширений для виртуальных машин Azure. Основная идея этой концепции заключается в том, чтобы сделать возможным развертывание специальных функций (например, расширения Azure для SAP) на виртуальной машине и настройку этого ПО во время развертывания.

Агент ВМ Azure, который позволяет управлять на виртуальной машине некоторыми расширениями для виртуальных машин Azure, по умолчанию внедряется в виртуальные машины Windows при создании виртуальной машины на портале Azure. На виртуальных машинах SUSE, Red Hat или Oracle Linux агент и раньше входил в образ, доступный на Azure Marketplace. Если вы передаете виртуальную машину Linux из локальной среды в Azure, установите агент виртуальной машины вручную.

Основные стандартные блоки решения для предоставления сведений об инфраструктуре Azure для агента узла SAP в Azure выглядят следующим образом.

![Компоненты расширения Microsoft Azure][planning-guide-figure-2400]

Как показано в приведенной выше блок-диаграмме, одна из частей решения размещается в коллекции образов виртуальных машин Azure и расширений Azure, которая представляет собой глобально реплицируемый репозиторий, управляемый службой операций Azure. Совместная команда разработки SAP и Майкрософт, занимающаяся реализацией SAP для работы с Azure, регулярно публикует новые версии расширения Azure для SAP.

Когда вы развертываете новую виртуальную машину Windows, в эту виртуальную машину автоматически добавляется агент виртуальной машины Azure. Функция этого агента заключается в координации нагрузки и конфигурации расширений Azure для виртуальных машин. На виртуальных машинах Linux агент виртуальной машины Azure и раньше существовал в образе ОС, доступном в Azure Marketplace.

Однако вам все же потребуется выполнить одну процедуру. Это включение и настройка сбора данных о производительности. Процесс конфигурации можно автоматизировать командами интерфейса командной строки или сценарием PowerShell. Этот сценарий PowerShell можно скачать из Центра сценариев Microsoft Azure, как описано в [руководстве по развертыванию][deployment-guide].

Общая архитектура расширения Azure для SAP выглядит так:

![Расширение Azure для SAP ][planning-guide-figure-2500]

**Подробные инструкции по использованию командлетов PowerShell или команд интерфейса командной строки во время развертывания включены в [руководство по развертыванию][deployment-guide].**

### <a name="integration-of-azure-located-sap-instance-into-saprouter"></a>Интеграция в SAProuter экземпляра SAP, размещенного в Azure

Экземпляры SAP, работающие в Azure, должны быть доступны для SAProuter.

![Сетевое подключение с помощью SAP-Router][planning-guide-figure-2600]

SAProuter позволяет осуществлять взаимодействие между участвующими системами по протоколу TCP/IP, если нет прямого IP-подключения. Это преимущество избавляет от необходимости создавать полное подключение на уровне сети между партнерами по взаимодействию. SAProuter по умолчанию прослушивает порт 3299.
Чтобы связать экземпляры SAP через SAProuter, следует передавать SAProuter строку и имя узла при каждой попытке подключения.

## <a name="sap-netweaver-as-java"></a>SAP NetWeaver AS Java

До сих пор мы рассматривали в этом документе систему SAP NetWeaver в целом или стек SAP NetWeaver ABAP. В этом небольшом разделе мы перечислим специальные рекомендации для стека SAP Java. Одним из самых важных приложений, созданных полностью на основе SAP NetWeaver Java, является корпоративный портал SAP. Еще несколько приложений на основе SAP NetWeaver, например SAP PI и менеджер решений SAP, используют оба стека: SAP NetWeaver ABAP и Java. Это означает, что есть необходимость рассмотреть некоторые аспекты, относящиеся к стеку SAP NetWeaver Java.

### <a name="sap-enterprise-portal"></a>Корпоративный портал SAP

Установка портала SAP на виртуальной машине Azure в распределенных сценариях не отличается от установки в локальной среде. Поскольку управление DNS выполняется локально, можно использовать такие же настройки портов для отдельных экземпляров, как и в локальной среде. Рекомендации и ограничения, описанные в этом документе, относятся и к отдельным приложениям, таким как корпоративный портал SAP, и к стеку SAP NetWeaver Java в целом.

![Портал SAP][planning-guide-figure-2700]

Некоторые клиенты используют особый сценарий развертывания. В нем корпоративный портал SAP напрямую подключен к Интернету, тогда как узел виртуальной машины соединяется с корпоративной сетью через VPN-туннель типа "сеть — сеть" или подключение ExpressRoute. В таком сценарии важно следить за тем, чтобы определенные порты были открыты и не блокировались брандмауэром или группой безопасности сети.

Исходный универсальный код ресурса (URI) портала: http(s):`<Portalserver`>:5XX00/irj, где номер порта вычисляется, как записано SAP в <https://help.sap.com/saphelp_nw70ehp1/helpdata/de/a2/f9d7fed2adc340ab462ae159d19509/frameset.htm>.

![Настройка конечной точки][planning-guide-figure-2800]

Если вы хотите настроить URL-адрес или порты для корпоративного портала SAP, ознакомьтесь со следующей документацией.

* [Change Portal URL (Изменение URL-адреса портала)](https://wiki.scn.sap.com/wiki/display/EP/Change+Portal+URL)
* [Change Default port numbers, Portal port numbers (Изменение номеров портов по умолчанию и номеров портов портала)](https://wiki.scn.sap.com/wiki/display/NWTech/Change+Default++port+numbers%2C+Portal+port+numbers)

## <a name="high-availability-ha-and-disaster-recovery-dr-for-sap-netweaver-running-on-azure-virtual-machines"></a>Высокая доступность и аварийное восстановление для SAP NetWeaver на виртуальных машинах Azure

### <a name="definition-of-terminologies"></a>Определение терминов

Термин **высокая доступность** обычно относится к набору технологий, которые минимизируют перебои в работе ИТ-инфраструктуры, обеспечивая непрерывность работы ИТ-служб благодаря использованию избыточных компонентов, отказоустойчивых компонентов или компонентов, защищенных с помощью отработки отказа в **одном** центре обработки данных. (в нашем случае — в одном регионе Azure).

**Аварийное восстановление** также предназначено для минимизации перебоев в работе ИТ-служб и их восстановления, но в **разных** центрах обработки данных, которые обычно находятся за сотни километров друг от друга. (в нашем случае — в разных регионах Azure в одном и том же геополитическом регионе или в соответствии с требованиями клиента).

### <a name="overview-of-high-availability"></a>Общие сведения о высокой доступности

Описание высокой доступности SAP в Azure можно разделить на две части.

* **Высокая доступность инфраструктуры Azure**, например высокая доступность вычислений (на виртуальных машинах), сети, хранилища и т. д. и ее преимущества для увеличения доступности приложений SAP.
* **Высокая доступность приложений SAP**, например высокая доступность программных компонентов SAP:
  * серверов приложений SAP,
  * экземпляров ASCS/SCS SAP,
  * сервера базы данных,

а также ее совместное использование с высокой доступностью инфраструктуры Azure.

Высокая доступность SAP в Azure несколько отличается от высокой доступности SAP в локальной физической или виртуальной среде. В следующем документе по SAP описываются стандартные конфигурации высокой доступности SAP в виртуализированных средах Windows: <https://scn.sap.com/docs/DOC-44415>. Конфигурации высокой доступности SAP, интегрированной с SAPinst, для Linux (такой же, как для Windows) не существует. Сведения о высокой доступности SAP в локальной среде Linux см. здесь: <https://scn.sap.com/docs/DOC-8541>.

### <a name="azure-infrastructure-high-availability"></a>Высокая доступность инфраструктуры Azure

В настоящее время предоставляется соглашение SLA для одной ВМ с временем бесперебойной работы на уровне 99,9 %. Чтобы понять, как может выглядеть доступность одной виртуальной машины, можно выполнить сборку продукта, регулируемого другими доступными Соглашениями об уровне обслуживания Azure: <https://azure.microsoft.com/support/legal/sla/>.

Основа для расчета — 30 дней в месяц, или 43 200 минут. Таким образом, время простоя 0,05 % соответствует 21,6 мин. Обычно доступность различных служб вычисляется следующим образом:

(доступность службы 1 / 100) * (доступность службы 2 / 100) * (доступность службы 3 / 100)

Например:

(99,95 / 100) * (99,9 / 100) * (99,9 / 100) = 0,9975, или общая доступность 99,75 %.

#### <a name="virtual-machine-vm-high-availability"></a>Высокая доступность виртуальных машин (ВМ)

Существует два типа событий платформы Azure, которые могут повлиять на доступность виртуальных машин: плановое и внеплановое обслуживание.

* События запланированного обслуживания — это периодические обновления, осуществляемые корпорацией Майкрософт на базовой платформе Azure для улучшения общей надежности, производительности и безопасности инфраструктуры платформы, на которой работают ваши виртуальные машины.
* События незапланированного обслуживания происходят в тех случаях, когда в оборудовании или физической инфраструктуре, на основе которых работает виртуальная машина, происходит какая-либо ошибка. Это могут быть сбои локальной сети или локальных жестких дисков, а также другие ошибки на уровне стойки. При выявлении такой ошибки платформа Azure автоматически выполнит перенос вашей виртуальной машины с неработоспособного физического сервера, где она размещена, на исправный. Это происходит редко, но также может быть причиной перезагрузки вашей виртуальной машины.

Дополнительные сведения см. [в статье доступность виртуальных машин Windows в Azure](../../availability.md) и [доступность виртуальных машин Linux в Azure](../../availability.md).

#### <a name="azure-storage-redundancy"></a>Избыточность хранилища Azure

Данные в учетной записи хранения Microsoft Azure всегда реплицируются, обеспечивая надежность, высокую доступность и соответствие Соглашению об уровне обслуживания для службы хранилища Azure даже при временных сбоях оборудования.

Так как в хранилище Azure по умолчанию хранятся три образа данных, использовать RAID5 или RAID1 на нескольких дисках Azure не требуется.

Дополнительные сведения см. в статье [избыточность службы хранилища Azure](../../../storage/common/storage-redundancy.md).

#### <a name="utilizing-azure-infrastructure-vm-restart-to-achieve-higher-availability-of-sap-applications"></a>Использование перезапуска виртуальной машины в инфраструктуре Azure для повышения доступности приложений SAP

Если вы решили не использовать такие функциональные возможности, как отказоустойчивая кластеризация Windows Server (WSFC) или Pacemaker для Linux (поддерживается только для SLES 12 или более поздней версии), для защиты системы SAP от запланированных и незапланированных простоев инфраструктуры физического сервера и всей платформы Azure можно использовать перезапуск виртуальной машины Azure.

> [!NOTE]
> Важно отметить, что перезапуск виртуальной машины Azure предназначен прежде всего для защиты виртуальных машин, а НЕ приложений. Перезапуск виртуальной машины не гарантирует высокую доступность приложений SAP, но обеспечивает определенный уровень доступности инфраструктуры, а значит — опосредованно — и более высокую доступность систем SAP. Кроме того, в соглашении об уровне обслуживания не указано время, необходимое для перезапуска виртуальной машины после запланированного или незапланированного сбоя узла. Таким образом, этот метод обеспечения высокой доступности не подходит для критически важных компонентов системы SAP, таких как (A)SCS или СУБД.
>
>

Другой элемент инфраструктуры, важный для обеспечения высокой доступности, — хранилище. Например, в Соглашении об уровне обслуживания хранилища Azure указана доступность 99.9 %. Если пользователь развернет все виртуальные машины с содержащимися на них дисками в одной учетной записи хранения Azure, потенциальная недоступность хранилища Azure приведет к недоступности всех виртуальных машин, размещенных в этой учетной записи хранения Azure, а также всех компонентов SAP, выполняющихся на этих виртуальных машинах.

Вместо размещения всех виртуальных машин в одной учетной записи хранения Azure можно также использовать выделенные учетные записи хранения для каждой виртуальной машины и таким образом повысить общую доступность виртуальных машин и приложения SAP с помощью нескольких независимых учетных записей хранения Azure.

Управляемые диски Azure автоматически помещаются в домен сбоя виртуальной машины, к которой они подключены. Если в группе доступности размещены две виртуальные машины и при этом используются управляемые диски, платформа позаботится о распространении управляемых дисков в разные домены сбоя. Если вы планируете использовать хранилище класса Premium, мы настоятельно рекомендуем использовать управляемые диски.

Пример архитектуры системы SAP NetWeaver, в которой используется высокая доступность инфраструктуры Azure и учетные записи хранения:

![Схема, на которой показана система SAP NetWeaver, использующая высокую доступность инфраструктуры Azure и учетные записи хранения.][planning-guide-figure-2900]

Пример архитектуры системы SAP NetWeaver, в которой используется высокая доступность инфраструктуры Azure и управляемые диски:

![Использование высокой доступности инфраструктуры Azure для повышения доступности приложений SAP][planning-guide-figure-2901]

Для критически важных компонентов SAP нам на данный момент удалось добиться следующих результатов.

* Высокая доступность серверов приложений (AS) SAP

  Экземпляры серверов приложений SAP являются избыточными компонентами. Каждый экземпляр SAP AS развертывается на отдельной виртуальной машине, которая работает в отдельном домене сбоя и домене обновления Azure (см. разделы [Домены сбоя][planning-guide-3.2.1] и [Домены обновления][planning-guide-3.2.2]). Это обеспечивается за счет групп доступности Azure (см. раздел [Группы доступности Azure][planning-guide-3.2.3]). Потенциальная недоступность домена сбоя или обновления Azure (запланированная или незапланированная) приведет к недоступности ограниченного количества виртуальных машин и их экземпляров SAP AS.

  Если каждый экземпляр SAP AS помещается в отдельную учетную запись хранения Azure, потенциальная недоступность одной учетной записи хранения Azure приведет к недоступности только одной виртуальной машины и соответствующего экземпляра SAP AS. Однако учтите, что существует ограничение на число учетных записей хранения Azure в рамках одной подписки Azure. Чтобы обеспечить автоматический запуск экземпляра (A)SCS после перезапуска виртуальной машины, настройте параметр автозапуска в профиле запуска экземпляра (A)SCS, как описано в разделе [Использование автозапуска для экземпляров SAP][planning-guide-11.5].
  Дополнительные сведения см. в главе [высокий уровень доступности для серверов приложений SAP][planning-guide-11.4.1] .

  Даже при использовании управляемых дисков эти диски также сохраняются в учетной записи хранения Azure и могут быть недоступны в случае сбоя хранилища.

* *Повышение* доступности экземпляра SAP (A)SCS

  Здесь мы используем перезапуск виртуальной машины Azure, чтобы защитить виртуальную машину с помощью установленного экземпляра SAP (A)SCS. В случае запланированного или незапланированного простоя серверов Azure виртуальные машины будут перезапущены на другом доступном сервере. Как упоминалось ранее, перезапуск виртуальной машины Azure в первую очередь защищает виртуальные машины, а не приложения, в данном случае экземпляр (A) SCS. С помощью перезапуска виртуальной машины мы косвенно добьемся повышения доступности экземпляра SAP (A)SCS. Чтобы обеспечить автоматический запуск экземпляра (A)SCS после перезапуска виртуальной машины, настройте параметр автозапуска в профиле запуска экземпляра (A)SCS, как описано в разделе [Использование автозапуска для экземпляров SAP][planning-guide-11.5]. Это означает, что экземпляр (A)SCS как единая точка отказа (SPOF), работающая на одной виртуальной машине, будет решающим фактором доступности всей системы SAP.

* *Повышение* доступности сервера СУБД

  Здесь, как и в случае с использованием экземпляра SAP (A)SCS, мы используем перезапуск виртуальной машины Azure для защиты виртуальной машины с установленным программным обеспечением СУБД и достигаем повышения доступности программного обеспечения СУБД посредством перезапуска виртуальной машины.
  СУБД, выполняемая на одной виртуальной машине, также выполняет функции SPOF и является решающим фактором доступности всей системы SAP.

### <a name="sap-application-high-availability-on-azure-iaas"></a>Высокая доступность приложений SAP в Azure IaaS

Для достижения высокой доступности всей системы SAP необходимо защитить все важные компоненты системы SAP, такие как избыточные серверы приложений SAP, и уникальные компоненты (например, единая точка отказа), такие как экземпляр SAP (A)SCS и СУБД.

#### <a name="high-availability-for-sap-application-servers"></a><a name="5d9d36f9-9058-435d-8367-5ad05f00de77"></a>Высокая доступность серверов приложений SAP

В случае с экземплярами серверов приложений или диалогов SAP специальное решение для обеспечения высокой доступности не требуется. Высокая доступность достигается за счет избыточности и, как следствие, достаточного количества серверов на разных виртуальных машинах. Они должны помещаться в одной и той же группе доступности Azure во избежание одновременного обновления виртуальных машин во время планового обслуживания. Основные функциональные возможности, основанные на разделении доменов обновления и доменов сбоя в рамках единицы масштабирования Azure, уже были описаны выше в разделе [Домены обновления][planning-guide-3.2.2]. Группы доступности Azure были описаны в разделе этого документа [Группы доступности Azure][planning-guide-3.2.3].

Количество доменов сбоя и обновления, которые можно использовать в группе доступности Azure в единице масштабирования Azure, не бесконечно. Это означает, что при помещении нескольких виртуальных машин в одну группу доступности рано или поздно в один и тот же домен сбоя или обновления попадут несколько виртуальных машин.

Если развернуть несколько экземпляров сервера приложений SAP на выделенных виртуальных машинах и предположить, что у нас есть пять доменов обновления, в конце концов сложится ситуация, проиллюстрированная ниже. Фактическое максимальное число доменов сбоя и обновления в группе доступности в будущем может измениться:

![Высокая доступность серверов приложений SAP в Azure][planning-guide-figure-3000]


#### <a name="high-availability-for-sap-central-services-on-azure"></a>Высокий уровень доступности для центральных служб SAP в Azure

Дополнительные сведения об архитектуре высокого уровня доступности центральных служб SAP в Azure см. в статье [Высокодоступная архитектура и сценарии для SAP NetWeaver](./sap-high-availability-architecture-scenarios.md). В этой статье приведено более подробные описания конкретных операционных систем.

#### <a name="high-availability-for-the-sap-database-instance"></a>Высокая доступность экземпляра базы данных SAP

Обычно организация высокой доступности СУБД SAP подразумевает использование двух виртуальных машин с СУБД. В этом случае данные активного экземпляра СУБД реплицируются в пассивный экземпляр на второй ВМ.

Функциональные возможности обеспечения высокой доступности и аварийного восстановления для СУБД в целом и для конкретных СУБД рассматриваются в [руководстве по развертыванию СУБД][dbms-guide].

#### <a name="end-to-end-high-availability-for-the-complete-sap-system"></a>Комплексная высокая доступность полной системы SAP

Ниже приведены два примера полной высокодоступной архитектуры SAP NetWeaver в Azure: по одному для Windows и Linux.

Только неуправляемые диски. Приведенные ниже данные, возможно, будут отличаться от реального положения дел. В частности, речь идет о ситуации, когда вы развертываете много систем SAP и количество развертываемых виртуальных машин превышает ограничение на количество учетных записей хранения в одной подписке. В таких случаях VHD-диски виртуальных машин нужно объединить в одной учетной записи хранения.  Обычно объединяются VHD-диски виртуальных машин уровня приложений SAP из разных систем SAP. Тем самым учитываются ограничения на объем операций ввода-вывода в секунду в учетных записях хранения Azure (<https://azure.microsoft.com/documentation/articles/storage-scalability-targets>).


##### <a name="windows-logologo_windows-ha-on-windows"></a>![Эмблема Windows.][Logo_Windows] Высокая доступность в Windows

![Схема, на которой показана архитектура приложения SAP NetWeaver с высоким уровнем доступности и SQL Server в Azure IaaS.][planning-guide-figure-3200]

Чтобы минимизировать воздействие проблем инфраструктуры и установки исправлений на узлы, система SAP NetWeaver развертывается следующим образом.

* Полная система развертывается в Azure (требуемый уровень СУБД, экземпляр (A) SCS, а полный уровень приложения должен выполняться в том же расположении).
* Вся система работает в рамках одной подписки Azure (обязательное условие).
* Вся система работает в пределах одной виртуальной сети Azure (обязательное условие).
* Разделение виртуальных машин одной системы SAP на три группы доступности возможно, даже если все виртуальные машины находятся в одной виртуальной сети.
* На каждом уровне (например, СУБД, ASCS, серверы приложений) должна использоваться выделенная группа доступности.
* Все виртуальные машины с экземплярами СУБД из одной системы SAP находятся в одной группе доступности. Мы предполагаем, что используется несколько ВМ с экземплярами СУБД на систему, так как используются встроенные функции высокой доступности СУБД (например, SQL Server AlwaysOn или Oracle Data Guard).
* Все виртуальные машины с экземплярами СУБД используют собственную учетную запись хранения. Файлы данных и журналов СУБД реплицируются из одной учетной записи хранения в другую с использованием функций высокой доступности СУБД для синхронизации данных. Недоступность одной учетной записи хранения приведет к недоступности одного узла кластера SQL в Windows, но не всей службы SQL Server.
* Все виртуальные машины с экземпляром (A)SCS из одной системы SAP находятся в одной группе доступности. Для защиты экземпляра (A)SCS на этих ВМ настроена отказоустойчивая кластеризация Windows Server (WSFC).
* Все виртуальные машины с экземплярами (A)SCS используют собственную учетную запись хранения. Файлы экземпляров (A)SCS и глобальные папки SAP реплицируются из одной учетной записи хранения в другую с помощью репликации SIOS DataKeeper. Недоступность одной учетной записи хранения приведет к недоступности одного узла кластера (A)SCS в Windows, но не всей службы (A)SCS.
* Все виртуальные машины, представляющие уровень сервера приложений SAP, находятся в третьей группе доступности.
* Все виртуальные машины с сервером приложений SAP используют собственную учетную запись хранения. Недоступность одной учетной записи хранения приведет к недоступности одного сервера приложений SAP, а остальные серверы приложений SAP будут продолжать работу.

На следующем рисунке показан тот же ландшафт при использовании управляемых дисков.

![Высокодоступная архитектура приложений SAP NetWeaver с сервером SQL Server в Azure IaaS][planning-guide-figure-3201]

##### <a name="linux-logologo_linux-ha-on-linux"></a>![Логотип Linux.][Logo_Linux] Высокая доступность в Linux

Высокодоступная архитектура SAP на платформе Linux в Azure почти такая же, как и для платформы Windows (см. выше). Список поддерживаемых решений с высоким уровнем доступности см. в примечании SAP [1928533].

### <a name="using-autostart-for-sap-instances"></a><a name="4e165b58-74ca-474f-a7f4-5e695a93204f"></a>Использование автозапуска для экземпляров SAP

В SAP есть возможность запуска экземпляров SAP сразу после запуска операционной системы на виртуальной машине. Точные действия описаны в статье базы знаний SAP [1909114]. Если вы используете несколько виртуальных машин или на одной ВМ работает несколько экземпляров, компания SAP не рекомендует использовать этот параметр, так как вы не можете контролировать порядок перезапуска экземпляров. В типичной ситуации с Azure, когда на ВМ работает один экземпляр сервера приложений SAP и перезапускается только одна ВМ, функция автозапуска не играет большой роли. В этом случае ее можно включить, добавив параметр:

`Autostart = 1`

в профиль запуска экземпляра SAP ABAP или Java.

> [!NOTE]
> Автозапуск имеет некоторые недостатки. В частности, этот параметр инициирует запуск экземпляра SAP ABAP или Java, когда в Windows или Linux запускается соответствующая служба экземпляра. Это происходит, например, при загрузке операционной системы. Кроме того, перезапуск служб SAP также обычно происходит при управлении жизненным циклом программного обеспечения SAP (например, SUM или другие обновления). В этом случае автоматический перезапуск экземпляра может иметь негативные последствия. Чтобы избежать их, перед выполнением таких задач параметр автозапуска нужно отключить. Параметр автозапуска также не следует использоваться для кластеризованных экземпляров SAP, таких как ASCS, SCS и CI.
>
>

Дополнительные сведения об автозапуске экземпляров SAP см. здесь:

* [Start/Stop SAP along with your Unix Server Start/Stop (Запуск и остановка SAP одновременно с запуском и остановкой Unix-сервера).](https://scn.sap.com/community/unix/blog/2012/08/07/startstop-sap-along-with-your-unix-server-startstop)
* [Starting and Stopping SAP NetWeaver Management Agents (Запуск и остановка агентов управления SAP NetWeaver).](https://help.sap.com/saphelp_nwpi711/helpdata/en/49/9a15525b20423ee10000000a421938/content.htm)
* [How to enable auto Start of HANA Database (Настройка автоматического запуска базы данных HANA).](http://sapbasisinfo.com/blog/2016/08/15/enabling-autostart-of-sap-hana-database-on-server-boot-situation/)

### <a name="larger-3-tier-sap-systems"></a>Более крупные трехуровневые системы SAP
Аспекты высокой доступности трехуровневых конфигураций SAP рассмотрены в предыдущих разделах. Но что насчет систем, в которых из-за слишком высоких требований сервер СУБД нельзя разместить в Azure, тогда как уровень приложений SAP можно было бы?

#### <a name="location-of-3-tier-sap-configurations"></a>Расположение трехуровневых конфигураций SAP
Ни разделение только уровня приложений, ни разделение уровня приложений и уровня СУБД между локальной средой и Azure не поддерживаются. Система SAP полностью развертывается либо локально, либо в среде Azure. Варианты, когда одна часть серверов приложений работает локально, а вторая — в Azure, также невозможны. Эта информация и будет отправной точкой обсуждения. Кроме того, компоненты СУБД SAP и уровень сервера приложений SAP тоже нельзя развернуть в двух разных регионах Azure. Например, СУБД в западной части США, а уровень приложений SAP в центральной части США. Такие конфигурации не поддерживаются из-за того, что архитектура SAP NetWeaver слишком чувствительна к задержкам.

Тем не менее в прошлом году партнерские ЦОД создали ряд колокационных центров для регионов Azure. Эти центры расположены в тех же регионах Azure, часто рядом с физическими ЦОД Azure. Короткая дистанция и подключение ресурсов в совместном расположении через ExpressRoute к Azure может привести к задержке менее 2 миллисекунд. В таких случаях вы можете разместить уровень СУБД (включая хранилище SAN или NAS) в колокационном центре, а уровень приложений SAP — в Azure. [Крупные экземпляры HANA](./hana-overview-architecture.md).

### <a name="offline-backup-of-sap-systems"></a>Автономное резервное копирование систем SAP
В зависимости от выбранной конфигурации SAP (двух- или трехуровневая) у вас может возникнуть необходимость в резервном копировании. Вам могут потребоваться резервные копии содержимого виртуальной машины и базы данных. Резервное копирование в СУБД должно выполняться средствами самой базы данных. Подробное описание для разных баз данных можно найти в [руководстве по СУБД][dbms-guide]. С другой стороны, резервные копии данных SAP, включая содержимое базы данных, можно создавать в автономном режиме (описано в этом разделе) или оперативном (описано в следующем разделе).

Автономное резервное копирование предполагает завершение работы виртуальной машины на портале Azure с последующим копированием базового диска ВМ и всех подключенных к ней дисков. Таким образом для ВМ и связанных с ней дисков создается резервная копия на определенный момент времени. Резервные копии рекомендуется копировать в другую учетную запись хранения Azure. Сведения о том, как это сделать, см. в разделе документа [Копирование дисков между учетными записями хранения Azure][planning-guide-5.4.2].


Восстановление предыдущего состояния заключается в удалении базовой виртуальной машины, ее исходных дисков и подключенных дисков с последующим копированием сохраненных дисков в исходную учетную запись хранения или группу ресурсов для управляемых дисков и повторным развертыванием системы.
В этой статье приводится пример создания сценария для этого процесса в PowerShell: <https://www.westerndevs.com/_/azure-snapshots/>.

Обязательно установите новую лицензию SAP с момента восстановления резервной копии виртуальной машины, как описано выше. создает новый ключ оборудования.

### <a name="online-backup-of-an-sap-system"></a>Оперативное резервное копирование системы SAP

Резервное копирование СУБД выполняется специализированными средствами СУБД, как описано в [руководстве по СУБД][dbms-guide].

Для других виртуальных машин в системе SAP можно создавать резервные копии с помощью функции резервного копирования виртуальных машин Azure. Функция резервного копирования виртуальных машин является стандартным методом создания полных резервных копий ВМ в Azure. Служба архивации Azure хранит резервные копии в Azure и позволяет восстанавливать состояние виртуальных машин.

> [!NOTE]
> По состоянию на декабрь 2015 г. при резервном копировании виртуальной машины ее уникальный идентификатор, используемый в лицензии SAP, не сохраняется. Это означает, что после восстановления ВМ из резервной копии вам нужно повторно установить лицензионный ключ SAP, так как восстановленная виртуальная машина считается абсолютно новой (а не заменой предыдущей ВМ).
>
> ![Эмблема Windows.][Logo_Windows] Windows
>
> Теоретически для виртуальных машин, на которых работают базы данных, можно также создавать согласованные резервные копии, если системы СУБД поддерживают Windows VSS (служба теневого копирования томов <https://msdn.microsoft.com/library/windows/desktop/bb968832(v=vs.85).aspx>). Например, в SQL Server эта служба поддерживается.
> Но следует помнить, что восстановление состояния базы данных на определенный момент времени невозможно, если резервная копия была создана с помощью функции резервного копирования ВМ Azure. Поэтому мы рекомендуем создавать резервные копии баз данных средствами СУБД, а не полагаться на функцию резервного копирования ВМ Azure.
>
> Ознакомиться с резервным копированием виртуальных машин Azure можно здесь: <https://docs.microsoft.com/azure/backup/backup-azure-vms>.
>
> Кроме того, для резервного копирования и восстановления баз данных вы можете использовать комбинацию Microsoft Data Protection Manager на виртуальной машине Azure и службы архивации Azure. Дополнительные сведения можно найти здесь: <https://docs.microsoft.com/azure/backup/backup-azure-dpm-introduction>.
>
> ![Логотип Linux.][Logo_Linux] Linux
>
> В Linux нет аналога службы Windows VSS, поэтому в этой ОС поддерживаются резервные копии только с согласованностью на уровне файлов, а не приложений. Резервное копирование СУБД SAP должно выполняться средствами СУБД. Файловую систему, в которой хранятся данные SAP, можно сохранить с помощью архиватора tar. Файловую систему, где имеются связанные с SAP данные, можно сохранить, например, с помощью архиватора tar, как описано здесь: <https://help.sap.com/saphelp_nw70ehp2/helpdata/en/d3/c0da3ccbb04d35b186041ba6ac301f/content.htm>.
>
>

### <a name="azure-as-dr-site-for-production-sap-landscapes"></a>Azure как сайт аварийного восстановления для производственных ландшафтов SAP

С середины 2014 г. расширения для различных компонентов Hyper-V, System Center и Azure позволяют использовать Azure как сайт аварийного восстановления для локальных виртуальных машин Hyper-V.

Здесь находится блог с подробными сведениями о развертывании этого решения: <https://docs.microsoft.com/archive/blogs/saponsqlserver/protecting-sap-solutions-with-azure-site-recovery>.

## <a name="summary-for-high-availability-for-sap-systems"></a>Сводка по высокой доступности для систем SAP

В качестве итогов перечислим основные моменты, касающиеся высокой доступности систем SAP в Azure.

* Сейчас единственную точку отказа SAP нельзя создать точно так же, как это можно сделать в локальных развертываниях. Это связано с тем, что кластеры общих дисков пока еще нельзя создавать в Azure без использования стороннего ПО.
* Для уровня СУБД необходимо использовать функции СУБД, не зависящие от технологии общего дискового кластера. Дополнительные сведения см. в [руководстве по СУБД][dbms-guide].
* Чтобы минимизировать влияние обслуживания узлов или проблем в доменах сбоя в инфраструктуре Azure, необходимо использовать группы доступности Azure.
  * Рекомендуется использовать одну группу доступности для уровня приложений SAP.
  * Рекомендуется использовать отдельную группу доступности для уровня СУБД SAP.
  * Не рекомендуется использовать один и тот же набор доступности для виртуальных машин разных систем SAP.
  * Рекомендуется использовать управляемые диски уровня "Премиум".
* Сведения об использовании уровня СУБД SAP в целях архивации см. в [руководстве по СУБД][dbms-guide].
* Резервное копирование экземпляров диалогов SAP не имеет особого смысла, так как повторное развертывание простых экземпляров диалогов обычно занимает меньше времени.
* Резервное копирование виртуальной машины, которая содержит глобальный каталог системы SAP и все профили разных экземпляров, имеет смысл и должно выполняться с помощью программы архивации данных (Windows) или архиватора tar (Linux). Так как между Windows Server 2008 (R2) и Windows Server 2012 (R2) существуют различия (в более новой версии Windows Server процесс резервного копирования упрощен), в качестве гостевой ОС мы рекомендуем использовать версию Windows Server 2012 (R2).

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со статьями:

- [Развертывание виртуальных машин Azure для SAP NetWeaver](./deployment-guide.md)
- [Вопросы развертывания СУБД для рабочей нагрузки SAP на виртуальных машинах Azure](./dbms_guide_general.md)
- [Конфигурации и операции инфраструктуры SAP HANA в Azure](/- azure/virtual-machines/workloads/sap/hana-vm-operations)
