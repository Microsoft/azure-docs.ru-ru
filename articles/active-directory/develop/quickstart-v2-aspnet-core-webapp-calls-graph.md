---
title: Краткое руководство. Веб-приложение ASP.NET Core, выполняющее вход пользователей в систему и вызывающее Microsoft Graph | Azure
titleSuffix: Microsoft identity platform
description: В этом кратком руководстве содержатся сведения об использовании приложения Microsoft.Identity.Web для реализации входа Microsoft в веб-приложение ASP.NET Core с помощью OpenID Connect и вызовов Microsoft Graph
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 12/10/2020
ms.author: jmprieur
ms.custom: devx-track-csharp, aaddev, scenarios:getting-started, languages:aspnet-core
ms.openlocfilehash: efa9465adc13b50e6ae12628d21347152c3fc2c0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104578725"
---
# <a name="quickstart-aspnet-core-web-app-that-signs-in-users-and-calls-microsoft-graph-on-their-behalf"></a>Краткое руководство. Веб-приложение ASP.NET Core, выполняющее вход пользователей в систему и вызывающее Microsoft Graph от их имени

В рамках этого краткого руководства вы загрузите и запустите пример кода, который демонстрирует выполнение веб-приложением ASP.NET Core входа в систему пользователей из любой организации Azure Active Directory (Azure AD) и вызовов Microsoft Graph.  

Иллюстрацию см. в разделе [Как работает этот пример](#how-the-sample-works).

> [!div renderon="docs"]
> ## <a name="prerequisites"></a>Предварительные требования
>
> * [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) или [Visual Studio Code](https://code.visualstudio.com/)
> * [Пакет SDK для .NET Core 3.1 +](https://dotnet.microsoft.com/download)
>
> ## <a name="register-and-download-the-quickstart-app"></a>Регистрация и скачивание приложения быстрого запуска
> У вас есть два варианта запуска приложения, используемого в этом кратком руководстве:
> * [Экспресс-способ] [Вариант 1. Регистрация и автоматическая настройка приложения, а затем скачивание примера кода](#option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample)
> * [Вручную] [Вариант 2. Регистрация и настройка приложения и примера кода вручную](#option-2-register-and-manually-configure-your-application-and-code-sample)
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>Вариант 1. Регистрация и автоматическая настройка приложения, а затем скачивание примера кода
>
> 1. Откройте страницу <a href="https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/AspNetCoreWebAppQuickstartPage/sourceType/docs" target="_blank">регистрации приложений</a> на портале Azure и приступите к работе.
> 1. Введите имя приложения и нажмите кнопку **Зарегистрировать**.
> 1. Следуйте инструкциям для загрузки и автоматической настройки нового приложения одним щелчком мыши.
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>Вариант 2. Регистрация и настройка приложения и примера кода вручную
>
> #### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения
> Чтобы зарегистрировать приложение и добавить сведения о его регистрации в решение вручную, сделайте следующее:
>
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
> 1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
> 1. Найдите и выберите **Azure Active Directory**.
> 1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
> 1. Введите **имя** приложения, например `AspNetCoreWebAppCallsGraph-Quickstart`. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
> 1. Введите **URI перенаправления** `https://localhost:44321/signin-oidc`.
> 1. Выберите **Зарегистрировать**.
> 1. В разделе **Управление** выберите **Проверка подлинности**.
> 1. Задайте для параметра **URL-адрес выхода Front-channel** значение `https://localhost:44321/signout-oidc`.
> 1. Нажмите **Сохранить**.
> 1. В разделе **Управление** выберите **Сертификаты и секреты** > **Создать секрет клиента**.
> 1. Введите **описание**, например `clientsecret1`.
> 1. Для истечения срока действия секрета выберите значение **Через один год**.
> 1. Выберите **Добавить** и сразу же запишите **значение** секрета для использования на следующем этапе. Значение секрета *больше не будет отображаться*. Его невозможно восстановить никаким другим способом. Запишите его в надежном месте как обычный пароль.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
> Чтобы пример кода, приведенный в этом кратком руководстве, работал, добавьте **URI перенаправления** `https://localhost:44321/signin-oidc` и **URL-адрес выхода Front-channel** `https://localhost:44321/signout-oidc` в регистрации приложения.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести это изменение для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-aspnet-webapp/green-check.png). Ваше приложение настроено с помощью этих атрибутов.

#### <a name="step-2-download-the-aspnet-core-project"></a>Шаг 2. Скачивание проекта ASP.NET Core

> [!div renderon="docs"]
> [Скачивание решения ASP.NET Core](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/archive/aspnetcore3-1-callsgraph.zip)

> [!div renderon="portal" class="sxs-lookup"]
> Запустите проект.

> [!div renderon="portal" class="sxs-lookup" id="autoupdate" class="nextstepaction"]
> [Скачивание примера кода](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/archive/aspnetcore3-1-callsgraph.zip)

[!INCLUDE [active-directory-develop-path-length-tip](../../../includes/active-directory-develop-path-length-tip.md)]

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Шаг 3. Приложение настроено и готово к запуску
> Мы уже настроили для проекта нужные значения свойств приложения, и его можно запускать.
> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`
> [!div renderon="docs"]
> #### <a name="step-3-configure-your-aspnet-core-project"></a>Шаг 3. Настройка проекта ASP.NET Core
> 1. Извлеките ZIP-архив в локальную папку рядом с корнем диска. Например, в папку *C:\Azure-Samples*.
> 1. Откройте решение в Visual Studio 2019.
> 1. Откройте файл *appsettings.json* и измените следующие значения:
>
>    ```json
>    "ClientId": "Enter_the_Application_Id_here",
>    "TenantId": "common",
>    "clientSecret": "Enter_the_Client_Secret_Here"
>    ```
>
>    - Замените `Enter_the_Application_Id_here`  **идентификатором приложения (клиента)** , зарегистрированного на портале Azure. **Идентификатор приложения (клиента)** можно найти на странице **Обзор** приложения.
>    - Замените `common` одним из следующих элементов:
>       - Если приложение поддерживает вариант **Учетные записи только в этом каталоге организации**, замените это значение **идентификатором каталога (клиента)** (GUID) или **именем клиента** (например, `contoso.onmicrosoft.com`). Вы можете найти **идентификатор каталога (клиента)** на странице **Обзор** приложения.
>       - Если ваше приложение поддерживает вариант **Учетные записи в любом каталоге организации**, замените это значение на `organizations`.
>       - Если приложение поддерживает вариант **Все пользователи с учетными записями Майкрософт**, оставьте это значение как `common`
>    - Замените `Enter_the_Client_Secret_Here` на **секрет клиента**, который вы создали и записали на предыдущем шаге.
> 
> Для целей этого руководства изменение других значений в файле *appsettings.json* не требуется.
>
> #### <a name="step-4-build-and-run-the-application"></a>Шаг 4. Сборка и запуск приложения
>
> Выполните сборку и запустите приложение в Visual Studio, выбрав меню **Отладка** > **Начать отладку** или нажав кнопку `F5`.
>
> Вы увидите запрос на ввод учетных данных, а затем запрос на предоставление согласия на разрешения, необходимые вашему приложению. Выберите **Принять** в запросе на согласие.
>
> :::image type="content" source="media/quickstart-v2-aspnet-core-webapp-calls-graph/webapp-01-consent.png" alt-text="Диалоговое окно согласия с разрешениями, запрашиваемыми приложением от > пользователя":::
>
> После принятия запрошенных разрешений приложение отобразит сведения об успешном входе в систему с использованием учетных данных Azure Active Directory и вы увидите свой адрес электронной почты в разделе API Result (Результат API) на странице. Он был извлечен с помощью Microsoft Graph.
>
> :::image type="content" source="media/quickstart-v2-aspnet-core-webapp-calls-graph/webapp-02-signed-in.png" alt-text="Веб-браузер, отображающий работающее веб-приложение и вошедшего в него пользователя":::

## <a name="about-the-code"></a>Примечания о коде

В этом разделе приведен обзор кода, необходимого для выполнения входа пользователей в систему и вызова API Microsoft Graph от их имени. В этом обзоре содержатся сведения о работе кода, основных аргументах, а также о добавлении входа в имеющееся приложение ASP.NET Core и вызове Microsoft Graph. В нем используется библиотека [Microsoft.Identity.Web](microsoft-identity-web.md), являющаяся оболочкой для [MSAL.NET](msal-overview.md).

### <a name="how-the-sample-works"></a>Как работает этот пример
![Схема работы приложения, создаваемого в этом кратком руководстве](media/quickstart-v2-aspnet-core-webapp-calls-graph/aspnetcorewebapp-intro.svg)

### <a name="startup-class"></a>Класс Startup

ПО промежуточного слоя *Microsoft.AspNetCore.Authentication* использует класс `Startup`, который выполняется при инициализации хост-процесса:

```csharp
  // Get the scopes from the configuration (appsettings.json)
  var initialScopes = Configuration.GetValue<string>("DownstreamApi:Scopes")?.Split(' ');

  public void ConfigureServices(IServiceCollection services)
  {
      // Add sign-in with Microsoft
      services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
        .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAd"))

            // Add the possibility of acquiring a token to call a protected web API
            .EnableTokenAcquisitionToCallDownstreamApi(initialScopes)

                // Enables controllers and pages to get GraphServiceClient by dependency injection
                // And use an in memory token cache
                .AddMicrosoftGraph(Configuration.GetSection("DownstreamApi"))
                .AddInMemoryTokenCaches();

      services.AddControllersWithViews(options =>
      {
          var policy = new AuthorizationPolicyBuilder()
              .RequireAuthenticatedUser()
              .Build();
          options.Filters.Add(new AuthorizeFilter(policy));
      });

      // Enables a UI and controller for sign in and sign out.
      services.AddRazorPages()
          .AddMicrosoftIdentityUI();
  }
```

Метод `AddAuthentication()` настраивает службу для добавления аутентификации на основе файлов cookie, которая используется при работе с браузером, и определения запроса для OpenID Connect.

Строка, содержащая `.AddMicrosoftIdentityWebApp`, добавляет в ваше приложение функцию аутентификации платформы удостоверений Майкрософт. Этот процесс выполняет [Microsoft.Identity.Web](microsoft-identity-web.md). Затем она настраивается для входа с использованием платформы удостоверений Майкрософт на основе сведений в разделе `AzureAD` файла конфигурации *appsettings.json*:

| Ключ *appsettings.json* | Описание                                                                                                                                                          |
|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ClientId`             | **Идентификатор приложения (клиента)** , зарегистрированного на портале Azure.                                                                                       |
| `Instance`             | Конечная точка службы токенов безопасности для проверки подлинности пользователей. Обычно это значение `https://login.microsoftonline.com/`, указывающее на общедоступное облако Azure. |
| `TenantId`             | Имя клиента, идентификатор клиента (GUID) или *общие* пользователи с рабочими или учебными учетными записями или личными учетными записями Майкрософт.                             |

Метод `EnableTokenAcquisitionToCallDownstreamApi` позволяет вашему приложению получить токен для вызова защищенных веб-API. `AddMicrosoftGraph` позволяет вашим контроллерам или страницам Razor напрямую использовать методы `GraphServiceClient` (путем внедрения зависимостей), а методы `AddInMemoryTokenCaches` позволяют вашему приложению воспользоваться преимуществами кэширования токенов.

Метод `Configure()` содержит два важных метода: `app.UseAuthentication()` и `app.UseAuthorization()`, которые обеспечивают упомянутые функциональные возможности. Кроме того, в методе `Configure()` необходимо зарегистрировать маршруты Microsoft Identity Web по крайней мере с одним вызовом `endpoints.MapControllerRoute()` или вызовом `endpoints.MapControllers()`.

```csharp
app.UseAuthentication();
app.UseAuthorization();

app.UseEndpoints(endpoints =>
{

    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
    endpoints.MapRazorPages();
});

// endpoints.MapControllers(); // REQUIRED if MapControllerRoute() isn't called.
```

### <a name="protect-a-controller-or-a-controllers-method"></a>Защита контроллера или метода контроллера

Вы можете защитить контроллер или его методы, применив атрибут `[Authorize]` к классу контроллера или одному/нескольким его методам. Этот атрибут `[Authorize]` ограничивает доступ, предоставляя его лишь пользователям, прошедшим проверку подлинности. Если пользователь еще не прошел проверку подлинности, для доступа к контроллеру может быть запущен запрос на проверку подлинности. В этом кратком руководстве области считываются из файла конфигурации:

```CSharp
[AuthorizeForScopes(ScopeKeySection = "DownstreamApi:Scopes")]
public async Task<IActionResult> Index()
{
    var user = await _graphServiceClient.Me.Request().GetAsync();
    ViewData["ApiResult"] = user.DisplayName;

    return View();
}
 ```

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Репозиторий GitHub, содержащий пример кода ASP.NET Core, упомянутый в этом кратком руководстве, содержит также инструкции и дополнительные примеры кода для выполнения следующих операций:

- Добавление проверки подлинности в новое веб-приложение ASP.NET Core
- Вызов Microsoft Graph, других API Майкрософт или собственных веб-API
- Добавление авторизации
- Вход пользователей в национальные облака или вход с помощью удостоверений социальных сетей

> [!div class="nextstepaction"]
> [Учебники по веб-приложениям ASP.NET Core в GitHub](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/)
