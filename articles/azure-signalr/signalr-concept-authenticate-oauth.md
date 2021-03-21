---
title: Руководство по аутентификации клиентов Службы Azure SignalR
description: Узнайте, как реализовать собственную аутентификацию и интегрировать ее в службу Azure SignalR на примере e2e.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.date: 11/13/2019
ms.author: zhshang
ms.custom: devx-track-js, devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: cd5b709843ba2d98170aaa58eb5c3cc1ec8883af
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98734687"
---
# <a name="azure-signalr-service-authentication"></a>Аутентификация в Службе Azure SignalR

Это руководство основано на приложении комнаты чата, представленном в кратком руководстве. Если вы еще не закончили работу с [Quickstart: Create a chat room with SignalR Service](signalr-quickstart-dotnet-core.md) (Краткое руководство по созданию чата с помощью службы SignalR), сначала выполните эти упражнения.

Из этого руководства вы узнаете, как реализовать собственную аутентификацию и интегрировать ее в службу Microsoft Azure SignalR.

Изначально используемая аутентификация из этого краткого руководства по приложению комнаты чата, является слишком простой для реального мира. Приложение позволяет каждому клиенту утверждать то, кем он является и сервер это просто принимает. Этот подход не очень полезен в реальном мире, где несанкционированный пользователь может олицетворять других, чтобы получить доступ к конфиденциальным данным.

[Портал GitHub](https://github.com/) предоставляет API аутентификацию, основанную на популярном протоколе отраслевого стандарта [OAuth](https://oauth.net/). Эти API позволяют сторонним приложениям проходить аутентификацию учетными записями GitHub. В этом руководстве используются API для реализации аутентификации через учетную запись GitHub, перед тем как разрешить клиентам войти в приложение комнаты чата. После аутентификации учетной записи GitHub, информация о ней будет добавлена в cookie файл, который будет использоваться веб-клиентом для аутентификации.

Дополнительные сведения о API аутентификации OAuth, предоставленной через GitHub, см. в статье [Basics of Authentication](https://developer.github.com/v3/guides/basics-of-authentication/) (Основы аутентификации).

Вы можете использовать любой редактор кода для выполнения шагов в этом кратком руководстве. Однако [Visual Studio Code](https://code.visualstudio.com/) является отличным вариантом, доступным на платформах Windows, MacOS и Linux.

Код для этого руководства доступен для скачивания в [репозитории примеров AzureSignalR на GitHub](https://github.com/aspnet/AzureSignalR-samples/tree/master/samples/GitHubChat).

![Размещение всего OAuth в Azure](media/signalr-concept-authenticate-oauth/signalr-oauth-complete-azure.png)

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Регистрировать новое приложение OAuth в учетной записи GitHub.
> * Добавлять контролер аутентификации к поддерживаемой аутентификации GitHub
> * Развертывать веб приложение ASP.NET Core в Azure

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством необходимо следующее:

- Учетная запись, созданная в [GitHub](https://github.com/)
- [Git](https://git-scm.com/);
- [Базовый пакет SDK для .NET](https://www.microsoft.com/net/download/windows)
- [Azure Cloud Shell](../cloud-shell/quickstart.md) , настроенный для среды bash.
- Скачанный или клонированный репозиторий [примеров AzureSignalR](https://github.com/aspnet/AzureSignalR-samples), доступный в репозитории GitHub.

## <a name="create-an-oauth-app"></a>Создание приложения OAuth

1. Откройте веб-браузер, перейдите к `https://github.com` и войдите в свою учетную запись.

2. В учетной записи перейдите к **Параметрам** > **Параметрам разработчика** и щелкните **Регистрация нового приложения** или **New OAuth App** (Создать приложение OAuth) в разделе *OAuth Apps* (Приложения OAuth).

3. Для нового приложения OAuth используйте следующие параметры, а затем щелкните **Регистрация приложения**.

    | Имя параметра | Рекомендуемое значение | Описание |
    | ------------ | --------------- | ----------- |
    | имя приложения; | *Чат SignalR Azure* | Скорее всего, пользователи GitHub смогут распознать и довериться приложению, с которым они проходят аутентификацию.   |
    | URL-адрес домашней страницы | `http://localhost:5000/home` | |
    | Описание приложения | *Пример использования службы Azure SignalR в комнате чата с помощью аутентификации GitHub* | Полезное описание приложения, которое поможет пользователям приложения распознать содержимое используемой аутентификации. |
    | URL-адрес обратного вызова проверки подлинности | `http://localhost:5000/signin-github` | Этот параметр является самым важным параметром для приложения OAuth. Он является URL-адресом обратного вызова, который GitHub возвращает пользователю, после успешной аутентификации. В этом руководстве по умолчанию необходимо использовать URL-адрес обратного вызова для пакета *AspNet.Security.OAuth.GitHub*, */signin-github*.  |

4. После завершения регистрации нового приложения OAuth добавьте *Идентификатор клиента* и *Секрет клиента* в менеджер секретов с помощью следующих команд. Замените *Your_GitHub_Client_Id* и *Your_GitHub_Client_Secret* на значения из собственного приложения OAuth.

    ```dotnetcli
    dotnet user-secrets set GitHubClientId Your_GitHub_Client_Id
    dotnet user-secrets set GitHubClientSecret Your_GitHub_Client_Secret
    ```

## <a name="implement-the-oauth-flow"></a>Реализация потока OAuth

### <a name="update-the-startup-class-to-support-github-authentication"></a>Чтобы поддержать аутентификацию GitHub, обновите класс Startup

1. Добавьте ссылку на последние пакеты *Microsoft.AspNetCore.Authentication.Cookies* и *AspNet.Security.OAuth.GitHub* восстановите все пакеты.

    ```dotnetcli
    dotnet add package Microsoft.AspNetCore.Authentication.Cookies -v 2.1.0-rc1-30656
    dotnet add package AspNet.Security.OAuth.GitHub -v 2.0.0-rc2-final
    dotnet restore
    ```

1. Откройте файл *Startup.cs* и добавьте инструкцию `using` для следующего пространства имен:

    ```csharp
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Security.Claims;
    using Microsoft.AspNetCore.Authentication.Cookies;
    using Microsoft.AspNetCore.Authentication.OAuth;
    using Newtonsoft.Json.Linq;
    ```

2. В начале класса `Startup` необходимо добавить константы для менеджера секретных ключей, который хранит секреты приложения OAuth GitHub.

    ```csharp
    private const string GitHubClientId = "GitHubClientId";
    private const string GitHubClientSecret = "GitHubClientSecret";
    ```

3. Добавьте следующий код в метод `ConfigureServices`, чтобы поддержать аутентификацию с приложением OAuth GitHub.

    ```csharp
    services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
        .AddCookie()
        .AddGitHub(options =>
        {
            options.ClientId = Configuration[GitHubClientId];
            options.ClientSecret = Configuration[GitHubClientSecret];
            options.Scope.Add("user:email");
            options.Events = new OAuthEvents
            {
                OnCreatingTicket = GetUserCompanyInfoAsync
            };
        });
    ```

4. Добавьте вспомогательный метод `GetUserCompanyInfoAsync` к классу `Startup`.

    ```csharp
    private static async Task GetUserCompanyInfoAsync(OAuthCreatingTicketContext context)
    {
        var request = new HttpRequestMessage(HttpMethod.Get, context.Options.UserInformationEndpoint);
        request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", context.AccessToken);

        var response = await context.Backchannel.SendAsync(request,
            HttpCompletionOption.ResponseHeadersRead, context.HttpContext.RequestAborted);

        var user = JObject.Parse(await response.Content.ReadAsStringAsync());
        if (user.ContainsKey("company"))
        {
            var company = user["company"].ToString();
            var companyIdentity = new ClaimsIdentity(new[]
            {
                new Claim("Company", company)
            });
            context.Principal.AddIdentity(companyIdentity);
        }
    }
    ```

5. Обновите метод `Configure` класса Startup с помощью следующей строчки кода и сохраните файл.

    ```csharp
    app.UseAuthentication();
    ```

### <a name="add-an-authentication-controller"></a>Добавление контролера аутентификации

В этом разделе вы узнаете, как реализовать API `Login`, которые интенсифицируют клиентов с помощью приложения OAuth GitHub. После аутентификации API добавит файл cookie к ответу веб-клиента перед тем, как перенаправить клиента обратно к приложению чата. Затем этот файл cookie будет использоваться для идентификации клиента.

1. Добавьте новый файл контроллера кода в каталог *chattest\Controllers*. Назовите файл *AuthController.cs*.

2. Добавьте следующий код для контроллера аутентификации. Убедитесь, что вы обновили пространство имен (если каталог вашего проекта был не *chattest*).

    ```csharp
    using AspNet.Security.OAuth.GitHub;
    using Microsoft.AspNetCore.Authentication;
    using Microsoft.AspNetCore.Mvc;

    namespace chattest.Controllers
    {
        [Route("/")]
        public class AuthController : Controller
        {
            [HttpGet("login")]
            public IActionResult Login()
            {
                if (!User.Identity.IsAuthenticated)
                {
                    return Challenge(GitHubAuthenticationDefaults.AuthenticationScheme);
                }

                HttpContext.Response.Cookies.Append("githubchat_username", User.Identity.Name);
                HttpContext.SignInAsync(User);
                return Redirect("/");
            }
        }
    }
    ```

3. Сохраните изменения.

### <a name="update-the-hub-class"></a>Обновление класса Hub

По умолчанию, когда веб-клиент пытается соединиться со службой SignalR, получение соединения основывается на маркере доступа, который предоставляется изнутри. Этот маркер доступа не связан с аутентифицированным удостоверением. Этот доступ является анонимным.

Из этого раздела вы узнаете, как включить настоящую аутентификацию, добавив атрибут `Authorize` к классу hub и обновив метод hub, что позволит считывать имена пользователей из аутентифицированных пользовательских утверждений.

1. Откройте *Hub\Chat.cs* и добавьте ссылки на следующие пространства имен:

    ```csharp
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Authorization;
    ```

2. Обновите код концентратора, согласно инструкциям наведенным ниже. Этот код добавляет атрибут `Authorize` в класс `Chat` и использует пользовательское интенсифицированное удостоверение в методах концентратора. Также будет добавлен метод `OnConnectedAsync`, который будет регистрировать системное сообщение в комнате чата каждый раз, когда подсоединяется новый клиент.

    ```csharp
    [Authorize]
    public class Chat : Hub
    {
        public override Task OnConnectedAsync()
        {
            return Clients.All.SendAsync("broadcastMessage", "_SYSTEM_", $"{Context.User.Identity.Name} JOINED");
        }

        // Uncomment this line to only allow user in Microsoft to send message
        //[Authorize(Policy = "Microsoft_Only")]
        public void BroadcastMessage(string message)
        {
            Clients.All.SendAsync("broadcastMessage", Context.User.Identity.Name, message);
        }

        public void Echo(string message)
        {
            var echoMessage = $"{message} (echo from server)";
            Clients.Client(Context.ConnectionId).SendAsync("echo", Context.User.Identity.Name, echoMessage);
        }
    }
    ```

3. Сохраните изменения.

### <a name="update-the-web-client-code"></a>Обновление кода веб-клиента

1. Откройте файл *wwwroot\index.html* и замените код, который предлагает ввести имя пользователя на код, который использует файл cookie, возвращенный контролером аутентификации.

    Из файла *index.html* удалите следующий код:

    ```javascript
    // Get the user name and store it to prepend to messages.
    var username = generateRandomName();
    var promptMessage = 'Enter your name:';
    do {
        username = prompt(promptMessage, username);
        if (!username || username.startsWith('_') || username.indexOf('<') > -1 || username.indexOf('>') > -1) {
            username = '';
            promptMessage = 'Invalid input. Enter your name:';
        }
    } while(!username)
    ```

    Чтобы использовать файл cookie, замените код, который находится выше, следующим кодом:

    ```javascript
    // Get the user name cookie.
    function getCookie(key) {
        var cookies = document.cookie.split(';').map(c => c.trim());
        for (var i = 0; i < cookies.length; i++) {
            if (cookies[i].startsWith(key + '=')) return unescape(cookies[i].slice(key.length + 1));
        }
        return '';
    }
    var username = getCookie('githubchat_username');
    ```

2. Сразу после добавленной строки кода для использования файла cookie для функции `appendMessage` необходимо добавить следующее определение:

    ```javascript
    function appendMessage(encodedName, encodedMsg) {
        var messageEntry = createMessageEntry(encodedName, encodedMsg);
        var messageBox = document.getElementById('messages');
        messageBox.appendChild(messageEntry);
        messageBox.scrollTop = messageBox.scrollHeight;
    }
    ```

3. Обновите функции `bindConnectionMessage` и `onConnected` с помощью следующего кода `appendMessage`.

    ```javascript
    function bindConnectionMessage(connection) {
        var messageCallback = function(name, message) {
            if (!message) return;
            // Html encode display name and message.
            var encodedName = name;
            var encodedMsg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
            appendMessage(encodedName, encodedMsg);
        };
        // Create a function that the hub can call to broadcast messages.
        connection.on('broadcastMessage', messageCallback);
        connection.on('echo', messageCallback);
        connection.onclose(onConnectionError);
    }

    function onConnected(connection) {
        console.log('connection started');
        document.getElementById('sendmessage').addEventListener('click', function (event) {
            // Call the broadcastMessage method on the hub.
            if (messageInput.value) {
                connection
                    .invoke('broadcastMessage', messageInput.value)
                    .catch(e => appendMessage('_BROADCAST_', e.message));
            }

            // Clear text box and reset focus for next comment.
            messageInput.value = '';
            messageInput.focus();
            event.preventDefault();
        });
        document.getElementById('message').addEventListener('keypress', function (event) {
            if (event.keyCode === 13) {
                event.preventDefault();
                document.getElementById('sendmessage').click();
                return false;
            }
        });
        document.getElementById('echo').addEventListener('click', function (event) {
            // Call the echo method on the hub.
            connection.send('echo', messageInput.value);

            // Clear text box and reset focus for next comment.
            messageInput.value = '';
            messageInput.focus();
            event.preventDefault();
        });
    }
    ```

4. Чтобы предложить пользователю войти в систему, обновите обработчик ошибок в конце файла *index.html* для `connection.start()`, как показано ниже.

    ```javascript
    connection.start()
        .then(function () {
            onConnected(connection);
        })
        .catch(function (error) {
            if (error) {
                if (error.message) {
                    console.error(error.message);
                }
                if (error.statusCode && error.statusCode === 401) {
                    appendMessage('_BROADCAST_', 'You\'re not logged in. Click <a href="/login">here</a> to login with GitHub.');
                }
            }
        });
    ```

5. Сохраните изменения.

## <a name="build-and-run-the-app-locally"></a>Локальное создание и запуск приложения

1. Сохраните изменения для всех файлов.

2. Создавайте приложение с помощью .NET Core CLI, выполните следующую команду в оболочке командной строки:

    ```dotnetcli
    dotnet build
    ```

3. После того как создание завершится, чтобы запустить веб-приложение локально, выполните следующую команду:

    ```dotnetcli
    dotnet run
    ```

    По умолчанию приложение будет размещаться локально, используя порт 5000.

    ```output
    E:\Testing\chattest>dotnet run
    Hosting environment: Production
    Content root path: E:\Testing\chattest
    Now listening on: http://localhost:5000
                    Application started. Press Ctrl+C to shut down.
    ```

4. Запустите окно браузера и перейдите к `http://localhost:5000`. Чтобы войти с помощью GitHub, щелкните **эту** ссылку вверху.

    ![Размещение всего OAuth в Azure](media/signalr-concept-authenticate-oauth/signalr-oauth-complete-azure.png)

    Вам будет предложено авторизовать доступ приложения чата к вашей учетной записи GitHub. Нажмите кнопку **Авторизовать**.

    ![Авторизация приложения OAuth](media/signalr-concept-authenticate-oauth/signalr-authorize-oauth-app.png)

    Вы будете перенаправлены обратно к приложению чата, куда необходимо войти со своими учетными данными GitHub. Веб приложение определило имя вашей учетной записи путем использования новой аутентификации, добавленной вами.

    ![Обнаружение учетной записи](media/signalr-concept-authenticate-oauth/signalr-oauth-account-identified.png)

    Теперь, когда приложение чата выполняет аутентификацию с GitHub и сохраняет сведения о аутентификации в качестве файлов cookie, следует развернуть его в Azure, чтобы другие пользователи могли идентифицироваться со своими учетными записями и взаимодействовать с другими рабочими станциями.

## <a name="deploy-the-app-to-azure"></a>Развертывание приложения в Azure

Подготовьте среду к работе с Azure CLI:

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

В этом разделе вы будете использовать интерфейс командной строки Azure (CLI) для создания нового веб-приложения в [службе приложений Azure](../app-service/index.yml) для размещения приложения ASP.NET в Azure. Веб-приложение будет настроено на использование развертывания локальной системы Git. Также веб-приложение будет настроено с помощью строки соединения SignalR, секретов приложения OAuth GitHub и пользователя развертывания.

При создании следующего ресурса, обязательно используйте ту же группу ресурсов, в которой находится ресурс службы SignalR. Позднее, когда возникнет желание удалить все ресурсы, такой подход сделает очистку намного легче. В приведенном примере предполагается использование имени группы *SignalRTestResources*, рекомендованного в предыдущих руководствах.

### <a name="create-the-web-app-and-plan"></a>Создание веб-приложения и плана

Скопируйте текст для команд, приведенных ниже, и обновите параметры. Вставьте обновленный сценарий в Azure Cloud Shell и нажмите клавишу **ВВОД**, чтобы создать новый план службы приложений и веб-приложений.

```azurecli-interactive
#========================================================================
#=== Update these variable for your resource group name.              ===
#========================================================================
ResourceGroupName=SignalRTestResources

#========================================================================
#=== Update these variable for your web app.                          ===
#========================================================================
WebAppName=myWebAppName
WebAppPlan=myAppServicePlanName

# Create an App Service plan.
az appservice plan create --name $WebAppPlan --resource-group $ResourceGroupName \
    --sku FREE

# Create the new Web App
az webapp create --name $WebAppName --resource-group $ResourceGroupName \
    --plan $WebAppPlan
```

| Параметр | Описание |
| -------------------- | --------------- |
| ResourceGroupName | Это имя группы ресурсов было предложено в предыдущих руководствах. Рекомендуется группировать все ресурсы, используемые в этих руководствах, вместе. Используйте ту же группу ресурсов, что и в предыдущих руководствах. |
| WebAppPlan | Введите новое, уникальное имя плана служб приложений. |
| WebAppName | Это станет именем нового веб-приложения и частью URL-адреса. Укажите уникальное имя. Например, signalrtestwebapp22665120.   |

### <a name="add-app-settings-to-the-web-app"></a>Добавление параметров приложения к веб-приложению

Из этого раздела вы узнаете как добавить параметры приложения для следующих компонентов:

* Ресурсная строка подключения службы SignalR
* Идентификатор клиента приложения OAuth GitHub
* Секрет клиента приложения OAuth GitHub

Скопируйте текст для команд, приведенных ниже, и обновите параметры. Вставьте обновленный сценарий в Azure Cloud Shell и нажмите клавишу **ВВОД**, чтобы добавить параметры приложения.

```azurecli-interactive
#========================================================================
#=== Update these variables for your GitHub OAuth App.                ===
#========================================================================
GitHubClientId=1234567890
GitHubClientSecret=1234567890

#========================================================================
#=== Update these variables for your resources.                       ===
#========================================================================
ResourceGroupName=SignalRTestResources
SignalRServiceResource=mySignalRresourcename
WebAppName=myWebAppName

# Get the SignalR primary connection string 
primaryConnectionString=$(az signalr key list --name $SignalRServiceResource \
  --resource-group $ResourceGroupName --query primaryConnectionString -o tsv)

#Add an app setting to the web app for the SignalR connection
az webapp config appsettings set --name $WebAppName \
    --resource-group $ResourceGroupName \
    --settings "Azure__SignalR__ConnectionString=$primaryConnectionString"

#Add the app settings to use with GitHub authentication
az webapp config appsettings set --name $WebAppName \
    --resource-group $ResourceGroupName \
    --settings "GitHubClientId=$GitHubClientId"
az webapp config appsettings set --name $WebAppName \
    --resource-group $ResourceGroupName \
    --settings "GitHubClientSecret=$GitHubClientSecret"
```

| Параметр | Описание |
| -------------------- | --------------- |
| GitHubClientId | Назначьте этой переменной идентификатор секрета клиента для приложения OAuth GitHub. |
| GitHubClientSecret | Назначьте этой переменной секретный пароль для приложения OAuth GitHub. |
| ResourceGroupName | Обновите эту переменную, чтобы у нее было то же имя, что и у группы ресурсов, которую вы использовали в предыдущем разделе. |
| SignalRServiceResource | Обновите эту переменную, чтобы у нее было то же имя, что и у ресурса SignalR, который вы создали в кратком руководстве. Например, signalrtestsvc48778624. |
| WebAppName | Обновите эту переменную, чтобы у нее было то же имя, что и у нового веб-приложения, которое вы создали в предыдущем разделе. |

### <a name="configure-the-web-app-for-local-git-deployment"></a>Настройка веб-приложения для локального развертывания Git

Вставьте следующий сценарий в Azure Cloud Shell. С помощью этого скрипта создаются имя и пароль пользователя развертывания, которые будут использоваться во время развертывания кода с помощью Git в веб-приложении. Сценарий также настраивает веб-приложение для развертывания с помощью локального репозитория Git и возвращает URL-адрес развертывания Git.

```azurecli-interactive
#========================================================================
#=== Update these variables for your resources.                       ===
#========================================================================
ResourceGroupName=SignalRTestResources
WebAppName=myWebAppName

#========================================================================
#=== Update these variables for your deployment user.                 ===
#========================================================================
DeploymentUserName=myUserName
DeploymentUserPassword=myPassword

# Add the desired deployment user name and password
az webapp deployment user set --user-name $DeploymentUserName \
    --password $DeploymentUserPassword

# Configure Git deployment and note the deployment URL in the output
az webapp deployment source config-local-git --name $WebAppName \
    --resource-group $ResourceGroupName \
    --query [url] -o tsv
```

| Параметр | Описание |
| -------------------- | --------------- |
| DeploymentUserName | Выберите новое имя пользователя развертывания. |
| DeploymentUserPassword | Выберите пароль для нового пользователя развертывания. |
| ResourceGroupName | Используйте имя группы ресурсов, которое соответствует имени группы ресурсов, использованному в предыдущем разделе. |
| WebAppName | Это станет именем нового веб-приложения, которое вы создали ранее. |

Запишите URL-адрес развертывания Git, возвращенный из этой команды. Этот URL-адрес будет использоваться позднее.

### <a name="deploy-your-code-to-the-azure-web-app"></a>Разверните код в веб-приложении Azure

Выполните следующую команду в Git Shell, чтобы развернуть код.

1. Перейдите в корневой каталог проекта. Если у вас нет проекта, инициализированного с помощью репозитория Git, выполните следующую команду:

    ```bash
    git init
    ```

2. Добавьте remote для URL-адреса развертывания Git, записанного ранее.

    ```bash
    git remote add Azure <your git deployment url>
    ```

3. Сгруппируйте все файлы в инициализированном репозитории и добавьте фиксацию.

    ```bash
    git add -A
    git commit -m "init commit"
    ```

4. Разверните код в веб-приложении Azure.

    ```bash
    git push Azure main
    ```

    Вам будет предложено пройти аутентификацию, чтобы развернуть код в Azure. Введите имя и пароль пользователя развертывания, созданного выше.

### <a name="update-the-github-oauth-app"></a>Обновление приложения OAuth GitHub

И наконец, для завершения процесса выполните обновление **URL-адреса домашней страницы** и **URL-адреса обратного вызова авторизации** приложения OAuth GitHub, чтобы указать новое размещенное приложение.

1. В браузере откройте [https://github.com](https://github.com) и в учетной записи перейдите к элементам **Параметры** > **Параметры разработчика** > **Oauth Apps** (Приложения OAuth).

2. Щелкните приложение аутентификации и обновите **URL-адрес домашней страницы** и **URL-адрес обратного вызова авторизации**, как показано ниже.

    | Параметр | Пример |
    | ------- | ------- |
    | URL-адрес домашней страницы | `https://signalrtestwebapp22665120.azurewebsites.net/home` |
    | URL-адрес обратного вызова проверки подлинности | `https://signalrtestwebapp22665120.azurewebsites.net/signin-github` |

3. Перейдите к URL-адресу веб-приложения и протестируйте приложение.

    ![Размещение всего OAuth в Azure](media/signalr-concept-authenticate-oauth/signalr-oauth-complete-azure.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

При переходе к следующему руководству в нем можно использовать ресурсы, созданные в этом руководстве.

В противном случае, если вы закончите работу с примером приложения из краткого руководства, вы можете удалить ресурсы Azure, созданные в текущем руководстве, чтобы избежать ненужных расходов.

> [!IMPORTANT]
> Удаление группы ресурсов — необратимая операция, и все соответствующие ресурсы удаляются окончательно. Будьте внимательны, чтобы случайно не удалить не ту группу ресурсов или не те ресурсы. Если ресурсы для размещения этого примера созданы в имеющейся группе ресурсов, содержащей ресурсы, которые следует сохранить, можно удалить каждый ресурс отдельно в соответствующих колонках вместо удаления группы ресурсов.

Войдите на [портал Azure](https://portal.azure.com) и щелкните **Группы ресурсов**.

Введите имя группы ресурсов в текстовое поле **Фильтровать по имени...** . В инструкциях в этой статье использовалась группа ресурсов с именем *SignalRTestResources*. В своей группе ресурсов в списке результатов щелкните **...** , а затем **Удалить группу ресурсов**.

![DELETE](./media/signalr-concept-authenticate-oauth/signalr-delete-resource-group.png)

Подтвердите операцию удаления группы ресурсов. Введите имя группы ресурсов и нажмите кнопку **Удалить**.

Через некоторое время группа ресурсов и все ее ресурсы будут удалены.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы научились добавлять аутентификацию с помощью OAuth, чтобы улучшить подход к аутентификации службы Azure SignalR. Чтобы узнать больше об использовании сервера Azure SignalR, перейдите к примерам Azure CLI для службы SignalR.

> [!div class="nextstepaction"]
> [Примеры CLI для Azure SignalR](./signalr-reference-cli.md)