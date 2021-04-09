---
title: Руководство. Использование динамической конфигурации в службе "Конфигурация приложений" в ASP.NET Core
titleSuffix: Azure App Configuration
description: Из этого руководства вы узнаете, как динамически обновлять данные конфигурации для приложений ASP.NET Core.
services: azure-app-configuration
documentationcenter: ''
author: AlexandraKemperMS
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 09/1/2020
ms.author: alkemper
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 083bd56b2b211d11206a277bf31eea797b37cdb9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99979935"
---
# <a name="tutorial-use-dynamic-configuration-in-an-aspnet-core-app"></a>Руководство. Использование динамической конфигурации в приложении ASP.NET Core

ASP.NET Core имеет подключаемую систему конфигурации, которая может считывать данные конфигурации из различных источников. ASP.NET Core может динамически обрабатывать изменения без необходимости перезапускать приложение. ASP.NET Core поддерживает привязку параметров конфигурации к строго типизированным классам .NET. ASP.NET Core внедряет их в код, используя различные шаблоны `IOptions<T>`. Один из таких шаблонов (`IOptionsSnapshot<T>`) автоматически перезагружает конфигурацию приложения в случае изменения базовых данных. Вы можете внедрить шаблон `IOptionsSnapshot<T>` в контроллеры вашего приложения, чтобы получить доступ к самой последней конфигурации в Azure App Configuration.

Вы также можете настроить клиентскую библиотеку конфигурации приложений ASP.NET Core, чтобы выполнить динамическое обновление набора параметров конфигурации с помощью ПО промежуточного слоя. Параметры конфигурации обновляются в хранилище конфигураций каждый раз, пока веб-приложение получает запросы.

Служба "Конфигурация приложений" кэширует каждый параметр, чтобы избежать слишком большого количества обращений к хранилищу конфигураций. Операция обновления ожидает, пока не истечет срок действия кэшированного значения параметра, даже если его значение изменяется в хранилище конфигураций. Срок действия кэша по умолчанию составляет 30 секунд. При необходимости этот срок действия можно переопределить.

Из этого руководства вы узнаете, как реализовать динамические обновления конфигурации в коде. При этом в качестве основы используется код веб-приложения, представленный в кратких руководствах. Прежде чем продолжить, ознакомьтесь со статьей [Краткое руководство. Создание приложения ASP.NET Core с помощью службы "Конфигурация приложений Azure"](./quickstart-aspnet-core-app.md).

Вы можете выполнять шаги в этом учебнике с помощью любого редактора кода. [Visual Studio Code](https://code.visualstudio.com/) является отличным вариантом, который доступен на платформах Windows, macOS и Linux.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * настройка приложения на обновление конфигурации при изменении данных в хранилище конфигураций приложений;
> * внедрение последней конфигурации в контроллеры приложения.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником установите [пакет SDK для .NET Core](https://dotnet.microsoft.com/download).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

Прежде чем продолжить, ознакомьтесь со статьей [Краткое руководство. Создание приложения ASP.NET Core с помощью службы "Конфигурация приложений Azure"](./quickstart-aspnet-core-app.md).

## <a name="add-a-sentinel-key"></a>Добавление ключа Sentinel

Ключ *Sentinel* — это специальный ключ для выявления изменения конфигурации. Ваше приложение отслеживает ключ Sentinel на наличие изменений. При обнаружении изменения все значения конфигурации обновляются. Этот подход сокращает общее число запросов, выполняемых вашим приложением к службе "Конфигурация приложений", по сравнению с отслеживанием всех ключей на наличие изменений.

1. На портале Azure выберите **Обозреватель конфигураций > Создать > Пара "ключ-значение"** .
1. В поле **Ключ** введите *TestApp:Settings:Sentinel*. В поле **Значение** введите 1. Оставьте поля **Метка** и **Тип содержимого** пустыми.
1. Нажмите кнопку **Применить**.

> [!NOTE]
> Если вы не используете ключ Sentinel, необходимо вручную зарегистрировать каждый ключ, который вы хотите отслеживать.

## <a name="reload-data-from-app-configuration"></a>Перезагрузка данных из App Configuration

1. Добавьте ссылку на пакет NuGet `Microsoft.Azure.AppConfiguration.AspNetCore`, выполнив следующую команду:

    ```dotnetcli
    dotnet add package Microsoft.Azure.AppConfiguration.AspNetCore
    ```

1. Откройте файл *Program.cs* и обновите метод `CreateWebHostBuilder`, чтобы добавить метод `config.AddAzureAppConfiguration()`.

   #### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
                webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
                {
                    var settings = config.Build();
                    config.AddAzureAppConfiguration(options =>
                    {
                        options.Connect(settings["ConnectionStrings:AppConfig"])
                               .ConfigureRefresh(refresh =>
                                    {
                                        refresh.Register("TestApp:Settings:Sentinel", refreshAll: true)
                                               .SetCacheExpiration(new TimeSpan(0, 5, 0));
                                    });
                    });
                })
            .UseStartup<Startup>());
    ```   
    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
                webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
                {
                    var settings = config.Build();
                    config.AddAzureAppConfiguration(options =>
                    {
                        options.Connect(settings["ConnectionStrings:AppConfig"])
                               .ConfigureRefresh(refresh =>
                                    {
                                        refresh.Register("TestApp:Settings:Sentinel", refreshAll: true)
                                               .SetCacheExpiration(new TimeSpan(0, 5, 0));
                                    });
                    });
                })
            .UseStartup<Startup>());
    ```
    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();

                config.AddAzureAppConfiguration(options =>
                {
                    options.Connect(settings["ConnectionStrings:AppConfig"])
                           .ConfigureRefresh(refresh =>
                                {
                                    refresh.Register("TestApp:Settings:Sentinel", refreshAll: true)
                                           .SetCacheExpiration(new TimeSpan(0, 5, 0));
                                });
                });
            })
            .UseStartup<Startup>();
    ```
    ---

    Метод `ConfigureRefresh` используется, чтобы указать параметры, используемые для обновления данных конфигурации в хранилище службы "Конфигурация приложений" при запуске операции обновления. Параметр `refreshAll` метода `Register` указывает, что при изменении ключа Sentinel нужно обновлять все значения конфигурации.

    Кроме того, метод `SetCacheExpiration` переопределяет срок действия кэша по умолчанию (30 секунд), устанавливая 5-минутный интервал. За счет этого сокращается количество запросов к службе "Конфигурация приложений".

    > [!NOTE]
    > Возможно, для тестирования потребуется сократить срок действия кэша.

    Чтобы непосредственно запускать операцию обновления, вам нужно настроить ПО промежуточного слоя для приложения, чтобы при любых изменениях данные конфигурации обновлялись. Настройка будет описана на следующем этапе.

2. Добавьте файл *Settings.cs* в каталог Controllers, который определяет и реализует новый класс `Settings`. Замените пространство имен именем проекта. 

    ```csharp
    namespace TestAppConfig
    {
        public class Settings
        {
            public string BackgroundColor { get; set; }
            public long FontSize { get; set; }
            public string FontColor { get; set; }
            public string Message { get; set; }
        }
    }
    ```

3. Откройте файл *Startup.cs* и укажите `IServiceCollection.Configure<T>` в методе `ConfigureServices`, чтобы создать привязку между данными конфигурации и классом `Settings`.

    #### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<Settings>(Configuration.GetSection("TestApp:Settings"));
        services.AddControllersWithViews();
        services.AddAzureAppConfiguration();
    }
    ```
    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<Settings>(Configuration.GetSection("TestApp:Settings"));
        services.AddControllersWithViews();
        services.AddAzureAppConfiguration();
    }
    ```
    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<Settings>(Configuration.GetSection("TestApp:Settings"));
        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
        services.AddAzureAppConfiguration();
    }
    ```
    ---
    > [!Tip]
    > Дополнительные сведения о шаблоне параметров, используемых при чтении значений конфигурации, см. в статье [Шаблон параметров в ASP.NET Core](/aspnet/core/fundamentals/configuration/options).

4. Обновите метод `Configure`, добавив ПО промежуточного слоя `UseAzureAppConfiguration`, чтобы обновлять зарегистрированные для обновления параметры конфигурации, в то время как веб-приложение ASP.NET Core продолжает получать запросы.


    #### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                app.UseHsts();
            }

            // Add the following line:
            app.UseAzureAppConfiguration();

            app.UseHttpsRedirection();
            
            app.UseStaticFiles();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(
                    name: "default",
                    pattern: "{controller=Home}/{action=Index}/{id?}");
            });
    }
    ```
    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                app.UseHsts();
            }

            // Add the following line:
            app.UseAzureAppConfiguration();

            app.UseHttpsRedirection();
            
            app.UseStaticFiles();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(
                    name: "default",
                    pattern: "{controller=Home}/{action=Index}/{id?}");
            });
    }
    ```
    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseAzureAppConfiguration();

        services.Configure<CookiePolicyOptions>(options =>
        {
            options.CheckConsentNeeded = context => true;
            options.MinimumSameSitePolicy = SameSiteMode.None;
        });

        app.UseMvc();
    }
    ```
    ---
    
    ПО промежуточного слоя использует обновление конфигурации, заданное в методе `AddAzureAppConfiguration`, в `Program.cs`, чтобы активировать обновление для каждого запроса, полученного веб-приложением ASP.NET Core. Операция обновления запускается для каждого запроса, а клиентская библиотека проверяет, не истек ли срок действия кэшированного значения для зарегистрированного параметра конфигурации. Если срок действия истек, значение параметра обновляется.

    > [!NOTE]
    > Если вы хотите обеспечить обновление конфигурации, как можно раньше добавьте ПО промежуточного слоя в конвейер запросов, чтобы его работа не прерывалась другим ПО промежуточного слоя в приложении.

## <a name="use-the-latest-configuration-data"></a>Использование последних данных конфигурации

1. Откройте файл *HomeController.cs* в каталоге "Контроллеры" и добавьте в него ссылку на пакет `Microsoft.Extensions.Options`.

    ```csharp
    using Microsoft.Extensions.Options;
    ```

2. Обновите класс `HomeController` для получения класса `Settings` путем внедрения зависимости и воспользуйтесь его значениями.

 #### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)

```csharp
    public class HomeController : Controller
    {
        private readonly Settings _settings;
        private readonly ILogger<HomeController> _logger;

        public HomeController(ILogger<HomeController> logger, IOptionsSnapshot<Settings> settings)
        {
            _logger = logger;
            _settings = settings.Value;
        }

        public IActionResult Index()
        {
            ViewData["BackgroundColor"] = _settings.BackgroundColor;
            ViewData["FontSize"] = _settings.FontSize;
            ViewData["FontColor"] = _settings.FontColor;
            ViewData["Message"] = _settings.Message;

            return View();
        }

        // ...
    }
```
#### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

```csharp
    public class HomeController : Controller
    {
        private readonly Settings _settings;
        private readonly ILogger<HomeController> _logger;

        public HomeController(ILogger<HomeController> logger, IOptionsSnapshot<Settings> settings)
        {
            _logger = logger;
            _settings = settings.Value;
        }

        public IActionResult Index()
        {
            ViewData["BackgroundColor"] = _settings.BackgroundColor;
            ViewData["FontSize"] = _settings.FontSize;
            ViewData["FontColor"] = _settings.FontColor;
            ViewData["Message"] = _settings.Message;

            return View();
        }

        // ...
    }
```
#### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

```csharp
    public class HomeController : Controller
    {
        private readonly Settings _settings;
        public HomeController(IOptionsSnapshot<Settings> settings)
        {
            _settings = settings.Value;
        }

        public IActionResult Index()
        {
            ViewData["BackgroundColor"] = _settings.BackgroundColor;
            ViewData["FontSize"] = _settings.FontSize;
            ViewData["FontColor"] = _settings.FontColor;
            ViewData["Message"] = _settings.Message;

            return View();
        }
    }
```
---



3. Откройте файл *Index.cshtml* в каталоге "Представления > Главная" и замените его содержимое следующим сценарием.

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <style>
        body {
            background-color: @ViewData["BackgroundColor"]
        }
        h1 {
            color: @ViewData["FontColor"];
            font-size: @ViewData["FontSize"]px;
        }
    </style>
    <head>
        <title>Index View</title>
    </head>
    <body>
        <h1>@ViewData["Message"]</h1>
    </body>
    </html>
    ```

## <a name="build-and-run-the-app-locally"></a>Создание и запуск приложения локально

1. Чтобы создать приложение с помощью .NET Core CLI, выполните следующую команду в оболочке командной строки:

    ```console
        dotnet build
    ```

1. Когда создание завершится, запустите веб-приложение локально с помощью следующей команды:

    ```console
        dotnet run
    ```

1. Откройте окно браузера и перейдите по URL-адресу, указанному в выходных данных `dotnet run`.

    ![Локальный запуск приложения быстрого запуска](./media/quickstarts/aspnet-core-app-launch-local-before.png)

1. Войдите на [портал Azure](https://portal.azure.com). Щелкните **Все ресурсы** и выберите экземпляр хранилища Конфигурации приложений, который вы создали по инструкциям из краткого руководства.

1. Выберите **Configuration Explorer** (Обозреватель конфигураций) и измените значения следующих ключей.

    | Клавиши | Значение |
    |---|---|
    | TestApp:Settings:BackgroundColor | green |
    | TestApp:Settings:FontColor | lightGray |
    | TestApp:Settings:FontSize | Данные из Azure App Configuration — теперь с обновлениями в реальном времени! |
    | TestApp:Settings:Sentinel | 2 |

1. Обновите страницу браузера, чтобы просмотреть новые параметры конфигурации. Возможно, это потребуется сделать несколько раз для отражения изменений. Или же измените частоту автоматического обновления на значение, не превышающее 5 мин. 

    ![Локальный запуск обновленного приложения быстрого запуска](./media/quickstarts/aspnet-core-app-launch-local-after.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого учебника вы включили в веб-приложении ASP.NET Core динамическое обновление параметров конфигурации из службы "Конфигурация приложения". Чтобы узнать, как с помощью удостоверения, управляемого Azure, упростить доступ к службе "Конфигурация приложений Azure", перейдите к следующему руководству.

> [!div class="nextstepaction"]
> [Руководство по интеграции с управляемыми удостоверениями Azure](./howto-integrate-azure-managed-service-identity.md)
