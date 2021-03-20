---
title: Управления подключениями в службе "Функции Azure"
description: Узнайте, как избежать проблем с производительностью в службе "Функции Azure" с помощью статического подключения клиентов.
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 02/25/2018
ms.openlocfilehash: ec16ce3e7f9793be2a012a029bcca31c9a7ea4cf
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97936708"
---
# <a name="manage-connections-in-azure-functions"></a>Управления подключениями в службе "Функции Azure"

Функции в приложении-функции совместно используют ресурсы. К общим ресурсам относятся подключения: HTTP-подключения, подключения к базам данных и подключения к службам, таким как служба хранилища Azure. При параллельном выполнении многих функций можно остаться без доступных подключений. В этой статье объясняется, как закодировать функции, чтобы не использовать больше соединений, чем требуется.

## <a name="connection-limit"></a>Ограничение числа подключений

Количество доступных подключений ограничено частично, поскольку приложение-функция выполняется в [изолированной среде](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox). Одним из ограничений, налагаемых песочницей в коде, является ограничение числа исходящих подключений, которое в настоящее время 600 активных (всего 1 200) подключений на экземпляр. При достижении этого предела среда выполнения функций записывает в журналы следующее сообщение: `Host thresholds exceeded: Connections` . Дополнительные сведения см. в статье [ограничения службы функций](functions-scale.md#service-limits).

Это ограничение для каждого экземпляра. Когда [контроллер масштабирования добавляет экземпляры приложения функции](event-driven-scaling.md) для обработки большего количества запросов, каждый экземпляр имеет независимое ограничение на число подключений. Это означает, что нет глобального ограничения на подключение, и вы можете использовать гораздо больше 600 активных подключений для всех активных экземпляров.

При устранении неполадок убедитесь, что вы включили Application Insights для приложения функции. Application Insights позволяет просматривать метрики для приложений функций, таких как выполнение. Дополнительные сведения см. [в разделе Просмотр телеметрии в Application Insights](analyze-telemetry-data.md#view-telemetry-in-application-insights).  

## <a name="static-clients"></a>Статические клиенты

Чтобы избежать большего количества подключений, чем необходимо, повторно используйте экземпляры клиента, а не создавайте новые с каждым вызовом функции. Рекомендуется повторно использовать клиентские подключения для любого языка, в котором можно написать функцию. Например, клиенты .NET, такие как [HttpClient](/dotnet/api/system.net.http.httpclient?view=netcore-3.1&preserve-view=true), [DocumentClient](/dotnet/api/microsoft.azure.documents.client.documentclient)и клиенты службы хранилища Azure, могут управлять подключениями, если используется один статический клиент.

Ниже приведены некоторые рекомендации, которые необходимо выполнить при использовании клиента, зависящего от службы, в приложении "функции Azure".

- *Не* создавайте новый клиент при каждом вызове функции.
- *Создайте один* статический клиент, который может использовать каждый вызов функции.
- *Рассмотрите возможность* создания отдельного статического клиента в общем вспомогательном классе, если разные функции используют одну и ту же службу.

## <a name="client-code-examples"></a>Примеры кода клиента

В этом разделе даются рекомендации по созданию и использованию клиентов из вашего кода функции.

### <a name="httpclient-example-c"></a>Пример HttpClient (C#)

Ниже приведен пример кода функции C#, который создает статический экземпляр [HttpClient](/dotnet/api/system.net.http.httpclient?view=netcore-3.1&preserve-view=true) :

```cs
// Create a single, static HttpClient
private static HttpClient httpClient = new HttpClient();

public static async Task Run(string input)
{
    var response = await httpClient.GetAsync("https://example.com");
    // Rest of function
}
```

Распространенный вопрос о [HttpClient](/dotnet/api/system.net.http.httpclient?view=netcore-3.1&preserve-view=true) в .NET: «следует ли мне избавиться от моего клиента?» Как правило, удаляются объекты, которые реализуют, `IDisposable` когда вы закончите их использовать. Но вы не удаляете статический клиент, так как вы не закончите использовать его при завершении функции. Необходимо, чтобы статический клиент существовал в течение срока жизни приложения.

### <a name="http-agent-examples-javascript"></a>Примеры агента HTTP (JavaScript)

Так как он предоставляет улучшенные возможности управления подключениями, следует использовать собственный [`http.agent`](https://nodejs.org/dist/latest-v6.x/docs/api/http.html#http_class_http_agent) класс вместо несобственных методов, таких как `node-fetch` модуль. Параметры соединения настраиваются с помощью параметров `http.agent` класса. Подробные сведения о параметрах, доступных в агенте HTTP, см. в разделе [Создание агента ( \[ Параметры \] )](https://nodejs.org/dist/latest-v6.x/docs/api/http.html#http_new_agent_options).

Глобальный `http.globalAgent` класс, используемый, `http.request()` имеет значения по умолчанию для всех этих значений. Чтобы настроить ограничения для подключений в Функциях Azure, рекомендуется глобально задать максимальное число. В следующем примере задается максимальное количество сокетов для приложения-функции.

```js
http.globalAgent.maxSockets = 200;
```

 В следующем примере создается новый HTTP-запрос с пользовательским агентом HTTP только для этого запроса:

```js
var http = require('http');
var httpAgent = new http.Agent();
httpAgent.maxSockets = 200;
options.agent = httpAgent;
http.request(options, onResponseCallback);
```

### <a name="documentclient-code-example-c"></a>Пример кода DocumentClient (C#)

[DocumentClient](/dotnet/api/microsoft.azure.documents.client.documentclient) подключается к экземпляру Azure Cosmos DB. В документации Azure Cosmos DB рекомендуется [использовать отдельный клиент Azure Cosmos DB в течении всего жизненного цикла приложения](../cosmos-db/performance-tips.md#sdk-usage). В следующем примере показан один шаблон в функции, чтобы это делать.

```cs
#r "Microsoft.Azure.Documents.Client"
using Microsoft.Azure.Documents.Client;

private static Lazy<DocumentClient> lazyClient = new Lazy<DocumentClient>(InitializeDocumentClient);
private static DocumentClient documentClient => lazyClient.Value;

private static DocumentClient InitializeDocumentClient()
{
    // Perform any initialization here
    var uri = new Uri("example");
    var authKey = "authKey";
    
    return new DocumentClient(uri, authKey);
}

public static async Task Run(string input)
{
    Uri collectionUri = UriFactory.CreateDocumentCollectionUri("database", "collection");
    object document = new { Data = "example" };
    await documentClient.UpsertDocumentAsync(collectionUri, document);
    
    // Rest of function
}
```
При работе с функциями v3. x требуется ссылка на Microsoft.Azure.DocУментдб. Core. Добавьте ссылку в код:

```cs
#r "Microsoft.Azure.DocumentDB.Core"
```
Кроме того, создайте файл с именем Function. proj для триггера и добавьте приведенное ниже содержимое.

```cs

<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>netcoreapp3.0</TargetFramework>
    </PropertyGroup>
    <ItemGroup>
        <PackageReference Include="Microsoft.Azure.DocumentDB.Core" Version="2.12.0" />
    </ItemGroup>
</Project>

```
### <a name="cosmosclient-code-example-javascript"></a>Пример кода Космосклиент (JavaScript)
[Космосклиент](/javascript/api/@azure/cosmos/cosmosclient) подключается к экземпляру Azure Cosmos DB. В документации Azure Cosmos DB рекомендуется [использовать отдельный клиент Azure Cosmos DB в течении всего жизненного цикла приложения](../cosmos-db/performance-tips.md#sdk-usage). В следующем примере показан один шаблон в функции, чтобы это делать.

```javascript
const cosmos = require('@azure/cosmos');
const endpoint = process.env.COSMOS_API_URL;
const key = process.env.COSMOS_API_KEY;
const { CosmosClient } = cosmos;

const client = new CosmosClient({ endpoint, key });
// All function invocations also reference the same database and container.
const container = client.database("MyDatabaseName").container("MyContainerName");

module.exports = async function (context) {
    const { resources: itemArray } = await container.items.readAll().fetchAll();
    context.log(itemArray);
}
```

## <a name="sqlclient-connections"></a>Подключения SqlClient

Код функции может использовать поставщик данных платформа .NET Framework для SQL Server ([SqlClient](/dotnet/api/system.data.sqlclient)) для создания соединений с реляционной базой данных SQL. Это также базовый поставщик для платформ данных, которые используют ADO.NET, например [Entity Framework](/ef/ef6/). В отличие от соединений [HttpClient](/dotnet/api/system.net.http.httpclient) и [DocumentClient](/dotnet/api/microsoft.azure.documents.client.documentclient) ADO.NET осуществляет объединение подключений в пул по умолчанию. Но поскольку вы по-прежнему можете работать с нехваткой подключений, следует оптимизировать соединения с базой данных. Дополнительные сведения см. в разделе [Пулы подключений SQL Server (ADO.NET)](/dotnet/framework/data/adonet/sql-server-connection-pooling).

> [!TIP]
> Некоторые платформы данных, такие как Entity Framework, обычно получают строки подключения из раздела **ConnectionString** файла конфигурации. В этом случае необходимо добавить строки подключений базы данных SQL непосредственно в список функциональных настроек приложения **Строки подключения** и в [файл local.settings.json](functions-run-local.md#local-settings-file) в локальном проекте. Если вы создаете экземпляр [SqlConnection](/dotnet/api/system.data.sqlclient.sqlconnection) в коде функции, вы должны сохранить значение строки подключения в **параметрах приложения** с другими соединениями.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о том, почему мы рекомендуем использовать статические клиенты, см. в разделе [неправильное антишаблонное создание экземпляра](/azure/architecture/antipatterns/improper-instantiation/).

Дополнительные советы по повышению производительности службы "Функции Azure" см. в статье [Оптимизация производительности и надежности Функций Azure](functions-best-practices.md).
