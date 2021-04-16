---
title: Руководство по Связывание виртуальной сети с каналом ExpressRoute с помощью портала Azure
description: В этом руководстве демонстрируется, как создать подключение для связи виртуальной сети с каналом Azure ExpressRoute, используя портал Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/15/2020
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: a9bfb4beea2c2bec1b819d228215cfff65e37fe4
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106110263"
---
# <a name="tutorial-connect-a-virtual-network-to-an-expressroute-circuit-using-the-portal"></a>Руководство по Подключение виртуальной сети к каналу ExpressRoute с помощью портала

> [!div class="op_single_selector"]
> * [Портал Azure](expressroute-howto-linkvnet-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-linkvnet-arm.md)
> * [Azure CLI](howto-linkvnet-cli.md)
> * [Видео — портал Azure](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit)
> * [PowerShell (классическая модель)](expressroute-howto-linkvnet-classic.md)
> 

Это руководство поможет вам создать подключение для связи виртуальной сети с каналом Azure ExpressRoute, используя портал Azure. Виртуальные сети, подключаемые к каналу Azure ExpressRoute, могут принадлежать к одной или разным подпискам.

В этом руководстве описано следующее:
> [!div class="checklist"]
> - подключение к каналу виртуальной сети в одной подписке;
> - подключение к каналу виртуальной сети в разных подписках;
> - удаление подключения между виртуальной сетью и каналом ExpressRoute.

## <a name="prerequisites"></a>Предварительные требования

* Прежде чем приступить к настройке, изучите [предварительные требования](expressroute-prerequisites.md), [требования к маршрутизации](expressroute-routing.md) и [рабочие процессы](expressroute-workflows.md).

* Вам потребуется активный канал ExpressRoute.
  * Следуйте инструкциям, чтобы [создать канал ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md) и включить его на стороне поставщика услуг подключения.
  * Убедитесь, что для вашего канала настроен частный пиринг Azure. Инструкции по пирингу и маршрутизации см. в руководстве по [созданию и изменению пиринга для канала ExpressRoute](expressroute-howto-routing-portal-resource-manager.md).
  * Для создания сквозного подключения обязательно настройте частный пиринг Azure, а также пиринг BGP между своей сетью и сетью Майкрософт.
  * Вам необходимо создать и полностью подготовить виртуальную сеть и шлюз виртуальной сети. Следуйте инструкциям по [созданию шлюза виртуальной сети для ExpressRoute](expressroute-howto-add-gateway-resource-manager.md). Для ExpressRoute используется шлюз виртуальной сети типа ExpressRoute, а не VPN.

* К стандартному каналу ExpressRoute можно подключить не более 10 виртуальных сетей. Если используется стандартный канал ExpressRoute, все виртуальные сети должны находиться в одном геополитическом регионе.

* Отдельную виртуальную сеть можно связать максимум с 16 каналами ExpressRoute. Для создания объекта подключения для каждого канала ExpressRoute, к которому вы подключаетесь, используйте процесс, описанный ниже. Каналы ExpressRoute могут быть размещены в той же подписке, в других подписках или и там, и там.

* Если вы включите надстройку ExpressRoute Premium, вы сможете подключить к каналу ExpressRoute виртуальные сети из другого геополитического региона. Надстройка Premium также позволяет подключить к каналу ExpressRoute более 10 виртуальных сетей (в зависимости от выбранной пропускной способности). Дополнительную информацию о надстройке Premium см. в разделе [Вопросы и ответы](expressroute-faqs.md).

* Вы можете [просмотреть видео](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit), прежде чем начать, чтобы лучше понять выполняемые действия.

## <a name="connect-a-vnet-to-a-circuit---same-subscription"></a>Подключение виртуальной сети к каналу в одной и той же подписке

> [!NOTE]
> Если пиринги были настроены поставщиком уровня 3, сведения о конфигурации BGP отображаться не будут. Если канал находится в подготовленном состоянии, то вы сможете создавать подключения.
>

### <a name="to-create-a-connection"></a>Создание подключения

1. Убедитесь, что канал ExpressRoute и частный пиринг Azure настроены. Выполните инструкции из статьи [Создание канала ExpressRoute](expressroute-howto-circuit-arm.md) и [Создание и изменение пиринга для канала ExpressRoute](expressroute-howto-routing-arm.md). Ваш канал ExpressRoute должен выглядеть, как показано на рисунке ниже.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/express-route-circuit.png" alt-text="Снимок экрана канала ExpressRoute":::

1. Теперь можно приступить к подготовке подключения для связи шлюза виртуальной сети с каналом ExpressRoute. Щелкните **Подключение** > **Добавить**, чтобы открыть страницу **Добавление подключения**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/add-connection.png" alt-text="Снимок экрана добавления подключения":::

1. Введите имя для подключения и нажмите кнопку **Next: Settings >** (Далее: параметры >).

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/create-connection-basic.png" alt-text="Основная страница создания подключения":::

1. Выберите шлюз, принадлежащий к виртуальной сети, которую необходимо связать с каналом, и выберите **Просмотр и создание**. После завершения проверки щелкните **Создать**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/create-connection-settings.png" alt-text="Страница параметров &quot;Создание подключения&quot;":::

1. После успешной настройки подключения в вашем объекте подключения будут показаны сведения о подключении.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-object.png" alt-text="Снимок экрана объекта подключения":::

## <a name="connect-a-vnet-to-a-circuit---different-subscription"></a>Подключение виртуальной сети к каналу в разных подписках

Канал ExpressRoute может совместно использоваться несколькими подписками. На рисунке ниже схематично показан способ совместного использования каналов ExpressRoute несколькими подписками.

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/cross-subscription.png" alt-text="Подключение между подписками":::

Каждое маленькое облако внутри большого облака представляет подписки, принадлежащие различным подразделениям одной организации. Любое подразделение в организации может использовать свою собственную подписку для развертывания служб. Кроме того, подразделения могут совместно использовать один канал ExpressRoute для подключения к локальной сети. Владельцем канала ExpressRoute может выступать одно подразделение (в данном примере — ИТ-подразделение). Другие подписки в организации также могут использовать канал ExpressRoute.

  > [!NOTE]
  > Плата за подключение выделенного канала ExpressRoute и использование полосы пропускания будет взиматься с владельца выделенного канала. Полоса пропускания распределяется между всеми виртуальными сетями.
  >

### <a name="administration---about-circuit-owners-and-circuit-users"></a>Администрирование: о владельцах и пользователях канала

Владельцем канала является уполномоченный опытный пользователь ресурса канала ExpressRoute. Владелец канала может создавать разрешения, которые могут быть активированы пользователями канала. Пользователи канала являются владельцами шлюзов виртуальных сетей, не включенных в подписку, к которой относится канал ExpressRoute. Пользователи канала могут активировать разрешения (по одному разрешению для каждой виртуальной сети).

Владелец канала имеет право изменить или отменить авторизацию в любое время. Отмена разрешения приводит к удалению всех связывающих подключений из подписки, доступ к которой был отменен.

### <a name="circuit-owner-operations"></a>Действия владельца канала

**Создание разрешения на подключение**

Владелец канала создает разрешение, в результате чего создается ключ авторизации, с помощью которого пользователь канала сможет подключить шлюзы виртуальной сети к каналу ExpressRoute. Разрешение действительно только для одного подключения.

> [!NOTE]
> Каждое соединение требует отдельной авторизации.
>

1. На странице ExpressRoute выберите элемент **Разрешения**, введите **имя** разрешения и щелкните команду **Сохранить**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/authorization.png" alt-text="Авторизации":::

2. После сохранения конфигурации скопируйте **идентификатор ресурса** и **ключ авторизации**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/authorization-key.png" alt-text="Ключ авторизации":::

**Удаление разрешения на подключение**

Подключение можно удалить, щелкнув значок **Удалить** возле ключа авторизации подключения.

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/delete-authorization-key.png" alt-text="Удаление ключа авторизации":::

Если вы хотите удалить подключение, но оставить ключ авторизации, можно удалить подключение на странице подключений канала.

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/delete-connection-owning-circuit.png" alt-text="Удаление подключения на странице соответствующего канала":::

### <a name="circuit-user-operations"></a>Действия пользователя канала

Пользователь канала должен получить от владельца канала идентификатор ресурса и ключ авторизации.

**Активация разрешения на подключение**

1. Нажмите кнопку **+ Создать ресурс**. Выполните поиск по запросу **Подключение** и выберите **Создать.**

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/create-new-resources.png" alt-text="Создание новых ресурсов":::

1. Убедитесь, что выбран *тип подключения* **ExpressRoute**. На странице "Основные сведения" выберите *группу ресурсов* и *расположение*, а затем щелкните **ОК**.

    > [!NOTE]
    > Расположение *должно* совпадать с расположением шлюза виртуальной сети, для которого создается подключение.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-basics.png" alt-text="Страница &quot;Основные&quot;":::

1. На странице **Параметры** выберите *Шлюз виртуальной сети* и установите флажок **Активировать авторизацию**. Введите значения *Ключ авторизации* и *Peer circuit URI* (URI однорангового канала) и присвойте подключению имя. Щелкните **ОК**. 
 
    > [!NOTE]
    > *URI одноранговой цепи* — это идентификатор ресурса для канала ExpressRoute (который можно найти в области "Параметры свойств" канала ExpressRoute).

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-settings.png" alt-text="Страница &quot;Параметры&quot;":::

1. Просмотрите сведения на странице **Сводка** и нажмите кнопку **ОК**.

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-summary.png" alt-text="Страница &quot;Сводка&quot;":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Можно удалить подключение и разорвать связь виртуальной сети с каналом ExpressRoute, выбрав значок **Удалить** на странице соответствующего подключения.

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/delete-connection.png" alt-text="Удаление подключения":::

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как подключить виртуальную сеть к каналу в одной или разных подписках. Дополнительные сведения о шлюзе ExpressRoute см. в следующих статьях: 

> [!div class="nextstepaction"]
> [Сведения о шлюзах виртуальных сетей ExpressRoute](expressroute-about-virtual-network-gateways.md)
