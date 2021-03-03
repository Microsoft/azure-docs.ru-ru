---
title: Использование транзакций с несколькими документами в Azure Cosmos DB API для MongoDB
description: Узнайте, как создать пример приложения Mongo Shell, которое может выполнять многодокументную транзакцию (семантику ALL-или Nothing) в фиксированной коллекции в Azure Cosmos DB API для MongoDB 4,0.
author: gahl-levy
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 03/02/2021
ms.author: gahllevy
ms.openlocfilehash: 4d7dcc829f25b7f1b7c6cb6b1d13a664d301bfe6
ms.sourcegitcommit: b4647f06c0953435af3cb24baaf6d15a5a761a9c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/02/2021
ms.locfileid: "101663654"
---
# <a name="use-multi-document-transactions-in-azure-cosmos-db-api-for-mongodb"></a>Использование транзакций с несколькими документами в Azure Cosmos DB API для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

В этой статье вы создадите приложение оболочки Mongo, которое выполняет многодокументную транзакцию в фиксированной коллекции в Azure Cosmos DB API для MongoDB с сервером версии 4,0.

## <a name="what-are-multi-document-transactions"></a>Что такое транзакции с несколькими документами?

В Azure Cosmos DB API для MongoDB операции над одним документом являются атомарными. Транзакции с несколькими документами позволяют приложениям выполнять атомарные операции в нескольких документах. Она предлагает семантике "все или ничего" для операций. При фиксации изменения, внесенные в транзакции, сохраняются, и если транзакция завершается ошибкой, все изменения в транзакции отбрасываются.

Транзакции с несколькими документами следуют семантике **ACID** :

* Атомарность. все операции обрабатываются как один
* Согласованность: зафиксированные данные допустимы
* Изоляция: изолированная от других операций
* Устойчивость: данные транзакций сохраняются, когда клиент сообщает о них

## <a name="requirements"></a>Требования

Транзакции с несколькими документами поддерживаются в несегментированной коллекции в версии 4,0. Транзакции с несколькими документами не поддерживаются в коллекциях или в сегментированных коллекциях.

Все драйверы, поддерживающие протокол проводной сети версии 4,0 или выше, будут поддерживать Azure Cosmos DB API для транзакций с несколькими документами MongoDB.

## <a name="run-multi-document-transactions-in-mongodb-shell"></a>Выполнение транзакций с несколькими документами в оболочке MongoDB

1. Откройте командную строку и перейдите в каталог, где установлена оболочка Mongo версии 4,0 и выше:

   ```powershell
   cd <path_to_mongo_shell_>
   ```

2. Создайте скрипт оболочки Mongo *connect_friends.js* и добавьте следующее содержимое.

   ```javascript
   // insert data into friends collection
   db.getMongo().getDB("users").friends.insert({name:"Tom"})
   db.getMongo().getDB("users").friends.insert({name:"Mike"})
   // start transaction
   var session = db.getMongo().startSession();
   var friendsCollection = session.getDatabase("users").friends;
   session.startTransaction();
   // operations in transaction
   try {
       friendsCollection.updateOne({ name: "Tom" }, { $set: { friendOf: "Mike" } } );
       friendsCollection.updateOne({ name: "Mike" }, { $set: { friendOf: "Tom" } } );
   } catch (error) {
       // abort transaction on error
       session.abortTransaction();
       throw error;
   }

    // commit transaction
    session.commitTransaction();

    ```

3. Выполните следующую команду, чтобы выполнить транзакцию с несколькими документами. Узел, порт, пользователь и ключ можно найти в портал Azure.

   ```powershell
   mongo "<HOST>:<PORT>" -u "<USER>" -p "KEY" --ssl connect_friends.js
   ```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте о новых возможностях [Azure Cosmos DB API для MongoDB 4,0](mongodb-feature-support-40.md)
