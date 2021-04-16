---
title: включить файл
description: включить файл
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: include
ms.date: 04/05/2021
ms.author: duau
ms.custom: include file
ms.openlocfilehash: 27f5755ce8b7d204cad6cdc2281d7992bf86615a
ms.sourcegitcommit: c2a41648315a95aa6340e67e600a52801af69ec7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106504713"
---
Во время создания шлюза виртуальной сети нужно указать SKU шлюза, который вы хотите использовать. Если выбрать более высокий SKU шлюза, то шлюз получит больше ЦП и большую пропускную способность сети. В результате шлюз сможет поддерживать более высокую пропускную способность для виртуальной сети. 

Для шлюзов виртуальной сети ExpressRoute можно использовать следующие значения SKU.

|     | Сосуществование VPN-шлюза и ExpressRoute | FastPath | Максимальное число подключений к каналу |
| --- | --- | --- | --- |
| **Standard Edition** | Да | Нет | 4 |
| **HighPerformance** | Да | Нет | 4 |
| **UltraPerformance** | Да | Да | 16 |