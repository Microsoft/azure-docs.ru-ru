---
title: Авторизация учетных записей разработчиков с помощью OAuth 2,0 в управлении API
titleSuffix: Azure API Management
description: Сведения об авторизации пользователей с помощью OAuth 2.0 в службе управления API. OAuth 2,0 защищает API таким образом, чтобы пользователи могли обращаться только к тем ресурсам, к которым они имеют право.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 08/14/2020
ms.author: apimpm
ms.openlocfilehash: fae4e349d46425c0c2b2b923d6a61e2e588708c1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93077257"
---
# <a name="how-to-authorize-developer-accounts-using-oauth-20-in-azure-api-management"></a>Авторизация учетных записей разработчиков с помощью протокола OAuth 2.0 в службе управления Azure API

Многие интерфейсы API поддерживают протокол [OAuth 2.0](https://oauth.net/2/) , позволяющий защитить API, а также и предоставлять доступ только действительным пользователям и только к ресурсам, на которые эти пользователи имеют право. Чтобы использовать интерактивную  консоль разработчика управления API Azure с такими API, служба позволяет настроить экземпляр службы для работы с API с поддержкой OAuth 2.0.

## <a name="prerequisites"></a><a name="prerequisites"> </a>Предварительные условия

В этом руководстве описано, как настроить экземпляр службы API Management для авторизации учетных записей разработчиков по протоколу OAuth 2.0. В нем не рассматривается настройка поставщика OAuth 2.0. Настройка каждого поставщика OAuth 2.0 имеет свои особенности, хотя инструкции в целом схожи, а данные, необходимые для настройки этого протокола в экземпляре службы API Management, одинаковы. В примерах в этом разделе в качестве поставщика OAuth 2.0 используется служба Azure Active Directory.

> [!NOTE]
> Дополнительные сведения о настройке OAuth 2.0 с использованием Azure Active Directory см. в примере [WebApp-GraphAPI-DotNet][WebApp-GraphAPI-DotNet].

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="configure-an-oauth-20-authorization-server-in-api-management"></a><a name="step1"> </a>Настройка сервера авторизации OAuth 2.0 в управлении API

> [!NOTE]
> Если экземпляр службы управления API еще не создан, выполните инструкции в статье [Создание экземпляра службы управления API Azure][Create an API Management service instance].

1. Перейдите на вкладку OAuth 2.0 в меню слева и щелкните **+ Добавить**.

    ![Меню OAuth 2.0](./media/api-management-howto-oauth2/oauth-01.png)

2. В полях **Имя** и **Описание** введите имя и (при желании) описание.

    > [!NOTE]
    > Эти поля служат для идентификации сервера авторизации OAuth 2.0 в текущем экземпляре службы API Management. Их значения не извлекаются с сервера OAuth 2.0.

3. Заполните поле **Client registration page URL** (URL-адрес страницы регистрации клиента). Это страница, на которой пользователи могут создавать свои учетные записи и управлять ими. Ее URL-адрес зависит от используемого поставщика OAuth 2.0. **URL-адрес страницы регистрации клиента** указывает на страницу, которую пользователи могут использовать для создания и настройки собственных учетных записей для поставщиков OAuth 2,0, которые поддерживают управление учетными записями пользователей, например `https://contoso.com/login` . В некоторых организациях эта функциональность не настраивается и не используется, даже если поставщик OAuth 2.0 ее поддерживает. Если у вашего поставщика OAuth 2.0 не настроено пользовательское управление учетными записями, введите здесь URL-адрес-заполнитель, такой как URL-адрес вашей компании или URL-адрес `https://placeholder.contoso.com`.

    ![Новый сервер OAuth 2.0](./media/api-management-howto-oauth2/oauth-02.png)

4. В следующем разделе формы содержатся параметры **Типы предоставления авторизации**, **URL-адрес конечной точки авторизации** и **Метод запроса авторизации**.

    Установите нужные флажки в разделе **Типы предоставления авторизации**. **Authorization code** (Код авторизации).

    Введите URL-адрес в поле **Authorization endpoint URL**. Для Azure Active Directory этот URL-адрес будет похож на следующий URL-адрес, где `<tenant_id>` заменяется идентификатором вашего клиента Azure AD.

    `https://login.microsoftonline.com/<tenant_id>/oauth2/authorize`

    Параметр **Authorization request method** указывает на то, каким образом запрос авторизации отправляется на сервер OAuth 2.0. По умолчанию выбран вариант **GET** .

5. Далее необходимо определить **Token endpoint URL** (URL-адрес конечной точки маркера), **Client authentication methods** (Способы проверки подлинности клиента), **Access token sending method** (Способ отправки маркера доступа) и **Default scope** (Область по умолчанию).

    ![Снимок экрана, на котором показан экран добавления службы OAuth2.](./media/api-management-howto-oauth2/oauth-03.png)

    Для сервера OAuth 2.0 Azure Active Directory параметр **Token endpoint URL** будет иметь следующий формат, где `<TenantID>` имеет вид `yourapp.onmicrosoft.com`.

    `https://login.microsoftonline.com/<TenantID>/oauth2/token`

    Параметр **Client authentication methods** по умолчанию имеет значение **Basic** (Обычная), а параметр **Access token sending method** — значение **Authorization header** (Заголовок авторизации). Эти значения можно настроить в данном разделе формы вместе с параметром **Default scope**.

6. В разделе **Client credentials** (Учетные данные клиента) содержатся поля **Client ID** (Идентификатор клиента) и **Client secret** (Секрет клиента), значения которых получены в ходе создания и настройки сервера OAuth 2.0. После определения значений **Client ID** и **Client secret** генерируется параметр **redirect_uri** (URI перенаправления) для **кода авторизации**. Этот универсальный код ресурса (URI) используется для настройки URL-адреса ответа в конфигурации сервера OAuth 2.0.

    На новом портале разработчика суффикс URI имеет вид:

    - `/signin-oauth/code/callback/{authServerName}` для потока предоставления кода авторизации
    - `/signin-oauth/implicit/callback` для неявного потока предоставления

    ![Снимок экрана, на котором показано, куда добавить учетные данные клиента для новой службы OAuth2.](./media/api-management-howto-oauth2/oauth-04.png)

    Если для параметра **Типы предоставления авторизации** задано значение **Пароль владельца ресурса**, то в разделе **Учетные данные владельца ресурса** нужно указать соответствующие учетные данные. В противном случае эти поля можно оставить пустыми.

    После заполнения формы щелкните **Создать**, чтобы сохранить конфигурацию сервера авторизации OAuth 2.0 для службы API Management. Сохранив конфигурацию сервера, вы можете настроить интерфейсы API для ее использования, как описано в следующем разделе.

## <a name="configure-an-api-to-use-oauth-20-user-authorization"></a><a name="step2"> </a>Настройка API для авторизации пользователей по протоколу OAuth 2.0

1. В меню **Управление API** слева щелкните **API**.

    ![Интерфейсы API OAuth 2.0](./media/api-management-howto-oauth2/oauth-05.png)

2. Выберете имя нужного API и щелкните **Параметры**. Прокрутите страницу до раздела **Безопасность** и установите флажок для **OAuth 2.0**.

    ![Параметры OAuth 2.0](./media/api-management-howto-oauth2/oauth-06.png)

3. В раскрывающемся списке **Authorization server** (Сервер авторизации) выберите нужный пункт и нажмите кнопку **Save** (Сохранить).

    ![Снимок экрана, на котором выделяется выбранный сервер авторизации и кнопка "Сохранить".](./media/api-management-howto-oauth2/oauth-07.png)

## <a name="legacy-developer-portal---test-the-oauth-20-user-authorization"></a><a name="step3"> </a>Устаревший портал разработчика. Тестирование авторизации пользователя OAuth 2,0

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

Настроив сервер авторизации OAuth 2.0 и его использование интерфейсом API, вы можете протестировать сервер, перейдя на портал разработчика и вызвав интерфейс API. Щелкните **портал разработчика (прежние версии)** в верхнем меню на странице **обзора** экземпляра службы управления API Azure.

Щелкните **APIs** в меню вверху и выберите **Echo API**.

![Echo API][api-management-apis-echo-api]

> [!NOTE]
> Если есть только один настроенный или видимый API для данной учетной записи, тогда щелчок по API сразу приведет к операциям для этого API.

Выберите операцию **GET Resource**, щелкните **Open Console** (Открыть консоль) и выберите в раскрывающемся списке пункт **Authorization code** (Код авторизации).

![Открытие консоли][api-management-open-console]

После выбора пункта **Authorization code** появляется всплывающее окно с формой входа поставщика OAuth 2.0. В этом примере форма входа предоставляется службой Azure Active Directory.

> [!NOTE]
> Если всплывающие окна отключены, то в браузере появится запрос на их включение. Включив всплывающие окна, еще раз выберите пункт **Authorization code** , чтобы открыть форму входа.

![Вход][api-management-oauth2-signin]

После входа поле **Request headers** (Заголовки запроса) заполняется заголовком `Authorization : Bearer`, используемым для авторизации запроса.

![Маркер заголовка запроса][api-management-request-header-token]

Теперь вы можете настроить остальные параметры и отправить запрос.

## <a name="next-steps"></a>Дальнейшие действия

Для получения дополнительных сведений об использовании OAuth 2.0 и службы управления API см. следующий видеоролик и эту [статью](api-management-howto-protect-backend-with-aad.md).

[api-management-oauth2-signin]: ./media/api-management-howto-oauth2/api-management-oauth2-signin.png
[api-management-request-header-token]: ./media/api-management-howto-oauth2/api-management-request-header-token.png
[api-management-open-console]: ./media/api-management-howto-oauth2/api-management-open-console.png
[api-management-apis-echo-api]: ./media/api-management-howto-oauth2/api-management-apis-echo-api.png

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

[Prerequisites]: #prerequisites
[Configure an OAuth 2.0 authorization server in API Management]: #step1
[Configure an API to use OAuth 2.0 user authorization]: #step2
[Test the OAuth 2.0 user authorization in the Developer Portal]: #step3
[Next steps]: #next-steps
