---
title: Создание приложения Go, использующего API Cassandra в Azure Cosmos DB, с помощью клиента gocql
description: В этом кратком руководстве показано, как использовать клиент Go для взаимодействия с API Cassandra в Azure Cosmos DB
ms.service: cosmos-db
author: abhirockzz
ms.author: abhishgu
ms.subservice: cosmosdb-cassandra
ms.devlang: go
ms.topic: quickstart
ms.date: 07/14/2020
ms.openlocfilehash: 595ec1aaa4aedc3916d1b4d46986dcabae887aaf
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93076407"
---
# <a name="quickstart-build-a-go-app-with-the-gocql-client-to-manage-azure-cosmos-db-cassandra-api-data"></a>Краткое руководство. Создание приложения Go с помощью клиента `gocql` для управления данными в Azure Cosmos DB через API Cassandra
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-cassandra-dotnet.md)
> * [.NET Core](create-cassandra-dotnet-core.md)
> * [Java, версия 3](create-cassandra-java.md)
> * [Java, версия 4](create-cassandra-java-v4.md)
> * [Node.js](create-cassandra-nodejs.md)
> * [Python](create-cassandra-python.md)
> * [Golang](create-cassandra-go.md)
>

Azure Cosmos DB — это служба многомодельной базы данных, позволяющая быстро создавать и запрашивать документы, таблицы, пары "ключ-значение" и графовые базы данных, используя возможности глобального распределения и горизонтального масштабирования. В начале этого краткого руководства вы создадите учетную запись для API Cassandra в Azure Cosmos DB. Затем вы запустите приложение Go для создания пространства ключей Cassandra, создания таблицы и выполнения нескольких операций. В этом приложении Go используется клиент Cassandra [gocql](https://github.com/gocql/gocql) для языка Go. 

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/?WT.mc_id=cassandrago-docs-abhishgu). Или [воспользуйтесь пробной версией Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/?WT.mc_id=cassandrago-docs-abhishgu) без подписки Azure.
- На компьютере должна быть установлена среда [Go](https://golang.org/), а у вас должен быть опыт работы с этим языком.
- [Git](https://git-scm.com/downloads).

## <a name="create-a-database-account"></a>Создание учетной записи базы данных

Прежде чем создавать базу данных, необходимо создать в Azure Cosmos DB учетную запись Cassandra.

[!INCLUDE [cosmos-db-create-dbaccount-cassandra](../../includes/cosmos-db-create-dbaccount-cassandra.md)]

## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Прежде всего, клонируйте приложение из репозитория GitHub.

1. Откройте командную строку и создайте новую папку с именем `git-samples`.

    ```bash
    md "C:\git-samples"
    ```

2. Откройте окно терминала Git, например Git Bash. С помощью команды `cd` перейдите в новую папку и установите пример приложения.

    ```bash
    cd "C:\git-samples"
    ```

3. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере.

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-cassandra-go-getting-started.git
    ```

## <a name="review-the-code"></a>Просмотр кода

Это необязательный шаг. Если вы хотите узнать, как создавать ресурсы базы данных в коде, изучите приведенные ниже фрагменты кода. В противном случае вы можете сразу перейти к разделу [Выполнение приложения](#run-the-application).

Функция `GetSession` (в файле `utils\utils.go`) возвращает структуру [`*gocql.Session`](https://godoc.org/github.com/gocql/gocql#Session), которая используется для выполнения в кластере операций вставки, поиска и т. д.

```go
func GetSession(cosmosCassandraContactPoint, cosmosCassandraPort, cosmosCassandraUser, cosmosCassandraPassword string) *gocql.Session {
    clusterConfig := gocql.NewCluster(cosmosCassandraContactPoint)
    port, err := strconv.Atoi(cosmosCassandraPort)
    
    clusterConfig.Authenticator = gocql.PasswordAuthenticator{Username: cosmosCassandraUser, Password: cosmosCassandraPassword}
    clusterConfig.Port = port
    clusterConfig.SslOpts = &gocql.SslOptions{Config: &tls.Config{MinVersion: tls.VersionTLS12}}
    clusterConfig.ProtoVersion = 4
    
    session, err := clusterConfig.CreateSession()
    ...
    return session
}
```

Узел Cassandra в Azure Cosmos DB передается функции [`gocql.NewCluster`](https://godoc.org/github.com/gocql/gocql#NewCluster), которая возвращает структуру [`*gocql.ClusterConfig`](https://godoc.org/github.com/gocql/gocql#ClusterConfig), для которой мы затем настраиваем имя пользователя, пароль, порт и версию TLS ([требование безопасности для шифрования HTTPS/SSL/TLS](./database-security.md?WT.mc_id=cassandrago-docs-abhishgu#how-does-azure-cosmos-db-secure-my-database)).

Затем из функции `main` (`main.go`) вызывается функция `GetSession`.

```go
func main() {
    session := utils.GetSession(cosmosCassandraContactPoint, cosmosCassandraPort, cosmosCassandraUser, cosmosCassandraPassword)
    defer session.Close()
    ...
}
```

Сведения о подключении и учетные данные принимаются в виде переменных среды (значения разрешаются в методе `init`).

```go
func init() {
    cosmosCassandraContactPoint = os.Getenv("COSMOSDB_CASSANDRA_CONTACT_POINT")
    cosmosCassandraPort = os.Getenv("COSMOSDB_CASSANDRA_PORT")
    cosmosCassandraUser = os.Getenv("COSMOSDB_CASSANDRA_USER")
    cosmosCassandraPassword = os.Getenv("COSMOSDB_CASSANDRA_PASSWORD")

    if cosmosCassandraContactPoint == "" || cosmosCassandraUser == "" || cosmosCassandraPassword == "" {
        log.Fatal("missing mandatory environment variables")
    }
}
```

Эти данные применяются для выполнения операций в Azure Cosmos DB (в файле `operations\setup.go`), начиная с создания `keyspace` и `table`.

Как видно из названия, функция `DropKeySpaceIfExists` удаляет `keyspace` только в том случае, если пространство ключей существует.

```go
const dropKeyspace = "DROP KEYSPACE IF EXISTS %s"

func DropKeySpaceIfExists(keyspace string, session *gocql.Session) {
    err := utils.ExecuteQuery(fmt.Sprintf(dropKeyspace, keyspace), session)
    if err != nil {
        log.Fatal("Failed to drop keyspace", err)
    }
    log.Println("Keyspace dropped")
}
```

Функция `CreateKeySpace` применяется для создания `keyspace` (`user_profile`).

```go
const createKeyspace = "CREATE KEYSPACE %s WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 1 }"

func CreateKeySpace(keyspace string, session *gocql.Session) {
    err := utils.ExecuteQuery(fmt.Sprintf(createKeyspace, keyspace), session)
    if err != nil {
        log.Fatal("Failed to create keyspace", err)
    }
    log.Println("Keyspace created")
}
```

За этим следует создание таблиц (`user`), которым занимается функция `CreateUserTable`.

```go
const createTable = "CREATE TABLE %s.%s (user_id int PRIMARY KEY, user_name text, user_bcity text)"

func CreateUserTable(keyspace, table string, session *gocql.Session) {
    err := session.Query(fmt.Sprintf(createTable, keyspace, table)).Exec()
    if err != nil {
        log.Fatal("failed to create table ", err)
    }
    log.Println("Table created")
}
```

После создания пространства ключей и таблицы мы вызываем операции CRUD (в файле `operations\crud.go`). 

Для создания `User` используется `InsertUser`. Она передает сведения о пользователе (идентификатор, имя и город) в качестве аргументов запроса, используя [`Bind`](https://godoc.org/github.com/gocql/gocql#Query.Bind).

```go
const createQuery = "INSERT INTO %s.%s (user_id, user_name , user_bcity) VALUES (?,?,?)"

func InsertUser(keyspace, table string, session *gocql.Session, user model.User) {
    err := session.Query(fmt.Sprintf(createQuery, keyspace, table)).Bind(user.ID, user.Name, user.City).Exec()
    if err != nil {
        log.Fatal("Failed to create user", err)
    }
    log.Println("User created")
}
```

Выполняется `FindUser` для поиска пользователя (`model\user.go`) по идентификатору пользователя, а затем [`Scan`](https://godoc.org/github.com/gocql/gocql#Iter.Scan) привязывает атрибуты пользователя (полученные от Cassandra) к отдельным переменным (`userid`, `name` и `city`). Это лишь один из возможных способов использовать полученный от запроса поиска результат.

```go
const selectQuery = "SELECT * FROM %s.%s where user_id = ?"

func FindUser(keyspace, table string, id int, session *gocql.Session) model.User {
    var userid int
    var name, city string
    err := session.Query(fmt.Sprintf(selectQuery, keyspace, table)).Bind(id).Scan(&userid, &name, &city)

    if err != nil {
        if err == gocql.ErrNotFound {
            log.Printf("User with id %v does not exist\n", id)
        } else {
            log.Printf("Failed to find user with id %v - %v\n", id, err)
        }
    }
    return model.User{ID: userid, Name: name, City: city}
}
```

`FindAllUsers` применяется для получения всех пользователей. [`SliceMap`](https://godoc.org/github.com/gocql/gocql#Iter.SliceMap) является краткой формой записи для получения сведений о пользователе в виде среза структуры `map`. Каждую структуру `map` можно рассматривать как пару "ключ — значение", где ключом является имя столбца (например, `user_id`), а значение извлекается из соответствующего столбца.

```go
const findAllUsersQuery = "SELECT * FROM %s.%s"

func FindAllUsers(keyspace, table string, session *gocql.Session) []model.User {
    var users []model.User
    results, _ := session.Query(fmt.Sprintf(findAllUsersQuery, keyspace, table)).Iter().SliceMap()

    for _, u := range results {
        users = append(users, mapToUser(u))
    }
    return users
}
```

Каждая `map` из набора сведений о пользователе преобразуется в `User` с помощью функции `mapToUser`, которая просто извлекает значение из соответствующего столбца и создает на его основе экземпляр структуры `User`.

```go
func mapToUser(m map[string]interface{}) model.User {
    id, _ := m["user_id"].(int)
    name, _ := m["user_name"].(string)
    city, _ := m["user_bcity"].(string)

    return model.User{ID: id, Name: name, City: city}
}
```

## <a name="run-the-application"></a>Выполнение приложения

Как упоминалось ранее, приложение принимает сведения о подключении и учетные данные в виде переменных среды. 

1. Выберите элемент **Строка подключения** в своей учетной записи Azure Cosmos DB на [портале Azure](https://portal.azure.com/). 

    :::image type="content" source="./media/create-cassandra-go/copy-username-connection-string-azure-portal.png" alt-text="Просмотр и копирование сведений со страницы Строка подключения на портале Azure":::

Скопируйте значения следующих атрибутов (`CONTACT POINT`, `PORT`, `USERNAME` и `PRIMARY PASSWORD`) и сохраните их в соответствующие переменные среды.

```shell
set COSMOSDB_CASSANDRA_CONTACT_POINT=<value for "CONTACT POINT">
set COSMOSDB_CASSANDRA_PORT=<value for "PORT">
set COSMOSDB_CASSANDRA_USER=<value for "USERNAME">
set COSMOSDB_CASSANDRA_PASSWORD=<value for "PRIMARY PASSWORD">
```

В окне терминала перейдите в нужную папку. Пример:

```shell
cd "C:\git-samples\azure-cosmosdb-cassandra-go-getting-started"
```

2. В сеансе терминала выполните следующую команду, чтобы запустить приложение.

```shell
go run main.go
```

3. В окне терминала отображаются уведомления об операциях, включая настройку пространства ключей и таблицы, создание пользователя и т. д.

4. На портале Azure откройте **обозреватель данных**, чтобы запросить, изменить и обработать новые данные. 

    :::image type="content" source="./media/create-cassandra-go/view-data-explorer-go-app.png" alt-text="Просмотр данных в обозревателе данных — Azure Cosmos DB":::

## <a name="review-slas-in-the-azure-portal"></a>Просмотр соглашений об уровне обслуживания на портале Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать учетную запись Azure Cosmos DB с помощью API Cassandra и запустить приложение Go, которое создает базу данных и контейнер Cassandra. Теперь вы можете импортировать дополнительные данные в учетную запись Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Импорт данных Cassandra в Azure Cosmos DB](cassandra-import-data.md)