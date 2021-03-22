---
title: Использование Кэша Azure для Redis с приложениями Rust
description: В этом кратком руководстве показано, как взаимодействовать с Кэшем Azure для Redis с помощью Rust.
author: abhirockzz
ms.author: abhishgu
ms.service: cache
ms.devlang: rust
ms.topic: quickstart
ms.date: 01/08/2021
ms.openlocfilehash: 17f38d79b75179d7a54ca5ed1d20dff18d0a0363
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102121105"
---
# <a name="quickstart-use-azure-cache-for-redis-with-rust"></a>Краткое руководство. Использование Кэша Azure для Redis с приложениями Rust

В этой статье показано, как использовать [язык программирования Rust](https://www.rust-lang.org/) для взаимодействия с [Кэшем Azure для Redis](./cache-overview.md). Здесь также демонстрируются примеры использования [String](https://redis.io/topics/data-types-intro#redis-strings), [Hash](https://redis.io/topics/data-types-intro#redis-hashes), [List](https://redis.io/topics/data-types-intro#redis-lists) и других популярных структур данных Redis с помощью библиотеки [redis-rs](https://github.com/mitsuhiko/redis-rs) для Redis. Этот клиент предоставляет высоко- и низкоуровневые API. Способ их использования показан на примерах кода в этой статье.

## <a name="skip-to-the-code-on-github"></a>Переход к коду на GitHub

Если вы хотите сразу перейти к коду, см. [краткое руководство по Rust](https://github.com/Azure-Samples/azure-redis-cache-rust-quickstart/) на сайте GitHub.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
- [Rust](https://www.rust-lang.org/tools/install) (версия 1.39 или более поздняя).
- [Git](https://git-scm.com/downloads)

## <a name="create-an-azure-cache-for-redis-instance"></a>Создание экземпляра кэша Azure для Redis
[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

[!INCLUDE [redis-cache-create](../../includes/redis-cache-access-keys.md)]

## <a name="review-the-code-optional"></a>Просмотр кода (необязательно)

Если вы хотите узнать, как работает код, изучите приведенные ниже фрагменты. Если нет, вы можете сразу перейти к разделу [Выполнение приложения](#run-the-application).

Для подключения к Кэшу Azure для Redis используется функция `connect`. Она принимает имя узла и пароль (ключ доступа) из переменных среды `REDIS_HOSTNAME` и `REDIS_PASSWORD` соответственно. Для URL-адреса подключения требуется формат `rediss://<username>:<password>@<hostname>`, так как Кэш Azure для Redis принимает только безопасные подключения с использованием [TLS 1.2 в качестве минимальной требуемой версии](cache-remove-tls-10-11.md).

Вызов [redis::Client::open](https://docs.rs/redis/0.19.0/redis/struct.Client.html#method.open) выполняет базовую проверку подлинности, а [get_connection()](https://docs.rs/redis/0.19.0/redis/struct.Client.html#method.get_connection) фактически инициирует подключение. Программа останавливается в случае сбоя подключения по любой причине, например из-за неправильного пароля.

```rust
fn connect() -> redis::Connection {
    let redis_host_name =
        env::var("REDIS_HOSTNAME").expect("missing environment variable REDIS_HOSTNAME");
    let redis_password =
        env::var("REDIS_PASSWORD").expect("missing environment variable REDIS_PASSWORD");
    let redis_conn_url = format!("rediss://:{}@{}", redis_password, redis_host_name);

    redis::Client::open(redis_conn_url)
        .expect("invalid connection URL")
        .get_connection()
        .expect("failed to connect to redis")
}
```

Функция `basics` поддерживает команды [SET](https://redis.io/commands/set), [GET](https://redis.io/commands/get) и [INCR](https://redis.io/commands/incr). Низкоуровневый API используется для `SET` и `GET`, которые задают и получают значение ключа с именем `foo`. Команда `INCRBY` выполняется черс использованием высокоуровневого API; например можно с помощью [incr](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.incr) увеличить значение ключа с именем `counter` на `2`, а затем вызвать [get](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.get), чтобы получить его значение.

```rust
fn basics() {
    let mut conn = connect();
    let _: () = redis::cmd("SET")
        .arg("foo")
        .arg("bar")
        .query(&mut conn)
        .expect("failed to execute SET for 'foo'");

    let bar: String = redis::cmd("GET")
        .arg("foo")
        .query(&mut conn)
        .expect("failed to execute GET for 'foo'");
    println!("value for 'foo' = {}", bar);

    let _: () = conn
        .incr("counter", 2)
        .expect("failed to execute INCR for 'counter'");
    let val: i32 = conn
        .get("counter")
        .expect("failed to execute GET for 'counter'");
    println!("counter = {}", val);
}
```

В приведенном ниже фрагменте кода демонстрируются возможности структуры данных Redis `HASH`. [HSET](https://redis.io/commands/hset) вызывается через низкоуровневый API для сохранения сведений (`name`, `version`, `repo`) о драйверах (клиентах) Redis. Например, сведения о драйвере Rust (который используется в этом примере кода) записывается в формате [BTreeMap](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html), а затем передаются в низкоуровневый API. Затем эти данные извлекается с помощью [HGETALL](https://redis.io/commands/hgetall).

Также можно выполнить `HSET` через высокоуровневый API, используя [hset_multiple](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.hset_multiple) с массивом кортежей. Затем выполняется [hget](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.hget) для извлечения значения одного атрибута (в нашем примере это `repo`).

```rust
fn hash() {
    let mut conn = connect();

    let mut driver: BTreeMap<String, String> = BTreeMap::new();
    let prefix = "redis-driver";
    driver.insert(String::from("name"), String::from("redis-rs"));
    driver.insert(String::from("version"), String::from("0.19.0"));
    driver.insert(
        String::from("repo"),
        String::from("https://github.com/mitsuhiko/redis-rs"),
    );

    let _: () = redis::cmd("HSET")
        .arg(format!("{}:{}", prefix, "rust"))
        .arg(driver)
        .query(&mut conn)
        .expect("failed to execute HSET");

    let info: BTreeMap<String, String> = redis::cmd("HGETALL")
        .arg(format!("{}:{}", prefix, "rust"))
        .query(&mut conn)
        .expect("failed to execute HGETALL");
    println!("info for rust redis driver: {:?}", info);

    let _: () = conn
        .hset_multiple(
            format!("{}:{}", prefix, "go"),
            &[
                ("name", "go-redis"),
                ("version", "8.4.6"),
                ("repo", "https://github.com/go-redis/redis"),
            ],
        )
        .expect("failed to execute HSET");

    let repo_name: String = conn
        .hget(format!("{}:{}", prefix, "go"), "repo")
        .expect("HGET failed");
    println!("go redis driver repo name: {:?}", repo_name);
}
```

В приведенной ниже функции показано, как использовать структуру данных `LIST`. Здесь выполняется [LPUSH](https://redis.io/commands/lpush) через низкоуровневый API для добавления записи в список, а затем метод [lpop](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.lpop) для получения этой записи из списка. После этого выполняется метод [rpush](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.rpush) для добавления в список нескольких записей, а затем они извлекаются с помощью низкоуровневого метода [lrange](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.lrange).

```rust
fn list() {
    let mut conn = connect();
    let list_name = "items";

    let _: () = redis::cmd("LPUSH")
        .arg(list_name)
        .arg("item-1")
        .query(&mut conn)
        .expect("failed to execute LPUSH for 'items'");

    let item: String = conn
        .lpop(list_name)
        .expect("failed to execute LPOP for 'items'");
    println!("first item: {}", item);

    let _: () = conn.rpush(list_name, "item-2").expect("RPUSH failed");
    let _: () = conn.rpush(list_name, "item-3").expect("RPUSH failed");

    let len: isize = conn
        .llen(list_name)
        .expect("failed to execute LLEN for 'items'");
    println!("no. of items in list = {}", len);

    let items: Vec<String> = conn
        .lrange(list_name, 0, len - 1)
        .expect("failed to execute LRANGE for 'items'");

    println!("listing items in list");
    for item in items {
        println!("item: {}", item)
    }
}
```

Здесь вы видите несколько примеров операций `SET`. Метод [sadd](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.sadd) (высокоуровневый API) используется для добавления нескольких записей в `SET` с именем `users`. Затем выполняется [SISMEMBER](https://redis.io/commands/hset) (низкоуровневый API) для проверки наличия `user1`. Наконец, выполняется [smembers](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.smembers) для получения и последовательного перебора полного набора записей в формате вектора ([Vec<String>](https://doc.rust-lang.org/std/vec/struct.Vec.html)).

```rust
fn set() {
    let mut conn = connect();
    let set_name = "users";

    let _: () = conn
        .sadd(set_name, "user1")
        .expect("failed to execute SADD for 'users'");
    let _: () = conn
        .sadd(set_name, "user2")
        .expect("failed to execute SADD for 'users'");

    let ismember: bool = redis::cmd("SISMEMBER")
        .arg(set_name)
        .arg("user1")
        .query(&mut conn)
        .expect("failed to execute SISMEMBER for 'users'");
    println!("does user1 exist in the set? {}", ismember);

    let users: Vec<String> = conn.smembers(set_name).expect("failed to execute SMEMBERS");
    println!("listing users in set");

    for user in users {
        println!("user: {}", user)
    }
}
```

В приведенной ниже функции `sorted_set` показана структура данных Sorted Set (упорядоченный набор). В ней вызывается [ZADD](https://redis.io/commands/zadd) (низкоуровневый API), чтобы добавить случайную целочисленную оценку для игрока `player-1`. Затем применяется метод [zadd](https://docs.rs/redis/0.19.0/redis/trait.Commands.html#method.zadd) (высокоуровневый API), чтобы добавить дополнительных игроков (с `player-2` по `player-5`) со случайными оценками для каждого. Число записей в отсортированном наборе вычисляется с помощью [ZCARD](https://redis.io/commands/zcard) и применяется в качестве ограничения для команды [ZRANGE](https://redis.io/commands/zrange) (низкоуровневый API), которая выводит список игроков в порядке возрастания.

```rust
fn sorted_set() {
    let mut conn = connect();
    let sorted_set = "leaderboard";

    let _: () = redis::cmd("ZADD")
        .arg(sorted_set)
        .arg(rand::thread_rng().gen_range(1..10))
        .arg("player-1")
        .query(&mut conn)
        .expect("failed to execute ZADD for 'leaderboard'");

    for num in 2..=5 {
        let _: () = conn
            .zadd(
                sorted_set,
                String::from("player-") + &num.to_string(),
                rand::thread_rng().gen_range(1..10),
            )
            .expect("failed to execute ZADD for 'leaderboard'");
    }

    let count: isize = conn
        .zcard(sorted_set)
        .expect("failed to execute ZCARD for 'leaderboard'");

    let leaderboard: Vec<(String, isize)> = conn
        .zrange_withscores(sorted_set, 0, count - 1)
        .expect("ZRANGE failed");

    println!("listing players and scores in ascending order");

    for item in leaderboard {
        println!("{} = {}", item.0, item.1)
    }
}
```

## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Прежде всего, клонируйте приложение из репозитория GitHub.

1. Откройте командную строку и создайте новую папку с именем `git-samples`.

    ```bash
    md "C:\git-samples"
    ```

1. Откройте окно терминала Git, например Git Bash. С помощью команды `cd` перейдите к новой папке, куда будете клонировать пример приложения.

    ```bash
    cd "C:\git-samples"
    ```

1. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере.

    ```bash
    git clone https://github.com/Azure-Samples/azure-redis-cache-rust-quickstart.git
    ```

## <a name="run-the-application"></a>Выполнение приложения

Приложение принимает сведения о подключении и учетные данные в виде переменных среды. 

1. Узнайте **имя узла** и **ключи доступа** в разделе "Ключи доступа" для экземпляра Кэша Azure для Redis на [портале Azure](https://portal.azure.com/).

1. Задайте их для соответствующих переменных среды:

    ```shell
    set REDIS_HOSTNAME=<Host name>:<port> (e.g. <name of cache>.redis.cache.windows.net:6380)
    set REDIS_PASSWORD=<Primary Access Key>
    ```

1. В окне терминала перейдите в нужную папку. Пример:

    ```shell
    cd "C:\git-samples\azure-redis-cache-rust-quickstart"
    ```

1. В сеансе терминала выполните следующую команду, чтобы запустить приложение.

    ```shell
    cargo run
    ```
    
    Должны отобразиться такие выходные данные:
    
    ```bash
    ******* Running SET, GET, INCR commands *******
    value for 'foo' = bar
    counter = 2
    ******* Running HASH commands *******
    info for rust redis driver: {"name": "redis-rs", "repo": "https://github.com/mitsuhiko/redis-rs", "version": "0.19.0"}
    go redis driver repo name: "https://github.com/go-redis/redis"
    ******* Running LIST commands *******
    first item: item-1
    no. of items in list = 2
    listing items in list
    item: item-2
    item: item-3
    ******* Running SET commands *******
    does user1 exist in the set? true
    listing users in set
    user: user2
    user: user1
    user: user3
    ******* Running SORTED SET commands *******
    listing players and scores
    player-2 = 2
    player-4 = 4
    player-1 = 7
    player-5 = 6
    player-3 = 8
    ```
    
    Если вы хотите выполнить определенную функцию, закомментируйте остальные функции в коде функции `main`:
    
    ```rust
    fn main() {
        basics();
        hash();
        list();
        set();
        sorted_set();
    }
    ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы закончили работу с группой ресурсов и ресурсами Azure, созданными в рамках этого краткого руководства, вы можете удалить их, чтобы избежать ненужных расходов.

> [!IMPORTANT]
> Удаление группы ресурсов — необратимая операция, и все соответствующие ресурсы удаляются окончательно. Если вы создали экземпляр Кэша Azure для Redis в существующей группе ресурсов, которую вы хотите сохранить, можно удалить только кэш, выбрав **Удалить** на странице **Обзор** кэша. 

Чтобы удалить группу ресурсов и ее экземпляра Кэша Azure для Redis, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com) найдите и выберите **Группы ресурсов**.
1. В текстовом поле **Фильтровать по имени** введите имя группы ресурсов, содержащей экземпляр кэша, а затем выберите его в результатах поиска. 
1. На странице группы ресурсов щелкните **Удалить группу ресурсов**.
1. Введите имя группы ресурсов, а затем щелкните **Удалить**.
   
   ![Удаление группы ресурсов для Кэша Azure для Redis](./media/cache-python-get-started/delete-your-resource-group-for-azure-cache-for-redis.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как с помощью драйвера Rust для Redis подключиться к Кэшу Azure для Redis и выполнять в нем операции.

> [!div class="nextstepaction"]
> [Создание простого веб-приложения ASP.NET, в котором используется кэш Redis для Azure](./cache-web-app-howto.md)
