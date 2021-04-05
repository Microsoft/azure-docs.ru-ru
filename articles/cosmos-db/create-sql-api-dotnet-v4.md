---
title: Управление ресурсами API SQL для Azure Cosmos DB с помощью пакета SDK для .NET версии 4
description: Краткое руководство по созданию консольного приложения c помощью пакета SDK для .NET версии 4 для управления ресурсами учетной записи API SQL для Azure Cosmos DB.
author: anfeldma-ms
ms.author: anfeldma
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 09/22/2020
ms.custom: devx-track-dotnet
ms.openlocfilehash: 224a1b67ff0282c216763229593fcfed81d7567b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93090109"
---
# <a name="quickstart-build-a-console-app-using-the-net-v4-sdk-to-manage-azure-cosmos-db-sql-api-account-resources"></a>Краткое руководство. Создание консольного приложения c помощью пакета SDK для .NET версии 4 для управления ресурсами учетной записи API SQL для Azure Cosmos DB.
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET версии 3](create-sql-api-dotnet.md)
> * [.NET версии 4](create-sql-api-dotnet-V4.md)
> * [Пакет SDK для Java версии 4](create-sql-api-java.md)
> * [Spring Data версии 3](create-sql-api-spring-data.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)

Azure Cosmos DB. Руководство по началу работы с клиентской библиотекой API SQL для .NET. Следуйте инструкциям в этом документе, чтобы установить пакет .NET версии 4 (Azure.Cosmos), создать приложение и опробовать пример кода для основных операций CRUD с данными, хранящимися в Azure Cosmos DB. 

Azure Cosmos DB — это быстрая база данных NoSQL от Майкрософт с открытыми API для использования в любом масштабе. С помощью Azure Cosmos DB вы можете быстро создавать базы данных с парами "ключ — значение", документами, графами и обращаться к ним. Клиентскую библиотеку API SQL для Azure Cosmos DB для .NET можно использовать для выполнения таких задач:

* Создание базы данных Azure Cosmos и контейнера.
* Добавление демонстрационных данных в контейнер.
* Запрос данных 
* Удаление базы данных

[Исходный код библиотеки](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/v4) | [Пакет (NuGet)](https://www.nuget.org/packages/Azure.Cosmos)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [можно создать](https://azure.microsoft.com/free/) или [воспользоваться бесплатной пробной версией Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) без подписки Azure, оплаты и каких-либо обязательств. 
* [Пакет SDK для .NET Core 3](https://dotnet.microsoft.com/download/dotnet-core). Чтобы проверить, какая версия доступна в вашей среде, выполните `dotnet --version`.

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

   ```bash
   dotnet new console –langVersion:8 -n todo
   ```

Измените каталог на созданную папку приложения. Чтобы создать приложение, выполните следующую команду:

   ```bash
   cd todo
   dotnet build
   ```

Ожидаемые выходные данные сборки будут выглядеть примерно следующим образом:

```bash
  Restore completed in 100.37 ms for C:\Users\user1\Downloads\CosmosDB_Samples\todo\todo.csproj.
  todo -> C:\Users\user1\Downloads\CosmosDB_Samples\todo\bin\Debug\netcoreapp3.0\todo.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:34.17
```

### <a name="install-the-azure-cosmos-db-package"></a><a id="install-package"></a>Установка пакета для Azure Cosmos DB

Оставаясь в каталоге приложения, установите клиентскую библиотеку Azure Cosmos DB для .NET Core с помощью команды.

   ```bash
   dotnet add package Azure.Cosmos --version 4.0.0-preview3
   ```

### <a name="copy-your-azure-cosmos-account-credentials-from-the-azure-portal"></a>Копирование учетных данных Azure Cosmos с портала Azure

Чтобы использовать пример приложения, нужно выполнить проверку подлинности доступа к вашей учетной записи хранения Azure Cosmos. Для проверки подлинности необходимо передать учетную запись Azure Cosmos в приложение. Чтобы просмотреть учетные данные учетной записи Azure Cosmos, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com/).

1. Перейдите к своей учетной записи Azure Cosmos.

1. Откройте панель **Ключи** и скопируйте **URI** и **первичный ключ** своей учетной записи. На следующем этапе вы добавите значение URI и ключа в переменную среды.

## <a name="object-model"></a><a id="object-model"></a>Объектная модель

Прежде чем приступить к созданию приложения, рассмотрим иерархию ресурсов в Azure Cosmos DB и объектную модель, используемую для создания и доступа к этим ресурсам. Azure Cosmos DB создает ресурсы в следующем порядке:

* Учетная запись Azure Cosmos 
* Базы данных 
* Контейнеры 
* Items

Дополнительные сведения об иерархии различных объектов см. в статье [Work with databases, containers, and items in Azure Cosmos DB](account-databases-containers-items.md) (Работа с базами данных, контейнерами и элементами в Azure Cosmos DB). Вы будете использовать следующие классы .NET для взаимодействия с этими ресурсами:

* CosmosClient: этот класс является логическим представлением службы баз данных Azure Cosmos DB на стороне клиента. Этот клиентский объект позволяет настраивать и выполнять запросы к службе.
* CreateDatabaseIfNotExistsAsync: этот метод создает (если не существует) или получает (если уже существует) ресурс базы данных в качестве асинхронной операции. 
* CreateContainerIfNotExistsAsync: этот метод создает (если не существует) или получает (если уже существует) контейнер данных в качестве асинхронной операции. Вы можете проверить код состояния из ответа, чтобы определить, был ли контейнер вновь создан (201) или возвращен имеющийся контейнер (200). 
* CreateItemAsync: этот метод создает элемент в контейнере.
* UpsertItemAsync: этот метод создает элемент в контейнере, если элемент еще не существует, или заменяет элемент, если он уже существует. 
* GetItemQueryIterator: этот метод создает запрос для элементов в контейнере базы данных Azure Cosmos с помощью инструкции SQL с параметризованными значениями. 
* DeleteAsync удаляет указанную базу данных из вашей учетной записи Azure Cosmos. Метод `DeleteAsync` только удаляет базу данных.

 ## <a name="code-examples"></a><a id="code-examples"></a>Примеры кода

Пример кода, описанный в этой статье, создает семейную базу данных в Azure Cosmos DB. База данных семьи содержит сведения о семье, такие как имя, адрес, местоположение, родители, дети и домашние животные. Перед заполнением данных в своей учетной записи Azure Cosmos определите свойства элемента семьи. Создайте класс с именем `Family.cs` на корневом уровне вашего примера приложения и добавьте в него следующий код:

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Family.cs)]

### <a name="add-the-using-directives--define-the-client-object"></a>Добавление директив using и определение объекта клиента

В каталоге проекта откройте файл `Program.cs` в редакторе и добавьте следующие директивы using в верхней части приложения:

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=Usings)]


В класс `Program` добавьте следующие глобальные переменные: конечные точки, ключи авторизации, имя базы данных и контейнер, которые будут созданы. Не забудьте заменить значения конечных точек и ключей авторизации в соответствии с вашей средой. 

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=Constants)]

Наконец, замените метод `Main`:

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=Main)]

### <a name="create-a-database"></a>Создание базы данных 

Определите метод `CreateDatabaseAsync` в классе `program.cs`. Этот метод создает `FamilyDatabase`, если он еще не существует.

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=CreateDatabaseAsync)]

### <a name="create-a-container"></a>Создание контейнера

Определите метод `CreateContainerAsync` в классе `Program`. Этот метод создает `FamilyContainer`, если он еще не существует. 

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=CreateContainerAsync)]

### <a name="create-an-item"></a>Создание элемента

Создайте элемент семейства, добавив в метод `AddItemsToContainerAsync` следующий код. Для создания элемента можно использовать метод `CreateItemAsync` или `UpsertItemAsync`.

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=AddItemsToContainerAsync)]

### <a name="query-the-items"></a>Запрос элементов

После вставки элемента вы можете выполнить запрос, чтобы получить подробную информацию о семействе "Андерсен". В следующем коде показано, как выполнить запрос, используя запрос SQL напрямую. SQL-запрос для получения сведений о семействе "Андерсон": `SELECT * FROM c WHERE c.LastName = 'Andersen'`. Определите метод `QueryItemsAsync` в классе `Program` и добавьте в него следующий код:

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=QueryItemsAsync)]

### <a name="replace-an-item"></a>Замена элемента 

Прочитайте элемент семейства и обновите его, добавив в метод `ReplaceFamilyItemAsync` следующий код.

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=ReplaceFamilyItemAsync)]

### <a name="delete-an-item"></a>Удаление элемента 

Удалите элемент семейства, добавив в метод `DeleteFamilyItemAsync` следующий код.

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=DeleteFamilyItemAsync)]

### <a name="delete-the-database"></a>Удаление базы данных 

Наконец, вы можете удалить базу данных, добавив в метод `DeleteDatabaseAndCleanupAsync` следующий код:

[!code-csharp[Main](~/cosmos-dotnet-v4-getting-started/src/Program.cs?name=DeleteDatabaseAndCleanupAsync)]

После добавления всех необходимых методов сохраните файл `Program`. 

## <a name="run-the-code"></a>Выполнение кода

Затем создайте и запустите приложение, чтобы создать ресурсы Azure Cosmos DB.

   ```bash
   dotnet run
   ```

Следующие выходные данные генерируются при запуске приложения. Вы также можете войти на портал Azure и проверить, созданы ли ресурсы:

   ```bash
   Created Database: FamilyDatabase
   
   Created Container: FamilyContainer
   
   Created item in database with id: Andersen.1
   
   Running query: SELECT * FROM c WHERE c.LastName = 'Andersen'
   
           Read {"id":"Andersen.1","LastName":"Andersen","Parents":[{"FamilyName":null,"FirstName":"Thomas"},{"FamilyName":null   "FirstName":"Mary Kay"}],"Children":[{"FamilyName":null,"FirstName":"Henriette Thaulow","Gender":"female","Grade":5,"Pets": [{"GivenName":"Fluffy"}]}],"Address":{"State":"WA","County":"King","City":"Seattle"},"IsRegistered":false}
   
   Updated Family [Wakefield,Wakefield.7].
           Body is now: {"id":"Wakefield.7","LastName":"Wakefield","Parents":[{"FamilyName":"Wakefield","FirstName":"Robin"}   {"FamilyName":"Miller","FirstName":"Ben"}],"Children":[{"FamilyName":"Merriam","FirstName":"Jesse","Gender":"female","Grade":6   "Pets":[{"GivenName":"Goofy"},{"GivenName":"Shadow"}]},{"FamilyName":"Miller","FirstName":"Lisa","Gender":"female","Grade":1   "Pets":null}],"Address":{"State":"NY","County":"Manhattan","City":"NY"},"IsRegistered":true}
   
   Deleted Family [Wakefield,Wakefield.7]
   
   Deleted Database: FamilyDatabase
   
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
