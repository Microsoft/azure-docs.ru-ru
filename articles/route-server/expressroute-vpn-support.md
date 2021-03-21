---
title: О поддержке Azure Route Server (Предварительная версия) для ExpressRoute и Azure VPN
description: Узнайте, как сервер маршрутизации Azure взаимодействует с ExpressRoute и VPN-шлюзами Azure.
services: route-server
author: duongau
ms.service: route-server
ms.topic: conceptual
ms.date: 03/02/2021
ms.author: duau
ms.openlocfilehash: 6e588c7c0381c6825bcf75cbbe28a1dd6b865940
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101680532"
---
# <a name="about-azure-route-server-preview-support-for-expressroute-and-azure-vpn"></a>О поддержке Azure Route Server (Предварительная версия) для ExpressRoute и Azure VPN

Сервер маршрутизации Azure поддерживает не только виртуальные сетевые устройства сторонних производителей (NVA), работающие в Azure, но также тесно интегрируется с ExpressRoute и VPN-шлюзами Azure. Вам не нужно настраивать пиринг BGP между шлюзом и сервером маршрутизации Azure, а также управлять им. Можно включить обмен маршрутами между шлюзом и сервером маршрутизации Azure с помощью простого [изменения конфигурации](quickstart-configure-route-server-powershell.md#route-exchange).

> [!IMPORTANT]
> Сервер маршрутизации Azure (предварительная версия) сейчас предоставляется в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="how-does-it-work"></a>Как это работает?

При развертывании сервера маршрутизации Azure вместе с шлюзом ExpressRoute и NVA в виртуальной сети по умолчанию сервер маршрутизации Azure не распространяет маршруты, получаемые от шлюза NVA и ExpressRoute, между ними. После включения обмена маршрутами ExpressRoute и NVA будут изучать маршруты друг друга.

Например, на следующей схеме:

* Устройство СДВАН получит от сервера маршрутизации Azure маршрут из локальной среды 2, которая подключена к ExpressRoute, а также маршрут виртуальной сети.

* Шлюз ExpressRoute получит маршрут от "локального 1", который подключен к устройству СДВАН, а также маршрут виртуальной сети с сервера маршрутизации Azure.

    ![Схема, на которой настроена ExpressRoute с сервером маршрутов.](./media/expressroute-vpn-support/expressroute-with-route-server.png)

Вы также можете заменить устройство СДВАН на VPN-шлюз Azure. Так как VPN-шлюзы Azure и ExpressRoute полностью управляются, необходимо только настроить обмен маршрутами между двумя локальными сетями для взаимодействия друг с другом.

> [!IMPORTANT] 
> VPN-шлюз Azure должен быть настроен в режиме " [**активный — активный**](../vpn-gateway/vpn-gateway-activeactive-rm-powershell.md) ".
>

![На схеме показана сеть ExpressRoute и VPN-шлюз, настроенный с помощью сервера маршрутов.](./media/expressroute-vpn-support/expressroute-and-vpn-with-route-server.png)

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [сервере маршрутизации Azure](route-server-faq.md).
- Узнайте, как [настроить сервер маршрутизации Azure](quickstart-configure-route-server-powershell.md).
- Узнайте больше о [сосуществовании Azure ExpressRoute и Azure VPN](../expressroute/expressroute-howto-coexist-resource-manager.md).
