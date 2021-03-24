---
title: Руководство по индексированию частично структурированных данных в больших двоичных объектах JSON
titleSuffix: Azure Cognitive Search
description: Сведения об индексировании и поиске частично структурированных больших двоичных объектов JSON в Azure с помощью интерфейсов REST API службы "Когнитивный поиск Azure" и Postman.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 01/25/2021
ms.openlocfilehash: a7a010e3c60d6b96947597878fcd870e9845b2b3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98746142"
---
# <a name="tutorial-index-json-blobs-from-azure-storage-using-rest"></a>Руководство по индексированию больших двоичных объектов JSON из службы хранилища Azure с помощью REST

Служба "Когнитивный поиск Azure" может индексировать документы и массивы JSON в хранилище BLOB-объектов Azure с помощью [индексатора](search-indexer-overview.md), который умеет читать полуструктурированные данные. Частично структурированные данные содержат теги или метки, отделяющие содержимое в данных. Этим они отличаются от неструктурированных данных, которые должны быть полностью проиндексированы, и от формально структурированных данных, которые соответствуют модели данных, например схеме реляционной базы данных, и поддерживают индексирование по отдельным полям.

В этом учебнике используется Postman и [REST API поиска](/rest/api/searchservice/), чтобы выполнять следующие задачи:

> [!div class="checklist"]
> * Настройка источника данных службы "Когнитивный поиск Azure" для контейнера больших двоичных объектов.
> * Создание индекса службы "Когнитивный поиск Azure", в котором будет находится содержимое для поиска.
> * Настройка и запуск индексатора для чтения контейнера и извлечения содержимого для поиска из хранилища BLOB-объектов Azure.
> * Поиск по индексу, который вы только что создали.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

+ [Хранилище Azure](../storage/common/storage-account-create.md)
+ [Классическое приложение Postman](https://www.getpostman.com/)
+ [Создайте службу поиска](search-create-service-portal.md) или [найдите существующую службу](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) 

> [!Note]
> Для выполнения инструкций из этого руководства вы можете использовать бесплатную версию службы. В бесплатной версии вы можете использовать не более трех индексов, трех индексаторов и трех источников данных. В этом руководстве создается по одному объекту из каждой категории. Перед началом работы убедитесь, что у службы есть достаточно места, чтобы принять новые ресурсы.

## <a name="download-files"></a>Загрузка файлов

Архив [clinical-trials-json.zip](https://github.com/Azure-Samples/storage-blob-integration-with-cdn-search-hdi/raw/master/clinical-trials-json.zip) содержит данные, используемые в этом учебнике. Скачайте этот файл и распакуйте его в собственной папке. Источником данных, преобразованных в JSON для этого учебника, является [clinicaltrials.gov](https://clinicaltrials.gov/ct2/results).

## <a name="1---create-services"></a>1\. Создание служб

В этом учебнике используются Когнитивный поиск Azure для индексирования и запросов, а также хранилище BLOB-объектов Azure для предоставления данных. 

Желательно создать все службы в одном регионе и одной группе ресурсов, чтобы упростить взаимодействие и управление. В реальной системе учетная запись хранения Azure может находиться в любом регионе.

### <a name="start-with-azure-storage"></a>Служба хранилища Azure

1. [Войдите на портал Azure](https://portal.azure.com/) и щелкните **+ Создать ресурс**.

1. Выполните поиск по строке *учетная запись хранения* и выберите "Учетная запись хранения Microsoft".

   :::image type="content" source="media/cognitive-search-tutorial-blob/storage-account.png" alt-text="Создание учетной записи хранения" border="false":::

1. На вкладке "Основные сведения" нужно ввести следующие сведения. Сохраните значения по умолчанию для остальных параметров.

   + **Группа ресурсов**. Вы можете выбрать существующую группу ресурсов или создать новую, но для всех служб это должна быть одна и та же группа для совместного управления.

   + **Имя учетной записи хранения.** Если вы ожидаете, что у вас будет несколько ресурсов одного типа, используйте имена, которые будут четко определять тип и регион, например *blobstoragewestus*. 

   + **Расположение.** Желательно выбрать то же расположение, которое использовалось для Когнитивного поиска Azure и Cognitive Services. При использовании одного расположения плата за трафик не взимается.

   + **Тип учетной записи.** Выберите значение по умолчанию *StorageV2 (общего назначения версии 2)* .

1. Щелкните **Просмотреть и создать**, чтобы создать службу.

1. По завершении процесса щелкните **Перейти к ресурсу** чтобы открыть страницу со сводкой.

1. Щелкните службу **Большие двоичные объекты**.

1. [Создайте контейнер больших двоичных объектов](../storage/blobs/storage-quickstart-blobs-portal.md) для хранения примера данных. Можно задать любое из допустимых значений уровня общего доступа.

1. Откройте контейнер после создания и на панели команд выберите **Загрузить**.

   :::image type="content" source="media/search-semi-structured-data/upload-command-bar.png" alt-text="Отправка на панели команд" border="false":::

1. Перейдите к папке, содержащей примеры файлов. Выберите их, а затем щелкните **Загрузить**.

   :::image type="content" source="media/search-semi-structured-data/clinicalupload.png" alt-text="Передача файлов" border="false":::

После завершения отправки файлы должны появиться в собственной вложенной папке внутри контейнера данных.

### <a name="azure-cognitive-search"></a>Когнитивный поиск Azure

Следующий ресурс — Когнитивный поиск Azure, который можно [создать на портале](search-create-service-portal.md). Для выполнения действий в этом пошаговом руководстве можно использовать уровень "Бесплатный". 

Как и с хранилищем BLOB-объектов Azure, не забудьте сохранить ключ доступа. Позже, когда вы начнете структурировать запросы, адрес конечной точки и административный ключ интерфейса нужно будет указывать в каждом запросе для проверки подлинности.

### <a name="get-a-key-and-url"></a>Получение ключа и URL-адреса

Вызовам REST требуется URL-адрес службы и ключ доступа при каждом запросе. Служба поиска создана с обоими элементами, поэтому если вы добавили службу "Когнитивный поиск Azure" в подписку, выполните следующие действия для получения необходимых сведений:

1. [Войдите на портал Azure](https://portal.azure.com/) и на странице **обзора** службы поиска получите URL-адрес. Пример конечной точки может выглядеть так: `https://mydemo.search.windows.net`.

1. В разделе **Параметры** > **Ключи** получите ключ администратора, чтобы обрести полные права на службу. Существуют два взаимозаменяемых ключа администратора, предназначенных для обеспечения непрерывности бизнес-процессов на случай, если вам потребуется сменить один из них. Вы можете использовать первичный или вторичный ключ для выполнения запросов на добавление, изменение и удаление объектов.

   :::image type="content" source="media/search-get-started-rest/get-url-key.png" alt-text="Получение конечной точки HTTP и ключа доступа" border="false":::

Для выполнения любого запроса к службе требуется использование ключа API. Если есть действительный ключ, для каждого запроса устанавливаются отношения доверия между приложением, которое отправляет запрос, и службой, которая его обрабатывает.

## <a name="2---set-up-postman"></a>2\. Настройка Postman

Запустите Postman и настройте HTTP-запрос. Если вы не работали с этим инструментом, изучите статью [Краткое руководство. Создание индекса службы "Когнитивный поиск Azure" с помощью интерфейсов REST API](search-get-started-rest.md).

Методы запроса для каждого вызова в этом учебнике — **POST** и **GET**. Вы сделаете три вызова API службы поиска для создания источника данных, индекса и индексатора. Источник данных содержит указатель к вашей учетной записи хранения и данным JSON. Служба поиска создает подключение при отправке данных.

В разделе "Заголовки" присвойте параметру Content-type значение `application/json`, а в параметре `api-key` сохраните значение ключа администрирования для службы "Когнитивный поиск Azure". Единожды выполнив настройку заголовков, вы сможете применять их для всех запросов в этом руководстве.

  :::image type="content" source="media/search-get-started-rest/postman-url.png" alt-text="URL-адрес и заголовок запроса Postman" border="false":::

Универсальный код ресурса (URI) должен указывать версию API, и каждый вызов должен возвращать ответ **201 Created** (201 — Создан ресурс). Общедоступная версия API (api-version) для использования массивов JSON — `2020-06-30`.

## <a name="3---create-a-data-source"></a>3\. Создание источника данных

[API создания источника данных](/rest/api/searchservice/create-data-source) создает объект службы "Когнитивный поиск Azure", в котором указываются данные для индексирования.

1. Задайте для конечной точки этого вызова значение `https://[service name].search.windows.net/datasources?api-version=2020-06-30`. Замените `[service name]` именем службы поиска. 

1. Скопируйте следующий код JSON в текст запроса.

    ```json
    {
        "name" : "clinical-trials-json-ds",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=[storage account name];AccountKey=[storage account key];" },
        "container" : { "name" : "[blob container name]"}
    }
    ```

1. Замените строку подключения допустимой строкой для вашей учетной записи.

1. Замените "[имя контейнера BLOB-объектов]" контейнером, созданным для примера данных. 

1. Отправьте запрос. Результат должен выглядеть следующим образом:

    ```json
    {
        "@odata.context": "https://exampleurl.search.windows.net/$metadata#datasources/$entity",
        "@odata.etag": "\"0x8D505FBC3856C9E\"",
        "name": "clinical-trials-json-ds",
        "description": null,
        "type": "azureblob",
        "subtype": null,
        "credentials": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=[mystorageaccounthere];AccountKey=[[myaccountkeyhere]]];"
        },
        "container": {
            "name": "[mycontainernamehere]",
            "query": null
        },
        "dataChangeDetectionPolicy": null,
        "dataDeletionDetectionPolicy": null
    }
    ```

## <a name="4---create-an-index"></a>4\. Создание индекса

Второй вызов — [API создания индекса](/rest/api/searchservice/create-index), создающий индекс службы "Когнитивный поиск Azure", в котором хранятся все доступные для поиска данные. Индекс указывает все параметры и их атрибуты.

1. Задайте для конечной точки этого вызова значение `https://[service name].search.windows.net/indexes?api-version=2020-06-30`. Замените `[service name]` именем службы поиска.

1. Скопируйте следующий код JSON в текст запроса.

    ```json
    {
      "name": "clinical-trials-json-index",  
      "fields": [
      {"name": "FileName", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": false, "filterable": false, "sortable": true},
      {"name": "Description", "type": "Edm.String", "searchable": true, "retrievable": false, "facetable": false, "filterable": false, "sortable": false},
      {"name": "MinimumAge", "type": "Edm.Int32", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": true},
      {"name": "Title", "type": "Edm.String", "searchable": true, "retrievable": true, "facetable": false, "filterable": true, "sortable": true},
      {"name": "URL", "type": "Edm.String", "searchable": false, "retrievable": false, "facetable": false, "filterable": false, "sortable": false},
      {"name": "MyURL", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": false, "filterable": false, "sortable": false},
      {"name": "Gender", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
      {"name": "MaximumAge", "type": "Edm.Int32", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": true},
      {"name": "Summary", "type": "Edm.String", "searchable": true, "retrievable": true, "facetable": false, "filterable": false, "sortable": false},
      {"name": "NCTID", "type": "Edm.String", "key": true, "searchable": true, "retrievable": true, "facetable": false, "filterable": true, "sortable": true},
      {"name": "Phase", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
      {"name": "Date", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": false, "filterable": false, "sortable": true},
      {"name": "OverallStatus", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
      {"name": "OrgStudyId", "type": "Edm.String", "searchable": true, "retrievable": true, "facetable": false, "filterable": true, "sortable": false},
      {"name": "HealthyVolunteers", "type": "Edm.String", "searchable": false, "retrievable": true, "facetable": true, "filterable": true, "sortable": false},
      {"name": "Keywords", "type": "Collection(Edm.String)", "searchable": true, "retrievable": true, "facetable": true, "filterable": false, "sortable": false},
      {"name": "metadata_storage_last_modified", "type":"Edm.DateTimeOffset", "searchable": false, "retrievable": true, "filterable": true, "sortable": false},
      {"name": "metadata_storage_size", "type":"Edm.String", "searchable": false, "retrievable": true, "filterable": true, "sortable": false},
      {"name": "metadata_content_type", "type":"Edm.String", "searchable": true, "retrievable": true, "filterable": true, "sortable": false}
      ]
    }
   ```

1. Отправьте запрос. Результат должен выглядеть следующим образом:

    ```json
    {
        "@odata.context": "https://exampleurl.search.windows.net/$metadata#indexes/$entity",
        "@odata.etag": "\"0x8D505FC00EDD5FA\"",
        "name": "clinical-trials-json-index",
        "fields": [
            {
                "name": "FileName",
                "type": "Edm.String",
                "searchable": false,
                "filterable": false,
                "retrievable": true,
                "sortable": true,
                "facetable": false,
                "key": false,
                "indexAnalyzer": null,
                "searchAnalyzer": null,
                "analyzer": null,
                "synonymMaps": []
            },
            {
                "name": "Description",
                "type": "Edm.String",
                "searchable": true,
                "filterable": false,
                "retrievable": false,
                "sortable": false,
                "facetable": false,
                "key": false,
                "indexAnalyzer": null,
                "searchAnalyzer": null,
                "analyzer": null,
                "synonymMaps": []
            },
            ...
          }
    ```

## <a name="5---create-and-run-an-indexer"></a>5\. Создание и запуск индексатора

Индексатор подключает источник данных, импортирует данные в целевой индекс поиска и при необходимости предоставляет расписание для автоматизации обновления данных. REST API — [создание индексатора](/rest/api/searchservice/create-indexer).

1. Задайте для универсального кода ресурса (URI) этого вызова значение `https://[service name].search.windows.net/indexers?api-version=2020-06-30`. Замените `[service name]` именем службы поиска.

1. Скопируйте следующий код JSON в текст запроса.

    ```json
    {
      "name" : "clinical-trials-json-indexer",
      "dataSourceName" : "clinical-trials-json-ds",
      "targetIndexName" : "clinical-trials-json-index",
      "parameters" : { "configuration" : { "parsingMode" : "jsonArray" } }
    }
    ```

1. Отправьте запрос. Запрос обрабатывается немедленно. При возвращении ответа вы получите индекс с возможностью полнотекстового поиска. Результат должен выглядеть следующим образом:

    ```json
    {
        "@odata.context": "https://exampleurl.search.windows.net/$metadata#indexers/$entity",
        "@odata.etag": "\"0x8D505FDE143D164\"",
        "name": "clinical-trials-json-indexer",
        "description": null,
        "dataSourceName": "clinical-trials-json-ds",
        "targetIndexName": "clinical-trials-json-index",
        "schedule": null,
        "parameters": {
            "batchSize": null,
            "maxFailedItems": null,
            "maxFailedItemsPerBatch": null,
            "base64EncodeKeys": null,
            "configuration": {
                "parsingMode": "jsonArray"
            }
        },
        "fieldMappings": [],
        "enrichers": [],
        "disabled": null
    }
    ```

## <a name="6---search-your-json-files"></a>6\. Поиск JSON-файлов

Вы можете начать поиск сразу после загрузки первого документа.

1. Измените команду на **GET**.

1. Задайте для универсального кода ресурса (URI) этого вызова значение `https://[service name].search.windows.net/indexes/clinical-trials-json-index/docs?search=*&api-version=2020-06-30&$count=true`. Замените `[service name]` именем службы поиска.

1. Отправьте запрос. Это неопределенный полнотекстовый поисковый запрос, который возвращает все поля, помеченные как доступные для получения в индексе, вместе с числом документов. Результат должен выглядеть следующим образом:

    ```json
    {
        "@odata.context": "https://exampleurl.search.windows.net/indexes('clinical-trials-json-index')/$metadata#docs(*)",
        "@odata.count": 100,
        "value": [
            {
                "@search.score": 1.0,
                "FileName": "NCT00000102.txt",
                "MinimumAge": 14,
                "Title": "Congenital Adrenal Hyperplasia: Calcium Channels as Therapeutic Targets",
                "MyURL": "https://azure.storagedemos.com/clinical-trials/NCT00000102.txt",
                "Gender": "Both",
                "MaximumAge": 35,
                "Summary": "This study will test the ability of extended release nifedipine (Procardia XL), a blood pressure medication, to permit a decrease in the dose of glucocorticoid medication children take to treat congenital adrenal hyperplasia (CAH).",
                "NCTID": "NCT00000102",
                "Phase": "Phase 1/Phase 2",
                "Date": "ClinicalTrials.gov processed this data on October 25, 2016",
                "OverallStatus": "Completed",
                "OrgStudyId": "NCRR-M01RR01070-0506",
                "HealthyVolunteers": "No",
                "Keywords": [],
                "metadata_storage_last_modified": "2019-04-09T18:16:24Z",
                "metadata_storage_size": "33060",
                "metadata_content_type": null
            },
            . . . 
    ```

1. Добавьте параметр запроса `$select`, чтобы ограничить результаты меньшим числом полей: `https://[service name].search.windows.net/indexes/clinical-trials-json-index/docs?search=*&$select=Gender,metadata_storage_size&api-version=2020-06-30&$count=true`.  Для этого запроса 100 документов совпадают, но по умолчанию Когнитивный поиск Azure возвращает в результатах только 50.

   :::image type="content" source="media/search-semi-structured-data/lastquery.png" alt-text="Параметризованный запрос" border="false":::

1. Пример более сложного запроса включал бы в себя элемент `$filter=MinimumAge ge 30 and MaximumAge lt 75`, который возвращает только результаты, в которых значение параметра MinimumAge не меньше 30, а MaximumAge — не больше 75. Замените выражение `$select` выражением `$filter`.

   :::image type="content" source="media/search-semi-structured-data/metadatashort.png" alt-text="Частично структурированный поиск" border="false":::

Вы можете также использовать логические операторы (and, or, not) и операторы сравнения (eq, ne, gt, lt, ge, le). При сравнении строк учитывается регистр. Дополнительные сведения о создании простого запроса и примеры см. в [этой статье](search-query-simple-examples.md).

> [!NOTE]
> Параметр `$filter` работает только с метаданными, которые отмечены как фильтруемые при создании индекса.

## <a name="reset-and-rerun"></a>Сброс и повторный запуск

На ранних экспериментальных этапах разработки самый практичный подход к итерации схемы — удалить все объекты из службы "Когнитивный поиск Azure" и восстановить их с помощью кода. Имена ресурсов являются уникальными. Удаление объекта позволяет воссоздать его с использованием того же имени.

Для удаления индексов, индексаторов и источников данных вы также можете использовать портал. Или используйте **DELETE** и укажите URL-адреса для каждого объекта. Следующая команда удаляет индексатор.

```http
DELETE https://[YOUR-SERVICE-NAME].search.windows.net/indexers/clinical-trials-json-indexer?api-version=2020-06-30
```

При успешном удалении возвращается код состояния 204.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы работаете в своей подписке, после завершения проекта целесообразно удалить созданные ресурсы, которые вам больше не потребуются. Работающие ресурсы могут означать лишние затраты. Можно удалить отдельные ресурсы или удалить группу ресурсов, что позволит удалить весь набор ресурсов.

Просматривать ресурсы и управлять ими можно на портале с помощью ссылок "Все ресурсы" или "Группы ресурсов" в области навигации слева.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы знакомы с основами индексирования больших двоичных объектов Azure, давайте подробнее рассмотрим конфигурацию индексатора для больших двоичных объектов JSON в службе хранилища Azure.

> [!div class="nextstepaction"]
> [Настройка индексирования больших двоичных объектов JSON](search-howto-index-json-blobs.md).