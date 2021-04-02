---
title: Общие сведения о функции определения следующего прыжка в Наблюдателе за сетями Azure | Документация Майкрософт
description: Эта статья содержит общие сведения о функции определения следующего прыжка в Наблюдателе за сетями.
services: network-watcher
documentationcenter: na
author: damendo
ms.assetid: febf7bca-e0b7-41d5-838f-a5a40ebc5aac
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: 33c22b5b68b6677e8cf271dc185007316ec44500
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "76844064"
---
# <a name="use-next-hop-to-diagnose-virtual-machine-routing-problems"></a>Используйте следующий прыжок для диагностики проблем маршрутизации виртуальной машины

Трафик от виртуальной машины отправляется в назначение в зависимости от эффективных маршрутов, связанных с сетевым интерфейсом. Следующий прыжок получает тип следующего прыжка и IP-адрес пакета из определенной виртуальной машины и сетевого интерфейса. Знание следующего прыжка поможет определить, направляется ли трафик в предполагаемый пункт назначения или же трафик отправляется неправильно. Неправильная настройка маршрутов, при которой трафик направляется в локальное расположение или на виртуальное устройство, может привести к проблемам подключения. Функция определения следующего прыжка также возвращает таблицу маршрутов, связанную со следующим прыжком. Если маршрут определен как определяемый пользователем, этот маршрут возвращается. В противном случае следующий прыжок возвращает **системный маршрут**.

![Обзор функции определения следующего прыжка](./media/network-watcher-next-hop-overview/figure1.png)

Следующие прыжки, которые могут быть возвращены возможностью следующего прыжка:

* Интернет
* VirtualAppliance
* VirtualNetworkGateway
* Виртуальная сеть
* VirtualNetworkPeering
* VirtualNetworkServiceEndpoint 
* MicrosoftEdge
* Нет

Дополнительные сведения о каждом типе следующего прыжка см. в [этой статье](../virtual-network/virtual-networks-udr-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json).

## <a name="next-steps"></a>Дальнейшие действия

Чтобы научиться использовать следующий прыжок для диагностики проблем маршрутизации сети виртуальных машин, ознакомьтесь с диагностикой проблем с маршрутизацией в сети виртуальной машины с помощью [портала Azure](diagnose-vm-network-routing-problem.md), [PowerShell](diagnose-vm-network-routing-problem-powershell.md) или [Azure CLI](diagnose-vm-network-routing-problem-cli.md).
