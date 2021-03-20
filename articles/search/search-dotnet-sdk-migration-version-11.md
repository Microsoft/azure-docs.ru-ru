---
title: Обновление до пакета SDK для .NET версии 11
titleSuffix: Azure Cognitive Search
description: Перенос кода в пакет SDK Azure Когнитивный поиск .NET версии 11 с предыдущих версий. Сведения о новых возможностях и необходимых изменениях кода.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 01/07/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: df8841cb2dcac6335b09a5e7715f42c508c69e76
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99536821"
---
# <a name="upgrade-to-azure-cognitive-search-net-sdk-version-11"></a>Обновление до Azure Когнитивный поиск .NET SDK версии 11

Если вы используете [пакет SDK для .NET](/dotnet/api/overview/azure/search)версии 10,0 или более ранней, эта статья поможет вам выполнить обновление до версии 11 и клиентской библиотеки **Azure.Search.Docументс** .

Версия 11 — это полностью переработанная клиентская библиотека, выпущенная группой разработки пакета SDK для Azure (предыдущие версии были созданы командой разработчиков Когнитивный поиск Azure). Библиотека была переработана для обеспечения большей согласованности с другими клиентскими библиотеками Azure, учитывая зависимость от [Azure. Core](/dotnet/api/azure.core) и [System.Text.Js](/dotnet/api/system.text.json)и реализуя привычные подходы к распространенным задачам.

Ниже приведены некоторые ключевые отличия, которые можно заметить в новой версии.

+ Один пакет и библиотека, а не несколько
+ Новое имя пакета: `Azure.Search.Documents` вместо `Microsoft.Azure.Search` .
+ Три клиента вместо двух: `SearchClient` , `SearchIndexClient` , `SearchIndexerClient`
+ Различия в именовании в диапазоне API и небольших структурных отличий, упрощающих некоторые задачи

В дополнение к этой статье вы можете просмотреть [Журнал изменений](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/CHANGELOG.md) с детализированным списком изменений в пакете SDK для .NET версии 11.

## <a name="package-and-library-consolidation"></a>Консолидация пакетов и библиотек

Версия 11 объединяет несколько пакетов и библиотек в одну. После миграции у вас будет меньше библиотек для управления.

+ [Azure.Search.Docпакет ументс](https://www.nuget.org/packages/Azure.Search.Documents/)

+ [Справочник по API для клиентской библиотеки](/dotnet/api/overview/azure/search.documents-readme)

## <a name="client-differences"></a>Различия клиента

Если это применимо, в следующей таблице сопоставлены клиентские библиотеки между двумя версиями.

| Область действия | Microsoft. Azure. Search &nbsp; (V10) | Azure.Search.Docументс &nbsp; (версии 11) |
|---------------------|------------------------------|------------------------------|
| Клиент, используемый для запросов и заполнения индекса. | [SearchIndexClient](/dotnet/api/azure.search.documents.indexes.searchindexclient) | [сеарчклиент](/dotnet/api/azure.search.documents.searchclient) |
| Клиент, используемый для индексов, анализаторов, сопоставлений синонимов | [SearchServiceClient](/dotnet/api/microsoft.azure.search.searchserviceclient) | [SearchIndexClient](/dotnet/api/azure.search.documents.indexes.searchindexclient) |
| Клиент, используемый для индексаторов, источников данных, навыков | [SearchServiceClient](/dotnet/api/microsoft.azure.search.searchserviceclient) | [Сеарчиндексерклиент (**новое**)](/dotnet/api/azure.search.documents.indexes.searchindexerclient) |

> [!Important]
> `SearchIndexClient` существует в обеих версиях, но поддерживает различные вещи. В версии 10 `SearchIndexClient` Создайте индексы и другие объекты. В версии 11 `SearchIndexClient` работает с существующими индексами. Чтобы избежать путаницы при обновлении кода, учитывать порядок обновления клиентских ссылок. После выполнения последовательности [действий по обновлению](#UpgradeSteps) необходимо устранить любые проблемы замены строк.

<a name="naming-differences"></a>

## <a name="naming-and-other-api-differences"></a>Различия в именовании и других API

Помимо отличий клиентов (отмеченных ранее, и, таким, здесь опущено), несколько других API были переименованы, а в некоторых случаях переработано. Ниже приведены отличия имен классов. Этот список не является исчерпывающим, но в нем изменения API группируются по задачам, что может быть полезным для редакций конкретных блоков кода. Детализированный список обновлений API см. в [журнале изменений](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/CHANGELOG.md) `Azure.Search.Documents` на GitHub.

### <a name="authentication-and-encryption"></a>Проверка подлинности и шифрование

| Версия 10 | Эквивалент версии 11 |
|------------|-----------------------|
| [сеарчкредентиалс](/dotnet/api/microsoft.azure.search.searchcredentials) | [азурекэйкредентиал](/dotnet/api/azure.azurekeycredential) |
| `EncryptionKey` (существовало в [предварительной версии пакета SDK](https://www.nuget.org/packages/Microsoft.Azure.Search/8.0.0-preview) как общедоступная функция) | [сеарчресаурцеенкриптионкэй](/dotnet/api/azure.search.documents.indexes.models.searchresourceencryptionkey) |

### <a name="indexes-analyzers-synonym-maps"></a>Индексы, анализаторы, сопоставления синонимов

| Версия 10 | Эквивалент версии 11 |
|------------|-----------------------|
| [Index](/dotnet/api/microsoft.azure.documents.index) | [сеарчиндекс](/dotnet/api/azure.search.documents.indexes.models.searchindex) |
| [Поле](/dotnet/api/microsoft.azure.search.models.field) | [сеарчфиелд](/dotnet/api/azure.search.documents.indexes.models.searchfield) |
| [DataType](/dotnet/api/microsoft.azure.search.models.datatype) | [сеарчфиелддататипе](/dotnet/api/azure.search.documents.indexes.models.searchfielddatatype) |
| [итемеррор](/dotnet/api/microsoft.azure.search.models.itemerror) | [сеарчиндексереррор](/dotnet/api/azure.search.documents.indexes.models.searchindexererror) |
| [Анализатор](/dotnet/api/microsoft.azure.search.models.analyzer) | [Лексикаланализер](/dotnet/api/azure.search.documents.indexes.models.lexicalanalyzer) (также `AnalyzerName` `LexicalAnalyzerName` ) |
| [анализерекуест](/dotnet/api/microsoft.azure.search.models.analyzerequest) | [анализетекстоптионс](/dotnet/api/azure.search.documents.indexes.models.analyzetextoptions) |
| [StandardAnalyzer](/dotnet/api/microsoft.azure.search.models.standardanalyzer) | [луценестандарданализер](/dotnet/api/azure.search.documents.indexes.models.lucenestandardanalyzer) |
| [стандардтокенизер](/dotnet/api/microsoft.azure.search.models.standardtokenizer) | [Луценестандардтокенизер](/dotnet/api/azure.search.documents.indexes.models.lucenestandardtokenizer) (также `StandardTokenizerV2` `LuceneStandardTokenizerV2` ) |
| [токенинфо](/dotnet/api/microsoft.azure.search.models.tokeninfo) | [анализедтокенинфо](/dotnet/api/azure.search.documents.indexes.models.analyzedtokeninfo) |
| [Разметчика](/dotnet/api/microsoft.azure.search.models.tokenizer) | [Лексикалтокенизер](/dotnet/api/azure.search.documents.indexes.models.lexicaltokenizer) (также `TokenizerName` `LexicalTokenizerName` ) |
| [SynonymMap. Format](/dotnet/api/microsoft.azure.search.models.synonymmap.format) | Отсутствует. Удалите ссылки на `Format` . |

Определения полей упрощаются: [сеарчаблефиелд](/dotnet/api/azure.search.documents.indexes.models.searchablefield), [симплефиелд](/dotnet/api/azure.search.documents.indexes.models.simplefield), [Комплексфиелд](/dotnet/api/azure.search.documents.indexes.models.complexfield) — это новые интерфейсы API для создания определений полей.

### <a name="indexers-datasources-skillsets"></a>Индексаторы, источники данных, навыков

| Версия 10 | Эквивалент версии 11 |
|------------|-----------------------|
| [Индексатор](/dotnet/api/microsoft.azure.search.models.indexer) | [сеарчиндексер](/dotnet/api/azure.search.documents.indexes.models.searchindexer) |
| [DataSource](/dotnet/api/microsoft.azure.search.models.datasource) | [сеарчиндексердатасаурцеконнектион](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourceconnection) |
| [Навык](/dotnet/api/microsoft.azure.search.models.skill) | [сеарчиндексерскилл](/dotnet/api/azure.search.documents.indexes.models.searchindexerskill) |
| [Набор навыков](/dotnet/api/microsoft.azure.search.models.skillset) | [сеарчиндексерскиллсет](/dotnet/api/azure.search.documents.indexes.models.searchindexerskill) |
| [DataSourceType](/dotnet/api/microsoft.azure.search.models.datasourcetype) | [сеарчиндексердатасаурцетипе](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourcetype) |

### <a name="data-import"></a>Импорт данных

| Версия 10 | Эквивалент версии 11 |
|------------|-----------------------|
| [IndexAction](/dotnet/api/microsoft.azure.search.models.indexaction) | [индексдокументсактион](/dotnet/api/azure.search.documents.models.indexdocumentsaction) |
| [IndexBatch](/dotnet/api/microsoft.azure.search.models.indexbatch) | [индексдокументсбатч](/dotnet/api/azure.search.documents.models.indexdocumentsbatch) |

### <a name="query-definitions-and-results"></a>Определения и результаты запросов

| Версия 10 | Эквивалент версии 11 |
|------------|-----------------------|
| [документсеарчресулт](/dotnet/api/microsoft.azure.search.models.documentsearchresult-1) | [SearchResult](/dotnet/api/azure.search.documents.models.searchresult-1) или [SearchResults](/dotnet/api/azure.search.documents.models.searchresults-1)в зависимости от того, является ли результат отдельным документом или несколькими. |
| [документсугжестресулт](/dotnet/api/microsoft.azure.search.models.documentsuggestresult-1) | [сугжестресултс](/dotnet/api/azure.search.documents.models.suggestresults-1) |
| [SearchParameters](/dotnet/api/microsoft.azure.search.models.searchparameters) |  [сеарчоптионс](/dotnet/api/azure.search.documents.searchoptions)  |

### <a name="json-serialization"></a>Сериализация JSON

По умолчанию пакет Azure SDK использует [System.Text.Js](/dotnet/api/system.text.json) для сериализации JSON, полагается на возможности этих API для работы с преобразованиями текста, ранее реализованными через собственный класс [сериализепропертинамесаскамелкасеаттрибуте](/dotnet/api/microsoft.azure.search.models.serializepropertynamesascamelcaseattribute) , который не имеет аналога в новой библиотеке.

Чтобы сериализовать имена свойств в camelCase, можно использовать [жсонпропертинамеаттрибуте](/dotnet/api/system.text.json.serialization.jsonpropertynameattribute) (как в [этом примере](https://github.com/Azure/azure-sdk-for-net/tree/d263f23aa3a28ff4fc4366b8dee144d4c0c3ab10/sdk/search/Azure.Search.Documents#use-c-types-for-search-results)).

Кроме того, можно задать [жсоннамингполици](/dotnet/api/system.text.json.jsonnamingpolicy) , предоставленный в [жсонсериализероптионс](/dotnet/api/system.text.json.jsonserializeroptions). Следующий System.Text.Jsв примере кода, взятом из [файла readme Microsoft. Azure. Core. пространственных](https://github.com/Azure/azure-sdk-for-net/blob/259df3985d9710507e2454e1591811f8b3a7ad5d/sdk/core/Microsoft.Azure.Core.Spatial/README.md#deserializing-documents) данных, демонстрирует использование camelCase без необходимости применения атрибута для каждого свойства:

```csharp
// Get the Azure Cognitive Search endpoint and read-only API key.
Uri endpoint = new Uri(Environment.GetEnvironmentVariable("SEARCH_ENDPOINT"));
AzureKeyCredential credential = new AzureKeyCredential(Environment.GetEnvironmentVariable("SEARCH_API_KEY"));

// Create serializer options with our converter to deserialize geographic points.
JsonSerializerOptions serializerOptions = new JsonSerializerOptions
{
    Converters =
    {
        new MicrosoftSpatialGeoJsonConverter()
    },
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase
};

SearchClientOptions clientOptions = new SearchClientOptions
{
    Serializer = new JsonObjectSerializer(serializerOptions)
};

SearchClient client = new SearchClient(endpoint, "mountains", credential, clientOptions);
Response<SearchResults<Mountain>> results = client.Search<Mountain>("Rainier");
```

При использовании Newtonsoft.Jsдля сериализации JSON можно передать глобальные политики именования с помощью аналогичных атрибутов или с помощью свойств в [JsonSerializerSettings](https://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_JsonSerializerSettings.htm). Пример, эквивалентный приведенному выше, см. в [примере десериализация документов](https://github.com/Azure/azure-sdk-for-net/blob/259df3985d9710507e2454e1591811f8b3a7ad5d/sdk/core/Microsoft.Azure.Core.Spatial.NewtonsoftJson/README.md) в Newtonsoft.Jsна странице readme.


<a name="WhatsNew"></a>

## <a name="whats-in-version-11"></a>Что находится в версии 11

Каждая версия клиентской библиотеки Azure Когнитивный поиск предназначена для соответствующей версии REST API. REST API считается основой службы, а отдельные пакеты SDK обтекает версию REST API. Разработчикам .NET может быть полезно ознакомиться с [REST API документацией](/rest/api/searchservice/) , если вы хотите получить дополнительные сведения о конкретных объектах или операциях.

Версия 11 предназначена для [службы поиска 2020-06-30](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/search/data-plane/Azure.Search/preview/2020-06-30/searchservice.json). Поскольку версия 11 также является новой клиентской библиотекой, созданной с нуля, большая часть усилий по разработке посвящена эквивалентности версии 10, а некоторые REST API функции все еще находятся в состоянии ожидания.

Версия 11,0 полностью поддерживает следующие объекты и операции:

+ Создание и управление индексами
+ Создание и управление картой синонимов
+ Все типы запросов и синтаксис (за исключением геопространственных фильтров)
+ Объекты и операции индексатора для индексирования источников данных Azure, включая источники данных и навыков

Версия 11,1 добавляет следующее:

+ [Фиелдбуилдер](/dotnet/api/azure.search.documents.indexes.fieldbuilder) (добавлено в 11,1)
+ [Свойство сериализатора](/dotnet/api/azure.search.documents.searchclientoptions.serializer) (добавленное в 11,1) для поддержки пользовательской сериализации

### <a name="pending-features"></a>Ожидающие функции

Следующие функции версии 10 пока недоступны в версии 11. Если вам нужны эти функции, отключите миграцию до тех пор, пока они не будут поддерживаться.

+ [Хранилище знаний](knowledge-store-concept-intro.md)
+ Геопространственные типы — поддержка первого класса для геопространственных типов по-прежнему выполняется. Теперь для поддержки географических операций можно использовать пакет [Microsoft. spatial](https://www.nuget.org/packages/Microsoft.Spatial/) . Примеры доступны для [System.Text.Js](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/core/Microsoft.Azure.Core.Spatial/README.md) и [Newtonsoft.Js](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/core/Microsoft.Azure.Core.Spatial.NewtonsoftJson/README.md).

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>Действия по обновлению

Следующие шаги помогут приступить к переносу кода с помощью прохода по первому набору обязательных задач, особенно в отношении клиентских ссылок.

1. Установите [Azure.Search.Docпакет ументс](https://www.nuget.org/packages/Azure.Search.Documents/) , щелкнув правой кнопкой мыши ссылку на проект и выбрав пункт "Управление пакетами NuGet...". в Visual Studio.

1. Замените директивы using для Microsoft. Azure. Search следующим:

   ```csharp
   using Azure;
   using Azure.Search.Documents;
   using Azure.Search.Documents.Indexes;
   using Azure.Search.Documents.Indexes.Models;
   using Azure.Search.Documents.Models;
   ```

1. Для классов, требующих сериализации JSON, замените `using Newtonsoft.Json` на `using System.Text.Json.Serialization` .

1. Пересмотр кода проверки подлинности клиента. В предыдущих версиях вы использовали свойства клиентского объекта для задания ключа API (например, свойства [SearchServiceClient. Credentials](/dotnet/api/microsoft.azure.search.searchserviceclient.credentials) ). В текущей версии используйте класс [азурекэйкредентиал](/dotnet/api/azure.azurekeycredential) для передачи ключа в качестве учетных данных, чтобы при необходимости можно было обновить ключ API, не создавая новые клиентские объекты.

   Свойства клиента были упрощены `Endpoint` , `ServiceName` и `IndexName` (где это уместно). В следующем примере используется класс [URI](/dotnet/api/system.uri) системы для предоставления конечной точки и класса [среды](/dotnet/api/system.environment) для считывания значения ключа:

   ```csharp
   Uri endpoint = new Uri(Environment.GetEnvironmentVariable("SEARCH_ENDPOINT"));
   AzureKeyCredential credential = new AzureKeyCredential(
      Environment.GetEnvironmentVariable("SEARCH_API_KEY"));
   SearchIndexClient indexClient = new SearchIndexClient(endpoint, credential);
   ```

1. Добавление новых клиентских ссылок для объектов, связанных с индексатором. Если используются индексаторы, источники данных или навыков, измените ссылки клиента на [сеарчиндексерклиент](/dotnet/api/azure.search.documents.indexes.searchindexerclient). Этот клиент впервые помещается в версии 11 и не имеет предшествующей задачи.

1. Пересмотреть коллекции и списки. В новом пакете SDK все списки доступны только для чтения, чтобы избежать возникновения проблем, если список содержит значения NULL. Изменение кода заключается в добавлении элементов в список. Например, вместо того, чтобы назначать строки свойству Select, необходимо добавить их следующим образом:

   ```csharp
   var options = new SearchOptions
    {
       SearchMode = SearchMode.All,
       IncludeTotalCount = true
    };

    // Select fields to return in results.
    options.Select.Add("HotelName");
    options.Select.Add("Description");
    options.Select.Add("Tags");
    options.Select.Add("Rooms");
    options.Select.Add("Rating");
    options.Select.Add("LastRenovationDate");
   ```

   SELECT, аспекты, SearchFields, Саурцефиелдс, Скорингпараметерс и OrderBy — это все списки, которые теперь необходимо перестроить.

1. Обновление клиентских ссылок для запросов и импорта данных. Экземпляры [SearchIndexClient](/dotnet/api/microsoft.azure.search.searchindexclient) должны быть изменены на [сеарчклиент](/dotnet/api/azure.search.documents.searchclient). Чтобы избежать путаницы с именами, убедитесь, что все экземпляры перехватываются перед переходом к следующему шагу.

1. Обновление клиентских ссылок для объектов индексов, сопоставлений синонимов и анализаторов. Экземпляры [SearchServiceClient](/dotnet/api/microsoft.azure.search.searchserviceclient) должны быть изменены на [SearchIndexClient](/dotnet/api/microsoft.azure.search.searchindexclient). 

1. В оставшейся части кода обновите классы, методы и свойства, чтобы использовать API новой библиотеки. Раздел [различия имен](#naming-differences) можно запустить, но вы также можете ознакомиться с [журналом изменений](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/CHANGELOG.md).

   Если у вас возникли проблемы с поиском эквивалентных API-интерфейсов, мы рекомендуем регистрировать проблему, [https://github.com/MicrosoftDocs/azure-docs/issues](https://github.com/MicrosoftDocs/azure-docs/issues) чтобы мы могли улучшить документацию или исследовать проблему.

1. Выполните повторную сборку решения. После исправления ошибок или предупреждений сборки можно внести дополнительные изменения в приложение, чтобы воспользоваться преимуществами [новых функциональных возможностей](#WhatsNew).

<a name="ListOfChanges"></a>

## <a name="breaking-changes"></a>Критические изменения

Учитывая изменения в библиотеках и API, обновление до версии 11 не является тривиальной задачей и является критическим изменением в том смысле, что ваш код больше не будет обратно совместим с версией 10 и более ранними версиями. Подробное рассмотрение различий см. в [журнале изменений](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/CHANGELOG.md) для `Azure.Search.Documents` .

В плане обновлений версий служб, в которых изменения кода в версии 11 связаны с существующими функциями (а не только с рефакторингом API), вы обнаружите следующие изменения в работе:

+ [Алгоритм ранжирования BM25](index-ranking-similarity.md) заменяет предыдущий алгоритм ранжирования на новую технологию. Новые службы будут использовать этот алгоритм автоматически. Для существующих служб необходимо задать параметры для использования нового алгоритма.

+ [Упорядоченные результаты](search-query-odata-orderby.md) для значений NULL были изменены в этой версии, и значения NULL появлялись первыми, если сортировка имеет значение `asc` и последний `desc` . Если вы написали код, который обрабатывает, как сортируются значения NULL, следует проверить и, возможно, удалить этот код, если он больше не нужен.

## <a name="next-steps"></a>Дальнейшие действия

+ [Azure.Search.Docпакет ументс](https://www.nuget.org/packages/Azure.Search.Documents/)
+ [Примеры на GitHub](https://github.com/azure/azure-sdk-for-net/tree/Azure.Search.Documents_11.0.0/sdk/search/Azure.Search.Documents/samples)
+ [ Справочник по APIAzure.Search.Docумент](/dotnet/api/overview/azure/search.documents-readme)