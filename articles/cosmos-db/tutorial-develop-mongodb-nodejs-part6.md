---
title: Добавление функций CRUD в приложение Angular с помощью API Azure Cosmos DB для MongoDB
description: Часть 6 из серии руководств по созданию в Azure Cosmos DB приложения MongoDB с помощью Angular и Node и тех же API, которые используются для MongoDB
author: johnpapa
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 12/26/2018
ms.author: jopapa
ms.custom: seodec18, devx-track-js
ms.reviewer: sngun
ms.openlocfilehash: c8e2c707566b08219b495e76be7f6f6130d876ab
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93081320"
---
# <a name="create-an-angular-app-with-azure-cosmos-dbs-api-for-mongodb---add-crud-functions-to-the-app"></a>Создание приложения Angular с помощью API Azure Cosmos DB для MongoDB. Добавление функций CRUD в приложение
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Из этого руководства в нескольких частях вы узнаете, как создать новое приложение, написанное на Node.js с использованием Express и Angular, а затем подключить его к [учетной записи Cosmos, настроенной с помощью API Cosmos DB для MongoDB](mongodb-introduction.md). Часть 6 руководства основана на [части 5](tutorial-develop-mongodb-nodejs-part5.md). В этой части рассматриваются следующие задачи:

> [!div class="checklist"]
> * Создание функций Post, Put и Delete для службы hero
> * Запустите приложение

> [!VIDEO https://www.youtube.com/embed/Y5mdAlFGZjc]

## <a name="prerequisites"></a>Предварительные требования

Прежде чем переходить к этой части руководства, следует выполнить все задачи из [части 5](tutorial-develop-mongodb-nodejs-part5.md).

> [!TIP]
> В этом руководстве изложены пошаговые инструкции по созданию приложения. Готовое приложение можно скачать из [репозитория angular-cosmosdb](https://github.com/Azure-Samples/angular-cosmosdb) на GitHub.

## <a name="add-a-post-function-to-the-hero-service"></a>Добавление функции Post в службу hero

1. Откройте в Visual Studio Code файлы **routes.js** и **hero.service.js** параллельно. Для этого нажмите кнопку **Разделить редактор** (:::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part6/split-editor-button.png":::).

    Вы увидите, что строка 7 файла routes.js вызывает функцию `getHeroes` в строке 5 файла **hero.service.js**.  Необходимо создать такие же пары для функций Post, Put и Delete. 

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part6/routes-heroservicejs.png" alt-text="Файлы routes.js и hero.service.js в Visual Studio Code":::
    
    Начнем с написания кода для службы hero. 

2. Вставьте следующий код в файл **hero.service.js** после функции `getHeroes` и перед `module.exports`. Этот код выполняет следующие действия:  
   * Использует модель hero для публикации нового элемента hero.
   * Проверяет ответы на наличие ошибок и возвращает значение состояния 500.

   ```javascript
   function postHero(req, res) {
     const originalHero = { uid: req.body.uid, name: req.body.name, saying: req.body.saying };
     const hero = new Hero(originalHero);
     hero.save(error => {
       if (checkServerError(res, error)) return;
       res.status(201).json(hero);
       console.log('Hero created successfully!');
     });
   }

   function checkServerError(res, error) {
     if (error) {
       res.status(500).send(error);
       return error;
     }
   }
   ```

3. В файле **hero.service.js** обновите `module.exports`, чтобы включить новую функцию `postHero`. 

    ```javascript
    module.exports = {
      getHeroes,
      postHero
    };
    ```

4. В файле **routes.js** добавьте маршрутизатор для функции `post` работу после маршрутизатора `get`. Этот маршрутизатор публикует один элемент hero в один момент времени. При таком структурировании файла маршрутизатора отображаются все доступные конечные точки API и делегируются все задачи файлу **hero.service.js**.

    ```javascript
    router.post('/hero', (req, res) => {
      heroService.postHero(req, res);
    });
    ```

5. Запустите приложение, чтобы убедиться в его работоспособности. В Visual Studio Code сохраните все изменения, нажмите кнопку **Отладка** (:::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part6/debug-button.png":::) слева, а затем нажмите кнопку **Начать отладку** (:::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part6/start-debugging-button.png":::).

6. Теперь вернитесь в браузер и откройте вкладку с параметрами сети в инструментах разработчика. На большинстве компьютеров эта вкладка открывается нажатием клавиши F12. Перейдите по адресу `http://localhost:3000` для просмотра вызовов по сети.

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part6/add-new-hero.png" alt-text="Вкладка с параметрами сети в браузере Chrome с отображением действий в сети":::

7. Добавьте новый элемент hero, нажав кнопку **Add New Hero** (Добавить героя). Введите идентификатор 999, имя Fred и фразу Hello. Затем нажмите кнопку **Save** (Сохранить). На вкладке с параметрами сети вы увидите, отправленный запрос POST для нового элемента hero. 

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part6/post-new-hero.png" alt-text="Вкладка с параметрами сети в браузере Chrome с отображением функций Get и Post в сети":::

    Теперь добавим в приложение функции Put и Delete.

## <a name="add-the-put-and-delete-functions"></a>Добавление функций Put и Delete

1. В файле **routes.js** добавьте маршрутизаторы `put` и `delete` после маршрутизатора post.

    ```javascript
    router.put('/hero/:uid', (req, res) => {
      heroService.putHero(req, res);
    });

    router.delete('/hero/:uid', (req, res) => {
      heroService.deleteHero(req, res);
    });
    ```

2. Вставьте следующий код в файл **hero.service.js** после функции `checkServerError`. Этот код выполняет следующие действия:
   * Создает функции `put` и `delete`.
   * Определяет наличие элемента hero.
   * Обрабатывает ошибки. 

   ```javascript
   function putHero(req, res) {
     const originalHero = {
       uid: parseInt(req.params.uid, 10),
       name: req.body.name,
       saying: req.body.saying
     };
     Hero.findOne({ uid: originalHero.uid }, (error, hero) => {
       if (checkServerError(res, error)) return;
       if (!checkFound(res, hero)) return;

      hero.name = originalHero.name;
       hero.saying = originalHero.saying;
       hero.save(error => {
         if (checkServerError(res, error)) return;
         res.status(200).json(hero);
         console.log('Hero updated successfully!');
       });
     });
   }

   function deleteHero(req, res) {
     const uid = parseInt(req.params.uid, 10);
     Hero.findOneAndRemove({ uid: uid })
       .then(hero => {
         if (!checkFound(res, hero)) return;
         res.status(200).json(hero);
         console.log('Hero deleted successfully!');
       })
       .catch(error => {
         if (checkServerError(res, error)) return;
       });
   }

   function checkFound(res, hero) {
     if (!hero) {
       res.status(404).send('Hero not found.');
       return;
     }
     return hero;
   }
   ```

3. В файле **hero.service.js** экспортируйте новые модули:

   ```javascript
    module.exports = {
      getHeroes,
      postHero,
      putHero,
      deleteHero
    };
    ```

4. Обновив код, нажмите кнопку **Перезапустить** (:::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part6/restart-debugger-button.png":::) в Visual Studio Code.

5. Обновите страницу в браузере и нажмите кнопку **Add New Hero** (Добавить героя). Добавьте новый элемент hero с идентификатором 9, именем Starlord и фразой "Hi". Нажмите кнопку **Save** (Сохранить), чтобы сохранить элемент hero.

6. Теперь выберите героя **Starlord** и измените его фразу с Hi на Bye. Нажмите кнопку **Save** (Сохранить). 

    Теперь выберите идентификатор на вкладке с параметрами сети, чтобы отобразить полезные данные. Вы увидите в полезных данных, что фраза героя поменялась на "Bye".

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part6/put-hero-function.png" alt-text="Приложение Heroes на вкладке с параметрами сети с отображением полезных данных"::: 

    Вы можете удалить из пользовательского интерфейса один элемент hero, чтобы узнать, сколько времени занимает операция удаления. Попробуйте нажать кнопку Delete (Удалить) рядом с героем Fred.

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part6/times.png" alt-text="Приложение Heroes на вкладке с параметрами сети с отображением времени выполнения функций"::: 

    Если обновить страницу, на вкладке с параметрами сети отобразится время, требуемое для получения элементов hero. Хотя эти операции выполняются быстро, многое зависит от расположения данных в мире и вашей возможности выполнять георепликацию в ближайшую к пользователям область. Дополнительные сведения о георепликации см. в следующем руководстве, которое скоро появится.

## <a name="next-steps"></a>Дальнейшие действия

В этой части руководства мы выполнили следующую задачу:

> [!div class="checklist"]
> * добавили функции Post, Put и Delete в приложение. 

Скоро в этой серии руководств появятся дополнительные видео. Следите за обновлениями.

