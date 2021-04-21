---
title: Краткое руководство. Использование Кэша Azure для Redis с приложениями .NET Framework
description: В этом кратком руководстве объясняется, как обращаться к кэшу Azure для Redis из приложений .NET.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.devlang: dotnet
ms.topic: quickstart
ms.custom: devx-track-csharp, mvc
ms.date: 06/18/2020
ms.openlocfilehash: 71e973e359c21c9ec6a77de93b8b56dfa16da342
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739176"
---
# <a name="quickstart-use-azure-cache-for-redis-in-net-framework"></a>Краткое руководство. Использование Кэша Azure для Redis с приложениями .NET Framework

Из этого краткого руководства вы узнаете, как реализовать кэш Azure для Redis в приложении .NET Framework для обеспечения доступа к защищенному выделенному кэшу, к которому может обращаться любое приложение в Azure. Вы будете использовать клиент [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) с кодом C# в консольном приложении .NET.

## <a name="skip-to-the-code-on-github"></a>Переход к коду на GitHub

Если вы хотите сразу перейти к коду, см. [краткое руководство по .NET Framework](https://github.com/Azure-Samples/azure-cache-redis-samples/tree/main/quickstart/dotnet) на сайте GitHub.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
- [Visual Studio 2019](https://www.visualstudio.com/downloads/)
- [.NET Framework 4 или более поздней версии](https://www.microsoft.com/net/download/dotnet-framework-runtime) для работы клиента StackExchange.Redis.

## <a name="create-a-cache"></a>Создание кэша
[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

[!INCLUDE [redis-cache-access-keys](../../includes/redis-cache-access-keys.md)]

Создайте на компьютере файл с именем *CacheSecrets.config* и поместите его в расположение, которое не будет записано после изменения с исходным кодом примера приложения. В этот кратком руководстве файл *CacheSecrets.config* расположен в папке *C:\AppSecrets\CacheSecrets.config*.

Измените файл *CacheSecrets.config* и добавьте содержимое, приведенное ниже.

```xml
<appSettings>
    <add key="CacheConnection" value="<host-name>,abortConnect=false,ssl=true,allowAdmin=true,password=<access-key>"/>
</appSettings>
```

Замените `<host-name>` на имя узла кэша.

Замените `<access-key>` первичным ключом для кэша.


## <a name="create-a-console-app"></a>Создание консольного приложения

В Visual Studio выберите **Файл** > **Создать** > **Проект**.

Выберите **Консольное приложение (.NET Framework)** , а затем нажмите кнопку **Далее**. Введите имя в поле **Имя проекта** и убедитесь, что выбрана версия **.NET Framework 4.6.1** или выше, а затем нажмите кнопку **Создать**, чтобы создать консольное приложение.

<a name="configure-the-cache-clients"></a>

## <a name="configure-the-cache-client"></a>Настройка клиента кэша

В этом разделе вы настроите в консольном приложении использование клиента [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) для .NET.

В Visual Studio последовательно выберите **Инструменты** > **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов** и выполните следующую команду в открывшемся окне консоли менеджера пакетов.

```powershell
Install-Package StackExchange.Redis
```

Когда установка завершится, клиент кэша *StackExchange.Redis* станет доступным для использования в проекте.


## <a name="connect-to-the-cache"></a>Подключение к кэшу

В Visual Studio откройте файл *App.config* и включите в него атрибут `appSettings` `file`, который ссылается на файл *CacheSecrets.config*.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <startup> 
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.7.2" />
    </startup>

    <appSettings file="C:\AppSecrets\CacheSecrets.config"></appSettings>
</configuration>
```

В обозревателе решений щелкните правой кнопкой мыши **Ссылки**, а затем выберите **Добавить ссылку**. Добавьте ссылку на сборку **System.Configuration**.

Добавьте следующие инструкции `using` в файл *Program.cs*:

```csharp
using StackExchange.Redis;
using System.Configuration;
```

Подключение к кэшу Azure для Redis управляется с помощью класса `ConnectionMultiplexer`. Этот класс нужно сделать общим и использовать во всех разделах клиентского приложения. Не создавайте новое подключение для каждой операции. 

Никогда не храните учетные данные в исходном коде. Для простоты в этом примере используется только внешний файл конфигурации с секретными данными. Гораздо лучше было бы применить [Azure Key Vault с сертификатами](/rest/api/keyvault/certificate-scenarios).

В файле *Program.cs* добавьте следующие члены в класс `Program` консольного приложения:

```csharp
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
```


При таком подходе к совместному использованию экземпляра `ConnectionMultiplexer` в приложении нужно создать статическое свойство, которое возвращает подключенный экземпляр. Такой код потокобезопасно инициализирует только один подключенный экземпляр `ConnectionMultiplexer`. Для параметра `abortConnect` задано значение false. Это означает, что вызов завершается успешно, даже если подключение к кэшу Azure для Redis не установлено. Одна из ключевых особенностей `ConnectionMultiplexer` заключается в том, что этот параметр автоматически восстанавливает соединение с кэшем, когда устраняется проблема с сетью или другие проблемы.

Значение параметра *CacheConnection* в appSetting позволяет указать ссылку на строку подключения к кэшу через портал Azure в качестве параметра пароля.

## <a name="handle-redisconnectionexception-and-socketexception-by-reconnecting"></a>Обработка RedisConnectionException и SocketException путем повторного подключения

При вызове методов в `ConnectionMultiplexer` рекомендуется попытаться автоматически разрешить исключения `RedisConnectionException` и `SocketException`, закрыв и повторно установив подключение.

Добавьте следующие инструкции `using` в файл *Program.cs*:

```csharp
using System.Net.Sockets;
using System.Threading;
```

В файле *Program.cs* добавьте следующие элементы в класс `Program`:

```csharp
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

## <a name="executing-cache-commands"></a>Выполнение команд кэша

Добавьте следующий код в процедуру `Main` класса `Program` для консольного приложения:

```csharp
static void Main(string[] args)
{
    IDatabase cache = GetDatabase();

    // Perform cache operations using the cache object...

    // Simple PING command
    string cacheCommand = "PING";
    Console.WriteLine("\nCache command  : " + cacheCommand);
    Console.WriteLine("Cache response : " + cache.Execute(cacheCommand).ToString());

    // Simple get and put of integral data types into the cache
    cacheCommand = "GET Message";
    Console.WriteLine("\nCache command  : " + cacheCommand + " or StringGet()");
    Console.WriteLine("Cache response : " + cache.StringGet("Message").ToString());

    cacheCommand = "SET Message \"Hello! The cache is working from a .NET console app!\"";
    Console.WriteLine("\nCache command  : " + cacheCommand + " or StringSet()");
    Console.WriteLine("Cache response : " + cache.StringSet("Message", "Hello! The cache is working from a .NET console app!").ToString());

    // Demonstrate "SET Message" executed as expected...
    cacheCommand = "GET Message";
    Console.WriteLine("\nCache command  : " + cacheCommand + " or StringGet()");
    Console.WriteLine("Cache response : " + cache.StringGet("Message").ToString());

    // Get the client list, useful to see if connection list is growing...
    // Note that this requires allowAdmin=true in the connection string
    cacheCommand = "CLIENT LIST";
    Console.WriteLine("\nCache command  : " + cacheCommand);
    var endpoint = (System.Net.DnsEndPoint)GetEndPoints()[0];
    IServer server = GetServer(endpoint.Host, endpoint.Port);
    ClientInfo[] clients = server.ClientList();

    Console.WriteLine("Cache response :");
    foreach (ClientInfo client in clients)
    {
        Console.WriteLine(client.Raw);
    }

    CloseConnection(lazyConnection);
}
```

В кэше Azure для Redis настраивается число баз данных (по умолчанию 16) для обеспечения логического разделения данных в нем. Этот код подключается к стандартной базе данных DB 0. Дополнительные сведения см. в разделах [What are Redis databases?](cache-configure.md#default-redis-server-configuration) (Что такое базы данных Redis) и [Конфигурация сервера Redis по умолчанию](cache-development-faq.md#what-are-redis-databases).

Элементы можно сохранять в кэше и извлекать из него с помощью методов `StringSet` и `StringGet`.

Redis хранит большинство данных в строках Redis, но эти строки могут содержать данные многих типов, включая сериализованные двоичные данные, которые можно использовать при сохранении объектов .NET в кэше.

Нажмите сочетание клавиш **CTRL+F5**, чтобы скомпилировать и запустить консольное приложение.

В приведенном ниже примере видно, что ключ `Message` ранее содержал кэшированное значение, установленное через консоль Redis на портале Azure. Приложение обновило кэшированное значение. Кроме того, оно выполнило команды `PING` и `CLIENT LIST`.

![Частичное консольное приложение](./media/cache-dotnet-how-to-use-azure-redis-cache/cache-console-app-partial.png)


## <a name="work-with-net-objects-in-the-cache"></a>Работа с объектами .NET в кэше

Кэш Azure для Redis может кэшировать и .NET-объекты, и примитивные типы данных, но прежде чем объект .NET можно будет кэшировать, его нужно сериализовать. За сериализацию объекта .NET отвечает разработчик приложения, что дает ему свободу в выборе сериализатора.

Простой способ сериализовать объекты — использовать методы сериализации `JsonConvert` из [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/) и выполнять сериализацию в нотацию JSON и из нее. В этом разделе вы добавите в кэш объект .NET.

В Visual Studio последовательно выберите **Инструменты** > **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов** и выполните следующую команду в открывшемся окне консоли менеджера пакетов.

```powershell
Install-Package Newtonsoft.Json
```

Добавьте следующую инструкцию `using` в начало файла *Program.cs*:

```csharp
using Newtonsoft.Json;
```

Добавьте следующее определение класса `Employee` в файл *Program.cs*:

```csharp
class Employee
{
    public string Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }

    public Employee(string employeeId, string name, int age)
    {
        Id = employeeId;
        Name = name;
        Age = age;
    }
}
```

В нижней части процедуры `Main()` в файле *Program.cs* прямо перед вызовом `CloseConnection()` добавьте следующие строки кода, которые сохраняют и извлекают сериализованный объект .NET:

```csharp
    // Store .NET object to cache
    Employee e007 = new Employee("007", "Davide Columbo", 100);
    Console.WriteLine("Cache response from storing Employee .NET object : " + 
    cache.StringSet("e007", JsonConvert.SerializeObject(e007)));

    // Retrieve .NET object from cache
    Employee e007FromCache = JsonConvert.DeserializeObject<Employee>(cache.StringGet("e007"));
    Console.WriteLine("Deserialized Employee .NET object :\n");
    Console.WriteLine("\tEmployee.Name : " + e007FromCache.Name);
    Console.WriteLine("\tEmployee.Id   : " + e007FromCache.Id);
    Console.WriteLine("\tEmployee.Age  : " + e007FromCache.Age + "\n");
```

Нажмите сочетание клавиш **Ctrl+F5**, чтобы скомпилировать и запустить консольное приложение для проверки сериализации объектов .NET. 

![Готовое консольное приложение](./media/cache-dotnet-how-to-use-azure-redis-cache/cache-console-app-complete.png)


## <a name="clean-up-resources"></a>Очистка ресурсов

При переходе к следующему руководству в нем можно использовать ресурсы, созданные в этом руководстве.

В противном случае, если вы закончите работу с примером приложения из краткого руководства, вы можете удалить ресурсы Azure, созданные в текущем руководстве, чтобы избежать ненужных расходов. 

> [!IMPORTANT]
> Удаление группы ресурсов — необратимая операция, и все соответствующие ресурсы удаляются окончательно. Будьте внимательны, чтобы случайно не удалить не ту группу ресурсов или не те ресурсы. Если ресурсы для размещения этого примера созданы в имеющейся группе ресурсов, содержащей ресурсы, которые следует сохранить, можно удалить каждый ресурс отдельно в соответствующих колонках вместо удаления группы ресурсов.
>

Войдите на [портал Azure](https://portal.azure.com) и щелкните **Группы ресурсов**.

Введите имя группы ресурсов в текстовое поле **Фильтровать по имени...** . В инструкциях в этой статье использовалась группа ресурсов с именем *TestResources*. В своей группе ресурсов в списке результатов щелкните **...** , а затем **Удалить группу ресурсов**.

![DELETE](./media/cache-dotnet-how-to-use-azure-redis-cache/cache-delete-resource-group.png)

Подтвердите операцию удаления группы ресурсов. Введите имя группы ресурсов и нажмите кнопку **Удалить**.

Через некоторое время группа ресурсов и все ее ресурсы будут удалены.



<a name="next-steps"></a>

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как использовать кэш Azure для Redis в приложениях .NET. Переходите к следующему краткому руководству по использованию кэша Azure для Redis в веб-приложениях ASP.NET.

> [!div class="nextstepaction"]
> [Создание веб-приложения ASP.NET, в котором используется кэш Azure для Redis](./cache-web-app-howto.md)

Хотите оптимизировать и сократить ваши расходы на облако?

> [!div class="nextstepaction"]
> [Начните анализировать затраты с помощью службы "Управление затратами"](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
