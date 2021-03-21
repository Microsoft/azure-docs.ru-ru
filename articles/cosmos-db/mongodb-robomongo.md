---
title: Подключение к Azure Cosmos DB с помощью Robo 3T
description: Сведения о подключении к Azure Cosmos DB с помощью Robo 3T и API Azure Cosmos DB для MongoDB
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 03/23/2020
author: timsander1
ms.author: tisande
ms.openlocfilehash: e096de5eea9bf5bbd47b976976192e62eb6aa33e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93096417"
---
# <a name="use-robo-3t-with-azure-cosmos-dbs-api-for-mongodb"></a>Использование Robo 3T с API Azure Cosmos DB для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Чтобы подключиться к учетной записи Cosmos с помощью Robo 3T, сделайте следующее.

* Загрузите и установите [Robo 3T](https://robomongo.org/).
* Получите сведений о [строке подключения](connect-mongodb-account.md) Cosmos DB.

> [!NOTE]
> В настоящее время совместная работа 3T v 1.2 и более ранние версии поддерживаются API Cosmos DB для MongoDB.

## <a name="connect-using-robo-3t"></a>Подключение с помощью Robo 3T

Чтобы добавить в диспетчер подключений Robo 3T учетную запись Cosmos, сделайте следующее.

1. Получите сведения о подключении для учетной записи Cosmos, настроенной с помощью API Azure Cosmos DB для MongoDB, следуя инструкциям [здесь](connect-mongodb-account.md).

    :::image type="content" source="./media/mongodb-robomongo/connectionstringblade.png" alt-text="Снимок экрана колонки строки подключения":::
2. Запустите приложение *Robomongo* .

3. Нажмите кнопку подключения под меню **File** (Файл) для управления подключениями. Щелкните **Create** (Создать) в окне **MongoDB Connections** (Подключения MongoDB). Откроется окно **Connection Settings** (Параметры подключения).

4. В окне **Connection Settings** (Параметры подключения) выберите имя. Затем найдите **узел** и **порт**, указанные в сведениях о подключении, полученных на шаге 1, и введите их в полях **Address** (Адрес) и **Port** (Порт) соответственно.

    :::image type="content" source="./media/mongodb-robomongo/manageconnections.png" alt-text="Снимок экрана Robomongo управления подключениями":::
5. На вкладке **Authentication** (Аутентификация) установите флажок **Perform authentication** (Выполнять аутентификацию). Затем введите базу данных (по умолчанию — *Admin*), **имя пользователя** и **пароль**.
**Имя пользователя** и **пароль** можно найти в данных подключения, полученных на шаге 1.

    :::image type="content" source="./media/mongodb-robomongo/authentication.png" alt-text="Снимок экрана вкладки проверки подлинности Robomongo":::
6. На вкладке **SSL** установите флажок **Use SSL protocol** (Использовать протокол SSL), затем измените значение параметра **Authentication Method** (Метод аутентификации) на **Self-signed Certificate** (Самозаверяющий сертификат).

    :::image type="content" source="./media/mongodb-robomongo/SSL.png" alt-text="Снимок экрана вкладки Robomongo SSL":::
7. Наконец, нажмите кнопку **Test** (Проверить), чтобы проверить возможность подключения, затем нажмите кнопку **Save** (Сохранить).

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [использовать Studio 3T](mongodb-mongochef.md) с API Azure Cosmos DB для MongoDB.
- Ознакомьтесь с [примерами](mongodb-samples.md) MongoDB с API Azure Cosmos DB для MongoDB.
