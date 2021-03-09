---
title: Уровень доступности SAP ASCS/SCS Multi-SID с кластером WSFC и общим диском Azure | Документация Майкрософт
description: Высокий уровень доступности с несколькими ИД безопасности для экземпляра SAP ASCS/SCS с кластером WSFC и общим диском Azure
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
ms.date: 08/12/2020
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: cef0630cfc7ec7d080073085e577153ae7a47ecf
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102504462"
---
# <a name="sap-ascsscs-instance-multi-sid-high-availability-with-windows-server-failover-clustering-and-azure-shared-disk"></a>Высокий уровень доступности экземпляра SAP ASCS/SCS с несколькими ИД безопасности с помощью отказоустойчивой кластеризации Windows Server и общего диска Azure

> ![ОС Windows][Logo_Windows] Windows
>

Эта статья посвящена переходу от одной установки ASCS/SCS к конфигурации SAP с несколькими ИД безопасности путем установки дополнительных кластеризованных экземпляров SAP ASCS/SCS в существующий кластер WSFC с общим диском Windows Server. Выполнив эти действия, вы настроите кластер SAP с несколькими ИД безопасности.

## <a name="prerequisites-and-limitations"></a>Предварительные условия и ограничения

В настоящее время вы можете использовать диски SSD (цен. категория "Премиум") Azure в качестве общего диска Azure для экземпляра SAP ASCS/SCS. Существуют следующие ограничения.

-  [Azure Ultra Disk](../../disks-types.md#ultra-disk) не поддерживается как общий диск Azure для рабочих нагрузок SAP. Сейчас невозможно разместить виртуальные машины Azure с помощью Azure Ultra Disk в группе доступности.
-  [Общий диск Azure](../../disks-shared.md) с SSD (цен. Категория "Премиум")ными дисками поддерживается только для виртуальных машин в группе доступности. Он не поддерживается в Зоны доступности развертывании. 
-  Значение общего диска Azure [максшарес](../../disks-shared-enable.md?tabs=azure-cli#disk-sizes) определяет, сколько узлов кластера может использовать общий диск. Как правило, для экземпляра SAP ASCS/SCS настраиваются два узла в отказоустойчивом кластере Windows, поэтому для параметра `maxShares` необходимо задать значение 2.
-  Все виртуальные машины кластера SAP ASCS/SCS должны быть развернуты в одной [группе размещения](../../windows/proximity-placement-groups.md)службы расположения Azure.   
   Несмотря на то, что вы можете развернуть виртуальные машины кластера Windows в группе доступности с общим диском Azure без ППГ, ППГ обеспечит близкое физическое сходство общих дисков Azure и виртуальных машин кластера, тем самым обеспечивая более низкую задержку между виртуальными машинами и уровнем хранилища.    

Для получения дополнительных сведений об ограничениях для общего диска Azure внимательно ознакомьтесь с разделом [ограничения](../../disks-shared.md#limitations) в документации по общим дискам Azure.  

> [!IMPORTANT]
> При развертывании отказоустойчивого кластера Windows SAP ASCS/SCS с помощью общего диска Azure имейте в виду, что развертывание будет работать с одним общим диском в одном кластере хранилища. В случае возникновения проблем с кластером хранения, в котором развернут общий диск Azure, будет затронуто ваше экземпляр SAP ASCS/SCS.  

> [!IMPORTANT]
> Установка должна соответствовать двум таким условиям:
> * Для каждого идентификатора безопасности системы управления базами данных (СУБД) должен быть выделен кластер WSFC.  
> * Серверы приложений SAP, связанные с одним ИД безопасности системы SAP, должны размещаться на выделенных виртуальных машинах.  
> * Не поддерживается сочетание сервера репликации очереди 1 и постановки в очередь сервера репликации 2 в одном кластере.  


## <a name="supported-os-versions"></a>Поддерживаемые версии ОС

Поддерживаются как Windows Server 2016, так и Windows Server 2019 (Используйте последние образы центра обработки данных).

Настоятельно рекомендуется использовать **Windows Server 2019 Datacenter**, как показано ниже.
- Служба отказоустойчивого кластера Windows 2019 поддерживает Azure
- Добавлена поддержка интеграции и поддержки обслуживания узла Azure и улучшена работа по планированию событий Azure.
- Можно использовать имя распределенной сети (это параметр по умолчанию). Поэтому нет необходимости иметь выделенный IP-адрес для сетевого имени кластера. Кроме того, не нужно настраивать этот IP-адрес во внутренних Load Balancerах Azure. 

## <a name="architecture"></a>Architecture

Сервер репликации с очередью 1 (ERS1) и сервер репликации очереди 2 (ERS2) поддерживаются в конфигурации с несколькими идентификаторами безопасности.  Сочетание ERS1 и ERS2 не поддерживается в одном кластере. 

1. В первом примере показаны два идентификатора безопасности SAP с архитектурой ERS1, где:

   - SAP SID1 развертывается на общем диске с помощью ERS1. Экземпляр ERS устанавливается на локальном узле и на локальном диске.
     SAP SID1 имеет собственный (виртуальный) IP-адрес (SID1 (A) SCS IP1), настроенный во внутренней подсистеме балансировки нагрузки Azure.

   - SAP SID2 развертывается на общем диске с помощью ERS1. Экземпляр ERS устанавливается на локальном узле и на локальном диске.
     SAP SID2 имеет собственный (виртуальный) IP-адрес (SID2 (A) SCS IP2), который также настроен во внутренней подсистеме балансировки нагрузки Azure.

![Высокий уровень доступности экземпляра SAP ASCS/SCS — два экземпляра с конфигурацией ERS1][sap-ha-guide-figure-6007]

2. Во втором примере показаны два идентификатора безопасности SAP с архитектурой ERS2, где: 

   - SAP SID1 с ERS2, который также является кластеризованным и развернут на локальном диске.  
     SAP SID1 имеет собственный (виртуальный) IP-адрес (SID1 (A) SCS IP1), настроенный во внутренней подсистеме балансировки нагрузки Azure.
     SAP ERS2, используемый системой SAP SID1, имеет собственный (виртуальный) IP-адрес (SID1 ERS2 IP2), настроенный во внутренней подсистеме балансировки нагрузки Azure.

   - SAP SID2 с ERS2, который также является кластеризованным и развернут на локальном диске.  
     SAP SID2 имеет собственный (виртуальный) IP-адрес (SID2 (A) SCS IP3), настроенный во внутренней подсистеме балансировки нагрузки Azure.
     SAP ERS2, используемый системой SAP SID2, имеет собственный (виртуальный) IP-адрес (SID2 ERS2 IP4-), настроенный во внутренней подсистеме балансировки нагрузки Azure.

   Здесь у нас всего четыре виртуальных IP-адреса:  
   - SID1 (A) SCS IP1
   - SID2 ERS2 IP2
   - SID2 (A) SCS IP3
   - SID2 ERS2 IP4-

![Высокий уровень доступности экземпляра SAP ASCS/SCS — два экземпляра с конфигурацией ERS1 и ERS2][sap-ha-guide-figure-6008]

## <a name="infrastructure-preparation"></a>Подготовка инфраструктуры

В дополнение к **существующему кластеризованному** экземпляру SAP **PR1** ASCS/SCS мы установим новый идентификатор безопасности SAP **PR2**.  

### <a name="host-names-and-ip-addresses"></a>Имена узлов и IP-адреса

| Роль имени узла | Имя узла | Статический IP-адрес | Группа доступности | Группа размещения с учетом расположения |
| --- | --- | --- |---| ---|
| 1-й кластер узлов кластера ASCS/SCS |PR1-ASCS-10 |10.0.0.4 |PR1-ASCS-avset |PR1PPG |
| кластер ASCS/SCS второго узла кластера |PR1-ASCS-11 |10.0.0.5 |PR1-ASCS-avset |PR1PPG |
| Сетевое имя кластера | pr1clust |10.0.0.42 (**только** для кластера Win 2016) | Недоступно | Недоступно |
| **SID1** Сетевое имя кластера ASCS | PR1 — аскскл |10.0.0.43 | Недоступно | Недоступно |
| **SID1** Сетевое имя кластера ERS (**только** для ERS2) | PR1 — ерскл |10.0.0.44 | Недоступно | Недоступно |
| **SID2** Сетевое имя кластера ASCS | PR2 — аскскл |10.0.0.45 | Недоступно | Недоступно |
| **SID2** Сетевое имя кластера ERS (**только** для ERS2) | PR1 — ерскл |10.0.0.46 | Недоступно | Недоступно |

### <a name="create-azure-internal-load-balancer"></a>Создание внутренней подсистемы балансировки нагрузки Azure

SAP ASCS, SAP SCS и New SAP ERS2 используют виртуальное имя узла и виртуальные IP-адреса. В Azure для использования виртуального IP-адреса требуется [балансировщик нагрузки](../../../load-balancer/load-balancer-overview.md) . Настоятельно рекомендуется использовать [стандартный балансировщик нагрузки](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md). 

Необходимо добавить конфигурацию в существующую подсистему балансировки нагрузки для второго экземпляра SAP SID ASCS/SCS/ERS **PR2**. Конфигурация первого идентификатора безопасности SAP **PR1** должна быть уже выполнена.  

**Конкретного SCS PR2 [номер экземпляра 02]**
- Конфигурация внешнего интерфейса:
    - Статический IP-адрес ASCS/SCS **10.0.0.45**
- Конфигурация серверной части:  
    Уже на месте — виртуальные машины уже добавлены в серверный пул при настройке SAP SID **PR1**
- Порт пробы:
    - Порт 620 **Nr** [**62002**] оставьте параметр по умолчанию для протокола TCP, Interval (5), порог неработоспособности (2)
- Правила балансировки нагрузки
    - Если используется Load Balancer (цен. категория "Стандартный"), выберите порты высокой доступности.
    - Если используется Load Balancer (цен. категория "Базовый"), создайте правила балансировки нагрузки для следующих портов.
        - 32 **Nr** TCP [**3202**]
        - 36 **Nr** TCP [**3602**]
        - 39 **Nr** TCP [**3902**]
        - 81 **Nr** TCP [**8102**]
        - 5 **Nr** 13 TCP [**50213**]
        - 5 **Nr** 14 TCP [**50214**]
        - 5 **Nr** 16 TCP [**50216**]
        - Свяжите с интерфейсным IP-адресом **PR2** ASCS, пробой работоспособности и существующим внутренним пулом.  

    - Убедитесь, что для параметра время ожидания простоя (мин.) задано максимальное значение 30, а также включен параметр плавающий IP-адрес (прямой возврат сервера).

**ERS2 PR2 [номер экземпляра 12]** 

Так как сервер репликации с очередью 2 (ERS2) также является кластеризованным, виртуальный IP-адрес ERS2 также должен быть настроен в Azure ILB в дополнение к выше IP-адреса SAP ASCS/SCS. Этот раздел применим только в том случае, если используется архитектура серверной очереди репликации 2 для **PR2**.  
- Новая конфигурация внешнего интерфейса
    - Статический IP-адрес SAP ERS2 **10.0.0.46**

- Конфигурация серверной части:  
  Виртуальные машины уже добавлены в серверный пул ILB.  

- Новый порт пробы
    - Порт 621 **Nr**  [**62112**] оставьте параметр по умолчанию для протокола TCP, Interval (5), порог неработоспособности (2)

- Новые правила балансировки нагрузки
    - Если используется Load Balancer (цен. категория "Стандартный"), выберите порты высокой доступности.
    - Если используется Load Balancer (цен. категория "Базовый"), создайте правила балансировки нагрузки для следующих портов.
        - 32 **Nr** TCP [**3212**]
        - 33 **Nr** TCP [**3312**]
        - 5 **Nr** 13 TCP [**51212**]
        - 5 **Nr** 14 TCP [**51212**]
        - 5 **Nr** 16 TCP [**51212**]
        - Свяжите с интерфейсным IP-адресом **PR2** ERS2, пробой работоспособности и существующим внутренним пулом.  

    - Убедитесь, что для параметра время ожидания простоя (в минутах) задано максимальное значение, например 30, и этот плавающий IP-адрес включен (прямой возврат сервера).


### <a name="create-and-attach-second-azure-shared-disk"></a>Создание и подключение второго общего диска Azure

Выполните эту команду на одном из узлов кластера. Вам потребуется настроить значения для группы ресурсов, региона Azure, SAPSID и т. д.  

```powershell
    $ResourceGroupName = "MyResourceGroup"
    $location = "MyRegion"
    $SAPSID = "PR2"
    $DiskSizeInGB = 512
    $DiskName = "$($SAPSID)ASCSSharedDisk"
    $NumberOfWindowsClusterNodes = 2
    $diskConfig = New-AzDiskConfig -Location $location -SkuName Premium_LRS  -CreateOption Empty  -DiskSizeGB $DiskSizeInGB -MaxSharesCount $NumberOfWindowsClusterNodes
    
    $dataDisk = New-AzDisk -ResourceGroupName $ResourceGroupName -DiskName $DiskName -Disk $diskConfig
    ##################################
    ## Attach the disk to cluster VMs
    ##################################
    # ASCS Cluster VM1
    $ASCSClusterVM1 = "pr1-ascs-10"
    # ASCS Cluster VM2
    $ASCSClusterVM2 = "pr1-ascs-11"
    # next free LUN number
    $LUNNumber = 1
    # Add the Azure Shared Disk to Cluster Node 1
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $ASCSClusterVM1 
    $vm = Add-AzVMDataDisk -VM $vm -Name $DiskName -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun $LUNNumber
    Update-AzVm -VM $vm -ResourceGroupName $ResourceGroupName -Verbose
    # Add the Azure Shared Disk to Cluster Node 2
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $ASCSClusterVM2
    $vm = Add-AzVMDataDisk -VM $vm -Name $DiskName -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun $LUNNumber
    Update-AzVm -VM $vm -ResourceGroupName $ResourceGroupName -Verbose
   ```
### <a name="format-the-shared-disk-with-powershell"></a>Форматирование общего диска с помощью PowerShell
1. Получите номер диска. Выполните команды PowerShell на одном из узлов кластера:

   ```powershell
    Get-Disk | Where-Object PartitionStyle -Eq "RAW"  | Format-Table -AutoSize 
    # Example output
    # Number Friendly Name     Serial Number HealthStatus OperationalStatus Total Size Partition Style
    # ------ -------------     ------------- ------------ ----------------- ---------- ---------------
    # 3      Msft Virtual Disk               Healthy      Online                512 GB RAW            

   ```
2. Отформатируйте диск. В этом примере это номер диска 3. 

   ```powershell
    # Format SAP ASCS Disk number '3', with drive letter 'S'
    $SAPSID = "PR2"
    $DiskNumber = 3
    $DriveLetter = "S"
    $DiskLabel = "$SAPSID" + "SAP"
    
    Get-Disk -Number $DiskNumber | Where-Object PartitionStyle -Eq "RAW" | Initialize-Disk -PartitionStyle GPT -PassThru |  New-Partition -DriveLetter $DriveLetter -UseMaximumSize | Format-Volume  -FileSystem ReFS -NewFileSystemLabel $DiskLabel -Force -Verbose
    # Example outout
    # DriveLetter FileSystemLabel FileSystem DriveType HealthStatus OperationalStatus SizeRemaining      Size
    # ----------- --------------- ---------- --------- ------------ ----------------- -------------      ----
    # S           PR2SAP          ReFS       Fixed     Healthy      OK                    504.98 GB 511.81 GB
   ```

3. Убедитесь, что диск теперь отображается как диск кластера.  
   ```powershell
    # List all disks
    Get-ClusterAvailableDisk -All
    # Example output
    # Cluster    : pr1clust
    # Id         : c469b5ad-d089-4d8f-ae4c-d834cbbde1a2
    # Name       : Cluster Disk 2
    # Number     : 3
    # Size       : 549755813888
    # Partitions : {\\?\GLOBALROOT\Device\Harddisk3\Partition2\}
   ```

4. Зарегистрируйте диск в кластере.  
   ```powershell
     # Add the disk to cluster 
    Get-ClusterAvailableDisk -All | Add-ClusterDisk
    # Example output 
    # Name           State  OwnerGroup        ResourceType 
    # ----           -----  ----------        ------------ 
    # Cluster Disk 2 Online Available Storage Physical Disk
   ```

## <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance"></a>Создание имени виртуального узла для кластеризованного экземпляра SAP ASCS/SCS

1. Создайте запись DNS для имени виртуального узла для нового экземпляра SAP ASCS/SCS в диспетчере DNS Windows.  
   IP-адрес, назначаемый имени виртуального узла в DNS, должен совпадать с IP-адресом, назначенным в Azure Load Balancer.  

   ![_Define запись DNS для виртуального имени и IP-адреса кластера SAP ASCS/SCS][sap-ha-guide-figure-6009]
 
   _Определение записи DNS для виртуального имени и IP-адреса кластера SAP ASCS/SCS_

2. Если используется очередь SAP с сервером репликации 2, который также является кластеризованным экземпляром, необходимо также зарезервировать в DNS имя виртуального узла для ERS2. 
   IP-адрес, назначаемый имени виртуального узла для ERS2 в DNS, должен совпадать с IP-адресом, назначенным в Azure Load Balancer.  

   ![_Define запись DNS для виртуального имени и IP-адреса кластера SAP ERS2][sap-ha-guide-figure-6010]
 
   _Определение записи DNS для виртуального имени и IP-адреса кластера SAP ERS2_

3. Чтобы определить IP-адрес, назначенный имени виртуального узла, выберите домен **диспетчера DNS**  >  .

   ![Новое виртуальное имя и IP-адрес для конфигурации кластера SAP ASCS/SCS и ERS2][sap-ha-guide-figure-6011]

   _Новое виртуальное имя и TCP/IP-адрес для конфигурации SAP ASCS/SCS и кластера ERS2_

## <a name="sap-installation"></a>Установка SAP 

### <a name="install-the-sap-first-cluster-node"></a> Установка первого узла кластера SAP

Выполните описанную в статье процедуру установки SAP. Убедитесь, что в параметре начать установку выбрано значение "первый узел кластера", и выберите параметр конфигурации "общий диск кластера".  
Выберите только что созданный общий диск.  

### <a name="modify-the-sap-profile-of-the-ascsscs-instance"></a> Изменение профиля SAP экземпляра ASCS/SCS

Если вы используете сервер репликации очереди 1, добавьте параметр профиля SAP, `enque/encni/set_so_keepalive` как описано ниже. Этот параметр профиля запрещает закрытие подключений между рабочими процессами SAP и сервером постановки в очередь, если они простаивают слишком долго. Параметр SAP не является обязательным для ERS2. 

1. Добавьте этот параметр профиля в профиль экземпляра SAP ASCS/SCS, если используется ERS1.

   ```
   enque/encni/set_so_keepalive = true
   ```
   
   Для ERS1 и ERS2 убедитесь, что `keepalive` Параметры ОС заданы, как описано в примечании к SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).   

2. Чтобы применить изменения в параметрах профиля SAP, перезапустите экземпляр SAP ASCS/SCS.

### <a name="configure-probe-port-on-the-cluster-resource"></a>Настройка порта пробы в ресурсе кластера

Чтобы вся конфигурация кластера работала с Azure Load Balancer, нужно использовать функцию проб внутреннего балансировщика нагрузки. Обычно внутренний балансировщик нагрузки Azure распределяет входящую рабочую нагрузку поровну между участвующими виртуальными машинами.

Однако это не будет работать в некоторых конфигурациях кластера, так как активен только один экземпляр. Другой экземпляр пассивен и не может принимать рабочую нагрузку. Функция проверки помогает, когда внутренний балансировщик нагрузки Azure определяет, какой экземпляр является активным, и предназначен только для активного экземпляра.  

> [!IMPORTANT]
> В этом примере конфигурации для **ProbePort** задано значение 620 **Nr**. Для экземпляра SAP ASCS с номером **02** это 620 **02**.
> Необходимо настроить конфигурацию в соответствии с номерами экземпляров SAP и идентификатором безопасности SAP.

Чтобы добавить порт пробы, запустите этот модуль PowerShell на одной из виртуальных машин кластера:

- В случае экземпляра SAP ASC/SCS с номером экземпляра **02** 
   ```powershell
   Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID PR2 -ProbePort 62002
   ```

- При использовании ERS2 с номером экземпляра **12**, который является кластеризованным. Нет необходимости настраивать порт пробы для ERS1, так как он не кластеризован.  
   ```powershell
   Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID PR2 -ProbePort 62012 -IsSAPERSClusteredInstance $True
   ```

 Код функции `Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource` будет выглядеть следующим образом:
   ```powershell
    function Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource {
    <#
    .SYNOPSIS 
    Set-AzureLoadBalancerHealthProbePortOnSAPClusterIPResource will set a new Azure Load Balancer Health Probe Port on 'SAP $SAPSID IP' cluster resource.
    
    .DESCRIPTION
    Set-AzureLoadBalancerHealthProbePortOnSAPClusterIPResource will set a new Azure Load Balancer Health Probe Port on 'SAP $SAPSID IP' cluster resource.
    It will also restart SAP Cluster group (default behavior), to activate the changes. 
    
    You need to run it on one of the SAP ASCS/SCS Windows cluster nodes.
    
    Expectation is that SAP group is installed with official SWPM installation tool, which will set default expected naming convention for:
    - SAP Cluster Group:               'SAP $SAPSID'
    - SAP Cluster IP Address Resource: 'SAP $SAPSID IP' 
    
    .PARAMETER SAPSID 
    SAP SID - 3 characters staring with letter.
    
    .PARAMETER ProbePort 
    Azure Load Balancer Health Check Probe Port.
    
    .PARAMETER RestartSAPClusterGroup 
    Optional parameter. Default value is '$True', so SAP cluster group will be restarted to activate the changes.
    
    .PARAMETER IsSAPERSClusteredInstance 
    Optional parameter.Default value is '$False'.
    If set to $True , then handle clsutered new SAP ERS2 instance.
    
    
    .EXAMPLE 
    # Set probe port to 62000, on SAP cluster resource 'SAP AB1 IP', and restart the SAP cluster group 'SAP AB1', to activate the changes.
    Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID AB1 -ProbePort 62000 
    
    .EXAMPLE 
    # Set probe port to 62000, on SAP cluster resource 'SAP AB1 IP'. SAP cluster group 'SAP AB1' IS NOT restarted, therefore changes are NOT active.
    # To activate the changes you need to manually restart 'SAP AB1' cluster group.
    Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID AB1 -ProbePort 62000 -RestartSAPClusterGroup $False
    
    .EXAMPLE 
    # Set probe port to 62001, on SAP cluster resource 'SAP AB1 ERS IP'. SAP cluster group 'SAP AB1 ERS' IS restarted, to activate the changes.
    Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID AB1 -ProbePort 62000 -IsSAPERSClusteredInstance $True
        
    #> 
    
        [CmdletBinding()]
        param(
            
            [Parameter(Mandatory=$True)]
            [ValidateNotNullOrEmpty()]  
            [ValidateLength(3,3)]      
            [string]$SAPSID,

            [Parameter(Mandatory=$True)]
            [ValidateNotNullOrEmpty()]        
            [int] $ProbePort,
    
            [Parameter(Mandatory=$False)] 
            [bool] $RestartSAPClusterGroup = $True,
    
            [Parameter(Mandatory=$False)] 
            [bool] $IsSAPERSClusteredInstance = $False
        
        )
    
        BEGIN{}
        
        PROCESS{
            try{                                      
                
                if($IsSAPERSClusteredInstance){
                    #Handle clustered SAP ERS Instance
                    $SAPClusterRoleName = "SAP $SAPSID ERS"
                    $SAPIPresourceName = "SAP $SAPSID ERS IP"            
                }else{
                    #Handle clustered SAP ASCS/SCS Instance
                    $SAPClusterRoleName = "SAP $SAPSID"
                    $SAPIPresourceName = "SAP $SAPSID IP"
                }

                $SAPIPResourceClusterParameters =  Get-ClusterResource $SAPIPresourceName | Get-ClusterParameter
                $IPAddress = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Address" }).Value
                $NetworkName = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Network" }).Value
                $SubnetMask = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "SubnetMask" }).Value
                $OverrideAddressMatch = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "OverrideAddressMatch" }).Value
                $EnableDhcp = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "EnableDhcp" }).Value
                $OldProbePort = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "ProbePort" }).Value
    
                $var = Get-ClusterResource | Where-Object {  $_.name -eq $SAPIPresourceName  }
    
                #Write-Host "Current configuration parameters for SAP IP cluster resource '$SAPIPresourceName' are:" -ForegroundColor Cyan
                Write-Output "Current configuration parameters for SAP IP cluster resource '$SAPIPresourceName' are:" 
    
                Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter
    
                Write-Output " "
                Write-Output "Current probe port property of the SAP cluster resource '$SAPIPresourceName' is '$OldProbePort'." 
                Write-Output " "
                Write-Output "Setting the new probe port property of the SAP cluster resource '$SAPIPresourceName' to '$ProbePort' ..." 
                Write-Output " "
    
                $var | Set-ClusterParameter -Multiple @{"Address"=$IPAddress;"ProbePort"=$ProbePort;"Subnetmask"=$SubnetMask;"Network"=$NetworkName;"OverrideAddressMatch"=$OverrideAddressMatch;"EnableDhcp"=$EnableDhcp}
    
                Write-Output " "
    
                #$ActivateChanges = Read-Host "Do you want to take restart SAP cluster role '$SAPClusterRoleName', to activate the changes (yes/no)?"
    
                if($RestartSAPClusterGroup){
                    Write-Output ""
                    Write-Output "Activating changes..." 
    
                    Write-Output " "
                    Write-Output "Taking SAP cluster IP resource '$SAPIPresourceName' offline ..."
                    Stop-ClusterResource -Name $SAPIPresourceName
                    sleep 5
    
                    Write-Output "Starting SAP cluster role '$SAPClusterRoleName' ..."
                    Start-ClusterGroup -Name $SAPClusterRoleName
    
                    Write-Output "New ProbePort parameter is active." 
                    Write-Output " "
    
                    Write-Output "New configuration parameters for SAP IP cluster resource '$SAPIPresourceName':" 
                    Write-Output " " 
                    Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter
                }else
                {
                    Write-Output "SAP cluster role '$SAPClusterRoleName' is not restarted, therefore changes are not activated."
                }
            }
            catch{
               Write-Error  $_.Exception.Message
           }
    
        }
    
        END {}
    }
   ```

### <a name="continue-with-the-sap-installation"></a>Продолжить установку SAP

1. Установите экземпляр базы данных, следуя процедуре, описанной в разделе руководства по установке SAP.  
2. Установите SAP на второй узел кластера, выполнив действия, описанные в руководстве по установке SAP.  
3. Установите экземпляр сервера SAP основного приложения (PAS) на виртуальной машине, назначенной для размещения PAS.   
   Выполните процедуру, описанную в разделе руководства по установке SAP. В Azure нет зависимостей.
4. Установите дополнительные серверы приложений SAP на виртуальных машинах, предназначенных для размещения экземпляров сервера приложений SAP.  
   Выполните процедуру, описанную в разделе руководства по установке SAP. В Azure нет зависимостей. 

## <a name="test-the-sap-ascsscs-instance-failover"></a>Тестирование отработки отказа экземпляра SAP ASCS/SCS
Для тестирования отработки отказа мы предполагаем, что SAP ASCS активен на узле A.  

1. Убедитесь, что система SAP может успешно выполнить отработку отказа с узла A на узел B. В этом примере тест выполнен для SAPSID **PR2**.  
   Убедитесь, что каждый из SAPSID может успешно перейти на другой узел кластера.   
   Выберите один из этих вариантов, чтобы инициировать отработку отказа \<SID\> кластерной группы SAP с узла a на узел B кластера.
    - через диспетчер отказоустойчивости кластеров;  
    - с помощью команд PowerShell для отказоустойчивого кластера.

    ```powershell
    $SAPSID = "PR2"     # SAP <SID>
 
    $SAPClusterGroup = "SAP $SAPSID"
    Move-ClusterGroup -Name $SAPClusterGroup

    ```
2. Перезапустите узел A кластера из гостевой ОС Windows. Это инициирует автоматическую отработку отказа \<SID\> кластерной группы SAP с узла A на узел B.  
3. Перезапустите узел A кластера с помощью портала Azure. Это инициирует автоматическую отработку отказа \<SID\> кластерной группы SAP с узла A на узел B.  
4. Перезапустите узел A кластера с помощью Azure PowerShell. Это инициирует автоматическую отработку отказа \<SID\> кластерной группы SAP с узла A на узел B.

## <a name="next-steps"></a>Дальнейшие действия

* [Узнайте, как подготовить инфраструктуру Azure для SAP высокого уровня доступности с помощью отказоустойчивого кластера Windows и общего диска для экземпляров SAP ASCS/SCS][sap-high-availability-infrastructure-wsfc-shared-disk]
* [Установите SAP NetWeaver высокого уровня доступности в отказоустойчивом кластере Windows на общем диске для экземпляра SAP ASCS/SCS][sap-high-availability-installation-wsfc-shared-disk]


[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1869038]:https://launchpad.support.sap.com/#/notes/1869038
[2287140]:https://launchpad.support.sap.com/#/notes/2287140
[2492395]:https://launchpad.support.sap.com/#/notes/2492395

[sap-installation-guides]:http://service.sap.com/instguides

[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[networking-limits-azure-resource-manager]:../../../azure-resource-manager/management/azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits
[load-balancer-multivip-overview]:../../../load-balancer/load-balancer-multivip-overview.md


[sap-net-weaver-ports]:https://help.sap.com/viewer/ports
[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md
[sap-net-weaver-ports-ascs-scs-ports]:sap-high-availability-infrastructure-wsfc-shared-disk.md#fe0bd8b5-2b43-45e3-8295-80bee5415716

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

[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-file-share]:sap-high-availability-infrastructure-wsfc-file-share.md

[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk-install-ascs]:sap-high-availability-installation-wsfc-shared-disk.md#31c6bd4f-51df-4057-9fdf-3fcbc619c170
[sap-high-availability-installation-wsfc-shared-disk-modify-ascs-profile]:sap-high-availability-installation-wsfc-shared-disk.md#e4caaab2-e90f-4f2c-bc84-2cd2e12a9556
[sap-high-availability-installation-wsfc-shared-disk-add-probe-port]:sap-high-availability-installation-wsfc-shared-disk.md#10822f4f-32e7-4871-b63a-9b86c76ce761
[sap-high-availability-installation-wsfc-shared-disk-win-firewall-probe-port]:sap-high-availability-installation-wsfc-shared-disk.md#4498c707-86c0-4cde-9c69-058a7ab8c3ac
[sap-high-availability-installation-wsfc-shared-disk-change-ers-service-startup-type]:sap-high-availability-installation-wsfc-shared-disk.md#094bc895-31d4-4471-91cc-1513b64e406a
[sap-high-availability-installation-wsfc-shared-disk-test-ascs-failover-and-sios-repl]:sap-high-availability-installation-wsfc-shared-disk.md#18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9


[sap-high-availability-installation-wsfc-file-share]:sap-high-availability-installation-wsfc-file-share.md
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5c8e5482-841e-45e1-a89d-a05c0907c868

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

[sap-ha-guide-figure-6007]:media/virtual-machines-shared-sap-high-availability-guide/6007-sap-multi-sid-ascs-azure-shared-disk-sid1.png
[sap-ha-guide-figure-6008]:media/virtual-machines-shared-sap-high-availability-guide/6008-sap-multi-sid-ascs-azure-shared-disk-sid2.png
[sap-ha-guide-figure-6009]:media/virtual-machines-shared-sap-high-availability-guide/6009-sap-multi-sid-ascs-azure-shared-disk-dns1.png
[sap-ha-guide-figure-6010]:media/virtual-machines-shared-sap-high-availability-guide/6010-sap-multi-sid-ascs-azure-shared-disk-dns2.png
[sap-ha-guide-figure-6011]:media/virtual-machines-shared-sap-high-availability-guide/6011-sap-multi-sid-ascs-azure-shared-disk-dns3.png

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