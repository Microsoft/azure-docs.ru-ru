---
title: Руководство по Создание веб-приложения (Кэш на стороне). Кэш Azure для Redis
description: Узнайте, как создать веб-приложение с помощью кэша Azure для Redis, использующего шаблон "Кэш на стороне".
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: tutorial
ms.custom: devx-track-csharp, mvc
ms.date: 03/30/2018
ms.openlocfilehash: 90e60044e227ea1a18ea032d302b29abda1ea2e8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92536850"
---
# <a name="tutorial-create-a-cache-aside-leaderboard-on-aspnet"></a>Руководство по Создание списка лидеров с применением шаблона "Кэш на стороне" в ASP.NET

В этом руководстве мы обновим веб-приложение ASP.NET *ContosoTeamStats*, созданное в [руководстве ASP.NET для кэша Azure для Redis](cache-web-app-howto.md), чтобы включить список лидеров, который использует [шаблон "Кэш на стороне"](/azure/architecture/patterns/cache-aside) с кэшем Azure для Redis. В примере приложения отображается список статистических данных команды из базы данных и различные способы использования кэша Azure для Redis для хранения и извлечения данных из кэша с целью улучшения производительности. Завершив работу с руководством, вы получите рабочее веб-приложение, которое выполняет чтение и запись в базе данных, оптимизированное для работы с кэшем Azure для Redis и размещенное в Azure.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * улучшение пропускной способности данных и снижение нагрузки на базу данных за счет хранения и извлечения данных с помощью кэша Azure для Redis;
> * получение пяти лучших команд с помощью отсортированного набора Redis;
> * подготовка ресурсов Azure к работе для приложения с помощью шаблона Resource Manager;
> * публикация приложения в Azure с помощью Visual Studio.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством необходимо следующее:

* Это руководство является продолжением [краткого руководства по использованию ASP.NET для кэша Azure для Redis](cache-web-app-howto.md). Если вы еще этого не сделали, сначала ознакомьтесь с этим кратким руководством.
* Установите [Visual Studio 2019](https://www.visualstudio.com/downloads/) с указанными ниже рабочими нагрузками:
    * ASP.NET и веб-разработка.
    * разработка Azure.
    * разработка настольных приложений .NET с SQL Server Express LocalDB или [выпуском SQL Server 2017 Express](https://www.microsoft.com/sql-server/sql-server-editions-express).

## <a name="add-a-leaderboard-to-the-project"></a>Добавление списка лидеров в проект

В этом разделе руководства настройте проект *ContosoTeamStats*, добавив список лидеров, предоставляющий статистку о победах, проигрышах и ничьей для списка вымышленных команд.

### <a name="add-the-entity-framework-to-the-project"></a>Добавление пакета Entity Framework в проект

1. В Visual Studio откройте решение *ContosoTeamStats*, созданное при работе с [кратким руководством по использованию ASP.NET для кэша Azure для Redis](cache-web-app-howto.md).
2. Щелкните **Инструменты > Диспетчер пакетов NuGet > Консоль диспетчера пакетов**.
3. В окне **консоли диспетчера пакетов** запустите следующую команду, чтобы установить Entity Framework:

    ```powershell
    Install-Package EntityFramework
    ```

Дополнительные сведения об этом пакете см. в документации [EntityFramework](https://www.nuget.org/packages/EntityFramework/).

### <a name="add-the-team-model"></a>Добавление модели группы

1. В **обозревателе решений** щелкните правой кнопкой мыши папку **Модели**, а затем выберите **Добавить** и **Класс**.

1. Введите `Team` в качестве имени класса и нажмите кнопку **Добавить**.

    ![Добавление класса модели](./media/cache-web-app-cache-aside-leaderboard/cache-model-add-class-dialog.png)

1. В начале файла *Team.cs* замените операторы `using` следующими операторами `using`:

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Data.Entity;
    using System.Data.Entity.SqlServer;
    ```

1. Замените определение класса `Team` приведенным ниже фрагментом кода с обновлением определения класса `Team`, а также некоторыми другими вспомогательными классами Entity Framework. Это руководство использует подход Code First с Entity Framework. Такой подход позволяет Entity Framework создать базу данных из кода. Для дополнительных сведений о подходе Code First в Entity Framework, использованном в этом руководстве, см. видео в статье [Использование Code First для создания базы данных](/ef/ef6/modeling/code-first/workflows/new-database).

    ```csharp
    public class Team
    {
        public int ID { get; set; }
        public string Name { get; set; }
        public int Wins { get; set; }
        public int Losses { get; set; }
        public int Ties { get; set; }

        static public void PlayGames(IEnumerable<Team> teams)
        {
            // Simple random generation of statistics.
            Random r = new Random();

            foreach (var t in teams)
            {
                t.Wins = r.Next(33);
                t.Losses = r.Next(33);
                t.Ties = r.Next(0, 5);
            }
        }
    }

    public class TeamContext : DbContext
    {
        public TeamContext()
            : base("TeamContext")
        {
        }

        public DbSet<Team> Teams { get; set; }
    }

    public class TeamInitializer : CreateDatabaseIfNotExists<TeamContext>
    {
        protected override void Seed(TeamContext context)
        {
            var teams = new List<Team>
            {
                new Team{Name="Adventure Works Cycles"},
                new Team{Name="Alpine Ski House"},
                new Team{Name="Blue Yonder Airlines"},
                new Team{Name="Coho Vineyard"},
                new Team{Name="Contoso, Ltd."},
                new Team{Name="Fabrikam, Inc."},
                new Team{Name="Lucerne Publishing"},
                new Team{Name="Northwind Traders"},
                new Team{Name="Consolidated Messenger"},
                new Team{Name="Fourth Coffee"},
                new Team{Name="Graphic Design Institute"},
                new Team{Name="Nod Publishers"}
            };

            Team.PlayGames(teams);

            teams.ForEach(t => context.Teams.Add(t));
            context.SaveChanges();
        }
    }

    public class TeamConfiguration : DbConfiguration
    {
        public TeamConfiguration()
        {
            SetExecutionStrategy("System.Data.SqlClient", () => new SqlAzureExecutionStrategy());
        }
    }
    ```

1. В **обозревателе решений** дважды щелкните файл **Web.config**, чтобы открыть его.

    ![Web.config](./media/cache-web-app-cache-aside-leaderboard/cache-web-config.png)

1. Добавьте раздел `connectionStrings` в раздел `configuration`. Имя строки подключения должно соответствовать имени класса контекста базы данных Entity Framework `TeamContext`.

    Эта строка подключения подразумевает выполнение [предварительных требований](#prerequisites) и установку SQL Server Express LocalDB, которая является частью рабочей нагрузки *разработки настольных приложений .NET*, устанавливаемой вместе с Visual Studio 2019.

    ```xml
    <connectionStrings>
        <add name="TeamContext" connectionString="Data Source=(LocalDB)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\Teams.mdf;Integrated Security=True" providerName="System.Data.SqlClient" />
    </connectionStrings>
    ```

    В следующем примере показаны разделы `connectionStrings` и `configSections` в разделе `configuration`.

    ```xml
    <configuration>
        <configSections>
        <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
        </configSections>
        <connectionStrings>
        <add name="TeamContext" connectionString="Data Source=(LocalDB)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\Teams.mdf;Integrated Security=True"     providerName="System.Data.SqlClient" />
        </connectionStrings>
        ...
    ```

### <a name="add-the-teamscontroller-and-views"></a>Добавление TeamsController и представлений

1. Создайте проект в Visual Studio. 

1. В **обозревателе решений** щелкните правой кнопкой мыши папку **Контроллеры**, а затем выберите **Добавить** и **Контроллер**.

1. Выберите **Контроллер MVC 5 с представлениями, использующий Entity Framework** и нажмите кнопку **Добавить**. Если после нажатия кнопки **Добавить** появилась ошибка, убедитесь, что вы сначала скомпилировали проект.

    ![Добавление класса контроллера](./media/cache-web-app-cache-aside-leaderboard/cache-add-controller-class.png)

1. В раскрывающемся списке **Класс модели** выберите **Team (ContosoTeamStats.Models)** . В раскрывающемся списке **Класс контекста данных** выберите **TeamContext (ContosoTeamStats.Models)** . Введите `TeamsController` в текстовое поле **Имя контроллера** (если оно не будет заполнено автоматически). Нажмите кнопку **Добавить** , чтобы создать класс контроллера и добавить представления по умолчанию.

    ![Настройка контроллера](./media/cache-web-app-cache-aside-leaderboard/cache-configure-controller.png)

1. В **обозревателе решений** разверните **Global.asax** и дважды щелкните файл **Global.asax.cs**, чтобы открыть его.

    ![Global.asax.cs](./media/cache-web-app-cache-aside-leaderboard/cache-global-asax.png)

1. В начале файла добавьте следующие два оператора `using` после остальных операторов `using`.

    ```csharp
    using System.Data.Entity;
    using ContosoTeamStats.Models;
    ```

1. В конце метода `Application_Start` добавьте следующую строку кода:

    ```csharp
    Database.SetInitializer<TeamContext>(new TeamInitializer());
    ```

1. В **обозревателе решений** разверните `App_Start` и дважды щелкните `RouteConfig.cs`.

    ![RouteConfig.cs.](./media/cache-web-app-cache-aside-leaderboard/cache-RouteConfig-cs.png)

1. В методе `RegisterRoutes` замените `controller = "Home"` в маршруте `Default` на `controller = "Teams"`, как показано в следующем примере:

    ```csharp
    routes.MapRoute(
        name: "Default",
        url: "{controller}/{action}/{id}",
        defaults: new { controller = "Teams", action = "Index", id = UrlParameter.Optional }
    );
    ```

### <a name="configure-the-layout-view"></a>Настройка представления макета

1. В **обозревателе решений** разверните папку **Представления**, а затем — папку **Общее** и дважды щелкните файл **_Layout.cshtml**. 

    ![_Layout.cshtml](./media/cache-web-app-cache-aside-leaderboard/cache-layout-cshtml.png)

1. Измените содержимое элемента `title` и замените `My ASP.NET Application` на `Contoso Team Stats`, как показано в следующем примере.

    ```html
    <title>@ViewBag.Title - Contoso Team Stats</title>
    ```

1. В разделе `body` добавьте следующую инструкцию `Html.ActionLink` для *Contoso Team Stats* под ссылкой для *тестирования кэша Azure для Redis*.

    ```csharp
    @Html.ActionLink("Contoso Team Stats", "Index", "Teams", new { area = "" }, new { @class = "navbar-brand" })`
    ```

    ![Изменения в коде](./media/cache-web-app-cache-aside-leaderboard/cache-layout-cshtml-code.png)

1. Нажмите клавиши **CTRL+F5** , чтобы создать и запустить приложение. В этой версии приложения результаты считываются непосредственно из базы данных. Примечание. Действия **Создать**, **Изменить**, **Сведения** и **Удалить** автоматически добавлены в приложение с помощью шаблона **Контроллер MVC 5 с представлениями, использующий Entity Framework**. В следующем разделе руководства будет добавлен кэш Azure для Redis для оптимизации доступа к данным и добавления дополнительных функций в приложение.

    ![Начальное приложение](./media/cache-web-app-cache-aside-leaderboard/cache-starter-application.png)

## <a name="configure-the-app-for-azure-cache-for-redis"></a>Настройка приложения для кэша Azure для Redis

В этом разделе руководства будет настроен пример приложения для хранения и извлечения статистики команды Contoso из экземпляра кэша Azure для Redis с помощью клиента кэша [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis).

### <a name="add-a-cache-connection-to-the-teams-controller"></a>Добавление подключения кэша в контроллер команд

Вы уже установили пакет клиентской библиотеки *StackExchange.Redis* в кратком руководстве. Также вы уже настроили параметр приложения *CacheConnection* для использования локально и с опубликованной службой приложений. Используйте ту же клиентскую библиотеку и сведения *CacheConnection* в *TeamsController*.

1. В **обозревателе решений** разверните папку **Контроллеры** и дважды щелкните файл **TeamsController.cs**, чтобы открыть его.

    ![Контроллер команд](./media/cache-web-app-cache-aside-leaderboard/cache-teamscontroller.png)

1. Добавьте в файл **TeamsController.cs** следующие два оператора `using`:

    ```csharp
    using System.Configuration;
    using StackExchange.Redis;
    ```

1. Добавьте в класс `TeamsController` следующие два свойства:

    ```csharp
    // Redis Connection string info
    private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
    {
        string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
        return ConnectionMultiplexer.Connect(cacheConnection);
    });

    public static ConnectionMultiplexer Connection
    {
        get
        {
            return lazyConnection.Value;
        }
    }
    ```

### <a name="update-the-teamscontroller-to-read-from-the-cache-or-the-database"></a>Обновление класса TeamsController для чтения из кэша или базы данных

В этом примере статистику команды можно получить из базы данных или из кэша. Статистика команды сохраняется в кэше в качестве сериализованного элемента `List<Team>`, а также в качестве отсортированного набора с помощью типов данных Redis. Из отсортированного набора можно извлечь все или некоторые элементы или же запросить определенные элементы. В этом примере из отсортированного набора будут запрашиваться 5 лучших команд, упорядоченных по количеству положительных результатов.

Чтобы использовать кэш Azure для Redis, необязательно хранить статистику команды в нескольких форматах в кэше. В этом руководстве несколько форматов используется для демонстрации различных способов кэширования данных и различных типов данных, используемых для этой операции.

1. В начале файла `TeamsController.cs` добавьте следующие операторы `using` к остальным операторам `using`.

    ```csharp
    using System.Diagnostics;
    using Newtonsoft.Json;
    ```

1. Замените текущую реализацию метода `public ActionResult Index()` следующей реализацией.

    ```csharp
    // GET: Teams
    public ActionResult Index(string actionType, string resultType)
    {
        List<Team> teams = null;

        switch(actionType)
        {
            case "playGames": // Play a new season of games.
                PlayGames();
                break;

            case "clearCache": // Clear the results from the cache.
                ClearCachedTeams();
                break;

            case "rebuildDB": // Rebuild the database with sample data.
                RebuildDB();
                break;
        }

        // Measure the time it takes to retrieve the results.
        Stopwatch sw = Stopwatch.StartNew();

        switch(resultType)
        {
            case "teamsSortedSet": // Retrieve teams from sorted set.
                teams = GetFromSortedSet();
                break;

            case "teamsSortedSetTop5": // Retrieve the top 5 teams from the sorted set.
                teams = GetFromSortedSetTop5();
                break;

            case "teamsList": // Retrieve teams from the cached List<Team>.
                teams = GetFromList();
                break;

            case "fromDB": // Retrieve results from the database.
            default:
                teams = GetFromDB();
                break;
        }

        sw.Stop();
        double ms = sw.ElapsedTicks / (Stopwatch.Frequency / (1000.0));

        // Add the elapsed time of the operation to the ViewBag.msg.
        ViewBag.msg += " MS: " + ms.ToString();

        return View(teams);
    }
    ```

1. Добавьте приведенные ниже три метода из оператора switch, добавленного в предыдущем фрагменте кода, в класс `TeamsController` для реализации типов действий `playGames`, `clearCache` и `rebuildDB`.

    Метод `PlayGames` обновляет статистику команды, имитируя сезон игр, сохраняет результаты в базу данных и удаляет устаревшие данные из кэша.

    ```csharp
    void PlayGames()
    {
        ViewBag.msg += "Updating team statistics. ";
        // Play a "season" of games.
        var teams = from t in db.Teams
                    select t;

        Team.PlayGames(teams);

        db.SaveChanges();

        // Clear any cached results
        ClearCachedTeams();
    }
    ```

    Метод `RebuildDB` повторно инициализирует базу данных с набором команд по умолчанию, создает для них статистику и удаляет устаревшие данные из кэша.

    ```csharp
    void RebuildDB()
    {
        ViewBag.msg += "Rebuilding DB. ";
        // Delete and re-initialize the database with sample data.
        db.Database.Delete();
        db.Database.Initialize(true);

        // Clear any cached results
        ClearCachedTeams();
    }
    ```

    Метод `ClearCachedTeams` удаляет всю кэшированную статистику команды из кэша.

    ```csharp
    void ClearCachedTeams()
    {
        IDatabase cache = Connection.GetDatabase();
        cache.KeyDelete("teamsList");
        cache.KeyDelete("teamsSortedSet");
        ViewBag.msg += "Team data removed from cache. ";
    }
    ```

1. Добавьте приведенные ниже четыре метода в класс `TeamsController` , чтобы реализовать различные способы получения статистики команды из кэша и базы данных. Каждый из этих методов возвращает элемент `List<Team>`, который затем отображается в представлении.

    Метод `GetFromDB` считывает статистику команды из базы данных.
    ```csharp
    List<Team> GetFromDB()
    {
        ViewBag.msg += "Results read from DB. ";
        var results = from t in db.Teams
            orderby t.Wins descending
            select t;

        return results.ToList<Team>();
    }
    ```

    Метод `GetFromList` считывает статистику команды из кэша в качестве сериализованного элемента `List<Team>`. Если в кэше отсутствует статистика, происходит промах кэша. В случае промаха кэша статистика команды считывается из базы данных и сохраняется в кэше для следующего запроса. В этом примере сериализуются объекты .NET из кэша и в кэш с использованием сериализации JSON.NET. Дополнительные сведения см. в разделе [Работа с объектами .NET в кэше](cache-dotnet-how-to-use-azure-redis-cache.md#work-with-net-objects-in-the-cache).

    ```csharp
    List<Team> GetFromList()
    {
        List<Team> teams = null;

        IDatabase cache = Connection.GetDatabase();
        string serializedTeams = cache.StringGet("teamsList");
        if (!String.IsNullOrEmpty(serializedTeams))
        {
            teams = JsonConvert.DeserializeObject<List<Team>>(serializedTeams);

            ViewBag.msg += "List read from cache. ";
        }
        else
        {
            ViewBag.msg += "Teams list cache miss. ";
            // Get from database and store in cache
            teams = GetFromDB();

            ViewBag.msg += "Storing results to cache. ";
            cache.StringSet("teamsList", JsonConvert.SerializeObject(teams));
        }
        return teams;
    }
    ```

    Метод `GetFromSortedSet` считывает статистику команды из кэшированного отсортированного набора. В случае промаха кэша статистика команды считывается из базы данных и сохраняется в кэше в качестве отсортированного набора.

    ```csharp
    List<Team> GetFromSortedSet()
    {
        List<Team> teams = null;
        IDatabase cache = Connection.GetDatabase();
        // If the key teamsSortedSet is not present, this method returns a 0 length collection.
        var teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", order: Order.Descending);
        if (teamsSortedSet.Count() > 0)
        {
            ViewBag.msg += "Reading sorted set from cache. ";
            teams = new List<Team>();
            foreach (var t in teamsSortedSet)
            {
                Team tt = JsonConvert.DeserializeObject<Team>(t.Element);
                teams.Add(tt);
            }
        }
        else
        {
            ViewBag.msg += "Teams sorted set cache miss. ";

            // Read from DB
            teams = GetFromDB();

            ViewBag.msg += "Storing results to cache. ";
            foreach (var t in teams)
            {
                Console.WriteLine("Adding to sorted set: {0} - {1}", t.Name, t.Wins);
                cache.SortedSetAdd("teamsSortedSet", JsonConvert.SerializeObject(t), t.Wins);
            }
        }
        return teams;
    }
    ```

    Метод `GetFromSortedSetTop5` считывает пять лучших команд из кэшированного отсортированного набора. Сначала проверяется наличие ключа `teamsSortedSet` в кэше. Если этот ключ не указан, вызывается метод `GetFromSortedSet` для считывания статистики команды и ее сохранения в кэше. Затем в кэшированном отсортированном наборе запрашивается пять первых команд, которые возвращаются.

    ```csharp
    List<Team> GetFromSortedSetTop5()
    {
        List<Team> teams = null;
        IDatabase cache = Connection.GetDatabase();

        // If the key teamsSortedSet is not present, this method returns a 0 length collection.
        var teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", stop: 4, order: Order.Descending);
        if(teamsSortedSet.Count() == 0)
        {
            // Load the entire sorted set into the cache.
            GetFromSortedSet();

            // Retrieve the top 5 teams.
            teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", stop: 4, order: Order.Descending);
        }

        ViewBag.msg += "Retrieving top 5 teams from cache. ";
        // Get the top 5 teams from the sorted set
        teams = new List<Team>();
        foreach (var team in teamsSortedSet)
        {
            teams.Add(JsonConvert.DeserializeObject<Team>(team.Element));
        }
        return teams;
    }
    ```

### <a name="update-the-create-edit-and-delete-methods-to-work-with-the-cache"></a>Обновление методов создания, изменения и удаления для работы с кэшем

Код для формирования шаблонов, созданный в этом примере, содержит методы для добавления, изменения и удаления команд. При каждом добавлении, изменении или удалении команды данные в кэше устаревают. В этом разделе приведенные три метода будут изменены, чтобы очистить кэшированные команды. Это поможет обновить кэш.

1. Перейдите к методу `Create(Team team)` в классе `TeamsController`. Добавьте вызов в метод `ClearCachedTeams`, как показано в следующем примере.

    ```csharp
    // POST: Teams/Create
    // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
    // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Create([Bind(Include = "ID,Name,Wins,Losses,Ties")] Team team)
    {
        if (ModelState.IsValid)
        {
            db.Teams.Add(team);
            db.SaveChanges();
            // When a team is added, the cache is out of date.
            // Clear the cached teams.
            ClearCachedTeams();
            return RedirectToAction("Index");
        }

        return View(team);
    }
    ```

2. Перейдите к методу `Edit(Team team)` в классе `TeamsController`. Добавьте вызов в метод `ClearCachedTeams`, как показано в следующем примере.

    ```csharp
    // POST: Teams/Edit/5
    // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
    // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Edit([Bind(Include = "ID,Name,Wins,Losses,Ties")] Team team)
    {
        if (ModelState.IsValid)
        {
            db.Entry(team).State = EntityState.Modified;
            db.SaveChanges();
            // When a team is edited, the cache is out of date.
            // Clear the cached teams.
            ClearCachedTeams();
            return RedirectToAction("Index");
        }
        return View(team);
    }
    ```

3. Перейдите к методу `DeleteConfirmed(int id)` в классе `TeamsController`. Добавьте вызов в метод `ClearCachedTeams`, как показано в следующем примере.

    ```csharp
    // POST: Teams/Delete/5
    [HttpPost, ActionName("Delete")]
    [ValidateAntiForgeryToken]
    public ActionResult DeleteConfirmed(int id)
    {
        Team team = db.Teams.Find(id);
        db.Teams.Remove(team);
        db.SaveChanges();
        // When a team is deleted, the cache is out of date.
        // Clear the cached teams.
        ClearCachedTeams();
        return RedirectToAction("Index");
    }
    ```

### <a name="add-caching-methods-to-the-teams-index-view"></a>Добавление методов кэширования в представление индекса команд

1. В **обозревателе решений** разверните папку **Представления**, а затем — папку **Команды** и дважды щелкните файл **Index.cshtml**.

    ![Index.cshtml](./media/cache-web-app-cache-aside-leaderboard/cache-views-teams-index-cshtml.png)

1. Найдите следующий элемент абзаца в начале файла.

    ![Таблица действий](./media/cache-web-app-cache-aside-leaderboard/cache-teams-index-table.png)

    Эта ссылка создает новую команду. Замените элемент абзаца таблицей, приведенной ниже. Эта таблица содержит ссылки на действия для создания новой команды, воспроизведения нового сезона игр, очистки кэша, получения команды из кэша в нескольких форматах, получения команд из базы данных и повторной сборки базы данных с использованием нового примера данных.

    ```html
    <table class="table">
        <tr>
            <td>
                @Html.ActionLink("Create New", "Create")
            </td>
            <td>
                @Html.ActionLink("Play Season", "Index", new { actionType = "playGames" })
            </td>
            <td>
                @Html.ActionLink("Clear Cache", "Index", new { actionType = "clearCache" })
            </td>
            <td>
                @Html.ActionLink("List from Cache", "Index", new { resultType = "teamsList" })
            </td>
            <td>
                @Html.ActionLink("Sorted Set from Cache", "Index", new { resultType = "teamsSortedSet" })
            </td>
            <td>
                @Html.ActionLink("Top 5 Teams from Cache", "Index", new { resultType = "teamsSortedSetTop5" })
            </td>
            <td>
                @Html.ActionLink("Load from DB", "Index", new { resultType = "fromDB" })
            </td>
            <td>
                @Html.ActionLink("Rebuild DB", "Index", new { actionType = "rebuildDB" })
            </td>
        </tr>
    </table>
    ```

1. Перейдите в конец файла **Index.cshtml** и добавьте элемент `tr` в последнюю строку последней таблицы.

    ```html
    <tr><td colspan="5">@ViewBag.Msg</td></tr>
    ```
    В этой строке отображаются значение `ViewBag.Msg`, которое содержит отчет о состоянии о текущей операции. Значение `ViewBag.Msg` устанавливается при выборе ссылки на любое действия предыдущего шага.

    ![Сообщение о состоянии](./media/cache-web-app-cache-aside-leaderboard/cache-status-message.png)

1. Нажмите клавишу **F6** , чтобы скомпилировать проект.

## <a name="run-the-app-locally"></a>Локальный запуск приложения

Запустите приложение локально на компьютере для проверки функциональности, которая была добавлена для поддержки команд.

В этом тесте приложение и база данных выполняются локально. Однако кэш Azure для Redis удаленно размещается в Azure. Таким образом, производительность кэша может быть ниже производительности базы данных. Чтобы повысить производительность, следует разместить клиентское приложение и экземпляр кэша Azure для Redis в одном расположении. В следующем разделе все ресурсы развертываются в Azure, чтобы достичь улучшенной производительности при использовании кэша.

Чтобы запустить приложение на локальном компьютере:

1. Для запуска приложения нажмите сочетание клавиш **Ctrl+F5** .

    ![Приложение, выполняющееся локально](./media/cache-web-app-cache-aside-leaderboard/cache-local-application.png)

1. Проверьте все новые методы, которые были добавлены в представление. Так как кэш является удаленным в этих тестах, база данных может превосходить кэш по производительности.

## <a name="publish-and-run-in-azure"></a>Публикация и выполнение в Azure

### <a name="provision-a-database-for-the-app"></a>Подготовка базы данных для приложения

В этом разделе выполняется подготовка новой базы данных SQL, которую приложение будет использовать при размещении в Azure.

1. На [портале Azure](https://portal.azure.com/) в верхнем левом углу щелкните **Создать ресурс**.

1. На странице **Создать** щелкните **Базы данных** > **База данных SQL**.

1. Используйте следующие параметры для новой базы данных SQL:

   | Параметр       | Рекомендуемое значение | Описание |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Имя базы данных** | *ContosoTeamsDatabase* | Допустимые имена баз данных см. в статье об [идентификаторах базы данных](/sql/relational-databases/databases/database-identifiers). |
   | **Подписка** | *Ваша подписка*  | Выберите ту же подписку, которая использовалась для создания кэша и размещения службы приложений. |
   | **Группа ресурсов**  | *TestResourceGroup* | Щелкните **Использовать существующий** и используйте ту же группу ресурсов, где был расположен ваш кэш и служба приложений. |
   | **Выбрать источник** | **Пустая база данных** | Начните с пустой базы данных. |

1. В разделе **Сервер** щелкните **Настроить обязательные параметры** > **Создать сервер** и укажите следующие сведения, а затем нажмите кнопку **Выбор**:

   | Параметр       | Рекомендуемое значение | Описание |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Имя сервера** | Любое глобально уникальное имя | Допустимые имена серверов см. в статье о [правилах и ограничениях именования](/azure/architecture/best-practices/resource-naming). |
   | **Имя для входа администратора сервера** | Любое допустимое имя | Допустимые имена входа см. в статье об [идентификаторах базы данных](/sql/relational-databases/databases/database-identifiers). |
   | **Пароль** | Любой допустимый пароль | Длина пароля должна составлять минимум 8 символов. Пароль должен содержать символы трех категорий из перечисленных: прописные буквы, строчные буквы, цифры и символы, не являющиеся буквами или цифрами. |
   | **Расположение** | *Восточная часть США* | Выберите тот же регион, в котором вы создали кэш и службу приложений. |

1. Щелкните **Закрепить на панели мониторинга**, а затем **Создать**, чтобы создать базу данных и сервер.

1. После создания новой базы данных щелкните **Показать строки подключения к базам данных** и скопируйте строку подключения **ADO.NET**.

    ![Показать строки подключения](./media/cache-web-app-cache-aside-leaderboard/cache-show-connection-strings.png)

1. Перейдите к службе приложения на портале Azure и щелкните **Параметры приложения**, а затем в разделе "Строки подключения" щелкните **Добавить строку подключения**.

1. Добавьте новую строку подключения с именем *TeamContext* для сопоставления с классом контекста базы данных Entity Framework. Вставьте строку подключения для новой базы данных в качестве значения. Обязательно замените следующие заполнители в строке подключения и щелкните **Сохранить**:

    | Заполнитель | Рекомендуемое значение |
    | --- | --- |
    | *{your_username}* | Используйте **имя входа администратора сервера** для сервера, который вы только что создали. |
    | *{your_password}* | Используйте пароль для сервера, который вы только что создали. |

    Если добавить имя пользователя и пароль как параметр приложения, они не будут включены в коде. Такой подход позволяет защитить эти учетные данные.

### <a name="publish-the-application-updates-to-azure"></a>Публикация обновлений приложения в Azure

На этом шаге руководства обновления приложения будут опубликованы в Azure и запущены в облаке.

1. В Visual Studio щелкните правой кнопкой мыши проект **ContosoTeamStats** и выберите пункт **Опубликовать**.

    ![Публикация](./media/cache-web-app-cache-aside-leaderboard/cache-publish-app.png)

2. Щелкните **Опубликовать**, чтобы использовать тот же профиль публикации, который вы создали в кратком руководстве.

3. После завершения публикации Visual Studio запустит приложение в веб-браузере по умолчанию.

    ![Кэш добавлен](./media/cache-web-app-cache-aside-leaderboard/cache-added-to-application.png)

    В следующей таблице описана каждая ссылка на действие из примера приложения.

    | Действие | Описание |
    | --- | --- |
    | Создать |Создание команды. |
    | Воспроизвести сезон |Воспроизведение сезона игр, обновление статистики команды и удаление всех устаревших данных из кэша команды. |
    | Очистить кэш |Удаление статистики команды из кэша. |
    | List from Cache (Перечислить из кэша) |Получение статистики команды из кэша. В случае промаха кэша статистика загружается из базы данных и сохраняется в кэше для последующего использования. |
    | Sorted Set from Cache (Отсортированный набор из кэша) |Получение статистики команды из кэша с использованием отсортированного набора. В случае промаха кэша статистика загружается из базы данных и сохраняется в кэше с использованием отсортированного набора. |
    | Top 5 Teams from Cache (5 лучших команд из кэша) |Получение 5 лучших команд из кэша с использованием отсортированного набора. В случае промаха кэша статистика загружается из базы данных и сохраняется в кэше с использованием отсортированного набора. |
    | Load from DB (Загрузить из базы данных) |Получение статистики команды из базы данных. |
    | Rebuild DB (Перестроить базу данных) |Повторная сборка базы данных и ее перезагрузка с использованием примера данных команды. |
    | Изменить; Сведения; Удалить |Изменение команды, просмотр сведений о команде, удаление команды. |

Выполните некоторые действия и попробуйте получить данные из различных источников. Обратите внимание на разницу во времени при получении данных из базы данных и кэша разными способами.

## <a name="clean-up-resources"></a>Очистка ресурсов

По окончании работы с примером приложения в руководстве можно удалить использованные ресурсы Azure, чтобы сократить затраты и сэкономить ресурсы. Все ресурсы должны находиться в одной группе ресурсов. В таком случае их можно удалить вместе в рамках одной операции, удалив группу ресурсов. В инструкциях в этой статье использовалась группа ресурсов с именем *TestResources*.

> [!IMPORTANT]
> Удаление группы ресурсов — необратимая операция, и все соответствующие ресурсы удаляются окончательно. Будьте внимательны, чтобы случайно не удалить не ту группу ресурсов или не те ресурсы. Если ресурсы для размещения этого примера созданы в имеющейся группе ресурсов, содержащей ресурсы, которые следует сохранить, можно удалить каждый ресурс отдельно в соответствующих колонках.
>

1. Войдите на [портал Azure](https://portal.azure.com) и щелкните **Группы ресурсов**.
2. Введите имя группы ресурсов в текстовое поле **Фильтровать элементы…** .
3. Щелкните **...** справа от группы ресурсов и выберите **Удалить группу ресурсов**.

    ![DELETE](./media/cache-web-app-cache-aside-leaderboard/cache-delete-resource-group.png)

4. Подтвердите операцию удаления группы ресурсов. Введите имя группы ресурсов и нажмите кнопку **Удалить**.

    Через некоторое время группа ресурсов и все ее ресурсы будут удалены.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Масштабирование кэша Azure для Redis](./cache-how-to-scale.md)