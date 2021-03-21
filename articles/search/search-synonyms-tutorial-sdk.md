---
title: Пример синонимов в C#
titleSuffix: Azure Cognitive Search
description: В этом примере на C# вы узнаете, как добавить функцию синонимов в индекс в Когнитивный поиск Azure. Карта синонимов — это список эквивалентных терминов. Поля с поддержкой синонимов расширяют запросы, позволяя включать в них указанный пользователем термин и все его синонимы.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 12/18/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 50d5d73e71b8129f061ec49b363a0ebb13d22bdf
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97704662"
---
# <a name="example-add-synonyms-for-azure-cognitive-search-in-c"></a>Пример. Добавление синонимов для Когнитивный поиск Azure в C #

Синонимы позволяют расширить запрос с помощью сопоставления семантически эквивалентных терминов с входными терминами. Например, вы можете сопоставить термин "машина" с документами, в которых встречается слово "автомобиль" или "транспортное средство". 

В Когнитивный поиск Azure синонимы определяются в сопоставлении *синонимов* с помощью *правил сопоставления* , связывающих эквивалентные термины. В этом примере рассматриваются основные шаги по добавлению и использованию синонимов с существующим индексом.

В этом примере вы научитесь:

> [!div class="checklist"]
> * Создайте карту синонимов с помощью [класса SynonymMap](/dotnet/api/azure.search.documents.indexes.models.synonymmap). 
> * Задайте [свойство синониммапснаме](/dotnet/api/azure.search.documents.indexes.models.searchfield.synonymmapnames) для полей, которые должны поддерживать расширение запросов через синонимы.

Вы можете запросить поле с поддержкой синонимов как обычно. Для доступа к синонимам не требуется дополнительный синтаксис запроса.

Вы можете создать несколько сопоставлений синонимов, разместить их как ресурс служб, доступный для любого индекса, а затем указать ссылку на используемый на уровне поля. Во время выполнения запроса в дополнение к поиску индекса Когнитивный поиск Azure выполняет поиск в сопоставлении синонимов, если он указан в полях, используемых в запросе.

> [!NOTE]
> Синонимы можно создавать программно, но не на портале.

## <a name="prerequisites"></a>Предварительные условия

Ниже приведены предварительные требования, описанные в этом руководстве.

* [Visual Studio](https://www.visualstudio.com/downloads/)
* [Служба Когнитивный поиск Azure](search-create-service-portal.md)
* [Azure.Search.Docпакет ументс](https://www.nuget.org/packages/Azure.Search.Documents/)

Если вы не знакомы с клиентской библиотекой .NET, см. статью [использование когнитивный Поиск Azure в .NET](search-howto-dotnet-sdk.md).

## <a name="sample-code"></a>Пример кода

Полный исходный код примера приложения, используемого в этом примере, можно найти на сайте [GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToSynonyms).

## <a name="overview"></a>Обзор

Запросы Before и After используются для демонстрации значений синонимов. В этом примере пример приложения выполняет запросы и возвращает результаты на примере индекса гостиниц, заполненного двумя документами. Во первых, приложение выполняет поисковые запросы с использованием терминов и фраз, которые не отображаются в индексе. Во-вторых, код включает функцию синонимов, а затем повторно выдает те же запросы, возвращая результаты на основе совпадений в сопоставлении синонимов. 

В примере кода ниже показан общий поток.

```csharp
static void Main(string[] args)
{
   SearchIndexClient indexClient = CreateSearchIndexClient();

   Console.WriteLine("Cleaning up resources...\n");
   CleanupResources(indexClient);

   Console.WriteLine("Creating index...\n");
   CreateHotelsIndex(indexClient);

   SearchClient searchClient = indexClient.GetSearchClient("hotels");

   Console.WriteLine("Uploading documents...\n");
   UploadDocuments(searchClient);

   SearchClient searchClientForQueries = CreateSearchClientForQueries();

   RunQueriesWithNonExistentTermsInIndex(searchClientForQueries);

   Console.WriteLine("Adding synonyms...\n");
   UploadSynonyms(indexClient);

   Console.WriteLine("Enabling synonyms in the test index...\n");
   EnableSynonymsInHotelsIndexSafely(indexClient);
   Thread.Sleep(10000); // Wait for the changes to propagate

   RunQueriesWithNonExistentTermsInIndex(searchClientForQueries);

   Console.WriteLine("Complete.  Press any key to end application...\n");

   Console.ReadKey();
}
```

## <a name="before-queries"></a>Запросы "до"

С помощью `RunQueriesWithNonExistentTermsInIndex` отправляются запросы на поиск таких выражений, как "пять звезд", "Интернет" и "отели среднего класса".

Запросы фраз, например "пять звездочек", должны быть заключены в кавычки, а также могут потребовать escape-символов в зависимости от клиента.

```csharp
Console.WriteLine("Search the entire index for the phrase \"five star\":\n");
results = searchClient.Search<Hotel>("\"five star\"", searchOptions);
WriteDocuments(results);

Console.WriteLine("Search the entire index for the term 'internet':\n");
results = searchClient.Search<Hotel>("internet", searchOptions);
WriteDocuments(results);

Console.WriteLine("Search the entire index for the terms 'economy' AND 'hotel':\n");
results = searchClient.Search<Hotel>("economy AND hotel", searchOptions);
WriteDocuments(results);
```

Ни один из двух индексированных документов не содержит термины, поэтому мы получаем следующие выходные данные из первого `RunQueriesWithNonExistentTermsInIndex` :  **документ не соответствует**.

## <a name="enable-synonyms"></a>Включение поиска синонимов

После выполнения запросов "до" пример кода включает синонимы. Чтобы включить поиск синонимов, нам потребуется выполнить два действия. Сначала определите и отправьте правила синонимов. Во вторых, настройте поля для их использования. Описание этого процесса вы можете найти в `UploadSynonyms` и `EnableSynonymsInHotelsIndex`.

1. Добавьте сопоставление синонимов в свою службу поиска. С помощью `UploadSynonyms` мы определим четыре правила в сопоставлении синонимов desc-synonymmap и добавим его в службу.

   ```csharp
   private static void UploadSynonyms(SearchIndexClient indexClient)
   {
      var synonymMap = new SynonymMap("desc-synonymmap", "hotel, motel\ninternet,wifi\nfive star=>luxury\neconomy,inexpensive=>budget");

      indexClient.CreateOrUpdateSynonymMap(synonymMap);
   }
   ```

1. Настройте поля, поддерживающие поиск, чтобы использовать сопоставление синонимов в определении индекса. С помощью `AddSynonymMapsToFields` мы можем включить поиск синонимов для двух полей — `category` и `tags`. Для этого необходимо задать свойство `SynonymMapNames` для имени добавленного сопоставления синонимов.

   ```csharp
   private static SearchIndex AddSynonymMapsToFields(SearchIndex index)
   {
      index.Fields.First(f => f.Name == "category").SynonymMapNames.Add("desc-synonymmap");
      index.Fields.First(f => f.Name == "tags").SynonymMapNames.Add("desc-synonymmap");
      return index;
   }
   ```

   Когда вы добавите сопоставление синонимов, необходимость в перестроении индексов отпадет. Чтобы использовать новое сопоставление синонимов, вы можете добавить это сопоставление в свою службу, а затем изменить существующие определения полей в любом индексе. Добавление новых атрибутов никак не скажется на доступности индексов. То же относится и к отключению поиска синонимов для поля. Вы можете просто задать для пустого списка свойство `SynonymMapNames`.

   ```csharp
   index.Fields.First(f => f.Name == "category").SynonymMapNames.Add("desc-synonymmap");
   ```

## <a name="after-queries"></a>Запросы "после"

После добавления сопоставления синонимов и обновления индекса, необходимых для использования этого сопоставления, мы получаем следующие выходные данные, вызвав функцию `RunQueriesWithNonExistentTermsInIndex` второй раз:

```dos
Search the entire index for the phrase "five star":

Name: Fancy Stay        Category: Luxury        Tags: [pool, view, wifi, concierge]

Search the entire index for the term 'internet':

Name: Fancy Stay        Category: Luxury        Tags: [pool, view, wifi, concierge]

Search the entire index for the terms 'economy' AND 'hotel':

Name: Roach Motel       Category: Budget        Tags: [motel, budget]
```

Первый запрос находит документ из правила `five star=>luxury`. Для второго запроса поиск расширяется с помощью термина `internet,wifi`, а для третьего при поиске соответствующих документов используются оба термина запросов — `hotel, motel` и `economy,inexpensive=>budget`.

Добавление синонимов полностью изменяет возможности поиска. В этом примере исходным запросом не удалось вернуть значимые результаты, даже если документы в нашем индексе оказались релевантными. Включив поиск синонимов, мы можем расширить индекс, чтобы включить распространенные термины, не изменяя при этом базовые данные в индексе.

## <a name="clean-up-resources"></a>Очистка ресурсов

Самый быстрый способ очистки после примера заключается в удалении группы ресурсов, содержащей службу Когнитивный поиск Azure. Теперь можно удалить группу ресурсов вместе со всем ее содержимым без возможности восстановления. На портале имя группы ресурсов находится на странице "Обзор" службы "Когнитивный поиск Azure".

## <a name="next-steps"></a>Дальнейшие действия

В этом примере демонстрируется функция синонимов в коде C# для создания и публикации правил сопоставления, а затем вызывается сопоставление синонимов для запроса. См. дополнительные сведения о [пакете SDK для .NET ](/dotnet/api/overview/azure/search.documents-readme) и [REST API](/rest/api/searchservice/).

> [!div class="nextstepaction"]
> [Использование синонимов в Azure Когнитивный поиск](search-synonyms.md)