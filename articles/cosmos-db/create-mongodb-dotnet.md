---
title: Создание веб-приложения с помощью API Azure Cosmos DB для MongoDB и пакета SDK для .NET
description: В этой статье представлен пример кода .NET, который можно использовать для подключения и выполнения запросов c помощью API Azure Cosmos DB для MongoDB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 10/15/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 65ceca7ee0a6482e489ca768b9b33981d313c3db
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101658035"
---
# <a name="quickstart-build-a-net-web-app-using-azure-cosmos-dbs-api-for-mongodb"></a>Краткое руководство. Создание веб-приложения .NET с использованием API Azure Cosmos DB для MongoDB 
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](./mongodb-introduction.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

Azure Cosmos DB — это быстрая база данных NoSQL от Майкрософт с открытыми API для использования в любом масштабе. Вы можете быстро создавать и запрашивать документы, пары "ключ-значение" и графовые базы данных, используя преимущества глобального распределения и горизонтального масштабирования Cosmos DB. 

В этом кратком руководстве показано, как создать учетную запись Cosmos с [API Azure Cosmos DB для MongoDB](mongodb-introduction.md). Затем вы можете развернуть веб-приложение списка задач, созданное с помощью [драйвера .NET MongoDB](https://docs.mongodb.com/ecosystem/drivers/csharp/).

## <a name="prerequisites-to-run-the-sample-app"></a>Необходимые компоненты для запуска примера приложения

Чтобы запустить пример, вам понадобится [Visual Studio](https://www.visualstudio.com/downloads/) и действительная учетная запись Azure Cosmos DB.

Если вы еще не установили Visual Studio, скачайте [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/) с установленной и настроенной рабочей нагрузкой **ASP.NET и веб-разработка**.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] 

<a id="create-account"></a>
## <a name="create-a-database-account"></a>Создание учетной записи базы данных

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

Описанный в этой статье пример приложения совместим с MongoDB.Driver версии 2.6.1.

## <a name="clone-the-sample-app"></a>Клонирования примера приложения

Сначала загрузите пример приложения из GitHub. 

1. Откройте командную строку, создайте папку git-samples, а затем закройте окно командной строки.

    ```bash
    mkdir "C:\git-samples"
    ```

2. Откройте окно терминала git, например git bash, и выполните команду `cd`, чтобы перейти в новую папку для установки примера приложения.

    ```bash
    cd "C:\git-samples"
    ```

3. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере. 

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-getting-started.git
    ```

Если вы не хотите использовать Git, можно [скачать проект как ZIP-файл](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-getting-started/archive/master.zip).

## <a name="review-the-code"></a>Просмотр кода

Это необязательный шаг. Если вы хотите узнать, как создать в коде ресурсы базы данных, изучите приведенные ниже фрагменты кода. Если вас это не интересует, можете сразу переходить к разделу [Обновление строки подключения](#update-your-connection-string). 

Следующие фрагменты кода взяты из файла Dal.cs в каталоге DAL.

* Инициализация клиента.

    ```cs
        MongoClientSettings settings = new MongoClientSettings();
        settings.Server = new MongoServerAddress(host, 10255);
        settings.UseSsl = true;
        settings.SslSettings = new SslSettings();
        settings.SslSettings.EnabledSslProtocols = SslProtocols.Tls12;

        MongoIdentity identity = new MongoInternalIdentity(dbName, userName);
        MongoIdentityEvidence evidence = new PasswordEvidence(password);

        settings.Credential = new MongoCredential("SCRAM-SHA-1", identity, evidence);

        MongoClient client = new MongoClient(settings);
    ```

* Получение базы данных и коллекции.

    ```cs
    private string dbName = "Tasks";
    private string collectionName = "TasksList";

    var database = client.GetDatabase(dbName);
    var todoTaskCollection = database.GetCollection<MyTask>(collectionName);
    ```

* Получение всех документов.

    ```cs
    collection.Find(new BsonDocument()).ToList();
    ```

Создание задачи и ее вставка в коллекцию

   ```csharp
    public void CreateTask(MyTask task)
    {
        var collection = GetTasksCollectionForEdit();
        try
        {
            collection.InsertOne(task);
        }
        catch (MongoCommandException ex)
        {
            string msg = ex.Message;
        }
    }
   ```
   Аналогичным образом можно обновлять и удалять документы с помощью методов [collection.UpdateOne()](https://docs.mongodb.com/stitch/mongodb/actions/collection.updateOne/index.html) и [collection.DeleteOne()](https://docs.mongodb.com/stitch/mongodb/actions/collection.deleteOne/index.html). 

## <a name="update-your-connection-string"></a>Обновление строки подключения

Теперь вернитесь на портал Azure, чтобы получить данные строки подключения. Скопируйте эти данные в приложение.

1. На [портале Azure](https://portal.azure.com/) перейдите к учетной записи базы данных Cosmos и на левой панели навигации щелкните **Строка подключения**, а затем щелкните **Ключи записи-чтения**. На следующем шаге используйте кнопку копирования в правой части экрана, чтобы скопировать имя пользователя, пароль и узел в файл Dal.cs.

2. Откройте файл **Dal.cs** в каталоге **DAL**. 

3. Скопируйте **имя пользователя** c портала (с помощью кнопки копирования) и добавьте его в качестве значения параметра **username** в файле **Dal.cs**. 

4. Затем скопируйте значение **узла** с портала и добавьте его в качестве значения параметра **host** в файле **Dal.cs**. 

5. И наконец, скопируйте **пароль** с портала и добавьте его в качестве значения параметра **password** в файле **Dal.cs**. 

Теперь приложение со всеми сведениями, необходимыми для взаимодействия с Cosmos DB, обновлено. 
    
## <a name="run-the-web-app"></a>Запуск веб-приложения

1. В Visual Studio в **обозревателе решений** щелкните проект правой кнопкой мыши и выберите **Управление пакетами NuGet**. 

2. В окне NuGet в поле **Обзор** введите *MongoDB.Driver*.

3. В результатах поиска найдите библиотеку **MongoDB.Driver** и установите ее. При этом будет установлен пакет MongoDB.Driver, а также все зависимости.

4. Нажмите клавиши CTRL+F5 для запуска приложения. Приложение откроется в браузере. 

5. Щелкните **Создать** в браузере и создайте несколько задач в приложении списка задач.

## <a name="review-slas-in-the-azure-portal"></a>Просмотр соглашений об уровне обслуживания на портале Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать учетную запись и коллекцию Cosmos и запустить консольное приложение. Теперь вы можете импортировать дополнительные данные в базу данных Cosmos. 

> [!div class="nextstepaction"]
> [Перенос данных MongoDB в Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)