---
title: Azure Cosmos DB — Создание приложения со списком дел с использованием Xamarin
description: В этой статье представлен пример кода Xamarin, который можно использовать для подключения и выполнения запросов к Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 10/09/2020
ms.author: anfeldma
ms.custom: devx-track-csharp
ms.openlocfilehash: f02a0673a3fae33f935ae9a9cbf94f101b25c39f
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106449789"
---
# <a name="quickstart-build-a-todo-app-with-xamarin-using-azure-cosmos-db-sql-api-account"></a>Краткое руководство. Создание приложения со списком дел с помощью Xamarin и API SQL для Azure Cosmos DB | Документация Майкрософт
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET версии 3](create-sql-api-dotnet.md)
> * [.NET версии 4](create-sql-api-dotnet-V4.md)
> * [Пакет SDK для Java версии 4](create-sql-api-java.md)
> * [Spring Data версии 3](create-sql-api-spring-data.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)

Azure Cosmos DB — это глобально распределенная многомодельная служба базы данных Майкрософт. Вы можете быстро создавать и запрашивать документы, пары "ключ — значение" и базы данных графов, используя преимущества возможностей глобального распределения и горизонтального масштабирования базы данных Azure Cosmos DB.

> [!NOTE]
> Полный канонический пример кода для приложения Xamarin, демонстрирующий множество предложений Azure, включая Cosmos DB, можно найти [на этой странице](https://github.com/xamarinhq/app-geocontacts) сайта GitHub. Это приложение позволяет просмотреть географически распределенные контакты и обеспечивает обновление их расположения.

В этом кратком руководстве описано, как с помощью портала Azure создать учетную запись API SQL для Azure Cosmos DB, базу данных документов и контейнер. Затем вы создадите и развернете мобильное приложение со списком дел на основе [API-интерфейса .NET для SQL](sql-api-sdk-dotnet.md) и [Xamarin](/xamarin/), используя [Xamarin.Forms](/xamarin/) и [шаблон архитектуры MVVM](/xamarin/xamarin-forms/xaml/xaml-basics/data-bindings-to-mvvm).

:::image type="content" source="./media/create-sql-api-xamarin-dotnet/ios-todo-screen.png" alt-text="Приложение со списком дел Xamarin в iOS":::

## <a name="prerequisites"></a>Предварительные требования

Если вы разрабатываете приложение в Windows, но еще не установили Visual Studio 2019, скачайте **бесплатный** [выпуск Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). При установке Visual Studio необходимо включить рабочие нагрузки **Разработка для Azure** и **Разработка мобильных приложений на .NET**.

Если вы используете Mac, скачайте **бесплатный** выпуск [Visual Studio для Mac](https://www.visualstudio.com/vs/mac/).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

## <a name="create-a-database-account"></a>Создание учетной записи базы данных

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="add-a-container"></a>Добавление контейнера

[!INCLUDE [cosmos-db-create-collection](../../includes/cosmos-db-create-collection.md)]

## <a name="add-sample-data"></a>Добавление демонстрационных данных

[!INCLUDE [cosmos-db-create-sql-api-add-sample-data](../../includes/cosmos-db-create-sql-api-add-sample-data.md)]

## <a name="query-your-data"></a>Обращение к данным

[!INCLUDE [cosmos-db-create-sql-api-query-data](../../includes/cosmos-db-create-sql-api-query-data.md)]

## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Теперь необходимо клонировать приложение API SQL Xamarin из GitHub, просмотреть код, получить ключи API и запустить приложение. Вы узнаете, как можно упростить работу с данными программным способом.

1. Откройте командную строку, создайте папку git-samples, а затем закройте окно командной строки.

    ```bash
    mkdir "C:\git-samples"
    ```

2. Откройте окно терминала git, например git bash, и выполните команду `cd`, чтобы перейти в новую папку для установки примера приложения.

    ```bash
    cd "C:\git-samples"
    ```

3. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере.

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-sql-xamarin-getting-started.git
    ```

4. В Visual Studio откройте файл **C:\git-samples\azure-cosmos-db-sql-xamarin-getting-started\src\ToDoItems.sln**. 

## <a name="obtain-your-api-keys"></a>Получение ключей API

Вернитесь на портал Azure, чтобы найти сведения о ключе API. Скопируйте эти данные в приложение.

1. На [портале Azure](https://portal.azure.com/) перейдите к учетной записи API SQL для Azure Cosmos DB и на левой панели навигации щелкните **Ключи**, а затем выберите **Ключи записи-чтения**. На следующем шаге используйте кнопки копирования в правой части экрана, чтобы скопировать универсальный код ресурса (URI) и первичный ключ в файл APIKeys.cs.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/keys.png" alt-text="Просмотр и копирование ключа доступа на портале Azure, колонка &quot;Ключи&quot;":::

2. В Visual Studio откройте файл **ToDoItems.Core/Helpers/APIKeys.cs**.

3. На портале Azure скопируйте значение **URI** с помощью кнопки копирования и добавьте его в качестве значения переменной `CosmosEndpointUrl` в файл APIKeys.cs.

    ```csharp
    //#error Enter the URL of your Azure Cosmos DB endpoint here
    public static readonly string CosmosEndpointUrl = "[URI Copied from Azure Portal]";
    ```

4. На портале Azure скопируйте значение **PRIMARY KEY** с помощью кнопки копирования и добавьте его в качестве значения `Cosmos Auth Key` в файл APIKeys.cs.

    ```csharp
    //#error Enter the read/write authentication key of your Azure Cosmos DB endpoint here
    public static readonly string CosmosAuthKey = "[PRIMARY KEY copied from Azure Portal";
    ```

[!INCLUDE [cosmos-db-auth-key-info](../../includes/cosmos-db-auth-key-info.md)]

## <a name="review-the-code"></a>Просмотр кода

В примере этого решения показано, как создать приложение со списком дел с помощью API SQL для Azure Cosmos DB и Xamarin.Forms. В приложении представлены две вкладки: на первой вкладке содержится представление списка с элементами списка дел, которые еще не завершены. На второй вкладке отображаются элементы списка дел, которые уже завершены. На первой вкладке можно не только просмотреть список незавершенных дел, но и добавить новые элементы списка дел, а также отметить элементы как завершенные.

:::image type="content" source="./media/create-sql-api-xamarin-dotnet/android-todo-screen.png" alt-text="Копирование данных JSON и нажатие кнопки &quot;Сохранить&quot; в обозревателе данных на портале Azure":::

Код в решении ToDoItems содержит:

* **ToDoItems.Core**:
   * проект .NET Standard, который содержит проект Xamarin.Forms и общий код логики приложения, используемый для элементов списка дел в Azure Cosmos DB.
* **ToDoItems.Android**:
  * проект, который содержит приложение Android.
* **ToDoItems.iOS**:
  * проект, который содержит приложение iOS.

Рассмотрим краткий обзор того, как приложение обменивается данными с Azure Cosmos DB.

* Во все проекты необходимо добавить пакет NuGet [Microsoft.Azure.DocumentDb.Core](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/).
* Класс `ToDoItem` в папке **ToDoItems.Core/Models** позволяет моделировать документы в созданном ранее контейнере **Элементы**. Обратите внимание, что в именах свойств учитывается регистр.
* Класс `CosmosDBService` в папке **ToDoItems.Core/Services** позволяет инкапсулировать данные о подключении к Azure Cosmos DB.
* Класс `CosmosDBService` содержит переменную типа `DocumentClient`. Переменная `DocumentClient` используется для настройки и выполнения запросов к учетной записи Azure Cosmos DB с созданием экземпляра:

    ```csharp
    docClient = new DocumentClient(new Uri(APIKeys.CosmosEndpointUrl), APIKeys.CosmosAuthKey);
    ```

* Для выполнения запроса к контейнеру на получение документов используется метод `DocumentClient.CreateDocumentQuery<T>`, как показано в функции `CosmosDBService.GetToDoItems`:

   [!code-csharp[](~/samples-cosmosdb-xamarin/src/ToDoItems.Core/Services/CosmosDBService.cs?name=GetToDoItems)] 

    `CreateDocumentQuery<T>` принимает URI, указывающий на контейнер, созданный в предыдущем разделе. Можно также указать операторы LINQ, такие как предложение `Where`. В этом случае возвращаются только незавершенные элементы списка дел.

    Функция `CreateDocumentQuery<T>` выполняется синхронно и возвращает `IQueryable<T>`. Но метод `AsDocumentQuery` позволяет преобразовать `IQueryable<T>` в объект `IDocumentQuery<T>`, который может выполняться асинхронно. Таким образом, поток пользовательского интерфейса для мобильных приложений не блокируется.

    Функция `IDocumentQuery<T>.ExecuteNextAsync<T>` позволяет извлечь страницу результатов из Azure Cosmos DB. Метод `HasMoreResults` проверяет на странице наличие дополнительных результатов, которые можно вернуть.

> [!TIP]
> Некоторые функции, которые работают с контейнерами и документами Azure Cosmos, принимают URI в качестве параметра, указывающего на адрес контейнера или документа. Этот код URI создается с помощью класса `URIFactory`. Этот класс можно использовать для создания URI баз данных, контейнеров и документов.

* Функция `ComsmosDBService.InsertToDoItem` демонстрирует, как вставить новый документ:

   [!code-csharp[](~/samples-cosmosdb-xamarin/src/ToDoItems.Core/Services/CosmosDBService.cs?name=InsertToDoItem)] 

    Наряду с элементом для вставки также указывается URI контейнера.

* Функция `CosmosDBService.UpdateToDoItem` демонстрирует, как заменить существующий документ новым:

   [!code-csharp[](~/samples-cosmosdb-xamarin/src/ToDoItems.Core/Services/CosmosDBService.cs?name=UpdateToDoItem)] 

    В этом случае нужно указать новый URI для идентификации заменяемого документа. Чтобы получить URI, передайте функции `UriFactory.CreateDocumentUri` имя базы данных и контейнера, а также идентификатор документа.

    Функция `DocumentClient.ReplaceDocumentAsync` заменяет документ, соответствующий URI, другим документом, указанным в параметре.

* Функция `CosmosDBService.DeleteToDoItem` демонстрирует, как удалить элемент:

   [!code-csharp[](~/samples-cosmosdb-xamarin/src/ToDoItems.Core/Services/CosmosDBService.cs?name=DeleteToDoItem)] 

    Снова обратите внимание на то, что для документа создается уникальный URI, который передается в функцию `DocumentClient.DeleteDocumentAsync`.

## <a name="run-the-app"></a>Запустите приложение

Теперь приложение со всеми сведениями, необходимыми для взаимодействия с Azure Cosmos DB, обновлено.

Ниже приведены инструкции по запуску приложения с помощью отладчика Visual Studio для Mac.

> [!NOTE]
> Для приложения версии Android используются такие же действия. Все различия будут рассмотрены далее. Чтобы выполнить отладку в Visual Studio в Windows, ознакомьтесь с документацией по предложению со списком дел [для iOS](/xamarin/ios/deploy-test/debugging-in-xamarin-ios?tabs=vswin) и [для Android](/xamarin/android/deploy-test/debugging/).

1. Сначала выберите целевую платформу. Для этого щелкните выделенный раскрывающийся список и выберите ToDoItems.iOS для iOS или ToDoItems.Android для Android.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/ide-select-platform.png" alt-text="Выбор платформы для отладки в Visual Studio для Mac":::

2. Чтобы начать отладку приложения, нажмите клавиши CMD+ВВОД или кнопку воспроизведения.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/ide-start-debug.png" alt-text="Запуск отладки в Visual Studio для Mac":::

3. По завершении запуска в симуляторе iOS или эмуляторе Android в приложении появятся две вкладки: в нижней части экрана — для iOS и в верхней части экрана — для Android. На первой вкладке отображаются незавершенные элементы списка дел, а на второй — завершенные.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/ios-droid-started.png" alt-text="Экран запуска приложения со списком дел":::

4. Чтобы завершить элемент в списке дел в iOS, переместите его влево и нажмите кнопку **Complete** (Завершить). Чтобы завершить элемент в списке дел в Android, коснитесь элемента и долго удерживайте его, а затем нажмите кнопку завершения.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/simulator-complete.png" alt-text="Завершение элемента списка дел":::

5. Чтобы изменить элемент списка дел, коснитесь элемента. Откроется новый экран для ввода новых значений. Нажмите кнопку сохранения, чтобы сохранить изменения в Azure Cosmos DB.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/simulator-edit.png" alt-text="Изменение элемента списка дел":::

6. Чтобы добавить элемент списка дел, нажмите кнопку **Добавить** в правом верхнем углу начального экрана. После этого откроется новая пустая страница для изменения.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/simulator-add.png" alt-text="Добавление элемента списка дел":::

## <a name="review-slas-in-the-azure-portal"></a>Просмотр соглашений об уровне обслуживания на портале Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве было показано, как создать учетную запись Azure Cosmos и контейнер с помощью обозревателя данных, а также выполнить сборку и развертывание приложения Xamarin. Теперь вы можете импортировать дополнительные данные в учетную запись Azure Cosmos.

> [!div class="nextstepaction"]
> [Импорт данных в DocumentDB с помощью средства миграции базы данных](import-data.md)
