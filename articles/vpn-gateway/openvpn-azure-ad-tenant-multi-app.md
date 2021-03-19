---
title: 'VPN-шлюз: клиент Azure AD для разных групп пользователей: проверка подлинности Azure AD'
description: VPN-подключение P2S можно использовать для подключения к виртуальной сети с помощью проверки подлинности Azure AD.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 10/07/2020
ms.author: cherylmc
ms.openlocfilehash: 1305ca603aef63dafcc7b055d55e3f0fe281f4fc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91819669"
---
# <a name="create-an-active-directory-ad-tenant-for-p2s-openvpn-protocol-connections"></a>Создание клиента Active Directory (AD) для подключений по протоколу Опенвпн P2S

При подключении к виртуальной сети можно использовать проверку подлинности на основе сертификатов или аутентификацию RADIUS. Однако при использовании открытого протокола VPN можно также использовать проверку подлинности Azure Active Directory. Если вы хотите, чтобы другой набор пользователей мог подключаться к разным VPN-шлюзам, можно зарегистрировать несколько приложений в AD и связать их с разными VPN-шлюзами. Эта статья поможет вам настроить клиент Azure AD для проверки подлинности P2S Опенвпн и создать и зарегистрировать несколько приложений в Azure AD, чтобы разрешить разные права доступа для разных пользователей и групп.

[!INCLUDE [Windows 10 and OpenVPN note](../../includes/vpn-gateway-openvpn-auth-include.md)]

[!INCLUDE [create](../../includes/openvpn-azure-ad-tenant-multi-app.md)]

## <a name="6-enable-authentication-on-the-gateway"></a><a name="enable-authentication"></a>6. Включение проверки подлинности на шлюзе

На этом шаге вы включите проверку подлинности Azure AD на VPN-шлюзе.

1. Включите проверку подлинности Azure AD на VPN-шлюзе, перейдя к **конфигурации "точка — сеть** " и выбрав **опенвпн (SSL)** в качестве **типа туннеля**. Выберите **Azure Active Directory** в качестве **типа проверки подлинности** , а затем введите сведения в разделе **Azure Active Directory** .

    ![Представление портал Azure](./media/openvpn-azure-ad-tenant-multi-app/azure-ad-auth-portal.png)

    > [!NOTE]
    > Не используйте идентификатор приложения клиента VPN Azure. он предоставит всем пользователям доступ к VPN-шлюзу. Используйте идентификатор зарегистрированного приложения.

2. Создайте и скачайте профиль, щелкнув ссылку **скачать VPN-клиент** .

3. Извлеките скачанный ZIP-файл.

4. Перейдите к распакованной папке "AzureVPN".

5. Запишите расположение файла "azurevpnconfig.xml". azurevpnconfig.xml содержит параметр для VPN-подключения и может быть импортирован непосредственно в клиентское приложение VPN Azure. Этот файл также можно передать всем пользователям, которым требуется подключение по электронной почте или другим средствам. Для успешного подключения пользователю понадобятся действительные учетные данные Azure AD.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы подключиться к виртуальной сети, необходимо создать и настроить профиль клиента VPN. См. раздел [Настройка VPN-клиента для P2S VPN-подключений](openvpn-azure-ad-client.md).
