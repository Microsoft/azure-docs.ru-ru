---
title: Создание приложения с помощью API Cassandra для Azure Cosmos DB и пакета SDK Java версии 4.0
description: В этом руководстве показано, как использовать API Cassandra Azure Cosmos DB для создания приложения профиля с помощью портала Azure и пакета SDK Java версии 4.0.
ms.service: cosmos-db
author: TheovanKraay
ms.author: thvankra
ms.subservice: cosmosdb-cassandra
ms.devlang: java
ms.topic: quickstart
ms.date: 05/18/2020
ms.custom: seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: ee30af3f07c8b350393822f01833feec6e3b52e2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93099816"
---
# <a name="quickstart-build-a-java-app-to-manage-azure-cosmos-db-cassandra-api-data-v4-driver"></a>Создание приложения Java для управления данными API Cassandra в Azure Cosmos DB (драйвер версии 4)
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-cassandra-dotnet.md)
> * [.NET Core](create-cassandra-dotnet-core.md)
> * [Java, версия 3](create-cassandra-java.md)
> * [Java, версия 4](create-cassandra-java-v4.md)
> * [Node.js](create-cassandra-nodejs.md)
> * [Python](create-cassandra-python.md)
> 

В этом кратком руководстве рассказывается, как создать учетную запись API Cassandra Azure Cosmos DB и использовать приложение Java Cassandra, клонированное из GitHub, чтобы создать базу данных и контейнер Cassandra с помощью [драйверов Apache Cassandra версии 4.x](https://github.com/datastax/java-driver/tree/4.x) для Java. Azure Cosmos DB — это служба многомодельной базы данных, позволяющая быстро создавать и запрашивать документы, таблицы, пары "ключ-значение" и графовые базы данных, используя возможности глобального распределения и горизонтального масштабирования.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Или [бесплатно поработайте с Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) без подписки Azure.
- [Комплект SDK для Java (JDK) версии 8](https://www.azul.com/downloads/azure-only/zulu/?&version=java-8-lts&architecture=x86-64-bit&package=jdk). Укажите для переменной среды `JAVA_HOME` папку, в которой установлен комплект JDK.
- [Двоичный архив Maven](https://maven.apache.org/download.cgi). В Ubuntu выполните команду `apt-get install maven`, чтобы установить Maven.
- [Git](https://www.git-scm.com/downloads). В Ubuntu выполните команду `sudo apt-get install git`, чтобы установить Git.

## <a name="create-a-database-account"></a>Создание учетной записи базы данных

Прежде чем создавать базу данных документов, необходимо создать в Azure Cosmos DB учетную запись Cassandra.

[!INCLUDE [cosmos-db-create-dbaccount-cassandra](../../includes/cosmos-db-create-dbaccount-cassandra.md)]

## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Теперь перейдем к работе с кодом. Клонируйте приложение API Cassandra с GitHub, укажите строку подключения и запустите это приложение. Вы узнаете, как можно упростить работу с данными программным способом. 

1. Откройте командную строку. Создайте папку с именем `git-samples`. Затем закройте командную строку.

    ```bash
    md "C:\git-samples"
    ```

2. Откройте окно терминала git, например git bash, и выполните команду `cd`, чтобы перейти в новую папку для установки примера приложения.

    ```bash
    cd "C:\git-samples"
    ```

3. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере.

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started-v4.git
    ```

## <a name="review-the-code"></a>Просмотр кода

Это необязательный шаг. Если вы хотите узнать, как создавать ресурсы базы данных в коде, изучите приведенные ниже фрагменты кода. Если вас это не интересует, можете сразу переходить к разделу [Обновление строки подключения](#update-your-connection-string). Эти фрагменты кода взяты из файла *src/main/java/com/azure/cosmosdb/cassandra/util/CassandraUtils.java*.  

* `CqlSession` подключается к API Cassandra для Azure Cosmos DB и возвращает сеанс для доступа (объект `Cluster` из драйвера версии 3 уже устарел). Имя пользователя, пароль, порт и узел Cassandra задаются с помощью страницы строки подключения на портале Azure.

    ```java
        this.session = CqlSession.builder().withSslContext(sc)
                .addContactPoint(new InetSocketAddress(cassandraHost, cassandraPort)).withLocalDatacenter(region)
                .withAuthCredentials(cassandraUsername, cassandraPassword).build();
    ```


Следующие фрагменты кода взяты из файла *src/main/java/com/azure/cosmosdb/cassandra/repository/UserRepository.java*.

* Если при предыдущем запуске было создано ключевое пространство, его нужно удалить.

    ```java
    public void dropKeyspace() {
        String query = "DROP KEYSPACE IF EXISTS "+keyspace+"";
        session.execute(query);
        LOGGER.info("dropped keyspace '"+keyspace+"'");
    } 
    ```
* Создается пространство ключей.

    ```java
    public void createKeyspace() {
        String query = "CREATE KEYSPACE "+keyspace+" WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 1 }";
        session.execute(query);
        LOGGER.info("Created keyspace '"+keyspace+"'");
    }
    ```

* Создается таблица.

   ```java
    public void createTable() {
        String query = "CREATE TABLE "+keyspace+"."+table+" (user_id int PRIMARY KEY, user_name text, user_bcity text)";
        session.execute(query);
        LOGGER.info("Created table '"+table+"'");
    }
   ```

* Пользовательские сущности вставляются с использованием подготовленного объекта инструкций.

    ```java
    public String prepareInsertStatement() {
        final String insertStatement = "INSERT INTO  "+keyspace+"."+table+" (user_id, user_name , user_bcity) VALUES (?,?,?)";
        return insertStatement;
    }

    public void insertUser(String preparedStatement, int id, String name, String city) {
        PreparedStatement prepared = session.prepare(preparedStatement);
        BoundStatement bound = prepared.bind(id, city, name).setIdempotent(true);
        session.execute(bound);
    }
    ```

* Выполняется запрос на получение информации обо всех пользователях.

    ```java
    public void selectAllUsers() {
        final String query = "SELECT * FROM "+keyspace+"."+table+"";
        List<Row> rows = session.execute(query).all();

        for (Row row : rows) {
            LOGGER.info("Obtained row: {} | {} | {} ", row.getInt("user_id"), row.getString("user_name"), row.getString("user_bcity"));
        }
    }
    ```

 * Выполняется запрос на получение информации об одном пользователе.

    ```java
    public void selectUser(int id) {
        final String query = "SELECT * FROM "+keyspace+"."+table+" where user_id = 3";
        Row row = session.execute(query).one();

        LOGGER.info("Obtained row: {} | {} | {} ", row.getInt("user_id"), row.getString("user_name"), row.getString("user_bcity"));
    }
    ```

## <a name="update-your-connection-string"></a>Обновление строки подключения

Теперь вернитесь на портал Azure, чтобы получить данные строки подключения. Скопируйте эти данные в приложение. Данные строки подключения обеспечивают обмен данными между вашим приложением и размещенной базой данных.

1. Выберите элемент **Строка подключения** в своей учетной записи Azure Cosmos DB на [портале Azure](https://portal.azure.com/). 

    :::image type="content" source="./media/create-cassandra-java/copy-username-connection-string-azure-portal.png" alt-text="Просмотр и копирование имени пользователя на странице &quot;Строка подключения&quot; на портале Azure":::

2. Используйте кнопку :::image type="icon" source="./media/create-cassandra-java/copy-button-azure-portal.png"::: в правой части экрана, чтобы скопировать значение параметра CONTACT POINT.

3. Откройте файл *config.properties* из папки *C:\git-samples\azure-cosmosdb-cassandra-java-getting-started\java-examples\src\main\resources*. 

3. Вставьте полученное на портале значение параметра CONTACT POINT вместо элемента `<Cassandra endpoint host>` в строке 2.

    Теперь строка 2 в файле *config.properties* будет выглядеть примерно так: 

    `cassandra_host=cosmos-db-quickstart.cassandra.cosmosdb.azure.com`

3. Вернитесь на портал и скопируйте значение USERNAME. Вставьте полученное на портале значение USERNAME вместо элемента `<cassandra endpoint username>` в строке 4.

    Теперь строка 4 в файле *config.properties* будет выглядеть примерно так: 

    `cassandra_username=cosmos-db-quickstart`

4. Вернитесь на портал и скопируйте значение PASSWORD. Вставьте полученное на портале значение PASSWORD вместо элемента `<cassandra endpoint password>` в строке 5.

    Теперь строка 5 в файле *config.properties* будет выглядеть примерно так: 

    `cassandra_password=2Ggkr662ifxz2Mg...==`

5. Если вы хотите использовать определенный TLS/SSL-сертификат, замените элемент `<SSL key store file location>` в строке 6 расположением своего TLS/SSL-сертификата. Если не указать это значение, будет использоваться сертификат JDK, установленный в папке <JAVA_HOME>/jre/lib/security/cacerts. 

6. Если вы изменили строку 6, чтобы использовать свой TLS/SSL-сертификат, укажите пароль для этого сертификата в строке 7. 

7. Обратите внимание, что для точки контакта требуется указать регион по умолчанию (например, `West US`):

    `region=West US`

    Это нужно, потому что драйвер версии 4.x не поддерживает связь нескольких локальных контроллеров домена с точкой контакта. Чтобы указать другой регион, помимо заданного по умолчанию (т. е. при создании учетной записи Cosmos DB), при добавлении точки контакта необходимо использовать региональный суффикс, например `host-westus.cassandra.cosmos.azure.com`.

8. Сохраните файл *config.properties*.

## <a name="run-the-java-app"></a>Запуск приложения Java

1. В окне терминала git с помощью команды `cd` перейдите к папке `azure-cosmosdb-cassandra-java-getting-started-v4`.

    ```git
    cd "C:\git-samples\azure-cosmosdb-cassandra-java-getting-started-v4"
    ```

2. В окне терминала Git используйте приведенную ниже команду, чтобы создать файл `cosmosdb-cassandra-examples.jar`.

    ```git
    mvn clean install
    ```

3. В окне терминала Git выполните следующие команды, чтобы запустить приложение Java.

    ```git
    java -cp target/cosmosdb-cassandra-examples.jar com.azure.cosmosdb.cassandra.examples.UserProfile
    ```

    Окно терминала отображает уведомления о создании пространства ключей и таблицы. Затем оно выбирает и отображает список всех пользователей в таблице, а затем выбирает строку по идентификатору и отображает значение.  

    Нажмите клавиши CTRL+C, чтобы остановить выполнение программы и закрыть окно консоли.

4. На портале Azure откройте **обозреватель данных**, чтобы запросить, изменить и обработать новые данные. 

    :::image type="content" source="./media/create-cassandra-java/view-data-explorer-java-app.png" alt-text="Просмотр данных в обозревателе данных — Azure Cosmos DB":::

## <a name="review-slas-in-the-azure-portal"></a>Просмотр соглашений об уровне обслуживания на портале Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать учетную запись Azure Cosmos DB с помощью API Cassandra и запустить приложение Cassandra Java, которое создает базу данных и контейнер Cassandra. Теперь вы можете импортировать дополнительные данные в учетную запись Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Импорт данных Cassandra в Azure Cosmos DB](cassandra-import-data.md)
