---
title: Триггер HTTP в Функциях Azure
description: Узнайте, как вызывать Функцию Azure через HTTP.
author: craigshoemaker
ms.topic: reference
ms.date: 02/21/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 4550be7ae8c543eea1bdfa085db6f23fe668a121
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105025702"
---
# <a name="azure-functions-http-trigger"></a>Триггер HTTP в Функциях Azure

Триггер HTTP позволяет вызвать функцию с помощью HTTP-запроса. Триггер HTTP можно использовать для создания независимых от сервера API-интерфейсов и для ответа веб-перехватчикам.

Возвращаемое по умолчанию значение для функции, активируемой по HTTP:

- `HTTP 204 No Content` с пустым телом в Функциях 2.x и более поздних версий;
- `HTTP 200 OK` с пустым телом в Функциях 1.x.

Чтобы изменить HTTP-ответ, настройте [выходную привязку](./functions-bindings-http-webhook-output.md).

Дополнительные сведения о привязках HTTP см. в [обзорной статье](./functions-bindings-http-webhook.md) и [справочнике по выходным привязкам](./functions-bindings-http-webhook-output.md).

[!INCLUDE [HTTP client best practices](../../includes/functions-http-client-best-practices.md)]

## <a name="example"></a>Пример

# <a name="c"></a>[C#](#tab/csharp)

В следующем примере показана [функция C#](functions-dotnet-class-library.md), выполняющая поиск параметра `name` в строке запроса или в тексте HTTP-запроса. Обратите внимание, что возвращаемое значение используется для привязки выходных данных, но атрибут возвращаемого значения не является обязательным.

```cs
[FunctionName("HttpTriggerCSharp")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)]
    HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string name = req.Query["name"];
    
    string requestBody = String.Empty;
    using (StreamReader streamReader =  new  StreamReader(req.Body))
    {
        requestBody = await streamReader.ReadToEndAsync();
    }
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;
    
    return name != null
        ? (ActionResult)new OkObjectResult($"Hello, {name}")
        : new BadRequestObjectResult("Please pass a name on the query string or in the request body");
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В следующем примере показаны привязка триггера в файле *function.json* и [функция сценария C#](functions-reference-csharp.md), которая использует эту привязку. В функции выполняется поиск параметра `name` в строке запроса или в тексте HTTP-запроса.

Ниже показан файл *function.json*.

```json
{
    "disabled": false,
    "bindings": [
        {
            "authLevel": "function",
            "name": "req",
            "type": "httpTrigger",
            "direction": "in",
            "methods": [
                "get",
                "post"
            ]
        },
        {
            "name": "$return",
            "type": "http",
            "direction": "out"
        }
    ]
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

Ниже приведен код сценария C#, который выполняет привязку к `HttpRequest`:

```cs
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;

public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string name = req.Query["name"];
    
    string requestBody = String.Empty;
    using (StreamReader streamReader =  new  StreamReader(req.Body))
    {
        requestBody = await streamReader.ReadToEndAsync();
    }
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;
    
    return name != null
        ? (ActionResult)new OkObjectResult($"Hello, {name}")
        : new BadRequestObjectResult("Please pass a name on the query string or in the request body");
}
```

Вы можете выполнить привязку к пользовательскому объекту вместо `HttpRequest`. Этот объект создается из текста запроса и анализируется как JSON. Аналогичным образом тип можно передать в привязку вывода ответа HTTP. Результатом будет текст ответа с кодом состояния `200`.

```csharp
using System.Net;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

public static string Run(Person person, ILogger log)
{   
    return person.Name != null
        ? (ActionResult)new OkObjectResult($"Hello, {person.Name}")
        : new BadRequestObjectResult("Please pass an instance of Person.");
}

public class Person {
     public string Name {get; set;}
}
```

# <a name="java"></a>[Java](#tab/java)

* [Чтение параметра из строки запроса](#read-parameter-from-the-query-string)
* [Чтение текста из запроса POST](#read-body-from-a-post-request)
* [Чтение параметра из маршрута](#read-parameter-from-a-route)
* [Чтение текста POJO из запроса POST](#read-pojo-body-from-a-post-request)

Этот тип привязки HTTP представлен в следующих примерах.

#### <a name="read-parameter-from-the-query-string"></a>Чтение параметра из строки запроса

Этот пример считывает параметр с именем `id` из строки запроса и использует его, чтобы создать документ JSON, возвращаемый клиенту, с типом содержимого `application/json`.

```java
@FunctionName("TriggerStringGet")
public HttpResponseMessage run(
        @HttpTrigger(name = "req", 
            methods = {HttpMethod.GET}, 
            authLevel = AuthorizationLevel.ANONYMOUS)
        HttpRequestMessage<Optional<String>> request,
        final ExecutionContext context) {
    
    // Item list
    context.getLogger().info("GET parameters are: " + request.getQueryParameters());

    // Get named parameter
    String id = request.getQueryParameters().getOrDefault("id", "");

    // Convert and display
    if (id.isEmpty()) {
        return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                        .body("Document not found.")
                        .build();
    } 
    else {
        // return JSON from to the client
        // Generate document
        final String name = "fake_name";
        final String jsonDocument = "{\"id\":\"" + id + "\", " + 
                                        "\"description\": \"" + name + "\"}";
        return request.createResponseBuilder(HttpStatus.OK)
                        .header("Content-Type", "application/json")
                        .body(jsonDocument)
                        .build();
    }
}
```

#### <a name="read-body-from-a-post-request"></a>Чтение текста из запроса POST

Этот пример считывает текст из запроса POST как `String` и использует его, чтобы создать документ JSON, возвращаемый клиенту, с типом содержимого `application/json`.

```java
    @FunctionName("TriggerStringPost")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", 
              methods = {HttpMethod.POST}, 
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {
        
        // Item list
        context.getLogger().info("Request body is: " + request.getBody().orElse(""));

        // Check request body
        if (!request.getBody().isPresent()) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        } 
        else {
            // return JSON from to the client
            // Generate document
            final String body = request.getBody().get();
            final String jsonDocument = "{\"id\":\"123456\", " + 
                                         "\"description\": \"" + body + "\"}";
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(jsonDocument)
                          .build();
        }
    }
```

#### <a name="read-parameter-from-a-route"></a>Чтение параметра из маршрута

Этот пример считывает обязательный параметр с именем `id` и необязательный параметр `name` из пути маршрута и использует их для создания документа JSON, возвращаемого клиенту, с типом содержимого `application/json`. T

```java
@FunctionName("TriggerStringRoute")
public HttpResponseMessage run(
        @HttpTrigger(name = "req", 
            methods = {HttpMethod.GET}, 
            authLevel = AuthorizationLevel.ANONYMOUS,
            route = "trigger/{id}/{name=EMPTY}") // name is optional and defaults to EMPTY
        HttpRequestMessage<Optional<String>> request,
        @BindingName("id") String id,
        @BindingName("name") String name,
        final ExecutionContext context) {
    
    // Item list
    context.getLogger().info("Route parameters are: " + id);

    // Convert and display
    if (id == null) {
        return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                        .body("Document not found.")
                        .build();
    } 
    else {
        // return JSON from to the client
        // Generate document
        final String jsonDocument = "{\"id\":\"" + id + "\", " + 
                                        "\"description\": \"" + name + "\"}";
        return request.createResponseBuilder(HttpStatus.OK)
                        .header("Content-Type", "application/json")
                        .body(jsonDocument)
                        .build();
    }
}
```

#### <a name="read-pojo-body-from-a-post-request"></a>Чтение текста POJO из запроса POST

Ниже приведен код для класса `ToDoItem`, указанного в этом примере:

```java

public class ToDoItem {

  private String id;
  private String description;  

  public ToDoItem(String id, String description) {
    this.id = id;
    this.description = description;
  }

  public String getId() {
    return id;
  }

  public String getDescription() {
    return description;
  }
  
  @Override
  public String toString() {
    return "ToDoItem={id=" + id + ",description=" + description + "}";
  }
}

```

Этот пример считывает текст запроса POST. Текст запроса автоматически десериализуется в объект `ToDoItem` и возвращается к клиенту с типом содержимого `application/json`. Параметр `ToDoItem` сериализуется средой выполнения службы "Функции", так как он назначен свойству `body` класса `HttpMessageResponse.Builder`.

```java
@FunctionName("TriggerPojoPost")
public HttpResponseMessage run(
        @HttpTrigger(name = "req", 
            methods = {HttpMethod.POST}, 
            authLevel = AuthorizationLevel.ANONYMOUS)
        HttpRequestMessage<Optional<ToDoItem>> request,
        final ExecutionContext context) {
    
    // Item list
    context.getLogger().info("Request body is: " + request.getBody().orElse(null));

    // Check request body
    if (!request.getBody().isPresent()) {
        return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                        .body("Document not found.")
                        .build();
    } 
    else {
        // return JSON from to the client
        // Generate document
        final ToDoItem body = request.getBody().get();
        return request.createResponseBuilder(HttpStatus.OK)
                        .header("Content-Type", "application/json")
                        .body(body)
                        .build();
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В следующем примере показана привязка триггера в файле *function.json* и [функция JavaScript](functions-reference-node.md), которая использует привязку. В функции выполняется поиск параметра `name` в строке запроса или в тексте HTTP-запроса.

Ниже показан файл *function.json*.

```json
{
    "disabled": false,    
    "bindings": [
        {
            "authLevel": "function",
            "type": "httpTrigger",
            "direction": "in",
            "name": "req"
        },
        {
            "type": "http",
            "direction": "out",
            "name": "res"
        }
    ]
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

Ниже показан код JavaScript.

```javascript
module.exports = function(context, req) {
    context.log('Node.js HTTP trigger function processed a request. RequestUri=%s', req.originalUrl);

    if (req.query.name || (req.body && req.body.name)) {
        context.res = {
            // status defaults to 200 */
            body: "Hello " + (req.query.name || req.body.name)
        };
    }
    else {
        context.res = {
            status: 400,
            body: "Please pass a name on the query string or in the request body"
        };
    }
    context.done();
};
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В следующем примере показана привязка триггера в *function.jsдля* файла и [функция PowerShell](functions-reference-node.md). В функции выполняется поиск параметра `name` в строке запроса или в тексте HTTP-запроса.

```json
{
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "Request",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "Response"
    }
  ]
}
```

```powershell
using namespace System.Net

# Input bindings are passed in via param block.
param($Request, $TriggerMetadata)

# Write to the Azure Functions log stream.
Write-Host "PowerShell HTTP trigger function processed a request."

# Interact with query parameters or the body of the request.
$name = $Request.Query.Name
if (-not $name) {
    $name = $Request.Body.Name
}

$body = "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response."

if ($name) {
    $body = "Hello, $name. This HTTP triggered function executed successfully."
}

# Associate values to output bindings by calling 'Push-OutputBinding'.
Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = [HttpStatusCode]::OK
    Body       = $body
})
```


# <a name="python"></a>[Python](#tab/python)

В следующем примере показаны привязка триггера в файле *function.json* и [функция Python](functions-reference-python.md), использующая эту привязку. В функции выполняется поиск параметра `name` в строке запроса или в тексте HTTP-запроса.

Ниже показан файл *function.json*.

```json
{
    "scriptFile": "__init__.py",
    "disabled": false,    
    "bindings": [
        {
            "authLevel": "function",
            "type": "httpTrigger",
            "direction": "in",
            "name": "req"
        },
        {
            "type": "http",
            "direction": "out",
            "name": "$return"
        }
    ]
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

Ниже приведен код Python.

```python
import logging
import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello {name}!")
    else:
        return func.HttpResponse(
            "Please pass a name on the query string or in the request body",
            status_code=400
        )
```

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

В [библиотеках классов для C#](functions-dotnet-class-library.md) и Java для настройки функции доступен атрибут `HttpTrigger`.

В параметрах конструктора атрибута можно задать уровень авторизации и допустимые методы HTTP, а также тип веб-перехватчика и шаблон маршрута. Дополнительные сведения об этих параметрах конфигурации см. [здесь](#configuration).

# <a name="c"></a>[C#](#tab/csharp)

В следующем примере кода показано, как использовать атрибут [HttpTrigger](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/dev/src/WebJobs.Extensions.Http/HttpTriggerAttribute.cs).

```csharp
[FunctionName("HttpTriggerCSharp")]
public static Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous)] HttpRequest req)
{
    ...
}
```

Полный пример триггера см. [здесь](#example).

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

В следующем примере кода показано, как использовать атрибут [HttpTrigger](https://github.com/Azure/azure-functions-java-library/blob/dev/src/main/java/com/microsoft/azure/functions/annotation/HttpTrigger.java).

```java
@FunctionName("HttpTriggerJava")
public HttpResponseMessage<String> HttpTrigger(
        @HttpTrigger(name = "req",
                     methods = {"get"},
                     authLevel = AuthorizationLevel.ANONYMOUS) HttpRequestMessage<String> request,
        final ExecutionContext context) {

    ...
}
```

Полный пример триггера см. [здесь](#example).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В JavaScript атрибуты не поддерживаются.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В PowerShell не поддерживаются атрибуты.

# <a name="python"></a>[Python](#tab/python)

В Python атрибуты не поддерживаются.

---

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `HttpTrigger`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
| **type** | Недоступно| Обязательное. Необходимо задать значение `httpTrigger`. |
| **direction** | Недоступно| Обязательное. Необходимо задать значение `in`. |
| **name** | Недоступно| Обязательное. Имя переменной, из которой в коде функции можно получить запрос или текст запроса. |
| <a name="http-auth"></a>**authLevel** |  **AuthLevel** |Определяет, какие ключи (если они требуются) должны присутствовать в запросе, чтобы вызвать функцию. Уровень авторизации может принимать одно из следующих значений: <ul><li><code>anonymous</code>&mdash; — ключи API не требуются.</li><li><code>function</code>&mdash; — требуется ключ API для конкретной функции. Это значение используется по умолчанию, если не указано иное.</li><li><code>admin</code>&mdash; — требуется главный ключ.</li></ul> Дополнительные сведения см. в разделе [Ключи авторизации](#authorization-keys). |
| **methods** |**Методы** | Массив методов HTTP, на которые отвечает функция. Если свойство не указано, функция отвечает на все методы HTTP. См. раздел [Настройка конечной точки HTTP](#customize-the-http-endpoint). |
| **route** | **Route** | Шаблон маршрута, определяющий URL-адреса запросов, на которые отвечает функция. Если значение не указано, по умолчанию используется `<functionname>`. См. раздел [Настройка конечной точки HTTP](#customize-the-http-endpoint). |
| **webHookType** | **WebHookType** | _Поддерживается только для среды выполнения версии 1.x._<br/><br/>Указывает, что триггер HTTP должен выступать в качестве получателя [веб-перехватчика](https://en.wikipedia.org/wiki/Webhook) для указанного поставщика. Если вы установите это свойство, не устанавливайте свойство `methods`. Тип веб-перехватчика может принимать одно из следующих значений:<ul><li><code>genericJson</code>&mdash; — конечная точка веб-перехватчика общего назначения без логики для конкретного поставщика. Этот параметр определяет, что принимаются только запросы HTTP POST с содержимым типа `application/json`.</li><li><code>github</code>&mdash; — функция отвечает на вызовы [веб-перехватчиков GitHub](https://developer.github.com/webhooks/). Не используйте свойство _authLevel_ вместе с веб-перехватчиками GitHub. Дополнительные сведения см. в этой статье, в разделе веб-перехватчики GitHub.</li><li><code>slack</code>&mdash; — функция отвечает на вызовы [веб-перехватчиков Slack](https://api.slack.com/outgoing-webhooks). Не используйте свойство _authLevel_ вместе с веб-перехватчиками Slack. Дополнительные сведения см. в разделе о веб-перехватчиках Slack далее в этой статье.</li></ul>|

## <a name="payload"></a>Полезные данные

Тип входных данных триггера объявлен как `HttpRequest` или пользовательский тип. Если вы выберете `HttpRequest`, то получите полный доступ к объекту запроса. При объявлении пользовательского типа среда выполнения попытается проанализировать текст запроса JSON для задания свойств объекта.

## <a name="customize-the-http-endpoint"></a>Настройка конечной точки HTTP

По умолчанию при создании функции для триггера HTTP маршрут для ее адресации имеет следующий вид:

```http
http://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>
```

Этот маршрут можно настроить с помощью дополнительного свойства `route` привязки для вывода триггера HTTP. Например, приведенный ниже файл *function.json* определяет свойство `route` для триггера HTTP.

```json
{
    "bindings": [
    {
        "type": "httpTrigger",
        "name": "req",
        "direction": "in",
        "methods": [ "get" ],
        "route": "products/{category:alpha}/{id:int?}"
    },
    {
        "type": "http",
        "name": "res",
        "direction": "out"
    }
    ]
}
```

При такой конфигурации функция доступна по приведенному ниже маршруту вместо исходного.

```
http://<APP_NAME>.azurewebsites.net/api/products/electronics/357
```

Эта конфигурация позволяет коду функции поддерживать два параметра в адресе, _категории_ и _идентификаторе_. Дополнительные сведения о том, как параметры маршрутов размечены в URL-адресе, см. в разделе [Маршрутизация в ASP.NET Core](/aspnet/core/fundamentals/routing#route-constraint-reference).

# <a name="c"></a>[C#](#tab/csharp)

Для параметров можно использовать любое [ограничение маршрута веб-API](https://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2#constraints). Приведенный ниже код функции C# используют оба параметра.

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;

public static IActionResult Run(HttpRequest req, string category, int? id, ILogger log)
{
    var message = String.Format($"Category: {category}, ID: {id}");
    return (ActionResult)new OkObjectResult(message);
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

Для параметров можно использовать любое [ограничение маршрута веб-API](https://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2#constraints). Приведенный ниже код функции C# используют оба параметра.

```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;

public static IActionResult Run(HttpRequest req, string category, int? id, ILogger log)
{
    var message = String.Format($"Category: {category}, ID: {id}");
    return (ActionResult)new OkObjectResult(message);
}
```

# <a name="java"></a>[Java](#tab/java)

Контекст выполнения функции включает свойства, как объявлено в атрибуте `HttpTrigger`. Этот атрибут позволяет определять параметры маршрута, уровни авторизации, HTTP-команды и экземпляр входящего запроса.

Параметры маршрута определяются в атрибуте `HttpTrigger`.

```java
package com.function;

import java.util.*;
import com.microsoft.azure.functions.annotation.*;
import com.microsoft.azure.functions.*;

public class HttpTriggerJava {
    public HttpResponseMessage<String> HttpTrigger(
            @HttpTrigger(name = "req",
                         methods = {"get"},
                         authLevel = AuthorizationLevel.FUNCTION,
                         route = "products/{category:alpha}/{id:int}") HttpRequestMessage<String> request,
            @BindingName("category") String category,
            @BindingName("id") int id,
            final ExecutionContext context) {

        String message = String.format("Category  %s, ID: %d", category, id);
        return request.createResponseBuilder(HttpStatus.OK).body(message).build();
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В Node среда выполнения Функций предоставляет текст запроса из объекта `context`. Дополнительные сведения см. в разделе [Пример JavaScript в триггере](#example).

В следующем примере показано, как считывать параметры маршрута из `context.bindingData`.

```javascript
module.exports = function (context, req) {

    var category = context.bindingData.category;
    var id = context.bindingData.id;
    var message = `Category: ${category}, ID: ${id}`;

    context.res = {
        body: message;
    }

    context.done();
}
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Параметры маршрута, объявленные в файле *function.json* , доступны как свойство `$Request.Params` объекта.

```powershell
$Category = $Request.Params.category
$Id = $Request.Params.id

$Message = "Category:" + $Category + ", ID: " + $Id

Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = [HttpStatusCode]::OK
    Body = $Message
})
```

# <a name="python"></a>[Python](#tab/python)

Контекст выполнения функции предоставляется с помощью параметра, объявленного как `func.HttpRequest`. Этот экземпляр позволяет функции обращаться к параметрам маршрута данных, значениям строк запросов и методам, которые позволяют возвращать HTTP-ответы.

После определения параметры маршрута становятся доступными в функции путем вызова метода `route_params`.

```python
import logging

import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:

    category = req.route_params.get('category')
    id = req.route_params.get('id')
    message = f"Category: {category}, ID: {id}"

    return func.HttpResponse(message)
```

---

По умолчанию все маршруты функций начинаются с префикса *api*. Вы можете настроить или удалить этот префикс с помощью свойства `extensions.http.routePrefix` в файле [host.json](functions-host-json.md). В следующем примере префикс маршрута *api* удаляется. Он заменяется пустой строкой в файле *host.json*.

```json
{
    "extensions": {
        "http": {
            "routePrefix": ""
        }
    }
}
```

## <a name="using-route-parameters"></a>Использование параметров маршрута

Каждой привязке доступны параметры маршрута, которые определяют шаблон `route` для функции. Например, если маршрут определен как `"route": "products/{id}"`, привязка к хранилищу таблиц может использовать в конфигурации привязки значение параметра `{id}`.

В следующей конфигурации показано, как параметр `{id}` передается в `rowKey` привязки.

```json
{
    "type": "table",
    "direction": "in",
    "name": "product",
    "partitionKey": "products",
    "tableName": "products",
    "rowKey": "{id}"
}
```

При использовании параметров маршрута `invoke_URL_template` автоматически создается для функции. Клиенты могут использовать шаблон URL-адреса, чтобы понять, какие параметры необходимы для передачи URL-адреса при вызове функции с помощью ее URL-адреса. Перейдите к одной из функций, активируемых с помощью HTTP, в [портал Azure](https://portal.azure.com) и выберите **получить URL-адрес функции**.

Доступ к можно получить программным `invoke_URL_template` путем с помощью Azure Resource Manager API-интерфейсов для [функций списка](/rest/api/appservice/webapps/listfunctions) или [функции Get](/rest/api/appservice/webapps/getfunction).

## <a name="working-with-client-identities"></a>Работа с удостоверениями клиентов

Если в приложении-функции используется [аутентификация и авторизация Службы приложений](../app-service/overview-authentication-authorization.md), сведения об аутентифицированных клиентах можно посмотреть прямо в коде. Эта информация доступна в виде [заголовков запросов, которые вставляет платформа](../app-service/app-service-authentication-how-to.md#access-user-claims).

Эти сведения также можно считывать из данных привязки. Эта возможность доступна только в среде выполнения Функций 2.x и более поздних версий. Кроме того, сейчас она доступна только для языков .NET.

# <a name="c"></a>[C#](#tab/csharp)

Информация о прошедших проверку клиентах доступна как класс [ClaimsPrincipal](/dotnet/api/system.security.claims.claimsprincipal). ClaimsPrincipal доступен как часть контекста запроса, как показано в следующем примере:

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

public static IActionResult Run(HttpRequest req, ILogger log)
{
    ClaimsPrincipal identities = req.HttpContext.User;
    // ...
    return new OkObjectResult();
}
```

Кроме того, ClaimsPrincipal можно включить как дополнительный параметр в сигнатуру функции:

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;
using Newtonsoft.Json.Linq;

public static void Run(JObject input, ClaimsPrincipal principal, ILogger log)
{
    // ...
    return;
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

Информация о прошедших проверку клиентах доступна как класс [ClaimsPrincipal](/dotnet/api/system.security.claims.claimsprincipal). ClaimsPrincipal доступен как часть контекста запроса, как показано в следующем примере:

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

public static IActionResult Run(HttpRequest req, ILogger log)
{
    ClaimsPrincipal identities = req.HttpContext.User;
    // ...
    return new OkObjectResult();
}
```

Кроме того, ClaimsPrincipal можно включить как дополнительный параметр в сигнатуру функции:

```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;
using Newtonsoft.Json.Linq;

public static void Run(JObject input, ClaimsPrincipal principal, ILogger log)
{
    // ...
    return;
}
```

# <a name="java"></a>[Java](#tab/java)

Прошедший проверку подлинности пользователь доступен через [заголовки HTTP](../app-service/app-service-authentication-how-to.md#access-user-claims).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Прошедший проверку подлинности пользователь доступен через [заголовки HTTP](../app-service/app-service-authentication-how-to.md#access-user-claims).

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Прошедший проверку подлинности пользователь доступен через [заголовки HTTP](../app-service/app-service-authentication-how-to.md#access-user-claims).

# <a name="python"></a>[Python](#tab/python)

Прошедший проверку подлинности пользователь доступен через [заголовки HTTP](../app-service/app-service-authentication-how-to.md#access-user-claims).


---

## <a name="function-access-keys"></a><a name="authorization-keys"></a>Ключи доступа к функции

[!INCLUDE [functions-authorization-keys](../../includes/functions-authorization-keys.md)]

## <a name="obtaining-keys"></a>Получение ключей

Ключи хранятся в Azure в составе приложения-функции в зашифрованном виде. Чтобы просмотреть ключи, создать новые или переместить ключи в новые значения, перейдите к одной из функций, активируемых с помощью HTTP, в [портал Azure](https://portal.azure.com) и выберите **функции ключи**.

Вы также можете управлять ключами узлов. Перейдите к приложению функции в [портал Azure](https://portal.azure.com) и выберите **ключи приложения**.

Ключи функций и узлов можно получить программным путем с помощью Azure Resource Manager API. Существуют API-интерфейсы для [перечисления ключей функций](/rest/api/appservice/webapps/listfunctionkeys) и [вывода списка ключей узлов](/rest/api/appservice/webapps/listhostkeys), а также при использовании слотов развертывания эквивалентные API-интерфейсы перечисляются в [области ключей функций](/rest/api/appservice/webapps/listfunctionkeysslot) и выводят [список ключей узла](/rest/api/appservice/webapps/listhostkeysslot).

Вы также можете создать новые функции и ключи узлов программным путем с помощью команды [создать или обновить секрет функции](/rest/api/appservice/webapps/createorupdatefunctionsecret), [создать](/rest/api/appservice/webapps/createorupdatefunctionsecretslot)или обновить секретный ключ функции, [создать или обновить секрет узла](/rest/api/appservice/webapps/createorupdatehostsecret) и [создать или обновить ключевые API слота узла](/rest/api/appservice/webapps/createorupdatehostsecretslot) .

Ключи функций и узлов можно удалить программно с помощью [функции удалить секретный](/rest/api/appservice/webapps/deletefunctionsecret)ключ, удалить секретный [сегмент функции](/rest/api/appservice/webapps/deletefunctionsecretslot), [Удалить секрет узла](/rest/api/appservice/webapps/deletehostsecret)и [Удалить API секретного слота узла](/rest/api/appservice/webapps/deletehostsecretslot) .

Вы также можете использовать [устаревшие API управления ключами для получения ключей функций](https://github.com/Azure/azure-functions-host/wiki/Key-management-API), но вместо этого рекомендуется использовать Azure Resource Manager API.

## <a name="api-key-authorization"></a>Проверка подлинности с помощью ключа API

Большинству шаблонов триггеров HTTP требуется ключ API в запросе. Поэтому HTTP-запрос обычно выглядит как следующий URL-адрес:

```http
https://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>?code=<API_KEY>
```

Ключ можно передать в переменной строки запроса с именем `code`, как показано выше. Его также можно включить в заголовок HTTP `x-functions-key`. Значением ключа может быть любой ключ функции, определенный для запрашиваемой функции, или любой ключ узла.

Вы можете разрешить анонимные запросы, не требующие наличие ключей. Вы также можете указать, что используется главный ключ. Уровень авторизации по умолчанию можно изменить с помощью свойства `authLevel` в объекте JSON для привязки. Дополнительные сведения см. в разделе [Конфигурация триггера](#configuration).

> [!NOTE]
> При локальном выполнении функций авторизация всегда отключается, независимо от указанного параметра уровня аутентификации. После публикации в Azure в триггере применяется параметр `authLevel`. Ключи по-прежнему требуются при [локальном выполнении в контейнере](functions-create-function-linux-custom-image.md#build-the-container-image-and-test-locally).


## <a name="secure-an-http-endpoint-in-production"></a>Защита конечной точки HTTP в рабочей среде

Чтобы полностью защитить функции конечных точек в рабочей среде, реализуйте один из следующих механизмов безопасности на уровне приложения-функции. При использовании одного из этих методов обеспечения безопасности на уровне приложения-функции необходимо задать для функции, активируемой по HTTP, уровень проверки подлинности `anonymous`.

[!INCLUDE [functions-enable-auth](../../includes/functions-enable-auth.md)]

#### <a name="deploy-your-function-app-in-isolation"></a>Развертывание приложения-функции в изолированной среде

[!INCLUDE [functions-deploy-isolation](../../includes/functions-deploy-isolation.md)]

## <a name="webhooks"></a>Веб-перехватчики

> [!NOTE]
> Режим веб-перехватчика доступен только для версии 1.x среды выполнения функций. Это изменение внесено для повышения производительности триггеров HTTP в версии 2.x и более поздних версиях.

В версии 1.x шаблоны веб-перехватчика обеспечивают дополнительную проверку полезных данных веб-перехватчика. В версии 2.x и более поздних версиях базовый триггер HTTP также работает, и мы рекомендуем использовать именно его для реализации веб-перехватчиков. 

### <a name="github-webhooks"></a>Веб-перехватчики GitHub

Чтобы настроить ответ на вызовы веб-перехватчика GitHub, прежде всего создайте функцию с триггером HTTP, для которого свойство **webHookType** будет иметь значение `github`. Затем скопируйте его URL-адрес и ключ API в репозиторий GitHub, используя страницу **Добавить веб-перехватчик**. 

![Снимок экрана, показывающий, как добавить веб-перехватчик для функции.](./media/functions-bindings-http-webhook/github-add-webhook.png)

### <a name="slack-webhooks"></a>Веб-перехватчики Slack

Webhook Slack создает маркер автоматически и не позволяет вам задать его, поэтому необходимо настроить ключ для конкретной функции с использованием маркера, предоставленного Slack. Ознакомьтесь с разделом [Ключи авторизации](#authorization-keys).

## <a name="webhooks-and-keys"></a>Веб-перехватчики и ключи

Авторизация веб-перехватчика обрабатывается компонентом получателя веб-перехватчика, который входит в состав триггера HTTP. Механизм обработки различается в зависимости от типа веб-перехватчика. Каждый из этих механизмов зависит от ключей. По умолчанию используется ключ функции с именем default. Чтобы использовать другой ключ, необходимо настроить поставщик веб-перехватчика так, чтобы он отправлял имя ключа в составе запроса одним из следующих способов.

* **Строка запроса:** Поставщик передает имя ключа в параметре `clientid` строки запроса, например `https://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>?clientid=<KEY_NAME>`.
* **Заголовок запроса.** Поставщик передает имя ключа в заголовке `x-functions-clientid`.

## <a name="content-types"></a>Типы содержимого

Передача двоичных данных и форм в функцию, не относящуюся к C #, требует использования соответствующего заголовка Content-Type. Поддерживаемые типы содержимого включают `octet-stream` в себя двоичные и [Составные типы](https://www.iana.org/assignments/media-types/media-types.xhtml#multipart)данных.

### <a name="known-issues"></a>Известные проблемы

В функциях, отличных от C #, запросы, отправленные с типом содержимого, `image/jpeg` приводят `string` к значению, переданному в функцию. В таких случаях можно вручную преобразовать `string` значение в массив байтов для доступа к необработанным двоичным данным.

## <a name="limits"></a>Ограничения

Длина HTTP-запроса ограничена 100 МБ (104 857 600 байт), а длина URL-адреса — 4 КБ (4096 байт). Эти ограничения задает элемент `httpRuntime`[файла Web.config](https://github.com/Azure/azure-functions-host/blob/v3.x/src/WebJobs.Script.WebHost/web.config) среды выполнения.

Если функция, которая использует триггер HTTP, выполняется дольше 230 секунд, допустимое время ожидания в [Azure Load Balancer](../app-service/faq-availability-performance-application-issues.md#why-does-my-request-time-out-after-230-seconds) истекает и возвращается ошибка HTTP 502. Функция продолжит работу, но вернуть ответ HTTP будет невозможно. Для долго выполняющихся функций рекомендуем следовать асинхронным шаблонам и возвращать расположение, в котором можно проверить состояние запроса. Чтобы узнать, как долго может выполняться функция, см. раздел [Масштабирование и размещение — план](functions-scale.md#timeout).


## <a name="next-steps"></a>Дальнейшие действия

- [Выходные привязки HTTP для Функций Azure](./functions-bindings-http-webhook-output.md)