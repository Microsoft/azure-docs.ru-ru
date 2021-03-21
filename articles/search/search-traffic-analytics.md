---
title: Данные телеметрии для аналитики трафика поиска
titleSuffix: Azure Cognitive Search
description: Включите аналитику трафика поиска для Когнитивного поиска Azure, собирайте данные телеметрии и сведения о событиях, инициированных пользователем, с помощью Application Insights, а затем анализируйте полученные результаты в отчете Power BI.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 1/29/2021
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 89d067162dacb7a0ca25de826630e1986f1035e1
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102485475"
---
# <a name="collect-telemetry-data-for-search-traffic-analytics"></a>Сбор данных телеметрии для аналитики трафика поиска

Аналитика трафика поиска — это шаблон для сбора данных телеметрии о взаимодействии пользователей с приложением Когнитивного поиска Azure, таких как инициированные пользователем события щелчка мышью и ввода с клавиатуры. С помощью этих сведений можно определить эффективность решения для поиска, включая популярные условия поиска, частоту переходов, а также то, какие запросы выдают нулевые результаты.

Этот шаблон зависит от [Application Insights](../azure-monitor/app/app-insights-overview.md) (функция [Azure Monitor](../azure-monitor/index.yml)) для сбора данных пользователей. Для этого необходимо добавить инструментирование в код клиента, как описано в этой статье. Наконец, для анализа данных потребуется механизм создания отчетов. Рекомендуется Power BI, но можно использовать панель мониторинга приложения или любое средство, которое подключается к Application Insights.

> [!NOTE]
> Шаблон, описанный в этой статье, предназначен для расширенных сценариев и навигации данных, создаваемых кодом, который добавляется в клиент. Журналы служб, напротив, просты в настройке, предоставляют ряд метрик и могут выполняться на портале без кода. Для всех сценариев рекомендуется включить ведение журнала. Дополнительные сведения см. в разделе [Сбор и анализ данных журналов](search-monitor-logs.md).

## <a name="identify-relevant-search-data"></a>Определение соответствующих данных поиска

Для получения полезных метрик трафика поиска необходимо организовать регистрацию некоторых пользовательских событий в приложении поиска. Эти события обозначают интересующее пользователей содержимое, которое является соответствующим для них. Для аналитики трафика поиска такие события включают следующее.

+ События поиска, инициированные пользователями. Представляют интерес только поисковые запросы, инициируемые пользователем. Другие запросы поиска, например, используемые для заполнения аспектов или получения внутренней информации, не важны. Не забудьте инструментировать только инициированные пользователем события, чтобы избежать смещения или смещения в результатах.

+ Пользовательские события щелчка мышью. На странице результатов поиска щелчок мышью обычно означает, что документ является релевантным для конкретного поискового запроса.

Связав события поиска и щелчка мышью с идентификатором корреляции, вы получите более глубокое представление о том, насколько хорошо работает функция поиска приложения.

## <a name="add-search-traffic-analytics"></a>Добавление аналитики поискового трафика

На странице [портала](https://portal.azure.com) для службы когнитивный Поиск Azure откройте страницу Поиск Аналитика трафика, чтобы получить доступ к листу Памятка по для следующего шаблона телеметрии. На этой странице можно выбрать или создать ресурс Application Insights, получить ключ инструментирования, скопировать фрагменты кода, которые можно адаптировать для решения, и загрузить отчет Power BI, созданный на основе схемы, которая отражена в шаблоне.

![Страница "Поиск аналитики трафика" на портале](media/search-traffic-analytics/azuresearch-trafficanalytics.png "Страница "Поиск аналитики трафика" на портале")

## <a name="1---set-up-application-insights"></a>1\. Настройка Application Insights

Выберите или [создайте](../azure-monitor/app/create-new-resource.md) ресурс Application Insights (если у вас его нет). При использовании страницы "Поиск аналитики трафика" можно скопировать ключ инструментирования, который требуется приложению для подключения к Application Insights.

Получив ресурс Application Insights, можно выполнить [инструкции для поддерживаемых языков и платформ](../azure-monitor/app/platforms.md), чтобы зарегистрировать приложение. Регистрация — это просто добавление ключа инструментирования из Application Insights в код, который настраивает связь. Ключ находится на портале или на странице "Поиск аналитики трафика" при выборе существующего ресурса.

Ярлык, который подходит для некоторых типов проектов Visual Studio, представлен на следующих шагах. Он служит для создания ресурса и регистрации приложения всего несколькими щелчками мышью.

1. Для разработки с помощью Visual Studio и ASP.NET откройте решение и выберите **Проект**  >  **Добавить телеметрию Application Insights**.

1. Щелкните **Начать работу**.

1. Зарегистрируйте приложение, предоставив учетную запись Майкрософт, подписку Azure и ресурс Application Insights (по умолчанию используется новый ресурс). Щелкните **Зарегистрировать**.

На этом этапе приложение настроено для мониторинга приложений, что означает, что все загрузки страниц отслеживаются с помощью метрик по умолчанию. Дополнительные сведения о предыдущих шагах см. в разделе [Включение телеметрии Enable Application Insights на стороне сервера](../azure-monitor/app/asp-net-core.md#enable-application-insights-server-side-telemetry-visual-studio).

## <a name="2---add-instrumentation"></a>2\. Добавление инструментирования

На этом шаге выполняется инструментирование собственного приложения поиска с помощью ресурса Application Insights, созданного на предыдущем шаге. Этот процесс состоит из четырех этапов, начиная с создания клиента телеметрии.

### <a name="step-1-create-a-telemetry-client"></a>Шаг 1. Создание клиента телеметрии

Создайте объект, отправляющий события в Application Insights. Можно добавить инструментирование в код приложения на стороне сервера или клиентский код, выполняемый в браузере, который представлен здесь как варианты C# и JavaScript (для других языков см. полный список [поддерживаемых платформ и инфраструктур](../azure-monitor/app/platforms.md)). Выберите подход, который обеспечивает нужную глубину информации.

Данные телеметрии на стороне сервера захватывают метрики на уровне приложения, например в приложениях, выполняющихся в качестве веб-службы в облаке или в качестве локального приложения в корпоративной сети. Данные телеметрии на стороне сервера захватывают события поиска и щелчка мышью, расположение документа в результатах и сведения о запросах, однако сбор данных будет ограничен любой информацией, доступной на этом уровне.

В клиенте может иметься дополнительный код, который управляет вводом запросов, добавляет навигацию или включает контекст (например, запросы, инициированные с домашней страницы и страницы продукта). Если это описание подходит для вашего решения, вы можете выбрать инструментирование на стороне клиента, чтобы в данных телеметрии отражались дополнительные сведения. Сбор этой дополнительной информации выходит за рамки этого шаблона, однако вы можете ознакомиться со статьей [Application Insights для веб-страниц](../azure-monitor/app/javascript.md#explore-browserclient-side-data), чтобы узнать подробнее. 

**Использование C#**

В C# **InstrumentationKey** должен быть определен в конфигурации приложения, например appsettings.json, если проект ASP.NET. Если вы точно не знаете, где расположен ключ, обратитесь к инструкциям по регистрации.

```csharp
private static TelemetryClient _telemetryClient;

// Add a constructor that accepts a telemetry client:
public HomeController(TelemetryClient telemetry)
{
    _telemetryClient = telemetry;
}
```

**Использование JavaScript**

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
    instrumentationKey: "<YOUR INSTRUMENTATION KEY>"
}});
</script>
```

> [!NOTE]
> Для удобства чтения и сокращения возможных ошибок JavaScript все возможные варианты настройки перечислены в новой строке в коде фрагмента кода выше, если вы не хотите изменять значение строки с комментарием, ее можно удалить.


### <a name="step-2-request-a-search-id-for-correlation"></a>Шаг 2. Запрос идентификатора поиска для корреляции

Для корреляции поисковых запросов с щелчками необходим идентификатор корреляции, связывающий эти два различных события. Служба Когнитивного поиска Azure предоставляет идентификатор поиска, когда он запрашивается с заголовком HTTP.

Наличие идентификатора поиска обеспечивает корреляцию метрик, созданных Когнитивным поиском Azure для самого запроса, с настраиваемыми метриками, которые вы регистрируете в Application Insights.

**Использование C# (более новый пакет SDK для версии 11)**

Для установки последнего пакета SDK необходимо использовать конвейер HTTP, чтобы задать заголовок, как описано в этом [примере](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/core/Azure.Core/samples/Pipeline.md#implementing-a-syncronous-policy).

```csharp
// Create a custom policy to add the correct headers
public class SearchIdPipelinePolicy : HttpPipelineSynchronousPolicy
{
    public override void OnSendingRequest(HttpMessage message)
    {
        message.Request.Headers.SetValue("x-ms-azs-return-searchid", "true");
    }
}
```

```csharp
// This sample uses the .NET SDK https://www.nuget.org/packages/Azure.Search.Documents

SearchClientOptions clientOptions = new SearchClientOptions();
clientOptions.AddPolicy(new SearchIdPipelinePolicy(), HttpPipelinePosition.PerCall);

var client = new SearchClient("<SearchServiceName>", "<IndexName>", new AzureKeyCredential("<QueryKey>"), options: clientOptions);

Response<SearchResults<SearchDocument>> response = await client.SearchAsync<SearchDocument>(searchText: searchText, searchOptions: options);
string searchId = string.Empty;
if (response.GetRawResponse().Headers.TryGetValues("x-ms-azs-searchid", out IEnumerable<string> headerValues))
{
    searchId = headerValues.FirstOrDefault();
}
```

**Использование C# (более старый пакет SDK для V10)**

```csharp
// This sample uses the .NET SDK https://www.nuget.org/packages/Microsoft.Azure.Search

var client = new SearchIndexClient(<SearchServiceName>, <IndexName>, new SearchCredentials(<QueryKey>));

// Use HTTP headers so that you can get the search ID from the response
var headers = new Dictionary<string, List<string>>() { { "x-ms-azs-return-searchid", new List<string>() { "true" } } };
var response = await client.Documents.SearchWithHttpMessagesAsync(searchText: searchText, searchParameters: parameters, customHeaders: headers);
string searchId = string.Empty;
if (response.Response.Headers.TryGetValues("x-ms-azs-searchid", out IEnumerable<string> headerValues))
{
    searchId = headerValues.FirstOrDefault();
}
```

**Использование JavaScript (вызов REST API)**

```javascript
request.setRequestHeader("x-ms-azs-return-searchid", "true");
request.setRequestHeader("Access-Control-Expose-Headers", "x-ms-azs-searchid");
var searchId = request.getResponseHeader('x-ms-azs-searchid');
```

### <a name="step-3-log-search-events"></a>Шаг 3. Регистрация событий поиска

Каждый отправляемый поисковой запрос необходимо регистрировать в качестве события поиска, используя следующую схему настраиваемого события Application Insights. Не забывайте регистрировать только те поисковые запросы, которые созданы пользователем.

+ **SearchServiceName**: (строка) имя службы поиска
+ **SearchId**: (GUID) уникальный идентификатор поискового запроса (входит в ответ поиска)
+ **IndexName**: (строка) индекс службы поиска для запроса
+ **QueryTerms**: (строка) слова для поиска, вводимые пользователем
+ **ResultCount**: (целое число) число возвращенных документов (входит в ответ поиска)
+ **ScoringProfile**: (строка) имя используемого профиля оценки, если имеется

> [!NOTE]
> Получите число запросов, создаваемых пользователем, добавив $count=true в поисковый запрос. Дополнительные сведения см. в статье [Поиск документов (REST)](/rest/api/searchservice/search-documents#query-parameters).
>

**Использование C#**

```csharp
var properties = new Dictionary <string, string> 
{
    {"SearchServiceName", <service name>},
    {"SearchId", <search Id>},
    {"IndexName", <index name>},
    {"QueryTerms", <search terms>},
    {"ResultCount", <results count>},
    {"ScoringProfile", <scoring profile used>}
};
_telemetryClient.TrackEvent("Search", properties);
```

**Использование JavaScript**

```javascript
appInsights.trackEvent("Search", {
  SearchServiceName: <service name>,
  SearchId: <search id>,
  IndexName: <index name>,
  QueryTerms: <search terms>,
  ResultCount: <results count>,
  ScoringProfile: <scoring profile used>
});
```

### <a name="step-4-log-click-events"></a>Шаг 4. Регистрация событий щелчка мышью

Каждый щелчок документа — это событие, которое необходимо зарегистрировать для анализа поиска. Используйте пользовательские события Application Insights для регистрации этих событий по следующей схеме:

+ **ServiceName**: (строка) имя службы поиска
+ **SearchId**: (GUID) уникальный идентификатор связанного поискового запроса
+ **DocId**: (строка) идентификатор документа
+ **Position**: (целое число) положение документа на странице результатов поиска

> [!NOTE]
> Позиция ссылается на количественный порядок в приложении. Вы можете задать этот номер для сравнения, при условии, что он всегда один и тот же.
>

**Использование C#**

```csharp
var properties = new Dictionary <string, string> 
{
    {"SearchServiceName", <service name>},
    {"SearchId", <search id>},
    {"ClickedDocId", <clicked document id>},
    {"Rank", <clicked document position>}
};
_telemetryClient.TrackEvent("Click", properties);
```

**Использование JavaScript**

```javascript
appInsights.trackEvent("Click", {
    SearchServiceName: <service name>,
    SearchId: <search id>,
    ClickedDocId: <clicked document id>,
    Rank: <clicked document position>
});
```

## <a name="3---analyze-in-power-bi"></a>3\. Анализ в Power BI

После инструментирования приложения и проверки правильности его подключения к Application Insights можно загрузить предварительно заданный шаблон отчета, чтобы проанализировать данные в Power BI Desktop. В отчете содержатся предварительно заданные диаграммы и таблицы, которые можно использовать при анализе дополнительных данных, полученных для аналитики поискового трафика.

1. В области навигации слева на панели мониторинга Когнитивного поиска Azure в разделе **Параметры** щелкните **Поиск аналитики трафика**.

1. На странице **Поиск аналитики трафика** на шаге 3 щелкните **Получить Power BI Desktop**, чтобы установить Power BI.

   ![Получение отчетов Power BI](./media/search-traffic-analytics/get-use-power-bi.png "Получение отчетов Power BI")

1. На этой же странице щелкните **Скачать отчет Power BI**.

1. В Power BI Desktop откроется отчет, и вам будет предложено подключиться к Application Insights и указать учетные данные. Сведения о подключении представлены на страницах портала Azure для вашего ресурса Application Insights. Для учетных данных укажите те же имя пользователя и пароль, которые используются для входа на портал.

   ![Подключение к Application Insights](./media/search-traffic-analytics/connect-to-app-insights.png "Подключение к Application Insights")

1. Нажмите кнопку **Загрузить**.

Этот отчет содержит диаграммы и таблицы, которые помогут вам принимать более взвешенные решения для улучшения производительности и релевантности поиска.

Метрики содержат следующие элементы.

+ Объем поиска и распространенные пары "условие —документ". Условия, в результате которых щелкают тот же документ, c упорядочиванием по щелчкам.
+ Поисковые запросы без щелчков. Условия основных запросов, для которых не выполнялись щелчки.

На следующем снимке экрана показано, как может выглядеть встроенный отчет, если вы использовали все элементы схемы.

![Панель мониторинга Power BI для Когнитивного поиска Azure](./media/search-traffic-analytics/azuresearch-powerbi-dashboard.png "Панель мониторинга Power BI для Когнитивного поиска Azure")

## <a name="next-steps"></a>Дальнейшие действия

Выполните инструментирование приложения поиска для получения детальных и полезных сведений о службе поиска.

Вы можете узнать больше об [Application Insights](../azure-monitor/app/app-insights-overview.md) и перейти на [страницу цен](https://azure.microsoft.com/pricing/details/application-insights/), чтобы получить дополнительные сведения о соответствующих уровнях служб.

Узнайте больше о создании удивительных отчетов. См. статью [Начало работы с Power BI Desktop](/power-bi/fundamentals/desktop-getting-started).
