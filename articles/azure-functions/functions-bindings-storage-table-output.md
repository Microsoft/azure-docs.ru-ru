---
title: Выходные привязки хранилища таблиц Azure для функций Azure
description: Узнайте, как использовать выходные привязки хранилища таблиц Azure в функциях Azure.
author: craigshoemaker
ms.topic: reference
ms.date: 09/03/2018
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: f793f96f55a258b2d7cb11f214984416557618df
ms.sourcegitcommit: 6386854467e74d0745c281cc53621af3bb201920
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102453008"
---
# <a name="azure-table-storage-output-bindings-for-azure-functions"></a>Выходные привязки хранилища таблиц Azure для функций Azure

Используйте выходную привязку хранилища таблиц Azure для записи сущностей в таблицу в учетной записи хранения Azure.

> [!NOTE]
> Выходная привязка не поддерживает обновление существующих сущностей. Для обновления существующей сущности используйте операцию `TableOperation.Replace` из [пакета SDK для службы хранилища Azure](../cosmos-db/tutorial-develop-table-dotnet.md#delete-an-entity).

## <a name="example"></a>Пример

# <a name="c"></a>[C#](#tab/csharp)

В следующем примере показана [функция C#](functions-dotnet-class-library.md), которая использует триггер HTTP для записи одной строки таблицы.

```csharp
public class TableStorage
{
    public class MyPoco
    {
        public string PartitionKey { get; set; }
        public string RowKey { get; set; }
        public string Text { get; set; }
    }

    [FunctionName("TableOutput")]
    [return: Table("MyTable")]
    public static MyPoco TableOutput([HttpTrigger] dynamic input, ILogger log)
    {
        log.LogInformation($"C# http trigger function processed: {input.Text}");
        return new MyPoco { PartitionKey = "Http", RowKey = Guid.NewGuid().ToString(), Text = input.Text };
    }
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В следующем примере показана выходная привязка таблицы в файле *function.json* и коде [скрипта C#](functions-reference-csharp.md), который использует привязку. Эта функция записывает несколько сущностей в таблице.

Ниже показан файл *function.json*.

```json
{
  "bindings": [
    {
      "name": "input",
      "type": "manualTrigger",
      "direction": "in"
    },
    {
      "tableName": "Person",
      "connection": "MyStorageConnectionAppSetting",
      "name": "tableBinding",
      "type": "table",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

Ниже приведен код скрипта C#.

```csharp
public static void Run(string input, ICollector<Person> tableBinding, ILogger log)
{
    for (int i = 1; i < 10; i++)
        {
            log.LogInformation($"Adding Person entity {i}");
            tableBinding.Add(
                new Person() { 
                    PartitionKey = "Test", 
                    RowKey = i.ToString(), 
                    Name = "Name" + i.ToString() }
                );
        }

}

public class Person
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public string Name { get; set; }
}

```

# <a name="java"></a>[Java](#tab/java)

В следующем примере показана функция Java, использующая триггер HTTP для записи одной строки таблицы.

```java
public class Person {
    private String PartitionKey;
    private String RowKey;
    private String Name;

    public String getPartitionKey() {return this.PartitionKey;}
    public void setPartitionKey(String key) {this.PartitionKey = key; }
    public String getRowKey() {return this.RowKey;}
    public void setRowKey(String key) {this.RowKey = key; }
    public String getName() {return this.Name;}
    public void setName(String name) {this.Name = name; }
}

public class AddPerson {

    @FunctionName("addPerson")
    public HttpResponseMessage get(
            @HttpTrigger(name = "postPerson", methods = {HttpMethod.POST}, authLevel = AuthorizationLevel.FUNCTION, route="persons/{partitionKey}/{rowKey}") HttpRequestMessage<Optional<Person>> request,
            @BindingName("partitionKey") String partitionKey,
            @BindingName("rowKey") String rowKey,
            @TableOutput(name="person", partitionKey="{partitionKey}", rowKey = "{rowKey}", tableName="%MyTableName%", connection="MyConnectionString") OutputBinding<Person> person,
            final ExecutionContext context) {

        Person outPerson = new Person();
        outPerson.setPartitionKey(partitionKey);
        outPerson.setRowKey(rowKey);
        outPerson.setName(request.getBody().get().getName());

        person.setValue(outPerson);

        return request.createResponseBuilder(HttpStatus.OK)
                        .header("Content-Type", "application/json")
                        .body(outPerson)
                        .build();
    }
}
```

В следующем примере показана функция Java, использующая триггер HTTP для записи нескольких строк таблицы.

```java
public class Person {
    private String PartitionKey;
    private String RowKey;
    private String Name;

    public String getPartitionKey() {return this.PartitionKey;}
    public void setPartitionKey(String key) {this.PartitionKey = key; }
    public String getRowKey() {return this.RowKey;}
    public void setRowKey(String key) {this.RowKey = key; }
    public String getName() {return this.Name;}
    public void setName(String name) {this.Name = name; }
}

public class AddPersons {

    @FunctionName("addPersons")
    public HttpResponseMessage get(
            @HttpTrigger(name = "postPersons", methods = {HttpMethod.POST}, authLevel = AuthorizationLevel.FUNCTION, route="persons/") HttpRequestMessage<Optional<Person[]>> request,
            @TableOutput(name="person", tableName="%MyTableName%", connection="MyConnectionString") OutputBinding<Person[]> persons,
            final ExecutionContext context) {

        persons.setValue(request.getBody().get());

        return request.createResponseBuilder(HttpStatus.OK)
                        .header("Content-Type", "application/json")
                        .body(request.getBody().get())
                        .build();
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В следующем примере показана выходная привязка таблицы в файле *function.json* и функции [JavaScript](functions-reference-node.md), которая использует привязку. Эта функция записывает несколько сущностей в таблице.

Ниже показан файл *function.json*.

```json
{
  "bindings": [
    {
      "name": "input",
      "type": "manualTrigger",
      "direction": "in"
    },
    {
      "tableName": "Person",
      "connection": "MyStorageConnectionAppSetting",
      "name": "tableBinding",
      "type": "table",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

Ниже показан код JavaScript.

```javascript
module.exports = function (context) {

    context.bindings.tableBinding = [];

    for (var i = 1; i < 10; i++) {
        context.bindings.tableBinding.push({
            PartitionKey: "Test",
            RowKey: i.toString(),
            Name: "Name " + i
        });
    }

    context.done();
};
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В следующем примере показано, как записать несколько сущностей в таблицу из функции.

Привязка конфигурации в _function.js_:

```json
{
  "bindings": [
    {
      "name": "InputData",
      "type": "manualTrigger",
      "direction": "in"
    },
    {
      "tableName": "Person",
      "connection": "MyStorageConnectionAppSetting",
      "name": "TableBinding",
      "type": "table",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

Код PowerShell в _run.ps1_:

```powershell
param($InputData, $TriggerMetadata)
  
foreach ($i in 1..10) {
    Push-OutputBinding -Name TableBinding -Value @{
        PartitionKey = 'Test'
        RowKey = "$i"
        Name = "Name $i"
    }
}
```

# <a name="python"></a>[Python](#tab/python)

В следующем примере показано, как использовать выходную привязку хранилища таблиц. `table`Привязка настраивается в *function.jsв* , присваивая значения `name` , `tableName` , `partitionKey` и `connection` :

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "message",
      "type": "table",
      "tableName": "messages",
      "partitionKey": "message",
      "connection": "AzureWebJobsStorage",
      "direction": "out"
    },
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

Следующая функция создает уникальный УУИ для `rowKey` значения и сохраняет сообщение в хранилище таблиц.

```python
import logging
import uuid
import json

import azure.functions as func

def main(req: func.HttpRequest, message: func.Out[str]) -> func.HttpResponse:

    rowKey = str(uuid.uuid4())

    data = {
        "Name": "Output binding message",
        "PartitionKey": "message",
        "RowKey": rowKey
    }

    message.set(json.dumps(data))

    return func.HttpResponse(f"Message created with the rowKey: {rowKey}")
```

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

В [библиотеках классов C#](functions-dotnet-class-library.md) используйте [TableAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Tables/TableAttribute.cs).

Конструктор атрибута использует имя таблицы. Атрибут может использоваться для `out` параметра или возвращаемого значения функции, как показано в следующем примере:

```csharp
[FunctionName("TableOutput")]
[return: Table("MyTable")]
public static MyPoco TableOutput(
    [HttpTrigger] dynamic input, 
    ILogger log)
{
    ...
}
```

Чтобы указать учетную запись хранения, которую нужно использовать, можно задать свойство `Connection`, как показано в следующем примере.

```csharp
[FunctionName("TableOutput")]
[return: Table("MyTable", Connection = "StorageConnectionAppSetting")]
public static MyPoco TableOutput(
    [HttpTrigger] dynamic input, 
    ILogger log)
{
    ...
}
```

Полный пример см. в примере на [C#](#example).

Чтобы указать учетную запись хранения на уровне класса, метода или параметра, можно использовать атрибут `StorageAccount`. Дополнительные сведения см. в разделе [Входные атрибуты](./functions-bindings-storage-table-input.md#attributes-and-annotations).

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

В [библиотеке времени выполнения функций Java](/java/api/overview/azure/functions/runtime)используйте аннотацию [таблеаутпут](https://github.com/Azure/azure-functions-java-library/blob/master/src/main/java/com/microsoft/azure/functions/annotation/TableOutput.java/) в параметрах для записи значений в хранилище таблиц.

[Дополнительные сведения](#example)см. в примере.

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
|**direction** | Недоступно | Нужно задать значение `out`. Это свойство задается автоматически при создании привязки на портале Azure. |
|**name** | Недоступно | Имя переменной, используемое в функции кода, которая представляет таблицу или сущность. Задайте значение `$return`, ссылающееся на возвращаемое значение функции.| 
|**tableName** |**TableName** | Имя таблицы.| 
|**partitionKey** |**PartitionKey** | Ключ раздела сущности таблицы, которую нужно записать. Дополнительные сведения о том, как использовать это свойство, см. в этом [разделе](#usage).| 
|**rowKey** |**RowKey** | Ключ строки сущности таблицы, которую нужно записать. Дополнительные сведения о том, как использовать это свойство, см. в этом [разделе](#usage).| 
|**connection**; |**Соединение** | Имя параметра приложения, содержащего строку подключения к службе хранилища, используемой для этой привязки. Если имя параметра приложения начинается с AzureWebJobs, можно указать только остальную часть имени. Например, если задано значение `connection` "MyStorage", среда выполнения функций ищет параметр приложения с именем "MyStorage". Если оставить строку `connection` пустой, среда выполнения службы "Функции" будет использовать строку подключения к службе хранилища по умолчанию для параметра приложения с именем `AzureWebJobsStorage`.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Использование

# <a name="c"></a>[C#](#tab/csharp)

Получить доступ к сущности выходной таблицы можно с помощью параметра `ICollector<T> paramName` метода `IAsyncCollector<T> paramName` или `T` , где `PartitionKey` включает `RowKey` Свойства и. Эти свойства часто сопровождаются реализацией `ITableEntity` или наследованием `TableEntity` .

Кроме того, можно использовать `CloudTable` параметр метода для записи в таблицу с помощью пакета SDK службы хранилища Azure. Если при попытке привязать к `CloudTable` вы получаете сообщение об ошибке, убедитесь, что у вас есть ссылка на [правильную версию пакета SDK для службы хранилища](./functions-bindings-storage-table.md#azure-storage-sdk-version-in-functions-1x).

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

Получить доступ к сущности выходной таблицы можно с помощью параметра `ICollector<T> paramName` метода `IAsyncCollector<T> paramName` или `T` , где `PartitionKey` включает `RowKey` Свойства и. Эти свойства часто сопровождаются реализацией `ITableEntity` или наследованием `TableEntity` . `paramName`Значение указывается в `name` свойстве *function.json*.

Кроме того, можно использовать `CloudTable` параметр метода для записи в таблицу с помощью пакета SDK службы хранилища Azure. Если при попытке привязать к `CloudTable` вы получаете сообщение об ошибке, убедитесь, что у вас есть ссылка на [правильную версию пакета SDK для службы хранилища](./functions-bindings-storage-table.md#azure-storage-sdk-version-in-functions-1x).

# <a name="java"></a>[Java](#tab/java)

Существует два варианта вывода строки хранилища таблицы из функции с помощью аннотации [таблесторажеаутпут](/java/api/com.microsoft.azure.functions.annotation.tableoutput) :

- **Возвращаемое значение**: применяя заметку к самой функции, возвращаемое значение функции сохраняется как строка хранилища таблицы.

- **Императивное**: чтобы явно задать значение сообщения, примените заметку к конкретному параметру типа [`OutputBinding<T>`](/java/api/com.microsoft.azure.functions.outputbinding) , где `T` включает `PartitionKey` Свойства и `RowKey` . Эти свойства часто сопровождаются реализацией `ITableEntity` или наследованием `TableEntity` .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Доступ к выходному событию с помощью `context.bindings.<name>`, где `<name>` — это значение, указанное в свойстве `name` файла *function.json*.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Для записи данных в таблицу используйте `Push-OutputBinding` командлет, присвойте `-Name TableBinding` параметру и `-Value` параметру значение, равное строке данных. Дополнительные сведения см. в [примере PowerShell](#example) .

# <a name="python"></a>[Python](#tab/python)

Существует два варианта вывода сообщения строки табличного хранилища из функции.

- **Возвращаемое значение** — задайте для свойства `name` в файле *function.json* значение `$return`. В этой конфигурации возвращаемое значение функции сохраняется как строка хранилища таблицы.

- **Императив** — передайте значение методу [set](/python/api/azure-functions/azure.functions.out#set-val--t-----none) параметра, объявленного с типом [Out](/python/api/azure-functions/azure.functions.out). Значение, переданное `set`, сохраняется как сообщение концентратора событий.

---

## <a name="exceptions-and-return-codes"></a>Исключения и коды возврата

| Привязка | Справочник |
|---|---|
| Таблица | [Коды ошибок таблицы](/rest/api/storageservices/fileservices/table-service-error-codes) |
| Большой двоичный объект, таблица, очередь | [Коды ошибок хранилища](/rest/api/storageservices/fileservices/common-rest-api-error-codes) |
| Большой двоичный объект, таблица, очередь | [Устранение неполадок](/rest/api/storageservices/fileservices/troubleshooting-api-operations) |

## <a name="next-steps"></a>Следующие шаги

> [!div class="nextstepaction"]
> [Основные понятия триггеров и привязок в Функциях Azure](functions-triggers-bindings.md)
