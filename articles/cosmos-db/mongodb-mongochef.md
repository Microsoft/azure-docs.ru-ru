---
title: Подключение к API Azure Cosmos DB для MongoDB с помощью Studio 3T
description: Узнайте, как подключиться к API Azure Cosmos DB для MongoDB с помощью 3T Studio.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 03/20/2020
author: timsander1
ms.author: tisande
ms.custom: seodec18
ms.openlocfilehash: a02aaadf8c774557eb182acf041b6f19337a0de8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93096523"
---
# <a name="connect-to-an-azure-cosmos-account-using-studio-3t"></a>Подключение к учетной записи Azure Cosmos с помощью 3T Studio
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Чтобы подключиться к API Azure Cosmos DB для MongoDB с помощью 3T Studio, необходимо выполнить следующие действия.

* Скачайте и установите [Studio 3T](https://studio3t.com/).
* Сведения о [строке подключения](connect-mongodb-account.md) учетной записи Azure Cosmos.

## <a name="create-the-connection-in-studio-3t"></a>Создание подключения в Studio 3T

Чтобы добавить учетную запись Azure Cosmos в Диспетчер соединений Studio 3T, выполните следующие действия.

1. Получите сведения о подключении для учетной записи API Azure Cosmos DB для MongoDB, следуя инструкциям в статье [Подключение приложения MongoDB к Azure Cosmos DB](connect-mongodb-account.md) .

    :::image type="content" source="./media/mongodb-mongochef/ConnectionStringBlade.png" alt-text="Снимок экрана со страницей строки подключения":::

2. Щелкните **Connect** (Подключиться), чтобы открыть диспетчер подключений, и нажмите кнопку **New Connection** (Новое подключение).

    :::image type="content" source="./media/mongodb-mongochef/ConnectionManager.png" alt-text="Снимок экрана диспетчера соединений Studio 3T, который выделяет новую кнопку подключения.":::
3. В окне **новое подключение** на вкладке **сервер** введите узел (FQDN) учетной записи Azure Cosmos и порта.

    :::image type="content" source="./media/mongodb-mongochef/ConnectionManagerServerTab.png" alt-text="Снимок экрана: вкладка &quot;сервер диспетчера подключений Studio 3T&quot;":::
4. В окне **New Connection** (Новое подключение) на вкладке **Authentication** (Аутентификация) выберите режим аутентификации **Basic (MONGODB-CR or SCARM-SHA-1)** (Базовая (MONGODB CR или SCARM-SHA-1)), а также введите имя пользователя и пароль.  Подтвердите базу данных по умолчанию для проверки подлинности (admin) или укажите другое значение.

    :::image type="content" source="./media/mongodb-mongochef/ConnectionManagerAuthenticationTab.png" alt-text="Снимок экрана с вкладкой проверки подлинности диспетчера подключений Studio 3T":::
5. В окне **New Connection** (Новое подключение) на вкладке **SSL** установите флажок **Use SSL protocol to connect** (Использовать для подключения протокол SSL) и переключатель **Accept server self-signed SSL certificates** (Принимать самозаверяющие SSL-сертификаты сервера).

    :::image type="content" source="./media/mongodb-mongochef/ConnectionManagerSSLTab.png" alt-text="Снимок экрана с вкладкой SSL 3T Connection Manager":::
6. Нажмите кнопку **Test Connection** (Проверить подключение), чтобы проверить сведения о подключении. Затем нажмите кнопку **ОК**, чтобы вернуться в окно "New Connection" (Новое подключение), а затем нажмите кнопку **Save** (Сохранить).

    :::image type="content" source="./media/mongodb-mongochef/TestConnectionResults.png" alt-text="Снимок экрана: окно &quot;Проверка подключения&quot; 3T Studio":::

## <a name="use-studio-3t-to-create-a-database-collection-and-documents"></a>Использование Studio 3T для создания базы данных, коллекции и документов
Чтобы создать базу данных, коллекцию и документы с помощью Studio 3T, выполните следующие действия.

1. В **диспетчере подключений** выделите нужное подключение и щелкните **Connect** (Подключиться).

    :::image type="content" source="./media/mongodb-mongochef/ConnectToAccount.png" alt-text="Снимок экрана диспетчера соединений 3T Studio":::
2. Щелкните узел правой кнопкой мыши и выберите **Add Database** (Добавить базу данных).  Укажите имя базы данных и нажмите кнопку **ОК**.

    :::image type="content" source="./media/mongodb-mongochef/AddDatabase1.png" alt-text="Снимок экрана с параметром &quot;добавить базу данных&quot; в Studio 3T":::
3. Щелкните правой кнопкой мыши базу данных и выберите **Add Collection** (Добавить коллекцию).  Укажите имя коллекции и нажмите кнопку **Создать**.

    :::image type="content" source="./media/mongodb-mongochef/AddCollection.png" alt-text="Снимок экрана с параметром &quot;добавить коллекцию&quot; в Studio 3T":::
4. Щелкните пункт меню **Collection** (Коллекция), затем щелкните **Add Document** (Добавить документ).

    :::image type="content" source="./media/mongodb-mongochef/AddDocument1.png" alt-text="Снимок экрана: пункт меню &quot;добавить документ&quot; в 3Tе Studio":::
5. В диалоговом окне "Добавление документа" вставьте следующий текст и щелкните **Добавить документ**.

    ```json
    {
        "_id": "AndersenFamily",
        "lastName": "Andersen",
        "parents": [
            { "firstName": "Thomas" },
            { "firstName": "Mary Kay"}
        ],
        "children": [
            {
                "firstName": "Henriette Thaulow", "gender": "female", "grade": 5,
                "pets": [{ "givenName": "Fluffy" }]
            }
        ],
        "address": { "state": "WA", "county": "King", "city": "seattle" },
        "isRegistered": true
    }
    ```
    
6. Добавьте еще один документ со следующим содержимым:

    ```json
    {
        "_id": "WakefieldFamily",
        "parents": [
            { "familyName": "Wakefield", "givenName": "Robin" },
            { "familyName": "Miller", "givenName": "Ben" }
        ],
        "children": [
            {
                "familyName": "Merriam",
                "givenName": "Jesse",
                "gender": "female", "grade": 1,
                "pets": [
                    { "givenName": "Goofy" },
                    { "givenName": "Shadow" }
                ]
            },
            {
                "familyName": "Miller",
                "givenName": "Lisa",
                "gender": "female",
                "grade": 8 }
        ],
        "address": { "state": "NY", "county": "Manhattan", "city": "NY" },
        "isRegistered": false
    }
    ```

7. Выполните пробный запрос. Например, попробуйте найти семьи с фамилией Andersen и вернуть для них поля parents и state.

    :::image type="content" source="./media/mongodb-mongochef/QueryDocument1.png" alt-text="Снимок экрана результатов запроса Mongo Chef":::

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [использовать Robo 3T](mongodb-robomongo.md) с API Azure Cosmos DB для MongoDB.
- Ознакомьтесь с [примерами](mongodb-samples.md) MongoDB с API Azure Cosmos DB для MongoDB.
