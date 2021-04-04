---
title: Руководство по оптимизации индексирования в C# с помощью API Push
titleSuffix: Azure Cognitive Search
description: Узнайте, как эффективно индексировать данные с помощью API Push в Когнитивном поиске Azure. В этом учебнике и примере кода используется язык C#.
manager: luisca
author: dereklegenzoff
ms.author: delegenz
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 1/29/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: f97a99bf2d055805ee665ab51aff8cff12dc5a69
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99094198"
---
# <a name="tutorial-optimize-indexing-with-the-push-api"></a>Руководство по Оптимизация индексирования с помощью API Push

Когнитивный поиск Azure поддерживает [два основных подхода](search-what-is-data-import.md) к импорту данных в индекс поиска: *отправка данных* в индекс программными средствами и указание [индексатору Когнитивного поиска Azure](search-indexer-overview.md) на поддерживаемый источник данных для *извлечения* данных.

В этом руководстве описано, как эффективно индексировать данные [методом отправки](search-what-is-data-import.md#pushing-data-to-an-index) с применением запросов пакетной обработки и стратегии попыток повторов с экспоненциальной задержкой. Вы можете [скачать и запустить пример приложения](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/optimize-data-indexing). В этой статье описываются ключевые аспекты приложения и факторы, которые нужно учитывать при индексировании данных.

В этом учебнике используется C# и пакет [SDK для .NET](/dotnet/api/overview/azure/search) для выполнения следующих задач:

> [!div class="checklist"]
> * Создание индекса
> * Попробуйте разные размеры пакетов, чтобы выбрать наиболее эффективный вариант.
> * Реализуйте асинхронное индексирование пакетов.
> * Используйте несколько потоков для увеличения скорости индексирования.
> * Реализуйте стратегию повторов с экспоненциальной задержкой для повторных попыток при неудачной обработке документов:

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Для выполнения инструкций из этого руководства потребуются перечисленные ниже службы и инструменты.

+ [Visual Studio](https://visualstudio.microsoft.com/downloads/) (любой выпуск). Пример кода и инструкции были протестированы с помощью бесплатного выпуска Community Edition.

+ [Создайте службу "Когнитивный поиск Azure"](search-create-service-portal.md) или [найдите имеющуюся службу](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) в рамках текущей подписки.

<a name="get-service-info"></a>

## <a name="download-files"></a>Загрузка файлов

Исходный код примера из этого руководства расположен в папке [optimzize-data-indexing/v11](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/optimize-data-indexing/v11) репозитория [Azure-Samples/azure-search-dotnet-samples](https://github.com/Azure-Samples/azure-search-dotnet-samples) на сайте GitHub.

## <a name="key-considerations"></a>Основные рекомендации

При отправке данных в индекс важно учитывать несколько аспектов, влияющих на скорость индексирования. Дополнительные сведения об этих факторах см. в статье [Индексирование больших наборов данных](search-howto-large-index.md).

Вот шесть основных из них:

+ **Уровень служб, количество секций и (или) реплик**. Добавление секций и увеличение уровня повысит скорость индексирования.
+ **Схема индекса**. Добавление полей и дополнительных свойств в них (*searchable*, *facetable*, *filterable* и т. п.) снижает скорость индексирования.
+ **Размер пакета**. Оптимальный размер пакета будет разным в зависимости от схемы индекса и набора данных.
+ **Число потоков и рабочих ролей**. Один поток никогда не сможет добиться максимальной скорости индексирования.
+ **Стратегия повторов**. Для оптимизации индексирования следует использовать стратегию повторных попыток с экспоненциальной задержкой.
+ **Скорость передачи данных по сети**. Она может иногда являться ограничивающим фактором. Чтобы увеличить скорость передачи данных, индексируйте данные из среды Azure.


## <a name="1---create-azure-cognitive-search-service"></a>1\. Создание экземпляра службы "Когнитивный поиск Azure"

Чтобы выполнить инструкции из этого руководства, вам потребуется служба "Когнитивный поиск Azure", экземпляр которой вы можете [создать на портале Azure](search-create-service-portal.md). Мы рекомендуем использовать тот же уровень, который планируется применять в рабочей среде, чтобы точнее оценить скорость индексирования и оптимизировать ее.

### <a name="get-an-admin-api-key-and-url-for-azure-cognitive-search"></a>Получение ключа API и URL-адреса конечной точки для администрирования Когнитивного поиска Azure

Для вызова API требуется URL-адрес службы и ключ доступа. Служба поиска создана с обоими элементами, поэтому если вы добавили службу "Когнитивный поиск Azure" в подписку, выполните следующие действия для получения необходимых сведений:

1. [Войдите на портал Azure](https://portal.azure.com/) и на странице **обзора** службы поиска получите URL-адрес. Пример конечной точки может выглядеть так: `https://mydemo.search.windows.net`.

1. В разделе **Параметры** > **Ключи** получите ключ администратора, чтобы обрести полные права на службу. Существуют два взаимозаменяемых ключа администратора, предназначенных для обеспечения непрерывности бизнес-процессов на случай, если вам потребуется сменить один из них. Вы можете использовать первичный или вторичный ключ для выполнения запросов на добавление, изменение и удаление объектов.

   ![Получение конечной точки HTTP и ключа доступа](media/search-get-started-rest/get-url-key.png "Получение конечной точки HTTP и ключа доступа")

## <a name="2---set-up-your-environment"></a>2\. Настройка среды

1. Запустите Visual Studio и откройте файл **OptimizeDataIndexing.sln**.
1. В обозревателе решений откройте файл **appsettings.json**, чтобы предоставить сведения о подключении.

```json
{
  "SearchServiceUri": "https://{service-name}.search.windows.net",
  "SearchServiceAdminApiKey": "",
  "SearchIndexName": "optimize-indexing"
}
```

## <a name="3---explore-the-code"></a>3\. Изучение кода

Когда вы завершите обновление файла *appsettings.json*, пример программы в файле **OptimizeDataIndexing.sln** будет готов к компиляции и запуску.

Этот код основан на [этом кратком руководстве для C#](search-get-started-dotnet.md). В нем вы найдете более подробные сведения об основах работы с пакетом SDK для .NET.

Это простое консольное приложение для C#/.NET выполняет следующие задачи:

+ создает индекс на основе структуры данных в классе C# Hotel (который также ссылается на класс Address);
+ выполняет тесты с разными размерами пакетов, чтобы выбрать наиболее эффективный вариант;
+ индексирует данные асинхронно;
    + использует несколько потоков для увеличения скорости индексирования;
    + применяет стратегию повторов с экспоненциальной задержкой, чтобы обрабатывать неудачные попытки.

 Перед выполнением этой программы тщательно изучите ее код и определения индекса, созданные для этого примера. Соответствующий код находится в нескольких файлах:

  + **Hotel.cs** и **Address.cs** содержат схему, которая определяет индекс;
  + **DataGenerator.cs** содержит простой класс, который облегчает создание данных об отелях в больших объемах;
  + **ExponentialBackoff.cs** содержит код для оптимизации процесса индексирования, который описан ниже;
  + **Program.cs** содержит функции, которые создают и удаляют индекс службы "Когнитивный поиск Azure", индексируют пакеты данных и выполняют тесты с разными размерами пакетов.

### <a name="creating-the-index"></a>Создание индекса

Этот пример программы использует пакет SDK для .NET, чтобы определить и создать индекс службы "Когнитивный поиск Azure". С помощью `FieldBuilder` из класса C# модели данных программа создает структуру индекса.

Модель данных определяется в классе Hotel, который содержит также ссылки на класс Address. FieldBuilder углубляется в структуру определений нескольких классов, создавая на их основе сложную структуру данных для индекса. Теги метаданных применяются для определения атрибутов каждого поля, например сведений о поддержке поиска или сортировки.

Следующие фрагменты кода из файла **Hotel.cs** демонстрируют, как можно определить одно поле и ссылку на другой класс модели данных.

```csharp
. . .
[SearchableField(IsSortable = true)]
public string HotelName { get; set; }
. . .
public Address Address { get; set; }
. . .
```

В файле **Program.cs** для индекса определяется имя и коллекция полей, созданных методом `FieldBuilder.Build(typeof(Hotel))`, а затем этот индекс создается, как показано ниже:

```csharp
private static async Task CreateIndexAsync(string indexName, SearchIndexClient indexClient)
{
    // Create a new search index structure that matches the properties of the Hotel class.
    // The Address class is referenced from the Hotel class. The FieldBuilder
    // will enumerate these to create a complex data structure for the index.
    FieldBuilder builder = new FieldBuilder();
    var definition = new SearchIndex(indexName, builder.Build(typeof(Hotel)));

    await indexClient.CreateIndexAsync(definition);
}
```

### <a name="generating-data"></a>Создание данных

В файле **DataGenerator.cs** реализован простой класс для создания тестовых данных. Единственным назначением этого класса является быстрое создание большого количества документов с уникальным идентификатором для последующего индексирования.

Чтобы получить список из 100 000 гостиниц с уникальными идентификаторами, выполните следующие строки кода:

```csharp
long numDocuments = 100000;
DataGenerator dg = new DataGenerator();
List<Hotel> hotels = dg.GetHotels(numDocuments, "large");
```

Этот пример поддерживает два размера гостиниц для тестирования: **small** (маленькие) и **large** (большие).

Схема индекса может оказать значительное влияние на скорость индексирования. Поэтому есть смысл преобразовать этот класс так, чтобы созданные данные соответствовали предполагаемой схеме индекса, который вы будете создавать после выполнения инструкций из этого руководства.

## <a name="4---test-batch-sizes"></a>4\. Проверка размеров пакетов

Для загрузки одного или нескольких документов в индекс служба "Когнитивный поиск Azure" поддерживает следующие интерфейсы API:

+ [добавление, обновление и удаление документов (REST API)](/rest/api/searchservice/AddUpdate-or-Delete-Documents);
+ [класс IndexDocumentsAction](/dotnet/api/azure.search.documents.models.indexdocumentsaction) или [класс IndexDocumentsBatch](/dotnet/api/azure.search.documents.models.indexdocumentsbatch).

Пакетное индексирование документов значительно повысит производительность индексирования. Каждый пакет может содержать до 1000 документов или до 16 МБ данных.

Определение оптимального размера пакета для данных является ключевой задачей при оптимизации скорости индексирования. Существует два основных фактора, которые влияют на оптимальный размер пакета:

+ схема индекса;
+ размер данных.

Поскольку оптимальный размер пакета зависит от индекса и данных, лучше всего всегда проверять пакеты разных размеров, чтобы найти самый быстрый вариант индексирования для конкретного сценария.

В следующем примере функции показан простой подход к выполнению тестирования с пакетами разных размеров.

```csharp
public static async Task TestBatchSizesAsync(SearchClient searchClient, int min = 100, int max = 1000, int step = 100, int numTries = 3)
{
    DataGenerator dg = new DataGenerator();

    Console.WriteLine("Batch Size \t Size in MB \t MB / Doc \t Time (ms) \t MB / Second");
    for (int numDocs = min; numDocs <= max; numDocs += step)
    {
        List<TimeSpan> durations = new List<TimeSpan>();
        double sizeInMb = 0.0;
        for (int x = 0; x < numTries; x++)
        {
            List<Hotel> hotels = dg.GetHotels(numDocs, "large");

            DateTime startTime = DateTime.Now;
            await UploadDocumentsAsync(searchClient, hotels).ConfigureAwait(false);
            DateTime endTime = DateTime.Now;
            durations.Add(endTime - startTime);

            sizeInMb = EstimateObjectSize(hotels);
        }

        var avgDuration = durations.Average(timeSpan => timeSpan.TotalMilliseconds);
        var avgDurationInSeconds = avgDuration / 1000;
        var mbPerSecond = sizeInMb / avgDurationInSeconds;

        Console.WriteLine("{0} \t\t {1} \t\t {2} \t\t {3} \t {4}", numDocs, Math.Round(sizeInMb, 3), Math.Round(sizeInMb / numDocs, 3), Math.Round(avgDuration, 3), Math.Round(mbPerSecond, 3));

        // Pausing 2 seconds to let the search service catch its breath
        Thread.Sleep(2000);
    }

    Console.WriteLine();
}
```

Обычно не все документы имеют одинаковый размер, как в нашем примере, поэтому нам нужно оценивать размер данных, отправляемых в службу поиска. Для этого используется приведенная ниже функция, которая сначала преобразует объект в формат JSON, а затем определяет его размер в байтах. Этот метод позволяет определить наиболее эффективные размеры пакетов с точки зрения скорости индексирования (МБ/с).

```csharp
// Returns size of object in MB
public static double EstimateObjectSize(object data)
{
    // converting object to byte[] to determine the size of the data
    BinaryFormatter bf = new BinaryFormatter();
    MemoryStream ms = new MemoryStream();
    byte[] Array;

    // converting data to json for more accurate sizing
    var json = JsonSerializer.Serialize(data);
    bf.Serialize(ms, json);
    Array = ms.ToArray();

    // converting from bytes to megabytes
    double sizeInMb = (double)Array.Length / 1000000;

    return sizeInMb;
}
```

Функция принимает обязательный параметр `SearchClient` и количество попыток тестирования для каждого размера пакета. Так как время индексирования для одного пакета может быть разным, по умолчанию каждый пакет обрабатывается три раза для повышения статистической достоверности результатов.

```csharp
await TestBatchSizesAsync(searchClient, numTries: 3);
```

При выполнении этой функции вы увидите в консоли примерно такие выходные данные:

   ![Выходные данные функции тестирования размера пакетов](media/tutorial-optimize-data-indexing/test-batch-sizes.png "Выходные данные функции тестирования размера пакетов")

Найдите наиболее эффективный размер пакета и примените его на следующем шаге руководства. Значение скорости в МБ/с может оказаться равным для нескольких разных размеров пакетов.

## <a name="5---index-data"></a>5\. Индексирование данных

Теперь, когда мы определили нужный размер пакета, можно переходить к индексированию данных. Для эффективного индексирования в этом примере применяется следующее:

+ несколько потоков и рабочих ролей;
+ стратегию повторов с экспоненциальной задержкой.

### <a name="use-multiple-threadsworkers"></a>Использование нескольких потоков и рабочих ролей

Чтобы добиться максимально возможных скоростей индексирования в Когнитивном поиске Azure, скорее всего потребуется несколько параллельных потоков, чтобы одновременно отправлять запросы пакетной индексации в службу.  

Некоторые из упомянутых выше факторов влияют на оптимальное число потоков. Вы можете изменить этот пример так, чтобы проверить работу с разным количеством потоков и выбрать оптимальный вариант для вашего сценария. Но обычно любое количество параллельных потоков позволяет добиться значительного повышения эффективности.

По мере повышения числа запросов, передаваемых в службу поиска, могут возвращаться [коды состояния HTTP](/rest/api/searchservice/http-status-codes) с ошибками, свидетельствующие о неудачном выполнении запроса. При индексировании часто используются следующие два кода состояния HTTP:

+ **503 Service Unavailable** (Служба недоступна). Эта ошибка означает, что система перегружена и запрос не может быть обработан сейчас.
+ **207 Multi-Status** (Множественное состояние). Эта ошибка означает, что некоторые документы обработаны успешно, но хотя бы один — со сбоем.

### <a name="implement-an-exponential-backoff-retry-strategy"></a>Реализация стратегии повторов с экспоненциальной задержкой

В случае сбоя следует повторить выполнение запроса, используя [стратегию повторов с экспоненциальной задержкой](/dotnet/architecture/microservices/implement-resilient-applications/implement-retries-exponential-backoff).

Пакет SDK .NET для Когнитивного поиска Azure реализует автоматический повтор попыток при получении ошибок 503 и других сбоев, но для повторной обработки ошибок 207 вам потребуется собственная логика повторов. Также для реализации стратегии повторных попыток можно использовать средства с открытым кодом, например [Polly](https://github.com/App-vNext/Polly).

В нашем примере есть собственная реализация стратегии повторов с экспоненциальной задержкой. Чтобы применить эту стратегию, мы для начала определяем несколько переменных, в том числе `maxRetryAttempts` и начальное значение `delay` для неудачно выполненного запроса:

```csharp
// Create batch of documents for indexing
var batch = IndexDocumentsBatch.Upload(hotels);

// Create an object to hold the result
IndexDocumentsResult result = null;

// Define parameters for exponential backoff
int attempts = 0;
TimeSpan delay = delay = TimeSpan.FromSeconds(2);
int maxRetryAttempts = 5;
```

Результаты операции индексирования хранятся в переменной `IndexDocumentResult result`. Эта переменная важна, так как она позволяет проверить, не завершилась ли обработка какого-либо документа в пакете сбоем, как показано ниже. В случае частичного сбоя создается новый пакет на основе идентификаторов невыполненных документов.

Следует также перехватывать исключения `RequestFailedException`, так как они указывают на полностью неудачное выполнение запроса и необходимость выполнить запрос повторно.

```csharp
// Implement exponential backoff
do
{
    try
    {
        attempts++;
        result = await searchClient.IndexDocumentsAsync(batch).ConfigureAwait(false);

        var failedDocuments = result.Results.Where(r => r.Succeeded != true).ToList();

        // handle partial failure
        if (failedDocuments.Count > 0)
        {
            if (attempts == maxRetryAttempts)
            {
                Console.WriteLine("[MAX RETRIES HIT] - Giving up on the batch starting at {0}", id);
                break;
            }
            else
            {
                Console.WriteLine("[Batch starting at doc {0} had partial failure]", id);
                Console.WriteLine("[Retrying {0} failed documents] \n", failedDocuments.Count);

                // creating a batch of failed documents to retry
                var failedDocumentKeys = failedDocuments.Select(doc => doc.Key).ToList();
                hotels = hotels.Where(h => failedDocumentKeys.Contains(h.HotelId)).ToList();
                batch = IndexDocumentsBatch.Upload(hotels);

                Task.Delay(delay).Wait();
                delay = delay * 2;
                continue;
            }
        }

        return result;
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine("[Batch starting at doc {0} failed]", id);
        Console.WriteLine("[Retrying entire batch] \n");

        if (attempts == maxRetryAttempts)
        {
            Console.WriteLine("[MAX RETRIES HIT] - Giving up on the batch starting at {0}", id);
            break;
        }

        Task.Delay(delay).Wait();
        delay = delay * 2;
    }
} while (true);
```

А здесь мы поместим код реализации экспоненциальной задержки в функцию, чтобы его было проще вызывать.

Кроме того, создается еще одна функция для управления активными потоками. Для простоты мы не приводим здесь эту функцию, но вы можете найти ее в файле [ExponentialBackoff.cs](https://github.com/Azure-Samples/azure-search-dotnet-samples/blob/master/optimize-data-indexing/v11/OptimizeDataIndexing/ExponentialBackoff.cs). Эту функцию можно вызвать с помощью следующей команды, где `hotels` — это данные для передачи, `1000` означает размер пакета, а `8` — количество параллельных потоков:

```csharp
await ExponentialBackoff.IndexData(indexClient, hotels, 1000, 8);
```

При выполнении этой функции вы увидите примерно такие выходные данные:

![Выходные данные функции индексирования данных](media/tutorial-optimize-data-indexing/index-data-start.png "Выходные данные функции индексирования данных")

Если обработка пакета документов завершается ошибкой, выводится соответствующее сообщение с информацией о сбое и о применении повторной попытки:

```
[Batch starting at doc 6000 had partial failure]
[Retrying 560 failed documents]
```

Когда функция завершит выполнение, вы можете убедиться, что все документы успешно добавлены в индекс.

## <a name="6---explore-index"></a>6\. Изучение индекса

Вы можете изучить заполненный индекс поиска после выполнения программы с помощью компонента [**Обозреватель поиска**](search-explorer.md) на портале или программным способом.

### <a name="programatically"></a>Программный способ

Есть два основных метода проверки количества документов в индексе: API [Count Documents](/rest/api/searchservice/count-documents) и API [Get Index Statistics](/rest/api/searchservice/get-index-statistics). Оба метода могут потребовать дополнительного времени на обновление, поэтому не следует беспокоиться, если при первой попытке число документов окажется ниже ожидаемого.

#### <a name="count-documents"></a>Подсчет документов

Операция Count Documents возвращает количество документов в индексе поиска.

```csharp
long indexDocCount = await searchClient.GetDocumentCountAsync();
```

#### <a name="get-index-statistics"></a>Получение статистических данных индекса

Операция Get Index Statistics возвращает количество документов в текущем индексе, а также сведения об использовании хранилища. Статистика индекса обновляется дольше, чем количество документов.

```csharp
var indexStats = await indexClient.GetIndexStatisticsAsync(indexName);
```

### <a name="azure-portal"></a>Портал Azure

На портале Azure откройте страницу **Обзор** для службы поиска и найдите в списке **Индексы** созданный индекс **optimize-indexing**.

  ![Список индексов службы Когнитивный поиск Azure](media/tutorial-optimize-data-indexing/portal-output.png "Список индексов службы Когнитивный поиск Azure")

Значения *Число документов* и *Размер хранилища* основаны на API [Get Index Statistics](/rest/api/searchservice/get-index-statistics) и на их обновление может потребоваться несколько минут.

## <a name="reset-and-rerun"></a>Сброс и повторный запуск

На ранних экспериментальных этапах разработки самый практичный подход к итерации схемы — удалить все объекты из службы "Когнитивный поиск Azure" и восстановить их с помощью кода. Имена ресурсов являются уникальными. Удаление объекта позволяет воссоздать его с использованием того же имени.

Пример кода для этого руководства проверяет имеющиеся индексы и удаляет их, чтобы вы могли повторно выполнить этот код.

Также для удаления индексов можно использовать портал.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы работаете в своей подписке, после завершения проекта целесообразно удалить созданные ресурсы, которые вам больше не потребуются. Работающие ресурсы могут означать лишние затраты. Можно удалить отдельные ресурсы или удалить группу ресурсов, что позволит удалить весь набор ресурсов.

Просматривать ресурсы и управлять ими можно на портале с помощью ссылок **Все ресурсы** или **Группы ресурсов** на панели навигации слева.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы знакомы с основными понятиями, влияющими на эффективность приема данных, мы рассмотрим архитектуру запросов Lucene и принцип работы полнотекстового поиска в службе "Когнитивный поиск Azure".

> [!div class="nextstepaction"]
> [How full text search works in Azure Cognitive Search](search-lucene-query-architecture.md) (Как выполняется полнотекстовый поиск в Когнитивном поиске Azure)