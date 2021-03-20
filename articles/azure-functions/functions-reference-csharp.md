---
title: Справочник разработчика скриптов C# по функциям Azure
description: Узнайте, как разрабатывать функции Azure с помощью скрипта C#.
author: craigshoemaker
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 12/12/2017
ms.author: cshoe
ms.openlocfilehash: 48614640660da6d85face5ea416d267fa9f59515
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92164845"
---
# <a name="azure-functions-c-script-csx-developer-reference"></a>Справочник разработчика скрипта C# (CSX) по решению "Функции Azure"

<!-- When updating this article, make corresponding changes to any duplicate content in functions-dotnet-class-library.md -->

В этой статье содержатся общие сведения о разработке функций Azure с помощью скрипта C# (*CSX*).

Решение "Функции Azure" поддерживает языки программирования C# и скрипт C#. Рекомендации по [использованию C# в проекте библиотеки классов Visual Studio](functions-develop-vs.md) см. в [справочнике разработчика C#](functions-dotnet-class-library.md).

В этой статье предполагается, что вы уже прочли [руководство для разработчиков по решению "Функции Azure"](functions-reference.md).

## <a name="how-csx-works"></a>Как работает формат CSX

Интерфейс скрипта C# для решения "Функции Azure" основан на [пакете SDK для веб-заданий Azure](https://github.com/Azure/azure-webjobs-sdk/wiki/Introduction). Данные поступают в функцию C# через аргументы метода. Имена аргументов указываются в файле `function.json`. Есть предварительно определенные имена для доступа к таким объектам, как средство ведения журнала функций и маркеры отмены.

Формат *CSX* позволяет писать меньше стандартного кода и сосредоточиться на написании только функции C#. а затем вместо помещения всего кода в пространство имен и класс просто определите метод `Run`. Как обычно, укажите ссылки на необходимые сборки и пространства имен в начале файла,

Файлы *CSX* приложения-функции компилируются при инициализации экземпляра. Этот этап компиляции означает, что такие операции, как холодный запуск, могут занимать больше времени при использовании функций скрипта C#, по сравнению с библиотеками классов C#. Кроме того, благодаря этому этапу компиляции функции скрипта C# доступны для редактирования на портале Azure, в отличие от библиотек классов C#.

## <a name="folder-structure"></a>Структура папок

Структура папок для проекта на C# выглядит следующим образом.

```
FunctionsProject
 | - MyFirstFunction
 | | - run.csx
 | | - function.json
 | | - function.proj
 | - MySecondFunction
 | | - run.csx
 | | - function.json
 | | - function.proj
 | - host.json
 | - extensions.csproj
 | - bin
```

Существует общий файл [host.json](functions-host-json.md), который можно использовать для настройки приложения-функции. У каждой функции есть собственный файл кода (.csx) и файлом конфигурации привязки (function.json).

Расширения привязки, необходимые в [версии 2.x и более поздних](functions-versions.md) среды выполнения функций, определены в файле `extensions.csproj` с фактическими файлами библиотеки в папке `bin`. При локальной разработке необходимо [зарегистрировать расширения привязки](./functions-bindings-register.md#extension-bundles). При разработке функций на портале Azure эта регистрация выполняется автоматически.

## <a name="binding-to-arguments"></a>Привязка к аргументам

Входные или выходные данные привязываются к параметру функции скрипта C# прпи помощи свойства `name` в файле конфигурации *function.json*. В следующем примере представлены файлы *function.json* и *run.csx* для функции, срабатывающей по запросу. Параметр для получения данных из сообщения очереди называется `myQueueItem`, так как это значение свойства `name`.

```json
{
    "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        }
    ]
}
```

```csharp
#r "Microsoft.WindowsAzure.Storage"

using Microsoft.Extensions.Logging;
using Microsoft.WindowsAzure.Storage.Queue;
using System;

public static void Run(CloudQueueMessage myQueueItem, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem.AsString}");
}
```

Оператор `#r` описан [далее в этой статье](#referencing-external-assemblies).

## <a name="supported-types-for-bindings"></a>Поддерживаемые типы для привязок

Для каждой привязки есть собственные поддерживаемые типы. Например, триггер больших двоичных объектов можно использовать со строковым параметром, параметрами POCO, `CloudBlockBlob` или любым из нескольких других поддерживаемых типов. В [справочной статье о привязках для больших двоичных объектов](functions-bindings-storage-blob-trigger.md#usage) содержится список всех поддерживаемых типов параметров для триггеров больших двоичных объектов. Дополнительные сведения см. в статье о [триггерах и привязках](functions-triggers-bindings.md) и в [справочной документации по каждому типу привязки](functions-triggers-bindings.md#next-steps).

[!INCLUDE [HTTP client best practices](../../includes/functions-http-client-best-practices.md)]

## <a name="referencing-custom-classes"></a>Ссылки на пользовательские классы

Если нужно использовать пользовательский класс простых старых объектов CLR (POCO), вы можете включить определении класса в тот же файл или поместить его в отдельный файл.

В следующем примере показан пример файла *run.csx*, содержащего определение класса POCO.

```csharp
public static void Run(string myBlob, out MyClass myQueueItem)
{
    log.Verbose($"C# Blob trigger function processed: {myBlob}");
    myQueueItem = new MyClass() { Id = "myid" };
}

public class MyClass
{
    public string Id { get; set; }
}
```

Для каждого свойства класса POCO нужно определить методы получения и задания.

## <a name="reusing-csx-code"></a>Повторное использование кода CSX

В файле *run.csx* можно использовать классы и методы, определенные в других *CSX*-файлах. Для этого используйте директивы `#load` в файле *run.csx*. В следующем примере к процедуре ведения журнала с именем `MyLogger` предоставляется общий доступ в файле *myLogger.csx*, а также она загружается в файл *run.csx*. Для этого используется директива `#load`:

Пример *run.csx*:

```csharp
#load "mylogger.csx"

using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"Log by run.csx: {DateTime.Now}");
    MyLogger(log, $"Log by MyLogger: {DateTime.Now}");
}
```

Пример *mylogger.csx*:

```csharp
public static void MyLogger(ILogger log, string logtext)
{
    log.LogInformation(logtext);
}
```

Обычно для задания строго типизированных данных, передаваемых между функциями с использованием объекта POCO, применяется общий файл с расширением *CSX*. В указанном ниже упрощенном примере триггер HTTP и очереди совместно используют объект POCO с именем `Order` для задания строго типизированных данных о заказах.

Пример файла *run.csx* для триггера HTTP:

```cs
#load "..\shared\order.csx"

using System.Net;
using Microsoft.Extensions.Logging;

public static async Task<HttpResponseMessage> Run(Order req, IAsyncCollector<Order> outputQueueItem, ILogger log)
{
    log.LogInformation("C# HTTP trigger function received an order.");
    log.LogInformation(req.ToString());
    log.LogInformation("Submitting to processing queue.");

    if (req.orderId == null)
    {
        return new HttpResponseMessage(HttpStatusCode.BadRequest);
    }
    else
    {
        await outputQueueItem.AddAsync(req);
        return new HttpResponseMessage(HttpStatusCode.OK);
    }
}
```

Пример файла *run.csx* для триггера очереди:

```cs
#load "..\shared\order.csx"

using System;
using Microsoft.Extensions.Logging;

public static void Run(Order myQueueItem, out Order outputQueueItem, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed order...");
    log.LogInformation(myQueueItem.ToString());

    outputQueueItem = myQueueItem;
}
```

Пример файла *order.csx*:

```cs
public class Order
{
    public string orderId {get; set; }
    public string custName {get; set;}
    public string custAddress {get; set;}
    public string custEmail {get; set;}
    public string cartId {get; set; }

    public override String ToString()
    {
        return "\n{\n\torderId : " + orderId +
                  "\n\tcustName : " + custName +
                  "\n\tcustAddress : " + custAddress +
                  "\n\tcustEmail : " + custEmail +
                  "\n\tcartId : " + cartId + "\n}";
    }
}
```

В директиве `#load` можно использовать относительный путь:

* `#load "mylogger.csx"` загружает файл, расположенный в папке функции.
* `#load "loadedfiles\mylogger.csx"` загружает файл, расположенный в папке, которая содержится в папке функции.
* `#load "..\shared\mylogger.csx"` загружает файл, расположенный в папке на том же уровне, что и папка функции, то есть непосредственно в разделе *wwwroot*.

Директива `#load` работает только с *CSX*-файлами, но не с *CS*-файлами.

## <a name="binding-to-method-return-value"></a>Привязка к возвращаемому значению метода

Используйте возвращаемое значение метода для выходной привязки, указав имя `$return` в *function.json*. Примеры см. в статье о [триггерах и привязках](./functions-bindings-return-value.md).

Используйте возвращаемое значение, только если в результате успешного выполнения функции всегда возвращается значение для передачи в привязку для вывода. В противном случае используйте `ICollector` или `IAsyncCollector`, как указано в следующем разделе.

## <a name="writing-multiple-output-values"></a>Написание нескольких значений выходных данных

Чтобы записать несколько значений в привязку для вывода или если после успешного вызова функции не возвращается значение для передачи в привязку для вывода, используйте типы [`ICollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) или [`IAsyncCollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs). Эти типы представляют собой доступные только для записи коллекции, записываемые в выходную привязку по завершении метода.

В следующем примере записываются несколько сообщений очереди в ту же очередь с помощью `ICollector`:

```csharp
public static void Run(ICollector<string> myQueue, ILogger log)
{
    myQueue.Add("Hello");
    myQueue.Add("World!");
}
```

## <a name="logging"></a>Logging

Для записи выходных данных в потоковые журналы в C# включите аргумент с типом [ILogger](/dotnet/api/microsoft.extensions.logging.ilogger). Рекомендуем присвоить ему имя `log`. Не используйте `Console.Write` в Функциях Azure.

```csharp
public static void Run(string myBlob, ILogger log)
{
    log.LogInformation($"C# Blob trigger function processed: {myBlob}");
}
```

> [!NOTE]
> Дополнительные сведения о новой платформе ведения журналов, которую можно использовать вместо `TraceWriter` , см. в документации по [ILogger](functions-dotnet-class-library.md#ilogger) в руководстве разработчика библиотеки классов .NET.

### <a name="custom-metrics-logging"></a>Запись в журнал пользовательских метрик

`LogMetric` `ILogger` Для создания пользовательских метрик в Application Insights можно использовать метод расширения. Ниже приведен пример вызова метода.

```csharp
logger.LogMetric("TestMetric", 1234);
```

Этот пример кода действует так же, как вызов `TrackMetric` с использованием API Application Insights для .NET.

## <a name="async"></a>Асинхронный режим

Чтобы сделать функцию [асинхронной](/dotnet/csharp/programming-guide/concepts/async/), используйте ключевое слово `async` и верните объект `Task`.

```csharp
public async static Task ProcessQueueMessageAsync(
        string blobName,
        Stream blobInput,
        Stream blobOutput)
{
    await blobInput.CopyToAsync(blobOutput, 4096);
}
```

Вы не можете использовать параметры `out` в асинхронных функциях. Вместо этих параметров для выходных привязок используйте [возвращаемое значение функции](#binding-to-method-return-value) или [объект сборщика](#writing-multiple-output-values).

## <a name="cancellation-tokens"></a>Токены отмены

Функция может принимать параметр [CancellationToken](/dotnet/api/system.threading.cancellationtoken), который позволяет операционной системе передавать в ваш код сведения о том, что выполнение функции будет завершено. Это уведомление можно использовать для предотвращения ситуации, когда выполнение функции завершается неожиданно, оставляя данные в несогласованном состоянии.

В следующем примере показано, как проверить, не приближается ли завершение выполнения функции.

```csharp
using System;
using System.IO;
using System.Threading;

public static void Run(
    string inputText,
    TextWriter logger,
    CancellationToken token)
{
    for (int i = 0; i < 100; i++)
    {
        if (token.IsCancellationRequested)
        {
            logger.WriteLine("Function was cancelled at iteration {0}", i);
            break;
        }
        Thread.Sleep(5000);
        logger.WriteLine("Normal processing for queue message={0}", inputText);
    }
}
```

## <a name="importing-namespaces"></a>Импорт пространств имен

Если требуется импортировать пространства имен, это можно сделать как обычно — с помощью предложения `using` .

```csharp
using System.Net;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

public static Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger log)
```

Следующие пространства имен импортируются автоматически и поэтому являются необязательными:

* `System`
* `System.Collections.Generic`
* `System.IO`
* `System.Linq`
* `System.Net.Http`
* `System.Threading.Tasks`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`

## <a name="referencing-external-assemblies"></a>Ссылки на внешние сборки

Для сборок платформы добавьте ссылки с помощью директивы `#r "AssemblyName"` .

```csharp
#r "System.Web.Http"

using System.Net;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

public static Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger log)
```

Следующие сборки автоматически добавляются средой внешнего размещения Функций Azure:

* `mscorlib`
* `System`
* `System.Core`
* `System.Xml`
* `System.Net.Http`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`
* `Microsoft.Azure.WebJobs.Extensions`
* `System.Web.Http`
* `System.Net.Http.Formatting`

К следующим сборкам можно обращаться по простому имени (например, `#r "AssemblyName"`):

* `Newtonsoft.Json`
* `Microsoft.WindowsAzure.Storage`
* `Microsoft.ServiceBus`
* `Microsoft.AspNet.WebHooks.Receivers`
* `Microsoft.AspNet.WebHooks.Common`
* `Microsoft.Azure.NotificationHubs`

## <a name="referencing-custom-assemblies"></a>Ссылки на пользовательские сборки

Чтобы указать ссылку на пользовательские сборки, можно использовать *общую* или *закрытую* сборку.

* Общие сборки совместно используют все функции в приложении-функции. Чтобы добавить ссылку на пользовательскую сборку, отправьте ее в папку с именем `bin` в [корневом каталоге приложения-функции](functions-reference.md#folder-structure) (wwwroot).

* Закрытые сборки входят в контекст указанной функции и поддерживают загрузку неопубликованных приложений разных версий. Закрытые сборки необходимо отправить в папку `bin` в каталоге функции. Укажите ссылки на сборки, используя имя файла, например `#r "MyAssembly.dll"`.

Дополнительные сведения о передаче файлов в папку функции см. в разделе об [управлении пакетами](#using-nuget-packages).

### <a name="watched-directories"></a>Каталоги отслеживания

Каталог, содержащий файл сценария функции, автоматически отслеживает изменения в сборках. Чтобы отслеживать изменения сборки в других каталогах, добавьте их в список `watchDirectories` в [host.json](functions-host-json.md).

## <a name="using-nuget-packages"></a>Использование пакетов NuGet
Чтобы использовать пакеты NuGet в функции C# версии 2.x и более поздних, отправьте файл *function.proj* в папку соответствующей функции в файловой системе приложения-функции. Ниже приведен пример файла *function.proj*, который добавляет ссылку на *Microsoft.ProjectOxford.Face* версии *1.1.0*.

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>netstandard2.0</TargetFramework>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="Microsoft.ProjectOxford.Face" Version="1.1.0" />
    </ItemGroup>
</Project>
```

Чтобы использовать настраиваемые веб-каналы NuGet, укажите веб-канал в файле *Nuget.Config* в корне приложения-функции. Дополнительные сведения см. в статье [Configuring NuGet behavior](/nuget/consume-packages/configuring-nuget-behavior) (Настройка поведения NuGet).

> [!NOTE]
> В функциях C# версии 1.x ссылки на пакеты NuGet указываются в файле *project.json*, а не *function.proj*.

Для функций 1.x используйте вместо этого файл *project.json*. Ниже приведен пример файла *project.json*:

```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "Microsoft.ProjectOxford.Face": "1.1.0"
      }
    }
   }
}
```

### <a name="using-a-functionproj-file"></a>Использование файла function.proj

1. Откройте функцию на портале Azure. На вкладке "Журналы" отображаются выходные данные установки пакета.
2. Чтобы отправить файл *function.proj*, используйте один из методов, описанных в разделе [Как обновить файлы приложения-функции](functions-reference.md#fileupdate) статьи "Руководство для разработчиков по Функциям Azure".
3. После отправки файла *function.proj* в потоковом журнале функции отобразятся выходные данные, как в следующем примере:

```
2018-12-14T22:00:48.658 [Information] Restoring packages.
2018-12-14T22:00:48.681 [Information] Starting packages restore
2018-12-14T22:00:57.064 [Information] Restoring packages for D:\local\Temp\9e814101-fe35-42aa-ada5-f8435253eb83\function.proj...
2016-04-04T19:02:50.511 Restoring packages for D:\home\site\wwwroot\HttpTriggerCSharp1\function.proj...
2018-12-14T22:01:00.844 [Information] Installing Newtonsoft.Json 10.0.2.
2018-12-14T22:01:01.041 [Information] Installing Microsoft.ProjectOxford.Common.DotNetStandard 1.0.0.
2018-12-14T22:01:01.140 [Information] Installing Microsoft.ProjectOxford.Face.DotNetStandard 1.0.0.
2018-12-14T22:01:09.799 [Information] Restore completed in 5.79 sec for D:\local\Temp\9e814101-fe35-42aa-ada5-f8435253eb83\function.proj.
2018-12-14T22:01:10.905 [Information] Packages restored.
```

## <a name="environment-variables"></a>Переменные среды

Чтобы получить значение переменной среды или значение параметра приложения, используйте `System.Environment.GetEnvironmentVariable`, как показано в следующем примере кода.

```csharp
public static void Run(TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
    log.LogInformation(GetEnvironmentVariable("AzureWebJobsStorage"));
    log.LogInformation(GetEnvironmentVariable("WEBSITE_SITE_NAME"));
}

public static string GetEnvironmentVariable(string name)
{
    return name + ": " +
        System.Environment.GetEnvironmentVariable(name, EnvironmentVariableTarget.Process);
}
```

<a name="imperative-bindings"></a>

## <a name="binding-at-runtime"></a>Привязка во время выполнения

Для C# и других языков .NET можно использовать шаблон [императивной](https://en.wikipedia.org/wiki/Imperative_programming) привязки, которая отличается от [*декларативной*](https://en.wikipedia.org/wiki/Declarative_programming) привязки в файле *function.json*. Императивную привязку удобно использовать, когда параметры привязки должны вычисляться не при проектировании, а во время выполнения. С использованием такого шаблона можно моментально выполнить привязку к поддерживаемым входным и выходным привязкам в коде функции.

Определите принудительную привязку следующим образом.

- **Не** добавляйте запись для нужных императивных привязок в файл *function.json*.
- Передайте входной параметр [`Binder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Host/Bindings/Runtime/Binder.cs) или [`IBinder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IBinder.cs).
- Используйте следующий шаблон C# для привязки данных,

```cs
using (var output = await binder.BindAsync<T>(new BindingTypeAttribute(...)))
{
    ...
}
```

где `BindingTypeAttribute` — атрибут .NET, определяющий пользовательскую привязку, а `T` — входной или выходной тип, поддерживаемый этим типом привязки. `T` не может быть параметром типа `out` (например, `out JObject`). Например, выходная привязка таблицы мобильных приложений поддерживает [шесть выходных типов](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs#L17-L22), но можно использовать только [ICollector \<T> ](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) или [`IAsyncCollector<T>`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs) для `T` .

### <a name="single-attribute-example"></a>Пример с одним атрибутом

В следующем примере кода создается [выходная привязка большого двоичного объекта службы хранилища](functions-bindings-storage-blob-output.md) с путем к большому двоичному объекту, определенному во время выполнения, а затем записывается строка в большой двоичный объект.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host.Bindings.Runtime;

public static async Task Run(string input, Binder binder)
{
    using (var writer = await binder.BindAsync<TextWriter>(new BlobAttribute("samples-output/path")))
    {
        writer.Write("Hello World!!");
    }
}
```

[BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs) определяет входную или выходную привязку [большого двоичного объекта службы хранилища](functions-bindings-storage-blob.md), а [TextWriter](/dotnet/api/system.io.textwriter) представляет собой поддерживаемый тип выходной привязки.

### <a name="multiple-attribute-example"></a>Пример с несколькими атрибутами

В предыдущем примере код получает параметр приложения для строки подключения основной учетной записи хранения приложения-функции (т. е. `AzureWebJobsStorage`). Вы можете указать пользовательский параметр приложения, который следует использовать для учетной записи хранения. Для этого добавьте [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) и передайте массив атрибутов в `BindAsync<T>()`. Используйте параметр `Binder`, а не `IBinder`.  Пример:

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host.Bindings.Runtime;

public static async Task Run(string input, Binder binder)
{
    var attributes = new Attribute[]
    {
        new BlobAttribute("samples-output/path"),
        new StorageAccountAttribute("MyStorageAccount")
    };

    using (var writer = await binder.BindAsync<TextWriter>(attributes))
    {
        writer.Write("Hello World!");
    }
}
```

В следующей таблице перечислены атрибуты .NET для каждого типа привязки и пакеты, в которых они определены.

> [!div class="mx-codeBreakAll"]
> | Привязка | attribute | Ссылка, которую нужно добавить |
> |------|------|------|
> | Cosmos DB | [`Microsoft.Azure.WebJobs.DocumentDBAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.CosmosDB/CosmosDBAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.CosmosDB"` |
> | Центры событий | [`Microsoft.Azure.WebJobs.ServiceBus.EventHubAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/v2.x/src/Microsoft.Azure.WebJobs.ServiceBus/EventHubs/EventHubAttribute.cs), [`Microsoft.Azure.WebJobs.ServiceBusAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/b798412ad74ba97cf2d85487ae8479f277bdd85c/test/Microsoft.Azure.WebJobs.ServiceBus.UnitTests/ServiceBusAccountTests.cs) | `#r "Microsoft.Azure.Jobs.ServiceBus"` |
> | Мобильные приложения | [`Microsoft.Azure.WebJobs.MobileTableAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.MobileApps"` |
> | Центры уведомлений | [`Microsoft.Azure.WebJobs.NotificationHubAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.NotificationHubs/NotificationHubAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.NotificationHubs"` |
> | Служебная шина | [`Microsoft.Azure.WebJobs.ServiceBusAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/b798412ad74ba97cf2d85487ae8479f277bdd85c/test/Microsoft.Azure.WebJobs.ServiceBus.UnitTests/ServiceBusAttributeTests.cs), [`Microsoft.Azure.WebJobs.ServiceBusAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/b798412ad74ba97cf2d85487ae8479f277bdd85c/test/Microsoft.Azure.WebJobs.ServiceBus.UnitTests/ServiceBusAccountTests.cs) | `#r "Microsoft.Azure.WebJobs.ServiceBus"` |
> | Очередь службы хранилища | [`Microsoft.Azure.WebJobs.QueueAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/), [`Microsoft.Azure.WebJobs.StorageAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) | |
> | Большой двоичный объект службы хранилища | [`Microsoft.Azure.WebJobs.BlobAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs), [`Microsoft.Azure.WebJobs.StorageAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) | |
> | Таблица службы хранилища | [`Microsoft.Azure.WebJobs.TableAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs), [`Microsoft.Azure.WebJobs.StorageAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) | |
> | Twilio | [`Microsoft.Azure.WebJobs.TwilioSmsAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.Twilio/TwilioSMSAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.Twilio"` |

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Дополнительные сведения о триггерах и привязках](functions-triggers-bindings.md)

> [!div class="nextstepaction"]
> [Дополнительные сведения о рекомендациях по решению "Функции Azure"](functions-best-practices.md)
