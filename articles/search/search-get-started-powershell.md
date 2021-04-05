---
title: Краткое руководство. Создание индекса поиска в PowerShell с помощью REST API
titleSuffix: Azure Cognitive Search
description: В этом кратком руководстве приведено описание процессов создания индекса, загрузки данных и выполнения запросов с помощью командлета Invoke-RestMethod PowerShell и REST API службы "Когнитивный поиск Azure".
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: quickstart
ms.devlang: rest-api
ms.date: 11/17/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9dd9de9dcb01e9be200e07e5925d8b856432b620
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94742384"
---
# <a name="quickstart-create-an-azure-cognitive-search-index-in-powershell-using-rest-apis"></a>Краткое руководство. Создание индекса службы "Когнитивный поиск Azure" в PowerShell с помощью REST API
> [!div class="op_single_selector"]
> * [PowerShell (REST)]()
> * [C#](./search-get-started-dotnet.md)
> * [REST](search-get-started-rest.md)
> * [Python](search-get-started-python.md)
> * [Портал](search-get-started-portal.md)
> 

Эта статья поможет вам создать, загрузить и запросить индекс службы "Когнитивный поиск Azure" с помощью PowerShell и [REST API службы "Когнитивный поиск Azure"](/rest/api/searchservice/). Здесь содержатся сведения о выполнении команд PowerShell в интерактивном режиме. Вы также можете [скачать и запустить скрипт PowerShell](https://github.com/Azure-Samples/azure-search-powershell-samples/tree/master/Quickstart), выполняющий те же операции.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Для выполнения инструкций из этого краткого руководства необходимы перечисленные ниже службы и инструменты. 

+ [Средство PowerShell 5.1 или более поздней версии](https://github.com/PowerShell/PowerShell), использующее [Invoke-RestMethod](/powershell/module/Microsoft.PowerShell.Utility/Invoke-RestMethod) для выполнения последовательных и интерактивных шагов.

+ [Создайте службу "Когнитивный поиск Azure"](search-create-service-portal.md) или [найдите имеющуюся службу](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) в рамках текущей подписки. Вы можете использовать бесплатную службу для выполнения инструкций, описанных в этом кратком руководстве. 

## <a name="copy-a-key-and-url"></a>Копирование ключа и URL-адреса

Вызовам REST требуется URL-адрес службы и ключ доступа при каждом запросе. Служба поиска создана с обоими элементами, поэтому если вы добавили службу "Когнитивный поиск Azure" в подписку, выполните следующие действия для получения необходимых сведений:

1. [Войдите на портал Azure](https://portal.azure.com/) и на странице **обзора** службы поиска получите URL-адрес. Пример конечной точки может выглядеть так: `https://mydemo.search.windows.net`.

2. В разделе **Параметры** > **Ключи** получите ключ администратора, чтобы обрести полные права на службу. Существуют два взаимозаменяемых ключа администратора, предназначенных для обеспечения непрерывности бизнес-процессов на случай, если вам потребуется сменить один из них. Вы можете использовать первичный или вторичный ключ для выполнения запросов на добавление, изменение и удаление объектов.

![Получение конечной точки HTTP и ключа доступа](media/search-get-started-rest/get-url-key.png "Получение конечной точки HTTP и ключа доступа")

Для выполнения любого запроса к службе требуется использование ключа API. Если есть действительный ключ, для каждого запроса устанавливаются отношения доверия между приложением, которое отправляет запрос, и службой, которая его обрабатывает.

## <a name="connect-to-azure-cognitive-search"></a>Подключение к Когнитивному поиску Azure

1. В PowerShell создайте объект **$headers** для хранения типа содержимого и ключа API. Замените ключ API администратора (YOUR-ADMIN-API-KEY) на ключ, который является допустимым для вашей службы поиска. Этот заголовок необходимо указать только один раз в течение сеанса, но он будет добавлен к каждому запросу. 

    ```powershell
    $headers = @{
    'api-key' = '<YOUR-ADMIN-API-KEY>'
    'Content-Type' = 'application/json' 
    'Accept' = 'application/json' }
    ```

2. Создайте объект **$url**, который определяет коллекцию индексов службы. Замените имя службы (YOUR-SEARCH-SERVICE-NAME) на допустимую службу поиска.

    ```powershell
    $url = "https://<YOUR-SEARCH-SERVICE-NAME>.search.windows.net/indexes?api-version=2020-06-30&$select=name"
    ```

3. Запустите командлет **Invoke-RestMethod**, чтобы отправить запрос GET в службу и проверить подключение. Добавьте командлет **ConvertTo-Json**, чтобы просматривать ответы, отправленные из службы.

    ```powershell
    Invoke-RestMethod -Uri $url -Headers $headers | ConvertTo-Json
    ```

   Если индексы в службе отсутствуют, результаты будут аналогичны следующему примеру. В противном случае вы увидите JSON-представление определений индекса.

    ```
    {
        "@odata.context":  "https://mydemo.search.windows.net/$metadata#indexes",
        "value":  [

                ]
    }
    ```

## <a name="1---create-an-index"></a>1\. Создание индекса

Если вы не используете портал, для загрузки данных необходимо наличие индекса в службе. Этот шаг определяет индекс и передает его в службу. Для этого используется [REST API для создания индекса](/rest/api/searchservice/create-index).

Обязательные элементы индекса включают в себя имя и коллекцию полей. Коллекция полей определяет структуру *документа*. Каждое поле имеет имя, тип и атрибуты, которые определяют его использование (например, обладает ли оно возможностью полнотекстового поиска, можно ли его отфильтровать или извлечь из результатов поиска). В индексе одно из полей типа `Edm.String` должно быть назначено в качестве *ключа* для идентификации документа.

Этот индекс называется hotels-quickstart и содержит приведенные ниже определения полей. Это подмножество более крупного [индекса Hotels](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotels/Hotels_IndexDefinition.JSON), используемого в других статьях. В этом кратком руководстве определения полей были усечены для краткости.

1. Вставьте этот пример в PowerShell, чтобы создать объект **$body**, содержащий схему индекса.

    ```powershell
    $body = @"
    {
        "name": "hotels-quickstart",  
        "fields": [
            {"name": "HotelId", "type": "Edm.String", "key": true, "filterable": true},
            {"name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": true, "facetable": false},
            {"name": "Description", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "en.lucene"},
            {"name": "Category", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true},
            {"name": "Tags", "type": "Collection(Edm.String)", "searchable": true, "filterable": true, "sortable": false, "facetable": true},
            {"name": "ParkingIncluded", "type": "Edm.Boolean", "filterable": true, "sortable": true, "facetable": true},
            {"name": "LastRenovationDate", "type": "Edm.DateTimeOffset", "filterable": true, "sortable": true, "facetable": true},
            {"name": "Rating", "type": "Edm.Double", "filterable": true, "sortable": true, "facetable": true},
            {"name": "Address", "type": "Edm.ComplexType", 
            "fields": [
            {"name": "StreetAddress", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false, "searchable": true},
            {"name": "City", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true},
            {"name": "StateProvince", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true},
            {"name": "PostalCode", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true},
            {"name": "Country", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true}
            ]
         }
      ]
    }
    "@
    ```

2. Задайте универсальный код ресурса (URI) для коллекции индексов вашей службы и индекса *hotels-quickstart*.

    ```powershell
    $url = "https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/hotels-quickstart?api-version=2020-06-30"
    ```

3. Запустите команду с **$url**, **$headers** и **$body**, чтобы создать индекс службы. 

    ```powershell
    Invoke-RestMethod -Uri $url -Headers $headers -Method Put -Body $body | ConvertTo-Json
    ```

    Результаты должны выглядеть следующим образом (для краткости показаны только два первых поля):

    ```
    {
        "@odata.context":  "https://mydemo.search.windows.net/$metadata#indexes/$entity",
        "@odata.etag":  "\"0x8D6EDE28CFEABDA\"",
        "name":  "hotels-quickstart",
        "defaultScoringProfile":  null,
        "fields":  [
                    {
                        "name":  "HotelId",
                        "type":  "Edm.String",
                        "searchable":  true,
                        "filterable":  true,
                        "retrievable":  true,
                        "sortable":  true,
                        "facetable":  true,
                        "key":  true,
                        "indexAnalyzer":  null,
                        "searchAnalyzer":  null,
                        "analyzer":  null,
                        "synonymMaps":  ""
                    },
                    {
                        "name":  "HotelName",
                        "type":  "Edm.String",
                        "searchable":  true,
                        "filterable":  false,
                        "retrievable":  true,
                        "sortable":  true,
                        "facetable":  false,
                        "key":  false,
                        "indexAnalyzer":  null,
                        "searchAnalyzer":  null,
                        "analyzer":  null,
                        "synonymMaps":  ""
                    },
    . . .
    ```

> [!Tip]
> Кроме того, вы можете просмотреть список индексов на портале.

<a name="load-documents"></a>

## <a name="2---load-documents"></a>2\. Загрузка документов

Чтобы передать документы, отправьте запрос HTTP POST в конечную точку URL-адреса индекса. В этой задаче используется REST API для [добавления, обновления и удаления документов](/rest/api/searchservice/addupdate-or-delete-documents).

1. Вставьте этот пример в PowerShell, чтобы создать объект **$body**, содержащий документы для передачи. 

    Этот запрос включает в себя две полные записи и одну частичную. Частичная запись демонстрирует возможность загрузки неполных документов. Параметр `@search.action` указывает способ выполнения индексации. Допустимые значения: upload, merge, mergeOrUpload и delete. Поведение mergeOrUpload создает новый документ для hotelId = 3 либо обновляет имеющееся содержимое.

    ```powershell
    $body = @"
    {
        "value": [
        {
        "@search.action": "upload",
        "HotelId": "1",
        "HotelName": "Secret Point Motel",
        "Description": "The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Time's Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.",
        "Category": "Boutique",
        "Tags": [ "pool", "air conditioning", "concierge" ],
        "ParkingIncluded": false,
        "LastRenovationDate": "1970-01-18T00:00:00Z",
        "Rating": 3.60,
        "Address": 
            {
            "StreetAddress": "677 5th Ave",
            "City": "New York",
            "StateProvince": "NY",
            "PostalCode": "10022",
            "Country": "USA"
            } 
        },
        {
        "@search.action": "upload",
        "HotelId": "2",
        "HotelName": "Twin Dome Motel",
        "Description": "The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.",
        "Category": "Boutique",
        "Tags": [ "pool", "free wifi", "concierge" ],
        "ParkingIncluded": false,
        "LastRenovationDate": "1979-02-18T00:00:00Z",
        "Rating": 3.60,
        "Address": 
            {
            "StreetAddress": "140 University Town Center Dr",
            "City": "Sarasota",
            "StateProvince": "FL",
            "PostalCode": "34243",
            "Country": "USA"
            } 
        },
        {
        "@search.action": "upload",
        "HotelId": "3",
        "HotelName": "Triple Landscape Hotel",
        "Description": "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel’s restaurant services.",
        "Category": "Resort and Spa",
        "Tags": [ "air conditioning", "bar", "continental breakfast" ],
        "ParkingIncluded": true,
        "LastRenovationDate": "2015-09-20T00:00:00Z",
        "Rating": 4.80,
        "Address": 
            {
            "StreetAddress": "3393 Peachtree Rd",
            "City": "Atlanta",
            "StateProvince": "GA",
            "PostalCode": "30326",
            "Country": "USA"
            } 
        },
        {
        "@search.action": "upload",
        "HotelId": "4",
        "HotelName": "Sublime Cliff Hotel",
        "Description": "Sublime Cliff Hotel is located in the heart of the historic center of Sublime in an extremely vibrant and lively area within short walking distance to the sites and landmarks of the city and is surrounded by the extraordinary beauty of churches, buildings, shops and monuments. Sublime Cliff is part of a lovingly restored 1800 palace.",
        "Category": "Boutique",
        "Tags": [ "concierge", "view", "24-hour front desk service" ],
        "ParkingIncluded": true,
        "LastRenovationDate": "1960-02-06T00:00:00Z",
        "Rating": 4.60,
        "Address": 
            {
            "StreetAddress": "7400 San Pedro Ave",
            "City": "San Antonio",
            "StateProvince": "TX",
            "PostalCode": "78216",
            "Country": "USA"
            }
        }
    ]
    }
    "@
    ```

1. Задайте в качестве конечной точки коллекцию документов *hotels-quickstart* и добавьте операцию индекса (indexes/hotels-quickstart/docs/index).

    ```powershell
    $url = "https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/hotels-quickstart/docs/index?api-version=2020-06-30"
    ```

1. Запустите команду с **$url**, **$headers** и **$body**, чтобы загрузить документы в индекс hotels-quickstart.

    ```powershell
    Invoke-RestMethod -Uri $url -Headers $headers -Method Post -Body $body | ConvertTo-Json
    ```
    Результаты должны выглядеть следующим образом. Вы должны увидеть [код состояния 201](/rest/api/searchservice/HTTP-status-codes).

    ```
    {
        "@odata.context":  "https://mydemo.search.windows.net/indexes(\u0027hotels-quickstart\u0027)/$metadata#Collection(Microsoft.Azure.Search.V2019_05_06.IndexResult)",
        "value":  [
                    {
                        "key":  "1",
                        "status":  true,
                        "errorMessage":  null,
                        "statusCode":  201
                    },
                    {
                        "key":  "2",
                        "status":  true,
                        "errorMessage":  null,
                        "statusCode":  201
                    },
                    {
                        "key":  "3",
                        "status":  true,
                        "errorMessage":  null,
                        "statusCode":  201
                    },
                    {
                        "key":  "4",
                        "status":  true,
                        "errorMessage":  null,
                        "statusCode":  201
                    }
                ]
    }
    ```

## <a name="3---search-an-index"></a>3\. Поиск в индексе

В рамках этого раздела мы покажем, как создать запрос в индекс с помощью [API поиска документов](/rest/api/searchservice/search-documents).

Обязательно используйте одинарные кавычки при поиске $urls. Строки запроса содержат символы **$** . Их можно использовать, если вся строка заключена в одинарные кавычки.

1. Задайте в качестве конечной точки коллекцию документов *hotels-quickstart* и добавьте параметр **search** для передачи в строку запроса. 
  
   Эта строка выполняет пустой поиск (search=*), возвращая неупорядоченный список (search score = 1.0) произвольных документов. По умолчанию служба "Когнитивный поиск Azure" возвращает 50 лучших соответствий за раз. Как структурированный этот запрос возвращает структуру и значения документа. Добавьте **$count=true**, чтобы получить количество всех документов в результатах.

    ```powershell
    $url = 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/hotels-quickstart/docs?api-version=2020-06-30&search=*&$count=true'
    ```

1. Запустите команду, чтобы отправить **$url** в службу.

    ```powershell
    Invoke-RestMethod -Uri $url -Headers $headers | ConvertTo-Json
    ```

    Результаты должны выглядеть следующим образом.

    ```
    {
    "@odata.context":  "https://mydemo.search.windows.net/indexes(\u0027hotels-quickstart\u0027)/$metadata#docs(*)",
    "@odata.count":  4,
    "value":  [
                  {
                      "@search.score":  0.1547872,
                      "HotelId":  "2",
                      "HotelName":  "Twin Dome Motel",
                      "Description":  "The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.",
                      "Category":  "Boutique",
                      "Tags":  "pool free wifi concierge",
                      "ParkingIncluded":  false,
                      "LastRenovationDate":  "1979-02-18T00:00:00Z",
                      "Rating":  3.6,
                      "Address":  "@{StreetAddress=140 University Town Center Dr; City=Sarasota; StateProvince=FL; PostalCode=34243; Country=USA}"
                  },
                  {
                      "@search.score":  0.009068266,
                      "HotelId":  "3",
                      "HotelName":  "Triple Landscape Hotel",
                      "Description":  "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel\u0027s restaurant services.",
                      "Category":  "Resort and Spa",
                      "Tags":  "air conditioning bar continental breakfast",
                      "ParkingIncluded":  true,
                      "LastRenovationDate":  "2015-09-20T00:00:00Z",
                      "Rating":  4.8,
                      "Address":  "@{StreetAddress=3393 Peachtree Rd; City=Atlanta; StateProvince=GA; PostalCode=30326; Country=USA}"
                  },
                . . . 
    ```

Выполните еще несколько примеров запросов, чтобы ознакомиться с синтаксисом. Вы можете выполнять поиск по строкам, буквальные запросы $filter, ограничивать наборы результатов, осуществлять поиск только по определенным полям и многое другое.

```powershell
# Query example 1
# Search the entire index for the terms 'restaurant' and 'wifi'
# Return only the HotelName, Description, and Tags fields
$url = 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/hotels-quickstart/docs?api-version=2020-06-30&search=restaurant wifi&$count=true&$select=HotelName,Description,Tags'

# Query example 2 
# Apply a filter to the index to find hotels rated 4 or higher
# Returns the HotelName and Rating. Two documents match.
$url = 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/hotels-quickstart/docs?api-version=2020-06-30&search=*&$filter=Rating gt 4&$select=HotelName,Rating'

# Query example 3
# Take the top two results, and show only HotelName and Category in the results
$url = 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/hotels-quickstart/docs?api-version=2020-06-30&search=boutique&$top=2&$select=HotelName,Category'

# Query example 4
# Sort by a specific field (Address/City) in ascending order

$url = 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/hotels-quickstart/docs?api-version=2020-06-30&search=pool&$orderby=Address/City asc&$select=HotelName, Address/City, Tags, Rating'
```
## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы работаете в своей подписке, по окончании проекта рекомендуем решить, нужны ли вам созданные ресурсы. Работающие ресурсы могут означать лишние затраты. Можно удалить отдельные ресурсы или удалить группу ресурсов, что позволит удалить весь набор ресурсов.

Просматривать ресурсы и управлять ими можно на портале с помощью ссылок **Все ресурсы** или **Группы ресурсов** на панели навигации слева.

При работе с бесплатной версией службы помните о том, что вам доступно максимум три индекса, индексатора и источника данных. Вы можете удалить отдельные элементы на портале, чтобы не превысить лимит. 

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого краткого руководства вы использовали PowerShell для выполнения базового рабочего процесса создания содержимого в службе "Когнитивный поиск Azure" и получения к нему доступа. Помня об этих концепциях, мы рекомендуем перейти к изучению более сложных сценариев, таких как индексирование из источников данных Azure.

> [!div class="nextstepaction"]
> [Руководство по использованию REST. Индексирование и поиск частично структурированных данных (больших двоичных объектов JSON) в службе "Когнитивный поиск Azure"](search-semi-structured-data.md)
