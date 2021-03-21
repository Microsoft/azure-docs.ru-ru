---
title: Анализ использования с помощью Azure Application Insights | Документация Майкрософт
description: Получение сведений о пользователях и их действиях с веб-приложением.
ms.topic: conceptual
ms.date: 03/25/2019
ms.openlocfilehash: d9de1e10363f2100b9dfe557dc12e0be951ce6b8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102489044"
---
# <a name="usage-analysis-with-application-insights"></a>Анализ использования с помощью Application Insights

Какие функции веб-приложения или мобильного приложения наиболее популярны? Достигают ли пользователи своих целей с помощью вашего приложения? Уходят ли они в определенные моменты и возвращаются ли после этого?  [Azure Application Insights](./app-insights-overview.md) помогает получить важные сведения о том, как люди используют ваше приложение. Каждый раз, обновив приложение, можно оценить, насколько хорошо оно подходит пользователям. Обладая такими сведениями, можно принять решения на основе данных по дальнейшим циклам разработки.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4Cijb]

## <a name="send-telemetry-from-your-app"></a>Отправка телеметрии из приложения

Максимальное удобство работы достигается путем установки Application Insights как в серверном коде приложения, так и на веб-страницах. Клиентские и серверные компоненты приложения отправляют данные телеметрии на портал Azure для анализа.

1. **Серверный код.** Установите соответствующий модуль для своего приложения [ASP.NET](./asp-net.md), [Azure](./app-insights-overview.md), [Java](./java-get-started.md), [Node.js](./nodejs.md) или приложения [иного типа](./platforms.md).

    * *Не хотите устанавливать серверный код? Просто [создайте ресурс Azure Application Insights](./create-new-resource.md).*

2. **Код веб-страницы.** Добавьте следующий скрипт на веб-страницу перед закрывающим тегом ``</head>``. Замените ключ инструментирования на подходящее значение для ресурса Application Insights.
    
    Текущий фрагмент кода (приведенный ниже) — версия 5, версия кодируется в фрагменте как SV: "#", а [Текущая версия также доступна на сайте GitHub](https://go.microsoft.com/fwlink/?linkid=2156318).

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
      instrumentationKey:"INSTRUMENTATION_KEY"
    }});
    </script>
    ```

    Чтобы узнать о более сложных настройках для мониторинга веб-сайтов, ознакомьтесь со статьей [о работе с пакетом SDK для JavaScript](./javascript.md).

3. **Код мобильного приложения.** Используйте пакет SDK для Центра приложений, чтобы собирать события из приложения, а затем отправлять копии этих событий в Application Insights для анализа, следуя инструкциям в [этом руководстве](../app/mobile-center-quickstart.md).

4. **Получение данных телеметрии.** Запустите проект в режиме отладки на несколько минут, а затем просмотрите результаты в колонке "Обзор" в Application Insights.

    Опубликуйте ваше приложение для отслеживания его производительности и узнайте, что делают с ним пользователи.

## <a name="explore-usage-demographics-and-statistics"></a>Изучение демографических данных об использовании и статистики
Узнайте, когда люди используют ваше приложение, какие страницы им наиболее интересны, где находятся ваши пользователи, какие браузеры и операционные системы они используют. 

Отчеты о пользователях и сеансах позволяют фильтровать данные по страницам или пользовательским событиям, а также разделять их по свойствам, таким как расположение, среда и страница. Можно также добавить собственные фильтры.

![Снимок экрана показывает страницу обзора пользователей для вымышленной компании.](./media/usage-overview/users.png)  

Аналитические сведения справа позволяют выделять важные закономерности в наборе данных.  

* В отчете **Пользователи** подсчитывается количество уникальных пользователей, которые обращаются к страницам в выбранные периоды времени. (Пользователи веб-приложений подсчитываются с помощью файлов cookie. Если пользователь обращается к вашему сайту, используя разные браузеры или клиентские компьютеры, или очищает файлы cookie, он будет учтен соответствующее количество раз.)
* В отчете **Сеансы** подсчитывается количество пользовательских сеансов, обращающихся к узлу. Сеанс — это период действий пользователя, завершающийся периодом бездействия более получаса.

[Дополнительные сведения об инструментах "Пользователи", "Сеансы" и "События".](usage-segmentation.md)  

## <a name="retention---how-many-users-come-back"></a>Период удержания — сколько пользователей вернулось?

Период удержания помогает понять, как часто пользователи возвращаются к использованию своего приложения. Для этого используются когорты пользователей, которые выполнили какое-либо коммерческое действие во время определенного горизонта планирования. 

- Поймите, какие именно функции заставляют пользователей возвращаться чаще, чем другие. 
- Сформируйте гипотезу на основе данных реальных пользователей. 
- Определите, является ли период удержания проблемой для вашего продукта. 

![На снимке экрана показана страница "Обзор хранения", на которой отображаются сведения о том, как часто пользователи возвращают данные для использования приложения.](./media/usage-overview/retention.png) 

Помимо прочего, элементы управления периодом удержания позволяют определить конкретные события и диапазон времени для вычисления периода удержания. График в середине дает визуальное представление об общем проценте удержания за указанный диапазон времени. График внизу отображает период удержания отдельных пользователей за заданный период времени. Такой уровень детализации позволяет лучше понять, что ваши пользователи делают и что может повлиять на возвращение пользователей.  

[Дополнительные сведения об инструменте "Хранение"](usage-retention.md).

## <a name="custom-business-events"></a>Пользовательские бизнес-события

Чтобы получить четкое представление о действиях пользователей с приложением, полезно добавить строки кода для ведения журнала пользовательских событий. Эти события могут отслеживать что угодно, начиная с детализированных действий пользователя (например, нажатие определенных кнопок) и заканчивая более серьезными бизнес-событиями (например, совершение покупки или победа в игре).

Для сбора пользовательских событий можно также использовать [подключаемый модуль автосбора в меню click Analytics](javascript-click-analytics-plugin.md) .

Хотя в некоторых случаях просмотры страниц могут представлять собой полезные события, но в целом это не так. Пользователь может открыть страницу продукта, не приобретая его. 

С помощью специальных бизнес-событий можно создать диаграмму перемещения пользователей по сайту. Можно узнать их предпочтения относительно различных параметров, а также определить, где они покидают сайт или испытывают сложности. Обладая такими сведениями, можно принимать обоснованные решения о приоритетах для невыполненной работы по разработке.

События могут регистрироваться на клиентской стороне приложения:

```JavaScript

    appInsights.trackEvent("ExpandDetailTab", {DetailTab: tabName});
```

Или на стороне сервера:

```csharp
    var tc = new Microsoft.ApplicationInsights.TelemetryClient();
    tc.TrackEvent("CreatedAccount", new Dictionary<string,string> {"AccountType":account.Type}, null);
    ...
    tc.TrackEvent("AddedItemToCart", new Dictionary<string,string> {"Item":item.Name}, null);
    ...
    tc.TrackEvent("CompletedPurchase");
```

В эти события можно вложить значения свойств, чтобы события можно было фильтровать или разделять при изучении на портале. Кроме того, в каждое событие вкладывается стандартный набор свойств, например идентификатор анонимного пользователя, который позволяет выполнять трассировку последовательности действий отдельного пользователя.

Узнайте больше о [пользовательских событиях](./api-custom-events-metrics.md#trackevent) и [свойствах](./api-custom-events-metrics.md#properties).

### <a name="slice-and-dice-events"></a>Анализ событий

В инструментах "Пользователи", "Сеансы" и "События" можно анализировать пользовательские события по пользователю, имени события и свойствам.
![Снимок экрана показывает страницу обзора пользователей для вымышленной компании.](./media/usage-overview/users.png)  
  
## <a name="design-the-telemetry-with-the-app"></a>Разработка телеметрии приложения

При разработке каждого компонента приложения обдумайте, как вы будете измерять его успех у пользователей. Решите, какие бизнес-события необходимо записывать, и с самого начала запрограммируйте вызовы отслеживания для этих событий в приложении.

## <a name="a--b-testing"></a>Тестирование A или B
Если неизвестно, какой вариант функции будет более эффективен, выпустите их оба, чтобы каждый из них был доступен для разных пользователей. Измерьте успешность каждого варианта, а затем перейдите к единой версии.

В рамках этого способа необходимо вложить разные значения свойств во все данные телеметрии, отправляемые каждой версией приложения. Для этого нужно определить свойства в активном контексте телеметрии TelemetryContext. Эти свойства по умолчанию добавляются к каждому сообщению телеметрии, которое отправляет приложение, — не только к настраиваемым сообщениям, но также к стандартным данным телеметрии.

Чтобы сравнить различные версии, на портале Application Insights можно отфильтровать и разделить данные по значениям свойств.

Для этого следует [настроить инициализатор телеметрии](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer).

**Приложения ASP.NET**

```csharp
    // Telemetry initializer class
    public class MyTelemetryInitializer : ITelemetryInitializer
    {
        public void Initialize(ITelemetry item)
            {
                var itemProperties = item as ISupportProperties;
                if (itemProperties != null && !itemProperties.Properties.ContainsKey("AppVersion"))
                {
                    itemProperties.Properties["AppVersion"] = "v2.1";
                }
            }
    }
```

В инициализаторе веб-приложения, например Global.asax.cs, добавьте следующий код.

```csharp

    protected void Application_Start()
    {
        // ...
        TelemetryConfiguration.Active.TelemetryInitializers
         .Add(new MyTelemetryInitializer());
    }
```

**Приложения ASP.NET Core**

> [!NOTE]
> Добавление инициализатора с помощью `ApplicationInsights.config` или использование `TelemetryConfiguration.Active` недопустимо для приложений ASP.NET Core. 

В приложениях [ASP.NET Core](asp-net-core.md#adding-telemetryinitializers) добавление нового `TelemetryInitializer` выполняется путем его добавления в контейнер внедрения зависимостей, как показано ниже. Это делается в методе `ConfigureServices` класса `Startup.cs`.

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;
 public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ITelemetryInitializer, MyTelemetryInitializer>();
}
```

Все новые клиенты телеметрии автоматически добавляют указанное значение свойства. Отдельные события телеметрии могут переопределять значения по умолчанию.

## <a name="next-steps"></a>Дальнейшие действия
   - [Пользователи, сеансы, события](usage-segmentation.md)
   - [Воронки](usage-funnels.md)
   - [Сохранение](usage-retention.md)
   - [Средство "Маршруты пользователей"](usage-flows.md)
   - [книги](../visualize/workbooks-overview.md)
