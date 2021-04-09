---
title: Руководство по созданию приложения чата Blazor Server с помощью Azure SignalR
description: В этом учебнике описано, как создать и изменить приложение Blazor Server с помощью Службы Azure SignalR.
author: JialinXin
ms.service: signalr
ms.topic: tutorial
ms.date: 09/09/2020
ms.author: jixin
ms.openlocfilehash: a1423e8aaf4b50db94cda0b935a7b1658249893e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105726351"
---
# <a name="tutorial-build-a-blazor-server-chat-app"></a>Руководство по Создание приложения чата Blazor Server

В этом учебнике показано, как создать и изменить приложение Blazor Server. Вы узнаете, как:

> [!div class="checklist"]
> * создать простую комнату чата с помощью приложение Blazor Server;
> * изменить компоненты Razor;
> * использовать обработку событий и привязку данных в компонентах;
> * выполнить быстрое развертывание в Службе приложений Azure с помощью Visual Studio;
> * перенести локальную службу SignalR в Службу Azure SignalR.

## <a name="prerequisites"></a>Предварительные требования
* Установите [пакет SDK для .NET Core 3.0](https://dotnet.microsoft.com/download/dotnet-core/3.0) (версии не ниже 3.0.100).
* Установите [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) (версии не ниже 16.3).
> Также подойдет версия Visual Studio 2019 Preview, которая выпускается с последним шаблоном приложения Blazor Server, предназначенным для более новой версии .Net Core.

[Возникли проблемы? Сообщите нам!](https://aka.ms/asrs/qsblazor)

## <a name="build-a-local-chat-room-in-blazor-server-app"></a>Создание локальной комнаты чата в приложении Blazor Server

В Visual Studio 2019 версии 16.2.0 Служба Azure SignalR является встроенным процессом публикации веб-приложений, а управление зависимостями между веб-приложением и службой SignalR реализовано гораздо удобнее. Вы можете одновременно работать с локальной службой SignalR в локальной среде разработки и со Службой Azure SignalR для Службы приложений Azure, не внося в код каких бы то ни было изменений.

1. Создание приложения Blazor
   
   В Visual Studio выберите "Создать проект" > "Приложение Blazor Server" > (введите имя приложения и выберите папку) > "Приложение Blazor Server". Убедитесь, что вы уже установили пакет SDK для .NET Core 3.0 (или более новой версии), чтобы позволить Visual Studio правильно распознавать требуемую версию .NET Framework.

   [ ![Выбранное окно шаблонов приложений Blazor в разделе "Создать проект".](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-create.png) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-create.png#lightbox)
   
   Или выполните команду cmd.
   ```dotnetcli
   dotnet new blazorserver -o BlazorChat
   ```
   
1. Добавьте файл `BlazorChatSampleHub.cs`, чтобы реализовать `Hub` для чата.
   
   ```cs
   using System;
   using System.Threading.Tasks;
   using Microsoft.AspNetCore.SignalR;
   
   namespace BlazorChat
   {
       public class BlazorChatSampleHub : Hub
       {
           public const string HubUrl = "/chat";
   
           public async Task Broadcast(string username, string message)
           {
               await Clients.All.SendAsync("Broadcast", username, message);
           }
   
           public override Task OnConnectedAsync()
           {
               Console.WriteLine($"{Context.ConnectionId} connected");
               return base.OnConnectedAsync();
           }
   
           public override async Task OnDisconnectedAsync(Exception e)
           {
               Console.WriteLine($"Disconnected {e?.Message} {Context.ConnectionId}");
               await base.OnDisconnectedAsync(e);
           }
       }
   }
   ```
   
1. Добавьте конечную точку для концентратора в `Startup.Configure()`.
   
   ```cs
   app.UseEndpoints(endpoints =>
   {
       endpoints.MapBlazorHub();
       endpoints.MapFallbackToPage("/_Host");
       endpoints.MapHub<BlazorChatSampleHub>(BlazorChatSampleHub.HubUrl);
   });
   ```
   
1. Установите пакет `Microsoft.AspNetCore.SignalR.Client` для использования клиента SignalR.

   ```dotnetcli
   dotnet add package Microsoft.AspNetCore.SignalR.Client --version 3.1.7
   ```

1. Создайте `ChatRoom.razor` в папке `Pages`, чтобы реализовать клиент SignalR. Выполните приведенные ниже действия или просто скопируйте [ChatRoom.razor](https://github.com/aspnet/AzureSignalR-samples/tree/master/samples/BlazorChat/Pages/ChatRoom.razor).

   1. Добавьте ссылку на страницу и ссылку.
      
      ```razor
      @page "/chatroom"
      @inject NavigationManager navigationManager
      @using Microsoft.AspNetCore.SignalR.Client;
      ```

   1. Добавьте в новый клиент SignalR код для отправки и получения сообщений.
      
      ```razor
      @code {
          // flag to indicate chat status
          private bool _isChatting = false;
          
          // name of the user who will be chatting
          private string _username;
      
          // on-screen message
          private string _message;
          
          // new message input
          private string _newMessage;
          
          // list of messages in chat
          private List<Message> _messages = new List<Message>();
          
          private string _hubUrl;
          private HubConnection _hubConnection;
      
          public async Task Chat()
          {
              // check username is valid
              if (string.IsNullOrWhiteSpace(_username))
              {
                  _message = "Please enter a name";
                  return;
              };
      
              try
              {
                  // Start chatting and force refresh UI.
                  _isChatting = true;
                  await Task.Delay(1);

                  // remove old messages if any
                  _messages.Clear();
         
                  // Create the chat client
                  string baseUrl = navigationManager.BaseUri;
      
                  _hubUrl = baseUrl.TrimEnd('/') + BlazorChatSampleHub.HubUrl;
      
                  _hubConnection = new HubConnectionBuilder()
                      .WithUrl(_hubUrl)
                      .Build();
      
                  _hubConnection.On<string, string>("Broadcast", BroadcastMessage);
      
                  await _hubConnection.StartAsync();
      
                  await SendAsync($"[Notice] {_username} joined chat room.");
              }
              catch (Exception e)
              {
                  _message = $"ERROR: Failed to start chat client: {e.Message}";
                  _isChatting = false;
              }
          }
      
          private void BroadcastMessage(string name, string message)
          {
              bool isMine = name.Equals(_username, StringComparison.OrdinalIgnoreCase);
      
              _messages.Add(new Message(name, message, isMine));
      
              // Inform blazor the UI needs updating
              StateHasChanged();
          }
      
          private async Task DisconnectAsync()
          {
              if (_isChatting)
              {
                  await SendAsync($"[Notice] {_username} left chat room.");
      
                  await _hubConnection.StopAsync();
                  await _hubConnection.DisposeAsync();
      
                  _hubConnection = null;
                  _isChatting = false;
              }
          }
      
          private async Task SendAsync(string message)
          {
              if (_isChatting && !string.IsNullOrWhiteSpace(message))
              {
                  await _hubConnection.SendAsync("Broadcast", _username, message);
      
                  _newMessage = string.Empty;
              }
          }
      
          private class Message
          {
              public Message(string username, string body, bool mine)
              {
                  Username = username;
                  Body = body;
                  Mine = mine;
              }
      
              public string Username { get; set; }
              public string Body { get; set; }
              public bool Mine { get; set; }
      
              public bool IsNotice => Body.StartsWith("[Notice]");
      
              public string CSS => Mine ? "sent" : "received";
          }
      }
      ```

   1. Добавьте часть отрисовки перед `@code` для пользовательского интерфейса, чтобы взаимодействовать с клиентом SignalR.
      
      ```razor
      <h1>Blazor SignalR Chat Sample</h1>
      <hr />
      
      @if (!_isChatting)
      {
          <p>
              Enter your name to start chatting:
          </p>
      
          <input type="text" maxlength="32" @bind="@_username" />
          <button type="button" @onclick="@Chat"><span class="oi oi-chat" aria-hidden="true"></span> Chat!</button>
      
          // Error messages
          @if (_message != null)
          {
              <div class="invalid-feedback">@_message</div>
              <small id="emailHelp" class="form-text text-muted">@_message</small>
          }
      }
      else
      {
          // banner to show current user
          <div class="alert alert-secondary mt-4" role="alert">
              <span class="oi oi-person mr-2" aria-hidden="true"></span>
              <span>You are connected as <b>@_username</b></span>
              <button class="btn btn-sm btn-warning ml-md-auto" @onclick="@DisconnectAsync">Disconnect</button>
          </div>
          // display messages
          <div id="scrollbox">
              @foreach (var item in _messages)
              {
                  @if (item.IsNotice)
                  {
                      <div class="alert alert-info">@item.Body</div>
                  }
                  else
                  {
                      <div class="@item.CSS">
                          <div class="user">@item.Username</div>
                          <div class="msg">@item.Body</div>
                      </div>
                  }
              }
              <hr />
              <textarea class="input-lg" placeholder="enter your comment" @bind="@_newMessage"></textarea>
              <button class="btn btn-default" @onclick="@(() => SendAsync(_newMessage))">Send</button>
          </div>
      }
      ```

1. Обновите `NavMenu.razor`, чтобы вставить меню ввода для комнаты чата в `NavMenuCssClass`, как и прочее.

   ```razor
   <li class="nav-item px-3">
       <NavLink class="nav-link" href="chatroom">
           <span class="oi oi-chat" aria-hidden="true"></span> Chat room
       </NavLink>
   </li>
   ```
   
1. Обновите `site.css`, чтобы оптимизировать всплывающие представления области диаграммы. Добавьте в конец приведенный ниже код.

   ```css
   /* improved for chat text box */
   textarea {
       border: 1px dashed #888;
       border-radius: 5px;
       width: 80%;
       overflow: auto;
       background: #f7f7f7
   }
   
   /* improved for speech bubbles */
   .received, .sent {
       position: relative;
       font-family: arial;
       font-size: 1.1em;
       border-radius: 10px;
       padding: 20px;
       margin-bottom: 20px;
   }
   
   .received:after, .sent:after {
       content: '';
       border: 20px solid transparent;
       position: absolute;
       margin-top: -30px;
   }
   
   .sent {
       background: #03a9f4;
       color: #fff;
       margin-left: 10%;
       top: 50%;
       text-align: right;
   }
   
   .received {
       background: #4CAF50;
       color: #fff;
       margin-left: 10px;
       margin-right: 10%;
   }
   
   .sent:after {
       border-left-color: #03a9f4;
       border-right: 0;
       right: -20px;
   }
   
   .received:after {
       border-right-color: #4CAF50;
       border-left: 0;
       left: -20px;
   }
   
   /* div within bubble for name */
   .user {
       font-size: 0.8em;
       font-weight: bold;
       color: #000;
   }
   
   .msg {
       /*display: inline;*/
   }
   ```

1. Нажмите клавишу <kbd>F5</kbd>, чтобы запустить приложение. Вы сможете общаться, как показано ниже.

   [ ![Анимированный чат между Бобом и Элис. Элис говорит "Здравствуй", Боб отвечает "Привет".](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat.gif) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat.gif#lightbox)
   
[Возникли проблемы? Сообщите нам!](https://aka.ms/asrs/qsblazor)

## <a name="publish-to-azure"></a>Публикация в Azure

   Пока что приложение Blazor работает в локальном экземпляре SignalR, и при развертывании в Службе приложений Azure рекомендуется использовать [Службу Azure SignalR](/aspnet/core/signalr/scale#azure-signalr-service), что позволит масштабировать приложение Blazor Server для поддержки большого количества одновременных подключений SignalR. Кроме того, глобальный охват службы SignalR и ее высокопроизводительные центры обработки данных обеспечивают низкую задержку благодаря доступности в разных регионах.

> [!IMPORTANT]
> В приложении Blazor Server состояния пользовательского интерфейса поддерживаются на стороне сервера. Это означает, что в данном случае необходима привязка сервера. Если используется один сервер приложений, то привязка сервера реализована при разработке. Но если существует несколько серверов приложений, существует вероятность, что при согласовании и подключении клиента могут быть выбраны разные серверы, что приведет к ошибкам пользовательского интерфейса в приложении Blazor. Поэтому необходимо включить привязку сервера, как показано ниже в `appsettings.json`.
> ```json
> "Azure:SignalR:ServerStickyMode": "Required"
> ```

1. Щелкните проект правой кнопкой мыши и выберите `Publish`.

   * Цель: Azure
   * Указанный целевой объект: поддерживаются все типы **Службы приложений Azure**.
   * Служба приложений: выберите существующий план службы приложений или создайте новый.

   [ ![Анимация: выбор Azure в качестве целевого объекта и Службы приложений Azure в качестве конкретизированного целевого объекта.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-profile.gif) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-profile.gif#lightbox)

1. Добавление зависимости Службы Azure SignalR

   После создания профиля публикации можно просмотреть рекомендуемое сообщение в разделе **Зависимости служб**. Щелкните **Настроить**, чтобы создать новую или выбрать существующую Службу Azure SignalR на панели.

   [ ![Ссылка для настройки, выделенная в окне публикации.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-dependency.png) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-dependency.png#lightbox)

   Зависимость службы выполнит приведенные ниже действия, чтобы ваше приложение автоматически переключалось на Службу Azure SignalR при запуске в Azure.

   * Обновите [`HostingStartupAssembly`](/aspnet/core/fundamentals/host/platform-specific-configuration), чтобы использовать Службу Azure SignalR.
   * Добавьте ссылку на пакет NuGet для Службы Azure SignalR.
   * Обновите свойства профиля, чтобы сохранить параметры зависимости.
   * Настройте хранилище секретов на свой выбор.
   * Добавьте конфигурацию `appsettings`, чтобы ваше приложение было нацелено на Службу Azure SignalR.

   [ ![Установленные флажки для выбора всех зависимостей в окне сводки изменений.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-dependency-summary.png) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-dependency-summary.png#lightbox)

1. Публикация приложения

   Теперь все готово к публикации. И после завершения публикации вы автоматически перейдете на соответствующую страницу. 
   > [!NOTE]
   > Возможно, в первый раз вы не сможете тут же перейти на эту страницу из-за задержки при развертывании Службы приложений Azure. Попробуйте обновить страницу, немного подождав.
   > Кроме того, можно использовать режим отладчика браузера, нажав клавишу <kbd>F12</kbd>, чтобы убедиться, что трафик уже перенаправляется в Службу Azure SignalR.

   [ ![Пример чата SignalR на платформе Blazor содержит текстовое поле для ввода имени и кнопку для начала чата.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-azure.png) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-azure.png#lightbox)
   
[Возникли проблемы? Сообщите нам!](https://aka.ms/asrs/qsblazor)

## <a name="further-topic-enable-azure-signalr-service-in-local-development"></a>Дополнительная тема: включение Службы Azure SignalR в локальной среде разработки

1. Добавьте ссылку на пакет SDK для Azure SignalR.

   ```dotnetcli
   dotnet add package Microsoft.Azure.SignalR --version 1.5.1
   ```

1. Добавьте вызов Службы Azure SignalR в `Startup.ConfigureServices()`.

   ```cs
   public void ConfigureServices(IServiceCollection services)
   {
       ...
       services.AddSignalR().AddAzureSignalR();
       ...
   }
   ```

1. Настройте `ConnectionString` Службы Azure SignalR в `appsetting.json` или с помощью инструмента [Диспетчер секретов](/aspnet/core/security/app-secrets?tabs=visual-studio#secret-manager).

> [!NOTE]
> Шаг 2 можно заменить с помощью [`HostingStartupAssembly`](/aspnet/core/fundamentals/host/platform-specific-configuration) в пакете SDK для SignalR.
> 
> 1. Добавьте конфигурацию для включения Службы Azure SignalR в `appsetting.json`.
>    ```js
>    "Azure": {
>      "SignalR": {
>        "Enabled": true,
>        "ConnectionString": <your-connection-string>
>      }
>    }
>    ```
> 
> 1. Назначьте стартовую сборку размещения для использования пакета SDK для Azure SignalR. Измените `launchSettings.json` и добавьте конфигурацию в `environmentVariables`, как показано ниже.
>    ```js
>    "environmentVariables": {
>        ...,
>        "ASPNETCORE_HOSTINGSTARTUPASSEMBLIES": "Microsoft.Azure.SignalR"
>      }
>    ```

[Возникли проблемы? Сообщите нам!](https://aka.ms/asrs/qsblazor)

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ресурсы, созданные при работе с этим руководством, удалите группу ресурсов через портал Azure.

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнаете, как:

> [!div class="checklist"]
> * создать простую комнату чата с помощью приложение Blazor Server;
> * изменить компоненты Razor;
> * использовать обработку событий и привязку данных в компонентах;
> * выполнить быстрое развертывание в Службе приложений Azure с помощью Visual Studio;
> * перенести локальную службу SignalR в Службу Azure SignalR.

Узнайте больше о высоком уровне доступности.
> [!div class="nextstepaction"]
> [Устойчивость и аварийное восстановление](signalr-concept-disaster-recovery.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

* [ASP.NET Core Blazor](/aspnet/core/blazor)
