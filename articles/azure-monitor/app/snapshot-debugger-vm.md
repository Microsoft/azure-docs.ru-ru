---
title: Включение Snapshot Debugger для приложений .NET в Service Fabric, облачной службе и виртуальных машинах Azure | Документация Майкрософт
description: Включение Snapshot Debugger для приложений .NET в Service Fabric, облачной службе и виртуальных машинах Azure
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 03/07/2019
ms.reviewer: mbullwin
ms.openlocfilehash: 4bccc2922cf20262149ef54fbe2a1a821d9551ab
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97673507"
---
# <a name="enable-snapshot-debugger-for-net-apps-in-azure-service-fabric-cloud-service-and-virtual-machines"></a>Включение Snapshot Debugger для приложений .NET в Service Fabric, облачной службе и виртуальных машинах Azure

Если приложение ASP.NET или ASP.NET Core работает в службе приложений Azure, настоятельно рекомендуется [включить snapshot Debugger на странице портала Application Insights](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json). Однако, если для приложения требуется настроенная конфигурация Snapshot Debugger или предварительная версия .NET Core, следует выполнить эту инструкцию, ***а также инструкции*** по [включению на странице портала Application Insights](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json).

Если приложение выполняется в Service Fabric Azure, облачной службе, виртуальных машинах или локальных компьютерах, следует использовать следующие инструкции. 

## <a name="configure-snapshot-collection-for-aspnet-applications"></a>Настройка сбора моментальных снимков для приложений

1. [Включите Application Insights в веб-приложении](./asp-net.md), если вы еще не сделали это.

2. Добавьте в приложение пакет NuGet [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector).

3. При необходимости настроил конфигурацию Snapshot Debugger, добавленную в [ApplicationInsights.config](./configuration-with-applicationinsights-config.md). Конфигурация Snapshot Debugger по умолчанию в основном пуста, и все параметры являются необязательными. Ниже приведен пример, в котором показана конфигурация, эквивалентная конфигурации по умолчанию.

    ```xml
    <TelemetryProcessors>
        <Add Type="Microsoft.ApplicationInsights.SnapshotCollector.SnapshotCollectorTelemetryProcessor, Microsoft.ApplicationInsights.SnapshotCollector">
        <!-- The default is true, but you can disable Snapshot Debugging by setting it to false -->
        <IsEnabled>true</IsEnabled>
        <!-- Snapshot Debugging is usually disabled in developer mode, but you can enable it by setting this to true. -->
        <!-- DeveloperMode is a property on the active TelemetryChannel. -->
        <IsEnabledInDeveloperMode>false</IsEnabledInDeveloperMode>
        <!-- How many times we need to see an exception before we ask for snapshots. -->
        <ThresholdForSnapshotting>1</ThresholdForSnapshotting>
        <!-- The maximum number of examples we create for a single problem. -->
        <MaximumSnapshotsRequired>3</MaximumSnapshotsRequired>
        <!-- The maximum number of problems that we can be tracking at any time. -->
        <MaximumCollectionPlanSize>50</MaximumCollectionPlanSize>
        <!-- How often we reconnect to the stamp. The default value is 15 minutes.-->
        <ReconnectInterval>00:15:00</ReconnectInterval>
        <!-- How often to reset problem counters. -->
        <ProblemCounterResetInterval>1.00:00:00</ProblemCounterResetInterval>
        <!-- The maximum number of snapshots allowed in ten minutes.The default value is 1. -->
        <SnapshotsPerTenMinutesLimit>3</SnapshotsPerTenMinutesLimit>
        <!-- The maximum number of snapshots allowed per day. -->
        <SnapshotsPerDayLimit>30</SnapshotsPerDayLimit>
        <!-- Whether or not to collect snapshot in low IO priority thread. The default value is true. -->
        <SnapshotInLowPriorityThread>true</SnapshotInLowPriorityThread>
        <!-- Agree to send anonymous data to Microsoft to make this product better. -->
        <ProvideAnonymousTelemetry>true</ProvideAnonymousTelemetry>
        <!-- The limit on the number of failed requests to request snapshots before the telemetry processor is disabled. -->
        <FailedRequestLimit>3</FailedRequestLimit>
        </Add>
    </TelemetryProcessors>
    ```

4. Моментальные снимки собираются только для исключений, которые передаются в Application Insights. В некоторых случаях (например, в более старых версиях платформы .NET) может потребоваться [настроить сбор исключений](./asp-net-exceptions.md#exceptions), чтобы просматривать исключения с помощью моментальных снимков, которые отображаются на портале.


## <a name="configure-snapshot-collection-for-applications-using-aspnet-core-20-or-above"></a>Настройка сбора моментальных снимков для приложений, использующих ASP.NET Core 2,0 или более поздней версии

1. [Включите Application Insights в веб-приложении ASP.NET Core](./asp-net-core.md), если вы еще не сделали это.

    > [!NOTE]
    > Убедитесь, что приложение ссылается на версию 2.1.1 или более позднюю версию пакета Microsoft.ApplicationInsights.AspNetCore.

2. Добавьте в приложение пакет NuGet [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector).

3. Чтобы добавить и настроить обработчик телеметрии сборщика моментальных снимков, измените класс `Startup` своего приложения.
    1. Если используется пакет NuGet [Microsoft. ApplicationInsights. SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) версии 1.3.5 или выше, добавьте следующие операторы using в `Startup.cs` .

       ```csharp
            using Microsoft.ApplicationInsights.SnapshotCollector;
       ```

       Добавьте следующий элемент в конец метода ConfigureServices в `Startup` классе в `Startup.cs` .

       ```csharp
            services.AddSnapshotCollector((configuration) => Configuration.Bind(nameof(SnapshotCollectorConfiguration), configuration));
       ```
    2. Если используется пакет NuGet [Microsoft. ApplicationInsights. SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) версии 1.3.4 или ниже, добавьте следующие операторы using в `Startup.cs` .

       ```csharp
       using Microsoft.ApplicationInsights.SnapshotCollector;
       using Microsoft.Extensions.Options;
       using Microsoft.ApplicationInsights.AspNetCore;
       using Microsoft.ApplicationInsights.Extensibility;
       ```

       Добавьте указанный ниже класс `SnapshotCollectorTelemetryProcessorFactory` в класс `Startup`.

       ```csharp
       class Startup
       {
           private class SnapshotCollectorTelemetryProcessorFactory : ITelemetryProcessorFactory
           {
               private readonly IServiceProvider _serviceProvider;

               public SnapshotCollectorTelemetryProcessorFactory(IServiceProvider serviceProvider) =>
                   _serviceProvider = serviceProvider;

               public ITelemetryProcessor Create(ITelemetryProcessor next)
               {
                   var snapshotConfigurationOptions = _serviceProvider.GetService<IOptions<SnapshotCollectorConfiguration>>();
                   return new SnapshotCollectorTelemetryProcessor(next, configuration: snapshotConfigurationOptions.Value);
               }
           }
           ...
        ```
        Добавьте службы `SnapshotCollectorConfiguration` и `SnapshotCollectorTelemetryProcessorFactory` в конвейер запуска:

        ```csharp
           // This method gets called by the runtime. Use this method to add services to the container.
           public void ConfigureServices(IServiceCollection services)
           {
               // Configure SnapshotCollector from application settings
               services.Configure<SnapshotCollectorConfiguration>(Configuration.GetSection(nameof(SnapshotCollectorConfiguration)));

               // Add SnapshotCollector telemetry processor.
               services.AddSingleton<ITelemetryProcessorFactory>(sp => new SnapshotCollectorTelemetryProcessorFactory(sp));

               // TODO: Add other services your application needs here.
           }
       }
       ```

4. При необходимости настроили конфигурацию Snapshot Debugger, добавив раздел Снапшотколлекторконфигуратион в appsettings.js. Все параметры в конфигурации Snapshot Debugger являются необязательными. Ниже приведен пример, в котором показана конфигурация, эквивалентная конфигурации по умолчанию.

   ```json
   {
     "SnapshotCollectorConfiguration": {
       "IsEnabledInDeveloperMode": false,
       "ThresholdForSnapshotting": 1,
       "MaximumSnapshotsRequired": 3,
       "MaximumCollectionPlanSize": 50,
       "ReconnectInterval": "00:15:00",
       "ProblemCounterResetInterval":"1.00:00:00",
       "SnapshotsPerTenMinutesLimit": 1,
       "SnapshotsPerDayLimit": 30,
       "SnapshotInLowPriorityThread": true,
       "ProvideAnonymousTelemetry": true,
       "FailedRequestLimit": 3
     }
   }
   ```

## <a name="configure-snapshot-collection-for-other-net-applications"></a>Настройка сбора моментальных снимков для других приложений .NET

1. Если приложение все еще не инструментировано с помощью Application Insights, начните с [включения Application Insights и установки ключа инструментирования](./windows-desktop.md).

2. Добавьте в приложение пакет NuGet [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector).

3. Моментальные снимки собираются только для исключений, которые передаются в Application Insights. Для их передачи может потребоваться изменить код. Код обработки исключений зависит от структуры приложения. Пример приведен ниже:
    ```csharp
   TelemetryClient _telemetryClient = new TelemetryClient();

   void ExampleRequest()
   {
        try
        {
            // TODO: Handle the request.
        }
        catch (Exception ex)
        {
            // Report the exception to Application Insights.
            _telemetryClient.TrackException(ex);

            // TODO: Rethrow the exception if desired.
        }
   }
    ```

## <a name="next-steps"></a>Дальнейшие действия

- Создание трафика для приложения, которое может вызвать исключение. Затем подождите 10 – 15 минут, чтобы моментальные снимки отправлялись на экземпляр Application Insights.
- См. раздел [моментальные снимки](snapshot-debugger.md?toc=/azure/azure-monitor/toc.json#view-snapshots-in-the-portal) в портал Azure.
- Справку по устранению неполадок Snapshot Debugger см. в разделе [snapshot Debugger устранение неполадок](snapshot-debugger-troubleshoot.md?toc=/azure/azure-monitor/toc.json).

