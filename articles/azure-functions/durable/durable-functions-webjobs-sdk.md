---
title: Запуск Устойчивые функции как веб-заданий в Azure
description: Узнайте, как программировать и настраивать запуск устойчивых функций в веб-заданиях с помощью пакета SDK для веб-заданий.
ms.topic: conceptual
ms.date: 04/25/2018
ms.author: azfuncdf
ms.openlocfilehash: 826e475eb71563b52d687903aeac4ec936e267f6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96009506"
---
# <a name="how-to-run-durable-functions-as-webjobs"></a>Запуск Устойчивые функции как веб-заданий

По умолчанию Устойчивые функции использует среду выполнения функций Azure для размещения оркестрации. Однако могут возникнуть определенные ситуации, когда требуется больший контроль над кодом, который прослушивает события. В этой статье показано, как реализовать согласование с помощью пакета SDK для веб-заданий. Чтобы просмотреть более подробное сравнение функций и веб-заданий, см. статью [Сравнение функций и заданий](../functions-compare-logic-apps-ms-flow-webjobs.md#compare-functions-and-webjobs).

Расширения [Функций Azure](../functions-overview.md) и [устойчивых функций](durable-functions-overview.md) созданы на основе [пакета SDK для веб-заданий](../../app-service/webjobs-sdk-how-to.md). Узел задания в пакете SDK веб-заданий — это среда выполнения в функциях Azure. Если вам нужно управлять поведением способами, невозможными в функциях Azure, вы можете разрабатывать и запускать Устойчивые функции с помощью пакета SDK для веб-заданий самостоятельно.

В версии 3. x пакета SDK веб-заданий узел является реализацией `IHost` , а в версии 2. x используется `JobHost` объект.

Пример цепочки Устойчивые функции можно найти в пакете SDK для веб-заданий версии 2. x: Скачайте или клонировать [репозиторий устойчивые функции](https://github.com/azure/azure-functions-durable-extension/), а затем *выберите ветвь версии* 1 и перейдите к папке *Samples \\ вебжобссдк \\ chaining* .

## <a name="prerequisites"></a>Предварительные условия

В этой статье предполагается, что вы знакомы с основами пакета SDK для веб-заданий, разработки библиотеки классов C# для Функций Azure и устойчивых функций. Ознакомиться с этими темами вам помогут следующие ресурсы:

* [Начало работы с пакетом SDK WebJobs](../../app-service/webjobs-sdk-get-started.md)
* [Создание первой функции с помощью Visual Studio](../functions-create-your-first-function-visual-studio.md)
* [Цепочки функций в устойчивых функциях — пример последовательности Hello](durable-functions-sequence.md)

Чтобы выполнить действия, описанные в этой статье, сделайте следующее:

* [Установите Visual Studio 2019](/visualstudio/install/) с рабочей нагрузкой **Разработка для Azure**.

  Если у вас уже есть Visual Studio, но нет такой рабочей нагрузки, добавьте рабочую нагрузку, выбрав **инструменты**  >  **получить средства и компоненты**.

  (Вместо этого можно использовать [Visual Studio Code](https://code.visualstudio.com/), но некоторые инструкции относятся к Visual Studio.)

* Установите и запустите [эмулятор хранилища Azure](../../storage/common/storage-use-emulator.md) версии 5.2 или выше. Альтернативой является добавление в файл *App.config* строки подключения к службе хранилища Azure.

## <a name="webjobs-sdk-versions"></a>Версии пакета SDK для веб-заданий

В этой статье объясняется, как разрабатывать проект пакета SDK для веб-заданий версии 2.x (эквивалентен Функциям Azure версии 1.x). Дополнительные сведения о версии 3.x см. в разделе [Пакет SDK для веб-заданий версии 3.x](#webjobs-sdk-3x) далее в этой статье.

## <a name="create-a-console-app"></a>Создание консольного приложения

Чтобы запустить Устойчивые функции как веб-задания, необходимо сначала создать консольное приложение. Проект пакета SDK для веб-заданий — это просто проект консольного приложения с установленными соответствующими пакетами NuGet.

В диалоговом окне **Новый проект** Visual Studio выберите **классическое**  >  **консольное приложение Windows (платформа .NET Framework)**. В файле проекта `TargetFrameworkVersion` должна иметь значение `v4.6.1`.

В Visual Studio также есть шаблон проекта веб-задания, который можно использовать, выбрав **Cloud**  >  **веб-задание Azure (платформа .NET Framework)**. Этот шаблон устанавливает множество пакетов, некоторые из которых могут не потребоваться.

## <a name="install-nuget-packages"></a>Установка пакетов Nuget

Вам потребуются пакеты NuGet для пакета SDK для веб-заданий, основные привязки, платформа ведения журналов и расширение устойчивых задач. Ниже приведены команды **консоли диспетчера пакетов** для этих пакетов с последними стабильными номерами версий по сравнению с датой написания этой статьи:

```powershell
Install-Package Microsoft.Azure.WebJobs.Extensions -version 2.2.0
Install-Package Microsoft.Extensions.Logging -version 2.0.1
Install-Package Microsoft.Azure.WebJobs.Extensions.DurableTask -version 1.8.3
```

Вам также понадобятся регистраторы. Следующие команды устанавливают поставщик Application Insights Azure и `ConfigurationManager` . `ConfigurationManager` позволяет получить ключ инструментирования Application Insights из параметров приложения.

```powershell
Install-Package Microsoft.Azure.WebJobs.Logging.ApplicationInsights -version 2.2.0
Install-Package System.Configuration.ConfigurationManager -version 4.4.1
```

Следующая команда устанавливает поставщик консоли:

```powershell
Install-Package Microsoft.Extensions.Logging.Console -version 2.0.1
```

## <a name="jobhost-code"></a>Код JobHost

Создав консольное приложение и установив необходимые пакеты NuGet, вы можете использовать Устойчивые функции. Это можно сделать с помощью кода JobHost.

Чтобы использовать расширение устойчивых функций, вызовите `UseDurableTask` для объекта `JobHostConfiguration` в вашем методе `Main`:

```cs
var config = new JobHostConfiguration();
config.UseDurableTask(new DurableTaskExtension
{
    HubName = "MyTaskHub",
};
```

Список свойств, которые можно задать в объекте `DurableTaskExtension`, см. в файле [host.json](../functions-host-json.md#durabletask).

Метод `Main` служит также местом для настройки регистраторов. В следующем примере настраиваются поставщики консоли и Application Insights.

```cs
static void Main(string[] args)
{
    using (var loggerFactory = new LoggerFactory())
    {
        var config = new JobHostConfiguration();

        config.DashboardConnectionString = "";

        var instrumentationKey =
            ConfigurationManager.AppSettings["APPINSIGHTS_INSTRUMENTATIONKEY"];

        config.LoggerFactory = loggerFactory
            .AddApplicationInsights(instrumentationKey, null)
            .AddConsole();

        config.UseTimers();
        config.UseDurableTask(new DurableTaskExtension
        {
            HubName = "MyTaskHub",
        });
        var host = new JobHost(config);
        host.RunAndBlock();
    }
}
```

## <a name="functions"></a>Функции

Устойчивые функции в контексте веб-заданий несколько отличается от Устойчивые функции в контексте функций Azure. Важно помнить о различиях при написании кода.

Пакет SDK для веб-заданий не поддерживает следующие возможности Функций Azure:

* [Атрибут FunctionName](#functionname-attribute)
* [триггером HTTP](#http-trigger)
* [API управления HTTP устойчивых функций](#http-management-api)

### <a name="functionname-attribute"></a>Атрибут FunctionName

В проекте пакета SDK для веб-заданий имя метода функции является именем функции. Атрибут `FunctionName` используется только в Функциях Azure.

### <a name="http-trigger"></a>Триггер HTTP

В пакете SDK для веб-заданий нет триггера HTTP. Клиент оркестрации в примере проекта использует триггер таймера:

```cs
public static async Task CronJob(
    [TimerTrigger("0 */2 * * * *")] TimerInfo timer,
    [OrchestrationClient] DurableOrchestrationClient client,
    ILogger logger)
{
  ...
}
```

### <a name="http-management-api"></a>API управления HTTP

Так как пакет SDK для веб-заданий не содержит ни одного триггера HTTP, в нем нет [API управления HTTP](durable-functions-http-api.md).

В проекте пакета SDK веб-заданий можно вызывать методы для объекта клиента оркестрации, а не отправлять HTTP-запросы. Следующие методы соответствуют трем задачам, которые можно выполнить с помощью API управления HTTP.

* `GetStatusAsync`
* `RaiseEventAsync`
* `TerminateAsync`

Функция клиента оркестрации в образце проекта запускает функцию Orchestrator, а затем переходит в цикл, который вызывается `GetStatusAsync` каждые 2 секунды:

```cs
string instanceId = await client.StartNewAsync(nameof(HelloSequence), input: null);
logger.LogInformation($"Started new instance with ID = {instanceId}.");

DurableOrchestrationStatus status;
while (true)
{
    status = await client.GetStatusAsync(instanceId);
    logger.LogInformation($"Status: {status.RuntimeStatus}, Last update: {status.LastUpdatedTime}.");

    if (status.RuntimeStatus == OrchestrationRuntimeStatus.Completed ||
        status.RuntimeStatus == OrchestrationRuntimeStatus.Failed ||
        status.RuntimeStatus == OrchestrationRuntimeStatus.Terminated)
    {
        break;
    }

    await Task.Delay(TimeSpan.FromSeconds(2));
}
```

## <a name="run-the-sample"></a>Запуск примера

У вас Устойчивые функции настроить запуск в качестве веб-задания, и теперь у вас есть представление о том, как это будет отличаться от запуска Устойчивые функции как автономных функций Azure. На этом этапе работа с примером может быть полезной.

Этот раздел содержит общие сведения о способах запуска [примера проекта](https://github.com/Azure/azure-functions-durable-extension/tree/v1/samples/webjobssdk/chaining). Подробные инструкции о том, как запустить проект пакета SDK для веб-заданий локально и развернуть его в веб-задание Azure, см. в разделе [Развертывание в качестве WebJob (веб-задания)](../../app-service/webjobs-sdk-get-started.md#deploy-as-a-webjob).

### <a name="run-locally"></a>Локальный запуск

1. Запустите эмулятор хранилища (см. раздел [Предварительные требования](#prerequisites)).

1. Если вы хотите просмотреть журналы в Application Insights при локальном запуске проекта:

    а. Создайте ресурс Application Insights и используйте для него **Общий** тип приложения.

    b. Сохраните ключ инструментирования в файл *App.config*.

1. Запустите проект.

### <a name="run-in-azure"></a>Запуск в Azure

1. Создайте веб-приложение и учетную запись хранения.

1. В веб-приложении сохраните строку подключения к хранилищу в параметре приложения с именем `AzureWebJobsStorage` .

1. Создайте ресурс Application Insights и используйте для него **Общий** тип приложения.

1. Сохраните ключ инструментирования в параметре приложения с именем `APPINSIGHTS_INSTRUMENTATIONKEY` .

1. Разверните приложение в качестве веб-задания.

## <a name="webjobs-sdk-3x"></a>Пакет SDK для веб-заданий версии 3.x

В этой статье описывается разработка проекта пакета SDK 2. x для веб-заданий. Если вы разрабатываете проект [пакета SDK 3. x](../../app-service/webjobs-sdk-get-started.md) для веб-заданий, этот раздел поможет вам понять различия.

Основным изменением является использование .NET Core вместо платформа .NET Framework. Чтобы создать проект пакета SDK 3. x для веб-заданий, инструкции одинаковы, за исключением следующих.

1. Создайте консольное приложение .NET Core. В диалоговом окне **Новый проект** Visual Studio выберите   >  **консольное приложение .NET Core (.NET Core)**. Файл проекта указывает, что `TargetFramework` является `netcoreapp2.x`.

1. Выберите версию пакета SDK для веб-заданий версии 3. x следующих пакетов:

    * `Microsoft.Azure.WebJobs.Extensions`
    * `Microsoft.Azure.WebJobs.Extensions.Storage`
    * `Microsoft.Azure.WebJobs.Logging.ApplicationInsights`

1. Задайте строку подключения к хранилищу и ключ инструментирования Application Insights в *appsettings.jsв* файле с помощью платформы конфигурации .NET Core. Приведем пример:

    ```json
        {
            "AzureWebJobsStorage": "<replace with storage connection string>",
            "APPINSIGHTS_INSTRUMENTATIONKEY": "<replace with Application Insights instrumentation key>"
        }
    ```

1. Измените `Main` код метода, чтобы сделать это. Ниже приведен пример:

   ```cs
   static void Main(string[] args)
   {
        var hostBuilder = new HostBuilder()
            .ConfigureWebJobs(config =>
            {
                config.AddAzureStorageCoreServices();
                config.AddAzureStorage();
                config.AddTimers();
                config.AddDurableTask(options =>
                {
                    options.HubName = "MyTaskHub";
                    options.AzureStorageConnectionStringName = "AzureWebJobsStorage";
                });
            })
            .ConfigureLogging((context, logging) =>
            {
                logging.AddConsole();
                logging.AddApplicationInsights(config =>
                {
                    config.InstrumentationKey = context.Configuration["APPINSIGHTS_INSTRUMENTATIONKEY"];
                });
            })
            .UseConsoleLifetime();

        var host = hostBuilder.Build();

        using (host)
        {
            host.Run();
        }
   }
   ```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о пакете SDK для веб-заданий см. в статье [Использование пакета SDK WebJobs для фоновой обработки управления событиями](../../app-service/webjobs-sdk-how-to.md).
