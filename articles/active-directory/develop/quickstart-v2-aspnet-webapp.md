---
title: Краткое руководство. Добавление возможности входа в веб-приложение ASP.NET с помощью учетной записи Майкрософт | Azure
titleSuffix: Microsoft identity platform
description: В этом кратком руководстве показано, как реализовать единый вход Майкрософт в веб-приложении ASP.NET с помощью OpenID Connect.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 09/25/2020
ms.author: jmprieur
ms.custom: devx-track-csharp, aaddev, identityplatformtop40, scenarios:getting-started, languages:ASP.NET, contperf-fy21q1
ms.openlocfilehash: 87948ed04f7b50820d94993d4c4fbcf2dfd94b31
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104578691"
---
# <a name="quickstart-add-microsoft-identity-platform-sign-in-to-an-aspnet-web-app"></a>Краткое руководство. Добавление функции входа платформы Microsoft Identity в веб-приложение ASP.NET

Из этого краткого руководства вы узнаете, как скачать и выполнить пример кода, в котором показана реализация входа в веб-приложение ASP.NET для пользователей из любой организации Azure Active Directory (Azure AD). 

> [!div renderon="docs"]
> На следующей схеме показано, как работает пример приложения.
>
> ![Схема взаимодействия между веб-браузером, веб-приложением и платформой удостоверений Майкрософт в примере приложения.](media/quickstart-v2-aspnet-webapp/aspnetwebapp-intro.svg)
>
> ## <a name="prerequisites"></a>Предварительные требования
>
> * Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
> * [Visual Studio 2019](https://visualstudio.microsoft.com/vs/)
> * [.NET Framework 4.7.2 или более поздней версии](https://dotnet.microsoft.com/download/visual-studio-sdks)
>
> ## <a name="register-and-download-the-app"></a>Регистрация и скачивание приложения
> Есть два варианта по созданию приложения: автоматическая или ручная настройка.
>
> ### <a name="automatic-configuration"></a>Автоматическая настройка
> Если вы хотите автоматически настроить приложение, а затем скачать пример кода, выполните следующие действия.
>
> 1. Перейдите на <a href="https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/AspNetWebAppQuickstartPage/sourceType/docs" target="_blank">страницу портала Azure для регистрации приложения</a>.
> 1. Введите имя приложения и нажмите кнопку **Зарегистрировать**.
> 1. Следуйте инструкциям, чтобы скачать и автоматически настроить новое приложение одним щелчком мыши.
>
> ### <a name="manual-configuration"></a>Настройка вручную
> Если вы хотите вручную настроить приложение и пример кода, воспользуйтесь следующими процедурами.
>
> #### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения
>
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
> 1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
> 1. Найдите и выберите **Azure Active Directory**.
> 1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
> 1. В поле **Имя** введите имя для своего приложения. Например, введите **ASPNET-Quickstart**. Это имя будут видеть пользователи приложения. Вы сможете изменить его позже.
> 1. Добавьте **https://localhost:44368/** в поле **URI перенаправления** и выберите **Зарегистрировать**.
> 1. В разделе **Управление** выберите **Проверка подлинности**.
> 1. В разделе **Неявное предоставление разрешения и гибридные потоки** выберите **Токены идентификатора**.
> 1. Щелкните **Сохранить**.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
> Чтобы пример кода, приведенный в этом кратком руководстве, работал, введите **https://localhost:44368/** в поле **URI перенаправления**.
>
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести это изменение для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-aspnet-webapp/green-check.png). Ваше приложение настроено с использованием этого атрибута.

#### <a name="step-2-download-the-project"></a>Шаг 2. Скачивание проекта

> [!div renderon="docs"]
> [Скачайте решение Visual Studio 2019](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-DotNet/archive/master.zip)

> [!div renderon="portal" class="sxs-lookup"]
> Запустите проект с помощью Visual Studio 2019.
> [!div renderon="portal" id="autoupdate" class="sxs-lookup nextstepaction"]
> [Скачивание примера кода](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-DotNet/archive/master.zip)

[!INCLUDE [active-directory-develop-path-length-tip](../../../includes/active-directory-develop-path-length-tip.md)]

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Шаг 3. Приложение настроено и готово к запуску
> Мы настроили проект, указав значения свойств приложения.

> [!div renderon="docs"]
> #### <a name="step-3-run-your-visual-studio-project"></a>Шаг 3. Выполнение проекта Visual Studio

1. Извлеките ZIP-файл в локальный каталог рядом с корневым. Например, извлеките в *C:\Azure-Samples*.
   
   Рекомендуется извлечь архив в каталог рядом с корнем диска, чтобы избежать ошибок, вызванных ограничениями длины пути в Windows.
2. Откройте решение в Visual Studio (*AppModelv2-WebApp-OpenIDConnect-DotNet.sln*).
3. В зависимости от версии Visual Studio может потребоваться щелкнуть правой кнопкой мыши проект **AppModelv2-WebApp-OpenIDConnect-DotNet** и выбрать **Восстановить пакеты NuGet**.
4. Откройте консоль диспетчера пакетов, выбрав пункт **Просмотр** > **Другие окна** > **Консоль диспетчера пакетов**. Затем выполните `Update-Package Microsoft.CodeDom.Providers.DotNetCompilerPlatform -r`.

> [!div renderon="docs"]
> 5. Измените файл *Web.config* и замените параметры `ClientId`, `Tenant` и `redirectUri` на:
>    ```xml
>    <add key="ClientId" value="Enter_the_Application_Id_here" />
>    <add key="Tenant" value="Enter_the_Tenant_Info_Here" />
>    <add key="redirectUri" value="https://localhost:44368/" />
>    ```
>    В этом коде:
>
>    - `Enter_the_Application_Id_here` — это идентификатор приложения (клиента) для регистрации ранее созданного приложения. Найдите идентификатор приложения (клиента) на странице приложения **Обзор** в разделе **Регистрация приложений** на портале Azure.
>    - Значение`Enter_the_Tenant_Info_Here` является одним из следующих вариантов:
>      - Если ваше приложение поддерживает вариант **Только моя организация**, замените это значение на идентификатор каталога (клиента) или имя клиента (например, `contoso.onmicrosoft.com`). Найдите идентификатор каталога (клиента) на странице приложения **Обзор** в разделе **Регистрация приложений** на портале Azure.
>      - Если приложение поддерживает вариант **Учетные записи в любом каталоге организации**, замените это значение на `organizations`.
>      - Если приложение поддерживает вариант **Все пользователи с учетными записями Майкрософт**, замените это значение на `common`.
>    - `redirectUri` — это **URI перенаправления**, введенный ранее в разделе **Регистрация приложений** на портале Azure.
>

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

## <a name="more-information"></a>Дополнительные сведения

В этом разделе представлен код, используемый для выполнения входа пользователей. Это может быть полезно для рассмотрения принципов работы кода и основных аргументов. Также вы поймете, нужно ли добавлять функцию входа в существующее приложение ASP.NET.

> [!div class="sxs-lookup" renderon="portal"]
> ### <a name="how-the-sample-works"></a>Как работает этот пример
>
> ![Схема взаимодействия между веб-браузером, веб-приложением и платформой удостоверений Майкрософт в примере приложения.](media/quickstart-v2-aspnet-webapp/aspnetwebapp-intro.svg)

### <a name="owin-middleware-nuget-packages"></a>Пакеты NuGet для ПО промежуточного слоя OWIN

Вы можете настроить конвейер проверки подлинности, используя проверку подлинности на основе файлов cookie, с помощью OpenID Connect в ASP.NET и пакетов ПО промежуточного слоя OWIN. Эти пакеты можно установить, выполнив в консоли диспетчера пакетов Visual Studio следующие команды:

```powershell
Install-Package Microsoft.Owin.Security.OpenIdConnect
Install-Package Microsoft.Owin.Security.Cookies
Install-Package Microsoft.Owin.Host.SystemWeb
```

### <a name="owin-startup-class"></a>Класс Startup OWIN

ПО промежуточного слоя OWIN использует *класс startup*, выполняемый при запуске процесса размещения. В этом кратком руководстве используется файл *startup.cs*, расположенный в корневом каталоге. В следующем коде показан параметр, используемый в этом кратком руководстве:

```csharp
public void Configuration(IAppBuilder app)
{
    app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

    app.UseCookieAuthentication(new CookieAuthenticationOptions());
    app.UseOpenIdConnectAuthentication(
        new OpenIdConnectAuthenticationOptions
        {
            // Sets the client ID, authority, and redirect URI as obtained from Web.config
            ClientId = clientId,
            Authority = authority,
            RedirectUri = redirectUri,
            // PostLogoutRedirectUri is the page that users will be redirected to after sign-out. In this case, it's using the home page
            PostLogoutRedirectUri = redirectUri,
            Scope = OpenIdConnectScope.OpenIdProfile,
            // ResponseType is set to request the code id_token, which contains basic information about the signed-in user
            ResponseType = OpenIdConnectResponseType.CodeIdToken,
            // ValidateIssuer set to false to allow personal and work accounts from any organization to sign in to your application
            // To only allow users from a single organization, set ValidateIssuer to true and the 'tenant' setting in Web.config to the tenant name
            // To allow users from only a list of specific organizations, set ValidateIssuer to true and use the ValidIssuers parameter
            TokenValidationParameters = new TokenValidationParameters()
            {
                ValidateIssuer = false // Simplification (see note below)
            },
            // OpenIdConnectAuthenticationNotifications configures OWIN to send notification of failed authentications to the OnAuthenticationFailed method
            Notifications = new OpenIdConnectAuthenticationNotifications
            {
                AuthenticationFailed = OnAuthenticationFailed
            }
        }
    );
}
```

> |Where  | Описание |
> |---------|---------|
> | `ClientId`     | Идентификатор приложения, зарегистрированного на портале Azure. |
> | `Authority`    | Конечная точка службы токенов безопасности для проверки подлинности пользователей. Для общедоступного облака это обычно `https://login.microsoftonline.com/{tenant}/v2.0`. В этом URL-адресе *{tenant}*  — это имя вашего клиента, идентификатор вашего клиента или значение `common` для ссылки на общую конечную точку. (Общая конечная точка используется для мультитенантных приложений.) |
> | `RedirectUri`  | URL-адрес, по которому пользователи переходят после проверки подлинности на платформе удостоверений Майкрософт. |
> | `PostLogoutRedirectUri`     | URL-адрес, куда пользователи переходят после выхода. |
> | `Scope`     | Список запрашиваемых областей, разделенных пробелами. |
> | `ResponseType`     | Запрос на то, чтобы ответ от проверки подлинности содержал код авторизации и маркер идентификации. |
> | `TokenValidationParameters`     | Список параметров для проверки маркеров. В этом случае для `ValidateIssuer` задано значение `false`, чтобы указать, что приложение может принимать операции входа под любыми типами личных, рабочих или учебных учетных записей. |
> | `Notifications`     | Список делегатов, которые можно запускать для сообщений `OpenIdConnect`. |


> [!NOTE]
> Параметр `ValidateIssuer = false` приводится в этом кратком руководстве для упрощения. В реальных приложениях нужно проверять издателя. См. примеры кода, чтобы узнать, как это сделать.

### <a name="authentication-challenge"></a>Запрос проверки подлинности

Вы можете настроить принудительный вход пользователя, настроив запрос проверки подлинности в контроллере:

```csharp
public void SignIn()
{
    if (!Request.IsAuthenticated)
    {
        HttpContext.GetOwinContext().Authentication.Challenge(
            new AuthenticationProperties{ RedirectUri = "/" },
            OpenIdConnectAuthenticationDefaults.AuthenticationType);
    }
}
```

> [!TIP]
> Запрос проверки подлинности с помощью этого метода не является обязательным. Его обычно используют, если требуется, чтобы представление было доступно как для пользователей, которые прошли проверку подлинности, так и для тех, которые еще не прошли. Кроме того, можно защитить контроллеры, используя метод, описанный в следующем разделе.

### <a name="attribute-for-protecting-a-controller-or-a-controller-actions"></a>Атрибут защиты контроллера или его действий

Контроллер или его действия можно защитить с помощью атрибута `[Authorize]`. Этот атрибут ограничивает доступ к контроллеру или действиям, разрешая доступ к действиям контроллера только пользователям, прошедшим проверку подлинности. После этого запрос на проверку подлинности будет выполняться автоматически, когда не прошедший проверку подлинности пользователь попытается получить доступ к одному из действий или контроллеров, обозначенных атрибутом `[Authorize]`.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

В руководстве по ASP.NET вы найдете пошаговые инструкции по созданию приложений и функций, а также полное описание того, о чем говорится в этом кратком руководстве.

> [!div class="nextstepaction"]
> [Реализация входа в веб-приложение ASP.NET с использованием учетной записи Майкрософт](tutorial-v2-asp-webapp.md)
