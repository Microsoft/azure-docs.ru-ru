---
title: Руководство по использованию флагов функций в приложении .NET Core | Документация Майкрософт
description: Из этого руководства вы узнаете, как реализовать флаги функций в приложениях .NET Core.
services: azure-app-configuration
documentationcenter: ''
author: AlexandraKemperMS
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 09/17/2020
ms.author: alkemper
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 4d54e1ff07b250b5595d2f8aee5f022bd2359721
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105729513"
---
# <a name="tutorial-use-feature-flags-in-an-aspnet-core-app"></a>Руководство по использованию флагов функций в приложении ASP.NET Core

Библиотеки управления функциями .NET Core обеспечивают идиоматическую поддержку реализации флагов функций в приложении .NET или ASP.NET Core. Эти библиотеки позволяют декларативно добавлять в код флаги компонентов, чтобы вам не нужно было вручную писать код для включения и отключения функций с помощью операторов `if`.

Библиотеки управления функциями также управляют жизненным циклом флагов функций в фоновом режиме. Например, библиотеки отвечают за обновление и кэширование состояний флагов или гарантируют неизменность состояния флага во время вызова запроса. Кроме того, библиотека ASP.NET Core предоставляет готовое решение по интеграции, включая действия контроллера MVC, представления, маршруты и ПО промежуточного слоя.

В [этом кратком руководстве](./quickstart-feature-flag-aspnet-core.md) показано несколько способов использования флагов компонентов в приложении ASP.NET Core. В этом руководстве показаны дополнительные параметры и возможности настройки библиотек управления компонентами. Вы можете использовать пример приложения, созданный с помощью этого краткого руководства, чтобы испытать пример кода, приведенный в этом руководстве. 

Справочную документацию по API управления компонентами ASP.NET Core см. в статье [Пространство имен Microsoft.FeatureManagement](/dotnet/api/microsoft.featuremanagement).

В этом учебнике рассматривается следующее.

> [!div class="checklist"]
> * Добавить флаги функций в ключевых частях приложения для управления доступностью функций.
> * Как выполнить интеграцию с Конфигурацией приложений при использовании этой службы для управления флагами функций.

## <a name="set-up-feature-management"></a>Настройка управления функциями

Чтобы получить доступ к диспетчеру компонентов .NET Core, в приложении должны быть ссылки на пакет NuGet `Microsoft.FeatureManagement.AspNetCore`.

Диспетчер компонентов .NET Core настраивается из собственной системы конфигурации платформы. Таким образом, вы можете определить параметры флагов компонентов приложения, используя любой источник конфигурации, поддерживаемый .NET Core, включая локальный файл *appsettings.json* или переменные среды.

По умолчанию диспетчер компонентов извлекает конфигурацию флагов компонентов из раздела `"FeatureManagement"` данных конфигурации .NET Core. Чтобы использовать расположение конфигурации по умолчанию, вызовите метод [AddFeatureManagement](/dotnet/api/microsoft.featuremanagement.servicecollectionextensions.addfeaturemanagement) **IServiceCollection**, переданный в метод **ConfigureServices** класса **Startup**.


```csharp
using Microsoft.FeatureManagement;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        ...
        services.AddFeatureManagement();
    }
}
```

Можно указать, чтобы конфигурация управления компонентами была получена из другого раздела конфигурации, вызвав [Configuration.GetSection](/dotnet/api/microsoft.web.administration.configuration.getsection) и передав имя нужного раздела. В следующем примере диспетчер функций считывает данные из другого раздела под названием `"MyFeatureFlags"`:

```csharp
using Microsoft.FeatureManagement;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        ...
        services.AddFeatureManagement(Configuration.GetSection("MyFeatureFlags"));
    }
}
```


При использовании фильтров во флагах компонентов необходимо включить пространство имен [Microsoft.FeatureManagement.FeatureFilters](/dotnet/api/microsoft.featuremanagement.featurefilters) и добавить вызов [AddFeatureFilters](/dotnet/api/microsoft.featuremanagement.ifeaturemanagementbuilder.addfeaturefilter), указав имя типа фильтра, который будет использоваться в качестве универсального типа метода. Дополнительные сведения об использовании фильтров компонентов для динамического включения и отключения возможностей см. в статье [Включение промежуточного развертывания компонентов для целевых аудиторий](./howto-targetingfilter-aspnet-core.md).

В следующем примере показано, как использовать встроенный фильтр функций под названием `PercentageFilter`:



```csharp
using Microsoft.FeatureManagement;
using Microsoft.FeatureManagement.FeatureFilters;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        ...
        services.AddFeatureManagement()
                .AddFeatureFilter<PercentageFilter>();
    }
}
```

Вместо того, чтобы прописывать флаги компонентов в коде приложения, рекомендуется хранить их за пределами приложения и управлять ими отдельно. Так вы можете изменять состояния флагов в любое время и немедленно применять эти изменения в приложении. Служба Конфигурации приложений Azure предоставляет выделенный пользовательский интерфейс портала для управления всеми флагами компонентов. Служба Конфигурации приложений Azure также предоставляет приложению флаги компонентов непосредственно через клиентские библиотеки .NET Core.

Самый простой способ подключить приложение ASP.NET Core к Конфигурации приложений — использование поставщика конфигураций, включенного в пакет NuGet `Microsoft.Azure.AppConfiguration.AspNetCore`. После включения ссылки на пакет выполните описанные ниже действия, чтобы использовать этот пакет NuGet.

1. Откройте файл *Program.cs* и добавьте следующий код:
    > [!IMPORTANT]
    > `CreateHostBuilder` заменяет `CreateWebHostBuilder` в .NET Core 3.x. Выберите правильный синтаксис в зависимости от среды.

    ### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)
    
    ```csharp
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
                webBuilder.ConfigureAppConfiguration(config =>
                {
                    var settings = config.Build();
                    config.AddAzureAppConfiguration(options =>
                        options.Connect(settings["ConnectionStrings:AppConfig"]).UseFeatureFlags());
                }).UseStartup<Startup>());
    ```

    ### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)
    
    ```csharp
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;

    public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
            webBuilder.ConfigureAppConfiguration(config =>
            {
                var settings = config.Build();
                config.AddAzureAppConfiguration(options =>
                    options.Connect(settings["ConnectionStrings:AppConfig"]).UseFeatureFlags());
            }).UseStartup<Startup>());
    ```
        
    ### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)
    
    ```csharp
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                    .ConfigureAppConfiguration(config =>
                    {
                        var settings = config.Build();
                        config.AddAzureAppConfiguration(options =>
                            options.Connect(settings["ConnectionStrings:AppConfig"]).UseFeatureFlags());
                    }).UseStartup<Startup>();
    ```
    ---

2. Откройте файл *Startup.cs* и обновите метод `Configure` и `ConfigureServices`, чтобы добавить ПО промежуточного слоя с именем `UseAzureAppConfiguration`. Это ПО позволяет периодически обновлять значения флагов функций, пока веб-приложение ASP.NET Core продолжает получать запросы.



    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        app.UseAzureAppConfiguration();
    }
    ```

   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddAzureAppConfiguration();
   }
   ```
   
В типичном сценарии значения флагов компонентов периодически обновляются при развертывании и включении различных компонентов приложения. По умолчанию значения флагов функций кэшируются на 30 секунд, поэтому операция обновления, запущенная в тот момент, когда ПО промежуточного слоя получает запрос, не обновит значение, пока не истечет срок действия кэшированного значения. В приведенном ниже коде показано, как изменить срок действия кэша или интервал опроса на 5 минут, установив [CacheExpirationInterval](/dotnet/api/microsoft.extensions.configuration.azureappconfiguration.featuremanagement.featureflagoptions.cacheexpirationinterval) в вызове **UseFeatureFlags**.


    
```csharp
config.AddAzureAppConfiguration(options =>
    options.Connect(settings["ConnectionStrings:AppConfig"]).UseFeatureFlags(featureFlagOptions => {
        featureFlagOptions.CacheExpirationInterval = TimeSpan.FromMinutes(5);
    }));
});
```


## <a name="feature-flag-declaration"></a>Объявление флага функции

Каждая декларация флага компонентов состоит из двух частей: имени и списка из одного или нескольких фильтров, использующихся для оценки того, *включена* ли функция (то есть, если его значение равно `True`). Фильтр определяет критерий, при котором должна быть включена функция.

Если флаг функции имеет несколько фильтров, список фильтров проходится по порядку, пока один из фильтров не определит, что функция должна быть включена. На этом этапе флаг функции *включен*, и все остальные результаты фильтрации пропускаются. Если фильтр не указывает, что эта функция должна быть включена, флаг функции остается *отключенным*.

Диспетчер функций поддерживает файл *appsettings.json* в качестве источника конфигурации для флагов функций. В следующем примере показано, как настроить флаги функций в JSON-файле:

```JSON
"FeatureManagement": {
    "FeatureA": true, // Feature flag set to on
    "FeatureB": false, // Feature flag set to off
    "FeatureC": {
        "EnabledFor": [
            {
                "Name": "Percentage",
                "Parameters": {
                    "Value": 50
                }
            }
        ]
    }
}
```

В соответствии с соглашением раздел `FeatureManagement` этого JSON-документа используется для параметров флагов функций. Предыдущий пример показывает три флага функций с фильтрами, определенными в свойстве `EnabledFor`:

* Флаг `FeatureA`*включен*.
* Флаг `FeatureB`*отключен*.
* `FeatureC` указывает фильтр с именем `Percentage` и свойством `Parameters`. `Percentage` — это настраиваемый фильтр. В этом примере `Percentage` задает 50-процентную вероятность того, что флаг `FeatureC` может быть *включен*. Инструкции по использованию фильтров компонентов см. в статье [Использование фильтров компонентов для включения условных флагов компонентов](./howto-feature-filters-aspnet-core.md).




## <a name="use-dependency-injection-to-access-ifeaturemanager"></a>Внедрение зависимостей для доступа к IFeatureManager 

Для некоторых операций, таких как проверка значений флагов компонентов вручную, необходимо получить экземпляр [IFeatureManager](/dotnet/api/microsoft.featuremanagement.ifeaturemanager?preserve-view=true&view=azure-dotnet-preview). В MVC ASP.NET Core можно получить доступ к диспетчеру компонентов `IFeatureManager` путем внедрения зависимостей. В следующем примере аргумент типа `IFeatureManager` добавляется в сигнатуру конструктора для контроллера. Среда выполнения автоматически разрешает ссылку и предоставляет интерфейс при вызове конструктора. Если вы используете шаблон приложения, в котором у контроллера уже есть один или несколько аргументов внедрения зависимостей в конструкторе, например `ILogger`, можно просто добавить `IFeatureManager` в качестве дополнительного аргумента:

### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)
    
```csharp
using Microsoft.FeatureManagement;

public class HomeController : Controller
{
    private readonly IFeatureManager _featureManager;

    public HomeController(ILogger<HomeController> logger, IFeatureManager featureManager)
    {
        _featureManager = featureManager;
    }
}
```

### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

```csharp
using Microsoft.FeatureManagement;

public class HomeController : Controller
{
    private readonly IFeatureManager _featureManager;

    public HomeController(ILogger<HomeController> logger, IFeatureManager featureManager)
    {
        _featureManager = featureManager;
    }
}
```
    
### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

```csharp
using Microsoft.FeatureManagement;

public class HomeController : Controller
{
    private readonly IFeatureManager _featureManager;

    public HomeController(IFeatureManager featureManager)
    {
        _featureManager = featureManager;
    }
}
```

---

## <a name="feature-flag-references"></a>Ссылки на флаги функций

Определите флаги компонентов как строковые переменные, чтобы ссылаться на них из кода:

```csharp
public static class MyFeatureFlags
{
    public const string FeatureA = "FeatureA";
    public const string FeatureB = "FeatureB";
    public const string FeatureC = "FeatureC";
}
```

## <a name="feature-flag-checks"></a>Проверка флага функции

Распространенный шаблон управления компонентами — проверка того, задано ли флагу компонента значение *on*. Если значение задано, выполните раздел кода. Например:

```csharp
IFeatureManager featureManager;
...
if (await featureManager.IsEnabledAsync(MyFeatureFlags.FeatureA))
{
    // Run the following code
}
```

## <a name="controller-actions"></a>Действия контроллера

В контроллерах MVC с помощью атрибута `FeatureGate` можно управлять тем, что именно будет включено: весь класс контроллера или определенное действие. Для указанного ниже контроллера `HomeController` необходимо, чтобы флаг `FeatureA` был *включен*, прежде чем любое содержащееся в классе контроллера действие будет выполнено.

```csharp
using Microsoft.FeatureManagement.Mvc;

[FeatureGate(MyFeatureFlags.FeatureA)]
public class HomeController : Controller
{
    ...
}
```

Для выполнения указанного ниже действия `Index` требуется, чтобы флаг `FeatureA` был *включен*.

```csharp
using Microsoft.FeatureManagement.Mvc;

[FeatureGate(MyFeatureFlags.FeatureA)]
public IActionResult Index()
{
    return View();
}
```

Когда контроллер MVC или действие блокируется из-за того, что флагу компонента контроллера присвоено значение *off*, вызывается зарегистрированный интерфейс [IDisabledFeaturesHandler](/dotnet/api/microsoft.featuremanagement.mvc.idisabledfeatureshandler?preserve-view=true&view=azure-dotnet-preview). Интерфейс `IDisabledFeaturesHandler` по умолчанию возвращает клиенту код состояния 404 без текста ответа.

## <a name="mvc-views"></a>Представления MVC

Откройте файл *_ViewImports.cshtml* в каталоге *Представления* и добавьте в него вспомогательную функцию тегов диспетчера функций.

```html
@addTagHelper *, Microsoft.FeatureManagement.AspNetCore
```

В представлениях MVC с помощью тега `<feature>` можно настроить отображение содержимого на основе того, включен ли флаг функции:

```html
<feature name="FeatureA">
    <p>This can only be seen if 'FeatureA' is enabled.</p>
</feature>
```

Для отображения альтернативного содержимого при невыполнении требований можно использовать атрибут `negate`.

```html
<feature name="FeatureA" negate="true">
    <p>This will be shown if 'FeatureA' is disabled.</p>
</feature>
```

Тег функции `<feature>` также можно использовать для отображения содержимого, если включены функции в списке.

```html
<feature name="FeatureA, FeatureB" requirement="All">
    <p>This can only be seen if 'FeatureA' and 'FeatureB' are enabled.</p>
</feature>
<feature name="FeatureA, FeatureB" requirement="Any">
    <p>This can be seen if 'FeatureA', 'FeatureB', or both are enabled.</p>
</feature>
```

## <a name="mvc-filters"></a>Фильтры MVC

Фильтры MVC можно настроить таким образом, чтобы они активировались в зависимости от состояния флага функции. Эта возможность ограничена фильтрами, которые реализуют [IAsyncActionFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncactionfilter). С помощью приведенного ниже кода добавляется фильтр MVC с именем `ThirdPartyActionFilter`. Этот фильтр активируется в пределах конвейера MVC, только если флаг `FeatureA` включен.  

```csharp
using Microsoft.FeatureManagement.FeatureFilters;

IConfiguration Configuration { get; set;}

public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc(options => {
        options.Filters.AddForFeature<ThirdPartyActionFilter>(MyFeatureFlags.FeatureA);
    });
}
```

## <a name="middleware"></a>ПО промежуточного слоя

Кроме того, флаги функций позволяют добавлять ветви приложений и ПО промежуточного слоя в случае выполнения условия. С помощью приведенного ниже кода компонент ПО промежуточного слоя вставляется в конвейер запроса, только если флаг `FeatureA` включен.

```csharp
app.UseMiddlewareForFeature<ThirdPartyMiddleware>(MyFeatureFlags.FeatureA);
```

Следующий код представляет более общую возможность разветвления всего приложения на основе флага функции:

```csharp
app.UseForFeature(featureName, appBuilder => {
    appBuilder.UseMiddleware<T>();
});
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как с помощью библиотек `Microsoft.FeatureManagement` реализовать флаги функций в приложении ASP.NET Core. Подробные сведения о поддержке управления функциями в службе "Конфигурация приложений" и ASP.NET Core см. по следующим ссылкам:

* [Пример кода для флага функции ASP.NET Core](./quickstart-feature-flag-aspnet-core.md)
* [Документация по Microsoft.FeatureManagement](/dotnet/api/microsoft.featuremanagement)
* [Управление флагами компонентов](./manage-feature-flags.md)