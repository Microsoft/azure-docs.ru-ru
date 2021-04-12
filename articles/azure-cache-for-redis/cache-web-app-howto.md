---
title: Создание веб-приложения ASP.NET с использованием Кэша Azure для Redis
description: В этом кратком руководстве содержатся сведения о создании веб-приложения ASP.NET с помощью кэша Azure для Redis
author: yegu-ms
ms.service: cache
ms.topic: quickstart
ms.date: 09/29/2020
ms.author: yegu
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 19c54ad62e45ecf6e31b46d0291f61dca8e8d9b3
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104722469"
---
# <a name="quickstart-use-azure-cache-for-redis-with-an-aspnet-web-app"></a>Краткое руководство. Использование Кэша Azure для Redis с веб-приложением ASP.NET 

Из этого краткого руководства вы узнаете, как с помощью Visual Studio 2019 создать веб-приложение ASP.NET, которое подключается к кэшу Azure для Redis для хранения и извлечения данных из кэша. После этого вы можете развернуть приложение в Службе приложений Azure.

## <a name="skip-to-the-code-on-github"></a>Переход к коду на GitHub

Если вы хотите сразу перейти к коду, см. [краткое руководство по ASP.NET](https://github.com/Azure-Samples/azure-cache-redis-samples/tree/main/quickstart/aspnet) на сайте GitHub.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/dotnet).
- [Visual Studio 2019](https://www.visualstudio.com/downloads/) с рабочими нагрузками **ASP.NET и веб-разработка** и **Разработка Azure**.

## <a name="create-the-visual-studio-project"></a>Создание проекта Visual Studio

1. Откройте Visual Studio и выберите **Файл** > **Создать** > **Проект**.

2. В диалоговом окне **Создание проекта** выполните следующие действия:

    ![Создание проекта](./media/cache-web-app-howto/cache-create-project.png)

    а. В поле поиска введите _веб-приложение C# ASP.NET_.

    b. Выберите **Веб-приложение ASP.NET (.NET Framework)**.

    c. Выберите **Далее**.

3. В поле **Имя проекта** укажите имя проекта. В этом примере используется имя **ContosoTeamStats**.

4. Убедитесь, что выбран вариант **.NET Framework 4.6.1** или более поздняя версия.

5. Нажмите кнопку **создания**.
   
6. Выберите тип проекта **MVC**.

7. Убедитесь, что для параметра **Проверка подлинности** указано значение **Без проверки подлинности**. В зависимости от установленной версии Visual Studio значение **Проверка подлинности** по умолчанию может быть другим. Чтобы изменить его, щелкните **Изменить способ проверки подлинности** и выберите **Без проверки подлинности**.

8. Выберите **Создать**, чтобы создать проект.

## <a name="create-a-cache"></a>Создание кэша

Создайте кэш для приложения.

[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

[!INCLUDE [redis-cache-access-keys](../../includes/redis-cache-access-keys.md)]

#### <a name="to-edit-the-cachesecretsconfig-file"></a>Изменение файла *CacheSecrets.config*

1. Создайте на компьютере файл с именем *CacheSecrets.config*. Поместите его в расположение, где он не будет записываться после изменения с исходным кодом примера приложения. В этот кратком руководстве файл *CacheSecrets.config* расположен в папке *C:\AppSecrets\CacheSecrets.config*.

1. Измените файл *CacheSecrets.config*. Затем добавьте следующее содержимое:

    ```xml
    <appSettings>
        <add key="CacheConnection" value="<cache-name>.redis.cache.windows.net,abortConnect=false,ssl=true,allowAdmin=true,password=<access-key>"/>
    </appSettings>
    ```

1. Замените `<cache-name>` на имя узла кэша.

1. Замените `<access-key>` первичным ключом для кэша.

    > [!TIP]
    > При повторном создании первичного ключа доступа вторичный ключ доступа можно использовать во время смены ключей как дополнительный ключ.
   >
1. Сохраните файл.

## <a name="update-the-mvc-application"></a>Обновление приложения MVC

В этом разделе вы обновите приложение для поддержки нового представления, которое отображает простой тест кэша Azure для Redis.

* [Добавление в файл web.config параметра приложения для использования кэша](#update-the-webconfig-file-with-an-app-setting-for-the-cache)
* Настройка приложения для использования клиента StackExchange.Redis
* Обновление HomeController и макета
* Добавление нового представления RedisCache

### <a name="update-the-webconfig-file-with-an-app-setting-for-the-cache"></a>Добавление в файл web.config параметра приложения для использования кэша

При локальном запуске приложения сведения в файле *CacheSecrets.config* используются для подключения к экземпляру кэша Azure для Redis. Позже вы развернете это приложение в Azure. Сейчас вы настроите параметр приложения в Azure, который приложение использует вместо этого файла для получения сведений о соединении кэша. 

Так как файл *CacheSecrets.config* не развертывается в Azure с помощью приложения, он используется только при тестировании приложения локально. Храните эти сведения в надежном месте для предотвращения вредоносного доступа к данным кэша.

#### <a name="to-update-the-webconfig-file"></a>Обновление файла *web.config*
1. В **обозревателе решений** откройте файл *web.config*, щелкнув его дважды.

    ![Web.config](./media/cache-web-app-howto/cache-web-config.png)

2. В файле *web.config* найдите элемент `<appSetting>`. Теперь добавьте следующий атрибут `file`: Если использовалось другое имя файла или расположение, замените эти значения на представленные в примере.

* До: `<appSettings>`
* После: `<appSettings file="C:\AppSecrets\CacheSecrets.config">`

Среда выполнения ASP.NET объединяет содержимое внешнего файла с разметкой в элементе `<appSettings>`. Если указанный файл не удается найти, среда выполнения игнорирует атрибут файла. Секреты (строка подключения к вашему кэшу) не включаются в исходный код приложения. При развертывании веб-приложения в Azure файл *CacheSecrets.config* не развертывается.

### <a name="to-configure-the-application-to-use-stackexchangeredis"></a>Настройка приложения для использования StackExchange.Redis

1. Чтобы настроить приложение для использования пакета NuGet [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) для Visual Studio, выберите **Инструменты > Диспетчер пакетов NuGet > Консоль диспетчера пакетов**.

2. Выполните следующую команду в окне `Package Manager Console`:

    ```powershell
    Install-Package StackExchange.Redis
    ```

3. Пакет NuGet загружает и добавляет необходимые ссылки на сборки в клиентском приложении для доступа к кэшу Redis для Azure из клиента StackExchange.Azure кэша Redis. Если вы предпочитаете использовать версию клиентской библиотеки `StackExchange.Redis` со строгими именами, установите пакет `StackExchange.Redis.StrongName`.

### <a name="to-update-the-homecontroller-and-layout"></a>Обновление HomeController и макета

1. В **обозревателе решений** разверните папку **Контроллеры**, а затем откройте файл *HomeController.cs*.

2. Добавьте следующие инструкции `using` в начало файла.

    ```csharp
    using StackExchange.Redis;
    using System.Configuration;
    using System.Net.Sockets;
    using System.Text;
    using System.Threading;
    ```

3. Добавьте следующие элементы в класс `HomeController`, чтобы включить поддержку нового действия `RedisCache`, которое выполняет некоторые команды для нового кэша.

    ```csharp
    public ActionResult RedisCache()
    {
        ViewBag.Message = "A simple example with Azure Cache for Redis on ASP.NET.";

        IDatabase cache = GetDatabase();

        // Perform cache operations using the cache object...

        // Simple PING command
        ViewBag.command1 = "PING";
        ViewBag.command1Result = cache.Execute(ViewBag.command1).ToString();

        // Simple get and put of integral data types into the cache
        ViewBag.command2 = "GET Message";
        ViewBag.command2Result = cache.StringGet("Message").ToString();

        ViewBag.command3 = "SET Message \"Hello! The cache is working from ASP.NET!\"";
        ViewBag.command3Result = cache.StringSet("Message", "Hello! The cache is working from ASP.NET!").ToString();

        // Demonstrate "SET Message" executed as expected...
        ViewBag.command4 = "GET Message";
        ViewBag.command4Result = cache.StringGet("Message").ToString();

        // Get the client list, useful to see if connection list is growing...
        // Note that this requires allowAdmin=true in the connection string
        ViewBag.command5 = "CLIENT LIST";
        StringBuilder sb = new StringBuilder();
        var endpoint = (System.Net.DnsEndPoint)GetEndPoints()[0];
        IServer server = GetServer(endpoint.Host, endpoint.Port);
        ClientInfo[] clients = server.ClientList();

        sb.AppendLine("Cache response :");
        foreach (ClientInfo client in clients)
        {
            sb.AppendLine(client.Raw);
        }

        ViewBag.command5Result = sb.ToString();

        return View();
    }

    private static long lastReconnectTicks = DateTimeOffset.MinValue.UtcTicks;
    private static DateTimeOffset firstErrorTime = DateTimeOffset.MinValue;
    private static DateTimeOffset previousErrorTime = DateTimeOffset.MinValue;

    private static readonly object reconnectLock = new object();

    // In general, let StackExchange.Redis handle most reconnects,
    // so limit the frequency of how often ForceReconnect() will
    // actually reconnect.
    public static TimeSpan ReconnectMinFrequency => TimeSpan.FromSeconds(60);

    // If errors continue for longer than the below threshold, then the
    // multiplexer seems to not be reconnecting, so ForceReconnect() will
    // re-create the multiplexer.
    public static TimeSpan ReconnectErrorThreshold => TimeSpan.FromSeconds(30);

    public static int RetryMaxAttempts => 5;

    private static Lazy<ConnectionMultiplexer> lazyConnection = CreateConnection();

    public static ConnectionMultiplexer Connection
    {
        get
        {
            return lazyConnection.Value;
        }
    }

    private static Lazy<ConnectionMultiplexer> CreateConnection()
    {
        return new Lazy<ConnectionMultiplexer>(() =>
        {
            string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
            return ConnectionMultiplexer.Connect(cacheConnection);
        });
    }

    private static void CloseConnection(Lazy<ConnectionMultiplexer> oldConnection)
    {
        if (oldConnection == null)
            return;

        try
        {
            oldConnection.Value.Close();
        }
        catch (Exception)
        {
            // Example error condition: if accessing oldConnection.Value causes a connection attempt and that fails.
        }
    }

    /// <summary>
    /// Force a new ConnectionMultiplexer to be created.
    /// NOTES:
    ///     1. Users of the ConnectionMultiplexer MUST handle ObjectDisposedExceptions, which can now happen as a result of calling ForceReconnect().
    ///     2. Don't call ForceReconnect for Timeouts, just for RedisConnectionExceptions or SocketExceptions.
    ///     3. Call this method every time you see a connection exception. The code will:
    ///         a. wait to reconnect for at least the "ReconnectErrorThreshold" time of repeated errors before actually reconnecting
    ///         b. not reconnect more frequently than configured in "ReconnectMinFrequency"
    /// </summary>
    public static void ForceReconnect()
    {
        var utcNow = DateTimeOffset.UtcNow;
        long previousTicks = Interlocked.Read(ref lastReconnectTicks);
        var previousReconnectTime = new DateTimeOffset(previousTicks, TimeSpan.Zero);
        TimeSpan elapsedSinceLastReconnect = utcNow - previousReconnectTime;

        // If multiple threads call ForceReconnect at the same time, we only want to honor one of them.
        if (elapsedSinceLastReconnect < ReconnectMinFrequency)
            return;

        lock (reconnectLock)
        {
            utcNow = DateTimeOffset.UtcNow;
            elapsedSinceLastReconnect = utcNow - previousReconnectTime;

            if (firstErrorTime == DateTimeOffset.MinValue)
            {
                // We haven't seen an error since last reconnect, so set initial values.
                firstErrorTime = utcNow;
                previousErrorTime = utcNow;
                return;
            }

            if (elapsedSinceLastReconnect < ReconnectMinFrequency)
                return; // Some other thread made it through the check and the lock, so nothing to do.

            TimeSpan elapsedSinceFirstError = utcNow - firstErrorTime;
            TimeSpan elapsedSinceMostRecentError = utcNow - previousErrorTime;

            bool shouldReconnect =
                elapsedSinceFirstError >= ReconnectErrorThreshold // Make sure we gave the multiplexer enough time to reconnect on its own if it could.
                && elapsedSinceMostRecentError <= ReconnectErrorThreshold; // Make sure we aren't working on stale data (e.g. if there was a gap in errors, don't reconnect yet).

            // Update the previousErrorTime timestamp to be now (e.g. this reconnect request).
            previousErrorTime = utcNow;

            if (!shouldReconnect)
                return;

            firstErrorTime = DateTimeOffset.MinValue;
            previousErrorTime = DateTimeOffset.MinValue;

            Lazy<ConnectionMultiplexer> oldConnection = lazyConnection;
            CloseConnection(oldConnection);
            lazyConnection = CreateConnection();
            Interlocked.Exchange(ref lastReconnectTicks, utcNow.UtcTicks);
        }
    }

    // In real applications, consider using a framework such as
    // Polly to make it easier to customize the retry approach.
    private static T BasicRetry<T>(Func<T> func)
    {
        int reconnectRetry = 0;
        int disposedRetry = 0;

        while (true)
        {
            try
            {
                return func();
            }
            catch (Exception ex) when (ex is RedisConnectionException || ex is SocketException)
            {
                reconnectRetry++;
                if (reconnectRetry > RetryMaxAttempts)
                    throw;
                ForceReconnect();
            }
            catch (ObjectDisposedException)
            {
                disposedRetry++;
                if (disposedRetry > RetryMaxAttempts)
                    throw;
            }
        }
    }

    public static IDatabase GetDatabase()
    {
        return BasicRetry(() => Connection.GetDatabase());
    }

    public static System.Net.EndPoint[] GetEndPoints()
    {
        return BasicRetry(() => Connection.GetEndPoints());
    }

    public static IServer GetServer(string host, int port)
    {
        return BasicRetry(() => Connection.GetServer(host, port));
    }
    ```

4. В **обозревателе решений** разверните папку **Представления** > **Общие**. Затем откройте файл *_Layout.cshtml*.

    Замените
    
    ```csharp
    @Html.ActionLink("Application name", "Index", "Home", new { area = "" }, new { @class = "navbar-brand" })
    ```

    на:

    ```csharp
    @Html.ActionLink("Azure Cache for Redis Test", "RedisCache", "Home", new { area = "" }, new { @class = "navbar-brand" })
    ```

### <a name="to-add-a-new-rediscache-view"></a>Добавление нового представления RedisCache

1. В **обозревателе решений** разверните папку **Views** и щелкните правой кнопкой мыши папку **Home**. Выберите **Добавить** > **Представление...** .

2. В диалоговом окне **Добавить представление** введите **RedisCache** для имени представления. Нажмите кнопку **Добавить**.

3. Замените код в файле *RedisCache.cshtml* следующим кодом:

    ```csharp
    @{
        ViewBag.Title = "Azure Cache for Redis Test";
    }

    <h2>@ViewBag.Title.</h2>
    <h3>@ViewBag.Message</h3>
    <br /><br />
    <table border="1" cellpadding="10">
        <tr>
            <th>Command</th>
            <th>Result</th>
        </tr>
        <tr>
            <td>@ViewBag.command1</td>
            <td><pre>@ViewBag.command1Result</pre></td>
        </tr>
        <tr>
            <td>@ViewBag.command2</td>
            <td><pre>@ViewBag.command2Result</pre></td>
        </tr>
        <tr>
            <td>@ViewBag.command3</td>
            <td><pre>@ViewBag.command3Result</pre></td>
        </tr>
        <tr>
            <td>@ViewBag.command4</td>
            <td><pre>@ViewBag.command4Result</pre></td>
        </tr>
        <tr>
            <td>@ViewBag.command5</td>
            <td><pre>@ViewBag.command5Result</pre></td>
        </tr>
    </table>
    ```

## <a name="run-the-app-locally"></a>Локальный запуск приложения

По умолчанию проект размещает приложение локально в [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) для тестирования и отладки.

### <a name="to-run-the-app-locally"></a>Запуск приложения на локальном компьютере
1. В Visual Studio выберите **Отладка** > **Начать отладку**, чтобы создать и запустить приложение локально для тестирования и отладки.

2. В браузере на панели навигации выберите **Azure Cache for Redis Test** (Проверка кэша Azure для Redis).

3. В приведенном ниже примере видно, что ключ `Message` ранее содержал кэшированное значение, установленное через консоль кэша Azure для Redis на портале. Приложение обновило кэшированное значение. Кроме того, оно выполнило команды `PING` и `CLIENT LIST`.

    ![Выполнение простого теста локально](./media/cache-web-app-howto/cache-simple-test-complete-local.png)

## <a name="publish-and-run-in-azure"></a>Публикация и выполнение в Azure

После успешного локального тестирования приложения вы сможете развернуть приложение в Azure и запустить его в облаке.

### <a name="to-publish-the-app-to-azure"></a>Публикация приложения в Azure

1. В Visual Studio щелкните узел проекта правой кнопкой мыши в обозревателе решений. Затем щелкните **Опубликовать**.

    ![Публикация](./media/cache-web-app-howto/cache-publish-app.png)

2. Выберите **Служба приложений Microsoft Azure**, **Создать**, а затем щелкните **Опубликовать**.

    ![Публикация в службе приложений](./media/cache-web-app-howto/cache-publish-to-app-service.png)

3. В диалоговом окне **Создать службу приложений** внесите следующие изменения:

    | Параметр | Рекомендуемое значение | Описание |
    | ------- | :---------------: | ----------- |
    | **Имя приложения** | Используйте значение по умолчанию. | При развертывании приложения в Azure его имя соответствует имени узла для приложения. В имя можно добавить суффикс метки времени, если необходимо сделать его уникальным. |
    | **Подписка** | Выберите подписку Azure. | В этой подписке взимается плата за связанные операции размещения. Если у вас несколько подписок Azure, убедитесь, что выбрана необходимая подписка.|
    | **Группа ресурсов** | Используйте группу ресурсов, в которой создан кэш (например *TestResourceGroup*). | Группа ресурсов позволяет управлять всеми ресурсами как группой. Позже, если вы захотите удалить приложение, можно просто удалить всю группу. |
    | **План обслуживания приложения** | Выберите **Создать** и создайте план службы приложений с именем *TestingPlan*. <br />Используйте то же **расположение**, что и при создании кэша. <br />В качестве размера выберите **Бесплатный**. | План службы приложений определяет набор вычислительных ресурсов, на которых выполняется веб-приложение. |

    ![Диалоговое окно "Служба приложений"](./media/cache-web-app-howto/cache-create-app-service-dialog.png)

4. После настройки параметров службы приложений щелкните **Создать**.

5. Просмотрите состояние публикации в окне Visual Studio **Вывод**. После публикации приложения регистрируется его URL-адрес:

    ![Выходные данные публикации](./media/cache-web-app-howto/cache-publishing-output.png)

### <a name="add-the-app-setting-for-the-cache"></a>Добавление параметра приложения для кэша

После публикации нового приложения добавьте новый параметр приложения. Этот параметр используется для хранения сведений о подключении к кэшу. 

#### <a name="to-add-the-app-setting"></a>Добавление параметров приложения 

1. В строке поиска в верхней части портала Azure введите имя приложения, чтобы найти созданное приложение.

    ![Поиск приложения](./media/cache-web-app-howto/cache-find-app-service.png)

2. Добавьте параметр нового приложения с именем **CacheConnection**, которое приложение будет использовать для подключения к кэшу. Используйте значение, заданное для `CacheConnection` в файле *CacheSecrets.config*. Значение содержит имя узла и ключ доступа кэша.

    ![Добавление параметра приложения](./media/cache-web-app-howto/cache-add-app-setting.png)

### <a name="run-the-app-in-azure"></a>Запуск приложения в Azure

В браузере перейдите по URL-адресу приложения. URL-адрес отображается в результатах выполнения операции публикации в окне выходных данных в Visual Studio. Его также можно найти на портале Azure на странице обзора созданного приложения.

На панели навигации щелкните **Azure Cache for Redis Test** (Проверка кэша Azure для Redis), чтобы проверить доступ к кэшу.

![Выполнение простого теста в Azure](./media/cache-web-app-howto/cache-simple-test-complete-azure.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

При переходе к следующему руководству в нем можно использовать ресурсы, созданные в рамках этого руководства.

В противном случае, если вы закончите работу с примером приложения из краткого руководства, вы можете удалить ресурсы Azure, созданные в рамках этого руководства, чтобы избежать ненужных расходов. 

> [!IMPORTANT]
> Удаление группы ресурсов — процесс необратимый. Группа ресурсов и все содержащиеся в ней ресурсы удаляются без возможности восстановления. Будьте внимательны, чтобы случайно не удалить не ту группу ресурсов или не те ресурсы. Если ресурсы для размещения этого примера созданы в имеющейся группе ресурсов, содержащей ресурсы, которые следует сохранить, можно удалить каждый ресурс отдельно в соответствующих колонках вместо удаления группы ресурсов.

### <a name="to-delete-a-resource-group"></a>Удаление группы ресурсов

1. Войдите на [портал Azure](https://portal.azure.com) и щелкните **Группы ресурсов**.

2. Введите имя группы ресурсов в поле **Фильтровать по имени...** В инструкциях в этой статье использовалась группа ресурсов с именем *TestResources*. В своей группе ресурсов в списке результатов щелкните **...**, а затем выберите **Удалить группу ресурсов**.

    ![Удалить](./media/cache-web-app-howto/cache-delete-resource-group.png)

Подтвердите операцию удаления группы ресурсов. Введите имя группы ресурсов, которую необходимо удалить, и щелкните **Удалить**.

Через некоторое время группа ресурсов и все ее ресурсы будут удалены.

## <a name="next-steps"></a>Дальнейшие действия

В следующем руководстве вы будете использовать кэш Azure для Redis в более реалистичном сценарии для улучшения производительности приложения. Вы обновите это приложение, чтобы кэшировать результаты списка лидеров с помощью шаблона "Кэш на стороне" с ASP.NET и базой данных.

> [!div class="nextstepaction"]
> [Создание списка лидеров с применением шаблона "Кэш на стороне" в ASP.NET](cache-web-app-cache-aside-leaderboard.md)