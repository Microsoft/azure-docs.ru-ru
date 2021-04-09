---
title: Учебник по разбиению результатов поиска на страницы в C#
titleSuffix: Azure Cognitive Search
description: Добавьте в результаты поиска разбивку на страницы и кнопки навигации, расширив существующий проект гостиниц с помощью кнопок первой, следующей, предыдущей, последней страниц и номеров страниц. Вторая система разбиения на страницы использует бесконечную прокрутку, которая включается в случае перемещения вертикальной полосы прокрутки до нижнего предела.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 01/26/2021
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: bad4bc4d0016b2898b315bfb9799dc8972be7b12
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98785875"
---
# <a name="tutorial-add-paging-to-search-results-using-the-net-sdk"></a>Руководство по Добавление разбиения результатов поиска на страницы с помощью пакета SDK для .NET

Узнайте, как реализовать две разные системы разбиения на страницы: одну на основе номеров страниц и вторую с бесконечной прокруткой. Обе системы разбиения на страницы широко используются, и правильный выбор между ними зависит от того, какое взаимодействие с пользователем вы намерены организовать. 

Из этого руководства вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * добавление страниц в приложение;
> * Добавление бесконечной прокрутки в приложение

## <a name="overview"></a>Обзор

Это руководство вводит систему разбиения на страницы в ранее созданный проект, описанный в руководстве по [созданию первого приложения для поиска](tutorial-csharp-create-first-app.md).

Готовую версию кода, который вы будете разрабатывать для этого руководства, можно найти в следующих проектах:

* [2a-add-paging (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/2a-add-paging)

* [2b-add-infinite-scroll (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/2b-add-infinite-scroll)

## <a name="prerequisites"></a>Предварительные требования

* Проект [1-basic-search-page (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/1-basic-search-page). Это может быть ваш собственный проект, созданный в рамках предыдущего руководства, или копия с GitHub.

Это руководство было обновлено для использования пакета [Azure.Search.Documents (версия 11)](https://www.nuget.org/packages/Azure.Search.Documents/). Сведения о более ранней версии пакета SDK для .NET приведены в [примере кода для Microsoft.Azure.Search (версия 10)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v10).

## <a name="extend-your-app-with-numbered-paging"></a>добавление страниц в приложение;

Разбиение на страницы с номерами страниц стандартно применяется во всех крупнейших коммерческих поисковых веб-системах и на большинстве других веб-сайтов с возможностью поиска. Система с номерами страниц обычно дополняется элементами для перехода на "следующую" и "предыдущую" страницы, помимо списка с порядковыми номерами страниц. Иногда бывают доступны и элементы "первая страница" и "последняя страница". Эти варианты дают пользователю уверенный контроль над навигацией по страницам с результатами.

В этом руководстве мы добавим систему с элементами для перехода к первой, предыдущей, следующей и последней страницам, а также с номерами страниц, которые начинаются не с 1, а с близких к текущей странице номеров (например, если пользователь просматривает страницу 10, будут отображаться номера для страниц 8, 9, 10, 11 и 12).

Система будет достаточно гибкой, чтобы настраивать количество номеров отображаемых страниц через глобальную переменную.

Система будет считать крайние номера страниц слева и справа особенными, то есть изменяющими диапазон отображаемых номеров страниц. Например, если отображаются номера страниц 8, 9, 10, 11 и 12, то после нажатия пользователем номера 8 система отображает другой диапазон номеров страниц: 6, 7, 8, 9 и 10. Аналогичный сдвиг вправо происходит при выборе номера 12.

### <a name="add-paging-fields-to-the-model"></a>Добавление полей разбиения на страницы в модель

Откройте базовое решение страницы поиска.

1. Откройте файл модели SearchData.cs.

1. Добавьте глобальные переменные для поддержки разбиения на страницы. В MVC глобальные переменные объявляются в отдельном статическом классе. **ResultsPerPage** задает количество результатов на странице. **MaxPageRange** определяет количество отображаемых номеров страниц. **PageRangeDelta** определяет, сколько страниц влево или вправо следует сдвинуть при выборе крайнего номера страницы слева или справа. Обычно это последнее значение составляет около половины от **MaxPageRange**. Добавьте следующий код в пространство имен.

    ```csharp
    public static class GlobalVariables
    {
        public static int ResultsPerPage
        {
            get
            {
                return 3;
            }
        }
        public static int MaxPageRange
        {
            get
            {
                return 5;
            }
        }

        public static int PageRangeDelta
        {
            get
            {
                return 2;
            }
        }
    }
    ```

    >[!Tip]
    >Если проект выполняется на устройстве с небольшим экраном, например на ноутбуке, для параметра **ResultsPerPage** можно применить значение 2.

1. Добавьте свойства разбиения на страницы в класс **SearchData** сразу после свойства **searchText**.

    ```csharp
    // The current page being displayed.
    public int currentPage { get; set; }

    // The total number of pages of results.
    public int pageCount { get; set; }

    // The left-most page number to display.
    public int leftMostPage { get; set; }

    // The number of page numbers to display - which can be less than MaxPageRange towards the end of the results.
    public int pageRange { get; set; }

    // Used when page numbers, or next or prev buttons, have been selected.
    public string paging { get; set; }
    ```

### <a name="add-a-table-of-paging-options-to-the-view"></a>Добавление таблицы с элементами разбиения на страницы в представление

1. Откройте файл index.cshtml и добавьте следующий код сразу перед закрывающим тегом &lt;/body&gt;. Этот код представляет таблицу для элементов разбиения на страницы: первая, предыдущая, 1, 2, 3, 4, 5, следующая, последняя.

    ```html
    @if (Model != null && Model.pageCount > 1)
    {
    // If there is more than one page of results, show the paging buttons.
    <table>
        <tr>
            <td>
                @if (Model.currentPage > 0)
                {
                    <p class="pageButton">
                        @Html.ActionLink("|<", "Page", "Home", new { paging = "0" }, null)
                    </p>
                }
                else
                {
                    <p class="pageButtonDisabled">|&lt;</p>
                }
            </td>

            <td>
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
                <td>
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

            <td>
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

            <td>
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
    ```

    Чтобы все выглядело аккуратно, мы используем таблицу HTML. Но все наши действия определены в инструкциях @Html.ActionLink, каждая из которых вызывает контроллер с **новой** моделью, для которой указаны другие значения добавленного ранее свойства **paging**.

    Элементы для перехода к первой и последней страницам отправляют не строковые значения "первый" и "последний", а правильные номера страниц.

1. Добавьте классы для разбиения на страницы в список стилей HTML в файле hotels.css. Класс **pageSelected** определяет текущую страницу (выделяя ее номер полужирным шрифтом) в списке номеров страниц.

    ```html
    .pageButton {
        border: none;
        color: darkblue;
        font-weight: normal;
        width: 50px;
    }

    .pageSelected {
        border: none;
        color: black;
        font-weight: bold;
        width: 50px;
    }

    .pageButtonDisabled {
        border: none;
        color: lightgray;
        font-weight: bold;
        width: 50px;
    }
    ```

### <a name="add-a-page-action-to-the-controller"></a>Добавление действия Page в контроллер

1. Откройте файл HomeController.cs и добавьте в него действие **PageAsync**. Это действие реагирует на выбор любого элемента перехода к другой странице.

    ```csharp
    public async Task<ActionResult> PageAsync(SearchData model)
    {
        try
        {
            int page;

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

            // Recover the search text and search for the data for the new page.
            model.searchText = TempData["searchfor"].ToString();

            await RunQueryAsync(model, page, leftMostPage);

            // Ensure Temp data is stored for next call, as TempData only stores for one call.
            TempData["page"] = (object)page;
            TempData["searchfor"] = model.searchText;
            TempData["leftMostPage"] = model.leftMostPage;
        }

        catch
        {
            return View("Error", new ErrorViewModel { RequestId = "2" });
        }
        return View("Index", model);
    }
    ```

    Метод **RunQueryAsync** теперь будет возвращать синтаксическую ошибку, так как у нас появился третий параметр, обработку которого мы добавим чуть позже.

    > [!Note]
    > Вызовы класса **TempData** сохраняют значение (**object**) во временное хранилище, которое действует _только_ на протяжении одного вызова. То, что сохраняется во временном хранилище данных, будет доступно для следующего вызова действия контроллера, но исчезнет сразу после этого вызова. Такое короткое время существования означает, что текст для поиска и параметры разбиения на страницы нужно сохранять во временное хранилище при каждом вызове действия **PageAsync**.

1. Обновите действие **Index(model)** так, чтобы оно сохраняло временные переменные, а также передавало в вызов **RunQueryAsync** крайний левый параметр, предназначенный для разбиения на страницы.

    ```csharp
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
            await RunQueryAsync(model, 0, 0);

            // Ensure temporary data is stored for the next call.
            TempData["page"] = 0;
            TempData["leftMostPage"] = 0;
            TempData["searchfor"] = model.searchText;
        }

        catch
        {
            return View("Error", new ErrorViewModel { RequestId = "1" });
        }
        return View(model);
    }
    ```

1. Чтобы устранить синтаксическую ошибку, требуется изменить метод **RunQueryAsync**, представленный на предыдущем занятии. Мы применим поля **Skip**, **Size** и **IncludeTotalCount** из класса [**SearchOptions**](/dotnet/api/azure.search.documents.searchoptions), чтобы запросить выдачу только одной страницы результатов, начиная с параметра **Skip**. Нам также нужно вычислить переменные для отображения элементов разбиения на страницы. Замените весь этот метод приведенным ниже кодом.

    ```csharp
    private async Task<ActionResult> RunQueryAsync(SearchData model, int page, int leftMostPage)
    {
        InitSearch();

        var options = new SearchOptions
        {
            // Skip past results that have already been returned.
            Skip = page * GlobalVariables.ResultsPerPage,

            // Take only the next page worth of results.
            Size = GlobalVariables.ResultsPerPage,

            // Include the total number of results.
            IncludeTotalCount = true
        };

        // Add fields to include in the search results.
        options.Select.Add("HotelName");
        options.Select.Add("Description");

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

        return View("Index", model);
    }
    ```

1. И, наконец, внесем небольшое изменение в представление. Переменная **resultList.Results.TotalCount** теперь будет содержать число результатов, возвращаемых на одной странице (3 в нашем примере), а не общее число результатов. Так как **IncludeTotalCount** имеет значение true, переменная **resultList.TotalCount** теперь содержит общее число результатов. Найдите, где в представлении отображается количество результатов, и замените этот код следующим фрагментом.

    ```csharp
    // Show the result count.
    <p class="sampleText">
        @Model.resultList.TotalCount Results
    </p>

    var results = Model.resultList.GetResults().ToList();

    @for (var i = 0; i < results.Count; i++)
    {
        // Display the hotel name and description.
        @Html.TextAreaFor(m => results[i].Document.HotelName, new { @class = "box1" })
        @Html.TextArea($"desc{1}", results[i].Document.Description, new { @class = "box2" })
    }
    ```

    > [!Note]
    > Если вы присвоите переменной **IncludeTotalCount** значение true, производительность ненамного снизится, так как Когнитивный поиск Azure будет каждый раз вычислять это значение. Для очень сложных наборов данных действует предупреждение о том, что полученный результат лишь _приблизительный_. Так как данных для поиска отелей немного, результат будет точным.

### <a name="compile-and-run-the-app"></a>Компиляция и запуск приложения

Теперь нажмите кнопку **Запуск без отладчика** или клавишу F5.

1. Выполните поиск строки, возвращающей много результатов (например, "wifi"). Удается ли вам аккуратно пролистывать результаты?

    ![Нумерованный постраничный просмотр результатов по запросу "pool"](./media/tutorial-csharp-create-first-app/azure-search-numbered-paging.png)

1. Попробуйте нажать крайний правый номер страницы, а затем крайний левый. Правильно ли изменяются номера страниц в соответствии с той страницей, которую вы просматриваете?

1. Полезны ли варианты страниц "первая" и "последняя"? Одни коммерческие поисковые системы используют эти параметры, а другие — нет.

1. Перейдите к последней странице результатов. Это единственная странице, которая может содержать меньше результатов, чем значение **ResultsPerPage**.

    ![Просмотр последней страницы результатов по запросу "wifi"](./media/tutorial-csharp-create-first-app/azure-search-pool-last-page.png)

1. Введите строку "town" (город) и нажмите кнопку поиска. Если количество результатов умещается на одной странице, элементы разбиения на страницы не отображаются.

    ![Поиск по строке "town"](./media/tutorial-csharp-create-first-app/azure-search-town.png)

Сохраните этот проект и перейдите к следующему разделу для использования альтернативного варианта разбиения на страницы.

## <a name="extend-your-app-with-infinite-scrolling"></a>Добавление бесконечной прокрутки в приложение

Бесконечная прокрутка активируется, когда пользователь доводит вертикальную полосу прокрутки до последнего из отображаемых результатов. В этом случае выполняется вызов к службе поиска для получения следующей страницы результатов. Если результатов больше нет, возвращается пустой результат и вертикальная полоса прокрутки не изменяется. Если имеются дополнительные результаты, они добавляются к текущей странице с соответствующим изменением положения полосы прокрутки.

Важно отметить, что текущая страница не заменяется, а расширяется для отображения дополнительных результатов. Пользователь может всегда прокрутить страницу обратно к первым результатам поиска.

Чтобы реализовать бесконечную прокрутку, мы начнем с того состояния проекта, в котором еще не было элементов для перехода по номерам страниц. На GitHub это решение [FirstAzureSearchApp](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/1-basic-search-page).

### <a name="add-paging-fields-to-the-model"></a>Добавление полей разбиения на страницы в модель

1. Во-первых, добавьте свойство **paging** в класс **SearchData** в файле модели SearchData.cs.

    ```csharp
    // Record if the next page is requested.
    public string paging { get; set; }
    ```

    Эта строковая переменная содержит значение "next" (далее), если отправляется очередная страница результатов, или значение NULL, если отображается первая страница результатов поиска.

1. В этом же файле и в том же пространстве имен добавьте класс глобальных переменных с одним свойством. В MVC глобальные переменные объявляются в отдельном статическом классе. **ResultsPerPage** задает количество результатов на странице. 

    ```csharp
    public static class GlobalVariables
    {
        public static int ResultsPerPage
        {
            get
            {
                return 3;
            }
        }
    }
    ```

### <a name="add-a-vertical-scroll-bar-to-the-view"></a>Добавление вертикальной полосы прокрутки в представление

1. Найдите раздел файла index.cshtml, который отображает результаты (он начинается со строки **@if (Model != null)** ).

1. Замените этот фрагмент следующим кодом: Новый раздел **&lt;div&gt;** содержит прокручиваемую область и добавляет новый атрибут **overflow-y** и вызов функции **onscroll** с именем scrolled(), как показано ниже.

    ```csharp
    @if (Model != null)
    {
        // Show the result count.
        <p class="sampleText">
            @Model.resultList.TotalCount Results
        </p>

        var results = Model.resultList.GetResults().ToList();

        <div id="myDiv" style="width: 800px; height: 450px; overflow-y: scroll;" onscroll="scrolled()">

            <!-- Show the hotel data. -->
            @for (var i = 0; i < results.Count; i++)
            {
                // Display the hotel name and description.
                @Html.TextAreaFor(m => results[i].Document.HotelName, new { @class = "box1" })
                @Html.TextArea($"desc{i}", results[i].Document.Description, new { @class = "box2" })
            }
    ```

1. Сразу после цикла и тега &lt;/div&gt; добавьте функцию **scrolled**.

    ```javascript
    <script>
        function scrolled() {
            if (myDiv.offsetHeight + myDiv.scrollTop >= myDiv.scrollHeight) {
                $.getJSON("/Home/NextAsync", function (data) {
                    var div = document.getElementById('myDiv');

                    // Append the returned data to the current list of hotels.
                    for (var i = 0; i < data.length; i += 2) {
                        div.innerHTML += '\n<textarea class="box1">' + data[i] + '</textarea>';
                        div.innerHTML += '\n<textarea class="box2">' + data[i + 1] + '</textarea>';
                    }
                });
            }
        }
    </script>
    ```

    Инструкция **if** в представленном выше скрипте проверяет, дошел ли пользователь до нижнего положения вертикальной полосы прокрутки. Если это так, выполняется действие **NextAsync** из контроллера **Home**. Этот контроллер не требует никакой информации и просто возвращает следующую страницу данных. Затем эти данные форматируются с применением тех же стилей HTML, что и для исходной страницы. Если результаты не возвращаются, данные на странице сохраняют прежний вид.

### <a name="handle-the-next-action"></a>Обработка действия Next

Контроллеру могут отправляться только три действия: первый запуск приложения вызывает **Index()** , первый поиск вызывает **Index(model)** , а все последующие обращения запрашивают дополнительные результаты через действие **Next(model)** .

1. Откройте файл контроллера Home и удалите метод **RunQueryAsync**, который был создан в прошлом учебнике.

1. Замените действие **Index(model)** следующим кодом: Теперь код обрабатывает поле **paging**, если оно содержит значение NULL или next, а затем выполняет вызов к Когнитивному поиску Azure.

    ```csharp
    public async Task<ActionResult> Index(SearchData model)
    {
        try
        {
            InitSearch();

            int page;

            if (model.paging != null && model.paging == "next")
            {
                // Increment the page.
                page = (int)TempData["page"] + 1;

                // Recover the search text.
                model.searchText = TempData["searchfor"].ToString();
            }
            else
            {
                // First call. Check for valid text input.
                if (model.searchText == null)
                {
                    model.searchText = "";
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
                IncludeTotalCount = true
            };

            // Specify which fields to include in results.
            options.Select.Add("HotelName");
            options.Select.Add("Description");

            // For efficiency, the search call should be asynchronous, so use SearchAsync rather than Search.
            model.resultList = await _searchClient.SearchAsync<Hotel>(model.searchText, options).ConfigureAwait(false);               

            // Ensure TempData is stored for the next call.
            TempData["page"] = page;
            TempData["searchfor"] = model.searchText;
        }
        catch
        {
            return View("Error", new ErrorViewModel { RequestId = "1" });
        }

        return View("Index", model);
    }
    ```

    Как и в методе разбиения на страницы с номерами, мы используем параметры поиска **Skip** и **Size** для получения только нужного подмножества данных.

1. Добавьте действие **NextAsync** в контроллер Home. Обратите внимание, что он возвращает список отелей, каждый из элементов которого добавляет два элемента: название и описание отеля. Используется тот же формат, что и в функции **scrolled** для отображения возвращаемых данных в представлении.

    ```csharp
    public async Task<ActionResult> NextAsync(SearchData model)
    {
        // Set the next page setting, and call the Index(model) action.
        model.paging = "next";
        await Index(model).ConfigureAwait(false);

        // Create an empty list.
        var nextHotels = new List<string>();

        // Add a hotel name, then description, to the list.
        await foreach (var searchResult in model.resultList.GetResultsAsync())
        {
            nextHotels.Add(searchResult.Document.HotelName);
            nextHotels.Add(searchResult.Document.Description);
        }

        // Rather than return a view, return the list of data.
        return new JsonResult(nextHotels);
    }
    ```

1. Если вы получите синтаксическую ошибку в вызове **List&lt;string&gt;** , добавьте следующую директиву **using** в заголовок файла контроллера.

    ```csharp
    using System.Collections.Generic;
    ```

### <a name="compile-and-run-your-project"></a>Компиляция и запуск проекта

Теперь нажмите кнопку **Запуск без отладчика** или клавишу F5.

1. Введите термин, который возвращает много результатов поиска, например "pool" (бассейн), и проверьте работу вертикальной полосы прокрутки. Подгружает ли она новую страницу результатов?

    ![Просмотр результатов по запросу "pool" с бесконечной прокруткой](./media/tutorial-csharp-create-first-app/azure-search-infinite-scroll.png)

    > [!Tip]
    > Чтобы на первой странице всегда отображалась полоса прокрутки, число результатов на ней должно превышать высоту области, в которой они отображаются. В нашем примере **.box1** имеет высоту 30 пикселей, а **.box2** — высоту 100 пикселей _и_ нижнее поле высотой 24 точки. Таким образом, каждая запись использует 154 пикселя. Три записи займут 3 x 154 = 462 пикселя в высоту. Чтобы всегда появлялась вертикальная полоса прокрутки, высота области отображения должна быть меньше 462 пикселей, подойдет даже значение 461. Такая проблема возникает только на первой странице, после этого полоса прокрутки отображается гарантированно. Следует изменить следующую строку: **&lt;div id="myDiv" style="width: 800px; height: 450px; overflow-y: scroll;" onscroll="scrolled()"&gt;** .

1. Прокрутите список результатов до конца. Обратите внимание, что вся информация теперь отображается на одной странице представления. Вы можете спокойно прокручивать ее до самого верха, не активируя новых вызовов к серверу.

Более сложные системы бесконечной прокрутки могут использовать для загрузки новой страницы результатов колесико мыши или другие механизмы. В рамках этих руководств мы не будем усложнять систему бесконечной прокрутки, но такие решения выглядят довольно привлекательно и позволяют снизить число щелчков мышью, так что советуем вам дополнительно изучить их!

## <a name="takeaways"></a>Общие выводы

По результатам этого проекта можно сделать такие выводы:

* Нумерованная система разбиения на страницы подходит для поисков, в которых порядок результатов достаточно произволен, то есть на любой из страниц может появиться что-то интересное для пользователя.
* Бесконечная прокрутка полезна в тех случаях, когда порядок результатов очень важен. Например, если результаты сортируются по расстоянию от центра нужного города.
* Нумерованное разбиение на страницы более удобно для навигации. Например, пользователь может запомнить, что интересный для него результат расположен на странице 6. В варианте с бесконечной прокруткой такие простые ориентиры отсутствуют.
* Бесконечная прокрутка привлекает простотой, так как позволяет использовать привычное перемещение вверх и вниз без щелчков по номерам страниц.
* Основная особенность бесконечный прокрутки заключается в том, что результаты добавляются на существующую страницу, а не заменяют ее, что повышает эффективность.
* Временное хранилище сохраняет данные только в течение одного вызова, а для последующих вызовов его необходимо обновлять.

## <a name="next-steps"></a>Дальнейшие действия

Разбиение на страницы — очень важная функция для поиска. Теперь, когда мы тщательно его изучили, можно перейти к упреждающим поисковым запросам, которые еще больше улучшат взаимодействие с пользователем.

> [!div class="nextstepaction"]
> [Руководство для разработчиков C#. Добавление функции автозаполнения и предложений — Когнитивный поиск Azure](tutorial-csharp-type-ahead-and-suggestions.md)