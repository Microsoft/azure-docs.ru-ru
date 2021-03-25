---
title: Учебник. Доступ из веб-приложения к Microsoft Graph от имени пользователя | Azure
description: Из этого учебника вы узнаете, как получить доступ к данным в Microsoft Graph для пользователя, выполнившего вход.
services: microsoft-graph, app-service-web
author: rwike77
manager: CelesteDG
ms.service: app-service-web
ms.topic: tutorial
ms.workload: identity
ms.date: 01/28/2021
ms.author: ryanwi
ms.reviewer: stsoneff
ms.custom: azureday1
ms.openlocfilehash: 3413c1a3f27b48c60ae730ad230c653928702faa
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99063388"
---
# <a name="tutorial-access-microsoft-graph-from-a-secured-app-as-the-user"></a>Руководство по получению доступа к Microsoft Graph из защищенного приложения от имени пользователя

Узнайте, как получить доступ к Microsoft Graph из веб-приложения, работающего в Службе приложений Azure.

:::image type="content" alt-text="Диаграмма, в которой показан доступ к Microsoft Graph." source="./media/scenario-secure-app-access-microsoft-graph/web-app-access-graph.svg" border="false":::

Предположим, вам нужно обращаться из веб-приложения к Microsoft Graph и выполнять какие-либо действия от имени пользователя, выполнившего вход. В этом разделе описывается предоставление веб-приложению делегированных разрешений и получение из Azure Active Directory (Azure AD) сведений о профиле пользователя, выполнившего вход.

В этом руководстве вы узнаете, как:

> [!div class="checklist"]
>
> * предоставить веб-приложению делегированные разрешения;
> * вызвать Microsoft Graph из веб-приложения для пользователя, выполнившего вход.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Обязательные условия

* Веб-приложение, выполняющееся в Службе приложений Azure, с [включенным модулем проверки подлинности или авторизации Службы приложений](scenario-secure-app-authentication-app-service.md).

## <a name="grant-front-end-access-to-call-microsoft-graph"></a>Предоставление доступа к интерфейсу для вызова Microsoft Graph

Теперь, когда вы включили проверку подлинности и авторизацию для веб-приложения, это веб-приложение зарегистрировано на платформе удостоверений Майкрософт и подключено к приложению Azure AD. На этом шаге вы предоставите веб-приложению разрешения для доступа к Microsoft Graph для пользователя. (Технически это выполняется через предоставление приложению Azure AD, сопоставленному с веб-приложением, разрешений на доступ к приложению Azure AD, сопоставленному с Microsoft Graph, для пользователя.)

В меню [портала Azure](https://portal.azure.com) выберите **Azure Active Directory** или выполните поиск по запросу **Azure Active Directory** на любой странице и выберите этот пункт.

Поочередно выберите **Регистрация приложений** > **Собственные приложения** > **View all applications in this directory** (Просмотреть все приложения в этом каталоге). Щелкните имя веб-приложения и выберите **Разрешения API**.

Щелкните **Добавить разрешения**, а затем выберите API-интерфейсы Майкрософт, а также Microsoft Graph.

Щелкните **Делегированные разрешения**, а затем выберите из списка **User.Read**. Выберите **Добавить разрешения**.

## <a name="configure-app-service-to-return-a-usable-access-token"></a>Настройка службы приложений для возвращения используемых маркеров доступа

Теперь у веб-приложения есть необходимые разрешения для доступа к Microsoft Graph от имени вошедшего в систему пользователя. На этом шаге настройте проверку подлинности и авторизацию в Службе приложений, чтобы получить пригодный к использованию маркер доступа для обращения к Microsoft Graph. Для этого шага требуется идентификатор клиента или приложения, присвоенный нижестоящей службе (Microsoft Graph). Идентификатор приложения для Microsoft Graph — *00000003-0000-0000-C000-000000000000*.

> [!IMPORTANT]
> Если вы не настроите в Службе приложений возврат пригодного к использованию маркера доступа, при вызове API Microsoft Graph из кода возникнет ошибка ```CompactToken parsing failed with error code: 80049217```.

Перейдите к [обозревателю ресурсов Azure](https://resources.azure.com/) и с помощью дерева ресурсов найдите нужное веб-приложение. URL-адрес ресурса должен выглядеть приблизительно так: `https://resources.azure.com/subscriptions/subscription-id/resourceGroups/SecureWebApp/providers/Microsoft.Web/sites/SecureWebApp20200915115914`.

Теперь откроется обозреватель ресурсов Azure с представлением дерева ресурсов, в котором выделено это веб-приложение. В верхней части страницы выберите **Чтение и запись**, чтобы внести изменения в обозревателе ресурсов Azure.

В браузере слева перейдите к разделу **config** > **authsettings**.

В представлении **authsettings** выберите **Изменить**. Установите значение ```additionalLoginParams``` в следующей строке JSON, используя идентификатор клиента, который вы скопировали.

```json
"additionalLoginParams": ["response_type=code id_token","resource=00000003-0000-0000-c000-000000000000"],
```

Сохраните настройки, выбрав **PUT**. Эти настройки могут вступить в силу через несколько минут. Теперь веб-приложение настроено для доступа к Microsoft Graph с использованием правильного маркера доступа. В противном случае Microsoft Graph вернет ошибку с сообщением о неправильном формате компактного маркера.

## <a name="call-microsoft-graph-net"></a>Вызов Microsoft Graph (.NET)

Теперь веб-приложение имеет требуемые разрешения и правильно включает идентификатор клиента Microsoft Graph в параметры входа. С помощью [библиотеки Microsoft.Identity.Web](https://github.com/AzureAD/microsoft-identity-web/) веб-приложение получает маркер доступа для проверки подлинности в Microsoft Graph. В версии 1.2.0 и более поздних библиотека Microsoft.Identity.Web поддерживает интеграцию и параллельную работу с модулем проверки подлинности и авторизации Службы приложений. Microsoft.Identity.Web определяет, что веб-приложение размещено в службах приложений и получает маркер доступа из модуля проверки подлинности и авторизации служб приложений. Затем этот маркер доступа передается в составе запросов, которые прошли проверку подлинности, в API Microsoft Graph.

Просмотреть этот код как часть примера приложения можно на [сайте GitHub](https://github.com/Azure-Samples/ms-identity-easyauth-dotnet-storage-graphapi/tree/main/2-WebApp-graphapi-on-behalf).

> [!NOTE]
> Библиотека Microsoft.Identity.Web не требуется в веб-приложении для обычной проверки подлинности и авторизации или для проверки подлинности запросов в Microsoft Graph. Вы можете [безопасно вызывать нижестоящие API](tutorial-auth-aad.md#call-api-securely-from-server-code), если включен только модуль проверки подлинности и авторизации Службы приложений.
> 
> Но проверка подлинности и авторизация Службы приложений предназначены только для самых простых сценариев проверки подлинности. Для более сложных сценариев (например, для обработки пользовательских утверждений) нужно использовать библиотеку Microsoft.Identity.Web или [библиотеку проверки подлинности Майкрософт](../active-directory/develop/msal-overview.md). Ее установка и настройка займет немного больше времени, но библиотека Microsoft.Identity.Web может работать параллельно с модулем проверки подлинности и авторизации Службы приложений. Позже, когда веб-приложению потребуется работа с более сложными сценариями, вы сможете отключить модуль проверки подлинности и авторизации Службы приложений, а Microsoft.Identity.Web останется частью приложения.

### <a name="install-client-library-packages"></a>Установка пакетов клиентских библиотек

Установите в проект пакеты NuGet [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) и [Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph) с помощью интерфейса командной строки .NET Core или консоли диспетчера пакетов в Visual Studio.

# <a name="command-line"></a>[Командная строка](#tab/command-line)

Откройте командную строку и перейдите в каталог с файлом проекта.

Выполните команды установки.

```dotnetcli
dotnet add package Microsoft.Identity.Web.MicrosoftGraph

dotnet add package Microsoft.Identity.Web
```

# <a name="package-manager"></a>[Диспетчер пакетов](#tab/package-manager)

Откройте проект или решение в Visual Studio, а затем — консоль, выбрав **Средства** > **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов**.

Выполните команды установки.
```powershell
Install-Package Microsoft.Identity.Web.MicrosoftGraph

Install-Package Microsoft.Identity.Web
```

---

### <a name="startupcs"></a>Startup.cs

В файле *Startup.cs* метод ```AddMicrosoftIdentityWebApp``` добавляет Microsoft.Identity.Web в веб-приложение. Метод ```AddMicrosoftGraph``` добавляет поддержку Microsoft Graph.

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Identity.Web;
using Microsoft.AspNetCore.Authentication.OpenIdConnect;

// Some code omitted for brevity.
public class Startup
{
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
                .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAd"))
                    .EnableTokenAcquisitionToCallDownstreamApi()
                        .AddMicrosoftGraph(Configuration.GetSection("Graph"))
                        .AddInMemoryTokenCaches();

        services.AddRazorPages();
    }
}

```

### <a name="appsettingsjson"></a>appsettings.json

*AzureAd* задает конфигурацию для библиотеки Microsoft.Identity.Web. На [портале Azure](https://portal.azure.com) выберите в меню портала **Azure Active Directory**, а затем — **Регистрация приложений**. Выберите регистрацию приложения, созданную при включении модуля проверки подлинности и авторизации Службы приложений. (Регистрация приложения должна иметь то же имя, что и веб-приложение.) Идентификаторы арендатора и клиента можно найти на странице обзора регистрации приложения. Доменное имя можно найти на странице обзора Azure AD для вашего арендатора.

*Graph* определяет конечную точку Microsoft Graph и начальные области, которые нужны для приложения.

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "fourthcoffeetest.onmicrosoft.com",
    "TenantId": "[tenant-id]",
    "ClientId": "[client-id]",
    // To call an API
    "ClientSecret": "[secret-from-portal]", // Not required by this scenario
    "CallbackPath": "/signin-oidc"
  },

  "Graph": {
    "BaseUrl": "https://graph.microsoft.com/v1.0",
    "Scopes": "user.read"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

### <a name="indexcshtmlcs"></a>Index.cshtml.cs

В следующем примере показано, как вызвать Microsoft Graph от имени выполнившего вход пользователя и получить сведения об этом пользователе. Объект ```GraphServiceClient``` внедряется в контроллер, а проверка подлинности уже автоматически настроена библиотекой Microsoft.Identity.Web.

```csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Graph;
using System.IO;
using Microsoft.Identity.Web;
using Microsoft.Extensions.Logging;

// Some code omitted for brevity.

[AuthorizeForScopes(Scopes = new[] { "user.read" })]
public class IndexModel : PageModel
{
    private readonly ILogger<IndexModel> _logger;
    private readonly GraphServiceClient _graphServiceClient;

    public IndexModel(ILogger<IndexModel> logger, GraphServiceClient graphServiceClient)
    {
        _logger = logger;
        _graphServiceClient = graphServiceClient;
    }

    public async Task OnGetAsync()
    {
        try
        {
            var user = await _graphServiceClient.Me.Request().GetAsync();
            ViewData["Me"] = user;
            ViewData["name"] = user.DisplayName;

            using (var photoStream = await _graphServiceClient.Me.Photo.Content.Request().GetAsync())
            {
                byte[] photoByte = ((MemoryStream)photoStream).ToArray();
                ViewData["photo"] = Convert.ToBase64String(photoByte);
            }
        }
        catch (Exception ex)
        {
            ViewData["photo"] = null;
        }
    }
}
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы завершили работу с этим учебником и вам больше не требуется веб-приложение или связанные с ним ресурсы, необходимо [очистить созданные ресурсы](scenario-secure-app-clean-up-resources.md).

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
>
> * предоставить веб-приложению делегированные разрешения;
> * вызвать Microsoft Graph из веб-приложения для пользователя, выполнившего вход.

> [!div class="nextstepaction"]
> [Доступ из Службы приложений к Microsoft Graph от имени приложения](scenario-secure-app-access-microsoft-graph-as-app.md)