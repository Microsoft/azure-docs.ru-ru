---
title: Маршрутизация трафика с помощью Azure PowerShell | Документация Майкрософт
description: Из этой статьи вы узнаете, как маршрутизировать трафик с помощью таблицы маршрутов и PowerShell.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
Customer intent: I want to route traffic from one subnet, to a different subnet, through a network virtual appliance.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: kumud
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 5bd52e8865bb704497740851f6a0e3c886ed9d6d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98790213"
---
# <a name="route-network-traffic-with-a-route-table-using-powershell"></a>Маршрутизация сетевого трафика с помощью таблицы маршрутов при использовании PowerShell

По умолчанию Azure автоматически маршрутизирует трафик между всеми подсетями в виртуальной сети. Для переопределения маршрутизации Azure по умолчанию можно создать собственные маршруты. Возможность создания настраиваемых маршрутов полезна, если, к примеру, требуется маршрутизировать трафик между подсетями через виртуальный сетевой модуль. Вы узнаете, как выполнять следующие задачи:

* Создание таблицы маршрутов
* Создание маршрута
* Создание виртуальной сети с несколькими подсетями
* Связывание таблицы маршрутов с подсетью
* создание виртуального сетевого модуля, который перенаправляет трафик;
* развертывание виртуальных машин в разных подсетях;
* направление трафика из одной подсети в другую через виртуальный сетевой модуль.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально для работы с этой статьей, вам понадобится модуль Azure PowerShell 1.0.0 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-a-route-table"></a>Создание таблицы маршрутов

Прежде чем можно будет создать таблицу маршрутов, создайте группу ресурсов с помощью [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup). В следующем примере создается группа ресурсов с именем *myResourceGroup* для всех ресурсов, создаваемых в этой статье.

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

Создайте таблицу маршрутов с помощью [New-азраутетабле](/powershell/module/az.network/new-azroutetable). В следующем примере создается таблица маршрутов с именем *myRouteTablePublic*.

```azurepowershell-interactive
$routeTablePublic = New-AzRouteTable `
  -Name 'myRouteTablePublic' `
  -ResourceGroupName myResourceGroup `
  -location EastUS
```

## <a name="create-a-route"></a>Создание маршрута

Создайте маршрут, извлекая объект таблицы маршрутов с помощью [Get-азраутетабле](/powershell/module/az.network/get-azroutetable), создав маршрут с помощью [Add-азраутеконфиг](/powershell/module/az.network/add-azrouteconfig), а затем запишите конфигурацию маршрута в таблицу маршрутов с помощью [Set-азраутетабле](/powershell/module/az.network/set-azroutetable).

```azurepowershell-interactive
Get-AzRouteTable `
  -ResourceGroupName "myResourceGroup" `
  -Name "myRouteTablePublic" `
  | Add-AzRouteConfig `
  -Name "ToPrivateSubnet" `
  -AddressPrefix 10.0.1.0/24 `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress 10.0.2.4 `
 | Set-AzRouteTable
```

## <a name="associate-a-route-table-to-a-subnet"></a>Связывание таблицы маршрутов с подсетью

Чтобы связать таблицу маршрутов с подсетью, необходимо создать виртуальную сеть и подсеть. Создайте виртуальную сеть с помощью командлета [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). В следующем примере создается виртуальная сеть с именем *myVirtualNetwork* и префиксом адреса *10.0.0.0/16*.

```azurepowershell-interactive
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

Создайте три подсети, создав три конфигурации подсети с помощью [New-азвиртуалнетворксубнетконфиг](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). В следующем примере создается три конфигурации подсети: для *общей* (Public), *частной* (Private)подсети и подсети *DMZ*.

```azurepowershell-interactive
$subnetConfigPublic = Add-AzVirtualNetworkSubnetConfig `
  -Name Public `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork

$subnetConfigPrivate = Add-AzVirtualNetworkSubnetConfig `
  -Name Private `
  -AddressPrefix 10.0.1.0/24 `
  -VirtualNetwork $virtualNetwork

$subnetConfigDmz = Add-AzVirtualNetworkSubnetConfig `
  -Name DMZ `
  -AddressPrefix 10.0.2.0/24 `
  -VirtualNetwork $virtualNetwork
```

Запишите конфигурации подсети в виртуальную сеть с помощью [Set-азвиртуалнетворк](/powershell/module/az.network/Set-azVirtualNetwork), который создает подсети в виртуальной сети.

```azurepowershell-interactive
$virtualNetwork | Set-AzVirtualNetwork
```

Свяжите таблицу маршрутов *myRouteTablePublic* с *общедоступной* подсетью с помощью [Set-азвиртуалнетворксубнетконфиг](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) , а затем запишите конфигурацию подсети в виртуальную сеть с помощью [Set-азвиртуалнетворк](/powershell/module/az.network/set-azvirtualnetwork).

```azurepowershell-interactive
Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $virtualNetwork `
  -Name 'Public' `
  -AddressPrefix 10.0.0.0/24 `
  -RouteTable $myRouteTablePublic | `
Set-AzVirtualNetwork
```

## <a name="create-an-nva"></a>Создание виртуального сетевого модуля

Виртуальный сетевой модуль представляет собой виртуальную машину, которая выполняет сетевые функции, такие как маршрутизация, защита брандмауэром или оптимизация доступа к глобальной сети.

Перед созданием виртуальной машины создайте сетевой интерфейс.

### <a name="create-a-network-interface"></a>Создание сетевого интерфейса

Перед созданием сетевого интерфейса необходимо получить идентификатор виртуальной сети с помощью [Get-азвиртуалнетворк](/powershell/module/az.network/get-azvirtualnetwork), а затем идентификатор подсети с помощью [Get-азвиртуалнетворксубнетконфиг](/powershell/module/az.network/get-azvirtualnetworksubnetconfig). Создайте сетевой интерфейс с помощью [New-азнетворкинтерфаце](/powershell/module/az.network/new-aznetworkinterface) в подсети *DMZ* с включенной IP-пересылкой:

```azurepowershell-interactive
# Retrieve the virtual network object into a variable.
$virtualNetwork=Get-AzVirtualNetwork `
  -Name myVirtualNetwork `
  -ResourceGroupName myResourceGroup

# Retrieve the subnet configuration into a variable.
$subnetConfigDmz = Get-AzVirtualNetworkSubnetConfig `
  -Name DMZ `
  -VirtualNetwork $virtualNetwork

# Create the network interface.
$nic = New-AzNetworkInterface `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name 'myVmNva' `
  -SubnetId $subnetConfigDmz.Id `
  -EnableIPForwarding
```

### <a name="create-a-vm"></a>Создание виртуальной машины

Чтобы создать виртуальную машину и подключить к ней существующий сетевой интерфейс, сначала необходимо создать конфигурацию виртуальной машины с помощью [New-азвмконфиг](/powershell/module/az.compute/new-azvmconfig). Конфигурация включает в себя сетевой интерфейс, созданный на предыдущем шаге. Когда появится запрос на ввод имени пользователя и пароля, выберите те, которые будут использоваться для входа в виртуальную машину.

```azurepowershell-interactive
# Create a credential object.
$cred = Get-Credential -Message "Enter a username and password for the VM."

# Create a VM configuration.
$vmConfig = New-AzVMConfig `
  -VMName 'myVmNva' `
  -VMSize Standard_DS2 | `
  Set-AzVMOperatingSystem -Windows `
    -ComputerName 'myVmNva' `
    -Credential $cred | `
  Set-AzVMSourceImage `
    -PublisherName MicrosoftWindowsServer `
    -Offer WindowsServer `
    -Skus 2016-Datacenter `
    -Version latest | `
  Add-AzVMNetworkInterface -Id $nic.Id
```

Создайте виртуальную машину с помощью конфигурации виртуальной машины с помощью команды [New-AzVM](/powershell/module/az.compute/new-azvm). В следующем примере создается виртуальная машина с именем *myVmNva*.

```azurepowershell-interactive
$vmNva = New-AzVM `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -VM $vmConfig `
  -AsJob
```

Чтобы можно было перейти к следующему шагу, параметр `-AsJob` позволяет создать виртуальную машину в фоновом режиме.

## <a name="create-virtual-machines"></a>Создание виртуальных машин

Создайте две виртуальные машины в виртуальной сети, чтобы в дальнейшем можно было проверить, что трафик из подсети *Public* направляется в подсеть *Private* через сетевой виртуальный модуль.

Создайте виртуальную машину в подсети *Public* с помощью [New-AzVM](/powershell/module/az.compute/new-azvm). В следующем примере создается виртуальная машина с именем *myVmPublic* в подсети *Public* в виртуальной сети *myVirtualNetwork*.

```azurepowershell-interactive
New-AzVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "East US" `
  -VirtualNetworkName "myVirtualNetwork" `
  -SubnetName "Public" `
  -ImageName "Win2016Datacenter" `
  -Name "myVmPublic" `
  -AsJob
```

Создайте виртуальную машину в подсети *Private*.

```azurepowershell-interactive
New-AzVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "East US" `
  -VirtualNetworkName "myVirtualNetwork" `
  -SubnetName "Private" `
  -ImageName "Win2016Datacenter" `
  -Name "myVmPrivate"
```

Создание виртуальной машины занимает несколько минут. Не переходите к следующему шагу, пока не создастся виртуальная машина и Azure не вернет выходные данные в PowerShell.

## <a name="route-traffic-through-an-nva"></a>Перенаправление трафика через виртуальный сетевой модуль

Используйте [Get-азпублиЦипаддресс](/powershell/module/az.network/get-azpublicipaddress) , чтобы получить общедоступный IP-адрес виртуальной машины *myVmPrivate* . Приведенный ниже пример возвращает общедоступный IP-адрес виртуальной машины *myVmPrivate*:

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVmPrivate `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

Для создания сеанса удаленного рабочего стола с виртуальной машиной *myVmPrivate* на локальном компьютере выполните следующую команду. Замените `<publicIpAddress>` IP-адресом, полученным от предыдущей команды.

```
mstsc /v:<publicIpAddress>
```

Откройте загруженный RDP-файл. При появлении запроса выберите **Подключиться**.

Введите имя пользователя и пароль, которые вы указали при создании виртуальной машины (возможно, для этого придется выбрать **Дополнительные варианты** и **Использовать другую учетную запись**). Затем нажмите кнопку **ОК**. При входе в систему может появиться предупреждение о сертификате. Чтобы продолжить процесс подключения, выберите **Да**.

На более позднем этапе `tracert.exe` команда используется для проверки маршрутизации. Tracert использует протокол ICMP, который запрещен в брандмауэре Windows. Разрешите протокол ICMP в брандмауэре Windows, выполнив в интерфейсе PowerShell на виртуальной машине *myVmPrivate* следующую команду:

```powershell
New-NetFirewallRule -DisplayName "Allow ICMPv4-In" -Protocol ICMPv4
```

Хотя в этой статье и используется команда трассировки для тестирования маршрутизации, мы рекомендуем не пропускать входящий трафик по протоколу ICMP через брандмауэр Windows в рабочей среде.

Вы уже включили в Azure IP-пересылку для сетевого интерфейса виртуальной машины, выполнив инструкции из раздела "Включение IP-пересылки". Операционная система или приложение, работающие на виртуальной машине, также должны уметь перенаправлять сетевой трафик. Включите IP-пересылку в операционной системе *myVmNva*.

Из командной строки на виртуальной машине *myVmPrivate* подключитесь к виртуальной машине *myVmNva* по протоколу удаленного рабочего стола:

``` 
mstsc /v:myvmnva
```

Чтобы включить IP-пересылку в операционной системе, введите следующую команду PowerShell на виртуальной машине *myVmNva*:

```powershell
Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters -Name IpEnableRouter -Value 1
```

Перезапустите виртуальную машину *myVmNva*. При этом автоматически завершится сеанс удаленного рабочего стола.

Когда виртуальная машина *myVmNva* перезапустится, создайте сеанс удаленного рабочего стола для виртуальной машины *myVmPublic*, сохраняя подключение к виртуальной машине *myVmPrivate*:

``` 
mstsc /v:myVmPublic
```

Разрешите протокол ICMP в брандмауэре Windows с помощью следующей команды PowerShell на виртуальной машине *myVmPublic*:

```powershell
New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
```

Протестируйте маршрутизацию сетевого трафика к виртуальной машине *myVmPrivate* от виртуальной машины *myVmPublic*, выполнив следующую команду PowerShell на виртуальной машине *myVmPublic*:

```
tracert myVmPrivate
```

Ответ будет выглядеть примерно так:

```
Tracing route to myVmPrivate.vpgub4nqnocezhjgurw44dnxrc.bx.internal.cloudapp.net [10.0.1.4]
over a maximum of 30 hops:

1    <1 ms     *        1 ms  10.0.2.4
2     1 ms     1 ms     1 ms  10.0.1.4

Trace complete.
```

Можно заметить, что первым прыжком указан частный IP-адрес виртуального сетевого модуля (10.0.2.4). Второй прыжок — 10.0.1.4. Это частный IP-адрес виртуальной машины *myVmPrivate*. Из-за маршрута, добавленного в таблицу маршрутов *myRouteTablePublic* и связанного с подсетью *Public*, трафик в Azure был маршрутизирован через NVA, а не напрямую в подсеть *Private*.

Закройте сеанс удаленного рабочего стола с виртуальной машиной *myVmPublic*, сохраняя подключение к виртуальной машине *myVmPrivate*.

Протестируйте маршрутизацию сетевого трафика к виртуальной машине *myVmPublic* от виртуальной машины *myVmPrivate*, выполнив следующую команду интерфейса командной строки на виртуальной машине *myVmPrivate*:

```
tracert myVmPublic
```

Ответ будет выглядеть примерно так:

```
Tracing route to myVmPublic.vpgub4nqnocezhjgurw44dnxrc.bx.internal.cloudapp.net [10.0.0.4]
over a maximum of 30 hops:

1     1 ms     1 ms     1 ms  10.0.0.4

Trace complete.
```

Вы увидите, что трафик направляется непосредственно от виртуальной машины *myVmPrivate* на виртуальную машину *myVmPublic*. По умолчанию Azure маршрутизирует трафик между подсетями напрямую.

Закройте сеанс удаленного рабочего стола с виртуальной машиной *myVmPrivate*.

## <a name="clean-up-resources"></a>Очистка ресурсов

При необходимости удалите группу ресурсов и все содержащиеся в ней ресурсы с помощью команды [Remove-азресаурцеграуп](/powershell/module/az.resources/remove-azresourcegroup) .

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Дальнейшие действия

При работе с этой статьей вы создали таблицу маршрутов и связали ее с подсетью. Вы создали простой сетевой виртуальный модуль, который направляет трафик из общедоступной подсети в частную подсеть. [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking) предоставляет широкий ассортимент предварительно настроенных виртуальных сетевых модулей для разных сетевых функций, например модули брандмауэра и оптимизации доступа к глобальной сети. См. дополнительные сведения о [маршрутизации](virtual-networks-udr-overview.md) и [управлении таблицей маршрутов](manage-route-table.md).

Хотя в виртуальной сети можно развернуть множество ресурсов Azure, ресурсы для некоторых служб PaaS Azure развернуть невозможно. Тем не менее вы все же можете ограничить доступ к ресурсам некоторых служб PaaS в Azure трафиком только из подсети виртуальной сети. Сведения о том, как это сделать, см. в статье [Ограничение сетевого доступа к ресурсам PaaS посредством конечных точек службы виртуальной сети с помощью Azure CLI](tutorial-restrict-network-access-to-resources-powershell.md).
