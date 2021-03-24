---
title: Руководство по Запрашивание данных из учетной записи API Cassandra в Azure Cosmos DB
description: В этом руководстве описывается запрос данных пользователя из учетной записи API Cassandra для Azure Cosmos DB с помощью приложения Java.
ms.service: cosmos-db
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.subservice: cosmosdb-cassandra
ms.topic: tutorial
ms.date: 09/24/2018
ms.openlocfilehash: 693984ceaef1a71ebfbe4122c18a45a563c9c101
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93100445"
---
# <a name="tutorial-query-data-from-a-cassandra-api-account-in-azure-cosmos-db"></a>Руководство по Запрашивание данных из учетной записи API Cassandra в Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Как у разработчика у вас должно быть приложение, использующее пары "ключ-значение". Можно использовать учетную запись Cassandra API в Azure Cosmos DB для хранения и запроса данных "ключ-значение". В этом руководстве описывается, как в Azure Cosmos DB запрашивать данные пользователя из учетной записи API Cassandra с помощью приложения Java. Приложение Java использует [драйвер Java](https://github.com/datastax/java-driver) и запрашивает данные пользователя, например, идентификатор пользователя, имя пользователя, город пользователя. 

В рамках этого руководства рассматриваются следующие задачи:

> [!div class="checklist"]
> * Запрос данных из таблицы Cassandra
> * Запустите приложение

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Эта статья относится к руководству из нескольких частей. Прежде чем начать, обязательно выполните предыдущие шаги, чтобы создать учетную запись API Cassandra, пространство ключей и таблицу, а также [загрузить пример данных в эту таблицу](cassandra-api-load-data.md). 

## <a name="query-data"></a>Данные запросов

Чтобы запросить данные из учетной записи API Cassandra, выполните следующие шаги:

1. Откройте файл `UserRepository.java`, расположенный в папке `src\main\java\com\azure\cosmosdb\cassandra`. Добавьте в него приведенный ниже блок кода. Этот код предоставляет три метода: 

   * для запрашивания данных всех пользователей в базе данных;
   * для запрашивания данных конкретного пользователя с применением фильтра по идентификатору пользователя;
   * для удаления таблицы.

   ```java
   /**
   * Select all rows from user table
   */
   public void selectAllUsers() {

     final String query = "SELECT * FROM uprofile.user";
     List<Row> rows = session.execute(query).all();

     for (Row row : rows) {
        LOGGER.info("Obtained row: {} | {} | {} ", row.getInt("user_id"), row.getString("user_name"), row.getString("user_bcity"));
     }
   }

   /**
   * Select a row from user table
   *
   * @param id user_id
   */
   public void selectUser(int id) {
      final String query = "SELECT * FROM uprofile.user where user_id = 3";
      Row row = session.execute(query).one();

      LOGGER.info("Obtained row: {} | {} | {} ", row.getInt("user_id"), row.getString("user_name"), row.getString("user_bcity"));
   }

   /**
   * Delete user table.
   */
   public void deleteTable() {
     final String query = "DROP TABLE IF EXISTS uprofile.user";
     session.execute(query);
   }
   ```

2. Откройте файл `UserProfile.java`, расположенный в папке `src\main\java\com\azure\cosmosdb\cassandra`. Этот класс содержит основной метод, который вызывает методы createKeyspace и createTable, а также метод вставки данных, определенные вами ранее. Теперь добавьте приведенный ниже код, который запрашивает всех пользователей или конкретного пользователя.

   ```java
   LOGGER.info("Select all users");
   repository.selectAllUsers();

   LOGGER.info("Select a user by id (3)");
   repository.selectUser(3);

   LOGGER.info("Delete the users profile table");
   repository.deleteTable();
   ```

## <a name="run-the-java-app"></a>Запуск приложения Java
1. Откройте командную строку или окно терминала. Вставьте следующий блок кода. 

   Этот код изменяет каталог (cd) для пути к папке, где создан проект. Затем он выполняет команду `mvn clean install`, чтобы создать файл `cosmosdb-cassandra-examples.jar` в целевой папке. Наконец, он запускает приложение Java.

   ```bash
   cd "cassandra-demo"
   
   mvn clean install
   
   java -cp target/cosmosdb-cassandra-examples.jar com.azure.cosmosdb.cassandra.examples.UserProfile
   ```

2. Теперь на портале Azure откройте **обозреватель данных** и убедитесь, что таблица пользователя удалена.

## <a name="clean-up-resources"></a>Очистка ресурсов

Можно удалить группу ресурсов, учетную запись Azure Cosmos и все связанные ресурсы, когда они больше не нужны. Для этого выберите группу ресурсов для виртуальной машины, выберите **Удалить** и подтвердите имя удаляемой группы ресурсов.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как в Azure Cosmos DB запросить данные из учетной записи API Cassandra. Теперь вы можете перейти к следующей статье:

> [!div class="nextstepaction"]
> [Перенос данных в учетную запись API Cassandra](cassandra-import-data.md)


