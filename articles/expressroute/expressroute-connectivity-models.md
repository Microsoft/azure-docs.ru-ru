---
title: Azure ExpressRoute. модели подключения
description: Проверьте подключение между сетью клиента, Microsoft Azure и службами Microsoft 365. Клиенты могут использовать поставщики MPLS, облачные Exchange и Ethernet.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 10/07/2020
ms.author: duau
ms.openlocfilehash: fb35a03b5dd8780445eb27f485966e3c3452feea
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91978766"
---
# <a name="expressroute-connectivity-models"></a>Модели подключения ExpressRoute
Вы можете создать подключение между локальной сетью и облаком Майкрософт четырьмя разными способами: [Клаудексчанже совместное размещение](#CloudExchange), [подключение Ethernet типа "точка-точка](#Ethernet)", подключение ["любой к любому" (IPVPN)](#IPVPN)и [ExpressRoute Direct](#Direct). Поставщики услуг подключения могут предложить одну или несколько моделей подключения. Обратитесь к поставщику услуг подключения и подберите наиболее подходящую вам модель.
<br><br>

:::image type="content" source="./media/expressroute-connectivity-models/expressroute-connectivity-models-diagram.png" alt-text="Схема моделей подключения ExpressRoute":::

## <a name="co-located-at-a-cloud-exchange"></a><a name="CloudExchange"></a>Совместное размещение в Cloud Exchange
Если вы совместно располагаете в одном месте с облачным Exchange, вы можете заказать виртуальные перекрестные подключения к облаку Майкрософт через Ethernet-обмен поставщика совместного размещения. Поставщики услуг совместного размещения могут предлагать либо кросс-подключения второго уровня, либо управляемые кросс-подключения третьего уровня между вашей инфраструктурой на сервере совместного размещения и облаком Майкрософт.

## <a name="point-to-point-ethernet-connections"></a><a name="Ethernet"></a>Подключения Ethernet типа "точка — точка"
Локальные центры обработки данных и офисы можно подключить к облаку Майкрософт через связи Ethernet типа "точка-точка". Поставщики подключений Ethernet типа "точка-точка" могут предлагать подключения второго уровня или управляемые подключения третьего уровня между вашей сетью и облаком Майкрософт.

## <a name="any-to-any-ipvpn-networks"></a><a name="IPVPN"></a>Сети типа "любой к любому" (IPVPN)
Глобальную вычислительную сеть можно интегрировать с облаком Майкрософт. Поставщики IP VPN (обычно это MPLS VPN) предлагают подключение между филиалами и центрами обработки данных типа "любой к любому". Облако Майкрософт можно связать с вашей глобальной вычислительной сетью, так чтобы оно выглядело как любой другой филиал. Обычно поставщики глобальных вычислительных сетей предлагают управляемые подключения третьего уровня. Возможности и функции ExpressRoute одинаковы для всех описанных выше моделей.

## <a name="direct-from-expressroute-sites"></a><a name="Direct"></a>Непосредственно с сайтов ExpressRoute
Вы можете напрямую подключаться к глобальной сети Майкрософт в расположении, стратегичено распределенном по всему миру. [ExpressRoute Direct](expressroute-erdirect-about.md) обеспечивает подключение с двумя 100 Гбит/с или 10 Гбит/с, которое поддерживает подключение "активный/активный" в масштабе.

## <a name="next-steps"></a>Дальнейшие действия
* Узнайте больше о подключениях ExpressRoute и доменах маршрутизации. См. статью [Каналы ExpressRoute и домены маршрутизации](expressroute-circuit-peerings.md).
* Узнайте о функциях ExpressRoute. См. статью [Технический обзор ExpressRoute](expressroute-introduction.md).
* Найти поставщика услуг. См. статью [Партнеры и одноранговые расположения ExpressRoute](expressroute-locations.md).
* Убедитесь, что выполнены все необходимые условия. Ознакомьтесь с разделом [Предварительные требования и контрольный список для ExpressRoute](expressroute-prerequisites.md).
* См. сведения о требованиях для [маршрутизации](expressroute-routing.md) и [NAT](expressroute-nat.md) и [QoS](expressroute-qos.md).
* Настройте подключение ExpressRoute.
  * [Создание канала ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md)
  * [Настройка маршрутизации](expressroute-howto-routing-portal-resource-manager.md)
  * [Связывание виртуальной сети с каналом ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md)