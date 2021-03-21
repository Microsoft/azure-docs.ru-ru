---
title: Решение Azure VMware по Клаудсимпле — локальное подключение с помощью ExpressRoute
description: В этой статье описывается, как запросить локальное подключение с помощью ExpressRoute из сети Клаудсимпле Region.
author: Ajayan1008
ms.author: v-hborys
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 27d1d9e54838d9f45a28d634fa1c24fb8785aee1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97899241"
---
# <a name="connect-from-on-premises-to-cloudsimple-using-expressroute"></a>Подключение из локальной среды к Клаудсимпле с помощью ExpressRoute

Если у вас уже есть подключение Azure ExpressRoute из внешнего расположения (например, из локальной среды) в Azure, вы можете подключить его к среде Клаудсимпле. Это можно сделать с помощью функции Azure, которая позволяет двум каналам ExpressRoute подключаться друг к другу. Этот метод устанавливает защищенную, частную, высокоскоростную и низкую задержку подключение между двумя средами.

[![Подключение к локальной системе ExpressRoute — Global Reach](media/cloudsimple-global-reach-connection.png)](media/cloudsimple-global-reach-connection.png)

## <a name="before-you-begin"></a>Перед началом

Для установления подключения Global Reach из локальной среды требуется блок сетевых адресов **/29** .  Адресное пространство/29 используется для транзитной сети между каналами ExpressRoute.  Транзитная сеть не должна перекрываться ни с одной из виртуальных сетей Azure, локальных сетей или Клаудсимпле частными облачными сетями.

## <a name="prerequisites"></a>Предварительные условия

* Перед установкой подключения между сетями и частными облаками Клаудсимпле необходимо создать канал ExpressRoute Azure.
* Пользователь должен иметь права доступа для создания ключей авторизации в канале ExpressRoute.

## <a name="scenarios"></a>Сценарии

Подключение локальной сети к сети частного облака позволяет использовать частное облако различными способами, включая следующие сценарии.

* Получите доступ к сети частного облака без создания VPN-подключения типа "сеть — сеть".
* Используйте локальные Active Directory в качестве источника удостоверений в частном облаке.
* Перенесите виртуальные машины, работающие локально, в частное облако.
* Используйте частное облако в качестве части решения для аварийного восстановления.
* Использование локальных ресурсов на виртуальных машинах рабочей нагрузки частного облака.

## <a name="connecting-expressroute-circuits"></a>Подключение каналов ExpressRoute

Чтобы установить подключение ExpressRoute, необходимо создать авторизацию в канале ExpressRoute и предоставить сведения об авторизации для Клаудсимпле.


### <a name="create-expressroute-authorization"></a>Создание авторизации ExpressRoute

1. Войдите на портал Azure.

2. В верхней строке поиска найдите **канал expressroute** и щелкните **каналы Expressroute** в разделе **службы**.
    [![Каналы ExpressRoute](media/azure-expressroute-transit-search.png)](media/azure-expressroute-transit-search.png)

3. Выберите канал ExpressRoute, который планируется подключить к сети Клаудсимпле.

4. На странице ExpressRoute щелкните **авторизации**, введите имя авторизации и нажмите кнопку **сохранить**.
    [![Авторизация канала ExpressRoute](media/azure-expressroute-transit-authorizations.png)](media/azure-expressroute-transit-authorizations.png)

5. Скопируйте идентификатор ресурса и ключ авторизации, щелкнув значок копирования. Вставьте идентификатор и ключ в текстовый файл.
    [![Копия авторизации канала ExpressRoute](media/azure-expressroute-transit-authorization-copy.png)](media/azure-expressroute-transit-authorization-copy.png)

    > [!IMPORTANT]
    > **Идентификатор ресурса** должен быть скопирован из пользовательского интерфейса и должен быть в формате, ```/subscriptions/<subscription-ID>/resourceGroups/<resource-group-name>/providers/Microsoft.Network/expressRouteCircuits/<express-route-circuit-name>``` если он предоставляется для поддержки.

6. Отправьте билет с <a href="https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest" target="_blank">поддержкой</a> создаваемого подключения.
    * Тип проблемы: **Техническая**
    * Подписка: **Подписка, в которой развернута служба клаудсимпле**
    * Служба: **решение VMware по клаудсимпле**
    * Тип проблемы: **запрос на обслуживание**
    * Подтип проблемы: **Создание подключения ExpressRoute к локальной среде**
    * Укажите идентификатор ресурса и ключ авторизации, которые были скопированы и сохранены в области сведений.
    * Укажите/29 адресного пространства сети для транзитной сети.
    * Вы отправляете маршрут по умолчанию через ExpressRoute?
    * Должен ли трафик частного облака использовать маршрут по умолчанию, отправленный через ExpressRoute?

    > [!IMPORTANT]
    > Отправка маршрута по умолчанию позволяет отправлять весь Интернет-трафик из частного облака с помощью локального подключения к Интернету.  Чтобы отключить маршрут по умолчанию, настроенный в частном облаке, и использовать локальный маршрут по умолчанию для подключения, укажите сведения в запрос в службу поддержки.

## <a name="next-steps"></a>Дальнейшие действия

* [Дополнительные сведения о сетевых подключениях Azure](cloudsimple-azure-network-connection.md)  
