---
title: Подключение приложения Go к API Azure Cosmos DB для MongoDB
description: В этом кратком руководстве показано, как подключить имеющееся приложение Go,к API Azure Cosmos DB для MongoDB.
author: abhirockzz
ms.author: abhishgu
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: go
ms.topic: quickstart
ms.date: 04/24/2020
ms.openlocfilehash: 92edfa148268db5a5458b2af4000bc9ffd9ecc83
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101659959"
---
# <a name="quickstart-connect-a-go-application-to-azure-cosmos-dbs-api-for-mongodb"></a>Краткое руководство. Подключение приложения Go к API Azure Cosmos DB для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](./mongodb-introduction.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

Azure Cosmos DB — это служба многомодельной базы данных, позволяющая быстро создавать и запрашивать документы, таблицы, пары "ключ-значение" и графовые базы данных, используя возможности глобального распределения и горизонтального масштабирования. В этом кратком руководстве описано, как создать учетную запись Azure Cosmos DB и управлять ею с помощью Azure Cloud Shell, клонировать имеющийся пример приложения из GitHub и настроить его для работы с Azure Cosmos DB. 

Пример приложения — это программа командной строки для управления элементами `todo`, написанная на языке Go. API-интерфейс Azure Cosmos DB для MongoDB [совместим с протоколом проводной сети MongoDB](./mongodb-introduction.md#wire-protocol-compatibility), благодаря чему к нему может подключиться любой клиентский драйвер MongoDB. Это приложение использует [драйвер Go для MongoDB](https://github.com/mongodb/mongo-go-driver) таким образом, что ему прозрачно предоставляются сведения о хранении данных в базе данных Azure Cosmos DB.

## <a name="prerequisites"></a>Предварительные требования
- Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free). Или [воспользуйтесь пробной версией Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) без подписки Azure. Вы также можете воспользоваться [эмулятором Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) со строкой подключения `.mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true`.
- На компьютере должна быть установлена среда [Go](https://golang.org/), а у вас должен быть опыт работы с этим языком.
- [Git](https://git-scm.com/downloads).
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Затем выполните следующие команды, чтобы клонировать репозиторий с примером.

1. Откройте командную строку, создайте папку `git-samples`, а затем закройте окно командной строки.

    ```bash
    mkdir "C:\git-samples"
    ```

2. Откройте окно терминала git, например git bash, и выполните команду `cd`, чтобы перейти в новую папку для установки примера приложения.

    ```bash
    cd "C:\git-samples"
    ```

3. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере. 

    ```bash
    git clone https://github.com/Azure-Samples/cosmosdb-go-mongodb-quickstart
    ```

## <a name="review-the-code"></a>Просмотр кода

Это необязательный шаг. Если вы хотите узнать, как работает приложение, изучите приведенные ниже фрагменты кода. В противном случае вы можете сразу перейти к разделу [Выполнение приложения](#run-the-application). Макет приложения выглядит следующим образом:

```bash
.
├── go.mod
├── go.sum
└── todo.go
```

Приведенные ниже фрагменты кода взяты из файла `todo.go`.

### <a name="connecting-the-go-app-to-azure-cosmos-db"></a>Подключение приложения Go к Azure Cosmos DB

[`clientOptions`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo/options?tab=doc#ClientOptions) инкапсулирует строку подключения для Azure Cosmos DB, которая передается с помощью переменной среды (дополнительные сведения см. в следующем разделе). Подключение инициализируется с помощью функции [`mongo.NewClient`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#NewClient), которой передается экземпляр `clientOptions`. В случае применения стратегии завершения работы при первой ошибке для подтверждения успешного подключения вызывается [функция `Ping`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Client.Ping).

```go
    ctx, cancel := context.WithTimeout(context.Background(), time.Second*10)
    defer cancel()

    clientOptions := options.Client().ApplyURI(mongoDBConnectionString).SetDirect(true)
    
    c, err := mongo.NewClient(clientOptions)
    err = c.Connect(ctx)
    if err != nil {
        log.Fatalf("unable to initialize connection %v", err)
    }

    err = c.Ping(ctx, nil)
    if err != nil {
        log.Fatalf("unable to connect %v", err)
    }
```

> [!NOTE] 
> При этом важно использовать конфигурацию [`SetDirect(true)`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo/options?tab=doc#ClientOptions.SetDirect), без которой произойдет следующая ошибка подключения: `unable to connect connection(cdb-ms-prod-<azure-region>-cm1.documents.azure.com:10255[-4]) connection is closed`.
>

### <a name="create-a-todo-item"></a>Создание элемента `todo`

Чтобы создать `todo`, можно запустить обработчик для [`mongo.Collection`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection) и вызвать функцию [`InsertOne`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection.InsertOne). 

```go
func create(desc string) {
    c := connect()
    ctx := context.Background()
    defer c.Disconnect(ctx)

    todoCollection := c.Database(database).Collection(collection)
    r, err := todoCollection.InsertOne(ctx, Todo{Description: desc, Status: statusPending})
    if err != nil {
        log.Fatalf("failed to add todo %v", err)
    }
```

Мы передаем структуру `Todo`, содержащую описание и состояние (для которого изначально установлено значение `pending`).

```go
type Todo struct {
    ID          primitive.ObjectID `bson:"_id,omitempty"`
    Description string             `bson:"description"`
    Status      string             `bson:"status"`
}
```
### <a name="list-todo-items"></a>Элементы списка `todo`

Список задач можно составлять на основе условий. Для инкапсуляции условий фильтра создается [`bson.D`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/bson?tab=doc#D).

```go
func list(status string) {
    .....
    var filter interface{}
    switch status {
    case listAllCriteria:
        filter = bson.D{}
    case statusCompleted:
        filter = bson.D{{statusAttribute, statusCompleted}}
    case statusPending:
        filter = bson.D{{statusAttribute, statusPending}}
    default:
        log.Fatal("invalid criteria for listing todo(s)")
    }
```

[`Find`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection.Find) используется для поиска документов, соответствующих условиям фильтра, а результат преобразуется в срез `Todo`.

```go
    todoCollection := c.Database(database).Collection(collection)
    rs, err := todoCollection.Find(ctx, filter)
    if err != nil {
        log.Fatalf("failed to list todo(s) %v", err)
    }
    var todos []Todo
    err = rs.All(ctx, &todos)
    if err != nil {
        log.Fatalf("failed to list todo(s) %v", err)
    }
```

Наконец, данные преобразовываются для просмотра в табличный формат.

```go
    todoTable := [][]string{}

    for _, todo := range todos {
        s, _ := todo.ID.MarshalJSON()
        todoTable = append(todoTable, []string{string(s), todo.Description, todo.Status})
    }

    table := tablewriter.NewWriter(os.Stdout)
    table.SetHeader([]string{"ID", "Description", "Status"})

    for _, v := range todoTable {
        table.Append(v)
    }
    table.Render()
```

### <a name="update-a-todo-item"></a>Обновление элемента `todo`

`todo` можно обновить на основе `_id`. Фильтр [`bson.D`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/bson?tab=doc#D) создается на основе `_id`. Для обновленных сведений, которые в данный момент находятся в состоянии new (`completed` или `pending`) создается еще один фильтр. Наконец, функция [`UpdateOne`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection.UpdateOne) вызывается с фильтром и обновленным документом.

```go
func update(todoid, newStatus string) {
....
    todoCollection := c.Database(database).Collection(collection)
    oid, err := primitive.ObjectIDFromHex(todoid)
    if err != nil {
        log.Fatalf("failed to update todo %v", err)
    }
    filter := bson.D{{"_id", oid}}
    update := bson.D{{"$set", bson.D{{statusAttribute, newStatus}}}}
    _, err = todoCollection.UpdateOne(ctx, filter, update)
    if err != nil {
        log.Fatalf("failed to update todo %v", err)
    }
```

### <a name="delete-a-todo"></a>Удаление `todo`

`todo` удаляется по `_id` и инкапсулируется в виде экземпляра [`bson.D`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/bson?tab=doc#D). Для удаления документа вызывается [`DeleteOne`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection.DeleteOne).

```go
func delete(todoid string) {
....
    todoCollection := c.Database(database).Collection(collection)
    oid, err := primitive.ObjectIDFromHex(todoid)
    if err != nil {
        log.Fatalf("invalid todo ID %v", err)
    }
    filter := bson.D{{"_id", oid}}
    _, err = todoCollection.DeleteOne(ctx, filter)
    if err != nil {
        log.Fatalf("failed to delete todo %v", err)
    }
}
```

## <a name="build-the-application"></a>создание приложения;

Перейдите в каталог, куда клонировано приложение, и создайте его (с помощью `go build`).

```bash
cd monogdb-go-quickstart
go build -o todo
```

Чтобы убедиться в правильности работы созданного приложения, воспользуйтесь следующей командой:

```bash
./todo --help
```

## <a name="setup-azure-cosmos-db"></a>Настройка Azure Cosmos DB

### <a name="sign-in-to-azure"></a>Вход в Azure

Если вы решили установить и использовать интерфейс командной строки локально, для работы с этим руководством вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI]. 

Если вы используете установленную версию Azure CLI, войдите в подписку Azure с помощью команды [az login](/cli/azure/reference-index#az-login) и следуйте инструкциям на экране. Этот шаг можно пропустить, если вы используете Azure Cloud Shell.

```azurecli
az login 
``` 
   
### <a name="add-the-azure-cosmos-db-module"></a>Добавление модуля Azure Cosmos DB

Если вы используете установленную версию Azure CLI, проверьте наличие компонента `cosmosdb`, выполнив команду `az`. Если компонент `cosmosdb` включен в список базовых команд, перейдите к следующей команде. Этот шаг можно пропустить, если вы используете Azure Cloud Shell.

Если компонента `cosmosdb` нет в списке базовых команд, переустановите [Azure CLI](/cli/azure/install-azure-cli).

### <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте [группу ресурсов](../azure-resource-manager/management/overview.md) с помощью команды [az group create](/cli/azure/group#az-group-create). Группа ресурсов Azure — это логический контейнер, в котором происходит развертывание ресурсов Azure (веб-приложений, баз данных и учетных записей хранения) и управление ими. 

В следующем примере показано создание группы ресурсов в регионе "Западная Европа". Выберите уникальное имя для группы ресурсов.

Если вы используете Azure Cloud Shell, щелкните **Попробовать**, войдите в систему, следуя указаниям на экране, а затем скопируйте команду в командную строку.

```azurecli-interactive
az group create --name myResourceGroup --location "West Europe"
```

### <a name="create-an-azure-cosmos-db-account"></a>создание учетной записи Azure Cosmos DB;

Создайте учетную запись Cosmos с помощью команды [az cosmosdb create](/cli/azure/cosmosdb#az-cosmosdb-create).

В следующей команде замените `<cosmosdb-name>` уникальным именем своей базы данных Cosmos везде, где встречается этот заполнитель. Это уникальное имя будет использоваться как часть конечной точки Cosmos DB (`https://<cosmosdb-name>.documents.azure.com/`), поэтому оно должно быть уникальным для всех учетных записей Cosmos в Azure. 

```azurecli-interactive
az cosmosdb create --name <cosmosdb-name> --resource-group myResourceGroup --kind MongoDB
```

Параметр `--kind MongoDB` разрешает клиентские подключения MongoDB.

После создания учетной записи Azure Cosmos DB в Azure CLI отображаются сведения, схожие с теми, которые приведены ниже. 

> [!NOTE]
> В этом примере JSON по умолчанию используется как формат выходных данных Azure CLI. Чтобы использовать другой формат выходных данных, ознакомьтесь со статьей [Форматы выходных данных для команд Azure CLI](/cli/azure/format-output-azure-cli). 

```json
{
  "databaseAccountOfferType": "Standard",
  "documentEndpoint": "https://<cosmosdb-name>.documents.azure.com:443/",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Document
DB/databaseAccounts/<cosmosdb-name>",
  "kind": "MongoDB",
  "location": "West Europe",
  "name": "<cosmosdb-name>",
  "readLocations": [
    {
      "documentEndpoint": "https://<cosmosdb-name>-westeurope.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "<cosmosdb-name>-westeurope",
      "locationName": "West Europe",
      "provisioningState": "Succeeded"
    }
  ],
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.DocumentDB/databaseAccounts",
  "writeLocations": [
    {
      "documentEndpoint": "https://<cosmosdb-name>-westeurope.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "<cosmosdb-name>-westeurope",
      "locationName": "West Europe",
      "provisioningState": "Succeeded"
    }
  ]
} 
```

### <a name="retrieve-the-database-key"></a>Получение ключа базы данных

Чтобы подключиться к базе данных Cosmos, вам понадобится ключ базы данных. Чтобы получить первичный ключ, выполните команду [az cosmosdb keys list](/cli/azure/cosmosdb/keys#az-cosmosdb-keys-list).

```azurecli-interactive
az cosmosdb keys list --name <cosmosdb-name> --resource-group myResourceGroup --query "primaryMasterKey"
```

В Azure CLI отображаются сведения, подобные следующим: 

```json
"RUayjYjixJDWG5xTqIiXjC..."
```

## <a name="configure-the-application"></a>Настройка приложения 

<a name="devconfig"></a>
### <a name="export-the-connection-string-mongodb-database-and-collection-names-as-environment-variables"></a>Экспортируйте строку подключения, базу данных MongoDB и имена коллекций в качестве переменных среды. 

```bash
export MONGODB_CONNECTION_STRING="mongodb://<COSMOSDB_ACCOUNT_NAME>:<COSMOSDB_PASSWORD>@<COSMOSDB_ACCOUNT_NAME>.documents.azure.com:10255/?ssl=true&replicaSet=globaldb&maxIdleTimeMS=120000&appName=@<COSMOSDB_ACCOUNT_NAME>@"
```

> [!NOTE] 
> Параметр `ssl=true` важен, так как он требуется для Cosmos DB. Дополнительные сведения см. в разделе [Требования к строке подключения](connect-mongodb-account.md#connection-string-requirements).
>

Для переменной среды `MONGODB_CONNECTION_STRING` замените заполнители для `<COSMOSDB_ACCOUNT_NAME>` и `<COSMOSDB_PASSWORD>`

1. `<COSMOSDB_ACCOUNT_NAME>`: имя учетной записи Azure Cosmos DB, которую вы создали.
2. `<COSMOSDB_PASSWORD>`: ключ базы данных, извлеченный на предыдущем шаге.

```bash
export MONGODB_DATABASE=todo-db
export MONGODB_COLLECTION=todos
```

Вы можете выбрать предпочтительные значения для `MONGODB_DATABASE` и `MONGODB_COLLECTION` или оставить их как есть.

## <a name="run-the-application"></a>Выполнение приложения

Создание `todo`

```bash
./todo --create "Create an Azure Cosmos DB database account"
```

В случае успеха вы увидите выходные данные с `_id` MongoDB для только что созданного документа.

```bash
added todo ObjectID("5e9fd6befd2f076d1f03bd8a")
```

Создайте еще один элемент `todo`.

```bash
./todo --create "Get the MongoDB connection string using the Azure CLI"
```

Перечислите все элементы `todo`.

```bash
./todo --list all
```

Только что добавленные элементы должны отображаться в табличном формате, как показано ниже.

```bash
+----------------------------+--------------------------------+-----------+
|             ID             |          DESCRIPTION           |  STATUS   |
+----------------------------+--------------------------------+-----------+
| "5e9fd6b1bcd2fa6bd267d4c4" | Create an Azure Cosmos DB      | pending   |
|                            | database account               |           |
| "5e9fd6befd2f076d1f03bd8a" | Get the MongoDB connection     | pending   |
|                            | string using the Azure CLI     |           |
+----------------------------+--------------------------------+-----------+
```

Чтобы обновить состояние `todo` (например, изменить его на `completed`), используйте идентификатор `todo`.

```bash
./todo --update 5e9fd6b1bcd2fa6bd267d4c4,completed
```

Перечислите только завершенные элементы `todo`.

```bash
./todo --list completed
```

Должны отобразиться те, которые вы только что обновили.

```bash
+----------------------------+--------------------------------+-----------+
|             ID             |          DESCRIPTION           |  STATUS   |
+----------------------------+--------------------------------+-----------+
| "5e9fd6b1bcd2fa6bd267d4c4" | Create an Azure Cosmos DB      | completed |
|                            | database account               |           |
+----------------------------+--------------------------------+-----------+
```

### <a name="view-data-in-data-explorer"></a>Просмотр данных в обозревателе данных

Данные, хранимые в Azure Cosmos DB, доступны для просмотра и запроса на портале Azure.

Чтобы просмотреть данные пользователя, созданные на предыдущем шаге, запросить их и начать с ними работать, войдите на [портал Azure](https://portal.azure.com) с помощью своего браузера.

В поле поиска в верхней области введите **Azure Cosmos DB**. Когда откроется колонка учетной записи Cosmos, выберите свою учетную запись Cosmos. В левой области навигации щелкните **Обозреватель данных**. Разверните свою коллекцию на панели коллекций. Вы сможете увидеть документы в коллекции, запросить данные и даже создать и запустить хранимые процедуры, триггеры и определенные пользователем функции. 

:::image type="content" source="./media/create-mongodb-go/go-cosmos-db-data-explorer.png" alt-text="Обозреватель данных, отображающий только что созданный документ":::


Удалите `todo`, используя его идентификатор.

```bash
./todo --delete 5e9fd6b1bcd2fa6bd267d4c4,completed
```

Выведите список элементов `todo` для подтверждения.

```bash
./todo --list all
```

Только что удаленный элемент `todo` должен отсутствовать.

```bash
+----------------------------+--------------------------------+-----------+
|             ID             |          DESCRIPTION           |  STATUS   |
+----------------------------+--------------------------------+-----------+
| "5e9fd6befd2f076d1f03bd8a" | Get the MongoDB connection     | pending   |
|                            | string using the Azure CLI     |           |
+----------------------------+--------------------------------+-----------+
```

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как создать учетную запись API MongoDB для Azure Cosmos DB с помощью Azure Cloud Shell, а также, как создать и запустить программу командной строки Go для управления элементами `todo`. Теперь можно импортировать дополнительные данные в учетную запись Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Перенос данных MongoDB в Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)