---
title: Начало работы с пакетом SDK WebJobs
description: Знакомство с пакетом SDK WebJobs для фоновой обработки на основе событий. Получение доступа к данным в службах Azure и службах сторонних разработчиков.
author: ggailey777
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: article
ms.date: 02/18/2019
ms.author: glenga
ms.openlocfilehash: 4a22602dd9638b981cfe3d8bae9b5cdaacbf90dc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91652046"
---
# <a name="get-started-with-the-azure-webjobs-sdk-for-event-driven-background-processing"></a>Использование пакета SDK для Веб-заданий Azure для фоновой обработки на основе событий

Из этой статьи вы узнаете, как в Visual Studio 2019 создать проект пакета SDK для веб-заданий Azure, запустить его в локальной среде, а затем развернуть в [Службе приложений Azure](overview.md). Версия 3.x пакета SDK веб-заданий поддерживает консольные приложения .NET Core и .NET Framework. Дополнительные сведения о работе с пакетом SDK веб-заданий см. в статье [Использование пакета SDK WebJobs Azure для фоновой обработки на основе событий](webjobs-sdk-how-to.md).

В этой статье показано, как развернуть веб-задания в формате консольного приложения .NET Core. Сведения о развертывании веб-заданий в формате консольного приложения .NET Framework см. [здесь](webjobs-dotnet-deploy-vs.md#webjobs-as-net-framework-console-apps). Если вас интересует информация о пакете SDK для веб-заданий версии 2.x, который поддерживает только .NET Framework, перейдите к статье [Разработка и развертывание веб-заданий в Службе приложений Azure с помощью Visual Studio](webjobs-dotnet-deploy-vs.md).

## <a name="prerequisites"></a>Предварительные требования

* [Установите Visual Studio 2019](/visualstudio/install/) с рабочей нагрузкой **Разработка для Azure**. Если у вас уже установлена среда Visual Studio, но нет компонентов для разработки приложений, добавьте их, выбрав **Tools > Get Tools and Features** (Инструменты > Добавить инструменты и компоненты).

* Для публикации проекта пакета SDK веб-заданий в Azure требуется [учетная запись Azure](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

## <a name="create-a-project"></a>Создание проекта

1. В Visual Studio выберите **Создать проект**.

2. Затем выберите **Консольное приложение (.NET Core)** .

3. Присвойте проекту имя *WebJobsSDKSample* и щелкните **Создать**.

   ![Диалоговое окно "Новый проект"](./media/webjobs-sdk-get-started/new-project.png)

## <a name="webjobs-nuget-packages"></a>Пакеты NuGet веб-заданий

1. Установите последнюю стабильную версию 3.x [пакета NuGet `Microsoft.Azure.WebJobs.Extensions`](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions/), которая включает `Microsoft.Azure.WebJobs`.

     Ниже приведена команда **консоли диспетчера пакетов**.

     ```powershell
     Install-Package Microsoft.Azure.WebJobs.Extensions -version <3_X_VERSION>
     ```

    Замените в этой команде заполнитель `<3_X_VERSION>` номером поддерживаемой версии пакета. 

## <a name="create-the-host"></a>Создание узла

Узел представляет собой контейнер среды выполнения для функций, который ожидает сигналов от триггеров и вызывает функции. Следующий код создает узел, который реализует [`IHost`](/dotnet/api/microsoft.extensions.hosting.ihost), универсальный узел для ASP.NET Core.

1. В файле *Program.cs* добавьте следующие инструкции `using`:

    ```cs
    using System.Threading.Tasks;
    using Microsoft.Extensions.Hosting;
    ```

1. Замените метод `Main` следующим кодом:

    ```cs
    static async Task Main()
    {
        var builder = new HostBuilder();
        builder.ConfigureWebJobs(b =>
                {
                    b.AddAzureStorageCoreServices();
                });
        var host = builder.Build();
        using (host)
        {
            await host.RunAsync();
        }
    }
    ```

В ASP.NET Core конфигурации узла настраиваются путем вызова методов в экземпляре [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder). Дополнительные сведения можно найти в статье [Универсальный узел .NET](/aspnet/core/fundamentals/host/generic-host). Метод расширения `ConfigureWebJobs` инициализирует узел веб-заданий. В `ConfigureWebJobs` можно инициализировать конкретные расширения веб-заданий и задать свойства для этих расширений.  

## <a name="enable-console-logging"></a>Включение журналирования консоли

В этом разделе описано, как настроить ведение журнала консоли с использованием [платформы ведения журналов для ASP.NET Core](/aspnet/core/fundamentals/logging).

1. Установите последнюю стабильную версию [пакета NuGet `Microsoft.Extensions.Logging.Console`](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console/), которая включает `Microsoft.Extensions.Logging`.

   Ниже приведена команда **консоли диспетчера пакетов**.

   ```powershell
   Install-Package Microsoft.Extensions.Logging.Console -version <3_X_VERSION>
   ```
   Замените в этой команде заполнитель `<3_X_VERSION>` номером поддерживаемой версии пакета 3.x.

1. В *Program.cs* добавьте инструкцию `using`:

   ```cs
   using Microsoft.Extensions.Logging;
   ```

1. Вызовите метод [`ConfigureLogging`](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configurelogging) в [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder). Метод [`AddConsole`](/dotnet/api/microsoft.extensions.logging.consoleloggerextensions.addconsole) добавляет в конфигурацию ведение журнала консоли.

    ```cs
    builder.ConfigureLogging((context, b) =>
    {
        b.AddConsole();
    });
    ```

    Метод `Main` теперь выглядит следующим образом:

    ```cs
    static async Task Main()
    {
        var builder = new HostBuilder();
        builder.ConfigureWebJobs(b =>
                {
                    b.AddAzureStorageCoreServices();
                });
        builder.ConfigureLogging((context, b) =>
                {
                    b.AddConsole();
                });
        var host = builder.Build();
        using (host)
        {
            await host.RunAsync();
        }
    }
    ```

    Это обновление реализует следующее:

    * Отключает [ведение журнала панели мониторинга](https://github.com/Azure/azure-webjobs-sdk/wiki/Queues#logs). Панель мониторинга — это устаревшее средство мониторинга, поэтому ведение журнала панели мониторинга не рекомендуется в высокопроизводительных сценариях.
    * Добавляет поставщика консоли с [фильтрацией](webjobs-sdk-how-to.md#log-filtering) по умолчанию.

Теперь можно добавить функцию, которая активируется сообщениями из очереди службы хранилища Azure.

## <a name="install-the-storage-binding-extension"></a>Установка расширения привязки Службы хранилища

Начиная с версии 3.x, необходимо явно устанавливать расширения привязки хранилища, которые являются обязательными для пакета SDK веб-заданий. В предыдущих версиях привязки хранилища включались в пакет SDK.

1. Установите последнюю стабильную версию пакета NuGet [Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage), версию 3.х. 

    Ниже приведена команда **консоли диспетчера пакетов**.

    ```powershell
    Install-Package Microsoft.Azure.WebJobs.Extensions.Storage -Version <3_X_VERSION>
    ```
    
    Замените в этой команде заполнитель `<3_X_VERSION>` номером поддерживаемой версии пакета. 

2. В методе расширения `ConfigureWebJobs` вызовите метод `AddAzureStorage` в экземпляре [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder), чтобы инициализировать расширение службы хранилища. На этом этапе метод `ConfigureWebJobs` выглядит следующим образом:

    ```cs
    builder.ConfigureWebJobs(b =>
                    {
                        b.AddAzureStorageCoreServices();
                        b.AddAzureStorage();
                    });
    ```

## <a name="create-a-function"></a>Создание функции

1. Щелкните проект правой кнопкой мыши, выберите **Добавить** > **Новый элемент...** , щелкните **Класс**, присвойте файлу нового класса C# имя *Functions.cs* и нажмите кнопку **Добавить**.

1. В сценарии Functions.cs замените созданный шаблон следующим кодом:
    
    ```cs
    using Microsoft.Azure.WebJobs;
    using Microsoft.Extensions.Logging;
    
    namespace WebJobsSDKSample
    {
        public class Functions
        {
            public static void ProcessQueueMessage([QueueTrigger("queue")] string message, ILogger logger)
            {
                logger.LogInformation(message);
            }
        }
    }
    ```

   Атрибут `QueueTrigger` вызывает в среде выполнения эту функцию при записи нового сообщения в очередь службы хранилища Azure с именем `queue`. Содержимое сообщения в очереди передается коду метода в параметре `message`. Тело метода реализуется при обработке данных триггера. В этом примере код просто записывает сообщение в журнал.

   Параметр `message` не должен быть строкой. Также можно выполнить привязку к объекту JSON, массиву байтов или [CloudQueueMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage). [См. "Использование триггера очереди"](../azure-functions/functions-bindings-storage-queue-trigger.md?tabs=csharp#usage). Каждый тип привязки (например, очереди, BLOB-объекты или таблицы) обладает своим набором типов параметров привязки.

## <a name="create-a-storage-account"></a>Создание учетной записи хранения

Эмулятор хранения Azure, запущенный локально, не имеет всех функций, необходимых для пакета SDK веб-заданий. Поэтому в этом разделе вы создадите учетную запись хранения в Azure и настроите ее использование в проекте. Если у вас уже есть учетная запись хранения, перейдите сразу к шагу 6.

1. В Visual Studio откройте **обозреватель сервера** и войдите в Azure. Щелкните правой кнопкой мыши узел **Azure** и выберите команду **Connect to Microsoft Azure Subscription** (Подключение к подписке Microsoft Azure).

   ![Вход в Azure](./media/webjobs-sdk-get-started/sign-in.png)

1. В узле **Azure** **обозревателя сервера** щелкните правой кнопкой мыши элемент **Хранилище** и выберите **Создать учетную запись хранения**.

   ![Создание меню учетной записи хранения](./media/webjobs-sdk-get-started/create-storage-account-menu.png)

1. В диалоговом окне **Создание учетной записи хранения** введите уникальное имя учетной записи хранения.

1. Выберите тот же **Регион**, в котором вы создали приложение службы приложений, или ближайший к вам регион.

1. Нажмите кнопку **создания**.

   ![Создание учетной записи хранения](./media/webjobs-sdk-get-started/create-storage-account.png)

1. В узле **Хранилище** **обозревателя серверов** выберите новую учетную запись хранения. В окне **Свойства** выберите многоточие ( **...** ) справа от поля со значением **Строка подключения**.

   ![Многоточие в строке подключения](./media/webjobs-sdk-get-started/conn-string-ellipsis.png)

1. Скопируйте строку подключения и сохраните это значение где-либо, откуда впоследствии его можно будет легко скопировать еще раз.

   ![Копирование строки подключения](./media/webjobs-sdk-get-started/copy-key.png)

## <a name="configure-storage-to-run-locally"></a>Настройка хранилища для локального запуска

Пакет SDK веб-заданий ищет строку подключения в параметрах приложения в Azure. При локальном запуске он ищет это значение в локальном файле конфигурации или среди переменных среды.

1. Щелкните проект правой кнопкой мыши, выберите **Добавить** > **Новый элемент...** , выберите **Файл конфигурации JSON JavaScript**, присвойте новому файлу имя *appSettings.json* и щелкните **Добавить**. 

1. В новом файле добавьте поле `AzureWebJobsStorage`, как показано в следующем примере:

    ```json
    {
        "AzureWebJobsStorage": "{storage connection string}"
    }
    ```

1. Замените значение *{storage connection string}* на строку подключения, скопированную ранее.

1. Выберите файл *appSettings.json* в Обозревателе решений и в окне **Свойства** установите для параметра **Копировать в выходной каталог** значение **Копировать более позднюю версию**.

Позже вы добавите тот же параметр приложения строки подключения в приложение в службе приложений Azure.

## <a name="test-locally"></a>Локальное тестирование

В этом разделе проект создается и запускается локально, а кроме того, посредством создания очереди сообщений запускается триггер вызова функции.

1. Нажмите клавиши **CTRL+F5**, чтобы запустить проект.

   Консоль показывает, что среда выполнения обнаружила функцию и ожидает сообщений в очереди для ее активации. На узле версии 3.x отобразятся следующие сообщения:

   ```console
    info: Microsoft.Azure.WebJobs.Hosting.JobHostService[0]
          Starting JobHost
    info: Host.Startup[0]
          Found the following functions:
          WebJobsSDKSample.Functions.ProcessQueueMessage

    info: Host.Startup[0]
          Job host started
    Application started. Press Ctrl+C to shut down.
    Hosting environment: Development
    Content root path: C:\WebJobsSDKSample\WebJobsSDKSample\bin\Debug\netcoreapp2.1\
   ```

1. Закройте окно консоли.

1. В Visual Studio в **обозревателе сервера** разверните узел новой учетной записи хранения и щелкните правой кнопкой мыши элемент **Очереди**.

1. Выберите **Создать очередь**.

1. Введите *queue* в качестве имени очереди, а затем нажмите кнопку **OK**.

   ![Снимок экрана, на котором показано, где создается очередь, и назовите ее "Queue". ](./media/webjobs-sdk-get-started/create-queue.png)

1. Щелкните правой кнопкой мыши узел новой очереди, а затем выберите команду **Просмотреть очередь**.

1. Выберите значок **Добавить сообщение**.

   ![Снимок экрана, на котором выделен значок добавления сообщения.](./media/webjobs-sdk-get-started/create-queue-message.png)

1. В диалоговом окне **Добавить сообщение** введите *Здравствуй, мир!* в поле **Текст сообщения**, а затем нажмите кнопку **OK**. В очереди есть новое сообщение.

   ![Создание очереди](./media/webjobs-sdk-get-started/hello-world-text.png)

1. Запустите проект снова.

   Поскольку в функции `ProcessQueueMessage` использовался атрибут `QueueTrigger`, среда выполнения пакета SDK WebJobs прослушивает запускаемые сообщения в очереди. Она находит новое сообщение в очереди с именем *queue* и вызывает функцию.

   Из-за [экспоненциально растущего интервала опроса очереди](../azure-functions/functions-bindings-storage-queue-trigger.md?tabs=csharp#polling-algorithm) процесс поиска сообщения и вызова функции может занимать до 2 минут. Время отклика можно сократить путем запуска в [режиме разработки](webjobs-sdk-how-to.md#host-development-settings).

   В консоли отобразятся следующие сообщения:

   ```console
    info: Function.ProcessQueueMessage[0]
          Executing 'Functions.ProcessQueueMessage' (Reason='New queue message detected on 'queue'.', Id=2c319369-d381-43f3-aedf-ff538a4209b8)
    info: Function.ProcessQueueMessage[0]
          Trigger Details: MessageId: b00a86dc-298d-4cd2-811f-98ec39545539, DequeueCount: 1, InsertionTime: 1/18/2019 3:28:51 AM +00:00
    info: Function.ProcessQueueMessage.User[0]
          Hello World!
    info: Function.ProcessQueueMessage[0]
          Executed 'Functions.ProcessQueueMessage' (Succeeded, Id=2c319369-d381-43f3-aedf-ff538a4209b8)
   ```

1. Закройте окно консоли. 

1. Затем вернитесь к окну "Очередь" и обновите страницу. Сообщение исчезнет, так как оно уже обработано функцией, выполняемой локально. 

## <a name="add-application-insights-logging"></a>Добавление журнала Application Insights

При выполнении проекта в Azure вам не удастся отследить выполнение функции в консоли. В качестве решения для мониторинга рекомендуется использовать [Application Insights](../azure-monitor/app/app-insights-overview.md). Дополнительные сведения см. в разделе [Мониторинг функций Azure](../azure-functions/functions-monitoring.md).

В этом разделе рассматриваются следующие задания по настройке ведения журнала Application Insights перед развертыванием в Azure:

* Убедитесь, что у вас есть приложение службы приложений Azure и экземпляр Application Insights, с которым будет вестись работа.
* Настройте приложение службы приложений Azure для использования экземпляра Application Insights и учетной записи хранения, созданной ранее.
* Настройте проект ведения журнала Application Insights.

### <a name="create-app-service-app-and-application-insights-instance"></a>Создание приложения службы приложений Azure и экземпляра Application Insights

1. Если у вас еще нет приложения службы приложений, которое можно использовать, [создайте его](quickstart-dotnet-framework.md). При создании приложения можно также создать подключенный ресурс Application Insights. После этого параметр `APPINSIGHTS_INSTRUMENTATIONKEY` будет настроен в вашем приложении.

1. Если у вас еще нет доступного ресурса Application Insights, [создайте его](../azure-monitor/app/create-new-resource.md ). Установите для параметра **Тип приложения** значение **Общий** и пропустите разделы, следующие за разделом **Copy the instrumentation key** (Копировать ключ инструментирования).

1. При наличии ресурса Application Insights, который вы собираетесь использовать, [скопируйте ключ инструментирования](../azure-monitor/app/create-new-resource.md#copy-the-instrumentation-key).

### <a name="configure-app-settings"></a>Настройка параметров приложения 

1. В Visual Studio в **обозревателе сервера** разверните узел **Служба приложений** в разделе **Azure**.

1. Разверните группу ресурсов с приложением службы приложений, а затем выберите это приложение службы приложений.

1. Выберите **Просмотр параметров**.

1. В поле **Строки подключения** добавьте следующую запись.

   |Имя  |Строка подключения  |Тип базы данных|
   |---------|---------|------|
   |AzureWebJobsStorage | {строка подключения к службе хранилища, скопированная ранее}|Другой|

1. Если поле **Параметры приложения** не содержит ключа инструментирования Application Insights, добавьте один, скопированный ранее. (Ключ инструментирования уже может быть там, в зависимости от способа создания приложения службы приложений).

   |Имя  |Значение  |
   |---------|---------|
   |APPINSIGHTS_INSTRUMENTATIONKEY | {instrumentation key} |

1. Замените строку *{instrumentation key}* ключом инструментирования с используемого ресурса Application Insights.

1. Щелкните **Сохранить**.

1. Добавьте подключение Application Insights в проект, чтобы можно было запустить его локально. В файле *appsettings.json* добавьте поле `APPINSIGHTS_INSTRUMENTATIONKEY`, как показано в следующем примере:

    ```json
    {
        "AzureWebJobsStorage": "{storage connection string}",
        "APPINSIGHTS_INSTRUMENTATIONKEY": "{instrumentation key}"
    }
    ```

    Замените строку *{instrumentation key}* ключом инструментирования с используемого ресурса Application Insights.

1. Сохраните изменения.

### <a name="add-application-insights-logging-provider"></a>Добавление поставщика журнала для Application Insights

Чтобы воспользоваться преимуществами ведения журнала [Application Insights](../azure-monitor/app/app-insights-overview.md), обновите код ведения журнала для следующих целей:

* Добавление поставщика журнала для Application Insights с [фильтрацией](webjobs-sdk-how-to.md#log-filtering) по умолчанию. При локальном выполнении вся информация и журналы более высокого уровня выводятся как в консоль, так и в Application Insights.
* Поместите объект [LoggerFactory](./webjobs-sdk-how-to.md#logging-and-monitoring) в блок `using`, чтобы выходные данные журнала записывались на диск при завершении работы узла.

1. Установите последнюю стабильную версию 3.x пакета [NuGet `Microsoft.Azure.WebJobs.Logging.ApplicationInsights`](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Logging.ApplicationInsights/).

   Ниже приведена команда **консоли диспетчера пакетов**.

   ```powershell
   Install-Package Microsoft.Azure.WebJobs.Logging.ApplicationInsights -Version <3_X_VERSION>
   ```
    Замените в этой команде заполнитель `<3_X_VERSION>` номером поддерживаемой версии пакета.

1. Откройте файл *Program.cs* и замените код в методе `Main` следующим кодом:

    ```cs
    static async Task Main()
    {
        var builder = new HostBuilder();
        builder.UseEnvironment(EnvironmentName.Development);
        builder.ConfigureWebJobs(b =>
                {
                    b.AddAzureStorageCoreServices();
                    b.AddAzureStorage();
                });
        builder.ConfigureLogging((context, b) =>
                {
                    b.AddConsole();

                    // If the key exists in settings, use it to enable Application Insights.
                    string instrumentationKey = context.Configuration["APPINSIGHTS_INSTRUMENTATIONKEY"];
                    if (!string.IsNullOrEmpty(instrumentationKey))
                    {
                        b.AddApplicationInsightsWebJobs(o => o.InstrumentationKey = instrumentationKey);
                    }
                });
        var host = builder.Build();
        using (host)
        {
            await host.RunAsync();
        }
    }
    ```

    Это действие добавляет поставщик Application Insights в конфигурацию ведения журналов с тем ключом, который вы ранее добавили в параметры приложения.

## <a name="test-application-insights-logging"></a>Тестирование ведения журнала Application Insights

В этом разделе выполняется повторный локальный запуск для проверки того, что данные журналов теперь передаются в Application Insights и в консоль.

1. В Visual Studio в **обозревателе сервера** создайте сообщение в очереди, как это было сделано [ранее](#test-locally), но в качестве текста сообщения введите *Hello App Insights!* на этот раз.

1. Запустите проект.

   Пакет SDK веб-заданий обрабатывает сообщения в очереди, а его журналы вы можете просматривать в окне консоли.

1. Закройте окно консоли.

1. Вернитесь к ресурсу Application Insights на [портале Azure](https://portal.azure.com/). Найдите и выберите **Application Insights**.

1. Выберите экземпляр Application Insights.

1. Выберите **Поиск**.

   ![Выбор команды "Поиск"](./media/webjobs-sdk-get-started/select-search.png)

1. Если вы не видите сообщения *Hello App Insights!* , в течение нескольких минут периодически нажимайте кнопку **Обновить**. (Журналы не появляются немедленно, так как клиенту Application Insights требуется некоторое время на запись обрабатываемых журналов на диск.)

   ![Журналы в Application Insights](./media/webjobs-sdk-get-started/logs-in-ai.png)

1. Закройте окно консоли.

## <a name="deploy-to-azure"></a><a name="deploy-as-a-webjob"></a>Развертывание в Azure

В процессе развертывания вы создаете экземпляр службы приложений, в котором будут выполняться функции. При публикации консольного приложения .NET Core в Службе приложений Azure оно автоматически запускается как веб-задание. Дополнительные сведения о публикации см. в статье [Разработка и развертывание веб-заданий с помощью Visual Studio](webjobs-dotnet-deploy-vs.md).

[!INCLUDE [webjobs-publish-net-core](../../includes/webjobs-publish-net-core.md)]

## <a name="trigger-the-function-in-azure"></a>Активация функции в Azure

1. Убедитесь, что вы не работаете локально (закройте окно консоли, если оно еще открыто). В противном случае локальный экземпляр может сначала начать обрабатывать созданные вами сообщения в очереди.

1. На странице **Очередь** в Visual Studio добавьте сообщение в очередь, как и раньше.

1. Обновите страницу **Очередь**, после чего новое сообщение исчезнет, так как оно было обработано с помощью функции, работающей в Azure.

   > [!TIP]
   > При тестировании в Azure используйте [режим разработки](webjobs-sdk-how-to.md#host-development-settings), чтобы убедиться, что функция триггера очереди вызывается сразу же, и избежать задержки из-за [экспоненциально растущего интервала опроса очереди](../azure-functions/functions-bindings-storage-queue-trigger.md?tabs=csharp#polling-algorithm).

### <a name="view-logs-in-application-insights"></a>Просмотр журналов Application Insights

1. Вернитесь к ресурсу Application Insights на [портале Azure](https://portal.azure.com/).

1. Выберите **Поиск**.

1. Если вы не видите сообщение *Hello Azure!* , в течение нескольких минут периодически нажимайте кнопку **Обновить**.

   С помощью функции, работающей в веб-задании, можно просмотреть журналы и сообщение *Hello Azure!* , введенное в предыдущем разделе.

## <a name="add-an-input-binding"></a>Добавление привязки входных данных

Привязка входных данных упрощает код чтения данных. В этом примере сообщением в очереди будет имя большого двоичного объекта, используемое при поиске и чтении большого двоичного объекта в службе хранилища Azure.

1. В сценарии *Functions.cs* замените метод `ProcessQueueMessage` следующим кодом:

   ```cs
   public static void ProcessQueueMessage(
       [QueueTrigger("queue")] string message,
       [Blob("container/{queueTrigger}", FileAccess.Read)] Stream myBlob,
       ILogger logger)
   {
       logger.LogInformation($"Blob name:{message} \n Size: {myBlob.Length} bytes");
   }
   ```

   В этом коде `queueTrigger` — это [выражение привязки](../azure-functions/functions-bindings-expressions-patterns.md), что означает, что его значение определяется во время выполнения.  Во время выполнения оно содержит сообщение в очереди.

1. Добавьте `using`:

   ```cs
   using System.IO;
   ```

1. Создайте контейнер BLOB-объектов в учетной записи хранения.

   а. В Visual Studio в **обозревателе сервера** разверните узел своей учетной записи хранения, щелкните правой кнопкой мыши кнопку **Большие двоичные объекты** и выберите команду **Создать контейнер BLOB-объектов**.

   b. В диалоговом окне **Создать контейнер BLOB-объектов** для имени контейнера введите *container*, а затем нажмите кнопку **OK**.

1. Отправьте файл *Program.cs* в контейнер больших двоичных объектов. (Этот файл используется здесь для примера; вы можете передать любой текстовый файл и создавать сообщение в очереди с его именем.)

   а. В **обозревателе сервера** дважды щелкните узел контейнера, который вы создали.

   b. В окне **Контейнер** нажмите кнопку **Отправить**.

   ![Кнопка отправки BLOB-объекта](./media/webjobs-sdk-get-started/blob-upload-button.png)

   c. Найдите и выберите файл *Program.cs*, а затем нажмите кнопку **OK**.

1. Создайте сообщение в ранее созданной очереди с *Program.cs* в качестве текста сообщения.

   ![Сообщение в очереди на основе сценария Program.cs](./media/webjobs-sdk-get-started/queue-msg-program-cs.png)

1. Запустите проект в локальной среде.

   Сообщение в очереди активирует функцию, которая затем считывает большой двоичный объект и записывает в журнал его размер. В консоли отобразятся следующие сообщения:

   ```console
   Found the following functions:
   ConsoleApp1.Functions.ProcessQueueMessage
   Job host started
   Executing 'Functions.ProcessQueueMessage' (Reason='New queue message detected on 'queue'.', Id=5a2ac479-de13-4f41-aae9-1361f291ff88)
   Blob name:Program.cs
   Size: 532 bytes
   Executed 'Functions.ProcessQueueMessage' (Succeeded, Id=5a2ac479-de13-4f41-aae9-1361f291ff88)
   ```

## <a name="add-an-output-binding"></a>Добавление выходной привязки

Привязки выходных данных упрощают код записи данных. Данный пример проще предыдущего, так как вместо регистрации размера большого двоичного объекта в журнале была сделана запись его копии. Привязки хранилища BLOB-объектов включены в пакет расширений службы хранилища Azure, установленный ранее.

1. Замените метод `ProcessQueueMessage` следующим кодом:

   ```cs
   public static void ProcessQueueMessage(
       [QueueTrigger("queue")] string message,
       [Blob("container/{queueTrigger}", FileAccess.Read)] Stream myBlob,
       [Blob("container/copy-{queueTrigger}", FileAccess.Write)] Stream outputBlob,
       ILogger logger)
   {
       logger.LogInformation($"Blob name:{message} \n Size: {myBlob.Length} bytes");
       myBlob.CopyTo(outputBlob);
   }
   ```

1. Создайте другое сообщение в очереди с использованием *Program.cs* в качестве текста сообщения.

1. Запустите проект в локальной среде.

   Сообщение в очереди активирует функцию, которая затем считывает большой двоичный объект, регистрирует в журнале его размер и создает новый большой двоичный объект. В консоли отобразится аналогичное сообщение, но если перейти в окно контейнера большого двоичного объекта и нажать кнопку **Обновить**, вы увидите новый BLOB-объект с именем *copy-Program.cs*.

## <a name="republish-the-updates-to-azure"></a>Повторная публикация обновлений в Azure

1. Щелкните правой кнопкой мыши проект в **обозревателе решений** и выберите пункт **Опубликовать**.

1. В диалоговом окне **Публикация** убедитесь, что выбран текущий профиль, а затем щелкните **Опубликовать**. Подробные результаты публикации выводятся в окно **Вывод**.
 
1. Проверьте функцию в Azure, повторно отправив файл в контейнер больших двоичных объектов и добавив в очередь сообщение с именем отправленного файла. Сообщение будет удалено из очереди, а в контейнере больших двоичных объектов появится копия отправленного файла. 

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали, как создать, запустить и развернуть проект пакета SDK для веб-заданий версии 3.x.

> [!div class="nextstepaction"]
> [Подробнее о пакете SDK веб-заданий](webjobs-sdk-how-to.md)