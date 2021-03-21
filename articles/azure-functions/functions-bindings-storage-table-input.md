---
title: Входные привязки хранилища таблиц Azure для функций Azure
description: Узнайте, как использовать входные привязки хранилища таблиц Azure в функциях Azure.
author: craigshoemaker
ms.topic: reference
ms.date: 09/03/2018
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 4fc2426189384856d2d2e95887cdabd2f9e9ebea
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98033784"
---
# <a name="azure-table-storage-input-bindings-for-azure-functions"></a>Входные привязки хранилища таблиц Azure для функций Azure

Используйте входную привязку хранилища таблиц Azure для чтения таблицы в учетной записи хранения Azure.

## <a name="example"></a>Пример

# <a name="c"></a>[C#](#tab/csharp)

### <a name="one-entity"></a>Одна сущность

В следующем примере показана [функция C#](functions-dotnet-class-library.md), которая считывает одну строку таблицы. Для каждого сообщения, отправленного в очередь, будет активирована функция.

Значение ключа строки {queueTrigger} указывает, что ключ строки получен из строки сообщения очереди.

```csharp
public class TableStorage
{
    public class MyPoco
    {
        public string PartitionKey { get; set; }
        public string RowKey { get; set; }
        public string Text { get; set; }
    }

    [FunctionName("TableInput")]
    public static void TableInput(
        [QueueTrigger("table-items")] string input, 
        [Table("MyTable", "MyPartition", "{queueTrigger}")] MyPoco poco, 
        ILogger log)
    {
        log.LogInformation($"PK={poco.PartitionKey}, RK={poco.RowKey}, Text={poco.Text}");
    }
}
```

### <a name="cloudtable"></a>CloudTable

`CloudTable` поддерживается только в [средах выполнения функций версии 2 и выше](functions-versions.md).

Используйте `CloudTable` параметр метода для чтения таблицы с помощью пакета SDK службы хранилища Azure. Ниже приведен пример функции, которая запрашивает таблицу журнала функций Azure:

```csharp
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;
using Microsoft.Azure.Cosmos.Table;
using System;
using System.Threading.Tasks;

namespace FunctionAppCloudTable2
{
    public class LogEntity : TableEntity
    {
        public string OriginalName { get; set; }
    }
    public static class CloudTableDemo
    {
        [FunctionName("CloudTableDemo")]
        public static async Task Run(
            [TimerTrigger("0 */1 * * * *")] TimerInfo myTimer, 
            [Table("AzureWebJobsHostLogscommon")] CloudTable cloudTable,
            ILogger log)
        {
            log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");

            TableQuery<LogEntity> rangeQuery = new TableQuery<LogEntity>().Where(
                TableQuery.CombineFilters(
                    TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, 
                        "FD2"),
                    TableOperators.And,
                    TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.GreaterThan, 
                        "t")));

            // Execute the query and loop through the results
            foreach (LogEntity entity in 
                await cloudTable.ExecuteQuerySegmentedAsync(rangeQuery, null))
            {
                log.LogInformation(
                    $"{entity.PartitionKey}\t{entity.RowKey}\t{entity.Timestamp}\t{entity.OriginalName}");
            }
        }
    }
}
```

Дополнительную информацию об использовании CloudTable см. в статье [Начало работы с хранилищем таблиц Azure и API таблиц Azure Cosmos DB с помощью .NET](../cosmos-db/tutorial-develop-table-dotnet.md).

Если при попытке привязать к `CloudTable` вы получаете сообщение об ошибке, убедитесь, что у вас есть ссылка на [правильную версию пакета SDK для службы хранилища](./functions-bindings-storage-table.md#azure-storage-sdk-version-in-functions-1x).

### <a name="iqueryable"></a>IQueryable

`IQueryable` поддерживается только в [среде выполнения функций v1](functions-versions.md).

В следующем примере показана [функция C#](functions-dotnet-class-library.md) , которая считывает несколько строк таблицы, `MyPoco` от которых наследуется класс `TableEntity` .

```csharp
public class TableStorage
{
    public class MyPoco : TableEntity
    {
        public string Text { get; set; }
    }

    [FunctionName("TableInput")]
    public static void TableInput(
        [QueueTrigger("table-items")] string input, 
        [Table("MyTable", "MyPartition")] IQueryable<MyPoco> pocos, 
        ILogger log)
    {
        foreach (MyPoco poco in pocos)
        {
            log.LogInformation($"PK={poco.PartitionKey}, RK={poco.RowKey}, Text={poco.Text}");
        }
    }
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

### <a name="one-entity"></a>Одна сущность

В следующем примере показана входная привязка таблицы в файле *function.json* и коде [скрипта C#](functions-reference-csharp.md), который использует привязку. Функция использует триггер очереди для чтения одной строки таблицы. 

Файл *function.json* определяет `partitionKey` и `rowKey`. Значение `rowKey` "{queueTrigger}" указывает, что ключ строки получен из строки сообщения очереди.

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "personEntity",
      "type": "table",
      "tableName": "Person",
      "partitionKey": "Test",
      "rowKey": "{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

Ниже приведен код скрипта C#.

```csharp
public static void Run(string myQueueItem, Person personEntity, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
    log.LogInformation($"Name in Person entity: {personEntity.Name}");
}

public class Person
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public string Name { get; set; }
}
```

### <a name="cloudtable"></a>CloudTable

`IQueryable` не поддерживается в среде выполнения функций для [версий 2. x и выше](functions-versions.md). Альтернативой является использование параметра метода `CloudTable` для чтения таблицы с помощью пакета SDK службы хранилища Azure. Ниже приведен пример функции, которая запрашивает таблицу журнала функций Azure:

```json
{
  "bindings": [
    {
      "name": "myTimer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 */1 * * * *"
    },
    {
      "name": "cloudTable",
      "type": "table",
      "connection": "AzureWebJobsStorage",
      "tableName": "AzureWebJobsHostLogscommon",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

```csharp
#r "Microsoft.WindowsAzure.Storage"
using Microsoft.WindowsAzure.Storage.Table;
using System;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

public static async Task Run(TimerInfo myTimer, CloudTable cloudTable, ILogger log)
{
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");

    TableQuery<LogEntity> rangeQuery = new TableQuery<LogEntity>().Where(
    TableQuery.CombineFilters(
        TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, 
            "FD2"),
        TableOperators.And,
        TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.GreaterThan, 
            "a")));

    // Execute the query and loop through the results
    foreach (LogEntity entity in 
    await cloudTable.ExecuteQuerySegmentedAsync(rangeQuery, null))
    {
        log.LogInformation(
            $"{entity.PartitionKey}\t{entity.RowKey}\t{entity.Timestamp}\t{entity.OriginalName}");
    }
}

public class LogEntity : TableEntity
{
    public string OriginalName { get; set; }
}
```

Дополнительную информацию об использовании CloudTable см. в статье [Начало работы с хранилищем таблиц Azure и API таблиц Azure Cosmos DB с помощью .NET](../cosmos-db/tutorial-develop-table-dotnet.md).

Если при попытке привязать к `CloudTable` вы получаете сообщение об ошибке, убедитесь, что у вас есть ссылка на [правильную версию пакета SDK для службы хранилища](./functions-bindings-storage-table.md#azure-storage-sdk-version-in-functions-1x).

### <a name="iqueryable"></a>IQueryable

В следующем примере показана входная привязка таблицы в файле *function.json* и коде [скрипта C#](functions-reference-csharp.md), который использует привязку. Функция считывает сущности ключа раздела, который указан в очереди сообщений.

Ниже показан файл *function.json*.

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "tableBinding",
      "type": "table",
      "connection": "MyStorageConnectionAppSetting",
      "tableName": "Person",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

В коде скрипта C# добавляется ссылка на пакет SDK для службы хранилища Azure, чтобы тип сущности мог быть производным от `TableEntity`.

```csharp
#r "Microsoft.WindowsAzure.Storage"
using Microsoft.WindowsAzure.Storage.Table;
using Microsoft.Extensions.Logging;

public static void Run(string myQueueItem, IQueryable<Person> tableBinding, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
    foreach (Person person in tableBinding.Where(p => p.PartitionKey == myQueueItem).ToList())
    {
        log.LogInformation($"Name: {person.Name}");
    }
}

public class Person : TableEntity
{
    public string Name { get; set; }
}
```

# <a name="java"></a>[Java](#tab/java)

В следующем примере показана функция, активируемая HTTP, которая возвращает список объектов Person, которые находятся в указанном разделе в хранилище таблиц. В этом примере ключ секции извлекается из маршрута HTTP, а tableName и Connection — из параметров функции. 

```java
public class Person {
    private String PartitionKey;
    private String RowKey;
    private String Name;

    public String getPartitionKey() { return this.PartitionKey; }
    public void setPartitionKey(String key) { this.PartitionKey = key; }
    public String getRowKey() { return this.RowKey; }
    public void setRowKey(String key) { this.RowKey = key; }
    public String getName() { return this.Name; }
    public void setName(String name) { this.Name = name; }
}

@FunctionName("getPersonsByPartitionKey")
public Person[] get(
        @HttpTrigger(name = "getPersons", methods = {HttpMethod.GET}, authLevel = AuthorizationLevel.FUNCTION, route="persons/{partitionKey}") HttpRequestMessage<Optional<String>> request,
        @BindingName("partitionKey") String partitionKey,
        @TableInput(name="persons", partitionKey="{partitionKey}", tableName="%MyTableName%", connection="MyConnectionString") Person[] persons,
        final ExecutionContext context) {

    context.getLogger().info("Got query for person related to persons with partition key: " + partitionKey);

    return persons;
}
```

Заметка Таблеинпут также может извлекать привязки из тела запроса JSON, как показано в следующем примере.

```java
@FunctionName("GetPersonsByKeysFromRequest")
public HttpResponseMessage get(
        @HttpTrigger(name = "getPerson", methods = {HttpMethod.GET}, authLevel = AuthorizationLevel.FUNCTION, route="query") HttpRequestMessage<Optional<String>> request,
        @TableInput(name="persons", partitionKey="{partitionKey}", rowKey = "{rowKey}", tableName="%MyTableName%", connection="MyConnectionString") Person person,
        final ExecutionContext context) {

    if (person == null) {
        return request.createResponseBuilder(HttpStatus.NOT_FOUND)
                    .body("Person not found.")
                    .build();
    }

    return request.createResponseBuilder(HttpStatus.OK)
                    .header("Content-Type", "application/json")
                    .body(person)
                    .build();
}
```

В следующем примере фильтр используется для запроса лиц с указанным именем в таблице Azure и ограничивает количество возможных совпадений до 10 результатов.

```java
@FunctionName("getPersonsByName")
public Person[] get(
        @HttpTrigger(name = "getPersons", methods = {HttpMethod.GET}, authLevel = AuthorizationLevel.FUNCTION, route="filter/{name}") HttpRequestMessage<Optional<String>> request,
        @BindingName("name") String name,
        @TableInput(name="persons", filter="Name eq '{name}'", take = "10", tableName="%MyTableName%", connection="MyConnectionString") Person[] persons,
        final ExecutionContext context) {

    context.getLogger().info("Got query for person related to persons with name: " + name);

    return persons;
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В следующем примере показана входная привязка таблицы в файле *function.json* и [код JavaScript](functions-reference-node.md), который использует привязку. Функция использует триггер очереди для чтения одной строки таблицы. 

Файл *function.json* определяет `partitionKey` и `rowKey`. Значение `rowKey` "{queueTrigger}" указывает, что ключ строки получен из строки сообщения очереди.

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "personEntity",
      "type": "table",
      "tableName": "Person",
      "partitionKey": "Test",
      "rowKey": "{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

Ниже показан код JavaScript.

```javascript
module.exports = function (context, myQueueItem) {
    context.log('Node.js queue trigger function processed work item', myQueueItem);
    context.log('Person entity name: ' + context.bindings.personEntity.Name);
    context.done();
};
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Следующая функция использует триггер очереди для чтения одной строки таблицы в качестве входных данных для функции.

В этом примере конфигурация привязки указывает явное значение для таблицы `partitionKey` и использует выражение для передачи в `rowKey` . `rowKey`Выражение, `{queueTrigger}` указывает, что ключ строки поступает из строки сообщения очереди.

Привязка конфигурации в _function.js_:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "MyQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "PersonEntity",
      "type": "table",
      "tableName": "Person",
      "partitionKey": "Test",
      "rowKey": "{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

Код PowerShell в _run.ps1_:

```powershell
param($MyQueueItem, $PersonEntity, $TriggerMetadata)
Write-Host "PowerShell queue trigger function processed work item: $MyQueueItem"
Write-Host "Person entity name: $($PersonEntity.Name)"
```

# <a name="python"></a>[Python](#tab/python)

Следующая функция использует триггер очереди для чтения одной строки таблицы в качестве входных данных для функции.

В этом примере конфигурация привязки указывает явное значение для таблицы `partitionKey` и использует выражение для передачи в `rowKey` . `rowKey`Выражение `{id}` указывает, что ключ строки поступает из строки сообщения очереди.

Привязка конфигурации в _function.js_ файла:

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "messageJSON",
      "type": "table",
      "tableName": "messages",
      "partitionKey": "message",
      "rowKey": "{id}",
      "connection": "AzureWebJobsStorage",
      "direction": "in"
    },
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ],
      "route": "messages/{id}"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ],
  "disabled": false
}
```

Код Python в файле *\_ \_ init \_ \_ . корректировки* :

```python
import json

import azure.functions as func

def main(req: func.HttpRequest, messageJSON) -> func.HttpResponse:

    message = json.loads(messageJSON)
    return func.HttpResponse(f"Table row: {messageJSON}")
```

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

 В [библиотеках класса C#](functions-dotnet-class-library.md) используйте следующие атрибуты для настройки входной привязки таблицы:

* [TableAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Tables/TableAttribute.cs)

  Конструктор атрибута принимает имя таблицы, ключ раздела и строки. Атрибут может использоваться для `out` параметра или возвращаемого значения функции, как показано в следующем примере:

  ```csharp
  [FunctionName("TableInput")]
  public static void Run(
      [QueueTrigger("table-items")] string input, 
      [Table("MyTable", "Http", "{queueTrigger}")] MyPoco poco, 
      ILogger log)
  {
      ...
  }
  ```

  Чтобы указать учетную запись хранения, которую нужно использовать, можно задать свойство `Connection`, как показано в следующем примере.

  ```csharp
  [FunctionName("TableInput")]
  public static void Run(
      [QueueTrigger("table-items")] string input, 
      [Table("MyTable", "Http", "{queueTrigger}", Connection = "StorageConnectionAppSetting")] MyPoco poco, 
      ILogger log)
  {
      ...
  }
  ```

  Полный пример см. в примере на C#.

* [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs)

  Предоставляет еще один способ указать используемую учетную запись хранения. Конструктор принимает имя параметра приложения, содержащего строку подключения к службе хранилища. Атрибут может применяться на уровне класса, метода или параметра. Ниже показан пример уровня класса и метода.

  ```csharp
  [StorageAccount("ClassLevelStorageAppSetting")]
  public static class AzureFunctions
  {
      [FunctionName("TableInput")]
      [StorageAccount("FunctionLevelStorageAppSetting")]
      public static void Run( //...
  {
      ...
  }
  ```

Используемая учетная запись хранения определяется в следующем порядке:

* Свойство `Table` атрибута `Connection`.
* Атрибут `StorageAccount`, примененный к тому же параметру, что и `Table`.
* Атрибут `StorageAccount`, примененный к функции.
* Атрибут `StorageAccount`, примененный к классу.
* Учетная запись хранения по умолчанию для приложения-функции (параметр приложения AzureWebJobsStorage).

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

В [библиотеке среды выполнения функций Java](/java/api/overview/azure/functions/runtime) используйте заметку `@TableInput` для параметров, значения которых будут поступать из Хранилища таблиц.  Эта заметка может использоваться с собственными типами Java, объектами POJO или значениями, допускающими значения NULL, используя `Optional<T>`.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В JavaScript атрибуты не поддерживаются.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В PowerShell не поддерживаются атрибуты.

# <a name="python"></a>[Python](#tab/python)

В Python атрибуты не поддерживаются.

---

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `Table`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
|**type** | Недоступно | Нужно задать значение `table`. Это свойство задается автоматически при создании привязки на портале Azure.|
|**direction** | Недоступно | Нужно задать значение `in`. Это свойство задается автоматически при создании привязки на портале Azure. |
|**name** | Недоступно | Имя переменной, представляющей таблицу или сущность в коде функции. | 
|**tableName** | **TableName** | Имя таблицы.| 
|**partitionKey** | **PartitionKey** |Необязательный параметр. Ключ раздела сущности таблицы, которую нужно считать. Инструкции по использованию этого свойства см. в разделе [Использование](#usage) .| 
|**rowKey** |**RowKey** | Необязательный параметр. Ключ строки сущности таблицы, которую нужно считать. Инструкции по использованию этого свойства см. в разделе [Использование](#usage) .| 
|**take** |**Take** | Необязательный параметр. Максимальное количество считываемых сущностей в JavaScript. Инструкции по использованию этого свойства см. в разделе [Использование](#usage) .| 
|**filter** |**Фильтр** | Необязательный параметр. Выражение фильтра OData для входных данных таблицы в JavaScript. Инструкции по использованию этого свойства см. в разделе [Использование](#usage) .| 
|**connection**; |**Соединение** | Имя параметра приложения, содержащего строку подключения к службе хранилища, используемой для этой привязки. Параметр может быть именем предопределенного параметра приложения "AzureWebJobs" или именем строки подключения. Например, если имя параметра — "Азуревебжобсмистораже", можно указать "MyStorage" здесь. Среда выполнения функций будет автоматически искать параметр приложения с именем «Азуревебжобсмистораже». Если оставить строку `connection` пустой, среда выполнения службы "Функции" будет использовать строку подключения к службе хранилища по умолчанию для параметра приложения с именем `AzureWebJobsStorage`.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Использование

# <a name="c"></a>[C#](#tab/csharp)

* **Чтение одной строки в**

  Задайте значения для `partitionKey` и `rowKey`. Получите доступ к данным таблицы с помощью параметра метода `T <paramName>`. В скрипте C# `paramName` — это значение, заданное в свойстве `name` файла *function.json*. `T` обычно представляет собой тип, реализующий `ITableEntity`, или является производным от `TableEntity`. Свойства `filter` и `take` не используются в этом сценарии.

* **Чтение одной или нескольких строк**

  Получите доступ к данным таблицы с помощью параметра метода `IQueryable<T> <paramName>`. В скрипте C# `paramName` — это значение, заданное в свойстве `name` файла *function.json*. `T` должен представлять собой тип, реализующий `ITableEntity`, или быть производным от `TableEntity`. Для выполнения фильтрации можно использовать методы `IQueryable`. Свойства `partitionKey`, `rowKey`, `filter` и `take` не используются в этом сценарии.  

  > [!NOTE]
  > `IQueryable` не поддерживается в [среде выполнения Функций версии 2](functions-versions.md). Альтернативой является [использование параметра метода CloudTable paramName](https://stackoverflow.com/questions/48922485/binding-to-table-storage-in-v2-azure-functions-using-cloudtable) для чтения таблицы с помощью пакета SDK службы хранилища Azure. Если при попытке привязать к `CloudTable` вы получаете сообщение об ошибке, убедитесь, что у вас есть ссылка на [правильную версию пакета SDK для службы хранилища](./functions-bindings-storage-table.md#azure-storage-sdk-version-in-functions-1x).

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

* **Чтение одной строки в**

  Задайте значения для `partitionKey` и `rowKey`. Получите доступ к данным таблицы с помощью параметра метода `T <paramName>`. В скрипте C# `paramName` — это значение, заданное в свойстве `name` файла *function.json*. `T` обычно представляет собой тип, реализующий `ITableEntity`, или является производным от `TableEntity`. Свойства `filter` и `take` не используются в этом сценарии.

* **Чтение одной или нескольких строк**

  Получите доступ к данным таблицы с помощью параметра метода `IQueryable<T> <paramName>`. В скрипте C# `paramName` — это значение, заданное в свойстве `name` файла *function.json*. `T` должен представлять собой тип, реализующий `ITableEntity`, или быть производным от `TableEntity`. Для выполнения фильтрации можно использовать методы `IQueryable`. Свойства `partitionKey`, `rowKey`, `filter` и `take` не используются в этом сценарии.  

  > [!NOTE]
  > `IQueryable` не поддерживается в [среде выполнения Функций версии 2](functions-versions.md). Альтернативой является [использование параметра метода CloudTable paramName](https://stackoverflow.com/questions/48922485/binding-to-table-storage-in-v2-azure-functions-using-cloudtable) для чтения таблицы с помощью пакета SDK службы хранилища Azure. Если при попытке привязать к `CloudTable` вы получаете сообщение об ошибке, убедитесь, что у вас есть ссылка на [правильную версию пакета SDK для службы хранилища](./functions-bindings-storage-table.md#azure-storage-sdk-version-in-functions-1x).

# <a name="java"></a>[Java](#tab/java)

Атрибут [таблеинпут](/java/api/com.microsoft.azure.functions.annotation.tableinput) предоставляет доступ к строке таблицы, которая активировала функцию.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Укажите свойства `filter` и `take`. Не определяйте свойства `partitionKey` и `rowKey`. Получите доступ к сущности (или сущностям) входной таблицы, используя `context.bindings.<BINDING_NAME>`. Десериализированный объект имеет свойства `RowKey` и `PartitionKey`.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Данные передаются в входной параметр, как указано `name` ключом в *function.js* в файле. Указание `partitionKey` и `rowKey` позволяет выполнять фильтрацию по конкретным записям. Дополнительные сведения см. в [примере PowerShell](#example) .

# <a name="python"></a>[Python](#tab/python)

Табличные данные передаются в функцию в виде строки JSON. Отмените сериализацию сообщения, вызвав, `json.loads` как показано во входном [примере](#example).

---

## <a name="next-steps"></a>Дальнейшие действия

* [Запись данных хранилища таблиц из функции](./functions-bindings-storage-table-output.md)
