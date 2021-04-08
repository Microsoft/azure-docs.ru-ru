---
title: Краткое руководство. Создание индекса поиска в JavaScript
titleSuffix: Azure Cognitive Search
description: Из этого краткого руководства по JavaScript вы узнаете, как создать индекс, загрузить данные и выполнить запросы к службе "Когнитивный поиск Azure" с помощью JavaScript.
author: dereklegenzoff
manager: luisca
ms.author: delegenz
ms.devlang: javascript
ms.service: cognitive-search
ms.topic: quickstart
ms.date: 02/02/2021
ms.custom: devx-track-js
ms.openlocfilehash: 7d31201f9d1a4519538aba3ac57ddfd340d936d1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99509407"
---
# <a name="quickstart-create-an-azure-cognitive-search-index-using-the-javascript-sdk"></a>Краткое руководство. Создание индекса службы "Когнитивный поиск Azure" с помощью пакета SDK для JavaScript
> [!div class="op_single_selector"]
> * [JavaScript](search-get-started-javascript.md)
> * [C#](search-get-started-dotnet.md)
> * [Портал](search-get-started-portal.md)
> * [PowerShell](./search-get-started-powershell.md)
> * [Python](search-get-started-python.md)
> * [REST](search-get-started-rest.md)


Чтобы создать приложение Node.js на JavaScript, которое создает, загружает и запрашивает индекс поиска, используйте [пакет SDK для Javascript/Typscript для Когнитивного поиска Azure](/javascript/api/overview/azure/search-documents-readme).

Здесь вы найдете пошаговое руководство по созданию приложения. Вы также можете [скачать исходный код и данные](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/quickstart/v11) и запустить приложение из командной строки.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем начать работу, убедитесь, что у вас есть следующие средства и службы:

+ Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/) бесплатно.

+ Служба "Когнитивный поиск Azure". [Создайте новую](search-create-service-portal.md) или [найдите существующую службу](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices). Вы можете использовать бесплатную службу для выполнения инструкций, описанных в этом кратком руководстве. 

+ [Node.js](https://nodejs.org) и [NPM](https://www.npmjs.com)

+ [Visual Studio Code](https://code.visualstudio.com) или другая интегрированная среда разработки


## <a name="set-up-your-project"></a>Настройка проекта

Начните с получения конечной точки и ключа для службы поиска. Затем создайте проект с NPM, как описано ниже.

<a name="get-service-info"></a>

### <a name="copy-a-key-and-endpoint"></a>Копирование ключа и конечной точки

Вызовы к службе требуют конечную точку URL-адреса и ключ доступа при каждом запросе. В первую очередь найдите ключ API и URL-адрес для добавления в проект. При создании клиента на более позднем этапе необходимо указать оба значения.

1. [Войдите на портал Azure](https://portal.azure.com/) и на странице **обзора** службы поиска получите URL-адрес. Пример конечной точки может выглядеть так: `https://mydemo.search.windows.net`.

2. В разделе **Параметры** > **Ключи** получите ключ администратора с полным доступом к службе, требуемый для создания и удаления объектов. Существует две взаимозаменяемых пары первичного и вторичного ключей. Вы можете использовать любую из этих пар.

   ![Получение конечной точки HTTP и ключа доступа](media/search-get-started-rest/get-url-key.png "Получение конечной точки HTTP и ключа доступа")

Для выполнения любого запроса к службе требуется использование ключа API. Если есть действительный ключ, для каждого запроса устанавливаются отношения доверия между приложением, которое отправляет запрос, и службой, которая его обрабатывает.

### <a name="create-a-new-npm-project"></a>Создание проекта NPM

Откройте VS Code и его [встроенный терминал](https://code.visualstudio.com/docs/editor/integrated-terminal) или другой терминал, такой как командная строка Node.js.

1. Создайте каталог разработки и присвойте ему имя `quickstart`:

    ```cmd
    mkdir quickstart
    cd quickstart
    ```

2. Инициализируйте пустой проект с помощью NPM, выполнив: 

    ```cmd
    npm init
    ```
     Примите значения по умолчанию, за исключением лицензии, которая должна иметь значение MIT. 

3. Установите `@azure/search-documents` — [пакет SDK для Javascript/Typscript для Когнитивного поиска Azure](/javascript/api/overview/azure/search-documents-readme).

    ```cmd
    npm install @azure/search-documents
    ```

4. Установите `dotenv` для импорта переменных среды, таких как имя службы и ключ API.
    ```cmd
    npm install dotenv
    ```

5. Убедитесь, что вы настроили проекты и зависимости. Файл **package.json** должен выглядеть примерно так:

    ```json
    {
      "name": "quickstart",
      "version": "1.0.0",
      "description": "Azure Cognitive Search Quickstart",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "keywords": [
        "Azure",
        "Search"
      ],
      "author": "Your Name",
      "license": "MIT",
      "dependencies": {
        "@azure/search-documents": "^11.0.3",
        "dotenv": "^8.2.0"
      }
    }
    ```

6. Создайте файл **ENV** для хранения параметров службы поиска:

    ```
    SEARCH_API_KEY=<search-admin-key>
    SEARCH_API_ENDPOINT=https://<search-service-name>.search.windows.net
    ```

Замените значение `<search-service-name>` именем службы поиска. Замените `<search-admin-key>` значением ключа, которое вы записали ранее. 

### <a name="create-indexjs-file"></a>Создание файла index.js

Далее мы создадим файл **index.js** — основной файл, в котором будет размещен наш код.

В верхней части этого файла мы импортируем библиотеку `@azure/search-documents`:

```javascript
const { SearchIndexClient, SearchClient, AzureKeyCredential, odata } = require("@azure/search-documents");
```

Далее необходимо, чтобы пакет `dotenv` считал параметры из файла **ENV** следующим образом:

```javascript
// Load the .env file if it exists
require("dotenv").config();

// Getting endpoint and apiKey from .env file
const endpoint = process.env.SEARCH_API_ENDPOINT || "";
const apiKey = process.env.SEARCH_API_KEY || "";
```

После импорта данных и переменных среды мы готовы определить функцию main.

Большинство функций в пакете SDK являются асинхронными, поэтому мы делаем функцию main `async`. Мы также включаем `main().catch()` ниже функции main для перехвата и записи в журнал всех обнаруженных ошибок:

```javascript
async function main() {
    console.log(`Running Azure Cognitive Search Javascript quickstart...`);
    if (!endpoint || !apiKey) {
        console.log("Make sure to set valid values for endpoint and apiKey with proper authorization.");
        return;
    }

    // remaining quickstart code will go here
}

main().catch((err) => {
    console.error("The sample encountered an error:", err);
});
```

После этого можно создать индекс.

## <a name="1---create-index"></a>1\. Создание индекса 

Создайте файл **hotels_quickstart_index.json**.  Этот файл определяет, как Когнитивный поиск Azure будет работать с документами, которые вы отправите на следующем шаге. Каждое поле идентифицируется по свойству `name`, и для каждого поля указано значение `type`. У каждого поля также есть ряд атрибутов индекса, которые определяют, может ли Когнитивный поиск Azure применять это поле для поиска, фильтрации, сортировки и определения аспектов. Большинство полей имеют простой тип данных, но некоторые (например, `AddressType`) являются сложными типами, что позволяет создавать сложные структуры данных в индексе.  Вы можете подробнее изучить [поддерживаемые типы данных](/rest/api/searchservice/supported-data-types) и атрибуты индекса, описанные в статье [Создание индекса (REST API службы "Когнитивный поиск Azure")](/rest/api/searchservice/create-index). 

Добавьте следующий код в **hotels_quickstart_index.json** или [скачайте готовый файл](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/quickstart/v11/hotels_quickstart_index.json). 

```json
{
    "name": "hotels-quickstart",
    "fields": [
        {
            "name": "HotelId",
            "type": "Edm.String",
            "key": true,
            "filterable": true
        },
        {
            "name": "HotelName",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "sortable": true,
            "facetable": false
        },
        {
            "name": "Description",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "sortable": false,
            "facetable": false,
            "analyzer": "en.lucene"
        },
        {
            "name": "Description_fr",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "sortable": false,
            "facetable": false,
            "analyzer": "fr.lucene"
        },
        {
            "name": "Category",
            "type": "Edm.String",
            "searchable": true,
            "filterable": true,
            "sortable": true,
            "facetable": true
        },
        {
            "name": "Tags",
            "type": "Collection(Edm.String)",
            "searchable": true,
            "filterable": true,
            "sortable": false,
            "facetable": true
        },
        {
            "name": "ParkingIncluded",
            "type": "Edm.Boolean",
            "filterable": true,
            "sortable": true,
            "facetable": true
        },
        {
            "name": "LastRenovationDate",
            "type": "Edm.DateTimeOffset",
            "filterable": true,
            "sortable": true,
            "facetable": true
        },
        {
            "name": "Rating",
            "type": "Edm.Double",
            "filterable": true,
            "sortable": true,
            "facetable": true
        },
        {
            "name": "Address",
            "type": "Edm.ComplexType",
            "fields": [
                {
                    "name": "StreetAddress",
                    "type": "Edm.String",
                    "filterable": false,
                    "sortable": false,
                    "facetable": false,
                    "searchable": true
                },
                {
                    "name": "City",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                },
                {
                    "name": "StateProvince",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                },
                {
                    "name": "PostalCode",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                },
                {
                    "name": "Country",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                }
            ]
        }
    ],
    "suggesters": [
        {
            "name": "sg",
            "searchMode": "analyzingInfixMatching",
            "sourceFields": [
                "HotelName"
            ]
        }
    ]
}
```

После определения индекса мы импортируем **hotels_quickstart_index.json** в верхней части файла **index.js**, чтобы функция main могла получить доступ к определению индекса.

```javascript
const indexDefinition = require('./hotels_quickstart_index.json');
```

В рамках функции main мы создадим класс `SearchIndexClient`, который используется для создания индексов и управления ими для службы "Когнитивный поиск Azure". 

```javascript
const indexClient = new SearchIndexClient(endpoint, new AzureKeyCredential(apiKey));
```

Затем нужно удалить индекс, если он уже существует. Это распространенная практика для тестового и демонстрационного кода.

Для этого нужно определить простую функцию, которая будет удалять индекс.

```javascript
async function deleteIndexIfExists(indexClient, indexName) {
    try {
        await indexClient.deleteIndex(indexName);
        console.log('Deleting index...');
    } catch {
        console.log('Index does not exist yet.');
    }
}
```

Чтобы запустить функцию, извлеките имя индекса из определения индекса и передайте `indexName` вместе с `indexClient` в функцию `deleteIndexIfExists()`.

```javascript
const indexName = indexDefinition["name"];

console.log('Checking if index exists...');
await deleteIndexIfExists(indexClient, indexName);
```

После этого можно создать индекс с помощью метода `createIndex()`.

```javascript
console.log('Creating index...');
let index = await indexClient.createIndex(indexDefinition);

console.log(`Index named ${index.name} has been created.`);
```

### <a name="run-the-sample"></a>Запуск примера

Теперь все готово к запуску примера. Выполните следующие команды в окне терминала.

```cmd
node index.js
```

Если вы [скачали исходный код](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/quickstart/v11) и еще не установили необходимые пакеты, сначала запустите `npm install`.

Вы увидите ряд сообщений с описанием действий, которые выполняются программой. 

Откройте **обзорные сведения** о службе Поиска Azure на портале Azure. Выберите вкладку **Индексы**. Вы должны увидеть нечто вроде этого:

:::image type="content" source="media/search-get-started-javascript/create-index-no-data.png" alt-text="Снимок экрана портала Azure, где открыта обзорная информация о Службе поиска на вкладке &quot;Индексы&quot;" border="false":::

На следующем шаге вы добавите данные в индекс. 

## <a name="2---load-documents"></a>2\. Загрузка документов 


Документы в службе "Когнитивный поиск Azure" представляют собой структуры данных, которые служат входами для индексирования и (или) выходами для запросов. Эти данные можно отправить в индекс или использовать [индексатор](search-indexer-overview.md). В этом случае мы будем программно отправлять документы в индекс.

Входные документы могут содержать строки базы данных, большие двоичные объекты из хранилища BLOB-объектов (как в этом примере) или сохраненные на диске документы JSON. Вы можете скачать файл [hotels.json](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/quickstart/v11/hotels.json) или создать собственный **hotels.json** со следующим содержимым:

```json
{
    "value": [
        {
            "HotelId": "1",
            "HotelName": "Secret Point Motel",
            "Description": "The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Time's Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.",
            "Description_fr": "L'hôtel est idéalement situé sur la principale artère commerciale de la ville en plein cœur de New York. A quelques minutes se trouve la place du temps et le centre historique de la ville, ainsi que d'autres lieux d'intérêt qui font de New York l'une des villes les plus attractives et cosmopolites de l'Amérique.",
            "Category": "Boutique",
            "Tags": ["pool", "air conditioning", "concierge"],
            "ParkingIncluded": false,
            "LastRenovationDate": "1970-01-18T00:00:00Z",
            "Rating": 3.6,
            "Address": {
                "StreetAddress": "677 5th Ave",
                "City": "New York",
                "StateProvince": "NY",
                "PostalCode": "10022"
            }
        },
        {
            "HotelId": "2",
            "HotelName": "Twin Dome Motel",
            "Description": "The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.",
            "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
            "Category": "Boutique",
            "Tags": ["pool", "free wifi", "concierge"],
            "ParkingIncluded": "false",
            "LastRenovationDate": "1979-02-18T00:00:00Z",
            "Rating": 3.6,
            "Address": {
                "StreetAddress": "140 University Town Center Dr",
                "City": "Sarasota",
                "StateProvince": "FL",
                "PostalCode": "34243"
            }
        },
        {
            "HotelId": "3",
            "HotelName": "Triple Landscape Hotel",
            "Description": "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel’s restaurant services.",
            "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
            "Category": "Resort and Spa",
            "Tags": ["air conditioning", "bar", "continental breakfast"],
            "ParkingIncluded": "true",
            "LastRenovationDate": "2015-09-20T00:00:00Z",
            "Rating": 4.8,
            "Address": {
                "StreetAddress": "3393 Peachtree Rd",
                "City": "Atlanta",
                "StateProvince": "GA",
                "PostalCode": "30326"
            }
        },
        {
            "HotelId": "4",
            "HotelName": "Sublime Cliff Hotel",
            "Description": "Sublime Cliff Hotel is located in the heart of the historic center of Sublime in an extremely vibrant and lively area within short walking distance to the sites and landmarks of the city and is surrounded by the extraordinary beauty of churches, buildings, shops and monuments. Sublime Cliff is part of a lovingly restored 1800 palace.",
            "Description_fr": "Le sublime Cliff Hotel est situé au coeur du centre historique de sublime dans un quartier extrêmement animé et vivant, à courte distance de marche des sites et monuments de la ville et est entouré par l'extraordinaire beauté des églises, des bâtiments, des commerces et Monuments. Sublime Cliff fait partie d'un Palace 1800 restauré avec amour.",
            "Category": "Boutique",
            "Tags": ["concierge", "view", "24-hour front desk service"],
            "ParkingIncluded": true,
            "LastRenovationDate": "1960-02-06T00:00:00Z",
            "Rating": 4.6,
            "Address": {
                "StreetAddress": "7400 San Pedro Ave",
                "City": "San Antonio",
                "StateProvince": "TX",
                "PostalCode": "78216"
            }
        }
    ]
}
```

Как и в случае с indexDefinition, нам также нужно импортировать `hotels.json` в верхней части **index.js**, чтобы к данным можно было получить доступ в нашей функции main.

```javascript
const hotelData = require('./hotels.json');
```


Чтобы индексировать данные в индексе поиска, нужно создать `SearchClient`. Хотя `SearchIndexClient` используется для создания индекса и управления им, для отправки документов и запроса индекса используется `SearchClient`.

Есть два способа создания представления данных `SearchClient`. Первый вариант — создать `SearchClient` с нуля:

```javascript
 const searchClient = new SearchClient(endpoint, indexName, new AzureKeyCredential(apiKey));
```

Кроме того, можно использовать метод `getSearchClient()` из `SearchIndexClient` для создания `SearchClient`:

```javascript
const searchClient = indexClient.getSearchClient(indexName);
```

Теперь, когда клиент определен, отправьте документы в индекс поиска. В этом случае мы используем метод `mergeOrUploadDocuments()`, который будет отправлять документы или объединять их с имеющимся документом, если документ с таким ключом уже есть.

```javascript
console.log('Uploading documents...');
let indexDocumentsResult = await searchClient.mergeOrUploadDocuments(hotelData['value']);

console.log(`Index operations succeeded: ${JSON.stringify(indexDocumentsResult.results[0].succeeded)}`);
```

Снова запустите программу командой `node index.js`. Теперь набор сообщений будет немного отличаться от тех, которые вы видели на шаге 1. На этот раз индекс уже *существует*, а вы получите сообщение о его удалении перед тем, как приложение создаст новый индекс и поместит в него данные. 

Прежде чем выполнять запросы на следующем шаге, определите функцию, чтобы программа ожидала в течение одной секунды. Это делается только для целей тестирования и демонстрации, чтобы обеспечить завершение индексирования и доступность документов в индексе для запросов.

```javascript
function sleep(ms) {
    var d = new Date();
    var d2 = null;
    do {
        d2 = new Date();
    } while (d2 - d < ms);
}
```

Чтобы программа ожидала в течение одной секунды, вызовите функцию `sleep`, как показано ниже:

```javascript
sleep(1000);
```

## <a name="3---search-an-index"></a>3\. Поиск в индексе

С помощью созданного индекса и отправленных документов можно отправлять запросы в индекс. В рамках этого раздела мы отправим пять различных запросов в индекс поиска, чтобы продемонстрировать различные доступные части функций запросов.

Запросы записываются в функцию `sendQueries()`, которую мы будем вызывать в функции main следующим образом:

```javascript
await sendQueries(searchClient);
```

Запросы отправляются с помощью метода `search()` объекта `searchClient`. Первым параметром является искомый текст, а вторым — дополнительные параметры поиска.

Первый запрос выполняет поиск `*`, что эквивалентно поиску по всем данным, и выбирает три поля в индексе. Рекомендуется использовать `select` только для нужных вам полей, так как извлечение ненужных данных приведет к задержкам в запросах.

`searchOptions` в этом запросе также имеет значение `true` для параметра `includeTotalCount`, возвращающего количество найденных результатов поиска.

```javascript
async function sendQueries(searchClient) {
    console.log('Query #1 - search everything:');
    let searchOptions = {
        includeTotalCount: true,
        select: ["HotelId", "HotelName", "Rating"]
    };

    let searchResults = await searchClient.search("*", searchOptions);
    for await (const result of searchResults.results) {
        console.log(`${JSON.stringify(result.document)}`);
    }
    console.log(`Result count: ${searchResults.count}`);

    // remaining queries go here
}
```

Оставшиеся запросы, описанные ниже, также нужно добавить в функцию `sendQueries()`. Они разделены для удобства чтения.

В следующем запросе укажите условие поиска `"wifi"`, а также включите фильтр, который возвращает результаты только в том случае, если состояние равно `'FL'`. Результаты также упорядочиваются по данным отелей `Rating`.

```javascript
console.log('Query #2 - Search with filter, orderBy, and select:');
let state = 'FL';
searchOptions = {
    filter: odata`Address/StateProvince eq ${state}`,
    orderBy: ["Rating desc"],
    select: ["HotelId", "HotelName", "Rating"]
};

searchResults = await searchClient.search("wifi", searchOptions);
for await (const result of searchResults.results) {
    console.log(`${JSON.stringify(result.document)}`);
}
```

Далее поиск ограничивается одним полем, поддерживающим поиск, с помощью параметра `searchFields`. Это отличный вариант, чтобы сделать запрос более эффективным, если вам требуются определенные поля. 

```javascript
console.log('Query #3 - Limit searchFields:');
searchOptions = {
    select: ["HotelId", "HotelName", "Rating"],
    searchFields: ["HotelName"]
};

searchResults = await searchClient.search("sublime cliff", searchOptions);
for await (const result of searchResults.results) {
    console.log(`${JSON.stringify(result.document)}`);
}
console.log();
```

Еще один распространенный параметр, включаемый в запрос, — `facets`. С помощью аспектов можно создавать фильтры в пользовательском интерфейсе, чтобы пользователям было проще узнать, какие значения они могут фильтровать.

```javascript
console.log('Query #4 - Use facets:');
searchOptions = {
    facets: ["Category"],
    select: ["HotelId", "HotelName", "Rating"],
    searchFields: ["HotelName"]
};

searchResults = await searchClient.search("*", searchOptions);
for await (const result of searchResults.results) {
    console.log(`${JSON.stringify(result.document)}`);
}
```

В последнем запросе используется метод `getDocument()` объекта `searchClient`. Это позволяет эффективно получать документ по ключу. 

```javascript
console.log('Query #5 - Lookup document:');
let documentResult = await searchClient.getDocument(key='3')
console.log(`HotelId: ${documentResult.HotelId}; HotelName: ${documentResult.HotelName}`)
```

### <a name="run-the-sample"></a>Запуск примера

Запустите программу командой `node index.js`. Теперь, помимо описанных выше шагов, будут отправлены запросы и выведены в консоль их результаты.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы работаете в своей подписке, по окончании проекта рекомендуем решить, нужны ли вам созданные ресурсы. Работающие ресурсы могут означать лишние затраты. Можно удалить отдельные ресурсы или удалить группу ресурсов, что позволит удалить весь набор ресурсов.

Просматривать ресурсы и управлять ими можно на портале с помощью ссылок **Все ресурсы** или **Группы ресурсов** на панели навигации слева.

При работе с бесплатной версией службы помните о том, что вам доступно максимум три индекса, индексатора и источника данных. Вы можете удалить отдельные элементы на портале, чтобы не превысить лимит. 

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого краткого руководства для JavaScript вы отработали ряд задач по созданию индекса, загрузке документов в него и выполнению запросов. 

Если у вас уже есть опыт работы со службой "Когнитивный поиск Azure", вы можете использовать этот пример как фундамент для средств подбора (запросы опережающего ввода или автозаполнения), фильтров и фасетной навигации. Если вы еще плохо знакомы со службой "Когнитивный поиск Azure", мы рекомендуем ознакомиться с другими учебниками, чтобы получить представление о его возможностях. Посетите нашу [страницу документации](https://azure.microsoft.com/documentation/services/search/), чтобы найти дополнительные ресурсы. 

> [!div class="nextstepaction"]
> [Создание внешнего интерфейса для службы"Когнитивный поиск Azure"](https://github.com/dereklegenzoff/azure-search-react-template)