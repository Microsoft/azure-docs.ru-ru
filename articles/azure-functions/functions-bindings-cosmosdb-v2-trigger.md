---
title: Триггер Azure Cosmos DB для функций 2. x и более поздних версий
description: Узнайте, как использовать триггер Azure Cosmos DB в функциях Azure.
author: craigshoemaker
ms.topic: reference
ms.date: 02/24/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 6f4e43efeb1882f52bd335d83a3660a94040ab8a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101729221"
---
# <a name="azure-cosmos-db-trigger-for-azure-functions-2x-and-higher"></a>Триггер Azure Cosmos DB для функций Azure 2. x и более поздних версий

Триггер Azure Cosmos DB использует [канал изменений Azure Cosmos DB](../cosmos-db/change-feed.md) для ожидания вставок и обновлений в разных разделах. На канале изменений публикуются вставки и обновления, а не удаления.

Сведения об установке и настройке см. в [этой обзорной статье](./functions-bindings-cosmosdb-v2.md).

<a id="example" name="example"></a>

# <a name="c"></a>[C#](#tab/csharp)

В следующем примере показана [функция C#](functions-dotnet-class-library.md), вызываемая при вставке или обновлении в указанной базе данных и коллекции.

```cs
using Microsoft.Azure.Documents;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class CosmosTrigger
    {
        [FunctionName("CosmosTrigger")]
        public static void Run([CosmosDBTrigger(
            databaseName: "ToDoItems",
            collectionName: "Items",
            ConnectionStringSetting = "CosmosDBConnection",
            LeaseCollectionName = "leases",
            CreateLeaseCollectionIfNotExists = true)]IReadOnlyList<Document> documents,
            ILogger log)
        {
            if (documents != null && documents.Count > 0)
            {
                log.LogInformation($"Documents modified: {documents.Count}");
                log.LogInformation($"First document Id: {documents[0].Id}");
            }
        }
    }
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В следующем примере показаны привязка триггера Cosmos DB в файле *function.json* и [функция сценария C#](functions-reference-csharp.md), которая использует эту привязку. Функция записывает сообщения журнала при добавлении или изменении записей Cosmos DB.

Данные привязки в файле *function.json*:

```json
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "<connection-app-setting>",
    "databaseName": "Tasks",
    "collectionName": "Items",
    "createLeaseCollectionIfNotExists": true
}
```

Ниже приведен код скрипта C#.

```cs
    #r "Microsoft.Azure.DocumentDB.Core"

    using System;
    using Microsoft.Azure.Documents;
    using System.Collections.Generic;
    using Microsoft.Extensions.Logging;

    public static void Run(IReadOnlyList<Document> documents, ILogger log)
    {
      log.LogInformation("Documents modified " + documents.Count);
      log.LogInformation("First document Id " + documents[0].Id);
    }
```

# <a name="java"></a>[Java](#tab/java)

Эта функция вызывается при наличии вставок или обновлений в указанной базе данных и коллекции.

```java
    @FunctionName("cosmosDBMonitor")
    public void cosmosDbProcessor(
        @CosmosDBTrigger(name = "items",
            databaseName = "ToDoList",
            collectionName = "Items",
            leaseCollectionName = "leases",
            createLeaseCollectionIfNotExists = true,
            connectionStringSetting = "AzureCosmosDBConnection") String[] items,
            final ExecutionContext context ) {
                context.getLogger().info(items.length + "item(s) is/are changed.");
            }
```


В [библиотеке среды выполнения функций Java](/java/api/overview/azure/functions/runtime) используйте заметку `@CosmosDBTrigger` для параметров, значения которых будут поступать из Cosmos DB.  Эта заметка может использоваться с собственными типами Java, объектами POJO или значениями, допускающими значения NULL, используя `Optional<T>`.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В следующем примере показаны привязка триггера Cosmos DB в файле *function.json* и [функция JavaScript](functions-reference-node.md), которая использует эту привязку. Функция записывает сообщения журнала при добавлении или изменении записей Cosmos DB.

Данные привязки в файле *function.json*:

```json
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "<connection-app-setting>",
    "databaseName": "Tasks",
    "collectionName": "Items",
    "createLeaseCollectionIfNotExists": true
}
```

Ниже показан код JavaScript.

```javascript
    module.exports = function (context, documents) {
      context.log('First document Id modified : ', documents[0].id);

      context.done();
    }
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В следующем примере показано, как выполнить функцию при изменении данных в Cosmos DB.

```json
{
  "type": "cosmosDBTrigger",
  "name": "Documents",
  "direction": "in",
  "leaseCollectionName": "leases",
  "connectionStringSetting": "MyStorageConnectionAppSetting",
  "databaseName": "Tasks",
  "collectionName": "Items",
  "createLeaseCollectionIfNotExists": true
}
```

В файле _run.ps1_ есть доступ к документу, который запускает функцию через `$Documents` параметр.

```powershell
param($Documents, $TriggerMetadata) 

Write-Host "First document Id modified : $($Documents[0].id)" 
```

# <a name="python"></a>[Python](#tab/python)

В следующем примере показаны привязка триггера Cosmos DB в файле *function.json* и [функция Python](functions-reference-python.md), которая использует эту привязку. При изменении записей в Cosmos DB функция записывает сообщения в журнале.

Данные привязки в файле *function.json*:

```json
{
    "name": "documents",
    "type": "cosmosDBTrigger",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "<connection-app-setting>",
    "databaseName": "Tasks",
    "collectionName": "Items",
    "createLeaseCollectionIfNotExists": true
}
```

Ниже приведен код Python.

```python
    import logging
    import azure.functions as func


    def main(documents: func.DocumentList) -> str:
        if documents:
            logging.info('First document Id modified: %s', documents[0]['id'])
```

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

В [библиотеках классов C#](functions-dotnet-class-library.md) используйте атрибут [CosmosDBTrigger](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.CosmosDB/Trigger/CosmosDBTriggerAttribute.cs).

Конструктор атрибута принимает имя базы данных и имя коллекции. Дополнительные сведения о параметрах и других свойствах, которые можно настроить, см. в разделе [Конфигурация триггера](#configuration). Ниже приведен пример атрибута `CosmosDBTrigger` в сигнатуре метода:

```csharp
    [FunctionName("DocumentUpdates")]
    public static void Run([CosmosDBTrigger("database", "collection", ConnectionStringSetting = "myCosmosDB")]
        IReadOnlyList<Document> documents,
        ILogger log)
    {
        ...
    }
```

Полный пример см. в разделе [триггер](#example).

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

В [библиотеке времени выполнения функций Java](/java/api/overview/azure/functions/runtime)используйте `@CosmosDBInput` заметку для параметров, считывающих данные из Cosmos DB.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В JavaScript атрибуты не поддерживаются.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В PowerShell не поддерживаются атрибуты.

# <a name="python"></a>[Python](#tab/python)

В Python атрибуты не поддерживаются.

---

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `CosmosDBTrigger`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
|**type** | Недоступно | Нужно задать значение `cosmosDBTrigger`. |
|**direction** | Недоступно | Нужно задать значение `in`. Этот параметр задается автоматически при создании триггера на портале Azure. |
|**name** | Недоступно | Имя переменной, используемое в коде функции, представляющей список документов с изменениями. |
|**коннектионстрингсеттинг**|**ConnectionStringSetting** | Имя параметра приложения, содержащего строку подключения, используемую для подключения к отслеживаемой учетной записи Azure Cosmos DB. |
|**databaseName**|**DatabaseName**  | Имя базы данных Azure Cosmos DB с отслеживаемой коллекцией. |
|**collectionName** |**CollectionName** | Имя отслеживаемой коллекции. |
|**леасеконнектионстрингсеттинг** | **LeaseConnectionStringSetting** | Используемых Имя параметра приложения, содержащего строку подключения к учетной записи Azure Cosmos DB, в которой находится сбор аренд. Если значение не задано, используется значение `connectionStringSetting`. Этот параметр задается автоматически при создании привязки на портале. Строка подключения для коллекции аренд должна иметь разрешения на запись.|
|**леаседатабасенаме** |**LeaseDatabaseName** | (Необязательно.) Имя базы данных, в которой содержится коллекция, используемая для хранения аренд. Если значение не задано, используется значение параметра `databaseName`. Этот параметр задается автоматически при создании привязки на портале. |
|**леасеколлектионнаме** | **LeaseCollectionName** | (Необязательно.) Имя коллекции, используемой для хранения аренд. Если значение не задано, используется значение `leases`. |
|**креателеасеколлектионифнотексистс** | **CreateLeaseCollectionIfNotExists** | (Необязательно.) Если задано значение `true`, коллекция аренд создается автоматически, если она не создана. Значение по умолчанию — `false`. |
|**leasesCollectionThroughput**| **леасесколлектионсраугхпут**| Используемых Определяет число единиц запроса, назначаемых при создании коллекции аренд. Этот параметр используется, только если `createLeaseCollectionIfNotExists` для задано значение `true` . Этот параметр задается автоматически при создании привязки с помощью портала.
|**леасеколлектионпрефикс**| **LeaseCollectionPrefix**| Используемых Если этот параметр задан, значение добавляется в качестве префикса в аренду, созданную в коллекции аренд для этой функции. Использование префикса позволяет двум отдельным функциям Azure совместно использовать одну и ту же коллекцию аренд, используя разные префиксы.
|**фидполлделай**| **FeedPollDelay**| Используемых Время (в миллисекундах) задержки между опросом секции для новых изменений в веб-канале после того, как все текущие изменения будут остановлены. Значение по умолчанию — 5 000 миллисекунд или 5 секунд.
|**леасеаккуиреинтервал**| **LeaseAcquireInterval**| (Дополнительно) Если параметр задан, то он определяет интервал для запуска задачи вычисления при условии равномерности распределения секций между известными экземплярами узла (в миллисекундах). По умолчанию это 13000 (13 секунд).
|**леасикспиратионинтервал**| **LeaseExpirationInterval**| (Дополнительно) Если параметр задан, то он определяет интервал, за который берется аренда, представляющая секцию (в миллисекундах). Если аренда не будет обновлена в течение этого интервала, это приведет к ее истечению и владение секцией перейдет к другому экземпляру. По умолчанию это 60000 (60 секунд).
|**леасереневинтервал**| **LeaseRenewInterval**| (Дополнительно) Если параметр задан, то он определяет интервал продления для всех аренд в разделах, которые на данный момент занятые экземплярами (в миллисекундах). По умолчанию это 17000 (17 секунд).
|**чеккпоинтфрекуенци**| **CheckpointFrequency**| (Дополнительно) Если параметр задан, то он определяет интервал между контрольными точками аренды (в миллисекундах). После каждого вызова функции всегда используется значение по умолчанию.
|**maxItemsPerInvocation**| **макситемсперинвокатион**| Используемых Если этот параметр задан, это свойство задает максимальное количество элементов, получаемых за вызов функции. Если операции в отслеживаемой коллекции выполняются с помощью хранимых процедур, [область транзакции](../cosmos-db/stored-procedures-triggers-udfs.md#transactions) сохраняется при считывании элементов из веб-канала изменений. В результате число полученных элементов может быть выше указанного значения, чтобы элементы, измененные одной транзакцией, возвращались как часть одного атомарного пакета.
|**стартфромбегиннинг**| **StartFromBeginning**| Используемых Этот параметр указывает триггеру считывать изменения с начала журнала изменений коллекции, а не начиная с текущего времени. Чтение с начала работает только при первом запуске триггера, как при последующих запусках контрольные точки уже сохранены. Установка этого параметра в `true` случае, если уже созданы аренды, не оказывает никакого влияния. |
|**preferredLocations**| **PreferredLocations**| Используемых Определяет предпочтительные расположения (регионы) для геореплицированных учетных записей базы данных в службе Azure Cosmos DB. Значения должны быть разделены запятыми. Например, "Восточная часть США, Юго-Центральный регион США, Северная Европа". |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Использование

Триггеру требуется вторая коллекция, которая используется для хранения _аренды_ в разделах. Отслеживаемая коллекция и та, в которой содержатся аренды, должны быть доступны для успешной работы триггера.

>[!IMPORTANT]
> Если несколько функций настроены на использование триггера Cosmos DB для одной и той же коллекции, то каждая из функций должна использовать выделенную коллекцию аренды или указывать на другой префикс `LeaseCollectionPrefix` для каждой функции. В противном случае активируется только одна из функций. Сведения о префиксе см. в разделе с [конфигурацией](#configuration).

Триггер не указывает, был ли документ обновлен или вставлен. Вместо этого он представляет сам документ. Если операции обновления и вставки необходимо обрабатывать по-разному, это можно сделать, внедрив поля меток времени для операций обновления и вставки.

## <a name="next-steps"></a>Дальнейшие действия

- [Чтение Azure Cosmos DB документа (входная привязка)](./functions-bindings-cosmosdb-v2-input.md)
- [Сохранение изменений в документе Azure Cosmos DB (Выходная привязка)](./functions-bindings-cosmosdb-v2-output.md)
