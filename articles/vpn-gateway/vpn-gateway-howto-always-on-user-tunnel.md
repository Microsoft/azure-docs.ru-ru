---
title: Настройка туннеля VPN-пользователя Always-On
titleSuffix: Azure VPN Gateway
description: В этой статье описывается, как настроить туннель VPN пользователя Always On для VPN-шлюза.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: cherylmc
ms.openlocfilehash: 99d21222a62ed6b27a6a1b2a73b704f4cb26457b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89435802"
---
# <a name="configure-an-always-on-vpn-user-tunnel"></a>Настройка туннеля Always On VPN для пользователя

[!INCLUDE [intro](../../includes/vpn-gateway-vwan-always-on-intro.md)]

## <a name="configure-the-gateway"></a>Настройка шлюза

 Используйте инструкции из статьи [Настройка VPN-подключения типа "точка — сеть](vpn-gateway-howto-point-to-site-resource-manager-portal.md) ", чтобы настроить VPN-шлюз для использования проверки подлинности IKEv2 и на основе сертификата.

## <a name="configure-a-user-tunnel"></a>Настройка пользовательского туннеля

[!INCLUDE [user configuration](../../includes/vpn-gateway-vwan-always-on-user.md)]

## <a name="to-remove-a-profile"></a>Удаление профиля

Чтобы удалить профиль, выполните следующие действия.

1. Выполните следующую команду.

   ```powershell
   C:\> Remove-VpnConnection UserTest  
   ```

1. Отключите подключение и снимите флажок **подключиться автоматически** .

   ![Очистка](./media/vpn-gateway-howto-always-on-user-tunnel/disconnect.jpg)

## <a name="next-steps"></a>Дальнейшие действия

Сведения об устранении проблем с подключением, которые могут возникнуть, см. в статье [проблемы с подключением "точка — сеть](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md)" в Azure.
