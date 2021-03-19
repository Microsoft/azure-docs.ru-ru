---
title: Azure CLI примеры — сетевые подключения
description: Узнайте о Azure CLI примерах для сетей, включая примеры подключения между ресурсами Azure и примерами балансировки нагрузки и направления трафика.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: timlt
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 04/25/2017
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: c038f0d238646f43b93ba2a2c6a1120ab5feccee
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87497296"
---
# <a name="azure-cli-samples-for-networking"></a>Примеры Azure CLI для работы с сетью

В следующей таблице содержатся ссылки на скрипты Bash, созданные с помощью Azure CLI.

| Скрипт | Описание |
|-|-|
|**Подключение между ресурсами Azure**||
| [Создание виртуальной сети для многоуровневых приложений](./scripts/virtual-network-cli-sample-multi-tier-application.md?toc=%2fazure%2fnetworking%2ftoc.json) | Создание виртуальной сети с интерфейсной и внутренней подсетями. Трафик к интерфейсной подсети принимается по протоколам HTTP и SSH, в то время как трафик к внутренней подсети принимается только от MySQL по порту 3306. |
| [Пиринг между двумя виртуальными сетями](./scripts/virtual-network-cli-sample-peer-two-virtual-networks.md?toc=%2fazure%2fnetworking%2ftoc.json) | Создание и подключение двух виртуальных сетей в одном регионе. |
| [Маршрутизация трафика через виртуальный сетевой модуль](./scripts/virtual-network-cli-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetworking%2ftoc.json) | Создание виртуальной сети с интерфейсной и внутренней подсетями, а также виртуальной машиной, которая может маршрутизировать трафик между этими двумя подсетями. |
| [Фильтрация входящего и исходящего сетевого трафика виртуальной машины](./scripts/virtual-network-filter-network-traffic.md?toc=%2fazure%2fnetworking%2ftoc.json) | Создание виртуальной сети с интерфейсной и внутренней подсетями. Входящий сетевой трафик в интерфейсной подсети ограничен протоколами HTTP, HTTPS и SSH. Исходящий трафик в Интернет из внутренней подсети запрещен. |
|**Направление трафика и балансировка нагрузки**||
| [Балансировка нагрузки нескольких веб-сайтов на виртуальных машинах](./scripts/load-balancer-linux-cli-load-balance-multiple-websites-vm.md?toc=%2fazure%2fnetworking%2ftoc.json) | Создание двух виртуальных машин с несколькими IP-конфигурациями, присоединенных к группе доступности Azure, которые доступны через Azure Load Balancer. |
| [Направление трафика через несколько регионов для обеспечения высокого уровня доступности приложений](./scripts/traffic-manager-cli-websites-high-availability.md?toc=%2fazure%2fnetworking%2ftoc.json) |  Создание двух планов службы приложений, двух веб-приложений, а также профиля и двух конечных точек диспетчера трафика. |
| | |
