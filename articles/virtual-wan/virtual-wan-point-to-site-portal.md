---
title: Руководство по созданию подключения типа "точка — сеть" с помощью Виртуальной глобальной сети Azure
description: В руководстве показано, как создать VPN-подключение типа "точка — сеть" в Azure с помощью Виртуальной глобальной сети Azure.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: tutorial
ms.date: 03/05/2021
ms.author: cherylmc
ms.openlocfilehash: db7345906605ce117f0d57deb80f9d26ebf84179
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102430538"
---
# <a name="tutorial-create-a-user-vpn-connection-using-azure-virtual-wan"></a>Руководство по созданию пользовательского соединения VPN с помощью Виртуальной глобальной сети Azure

В этом руководстве показано, как создать подключение к ресурсам в Azure через VPN-соединение IPsec/IKE (IKEv2) или OpenVPN с помощью Виртуальной глобальной сети. Этот тип подключения требует, чтобы VPN-клиент был настроен на клиентском компьютере. Дополнительные сведения о Виртуальной глобальной сети см. в [этой статье](virtual-wan-about.md).

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание виртуальной глобальной сети
> * Создание конфигурации подключения "точка — сеть"
> * Создание виртуального концентратора
> * Выбор пулов адресов клиента
> * Указание DNS-серверов
> * Создание пакета конфигурации профиля VPN-клиента
> * Настройка VPN-клиентов
> * Просмотр виртуальной глобальной сети

![Схема Виртуальной глобальной сети](./media/virtual-wan-about/virtualwanp2s.png)

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [Before beginning](../../includes/virtual-wan-before-include.md)]

## <a name="create-a-virtual-wan"></a><a name="wan"></a>Создание виртуальной глобальной сети

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-create-vwan-include.md)]

## <a name="create-a-p2s-configuration"></a><a name="p2sconfig"></a>Создание конфигурации подключения типа "точка — сеть"

Конфигурация подключения типа "точка — сеть" (P2S) определяет параметры для подключения удаленных клиентов.

[!INCLUDE [Create P2S configuration](../../includes/virtual-wan-p2s-configuration-include.md)]

## <a name="create-virtual-hub-and-gateway"></a><a name="hub"></a>Создание виртуального концентратора и шлюза

[!INCLUDE [Create hub](../../includes/virtual-wan-p2s-hub-include.md)]

## <a name="choose-p2s-client-address-pools"></a><a name="chooseclientpools"></a> Выбор пулов адресов клиента для подключений "точка — сеть"

[!INCLUDE [Choose pools](../../includes/virtual-wan-allocating-p2s-pools.md)]

## <a name="specify-dns-server"></a><a name="dns"></a>Указание DNS-сервера

Эти параметры можно настроить при создании концентратора или изменить позже. Для этого найдите нужный виртуальный концентратор. В разделе **VPN пользователя (точка — сеть)** щелкните **Настроить** и введите IP-адреса DNS-серверов в текстовые поля **Пользовательские DNS-серверы**. Можно указать до 5 DNS-серверов.

   :::image type="content" source="media/virtual-wan-point-to-site-portal/custom-dns.png" alt-text="Пользовательские DNS-серверы" lightbox="media/virtual-wan-point-to-site-portal/custom-dns-expand.png":::

## <a name="generate-vpn-client-profile-package"></a><a name="download"></a>Создание пакета профиля VPN-клиента

Создайте и скачайте пакет профиля VPN-клиента для настройки VPN-клиентов.

[!INCLUDE [Download profile](../../includes/virtual-wan-p2s-download-profile-include.md)]

## <a name="configure-vpn-clients"></a><a name="configure-client"></a>Настройка VPN-клиентов

Используйте загруженный пакет профиля для настройки VPN-клиентов удаленного доступа. Процедуры для каждой операционной системы отличаются, поэтому следуйте соответствующим инструкциям.
Завершив настройку клиента, можно установить подключение.

[!INCLUDE [Configure clients](../../includes/virtual-wan-p2s-configure-clients-include.md)]

## <a name="view-your-virtual-wan"></a><a name="viewwan"></a>Просмотр виртуальной глобальной сети

1. Перейдите к виртуальной глобальной сети.
1. На странице **Обзор** каждая точка на карте представляет собой концентратор.
1. В разделе **Концентраторы и подключения** можно просмотреть сведения о состоянии концентратора, сайте, регионе, состоянии VPN-подключения, а также количестве принятых и переданных байтов.

## <a name="clean-up-resources"></a><a name="cleanup"></a>Очистка ресурсов

Если созданные ресурсы вам больше не нужны, удалите их. Некоторые ресурсы Виртуальной глобальной сети необходимо удалять в определенном порядке с учетом зависимостей. Удаление может занять около 30 минут.

[!INCLUDE [Delete resources](../../includes/virtual-wan-resource-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Виртуальной глобальной сети см. в следующей статье:

> [!div class="nextstepaction"]
> * [Вопросы и ответы о Виртуальной глобальной сети](virtual-wan-faq.md)
