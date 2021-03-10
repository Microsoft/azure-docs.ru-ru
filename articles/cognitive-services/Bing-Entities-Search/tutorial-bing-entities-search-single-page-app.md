---
title: Руководство по Одностраничное веб-приложение для службы Поиск сущностей Bing
titleSuffix: Azure Cognitive Services
description: В этом руководстве показано, как использовать API "Поиск сущностей Bing" в одностраничном веб-приложении.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-entity-search
ms.topic: tutorial
ms.date: 03/05/2020
ms.author: aahi
ms.custom: devx-track-js
ms.openlocfilehash: f2c15221268635ca1892a9292d5b0c208c13dd34
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102426813"
---
# <a name="tutorial-single-page-web-app"></a>Руководство по Одностраничное веб-приложение

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

API Bing для поиска сущностей позволяет выполнять в Интернете поиск сведений о *сущностях* и *местах*. В каждом запросе можно запросить один из типов результатов или оба типа. Определения мест и сущностей приводятся ниже.

| Результат | Описание |
|-|-|
|Сущности|Известные люди, места и вещи, которые можно найти по имени|
|Места|Рестораны, отели и другие местные учреждения, которые можно найти по имени *или* по типу (итальянские рестораны)|

В этом руководстве мы создаем одностраничное веб-приложение, использующее API Bing для поиска сущностей для отображения результатов поиска прямо на странице. Приложение включает в себя компоненты HTML, CSS и JavaScript.

API позволяет назначать результатам приоритеты по расположению. В мобильном приложении можно запросить у устройства его собственное расположение. В веб-приложении можно использовать функцию `getPosition()`. Однако этот вызов работает только в защищенных контекстах и он может не предоставлять точное расположение. Кроме того, пользователю может потребоваться поиск сущностей рядом с расположением, отличным от его собственного.

Поэтому наше приложение вызывает службу "Карты Bing" для получения сведений о широте и долготе расположения, указанного пользователем. Затем пользователь может ввести имя какой-то достопримечательности (Space Needle) либо полный или частичный адрес (New York City), и API карт Bing предоставит необходимые координаты.

<!-- Remove until we can replace with a sanitized version.
![[Single-page Bing Entity Search app]](media/entity-search-spa-demo.png)
-->

> [!NOTE]
> При нажатии заголовков JSON и HTTP в нижней части страницы отображаются сведения об ответе JSON и HTTP-запросе. Эти данные позволяют лучше изучить службу.

На примере учебного приложения показано, как выполнить такие задачи:

> [!div class="checklist"]
> * Вызов API Bing для поиска сущностей в JavaScript.
> * Вызов API `locationQuery` карт Bing для в JavaScript.
> * Передача параметров поиска в вызовы API.
> * Отображение результатов поиска.
> * Обработка ключей идентификатора клиента Bing и подписки API.
> * Устранение ошибок, которые могут возникнуть.

Страница руководства является полностью автономной. На ней не используются внешние платформы, таблицы стилей и даже файлы изображений. Она использует только широко распространенные функции языка JavaScript и работает с текущими версиями всех основных браузеров.

В этом руководстве затрагиваются только отдельные части исходного кода. Полный исходный код доступен на [отдельной странице](). Скопируйте и вставьте этот код в текстовый редактор и сохраните его как файл `bing.html`.

> [!NOTE]
> Это руководство в большей части схоже с [руководством по одностраничному веб-приложению для API Bing для поиска в Интернете](../Bing-Web-Search/tutorial-bing-web-search-single-page-app.md), но описывает только результаты поиска сущностей.

## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить задания, описанные в этом руководстве, требуются ключи подписок для API "Поиск Bing" и API "Карты Bing". 

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Получив подписку Azure, сделайте следующее:
  * <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="Создайте ресурс Поиска Bing"  target="_blank">Create a Bing Search resource </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
  * <a href="https://www.microsoft.com/maps/create-a-bing-maps-key.aspx"  title="Создайте ресурс Компьютерного зрения"  target="_blank">Create a Computer Vision resource </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.

## <a name="app-components"></a>Компоненты приложения

Как и любое другое одностраничное веб-приложение, это учебное приложение состоит из трех частей:

> [!div class="checklist"]
> * HTML — определяет структуру и содержимое страницы;
> * CSS — определяет внешний вид страницы;
> * JavaScript — определяет поведение страницы.

В этом руководстве большинство аспектов работы с HTML и CSS не описываются подробно, так как они довольно просты.

HTML-код содержит форму поиска, в которой пользователь вводит запрос и выбирает параметры поиска. Эта форма связана с языком JavaScript, на котором фактически выполняется поиск по атрибуту `onsubmit` тега `<form>`:

```html
<form name="bing" onsubmit="return newBingEntitySearch(this)">
```

Обработчик `onsubmit` возвращает значение `false`, что препятствует отправке формы на сервер. Код JavaScript собирает необходимую информацию из формы и выполняет поиск.

Поиск выполняется в два этапа. Во-первых, если пользователь ввел ограничение расположения, то выполняется запрос "Карт Bing" для преобразования его в координаты. А затем обратный вызов для этого запроса запускает запрос службы "Поиск сущностей Bing".

HTML также содержит разделы (HTML-теги `<div>`), где отображаются результаты поиска.

## <a name="managing-subscription-keys"></a>Управление ключами подписки

> [!NOTE]
> Для этого приложения требуются ключи подписки как для API-интерфейса поиска Bing, так и для API карт Bing.

Чтобы не включать в код ключи подписки для API-интерфейса поиска BingI и API карт Bing, мы используем для их хранения постоянное хранилище браузера. Если какой-то из ключей не был сохранен, мы запрашиваем его и сохраняем для последующего использования. Если на дальнейшем этапе ключ отклоняется API, мы аннулируем сохраненный ключ. В результате пользователю предлагается ввести его при следующем поисковом запросе.

Мы определяем функции `storeValue` и `retrieveValue`, которые используют объект `localStorage` (если браузер поддерживает его) или файл cookie. Наша функция `getSubscriptionKey()` использует эти функции для хранения и извлечения ключа пользователя. Вы можете использовать указанную ниже глобальную конечную точку или конечную точку [пользовательского поддомена](../../cognitive-services/cognitive-services-custom-subdomains.md), отображаемого на портале Azure для вашего ресурса.

```javascript
// cookie names for data we store
SEARCH_API_KEY_COOKIE = "bing-search-api-key";
MAPS_API_KEY_COOKIE   = "bing-maps-api-key";
CLIENT_ID_COOKIE      = "bing-search-client-id";

// API endpoints
SEARCH_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/entities";
MAPS_ENDPOINT   = "https://dev.virtualearth.net/REST/v1/Locations";

// ... omitted definitions of storeValue() and retrieveValue()

// get stored API subscription key, or prompt if it's not found
function getSubscriptionKey(cookie_name, key_length, api_name) {
    var key = retrieveValue(cookie_name);
    while (key.length !== key_length) {
        key = prompt("Enter " + api_name + " API subscription key:", "").trim();
    }
    // always set the cookie in order to update the expiration date
    storeValue(cookie_name, key);
    return key;
}

function getMapsSubscriptionKey() {
    return getSubscriptionKey(MAPS_API_KEY_COOKIE, 64, "Bing Maps");
}

function getSearchSubscriptionKey() {
    return getSubscriptionKey(SEARCH_API_KEY_COOKIE, 32, "Bing Search");
}
```

HTML-тег `<body>` содержит атрибут `onload`, который вызывает функции `getSearchSubscriptionKey()` и `getMapsSubscriptionKey()` после завершения загрузки страницы. Эти вызовы служат для немедленной отправки запроса пользователю ввести свои ключи, если он еще этого не сделал.

```html
<body onload="document.forms.bing.query.focus(); getSearchSubscriptionKey(); getMapsSubscriptionKey();">
```

## <a name="selecting-search-options"></a>Выбор параметров поиска

![[Форма "Поиска сущностей Bing"]](media/entity-search-spa-form.png)

HTML-форма включает в себя следующие элементы управления:

| Control | Описание |
|-|-|
|`where`|Раскрывающееся меню для выбора рынка (расположения и языка), который используется для поиска.|
|`query`|Текстовое поле для ввода условий поиска.|
|`safe`|Флажок, указывающей, включен ли параметр SafeSearch (ограничивает результаты поиска "для взрослых").|
|`what`|Меню для выбора зоны поиска — сущности, места или обе эти зоны.|
|`mapquery`|Текстовое поле, в котором пользователь может указать полный или частичный адрес, название достопримечательности или какое-то другой ориентир, который поможет службе "Поиск сущностей Bing" вернуть более релевантные результаты.|

> [!NOTE]
> В настоящее время результаты по местам доступны только в США. Меню `where` и `what` содержат код, который применяет это ограничение. Если выбрать рынок, отличный от США, но при этом в меню `what` выбрать Places (Места), то значение `what` изменится на Anything (Все). Если выбрать Places (Места), когда рынок, отличный от США, выбран в меню `where`, то значение `where` изменится на US (США).

Наша функция JavaScript `bingSearchOptions()` преобразует эти поля в частичную строку запроса для API поиска Bing.

```javascript
// build query options from the HTML form
function bingSearchOptions(form) {

    var options = [];
    options.push("mkt=" + form.where.value);
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "off"));
    if (form.what.selectedIndex) options.push("responseFilter=" + form.what.value);
    return options.join("&");
}
```

Например, функция SafeSearch может иметь значения `strict`, `moderate` или `off`, где `moderate` используется как значение по умолчанию. Но в нашей форме он представлен флажком, который имеет только два состояния. Код JavaScript преобразует это значение в `strict` или `off` (`moderate` не используется).

Поле `mapquery` не обрабатывается в `bingSearchOptions()`, так как оно используется для запроса расположения в "Картах Bing", а не для "Поиска сущностей Bing".

## <a name="obtaining-a-location"></a>Получение расположения

API карт Bing предлагает [метод `locationQuery`](//msdn.microsoft.com/library/ff701711.aspx), который мы используем для определения широты и долготы расположения, указанного пользователем. Эти координаты затем передаются в API Bing для поиска сущностей с запросом пользователя. В результатах поиска приоритет имеют сущности и места, которые находятся ближе в указанному расположению.

Мы не может получить доступ к API карт Bing, используя в веб-приложении обычный запрос `XMLHttpRequest`, так как служба не поддерживает запросы независимо от источника. Но, к счастью, она поддерживает JSONP (буква P означает "дополненный"). Ответ JSONP — это обычный ответ JSON, заключенный в вызов функции. Запрос выполняется с помощью вставки тега `<script>` в документ. (Политики безопасности браузера не распространяются на загрузку сценариев.)

Функция `bingMapsLocate()` создает и вставляет тег `<script>` для запроса. Сегмент `jsonp=bingMapsCallback` строки запроса указывает имя функции, которая вызывается с ответом.

```javascript
function bingMapsLocate(where) {

    where = where.trim();
    var url = MAPS_ENDPOINT + "?q=" + encodeURIComponent(where) + 
                "&jsonp=bingMapsCallback&maxResults=1&key=" + getMapsSubscriptionKey();

    var script = document.getElementById("bingMapsResult")
    if (script) script.parentElement.removeChild(script);

    // global variable holds reference to timer that will complete the search if the maps query fails
    timer = setTimeout(function() {
        timer = null;
        var form = document.forms.bing;
        bingEntitySearch(form.query.value, "", bingSearchOptions(form), getSearchSubscriptionKey());
    }, 5000);

    script = document.createElement("script");
    script.setAttribute("type", "text/javascript");
    script.setAttribute("id", "bingMapsResult");
    script.setAttribute("src", url);
    script.setAttribute("onerror", "BingMapsCallback(null)");
    document.body.appendChild(script);

    return false;
}
```

> [!NOTE]
> Если API карт Bing не отвечает, то функция `bingMapsCallBack()` никогда не вызывается. Как правило, это означает, что `bingEntitySearch()` не вызывается, а результаты поиска сущностей не отображаются. Для избежания такого сценария функция `bingMapsLocate()` также устанавливает таймер на вызов `bingEntitySearch()` через пять секунд. В функции обратного вызова есть логика, чтобы не выполнять поиск сущностей дважды.

По завершении запроса вызывается функция `bingMapsCallback()`, в соответствии с запросом.

```javascript
function bingMapsCallback(response) {

    if (timer) {    // we beat the timer; stop it from firing
        clearTimeout(timer);
        timer = null;
    } else {        // the timer beat us; don't do anything
        return; 
    }

    var location = "";
    var name = "";
    var radius = 1000;

    if (response) {
        try {
            if (response.statusCode === 401) {
                invalidateMapsKey();
            } else if (response.statusCode === 200) {
                var resource = response.resourceSets[0].resources[0];
                var coords   = resource.point.coordinates;
                name         = resource.name;

                // the radius is the largest of the distances between the location and the corners
                // of its bounding box (in case it's not in the center) with a minimum of 1 km
                try {
                    var bbox    = resource.bbox;
                    radius  = Math.max(haversineDistance(bbox[0], bbox[1], coords[0], coords[1]),
                                       haversineDistance(coords[0], coords[1], bbox[2], bbox[1]),
                                       haversineDistance(bbox[0], bbox[3], coords[0], coords[1]),
                                       haversineDistance(coords[0], coords[1], bbox[2], bbox[3]), 1000);
                } catch(e) {  }
                var location = "lat:" + coords[0] + ";long:" + coords[1] + ";re:" + Math.round(radius);
            }
        }
        catch (e) { }   // response is unexpected. this isn't fatal, so just don't provide location
    }

    var form = document.forms.bing;
    if (name) form.mapquery.value = name;
    bingEntitySearch(form.query.value, location, bingSearchOptions(form), getSearchSubscriptionKey());

}
```

Помимо широты и долготы для запроса службе "Поиск сущностей Bing" требуется *радиус*, указывающий точность сведений о расположении. Для вычисления радиуса мы используем *ограничивающий прямоугольник*, предоставляемый в ответе службы "Карты Bing". Ограничивающий прямоугольник окружает собой все расположение. Например, если пользователь вводит `NYC`, то результат содержит примерные координаты центра города Нью-Йорк и ограничивающий прямоугольник, который охватывает город. 

Сначала мы вычисляем расстояния от начальных координат до каждого из четырех углов ограничивающего прямоугольника с помощью функции `haversineDistance()` (не показано). Самое большое из этих четырех значений мы используем в качестве радиуса. Минимальный радиус равен одному километру. Это значение также используется по умолчанию, если ограничивающий прямоугольник не представлен в ответе.

Получив координаты и радиус, мы вызываем функцию `bingEntitySearch()`, чтобы непосредственно выполнить поиск.

## <a name="performing-the-search"></a>Выполнение поиска

Имея запрос, расположение, строку параметров и ключ API, функция `BingEntitySearch()` выполняет запрос службы "Поиск сущностей Bing".

```javascript
// perform a search given query, location, options string, and API keys
function bingEntitySearch(query, latlong, options, key) {

    // scroll to top of window
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;     // empty query, do nothing

    showDiv("noresults", "Working. Please wait.");
    hideDivs("pole", "mainline", "sidebar", "_json", "_http", "error");

    var request = new XMLHttpRequest();
    var queryurl = SEARCH_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;

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

    if (latlong) request.setRequestHeader("X-Search-Location", latlong);

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

После успешного выполнения HTTP-запроса JavaScript вызывает обработчик событий `load`, функцию `handleBingResponse()`, чтобы передать успешный запрос HTTP GET в API. 

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
            if (jsobj._type === "SearchResponse") {
                renderSearchResults(jsobj);
            } else {
                renderErrorMessage("No search results in JSON response");
            }
        } else {
            renderErrorMessage("Empty response (are you sending too many requests too quickly?)");
        }
    if (divHidden("pole") && divHidden("mainline") && divHidden("sidebar")) 
        showDiv("noresults", "No results.<p><small>Looking for restaurants or other local businesses? Those currently areen't supported outside the US.</small>");
    }

    // Any other HTTP status is an error
    else {
        // 401 is unauthorized; force re-prompt for API key for next request
        if (this.status === 401) invalidateSearchKey();

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
> Успешный HTTP-запрос *не* обязательно означает, что сам поиск был успешным. Если во время выполнения операции поиска возникает ошибка, то API Bing для поиска сущностей возвращает код состояния, отличный от "HTTP: 200", и включает сведения об ошибке в ответ JSON. Кроме того, если запрос имел ограничение по скорости, то API возвращает пустой ответ.

Большая часть кода в обеих описанных выше функциях предназначена для обработки ошибок. Ошибки могут возникать на следующих этапах:

|Этап|Возможные ошибки|Чем обрабатывается|
|-|-|-|
|Создание объекта запроса JavaScript|Недопустимый URL-адрес|Блок `try`/`catch`|
|Выполнение запроса|Ошибки сети, прерванные соединения|Обработчики событий `error` и `abort`|
|Выполнение поиска|Недопустимый запрос, недопустимый JSON-файл, ограничения скорости|Тесты в обработчике событий `load`|

Ошибки обрабатываются путем вызова функции `renderErrorMessage()` со всеми имеющимися сведениями об ошибке. Если ответ передает полный набор тестов ошибок, то мы вызываем функцию `renderSearchResults()` для отображения результатов поиска на странице.

## <a name="displaying-search-results"></a>Отображение результатов поиска

Для API Bing для поиска сущностей [требуется, чтобы результаты отображались в указанном порядке](../bing-web-search/use-display-requirements.md). Так как API может возвращать два типа ответа, недостаточно выполнить итерацию через коллекцию `Entities` или `Places` верхнего уровня в ответе JSON и отобразить полученные результаты. (Если необходим только один тип результата, то используйте параметр запроса `responseFilter`.)

Вместо этого мы используем коллекцию `rankingResponse` в результатах поиска, чтобы упорядочить результаты для отображения. Этот объект ссылается на элементы в коллекциях `Entitiess` и (или) `Places`.

`rankingResponse` может содержать до трех коллекций результатов поиска, назначенных как `pole`, `mainline` и `sidebar`. 

`pole` (если есть) — это самый релевантный результат поиска, и его необходимо отображать в приоритетном порядке. `mainline` ссылается на основную часть результатов поиска. Результаты mainline необходимо отображать сразу после результатов `pole` (или первыми, если `pole` отсутствуют). 

И наконец. `sidebar` ссылается на вспомогательные результаты поиска. Они могут отображаться на боковой панели или просто после основной (mainline) части результатов. Для нашего учебного приложения мы выбрали второй вариант.

Каждый элемент в коллекции `rankingResponse` ссылается на фактические элементы результатов поиска двумя разными, но похожими способами.

| Item | Описание |
|-|-|
|`id`|Элемент `id` выглядит как URL-адрес, но он не должен использоваться для ссылок. Тип `id` результата ранжирования совпадает с `id` элемента результатов поиска в коллекции ответов *или* со всей коллекцией ответов (например `Entities`).
|`answerType`<br>`resultIndex`|`answerType` ссылается на коллекцию ответов верхнего уровня, содержащую результат (например `Entities`). `resultIndex` ссылается на индекс результата в этой коллекции. Если `resultIndex` отсутствует, то результат ранжирования ссылается на всю коллекцию.

> [!NOTE]
> Дополнительные сведения об этой части ответа поиска см. в статье [Using ranking to display results](rank-results.md) (Использование рейтинга для отображения результатов).

Вы можете использовать любой метод расстановки элемента в результатах поиска, который наилучшим образом подходит для вашего приложения. В примере кода в этом руководстве мы используем `answerType` и `resultIndex` для расстановки каждого результата поиска.

Вот и пришло время взглянуть на нашу функцию `renderSearchResults()`. Эта функция выполняет итерацию по трем коллекциям `rankingResponse`, которые предоставляют три раздела результатов поиска. Для каждого раздела мы вызываем функцию `renderResultsItems()`, чтобы преобразовать результаты для просмотра для этого раздела.

```javascript
// render the search results given the parsed JSON response
function renderSearchResults(results) {

    // if spelling was corrected, update search field
    if (results.queryContext.alteredQuery) 
        document.forms.bing.query.value = results.queryContext.alteredQuery;

    // for each possible section, render the results from that section
    for (section in {pole: 0, mainline: 0, sidebar: 0}) {
        if (results.rankingResponse[section])
            showDiv(section, renderResultsItems(section, results));
    }
}
```

## <a name="rendering-result-items"></a>Отображение элементов результата

В нашем коде JavaScript есть объект `searchItemRenderers`, который содержит *отрисовщики* — функции, создающие HTML-код для каждого типа результатов поиска.

```javascript
searchItemRenderers = { 
    entities: function(item) { ... },
    places: function(item) { ... }
}
```

Функция-отрисовщик может принимать следующие параметры:

| Параметр | Описание |
|-|-|
|`item`|Объект JavaScript, содержащий свойства элемента, такие как URL-адрес и его описание.|
|`index`|Индекс элемента результата в коллекции.|
|`count`|Число элементов в коллекции результатов поиска.|

Параметры `index` и `count` можно использовать для нумерации результатов, создания специального HTML-кода для начала или конца коллекции, вставки разрывов строки после определенного числа элементов и т. д. Если от отрисовщика не требуется выполнять эту функцию, то ему не нужно принимать эти два параметра. Фактически, мы не используем их в обработчиках для нашего учебного приложения.

Давайте рассмотрим отрисовщик `entities` подробнее:

```javascript
    entities: function(item) {
        var html = [];
        html.push("<p class='entity'>");
        if (item.image) {
            var img = item.image;
            if (img.hostPageUrl) html.push("<a href='" + img.hostPageUrl + "'>");
            html.push("<img src='" + img.thumbnailUrl +  "' title='" + img.name + "' height=" + img.height + " width= " + img.width + ">");
            if (img.hostPageUrl) html.push("</a>");
            if (img.provider) {
                var provider = img.provider[0];
                html.push("<small>Image from ");
                if (provider.url) html.push("<a href='" + provider.url + "'>");
                html.push(provider.name ? provider.name : getHost(provider.url));
                if (provider.url) html.push("</a>");
                html.push("</small>");
            }
        }
        html.push("<p>");
        if (item.entityPresentationInfo) {
            var pi = item.entityPresentationInfo;
            if (pi.entityTypeHints || pi.entityTypeDisplayHint) {
                html.push("<i>");
                if (pi.entityTypeDisplayHint) html.push(pi.entityTypeDisplayHint);
                else if (pi.entityTypeHints) html.push(pi.entityTypeHints.join("/"));
                html.push("</i> - ");
            }
        }
        html.push(item.description);
        if (item.webSearchUrl) html.push("&nbsp;<a href='" + item.webSearchUrl + "'>More</a>")
        if (item.contractualRules) {
            html.push("<p><small>");
            var rules = [];
            for (var i = 0; i < item.contractualRules.length; i++) {
                var rule = item.contractualRules[i];
                var link = [];
                if (rule.license) rule = rule.license;
                if (rule.url) link.push("<a href='" + rule.url + "'>");
                link.push(rule.name || rule.text || rule.targetPropertyName + " source");
                if (rule.url) link.push("</a>");
                rules.push(link.join(""));
            }
            html.push("License: " + rules.join(" - "));
            html.push("</small>");
        }
        return html.join("");
    }, // places renderer omitted
```

Наша функция-обработчик сущностей:

> [!div class="checklist"]
> * Создает HTML-тег `<img>` для отображения эскиза изображения (если есть). 
> * Создает HTML-тег `<a>`, который связывает со страницей, содержащей это изображение.
> * создает описание со сведениями об этом изображении и о сайте, на котором оно размещено.
> * Включает классификацию сущности, используя отображение подсказок (если есть).
> * Включает ссылку на поиск Bing для получения дополнительных сведений о сущности.
> * Отображает любые сведения о лицензировании или атрибутах, которые требуются для источников данных.

## <a name="persisting-client-id"></a>Сохранение идентификатора клиента

Ответы от интерфейсов API поиска Bing могут содержать заголовок `X-MSEdge-ClientID`, который необходимо отправить обратно в API с последующими запросами. Если используется несколько API-интерфейсов поиска Bing, то по возможности со всеми ими необходимо использовать один идентификатор клиента.

Наличие заголовка `X-MSEdge-ClientID` позволяет интерфейсам API Bing связывать все поисковые запросы пользователя, а это дает два важных преимущества.

Во-первых, поисковая система Bing может применять к операциям поиска прошлый контекст и находить результаты, которые лучше соответствуют требованиям пользователя. Если пользователь ранее уже вводил поисковые запросы, например, связанные с мореплаванием под парусом, то при последующем поиске по слову "docks" в приоритетном порядке могут возвращаться сведения о местах для стоянки парусных лодок.

Во-вторых, Bing может случайным образом выбирать пользователей для опробования новых возможностей, прежде чем делать их общедоступными. Предоставление одного и того же идентификатора клиента при каждом запросе гарантирует, что пользователи, которым был открыт доступ к определенному компоненту, всегда будут иметь к нему доступ. Без идентификатора клиента такой компонент может появляться и исчезать в результатах поиска пользователя как бы случайным образом.

Из-за политик безопасности браузера (CORS) заголовок `X-MSEdge-ClientID` может быть недоступным для кода JavaScript. Это ограничение возникает, когда ответ поиска и страница, его запросившая, имеют разные источники. В рабочей среде такая проблема с политикой решается путем размещения серверного скрипта, который выполняет вызов API, на одном домене с веб-страницей. Так как скрипт будет иметь тот же источник, что и веб-страница, заголовок `X-MSEdge-ClientID` станет доступным для JavaScript.

> [!NOTE]
> В рабочем веб-приложении запрос следует всегда выполнять на стороне сервера. В противном случае вам придется включать в веб-страницу ключ API поиска Bing, где он будет доступен для всех, кто просматривает исходный код. С вас будет взиматься плата за любое использование ресурсов в рамках вашего ключа подписки API, включая запросы, выполненные неавторизованными сторонами, поэтому важно не предоставлять доступ к своему ключу.

В целях разработки запрос к API Bing для поиска в Интернете можно выполнить через прокси-сервер CORS. Ответ от прокси-сервера содержит заголовок `Access-Control-Expose-Headers`, который позволяет перечислять заголовки ответов и делает их доступными для JavaScript.

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
> [Справочник по API Bing для поиска сущностей](//docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference)

> [!div class="nextstepaction"]
> [Документация по API карт Bing](//msdn.microsoft.com/library/dd877180.aspx)