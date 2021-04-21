---
title: О поддержке Azure Route Server (предварительная версия) для ExpressRoute и Azure VPN
description: Узнайте, как Azure Route Server взаимодействует с ExpressRoute и VPN-шлюзами Azure.
services: route-server
author: duongau
ms.service: route-server
ms.topic: conceptual
ms.date: 03/02/2021
ms.author: duau
ms.openlocfilehash: 6e588c7c0381c6825bcf75cbbe28a1dd6b865940
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101680532"
---
# <a name="about-azure-route-server-preview-support-for-expressroute-and-azure-vpn"></a>О поддержке Azure Route Server (предварительная версия) для ExpressRoute и Azure VPN

Azure Route Server поддерживает не только виртуальные сетевые устройства сторонних производителей (NVA), работающие в Azure, но также тесно интегрируется с ExpressRoute и VPN-шлюзами Azure. Пиринг BGP между шлюзом и Azure Route Server не нужно настраивать или управлять им. Обмен маршрутами между шлюзом и Azure Route Server можно включить с помощью простого [изменения конфигурации](quickstart-configure-route-server-powershell.md#route-exchange).

> [!IMPORTANT]
> Сервер маршрутизации Azure (предварительная версия) сейчас предоставляется в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="how-does-it-work"></a>Как это работает?

При развертывании Azure Route Server вместе со шлюзом ExpressRoute и с виртуальным сетевым модулем (NVA) в виртуальной сети по умолчанию, маршруты, которые Azure Route Server получает от NVA и шлюза ExpressRoute, не распространяются между ними. После включения обмена маршрутами шлюз ExpressRoute и модуль NVA будут знать о маршрутах друг друга.

Например, рассмотрим следующую диаграмму:

* Устройство SDWAN получит с помощью Azure Route Server маршрут от сети "On-prem 2" с подключением к ExpressRoute, а также маршрут виртуальной сети.

* Шлюз ExpressRoute получит маршрут от сети "On-prem 1" с подключением к устройству SDWAN, а также маршрут виртуальной сети от Azure Route Server.

    ![Схема: настройка ExpressRoute с Route Server.](./media/expressroute-vpn-support/expressroute-with-route-server.png)

Устройство SDWAN можно также заменить на VPN-шлюз Azure. Так как VPN-шлюз Azure и шлюз ExpressRoute являются полностью управляемыми, нужно настроить только обмен маршрутами между двумя локальными сетями для обеспечения их взаимодействия друг с другом.

> [!IMPORTANT] 
> VPN-шлюз Azure должен быть настроен в режиме [**активный — активный**](../vpn-gateway/vpn-gateway-activeactive-rm-powershell.md).
>

![На схеме показана настройка ExpressRoute и VPN-шлюза, выполненная с помощью Route Server.](./media/expressroute-vpn-support/expressroute-and-vpn-with-route-server.png)

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте подробности об [Azure Route Server](route-server-faq.md).
- Узнайте, как [настраивать Azure Route Server](quickstart-configure-route-server-powershell.md).
- Узнайте подробности о [Совместной работе Azure ExpressRoute и Azure VPN](../expressroute/expressroute-howto-coexist-resource-manager.md).
