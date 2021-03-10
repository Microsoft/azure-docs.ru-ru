---
title: Руководство по Создание одностраничного веб-приложения с помощью API Bing для поиска в Интернете
titleSuffix: Azure Cognitive Services
description: Это одностраничное приложение показывает, как можно использовать API Bing для поиска в Интернете, чтобы извлекать, анализировать и отображать актуальные результаты поиска в одностраничном приложении.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: tutorial
ms.date: 03/05/2020
ms.author: aahi
ms.custom: devx-track-js
ms.openlocfilehash: ca542ac6c5184a0307a9293fada305a6c6356edd
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102434174"
---
# <a name="tutorial-create-a-single-page-app-using-the-bing-web-search-api"></a>Руководство по Создание одностраничного веб-приложения с помощью API Bing для поиска в Интернете

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Это одностраничное приложение показывает, как можно извлекать, анализировать и отображать актуальные результаты поиска из API Bing для поиска в Интернете. fВ этом руководстве используется стандартный HTML и CSS и оно ориентировано на код JavaScript. HTML, CSS и JS-файлы доступны на [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/tree/master/Tutorials/Bing-Web-Search) вместе с кратким руководством.

Этот пример приложения может выполнять такие задачи:

> [!div class="checklist"]
> * Вызов API Bing для поиска в Интернете с параметрами поиска.
> * Отображение результатов веб-поиска, а также результатов поиска новостей, видео и изображений.
> * Разбивка результатов на страницы.
> * Управление ключами подписки
> * Обработка ошибок

Чтобы использовать это приложение, требуется [учетная запись Azure Cognitive Services](../cognitive-services-apis-create-account.md) с API-интерфейсами поиска Bing.

## <a name="prerequisites"></a>Предварительные требования

Для запуска приложения необходимы следующие элементы:

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Получив подписку Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="создайте ресурс Поиска Bing"  target="_blank">Create a Bing Search resource </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.

* Node.js 8 или более поздняя версия.

Первым шагом является клонирование репозитория с исходным кодом примера приложения.

```console
git clone https://github.com/Azure-Samples/cognitive-services-REST-api-samples.git
```

Затем выполните `npm install`. В этом руководстве единственной зависимостью является Express.js.

```console
cd <path-to-repo>/cognitive-services-REST-api-samples/Tutorials/Bing-Web-Search
npm install
```

## <a name="app-components"></a>Компоненты приложения

Пример приложения, который мы создаем, состоит из четырех частей.

* `bing-web-search.js` — приложение Express.js. Оно обрабатывает логику запроса и ответа и маршрутизацию.
* `public/index.html` — структура приложения. Она определяет способ представления данных пользователю.
* `public/css/styles.css` — определяет стили страницы, такие как шрифты, цвет, размер текста.
* `public/js/scripts.js` — содержит логику для выполнения запросов к API Bing для поиска в Интернете, управления ключами подписки, обработки и анализа ответов, а также логику отображения результатов.

В этом руководстве рассматривается `scripts.js` и логика, необходимая для вызова API Bing для поиска в Интернете и обработки ответов.

## <a name="html-form"></a>HTML-формы

`index.html` включает форму, которая позволяет выполнять поиск и выбирать параметры поиска. Атрибут `onsubmit` активируется при отправке формы, вызывая метод `bingWebSearch()`, определенный в `scripts.js`. Он принимает три аргумента:

* Поисковый запрос
* Выбранные параметры
* Ключ подписки

```html
<form name="bing" onsubmit="return bingWebSearch(this.query.value,
    bingSearchOptions(this), getSubscriptionKey())">
```

## <a name="query-options"></a>Параметры запроса

HTML-форма содержит параметры, которые соответствуют параметрам запросов в [API Bing для поиска в Интернете версии 7](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#query-parameters). В этой таблице содержатся подробные сведения о том, как пользователи могут фильтровать результаты поиска с помощью примера приложения:

| Параметр | Описание |
|-----------|-------------|
| `query` | Текстовое поле для ввода строки запроса. |
| `where` | Раскрывающееся меню для выбора рынка (язык и страна). |
| `what` | Флажки для повышения уровня определенных типов результатов. Например, повышение уровня изображений увеличивает их ранжирование в результатах поиска. |
| `when` | Раскрывающееся меню, которое позволяет ограничить результаты поиска на сегодня, на этой неделе и в этом месяце. |
| `safe` | Флажок для включения безопасного поиска Bing, который отфильтровывает материалы для взрослых. |
| `count` | Скрытое поле. Количество результатов поиска для каждого запроса. Измените значение, чтобы на странице отображалось меньше или больше результатов. |
| `offset` | Скрытое поле. Смещение первого результата поиска в запросе. Используется для разбиения по страницам. Сбрасывается к `0` при каждом новом запросе. |

> [!NOTE]
> API Bing для поиска в Интернете предоставляет дополнительные параметры запроса, чтобы уточнить результаты поиска. В этом примере используются всего несколько из них. Полный список доступных параметров см. в [этом разделе](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#query-parameters).

Функция `bingSearchOptions()` преобразует эти параметры, чтобы они соответствовали формату, требуемому API поиска Bing.

```javascript
// Build query options from selections in the HTML form.
function bingSearchOptions(form) {

    var options = [];
    // Where option.
    options.push("mkt=" + form.where.value);
    // SafeSearch option.
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "moderate"));
    // Freshness option.
    if (form.when.value.length) options.push("freshness=" + form.when.value);
    var what = [];
    for (var i = 0; i < form.what.length; i++)
        if (form.what[i].checked) what.push(form.what[i].value);
    // Promote option.
    if (what.length) {
        options.push("promote=" + what.join(","));
        options.push("answerCount=9");
    }
    // Count option.
    options.push("count=" + form.count.value);
    // Offset option.
    options.push("offset=" + form.offset.value);
    // Hardcoded text decoration option.
    options.push("textDecorations=true");
    // Hardcoded text format option.
    options.push("textFormat=HTML");
    return options.join("&");
}
```

Параметру `SafeSearch` можно присвоить значение `strict`, `moderate` или `off`. `moderate` — значение по умолчанию для службы "Поиск в Интернете Bing". В этой форме используется флажок, у которого есть два состояния: `strict` или `moderate`.

Если выбраны какие-либо флажки **повышения уровня**, то параметр `answerCount` также добавится к запросу. `answerCount` необходим при использовании параметра `promote`. В этом фрагменте кода установлено значение `9` для возврата всех доступных типов результатов.
> [!NOTE]
> Повышение уровня типа результата не *гарантирует*, что он будет включен в результаты поиска. Оно, скорее всего, повышает оценку этого типа результатов по сравнению со стандартной. Чтобы ограничить поиск конкретными видами результатов, используйте параметр запроса `responseFilter` или вызовите более конкретную конечную точку, например API Bing для поиска изображений или API Bing для поиска новостей.

Параметры запроса `textDecoration` и `textFormat` жестко заданы в сценарии, чтобы поисковой запрос выделялся жирным шрифтом в результатах поиска. Эти параметры необязательные.

## <a name="manage-subscription-keys"></a>Управление ключами подписки

Чтобы не включать в код ключ подписки для API Поиска Bing, в этом примере приложения для его хранения используется постоянное хранилище браузера. Если ключ подписки не сохранен, пользователю предлагается ввести его. Если он отклоняется API-интерфейсом, пользователю предлагается повторно ввести ключ подписки.

Функция `getSubscriptionKey()` использует функции `storeValue` и `retrieveValue` для хранения и извлечения пользовательского ключа подписки. Эти функции используют объект `localStorage`, если он поддерживается, или файлы cookie.

```javascript
// Cookie names for stored data.
API_KEY_COOKIE   = "bing-search-api-key";
CLIENT_ID_COOKIE = "bing-search-client-id";

BING_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/search";

// See source code for storeValue and retrieveValue definitions.

// Get stored subscription key, or prompt if it isn't found.
function getSubscriptionKey() {
    var key = retrieveValue(API_KEY_COOKIE);
    while (key.length !== 32) {
        key = prompt("Enter Bing Search API subscription key:", "").trim();
    }
    // Always set the cookie in order to update the expiration date.
    storeValue(API_KEY_COOKIE, key);
    return key;
}
```

Как было показано ранее, при отправке формы активируется `onsubmit`, вызывая функцию `bingWebSearch`. Она инициализирует и отправляет запрос. `getSubscriptionKey` вызывается для аутентификации запроса при каждой отправке.

## <a name="call-bing-web-search"></a>Вызов Поиска в Интернете Bing

Учитывая запрос, строку параметров и ключ подписки, функция `BingWebSearch` создает объект `XMLHttpRequest`, чтобы вызвать конечную точку службы "Поиск в Интернете Bing".

```javascript
// Perform a search constructed from the query, options, and subscription key.
function bingWebSearch(query, options, key) {
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;

    showDiv("noresults", "Working. Please wait.");
    hideDivs("pole", "mainline", "sidebar", "_json", "_http", "paging1", "paging2", "error");

    var request = new XMLHttpRequest();
    var queryurl = BING_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;

    // Initialize the request.
    try {
        request.open("GET", queryurl);
    }
    catch (e) {
        renderErrorMessage("Bad request (invalid URL)\n" + queryurl);
        return false;
    }

    // Add request headers.
    request.setRequestHeader("Ocp-Apim-Subscription-Key", key);
    request.setRequestHeader("Accept", "application/json");
    var clientid = retrieveValue(CLIENT_ID_COOKIE);
    if (clientid) request.setRequestHeader("X-MSEdge-ClientID", clientid);

    // Event handler for successful response.
    request.addEventListener("load", handleBingResponse);

    // Event handler for errors.
    request.addEventListener("error", function() {
        renderErrorMessage("Error completing request");
    });

    // Event handler for an aborted request.
    request.addEventListener("abort", function() {
        renderErrorMessage("Request aborted");
    });

    // Send the request.
    request.send();
    return false;
}
```

После успешного запроса активируется обработчик событий `load` и вызывает функцию `handleBingResponse`. `handleBingResponse` анализирует объект результата, отображает результаты и содержит логику ошибок для невыполненных запросов.

```javascript
function handleBingResponse() {
    hideDivs("noresults");

    var json = this.responseText.trim();
    var jsobj = {};

    // Try to parse results object.
    try {
        if (json.length) jsobj = JSON.parse(json);
    } catch(e) {
        renderErrorMessage("Invalid JSON response");
        return;
    }

    // Show raw JSON and the HTTP request.
    showDiv("json", preFormat(JSON.stringify(jsobj, null, 2)));
    showDiv("http", preFormat("GET " + this.responseURL + "\n\nStatus: " + this.status + " " +
        this.statusText + "\n" + this.getAllResponseHeaders()));

    // If the HTTP response is 200 OK, try to render the results.
    if (this.status === 200) {
        var clientid = this.getResponseHeader("X-MSEdge-ClientID");
        if (clientid) retrieveValue(CLIENT_ID_COOKIE, clientid);
        if (json.length) {
            if (jsobj._type === "SearchResponse" && "rankingResponse" in jsobj) {
                renderSearchResults(jsobj);
            } else {
                renderErrorMessage("No search results in JSON response");
            }
        } else {
            renderErrorMessage("Empty response (are you sending too many requests too quickly?)");
        }
    }

    // Any other HTTP response is considered an error.
    else {
        // 401 is unauthorized; force a re-prompt for the user's subscription
        // key on the next request.
        if (this.status === 401) invalidateSubscriptionKey();

        // Some error responses don't have a top-level errors object, if absent
        // create one.
        var errors = jsobj.errors || [jsobj];
        var errmsg = [];

        // Display the HTTP status code.
        errmsg.push("HTTP Status " + this.status + " " + this.statusText + "\n");

        // Add all fields from all error responses.
        for (var i = 0; i < errors.length; i++) {
            if (i) errmsg.push("\n");
            for (var k in errors[i]) errmsg.push(k + ": " + errors[i][k]);
        }

        // Display Bing Trace ID if it isn't blocked by CORS.
        var traceid = this.getResponseHeader("BingAPIs-TraceId");
        if (traceid) errmsg.push("\nTrace ID " + traceid);

        // Display the error message.
        renderErrorMessage(errmsg.join("\n"));
    }
}
```

> [!IMPORTANT]
> Успешный HTTP-запрос *не* означает, что сам поиск был успешным. Если во время выполнения операции поиска возникает ошибка, то API Bing для поиска в Интернете возвращает код состояния, отличный от "HTTP: 200", и включает сведения об ошибке в ответ JSON. Если у запроса было ограничение по скорости, то API возвращает пустой ответ.

Большая часть кода в обеих описанных выше функциях предназначена для обработки ошибок. Ошибки могут возникать на следующих этапах:

| Этап | Возможные ошибки | Чем обрабатывается |
|-------|--------------------|------------|
| Создание объекта запроса | Недопустимый URL-адрес | Блок `try` / `catch` |
| Выполнение запроса | Ошибки сети, прерванные соединения | Обработчики событий `error` и `abort` |
| Выполнение поиска | Недопустимый запрос, недопустимый JSON-файл, ограничения скорости | Тестируется в обработчике событий `load` |

Ошибки обрабатываются путем вызова `renderErrorMessage()`. Если ответ проходит все проверки на ошибки, вызывается `renderSearchResults()` для отображения результатов поиска.

## <a name="display-search-results"></a>Отображение результатов поиска.

Существуют [требования к использованию и отображению](./use-display-requirements.md) для результатов, возвращаемых API Bing для поиска в Интернете. Так как ответ может содержать различные типы результатов, то недостаточно выполнить итерацию коллекции верхнего уровня `WebPages`. Вместо этого пример приложения использует `RankingResponse` для упорядочения результатов по спецификациям.

> [!NOTE]
> Если требуется только один тип результата, используйте параметр запроса `responseFilter` или стоит воспользоваться другой конечной точкой Поиска Bing, например Поиском изображений Bing.

Каждый ответ содержит объект `RankingResponse`, который может содержать до трех коллекций: `pole`, `mainline` и `sidebar`. `pole` (если есть) — это самый релевантный результат поиска, и он должен отображаться в приоритетном порядке. `mainline` содержит большую часть результатов поиска и отобразится сразу после `pole`. `sidebar` включает вспомогательные результаты поиска. Если возможно, эти результаты должны отображаться на боковой панели. Если из-за ограничений экрана использование боковой панели становится нецелесообразным, эти результаты должны располагаться после результатов `mainline`.

Каждый объект `RankingResponse` включает массив `RankingItem`, определяющий, как должны быть упорядочены результаты. Наш пример приложения использует параметры `answerType` и `resultIndex` для идентификации результатов.

> [!NOTE]
> Существуют дополнительные способы идентификации и отображения результатов. Дополнительные сведения см. в статье [Использование ранжирования для отображения ответов](rank-results.md).

Рассмотрим элементы кода:

```javascript
// Render the search results from the JSON response.
function renderSearchResults(results) {

    // If spelling was corrected, update the search field.
    if (results.queryContext.alteredQuery)
        document.forms.bing.query.value = results.queryContext.alteredQuery;

    // Add Prev / Next links with result count.
    var pagingLinks = renderPagingLinks(results);
    showDiv("paging1", pagingLinks);
    showDiv("paging2", pagingLinks);

    // Render the results for each section.
    for (section in {pole: 0, mainline: 0, sidebar: 0}) {
        if (results.rankingResponse[section])
            showDiv(section, renderResultsItems(section, results));
    }
}
```

Функция `renderResultsItems()` выполняет перебор элементов в каждой коллекции `RankingResponse`, сопоставляет каждый результат ранжирования с результатом поиска с использованием значений `answerType` и `resultIndex` и вызывает соответствующую функцию рендеринга для создания HTML-кода. Если `resultIndex` для данного элемента не указан, `renderResultsItems()` выполняет итерацию по всем результатам этого типа и вызывает функцию рендеринга для каждого элемента. Полученный HTML вставляется в соответствующий элемент `<div>` в `index.html`.

```javascript
// Render search results from the RankingResponse object per rank response and
// use and display requirements.
function renderResultsItems(section, results) {

    var items = results.rankingResponse[section].items;
    var html = [];
    for (var i = 0; i < items.length; i++) {
        var item = items[i];
        // Collection name has lowercase first letter while answerType has uppercase
        // e.g. `WebPages` RankingResult type is in the `webPages` top-level collection.
        var type = item.answerType[0].toLowerCase() + item.answerType.slice(1);
        if (type in results && type in searchItemRenderers) {
            var render = searchItemRenderers[type];
            // This ranking item refers to ONE result of the specified type.
            if ("resultIndex" in item) {
                html.push(render(results[type].value[item.resultIndex], section));
            // This ranking item refers to ALL results of the specified type.
            } else {
                var len = results[type].value.length;
                for (var j = 0; j < len; j++) {
                    html.push(render(results[type].value[j], section, j, len));
                }
            }
        }
    }
    return html.join("\n\n");
}
```

## <a name="review-renderer-functions"></a>Просмотр функций отрисовщика

В нашем примере объект `searchItemRenderers` включает функции, которые создают HTML-код для каждого типа результатов поиска.

```javascript
// Render functions for each result type.
searchItemRenderers = {
    webPages: function(item) { ... },
    news: function(item) { ... },
    images: function(item, section, index, count) { ... },
    videos: function(item, section, index, count) { ... },
    relatedSearches: function(item, section, index, count) { ... }
}
```

> [!IMPORTANT]
> В примере приложения есть отрисовщики для веб-страниц, новостей, изображений, видео и связанных с ними поисковых запросов. Вашему приложению нужны отрисовщики для любого вида полученных результатов, которые могут включать вычисления, предложения орфографии, сущности, часовые пояса и определения.

Некоторые функции рендеринга принимают только параметр `item`. Другие принимают дополнительные параметры, которые могут использоваться для преобразования элементов для просмотра по-разному в разных контекстах. Отрисовщик, который не использует эту информацию, не должен принимать эти параметры.

Контекстные аргументы:

| Параметр  | Описание |
|------------|-------------|
| `section` | Раздел результатов (`pole`, `mainline` или `sidebar`), в котором содержится элемент. |
| `index`<br>`count` | Доступен, когда элемент `RankingResponse` указывает, что все результаты в данной коллекции должны отображаться. В противном случае `undefined`. Это индекс элемента и общее число элементов коллекции. Вы можете использовать эту информацию для подсчета результатов, для создания разного HTML-кода для первого или последнего результата и т. д. |

В нашем примере приложения отрисовщики `images` и `relatedSearches` используют контекстные аргументы для настройки создаваемого HTML-кода. Давайте рассмотрим отрисовщик `images` подробнее:

```javascript
searchItemRenderers = {
    // Render image result with thumbnail.
    images: function(item, section, index, count) {
        var height = 60;
        var width = Math.round(height * item.thumbnail.width / item.thumbnail.height);
        var html = [];
        if (section === "sidebar") {
            if (index) html.push("<br>");
        } else {
            if (!index) html.push("<p class='images'>");
        }
        html.push("<a href='" + item.hostPageUrl + "'>");
        var title = escape(item.name) + "\n" + getHost(item.hostPageDisplayUrl);
        html.push("<img src='"+ item.thumbnailUrl + "&h=" + height + "&w=" + width +
            "' height=" + height + " width=" + width + " title='" + title + "' alt='" + title + "'>");
        html.push("</a>");
        return html.join("");
    },
    // Other renderers are omitted from this sample...
}
```

Отрисовщик изображений:

* Вычисляет размер эскиза изображения (ширина изменяется, а высота фиксируется на 60 пикселях).
* Вставляет HTML-код, предшествующий результату изображения на основе контекста.
* Создает HTML-тег `<a>`, который связывает со страницей, содержащей это изображение.
* создает HTML-тег `<img>` для отображения эскиза изображения;

Отрисовщик изображений использует переменные `section` и `index`, чтобы отобразить результаты по-разному, в зависимости от того, где они отображаются. Разрыв строки (тег `<br>`) вставляется между результатами изображения на боковой панели, чтобы в боковой панели изображения отображались в столбце. В других разделах первый результат изображения `(index === 0)` предшествует тегу `<p>`.

Размер эскиза указывается в теге `<img>` и в полях `h` и `w` URL-адреса эскиза. Атрибуты `title` и `alt` (текстовое описание изображения) формируются на основе имени изображения и имени узла в URL-адресе.

Ниже приведен пример отображения изображений в примере приложения:

![[Результаты изображения Bing]](media/cognitive-services-bing-web-api/web-search-spa-images.png)

## <a name="persist-the-client-id"></a>Сохранение идентификатора клиента

Ответы от интерфейсов API для поиска Bing могут содержать заголовок `X-MSEdge-ClientID`, который необходимо отправить обратно в API с каждым последующим запросом. Если приложение использует несколько API для поиска Bing, убедитесь, что с каждым запросом в службах отправляется одинаковый идентификатор клиента.

Наличие заголовка `X-MSEdge-ClientID` позволяет API-интерфейсам Bing связывать поисковые запросы пользователя. Во-первых, поисковая система Bing может применять к операциям поиска прошлый контекст и находить результаты, которые лучше соответствуют требованиям запроса. Если пользователь ранее уже вводил поисковые запросы, например, связанные с мореплаванием под парусом, то при последующем поиске по слову "морские узлы" в приоритетном порядке могут возвращаться сведения об узлах, которые используются в мореплавании. Во-вторых, Bing может случайным образом выбирать пользователей для опробования новых возможностей, прежде чем делать их общедоступными. Предоставление одного и того же идентификатора клиента при каждом запросе гарантирует, что пользователи, которым был открыт доступ к определенному компоненту, всегда будут его видеть. Без идентификатора клиента такой компонент может появляться и исчезать в результатах поиска пользователя как бы случайным образом.

Политики безопасности браузера, такие как "Общий доступ к ресурсам независимо от источника" (CORS), могут запретить доступ примеру приложения к заголовку `X-MSEdge-ClientID`. Это ограничение возникает, когда ответ поиска и страница, его запросившая, имеют разные источники. В рабочей среде такая проблема с политикой решается путем размещения серверного скрипта, который выполняет вызов API, на одном домене с веб-страницей. Так как скрипт будет иметь тот же источник, что и веб-страница, заголовок `X-MSEdge-ClientID` станет доступным для JavaScript.

> [!NOTE]
> В рабочем веб-приложении запрос следует всегда выполнять на стороне сервера. В противном случае вам придется включать в веб-страницу ключ подписки API для поиска Bing, где он будет доступен для всех, кто просматривает исходный код. С вас будет взиматься плата за любое использование ресурсов в рамках вашего ключа подписки API, включая запросы, выполненные неавторизованными сторонами, поэтому важно не предоставлять доступ к своему ключу.

В целях разработки запрос можно выполнить через прокси-сервер CORS. Ответ от прокси-сервера такого типа содержит заголовок `Access-Control-Expose-Headers`, который позволяет отфильтровать заголовки ответов и делает их доступными для JavaScript.

Установить прокси-сервер CORS довольно просто. Это позволит нашему примеру приложения получить доступ к заголовку идентификатора клиента. Выполните следующую команду:

```console
npm install -g cors-proxy-server
```

После этого измените конечную точку службы "Поиск в Интернете Bing" в `script.js` на следующую:

```javascript
http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/search
```

Запустите прокси-сервер CORS с помощью следующей команды:

```console
cors-proxy-server
```

Не закрывайте командное окно, пока используете пример приложения. Если закрыть окно, это приведет к остановке прокси-сервера. В развертываемом разделе заголовков HTTP под результатами поиска можно увидеть заголовок `X-MSEdge-ClientID`. Убедитесь, что он одинаковый для всех запросов.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Справка по API Bing для поиска в Интернете версии 7](//docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference)