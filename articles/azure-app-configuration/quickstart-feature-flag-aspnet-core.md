---
title: Краткое руководство по добавлению флагов функций в ASP.NET Core
description: Добавление флагов функций в приложения ASP.NET Core и управление ими с помощью Конфигурации приложений Azure
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.custom: devx-track-csharp
ms.topic: quickstart
ms.date: 09/28/2020
ms.author: alkemper
ms.openlocfilehash: 8ca4a6e78ab3c6ccf492869bb68c5296ae91de21
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99806076"
---
# <a name="quickstart-add-feature-flags-to-an-aspnet-core-app"></a>Краткое руководство. Добавление флагов функций в приложение ASP.NET Core

В рамках этого краткого руководства вы создадите комплексную реализацию управления функциями в приложении ASP.NET Core с помощью Конфигурации приложений Azure. Вы сможете использовать службу "Конфигурация приложений" для централизованного хранения всех флагов функций и управления их состояниями. 

Библиотеки управления функциями .NET Core расширяют возможности платформы за счет всесторонней поддержки флагов функций. Эти библиотеки создаются на основе системы конфигурации .NET Core. Они легко интегрируются с Конфигурацией приложений посредством поставщика конфигураций .NET Core.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/dotnet).
* [Базовый пакет SDK для .NET](https://dotnet.microsoft.com/download)

## <a name="create-an-app-configuration-store"></a>Создание хранилища Конфигурации приложений

[!INCLUDE[Azure App Configuration resource creation steps](../../includes/azure-app-configuration-create.md)]

8. Последовательно выберите **Операции** > **Диспетчер компонентов** > **Добавить**, чтобы добавить флаг функции с именем *Beta*.

    > [!div class="mx-imgBorder"]
    > ![Включение флага функции с именем Beta](media/add-beta-feature-flag.png)

    Поле **Метка** пока заполнять не нужно. Чтобы сохранить новый флаг функции, выберите **Применить**.

## <a name="create-an-aspnet-core-web-app"></a>Создание веб-приложения ASP.NET Core

Используйте [интерфейс командной строки .NET Core](/dotnet/core/tools), чтобы создать проект MVC для ASP.NET Core. Преимущество использования .NET Core CLI вместо Visual Studio заключается в том, что .NET Core CLI доступен на платформах Windows, MacOS и Linux.

В новой папке *TestFeatureFlags* выполните следующую команду, чтобы создать проект MVC для ASP.NET Core:

```dotnetcli
dotnet new mvc --no-https --output TestFeatureFlags
```

[!INCLUDE[Add Secret Manager support to an ASP.NET Core project](../../includes/azure-app-configuration-add-secret-manager.md)]

## <a name="connect-to-an-app-configuration-store"></a>Подключение к хранилищу Конфигурации приложений

1. Установите пакеты NuGet [Microsoft.Azure.AppConfiguration.AspNetCore](https://www.nuget.org/packages/Microsoft.Azure.AppConfiguration.AspNetCore) и [Microsoft.FeatureManagement.AspNetCore](https://www.nuget.org/packages/Microsoft.FeatureManagement.AspNetCore), выполнив следующие команды:

    ```dotnetcli
    dotnet add package Microsoft.Azure.AppConfiguration.AspNetCore
    ```

    ```dotnetcli
    dotnet add package Microsoft.FeatureManagement.AspNetCore
    ```

1. Эту команду необходимо выполнять в том же каталоге, где расположен *CSPROJ*-файл. Команда использует диспетчер секретов для хранения секрета с именем `ConnectionStrings:AppConfig`, в котором хранится строка подключения для вашего хранилища Конфигурации приложений. Замените заполнитель `<your_connection_string>` строкой подключения из хранилища Конфигурации приложений. Строку подключения можно найти в разделе **Ключи доступа** на портале Azure.

    ```dotnetcli
    dotnet user-secrets set ConnectionStrings:AppConfig "<your_connection_string>"
    ```

    Диспетчер секретов используется только для локальной проверки веб-приложения. При развертывании приложения в [Службе приложений Azure](https://azure.microsoft.com/services/app-service/web) для хранения строки подключения используйте параметр приложения **Строки подключения** в Службе приложений, а не диспетчер секретов.

    Доступ к этому секрету можно получить с помощью API конфигурации .NET Core. При работе с API конфигурации в имени конфигурации используется двоеточие (`:`) на всех поддерживаемых платформах. Дополнительные сведения см. в разделе [Ключи и значения конфигурации](/aspnet/core/fundamentals/configuration#configuration-keys-and-values).

1. В файле *Program.cs* обновите метод `CreateWebHostBuilder` для использования службы "Конфигурация приложений" с помощью вызова метода `AddAzureAppConfiguration`.

    > [!IMPORTANT]
    > `CreateHostBuilder` заменяет `CreateWebHostBuilder` в .NET Core 3.x. Выберите правильный синтаксис в зависимости от среды.

     #### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
                webBuilder.ConfigureAppConfiguration(config =>
                {
                    var settings = config.Build();
                    var connection = settings.GetConnectionString("AppConfig");
                    config.AddAzureAppConfiguration(options =>
                        options.Connect(connection).UseFeatureFlags());
                }).UseStartup<Startup>());
    ```

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
                webBuilder.ConfigureAppConfiguration(config =>
                {
                    var settings = config.Build();
                    var connection = settings.GetConnectionString("AppConfig");
                    config.AddAzureAppConfiguration(options =>
                        options.Connect(connection).UseFeatureFlags());
                }).UseStartup<Startup>());
    ```

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
               .ConfigureAppConfiguration(config =>
               {
                   var settings = config.Build();
                   var connection = settings.GetConnectionString("AppConfig");
                   config.AddAzureAppConfiguration(options =>
                       options.Connect(connection).UseFeatureFlags());
               }).UseStartup<Startup>();
    ```

    ---

    При предыдущем изменении [поставщик конфигурации для Конфигурации приложений](/dotnet/api/Microsoft.Extensions.Configuration.AzureAppConfiguration) зарегистрирован в API конфигурации .NET Core.

1. В файл *Startup.cs* добавьте ссылку на диспетчер функций .NET Core.

    ```csharp
    using Microsoft.FeatureManagement;
    ```

1. Обновите метод `Startup.ConfigureServices`, добавив поддержку флагов функций с помощью вызова метода `AddFeatureManagement`. При необходимости можно добавить любой фильтр для флагов функций, вызвав `AddFeatureFilter<FilterType>()`.

     #### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)

    ```csharp    
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllersWithViews();
        services.AddFeatureManagement();
    }
    ```
    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp    
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllersWithViews();
        services.AddFeatureManagement();
    }
    ```

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc()
            .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
        services.AddFeatureManagement();
    }
    ```

    ---

1. Добавьте файл *MyFeatureFlags.cs* в корневой каталог проекта с помощью следующего кода:

    ```csharp
    namespace TestFeatureFlags
    {
        public enum MyFeatureFlags
        {
            Beta
        }
    }
    ```

1. Добавьте файл *BetaController.cs* в каталог *Controllers* с помощью следующего кода:

    ```csharp
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.FeatureManagement;
    using Microsoft.FeatureManagement.Mvc;

    namespace TestFeatureFlags.Controllers
    {
        public class BetaController: Controller
        {
            private readonly IFeatureManager _featureManager;

            public BetaController(IFeatureManagerSnapshot featureManager) =>
                _featureManager = featureManager;

            [FeatureGate(MyFeatureFlags.Beta)]
            public IActionResult Index() => View();
        }
    }
    ```

1. Зарегистрируйте в файле *Views/_ViewImports.cshtml* вспомогательную функцию тегов диспетчера функций с помощью директивы `@addTagHelper`:

    ```cshtml
    @addTagHelper *, Microsoft.FeatureManagement.AspNetCore
    ```

    Приведенный выше код позволяет использовать вспомогательную функцию тега `<feature>` в *CSHTML*-файлах проекта.

1. Откройте файл *_Layout.cshtml* в каталоге *Views*\\*Shared*. Найдите штрихкод `<nav>` в `<body>` > `<header>`. Вставьте новый тег `<feature>` между пунктами навигации *Home* и *Privacy*, как показано ниже.

    :::code language="html" source="../../includes/azure-app-configuration-navbar.md" range="15-38" highlight="14-18":::

1. Создайте каталог *Views/Beta* и файл *Index.cshtml*, содержащий следующую разметку:

    ```cshtml
    @{
        ViewData["Title"] = "Beta Home Page";
    }

    <h1>This is the beta website.</h1>
    ```

## <a name="build-and-run-the-app-locally"></a>Создание и запуск приложения локально

1. Чтобы создать приложение с помощью .NET Core CLI, выполните следующую команду в оболочке командной строки:

    ```dotnetcli
    dotnet build
    ```

1. Когда создание завершится, запустите веб-приложение локально с помощью следующей команды:

    ```dotnetcli
    dotnet run
    ```

1. Откройте окно браузера и перейдите по адресу `http://localhost:5000`, который является URL-адресом по умолчанию для веб-приложения, размещенного локально. Если вы работаете в Azure Cloud Shell, нажмите кнопку **Просмотр в Интернете**, а затем — **Настроить**. При появлении запроса выберите порт 5000.

    ![Найдите кнопку "Просмотр в Интернете"](./media/quickstarts/cloud-shell-web-preview.png)

    Браузер должен отображать страницу, похожую на изображение ниже.

    :::image type="content" source="media/quickstarts/aspnet-core-feature-flag-local-before.png" alt-text="Локальное приложение быстрого запуска перед изменением" border="true":::

1. Войдите на [портал Azure](https://portal.azure.com). Щелкните **Все ресурсы** и выберите экземпляр хранилища Конфигурации приложений, который вы создали по инструкциям из краткого руководства.

1. Выберите **Диспетчер функций**. 

1. Включите флаг *Бета-версия*, установив флажок для пункта **Включено**.

1. Вернитесь в командную оболочку. Отмените запущенный процесс `dotnet`, нажав клавиши <kbd>Ctrl+C</kbd>. Перезапустите приложение с помощью `dotnet run`.

1. Обновите страницу браузера, чтобы просмотреть новые параметры конфигурации.

    :::image type="content" source="media/quickstarts/aspnet-core-feature-flag-local-after.png" alt-text="Локальное приложение быстрого запуска после изменения" border="true":::

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE[Azure App Configuration cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали хранилище Конфигурации приложений и использовали его для управления функциями веб-приложения ASP.NET Core с помощью [библиотек управления функциями](/dotnet/api/Microsoft.Extensions.Configuration.AzureAppConfiguration).

* Узнайте больше об [управлении функциями](./concept-feature-management.md).
* [Управляйте флагами функций](./manage-feature-flags.md).
* [Используйте флаги функций в приложении ASP.NET Core](./use-feature-flags-dotnet-core.md).
* [Использование динамической конфигурации в приложении ASP.NET Core](./enable-dynamic-configuration-aspnet-core.md)