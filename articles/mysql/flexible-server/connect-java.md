---
title: Краткое руководство. Использование Java и JDBC с Гибким сервером Базы данных Azure для MySQL
description: Узнайте, как использовать Java и JDBC с Гибким сервером Базы данных Azure для MySQL.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.custom: mvc, devcenter, devx-track-azurecli
ms.topic: quickstart
ms.devlang: java
ms.date: 01/16/2021
ms.openlocfilehash: 93a8a7644c7ca729f0e4fd201b88e995576d4bda
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98606028"
---
# <a name="quickstart-use-java-and-jdbc-with-azure-database-for-mysql-flexible-server"></a>Краткое руководство. Использование Java и JDBC с Гибким сервером Базы данных Azure для MySQL

В этой статье показано, как создать пример приложения, которое использует Java и [JDBC](https://en.wikipedia.org/wiki/Java_Database_Connectivity) для сохранения данных на [Гибком сервере Базы данных Azure для MySQL](./index.yml) и их извлечения из нее.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure. Если у вас ее нет, [получите бесплатную пробную версию](https://azure.microsoft.com/free/).
- [Azure Cloud Shell](../../cloud-shell/quickstart.md) или [Azure CLI](/cli/azure/install-azure-cli) Мы рекомендуем использовать Azure Cloud Shell, так вы автоматически войдете в систему и получите доступ ко всем необходимым средствам.
- Поддерживаемый [пакет средств разработки Java (JDK)](/azure/developer/java/fundamentals/java-jdk-long-term-support) версии 8 (включена в Azure Cloud Shell).
- Средство сборки [Apache Maven](https://maven.apache.org/).

## <a name="prepare-the-working-environment"></a>Подготовка среды выполнения

Мы будем использовать переменные среды, чтобы минимизировать ошибки ввода и упростить дальнейшую настройку конфигурации в соответствии с требованиями.

Настройте следующие переменные среды с помощью следующих команд.

```bash
AZ_RESOURCE_GROUP=database-workshop
AZ_DATABASE_NAME= flexibleserverdb
AZ_LOCATION=<YOUR_AZURE_REGION>
AZ_MYSQL_USERNAME=demo
AZ_MYSQL_PASSWORD=<YOUR_MYSQL_PASSWORD>
AZ_LOCAL_IP_ADDRESS=<YOUR_LOCAL_IP_ADDRESS>
```

Замените заполнители следующими значениями, которые используются в этой статье:

- `<YOUR_DATABASE_NAME>`: имя сервера MySQL. Оно должно быть уникальным в Azure.
- `<YOUR_AZURE_REGION>`: регион Azure, который вы будете использовать. Вы можете использовать `eastus` по умолчанию, но мы рекомендуем настроить регион, расположенный близко к месту проживания. Полный список доступных регионов можно получить, введя `az account list-locations`.
- `<YOUR_MYSQL_PASSWORD>`: пароль вашего сервера базы данных MySQL. Такой пароль должен содержать не менее восьми символов и включать знаки всех следующих типов: прописные латинские буквы, строчные латинские буквы, цифры (0–9) и небуквенно-цифровые знаки (!, $, #, % и т. д.).
- `<YOUR_LOCAL_IP_ADDRESS>`: IP-адрес локального компьютера, с которого будет запускаться приложение Java. Одним из удобных способов его поиска является ввод в обозревателе адреса [whatismyip.akamai.com](http://whatismyip.akamai.com/).

Затем создайте группу ресурсов:

```azurecli
az group create \
    --name $AZ_RESOURCE_GROUP \
    --location $AZ_LOCATION \
  	| jq
```

> [!NOTE]
> Чтобы отобразить данные JSON и сделать их более удобными для чтения, мы используем служебную программу `jq`, которая по умолчанию устанавливается в [Azure Cloud Shell](https://shell.azure.com/).
> Если вам не нравится эта служебная программа, вы можете безопасно удалить `| jq` часть всех команд, которые мы будем использовать.

## <a name="create-an-azure-database-for-mysql-instance"></a>Создание экземпляра Базы данных Azure для MySQL

Первое, что мы создадим, — это управляемый сервер MySQL.

> [!NOTE]
> См. сведения о создании серверов MySQL в руководстве по [созданию базы данных Azure для сервера MySQL с помощью портала Azure](./quickstart-create-server-portal.md).

Выполните следующий скрипт в [Azure Cloud Shell](https://shell.azure.com/):

```azurecli
az mysql flexible-server create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name $AZ_DATABASE_NAME \
    --location $AZ_LOCATION \
    --sku-name Standard_B1ms \
    --storage-size 5120 \
    --admin-user $AZ_MYSQL_USERNAME \
    --admin-password $AZ_MYSQL_PASSWORD \
    --public-access $AZ_LOCAL_IP_ADDRESS
  	| jq
```

Чтобы получить доступ к серверу с локального компьютера, введите <ВАШ_IP-АДРЕС>. Эта команда позволяет создать уровень гибкого сервера MySQL с увеличивающейся производительностью, который подходит для разработки.

На созданном сервере MySQL есть пустая база данных с именем **flexibleserverdb**. Эту базу данных мы будем использовать для нашей статьи.

[Возникли проблемы? Сообщите нам!](https://github.com/MicrosoftDocs/azure-docs/issues)

### <a name="create-a-new-java-project"></a>Создание нового проекта Java

Используя любую интегрированную среду разработки, создайте проект Java и добавьте файл `pom.xml` в его корневой каталог.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>

    <properties>
        <java.version>1.8</java.version>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.20</version>
        </dependency>
    </dependencies>
</project>
```

Этот файл в формате [Apache Maven](https://maven.apache.org/) настраивает в проекте использование следующих компонентов:

- Java 8
- Последний драйвер MySQL для Java

### <a name="prepare-a-configuration-file-to-connect-to-azure-database-for-mysql"></a>Подготовка файла конфигурации для подключения к Базе данных Azure для MySQL

Создайте файл *src/main/resources/application.properties* и добавьте в него следующее содержимое.

```properties
url=jdbc:mysql://$AZ_DATABASE_NAME.mysql.database.azure.com:3306/demo?serverTimezone=UTC
user=demo
password=$AZ_MYSQL_PASSWORD
```

- Замените две переменные `$AZ_DATABASE_NAME` значением, которое вы настроили ранее.
- Замените переменную `$AZ_MYSQL_PASSWORD` значением, которое вы настроили ранее.

> [!NOTE]
> Мы добавляем `?serverTimezone=UTC` к свойству конфигурации `url`, чтобы драйвер JDBC при соединении с базой данных использовал UTC в качестве формата даты. В противном случае сервер Java и база данных будут использовать разные форматы даты, что неизбежно приведет к ошибке.

### <a name="create-an-sql-file-to-generate-the-database-schema"></a>Создание файла SQL для генерации схемы базы данных

Для создания схемы базы данных мы будем использовать файл *src/main/resources/`schema.sql`* . Создайте такой файл со следующим содержимым:

```sql
DROP TABLE IF EXISTS todo;
CREATE TABLE todo (id SERIAL PRIMARY KEY, description VARCHAR(255), details VARCHAR(4096), done BOOLEAN);
```

## <a name="code-the-application"></a>Добавление кода приложения

### <a name="connect-to-the-database"></a>Подключение к базе данных

Затем добавьте код Java, который будет использовать JDBC для сохранения и извлечения данных на сервере MySQL.

Создайте файл *src/main/java/DemoApplication.java* со следующим содержимым.

```java
package com.example.demo;

import com.mysql.cj.jdbc.AbandonedConnectionCleanupThread;

import java.sql.*;
import java.util.*;
import java.util.logging.Logger;

public class DemoApplication {

    private static final Logger log;

    static {
        System.setProperty("java.util.logging.SimpleFormatter.format", "[%4$-7s] %5$s %n");
        log =Logger.getLogger(DemoApplication.class.getName());
    }

    public static void main(String[] args) throws Exception {
        log.info("Loading application properties");
        Properties properties = new Properties();
        properties.load(DemoApplication.class.getClassLoader().getResourceAsStream("application.properties"));

        log.info("Connecting to the database");
        Connection connection = DriverManager.getConnection(properties.getProperty("url"), properties);
        log.info("Database connection test: " + connection.getCatalog());

        log.info("Create database schema");
        Scanner scanner = new Scanner(DemoApplication.class.getClassLoader().getResourceAsStream("schema.sql"));
        Statement statement = connection.createStatement();
        while (scanner.hasNextLine()) {
            statement.execute(scanner.nextLine());
        }

        /*
        Todo todo = new Todo(1L, "configuration", "congratulations, you have set up JDBC correctly!", true);
        insertData(todo, connection);
        todo = readData(connection);
        todo.setDetails("congratulations, you have updated data!");
        updateData(todo, connection);
        deleteData(todo, connection);
        */

        log.info("Closing database connection");
        connection.close();
        AbandonedConnectionCleanupThread.uncheckedShutdown();
    }
}
```

[Возникли проблемы? Сообщите нам!](https://github.com/MicrosoftDocs/azure-docs/issues)

Этот код Java, используя ранее созданные файлы *application.properties* и *schema.sql*, подключится к серверу MySQL и создаст схему для хранения данных.

Как видите, в этом файле мы закомментировали методы вставки, чтения, обновления и удаления данных. Эти методы мы создадим позже, и вы просто последовательно раскомментируете их.

> [!NOTE]
> Учетные данные для базы данных хранятся в свойствах *user* и *password* в файле *application.properties*. Эти учетные данные используются при выполнении `DriverManager.getConnection(properties.getProperty("url"), properties);`, так как файл свойств передается в качестве аргумента.

> [!NOTE]
> Строка `AbandonedConnectionCleanupThread.uncheckedShutdown();` в конце — это команда, связанная с драйвером MySQL. Она используется для уничтожения внутреннего потока при завершении работы приложения.
> Его можно игнорировать.

Теперь вы можете выполнить класс main в любом удобном инструменте.

- В любой среде IDE щелкните правой кнопкой мыши класс *DemoApplication* и выполните его.
- В Maven приложение можно запустить, выполнив команду `mvn exec:java -Dexec.mainClass="com.example.demo.DemoApplication"`.

Приложение должно подключиться к Базе данных Azure для MySQL, создать схему базы данных и закрыть подключение. При этом в журналах консоли вы увидите следующий результат.

```
[INFO   ] Loading application properties
[INFO   ] Connecting to the database
[INFO   ] Database connection test: demo
[INFO   ] Create database schema
[INFO   ] Closing database connection
```

### <a name="create-a-domain-class"></a>Создание доменного класса

Создайте новый класс Java `Todo` рядом с классом `DemoApplication` и добавьте следующий код:

```java
package com.example.demo;

public class Todo {

    private Long id;
    private String description;
    private String details;
    private boolean done;

    public Todo() {
    }

    public Todo(Long id, String description, String details, boolean done) {
        this.id = id;
        this.description = description;
        this.details = details;
        this.done = done;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getDetails() {
        return details;
    }

    public void setDetails(String details) {
        this.details = details;
    }

    public boolean isDone() {
        return done;
    }

    public void setDone(boolean done) {
        this.done = done;
    }

    @Override
    public String toString() {
        return "Todo{" +
                "id=" + id +
                ", description='" + description + '\'' +
                ", details='" + details + '\'' +
                ", done=" + done +
                '}';
    }
}
```

Этот класс является доменной моделью, сопоставленной с таблицей `todo`, которую вы создали при выполнении скрипта *schema.sql*.

### <a name="insert-data-into-azure-database-for-mysql"></a>Вставка данных в Базу данных Azure для MySQL

В файле *src/main/java/demoapplication.java* добавьте после метода main следующий метод, который вставляет данные в базу данных.

```java
private static void insertData(Todo todo, Connection connection) throws SQLException {
    log.info("Insert data");
    PreparedStatement insertStatement = connection
            .prepareStatement("INSERT INTO todo (id, description, details, done) VALUES (?, ?, ?, ?);");

    insertStatement.setLong(1, todo.getId());
    insertStatement.setString(2, todo.getDescription());
    insertStatement.setString(3, todo.getDetails());
    insertStatement.setBoolean(4, todo.isDone());
    insertStatement.executeUpdate();
}
```

Теперь можно раскомментировать две следующие строки в методе `main`.

```java
Todo todo = new Todo(1L, "configuration", "congratulations, you have set up JDBC correctly!", true);
insertData(todo, connection);
```

Теперь при запуске класса main вы увидите следующие выходные данные.

```
[INFO   ] Loading application properties
[INFO   ] Connecting to the database
[INFO   ] Database connection test: demo
[INFO   ] Create database schema
[INFO   ] Insert data
[INFO   ] Closing database connection
```

### <a name="reading-data-from-azure-database-for-mysql"></a>Считывание данных из Базы данных Azure для MySQL

Теперь мы считаем ранее вставленные данные, чтобы проверить работу нашего кода.

В файле *src/main/java/demoapplication.java* добавьте после метода `insertData` следующий метод, который считывает данные из базы данных.

```java
private static Todo readData(Connection connection) throws SQLException {
    log.info("Read data");
    PreparedStatement readStatement = connection.prepareStatement("SELECT * FROM todo;");
    ResultSet resultSet = readStatement.executeQuery();
    if (!resultSet.next()) {
        log.info("There is no data in the database!");
        return null;
    }
    Todo todo = new Todo();
    todo.setId(resultSet.getLong("id"));
    todo.setDescription(resultSet.getString("description"));
    todo.setDetails(resultSet.getString("details"));
    todo.setDone(resultSet.getBoolean("done"));
    log.info("Data read from the database: " + todo.toString());
    return todo;
}
```

Теперь можно раскомментировать следующую строку в методе `main`.

```java
todo = readData(connection);
```

Теперь при запуске класса main вы увидите следующие выходные данные.

```
[INFO   ] Loading application properties
[INFO   ] Connecting to the database
[INFO   ] Database connection test: demo
[INFO   ] Create database schema
[INFO   ] Insert data
[INFO   ] Read data
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have set up JDBC correctly!', done=true}
[INFO   ] Closing database connection
```

[Возникли проблемы? Сообщите нам!](https://github.com/MicrosoftDocs/azure-docs/issues)

### <a name="updating-data-in-azure-database-for-mysql"></a>Обновление данных в Базе данных Azure для MySQL

Давайте изменим ранее вставленные данные.

В том же файле *src/main/java/demoapplication.java* добавьте после метода `readData` следующий метод, который обновляет данные в базе данных.

```java
private static void updateData(Todo todo, Connection connection) throws SQLException {
    log.info("Update data");
    PreparedStatement updateStatement = connection
            .prepareStatement("UPDATE todo SET description = ?, details = ?, done = ? WHERE id = ?;");

    updateStatement.setString(1, todo.getDescription());
    updateStatement.setString(2, todo.getDetails());
    updateStatement.setBoolean(3, todo.isDone());
    updateStatement.setLong(4, todo.getId());
    updateStatement.executeUpdate();
    readData(connection);
}
```

Теперь можно раскомментировать две следующие строки в методе `main`.

```java
todo.setDetails("congratulations, you have updated data!");
updateData(todo, connection);
```

Теперь при запуске класса main вы увидите следующие выходные данные.

```
[INFO   ] Loading application properties
[INFO   ] Connecting to the database
[INFO   ] Database connection test: demo
[INFO   ] Create database schema
[INFO   ] Insert data
[INFO   ] Read data
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have set up JDBC correctly!', done=true}
[INFO   ] Update data
[INFO   ] Read data
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have updated data!', done=true}
[INFO   ] Closing database connection
```

### <a name="deleting-data-in-azure-database-for-mysql"></a>Удаление данных в Базе данных Azure для MySQL

Наконец, давайте удалим ранее вставленные данные.

В том же файле *src/main/java/demoapplication.java* добавьте после метода `updateData` следующий метод, который удаляет данные из базы данных.

```java
private static void deleteData(Todo todo, Connection connection) throws SQLException {
    log.info("Delete data");
    PreparedStatement deleteStatement = connection.prepareStatement("DELETE FROM todo WHERE id = ?;");
    deleteStatement.setLong(1, todo.getId());
    deleteStatement.executeUpdate();
    readData(connection);
}
```

Теперь можно раскомментировать следующую строку в методе `main`.

```java
deleteData(todo, connection);
```

Теперь при запуске класса main вы увидите следующие выходные данные.

```
[INFO   ] Loading application properties
[INFO   ] Connecting to the database
[INFO   ] Database connection test: demo
[INFO   ] Create database schema
[INFO   ] Insert data
[INFO   ] Read data
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have set up JDBC correctly!', done=true}
[INFO   ] Update data
[INFO   ] Read data
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have updated data!', done=true}
[INFO   ] Delete data
[INFO   ] Read data
[INFO   ] There is no data in the database!
[INFO   ] Closing database connection
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Поздравляем! Вы создали приложение Java, которое использует JDBC для сохранения данных в Базе данных Azure для MySQL и их извлечения из нее.

Чтобы очистить все ресурсы, используемые во время этого краткого руководства, удалите группу ресурсов с помощью следующей команды:

```azurecli
az group delete \
    --name $AZ_RESOURCE_GROUP \
    --yes
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Перенос базы данных MySQL в базу данных Azure для MySQL с помощью резервного копирования и восстановления](../concepts-migrate-dump-restore.md)
