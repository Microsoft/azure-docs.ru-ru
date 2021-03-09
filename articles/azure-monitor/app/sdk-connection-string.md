---
title: Строки подключения в Application Insights Azure | Документация Майкрософт
description: Как использовать строки подключения.
ms.topic: conceptual
author: timothymothra
ms.author: tilee
ms.date: 01/17/2020
ms.custom: devx-track-js, devx-track-csharp
ms.reviewer: mbullwin
ms.openlocfilehash: df87b060423aeff9fa5f83f21634395fe30e0bbb
ms.sourcegitcommit: 8d1b97c3777684bd98f2cfbc9d440b1299a02e8f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102486290"
---
# <a name="connection-strings"></a>Строки подключения

## <a name="overview"></a>Обзор

Строки подключения предоставляют пользователям Application Insights один параметр конфигурации, устраняя необходимость в нескольких параметрах прокси-сервера. Очень полезна для веб-серверов в интрасети, независимых или гибридных облачных сред, которые ищут передачу данных в службу мониторинга.

Пары "ключ-значение" предоставляют пользователям простой способ определения префикса суффикса для каждой службы Application Insights (AI) или продукта.

> [!IMPORTANT]
> Не рекомендуется задавать как строку подключения, так и ключ инструментирования. В случае, если пользователь установил оба значения, приоритет будет задается в зависимости от того, какое значение было задано последним. 


## <a name="scenario-overview"></a>Общие сведения о сценарии 

Сценарии клиентов, в которых мы видим, что это имеет наибольшее влияние:

- Исключения брандмауэра или перенаправления прокси-сервера 

    В тех случаях, когда требуется мониторинг веб-сервера интрасети, наше более раннее решение попросило пользователей добавить в конфигурацию отдельные конечные точки служб. Дополнительные сведения см. [здесь](../faq.md#can-i-monitor-an-intranet-web-server). 
    Строки подключения предлагают лучшую альтернативу, уменьшая эти усилия на один параметр. Простой префикс, поправку суффиксов позволяет автоматически выполнять заполнение и перенаправление всех конечных точек в нужные службы. 

- Независимых или гибридные облачные среды

    Пользователи могут передавать данные в определенный [регион Azure для государственных организаций](../../azure-government/compare-azure-government-global-azure.md#application-insights).
    Строки подключения позволяют определить параметры конечной точки для серверов интрасети или параметров гибридного облака. 

## <a name="getting-started"></a>Начало работы

### <a name="finding-my-connection-string"></a>Найти строку подключения?

Строка подключения отобразится в колонке обзора ресурса Application Insights.

![Строка подключения в колонке "Обзор"](media/overview-dashboard/overview-connection-string.png)

### <a name="schema"></a>схема

#### <a name="max-length"></a>Максимальная длина

Максимальная поддерживаемая длина подключения — 4096 символов.

#### <a name="key-value-pairs"></a>Пары "ключ-значение"

Строка подключения состоит из списка параметров, представленных парами "ключ-значение", разделенных точкой с запятой. `key1=value1;key2=value2;key3=value3`

#### <a name="syntax"></a>Синтаксис

- `InstrumentationKey` (пример: 00000000-0000-0000-0000-000000000000)  Строка подключения является **обязательным** полем.
- `Authorization` (пример: iKey) (Этот параметр является необязательным, так как сегодня поддерживается только ikeyная авторизация.)
- `EndpointSuffix` (пример: applicationinsights.azure.cn) При настройке суффикса конечной точки будет указано, к какому пакету SDK подключается облако Azure. Пакет SDK будет собирать оставшуюся часть конечной точки для отдельных служб.
- Явные конечные точки.
  Любая служба может быть явно переопределена в строке подключения.
   - `IngestionEndpoint` (пример: `https://dc.applicationinsights.azure.com` )
   - `LiveEndpoint` (пример: `https://live.applicationinsights.azure.com` )
   - `ProfilerEndpoint` (пример: `https://profiler.monitor.azure.com` )
   - `SnapshotEndpoint` (пример: `https://snapshot.monitor.azure.com` )

#### <a name="endpoint-schema"></a>Схема конечной точки

`<prefix>.<suffix>`
- Префикс: определяет службу. 
- Суффикс: определяет общее доменное имя.

##### <a name="valid-suffixes"></a>Допустимые суффиксы

Ниже приведен список допустимых суффиксов. 
- applicationinsights.azure.cn
- applicationinsights.us


Дополнительные сведения см. по ссылке https://docs.microsoft.com/azure/azure-monitor/app/custom-endpoints#regions-that-require-endpoint-modification


##### <a name="valid-prefixes"></a>Допустимые префиксы

- Прием [телеметрии](./app-insights-overview.md):`dc`
- [Динамические метрики](./live-stream.md): `live`
- [Профилировщик](./profiler-overview.md): `profiler`
- [Моментальный снимок](./snapshot-debugger.md): `snapshot`



## <a name="connection-string-examples"></a>Примеры строк подключения


### <a name="minimal-valid-connection-string"></a>Минимальная допустимая строка подключения

`InstrumentationKey=00000000-0000-0000-0000-000000000000;`

В этом примере задан только ключ инструментирования.

- По умолчанию используется схема авторизации "iKey" 
- Ключ инструментирования: 00000000-0000-0000-0000-000000000000
- Коды URI региональных служб основаны на [стандартах SDK](https://github.com/microsoft/ApplicationInsights-dotnet/blob/develop/BASE/src/Microsoft.ApplicationInsights/Extensibility/Implementation/Endpoints/Constants.cs) и будут подключаться к общедоступному глобальному серверу Azure:
   - Приема `https://dc.services.visualstudio.com/`
   - Динамические метрики: `https://rt.services.visualstudio.com/`
   - Profiler `https://profiler.monitor.azure.com/`
   - Отладчик `https://snapshot.monitor.azure.com/`



### <a name="connection-string-with-endpoint-suffix"></a>Строка подключения с суффиксом конечной точки

`InstrumentationKey=00000000-0000-0000-0000-000000000000;EndpointSuffix=ai.contoso.com;`

В этом примере в этой строке подключения указывается суффикс конечной точки, и пакет SDK конструирует конечные точки службы.

- По умолчанию используется схема авторизации "iKey" 
- Ключ инструментирования: 00000000-0000-0000-0000-000000000000
- Коды URI региональных служб основаны на указанном суффиксе конечной точки: 
   - Приема `https://dc.ai.contoso.com`
   - Динамические метрики: `https://live.ai.contoso.com`
   - Profiler `https://profiler.ai.contoso.com`
   - Отладчик `https://snapshot.ai.contoso.com`  



### <a name="connection-string-with-explicit-endpoint-overrides"></a>Строка подключения с явными переопределениями конечной точки 

`InstrumentationKey=00000000-0000-0000-0000-000000000000;IngestionEndpoint=https://custom.com:111/;LiveEndpoint=https://custom.com:222/;ProfilerEndpoint=https://custom.com:333/;SnapshotEndpoint=https://custom.com:444/;`

В этом примере эта строка подключения указывает явные переопределения для каждой службы. Пакет SDK будет использовать точные указанные конечные точки без изменения.

- По умолчанию используется схема авторизации "iKey" 
- Ключ инструментирования: 00000000-0000-0000-0000-000000000000
- Региональные URI служб основаны на явных значениях переопределения: 
   - Приема `https://custom.com:111/`
   - Динамические метрики: `https://custom.com:222/`
   - Profiler `https://custom.com:333/`
   - Отладчик `https://custom.com:444/`  


## <a name="how-to-set-a-connection-string"></a>Как задать строку подключения

Строки подключения поддерживаются в следующих версиях пакета SDK:
- .NET и .NET Core v 2.12.0
- Java v 2.5.1 и Java 3,0
- JavaScript v 2.3.0
- NodeJS v 1.5.0
- Версия Python версии 1.0.0

Строку подключения можно задать в коде, переменной среды или файле конфигурации.



### <a name="environment-variable"></a>Переменная среды

- Строка подключения: `APPLICATIONINSIGHTS_CONNECTION_STRING`

### <a name="code-samples"></a>Примеры кода

# <a name="netnetcore"></a>[.NET/. NetCore](#tab/net)

Задайте свойство [телеметриконфигуратион. ConnectionString](https://github.com/microsoft/ApplicationInsights-dotnet/blob/add45ceed35a817dc7202ec07d3df1672d1f610d/BASE/src/Microsoft.ApplicationInsights/Extensibility/TelemetryConfiguration.cs#L271-L274) или [аппликатионинсигхтссервицеоптионс. ConnectionString.](https://github.com/microsoft/ApplicationInsights-dotnet/blob/81288f26921df1e8e713d31e7e9c2187ac9e6590/NETCORE/src/Shared/Extensions/ApplicationInsightsServiceOptions.cs#L66-L69)

.NET явным образом задается:
```csharp
var configuration = new TelemetryConfiguration
{
    ConnectionString = "InstrumentationKey=00000000-0000-0000-0000-000000000000;"
};
```

Файл конфигурации .NET:

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">
    <ConnectionString>InstrumentationKey=00000000-0000-0000-0000-000000000000</ConnectionString>
</ApplicationInsights>
```

NetCore явно задано:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    var options = new ApplicationInsightsServiceOptions { ConnectionString = "InstrumentationKey=00000000-0000-0000-0000-000000000000;" };
    services.AddApplicationInsightsTelemetry(options: options);
}
```

config.jsNetCore: 

```json
{
  "ApplicationInsights": {
    "ConnectionString" : "InstrumentationKey=00000000-0000-0000-0000-000000000000;"
    }
  }
```


# <a name="java"></a>[Java](#tab/java)


Java (версия 2.5. x) явно задано:
```java
TelemetryConfiguration.getActive().setConnectionString("InstrumentationKey=00000000-0000-0000-0000-000000000000");
```

ApplicationInsights.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">
    <ConnectionString>InstrumentationKey=00000000-0000-0000-0000-000000000000;</ConnectionString>
</ApplicationInsights>
```

# <a name="javascript"></a>[JavaScript](#tab/js)

Важно. JavaScript не поддерживает использование переменных среды.

Использование фрагмента кода:

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
  connectionString:"InstrumentationKey=00000000-0000-0000-0000-000000000000;"
}});
</script>
```

> [!NOTE]
> Для удобства чтения и сокращения возможных ошибок JavaScript все возможные варианты настройки перечислены в новой строке в коде фрагмента кода выше, если вы не хотите изменять значение строки с комментарием, ее можно удалить.

Ручная настройка:
```javascript
import { ApplicationInsights } from '@microsoft/applicationinsights-web'

const appInsights = new ApplicationInsights({ config: {
  connectionString: 'InstrumentationKey=00000000-0000-0000-0000-000000000000;'
  /* ...Other Configuration Options... */
} });
appInsights.loadAppInsights();
appInsights.trackPageView();
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

```javascript
const appInsights = require("applicationinsights");
appInsights.setup("InstrumentationKey=00000000-0000-0000-0000-000000000000;");
appInsights.start();
```

# <a name="python"></a>[Python](#tab/python)

Мы рекомендуем пользователям задать переменную среды.

Чтобы явно задать строку подключения, сделайте следующее:

```python
from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.trace.samplers import ProbabilitySampler
from opencensus.trace.tracer import Tracer

tracer = Tracer(exporter=AzureExporter(connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000'), sampler=ProbabilitySampler(1.0))
```

---

## <a name="next-steps"></a>Дальнейшие действия

Приступите к работе во время выполнения с помощью:

* [Приложения, размещенные в IIS на виртуальной машине Azure или в масштабируемом наборе виртуальных машин Azure](./azure-vm-vmss-apps.md)
* [сервера IIS;](./monitor-performance-live-website-now.md)
* [Веб-приложения Azure](./azure-web-apps.md)

Приступите к работе во время разработки с помощью:

* [ASP.NET](./asp-net.md)
* [ASP.NET Core](./asp-net-core.md)
* [Java](./java-get-started.md)
* [Node.js](./nodejs.md)
* [Python](./opencensus-python.md)

