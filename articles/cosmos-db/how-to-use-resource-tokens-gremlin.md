---
title: Использование маркеров ресурса Azure Cosmos DB с пакетом SDK Gremlin
description: Сведения о создании маркеров ресурса и их использования для доступа к базе данных Graph.
author: christopheranderson
ms.author: chrande
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 09/06/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 22c048b748806404ccfa580e660552a1744f3781
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93361699"
---
# <a name="use-azure-cosmos-db-resource-tokens-with-the-gremlin-sdk"></a>Использование маркеров ресурса Azure Cosmos DB с пакетом SDK Gremlin
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

В этой статье объясняется, как использовать [маркеры ресурса Azure Cosmos DB](secure-access-to-data.md) для доступа к базе данных графов через пакет SDK Gremlin.

## <a name="create-a-resource-token"></a>Создание маркера ресурса

Пакет SDK для Apache TinkerPop Gremlin не имеет API для создания маркеров ресурса. Термин *маркер ресурса* является концепцией Azure Cosmos DB. Для создания маркеров ресурса, скачайте [пакет SDK для Azure Cosmos DB](sql-api-sdk-dotnet.md). Если приложению необходимо создать маркеры ресурса и использовать их для доступа к базе данных Graph, потребуется 2 отдельных пакета SDK.

Следующая структура отображает иерархию объектной модели над маркерами ресурса:

- **Учетная запись Azure Cosmos DB** — объект верхнего уровня, с которым связана служба доменных имен (DNS), (например `contoso.gremlin.cosmos.azure.com`).
  - **База данных Azure Cosmos DB**
    - **Пользователь**
      - **Разрешение**
        - **Маркер** — свойство объекта "Разрешения", определяющий, какие действия разрешены или запрещены.

Маркер ресурса использует следующий формат: `"type=resource&ver=1&sig=<base64 string>;<base64 string>;"`. Эта строка является непрозрачной для клиентов и должна использоваться как есть без изменения или интерпретации.

```csharp
// Notice that document client is created against .NET SDK endpoint, rather than Gremlin.
DocumentClient client = new DocumentClient(
  new Uri("https://contoso.documents.azure.com:443/"), 
  "<primary key>", 
  new ConnectionPolicy 
  {
    EnableEndpointDiscovery = false, 
    ConnectionMode = ConnectionMode.Direct 
  });

  // Read specific permission to obtain a token.
  // The token isn't returned during the ReadPermissionReedAsync() call.
  // The call succeeds only if database id, user id, and permission id already exist. 
  // Note that <database id> is not a database name. It is a base64 string that represents the database identifier, for example "KalVAA==".
  // Similar comment applies to <user id> and <permission id>.
  Permission permission = await client.ReadPermissionAsync(UriFactory.CreatePermissionUri("<database id>", "<user id>", "<permission id>"));

  Console.WriteLine("Obtained token {0}", permission.Token);
}
```

## <a name="use-a-resource-token"></a>Использование маркера ресурса
При создании класса GremlinServer можно напрямую использовать маркеры ресурса в качестве свойства "пароль".

```csharp
// The Gremlin application needs to be given a resource token. It can't discover the token on its own.
// You can obtain the token for a given permission by using the Azure Cosmos DB SDK, or you can pass it into the application as a command line argument or configuration value.
string resourceToken = GetResourceToken();

// Configure the Gremlin server to use a resource token rather than a primary key.
GremlinServer server = new GremlinServer(
  "contoso.gremlin.cosmosdb.azure.com",
  port: 443,
  enableSsl: true,
  username: "/dbs/<database name>/colls/<collection name>",

  // The format of the token is "type=resource&ver=1&sig=<base64 string>;<base64 string>;".
  password: resourceToken);

  using (GremlinClient gremlinClient = new GremlinClient(server, new GraphSON2Reader(), new GraphSON2Writer(), GremlinClient.GraphSON2MimeType))
  {
      await gremlinClient.SubmitAsync("g.V().limit(1)");
  }
```

Аналогичный подход работает во всех пакетах SDK для TinkerPop Gremlin.

```java
Cluster.Builder builder = Cluster.build();

AuthProperties authenticationProperties = new AuthProperties();
authenticationProperties.with(AuthProperties.Property.USERNAME,
    String.format("/dbs/%s/colls/%s", "<database name>", "<collection name>"));

// The format of the token is "type=resource&ver=1&sig=<base64 string>;<base64 string>;".
authenticationProperties.with(AuthProperties.Property.PASSWORD, resourceToken);

builder.authProperties(authenticationProperties);
```

## <a name="limit"></a>Ограничение

Одна учетная запись Gremlin позволяет выпускать неограниченное количество маркеров. Однако на протяжении 1 часа можно использовать до 100 маркеров одновременно. Если приложение превышает ограничение на количество маркеров в час, запрос на проверку подлинности отклоняется и появляется следующее сообщение об ошибке: "превышено допустимое ограничение маркера ресурсов 100, которое можно использовать одновременно". Это не работает для закрытия активных соединений, использующих определенные маркеры, чтобы освободить слоты для новых маркеров. Ядро базы данных Azure Cosmos DB Gremlin отслеживает уникальные маркеры в течение часа непосредственно перед запросом аутентификации.

## <a name="permission"></a>Разрешение

Распространенная ошибка, с которой сталкиваются приложения при использовании маркеров ресурсов: "Недостаточно разрешений в заголовке авторизации для соответствующего запроса. Повторите попытку используя другой заголовок авторизации." Эта ошибка возвращается, когда обход Gremlin пытается записать край или вершину, но маркер ресурса предоставляет только разрешения на *Чтение*. Проверьте обход на содержание какого-либо из следующих шагов: *.addV ()*, *.addE ()*, *.drop ()* или *.property()*.

## <a name="next-steps"></a>Дальнейшие действия
* [Управление доступом на основе ролей в Azure (Azure RBAC)](role-based-access-control.md) в Azure Cosmos DB
* [Узнайте, как защитить доступ к данным](secure-access-to-data.md) в Azure Cosmos DB
