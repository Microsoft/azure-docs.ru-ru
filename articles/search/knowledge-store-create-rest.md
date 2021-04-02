---
title: Создание хранилища знаний с помощью REST
titleSuffix: Azure Cognitive Search
description: Применение REST API и Postman для создания хранилища знаний службы "Когнитивный поиск Azure" для сохранения дополнений из конвейера обогащения данных с помощью ИИ.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 11/18/2020
ms.openlocfilehash: 6af9b8f97d622ae10cfdbcaa8ca50abb42ec7332
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94889060"
---
# <a name="create-a-knowledge-store-using-rest-and-postman"></a>Создание хранилища знаний с помощью REST и Postman

Хранилище знаний содержит выходные данные из конвейера обогащения службы "Когнитивный поиск Azure" для последующего анализа или обработки. Конвейер обогащения с помощью ИИ способен получать файлы изображений или неструктурированные текстовые файлы, отправлять их для индексирования в службу "Когнитивный поиск Azure", применять результаты дополнения с помощью ИИ, выполняемого в Cognitive Services (например, анализ изображений и обработка естественного языка), а затем сохранять результаты в хранилище знаний в службе хранилища Azure. Вы можете использовать на портале Azure такие средства, как Power BI или Обозреватель службы хранилища, чтобы исследовать хранилище знаний.

В этой статье объясняется, как использовать интерфейс REST API для приема, индексирования и применения дополнений с помощью ИИ к набору отзывов об отеле. Отзывы об отелях импортируются в хранилище BLOB-объектов Azure. Результаты сохраняются как хранилище знаний в хранилище таблиц Azure.

После создания хранилища знаний вы можете узнать, как получить к нему доступ с помощью [Обозревателя службы хранилища](knowledge-store-view-storage-explorer.md) или [Power BI](knowledge-store-connect-power-bi.md).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

> [!TIP]
> При работе с этой статьей мы рекомендуем использовать [классическое приложение Postman](https://www.getpostman.com/). [Исходный код](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/knowledge-store) для этой статьи содержит коллекцию Postman со всеми запросами. 

## <a name="create-services-and-load-data"></a>Создание служб и загрузка данных

В этом кратком руководстве используется Когнитивный поиск Azure, хранилище BLOB-объектов Azure и [Azure Cognitive Services](https://azure.microsoft.com/services/cognitive-services/) для искусственного интеллекта. 

Так как рабочая нагрузка мала, Cognitive Services работает в фоновом режиме, чтобы обеспечить бесплатную обработку до 20 транзакций в день. Так как набор данных небольшой, вы можете пропустить создание или присоединение ресурса Cognitive Services.

1. [Скачайте HotelReviews_Free.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Free.csv?sp=r&st=2019-11-04T01:23:53Z&se=2025-11-04T16:00:00Z&spr=https&sv=2019-02-02&sr=b&sig=siQgWOnI%2FDamhwOgxmj11qwBqqtKMaztQKFNqWx00AY%3D). Это данные отзывов об отелях, сохраненные в CSV-файле (с сайта Kaggle.com) и содержащие 19 элементов отзывов клиентов об одном отеле. 

1. [Создайте учетную запись хранения Azure](../storage/common/storage-account-create.md?tabs=azure-portal) или [найдите имеющуюся учетную запись](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Storage%2storageAccounts/) в текущей подписке. Служба хранилища Azure будет использоваться как для импорта необработанного содержимого, так и для хранилища знаний, которое является конечным результатом.

   Выберите тип учетной записи **StorageV2 (общего назначения версии 2)** .

1. Откройте страницы служб BLOB-объектов и создайте контейнер *hotel-reviews*.

1. Щелкните **Отправить**.

    ![Передача данных](media/knowledge-store-create-portal/upload-command-bar.png "Отправка отзывов о гостинице")

1. Выберите файл **HotelReviews-Free.csv**, скачанный на первом шаге.

    ![Создание контейнера BLOB-объектов Azure](media/knowledge-store-create-portal/hotel-reviews-blob-container.png "Создание контейнера BLOB-объектов Azure")

1. Вы почти закончили работу с этим ресурсом, но до выхода из этих страниц используйте ссылку в левой области навигации, чтобы открыть страницу **Ключи доступа**. Получите строку подключения для извлечения данных из хранилища BLOB-объектов. Строка подключения выглядит следующим образом: `DefaultEndpointsProtocol=https;AccountName=<YOUR-ACCOUNT-NAME>;AccountKey=<YOUR-ACCOUNT-KEY>;EndpointSuffix=core.windows.net`

1. Оставаясь на портале, перейдите к службе "Когнитивный поиск Azure". [Создайте](search-create-service-portal.md) или [найдите существующую службу](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices). Вы можете использовать для этого упражнения бесплатную службу.

## <a name="configure-postman"></a>Настройка Postman

Установите и настройте Postman.

### <a name="download-and-install-postman"></a>Скачивание и установка Postman

1. Скачайте [исходный код коллекции Postman](https://github.com/Azure-Samples/azure-search-postman-samples/blob/master/knowledge-store/KnowledgeStore.postman_collection.json).
1. Щелкните **File** > **Import** (Файл > Импорт), чтобы импортировать этот исходный код в Postman.
1. Выберите вкладку **Collections** (Коллекции) и нажмите кнопку **...** (многоточие).
1. Выберите команду **Изменить**. 
   
   ![Приложение Postman, демонстрирующее навигацию](media/knowledge-store-create-rest/postman-edit-menu.png "Переход в меню Правка в Postman")
1. В диалоговом окне **Edit** (Правка) выберите вкладку **Variables** (Переменные). 

На вкладке **Переменные** вы можете добавить значения, которые Postman будет заменять во всех местах, где они указаны в двойных скобках. Например, Postman заменит символ `{{admin-key}}` текущим значением, которое вы указали для `admin-key`. Postman выполняет такие замены в URL-адресах, заголовках, текстах запросов и так далее. 

Чтобы получить значение `admin-key`, перейдите в службу "Когнитивный поиск Azure" и откройте вкладку **Ключи**. Замените `search-service-name` и `storage-account-name` теми значениями, которые вы выбрали на шаге [Создание служб](#create-services-and-load-data). Задайте параметру `storage-connection-string` значение, полученное на вкладке **Ключи доступа** для учетной записи хранения. Для остальных параметров можно сохранить значения по умолчанию.

![Вкладка переменных приложения Postman](media/knowledge-store-create-rest/postman-variables-window.png "Окно переменных Postman")


| Переменная    | Как получить |
|-------------|-----------------|
| `admin-key` | На странице **Ключи** службы "Когнитивный поиск Azure".  |
| `api-version` | Сохраните значение **2020-06-30**. |
| `datasource-name` | Сохраните значение **hotel-reviews-ds**. | 
| `indexer-name` | Сохраните значение **hotel-reviews-ixr**. | 
| `index-name` | Сохраните значение **hotel-reviews-ix**. | 
| `search-service-name` | Имя службы "Когнитивный поиск Azure". Введите URL-адрес `https://{{search-service-name}}.search.windows.net`. | 
| `skillset-name` | Сохраните значение **hotel-reviews-ss**. | 
| `storage-account-name` | Имя учетной записи хранения Azure. | 
| `storage-connection-string` | На вкладке **Ключи доступа** для учетной записи хранения выберите **key1** > **Строка подключения**. | 
| `storage-container-name` | Сохраните значение **hotel-reviews**. | 

### <a name="review-the-request-collection-in-postman"></a>Обзор коллекции запросов в Postman

При создании хранилища знаний следует отправить четыре HTTP-запроса: 

- **Запрос PUT для создания индекса.** Этот индекс содержит данные, которые служба "Когнитивный поиск Azure" использует и возвращает.
- **Запрос POST для создания источника данных.** Этот источник данных связывает поведение службы "Когнитивный поиск Azure" с учетной записью хранения данных и хранилища знаний. 
- **Запрос PUT для создания набора навыков.** Набор навыков определяет дополнения, которые применяются к данным и структуре хранилища знаний.
- **Запрос PUT для создания индексатора.** При выполнении индексатора считываются данные, применяется набор навыков и сохраняются результаты. Этот запрос необходимо выполнить последним.

[Исходный код](https://github.com/Azure-Samples/azure-search-postman-samples/blob/master/knowledge-store/KnowledgeStore.postman_collection.json) содержит коллекцию Postman с этими четырьмя запросами. Чтобы отправить запросы, выберите в Postman вкладку запроса. Добавьте здесь заголовки запроса `api-key` и `Content-Type`. Присвойте параметру `api-key` значение `{{admin-key}}`. Присвойте параметру `Content-type` значение `application/json`. 

![Снимок экрана с интерфейсом Postman для заголовков](media/knowledge-store-create-rest/postman-headers-ui.png)

> [!Note]
> Во всех запросах следует задать заголовки `api-key` и `Content-type`. Если Postman распознает переменную, она отображается оранжевым цветом, как `{{admin-key}}` на предыдущем снимке экрана. Если имя переменной написано с ошибкой, она отображается красным цветом.
>

## <a name="create-an-azure-cognitive-search-index"></a>Создание индекса службы "Когнитивный поиск Azure"

Создайте индекс службы "Когнитивный поиск Azure", чтобы представить данные, которые вас интересуют для поиска, фильтрации и улучшений. Для создания индекса отправьте запрос PUT на адрес `https://{{search-service-name}}.search.windows.net/indexes/{{index-name}}?api-version={{api-version}}`. Postman заменит символы, заключенные в двойные скобки (например `{{search-service-name}}`, `{{index-name}}` и `{{api-version}}`), значениями, указанными на шаге [Настройка Postman](#configure-postman). Если вы используете другой инструмент для выполнения команд REST, вам придется заменить переменные самостоятельно.

Укажите структуру индекса службы "Когнитивный поиск Azure" в тексте запроса. Настроив заголовки `api-key` и `Content-type` в Postman, перейдите на панель **Body** (Текст) для запроса. Вы увидите приведенный ниже код JSON. Если его здесь нет, выберите **Raw** (Необработанный) > **JSON (application/json)** и вставьте в текст запроса следующий код:

```JSON
{
    "name": "{{index-name}}",
    "fields": [
        { "name": "address", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "categories", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "city", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false },
        { "name": "country", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "latitude", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "longitude", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "name", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false },
        { "name": "postalCode", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "province", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "reviews_date", "type": "Edm.DateTimeOffset", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "reviews_dateAdded", "type": "Edm.DateTimeOffset", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "reviews_rating", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "reviews_text", "type": "Edm.String", "filterable": false,  "sortable": false, "facetable": false },
        { "name": "reviews_title", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "reviews_username", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "AzureSearch_DocumentKey", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false, "key": true },
        { "name": "metadata_storage_content_type", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "metadata_storage_size", "type": "Edm.Int64", "searchable": false, "filterable": false, "sortable": false, "facetable": false},
        { "name": "metadata_storage_last_modified", "type": "Edm.DateTimeOffset", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "metadata_storage_name", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "metadata_storage_path", "type": "Edm.String", "searchable": false, "filterable": false, "sortable": false, "facetable": false },
        { "name": "Sentiment", "type": "Collection(Edm.Double)", "searchable": false, "filterable": true, "retrievable": true, "sortable": false, "facetable": true },
        { "name": "Language", "type": "Edm.String", "filterable": true, "sortable": false, "facetable": true },
        { "name": "Keyphrases", "type": "Collection(Edm.String)", "filterable": true, "sortable": false, "facetable": true }
    ]
}

```

Это определение индекса представляет собой комбинацию данных, которые необходимо предоставить пользователю (название отеля, содержание отзыва, дата), метаданных поиска и данных для дополнения с помощью ИИ (тональность, ключевые фразы и язык).

Щелкните **Send** (Отправить), чтобы отправить запрос PUT. Отобразится состояние `201 - Created`. Если вы увидите другое сообщение о состоянии, на панели **Body** (Текст) найдите ответ JSON с сообщением об ошибке. 

## <a name="create-the-datasource"></a>Создание источника данных

Теперь подключите службу "Когнитивный поиск Azure" к данным об отелях, которые вы сохранили в хранилище BLOB-объектов. Чтобы создать источник данных, отправьте запрос POST на адрес `https://{{search-service-name}}.search.windows.net/datasources?api-version={{api-version}}`. Установите также заголовки `api-key` и `Content-Type`, как упоминалось ранее. 

В интерфейсе Postman перейдите к запросу **Create Datasource** (Создание источника данных) и откройте панель **Body** (Текст). Вы увидите приведенный ниже код.

```json
{
  "name" : "{{datasource-name}}",
  "description" : "Demo files to demonstrate knowledge store capabilities.",
  "type" : "azureblob",
  "credentials" : { "connectionString" : "{{storage-connection-string}}" },
  "container" : { "name" : "{{storage-container-name}}" }
}
```

Щелкните **Send** (Отправить), чтобы отправить запрос POST. 

## <a name="create-the-skillset"></a>Создание набора навыков 

Дальше нужно задать набор навыков, который указывает как применяемые улучшения, так и хранилище знаний, в котором будут храниться результаты. В Postman откройте вкладку **Create the Skillset** (Создание набора навыков). Этот запрос отправляет PUT на `https://{{search-service-name}}.search.windows.net/skillsets/{{skillset-name}}?api-version={{api-version}}`. Задайте заголовки `api-key` и `Content-type`, как вы уже делали ранее. 

Существует два больших объекта верхнего уровня: `skills` и `knowledgeStore`. Каждый объект внутри объекта `skills` является службой дополнения. Каждая служба добавления имеет `inputs` и `outputs`. Для `LanguageDetectionSkill` определено выходное значение `targetName` с типом `Language`. Значение этого узла используется большинством других навыков в качестве входных данных. Источником является `document/Language`. Возможность использовать выходные данные одного узла в качестве входных данных для другого еще более важна для `ShaperSkill`, который определяет методы передачи данных в таблицы хранилища знаний.

Объект `knowledge_store` подключается к учетной записи хранения через переменную `{{storage-connection-string}}` Postman. `knowledge_store` содержит набор сопоставлений между расширенным документом с одной стороны, таблицами и столбцами в хранилище знаний с другой. 

Чтобы создать набор навыков, нажмите кнопку **Send** (Отправить) в Postman для отправки следующего запроса PUT.

```json
{
    "name": "{{skillset-name}}",
    "description": "Skillset to detect language, extract key phrases, and detect sentiment",
    "skills": [ 
        {
            "@odata.type": "#Microsoft.Skills.Text.SplitSkill", 
            "context": "/document/reviews_text", "textSplitMode": "pages", "maximumPageLength": 5000,
            "inputs": [ 
                { "name": "text", "source": "/document/reviews_text" },
                { "name": "languageCode", "source": "/document/Language" }
            ],
            "outputs": [
                { "name": "textItems", "targetName": "pages" }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
            "context": "/document/reviews_text/pages/*",
            "inputs": [
                { "name": "text", "source": "/document/reviews_text/pages/*" },
                { "name": "languageCode", "source": "/document/Language" }
            ],
            "outputs": [
                { "name": "score", "targetName": "Sentiment" }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",
            "context": "/document",
            "inputs": [
                { "name": "text", "source": "/document/reviews_text" }
            ],
            "outputs": [
                { "name": "languageCode", "targetName": "Language" }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
            "context": "/document/reviews_text/pages/*",
            "inputs": [
                { "name": "text",  "source": "/document/reviews_text/pages/*" },
                { "name": "languageCode",  "source": "/document/Language" }
            ],
            "outputs": [
                { "name": "keyPhrases" , "targetName": "Keyphrases" }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
            "context": "/document",
            "inputs": [
                { "name": "name",  "source": "/document/name" },
                { "name": "reviews_date",  "source": "/document/reviews_date" },
                { "name": "reviews_rating",  "source": "/document/reviews_rating" },
                { "name": "reviews_text",  "source": "/document/reviews_text" },
                { "name": "reviews_title",  "source": "/document/reviews_title" },
                { "name": "AzureSearch_DocumentKey",  "source": "/document/AzureSearch_DocumentKey" },
                { 
                    "name": "pages",
                    "sourceContext": "/document/reviews_text/pages/*",
                    "inputs": [
                        { "name": "SentimentScore", "source": "/document/reviews_text/pages/*/Sentiment" },
                        { "name": "LanguageCode", "source": "/document/Language" },
                        { "name": "Page", "source": "/document/reviews_text/pages/*" },
                        { 
                            "name": "keyphrase", "sourceContext": "/document/reviews_text/pages/*/Keyphrases/*",
                            "inputs": [
                                { "name": "Keyphrases", "source": "/document/reviews_text/pages/*/Keyphrases/*" }
                            ]
                        }
                    ]
                }
            ],
            "outputs": [
                { "name": "output" , "targetName": "tableprojection" }
            ]
        }
    ],
    "knowledgeStore": {
        "storageConnectionString": "{{storage-connection-string}}",
        "projections": [
            {
                "tables": [
                    { "tableName": "hotelReviewsDocument", "generatedKeyName": "Documentid", "source": "/document/tableprojection" },
                    { "tableName": "hotelReviewsPages", "generatedKeyName": "Pagesid", "source": "/document/tableprojection/pages/*" },
                    { "tableName": "hotelReviewsKeyPhrases", "generatedKeyName": "KeyPhrasesid", "source": "/document/tableprojection/pages/*/keyphrase/*" },
                    { "tableName": "hotelReviewsSentiment", "generatedKeyName": "Sentimentid", "source": "/document/tableprojection/pages/*/sentiment/*" }
                ],
                "objects": []
            },
            {
                "tables": [
                    { 
                        "tableName": "hotelReviewsInlineDocument", "generatedKeyName": "Documentid", "sourceContext": "/document",
                        "inputs": [
                            { "name": "name", "source": "/document/name"},
                            { "name": "reviews_date", "source": "/document/reviews_date"},
                            { "name": "reviews_rating", "source": "/document/reviews_rating"},
                            { "name": "reviews_text", "source": "/document/reviews_text"},
                            { "name": "reviews_title", "source": "/document/reviews_title"},
                            { "name": "AzureSearch_DocumentKey", "source": "/document/AzureSearch_DocumentKey" }
                        ]
                    },
                    { 
                        "tableName": "hotelReviewsInlinePages", "generatedKeyName": "Pagesid", "sourceContext": "/document/reviews_text/pages/*",
                        "inputs": [
                            { "name": "SentimentScore", "source": "/document/reviews_text/pages/*/Sentiment"},
                            { "name": "LanguageCode", "source": "/document/Language"},
                            { "name": "Page", "source": "/document/reviews_text/pages/*" }
                        ]
                    },
                    { 
                        "tableName": "hotelReviewsInlineKeyPhrases", "generatedKeyName": "kpidv2", "sourceContext": "/document/reviews_text/pages/*/Keyphrases/*",
                        "inputs": [
                            { "name": "Keyphrases", "source": "/document/reviews_text/pages/*/Keyphrases/*" }
                        ]
                    }
                ],
                "objects": []
            }
        ]
    }
}
```

## <a name="create-the-indexer"></a>Создание индексатора

Последним шагом является создание индексатора. Индексатор считывает данные и активирует набор навыков. В окне Postman выберите запрос **Create Indexer** (Создание индексатора) и проверьте его текст. Определение этого индексатора содержит ссылки на другие ресурсы, которые вы уже создали — источник данных, индекс и набор навыков. 

Объект `parameters/configuration` управляет процессом приема данных индексатором. В этом примере входные данные находятся в одном документе со строкой заголовка и значениями, разделенными запятыми. Ключ документа выполняет роль уникального идентификатора этого документа. По сути, ключ документа содержит закодированный URL-адрес исходного документа. Наконец, выходные значения набора навыков, такие как код языка, тональность и ключевые фразы, сопоставляются с нужными местами в документе. `Language` имеет только одно значение, но `Sentiment` применяется к каждому элементу массива `pages`. `Keyphrases` является массивом и также применяется к каждому элементу массива `pages`.

Указав заголовки `api-key` и `Content-type` и убедившись, что текст запроса похож на приведенный ниже исходный код, нажмите кнопку **Send** (Отправить) в Postman. Postman отправит запрос PUT на адрес `https://{{search-service-name}}.search.windows.net/indexers/{{indexer-name}}?api-version={{api-version}}`. Служба "Когнитивный поиск Azure" создаст и запустит индексатор. 

```json
{
    "name": "{{indexer-name}}",
    "dataSourceName": "{{datasource-name}}",
    "skillsetName": "{{skillset-name}}",
    "targetIndexName": "{{index-name}}",
    "parameters": {
        "configuration": {
            "dataToExtract": "contentAndMetadata",
            "parsingMode": "delimitedText",
            "firstLineContainsHeaders": true,
            "delimitedTextDelimiter": ","
        }
    },
    "fieldMappings": [
        {
            "sourceFieldName": "AzureSearch_DocumentKey",
            "targetFieldName": "AzureSearch_DocumentKey",
            "mappingFunction": { "name": "base64Encode" }
        }
    ],
    "outputFieldMappings": [
        { "sourceFieldName": "/document/reviews_text/pages/*/Keyphrases/*", "targetFieldName": "Keyphrases" },
        { "sourceFieldName": "/document/Language", "targetFieldName": "Language" },
        { "sourceFieldName": "/document/reviews_text/pages/*/Sentiment", "targetFieldName": "Sentiment" }
    ]
}
```

## <a name="run-the-indexer"></a>Запуск индексатора 

На портале Azure перейдите на страницу **обзора** службы "Когнитивный поиск Azure". Выберите вкладку **Индексаторы** и элемент **hotels-reviews-ixr**. Если индексатор еще ни разу не выполнялся, щелкните **Запустить**. Задача индексатора может выдать несколько предупреждений, относящихся к распознаванию языка. В наборе данных есть отзывы на некоторых языках, которые пока не поддерживаются в когнитивных навыках. 

## <a name="next-steps"></a>Дальнейшие действия

Итак, вы дополнили данные с помощью Cognitive Services и спроецировали результаты в хранилище знаний. Теперь можете изучить дополненный набор данных с помощью Обозревателя службы хранилища или Power BI.

Сведения о том, как исследовать это хранилище знаний с помощью Обозревателя службы хранилища, см. в следующем пошаговом руководстве.

> [!div class="nextstepaction"]
> [Просмотр данных с помощью Обозревателя службы хранилища](knowledge-store-view-storage-explorer.md)

Сведения о том, как подключить это хранилище знаний к Power BI, см. в следующем пошаговом руководстве.

> [!div class="nextstepaction"]
> [Подключение с помощью Power BI](knowledge-store-connect-power-bi.md)

Если вы хотите повторить это упражнение или выполнить инструкции из другого пошагового руководства по обогащению с помощью ИИ, удалите только индексатор **hotel-reviews-idxr**. При удалении индексатора счетчик бесплатных ежедневных транзакций будет обнулен.