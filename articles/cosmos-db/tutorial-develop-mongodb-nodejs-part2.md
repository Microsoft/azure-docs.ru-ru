---
title: Создание приложения Express Node.js с помощью API Azure Cosmos DB для MongoDB (часть 2)
description: Часть 2 серии руководств по созданию приложения MongoDB в Azure Cosmos DB с помощью Angular и Node и тех же API, которые используются для MongoDB.
author: johnpapa
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 12/26/2018
ms.author: jopapa
ms.custom: seodec18
ms.reviewer: sngun
ms.openlocfilehash: e3af713d6e120e66a0cde5116f4556bb8caedf55
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93097861"
---
# <a name="create-an-angular-app-with-azure-cosmos-dbs-api-for-mongodb---create-a-nodejs-express-app"></a>Создание приложения Angular с помощью API Azure Cosmos DB для MongoDB. Создание приложения Express Node.js
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Из этого руководства в нескольких частях вы узнаете, как создать новое приложение, написанное на Node.js с использованием Express и Angular, а затем подключить его к [учетной записи Cosmos, настроенной с помощью API Cosmos DB для MongoDB](mongodb-introduction.md).

Часть 2 руководства основана на [введении](tutorial-develop-mongodb-nodejs.md). Здесь рассматриваются следующие задачи:

> [!div class="checklist"]
> * установка Angular CLI и TypeScript;
> * создание нового проекта с помощью Angular;
> * создание приложения с помощью платформы Express;
> * тестирование приложения в Postman.

## <a name="video-walkthrough"></a>Видеоруководство

> [!VIDEO https://www.youtube.com/embed/lIwJIYcGSUg]

## <a name="prerequisites"></a>Предварительные требования

Прежде чем начинать работу с этой частью руководства, обязательно просмотрите [вводный видеоролик](tutorial-develop-mongodb-nodejs.md).

Для работы с этим руководством также требуется следующее: 
* [Node.js](https://nodejs.org/) версии 8.4.0 или более поздней;
* [Postman](https://www.getpostman.com/)
* [Visual Studio Code](https://code.visualstudio.com/) или любой другой редактор кода.

> [!TIP]
> В этом руководстве изложены пошаговые инструкции по созданию приложения. Готовое приложение можно скачать из [репозитория angular-cosmosdb](https://github.com/Azure-Samples/angular-cosmosdb) на GitHub.

## <a name="install-the-angular-cli-and-typescript"></a>установка Angular CLI и TypeScript;

1. Откройте командную строку Windows или окно терминала Mac и установите Angular CLI.

    ```bash
    npm install -g @angular/cli
    ```

2. Введите следующую команду в командной строке, чтобы установить TypeScript. 

    ```bash
    npm install -g typescript
    ```

## <a name="use-the-angular-cli-to-create-a-new-project"></a>Создание проекта с помощью Angular CLI

1. В командной строке укажите папку, в которой требуется создать новый проект, а затем выполните приведенную ниже команду. Эта команда создает новую папку и проект с именем angular-cosmosdb и устанавливает компоненты Angular, необходимые для нового приложения. Она использует минимальные настройки (--minimal) и указывает, что в проекте используется SASS (CSS-подобный синтаксис с флагом ---style scss).

    ```bash
    ng new angular-cosmosdb --minimal --style scss
    ```

2. После выполнения команды перейдите в папку src/client.

    ```bash
    cd angular-cosmosdb
    ```

3. Затем откройте папку в Visual Studio Code.

    ```bash
    code .
    ```

## <a name="build-the-app-using-the-express-framework"></a>Создание приложения с помощью платформы Express

1. В Visual Studio Code в области **Обозреватель** щелкните правой кнопкой мыши папку **src**, выберите пункт **Создать папку** и присвойте новой папке имя *server*.

2. В области **Обозреватель** щелкните правой кнопкой мыши папку **server**, выберите **Создать файл** и присвойте новому файлу имя *index.js*.

3. Вернитесь в командную строку и введите указанную ниже команду, чтобы установить средство синтаксического анализа текста. Это поможет нашему приложению проанализировать данные JSON, передаваемые через API-интерфейсы.

    ```bash
    npm i express body-parser --save
    ```

4. В Visual Studio Code скопируйте приведенный ниже код в файл index.js. Этот код выполняет следующие действия:
    * ссылается на платформу Express;
    * вызывает средство синтаксического анализа текста для чтения данных JSON в тексте запросов;
    * использует встроенную функцию path;
    * задает переменные для корневого каталога, чтобы было проще найти расположение кода;
    * задает порт;
    * запускает платформу Express;
    * указывает приложению, как следует использовать ПО промежуточного слоя, которое будет применяться для обслуживания сервера;
    * отображает все содержимое папки dist (статическое содержимое);
    * обслуживает приложение и отображает файл index.html для запросов GET, не найденных на сервере (для прямых ссылок);
    * запускает сервер с помощью команды app.listen;
    * использует стрелочную функцию для проверки активности порта.
    
   ```node
   const express = require('express');
   const bodyParser = require('body-parser');
   const path = require('path');
   const routes = require('./routes');

   const root = './';
   const port = process.env.PORT || '3000';
   const app = express();

   app.use(bodyParser.json());
   app.use(bodyParser.urlencoded({ extended: false }));
   app.use(express.static(path.join(root, 'dist/angular-cosmosdb')));
   app.use('/api', routes);
   app.get('*', (req, res) => {
     res.sendFile('dist/angular-cosmosdb/index.html', {root});
   });

   app.listen(port, () => console.log(`API running on localhost:${port}`));
   ```

5. В Visual Studio Code в области **Обозреватель** щелкните правой кнопкой мыши папку **server** и выберите пункт **Создать файл**. Присвойте новому файлу имя *routes.js*. 

6. Скопируйте приведенный ниже код в файл **routes.js**. Этот код выполняет следующие действия:
   * ссылается на маршрутизатор Express;
   * получает элементы hero;
   * отправляет данные JSON для определенного элемента hero.

   ```node
   const express = require('express');
   const router = express.Router();

   router.get('/heroes', (req, res) => {
    res.send(200, [
       {"id": 10, "name": "Starlord", "saying": "oh yeah"}
    ])
   });

   module.exports=router;
   ```

7. Сохраните все измененные файлы. 

8. В Visual Studio Code нажмите кнопку **Отладка** (:::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part2/debug-button.png":::), а затем нажмите кнопку с шестеренкой :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part2/gear-button.png":::. В Visual Studio Code откроется новый файл launch.json. 

8. В 11-й строке файла launch.json замените `"${workspaceFolder}\\server"` на `"program": "${workspaceRoot}/src/server/index.js"` и сохраните файл.

9. Чтобы запустить приложение, нажмите кнопку **Начать отладку** (:::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part2/start-debugging-button.png":::).

    Приложение должно запуститься без ошибок.

## <a name="use-postman-to-test-the-app"></a>Тестирование приложения с помощью Postman

1. Теперь откройте Postman и введите `http://localhost:3000/api/heroes` в поле GET (Получить). 

2. Нажмите кнопку **Send** (Отправить) и получите от приложения ответ в формате JSON. 

    Этот ответ показывает, что приложение работает локально. 

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part2/azure-cosmos-db-postman.png" alt-text="Запрос и ответ в Postman":::


## <a name="next-steps"></a>Дальнейшие действия

В этой части руководства мы выполнили следующую задачу:

> [!div class="checklist"]
> * создали проект Node.js с помощью Angular CLI;
> * тестировали приложение с помощью Postman.

Теперь можно перейти к следующей части руководства, чтобы создать пользовательский интерфейс.

> [!div class="nextstepaction"]
> [создание пользовательского интерфейса с помощью Angular](tutorial-develop-mongodb-nodejs-part3.md);
