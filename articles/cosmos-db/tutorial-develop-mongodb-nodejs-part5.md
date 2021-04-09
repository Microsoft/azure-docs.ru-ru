---
title: Подключение приложения Angular к API Azure Cosmos DB для MongoDB с помощью Mongoose
description: В этом руководстве описывается создание приложения Node.js (с помощью Angular и Express), которое управляет данными, хранящимися в Cosmos DB. В этой части для подключения к Azure Cosmos DB используется Mongoose.
author: johnpapa
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 12/26/2018
ms.author: jopapa
ms.custom: seodec18, devx-track-js
ms.reviewer: sngun
ms.openlocfilehash: a3097fa539f460ef5e8ffe73598fa5d55516717e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93097810"
---
# <a name="create-an-angular-app-with-azure-cosmos-dbs-api-for-mongodb---use-mongoose-to-connect-to-cosmos-db"></a>Создание приложения Angular с использованием API Azure Cosmos DB для MongoDB. Подключение к Cosmos DB с помощью Mongoose
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Из этого руководства в нескольких частях вы узнаете, как создать приложение Node.js с использованием Express и Angular и подключить его к учетной записи [Cosmos, настроенной с API Cosmos DB для MongoDB](mongodb-introduction.md). В этой статье описана часть 5 руководства, которая основана на [части 4](tutorial-develop-mongodb-nodejs-part4.md).

В этой части руководства вы выполните следующее:

> [!div class="checklist"]
> * подключитесь к Cosmos DB с помощью Mongoose;
> * получите строку подключения к Cosmos DB;
> * создадите модель Hero;
> * создадите службу Hero для получения данных Hero.
> * Запустите приложение на локальном компьютере.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Перед началом работы с этим руководством выполните шаги в [части 4](tutorial-develop-mongodb-nodejs-part4.md).

* Для этого руководства требуется запустить Azure CLI локально. Требуется Azure CLI версии 2.0 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо установить или обновить Azure CLI, ознакомьтесь со статьей об [установке Azure CLI 2.0](/cli/azure/install-azure-cli).

* В этом руководстве изложены пошаговые инструкции по созданию приложения. Готовое приложение можно скачать из [репозитория angular-cosmosdb](https://github.com/Azure-Samples/angular-cosmosdb) на GitHub.

## <a name="use-mongoose-to-connect"></a>Использование Mongoose для подключения

Mongoose является библиотекой моделирования данных объектов (ODM) для MongoDB и Node.js. Вы можете использовать Mongoose для подключения к учетной записи Azure Cosmos DB. Следуйте инструкциям ниже, чтобы установить Mongoose и подключиться к Azure Cosmos DB:

1. Установите модуль npm mongoose, который является API, используемым для обмена данными с MongoDB.

    ```bash
    npm i mongoose --save
    ```

1. В папке **server** создайте файл с именем **mongo.js**. Затем добавьте в этот файл сведения о подключении учетной записи Azure Cosmos DB.

1. Скопируйте следующий код в файл **mongo.js**. Этот код предоставляет следующие функциональные возможности:

   * Требует установки Mongoose.
   * Переопределяет объект Promise Mongo, чтобы использовался базовый объект Promise, который встроен в ES6 или ES2015 и более поздней версии.
   * Вызывает ENV-файл, который позволяет задавать определенные настройки в зависимости от среды — промежуточной, производственной или среды разработки. Вы создадите этот файл при работе со следующим разделом.
   * Содержит строку подключения MongoDB, которая задается в ENV-файле.
   * Создает функцию подключения, которая вызывает Mongoose.

     ```javascript
     const mongoose = require('mongoose');
     /**
     * Set to Node.js native promises
     * Per https://mongoosejs.com/docs/promises.html
     */
     mongoose.Promise = global.Promise;

     const env = require('./env/environment');

     // eslint-disable-next-line max-len
     const mongoUri = `mongodb://${env.accountName}:${env.key}@${env.accountName}.documents.azure.com:${env.port}/${env.databaseName}?ssl=true`;

     function connect() {
     mongoose.set('debug', true);
     return mongoose.connect(mongoUri, { useMongoClient: true });
     }

     module.exports = {
     connect,
     mongoose
     };
     ```
    
1. В области Explorer в разделе **server**, создайте папку с именем **environment**. В папке **environment** создайте файл с именем **environment.js**.

1. Из файла mongo.js нужно включить значения для параметров `dbName`, `key` и `cosmosPort`. Скопируйте следующий код в файл **environment.js**:

    ```javascript
    // TODO: replace if yours are different
    module.exports = {
      accountName: 'your-cosmosdb-account-name-goes-here',
      databaseName: 'admin', 
      key: 'your-key-goes-here',
      port: 10255
    };
    ```

## <a name="get-the-connection-string"></a>Получение строки подключения

Для подключения приложения к Azure Cosmos DB вам потребуется обновить параметры конфигурации этого приложения. Чтобы обновить параметры, выполните следующие шаги: 

1. На портале Azure получите номер порта, имя учетной записи Azure Cosmos DB и значение ее первичного ключа.

1. В файле **environment.js** измените значение `port` на 10255. 

    ```javascript
    const port = 10255;
    ```

1. В файле **environment.js** измените значение `accountName` на имя учетной записи Azure Cosmos DB, которая была создана на [шаге 4](tutorial-develop-mongodb-nodejs-part4.md) руководства. 

1. Получите первичный ключ для учетной записи Azure Cosmos DB, использовав следующую команду CLI в окне терминала: 

    ```azurecli-interactive
    az cosmosdb list-keys --name <cosmosdb-name> -g myResourceGroup
    ```    
    
    \<cosmosdb-name> — имя учетной записи Azure Cosmos DB, созданной в [части 4](tutorial-develop-mongodb-nodejs-part4.md) этого руководства.

1. Скопируйте первичный ключ в файл **environment.js** как значение `key`.

Теперь ваше приложение имеет все необходимые сведения для подключения к Azure Cosmos DB. 

## <a name="create-a-hero-model"></a>Создание модели Hero

Далее необходимо определить схему данных для хранения в Azure Cosmos DB, определив файл модели. Выполните следующие действия, чтобы создать _модель Hero_, определяющую схему данных:

1. В области Explorer в папке **server** создайте файл с именем **hero.model.js**.

1. Скопируйте следующий код в файл **hero.model.js**. Этот код предоставляет следующие функциональные возможности:

   * Требует установки Mongoose.
   * Создает схему с идентификатором, именем и фразой.
   * Создает модель с помощью схемы.
   * Экспортирует модель. 
   * Коллекция должна называться **Heroes** (а не **Heros**, как бы она называлась по умолчанию в соответствии с правилами именования Mongoose для множественного числа).

   ```javascript
   const mongoose = require('mongoose');

   const Schema = mongoose.Schema;

   const heroSchema = new Schema(
     {
       id: { type: Number, required: true, unique: true },
       name: String,
       saying: String
     },
     {
       collection: 'Heroes'
     }
   );

   const Hero = mongoose.model('Hero', heroSchema);

   module.exports = Hero;
   ```

## <a name="create-a-hero-service"></a>Создание службы Hero

После создания модели Hero нужно определить службу для чтения данных, а также выполнения операций перечисления, создания, удаления и обновления. Выполните следующие действия для создания _службы Hero_, которая запрашивает данные из Azure Cosmos DB:

1. В области Explorer в папке **server** создайте файл с именем **hero.service.js**.

1. Скопируйте приведенный ниже код в файл **hero.service.js**. Этот код предоставляет следующие функциональные возможности:

   * Получает созданную модель.
   * Устанавливает подключение к базе данных.
   * Создает переменную `docquery`, которая использует метод `hero.find`, чтобы определить запрос, возвращающий все элементы hero.
   * Выполняет запрос с помощью функции `docquery.exec` с разрешением на получение списка всех элементов hero, для состояния ответа которых установлено значение 200. 
   * Если для состояния установлено значение 500, возвращается сообщение об ошибке.
   * Так как мы используем модули, извлекаются элементы hero. 

   ```javascript
   const Hero = require('./hero.model');

   require('./mongo').connect();

   function getHeroes() {
     const docquery = Hero.find({});
     docquery
       .exec()
       .then(heroes => {
         res.status(200).json(heroes);
       })
       .catch(error => {
         res.status(500).send(error);
         return;
       });
   }

   module.exports = {
     getHeroes
   };
   ```

## <a name="configure-routes"></a>Настройка маршрутов

Теперь необходимо настроить маршруты для обработки URL-адресов для запросов на получение, создание, чтение и удаление. Методы маршрутизации указывают функции обратного вызова, также называемые _функциями обработчика_. Эти функции вызываются, когда приложение получает запрос к указанной конечной точке и метод HTTP. Используйте следующие шаги, чтобы добавить службу Hero и определить свои маршруты:

1. В Visual Studio Code в файле **routes.js** закомментируйте функцию `res.send`, которая отправляет пример данных hero. Вместо нее добавьте строку для вызова функции `heroService.getHeroes`.

    ```javascript
    router.get('/heroes', (req, res) => {
      heroService.getHeroes(req, res);
    //  res.send(200, [
    //      {"id": 10, "name": "Starlord", "saying": "oh yeah"}
    //  ])
    });
    ```

1. В файле **routes.js** добавьте для службы Hero команду `require`:

    ```javascript
    const heroService = require('./hero.service'); 
    ```

1. В файле **hero.service.js** обновите функцию `getHeroes`, чтобы она принимала параметры `req` и `res`, как показано ниже:

    ```javascript
    function getHeroes(req, res) {
    ```

Давайте выделим пару минут на проверку и просмотрим предыдущий код. Сначала мы переходим к файлу index.js, который задает сервер узла. Обратите внимание, что он настраивает и определяет ваши маршруты. Затем файл routes.js обращается к службе Hero и дает ей команду получить функции, например **getHeroes**, а также передать запрос и ответ. Файл hero.service.js получает модель и подключается к Mongo. При вызове он выполняет **getHeroes** и возвращает обратно ответ 200. 

## <a name="run-the-app"></a>Запустите приложение

Затем выполните приложение, следуя приведенным ниже шагам:

1. Сохраните все изменения в Visual Studio Code. В левой части экрана нажмите кнопку **Отладка** (:::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part5/debug-button.png":::), а затем нажмите кнопку **Начать отладку** (:::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part5/start-debugging-button.png":::).

1. Теперь переключитесь на браузер. Откройте **средства разработчика** и перейдите на **вкладку "Сеть"** . Перейдите по адресу `http://localhost:3000`, где расположено ваше приложение.

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part5/azure-cosmos-db-heroes-app.png" alt-text="Новая учетная запись Azure Cosmos DB на портале Azure":::

В приложении пока нет элементов hero. В следующей части этого руководства мы добавим функции put, push и delete. Затем мы сможем добавлять, обновлять и удалять элементы hero из пользовательского интерфейса, используя подключения Mongoose к базе данных Azure Cosmos. 

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить группу ресурсов, учетную запись Azure Cosmos DB и все связанные ресурсы, когда они больше не нужны. Чтобы удалить группу ресурсов, выполните шаги, описанные ниже:

 1. Перейдите к группе ресурсов, в которой вы создали учетную запись Azure Cosmos DB.
 1. Выберите **Удалить группу ресурсов**.
 1. Подтвердите имя удаляемой группы ресурсов и выберите **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

Приступите к части 6 руководства и добавьте в приложение функции Post, Put и Delete:

> [!div class="nextstepaction"]
> [Часть 6. Добавление в приложение функций Post, Put и Delete](tutorial-develop-mongodb-nodejs-part6.md).