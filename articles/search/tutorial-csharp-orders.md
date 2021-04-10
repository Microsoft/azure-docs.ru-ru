---
title: Учебник по C#. Упорядочение результатов
titleSuffix: Azure Cognitive Search
description: В этом учебнике показано, как упорядочить результаты поиска. В его основе лежит имеющийся проект гостиниц с упорядочиванием по первичному и дополнительному свойству, а также профилем повышения для добавления критериев.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 01/26/2021
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 1f8100dd6340383eadec5d10b7f23db59ba0ebdf
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98786390"
---
# <a name="tutorial-order-search-results-using-the-net-sdk"></a>Руководство по Упорядочивание результатов поиска с помощью пакета SDK для .NET

В этой серии руководств результаты возвращались и отображались в [порядке по умолчанию](index-add-scoring-profiles.md#what-is-default-scoring). В этом руководстве вы добавите первичный и вторичный критерии сортировки. Вместо упорядочения по числовым значениям в последнем примере мы покажем, как ранжировать результаты по пользовательскому профилю повышения. Мы также немного изучим _сложные типы_.

В этом руководстве описано следующее:
> [!div class="checklist"]
> * Упорядочение результатов по одному свойству
> * упорядочение результатов по нескольким свойствам;
> * Упорядочение результатов по профилю повышения

## <a name="overview"></a>Обзор

Это руководство расширяет функционал проекта с бесконечной прокруткой, созданного в руководстве [Добавление разбивки по страницам в результаты поиска](tutorial-csharp-paging.md).

Готовую версию кода для этого руководства можно найти в следующем проекте:

* [5-order-results (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/5-order-results)

## <a name="prerequisites"></a>Предварительные требования

* Решение [2b-add-infinite-scroll (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/2b-add-infinite-scroll). Это может быть ваш собственный проект, созданный в рамках предыдущего руководства, или копия с GitHub.

Это руководство было обновлено для использования пакета [Azure.Search.Documents (версия 11)](https://www.nuget.org/packages/Azure.Search.Documents/). Сведения о более ранней версии пакета SDK для .NET приведены в [примере кода для Microsoft.Azure.Search (версия 10)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v10).

## <a name="order-results-based-on-one-property"></a>Упорядочение результатов по одному свойству

При упорядочении результатов по одному свойству, например по рейтингу отелей, нам нужно не только упорядочить результаты, но и убедиться в правильности этого порядка. Добавление поля Rating к результатам позволяет убедиться в правильной сортировке результатов.

В этом упражнении мы добавим в отображение результатов еще несколько элементов: самая низкая и самая высокая цена номеров для каждого отеля.

Для поддержки упорядочения не придется менять модели. Обновления потребуются только для представления и контроллера. Для начала откройте контроллер Home.

### <a name="add-the-orderby-property-to-the-search-parameters"></a>Добавление свойства OrderBy в параметры поиска

1. В HomeController.cs добавьте параметр **OrderBy** и укажите свойство Rating с убывающим порядком сортировки. Добавьте в методе **Index(SearchData model)** следующую строку к параметрам поиска.

    ```cs
    options.OrderBy.Add("Rating desc");
    ```

    >[!Note]
    > По умолчанию упорядочение выполняется по возрастанию, но для наглядности вы можете добавить к свойству ключевое слово `asc`. Порядок по убыванию определяется ключевым словом `desc`.

1. Теперь запустите приложение и введите любое слово, часто используемое в поиске. Результаты будут возвращаться в некотором порядке, правильность которого пока не могут проверить ни сами разработчики, ни пользователи.

1. Давайте сделаем так, чтобы упорядочение по рейтингу стало очевидным в результатах. Во-первых, замените классы **box1** и **box2** в файле hotels.css следующими классами (это новые классы, которые потребуются нам для работы с этим руководством).

    ```html
    textarea.box1A {
        width: 324px;
        height: 32px;
        border: none;
        background-color: azure;
        font-size: 14pt;
        color: blue;
        padding-left: 5px;
        text-align: left;
    }

    textarea.box1B {
        width: 324px;
        height: 32px;
        border: none;
        background-color: azure;
        font-size: 14pt;
        color: blue;
        text-align: right;
        padding-right: 5px;
    }

    textarea.box2A {
        width: 324px;
        height: 32px;
        border: none;
        background-color: azure;
        font-size: 12pt;
        color: blue;
        padding-left: 5px;
        text-align: left;
    }

    textarea.box2B {
        width: 324px;
        height: 32px;
        border: none;
        background-color: azure;
        font-size: 12pt;
        color: blue;
        text-align: right;
        padding-right: 5px;
    }

    textarea.box3 {
        width: 648px;
        height: 100px;
        border: none;
        background-color: azure;
        font-size: 12pt;
        padding-left: 5px;
        margin-bottom: 24px;
    }
    ```

    > [!Tip]
    > Браузеры обычно кэшируют CSS-файлы и это может привести к тому, что будет использоваться старый файл без внесенных изменений. Чтобы обойти эту проблему, мы рекомендуем добавить в ссылку строку запроса с параметром версии. Пример:
    >
    >```html
    >   <link rel="stylesheet" href="~/css/hotels.css?v1.1" />
    >```
    >
    > Обновите номер версии, если вы считаете, что браузер использует старый CSS-файл.

1. Добавьте свойство **Rating** к параметру **Select** в методе **Index(SearchData model)** , чтобы результаты включали следующие три поля:

    ```cs
    options.Select.Add("HotelName");
    options.Select.Add("Description");
    options.Select.Add("Rating");
    ```

1. Откройте представление (index.cshtml) и замените цикл отрисовки (с комментарием **&lt;!-- Show the hotel data. --&gt;** ) следующим кодом.

    ```cs
    <!-- Show the hotel data. -->
    @for (var i = 0; i < result.Count; i++)
    {
        var ratingText = $"Rating: {result[i].Document.Rating}";

        // Display the hotel details.
        @Html.TextArea($"name{i}", result[i].Document.HotelName, new { @class = "box1A" })
        @Html.TextArea($"rating{i}", ratingText, new { @class = "box1B" })
        @Html.TextArea($"desc{i}", fullDescription, new { @class = "box3" })
    }
    ```

1. Рейтинг должен быть доступен как на первой отображаемой странице, так и на всех последующих страницах, которые вызываются через механизм бесконечный прокрутки. Для второй из этих ситуаций необходимо обновить действие **Next** в контроллере и функцию **scrolled** в представлении. Начните изменения с контроллера, заменив метод **Next** следующим кодом. Этот код создает и передает текстовое значение рейтинга.

    ```cs
    public async Task<ActionResult> Next(SearchData model)
    {
        // Set the next page setting, and call the Index(model) action.
        model.paging = "next";
        await Index(model);

        // Create an empty list.
        var nextHotels = new List<string>();

        // Add a hotel details to the list.
        await foreach (var result in model.resultList.GetResultsAsync())
        {
            var ratingText = $"Rating: {result.Document.Rating}";
            var rateText = $"Rates from ${result.Document.cheapest} to ${result.Document.expensive}";
    
            string fullDescription = result.Document.Description;
    
            // Add strings to the list.
            nextHotels.Add(result.Document.HotelName);
            nextHotels.Add(ratingText);
            nextHotels.Add(fullDescription);
        }

        // Rather than return a view, return the list of data.
        return new JsonResult(nextHotels);
    }
    ```

1. Теперь обновите функцию **scrolled** в представлении, чтобы отображать этот текст.

    ```javascript
    <script>
        function scrolled() {
            if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                $.getJSON("/Home/Next", function (data) {
                    var div = document.getElementById('myDiv');

                    // Append the returned data to the current list of hotels.
                    for (var i = 0; i < data.length; i += 3) {
                        div.innerHTML += '\n<textarea class="box1A">' + data[i] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box1B">' + data[i + 1] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box3">' + data[i + 2] + '</textarea>';
                    }
                });
            }
        }
    </script>
    ```

1. Снова запустите приложение. Выполните поиск по любому распространенному слову, например wifi, и убедитесь, что результаты сортируются по убыванию рейтинга отелей.

    ![Упорядочение по рейтингу](./media/tutorial-csharp-create-first-app/azure-search-orders-rating.png)

    Вы можете заметить, что у нескольких отелей значения рейтинга совпадают, и их положение в результатах снова зависит от расположения данных, то есть может считаться произвольным.

    Прежде чем добавить второй уровень упорядочения, давайте добавим код для отображения диапазона стоимости номеров. Этот код позволит нам, во-первых, продемонстрировать извлечение данных из _сложного типа_, а, во-вторых, рассмотреть упорядочение результатов по цене (например, сначала самые дешевые).

### <a name="add-the-range-of-room-rates-to-the-view"></a>Добавление диапазона стоимости номеров в представление

1. Добавьте в модель Hotel.cs свойства, содержащие стоимость самого дешевого и самого дорогого номера.

    ```cs
    // Room rate range
    public double cheapest { get; set; }
    public double expensive { get; set; }
    ```

1. Стоимость номеров мы будем вычислять в конце действия **Index(SearchData model)** в контроллере Home. Разместите эти вычисления после сохранения временных данных.

    ```cs
    // Ensure TempData is stored for the next call.
    TempData["page"] = page;
    TempData["searchfor"] = model.searchText;

    // Calculate the room rate ranges.
    await foreach (var result in model.resultList.GetResultsAsync())
    {
        var cheapest = 0d;
        var expensive = 0d;

        foreach (var room in result.Document.Rooms)
        {
            var rate = room.BaseRate;
            if (rate < cheapest || cheapest == 0)
            {
                cheapest = (double)rate;
            }
            if (rate > expensive)
            {
                expensive = (double)rate;
            }
        }
        model.resultList.Results[n].Document.cheapest = cheapest;
        model.resultList.Results[n].Document.expensive = expensive;
    }
    ```

1. Добавьте свойство **Rooms** в параметр **Select** в методе действия **Index(SearchData model)** в контроллере.

    ```cs
    options.Select.Add("Rooms");
    ```

1. Измените в представлении цикл отрисовки, чтобы отображать диапазон стоимости на первой странице результатов.

    ```cs
    <!-- Show the hotel data. -->
    @for (var i = 0; i < result.Count; i++)
    {
        var rateText = $"Rates from ${result[i].Document.cheapest} to ${result[i].Document.expensive}";
        var ratingText = $"Rating: {result[i].Document.Rating}";

        string fullDescription = result[i].Document.Description;

        // Display the hotel details.
        @Html.TextArea($"name{i}", result[i].Document.HotelName, new { @class = "box1A" })
        @Html.TextArea($"rating{i}", ratingText, new { @class = "box1B" })
        @Html.TextArea($"rates{i}", rateText, new { @class = "box2A" })
        @Html.TextArea($"desc{i}", fullDescription, new { @class = "box3" })
    }
    ```

1. Измените метод **Next** в контроллере Home, чтобы передавать этот диапазон стоимости для последующих страниц результатов.

    ```cs
    public async Task<ActionResult> Next(SearchData model)
    {
        // Set the next page setting, and call the Index(model) action.
        model.paging = "next";
        await Index(model);

        // Create an empty list.
        var nextHotels = new List<string>();

        // Add a hotel details to the list.
        await foreach (var result in model.resultList.GetResultsAsync())
        {
            var ratingText = $"Rating: {result.Document.Rating}";
            var rateText = $"Rates from ${result.Document.cheapest} to ${result.Document.expensive}";

            string fullDescription = result.Document.Description;

            // Add strings to the list.
            nextHotels.Add(result.Document.HotelName);
            nextHotels.Add(ratingText);
            nextHotels.Add(rateText);
            nextHotels.Add(fullDescription);
        }

        // Rather than return a view, return the list of data.
        return new JsonResult(nextHotels);
    }
    ```

1. Обновите функцию **scrolled** в представлении, чтобы обрабатывать текстовое значение диапазона стоимости.

    ```javascript
    <script>
        function scrolled() {
            if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                $.getJSON("/Home/Next", function (data) {
                    var div = document.getElementById('myDiv');

                    // Append the returned data to the current list of hotels.
                    for (var i = 0; i < data.length; i += 4) {
                        div.innerHTML += '\n<textarea class="box1A">' + data[i] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box1B">' + data[i + 1] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box2A">' + data[i + 2] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box3">' + data[i + 4] + '</textarea>';
                    }
                });
            }
        }
    </script>
    ```

1. Запустите приложение и убедитесь, что оно отображает диапазоны стоимости номеров.

    ![Отображение диапазонов стоимости номеров](./media/tutorial-csharp-create-first-app/azure-search-orders-rooms.png)

Свойство **OrderBy** в параметрах поиска не будет принимать такие значения, как **Rooms.BaseRate**, для отображения самой низкой стоимости номеров, даже если номера уже отсортированы по стоимости. В этом случае номера не сортируются по стоимости. Чтобы отображать отели из тестового набора данных в порядке увеличения стоимости номеров, вам придется сортировать результаты в контроллере Home и отправлять их в представление уже в нужном порядке.

## <a name="order-results-based-on-multiple-values"></a>Упорядочение результатов по нескольким значениям

Теперь мы перейдем к тому, как различать отели с одинаковым рейтингом. Одним из подходов может быть вторичное упорядочивание по показателям последнего проведения ремонта в отеле, чтобы отели со свежим ремонтом оказались выше в результатах.

1. Чтобы добавить второй уровень упорядочивания, добавьте **LastRenovationDate** в результаты поиска и в **OrderBy** в методе **Index (SearchData model)** .

    ```cs
    options.Select.Add("LastRenovationDate");

    options.OrderBy.Add("LastRenovationDate desc");
    ```

    >[!Tip]
    > В список **OrderBy** можно включить любое количество свойств. Если у отелей совпадают и рейтинг, и дата последнего ремонта, можно ввести третье свойство различения между этими отелями.

1. Теперь нам понадобится включить в представление еще и дату ремонта, чтобы контролировать правильность порядка. Для сведений о ремонте, пожалуй, будет достаточно отобразить только год. Замените цикл отрисовки в представлении следующим кодом.

    ```cs
    <!-- Show the hotel data. -->
    @for (var i = 0; i < result.Count; i++)
    {
        var rateText = $"Rates from ${result[i].Document.cheapest} to ${result[i].Document.expensive}";
        var lastRenovatedText = $"Last renovated: { result[i].Document.LastRenovationDate.Value.Year}";
        var ratingText = $"Rating: {result[i].Document.Rating}";

        string fullDescription = result[i].Document.Description;

        // Display the hotel details.
        @Html.TextArea($"name{i}", result[i].Document.HotelName, new { @class = "box1A" })
        @Html.TextArea($"rating{i}", ratingText, new { @class = "box1B" })
        @Html.TextArea($"rates{i}", rateText, new { @class = "box2A" })
        @Html.TextArea($"renovation{i}", lastRenovatedText, new { @class = "box2B" })
        @Html.TextArea($"desc{i}", fullDescription, new { @class = "box3" })
    }
    ```

1. Измените метод **Далее** в контроллере Home, чтобы он отправлял значение года из параметра даты последнего ремонта.

    ```cs
        public async Task<ActionResult> Next(SearchData model)
        {
            // Set the next page setting, and call the Index(model) action.
            model.paging = "next";
            await Index(model);

            // Create an empty list.
            var nextHotels = new List<string>();

            // Add a hotel details to the list.
            await foreach (var result in model.resultList.GetResultsAsync())
            {
                var ratingText = $"Rating: {result.Document.Rating}";
                var rateText = $"Rates from ${result.Document.cheapest} to ${result.Document.expensive}";
                var lastRenovatedText = $"Last renovated: {result.Document.LastRenovationDate.Value.Year}";

                string fullDescription = result.Document.Description;

                // Add strings to the list.
                nextHotels.Add(result.Document.HotelName);
                nextHotels.Add(ratingText);
                nextHotels.Add(rateText);
                nextHotels.Add(lastRenovatedText);
                nextHotels.Add(fullDescription);
            }

            // Rather than return a view, return the list of data.
            return new JsonResult(nextHotels);
        }
    ```

1. Обновите в представлении функцию **scrolled**, чтобы отображать этот текст.

    ```javascript
    <script>
        function scrolled() {
            if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                $.getJSON("/Home/Next", function (data) {
                    var div = document.getElementById('myDiv');

                    // Append the returned data to the current list of hotels.
                    for (var i = 0; i < data.length; i += 5) {
                        div.innerHTML += '\n<textarea class="box1A">' + data[i] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box1B">' + data[i + 1] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box2A">' + data[i + 2] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box2B">' + data[i + 3] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box3">' + data[i + 4] + '</textarea>';
                    }
                });
            }
        }
    </script>
    ```

1. Запустите приложение. Выполните поиск по распространенному слову, например pool (бассейн) или view (вид) и убедитесь, что отели с одинаковым рейтингом теперь отображаются в порядке даты последнего ремонта.

    ![Упорядочение по дате ремонта](./media/tutorial-csharp-create-first-app/azure-search-orders-renovation.png)

## <a name="order-results-based-on-a-scoring-profile"></a>Упорядочение результатов по профилю повышения

Пока во всех примерах этого руководства демонстрировалось использование упорядочения по числовым значениям (рейтинг и дата ремонта) с реализацией _точной_ последовательности упорядочения. Но некоторые поисковые запросы для некоторых данных не позволяют выполнить простое сравнение двух элементов. Для запросов полнотекстового поиска Когнитивный поиск реализует концепцию _ранжирования_. Можно указать _профили повышения_, чтобы повлиять на ранжирование результатов, обеспечивая более сложные и качественные сравнения.

[Профили повышения](index-add-scoring-profiles.md) определяются в схеме индекса. Для данных об отелях создано несколько профилей повышения. Давайте рассмотрим, как определяется профиль повышения, а затем напишем код для их применения в поиске.

### <a name="how-scoring-profiles-are-defined"></a>Определение профилей повышения

Профили повышения определяются в индексе поиска во время разработки. Индекс hotels только для чтения, размещаемый корпорацией Майкрософт, имеет три профиля повышения. В этом разделе рассматриваются профили повышения, а также показано, как использовать такие профили повышения в коде.

1. Ниже приведен стандартный профиль повышения для набора данных об отелях, который используется при отсутствии параметров **OrderBy** или **ScoringProfile**. Этот профиль повышает _оценку_ отеля, если искомый текст присутствует в названии отеля, описании или списке тегов (удобств). Обратите внимание, как весовые коэффициенты повышения отдают приоритеты определенным полям. Если искомый текст встречается в другом поле, кроме перечисленных ниже, он получает весовой коэффициент 1. Очевидно, что чем выше оценка, тем выше отображается результат в представлении.

     ```cs
    {
        "name": "boostByField",
        "text": {
            "weights": {
                "Tags": 3,
                "HotelName": 2,
                "Description": 1.5,
                "Description_fr": 1.5,
            }
        }
    }
    ```

1. Следующий альтернативный профиль повышения существенно увеличивает оценку, если переданный параметр включает один или несколько тегов из списка (условно именуемого списком "удобств"). Важной характеристикой этого профиля является то, что параметр является _обязательным_ и должен содержать текстовое значение. Если параметр пуст или не указан, возникает ошибка.

    ```cs
    {
        "name":"boostAmenities",
        "functions":[
            {
            "fieldName":"Tags",
            "freshness":null,
            "interpolation":"linear",
            "magnitude":null,
            "distance":null,
            "tag":{
                "tagsParameter":"amenities"
            },
            "type":"tag",
            "boost":5
            }
        ],
        "functionAggregation":0
    },
    ```

1. В этом третьем профиле рейтинг отеля значительно влияет на оценку. Дата последнего ремонта также повышает оценку, но только если он выполнялся не ранее, чем 730 дней (2 года) назад от текущей даты.

    ```cs
    {
        "name":"renovatedAndHighlyRated",
        "functions":[
            {
            "fieldName":"Rating",
            "freshness":null,
            "interpolation":"linear",
            "magnitude":{
                "boostingRangeStart":0,
                "boostingRangeEnd":5,
                "constantBoostBeyondRange":false
            },
            "distance":null,
            "tag":null,
            "type":"magnitude",
            "boost":20
            },
            {
            "fieldName":"LastRenovationDate",
            "freshness":{
                "boostingDuration":"P730D"
            },
            "interpolation":"quadratic",
            "magnitude":null,
            "distance":null,
            "tag":null,
            "type":"freshness",
            "boost":10
            }
        ],
        "functionAggregation":0
    }
    ```

    Давайте посмотрим, работают ли эти профили так, как задумано.

### <a name="add-code-to-the-view-to-compare-profiles"></a>Добавление кода сравнения профилей в представление

1. Откройте файл index.cshtml и замените раздел &lt;body&gt; следующим кодом.

    ```html
    <body>

    @using (Html.BeginForm("Index", "Home", FormMethod.Post))
    {
        <table>
            <tr>
                <td></td>
                <td>
                    <h1 class="sampleTitle">
                        <img src="~/images/azure-logo.png" width="80" />
                        Hotels Search - Order Results
                    </h1>
                </td>
            </tr>
            <tr>
                <td></td>
                <td>
                    <!-- Display the search text box, with the search icon to the right of it. -->
                    <div class="searchBoxForm">
                        @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox" }) <input class="searchBoxSubmit" type="submit" value="">
                    </div>

                    <div class="searchBoxForm">
                        <b>&nbsp;Order:&nbsp;</b>
                        @Html.RadioButtonFor(m => m.scoring, "Default") Default&nbsp;&nbsp;
                        @Html.RadioButtonFor(m => m.scoring, "RatingRenovation") By numerical Rating&nbsp;&nbsp;
                        @Html.RadioButtonFor(m => m.scoring, "boostAmenities") By Amenities&nbsp;&nbsp;
                        @Html.RadioButtonFor(m => m.scoring, "renovatedAndHighlyRated") By Renovated date/Rating profile&nbsp;&nbsp;
                    </div>
                </td>
            </tr>

            <tr>
                <td valign="top">
                    <div id="facetplace" class="facetchecks">

                        @if (Model != null && Model.facetText != null)
                        {
                            <h5 class="facetheader">Amenities:</h5>
                            <ul class="facetlist">
                                @for (var c = 0; c < Model.facetText.Length; c++)
                                {
                                    <li> @Html.CheckBoxFor(m => m.facetOn[c], new { @id = "check" + c.ToString() }) @Model.facetText[c] </li>
                                }

                            </ul>
                        }
                    </div>
                </td>
                <td>
                    @if (Model != null && Model.resultList != null)
                    {
                        // Show the total result count.
                        <p class="sampleText">
                            @Html.DisplayFor(m => m.resultList.Count) Results <br />
                        </p>

                        <div id="myDiv" style="width: 800px; height: 450px; overflow-y: scroll;" onscroll="scrolled()">

                            <!-- Show the hotel data. -->
                            @for (var i = 0; i < Model.resultList.Results.Count; i++)
                            {
                                var rateText = $"Rates from ${Model.resultList.Results[i].Document.cheapest} to ${Model.resultList.Results[i].Document.expensive}";
                                var lastRenovatedText = $"Last renovated: { Model.resultList.Results[i].Document.LastRenovationDate.Value.Year}";
                                var ratingText = $"Rating: {Model.resultList.Results[i].Document.Rating}";

                                string amenities = string.Join(", ", Model.resultList.Results[i].Document.Tags);
                                string fullDescription = Model.resultList.Results[i].Document.Description;
                                fullDescription += $"\nAmenities: {amenities}";

                                // Display the hotel details.
                                @Html.TextArea($"name{i}", Model.resultList.Results[i].Document.HotelName, new { @class = "box1A" })
                                @Html.TextArea($"rating{i}", ratingText, new { @class = "box1B" })
                                @Html.TextArea($"rates{i}", rateText, new { @class = "box2A" })
                                @Html.TextArea($"renovation{i}", lastRenovatedText, new { @class = "box2B" })
                                @Html.TextArea($"desc{i}", fullDescription, new { @class = "box3" })
                            }
                        </div>

                        <script>
                            function scrolled() {
                                if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                                    $.getJSON("/Home/Next", function (data) {
                                        var div = document.getElementById('myDiv');

                                        // Append the returned data to the current list of hotels.
                                        for (var i = 0; i < data.length; i += 5) {
                                            div.innerHTML += '\n<textarea class="box1A">' + data[i] + '</textarea>';
                                            div.innerHTML += '<textarea class="box1B">' + data[i + 1] + '</textarea>';
                                            div.innerHTML += '\n<textarea class="box2A">' + data[i + 2] + '</textarea>';
                                            div.innerHTML += '<textarea class="box2B">' + data[i + 3] + '</textarea>';
                                            div.innerHTML += '\n<textarea class="box3">' + data[i + 4] + '</textarea>';
                                        }
                                    });
                                }
                            }
                        </script>
                    }
                </td>
            </tr>
        </table>
    }
    </body>
    ```

1. Откройте файл SearchData.cs и замените класс **SearchData** следующим кодом.

    ```cs
    public class SearchData
    {
        public SearchData()
        {
        }

        // Constructor to initialize the list of facets sent from the controller.
        public SearchData(List<string> facets)
        {
            facetText = new string[facets.Count];

            for (int i = 0; i < facets.Count; i++)
            {
                facetText[i] = facets[i];
            }
        }

        // Array to hold the text for each amenity.
        public string[] facetText { get; set; }

        // Array to hold the setting for each amenitity.
        public bool[] facetOn { get; set; }

        // The text to search for.
        public string searchText { get; set; }

        // Record if the next page is requested.
        public string paging { get; set; }

        // The list of results.
        public DocumentSearchResult<Hotel> resultList;

        public string scoring { get; set; }       
    }
    ```

1. Откройте файл hotels.css и добавьте следующие классы HTML.

    ```html
    .facetlist {
        list-style: none;
    }
    
    .facetchecks {
        width: 250px;
        display: normal;
        color: #666;
        margin: 10px;
        padding: 5px;
    }

    .facetheader {
        font-size: 10pt;
        font-weight: bold;
        color: darkgreen;
    }
    ```

### <a name="add-code-to-the-controller-to-specify-a-scoring-profile"></a>Добавление кода с определением профиля оценки в контроллер

1. Откройте файл с контроллером Home. Добавьте следующую инструкцию **using**, которая поможет в создании списков.

    ```cs
    using System.Linq;
    ```

1. В нашем примере первый вызов метода **Index** должен делать немного больше, чем просто вернуть начальное представление. Теперь этот метод извлекает до 20 удобств, которые можно отобразить в представлении.

    ```cs
        public async Task<ActionResult> Index()
        {
            InitSearch();

            // Set up the facets call in the search parameters.
            SearchOptions options = new SearchOptions();
            // Search for up to 20 amenities.
            options.Facets.Add("Tags,count:20");

            SearchResults<Hotel> searchResult = await _searchClient.SearchAsync<Hotel>("*", options);

            // Convert the results to a list that can be displayed in the client.
            List<string> facets = searchResult.Facets["Tags"].Select(x => x.Value.ToString()).ToList();

            // Initiate a model with a list of facets for the first view.
            SearchData model = new SearchData(facets);

            // Save the facet text for the next view.
            SaveFacets(model, false);

            // Render the view including the facets.
            return View(model);
        }
    ```

1. Нам нужны два частных метода, которые сохраняют аспекты во временном хранилище и восстанавливают их из временного хранилища для заполнения модели.

    ```cs
        // Save the facet text to temporary storage, optionally saving the state of the check boxes.
        private void SaveFacets(SearchData model, bool saveChecks = false)
        {
            for (int i = 0; i < model.facetText.Length; i++)
            {
                TempData["facet" + i.ToString()] = model.facetText[i];
                if (saveChecks)
                {
                    TempData["faceton" + i.ToString()] = model.facetOn[i];
                }
            }
            TempData["facetcount"] = model.facetText.Length;
        }

        // Recover the facet text to a model, optionally recoving the state of the check boxes.
        private void RecoverFacets(SearchData model, bool recoverChecks = false)
        {
            // Create arrays of the appropriate length.
            model.facetText = new string[(int)TempData["facetcount"]];
            if (recoverChecks)
            {
                model.facetOn = new bool[(int)TempData["facetcount"]];
            }

            for (int i = 0; i < (int)TempData["facetcount"]; i++)
            {
                model.facetText[i] = TempData["facet" + i.ToString()].ToString();
                if (recoverChecks)
                {
                    model.facetOn[i] = (bool)TempData["faceton" + i.ToString()];
                }
            }
        }
    ```

1. Нам нужно настроить параметры **OrderBy** и **ScoringProfile**. Замените существующий метод **Index(SearchData model)** следующим кодом.

    ```cs
    public async Task<ActionResult> Index(SearchData model)
    {
        try
        {
            InitSearch();

            int page;

            if (model.paging != null && model.paging == "next")
            {
                // Recover the facet text, and the facet check box settings.
                RecoverFacets(model, true);

                // Increment the page.
                page = (int)TempData["page"] + 1;

                // Recover the search text.
                model.searchText = TempData["searchfor"].ToString();
            }
            else
            {
                // First search with text. 
                // Recover the facet text, but ignore the check box settings, and use the current model settings.
                RecoverFacets(model, false);

                // First call. Check for valid text input, and valid scoring profile.
                if (model.searchText == null)
                {
                    model.searchText = "";
                }
                if (model.scoring == null)
                {
                    model.scoring = "Default";
                }
                page = 0;
            }

            // Setup the search parameters.
            var options = new SearchOptions
            {
                SearchMode = SearchMode.All,

                // Skip past results that have already been returned.
                Skip = page * GlobalVariables.ResultsPerPage,

                // Take only the next page worth of results.
                Size = GlobalVariables.ResultsPerPage,

                // Include the total number of results.
                IncludeTotalCount = true,
            };
            // Select the data properties to be returned.
            options.Select.Add("HotelName");
            options.Select.Add("Description");
            options.Select.Add("Tags");
            options.Select.Add("Rooms");
            options.Select.Add("Rating");
            options.Select.Add("LastRenovationDate");

            List<string> parameters = new List<string>();
            // Set the ordering based on the user's radio button selection.
            switch (model.scoring)
            {
                case "RatingRenovation":
                    // Set the ordering/scoring parameters.
                    options.OrderBy.Add("Rating desc");
                    options.OrderBy.Add("LastRenovationDate desc");
                    break;

                case "boostAmenities":
                    {
                        options.ScoringProfile = model.scoring;

                        // Create a string list of amenities that have been clicked.
                        for (int a = 0; a < model.facetOn.Length; a++)
                        {
                            if (model.facetOn[a])
                            {
                                parameters.Add(model.facetText[a]);
                            }
                        }

                        if (parameters.Count > 0)
                        {
                            options.ScoringParameters.Add($"amenities-{ string.Join(',', parameters)}");
                        }
                        else
                        {
                            // No amenities selected, so set profile back to default.
                            options.ScoringProfile = "";
                        }
                    }
                    break;

                case "renovatedAndHighlyRated":
                    options.ScoringProfile = model.scoring;
                    break;

                default:
                    break;
            }

            // For efficiency, the search call should be asynchronous, so use SearchAsync rather than Search.
            model.resultList = await _searchClient.SearchAsync<Hotel>(model.searchText, options);

            // Ensure TempData is stored for the next call.
            TempData["page"] = page;
            TempData["searchfor"] = model.searchText;
            TempData["scoring"] = model.scoring;
            SaveFacets(model, true);

            // Calculate the room rate ranges.
            await foreach (var result in model.resultList.GetResultsAsync())
            {
                var cheapest = 0d;
                var expensive = 0d;

                foreach (var room in result.Document.Rooms)
                {
                    var rate = room.BaseRate;
                    if (rate < cheapest || cheapest == 0)
                    {
                        cheapest = (double)rate;
                    }
                    if (rate > expensive)
                    {
                        expensive = (double)rate;
                    }
                }

                result.Document.cheapest = cheapest;
                result.Document.expensive = expensive;
            }
        }
        catch
        {
            return View("Error", new ErrorViewModel { RequestId = "1" });
        }

        return View("Index", model);
    }
    ```

    Прочтите комментарии для каждого из блоков выбора **switch**.

1. Нам не нужно вносить изменения в действие **Next**, если вы уже выполнили дополнительный код из предыдущего раздела, посвященного упорядочению по нескольким свойствам.

### <a name="run-and-test-the-app"></a>Запуск и тестирование приложения

1. Запустите приложение. Вы увидите в представлении полный набор удобств.

1. Если выбрать для сортировки вариант "By numerical Rating" (По числовому рейтингу), будет использоваться ранее созданное упорядочение по числовым значениям с дополнительным упорядочением по дате ремонта для отелей с одинаковым рейтингом.

   ![Поиск по слову beach с упорядочением по рейтингу](./media/tutorial-csharp-create-first-app/azure-search-orders-beach.png)

1. Теперь попробуйте профиль By amenities (По удобствам). Попробуйте разные наборы удобств и убедитесь, что отели с наличием этих удобств оказываются выше в списке результатов.

   ![Поиск по слову beach с упорядочением по профилю](./media/tutorial-csharp-create-first-app/azure-search-orders-beach-profile.png)

1. Попробуйте вариант By Renovated date/Rating profile (По дате ремонта и профилю рейтинга) и убедитесь, что результаты соответствуют ожиданиям. Только недавно отремонтированные отели должны получать бонус за "актуальность" (_freshness_).

### <a name="resources"></a>Ресурсы

Подробные сведения см. в статье [Add scoring profiles to an Azure Search index](./index-add-scoring-profiles.md) (Добавление профилей повышения в индекс Когнитивного поиска Azure).

## <a name="takeaways"></a>Общие выводы

По результатам этого проекта можно сделать такие выводы:

* Пользователи будут ожидать, что результаты выводятся упорядоченными и первыми отображаются самые важные из них.
* Данные нужно структурировать так, чтобы упорядочение выполнялось очень просто. Мы не смогли легко выполнить сортировку по стоимости номеров, так как структура данных не поддерживает такое упорядочение без использования дополнительного кода.
* Можно использовать много уровней упорядочения, чтобы различать результаты с одинаковыми значениями более высоких уровней упорядочения.
* Для некоторых результатов естественным будет упорядочение по возрастанию (например, расстояние от некоторой точки), а для других — по убыванию (например, оценки гостей).
* Вы можете определить профили повышения, когда числовые параметры для набора данных неприменимы или не дают осмысленных результатов. Оценка каждого результата позволяет более точно упорядочивать и отображать результаты.

## <a name="next-steps"></a>Дальнейшие действия

Вы завершили серию учебников по C#, и получили ценные знания об API-интерфейсах Когнитивного поиска Azure.

Подробные сведения и учебники вы можете найти в библиотеке документации [Microsoft Learn](/learn/browse/?products=azure) или в других разделах [документации по Когнитивному поиску Azure](./index.yml).