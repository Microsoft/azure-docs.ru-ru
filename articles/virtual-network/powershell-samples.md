---
title: Примеры скриптов Azure PowerShell для виртуальной сети
description: Изучите примеры Azure PowerShell для управления виртуальными сетями, включая пример сценария для создания виртуальной сети для многоуровневых приложений.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 07/15/2019
ms.author: kumud
ms.openlocfilehash: b3107d521fb79e3ea6cfe190abd2ec3fb7859cb3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "87288225"
---
# <a name="azure-powershell-samples-for-virtual-network"></a>Примеры скриптов Azure PowerShell для виртуальной сети

В таблице ниже приведены ссылки на скрипты Azure PowerShell.

| Скрипт | Описание |
|----|----|
| [Создание виртуальной сети для многоуровневых приложений](./scripts/virtual-network-powershell-sample-multi-tier-application.md) | Создание виртуальной сети с интерфейсной и внутренней подсетями. Трафик к интерфейсной подсети принимается по протоколу HTTP, в то время как трафик к внутренней подсети принимается только от SQL через порт 1433. |
| [Пиринг между двумя виртуальными сетями](./scripts/virtual-network-powershell-sample-peer-two-virtual-networks.md) | Создание и подключение двух виртуальных сетей в одном регионе. |
| [Маршрутизация трафика через виртуальный сетевой модуль](./scripts/virtual-network-powershell-sample-route-traffic-through-nva.md) | Создание виртуальной сети с интерфейсной и внутренней подсетями, а также виртуальной машиной, которая может маршрутизировать трафик между этими двумя подсетями. |
| [Фильтрация входящего и исходящего сетевого трафика виртуальной машины](./scripts/virtual-network-powershell-sample-filter-network-traffic.md) | Создание виртуальной сети с интерфейсной и внутренней подсетями. Входящий сетевой трафик в интерфейсной подсети ограничен протоколами HTTP и HTTPS. Исходящий интернет-трафик из внутренней подсети запрещен. |
|[Настройка виртуальной сети с двойным стеком (IPv4 и IPv6) и Load Balancer (цен. категория "Базовый")](./scripts/virtual-network-powershell-sample-ipv6-dual-stack.md)|Развертывание виртуальной сети с двойным стеком (IPv4 и IPv6), двумя виртуальными машинами и Azure Load Balancer категории "Базовый" с общедоступными IP-адресами IPv4 и IPv6. |
|[Настройка виртуальной сети с двойным стеком (IPv4 и IPv6) и Load Balancer (цен. категория "Стандартный")](./scripts/virtual-network-powershell-sample-ipv6-dual-stack-standard-load-balancer.md)|Развертывает виртуальную сеть с двойным стеком (IPv4 и IPv6), двумя виртуальными машинами и Azure Load Balancer (цен. категория "Стандартный") с общедоступными IP-адресами IPv4 и IPv6. |
