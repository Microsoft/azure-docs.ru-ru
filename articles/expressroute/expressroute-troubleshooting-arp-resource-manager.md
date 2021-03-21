---
title: 'Azure ExpressRoute: таблицы ARP — устранение неполадок'
description: На этой странице приводятся инструкции по получению таблиц протокола ARP для канала ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: troubleshooting
ms.date: 12/15/2020
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 7d8ae2c58979c66ebbbab366d172179bdeee4253
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97561585"
---
# <a name="getting-arp-tables-in-the-resource-manager-deployment-model"></a>Получение таблиц ARP в модели развертывания с помощью Resource Manager
> [!div class="op_single_selector"]
> * [PowerShell — Resource Manager](expressroute-troubleshooting-arp-resource-manager.md)
> * [PowerShell — классическая модель](expressroute-troubleshooting-arp-classic.md)
> 
> 

В этой статье описана процедура изучения таблиц ARP для канала ExpressRoute.

> [!IMPORTANT]
> Данный документ предназначен для диагностики и устранения простых проблем. Он не заменит услуг службы поддержки Майкрософт. Если вам не удается устранить проблему, используя рекомендации, описанные ниже, то необходимо отправить запрос в [службу поддержки Майкрософт](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) .
> 
> 

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

## <a name="address-resolution-protocol-arp-and-arp-tables"></a>Протокол ARP и таблицы ARP
Протокол ARP — это протокол уровня 2, определенный в стандарте [RFC 826](https://tools.ietf.org/html/rfc826). Протокол ARP используется для сопоставления адреса Ethernet (MAC-адреса) с IP-адресом.

Таблица ARP предоставляет следующие сведения как для основного, так и для дополнительного интерфейсов для каждого типа пиринга:

1. Сопоставление локального IP-адреса интерфейса маршрутизатора с MAC-адресом.
2. Сопоставление IP-адреса интерфейса маршрутизатора ExpressRoute с MAC-адресом.
3. Длительность сопоставления.

Таблицы ARP помогают проверить конфигурацию уровня 2 и устранить проблемы с подключением на базовом уровне 2. 

Пример таблицы ARP. 

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           10.0.0.1   ffff.eeee.dddd
  0 Microsoft         10.0.0.2   aaaa.bbbb.cccc
```


В следующем разделе приведены сведения о том, как можно просмотреть таблицы ARP, видимые на граничных маршрутизаторах ExpressRoute. 

## <a name="prerequisites-for-learning-arp-tables"></a>Необходимые условия для изучения таблиц ARP
Перед выполнением дальнейших действий убедитесь, что приведенные ниже сведения имеют значение true:

* Допустимый канал ExpressRoute, настроенный по крайней мере с одним пиринг. Канал должен быть полностью настроен поставщиком услуг подключения. Вы или ваш поставщик услуг подключения должны настроить по крайней мере частный, общедоступный или пиринг Майкрософт для этого канала.
* Диапазоны IP-адресов, используемые для настройки пиринга. Ознакомьтесь с примерами назначения IP-адресов на [странице требований к маршрутизации ExpressRoute](expressroute-routing.md) , чтобы понять, как IP-адреса сопоставляются с интерфейсами. Сведения о конфигурации пиринга можно получить, просмотрев [страницу настройки пиринга ExpressRoute](expressroute-howto-routing-arm.md).
* Получена информация от вашей группы сетевых администраторов или поставщика услуг подключения о MAC-адресах интерфейсов, используемых для этих IP-адресов.
* Требуется последняя версия модуля PowerShell для Azure (1.50 или более поздняя версия).

> [!NOTE]
> Если поставщик услуг предоставил уровень 3, а таблицы ARP пусты на портале или в выходных данных, обновите конфигурацию канала с помощью кнопки Обновить на портале. Эта операция применит правильную конфигурацию маршрутизации в вашей схеме. 
>
>

## <a name="getting-the-arp-tables-for-your-expressroute-circuit"></a>Получение таблиц ARP для канала ExpressRoute
В этом разделе описано, как просмотреть таблицы ARP для пиринга с помощью PowerShell. Прежде чем продолжить, вам или вашему поставщику услуг подключения нужно настроить пиринг. Каждый канал имеет два пути (первичный и вторичный). Вы можете просмотреть эти пути в таблице ARP независимо друг от друга.

### <a name="arp-tables-for-azure-private-peering"></a>Таблицы ARP для частного пиринга Azure
Следующий командлет предоставляет таблицы ARP для частного пиринга Azure.

```azurepowershell
# Required Variables
$RG = "<Your Resource Group Name Here>"
$Name = "<Your ExpressRoute Circuit Name Here>"

# ARP table for Azure private peering - Primary path
Get-AzExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePrivatePeering -DevicePath Primary

# ARP table for Azure private peering - Secondary path
Get-AzExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePrivatePeering -DevicePath Secondary 
```

Ниже приведен пример выходных данных для одного из путей.

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           10.0.0.1   ffff.eeee.dddd
  0 Microsoft         10.0.0.2   aaaa.bbbb.cccc
```


### <a name="arp-tables-for-azure-public-peering"></a>Таблицы ARP для общедоступного пиринга Azure
Следующий командлет предоставляет таблицы ARP для общедоступного пиринга Azure.

```azurepowershell
# Required Variables
$RG = "<Your Resource Group Name Here>"
$Name = "<Your ExpressRoute Circuit Name Here>"

# ARP table for Azure public peering - Primary path
Get-AzExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePublicPeering -DevicePath Primary

# ARP table for Azure public peering - Secondary path
Get-AzExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePublicPeering -DevicePath Secondary 
```


Ниже приведен пример выходных данных для одного из путей.

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           64.0.0.1   ffff.eeee.dddd
  0 Microsoft         64.0.0.2   aaaa.bbbb.cccc
```


### <a name="arp-tables-for-microsoft-peering"></a>Таблицы ARP для пиринга Майкрософт
Следующий командлет предоставляет таблицы ARP для пиринга Майкрософт.

```azurepowershell
# Required Variables
$RG = "<Your Resource Group Name Here>"
$Name = "<Your ExpressRoute Circuit Name Here>"

# ARP table for Microsoft peering - Primary path
Get-AzExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType MicrosoftPeering -DevicePath Primary

# ARP table for Microsoft peering - Secondary path
Get-AzExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType MicrosoftPeering -DevicePath Secondary 
```


Ниже приведен пример выходных данных для одного из путей.

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           65.0.0.1   ffff.eeee.dddd
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```


## <a name="how-to-use-this-information"></a>Как использовать эти сведения
С помощью таблицы ARP для пиринга можно проверить конфигурацию и подключение уровня 2. В этом разделе описывается, как будут выглядеть таблицы ARP в различных сценариях.

### <a name="arp-table-when-a-circuit-is-in-operational-state-expected-state"></a>Таблица ARP, когда канал находится в рабочем состоянии (ожидаемое состояние)
* В таблице ARP будет размещена запись для локальной среды с допустимым IP-адресом и адресом MAC. То же самое можно увидеть на стороне Майкрософт. 
* Последний октет локального IP-адреса всегда будет нечетным числом.
* Последний октет IP-адреса сети Майкрософт всегда будет четным числом.
* Один и тот же MAC-адрес будет отображаться на стороне Майкрософт для всех трех кэширующих узлов (первичный/вторичный). 

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           65.0.0.1   ffff.eeee.dddd
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```

### <a name="arp-table-when-on-premises--connectivity-provider-side-has-problems"></a>Таблица ARP в случае проблем на стороне локальной сети или поставщика услуг подключения
При возникновении проблемы с локальным или поставщиком услуг подключения в таблице ARP будет показано одно из двух действий. Вы увидите, что локальный MAC-адрес отображается как неполный или отображается только запись Майкрософт в таблице ARP.
  
```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------   
  0 On-Prem           65.0.0.1   Incomplete
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```
или диспетчер конфигурации служб
   
```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```  

> [!NOTE]
> Отправьте запрос в службу поддержки своего поставщика услуг подключения, чтобы устранить подобные проблемы. Если в таблице ARP нет IP-адресов интерфейсов, сопоставленных с MAC-адресами, ознакомьтесь со следующими сведениями.
> 
> 1. Если первый IP-адрес подсети /30 назначен для связи между маршрутизатором MSEE-PR и MSEE используется интерфейсом MSEE-PR, Azure всегда использует второй IP-адрес для маршрутизаторов MSEE.
> 2. Убедитесь, что клиентские (C-тег) и служебные теги (S-тег) виртуальной сети соответствуют паре MSEE-PR и MSEE.
> 

### <a name="arp-table-when-microsoft-side-has-problems"></a>Таблица ARP в случае проблем на стороне сети Майкрософт
* Если на стороне Майкрософт возникли проблемы, вы не увидите таблицу ARP, отображаемую для пиринга. 
* Отправьте запрос в [службу поддержки Майкрософт](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). Укажите, что у вас проблема с подключением уровня 2. 

## <a name="next-steps"></a>Дальнейшие действия
* Проверьте конфигурации уровня 3 для канала ExpressRoute.
  * Получение сводки маршрутов для определения состояния сеансов BGP.
  * Получите таблицу маршрутов, чтобы определить, какие префиксы объявляются в ExpressRoute.
* Проверка передаваемых данных путем просмотра байт/с.
* Если у вас по-прежнему возникают проблемы, откройте запрос в службу поддержки [Майкрософт](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) .

