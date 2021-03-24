---
title: Публикация в локальной среде SharePoint с помощью прокси приложения Azure AD
description: В этой статье рассматриваются основы интеграции локального сервера SharePoint с Azure AD Application Proxy для SAML.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/02/2019
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 34aaafcd03e737b1e59529f8001e0c008bd39b70
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104888878"
---
# <a name="integrate-with-sharepoint-saml"></a>Интеграция с SharePoint (SAML)

В этом пошаговом руководство объясняется, как защитить доступ к [Azure Active Directory интегрированной локальной среде SharePoint (SAML)](../saas-apps/sharepoint-on-premises-tutorial.md) с помощью AD application proxy Azure, где пользователи в вашей организации (Azure AD, B2B) подключаются к SharePoint через Интернет.

> [!NOTE] 
> Если вы не знакомы с Azure AD Application Proxy и хотите узнать больше, ознакомьтесь со статьей [Удаленный доступ к локальным приложениям с помощью azure AD application proxy](./application-proxy.md).

Существует три основных преимущества этой установки.

- AD Application Proxy Azure гарантирует, что прошедший проверку подлинности трафик будет достигать внутренней сети и сервера SharePoint.
- Пользователи могут получить доступ к сайтам SharePoint обычным образом без использования VPN.
- Вы можете управлять доступом пользователей на уровне AD Application Proxy Azure, а также повысить безопасность с помощью таких функций Azure AD, как условный доступ и многофакторная идентификация (MFA).

Этот процесс требует наличия двух корпоративных приложений. Один из них — локальный экземпляр SharePoint, который публикуется из коллекции в списке управляемых приложений SaaS. Второй — это локальное приложение (приложение не из коллекции), которое будет использоваться для публикации первого приложения из коллекции для предприятий.

## <a name="prerequisites"></a>Предварительные условия

Для выполнения этой настройки необходимы следующие ресурсы:
 - Ферма SharePoint 2013 или более поздней версии. Ферма SharePoint должна быть [интегрирована с Azure AD](../saas-apps/sharepoint-on-premises-tutorial.md).
 - Клиент Azure AD с планом, который включает прокси приложения. Узнайте больше о [планах и ценах Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).
 - [Пользовательский проверенный домен](../fundamentals/add-custom-domain.md) в клиенте Azure AD. Проверенный домен должен соответствовать суффиксу URL-адреса SharePoint.
 - Требуется SSL-сертификат. См. Дополнительные сведения о [публикации личного домена](./application-proxy-configure-custom-domain.md).
 - Локальные Active Directory пользователи должны быть синхронизированы с Azure AD Connect и должны быть настроены для [входа в Azure](../hybrid/plan-connect-user-signin.md). 
 - Для гостевых пользователей, использующих только облачные технологии и службы B2B, необходимо [предоставить доступ к гостевой учетной записи в локальной среде SharePoint в портал Azure](../saas-apps/sharepoint-on-premises-tutorial.md#grant-access-to-a-guest-account-to-sharepoint-on-premises-in-the-azure-portal).
 - Соединитель прокси приложения установлен и работает на компьютере в корпоративном домене.


## <a name="step-1-integrate-sharepoint-on-premises-with-azure-ad"></a>Шаг 1. Интеграция SharePoint в локальной среде с помощью Azure AD 

1. Настройте локальное приложение SharePoint. Дополнительные сведения см. [в разделе Учебник. Интеграция единого входа Azure Active Directory с SharePoint в локальной среде](../saas-apps/sharepoint-on-premises-tutorial.md).
2. Проверьте конфигурацию перед переходом к следующему шагу. Чтобы проверить это, попытайтесь получить доступ к локальной сети SharePoint из внутренних сетей и убедиться, что она доступна для внутренних целей. 


## <a name="step-2-publish-the-sharepoint-on-premises-application-with-application-proxy"></a>Шаг 2. Публикация локального приложения SharePoint с помощью прокси приложения

На этом шаге вы создадите приложение в клиенте Azure AD, использующем прокси приложения. Задайте внешний URL-адрес и укажите внутренний URL-адрес, который будет использоваться позже в SharePoint.

> [!NOTE] 
> Внутренний и внешний URL-адреса должны соответствовать **URL-адресу входа** в конфигурации приложения на основе SAML на шаге 1.

   ![Снимок экрана, показывающий значение URL-адреса входа.](./media/application-proxy-integrate-with-sharepoint-server/sso-url-saml.png)


 1. Создайте новое приложение AD Application Proxy Azure с пользовательским доменом. Пошаговые инструкции см. [в разделе Личные домены в Azure AD application proxy](./application-proxy-configure-custom-domain.md).

    - Внутренний URL-адрес: " https://portal.contoso.com/ "
    - Внешний URL-адрес: " https://portal.contoso.com/ "
    - Предварительная проверка подлинности: Azure Active Directory
    - Перевод URL-адресов в заголовки: нет
    - Перевод URL-адресов в тексте приложения: нет

        ![Снимок экрана, на котором показаны параметры, используемые для создания приложения.](./media/application-proxy-integrate-with-sharepoint-server/create-application-azure-active-directory.png)

2. Назначьте те [же группы](../saas-apps/sharepoint-on-premises-tutorial.md#create-an-azure-ad-security-group-in-the-azure-portal) , которые вы назначили локальному приложению коллекции SharePoint.

3. Наконец, перейдите в раздел **Properties (свойства** ) и задайте значение **нет** **для параметра Показывать пользователям** . Этот параметр гарантирует, что на портале "Мои приложения" отображается только значок первого приложения ( https://myapplications.microsoft.com) .

   ![Снимок экрана, на котором показано, где можно задать видимость для пользователей? функцию.](./media/application-proxy-integrate-with-sharepoint-server/configure-properties.png)
 
## <a name="step-3-test-your-application"></a>Шаг 3. Тестирование приложения

Используя браузер из компьютера во внешней сети, перейдите по ссылке, настроенной на этапе публикации. Убедитесь, что вы можете войти в систему с помощью настроенной тестовой учетной записи.