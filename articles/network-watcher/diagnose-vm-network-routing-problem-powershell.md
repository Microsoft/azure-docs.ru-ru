---
title: Диагностика проблемы сетевой маршрутизации виртуальной машины — Azure PowerShell
titleSuffix: Azure Network Watcher
description: Из этой статьи вы узнаете, как диагностировать проблему с маршрутизацией в сети виртуальной машины с помощью возможности следующего прыжка в Наблюдателе за сетями Azure.
services: network-watcher
documentationcenter: network-watcher
author: damendo
editor: ''
tags: azure-resource-manager
Customer intent: I need to diagnose virtual machine (VM) network routing problem that prevents communication to different destinations.
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 01/07/2021
ms.author: damendo
ms.custom: ''
ms.openlocfilehash: 6569a99ec851da478151665921e7689e1c3488f1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98020219"
---
# <a name="diagnose-a-virtual-machine-network-routing-problem---azure-powershell"></a>Диагностика проблем с маршрутизацией в сети виртуальной машины с помощью Azure PowerShell

В этой статье вы развернете виртуальную машину и проверите доступ к IP-адресу и URL-адресу. Затем вы определите причину проблемы с подключением и найдете способ ее устранения.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Если вы решили установить и использовать PowerShell локально, для работы с этой статьей требуется `Az` модуль Azure PowerShell. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.



## <a name="create-a-vm"></a>Создание виртуальной машины

Прежде чем создать виртуальную машину, создайте группу ресурсов, которая будет содержать эту виртуальную машину. Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

Создайте виртуальную машину с помощью команды [New-AzVM](/powershell/module/az.compute/new-azvm). При выполнении этого шага будут запрошены учетные данные. В качестве вводимых значений указываются имя пользователя и пароль для виртуальной машины.

```azurepowershell-interactive
$vM = New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVm" `
    -Location "East US"
```

Создание виртуальной машины занимает несколько минут. Не приступайте к дальнейшим инструкциям, пока не завершится создание виртуальной машины и PowerShell не вернет выходные данные.

## <a name="test-network-communication"></a>Тестирование взаимодействия по сети

Чтобы проверить сетевое подключение с помощью Наблюдателя за сетями, сначала включите Наблюдатель за сетями в регионе, где размещается тестируемая виртуальная машина. Затем примените в Наблюдателе за сетями возможность следующего прыжка.

## <a name="enable-network-watcher"></a>Включение Наблюдателя за сетями

Если Наблюдатель за сетями уже включен в регионе "Восточная часть США", получите его экземпляр с помощью командлета [Get-AzNetworkWatcher](/powershell/module/az.network/get-aznetworkwatcher). В следующем примере извлекается существующий Наблюдатель за сетями с именем *NetworkWatcher_eastus*, размещенный в группе ресурсов *NetworkWatcherRG*:

```azurepowershell-interactive
$networkWatcher = Get-AzNetworkWatcher `
  -Name NetworkWatcher_eastus `
  -ResourceGroupName NetworkWatcherRG
```

Если Наблюдатель за сетями еще не включен в регионе "Восточная часть США", создайте его с помощью командлета [New-AzNetworkWatcher](/powershell/module/az.network/new-aznetworkwatcher):

```azurepowershell-interactive
$networkWatcher = New-AzNetworkWatcher `
  -Name "NetworkWatcher_eastus" `
  -ResourceGroupName "NetworkWatcherRG" `
  -Location "East US"
```

### <a name="use-next-hop"></a>Использование следующего прыжка

Azure автоматически создает маршруты к пунктам назначения по умолчанию. Вы можете создать настраиваемые маршруты, которые переопределяют маршруты по умолчанию. В некоторых случаях настраиваемые маршруты могут привести к сбою связи. Чтобы протестировать маршрутизацию из виртуальной машины, используйте команду [Get-азнетворкватчернекссоп](/powershell/module/az.network/get-aznetworkwatchernexthop) , чтобы определить следующий прыжок маршрутизации при передаче трафика по определенному адресу.

Проверьте исходящее подключение виртуальной машины по любому из IP-адресов сайта www.bing.com.

```azurepowershell-interactive
Get-AzNetworkWatcherNextHop `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $VM.Id `
  -SourceIPAddress 192.168.1.4 `
  -DestinationIPAddress 13.107.21.200
```

Через несколько секунд вы получите результат, где указано, что **nextHopType** имеет значение **Интернет**, а **routeTableId** — **Системный маршрут**. Из этого результата вы узнаете, что существует допустимый системный маршрут к назначению.

Проверьте исходящее подключение виртуальной машины к адресу 172.31.0.100:

```azurepowershell-interactive
Get-AzNetworkWatcherNextHop `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $VM.Id `
  -SourceIPAddress 192.168.1.4 `
  -DestinationIPAddress 172.31.0.100
```

Возвращенный результат сообщает, что **nextHopType** имеет значение **Нет**, а **routeTableId** — **Системный маршрут**. Так мы узнаем, что, хотя есть допустимый системный маршрут к месту назначения, следующий прыжок для маршрутизации трафика в пункт назначения отсутствует.

## <a name="view-details-of-a-route"></a>Просмотр сведений о маршруте

Для дальнейшего анализа маршрутизации Проверьте действующие маршруты сетевого интерфейса с помощью команды [Get-азеффективераутетабле](/powershell/module/az.network/get-azeffectiveroutetable) :

```azurepowershell-interactive
Get-AzEffectiveRouteTable `
  -NetworkInterfaceName myVm `
  -ResourceGroupName myResourceGroup |
  Format-table
```

Возвращаются выходные данные, содержащие следующий текст:

```powershell
Name State  Source  AddressPrefix           NextHopType NextHopIpAddress
---- -----  ------  -------------           ----------- ----------------
     Active Default {192.168.0.0/16}        VnetLocal   {}              
     Active Default {0.0.0.0/0}             Internet    {}              
     Active Default {10.0.0.0/8}            None        {}              
     Active Default {100.64.0.0/10}         None        {}              
     Active Default {172.16.0.0/12}         None        {}              
```

Как можно увидеть в предыдущих выходных данных, маршрут с **префиксом адреса****0.0.0.0/0** перенаправляет весь трафик, не предназначенный для адресов в пределах префиксов адресов другого маршрута, с помощью прыжка **Интернета**. Кроме того, как можно увидеть в выходных данных, хотя есть маршрут по умолчанию к префиксу 172.16.0.0/12, который включает адрес 172.31.0.100, **nextHopType** имеет значение **Нет**. Azure создает маршрут по умолчанию 172.16.0.0/12, но не указывает тип следующего прыжка, пока для этого не возникнут основания. Если, например, добавить диапазон адресов 172.16.0.0/12 в адресное пространство виртуальной сети, Azure изменит **nextHopType** для маршрута на **Виртуальная сеть**. Затем проверка отобразит **Виртуальная сеть** в качестве значения **nextHopType**.

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы с помощью командлета [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup):

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы создали виртуальную машину и выполнили диагностику сетевой маршрутизации из виртуальной машины. Вы узнали, что Azure создает несколько маршрутов по умолчанию и тестирует маршрутизацию в два разные пункта назначения. Дополнительные сведения см. в статье [Маршрутизация трафика в виртуальной сети](../virtual-network/virtual-networks-udr-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) и разделе [Создание маршрута](../virtual-network/manage-route-table.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-route).

Для исходящих подключений виртуальной машины можно определить задержку, разрешенный и запрещенный сетевой трафик между виртуальной машиной и конечной точкой с помощью функции [устранения неполадок с подключениями](network-watcher-connectivity-powershell.md) Наблюдателя за сетями. Вы можете отслеживать обмен данными между виртуальной машиной и конечной точкой, представленных IP-адресом или URL-адресом, с течением времени, используя возможность мониторинга подключения Наблюдателя за сетями. Инструкции см. в статье [Руководство. Мониторинг сетевого взаимодействия между двумя виртуальными машинами с помощью портала Azure](connection-monitor.md).
