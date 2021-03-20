---
title: Перенос приложения из Amazon DynamoDB в Azure Cosmos DB
description: Узнайте, как переносить приложения .NET из Amazon DynamoDB в Azure Cosmos DB
author: manishmsfte
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 04/29/2020
ms.author: mansha
ms.openlocfilehash: 9b4b5fca8017a906fa44b02edcf5f0bdcf6166b3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93334231"
---
# <a name="migrate-your-application-from-amazon-dynamodb-to-azure-cosmos-db"></a>Перенос приложения из Amazon DynamoDB в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB — это масштабируемая, глобально распределенная, полностью управляемая база данных. Она обеспечивает гарантированный доступ к данным с низкой задержкой. Дополнительные сведения об Azure Cosmos DB см. в [этой обзорной статье](introduction.md). В этой статье описывается, как перенести приложение .NET из DynamoDB в Azure Cosmos DB с минимальным изменением кода.

## <a name="conceptual-differences"></a>Концептуальные различия

Ниже указаны основные концептуальные различия между Azure Cosmos DB и DynamoDB.

|  DynamoDB | Azure Cosmos DB  |
|---|---|
|Неприменимо|  База данных |
|Таблица      |  Коллекция |
|  Item |  Документ |
|attribute|Поле|
|Вторичный индекс|Вторичный индекс|
|Первичный ключ — ключ секции|Ключ раздела|
|Первичный ключ — ключ сортировки| Не требуется |
|STREAM|Канал изменений|
|Единица вычислений для записи|Единица запроса (гибкая, может использоваться для операций как чтения, так и записи)|
|Единица вычислений для чтения    |Единица запроса (гибкая, может использоваться для операций как чтения, так и записи)|
|Глобальные таблицы| Не требуется. Вы можете напрямую выбрать регион при подготовке учетной записи Azure Cosmos (этот регион можно изменить позже)|

## <a name="structural-differences"></a>Структурные различия

У Azure Cosmos DB более простая структура JSON по сравнению с DynamoDB. Различия можно увидеть в примере ниже

**DynamoDB**.

Следующий объект JSON представляет формат данных в DynamoDB

```json
{
TableName: "Music",
KeySchema: [
{ 
  AttributeName: "Artist",
  KeyType: "HASH", //Partition key
},
{ 
  AttributeName: "SongTitle",
  KeyType: "RANGE" //Sort key
}
],
AttributeDefinitions: [
{ 
  AttributeName: "Artist",
  AttributeType: "S"
},
{ 
  AttributeName: "SongTitle",
  AttributeType: "S"
}
],
ProvisionedThroughput: {
  ReadCapacityUnits: 1,
  WriteCapacityUnits: 1
 }
}
 ```

**Azure Cosmos DB**.

Следующий объект JSON представляет формат данных в Azure Cosmos DB

```json
{
"Artist": "",
"SongTitle": "",
"AlbumTitle": "",
"Year": 9999,
"Price": 0.0,
"Genre": "",
"Tags": ""
}
```

## <a name="migrate-your-data"></a>Перенос данных

Перенести данные в Azure Cosmos DB можно различными способами. Подробнее см. в статье о [способах переноса локальных или облачных данных в Azure Cosmos DB](cosmosdb-migrationchoices.md).

## <a name="migrate-your-code"></a>Перенос кода

В этой статье рассматривается адаптация кода приложения для Azure Cosmos DB — важнейший аспект переноса базы данных. Чтобы сократить для вас время изучения этого вопроса, в последующих разделах фрагменты кода для Amazon DynamoDB будут сравниваться с эквивалентными фрагментами кода для Azure Cosmos DB.

Чтобы скачать исходный код, выполните клонирование следующего репозитория.

```bash
git clone https://github.com/Azure-Samples/DynamoDB-to-CosmosDB
```

### <a name="pre-requisites"></a>Предварительные требования

- .NET Framework 4.7.2.
- Visual Studio 2019
- Доступ к учетной записи API SQL для Azure Cosmos DB
- Локально установленный экземпляр Amazon DynamoDB
- Java 8
- Загружаемая версия Amazon DynamoDB должна быть запущена на порте 8000 (вы можете соответствующим образом изменить и настроить код).

### <a name="set-up-your-code"></a>Настройка кода

Добавьте в проект следующий пакет NuGet.

```bash
Install-Package Microsoft.Azure.Cosmos 
```

### <a name="establish-connection"></a>Установка подключения

**DynamoDB**.

В Amazon DynamoDB для подключения используется следующий код.

```csharp
    AmazonDynamoDBConfig addbConfig = new AmazonDynamoDBConfig() ;
        addbConfig.ServiceURL = "endpoint";
        try { aws_dynamodbclient = new AmazonDynamoDBClient( addbConfig ); }
```

**Azure Cosmos DB**.

Для подключения к Azure Cosmos DB измените код следующим образом.

```csharp
client_documentDB = new CosmosClient("your connectionstring from the Azure portal");
```

**Оптимизация подключения в Azure Cosmos DB**

В Azure Cosmos DB можно оптимизировать подключение с помощью следующих параметров.

* **ConnectionMode** — использование режима прямого подключения для подключения к узлам данных в службе Azure Cosmos DB. Используйте режим шлюза только для инициализации и кэширования логических адресов и выполняйте обновление в случае изменений. Подробнее см. в статье о [режимах подключения](sql-sdk-connection-modes.md).

* **ApplicationRegion** — этот параметр используется для задания предпочитаемого геореплицированного региона, который используется для взаимодействия с Azure Cosmos DB. Подробнее см. в статье о [глобальном распределении](distribute-data-globally.md).

* **ConsistencyLevel** — этот параметр используется для переопределения уровня согласованности по умолчанию. Подробнее см. в статье об [уровнях согласованности](consistency-levels.md).

* **BulkExecutionMode** — этот параметр используется для выполнения массовых операций путем установки для свойства *AllowBulkExecution* значения true. Подробнее см. в статье о [массовом импорте](tutorial-sql-api-dotnet-bulk-import.md).

   ```csharp
   client_cosmosDB = new CosmosClient(" Your connection string ",new CosmosClientOptions()
   { 
    ConnectionMode=ConnectionMode.Direct,
    ApplicationRegion=Regions.EastUS2,
    ConsistencyLevel=ConsistencyLevel.Session,
    AllowBulkExecution=true  
   });
   ```

### <a name="provision-the-container"></a>Подготовка контейнера

**DynamoDB**.

Чтобы сохранить данные в Amazon DynamoDB, сначала необходимо создать таблицу. В ходе этого процесса вам необходимо определить схему, тип ключа и атрибуты, как показано в следующем коде.

```csharp
// movies_key_schema
public static List<KeySchemaElement> movies_key_schema
  = new List<KeySchemaElement>
{
  new KeySchemaElement
  {
    AttributeName = partition_key_name,
    KeyType = "HASH"
  },
  new KeySchemaElement
  {
    AttributeName = sort_key_name,
    KeyType = "RANGE"
  }
};

// key names for the Movies table
public const string partition_key_name = "year";
public const string sort_key_name      = "title";
  public const int readUnits=1, writeUnits=1; 

    // movie_items_attributes
    public static List<AttributeDefinition> movie_items_attributes
  = new List<AttributeDefinition>
{
  new AttributeDefinition
  {
    AttributeName = partition_key_name,
    AttributeType = "N"
  },
  new AttributeDefinition
  {
    AttributeName = sort_key_name,
    AttributeType = "S"
  }

CreateTableRequest  request;
CreateTableResponse response;

// Build the 'CreateTableRequest' structure for the new table
request = new CreateTableRequest
{
  TableName             = table_name,
  AttributeDefinitions  = table_attributes,
  KeySchema             = table_key_schema,
  // Provisioned-throughput settings are always required,
  // although the local test version of DynamoDB ignores them.
  ProvisionedThroughput = new ProvisionedThroughput( readUnits, writeUnits );
};
```

**Azure Cosmos DB**.

В Amazon DynamoDB необходимо подготовить единицы вычислений для чтения и единицы вычислений для записи. В Azure Cosmos DB вы указываете пропускную способность в виде [единиц запросов (единиц запросов в секунду)](request-units.md), которые можно динамически использовать для любых операций. Данные организованы следующим образом: база данных --> контейнер --> элемент. Пропускную способность можно задать на уровне базы данных, на уровне сбора или на обоих этих уровнях.

Создание базы данных:

```csharp
await client_cosmosDB.CreateDatabaseIfNotExistsAsync(movies_table_name);
```

Создание контейнера:

```csharp
await cosmosDatabase.CreateContainerIfNotExistsAsync(new ContainerProperties() { PartitionKeyPath = "/" + partitionKey, Id = new_collection_name }, provisionedThroughput);
```

### <a name="load-the-data"></a>Загрузка данных

**DynamoDB**.

В приведенном ниже коде показано, как необходимо загружать данные в Amazon DynamoDB. Объект moviesArray содержит список документов JSON, по элементам которого необходимо последовательно пройти и загрузить каждый соответствующий документ JSON в Amazon DynamoDB.

```csharp
int n = moviesArray.Count;
for( int i = 0, j = 99; i < n; i++ )
    {
  try
  {
    string itemJson = moviesArray[i].ToString();
    Document doc = Document.FromJson(itemJson);
    Task putItem = moviesTable.PutItemAsync(doc);
    if( i >= j )
    {
      j++;
      Console.Write( "{0,5:#,##0}, ", j );
      if( j % 1000 == 0 )
        Console.Write( "\n " );
      j += 99;
    }
    await putItem;
```

**Azure Cosmos DB**.

В Azure Cosmos DB можно выбрать потоковую передачу и запись с помощью метода `moviesContainer.CreateItemStreamAsync()`. Однако в этом примере JSON будет десериализирован в тип *MovieModel*, чтобы продемонстрировать функцию приведения типов. Код является многопоточным. Благодаря этому будет использоваться распределенная архитектура Azure Cosmos DB и загрузка будет происходить быстрее.

```csharp
List<Task> concurrentTasks = new List<Task>();
for (int i = 0, j = 99; i < n; i++)
{
  try
  {
      MovieModel doc= JsonConvert.DeserializeObject<MovieModel>(moviesArray[i].ToString());
      doc.Id = Guid.NewGuid().ToString();
      concurrentTasks.Add(moviesContainer.CreateItemAsync(doc,new PartitionKey(doc.Year)));
      {
          j++;
          Console.Write("{0,5:#,##0}, ", j);
          if (j % 1000 == 0)
              Console.Write("\n               ");
          j += 99;
      }
      
  }
  catch (Exception ex)
  {
      Console.WriteLine("\n     ERROR: Could not write the movie record #{0:#,##0}, because:\n       {1}",
                          i, ex.Message);
      operationFailed = true;
      break;
  }
}
await Task.WhenAll(concurrentTasks);
```

### <a name="create-a-document"></a>Создание документа

**DynamoDB**.

Запись нового документа в Amazon DynamoDB не является типобезопасной. В следующем примере элемент newItem используется в качестве типа документа.

```csharp
Task<Document> writeNew = moviesTable.PutItemAsync(newItem, token);
await writeNew;
```

**Azure Cosmos DB**.

Azure Cosmos DB обеспечивает безопасность типов с помощью модели данных. Мы используем модель данных с именем MovieModel.

```csharp
public class MovieModel
{
    [JsonProperty("id")]
    public string Id { get; set; }
    [JsonProperty("title")]
    public string Title{ get; set; }
    [JsonProperty("year")]
    public int Year { get; set; }
    public MovieModel(string title, int year)
    {
        this.Title = title;
        this.Year = year;
    }
    public MovieModel()
    {

    }
    [JsonProperty("info")]
    public   MovieInfo MovieInfo { get; set; }

    internal string PrintInfo()
    {
        if(this.MovieInfo!=null)
        return            string.Format("\nMovie with title:{1}\n Year: {2}, Actors: {3}\n Directors:{4}\n Rating:{5}\n", this.Id, this.Title, this.Year, String.Join(",",this.MovieInfo.Actors), this.MovieInfo, this.MovieInfo.Rating);
        else
            return string.Format("\nMovie with  title:{0}\n Year: {1}\n",  this.Title, this.Year);
    }
}
```

В Azure Cosmos DB элемент newItem будет являться моделью MovieModel.

```csharp
 MovieModel movieModel = new MovieModel()
            {
                Id = Guid.NewGuid().ToString(),
                Title = "The Big New Movie",
                Year = 2018,
                MovieInfo = new MovieInfo() { Plot = "Nothing happens at all.", Rating = 0 }
            };
    var writeNew= moviesContainer.CreateItemAsync(movieModel, new Microsoft.Azure.Cosmos.PartitionKey(movieModel.Year));
    await writeNew;
```

### <a name="read-a-document"></a>Чтение документа

**DynamoDB**.

Для чтения данных в Amazon DynamoDB необходимо определить примитивы.

```csharp
// Create Primitives for the HASH and RANGE portions of the primary key
Primitive hash = new Primitive(year.ToString(), true);
Primitive range = new Primitive(title, false);

  Task<Document> readMovie = moviesTable.GetItemAsync(hash, range, token);
  movie_record = await readMovie;
```

**Azure Cosmos DB**.

Однако в Azure Cosmos DB такой запрос является естественным (LINQ).

```csharp
IQueryable<MovieModel> movieQuery = moviesContainer.GetItemLinqQueryable<MovieModel>(true)
                        .Where(f => f.Year == year && f.Title == title);
// The query is executed synchronously here, but can also be executed asynchronously via the IDocumentQuery<T> interface
    foreach (MovieModel movie in movieQuery)
    {
      movie_record_cosmosdb = movie;
    }
```

Коллекция документов в примере выше:

- является типобезопасной;
- предоставляет естественный параметр запроса.

### <a name="update-an-item"></a>Обновите элемент.

**DynamoDB**. Обновление элемента в Amazon DynamoDB.

```csharp
updateResponse = await client.UpdateItemAsync( updateRequest );
````

**Azure Cosmos DB**.

В Azure Cosmos DB операция обновления рассматривается как операция Upsert — это означает, что документ будет вставлен, если он не существует.

```csharp
await moviesContainer.UpsertItemAsync<MovieModel>(updatedMovieModel);
```

### <a name="delete-a-document"></a>Удаление документа

**DynamoDB**.

Чтобы удалить элемент в Amazon DynamoDB, необходимо еще раз обратиться к примитивам.

```csharp
Primitive hash = new Primitive(year.ToString(), true);
      Primitive range = new Primitive(title, false);
      DeleteItemOperationConfig deleteConfig = new DeleteItemOperationConfig( );
      deleteConfig.ConditionalExpression = condition;
      deleteConfig.ReturnValues = ReturnValues.AllOldAttributes;
      
  Task<Document> delItem = table.DeleteItemAsync( hash, range, deleteConfig );
        deletedItem = await delItem;
```

**Azure Cosmos DB**.

В Azure Cosmos DB можно асинхронно получить документ и удалить его.

```csharp
var result= ReadingMovieItem_async_List_CosmosDB("select * from c where c.info.rating>7 AND c.year=2018 AND c.title='The Big New Movie'");
while (result.HasMoreResults)
{
  var resultModel = await result.ReadNextAsync();
  foreach (var movie in resultModel.ToList<MovieModel>())
  {
    await moviesContainer.DeleteItemAsync<MovieModel>(movie.Id, new PartitionKey(movie.Year));
  }
  }
```

### <a name="query-documents"></a>Запрос документов

**DynamoDB**.

В Amazon DynamoDB для запроса данных необходимы функции API.

```csharp
QueryOperationConfig config = new QueryOperationConfig( );
  config.Filter = new QueryFilter( );
  config.Filter.AddCondition( "year", QueryOperator.Equal, new DynamoDBEntry[ ] { 1992 } );
  config.Filter.AddCondition( "title", QueryOperator.Between, new DynamoDBEntry[ ] { "B", "Hzz" } );
  config.AttributesToGet = new List<string> { "year", "title", "info" };
  config.Select = SelectValues.SpecificAttributes;
  search = moviesTable.Query( config ); 
```

**Azure Cosmos DB**.

В Azure Cosmos DB можно выполнить проекцию и фильтрацию внутри простого SQL-запроса.

```csharp
var result = moviesContainer.GetItemQueryIterator<MovieModel>( 
  "select c.Year, c.Title, c.info from c where Year=1998 AND (CONTAINS(Title,'B') OR CONTAINS(Title,'Hzz'))");
```

Для операций с диапазонами (например, BETWEEN) в Amazon DynamoDB необходимо выполнить проверку.

```csharp
ScanRequest sRequest = new ScanRequest
{
  TableName = "Movies",
  ExpressionAttributeNames = new Dictionary<string, string>
  {
    { "#yr", "year" }
  },
  ExpressionAttributeValues = new Dictionary<string, AttributeValue>
  {
      { ":y_a", new AttributeValue { N = "1960" } },
      { ":y_z", new AttributeValue { N = "1969" } },
  },
  FilterExpression = "#yr between :y_a and :y_z",
  ProjectionExpression = "#yr, title, info.actors[0], info.directors, info.running_time_secs"
};

ClientScanning_async( sRequest ).Wait( );
```

В Azure Cosmos DB можно использовать SQL-запрос и однострочную инструкцию.

```csharp
var result = moviesContainer.GetItemQueryIterator<MovieModel>( 
  "select c.title, c.info.actors[0], c.info.directors,c.info.running_time_secs from c where BETWEEN year 1960 AND 1969");
```

### <a name="delete-a-container"></a>Удаление контейнера

**DynamoDB**.

Для удаления таблицы в Amazon DynamoDB можно указать следующую инструкцию.

```csharp
client.DeleteTableAsync( tableName );
```

**Azure Cosmos DB**.

Для удаления коллекции в Azure Cosmos DB укажите следующую инструкцию.

```csharp
await moviesContainer.DeleteContainerAsync();
```
Затем можно удалить базу данных, если это требуется.

```csharp
await cosmosDatabase.DeleteAsync();
```

Как видите, Azure Cosmos DB поддерживает естественные запросы (SQL), а операции являются асинхронными и более простыми. Сложный код можно легко адаптировать для Azure Cosmos DB, и после этого он станет проще.

### <a name="next-steps"></a>Next Steps

- Дополнительные сведения об [оптимизации производительности](performance-tips.md).
- Дополнительные сведения об [оптимизации операций чтения и записи](key-value-store-cost.md).
- Дополнительные сведения о [мониторинге в Cosmos DB](monitor-cosmos-db.md).

