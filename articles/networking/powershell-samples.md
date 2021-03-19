---
title: Azure PowerShell примеры — сетевые подключения
description: Узнайте о Azure PowerShell примерах для работы в сети, включая пример создания виртуальной сети для многоуровневых приложений.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 05/24/2017
ms.author: gwallace
ms.openlocfilehash: aa3304cbabb183368f0e47415e64120854f43535
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87282081"
---
# <a name="azure-powershell-samples-for-networking"></a>Примеры Azure PowerShell для работы с сетями

В таблице ниже приведены ссылки на скрипты, созданные с помощью Azure PowerShell.

| Скрипт | Описание |
|-|-|
|**Подключение между ресурсами Azure**||
| [Создание виртуальной сети для многоуровневых приложений](./scripts/virtual-network-powershell-sample-multi-tier-application.md?toc=%2fazure%2fnetworking%2ftoc.json) | Создание виртуальной сети с интерфейсной и внутренней подсетями. Трафик к интерфейсной подсети принимается по протоколу HTTP, в то время как трафик к внутренней подсети принимается только от SQL через порт 1433. |
| [Пиринг между двумя виртуальными сетями](./scripts/virtual-network-powershell-sample-peer-two-virtual-networks.md?toc=%2fazure%2fnetworking%2ftoc.json) | Создание и подключение двух виртуальных сетей в одном регионе. |
| [Маршрутизация трафика через виртуальный сетевой модуль](./scripts/virtual-network-powershell-sample-route-traffic-through-nva.md?toc=%2fazure%2fnetworking%2ftoc.json) | Создание виртуальной сети с интерфейсной и внутренней подсетями, а также виртуальной машиной, которая может маршрутизировать трафик между этими двумя подсетями. |
| [Фильтрация входящего и исходящего сетевого трафика виртуальной машины](./scripts/virtual-network-powershell-filter-network-traffic.md?toc=%2fazure%2fnetworking%2ftoc.json) | Создание виртуальной сети с интерфейсной и внутренней подсетями. Входящий сетевой трафик в интерфейсной подсети ограничен протоколами HTTP и HTTPS. Исходящий трафик в Интернет из внутренней подсети запрещен. |
|**Направление трафика и балансировка нагрузки**||
| [Балансировка трафика на виртуальных машинах для обеспечения высокой доступности](./scripts/load-balancer-windows-powershell-sample-nlb.md?toc=%2fazure%2fnetworking%2ftoc.json) | Создает несколько виртуальных машин с высокодоступной конфигурацией с балансировкой нагрузки. |
| [Направление трафика через несколько регионов для обеспечения высокого уровня доступности приложений](./scripts/traffic-manager-powershell-websites-high-availability.md?toc=%2fazure%2fnetworking%2ftoc.json) |  Создание двух планов службы приложений, двух веб-приложений, а также профиля и двух конечных точек диспетчера трафика. |
| | |
