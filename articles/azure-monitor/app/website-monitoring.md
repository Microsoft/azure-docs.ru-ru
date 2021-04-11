---
title: Краткое руководство. Мониторинг веб-сайтов с помощью Azure Monitor Application Insights
description: В этом кратком руководстве показано, как настроить мониторинг веб-сайта на стороне клиента или браузера с помощью Application Insights в Azure Monitor.
ms.topic: quickstart
ms.date: 03/19/2021
ms.custom: mvc
ms.openlocfilehash: 0e10db39c8dbbf81087d696cfbb5b2ded1ae79ac
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104654934"
---
# <a name="quickstart-start-monitoring-your-website-with-azure-monitor-application-insights"></a>Краткое руководство. Запуск мониторинга веб-сайта с помощью Application Insights в Azure Monitor

В этом кратком руководстве содержатся сведения о добавлении пакета SDK с открытым кодом для JavaScript Application Insights на веб-сайт. Вы также получите сведения, которые помогут улучшить понимание взаимодействия на стороне клиента или браузера для посетителей веб-сайта.

С помощью Azure Monitor Application Insights вы можете легко наблюдать за доступностью, производительностью и использованием вашего веб-сайта. Вы также можете быстро идентифицировать и диагностировать в нем ошибки, не дожидаясь, пока пользователь сообщит о них. Application Insights обеспечивает мониторинг на стороне сервера, а также на стороне клиента или браузера.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
* Веб-сайт, к которому можно добавить пакет SDK для Application Insights JavaScript.

## <a name="enable-application-insights"></a>Включение Application Insights

Application Insights может собирать данные телеметрии из любого подключенного к Интернету приложения, которое работает в локальной среде или облаке. Чтобы просмотреть эти данные, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Выберите **Создать ресурс** > **Средства управления** > **Application Insights**.

   > [!NOTE]
   >См. сведения о [создании ресурса Application Insights](./create-new-resource.md).
1. Когда откроется окно конфигурации, заполните поля, как показано в таблице ниже.

    | Настройки        | Значение           | Описание  |
   | ------------- |:-------------|:-----|
   | **имя**;      | Глобально уникальное значение | Имя, идентифицирующее отслеживаемое приложение. |
   | **Группа ресурсов**     | myResourceGroup      | Имя новой группы ресурсов для размещения данных Application Insights. Создайте новую группу ресурсов или выберите существующую. |
   | **Расположение** | Восточная часть США | Выберите ближайшее к вам расположение или расположение рядом с местом размещения приложения. |
1. Нажмите кнопку **создания**.

## <a name="create-an-html-file"></a>Создание файла HTML

1. На локальном компьютере создайте файл с именем ``hello_world.html``. Для этого примера создайте файл в корне диска C, чтобы он выглядел так: ``C:\hello_world.html``.
1. Скопируйте и вставьте следующий скрипт в ``hello_world.html``.

    ```html
    <!DOCTYPE html>
    <html>
    <head>
    <title>Azure Monitor Application Insights</title>
    </head>
    <body>
    <h1>Azure Monitor Application Insights Hello World!</h1>
    <p>You can use the Application Insights JavaScript SDK to perform client/browser-side monitoring of your website. To learn about more advanced JavaScript SDK configurations, visit the <a href="https://github.com/Microsoft/ApplicationInsights-JS/blob/master/API-reference.md" title="API Reference">API reference</a>.</p>
    </body>
    </html>
    ```

## <a name="configure-application-insights-sdk"></a>Настройка пакета SDK для Application Insights

1. Выберите **Обзор** и скопируйте **строку подключения** приложения. В этом примере нам нужна только часть ключа инструментирования в строке подключения `InstrumentationKey=00000000-0000-0000-0000-000000000000;`.

    :::image type="content" source="media/website-monitoring/keys.png" alt-text="Снимок экрана: страница обзора с ключом инструментирования и строкой подключения.":::

1. Добавьте следующий скрипт в файл ``hello_world.html`` перед закрывающим тегом ``</head>``.

    ```html
    <script type="text/javascript">
    !function(T,l,y){var S=T.location,k="script",D="instrumentationKey",C="ingestionendpoint",I="disableExceptionTracking",E="ai.device.",b="toLowerCase",w="crossOrigin",N="POST",e="appInsightsSDK",t=y.name||"appInsights";(y.name||T[e])&&(T[e]=t);var n=T[t]||function(d){var g=!1,f=!1,m={initialize:!0,queue:[],sv:"5",version:2,config:d};function v(e,t){var n={},a="Browser";return n[E+"id"]=a[b](),n[E+"type"]=a,n["ai.operation.name"]=S&&S.pathname||"_unknown_",n["ai.internal.sdkVersion"]="javascript:snippet_"+(m.sv||m.version),{time:function(){var e=new Date;function t(e){var t=""+e;return 1===t.length&&(t="0"+t),t}return e.getUTCFullYear()+"-"+t(1+e.getUTCMonth())+"-"+t(e.getUTCDate())+"T"+t(e.getUTCHours())+":"+t(e.getUTCMinutes())+":"+t(e.getUTCSeconds())+"."+((e.getUTCMilliseconds()/1e3).toFixed(3)+"").slice(2,5)+"Z"}(),iKey:e,name:"Microsoft.ApplicationInsights."+e.replace(/-/g,"")+"."+t,sampleRate:100,tags:n,data:{baseData:{ver:2}}}}var h=d.url||y.src;if(h){function a(e){var t,n,a,i,r,o,s,c,u,p,l;g=!0,m.queue=[],f||(f=!0,t=h,s=function(){var e={},t=d.connectionString;if(t)for(var n=t.split(";"),a=0;a<n.length;a++){var i=n[a].split("=");2===i.length&&(e[i[0][b]()]=i[1])}if(!e[C]){var r=e.endpointsuffix,o=r?e.location:null;e[C]="https://"+(o?o+".":"")+"dc."+(r||"services.visualstudio.com")}return e}(),c=s[D]||d[D]||"",u=s[C],p=u?u+"/v2/track":d.endpointUrl,(l=[]).push((n="SDK LOAD Failure: Failed to load Application Insights SDK script (See stack for details)",a=t,i=p,(o=(r=v(c,"Exception")).data).baseType="ExceptionData",o.baseData.exceptions=[{typeName:"SDKLoadFailed",message:n.replace(/\./g,"-"),hasFullStack:!1,stack:n+"\nSnippet failed to load ["+a+"] -- Telemetry is disabled\nHelp Link: https://go.microsoft.com/fwlink/?linkid=2128109\nHost: "+(S&&S.pathname||"_unknown_")+"\nEndpoint: "+i,parsedStack:[]}],r)),l.push(function(e,t,n,a){var i=v(c,"Message"),r=i.data;r.baseType="MessageData";var o=r.baseData;return o.message='AI (Internal): 99 message:"'+("SDK LOAD Failure: Failed to load Application Insights SDK script (See stack for details) ("+n+")").replace(/\"/g,"")+'"',o.properties={endpoint:a},i}(0,0,t,p)),function(e,t){if(JSON){var n=T.fetch;if(n&&!y.useXhr)n(t,{method:N,body:JSON.stringify(e),mode:"cors"});else if(XMLHttpRequest){var a=new XMLHttpRequest;a.open(N,t),a.setRequestHeader("Content-type","application/json"),a.send(JSON.stringify(e))}}}(l,p))}function i(e,t){f||setTimeout(function(){!t&&m.core||a()},500)}var e=function(){var n=l.createElement(k);n.src=h;var e=y[w];return!e&&""!==e||"undefined"==n[w]||(n[w]=e),n.onload=i,n.onerror=a,n.onreadystatechange=function(e,t){"loaded"!==n.readyState&&"complete"!==n.readyState||i(0,t)},n}();y.ld<0?l.getElementsByTagName("head")[0].appendChild(e):setTimeout(function(){l.getElementsByTagName(k)[0].parentNode.appendChild(e)},y.ld||0)}try{m.cookie=l.cookie}catch(p){}function t(e){for(;e.length;)!function(t){m[t]=function(){var e=arguments;g||m.queue.push(function(){m[t].apply(m,e)})}}(e.pop())}var n="track",r="TrackPage",o="TrackEvent";t([n+"Event",n+"PageView",n+"Exception",n+"Trace",n+"DependencyData",n+"Metric",n+"PageViewPerformance","start"+r,"stop"+r,"start"+o,"stop"+o,"addTelemetryInitializer","setAuthenticatedUserContext","clearAuthenticatedUserContext","flush"]),m.SeverityLevel={Verbose:0,Information:1,Warning:2,Error:3,Critical:4};var s=(d.extensionConfig||{}).ApplicationInsightsAnalytics||{};if(!0!==d[I]&&!0!==s[I]){var c="onerror";t(["_"+c]);var u=T[c];T[c]=function(e,t,n,a,i){var r=u&&u(e,t,n,a,i);return!0!==r&&m["_"+c]({message:e,url:t,lineNumber:n,columnNumber:a,error:i}),r},d.autoExceptionInstrumented=!0}return m}(y.cfg);function a(){y.onInit&&y.onInit(n)}(T[t]=n).queue&&0===n.queue.length?(n.queue.push(a),n.trackPageView({})):a()}(window,document,{
    src: "https://js.monitor.azure.com/scripts/b/ai.2.min.js", // The SDK URL Source
    // name: "appInsights", // Global SDK Instance name defaults to "appInsights" when not supplied
    // ld: 0, // Defines the load delay (in ms) before attempting to load the sdk. -1 = block page load and add to head. (default) = 0ms load after timeout,
    // useXhr: 1, // Use XHR instead of fetch to report failures (if available),
    crossOrigin: "anonymous", // When supplied this will add the provided value as the cross origin attribute on the script tag
    // onInit: null, // Once the application insights instance has loaded and initialized this callback function will be called with 1 argument -- the sdk instance (DO NOT ADD anything to the sdk.queue -- As they won't get called)
    cfg: { // Application Insights Configuration
        connectionString:"InstrumentationKey=YOUR_INSTRUMENTATION_KEY_GOES_HERE;" 
        /* ...Other Configuration Options... */
    }});
    </script>
    ```

    > [!NOTE]
    > Текущий фрагмент кода (приведенный выше) имеет версию 5, версия кодируется в фрагменте в виде sv:"#", а [сведения о текущей версии и конфигурации доступны на сайте GitHub](https://go.microsoft.com/fwlink/?linkid=2156318).

1. Измените ``hello_world.html`` и добавьте ключ инструментирования.

1. Откройте ``hello_world.html`` в сеансе локального браузера. Это действие создает один просмотр страницы. Вы можете обновить свой браузер, чтобы сгенерировать несколько просмотров тестовых страниц.

## <a name="monitor-your-website-in-the-azure-portal"></a>Мониторинг веб-сайта на портале Azure

1. Откройте страницу **Обзор** Application Insights на портале Azure для просмотра сведений о выполняющемся приложении. Страница **Обзор** является местом получения ключа инструментирования.

   Четыре диаграммы по умолчанию на обзорной странице привязаны к серверным данным приложения. Так как мы рассматриваем взаимодействие на стороне клиента или браузера с использованием пакета SDK для JavaScript, это конкретное представление не применяется, если у нас также не установлен пакет SDK на стороне сервера.

1. Выберите **Журналы**.  Это действие приводит к открытию окна **Журналы**, где доступен полнофункциональный язык запросов для анализа всех данных, собранных Application Insights. Для просмотра данных, связанных с запросами браузера на стороне клиента, выполните следующий запрос.

    ```kusto
    // average pageView duration by name
    let timeGrain=1s;
    let dataset=pageViews
    // additional filters can be applied here
    | where timestamp > ago(15m)
    | where client_Type == "Browser" ;
    // calculate average pageView duration for all pageViews
    dataset
    | summarize avg(duration) by bin(timestamp, timeGrain)
    | extend pageView='Overall'
    // render result in a chart
    | render timechart
    ```

   :::image type="content" source="media/website-monitoring/log-query.png" alt-text="Снимок экрана: граф анализа журналов с запросами пользователей за определенный период времени.":::

1. Вернитесь на страницу **Обзор**. Под заголовком **Изучение** щелкните **Производительность** и откройте вкладку **Браузер**.  Отобразятся метрики, отображающие сведения о производительности вашего веб-сайта. Доступно соответствующее представление для анализа сбоев и исключений на веб-сайте. Щелкните **Примеры**, чтобы получить доступ к [полным сведениям о транзакциях](./transaction-diagnostics.md).

     :::image type="content" source="media/website-monitoring/performance.png" alt-text="Снимок экрана: вкладка &quot;Производительность&quot; с графом метрик браузера.":::

1. Чтобы изучить возможности [средств аналитики поведения пользователей](./usage-overview.md), в главном меню Application Insights под заголовком **Использование** щелкните [**Пользователи**](./usage-segmentation.md). Так как мы выполняем тестирование на одном компьютере, мы увидим данные только для одного пользователя.

1. Для более сложного веб-сайта с несколькими страницами можно использовать средство [**Маршруты пользователей**](./usage-flows.md) для визуализации перемещения посетителей по веб-сайту.

Сведения о более сложных настройках для мониторинга веб-сайтов см. в справочнике по [API пакета SDK для JavaScript](./javascript.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете продолжать работу с другими руководствами, не удаляйте созданные ресурсы. В противном случае выполните на портале Azure следующие действия, чтобы удалить все созданные ресурсы.

> [!NOTE]
> Если вы использовали существующую группу ресурсов, приведенные ниже инструкции неприменимы. Вместо этого можете просто удалить отдельный ресурс Application Insights. Помните, что при удалении группы ресурсов все входящие в нее базовые ресурсы также будут удалены.

1. В меню слева на портале Azure щелкните **Группы ресурсов** и выберите **myResourceGroup** или имя временной группы ресурсов.
1. На странице группы ресурсов щелкните **Удалить**, в текстовом поле введите **myResourceGroup** и выберите **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Аналитика в Application Insights](../logs/log-query-overview.md)

