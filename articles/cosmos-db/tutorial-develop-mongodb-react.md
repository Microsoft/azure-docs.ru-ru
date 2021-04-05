---
title: Руководство по MongoDB, React и Node.js для Azure
description: Узнайте, как создать в Azure Cosmos DB приложение MongoDB с помощью React и Node.js и тех же API-интерфейсов, которые используются для MongoDB в этой серии руководств с видео.
author: johnpapa
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 09/05/2018
ms.author: jopapa
ms.reviewer: sngun
ms.custom: devx-track-js
ms.openlocfilehash: 1425b89e42450123c1696ddcee4458e1f69b8a6c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96348574"
---
# <a name="create-a-mongodb-app-with-react-and-azure-cosmos-db"></a>Создание приложения MongoDB с помощью React и Azure Cosmos DB  
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

В этом руководстве с видео, состоящем из нескольких частей, показано, как создать приложение с внешним интерфейсом React, которое используется для отслеживания героев (элементов hero). Приложение, серверная часть которого создана с помощью Node и Express, подключается к базе данных Cosmos DB с помощью [API Azure Cosmos DB для MongoDB](mongodb-introduction.md), а затем подключает внешний интерфейс React к серверной части. В этом руководстве также демонстрируется, как реализовать интерактивное масштабирование Cosmos DB на портале Azure и как развернуть приложение в Интернете, чтобы любой пользователь мог отслеживать своих любимых героев (избранные элементы hero). 

[Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) поддерживает совместимость коммуникационных протоколов с MongoDB, позволяя клиентам использовать Azure Cosmos DB вместо MongoDB.  

В этом руководстве из нескольких частей рассматриваются следующие задачи:

> [!div class="checklist"]
> * Введение
> * настройка проекта;
> * создание пользовательского интерфейса с помощью React;
> * создание учетной записи Azure Cosmos DB с помощью портала Azure;
> * подключение к Azure Cosmos DB с помощью Mongoose;
> * добавление в приложение операций React: создания, обновления и удаления.

Хотите создать такое же приложение с помощью Angular? См. [серию видеоруководств по Angular](tutorial-develop-mongodb-nodejs.md).

## <a name="prerequisites"></a>Предварительные требования
* [Node.js](https://www.nodejs.org)

### <a name="finished-project"></a>Готовый проект
Готовое приложение можно скачать с [сайта GitHub](https://github.com/Azure-Samples/react-cosmosdb).

## <a name="introduction"></a>Введение 

В этом видеоролике Берк Холланд (Burke Holland) познакомит вас с Azure Cosmos DB и расскажет о приложении, которое создается в этой серии видеоруководств. 

> [!VIDEO https://www.youtube.com/embed/58IflnJbYJc]

## <a name="project-setup"></a>Настройка проекта

В этом видеоролике показано, как настроить использование Express и React в одном проекте. После этого Берк подробно опишет код проекта.

> [!VIDEO https://www.youtube.com/embed/ytFUPStJJds]

## <a name="build-the-ui"></a>Создание пользовательского интерфейса

В этом видеоролике показано, как создать пользовательский интерфейс приложения с помощью React. 

> [!NOTE]
> CSS-файл, который упоминается в этом видеоролике, можно найти в [репозитории GitHub react-cosmosdb](https://github.com/Azure-Samples/react-cosmosdb/blob/master/src/index.css).

> [!VIDEO https://www.youtube.com/embed/SzHzX0fTUUQ]

## <a name="connect-to-azure-cosmos-db"></a>Подключение к Azure Cosmos DB

В этом видеоролике показано, как создать учетную запись Azure Cosmos DB на портале Azure, установить пакеты MongoDB и Mongoose и подключить приложение к новой учетной записи с помощью строки подключения Azure Cosmos DB. 

> [!VIDEO https://www.youtube.com/embed/0U2jV1thfvs]

## <a name="read-and-create-heroes-in-the-app"></a>Считывание и создание элементов hero в приложении

В этом видео показано, как считывать и создавать элементы hero в базе данных Cosmos, а также как проверить эти методы с помощью Postman и пользовательского интерфейса React. 

> [!VIDEO https://www.youtube.com/embed/AQK9n_8fsQI] 

## <a name="delete-and-update-heroes-in-the-app"></a>Удаление и изменение элементов hero в приложении

В этом видеоролике показано, как удалять и обновлять элементы hero в приложении и как отображать обновления в пользовательском интерфейсе. 

> [!VIDEO https://www.youtube.com/embed/YmaGT7ztTQM] 

## <a name="complete-the-app"></a>Завершение создания приложения

В этом видео показано, как завершить создание приложения и подключение пользовательского интерфейса к API серверной части. 

> [!VIDEO https://www.youtube.com/embed/TcSm2ISfTu8]

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение дальше, удалите все ресурсы, созданные в ходе работы с этим руководством, на портале Azure, выполнив следующие действия. 

1. В меню слева на портале Azure щелкните **Группы ресурсов**, а затем выберите имя созданного ресурса. 
2. На странице группы ресурсов щелкните **Удалить**, в текстовом поле введите имя ресурса для удаления и щелкните **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как выполнять такие задачи:

> [!div class="checklist"]
> * создание приложения с помощью React, Node, Express и Azure Cosmos DB; 
> * Создание учетной записи Azure Cosmos DB
> * подключение приложения к учетной записи Azure Cosmos DB;
> * тестирование приложения с помощью Postman;
> * запуск приложения и добавление элементов hero в базу данных.

Вы можете перейти к следующему руководству, из которого вы узнаете, как импортировать данные MongoDB в Azure Cosmos DB.  

> [!div class="nextstepaction"]
> [Перенос данных MongoDB в Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)
