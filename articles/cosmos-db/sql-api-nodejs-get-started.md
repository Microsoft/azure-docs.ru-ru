---
title: Руководство по Node.js для API SQL в Azure Cosmos DB
description: Руководство по Node.js, в котором описывается, как подключиться к Azure Cosmos DB и выполнить ее запрос с помощью API SQL
author: deborahc
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 04/20/2020
ms.author: dech
ms.custom: devx-track-js
ms.openlocfilehash: 1115d7bb0d6857aa39f246743df54a6cf3fd3676
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93098668"
---
# <a name="tutorial-build-a-nodejs-console-app-with-the-javascript-sdk-to-manage-azure-cosmos-db-sql-api-data"></a>Руководство по Создание консольного приложения Node.js с помощью пакета SDK JavaScript для управления данными API SQL для Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET](sql-api-get-started.md)
> * [Java](./create-sql-api-java.md)
> * [Async Java](./create-sql-api-java.md)
> * [Node.js](sql-api-nodejs-get-started.md)
> 

Как у разработчика у вас могут быть приложения, использующие данные документов NoSQL. Вы можете использовать учетную запись API SQL в Azure Cosmos DB для хранения этих данных документов и получения доступа к ним. В этом руководстве показано, как создать консольное приложение Node.js, которое создает ресурсы Azure Cosmos DB, и запросить эти ресурсы.

Выполняя данное руководство, вы сделаете следующее:

> [!div class="checklist"]
> * создавать учетную запись Azure Cosmos DB и подключаться к ней;
> * настраивать свое приложение;
> * создавать базу данных;
> * Создайте контейнер.
> * добавлять элементы в контейнер;
> * выполнять операции CRUD с элементами, контейнером и базой данных.

## <a name="prerequisites"></a>Предварительные требования 

Убедитесь, что у вас есть указанные ниже ресурсы.

* Активная учетная запись Azure. Если у вас нет такой учетной записи, вы можете зарегистрироваться для использования [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/). 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Node.js](https://nodejs.org/) 6.0.0 или более поздней версии.

## <a name="create-azure-cosmos-db-account"></a>Создание учетной записи Azure Cosmos DB

Давайте создадим учетную запись Azure Cosmos DB. Если у вас уже есть учетная запись, которую вы собираетесь использовать, можно перейти к шагу [Настройка приложения Node.js](#SetupNode). Если вы используете эмулятор Azure Cosmos DB, выполните действия, описанные в статье об [эмуляторе Azure Cosmos DB](local-emulator.md), чтобы настроить эмулятор и сразу перейти к [настройке приложения Node.js](#SetupNode). 

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="set-up-your-nodejs-application"></a><a id="SetupNode"></a>Настройка приложения Node.js

Прежде чем вы начнете писать код для создания приложения, можете создать платформу для приложения. Выполните следующие шаги, чтобы настроить приложение Node.js, содержащее код платформы:

1. Откройте удобный для вас терминал.
2. Перейдите в папку или каталог, где вы хотите сохранить приложение Node.js.
3. Создайте пустые файлы JavaScript с помощью следующих команд.

   * Windows:
     * `fsutil file createnew app.js 0`
     * `fsutil file createnew config.js 0`
     * `md data`
     * `fsutil file createnew data\databaseContext.js 0`

   * Linux или OS X:
     * `touch app.js`
     * `touch config.js`
     * `mkdir data`
     * `touch data/databaseContext.js`

4. Создание и инициализация файла `package.json`. Используйте следующую команду:
   * ```npm init -y```

5. Установите модуль @azure/cosmos с помощью npm. Используйте следующую команду:
   * ```npm install @azure/cosmos --save```

## <a name="set-your-apps-configurations"></a><a id="Config"></a>Настройка конфигурации приложения

Теперь, когда ваше приложение существует, необходимо убедиться, что оно может взаимодействовать с Azure Cosmos DB. Как показано в следующих шагах, обновляя некоторые параметры конфигурации, вы можете настроить свое приложение для взаимодействия с Azure Cosmos DB.

1. Откройте файл *config.js* в любом текстовом редакторе.

1. Скопируйте и вставьте приведенный ниже фрагмент кода в файл *config.js*, после чего задайте URI конечной точки и первичный ключ Azure Cosmos DB в качестве значений для свойств `endpoint` и `key`. В базе данных контейнеры имеют имена **Tasks** и **Items**. Для этого приложения будет использоваться ключ раздела **/category**.

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/config.js":::

   Сведения о конечной точке и ключе можно найти на панели **Ключи** [портала Azure](https://portal.azure.com).

   :::image type="content" source="media/sql-api-nodejs-get-started/node-js-tutorial-keys.png" alt-text="Снимок экрана получения ключей на портале Azure":::

В пакете SDK для JavaScript используются универсальные термины — *контейнер* и *элемент*. Контейнер может представлять собой коллекцию, граф или таблицу. Элемент может представлять собой документ, ребро, вершину или запись и является содержимым внутри контейнера. В предыдущем фрагменте кода `module.exports = config;` используется для экспорта объекта конфигурации, чтобы на него можно было ссылаться в файле *app.js*.

## <a name="create-a-database-and-a-container"></a>Создание базы данных и контейнера

1. Откройте файл *databaseContext.js* в любом текстовом редакторе.

1. Скопируйте и вставьте в файл *databaseContext.js* приведенный ниже код. Этот код определяет функцию, которая создает базу данных и контейнер (Tasks и Items), если они еще не существуют в вашей учетной записи Azure Cosmos:

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/data/databaseContext.js" id="createDatabaseAndContainer":::

   База данных представляет собой логический контейнер элементов, разделенных между контейнерами. Чтобы создать базу данных, используйте метод `createIfNotExists` или функцию create из класса **Databases**. Контейнер состоит из элементов, которые в случае API SQL являются документами JSON. Контейнер создается с помощью метода `createIfNotExists` или функции create из класса **Containers**. После создания контейнера вы сможете сохранять и запрашивать данные.

   > [!WARNING]
   > Создание контейнера связано с ценовыми требованиями. Посетите нашу [страницу с ценами](https://azure.microsoft.com/pricing/details/cosmos-db/), чтобы знать, чего следует ожидать.

## <a name="import-the-configuration"></a>Импорт конфигурации

1. Откройте файл *app.js* в любом текстовом редакторе.

1. Скопируйте и вставьте приведенный ниже код, чтобы импортировать модуль `@azure/cosmos`, конфигурацию и databaseContext, определенные на предыдущих шагах. 

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="ImportConfiguration":::

## <a name="connect-to-the-azure-cosmos-account"></a>Подключение к учетной записи Azure Cosmos

Скопируйте и вставьте в файл *app.js* приведенный ниже код, чтобы на основании ранее сохраненных значений конечной точки и ключа создать объект CosmosClient.

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="CreateClientObjectDatabaseContainer":::

> [!Note]
> При подключении к **эмулятору Cosmos DB** отключите проверку TLS для процесса node.
>   ```javascript
>   process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";
>   const client = new CosmosClient({ endpoint, key });
>   ```

Теперь, когда у нас есть код для инициализации клиента Azure Cosmos DB, давайте рассмотрим принципы работы с ресурсами Azure Cosmos DB.

## <a name="query-items"></a><a id="QueryItem"></a>Элементы запроса

Azure Cosmos DB поддерживает полнофункциональные запросы по элементам JSON, хранящимся в каждом контейнере. В следующем примере кода показан запрос, который вы можете выполнить по элементам в контейнере. Для обращения к элементам можно использовать функцию query из класса `Items`. Добавьте следующий код в файл *app.js*, чтобы получить элементы из учетной записи Azure Cosmos:

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="QueryItems":::

## <a name="create-an-item"></a><a id="CreateItem"></a>Создание элемента

Элемент можно создать с помощью функции create из класса `Items`. При использовании API SQL элементы проецируются как документы, которые являются содержимым JSON, определенным пользователем (произвольным). В этом руководстве описано, как создать элемент в базе данных Tasks.

1. В файле app.js укажите определение элемента:

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="DefineNewItem":::

1. Добавьте следующий код, чтобы создать ранее определенный элемент:

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="CreateItem":::

## <a name="update-an-item"></a><a id="ReplaceItem"></a>Обновление элемента

Azure Cosmos DB поддерживает замену содержимого элементов. Скопируйте следующий код и вставьте его в файл *app.js*. Этот код позволяет получить элемент из контейнера и обновить поле *isComplete*, присвоив ему значение true.

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="UpdateItem":::

## <a name="delete-an-item"></a><a id="DeleteItem"></a>Удаление элемента

Azure Cosmos DB поддерживает удаление элементов JSON. В следующем примере кода показано, как получить и удалить элемент по его идентификатору. Скопируйте следующий код и вставьте его в файл *app.js*.

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="DeleteItem":::

## <a name="run-your-nodejs-application"></a><a id="Run"></a>Запуск приложения Node.js

В целом, код должен выглядеть следующим образом:

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js":::

В окне терминала перейдите к файлу ```app.js``` и выполните эту команду:

```bash 
node app.js
```

Должны отобразиться выходные данные вашего приложения. Выходные данные должны соответствовать примеру текста ниже.

```
Created database:
Tasks

Created container:
Items

Querying container: Items
1 - Pick up apples and strawberries.

Created new item: 3 - Complete Cosmos DB Node.js Quickstart ⚡

Updated item: 3 - Complete Cosmos DB Node.js Quickstart ⚡
Updated isComplete to true

Deleted item with id: 3
```

## <a name="get-the-complete-nodejs-tutorial-solution"></a><a id="GetSolution"></a>Получение полного решения для руководства по Node.js

Если вы не располагаете временем, чтобы выполнить шаги из этого руководства, или просто хотите скачать код, это можно сделать на портале [GitHub](https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-getting-started ).

Чтобы запустить решение по началу работы, которое содержит все коды из этой статьи, вам понадобится следующее:

* [Учетная запись Azure Cosmos DB][create-account].
* Решение для [начала работы](https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-getting-started), доступное на GitHub.

Установите зависимости проекта с помощью npm. Используйте следующую команду:

* ```npm install``` 

Затем в файле ```config.js``` обновите значения config.endpoint и config.key, как описано в разделе [Шаг 3. Настройка конфигурации приложения](#Config).  

Затем в окне терминала перейдите к файлу ```app.js``` и выполните эту команду:  

```bash  
node app.js 
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Можно удалить группу ресурсов, учетную запись Azure Cosmos DB и все связанные ресурсы, когда они больше не нужны. Для этого выберите группу ресурсов, которая использовалась для учетной записи Azure Cosmos DB, выберите **Удалить**, а затем подтвердите имя удаляемой группы ресурсов.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Мониторинг Azure Cosmos DB](./monitor-cosmos-db.md)

[create-account]: create-sql-api-dotnet.md#create-account