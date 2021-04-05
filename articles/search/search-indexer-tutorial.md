---
title: Руководство по C#. Индексирование данных SQL Azure
titleSuffix: Azure Cognitive Search
description: В этом учебнике по C# описано, как подключить Базу данных SQL Azure, извлечь доступные для поиска данные и отправить их в индекс в службе "Когнитивный поиск Azure".
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 01/23/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: e2ca5f42120661b887d07e697596f41cb7a7fce4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98745772"
---
# <a name="tutorial-index-azure-sql-data-using-the-net-sdk"></a>Руководство по Индексирование данных SQL Azure с помощью пакета SDK для .NET

Вы можете настроить [индексатор](search-indexer-overview.md) извлекать из Базы данных Azure SQL данные, доступные для поиска, и отправлять их для создания индекса поиска в службу "Когнитивный поиск Azure". 

В этом учебнике используется C# и пакет [SDK для .NET](/dotnet/api/overview/azure/search) для выполнения следующих задач:

> [!div class="checklist"]
> * создание источника данных, который подключается к Базе данных SQL Azure;
> * Создание индексатора
> * выполнение индексатора для загрузки данных в индекс;
> * обращение к индексу для проверки.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

+ [База данных SQL Azure](https://azure.microsoft.com/services/sql-database/)
+ [Visual Studio](https://visualstudio.microsoft.com/downloads/)
+ [Создайте службу поиска](search-create-service-portal.md) или [найдите существующую службу](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) 

> [!Note]
> Для выполнения инструкций из этого руководства вы можете использовать бесплатную версию службы. В бесплатной версии вы можете использовать не более трех индексов, трех индексаторов и трех источников данных. В этом руководстве создается по одному объекту из каждой категории. Перед началом работы убедитесь, что у службы есть достаточно места, чтобы принять новые ресурсы.

## <a name="download-files"></a>Загрузка файлов

Исходный код для этого руководства размещен в папке [DotNetHowToIndexer](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToIndexers) репозитория [Azure-Samples/search-dotnet-getting-started](https://github.com/Azure-Samples/search-dotnet-getting-started) на GitHub.

## <a name="1---create-services"></a>1\. Создание служб

В этом руководстве используются Когнитивный поиск Azure для индексирования и отправки запросов, а также база данных SQL — в качестве внешнего источника данных. Желательно создать обе службы в одном регионе и группе ресурсов, чтобы упростить взаимодействие и управление. На практике база данных SQL Azure может находиться в любом регионе.

### <a name="start-with-azure-sql-database"></a>Начните с базы данных SQL Azure

На этом этапе в Базе данных SQL Azure создается внешний источник данных, который может сканироваться индексатором. Чтобы создать набор данных в Базе данных Azure SQL, вы можете использовать на портале Azure файл *hotels.sql* из скачанного образца. В службе "Когнитивный поиск Azure" используются плоские наборы строк. Такие наборы обычно создаются на основе представления или запроса. При помощи файла SQL в примере решения создается и заполняется одна таблица.

Если у вас уже есть ресурс в Базе данных SQL Azure, можно добавить в него таблицу отелей, сразу перейдя к шагу 4.

1. [Войдите на портал Azure](https://portal.azure.com/).

1. Щелкните или создайте **Базу данных SQL**. Вы можете использовать значения по умолчанию и самую низкую ценовую категорию. Одним из преимуществ создания сервера является то, что вы можете указать имя пользователя и пароль администратора, которые потребуются для создания и загрузки таблиц в дальнейшем.

   :::image type="content" source="media/search-indexer-tutorial/indexer-new-sqldb.png" alt-text="Страница создания базы данных" border="false":::

1. Щелкните **Проверить и создать**, чтобы развернуть новый сервер и базу данных. Дождитесь, пока завершится развертывание сервера и базы данных.

1. В области навигации щелкните **Редактор запроса (предварительная версия)**, а затем введите имя пользователя и пароль администратора сервера. 

   Если доступ запрещен, скопируйте IP-адрес клиента из сообщения об ошибке, а затем щелкните ссылку **Настройка брандмауэра для сервера**, чтобы добавить правило, которое разрешает доступ с клиентского компьютера по определенному IP-адресу. Новое правило может вступить в силу через несколько минут.

1. В редакторе запросов выберите действие **Открыть запрос** и перейдите к файлу *hotels.sql* на локальном компьютере. 

1. Выберите файл и нажмите **Открыть**. Скрипт должен выглядеть, как на следующем снимке экрана:

   :::image type="content" source="media/search-indexer-tutorial/sql-script.png" alt-text="Скрипт SQL" border="false":::

1. Нажмите **Выполнить** для выполнения запроса. В области результатов появится сообщение об успешном выполнении запроса для 3 строк.

1. Чтобы получить набор строк из этой таблицы, можно выполнить следующий запрос для проверки:

    ```sql
    SELECT * FROM Hotels
    ```

1. Скопируйте строку подключения ADO.NET к базе данных. Щелкните **Параметры** > **Строки подключения** и скопируйте строку подключения ADO.NET, как показано в примере ниже.

    ```sql
    Server=tcp:{your_dbname}.database.windows.net,1433;Initial Catalog=hotels-db;Persist Security Info=False;User ID={your_username};Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
    ```

Эта строка подключения потребуется в следующем упражнении при настройке среды.

### <a name="azure-cognitive-search"></a>Когнитивный поиск Azure

Следующий компонент — Когнитивный поиск Azure, который вы можете [создать на портале](search-create-service-portal.md). Для выполнения действий в этом пошаговом руководстве можно использовать уровень "Бесплатный". 

### <a name="get-an-admin-api-key-and-url-for-azure-cognitive-search"></a>Получение ключа API и URL-адреса конечной точки для администрирования Когнитивного поиска Azure

Для вызова API требуется URL-адрес службы и ключ доступа. Служба поиска создана с обоими элементами, поэтому если вы добавили службу "Когнитивный поиск Azure" в подписку, выполните следующие действия для получения необходимых сведений:

1. [Войдите на портал Azure](https://portal.azure.com/) и на странице **обзора** службы поиска получите URL-адрес. Пример конечной точки может выглядеть так: `https://mydemo.search.windows.net`.

1. В разделе **Параметры** > **Ключи** получите ключ администратора, чтобы обрести полные права на службу. Существуют два взаимозаменяемых ключа администратора, предназначенных для обеспечения непрерывности бизнес-процессов на случай, если вам потребуется сменить один из них. Вы можете использовать первичный или вторичный ключ для выполнения запросов на добавление, изменение и удаление объектов.

   :::image type="content" source="media/search-get-started-rest/get-url-key.png" alt-text="Получение конечной точки HTTP и ключа доступа" border="false":::

## <a name="2---set-up-your-environment"></a>2\. Настройка среды

1. Запустите Visual Studio и откройте файл **DotNetHowToIndexers.sln**.

1. В обозревателе решений откройте файл **appsettings.json**, чтобы предоставить сведения о подключении.

1. Если полный URL-адрес на странице с общими сведениями службы для `SearchServiceEndPoint` имеет значение "https://my-demo-service.search.windows.net", укажите этот URL-адрес.

1. Для `AzureSqlConnectionString` формат строки аналогичен следующему: `"Server=tcp:{your_dbname}.database.windows.net,1433;Initial Catalog=hotels-db;Persist Security Info=False;User ID={your_username};Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"`

    ```json
    {
      "SearchServiceEndPoint": "<placeholder-search-url>",
      "SearchServiceAdminApiKey": "<placeholder-admin-key-for-search-service>",
      "AzureSqlConnectionString": "<placeholder-ADO.NET-connection-string",
    }
    ```

1. В строке подключения убедитесь, что она содержит допустимый пароль. Имя базы данных и пользователя копируются автоматически, а пароль необходимо вводить вручную.

## <a name="3---create-the-pipeline"></a>3\. Создание конвейера

Для индексаторов требуется объект источника данных и индекс. Нужный нам код находится в двух файлах:

  + **hotel.cs** содержит схему, которая определяет индекс;
  + **Program.cs** содержит функции для создания структур и управления ими в службе.

### <a name="in-hotelcs"></a>hotell.cs

Схема индексов определяет коллекцию полей, включая атрибуты, в которых указаны разрешенные операции. Например, в них указывается, поддерживает ли схема полнотекстовый поиск, фильтрацию или сортировку, как в представленном ниже определении для HotelName. [SearchableField](/dotnet/api/azure.search.documents.indexes.models.searchablefield) — это полнотекстовые данные с возможностью поиска по определению. Другие атрибуты назначаются явно.

```csharp
. . . 
[SearchableField(IsFilterable = true, IsSortable = true)]
[JsonPropertyName("hotelName")]
public string HotelName { get; set; }
. . .
```

Кроме того, схема может включать профили повышения для оценки результатов поиска, пользовательские анализаторы и другие конструкции. Но для наших целей схема определена не так подробно. Она содержит только поля, обнаруженные в примерах наборов данных.

### <a name="in-programcs"></a>Program.cs

Основная программа содержит логику для создания [клиента индексатора](/dotnet/api/azure.search.documents.indexes.models.searchindexer), индекса, источника данных и индексатора. При помощи этого кода проверяется наличие ресурсов с таким же именем. При обнаружении такие ресурсы удаляются, так как есть вероятность, что вы будете запускать эту программу несколько раз.

Объект источника данных настраивается с помощью параметров, относящихся к ресурсам Базы данных Azure SQL, включая [частичное или добавочное индексирование](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md#capture-new-changed-and-deleted-rows) для использования встроенных [функций обнаружения изменений](/sql/relational-databases/track-changes/about-change-tracking-sql-server) SQL Azure. Исходная демонстрационная база данных гостиниц в SQL Azure содержит столбец "обратимого удаления", **IsDeleted**. Если этот столбец имеет значение true в базе данных, индексатор удаляет соответствующий документ из индекса Когнитивного поиска Azure.

```csharp
Console.WriteLine("Creating data source...");

var dataSource =
      new SearchIndexerDataSourceConnection(
         "hotels-sql-ds",
         SearchIndexerDataSourceType.AzureSql,
         configuration["AzureSQLConnectionString"],
         new SearchIndexerDataContainer("hotels"));

indexerClient.CreateOrUpdateDataSourceConnection(dataSource);
```

Объект индексатора не зависит от платформы. Его настройка, планирование и вызов одинаковы независимо от источника. Этот пример индексатора включает в себя расписание, возможность сброса, которая очищает журнал индексатора и вызывает метод для создания и немедленного запуска индексатора. Чтобы создать или обновить индексатор, используйте [CreateOrUpdateIndexerAsync](/dotnet/api/azure.search.documents.indexes.searchindexerclient.createorupdateindexerasync).

```csharp
Console.WriteLine("Creating Azure SQL indexer...");

var schedule = new IndexingSchedule(TimeSpan.FromDays(1))
{
      StartTime = DateTimeOffset.Now
};

var parameters = new IndexingParameters()
{
      BatchSize = 100,
      MaxFailedItems = 0,
      MaxFailedItemsPerBatch = 0
};

// Indexer declarations require a data source and search index.
// Common optional properties include a schedule, parameters, and field mappings
// The field mappings below are redundant due to how the Hotel class is defined, but 
// we included them anyway to show the syntax 
var indexer = new SearchIndexer("hotels-sql-idxr", dataSource.Name, searchIndex.Name)
{
      Description = "Data indexer",
      Schedule = schedule,
      Parameters = parameters,
      FieldMappings =
      {
         new FieldMapping("_id") {TargetFieldName = "HotelId"},
         new FieldMapping("Amenities") {TargetFieldName = "Tags"}
      }
};

await indexerClient.CreateOrUpdateIndexerAsync(indexer);
```

Запуски индексаторов обычно выполняются по расписанию, но если при разработке вам потребуется немедленно запустить индексатор, используйте [RunIndexerAsync](/dotnet/api/azure.search.documents.indexes.searchindexerclient.runindexerasync).

```csharp
Console.WriteLine("Running Azure SQL indexer...");

try
{
      await indexerClient.RunIndexerAsync(indexer.Name);
}
catch (CloudException e) when (e.Response.StatusCode == (HttpStatusCode)429)
{
      Console.WriteLine("Failed to run indexer: {0}", e.Response.Content);
}
```

## <a name="4---build-the-solution"></a>4. Сборка решения

Нажмите клавишу F5 для сборки и запуска решения. Программа выполняется в режиме отладки. В окне консоли отображаются сведения о состоянии каждой операции.

   :::image type="content" source="media/search-indexer-tutorial/console-output.png" alt-text="Выходные данные консоли" border="false":::

Код выполняется локально в Visual Studio. Устанавливается подключение к службе поиска в Azure, которая в свою очередь подключается к Базе данных SQL Azure, чтобы получить набор данных. При таком большом количестве операций есть несколько возможных точек сбоя. Если поступает сообщение об ошибке, прежде всего проверьте следующее:

+ Выполните поиск данных подключения к службе, которые вы указали в полном URL-адресе. Если вы ввели только имя службы, выполнение операций прекращается на создании индекса и отображается сообщение о сбое подключения.

+ Сведения о подключении к базе данных в **appsettings.json**. Это должна быть строка подключения ADO.NET, полученная на портале, в которую добавлены допустимые для базы данных имя пользователя и пароль. Учетная запись пользователя должна предоставлять разрешение на получение данных. Для IP-адреса локального клиента должен быть разрешен входящий доступ через брандмауэр.

+ Ограничения ресурсов. Помните, что для уровня "Бесплатный" действуют ограничения в три индекса, индексатора и источника данных. В службе с максимальным количеством объектов нельзя создавать новые.

## <a name="5---search"></a>5\. Поиск

Проверьте создание объекта на портале Azure и создайте запрос по индексу с помощью **обозревателя поиска**.

1. [Войдите на портал Azure](https://portal.azure.com/), откройте страницу **Обзор** для службы поиска и поочередно откройте каждый список, чтобы убедиться в том, что объект создан. В списках **Индексы**, **Индексаторы** и **Источники данных** должны присутствовать элементы hotels, azure-sql-indexer и azure-sql соответственно.

   :::image type="content" source="media/search-indexer-tutorial/tiles-portal.png" alt-text="Плитки для индексаторов и источников данных" border="false":::

1. Выберите индекс hotels. На странице индекса hotels на первой вкладке вы найдете **обозреватель поиска**. 

1. Щелкните **Поиск**, чтобы отправить пустой запрос. 

   В индексе возвращаются три записи в виде документов JSON. Проводник поиска возвращает документы в формате JSON, чтобы можно было просматривать всю структуру.

   :::image type="content" source="media/search-indexer-tutorial/portal-search.png" alt-text="Запрос индекса" border="false":::
   
1. Затем введите строку поиска `search=river&$count=true`. 

   При помощи этого запроса вызывается полнотекстовый поиск по слову `river`. В результате выводится количество соответствующих документов. Количество соответствующих документов полезно знать при тестировании, когда используется большой индекс с тысячами или миллионами документов. В этом случае только один документ соответствует запросу.

1. Наконец, введите строку поиска, которая ограничивает выходные данные JSON до требуемых полей: `search=river&$count=true&$select=hotelId, baseRate, description`. 

   Ответ на запрос сокращается до выбранных полей, обеспечивая более точный результат.

## <a name="reset-and-rerun"></a>Сброс и повторный запуск

На ранних экспериментальных этапах разработки самый практичный подход к итерации схемы — удалить все объекты из службы "Когнитивный поиск Azure" и восстановить их с помощью кода. Имена ресурсов являются уникальными. Удаление объекта позволяет воссоздать его с использованием того же имени.

Пример кода для этого учебника проверяет имеющиеся объекты и удаляет их, чтобы вы могли повторно выполнить код.

Для удаления индексов, индексаторов и источников данных вы также можете использовать портал.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы работаете в своей подписке, после завершения проекта целесообразно удалить созданные ресурсы, которые вам больше не потребуются. Работающие ресурсы могут означать лишние затраты. Можно удалить отдельные ресурсы или удалить группу ресурсов, что позволит удалить весь набор ресурсов.

Просматривать ресурсы и управлять ими можно на портале с помощью ссылок "Все ресурсы" или "Группы ресурсов" в области навигации слева.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы знакомы с основами индексирования Базы данных SQL, давайте подробнее рассмотрим конфигурацию индексатора.

> [!div class="nextstepaction"]
> [Настройка индексатора Базы данных SQL](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)