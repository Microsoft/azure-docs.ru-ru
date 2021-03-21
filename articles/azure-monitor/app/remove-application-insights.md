---
title: Удаление Application Insights в Visual Studio — Azure Monitor
description: Удаление Application Insights пакета SDK для ASP.NET и ASP.NET Core в Visual Studio.
ms.topic: conceptual
ms.date: 04/06/2020
ms.openlocfilehash: 1d70413fa6a47e2d41693db6eb705f31b2a2b1b2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101704316"
---
# <a name="how-to-remove-application-insights-in-visual-studio"></a>Удаление Application Insights в Visual Studio

В этой статье показано, как удалить пакет SDK для ASP.NET и ASP.NET Core Application Insights в Visual Studio.

Чтобы удалить Application Insights, необходимо удалить пакеты NuGet и ссылки из API в приложении. Вы можете удалить пакеты NuGet с помощью консоли Управление пакетами или управлять решением NuGet в Visual Studio. В следующих разделах мы рассмотрим два способа удаления пакетов NuGet и автоматически добавленных в проект. Обязательно убедитесь, что файлы добавлены и области в собственном коде, в котором были сделаны вызовы API.

## <a name="uninstall-using-the-package-management-console"></a>Удаление с помощью консоли Управление пакетами

# <a name="net"></a>[.NET](#tab/net)

1. Чтобы открыть консоль Управление пакетами, в верхнем меню выберите Сервис > диспетчер пакетов NuGet > консоль диспетчера пакетов.
     
    ![В верхнем меню выберите инструменты > диспетчер пакетов NuGet > консоль диспетчера пакетов.](./media/remove-application-insights/package-manager.png)

    > [!NOTE]
    > Если включена коллекция трассировки, сначала необходимо удалить Microsoft. ApplicationInsights. TraceListener. Введите `Uninstall-package Microsoft.ApplicationInsights.TraceListener` , а затем выполните следующие действия, чтобы удалить Microsoft. ApplicationInsights. Web.

1. Введите следующую команду: `Uninstall-Package Microsoft.ApplicationInsights.Web -RemoveDependencies`

    После ввода команды пакет Application Insights и все его зависимости будут удалены из проекта.
    
    ![Введите команду в консоли](./media/remove-application-insights/package-management-console.png)

# <a name="net-core"></a>[.NET Core](#tab/netcore)

1. Чтобы открыть консоль Управление пакетами, в верхнем меню выберите Сервис > диспетчер пакетов NuGet > консоль диспетчера пакетов.

    ![В верхнем меню выберите инструменты > диспетчер пакетов NuGet > консоль диспетчера пакетов.](./media/remove-application-insights/package-manager.png)

1. Введите следующую команду: ` Uninstall-Package Microsoft.ApplicationInsights.AspNetCore -RemoveDependencies`

    После ввода команды пакет Application Insights и все его зависимости будут удалены из проекта.

---

## <a name="uninstall-using-the-visual-studio-nuget-ui"></a>Удаление с помощью пользовательского интерфейса NuGet Visual Studio

# <a name="net"></a>[.NET](#tab/net)

1. В *Обозреватель решений* справа щелкните правой кнопкой мыши **решение** и выберите пункт **Управление пакетами NuGet для решения**.

    После этого появится экран, позволяющий изменить все пакеты NuGet, которые являются частью проекта.
    
     ![Щелкните правой кнопкой мыши решение в обозреватель решений, а затем выберите Управление пакетами NuGet для решения.](./media/remove-application-insights/manage-nuget-framework.png)

    > [!NOTE]
    > Если включена коллекция трассировки, сначала необходимо удалить Microsoft. ApplicationInsights. TraceListener без выбранных зависимостей, а затем выполнить следующие действия, чтобы удалить Microsoft. ApplicationInsights. Web с выбранным параметром удалить зависимости.

1. Щелкните пакет **Microsoft. ApplicationInsights. Web** . Справа установите флажок **Project (проект** ), чтобы выбрать все проекты.

1. Чтобы удалить все зависимости при удалении, нажмите кнопку раскрывающегося списка **Параметры** под разделом, в котором был выбран проект.

    В разделе *Параметры удаления* установите флажок рядом с элементом *удалить зависимости*.

1. Выберите **Удалить**.
    
    ![На снимке экрана показано окно Microsoft. ApplicationInsights. Web с выделенными флажками удалить зависимости и удалить.](./media/remove-application-insights/uninstall-framework.png)

    Отобразится диалоговое окно, в котором будут показаны все зависимости, удаляемые из приложения. Нажмите кнопку **ОК** , чтобы удалить.
    
    ![На снимке экрана показано диалоговое окно с удаляемыми зависимостями.](./media/remove-application-insights/preview-uninstall-framework.png)
    
1.  После удаления все еще можно увидеть "ApplicationInsights.config" и "Аихандлиррораттрибуте. cs" в *Обозреватель решений*. Эти два файла можно удалить вручную.

# <a name="net-core"></a>[.NET Core](#tab/netcore)

1. В *Обозреватель решений* справа щелкните правой кнопкой мыши **решение** и выберите пункт **Управление пакетами NuGet для решения**.

    После этого появится экран, позволяющий изменить все пакеты NuGet, которые являются частью проекта.

    ![Щелкните правой кнопкой мыши решение в обозреватель решений, а затем выберите Управление пакетами NuGet для решения.](./media/remove-application-insights/manage-nuget-core.png)

1. Щелкните пакет Microsoft. ApplicationInsights. AspNetCore. Справа установите флажок *проект* , чтобы выбрать все проекты, а затем выберите **Удалить**.

    ![Установите флажок удалить зависимости, а затем удалите](./media/remove-application-insights/uninstall-core.png)

---

## <a name="what-is-created-when-you-add-application-insights"></a>Что создается при добавлении Application Insights

При добавлении Application Insights в проект он создает файлы и добавляет код в некоторые файлы. Только удаление пакетов NuGet не всегда приводит к удалению файлов и кода. Чтобы полностью удалить Application Insights, необходимо проверить и вручную удалить добавленный код или файлы вместе с любыми вызовами API, добавленными в проект.

# <a name="net"></a>[.NET](#tab/net)

При добавлении телеметрия Application Insights в проект Visual Studio ASP.NET добавляются следующие файлы:

- ApplicationInsights.config
- Аихандлиррораттрибуте. CS

Добавляются следующие фрагменты кода:

- [Имя вашего проекта]. csproj

    ```C#
     <ApplicationInsightsResourceId>/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/Default-ApplicationInsights-EastUS/providers/microsoft.insights/components/WebApplication4</ApplicationInsightsResourceId>
    ```

- Packages.config

    ```xml
    <packages>
    ...
    
      <package id="Microsoft.ApplicationInsights" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.Agent.Intercept" version="2.4.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.DependencyCollector" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.PerfCounterCollector" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.Web" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.WindowsServer" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel" version="2.12.0" targetFramework="net472" />
    
      <package id="Microsoft.AspNet.TelemetryCorrelation" version="1.0.7" targetFramework="net472" />
    
      <package id="System.Buffers" version="4.4.0" targetFramework="net472" />
      <package id="System.Diagnostics.DiagnosticSource" version="4.6.0" targetFramework="net472" />
      <package id="System.Memory" version="4.5.3" targetFramework="net472" />
      <package id="System.Numerics.Vectors" version="4.4.0" targetFramework="net472" />
      <package id="System.Runtime.CompilerServices.Unsafe" version="4.5.2" targetFramework="net472" />
    ...
    </packages>
    ```

- Layout. cshtml

    Если в проекте есть файл Layout. cshtml, добавляется приведенный ниже код.
    
    ```html
    <head>
    ...
        <script type = 'text/javascript' >
            var appInsights=window.appInsights||function(config)
            {
                function r(config){ t[config] = function(){ var i = arguments; t.queue.push(function(){ t[config].apply(t, i)})} }
                var t = { config:config},u=document,e=window,o='script',s=u.createElement(o),i,f;for(s.src=config.url||'//az416426.vo.msecnd.net/scripts/a/ai.0.js',u.getElementsByTagName(o)[0].parentNode.appendChild(s),t.cookie=u.cookie,t.queue=[],i=['Event','Exception','Metric','PageView','Trace','Ajax'];i.length;)r('track'+i.pop());return r('setAuthenticatedUserContext'),r('clearAuthenticatedUserContext'),config.disableExceptionTracking||(i='onerror',r('_'+i),f=e[i],e[i]=function(config, r, u, e, o) { var s = f && f(config, r, u, e, o); return s !== !0 && t['_' + i](config, r, u, e, o),s}),t
            }({
                instrumentationKey:'00000000-0000-0000-0000-000000000000'
            });
            
            window.appInsights=appInsights;
            appInsights.trackPageView();
        </script>
    ...
    </head>
    ```

- ConnectedService.json

    ```json
    {
      "ProviderId": "Microsoft.ApplicationInsights.ConnectedService.ConnectedServiceProvider",
      "Version": "16.0.0.0",
      "GettingStartedDocument": {
        "Uri": "https://go.microsoft.com/fwlink/?LinkID=613413"
      }
    }
    ```

- FilterConfig.cs

    ```csharp
            public static void RegisterGlobalFilters(GlobalFilterCollection filters)
            {
                filters.Add(new ErrorHandler.AiHandleErrorAttribute());// This line was added
            }
    ```

# <a name="net-core"></a>[.NET Core](#tab/netcore)

При добавлении телеметрия Application Insights в проект шаблона Visual Studio ASP.NET Core добавляется следующий код:

- [Имя вашего проекта]. csproj

    ```csharp
      <PropertyGroup>
        <TargetFramework>netcoreapp3.1</TargetFramework>
        <ApplicationInsightsResourceId>/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/Default-ApplicationInsights-EastUS/providers/microsoft.insights/components/WebApplication4core</ApplicationInsightsResourceId>
      </PropertyGroup>
    
      <ItemGroup>
        <PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.12.0" />
      </ItemGroup>
    
      <ItemGroup>
        <WCFMetadata Include="Connected Services" />
      </ItemGroup>
    ```

- Appsettings.js:

    ```json
    "ApplicationInsights": {
        "InstrumentationKey": "00000000-0000-0000-0000-000000000000"
    ```

- ConnectedService.json
    
    ```json
    {
      "ProviderId": "Microsoft.ApplicationInsights.ConnectedService.ConnectedServiceProvider",
      "Version": "16.0.0.0",
      "GettingStartedDocument": {
        "Uri": "https://go.microsoft.com/fwlink/?LinkID=798432"
      }
    }
    ```
- Startup.cs

    ```csharp
       public void ConfigureServices(IServiceCollection services)
            {
                services.AddRazorPages();
                services.AddApplicationInsightsTelemetry(); // This is added
            }
    ```

---

## <a name="next-steps"></a>Дальнейшие действия

- [Azure Monitor](../overview.md)
