---
title: Примеры Azure CLI для виртуальной сети
description: Изучите различные примеры сценариев, которые можно использовать для выполнения задач в Azure CLI, включая создание виртуальной сети для многоуровневых приложений.
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
ms.custom: devx-track-azurecli
ms.openlocfilehash: 97d63598f6a9cca199779e56cc211b9622117301
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105048839"
---
# <a name="azure-cli-samples-for-virtual-network"></a>Примеры Azure CLI для виртуальной сети

Следующая таблица содержит ссылки на скрипты Bash, созданные с помощью команд Azure CLI:

| Скрипт | Описание |
|----|----|
| [Создание виртуальной сети для многоуровневых приложений](./scripts/virtual-network-cli-sample-multi-tier-application.md) | Создание виртуальной сети с интерфейсной и внутренней подсетями. Трафик к интерфейсной подсети принимается по протоколам HTTP и SSH, в то время как трафик к внутренней подсети принимается только от MySQL по порту 3306. |
| [Пиринг между двумя виртуальными сетями](./scripts/virtual-network-cli-sample-peer-two-virtual-networks.md) | Создание и подключение двух виртуальных сетей в одном регионе. |
| [Маршрутизация трафика через виртуальный сетевой модуль](./scripts/virtual-network-cli-sample-route-traffic-through-nva.md) | Создание виртуальной сети с интерфейсной и внутренней подсетями, а также виртуальной машиной, которая может маршрутизировать трафик между этими двумя подсетями. |
| [Фильтрация входящего и исходящего сетевого трафика виртуальной машины](./scripts/virtual-network-cli-sample-filter-network-traffic.md) | Создание виртуальной сети с интерфейсной и внутренней подсетями. Входящий сетевой трафик в интерфейсной подсети ограничен протоколами HTTP, HTTPS и SSH. Исходящий интернет-трафик из внутренней подсети запрещен. |
|[Настройка виртуальной сети с двойным стеком (IPv4 и IPv6) и Load Balancer (цен. категория "Базовый")](./scripts/virtual-network-cli-sample-ipv6-dual-stack.md)|Развертывание виртуальной сети с двойным стеком (IPv4 и IPv6), двумя виртуальными машинами и Azure Load Balancer категории "Базовый" с общедоступными IP-адресами IPv4 и IPv6. |
|[Настройка виртуальной сети с двойным стеком (IPv4 и IPv6) и Load Balancer (цен. категория "Стандартный")](./scripts/virtual-network-cli-sample-ipv6-dual-stack-standard-load-balancer.md)|Развертывает виртуальную сеть с двойным стеком (IPv4 и IPv6), двумя виртуальными машинами и Azure Load Balancer (цен. категория "Стандартный") с общедоступными IP-адресами IPv4 и IPv6. |
|[Руководство. Создание и проверка шлюза NAT (Azure CLI)](./tutorial-create-nat-gateway-cli.md)|Создайте и проверьте шлюз NAT, используя виртуальную машину в качестве источника и назначения. |