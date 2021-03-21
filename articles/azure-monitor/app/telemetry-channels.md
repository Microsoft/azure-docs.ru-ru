---
title: Каналы телеметрии в Azure Application Insights | Документация Майкрософт
description: Как настроить каналы телеметрии в Azure Application Insights SDK для .NET и .NET Core.
ms.topic: conceptual
ms.date: 05/14/2019
ms.custom: devx-track-csharp
ms.reviewer: mbullwin
ms.openlocfilehash: a22a0d112671019d73eb4c9a3853462e4e9c8c75
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98937362"
---
# <a name="telemetry-channels-in-application-insights"></a>Каналы телеметрии в Application Insights

Каналы телеметрии являются неотъемлемой частью [пакетов SDK для Application Insights Azure](./app-insights-overview.md). Они управляют буферизацией и передачей данных телеметрии в службу Application Insights. Версии пакетов SDK для .NET и .NET Core имеют два встроенных канала телеметрии: `InMemoryChannel` и `ServerTelemetryChannel` . В этой статье подробно описывается каждый канал, в том числе Настройка поведения канала.

## <a name="what-are-telemetry-channels"></a>Что такое каналы телеметрии?

Каналы телеметрии отвечают за буферизацию элементов телеметрии и их отправку в службу Application Insights, где они хранятся для выполнения запросов и анализа. Канал телеметрии — это любой класс, реализующий [`Microsoft.ApplicationInsights.ITelemetryChannel`](/dotnet/api/microsoft.applicationinsights.channel.itelemetrychannel) интерфейс.

`Send(ITelemetry item)`Метод канала телеметрии вызывается после вызова всех инициализаторов телеметрии и обработчиков данных телеметрии. Таким образом, все элементы, отброшенные обработчиком данных телеметрии, не будут обращаться к каналу. `Send()` обычно не отправляет элементы в серверную части мгновенно. Как правило, он помещает их в память и отправляет их пакетами для эффективной передачи.

[Live Metrics Stream](live-stream.md) также имеет настраиваемый канал, который обеспечивает потоковую передачу телеметрии в реальном времени. Этот канал не зависит от обычного канала телеметрии, и этот документ не применяется к нему.

## <a name="built-in-telemetry-channels"></a>Встроенные каналы телеметрии

Application Insights .NET и .NET Core SDK поставляются с двумя встроенными каналами:

* `InMemoryChannel`: Упрощенный канал, который помещает элементы в память до тех пор, пока они не будут отправлены. Элементы помещаются в буфер памяти и записываются каждые 30 секунд или каждый раз, когда 500 элементов буферизуются. Этот канал предлагает минимальные гарантии надежности, так как не повторяет повторную отправку данных телеметрии после сбоя. Этот канал также не сохраняет элементы на диске, поэтому все неотправленные элементы теряются навсегда после завершения работы приложения (корректно или не работает). Этот канал реализует `Flush()` метод, который можно использовать для принудительного сброса всех элементов телеметрии в памяти в синхронном режиме. Этот канал хорошо подходит для кратковременно выполняемых приложений, в которых синхронный сброс является идеальным.

    Этот канал является частью более крупного пакета NuGet Microsoft. ApplicationInsights и является каналом по умолчанию, который пакет SDK использует при отсутствии других настроек.

* `ServerTelemetryChannel`— Более сложный канал с политиками повтора и возможностью хранения данных на локальном диске. Этот канал повторяет передачу данных телеметрии при возникновении временных ошибок. Этот канал также использует локальное дисковое хранилище для хранения элементов на диске во время простоя сети или больших объемов данных телеметрии. Из-за этих механизмов повтора и хранилища локального диска этот канал считается более надежным и рекомендуется для всех рабочих сценариев. Этот канал используется по умолчанию для [ASP.NET](./asp-net.md) и [ASP.NET Core](./asp-net-core.md) приложений, которые настроены в соответствии с официальной документацией. Этот канал оптимизирован для серверных сценариев с долго выполняющимися процессами. [`Flush()`](#which-channel-should-i-use)Метод, реализуемый этим каналом, не является синхронным.

    Этот канал поставляется в качестве пакета NuGet Microsoft. ApplicationInsights. WindowsServer. Телеметричаннел и запрашивается автоматически при использовании пакета NuGet Microsoft. ApplicationInsights. Web или Microsoft. ApplicationInsights. AspNetCore.

## <a name="configure-a-telemetry-channel"></a>Настройка канала телеметрии

Канал телеметрии настраивается с помощью настройки активной конфигурации телеметрии. Для приложений ASP.NET конфигурация предполагает установку экземпляра канала телеметрии `TelemetryConfiguration.Active` или изменение `ApplicationInsights.config` . Для ASP.NET Core приложений конфигурация включает добавление канала в контейнер внедрения зависимостей.

В следующих разделах приведены примеры настройки `StorageFolder` параметра для канала в различных типах приложений. `StorageFolder` — Это только один из настраиваемых параметров. Полный список параметров конфигурации см. в [разделе Параметры](#configurable-settings-in-channels) далее в этой статье.

### <a name="configuration-by-using-applicationinsightsconfig-for-aspnet-applications"></a>Настройка с помощью ApplicationInsights.config для приложений ASP.NET

В следующем разделе из [ApplicationInsights.config](configuration-with-applicationinsights-config.md) показан `ServerTelemetryChannel` канал, для которого настроено `StorageFolder` Задание настраиваемого расположения.

```xml
    <TelemetrySinks>
        <Add Name="default">
            <TelemetryProcessors>
                <!-- Telemetry processors omitted for brevity  -->
            </TelemetryProcessors>
            <TelemetryChannel Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.ServerTelemetryChannel, Microsoft.AI.ServerTelemetryChannel">
                <StorageFolder>d:\temp\applicationinsights</StorageFolder>
            </TelemetryChannel>
        </Add>
    </TelemetrySinks>
```

### <a name="configuration-in-code-for-aspnet-applications"></a>Настройка в коде для приложений ASP.NET

Следующий код настраивает экземпляр "Сервертелеметричаннел" с `StorageFolder` установленным пользовательским расположением. Добавьте этот код в начало приложения, как правило, в `Application_Start()` методе в Global. aspx. cs.

```csharp
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;
protected void Application_Start()
{
    var serverTelemetryChannel = new ServerTelemetryChannel();
    serverTelemetryChannel.StorageFolder = @"d:\temp\applicationinsights";
    serverTelemetryChannel.Initialize(TelemetryConfiguration.Active);
    TelemetryConfiguration.Active.TelemetryChannel = serverTelemetryChannel;
}
```

### <a name="configuration-in-code-for-aspnet-core-applications"></a>Настройка в коде для приложений ASP.NET Core

Измените `ConfigureServices` метод `Startup.cs` класса, как показано ниже:

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;

public void ConfigureServices(IServiceCollection services)
{
    // This sets up ServerTelemetryChannel with StorageFolder set to a custom location.
    services.AddSingleton(typeof(ITelemetryChannel), new ServerTelemetryChannel() {StorageFolder = @"d:\temp\applicationinsights" });

    services.AddApplicationInsightsTelemetry();
}

```

> [!IMPORTANT]
> Настройка канала с помощью не `TelemetryConfiguration.Active` рекомендуется для приложений ASP.NET Core.

### <a name="configuration-in-code-for-netnet-core-console-applications"></a>Настройка в коде для консольных приложений .NET и .NET Core

Для консольных приложений код одинаков для .NET и .NET Core:

```csharp
var serverTelemetryChannel = new ServerTelemetryChannel();
serverTelemetryChannel.StorageFolder = @"d:\temp\applicationinsights";
serverTelemetryChannel.Initialize(TelemetryConfiguration.Active);
TelemetryConfiguration.Active.TelemetryChannel = serverTelemetryChannel;
```

## <a name="operational-details-of-servertelemetrychannel"></a>Операционные сведения о Сервертелеметричаннел

`ServerTelemetryChannel` хранит поступающие элементы в буфере, находящиеся в памяти. Элементы сериализуются, сжимаются и сохраняются в `Transmission` экземпляре каждые 30 секунд, или когда 500 элементов были помещены в буфер. Один `Transmission` экземпляр содержит до 500 элементов и представляет пакет телеметрии, который отправляется через один вызов HTTPS в службу Application Insights.

По умолчанию `Transmission` параллельно может отправляться не более 10 экземпляров. Если данные телеметрии поступают с более высокой скоростью или скорость сети или серверной части Application Insights медленной, то `Transmission` экземпляры хранятся в памяти. Емкость этого буфера в памяти по умолчанию `Transmission` составляет 5 МБ. При превышении объема памяти `Transmission` экземпляры хранятся на локальном диске до ограничения в 50 МБ. `Transmission` экземпляры хранятся на локальном диске при возникновении проблем с сетью. Сбои приложения сохраняются только в тех элементах, которые хранятся на локальном диске. Они отправляются при каждом запуске приложения.

## <a name="configurable-settings-in-channels"></a>Настраиваемые параметры в каналах

Полный список настраиваемых параметров для каждого канала см. в следующих статьях:

* [InMemoryChannel](https://github.com/microsoft/ApplicationInsights-dotnet/blob/develop/BASE/src/Microsoft.ApplicationInsights/Channel/InMemoryChannel.cs)

* [сервертелеметричаннел](https://github.com/microsoft/ApplicationInsights-dotnet/blob/develop/BASE/src/ServerTelemetryChannel/ServerTelemetryChannel.cs)

Ниже приведены наиболее часто используемые параметры для `ServerTelemetryChannel` .

1. `MaxTransmissionBufferCapacity`: Максимальный объем памяти (в байтах), используемый каналом для передачи данных в буфер памяти. При достижении этой емкости новые элементы хранятся непосредственно на локальном диске. Значение по умолчанию — 5 МБ. Если задать более высокое значение, это приводит к меньшему использованию дискового пространства, но помните, что элементы в памяти будут потеряны при сбое приложения.

1. `MaxTransmissionSenderCapacity`: Максимальное число `Transmission` экземпляров, которые будут отправлены в Application Insights в одно и то же время. Значение по умолчанию — 10. Для этого параметра можно задать большее значение, что рекомендуется при создании огромного объема данных телеметрии. Большое количество томов обычно происходит во время нагрузочного тестирования или при отключении выборки.

1. `StorageFolder`— Папка, используемая каналом для хранения элементов на диске по мере необходимости. В Windows используется либо% LOCALAPPDATA%, либо% TEMP%, если никакой другой путь не указан явным образом. В средах, отличных от Windows, необходимо указать допустимое расположение или данные телеметрии, которые не будут храниться на локальном диске.

## <a name="which-channel-should-i-use"></a>Какой канал следует использовать?

`ServerTelemetryChannel` рекомендуется использовать для большинства рабочих сценариев, использующих длительно работающие приложения. `Flush()`Метод, реализованный, `ServerTelemetryChannel` не является синхронным, и он также не гарантирует отправку всех ожидающих элементов из памяти или с диска. Если вы используете этот канал в сценариях, где приложение собирается завершить работу, рекомендуется ввести некоторую задержку после вызова `Flush()` . Точный объем задержки, который может потребоваться, не является прогнозируемым. Это зависит от таких факторов, как количество элементов или `Transmission` экземпляров в памяти, количество объектов на диске, количество передаваемых данных в серверную часть, а также сведения о том, входит ли канал в середину экспоненциальных вариантов обратной передачи.

Если необходимо выполнить синхронную очистку, рекомендуется использовать `InMemoryChannel` .

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="does-the-application-insights-channel-guarantee-telemetry-delivery-if-not-what-are-the-scenarios-in-which-telemetry-can-be-lost"></a>Гарантирует ли Application Insights канал доставку данных телеметрии? Если нет, Каковы сценарии, в которых можно потерять данные телеметрии?

Короткий ответ заключается в том, что ни один из встроенных каналов не обеспечивает гарантию доставки данных телеметрии в серверную часть. `ServerTelemetryChannel` более сложная по сравнению с `InMemoryChannel` для обеспечения надежной доставки, но она также делает попытку отправки данных телеметрии более эффективной. Данные телеметрии по-прежнему могут быть потеряны в нескольких ситуациях, включая следующие распространенные сценарии:

1. При сбое приложения элементы в памяти теряются.

1. Данные телеметрии теряются в течение продолжительных периодов сетевых проблем. Данные телеметрии сохраняются на локальном диске во время простоя сети или при возникновении проблем с серверной части Application Insights. Однако элементы, возраст которых превышает 48 часов, отбрасываются.

1. Места хранения телеметрии в Windows по умолчанию:% LOCALAPPDATA% или% TEMP%. Эти расположения обычно являются локальными для компьютера. Если приложение переносится физически из одного расположения в другое, все данные телеметрии, хранящиеся в исходном расположении, теряются.

1. В веб-приложениях Windows в качестве места хранения диска по умолчанию используется Д:\локал\локалаппдата. Это расположение не сохраняется. Она уничтожается при перезапуске, масштабировании и других подобных операциях, что приводит к утрате всех данных телеметрии, хранящихся в ней. Можно переопределить значение по умолчанию и указать хранилище в сохраненном расположении, например Д:\хоме. Тем не менее, такие сохраненные расположения обслуживаются удаленным хранилищем, поэтому они могут выполняться слишком долго.

### <a name="does-servertelemetrychannel-work-on-systems-other-than-windows"></a>Работает ли Сервертелеметричаннел на системах, отличных от Windows?

Хотя имя пакета и пространства имен содержит "WindowsServer", этот канал поддерживается не в Windows, за исключением следующего. В системах, отличных от Windows, канал не создает папку локального хранилища по умолчанию. Необходимо создать папку локального хранилища и настроить канал для ее использования. После настройки локального хранилища канал работает так же, как и во всех системах.

> [!NOTE]
> После выпуска 2.15.0-beta3 и более поздних версий локальное хранилище автоматически создается для Linux, Mac и Windows. Для систем, отличных от Windows, пакет SDK автоматически создаст папку локального хранилища на основе следующей логики:
> - `${TMPDIR}` — Если `${TMPDIR}` задано значение "переменная среды", используется это расположение.
> - `/var/tmp` — Если предыдущее расположение не существует, будет предпринята попытка `/var/tmp` .
> - `/tmp` — Если и предыдущее расположение не существуют, попробуйте `tmp` . 
> - Если ни одно из этих расположений не существует, локальное хранилище не создается, и по-прежнему требуется настройка вручную. [Для получения полной информации о реализации](https://github.com/microsoft/ApplicationInsights-dotnet/pull/1860).

### <a name="does-the-sdk-create-temporary-local-storage-is-the-data-encrypted-at-storage"></a>Создает ли пакет SDK временное локальное хранилище? Зашифрованы ли данные в хранилище?

Пакет SDK хранит элементы телеметрии в локальном хранилище во время сетевых проблем или при регулировании. Эти данные не шифруются локально.

Для систем Windows пакет SDK автоматически создает временную локальную папку в каталоге% TEMP% или% LOCALAPPDATA% и разрешает доступ только администраторам и текущему пользователю.

Для систем, отличных от Windows, локальное хранилище не создается автоматически пакетом SDK, поэтому данные по умолчанию не хранятся локально.

> [!NOTE]
> После выпуска 2.15.0-beta3 и более поздних версий локальное хранилище автоматически создается для Linux, Mac и Windows. 

 Вы можете создать каталог хранилища самостоятельно и настроить его для использования в канале. В этом случае вы несете ответственность за обеспечение безопасности каталога.
Узнайте больше о [защите и конфиденциальности данных](data-retention-privacy.md#does-the-sdk-create-temporary-local-storage).

## <a name="open-source-sdk"></a>Пакет SDK с открытым исходным кодом
Как и каждый пакет SDK для Application Insights, каналы являются открытым кодом. Прочтите и отучите код или сообщите о проблемах в [официальном репозитории GitHub](https://github.com/Microsoft/ApplicationInsights-dotnet).

## <a name="next-steps"></a>Дальнейшие действия

* [Выборка](./sampling.md)
* [Устранение неполадок пакета SDK](./asp-net-troubleshoot-no-data.md)

