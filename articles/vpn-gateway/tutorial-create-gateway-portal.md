---
title: Руководство по созданию VPN-шлюза и управлению им Портал Azure
description: Из этого учебника вы узнаете, как создать, развернуть VPN-шлюз Azure и обеспечить управление им с помощью портала
author: cherylmc
ms.author: cherylmc
ms.service: vpn-gateway
ms.topic: tutorial
ms.date: 12/01/2020
ms.openlocfilehash: 820482a268af038737557c517ccda086cd65d943
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98880564"
---
# <a name="tutorial-create-and-manage-a-vpn-gateway-using-azure-portal"></a>Руководство по созданию VPN-шлюза и управление им с помощью портала Azure

VPN-шлюзы Azure предоставляют распределенное подключение между локальными сетями клиента и Azure. В этом руководстве приведены основные этапы развертывания VPN-шлюза Azure, такие как создание VPN-шлюза и управление им. Вы также можете создать шлюз с помощью [Azure CLI](create-routebased-vpn-gateway-cli.md) или [Azure PowerShell](create-routebased-vpn-gateway-powershell.md).

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание виртуальной сети
> * Создание VPN-шлюза
> * Просмотр общедоступного IP-адреса шлюза
> * Изменение размера VPN-шлюза (изменение размера SKU)
> * Сброс VPN-шлюза

На следующей схеме показаны виртуальная сеть и VPN-шлюз, созданные в рамках этого руководства.

:::image type="content" source="./media/tutorial-create-gateway-portal/gateway-diagram.png" alt-text="Схема виртуальной сети и VPN-шлюза":::

## <a name="prerequisites"></a>Предварительные требования

Учетная запись Azure с активной подпиской. Если у вас ее нет, [создайте подписку бесплатно](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

## <a name="create-a-virtual-network"></a><a name="CreatVNet"></a>Создание виртуальной сети

Создайте виртуальную сеть, используя следующие значения:

* **Группа ресурсов**. TestRG1
* **Имя.** VNet1
* **Регион**: Восточная часть США (США)
* **Диапазон IPv4-адресов**: 10.1.0.0/16
* **Имя подсети**: FrontEnd
* **Диапазон адресов подсети**: 10.1.0.0/24

[!INCLUDE [Create a virtual network](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="create-a-vpn-gateway"></a><a name="VNetGateway"></a>Создание VPN-шлюза

На этом шаге вы создадите шлюз для своей виртуальной сети. Создание шлюза часто занимает 45 минут и более, в зависимости от выбранного SKU шлюза.

Создайте шлюз виртуальной сети, используя следующие значения:

* **Имя.** VNet1GW
* **Регион**: Восточная часть США
* **Тип шлюза**: Виртуальная частная сеть
* **Тип VPN**: на основе маршрута
* **SKU**: VpnGw1
* **Поколение**. Поколение1
* **Виртуальная сеть**. VNet1
* **Диапазон адресов подсети шлюза**: 10.1.255.0/27
* **Общедоступный IP-адрес**. Создание
* **Имя общедоступного IP-адреса:** VNet1GWpip
* **Включить режим "активный — активный"** . Выключено
* **Настройка BGP**. Выключено

[!INCLUDE [Create a vpn gateway](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

[!INCLUDE [NSG warning](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="view-the-public-ip-address"></a><a name="view"></a>Просмотр общедоступного IP-адреса

Общедоступный IP-адрес шлюза можно просмотреть на странице **Обзор** для шлюза.

:::image type="content" source="./media/tutorial-create-gateway-portal/address.png" alt-text="Страница &quot;Обзор&quot;":::

Чтобы просмотреть дополнительные сведения об объекте общедоступного IP-адреса, щелкните ссылку имени/IP-адреса возле параметра **Общедоступный IP-адрес**.

## <a name="resize-a-gateway-sku"></a><a name="resize"></a>Изменение размера SKU шлюза

Существуют определенные правила, касающиеся изменения размера и изменения SKU шлюза. В этом разделе показано, как изменить размер SKU. Дополнительные сведения см. в разделе об [изменении размера и изменении номеров SKU](vpn-gateway-about-vpn-gateway-settings.md#resizechange).

[!INCLUDE [resize a gateway](../../includes/vpn-gateway-resize-gw-portal-include.md)]

## <a name="reset-a-gateway"></a><a name="reset"></a>Сброс параметров шлюза

[!INCLUDE [reset a gateway](../../includes/vpn-gateway-reset-gw-portal-include.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение в дальнейшем или переходить к следующему учебнику, удалите эти ресурсы, выполнив следующие действия.

1. Введите имя группы ресурсов в поле **Поиск** в верхней части портала и выберите ее в результатах поиска.

1. Выберите **Удалить группу ресурсов**.

1. В поле **TYPE THE RESOURCE GROUP NAME** (ВВЕДИТЕ ИМЯ ГРУППЫ РЕСУРСОВ) введите имя и выберите **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

После создания VPN-шлюза можно настроить подключения. С помощью инструкций в приведенных ниже статьях можно создать несколько наиболее распространенных конфигураций:

> [!div class="nextstepaction"]
> [VPN-подключения типа "сеть —сеть"](./tutorial-site-to-site-portal.md)

> [!div class="nextstepaction"]
> [VPN-подключения типа "точка — сеть"](vpn-gateway-howto-point-to-site-resource-manager-portal.md)