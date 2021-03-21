---
title: Подключение виртуальных сетей с помощью пиринга виртуальной сети — Azure PowerShell
description: В этой статье вы узнаете, как подключить виртуальные сети между собой с помощью пиринга, используя Azure PowerShell.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
tags: azure-resource-manager
Customer intent: I want to connect two virtual networks so that virtual machines in one virtual network can communicate with virtual machines in the other virtual network.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: kumud
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d93efaedbb0596382b0601a17098311e075618b7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98791986"
---
# <a name="connect-virtual-networks-with-virtual-network-peering-using-powershell"></a>Установка пирингового подключения между виртуальными сетями с помощью PowerShell

Виртуальные сети можно подключить друг к другу с помощью пиринговой связи. После установки пиринговой связи ресурсы в обеих виртуальных сетях могут взаимодействовать друг с другом с такой же задержкой и пропускной способностью, как если бы эти ресурсы находились в одной виртуальной сети. Вы узнаете, как выполнять следующие задачи:

* создание двух виртуальных сетей;
* соединение двух виртуальных сетей с помощью пиринговой связи;
* развертывание виртуальной машины в каждой из виртуальных сетей;
* Взаимодействие между виртуальными машинами

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально для работы с этой статьей, вам понадобится модуль Azure PowerShell 1.0.0 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-virtual-networks"></a>Создание виртуальных сетей

Перед созданием виртуальной сети необходимо создать для нее группу ресурсов и другие компоненты, указанные в этой статье. Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

Создайте виртуальную сеть с помощью командлета [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). В следующем примере создается виртуальная сеть с именем *myVirtualNetwork1* и префиксом адреса *10.0.0.0/16*.

```azurepowershell-interactive
$virtualNetwork1 = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork1 `
  -AddressPrefix 10.0.0.0/16
```

Создайте конфигурацию подсети с помощью [New-азвиртуалнетворксубнетконфиг](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). В следующем примере создается конфигурация подсети с префиксом адреса 10.0.0.0/24:

```azurepowershell-interactive
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
  -Name Subnet1 `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork1
```

Запишите конфигурацию подсети в виртуальную сеть с помощью [Set-азвиртуалнетворк](/powershell/module/az.network/Set-azVirtualNetwork), которая создает подсеть.

```azurepowershell-interactive
$virtualNetwork1 | Set-AzVirtualNetwork
```

Создайте виртуальную сеть с префиксом адреса 10.1.0.0/16 и одной подсетью:

```azurepowershell-interactive
# Create the virtual network.
$virtualNetwork2 = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork2 `
  -AddressPrefix 10.1.0.0/16

# Create the subnet configuration.
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
  -Name Subnet1 `
  -AddressPrefix 10.1.0.0/24 `
  -VirtualNetwork $virtualNetwork2

# Write the subnet configuration to the virtual network.
$virtualNetwork2 | Set-AzVirtualNetwork
```

## <a name="peer-virtual-networks"></a>Установка пиринговой связи между виртуальными сетями

Создайте пиринг с помощью [Add-азвиртуалнетворкпиринг](/powershell/module/az.network/add-azvirtualnetworkpeering). В следующем примере создается пиринговая связь между сетями *myVirtualNetwork1* и *myVirtualNetwork2*.

```azurepowershell-interactive
Add-AzVirtualNetworkPeering `
  -Name myVirtualNetwork1-myVirtualNetwork2 `
  -VirtualNetwork $virtualNetwork1 `
  -RemoteVirtualNetworkId $virtualNetwork2.Id
```

В выходных данных, возвращенных после выполнения предыдущей команды, для **состояния пиринга** будет отображаться значение *Инициировано*. Состояние *Инициировано* не изменится, пока не будет создан пиринг сети *myVirtualNetwork2* с сетью *myVirtualNetwork1*. Установите пиринговую связь сети *myVirtualNetwork2* с сетью *myVirtualNetwork1*.

```azurepowershell-interactive
Add-AzVirtualNetworkPeering `
  -Name myVirtualNetwork2-myVirtualNetwork1 `
  -VirtualNetwork $virtualNetwork2 `
  -RemoteVirtualNetworkId $virtualNetwork1.Id
```

В выходных данных, возвращенных после выполнения предыдущей команды, для **состояния пиринга** будет отображаться значение *Подключено*. Одновременно с этим Azure установит значение *Подключено* и для первого пиринга *myVirtualNetwork1-myVirtualNetwork2*. Убедитесь, что состояние пиринга для пиринга *myVirtualNetwork1-myVirtualNetwork2* изменилось на " *подключено* " с помощью [Get-азвиртуалнетворкпиринг](/powershell/module/az.network/get-azvirtualnetworkpeering).

```azurepowershell-interactive
Get-AzVirtualNetworkPeering `
  -ResourceGroupName myResourceGroup `
  -VirtualNetworkName myVirtualNetwork1 `
  | Select PeeringState
```

Ресурсы одной виртуальной сети не могут взаимодействовать с ресурсами другой виртуальной сети, пока **состояние пиринга** для обеих виртуальных сетей не будет иметь значение *Подключено*.

## <a name="create-virtual-machines"></a>Создание виртуальных машин

Создайте виртуальную машину в каждой виртуальной сети, чтобы на следующем шаге организовать взаимодействие между такими машинами.

### <a name="create-the-first-vm"></a>Создание первой виртуальной машины

Создайте виртуальную машину с помощью команды [New-AzVM](/powershell/module/az.compute/new-azvm). В приведенном ниже примере в виртуальной сети *myVirtualNetwork1* создается виртуальная машина с именем *myVm1*. Чтобы можно было перейти к следующему шагу, параметр `-AsJob` позволяет создать виртуальную машину в фоновом режиме. Когда появится запрос, введите имя пользователя и пароль, которые вы будете использовать для входа на виртуальную машину.

```azurepowershell-interactive
New-AzVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "East US" `
  -VirtualNetworkName "myVirtualNetwork1" `
  -SubnetName "Subnet1" `
  -ImageName "Win2016Datacenter" `
  -Name "myVm1" `
  -AsJob
```

### <a name="create-the-second-vm"></a>Создание второй виртуальной машины

```azurepowershell-interactive
New-AzVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "East US" `
  -VirtualNetworkName "myVirtualNetwork2" `
  -SubnetName "Subnet1" `
  -ImageName "Win2016Datacenter" `
  -Name "myVm2"
```

Создание виртуальной машины занимает несколько минут. Не выполняйте следующие шаги, пока в Azure не будет создана виртуальная машина и не будут возвращены выходные данные в PowerShell.

## <a name="communicate-between-vms"></a>Взаимодействие между виртуальными машинами

Вы можете подключиться к общедоступному IP-адресу виртуальной машины через Интернет. Чтобы получить общедоступный IP-адрес виртуальной машины, выполните командлет [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress). Приведенный ниже пример возвращает общедоступный IP-адрес виртуальной машины *myVm1*:

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVm1 `
  -ResourceGroupName myResourceGroup | Select IpAddress
```

На локальном компьютере выполните следующую команду для создания сеанса удаленного рабочего стола с виртуальной машиной *myVm1*. Замените `<publicIpAddress>` IP-адресом, полученным от предыдущей команды.

```
mstsc /v:<publicIpAddress>
```

Будет создан файл протокола удаленного рабочего стола (RDP-файл). Он будет скачан на ваш компьютер и открыт. Введите имя пользователя и пароль, а затем нажмите кнопку **ОК**. Возможно, понадобится выбрать **More choices** (Дополнительные варианты) и **Use a different account** (Использовать другую учетную запись), чтобы указать учетные данные, введенные при создании виртуальной машины. При входе в систему может появиться предупреждение о сертификате. Чтобы продолжить процесс подключения, нажмите кнопку **Да** или **Продолжить**.

На виртуальной машине *myVm1* с помощью PowerShell разрешите протокол ICMP в брандмауэре Windows, чтобы на следующем шаге нормально работала проверка связи с виртуальной машины *myVm2*:

```powershell
New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
```

Хотя проверка связи используется в этой статье для взаимодействия между виртуальными машинами, мы не рекомендуем разрешать протокол ICMP в брандмауэре Windows в рабочей среде.

Для подключения к виртуальной машине *myVm2* введите следующую команду в командной строке на виртуальной машине *myVm1*:

```
mstsc /v:10.1.0.4
```

Так как вы включили проверку связи на виртуальной машине *myVm1*, теперь можно проверить связь с ней с помощью IP-адреса из командной строки на виртуальной машине *myVm2*:

```
ping 10.0.0.4
```

Вы получите четыре ответа. Отключите RDP-сеансы с *myVm1* и *myVm2*.

## <a name="clean-up-resources"></a>Очистка ресурсов

При необходимости удалите группу ресурсов и все содержащиеся в ней ресурсы с помощью команды [Remove-азресаурцеграуп](/powershell/module/az.resources/remove-azresourcegroup) .

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали, как с помощью пиринга соединить две виртуальные сети в одном регионе Azure. Пиринг можно использовать и для подключения виртуальных сетей из разных [поддерживаемых регионов](virtual-network-manage-peering.md#cross-region) или [разных подписок Azure](create-peering-different-subscriptions.md#powershell), а также для создания [звездообразной топологии сети](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke#virtual-network-peering). Чтобы узнать больше о пиринге виртуальных сетей, ознакомьтесь с разделами [Пиринг между виртуальными сетями](virtual-network-peering-overview.md) и [Создание, изменение и удаление пиринга в виртуальной сети](virtual-network-manage-peering.md).

Теперь вы можете [подключить свой компьютер к виртуальной сети](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json) через VPN и взаимодействовать с ресурсами в виртуальной сети или в виртуальных сетях, соединенных пиринговой связью. Примеры сценариев для многократного использования, с помощью которых можно выполнять многие задачи, описанные в статьях о виртуальных сетях, можно найти [здесь](powershell-samples.md).
