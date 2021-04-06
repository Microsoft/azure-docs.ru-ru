---
title: Миграция с CouchBase на API SQL для Azure Cosmos DB
description: Пошаговое руководство по миграции с CouchBase на API SQL для Azure Cosmos DB
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 02/11/2020
ms.author: mansha
author: manishmsfte
ms.custom: devx-track-java
ms.openlocfilehash: a15c6b5919f428b28daab86fea9c3b6473d19162
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97606204"
---
# <a name="migrate-from-couchbase-to-azure-cosmos-db-sql-api"></a>Миграция с CouchBase на API SQL для Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB — это масштабируемая, глобально распределенная, полностью управляемая база данных. Она обеспечивает гарантированный доступ к данным с низкой задержкой. Дополнительные сведения об Azure Cosmos DB см. в [этой обзорной статье](introduction.md). В этой статье приведены инструкции по переносу приложений Java, подключенных к Couchbase, в учетную запись API SQL в Azure Cosmos DB.

## <a name="differences-in-nomenclature"></a>Различия в номенклатуре

Ниже приведены основные функции, которые по-разному работают в Azure Cosmos DB и Couchbase.

| Couchbase | Azure Cosmos DB |
|--|--|
| Сервер Couchbase | Учетная запись |
| Сегмент | База данных |
| Сегмент | Контейнер или коллекция |
| Документ JSON | Элемент или документ |

## <a name="key-differences"></a>Основные отличия

* Azure Cosmos DB содержит поле ID в документе, тогда как в Couchbase идентификатор является частью сегмента. Поле ID является уникальным в пределах секции.

* Azure Cosmos DB масштабируется путем секционирования или сегментирования. Это означает, что данные разбиваются на несколько сегментов или секций. Эти секции или сегменты создаются на основе заданного свойства ключа секции. Вы можете выбрать ключ секции, чтобы оптимизировать операции чтения, записи или и те, и другие. Дополнительные сведения см. в статье о [секционировании](./partitioning-overview.md).

* В Azure Cosmos DB не требуется, чтобы иерархия верхнего уровня обозначала коллекцию, так как имя коллекции уже существует. Эта возможность значительно упрощает структуру JSON. Ниже приведен пример, демонстрирующий различия в модели данных между Couchbase и Azure Cosmos DB.

   **Couchbase**: Document ID = "99FF4444"

    ```json
    {
      "TravelDocument":
      {
       "Country":"India",
      "Validity" : "2022-09-01",
        "Person":
        {
          "Name": "Manish",
          "Address": "AB Road, City-z"
        },
        "Visas":
        [
          {
          "Country":"India",
          "Type":"Multi-Entry",
          "Validity":"2022-09-01"
          },
          {
          "Country":"US",
          "Type":"Single-Entry",
          "Validity":"2022-08-01"
          }
        ]
      }
    }
   ```

   **Azure Cosmos DB**: поле ID содержится в документе, как показано ниже

    ```json
    {
      "id" : "99FF4444",
    
      "Country":"India",
       "Validity" : "2022-09-01",
        "Person":
        {
          "Name": "Manish",
          "Address": "AB Road, City-z"
        },
        "Visas":
        [
          {
          "Country":"India",
          "Type":"Multi-Entry",
          "Validity":"2022-09-01"
          },
          {
          "Country":"US",
          "Type":"Single-Entry",
          "Validity":"2022-08-01"
          }
        ]
      }
    
    ```
         
## <a name="java-sdk-support"></a>Поддержка Java SDK

Azure Cosmos DB содержит следующие пакеты SDK для поддержки различных платформ Java:

* Async SDK;
* Spring Boot SDK.

В разделах ниже описано использование каждого из этих пакетов SDK. Рассмотрим пример, в котором у нас есть три типа рабочих нагрузок.

## <a name="couchbase-as-document-repository--spring-data-based-custom-queries"></a>Couchbase в качестве репозитория документов и настраиваемые запросы на основе данных Spring

Если переносимая рабочая нагрузка основана на Spring Boot SDK, можно выполнить следующие действия.

1. Добавьте родительский элемент в файл POM.xml:

   ```java
   <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.1.5.RELEASE</version>
      <relativePath/>
   </parent>
   ```

1. Добавьте свойства в файл POM.xml:

   ```java
   <azure.version>2.1.6</azure.version>
   ```

1. Добавьте зависимости в файл POM.xml:

   ```java
   <dependency>
       <groupId>com.microsoft.azure</groupId>
       <artifactId>azure-cosmosdb-spring-boot-starter</artifactId>
       <version>2.1.6</version>
   </dependency>
   ```

1. Добавьте свойства приложения в раздел ресурсов и укажите следующие параметры. Обязательно замените параметры URL-адреса, ключа и имени базы данных:

   ```java
      azure.cosmosdb.uri=<your-cosmosDB-URL>
      azure.cosmosdb.key=<your-cosmosDB-key>
      azure.cosmosdb.database=<your-cosmosDB-dbName>
   ```

1. Определите имя коллекции в модели. Можно также указать дополнительные заметки. Например, можно явно задать идентификатор и ключ секции:

   ```java
   @Document(collection = "mycollection")
       public class User {
           @id
           private String id;
           private String firstName;
           @PartitionKey
           private String lastName;
       }
   ```

Ниже приведены фрагменты кода для операций CRUD.

### <a name="insert-and-update-operations"></a>Операции вставки и обновления

Где *_repo* — объект репозитория, а *doc* — объект класса POJO. Метод `.save` можно использовать для вставки или операции upsert (если найден документ с указанным идентификатором). В следующем фрагменте кода показано, как вставить или обновить объект doc:

```_repo.save(doc);```

### <a name="delete-operation"></a>Операция удаления

Ниже приведен фрагмент кода, в котором объект doc будет иметь идентификатор и ключ секции, обязательные для нахождение и удаления объекта:

```_repo.delete(doc);```

### <a name="read-operation"></a>Операция чтения

Документ можно прочитать с указанием ключа секции или без него. Если ключ секции не указан, он рассматривается как межсекционный запрос. Ниже приведены примеры кода, первый из которых будет выполнять операцию с использованием идентификатора и поля ключа секции. Во втором примере используется обычное поле и не указывается поле ключа секции.

* ```_repo.findByIdAndName(objDoc.getId(),objDoc.getName());```
* ```_repo.findAllByStatus(objDoc.getStatus());```

Теперь вы можете использовать свое приложение с Azure Cosmos DB. Полный код примера, описанного в этом документе, доступен в репозитории GitHub под названием [CouchbaseToCosmosDB-SpringCosmos](https://github.com/Azure-Samples/couchbaseTocosmosdb/tree/main/SpringCosmos).

## <a name="couchbase-as-a-document-repository--using-n1ql-queries"></a>Couchbase в качестве репозитория документов и запросы N1QL

Запросы N1QL — это способ определения запросов в Couchbase.

|Запрос N1QL | Запрос Azure CosmosDB|
|-------------------|-------------------|
|SELECT META(`TravelDocument`).id AS id, `TravelDocument`.* FROM `TravelDocument` WHERE `_type` = "com.xx.xx.xx.xxx.xxx.xxxx " and country = 'India’ and ANY m in Visas SATISFIES m.type == 'Multi-Entry' and m.Country IN ['India', Bhutan’] ORDER BY ` Validity` DESC LIMIT 25 OFFSET 0 | SELECT c.id,c FROM c JOIN m in  c.country=’India’ WHERE c._type = " com.xx.xx.xx.xxx.xxx.xxxx" and c.country = 'India' and m.type = 'Multi-Entry' and m.Country IN ('India', 'Bhutan') ORDER BY c.Validity DESC OFFSET 0 LIMIT 25 |

В запросах N1QL можно заметить следующие изменения:

* Не нужно использовать ключевое слово META или ссылаться на документ первого уровня. Вместо этого можно создать собственную ссылку на контейнер. В этом примере мы обращались к ней по имени "c" (оно может быть любым). Эта ссылка используется в качестве префикса для всех полей первого уровня. Например: c.id, c.country и т. д.

* Вместо оператора ANY теперь можно выполнить объединение с вложенным документом и ссылаться на него с помощью выделенного псевдонима, например "m". После создания псевдонима для вложенного документа необходимо использовать псевдоним. Например: m. Country.

* Порядок использования предложения OFFSET в запросе Azure Cosmos DB отличается: сначала необходимо указать OFFSET, а затем LIMIT. Не рекомендуется использовать пакет Spring Data SDK, если вы используете максимальное количество пользовательских запросов, так как он может создать ненужную дополнительную нагрузку на стороне клиента при передаче запроса в Azure Cosmos DB. Вместо него гораздо эффективнее будет использовать пакет Async Java SDK.

### <a name="read-operation"></a>Операция чтения

Чтобы использовать Async Java SDK, выполнив указанные ниже действия.

1. Настройте следующую зависимость в файле POM.xml:

   ```java
   <!-- https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb -->
   <dependency>
       <groupId>com.microsoft.azure</groupId>
       <artifactId>azure-cosmos</artifactId>
       <version>3.0.0</version>
   </dependency>
   ```

1. Создайте объект подключения для Azure Cosmos DB с помощью метода `ConnectionBuilder`, как показано в следующем примере. Поместите это объявление в класс (бин) таким образом, чтобы следующий код выполнялся только один раз:

   ```java
   ConnectionPolicy cp=new ConnectionPolicy();
   cp.connectionMode(ConnectionMode.DIRECT);
    
   if(client==null)
      client= CosmosClient.builder()
         .endpoint(Host)//(Host, PrimaryKey, dbName, collName).Builder()
          .connectionPolicy(cp)
          .key(PrimaryKey)
          .consistencyLevel(ConsistencyLevel.EVENTUAL)
          .build();
   
   container = client.getDatabase(_dbName).getContainer(_collName);
   ```

1. Чтобы выполнить запрос, необходимо выполнить следующий фрагмент кода:

   ```java
   Flux<FeedResponse<CosmosItemProperties>> objFlux= container.queryItems(query, fo);
   ```

Теперь с помощью указанного выше метода вы сможете передавать различные запросы и без проблем выполнять их. Если вам необходимо выполнить один большой запрос, который можно разделить на несколько, попробуйте выполнить следующий фрагмент кода, а не предыдущий:

```java
for(SqlQuerySpec query:queries)
{
   objFlux= container.queryItems(query, fo);
   objFlux .publishOn(Schedulers.elastic())
         .subscribe(feedResponse->
            {
               if(feedResponse.results().size()>0)
               {
                  _docs.addAll(feedResponse.results());
               }
            
            },
            Throwable::printStackTrace,latch::countDown);
   lstFlux.add(objFlux);
}
                  
      Flux.merge(lstFlux);
      latch.await();
}
```

С помощью предыдущего фрагмента можно выполнять запросы параллельно, оптимизируя работу за счет распределенных выполнений. Кроме того, можно выполнять операции вставки и обновления.

### <a name="insert-operation"></a>Операция вставки

Чтобы вставить документ, выполните следующий код:

```java
Mono<CosmosItemResponse> objMono= container.createItem(doc,ro);
```

Затем подпишитесь на Mono:

```java
CountDownLatch latch=new CountDownLatch(1);
objMono .subscribeOn(Schedulers.elastic())
        .subscribe(resourceResponse->
        {
           if(resourceResponse.statusCode()!=successStatus)
              {
                 throw new RuntimeException(resourceResponse.toString());
              }
           },
        Throwable::printStackTrace,latch::countDown);
latch.await();
```

### <a name="upsert-operation"></a>Операция upsert

Для операции Upsert требуется указать документ, который необходимо обновить. Чтобы получить весь документ, можно использовать фрагмент, приведенный под заголовком "Операция чтения", а затем изменить нужные поля. Следующий фрагмент кода выполняет для документа операцию upsert:

```java
Mono<CosmosItemResponse> obs= container.upsertItem(doc, ro);
```
Затем подпишитесь на Mono. Фрагмент подписки на Mono см. в описании операции вставки.

### <a name="delete-operation"></a>Операция удаления

Следующий фрагмент кода выполняет операцию удаления:

```java
CosmosItem objItem= container.getItem(doc.Id, doc.Tenant);
Mono<CosmosItemResponse> objMono = objItem.delete(ro);
```

Затем подпишитесь на Mono. Фрагмент подписки на Mono см. в описании операции вставки. Полный пример кода доступен в репозитории GitHub под названием [CouchbaseToCosmosDB-AsyncInSpring](https://github.com/Azure-Samples/couchbaseTocosmosdb/tree/main/AsyncInSpring).

## <a name="couchbase-as-a-keyvalue-pair"></a>Couchbase в качестве пары "ключ-значение"

Это простой тип рабочей нагрузки, который допускает выполнение поиска вместо запросов. Для пар "ключ-значение" выполните указанные ниже действия.

1. Рекомендуется использовать "/ID" в качестве первичного ключа, что гарантирует возможность выполнения операции поиска непосредственно в определенной секции. Создайте коллекцию и укажите "/ID" в качестве ключа секции.

1. Полностью выключите индексирование. Так как вы будете выполнять операции поиска, нет смысла создавать дополнительную нагрузку за счет индексирования. Чтобы выключить индексирование, войдите на портал Azure и перейдите в учетную запись Azure Cosmos DB. Откройте **обозреватель данных**, выберите свою **базу данных** и **контейнер**. Откройте вкладку **Масштаб и параметры** и выберите **политику индексирования**. Текущая политика индексации выглядит следующим образом:
    
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
            "path": "/\"_etag\"/?"
        }
    ]
    }
   ````

   Замените указанную выше политику индексирования следующей:

   ```json
   {
    "indexingMode": "none",
    "automatic": false,
    "includedPaths": [],
    "excludedPaths": []
    }
   ```

1. Чтобы создать объект подключения, используйте следующий фрагмент кода. Объект соединения (который будет помещен в @Bean или сделан статическим):

   ```java
   ConnectionPolicy cp=new ConnectionPolicy();
   cp.connectionMode(ConnectionMode.DIRECT);
   
   if(client==null)
      client= CosmosClient.builder()
         .endpoint(Host)//(Host, PrimaryKey, dbName, collName).Builder()
          .connectionPolicy(cp)
          .key(PrimaryKey)
          .consistencyLevel(ConsistencyLevel.EVENTUAL)
          .build();
    
   container = client.getDatabase(_dbName).getContainer(_collName);
   ```

Теперь можно выполнить операции CRUD, как описано ниже.

### <a name="read-operation"></a>Операция чтения

Чтобы прочитать элемент, используйте следующий фрагмент кода:

```java        
CosmosItemRequestOptions ro=new CosmosItemRequestOptions();
ro.partitionKey(new PartitionKey(documentId));
CountDownLatch latch=new CountDownLatch(1);
      
var objCosmosItem= container.getItem(documentId, documentId);
Mono<CosmosItemResponse> objMono = objCosmosItem.read(ro);
objMono .subscribeOn(Schedulers.elastic())
        .subscribe(resourceResponse->
        {
           if(resourceResponse.item()!=null)
           {
              doc= resourceResponse.properties().toObject(UserModel.class);
           }
        },
        Throwable::printStackTrace,latch::countDown);
latch.await();
```

### <a name="insert-operation"></a>Операция вставки

Чтобы вставить элемент, можно выполнить следующий код:

```java
Mono<CosmosItemResponse> objMono= container.createItem(doc,ro);
```

Затем подпишитесь на Mono:

```java
CountDownLatch latch=new CountDownLatch(1);
objMono.subscribeOn(Schedulers.elastic())
      .subscribe(resourceResponse->
      {
         if(resourceResponse.statusCode()!=successStatus)
            {
               throw new RuntimeException(resourceResponse.toString());
            }
         },
      Throwable::printStackTrace,latch::countDown);
latch.await();
```

### <a name="upsert-operation"></a>Операция upsert

Чтобы обновить значение элемента, обратитесь к фрагменту кода ниже:

```java
Mono<CosmosItemResponse> obs= container.upsertItem(doc, ro);
```
Затем подпишитесь на Mono. Фрагмент подписки на Mono см. в описании операции вставки.

### <a name="delete-operation"></a>Операция удаления

Используйте следующий фрагмент кода для выполнения операции удаления:

```java
CosmosItem objItem= container.getItem(id, id);
Mono<CosmosItemResponse> objMono = objItem.delete(ro);
```

Затем подпишитесь на Mono. Фрагмент подписки на Mono см. в описании операции вставки. Полный пример кода доступен в репозитории GitHub под названием [CouchbaseToCosmosDB-AsyncKeyValue](https://github.com/Azure-Samples/couchbaseTocosmosdb/tree/main/AsyncKeyValue).

## <a name="data-migration"></a>Перенос данных

Существует два способа переноса данных.

* **Использование Фабрики данных Azure.** Это наиболее предпочтительный способ переноса данных. Настройте источник в качестве Couchbase, а приемник в качестве API SQL для Azure Cosmos DB. Подробные инструкции см. в статье [Соединитель Фабрики данных для Cosmos DB](../data-factory/connector-azure-cosmos-db.md).

* **Использование средства импорта данных Azure Cosmos DB.** Этот вариант рекомендуется использовать для миграции с использованием виртуальных машин с меньшим объемом данных. Подробные инструкции см. в статье [Средство импорта данных](./import-data.md).

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы выполнить тестирование производительности, см. статью [Тестирование производительности и масштабирования с помощью Azure Cosmos DB](./performance-testing.md).
* Чтобы оптимизировать код, см. статью [Советы по повышению производительности для Azure Cosmos DB](./performance-tips-async-java.md).
* Ознакомьтесь с Java Async V3 SDK и [справочником по пакету SDK](https://github.com/Azure/azure-cosmosdb-java/tree/v3) в репозитории GitHub.
