---
title: Учебник по созданию приложения Blazor Server, использующего платформу удостоверений Майкрософт для аутентификации | Azure
titleSuffix: Microsoft identity platform
description: Из этого учебника вы узнаете, как настроить проверку подлинности с помощью платформы идентификации Майкрософт в приложении Blazor Server.
author: knicholasa
ms.author: nichola
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.date: 09/15/2020
ms.openlocfilehash: 5a631d9ae7a7d1792e3c4e4a2cbf8281e1168283
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99226022"
---
# <a name="tutorial-create-a-blazor-server-app-that-uses-the-microsoft-identity-platform-for-authentication"></a>Руководство по Создание приложения Blazor Server, которое использует платформу удостоверений Майкрософт для аутентификации

Из этого руководства вы узнаете, как создать приложение Blazor Server с поддержкой входа пользователей и получения данных из Microsoft Graph, используя платформу удостоверений Майкрософт и зарегистрировав приложение в Azure Active Directory (Azure AD).

У нас также есть руководство по [Blazor WASM](tutorial-blazor-webassembly.md).

В этом руководстве рассматриваются следующие темы:

> [!div class="checklist"]
> * Создание нового приложения Blazor Server, настроенного для использования Azure Active Directory (Azure AD) для проверки подлинности
> * Одновременное выполнение проверки подлинности и авторизации с помощью Microsoft.Identity.Web
> * Получение данных из защищенного веб-API Microsoft Graph

## <a name="prerequisites"></a>Предварительные требования

- [Пакет SDK для .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1)
- Арендатор Azure AD, на котором можно зарегистрировать приложение. Если у вас нет доступа к арендатору Azure AD, вы можете получить его, зарегистрировавшись в [Программе для разработчиков Microsoft 365](https://developer.microsoft.com/microsoft-365/dev-program) или создав [бесплатную учетную запись Azure](https://azure.microsoft.com/free).

## <a name="register-the-app-in-the-azure-portal"></a>Регистрация приложения на портале Azure

Любое приложение, использующее Azure Active Directory (Azure AD) для проверки подлинности, необходимо зарегистрировать в Azure AD. Следуйте инструкциям на странице [Регистрации приложения](quickstart-register-app.md), придерживаясь следующих указаний:

- Для параметра **Поддерживаемые типы учетных записей** выберите **Учетные записи только в этом каталоге организации**.
- В раскрывающемся списке **URI перенаправления** оставьте значение **Веб** и введите `https://localhost:5001/signin-oidc`. Порт по умолчанию для приложения, работающего на Kestrel, — 5001. Если приложение доступно на другом порте, укажите его вместо `5001`.

В разделе **Управление** выберите **Проверка подлинности** > **Implicit grant and hybrid flows** (Неявное предоставление разрешения и гибридные потоки). Выберите элементы **Маркеры доступа** и **Токен идентификатора**, а затем нажмите кнопку **Сохранить**.

Наконец, поскольку приложение вызывает защищенный API (в данном случае Microsoft Graph), ему требуется секрет клиента, чтобы проверить его удостоверение при получении запроса на маркер доступа для вызова этого API.

1. В рамках одной регистрации приложения в разделе **Управление** выберите **Сертификаты и секреты**.
2. Создайте бессрочный **секрет клиента**.
3. Запишите **значение** секрета так, как оно будет использоваться на следующем шаге. После выхода из этой области вы не сможете вернуться к ней снова. Однако при необходимости его можно создать повторно.

## <a name="create-the-app-using-the-net-cli"></a>Создание приложения с помощью .NET CLI

Воспользуйтесь следующей командой, чтобы скачать шаблоны для Microsoft.Identity.Web, которые мы будем использовать в этом учебнике.

```dotnetcli
dotnet new --install Microsoft.Identity.Web.ProjectTemplates::0.4.0-preview
```

Затем воспользуйтесь следующей командой, чтобы создать приложение. Замените заполнители в команде соответствующими данными со страницы обзора приложения и запустите выполнение команды в командной оболочке. Расположение выходных данных, указанное в параметре `-o|--output`, создает папку проекта, если она не существует, и внедряется в имя приложения.

```dotnetcli
dotnet new blazorserver2 --auth SingleOrg --calls-graph -o {APP NAME} --client-id "{CLIENT ID}" --tenant-id "{TENANT ID}"
```

| Заполнитель   | Название на портале Azure       | Пример                                |
| ------------- | ----------------------- | -------------------------------------- |
| `{APP NAME}`  | &mdash;                 | `BlazorSample`                         |
| `{CLIENT ID}` | Идентификатор приложения (клиента) | `41451fa7-0000-0000-0000-69eff5a761fd` |
| `{TENANT ID}` | Идентификатор каталога (клиента)   | `e86c78e2-0000-0000-0000-918e0565a45e` |

Теперь перейдите к новому приложению Blazor в редакторе и добавьте секрет клиента в файл *appsettings.json*, заменив текст "secret-from-app-registration".

```json
"ClientSecret": "secret-from-app-registration",
```

## <a name="test-the-app"></a>Тестирование приложения

Теперь можно выполнить сборку и запуск приложения. При запуске этого шаблона приложения необходимо указать с помощью параметра --framework платформу для запуска. В этом учебнике используется пакет SDK для .NET Core 3.1.

```dotnetcli
dotnet run --framework netcoreapp3.1
```

В браузере перейдите к `https://localhost:5001`, войдите в систему с помощью учетной записи пользователя Azure AD, и убедитесь, что приложение выполняется.

## <a name="retrieving-data-from-microsoft-graph"></a>Получение данных из Microsoft Graph

[Microsoft Graph](/graph/overview) предлагает ряд интерфейсов API, предоставляющих доступ к данным пользователей в Microsoft 365. Используя платформу удостоверений Майкрософт в качестве поставщика удостоверений для приложения, вы получаете более удобный доступ к этим сведениям, так как Microsoft Graph напрямую поддерживает маркеры, выданные этой платформой. В этом разделе описано, как добавить код для отображения сообщений электронной почты пользователя, выполнившего вход, на странице Fetch data (Получение данных) в приложении.

Прежде чем начать, выйдите из приложения, так как вы будете вносить изменения в обязательные разрешения, и текущий маркер не будет работать. Если вы этого еще не сделали, прежде чем обновлять код ниже, запустите приложение еще раз и выберите **Выйти**.

Теперь вы обновите регистрацию и код приложения, чтобы он извлекал электронную почту пользователя и отображал сообщения в приложении. Чтобы этого добиться, сначала расширьте разрешения на регистрацию приложений в Azure AD, чтобы обеспечить доступ к данным электронной почты. Затем добавьте в приложение Blazor код для получения и отображения этих данных на одной из страниц.

1. На портале Azure выберите приложение в разделе **Регистрация приложений**.
1. В разделе **Управление** выберите **Разрешения API**.
1. Выберите **Добавить разрешение** > **Microsoft Graph**.
1. Выберите **Делегированные разрешения**, а затем найдите и выберите разрешение **Mail.Read**.
1. Выберите **Добавить разрешения**.

В файле *appsettings.json* обновите код таким образом, чтобы он выбирал соответствующий маркер с нужными разрешениями. В разделе "DownstreamAPI" добавьте "mail.read" после области "user.read". Таким образом вы укажете, к каким областям (или разрешениям) приложение будет запрашивать доступ.

```json
"Scopes": "user.read mail.read"
```

Затем обновите код в файле *FetchData.razor*, чтобы получить данные электронной почты вместо стандартных (случайных) сведений. Замените код в файле следующим:

```csharp
@page "/fetchdata"

@inject IHttpClientFactory HttpClientFactory
@inject Microsoft.Identity.Web.ITokenAcquisition TokenAcquisitionService

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
        _httpClient = HttpClientFactory.CreateClient();


        // get a token
        var token = await TokenAcquisitionService.GetAccessTokenForUserAsync(new string[] { "User.Read", "Mail.Read" });

        // make API call
        _httpClient.DefaultRequestHeaders.Authorization = new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", token);
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

    public class MailMessage
    {
        public string Subject;
        public string Sender;
        public DateTime ReceivedTime;
    }
}

```

запуск приложения; Если при этом будут запрашиваться только что добавленные разрешения, это означает, что все работает правильно. Теперь, помимо запроса основных данных профиля пользователя, приложение запрашивает доступ к данным электронной почты.

Дав свое согласие, перейдите на страницу Fetch data (Получение данных), чтобы прочесть электронную почту.

:::image type="content" source="./media/tutorial-blazor-server/final-app-2.png" alt-text="Снимок экрана конечного приложения. У него есть заголовок с текстом Hello Nicholas (Здравствуйте, Николай), в котором отображается список сообщений электронной почты, отправленных Николаю.":::

## <a name="next-steps"></a>Дальнейшие действия

Подробнее о веб-приложениях, которые вызывают создание, с поддержкой входа пользователей узнайте в нашей серии сценариев.

> [!div class="nextstepaction"]
> [Scenario: Веб-приложение, выполняющее вход пользователей](scenario-web-app-sign-user-overview.md)
