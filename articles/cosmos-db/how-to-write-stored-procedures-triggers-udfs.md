---
title: Запись хранимых процедур, триггеров и пользовательских функций в Azure Cosmos DB
description: Сведения о том, как определить хранимые процедуры, триггеры и определяемые пользователем функции в Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 06/16/2020
ms.author: tisande
ms.custom: devx-track-js
ms.openlocfilehash: 7600d8aa2f78e06ea4046273635fdbba18042010
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98028868"
---
# <a name="how-to-write-stored-procedures-triggers-and-user-defined-functions-in-azure-cosmos-db"></a>Как писать хранимые процедуры, триггеры и определяемые пользователем функции в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB обеспечивает встроенное в язык транзакционное выполнение JavaScript, которое позволяет писать **хранимые процедуры**, **триггеры** и **определяемые пользователем функции**. При использовании API SQL в Azure Cosmos DB можно определить хранимые процедуры, триггеры и определяемые пользователем функции на языке JavaScript. Вы можете написать свою логику на JavaScript и выполнить ее в ядре СУБД. Вы можете создавать и выполнять триггеры, хранимые процедуры и определяемые пользователем функции с помощью [портала Azure](https://portal.azure.com/), [API запросов с интегрированным языком JavaScript в Azure Cosmos DB](javascript-query-api.md) и [клиентских пакетов SDK API SQL в Cosmos DB](sql-api-dotnet-samples.md). 

Чтобы вызвать хранимую процедуру, триггер и определяемую пользователем функцию, вам необходимо зарегистрировать ее. Дополнительные сведения см. в статье о том, как [зарегистрировать и использовать хранимые процедуры, триггеры и определяемые пользователем функции в Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md).

> [!NOTE]
> Для секционированных контейнеров при выполнении хранимой процедуры в параметрах запроса необходимо указать значение ключа секции. Хранимые процедуры всегда ограничиваются ключом секции. Элементы с другим значением ключа секции не будут видны хранимой процедуре. Это также применяется к триггерам.
> [!Tip]
> Cosmos поддерживает развертывание контейнеров с хранимыми процедурами, триггерами и определяемыми пользователем функциями. Дополнительные сведения см. в статье [Создание контейнера Azure Cosmos DB с функциональностью на стороне сервера.](./manage-with-templates.md#create-sproc)

## <a name="how-to-write-stored-procedures"></a><a id="stored-procedures"></a>Как написать хранимые процедуры

Хранимые процедуры пишутся с помощью языка JavaScript. С их помощью можно создавать, обновлять, читать, запрашивать и удалять элементы внутри контейнера Cosmos в Azure. Хранимые процедуры регистрируются в коллекциях и могут работать с любым документом и вложением, существующим в этой коллекции.

Давайте начнем с простой хранимой процедуры, которая возвращает ответ "Hello World".

```javascript
var helloWorldStoredProc = {
    id: "helloWorld",
    serverScript: function () {
        var context = getContext();
        var response = context.getResponse();

        response.setBody("Hello, World");
    }
}
```

Объект контекста предоставляет доступ ко всем операциям, которые можно выполнить в Azure Cosmos DB, а также доступ к объектам запросов и ответов. В этом примере вы применяете объект ответа, чтобы задать текст ответа для отправки клиенту.

После написания хранимую процедуру нужно зарегистрировать с помощью коллекции. Дополнительные сведения см. в разделе о том, [как выполнять хранимые процедуры](how-to-use-stored-procedures-triggers-udfs.md#stored-procedures).

### <a name="create-an-item-using-stored-procedure"></a><a id="create-an-item"></a>Создание элемента с помощью хранимой процедуры

При создании элемента с помощью хранимой процедуры этот элемент вставляется в контейнер Azure Cosmos и возвращается идентификатор вновь созданного элемента. Создание элемента является асинхронной операцией и зависит от функций обратного вызова JavaScript. Функция обратного вызова имеет два параметра: один для объекта ошибки в случае, если операция не выполнится, и один для возвращаемого значения, то есть — созданного объекта. Внутри функции обратного вызова можно либо обработать исключение, либо вызвать ошибку. Если обратного вызова не предусмотрено и возникла ошибка, то среда выполнения Azure Cosmos DB выдаст ошибку.

Хранимая процедура также включает параметр для установки описания. Это логическое значение. Если параметр имеет значение true, а описание отсутствует, хранимая процедура приведет к возникновению исключения. В противном случае остальная часть хранимой процедуры продолжит выполняться.

В следующем примере хранимая процедура принимает массив новых элементов Azure Cosmos в качестве входных данных, вставляет его в контейнер Azure Cosmos и возвращает количество вставленных элементов. В этом примере мы используем образец ToDoList из статьи [Краткое руководство. Создание веб-приложения .NET при помощи Azure Cosmos DB с использованием API SQL и портала Azure](create-sql-api-dotnet.md).

```javascript
function createToDoItems(items) {
    var collection = getContext().getCollection();
    var collectionLink = collection.getSelfLink();
    var count = 0;

    if (!items) throw new Error("The array is undefined or null.");

    var numItems = items.length;

    if (numItems == 0) {
        getContext().getResponse().setBody(0);
        return;
    }

    tryCreate(items[count], callback);

    function tryCreate(item, callback) {
        var options = { disableAutomaticIdGeneration: false };

        var isAccepted = collection.createDocument(collectionLink, item, options, callback);

        if (!isAccepted) getContext().getResponse().setBody(count);
    }

    function callback(err, item, options) {
        if (err) throw err;
        count++;
        if (count >= numItems) {
            getContext().getResponse().setBody(count);
        } else {
            tryCreate(items[count], callback);
        }
    }
}
```

### <a name="arrays-as-input-parameters-for-stored-procedures"></a>Массивы в качестве входных параметров для хранимых процедур

При определении хранимой процедуры на портале Azure входные параметры всегда отправляются в хранимую процедуру в виде строки. Даже если передать массив строк в качестве входных данных, массив преобразуется в строку и отправляются в хранимую процедуру. Чтобы обойти эту проблему, можно определить функцию в хранимой процедуре для анализа строки как массива. В следующем коде показано, как анализировать входной параметр строки как массив:

```javascript
function sample(arr) {
    if (typeof arr === "string") arr = JSON.parse(arr);

    arr.forEach(function(a) {
        // do something here
        console.log(a);
    });
}
```

### <a name="transactions-within-stored-procedures"></a><a id="transactions"></a>Транзакции в хранимых процедурах

Вы можете реализовать транзакции для элементов в контейнере с помощью хранимой процедуры. В следующем примере транзакции в игровом приложении для фэнтези-футбола используются для обмена игроками между двумя командами за одну операцию. Хранимая процедура пытается прочитать два элемента Azure Cosmos, каждый из которых соответствует идентификаторам игроков, переданным в качестве аргумента. Если оба игрока найдены, то хранимая процедура обновляет элементы, меняя их команды. При обнаружении ошибок вызывается исключение JavaScript, которое неявно отменяет транзакцию.

```javascript
// JavaScript source code
function tradePlayers(playerId1, playerId2) {
    var context = getContext();
    var container = context.getCollection();
    var response = context.getResponse();

    var player1Document, player2Document;

    // query for players
    var filterQuery =
    {
        'query' : 'SELECT * FROM Players p where p.id = @playerId1',
        'parameters' : [{'name':'@playerId1', 'value':playerId1}] 
    };

    var accept = container.queryDocuments(container.getSelfLink(), filterQuery, {},
        function (err, items, responseOptions) {
            if (err) throw new Error("Error" + err.message);

            if (items.length != 1) throw "Unable to find both names";
            player1Item = items[0];

            var filterQuery2 =
            {
                'query' : 'SELECT * FROM Players p where p.id = @playerId2',
                'parameters' : [{'name':'@playerId2', 'value':playerId2}]
            };
            var accept2 = container.queryDocuments(container.getSelfLink(), filterQuery2, {},
                function (err2, items2, responseOptions2) {
                    if (err2) throw new Error("Error" + err2.message);
                    if (items2.length != 1) throw "Unable to find both names";
                    player2Item = items2[0];
                    swapTeams(player1Item, player2Item);
                    return;
                });
            if (!accept2) throw "Unable to read player details, abort ";
        });

    if (!accept) throw "Unable to read player details, abort ";

    // swap the two players’ teams
    function swapTeams(player1, player2) {
        var player2NewTeam = player1.team;
        player1.team = player2.team;
        player2.team = player2NewTeam;

        var accept = container.replaceDocument(player1._self, player1,
            function (err, itemReplaced) {
                if (err) throw "Unable to update player 1, abort ";

                var accept2 = container.replaceDocument(player2._self, player2,
                    function (err2, itemReplaced2) {
                        if (err) throw "Unable to update player 2, abort"
                    });

                if (!accept2) throw "Unable to update player 2, abort";
            });

        if (!accept) throw "Unable to update player 1, abort";
    }
}
```

### <a name="bounded-execution-within-stored-procedures"></a><a id="bounded-execution"></a>Ограниченное выполнение в хранимых процедурах

Ниже приведен пример хранимой процедуры, которая массово импортирует элементы в контейнер Cosmos в Azure. Хранимая процедура обрабатывает ограниченное выполнение, проверяя логическое возвращаемое значение из `createDocument`, а затем использует подсчет элементов, помещенных в каждом вызове хранимой процедуры, чтобы отслеживать и возобновлять выполнение пакетного задания.

```javascript
function bulkImport(items) {
    var container = getContext().getCollection();
    var containerLink = container.getSelfLink();

    // The count of imported items, also used as current item index.
    var count = 0;

    // Validate input.
    if (!items) throw new Error("The array is undefined or null.");

    var itemsLength = items.length;
    if (itemsLength == 0) {
        getContext().getResponse().setBody(0);
    }

    // Call the create API to create an item.
    tryCreate(items[count], callback);

    // Note that there are 2 exit conditions:
    // 1) The createDocument request was not accepted.
    //    In this case the callback will not be called, we just call setBody and we are done.
    // 2) The callback was called items.length times.
    //    In this case all items were created and we don’t need to call tryCreate anymore. Just call setBody and we are done.
    function tryCreate(item, callback) {
        var isAccepted = container.createDocument(containerLink, item, callback);

        // If the request was accepted, callback will be called.
        // Otherwise report current count back to the client,
        // which will call the script again with remaining set of items.
        if (!isAccepted) getContext().getResponse().setBody(count);
    }

    // This is called when container.createDocument is done in order to process the result.
    function callback(err, item, options) {
        if (err) throw err;

        // One more item has been inserted, increment the count.
        count++;

        if (count >= itemsLength) {
            // If we created all items, we are done. Just set the response.
            getContext().getResponse().setBody(count);
        } else {
            // Create next document.
            tryCreate(items[count], callback);
        }
    }
}
```

### <a name="async-await-with-stored-procedures"></a><a id="async-promises"></a>Асинхронный ожидание с помощью хранимых процедур

Ниже приведен пример хранимой процедуры, которая использует async-await с обещания с помощью вспомогательной функции. Хранимая процедура запрашивает элемент и заменяет его.

```javascript
function async_sample() {
    const ERROR_CODE = {
        NotAccepted: 429
    };

    const asyncHelper = {
        queryDocuments(sqlQuery, options) {
            return new Promise((resolve, reject) => {
                const isAccepted = __.queryDocuments(__.getSelfLink(), sqlQuery, options, (err, feed, options) => {
                    if (err) reject(err);
                    resolve({ feed, options });
                });
                if (!isAccepted) reject(new Error(ERROR_CODE.NotAccepted, "replaceDocument was not accepted."));
            });
        },

        replaceDocument(doc) {
            return new Promise((resolve, reject) => {
                const isAccepted = __.replaceDocument(doc._self, doc, (err, result, options) => {
                    if (err) reject(err);
                    resolve({ result, options });
                });
                if (!isAccepted) reject(new Error(ERROR_CODE.NotAccepted, "replaceDocument was not accepted."));
            });
        }
    };

    async function main() {
        let continuation;
        do {
            let { feed, options } = await asyncHelper.queryDocuments("SELECT * from c", { continuation });

            for (let doc of feed) {
                doc.newProp = 1;
                await asyncHelper.replaceDocument(doc);
            }

            continuation = options.continuation;
        } while (continuation);
    }

    main().catch(err => getContext().abort(err));
}
```

## <a name="how-to-write-triggers"></a><a id="triggers"></a>Как записать триггеры

Azure Cosmos DB поддерживает триггеры предварительного и последующего выполнения. Триггеры предварительного выполнения выполняются до изменения элемента базы данных, а триггеры последующего — после изменения элемента базы данных. Триггеры не выполняются автоматически, они должны быть указаны для каждой операции базы данных, в которой их необходимо выполнить. После определения триггера необходимо [зарегистрировать и вызвать предварительный триггер](how-to-use-stored-procedures-triggers-udfs.md#pre-triggers) с помощью Azure Cosmos DB пакетов SDK.

### <a name="pre-triggers"></a><a id="pre-triggers"></a>Предварительные триггеры

В следующем примере показано, как триггер предварительного выполнения используется для проверки свойств созданного элемента Azure Cosmos. В этом примере мы используем образец ToDoList, описанный в статье [Краткое руководство. Создание веб-приложения .NET при помощи Azure Cosmos DB с использованием API SQL и портала Azure](create-sql-api-dotnet.md), чтобы добавить свойство метки времени в новый добавленный элемент при отсутствии в нем другого свойства.

```javascript
function validateToDoItemTimestamp() {
    var context = getContext();
    var request = context.getRequest();

    // item to be created in the current operation
    var itemToCreate = request.getBody();

    // validate properties
    if (!("timestamp" in itemToCreate)) {
        var ts = new Date();
        itemToCreate["timestamp"] = ts.getTime();
    }

    // update the item that will be created
    request.setBody(itemToCreate);
}
```

Триггеры со срабатыванием до наступления события не могут иметь никаких входных параметров. Объект запроса в триггере используется для управления сообщением запроса, связанным с операцией. В предыдущем примере триггер предварительного выполнения запускается при создании элемента Azure Cosmos, а текст сообщения запроса содержит элемент, который будет создан в формате JSON.

При регистрации триггеров можно указать операции, с помощью которых их можно запустить. Этот триггер нужно создать с помощью значения `TriggerOperation` функции `TriggerOperation.Create`, что означает, что использование триггера в операции замены, как показано в следующем коде, недопустимо.

Ознакомьтесь с примерами регистрации и вызова триггера [предварительного](how-to-use-stored-procedures-triggers-udfs.md#pre-triggers) и [последующего выполнения](how-to-use-stored-procedures-triggers-udfs.md#post-triggers). 

### <a name="post-triggers"></a><a id="post-triggers"></a>Триггеры после

В примере ниже показан триггер последующего выполнения. Этот триггер запрашивает элемент метаданных и обновляет его дополнительными сведениями о вновь созданном элементе.


```javascript
function updateMetadata() {
var context = getContext();
var container = context.getCollection();
var response = context.getResponse();

// item that was created
var createdItem = response.getBody();

// query for metadata document
var filterQuery = 'SELECT * FROM root r WHERE r.id = "_metadata"';
var accept = container.queryDocuments(container.getSelfLink(), filterQuery,
    updateMetadataCallback);
if(!accept) throw "Unable to update metadata, abort";

function updateMetadataCallback(err, items, responseOptions) {
    if(err) throw new Error("Error" + err.message);
        if(items.length != 1) throw 'Unable to find metadata document';

        var metadataItem = items[0];

        // update metadata
        metadataItem.createdItems += 1;
        metadataItem.createdNames += " " + createdItem.id;
        var accept = container.replaceDocument(metadataItem._self,
            metadataItem, function(err, itemReplaced) {
                    if(err) throw "Unable to update metadata, abort";
            });
        if(!accept) throw "Unable to update metadata, abort";
        return;
}
```

Важно отметить одну вещь, а именно — транзакционное выполнение триггеров в Azure Cosmos DB. Триггер последующего выполнения запускается как часть транзакции базового элемента. Исключение во время выполнения такого триггера приводит к сбою всей транзакции. Любая фиксация при этом откатывается и возвращается исключение.

Ознакомьтесь с примерами регистрации и вызова триггера [предварительного](how-to-use-stored-procedures-triggers-udfs.md#pre-triggers) и [последующего выполнения](how-to-use-stored-procedures-triggers-udfs.md#post-triggers). 

## <a name="how-to-write-user-defined-functions"></a><a id="udfs"></a>Как записать определяемые пользователем функции

В следующем примере создается определяемая пользователем функция для расчета подоходного налога для различных доходов. Эта функция затем будет использоваться в запросе. В целях этого примера предположим, что имеется контейнер с именем "Доходы" со свойствами, как показано ниже:

```json
{
   "name": "Satya Nadella",
   "country": "USA",
   "income": 70000
}
```

Ниже приведено определение функции для расчета подоходного налога для различных доходов:

```javascript
function tax(income) {

        if(income == undefined)
            throw 'no input';

        if (income < 1000)
            return income * 0.1;
        else if (income < 10000)
            return income * 0.2;
        else
            return income * 0.4;
    }
```

Примеры регистрации и использования определяемой пользователем функции см. в разделе о [работе с такой функцией](how-to-use-stored-procedures-triggers-udfs.md#udfs).

## <a name="logging"></a>Ведение журнала 

При использовании хранимой процедуры, триггеров или определяемых пользователем функций можно регистрировать шаги с помощью `console.log()` команды. Эта команда повлияет на строку для отладки, если `EnableScriptLogging` для задано значение true, как показано в следующем примере:

```javascript
var response = await client.ExecuteStoredProcedureAsync(
document.SelfLink,
new RequestOptions { EnableScriptLogging = true } );
Console.WriteLine(response.ScriptLog);
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о том, как записать или использовать хранимые процедуры, триггеры и определяемые пользователем функции в Azure Cosmos DB, см. в статьях ниже:

* [Как зарегистрировать и использовать хранимые процедуры, триггеры и определяемые пользователем функции в Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md)

* [Как записывать хранимые процедуры и триггеры в Azure Cosmos DB с помощью API запросов JavaScript](how-to-write-javascript-query-api.md)

* [Working with Azure Cosmos DB stored procedures, triggers, and user-defined functions (Работа с хранимыми процедурами, триггерами и определяемыми пользователем функциями в Azure Cosmos DB)](stored-procedures-triggers-udfs.md)

* [API для выполнения запросов JavaScript в Azure Cosmos DB](javascript-query-api.md)
