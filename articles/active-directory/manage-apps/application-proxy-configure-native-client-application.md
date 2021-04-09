---
title: Публикация собственных клиентских приложений — Azure AD | Документация Майкрософт
description: В этой статье описано, как включить собственные клиентские приложения для взаимодействия с соединителем прокси приложений Azure AD, чтобы обеспечить безопасный удаленный доступ к локальным приложениям.
services: active-directory
documentationcenter: ''
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/12/2020
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: e830b52accefa0bcb76cc3434cae2fe94eed97db
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99258668"
---
# <a name="how-to-enable-native-client-applications-to-interact-with-proxy-applications"></a>Включение собственных клиентских приложений для взаимодействия с приложениями прокси

С помощью Azure Active Directory (Azure AD) Application Proxy можно публиковать не только веб-приложения, но и собственные клиентские приложения, настроенные с помощью библиотеки аутентификации Майкрософт (MSAL). Собственные клиентские приложения отличаются от веб-приложений, так как они устанавливаются на устройство, а веб-приложения предоставляются через браузер.

Для поддержки собственных клиентских приложений Application Proxy принимает маркеры, выданные Azure AD, которые отправляются в заголовке. Служба Application Proxy выполняет проверку подлинности пользователей. Это решение не использует маркеры приложения для проверки подлинности.

![Связь между конечными пользователями, Azure AD и опубликованными приложениями](./media/application-proxy-configure-native-client-application/richclientflow.png)

Для публикации собственных приложений используйте библиотеку аутентификации Майкрософт, которая отвечает за аутентификацию и поддерживает многие клиентские среды. Application Proxy подходит для сценариев, когда [классическое приложение вызывает веб-API от имени пользователя, выполнившего вход](../develop/authentication-flows-app-scenarios.md#desktop-app-that-calls-a-web-api-on-behalf-of-a-signed-in-user).

В этой статье рассматриваются четыре действия, которые нужно выполнить, чтобы опубликовать собственное приложение с помощью прокси приложения и библиотеки аутентификации Azure AD.

## <a name="step-1-publish-your-proxy-application"></a>Шаг 1. Публикация приложения прокси

Опубликуйте приложение прокси, как любое другое приложение, и назначьте пользователей, имеющих доступ к вашему приложению. Дополнительные сведения см. в статье [Публикация приложений с помощью прокси приложения Azure AD](application-proxy-add-on-premises-application.md).

## <a name="step-2-register-your-native-application"></a>Шаг 2. Регистрация собственного приложения

Теперь необходимо зарегистрировать приложение в Azure AD следующим образом.

1. Войдите на [портал Azure Active Directory](https://aad.portal.azure.com/). Появится **панель мониторинга** для **центра администрирования Azure Active Directory**.
1. На панели сбоку выберите **Azure Active Directory**. Появится страница с общими сведениями об **Azure Active Directory**.
1. На боковой панели с общими сведениями об Azure AD выберите **Регистрация приложений**. Отобразится список всех зарегистрированных приложений.
1. Выберите **Новая регистрация**. Появится страница **Регистрация приложения**.

   ![Создание новой регистрации приложения на портале Azure](./media/application-proxy-configure-native-client-application/create.png)

1. В заголовке **Имя** укажите отображаемое имя пользователя для приложения.
1. В заголовке **Поддерживаемые типы учетных записей** выберите уровень доступа, используя следующие рекомендации.

   - Чтобы выбрать только те учетные записи, которые являются внутренними для вашей организации, выберите **Учетные записи только в этом каталоге организации**.
   - Чтобы выбрать только предприятия или учебные заведения, выберите **Учетные записи в любом каталоге организации**.
   - Чтобы охватить максимально широкий набор клиентов, выберите **Учетные записи в любом каталоге организации и личные учетные записи Майкрософт**.
1. В разделе **Универсальный код ресурса (URI) перенаправления** выберите **Общедоступный клиент (мобильный и классический)** , а затем введите код URI перенаправления `https://login.microsoftonline.com/common/oauth2/nativeclient` для приложения.
1. Выберите и прочтите **политики платформы Майкрософт**, а затем выберите **Зарегистрировать**. После этого появится страница со сведениями о новой регистрации приложения.

Дополнительные сведения о создании регистрации приложения см. в разделе [Интеграция приложений с Azure Active Directory](../develop/quickstart-register-app.md).

## <a name="step-3-grant-access-to-your-proxy-application"></a>Шаг 3. Предоставление доступа к приложению прокси

После регистрации собственного приложения можно предоставить ему доступ к другим приложениям в каталоге, в данном случае для доступа к приложению прокси. Включение собственного приложения, чтобы оно было доступно для приложения прокси:

1. На боковой панели страницы регистрации нового приложения выберите **Разрешения API**. Откроется страница **Разрешения API** для новой регистрации приложения.
1. Выберите **Добавить разрешение**. Откроется страница **Запрос разрешений API**.
1. В разделе **Выберите API** выберите **Интерфейсы API, используемые моей организацией**. Появится список приложений в каталоге, которые предоставляют интерфейсы API.
1. Введите текст в поле поиска или прокрутите страницу, чтобы найти приложение прокси, опубликованное в разделе [Шаг 1. Публикация приложения прокси](#step-1-publish-your-proxy-application), а затем выберите приложение прокси.
1. В заголовке **Какие разрешения требуются вашему приложению?** выберите тип разрешения. Если собственному приложению требуется доступ к API приложения прокси в качестве пользователя, выполнившего вход, выберите **Делегированные разрешения**.
1. В заголовке **Выберите разрешения** выберите нужное разрешение и **Добавить разрешения**. На странице **Разрешения API** для собственного приложения теперь отображается добавленное приложение прокси и API разрешения.

## <a name="step-4-add-the-microsoft-authentication-library-to-your-code-net-c-sample"></a>Шаг 4. Добавление библиотеки проверки подлинности Майкрософт (MSAL) в код (пример .NET C#)

Измените код собственного приложения с учетом проверки подлинности с помощью библиотеки MSAL. Для этого включите в код следующий текст: 

```         
// Acquire Access Token from AAD for Proxy Application
IPublicClientApplication clientApp = PublicClientApplicationBuilder
.Create(<App ID of the Native app>)
.WithDefaultRedirectUri() // will automatically use the default Uri for native app
.WithAuthority("https://login.microsoftonline.com/{<Tenant ID>}")
.Build();

AuthenticationResult authResult = null;
var accounts = await clientApp.GetAccountsAsync();
IAccount account = accounts.FirstOrDefault();

IEnumerable<string> scopes = new string[] {"<Scope>"};

try
 {
    authResult = await clientApp.AcquireTokenSilent(scopes, account).ExecuteAsync();
 }
    catch (MsalUiRequiredException ex)
 {
     authResult = await clientApp.AcquireTokenInteractive(scopes).ExecuteAsync();                
 }

if (authResult != null)
 {
  //Use the Access Token to access the Proxy Application

  HttpClient httpClient = new HttpClient();
  HttpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", authResult.AccessToken);
  HttpResponseMessage response = await httpClient.GetAsync("<Proxy App Url>");
 }
```

Необходимые сведения в примере кода можно найти на портале Azure AD следующим образом.

| Требуемые сведения | Поиск на портале Azure AD |
| --- | --- |
| \<Tenant ID> | **Azure Active Directory** > **Свойства** > **ИД каталога** |
| \<App ID of the Native app> | **Регистрация приложений** > *ваше собственное приложение* > **Обзор** > **ИД приложения** |
| \<Scope> | **Регистрация приложений** > *ваше собственное приложение* > **Разрешения API** > щелкните API разрешения (user_impersonation), после чего справа появится панель с заголовком **user_impersonation**. > Область является URL-адресом в поле ввода.
| \<Proxy App Url> | Внешний URL-адрес и путь к API

После внесения в код MSAL этих параметров ваши пользователи смогут выполнять проверку подлинности в собственных клиентских приложениях за пределами корпоративной сети.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о блок-схеме собственного приложения см. в разделе [Собственные приложения в Azure Active Directory](../azuread-dev/native-app.md).

Узнайте подробнее о том, как настроить [единый вход в приложениях в Azure Active Directory](sso-options.md#choosing-a-single-sign-on-method).