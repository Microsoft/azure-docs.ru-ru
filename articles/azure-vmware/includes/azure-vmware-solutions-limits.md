---
title: Ограничения решения Azure VMware
description: Ограничения решения Azure VMware.
ms.topic: include
ms.date: 03/16/2021
ms.openlocfilehash: 0e2359d951f5348b69e95ab7fa046981b2b7b32d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103622164"
---
<!-- Used in /azure/azure-resource-manager/management/azure-subscription-service-limits.md -->

В следующей таблице описаны максимальные ограничения для решения VMware для Azure.

| **Ресурс** | **Ограничение** |
| :-- | :-- |
| Кластеров на частное облако | 12 |
| Минимальное число узлов на кластер | 3 |
| Максимальное число узлов на кластер | 16 |
| Количество узлов в частном облаке | 64 |
| vCenter на частное облако | 1  |
| Пары сайтов ХККС | 3 с расширенным выпуском, 10 с выпуском Enterprise Edition |
| Максимальное число связанных Сддкс для AVS ExpressRoute | 4 |
| AVS ExpressRoute портспид | 10 Гбит/с<br />Используемый шлюз виртуальной сети определяет фактическую пропускную способность. Дополнительные сведения см. в статье [о шлюзах виртуальной сети ExpressRoute](../../expressroute/expressroute-about-virtual-network-gateways.md) . | 
| Общедоступные IP-адреса, предоставляемые через Вван | 100 |
| ограничения емкости vSAN | 75% от общего числа доступных для использования (используйте 25% для соглашения об уровне обслуживания)  |

Для других ограничений, характерных для VMware, используйте [средство настройки VMware "максимум](https://configmax.vmware.com/)".
