---
title: Управление политиками индексирования в Azure Cosmos DB
description: Узнайте, как управлять политиками индексации, включать или исключать свойства из индексирования, определять индексацию с помощью различных пакетов SDK Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 11/02/2020
ms.author: tisande
ms.custom: devx-track-python, devx-track-js, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 8d52f8c59e83a4aae8724100770965f756a439fb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98015697"
---
# <a name="manage-indexing-policies-in-azure-cosmos-db"></a>Управление политиками индексирования в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В Azure Cosmos DB данные индексируются в соответствии с [политиками индексации](index-policy.md) , определенными для каждого контейнера. Политика индексирования по умолчанию, задаваемая для только что созданных контейнеров, применяет диапазонные индексы для любых строк или чисел. Эту политику можно переопределить собственной политикой индексирования.

> [!NOTE]
> Метод обновления политик индексации, описанный в этой статье, применим только к API-интерфейсу SQL (Core) Azure Cosmos DB. Сведения об индексировании в [API Azure Cosmos DB для MongoDB](mongodb-indexing.md) и [дополнительного индексирования в Azure Cosmos DB API Cassandra.](cassandra-secondary-index.md)

## <a name="indexing-policy-examples"></a>Примеры политик индексирования

Ниже приведены некоторые примеры политик индексации, которые отображаются в [формате JSON](index-policy.md#include-exclude-paths), то есть о том, как они представлены в портал Azure. Значения параметров можно задать с помощью Azure CLI или любого пакета SDK.

### <a name="opt-out-policy-to-selectively-exclude-some-property-paths"></a><a id="range-index"></a>Политика отказа для выборочного исключения некоторых путей к свойствам

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/path/to/single/excluded/property/?"
            },
            {
                "path": "/path/to/root/of/multiple/excluded/properties/*"
            }
        ]
    }
```

Эта политика индексирования эквивалентна приведенной ниже таблице, в которой вручную устанавливаются ```kind``` , ```dataType``` и ```precision``` в значения по умолчанию. Эти свойства больше не нужны для явной установки, и их следует полностью опустить в политике индексирования (как показано в примере выше).

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/*",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number",
                        "precision": -1
                    },
                    {
                        "kind": "Range",
                        "dataType": "String",
                        "precision": -1
                    }
                ]
            }
        ],
        "excludedPaths": [
            {
                "path": "/path/to/single/excluded/property/?"
            },
            {
                "path": "/path/to/root/of/multiple/excluded/properties/*"
            }
        ]
    }
```

### <a name="opt-in-policy-to-selectively-include-some-property-paths"></a>Политика принятия для выборочного включения некоторых путей к свойствам

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/path/to/included/property/?"
            },
            {
                "path": "/path/to/root/of/multiple/included/properties/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            }
        ]
    }
```

Эта политика индексирования эквивалентна приведенной ниже таблице, в которой вручную устанавливаются ```kind``` , ```dataType``` и ```precision``` в значения по умолчанию. Эти свойства больше не нужны для явной установки, и их следует полностью опустить в политике индексирования (как показано в примере выше).

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/path/to/included/property/?",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number"
                    },
                    {
                        "kind": "Range",
                        "dataType": "String"
                    }
                ]
            },
            {
                "path": "/path/to/root/of/multiple/included/properties/*",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number"
                    },
                    {
                        "kind": "Range",
                        "dataType": "String"
                    }
                ]
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            }
        ]
    }
```

> [!NOTE]
> Обычно рекомендуется использовать политику индексирования **отказа** , чтобы Azure Cosmos DB заранее индексировать любое новое свойство, которое может быть добавлено в модель данных.

### <a name="using-a-spatial-index-on-a-specific-property-path-only"></a><a id="spatial-index"></a>Использование пространственного индекса только для определенного пути к свойству

```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/*"
        }
    ],
    "excludedPaths": [
        {
            "path": "/_etag/?"
        }
    ],
    "spatialIndexes": [
        {
            "path": "/path/to/geojson/property/?",
            "types": [
                "Point",
                "Polygon",
                "MultiPolygon",
                "LineString"
            ]
        }
    ]
}
```

## <a name="composite-indexing-policy-examples"></a><a id="composite-index"></a>Примеры политик составного индексирования

Кроме добавления и удаления путей отдельных свойств, вы также можете указать составной индекс. Если вы хотите выполнить запрос, который содержит предложение `ORDER BY` для нескольких свойств, эти свойства должны содержать [составной индекс](index-policy.md#composite-indexes) Кроме того, составные индексы будут иметь преимущество в производительности для запросов, имеющих несколько фильтров или как фильтр, так и предложение ORDER BY.

> [!NOTE]
> Составные пути имеют неявные, `/?` так как только скалярное значение в этом пути индексируется. `/*`Подстановочные знаки не поддерживаются в составных путях. Не следует указывать `/?` или `/*` в составном пути.

### <a name="composite-index-defined-for-name-asc-age-desc"></a>Определенный составной индекс (name asc, age desc):

```json
    {  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"descending"
                }
            ]
        ]
    }
```

Приведенный выше составной индекс по имени и возрасту необходим для #1 запросов и #2 запросов:

Запрос 1:

```sql
    SELECT *
    FROM c
    ORDER BY c.name ASC, c.age DESC
```

Запрос 2:

```sql
    SELECT *
    FROM c
    ORDER BY c.name DESC, c.age ASC
```

Этот составной индекс поможет #3 запросов и #4 запросов и оптимизации фильтров.

#3 запроса:

```sql
SELECT *
FROM c
WHERE c.name = "Tim"
ORDER BY c.name DESC, c.age ASC
```

#4 запроса:

```sql
SELECT *
FROM c
WHERE c.name = "Tim" AND c.age > 18
```

### <a name="composite-index-defined-for-name-asc-age-asc-and-name-asc-age-desc"></a>Составной индекс, определенный для (Name ASC, Age ASC) и (Name ASC, Age DESC):

Вы можете определить несколько разных составных индексов в одной и той же политике индексирования.

```json
    {  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"ascending"
                }
            ],
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"descending"
                }
            ]
        ]
    }
```

### <a name="composite-index-defined-for-name-asc-age-asc"></a>Составной индекс, определенный для (Name ASC, Age ASC):

Порядок указывать необязательно. Если он не указан, по умолчанию используется порядок по возрастанию.

```json
{  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                },
                {  
                    "path":"/age",
                }
            ]
        ]
}
```

### <a name="excluding-all-property-paths-but-keeping-indexing-active"></a>Исключение всех путей к свойствам, но с активным индексированием

Эту политику можно использовать в ситуациях, когда [функция срока жизни (TTL)](time-to-live.md) активна, но дополнительные индексы не требуются (для использования Azure Cosmos DB в качестве чистого хранилища ключей).

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [],
        "excludedPaths": [{
            "path": "/*"
        }]
    }
```

### <a name="no-indexing"></a>Без индексирования

Эта политика отключит индексирование. Если параметр `indexingMode` имеет значение `none` , то нельзя задать TTL для контейнера.

```json
    {
        "indexingMode": "none"
    }
```

## <a name="updating-indexing-policy"></a>Обновление политики индексации

В Azure Cosmos DB политику индексирования можно обновить одним из следующих способов.

- на портале Azure;
- с помощью Azure CLI;
- Использование PowerShell
- с помощью одного из пакетов SDK.

[Обновление политики индексирования](index-policy.md#modifying-the-indexing-policy) приводит к преобразованию индекса. Ход выполнения этого преобразования можно отслеживать с помощью пакетов SDK.

> [!NOTE]
> При обновлении политики индексации операции записи в Azure Cosmos DB будут непрерывными. Дополнительные сведения о [преобразованиях индексирования](index-policy.md#modifying-the-indexing-policy)

## <a name="use-the-azure-portal"></a>Использование портала Azure

Контейнеры Azure Cosmos хранят используемую политику индексирования в виде документа JSON, который можно непосредственно редактировать на портале Azure.

1. Войдите на [портал Azure](https://portal.azure.com/).

1. Создайте новую учетную запись Azure Cosmos или выберите имеющуюся.

1. Откройте панель **обозревателя данных** и выберите контейнер, с которым собираетесь работать.

1. Щелкните **Scale & Settings** (Масштаб и параметры).

1. Измените документ JSON политики индексирования (ознакомьтесь с примерами [ниже](#indexing-policy-examples)).

1. После завершения нажмите кнопку **Save** (Сохранить).

:::image type="content" source="./media/how-to-manage-indexing-policy/indexing-policy-portal.png" alt-text="Управление индексированием с помощью портал Azure":::

## <a name="use-the-azure-cli"></a>Использование Azure CLI

Сведения о создании контейнера с пользовательской политикой индексации см. в разделе [Создание контейнера с настраиваемой политикой индексов с помощью интерфейса командной строки](manage-with-cli.md#create-a-container-with-a-custom-index-policy) .

## <a name="use-powershell"></a>Использование PowerShell

Сведения о создании контейнера с пользовательской политикой индексации см. в разделе [Создание контейнера с настраиваемой политикой индексов с помощью PowerShell](manage-with-powershell.md#create-container-custom-index) .

## <a name="use-the-net-sdk"></a><a id="dotnet-sdk"></a> Использование пакета SDK для .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

`DocumentCollection`Объект из [пакета SDK .NET v2](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/) предоставляет `IndexingPolicy` свойство, которое позволяет изменять `IndexingMode` и добавлять и удалять `IncludedPaths` и `ExcludedPaths` .

```csharp
// Retrieve the container's details
ResourceResponse<DocumentCollection> containerResponse = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("database", "container"));
// Set the indexing mode to consistent
containerResponse.Resource.IndexingPolicy.IndexingMode = IndexingMode.Consistent;
// Add an included path
containerResponse.Resource.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
// Add an excluded path
containerResponse.Resource.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/name/*" });
// Add a spatial index
containerResponse.Resource.IndexingPolicy.SpatialIndexes.Add(new SpatialSpec() { Path = "/locations/*", SpatialTypes = new Collection<SpatialType>() { SpatialType.Point } } );
// Add a composite index
containerResponse.Resource.IndexingPolicy.CompositeIndexes.Add(new Collection<CompositePath> {new CompositePath() { Path = "/name", Order = CompositePathSortOrder.Ascending }, new CompositePath() { Path = "/age", Order = CompositePathSortOrder.Descending }});
// Update container with changes
await client.ReplaceDocumentCollectionAsync(containerResponse.Resource);
```

Чтобы отслеживать ход выполнения преобразования индекса, передайте объект `RequestOptions`, который задает для свойства `PopulateQuotaInfo` значение `true`.

```csharp
// retrieve the container's details
ResourceResponse<DocumentCollection> container = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("database", "container"), new RequestOptions { PopulateQuotaInfo = true });
// retrieve the index transformation progress from the result
long indexTransformationProgress = container.IndexTransformationProgress;
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

`ContainerProperties`Объект из [пакета SDK .NET v3](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/) (см. [это краткое руководство](create-sql-api-dotnet.md) по использованию) предоставляет `IndexingPolicy` свойство, которое позволяет изменять `IndexingMode` и добавлять и удалять `IncludedPaths` и `ExcludedPaths` .

```csharp
// Retrieve the container's details
ContainerResponse containerResponse = await client.GetContainer("database", "container").ReadContainerAsync();
// Set the indexing mode to consistent
containerResponse.Resource.IndexingPolicy.IndexingMode = IndexingMode.Consistent;
// Add an included path
containerResponse.Resource.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
// Add an excluded path
containerResponse.Resource.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/name/*" });
// Add a spatial index
SpatialPath spatialPath = new SpatialPath
{
    Path = "/locations/*"
};
spatialPath.SpatialTypes.Add(SpatialType.Point);
containerResponse.Resource.IndexingPolicy.SpatialIndexes.Add(spatialPath);
// Add a composite index
containerResponse.Resource.IndexingPolicy.CompositeIndexes.Add(new Collection<CompositePath> { new CompositePath() { Path = "/name", Order = CompositePathSortOrder.Ascending }, new CompositePath() { Path = "/age", Order = CompositePathSortOrder.Descending } });
// Update container with changes
await client.GetContainer("database", "container").ReplaceContainerAsync(containerResponse.Resource);
```

Чтобы отслеживать ход преобразования индекса, передайте `RequestOptions` объект, который устанавливает `PopulateQuotaInfo` свойство, в `true` , а затем извлекает значение из `x-ms-documentdb-collection-index-transformation-progress` заголовка ответа.

```csharp
// retrieve the container's details
ContainerResponse containerResponse = await client.GetContainer("database", "container").ReadContainerAsync(new ContainerRequestOptions { PopulateQuotaInfo = true });
// retrieve the index transformation progress from the result
long indexTransformationProgress = long.Parse(containerResponse.Headers["x-ms-documentdb-collection-index-transformation-progress"]);
```

При определении пользовательской политики индексирования при создании нового контейнера API-интерфейс V3's Fluent пакета SDK позволяет быстро и эффективно писать это определение:

```csharp
await client.GetDatabase("database").DefineContainer(name: "container", partitionKeyPath: "/myPartitionKey")
    .WithIndexingPolicy()
        .WithIncludedPaths()
            .Path("/*")
        .Attach()
        .WithExcludedPaths()
            .Path("/name/*")
        .Attach()
        .WithSpatialIndex()
            .Path("/locations/*", SpatialType.Point)
        .Attach()
        .WithCompositeIndex()
            .Path("/name", CompositePathSortOrder.Ascending)
            .Path("/age", CompositePathSortOrder.Descending)
        .Attach()
    .Attach()
    .CreateIfNotExistsAsync();
```
---

## <a name="use-the-java-sdk"></a>Использование пакета SDK для Java

Объект `DocumentCollection` из [пакета SDK для Java](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb) (его использование описано в [этом кратком руководстве](create-sql-api-java.md)) предоставляет методы `getIndexingPolicy()` и `setIndexingPolicy()`. Объект `IndexingPolicy`, которым они управляют, позволяет изменить режим индексирования и добавить или удалить включенные и исключенные пути.

```java
// Retrieve the container's details
Observable<ResourceResponse<DocumentCollection>> containerResponse = client.readCollection(String.format("/dbs/%s/colls/%s", "database", "container"), null);
containerResponse.subscribe(result -> {
DocumentCollection container = result.getResource();
IndexingPolicy indexingPolicy = container.getIndexingPolicy();

// Set the indexing mode to consistent
indexingPolicy.setIndexingMode(IndexingMode.Consistent);

// Add an included path

Collection<IncludedPath> includedPaths = new ArrayList<>();
IncludedPath includedPath = new IncludedPath();
includedPath.setPath("/*");
includedPaths.add(includedPath);
indexingPolicy.setIncludedPaths(includedPaths);

// Add an excluded path

Collection<ExcludedPath> excludedPaths = new ArrayList<>();
ExcludedPath excludedPath = new ExcludedPath();
excludedPath.setPath("/name/*");
excludedPaths.add(excludedPath);
indexingPolicy.setExcludedPaths(excludedPaths);

// Add a spatial index

Collection<SpatialSpec> spatialIndexes = new ArrayList<SpatialSpec>();
Collection<SpatialType> collectionOfSpatialTypes = new ArrayList<SpatialType>();

SpatialSpec spec = new SpatialSpec();
spec.setPath("/locations/*");
collectionOfSpatialTypes.add(SpatialType.Point);
spec.setSpatialTypes(collectionOfSpatialTypes);
spatialIndexes.add(spec);

indexingPolicy.setSpatialIndexes(spatialIndexes);

// Add a composite index

Collection<ArrayList<CompositePath>> compositeIndexes = new ArrayList<>();
ArrayList<CompositePath> compositePaths = new ArrayList<>();

CompositePath nameCompositePath = new CompositePath();
nameCompositePath.setPath("/name");
nameCompositePath.setOrder(CompositePathSortOrder.Ascending);

CompositePath ageCompositePath = new CompositePath();
ageCompositePath.setPath("/age");
ageCompositePath.setOrder(CompositePathSortOrder.Descending);

compositePaths.add(ageCompositePath);
compositePaths.add(nameCompositePath);

compositeIndexes.add(compositePaths);
indexingPolicy.setCompositeIndexes(compositeIndexes);

// Update the container with changes

 client.replaceCollection(container, null);
});
```

Чтобы отслеживать ход выполнения преобразования индекса в контейнере, передайте объект `RequestOptions`, который запрашивает сведения о квоте для заполнения, а затем извлеките значение из заголовка ответа `x-ms-documentdb-collection-index-transformation-progress`.

```java
// set the RequestOptions object
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPopulateQuotaInfo(true);
// retrieve the container's details
Observable<ResourceResponse<DocumentCollection>> containerResponse = client.readCollection(String.format("/dbs/%s/colls/%s", "database", "container"), requestOptions);
containerResponse.subscribe(result -> {
    // retrieve the index transformation progress from the response headers
    String indexTransformationProgress = result.getResponseHeaders().get("x-ms-documentdb-collection-index-transformation-progress");
});
```

## <a name="use-the-nodejs-sdk"></a>Использование пакета SDK для Node.js

Интерфейс `ContainerDefinition` из [пакета SDK для Node.js](https://www.npmjs.com/package/@azure/cosmos) (его использование описано в [этом кратком руководстве](create-sql-api-nodejs.md)) предоставляет свойство `indexingPolicy`, которое позволяет изменить `indexingMode` и добавить или удалить `includedPaths` и `excludedPaths`.

Получение сведений о контейнере

```javascript
const containerResponse = await client.database('database').container('container').read();
```

Установить режим индексирования "последовательный"

```javascript
containerResponse.body.indexingPolicy.indexingMode = "consistent";
```

Добавление включенного пути, включая пространственный индекс

```javascript
containerResponse.body.indexingPolicy.includedPaths.push({
    includedPaths: [
      {
        path: "/age/*",
        indexes: [
          {
            kind: cosmos.DocumentBase.IndexKind.Range,
            dataType: cosmos.DocumentBase.DataType.String
          },
          {
            kind: cosmos.DocumentBase.IndexKind.Range,
            dataType: cosmos.DocumentBase.DataType.Number
          }
        ]
      },
      {
        path: "/locations/*",
        indexes: [
          {
            kind: cosmos.DocumentBase.IndexKind.Spatial,
            dataType: cosmos.DocumentBase.DataType.Point
          }
        ]
      }
    ]
  });
```

Добавление исключенного пути

```javascript
containerResponse.body.indexingPolicy.excludedPaths.push({ path: '/name/*' });
```

Обновление контейнера с изменениями

```javascript
const replaceResponse = await client.database('database').container('container').replace(containerResponse.body);
```

Чтобы отслеживать ход выполнения преобразования индекса в контейнере, передайте объект `RequestOptions`, который задает для свойства `populateQuotaInfo` значение `true`, а затем извлеките значение из заголовка ответа `x-ms-documentdb-collection-index-transformation-progress`.

```javascript
// retrieve the container's details
const containerResponse = await client.database('database').container('container').read({
    populateQuotaInfo: true
});
// retrieve the index transformation progress from the response headers
const indexTransformationProgress = replaceResponse.headers['x-ms-documentdb-collection-index-transformation-progress'];
```

## <a name="use-the-python-sdk"></a>Использование пакета SDK для Python

# <a name="python-sdk-v3"></a>[Пакет SDK для Python v3](#tab/pythonv3)

При использовании [пакета SDK версии 3 для Python](https://pypi.org/project/azure-cosmos/) (см. [это краткое руководство](create-sql-api-python.md) по использованию) конфигурация контейнера управляется как словарь. Из этого словаря можно осуществлять доступ к политике индексации и всем ее атрибутам.

Получение сведений о контейнере

```python
containerPath = 'dbs/database/colls/collection'
container = client.ReadContainer(containerPath)
```

Установить режим индексирования "последовательный"

```python
container['indexingPolicy']['indexingMode'] = 'consistent'
```

Определение политики индексации с помощью включаемого пути и пространственного индекса

```python
container["indexingPolicy"] = {

    "indexingMode":"consistent",
    "spatialIndexes":[
                {"path":"/location/*","types":["Point"]}
             ],
    "includedPaths":[{"path":"/age/*","indexes":[]}],
    "excludedPaths":[{"path":"/*"}]
}
```

Определение политики индексации с исключенным путем

```python
container["indexingPolicy"] = {
    "indexingMode":"consistent",
    "includedPaths":[{"path":"/*","indexes":[]}],
    "excludedPaths":[{"path":"/name/*"}]
}
```

Добавление составного индекса

```python
container['indexingPolicy']['compositeIndexes'] = [
                [
                    {
                        "path": "/name",
                        "order": "ascending"
                    },
                    {
                        "path": "/age",
                        "order": "descending"
                    }
                ]
                ]
```

Обновление контейнера с изменениями

```python
response = client.ReplaceContainer(containerPath, container)
```

# <a name="python-sdk-v4"></a>[Пакет SDK для Python v4](#tab/pythonv4)

При использовании [пакета SDK для Python v4](https://pypi.org/project/azure-cosmos/)конфигурация контейнеров управляется как словарь. Из этого словаря можно осуществлять доступ к политике индексации и всем ее атрибутам.

Получение сведений о контейнере

```python
database_client = cosmos_client.get_database_client('database')
container_client = database_client.get_container_client('container')
container = container_client.read()
```

Установить режим индексирования "последовательный"

```python
indexingPolicy = {
    'indexingMode': 'consistent'
}
```

Определение политики индексации с помощью включаемого пути и пространственного индекса

```python
indexingPolicy = {
    "indexingMode":"consistent",
    "spatialIndexes":[
        {"path":"/location/*","types":["Point"]}
    ],
    "includedPaths":[{"path":"/age/*","indexes":[]}],
    "excludedPaths":[{"path":"/*"}]
}
```

Определение политики индексации с исключенным путем

```python
indexingPolicy = {
    "indexingMode":"consistent",
    "includedPaths":[{"path":"/*","indexes":[]}],
    "excludedPaths":[{"path":"/name/*"}]
}
```

Добавление составного индекса

```python
indexingPolicy['compositeIndexes'] = [
    [
        {
            "path": "/name",
            "order": "ascending"
        },
        {
            "path": "/age",
            "order": "descending"
        }
    ]
]
```

Обновление контейнера с изменениями

```python
response = database_client.replace_container(container_client, container['partitionKey'], indexingPolicy)
```

Получение сведений о ходе преобразования индекса из заголовков ответа
```python
container_client.read(populate_quota_info = True,
                      response_hook = lambda h,p: print(h['x-ms-documentdb-collection-index-transformation-progress']))
```

---

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об индексировании см. по следующим ссылкам:

- [Общие сведения об индексировании](index-overview.md)
- [Политика индексирования](index-policy.md)
