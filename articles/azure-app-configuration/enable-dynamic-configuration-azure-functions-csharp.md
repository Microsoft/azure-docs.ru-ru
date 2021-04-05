---
title: Руководство по использованию динамической конфигурации службы "Конфигурация приложений Azure" в приложении Функций Azure | Документация Майкрософт
description: Из этого учебника вы узнаете, как динамически обновлять данные конфигурации для приложений Функций Azure.
services: azure-app-configuration
documentationcenter: ''
author: zhenlan
manager: qingye
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 11/17/2019
ms.author: zhenlwa
ms.custom: devx-track-csharp, azure-functions
ms.tgt_pltfrm: Azure Functions
ms.openlocfilehash: add4b54adb02db09536f4e56a7f039c46245c182
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97963570"
---
# <a name="tutorial-use-dynamic-configuration-in-an-azure-functions-app"></a>Руководство по использованию динамической конфигурации в приложении Функций Azure

Конфигурация .NET службы "Конфигурация приложений" поддерживает кэширование и обновление, динамически управляемое активностью приложения. Из этого руководства вы узнаете, как реализовать динамические обновления конфигурации в коде. При этом в качестве основы используется код приложения Функций Azure, представленный в кратких руководствах. Прежде чем продолжить, ознакомьтесь со статьей [Краткое руководство. Создание приложения Функций Azure с помощью службы "Конфигурация приложений Azure"](./quickstart-azure-functions-csharp.md).

В этом руководстве описано следующее:

> [!div class="checklist"]
> * настройка в приложении Функций Azure для обновления конфигурации при изменении данных в хранилище службы "Конфигурации приложений";
> * добавление последней конфигурации в вызовы Функций Azure.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
- [Visual Studio 2019](https://visualstudio.microsoft.com/vs) с рабочей нагрузкой **разработки Azure**.
- [Инструменты Функций Azure](../azure-functions/functions-develop-vs.md#check-your-tools-version)
- Ознакомьтесь со статьей [Краткое руководство. Создание приложения Функций Azure с помощью службы "Конфигурация приложений Azure"](./quickstart-azure-functions-csharp.md).

## <a name="reload-data-from-app-configuration"></a>Перезагрузка данных из App Configuration

1. Откройте файл *Startup.cs* и измените метод `ConfigureAppConfiguration`. 

   Метод `ConfigureRefresh` регистрирует параметр, который будет проверяться на наличие изменений при каждом запуске обновления в приложении, что вы сделаете на следующем шаге при добавлении `_configurationRefresher.TryRefreshAsync()`. Параметр `refreshAll` указывает поставщику Конфигурации приложений перезагрузить всю конфигурацию при обнаружении изменений в зарегистрированном параметре.

    Срок действия кэша для всех параметров, зарегистрированных для обновления, по умолчанию составляет 30 секунд. Его можно обновить, вызвав метод `AzureAppConfigurationRefreshOptions.SetCacheExpiration`.

    ```csharp
    public override void ConfigureAppConfiguration(IFunctionsConfigurationBuilder builder)
    {
        builder.ConfigurationBuilder.AddAzureAppConfiguration(options =>
        {
            options.Connect(Environment.GetEnvironmentVariable("ConnectionString"))
                   // Load all keys that start with `TestApp:`
                   .Select("TestApp:*")
                   // Configure to reload configuration if the registered 'Sentinel' key is modified
                   .ConfigureRefresh(refreshOptions =>
                      refreshOptions.Register("TestApp:Settings:Sentinel", refreshAll: true));
        });
    }
    ```

   > [!TIP]
   > При обновлении нескольких значений ключа в Конфигурации приложений обычно не требуется, чтобы приложение перезагружало конфигурацию до внесения всех изменений. Вы можете зарегистрировать ключ **Sentinel** и обновить его только после внесения всех остальных изменений конфигурации. Это помогает обеспечить согласованность конфигурации в приложении.

2. Обновите метод `Configure`, чтобы службы Конфигурации приложений Azure были доступны посредством внедрения зависимостей.

    ```csharp
    public override void Configure(IFunctionsHostBuilder builder)
    {
        builder.Services.AddAzureAppConfiguration();
    }
    ```

3. Откройте файл *Function1.cs* и добавьте указанные ниже пространства имен.

    ```csharp
    using System.Linq;
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    ```

   Обновите конструктор, чтобы получить экземпляр `IConfigurationRefresherProvider` посредством внедрения зависимостей, благодаря чему можно получить экземпляр `IConfigurationRefresher`.

    ```csharp
    private readonly IConfiguration _configuration;
    private readonly IConfigurationRefresher _configurationRefresher;

    public Function1(IConfiguration configuration, IConfigurationRefresherProvider refresherProvider)
    {
        _configuration = configuration;
        _configurationRefresher = refresherProvider.Refreshers.First();
    }
    ```

4. Обновите метод `Run` и оповещение об обновлениях конфигурации, используя метод `TryRefreshAsync` в начале вызова функций. Это не будет работать, если время истечения срока действия кэша не достигнуто. Удалите оператор `await`, если вы предпочитаете, чтобы конфигурация обновлялась без блокировки текущего вызова Функций. В этом случае при последующих вызовах Функций будет поступать обновленное значение.

    ```csharp
    public async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req, ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        await _configurationRefresher.TryRefreshAsync(); 

        string keyName = "TestApp:Settings:Message";
        string message = _configuration[keyName];
            
        return message != null
            ? (ActionResult)new OkObjectResult(message)
            : new BadRequestObjectResult($"Please create a key-value with the key '{keyName}' in App Configuration.");
    }
    ```

## <a name="test-the-function-locally"></a>Локальное тестирование функции

1. Задайте переменную среды с именем **ConnectionString** и укажите для нее ключ доступа к хранилищу конфигураций приложений. Если вы используете командную строку Windows, выполните следующую команду и перезапустите командную строку, чтобы изменения вступили в силу:

    ```console
    setx ConnectionString "connection-string-of-your-app-configuration-store"
    ```

    Если вы используете Windows PowerShell, выполните следующую команду:

    ```powershell
    $Env:ConnectionString = "connection-string-of-your-app-configuration-store"
    ```

    Если вы используете macOS или Linux, выполните следующую команду:

    ```console
    export ConnectionString='connection-string-of-your-app-configuration-store'
    ```

2. Чтобы проверить работу функции, нажмите клавишу F5. Если будет предложено, примите запрос от Visual Studio на скачивание и установку **основных инструментов решения "Функции Azure" (CLI)**. Кроме того, возможно, вам понадобиться включить исключение брандмауэра, чтобы инструменты могли обрабатывать HTTP-запросы.

3. Скопируйте URL-адрес функции из выходных данных среды выполнения функций Azure.

    ![Отладки рассматриваемой в этом кратком руководстве функции в VS](./media/quickstarts/function-visual-studio-debugging.png)

4. Вставьте URL-адрес для HTTP-запроса в адресной строке браузера. На изображении ниже показан ответ в браузере на локальный запрос GET, возвращаемый функцией.

    ![Локальный запуск рассматриваемой в этом кратком руководстве функции](./media/quickstarts/dotnet-core-function-launch-local.png)

5. Войдите на [портал Azure](https://portal.azure.com). Щелкните **Все ресурсы** и выберите хранилище Конфигурации приложений, которое вы создали по инструкциям из краткого руководства.

6. Выберите **Обозреватель конфигураций** и измените значения следующего ключа:

    | Ключ | Значение |
    |---|---|
    | TestApp:Settings:FontSize | Данные из конфигурации приложений Azure. Обновлено |

   Затем создайте ключ Sentinel или измените его значение, если он уже существует, например:

    | Ключ | Значение |
    |---|---|
    | TestApp:Settings:Sentinel | Версия 1 |


7. Обновите приложение браузера несколько раз. Если истечение срока действия кэша происходит через 30 секунд, на странице будет отображаться ответ на вызов функций с обновленным значением.

    ![Локальное обновление рассматриваемой в этом кратком руководстве функции](./media/quickstarts/dotnet-core-function-refresh-local.png)

> [!NOTE]
> Пример кода, используемый в этом руководстве, можно скачать в [репозитории GitHub](https://github.com/Azure/AppConfiguration/tree/master/examples/DotNetCore/AzureFunction).

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого учебника вы включили в приложении Функций Azure динамическое обновление параметров конфигурации из службы "Конфигурация приложения". Чтобы узнать, как с помощью удостоверения, управляемого Azure, упростить доступ к службе "Конфигурация приложений Azure", перейдите к следующему учебнику.

> [!div class="nextstepaction"]
> [Руководство по интеграции с управляемыми удостоверениями Azure](./howto-integrate-azure-managed-service-identity.md)
