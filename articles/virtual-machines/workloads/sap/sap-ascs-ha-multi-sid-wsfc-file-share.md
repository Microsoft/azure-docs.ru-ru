---
title: Обеспечение высокого уровня доступности экземпляра SAP ASCS/SCS с несколькими ИД безопасности с помощью отказоустойчивой кластеризации Windows Server и файлового ресурса в Azure | Документация Майкрософт
description: Обеспечение высокого уровня доступности экземпляра SAP ASCS/SCS с несколькими ИД безопасности с помощью отказоустойчивой кластеризации Windows Server и файлового ресурса в Azure
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: cbf18abe-41cb-44f7-bdec-966f32c89325
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 03/15/2021
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 8b1a29b0f94c5009d0535ca92363c25ad5c6c884
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103493498"
---
# <a name="sap-ascsscs-instance-multi-sid-high-availability-with-windows-server-failover-clustering-and-file-share-on-azure"></a>Обеспечение высокого уровня доступности экземпляра ASCS/SCS с несколькими ИД безопасности с помощью отказоустойчивой кластеризации Windows Server и файлового ресурса в Azure

> ![ОС Windows][Logo_Windows] Windows
>

Можно управлять несколькими виртуальными IP-адресами с помощью [внутренней подсистемы балансировки нагрузки Azure][load-balancer-multivip-overview]. 

Если у вас есть развертывание SAP, для создания конфигурации кластера Windows для экземпляров SAP Central Services (ASCS/SCS) можно использовать внутреннюю подсистему балансировки нагрузки.

В этой статье основное внимание уделяется переходу от отдельной установки ASCS/SCS к конфигурации SAP с несколькими ИД безопасности с помощью установки дополнительных кластеризованных экземпляров SAP ASCS/SCS в существующем отказоустойчивом кластере Windows Server (WSFC) с **файловым ресурсом**. Выполнив эти действия, вы настроите кластер SAP с несколькими ИД безопасности.

> [!NOTE]
>
> Эта функция доступна только в модели развертывания с помощью Azure Resource Manager.
>
>Существует ограничение на число частных внешних IP-адресов для каждой внутренней подсистемы балансировки нагрузки Azure.
>
>Максимальное количество экземпляров SAP ASCS/SCS в одном кластере WSFC равно максимальному количеству частных внешних IP-адресов на одну внутреннюю подсистему балансировки нагрузки Azure.
>
> Конфигурация, описанная в этой документации, еще не поддерживает использование [Зон доступности Azure](../../../availability-zones/az-overview.md).
> 

Дополнительные сведения об ограничениях подсистемы балансировки нагрузки см. в разделе [Ограничения сети — Azure Resource Manager][networking-limits-azure-resource-manager]. Также рассмотрите возможность использования [номера SKU Azure Load Balancer (цен. категория "Стандартный")](../../../load-balancer/load-balancer-standard-availability-zones.md) вместо номера SKU Azure Load Balancer (цен. категория "Базовый").

## <a name="prerequisites"></a>Предварительные требования

Вы уже настроили кластер WSFC, который используется для одного экземпляра SAP ASCS/SCS, с помощью **файлового ресурса**, как показано на этой схеме.

![Рис. 1. Экземпляр SAP ASCS/SCS и SOFS, развернутые в двух кластерах][sap-ha-guide-figure-8007]

_**Рис. 1.** Экземпляр SAP ASCS/SCS и SOFS, развернутые в двух кластерах_

> [!IMPORTANT]
> Установка должна соответствовать двум таким условиям:
> * Экземпляры SAP ASCS/SCS должны совместно использовать один кластер WSFC.
> * Другие общие файловые ресурсы глобальных узлов SAP, относящиеся к разным идентификаторам безопасности SAP, должны совместно использовать один кластер SOFS.
> * Экземпляры SAP ASCS/SCS и общие папки SOFS не должны объединяться в одном кластере. 
> * Для каждого идентификатора безопасности системы управления базами данных (СУБД) должен быть выделен кластер WSFC.
> * Серверы приложений SAP, связанные с одним ИД безопасности системы SAP, должны размещаться на выделенных виртуальных машинах.
> * Не поддерживается сочетание сервера репликации очереди 1 и постановки в очередь сервера репликации 2 в одном кластере.  

## <a name="sap-ascsscs-multi-sid-architecture-with-file-share"></a>Архитектура SAP ASCS/SCS с несколькими идентификаторами безопасности и файловым ресурсом

Здесь описано, как установить несколько кластеризованных экземпляров SAP Advanced Business Application Programming (ASCS) или SAP Java (SCS) в одном кластере WSFC, как показано на этой схеме: 

![Рис. 2. Конфигурация SAP с несколькими идентификаторами безопасности в двух кластерах][sap-ha-guide-figure-8008]

_**Рис. 2.** Конфигурация SAP с несколькими идентификаторами безопасности в двух кластерах_

Установка дополнительной системы **\<SID2> SAP** идентична установке одной \<SID> системы. Необходимо выполнить два дополнительных этапа подготовки в кластере ASCS/SCS и в кластере SOFS файлового ресурса.

## <a name="prepare-the-infrastructure-for-an-sap-multi-sid-scenario"></a>Подготовка инфраструктуры для сценария SAP с несколькими ИД безопасности

### <a name="prepare-the-infrastructure-on-the-domain-controller"></a>Подготовка инфраструктуры на контроллере домена

Создайте доменную группу **\<Domain> \ SAP_ \<SID2> _GlobalAdmin**, например with \<SID2> = PR2. Имя группы домена — \<Domain> \ SAP_PR2_GlobalAdmin.

### <a name="prepare-the-infrastructure-on-the-ascsscs-cluster"></a>Подготовка инфраструктуры в кластере ASCS/SCS

Необходимо подготовить инфраструктуру в существующем кластере ASCS/SCS для второго SAP \<SID> :

* Создайте на DNS-сервере имя виртуального узла для кластеризованного экземпляра SAP ASCS/SCS.
* С помощью PowerShell добавьте IP-адрес в существующую внутреннюю подсистему балансировки нагрузки Azure.

Дополнительные сведения см. в статье [о подготовке инфраструктуры для сценария SAP с несколькими идентификаторами безопасности][sap-ascs-ha-multi-sid-wsfc-shared-disk-infrast-prepare].


### <a name="prepare-the-infrastructure-on-an-sofs-cluster-by-using-the-existing-sap-global-host"></a>Подготовка инфраструктуры в кластере SOFS с использованием существующего глобального узла SAP

Вы можете повторно использовать существующие \<SAPGlobalHost> и Volume1 в первой системе SAP \<SID1> .

![Рис. 3. SOFS с несколькими ИД безопасности размещается на глобальном узле SAP][sap-ha-guide-figure-8014]

_**Рис. 3.** SOFS с несколькими ИД безопасности размещается на глобальном узле SAP_

> [!IMPORTANT]
>Для второй системы **SAP \<SID2>** используются те же Volume1 и одно и то же **\<SAPGlobalHost>** Сетевое имя.
>Так как вы уже установили **SAPMNT** в качестве имени общего ресурса для различных систем SAP, для повторного использования **\<SAPGlobalHost>** сетевого имени необходимо использовать тот же **Volume1**.
>
>Путь к файлу \<SID2> глобального узла — C:\ClusterStorage \\ **Volume1**\usr\sap \<SID2> \SYS\.
>

Для \<SID2> системы необходимо подготовить глобальный узел SAP. \SYS \. . в кластере SOFS.

Чтобы подготовить глобальный узел SAP для \<SID2> экземпляра, выполните следующий сценарий PowerShell:


```powershell
##################
# SAP multi-SID
##################

$SAPSID2 = "PR2"
$DomainName2 = "SAPCLUSTER"
$SAPSIDGlobalAdminGroupName2 = "$DomainName2\SAP_" + $SAPSID2 + "_GlobalAdmin"

# SAP ASCS/SCS cluster nodes
$ASCSCluster2Node1 = "ja1-ascs-0"
$ASCSCluster2Node2 = "ja1-ascs-1"

# Define the SAP ASCS/SCS cluster node computer objects
$ASCSCluster2ObjectNode1 = "$DomainName2\$ASCSCluster2Node1$"
$ASCSCluster2ObjectNode2 = "$DomainName2\$ASCSCluster2Node2$"

# Create usr\sap\.. folders on CSV
$SAPGlobalFolder2 = "C:\ClusterStorage\Volume1\usr\sap\$SAPSID2\SYS"
New-Item -Path $SAPGlobalFolder2 -ItemType Directory

# Add permissions for the SAP SID2 system
Grant-SmbShareAccess -Name sapmnt -AccountName $SAPSIDGlobalAdminGroupName2, $ASCSCluster2ObjectNode1, $ASCSCluster2ObjectNode2 -AccessRight Full -Force


$UsrSAPFolder = "C:\ClusterStorage\Volume1\usr\sap\"

# Set file and folder security
$Acl = Get-Acl $UsrSAPFolder

# Add the security object of the SAP_<sid>_GlobalAdmin group
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($SAPSIDGlobalAdminGroupName2,"FullControl", 'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Add the security object of the clusternode1$ computer object
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($ASCSCluster2ObjectNode1,"FullControl",'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Add the security object of the clusternode2$ computer object
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($ASCSCluster2ObjectNode2,"FullControl",'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Set security
Set-Acl $UsrSAPFolder $Acl -Verbose
```

### <a name="prepare-the-infrastructure-on-the-sofs-cluster-by-using-a-different-sap-global-host"></a>Подготовка инфраструктуры в кластере SOFS с использованием другого глобального узла SAP

Можно настроить второй SOFS (например, вторую роль кластера SOFS с **\<SAPGlobalHost2>** и другой **Volume2** для второго **\<SID2>** ).

![Рис. 4. SOFS с несколькими ИД безопасности размещается на глобальном узле SAP 2][sap-ha-guide-figure-8015]

_**Рис. 4.** SOFS с несколькими ИД безопасности размещается на глобальном узле SAP 2_

Чтобы создать вторую роль SOFS с помощью \<SAPGlobalHost2> , выполните следующий сценарий PowerShell:

```powershell
# Create SOFS with SAP Global Host Name 2
$SAPGlobalHostName = "sapglobal2"
Add-ClusterScaleOutFileServerRole -Name $SAPGlobalHostName
```

Создайте второй том **Volume2**. Выполните следующий скрипт PowerShell:

```powershell
New-Volume -StoragePoolFriendlyName S2D* -FriendlyName SAPPR2 -FileSystem CSVFS_ReFS -Size 5GB -ResiliencySettingName Mirror
```

![Рис 5. Второй Volume2 в диспетчере отказоустойчивости кластеров][sap-ha-guide-figure-8016]

_**Рис 5.** Второй Volume2 в диспетчере отказоустойчивости кластеров_

Создайте глобальную папку SAP для второго \<SID2> и задайте безопасность файла.

Выполните следующий скрипт PowerShell:

```powershell
# Create a folder for <SID2> on a second Volume2 and set file security
$SAPSID = "PR2"
$DomainName = "SAPCLUSTER"
$SAPSIDGlobalAdminGroupName = "$DomainName\SAP_" + $SAPSID + "_GlobalAdmin"

# SAP ASCS/SCS cluster nodes
$ASCSClusterNode1 = "ascs-1"
$ASCSClusterNode2 = "ascs-2"

# Define SAP ASCS/SCS cluster node computer objects
$ASCSClusterObjectNode1 = "$DomainName\$ASCSClusterNode1$"
$ASCSClusterObjectNode2 = "$DomainName\$ASCSClusterNode2$"

# Create usr\sap\.. folders on CSV
$SAPGlobalFolder = "C:\ClusterStorage\Volume2\usr\sap\$SAPSID\SYS"
New-Item -Path $SAPGlobalFOlder -ItemType Directory

$UsrSAPFolder = "C:\ClusterStorage\Volume2\usr\sap\"

# Set file and folder security
$Acl = Get-Acl $UsrSAPFolder

# Add the file security object of the SAP_<sid>_GlobalAdmin group
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($SAPSIDGlobalAdminGroupName,"FullControl", 'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Add the security object of the clusternode1$ computer object
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($ASCSClusterObjectNode1,"FullControl",'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Add the security object of the clusternode2$ computer object
$Ar = New-Object  system.security.accesscontrol.filesystemaccessrule($ASCSClusterObjectNode2,"FullControl",'ContainerInherit,ObjectInherit', 'None', 'Allow')
$Acl.SetAccessRule($Ar)

# Set security
Set-Acl $UsrSAPFolder $Acl -Verbose
```

Чтобы создать общую папку SAPMNT в Volume2 с *\<SAPGlobalHost2>* именем узла для второго SAP \<SID2> , запустите мастер **добавления общих файловых ресурсов** в Диспетчер отказоустойчивости кластеров.

Щелкните правой кнопкой мыши группу кластеров SOFS **saoglobal2** и выберите действие **Добавить файловый ресурс**.

![Рис. 6. Запуск мастера добавления файлового ресурса][sap-ha-guide-figure-8017]

_**Рис. 6.** Добавление файлового ресурса в мастере_

<br>

![Рис. 7. "Выбор общего ресурса SMB — быстрая"][sap-ha-guide-figure-8018]

_**Рис 7.** "Общий ресурс SMB — быстрый профиль"_

<br>

![Рис 8. Выбор "sapglobalhost2" и указание пути на Volume2][sap-ha-guide-figure-8019]

_**Рис. 8.** Выберите "sapglobalhost2" и укажите путь в Volume2_

<br>

![Рис. 9. Задание имени общего файлового ресурса для «sapmnt»][sap-ha-guide-figure-8020]

_**Рис. 9.** Задайте для файлового ресурса имя "sapmnt"._

<br>

![Рис. 10. Отключение всех параметров][sap-ha-guide-figure-8021]

_**Рис. 10.** Отключение всех параметров_

<br>

Предоставьте разрешения *Полный доступ* на файлы и ресурс sapmnt для следующих пользователей.
* Группа пользователей домена **SAP_ \<SID> _GlobalAdmin**
* Объект-компьютер для узлов кластера ASCS/SCS **ascs-1$** и **ascs-2$**.

![Рис. 11. Разрешения на полный доступ для учетных записей группы пользователей и компьютера][sap-ha-guide-figure-8022]

_**Рис. 11**. Предоставление полного доступа для учетных записей группы пользователей и компьютера_

<br>

![Рис. 12. Выберите "Создать"][sap-ha-guide-figure-8023]

_**Рис. 12.** Выберите "создать"_

<br>

![Рис. 13. Привязка второго sapmnt к узлу sapglobal2 и создание Volume2][sap-ha-guide-figure-8024]

_**Рис. 13.** Привязка второго sapmnt к узлу sapglobal2 и создание Volume2_

<br>

## <a name="install-sap-netweaver-multi-sid"></a>Установка SAP NetWeaver с несколькими ИД безопасности

### <a name="install-sap-sid2-ascsscs-and-ers-instances"></a>Установка \<SID2> экземпляров SAP ASCS/SCS и ERS

Выполните те же действия по установке и настройке, которые были описаны ранее для одного SAP \<SID> .

### <a name="install-dbms-and-sap-application-servers"></a>Установка СУБД и серверов приложений SAP
Установите СУБД и серверы приложений SAP, как описано выше.

## <a name="next-steps"></a>Дальнейшие действия

* Официальные рекомендации SAP для файлового ресурса высокой доступности: [Installation of an ASCS/SCS Instance on a Failover Cluster with no Shared Disks][sap-official-ha-file-share-document] (Установка экземпляра ASCS/SCS в отказоустойчивом кластере без использования общих дисков)

* [Локальные дисковые пространства в Windows Server 2016][s2d-in-win-2016]

* [Обзор масштабируемого файлового сервера для данных приложений][sofs-overview]

* [Новые возможности хранилища в Windows Server 2016][new-in-win-2016-storage]


[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1869038]:https://launchpad.support.sap.com/#/notes/1869038
[2287140]:https://launchpad.support.sap.com/#/notes/2287140
[2492395]:https://launchpad.support.sap.com/#/notes/2492395

[sap-official-ha-file-share-document]:https://www.sap.com/documents/2017/07/f453332f-c97c-0010-82c7-eda71af511fa.html
[s2d-in-win-2016]:https://docs.microsoft.com/windows-server/storage/storage-spaces/storage-spaces-direct-overview
[sofs-overview]:https://technet.microsoft.com/library/hh831349(v=ws.11).aspx
[new-in-win-2016-storage]:https://docs.microsoft.com/windows-server/storage/whats-new-in-storage

[sap-installation-guides]:http://service.sap.com/instguides
[sap-installation-guides-file-share]:https://www.sap.com/documents/2017/07/f453332f-c97c-0010-82c7-eda71af511fa.html
[networking-limits-azure-resource-manager]:../../../azure-resource-manager/management/azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits
[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[load-balancer-multivip-overview]:../../../load-balancer/load-balancer-multivip-overview.md
[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

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

[sap-ascs-ha-multi-sid-wsfc-shared-disk]:sap-ascs-ha-multi-sid-wsfc-shared-disk.md
[sap-ascs-ha-multi-sid-wsfc-shared-disk-infrast-prepare]:sap-ascs-ha-multi-sid-wsfc-shared-disk.md#25e358f8-92e5-4e8d-a1e5-df7580a39cb0
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

[sap-ha-guide-figure-6001]:media/virtual-machines-shared-sap-high-availability-guide/6001-sap-multi-sid-ascs-scs-sid1.png
[sap-ha-guide-figure-6002]:media/virtual-machines-shared-sap-high-availability-guide/6002-sap-multi-sid-ascs-scs.png
[sap-ha-guide-figure-6003]:media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png
[sap-ha-guide-figure-6004]:media/virtual-machines-shared-sap-high-availability-guide/6004-sap-multi-sid-dns.png
[sap-ha-guide-figure-6005]:media/virtual-machines-shared-sap-high-availability-guide/6005-sap-multi-sid-azure-portal.png
[sap-ha-guide-figure-6006]:media/virtual-machines-shared-sap-high-availability-guide/6006-sap-multi-sid-sios-replication.png

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

[virtual-machines-manage-availability]:../../availability.md
