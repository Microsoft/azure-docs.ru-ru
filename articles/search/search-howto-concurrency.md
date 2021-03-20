---
title: Управление параллельными операциями записи в ресурсы
titleSuffix: Azure Cognitive Search
description: Используйте оптимистичный параллелизм, чтобы избежать конфликтов середины воздуха при обновлениях или удалениях в Azure Когнитивный поиск индексы, индексаторы и источники данных.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/26/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 52260facc1a458a3ef18b067e6d52e189f5d329c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98881760"
---
# <a name="how-to-manage-concurrency-in-azure-cognitive-search"></a>Управление параллелизмом в Azure Когнитивный поиск

При управлении ресурсами Когнитивный поиск Azure, такими как индексы и источники данных, важно безопасно обновлять ресурсы, особенно если доступ к ресурсам осуществляется параллельно разными компонентами приложения. Если два клиента одновременно обновляют ресурс, не координируя свои действия, то могут возникать состояния гонки. Чтобы избежать этого, Azure Когнитивный поиск предлагает *модель оптимистичного параллелизма*. При этом в отношении ресурса нет никаких блокировок. Вместо этого существует ETag для каждого ресурса, который определяет версию ресурса, чтобы можно было сформулировать запросы, которые позволят избежать случайной перезаписи.

> [!Tip]
> Концептуальный код в [примере решения C#](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetETagsExplainer) объясняет, как работает управление параллелизмом в Azure когнитивный Поиск. Код создает условия, которые вызывают управление параллелизмом. Чтение [фрагмента кода, приведенного ниже](#samplecode) , может быть достаточно для большинства разработчиков, но если вы хотите его запустить, измените appsettings.jsв, чтобы добавить имя службы и ключ API администратора. URL-адрес службы — `http://myservice.search.windows.net`, а имя службы — `myservice`.

## <a name="how-it-works"></a>Принцип работы

Оптимистическая блокировка реализуется с помощью проверки условий доступа в вызовах API для записи в индексы, индексаторы, источники данных, навыков и ресурсы synonymMap.

Все ресурсы имеют [*тег сущности (ETag)*](https://en.wikipedia.org/wiki/HTTP_ETag), который предоставляет сведения о версии объекта. Если сначала проверить ETag, то можно избежать параллельных обновлений в стандартном рабочем процессе (получение, локальное изменение, обновление), убедившись, что ETag ресурса соответствует локальной копии.

+ Интерфейс REST API использует [ETag](/rest/api/searchservice/common-http-request-and-response-headers-used-in-azure-search) в заголовке запроса.

+ Пакет SDK для .NET задает ETag с помощью объекта accessCondition, устанавливая в ресурсе [заголовок If-Match | If-None-Match](/rest/api/searchservice/common-http-request-and-response-headers-used-in-azure-search). Объекты, использующие Теги ETag, такие как [SynonymMap. ETag](/dotnet/api/azure.search.documents.indexes.models.synonymmap.etag) и [сеарчиндекс. ETag](/dotnet/api/azure.search.documents.indexes.models.searchindex.etag), имеют объект accessCondition.

Каждый раз при обновлении ресурса его ETag изменяется автоматически. При реализации управления параллелизмом все, что необходимо сделать, это поместить предварительное условие в запрос на обновление. Этим условием должно быть требование, чтобы удаленный ресурс имел такой же ETag, как и копия ресурса, измененного вами на клиенте. Если параллельный процесс уже изменил удаленный ресурс, то ETag не будет соответствовать предварительному условию и, следовательно, запрос завершится ошибкой HTTP 412. Если используется пакет SDK для .NET, то это объявляется в качестве `CloudException`, где метод расширения `IsAccessConditionFailed()` возвращает значение true.

> [!Note]
> Для параллелизма существует только один механизм. Он всегда используется независимо от того, какой интерфейс API задействован для обновления ресурсов.

<a name="samplecode"></a>
## <a name="use-cases-and-sample-code"></a>Варианты использования и пример кода

В следующем коде показано, как проверяется условие accessCondition для операций обновления ключа:

+ Сбой обновления, если ресурс больше не существует
+ Сбой обновления, если версия ресурса изменяется

### <a name="sample-code-from-dotnetetagsexplainer-program"></a>Пример кода из [программы DotNetETagsExplainer](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetETagsExplainer)

```csharp
class Program
{
    // This sample shows how ETags work by performing conditional updates and deletes
    // on an Azure Cognitive Search index.
    static void Main(string[] args)
    {
        IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
        IConfigurationRoot configuration = builder.Build();

        SearchServiceClient serviceClient = CreateSearchServiceClient(configuration);

        Console.WriteLine("Deleting index...\n");
        DeleteTestIndexIfExists(serviceClient);

        // Every top-level resource in Azure Cognitive Search has an associated ETag that keeps track of which version
        // of the resource you're working on. When you first create a resource such as an index, its ETag is
        // empty.
        Index index = DefineTestIndex();
        Console.WriteLine(
            $"Test index hasn't been created yet, so its ETag should be blank. ETag: '{index.ETag}'");

        // Once the resource exists in Azure Cognitive Search, its ETag will be populated. Make sure to use the object
        // returned by the SearchServiceClient! Otherwise, you will still have the old object with the
        // blank ETag.
        Console.WriteLine("Creating index...\n");
        index = serviceClient.Indexes.Create(index);
        
        Console.WriteLine($"Test index created; Its ETag should be populated. ETag: '{index.ETag}'");

        // ETags let you do some useful things you couldn't do otherwise. For example, by using an If-Match
        // condition, we can update an index using CreateOrUpdate and be guaranteed that the update will only
        // succeed if the index already exists.
        index.Fields.Add(new Field("name", AnalyzerName.EnMicrosoft));
        index =
            serviceClient.Indexes.CreateOrUpdate(
                index,
                accessCondition: AccessCondition.GenerateIfExistsCondition());

        Console.WriteLine(
            $"Test index updated; Its ETag should have changed since it was created. ETag: '{index.ETag}'");

        // More importantly, ETags protect you from concurrent updates to the same resource. If another
        // client tries to update the resource, it will fail as long as all clients are using the right
        // access conditions.
        Index indexForClient1 = index;
        Index indexForClient2 = serviceClient.Indexes.Get("test");

        Console.WriteLine("Simulating concurrent update. To start, both clients see the same ETag.");
        Console.WriteLine($"Client 1 ETag: '{indexForClient1.ETag}' Client 2 ETag: '{indexForClient2.ETag}'");

        // Client 1 successfully updates the index.
        indexForClient1.Fields.Add(new Field("a", DataType.Int32));
        indexForClient1 =
            serviceClient.Indexes.CreateOrUpdate(
                indexForClient1,
                accessCondition: AccessCondition.IfNotChanged(indexForClient1));

        Console.WriteLine($"Test index updated by client 1; ETag: '{indexForClient1.ETag}'");

        // Client 2 tries to update the index, but fails, thanks to the ETag check.
        try
        {
            indexForClient2.Fields.Add(new Field("b", DataType.Boolean));
            serviceClient.Indexes.CreateOrUpdate(
                indexForClient2,
                accessCondition: AccessCondition.IfNotChanged(indexForClient2));

            Console.WriteLine("Whoops; This shouldn't happen");
            Environment.Exit(1);
        }
        catch (CloudException e) when (e.IsAccessConditionFailed())
        {
            Console.WriteLine("Client 2 failed to update the index, as expected.");
        }

        // You can also use access conditions with Delete operations. For example, you can implement an
        // atomic version of the DeleteTestIndexIfExists method from this sample like this:
        Console.WriteLine("Deleting index...\n");
        serviceClient.Indexes.Delete("test", accessCondition: AccessCondition.GenerateIfExistsCondition());

        // This is slightly better than using the Exists method since it makes only one round trip to
        // Azure Cognitive Search instead of potentially two. It also avoids an extra Delete request in cases where
        // the resource is deleted concurrently, but this doesn't matter much since resource deletion in
        // Azure Cognitive Search is idempotent.

        // And we're done! Bye!
        Console.WriteLine("Complete.  Press any key to end application...\n");
        Console.ReadKey();
    }

    private static SearchServiceClient CreateSearchServiceClient(IConfigurationRoot configuration)
    {
        string searchServiceName = configuration["SearchServiceName"];
        string adminApiKey = configuration["SearchServiceAdminApiKey"];

        SearchServiceClient serviceClient =
            new SearchServiceClient(searchServiceName, new SearchCredentials(adminApiKey));
        return serviceClient;
    }

    private static void DeleteTestIndexIfExists(SearchServiceClient serviceClient)
    {
        if (serviceClient.Indexes.Exists("test"))
        {
            serviceClient.Indexes.Delete("test");
        }
    }

    private static Index DefineTestIndex() =>
        new Index()
        {
            Name = "test",
            Fields = new[] { new Field("id", DataType.String) { IsKey = true } }
        };
    }
}
```

## <a name="design-pattern"></a>Конструктивный шаблон

Конструктивный шаблон для реализации оптимистической блокировки должен включать в себя цикл повторных попыток проверки условия доступа, тест для условия доступа и, при необходимости, должен получать обновленный ресурс перед попыткой повторного применения изменений.

В этом фрагменте кода показано добавление synonymMap в индекс, который уже существует. Этот код из примера на [C# синонима для Azure когнитивный Поиск](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/v10/DotNetHowToSynonyms).

Этот фрагмент получает индекс "hotels", проверяет версию объекта при операции обновления, порождает исключение, если условие не выполняется, а затем совершает повторную попытку операции (до трех раз), начиная с извлечения индекса с сервера, чтобы получить последнюю версию.

```csharp
private static void EnableSynonymsInHotelsIndexSafely(SearchServiceClient serviceClient)
{
    int MaxNumTries = 3;

    for (int i = 0; i < MaxNumTries; ++i)
    {
        try
        {
            Index index = serviceClient.Indexes.Get("hotels");
            index = AddSynonymMapsToFields(index);

            // The IfNotChanged condition ensures that the index is updated only if the ETags match.
            serviceClient.Indexes.CreateOrUpdate(index, accessCondition: AccessCondition.IfNotChanged(index));

            Console.WriteLine("Updated the index successfully.\n");
            break;
        }
        catch (CloudException e) when (e.IsAccessConditionFailed())
        {
            Console.WriteLine($"Index update failed : {e.Message}. Attempt({i}/{MaxNumTries}).\n");
        }
    }
}

private static Index AddSynonymMapsToFields(Index index)
{
    index.Fields.First(f => f.Name == "category").SynonymMaps = new[] { "desc-synonymmap" };
    index.Fields.First(f => f.Name == "tags").SynonymMaps = new[] { "desc-synonymmap" };
    return index;
}
```

## <a name="next-steps"></a>Дальнейшие действия

Попробуйте изменить другие примеры, чтобы применить теги ETag или объекты AccessCondition.

+ [выполните поиск-DotNet-Приступая к работе на GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started). Этот репозиторий включает проект "DotNetEtagsExplainer".

+ [Azure-Search-DotNet-Samples на GitHub](https://github.com/Azure-Samples/azure-search-dotnet-samples) содержит дополнительные примеры на C#.

## <a name="see-also"></a>См. также раздел

+ [Common HTTP request and response headers used in Azure Search](/rest/api/searchservice/common-http-request-and-response-headers-used-in-azure-search) (Стандартные заголовки HTTP-запросов и ответов, используемые в Поиске Azure)
+ [Коды состояния HTTP](/rest/api/searchservice/http-status-codes)
+ [Операции с индексами (REST API)](/rest/api/searchservice/index-operations)