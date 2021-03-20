---
title: Разработка & Настройка приложения "функции Azure" Azure SignalR
description: Сведения о разработке и настройке безсерверных приложений в режиме реального времени с помощью функций Azure и службы SignalR Azure
author: anthonychu
ms.service: signalr
ms.topic: conceptual
ms.date: 03/01/2019
ms.author: antchu
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 3d69b72012819e3d9099e447b9048fe07aea86d3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97858711"
---
# <a name="azure-functions-development-and-configuration-with-azure-signalr-service"></a>Azure Functions development and configuration with Azure SignalR Service (Разработка и настройка функций Azure с помощью Службы Azure SignalR)

Приложения функций Azure могут использовать [привязки службы Azure SignalR](../azure-functions/functions-bindings-signalr-service.md) для добавления возможностей в режиме реального времени. Клиентские приложения используют клиентские пакеты SDK, доступные на нескольких языках, для подключения к службе Azure SignalR и получения сообщений в режиме реального времени.

В этой статье описываются основные понятия разработки и настройки приложения-функции Azure, интегрированного со службой SignalR.

## <a name="signalr-service-configuration"></a>Конфигурация службы SignalR

Службу Azure SignalR можно настроить в различных режимах. При использовании с функциями Azure служба должна быть настроена в *Бессерверном* режиме.

На портал Azure выберите страницу *Параметры* для ресурса службы SignalR. Установите *режим обслуживания* " *бессерверный*".

![Режим Службы SignalR](media/signalr-concept-azure-functions/signalr-service-mode.png)

## <a name="azure-functions-development"></a>Разработка функций Azure

Для бессерверного приложения, работающего в реальном времени и созданного с помощью Функций Azure и Службы Azure SignalR, обычно требуется две функции Azure:

* функция negotiate, которую клиент вызывает, чтобы получить действительный маркер доступа Службы SignalR и URL-адрес конечной точки службы;
* Одна или несколько функций, которые обрабатывали сообщения из службы SignalR и отправляют сообщения или управляют членством в группе

### <a name="negotiate-function"></a>Функция Negotiate

Клиентскому приложению требуется допустимый маркер доступа для подключения к службе Azure SignalR. Маркер доступа может быть анонимным или прошедшим проверку подлинности по указанному ИДЕНТИФИКАТОРу пользователя. Для приложений службы SignalR без сервера требуется конечная точка HTTP с именем "Negotiate" для получения маркера и других сведений о соединении, таких как URL-адрес конечной точки службы SignalR.

Используйте функцию Azure, активируемую с помощью HTTP, и входную привязку *сигналрконнектионинфо* для создания объекта сведений о соединении. Функция должна иметь маршрут HTTP, который заканчивается на `/negotiate` .

[Модель на основе класса](#class-based-model) в C# не требует *сигналрконнектионинфо* входной привязки и может значительно упростить добавление пользовательских утверждений. См. статью о процессе [согласования в модели на основе классов](#negotiate-experience-in-class-based-model) .

Дополнительные сведения о создании функции Negotiate см. в [справочнике по входной привязке *сигналрконнектионинфо*](../azure-functions/functions-bindings-signalr-service-input.md).

Дополнительные сведения о создании маркера с проверкой подлинности см. в разделе [использование проверки подлинности службы приложений](#using-app-service-authentication).

### <a name="handle-messages-sent-from-signalr-service"></a>Обработку сообщений, отправленных из службы SignalR

Используйте привязку *триггера SignalR* для обработки сообщений, отправленных службой SignalR. Вы можете получать уведомления, когда клиенты отправляют сообщения или клиенты могут подключиться или отключиться.

Дополнительные сведения см. в [справочнике по привязке *триггера SignalR*](../azure-functions/functions-bindings-signalr-service-trigger.md).

Кроме того, необходимо настроить конечную точку функции в качестве вышестоящей, чтобы служба активировала функцию при наличии сообщения от клиента. Дополнительные сведения о настройке вышестоящей конфигурации см. в этом [документе](concept-upstream.md).

### <a name="sending-messages-and-managing-group-membership"></a>Отправка сообщений и управление членством в группах

Используйте выходную привязку *SignalR* для отправки сообщений клиентам, подключенным к службе Azure SignalR. Вы можете рассылать сообщения всем клиентам или отправить их подмножеству клиентов, которые прошли проверку подлинности с определенным ИДЕНТИФИКАТОРом пользователя или были добавлены в определенную группу.

Пользователи могут быть добавлены в одну или несколько групп. Вы также можете использовать выходную привязку *SignalR* для добавления или удаления пользователей в группы.

Дополнительные сведения см. в [справочнике по выходной привязке *SignalR*](../azure-functions/functions-bindings-signalr-service-output.md).

### <a name="signalr-hubs"></a>Концентраторы SignalR

SignalR имеет концепцию "концентраторов". Каждое клиентское подключение и каждое сообщение, отправленное из функций Azure, ограничены конкретным концентратором. Концентраторы можно использовать как способ разделения подключений и сообщений на логические пространства имен.

## <a name="class-based-model"></a>Модель на основе класса

Модель на основе класса выделяется для C#. Модель на основе классов может иметь совместимый интерфейс программирования на стороне сервера SignalR. Он имеет следующие возможности.

* Меньше работы по настройке: имя класса используется как `HubName` , имя метода используется как, `Event` а параметр `Category` — автоматически в соответствии с именем метода.
* Автоматическая привязка параметра: ни один `ParameterNames` атрибут `[SignalRParameter]` , ни не требуется. Параметры имеют автоматическую привязку к аргументам метода функции Azure по порядку.
* Удобный процесс вывода и согласования.

Эти функции демонстрируются в следующих кодах:

```cs
public class SignalRTestHub : ServerlessHub
{
    [FunctionName("negotiate")]
    public SignalRConnectionInfo Negotiate([HttpTrigger(AuthorizationLevel.Anonymous)]HttpRequest req)
    {
        return Negotiate(req.Headers["x-ms-signalr-user-id"], GetClaims(req.Headers["Authorization"]));
    }

    [FunctionName(nameof(OnConnected))]
    public async Task OnConnected([SignalRTrigger]InvocationContext invocationContext, ILogger logger)
    {
        await Clients.All.SendAsync(NewConnectionTarget, new NewConnection(invocationContext.ConnectionId));
        logger.LogInformation($"{invocationContext.ConnectionId} has connected");
    }

    [FunctionName(nameof(Broadcast))]
    public async Task Broadcast([SignalRTrigger]InvocationContext invocationContext, string message, ILogger logger)
    {
        await Clients.All.SendAsync(NewMessageTarget, new NewMessage(invocationContext, message));
        logger.LogInformation($"{invocationContext.ConnectionId} broadcast {message}");
    }

    [FunctionName(nameof(OnDisconnected))]
    public void OnDisconnected([SignalRTrigger]InvocationContext invocationContext)
    {
    }
}
```

Все функции, которые хотят использовать модель на основе класса, должны быть методом класса, который наследует от **серверлесшуб**. Имя класса `SignalRTestHub` в примере — это имя концентратора.

### <a name="define-hub-method"></a>Определение метода концентратора

Все методы концентратора **должны** иметь аргумент, `InvocationContext` объявленный `[SignalRTrigger]` атрибутом, и использовать конструктор без параметров. Затем **имя метода** обрабатывается как **событие** параметра.

По умолчанию, `category=messages` за исключением имени метода является одно из следующих имен:

* **Onconnected**: обрабатывается как `category=connections, event=connected`
* **Ondisconnectо**: обрабатывается как `category=connections, event=disconnected`

### <a name="parameter-binding-experience"></a>Интерфейс привязки параметров

В модели на основе класса `[SignalRParameter]` не требуется, так как все аргументы помечаются как по `[SignalRParameter]` умолчанию, за исключением одного из следующих ситуаций:

* Аргумент снабжен атрибутом привязки.
* Тип аргумента — `ILogger` или. `CancellationToken`
* Аргумент оформлен атрибутом `[SignalRIgnore]`

### <a name="negotiate-experience-in-class-based-model"></a>Процесс согласования в модели на основе классов

Вместо использования входной привязки SignalR `[SignalR]` согласование в модели на основе классов может быть более гибким. Базовый класс `ServerlessHub` содержит метод

```cs
SignalRConnectionInfo Negotiate(string userId = null, IList<Claim> claims = null, TimeSpan? lifeTime = null)
```

Эти функции пользователь настраивает `userId` или `claims` во время выполнения функции.

## <a name="use-signalrfilterattribute"></a>Используйте `SignalRFilterAttribute`.

Пользователь может наследовать и реализовывать абстрактный класс `SignalRFilterAttribute` . Если исключения вызываются в `FilterAsync` , `403 Forbidden` будут отправляться обратно клиентам.

В следующем примере показано, как реализовать фильтр клиента, позволяющий только `admin` вызывать `broadcast` .

```cs
[AttributeUsage(AttributeTargets.Method, AllowMultiple = true, Inherited = true)]
internal class FunctionAuthorizeAttribute: SignalRFilterAttribute
{
    private const string AdminKey = "admin";

    public override Task FilterAsync(InvocationContext invocationContext, CancellationToken cancellationToken)
    {
        if (invocationContext.Claims.TryGetValue(AdminKey, out var value) &&
            bool.TryParse(value, out var isAdmin) &&
            isAdmin)
        {
            return Task.CompletedTask;
        }

        throw new Exception($"{invocationContext.ConnectionId} doesn't have admin role");
    }
}
```

Используйте атрибут для авторизации функции.

```cs
[FunctionAuthorize]
[FunctionName(nameof(Broadcast))]
public async Task Broadcast([SignalRTrigger]InvocationContext invocationContext, string message, ILogger logger)
{
}
```

## <a name="client-development"></a>Разработка клиентских приложений

Клиентские приложения SignalR могут использовать клиентский пакет SDK SignalR на одном из нескольких языков, чтобы легко подключаться и получать сообщения от службы Azure SignalR.

### <a name="configuring-a-client-connection"></a>Настройка подключения клиента

Чтобы подключиться к службе SignalR, клиент должен завершить успешное согласование подключения, которое состоит из следующих действий.

1. Выполните запрос к конечной точке *Negotiate* HTTP, описанной выше, чтобы получить допустимые сведения о соединении.
1. Подключение к службе SignalR с помощью URL-адреса конечной точки службы и маркера доступа, полученных из конечной точки *согласования*

Пакеты SDK клиента SignalR уже содержат логику, необходимую для выполнения подтверждения согласования. Передайте в пакет SDK URL-адрес конечной точки для согласования, за исключением `negotiate` сегмента `HubConnectionBuilder` . Ниже приведен пример в JavaScript:

```javascript
const connection = new signalR.HubConnectionBuilder()
  .withUrl('https://my-signalr-function-app.azurewebsites.net/api')
  .build()
```

По соглашению пакет SDK автоматически добавляет `/negotiate` к URL-адресу и использует его для начала согласования.

> [!NOTE]
> Если вы используете JavaScript/TypeScript SDK в браузере, необходимо [включить общий доступ к ресурсам на разных источниках (CORS)](#enabling-cors) на приложение-функция.

Дополнительные сведения о том, как использовать клиентский пакет SDK для SignalR, см. в документации по вашему языку:

* [.NET Standard](/aspnet/core/signalr/dotnet-client)
* [JavaScript](/aspnet/core/signalr/javascript-client)
* [Java](/aspnet/core/signalr/java-client)

### <a name="sending-messages-from-a-client-to-the-service"></a>Отправка сообщений от клиента службе

Если для ресурса SignalR настроено восходящий [поток](concept-upstream.md) , вы можете отправить сообщения от клиента в функции Azure с помощью любого клиента SignalR. Ниже приведен пример в JavaScript:

```javascript
connection.send('method1', 'arg1', 'arg2');
```

## <a name="azure-functions-configuration"></a>Настройка функций Azure

Приложения-функции Azure, которые интегрируются со службой Azure SignalR, можно развернуть как любое типичное приложение-функцию Azure с помощью таких методов, как [непрерывное развертывание](../azure-functions/functions-continuous-deployment.md), [ZIP-развертывание](../azure-functions/deployment-zip-push.md)и [Запуск из пакета](../azure-functions/run-functions-from-deployment-package.md).

Однако существует несколько особых соображений для приложений, использующих привязки службы SignalR. Если клиент выполняется в браузере, необходимо включить CORS. Если для приложения требуется проверка подлинности, можно интегрировать конечную точку Negotiate с проверкой подлинности службы приложений.

### <a name="enabling-cors"></a>Включение CORS

Клиент JavaScript/TypeScript выполняет HTTP-запросы к функции Negotiate, чтобы инициировать согласование подключения. Если клиентское приложение размещается в домене, отличном от домена приложения-функции Azure, необходимо включить общий доступ к ресурсам на основе источника (CORS) в приложении-функции или браузер будет блокировать запросы.

#### <a name="localhost"></a>Localhost

При запуске приложения-функции на локальном компьютере можно добавить `Host` раздел в *local.settings.js* , чтобы включить CORS. В `Host` разделе добавьте два свойства:

* `CORS` — Введите базовый URL-адрес, который является источником клиентского приложения.
* `CORSCredentials` -Задайте значение `true` , чтобы разрешить запросы "вискредентиалс"

Пример

```json
{
  "IsEncrypted": false,
  "Values": {
    // values
  },
  "Host": {
    "CORS": "http://localhost:8080",
    "CORSCredentials": true
  }
}
```

#### <a name="cloud---azure-functions-cors"></a>Облако — функции Azure CORS

Чтобы включить CORS в приложении функции Azure, перейдите на экран конфигурации CORS на вкладке *функции платформы* приложения функции в портал Azure.

> [!NOTE]
> Конфигурация CORS пока недоступна в плане потребления Azure для функций Linux. Используйте службу [управления API Azure](#cloud---azure-api-management) для включения CORS.

Чтобы клиент SignalR вызывал функцию Negotiate, необходимо включить CORS с правами доступа-Control-Allow-Credential. Установите флажок, чтобы включить его.

В разделе *Разрешенные источники* добавьте запись с исходным URL-адресом исходной базы данных.

![Настройка CORS](media/signalr-concept-serverless-development-config/cors-settings.png)

#### <a name="cloud---azure-api-management"></a>Облако — управление API Azure

Служба управления API Azure предоставляет шлюз API, который добавляет возможности к существующим серверным службам. Его можно использовать для добавления CORS в приложение функции. Она предлагает уровень потребления с ценами оплаты за действие и ежемесячный бесплатный предоставление.

Сведения о том, как [импортировать приложение-функцию Azure](../api-management/import-function-app-as-api.md), см. в документации по управлению API. После импорта можно добавить политику входящего трафика, чтобы включить CORS с поддержкой Access-Control-Allow-Credentials.

```xml
<cors allow-credentials="true">
  <allowed-origins>
    <origin>https://azure-samples.github.io</origin>
  </allowed-origins>
  <allowed-methods>
    <method>GET</method>
    <method>POST</method>
  </allowed-methods>
  <allowed-headers>
    <header>*</header>
  </allowed-headers>
  <expose-headers>
    <header>*</header>
  </expose-headers>
</cors>
```

Настройте клиенты SignalR для использования URL-адреса управления API.

### <a name="using-app-service-authentication"></a>Использование проверки подлинности службы приложений

Функции Azure имеют встроенную проверку подлинности, поддерживающую популярные поставщики, такие как Facebook, Twitter, учетная запись Майкрософт, Google и Azure Active Directory. Эта функция может быть интегрирована с привязкой *сигналрконнектионинфо* для создания подключений к службе Azure SignalR, которая была проверена на идентификатор пользователя. Приложение может отсылать сообщения с помощью выходной привязки *SignalR* , предназначенного для этого идентификатора пользователя.

В портал Azure на вкладке *функции платформы* приложения-функции откройте окно Параметры *проверки подлинности и авторизации* . Чтобы настроить проверку подлинности с помощью выбранного поставщика удостоверений, следуйте инструкциям в документации по [проверке подлинности службы приложений](../app-service/overview-authentication-authorization.md) .

После настройки прошедшие проверку подлинности HTTP-запросы будут включать `x-ms-client-principal-name` `x-ms-client-principal-id` заголовки, содержащие имя пользователя и идентификатор пользователя, прошедшего проверку подлинности, соответственно.

Эти заголовки можно использовать в конфигурации привязки *сигналрконнектионинфо* для создания подключений, прошедших проверку подлинности. Ниже приведен пример функции Negotiate C#, которая использует `x-ms-client-principal-id` заголовок.

```csharp
[FunctionName("negotiate")]
public static SignalRConnectionInfo Negotiate(
    [HttpTrigger(AuthorizationLevel.Anonymous)]HttpRequest req,
    [SignalRConnectionInfo
        (HubName = "chat", UserId = "{headers.x-ms-client-principal-id}")]
        SignalRConnectionInfo connectionInfo)
{
    // connectionInfo contains an access key token with a name identifier claim set to the authenticated user
    return connectionInfo;
}
```

Затем можно отправить сообщения этому пользователю, задав `UserId` свойство сообщения SignalR.

```csharp
[FunctionName("SendMessage")]
public static Task SendMessage(
    [HttpTrigger(AuthorizationLevel.Anonymous, "post")]object message,
    [SignalR(HubName = "chat")]IAsyncCollector<SignalRMessage> signalRMessages)
{
    return signalRMessages.AddAsync(
        new SignalRMessage
        {
            // the message will only be sent to these user IDs
            UserId = "userId1",
            Target = "newMessage",
            Arguments = new [] { message }
        });
}
```

Сведения о других языках см. в справочнике по [привязкам службы Azure SignalR](../azure-functions/functions-bindings-signalr-service.md) для функций Azure.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как разрабатывать и настраивать приложения службы SignalR для бессерверных приложений с помощью функций Azure. Попробуйте создать приложение самостоятельно с помощью одного из быстрых запусков или руководств на [странице обзора службы SignalR](index.yml).
