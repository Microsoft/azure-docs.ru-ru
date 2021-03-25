---
title: Руководство по Создание приложения Java для создания учетной записи API Cassandra Azure Cosmos DB
description: В этом руководстве показано, как создать учетную запись API Cassandra, добавить базу данных (также называемую пространством ключей) и таблицу в эту учетную запись с помощью приложения Java.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: tutorial
ms.date: 12/06/2018
ms.custom: seodec18, devx-track-java
ms.openlocfilehash: ca1fdbd9aa2c98358489d91fe0839c98adec293b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102212711"
---
# <a name="tutorial-create-a-cassandra-api-account-in-azure-cosmos-db-by-using-a-java-application-to-store-keyvalue-data"></a>Руководство по Создание учетной записи API Cassandra в Azure Cosmos DB с помощью приложения Java для хранения данных пар "ключ — значение"
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Как у разработчика у вас должно быть приложение, использующее пары "ключ-значение". Можно использовать учетную запись API Cassandra в Azure Cosmos DB для хранения данных пар "ключ — значение". В этом руководстве описывается, как использовать приложение Java для создания учетной записи API Cassandra в Azure Cosmos DB, добавить базу данных (также называемую пространством ключей) и добавить таблицу. Приложение Java использует [драйвер Java](https://github.com/datastax/java-driver), чтобы создать базу данных пользователя, которая содержит такие сведения, как идентификатор пользователя, имя пользователя и город пользователя.  

В рамках этого руководства рассматриваются следующие задачи:

> [!div class="checklist"]
> * Создание учетной записи базы данных Cassandra
> * Получение строки подключения к учетной записи
> * Создание проекта Maven и зависимостей
> * Добавление базы данных и таблицы
> * Запустите приложение

## <a name="prerequisites"></a>Предварительные требования 

* Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio), прежде чем начинать работу. 

* Получите последнюю версию [пакета средств разработки Java (JDK)](/java/azure/jdk/). 

* [Скачайте](https://maven.apache.org/download.cgi) и [установите](https://maven.apache.org/install.html) двоичный архив [Maven](https://maven.apache.org/). 
  - В Ubuntu выполните команду `apt-get install maven`, чтобы установить Maven. 

## <a name="create-a-database-account"></a>Создание учетной записи базы данных 

1. Войдите на [портал Azure](https://portal.azure.com/). 

2. Последовательно выберите **Создать ресурс** > **Базы данных** > **Azure Cosmos DB**. 

3. На панели **Новая учетная запись** введите параметры для новой учетной записи Azure Cosmos. 

   |Параметр   |Рекомендуемое значение  |Описание  |
   |---------|---------|---------|
   |ID   |   Введите уникальное имя.    | Введите уникальное имя для идентификации этой учетной записи Azure Cosmos. <br/><br/>Так как элемент cassandra.cosmosdb.azure.com добавляется к указанному идентификатору для создания точки контакта, используйте уникальный, но узнаваемый идентификатор.         |
   |API    |  Cassandra   |  API определяет тип учетной записи, которую нужно создать. <br/> Выберите **Cassandra**, так как в этой статье вы создадите базу данных с широкими столбцами, к которой можно отправлять запросы с помощью синтаксиса языка запросов Cassandra (CQL).  |
   |Подписка    |  Ваша подписка        |  Выберите подписку Azure, которую планируете использовать для этой учетной записи Azure Cosmos.        |
   |Группа ресурсов   | Введите имя.    |  Выберите **Создать** и введите новое имя группы ресурсов для учетной записи. Для удобства можно использовать то же имя, которое присвоено идентификатору.    |
   |Расположение    |  Выберите ближайший к пользователям регион    |  Выберите географическое расположение, в котором будет размещена учетная запись Azure Cosmos. Используйте ближайшее к пользователям расположение, чтобы предоставить им максимально быстрый доступ к данным.    |

   :::image type="content" source="./media/create-cassandra-api-account-java/create-account.png" alt-text="Создание учетной записи на портале":::

4. Нажмите кнопку **создания**. <br/>Создание учетной записи займет несколько минут. После создания ресурса вы увидите уведомление **об успешном развертывании** в правой части портала.

## <a name="get-the-connection-details-of-your-account"></a>Получение сведений о подключении учетной записи  

Получите сведения о строке подключения с портала Azure и скопируйте их в файл конфигурации Java. Строка подключения обеспечивает обмен данными между вашим приложением и размещенной базой данных. 

1. С [портала Azure](https://portal.azure.com/) перейдите в учетную запись Azure Cosmos. 

2. Откройте панель **Строка подключения**.  

3. Скопируйте значения **CONTACT POINT**, **PORT**, **USERNAME** и **PRIMARY PASSWORD**, которые пригодятся вам на следующих этапах.

## <a name="create-the-project-and-the-dependencies"></a>Создание проекта и зависимостей 

Пример проекта Java, используемый в этой статье, размещен на GitHub. Можно выполнять действия, описанные в этом документе, или загрузить пример из репозитория [azure-cosmos-db-cassandra-java-getting-started](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started). 

После загрузки файлов обновите строку подключения в файле `java-examples\src\main\resources\config.properties` и запустите его.  

```java
cassandra_host=<FILLME_with_CONTACT POINT> 
cassandra_port = 10350 
cassandra_username=<FILLME_with_USERNAME> 
cassandra_password=<FILLME_with_PRIMARY PASSWORD> 
```

Чтобы создать образец с нуля, сделайте следующее: 

1. Из терминала или командной строки создайте новый проект Maven с именем Cassandra-demo. 

   ```bash
   mvn archetype:generate -DgroupId=com.azure.cosmosdb.cassandra -DartifactId=cassandra-demo -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false 
   ```
 
2. Найдите папку `cassandra-demo`. Откройте в текстовом редакторе созданный файл `pom.xml`. 

   Добавьте зависимости Cassandra и создайте подключаемые модули, необходимые в проекте, как показано в файле [pom.xml](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/main/pom.xml).  

3. В папке `cassandra-demo\src\main` создайте папку с именем `resources`.  В папке ресурсов добавьте файлы config.properties и log4j.properties:

   - Файл [config.properties](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/main/src/main/resources/config.properties) хранит значения конечной точки подключения и ключа учетной записи API Cassandra. 
   
   - Файл [log4j.properties](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/main/src/main/resources/log4j.properties) определяет уровень ведения журнала, необходимый для взаимодействия с API Cassandra.  

4. Перейдите в папку `src/main/java/com/azure/cosmosdb/cassandra/`. В папке cassandra создайте вторую папку с именем `utils`. Новая папка хранит служебные классы, необходимые для подключения к учетной записи API Cassandra. 

   Добавьте класс [CassandraUtils](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/main/src/main/java/com/azure/cosmosdb/cassandra/util/CassandraUtils.java) для создания кластера и открытия и закрытия сеансов Cassandra. Кластер подключается к учетной записи API Cassandra в Azure Cosmos DB и возвращает сеанс для доступа. Используйте класс [Configurations](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/main/src/main/java/com/azure/cosmosdb/cassandra/util/Configurations.java) для чтения строки подключения из файла config.properties. 

5. Пример Java создает базу данных со сведениями о пользователе, такими как имя пользователя, идентификатор пользователя и город пользователя. Необходимо определить методы get и set для доступа к сведениям о пользователе в основной функции.
 
   Создайте класс [User.java](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/main/src/main/java/com/azure/cosmosdb/cassandra/examples/UserProfile.java) в папке `src/main/java/com/azure/cosmosdb/cassandra/` с методами get и set. 

## <a name="add-a-database-and-a-table"></a>Добавление базы данных и таблицы  

В этом разделе описывается добавление базы данных (пространства ключей) и таблицы с помощью CQL.

1. В папке `src\main\java\com\azure\cosmosdb\cassandra` создайте папку с именем `repository`. 

2. Создайте класс Java `UserRepository` и добавьте в него следующий код: 

   ```java
   package com.azure.cosmosdb.cassandra.repository; 
   import java.util.List; 
   import com.datastax.driver.core.BoundStatement; 
   import com.datastax.driver.core.PreparedStatement; 
   import com.datastax.driver.core.Row; 
   import com.datastax.driver.core.Session; 
   import org.slf4j.Logger; 
   import org.slf4j.LoggerFactory; 
   
   /** 
    * Create a Cassandra session 
    */ 
   public class UserRepository { 
   
       private static final Logger LOGGER = LoggerFactory.getLogger(UserRepository.class); 
       private Session session; 
       public UserRepository(Session session) { 
           this.session = session; 
       } 
   
       /** 
       * Create keyspace uprofile in cassandra DB 
        */ 
   
       public void createKeyspace() { 
            final String query = "CREATE KEYSPACE IF NOT EXISTS uprofile WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 1 }"; 
           session.execute(query); 
           LOGGER.info("Created keyspace 'uprofile'"); 
       } 
   
       /** 
        * Create user table in cassandra DB 
        */ 
   
       public void createTable() { 
           final String query = "CREATE TABLE IF NOT EXISTS uprofile.user (user_id int PRIMARY KEY, user_name text, user_bcity text)"; 
           session.execute(query); 
           LOGGER.info("Created table 'user'"); 
       } 
   } 
   ```

3. Найдите папку `src\main\java\com\azure\cosmosdb\cassandra` и создайте новую вложенную папку с именем `examples`.

4. Создайте класс Java `UserProfile`. Этот класс содержит основной метод, который вызывает методы createKeyspace и createTable, определенные ранее: 

   ```java
   package com.azure.cosmosdb.cassandra.examples; 
   import java.io.IOException; 
   import com.azure.cosmosdb.cassandra.repository.UserRepository; 
   import com.azure.cosmosdb.cassandra.util.CassandraUtils; 
   import com.datastax.driver.core.PreparedStatement; 
   import com.datastax.driver.core.Session; 
   import org.slf4j.Logger; 
   import org.slf4j.LoggerFactory; 
   
   /** 
    * Example class which will demonstrate following operations on Cassandra Database on CosmosDB 
    * - Create Keyspace 
    * - Create Table 
    * - Insert Rows 
    * - Select all data from a table 
    * - Select a row from a table 
    */ 
   
   public class UserProfile { 
   
       private static final Logger LOGGER = LoggerFactory.getLogger(UserProfile.class); 
       public static void main(String[] s) throws Exception { 
           CassandraUtils utils = new CassandraUtils(); 
           Session cassandraSession = utils.getSession(); 
   
           try { 
               UserRepository repository = new UserRepository(cassandraSession); 
               //Create keyspace in cassandra database 
               repository.createKeyspace(); 
               //Create table in cassandra database 
               repository.createTable(); 
   
           } finally { 
               utils.close(); 
               LOGGER.info("Please delete your table after verifying the presence of the data in portal or from CQL"); 
           } 
       } 
   } 
   ```
 
## <a name="run-the-app"></a>Запустите приложение 

1. Откройте командную строку или окно терминала. Вставьте следующий блок кода. 

   Этот код изменяет каталог (cd) для пути к папке, где создан проект. Затем он выполняет команду `mvn clean install`, чтобы создать файл `cosmosdb-cassandra-examples.jar` в целевой папке. Наконец, он запускает приложение Java.

   ```bash
   cd cassandra-demo

   mvn clean install 

   java -cp target/cosmosdb-cassandra-examples.jar com.azure.cosmosdb.cassandra.examples.UserProfile 
   ```

   Окно терминала отображает уведомления о создании пространства ключей и таблицы. 
   
2. Теперь на портале Azure откройте **обозреватель данных** и убедитесь, что пространство ключей и таблица созданы.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как создать учетную запись API Cassandra в Azure Cosmos DB, базу данных и таблицу с помощью приложения Java. Теперь вы можете перейти к следующей статье:

> [!div class="nextstepaction"]
> [Загрузка образца данных в таблицу API Cassandra](cassandra-api-load-data.md).
