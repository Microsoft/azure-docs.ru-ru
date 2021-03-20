---
title: 'Azure ExpressRoute: предварительные требования'
description: На этой странице приведен список требований, которые необходимо выполнить, прежде чем заказать канал ExpressRoute. Здесь также приведен контрольный список.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 09/18/2019
ms.author: duau
ms.openlocfilehash: 1fad1bca18d16ac3b6a654a3c289d0a14e3cd2e2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92204798"
---
# <a name="expressroute-prerequisites--checklist"></a>Предварительные требования и контрольный список для ExpressRoute
Чтобы подключиться к службам Microsoft Cloud через ExpressRoute, вам нужно выполнить условия в следующих разделах.

[!INCLUDE [expressroute-office365-include](../../includes/expressroute-office365-include.md)]

## <a name="azure-account"></a>учетной записи Azure
* Наличие действительной и активной учетной записи Microsoft Azure. Она необходима для организации канала ExpressRoute. Каналы ExpressRoute являются ресурсами в подписках Azure. Подписка Azure является обязательной, даже если подключение ограничено облачными службами Майкрософт, отличными от Azure, например Microsoft 365.
* Активная подписка Microsoft 365 (если используется Microsoft 365 Services). Дополнительные сведения см. в разделе Microsoft 365 специальные требования этой статьи.

## <a name="connectivity-provider"></a>Поставщик услуг подключения

* Чтобы подключиться к Microsoft Cloud, можно работать с [партнером по подключению ExpressRoute](expressroute-locations.md#partners) . Подключение между локальной сетью и Майкрософт можно настроить [тремя различными способами](expressroute-introduction.md).
* Если ваш поставщик не является партнером по подключению ExpressRoute, вы можете подключиться к Microsoft Cloud через [поставщика облачных служб Exchange](expressroute-locations.md#connectivity-through-exchange-providers).

## <a name="network-requirements"></a>Требования к сети
* **Избыточность в каждом одноранговом расположении**. Майкрософт требует, чтобы между маршрутизаторами Майкрософт и маршрутизаторами на каждом канале ExpressRoute были настроены лишние сеансы BGP (даже если имеется только [одно физическое подключение к облачному Exchange](expressroute-faqs.md#onep2plink)).
* **Избыточность для аварийного восстановления**. Корпорация Майкрософт настоятельно рекомендует настроить по крайней мере две цепи ExpressRoute в разных расположениях пиринга, чтобы избежать единой точки отказа.
* **Маршрутизация**. В зависимости от способа подключения к Microsoft Cloud вам или вашему поставщику необходимо настроить и администрировать сеансы BGP для [доменов маршрутизации](expressroute-circuit-peerings.md). Некоторые поставщики подключений Ethernet или облачных услуг Exchange могут предлагать управление BGP в качестве услуги с добавленной стоимостью.
* **Преобразование сетевых адресов (NAT).** Майкрософт принимает только общедоступные IP-адреса через пиринг. Если в локальной сети используются частные IP-адреса, вам или вашему поставщику услуг необходимо преобразовать их в общедоступные IP-адреса [с помощью NAT](expressroute-nat.md).
* **Качество обслуживания.** Skype для бизнеса предоставляет различные услуги (например, передача голоса, видео, текста), которые требуют дифференцированного подхода к качеству обслуживания. Вам и вашему поставщику услуг необходимо соблюдать [требования к качеству обслуживания](expressroute-qos.md).
* **Сетевая безопасность.** При подключении к Microsoft Cloud через ExpressRoute необходимо помнить о [сетевой безопасности](/azure/cloud-adoption-framework/reference/networking-vdc).

## <a name="microsoft-365"></a>Microsoft 365
Если вы планируете включить Microsoft 365 в ExpressRoute, ознакомьтесь со следующими документами, чтобы получить дополнительные сведения о требованиях к Microsoft 365.

* [Azure ExpressRoute для Microsoft 365](/microsoft-365/enterprise/azure-expressroute)
* [Маршрутизация с помощью ExpressRoute для Microsoft 365](/microsoft-365/enterprise/routing-with-expressroute)
* [Высокий уровень доступности и отработка отказов при использовании ExpressRoute](/microsoft-365/enterprise/network-planning-with-expressroute)
* [URL-адреса Microsoft 365 и диапазоны IP-адресов](/microsoft-365/enterprise/urls-and-ip-address-ranges)
* [Планирование сети и настройка производительности для Microsoft 365](/microsoft-365/enterprise/network-planning-and-performance)
* [Планирование сети и миграции для Microsoft 365](/microsoft-365/enterprise/network-and-migration-planning)
* [Интеграция Microsoft 365 с локальными средами](/microsoft-365/enterprise/microsoft-365-integration)
* [Расширенные учебные видеоматериалы об использовании ExpressRoute в Office 365](https://channel9.msdn.com/series/aer/)

## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения об ExpressRoute см. в статье [Вопросы и ответы по ExpressRoute](expressroute-faqs.md).
* Найдите поставщика услуг подключения ExpressRoute. См. статью [Партнеры и одноранговые расположения ExpressRoute](expressroute-locations.md).
* См. требования к [маршрутизации](expressroute-routing.md), [NAT](expressroute-nat.md)и [качеству обслуживания](expressroute-qos.md).
* Настройте подключение ExpressRoute.
  * [Создание канала ExpressRoute](expressroute-howto-circuit-arm.md)
  * [Настройка маршрутизации](expressroute-howto-routing-arm.md)
  * [Связывание виртуальной сети с каналом ExpressRoute](expressroute-howto-linkvnet-arm.md)