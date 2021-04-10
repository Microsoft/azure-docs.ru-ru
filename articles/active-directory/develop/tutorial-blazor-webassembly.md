---
title: Руководство. Выполнение входа от имени пользователя и вызов защищенного API из приложения Blazor WebAssembly
titleSuffix: Microsoft identity platform
description: В этом руководстве показано, как выполнить вход от имени пользователя и вызвать защищенный API из приложения Blazor WebAssembly (WASM) с помощью платформы удостоверений Майкрософт.
author: knicholasa
ms.author: nichola
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.date: 10/16/2020
ms.openlocfilehash: 47b4f36aec9a906317a9704a7d73bf66385d9e88
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102552122"
---
# <a name="tutorial-sign-in-users-and-call-a-protected-api-from-a-blazor-webassembly-app"></a>Руководство. Выполнение входа от имени пользователя и вызов защищенного API из приложения Blazor WebAssembly

Из этого руководства вы узнаете, как создать приложение Blazor WebAssembly с поддержкой входа пользователей и получения данных из Microsoft Graph, используя платформу удостоверений Майкрософт и зарегистрировав приложение в Azure Active Directory (Azure AD). 

В этом руководстве рассматриваются следующие темы:

> [!div class="checklist"]
>
> * создание нового приложения Blazor WebAssembly, в котором настроено использование Azure Active Directory (Azure AD) для [аутентификации и авторизации](authentication-vs-authorization.md) с помощью платформы удостоверений Майкрософт;
> * получение данных из защищенного веб-API (на примере [Microsoft Graph](/graph/overview)).

В этом руководстве используется .NET Core 3.1. Документация по .NET содержит инструкции по [защите приложения Blazor WebAssembly](/aspnet/core/blazor/security/webassembly/graph-api) с помощью ASP.NET Core 5.0. 

Кроме того, у нас есть руководство по [Blazor Server](tutorial-blazor-server.md). 

## <a name="prerequisites"></a>Предварительные требования

* [Пакет SDK для .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1)
* Арендатор Azure AD, на котором можно зарегистрировать приложение. Если у вас нет доступа к арендатору Azure AD, вы можете получить его, зарегистрировавшись в [Программе для разработчиков Microsoft 365](https://developer.microsoft.com/microsoft-365/dev-program) или создав [бесплатную учетную запись Azure](https://azure.microsoft.com/free).

## <a name="register-the-app-in-the-azure-portal"></a>Регистрация приложения на портале Azure

Любое приложение, использующее Azure Active Directory (Azure AD) для проверки подлинности, необходимо зарегистрировать в Azure AD. Выполните инструкции, приведенные в статье [Краткое руководство. Регистрация приложения с помощью платформы удостоверений Майкрософт](quickstart-register-app.md), сделав следующее:

- Для параметра **Поддерживаемые типы учетных записей** выберите **Учетные записи только в этом каталоге организации**.
- В раскрывающемся списке **URI перенаправления** оставьте значение **Веб** и введите `https://localhost:5001/authentication/login-callback`. Порт по умолчанию для приложения, работающего на Kestrel, — 5001. Если приложение доступно на другом порте, укажите его вместо `5001`.

После регистрации в разделе **Управление** выберите **Проверка подлинности** > **Implicit grant and hybrid flows** (Неявное предоставление разрешения и гибридные потоки). Выберите элементы **Маркеры доступа** и **Токен идентификатора**, а затем нажмите кнопку **Сохранить**.

## <a name="create-the-app-using-the-net-core-cli"></a>Создание приложения с помощью .NET Core CLI

Чтобы создать приложение, вам понадобятся последние шаблоны Blazor. Их можно установить для .NET Core CLI с помощью следующей команды:

```dotnetcli
dotnet new -i Microsoft.Identity.Web.ProjectTemplates::1.6.0
```

Затем выполните следующую команду, чтобы создать приложение. Замените заполнители в команде соответствующими данными со страницы обзора приложения и запустите выполнение команды в командной оболочке. Расположение выходных данных, указанное в параметре `-o|--output`, создает папку проекта, если она не существует, и внедряется в имя приложения.

```dotnetcli
dotnet new blazorwasm2 --auth SingleOrg --calls-graph -o {APP NAME} --client-id "{CLIENT ID}" --tenant-id "{TENANT ID}"
```

| Заполнитель   | Название на портале Azure       | Пример                                |
| ------------- | ----------------------- | -------------------------------------- |
| `{APP NAME}`  | &mdash;                 | `BlazorWASMSample`                         |
| `{CLIENT ID}` | Идентификатор приложения (клиента) | `41451fa7-0000-0000-0000-69eff5a761fd` |
| `{TENANT ID}` | Идентификатор каталога (клиента)   | `e86c78e2-0000-0000-0000-918e0565a45e` |

## <a name="test-the-app"></a>Тестирование приложения

Теперь можно выполнить сборку и запуск приложения. При запуске этого шаблона приложения необходимо указать с помощью параметра --framework платформу для запуска. В этом руководстве используется .NET Standard 2.1, но шаблон поддерживает и другие платформы.

```dotnetcli
dotnet run --framework netstandard2.1
```

В браузере перейдите по адресу `https://localhost:5001` и выполните вход с учетной записью пользователя Azure AD, чтобы проверить работу приложения и обработку входа пользователей с помощью платформы удостоверений Майкрософт.

Компоненты этого шаблона, которые поддерживают вход с использованием Azure AD и платформы удостоверений Майкрософт, описаны в [документации по ASP.NET](/aspnet/core/blazor/security/webassembly/standalone-with-azure-active-directory#authentication-package).

## <a name="retrieving-data-from-a-protected-api-microsoft-graph"></a>Извлечение данных из защищенного API (Microsoft Graph)

[Microsoft Graph](/graph/overview) включает интерфейсы API, которые предоставляют доступ к данным Microsoft 365 для пользователей, и поддерживает маркеры, выданные платформой удостоверений Майкрософт, благодаря чему этот API хорошо использовать в качестве примера. В этом разделе описано, как добавить код для вызова Microsoft Graph и отображения адресов электронной почты пользователей на странице Fetch data (Получение данных) в приложении.

Этот раздел написан с использованием общего подхода к вызову защищенного API с помощью именованного клиента. Этот же метод можно использовать для других защищенных API, которые вы хотите вызывать. Но если вы планируете вызывать Microsoft Graph из приложения, вы можете использовать пакет SDK Graph для уменьшения объема кода. В документации по .NET приведены инструкции по [использованию пакета SDK Graph](/aspnet/core/blazor/security/webassembly/graph-api?view=aspnetcore-5.0&preserve-view=true).

Прежде чем начать, выйдите из приложения, так как вы будете вносить изменения в обязательные разрешения, и текущий маркер не будет работать. Если вы этого еще не сделали, прежде чем обновлять код ниже, запустите приложение еще раз и выберите **Выйти**.

Теперь вы обновите регистрацию и код приложения, чтобы извлекать электронную почту пользователя и отображать сообщения в приложении.

Сначала добавьте разрешение API `Mail.Read` в регистрацию приложения, чтобы служба Azure AD знала, что приложение будет запрашивать доступ к электронной почте своих пользователей.

1. На портале Azure выберите приложение в разделе **Регистрация приложений**.
1. В разделе **Управление** выберите **Разрешения API**.
1. Выберите **Добавить разрешение** > **Microsoft Graph**.
1. Выберите **Делегированные разрешения**, а затем найдите и выберите разрешение **Mail.Read**.
1. Выберите **Добавить разрешения**.

Затем добавьте следующий текст в файл *.csproj* проекта в разделе **ItemGroup** netstandard2.1. Это позволит создать на следующем шаге пользовательский экземпляр HttpClient.

```xml
<PackageReference Include="Microsoft.Extensions.Http" Version="3.1.7" />
```

Затем измените код, как указано в следующих шагах. Эти изменения добавляют [маркеры доступа](access-tokens.md) к исходящим запросам, которые отправляются в API Microsoft Graph. Этот подход более подробно описан в статье [Сценарии обеспечения дополнительной безопасности Blazor WebAssembly для ASP.NET Core](/aspnet/core/blazor/security/webassembly/additional-scenarios).

Сначала создайте новый файл с именем *GraphAPIAuthorizationMessageHandler.cs*, используя следующий код. Этот обработчик будет использоваться для добавления маркера доступа для областей `User.Read` и `Mail.Read` в исходящих запросах к API Microsoft Graph.

```csharp
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class GraphAPIAuthorizationMessageHandler : AuthorizationMessageHandler
{
    public GraphAPIAuthorizationMessageHandler(IAccessTokenProvider provider,
        NavigationManager navigationManager)
        : base(provider, navigationManager)
    {
        ConfigureHandler(
            authorizedUrls: new[] { "https://graph.microsoft.com" },
            scopes: new[] { "https://graph.microsoft.com/User.Read", "https://graph.microsoft.com/Mail.Read" });
    }
}
```

Замените содержимое метода `Main` в файле *Program.cs* приведенным ниже кодом. Этот код использует новый экземпляр `GraphAPIAuthorizationMessageHandler` и добавляет `User.Read` и `Mail.Read` в качестве областей по умолчанию, которые приложение будет запрашивать при первом входе пользователя в систему.

```csharp
var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("app");

builder.Services.AddScoped<GraphAPIAuthorizationMessageHandler>();

builder.Services.AddHttpClient("GraphAPI",
        client => client.BaseAddress = new Uri("https://graph.microsoft.com"))
    .AddHttpMessageHandler<GraphAPIAuthorizationMessageHandler>();

builder.Services.AddMsalAuthentication(options =>
{
    builder.Configuration.Bind("AzureAd", options.ProviderOptions.Authentication);
    options.ProviderOptions.DefaultAccessTokenScopes.Add("User.Read");
    options.ProviderOptions.DefaultAccessTokenScopes.Add("Mail.Read");
});

await builder.Build().RunAsync();
```

Наконец, замените содержимое страницы *FetchData.razor* следующим кодом. Этот код извлекает данные электронной почты пользователя из API Microsoft Graph и отображает их в виде списка. В `OnInitializedAsync` создается новый экземпляр `HttpClient`, который создает подходящий маркер доступа и использует его для выполнения запроса к API Microsoft Graph.

```c#
@page "/fetchdata"
@using System.ComponentModel.DataAnnotations
@using System.Text.Json.Serialization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@using Microsoft.Extensions.Logging
@inject IAccessTokenProvider TokenProvider
@inject IHttpClientFactory ClientFactory
@inject IHttpClientFactory HttpClientFactory

<p>This component demonstrates fetching data from a service.</p>

@if (messages == null)
{
    <p><em>Loading...</em></p>
}
else
{
    <h1>Hello @userDisplayName !!!!</h1>
    <table class="table">
        <thead>
            <tr>
                <th>Subject</th>
                <th>Sender</th>
                <th>Received Time</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var mail in messages)
            {
                <tr>
                    <td>@mail.Subject</td>
                    <td>@mail.Sender</td>
                    <td>@mail.ReceivedTime</td>
                </tr>
            }
        </tbody>
    </table>
}

@code {

    private string userDisplayName;
    private List<MailMessage> messages = new List<MailMessage>();

    private HttpClient _httpClient;

    protected override async Task OnInitializedAsync()
    {
        _httpClient = HttpClientFactory.CreateClient("GraphAPI");
        try {
            var dataRequest = await _httpClient.GetAsync("https://graph.microsoft.com/beta/me");

            if (dataRequest.IsSuccessStatusCode)
            {
                var userData = System.Text.Json.JsonDocument.Parse(await dataRequest.Content.ReadAsStreamAsync());
                userDisplayName = userData.RootElement.GetProperty("displayName").GetString();
            }

            var mailRequest = await _httpClient.GetAsync("https://graph.microsoft.com/beta/me/messages?$select=subject,receivedDateTime,sender&$top=10");

            if (mailRequest.IsSuccessStatusCode)
            {
                var mailData = System.Text.Json.JsonDocument.Parse(await mailRequest.Content.ReadAsStreamAsync());
                var messagesArray = mailData.RootElement.GetProperty("value").EnumerateArray();

                foreach (var m in messagesArray)
                {
                    var message = new MailMessage();
                    message.Subject = m.GetProperty("subject").GetString();
                    message.Sender = m.GetProperty("sender").GetProperty("emailAddress").GetProperty("address").GetString();
                    message.ReceivedTime = m.GetProperty("receivedDateTime").GetDateTime();
                    messages.Add(message);
                }
            }
        }
        catch (AccessTokenNotAvailableException ex)
        {
            // Tokens are not valid - redirect the user to log in again
            ex.Redirect();
        }
    }

    public class MailMessage
    {
        public string Subject;
        public string Sender;
        public DateTime ReceivedTime;
    }
}
```

Теперь запустите приложение еще раз. Вы увидите запрос на предоставление приложению доступа на чтение почты. Это ожидаемое поведение, когда приложение запрашивает область `Mail.Read`.

Дав свое согласие, перейдите на страницу Fetch data (Получение данных), чтобы прочесть электронную почту.

:::image type="content" source="./media/tutorial-blazor-webassembly/final-app.png" alt-text="Снимок экрана конечного приложения. У него есть заголовок с текстом Hello Nicholas (Здравствуйте, Николай), в котором отображается список сообщений электронной почты, отправленных Николаю.":::

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Рекомендации и советы по платформе удостоверений Майкрософт](./identity-platform-integration-checklist.md)
