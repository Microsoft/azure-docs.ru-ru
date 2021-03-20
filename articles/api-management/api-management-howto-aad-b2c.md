---
title: Авторизация учетных записей разработчиков с помощью Azure Active Directory B2C
titleSuffix: Azure API Management
description: Сведения об авторизации пользователей с помощью Azure Active Directory B2C в службе управления API.
services: api-management
documentationcenter: API Management
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: apimpm
ms.openlocfilehash: 7b586edd7adce8bcea61419005a3ce8cfc814fb3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96013570"
---
# <a name="how-to-authorize-developer-accounts-by-using-azure-active-directory-b2c-in-azure-api-management"></a>Как авторизовать учетные записи разработчиков с помощью Azure Active Directory B2C в службе управления API Azure

## <a name="overview"></a>Обзор

Azure Active Directory B2C — это облачное решение, позволяющее управлять удостоверениями в веб-приложениях и мобильных приложениях, с которыми взаимодействуют клиенты. Его можно использовать для управления доступом к порталу разработчика. В этом руководстве объясняется, как настроить службу управления API для интеграции с Azure Active Directory B2C. Сведения о предоставлении доступа к порталу разработчика с помощью классической службы Azure Active Directory см. в статье [Авторизация учетных записей разработчиков с помощью Azure Active Directory в управлении API Azure].

> [!NOTE]
> Для выполнения шагов в этом руководстве вам сначала потребуется клиент Azure Active Directory B2C для создания в нем приложения. Кроме того, необходимо настроить политики регистрации и входа. Дополнительные сведения см. в статье [Azure Active Directory B2C: регистрация и вход пользователей в приложения].

[!INCLUDE [premium-dev-standard.md](../../includes/api-management-availability-premium-dev-standard.md)]

## <a name="authorize-developer-accounts-by-using-azure-active-directory-b2c"></a>Авторизация учетных записей разработчиков с помощью Azure Active Directory B2C

1. Чтобы приступить к работе, перейдите на [портал Azure](https://portal.azure.com) и найдите свой экземпляр управления API.

   > [!NOTE]
   > Если вы еще не создали экземпляр службы управления API, ознакомьтесь со статьей [Создание экземпляра службы управления API][Create an API Management service instance] в [руководстве Приступая к работе с управлением API Azure][Get started with Azure API Management].

1. В разделе **удостоверения**. В верхней части щелкните **+ Добавить** .

   Справа отобразится область **Добавление поставщика удостоверений**. Выберите **Azure Active Directory B2C**.
    
   ![Добавление AAD B2C в качестве поставщика удостоверений][api-management-howto-add-b2c-identity-provider]

1. Скопируйте **URL-адрес перенаправления**.

   ![URL-адрес перенаправления поставщика удостоверений AAD B2C][api-management-howto-copy-b2c-identity-provider-redirect-url]

1. В новой вкладке получите доступ к клиенту Azure Active Directory B2C на портале Azure и откройте колонку **Приложение**.

   ![Регистрация нового приложения 1][api-management-howto-aad-b2c-portal-menu]

1. Нажмите кнопку **Добавить**, чтобы создать новое приложение Azure Active Directory B2C.

   ![Регистрация нового приложения 2][api-management-howto-aad-b2c-add-button]

1. В колонке **Новое приложение** введите имя для приложения. Выберите **Да** под параметром **Веб-приложения и веб-API** и **Да** под параметром **Разрешить неявный поток**. Вставьте **URL-адрес перенаправления**, скопированный в шаге 3, в текстовое поле **URL-адреса ответа**.

   ![Регистрация нового приложения 3][api-management-howto-aad-b2c-app-details]

1. Если вы используете новый портал разработчика (не устаревший портал разработчика), включите в утверждения приложения **заданное имя**, **фамилию** и **идентификатор объекта пользователя** .

    ![Утверждения приложения](./media/api-management-howto-aad-b2c/api-management-application-claims.png)

1. Нажмите кнопку **Создать** . Когда приложение будет создано, оно появится в колонке **Приложения**. Щелкните имя приложения, чтобы просмотреть сведения о нем.

   ![Регистрация нового приложения 4][api-management-howto-aad-b2c-app-created]

1. В колонке **Свойства** скопируйте в буфер обмена **Идентификатор приложения**.

   ![Идентификатор приложения 1][api-management-howto-aad-b2c-app-id]

1. Вернитесь на панель **Добавление поставщика удостоверений** управления API и вставьте идентификатор в текстовое поле **идентификатора клиента**.
    
1.  Вернитесь к регистрации приложения B2C, щелкните раздел **Ключи**, а затем нажмите кнопку **Создать ключ**. Нажмите кнопку **Сохранить**, чтобы сохранить конфигурацию и отобразить **Ключ приложения**. Скопируйте этот ключ в буфер обмена.

    ![Ключ приложения 1][api-management-howto-aad-b2c-app-key]

1.  Вернитесь на панель **Добавление поставщика удостоверений** управления API и вставьте ключ в текстовое поле **секрет клиента**.
    
1.  Укажите доменное имя клиента Azure Active Directory B2C в **клиенте SignIn**.

1.  Поле " **центр** " позволяет управлять используемым URL-адресом входа Azure AD B2C. Задайте значение **<your_b2c_tenant_name>. b2clogin.com**.

1. Из политик клиента B2C укажите **политику регистрации** и **политику входа**. Дополнительно можно также указать **политику редактирования профиля** и **политику сброса пароля**.

1. После указания требуемой конфигурации нажмите кнопку **Сохранить**.

    После сохранения изменений разработчики смогут создавать новые учетные записи и входить на портал разработчика с помощью Azure Active Directory B2C.

## <a name="developer-portal---add-azure-ad-b2c-account-authentication"></a>Портал разработчика. Добавление проверки подлинности учетной записи Azure AD B2C

На портале разработчика вход с помощью AAD B2C возможен с помощью **кнопки входа: мини-приложение OAuth** . Мини-приложение уже включено на странице входа в содержимое портала разработчика по умолчанию.

Несмотря на то что новая учетная запись будет создана автоматически при входе нового пользователя с AAD B2C, вы можете добавить это же мини-приложение на страницу регистрации.

**Форма регистрации: мини-приложение OAuth** представляет форму, используемую для регистрации с помощью OAuth.

> [!IMPORTANT]
> Чтобы изменения AAD вступили в силу, необходимо [повторно опубликовать портал](api-management-howto-developer-portal-customize.md#publish) .

## <a name="legacy-developer-portal---how-to-sign-up-with-azure-ad-b2c"></a>Устаревший портал разработчика. Регистрация с помощью Azure AD B2C

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

1. Чтобы зарегистрировать учетную запись разработчика с помощью Azure Active Directory B2C, откройте новое окно браузера и перейдите на портал разработчика. Нажмите кнопку **Зарегистрироваться**.

   ![Портал разработчика 1][api-management-howto-aad-b2c-dev-portal]

2. Выберите регистрацию с помощью **Azure Active Directory B2C**.

   ![Портал разработчика 2][api-management-howto-aad-b2c-dev-portal-b2c-button]

3. Вы будете перенаправлены к политике регистрации, настроенной в предыдущем разделе. Выберите регистрацию с помощью адреса электронной почты или существующей учетной записи социальных сетей.

   > [!NOTE]
   > Если Azure Active Directory B2C является единственным активным параметром на вкладке **Удостоверения** на портале издателя, вы будете сразу перенаправлены к политике регистрации.

   ![Портал разработчика][api-management-howto-aad-b2c-dev-portal-b2c-options]

   После завершения регистрации вы будете перенаправлены обратно на портал разработчика. Теперь вы вошли на портал разработчика вашего экземпляра службы управления API.

    ![Регистрация завершена][api-management-registration-complete]

## <a name="next-steps"></a>Дальнейшие действия

*  [Azure Active Directory B2C: регистрация и вход пользователей в приложения]
*  [Azure Active Directory B2C: расширяемая инфраструктура политик]
*  [Azure Active Directory (AD) B2C: организация регистрации и входа для потребителей с учетными записями Microsoft]
*  [Azure Active Directory B2C: организация регистрации и входа для потребителей с учетными записями Google+]
*  [Azure Active Directory B2C: регистрация и вход пользователей с помощью учетных записей LinkedIn]
*  [Azure Active Directory B2C: регистрация и вход пользователей с учетными записями Facebook]



[api-management-howto-add-b2c-identity-provider]: ./media/api-management-howto-aad-b2c/api-management-add-b2c-identity-provider.PNG
[api-management-howto-copy-b2c-identity-provider-redirect-url]: ./media/api-management-howto-aad-b2c/api-management-b2c-identity-provider-redirect-url.PNG
[api-management-howto-aad-b2c-portal-menu]: ./media/api-management-howto-aad-b2c/api-management-b2c-portal-menu.PNG
[api-management-howto-aad-b2c-add-button]: ./media/api-management-howto-aad-b2c/api-management-b2c-add-button.PNG
[api-management-howto-aad-b2c-app-details]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-details.PNG
[api-management-howto-aad-b2c-app-created]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-created.PNG
[api-management-howto-aad-b2c-app-id]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-id.PNG
[api-management-howto-aad-b2c-client-id]: ./media/api-management-howto-aad-b2c/api-management-b2c-client-id.PNG
[api-management-howto-aad-b2c-app-key]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-key.PNG
[api-management-howto-aad-b2c-app-key-saved]: ./media/api-management-howto-aad-b2c/api-management-b2c-app-key-saved.PNG
[api-management-howto-aad-b2c-client-secret]: ./media/api-management-howto-aad-b2c/api-management-b2c-client-secret.PNG
[api-management-howto-aad-b2c-allowed-tenant]: ./media/api-management-howto-aad-b2c/api-management-b2c-allowed-tenant.PNG
[api-management-howto-aad-b2c-policies]: ./media/api-management-howto-aad-b2c/api-management-b2c-policies.PNG
[api-management-howto-aad-b2c-dev-portal]: ./media/api-management-howto-aad-b2c/api-management-b2c-dev-portal.PNG
[api-management-howto-aad-b2c-dev-portal-b2c-button]: ./media/api-management-howto-aad-b2c/api-management-b2c-dev-portal-b2c-button.PNG
[api-management-howto-aad-b2c-dev-portal-b2c-options]: ./media/api-management-howto-aad-b2c/api-management-b2c-dev-portal-b2c-options.PNG
[api-management-complete-registration]: ./media/api-management-howto-aad/api-management-complete-registration.PNG
[api-management-registration-complete]: ./media/api-management-howto-aad/api-management-registration-complete.png

[api-management-security-external-identities]: ./media/api-management-howto-aad/api-management-b2c-security-tab.png
[api-management-security-aad-new]: ./media/api-management-howto-aad/api-management-security-aad-new.png
[api-management-new-aad-application-menu]: ./media/api-management-howto-aad/api-management-new-aad-application-menu.png
[api-management-new-aad-application-1]: ./media/api-management-howto-aad/api-management-new-aad-application-1.png
[api-management-new-aad-application-2]: ./media/api-management-howto-aad/api-management-new-aad-application-2.png
[api-management-new-aad-app-created]: ./media/api-management-howto-aad/api-management-new-aad-app-created.png
[api-management-aad-app-permissions]: ./media/api-management-howto-aad/api-management-aad-app-permissions.png
[api-management-aad-app-client-id]: ./media/api-management-howto-aad/api-management-aad-app-client-id.png
[api-management-client-id]: ./media/api-management-howto-aad/api-management-client-id.png
[api-management-aad-key-before-save]: ./media/api-management-howto-aad/api-management-aad-key-before-save.png
[api-management-aad-key-after-save]: ./media/api-management-howto-aad/api-management-aad-key-after-save.png
[api-management-client-secret]: ./media/api-management-howto-aad/api-management-client-secret.png
[api-management-client-allowed-tenants]: ./media/api-management-howto-aad/api-management-client-allowed-tenants.png
[api-management-client-allowed-tenants-save]: ./media/api-management-howto-aad/api-management-client-allowed-tenants-save.png
[api-management-aad-delegated-permissions]: ./media/api-management-howto-aad/api-management-aad-delegated-permissions.png
[api-management-dev-portal-signin]: ./media/api-management-howto-aad/api-management-dev-portal-signin.png
[api-management-aad-signin]: ./media/api-management-howto-aad/api-management-aad-signin.png
[api-management-aad-app-multi-tenant]: ./media/api-management-howto-aad/api-management-aad-app-multi-tenant.png
[api-management-aad-reply-url]: ./media/api-management-howto-aad/api-management-aad-reply-url.png
[api-management-permissions-form]: ./media/api-management-howto-aad/api-management-permissions-form.png
[api-management-configure-product]: ./media/api-management-howto-aad/api-management-configure-product.png
[api-management-add-groups]: ./media/api-management-howto-aad/api-management-add-groups.png
[api-management-select-group]: ./media/api-management-howto-aad/api-management-select-group.png
[api-management-aad-groups-list]: ./media/api-management-howto-aad/api-management-aad-groups-list.png
[api-management-aad-group-added]: ./media/api-management-howto-aad/api-management-aad-group-added.png
[api-management-groups]: ./media/api-management-howto-aad/api-management-groups.png
[api-management-edit-group]: ./media/api-management-howto-aad/api-management-edit-group.png

[How to add operations to an API]: ./mock-api-responses.md
[How to add and publish a product]: api-management-howto-add-products.md
[Monitoring and analytics]: api-management-monitoring.md
[Add APIs to a product]: api-management-howto-add-products.md#add-apis
[Publish a product]: api-management-howto-add-products.md#publish-product
[Get started with Azure API Management]: get-started-create-service-instance.md
[API Management policy reference]: ./api-management-policies.md
[Caching policies]: ./api-management-policies.md#caching-policies
[Create an API Management service instance]: get-started-create-service-instance.md

[https://oauth.net/2/]: https://oauth.net/2/
[WebApp-GraphAPI-DotNet]: https://github.com/AzureADSamples/WebApp-GraphAPI-DotNet
[Azure Active Directory B2C: регистрация и вход пользователей в приложения]: ../active-directory-b2c/overview.md
[Авторизация учетных записей разработчиков с помощью Azure Active Directory в управлении API Azure]: ./api-management-howto-aad.md
[Azure Active Directory B2C: расширяемая инфраструктура политик]: ../active-directory-b2c/user-flow-overview.md
[Azure Active Directory (AD) B2C: организация регистрации и входа для потребителей с учетными записями Microsoft]: ../active-directory-b2c/identity-provider-microsoft-account.md
[Azure Active Directory B2C: организация регистрации и входа для потребителей с учетными записями Google+]: ../active-directory-b2c/identity-provider-google.md
[Azure Active Directory B2C: регистрация и вход пользователей с учетными записями Facebook]: ../active-directory-b2c/identity-provider-facebook.md
[Azure Active Directory B2C: регистрация и вход пользователей с помощью учетных записей LinkedIn]: ../active-directory-b2c/identity-provider-linkedin.md

[Prerequisites]: #prerequisites
[Configure an OAuth 2.0 authorization server in API Management]: #step1
[Configure an API to use OAuth 2.0 user authorization]: #step2
[Test the OAuth 2.0 user authorization in the Developer Portal]: #step3
[Next steps]: #next-steps

[Log in to the Developer portal using an Azure Active Directory account]: #Log-in-to-the-Developer-portal-using-an-Azure-Active-Directory-account
