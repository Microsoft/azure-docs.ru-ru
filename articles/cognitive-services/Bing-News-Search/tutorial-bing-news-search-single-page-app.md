---
title: Руководство по Создание одностраничного веб-приложения с использованием API Поиска новостей Bing
titleSuffix: Azure Cognitive Services
description: Из этого руководства вы узнаете, как создать одностраничное веб-приложение, которое может отправлять поисковые запросы к API Bing для поиска новостей и отображать результаты на веб-странице.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-news-search
ms.topic: tutorial
ms.date: 06/23/2020
ms.author: aahi
ms.custom: seodec2018, devx-track-js
ms.openlocfilehash: cb9e68efd27deb3bf66d3c286c0cd7a128d8bf59
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102430587"
---
# <a name="tutorial-create-a-single-page-web-app"></a>Руководство по Создание одностраничного веб-приложения

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

API Bing для поиска новостей позволяет выполнять поиск новостей и получать результаты различных типов, релевантных поисковому запросу. В этом руководстве мы создаем одностраничное веб-приложение, использующее API Bing для поиска новостей для отображения результатов поиска прямо на странице. Приложение включает в себя компоненты HTML, CSS и JavaScript. Исходный код этого примера доступен на [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/Tutorials/BingNewsSearchApp.html).

<!-- Remove until we can replace it with sanitized copy
![Single-page Bing News Search app](media/news-search-singlepage.png)
-->

> [!NOTE]
> При нажатии заголовков JSON и HTTP в нижней части страницы отображаются сведения об ответе JSON и HTTP-запросе. Эти данные позволяют лучше изучить службу.

На примере учебного приложения показано, как выполнить такие задачи:
> [!div class="checklist"]
> * Вызов API Bing для поиска новостей из JavaScript.
> * Передача параметров поиска в API Bing для поиска новостей.
> * Отображение результатов поиска новостей по четырем категориям: любой тип, деловые, здравоохранение или политика; за определенные периоды: 24 часа, за прошедшую неделю, месяц или за любое время.
> * Переход по страницам результатов поиска.
> * Обработка идентификатора клиента Bing и ключа подписки API.
> * Обработка ошибок, которые могут возникнуть.

Страница руководства является полностью автономной; на ней не используется никаких внешних платформ, таблиц стилей и даже файлов изображений. Она использует только широко распространенные функции языка JavaScript и работает с текущими версиями всех основных браузеров.


## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить задания, описанные в этом руководстве, требуются ключи подписок для API "Поиск Bing". Если у вас их нет, вам потребуется следующее:

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Получив подписку Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="создайте ресурс Поиска Bing"  target="_blank">Create a Bing Search resource </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.

## <a name="app-components"></a>Компоненты приложения
Как и любое другое одностраничное веб-приложение, это учебное приложение состоит из трех частей:

> [!div class="checklist"]
> * HTML — определяет структуру и содержимое страницы;
> * CSS — определяет внешний вид страницы;
> * JavaScript — определяет поведение страницы.

Большинство компонентов HTML и CSS являются обычными, поэтому в руководстве это не обсуждается. HTML-код содержит форму поиска, в которой пользователь вводит запрос и выбирает параметры поиска. Форма связана с JavaScript, который выполняет поиск, используя атрибут `onsubmit` тега `<form>`:

```html
<form name="bing" onsubmit="return newBingNewsSearch(this)">
```
Обработчик `onsubmit` возвращает значение `false`, что препятствует отправке формы на сервер. Код JavaScript выполняет сбор необходимой информации из формы и выполняет поиск.

HTML также содержит разделы (HTML-теги `<div>`), где отображаются результаты поиска.

## <a name="managing-subscription-key"></a>Управление ключом подписки

Чтобы не включать ключ подписки для API поиска Bing прямо в код, мы используем для его хранения постоянное хранилище браузера. Перед сохранением ключа мы запрашиваем ключ пользователя. Если на дальнейшем этапе ключ отклоняется API, мы аннулируем сохраненный ключ и снова выводим запрос пользователю.

Мы определяем функции `storeValue` и `retrieveValue`, которые используют объект `localStorage` (если браузер поддерживает его) или файл cookie. Наша функция `getSubscriptionKey()` использует эти функции для хранения и извлечения ключа пользователя. Вы можете использовать указанную ниже глобальную конечную точку или конечную точку [пользовательского поддомена](../../cognitive-services/cognitive-services-custom-subdomains.md), отображаемого на портале Azure для вашего ресурса.

``` javascript
// Cookie names for data we store
API_KEY_COOKIE   = "bing-search-api-key";
CLIENT_ID_COOKIE = "bing-search-client-id";

// Bing Search API endpoint
BING_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/news";

// ... omitted definitions of storeValue() and retrieveValue()
// Browsers differ in their support for persistent storage by 
// local HTML files. See the source code for browser-specific
// options.

// Get stored API subscription key, or 
// prompt if it's not found.
function getSubscriptionKey() {
    var key = retrieveValue(API_KEY_COOKIE);
    while (key.length !== 32) {
        key = prompt("Enter Bing Search API subscription key:", "").trim();
    }
    // always set the cookie in order to update the expiration date
    storeValue(API_KEY_COOKIE, key);
    return key;
}
```
В теге HTML `<form>` атрибут `onsubmit` вызывает функцию `bingWebSearch`, чтобы возвращать результаты поиска. Функция `bingWebSearch` использует `getSubscriptionKey()` для аутентификации каждого запроса. Как показано в предыдущем определении, `getSubscriptionKey` запрашивает у пользователя ключ, если он еще не был введен. Затем ключ сохраняется для последующего использования приложением.

```html
<form name="bing" onsubmit="this.offset.value = 0; return bingWebSearch(this.query.value, 
    bingSearchOptions(this), getSubscriptionKey())">
```
## <a name="selecting-search-options"></a>Выбор параметров поиска
На следующем изображении показано текстовое поле для запроса и параметры, которые определяют поиск новостей о финансировании школ.

![Параметры Bing для поиска новостей](media/news-search-categories.png)

HTML-форма включает элементы со следующими именами:

|Элемент|Описание|
|-|-|
| `where` | Раскрывающееся меню для выбора рынка (расположения и языка), который используется для поиска. |
| `query` | Текстовое поле для ввода условий поиска. |
| `category` | Флажки для повышения уровня определенных видов результатов. Например, повышение уровня здравоохранения повышает положение новостей о здравоохранении в списке результатов. |
| `when` | Раскрывающееся меню для ограничения поиска результатами за последний день, неделю или месяц (необязательный параметр). |
| `safe` | Флажок, указывающий, следует ли использовать функцию безопасного поиска Bing для фильтрации результатов с материалами для взрослых. |
| `count` | Скрытое поле. Количество результатов поиска для каждого запроса. Измените значение, чтобы на странице отображалось меньше или больше результатов. |
| `offset`|  Скрытое поле. Смещение первого результата поиска в запросе. Используется для разбиения по страницам. Сбрасывается на `0` при новом запросе. |

> [!NOTE]
> Поиск в Интернете Bing поддерживает дополнительные параметры запроса. Мы используем только некоторые из них.

``` javascript
// build query options from the HTML form
function bingSearchOptions(form) {

    var options = [];
    options.push("mkt=" + form.where.value);
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "off"));
    if (form.when.value.length) options.push("freshness=" + form.when.value);

    for (var i = 0; i < form.category.length; i++) {
        if (form.category[i].checked) {
            category = form.category[i].value;
            break;
        }
    }
    if (category.valueOf() != "all".valueOf()) { 
        options.push("category=" + category); 
        }
    options.push("count=" + form.count.value);
    options.push("offset=" + form.offset.value);
    return options.join("&");
}
```

Например, параметр `SafeSearch` в реальном вызове API может иметь значение `strict`, `moderate` или `off`, при этом `moderate` является значением по умолчанию. Тем не менее в форме используется флажок, который имеет только два состояния. Код JavaScript преобразует это значение в `strict` или `off` (`moderate` не используется).

## <a name="performing-the-request"></a>Выполнение запроса
Получив на вход запрос, строку параметров и ключ API, функция `BingNewsSearch` применяет объект `XMLHttpRequest` для выполнения запроса к конечной точке Поиска новостей Bing.

```javascript
// perform a search given query, options string, and API key
function bingNewsSearch(query, options, key) {

    // scroll to top of window
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;     // empty query, do nothing

    showDiv("noresults", "Working. Please wait.");
    hideDivs("results", "related", "_json", "_http", "paging1", "paging2", "error");

    var request = new XMLHttpRequest();
     if (category.valueOf() != "all".valueOf()) {
        var queryurl = BING_ENDPOINT + "/search?" + "?q=" + encodeURIComponent(query) + "&" + options;
    }
    else
    {
        if (query){
        var queryurl = BING_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;
        }
        else {
            var queryurl = BING_ENDPOINT + "?" + options;
        }
    }

    // open the request
    try {
        request.open("GET", queryurl);
    } 
    catch (e) {
        renderErrorMessage("Bad request (invalid URL)\n" + queryurl);
        return false;
    }

    // add request headers
    request.setRequestHeader("Ocp-Apim-Subscription-Key", key);
    request.setRequestHeader("Accept", "application/json");
    var clientid = retrieveValue(CLIENT_ID_COOKIE);
    if (clientid) request.setRequestHeader("X-MSEdge-ClientID", clientid);

    // event handler for successful response
    request.addEventListener("load", handleBingResponse);

    // event handler for erorrs
    request.addEventListener("error", function() {
        renderErrorMessage("Error completing request");
    });

    // event handler for aborted request
    request.addEventListener("abort", function() {
        renderErrorMessage("Request aborted");
    });

    // send the request
    request.send();
    return false;
}
```
После успешного выполнения HTTP-запроса JavaScript вызывает обработчик событий `load`, функцию `handleBingResponse()`, чтобы предать успешный запрос HTTP GET в API. 

```javascript
// handle Bing search request results
function handleBingResponse() {
    hideDivs("noresults");

    var json = this.responseText.trim();
    var jsobj = {};

    // try to parse JSON results
    try {
        if (json.length) jsobj = JSON.parse(json);
    } catch(e) {
        renderErrorMessage("Invalid JSON response");
    }

    // show raw JSON and HTTP request
    showDiv("json", preFormat(JSON.stringify(jsobj, null, 2)));
    showDiv("http", preFormat("GET " + this.responseURL + "\n\nStatus: " + this.status + " " + 
        this.statusText + "\n" + this.getAllResponseHeaders()));

    // if HTTP response is 200 OK, try to render search results
    if (this.status === 200) {
        var clientid = this.getResponseHeader("X-MSEdge-ClientID");
        if (clientid) retrieveValue(CLIENT_ID_COOKIE, clientid);
        if (json.length) {
            if (jsobj._type === "News") {
                renderSearchResults(jsobj);
            } else {
                renderErrorMessage("No search results in JSON response");
            }
        } else {
            renderErrorMessage("Empty response (are you sending too many requests too quickly?)");
        }
    }

    // Any other HTTP response is an error
    else {
        // 401 is unauthorized; force re-prompt for API key for next request
        if (this.status === 401) invalidateSubscriptionKey();

        // some error responses don't have a top-level errors object, so gin one up
        var errors = jsobj.errors || [jsobj];
        var errmsg = [];

        // display HTTP status code
        errmsg.push("HTTP Status " + this.status + " " + this.statusText + "\n");

        // add all fields from all error responses
        for (var i = 0; i < errors.length; i++) {
            if (i) errmsg.push("\n");
            for (var k in errors[i]) errmsg.push(k + ": " + errors[i][k]);
        }

        // also display Bing Trace ID if it isn't blocked by CORS
        var traceid = this.getResponseHeader("BingAPIs-TraceId");
        if (traceid) errmsg.push("\nTrace ID " + traceid);

        // and display the error message
        renderErrorMessage(errmsg.join("\n"));
    }
}
```

> [!IMPORTANT]
> Успешный HTTP-запрос *не* обязательно означает, что сам поиск был успешным. Если во время выполнения операции поиска возникает ошибка, то API Bing для поиска новостей возвращает код состояния, отличный от "HTTP: 200", и включает сведения об ошибке в ответ JSON. Кроме того, если запрос имел ограничение по скорости, то API возвращает пустой ответ.

Большая часть кода в обеих описанных выше функциях предназначена для обработки ошибок. Ошибки могут возникать на следующих этапах:

|Этап|Возможные ошибки|Чем обрабатывается|
|-|-|-|
|Построение объекта запроса JavaScript|Недопустимый URL-адрес|Блок `try`/`catch`|
|Выполнение запроса|Ошибки сети, прерванные соединения|Обработчики событий `error` и `abort`|
|Выполнение поиска|Недопустимый запрос, недопустимый JSON-файл, ограничения скорости|Тесты в обработчике событий `load`|

Ошибки обрабатываются путем вызова функции `renderErrorMessage()` со всеми имеющимися сведениями об ошибке. Если ответ передает полный набор тестов ошибок, то мы вызываем функцию `renderSearchResults()` для отображения результатов поиска на странице.

## <a name="displaying-search-results"></a>Отображение результатов поиска
Основная функция отображения результатов поиска — `renderSearchResults()`. Эта функция принимает JSON, возвращенный службой поиска новостей Bing, и отображает результаты новостей и связанные с ними поисковые запросы, если они есть.

```javascript
// render the search results given the parsed JSON response
    function renderSearchResults(results) {

    // add Prev / Next links with result count
    var pagingLinks = renderPagingLinks(results);
    showDiv("paging1", pagingLinks);
    showDiv("paging2", pagingLinks);

    showDiv("results", renderResults(results.value));
    if (results.relatedSearches)
        showDiv("sidebar", renderRelatedItems(results.relatedSearches));
}
```
Основные результаты поиска возвращаются как объекты верхнего уровня `value` в ответе JSON. Мы передаем их в нашу функцию `renderResults()`, которая поочередно вызывает для них функцию отображения в формате HTML. Полученный HTML-код возвращается в `renderSearchResults()`, где он вставляется в раздел `results` на странице.

```javascript
function renderResults(items) {
    var len = items.length;
    var html = [];
    if (!len) {
        showDiv("noresults", "No results.");
        hideDivs("paging1", "paging2");
        return "";
    }
    for (var i = 0; i < len; i++) {
        html.push(searchItemRenderers.news(items[i], i, len));
    }
    return html.join("\n\n");
}
```
API Bing для поиска новостей возвращает до четырех различных видов связанных результатов, каждый в своем собственном объекте верхнего уровня. К ним относятся:

|Тип связи|Описание|
|-|-|
|`pivotSuggestions`|Запросы, которые заменяют сводное слово в исходном поиске другим. Например, если вы ищете "красные цветы", сводным словом может быть "красные", а сводным предложением может быть "желтые цветы".|
|`queryExpansions`|Запросы, которые сужают исходный поиск, добавляя больше условий. Например, если вы ищете "Microsoft Surface", расширением запроса может быть "Microsoft Surface Pro".|
|`relatedSearches`|Запросы, которые вводились другими пользователями, использовавшими ранее исходный поиск. Например, если вы ищете "гора Рейнир", может применяться связанный поиск "гора Святой Елены".|
|`similarTerms`|Запросы, похожие по смыслу на исходный запрос. Например, если вы ищете "школы", аналогичным термином может быть "образование".|

Как ранее было показано в `renderSearchResults()`, мы преобразовываем для просмотра только предложения `relatedItems` и размещаем результирующие ссылки на боковой панели страницы.

## <a name="rendering-result-items"></a>Отображение элементов результата

В коде JavaScript есть объект `searchItemRenderers`, который содержит функции-*обработчики*, создающие HTML-код для каждого типа результатов поиска.

```javascript
searchItemRenderers = {
    news: function(item) { ... },
    webPages: function (item) { ... }, 
    images: function(item, index, count) { ... },
    relatedSearches: function(item) { ... }
}
```
Функция-обработчик может принимать следующие параметры:

|Параметр|Описание|
|-|-|
|`item`| Объект JavaScript, содержащий свойства элемента, такие как URL-адрес и его описание.|
|`index`| Индекс элемента результата в коллекции.|
|`count`| Число элементов в коллекции результатов поиска.|

Параметры `index` и `count` можно использовать для нумерации результатов, создания специального HTML-кода для начала или конца коллекции, вставки разрывов строки после определенного числа элементов и т. д. Если от отрисовщика не требуется выполнять эту функцию, то ему не нужно принимать эти два параметра.

Отрисовщик `news` показан в следующем фрагменте JavaScript:
```javascript
    // render news story
    news: function (item) {
        var html = [];
        html.push("<p class='news'>");
        if (item.image) {
            width = 60;
            height = Math.round(width * item.image.thumbnail.height / item.image.thumbnail.width);
            html.push("<img src='" + item.image.thumbnail.contentUrl +
                "&h=" + height + "&w=" + width + "' width=" + width + " height=" + height + ">");
        }
        html.push("<a href='" + item.url + "'>" + item.name + "</a>");
        if (item.category) html.push(" - " + item.category);
        if (item.contractualRules) {    // MUST display source attributions
            html.push(" (");
            var rules = [];
            for (var i = 0; i < item.contractualRules.length; i++)
                rules.push(item.contractualRules[i].text);
                html.push(rules.join(", "));
                html.push(")");
            }
        html.push(" (" + getHost(item.url) + ")");
        html.push("<br>" + item.description);
        return html.join("");
    },
```
Функция отрисовщика новостей выполняет следующее.
> [!div class="checklist"]
> * Создает тег параграфа, назначает его классу `news` ​​и передает его в массив html.
> * Вычисляет размер эскиза изображения (с фиксированной шириной 60 пикселей, высота рассчитывается пропорционально).
> * создает HTML-тег `<img>` для отображения эскиза изображения; 
> * создает HTML-теги `<a>` со ссылкой на изображение и страницей, которая его содержит;
> * создает описание со сведениями об этом изображении и о сайте, на котором оно размещено.

Размер эскиза указывается в теге `<img>` и в полях `h` и `w` URL-адреса эскиза. Затем [служба эскизов Bing](../bing-web-search/resize-and-crop-thumbnails.md) предоставляет эскиз точно такого размера.

## <a name="persisting-client-id"></a>Сохранение идентификатора клиента
Ответы от интерфейсов API поиска Bing могут содержать заголовок `X-MSEdge-ClientID`, который необходимо отправить обратно в API с последующими запросами. Если используется несколько API-интерфейсов поиска Bing, то по возможности со всеми ими необходимо использовать один идентификатор клиента.

Наличие заголовка `X-MSEdge-ClientID` позволяет API-интерфейсам Bing связывать все поисковые запросы пользователя, а это дает два важных преимущества.

Во-первых, поисковая система Bing может применять к операциям поиска прошлый контекст и находить результаты, которые лучше соответствуют требованиям пользователя. Если пользователь ранее уже вводил поисковые запросы, например, связанные с мореплаванием под парусом, то при последующем поиске по слову "морские узлы" в приоритетном порядке могут возвращаться сведения об узлах, которые используются в мореплавании.

Во-вторых, Bing может случайным образом выбирать пользователей для опробования новых возможностей, прежде чем делать их общедоступными. Предоставление одинакового идентификатора клиента при каждом запросе гарантирует, что пользователи, которым был открыт доступ к определенному компоненту, всегда будут иметь к нему доступ. Без идентификатора клиента такой компонент может появляться и исчезать в результатах поиска пользователя как бы случайным образом.

Из-за политик безопасности браузера (CORS) заголовок `X-MSEdge-ClientID` может быть недоступным для кода JavaScript. Это ограничение возникает, когда ответ поиска и страница, его запросившая, имеют разные источники. В рабочей среде такая проблема с политикой решается путем размещения серверного скрипта, который выполняет вызов API, на одном домене с веб-страницей. Так как скрипт будет иметь тот же источник, что и веб-страница, заголовок `X-MSEdge-ClientID` станет доступным для JavaScript.

> [!NOTE]
> В рабочем веб-приложении запрос следует выполнять на стороне сервера. В противном случае вам придется включать в веб-страницу ключ API поиска Bing, где он будет доступен для всех, кто просматривает исходный код. С вас будет взиматься плата за любое использование ресурсов в рамках вашего ключа подписки API, включая запросы, выполненные неавторизованными сторонами, поэтому важно не предоставлять доступ к своему ключу.

В целях разработки запрос к API Bing для поиска в Интернете можно выполнить через прокси-сервер CORS. Ответ от прокси-сервера содержит заголовок `Access-Control-Expose-Headers`, который поддерживает заголовки ответов и делает их доступными для JavaScript.

Установить прокси-сервер CORS довольно просто. Это позволит нашему учебному приложению иметь доступ к заголовку идентификатора клиента. Для начала [установите платформу Node.js](https://nodejs.org/en/download/), если она еще не установлена. Затем введите следующую команду в командном окне:

```console
npm install -g cors-proxy-server
```

После этого в HTML-файле измените конечную точку службы "Поиск в Интернете Bing" на следующую:\
`http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/search`

И наконец, запустите прокси-сервер CORS с помощью следующей команды:

```console
cors-proxy-server
```

Не закрывайте командное окно, пока используете учебное приложение. Это приведет к остановке прокси-сервера. Теперь в развертываемом разделе заголовков HTTP под результатами поиска можно увидеть заголовок `X-MSEdge-ClientID` (среди прочих) и убедиться, что он одинаковый для всех запросов.

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Справочник по API Bing для поиска новостей](//docs.microsoft.com/rest/api/cognitiveservices/bing-news-api-v7-reference)