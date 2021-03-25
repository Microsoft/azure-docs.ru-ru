---
title: Руководство по Создание консольного приложения .NET для управления данными в учетной записи API SQL для Azure Cosmos DB
description: Руководство по Из этой статьи вы узнаете, как создать ресурсы API SQL для Azure Cosmos DB с помощью консольного приложения C#.
author: kirankumarkolli
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 11/05/2019
ms.author: kirankk
ms.custom: devx-track-csharp
ms.openlocfilehash: 2ece9ed5cb6eb6ea706db1ed615a64d001f2b17c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102423961"
---
# <a name="tutorial-build-a-net-console-app-to-manage-data-in-azure-cosmos-db-sql-api-account"></a>Руководство по Создание консольного приложения .NET для управления данными в учетной записи API SQL для Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET](sql-api-get-started.md)
> * [Java](./create-sql-api-java.md)
> * [Async Java](./create-sql-api-java.md)
> * [Node.js](sql-api-nodejs-get-started.md)
>

Добро пожаловать в руководство по началу работы с API SQL для Azure Cosmos DB. После изучения этого руководства у вас будет консольное приложение, которое создает ресурсы Azure Cosmos DB и отправляет запросы к ним.

В рамках этого руководства используется [пакет SDK Azure Cosmos DB для .NET](https://www.nuget.org/packages/Microsoft.Azure.Cosmos) версии 3.0 или более поздней версии. Вы можете работать с платформой [.NET Framework или .NET Core](https://dotnet.microsoft.com/download).

Темы, рассматриваемые в этом руководстве:

> [!div class="checklist"]
>
> * создание учетной записи Azure Cosmos и подключение к ней;
> * настройка проекта в Visual Studio;
> * создание базы данных и контейнера;
> * добавление элементов в контейнер;
> * выполнение запросов контейнера;
> * создание, чтение, обновление и удаление (CRUD) для элемента;
> * удаление базы данных.

У вас нет времени? Не беспокойтесь! Полное решение доступно на [GitHub](https://github.com/Azure-Samples/cosmos-dotnet-getting-started). Краткие инструкции см. в разделе [Получение полного решения для этого руководства](#GetSolution).

А теперь приступим к работе!

## <a name="prerequisites"></a>Предварительные требования

* Активная учетная запись Azure. Если у вас ее нет, зарегистрируйте [бесплатную учетную запись](https://azure.microsoft.com/free/).

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [!INCLUDE [cosmos-db-emulator-vs](../../includes/cosmos-db-emulator-vs.md)]

## <a name="step-1-create-an-azure-cosmos-db-account"></a>Шаг 1. создание учетной записи Azure Cosmos DB;

Давайте создадим учетную запись Azure Cosmos DB. Если у вас уже есть учетная запись, которую вы планируете использовать, пропустите этот раздел. Чтобы применить эмулятор Azure Cosmos DB, его нужно настроить в соответствии с указаниями в статье [Использование эмулятора Azure Cosmos для разработки и тестирования в локальной среде](local-emulator.md). После этого переходите к разделу [Шаг 2. Настройка проекта Visual Studio](#SetupVS).

[!INCLUDE [create-dbaccount-preview](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="step-2-set-up-your-visual-studio-project"></a><a id="SetupVS"></a>Шаг 2. Настройка проекта Visual Studio

1. Откройте Visual Studio и выберите **Создать проект**.
1. В окне **Создание проекта** выберите **Консольное приложение (.NET Framework)** для C# и нажмите кнопку **Далее**.
1. Присвойте проекту имя *CosmosGettingStartedTutorial* и выберите **Создать**.

    :::image type="content" source="./media/sql-api-get-started/configure-cosmos-getting-started-2019.png" alt-text="Настройка проекта":::

1. В **обозревателе решений** щелкните правой кнопкой мыши новое консольное приложение (оно находится в решении Visual Studio). Затем щелкните **Управление пакетами NuGet**.
1. В окне **Диспетчер пакетов NuGet** выберите **Обзор** и найдите *Microsoft.Azure.Cosmos*. Выберите **Microsoft.Azure.Cosmos** и щелкните **Установить**.

   :::image type="content" source="./media/sql-api-get-started/cosmos-getting-started-manage-nuget-2019.png" alt-text="Установка NuGet для клиентского пакета SDK для Azure Cosmos DB":::

   Идентификатором пакета для клиентской библиотеки API SQL Azure Cosmos DB является [Microsoft Azure Cosmos DB Client Library](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/).

Отлично! Теперь, когда мы завершили настройку, начнем писать код. Готовый проект, создаваемый в рамках этого руководства, вы найдете на странице [Developing a .NET console app using Azure Cosmos DB](https://github.com/Azure-Samples/cosmos-dotnet-getting-started) (Разработка консольного приложения .NET с помощью Azure Cosmos DB).

## <a name="step-3-connect-to-an-azure-cosmos-db-account"></a><a id="Connect"></a>Шаг 3. Подключение к учетной записи Azure Cosmos DB

1. Замените ссылки в начале приложения C# в файле *Program.cs* на указанные ниже:

   ```csharp
   using System;
   using System.Threading.Tasks;
   using System.Configuration;
   using System.Collections.Generic;
   using System.Net;
   using Microsoft.Azure.Cosmos;
   ```

1. Добавьте перечисленные ниже константы и переменные в класс `Program`.

    ```csharp
    public class Program
    {
        // ADD THIS PART TO YOUR CODE

        // The Azure Cosmos DB endpoint for running this sample.
        private static readonly string EndpointUri = "<your endpoint here>";
        // The primary key for the Azure Cosmos account.
        private static readonly string PrimaryKey = "<your primary key>";

        // The Cosmos client instance
        private CosmosClient cosmosClient;

        // The database we will create
        private Database database;

        // The container we will create.
        private Container container;

        // The name of the database and container we will create
        private string databaseId = "FamilyDatabase";
        private string containerId = "FamilyContainer";
    }
    ```

   > [!NOTE]
   > Если вы уже знакомы с предыдущей версией пакета SDK для .NET, термины *коллекция* и *документ* не будут для вас новыми. Так как Azure Cosmos DB поддерживает несколько моделей API, в версии 3.0+ пакета SDK для .NET используются стандартные термины *контейнер* и *элемент*. *Контейнер* может представлять собой коллекцию, граф или таблицу. *Элемент* может представлять собой документ, ребро, вершину или запись с некоторым содержимым и размещается внутри контейнера. Дополнительные сведения см. в статье о [работе с базами данных, контейнерами и элементами в Azure Cosmos DB](account-databases-containers-items.md).

1. Откройте [портал Azure](https://portal.azure.com). Найдите учетную записи Azure Cosmos DB и выберите элемент **Ключи**.

   :::image type="content" source="./media/sql-api-get-started/cosmos-getting-started-portal-keys.png" alt-text="Получение ключей Azure Cosmos DB на портале Azure":::

1. В файле *Program.cs* замените `<your endpoint URL>` значением **URI**. Замените `<your primary key>` фактическим значением **первичного ключа**.

1. Ниже метода **Main** добавьте новую асинхронную задачу с именем **GetStartedDemoAsync**, с помощью которой создается экземпляр `CosmosClient`.

    ```csharp
    public static async Task Main(string[] args)
    {
    }

    // ADD THIS PART TO YOUR CODE
    /*
        Entry point to call methods that operate on Azure Cosmos DB resources in this sample
    */
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
    }
    ```

    Мы используем **GetStartedDemoAsync** в качестве точки входа, которая вызывает методы для работы с ресурсами Azure Cosmos DB.

1. Добавьте указанный далее код, чтобы запустить асинхронную задачу **GetStartedDemoAsync** из метода **Main**. Метод **Main** будет перехватывать исключения и записывать их в консоль.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=Main)]

1. Нажмите клавишу F5, чтобы запустить приложение.

    В консоли появится такое сообщение: **End of demo, press any key to exit.** (Демонстрация завершена, нажмите любую клавишу, чтобы выйти). Это сообщение подтверждает, что приложение успешно подключилось к Azure Cosmos DB. Теперь можно закрыть окно консоли.

Поздравляем! Вы успешно подключились к учетной записи Azure Cosmos DB.

## <a name="step-4-create-a-database"></a>Шаг 4. Создание базы данных

База данных представляет собой логический контейнер элементов, разделенных между контейнерами. Базу данных можно создать с помощью метода `CreateDatabaseIfNotExistsAsync` или `CreateDatabaseAsync` из класса [CosmosClient](/dotnet/api/microsoft.azure.cosmos.cosmosclient).

1. Скопируйте и вставьте метод `CreateDatabaseAsync` ниже метода `GetStartedDemoAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=CreateDatabaseAsync&highlight=7)]

    Функция `CreateDatabaseAsync` позволяет создать базу данных с идентификатором `FamilyDatabase` (если она не была создана ранее), где идентификатор извлекается из поля `databaseId`.

1. Скопируйте и вставьте приведенный ниже код для вызова нового метода **CreateDatabaseAsync** ниже строки, в которой создается экземпляр CosmosClient.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);

        //ADD THIS PART TO YOUR CODE
        await this.CreateDatabaseAsync();
    }
    ```

    На этом этапе файл *Program.cs* должен выглядеть, как показано ниже, но с реальными значениями конечной точки и первичного ключа.

    ```csharp
    using System;
    using System.Threading.Tasks;
    using System.Configuration;
    using System.Collections.Generic;
    using System.Net;
    using Microsoft.Azure.Cosmos;

    namespace CosmosGettingStartedTutorial
    {
        class Program
        {
            // The Azure Cosmos DB endpoint for running this sample.
            private static readonly string EndpointUri = "<your endpoint here>";
            // The primary key for the Azure Cosmos account.
            private static readonly string PrimaryKey = "<your primary key>";

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
                    Console.WriteLine("Beginning operations...");
                    Program p = new Program();
                    await p.GetStartedDemoAsync();
                }
                catch (CosmosException de)
                {
                    Exception baseException = de.GetBaseException();
                    Console.WriteLine("{0} error occurred: {1}\n", de.StatusCode, de);
                }
                catch (Exception e)
                {
                    Console.WriteLine("Error: {0}\n", e);
                }
                finally
                {
                    Console.WriteLine("End of demo, press any key to exit.");
                    Console.ReadKey();
                }
            }

            /// <summary>
            /// Entry point to call methods that operate on Azure Cosmos DB resources in this sample
            /// </summary>
            public async Task GetStartedDemoAsync()
            {
                // Create a new instance of the Cosmos Client
                this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
                await this.CreateDatabaseAsync();
            }

            /// <summary>
            /// Create the database if it does not exist
            /// </summary>
            private async Task CreateDatabaseAsync()
            {
                // Create a new database
                this.database = await this.cosmosClient.CreateDatabaseIfNotExistsAsync(databaseId);
                Console.WriteLine("Created Database: {0}\n", this.database.Id);
            }
        }
    }
    ```

1. Нажмите клавишу F5, чтобы запустить приложение.

   > [!NOTE]
   > Если вы получаете ошибку исключения "503 — служба недоступна", то возможно, что требуемые [порты](sql-sdk-connection-modes.md#service-port-ranges) для прямого подключения блокируются брандмауэром. Чтобы устранить эту проблему, откройте необходимые порты или используйте подключение в режиме шлюза, как показано в следующем коде:
   ```csharp
     // Create a new instance of the Cosmos Client in Gateway mode
     this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey, new CosmosClientOptions()
            {
                ConnectionMode = ConnectionMode.Gateway
            });
   ```

Поздравляем! Вы успешно создали базу данных Azure Cosmos.  

## <a name="step-5-create-a-container"></a><a id="CreateColl"></a>Шаг 5. Создание контейнера

> [!WARNING]
> С помощью метода `CreateContainerIfNotExistsAsync` создается контейнер. Это влечет за собой необходимость оплаты. Дополнительные сведения см. на нашей [странице цен](https://azure.microsoft.com/pricing/details/cosmos-db/).
>
>

Контейнер можно создать с помощью метода [**CreateContainerIfNotExistsAsync**](/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync#Microsoft_Azure_Cosmos_Database_CreateContainerIfNotExistsAsync_Microsoft_Azure_Cosmos_ContainerProperties_System_Nullable_System_Int32__Microsoft_Azure_Cosmos_RequestOptions_System_Threading_CancellationToken_) или [**CreateContainerAsync**](/dotnet/api/microsoft.azure.cosmos.database.createcontainerasync#Microsoft_Azure_Cosmos_Database_CreateContainerAsync_Microsoft_Azure_Cosmos_ContainerProperties_System_Nullable_System_Int32__Microsoft_Azure_Cosmos_RequestOptions_System_Threading_CancellationToken_) из класса `CosmosDatabase`. Контейнер состоит из элементов (которые в случае API SQL являются документами JSON) и связанной логики приложения JavaScript на стороне сервера, например хранимых процедур, определяемых пользователем функций и триггеров.

1. Скопируйте и вставьте метод `CreateContainerAsync` ниже метода `CreateDatabaseAsync`. Функция `CreateContainerAsync` позволяет создать контейнер (если он не был создан ранее) с идентификатором `FamilyContainer`. Для идентификатора используется значение из поля `containerId` с разделением по свойству `LastName`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=CreateContainerAsync&highlight=9)]

1. Скопируйте и вставьте приведенный ниже код, где вы создали экземпляр CosmosClient, для вызова добавленного метода **CreateContainer**.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();

        //ADD THIS PART TO YOUR CODE
        await this.CreateContainerAsync();
    }
    ```

1. Нажмите клавишу F5, чтобы запустить приложение.

Поздравляем! Вы успешно создали контейнер Azure Cosmos.  

## <a name="step-6-add-items-to-the-container"></a><a id="CreateDoc"></a>Шаг 6. Добавление элементов в контейнер

Элемент можно создать с помощью метода [**CreateItemAsync**](/dotnet/api/microsoft.azure.cosmos.container.createitemasync#Microsoft_Azure_Cosmos_Container_CreateItemAsync__1___0_System_Nullable_Microsoft_Azure_Cosmos_PartitionKey__Microsoft_Azure_Cosmos_ItemRequestOptions_System_Threading_CancellationToken_) из класса `CosmosContainer`. При использовании API SQL все элементы проецируются как документы с произвольным содержимым JSON, которое определяет пользователь. Теперь можно вставить элемент в ваш контейнер Azure Cosmos.

Сначала создайте класс `Family`, который представляет объекты, хранящиеся в Azure Cosmos DB для нашего примера. Также создайте подклассы `Parent`, `Child`, `Pet` и `Address`, которые вызываются из `Family`. Элемент должен иметь свойство `Id`, сериализованное как `id` в файле JSON.

1. Нажмите сочетание клавиш Ctrl+Shift+A, чтобы открыть окно **Добавить новый элемент**. Добавьте в ваш проект новый класс `Family.cs`.

    :::image type="content" source="./media/sql-api-get-started/cosmos-getting-started-add-family-class-2019.png" alt-text="Снимок экрана: добавление в проект нового класса Family.cs":::

1. Скопируйте и вставьте в `Family.cs` элементы `Family`, `Parent`, `Child`, `Pet` и класс `Address`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Family.cs)]


1. Вернитесь к файлу *Program.cs* и добавьте метод `AddItemsToContainerAsync` после метода `CreateContainerAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=AddItemsToContainerAsync)]


    Этот код позволяет проверить, есть ли элемент с таким же идентификатором. Мы вставим два элемента: по одному для семей *Андерсен* и *Вейкфилд*.

1. Добавьте вызов в `AddItemsToContainerAsync` в метод `GetStartedDemoAsync`.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();
        await this.CreateContainerAsync();

        //ADD THIS PART TO YOUR CODE
        await this.AddItemsToContainerAsync();
    }
    ```

1. Нажмите клавишу F5, чтобы запустить приложение.

Поздравляем! Вы успешно создали два элемента Azure Cosmos.  

## <a name="step-7-query-azure-cosmos-db-resources"></a><a id="Query"></a>Шаг 7. Запрашивание ресурсов Azure Cosmos DB

Azure Cosmos DB поддерживает полнофункциональные запросы к документам JSON, хранящимся в каждом контейнере. Дополнительные сведения см. в статье о [начале работы с запросами SQL](./sql-query-getting-started.md). В следующем примере кода показано, как выполнить запрос к элементам, вставленным на предыдущем шаге.

1. Скопируйте метод `QueryItemsAsync` и вставьте его после метода `AddItemsToContainerAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=QueryItemsAsync&highlight=10-11,17-18)]

1. Добавьте вызов в ``QueryItemsAsync`` в метод ``GetStartedDemoAsync``.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();
        await this.CreateContainerAsync();
        await this.AddItemsToContainerAsync();

        //ADD THIS PART TO YOUR CODE
        await this.QueryItemsAsync();
    }
    ```

1. Нажмите клавишу F5, чтобы запустить приложение.

Поздравляем! Вы успешно отправили запрос в контейнер Azure Cosmos.

## <a name="step-8-replace-a-json-item"></a><a id="ReplaceItem"></a>Шаг 8. Замена элемента JSON

Теперь мы будем обновлять элемент в Azure Cosmos DB. Мы изменим свойство `IsRegistered` элемента `Family` и параметр `Grade` одного из его дочерних элементов.

1. Скопируйте метод `ReplaceFamilyItemAsync` и вставьте его после метода `QueryItemsAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=ReplaceFamilyItemAsync&highlight=15)]

1. Добавьте вызов в `ReplaceFamilyItemAsync` в метод `GetStartedDemoAsync`.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();
        await this.CreateContainerAsync();
        await this.AddItemsToContainerAsync();
        await this.QueryItemsAsync();

        //ADD THIS PART TO YOUR CODE
        await this.ReplaceFamilyItemAsync();
    }
    ```

1. Нажмите клавишу F5, чтобы запустить приложение.

Поздравляем! Вы успешно заменили элемент Azure Cosmos.

## <a name="step-9-delete-item"></a><a id="DeleteDocument"></a>Шаг 9. Удаление элемента

Теперь мы будем удалять элемент из Azure Cosmos DB.

1. Скопируйте метод `DeleteFamilyItemAsync` и вставьте его после метода `ReplaceFamilyItemAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=DeleteFamilyItemAsync&highlight=10)]

1. Добавьте вызов в `DeleteFamilyItemAsync` в метод `GetStartedDemoAsync`.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();
        await this.CreateContainerAsync();
        await this.AddItemsToContainerAsync();
        await this.QueryItemsAsync();
        await this.ReplaceFamilyItemAsync();

        //ADD THIS PART TO YOUR CODE
        await this.DeleteFamilyItemAsync();
    }
    ```

1. Нажмите клавишу F5, чтобы запустить приложение.

Поздравляем! Вы успешно удалили элемент Azure Cosmos.

## <a name="step-10-delete-the-database"></a><a id="DeleteDatabase"></a>Шаг 10. Удаление базы данных

Теперь мы удалим базу данных. При удалении созданной базы данных удаляются и все ее дочерние ресурсы. К таким ресурсам относятся контейнеры, элементы, а также хранимые процедуры, определяемые пользователем функции и триггеры. Мы также избавимся от экземпляра `CosmosClient`.

1. Скопируйте метод `DeleteDatabaseAndCleanupAsync` и вставьте его после метода `DeleteFamilyItemAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=DeleteDatabaseAndCleanupAsync)]

1. Добавьте вызов в ``DeleteDatabaseAndCleanupAsync`` в метод ``GetStartedDemoAsync``.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=GetStartedDemoAsync&highlight=14)]

1. Нажмите клавишу F5, чтобы запустить приложение.

Поздравляем! Вы успешно удалили базу данных Azure Cosmos.

## <a name="step-11-run-your-c-console-application-all-together"></a><a id="Run"></a>Шаг 11. Запуск консольного приложения C#

Чтобы создать приложение в режиме отладки, откройте Visual Studio и нажмите клавишу F5.

В окне консоли должны отобразиться все выходные данные вашего приложения. Эти выходные данные содержат результаты добавленных запросов. Они должны выглядеть так же, как представленный ниже пример текста.

```cmd
Beginning operations...

Created Database: FamilyDatabase

Created Container: FamilyContainer

Created item in database with id: Andersen.1 Operation consumed 11.43 RUs.

Created item in database with id: Wakefield.7 Operation consumed 14.29 RUs.

Running query: SELECT * FROM c WHERE c.LastName = 'Andersen'

        Read {"id":"Andersen.1","LastName":"Andersen","Parents":[{"FamilyName":null,"FirstName":"Thomas"},{"FamilyName":null,"FirstName":"Mary Kay"}],"Children":[{"FamilyName":null,"FirstName":"Henriette Thaulow","Gender":"female","Grade":5,"Pets":[{"GivenName":"Fluffy"}]}],"Address":{"State":"WA","County":"King","City":"Seattle"},"IsRegistered":false}

Updated Family [Wakefield,Wakefield.7].
        Body is now: {"id":"Wakefield.7","LastName":"Wakefield","Parents":[{"FamilyName":"Wakefield","FirstName":"Robin"},{"FamilyName":"Miller","FirstName":"Ben"}],"Children":[{"FamilyName":"Merriam","FirstName":"Jesse","Gender":"female","Grade":6,"Pets":[{"GivenName":"Goofy"},{"GivenName":"Shadow"}]},{"FamilyName":"Miller","FirstName":"Lisa","Gender":"female","Grade":1,"Pets":null}],"Address":{"State":"NY","County":"Manhattan","City":"NY"},"IsRegistered":true}

Deleted Family [Wakefield,Wakefield.7]

Deleted Database: FamilyDatabase

End of demo, press any key to exit.
```

Поздравляем! Вы завершили работу с этим руководством, и теперь у вас есть работающее консольное приложение C#.

## <a name="get-the-complete-tutorial-solution"></a><a id="GetSolution"></a>Получение готового решения для этого руководства

Если у вас нет времени на выполнение шагов из этого руководства или вы хотите просто скачать примеры кода, вы можете это сделать.

Для создания решения `GetStarted` вам потребуются указанные ниже компоненты.

* Активная учетная запись Azure. Если у вас ее нет, зарегистрируйте [бесплатную учетную запись](https://azure.microsoft.com/free/).
* [Учетная запись Azure Cosmos DB][cosmos-db-create-account].
* решение [GetStarted](https://github.com/Azure-Samples/cosmos-dotnet-getting-started) , доступное в GitHub.

Чтобы в Visual Studio восстановить ссылки на пакет SDK .NET для Azure Cosmos DB, в **обозревателе решений** щелкните правой кнопкой мыши решение, а затем щелкните **Восстановить пакеты NuGet**. В файле *App.config* обновите значения `EndPointUri` и `PrimaryKey`, как описано в разделе [Шаг 3. Подключение к учетной записи Azure Cosmos DB](#Connect).

Теперь все готово. Выполните сборку и начинайте работу с решением.

## <a name="next-steps"></a>Дальнейшие действия

* Требуется более подробное руководство по ASP.NET MVC? См. [Учебник. по разработке веб-приложения ASP.NET Core MVC с использованием Azure Cosmos DB с помощью пакета SDK для .NET](sql-api-dotnet-application.md).
* Нужно протестировать масштабирование и производительность с помощью Azure Cosmos DB? См. инструкции по [проверке производительности и масштабирования с использованием Azure Cosmos DB](performance-testing.md).
* Чтобы узнать, как отслеживать запросы, использование и хранение данных в Azure Cosmos DB, ознакомьтесь со статьей о [мониторинге метрик производительности и хранения в Azure Cosmos DB](./monitor-cosmos-db.md).
* Вы можете выполнить запросы к тестовому набору на [Площадке для тестирования запросов](https://www.documentdb.com/sql/demo).
* Дополнительные сведения об Azure Cosmos DB см. в статье [Добро пожаловать в базу данных Azure Cosmos DB](./introduction.md).

[cosmos-db-create-account]: create-sql-api-java.md#create-a-database-account