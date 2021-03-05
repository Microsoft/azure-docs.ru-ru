---
title: Ограничения решения Azure VMware
description: Ограничения решения Azure VMware.
ms.topic: include
ms.date: 03/04/2021
ms.openlocfilehash: 99c27694a300284cfd99b8411306c90ad4d8a12e
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102194060"
---
<!-- Used in /azure/azure-resource-manager/management/azure-subscription-service-limits.md -->

В следующей таблице описаны максимальные ограничения для решения VMware для Azure.

| **Ресурс** | **Ограничение** |
| :-- | :-- |
| Кластеров на частное облако | 4 |
| Минимальное число узлов на кластер | 3 |
| Максимальное число узлов на кластер | 16 |
| Количество узлов в частном облаке | 64 |
| vCenter на частное облако | 1  |
| Пары сайтов ХККС | 3 с расширенным выпуском, 10 с выпуском Enterprise Edition |
| Максимальное число связанных Сддкс для AVS ExpressRoute | 4 |
| AVS ExpressRoute портспид | 10 Гбит/с | 
| Общедоступные IP-адреса, предоставляемые через Вван | 100 |
| ограничения емкости vSAN | 75% от общего числа доступных для использования (используйте 25% для соглашения об уровне обслуживания)  |

Для других ограничений, характерных для VMware, используйте [средство настройки VMware "максимум](https://configmax.vmware.com/)".
