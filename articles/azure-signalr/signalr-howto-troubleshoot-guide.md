---
title: Руководство по устранению неполадок для Службы Azure SignalR
description: Сведения об устранении распространенных неполадок
author: yjin81
ms.service: signalr
ms.topic: conceptual
ms.date: 11/06/2020
ms.author: yajin1
ms.openlocfilehash: e26def56fbd03626c3efc660db57012ee1b767ea
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105048210"
---
# <a name="troubleshooting-guide-for-azure-signalr-service-common-issues"></a>Руководство по устранению неполадок службы Azure SignalR

Это руководство содержит полезные рекомендации по устранению неполадок на основе распространенных проблем, которые пользователи составили и устраняют за последние годы.

## <a name="access-token-too-long"></a>Слишком длинный маркер доступа

### <a name="possible-errors"></a>Возможные ошибки

* На стороне клиента `ERR_CONNECTION_`
* 414 — слишком длинный универсальный код ресурса (URI)
* 413 (слишком большой объем полезных данных)
* Длина маркера доступа не должна превышать 4 КБ. 413 — размер запрашиваемой сущности слишком большой

### <a name="root-cause"></a>Первопричина

Для HTTP/2 Максимальная длина для одного заголовка равна **4 КБ**, поэтому при использовании браузера для доступа к службе Azure произойдет ошибка `ERR_CONNECTION_` для этого ограничения.

Для клиентов HTTP/1.1 или C# максимальная длина URI составляет **12 КБ**, максимальная длина заголовка — **16 КБ**.

При использовании пакета SDK версии **1.0.6** или более поздней `/negotiate` выдается исключение, `413 Payload Too Large` Если созданный маркер доступа больше **4 КБ**.

### <a name="solution"></a>Решение

По умолчанию утверждения из `context.User.Claims` включаются при создании маркера доступа JWT для **АСРС**(Zure игнал **R** **s** ервице), чтобы утверждения сохранялись и можно было передавать из **АСРС** в, `Hub` когда клиент подключается к `Hub` .

В некоторых случаях `context.User.Claims` используется для хранения большого количества сведений для сервера приложений, большинство из которых не используются, `Hub` а другими компонентами.

Созданный маркер доступа передается через сеть, а для соединений WebSocket и SSE маркеры доступа передаются через строки запроса. Поэтому мы рекомендуем передавать на сервер приложений только **необходимые** утверждения от клиента через **АСРС** , когда это необходимо.

`ClaimsProvider`Для настройки утверждений, передаваемых в **АСРС** внутри маркера доступа, можно настроить.

Для ASP.NET Core:

```csharp
services.AddSignalR()
        .AddAzureSignalR(options =>
            {
                // pick up necessary claims
                options.ClaimsProvider = context => context.User.Claims.Where(...);
            });
```

Для ASP.NET:

```csharp
services.MapAzureSignalR(GetType().FullName, options =>
            {
                // pick up necessary claims
                options.ClaimsProvider = context.Authentication?.User.Claims.Where(...);
            });
```

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="tls-12-required"></a>Требуется TLS 1,2

### <a name="possible-errors"></a>Возможные ошибки

* ASP.NET об ошибке "сервер недоступен" [#279](https://github.com/Azure/azure-signalr/issues/279)
* ASP.NET "Подключение неактивно, данные не могут быть отправлены в службу". [#324](https://github.com/Azure/azure-signalr/issues/324) об ошибке
* "Произошла ошибка при выполнении HTTP-запроса к https:// <API endpoint> . Эта ошибка может быть вызвана тем, что сертификат сервера неправильно настроен с HTTP.SYS в случае HTTPS. Эта ошибка также может быть вызвана несоответствием привязки безопасности между клиентом и сервером. "

### <a name="root-cause"></a>Первопричина

Служба Azure поддерживает только TLS 1.2 в целях безопасности. В .NET Framework возможно, что TLS 1.2 не является протоколом по умолчанию. В результате невозможно успешно установить подключения сервера к АСРС.

### <a name="troubleshooting-guide"></a>Руководство по устранению неполадок

1. Если эту ошибку можно воспроизвести локально, снимите флажок *только мой код* и вызывайте все исключения CLR и отлаживать сервер приложений локально, чтобы увидеть, что вызывает исключение.
    * Снять *только мой код*

        :::image type="content" source="./media/signalr-howto-troubleshoot-guide/uncheck-just-my-code.png" alt-text="Снять Только мой код":::

    * Выдавать исключения CLR

        :::image type="content" source="./media/signalr-howto-troubleshoot-guide/throw-clr-exceptions.png" alt-text="Выдавать исключения CLR":::

    * См. исключения, возникающие при отладке кода на стороне сервера приложений:
        
        :::image type="content" source="./media/signalr-howto-troubleshoot-guide/tls-throws.png" alt-text="Исключение":::

2. Для ASP.NET можно также добавить следующий код в, `Startup.cs` чтобы включить подробную трассировку и просмотреть ошибки из журнала.

    ```cs
    app.MapAzureSignalR(this.GetType().FullName);
    // Make sure this switch is called after MapAzureSignalR
    GlobalHost.TraceManager.Switch.Level = SourceLevels.Information;
    ```

### <a name="solution"></a>Решение

Добавьте в запуск следующий код:

```csharp
ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;
```

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="400-bad-request-returned-for-client-requests"></a>400 для клиентских запросов возвращен неверный запрос

### <a name="root-cause"></a>Первопричина

Проверьте, содержит ли ваш запрос клиента несколько `hub` строк запроса. `hub` сохраненный параметр запроса, и 400 выдаст исключение, если служба обнаружит `hub` в запросе более одного.

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="401-unauthorized-returned-for-client-requests"></a>401 Unauthorized returned for client requests (Возврат ошибки с кодом "401 — не санкционировано" для клиентских запросов)

### <a name="root-cause"></a>Первопричина

В настоящее время время жизни маркера JWT по умолчанию — 1 час.

Для ASP.NET Core SignalR, когда он использует тип транспорта WebSocket, это нормально.

Для другого типа транспорта ASP.NET Core SignalR, SSE и длительного опроса, это означает, что по умолчанию подключение может храниться не более 1 часа.

Для ASP.NET SignalR клиент отправляет `/ping` службе запрос на KeepAlive время от времени, когда `/ping` происходит сбой, клиент **прерывает** подключение и никогда не переключается. Это означает, что для ASP.NET SignalR время существования токена по умолчанию делает подключение продолжительностью не **более 1 часа для всех** типов транспорта.

### <a name="solution"></a>Решение

В целях безопасности не рекомендуется расширять TTL. Мы рекомендуем добавить логику повторного подключения из клиента, чтобы перезапустить подключение при возникновении такой ошибки 401. Когда клиент перезапускает подключение, он будет согласовываться с сервером приложений для повторного получения маркера JWT и получения обновленного маркера.

Установите [здесь](#restart_connection) , чтобы перезапустить клиентские подключения.

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="404-returned-for-client-requests"></a>404 returned for client requests (Возврат ошибки с кодом 404 для клиентских запросов)

Для постоянного подключения SignalR сначала `/negotiate` необходимо установить службу Azure SignalR, а затем — реальное подключение к службе Azure SignalR.

### <a name="troubleshooting-guide"></a>Руководство по устранению неполадок

* Чтобы [Просмотреть исходящие запросы](#view_request) на получение запроса от клиента к службе, необходимо выполнить следующие требования.
* Проверьте URL-адрес запроса при возникновении ошибки 404. Если URL-адрес предназначен для веб-приложения и аналогичен `{your_web_app}/hubs/{hubName}` , проверьте, имеет ли клиент `SkipNegotiation` `true` . При использовании Azure SignalR клиент получает URL-адрес перенаправления при первом согласовании с сервером приложений. Клиент **не** должен пропускать согласование при использовании Azure SignalR.
* Еще один 404 может произойти, когда запрос на подключение обрабатывается более **5** секунд после `/negotiate` вызова. Проверьте метку времени запроса клиента и откройте сообщение о том, что в случае, если запрос к службе имеет слишком много откликов.

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="404-returned-for-aspnet-signalrs-reconnect-request"></a>404 возвращается для запроса на повторное подключение SignalR ASP.NET

Для ASP.NET SignalR при [удалении клиентского соединения](#client_connection_drop)он повторно подключается в `connectionId` течение трех раз, прежде чем остановить подключение. `/reconnect` может помочь, если подключение отбрасывается из-за временных неполадок сети, которые `/reconnect` могут успешно восстановить постоянное подключение. В других случаях, например, подключение клиента отбрасывается из-за разрыва соединения с перенаправленным сервером, или служба SignalR имеет некоторые внутренние ошибки, такие как перезапуск экземпляра, отработка отказа или развертывание, поэтому соединение больше не существует, поэтому `/reconnect` возвращается `404` . Это ожидаемое поведение в `/reconnect` и после трех раз, когда повторная попытка подключения прекращается. Мы рекомендуем подать логику [перезапуска подключения](#restart_connection) при остановке соединения.

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="429-too-many-requests-returned-for-client-requests"></a>429 (слишком много запросов) возвращено для клиентских запросов

Есть два способа.

### <a name="concurrent-connection-count-exceeds-limit"></a>Число **одновременных** подключений превышает предел

Для **свободных** экземпляров ограничение числа **одновременных** подключений равно 20 для **стандартных** экземпляров, ограничение числа **одновременных** подключений **на единицу** равно 1 K, что означает, что Unit100 допускает одновременные подключения 100-K.

Подключения включают как клиентские, так и серверные соединения. Проверьте [,](./signalr-concept-messages-and-connections.md#how-connections-are-counted) как подсчитываются подключения.

### <a name="too-many-negotiate-requests-at-the-same-time"></a>Слишком много запросов на согласование одновременно

Мы рекомендуем установить произвольную задержку перед повторной попыткой подключения, [чтобы проверить примеры](#restart_connection) повторных попыток.

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="500-error-when-negotiate-azure-signalr-service-is-not-connected-yet-please-try-again-later"></a>500 ошибка при согласовании: служба Azure SignalR еще не подключена, повторите попытку позже.

### <a name="root-cause"></a>Первопричина

Эта ошибка возникает при отсутствии подключения сервера к службе SignalR Azure.

### <a name="troubleshooting-guide"></a>Руководство по устранению неполадок

Включите трассировку на стороне сервера, чтобы узнать сведения об ошибке, когда сервер пытается подключиться к службе Azure SignalR.

### <a name="enable-server-side-logging-for-aspnet-core-signalr"></a>Включение ведения журнала на стороне сервера для ASP.NET Core SignalR

Ведение журнала на стороне сервера для ASP.NET Core SignalR интегрируется с `ILogger` [журналом](/aspnet/core/fundamentals/logging/?tabs=aspnetcore2x&view=aspnetcore-2.1&preserve-view=true) на основе ASP.NET Core Framework. Вы можете включить ведение журнала на стороне сервера с помощью `ConfigureLogging` , выбрав пример использования следующим образом:

```csharp
.ConfigureLogging((hostingContext, logging) =>
        {
            logging.AddConsole();
            logging.AddDebug();
        })
```

Категории средств ведения журнала для Azure SignalR всегда начинаются с `Microsoft.Azure.SignalR` . Чтобы включить подробные журналы из Azure SignalR, настройте предыдущие префиксы для `Debug` уровня **appsettings.jsв** файле, как показано ниже:

```json
{
    "Logging": {
        "LogLevel": {
            ...
            "Microsoft.Azure.SignalR": "Debug",
            ...
        }
    }
}
```

#### <a name="enable-server-side-traces-for-aspnet-signalr"></a>Включение трассировки на стороне сервера для ASP.NET SignalR

При использовании пакета SDK версии >= `1.0.0` можно включить трассировку, добавив следующую команду в `web.config` : ([Details](https://github.com/Azure/azure-signalr/issues/452#issuecomment-478858102)).

```xml
<system.diagnostics>
    <sources>
      <source name="Microsoft.Azure.SignalR" switchName="SignalRSwitch">
        <listeners>
          <add name="ASRS" />
        </listeners>
      </source>
    </sources>
    <!-- Sets the trace verbosity level -->
    <switches>
      <add name="SignalRSwitch" value="Information" />
    </switches>
    <!-- Specifies the trace writer for output -->
    <sharedListeners>
      <add name="ASRS" type="System.Diagnostics.TextWriterTraceListener" initializeData="asrs.log.txt" />
    </sharedListeners>
    <trace autoflush="true" />
  </system.diagnostics>
```

<a name="client_connection_drop"></a>

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="client-connection-drops"></a>Разрывы соединения с клиентом

Когда клиент подключается к Azure SignalR, постоянное подключение между клиентом и Azure SignalR иногда может быть удалено по разным причинам. В этом разделе описываются некоторые возможности, вызывающие такое удаление подключения, и приводятся некоторые рекомендации по определению основной причины.

### <a name="possible-errors-seen-from-the-client-side"></a>Возможные ошибки со стороны клиента

* `The remote party closed the WebSocket connection without completing the close handshake`
* `Service timeout. 30.00ms elapsed without receiving a message from service.`
* `{"type":7,"error":"Connection closed with an error."}`
* `{"type":7,"error":"Internal server error."}`

### <a name="root-cause"></a>Первопричина

Клиентские подключения могут удаляться при различных обстоятельствах:
* При `Hub` вырождении исключений с входящим запросом.
* Когда соединение с сервером, на которое направляется клиент, удаляется, см. ниже в разделе с подробными сведениями о [падении соединения с сервером](#server_connection_drop).
* При возникновении проблемы с сетевым подключением между клиентом и службой SignalR.
* Когда служба SignalR имеет некоторые внутренние ошибки, такие как перезапуск экземпляра, отработка отказа, развертывание и т. д.

### <a name="troubleshooting-guide"></a>Руководство по устранению неполадок

1. Откройте журнал на стороне сервера приложений, чтобы проверить, не произошло ли что-либо непредвиденное
2. Проверьте журнал событий на стороне сервера приложений, чтобы узнать, не перезапущен ли сервер приложений.
3. Создайте ошибку, предоставляющую время, и отправьте имя ресурса по адресу

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="client-connection-increases-constantly"></a>Подключение клиента постоянно увеличивается

Это может быть вызвано неправильным использованием клиентского соединения. Если кто-то забыл закрыть или ликвидировать клиент SignalR, соединение остается открытым.

### <a name="possible-errors-seen-from-the-signalrs-metrics-that-is-in-monitoring-section-of-azure-portal-resource-menu"></a>Возможные ошибки в метриках SignalR, которые находятся в разделе "Мониторинг" портал Azure меню ресурсов

Клиентские подключения постоянно наиграют в течение длительного времени в метриках Azure SignalR.

:::image type="content" source="./media/signalr-howto-troubleshoot-guide/client-connection-increasing-constantly.jpg" alt-text="Подключение клиента постоянно растет":::

### <a name="root-cause"></a>Первопричина

Подключение клиента SignalR `DisposeAsync` никогда не вызывается, подключение остается открытым.

### <a name="troubleshooting-guide"></a>Руководство по устранению неполадок

Проверьте, **не** закрывается ли клиент SignalR.

### <a name="solution"></a>Решение

Убедитесь, что подключение закрыто. Вызов метода `HubConnection.DisposeAsync()` останавливает подключение вручную после его использования.

Пример:

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl(...)
    .Build();
try
{
    await connection.StartAsync();
    // Do your stuff
    await connection.StopAsync();
}
finally
{
    await connection.DisposeAsync();
}
```

### <a name="common-improper-client-connection-usage"></a>Распространенное неправильное использование подключения клиента

#### <a name="azure-function-example"></a>Пример функции Azure 

Эта проблема часто возникает, когда кто-то устанавливает клиентское подключение SignalR в методе функции Azure вместо того, чтобы сделать его статическим членом класса функции. Вы можете ожидать, что установлено только одно клиентское подключение, но количество подключений клиентов постоянно увеличивается в метриках, которые находятся в разделе "Мониторинг" в портал Azure меню ресурсов. все эти подключения отменяются только после перезапуска функции Azure или службы Azure SignalR. Это связано с тем, что при **каждом** запросе функция Azure создает **одно** клиентское соединение, если подключение клиента в методе функции не будет прервано, клиент будет поддерживать подключения в службе Azure SignalR.

#### <a name="solution"></a>Решение

* Не забудьте закрыть клиентское подключение, если вы используете SignalR Clients в функции Azure или используете клиент SignalR в качестве одноэлементного.
* Вместо использования клиентов SignalR в функции Azure вы можете создавать клиенты SignalR в любом месте и использовать [привязки функций Azure для службы Azure SignalR](https://github.com/Azure/azure-functions-signalrservice-extension) для [согласования](https://github.com/Azure/azure-functions-signalrservice-extension/blob/dev/samples/simple-chat/csharp/FunctionApp/Functions.cs#L22) клиента с Azure SignalR. Кроме того, можно использовать привязку для [отправки сообщений](https://github.com/Azure/azure-functions-signalrservice-extension/blob/dev/samples/simple-chat/csharp/FunctionApp/Functions.cs#L40). Примеры для согласования клиентов и отправки сообщений можно найти [здесь](https://github.com/Azure/azure-functions-signalrservice-extension/tree/dev/samples). Дополнительные сведения можно найти [здесь](https://github.com/Azure/azure-functions-signalrservice-extension).
* При использовании клиентов SignalR в функции Azure может быть улучшена архитектура вашего сценария. Проверьте, правильно ли разрабатывается бессерверная архитектура. Вы можете обращаться к [бессерверным приложениям в режиме реального времени с привязками службы SignalR в функциях Azure](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.SignalRService).

<a name="server_connection_drop"></a>

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="server-connection-drops"></a>Падения подключений к серверу

Когда сервер приложений запускается, в фоновом режиме пакет SDK для Azure начинает инициировать подключения к удаленному серверу Azure SignalR. Как описано в статье [внутренние компоненты службы Azure SignalR](https://github.com/Azure/azure-signalr/blob/dev/docs/internal.md), Azure SignalR направляет входящие клиентские трафик на эти подключения к серверу. После удаления соединения с сервером все клиентские подключения, которые он обслуживает, будут закрыты.

Поскольку подключения между сервером приложений и службой SignalR являются постоянными подключениями, они могут испытывать проблемы с сетевым подключением. В пакете SDK для сервера мы **всегда повторно подключаем** стратегию к подключениям к серверу. Рекомендуется также рекомендовать пользователям добавлять логику непрерывного повторного подключения к клиентам с произвольным временем задержки, чтобы избежать массовых одновременных запросов к серверу.

На регулярной основе для службы Azure SignalR предусмотрены новые версии версий, а иногда установка исправлений или обновления операционной системы для всего Azure, а также иногда прерывание от наших зависимых служб. Это может привести к кратковременному прерыванию работы службы, но при условии, что клиент на стороне клиента имеет механизм отключения и повторного подключения, такое влияние минимально, как и на стороне клиента, вызванного отключением и повторному подключению.

В этом разделе описаны несколько возможностей, ведущих к удалению подключения к серверу, и приводятся некоторые рекомендации по определению основной причины.

### <a name="possible-errors-seen-from-the-server-side"></a>Возможные ошибки со стороны сервера

* `[Error]Connection "..." to the service was dropped`
* `The remote party closed the WebSocket connection without completing the close handshake`
* `Service timeout. 30.00ms elapsed without receiving a message from service.`

### <a name="root-cause"></a>Первопричина

Подключение к службе сервера закрыто **АСРС**(**A** **Zure игнал****R** **s** ервице).

Для времени ожидания проверки связи это может быть вызвано высокой нагрузкой ЦП или нехватка пула потоков на стороне сервера.

В ASP.NET SignalR была исправлена известная проблема в пакете SDK 1.6.0. Обновите пакет SDK до последней версии.

## <a name="thread-pool-starvation"></a>Нехватка ресурсов пула потоков

Если сервер не используется, это означает, что потоки не работают над обработкой сообщений. Все потоки не отвечают в определенном методе.

Обычно этот сценарий вызывается асинхронно для синхронизации или `Task.Result` / `Task.Wait()` в асинхронных методах.

См. ASP.NET Core рекомендации по [повышению производительности](/aspnet/core/performance/performance-best-practices#avoid-blocking-calls).

См. Дополнительные сведения о нехватке [пула потоков](https://docs.microsoft.com/archive/blogs/vancem/diagnosing-net-core-threadpool-starvation-with-perfview-why-my-service-is-not-saturating-all-cores-or-seems-to-stall).

### <a name="how-to-detect-thread-pool-starvation"></a>Как определить нехватку пула потоков

Проверьте число потоков. Если в это время нет пиковых нагрузок, сделайте следующее:
* Если вы используете службу приложений Azure, проверьте количество потоков в метриках. Проверьте `Max` агрегирование:
    
  :::image type="content" source="media/signalr-howto-troubleshoot-guide/metrics-thread-count.png" alt-text="Снимок экрана: панель &quot;максимальное число потоков&quot; в службе приложений Azure.":::

* Если вы используете платформа .NET Framework, вы можете найти [метрики](https://docs.microsoft.com/dotnet/framework/debug-trace-profile/performance-counters#lock-and-thread-performance-counters) в системном мониторе на виртуальной машине сервера.
* Если вы используете .NET Core в контейнере, см. раздел [получение диагностических сведений в контейнерах](https://docs.microsoft.com/dotnet/core/diagnostics/diagnostics-in-containers).

Можно также использовать код для обнаружения недостаточности пула потоков:

```csharp
public class ThreadPoolStarvationDetector : EventListener
{
    private const int EventIdForThreadPoolWorkerThreadAdjustmentAdjustment = 55;
    private const uint ReasonForStarvation = 6;

    private readonly ILogger<ThreadPoolStarvationDetector> _logger;

    public ThreadPoolStarvationDetector(ILogger<ThreadPoolStarvationDetector> logger)
    {
        _logger = logger;
    }

    protected override void OnEventSourceCreated(EventSource eventSource)
    {
        if (eventSource.Name == "Microsoft-Windows-DotNETRuntime")
        {
            EnableEvents(eventSource, EventLevel.Informational, EventKeywords.All);
        }
    }

    protected override void OnEventWritten(EventWrittenEventArgs eventData)
    {
        // See: https://docs.microsoft.com/en-us/dotnet/framework/performance/thread-pool-etw-events#threadpoolworkerthreadadjustmentadjustment
        if (eventData.EventId == EventIdForThreadPoolWorkerThreadAdjustmentAdjustment &&
            eventData.Payload[3] as uint? == ReasonForStarvation)
        {
            _logger.LogWarning("Thread pool starvation detected!");
        }
    }
}
```
    
Добавьте его в службу:
    
```csharp
service.AddSingleton<ThreadPoolStarvationDetector>();
```

Затем проверьте журнал, когда соединение с сервером отключается по истечении времени ожидания проверки связи.

### <a name="how-to-find-the-root-cause-of-thread-pool-starvation"></a>Как найти основную причину нехватка ресурсов пула потоков

Чтобы найти основную причину нехватка ресурсов пула потоков, выполните следующие действия.

* Выведите дамп памяти, а затем проанализируйте стек вызовов. Дополнительные сведения см. в разделе [Получение и анализ дампов памяти](https://devblogs.microsoft.com/dotnet/collecting-and-analyzing-memory-dumps/).
* Используйте [клрмд](https://github.com/microsoft/clrmd) для дампа памяти при обнаружении нехватки ресурсов пула потоков. Затем зайдите в журнал стека вызовов.

### <a name="troubleshooting-guide"></a>Руководство по устранению неполадок

1. Откройте журнал на стороне сервера приложений, чтобы проверить, не выполнялось ли что-либо непредвиденное.
2. Проверьте журнал событий на стороне сервера приложений, чтобы узнать, не перезапущен ли сервер приложений.
3. Создайте вопрос. Укажите интервал времени и отправьте имя ресурса по электронной почте.

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="tips"></a>Советы

<a name="view_request"></a>

* Как просмотреть исходящий запрос от клиента?
Возьмем ASP.NET Core один пример (ASP.NET похож):
    * Из браузера:

        Возьмем Chrome в качестве примера. можно использовать **F12** , чтобы открыть окно консоли, и перейти на вкладку **сеть** . Возможно, потребуется обновить страницу с помощью **клавиши F5** , чтобы захватить сеть с самого начала.

        :::image type="content" source="./media/signalr-howto-troubleshoot-guide/chrome-network.gif" alt-text="Сеть представления Chrome":::

    * Из клиента C#:

        Вы можете просматривать локальные веб-трафик с помощью [Fiddler](https://www.telerik.com/fiddler). Трафик WebSocket поддерживается с момента Fiddler 4,5.

        :::image type="content" source="./media/signalr-howto-troubleshoot-guide/fiddler-view-network-inline.png" alt-text="Fiddler просмотр сети" lightbox="./media/signalr-howto-troubleshoot-guide/fiddler-view-network.png":::

<a name="restart_connection"></a>

* Как перезапустить клиентское подключение?
    
    Ниже приведены [примеры кодов](https://github.com/Azure/azure-signalr/tree/dev/samples) , содержащих перезапуск логики подключения с использованием *всегда стратегии повторных попыток* :

    * [Клиент ASP.NET Core C#](https://github.com/Azure/azure-signalr/tree/dev/samples/ChatSample/ChatSample.CSharpClient/Program.cs#L64)

    * [ASP.NET Core клиент JavaScript](https://github.com/Azure/azure-signalr/blob/release/1.0.0-preview1/samples/ChatSample/wwwroot/index.html#L164)

    * [Клиент ASP.NET C#](https://github.com/Azure/azure-signalr/tree/dev/samples/AspNet.ChatSample/AspNet.ChatSample.CSharpClient/Program.cs#L78)

    * [Клиент ASP.NET JavaScript](https://github.com/Azure/azure-signalr/tree/dev/samples/AspNet.ChatSample/AspNet.ChatSample.JavaScriptClient/wwwroot/index.html#L71)

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство вы узнали, как решать распространенные проблемы. Вы также можете узнать больше об общих методах устранения неполадок. 

> [!div class="nextstepaction"]
> [Устранение проблем с подключением и доставкой сообщений](./signalr-howto-troubleshoot-method.md)
