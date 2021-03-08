---
title: Изолированное программное планирование .NET для .NET 5,0 в службе "функции Azure"
description: Узнайте, как использовать изолированный процесс .NET для выполнения функций C# в .NET 5,0 вне процесса в Azure.
ms.service: azure-functions
ms.topic: conceptual
ms.date: 03/01/2021
ms.custom: template-concept
ms.openlocfilehash: ab89c012c985afa8d7375ff94d0f55b0ea6941cc
ms.sourcegitcommit: f6193c2c6ce3b4db379c3f474fdbb40c6585553b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102449464"
---
# <a name="guide-for-running-functions-on-net-50-in-azure"></a>Инструкции по выполнению функций в .NET 5,0 в Azure

_Поддержка .NET 5,0 сейчас доступна в предварительной версии._

Эта статья содержит вводные сведения об использовании C# для разработки функций изолированного процесса .NET, которые выполняются вне процесса в функциях Azure. Выполнение вне процесса позволяет отделить код функции от среды выполнения функций Azure. Он также позволяет создавать и запускать функции, предназначенные для текущего выпуска .NET 5,0. 

Если вам не требуется поддержка .NET 5,0 или запуск функций вне процесса, вместо этого может потребоваться [разработка функций библиотеки классов C#](functions-dotnet-class-library.md).

## <a name="why-net-isolated-process"></a>Зачем работает изолированный процесс .NET?

Ранее функции Azure поддерживали только тесно интегрированный режим для функций .NET, которые выполняются [как библиотека классов](functions-dotnet-class-library.md) в том же процессе, что и узел. Этот режим обеспечивает глубокую интеграцию между ведущим процессом и функциями. Например, функции библиотеки классов .NET могут совместно использовать интерфейсы API привязки и типы. Однако для этой интеграции также требуется тесная связь между ведущим процессом и функцией .NET. Например, функции .NET, выполняемые в процессе, должны выполняться в той же версии .NET, что и среда выполнения функций. Чтобы разрешить выполнение вне этих ограничений, теперь можно выбрать запуск в изолированном процессе. Эта изоляция процессов также позволяет разрабатывать функции, использующие текущие выпуски .NET (например, .NET 5,0), которые изначально не поддерживаются средой выполнения функций.

Поскольку эти функции выполняются в отдельном процессе, между приложениями изолированных функций .NET и приложениями-функциями библиотеки классов .NET существуют [различия функций и функциональных возможностей](#differences-with-net-class-library-functions) .

### <a name="benefits-of-running-out-of-process"></a>Преимущества выполнения вне процесса

При выполнении вне процесса функции .NET могут воспользоваться следующими преимуществами: 

+ Меньше конфликтов. Поскольку функции выполняются в отдельном процессе, сборки, используемые в приложении, не будут конфликтовать с разными версиями тех же сборок, которые используются ведущим процессом.  
+ Полный контроль над процессом: вы управляете запуском приложения и можете управлять используемыми конфигурациями и начальным по по промежуточного слоя.
+ Внедрение зависимостей. Поскольку вы полностью контролируете процесс, вы можете использовать текущие поведения .NET для внедрения зависимостей и внедрять по промежуточного слоя в приложение-функцию. 

## <a name="supported-versions"></a>Поддерживаемые версии

Единственной версией .NET, которая сейчас поддерживается для выполнения вне процесса, является .NET 5,0.

## <a name="net-isolated-project"></a>Изолированный проект .NET

Проект изолированной функции .NET — это, по сути, проект консольного приложения .NET, предназначенный для .NET 5,0. Ниже приведены основные файлы, необходимые для любого изолированного проекта .NET.

+ [host.jsв](functions-host-json.md) файле.
+ [local.settings.jsв](functions-run-local.md#local-settings-file) файле.
+ Файл проекта C# (csproj), определяющий проект и зависимости.
+ Program.cs файл, который является точкой входа для приложения.

## <a name="package-references"></a>Ссылки на пакеты

При выполнении вне процесса проект .NET использует уникальный набор пакетов, которые реализуют как основные функциональные возможности, так и расширения привязки. 

### <a name="core-packages"></a>Основные пакеты 

Для выполнения функций .NET в изолированном процессе требуются следующие пакеты:

+ [Microsoft.Azure.Functions.Worker](https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker/)
+ [Microsoft.Azure.Functions.Worker.Sdk](https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker.Sdk/)

### <a name="extension-packages"></a>Пакеты расширений

Поскольку функции, выполняемые в изолированном процессе .NET, используют различные типы привязок, им требуется уникальный набор пакетов расширений привязки. 

Эти пакеты расширений можно найти в [Microsoft. Azure. functions. Worker. Extensions](https://www.nuget.org/packages?q=Microsoft.Azure.Functions.Worker.Extensions).

## <a name="start-up-and-configuration"></a>Запуск и настройка 

При использовании изолированных функций .NET у вас есть доступ к запуску приложения-функции, которое обычно находится в Program.cs. Вы несете ответственность за создание и запуск собственного экземпляра узла. Таким образом, у вас также есть прямой доступ к конвейеру конфигурации приложения. Можно значительно упростить внедрение зависимостей и запуск по промежуточного слоя при выполнении вне процесса. 

В следующем коде показан пример `HostBuilder` конвейера:

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/FunctionApp/Program.cs" id="docsnippet_startup":::

Объект `HostBuilder` используется для создания и возврата полностью инициализированного `IHost` экземпляра, который выполняется асинхронно для запуска приложения-функции. 

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/FunctionApp/Program.cs" id="docsnippet_host_run":::

### <a name="configuration"></a>Конфигурация

Наличие доступа к конвейеру построителя узлов означает, что во время инициализации можно задать любые конфигурации для конкретного приложения. Эти конфигурации применяются к приложению-функции, выполняемому в отдельном процессе. Чтобы внести изменения в узел функций или в конфигурацию триггера и привязки, по-прежнему необходимо использовать [host.jsв файле](functions-host-json.md).      

В следующем примере показано, как добавить конфигурацию `args` , которая считывается как аргументы командной строки: 
 
:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/FunctionApp/Program.cs" id="docsnippet_configure_app" :::

`ConfigureAppConfiguration`Метод используется для настройки оставшейся части процесса сборки и приложения. В этом примере также используется [IConfigurationBuilder](/dotnet/api/microsoft.extensions.configuration.iconfigurationbuilder?view=dotnet-plat-ext-5.0&preserve-view=true), который упрощает добавление нескольких элементов конфигурации. Поскольку `ConfigureAppConfiguration` возвращает один и тот же экземпляр [`IConfiguration`](/dotnet/api/microsoft.extensions.configuration.iconfiguration?view=dotnet-plat-ext-5.0&preserve-view=true) , можно также вызвать его несколько раз, чтобы добавить несколько элементов конфигурации. Доступ к полному набору конфигураций можно получить [`HostBuilderContext.Configuration`](/dotnet/api/microsoft.extensions.hosting.hostbuildercontext.configuration?view=dotnet-plat-ext-5.0&preserve-view=true) и в, и в [`IHost.Services`](/dotnet/api/microsoft.extensions.hosting.ihost.services?view=dotnet-plat-ext-5.0&preserve-view=true) .

Дополнительные сведения о конфигурации см. [в разделе Configuration in ASP.NET Core](/aspnet/core/fundamentals/configuration/?view=aspnetcore-5.0&preserve-view=true). 

### <a name="dependency-injection"></a>Внедрение зависимостей

Внедрение зависимостей упрощено по сравнению с библиотеками классов .NET. Вместо создания класса Startup для регистрации служб необходимо просто вызвать `ConfigureServices` в построителе узлов и использовать методы расширения [`IServiceCollection`](/dotnet/api/microsoft.extensions.dependencyinjection.iservicecollection?view=dotnet-plat-ext-5.0&preserve-view=true) для внедрения конкретных служб. 

В следующем примере вводится зависимость одноэлементной службы:  
 
:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/FunctionApp/Program.cs" id="docsnippet_dependency_injection" :::

Дополнительные сведения см. [в разделе внедрение зависимостей в ASP.NET Core](/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-5.0&preserve-view=true).

### <a name="middleware"></a>ПО промежуточного слоя

.NET изолирует также поддерживает регистрацию по промежуточного слоя, используя модель, аналогичную той, что существует в ASP.NET. Эта модель дает возможность вставлять логику в конвейер вызовов, а также до и после выполнения функций.

Пока не предоставлен полный набор API-интерфейсов для регистрации по промежуточного слоя, поддерживается регистрация по промежуточного слоя, и мы добавили пример в пример приложения в папке по промежуточного слоя.

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/FunctionApp/Program.cs" id="docsnippet_middleware" :::

## <a name="execution-context"></a>Контекст выполнения

.NET изолирует передает `FunctionContext` объект методам функций. Этот объект позволяет получить [`ILogger`](/dotnet/api/microsoft.extensions.logging.ilogger?view=dotnet-plat-ext-5.0&preserve-view=true) экземпляр для записи в журналы, вызвав `GetLogger` метод и указав `categoryName` строку. Дополнительные сведения см. в разделе [ведение журнала](#logging). 

## <a name="bindings"></a>Привязки 

Привязки определяются с помощью атрибутов методов, параметров и возвращаемых типов. Метод функции — это метод с `Function` атрибутом и, который применяется к входному параметру, как показано в следующем примере:

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/SampleApp/Queue/QueueFunction.cs" id="docsnippet_queue_trigger" :::

Атрибут триггера указывает тип триггера и привязывает входные данные к параметру метода. Предыдущий пример функции активируется сообщением очереди, а сообщение очереди передается в метод в `myQueueItem` параметре.

Атрибут `Function` помечает метод как точку входа функции. Имя должно быть уникальным в пределах проекта, начинаться с буквы и содержать только буквы, цифры, `_` и `-` , до 127 символов. Шаблоны проектов часто создают метод `Run`, но метод может иметь любое допустимое имя для метода C#.

Поскольку изолированные проекты .NET выполняются в отдельном рабочем процессе, привязки не могут использовать преимущества классов с широкими привязками, таких как `ICollector<T>` , `IAsyncCollector<T>` и `CloudBlockBlob` . Также отсутствует прямая поддержка типов, унаследованных от основных пакетов SDK службы, таких как [DocumentClient](/dotnet/api/microsoft.azure.documents.client.documentclient) и [BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage). Вместо этого привязки основываются на строках, массивах и сериализуемых типах, таких как простые старые объекты классов (POCO). 

Для триггеров HTTP необходимо использовать `HttpRequestData` и `HttpResponseData` для доступа к данным запроса и ответа. Это связано с тем, что у вас нет доступа к исходным объектам HTTP-запросов и ответов при выполнении вне процесса. 

### <a name="input-bindings"></a>Входные привязки

Функция может иметь ноль или более входных привязок, которые могут передавать данные в функцию. Как и триггеры, входные привязки определяются путем применения атрибута привязки к входному параметру. При выполнении функции среда выполнения пытается получить данные, указанные в привязке. Запрашиваемые данные часто зависят от сведений, предоставленных триггером с помощью параметров привязки.  

### <a name="output-bindings"></a>Выходные привязки

Для записи в выходную привязку необходимо применить атрибут выходной привязки к методу функции, который определил, как выполнять запись в привязанную службу. Значение, возвращаемое методом, записывается в выходную привязку. Например, в следующем примере строковое значение записывается в очередь сообщений с `functiontesting2` помощью выходной привязки.

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/SampleApp/Queue/QueueFunction.cs" id="docsnippet_queue_output_binding" :::

### <a name="multiple-output-bindings"></a>Несколько выходных привязок

Данные, записанные в выходную привязку, всегда являются возвращаемым значением функции. Если необходимо выполнить запись в более чем одну выходную привязку, необходимо создать пользовательский тип возвращаемого значения. Этот тип возвращаемого значения должен иметь атрибут выходной привязки, примененный к одному или нескольким свойствам класса. В следующем примере выполняется запись в ответ HTTP и в выходную привязку очереди.

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/FunctionApp/Function1/Function1.cs" id="docsnippet_multiple_outputs":::

### <a name="http-trigger"></a>Триггер HTTP

Триггеры HTTP переводит сообщение входящего HTTP-запроса в `HttpRequestData` объект, который передается в функцию. Этот объект предоставляет данные из запроса, включая `Headers` ,, `Cookies` , `Identities` `URL` и необязательное сообщение `Body` . Этот объект представляет собой представление объекта HTTP-запроса, а не самого запроса. 

Аналогичным образом функция возвращает `HttpReponseData` объект, который предоставляет данные, используемые для создания HTTP-ответа, включая сообщение `StatusCode` , `Headers` и, при необходимости, сообщение `Body` .  

Следующий код является триггером HTTP 

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/SampleApp/Http/HttpFunction.cs" id="docsnippet_http_trigger" :::

## <a name="logging"></a>Ведение журнала

В изолированной среде .NET можно выполнять запись в журналы с помощью [`ILogger`](/dotnet/api/microsoft.extensions.logging.ilogger?view=dotnet-plat-ext-5.0&preserve-view=true) экземпляра, полученного из `FunctionContext` объекта, переданного в функцию. Вызовите `GetLogger` метод, передав строковое значение, представляющее собой имя категории, в которую записываются журналы. Категория обычно представляет собой имя конкретной функции, из которой записываются журналы. Дополнительные сведения о категориях см. в [статье мониторинг](functions-monitoring.md#log-levels-and-categories). 

В следующем примере показано, как получить `ILogger` и записать журналы внутри функции:

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/SampleApp/Http/HttpFunction.cs" id="docsnippet_logging" ::: 

Используйте различные методы `ILogger` для записи различных уровней ведения журнала, например `LogWarning` или `LogError` . Дополнительные сведения об уровнях ведения журнала см. в [статье мониторинг](functions-monitoring.md#log-levels-and-categories).

[`ILogger`](/dotnet/api/microsoft.extensions.logging.ilogger?view=dotnet-plat-ext-5.0&preserve-view=true)Также предоставляется при использовании [внедрения зависимостей](#dependency-injection).

## <a name="differences-with-net-class-library-functions"></a>Различия с функциями библиотеки классов .NET

В этом разделе описывается текущее состояние функциональных различий функциональности и поведения, выполняемых в .NET 5,0, по сравнению с функциями библиотеки классов .NET, выполняемыми в процессе.

| Функция и поведение |  Внутрипроцессный (.NET Core 3,1) | Вне процесса (.NET 5,0) |
| ---- | ---- | ---- |
| Версии .NET | LTS (.NET Core 3,1) | Текущий (.NET 5,0) |
| Основные пакеты | [Microsoft.NET.Sdk.Functions](https://www.nuget.org/packages/Microsoft.NET.Sdk.Functions/) | [Microsoft.Azure.Functions.Worker](https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker/)<br/>[Microsoft.Azure.Functions.Worker](https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker.Sdk) | 
| Привязка пакетов расширений | [`Microsoft.Azure.WebJobs.Extensions.*`](https://www.nuget.org/packages?q=Microsoft.Azure.WebJobs.Extensions)  | Ниже [`Microsoft.Azure.Functions.Worker.Extensions.*`](https://www.nuget.org/packages?q=Microsoft.Azure.Functions.Worker.Extensions) | 
| Ведение журнала | [`ILogger`](/dotnet/api/microsoft.extensions.logging.ilogger?view=dotnet-plat-ext-5.0&preserve-view=true) передается в функцию | [`ILogger`](/dotnet/api/microsoft.extensions.logging.ilogger?view=dotnet-plat-ext-5.0&preserve-view=true) получено из `FunctionContext` |
| Токены отмены | [Поддерживается](functions-dotnet-class-library.md#cancellation-tokens) | Не поддерживается |
| Выходные привязки | Выходные параметры | Возвращаемые значения |
| Типы выходных привязок |  `IAsyncCollector`, [DocumentClient](/dotnet/api/microsoft.azure.documents.client.documentclient?view=azure-dotnet&preserve-view=true), [BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage?view=azure-dotnet&preserve-view=true)и другие типы, зависящие от клиента | Простые типы, сериализуемые типы JSON и массивы. |
| Несколько выходных привязок | Поддерживается | [Поддерживается](#multiple-output-bindings) |
| Триггер HTTP | [`HttpRequest`](/dotnet/api/microsoft.aspnetcore.http.httprequest?view=aspnetcore-5.0&preserve-view=true)/[`ObjectResult`](/dotnet/api/microsoft.aspnetcore.mvc.objectresult?view=aspnetcore-5.0&preserve-view=true) | `HttpRequestData`/`HttpResponseData` |
| Устойчивые функции | [Поддерживается](durable/durable-functions-overview.md) | Не поддерживается | 
| Императивные привязки | [Поддерживается](functions-dotnet-class-library.md#binding-at-runtime) | Не поддерживается |
| function.jsартефакта | Сформировано | Не создано |
| Конфигурация | [host.json](functions-host-json.md) | [host.js](functions-host-json.md) и [Настраиваемая инициализация](#configuration) |
| Внедрение зависимостей | [Поддерживается](functions-dotnet-dependency-injection.md)  | [Поддерживается](#dependency-injection) |
| ПО промежуточного слоя | Не поддерживается | [Поддерживается](#middleware) |
| Время холодного запуска | "Typical" (Стандартный) | Больше времени из-за JIT-запуска. Для уменьшения возможных задержек запустите в Linux вместо Windows. |
| ReadyToRun | [Поддерживается](functions-dotnet-class-library.md#readytorun) | _TBD_ |


## <a name="next-steps"></a>Дальнейшие действия

+ [Дополнительные сведения о триггерах и привязках](functions-triggers-bindings.md)
+ [Дополнительные сведения о рекомендациях по решению "Функции Azure"](functions-best-practices.md)
