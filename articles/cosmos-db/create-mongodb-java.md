---
title: Краткое руководство. Создание веб-приложения с помощью API Azure Cosmos DB для MongoDB и пакета SDK для Java
description: Узнайте, как создать пример кода Java, который можно использовать для подключения и выполнения запросов c помощью API Azure Cosmos DB для MongoDB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: java
ms.topic: quickstart
ms.date: 12/26/2018
ms.custom: seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: f91cf5103fa8bf324fe372d2f12736a426fe8aed
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101656539"
---
# <a name="quickstart-create-a-console-app-with-java-and-the-mongodb-api-in-azure-cosmos-db"></a>Краткое руководство. Создание консольного приложения с использованием Java и API MongoDB в Azure Cosmos DB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](./mongodb-introduction.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

В этом кратком руководстве вы создадите учетную запись API Azure Cosmos DB для MongoDB и будете управлять ею из портала Azure, после чего добавите данные с помощью приложения Java SDK, клонированного из GitHub. Azure Cosmos DB — это служба многомодельной базы данных, позволяющая быстро создать и запрашивать документы, таблицы, пары "ключ-значение", а также графовые базы данных, используя возможности глобального распределения и горизонтального масштабирования.

## <a name="prerequisites"></a>Предварительные требования
- Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Или [воспользуйтесь пробной версией Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) без подписки Azure. Вы также можете воспользоваться [эмулятором Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) со строкой подключения `.mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true`.
- [Комплект SDK для Java (JDK) версии 8](https://www.azul.com/downloads/azure-only/zulu/?&version=java-8-lts&architecture=x86-64-bit&package=jdk). 
- [Maven](https://maven.apache.org/download.cgi). Или выполните команду `apt-get install maven`, чтобы установить Maven.
- [Git](https://git-scm.com/downloads). 

## <a name="create-a-database-account"></a>Создание учетной записи базы данных

[!INCLUDE [mongodb-create-dbaccount](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

## <a name="add-a-collection"></a>Добавление коллекции

Присвойте новой базе данных имя **db**, а новой коллекции — **coll**.

[!INCLUDE [cosmos-db-create-collection](../../includes/cosmos-db-mongodb-create-collection.md)] 

## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Теперь клонируйте приложение из GitHub. Задайте строку подключения и выполните ее. Вы узнаете, как можно упростить работу с данными программным способом. 

1. Откройте командную строку, создайте папку git-samples, а затем закройте окно командной строки.

    ```bash
    md "C:\git-samples"
    ```

2. Откройте окно терминала git, например git bash, и выполните команду `cd`, чтобы перейти в новую папку для установки примера приложения.

    ```bash
    cd "C:\git-samples"
    ```

3. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере.

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-getting-started.git
    ```

3. Затем откройте код в любом удобном редакторе. 

## <a name="review-the-code"></a>Просмотр кода

Это необязательный шаг. Если вы хотите узнать, как создать в коде ресурсы базы данных, изучите приведенные ниже фрагменты кода. Если вас это не интересует, можете сразу переходить к разделу [Обновление строки подключения](#update-your-connection-string). 

Приведенные ниже фрагменты кода взяты из файла *Program.java*.

Это консольное приложение использует [драйвер Java для MongoDB](https://docs.mongodb.com/ecosystem/drivers/java/). 

* Инициализация экземпляра DocumentClient.

    ```java
    MongoClientURI uri = new MongoClientURI("FILLME");`

    MongoClient mongoClient = new MongoClient(uri);            
    ```

* Создание базы данных и коллекции.

    ```java
    MongoDatabase database = mongoClient.getDatabase("db");

    MongoCollection<Document> collection = database.getCollection("coll");
    ```

* Вставка документов с использованием `MongoCollection.insertOne`.

    ```java
    Document document = new Document("fruit", "apple")
    collection.insertOne(document);
    ```

* Выполнение запросов с использованием `MongoCollection.find`.

    ```java
    Document queryResult = collection.find(Filters.eq("fruit", "apple")).first();
    System.out.println(queryResult.toJson());       
    ```

## <a name="update-your-connection-string"></a>Обновление строки подключения

Теперь вернитесь на портал Azure, чтобы получить данные строки подключения. Скопируйте эти данные в приложение.

1. В учетной записи Azure Cosmos DB щелкните **Быстрый запуск**, выберите **Java**, а затем скопируйте строку подключения в буфер обмена.

2. Откройте файл *Program.java*, замените аргумент конструктора MongoClientURI строкой подключения. Теперь приложение со всеми сведениями, необходимыми для взаимодействия с Azure Cosmos DB, обновлено. 
    
## <a name="run-the-console-app"></a>Запуск консольного приложения

1. В окне терминала запустите `mvn package`, чтобы установить необходимые модули npm.

2. В окне терминала запустите `mvn exec:java -D exec.mainClass=GetStarted.Program`, чтобы запустить приложение Java.

Теперь вы можете использовать [Robomongo](mongodb-robomongo.md) / [Studio 3T](mongodb-mongochef.md), чтобы запрашивать и изменять новые данные, а также работать с ними.

## <a name="review-slas-in-the-azure-portal"></a>Просмотр соглашений об уровне обслуживания на портале Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать учетную запись API Azure Cosmos DB для MongoDB, добавить базу данных и контейнер с помощью обозревателя данных, а также добавить данные с помощью консольного приложения Java. Теперь вы можете импортировать дополнительные данные в базу данных Cosmos. 

> [!div class="nextstepaction"]
> [Перенос данных MongoDB в Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)