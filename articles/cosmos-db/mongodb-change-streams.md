---
title: Потоки изменений в API Azure Cosmos DB для MongoDB
description: Узнайте, как использовать потоки изменений в API Azure Cosmos DB для MongoDB, чтобы получать изменения, внесенные в данные.
author: Rodrigossz
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 03/02/2021
ms.author: rosouz
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 941fe8929b75fdebf187186ca7078b0ae1dd261c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101658529"
---
# <a name="change-streams-in-azure-cosmos-dbs-api-for-mongodb"></a>Потоки изменений в API Azure Cosmos DB для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Поддержка [канала изменений](change-feed.md) в API Azure Cosmos DB для MongoDB доступна с помощью API потоков изменений. С помощью API потоков изменений приложения могут получать изменения, внесенные в коллекцию или элементы в пределах одного сегмента. На основе полученных результатов можно будет выполнять дальнейшие действия. Изменения элементов в коллекции фиксируются в порядке времени их изменения, и в каждом ключе сегмента обязательно соблюдается порядок сортировки.

> [!NOTE]
> Чтобы использовать потоки изменений, создайте API Azure Cosmos DB для учетной записи MongoDB с сервером версии 3,6 или более поздней. При запуске примеров потока изменений для более ранней версии может отобразиться *нераспознанное имя этапа конвейера: $changeStream* ошибка.

## <a name="examples"></a>Примеры

В следующем примере показано, как получить потоки изменений для всех элементов в коллекции. В этом примере создается курсор для просмотра элементов при их вставке, обновлении или замене. Для получения потоков изменений требуются этап `$match`, этап `$project` и параметр `fullDocument`. Наблюдение за операциями удаления с помощью потоков изменений сейчас не поддерживается. В качестве обходного решения в удаляемые элементы можно добавить программный маркер. Например, можно добавить атрибут в элемент с именем "deleted". Если вы захотите удалить элемент, можно присвоить аргументу "deleted" значение `true` и задать срок жизни для элемента. Так как обновление "deleted" до `true` является изменением, оно будет отображено в потоке изменений.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
var cursor = db.coll.watch(
    [
        { $match: { "operationType": { $in: ["insert", "update", "replace"] } } },
        { $project: { "_id": 1, "fullDocument": 1, "ns": 1, "documentKey": 1 } }
    ],
    { fullDocument: "updateLookup" });

while (!cursor.isExhausted()) {
    if (cursor.hasNext()) {
        printjson(cursor.next());
    }
}
```

# <a name="c"></a>[C#](#tab/csharp)

```csharp
var pipeline = new EmptyPipelineDefinition<ChangeStreamDocument<BsonDocument>>()
    .Match(change => change.OperationType == ChangeStreamOperationType.Insert || change.OperationType == ChangeStreamOperationType.Update || change.OperationType == ChangeStreamOperationType.Replace)
    .AppendStage<ChangeStreamDocument<BsonDocument>, ChangeStreamDocument<BsonDocument>, BsonDocument>(
    "{ $project: { '_id': 1, 'fullDocument': 1, 'ns': 1, 'documentKey': 1 }}");

var options = new ChangeStreamOptions{
        FullDocument = ChangeStreamFullDocumentOption.UpdateLookup
    };

var enumerator = coll.Watch(pipeline, options).ToEnumerable().GetEnumerator();

while (enumerator.MoveNext()){
        Console.WriteLine(enumerator.Current);
    }

enumerator.Dispose();
```

# <a name="java"></a>[Java](#tab/java)

В следующем примере показано, как использовать функции потока изменений в Java. полный пример см. в этом [репозитории GitHub](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-changestream/blob/main/mongostream/src/main/java/com/azure/cosmos/mongostream/App.java). В этом примере также показано, как использовать `resumeAfter` метод для поиска всех изменений с момента последнего считывания. 

```java
Bson match = Aggregates.match(Filters.in("operationType", asList("update", "replace", "insert")));

// Pick the field you are most interested in
Bson project = Aggregates.project(fields(include("_id", "ns", "documentKey", "fullDocument")));

// This variable is for second example
BsonDocument resumeToken = null;

// Now time to build the pipeline
List<Bson> pipeline = Arrays.asList(match, project);

//#1 Simple example to seek changes

// Create cursor with update_lookup
MongoChangeStreamCursor<ChangeStreamDocument<org.bson.Document>> cursor = collection.watch(pipeline)
        .fullDocument(FullDocument.UPDATE_LOOKUP).cursor();

Document document = new Document("name", "doc-in-step-1-" + Math.random());
collection.insertOne(document);

while (cursor.hasNext()) {
    // There you go, we got the change document.
    ChangeStreamDocument<Document> csDoc = cursor.next();

    // Let is pick the token which will help us resuming
    // You can save this token in any persistent storage and retrieve it later
    resumeToken = csDoc.getResumeToken();
    //Printing the token
    System.out.println(resumeToken);
    
    //Printing the document.
    System.out.println(csDoc.getFullDocument());
    //This break is intentional but in real project feel free to remove it.
    break;
}

cursor.close();

```
---

## <a name="changes-within-a-single-shard"></a>Изменения в пределах одного сегмента

В следующем примере показано, как получить изменения элементов в пределах одного сегмента. В этом примере показано получение изменений элементов, ключ сегмента которых равен "a", а значение ключа сегмента равно "1". Допускается ситуация, когда разные клиенты могут одновременно считывать изменения из разных сегментов.

```javascript
var cursor = db.coll.watch(
    [
        { 
            $match: { 
                $and: [
                    { "fullDocument.a": 1 }, 
                    { "operationType": { $in: ["insert", "update", "replace"] } }
                ]
            }
        },
        { $project: { "_id": 1, "fullDocument": 1, "ns": 1, "documentKey": 1} }
    ],
    { fullDocument: "updateLookup" });

```

## <a name="current-limitations"></a>Текущие ограничения

При использовании потоков изменений действуют следующие ограничения.

* Свойства `operationType` и `updateDescription` пока не поддерживаются в выходном документе.
* В настоящее время поддерживаются типы операций `insert`, `update` и `replace`. Однако операция удаления или другие события пока не поддерживаются.

В связи с этими ограничениями этап $match, этап $project и параметры fullDocument являются обязательными, как показано в предыдущих примерах.

В отличие от канала изменений, в API Cosmos DB для SQL нет отдельной [библиотеки обработчика канала изменений](change-feed-processor.md) для использования потоков изменений и нет необходимости в контейнере аренды. В настоящее время поддержка [триггеров Функций Azure](change-feed-functions.md) для обработки потоков изменений отсутствует.

## <a name="error-handling"></a>Обработка ошибок

При использовании потоков изменений поддерживаются следующие коды ошибок и сообщения.

* **Код ошибки HTTP: 16500** — когда поток изменений регулируется, он возвращает пустую страницу.

* **NamespaceNotFound (OperationType Invalidate)**  — при запуске потока изменений в несуществующей коллекции или при удалении коллекции возвращается ошибка `NamespaceNotFound`. Так как свойство `operationType` не может быть возвращено в выходном документе, вместо ошибки `operationType Invalidate` возвращается ошибка `NamespaceNotFound`.

## <a name="next-steps"></a>Дальнейшие действия

* [Завершение срока действия данных с помощью API Azure Cosmos DB для MongoDB](mongodb-time-to-live.md)
* [Индексирование в API Azure Cosmos DB для MongoDB](mongodb-indexing.md)
