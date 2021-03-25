---
title: Краткое руководство по использованию службы "Конфигурация приложений Azure" с решением "Функции Azure"| Документация Майкрософт
description: В этом кратком руководстве показано, как создать приложение Функций Azure с помощью службы "Конфигурация приложений Azure"и языка C#. Создание и подключение к хранилищу Конфигурации приложений. Локальное тестирование функции.
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.custom: devx-track-csharp
ms.topic: quickstart
ms.date: 09/28/2020
ms.author: alkemper
ms.openlocfilehash: 9d378b21132e6646329c459401255ef9a3ed9426
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98724250"
---
# <a name="quickstart-create-an-azure-functions-app-with-azure-app-configuration"></a>Краткое руководство. Создание приложения Функций Azure с использованием службы "Конфигурация приложений Azure"

В этом кратком руководстве описано, как добавить службу "Конфигурация приложений Azure" в приложение Функций Azure, чтобы обеспечить централизованное хранение всех параметров приложения и управление ими отдельно от кода.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/dotnet).
- [Visual Studio 2019](https://visualstudio.microsoft.com/vs) с рабочей нагрузкой **разработки Azure**.
- [Средства функций Azure](../azure-functions/functions-develop-vs.md#check-your-tools-version)

## <a name="create-an-app-configuration-store"></a>Создание хранилища Конфигурации приложений

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

7. Выберите **Обозреватель конфигураций** >  **+ Создать** > **Ключ-значение**, чтобы добавить следующие пары "ключ-значение":

    | Клавиши | Значение |
    |---|---|
    | TestApp:Settings:FontSize | Данные из конфигурации приложения Azure |

    Поля **Метка** и **Тип контента** пока заполнять не нужно.

8. Нажмите кнопку **Применить**.

## <a name="create-a-functions-app"></a>Создание приложения-функции

[!INCLUDE [Create a project using the Azure Functions template](../../includes/functions-vstools-create.md)]

## <a name="connect-to-an-app-configuration-store"></a>Подключение к хранилищу Конфигурации приложений
Этот проект будет использовать [внедрение зависимостей в Функциях Azure .NET](../azure-functions/functions-dotnet-dependency-injection.md) и добавит Конфигурацию приложений Azure в качестве дополнительного источника конфигурации.

1. Щелкните проект правой кнопкой мыши и выберите **Управление пакетами NuGet**. На вкладке **Обзор** найдите и добавьте в проект следующие пакеты NuGet:
   - [Microsoft.Extensions.Configuration.AzureAppConfiguration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureAppConfiguration/) 4.1.0 или более поздней версии;
   - [Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions/) 1.1.0 или более поздней версии. 

2. Добавьте новый файл *Startup.cs* с приведенным ниже кодом. В нем определен класс с именем `Startup`, который реализует абстрактный класс `FunctionsStartup`. С помощью атрибута сборки указывается имя типа, используемого при запуске Функций Azure.

    Метод `ConfigureAppConfiguration` переопределяется, при этом добавляется поставщик Конфигурации приложений Azure в качестве дополнительного источника конфигурации с помощью вызова `AddAzureAppConfiguration()`. Метод `Configure` остается пустым, так как на этом этапе не нужно регистрировать какие-либо службы.
    
    ```csharp
    using System;
    using Microsoft.Azure.Functions.Extensions.DependencyInjection;
    using Microsoft.Extensions.Configuration;

    [assembly: FunctionsStartup(typeof(FunctionApp.Startup))]

    namespace FunctionApp
    {
        class Startup : FunctionsStartup
        {
            public override void ConfigureAppConfiguration(IFunctionsConfigurationBuilder builder)
            {
                string cs = Environment.GetEnvironmentVariable("ConnectionString");
                builder.ConfigurationBuilder.AddAzureAppConfiguration(cs);
            }

            public override void Configure(IFunctionsHostBuilder builder)
            {
            }
        }
    }
    ```

3. Откройте *Function1.cs* и добавьте указанное ниже пространство имен.

    ```csharp
    using Microsoft.Extensions.Configuration;
    ```

   Добавьте конструктор для получения экземпляра `IConfiguration` с помощью внедрения зависимостей.

    ```csharp
    private readonly IConfiguration _configuration;

    public Function1(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    ```

4. Обновите метод `Run`, чтобы считать значения из конфигурации.

    ```csharp
    public async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req, ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        string keyName = "TestApp:Settings:Message";
        string message = _configuration[keyName];

        return message != null
            ? (ActionResult)new OkObjectResult(message)
            : new BadRequestObjectResult($"Please create a key-value with the key '{keyName}' in App Configuration.");
    }
    ```

   Класс `Function1` и метод `Run` не должны быть статическими. Удалите модификатор `static`, если он был создан автоматически.

## <a name="test-the-function-locally"></a>Локальное тестирование функции

1. Задайте переменную среды с именем **ConnectionString** и укажите для нее ключ доступа к хранилищу службы "Конфигурация приложений". Если вы используете командную строку Windows, выполните следующую команду и перезапустите командную строку, чтобы изменения вступили в силу:

    ```cmd
        setx ConnectionString "connection-string-of-your-app-configuration-store"
    ```

    Если вы используете Windows PowerShell, выполните следующую команду:

    ```azurepowershell
        $Env:ConnectionString = "connection-string-of-your-app-configuration-store"
    ```

    Если вы используете macOS или Linux, выполните следующую команду:

    ```bash
        export ConnectionString='connection-string-of-your-app-configuration-store'
    ```

2. Чтобы проверить работу функции, нажмите клавишу F5. Если будет предложено, примите запрос от Visual Studio на скачивание и установку **основных инструментов решения "Функции Azure" (CLI)**. Кроме того, возможно, вам понадобиться включить исключение брандмауэра, чтобы инструменты могли обрабатывать HTTP-запросы.

3. Скопируйте URL-адрес функции из выходных данных среды выполнения функций Azure.

    ![Отладки рассматриваемой в этом кратком руководстве функции в VS](./media/quickstarts/function-visual-studio-debugging.png)

4. Вставьте URL-адрес для HTTP-запроса в адресной строке браузера. На изображении ниже показан ответ в браузере на локальный запрос GET, возвращаемый функцией.

    ![Локальный запуск рассматриваемой в этом кратком руководстве функции](./media/quickstarts/dotnet-core-function-launch-local.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали хранилище службы "Конфигурация приложений" и использовали его с приложением Функций Azure с помощью [поставщика Конфигурации приложений](/dotnet/api/Microsoft.Extensions.Configuration.AzureAppConfiguration). Чтобы узнать, как настроить приложение Функций Azure для динамического обновления конфигурации, перейдите к следующему учебнику.

> [!div class="nextstepaction"]
> [Включение динамической конфигурации в Функциях Azure](./enable-dynamic-configuration-azure-functions-csharp.md)