---
title: Краткое руководство. Подключение к службе "База данных Azure для PostgreSQL — отдельный сервер" с помощью Rust
description: В этом кратком руководстве представлены примеры кода Rust, которые можно использовать для подключения к службе "База данных Azure для PostgreSQL — отдельный сервер" и запроса данных из нее.
author: abhirockzz
ms.author: abhishgu
ms.service: postgresql
ms.devlang: rust
ms.topic: quickstart
ms.date: 03/26/2021
ms.openlocfilehash: 00adc50ac14e627eb356a3e62ce0faa5a9716aa3
ms.sourcegitcommit: c2a41648315a95aa6340e67e600a52801af69ec7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106505561"
---
# <a name="quickstart-use-rust-to-connect-and-query-data-in-azure-database-for-postgresql---single-server"></a>Краткое руководство. Подключение к службе "База данных Azure для PostgreSQL — отдельный сервер" и выполнение запроса данных с помощью Rust

Вы узнаете, как применять [драйвер PostgreSQL для Rust](https://github.com/sfackler/rust-postgres) для взаимодействия с БД Azure для PostgreSQL, изучая код операций CRUD (создание, чтение, обновление, удаление), реализованный в нашем примере. Наконец, вы сможете запустить приложение локально, чтобы проверить его в действии.

## <a name="prerequisites"></a>Предварительные требования
Для целей этого краткого руководства понадобится:

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free) бесплатно.
- Установлена последняя версия [Rust](https://www.rust-lang.org/tools/install).
- Отдельный сервер Базы данных Azure для PostgreSQL, созданный на [портале Azure](./quickstart-create-server-database-portal.md) <br/> или с помощью [Azure CLI](./quickstart-create-server-database-azure-cli.md).
- Выполнить **ОДНО** из действий (в зависимости от того, пользуетесь вы общим или частным доступом), чтобы настроить возможность подключения.

  |Действие| Метод подключения|Практическое руководство|
  |:--------- |:--------- |:--------- |
  | **Настройка правил брандмауэра** | Общие | [Портал](./howto-manage-firewall-using-portal.md) <br/> [CLI](./howto-manage-firewall-using-cli.md)|
  | **Настройка конечной точки службы** | Общие | [Портал](./howto-manage-vnet-using-portal.md) <br/> [CLI](./howto-manage-vnet-using-cli.md)|
  | **Настройка приватного канала** | Private | [Портал](./howto-configure-privatelink-portal.md) <br/> [CLI](./howto-configure-privatelink-cli.md) |

- [GIT](https://git-scm.com/downloads) установлен.

## <a name="get-database-connection-information"></a>Получение сведений о подключении к базе данных
Чтобы подключиться к базе данных службы "База данных Azure для PostgreSQL", требуется полное имя сервера и учетные данные для входа. Эти сведения можно получить на портале Azure.

1. На [портале Azure](https://portal.azure.com/) выполните поиск по имени сервера и выберите сервер службы "База данных Azure для PostgreSQL".
1. На странице **Обзор** сервера скопируйте **полное имя сервера** и **имя администратора**. Полное **имя сервера** всегда имеет формат *\<my-server-name>.postgres.database.azure.com*, а **имя администратора** — формат *\<my-admin-username>@\<my-server-name>* .

## <a name="review-the-code-optional"></a>Просмотр кода (необязательно)

Если вы хотите узнать, как работает код, изучите приведенные ниже фрагменты. Если нет, вы можете сразу перейти к разделу [Выполнение приложения](#run-the-application).

### <a name="connect"></a>Подключение

Функция `main` начинает работу с подключения к базе данных Azure для PostgreSQL и зависит от следующих переменных среды для получения сведений о подключении: `POSTGRES_HOST`, `POSTGRES_USER`, `POSTGRES_PASSWORD` и `POSTGRES_DBNAME`. По умолчанию в службе базы данных PostgreSQL настроено обязательное использование соединения `TLS`. Вы можете отключить необходимое использование `TLS` в том случае, если клиентское приложение не поддерживает подключения типа `TLS`. Дополнительные сведения см. в статье [Настройка подключения TLS в базе данных Azure для PostgreSQL — отдельный сервер](./concepts-ssl-connection-security.md).

В примере приложения в этой статье используется протокол TLS с [ящиком postgres-OpenSSL](https://crates.io/crates/postgres-openssl/). Функция [postgres:: Client:: Connect](https://docs.rs/postgres/0.19.0/postgres/struct.Client.html#method.connect) используется для запуска подключения, и программа завершает работу, если это не удается.

```rust
fn main() {
    let pg_host = std::env::var("POSTGRES_HOST").expect("missing environment variable POSTGRES_HOST");
    let pg_user = std::env::var("POSTGRES_USER").expect("missing environment variable POSTGRES_USER");
    let pg_password = std::env::var("POSTGRES_PASSWORD").expect("missing environment variable POSTGRES_PASSWORD");
    let pg_dbname = std::env::var("POSTGRES_DBNAME").unwrap_or("postgres".to_string());

    let builder = SslConnector::builder(SslMethod::tls()).unwrap();
    let tls_connector = MakeTlsConnector::new(builder.build());

    let url = format!(
        "host={} port=5432 user={} password={} dbname={} sslmode=require",
        pg_host, pg_user, pg_password, pg_dbname
    );
    let mut pg_client = postgres::Client::connect(&url, tls_connector).expect("failed to connect to postgres");
...
}
```

### <a name="drop-and-create-table"></a>Сброс и создание таблицы

Пример приложения использует простую таблицу `inventory` для демонстрации операций CRUD (создание, чтение, обновление, удаление).

```sql
CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);
```

Функция `drop_create_table` изначально пытается выполнить `DROP` таблицы `inventory` перед созданием новой. Это упрощает обучение и экспериментирование, так как всегда начинается с известного (чистого) состояния. Метод [EXECUTE](https://docs.rs/postgres/0.19.0/postgres/struct.Client.html#method.execute) используется для операций создания и удаления. 

```rust
const CREATE_QUERY: &str =
    "CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);";

const DROP_TABLE: &str = "DROP TABLE inventory";

fn drop_create_table(pg_client: &mut postgres::Client) {
    let res = pg_client.execute(DROP_TABLE, &[]);
    match res {
        Ok(_) => println!("dropped table"),
        Err(e) => println!("failed to drop table {}", e),
    }
    pg_client
        .execute(CREATE_QUERY, &[])
        .expect("failed to create 'inventory' table");
}
```

### <a name="insert-data"></a>Добавление данных

`insert_data` добавляет записи в таблицу `inventory`. Он создает [подготовленную инструкцию](https://docs.rs/postgres/0.19.0/postgres/struct.Statement.html) с функцией [Prepare](https://docs.rs/postgres/0.19.0/postgres/struct.Client.html#method.prepare) . 


```rust
const INSERT_QUERY: &str = "INSERT INTO inventory (name, quantity) VALUES ($1, $2) RETURNING id;";

fn insert_data(pg_client: &mut postgres::Client) {

    let prep_stmt = pg_client
        .prepare(&INSERT_QUERY)
        .expect("failed to create prepared statement");

    let row = pg_client
        .query_one(&prep_stmt, &[&"item-1", &42])
        .expect("insert failed");

    let id: i32 = row.get(0);
    println!("inserted item with id {}", id);
...
}
```

Также обратите внимание на использование метода [prepare_typed](https://docs.rs/postgres/0.19.0/postgres/struct.Client.html#method.prepare_typed), который позволяет явно указывать типы параметров запроса.

```rust
...
let typed_prep_stmt = pg_client
        .prepare_typed(&INSERT_QUERY, &[Type::VARCHAR, Type::INT4])
        .expect("failed to create prepared statement");

let row = pg_client
        .query_one(&typed_prep_stmt, &[&"item-2", &43])
        .expect("insert failed");

let id: i32 = row.get(0);
println!("inserted item with id {}", id);
...
```

Наконец, цикл `for` используется для добавления `item-3`, `item-4` и `item-5` с произвольным количеством для каждого из них.

```rust
...
    for n in 3..=5 {
        let row = pg_client
            .query_one(
                &typed_prep_stmt,
                &[
                    &("item-".to_owned() + &n.to_string()),
                    &rand::thread_rng().gen_range(10..=50),
                ],
            )
            .expect("insert failed");

        let id: i32 = row.get(0);
        println!("inserted item with id {} ", id);
    }
...
```

### <a name="query-data"></a>Данные запросов

Функция `query_data` демонстрирует извлечение данных из таблицы `inventory`. Метод [query_one](https://docs.rs/postgres/0.19.0/postgres/struct.Client.html#method.query_one) используется для получения элемента по его свойству `id`. 

```rust
const SELECT_ALL_QUERY: &str = "SELECT * FROM inventory;";
const SELECT_BY_ID: &str = "SELECT name, quantity FROM inventory where id=$1;";

fn query_data(pg_client: &mut postgres::Client) {

    let prep_stmt = pg_client
        .prepare_typed(&SELECT_BY_ID, &[Type::INT4])
        .expect("failed to create prepared statement");

    let item_id = 1;

    let c = pg_client
        .query_one(&prep_stmt, &[&item_id])
        .expect("failed to query item");

    let name: String = c.get(0);
    let quantity: i32 = c.get(1);
    println!("quantity for item {} = {}", name, quantity);
...
}
```

Все строки в таблице инвентаризации извлекаться с помощью запроса `select * from` с методом [Query](https://docs.rs/postgres/0.19.0/postgres/struct.Client.html#method.query). Возвращаемые строки передаются для извлечения значения для каждого столбца с помощью [Get](https://docs.rs/postgres/0.19.0/postgres/row/struct.Row.html#method.get). 

> [!TIP]
> Обратите внимание, что `get` позволяет указать столбец по числовому индексу в строке или по имени столбца.

```rust
...
    let items = pg_client
        .query(SELECT_ALL_QUERY, &[])
        .expect("select all failed");

    println!("listing items...");

    for item in items {
        let id: i32 = item.get("id");
        let name: String = item.get("name");
        let quantity: i32 = item.get("quantity");
        println!(
            "item info: id = {}, name = {}, quantity = {} ",
            id, name, quantity
        );
    }
...
```

### <a name="update-data"></a>Обновление данных

Функция `update_date` случайным образом обновляет количество для всех элементов. Так как функция `insert_data` добавила `5` строк, она учитывается в `for` в цикле `for n in 1..=5`.

> [!TIP]
> Обратите внимание, что мы используем `query` вместо, `execute` поскольку мы планируем возвращать `id` и вновь созданный `quantity` (с помощью предложения [RETURNING](https://www.postgresql.org/docs/current/dml-returning.html)).

```rust
const UPDATE_QUERY: &str = "UPDATE inventory SET quantity = $1 WHERE name = $2 RETURNING quantity;";

fn update_data(pg_client: &mut postgres::Client) {
    let stmt = pg_client
        .prepare_typed(&UPDATE_QUERY, &[Type::INT4, Type::VARCHAR])
        .expect("failed to create prepared statement");

    for id in 1..=5 {
        let row = pg_client
            .query_one(
                &stmt,
                &[
                    &rand::thread_rng().gen_range(10..=50),
                    &("item-".to_owned() + &id.to_string()),
                ],
            )
            .expect("update failed");
        
        let quantity: i32 = row.get("quantity");
        println!("updated item id {} to quantity = {}", id, quantity);
    }
}
```

### <a name="delete-data"></a>Удаление данных

Наконец, `delete` функция показывает, как удалить элемент из таблицы `inventory` по его свойству `id`. `id` выбирается случайным образом — это случайное целое число от `1` до `5` (`5` включительно), так как `insert_data` функция добавила `5` строк. 

> [!TIP]
> Обратите внимание, что мы используем `query` вместо `execute`, поскольку планируется возвратить сведения о только что удаленном элементе (с помощью предложения [RETURNING](https://www.postgresql.org/docs/current/dml-returning.html)).

```rust
const DELETE_QUERY: &str = "DELETE FROM inventory WHERE id = $1 RETURNING id, name, quantity;";

fn delete(pg_client: &mut postgres::Client) {
    let stmt = pg_client
        .prepare_typed(&DELETE_QUERY, &[Type::INT4])
        .expect("failed to create prepared statement");

    let item = pg_client
        .query_one(&stmt, &[&rand::thread_rng().gen_range(1..=5)])
        .expect("delete failed");

    let id: i32 = item.get(0);
    let name: String = item.get(1);
    let quantity: i32 = item.get(2);
    println!(
        "deleted item info: id = {}, name = {}, quantity = {} ",
        id, name, quantity
    );
}
```

## <a name="run-the-application"></a>Выполнение приложения

1. Выполните следующую команду, чтобы клонировать репозиторий с примером.

    ```bash
    git clone https://github.com/Azure-Samples/azure-postgresql-rust-quickstart.git
    ```

2. Задайте необходимые переменные среды со значениями, скопированными из портала Azure:

    ```bash
    export POSTGRES_HOST=<server name e.g. my-server.postgres.database.azure.com>
    export POSTGRES_USER=<admin username e.g. my-admin-user@my-server>
    export POSTGRES_PASSWORD=<admin password>
    export POSTGRES_DBNAME=<database name. it is optional and defaults to postgres>
    ```

3. Чтобы запустить приложение, перейдите в каталог, в котором он был клонирован, и выполните `cargo run`.

    ```bash
    cd azure-postgresql-rust-quickstart
    cargo run
    ```

    Выходные данные должны иметь следующий вид:

    ```bash
    dropped 'inventory' table
    inserted item with id 1
    inserted item with id 2
    inserted item with id 3 
    inserted item with id 4 
    inserted item with id 5 
    quantity for item item-1 = 42
    listing items...
    item info: id = 1, name = item-1, quantity = 42 
    item info: id = 2, name = item-2, quantity = 43 
    item info: id = 3, name = item-3, quantity = 11 
    item info: id = 4, name = item-4, quantity = 32 
    item info: id = 5, name = item-5, quantity = 24 
    updated item id 1 to quantity = 27
    updated item id 2 to quantity = 14
    updated item id 3 to quantity = 31
    updated item id 4 to quantity = 16
    updated item id 5 to quantity = 10
    deleted item info: id = 4, name = item-4, quantity = 16 
    ```

4. Для подтверждения можно также подключиться к базе данных Azure для PostgreSQL [с помощью psql](./quickstart-create-server-database-portal.md#connect-to-the-server-with-psql) и выполнить запросы к базе данных, например:

    ```sql
    select * from inventory;
    ```

[Возникли проблемы? Сообщите нам об этом](https://aka.ms/postgres-doc-feedback)

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить все ресурсы, используемые во время этого краткого руководства, удалите группу ресурсов с помощью следующей команды:

```azurecli
az group delete \
    --name $AZ_RESOURCE_GROUP \
    --yes
```

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Управление сервером службы "База данных Azure для PostgreSQL" через портал](./howto-create-manage-server-portal.md)<br/>

> [!div class="nextstepaction"]
> [Управление сервером службы "База данных Azure для PostgreSQL" через CLI](./how-to-manage-server-cli.md)<br/>

[Не можете найти нужную информацию? Сообщите нам!](https://aka.ms/postgres-doc-feedback)
