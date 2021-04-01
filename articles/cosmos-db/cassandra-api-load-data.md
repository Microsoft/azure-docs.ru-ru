---
title: Руководство. Использование приложения Java для загрузки примеров данных в таблицу API Cassandra в Azure Cosmos DB
description: Данное руководство описывает, как в Azure Cosmos DB загрузить пример данных пользователя в таблицу учетной записи API Cassandra с помощью приложения Java.
author: kanshiG
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: tutorial
ms.date: 05/20/2019
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: ad3dfc8c86d22e58377ebfbdb94b2beb004ded56
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93092472"
---
# <a name="tutorial-load-sample-data-into-a-cassandra-api-table-in-azure-cosmos-db"></a>Руководство. Загрузка примера данных в таблицу API Cassandra в Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Как у разработчика у вас должно быть приложение, использующее пары "ключ-значение". Можно использовать учетную запись Cassandra API в Azure Cosmos DB для хранения данных "ключ-значение" и управления ими. Данное руководство описывает, как в Azure Cosmos DB загрузить пример данных пользователя в таблицу в учетной записи API Cassandra с помощью приложения Java. Приложение Java использует [драйвер Java](https://github.com/datastax/java-driver) и загружает данные пользователя, например, идентификатор пользователя, имя пользователя, город пользователя. 

В рамках этого руководства рассматриваются следующие задачи:

> [!div class="checklist"]
> * Загрузка данных в таблицу Cassandra
> * Запустите приложение

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Эта статья относится к руководству из нескольких частей. Перед началом чтения документа убедитесь, что [вы создали учетную запись API Cassandra, ключевое пространство и таблицу](create-cassandra-api-account-java.md).   

## <a name="load-data-into-the-table"></a>Загрузка данных в таблицу

Чтобы загрузить данные в таблицу API Cassandra, выполните следующие шаги:

1. Откройте файл "UserRepository.java" в папке "src\main\java\com\azure\cosmosdb\cassandra" и добавьте код для вставки полей user_id, user_name и user_bcity в таблицу.

   ```java
   /**
   * Insert a row into user table
   *
   * @param id   user_id
   * @param name user_name
   * @param city user_bcity
   */
   public void insertUser(PreparedStatement statement, int id, String name, String city) {
        BoundStatement boundStatement = new BoundStatement(statement);
        session.execute(boundStatement.bind(id, name, city));
   }

   /**
   * Create a PrepareStatement to insert a row to user table
   *
   * @return PreparedStatement
   */
   public PreparedStatement prepareInsertStatement() {
      final String insertStatement = "INSERT INTO  uprofile.user (user_id, user_name , user_bcity) VALUES (?,?,?)";
   return session.prepare(insertStatement);
   }
   ```
 
2. Откройте файл "UserProfile.java" в папке "src\main\java\com\azure\cosmosdb\cassandra". Этот класс содержит основной метод, который вызывает методы createKeyspace и createTable, определенные вами ранее. Теперь добавьте следующий код, чтобы вставить демонстрационные данные в таблицу Cassandra API.

   ```java
   //Insert rows into user table
   PreparedStatement preparedStatement = repository.prepareInsertStatement();
     repository.insertUser(preparedStatement, 1, "JohnH", "Seattle");
     repository.insertUser(preparedStatement, 2, "EricK", "Spokane");
     repository.insertUser(preparedStatement, 3, "MatthewP", "Tacoma");
     repository.insertUser(preparedStatement, 4, "DavidA", "Renton");
     repository.insertUser(preparedStatement, 5, "PeterS", "Everett");
   ```

## <a name="run-the-app"></a>Запустите приложение

Откройте командную строку или окно терминала и измените путь к папке, в которой был создан проект. Выполните команду "mvn clean install", чтобы в целевой папке создать файл cosmosdb-cassandra-examples.jar и запустить приложение. 

```bash
cd "cassandra-demo"

mvn clean install

java -cp target/cosmosdb-cassandra-examples.jar com.azure.cosmosdb.cassandra.examples.UserProfile
```

Теперь вы можете открыть обозреватель данных на портале Azure, чтобы убедиться, что сведения о пользователе добавлены в таблицу.
    
## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как в Azure Cosmos DB загрузить демонстрационные данные в учетную запись Cassandra API. Теперь вы можете перейти к следующей статье:

> [!div class="nextstepaction"]
> [Запрос данных из учетной записи Cassandra API](cassandra-api-query-data.md)
