---
title: Рекомендации по устранению неполадок службы Azure SignalR
description: Узнайте, как устранять проблемы с подключением и доставкой сообщений
author: yjin81
ms.service: signalr
ms.topic: conceptual
ms.date: 11/17/2020
ms.author: yajin1
ms.openlocfilehash: 2e22777b747ae24c3e643cbd43bfdb0604d453a2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97707662"
---
# <a name="how-to-troubleshoot-connectivity-and-message-delivery-issues"></a>Устранение проблем с подключением и доставкой сообщений

В этом руководстве представлено несколько способов самостоятельной диагностики, чтобы найти основную причину ошибки или устранить ее. Результат самодиагностики также полезен при отправке отчета нам для дальнейшего изучения.

Во-первых, необходимо проверить портал Azure, для которой [сервицемоде](./concept-service-mode.md) — служба SignalR Azure (также известная как **АСРС**), настроенная для.

:::image type="content" source="./media/signalr-howto-troubleshoot-method/service-mode.png" alt-text="сервицемоде":::

* Сведения о режиме см. `Default` в разделе [Устранение неполадок режима по умолчанию](#default_mode_tsg)

* Сведения о режиме см. `Serverless` в разделе [Устранение неполадок в режиме сервера](#serverless_mode_tsg)

* Сведения о режиме см. `Classic` в разделе [Устранение неполадок в классическом режиме](#classic_mode_tsg) .

<a name="default_mode_tsg"></a>

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="default-mode-troubleshooting"></a>Устранение неполадок режима по умолчанию

Если **АСРС** работает в режиме *по умолчанию* , то существуют **три** роли: *клиент*, *сервер* и *Служба*:

* *Клиент*: *клиент* означает, что клиенты подключены к **АСРС**. В этом руководстве клиенты, подключающиеся к постоянным подключениям и **АСРС** , называются *клиентскими подключениями* .

* *Сервер*. *сервер* означает сервер, который обслуживает согласование клиента и размещает `Hub` логику SignalR. В этом руководстве постоянные подключения между *сервером* и **АСРС** называются *подключениями к серверу* .

* *Service*: *Service* — это краткое имя для **АСРС** в этом руководстве.

Подробные сведения о всей архитектуре и рабочем процессе см. в статье [внутренние компоненты службы Azure SignalR](https://github.com/Azure/azure-signalr/blob/dev/docs/internal.md) .

Существует несколько способов, которые могут помочь в устранении проблемы. 

* Если эта ошибка происходит сразу же или в режиме воспроизведения, Прямая переадресация — это просмотр непрерывного трафика. 

* Если трудно воспроизвести проблемы, трассировка и журналы могут помочь.

### <a name="how-to-view-the-traffic-and-narrow-down-the-issue"></a>Просмотр трафика и ограничение проблемы

Захват трафика в ходе выполнения является самым прямым способом для устранения проблемы. [Трассировку сети](/aspnet/core/signalr/diagnostics#network-traces) можно записать с помощью описанных ниже параметров.

* [Получение трассировки сети с помощью Fiddler](/aspnet/core/signalr/diagnostics#network-traces)

* [Получение трассировки сети с помощью tcpdump](/aspnet/core/signalr/diagnostics#collect-a-network-trace-with-tcpdump-macos-and-linux-only)

* [Получение трассировки сети в браузере](/aspnet/core/signalr/diagnostics#collect-a-network-trace-in-the-browser)

<a name="view_traffic_client"></a>

#### <a name="client-requests"></a>Клиентские запросы

Чтобы получить постоянное подключение SignalR, сначала `/negotiate` на сервере размещенного приложения, а затем перенаправляется в службу Azure SignalR, а затем устанавливается реальное постоянное подключение к службе Azure SignalR. Подробные инструкции см. в разделе [внутренние компоненты службы Azure SignalR](https://github.com/Azure/azure-signalr/blob/dev/docs/internal.md) .

С пропускной трассировкой сети на стороне клиента проверьте, какой запрос завершается ошибкой с кодом состояния и ответом, и найдите решения в разделе [руководство по устранению неполадок](./signalr-howto-troubleshoot-guide.md).

#### <a name="server-requests"></a>Запросы сервера

*Сервер* SignalR поддерживает *Подключение сервера* между *сервером* и *службой*. При запуске сервер приложений запускает подключение **WebSocket** к службе Azure SignalR. Все клиентские трафик направляются через службу Azure SignalR в эти соединения с *сервером*, а затем отправляются в `Hub` . При удалении *соединения с сервером* будут затронуты клиенты, направляются на это *Подключение к серверу* . В пакете SDK Azure SignalR предусмотрена логика "всегда повторяется", чтобы повторно подключить *Подключение к серверу* с максимальной задержкой в 1 минуту, чтобы снизить влияние.

*Соединение с сервером* может быть удалено из-за нестабильной работы сети или регулярного обслуживания службы Azure SignalR или обновления или обслуживания размещенного сервера приложений. При условии, что на стороне клиента есть механизм отключения и повторного подключения, влияние будет минимально, как и на стороне клиента, что вызвало отключение-повторное подключение.

Просмотрите трассировку сети на стороне сервера, чтобы узнать код состояния и сведения об ошибке, почему *Подключение к серверу* отклоняется или отклонено *службой*, и найдите основную причину в разделе [руководство по устранению неполадок](./signalr-howto-troubleshoot-guide.md).

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

### <a name="how-to-add-logs"></a>Добавление журналов

Журналы могут быть полезны для диагностики проблем и отслеживания состояния выполнения.

<a name="add_logs_client"></a>

#### <a name="how-to-enable-client-side-log"></a>Как включить журнал на стороне клиента

Процесс ведения журнала на стороне клиента точно такой же, как и при использовании самостоятельно размещенного SignalR.

##### <a name="enable-client-side-logging-for-aspnet-core-signalr"></a>Включить ведение журнала на стороне клиента для `ASP.NET Core SignalR`

* [Ведение журнала клиента JavaScript](/aspnet/core/signalr/diagnostics#javascript-client-logging)

* [Ведение журнала клиента .NET](/aspnet/core/signalr/diagnostics#net-client-logging)


##### <a name="enable-client-side-logging-for-aspnet-signalr"></a>Включить ведение журнала на стороне клиента для `ASP.NET SignalR`

* [Клиент .NET](/aspnet/signalr/overview/testing-and-debugging/enabling-signalr-tracing#enabling-tracing-in-the-net-client-windows-desktop-apps)

* [Включение трассировки в клиентах Windows Phone 8](/aspnet/signalr/overview/testing-and-debugging/enabling-signalr-tracing#enabling-tracing-in-windows-phone-8-clients)

* [Включение трассировки в клиенте JavaScript](/aspnet/signalr/overview/testing-and-debugging/enabling-signalr-tracing#enabling-tracing-in-the-javascript-client)

<a name="add_logs_server"></a>

#### <a name="how-to-enable-server-side-log"></a>Как включить журнал на стороне сервера

##### <a name="enable-server-side-logging-for-aspnet-core-signalr"></a>Включить ведение журнала на стороне сервера для `ASP.NET Core SignalR`

Ведение журнала на стороне сервера для `ASP.NET Core SignalR` интеграции с `ILogger` [журналом](/aspnet/core/fundamentals/logging/?tabs=aspnetcore2x&view=aspnetcore-2.1) на основе, предоставленным в `ASP.NET Core` платформе. Вы можете включить ведение журнала на стороне сервера с помощью `ConfigureLogging` , выбрав пример использования следующим образом:

```cs
.ConfigureLogging((hostingContext, logging) =>
        {
            logging.AddConsole();
            logging.AddDebug();
        })
```

Категории средств ведения журнала для Azure SignalR всегда начинаются с `Microsoft.Azure.SignalR` . Чтобы включить подробные журналы из Azure SignalR, настройте предыдущие префиксы для `Information` уровня **appsettings.jsв** файле, как показано ниже:

```JSON
{
    "Logging": {
        "LogLevel": {
            ...
            "Microsoft.Azure.SignalR": "Information",
            ...
        }
    }
}
```

Проверьте, записаны ли необычные журналы предупреждений или ошибок. 


##### <a name="enable-server-side-traces-for-aspnet-signalr"></a>Включить трассировки на стороне сервера для `ASP.NET SignalR`

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

Проверьте, записаны ли необычные журналы предупреждений или ошибок. 


#### <a name="how-to-enable-logs-inside-azure-signalr-service"></a>Как включить журналы в службе SignalR Azure

Вы также можете [включить журналы диагностики](./signalr-howto-diagnostic-logs.md) для службы Azure SignalR. Эти журналы содержат подробные сведения о каждом подключении, подключенном к службе Azure SignalR.

<a name="serverless_mode_tsg"></a>

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="serverless-mode-troubleshooting"></a>Устранение неполадок безсерверного режима

Если **АСРС** работает в *бессерверном* режиме, только **ASP.NET Core signalr** поддерживает `Serverless` режим, а **ASP.NET SignalR** не поддерживает  этот режим.

Чтобы диагностировать проблемы с подключением в `Serverless` режиме, наиболее прямым способом является [Просмотр трафика на стороне клиента](#view_traffic_client). Также могут быть полезны [журналы на стороне клиента](#add_logs_client) и [журналы на стороне службы](#add_logs_server) .

<a name="classic_mode_tsg"></a>

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="classic-mode-troubleshooting"></a>Устранение неполадок в классическом режиме

`Classic` режим устарел и не рекомендуется использовать. В этом режиме служба Azure SignalR использует *подключения* подключенного сервера, чтобы определить, находится ли текущая служба в `default` режиме или `serverless` режиме. Это может привести к некоторым проблемам с подключением клиентов, так как при внезапном удалении подключения к подключенному *серверу*, например из-за нестабильной работы сети, Azure SignalR считает, что теперь он переключен на `serverless` режим, а клиенты, подключенные в течение этого периода, никогда не будут маршрутизироваться на размещенный сервер приложений. Включите [журналы на стороне службы](#add_logs_server) и проверьте, зарегистрированы ли клиенты так, как `ServerlessModeEntered` если бы у вас был размещен сервер приложений, но некоторые клиенты никогда не достигли серверного приложения. Если таковые имеются, [прервать эти клиентские подключения](https://github.com/Azure/azure-signalr/blob/dev/docs/rest-api.md#API) и позволить клиентам перезапускать помощь.

Устранение неполадок с `classic` подключением и доставкой сообщений аналогичны проблемам, связанным с [режимами по умолчанию](#default_mode_tsg).

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="service-health"></a>Работоспособность службы

Вы можете проверить работоспособность службы в API работоспособности.

* Запрос: GET `https://{instance_name}.service.signalr.net/api/v1/health`

* Код состояния отклика:
  * 200: исправен.
  * 503: служба неработоспособна. Вы можете:
    * Подождите несколько минут для автовосстановления.
    * Проверьте, что IP-адрес совпадает с IP на портале.
    * Или перезапустите экземпляр.
    * Если все приведенные выше параметры не работают, свяжитесь с нами, добавив новый запрос в службу поддержки в портал Azure.

Дополнительные сведения об [аварийном восстановлении](./signalr-concept-disaster-recovery.md).

[Возникли проблемы или отзывы об устранении неполадок? Сообщите нам об этом.](https://aka.ms/asrs/survey/troubleshooting)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство вы узнали, как устранять проблемы с подключением и доставкой сообщений. Вы также можете научиться решать распространенные проблемы. 

> [!div class="nextstepaction"]
> [Руководство по устранению неполадок](./signalr-howto-troubleshoot-guide.md)