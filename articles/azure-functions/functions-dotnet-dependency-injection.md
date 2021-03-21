---
title: Использование внедрения зависимостей в Функциях Azure .NET
description: Узнайте, как использовать внедрение зависимостей для регистрации и использования служб в функциях .NET
author: ggailey777
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 01/27/2021
ms.author: glenga
ms.reviewer: jehollan
ms.openlocfilehash: 66e2cd22f4bcb95be65d6d04345dcac622436a04
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98955094"
---
# <a name="use-dependency-injection-in-net-azure-functions"></a>Использование внедрения зависимостей в Функциях Azure .NET

Функции Azure поддерживают шаблон разработки программного обеспечения с внедрением зависимостей (DI), который является приемом [инверсии управления (IoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) между классами и их зависимостями.

- Внедрение зависимостей в Функциях Azure основано на функциях внедрения зависимостей .NET Core. Рекомендуется ознакомиться с [Внедрением зависимостей .NET Core](/aspnet/core/fundamentals/dependency-injection). Существуют различия в переопределении зависимостей и считывании значений конфигурации, если служба Функций Azure использует план потребления.

- Поддержка внедрения зависимостей появилась в Функциях Azure 2.x.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем можно будет использовать внедрение зависимостей, необходимо установить следующие пакеты NuGet:

- [Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions/);

- пакет [Microsoft.NET.Sdk.Functions](https://www.nuget.org/packages/Microsoft.NET.Sdk.Functions/) версии 1.0.28 или более поздней.

- [Microsoft. Extensions. депенденциинжектион](https://www.nuget.org/packages/Microsoft.Extensions.DependencyInjection/) (в настоящее время поддерживается только версия 3. x и более ранние версии)

## <a name="register-services"></a>Регистрация служб

Чтобы зарегистрировать службы, создайте метод для настройки и добавления компонентов в экземпляр `IFunctionsHostBuilder`.  Узел функций Azure создает экземпляр `IFunctionsHostBuilder` и передает его непосредственно в метод.

Чтобы зарегистрировать метод, добавьте атрибут сборки `FunctionsStartup`, указывающий имя типа, используемого во время запуска.

```csharp
using Microsoft.Azure.Functions.Extensions.DependencyInjection;
using Microsoft.Extensions.DependencyInjection;

[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

namespace MyNamespace
{
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            builder.Services.AddHttpClient();

            builder.Services.AddSingleton<IMyService>((s) => {
                return new MyService();
            });

            builder.Services.AddSingleton<ILoggerProvider, MyLoggerProvider>();
        }
    }
}
```

В этом примере используется пакет [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http/), необходимый для регистрации `HttpClient` при запуске.

### <a name="caveats"></a>Предупреждения

Последовательность действий по регистрации выполняется до и после того, как среда выполнения обработает класс запуска. Поэтому помните следующие важные моменты.

- *Класс запуска предназначен только для установки и регистрации.* Не используйте службы, зарегистрированные при запуске, в процессе запуска. Например, не пытайтесь зарегистрировать сообщение в средстве ведения журнала, регистрируемом во время запуска. Эта точка процесса регистрации слишком ранняя, чтобы службы были доступны для использования. После выполнения метода `Configure` среда выполнения Функций продолжит регистрировать дополнительные зависимости, что может повлиять на работу служб.

- *Контейнер внедрения зависимостей содержит только явно зарегистрированные типы*. Единственными службами, доступными в качестве внедряемых типов, являются те, что настроены в методе `Configure`. В результате такие относящиеся к Функциям типы, как `BindingContext` и `ExecutionContext`, недоступны во время установки или в качестве типов для внедрения.

## <a name="use-injected-dependencies"></a>Использование внедренных зависимостей

Внедрение конструктора используется для обеспечения доступности зависимостей в функции. Использование внедрения конструктора требует, чтобы не использовались статические классы для внедренных служб или классов функций.

В следующем примере показано, как зависимости `IMyService` и `HttpClient` вставляются в функцию, активируемую HTTP.

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;
using System.Net.Http;
using System.Threading.Tasks;

namespace MyNamespace
{
    public class MyHttpTrigger
    {
        private readonly HttpClient _client;
        private readonly IMyService _service;

        public MyHttpTrigger(HttpClient httpClient, IMyService service)
        {
            this._client = httpClient;
            this._service = service;
        }

        [FunctionName("MyHttpTrigger")]
        public async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            var response = await _client.GetAsync("https://microsoft.com");
            var message = _service.GetMessage();

            return new OkObjectResult("Response from function with injected dependencies.");
        }
    }
}
```

В этом примере используется пакет [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http/), необходимый для регистрации `HttpClient` при запуске.

## <a name="service-lifetimes"></a>Время существования служб

Приложения Функций Azure предоставляют те же самые времена существования служб, что и [внедрение зависимостей ASP.NET](/aspnet/core/fundamentals/dependency-injection#service-lifetimes). Для приложения Функций различные времена существования службы ведут себя следующим образом.

- **Временное**: временные службы создаются при каждом разрешении службы.
- **С заданной областью**: время существования службы с заданной областью соответствует времени выполнения функции. Службы с заданной областью создаются один раз для каждого выполнения функции. Последующие запросы к этой службе во время выполнения повторно используют существующий экземпляр службы.
- **Отдельная**: время существования отдельной службы соответствует времени существования узла. Она повторно используется при повторном выполнении функций на этом экземпляре. Для подключений и клиентов рекомендуется использовать службы со временем жизни «отдельная», например экземпляры `DocumentClient` или `HttpClient`.

Просмотрите или скачайте [образцы служб с различными временами существования](https://github.com/Azure/azure-functions-dotnet-extensions/tree/main/src/samples/DependencyInjection/Scopes) на GitHub.

## <a name="logging-services"></a>Службы ведения журналов

Если вам нужен собственный поставщик ведения журнала, зарегистрируйте пользовательский тип в качестве экземпляра [`ILoggerProvider`](/dotnet/api/microsoft.extensions.logging.iloggerfactory) , который доступен в пакете NuGet [Microsoft. Extensions. Logging. абстракций](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Abstractions/) .

Application Insights автоматически добавляется Функциями Azure.

> [!WARNING]
> - Не добавляйте `AddApplicationInsightsTelemetry()` в коллекцию служб, которая регистрирует службы, конфликтующие со службами, предоставляемыми средой.
> - Не зарегистрируйте собственный `TelemetryConfiguration` `TelemetryClient` объект или при использовании встроенных функций Application Insights. Если необходимо настроить собственный `TelemetryClient` экземпляр, создайте его с помощью внедренного, `TelemetryConfiguration` как показано в статьи [Регистрация пользовательской телеметрии в функциях C#](functions-dotnet-class-library.md?tabs=v2%2Ccmd#log-custom-telemetry-in-c-functions).

### <a name="iloggert-and-iloggerfactory"></a>ILogger<T> и ILoggerFactory

Узел внедряет `ILogger<T>` и `ILoggerFactory` службы в конструкторы.  Однако по умолчанию эти новые фильтры журнала отфильтровываются из журналов функций.  Необходимо изменить `host.json` файл, чтобы принять участие в дополнительных фильтрах и категориях.

В следующем примере показано, как добавить `ILogger<HttpTrigger>` с журналами, которые доступны для узла.

```csharp
namespace MyNamespace
{
    public class HttpTrigger
    {
        private readonly ILogger<HttpTrigger> _log;

        public HttpTrigger(ILogger<HttpTrigger> log)
        {
            _log = log;
        }

        [FunctionName("HttpTrigger")]
        public async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req)
        {
            _log.LogInformation("C# HTTP trigger function processed a request.");

            // ...
    }
}
```

В следующем примере `host.json` файла добавляется фильтр журнала.

```json
{
    "version": "2.0",
    "logging": {
        "applicationInsights": {
            "samplingSettings": {
                "isEnabled": true,
                "excludedTypes": "Request"
            }
        },
        "logLevel": {
            "MyNamespace.HttpTrigger": "Information"
        }
    }
}
```

Дополнительные сведения об уровнях ведения журнала см. в разделе [Настройка уровней ведения журнала](configure-monitoring.md#configure-log-levels).

## <a name="function-app-provided-services"></a>Службы, предоставляемые приложением-функцией

Узел функции регистрирует множество служб. Следующие службы можно уверенно использовать в качестве зависимости в приложении.

|Тип службы|Время существования|Описание|
|--|--|--|
|`Microsoft.Extensions.Configuration.IConfiguration`|Отдельная|Конфигурация среды выполнения|
|`Microsoft.Azure.WebJobs.Host.Executors.IHostIdProvider`|Отдельная|Отвечает за предоставление идентификатора экземпляра узла.|

Если существуют другие службы, от которых требуется зависимость, [создайте вопрос и предложите его на GitHub](https://github.com/azure/azure-functions-host).

### <a name="overriding-host-services"></a>Переопределение служб узла

Переопределение служб, предоставляемых узлом, в настоящее время не поддерживается.  Если имеются службы, которые необходимо переопределить, [создайте вопрос и предложите его на GitHub](https://github.com/azure/azure-functions-host).

## <a name="working-with-options-and-settings"></a>Работа с настройками и параметрами

Значения, определенные в [параметрах приложения](./functions-how-to-use-azure-function-app-settings.md#settings), доступны в экземпляре `IConfiguration`, который позволяет считывать значения параметров приложения в классе запуска.

Значения можно извлечь из экземпляра `IConfiguration` в пользовательский тип. Копирование значений параметров приложения в пользовательский тип упрощает тестирование служб, делая эти значения внедряемыми. Параметры, считываемые в экземпляр конфигурации, должны быть простыми парами «ключ-значение».

Рассмотрим следующий класс, который включает свойство, именуемое consistent, с параметром приложения,

```csharp
public class MyOptions
{
    public string MyCustomSetting { get; set; }
}
```

и файл `local.settings.json`, который может структурировать пользовательский параметр следующим образом.
```json
{
  "IsEncrypted": false,
  "Values": {
    "MyOptions:MyCustomSetting": "Foobar"
  }
}
```

Внутри метода `Startup.Configure` можно извлечь значения из экземпляра `IConfiguration` в пользовательский тип с помощью следующего кода.

```csharp
builder.Services.AddOptions<MyOptions>()
    .Configure<IConfiguration>((settings, configuration) =>
    {
        configuration.GetSection("MyOptions").Bind(settings);
    });
```

Вызов `Bind` копирует значения, которые соответствуют именам свойств из конфигурации, в пользовательский экземпляр. Экземпляр параметров теперь доступен в контейнере IoC для внедрения в функцию.

Объект параметров внедряется в функцию как экземпляр универсального интерфейса `IOptions`. Используйте свойство `Value` для доступа к значениям, найденным в конфигурации.

```csharp
using System;
using Microsoft.Extensions.Options;

public class HttpTrigger
{
    private readonly MyOptions _settings;

    public HttpTrigger(IOptions<MyOptions> options)
    {
        _settings = options.Value;
    }
}
```

Дополнительные сведения о работе с параметрами см. в [Шаблон параметров в ASP.NET Core](/aspnet/core/fundamentals/configuration/options).

## <a name="using-aspnet-core-user-secrets"></a>Использование ASP.NET Core пользовательских секретов

При локальной разработке ASP.NET Core предоставляет [средство диспетчера секретов](/aspnet/core/security/app-secrets#secret-manager) , позволяющее хранить секретные данные вне корневого каталога проекта. Это уменьшает вероятность того, что секреты случайно фиксируются в системе управления версиями. Azure Functions Core Tools (версия 3.0.3233 или более поздняя) автоматически считывает секреты, созданные диспетчером секретов ASP.NET Core.

Чтобы настроить проект функций Azure .NET для использования секретов пользователя, выполните следующую команду в корневом каталоге проекта.

```bash
dotnet user-secrets init
```

Затем используйте `dotnet user-secrets set` команду для создания или обновления секретов.

```bash
dotnet user-secrets set MySecret "my secret value"
```

Для доступа к значениям секретов пользователя в коде приложения функции используйте `IConfiguration` или `IOptions` .

## <a name="customizing-configuration-sources"></a>Настройка источников конфигурации

> [!NOTE]
> Настройка источника конфигурации доступна начиная с версии узла "функции Azure" 2.0.14192.0 и 3.0.14191.0.

Чтобы указать дополнительные источники конфигурации, переопределите `ConfigureAppConfiguration` метод в классе приложения функции `StartUp` .

В следующем примере добавляются значения конфигурации из базового и необязательного файла параметров приложения для конкретной среды.

```csharp
using System.IO;
using Microsoft.Azure.Functions.Extensions.DependencyInjection;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

namespace MyNamespace
{
    public class Startup : FunctionsStartup
    {
        public override void ConfigureAppConfiguration(IFunctionsConfigurationBuilder builder)
        {
            FunctionsHostBuilderContext context = builder.GetContext();

            builder.ConfigurationBuilder
                .AddJsonFile(Path.Combine(context.ApplicationRootPath, "appsettings.json"), optional: true, reloadOnChange: false)
                .AddJsonFile(Path.Combine(context.ApplicationRootPath, $"appsettings.{context.EnvironmentName}.json"), optional: true, reloadOnChange: false)
                .AddEnvironmentVariables();
        }
    }
}
```

Добавьте поставщики конфигурации в `ConfigurationBuilder` свойство объекта `IFunctionsConfigurationBuilder` . Дополнительные сведения об использовании поставщиков конфигурации см. [в разделе Configuration in ASP.NET Core](/aspnet/core/fundamentals/configuration/#configuration-providers).

Объект `FunctionsHostBuilderContext` получен из `IFunctionsConfigurationBuilder.GetContext()` . Используйте этот контекст, чтобы получить имя текущей среды и разрешить расположение файлов конфигурации в папке приложения функции.

По умолчанию файлы конфигурации, такие как *appsettings.js* , не копируются автоматически в выходную папку приложения функции. Обновите *CSPROJ* файл в соответствии с приведенным ниже образцом, чтобы убедиться, что файлы скопированы.

```xml
<None Update="appsettings.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>      
</None>
<None Update="appsettings.Development.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    <CopyToPublishDirectory>Never</CopyToPublishDirectory>
</None>
```

> [!IMPORTANT]
> Для приложений функций, выполняющихся в планах потребления или Premium, изменение значений конфигурации, используемых в триггерах, может привести к ошибкам масштабирования. Любые изменения этих свойств `FunctionsStartup` класса приводят к ошибке запуска приложения-функции.

## <a name="next-steps"></a>Дальнейшие действия

Для получения дополнительных сведений см. следующие ресурсы:

- [Как выполнять мониторинг приложений-функций](functions-monitoring.md)
- [Рекомендации по функциям](functions-best-practices.md)
