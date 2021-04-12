---
title: Краткое руководство по использованию Службы Azure SignalR
description: Краткое руководство по использованию службы Azure SignalR для создания комнаты чата с помощью приложений MVC для ASP.NET Core.
author: sffamily
ms.service: signalr
ms.devlang: dotnet
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.date: 09/28/2020
ms.author: zhshang
ms.openlocfilehash: 3cfcf57f455a5c3b17b794acf2ded66ed2285eff
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94873511"
---
# <a name="quickstart-create-a-chat-room-by-using-signalr-service"></a>Краткое руководство. Создание чата с помощью Службы SignalR

Azure SignalR — это служба Azure, которая помогает разработчикам без труда создавать веб-приложения с функциями в режиме реального времени. Эта служба была изначально основана на [SignalR для ASP.NET Core 2.1](/aspnet/core/signalr/introduction?preserve-view=true&view=aspnetcore-2.1), но теперь поддерживает более поздние версии.

В этой статье показано, как приступить к работе со службой Azure SignalR. В этом кратком руководстве вы создадите приложение чата с помощью веб-приложения MVC для ASP.NET Core. Это приложение установит соединение с ресурсом службы Azure SignalR для включения обновлений содержимого в режиме реального времени. Вы разместите веб-приложение локально и подключите к нему несколько браузерных клиентов. Каждый клиент сможет принудительно отправлять обновления содержимого остальным клиентам. 

Вы можете использовать любой редактор кода для выполнения шагов в этом кратком руководстве. Одним из вариантов является [Visual Studio Code](https://code.visualstudio.com/), который доступен на платформах Windows, macOS и Linux.

Код для этого руководства доступен для скачивания в [репозитории примеров AzureSignalR на GitHub](https://github.com/aspnet/AzureSignalR-samples/tree/master/samples/ChatRoom). Вы также можете создать ресурсы Azure, используемые в этом кратком руководстве, с помощью [скрипта создания Службы SignalR](scripts/signalr-cli-create-service.md).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note-dotnet.md)]

## <a name="prerequisites"></a>Предварительные требования

* Установите [пакет SDK для .NET Core](https://www.microsoft.com/net/download/windows).
* Скачанный или клонированный репозиторий [примеров AzureSignalR](https://github.com/aspnet/AzureSignalR-samples), доступный в репозитории GitHub. 

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsnetcore).

## <a name="create-an-azure-signalr-resource"></a>Создание ресурса Azure SignalR

[!INCLUDE [azure-signalr-create](../../includes/signalr-create.md)]

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsnetcore).

## <a name="create-an-aspnet-core-web-app"></a>Создание веб-приложения ASP.NET Core

В этом разделе вы примените [.NET Core CLI](/dotnet/core/tools/) для создания проекта веб-приложения MVC для ASP.NET Core. Преимущество использования .NET Core CLI по сравнению с Visual Studio заключается в том, что он доступен на платформах Windows, macOS и Linux. 

1. Создайте каталог для своего проекта. В этом кратком руководстве используется папка *E:\Testing\chattest*.

2. В новой папке выполните следующую команду, чтобы создать проект:

    ```dotnetcli
    dotnet new mvc
    ```

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsnetcore).

## <a name="add-secret-manager-to-the-project"></a>Добавление менеджера секретных ключей в проект

В этом разделе вы добавите в проект инструмент [Диспетчер секретов](/aspnet/core/security/app-secrets). Инструмент "Диспетчер секретов" хранит конфиденциальные данные для разработки вне вашего дерева проектов. Такой подход помогает предотвратить случайную компрометацию секретов приложений в исходном коде.

1. Откройте файл с расширением *CSPROJ*. Добавьте элемент `DotNetCliToolReference`, чтобы включить *Microsoft.Extensions.SecretManager.Tools*. Кроме того, добавьте элемент `UserSecretsId`, как показано в следующем коде *chattest.csproj*, и сохраните файл.

    ```xml
    <Project Sdk="Microsoft.NET.Sdk.Web">

    <PropertyGroup>
        <TargetFramework>netcoreapp3.1</TargetFramework>
        <UserSecretsId>SignalRChatRoomEx</UserSecretsId>
    </PropertyGroup>

    <ItemGroup>
        <DotNetCliToolReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Tools" Version="2.0.4" />
        <DotNetCliToolReference Include="Microsoft.Extensions.SecretManager.Tools" Version="2.0.2" />
    </ItemGroup>

    </Project>
    ```

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsnetcore).

## <a name="add-azure-signalr-to-the-web-app"></a>Добавление веб-приложения Azure SignalR

1. Добавьте ссылку на пакет NuGet `Microsoft.Azure.SignalR`, выполнив следующую команду:

    ```dotnetcli
    dotnet add package Microsoft.Azure.SignalR
    ```

2. Выполните следующую команду, чтобы восстановить пакеты проекта:

    ```dotnetcli
    dotnet restore
    ```

3. Добавьте секрет с именем *Azure: SignalR: ConnectionString* в диспетчер секретов. 

    Этот секрет будет содержать строку подключения для получения доступа к ресурсам службы SignalR. *Azure:SignalR:ConnectionString* — ключ конфигурации по умолчанию, который SignalR ищет для установки подключения. Замените значение, включенное в приведенную ниже команду, строкой подключения к реальному ресурсу службы SignalR.

    Эту команду необходимо выполнять в том же каталоге, где расположен *CSPROJ*-файл.

    ```dotnetcli
    dotnet user-secrets set Azure:SignalR:ConnectionString "<Your connection string>"
    ```

    Диспетчер секретов будет использоваться только для тестирования веб-приложения при локальном размещении. В следующем руководстве вы развернете веб-приложение чата в Azure. После развертывания веб-приложения в Azure вы будете использовать параметр приложения, а не строку подключения из диспетчера секретов.

    Доступ к этому секрету осуществляется через API конфигурации. Двоеточие (:) используется в имени конфигурации при работе с API конфигурации на всех поддерживаемых платформах. Дополнительные сведения см. в статье [Конфигурация в .NET Core](/dotnet/core/extensions/configuration-providers#environment-variable-configuration-provider).


4. Откройте файл *Startup.cs* и обновите метод `ConfigureServices` для использования службы Azure SignalR, вызвав методы `AddSignalR()` и `AddAzureSignalR()`:

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSignalR()
                .AddAzureSignalR();
    }
    ```

    В этом коде не указан параметр `AddAzureSignalR()`, а значит в нем для строки подключения к ресурсу SignalR используется ключ конфигурации по умолчанию. По умолчанию используется ключ конфигурации *Azure:SignalR:ConnectionString*.

5. В файле *Startup.cs* обновите метод `Configure`, заменив его следующим кодом.

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        app.UseRouting();
        app.UseFileServer();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapHub<ChatHub>("/chat");
        });
    }
    ```

### <a name="add-a-hub-class"></a>Добавление класса концентратора

В SignalR основным компонентом является концентратор, который предоставляет набор методов для вызова из клиента. В этом разделе необходимо определить класс концентратора с двумя методами:

* `Broadcast`: этот метод осуществляет широковещательную рассылку сообщений всем клиентам.
* `Echo`: этот метод отправляет сообщение обратно в вызывающий объект.

Оба метода используют интерфейс `Clients`, предоставляемый пакетом SDK для SignalR и ASP.NET Core. Этот интерфейс предоставляет доступ ко всем подключенным клиентам, позволяя отправлять в них содержимое.

1. В каталоге проекта добавьте новую папку с именем *Hub*. Добавьте новый файл с кодом концентратора с именем *ChatHub.cs* в новую папку.

2. Добавьте в файл *ChatHub.cs* следующий код, который определяет класс концентратора, и сохраните файл.

    Обновите пространство имен для этого класса, если используется имя проекта, отличное от *SignalR.Mvc*.

    ```csharp
    using Microsoft.AspNetCore.SignalR;
    using System.Threading.Tasks;
    
    namespace SignalR.Mvc
    {
        public class ChatHub : Hub
        {
            public Task BroadcastMessage(string name, string message) =>
                Clients.All.SendAsync("broadcastMessage", name, message);
    
            public Task Echo(string name, string message) =>
                Clients.Client(Context.ConnectionId)
                       .SendAsync("echo", name, $"{message} (echo from server)");
        }
    }
    ```

### <a name="add-the-client-interface-for-the-web-app"></a>Добавление клиентского интерфейса для веб-приложения

Пользовательский интерфейс клиента для этого приложения чата представлен в коде HTML и JavaScript в файле с именем *index.html* в каталоге *wwwroot*.

Скопируйте файл *css/site.css* из папки *wwwroot* в [репозитории примеров](https://github.com/aspnet/AzureSignalR-samples/tree/master/samples/ChatRoom/wwwroot). Замените файл *css/site.css* проекта скопированным файлом.

Так выглядит основной код файла *index.html*:

Создайте новый файл в каталоге *wwwroot* с именем *index.html*, а затем скопируйте и вставьте в него следующий код HTML:

```html
<!DOCTYPE html>
<html>
<head>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" rel="stylesheet" />
    <link href="css/site.css" rel="stylesheet" />
    <title>Azure SignalR Group Chat</title>
</head>
<body>
    <h2 class="text-center" style="margin-top: 0; padding-top: 30px; padding-bottom: 30px;">Azure SignalR Group Chat</h2>
    <div class="container" style="height: calc(100% - 110px);">
        <div id="messages" style="background-color: whitesmoke; "></div>
        <div style="width: 100%; border-left-style: ridge; border-right-style: ridge;">
            <textarea id="message"
                      style="width: 100%; padding: 5px 10px; border-style: hidden;"
                      placeholder="Type message and press Enter to send..."></textarea>
        </div>
        <div style="overflow: auto; border-style: ridge; border-top-style: hidden;">
            <button class="btn-warning pull-right" id="echo">Echo</button>
            <button class="btn-success pull-right" id="sendmessage">Send</button>
        </div>
    </div>
    <div class="modal alert alert-danger fade" id="myModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <div>Connection Error...</div>
                    <div><strong style="font-size: 1.5em;">Hit Refresh/F5</strong> to rejoin. ;)</div>
                </div>
            </div>
        </div>
    </div>

    <!--Reference the SignalR library. -->
    <script src="https://cdn.jsdelivr.net/npm/@microsoft/signalr@3.1.8/dist/browser/signalr.min.js"></script>

    <!--Add script to update the page and send messages.-->
    <script type="text/javascript">
        document.addEventListener('DOMContentLoaded', function () {

            const generateRandomName = () =>
                Math.random().toString(36).substring(2, 10);

            let username = generateRandomName();
            const promptMessage = 'Enter your name:';
            do {
                username = prompt(promptMessage, username);
                if (!username || username.startsWith('_') || username.indexOf('<') > -1 || username.indexOf('>') > -1) {
                    username = '';
                    promptMessage = 'Invalid input. Enter your name:';
                }
            } while (!username)

            const messageInput = document.getElementById('message');
            messageInput.focus();

            function createMessageEntry(encodedName, encodedMsg) {
                var entry = document.createElement('div');
                entry.classList.add("message-entry");
                if (encodedName === "_SYSTEM_") {
                    entry.innerHTML = encodedMsg;
                    entry.classList.add("text-center");
                    entry.classList.add("system-message");
                } else if (encodedName === "_BROADCAST_") {
                    entry.classList.add("text-center");
                    entry.innerHTML = `<div class="text-center broadcast-message">${encodedMsg}</div>`;
                } else if (encodedName === username) {
                    entry.innerHTML = `<div class="message-avatar pull-right">${encodedName}</div>` +
                        `<div class="message-content pull-right">${encodedMsg}<div>`;
                } else {
                    entry.innerHTML = `<div class="message-avatar pull-left">${encodedName}</div>` +
                        `<div class="message-content pull-left">${encodedMsg}<div>`;
                }
                return entry;
            }

            function bindConnectionMessage(connection) {
                var messageCallback = function (name, message) {
                    if (!message) return;
                    var encodedName = name;
                    var encodedMsg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
                    var messageEntry = createMessageEntry(encodedName, encodedMsg);

                    var messageBox = document.getElementById('messages');
                    messageBox.appendChild(messageEntry);
                    messageBox.scrollTop = messageBox.scrollHeight;
                };
                connection.on('broadcastMessage', messageCallback);
                connection.on('echo', messageCallback);
                connection.onclose(onConnectionError);
            }

            function onConnected(connection) {
                console.log('connection started');
                connection.send('broadcastMessage', '_SYSTEM_', username + ' JOINED');
                document.getElementById('sendmessage').addEventListener('click', function (event) {
                    if (messageInput.value) {
                        connection.send('broadcastMessage', username, messageInput.value);
                    }

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
                    connection.send('echo', username, messageInput.value);

                    messageInput.value = '';
                    messageInput.focus();
                    event.preventDefault();
                });
            }

            function onConnectionError(error) {
                if (error && error.message) {
                    console.error(error.message);
                }
                var modal = document.getElementById('myModal');
                modal.classList.add('in');
                modal.style = 'display: block;';
            }

            const connection = new signalR.HubConnectionBuilder()
                .withUrl('/chat')
                .build();
            bindConnectionMessage(connection);
            connection.start()
                .then(() => onConnected(connection))
                .catch(error => console.error(error.message));
        });
    </script>
</body>
</html>
```

Код в файле *index.html* вызывает `HubConnectionBuilder.build()` для установки HTTP-подключения к ресурсу Azure SignalR.

Если подключение выполнено успешно, это соединение передается в `bindConnectionMessage`, при этом добавляются обработчики событий для входящей передачи содержимого в клиент. 

`HubConnection.start()` запускает обмен данными с концентратором. Затем `onConnected()` добавляет обработчики событий кнопки. Эти обработчики используют это соединение, чтобы позволить клиенту передавать обновления содержимого для всех подключенных клиентов.

## <a name="add-a-development-runtime-profile"></a>Добавление профиля среды выполнения разработки

В этом разделе вы добавите среду выполнения для разработки на ASP.NET Core. Дополнительные сведения см. в статье [Использование нескольких сред в ASP.NET Core](/aspnet/core/fundamentals/environments).

1. Создайте в проекте папку с именем *Properties*.

2. Добавьте в эту папку новый файл с именем *launchSettings.json*, поместите в него следующее содержимое и сохраните файл.

    ```json
    {
        "profiles" : {
            "ChatRoom": {
                "commandName": "Project",
                "launchBrowser": true,
                "environmentVariables": {
                    "ASPNETCORE_ENVIRONMENT": "Development"
                },
                "applicationUrl": "http://localhost:5000/"
            }
        }
    }
    ```

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsnetcore).

## <a name="build-and-run-the-app-locally"></a>Создание и запуск приложения локально

1. Чтобы создать приложение с помощью .NET Core CLI, выполните следующую команду в оболочке командной строки:

    ```dotnetcli
    dotnet build
    ```

1. Когда сборка завершится, запустите веб-приложение локально с помощью следующей команды:

    ```dotnetcli
    dotnet run
    ```

    Приложение будет размещено на порту 5000 локального компьютера, как указано в профиле среды выполнения разработки:

    ```output
    info: Microsoft.Hosting.Lifetime[0]
          Now listening on: https://localhost:5001
    info: Microsoft.Hosting.Lifetime[0]
          Now listening on: http://localhost:5000
    info: Microsoft.Hosting.Lifetime[0]
          Application started. Press Ctrl+C to shut down.
    info: Microsoft.Hosting.Lifetime[0]
          Hosting environment: Development
    info: Microsoft.Hosting.Lifetime[0]
          Content root path: E:\Testing\chattest
    ```

1. Откройте два окна браузера. В каждом из них откройте адрес `http://localhost:5000`. Вам будет предложено ввести имя. Введите имя клиента для обоих клиентов и протестируйте передачу сообщений между клиентами с помощью кнопки **Отправить**.

    ![Пример групповой беседы Azure SignalR](media/signalr-quickstart-dotnet-core/signalr-quickstart-complete-local.png)

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsnetcore).

## <a name="clean-up-resources"></a>Очистка ресурсов

Для работы со следующим руководством можно сохранить и использовать ресурсы, созданные в этом кратком руководстве.

Если вы не намерены далее работать с примером приложения из этого краткого руководства, удалите созданные для него ресурсы Azure, чтобы избежать ненужных расходов. 

> [!IMPORTANT]
> Удаление группы ресурсов необратимо и распространяется на все ресурсы в группе. Будьте внимательны, чтобы случайно не удалить не ту группу ресурсов или не те ресурсы. Если ресурсы для размещения этого примера созданы в имеющейся группе ресурсов, которая содержит другие нужные ресурсы, каждый ресурс можно удалить отдельно через колонку приложения, чтобы не удалять группу ресурсов.

Войдите в портал [Azure](https://portal.azure.com) и выберите **Группы ресурсов**.

Введите имя группы ресурсов в текстовое поле **Фильтровать по имени**. В инструкциях к этому краткому руководству была использована группа ресурсов с именем *SignalRTestResources*. В списке результатов щелкните значок многоточия ( **...** ) рядом с этой группой ресурсов и выберите действие **Удалить группу ресурсов**.

![Последовательность действий для удаления группы ресурсов](./media/signalr-quickstart-dotnet-core/signalr-delete-resource-group.png)

Подтвердите операцию удаления группы ресурсов. Введите имя группы ресурсов, которую необходимо удалить, и нажмите **Удалить**.

Через некоторое время группа ресурсов и все ее ресурсы будут удалены.

Возникли проблемы? См. [руководство по устранению неполадок](signalr-howto-troubleshoot-guide.md) или [сообщите о проблеме нам](https://aka.ms/asrs/qsnetcore).

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали ресурс Службы Azure SignalR. Затем вы использовали его в веб-приложении ASP.NET Core, чтобы в реальном времени передавать обновленное содержимое в несколько подключенных клиентов. Чтобы получить дополнительные сведения об использовании Службы Azure SignalR, перейдите к руководству по проверке подлинности.

> [!div class="nextstepaction"]
> [Tutorial: Azure SignalR Service authentication](./signalr-concept-authenticate-oauth.md) (Руководство по аутентификации службы Azure SignalR)
