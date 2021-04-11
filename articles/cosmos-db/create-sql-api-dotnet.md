---
title: Краткое руководство. Создание консольного приложения .NET для управления ресурсами API SQL для Azure Cosmos DB
description: В этом кратком руководстве вы узнаете, как создать консольное приложение .NET для управления ресурсами учетной записи API SQL для Azure Cosmos DB.
author: anfeldma-ms
ms.author: anfeldma
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 10/21/2020
ms.custom: devx-track-dotnet
ms.openlocfilehash: f25c8e2ca7abeae0a918825654c55256182189ae
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104785365"
---
# <a name="quickstart-build-a-net-console-app-to-manage-azure-cosmos-db-sql-api-resources"></a>Краткое руководство. Создание консольного приложения .NET для управления ресурсами API SQL для Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET версии 3](create-sql-api-dotnet.md)
> * [.NET версии 4](create-sql-api-dotnet-V4.md)
> * [Пакет SDK для Java версии 4](create-sql-api-java.md)
> * [Spring Data версии 3](create-sql-api-spring-data.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)

Azure Cosmos DB. Руководство по началу работы с клиентской библиотекой API SQL для .NET. Следуйте инструкциям в этом документе, чтобы установить пакет .NET, создайте приложение и опробуйте пример кода для основных операций CRUD с данными, хранящимися в базе данных Azure Cosmos. 

Azure Cosmos DB — это быстрая база данных NoSQL от Майкрософт с открытыми API для использования в любом масштабе. С помощью Azure Cosmos DB вы можете быстро создавать базы данных с парами "ключ — значение", документами, графами и обращаться к ним. Клиентскую библиотеку API SQL для Azure Cosmos DB для .NET можно использовать для выполнения таких задач:

* Создание базы данных Azure Cosmos и контейнера.
* Добавление демонстрационных данных в контейнер.
* Запрос данных 
* Удаление базы данных

[Справочная документация по API](/dotnet/api/microsoft.azure.cosmos) | [Исходный код библиотеки](https://github.com/Azure/azure-cosmos-dotnet-v3) | [Пакет (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.Cosmos).

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [можно создать](https://azure.microsoft.com/free/) или [воспользоваться бесплатной пробной версией Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) без подписки Azure, оплаты и каких-либо обязательств. 
* [Пакет SDK для .NET Core 2.1 или более поздней версии](https://dotnet.microsoft.com/download/dotnet-core/2.1).

## <a name="setting-up"></a>Настройка

В этом разделе описывается создание учетной записи Azure Cosmos и настройка проекта, для управления ресурсами которого используется клиентская библиотека API SQL для Azure Cosmos DB для. NET. Пример кода, описанный в этой статье, создает базу данных `FamilyDatabase` и членов семьи (каждый член семьи является элементом) в этой базе данных. Каждый член семьи имеет такие свойства, как `Id, FamilyName, FirstName, LastName, Parents, Children, Address,`. Свойство `LastName` используется в качестве ключа секции для контейнера. 

### <a name="create-an-azure-cosmos-account"></a><a id="create-account"></a>Создание учетной записи Azure Cosmos

Если вы используете опцию [Бесплатная пробная версия Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) для создания учетной записи Azure Cosmos, необходимо создать учетную запись Azure Cosmos DB типа **API SQL**. Тестовая учетная запись Azure Cosmos DB уже создана. Нет необходимости явно создавать учетную запись, поэтому вы можете пропустить этот раздел и перейти к следующему.

Если у вас есть собственная подписка Azure или вы создали бесплатную подписку, вам следует явно создать учетную запись Azure Cosmos. Следующий код создаст учетную запись Azure Cosmos с согласованностью сеанса. Учетная запись реплицируется в `South Central US` и в `North Central US`.  

Для создания учетной записи Cosmos Azure можно использовать Azure Cloud Shell. Azure Cloud Shell — это интерактивная, аутентифицированная, доступная в браузере оболочка для управления ресурсами Azure. Она предоставляет гибкие возможности при выборе оболочки, соответствующей вашим методам работы, будь то Bash или PowerShell. В рамках этого краткого руководства выберите режим **Bash**. Для Azure Cloud Shell также требуется учетная запись хранения, ее можно создать при появлении соответствующего запроса.

Нажмите кнопку **Попробуйте** рядом со следующим кодом, выберите режим **Bash**, а затем — **Создать учетную запись хранения** и войдите в Cloud Shell. Затем скопируйте и вставьте следующий код в Azure Cloud Shell, после чего запустите его. Имя учетной записи Azure Cosmos должно быть глобально уникальным. Перед запуском команды обязательно обновите значение `mysqlapicosmosdb`.

```azurecli-interactive

# Set variables for the new SQL API account, database, and container
resourceGroupName='myResourceGroup'
location='southcentralus'

# The Azure Cosmos account name must be globally unique, make sure to update the `mysqlapicosmosdb` value before you run the command
accountName='mysqlapicosmosdb'

# Create a resource group
az group create \
    --name $resourceGroupName \
    --location $location

# Create a SQL API Cosmos DB account with session consistency and multi-region writes enabled
az cosmosdb create \
    --resource-group $resourceGroupName \
    --name $accountName \
    --kind GlobalDocumentDB \
    --locations regionName="South Central US" failoverPriority=0 --locations regionName="North Central US" failoverPriority=1 \
    --default-consistency-level "Session" \
    --enable-multiple-write-locations true

```

Создание учетной записи Cosmos Azure занимает некоторое время. После успешного выполнения операции можно просмотреть выходные данные подтверждения. После успешного выполнения команды войдите на [портал Azure](https://portal.azure.com/) и убедитесь, что учетная запись Azure Cosmos с указанным именем существует. Окно Azure Cloud Shell можно закрыть после создания ресурса. 

### <a name="create-a-new-net-app"></a><a id="create-dotnet-core-app"></a>Создание приложения .NET

Создайте консольное приложение .NET на C# в предпочитаемой интегрированной среде разработки или редакторе. Откройте командную строку Windows или окно терминала с локального компьютера. Все команды в следующих разделах вы будете запускать из командной строки или терминала.  Запустите следующую команду dotnet new, чтобы создать новое приложение с именем `todo`. Параметр langVersion задает свойство LangVersion в созданном файле проекта.

```console
dotnet new console --langVersion 7.1 -n todo
```

Измените каталог на созданную папку приложения. Чтобы создать приложение, выполните следующую команду:

```console
cd todo
dotnet build
```

Ожидаемые выходные данные сборки будут выглядеть примерно следующим образом:

```console
  Restore completed in 100.37 ms for C:\Users\user1\Downloads\CosmosDB_Samples\todo\todo.csproj.
  todo -> C:\Users\user1\Downloads\CosmosDB_Samples\todo\bin\Debug\netcoreapp2.2\todo.dll
  todo -> C:\Users\user1\Downloads\CosmosDB_Samples\todo\bin\Debug\netcoreapp2.2\todo.Views.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:34.17
```

### <a name="install-the-azure-cosmos-db-package"></a><a id="install-package"></a>Установка пакета для Azure Cosmos DB

Оставаясь в каталоге приложения, установите клиентскую библиотеку Azure Cosmos DB для .NET Core с помощью команды.

```console
dotnet add package Microsoft.Azure.Cosmos
```

### <a name="copy-your-azure-cosmos-account-credentials-from-the-azure-portal"></a>Копирование учетных данных Azure Cosmos с портала Azure

Чтобы использовать пример приложения, нужно выполнить проверку подлинности доступа к вашей учетной записи хранения Azure Cosmos. Для проверки подлинности необходимо передать учетную запись Azure Cosmos в приложение. Чтобы просмотреть учетные данные учетной записи Azure Cosmos, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com/).

1. Перейдите к своей учетной записи Azure Cosmos.

1. Откройте панель **Ключи** и скопируйте **URI** и **первичный ключ** своей учетной записи. На следующем этапе вы добавите значение URI и ключа в переменную среды.

### <a name="set-the-environment-variables"></a>Настройка переменных среды

После того, как вы скопировали **URI** и **первичный ключ** своей учетной записи, сохраните их в новой переменной среды на локальном компьютере, на котором запущено приложение. Чтобы установить переменную среды, откройте окно консоли и выполните следующую команду. Обязательно замените значения `<Your_Azure_Cosmos_account_URI>` и `<Your_Azure_Cosmos_account_PRIMARY_KEY>`.

**Windows**

```console
setx EndpointUrl "<Your_Azure_Cosmos_account_URI>"
setx PrimaryKey "<Your_Azure_Cosmos_account_PRIMARY_KEY>"
```

**Linux**

```bash
export EndpointUrl = "<Your_Azure_Cosmos_account_URI>"
export PrimaryKey = "<Your_Azure_Cosmos_account_PRIMARY_KEY>"
```

**macOS**

```bash
export EndpointUrl = "<Your_Azure_Cosmos_account_URI>"
export PrimaryKey = "<Your_Azure_Cosmos_account_PRIMARY_KEY>"
```

 ## <a name="object-model"></a><a id="object-model"></a>Объектная модель

Прежде чем приступить к созданию приложения, рассмотрим иерархию ресурсов в Azure Cosmos DB и объектную модель, используемую для создания и доступа к этим ресурсам. Azure Cosmos DB создает ресурсы в следующем порядке:

* Учетная запись Azure Cosmos 
* Базы данных 
* Контейнеры 
* Items

Дополнительные сведения об иерархии различных объектов см. в статье [Work with databases, containers, and items in Azure Cosmos DB](account-databases-containers-items.md) (Работа с базами данных, контейнерами и элементами в Azure Cosmos DB). Вы будете использовать следующие классы .NET для взаимодействия с этими ресурсами:

* [CosmosClient](/dotnet/api/microsoft.azure.cosmos.cosmosclient). Этот класс является логическим представлением службы баз данных Azure Cosmos DB на стороне клиента. Этот клиентский объект позволяет настраивать и выполнять запросы к службе.

* [CreateDatabaseIfNotExistsAsync](/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync). Этот метод создает (если не существует) или получает (если уже существует) ресурс базы данных в качестве асинхронной операции. 

* [CreateContainerIfNotExistsAsync](/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync). Этот метод создает (если не существует) или получает (если уже существует) контейнер данных в качестве асинхронной операции. Вы можете проверить код состояния из ответа, чтобы определить, был ли контейнер вновь создан (201) или возвращен имеющийся контейнер (200). 
* [CreateItemAsync](/dotnet/api/microsoft.azure.cosmos.container.createitemasync). Этот метод создает элемент в контейнере. 

* [UpsertItemAsync](/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync). Этот метод создает элемент в контейнере, если элемент еще не существует, или заменяет элемент, если он уже существует. 

* [GetItemQueryIterator](/dotnet/api/microsoft.azure.cosmos.container.GetItemQueryIterator) — этот метод создает запрос для элементов в контейнере базы данных Azure Cosmos с помощью инструкции SQL с параметризованными значениями. 

* [DeleteAsync](/dotnet/api/microsoft.azure.cosmos.database.deleteasync). Удаляет указанную базу данных из вашей учетной записи Azure Cosmos. Метод `DeleteAsync` только удаляет базу данных. Удаление экземпляра `Cosmosclient` должно происходить отдельно (что происходит в методе DeleteDatabaseAndCleanupAsync). 

 ## <a name="code-examples"></a><a id="code-examples"></a>Примеры кода

Пример кода, описанный в этой статье, создает семейную базу данных в Azure Cosmos DB. База данных семьи содержит сведения о семье, такие как имя, адрес, местоположение, родители, дети и домашние животные. Перед заполнением данных в своей учетной записи Azure Cosmos определите свойства элемента семьи. Создайте класс с именем `Family.cs` на корневом уровне вашего примера приложения и добавьте в него следующий код:

```csharp
using Newtonsoft.Json;

namespace todo
{
    public class Family
    {
        [JsonProperty(PropertyName = "id")]
        public string Id { get; set; }
        public string LastName { get; set; }
        public Parent[] Parents { get; set; }
        public Child[] Children { get; set; }
        public Address Address { get; set; }
        public bool IsRegistered { get; set; }
        // The ToString() method is used to format the output, it's used for demo purpose only. It's not required by Azure Cosmos DB
        public override string ToString()
        {
            return JsonConvert.SerializeObject(this);
        }
    }

    public class Parent
    {
        public string FamilyName { get; set; }
        public string FirstName { get; set; }
    }

    public class Child
    {
        public string FamilyName { get; set; }
        public string FirstName { get; set; }
        public string Gender { get; set; }
        public int Grade { get; set; }
        public Pet[] Pets { get; set; }
    }

    public class Pet
    {
        public string GivenName { get; set; }
    }

    public class Address
    {
        public string State { get; set; }
        public string County { get; set; }
        public string City { get; set; }
    }
}
```

### <a name="add-the-using-directives--define-the-client-object"></a>Добавление директив using и определение объекта клиента

В каталоге проекта откройте файл `Program.cs` в редакторе и добавьте следующие директивы using в верхней части приложения:

```csharp

using System;
using System.Threading.Tasks;
using System.Configuration;
using System.Collections.Generic;
using System.Net;
using Microsoft.Azure.Cosmos;
```

В файл **Program.cs** добавьте код для чтения переменных среды, которые вы установили на предыдущем шаге. Определите объекты `CosmosClient`, `Database` и `Container`. Затем добавьте код в метод main, который вызывает метод `GetStartedDemoAsync`, где вы управляете ресурсами учетной записи Azure Cosmos. 

```csharp
namespace todo
{
public class Program
{

    /// The Azure Cosmos DB endpoint for running this GetStarted sample.
    private string EndpointUrl = Environment.GetEnvironmentVariable("EndpointUrl");

    /// The primary key for the Azure DocumentDB account.
    private string PrimaryKey = Environment.GetEnvironmentVariable("PrimaryKey");

    // The Cosmos client instance
    private CosmosClient cosmosClient;

    // The database we will create
    private Database database;

    // The container we will create.
    private Container container;

    // The name of the database and container we will create
    private string databaseId = "FamilyDatabase";
    private string containerId = "FamilyContainer";

    public static async Task Main(string[] args)
    {
       try
       {
           Console.WriteLine("Beginning operations...\n");
           Program p = new Program();
           await p.GetStartedDemoAsync();

        }
        catch (CosmosException de)
        {
           Exception baseException = de.GetBaseException();
           Console.WriteLine("{0} error occurred: {1}", de.StatusCode, de);
        }
        catch (Exception e)
        {
            Console.WriteLine("Error: {0}", e);
        }
        finally
        {
            Console.WriteLine("End of demo, press any key to exit.");
            Console.ReadKey();
        }
    }
}
}
```

### <a name="create-a-database"></a>Создание базы данных 

Определите метод `CreateDatabaseAsync` в классе `program.cs`. Этот метод создает `FamilyDatabase`, если он еще не существует.

```csharp
private async Task CreateDatabaseAsync()
{
    // Create a new database
    this.database = await this.cosmosClient.CreateDatabaseIfNotExistsAsync(databaseId);
    Console.WriteLine("Created Database: {0}\n", this.database.Id);
}
```

### <a name="create-a-container"></a>Создание контейнера

Определите метод `CreateContainerAsync` в классе `program.cs`. Этот метод создает `FamilyContainer`, если он еще не существует. 

```csharp
/// Create the container if it does not exist. 
/// Specifiy "/LastName" as the partition key since we're storing family information, to ensure good distribution of requests and storage.
private async Task CreateContainerAsync()
{
    // Create a new container
    this.container = await this.database.CreateContainerIfNotExistsAsync(containerId, "/LastName");
    Console.WriteLine("Created Container: {0}\n", this.container.Id);
}
```

### <a name="create-an-item"></a>Создание элемента

Создайте элемент семейства, добавив в метод `AddItemsToContainerAsync` следующий код. Для создания элемента можно использовать метод `CreateItemAsync` или `UpsertItemAsync`.

```csharp
private async Task AddItemsToContainerAsync()
{
    // Create a family object for the Andersen family
    Family andersenFamily = new Family
    {
        Id = "Andersen.1",
        LastName = "Andersen",
        Parents = new Parent[]
        {
           new Parent { FirstName = "Thomas" },
           new Parent { FirstName = "Mary Kay" }
        },
        Children = new Child[]
        {
           new Child
            {
                FirstName = "Henriette Thaulow",
                Gender = "female",
                Grade = 5,
                Pets = new Pet[]
                {
                    new Pet { GivenName = "Fluffy" }
                }
            }
        },
        Address = new Address { State = "WA", County = "King", City = "Seattle" },
        IsRegistered = false
    };

    try
    {
        // Create an item in the container representing the Andersen family. Note we provide the value of the partition key for this item, which is "Andersen".
        ItemResponse<Family> andersenFamilyResponse = await this.container.CreateItemAsync<Family>(andersenFamily, new PartitionKey(andersenFamily.LastName));
        // Note that after creating the item, we can access the body of the item with the Resource property of the ItemResponse. We can also access the RequestCharge property to see the amount of RUs consumed on this request.
        Console.WriteLine("Created item in database with id: {0} Operation consumed {1} RUs.\n", andersenFamilyResponse.Resource.Id, andersenFamilyResponse.RequestCharge);
    }
    catch (CosmosException ex) when (ex.StatusCode == HttpStatusCode.Conflict)
    {
        Console.WriteLine("Item in database with id: {0} already exists\n", andersenFamily.Id);                
    }
}

```

### <a name="query-the-items"></a>Запрос элементов

После вставки элемента вы можете выполнить запрос, чтобы получить подробную информацию о семействе "Андерсен". В следующем коде показано, как выполнить запрос, используя запрос SQL напрямую. SQL-запрос для получения сведений о семействе "Андерсон": `SELECT * FROM c WHERE c.LastName = 'Andersen'`. Определите метод `QueryItemsAsync` в классе `program.cs` и добавьте в него следующий код:


```csharp
private async Task QueryItemsAsync()
{
    var sqlQueryText = "SELECT * FROM c WHERE c.LastName = 'Andersen'";

    Console.WriteLine("Running query: {0}\n", sqlQueryText);

    QueryDefinition queryDefinition = new QueryDefinition(sqlQueryText);
    FeedIterator<Family> queryResultSetIterator = this.container.GetItemQueryIterator<Family>(queryDefinition);

    List<Family> families = new List<Family>();

    while (queryResultSetIterator.HasMoreResults)
    {
        FeedResponse<Family> currentResultSet = await queryResultSetIterator.ReadNextAsync();
        foreach (Family family in currentResultSet)
        {
            families.Add(family);
            Console.WriteLine("\tRead {0}\n", family);
        }
    }
}

```

### <a name="delete-the-database"></a>Удаление базы данных 

Наконец, вы можете удалить базу данных, добавив в метод `DeleteDatabaseAndCleanupAsync` следующий код:

```csharp
private async Task DeleteDatabaseAndCleanupAsync()
{
   DatabaseResponse databaseResourceResponse = await this.database.DeleteAsync();
   // Also valid: await this.cosmosClient.Databases["FamilyDatabase"].DeleteAsync();

   Console.WriteLine("Deleted Database: {0}\n", this.databaseId);

   //Dispose of CosmosClient
    this.cosmosClient.Dispose();
}
```

### <a name="execute-the-crud-operations"></a>Выполнение операций CRUD

После того как вы определили все необходимые методы, выполните их с помощью метода `GetStartedDemoAsync`. Метод `DeleteDatabaseAndCleanupAsync` закомментирован в этом коде, поэтому вы не увидите никаких ресурсов, если этот метод выполняется. Вы можете раскомментировать его после проверки того, что ваши ресурсы Azure Cosmos DB созданы на портале Azure. 

```csharp
public async Task GetStartedDemoAsync()
{
    // Create a new instance of the Cosmos Client
    this.cosmosClient = new CosmosClient(EndpointUrl, PrimaryKey);
    await this.CreateDatabaseAsync();
    await this.CreateContainerAsync();
    await this.AddItemsToContainerAsync();
    await this.QueryItemsAsync();
}
```

После добавления всех необходимых методов сохраните файл `Program.cs`. 

## <a name="run-the-code"></a>Выполнение кода

Затем создайте и запустите приложение, чтобы создать ресурсы Azure Cosmos DB. Обязательно откройте новое окно командной строки, не используйте тот же экземпляр, который вы использовали для установки переменных среды, так как переменные среды не установлены в текущем открытом окне. Вам нужно будет открыть новую командную строку, чтобы увидеть обновления. 

```console
dotnet build
```

```console
dotnet run
```

Следующие выходные данные генерируются при запуске приложения. Вы также можете войти на портал Azure и проверить, созданы ли ресурсы:

```console
Created Database: FamilyDatabase

Created Container: FamilyContainer

Created item in database with id: Andersen.1 Operation consumed 11.62 RUs.

Running query: SELECT * FROM c WHERE c.LastName = 'Andersen'

        Read {"id":"Andersen.1","LastName":"Andersen","Parents":[{"FamilyName":null,"FirstName":"Thomas"},{"FamilyName":null,"FirstName":"Mary Kay"}],"Children":[{"FamilyName":null,"FirstName":"Henriette Thaulow","Gender":"female","Grade":5,"Pets":[{"GivenName":"Fluffy"}]}],"Address":{"State":"WA","County":"King","City":"Seattle"},"IsRegistered":false}

End of demo, press any key to exit.
```

Вы можете проверить, что данные созданы, войдя на портал Azure и просмотрев необходимые элементы в своей учетной записи Azure Cosmos. 

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы больше не нужны, вы можете использовать интерфейс командной строки Azure или Azure PowerShell, чтобы удалить учетную запись Azure Cosmos и соответствующую группу ресурсов. Следующая команда показывает, как удалить группу ресурсов с помощью интерфейса командной строки Azure:

```azurecli
az group delete -g "myResourceGroup"
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как создать учетную запись Azure Cosmos, базу данных и контейнер с помощью приложения .NET Core. Теперь вы можете импортировать дополнительные данные в свою учетную запись Azure Cosmos, следуя инструкциям в следующей статье. 

> [!div class="nextstepaction"]
> [Импорт данных в DocumentDB с помощью средства миграции базы данных](import-data.md)
