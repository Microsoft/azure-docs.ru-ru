---
title: Учебник по C#. Использование аспектов для улучшения навигации
titleSuffix: Azure Cognitive Search
description: После разбиения результатов на страницы вы можете добавить фасетную навигацию. Узнайте, как с помощью аспектов можно легко сузить поиск.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 01/26/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 44a05ed5b4386f2787a1c84dfeb61a5d539c7cb5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98789808"
---
# <a name="tutorial-add-faceted-navigation-using-the-net-sdk"></a>Руководство по Добавление фасетной навигации с помощью пакета SDK для .NET

Аспекты обеспечивают самостоятельную навигацию, предоставляя набор ссылок для фильтрации результатов. В этом учебнике структура навигации с аспектами размещается в левой части страницы с метками и гиперактивного текста.

В этом руководстве описано следующее:
> [!div class="checklist"]
> * Пометка свойств модели как _IsFacetable_.
> * Добавление аспектной навигации в приложение

## <a name="overview"></a>Обзор

Аспекты основаны на полях в индексе поиска. Запрос, включающий аспект = [строка], предоставляет поле для аспекта. Запрос может содержать несколько аспектов, таких как `&facet=category&facet=amenities`, которые разделяются символом "амперсанд" (&). Для реализации структуры навигации с аспектами необходимо указать и аспекты, и фильтры. Фильтр используется для события щелчка, чтобы уменьшить результаты. Например, если щелкнуть "бюджет", результаты фильтруются на основе этих критериев.

Этот учебник служит расширением проекта разбивки на страницы, созданного в учебнике [Добавление разбивки по страницам в результаты поиска](tutorial-csharp-paging.md).

Готовую версию кода для этого руководства можно найти в следующем проекте:

* [4-add-facet-navigation (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/4-add-facet-navigation)

## <a name="prerequisites"></a>Предварительные требования

* Решение [2a-add-paging (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/2a-add-paging). Это может быть ваш собственный проект, созданный в рамках предыдущего руководства, или копия с GitHub.

Это руководство было обновлено для использования пакета [Azure.Search.Documents (версия 11)](https://www.nuget.org/packages/Azure.Search.Documents/). Сведения о более ранней версии пакета SDK для .NET приведены в [примере кода для Microsoft.Azure.Search (версия 10)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v10).

## <a name="set-model-properties-as-isfacetable"></a>Пометка свойств модели как IsFacetable

Чтобы свойство модели находилось в аспектном поиске, оно должно быть помечено как **IsFacetable**.

1. Изучите класс **Hotel**. Например, свойства **Category** и **Tags** помечены как **IsFacetable**, а **HotelName** и **Description** — нет. 

    ```cs
    public partial class Hotel
    {
        [SimpleField(IsFilterable = true, IsKey = true)]
        public string HotelId { get; set; }

        [SearchableField(IsSortable = true)]
        public string HotelName { get; set; }

        [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.EnLucene)]
        public string Description { get; set; }

        [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.FrLucene)]
        [JsonPropertyName("Description_fr")]
        public string DescriptionFr { get; set; }

        [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
        public string Category { get; set; }

        [SearchableField(IsFilterable = true, IsFacetable = true)]
        public string[] Tags { get; set; }

        [SimpleField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
        public bool? ParkingIncluded { get; set; }

        [SimpleField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
        public DateTimeOffset? LastRenovationDate { get; set; }

        [SimpleField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
        public double? Rating { get; set; }

        public Address Address { get; set; }

        [SimpleField(IsFilterable = true, IsSortable = true)]
        public GeographyPoint Location { get; set; }

        public Room[] Rooms { get; set; }
    }
    ```

1. Мы не будем изменять какие-либо теги в этом учебнике, поэтому закройте файл hotel.cs без изменений.

    > [!Note]
    > Аспектный поиск выдает ошибку, если поле, запрошенное при поиске, не помечено соответствующим образом.

## <a name="add-facet-navigation-to-your-app"></a>Добавление аспектной навигации в приложение

В этом примере мы дадим пользователю возможность выбрать одну категорию отелей или одно удобство из списков ссылок, показанных слева от результатов. Пользователь начинает с ввода текста для поиска, а затем постепенно ограничивает результаты поиска, выбирая категорию или удобство.

Передачу списков аспектов в представление выполняет контроллер. Необходимо сохранять выбор пользователя в процессе поиска. В качестве механизма сохранения данных мы используем временное хранилище.

![Использование аспектной навигации для сужения поиска по удобству "бассейн"](./media/tutorial-csharp-create-first-app/azure-search-facet-nav.png)

### <a name="add-filter-strings-to-the-searchdata-model"></a>Добавление строк фильтра в модель SearchData

1. Откройте файл SearchData.cs и добавьте строковые свойства в класс **SearchData** для хранения строк фильтра аспектов.

    ```cs
    public string categoryFilter { get; set; }
    public string amenityFilter { get; set; }
    ```

### <a name="add-the-facet-action-method"></a>Добавление метода действия Facet

Домашнему контроллеру требуется одно новое действие, **Facet**, и обновления к его существующим действиям **Index** и **Page**, а также обновления метода **RunQueryAsync**.

<!-- 1. Open the home controller file, and add the **using** statement, to enable the **List&lt;string&gt;** construct.

    ```cs
    using System.Collections.Generic; 
    ```-->

1. Замените метод действия **Index(SearchData model)** .

    ```cs
    public async Task<ActionResult> Index(SearchData model)
    {
        try
        {
            // Ensure the search string is valid.
            if (model.searchText == null)
            {
                model.searchText = "";
            }

            // Make the search call for the first page.
            await RunQueryAsync(model, 0, 0, "", "").ConfigureAwait(false);
        }
        catch
        {
            return View("Error", new ErrorViewModel { RequestId = "1" });
        }

        return View(model);
    }
    ```

1. Замените метод действия **PageAsync(SearchData model)** .

    ```cs
    public async Task<ActionResult> PageAsync(SearchData model)
    {
        try
        {
            int page;

            // Calculate the page that should be displayed.
            switch (model.paging)
            {
                case "prev":
                    page = (int)TempData["page"] - 1;
                    break;

                case "next":
                    page = (int)TempData["page"] + 1;
                    break;

                default:
                    page = int.Parse(model.paging);
                    break;
            }

            // Recover the leftMostPage.
            int leftMostPage = (int)TempData["leftMostPage"];

            // Recover the filters.
            string catFilter = TempData["categoryFilter"].ToString();
            string ameFilter = TempData["amenityFilter"].ToString();

            // Recover the search text.
            model.searchText = TempData["searchfor"].ToString();

            // Search for the new page.
            await RunQueryAsync(model, page, leftMostPage, catFilter, ameFilter);
        }

        catch
        {
            return View("Error", new ErrorViewModel { RequestId = "2" });
        }
        return View("Index", model);
    }
    ```

1. Добавьте метод действия **FacetAsync(SearchData model)** , который будет активирован, когда пользователь щелкнет ссылку на аспект. Модель будет содержать либо фильтр поиска по категориям, либо фильтр поиска по удобствам. Добавьте это действие после действия **PageAsync**.

    ```cs
    public async Task<ActionResult> FacetAsync(SearchData model)
    {
        try
        {
            // Filters set by the model override those stored in temporary data.
            string catFilter;
            string ameFilter;
            if (model.categoryFilter != null)
            {
                catFilter = model.categoryFilter;
            } else
            {
                catFilter = TempData["categoryFilter"].ToString();
            }

            if (model.amenityFilter != null)
            {
                ameFilter = model.amenityFilter;
            } else
            {
                ameFilter = TempData["amenityFilter"].ToString();
            }

            // Recover the search text.
            model.searchText = TempData["searchfor"].ToString();

            // Initiate a new search.
            await RunQueryAsync(model, 0, 0, catFilter, ameFilter).ConfigureAwait(false);
        }
        catch
        {
            return View("Error", new ErrorViewModel { RequestId = "2" });
        }

        return View("Index", model);
    }
    ```

### <a name="set-up-the-search-filter"></a>Настройка фильтра поиска

Если пользователь выбирает определенный аспект, например он щелкает категорию **Resort and Spa**, тогда в результатах должны быть возвращены только отели, указанные в этой категории. Чтобы сузить поиск таким образом, нам нужно настроить _фильтр_.

1. Замените метод **RunQueryAsync** следующим кодом. Прежде всего, он принимает строку фильтра категории и строку фильтра удобства и устанавливает параметр **Filter** класса **SearchOptions**.

    ```cs
    private async Task<ActionResult> RunQueryAsync(SearchData model, int page, int leftMostPage, string catFilter, string ameFilter)
    {
        InitSearch();

        string facetFilter = "";

        if (catFilter.Length > 0 && ameFilter.Length > 0)
        {
            // Both facets apply.
            facetFilter = $"{catFilter} and {ameFilter}"; 
        } else
        {
            // One, or zero, facets apply.
            facetFilter = $"{catFilter}{ameFilter}";
        }

        var options = new SearchOptions
        {
            Filter = facetFilter,

            SearchMode = SearchMode.All,

            // Skip past results that have already been returned.
            Skip = page * GlobalVariables.ResultsPerPage,

            // Take only the next page worth of results.
            Size = GlobalVariables.ResultsPerPage,

            // Include the total number of results.
            IncludeTotalCount = true,
        };

        // Return information on the text, and number, of facets in the data.
        options.Facets.Add("Category,count:20");
        options.Facets.Add("Tags,count:20");

        // Enter Hotel property names into this list, so only these values will be returned.
        options.Select.Add("HotelName");
        options.Select.Add("Description");
        options.Select.Add("Category");
        options.Select.Add("Tags");

        // For efficiency, the search call should be asynchronous, so use SearchAsync rather than Search.
        model.resultList = await _searchClient.SearchAsync<Hotel>(model.searchText, options).ConfigureAwait(false);

        // This variable communicates the total number of pages to the view.
        model.pageCount = ((int)model.resultList.TotalCount + GlobalVariables.ResultsPerPage - 1) / GlobalVariables.ResultsPerPage;

        // This variable communicates the page number being displayed to the view.
        model.currentPage = page;

        // Calculate the range of page numbers to display.
        if (page == 0)
        {
            leftMostPage = 0;
        }
        else if (page <= leftMostPage)
        {
            // Trigger a switch to a lower page range.
            leftMostPage = Math.Max(page - GlobalVariables.PageRangeDelta, 0);
        }
        else if (page >= leftMostPage + GlobalVariables.MaxPageRange - 1)
        {
            // Trigger a switch to a higher page range.
            leftMostPage = Math.Min(page - GlobalVariables.PageRangeDelta, model.pageCount - GlobalVariables.MaxPageRange);
        }
        model.leftMostPage = leftMostPage;

        // Calculate the number of page numbers to display.
        model.pageRange = Math.Min(model.pageCount - leftMostPage, GlobalVariables.MaxPageRange);

        // Ensure Temp data is stored for the next call.
        TempData["page"] = page;
        TempData["leftMostPage"] = model.leftMostPage;
        TempData["searchfor"] = model.searchText;
        TempData["categoryFilter"] = catFilter;
        TempData["amenityFilter"] = ameFilter;

        // Return the new view.
        return View("Index", model);
    }
    ```

    Обратите внимание, что свойства **Category** и **Tags** добавлены в список элементов **Select** для возврата. Это дополнение не является обязательным условием для работы аспектной навигации, но мы используем эту информацию для проверки правильности работы фильтров.

### <a name="add-lists-of-facet-links-to-the-view"></a>Добавление списков ссылок на аспекты в представление

В представление потребуется внести существенные изменения. 

1. Откройте файл hotels.css (в папке wwwroot/css) и добавьте следующие классы.

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

1. Для представления упорядочите выходные данные в виде таблицы, чтобы разместить списки аспектов слева, а результаты — справа. Откройте файл index.cshtml. Замените все содержимое тегов &lt;body&gt; в HTML следующим кодом.

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
                            Hotels Search - Facet Navigation
                        </h1>
                    </td>
                </tr>

                <tr>
                    <td></td>
                    <td>
                        <!-- Display the search text box, with the search icon to the right of it.-->
                        <div class="searchBoxForm">
                            @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox" }) <input value="" class="searchBoxSubmit" type="submit">
                        </div>
                    </td>
                </tr>

                <tr>
                    <td valign="top">
                        <div id="facetplace" class="facetchecks">

                            @if (Model != null && Model.resultList != null)
                            {
                                List<string> categories = Model.resultList.Facets["Category"].Select(x => x.Value.ToString()).ToList();

                                if (categories.Count > 0)
                                {
                                    <h5 class="facetheader">Category:</h5>
                                    <ul class="facetlist">
                                        @for (var c = 0; c < categories.Count; c++)
                                        {
                                            var facetLink = $"{categories[c]} ({Model.resultList.Facets["Category"][c].Count})";
                                            <li>
                                                @Html.ActionLink(facetLink, "FacetAsync", "Home", new { categoryFilter = $"Category eq '{categories[c]}'" }, null)
                                            </li>
                                        }
                                    </ul>
                                }

                                List<string> tags = Model.resultList.Facets["Tags"].Select(x => x.Value.ToString()).ToList();

                                if (tags.Count > 0)
                                {
                                    <h5 class="facetheader">Amenities:</h5>
                                    <ul class="facetlist">
                                        @for (var c = 0; c < tags.Count; c++)
                                        {
                                            var facetLink = $"{tags[c]} ({Model.resultList.Facets["Tags"][c].Count})";
                                            <li>
                                                @Html.ActionLink(facetLink, "FacetAsync", "Home", new { amenityFilter = $"Tags/any(t: t eq '{tags[c]}')" }, null)
                                            </li>
                                        }
                                    </ul>
                                }
                            }
                        </div>
                    </td>
                    <td valign="top">
                        <div id="resultsplace">
                            @if (Model != null && Model.resultList != null)
                            {
                                // Show the result count.
                                <p class="sampleText">
                                    @Model.resultList.TotalCount Results
                                </p>

                                var results = Model.resultList.GetResults().ToList();

                                @for (var i = 0; i < results.Count; i++)
                                {
                                    string amenities = string.Join(", ", results[i].Document.Tags);

                                    string fullDescription = results[i].Document.Description;
                                    fullDescription += $"\nCategory: {results[i].Document.Category}";
                                    fullDescription += $"\nAmenities: {amenities}";


                                    // Display the hotel name and description.
                                    @Html.TextAreaFor(m => results[i].Document.HotelName, new { @class = "box1" })
                                    @Html.TextArea($"desc{i}", fullDescription, new { @class = "box2" })
                                }
                            }
                        </div>
                    </td>
                </tr>

                <tr>
                    <td></td>
                    <td valign="top">
                        @if (Model != null && Model.pageCount > 1)
                        {
                            // If there is more than one page of results, show the paging buttons.
                            <table>
                                <tr>
                                    <td class="tdPage">
                                        @if (Model.currentPage > 0)
                                        {
                                            <p class="pageButton">
                                                @Html.ActionLink("|<", "PageAsync", "Home", new { paging = "0" }, null)
                                            </p>
                                        }
                                        else
                                        {
                                            <p class="pageButtonDisabled">|&lt;</p>
                                        }
                                    </td>

                                    <td class="tdPage">
                                        @if (Model.currentPage > 0)
                                        {
                                            <p class="pageButton">
                                                @Html.ActionLink("<", "PageAsync", "Home", new { paging = "prev" }, null)
                                            </p>
                                        }
                                        else
                                        {
                                            <p class="pageButtonDisabled">&lt;</p>
                                        }
                                    </td>

                                    @for (var pn = Model.leftMostPage; pn < Model.leftMostPage + Model.pageRange; pn++)
                                    {
                                        <td class="tdPage">
                                            @if (Model.currentPage == pn)
                                            {
                                                // Convert displayed page numbers to 1-based and not 0-based.
                                                <p class="pageSelected">@(pn + 1)</p>
                                            }
                                            else
                                            {
                                                <p class="pageButton">
                                                    @Html.ActionLink((pn + 1).ToString(), "PageAsync", "Home", new { paging = @pn }, null)
                                                </p>
                                            }
                                        </td>
                                    }

                                    <td class="tdPage">
                                        @if (Model.currentPage < Model.pageCount - 1)
                                        {
                                            <p class="pageButton">
                                                @Html.ActionLink(">", "PageAsync", "Home", new { paging = "next" }, null)
                                            </p>
                                        }
                                        else
                                        {
                                            <p class="pageButtonDisabled">&gt;</p>
                                        }
                                    </td>

                                    <td class="tdPage">
                                        @if (Model.currentPage < Model.pageCount - 1)
                                        {
                                            <p class="pageButton">
                                                @Html.ActionLink(">|", "PageAsync", "Home", new { paging = Model.pageCount - 1 }, null)
                                            </p>
                                        }
                                        else
                                        {
                                            <p class="pageButtonDisabled">&gt;|</p>
                                        }
                                    </td>
                                </tr>
                            </table>
                        }
                    </td>
                </tr>
            </table>
        }
    </body>
    ```

    Обратите внимание на использование вызова **Html.ActionLink**. Этот вызов передает действительные строки фильтра на контроллер, когда пользователь щелкает ссылку на аспект. 

### <a name="run-and-test-the-app"></a>Запуск и тестирование приложения

Преимущество аспектной навигации для пользователя заключается в том, что он может сузить поиск одним щелчком мыши. Это будет видно в следующей последовательности.

1. Запустите приложение и введите airport в качестве текста для поиска. Убедитесь, что список аспектов отображается слева. Эти аспекты — это все, что относится к отелям, в текстовых данных которых упоминается "аэропорт", с подсчетом частоты их появления.

    ![Использование аспектной навигации для сужения поиска по аэропортам](./media/tutorial-csharp-create-first-app/azure-search-facet-airport.png)

1. Выберите категорию **Resort and Spa**. Убедитесь, что все результаты относятся к этой категории.

    ![Сужение поиска до Resort and Spa](./media/tutorial-csharp-create-first-app/azure-search-facet-airport-ras.png)

1. Щелкните удобство **континентальный завтрак**. Убедитесь, что все результаты по-прежнему относятся к категории Resort and Spa и содержат выбранные удобства.

    ![Сужение поиска до аспекта "континентальный завтрак"](./media/tutorial-csharp-create-first-app/azure-search-facet-airport-ras-cb.png)

1. Попробуйте выбрать любую другую категорию, а затем одно удобство и посмотрите суженные результаты поиска. Затем попробуйте другой способ (одно удобство, а затем одну категорию). Чтобы сбросить страницу, отправьте пустой поисковый запрос.

    >[!Note]
    > После выбора в списке аспектов (например, категории) любой предыдущий выбор в списке категорий будет переопределен.

## <a name="takeaways"></a>Общие выводы

По результатам этого проекта можно сделать такие выводы:

* Необходимо отметить каждое поле с аспектом свойством **IsFacetable** для включения в навигацию по аспектам.
* Аспекты объединяются с фильтрами для уменьшения количества результатов.
* Аспекты являются накопительными, и каждый выбор, построенный на предыдущем, позволяет сузить область результатов.

## <a name="next-steps"></a>Дальнейшие действия

В следующем учебнике мы рассмотрим результаты бронирования. До этого момента результаты упорядочиваются просто — в том порядке, в котором они находятся в базе данных.

> [!div class="nextstepaction"]
> [Учебник C# по упорядочиванию результатов в службе "Когнитивный поиск Azure"](tutorial-csharp-orders.md)
