---
title: Развертывание и настройка брандмауэра Azure с помощью Azure PowerShell
description: Из этой статьи вы узнаете, как развернуть и настроить брандмауэр Azure с помощью Azure PowerShell.
services: firewall
author: vhorne
ms.service: firewall
ms.date: 12/03/2020
ms.author: victorh
ms.topic: how-to
ms.openlocfilehash: dc36d45e226cffafb51cf7aa09ea6f0d528ee016
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98051383"
---
# <a name="deploy-and-configure-azure-firewall-using-azure-powershell"></a>Развертывание и настройка брандмауэра Azure с помощью Azure PowerShell

Управление доступом исходящих сетевых подключений является важной частью общего плана безопасности сети. Например, вы можете ограничить доступ к веб-сайтам или исходящим IP-адресам и портам, которые могут быть доступными.

Вы можете управлять доступом к исходящей сети из подсети Azure только с помощью Брандмауэра Azure. Брандмауэр Azure позволяет настроить:

* Правила приложений, определяющие полные доменные имена (FQDN), к которым можно получить доступ из подсети.
* Правила сети, определяющие адрес источника, протокол, порт назначения и адрес назначения.

При маршрутизации трафика на брандмауэр, используемый в качестве шлюза по умолчанию, для подсети к трафику применяются настроенные правила брандмауэра.

В этой статье вы создадите упрощенную отдельную виртуальную сеть с тремя подсетями для простоты развертывания. Для развертываний в рабочей среде рекомендуется использовать [звездообразную модель](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke), в которой брандмауэр находится в собственной виртуальной сети. Серверы рабочей нагрузки размещены в одноранговых виртуальных сетях в одном регионе с одной или несколькими подсетями.

* **AzureFirewallSubnet** — в этой подсети находится брандмауэр.
* **Workload-SN** — в этой подсети находится сервер рабочей нагрузки. Трафик этой подсети проходит через брандмауэр.
* **Азуребастионсубнет** — подсеть, используемая для Azure бастиона, которая используется для подключения к серверу рабочей нагрузки. Дополнительные сведения об Azure бастиона см. в статье [что такое Azure бастиона?](../bastion/bastion-overview.md)

![Инфраструктура сети, используемая в руководстве](media/deploy-ps/tutorial-network.png)

Вы узнаете, как выполнять следующие задачи:


* настройка тестовой сетевой среды;
* развертывание брандмауэра;
* создание маршрута по умолчанию;
* настройка правила приложения для предоставления доступа к [www.google.com]\(www.google.com);
* настройка сетевых правил для предоставления доступа к внешним DNS-серверам;
* тестирование брандмауэра.

При желании эту процедуру можно выполнить с помощью [портал Azure](tutorial-firewall-deploy-portal.md).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Для выполнения этой процедуры необходимо запустить PowerShell локально. Необходимо установить модуль Azure PowerShell. Чтобы узнать версию, выполните команду `Get-Module -ListAvailable Az`. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). После проверки версии PowerShell выполните командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="set-up-the-network"></a>Настройка сети

Сначала создайте группу ресурсов, необходимых для развертывания брандмауэра. Затем создайте виртуальную сеть, подсети и тестовые серверы.

### <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов содержит все ресурсы для развертывания.

```azurepowershell
New-AzResourceGroup -Name Test-FW-RG -Location "East US"
```

### <a name="create-a-virtual-network-and-azure-bastion-host"></a>Создание виртуальной сети и узла Бастиона Azure

Эта виртуальная сеть имеет три подсети:

> [!NOTE]
> Размер подсети AzureFirewallSubnet равен /26. Дополнительные сведения о размере подсети см. в статье с [часто задаваемыми вопросами о Брандмауэре Azure](firewall-faq.yml#why-does-azure-firewall-need-a--26-subnet-size).

```azurepowershell
$Bastionsub = New-AzVirtualNetworkSubnetConfig -Name AzureBastionSubnet -AddressPrefix 10.0.0.0/27
$FWsub = New-AzVirtualNetworkSubnetConfig -Name AzureFirewallSubnet -AddressPrefix 10.0.1.0/26
$Worksub = New-AzVirtualNetworkSubnetConfig -Name Workload-SN -AddressPrefix 10.0.2.0/24
```
Теперь создайте виртуальную сеть.

```azurepowershell
$testVnet = New-AzVirtualNetwork -Name Test-FW-VN -ResourceGroupName Test-FW-RG `
-Location "East US" -AddressPrefix 10.0.0.0/16 -Subnet $Bastionsub, $FWsub, $Worksub
```
### <a name="create-public-ip-address-for-azure-bastion-host"></a>Создание общедоступного IP-адреса для узла Бастиона Azure

```azurepowershell
$publicip = New-AzPublicIpAddress -ResourceGroupName Test-FW-RG -Location "East US" `
   -Name Bastion-pip -AllocationMethod static -Sku standard
```

### <a name="create-azure-bastion-host"></a>Создание узла-бастиона Azure

```azurepowershell
New-AzBastion -ResourceGroupName Test-FW-RG -Name Bastion-01 -PublicIpAddress $publicip -VirtualNetwork $testVnet
```
### <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Теперь создайте виртуальную машину рабочей нагрузки и поместите ее в соответствующую подсеть.
При появлении запроса введите имя пользователя и пароль для виртуальной машины.


Создайте виртуальную машину рабочей нагрузки.
При появлении запроса введите имя пользователя и пароль для виртуальной машины.

```azurepowershell
#Create the NIC
$wsn = Get-AzVirtualNetworkSubnetConfig -Name  Workload-SN -VirtualNetwork $testvnet
$NIC01 = New-AzNetworkInterface -Name Srv-Work -ResourceGroupName Test-FW-RG -Location "East us" -Subnet $wsn

#Define the virtual machine
$VirtualMachine = New-AzVMConfig -VMName Srv-Work -VMSize "Standard_DS2"
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName Srv-Work -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC01.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2019-Datacenter' -Version latest

#Create the virtual machine
New-AzVM -ResourceGroupName Test-FW-RG -Location "East US" -VM $VirtualMachine -Verbose
```

## <a name="deploy-the-firewall"></a>Развертывание брандмауэра

Теперь разверните брандмауэр в виртуальной сети.

```azurepowershell
# Get a Public IP for the firewall
$FWpip = New-AzPublicIpAddress -Name "fw-pip" -ResourceGroupName Test-FW-RG `
  -Location "East US" -AllocationMethod Static -Sku Standard
# Create the firewall
$Azfw = New-AzFirewall -Name Test-FW01 -ResourceGroupName Test-FW-RG -Location "East US" -VirtualNetwork $testVnet -PublicIpAddress $FWpip

#Save the firewall private IP address for future use

$AzfwPrivateIP = $Azfw.IpConfigurations.privateipaddress
$AzfwPrivateIP
```

Запишите частный IP-адрес. Вы будете использовать его позже при создании маршрута по умолчанию.

## <a name="create-a-default-route"></a>создание маршрута по умолчанию;

Создание таблицы с отключенным распространением маршрута BGP

```azurepowershell
$routeTableDG = New-AzRouteTable `
  -Name Firewall-rt-table `
  -ResourceGroupName Test-FW-RG `
  -location "East US" `
  -DisableBgpRoutePropagation

#Create a route
 Add-AzRouteConfig `
  -Name "DG-Route" `
  -RouteTable $routeTableDG `
  -AddressPrefix 0.0.0.0/0 `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress $AzfwPrivateIP `
 | Set-AzRouteTable

#Associate the route table to the subnet

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $testVnet `
  -Name Workload-SN `
  -AddressPrefix 10.0.2.0/24 `
  -RouteTable $routeTableDG | Set-AzVirtualNetwork
```

## <a name="configure-an-application-rule"></a>Настройка правила приложения

Правило приложения разрешает исходящий доступ к www.google.com.

```azurepowershell
$AppRule1 = New-AzFirewallApplicationRule -Name Allow-Google -SourceAddress 10.0.2.0/24 `
  -Protocol http, https -TargetFqdn www.google.com

$AppRuleCollection = New-AzFirewallApplicationRuleCollection -Name App-Coll01 `
  -Priority 200 -ActionType Allow -Rule $AppRule1

$Azfw.ApplicationRuleCollections.Add($AppRuleCollection)

Set-AzFirewall -AzureFirewall $Azfw
```

Брандмауэр Azure содержит встроенную коллекцию правил для целевых полных доменных имен инфраструктуры, которые разрешены по умолчанию. Эти доменные имена предназначены для платформы и не могут использоваться для других целей. См. дополнительные сведения об [FQDN инфраструктуры](infrastructure-fqdns.md).

## <a name="configure-a-network-rule"></a>Настройка правила сети

Правило сети разрешает исходящий доступ к двум IP-адресам через порт 53 (DNS).

```azurepowershell
$NetRule1 = New-AzFirewallNetworkRule -Name "Allow-DNS" -Protocol UDP -SourceAddress 10.0.2.0/24 `
   -DestinationAddress 209.244.0.3,209.244.0.4 -DestinationPort 53

$NetRuleCollection = New-AzFirewallNetworkRuleCollection -Name RCNet01 -Priority 200 `
   -Rule $NetRule1 -ActionType "Allow"

$Azfw.NetworkRuleCollections.Add($NetRuleCollection)

Set-AzFirewall -AzureFirewall $Azfw
```

### <a name="change-the-primary-and-secondary-dns-address-for-the-srv-work-network-interface"></a>Изменение первичного и вторичного адресов DNS для сетевого интерфейса **Srv-Work**

Для целей тестирования в этой процедуре Настройте основной и дополнительный DNS-адреса сервера. Это не является общим требованием службы "Брандмауэр Azure".

```azurepowershell
$NIC01.DnsSettings.DnsServers.Add("209.244.0.3")
$NIC01.DnsSettings.DnsServers.Add("209.244.0.4")
$NIC01 | Set-AzNetworkInterface
```

## <a name="test-the-firewall"></a>тестирование брандмауэра.

Теперь проверьте брандмауэр, чтобы убедиться, что он работает должным образом.

1. Подключитесь к виртуальной машине с **SRV-рабочими** с помощью бастиона и выполните вход. 

   :::image type="content" source="media/deploy-ps/bastion.png" alt-text="Подключение с помощью бастиона.":::

3. На **SRV-работе** откройте окно PowerShell и выполните следующие команды:

   ```
   nslookup www.google.com
   nslookup www.microsoft.com
   ```

   Обе команды должны возвращать ответы, показывая, что запросы DNS проходят через брандмауэр.

1. Выполните следующие команды:

   ```
   Invoke-WebRequest -Uri https://www.google.com
   Invoke-WebRequest -Uri https://www.google.com

   Invoke-WebRequest -Uri https://www.microsoft.com
   Invoke-WebRequest -Uri https://www.microsoft.com
   ```

   `www.google.com`Запросы должны выполняться успешно, и `www.microsoft.com` запросы должны завершаться ошибкой. Это показывает, что правила брандмауэра работают должным образом.

Итак, теперь вы убедились в том, что правила брандмауэра работают:

* Вы можете разрешать имена DNS с помощью настроенного внешнего DNS-сервера.
* Вы можете перейти только к одному разрешенному имени FQDN.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете разместить ресурсы брандмауэра для следующего руководства или, если они больше не нужны, удалить группу ресурсов **Test-FW-RG** , чтобы удалить все ресурсы, связанные с брандмауэром:

```azurepowershell
Remove-AzResourceGroup -Name Test-FW-RG
```

## <a name="next-steps"></a>Дальнейшие действия

* [Руководство. Мониторинг журналов и метрик Брандмауэра Azure](./firewall-diagnostics.md)
