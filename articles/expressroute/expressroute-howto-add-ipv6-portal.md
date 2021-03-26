---
title: 'Azure ExpressRoute: Добавление поддержки IPv6 с помощью портал Azure'
description: Узнайте, как добавить поддержку IPv6 для подключения к развертываниям Azure с помощью портал Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 03/09/2021
ms.author: duau
ms.openlocfilehash: 7f5afc05a8d03d33366a2f76318bcf5e039d4d30
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105561667"
---
# <a name="add-ipv6-support-for-private-peering-using-the-azure-portal-preview"></a>Добавление поддержки IPv6 для частного пиринга с помощью портал Azure (Предварительная версия)

В этой статье описывается, как добавить поддержку IPv6 для подключения через ExpressRoute к ресурсам в Azure с помощью портал Azure. 

> [!Note]
> Сейчас эта функция доступна для предварительной версии в [регионах Azure с зоны доступности](../availability-zones/az-region.md#azure-regions-with-availability-zones). Таким образом, канал ExpressRoute можно создать с помощью любого расположения пиринга, но развертываемые на основе IPv6 развертывания должны находиться в регионе с Зоны доступности.

## <a name="register-for-public-preview"></a>Зарегистрируйтесь для просмотра общедоступной предварительной версии
Перед добавлением поддержки IPv6 сначала необходимо зарегистрировать подписку. Для регистрации выполните следующие команды с помощью Azure PowerShell:

1.  Войдите в Azure и выберите подписку. Выполните следующие команды для подписки, содержащей канал ExpressRoute, и подписки, содержащей развертывания Azure (если они отличаются).

    ```azurepowershell-interactive
    Connect-AzAccount 

    Select-AzSubscription -Subscription "<SubscriptionID or SubscriptionName>"
    ```

1. Зарегистрируйте подписку для общедоступной предварительной версии с помощью следующей команды:
    ```azurepowershell-interactive
    Register-AzProviderFeature -FeatureName AllowIpv6PrivatePeering -ProviderNamespace Microsoft.Network
    ```

Ваш запрос будет утвержден командой ExpressRoute в течение 2-3 рабочих дней.

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

В браузере перейдите к [портал Azure](https://portal.azure.com), а затем выполните вход с помощью учетной записи Azure.

## <a name="add-ipv6-private-peering-to-your-expressroute-circuit"></a>Добавление частного пиринга IPv6 в канал ExpressRoute

1. [Создайте канал ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md) или перейдите к существующему каналу, который необходимо изменить.

    :::image type="content" source="./media/expressroute-howto-add-ipv6-portal/navigate-to-circuit.png" alt-text="Снимок экрана со списком каналов ExpressRoute.":::

1. Выберите конфигурацию **частного пиринга Azure** .

    :::image type="content" source="./media/expressroute-howto-add-ipv6-portal/navigate-to-peering.png" alt-text="Снимок экрана со страницей обзора ExpressRoute.":::

1. Добавьте частный пиринг IPv6 в существующую конфигурацию частного пиринга IPv4, выбрав "оба" для **подсетей**, или включите частный пиринг IPv6 в новом канале, выбрав "IPv6". Предоставьте пару из/126 подсетей IPv6, которыми вы владеете, для основной ссылки и дополнительных ссылок. В каждой из этих подсетей вы назначаете маршрутизатору первый подходящий IP-адрес, так как корпорация Майкрософт использует для его маршрутизатора второй пригодный IP-протокол. **Сохраните** конфигурацию пиринга после указания всех параметров.

    :::image type="content" source="./media/expressroute-howto-add-ipv6-portal/add-ipv6-peering.png" alt-text="Снимок экрана: Добавление IPv6 на странице частного пиринга.":::

1. После успешного завершения настройки вы увидите нечто вроде следующего.

    :::image type="content" source="./media/expressroute-howto-add-ipv6-portal/view-ipv6-peering.png" alt-text="Снимок экрана IPv6, настроенного для частного пиринга.":::

## <a name="update-your-connection-to-an-existing-virtual-network"></a>Обновление подключения к существующей виртуальной сети

Выполните следующие действия, если у вас есть среда ресурсов Azure в регионе с Зоны доступности, с которыми вы хотите использовать частный пиринг IPv6.

1. Перейдите к виртуальной сети, к которой подключен канал ExpressRoute.

    :::image type="content" source="./media/expressroute-howto-add-ipv6-portal/navigate-to-vnet.png" alt-text="Снимок экрана со списком виртуальных сетей.":::

1. Перейдите на вкладку **адресное пространство** и добавьте адресное пространство IPv6 в виртуальную сеть. **Сохраните** адресное пространство.

    :::image type="content" source="./media/expressroute-howto-add-ipv6-portal/add-ipv6-space.png" alt-text="Снимок экрана: Добавление пространства адресов IPv6 в виртуальную сеть.":::

1. Перейдите на вкладку **подсети** и выберите **GatewaySubnet**. Установите флажок **Добавить адресное пространство IPv6** и укажите адресное пространство IPv6 для вашей подсети. Подсеть IPv6 шлюза должна иметь значение/64 или больше. **Сохраните** конфигурацию после указания всех параметров.

    :::image type="content" source="./media/expressroute-howto-add-ipv6-portal/add-ipv6-gateway-space.png" alt-text="Снимок экрана: Добавление пространства адресов IPv6 в подсеть.":::

1. Если у вас есть шлюз, избыточный в зоне, перейдите к своему шлюзу ExpressRoute и обновите ресурс, чтобы включить IPv6-подключение. В противном случае [Создайте шлюз виртуальной сети](expressroute-howto-add-gateway-portal-resource-manager.md) с помощью SKU, избыточного в виде зоны (ErGw1AZ, ErGw2AZ, ErGw3AZ). Если вы планируете использовать Фастпас, используйте ErGw3AZ.

    :::image type="content" source="./media/expressroute-howto-add-ipv6-portal/refresh-gateway.png" alt-text="Снимок экрана обновления шлюза.":::

## <a name="create-a-connection-to-a-new-virtual-network"></a>Создание подключения к новой виртуальной сети

Если вы планируете подключиться к новому набору ресурсов Azure в регионе с Зоны доступности помощью частного пиринга IPv6, выполните следующие действия.

1. Создайте виртуальную сеть с двумя стеками с адресным пространством IPv4 и IPv6. Дополнительные сведения см. [в разделе Создание виртуальной сети](../virtual-network/quick-create-portal.md#create-a-virtual-network).

1. [Создайте подсеть шлюза с двойным стеком](expressroute-howto-add-gateway-portal-resource-manager.md#create-the-gateway-subnet).

1. [Создайте шлюз виртуальной сети](expressroute-howto-add-gateway-portal-resource-manager.md#create-the-virtual-network-gateway) с помощью SKU, избыточного в виде зоны (ErGw1AZ, ErGw2AZ, ErGw3AZ). Если вы планируете использовать Фастпас, используйте ErGw3AZ (Обратите внимание, что этот параметр доступен только для каналов с помощью ExpressRoute Direct).

1. [Свяжите виртуальную сеть с каналом ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md).

## <a name="limitations"></a>Ограничения
Хотя поддержка IPv6 доступна для подключений к развертываниям в регионах с Зоны доступности, она не поддерживает следующие варианты использования:

* Подключения к развертываниям в Azure с помощью SKU шлюза, отличного от AZ ExpressRoute
* Подключения к развертываниям в регионах, не относящихся к AZ
* Global Reach соединений между каналами ExpressRoute
* Использование ExpressRoute с виртуальной глобальной сетью
* Фастпас с прямыми каналами, отличными от ExpressRoute
* Фастпас с каналами в следующих расположениях пиринга: Дубаи
* Сосуществование с VPN-шлюзом

## <a name="next-steps"></a>Дальнейшие действия

Сведения об устранении неполадок ExpressRoute см. в следующих статьях:

* [Проверка подключения ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
* [Устранение проблем с производительностью сети](expressroute-troubleshooting-network-performance.md)