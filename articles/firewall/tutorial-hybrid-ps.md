---
title: Развертывание & Настройка брандмауэра Azure в гибридной сети с помощью PowerShell
description: Из этой статьи вы узнаете, как развернуть и настроить брандмауэр Azure с помощью Azure PowerShell.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 03/26/2021
ms.author: victorh
customer intent: As an administrator, I want to control network access from an on-premises network to an Azure virtual network.
ms.openlocfilehash: db60c26ed50dae3b4b28a6c44d152a921eb96a69
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105627563"
---
# <a name="deploy-and-configure-azure-firewall-in-a-hybrid-network-using-azure-powershell"></a>Развертывание и настройка Брандмауэра Azure в гибридной сети с помощью Azure PowerShell

Возможность контролировать доступ к сетевым ресурсам Azure при подключении локальной сети к виртуальной сети Azure для создания гибридной сети является важной частью общего плана безопасности.

Брандмауэр Azure позволяет контролировать доступ к гибридной сети с помощью правил, которые определяют разрешенный и запрещенный сетевой трафик.

В этой статье вы создадите три виртуальные сети:

- **VNet-Hub** — виртуальная сеть, в которой размещен брандмауэр;
- **VNet-Spoke** — периферийная виртуальная сеть, которая представляет рабочую нагрузку, размещенную в Azure;
- **VNet-Onprem** — локальная виртуальная сеть, которая представляет локальную среду. При фактическом развертывании ее можно подключить либо через соединение VPN, либо через ExpressRoute. Для простоты в этой статье используется подключение к VPN-шлюзу, а виртуальная сеть, размещенная в Azure, используется для представления локальной сети.

![Брандмауэр в гибридной сети](media/tutorial-hybrid-ps/hybrid-network-firewall.png)

Вы узнаете, как выполнять следующие задачи:

* Объявление переменных
* Создание центральной виртуальной сети с брандмауэром
* Создание периферийной виртуальной сети
* Создание локальной виртуальной сети
* Настройка и развертывание брандмауэра
* Создание и подключение шлюзов VPN
* Настройка пиринга между центральной и периферийной виртуальными сетями
* Создание маршрутов.
* Создание виртуальных машин
* тестирование брандмауэра.

Если вместо работы с этим руководством вы хотите использовать портал Azure, см. [руководство по развертыванию и настройке брандмауэра Azure в гибридной сети с помощью портал Azure](tutorial-hybrid-portal.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные условия

Для работы с этой статьей необходимо запустить PowerShell локально. Необходимо установить модуль Azure PowerShell. Чтобы узнать версию, выполните команду `Get-Module -ListAvailable Az`. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). После проверки версии PowerShell выполните командлет `Login-AzAccount`, чтобы создать подключение к Azure.

Чтобы этот сценарий работал правильно, есть три ключевых требования:

- Определяемый пользователем маршрут (UDR) в периферийной подсети, указывающий на IP-адрес Брандмауэра Azure как шлюз по умолчанию. Распространение маршрута шлюза виртуальной сети должно быть **отключено** в этой таблице маршрутов.
- Маршрут UDR в подсети шлюза центра должен указывать на IP-адрес брандмауэра в качестве следующего прыжка к периферийным сетям.

   В подсети Брандмауэра Azure не требуется указывать UDR, так как он узнает о маршрутах из BGP.
- Не забудьте указать параметр **AllowGatewayTransit** при пиринге из VNet-Hub в VNet-Spoke и параметр **UseRemoteGateways** при пиринге из VNet-Spoke в VNet-Hub.

Сведения о создании маршрутов см. в разделе [Создание маршрутов](#create-the-routes) этой статьи.

>[!NOTE]
>Брандмауэр Azure должен быть напрямую подключен к Интернету. Если ваш Азурефиреваллсубнет узнает маршрут по умолчанию для локальной сети по протоколу BGP, необходимо настроить брандмауэр Azure в режиме принудительного туннелирования. Если это существующий брандмауэр Azure, который не может быть перенастроен в режиме принудительного туннелирования, рекомендуется добавить 0.0.0.0/0 UDR в Азурефиреваллсубнет со значением **NextHopType** , установленным в **Internet** для поддержания прямого подключения к Интернету.
>
>Дополнительные сведения см. в статье [Azure Firewall forced tunneling](forced-tunneling.md) (Принудительное туннелирование в Брандмауэре Azure).

>[!NOTE]
>Трафик между виртуальными сетями с прямым пирингом передается напрямую, даже если маршрут UDR указывает на Брандмауэр Azure как шлюз по умолчанию. Чтобы маршрутизировать трафик между подсетями к брандмауэру в этом сценарии, в UDR для обеих подсетей нужно явно указать префикс целевой подсети.

Связанную справочную документацию по Azure PowerShell см. [здесь](/powershell/module/az.network/new-azfirewall).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="declare-the-variables"></a>Объявление переменных

В следующем примере переменные объявляются с помощью значений для этой статьи. В ряде случаев для работы со своей подпиской требуется заменить некоторые значения собственными. Измените переменные (при необходимости), а затем скопируйте и вставьте их в консоль PowerShell.

```azurepowershell
$RG1 = "FW-Hybrid-Test"
$Location1 = "East US"

# Variables for the firewall hub VNet

$VNetnameHub = "VNet-hub"
$SNnameHub = "AzureFirewallSubnet"
$VNetHubPrefix = "10.5.0.0/16"
$SNHubPrefix = "10.5.0.0/24"
$SNGWHubPrefix = "10.5.1.0/24"
$GWHubName = "GW-hub"
$GWHubpipName = "VNet-hub-GW-pip"
$GWIPconfNameHub = "GW-ipconf-hub"
$ConnectionNameHub = "hub-to-Onprem"

# Variables for the spoke virtual network

$VnetNameSpoke = "VNet-Spoke"
$SNnameSpoke = "SN-Workload"
$VNetSpokePrefix = "10.6.0.0/16"
$SNSpokePrefix = "10.6.0.0/24"
$SNSpokeGWPrefix = "10.6.1.0/24"

# Variables for the on-premises virtual network

$VNetnameOnprem = "Vnet-Onprem"
$SNNameOnprem = "SN-Corp"
$VNetOnpremPrefix = "192.168.0.0/16"
$SNOnpremPrefix = "192.168.1.0/24"
$SNGWOnpremPrefix = "192.168.2.0/24"
$GWOnpremName = "GW-Onprem"
$GWIPconfNameOnprem = "GW-ipconf-Onprem"
$ConnectionNameOnprem = "Onprem-to-hub"
$GWOnprempipName = "VNet-Onprem-GW-pip"

$SNnameGW = "GatewaySubnet"
```


## <a name="create-the-firewall-hub-virtual-network"></a>Создание центральной виртуальной сети с брандмауэром

Сначала создайте группу ресурсов, которая будет содержать ресурсы для этой статьи:

```azurepowershell
  New-AzResourceGroup -Name $RG1 -Location $Location1
  ```

Определите подсети, которые будут включены в виртуальную сеть:

```azurepowershell
$FWsub = New-AzVirtualNetworkSubnetConfig -Name $SNnameHub -AddressPrefix $SNHubPrefix
$GWsub = New-AzVirtualNetworkSubnetConfig -Name $SNnameGW -AddressPrefix $SNGWHubPrefix
```

Теперь создайте центральную виртуальную сеть с брандмауэром:

```azurepowershell
$VNetHub = New-AzVirtualNetwork -Name $VNetnameHub -ResourceGroupName $RG1 `
-Location $Location1 -AddressPrefix $VNetHubPrefix -Subnet $FWsub,$GWsub
```

Запросите общедоступный IP-адрес для шлюза VPN, который будет создан для виртуальной сети. Учтите, что параметр *AllocationMethod* является **динамическим**. Указать необходимый вам IP-адрес нельзя. Он выделяется для шлюза VPN динамически.

  ```azurepowershell
  $gwpip1 = New-AzPublicIpAddress -Name $GWHubpipName -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Dynamic
```

## <a name="create-the-spoke-virtual-network"></a>Создание периферийной виртуальной сети

Определите подсети, которые будут включены в периферийную виртуальную сеть:

```azurepowershell
$Spokesub = New-AzVirtualNetworkSubnetConfig -Name $SNnameSpoke -AddressPrefix $SNSpokePrefix
$GWsubSpoke = New-AzVirtualNetworkSubnetConfig -Name $SNnameGW -AddressPrefix $SNSpokeGWPrefix
```

Создайте периферийную виртуальную сеть:

```azurepowershell
$VNetSpoke = New-AzVirtualNetwork -Name $VnetNameSpoke -ResourceGroupName $RG1 `
-Location $Location1 -AddressPrefix $VNetSpokePrefix -Subnet $Spokesub,$GWsubSpoke
```

## <a name="create-the-on-premises-virtual-network"></a>Создание локальной виртуальной сети

Определите подсети, которые будут включены в виртуальную сеть:

```azurepowershell
$Onpremsub = New-AzVirtualNetworkSubnetConfig -Name $SNNameOnprem -AddressPrefix $SNOnpremPrefix
$GWOnpremsub = New-AzVirtualNetworkSubnetConfig -Name $SNnameGW -AddressPrefix $SNGWOnpremPrefix
```

Теперь создайте локальную виртуальную сеть:

```azurepowershell
$VNetOnprem = New-AzVirtualNetwork -Name $VNetnameOnprem -ResourceGroupName $RG1 `
-Location $Location1 -AddressPrefix $VNetOnpremPrefix -Subnet $Onpremsub,$GWOnpremsub
```

Запросите общедоступный IP-адрес для шлюза, который будет создан для виртуальной сети. Учтите, что параметр *AllocationMethod* является **динамическим**. Указать необходимый вам IP-адрес нельзя. Он выделяется для шлюза динамически.

  ```azurepowershell
  $gwOnprempip = New-AzPublicIpAddress -Name $GWOnprempipName -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Dynamic
```

## <a name="configure-and-deploy-the-firewall"></a>Настройка и развертывание брандмауэра

Теперь разверните брандмауэр в центральной виртуальной сети.

```azurepowershell
# Get a Public IP for the firewall
$FWpip = New-AzPublicIpAddress -Name "fw-pip" -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Static -Sku Standard
# Create the firewall
$Azfw = New-AzFirewall -Name AzFW01 -ResourceGroupName $RG1 -Location $Location1 -VirtualNetworkName $VNetnameHub -PublicIpName fw-pip

#Save the firewall private IP address for future use

$AzfwPrivateIP = $Azfw.IpConfigurations.privateipaddress
$AzfwPrivateIP

```

### <a name="configure-network-rules"></a>Настройка правил сети

<!--- $Rule3 = New-AzFirewallNetworkRule -Name "AllowPing" -Protocol ICMP -SourceAddress $SNOnpremPrefix `
   -DestinationAddress $VNetSpokePrefix -DestinationPort *--->

```azurepowershell
$Rule1 = New-AzFirewallNetworkRule -Name "AllowWeb" -Protocol TCP -SourceAddress $SNOnpremPrefix `
   -DestinationAddress $VNetSpokePrefix -DestinationPort 80

$Rule2 = New-AzFirewallNetworkRule -Name "AllowRDP" -Protocol TCP -SourceAddress $SNOnpremPrefix `
   -DestinationAddress $VNetSpokePrefix -DestinationPort 3389

$NetRuleCollection = New-AzFirewallNetworkRuleCollection -Name RCNet01 -Priority 100 `
   -Rule $Rule1,$Rule2 -ActionType "Allow"
$Azfw.NetworkRuleCollections = $NetRuleCollection
Set-AzFirewall -AzureFirewall $Azfw
```

## <a name="create-and-connect-the-vpn-gateways"></a>Создание и подключение шлюзов VPN

Центральная и локальная виртуальные сети подключены друг к другу с помощью VPN-шлюзов.

### <a name="create-a-vpn-gateway-for-the-hub-virtual-network"></a>Создание VPN-шлюза в центральной виртуальной сети

Создайте конфигурацию VPN-шлюза. Конфигурация VPN-шлюза определяет используемые подсеть и общедоступный IP-адрес.

  ```azurepowershell
  $vnet1 = Get-AzVirtualNetwork -Name $VNetnameHub -ResourceGroupName $RG1
  $subnet1 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
  $gwipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfNameHub `
  -Subnet $subnet1 -PublicIpAddress $gwpip1
  ```

Теперь создайте VPN-шлюз в центральной виртуальной сети. Параметр VpnType для конфигураций межсетевых подключений должен иметь значение RouteBased. Создание VPN-шлюза часто занимает 45 минут и более, в зависимости от выбранного SKU VPN-шлюза.

```azurepowershell
New-AzVirtualNetworkGateway -Name $GWHubName -ResourceGroupName $RG1 `
-Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn `
-VpnType RouteBased -GatewaySku basic
```

### <a name="create-a-vpn-gateway-for-the-on-premises-virtual-network"></a>Создание VPN-шлюза для локальной виртуальной сети

Создайте конфигурацию VPN-шлюза. Конфигурация VPN-шлюза определяет используемые подсеть и общедоступный IP-адрес.

  ```azurepowershell
$vnet2 = Get-AzVirtualNetwork -Name $VNetnameOnprem -ResourceGroupName $RG1
$subnet2 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gwipconf2 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfNameOnprem `
  -Subnet $subnet2 -PublicIpAddress $gwOnprempip
  ```

Теперь создайте VPN-шлюз для локальной виртуальной сети. Параметр VpnType для конфигураций межсетевых подключений должен иметь значение RouteBased. Создание VPN-шлюза часто занимает 45 минут и более, в зависимости от выбранного SKU VPN-шлюза.

```azurepowershell
New-AzVirtualNetworkGateway -Name $GWOnpremName -ResourceGroupName $RG1 `
-Location $Location1 -IpConfigurations $gwipconf2 -GatewayType Vpn `
-VpnType RouteBased -GatewaySku basic
```

### <a name="create-the-vpn-connections"></a>Создание VPN-подключений

Теперь можно создать VPN-подключения между центральным и локальным шлюзами.

#### <a name="get-the-vpn-gateways"></a>Получение VPN-шлюзов

```azurepowershell
$vnetHubgw = Get-AzVirtualNetworkGateway -Name $GWHubName -ResourceGroupName $RG1
$vnetOnpremgw = Get-AzVirtualNetworkGateway -Name $GWOnpremName -ResourceGroupName $RG1
```

#### <a name="create-the-connections"></a>Создание подключений

На этом этапе вы создадите подключение между центральной и локальной виртуальными сетями. Вы увидите ссылки на общий ключ в примерах. Можно использовать собственные значения для общего ключа. Важно, чтобы общий ключ в обоих подключениях был одинаковым. Создание подключения может занять некоторое время.

```azurepowershell
New-AzVirtualNetworkGatewayConnection -Name $ConnectionNameHub -ResourceGroupName $RG1 `
-VirtualNetworkGateway1 $vnetHubgw -VirtualNetworkGateway2 $vnetOnpremgw -Location $Location1 `
-ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
```
Создайте подключение между локальной и центральной виртуальными сетями. Этот шаг похож на предыдущий, за исключением того, что вы создаете подключение из VNet-Onprem к VNet-hub. Убедитесь, что общие ключи совпадают. Подключение установится через несколько минут.

  ```azurepowershell
  New-AzVirtualNetworkGatewayConnection -Name $ConnectionNameOnprem -ResourceGroupName $RG1 `
  -VirtualNetworkGateway1 $vnetOnpremgw -VirtualNetworkGateway2 $vnetHubgw -Location $Location1 `
  -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
  ```

#### <a name="verify-the-connection"></a>Проверка подключения

Убедиться в успешном выполнении подключения можно с помощью командлета *Get-AzVirtualNetworkGatewayConnection* с параметром *-Debug* или без него. Используйте командлет из следующего примера, подставив свои значения. При появлении запроса выберите **A**, чтобы выполнить команду **All** (Все). В примере параметр *-Name* — это имя подключения, которое требуется проверить.

```azurepowershell
Get-AzVirtualNetworkGatewayConnection -Name $ConnectionNameHub -ResourceGroupName $RG1
```

После завершения работы командлета просмотрите результаты. В следующем примере показано, что подключение установлено (состояние *Подключено*), а также указан объем полученных и отправленных данных в байтах.

```
"connectionStatus": "Connected",
"ingressBytesTransferred": 33509044,
"egressBytesTransferred": 4142431
```

## <a name="peer-the-hub-and-spoke-virtual-networks"></a>Настройка пиринга между центральной и периферийной виртуальными сетями

Теперь создайте пиринговое подключение между центральной и периферийной виртуальными сетями.

```azurepowershell
# Peer hub to spoke
Add-AzVirtualNetworkPeering -Name HubtoSpoke -VirtualNetwork $VNetHub -RemoteVirtualNetworkId $VNetSpoke.Id -AllowGatewayTransit

# Peer spoke to hub
Add-AzVirtualNetworkPeering -Name SpoketoHub -VirtualNetwork $VNetSpoke -RemoteVirtualNetworkId $VNetHub.Id -AllowForwardedTraffic -UseRemoteGateways
```

## <a name="create-the-routes"></a>Создание маршрутов.

Создайте несколько маршрутов:

- Маршрут от подсети шлюза центра до периферийной подсети через IP-адрес брандмауэра.
- Маршрут по умолчанию из периферийной подсети через IP-адрес брандмауэра.

```azurepowershell
#Create a route table
$routeTableHubSpoke = New-AzRouteTable `
  -Name 'UDR-Hub-Spoke' `
  -ResourceGroupName $RG1 `
  -location $Location1

#Create a route
Get-AzRouteTable `
  -ResourceGroupName $RG1 `
  -Name UDR-Hub-Spoke `
  | Add-AzRouteConfig `
  -Name "ToSpoke" `
  -AddressPrefix $VNetSpokePrefix `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress $AzfwPrivateIP `
 | Set-AzRouteTable

#Associate the route table to the subnet

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNetHub `
  -Name $SNnameGW `
  -AddressPrefix $SNGWHubPrefix `
  -RouteTable $routeTableHubSpoke | `
Set-AzVirtualNetwork

#Now create the default route

#Create a table, with BGP route propagation disabled. The property is now called "Virtual network gateway route propagation," but the API still refers to the parameter as "DisableBgpRoutePropagation."
$routeTableSpokeDG = New-AzRouteTable `
  -Name 'UDR-DG' `
  -ResourceGroupName $RG1 `
  -location $Location1 `
  -DisableBgpRoutePropagation

#Create a route
Get-AzRouteTable `
  -ResourceGroupName $RG1 `
  -Name UDR-DG `
  | Add-AzRouteConfig `
  -Name "ToFirewall" `
  -AddressPrefix 0.0.0.0/0 `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress $AzfwPrivateIP `
 | Set-AzRouteTable

#Associate the route table to the subnet

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNetSpoke `
  -Name $SNnameSpoke `
  -AddressPrefix $SNSpokePrefix `
  -RouteTable $routeTableSpokeDG | `
Set-AzVirtualNetwork
```

## <a name="create-virtual-machines"></a>Создание виртуальных машин

Теперь создайте виртуальные машины для периферийной рабочей нагрузки и локальной среды и поместите их в соответствующие подсети.

### <a name="create-the-workload-virtual-machine"></a>Создание виртуальной машины с рабочей нагрузкой

В периферийной виртуальной сети создайте виртуальную машину со службами IIS без общедоступного IP-адреса и с разрешенной проверкой связи.
При появлении запроса введите имя пользователя и пароль для виртуальной машины.

```azurepowershell
# Create an inbound network security group rule for ports 3389 and 80
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name Allow-RDP  -Protocol Tcp `
  -Direction Inbound -Priority 200 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix $SNSpokePrefix -DestinationPortRange 3389 -Access Allow
$nsgRuleWeb = New-AzNetworkSecurityRuleConfig -Name Allow-web  -Protocol Tcp `
  -Direction Inbound -Priority 202 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix $SNSpokePrefix -DestinationPortRange 80 -Access Allow

# Create a network security group
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $RG1 -Location $Location1 -Name NSG-Spoke02 -SecurityRules $nsgRuleRDP,$nsgRuleWeb

#Create the NIC
$NIC = New-AzNetworkInterface -Name spoke-01 -ResourceGroupName $RG1 -Location $Location1 -SubnetId $VnetSpoke.Subnets[0].Id -NetworkSecurityGroupId $nsg.Id

#Define the virtual machine
$VirtualMachine = New-AzVMConfig -VMName VM-Spoke-01 -VMSize "Standard_DS2"
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName Spoke-01 -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2016-Datacenter' -Version latest

#Create the virtual machine
New-AzVM -ResourceGroupName $RG1 -Location $Location1 -VM $VirtualMachine -Verbose

#Install IIS on the VM
Set-AzVMExtension `
    -ResourceGroupName $RG1 `
    -ExtensionName IIS `
    -VMName VM-Spoke-01 `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server"}' `
    -Location $Location1
```

<!---#Create a host firewall rule to allow ping in
Set-AzVMExtension `
    -ResourceGroupName $RG1 `
    -ExtensionName IIS `
    -VMName VM-Spoke-01 `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4"}' `
    -Location $Location1--->

### <a name="create-the-on-premises-virtual-machine"></a>Создание локальной виртуальной машины

Это простая машина, из которой можно подключиться к удаленному рабочему столу с помощью общедоступного IP-адреса. Далее можно подключиться к локальному серверу за брандмауэром. При появлении запроса введите имя пользователя и пароль для виртуальной машины.

```azurepowershell
New-AzVm `
    -ResourceGroupName $RG1 `
    -Name "VM-Onprem" `
    -Location $Location1 `
    -VirtualNetworkName $VNetnameOnprem `
    -SubnetName $SNNameOnprem `
    -OpenPorts 3389 `
    -Size "Standard_DS2"
```

## <a name="test-the-firewall"></a>тестирование брандмауэра.

Сначала получите и запишите частный IP-адрес для виртуальной машины **VM-spoke-01**.

```azurepowershell
$NIC.IpConfigurations.privateipaddress
```

На портале Azure подключитесь к виртуальной машине **VM-Onprem**.
<!---2. Open a Windows PowerShell command prompt on **VM-Onprem**, and ping the private IP for **VM-spoke-01**.

   You should get a reply.--->
Откройте веб-браузер в **VM-Onprem** и перейдите по адресу http://\<VM-spoke-01 private IP\>.

Вы должны увидеть веб-страницу по умолчанию "Службы IIS".

На виртуальной машине **VM-Onprem** откройте удаленный рабочий стол и выполните подключение к **VM-talk-01** с помощью частного IP-адреса.

Ваше соединение должно успешно выполниться, и вы сможете войти в систему, используя свое имя пользователя и пароль.

Итак, теперь вы убедились в том, что правила брандмауэра работают:

<!---- You can ping the server on the spoke VNet.--->
- При помощи браузера можно подключиться к веб-серверу в периферийной виртуальной сети.
- К серверу в периферийной виртуальной сети можно подключиться с помощью RDP.

Затем измените действие коллекции сетевых правил брандмауэра на **Запретить**, чтобы убедиться, что правила брандмауэра работают должным образом. Выполните следующий скрипт, чтобы изменить действие коллекции правил на **Запретить**.

```azurepowershell
$rcNet = $azfw.GetNetworkRuleCollectionByName("RCNet01")
$rcNet.action.type = "Deny"

Set-AzFirewall -AzureFirewall $azfw
```

Теперь снова запустите тесты. На этот раз все они должны завершиться сбоем. Закройте имеющиеся удаленные рабочие столы перед тестированием измененных правил.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете сохранить ресурсы брандмауэра для следующего руководства или, если он больше не нужен, удалить группу ресурсов **FW-Hybrid-Test**, чтобы удалить ресурсы, связанные с брандмауэром.

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы можете отследить журналы Брандмауэра Azure.

[Руководство. Мониторинг журналов и метрик Брандмауэра Azure](./firewall-diagnostics.md)