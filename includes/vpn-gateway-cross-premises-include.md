---
title: включить файл
description: включить файл
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 12/17/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: b283add6cff1400cc3141f4fba3f0f3939ee34aa
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "97664983"
---
|  | **Точка-сеть** | **Подключение "сеть — сеть"** . | **ExpressRoute** |
| --- | --- | --- | --- |
| **Поддерживаемые службы Azure** |Облачные службы и виртуальные машины |Облачные службы и виртуальные машины |[Список служб](../articles/expressroute/expressroute-faqs.md#supported-services) |
| **Типичные пропускные способности** |В зависимости от SKU шлюза |Обычно < 1 Гбит/с (совокупная) |50 Мбит/с, 100 Мбит/с, 200 Мбит/с, 500 Мбит/с, 1 Гбит/с, 2 Гбит/с, 5 Гбит/с, 10 Гбит/с |
| **Поддерживаемые протоколы** |SSTP, OpenVPN и IPsec |IPsec |Прямое подключение через виртуальные сети, технологии виртуальных частных сетей NSP (MPLS, VPLS) |
| **Маршрутизация** |На основе маршрутов (динамическая) |Мы поддерживаем маршрутизацию на основе политик (PolicyBased, статическая маршрутизация) и маршрутов (RouteBased, VPN с динамической маршрутизацией). |BGP |
| **Устойчивость подключения** |Активное-пассивное |"Активный — пассивный" или "активный —активный" |Активное-активное |
| **Типичный случай использования** |Безопасный доступ к виртуальным сетям Azure для удаленных пользователей |Сценарии разработки, тестирования и лабораторного использования, а также рабочие нагрузки небольшого и среднего масштаба для облачных служб и виртуальных машин |Доступ ко всем службам Azure (проверенный список), критически важные рабочие нагрузки и рабочие нагрузки корпоративного уровня, архивация, большие данные, Azure в качестве сайта аварийного восстановления |
| **Соглашение об уровне обслуживания** |[Соглашение об уровне обслуживания](https://azure.microsoft.com/support/legal/sla/) |[Соглашение об уровне обслуживания](https://azure.microsoft.com/support/legal/sla/) |[Соглашение об уровне обслуживания](https://azure.microsoft.com/support/legal/sla/) |
| **Цены** |[Цены](https://azure.microsoft.com/pricing/details/vpn-gateway/) |[Цены](https://azure.microsoft.com/pricing/details/vpn-gateway/) |[Цены](https://azure.microsoft.com/pricing/details/expressroute/) |
| **Техническая документация** |[Документация по VPN-шлюзу](https://azure.microsoft.com/documentation/services/vpn-gateway/) |[Документация по VPN-шлюзу](https://azure.microsoft.com/documentation/services/vpn-gateway/) |[Документация по ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) |
| **Часто задаваемые вопросы** |[VPN-шлюз: вопросы и ответы](../articles/vpn-gateway/vpn-gateway-vpn-faq.md) |[VPN-шлюз: вопросы и ответы](../articles/vpn-gateway/vpn-gateway-vpn-faq.md) |[Часто задаваемые вопросы об ExpressRoute](../articles/expressroute/expressroute-faqs.md) |
