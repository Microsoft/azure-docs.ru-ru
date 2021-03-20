---
title: Совместное использование службы частных ссылок в виртуальной глобальной сети
titleSuffix: Azure Virtual WAN
description: Действия по настройке частной связи в виртуальной глобальной сети
services: virtual-wan
author: erjosito
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/22/2020
ms.author: jomore
ms.custom: fasttrack-new
ms.openlocfilehash: cc8e7314c941035207ecf809a9d85ef46bd58379
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92913761"
---
# <a name="use-private-link-in-virtual-wan"></a>Использование частной ссылки в виртуальной глобальной сети

[Частная ссылка Azure](../private-link/private-link-overview.md) — это технология, которая позволяет подключать предложения "платформа как услуга" с использованием частного IP-адреса, предоставляя [частные конечные точки](../private-link/private-endpoint-overview.md). Виртуальная глобальная сеть Azure позволяет развернуть закрытую конечную точку в одной из виртуальных сетей, подключенных к любому виртуальному концентратору. Это обеспечивает подключение к любой виртуальной глобальной сети или ветви, подключенной к той же виртуальной сети.

## <a name="before-you-begin"></a>Перед началом

В этом разделе предполагается, что вы уже развернули виртуальную сеть WAN с одним или несколькими концентраторами, а также по крайней мере две виртуальные сети, подключенные к виртуальной глобальной сети.

Чтобы создать виртуальную глобальную сеть и новый концентратор, выполните действия, описанные в следующих статьях.

* [Создание виртуальной глобальной сети](virtual-wan-site-to-site-portal.md#openvwan)
* [Создание концентратора](virtual-wan-site-to-site-portal.md#hub)
* [Подключение виртуальной сети к концентратору.](virtual-wan-site-to-site-portal.md#hub)

## <a name="create-a-private-link-endpoint"></a><a name="endpoint"></a>Создание конечной точки частной ссылки

Вы можете создать конечную точку закрытой ссылки для многих разных служб. В этом примере мы будем использовать базу данных SQL Azure. Дополнительные сведения о создании частной конечной точки для базы данных SQL Azure см. в [кратком руководстве по созданию частной конечной точки с помощью портал Azure](../private-link/create-private-endpoint-portal.md). На следующем рисунке показана конфигурация сети базы данных SQL Azure.

:::image type="content" source="./media/howto-private-link/create-private-link.png" alt-text="создать закрытую ссылку" lightbox="./media/howto-private-link/create-private-link.png":::

После создания базы данных SQL Azure вы можете проверить IP-адрес частной конечной точки, просмотрев частные конечные точки:

:::image type="content" source="./media/howto-private-link/endpoints.png" alt-text="частные конечные точки" lightbox="./media/howto-private-link/endpoints.png":::

Щелкнув созданную закрытую конечную точку, вы должны увидеть его частный IP-адрес, а также полное доменное имя (FQDN). Обратите внимание, что частная конечная точка имеет IP-адрес в диапазоне виртуальной сети, в которой он был развернут (10.1.3.0/24):

:::image type="content" source="./media/howto-private-link/sql-endpoint.png" alt-text="Конечная точка SQL" lightbox="./media/howto-private-link/sql-endpoint.png":::

## <a name="verify-connectivity-from-the-same-vnet"></a><a name="connectivity"></a>Проверка подключения из той же виртуальной сети

В этом примере мы проверяем подключение к базе данных SQL Azure с помощью виртуальной машины Ubuntu с установленными инструментами MS SQL. Первым шагом является проверка того, что разрешение DNS работает, а полное доменное имя базы данных SQL Azure разрешается в частный IP-адрес в той же виртуальной сети, в которой развернута частная конечная точка (10.1.3.0/24):

```bash
$ nslookup wantest.database.windows.net
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
wantest.database.windows.net    canonical name = wantest.privatelink.database.windows.net.
Name:   wantest.privatelink.database.windows.net
Address: 10.1.3.228
```

Как видно из предыдущих выходных данных, полное доменное имя `wantest.database.windows.net` сопоставляется с `wantest.privatelink.database.windows.net` , что частная зона DNS, созданная на частной конечной точке, будет разрешаться в частный IP-адрес `10.1.3.228` . При поиске в частной зоне DNS будет подтверждено наличие записи A для частной конечной точки, сопоставленной с частным IP-адресом:

:::image type="content" source="./media/howto-private-link/dns-zone.png" alt-text="Зона DNS" lightbox="./media/howto-private-link/dns-zone.png":::

После проверки правильного разрешения DNS можно попытаться подключиться к базе данных:

```bash
$ query="SELECT CONVERT(char(15), CONNECTIONPROPERTY('client_net_address'));"
$ sqlcmd -S wantest.database.windows.net -U $username -P $password -Q "$query"

10.1.3.75
```

Как вы видите, мы используем Специальный SQL-запрос, который предоставляет нам адрес источника, который SQL Server видит от клиента. В этом случае сервер видит Клиент своим частным IP-адресом ( `10.1.3.75` ), что означает, что трафик поступает из виртуальной сети прямо в конечную точку закрытой.

Обратите внимание, что необходимо задать переменные `username` и `password` в соответствии с учетными данными, определенными в базе данных SQL Azure, чтобы сделать примеры в этом пошаговом окне.

## <a name="connect-from-a-different-vnet"></a><a name="vnet"></a>Подключение из другой виртуальной сети

Теперь, когда одна виртуальная сеть в виртуальной сети Azure имеет подключение к частной конечной точке, все остальные виртуальных сетей и ветви, подключенные к виртуальной глобальной сети, могут также получить доступ к ней. Необходимо предоставить возможность подключения с помощью любой модели, поддерживаемой виртуальной глобальной сетью Azure, такой как [любой сценарий](scenario-any-to-any.md) или [Виртуальная сеть общих служб](scenario-shared-services-vnet.md), для именования двух примеров.

После подключения виртуальной сети или ветви к виртуальной сети, в которой развернута частная конечная точка, необходимо настроить разрешение DNS.

* При подключении к частной конечной точке из виртуальной сети можно использовать ту же частную зону, которая была создана в базе данных SQL Azure.
* При подключении к частной конечной точке из ветви (VPN типа "сеть — сеть", VPN-подключение типа "точка — сеть" или ExpressRoute) необходимо использовать локальное разрешение DNS.

В этом примере мы будем подключаться из другой виртуальной сети, поэтому сначала мы подключим частную зону DNS к новой виртуальной сети, чтобы ее рабочие нагрузки могли разрешить полное доменное имя базы данных SQL Azure в частный IP-адрес. Это делается путем связывания частной зоны DNS с новой виртуальной сетью.

:::image type="content" source="./media/howto-private-link/dns-link.png" alt-text="Ссылка на DNS" lightbox="./media/howto-private-link/dns-link.png":::

Теперь любая виртуальная машина в подключенной виртуальной сети должна правильно разрешить полное доменное имя базы данных SQL Azure в частный IP-адрес частной ссылки:

```bash
$ nslookup wantest.database.windows.net
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
wantest.database.windows.net    canonical name = wantest.privatelink.database.windows.net.
Name:   wantest.privatelink.database.windows.net
Address: 10.1.3.228
```

Чтобы убедиться, что эта виртуальная сеть (10.1.1.0/24) имеет подключение к исходной виртуальной сети, в которой была настроена частная конечная точка (10.1.3.0/24), вы можете проверить действующую таблицу маршрутов на любой виртуальной машине в виртуальной сети:

:::image type="content" source="./media/howto-private-link/effective-routes.png" alt-text="действующие маршруты" lightbox="./media/howto-private-link/effective-routes.png":::

Как видите, существует маршрут, указывающий на виртуальную сеть 10.1.3.0/24, которая была введена шлюзами виртуальной глобальной сети Azure. Теперь мы можем проверить возможность подключения к базе данных:

```bash
$ query="SELECT CONVERT(char(15), CONNECTIONPROPERTY('client_net_address'));"
$ sqlcmd -S wantest.database.windows.net -U $username -P $password -Q "$query"

10.1.1.75
```

В этом примере мы увидели, как создание частной конечной точки в одном из виртуальных сетей, подключенных к виртуальной глобальной сети, обеспечивает подключение к остальной части виртуальных сетей и ветвей в виртуальной глобальной сети.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о виртуальной глобальной сети см. в статье, содержащей [Часто задаваемые вопросы](virtual-wan-faq.md) о ней.
