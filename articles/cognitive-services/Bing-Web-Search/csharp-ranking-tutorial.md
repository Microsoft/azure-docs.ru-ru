---
title: Использование ранжирования для отображения результатов поиска
titleSuffix: Azure Cognitive Services
description: Сведения об использовании ответа RankingResponse в Bing для отображения результатов поиска в определенном порядке.
services: cognitive-services
author: aahill
manager: nitinme
ms.assetid: 2575A80C-FC74-4631-AE5D-8101CF2591D3
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: tutorial
ms.date: 06/24/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: 907899d8c5be2cbba7ff3eee4a59713718357a5c
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102430094"
---
# <a name="build-a-console-app-search-client-in-c"></a>Создание клиента поиска в виде консольного приложения на C#

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

В этом руководстве объясняется, как создать простое консольное приложение .NET Core, с помощью которого пользователи могут отправлять запросы в API Bing для поиска в Интернете и отображает ранжированные результаты.

В этом учебнике описаны следующие действия:

- выполнение простого запроса к API Bing для поиска в Интернете;
- отображение результатов запроса в порядке приоритета.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуется следующее:

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Получив подписку Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="создайте ресурс Поиска Bing"  target="_blank">Create a Bing Search resource </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
* [Интегрированная среда разработки Visual Studio](https://www.visualstudio.com/downloads/).

## <a name="create-a-new-console-app-project"></a>Создание проекта консольного приложения

В Visual Studio создайте проект с помощью клавиш `Ctrl`+`Shift`+`N`.

В диалоговом окне **Создать проект** выберите **Visual C# > Классический рабочий стол Windows > Консольное приложение (.NET Framework)** .

Присвойте приложению имя **MyConsoleSearchApp** и нажмите кнопку **ОК**.

## <a name="add-the-jsonnet-nuget-package-to-the-project"></a>Добавление в проект пакета NuGet для JSON.NET

Json.NET позволяет работать с ответами JSON, которые возвращает API. Добавьте в проект этот пакет NuGet.

- В **обозревателе решений** щелкните проект правой кнопкой мыши и выберите пункт **Управление пакетами NuGet...** .
- На вкладке **Обзор** выполните поиск по `Newtonsoft.Json`. Выберите последнюю версию и щелкните **Установить**.
- Нажмите кнопку **ОК** в окне **Просмотреть изменения**.
- Закройте вкладку Visual Studio **NuGet: MyConsoleSearchApp**.

## <a name="add-a-reference-to-systemweb"></a>Добавление ссылки на System.Web

В этом руководстве используется сборка `System.Web`. Добавьте в проект ссылку на эту сборку.

- В **обозревателе решений** щелкните правой кнопкой мыши **Ссылки** и выберите **Добавить ссылку...** .
- Выберите **Сборки > Платформа**, затем прокрутите вниз и установите флажок **System.Web**.
- Нажмите кнопку **ОК**.

## <a name="add-some-necessary-using-statements"></a>Добавление необходимых операторов using

Чтобы работал код из этого руководства, нужны три дополнительных оператора using. Добавьте эти операторы после существующих операторов `using` в верхней части файла **Program.cs**:

```csharp
using System.Web;
using System.Net.Http;
```

## <a name="ask-the-user-for-a-query"></a>Предложение на ввод запроса

В **обозревателе решений** откройте **Program.cs**. Внесите изменения в метод `Main()`:

```csharp
static void Main()
{
    // Get the user's query
    Console.Write("Enter Bing query: ");
    string userQuery = Console.ReadLine();
    Console.WriteLine();

    // Run the query and display the results
    RunQueryAndDisplayResults(userQuery);

    // Prevent the console window from closing immediately
    Console.WriteLine("\nHit ENTER to exit...");
    Console.ReadLine();
}
```

Этот метод выполняет следующее:

- предлагает пользователю ввести запрос;
- вызывает `RunQueryAndDisplayResults(userQuery)` для выполнения запроса и отображения результатов;
- ожидает ввода данных пользователем, чтобы окно консоли не закрывалось сразу после вывода.

## <a name="search-for-query-results-using-the-bing-web-search-api"></a>Поиск результатов запроса с помощью API Bing для поиска в Интернете

Добавьте метод, который передает запрос в API и отображает результаты.

```csharp
static void RunQueryAndDisplayResults(string userQuery)
{
    try
    {
        // Create a query
        var client = new HttpClient();
        client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "<YOUR_SUBSCRIPTION_KEY_GOES_HERE>");
        var queryString = HttpUtility.ParseQueryString(string.Empty);
        queryString["q"] = userQuery;
        var query = "https://api.cognitive.microsoft.com/bing/v7.0/search?" + queryString;

        // Run the query
        HttpResponseMessage httpResponseMessage = client.GetAsync(query).Result;

        // Deserialize the response content
        var responseContentString = httpResponseMessage.Content.ReadAsStringAsync().Result;
        Newtonsoft.Json.Linq.JObject responseObjects = Newtonsoft.Json.Linq.JObject.Parse(responseContentString);

        // Handle success and error codes
        if (httpResponseMessage.IsSuccessStatusCode)
        {
            DisplayAllRankedResults(responseObjects);
        }
        else
        {
            Console.WriteLine($"HTTP error status code: {httpResponseMessage.StatusCode.ToString()}");
        }
    }
    catch (Exception e)
    {
        Console.WriteLine(e.Message);
    }
}
```

Этот метод выполняет следующее:

- создает `HttpClient` для отправки запроса к API поиска в Интернете;
- задает заголовок HTTP `Ocp-Apim-Subscription-Key`, по которому Bing проверяет подлинность запроса;
- выполняет запрос и использует JSON.net для десериализации результатов;
- вызывает `DisplayAllRankedResults(responseObjects)` для отображения результатов в порядке приоритета.

Не забудьте задать правильное значение ключа подписки в параметре `Ocp-Apim-Subscription-Key`.

## <a name="display-ranked-results"></a>Отображение ранжированных результатов

Перед тем, как переходить к отображению результатов в порядке приоритета, давайте рассмотрим пример ответа на простой запрос поиска в Интернете:

```json
{
    "_type" : "SearchResponse",
    "webPages" : {
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=70BE289346...",
        "totalEstimatedMatches" : 982000,
        "value" : [{
            "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.0",
            "name" : "Contoso Sailing Club - Seattle",
            "url" : "https:\/\/www.bing.com\/cr?IG=70BE289346ED4594874FE...",
            "displayUrl" : "https:\/\/contososailingsea...",
            "snippet" : "Come sail with Contoso in Seattle...",
            "dateLastCrawled" : "2017-04-07T02:25:00"
        },
        {
            "id" : "https:\/\/api.cognitive.microsoft.com\/api\/7\/#WebPages.6",
            "name" : "Contoso Sailing Lessons - Official Site",
            "url" : "http:\/\/www.bing.com\/cr?IG=70BE289346ED4594874FE...",
            "displayUrl" : "https:\/\/www.constososailinglessonsseat...",
            "snippet" : "Contoso sailing lessons in Seattle...",
            "dateLastCrawled" : "2017-04-09T14:30:00"
        },

        ...

        ],
        "someResultsRemoved" : true
    },
    "relatedSearches" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/7\/#RelatedSearches",
        "value" : [{
            "text" : "sailing lessons",
            "displayText" : "sailing lessons",
            "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=70BE289346E..."
        }

        ...

        ]
    },
    "rankingResponse" : {
        "mainline" : {
            "items" : [{
                "answerType" : "WebPages",
                "resultIndex" : 0,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.0"
                }
            },
            {
                "answerType" : "WebPages",
                "resultIndex" : 1,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.1"
                }
            }

            ...

            ]
        },
        "sidebar" : {
            "items" : [{
                "answerType" : "RelatedSearches",
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#RelatedSearches"
                }
            }]
        }
    }
}
```

Объект JSON `rankingResponse` (подробнее описан в [документации](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#rankingresponse)) предоставляет правильный порядок отображения для результатов поиска. Он содержит одну или несколько следующих групп с учетом приоритета:

- `pole`: Результаты поиска, которые должны отображаться наиболее заметным образом (например, над основной частью и боковой панелью).
- `mainline`: Результаты поиска для отображения в основном поле.
- `sidebar`: Результаты поиска для отображения на боковой панели. Если боковая панель отсутствует, эти результаты отображаются под основной частью.

JSON с ранжированным ответом может содержать одну или несколько этих групп.

Добавьте в файл **Program.cs** следующий метод для отображения результатов в порядке приоритета:

```csharp
static void DisplayAllRankedResults(Newtonsoft.Json.Linq.JObject responseObjects)
{
    string[] rankingGroups = new string[] { "pole", "mainline", "sidebar" };

    // Loop through the ranking groups in priority order
    foreach (string rankingName in rankingGroups)
    {
        Newtonsoft.Json.Linq.JToken rankingResponseItems = responseObjects.SelectToken($"rankingResponse.{rankingName}.items");
        if (rankingResponseItems != null)
        {
            foreach (Newtonsoft.Json.Linq.JObject rankingResponseItem in rankingResponseItems)
            {
                Newtonsoft.Json.Linq.JToken resultIndex;
                rankingResponseItem.TryGetValue("resultIndex", out resultIndex);
                var answerType = rankingResponseItem.Value<string>("answerType");
                switch (answerType)
                {
                    case "WebPages":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("webPages.value"), "WebPage", "name", "url", "displayUrl", "snippet");
                        break;
                    case "News":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("news.value"), "News", "name", "url", "description");
                        break;
                    case "Images":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("images.value"), "Image", "thumbnailUrl");
                        break;
                    case "Videos":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("videos.value"), "Video", "embedHtml");
                        break;
                    case "RelatedSearches":
                        DisplaySpecificResults(resultIndex, responseObjects.SelectToken("relatedSearches.value"), "RelatedSearch", "displayText", "webSearchUrl");
                        break;
                }
            }
        }
    }
}
```

Этот метод выполняет следующее:

- в цикле обрабатывает группы `rankingResponse`, которые содержит ответ;
- отображает элементы каждой из групп, вызывая `DisplaySpecificResults(...)`.

Добавьте в файл **Program.cs** следующие два метода:

```csharp
static void DisplaySpecificResults(Newtonsoft.Json.Linq.JToken resultIndex, Newtonsoft.Json.Linq.JToken items, string title, params string[] fields)
{
    if (resultIndex == null)
    {
        foreach (Newtonsoft.Json.Linq.JToken item in items)
        {
            DisplayItem(item, title, fields);
        }
    }
    else
    {
        DisplayItem(items.ElementAt((int)resultIndex), title, fields);
    }
}

static void DisplayItem(Newtonsoft.Json.Linq.JToken item, string title, string[] fields)
{
    Console.WriteLine($"{title}: ");
    foreach( string field in fields )
    {
        Console.WriteLine($"- {field}: {item[field]}");
    }
    Console.WriteLine();
}
```

Эти методы работают совместно, чтобы вывести в консоль результаты поиска.

## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение. Результат должен выглядеть следующим образом.

```
Enter Bing query: sailing lessons seattle

WebPage:
- name: Contoso Sailing Club - Seattle
- url: https://www.bing.com/cr?IG=70BE289346ED4594874FE...
- displayUrl: https://contososailingsea....
- snippet: Come sail with Contoso in Seattle...

WebPage:
- name: Contoso Sailing Lessons Seattle - Official Site
- url: http://www.bing.com/cr?IG=70BE289346ED4594874FE...
- displayUrl: https://www.constososailinglessonsseat...
- snippet: Contoso sailing lessons in Seattle...

...
```

## <a name="next-steps"></a>Дальнейшие действия

Изучите дополнительные сведения [об использовании ранжирования при отображении ответов](rank-results.md).