---
title: Краткое руководство по Конфигурации приложения Azure с ASP.NET Core | Документация Майкрософт
description: Создание приложения ASP.NET Core с помощью службы "Конфигурация приложений Azure" и централизованное хранение параметров приложения и управление ими.
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.devlang: csharp
ms.custom: devx-track-csharp, contperf-fy21q1
ms.topic: quickstart
ms.date: 09/25/2020
ms.author: alkemper
ms.openlocfilehash: 353a934842fd872d48d091f1df1b6d94ac4599ff
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98663440"
---
# <a name="quickstart-create-an-aspnet-core-app-with-azure-app-configuration"></a>Краткое руководство. Создание приложения ASP.NET Core с помощью службы "Конфигурация приложений Azure"

В этом кратком руководстве описано, как использовать Конфигурацию приложений Azure, чтобы обеспечить централизованное хранение параметров приложения ASP.NET Core и управление ими. ASP.NET Core создает один объект конфигурации на основе пары "ключ-значение" с использованием параметров из одного или нескольких источников данных, указанных приложением. Эти источники данных называются *поставщиками конфигурации*. Так как в качестве поставщика конфигурации реализован клиент .NET Core службы "Конфигурация приложений", служба выглядит как другой источник данных.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/dotnet).
* [Базовый пакет SDK для .NET](https://dotnet.microsoft.com/download)

> [!TIP]
> Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять инструкции командной строки, описанные в этой статье. Она содержит предварительно установленные общие инструменты Azure, включая пакет SDK для .NET Core. Если вы вошли в подписку Azure, запустите [Azure Cloud Shell](https://shell.azure.com) на сайте shell.azure.com. Дополнительные сведения об Azure Cloud Shell см. в [нашей документации](../cloud-shell/overview.md)

## <a name="create-an-app-configuration-store"></a>Создание хранилища Конфигурации приложений

[!INCLUDE[Azure App Configuration resource creation steps](../../includes/azure-app-configuration-create.md)]

7. Выберите **Операции** > **Обозреватель конфигураций** > **Создать** > **Ключ-значение**, чтобы добавить следующие пары "ключ-значение":

    | Клавиши                                | Значение                               |
    |------------------------------------|-------------------------------------|
    | `TestApp:Settings:BackgroundColor` | *#FFF*                              |
    | `TestApp:Settings:FontColor`       | *#000*                              |
    | `TestApp:Settings:FontSize`        | *24*                                |
    | `TestApp:Settings:Message`         | *Данные из конфигурации приложения Azure* |

    Поля **Метка** и **Тип содержимого** пока заполнять не нужно. Нажмите кнопку **Применить**.

## <a name="create-an-aspnet-core-web-app"></a>Создание веб-приложения ASP.NET Core

Используйте [интерфейс командной строки .NET Core](/dotnet/core/tools), чтобы создать проект MVC для ASP.NET Core. [Azure Cloud Shell](https://shell.azure.com) предоставляет эти инструменты. Они также доступны на платформах Windows, macOS и Linux.

В новой папке *TestAppConfig* выполните следующую команду, чтобы создать проект MVC для ASP.NET Core:

```dotnetcli
dotnet new mvc --no-https --output TestAppConfig
```

[!INCLUDE[Add Secret Manager support to an ASP.NET Core project](../../includes/azure-app-configuration-add-secret-manager.md)]

## <a name="connect-to-the-app-configuration-store"></a>Подключение к хранилищу Конфигурации приложений

1. Выполните следующую команду, чтобы добавить ссылку на пакет NuGet [Microsoft.Azure.AppConfiguration.AspNetCore](https://www.nuget.org/packages/Microsoft.Azure.AppConfiguration.AspNetCore):

    ```dotnetcli
    dotnet add package Microsoft.Azure.AppConfiguration.AspNetCore
    ```

1. Эту команду необходимо выполнять в том же каталоге, где расположен *CSPROJ*-файл. Команда использует диспетчер секретов для хранения секрета с именем `ConnectionStrings:AppConfig`, в котором хранится строка подключения для вашего хранилища Конфигурации приложений. Замените заполнитель `<your_connection_string>` строкой подключения из хранилища Конфигурации приложений. Строку подключения можно найти в разделе **Ключи доступа** на портале Azure.

    ```dotnetcli
    dotnet user-secrets set ConnectionStrings:AppConfig "<your_connection_string>"
    ```

    > [!IMPORTANT]
    > Некоторые оболочки усекают строки подключения, которые не заключены в кавычки. Убедитесь, что вывод команды `dotnet user-secrets list` отображает полную строку подключения. В противном случае повторно выполните команду, заключив строку подключения в кавычки.

    Диспетчер секретов используется только для локальной проверки веб-приложения. При развертывании приложения в [Службе приложений Azure](https://azure.microsoft.com/services/app-service/web) для хранения строки подключения используйте параметр приложения **Строки подключения** в Службе приложений, а не диспетчер секретов.

    Доступ к этому секрету можно получить с помощью API конфигурации .NET Core. При работе с API конфигурации в имени конфигурации используется двоеточие (`:`) на всех поддерживаемых платформах. Дополнительные сведения см. в разделе [Ключи и значения конфигурации](/aspnet/core/fundamentals/configuration#configuration-keys-and-values).

1. В *Program.cs* добавьте ссылку на пространство имен API конфигурации .NET Core:

    ```csharp
    using Microsoft.Extensions.Configuration;
    ```

1. Обновите метод `CreateWebHostBuilder`, чтобы использовать службу "Конфигурация приложений", путем вызова метода `AddAzureAppConfiguration`.

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
                    config.AddAzureAppConfiguration(connection);
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
                    config.AddAzureAppConfiguration(connection);
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
                config.AddAzureAppConfiguration(connection);
            })
            .UseStartup<Startup>();
    ```

    ---

    При предыдущем изменении [поставщик конфигурации для Конфигурации приложений](/dotnet/api/Microsoft.Extensions.Configuration.AzureAppConfiguration) зарегистрирован в API конфигурации .NET Core.

## <a name="read-from-the-app-configuration-store"></a>Чтение из хранилища конфигураций приложений

Выполните следующие действия, чтобы прочитать и отобразить значения, хранящиеся в хранилище конфигураций приложений. Для доступа к хранилищу будет использоваться API конфигурации .NET Core. Для вывода значений ключей будет использоваться синтаксис Razor.

Откройте файл *\<app root>/Views/Home/Index.cshtml* и замените его содержимое следующим кодом:

```cshtml
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration

<style>
    body {
        background-color: @Configuration["TestApp:Settings:BackgroundColor"]
    }
    h1 {
        color: @Configuration["TestApp:Settings:FontColor"];
        font-size: @Configuration["TestApp:Settings:FontSize"]px;
    }
</style>

<h1>@Configuration["TestApp:Settings:Message"]</h1>
```

В приведенном выше коде ключи хранилища конфигурации приложений используются следующим образом:

* Значение ключа `TestApp:Settings:BackgroundColor` присваивается свойству `background-color` CSS.
* Значение ключа `TestApp:Settings:FontColor` присваивается свойству `color` CSS.
* Значение ключа `TestApp:Settings:FontSize` присваивается свойству `font-size` CSS.
* Значение ключа `TestApp:Settings:Message` отображается в качестве заголовка.

## <a name="build-and-run-the-app-locally"></a>Создание и запуск приложения локально

1. Чтобы создать приложение с помощью .NET Core CLI, перейдите в корневой каталог проекта. Выполните следующую команду в командной оболочке:

    ```dotnetcli
    dotnet build
    ```

1. Когда создание завершится, запустите веб-приложение локально с помощью следующей команды:

    ```dotnetcli
    dotnet run
    ```

1. Если вы работаете на локальном компьютере, перейдите по адресу `http://localhost:5000` в браузере. Этот адрес является URL-адресом по умолчанию для локально размещенного веб-приложения. Если вы работаете в Azure Cloud Shell, нажмите кнопку **Просмотр в Интернете**, а затем — **Настроить**.

    ![Найдите кнопку "Просмотр в Интернете"](./media/quickstarts/cloud-shell-web-preview.png)

    При появлении запроса на настройку порта для предварительного просмотра введите *5000* и выберите **Открыть и перейти**. На веб-странице будет указано "Данные из конфигурации приложения Azure".

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE[Azure App Configuration cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как:

* Подготовлено новое хранилище конфигураций приложений.
* Зарегистрирован поставщик конфигурации .NET Core для хранилища конфигураций приложений.
* Прочитаны ключи хранилища конфигурации приложений с поставщиком конфигурации.
* Отображаются значения ключей хранилища конфигураций приложений с помощью синтаксиса Razor.

Чтобы узнать, как настроить приложение ASP.NET Core для динамического обновления параметров конфигурации, перейдите к следующему учебнику.

> [!div class="nextstepaction"]
> [Включение динамической конфигурации](./enable-dynamic-configuration-aspnet-core.md)