---
title: Изолированное программное планирование .NET для .NET 5,0 в службе "функции Azure"
description: Узнайте, как использовать изолированный процесс .NET для выполнения функций C# в .NET 5,0 вне процесса в Azure.
ms.service: azure-functions
ms.topic: conceptual
ms.date: 03/01/2021
ms.custom: template-concept
ms.openlocfilehash: 4da685c247427e78297df1753779ee9b5c7866b8
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105023203"
---
# <a name="guide-for-running-functions-on-net-50-in-azure"></a>Инструкции по выполнению функций в .NET 5,0 в Azure

Эта статья содержит вводные сведения об использовании C# для разработки функций изолированного процесса .NET, которые выполняются вне процесса в функциях Azure. Выполнение вне процесса позволяет отделить код функции от среды выполнения функций Azure. Он также позволяет создавать и запускать функции, предназначенные для текущего выпуска .NET 5,0. 

| Начало работы | Основные понятия| Примеры |
|--|--|--| 
| <ul><li>[Использование Visual Studio Code](dotnet-isolated-process-developer-howtos.md?pivots=development-environment-vscode)</li><li>[Использование средств командной строки](dotnet-isolated-process-developer-howtos.md?pivots=development-environment-cli)</li><li>[Использование Visual Studio](dotnet-isolated-process-developer-howtos.md?pivots=development-environment-vs)</li></ul> | <ul><li>[Сравнение вариантов размещения](functions-scale.md)</li><li>[Мониторинг](functions-monitoring.md)</li> | <ul><li>[Образцы ссылок](https://github.com/Azure/azure-functions-dotnet-worker/tree/main/samples)</li></ul> |

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
+ Файл Program. cs, который является точкой входа для приложения.

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

При использовании изолированных функций .NET у вас есть доступ к запуску приложения-функции, которое обычно находится в Program. cs. Вы несете ответственность за создание и запуск собственного экземпляра узла. Таким образом, у вас также есть прямой доступ к конвейеру конфигурации приложения. При выполнении вне процесса можно значительно упростить добавление конфигураций, внедрение зависимостей и запуск собственного по промежуточного слоя. 

В следующем коде показан пример конвейера [хостбуилдер] :

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/FunctionApp/Program.cs" id="docsnippet_startup":::

[Хостбуилдер] используется для создания и возврата полностью инициализированного экземпляра [ихост] , который выполняется асинхронно для запуска приложения-функции. 

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/FunctionApp/Program.cs" id="docsnippet_host_run":::

### <a name="configuration"></a>Конфигурация

Метод [конфигурефунктионсворкердефаултс] используется для добавления параметров, необходимых для выполнения вне процесса приложения-функции, в том числе следующие функциональные возможности.

+ Набор преобразователей по умолчанию.
+ Задайте для параметра [жсонсериализероптионс] по умолчанию значение игнорировать регистр в именах свойств.
+ Интеграция с журналом функций Azure.
+ Выходные данные и компоненты по промежуточного слоя для привязки.
+ По промежуточного слоя выполнения функции.
+ Поддержка gRPC по умолчанию. 

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/FunctionApp/Program.cs" id="docsnippet_configure_defaults" :::   

Наличие доступа к конвейеру построителя узлов означает, что во время инициализации можно также задать любые конфигурации для конкретного приложения. Вы можете вызвать метод [конфигуреаппконфигуратион] на [хостбуилдер] один или несколько раз, чтобы добавить конфигурации, необходимые для приложения функции. Дополнительные сведения о конфигурации приложений см. [в разделе Configuration in ASP.NET Core](/aspnet/core/fundamentals/configuration/?view=aspnetcore-5.0&preserve-view=true). 

Эти конфигурации применяются к приложению-функции, выполняемому в отдельном процессе. Чтобы внести изменения в узел функций или в конфигурацию триггера и привязки, по-прежнему необходимо использовать [host.jsв файле](functions-host-json.md).   

### <a name="dependency-injection"></a>Внедрение зависимостей

Внедрение зависимостей упрощено по сравнению с библиотеками классов .NET. Вместо того чтобы создавать класс запуска для регистрации служб, достаточно вызвать [ConfigureServices] в построителе узлов и использовать методы расширения в [IServiceCollection] для внедрения конкретных служб. 

В следующем примере вводится зависимость одноэлементной службы:  
 
:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/FunctionApp/Program.cs" id="docsnippet_dependency_injection" :::

Дополнительные сведения см. [в разделе внедрение зависимостей в ASP.NET Core](/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-5.0&preserve-view=true).

### <a name="middleware"></a>ПО промежуточного слоя

.NET изолирует также поддерживает регистрацию по промежуточного слоя, используя модель, аналогичную той, что существует в ASP.NET. Эта модель дает возможность вставлять логику в конвейер вызовов, а также до и после выполнения функций.

Метод расширения [конфигурефунктионсворкердефаултс] имеет перегрузку, которая позволяет зарегистрировать собственное по промежуточного слоя, как видно в следующем примере.  

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/CustomMiddleware/Program.cs" id="docsnippet_middleware_register" :::

Более полный пример использования пользовательского по промежуточного слоя в приложении-функции см. в разделе [образец настраиваемого по промежуточного слоя](https://github.com/Azure/azure-functions-dotnet-worker/blob/main/samples/CustomMiddleware).

## <a name="execution-context"></a>Контекст выполнения

.NET изолирует передает объект [функтионконтекст] методам функций. Этот объект позволяет получить экземпляр [ILogger] для записи в журналы, вызвав метод [GetLogger] и указав `categoryName` строку. Дополнительные сведения см. в разделе [ведение журнала](#logging). 

## <a name="bindings"></a>Привязки 

Привязки определяются с помощью атрибутов методов, параметров и возвращаемых типов. Метод функции — это метод с `Function` атрибутом и атрибутом Trigger, примененным к входному параметру, как показано в следующем примере:

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/Extensions/Queue/QueueFunction.cs" id="docsnippet_queue_trigger" :::

Атрибут триггера указывает тип триггера и привязывает входные данные к параметру метода. Предыдущий пример функции активируется сообщением очереди, а сообщение очереди передается в метод в `myQueueItem` параметре.

Атрибут `Function` помечает метод как точку входа функции. Имя должно быть уникальным в пределах проекта, начинаться с буквы и содержать только буквы, цифры, `_` и `-` , до 127 символов. Шаблоны проектов часто создают метод `Run`, но метод может иметь любое допустимое имя для метода C#.

Поскольку изолированные проекты .NET выполняются в отдельном рабочем процессе, привязки не могут использовать преимущества классов с широкими привязками, таких как `ICollector<T>` , `IAsyncCollector<T>` и `CloudBlockBlob` . Также отсутствует прямая поддержка типов, унаследованных от основных пакетов SDK службы, таких как [DocumentClient] и [BrokeredMessage]. Вместо этого привязки основываются на строках, массивах и сериализуемых типах, таких как простые старые объекты классов (POCO). 

Для триггеров HTTP необходимо использовать [хттпрекуестдата] и [хттпреспонседата] для доступа к данным запроса и ответа. Это связано с тем, что у вас нет доступа к исходным объектам HTTP-запросов и ответов при выполнении вне процесса.

Полный набор эталонных примеров использования триггеров и привязок при выполнении вне процесса см. в [образце ссылок на расширения привязки](https://github.com/Azure/azure-functions-dotnet-worker/blob/main/samples/Extensions). 

### <a name="input-bindings"></a>Входные привязки

Функция может иметь ноль или более входных привязок, которые могут передавать данные в функцию. Как и триггеры, входные привязки определяются путем применения атрибута привязки к входному параметру. При выполнении функции среда выполнения пытается получить данные, указанные в привязке. Запрашиваемые данные часто зависят от сведений, предоставленных триггером с помощью параметров привязки.  

### <a name="output-bindings"></a>Выходные привязки

Для записи в выходную привязку необходимо применить атрибут выходной привязки к методу функции, который определил, как выполнять запись в привязанную службу. Значение, возвращаемое методом, записывается в выходную привязку. Например, в следующем примере строковое значение записывается в очередь сообщений с `functiontesting2` помощью выходной привязки.

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/Extensions/Queue/QueueFunction.cs" id="docsnippet_queue_output_binding" :::

### <a name="multiple-output-bindings"></a>Несколько выходных привязок

Данные, записанные в выходную привязку, всегда являются возвращаемым значением функции. Если необходимо выполнить запись в более чем одну выходную привязку, необходимо создать пользовательский тип возвращаемого значения. Этот тип возвращаемого значения должен иметь атрибут выходной привязки, примененный к одному или нескольким свойствам класса. В следующем примере из триггера HTTP записывается как в ответ HTTP, так и в выходную привязку очереди.

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/Extensions/MultiOutput/MultiOutput.cs" id="docsnippet_multiple_outputs":::

Ответ от триггера HTTP всегда считается выходным, поэтому атрибут возвращаемого значения не требуется.

### <a name="http-trigger"></a>Триггер HTTP

Триггеры HTTP переводит входящее сообщение HTTP-запроса в объект [хттпрекуестдата] , который передается в функцию. Этот объект предоставляет данные из запроса, включая `Headers` ,, `Cookies` , `Identities` `URL` и необязательное сообщение `Body` . Этот объект представляет собой представление объекта HTTP-запроса, а не самого запроса. 

Аналогичным образом функция возвращает объект [хттпреспонседата] , который предоставляет данные, используемые для создания HTTP-ответа, включая сообщение `StatusCode` , `Headers` и, при необходимости, сообщение `Body` .  

Следующий код является триггером HTTP 

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/Extensions/Http/HttpFunction.cs" id="docsnippet_http_trigger" :::

## <a name="logging"></a>Ведение журнала

В изолированной среде .NET можно выполнять запись в журналы с помощью экземпляра [ILogger] , полученного из объекта [функтионконтекст] , переданного в функцию. Вызовите метод [GetLogger] , передав строковое значение, представляющее собой имя категории, в которую записываются журналы. Категория обычно представляет собой имя конкретной функции, из которой записываются журналы. Дополнительные сведения о категориях см. в [статье мониторинг](functions-monitoring.md#log-levels-and-categories). 

В следующем примере показано, как получить [ILogger] и записать журналы внутри функции:

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/Extensions/Http/HttpFunction.cs" id="docsnippet_logging" ::: 

Используйте различные методы [ILogger] для записи различных уровней ведения журнала, например `LogWarning` или `LogError` . Дополнительные сведения об уровнях ведения журнала см. в [статье мониторинг](functions-monitoring.md#log-levels-and-categories).

При использовании [внедрения зависимостей](#dependency-injection)также предоставляется [ILogger] .

## <a name="differences-with-net-class-library-functions"></a>Различия с функциями библиотеки классов .NET

В этом разделе описывается текущее состояние функциональных различий функциональности и поведения, выполняемых в .NET 5,0, по сравнению с функциями библиотеки классов .NET, выполняемыми в процессе.

| Функция и поведение |  Внутрипроцессный (.NET Core 3,1) | Вне процесса (.NET 5,0) |
| ---- | ---- | ---- |
| Версии .NET | LTS (.NET Core 3,1) | Текущий (.NET 5,0) |
| Основные пакеты | [Microsoft.NET.Sdk.Functions](https://www.nuget.org/packages/Microsoft.NET.Sdk.Functions/) | [Microsoft.Azure.Functions.Worker](https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker/)<br/>[Microsoft.Azure.Functions.Worker.Sdk](https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker.Sdk) | 
| Привязка пакетов расширений | [Microsoft. Azure. веб-задания. Extensions. *](https://www.nuget.org/packages?q=Microsoft.Azure.WebJobs.Extensions)  | В [Microsoft. Azure. functions. Worker. Extensions. *](https://www.nuget.org/packages?q=Microsoft.Azure.Functions.Worker.Extensions) | 
| Ведение журнала | [ILogger] , переданный функции | [ILogger] получен из [функтионконтекст] |
| Токены отмены | [Поддерживается](functions-dotnet-class-library.md#cancellation-tokens) | Не поддерживается |
| Выходные привязки | Выходные параметры | Возвращаемые значения |
| Типы выходных привязок |  `IAsyncCollector`, [DocumentClient], [BrokeredMessage]и другие типы, зависящие от клиента | Простые типы, сериализуемые типы JSON и массивы. |
| Несколько выходных привязок | Поддерживается | [Поддерживается](#multiple-output-bindings) |
| Триггер HTTP | [HttpRequest] / [Обжектресулт] | [Хттпрекуестдата] / [Хттпреспонседата] |
| Устойчивые функции | [Поддерживается](durable/durable-functions-overview.md) | Не поддерживается | 
| Императивные привязки | [Поддерживается](functions-dotnet-class-library.md#binding-at-runtime) | Не поддерживается |
| function.jsартефакта | Сформировано | Не создано |
| Конфигурация | [host.json](functions-host-json.md) | [host.js](functions-host-json.md) и настраиваемая инициализация |
| Внедрение зависимостей | [Поддерживается](functions-dotnet-dependency-injection.md)  | [Поддерживается](#dependency-injection) |
| ПО промежуточного слоя | Не поддерживается | Поддерживается |
| Время холодного запуска | "Typical" (Стандартный) | Больше времени из-за JIT-запуска. Для уменьшения возможных задержек запустите в Linux вместо Windows. |
| ReadyToRun | [Поддерживается](functions-dotnet-class-library.md#readytorun) | _TBD_ |

## <a name="known-issues"></a>Известные проблемы

Дополнительные сведения о способах устранения проблем, связанных с функциями изолированных процессов .NET, см. на [этой странице известных проблем](https://aka.ms/AAbh18e). Чтобы сообщить о проблемах, [Создайте проблему в этом репозитории GitHub](https://github.com/Azure/azure-functions-dotnet-worker/issues/new/choose).  

## <a name="next-steps"></a>Дальнейшие действия

+ [Дополнительные сведения о триггерах и привязках](functions-triggers-bindings.md)
+ [Дополнительные сведения о рекомендациях по решению "Функции Azure"](functions-best-practices.md)


[хостбуилдер]: /dotnet/api/microsoft.extensions.hosting.hostbuilder?view=dotnet-plat-ext-5.0&preserve-view=true
[ихост]: /dotnet/api/microsoft.extensions.hosting.ihost?view=dotnet-plat-ext-5.0&preserve-view=true
[конфигурефунктионсворкердефаултс]: /dotnet/api/microsoft.extensions.hosting.workerhostbuilderextensions.configurefunctionsworkerdefaults?view=azure-dotnet&preserve-view=true#Microsoft_Extensions_Hosting_WorkerHostBuilderExtensions_ConfigureFunctionsWorkerDefaults_Microsoft_Extensions_Hosting_IHostBuilder_
[ConfigureAppConfiguration]: /dotnet/api/microsoft.extensions.hosting.hostbuilder.configureappconfiguration?view=dotnet-plat-ext-5.0&preserve-view=true
[IServiceCollection]: /dotnet/api/microsoft.extensions.dependencyinjection.iservicecollection?view=dotnet-plat-ext-5.0&preserve-view=true
[ConfigureServices]: /dotnet/api/microsoft.extensions.hosting.hostbuilder.configureservices?view=dotnet-plat-ext-5.0&preserve-view=true
[функтионконтекст]: /dotnet/api/microsoft.azure.functions.worker.functioncontext?view=azure-dotnet&preserve-view=true
[ILogger]: /dotnet/api/microsoft.extensions.logging.ilogger?view=dotnet-plat-ext-5.0&preserve-view=true
[GetLogger]: /dotnet/api/microsoft.azure.functions.worker.functioncontextloggerextensions.getlogger?view=azure-dotnet&preserve-view=true
[DocumentClient]: /dotnet/api/microsoft.azure.documents.client.documentclient
[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
[хттпрекуестдата]: /dotnet/api/microsoft.azure.functions.worker.http.httprequestdata?view=azure-dotnet&preserve-view=true
[хттпреспонседата]: /dotnet/api/microsoft.azure.functions.worker.http.httpresponsedata?view=azure-dotnet&preserve-view=true
[HttpRequest]: /dotnet/api/microsoft.aspnetcore.http.httprequest?view=aspnetcore-5.0&preserve-view=true
[обжектресулт]: /dotnet/api/microsoft.aspnetcore.mvc.objectresult?view=aspnetcore-5.0&preserve-view=true
[жсонсериализероптионс]: /api/system.text.json.jsonserializeroptions?view=net-5.0&preserve-view=true
