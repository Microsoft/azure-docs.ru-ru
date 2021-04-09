---
title: Учебник по созданию первого приложения на C#
titleSuffix: Azure Cognitive Search
description: Узнайте, как создать первое приложение поиска C# с помощью пошаговых инструкций. В нем описывается весь процесс создания приложения, а также приводится ссылка для скачивания готового приложения из репозитория GitHub.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 02/26/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 0a57e45b264badffd0305eb6ac5b3c8f7c42adf3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101695130"
---
# <a name="tutorial-create-your-first-search-app-using-the-net-sdk"></a>Руководство по Создание первого приложения поиска с помощью пакета SDK для .NET

В этом руководстве показано, как создать веб-приложение, которое выполняет запрос и возвращает результаты из индекса поиска с помощью Когнитивного поиска Azure и Visual Studio.

В этом учебнике рассматривается следующее.

> [!div class="checklist"]
> * Настройка среды разработки
> * Моделирование структур данных
> * Создание веб-страницы для сбора входных данных запроса и отображения результатов.
> * Определение метода поиска.
> * Тестирование приложения

Вы также узнаете, насколько простым является вызов поиска. Ключевые инструкции в создаваемом вами коде заключены в следующие несколько строк.

```csharp
var options = new SearchOptions()
{
    // The Select option specifies fields for the result set
    options.Select.Add("HotelName");
    options.Select.Add("Description");
};

var searchResult = await _searchClient.SearchAsync<Hotel>(model.searchText, options).ConfigureAwait(false);
model.resultList = searchResult.Value.GetResults().ToList();
```

Для выполнения запроса к индексу и возвращения результатов используется только один вызов.

:::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-pool.png" alt-text="Поиск по запросу &quot;бассейн&quot;" border="true":::

## <a name="overview"></a>Обзор

В этом учебнике используется функция hotels-sample-index, которую можно быстро создать в собственной службе поиска с помощью [Краткого руководства по импорту данных](search-get-started-portal.md). Этот индекс содержит вымышленные данные об отелях, доступные в качестве встроенного источника данных в каждой службе поиска.

В первом уроке этого учебника создается базовая структура запроса и страница поиска, которые улучшаются в последующих уроках, включая разбиение на страницы, аспекты и автозавершение.

Готовую версию кода можно найти в следующем проекте:

* [1-basic-search-page (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/1-basic-search-page)

Это руководство было обновлено для использования пакета Azure.Search.Documents (версия 11). Сведения о более ранней версии пакета SDK для .NET приведены в [примере кода для Microsoft.Azure.Search (версия 10)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v10).

## <a name="prerequisites"></a>Предварительные требования

* [Создайте службу поиска](search-create-service-portal.md) или [найдите существующую службу](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices).

* Создайте функцию hotels-sample-index, используя инструкции из [Краткого руководства по созданию индекса поиска](search-get-started-portal.md).

* [Visual Studio](https://visualstudio.microsoft.com/)

* [Клиентская библиотека Когнитивного поиска Azure (версия 11)](https://www.nuget.org/packages/Azure.Search.Documents/)

### <a name="install-and-run-the-project-from-github"></a>Установка и запуск проекта из GitHub

Если вы хотите сразу перейти к рабочему приложению, выполните приведенные ниже действия, чтобы скачать и выполнить готовый код.

1. Найдите пример приложения на GitHub: [Создание первого приложения](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11).

1. В [корневой папке](https://github.com/Azure-Samples/azure-search-dotnet-samples) выберите **Code** (Код), а затем — **Clone** (Клонировать) или **Download ZIP** (Скачать ZIP-файл), чтобы создать собственную локальную копию проекта.

1. В Visual Studio откройте базовую страницу поиска этого решения (1-basic-search-page) и выберите **Запуск без отладки** (или нажмите клавишу F5), чтобы выполнить сборку программы и запустить ее.

1. Это индекс гостиниц, поэтому введите несколько слов, которые вы могли бы использовать для поиска гостиниц (например, "wifi", "view", "bar", "parking"), и изучите результаты.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-wifi.png" alt-text="Поиск по запросу &quot;Wi-Fi&quot;" border="true":::

Мы надеемся, что проблем с проектом не возникло и веб-приложение успешно работает. В этом приложении присутствует ряд основных компонентов расширенных поисковых функций, поэтому рекомендуется последовательно выполнить все инструкции из этого руководства. Они приведены в следующих разделах.

## <a name="set-up-a-development-environment"></a>Настройка среды разработки

Чтобы создать этот проект "с нуля" и тем самым закрепить сведения о принципах работы Когнитивного поиска Azure, начните с создания проекта в Visual Studio.

1. В Visual Studio выберите **Создать** > **Проект**, а затем — **Веб-приложение ASP.NET Core**.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-project1.png" alt-text="Создание облачного проекта" border="true":::

1. Присвойте проекту имя, например "FirstSearchApp", и задайте расположение. Нажмите кнопку **создания**.

1. Используйте шаблон проекта **Веб-приложение (модель — представление — контроллер)** .

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-project2.png" alt-text="Создание проекта MVC" border="true":::

1. Установите клиентскую библиотеку. Выберите **Средства** > **Диспетчер пакетов NuGet** > **Управление пакетами NuGet для решения**. Щелкните **Обзор** и выполните поиск пакета "Azure.Search.Documents". Установите пакет **Azure.Search.Documents** (версии 11 или более поздней версии), приняв условия лицензионного соглашения и использования зависимостей.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-nuget-azure.png" alt-text="Добавление библиотек Azure с помощью NuGet" border="true":::

### <a name="initialize-azure-cognitive-search"></a>Инициализация Когнитивного поиска Azure

В этом примере вы используете общедоступные данные об отелях. Это произвольный набор из 50 вымышленных отелей с именами и описаниями, который создан исключительно для предоставления демонстрационных данных. Чтобы получить доступ к этим данным, укажите имя и ключ API.

1. Откройте **appsettings.json** и замените строки по умолчанию URL-адресом службы поиска (в формате `https://<service-name>.search.windows.net`) и [ключом администратора или API](search-security-api-keys.md) службы поиска. Поскольку вам не нужно создавать или обновлять индекс, для выполнения заданий из этого учебника можно использовать ключ запроса.

    ```csharp
    {
        "SearchServiceName": "<YOUR-SEARCH-SERVICE-URI>",
        "SearchServiceQueryApiKey": "<YOUR-SEARCH-SERVICE-API-KEY>"
    }
    ```

1. Выберите файл в обозревателе решений и в окне "Свойства" измените значение параметра **Копировать в выходной каталог** на **Копировать более новые**.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-copy-if-newer.png" alt-text="Копирование параметров приложения в выходные данные" border="true":::

## <a name="model-data-structures"></a>Моделирование структур данных

Модели (классы C#) используются для обмена данными между клиентом (представление), сервером (контроллер) и облаком Azure с использованием архитектуры MVC (модель, представление и контроллер). Обычно эти модели отражают структуру конкретных данных, к которым выполняется доступ.

На этом шаге вы будете моделировать структуры данных индекса поиска, а также строку поиска, используемую при взаимодействии между представлением и контроллером. В индексе гостиниц каждая гостиница содержит много номеров, и у каждой гостиницы есть адрес, состоящий из нескольких частей Полное представление гостиницы является иерархической вложенной структурой данных. Для создания каждого компонента потребуются три класса.

Набор классов **Hotel**, **Address** и **Room** называется [*сложным типом*](search-howto-complex-data-types.md). Это очень важная особенность в Когнитивном поиске Azure. Сложные типы могут иметь много уровней вложенности, классов и подклассов, поддерживая намного более сложные структуры данных, чем *простые типы* (классы, содержащие только примитивные члены).

1. В обозревателе решений щелкните правой кнопкой мыши и выберите **Модели** > **Добавить** > **Новый элемент**.

1. Выберите **Класс** и назовите элемент "Hotel.cs". Замените содержимое файла Hotel.cs кодом, приведенным ниже. Обратите внимание на то, что члены этого класса, поля **Address** и **Room**, сами являются классами, поэтому для них вам тоже нужны модели.

    ```csharp
    using Azure.Search.Documents.Indexes;
    using Azure.Search.Documents.Indexes.Models;
    using Microsoft.Spatial;
    using System;
    using System.Text.Json.Serialization;

    namespace FirstAzureSearchApp.Models
    {
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
    }
    ```

1. Повторите тот же процесс создания модели для класса **Address**, только назовите файл Address.cs. Замените содержимое этого файла следующим кодом.

    ```csharp
    using Azure.Search.Documents.Indexes;

    namespace FirstAzureSearchApp.Models
    {
        public partial class Address
        {
            [SearchableField]
            public string StreetAddress { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string City { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string StateProvince { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string PostalCode { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string Country { get; set; }
        }
    }
    ```

1. И еще раз повторите процедуру для класса **Room**, используя имя файла Room.cs.

    ```csharp
    using Azure.Search.Documents.Indexes;
    using Azure.Search.Documents.Indexes.Models;
    using System.Text.Json.Serialization;

    namespace FirstAzureSearchApp.Models
    {
        public partial class Room
        {
            [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.EnMicrosoft)]
            public string Description { get; set; }

            [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.FrMicrosoft)]
            [JsonPropertyName("Description_fr")]
            public string DescriptionFr { get; set; }

            [SearchableField(IsFilterable = true, IsFacetable = true)]
            public string Type { get; set; }

            [SimpleField(IsFilterable = true, IsFacetable = true)]
            public double? BaseRate { get; set; }

            [SearchableField(IsFilterable = true, IsFacetable = true)]
            public string BedOptions { get; set; }

            [SimpleField(IsFilterable = true, IsFacetable = true)]
            public int SleepsCount { get; set; }

            [SimpleField(IsFilterable = true, IsFacetable = true)]
            public bool? SmokingAllowed { get; set; }

            [SearchableField(IsFilterable = true, IsFacetable = true)]
            public string[] Tags { get; set; }
        }
    }
    ```

1. Последняя модель, которую вы создадите в этом руководстве, — это класс **SearchData**, который представляет входные данные пользователя (**searchText**) и выходные данные поиска (**resultList**). Тип выходных данных (**SearchResult&lt;Hotel&gt;** ) критически важен, так как он точно соответствует результатам поиска, и эту ссылку необходимо передать в представление. Замените шаблон по умолчанию приведенным ниже кодом.

    ```csharp
    using Azure.Search.Documents.Models;

    namespace FirstAzureSearchApp.Models
    {
        public class SearchData
        {
            // The text to search for.
            public string searchText { get; set; }

            // The list of results.
            public SearchResults<Hotel> resultList;
        }
    }
    ```

## <a name="create-a-web-page"></a>Создание веб-страницы

Шаблоны проектов содержат несколько клиентских представлений, расположенных в папке **Views**. Точный состав представлений зависит от используемой версии платформы .NET Core (в данном примере это версия 3.1). В этом руководстве вы измените **Index.cshtml**, чтобы добавить элементы страницы поиска.

Полностью удалите содержимое файла Index.cshtml и создайте его заново, выполнив следующие действия.

1. В этом руководстве используются два небольших изображения в представлении: логотип Azure и значок лупы (azure-logo.png и search.png). Скопируйте эти изображения из проекта GitHub в папку **wwwroot/Images** в своем проекте.

1. Первая строка файла Index.cshtml должна содержать ссылку на модель для обмена данными между клиентом (представлением) и сервером (контроллером). Это созданная ранее модель **SearchData**. Добавьте эту строку в файл Index.cshtml.

    ```csharp
    @model FirstAzureSearchApp.Models.SearchData
    ```

1. Обычно принято присваивать представлениям названия, и для этого добавьте следующие строки:

    ```csharp
    @{
        ViewData["Title"] = "Home Page";
    }
    ```

1. После заголовка введите ссылку на таблицу стилей HTML, которую вы создадите чуть позже.

    ```csharp
    <head>
        <link rel="stylesheet" href="~/css/hotels.css" />
    </head>
    ```

1. Текст представления обрабатывает два варианта использования. Во-первых, при первом вводе искомого текста необходимо предоставить пустую страницу. Во-вторых, помимо текстового поля поиска, нужно обрабатывать результаты для повторных запросов.

   Чтобы реализовать эти два варианта, нужно проверять, содержит ли переданная в представление модель значение NULL. Модель со значением NULL указывает на первый вариант использования (исходный запуск приложения). Добавьте следующий код в файл Index.cshtml и просмотрите комментарии.

    ```csharp
    <body>
    <h1 class="sampleTitle">
        <img src="~/images/azure-logo.png" width="80" />
        Hotels Search
    </h1>

    @using (Html.BeginForm("Index", "Home", FormMethod.Post))
    {
        // Display the search text box, with the search icon to the right of it.
        <div class="searchBoxForm">
            @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox" }) <input class="searchBoxSubmit" type="submit" value="">
        </div>

        @if (Model != null)
        {
            // Show the result count.
            <p class="sampleText">
                @Model.resultList.Count Results
            </p>

            @for (var i = 0; i < Model.resultList.Count; i++)
            {
                // Display the hotel name and description.
                @Html.TextAreaFor(m => m.resultList[i].Document.HotelName, new { @class = "box1" })
                @Html.TextArea($"desc{i}", Model.resultList[i].Document.Description, new { @class = "box2" })
            }
        }
    }
    </body>
    ```

1. Добавьте таблицу стилей. В Visual Studio выберите **Файл**> **Создать** > **Файл**, а затем — **Таблица стилей** (при выделенном параметре **Общие**).

   Замените код по умолчанию приведенным ниже: Мы не будем здесь подробно рассматривать этот файл, так как он содержит стандартные стили HTML.

    ```html
    textarea.box1 {
        width: 648px;
        height: 30px;
        border: none;
        background-color: azure;
        font-size: 14pt;
        color: blue;
        padding-left: 5px;
    }

    textarea.box2 {
        width: 648px;
        height: 100px;
        border: none;
        background-color: azure;
        font-size: 12pt;
        padding-left: 5px;
        margin-bottom: 24px;
    }

    .sampleTitle {
        font: 32px/normal 'Segoe UI Light',Arial,Helvetica,Sans-Serif;
        margin: 20px 0;
        font-size: 32px;
        text-align: left;
    }

    .sampleText {
        font: 16px/bold 'Segoe UI Light',Arial,Helvetica,Sans-Serif;
        margin: 20px 0;
        font-size: 14px;
        text-align: left;
        height: 30px;
    }

    .searchBoxForm {
        width: 648px;
        box-shadow: 0 0 0 1px rgba(0,0,0,.1), 0 2px 4px 0 rgba(0,0,0,.16);
        background-color: #fff;
        display: inline-block;
        border-collapse: collapse;
        border-spacing: 0;
        list-style: none;
        color: #666;
    }

    .searchBox {
        width: 568px;
        font-size: 16px;
        margin: 5px 0 1px 20px;
        padding: 0 10px 0 0;
        border: 0;
        max-height: 30px;
        outline: none;
        box-sizing: content-box;
        height: 35px;
        vertical-align: top;
    }

    .searchBoxSubmit {
        background-color: #fff;
        border-color: #fff;
        background-image: url(/images/search.png);
        background-repeat: no-repeat;
        height: 20px;
        width: 20px;
        text-indent: -99em;
        border-width: 0;
        border-style: solid;
        margin: 10px;
        outline: 0;
    }
    ```

1. Сохраните файл таблицы стилей с именем hotels.css в папку **wwwroot/css** рядом с предложенным по умолчанию файлом site.css.

Этим мы завершаем подготовку представления. На этом этапе модели и представления полностью готовы. Осталось лишь добавить контроллер.

## <a name="define-methods"></a>Определение методов

На этом шаге измените содержимое **HomeController**.

1. Откройте файл HomeController.cs и замените операторы **using** следующим кодом.

    ```csharp
    using Azure;
    using Azure.Search.Documents;
    using Azure.Search.Documents.Indexes;
    using FirstAzureSearchApp.Models;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Configuration;
    using System;
    using System.Diagnostics;
    using System.Linq;
    using System.Threading.Tasks;
    ```

### <a name="add-index-methods"></a>Добавление методов Index

В приложении MVC метод **Index()** является методом действия по умолчанию для любого контроллера. Он открывает HTML-страницу индекса. Метод по умолчанию, который не принимает параметры, используется в этом руководстве для варианта первого запуска приложения, то есть для подготовки пустой страницы поиска.

В этом разделе мы расширяем данный метод для поддержки второго варианта использования, то есть для отображения страницы после того, как пользователь ввел искомый текст. Для поддержки этого варианта метод Index расширяется, чтобы принимать модель в качестве параметра.

1. Добавьте следующий метод после уже существующего метода **Index()** .

    ```csharp
        [HttpPost]
        public async Task<ActionResult> Index(SearchData model)
        {
            try
            {
                // Ensure the search string is valid.
                if (model.searchText == null)
                {
                    model.searchText = "";
                }

                // Make the Azure Cognitive Search call.
                await RunQueryAsync(model);
            }

            catch
            {
                return View("Error", new ErrorViewModel { RequestId = "1" });
            }
            return View(model);
        }
    ```

    Обратите внимание, что метод объявлен как **async** и содержит вызов **await** для **RunQueryAsync**. Эти ключевые слова позволяют выполнять асинхронные вызовы, чтобы избежать блокирования потоков на сервере.

    В блоке **Catch** используется модель ошибок, которая была создана по умолчанию.

### <a name="note-the-error-handling-and-other-default-views-and-methods"></a>Изучение механизма обработки ошибок и других представлений и методов, созданных по умолчанию

В зависимости от используемой версии .NET Core набор создаваемых по умолчанию представлений будет немного различаться. Для платформы .NET Core 3.1 по умолчанию создаются представления Index, Privacy и Error. Вы можете просмотреть эти страницы по умолчанию, запустив приложение, и изучить их обработку в контроллере.

Представление Error вы будете тестировать позже.

В примере на GitHub неиспользуемые представления и связанные с ними действия удалены.

### <a name="add-the-runqueryasync-method"></a>Добавление метода RunQueryAsync

Вызов к Когнитивному поиску Azure инкапсулируется в метод **RunQueryAsync**.

1. Сначала добавьте статические переменные, которые настраивают службу Azure, и вызов для их запуска.

    ```csharp
        private static SearchClient _searchClient;
        private static SearchIndexClient _indexClient;
        private static IConfigurationBuilder _builder;
        private static IConfigurationRoot _configuration;

        private void InitSearch()
        {
            // Create a configuration using appsettings.json
            _builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
            _configuration = _builder.Build();

            // Read the values from appsettings.json
            string searchServiceUri = _configuration["SearchServiceUri"];
            string queryApiKey = _configuration["SearchServiceQueryApiKey"];

            // Create a service and index client.
            _indexClient = new SearchIndexClient(new Uri(searchServiceUri), new AzureKeyCredential(queryApiKey));
            _searchClient = _indexClient.GetSearchClient("hotels");
        }
    ```

2. Теперь добавьте метод **RunQueryAsync**.

    ```csharp
    private async Task<ActionResult> RunQueryAsync(SearchData model)
    {
        InitSearch();

        var options = new SearchOptions() { };

        // Enter Hotel property names into this list so only these values will be returned.
        // If Select is empty, all values will be returned, which can be inefficient.
        options.Select.Add("HotelName");
        options.Select.Add("Description");

        // For efficiency, the search call should be asynchronous, so use SearchAsync rather than Search.
        var searchResult = await _searchClient.SearchAsync<Hotel>(model.searchText, options).ConfigureAwait(false);
        model.resultList = searchResult.Value.GetResults().ToList();

        // Display the results.
        return View("Index", model);
    }
    ```

    В этом методе сначала нужно убедиться, что конфигурация Azure успешно инициирована, а затем — настроить несколько параметров поиска. Параметр **Select** указывает, какие поля должны возвращаться в результатах, что соответствуют именам свойств в классе **hotel**. Если опустить параметр **Select**, возвращаются все нескрытые поля. Это может быть неоптимально, если вас интересует только часть всех возможных полей.

    Асинхронный вызов для поиска формирует запрос (моделируемый как **searchText**) и ответ (моделируемый как **searchResult**). При отладке этого кода удобно добавить точку останова в класс **SearchResult**, если необходимо изучить содержимое **model.resultList**. Вы увидите, что он интуитивно понятен, предоставляет искомые данные и почти ничего сверх того.

### <a name="test-the-app"></a>Тестирование приложения

Теперь давайте проверим, правильно ли выполняется приложение.

1. Выберите **Отладка** > **Запустить без отладки** или нажмите клавишу **F5**. Если приложение выполнится ожидаемым образом, отобразится исходное представление индекса.

     :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-index.png" alt-text="Открытие приложения" border="true":::

1. Введите строку запроса, например "beach" (или любой другой вариант, который придет вам в голову), и щелкните значок поиска, чтобы отправить запрос.

     :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-beach.png" alt-text="Поиск по запросу &quot;пляж&quot;" border="true":::

1. Попробуйте ввести five star (пятизвездочный). Обратите внимание на то, что этот запрос не возвращает результатов. Более сложный поиск будет воспринимать "пятизвездочный" как синоним слова "первоклассный" и возвратит соответствующие результаты. Поддержка [синонимов](search-synonyms.md) доступна в Когнитивном поиске Azure, но не рассматривается в этой серии руководств.

1. Выполните поиск по запросу hot (горячий). В результатах будут _отсутствовать_ записи со словом hotel (отель). Наша функция поиска умеет искать только целые слова, и результатов здесь будет мало.

1. Поэкспериментируйте с разными словами: pool (бассейн), sunshiny (солнечный), view (вид) и так далее. Вы наверняка убедитесь, что Когнитивный поиск Azure выдает правильный, хотя и самый простой, результат.

## <a name="test-edge-conditions-and-errors"></a>Проверка пограничных условий и ошибок

Важно убедиться, что все функции обработки ошибок работают правильно. 

1. В методе **Index** после вызова **try {** введите строку **Throw new Exception()** . Это исключение создаст ошибку при поиске любого текста.

2. Откройте приложение, введите для поиска слово bar и щелкните значок поиска. Исключение сразу откроет представление ошибки.

     :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-error.png" alt-text="Принудительный вызов ошибки" border="true":::

    > [!Important]
    > Считается, что отображение внутренних номеров ошибок на страницах ошибок, несет угрозу безопасности. Если вы планируете сделать это приложение общедоступным, изучите рекомендации по безопасности для отображения сообщений об ошибках.

3. Удалите строку **Throw new Exception()** , убедившись в том, что ошибки обрабатываются как положено.

## <a name="takeaways"></a>Общие выводы

По результатам этого проекта можно сделать такие выводы:

* Вызов службы "Когнитивный поиск Azure" и интерпретация результатов не представляют никаких сложностей.
* Асинхронный вызов немного усложняет контроллер, но это правильный подход к разработке качественных приложений.
* Данное приложение выполняет простой текстовый поиск в соответствии с настройками в **searchOptions**. Но наш единственный класс может быть заполнен большим числом элементов, что повышает сложность поиска. Чтобы существенно увеличить возможности нашего приложения, не потребуется много усилий.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы повысить удобство работы пользователей, можно добавить еще несколько функций, в частности разбивку на страницы (по номерам страниц или с бесконечной прокруткой) и автозаполнение с поисковыми подсказками. Вы также можете подумать над усложнением параметров поиска (например, возможностью поиска отелей в радиусе от заданной точки или упорядочением результатов поиска).

Это рассматривается в оставшихся руководствах серии. Давайте начнем с разбивки на страницы.

> [!div class="nextstepaction"]
> [Руководство для разработчиков C#. Разбиение результатов поиска на страницы — Когнитивный поиск Azure](tutorial-csharp-paging.md)