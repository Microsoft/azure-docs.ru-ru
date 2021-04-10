---
title: Руководство. Создание веб-приложения Node.js с помощью пакета SDK JavaScript для Azure Cosmos DB для управления данными API SQL
description: В этом руководстве по Node.js описывается использование Microsoft Azure Cosmos DB для хранения данных и доступа к ним из веб-приложения Node.js Express, размещенного в компоненте "Веб-приложения" Службы приложений Microsoft Azure.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 11/05/2019
ms.author: sngun
ms.custom: devx-track-js
ms.openlocfilehash: 49cf54bda985f7d97b2db6a3ada7859aee829cff
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97359546"
---
# <a name="tutorial-build-a-nodejs-web-app-using-the-javascript-sdk-to-manage-a-sql-api-account-in-azure-cosmos-db"></a>Руководство по Создание веб-приложения Node.js с помощью пакета SDK для JavaScript для управления учетной записью API SQL в Azure Cosmos DB 
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.js](sql-api-nodejs-application.md)
> * [Python](./create-sql-api-python.md)
> * [Xamarin](mobile-apps-with-xamarin.md)
> 

Как у разработчика у вас могут быть приложения, использующие данные документов NoSQL. Вы можете использовать учетную запись API SQL в Azure Cosmos DB для хранения этих данных документов и получения доступа к ним. В этом руководстве по Node.js показано, как с помощью приложения Node.js Express, размещенного в компоненте "Веб-приложения" Службы приложений Microsoft Azure, хранить данные и получать доступ к ним из учетной записи API SQL в Azure Cosmos DB. Здесь вы создадите веб-приложение (приложение со списком задач), которое позволит вам создавать, извлекать и выполнять задачи. Задачи будут храниться в виде документов JSON в Azure Cosmos DB. 

В этом руководстве показано, как создать учетную запись API SQL в Azure Cosmos DB с помощью портала Azure. Затем вы создадите и запустите веб-приложение, созданное на основе пакета SDK для Node.js, для создания базы данных и контейнера, а также для добавления элементов в контейнер. В этом руководстве используется пакет SDK для JavaScript версии 3.0.

В рамках этого руководства рассматриваются следующие задачи:

> [!div class="checklist"]
> * создание учетной записи Azure Cosmos DB;
> * создание приложения Node.js;
> * подключение приложения к Azure Cosmos DB;
> * запуск и развертывание приложения в Azure.

## <a name="prerequisites"></a><a name="_Toc395783176"></a>Предварительные требования

Перед выполнением инструкций, приведенных в этой статье, обеспечьте наличие следующих ресурсов:

* Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу. 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Node.js][Node.js] версии 6.10 или последующей.
* [Генератор Express](https://www.expressjs.com/starter/generator.html) (его можно установить через `npm install express-generator -g`).
* Установите [Git][Git] на локальной рабочей станции.

## <a name="create-an-azure-cosmos-db-account"></a><a name="_Toc395637761"></a>Создание учетной записи Azure Cosmos DB
Давайте сначала создадим учетную запись Azure Cosmos DB. Если у вас уже есть учетная запись или вы используете эмулятор Azure Cosmos DB для работы с этим руководством, можно перейти к разделу [Шаг 2. Создание нового приложения Node.js](#_Toc395783178).

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

[!INCLUDE [cosmos-db-keys](../../includes/cosmos-db-keys.md)]

## <a name="create-a-new-nodejs-application"></a><a name="_Toc395783178"></a>Создание нового приложения Node.js
Теперь создадим базовый проект Node.js Hello World с помощью платформы Express.

1. Откройте предпочитаемый терминал, например командную строку Node.js.

1. Перейдите в каталог, в котором будет сохранено новое приложение.

1. С помощью генератора Express создайте новое приложение с именем **todo**.

   ```bash
   express todo
   ```

1. Откройте каталог **todo** и установите зависимости.

   ```bash
   cd todo
   npm install
   ```

1. Запустите новое приложение.

   ```bash
   npm start
   ```

1. Новое приложение можно просмотреть, перейдя в браузере по адресу `http://localhost:3000`.
   
   :::image type="content" source="./media/sql-api-nodejs-application/cosmos-db-node-js-express.png" alt-text="Изучение Node.js — снимок экрана приложения Hello World в окне браузера":::

   Остановите приложение, нажав клавиши CTRL+C в окне терминала, и выберите **Y**, чтобы завершить пакетное задание.

## <a name="install-the-required-modules"></a><a name="_Toc395783179"></a>Установка необходимых модулей

Файл **package.json** является одним из файлов, создаваемых в корневой папке проекта. Этот файл содержит список дополнительных модулей, необходимых для приложения Node.js. При развертывании этого приложения в Azure этот файл будет использоваться для определения модулей, которые следует установить в Azure для поддержки вашего приложения. Для работы с этим руководством установите еще два пакета.

1. Установите модуль **\@azure/cosmos** с помощью npm. 

   ```bash
   npm install @azure/cosmos
   ```

## <a name="connect-the-nodejs-application-to-azure-cosmos-db"></a><a name="_Toc395783180"></a>Подключение приложения Node.js к Azure Cosmos DB
Теперь, когда вы завершили первоначальную настройку и установку, нужно написать код, который требуется приложению со списком задач для обмена данными с Azure Cosmos DB.

### <a name="create-the-model"></a>Создание модели
1. В корне каталога проекта создайте каталог с именем **models**.  

2. В каталоге **models** создайте новый файл с именем **taskDao.js**. Этот файл содержит код, необходимый для создания базы данных и контейнера. Он также определяет методы для чтения, обновления, создания и поиска задач в Azure Cosmos DB. 

3. Скопируйте следующий код в файл **taskDao.js**:

   ```javascript
    // @ts-check
    const CosmosClient = require('@azure/cosmos').CosmosClient
    const debug = require('debug')('todo:taskDao')

    // For simplicity we'll set a constant partition key
    const partitionKey = undefined
    class TaskDao {
      /**
       * Manages reading, adding, and updating Tasks in Cosmos DB
       * @param {CosmosClient} cosmosClient
       * @param {string} databaseId
       * @param {string} containerId
       */
      constructor(cosmosClient, databaseId, containerId) {
        this.client = cosmosClient
        this.databaseId = databaseId
        this.collectionId = containerId

        this.database = null
        this.container = null
      }

      async init() {
        debug('Setting up the database...')
        const dbResponse = await this.client.databases.createIfNotExists({
          id: this.databaseId
        })
        this.database = dbResponse.database
        debug('Setting up the database...done!')
        debug('Setting up the container...')
        const coResponse = await this.database.containers.createIfNotExists({
          id: this.collectionId
        })
        this.container = coResponse.container
        debug('Setting up the container...done!')
      }

      async find(querySpec) {
        debug('Querying for items from the database')
        if (!this.container) {
          throw new Error('Collection is not initialized.')
        }
        const { resources } = await this.container.items.query(querySpec).fetchAll()
        return resources
      }

      async addItem(item) {
        debug('Adding an item to the database')
        item.date = Date.now()
        item.completed = false
        const { resource: doc } = await this.container.items.create(item)
        return doc
      }

      async updateItem(itemId) {
        debug('Update an item in the database')
        const doc = await this.getItem(itemId)
        doc.completed = true

        const { resource: replaced } = await this.container
          .item(itemId, partitionKey)
          .replace(doc)
        return replaced
      }

      async getItem(itemId) {
        debug('Getting an item from the database')
        const { resource } = await this.container.item(itemId, partitionKey).read()
        return resource
      }
    }

    module.exports = TaskDao
   ```
4. Сохраните и закройте файл **taskDao.js** .  

### <a name="create-the-controller"></a>Создание контроллера

1. В каталоге проекта **routes** создайте новый файл с именем **tasklist.js**.  

2. Добавьте в **tasklist.js** следующий код. Этот код загружает модули CosmosClient и async, используемые файлом **tasklist.js**. Он также определяет класс **TaskList**, который передается как экземпляр определенного ранее объекта **TaskDao**:
   
   ```javascript
    const TaskDao = require("../models/TaskDao");
    
    class TaskList {
      /**
       * Handles the various APIs for displaying and managing tasks
       * @param {TaskDao} taskDao
       */
      constructor(taskDao) {
        this.taskDao = taskDao;
      }
      async showTasks(req, res) {
        const querySpec = {
          query: "SELECT * FROM root r WHERE r.completed=@completed",
          parameters: [
            {
              name: "@completed",
              value: false
            }
          ]
        };

        const items = await this.taskDao.find(querySpec);
        res.render("index", {
          title: "My ToDo List ",
          tasks: items
        });
      }

      async addTask(req, res) {
        const item = req.body;

        await this.taskDao.addItem(item);
        res.redirect("/");
      }

      async completeTask(req, res) {
        const completedTasks = Object.keys(req.body);
        const tasks = [];

        completedTasks.forEach(task => {
          tasks.push(this.taskDao.updateItem(task));
        });

        await Promise.all(tasks);

        res.redirect("/");
      }
    }

    module.exports = TaskList;
   ```

3. Сохраните и закройте файл **tasklist.js** .

### <a name="add-configjs"></a>Добавление config.js

1. В корне каталога проекта создайте файл с именем **config.js**. 

2. Добавьте следующий код в файл **config.js**. Он определяет значения и параметры конфигурации, необходимые нашему приложению.
   
   ```javascript
   const config = {};

   config.host = process.env.HOST || "[the endpoint URI of your Azure Cosmos DB account]";
   config.authKey =
     process.env.AUTH_KEY || "[the PRIMARY KEY value of your Azure Cosmos DB account";
   config.databaseId = "ToDoList";
   config.containerId = "Items";

   if (config.host.includes("https://localhost:")) {
     console.log("Local environment detected");
     console.log("WARNING: Disabled checking of self-signed certs. Do not have this code in production.");
     process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";
     console.log(`Go to http://localhost:${process.env.PORT || '3000'} to try the sample.`);
   }

   module.exports = config;
   ```

3. В файле **config.js** замените значения HOST и AUTH_KEY значениями, найденными на странице "Ключи" учетной записи Azure Cosmos DB на [портале Azure](https://portal.azure.com). 

4. Сохраните и закройте файл **config.js** .

### <a name="modify-appjs"></a>Изменение app.js

1. В каталоге проекта откройте файл **app.js** . Этот файл был создан ранее, при создании веб-приложения Express.  

2. Добавьте следующий код в файл **app.js**. Этот код определяет используемый файл конфигурации и загружает значения в переменные, которые будут использоваться в следующих разделах. 
   
   ```javascript
    const CosmosClient = require('@azure/cosmos').CosmosClient
    const config = require('./config')
    const TaskList = require('./routes/tasklist')
    const TaskDao = require('./models/taskDao')

    const express = require('express')
    const path = require('path')
    const logger = require('morgan')
    const cookieParser = require('cookie-parser')
    const bodyParser = require('body-parser')

    const app = express()

    // view engine setup
    app.set('views', path.join(__dirname, 'views'))
    app.set('view engine', 'jade')

    // uncomment after placing your favicon in /public
    //app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
    app.use(logger('dev'))
    app.use(bodyParser.json())
    app.use(bodyParser.urlencoded({ extended: false }))
    app.use(cookieParser())
    app.use(express.static(path.join(__dirname, 'public')))

    //Todo App:
    const cosmosClient = new CosmosClient({
      endpoint: config.host,
      key: config.authKey
    })
    const taskDao = new TaskDao(cosmosClient, config.databaseId, config.containerId)
    const taskList = new TaskList(taskDao)
    taskDao
      .init(err => {
        console.error(err)
      })
      .catch(err => {
        console.error(err)
        console.error(
          'Shutting down because there was an error settinig up the database.'
        )
        process.exit(1)
      })

    app.get('/', (req, res, next) => taskList.showTasks(req, res).catch(next))
    app.post('/addtask', (req, res, next) => taskList.addTask(req, res).catch(next))
    app.post('/completetask', (req, res, next) =>
      taskList.completeTask(req, res).catch(next)
    )
    app.set('view engine', 'jade')

    // catch 404 and forward to error handler
    app.use(function(req, res, next) {
      const err = new Error('Not Found')
      err.status = 404
      next(err)
    })

    // error handler
    app.use(function(err, req, res, next) {
      // set locals, only providing error in development
      res.locals.message = err.message
      res.locals.error = req.app.get('env') === 'development' ? err : {}

      // render the error page
      res.status(err.status || 500)
      res.render('error')
    })

    module.exports = app
   ```

3. Наконец, сохраните и закройте файл **app.js**.

## <a name="build-a-user-interface"></a><a name="_Toc395783181"></a>Создание пользовательского интерфейса

Теперь давайте создадим пользовательский интерфейс, чтобы пользователь мог взаимодействовать с приложением. Созданное нами в предыдущих разделах приложение Express использует в качестве обработчика представлений **Jade**.

1. Файл **layout.jade** в каталоге **views** используется как глобальный шаблон для других файлов **.jade**. На этом шаге он будет изменен для использования Twitter Bootstrap — набора средств для разработки веб-сайта.  

2. Откройте файл **layout.jade**, расположенный в папке **views**, и замените его содержимое следующим кодом:

   ```html
   doctype html
   html
     head
       title= title
       link(rel='stylesheet', href='//ajax.aspnetcdn.com/ajax/bootstrap/3.3.2/css/bootstrap.min.css')
       link(rel='stylesheet', href='/stylesheets/style.css')
     body
       nav.navbar.navbar-inverse.navbar-fixed-top
         div.navbar-header
           a.navbar-brand(href='#') My Tasks
       block content
       script(src='//ajax.aspnetcdn.com/ajax/jQuery/jquery-1.11.2.min.js')
       script(src='//ajax.aspnetcdn.com/ajax/bootstrap/3.3.2/bootstrap.min.js')
   ```

    Этот код позволяет эффективно сообщить подсистеме **Jade** о том, что необходимо выполнить прорисовку HTML-кода для нашего приложения, и создает **block** с именем **content**, где можно указать макет для страниц содержимого. Сохраните и закройте файл **layout.jade**.

3. Теперь откройте файл **index.jade** (представление, которое будет использоваться нашим приложением) и замените содержимое файла следующим кодом:

   ```html
   extends layout
   block content
        h1 #{title}
        br
        
        form(action="/completetask", method="post")
         table.table.table-striped.table-bordered
            tr
              td Name
              td Category
              td Date
              td Complete
            if (typeof tasks === "undefined")
              tr
                td
            else
              each task in tasks
                tr
                  td #{task.name}
                  td #{task.category}
                  - var date  = new Date(task.date);
                  - var day   = date.getDate();
                  - var month = date.getMonth() + 1;
                  - var year  = date.getFullYear();
                  td #{month + "/" + day + "/" + year}
                  td
                   if(task.completed) 
                    input(type="checkbox", name="#{task.id}", value="#{!task.completed}", checked=task.completed)
                   else
                    input(type="checkbox", name="#{task.id}", value="#{!task.completed}", checked=task.completed)
          button.btn.btn-primary(type="submit") Update tasks
        hr
        form.well(action="/addtask", method="post")
          label Item Name:
          input(name="name", type="textbox")
          label Item Category:
          input(name="category", type="textbox")
          br
          button.btn(type="submit") Add item
   ```

Этот код дополняет макет и предоставляет содержимое для заполнителя **content**, который мы ранее видели в файле **layout.jade**. В макете мы создали две формы HTML.

Первая форма содержит таблицу для наших данных и кнопку, позволяющую обновлять элементы путем отправки POST-запроса к методу **/completeTask** контроллера.
    
Вторая форма содержит два поля ввода и кнопку, позволяющую создать новый элемент путем POST-запроса к методу **/addtask** контроллера. Это все, что нужно для работы приложения.

## <a name="run-your-application-locally"></a><a name="_Toc395783181"></a>Локальный запуск приложения

Теперь, когда вы создали приложение, можете запустить его локально, используя следующие действия:  

1. Чтобы протестировать приложение на локальном компьютере, выполните `npm start` в терминале для запуска приложения, а затем обновите в браузере страницу `http://localhost:3000`. Теперь страница будет выглядеть, как показано на следующем снимке экрана:
   
    :::image type="content" source="./media/sql-api-nodejs-application/cosmos-db-node-js-localhost.png" alt-text="Снимок экрана приложения My ToDo List в окне браузера":::

    > [!TIP]
    > Если появляется сообщение об ошибке отступа в файле layout.jade или index.jade, выровняйте две первые строки в этих двух файлах по левому краю без пробелов. Если перед первыми двумя строками есть пробелы, удалите их, сохраните оба файла и затем обновите окно браузера. 

2. Используйте поля "Элемент", "Имя элемента" и "Категория", чтобы указать новую задачу, а затем выберите **Добавить элемент**. Будет создан документ в Azure Cosmos DB с заданными свойствами. 

3. Страница должна обновиться, чтобы отобразить только что созданный элемент в списке ToDo.
   
    :::image type="content" source="./media/sql-api-nodejs-application/cosmos-db-node-js-added-task.png" alt-text="Снимок экрана приложения с новым элементом в списке дел":::

4. Чтобы завершить задачу, установите флажок в столбце "Завершено" и выберите **Обновить задачи**. После этого созданный документ будет обновлен и удален из представления.

5. Чтобы остановить приложение, нажмите клавиши CTRL+C в окне терминала и выберите **Y** для завершения пакетного задания.

## <a name="deploy-your-application-to-web-apps"></a><a name="_Toc395783182"></a>Развертывание приложения в компоненте "Веб-приложения"

Если ваше приложение успешно работает локально, можно развернуть его в Azure, выполнив следующие действия:

1. Если это еще не сделано, включите репозиторий Git для приложения компонента "Веб-приложения".

2. Добавьте приложение компонента "Веб-приложения" в качестве удаленного репозитория Git.
   
   ```bash
   git remote add azure https://username@your-azure-website.scm.azurewebsites.net:443/your-azure-website.git
   ```

3. Разверните приложение, отправив его в удаленный репозиторий.
   
   ```bash
   git push azure main
   ```

4. Через несколько секунд веб-приложение будет опубликовано и запущено в браузере.

## <a name="clean-up-resources"></a>Очистка ресурсов

Можно удалить группу ресурсов, учетную запись Azure Cosmos DB и все связанные ресурсы, когда они больше не нужны. Для этого выберите группу ресурсов, которая использовалась для учетной записи Azure Cosmos DB, выберите **Удалить**, а затем подтвердите имя удаляемой группы ресурсов.

## <a name="next-steps"></a><a name="_Toc395637775"></a>Следующие шаги

> [!div class="nextstepaction"]
> [Создание мобильных приложений с помощью Xamarin и Azure Cosmos DB](mobile-apps-with-xamarin.md)


[Node.js]: https://nodejs.org/
[Git]: https://git-scm.com/
[GitHub]: https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-todo-app