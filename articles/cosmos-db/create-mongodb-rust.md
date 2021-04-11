---
title: Подключение приложения Rust к API Azure Cosmos DB для MongoDB
description: В этом кратком руководстве показано, как создать приложение Rust на основе API Azure Cosmos DB для MongoDB.
author: abhirockzz
ms.author: abhishgu
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: rust
ms.topic: quickstart
ms.date: 01/12/2021
ms.openlocfilehash: 91e7bafe98b1aceaf8fe27b07029291a48a31351
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102555658"
---
# <a name="quickstart-connect-a-rust-application-to-azure-cosmos-dbs-api-for-mongodb"></a>Краткое руководство. Подключение приложения Rust к API Azure Cosmos DB для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](./mongodb-introduction.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
> * [Rust](create-mongodb-rust.md)
>

Azure Cosmos DB — это служба многомодельной базы данных, позволяющая быстро создавать и запрашивать документы, таблицы, пары "ключ-значение" и графовые базы данных, используя возможности глобального распределения и горизонтального масштабирования. Приведенный в этой статье пример содержит простое приложение на основе командной строки, которое использует [драйвер Rust для MongoDB](https://github.com/mongodb/mongo-rust-driver). Так как API-интерфейс Azure Cosmos DB для MongoDB [совместим с протоколом проводной сети MongoDB](./mongodb-introduction.md#wire-protocol-compatibility), к нему может подключиться любой клиентский драйвер MongoDB.

Вы узнаете, как применять драйвер MongoDB Rust для взаимодействия с API Azure Cosmos DB для MongoDB, изучая код операций CRUD (создание, чтение, обновление, удаление), реализованный в нашем примере. Наконец, вы сможете запустить приложение локально, чтобы проверить его в действии.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free). Или [воспользуйтесь пробной версией Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) без подписки Azure. Вы также можете воспользоваться [эмулятором Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) со строкой подключения `.mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true`.
- [Rust](https://www.rust-lang.org/tools/install) (версия 1.39 или более поздняя).
- [Git](https://git-scm.com/downloads)

## <a name="set-up-azure-cosmos-db"></a>Настройка Azure Cosmos DB

Чтобы настроить учетную запись Azure Cosmos DB, следуйте приведенным [здесь](create-mongodb-dotnet.md) инструкциям. Приложению потребуется строка подключения к MongoDB, которую можно получить с помощью портала Azure. Подробные сведения см. в разделе [Получение строки подключения MongoDB для настройки](connect-mongodb-account.md#get-the-mongodb-connection-string-to-customize).

## <a name="run-the-application"></a>Выполнение приложения

### <a name="clone-the-sample-application"></a>Клонирование примера приложения

Затем выполните следующие команды, чтобы клонировать репозиторий с примером.

1. Откройте командную строку, создайте папку `git-samples`, а затем закройте окно командной строки.

    ```bash
    mkdir "C:\git-samples"
    ```

1. Откройте окно терминала git, например git bash, и выполните команду `cd`, чтобы перейти в новую папку для установки примера приложения.

    ```bash
    cd "C:\git-samples"
    ```

1. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере. 

    ```bash
    git clone https://github.com/Azure-Samples/cosmosdb-rust-mongodb-quickstart
    ```

### <a name="build-the-application"></a>Построение приложения

Чтобы собрать двоичный файл, сделайте следующее.

```bash
cargo build --release
```

### <a name="configure-the-application"></a>Настройка веб-приложения 

Экспортируйте строку подключения, базу данных MongoDB и имена коллекций как переменные среды. 

```bash
export MONGODB_URL="mongodb://<COSMOSDB_ACCOUNT_NAME>:<COSMOSDB_PASSWORD>@<COSMOSDB_ACCOUNT_NAME>.mongo.cosmos.azure.com:10255/?ssl=true&replicaSet=globaldb&maxIdleTimeMS=120000&appName=@<COSMOSDB_ACCOUNT_NAME>@"
```

> [!NOTE] 
> Параметр `ssl=true` важен, так как он требуется для Cosmos DB. Дополнительные сведения см. в разделе [Требования к строке подключения](connect-mongodb-account.md#connection-string-requirements).
>

Для переменной среды `MONGODB_URL` замените заполнители для `<COSMOSDB_ACCOUNT_NAME>` и `<COSMOSDB_PASSWORD>`

- `<COSMOSDB_ACCOUNT_NAME>`: имя учетной записи Azure Cosmos DB, которую вы создали.
- `<COSMOSDB_PASSWORD>`: ключ базы данных, извлеченный на предыдущем шаге.

```bash
export MONGODB_DATABASE=todos_db
export MONGODB_COLLECTION=todos
```

Вы можете выбрать предпочтительные значения для `MONGODB_DATABASE` и `MONGODB_COLLECTION` или оставить их как есть.

Чтобы запустить приложение, перейдите в соответствующую папку (где сохранен двоичный файл приложения):

```bash
cd target/release
```

Создание `todo`

```bash
./todo create "Create an Azure Cosmos DB database account"
```

В случае успеха вы увидите выходные данные с `_id` MongoDB для только что созданного документа.

```bash
inserted todo with id = ObjectId("5ffd1ca3004cc935004a0959")
```

Создайте еще один элемент `todo`.

```bash
./todo create "Get the MongoDB connection string using the Azure CLI"
```

Перечислите все элементы `todo`.

```bash
./todo list all
```

Вы увидите среди них те, которые только что добавили:

```bash
todo_id: 5ffd1ca3004cc935004a0959 | description: Create an Azure Cosmos DB database account | status: pending
todo_id: 5ffd1cbe003bcec40022c81c | description: Get the MongoDB connection string using the Azure CLI | status: pending
```

Чтобы изменить состояние `todo` (например, изменить его на `completed`), используйте идентификатор `todo` следующим образом:

```bash
./todo update 5ffd1ca3004cc935004a0959 completed

#output
updating todo_id 5ffd1ca3004cc935004a0959 status to completed
updated status for todo id 5ffd1ca3004cc935004a0959
```

Перечислите только завершенные элементы `todo`.

```bash
./todo list completed
```

Должны отобразиться те, которые вы только что обновили.

```bash
listing 'completed' todos

todo_id: 5ffd1ca3004cc935004a0959 | description: Create an Azure Cosmos DB database account | status: completed
```

Удалите `todo`, используя его идентификатор.

```bash
./todo delete 5ffd1ca3004cc935004a0959
```

Выведите список элементов `todo` для подтверждения.

```bash
./todo list all
```

Только что удаленный элемент `todo` должен отсутствовать.

### <a name="view-data-in-data-explorer"></a>Просмотр данных в обозревателе данных

Данные, хранимые в Azure Cosmos DB, доступны для просмотра и запроса на портале Azure.

Чтобы просмотреть данные пользователя, созданные на предыдущем шаге, запросить их и начать с ними работать, войдите на [портал Azure](https://portal.azure.com) с помощью своего браузера.

В поле поиска в верхней области введите **Azure Cosmos DB**. Когда откроется колонка учетной записи Cosmos, выберите свою учетную запись Cosmos. В левой области навигации щелкните **Обозреватель данных**. Разверните свою коллекцию на панели коллекций. Вы сможете увидеть документы в коллекции, запросить данные и даже создать и запустить хранимые процедуры, триггеры и определенные пользователем функции.

## <a name="review-the-code-optional"></a>Просмотр кода (необязательно)

Если вы хотите узнать, как работает приложение, изучите приведенные ниже в этом разделе фрагменты кода. Приведенные ниже фрагменты кода взяты из файла `src/main.rs`.

Функция `main` является для приложения `todo` точкой входа. Она ожидает, что URL-адрес подключения к API Azure Cosmos DB для MongoDB будет предоставлен в переменной среды `MONGODB_URL`. Создается новый экземпляр `TodoManager`, за которым следует [выражение `match`](https://doc.rust-lang.org/book/ch06-02-match.html), которое делегирует выполнение соответствующему методу `TodoManager` в зависимости от выбранной пользователем операции: `create`, `update`, `list` или `delete`.

```rust
fn main() {
    let conn_string = std::env::var_os("MONGODB_URL").expect("missing environment variable MONGODB_URL").to_str().expect("failed to get MONGODB_URL").to_owned();
    let todos_db_name = std::env::var_os("MONGODB_DATABASE").expect("missing environment variable MONGODB_DATABASE").to_str().expect("failed to get MONGODB_DATABASE").to_owned();
    let todos_collection_name = std::env::var_os("MONGODB_COLLECTION").expect("missing environment variable MONGODB_COLLECTION").to_str().expect("failed to get MONGODB_COLLECTION").to_owned();

    let tm = TodoManager::new(conn_string,todos_db_name.as_str(), todos_collection_name.as_str());
      
    let ops: Vec<String> = std::env::args().collect();
    let op = ops[1].as_str();

    match op {
        CREATE_OPERATION_NAME => tm.add_todo(ops[2].as_str()),
        LIST_OPERATION_NAME => tm.list_todos(ops[2].as_str()),
        UPDATE_OPERATION_NAME => tm.update_todo_status(ops[2].as_str(), ops[3].as_str()),
        DELETE_OPERATION_NAME => tm.delete_todo(ops[2].as_str()),
        _ => panic!(INVALID_OP_ERR_MSG)
    }
}
```

`TodoManager` представляет собой `struct` и инкапсулирует [mongodb::sync::Collection](https://docs.rs/mongodb/1.1.1/mongodb/sync/struct.Collection.html). При попытке создать экземпляр `TodoManager` с помощью функции `new` она инициирует подключение к API Azure Cosmos DB для MongoDB.

```rust
struct TodoManager {
    coll: Collection
}
....
impl TodoManager{
    fn new(conn_string: String, db_name: &str, coll_name: &str) -> Self{
        let mongo_client = Client::with_uri_str(&*conn_string).expect("failed to create client");
        let todo_coll = mongo_client.database(db_name).collection(coll_name);
            
        TodoManager{coll: todo_coll}
    }
....
```

И самое главное, `TodoManager` содержит вспомогательные методы для управления `todo`. Рассмотрим подробнее каждый из разделов.

Метод `add_todo` принимает параметр `todo` с описанием, указанным пользователем, и создает экземпляр структуры `Todo` следующего вида. Платформа [serde](https://github.com/serde-rs/serde) применяется для сопоставления (сериализации и десериализации) данных BSON в экземпляры структур `Todo` и наоборот. Обратите внимание, как используются атрибуты поля `serde` для настройки процесса сериализации и десериализации. Например, поле `todo_id` в Todo `struct` имеет тип `ObjectId` и хранится в MongoDB в формате `_id`.

```rust
#[derive(Serialize, Deserialize)]
struct Todo {
    #[serde(rename = "_id", skip_serializing_if = "Option::is_none")]
    todo_id: Option<bson::oid::ObjectId>,
    #[serde(rename = "description")]
    desc: String,
    status: String,
}
```

[Collection.insert_one](https://docs.rs/mongodb/1.1.1/mongodb/struct.Collection.html#method.insert_one) принимает экземпляр [Document](https://docs.rs/bson/1.1.0/bson/document/struct.Document.html) с представлением добавляемых данных `todo`. Обратите внимание, что преобразование из `Todo` в `Document` выполняется в два этапа с помощью сочетания [to_bson](https://docs.rs/bson/1.1.0/bson/ser/fn.to_bson.html) и [as_document](https://docs.rs/bson/1.1.0/bson/enum.Bson.html#method.as_document).

```rust
fn add_todo(self, desc: &str) {
    let new_todo = Todo {
        todo_id: None,
        desc: String::from(desc),
        status: String::from(TODO_PENDING_STATUS),
    };
    
    let todo_doc = mongodb::bson::to_bson(&new_todo).expect("struct to BSON conversion failed").as_document().expect("BSON to Document conversion failed").to_owned();
        
    let r = self.coll.insert_one(todo_doc, None).expect("failed to add todo");    
    println!("inserted todo with id = {}", r.inserted_id);
}
```

[Collection.find](https://docs.rs/mongodb/1.1.1/mongodb/struct.Collection.html#method.find) применяется для получения *всех* экземпляров `todo` или фильтрации их части по указанному пользователем значению состояния (`pending` или `completed`). Обратите внимание, что в цикле `while` каждый экземпляр `Document`, полученный в результате поиска, преобразуется в структуру `Todo` с помощью [bson::from_bson](https://docs.rs/bson/1.1.0/bson/de/fn.from_bson.html). Это противоположено тому действию, которое выполнялось в методе `add_todo`.

```rust
fn list_todos(self, status_filter: &str) {
    let mut filter = doc!{};
    if status_filter == TODO_PENDING_STATUS ||  status_filter == TODO_COMPLETED_STATUS{
        println!("listing '{}' todos",status_filter);
        filter = doc!{"status": status_filter}
    } else if status_filter != "all" {
        panic!(INVALID_FILTER_ERR_MSG)
    }

    let mut todos = self.coll.find(filter, None).expect("failed to find todos");
    
    while let Some(result) = todos.next() {
        let todo_doc = result.expect("todo not present");
        let todo: Todo = bson::from_bson(Bson::Document(todo_doc)).expect("BSON to struct conversion failed");
        println!("todo_id: {} | description: {} | status: {}", todo.todo_id.expect("todo id missing"), todo.desc, todo.status);
    }
}
```

Состояние `todo` можно изменить (с `pending` на `completed` или наоборот). `todo` преобразуется в [bson::oid::ObjectId](https://docs.rs/bson/1.1.0/bson/oid/struct.ObjectId.html), который затем применяется в методе [Collection.update_one](https://docs.rs/mongodb/1.1.1/mongodb/struct.Collection.html#method.update_one) для определения документа, который нужно обновить.

```rust
fn update_todo_status(self, todo_id: &str, status: &str) {

    if status != TODO_COMPLETED_STATUS && status != TODO_PENDING_STATUS {
        panic!(INVALID_FILTER_ERR_MSG)
    }

    println!("updating todo_id {} status to {}", todo_id, status);
    
    let id_filter = doc! {"_id": bson::oid::ObjectId::with_string(todo_id).expect("todo_id is not valid ObjectID")};

    let r = self.coll.update_one(id_filter, doc! {"$set": { "status": status }}, None).expect("update failed");
    if r.modified_count == 1 {
        println!("updated status for todo id {}",todo_id);
    } else if r.matched_count == 0 {
        println!("could not update. check todo id {}",todo_id);
    }
}
```

Удаление `todo` выполняется простым вызовом метода [Collection.delete_one](https://docs.rs/mongodb/1.1.1/mongodb/struct.Collection.html#method.delete_one).


```rust
fn delete_todo(self, todo_id: &str) {
    println!("deleting todo {}", todo_id);
    
    let id_filter = doc! {"_id": bson::oid::ObjectId::with_string(todo_id).expect("todo_id is not valid ObjectID")};

    self.coll.delete_one(id_filter, None).expect("delete failed").deleted_count;
}
``` 

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать учетную запись API MongoDB для Azure Cosmos DB с помощью Azure Cloud Shell, а также как создать и запустить приложение командной строки Rust для управления элементами `todo`. Теперь можно импортировать дополнительные данные в учетную запись Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Перенос данных MongoDB в Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)
