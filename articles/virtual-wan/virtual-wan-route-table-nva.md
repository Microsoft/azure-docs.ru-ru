---
title: 'Виртуальная глобальная сеть: создание таблицы маршрутов виртуального концентратора в NVA: Azure PowerShell'
description: Таблица маршрутов виртуального концентратора виртуальной глобальной сети для маршрутизации трафика к сетевому виртуальному модулю.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to work with routing tables for NVA.
ms.openlocfilehash: 40154a9c3cefed69bd3f1639153099b944da79b5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91267131"
---
# <a name="create-a-virtual-hub-route-table-to-steer-traffic-to-a-network-virtual-appliance"></a>Создание таблицы маршрутов виртуального концентратора для маршрутизации трафика к сетевому виртуальному модулю

В этой статье показано, как маршрутизировать трафик с виртуального концентратора на сетевой виртуальный модуль. 

![Схема Виртуальной глобальной сети](./media/virtual-wan-route-table-nva/vwanroute.png)

В этой статье раскрываются следующие темы:

* Создание глобальной сети.
* Создание концентратора.
* Создание виртуального концентратора сетевых подключений
* Создание маршрута концентратора
* Создание таблицы маршрутов
* Применение таблицы маршрутов

## <a name="before-you-begin"></a>Перед началом

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Убедитесь, что вы выполнили следующие критерии:

* У вас есть сетевой виртуальный модуль (NVA). Это программное обеспечение стороннего производителя, которое обычно подготавливается из Azure Marketplace в виртуальной сети.
* У вас есть частный IP-адрес, присвоенный сетевому интерфейсу виртуального сетевого модуля. 
* NVA невозможно развернуть в виртуальном концентраторе. Он должен быть развернут в отдельной виртуальной сети (VNet). В этой статье виртуальная сеть обозначена как DMZ VNet.
* К "DMZ VNet" может быть подключена одна или несколько виртуальных сетей. В этой статье такие виртуальные сети обозначены как "косвенные виртуальные сети периферийных зон". Такие виртуальные сети могут подключаться к DMZ VNet с помощью пиринговой связи между виртуальными сетями.
* Убедитесь, что у вас уже созданы две виртуальные сети. Они будут использоваться в качестве виртуальных сетей периферийных зон. В этой статье адресные пространства периферийных зон виртуальной сети — 10.0.2.0/24 и 10.0.3.0/24. Если вам нужна информация о том, как создать виртуальную сеть, см. раздел [Краткое руководство. Создание виртуальной сети с помощью PowerShell](../virtual-network/quick-create-powershell.md).
* Убедитесь, что ни в одной из виртуальных сетей нет шлюзов виртуальных сетей.

## <a name="1-sign-in"></a><a name="signin"></a>1. Вход

Убедитесь, что вы установили последнюю версию командлетов PowerShell для Resource Manager. Дополнительные сведения об установке командлетов PowerShell см. в статье [Overview of Azure PowerShell](/powershell/azure/install-az-ps) (Обзор Azure PowerShell). Это важно, так как более ранние версии командлетов не содержат текущие значения, необходимые в этом сценарии.

1. Откройте консоль PowerShell с повышенными привилегиями и войдите в свою учетную запись Azure. Этот командлет запрашивает учетные данные входа. После выполнения входа он загружает параметры учетной записи, чтобы они были доступны в Azure PowerShell.

   ```powershell
   Connect-AzAccount
   ```
2. Получите список подписок Azure.

   ```powershell
   Get-AzSubscription
   ```
3. Укажите подписку, которую нужно использовать.

   ```powershell
   Select-AzSubscription -SubscriptionName "Name of subscription"
   ```

## <a name="2-create-resources"></a><a name="rg"></a>2. Создание ресурсов

1. Создайте группу ресурсов.

   ```powershell
   New-AzResourceGroup -Location "West US" -Name "testRG"
   ```
2. Создание виртуальной глобальной сети.

   ```powershell
   $virtualWan = New-AzVirtualWan -ResourceGroupName "testRG" -Name "myVirtualWAN" -Location "West US"
   ```
3. Создание виртуального концентратора.

   ```powershell
   New-AzVirtualHub -VirtualWan $virtualWan -ResourceGroupName "testRG" -Name "westushub" -AddressPrefix "10.0.1.0/24" -Location "West US"
   ```

## <a name="3-create-connections"></a><a name="connections"></a>3. Создание подключений

Создайте подключения косвенных виртуальных сетей периферийных зон и DMZ VNet к виртуальному концентратору.

  ```powershell
  $remoteVirtualNetwork1= Get-AzVirtualNetwork -Name "indirectspoke1" -ResourceGroupName "testRG"
  $remoteVirtualNetwork2= Get-AzVirtualNetwork -Name "indirectspoke2" -ResourceGroupName "testRG"
  $remoteVirtualNetwork3= Get-AzVirtualNetwork -Name "dmzvnet" -ResourceGroupName "testRG"

  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "westushub" -Name  "testvnetconnection1" -RemoteVirtualNetwork $remoteVirtualNetwork1
  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "westushub" -Name  "testvnetconnection2" -RemoteVirtualNetwork $remoteVirtualNetwork2
  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "westushub" -Name  "testvnetconnection3" -RemoteVirtualNetwork $remoteVirtualNetwork3
  ```

## <a name="4-create-a-virtual-hub-route"></a><a name="route"></a>4. Создание маршрута виртуального концентратора

Для этой статьи адресные пространства косвенных виртуальных сетей периферийных зон: 10.0.2.0/24 и 10.0.3.0/24, а частный IP-адрес сетевого интерфейса DMZ NVA — 10.0.4.5.

```powershell
$route1 = New-AzVirtualHubRoute -AddressPrefix @("10.0.2.0/24", "10.0.3.0/24") -NextHopIpAddress "10.0.4.5"
```

## <a name="5-create-a-virtual-hub-route-table"></a><a name="applyroute"></a>5. Создание таблицы маршрутов виртуального концентратора

Создайте таблицу маршрутов виртуального концентратора, а затем примените к ней созданный маршрут.
 
```powershell
$routeTable = New-AzVirtualHubRouteTable -Route @($route1)
```

## <a name="6-commit-the-changes"></a><a name="commit"></a>6. Фиксация изменений

Зафиксируйте изменения в виртуальном концентраторе.

```powershell
Update-AzVirtualHub -ResourceGroupName "testRG" -Name "westushub" -RouteTable $routeTable
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Виртуальной глобальной сети см. в [этой статье](virtual-wan-about.md).
