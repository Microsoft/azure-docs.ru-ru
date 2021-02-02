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
ms.openlocfilehash: 2f141b896ef11fecdf156d062a78252ce6f7ffb3
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2021
ms.locfileid: "98734988"
---
# <a name="tutorial-use-feature-flags-in-an-aspnet-core-app"></a>Руководство по использованию флагов функций в приложении ASP.NET Core

Библиотеки управления функциями .NET Core обеспечивают идиоматическую поддержку реализации флагов функций в приложении .NET или ASP.NET Core. Эти библиотеки позволяют декларативно добавлять флаги функций в код, чтобы вам не пришлось записывать все операторы `if` для них вручную.

Библиотеки управления функциями также управляют жизненным циклом флагов функций в фоновом режиме. Например, библиотеки отвечают за обновление и кэширование состояний флагов или гарантируют неизменность состояния флага во время вызова запроса. Кроме того, библиотека ASP.NET Core предоставляет готовое решение по интеграции, включая действия контроллера MVC, представления, маршруты и ПО промежуточного слоя.

В [кратком руководстве по добавлению флагов функций в приложение ASP.NET Core](./quickstart-feature-flag-aspnet-core.md) описано несколько использующихся при этом методов. В нашем руководстве такие методы рассматриваются более подробно. Полную справочную документацию см. в [документации по управлению функциями ASP.NET Core](/dotnet/api/microsoft.featuremanagement).

В этом учебнике рассматривается следующее.

> [!div class="checklist"]
> * Добавить флаги функций в ключевых частях приложения для управления доступностью функций.
> * Как выполнить интеграцию с Конфигурацией приложений при использовании этой службы для управления флагами функций.

## <a name="set-up-feature-management"></a>Настройка управления функциями

Чтобы использовать диспетчер функций .NET Core, добавьте ссылку на пакеты NuGet `Microsoft.FeatureManagement.AspNetCore` и `Microsoft.FeatureManagement`.
Диспетчер функций .NET Core `IFeatureManager` возвращает флаги функций из собственной системы конфигурации платформы. Таким образом, вы можете определить флаги функций приложения, используя любой источник конфигурации, поддерживаемый .NET Core, включая локальный файл *appsettings.json* или переменные среды. В `IFeatureManager` используется внедрение зависимостей .NET Core. Вы можете зарегистрировать службы управления функциями с помощью стандартных соглашений:

```csharp
using Microsoft.FeatureManagement;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement();
    }
}
```

По умолчанию диспетчер функций извлекает флаги функций из раздела `"FeatureManagement"` данных конфигурации .NET Core. В следующем примере диспетчер функций считывает данные из другого раздела под названием `"MyFeatureFlags"`:

```csharp
using Microsoft.FeatureManagement;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement(options =>
        {
                options.UseConfiguration(Configuration.GetSection("MyFeatureFlags"));
        });
    }
}
```

При использовании фильтров в флагах функций вам необходимо включить дополнительную библиотеку и зарегистрировать ее. В следующем примере показано, как использовать встроенный фильтр функций под названием `PercentageFilter`:

```csharp
using Microsoft.FeatureManagement;
using Microsoft.FeatureManagement.FeatureFilters;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement()
                .AddFeatureFilter<PercentageFilter>();
    }
}
```

Рекомендуем хранить флаги функций вне приложения и управлять ими отдельно. Так вы можете изменять состояния флагов в любое время и немедленно применять эти изменения в приложении. Конфигурация приложений — это централизованное расположение для организации всех флагов функций и управления такими флагами с помощью выделенного пользовательского интерфейса портала. Конфигурация приложений также предоставляет приложению флаги непосредственно через клиентские библиотеки .NET Core.

Самый простой способ подключить приложение ASP.NET Core к Конфигурации приложений — использовать поставщика конфигураций `Microsoft.Azure.AppConfiguration.AspNetCore`. Чтобы использовать этот пакет NuGet, выполните следующие действия.

1. Откройте файл *Program.cs* и добавьте следующий код:

   ```csharp
   using Microsoft.Extensions.Configuration.AzureAppConfiguration;

   public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
       WebHost.CreateDefaultBuilder(args)
              .ConfigureAppConfiguration((hostingContext, config) => {
                  var settings = config.Build();
                  config.AddAzureAppConfiguration(options => {
                      options.Connect(settings["ConnectionStrings:AppConfig"])
                             .UseFeatureFlags();
                   });
              })
              .UseStartup<Startup>();
   ```

2. Откройте файл *Startup.cs* и обновите метод `Configure` и `ConfigureServices`, чтобы добавить ПО промежуточного слоя с именем `UseAzureAppConfiguration`. Это ПО позволяет периодически обновлять значения флагов функций, пока веб-приложение ASP.NET Core продолжает получать запросы.

   ```csharp
   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
       app.UseAzureAppConfiguration();
       app.UseMvc();
   }
   ```

   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddAzureAppConfiguration();
   }
   ```
   
Значения флагов функций должны изменяться со временем. По умолчанию значения флагов функций кэшируются на 30 секунд, поэтому операция обновления, запущенная в тот момент, когда ПО промежуточного слоя получает запрос, не обновит значение, пока не истечет срок действия кэшированного значения. В приведенном ниже коде показано, как изменить срок действия кэша или интервал опроса на 5 минут в вызове `options.UseFeatureFlags()`.

```csharp
config.AddAzureAppConfiguration(options => {
    options.Connect(settings["ConnectionStrings:AppConfig"])
           .UseFeatureFlags(featureFlagOptions => {
                featureFlagOptions.CacheExpirationTime = TimeSpan.FromMinutes(5);
           });
});
```

## <a name="feature-flag-declaration"></a>Объявление флага функции

Каждый флаг функции состоит из двух частей: имени и списка из одного или нескольких фильтров, использующихся, чтобы оценить, *включена* ли функция (то есть, если его значение равно `True`). Фильтр определяет вариант использования, при котором должна быть включена функция.

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
* `FeatureC` указывает фильтр с именем `Percentage` и свойством `Parameters`. `Percentage` — это настраиваемый фильтр. В этом примере `Percentage` задает 50-процентную вероятность того, что флаг `FeatureC` может быть *включен*.

## <a name="feature-flag-references"></a>Ссылки на флаги функций

Чтобы можно было без труда ссылаться на флаги функций в коде, нужно определить их в виде переменных `enum`:

```csharp
public enum MyFeatureFlags
{
    FeatureA,
    FeatureB,
    FeatureC
}
```

## <a name="feature-flag-checks"></a>Проверка флага функции

В базовой схеме управления функциями сначала проверяется, *включен* ли флаг функции. Если это так, диспетчер функций выполняет действия, содержащиеся в функции. Пример:

```csharp
IFeatureManager featureManager;
...
if (await featureManager.IsEnabledAsync(nameof(MyFeatureFlags.FeatureA)))
{
    // Run the following code
}
```

## <a name="dependency-injection"></a>Внедрение зависимостей

В MVC ASP.NET Core можно получить доступ к диспетчеру функций `IFeatureManager` с помощью внедрения зависимостей:

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

## <a name="controller-actions"></a>Действия контроллера

В контроллерах MVC с помощью атрибута `FeatureGate` можно управлять объектом включения: весь класс контроллера или определенное действие. Для указанного ниже контроллера `HomeController` необходимо, чтобы флаг `FeatureA` был *включен*, прежде чем любое содержащееся в классе контроллера действие будет выполнено.

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

Когда контроллер MVC или действие блокируется из-за того, что флаг функции контроллера *отключен*, вызывается зарегистрированный интерфейс `IDisabledFeaturesHandler`. Интерфейс `IDisabledFeaturesHandler` по умолчанию возвращает клиенту код состояния 404 без текста ответа.

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

Фильтры MVC можно настроить таким образом, чтобы они активировались в зависимости от состояния флага функции. С помощью приведенного ниже кода добавляется фильтр MVC с именем `SomeMvcFilter`. Этот фильтр активируется в пределах конвейера MVC, только если флаг `FeatureA` включен. Эта возможность ограничена `IAsyncActionFilter`. 

```csharp
using Microsoft.FeatureManagement.FeatureFilters;

IConfiguration Configuration { get; set;}

public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc(options => {
        options.Filters.AddForFeature<SomeMvcFilter>(nameof(MyFeatureFlags.FeatureA));
    });
}
```

## <a name="middleware"></a>ПО промежуточного слоя

Кроме того, флаги функций позволяют добавлять ветви приложений и ПО промежуточного слоя в случае выполнения условия. С помощью приведенного ниже кода компонент ПО промежуточного слоя вставляется в конвейер запроса, только если флаг `FeatureA` включен.

```csharp
app.UseMiddlewareForFeature<ThirdPartyMiddleware>(nameof(MyFeatureFlags.FeatureA));
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
