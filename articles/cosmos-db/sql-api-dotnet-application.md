---
title: Руководство по созданию веб-приложения ASP.NET Core MVC с помощью Azure Cosmos DB
description: Это руководство по MVC Core для ASP.NET поможет вам создать веб-приложение MVC с использованием Azure Cosmos DB. С помощью этого руководства по ASP NET Core MVC вы сможете сохранить файл JSON и получить доступ к данным из приложения "Список дел", размещенного в Службе приложений Azure.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 05/08/2020
ms.author: sngun
ms.custom: devx-track-dotnet
ms.openlocfilehash: 528cab915a1ac3918146e428e9ae6b3c401324c8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96010363"
---
# <a name="tutorial-develop-an-aspnet-core-mvc-web-application-with-azure-cosmos-db-by-using-net-sdk"></a>Руководство по Разработка веб-приложения ASP.NET Core MVC с использованием Azure Cosmos DB с помощью пакета SDK для .NET
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.js](sql-api-nodejs-application.md)
> * [Python](./create-sql-api-python.md)
> * [Xamarin](mobile-apps-with-xamarin.md)

В этом руководстве показано, как использовать Azure Cosmos DB для хранения данных и обеспечения доступа к ним из приложения ASP.NET MVC, размещенного в Azure. В этом руководстве используется пакет SDK для .NET версии 3. На следующем рисунке показана веб-страница, которая создается с помощью примера в этой статье:

:::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-image01.png" alt-text="Снимок экрана: веб-приложение MVC со списком дел, созданное с помощью этого руководства по ASP.NET Core MVC":::

Если у вас нет времени для работы с руководством, можно скачать полный пример проекта с [GitHub][GitHub].

Темы, рассматриваемые в этом руководстве:

> [!div class="checklist"]
>
> * Создание учетной записи Azure Cosmos.
> * Создание приложения ASP.NET Core MVC
> * Подключение приложения к Azure Cosmos DB.
> * Выполнение операций создания, чтения, обновления и удаления (CRUD) данных

> [!TIP]
> Предполагается, что у вас есть опыт использования ASP.NET Core MVC и службы приложений Azure. Если вы никогда не работали с ASP.NET Core или [необходимыми инструментами](#prerequisites), мы рекомендуем скачать полный пример проекта с портала [GitHub][GitHub], добавить требуемые пакеты NuGet и запустить его. Когда создадите проект, вы можете просмотреть эту статью, чтобы разобраться в коде этого проекта.

## <a name="prerequisites"></a><a name="prerequisites"></a>Предварительные требования

Перед выполнением инструкций, приведенных в этой статье, подготовьте следующие ресурсы:

* Активная учетная запись Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* Visual Studio 2019. [!INCLUDE [cosmos-db-emulator-vs](../../includes/cosmos-db-emulator-vs.md)]  

Все снимки экранов в этой статье взяты из Microsoft Visual Studio Community 2019. Если вы используете другую версию, экраны и параметры могут не совпадать полностью. Решение будет работать, если вы выполните все предварительные требования.

## <a name="step-1-create-an-azure-cosmos-account"></a><a name="create-an-azure-cosmos-account"></a>Шаг 1. Создание учетной записи Azure Cosmos

Давайте сначала создадим учетную запись Azure Cosmos. Если у вас уже есть учетная запись API SQL Azure Cosmos DB или вы используете эмулятор Azure Cosmos DB, можно перейти к разделу [Шаг 2. Создание приложения ASP.NET MVC](#create-a-new-mvc-application).

[!INCLUDE [create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

[!INCLUDE [keys](../../includes/cosmos-db-keys.md)]

В следующем разделе вы создадите приложение ASP.NET Core MVC.

## <a name="step-2-create-a-new-aspnet-core-mvc-application"></a><a name="create-a-new-mvc-application"></a>Шаг 2. Создание приложения ASP.NET Core MVC

1. Откройте Visual Studio и выберите **Создать проект**.

1. В окне **Создание проекта** найдите и выберите **Веб-приложение ASP.NET Core** для C#. Нажмите кнопку **Далее**, чтобы продолжить.

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-new-project-dialog.png" alt-text="Создание проекта веб-приложения ASP.NET Core":::

1. В окне **Настроить новый проект** присвойте проекту имя *todo* и щелкните **Создать**.

1. В окне **Создать веб-приложение ASP.NET Core** выберите **Веб-приложение (модель-представление-контроллер)** . Щелкните **Создать** , чтобы продолжить.

   Visual Studio создаст пустой проект MVC.

1. Щелкните **Отладка** > **Начать отладку** или нажмите клавишу F5, чтобы запустить приложение ASP.NET в локальной среде.

## <a name="step-3-add-azure-cosmos-db-nuget-package-to-the-project"></a><a name="add-nuget-packages"></a>Шаг 3. Добавление пакета Azure Cosmos DB NuGet в проект

Теперь, когда у нас есть большая часть кода платформы ASP.NET Core MVC, необходимого для этого решения, давайте добавим пакеты NuGet, требуемые для подключения к Azure Cosmos DB.

1. В **обозревателе решений** щелкните проект правой кнопкой мыши и выберите **Управление пакетами NuGet**.

1. В окне **Диспетчер пакетов NuGet** найдите и выберите **Microsoft.Azure.Cosmos**. Выберите пункт **Установить**.

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-nuget.png" alt-text="Установка пакета NuGet":::

   Visual Studio скачает и установит пакет Azure Cosmos DB вместе с зависимостями.

   Для установки пакета NuGet можно также воспользоваться **консолью диспетчера пакетов**. Для этого выберите **Инструменты** > **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов**. При появлении запроса введите следующую команду:

   ```ps
   Install-Package Microsoft.Azure.Cosmos
   ```
  
## <a name="step-4-set-up-the-aspnet-core-mvc-application"></a><a name="set-up-the-mvc-application"></a>Шаг 4. Настройка приложения ASP.NET Core MVC

Теперь мы добавим в это приложение MVC модели, представления и контроллеры.

### <a name="add-a-model"></a><a name="add-a-model"></a> Добавление модели

1. В **обозревателе решений** щелкните правой кнопкой мыши папку **Models** и выберите **Добавить** > **Класс**.

1. В области **Добавить новый элемент** присвойте классу имя *Item.cs* и щелкните **Добавить**.

1. Замените содержимое класса *Item.cs* приведенным ниже кодом.

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Models/Item.cs":::

Azure Cosmos DB использует для перемещения и хранения данных формат JSON. Чтобы управлять тем, как JSON сериализует и десериализует объекты, можно использовать атрибут `JsonProperty`. Класс `Item` демонстрирует использование атрибута `JsonProperty`. Этот код позволяет управлять форматом имени свойства в представлении JSON. Также этот код отвечает за переименование свойства .NET `Completed`.

### <a name="add-views"></a><a name="add-views"></a>Добавление представлений

Теперь давайте добавим следующие представления:

* представление создания элемента;
* представление удаления элемента;
* представление для получения сведений об элементе;
* представление изменения элемента;
* представление перечисления всех элементов.

#### <a name="create-item-view"></a><a name="AddNewIndexView"></a>Представление создания элемента

1. В **обозревателе решений** щелкните правой кнопкой мыши папку **Views** и выберите **Добавить** > **Новая папка**. Присвойте этой папке имя *Item*.

1. Щелкните правой кнопкой мыши пустую папку **Item** и выберите **Добавить** > **Представление**.

1. В разделе **Добавить представление MVC ASP.NET** внесите следующие изменения:

   * В поле **Имя представления** введите *Create*.
   * В поле **Шаблон** выберите **Create**.
   * В поле **Класс модели** выберите **Item (todo.Models)** (Элемент (todo.Models)).
   * Выберите **Use a layout page** (Использовать страницу макета) и введите *~/Views/Shared/_Layout.cshtml*.
   * Выберите **Добавить**.

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-add-mvc-view.png" alt-text="Снимок экрана: диалоговое окно &quot;Добавление представления MVC&quot;":::

1. Затем выберите **Добавить**, и Visual Studio создаст представление шаблона. Замените код в созданном файле следующим содержимым:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Views/Item/Create.cshtml":::

#### <a name="delete-item-view"></a><a name="AddEditIndexView"></a>Представление удаления элемента

1. В **обозревателе решений** cyjdf щелкните правой кнопкой мыши папку **Item** и выберите **Добавить** > **Представление**.

1. В разделе **Добавить представление MVC ASP.NET** внесите следующие изменения:

   * В поле **Имя представления** введите *Delete*.
   * В поле **Шаблон** выберите **Delete**.
   * В поле **Класс модели** выберите **Item (todo.Models)** (Элемент (todo.Models)).
   * Выберите **Use a layout page** (Использовать страницу макета) и введите *~/Views/Shared/_Layout.cshtml*.
   * Выберите **Добавить**.

1. Затем выберите **Добавить**, и Visual Studio создаст представление шаблона. Замените код в созданном файле следующим содержимым:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Views/Item/Delete.cshtml":::

#### <a name="add-a-view-to-get-an-item-details"></a><a name="AddItemIndexView"></a>Добавление представления для получения сведений об элементе

1. В **обозревателе решений** снова щелкните правой кнопкой мыши папку **Item** и выберите **Добавить** > **Представление**.

1. В окне **Добавление представления MVC** предоставьте следующие значения.

   * В поле **Имя представления** введите *Details*.
   * В поле **Шаблон** выберите **Details**.
   * В поле **Класс модели** выберите **Item (todo.Models)** (Элемент (todo.Models)).
   * Выберите **Use a layout page** (Использовать страницу макета) и введите *~/Views/Shared/_Layout.cshtml*.

1. Затем выберите **Добавить**, и Visual Studio создаст представление шаблона. Замените код в созданном файле следующим содержимым:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Views/Item/Details.cshtml":::

#### <a name="add-an-edit-item-view"></a><a name="AddEditIndexView"></a>Добавление представления "Редактирование элементов"

1. В **обозревателе решений** cyjdf щелкните правой кнопкой мыши папку **Item** и выберите **Добавить** > **Представление**.

1. В разделе **Добавить представление MVC ASP.NET** внесите следующие изменения:

   * В поле **Имя представления** введите *Изменить*.
   * В поле **Шаблон** выберите **Изменить**.
   * В поле **Класс модели** выберите **Item (todo.Models)** (Элемент (todo.Models)).
   * Выберите **Use a layout page** (Использовать страницу макета) и введите *~/Views/Shared/_Layout.cshtml*.
   * Выберите **Добавить**.

1. Затем выберите **Добавить**, и Visual Studio создаст представление шаблона. Замените код в созданном файле следующим содержимым:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Views/Item/Edit.cshtml":::

#### <a name="add-a-view-to-list-all-the-items"></a><a name="AddEditIndexView"></a>Добавление представления перечисления всех элементов

И наконец, добавьте представление для получения списка всех элементов, сделав следующее:

1. В **обозревателе решений** cyjdf щелкните правой кнопкой мыши папку **Item** и выберите **Добавить** > **Представление**.

1. В разделе **Добавить представление MVC ASP.NET** внесите следующие изменения:

   * В поле **Имя представления** введите *Индекс*.
   * В поле **Шаблон** выберите **Список**.
   * В поле **Класс модели** выберите **Item (todo.Models)** (Элемент (todo.Models)).
   * Выберите **Use a layout page** (Использовать страницу макета) и введите *~/Views/Shared/_Layout.cshtml*.
   * Выберите **Добавить**.

1. Затем выберите **Добавить**, и Visual Studio создаст представление шаблона. Замените код в созданном файле следующим содержимым:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Views/Item/Index.cshtml":::

Как только вы выполните эти действия, закройте в Visual Studio все документы *CSHTML*.

### <a name="declare-and-initialize-services"></a><a name="initialize-services"></a>Объявление и инициализация служб

Сначала мы добавим класс, который содержит необходимую логику для подключения к службе Azure Cosmos DB и ее использования. В этом руководстве мы инкапсулируем эту логику в класс с именем `CosmosDbService` и интерфейсом с именем `ICosmosDbService`. Эта служба выполняет операции CRUD. Также она отвечает за операции веб-каналов, такие как отображение незавершенных элементов, а также создание, редактирование и удаление элементов.

1. В **обозревателе решений** щелкните правой кнопкой мыши проект и выберите **Добавить** > **Новая папка**. Присвойте этой папке имя *Services*.

1. Щелкните правой кнопкой мыши папку **Services**, выберите **Добавить** > **Класс**. Назовите новый класс *CosmosDbService* и выберите **Добавить**.

1. Замените содержимое файла *CosmosDbService.cs* следующим кодом:

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Services/CosmosDbService.cs":::

1. Щелкните правой кнопкой мыши папку **Services**, выберите **Добавить** > **Класс**. Присвойте новому классу имя *ICosmosDbService* и щелкните **Добавить**.

1. Добавьте в класс *ICosmosDbService* приведенный далее код.

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Services/ICosmosDbService.cs":::

1. Откройте файл *Startup.cs* в решении и добавьте следующий метод **InitializeCosmosClientInstanceAsync**, который считывает конфигурацию и инициализирует клиент.

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Startup.cs" id="InitializeCosmosClientInstanceAsync" :::

1. В том же файле замените метод `ConfigureServices` следующим содержимым.

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Startup.cs" id="ConfigureServices":::

   Код на этом этапе инициализирует клиент, настроенный как одноэлементный экземпляр, который будет внедрен с помощью [внедрения зависимостей в ASP.NET Core](/aspnet/core/fundamentals/dependency-injection).

   Измените контроллер MVC по умолчанию на `Item`, изменив маршруты в методе `Configure` в том же файле.

   ```csharp
    app.UseEndpoints(endpoints =>
          {
                endpoints.MapControllerRoute(
                   name: "default",
                   pattern: "{controller=Item}/{action=Index}/{id?}");
          });
   ```


1. Определите конфигурацию в файле проекта *appsettings.json*, как показано в следующем примере кода.

   :::code language="json" source="~/samples-cosmosdb-dotnet-core-web-app/src/appsettings.json":::

### <a name="add-a-controller"></a><a name="add-a-controller"></a>Добавление контроллера

1. В **обозревателе решений** щелкните правой кнопкой мыши папку **Controllers** и выберите **Добавить** > **Контроллер**.

1. В разделе **Добавление шаблона** выберите **Контроллер MVC — пустой** и щелкните **Добавить**.

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-controller-add-scaffold.png" alt-text="Выбор пункта &quot;Контроллер MVC — пустой&quot; в разделе &quot;Добавление шаблона&quot;":::

1. Присвойте новому контроллеру имя *ItemController*.

1. Замените содержимое файла *ItemController.cs* приведенным ниже кодом.

   :::code language="csharp" source="~/samples-cosmosdb-dotnet-core-web-app/src/Controllers/ItemController.cs":::

Атрибут **ValidateAntiForgeryToken** используется здесь для защиты этого приложения от атак с подделкой межсайтовых запросов. Все представления должны нормально работать с этим маркером защиты от подделки запросов. Чтобы узнать больше и просмотреть примеры, ознакомьтесь со статьей [Preventing Cross-Site Request Forgery (CSRF) Attacks in ASP.NET MVC Application][Preventing Cross-Site Request Forgery] (Предупреждения атак с подделкой межсайтовых запросов в приложении ASP.NET MVC). Исходный код, предоставленный на [GitHub][GitHub], имеет полную реализацию.

Для защиты от атак overposting также используется атрибут **Bind** в параметре метода. Дополнительные сведения см. в статье [Учебник. Implement CRUD Functionality with the Entity Framework in ASP.NET MVC][Basic CRUD Operations in ASP.NET MVC] (Реализация функций CRUD с использованием Entity Framework в ASP.NET MVC).

## <a name="step-5-run-the-application-locally"></a><a name="run-the-application"></a>Шаг 5. Локальный запуск приложения

Чтобы проверить приложение на локальном компьютере, выполните следующие шаги.

1. Чтобы создать приложение в режиме отладки, откройте Visual Studio и нажмите клавишу F5. После этого будет создано приложение и откроется окно браузера с пустой сеткой, которую мы уже видели:

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-create-an-item-a.png" alt-text="Снимок экрана: веб-приложение со списком дел, созданное с помощью этого руководства":::
   
   Если вместо этого приложение открывает домашнюю страницу, добавьте к URL-адресу `/Item`.

1. Щелкните ссылку **Создать** и введите значения в поля **Имя** и **Описание**. Не устанавливайте флажок **Выполнено**. Если вы его установите, приложение добавит новый элемент в завершенном состоянии. Элемент больше не отображается в исходном списке.

1. Нажмите кнопку **создания**. Приложение возвращает вас к представлению **Индекс**, где в списке уже появился новый элемент. Вы можете добавить еще несколько элементов в **список задач**.

    :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-create-an-item.png" alt-text="Снимок экрана: представление &quot;Индекс&quot;":::
  
1. Выберите действие **Редактировать** рядом со строкой **Элемент** в списке. В приложении откроется представление **Изменить**, где вы можете изменить любое свойство объекта, в том числе флажок **Выполнено**. Если установить флажок **Выполнено** и щелкнуть **Сохранить**, приложение отобразит для пункта **Элемент** в списке состояние "Выполнено".

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-completed-item.png" alt-text="Снимок экрана: представление &quot;Индекс&quot; с установленным флажком &quot;Завершено&quot;":::

1. Проверьте состояние данных в службе Azure Cosmos DB с помощью [обозревателя Cosmos](https://cosmos.azure.com) или обозревателя данных эмулятора Azure Cosmos DB.

1. После проверки приложения нажмите сочетание клавиш CTRL+F5, чтобы остановить отладку приложения. Теперь все готово к развертыванию.

## <a name="step-6-deploy-the-application"></a><a name="deploy-the-application-to-azure"></a>Шаг 6. Развертывание приложения

Теперь, когда у вас есть готовое приложение, которое корректно работает в Azure Cosmos DB, мы собираемся развернуть его в службу приложений Azure.  

1. Чтобы опубликовать это приложение, щелкните проект правой кнопкой мыши в **обозревателе решений** и выберите действие **Опубликовать**.

1. В поле **Выберите целевой объект публикации** выберите **Служба приложений**.

1. Чтобы использовать существующий профиль Службы приложений, щелкните **Выбрать существующий**, а затем **Опубликовать**.

1. В **Службе приложений** выберите элемент **Подписка**. Используйте фильтр **Просмотр** для сортировки по группе ресурсов или типу ресурсов.

1. Найдите нужный профиль и щелкните **OK**. Затем выполните поиск необходимой Службы приложений Azure и нажмите кнопку **ОК**.

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-app-service-2019.png" alt-text="Диалоговое окно &quot;Служба приложений&quot; в Visual Studio":::

Другой вариант — создать новый профиль.

1. Как и в предыдущей процедуре, щелкните правой кнопкой мыши проект в **обозревателе решений** и выберите **Опубликовать**.
  
1. В поле **Выберите целевой объект публикации** выберите **Служба приложений**.

1. В поле **Выберите целевой объект публикации** выберите **Создать новый**, а затем **Опубликовать**.

1. В разделе **Служба приложений** введите имя веб-приложения и укажите нужные варианты подписки, группы ресурсов и плана службы приложений. Затем щелкните **Создать**.

   :::image type="content" source="./media/sql-api-dotnet-application/asp-net-mvc-tutorial-create-app-service-2019.png" alt-text="Диалоговое окно &quot;Создание Службы приложений&quot; в Visual Studio":::

Через несколько секунд Visual Studio опубликует ваше веб-приложение и запустит браузер, где вы увидите свой проект, запущенный в Azure!

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как создать веб-приложение ASP.NET Core MVC. Это приложение может обращаться к данным, сохраненным в Azure Cosmos DB. Теперь вы можете продолжить работу с этими ресурсами:

* [Partitioning in Azure Cosmos DB](./partitioning-overview.md) (Секционирование в Azure Cosmos DB)
* [Getting started with SQL queries](./sql-query-getting-started.md) (Начало работы с запросами SQL)
* [Как моделировать и секционировать данные в Azure Cosmos DB на примере реальной задачи](./how-to-model-partition-example.md)

[Visual Studio Express]: https://www.visualstudio.com/products/visual-studio-express-vs.aspx
[Microsoft Web Platform Installer]: https://www.microsoft.com/web/downloads/platform.aspx
[Preventing Cross-Site Request Forgery]: /aspnet/web-api/overview/security/preventing-cross-site-request-forgery-csrf-attacks
[Basic CRUD Operations in ASP.NET MVC]: /aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/implementing-basic-crud-functionality-with-the-entity-framework-in-asp-net-mvc-application
[GitHub]: https://github.com/Azure-Samples/cosmos-dotnet-core-todo-app
