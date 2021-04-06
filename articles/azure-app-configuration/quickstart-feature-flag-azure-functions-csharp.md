---
title: Краткое руководство по добавлению флагов функций в Функции Azure | Документация Майкрософт
description: В этом кратком руководстве приведены сведения об использовании Функции Azure с флагами функций из службы "Конфигурации приложений Azure" и выполнении локальной проверки функций.
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.custom: devx-track-csharp
ms.topic: quickstart
ms.date: 8/26/2020
ms.author: alkemper
ms.openlocfilehash: 96efc0ea6300e482ddeeda8fa177847f02b7e126
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98724260"
---
# <a name="quickstart-add-feature-flags-to-an-azure-functions-app"></a>Краткое руководство. Добавление флагов функций в приложение "Функции Azure"

При работе с этим кратким руководством вы создадите приложение Функций Azure и используете в нем флаги функций. Вы будете осуществлять управление функциями из службы "Конфигурация приложений Azure" для централизованного хранения всех флагов функций и управления их состояниями.

Библиотеки управления функциями .NET расширяют возможности платформы за счет поддержки флагов функций. Эти библиотеки создаются на основе системы конфигурации .NET. Они интегрируются со службой "Конфигурация приложений" посредством поставщика конфигураций .NET.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
- [Visual Studio 2019](https://visualstudio.microsoft.com/vs) с рабочей нагрузкой **разработки Azure**.
- [Средства функций Azure](../azure-functions/functions-develop-vs.md#check-your-tools-version)

## <a name="create-an-app-configuration-store"></a>Создание хранилища Конфигурации приложений

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

7. Выберите **Диспетчер компонентов** >  **+Добавить**, чтобы добавить флаг функции `Beta`.

    > [!div class="mx-imgBorder"]
    > ![Включение флага функции с именем Beta](media/add-beta-feature-flag.png)

    Не определяйте `label` и `Description` сейчас.

8. Чтобы сохранить новый флаг функции, выберите **Применить**.

## <a name="create-a-functions-app"></a>Создание приложения-функции

[!INCLUDE [Create a project using the Azure Functions template](../../includes/functions-vstools-create.md)]

## <a name="connect-to-an-app-configuration-store"></a>Подключение к хранилищу Конфигурации приложений

В этом проекте будет использоваться [внедрение зависимостей в Функциях Azure .NET](../azure-functions/functions-dotnet-dependency-injection.md). При этом в качестве дополнительного источника конфигурации, в котором хранятся флаги функций, добавляется служба "Конфигурация приложений Azure".

1. Щелкните проект правой кнопкой мыши и выберите **Управление пакетами NuGet**. На вкладке **Обзор** найдите и добавьте в проект следующие пакеты NuGet:
   - [Microsoft.Extensions.Configuration.AzureAppConfiguration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureAppConfiguration/) 4.1.0 или более поздней версии;
   - [Microsoft.FeatureManagement](https://www.nuget.org/packages/Microsoft.FeatureManagement/) 2.2.0 или более поздней версии;
   - [Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions/) 1.1.0 или более поздней версии. 

2. Добавьте новый файл *Startup.cs* с приведенным ниже кодом. В нем определен класс с именем `Startup`, который реализует абстрактный класс `FunctionsStartup`. С помощью атрибута сборки указывается имя типа, используемого при запуске Функций Azure.

    ```csharp
    using System;
    using Microsoft.Azure.Functions.Extensions.DependencyInjection;
    using Microsoft.Extensions.Configuration;
    using Microsoft.FeatureManagement;

    [assembly: FunctionsStartup(typeof(FunctionApp.Startup))]

    namespace FunctionApp
    {
        class Startup : FunctionsStartup
        {
            public override void ConfigureAppConfiguration(IFunctionsConfigurationBuilder builder)
            {
            }

            public override void Configure(IFunctionsHostBuilder builder)
            {
            }
        }
    }
    ```


3. Обновите метод `ConfigureAppConfiguration` и добавьте поставщик Конфигурации приложений Azure в качестве дополнительного источника конфигурации, вызвав `AddAzureAppConfiguration()`. 

   Метод `UseFeatureFlags()` указывает поставщику загрузить флаги функций. Для всех флагов функций срок действия кэша по умолчанию истекает через 30 секунд, а затем выполняется повторная проверка на предмет изменений. Интервал времени, по истечении которого действие прекращается, можно обновить. Для этого задайте свойство `FeatureFlagsOptions.CacheExpirationInterval`, передаваемое в метод `UseFeatureFlags`. 

    ```csharp
    public override void ConfigureAppConfiguration(IFunctionsConfigurationBuilder builder)
    {
        builder.ConfigurationBuilder.AddAzureAppConfiguration(options =>
        {
            options.Connect(Environment.GetEnvironmentVariable("ConnectionString"))
                   .Select("_")
                   .UseFeatureFlags();
        });
    }
    ```
   > [!TIP]
   > Если вы не хотите, чтобы в приложение загружались какие-либо конфигурации, кроме флагов функций, можно вызвать `Select("_")`, чтобы загрузить только несуществующий фиктивный ключ "_". Если не вызвать метод `Select`, по умолчанию будут загружены все значения ключей конфигурации в хранилище службы "Конфигурация приложений".

4. Обновите метод `Configure`, чтобы службы Конфигурации приложений Azure и диспетчер функций были доступны посредством внедрения зависимостей.

    ```csharp
    public override void Configure(IFunctionsHostBuilder builder)
    {
        builder.Services.AddAzureAppConfiguration();
        builder.Services.AddFeatureManagement();
    }
    ```

5. Откройте *Function1.cs* и добавьте указанные ниже пространства имен.

    ```csharp
    using System.Linq;
    using Microsoft.FeatureManagement;
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    ```

   Добавьте конструктор для получения экземпляров `_featureManagerSnapshot` и `IConfigurationRefresherProvider` с помощью внедрения зависимостей. Из `IConfigurationRefresherProvider` можно получить экземпляр `IConfigurationRefresher`.

    ```csharp
    private readonly IFeatureManagerSnapshot _featureManagerSnapshot;
    private readonly IConfigurationRefresher _configurationRefresher;

    public Function1(IFeatureManagerSnapshot featureManagerSnapshot, IConfigurationRefresherProvider refresherProvider)
    {
        _featureManagerSnapshot = featureManagerSnapshot;
        _configurationRefresher = refresherProvider.Refreshers.First();
    }
    ```

6. Обновите метод `Run`, чтобы изменить значение отображаемого сообщения в зависимости от состояния флага функции.

   Метод `TryRefreshAsync` вызывается в начале вызова Функций для обновления флагов функций. Это не будет работать, если время истечения срока действия кэша не достигнуто. Удалите оператор `await`, если вы предпочитаете, чтобы флаги функций обновлялась без блокировки текущего вызова Функций. В этом случае при последующих вызовах Функций будет поступать обновленное значение.

    ```csharp
    [FunctionName("Function1")]
    public async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
        ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        await _configurationRefresher.TryRefreshAsync();

        string message = await _featureManagerSnapshot.IsEnabledAsync("Beta")
                ? "The Feature Flag 'Beta' is turned ON"
                : "The Feature Flag 'Beta' is turned OFF";

        return (ActionResult)new OkObjectResult(message);
    }
    ```

## <a name="test-the-function-locally"></a>Локальное тестирование функции

1. Задайте переменную среды с именем **ConnectionString**, где значение является строкой подключения, полученной ранее в хранилище службы "Конфигурация приложений" в разделе **Ключи доступа**. Если вы используете командную строку Windows, выполните следующую команду и перезапустите командную строку, чтобы изменения вступили в силу:

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

1. Чтобы проверить работу функции, нажмите клавишу F5. Если будет предложено, примите запрос от Visual Studio на скачивание и установку **основных инструментов решения "Функции Azure" (CLI)**. Кроме того, возможно, вам понадобиться включить исключение брандмауэра, чтобы инструменты могли обрабатывать HTTP-запросы.

1. Скопируйте URL-адрес функции из выходных данных среды выполнения функций Azure.

    ![Отладки рассматриваемой в этом кратком руководстве функции в VS](./media/quickstarts/function-visual-studio-debugging.png)

1. Вставьте URL-адрес для HTTP-запроса в адресной строке браузера. На следующем рисунке показан ответ, указывающий, что флаг функций `Beta` отключен. 

    ![Флаг функции, рассматриваемый в этом кратком руководстве, отключен](./media/quickstarts/functions-launch-ff-disabled.png)

1. Войдите на [портал Azure](https://portal.azure.com). Щелкните элемент **Все ресурсы** и выберите хранилище службы "Конфигурация приложений", которое вы создали.

1. Выберите **Диспетчер компонентов** и измените состояние ключа **Beta** на **Вкл.**

1. Обновите приложение браузера несколько раз. Если действие кэшированного флага функции прекращается по истечении 30 секунд, страница должна измениться и отобразить сведения о том, что флаг функции `Beta` включен, как показано на приведенном ниже изображении.
 
    ![Флаг функций, рассматриваемый в этом кратком руководстве, включен](./media/quickstarts/functions-launch-ff-enabled.png)

> [!NOTE]
> Пример кода, используемый в этом руководстве, можно скачать в [репозитории GitHub для службы "Конфигурация приложений Azure"](https://github.com/Azure/AppConfiguration/tree/master/examples/DotNetCore/AzureFunction).

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

Используя сведения из этого краткого руководства, вы создали флаг функции и использовали его с приложением Функций Azure с помощью библиотеки [Microsoft.FeatureManagement](/dotnet/api/microsoft.featuremanagement).

- Узнайте больше об [управлении функциями](./concept-feature-management.md).
- [Управление флагами компонентов](./manage-feature-flags.md)
- [Использование условных флагов функций](./howto-feature-filters-aspnet-core.md)
- [Включение поэтапного развертывания функций для определенных аудиторий](./howto-targetingfilter-aspnet-core.md)
- [использованию динамической конфигурации в приложении Функций Azure](./enable-dynamic-configuration-azure-functions-csharp.md)