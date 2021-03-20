---
title: Привязки мобильных приложений для службы "Функции Azure"
description: Узнайте, как использовать привязки мобильных приложений Azure в функциях Azure.
author: craigshoemaker
ms.topic: reference
ms.custom: devx-track-csharp
ms.date: 11/21/2017
ms.author: cshoe
ms.openlocfilehash: 5ea58cc3d9f3615a74249b36f3f9ffb79caddda1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88212235"
---
# <a name="mobile-apps-bindings-for-azure-functions"></a>Привязки мобильных приложений для службы "Функции Azure" 

> [!NOTE]
> Привязки мобильных приложений Azure доступны только для функций Azure версии 1.x. Они не поддерживаются в функциях Azure 2. x и более поздних версий.

В этой статье описывается использование привязок [мобильных приложений Azure](/previous-versions/azure/app-service-mobile/app-service-mobile-value-prop) в "Функциях Azure". Функции Azure поддерживают входные и выходные привязки для мобильных приложений.

Привязки мобильных приложений позволяют вам считывать и обновлять данные таблиц в мобильных приложениях.

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>Пакеты – Функции 1.x

Привязки мобильных приложений доступны в пакете NuGet [Microsoft.Azure.WebJobs.Extensions.MobileApps](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.MobileApps) версии 1.х. Исходный код для пакета находится в репозитории GitHub [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.MobileApps/).

[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="input"></a>Входные данные

Входная привязка мобильных приложений загружает запись из конечной точки мобильной таблицы и передает ее в функцию. В функциях C# и F# любые изменения, внесенные в запись, автоматически отправляются обратно в таблицу после успешного выхода из функции.

## <a name="input---example"></a>Пример входных данных

Языковой пример см. в разделах:

* [Скрипт C# (.csx)](#input---c-script-example)
* JavaScript

### <a name="input---c-script-example"></a>Пример входных данных сценария C#

В следующем примере показаны входная привязка мобильных приложений в файле *function.json* и [функция сценария C#](functions-reference-csharp.md), которая использует эту привязку. Функция активируется сообщением из очереди с идентификатором записи. Функция считывает указанную запись и изменяет ее свойство `Text`.

Данные привязки в файле *function.json*:

```json
{
"bindings": [
    {
        "name": "myQueueItem",
        "queueName": "myqueue-items",
        "connection": "",
        "type": "queueTrigger",
        "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "id": "{queueTrigger}",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "in"
    }
]
}
```
В разделе [Конфигурация](#input---configuration) описываются эти свойства.

Ниже приведен код скрипта C#.

```cs
#r "Newtonsoft.Json"    
using Newtonsoft.Json.Linq;

public static void Run(string myQueueItem, JObject record)
{
    if (record != null)
    {
        record["Text"] = "This has changed.";
    }    
}
```

### <a name="input---javascript"></a>Входные данные JavaScript

В следующем примере показаны входная привязка мобильных приложений в файле *function.json* и [функция JavaScript](functions-reference-node.md), которая использует эту привязку. Функция активируется сообщением из очереди с идентификатором записи. Функция считывает указанную запись и изменяет ее свойство `Text`.

Данные привязки в файле *function.json*:

```json
{
"bindings": [
    {
        "name": "myQueueItem",
        "queueName": "myqueue-items",
        "connection": "",
        "type": "queueTrigger",
        "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "id": "{queueTrigger}",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "in"
    }
]
}
```
В разделе [Конфигурация](#input---configuration) описываются эти свойства.

Ниже показан код JavaScript.

```javascript
module.exports = function (context, myQueueItem) {    
    context.log(context.bindings.record);
    context.done();
};
```

## <a name="input---attributes"></a>Входные атрибуты

В [библиотеках классов C#](functions-dotnet-class-library.md) используйте атрибут [MobileTable](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs).

Дополнительные сведения о настройке свойств атрибутов см. в следующем разделе о [настройке](#input---configuration).

## <a name="input---configuration"></a>Входная конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `MobileTable`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
| **type**| Недоступно | Для этого свойства необходимо задать значение mobileTable.|
| **direction**| Недоступно |Для этого свойства необходимо задать значение in|
| **name**| Недоступно | Имя входного параметра в сигнатуре функции.|
|**tableName** |**TableName**|Имя таблицы данных мобильного приложения|
| **id**| **Id** | Идентификатор извлекаемой записи. Может быть статическим или определяться по триггеру, который вызывает функцию. Например, если вы используете триггер очереди для функции, то `"id": "{queueTrigger}"` использует строковое значение сообщения очереди в качестве идентификатора записи, который нужно получить.|
|**connection**;|**Соединение**|Имя параметра приложения, в котором содержится URL-адрес мобильного приложения. Функция использует этот URL-адрес для создания необходимых операций REST с мобильным приложением. Создайте параметр приложения в приложении-функции, содержащем URL-адрес мобильного приложения, а затем укажите имя параметра приложения в свойстве `connection` во входной привязке. URL-адрес выглядит следующим образом: `http://<appname>.azurewebsites.net`.
|**apiKey**|**ApiKey**|Имя параметра приложения, в котором содержится ваш ключ API для мобильных приложений. При [внедрении ключа API в мобильное приложение Node.js](https://github.com/Azure/azure-mobile-apps-node/tree/master/samples/api-key)[или](https://github.com/Azure/azure-mobile-apps-net-server/wiki/Implementing-Application-Key) .NET предоставьте ключ API. Для этого создайте параметр приложения в приложении-функции, содержащем ключ API, а затем добавьте свойство `apiKey` в свою входную привязку с именем параметра приложения. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!IMPORTANT]
> Не используйте ключ API совместно с клиентами мобильных приложений. Его нужно безопасно распространять среди клиентов на стороне службы, таких как Функции Azure. Функции Azure хранят сведения о подключении и ключи API в качестве параметров приложения, чтобы они не возвращались в репозиторий системы управления версиями. Таким образом обеспечивается защита конфиденциальной информации.

## <a name="input---usage"></a>Использование входной привязки

В функциях C#, когда найденная запись з указанным идентификатором передается именованному параметру [JObject](https://www.newtonsoft.com/json/help/html/t_newtonsoft_json_linq_jobject.htm). Если запись не найдена, параметр имеет значение `null`. 

В функциях JavaScript запись передается объекту `context.bindings.<name>`. Если запись не найдена, параметр имеет значение `null`. 

В функциях C# и F# любые изменения, внесенные во входную запись (входной параметр), после успешного выхода из функции автоматически отправятся обратно в таблицу. Изменить запись в JavaScript невозможно.

## <a name="output"></a>Выходные данные

Используйте выходную привязку мобильных приложений, чтобы сделать новую запись в таблице мобильных приложений.  

## <a name="output---example"></a>Пример выходных данных

Языковой пример см. в разделах:

* [C#](#output---c-example)
* [Скрипт C# (.csx)](#output---c-script-example)
* [JavaScript](#output---javascript-example)

### <a name="output---c-example"></a>Пример выходных данных C#

В следующем примере показана [функция C#](functions-dotnet-class-library.md), которая активируется сообщением из очереди и создает запись в таблице мобильного приложения.

```csharp
[FunctionName("MobileAppsOutput")]        
[return: MobileTable(ApiKeySetting = "MyMobileAppKey", TableName = "MyTable", MobileAppUriSetting = "MyMobileAppUri")]
public static object Run(
    [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] string myQueueItem,
    TraceWriter log)
{
    return new { Text = $"I'm running in a C# function! {myQueueItem}" };
}
```

### <a name="output---c-script-example"></a>Пример выходных данных скрипта C#

В следующем примере показаны выходная привязка мобильных приложений в файле *function.json* и [функция сценария C#](functions-reference-csharp.md), которая использует эту привязку. Функция активируется сообщением из очереди и создает запись с жестко заданным значением для свойства `Text`.

Данные привязки в файле *function.json*:

```json
{
"bindings": [
    {
        "name": "myQueueItem",
        "queueName": "myqueue-items",
        "connection": "",
        "type": "queueTrigger",
        "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "out"
    }
]
}
```

В разделе [Конфигурация](#output---configuration) описываются эти свойства.

Ниже приведен код скрипта C#.

```cs
public static void Run(string myQueueItem, out object record)
{
    record = new {
        Text = $"I'm running in a C# function! {myQueueItem}"
    };
}
```

### <a name="output---javascript-example"></a>Пример выходных данных JavaScript

В следующем примере показаны выходная привязка мобильного приложения в файле *function.json* и [функция JavaScript](functions-reference-node.md), которая использует эту привязку. Функция активируется сообщением из очереди и создает запись с жестко заданным значением для свойства `Text`.

Данные привязки в файле *function.json*:

```json
{
"bindings": [
    {
        "name": "myQueueItem",
        "queueName": "myqueue-items",
        "connection": "",
        "type": "queueTrigger",
        "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "out"
    }
],
"disabled": false
}
```

В разделе [Конфигурация](#output---configuration) описываются эти свойства.

Ниже показан код JavaScript.

```javascript
module.exports = function (context, myQueueItem) {

    context.bindings.record = {
        text : "I'm running in a Node function! Data: '" + myQueueItem + "'"
    }   

    context.done();
};
```

## <a name="output---attributes"></a>Выходные атрибуты

В [библиотеках классов C#](functions-dotnet-class-library.md) используйте атрибут [MobileTable](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs).

Дополнительные сведения о настройке свойств атрибутов см. в разделе [Выходная конфигурация](#output---configuration). Ниже приведен пример атрибута `MobileTable` в сигнатуре метода:

```csharp
[FunctionName("MobileAppsOutput")]        
[return: MobileTable(ApiKeySetting = "MyMobileAppKey", TableName = "MyTable", MobileAppUriSetting = "MyMobileAppUri")]
public static object Run(
    [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] string myQueueItem,
    TraceWriter log)
{
    ...
}
```

Полный пример см. в разделе [Пример выходных данных C#](#output---c-example).

## <a name="output---configuration"></a>Выходная конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `MobileTable`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
| **type**| Недоступно | Для этого свойства необходимо задать значение mobileTable.|
| **direction**| Недоступно |Для этого свойства необходимо задать значение out.|
| **name**| Недоступно | Имя выходного параметра в сигнатуре функции.|
|**tableName** |**TableName**|Имя таблицы данных мобильного приложения|
|**connection**;|**MobileAppUriSetting**|Имя параметра приложения, в котором содержится URL-адрес мобильного приложения. Функция использует этот URL-адрес для создания необходимых операций REST с мобильным приложением. Создайте параметр приложения в приложении-функции, содержащем URL-адрес мобильного приложения, а затем укажите имя параметра приложения в свойстве `connection` во входной привязке. URL-адрес выглядит следующим образом: `http://<appname>.azurewebsites.net`.
|**apiKey**|**ApiKeySetting**|Имя параметра приложения, в котором содержится ваш ключ API для мобильных приложений. При [внедрении ключа API в серверную часть мобильного приложения Node.js](https://github.com/Azure/azure-mobile-apps-node/tree/master/samples/api-key) или [.NET предоставьте ключ API](https://github.com/Azure/azure-mobile-apps-net-server/wiki/Implementing-Application-Key). Для этого создайте параметр приложения в приложении-функции, содержащем ключ API, а затем добавьте свойство `apiKey` в свою входную привязку с именем параметра приложения. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!IMPORTANT]
> Не используйте ключ API совместно с клиентами мобильных приложений. Его нужно безопасно распространять среди клиентов на стороне службы, таких как Функции Azure. Функции Azure хранят сведения о подключении и ключи API в качестве параметров приложения, чтобы они не возвращались в репозиторий системы управления версиями. Таким образом обеспечивается защита конфиденциальной информации.

## <a name="output---usage"></a>Использование выходной привязки

Используйте именованный выходной параметр типа `out object` для доступа к выходной записи в функциях сценария C#. В библиотеках классов C# атрибут `MobileTable` можно использовать в любом из следующих типов:

* `ICollector<T>` или `IAsyncCollector<T>`, где `T` имеет значение `JObject` или является типом со свойством `public string Id`.
* `out JObject`
* `out T` или `out T[]`, где `T` является любим типом со свойством `public string Id`.

Для доступа к выходной записи в функциях Node.js используйте `context.bindings.<name>`.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Основные понятия триггеров и привязок в Функциях Azure](functions-triggers-bindings.md)
