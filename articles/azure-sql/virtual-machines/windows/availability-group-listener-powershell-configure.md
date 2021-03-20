---
title: Настройка прослушивателей групп доступности и балансировщика нагрузки (PowerShell)
description: Настройте прослушиватели группы доступности в модели Azure Resource Manager, используя внутренний балансировщик нагрузки с одним или несколькими IP-адресами.
services: virtual-machines
documentationcenter: na
author: MashaMSFT
editor: monicar
ms.assetid: 14b39cde-311c-4ddf-98f3-8694e01a7d3b
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 02/06/2019
ms.author: mathoma
ms.custom: seo-lt-2019
ms.openlocfilehash: 9337d1c2767923e6dc7c6b267e0c180b460a116e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97359427"
---
# <a name="configure-one-or-more-always-on-availability-group-listeners---resource-manager"></a>Настройка одного или нескольких прослушивателей групп доступности AlwaysOn в модели Resource Manager

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

В этом документе показано, как использовать PowerShell для выполнения одной из следующих задач.
- Создание балансировщика нагрузки
- Добавьте IP-адреса в существующую подсистему балансировки нагрузки для SQL Server групп доступности.

Прослушиватель группы доступности — это виртуальная сеть, к которой подключаются клиенты, чтобы получить доступ к базе данных. На виртуальных машинах Azure подсистема балансировки нагрузки содержит IP-адрес для прослушивателя. Балансировщик нагрузки направляет трафик в экземпляр SQL Server, который прослушивает порт пробы. Обычно группа доступности использует внутреннюю подсистему балансировки нагрузки. В такой системе можно разместить один или несколько IP-адресов. Каждый IP-адрес использует определенный порт пробы. 

Возможность назначения нескольких IP-адресов внутренней подсистеме балансировки нагрузки является новой в Azure и доступна только в модели диспетчер ресурсов. Для выполнения этой задачи необходимо, чтобы группа доступности SQL Server развернута на виртуальных машинах Azure в модели диспетчер ресурсов. Обе виртуальные машины SQL Server должны принадлежать к одной и той же группе доступности. Вы можете использовать [шаблон Майкрософт](./availability-group-quickstart-template-configure.md) , чтобы автоматически создать группу доступности в Azure Resource Manager. Этот шаблон также автоматически создает внутренний балансировщик нагрузки. При необходимости можно [вручную настроить группу доступности AlwaysOn](availability-group-manually-configure-tutorial.md).

Для выполнения действий, описанных в этой статье, группы доступности должны быть уже настроены.  

Связанные разделы включают:

* [Настройка групп доступности AlwaysOn в виртуальной машине Azure (графический пользовательский интерфейс)](availability-group-manually-configure-tutorial.md)   
* [Настройка подключения между виртуальными сетями с помощью Azure Resource Manager и PowerShell](../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

[!INCLUDE [Start your PowerShell session](../../../../includes/sql-vm-powershell.md)]

## <a name="verify-powershell-version"></a>Проверка версии PowerShell

Примеры из этой статьи были проверены с помощью модуля Azure PowerShell 5.4.1.

Убедитесь, что модуль PowerShell находится в 5.4.1 или более поздней версии.

См. статью [Установка модуля Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="configure-the-windows-firewall"></a>Настройка брандмауэра Windows

Настройте брандмауэр Windows для разрешения доступа к SQL Server. Правила брандмауэра разрешают TCP-подключения к портам, используемым экземпляром SQL Server и пробой прослушивателя. Дополнительные сведения см. в разделе [Настройка брандмауэра Windows для доступа к ядру СУБД](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access#Anchor_1). Создайте правило входящего трафика для порта SQL Server и порта пробы.

Ограничивая доступ с помощью группы безопасности сети Azure, убедитесь, что правило разрешения включает IP-адреса виртуальных машин SQL Server и динамический IP-адрес балансировщика нагрузки для прослушивателя AG и IP-адрес основного кластера, если они применяются.

## <a name="determine-the-load-balancer-sku-required"></a>Определение требований для номера SKU в подсистеме балансировки нагрузки

[Балансировщик нагрузки Azure](../../../load-balancer/load-balancer-overview.md) доступен в двух SKU: Basic & Standard. Рекомендуется использовать Load Balancer (цен. категория "Стандартный"). Если виртуальные машины находятся в группе доступности, разрешено использовать Load Balancer (цен. категория "Базовый"). Если виртуальные машины находятся в зоне доступности, требуется Load Balancer (цен. категория "Стандартный"). Для Load Balancer (цен. категория "Стандартный") требуется, чтобы во всех виртуальных машинах использовались стандартные IP-адреса.

В текущем [шаблоне Microsoft](./availability-group-quickstart-template-configure.md) для группы доступности используется Load Balancer (цен. категория "Базовый") с базовыми IP-адресами.

   > [!NOTE]
   > Потребуется настроить [конечную точку службы](../../../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network), если вы используете Load Balancer (цен. категория "Стандартный") и службу хранилища Azure для облачного свидетеля. 
   > 

В данных примерах статьи приведена Load Balancer (цен. категория "Стандартный"). Сценарий `-sku Standard` включен в примеры.

```powershell
$ILB= New-AzLoadBalancer -Location $Location -Name $ILBName -ResourceGroupName $ResourceGroupName -FrontendIpConfiguration $FEConfig -BackendAddressPool $BEConfig -LoadBalancingRule $ILBRule -Probe $SQLHealthProbe -sku Standard
```

Чтобы создать Load Balancer (цен. категория "Базовый"), из строки ее создания необходимо удалить сценарий `-sku Standard`. Пример:

```powershell
$ILB= New-AzLoadBalancer -Location $Location -Name $ILBName -ResourceGroupName $ResourceGroupName -FrontendIpConfiguration $FEConfig -BackendAddressPool $BEConfig -LoadBalancingRule $ILBRule -Probe $SQLHealthProbe
```

## <a name="example-script-create-an-internal-load-balancer-with-powershell"></a>Пример сценария. Создание внутреннего балансировщика нагрузки с помощью PowerShell

> [!NOTE]
> Если вы создали группу доступности с помощью [шаблона Майкрософт](./availability-group-quickstart-template-configure.md), внутренняя подсистема балансировки нагрузки уже создана.

Следующий сценарий PowerShell создает внутренний балансировщик нагрузки, настраивает правила балансировки нагрузки и задает IP-адрес для балансировщика нагрузки. Чтобы запустить скрипт, откройте интегрированную среду сценариев Windows PowerShell и вставьте скрипт в область сценариев. Войдите в PowerShell, используя командлет `Connect-AzAccount`. Если подписок Azure несколько, для установки подписки используется `Select-AzSubscription` . 

```powershell
# Connect-AzAccount
# Select-AzSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>

$ResourceGroupName = "<Resource Group Name>" # Resource group name
$VNetName = "<Virtual Network Name>"         # Virtual network name
$SubnetName = "<Subnet Name>"                # Subnet name
$ILBName = "<Load Balancer Name>"            # ILB name
$Location = "<Azure Region>"                 # Azure location
$VMNames = "<VM1>","<VM2>"                   # Virtual machine names

$ILBIP = "<n.n.n.n>"                         # IP address
[int]$ListenerPort = "<nnnn>"                # AG listener port
[int]$ProbePort = "<nnnn>"                   # Probe port

$LBProbeName ="ILBPROBE_$ListenerPort"       # The Load balancer Probe Object Name              
$LBConfigRuleName = "ILBCR_$ListenerPort"    # The Load Balancer Rule Object Name

$FrontEndConfigurationName = "FE_SQLAGILB_1" # Object name for the front-end configuration 
$BackEndConfigurationName ="BE_SQLAGILB_1"   # Object name for the back-end configuration

$VNet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName 

$Subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName 

$FEConfig = New-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.id

$BEConfig = New-AzLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName 

$SQLHealthProbe = New-AzLoadBalancerProbeConfig -Name $LBProbeName -Protocol tcp -Port $ProbePort -IntervalInSeconds 15 -ProbeCount 2

$ILBRule = New-AzLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig -BackendAddressPool $BEConfig -Probe $SQLHealthProbe -Protocol tcp -FrontendPort $ListenerPort -BackendPort $ListenerPort -LoadDistribution Default -EnableFloatingIP 

$ILB= New-AzLoadBalancer -Location $Location -Name $ILBName -ResourceGroupName $ResourceGroupName -FrontendIpConfiguration $FEConfig -BackendAddressPool $BEConfig -LoadBalancingRule $ILBRule -Probe $SQLHealthProbe 

$bepool = Get-AzLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB 

foreach($VMName in $VMNames)
    {
        $VM = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $VMName 
        $NICName = ($vm.NetworkProfile.NetworkInterfaces.Id.split('/') | select -last 1)
        $NIC = Get-AzNetworkInterface -name $NICName -ResourceGroupName $ResourceGroupName
        $NIC.IpConfigurations[0].LoadBalancerBackendAddressPools = $BEPool
        Set-AzNetworkInterface -NetworkInterface $NIC
        start-AzVM -ResourceGroupName $ResourceGroupName -Name $VM.Name 
    }
```

## <a name="example-script-add-an-ip-address-to-an-existing-load-balancer-with-powershell"></a><a name="Add-IP"></a> Пример сценария. Добавление IP-адреса в имеющуюся подсистему балансировки нагрузки с помощью PowerShell

Чтобы применить несколько групп доступности, добавьте другой IP-адрес в подсистему балансировки нагрузки. Каждому IP-адресу требуется собственное правило балансировки нагрузки, порт пробы и интерфейсный порт.

Внешний порт — это порт, используемый приложениями для подключения к экземпляру SQL Server. Один и тот же внешний порт может использоваться для IP-адресов различных групп доступности.

> [!NOTE]
> Для групп доступности SQL Server для каждого IP-адреса требуется определенный порт пробы. Например, если один IP-адрес в балансировщике нагрузки использует порт пробы 59999, другие IP-адреса не могут использовать этот порт пробы.

* Дополнительные сведения об ограничениях см. в пункте **Частный внешний IP-адрес на балансировщик нагрузки** раздела [Ограничения сети — Azure Resource Manager](../../../azure-resource-manager/management/azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits).
* Дополнительные сведения об ограничениях групп доступности см. в разделе [Ограничения (группы доступности)](/sql/database-engine/availability-groups/windows/prereqs-restrictions-recommendations-always-on-availability#RestrictionsAG).

Следующий сценарий добавляет новый IP-адрес в имеющийся балансировщик нагрузки. ILB использует порт прослушивателя для порта переднего плана балансировки нагрузки. Это может быть порт, который прослушивает SQL Server. Для экземпляров SQL Server по умолчанию — это порт 1433. Для правила балансировки нагрузки для группы доступности требуется плавающий IP-адрес (прямой возврат сервера), поэтому внутренний порт совпадает с интерфейсным портом. Обновите переменные для среды. 

```powershell
# Connect-AzAccount
# Select-AzSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>

$ResourceGroupName = "<ResourceGroup>"          # Resource group name
$VNetName = "<VirtualNetwork>"                  # Virtual network name
$SubnetName = "<Subnet>"                        # Subnet name
$ILBName = "<ILBName>"                          # ILB name                      

$ILBIP = "<n.n.n.n>"                            # IP address
[int]$ListenerPort = "<nnnn>"                   # AG listener port
[int]$ProbePort = "<nnnnn>"                     # Probe port 

$ILB = Get-AzLoadBalancer -Name $ILBName -ResourceGroupName $ResourceGroupName 

$count = $ILB.FrontendIpConfigurations.Count+1
$FrontEndConfigurationName ="FE_SQLAGILB_$count"  

$LBProbeName = "ILBPROBE_$count"
$LBConfigrulename = "ILBCR_$count"

$VNet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName 
$Subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName

$ILB | Add-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.Id 

$ILB | Add-AzLoadBalancerProbeConfig -Name $LBProbeName  -Protocol Tcp -Port $Probeport -ProbeCount 2 -IntervalInSeconds 15  | Set-AzLoadBalancer 

$ILB = Get-AzLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName

$FEConfig = get-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB

$SQLHealthProbe  = Get-AzLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

$BEConfig = Get-AzLoadBalancerBackendAddressPoolConfig -Name $ILB.BackendAddressPools[0].Name -LoadBalancer $ILB 

$ILB | Add-AzLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig  -BackendAddressPool $BEConfig -Probe $SQLHealthProbe -Protocol tcp -FrontendPort  $ListenerPort -BackendPort $ListenerPort -LoadDistribution Default -EnableFloatingIP | Set-AzLoadBalancer   
```

## <a name="configure-the-listener"></a>Настройка прослушивателя

[!INCLUDE [ag-listener-configure](../../../../includes/virtual-machines-ag-listener-configure.md)]

## <a name="set-the-listener-port-in-sql-server-management-studio"></a>Настройка порта прослушивателя в SQL Server Management Studio

1. Запустите SQL Server Management Studio и подключитесь к основной реплике.

1. Перейдите в раздел **Высокий уровень доступности AlwaysOn** > **Группы доступности** > **Прослушиватели группы доступности**. 

1. Теперь вы увидите имя прослушивателя, созданного в диспетчере отказоустойчивости кластеров. Щелкните имя прослушивателя правой кнопкой мыши и выберите пункт **Свойства**.

1. В поле **порт** укажите номер порта для прослушивателя группы доступности, используя $EndpointPort, который использовался ранее (по умолчанию 1433), а затем нажмите кнопку **ОК**.

## <a name="test-the-connection-to-the-listener"></a>Проверка подключения к прослушивателю

Чтобы проверить подключение:

1. Используйте протокол удаленного рабочего стола (RDP) для подключения к SQL Server, который находится в той же виртуальной сети, но не является владельцем реплики. Это может быть другой SQL Server в кластере.

1. Для проверки подключения используйте служебную программу **sqlcmd** . Например, в следующем сценарии **sqlcmd** подключается к основной реплике через прослушиватель с использованием аутентификации Windows.
   
    ```
    sqlcmd -S <listenerName> -E
    ```
   
    Если прослушиватель использует порт, отличный от порта по умолчанию (1433), укажите порт в строке подключения. Например, следующая команда sqlcmd подключается к прослушивателю через порт 1435. 
   
    ```
    sqlcmd -S <listenerName>,1435 -E
    ```

sqlcmd автоматически подключается к любому экземпляру SQL Server, на котором размещена основная реплика. 

> [!NOTE]
> Указанный порт должен быть открыт в брандмауэре обоих серверов SQL Server. Для обоих серверов требуется правило для входящего трафика используемого TCP-порта. Дополнительные сведения см. в статье [Добавление и изменение правила брандмауэра](/previous-versions/orphan-topics/ws.11/cc753558(v=ws.11)). 
> 

## <a name="guidelines-and-limitations"></a>Рекомендации и ограничения

Обратите внимание на следующие рекомендации относительно прослушивателя группы доступности в Azure, использующего внутренний балансировщик нагрузки Azure.

* Если вы используете внутреннюю подсистему балансировки нагрузки, доступ к прослушивателю можно получить только из этой же виртуальной сети.

* Если вы ограничиваюи доступ с помощью группы безопасности сети Azure, убедитесь, что правила Allow включают:
  - IP-адреса серверной SQL Server виртуальной машины
  - Плавающие IP-адреса балансировщика нагрузки для прослушивателя доступности
  - Основной IP-адрес кластера, если применимо.

* Создайте конечную точку службы в случае использования Load Balancer (цен. категория "Стандартный") и службы хранилища Azure для облачного свидетеля. Дополнительные сведения см. в разделе [Предоставление доступа из виртуальной сети](../../../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network).

## <a name="for-more-information"></a>Дополнительные сведения

Дополнительные сведения см. в статье [Введение в группы доступности AlwaysOn SQL Server на виртуальных машинах Azure](availability-group-manually-configure-tutorial.md).

## <a name="powershell-cmdlets"></a>Командлеты PowerShell

Используйте следующие командлеты PowerShell для создания внутренней подсистемы балансировки нагрузки для виртуальных машин Azure.

* Командлет [New-AzLoadBalancer](/powershell/module/Azurerm.Network/New-AzureRmLoadBalancer) создает подсистему балансировки нагрузки. 
* Командлет [New-AzLoadBalancerFrontendIpConfig](/powershell/module/Azurerm.Network/New-AzureRmLoadBalancerFrontendIpConfig) создает конфигурацию внешних IP-адресов для подсистемы балансировки нагрузки. 
* Командлет [New-AzLoadBalancerRuleConfig](/powershell/module/Azurerm.Network/New-AzureRmLoadBalancerRuleConfig) создает конфигурацию правила для подсистемы балансировки нагрузки. 
* Командлет [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/Azurerm.Network/New-AzureRmLoadBalancerBackendAddressPoolConfig) создает конфигурацию внутреннего пула IP-адресов для подсистемы балансировки нагрузки. 
* Командлет [New-AzLoadBalancerProbeConfig](/powershell/module/Azurerm.Network/New-AzureRmLoadBalancerProbeConfig) создает конфигурацию пробы для подсистемы балансировки нагрузки.
* Командлет [Remove-AzLoadBalancer](/powershell/module/Azurerm.Network/Remove-AzureRmLoadBalancer) удаляет подсистему балансировки нагрузки из группы ресурсов Azure.