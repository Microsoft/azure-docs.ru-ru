---
title: Подключение приложения Mongoose Node.js к Azure Cosmos DB
description: Узнайте, как использование платформу Mongoose для хранения и администрирования данных в Azure Cosmos DB.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: how-to
ms.date: 03/20/2020
author: timsander1
ms.author: tisande
ms.custom: seodec18, devx-track-js
ms.openlocfilehash: 8958699ae279d2613f8dbadca802ee2137407e75
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94442418"
---
# <a name="connect-a-nodejs-mongoose-application-to-azure-cosmos-db"></a>Подключение приложения Mongoose Node.js к Azure Cosmos DB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Это руководство содержит сведения об использовании [платформы Mongoose](https://mongoosejs.com/) при сохранении данных в Cosmos DB. В этом пошаговом руководстве используется API Azure Cosmos DB для MongoDB. Для тех из вас, кто еще не знаком с Mongoose, — это платформа объектного моделирования для MongoDB в Node.js, которая представляет собой простое решение на основе схемы для моделирования данных приложения.

Cosmos DB — это глобально распределенная многомодельная служба базы данных Майкрософт. Вы можете быстро создавать и запрашивать документы, пары "ключ-значение" и графовые базы данных, используя преимущества глобального распределения и горизонтального масштабирования Cosmos DB.

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

[Node.js](https://nodejs.org/) версии v 0.10.29 или выше.

## <a name="create-a-cosmos-account"></a>Создание учетной записи Cosmos

Создадим учетную запись Cosmos. Если у вас уже есть учетная запись, которую вы собираетесь использовать, можно перейти к шагу "Настройка приложения Node.js". Если вы используете эмулятор Azure Cosmos DB, выполните действия, описанные в разделе [эмулятор Azure Cosmos DB](local-emulator.md) , чтобы настроить эмулятор и сразу перейти к настройке приложения Node.js.

[!INCLUDE [cosmos-db-create-dbaccount-mongodb](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

### <a name="create-a-database"></a>Создание базы данных 
В этом приложении мы рассмотрим два способа создания коллекций в Azure Cosmos DB: 
- **Хранение каждой объектной модели в отдельной коллекции**: рекомендуется [создать базу данных с выделенной пропускной способностью](set-throughput.md#set-throughput-on-a-database). Использование этой модели емкости обеспечит более эффективную экономичность.

    :::image type="content" source="./media/mongodb-mongoose/db-level-throughput.png" alt-text="Node.js учебник — снимок экрана портал Azure, в котором показано, как создать базу данных в обозреватель данных для учетной записи Azure Cosmos DB для использования с модулем узла Mongoose.":::

- **Сохранение всех объектных моделей в одной коллекции Cosmos DB**. Если вы предпочитаете хранить все модели в одной коллекции, можно просто создать новую базу данных, не выбрав параметр "обеспечить пропускную способность". При использовании этой модели емкости каждая коллекция будет создана с собственной пропускной способностью для каждой объектной модели.

После создания базы данных имя будет использоваться в `COSMOSDB_DBNAME` переменной среды, приведенной ниже.

## <a name="set-up-your-nodejs-application"></a>Настройка приложения Node.js

>[!Note]
> Если вы просто хотите ознакомиться с примером кода, а не настраивать приложение, клонируйте [пример](https://github.com/Azure-Samples/Mongoose_CosmosDB), используемый для этого руководства, и создайте приложение Mongoose на Node.js в Azure Cosmos DB.

1. Чтобы создать приложение Node.js в выбранной папке, выполните следующую команду в командной строке узла.

    ```npm init```

    Ответьте на вопросы, чтобы приступить к работе с проектом.

2. Добавьте новый файл в папку и назовите его ```index.js```.
3. Установите необходимые пакеты, используя один из параметров ```npm install```.
   * Mongoose: ```npm install mongoose@5 --save```.

     > [!Note]
     > Подключение примера Mongoose, приведенного ниже, основанно на Mongoose 5+. Процесс подключения изменился с выходом более поздних версий.
    
   * Dotenv (если вы хотите загрузить секреты из ENV-файла): ```npm install dotenv --save```.

     >[!Note]
     > Параметр ```--save``` позволяет добавить зависимость в файл package.json.

4. Импортируйте зависимости в файл index.js.

    ```JavaScript
   var mongoose = require('mongoose');
   var env = require('dotenv').config();   //Use the .env file to load the variables
    ```

5. Добавьте строку подключения и имя Cosmos DB в файл ```.env```. Замените заполнители {Cosmos-Account-Name} и {dbname} собственным именем учетной записи Cosmos и именем базы данных, без символов фигурных скобок.

    ```JavaScript
   # You can get the following connection details from the Azure portal. You can find the details on the Connection string pane of your Azure Cosmos account.

   COSMOSDB_USER = "<Azure Cosmos account's user name, usually the database account name>"
   COSMOSDB_PASSWORD = "<Azure Cosmos account password, this is one of the keys specified in your account>"
   COSMOSDB_DBNAME = "<Azure Cosmos database name>"
   COSMOSDB_HOST= "<Azure Cosmos Host name>"
   COSMOSDB_PORT=10255
    ```

6. Подключитесь к Cosmos DB с помощью платформы Mongoose, добавив следующий код в конец файла index.js.
    ```JavaScript
   mongoose.connect("mongodb://"+process.env.COSMOSDB_HOST+":"+process.env.COSMOSDB_PORT+"/"+process.env.COSMOSDB_DBNAME+"?ssl=true&replicaSet=globaldb", {
      auth: {
        user: process.env.COSMOSDB_USER,
        password: process.env.COSMOSDB_PASSWORD
      },
    useNewUrlParser: true,
    useUnifiedTopology: true,
    retryWrites: false
    })
    .then(() => console.log('Connection to CosmosDB successful'))
    .catch((err) => console.error(err));
    ```
    >[!Note]
    > Здесь переменные среды загружаются в виде process.env.{имя_переменной}, используя пакет npm dotenv.

    Подключившись к Azure Cosmos DB, вы можете приступить к настройке объектных моделей в Mongoose.

## <a name="best-practices-for-using-mongoose-with-cosmos-db"></a>Рекомендации по использованию Mongoose с Cosmos DB

Для каждой создаваемой вами модели Mongoose создает новую коллекцию. Это лучше решить с помощью [параметра пропускной способности уровня базы данных](set-throughput.md#set-throughput-on-a-database), который ранее обсуждался. Чтобы использовать одну коллекцию, необходимо использовать [Дискриминатор](https://mongoosejs.com/docs/discriminators.html)Mongoose. "Дискриминаторы" — это механизм наследования схемы. Он позволяет иметь несколько моделей с пересекающимися схемами поверх той же базовой коллекции MongoDB.

Вы можете хранить различные модели данных в одной и той же коллекции, а затем использовать предложения фильтра во время запроса, чтобы извлекать только необходимые данные. Давайте подробно рассмотрим каждую из моделей.

### <a name="one-collection-per-object-model"></a>Одна коллекция на объектную модель

В этом разделе описано, как это сделать с помощью API Azure Cosmos DB для MongoDB. Этот метод является рекомендуемым, поскольку он позволяет управлять затратами и емкостью. В результате количество единиц запросов в базе данных не зависит от количества объектных моделей. Это операционная модель по умолчанию для Mongoose, поэтому вы, возможно, знакомы с этим.

1. Откройте ```index.js``` еще раз.

1. Создайте определение схемы Family (семья).

    ```JavaScript
    const Family = mongoose.model('Family', new mongoose.Schema({
        lastName: String,
        parents: [{
            familyName: String,
            firstName: String,
            gender: String
        }],
        children: [{
            familyName: String,
            firstName: String,
            gender: String,
            grade: Number
        }],
        pets:[{
            givenName: String
        }],
        address: {
            country: String,
            state: String,
            city: String
        }
    }));
    ```

1. Создайте объект Family (семья).

    ```JavaScript
    const family = new Family({
        lastName: "Volum",
        parents: [
            { firstName: "Thomas" },
            { firstName: "Mary Kay" }
        ],
        children: [
            { firstName: "Ryan", gender: "male", grade: 8 },
            { firstName: "Patrick", gender: "male", grade: 7 }
        ],
        pets: [
            { givenName: "Buddy" }
        ],
        address: { country: "USA", state: "WA", city: "Seattle" }
    });
    ```

1. Наконец, сохраните объект в Cosmos DB. При этом в фоновом режиме будет создана коллекция.

    ```JavaScript
    family.save((err, saveFamily) => {
        console.log(JSON.stringify(saveFamily));
    });
    ```

1. Теперь создайте другие схему и объект. В этот раз создайте их для мест отдыха (VacationDestinations), которые могут заинтересовать семьи.
   1. Как и в прошлом примере, создайте схему.
      ```JavaScript
      const VacationDestinations = mongoose.model('VacationDestinations', new mongoose.Schema({
       name: String,
       country: String
      }));
      ```

   1. Создайте и сохраните пример объекта (к этой схеме можно добавить несколько объектов).
      ```JavaScript
      const vacaySpot = new VacationDestinations({
       name: "Honolulu",
       country: "USA"
      });

      vacaySpot.save((err, saveVacay) => {
       console.log(JSON.stringify(saveVacay));
      });
      ```

1. Теперь при входе на портал Azure вы увидите две коллекции, созданные в Cosmos DB.

   :::image type="content" source="./media/mongodb-mongoose/mongo-mutliple-collections.png" alt-text="Node.js учебник. снимок экрана портал Azure, отображающий учетную запись Azure Cosmos DB с выделенным именем нескольких коллекций — база данных узла":::

1. Наконец, выполните считывание данных из Cosmos DB. Так как используется стандартная операционная модель Mongoose, операции чтения идентичны таким же операциям с Mongoose.

    ```JavaScript
    Family.find({ 'children.gender' : "male"}, function(err, foundFamily){
        foundFamily.forEach(fam => console.log("Found Family: " + JSON.stringify(fam)));
    });
    ```

### <a name="using-mongoose-discriminators-to-store-data-in-a-single-collection"></a>Хранение данных в одной коллекции с помощью дискриминаторов Mongoose

Чтобы оптимизировать затраты каждой коллекции, в этом методе используются [дискриминаторы Mongoose](https://mongoosejs.com/docs/discriminators.html). Дискриминаторы дают возможность определить отличительный ключ, который позволяет хранить, различать и фильтровать разные объектные модели.

В этом примере вы создадите базовую объектную модель, определите отличительный ключ и добавите Family и VacationDestinations в качестве расширения к базовой модели.

1. Настройте базовую конфигурацию и определите ключ дискриминатора.

    ```JavaScript
    const baseConfig = {
        discriminatorKey: "_type", //If you've got a lot of different data types, you could also consider setting up a secondary index here.
        collection: "alldata"   //Name of the Common Collection
    };
    ```

1. Затем определите общую объектную модель.

    ```JavaScript
    const commonModel = mongoose.model('Common', new mongoose.Schema({}, baseConfig));
    ```

1. Теперь определите модель Family. Обратите внимание, что вместо ```mongoose.model``` используется ```commonModel.discriminator```. Кроме того, к схеме Mongoose добавлена базовая конфигурация. Итак, ```FamilyType``` — это ключ дискриминатора.

    ```JavaScript
    const Family_common = commonModel.discriminator('FamilyType', new     mongoose.Schema({
        lastName: String,
        parents: [{
            familyName: String,
            firstName: String,
            gender: String
        }],
        children: [{
            familyName: String,
            firstName: String,
           gender: String,
            grade: Number
        }],
        pets:[{
            givenName: String
        }],
        address: {
            country: String,
            state: String,
            city: String
        }
    }, baseConfig));
    ```

1. Аналогичным образом добавьте другую схему, на этот раз для VacationDestinations. В этом случае ключ дискриминатора — это ```VacationDestinationsType```.

    ```JavaScript
    const Vacation_common = commonModel.discriminator('VacationDestinationsType', new mongoose.Schema({
        name: String,
        country: String
    }, baseConfig));
    ```

1. Наконец, создайте объекты для модели и сохраните ее.
   1. Добавьте объекты к модели Family.
      ```JavaScript
      const family_common = new Family_common({
       lastName: "Volum",
       parents: [
           { firstName: "Thomas" },
           { firstName: "Mary Kay" }
       ],
       children: [
           { firstName: "Ryan", gender: "male", grade: 8 },
           { firstName: "Patrick", gender: "male", grade: 7 }
       ],
       pets: [
           { givenName: "Buddy" }
       ],
       address: { country: "USA", state: "WA", city: "Seattle" }
      });

      family_common.save((err, saveFamily) => {
       console.log("Saved: " + JSON.stringify(saveFamily));
      });
      ```

   1. Затем добавьте объекты к модели VacationDestinations и сохраните ее.
      ```JavaScript
      const vacay_common = new Vacation_common({
       name: "Honolulu",
       country: "USA"
      });

      vacay_common.save((err, saveVacay) => {
       console.log("Saved: " + JSON.stringify(saveVacay));
      });
      ```

1. Теперь, если вернуться на портал Azure, вы заметите, что у вас есть лишь одна коллекция ```alldata``` с данными Family и VacationDestinations.

   :::image type="content" source="./media/mongodb-mongoose/mongo-collections-alldata.png" alt-text="Node.js учебник. снимок экрана портал Azure, отображающий учетную запись Azure Cosmos DB, с выделенным именем коллекции — база данных узла":::

1. Кроме того, обратите внимание, что каждый объект имеет еще один атрибут с именем ```__type```, который позволяет различать две разные объектные модели.

1. Наконец, выполните считывание данных, хранящихся в Azure Cosmos DB. Mongoose позволяет фильтровать данные на основе модели. Таким образом, вам не нужно будет выполнять другие действия при считывании данных. Просто укажите свою модель (в этом случае ```Family_common```), и Mongoose выполнит фильтрацию по DiscriminatorKey.

    ```JavaScript
    Family_common.find({ 'children.gender' : "male"}, function(err, foundFamily){
        foundFamily.forEach(fam => console.log("Found Family (using discriminator): " + JSON.stringify(fam)));
    });
    ```

Как видите, работать с дискриминаторами Mongoose очень легко. Если у вас есть приложение, использующее платформу Mongoose, это руководство поможет настроить и запустить его с помощью API Azure Cosmos DB для MongoDB без большого количества изменений.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [использовать Studio 3T](mongodb-mongochef.md) с API Azure Cosmos DB для MongoDB.
- Узнайте, как [использовать Robo 3T](mongodb-robomongo.md) с API Azure Cosmos DB для MongoDB.
- Ознакомьтесь с [примерами](mongodb-samples.md) MongoDB с API Azure Cosmos DB для MongoDB.

[dbleveltp]: ./media/mongodb-mongoose/db-level-throughput.png
