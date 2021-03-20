---
title: включить файл
titleSuffix: Azure
description: включить файл
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: include
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: b2609a069872ec55ac9068fadcbb3f312d68a630
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "81680767"
---
В этом разделе описывается выполнение следующих операций изменения для прямого пиринга.

* Добавление соединений с прямым пирингом.
* Удалите прямые соединения пиринга.
* Обновление или понижение пропускной способности для активных соединений.
* Добавьте сеансы IPv4 или IPv6 на активные соединения.
* Удалите сеансы IPv4 или IPv6 на активных подключениях.

### <a name="add-direct-peering-connections"></a>Добавление соединений с прямыми пиринга

В этом примере описывается добавление подключений к существующему прямому пиринг.

```powershell

$directPeering = Get-AzPeering -Name "SeattleDirectPeering" -ResourceGroupName "PeeringResourceGroup"

$connection = New-AzPeeringDirectConnection `
    -PeeringDBFacilityId $peeringLocation.PeeringDBFacilityId `
    -SessionPrefixV4 "10.22.31.0/31" `
    -SessionPrefixV6 "fe02::3e:0/127" `
    -MaxPrefixesAdvertisedIPv4 1000 `
    -MaxPrefixesAdvertisedIPv6 100 `
    -BandwidthInMbps 10000

$directPeering.Connections.Add($connection)

$directPeering | Update-AzPeering
```

### <a name="remove-direct-peering-connections"></a>Удаление подключений прямого пиринга

Удаление подключения в настоящее время не поддерживается в PowerShell. Для получения дополнительных сведений обратитесь в службу [пиринга Майкрософт](mailto:peeringexperience@microsoft.com).

<!--
```powershell
$directPeering.Connections.Remove($directPeering.Connections[0])

$directPeering | Update-AzPeering
```
-->

### <a name="upgrade-or-downgrade-bandwidth-on-active-connections"></a>Обновление или понижение пропускной способности для активных соединений

В этом примере описывается добавление 10 Гбит/с для существующего прямого подключения.

```powershell

$directPeering = Get-AzPeering -Name "SeattleDirectPeering" -ResourceGroupName "PeeringResourceGroup"
$directPeering.Connections[0].BandwidthInMbps  = 20000
$directPeering | Update-AzPeering

```

### <a name="add-ipv4-or-ipv6-sessions-on-active-connections"></a>Добавление сеансов IPv4 или IPv6 для активных соединений

В этом примере описывается добавление сеанса IPv6 в существующее прямое соединение только с сеансом IPv4. 

```powershell

$directPeering = Get-AzPeering -Name "SeattleDirectPeering" -ResourceGroupName "PeeringResourceGroup"
$directPeering.Connections[0].BGPSession.SessionPrefixv6 = "fe01::3e:0/127"
$directPeering | Update-AzPeering

```

### <a name="remove-ipv4-or-ipv6-sessions-on-active-connections"></a>Удаление сеансов IPv4 или IPv6 на активных подключениях

Удаление сеанса IPv4 или IPv6 из существующего подключения сейчас не поддерживается в PowerShell. Для получения дополнительных сведений обратитесь в службу [пиринга Майкрософт](mailto:peeringexperience@microsoft.com).