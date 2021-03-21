---
title: Фильтрация сетевого трафика с помощью Azure PowerShell | Документация Майкрософт
description: В этой статье вы узнаете, как фильтровать сетевой трафик в подсети с помощью группы безопасности сети, используя PowerShell.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
Customer intent: I want to filter network traffic to virtual machines that perform similar functions, such as web servers.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/30/2018
ms.author: kumud
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 8ab22e7960e233d6ae934fb52989aa73a494b33a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98791516"
---
# <a name="filter-network-traffic-with-a-network-security-group-using-powershell"></a>Фильтрация сетевого трафика с помощью группы безопасности сети посредством PowerShell

Вы можете отфильтровать входящий и исходящий трафик в подсети виртуальной сети с помощью группы безопасности сети. Группы безопасности сети содержат правила безопасности, которые фильтруют трафик по IP-адресу, порту и протоколу. Правила безопасности применяются к ресурсам, развернутым в подсети. Вы узнаете, как выполнять следующие задачи:

* Создание группы безопасности сети и правил безопасности.
* Создание виртуальной сети и привязка группы безопасности сети к подсети.
* Развертывание виртуальных машин в подсеть.
* Тестирование фильтров трафика

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально для работы с этой статьей, вам понадобится модуль Azure PowerShell 1.0.0 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-a-network-security-group"></a>Создание группы безопасности сети

Группа безопасности сети содержит правила безопасности. Правила безопасности указывают источник и назначение. Источники и назначения могут быть группами безопасности приложений.

### <a name="create-application-security-groups"></a>Создание групп безопасности приложений

Сначала создайте группу ресурсов для всех ресурсов, созданных в этой статье, с помощью [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup). В следующем примере создается группа ресурсов в расположении *eastus*.

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

Создайте группу безопасности приложений с помощью [New-азаппликатионсекуритиграуп](/powershell/module/az.network/new-azapplicationsecuritygroup). Группа безопасности приложений позволяет группировать серверы с аналогичными требованиями к фильтрации портов. В следующем примере создаются две группы безопасности приложений.

```azurepowershell-interactive
$webAsg = New-AzApplicationSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Name myAsgWebServers `
  -Location eastus

$mgmtAsg = New-AzApplicationSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Name myAsgMgmtServers `
  -Location eastus
```

### <a name="create-security-rules"></a>Создание правил безопасности

Создайте правило безопасности с помощью [New-азнетворксекуритирулеконфиг](/powershell/module/az.network/new-aznetworksecurityruleconfig). В следующем примере создается правило, разрешающее входящий поток трафика из Интернета в группу безопасности приложений *myWebServers* через порты 80 и 443:

```azurepowershell-interactive
$webRule = New-AzNetworkSecurityRuleConfig `
  -Name "Allow-Web-All" `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 100 `
  -SourceAddressPrefix Internet `
  -SourcePortRange * `
  -DestinationApplicationSecurityGroupId $webAsg.id `
  -DestinationPortRange 80,443

The following example creates a rule that allows traffic inbound from the internet to the *myMgmtServers* application security group over port 3389:

$mgmtRule = New-AzNetworkSecurityRuleConfig `
  -Name "Allow-RDP-All" `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 110 `
  -SourceAddressPrefix Internet `
  -SourcePortRange * `
  -DestinationApplicationSecurityGroupId $mgmtAsg.id `
  -DestinationPortRange 3389
```

В этой статье RDP-подключение (порт 3389) доступно в Интернете для виртуальной машины *myAsgMgmtServers*. Для рабочих сред вместо доступа к порту 3389 через Интернет рекомендуется подключаться к ресурсам Azure, которыми вы хотите управлять с помощью подключения [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json) или [частной](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json) сети.

### <a name="create-a-network-security-group"></a>Создание группы безопасности сети

Создайте группу безопасности сети с помощью командлета [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). В следующем примере создается группа безопасности сети с именем *myNsg*:

```powershell-interactive
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -Name myNsg `
  -SecurityRules $webRule,$mgmtRule
```

## <a name="create-a-virtual-network"></a>Создание виртуальной сети

Создайте виртуальную сеть с помощью командлета [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). В следующем примере создается виртуальная сеть с именем *myVirtualNetwork*:

```azurepowershell-interactive
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

Создайте конфигурацию подсети с помощью [New-азвиртуалнетворксубнетконфиг](/powershell/module/az.network/new-azvirtualnetworksubnetconfig), а затем запишите конфигурацию подсети в виртуальную сеть с помощью [Set-азвиртуалнетворк](/powershell/module/az.network/set-azvirtualnetwork). В следующем примере добавляется подсеть с именем *mySubnet* в виртуальную сеть и ей назначается группа безопасности сети *myNsg*:

```azurepowershell-interactive
Add-AzVirtualNetworkSubnetConfig `
  -Name mySubnet `
  -VirtualNetwork $virtualNetwork `
  -AddressPrefix "10.0.2.0/24" `
  -NetworkSecurityGroup $nsg
$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="create-virtual-machines"></a>Создание виртуальных машин

Перед созданием виртуальных машин извлеките объект виртуальной сети с подсетью с помощью [Get-азвиртуалнетворк](/powershell/module/az.network/get-azvirtualnetwork):

```powershell-interactive
$virtualNetwork = Get-AzVirtualNetwork `
 -Name myVirtualNetwork `
 -Resourcegroupname myResourceGroup
```

Создайте общедоступный IP-адрес для каждой виртуальной машины с помощью [New-азпублиЦипаддресс](/powershell/module/az.network/new-azpublicipaddress):

```powershell-interactive
$publicIpWeb = New-AzPublicIpAddress `
  -AllocationMethod Dynamic `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -Name myVmWeb

$publicIpMgmt = New-AzPublicIpAddress `
  -AllocationMethod Dynamic `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -Name myVmMgmt
```

Создайте два сетевых интерфейса с помощью [New-азнетворкинтерфаце](/powershell/module/az.network/new-aznetworkinterface)и назначьте сетевому интерфейсу общедоступный IP-адрес. В следующем примере создается сетевой интерфейс, которому назначается общедоступный IP-адрес *myVmWeb*. Этот сетевой интерфейс добавляется в группу безопасности приложений *myAsgWebServers*.

```powershell-interactive
$webNic = New-AzNetworkInterface `
  -Location eastus `
  -Name myVmWeb `
  -ResourceGroupName myResourceGroup `
  -SubnetId $virtualNetwork.Subnets[0].Id `
  -ApplicationSecurityGroupId $webAsg.Id `
  -PublicIpAddressId $publicIpWeb.Id
```

В следующем примере создается сетевой интерфейс, которому назначается общедоступный IP-адрес *myVmMgmt*. Этот сетевой интерфейс добавляется в группу безопасности приложений *myAsgMgmtServers*.

```powershell-interactive
$mgmtNic = New-AzNetworkInterface `
  -Location eastus `
  -Name myVmMgmt `
  -ResourceGroupName myResourceGroup `
  -SubnetId $virtualNetwork.Subnets[0].Id `
  -ApplicationSecurityGroupId $mgmtAsg.Id `
  -PublicIpAddressId $publicIpMgmt.Id
```

Создайте две виртуальные машины в виртуальной сети, чтобы вы могли проверить фильтрацию трафика на более позднем этапе.

Создайте конфигурацию виртуальной машины с помощью [New-азвмконфиг](/powershell/module/az.compute/new-azvmconfig), а затем создайте виртуальную машину с помощью [New-AzVM](/powershell/module/az.compute/new-azvm). В следующем примере создается виртуальная машина, которая будет служить веб-сервером. Чтобы можно было перейти к следующему шагу, параметр `-AsJob` позволяет создать виртуальную машину в фоновом режиме.

```azurepowershell-interactive
# Create user object
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."

$webVmConfig = New-AzVMConfig `
  -VMName myVmWeb `
  -VMSize Standard_DS1_V2 | `
Set-AzVMOperatingSystem -Windows `
  -ComputerName myVmWeb `
  -Credential $cred | `
Set-AzVMSourceImage `
  -PublisherName MicrosoftWindowsServer `
  -Offer WindowsServer `
  -Skus 2016-Datacenter `
  -Version latest | `
Add-AzVMNetworkInterface `
  -Id $webNic.Id
New-AzVM `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -VM $webVmConfig `
  -AsJob
```

Создайте виртуальную машину в качестве сервера управления:

```azurepowershell-interactive
# Create user object
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."

# Create the web server virtual machine configuration and virtual machine.
$mgmtVmConfig = New-AzVMConfig `
  -VMName myVmMgmt `
  -VMSize Standard_DS1_V2 | `
Set-AzVMOperatingSystem -Windows `
  -ComputerName myVmMgmt `
  -Credential $cred | `
Set-AzVMSourceImage `
  -PublisherName MicrosoftWindowsServer `
  -Offer WindowsServer `
  -Skus 2016-Datacenter `
  -Version latest | `
Add-AzVMNetworkInterface `
  -Id $mgmtNic.Id
New-AzVM `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -VM $mgmtVmConfig
```

Создание виртуальной машины занимает несколько минут. Не переходите к следующему шагу, пока Azure не закончит создание виртуальной машины.

## <a name="test-traffic-filters"></a>Тестирование фильтров трафика

Чтобы получить общедоступный IP-адрес виртуальной машины, выполните командлет [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress). Приведенный ниже пример возвращает общедоступный IP-адрес виртуальной машины *myVmMgmt*.

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVmMgmt `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

На локальном компьютере выполните следующую команду для создания сеанса удаленного рабочего стола с виртуальной машиной *myVmMgmt*. Замените `<publicIpAddress>` IP-адресом, полученным от предыдущей команды.

```
mstsc /v:<publicIpAddress>
```

Откройте загруженный RDP-файл. При появлении запроса выберите **Подключиться**.

Введите имя пользователя и пароль, которые вы указали при создании виртуальной машины (возможно, для этого придется выбрать **Дополнительные варианты** и **Использовать другую учетную запись**). Затем нажмите кнопку **ОК**. При входе в систему может появиться предупреждение о сертификате. Чтобы продолжить процесс подключения, выберите **Да**.

Подключение будет установлено успешно, так как через порт 3389 разрешено получать входящий трафик из Интернета в группу безопасности приложений *myAsgMgmtServers*, в которой находится сетевой интерфейс, подключенный к виртуальной машине *myVmMgmt*.

Используйте следующую команду PowerShell, чтобы создать подключение к удаленному рабочему столу виртуальной машины *myVmWeb* из виртуальной машины *myVmMgmt*.

``` 
mstsc /v:myvmWeb
```

Подключение будет установлено успешно, так как правило безопасности по умолчанию в каждой группе безопасности сети разрешает трафик через все порты между всеми IP-адресами в виртуальной сети. Невозможно установить подключение к удаленному рабочему столу виртуальной машины *myVmWeb* из Интернета, потому что правило безопасности для *myAsgWebServers* не разрешает получение входящего трафика через порт 3389 из Интернета.

Используйте следующую команду PowerShell для установки Microsoft IIS на виртуальной машине *myVmWeb*.

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

Когда установка служб IIS завершится, отключитесь от виртуальной машины *myVmWeb*, после чего вы останетесь в сеансе удаленного рабочего стола виртуальной машины *myVmMgmt*. Чтобы просмотреть экран приветствия IIS, откройте Интернет-браузер и перейдите по адресу http: \/ /мивмвеб.

Отключитесь от виртуальной машины *myVmMgmt* .

На своем компьютере введите приведенную ниже команду PowerShell, чтобы получить общедоступный IP-адрес сервера *myVmWeb*.

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVmWeb `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

Чтобы убедиться в наличии доступа к веб-серверу *myVmWeb* извне Azure, откройте веб-браузер на своем компьютере и перейдите по адресу `http://<public-ip-address-from-previous-step>`. Подключение будет установлено успешно, так как через порт 80 разрешено получение входящего трафика из Интернета в группу безопасности приложений *myAsgWebServers*, в которой находится сетевой интерфейс, подключенный к виртуальной машине *myVmWeb*.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы с помощью командлета [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup):

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы создали группу безопасности сети и связали ее с подсетью виртуальной сети. Дополнительные сведения о группах безопасности сети см. в статьях [Безопасность сети](./network-security-groups-overview.md) и [Create, change, or delete a network security group](manage-network-security-group.md) (Создание, изменение или удаление группы безопасности сети).

Azure маршрутизирует трафик между подсетями по умолчанию. Вместо этого вы можете перенаправлять трафик между подсетями через виртуальную машину, которая используется в качестве брандмауэра. Дополнительные сведения см. в статье [Маршрутизация сетевого трафика с помощью таблицы маршрутов с использованием портала Azure](tutorial-create-route-table-powershell.md).