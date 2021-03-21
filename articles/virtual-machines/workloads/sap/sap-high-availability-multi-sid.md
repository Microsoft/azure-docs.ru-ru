---
title: Создание конфигурации с несколькими идентификаторами безопасности SAP с высоким уровнем доступности в Azure
description: Узнайте, как создать высокодоступную конфигурацию SAP NetWeaver с несколькими ИД безопасности на виртуальных машинах Windows.
services: virtual-machines-windows, virtual-network, storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 0b89b4f8-6d6c-45d7-8d20-fe93430217ca
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/05/2017
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 079ac273f7b2aa91b61ad2cdbcfe62cd989bb2f2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101675263"
---
# <a name="create-a-high-availability-sap-netweaver-multi-sid-configuration"></a>Создание конфигурации с несколькими идентификаторами безопасности SAP NetWeaver с высоким уровнем доступности

[load-balancer-multivip-overview]:../../../load-balancer/load-balancer-multivip-overview.md
[sap-ha-guide]:sap-high-availability-guide.md 
[sap-ha-guide-figure-6001]:./media/virtual-machines-shared-sap-high-availability-guide/6001-sap-multi-sid-ascs-scs-sid1.png
[sap-ha-guide-figure-6002]:./media/virtual-machines-shared-sap-high-availability-guide/6002-sap-multi-sid-ascs-scs.png
[sap-ha-guide-figure-6003]:./media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png
[sap-ha-guide-figure-6004]:./media/virtual-machines-shared-sap-high-availability-guide/6004-sap-multi-sid-dns.png
[sap-ha-guide-figure-6005]:./media/virtual-machines-shared-sap-high-availability-guide/6005-sap-multi-sid-azure-portal.png
[sap-ha-guide-figure-6006]:./media/virtual-machines-shared-sap-high-availability-guide/6006-sap-multi-sid-sios-replication.png
[networking-limits-azure-resource-manager]:../../../azure-resource-manager/management/azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits
[sap-ha-guide-9.1.1]:sap-high-availability-guide.md#a97ad604-9094-44fe-a364-f89cb39bf097 
[sap-ha-guide-8.8]:sap-high-availability-guide.md#f19bd997-154d-4583-a46e-7f5a69d0153c
[sap-ha-guide-8.12.3.3]:sap-high-availability-guide.md#d9c1fc8e-8710-4dff-bec2-1f535db7b006 
[sap-ha-guide-9]:sap-high-availability-guide.md#a06f0b49-8a7a-42bf-8b0d-c12026c5746b
[sap-ha-guide-9.1]:sap-high-availability-guide.md#31c6bd4f-51df-4057-9fdf-3fcbc619c170 
[sap-ha-guide-9.1.2]:sap-high-availability-guide.md#eb5af918-b42f-4803-bb50-eff41f84b0b0 
[sap-ha-guide-9.1.3]:sap-high-availability-guide.md#e4caaab2-e90f-4f2c-bc84-2cd2e12a9556 
[sap-ha-guide-9.1.4]:sap-high-availability-guide.md#10822f4f-32e7-4871-b63a-9b86c76ce761 
[sap-ha-guide-9.4]:sap-high-availability-guide.md#094bc895-31d4-4471-91cc-1513b64e406a 
[sap-ha-guide-9.5]:sap-high-availability-guide.md#2477e58f-c5a7-4a5d-9ae3-7b91022cafb5 
[sap-ha-guide-9.6]:sap-high-availability-guide.md#0ba4a6c1-cc37-4bcf-a8dc-025de4263772 
[sap-ha-guide-10]:sap-high-availability-guide.md#18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9

В сентябре 2016 г. корпорация Майкрософт выпустила функцию, которая дает возможность управлять несколькими виртуальными IP-адресами с помощью внутренней подсистемы балансировки нагрузки Azure. Эта функция уже существует во внешней подсистеме балансировки нагрузки Azure.

При наличии развертывания SAP внутреннюю подсистему балансировки нагрузки Azure можно использовать для создания конфигурации кластера Windows для SAP ASCS/SCS, как описано в статье [SAP NetWeaver на виртуальных машинах Windows. Руководство по обеспечению высокого уровня доступности][sap-ha-guide].

В этой статье основное внимание уделяется переходу от отдельной установки ASCS/SCS к конфигурации SAP с несколькими ИД безопасности с помощью установки дополнительных кластеризованных экземпляров SAP ASCS/SCS в существующем отказоустойчивом кластере Windows Server (WSFC). Выполнив эти действия, вы настроите кластер SAP с несколькими ИД безопасности.

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования
Необходимо настроить кластер WSFC, используемый для одного экземпляра SAP ASCS/SCS, как описано в статье [SAP NetWeaver на виртуальных машинах Windows. Руководство по обеспечению высокого уровня доступности][sap-ha-guide] и как показано на этой диаграмме:

![Высокодоступный экземпляр SAP ASCS/SCS][sap-ha-guide-figure-6001]

## <a name="target-architecture"></a>Целевая архитектура

Цель описанных здесь действий — установить несколько кластеризованных экземпляров SAP ABAP ASCS или SAP Java SCS в одном кластере WSFC, как показано на этой схеме:

![Несколько кластеризованных экземпляров SAP ASCS/SCS в Azure][sap-ha-guide-figure-6002]

> [!NOTE]
>Существует ограничение для числа частных внешних IP-адресов на одну внутреннюю подсистему балансировки нагрузки Azure.
>
>Максимальное количество экземпляров SAP ASCS/SCS в одном кластере WSFC равно максимальному количеству частных внешних IP-адресов на одну внутреннюю подсистему балансировки нагрузки Azure.
>

Дополнительные сведения об ограничениях подсистемы балансировки нагрузки см. в пункте "Частный внешний IP-адрес на подсистему балансировки нагрузки" раздела [Ограничения сети — Azure Resource Manager][networking-limits-azure-resource-manager].

Общая картина с двумя системами SAP высокого уровня доступности будет выглядеть следующим образом:

![Установка SAP высокого уровня доступности с несколькими ИД безопасности и с двумя ИД безопасности системы SAP][sap-ha-guide-figure-6003]

> [!IMPORTANT]
> Установка должна соответствовать двум таким условиям:
> - Экземпляры SAP ASCS/SCS должны совместно использовать один кластер WSFC.
> - Для каждого ИД безопасности СУБД должен быть выделен кластер WSFC.
> - Серверы приложений SAP, связанные с одним ИД безопасности системы SAP, должны размещаться на выделенных виртуальных машинах.


## <a name="prepare-the-infrastructure"></a>Подготовка инфраструктуры
Для подготовки инфраструктуры можно установить дополнительный экземпляр SAP ASCS/SCS со следующими параметрами.

| Имя параметра | Значение |
| --- | --- |
| ИД безопасности SAP ASCS/SCS |pr1-lb-ascs |
| Внутренний балансировщик нагрузки экземпляра СУБД SAP | PR5 |
| Имя виртуального узла SAP | pr5-sap-cl |
| IP-адрес виртуального узла SAP ASCS/SCS (IP-адрес дополнительной подсистемы Azure Load Balancer) | 10.0.0.50 |
| Количество экземпляров SAP ASCS/SCS | 50 |
| Порт пробы внутренней подсистемы балансировки нагрузки для дополнительного экземпляра SAP ASCS/SCS | 62350 |

> [!NOTE]
> В случае с экземплярами кластера SAP ASCS/SCS каждому IP-адресу требуется уникальный порт пробы. Например, если один IP-адрес во внутренней подсистеме балансировки нагрузки Azure использует порт пробы 62300, другой IP-адрес не может использовать этот порт.
>
>Так как порт пробы 62300 уже зарезервирован, для наших целей мы используем порт пробы 62350.

Установить дополнительные экземпляры SAP ASCS/SCS можно в существующем кластере WSFC с двумя узлами:

| Роль виртуальной машины | Имя узла виртуальной машины | Статический IP-адрес |
| --- | --- | --- |
| 1-й узел кластера для экземпляра ASCS/SCS |pr1-ascs-0 |10.0.0.10 |
| 2-й узел кластера для экземпляра ASCS/SCS |pr1-ascs-1 |10.0.0.9 |

### <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance-on-the-dns-server"></a>Создание имени виртуального узла для кластеризованного экземпляра SAP ASCS/SCS на DNS-сервере

Вы можете создать запись DNS для имени виртуального узла экземпляра ASCS/SCS с использованием следующих параметров.

| Новое имя виртуального узла SAP ASCS/SCS | Связанный IP-адрес |
| --- | --- |
|pr5-sap-cl |10.0.0.50 |

Новое имя узла и IP-адрес отображаются в диспетчере DNS, как показано на следующем снимке экрана:

![Список диспетчера DNS с выделенной определенной записью DNS для нового виртуального имени и TCP/IP-адреса кластера SAP ASCS/SCS][sap-ha-guide-figure-6004]

Процедура создания записи DNS также подробно описана в статье [SAP NetWeaver на виртуальных машинах Windows. Руководство по обеспечению высокого уровня доступности][sap-ha-guide-9.1.1].

> [!NOTE]
> Новый IP-адрес, назначаемый имени виртуального узла дополнительного экземпляра ASCS/SCS, должен совпадать с новым IP-адресом, который назначен подсистеме Azure Load Balancer для SAP.
>
>В нашем сценарии используется IP-адрес 10.0.0.50.

### <a name="add-an-ip-address-to-an-existing-azure-internal-load-balancer-by-using-powershell"></a>Добавление IP-адреса в существующую внутреннюю подсистему балансировки нагрузки Azure с помощью PowerShell

Чтобы создать несколько экземпляров SAP ASCS/SCS в одном кластере WSFC, добавьте дополнительный IP-адрес в существующую внутреннюю подсистему балансировки Azure с помощью PowerShell. Каждому IP-адресу требуются собственные правила балансировки нагрузки, порт пробы, пул внешних IP-адресов и внутренний пул.

Следующий сценарий добавляет новый IP-адрес в имеющийся балансировщик нагрузки. Обновите переменные PowerShell для своей среды. Скрипт создаст все необходимые правила балансировки нагрузки для всех портов SAP ASCS/SCS.

```powershell

# Select-AzSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>
Clear-Host
$ResourceGroupName = "SAP-MULTI-SID-Landscape"      # Existing resource group name
$VNetName = "pr2-vnet"                        # Existing virtual network name
$SubnetName = "Subnet"                        # Existing subnet name
$ILBName = "pr2-lb-ascs"                      # Existing ILB name                      
$ILBIP = "10.0.0.50"                          # New IP address
$VMNames = "pr2-ascs-0","pr2-ascs-1"          # Existing cluster virtual machine names
$SAPInstanceNumber = 50                       # SAP ASCS/SCS instance number: must be a unique value for each cluster
[int]$ProbePort = "623$SAPInstanceNumber"     # Probe port: must be a unique value for each IP and load balancer

$ILB = Get-AzLoadBalancer -Name $ILBName -ResourceGroupName $ResourceGroupName

$count = $ILB.FrontendIpConfigurations.Count + 1
$FrontEndConfigurationName ="lbFrontendASCS$count"
$LBProbeName = "lbProbeASCS$count"

# Get the Azure VNet and subnet
$VNet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName
$Subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName

# Add second front-end and probe configuration
Write-Host "Adding new front end IP Pool '$FrontEndConfigurationName' ..." -ForegroundColor Green
$ILB | Add-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.Id
$ILB | Add-AzLoadBalancerProbeConfig -Name $LBProbeName  -Protocol Tcp -Port $Probeport -ProbeCount 2 -IntervalInSeconds 10  | Set-AzLoadBalancer

# Get new updated configuration
$ILB = Get-AzLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName
# Get new updated LP FrontendIP COnfig
$FEConfig = Get-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB
$HealthProbe  = Get-AzLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

# Add new back-end configuration into existing ILB
$BackEndConfigurationName  = "backendPoolASCS$count"
Write-Host "Adding new backend Pool '$BackEndConfigurationName' ..." -ForegroundColor Green
$BEConfig = Add-AzLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB | Set-AzLoadBalancer

# Get new updated config
$ILB = Get-AzLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName

# Assign VM NICs to backend pool
$BEPool = Get-AzLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB
foreach($VMName in $VMNames){
        $VM = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $VMName
        $NICName = ($VM.NetworkInterfaceIDs[0].Split('/') | select -last 1)        
        $NIC = Get-AzNetworkInterface -name $NICName -ResourceGroupName $ResourceGroupName                
        $NIC.IpConfigurations[0].LoadBalancerBackendAddressPools += $BEPool
        Write-Host "Assigning network card '$NICName' of the '$VMName' VM to the backend pool '$BackEndConfigurationName' ..." -ForegroundColor Green
        Set-AzNetworkInterface -NetworkInterface $NIC
        #start-AzVM -ResourceGroupName $ResourceGroupName -Name $VM.Name
}


# Create load-balancing rules
$Ports = "445","32$SAPInstanceNumber","33$SAPInstanceNumber","36$SAPInstanceNumber","39$SAPInstanceNumber","5985","81$SAPInstanceNumber","5$SAPInstanceNumber`13","5$SAPInstanceNumber`14","5$SAPInstanceNumber`16"
$ILB = Get-AzLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName
$FEConfig = get-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB
$BEConfig = Get-AzLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB
$HealthProbe  = Get-AzLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

Write-Host "Creating load balancing rules for the ports: '$Ports' ... " -ForegroundColor Green

foreach ($Port in $Ports) {

        $LBConfigrulename = "lbrule$Port" + "_$count"
        Write-Host "Creating load balancing rule '$LBConfigrulename' for the port '$Port' ..." -ForegroundColor Green

        $ILB | Add-AzLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig  -BackendAddressPool $BEConfig -Probe $HealthProbe -Protocol tcp -FrontendPort  $Port -BackendPort $Port -IdleTimeoutInMinutes 30 -LoadDistribution Default -EnableFloatingIP   
}

$ILB | Set-AzLoadBalancer

Write-Host "Successfully added new IP '$ILBIP' to the internal load balancer '$ILBName'!" -ForegroundColor Green

```
После выполнения скрипта результаты отобразятся на портале Azure, как показано на следующем снимке экрана:

![Новый пул внешних IP-адресов на портале Azure][sap-ha-guide-figure-6005]

### <a name="add-disks-to-cluster-machines-and-configure-the-sios-cluster-share-disk"></a>Добавление дисков на кластерные компьютеры и настройка общего диска для кластера SIOS

Для каждого дополнительного экземпляра SAP ASCS/SCS необходимо добавить новый общий диск кластера. Для общего диска кластера WSFC Windows Server 2012 R2 сейчас используется программное решение SIOS DataKeeper.

Выполните следующие действия.
1. Добавьте дополнительный диск (или несколько дисков одного размера, которые необходимо чередовать) к каждому узлу кластера и отформатируйте их.
2. Настройте репликацию хранилища с помощью SIOS DataKeeper.

Для выполнения этой процедуры необходимо установить SIOS DataKeeper на компьютерах кластера WSFC. Если это программное решение установлено, теперь нужно настроить репликацию между компьютерами. Процедура подробно описана в статье [SAP NetWeaver на виртуальных машинах Windows. Руководство по обеспечению высокого уровня доступности][sap-ha-guide-8.12.3.3].  

![Синхронное зеркальное отображение DataKeeper для нового общего диска SAP ASCS/SCS][sap-ha-guide-figure-6006]

### <a name="deploy-vms-for-sap-application-servers-and-dbms-cluster"></a>Развертывание виртуальных машин для серверов приложений SAP и кластера СУБД

Чтобы завершить подготовку инфраструктуры для второй системы SAP, необходимо сделать следующее.

1. Разверните выделенные виртуальные машины для серверов приложений SAP и поместите их в отдельную выделенную группу доступности.
2. Разверните выделенные виртуальные машины для кластера СУБД и поместите их в отдельную выделенную группу доступности.


## <a name="install-the-second-sap-sid2-netweaver-system"></a>Установка второй системы SAP SID2 NetWeaver

Вся процедура установки второй системы SAP SID2 подробно описана в статье [SAP NetWeaver на виртуальных машинах Windows. Руководство по обеспечению высокого уровня доступности][sap-ha-guide-9].

Основные действия:

1. [Установка первого узла кластера SAP.][sap-ha-guide-9.1.2]  
 На этом шаге устанавливается SAP с высокодоступным экземпляром ASCS/SCS на **СУЩЕСТВУЮЩЕМ узле 1 кластера WSFC**.

2. [Изменение профиля SAP для экземпляра ASCS/SCS.][sap-ha-guide-9.1.3]

3. [Настройка порта пробы.][sap-ha-guide-9.1.4]  
 На этом шаге настраивается порт пробы SAP-SID2-IP кластерного ресурса SAP с помощью PowerShell. Настройку следует выполнять на одном из узлов кластера SAP ASCS/SCS.

4. [Установка экземпляра базы данных][sap-ha-guide-9.2].  
 На этом шаге в выделенном кластере WSFC устанавливается СУБД.

5. [Установка второго узла кластера][sap-ha-guide-9.3].  
 На этом шаге устанавливается SAP с высокодоступным экземпляром ASCS/SCS на существующем узле 2 кластера WSFC.

6. Открытие портов брандмауэра Windows для экземпляра SAP ASCS/SCS и порта пробы.  
 На обоих узлах кластера, используемых для экземпляров SAP ASCS/SCS, откройте все порты брандмауэра Windows, используемые SAP ASCS/SCS. Эти порты подробно описаны в статье [SAP NetWeaver на виртуальных машинах Windows. Руководство по обеспечению высокого уровня доступности][sap-ha-guide-8.8].  
 Кроме того, откройте порт пробы внутренней подсистемы балансировки нагрузки Azure. В нашем случае это порт 62350.

7. [Изменение типа запуска службы Windows для экземпляра SAP ERS.][sap-ha-guide-9.4]

8. [Установка основного сервера приложений SAP][sap-ha-guide-9.5] на новой выделенной виртуальной машине.

9. [Установка дополнительного сервера приложений SAP][sap-ha-guide-9.6] на новой выделенной виртуальной машине.

10. [Тестирование отработки отказа экземпляра SAP ASCS/SCS и репликации SIOS.][sap-ha-guide-10]

## <a name="next-steps"></a>Дальнейшие действия

- [Ограничения сети — Azure Resource Manager][networking-limits-azure-resource-manager]
- [Несколько виртуальных IP-адресов для Azure Load Balancer][load-balancer-multivip-overview]
- [SAP NetWeaver на виртуальных машинах Windows. Руководство по обеспечению высокого уровня доступности][sap-ha-guide]
