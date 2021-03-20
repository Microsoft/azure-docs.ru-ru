---
title: 'Azure ExpressRoute. Устранение неполадок в таблицах ARP: классическая модель'
description: Инструкции по получению таблиц ARP для канала ExpressRoute в классической модели развертывания.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: troubleshooting
ms.date: 01/30/2017
ms.author: duau
ms.openlocfilehash: 7c83b6d25936e184c70d4bf0a51636a3986bf4b2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89394845"
---
# <a name="getting-arp-tables-in-the-classic-deployment-model"></a>Получение таблиц ARP в классической модели развертывания
> [!div class="op_single_selector"]
> * [PowerShell — Resource Manager](expressroute-troubleshooting-arp-resource-manager.md)
> * [PowerShell — классическая модель](expressroute-troubleshooting-arp-classic.md)
> 
> 

В этой статье представлены пошаговые указания по получению таблиц протокола ARP для канала Azure ExpressRoute.

> [!IMPORTANT]
> Данный документ предназначен для диагностики и устранения простых проблем. Он не заменит услуг службы поддержки Майкрософт. Если проблему не удается устранить с помощью приведенных указаний, отправьте запрос на поддержку с помощью функции [Справка и поддержка Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
> 
> 

## <a name="address-resolution-protocol-arp-and-arp-tables"></a>Протокол ARP и таблицы ARP
ARP — это протокол уровня 2, который определен в [RFC 826](https://tools.ietf.org/html/rfc826). Протокол ARP используется для сопоставления адреса Ethernet (MAC-адреса) с IP-адресом.

Таблица ARP обеспечивает сопоставление IPv4-адреса и MAC-адреса для конкретного пиринга. Таблица ARP для пиринга канала ExpressRoute содержит следующие сведения о каждом интерфейсе (первичном и вторичном).

1. Сопоставление IP-адреса локального интерфейса маршрутизатора с MAC-адресом.
2. Сопоставление IP-адреса интерфейса ExpressRoute маршрутизатора с MAC-адресом.
3. Длительность сопоставления.

Таблицы ARP помогают проверять конфигурацию уровня 2 и устранять проблемы с подключением на базовом уровне 2.

Ниже приведен пример таблицы ARP.

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           10.0.0.1   ffff.eeee.dddd
  0 Microsoft         10.0.0.2   aaaa.bbbb.cccc
```


В следующем разделе приведены сведения о том, как просмотреть таблицы ARP, видимые на пограничных маршрутизаторах ExpressRoute.

## <a name="prerequisites-for-using-arp-tables"></a>Необходимые условия для использования таблиц ARP
Прежде чем продолжить, убедитесь, что выполнены следующие условия.

* Настроен действительный канал ExpressRoute по крайней мере с одним пирингом. Канал должен быть полностью настроен поставщиком услуг подключения. Вы (или ваш поставщик услуг подключения) должны настроить хотя бы один из пирингов (частный или общедоступный пиринг Azure либо пиринг Майкрософт) для этого канала.
* Настроены диапазоны IP-адресов, используемые для настройки пирингов (частных или общедоступных пирингов Azure либо пирингов Майкрософт). Ознакомьтесь с примерами назначения IP-адресов на [странице требований к маршрутизации expressroute](expressroute-routing.md) , чтобы получить представление о том, как IP-адреса сопоставлены с интерфейсами на стороне пользователя и с помощью expressroute. Сведения о конфигурации пиринга можно получить, просмотрев [страницу настройки пиринга ExpressRoute](expressroute-howto-routing-classic.md).
* От вашей группы сетевых администраторов или поставщика услуг подключения получена информация о MAC-адресах интерфейсов, используемых для этих IP-адресов.
* Последняя версия модуля PowerShell для Azure (1.50 или более поздняя версия).

## <a name="arp-tables-for-your-expressroute-circuit"></a>Таблицы ARP для канала ExpressRoute
Этот раздел содержит инструкции о том, как просмотреть таблицы ARP для каждого типа пиринга с помощью PowerShell. Прежде чем продолжить, вам или вашему поставщику услуг подключения необходимо настроить пиринг. Каждый канал имеет два пути (первичный и вторичный). Вы можете просмотреть эти пути в таблице ARP независимо друг от друга.

### <a name="arp-tables-for-azure-private-peering"></a>Таблицы ARP для частного пиринга Azure
Следующий командлет предоставляет таблицы ARP для частного пиринга Azure.

```azurepowershell
# Required variables
$ckt = "<your Service Key here>

# ARP table for Azure private peering--primary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Private -Path Primary

# ARP table for Azure private peering--secondary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Private -Path Secondary
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
# Required variables
$ckt = "<your Service Key here>

# ARP table for Azure public peering--primary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Public -Path Primary

# ARP table for Azure public peering--secondary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Public -Path Secondary
```

Ниже приведен пример выходных данных для одного из путей.

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           10.0.0.1   ffff.eeee.dddd
  0 Microsoft         10.0.0.2   aaaa.bbbb.cccc
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
# ARP table for Microsoft peering--primary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Microsoft -Path Primary

# ARP table for Microsoft peering--secondary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Microsoft -Path Secondary
```


Ниже приведен пример выходных данных для одного из путей.

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           65.0.0.1   ffff.eeee.dddd
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```


## <a name="how-to-use-this-information"></a>Как использовать эти сведения
С помощью таблицы ARP для пиринга можно проверить конфигурацию и возможности подключения уровня 2. В этом разделе описывается, как выглядят таблицы ARP в различных сценариях.

### <a name="arp-table-when-a-circuit-is-in-an-operational-expected-state"></a>Таблица ARP, когда канал находится в рабочем состоянии (ожидаемое состояние)
* В таблице ARP отображается запись для локальной стороны с действительным IP-адресом и MAC-адресом, а также аналогичная запись со стороны сети Майкрософт.
* Последний октет локального IP-адреса всегда является нечетным числом.
* Последний октет локального IP-адреса сети Майкрософт всегда является четным числом.
* Одинаковый MAC-адрес отображается на стороне сети Майкрософт для всех 3 пирингов (первичных или вторичных).

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           65.0.0.1   ffff.eeee.dddd
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```

### <a name="arp-table-when-its-on-premises-or-when-the-connectivity-provider-side-has-problems"></a>Таблица ARP, когда она находится в локальной среде или когда на стороне поставщика услуг подключения возникли проблемы
 В таблице ARP отображается только одна запись. Она содержит сопоставление MAC-адреса и IP-адреса, используемого на стороне сети Майкрософт.

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
   0 Microsoft         65.0.0.2  aaaa.bbbb.cccc
```

> [!NOTE]
> В случае возникновения подобной проблемы отправьте запрос на поддержку своему поставщику услуг подключения, чтобы устранить ее.
> 
> 

### <a name="arp-table-when-the-microsoft-side-has-problems"></a>Таблица ARP в случае проблем на стороне сети Майкрософт
* Вы не увидите таблицу ARP для пиринга при наличии проблем на стороне сети Майкрософт.
* Отправьте запрос на поддержку с помощью функции [Справка и поддержка Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). Укажите, что у вас возникла проблема с возможностями подключения уровня 2.

## <a name="next-steps"></a>Дальнейшие действия
* Проверка конфигураций уровня 3 для канала ExpressRoute.
  * Получение сводки маршрутов для определения состояния сеансов BGP.
  * Получение таблицы маршрутов для определения того, какие префиксы объявляются в ExpressRoute.
* Проверка передачи данных путем просмотра входящих и исходящих байтов.
* Отправьте запрос на поддержку с помощью функции [Справка и поддержка Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) , если у вас по-прежнему возникают проблемы.

