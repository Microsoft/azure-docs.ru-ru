---
title: Краткое руководство. Создание индекса поиска в Python
titleSuffix: Azure Cognitive Search
description: Сведения о том, как создать индекс, загрузить данные и выполнять запросы с помощью Python, записных книжек Jupyter Notebook и клиентской библиотеки Azure.Documents.Search для Python.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: quickstart
ms.date: 03/12/2021
ms.custom: devx-track-python
ms.openlocfilehash: 8b9c4792fa6dbdc70f657ce3c5f1757473a22fda
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103225223"
---
# <a name="quickstart-create-an-azure-cognitive-search-index-in-python-using-jupyter-notebook"></a>Краткое руководство. Создание индекса службы "Когнитивный поиск Azure" в Python с помощью записных книжек Jupyter Notebook

> [!div class="op_single_selector"]
> * [Python](search-get-started-python.md)
> * [PowerShell (REST)](search-get-started-powershell.md)
> * [C#](search-get-started-dotnet.md)
> * [REST](search-get-started-rest.md)
> * [Портал](search-get-started-portal.md)
>

Создайте записную книжку, которая создает, загружает и запрашивает индекс Когнитивного поиска Azure с помощью Python и [библиотеки azure-search-documents](/python/api/overview/azure/search-documents-readme) в пакете SDK Azure для Python. В этой статье содержатся сведения о пошаговом создании записной книжки. Вы также можете [скачать и запустить готовую записную книжку Jupyter Python](https://github.com/Azure-Samples/azure-search-python-samples).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Для выполнения инструкций из этого краткого руководства необходимы перечисленные ниже службы и инструменты.

* [Anaconda 3.x](https://www.anaconda.com/distribution/#download-section) с Python 3.x и записными книжками Jupyter Notebook.

* [пакет azure-search-documents](https://pypi.org/project/azure-search-documents/)

* [Создайте службу поиска](search-create-service-portal.md) или [найдите имеющуюся службу](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) в рамках текущей подписки. Для выполнения инструкций, описанных в этом кратком руководстве, можно использовать уровень "Бесплатный". 

## <a name="copy-a-key-and-url"></a>Копирование ключа и URL-адреса

Вызовам REST требуется URL-адрес службы и ключ доступа при каждом запросе. Служба поиска создана с обоими элементами, поэтому если вы добавили службу "Когнитивный поиск Azure" в подписку, выполните следующие действия для получения необходимых сведений:

1. [Войдите на портал Azure](https://portal.azure.com/) и на странице **обзора** службы поиска получите URL-адрес. Пример конечной точки может выглядеть так: `https://mydemo.search.windows.net`.

1. В разделе **Параметры** > **Ключи** получите ключ администратора, чтобы обрести полные права на службу. Существуют два взаимозаменяемых ключа администратора, предназначенных для обеспечения непрерывности бизнес-процессов на случай, если вам потребуется сменить один из них. Вы можете использовать первичный или вторичный ключ для выполнения запросов на добавление, изменение и удаление объектов.

   ![Получение конечной точки HTTP и ключа доступа](media/search-get-started-rest/get-url-key.png "Получение конечной точки HTTP и ключа доступа")

Для выполнения любого запроса к службе требуется использование ключа API. Если есть действительный ключ, для каждого запроса устанавливаются отношения доверия между приложением, которое отправляет запрос, и службой, которая его обрабатывает.

## <a name="connect-to-azure-cognitive-search"></a>Подключение к Когнитивному поиску Azure

Для выполнения этой задачи запустите записную книжку Jupyter Notebook и проверьте, можете ли вы подключиться к службе "Когнитивный поиск Azure". Для этого запросите список индексов из службы. В Windows с Anaconda3 для запуска записной книжки можно использовать Anaconda Navigator.

1. Создайте записную книжку Python3.

1. В первой ячейке загрузите библиотеки из пакета SDK Azure для Python, включая [azure-search-documents](/python/api/azure-search-documents).

   ```python
    !pip install azure-search-documents --pre
    !pip show azure-search-documents
    
    import os
    from azure.core.credentials import AzureKeyCredential
    from azure.search.documents.indexes import SearchIndexClient 
    from azure.search.documents import SearchClient
    from azure.search.documents.indexes.models import (
        ComplexField,
        CorsOptions,
        SearchIndex,
        ScoringProfile,
        SearchFieldDataType,
        SimpleField,
        SearchableField
    )
   ```

1. Во второй ячейке введите элементы запроса, которые будут выполнять роль констант при каждом запросе. Укажите имя службы поиска, ключ API администратора и ключ API запроса, скопированные на предыдущем шаге. Эта ячейка также позволяет настраивать клиенты, которые будут использоваться для выполнения конкретных операций: [SearchIndexClient](/python/api/azure-search-documents/azure.search.documents.indexes.searchindexclient) для создания индекса и [SearchClient](/python/api/azure-search-documents/azure.search.documents.searchclient) для запроса индекса.

   ```python
    service_name = "YOUR-SEARCH-SERIVCE-NAME"
    admin_key = "YOUR-SEARCH-SERVICE-ADMIN-API-KEY"
    
    index_name = "hotels-quickstart"
    
    # Create an SDK client
    endpoint = "https://{}.search.windows.net/".format(service_name)
    admin_client = SearchIndexClient(endpoint=endpoint,
                          index_name=index_name,
                          credential=AzureKeyCredential(admin_key))
    
    search_client = SearchClient(endpoint=endpoint,
                          index_name=index_name,
                          credential=AzureKeyCredential(admin_key))
   ```

1. В третьей ячейке выполните операцию delete_index, чтобы очистить службу от любых существующих индексов *hotels-quickstart*. Удаление индекса позволяет создать еще один индекс *hotels-quickstart* с тем же именем.

   ```python
    try:
        result = admin_client.delete_index(index_name)
        print ('Index', index_name, 'Deleted')
    except Exception as ex:
        print (ex)
   ```

1. Выполните каждый шаг.

## <a name="1---create-an-index"></a>1\. Создание индекса

Обязательные элементы индекса включают в себя имя, коллекцию полей и ключ. Коллекция полей определяет структуру логического *документа поиска*, используемого для загрузки данных и возврата результатов. 

Каждое поле имеет имя, тип и атрибуты, которые определяют использование поля (например, обладает ли оно возможностью полнотекстового поиска, можно ли его отфильтровать или извлечь из результатов поиска). В индексе одно из полей типа `Edm.String` должно быть назначено в качестве *ключа* для идентификации документа.

Этот индекс называется hotels-quickstart и содержит приведенные ниже определения полей. Это подмножество более крупного [индекса Hotels](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotels/Hotels_IndexDefinition.JSON), используемого в рамках других пошаговых руководств. В этом руководстве мы его сократили.

1. В следующую ячейку вставьте приведенный ниже пример для предоставления схемы.

    ```python
    # Specify the index schema
    name = index_name
    fields = [
            SimpleField(name="HotelId", type=SearchFieldDataType.String, key=True),
            SearchableField(name="HotelName", type=SearchFieldDataType.String, sortable=True),
            SearchableField(name="Description", type=SearchFieldDataType.String, analyzer_name="en.lucene"),
            SearchableField(name="Description_fr", type=SearchFieldDataType.String, analyzer_name="fr.lucene"),
            SearchableField(name="Category", type=SearchFieldDataType.String, facetable=True, filterable=True, sortable=True),
        
            SearchableField(name="Tags", collection=True, type=SearchFieldDataType.String, facetable=True, filterable=True),
    
            SimpleField(name="ParkingIncluded", type=SearchFieldDataType.Boolean, facetable=True, filterable=True, sortable=True),
            SimpleField(name="LastRenovationDate", type=SearchFieldDataType.DateTimeOffset, facetable=True, filterable=True, sortable=True),
            SimpleField(name="Rating", type=SearchFieldDataType.Double, facetable=True, filterable=True, sortable=True),
    
            ComplexField(name="Address", fields=[
                SearchableField(name="StreetAddress", type=SearchFieldDataType.String),
                SearchableField(name="City", type=SearchFieldDataType.String, facetable=True, filterable=True, sortable=True),
                SearchableField(name="StateProvince", type=SearchFieldDataType.String, facetable=True, filterable=True, sortable=True),
                SearchableField(name="PostalCode", type=SearchFieldDataType.String, facetable=True, filterable=True, sortable=True),
                SearchableField(name="Country", type=SearchFieldDataType.String, facetable=True, filterable=True, sortable=True),
            ])
        ]
    cors_options = CorsOptions(allowed_origins=["*"], max_age_in_seconds=60)
    scoring_profiles = []
    suggester = [{'name': 'sg', 'source_fields': ['Tags', 'Address/City', 'Address/Country']}]
    ```

1. В другой ячейке сформулируйте сам запрос. Этот запрос create_index отправляется в коллекцию индексов службы поиска и создает [SearchIndex](/python/api/azure-search-documents/azure.search.documents.indexes.models.searchindex) на основе схемы индекса, которую вы указали в предыдущей ячейке.

    ```python
    index = SearchIndex(
        name=name,
        fields=fields,
        scoring_profiles=scoring_profiles,
        suggesters = suggester,
        cors_options=cors_options)
    
    try:
        result = admin_client.create_index(index)
        print ('Index', result.name, 'created')
    except Exception as ex:
        print (ex)
    ```

1. Выполните каждый шаг.

<a name="load-documents"></a>

## <a name="2---load-documents"></a>2\. Загрузка документов

Чтобы загрузить документы, создайте коллекцию документов, используя значение [index action](/python/api/azure-search-documents/azure.search.documents.models.indexaction) для типа операции (передача, слияние и передача и т. п.). Источником документов является [HotelsData](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotels/HotelsData_toAzureSearch.JSON) на сайте GitHub.

1. В новой ячейке укажите четыре документа, соответствующие схеме индекса. Укажите действие отправки для каждого документа.

    ```python
    documents = [
        {
        "@search.action": "upload",
        "HotelId": "1",
        "HotelName": "Secret Point Motel",
        "Description": "The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Time's Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.",
        "Description_fr": "L'hôtel est idéalement situé sur la principale artère commerciale de la ville en plein cœur de New York. A quelques minutes se trouve la place du temps et le centre historique de la ville, ainsi que d'autres lieux d'intérêt qui font de New York l'une des villes les plus attractives et cosmopolites de l'Amérique.",
        "Category": "Boutique",
        "Tags": [ "pool", "air conditioning", "concierge" ],
        "ParkingIncluded": "false",
        "LastRenovationDate": "1970-01-18T00:00:00Z",
        "Rating": 3.60,
        "Address": {
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
        "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
        "Category": "Boutique",
        "Tags": [ "pool", "free wifi", "concierge" ],
        "ParkingIncluded": "false",
        "LastRenovationDate": "1979-02-18T00:00:00Z",
        "Rating": 3.60,
        "Address": {
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
        "Description": "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel's restaurant services.",
        "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
        "Category": "Resort and Spa",
        "Tags": [ "air conditioning", "bar", "continental breakfast" ],
        "ParkingIncluded": "true",
        "LastRenovationDate": "2015-09-20T00:00:00Z",
        "Rating": 4.80,
        "Address": {
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
        "Description_fr": "Le sublime Cliff Hotel est situé au coeur du centre historique de sublime dans un quartier extrêmement animé et vivant, à courte distance de marche des sites et monuments de la ville et est entouré par l'extraordinaire beauté des églises, des bâtiments, des commerces et Monuments. Sublime Cliff fait partie d'un Palace 1800 restauré avec amour.",
        "Category": "Boutique",
        "Tags": [ "concierge", "view", "24-hour front desk service" ],
        "ParkingIncluded": "true",
        "LastRenovationDate": "1960-02-06T00:00:00Z",
        "Rating": 4.60,
        "Address": {
            "StreetAddress": "7400 San Pedro Ave",
            "City": "San Antonio",
            "StateProvince": "TX",
            "PostalCode": "78216",
            "Country": "USA"
            }
        }
    ]
    ```  

1. В другой ячейке сформулируйте сам запрос. Этот запрос upload_documents обращается к коллекции документов в индексе hotels-quickstart и помещает документы, переданные на предыдущем шаге, в индекс Когнитивного поиска.

    ```python
    try:
        result = search_client.upload_documents(documents=documents)
        print("Upload of new document succeeded: {}".format(result[0].succeeded))
    except Exception as ex:
        print (ex.message)
    ```

1. Выполните каждый шаг, чтобы отправить документы в индекс в службе поиска.

## <a name="3---search-an-index"></a>3\. Поиск в индексе

На этом шаге показано, как запросить индекс с помощью метода **search** [класса search.client](/python/api/azure-search-documents/azure.search.documents.searchclient).

1. Следующий шаг выполняет пустой поиск (`search=*`), который возвращает неупорядоченный список (оценка поиска = 1.0) произвольных документов. Поскольку условия не заданы, в результаты включаются все документы. Этот запрос выводит только два поля в каждом документе. Он также добавляет `include_total_count=True`, чтобы получить количество всех документов (4) в результатах.

    ```python
    results =  search_client.search(search_text="*", include_total_count=True)
    
    print ('Total Documents Matching Query:', results.get_count())
    for result in results:
        print("{}: {}".format(result["HotelId"], result["HotelName"]))
    ```

1. Следующий запрос добавляет в выражение поиска целые термины ("wifi"). Этот запрос указывает, что результаты содержат только поля в операторе `select`. Ограничение числа полей, которые возвращаются обратно, минимизирует объем данных, передаваемых по сети, и сокращает задержку поиска.

    ```python
    results =  search_client.search(search_text="wifi", include_total_count=True, select='HotelId,HotelName,Tags')
    
    print ('Total Documents Matching Query:', results.get_count())
    for result in results:
        print("{}: {}: {}".format(result["HotelId"], result["HotelName"], result["Tags"]))
    ```

1. Затем примените критерий фильтра, возвращая только те отели, для которых рейтинг больше 4, отсортированные в убывающем порядке.

    ```python
    results =  search_client.search(search_text="hotels", select='HotelId,HotelName,Rating', filter='Rating gt 4', order_by='Rating desc')
    
    for result in results:
        print("{}: {} - {} rating".format(result["HotelId"], result["HotelName"], result["Rating"]))
    ```

1. Добавьте `search_fields`, чтобы ограничить запрос до одного поля.

    ```python
    results =  search_client.search(search_text="sublime", search_fields='HotelName', select='HotelId,HotelName')
    
    for result in results:
        print("{}: {}".format(result["HotelId"], result["HotelName"]))
    ```

1. Аспекты — это метки, которые можно использовать для создания структуры навигации по аспектам. Этот запрос возвращает аспекты и счетчики для категории.

    ```python
    results =  search_client.search(search_text="*", facets=["Category"])
    
    facets = results.get_facets()
    
    for facet in facets["Category"]:
        print("    {}".format(facet))
    ```

1. В этом примере выполняется поиск определенного документа на основе его ключа. Обычно требуется вернуть документ, когда пользователь щелкает документ в результатах поиска.

    ```python
    result = search_client.get_document(key="3")
    
    print("Details for hotel '3' are:")
    print("Name: {}".format(result["HotelName"]))
    print("Rating: {}".format(result["Rating"]))
    print("Category: {}".format(result["Category"]))
    ```

1. В этом примере мы будем использовать функцию автозаполнения. Она используется в поле поиска для автоматического завершения потенциальных совпадений по мере того, как пользователь вводит запрос в поле поиска.

   При создании индекса в составе запроса также создается средство подбора с именем "sg". Определение средства подбора указывает, какие поля можно использовать для поиска потенциальных соответствий для запросов предложения. В этом примере это поля "теги", "адрес/город", "адрес/страна". Чтобы имитировать автозавершение, введите "sa" в виде части строки. Метод автозаполнения [SearchClient](/python/api/azure-search-documents/azure.search.documents.searchclient) возвращает подходящие термины.

    ```python
    search_suggestion = 'sa'
    results = search_client.autocomplete(search_text=search_suggestion, suggester_name="sg", mode='twoTerms')
    
    print("Autocomplete for:", search_suggestion)
    for result in results:
        print (result['text'])
    ```

## <a name="clean-up"></a>Очистка

Если вы работаете в своей подписке, по окончании проекта рекомендуем решить, нужны ли вам созданные ресурсы. Работающие ресурсы могут означать лишние затраты. Можно удалить отдельные ресурсы или удалить группу ресурсов, что позволит удалить весь набор ресурсов.

Просматривать ресурсы и управлять ими можно на портале с помощью ссылок **Все ресурсы** или **Группы ресурсов** на панели навигации слева.

При работе с бесплатной версией службы помните о том, что вам доступно максимум три индекса, индексатора и источника данных. Вы можете удалить отдельные элементы на портале, чтобы не превысить лимит. 

## <a name="next-steps"></a>Дальнейшие действия

Для упрощения в этом кратком руководстве используется сокращенная версия индекса Hotels. Вы можете создать полную версию, чтобы выполнить более интересные запросы. Чтобы получить полную версию и все 50 документов, запустите мастер **импорта данных**, выбрав *hotels-sample* из встроенных примеров источников данных.

> [!div class="nextstepaction"]
> [Краткое руководство. Создание индекса службы "Поиск Azure" с помощью портала Azure](search-get-started-portal.md)