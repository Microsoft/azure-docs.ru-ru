---
title: Руководство по защите виртуального концентратора с помощью Azure PowerShell
description: В этой статье описывается, как создать Виртуальную глобальную сеть Azure в одном регионе с включенным Брандмауэром Azure в концентраторе.
services: firewall-manager
author: jomore
ms.topic: tutorial
ms.service: firewall-manager
ms.date: 10/22/2020
ms.author: victorh
ms.openlocfilehash: 5ef39118d16d47aeb46294658adf616515547ea6
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105729530"
---
# <a name="tutorial-secure-your-virtual-hub-using-azure-powershell"></a>Руководство по защите виртуального концентратора с помощью Azure PowerShell

В рамках работы с этим учебником вы создадите экземпляр Виртуальной глобальной сети с виртуальным концентратором в одном регионе, а также развернете Брандмауэр Azure в концентраторе для безопасного подключения. В этом примере показано, как можно настроить безопасное подключение между виртуальными сетями. Защищенный виртуальный концентратор также поддерживает трафик между виртуальными сетями и ветвями "сеть — сеть", "точка — сеть" или ExpressRoute.

В этом руководстве вы узнаете, как:

> [!div class="checklist"]
> * развертывать виртуальную глобальную сеть;
> * развертывать Брандмауэр Azure и настраивать пользовательскую маршрутизацию.
> * Проверка подключения

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

- PowerShell 7

   Для работы с этим учебником требуется запустить Azure PowerShell в локальной среде в PowerShell 7. Дополнительные сведения об установке PowerShell 7 см. в статье [Миграция с Windows PowerShell 5.1 на PowerShell 7](/powershell/scripting/install/migrating-from-windows-powershell-51-to-powershell-7?view=powershell-7&preserve-view=true).
- Az.Network версии 3.2.0

    Если у вас есть Az.Network версии 3.4.0 или более поздней, вам потребуется понизить версию для использования некоторых команд из этого учебника. Версию модуля Az.Network можно проверить с помощью команды `Get-InstalledModule -Name Az.Network`. Чтобы удалить модуль Az.Network, запустите `Uninstall-Module -name az.network`. Чтобы установить модуль Az.Network 3.2.0, запустите `Install-Module az.network -RequiredVersion 3.2.0 -force`.

## <a name="sign-in-to-azure"></a>Вход в Azure

```azurepowershell
Connect-AzAccount
Select-AzSubscription -Subscription "<sub name>"
```

## <a name="initial-virtual-wan-deployment"></a>Начальное развертывание виртуальной глобальной сети

В качестве первого шага необходимо задать переменные и создать группу ресурсов, экземпляр виртуальной глобальной сети и виртуальный концентратор.

```azurepowershell
# Variable definition
$RG = "vwan-rg"
$Location = "westeurope"
$VwanName = "vwan"
$HubName =  "hub1"
# Create Resource Group, Virtual WAN and Virtual Hub
New-AzResourceGroup -Name $RG -Location $Location
$Vwan = New-AzVirtualWan -Name $VwanName -ResourceGroupName $RG -Location $Location -AllowVnetToVnetTraffic -AllowBranchToBranchTraffic -VirtualWANType "Standard"
$Hub = New-AzVirtualHub -Name $HubName -ResourceGroupName $RG -VirtualWan $Vwan -Location $Location -AddressPrefix "192.168.1.0/24" -Sku "Standard"
```

Создайте две виртуальные сети и подключите их к концентратору как периферийные зоны:

```azurepowershell
# Create Virtual Network
$Spoke1 = New-AzVirtualNetwork -Name "spoke1" -ResourceGroupName $RG -Location $Location -AddressPrefix "10.1.1.0/24"
$Spoke2 = New-AzVirtualNetwork -Name "spoke2" -ResourceGroupName $RG -Location $Location -AddressPrefix "10.1.2.0/24"
# Connect Virtual Network to Virtual WAN
$Spoke1Connection = New-AzVirtualHubVnetConnection -ResourceGroupName $RG -ParentResourceName  $HubName -Name "spoke1" -RemoteVirtualNetwork $Spoke1
$Spoke2Connection = New-AzVirtualHubVnetConnection -ResourceGroupName $RG -ParentResourceName  $HubName -Name "spoke2" -RemoteVirtualNetwork $Spoke2
```

На этом этапе у вас есть полнофункциональная Виртуальная глобальная сеть, обеспечивающая подключение типа "любой к любому". Чтобы улучшить безопасность, необходимо развернуть Брандмауэр Azure для каждого виртуального концентратора. Политики брандмауэра можно использовать для эффективного управления экземпляром Брандмауэра Azure виртуальной глобальной сети. Поэтому в этом примере создается политика брандмауэра.

```azurepowershell
# New Firewall Policy
$FWPolicy = New-AzFirewallPolicy -Name "VwanFwPolicy" -ResourceGroupName $RG -Location $Location
# New Firewall Public IP
$AzFWPIPs = New-AzFirewallHubPublicIpAddress -Count 1
$AzFWHubIPs = New-AzFirewallHubIpAddress -PublicIP $AzFWPIPs
# New Firewall
$AzFW = New-AzFirewall -Name "azfw1" -ResourceGroupName $RG -Location $Location `
            -VirtualHubId $Hub.Id -FirewallPolicyId $FWPolicy.Id `
            -Sku AZFW_Hub -HubIPAddress $AzFWHubIPs
```

Включение ведения журнала в Брандмауэре Azure для Azure Monitor является необязательным, но в этом примере журналы Брандмауэра используют для подтверждения того, что трафик проходит через брандмауэр:

```azurepowershell
# Optionally, enable looging of Azure Firewall to Azure Monitor
$LogWSName = "vwan-" + (Get-Random -Maximum 99999) + "-" + $RG
$LogWS = New-AzOperationalInsightsWorkspace -Location $Location -Name $LogWSName -Sku Standard -ResourceGroupName $RG
Set-AzDiagnosticSetting -ResourceId $AzFW.Id -Enabled $True -Category AzureFirewallApplicationRule, AzureFirewallNetworkRule -WorkspaceId $LogWS.ResourceId
```

## <a name="deploy-azure-firewall-and-configure-custom-routing"></a>Развертывание Брандмауэра Azure и настройка пользовательской маршрутизации

Теперь у вас есть Брандмауэр Azure в концентраторе, но по-прежнему необходимо изменить маршрутизацию, чтобы Виртуальная глобальная сеть отправляла трафик из виртуальных сетей и из ветвей через брандмауэр. Это необходимо выполнять в два этапа.

1. Настройте все подключения к виртуальной сети (и подключения ветвей, если они есть) для распространения в таблицу маршрутизации `None`. Эта конфигурация заключается в том, что другие виртуальные сети и ветви не будут знать свои префиксы и, следовательно, не смогут связаться с ними.
1. Теперь можно вставить статические маршруты в таблицу маршрутизации `Default` (где все виртуальные сети и ветви связаны по умолчанию), чтобы весь трафик отправлялся в Брандмауэр Azure.

> [!NOTE]
> Это конфигурация, развернутая при защите подключения на портале Azure с помощью Диспетчера брандмауэра Azure.

Начните с первого шага, чтобы настроить подключения виртуальной сети для распространения в таблицу маршрутизации `None`:

```azurepowershell
# Configure Virtual Network connections in hub to propagate to None
$VnetRoutingConfig = $Spoke1Connection.RoutingConfiguration    # We take $Spoke1Connection as baseline for the future vnet config, all vnets will have an identical config
$NoneRT = Get-AzVhubRouteTable -ResourceGroupName $RG -HubName $HubName -Name "noneRouteTable"
$NewPropRT = @{}
$NewPropRT.Add('Id', $NoneRT.id)
$PropRTList = @()
$PropRTList += $NewPropRT
$VnetRoutingConfig.PropagatedRouteTables.Ids = $PropRTList
$VnetRoutingConfig.PropagatedRouteTables.Labels = @()
$Spoke1Connection = Update-AzVirtualHubVnetConnection -ResourceGroupName $RG -ParentResourceName  $HubName -Name "spoke1" -RoutingConfiguration $VnetRoutingConfig
$Spoke2Connection = Update-AzVirtualHubVnetConnection -ResourceGroupName $RG -ParentResourceName  $HubName -Name "spoke2" -RoutingConfiguration $VnetRoutingConfig
```

Теперь можно продолжить второй шаг, чтобы добавить статические маршруты в таблицу маршрутизации `Default`. В этом примере применяется конфигурация по умолчанию, которая будет создаваться Диспетчером брандмауэра Azure при защите подключения в виртуальной глобальной сети, но вы можете изменить список префиксов в статическом маршруте в соответствии с вашими требованиями:

```azurepowershell
# Create static routes in default Route table
$AzFWId = $(Get-AzVirtualHub -ResourceGroupName $RG -name  $HubName).AzureFirewall.Id
$AzFWRoute = New-AzVHubRoute -Name "private-traffic" -Destination @("0.0.0.0/0", "10.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16") -DestinationType "CIDR" -NextHop $AzFWId -NextHopType "ResourceId"
$DefaultRT = Update-AzVHubRouteTable -Name "defaultRouteTable" -ResourceGroupName $RG -VirtualHubName  $HubName -Route @($AzFWRoute)
```

## <a name="test-connectivity"></a>Проверка подключения

Теперь у вас есть полностью работоспособный центр безопасности. Для проверки подключения требуется одна виртуальная машина в каждой периферийной виртуальной сети, подключенной к концентратору:

```azurepowershell
# Create VMs in spokes for testing
$VMLocalAdminUser = "lab-user"
$VMLocalAdminSecurePassword = ConvertTo-SecureString -AsPlainText -Force
$VMCredential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);
$VMSize = "Standard_B2ms"
# Spoke1
$Spoke1 = Get-AzVirtualNetwork -ResourceGroupName $RG -Name "spoke1"
Add-AzVirtualNetworkSubnetConfig -Name "vm" -VirtualNetwork $Spoke1 -AddressPrefix "10.1.1.0/26"
$Spoke1 | Set-AzVirtualNetwork
$VM1 = New-AzVM -Name "spoke1-vm" -ResourceGroupName $RG -Location $Location `
            -Image "UbuntuLTS" -credential $VMCredential `
            -VirtualNetworkName "spoke1" -SubnetName "vm" -PublicIpAddressName "spoke1-pip"
$NIC1 = Get-AzNetworkInterface -ResourceId $($VM1.NetworkProfile.NetworkInterfaces[0].Id)
$Spoke1VMPrivateIP = $NIC1.IpConfigurations[0].PrivateIpAddress
$Spoke1VMPIP = $(Get-AzPublicIpAddress -ResourceGroupName $RG -Name "spoke1-pip")
# Spoke2
$Spoke2 = Get-AzVirtualNetwork -ResourceGroupName $RG -Name "spoke2"
Add-AzVirtualNetworkSubnetConfig -Name "vm" -VirtualNetwork $Spoke2 -AddressPrefix "10.1.2.0/26"
$Spoke2 | Set-AzVirtualNetwork
$VM2 = New-AzVM -Name "spoke2-vm" -ResourceGroupName $RG -Location $Location `
            -Image "UbuntuLTS" -credential $VMCredential `
            -VirtualNetworkName "spoke2" -SubnetName "vm" -PublicIpAddressName "spoke2-pip"
$NIC2 = Get-AzNetworkInterface -ResourceId $($VM2.NetworkProfile.NetworkInterfaces[0].Id)
$Spoke2VMPrivateIP = $NIC2.IpConfigurations[0].PrivateIpAddress
$Spoke2VMPIP = $(Get-AzPublicIpAddress -ResourceGroupName $RG -Name "spoke2-pip")
```

Конфигурация по умолчанию в политике брандмауэра — это блокировка всех подключений. Поэтому необходимо настроить некоторые правила. Начните с правил DNAT, чтобы тестовые виртуальные машины были доступны через общедоступный IP-адрес Брандмауэра:

```azurepowershell
# Adding DNAT rules for virtual machines in the spokes
$AzFWPublicAddress = $AzFW.HubIPAddresses.PublicIPs.Addresses[0].Address
$NATRuleSpoke1 = New-AzFirewallPolicyNatRule -Name "Spoke1SSH" -Protocol "TCP" `
        -SourceAddress "*" -DestinationAddress $AzFWPublicAddress -DestinationPort 10001 `
        -TranslatedAddress $Spoke1VMPrivateIP -TranslatedPort 22
$NATRuleSpoke2 = New-AzFirewallPolicyNatRule -Name "Spoke2SSH" -Protocol "TCP" `
        -SourceAddress "*" -DestinationAddress $AzFWPublicAddress -DestinationPort 10002 `
        -TranslatedAddress $Spoke2VMPrivateIP -TranslatedPort 22
$NATCollection = New-AzFirewallPolicyNatRuleCollection -Name "SSH" -Priority 100 `
        -Rule @($NATRuleSpoke1, $NATRuleSpoke2) -ActionType "Dnat"
$NATGroup = New-AzFirewallPolicyRuleCollectionGroup -Name "NAT" -Priority 100 -RuleCollection $NATCollection -FirewallPolicyObject $FWPolicy
```

Теперь можно настроить некоторые примеры правил. Определите сетевое правило, разрешающее трафик SSH, а также правило приложения, которое разрешает Интернет-доступ к полному доменному имени `ifconfig.co`. Этот URL-адрес возвращает исходный IP-адрес, который он видит в HTTP-запросе:

```azurepowershell
# Add Network Rule
$SSHRule = New-AzFirewallPolicyNetworkRule -Name PermitSSH -Protocol TCP `
        -SourceAddress "10.0.0.0/8" -DestinationAddress "10.0.0.0/8" -DestinationPort 22
$NetCollection = New-AzFirewallPolicyFilterRuleCollection -Name "Management" -Priority 100 -ActionType Allow -Rule $SSHRule
$NetGroup = New-AzFirewallPolicyRuleCollectionGroup -Name "Management" -Priority 200 -RuleCollection $NetCollection -FirewallPolicyObject $FWPolicy
# Add Application Rul
$ifconfigRule = New-AzFirewallPolicyApplicationRule -Name PermitIfconfig -SourceAddress "10.0.0.0/8" -TargetFqdn "ifconfig.co" -Protocol "http:80","https:443"
$AppCollection = New-AzFirewallPolicyFilterRuleCollection -Name "TargetURLs" -Priority 300 -ActionType Allow -Rule $ifconfigRule
$NetGroup = New-AzFirewallPolicyRuleCollectionGroup -Name "TargetURLs" -Priority 300 -RuleCollection $AppCollection -FirewallPolicyObject $FWPolicy
```

Перед фактической отправкой трафика можно проверить действующие маршруты виртуальных машин. Они должны содержать префиксы, полученные из виртуальной глобальной сети (`0.0.0.0/0` и RFC1918), но не префиксы в другой периферической зоне:

```azurepowershell
# Check effective routes in the VM NIC in spoke 1
# Note that 10.1.2.0/24 (the prefix for spoke2) should not appear
Get-AzEffectiveRouteTable -ResourceGroupName $RG -NetworkInterfaceName $NIC1.Name | ft
# Check effective routes in the VM NIC in spoke 2
# Note that 10.1.1.0/24 (the prefix for spoke1) should not appear
Get-AzEffectiveRouteTable -ResourceGroupName $RG -NetworkInterfaceName $NIC2.Name | ft
```

Теперь создайте трафик из одной виртуальной машины в другую и убедитесь, что в Брандмауэре Azure прерывается подключение. В следующих командах SSH необходимо принять отпечатки для виртуальных машин и указать пароль, определенный при их создании. В рамках работы с этим примером вы отправите пять пакетов запроса проверки связи ICMP от виртуальной машины в spoke1 к spoke2, а также выполните подключение TCP через порт 22 с помощью служебной программы Linux `nc` (с флагами `-vz` просто отправляется запрос на соединение и отображается результат). Вы должны увидеть ошибку проверки связи и попытку подключения TCP через порт 22, так как это правило разрешено сетевым правилом, настроенным ранее.

```azurepowershell
# Connect to one VM and ping the other. It shouldnt work, because the firewall should drop the traffic, since no rule for ICMP is configured
ssh $AzFWPublicAddress -p 10001 -l $VMLocalAdminUser "ping $Spoke2VMPrivateIP -c 5"
# Connect to one VM and send a TCP request on port 22 to the other. It should work, because the firewall is configured to allow SSH traffic (port 22)
ssh $AzFWPublicAddress -p 10001 -l $VMLocalAdminUser "nc -vz $Spoke2VMPrivateIP 22"
```

Кроме того, можно проверить трафик Интернета. HTTP-запросы через служебную программу `curl` с полным доменным именем, разрешенным в политике брандмауэра (`ifconfig.co`), должны работать, однако HTTP-запросы к любому другому назначению работать не должны (в этом примере выполняется проверка с помощью `bing.com`):

```azurepowershell
# This HTTP request should succeed, since it is allowed in an app rule in the AzFW, and return the public IP of the FW
ssh $AzFWPublicAddress -p 10001 -l $VMLocalAdminUser "curl -s4 ifconfig.co"
# This HTTP request should fail, since the FQDN bing.com is not in any app rule in the firewall policy
ssh $AzFWPublicAddress -p 10001 -l $VMLocalAdminUser "curl -s4 bing.com"
```

Самый простой способ проверить, что пакеты блокируются брандмауэром, — это проверить журналы. Так как вы настроили Брандмауэр Azure для отправки журналов в Azure Monitor, вы можете использовать язык запросов Kusto для получения соответствующих журналов из Azure Monitor:

> [!NOTE]
> Чтобы журналы отправлялись в Azure Monitor, может потребоваться около 1 минуты.

```azurepowershell
# Getting Azure Firewall network rule Logs
$LogWS = Get-AzOperationalInsightsWorkspace -ResourceGroupName $RG
$LogQuery = 'AzureDiagnostics 
| where Category == "AzureFirewallNetworkRule" 
| where TimeGenerated >= ago(5m) 
| parse msg_s with Protocol " request from " SourceIP ":" SourcePortInt:int " to " TargetIP ":" TargetPortInt:int * 
| parse msg_s with * ". Action: " Action1a 
| parse msg_s with * " was " Action1b " to " NatDestination 
| parse msg_s with Protocol2 " request from " SourceIP2 " to " TargetIP2 ". Action: " Action2 
| extend SourcePort = tostring(SourcePortInt),TargetPort = tostring(TargetPortInt) 
| extend Action = case(Action1a == "", case(Action1b == "",Action2,Action1b), Action1a),Protocol = case(Protocol == "", Protocol2, Protocol),SourceIP = case(SourceIP == "", SourceIP2, SourceIP),TargetIP = case(TargetIP == "", TargetIP2, TargetIP),SourcePort = case(SourcePort == "", "N/A", SourcePort),TargetPort = case(TargetPort == "", "N/A", TargetPort),NatDestination = case(NatDestination == "", "N/A", NatDestination) 
| project TimeGenerated, Protocol, SourceIP,SourcePort,TargetIP,TargetPort,Action, NatDestination, Resource 
| take 25 '
$(Invoke-AzOperationalInsightsQuery -Workspace $LogWS -Query $LogQuery).Results | ft
```

В предыдущей команде вы увидите различные записи:

* SSH-подключение подвергается преобразованию сетевых адресов назначения (DNAT).
* Удалены ICMP-пакеты между виртуальными машинами в периферийных зонах (10.1.1.4 и 10.1.2.4).
* Разрешенные SSH-подключения между виртуальными машинами в периферийных зонах.

Ниже приведен пример выходных данных, созданных командой.

```
TimeGenerated            Protocol    SourceIP       SourcePort TargetIP      TargetPort Action  NatDestination Resource
-------------            --------    --------       ---------- --------      ---------- ------  -------------- --------
2020-10-04T20:53:02.41Z  TCP         109.125.122.99 62281      51.105.224.44 10001      DNAT'ed 10.1.1.4:22    AZFW1
2020-10-04T20:53:07.045Z TCP         10.1.1.4       35932      10.1.2.4      22         Allow   N/A            AZFW1
2020-10-04T20:53:50.119Z TCP         109.125.122.99 62293      51.105.224.44 10001      DNAT'ed 10.1.2.4:22    AZFW1
2020-10-04T20:52:47.475Z TCP         109.125.122.99 62273      51.105.224.44 10001      DNAT'ed 10.1.2.4:22    AZFW1
2020-10-04T20:51:04.682Z TCP         109.125.122.99 62200      51.105.224.44 10001      DNAT'ed 10.1.2.4:22    AZFW1
2020-10-04T20:51:17.031Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:51:18.049Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:51:19.075Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:51:20.097Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:51:21.121Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:52:52.356Z TCP         10.1.1.4       53748      10.1.2.4      22         Allow   N/A            AZFW1
```

Если вы хотите просмотреть правила приложений в журналах (описывающих разрешенные и запрещенные HTTP-соединения) или изменить способ отображения журналов, попробуйте использовать другие запросы KQL. Некоторые примеры можно найти в статье [Журналы Azure Monitor для Брандмауэра Azure](../firewall/firewall-workbook.md).


## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить тестовую среду, удалите группу ресурсов со всеми содержащимися в ней объектами:

```azurepowershell
# Delete resource group and all contained resources
Remove-AzResourceGroup -Name $RG
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о доверенных партнерах по безопасности](trusted-security-partners.md)