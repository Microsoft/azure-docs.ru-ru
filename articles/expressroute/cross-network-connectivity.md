---
title: Подключение между сетями в Azure
description: На этой странице описывается сценарий приложения для подключения между сетями и решения на основе сетевых компонентов Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 04/03/2019
ms.author: duau
ms.openlocfilehash: 018afa1b2a31ebd44925a3fc79cbdc729b2d4695
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92202503"
---
# <a name="cross-network-connectivity"></a>Возможность подключения нескольких сетей

Компания Fabrikam Inc. активно работает в восточной части США и имеет развернутые службы Azure в этом регионе. Fabrikam использует ExpressRoute для связи между серверными компонентами, размещенными в локальной сети и в Azure. Аналогично, Contoso Ltd. имеет присутствие и развертывание Azure в западной части США. Contoso использует ExpressRoute для связи между серверными компонентами, размещенными в локальной сети и в Azure.  

Fabrikam Inc. получает Contoso Ltd. После слияния компания Fabrikam хочет присоединиться к сетям. Этот сценарий показан на следующей схеме:

![Сценарий приложения](./media/cross-network-connectivity/premergerscenario.png)

Пунктирные стрелки в середине приведенного выше рисунка указывают на требуемые сетевые соединения. В частности, существует три типа перекрестных подключений: 1) Fabrikam и Contoso виртуальных сетей Cross Connect, 2) кросс-региональный локальный и виртуальных сетей перекрестное подключение (т. е. подключение локальной сети Fabrikam к виртуальной сети Contoso и подключение локальной сети Contoso к сети Fabrikam) и 3) Fabrikam и локальная сеть Contoso Cross Connect. 

В следующей таблице показана таблица маршрутов частного пиринга ExpressRoute компании Contoso Ltd. до слияния.

![Таблица маршрутов Contoso ExpressRoute перед слиянием](./media/cross-network-connectivity/contosoexr-rt-premerger.png)

В следующей таблице показаны действующие маршруты виртуальной машины в подписке Contoso до слияния. В каждой таблице виртуальная машина в виртуальной сети учитывает адресное пространство виртуальной сети и локальную сеть Contoso, помимо используемых по умолчанию.

![Маршруты виртуальных машин Contoso перед слиянием](./media/cross-network-connectivity/contosovm-routes-premerger.png)

В следующей таблице показана таблица маршрутов частного пиринга ExpressRoute компании Fabrikam Inc. до слияния.

![Таблица маршрутов компании Fabrikam ExpressRoute перед слиянием](./media/cross-network-connectivity/fabrikamexr-rt-premerger.png)

В следующей таблице показаны действующие маршруты виртуальной машины в подписке Fabrikam до слияния. В каждой таблице виртуальная машина в виртуальной сети учитывает адресное пространство виртуальной сети и локальную сеть Fabrikam, помимо значений по умолчанию.

![Маршруты виртуальной машины Fabrikam перед слиянием](./media/cross-network-connectivity/fabrikamvm-routes-premerger.png)

В этой статье мы рассмотрим пошаговые инструкции и расскажем, как добиться нужных перекрестных подключений с помощью следующих компонентов сети Azure:

* [Пиринг виртуальной сети][Virtual network peering] 
* [Подключение ExpressRoute виртуальной сети][connection]
* [Global Reach][Global Reach] 

## <a name="cross-connecting-vnets"></a>Перекрестное подключение виртуальных сетей

Пиринг виртуальных сетей (пиринг виртуальной сети) обеспечивает наиболее оптимальную и оптимальную производительность сети при подключении двух виртуальных сетей. Пиринг виртуальных сетей поддерживает пиринг двух виртуальных сетей как в одном регионе Azure (называемом пирингом виртуальных сетей), так и в двух разных регионах Azure (обычно именуемых глобальным пирингом виртуальных сетей). 

Давайте настроим глобальную пиринг виртуальной сети между виртуальных сетей в подписках Contoso и Fabrikam Azure. Сведения о создании пиринга между двумя виртуальными сетями см. в статье [Создание пиринга виртуальной сети][Configure VNet peering] .

На следующем рисунке показана сетевая архитектура после настройки пиринга глобальной виртуальной сети.

![Архитектура после пиринга VNet](./media/cross-network-connectivity/vnet-peering.png )

В следующей таблице показаны маршруты, известные виртуальной машине подписки contoso. Обратите внимание на последнюю запись таблицы. Эта запись является результатом перекрестного подключения виртуальных сетей.

![Маршруты виртуальных машин Contoso после пиринга виртуальной сети](./media/cross-network-connectivity/contosovm-routes-peering.png)

В следующей таблице показаны маршруты, известные для виртуальной машины подписки Fabrikam. Обратите внимание на последнюю запись таблицы. Эта запись является результатом перекрестного подключения виртуальных сетей.

![Маршруты виртуальных машин Fabrikam после пиринга виртуальной сети](./media/cross-network-connectivity/fabrikamvm-routes-peering.png)

Пиринг виртуальных сетей напрямую связывает две виртуальные сети (см. следующий прыжок для записи *внетглобалпиринг* в двух приведенных выше таблицах)

## <a name="cross-connecting-vnets-to-the-on-premises-networks"></a>Перекрестное подключение виртуальных сетей к локальным сетям

Мы можем подключить канал ExpressRoute к нескольким виртуальным сетям. См. раздел [ограничения подписки и службы][Subscription limits] для максимального числа виртуальных сетей, которые могут быть подключены к каналу ExpressRoute. 

Давайте подключим канал Fabrikam ExpressRoute к виртуальной сети подписки Contoso и аналогичному каналу Contoso ExpressRoute в виртуальную сеть подписки Fabrikam, чтобы обеспечить перекрестное подключение между виртуальными сетями и локальными сетями. Чтобы подключить виртуальную сеть к каналу ExpressRoute в другой подписке, необходимо создать и использовать авторизацию.  См. статью: [подключение виртуальной сети к каналу ExpressRoute][Connect-ER-VNet].

На следующем рисунке показана сетевая архитектура после настройки перекрестного подключения ExpressRoute к виртуальным сетям.

![Архитектура после перекрестного подключения которыми](./media/cross-network-connectivity/exr-x-connect.png)

В следующей таблице показана таблица маршрутов частного пиринга ExpressRoute компании Contoso Ltd. после перекрестного подключения виртуальных сетей к локальным сетям через ExpressRoute. Проверьте, что таблица маршрутов имеет маршруты, принадлежащие обеим виртуальным сетям.

![Таблица маршрутизации Contoso ExpressRoute после перекрестного подключения Екср и виртуальных сетей](./media/cross-network-connectivity/contosoexr-rt-xconnect.png)

В следующей таблице показана таблица маршрутов частного пиринга ExpressRoute компании Fabrikam Inc. после перекрестного подключения виртуальных сетей к локальным сетям через ExpressRoute. Проверьте, что таблица маршрутов имеет маршруты, принадлежащие обеим виртуальным сетям.

![Таблица маршрутов Fabrikam ExpressRoute после перекрестного подключения Екср и виртуальных сетей](./media/cross-network-connectivity/fabrikamexr-rt-xconnect.png)

В следующей таблице показаны маршруты, известные виртуальной машине подписки contoso. Обратите внимание на записи *шлюза виртуальной сети* таблицы. Виртуальная машина видит маршруты для локальных сетей.

![Маршруты виртуальных машин Contoso после перекрестного подключения Екср и виртуальных сетей](./media/cross-network-connectivity/contosovm-routes-xconnect.png)

В следующей таблице показаны маршруты, известные для виртуальной машины подписки Fabrikam. Обратите внимание на записи *шлюза виртуальной сети* таблицы. Виртуальная машина видит маршруты для локальных сетей.

![Маршруты виртуальных машин Fabrikam после перекрестного подключения Екср и виртуальных сетей](./media/cross-network-connectivity/fabrikamvm-routes-xconnect.png)

>[!NOTE]
>В подписках Fabrikam и/или Contoso также можно иметь периферийную виртуальных сетей в соответствующую виртуальную сеть концентратора (в схемах архитектуры в этой статье не проиллюстрирована конструкция HUB и лучевой конструкции). Перекрестные подключения между шлюзами виртуальной сети концентратора и ExpressRoute также разрешают обмен данными между налево и западными центрами и периферийными серверами.
>

## <a name="cross-connecting-on-premises-networks"></a>Перекрестное подключение локальных сетей

ExpressRoute Global Reach обеспечивает подключение между локальными сетями, подключенными к разным каналам ExpressRoute. Давайте настроим Global Reach между каналами Contoso и Fabrikam ExpressRoute. Так как каналы ExpressRoute находятся в разных подписках, необходимо создать и использовать авторизацию. Пошаговые инструкции см. в статье [настройка Global REACH ExpressRoute][Configure Global Reach] .

На следующем рисунке показана сетевая архитектура после настройки Global Reach.

![Архитектура после настройки Global Reach](./media/cross-network-connectivity/globalreach.png)

В следующей таблице показана таблица маршрутов частного пиринга ExpressRoute компании Contoso Ltd. После настройки Global Reach. Проверьте, что таблица маршрутов имеет маршруты, принадлежащие обеим локальным сетям. 

![Таблица маршрутов Contoso ExpressRoute после Global Reach](./media/cross-network-connectivity/contosoexr-rt-gr.png)

В следующей таблице показана таблица маршрутов частного пиринга ExpressRoute компании Fabrikam Inc. После настройки Global Reach. Проверьте, что таблица маршрутов имеет маршруты, принадлежащие обеим локальным сетям.

![Таблица маршрутов Fabrikam ExpressRoute после Global Reach]( ./media/cross-network-connectivity/fabrikamexr-rt-gr.png )

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные вопросы по виртуальной сети и пирингу виртуальных сетей см. в разделе [часто ЗАдаваемые вопросы о виртуальном сетевом][VNet-FAQ]уровне. Дополнительные вопросы об ExpressRoute и подключении к виртуальной сети см. в разделе [вопросы и ответы по expressroute][ER-FAQ] .

Global Reach размещается в стране или регионе по странам или регионам. Чтобы узнать, доступны ли Global Reach в нужных странах или регионах, см. раздел [ExpressRoute Global REACH][Global Reach].

<!--Link References-->
[Virtual network peering]: ../virtual-network/virtual-network-peering-overview.md
[connection]: ./expressroute-howto-linkvnet-portal-resource-manager.md
[Global Reach]: ./expressroute-global-reach.md
[Configure VNet peering]: ../virtual-network/create-peering-different-subscriptions.md
[Configure Global Reach]: ./expressroute-howto-set-global-reach.md
[Subscription limits]: ../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits
[Connect-ER-VNet]: ./expressroute-howto-linkvnet-portal-resource-manager.md
[ER-FAQ]: ./expressroute-faqs.md
[VNet-FAQ]: https://docs.microsoft.com/azure/virtual-network/virtual-networks-faq