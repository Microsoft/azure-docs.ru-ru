---
title: 'Клиент Azure AD для VPN-подключений пользователей: проверка подлинности Azure AD'
description: Для подключения к виртуальной сети с помощью проверки подлинности Azure AD можно использовать VPN-подключение пользователя виртуальной глобальной сети Azure ("точка-сеть").
titleSuffix: Azure Virtual WAN
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/22/2020
ms.author: alzam
ms.openlocfilehash: a54397fe7cfecef6813105645b7f2b218894875e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91367858"
---
# <a name="prepare-azure-active-directory-tenant-for-user-vpn-openvpn-protocol-connections"></a>Подготовка клиента Azure Active Directory для подключений по протоколу VPN Опенвпн для пользователя

При подключении к виртуальному концентратору по протоколу IKEv2 можно использовать проверку подлинности на основе сертификатов или аутентификацию RADIUS. Однако при использовании протокола Опенвпн можно также использовать проверку подлинности Azure Active Directory. Эта статья поможет вам настроить клиент Azure AD для VPN-подключения пользователей виртуальной глобальной сети (точка-сеть) с помощью проверки подлинности Опенвпн.

> [!NOTE]
> Аутентификация Azure AD поддерживается только для &reg; подключений по протоколу опенвпн.
>

## <a name="1-create-the-azure-ad-tenant"></a><a name="tenant"></a>1. Создание клиента Azure AD

Убедитесь, что у вас есть клиент Azure AD. Если у вас нет клиента Azure AD, можно создать его, выполнив действия, описанные в статье [Создание нового клиента](../active-directory/fundamentals/active-directory-access-create-new-tenant.md) :

* Название организации
* Первоначальное доменное имя

Пример

   ![Новый клиент Azure AD](./media/openvpn-create-azure-ad-tenant/newtenant.png)

## <a name="2-create-azure-ad-tenant-users"></a><a name="users"></a>2. Создание пользователей клиента Azure AD

Затем создайте две учетные записи пользователей во вновь созданном клиенте Azure AD, одну учетную запись глобального администратора и одну учетную запись пользователя. Учетную запись пользователя можно использовать для проверки подлинности Опенвпн, и учетная запись глобального администратора будет использоваться для предоставления согласия на регистрацию приложения VPN Azure. После создания учетной записи пользователя Azure AD назначьте пользователю **роль каталога** , чтобы делегировать административные разрешения.

Выполните действия, описанные в [этой статье](../active-directory/fundamentals/add-users-azure-active-directory.md) , чтобы создать двух пользователей для вашего клиента Azure AD. Не забудьте изменить **роль каталога** для одной из созданных учетных записей на **глобального администратора**.

## <a name="3-grant-consent-to-the-azure-vpn-app-registration"></a><a name="enable-authentication"></a>3. Предоставьте согласие на регистрацию приложения VPN Azure.

1. Войдите на портал Azure в качестве пользователя, которому назначена роль **глобального администратора** .

2. Затем предоставьте согласие администратора для вашей организации. Это позволит приложению VPN Azure входить в систему и читать профили пользователей. Скопируйте и вставьте URL-адрес, относящийся к расположению развертывания, в адресную строку браузера:

    Общедоступные

    ```
    https://login.microsoftonline.com/common/oauth2/authorize?client_id=41b23e61-6c1e-4545-b367-cd054e0ed4b4&response_type=code&redirect_uri=https://portal.azure.com&nonce=1234&prompt=admin_consent
    ````

    Azure для государственных организаций

    ```
    https://login-us.microsoftonline.com/common/oauth2/authorize?client_id=51bb15d4-3a4f-4ebf-9dca-40096fe32426&response_type=code&redirect_uri=https://portal.azure.us&nonce=1234&prompt=admin_consent
    ````

    Microsoft Cloud Germany

    ```
    https://login-us.microsoftonline.de/common/oauth2/authorize?client_id=538ee9e6-310a-468d-afef-ea97365856a9&response_type=code&redirect_uri=https://portal.microsoftazure.de&nonce=1234&prompt=admin_consent
    ````

    Azure China 21Vianet

    ```
    https://https://login.chinacloudapi.cn/common/oauth2/authorize?client_id=49f817b6-84ae-4cc0-928c-73f27289b3aa&response_type=code&redirect_uri=https://portal.azure.cn&nonce=1234&prompt=admin_consent
    ```

3. При появлении запроса выберите учетную запись **глобального администратора** .

    ![Идентификатор каталога](./media/openvpn-create-azure-ad-tenant/pick.png)

4. Выберите **Принять** при появлении соответствующего запроса.

    ![Снимок экрана показывает диалоговое окно с запрошенными разрешениями на сообщения, принятыми для вашей организации и дополнительными сведениями.](./media/openvpn-create-azure-ad-tenant/accept.jpg)

5. В Azure AD в разделе **корпоративные приложения** вы должны увидеть список **VPN Azure** .

    ![VPN Azure](./media/openvpn-create-azure-ad-tenant/azurevpn.png)

## <a name="next-steps"></a>Дальнейшие действия

Чтобы подключиться к виртуальным сетям с помощью аутентификации Azure AD, необходимо создать конфигурацию VPN пользователя и связать ее с виртуальным концентратором. См. статью [Настройка аутентификации Azure AD для подключения "точка — сеть" к Azure](virtual-wan-point-to-site-azure-ad.md).
