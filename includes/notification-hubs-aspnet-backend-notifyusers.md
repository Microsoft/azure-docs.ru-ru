---
title: включить файл
description: Включение файла, который содержит код для создания проекта серверной части веб-API ASP.NET.
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 09/11/2019
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 6f490b6f25112ed8a10bbd865070bd07ea3ee84f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96016964"
---
## <a name="create-the-webapi-project"></a>Создание проекта веб-API

В этой статье описывается создание серверной части веб-API ASP.NET. Этот процесс состоит из трех главных задач.

- **Аутентификация клиентов.** Вы добавите обработчик сообщений для аутентификации клиентских запросов и связывания пользователя с запросом.
- **Регистрация для получения уведомлений с помощью серверной части веб-API.** Вы добавите контроллер для обработки новых регистраций клиентских устройств для получения уведомлений. Имя пользователя, прошедшего аутентификацию, автоматически добавляется в регистрацию как [тег](../articles/notification-hubs/notification-hubs-tags-segment-push-message.md).
- **Отправка уведомлений клиентам.** Вы также добавите контроллер, который позволит активировать безопасную отправку push-уведомлений устройствам и клиентам, связанным с тегом.

Чтобы создать серверную часть веб-API ASP.NET, сделайте следующее:

> [!IMPORTANT]
> Если вы используете Visual Studio 2015 или более ранние версии, перед работой с этим руководством установите последнюю версию диспетчера пакетов NuGet для Visual Studio.
>
>Чтобы проверить, запустите Visual Studio. Откройте меню **Средства** и выберите пункт **Расширения и обновления**. Найдите **диспетчер пакетов NuGet** в своей версии Visual Studio и убедитесь, что у вас установлена последняя версия. Если вы используете не последнюю версию, удалите ее, а затем переустановите диспетчер пакетов NuGet.

![Снимок экрана: диалоговое окно "Расширения и обновления" с выделенным диспетчером пакетов NuGet для Visual Studio][B4]

> [!NOTE]
> Убедитесь, что вы установили [пакет SDK для Azure](https://azure.microsoft.com/downloads/) для Visual Studio, используемый для развертывания веб-сайта.

1. Запустите Visual Studio или Visual Studio Express.

2. Щелкните **Обозреватель серверов** и войдите в свою учетную запись Azure. Чтобы создать ресурсы веб-сайта в своей учетной записи, вы должны войти.

3. В Visual Studio щелкните правой кнопкой мыши решение Visual Studio и последовательно выберите **Добавить** и **Новый проект**.
4. Разверните узел **Visual C#**, выберите **Веб** и щелкните **Веб-приложение ASP.NET**.

5. В поле **Имя** введите **AppBackend**, а затем нажмите кнопку **ОК**.

    ![Окно создания проекта][B1]

6. В окне **Создание проекта ASP.NET** установите флажок **Веб-API**, а затем нажмите кнопку **ОК**.

    ![Окно создания проекта ASP.NET][B2]

7. В окне **Настройка веб-приложения Microsoft Azure** выберите подписку, а затем в списке **План службы приложений** выполните одно из следующих действий:

    * Выберите план службы приложений, который вы уже создали.
    * Выберите **Создать план служб приложений**, а затем создайте его.

   База данных для этого руководства не требуется. Выбрав план служб приложений, нажмите кнопку **ОК**, чтобы создать проект.

    ![Диалоговое окно "Настройка веб-приложения Microsoft Azure"][B5]

    Если вы не видите эту страницу для настройки плана службы приложений, продолжайте выполнять руководство. Его можно настроить позднее при публикации приложения. 

## <a name="authenticate-clients-to-the-webapi-backend"></a>Аутентификация клиентов в серверной части веб-API.

В этом разделе вы создадите класс обработчика сообщений с именем **AuthenticationTestHandler** для новой серверной части. Этот класс является производным от [DelegatingHandler](/previous-versions/visualstudio/hh193679(v=vs.118)). Он добавляется в качестве обработчика сообщений, чтобы обрабатывать все запросы, поступающие в серверную часть.

1. В обозревателе решений щелкните правой кнопкой мыши проект **AppBackend**, выберите **Добавить**, а затем щелкните **Класс**.
2. Присвойте новому классу имя **AuthenticationTestHandler.cs** и нажмите кнопку **Добавить**, чтобы создать класс. Этот класс используется для аутентификации пользователей с помощью *обычной проверки подлинности* для простоты. Ваше приложение может использовать любую схему аутентификации.
3. В AuthenticationTestHandler.cs добавьте следующие операторы `using` :

    ```csharp
    using System.Net.Http;
    using System.Threading;
    using System.Security.Principal;
    using System.Net;
    using System.Text;
    using System.Threading.Tasks;
    ```

4. В классе AuthenticationTestHandler.cs замените определение `AuthenticationTestHandler` следующим кодом:

    Обработчик авторизует запрос, если выполнены все три следующих условия:

   * Запрос включен в заголовок *авторизации*.
   * Запрос использует *обычную* проверку подлинности.
   * Строка имени пользователя и строка пароля являются одной стройкой.

   В противном случае запрос отклоняется. Этот способ аутентификации нельзя назвать настоящим методом аутентификации и авторизации. Это простой пример для этого руководства.

   Если сообщение запроса аутентифицируется и авторизуется `AuthenticationTestHandler`, пользователь обычной проверки подлинности подключается к текущему запросу в [HttpContext](/dotnet/api/system.web.httpcontext.current). Позднее информацию о пользователе в HttpContext будет использовать другой контроллер (RegisterController), чтобы добавить [тег](/previous-versions/azure/azure-services/dn530749(v=azure.100)) в запрос на регистрацию для получения уведомлений.

    ```csharp
    public class AuthenticationTestHandler : DelegatingHandler
    {
        protected override Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, CancellationToken cancellationToken)
        {
            var authorizationHeader = request.Headers.GetValues("Authorization").First();

            if (authorizationHeader != null && authorizationHeader
                .StartsWith("Basic ", StringComparison.InvariantCultureIgnoreCase))
            {
                string authorizationUserAndPwdBase64 =
                    authorizationHeader.Substring("Basic ".Length);
                string authorizationUserAndPwd = Encoding.Default
                    .GetString(Convert.FromBase64String(authorizationUserAndPwdBase64));
                string user = authorizationUserAndPwd.Split(':')[0];
                string password = authorizationUserAndPwd.Split(':')[1];

                if (VerifyUserAndPwd(user, password))
                {
                    // Attach the new principal object to the current HttpContext object
                    HttpContext.Current.User =
                        new GenericPrincipal(new GenericIdentity(user), new string[0]);
                    System.Threading.Thread.CurrentPrincipal =
                        System.Web.HttpContext.Current.User;
                }
                else return Unauthorized();
            }
            else return Unauthorized();

            return base.SendAsync(request, cancellationToken);
        }

        private bool VerifyUserAndPwd(string user, string password)
        {
            // This is not a real authentication scheme.
            return user == password;
        }

        private Task<HttpResponseMessage> Unauthorized()
        {
            var response = new HttpResponseMessage(HttpStatusCode.Forbidden);
            var tsc = new TaskCompletionSource<HttpResponseMessage>();
            tsc.SetResult(response);
            return tsc.Task;
        }
    }
    ```

    > [!NOTE]
    > Примечание о безопасности. Класс `AuthenticationTestHandler` не обеспечивает настоящую аутентификацию. Этот класс используется лишь для имитации базовой проверки подлинности и не является безопасным. В реальных приложениях и службах необходимо реализовать безопасный механизм проверки подлинности.
5. Чтобы зарегистрировать обработчик событий, добавьте в конец метода `Register` в классе **App_Start/WebApiConfig.cs** следующий код:

    ```csharp
    config.MessageHandlers.Add(new AuthenticationTestHandler());
    ```
6. Сохраните изменения.

## <a name="register-for-notifications-by-using-the-webapi-backend"></a>Регистрация для получения уведомлений с помощью серверной части веб-API.

В этом разделе вы добавите новый контроллер в серверную часть веб-API, которая будет обрабатывать запросы на регистрацию пользователя и устройства для получения уведомлений с помощью клиентской библиотеки центров уведомлений. Контроллер будет добавлять тег пользователя для пользователя, который прошел аутентификацию и подключен к HttpContext с помощью `AuthenticationTestHandler`. Тег имеет формат строки: `"username:<actual username>"`.

1. В обозревателе решений щелкните правой кнопкой мыши проект **AppBackend** и выберите пункт **Управление пакетами NuGet**.

2. На панели слева выберите **В сети**, затем в поле **поиска** введите **Microsoft.Azure.NotificationHubs**.

3. В списке результатов выберите **Центры уведомлений Microsoft Azure**, а затем нажмите кнопку **Установить**. Завершите установку и закройте окно диспетчера пакетов NuGet.

    Это действие добавляет ссылку на пакет SDK для Центров уведомлений Azure с помощью [пакета NuGet Microsoft.Azure.Notification Hubs](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/).

4. Создайте файл класса, представляющий подключение к концентратору уведомлений, который используется для отправки уведомлений. В обозревателе решений щелкните правой кнопкой мыши папку **Модели**, выберите **Добавить**, а затем щелкните **Класс**. Назовите новый класс **Notifications.cs** и нажмите кнопку **Добавить**, чтобы создать класс.

    ![Окно добавления нового элемента][B6]

5. В Notifications.cs добавьте следующий оператор `using` в начало файла:

    ```csharp
    using Microsoft.Azure.NotificationHubs;
    ```

6. Замените определение класса `Notifications` следующим кодом и замените два заполнителя строкой подключения (с полным доступом) для своего концентратора уведомлений и именем концентратора (доступно на [портале Azure](https://portal.azure.com)):

    ```csharp
    public class Notifications
    {
        public static Notifications Instance = new Notifications();

        public NotificationHubClient Hub { get; set; }

        private Notifications() {
            Hub = NotificationHubClient.CreateClientFromConnectionString("<your hub's DefaultFullSharedAccessSignature>",
                                                                            "<hub name>");
        }
    }
    ```
    > [!IMPORTANT]
    > Прежде чем продолжить, введите **имя** и **DefaultFullSharedAccessSignature** вашего концентратора. 
    
7. Теперь создайте новый контроллер с именем **RegisterController**. В обозревателе решений щелкните правой кнопкой мыши папку **Контроллеры**, выберите **Добавить**, а затем щелкните **Контроллер**.

8. Щелкните элемент **Контроллер Web API 2 — пустой** и нажмите кнопку **Добавить**.

    ![Окно добавления шаблона][B7]

9. В поле **Имя контроллера** введите имя нового класса **RegisterController**, а затем нажмите кнопку **Добавить**.

    ![Окно добавления контроллера][B8]

10. В RegiterController.cs добавьте следующие операторы `using` :

    ```csharp
    using Microsoft.Azure.NotificationHubs;
    using Microsoft.Azure.NotificationHubs.Messaging;
    using AppBackend.Models;
    using System.Threading.Tasks;
    using System.Web;
    ```
11. Добавьте в определение класса `RegisterController` следующий код: В этом коде вы добавите тег для пользователя, который подключен к HttpContext. Пользователь прошел аутентификацию и подключен к HttpContext с помощью добавленного фильтра сообщений `AuthenticationTestHandler`. Вы можете также добавить дополнительные проверки, чтобы убедиться, что у пользователя есть право регистрации запрошенных тегов.

    ```csharp
    private NotificationHubClient hub;

    public RegisterController()
    {
        hub = Notifications.Instance.Hub;
    }

    public class DeviceRegistration
    {
        public string Platform { get; set; }
        public string Handle { get; set; }
        public string[] Tags { get; set; }
    }

    // POST api/register
    // This creates a registration id
    public async Task<string> Post(string handle = null)
    {
        string newRegistrationId = null;

        // make sure there are no existing registrations for this push handle (used for iOS and Android)
        if (handle != null)
        {
            var registrations = await hub.GetRegistrationsByChannelAsync(handle, 100);

            foreach (RegistrationDescription registration in registrations)
            {
                if (newRegistrationId == null)
                {
                    newRegistrationId = registration.RegistrationId;
                }
                else
                {
                    await hub.DeleteRegistrationAsync(registration);
                }
            }
        }

        if (newRegistrationId == null) 
            newRegistrationId = await hub.CreateRegistrationIdAsync();

        return newRegistrationId;
    }

    // PUT api/register/5
    // This creates or updates a registration (with provided channelURI) at the specified id
    public async Task<HttpResponseMessage> Put(string id, DeviceRegistration deviceUpdate)
    {
        RegistrationDescription registration = null;
        switch (deviceUpdate.Platform)
        {
            case "mpns":
                registration = new MpnsRegistrationDescription(deviceUpdate.Handle);
                break;
            case "wns":
                registration = new WindowsRegistrationDescription(deviceUpdate.Handle);
                break;
            case "apns":
                registration = new AppleRegistrationDescription(deviceUpdate.Handle);
                break;
            case "fcm":
                registration = new FcmRegistrationDescription(deviceUpdate.Handle);
                break;
            default:
                throw new HttpResponseException(HttpStatusCode.BadRequest);
        }

        registration.RegistrationId = id;
        var username = HttpContext.Current.User.Identity.Name;

        // add check if user is allowed to add these tags
        registration.Tags = new HashSet<string>(deviceUpdate.Tags);
        registration.Tags.Add("username:" + username);

        try
        {
            await hub.CreateOrUpdateRegistrationAsync(registration);
        }
        catch (MessagingException e)
        {
            ReturnGoneIfHubResponseIsGone(e);
        }

        return Request.CreateResponse(HttpStatusCode.OK);
    }

    // DELETE api/register/5
    public async Task<HttpResponseMessage> Delete(string id)
    {
        await hub.DeleteRegistrationAsync(id);
        return Request.CreateResponse(HttpStatusCode.OK);
    }

    private static void ReturnGoneIfHubResponseIsGone(MessagingException e)
    {
        var webex = e.InnerException as WebException;
        if (webex.Status == WebExceptionStatus.ProtocolError)
        {
            var response = (HttpWebResponse)webex.Response;
            if (response.StatusCode == HttpStatusCode.Gone)
                throw new HttpRequestException(HttpStatusCode.Gone.ToString());
        }
    }
    ```
12. Сохраните изменения.

## <a name="send-notifications-from-the-webapi-backend"></a>Отправка уведомлений из серверной части веб-API.

В этом разделе вы добавите новый контроллер, который позволяет клиентским устройствам отправлять уведомления. Уведомление основано на теге имени пользователя, который использует библиотеку .NET центров уведомлений Azure в серверной части веб-API ASP.NET.

1. Создайте еще один контроллер с именем **NotificationsController** так же, как вы создали **RegisterController** в предыдущем разделе.

2. В NotificationsController.cs добавьте следующие операторы `using` :

    ```csharp
    using AppBackend.Models;
    using System.Threading.Tasks;
    using System.Web;
    ```
3. Добавьте следующий метод в класс **NotificationsController**:

    Этот код отправляет тип уведомлений, основанный на параметре `pns` системы отправки уведомлений платформы (PNS). Значение `to_tag` используется для задания тега *имени пользователя* в сообщении. Этот тег должен соответствовать тегу имени пользователя активной регистрации центра уведомлений. Сообщение уведомления извлекается из текста запроса POST и форматируется для целевого PNS.

    Поддержка форматов уведомлений зависит от того, какую систему PNS используют поддерживаемые устройства. Например, на устройствах Windows можно использовать [всплывающие уведомления с помощью WNS](/uwp/schemas/tiles/toastschema/schema-root), которые не поддерживает другая система PNS. В этом случае серверная часть решения конвертирует уведомление в формат, соответствующий PNS устройств, которые вы планируете поддерживать. Затем используйте соответствующий API отправки для [класса NotificationHubClient](/dotnet/api/microsoft.azure.notificationhubs.notificationhubclient).

    ```csharp
    public async Task<HttpResponseMessage> Post(string pns, [FromBody]string message, string to_tag)
    {
        var user = HttpContext.Current.User.Identity.Name;
        string[] userTag = new string[2];
        userTag[0] = "username:" + to_tag;
        userTag[1] = "from:" + user;

        Microsoft.Azure.NotificationHubs.NotificationOutcome outcome = null;
        HttpStatusCode ret = HttpStatusCode.InternalServerError;

        switch (pns.ToLower())
        {
            case "wns":
                // Windows 8.1 / Windows Phone 8.1
                var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" + 
                            "From " + user + ": " + message + "</text></binding></visual></toast>";
                outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);
                break;
            case "apns":
                // iOS
                var alert = "{\"aps\":{\"alert\":\"" + "From " + user + ": " + message + "\"}}";
                outcome = await Notifications.Instance.Hub.SendAppleNativeNotificationAsync(alert, userTag);
                break;
            case "fcm":
                // Android
                var notif = "{ \"data\" : {\"message\":\"" + "From " + user + ": " + message + "\"}}";
                outcome = await Notifications.Instance.Hub.SendFcmNativeNotificationAsync(notif, userTag);
                break;
        }

        if (outcome != null)
        {
            if (!((outcome.State == Microsoft.Azure.NotificationHubs.NotificationOutcomeState.Abandoned) ||
                (outcome.State == Microsoft.Azure.NotificationHubs.NotificationOutcomeState.Unknown)))
            {
                ret = HttpStatusCode.OK;
            }
        }

        return Request.CreateResponse(ret);
    }
    ```
4. Нажмите клавишу **F5**, чтобы запустить приложение и убедиться, что все правильно работает. Приложение откроется в веб-браузере и отобразится на домашней странице ASP.NET.

## <a name="publish-the-new-webapi-backend"></a>Публикация новой серверной части веб-API.

Теперь разверните приложение на веб-сайте Azure, чтобы сделать его доступным для всех устройств.

1. Щелкните правой кнопкой мыши проект **AppBackend** и нажмите кнопку **Опубликовать**.

2. Выберите **службу приложений Microsoft Azure**, в которой вы хотите опубликовать приложение, и нажмите кнопку \*\*Опубликовать. Откроется окно создания службы приложений. В этом окне можно создать все необходимые ресурсы Azure для запуска веб-приложения ASP.NET в Azure.

    ![Плитка службы приложений Microsoft Azure][B15]

3. В окне **Создание службы приложений** выберите свою учетную запись Azure. Выберите **Изменить тип** > **Веб-приложение**. Не изменяйте стандартное **имя веб-приложения**. Выберите **подписку**, **группу ресурсов** и **план службы приложений**.

4. Нажмите кнопку **создания**.

5. Запишите свойство **URL-адрес сайта** в разделе **Сводка**. Это URL-адрес *конечной точки внутренней части*.

6. Нажмите **Публиковать**.

Когда мастер завершит работу, веб-приложение ASP.NET будет опубликовано в Azure и откроется в браузере по умолчанию.  Приложение также будет отображаться в службах приложений Azure.

В URL-адресе используется имя веб-приложения, указанное ранее, в формате http://<имя_приложения>.azurewebsites.net.

[B1]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push1.png
[B2]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push2.png
[B3]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push3.png
[B4]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push4.png
[B5]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push5.png
[B6]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push6.png
[B7]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push7.png
[B8]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push8.png
[B14]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push14.png
[B15]: ./media/notification-hubs-aspnet-backend-notifyusers/publish-to-app-service.png
[B16]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users16.PNG
[B18]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users18.PNG