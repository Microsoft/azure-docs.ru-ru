---
title: Создание приложения Node.js с помощью API Azure Cosmos DB для MongoDB
description: Руководство, в котором создается оперативная база данных с помощью API-интерфейса Azure Cosmos DB для MongoDB.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: sample
ms.date: 12/26/2018
author: sivethe
ms.author: sivethe
ms.custom: devx-track-js
ms.openlocfilehash: c2117c535cca679d3a7a3b75491bf60ab9567ceb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93098218"
---
# <a name="build-an-app-using-nodejs-and-azure-cosmos-dbs-api-for-mongodb"></a>Создание приложения с помощью Node.js и API Azure Cosmos DB для MongoDB 
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!div class="op_single_selector"]
> * [.NET](sql-api-get-started.md)
> * [.NET Core](./sql-api-get-started.md)
> * [Java](./create-sql-api-java.md)
> * [Node.js для MongoDB](mongodb-samples.md)
> * [Node.js](sql-api-nodejs-get-started.md)
>

В этом примере показано, как создать консольное приложение с использованием Node.js и API Azure Cosmos DB для MongoDB

Чтобы использовать этот пример, необходимо выполнить следующие действия.

* [Создайте](create-mongodb-dotnet.md#create-account) учетную запись Cosmos и настройте ее на использование API Azure Cosmos DB для MongoDB.
* Получите сведения о [строке подключения](connect-mongodb-account.md).

## <a name="create-the-app"></a>Создайте приложение

1. Создайте файл *app.js*, а затем скопируйте и вставьте в него приведенный ниже код.

    ```javascript
    var MongoClient = require('mongodb').MongoClient;
    var assert = require('assert');
    var ObjectId = require('mongodb').ObjectID;
    var url = 'mongodb://<username>:<password>@<endpoint>.documents.azure.com:10255/?ssl=true';

    var insertDocument = function(db, callback) {
    db.collection('families').insertOne( {
            "id": "AndersenFamily",
            "lastName": "Andersen",
            "parents": [
                { "firstName": "Thomas" },
                { "firstName": "Mary Kay" }
            ],
            "children": [
                { "firstName": "John", "gender": "male", "grade": 7 }
            ],
            "pets": [
                { "givenName": "Fluffy" }
            ],
            "address": { "country": "USA", "state": "WA", "city": "Seattle" }
        }, function(err, result) {
        assert.equal(err, null);
        console.log("Inserted a document into the families collection.");
        callback();
    });
    };
    
    var findFamilies = function(db, callback) {
    var cursor =db.collection('families').find( );
    cursor.each(function(err, doc) {
        assert.equal(err, null);
        if (doc != null) {
            console.dir(doc);
        } else {
            callback();
        }
    });
    };
    
    var updateFamilies = function(db, callback) {
    db.collection('families').updateOne(
        { "lastName" : "Andersen" },
        {
            $set: { "pets": [
                { "givenName": "Fluffy" },
                { "givenName": "Rocky"}
            ] },
            $currentDate: { "lastModified": true }
        }, function(err, results) {
        console.log(results);
        callback();
    });
    };
    
    var removeFamilies = function(db, callback) {
    db.collection('families').deleteMany(
        { "lastName": "Andersen" },
        function(err, results) {
            console.log(results);
            callback();
        }
    );
    };
    
    MongoClient.connect(url, function(err, client) {
    assert.equal(null, err);
    var db = client.db('familiesdb');
    insertDocument(db, function() {
        findFamilies(db, function() {
        updateFamilies(db, function() {
            removeFamilies(db, function() {
                client.close();
            });
        });
        });
    });
    });
    ```
    
    **Необязательно.** При использовании **драйвера MongoDB Node.js 2.2** замените следующий фрагмент кода:

    Исходный код:

    ```javascript
    MongoClient.connect(url, function(err, client) {
    assert.equal(null, err);
    var db = client.db('familiesdb');
    insertDocument(db, function() {
        findFamilies(db, function() {
        updateFamilies(db, function() {
            removeFamilies(db, function() {
                client.close();
            });
        });
        });
    });
    });
    ```
    
    Замените исходный код этим:

    ```javascript
    MongoClient.connect(url, function(err, db) {
    assert.equal(null, err);
    insertDocument(db, function() {
        findFamilies(db, function() {
        updateFamilies(db, function() {
            removeFamilies(db, function() {
                db.close();
            });
        });
        });
    });
    });
    ```
    
2. Замените следующие переменные в файле *app.js* параметрами учетной записи (узнайте, как найти [строку подключения](connect-mongodb-account.md)):

    > [!IMPORTANT]
    > **Драйвер MongoDB Node.js 3.0** требует кодирования специальных символов в пароле Cosmos DB. Кодируйте символы '=' как %3D.
    >
    > Пример: пароль *jm1HbNdLg5zxEuyD86ajvINRFrFCUX0bIWP15ATK3BvSv ==* кодируется в *jm1HbNdLg5zxEuyD86ajvINRFrFCUX0bIWP15ATK3BvSv%3D%3D*
    >
    > **Драйвер MongoDB Node.js 2.2** не требует кодирования специальных символов в пароле Cosmos DB.
    >
    >
   
    ```javascript
    var url = 'mongodb://<endpoint>:<password>@<endpoint>.documents.azure.com:10255/?ssl=true';
    ```
     
3. Откройте удобный для вас терминал, выполните команду **npm install mongodb --save**, а затем запустите приложение с помощью **node app.js**.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [использовать Studio 3T](mongodb-mongochef.md) с API Azure Cosmos DB для MongoDB.
- Узнайте, как [использовать Robo 3T](mongodb-robomongo.md) с API Azure Cosmos DB для MongoDB.
- Ознакомьтесь с [примерами](mongodb-samples.md) MongoDB с API Azure Cosmos DB для MongoDB.