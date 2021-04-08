---
title: Краткое руководство. Добавление возможности входа в веб-приложение ASP.NET Core с помощью учетной записи Майкрософт | Azure
titleSuffix: Microsoft identity platform
description: Из этого краткого руководства вы узнаете, как реализовать вход с помощью учетной записи Майкрософт в веб-приложении ASP.NET Core, используя OpenID Connect.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 09/11/2020
ms.author: jmprieur
ms.custom: devx-track-csharp, aaddev, identityplatformtop40, scenarios:getting-started, languages:aspnet-core
ms.openlocfilehash: e7296b04e3e912e96ac8c2ed77b44288324c262f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104578708"
---
# <a name="quickstart-add-sign-in-with-microsoft-to-an-aspnet-core-web-app"></a>Краткое руководство. Добавление возможности входа в веб-приложение ASP.NET Core с помощью учетной записи Майкрософт

Из этого краткого руководства вы узнаете, как скачать и выполнить пример кода, в котором показана реализация входа в веб-приложение ASP.NET Core для пользователей из любой организации Azure Active Directory (Azure AD).  

> [!div renderon="docs"]
> На следующей схеме показано, как работает пример приложения.
>
> ![Схема взаимодействия между веб-браузером, веб-приложением и платформой удостоверений Майкрософт в примере приложения.](media/quickstart-v2-aspnet-core-webapp/aspnetcorewebapp-intro.svg)
>
> ## <a name="prerequisites"></a>Предварительные требования
>
> * [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) или [Visual Studio Code](https://code.visualstudio.com/)
> * [Пакет SDK для .NET Core 3.1 +](https://dotnet.microsoft.com/download)
>
> ## <a name="register-and-download-the-app"></a>Регистрация и скачивание приложения
> Есть два варианта по созданию приложения: автоматическая или ручная настройка.
>
> ### <a name="automatic-configuration"></a>Автоматическая настройка
> Если вы хотите автоматически настроить приложение, а затем скачать пример кода, выполните следующие действия.
>
> 1. Перейдите на <a href="https://aka.ms/aspnetcore2-1-aad-quickstart-v2/" target="_blank">страницу портала Azure для регистрации приложения</a>.
> 1. Введите имя приложения и нажмите кнопку **Зарегистрировать**.
> 1. Следуйте инструкциям, чтобы скачать и автоматически настроить новое приложение одним щелчком мыши.
>
> ### <a name="manual-configuration"></a>Настройка вручную
> Если вы хотите вручную настроить приложение и пример кода, воспользуйтесь следующими процедурами.
> #### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
> 1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
> 1. Найдите и выберите **Azure Active Directory**.
> 1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
> 1. В поле **Имя** введите имя для своего приложения. Например, введите **AspNetCore-Quickstart**. Это имя будут видеть пользователи приложения. Вы сможете изменить его позже.
> 1. В поле **URI перенаправления** введите **https://localhost:44321/signin-oidc** .
> 1. Выберите **Зарегистрировать**.
> 1. В разделе **Управление** выберите **Проверка подлинности**.
> 1. Для параметра **Front-channel logout URL** (URL-адрес выхода Front-channel) введите значение **https://localhost:44321/signout-oidc** .
> 1. В разделе **Неявное предоставление разрешения и гибридные потоки** выберите **Маркеры идентификации**.
> 1. Щелкните **Сохранить**.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
> Чтобы пример кода, приведенный в этом кратком руководстве, работал, сделайте следующее:
> - В поле **URI перенаправления** введите **https://localhost:44321/** и **https://localhost:44321/signin-oidc** .
> - Для параметра **Front-channel logout URL** (URL-адрес выхода Front-channel) введите значение **https://localhost:44321/signout-oidc** . 
>
> Конечная точка авторизации будет выдавать маркеры идентификации запросов.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести это изменение для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-aspnet-webapp/green-check.png). Ваше приложение настроено с помощью этих атрибутов.

#### <a name="step-2-download-the-aspnet-core-project"></a>Шаг 2. Скачивание проекта ASP.NET Core

> [!div renderon="docs"]
> [Скачивание решения ASP.NET Core](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/archive/aspnetcore3-1.zip)

> [!div renderon="portal" class="sxs-lookup"]
> Запустите проект.

> [!div renderon="portal" class="sxs-lookup" id="autoupdate" class="nextstepaction"]
> [Скачивание примера кода](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/archive/aspnetcore3-1.zip)

[!INCLUDE [active-directory-develop-path-length-tip](../../../includes/active-directory-develop-path-length-tip.md)]

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Шаг 3. Приложение настроено и готово к запуску
> Мы уже настроили для проекта нужные значения свойств приложения, и его можно запускать.
> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`
> [!div renderon="docs"]
> #### <a name="step-3-configure-your-aspnet-core-project"></a>Шаг 3. Настройка проекта ASP.NET Core
> 1. Извлеките ZIP-архив в локальную папку рядом с корнем диска. Например, извлеките в *C:\Azure-Samples*.
> 
>    Рекомендуется извлечь архив в каталог рядом с корнем диска, чтобы избежать ошибок, вызванных ограничениями длины пути в Windows.
> 1. Откройте решение в Visual Studio 2019.
> 1. Откройте файл *appsettings.json* и измените следующий код:
>
>    ```json
>    "Domain": "Enter the domain of your tenant, e.g. contoso.onmicrosoft.com",
>    "ClientId": "Enter_the_Application_Id_here",
>    "TenantId": "common",
>    ```
>
>    - Замените `Enter_the_Application_Id_here` идентификатором приложения (клиента), зарегистрированного на портале Azure. Значение **идентификатора приложения (клиента)** можно найти на странице приложения **Обзор**.
>    - Замените `common` одним из следующих элементов:
>       - Если приложение поддерживает вариант **Учетные записи только в этом каталоге организации**, замените это значение идентификатором каталога (клиента) (уникальный идентификатор) или именем клиента (например, `contoso.onmicrosoft.com`). Значение **идентификатора каталога (клиента)** можно найти на странице приложения **Обзор**.
>       - Если приложение поддерживает вариант **Учетные записи в любом каталоге организации**, замените это значение на `organizations`.
>       - Если приложение поддерживает вариант **Все пользователи с учетными записями Майкрософт**, оставьте это значение как `common`.
>
> Для целей этого руководства изменение других значений в файле *appsettings.json* не требуется.
>
> #### <a name="step-4-build-and-run-the-application"></a>Шаг 4. Сборка и запуск приложения
>
> Выполните сборку и запустите приложение в Visual Studio, выбрав меню **Отладка** > **Начать отладку** или нажав клавишу F5.
>
> Сначала вы увидите запрос на ввод учетных данных, а затем запрос на предоставление разрешений, необходимых вашему приложению. Выберите **Принять** в запросе на согласие.
>
> :::image type="content" source="media/quickstart-v2-aspnet-core-webapp/webapp-01-consent.png" alt-text="Экран &quot;Снимок экрана: диалоговое окно согласия, на котором представлены разрешения, которые приложение запрашивает у пользователя&quot;.":::
>
> После предоставления согласия на запрошенные разрешения в приложении отобразится оповещение об успешном входе с использованием учетных данных Azure Active Directory.
>
> :::image type="content" source="media/quickstart-v2-aspnet-core-webapp/webapp-02-signed-in.png" alt-text="Экран &quot;Снимок экрана веб-браузера: работающее веб-приложение и вошедший в систему пользователь&quot;.":::

## <a name="more-information"></a>Дополнительные сведения

В этом разделе представлен код, используемый для выполнения входа пользователей. Это может быть полезно для рассмотрения принципов работы кода и основных аргументов. Вы также поймете, нужно ли добавлять функцию входа в существующее приложение ASP.NET Core.

> [!div class="sxs-lookup" renderon="portal"]
> ### <a name="how-the-sample-works"></a>Как работает этот пример
>
> ![Схема взаимодействия между веб-браузером, веб-приложением и платформой удостоверений Майкрософт в примере приложения.](media/quickstart-v2-aspnet-core-webapp/aspnetcorewebapp-intro.svg)

### <a name="startup-class"></a>Класс Startup

ПО промежуточного слоя *Microsoft.AspNetCore.Authentication* использует класс `Startup`, который выполняется при запуске хост-процесса.

```csharp
  public void ConfigureServices(IServiceCollection services)
  {
      services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
          .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAd"));

      services.AddControllersWithViews(options =>
      {
          var policy = new AuthorizationPolicyBuilder()
              .RequireAuthenticatedUser()
              .Build();
          options.Filters.Add(new AuthorizeFilter(policy));
      });
      services.AddRazorPages()
          .AddMicrosoftIdentityUI();
  }
```

С помощью метода `AddAuthentication()` службу можно настроить так, чтобы она добавляла проверку подлинности на основе файлов cookie. Эта проверка подлинности используется в сценариях браузера, а также для установки запроса OpenID Connect.

Строка, содержащая `.AddMicrosoftIdentityWebApp`, добавляет в ваше приложение функцию проверки подлинности платформы удостоверений Майкрософт. Затем приложение настраивается для входа пользователей на основе следующей информации в разделе `AzureAD` файла конфигурации *appsettings.json*:

| Ключ *appsettings.json* | Описание                                                                                                                                                          |
|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ClientId`             | Идентификатор приложения (клиента) , зарегистрированного на портале Azure.                                                                                       |
| `Instance`             | Конечная точка службы токенов безопасности для проверки подлинности пользователей. Обычно это значение `https://login.microsoftonline.com/`, указывающее на общедоступное облако Azure. |
| `TenantId`             | Имя клиента, идентификатор клиента (уникальный идентификатор) или `common` для входа пользователей под рабочими, учебными или личными учетными записями Майкрософт.                             |

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

### <a name="attribute-for-protecting-a-controller-or-methods"></a>Атрибут для защиты контроллера или методов

Контроллер или его методы можно защитить с помощью атрибута `[Authorize]`. Этот атрибут разрешает доступ к контроллеру или методам только тем пользователям, которые прошли проверку подлинности. Если пользователь еще не прошел проверку подлинности, при доступе к контроллеру может быть запущен запрос на проверку подлинности.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

В репозитории GitHub, содержащем это руководство ASP.NET Core, есть также инструкции и дополнительные примеры кода, демонстрирующие выполнение следующих действий.

- Добавление проверки подлинности в новое веб-приложение ASP.NET Core.
- Вызов Microsoft Graph, других API Майкрософт или собственных веб-API.
- Добавление авторизации.
- Вход пользователей в национальные облака или вход с помощью удостоверений социальных сетей.

> [!div class="nextstepaction"]
> [Учебники по веб-приложениям ASP.NET Core в GitHub](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/)
