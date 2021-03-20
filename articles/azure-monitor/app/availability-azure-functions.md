---
title: Создание и запуск пользовательских тестов доступности с помощью функций Azure
description: В этом документе рассматривается создание функции Azure с TrackAvailability (), которая будет выполняться периодически в соответствии с конфигурацией, заданной в функции TimerTrigger. Результаты этого теста будут отправлены в ресурс Application Insights, где вы сможете запрашивать данные о результатах доступности и получать оповещения о них. Настроенные тесты позволяют создавать более сложные тесты доступности, чем возможно с помощью пользовательского интерфейса портала, мониторинга приложения в виртуальной сети Azure, изменения адреса конечной точки или создания теста доступности, если он недоступен в вашем регионе.
ms.topic: conceptual
ms.date: 05/04/2020
ms.openlocfilehash: 98d9eaadb31ffdeabe85752f7c76bdd4f7c0d4f3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100589931"
---
# <a name="create-and-run-custom-availability-tests-using-azure-functions"></a>Создание и запуск пользовательских тестов доступности с помощью функций Azure

В этой статье рассматривается создание функции Azure с TrackAvailability (), которая будет выполняться периодически в соответствии с конфигурацией, заданной в функции TimerTrigger, с помощью собственной бизнес-логики. Результаты этого теста будут отправлены в ресурс Application Insights, где вы сможете запрашивать данные о результатах доступности и получать оповещения о них. Это позволяет создавать настраиваемые тесты, аналогичные тому, что можно сделать с помощью [мониторинга доступности](./monitor-web-app-availability.md) на портале. Настроенные тесты позволяют создавать более сложные тесты доступности, чем возможно с помощью пользовательского интерфейса портала, мониторинга приложения в виртуальной сети Azure, изменения адреса конечной точки или создания теста доступности, даже если эта функция недоступна в вашем регионе.

> [!NOTE]
> Этот пример предназначен только для того, чтобы продемонстрировать, как работает вызов API TrackAvailability () в функции Azure. Не нужно писать базовый код теста HTTP или бизнес-логику, которая потребовалась бы для того, чтобы превратить это в полностью функциональный тест доступности. По умолчанию при пошаговом выполнении этого примера будет создан тест доступности, который всегда будет создавать ошибку.

## <a name="create-timer-triggered-function"></a>Создание функции, активируемой с таймером

- Если у вас есть Application Insights ресурс:
    - По умолчанию функции Azure создают ресурс Application Insights, но если вы хотите использовать один из уже созданных ресурсов, необходимо указать это во время создания.
    - Следуйте инструкциям, чтобы [создать ресурс функций Azure и функцию, активируемую с помощью таймера](../../azure-functions/functions-create-scheduled-function.md) ("закончить перед очисткой") со следующими параметрами.
        -  Выберите вкладку **мониторинг** в верхней части окна.

            ![ Создание приложения "функции Azure" с собственным ресурсом App Insights](media/availability-azure-functions/create-function-app.png)

        - Выберите Application Insights раскрывающийся список и введите или выберите имя ресурса.

            ![Выбор существующего Application Insights ресурса](media/availability-azure-functions/app-insights-resource.png)

        - Нажмите **Проверить и создать**.
- Если у вас еще нет созданного ресурса Application Insights для функции, активируемой с помощью таймера:
    - По умолчанию при создании приложения "функции Azure" будет создан ресурс Application Insights.
    - Следуйте инструкциям, чтобы [создать ресурс функций Azure и функцию, активируемую с помощью таймера](../../azure-functions/functions-create-scheduled-function.md) (до завершения очистки).

## <a name="sample-code"></a>Пример кода

Скопируйте приведенный ниже код в файл run. CSX (это заменит уже существующий код). Для этого перейдите в приложение функций Azure и выберите функцию триггера таймера слева.

>[!div class="mx-imgBorder"]
>![Функция Azure Run. CSX в портал Azure](media/availability-azure-functions/runcsx.png)

> [!NOTE]
> Для адреса конечной точки, который будет использоваться: `EndpointAddress= https://dc.services.visualstudio.com/v2/track` . Если ресурс не находится в регионе, например Azure для государственных организаций или Azure для Китая, в этом случае обратитесь к этой статье, чтобы узнать о [переопределении конечных точек по умолчанию](./custom-endpoints.md#regions-that-require-endpoint-modification) и выбрать соответствующую конечную точку канала телеметрии для вашего региона.

```C#
#load "runAvailabilityTest.csx"
 
using System;
using System.Diagnostics;
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;
 
// The Application Insights Instrumentation Key can be changed by going to the overview page of your Function App, selecting configuration, and changing the value of the APPINSIGHTS_INSTRUMENTATIONKEY Application setting.
// DO NOT replace the code below with your instrumentation key, the key's value is pulled from the environment variable/application setting key/value pair.
private static readonly string instrumentationKey = Environment.GetEnvironmentVariable("APPINSIGHTS_INSTRUMENTATIONKEY");
 
//[CONFIGURATION_REQUIRED]
// If your resource is in a region like Azure Government or Azure China, change the endpoint address accordingly.
// Visit https://docs.microsoft.com/azure/azure-monitor/app/custom-endpoints#regions-that-require-endpoint-modification for more details.
private const string EndpointAddress = "https://dc.services.visualstudio.com/v2/track";
 
private static readonly TelemetryConfiguration telemetryConfiguration = new TelemetryConfiguration(instrumentationKey, new InMemoryChannel { EndpointAddress = EndpointAddress });
private static readonly TelemetryClient telemetryClient = new TelemetryClient(telemetryConfiguration);
 
public async static Task Run(TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"Entering Run at: {DateTime.Now}");
 
    if (myTimer.IsPastDue)
    {
        log.LogWarning($"[Warning]: Timer is running late! Last ran at: {myTimer.ScheduleStatus.Last}");
    }
 
    // [CONFIGURATION_REQUIRED] provide {testName} accordingly for your test function
    string testName = "AvailabilityTestFunction";
 
    // REGION_NAME is a default environment variable that comes with App Service
    string location = Environment.GetEnvironmentVariable("REGION_NAME");
 
    log.LogInformation($"Executing availability test run for {testName} at: {DateTime.Now}");
    string operationId = Guid.NewGuid().ToString("N");
 
    var availability = new AvailabilityTelemetry
    {
        Id = operationId,
        Name = testName,
        RunLocation = location,
        Success = false
    };
 
    var stopwatch = new Stopwatch();
    stopwatch.Start();
 
    try
    {
        await RunAvailbiltyTestAsync(log);
        availability.Success = true;
    }
    catch (Exception ex)
    {
        availability.Message = ex.Message;
 
        var exceptionTelemetry = new ExceptionTelemetry(ex);
        exceptionTelemetry.Context.Operation.Id = operationId;
        exceptionTelemetry.Properties.Add("TestName", testName);
        exceptionTelemetry.Properties.Add("TestLocation", location);
        telemetryClient.TrackException(exceptionTelemetry);
    }
    finally
    {
        stopwatch.Stop();
        availability.Duration = stopwatch.Elapsed;
        availability.Timestamp = DateTimeOffset.UtcNow;
 
        telemetryClient.TrackAvailability(availability);
        // call flush to ensure telemetry is sent
        telemetryClient.Flush();
    }
}

```

Справа в разделе Просмотр файлов выберите **Добавить**. Вызовите новый файл **Function. proj** со следующей конфигурацией.

```C#
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>netstandard2.0</TargetFramework>
    </PropertyGroup>
    <ItemGroup>
        <PackageReference Include="Microsoft.ApplicationInsights" Version="2.15.0" /> <!-- Ensure you’re using the latest version -->
    </ItemGroup>
</Project>

```

>[!div class="mx-imgBorder"]
>![Справа выберите Добавить. Назовите файл function. proj](media/availability-azure-functions/addfile.png)

Справа в разделе Просмотр файлов выберите **Добавить**. Вызовите новый файл **рунаваилабилититест. CSX** со следующей конфигурацией.

```C#
public async static Task RunAvailbiltyTestAsync(ILogger log)
{
    // Add your business logic here.
    throw new NotImplementedException();
}

```

## <a name="check-availability"></a>Проверка доступности

Чтобы убедиться, что все работает, можно просмотреть диаграмму на вкладке доступность Application Insights ресурса.

> [!NOTE]
> Если вы реализовали собственную бизнес-логику в Рунаваилабилититест. CSX, вы увидите результаты, как на снимках экрана ниже, если вы не сделали этого, вы увидите результаты с ошибками. Тесты, созданные с помощью `TrackAvailability()` , будут отображаться с **настраиваемым** рядом с именем теста.

>[!div class="mx-imgBorder"]
>![Вкладка "доступность" с успешными результатами](media/availability-azure-functions/availability-custom.png)

Чтобы просмотреть сведения о сквозной транзакции, выберите **успех** или **сбой** в разделе Детализация, а затем выберите пример. Можно также перейти к подробным сведениям о транзакции, выбрав на диаграмме точку данных.

>[!div class="mx-imgBorder"]
>![Выбор примера теста доступности](media/availability-azure-functions/sample.png)

>[!div class="mx-imgBorder"]
>![Сведения о сквозной транзакции](media/availability-azure-functions/end-to-end.png)

Если вы запускали все как есть (без добавления бизнес-логики), вы увидите, что тест не пройден.

## <a name="query-in-logs-analytics"></a>Запрос в журналах (аналитика)

С помощью журналов (Analytics) можно просматривать результаты доступности, зависимости и многое другое. Дополнительные сведения о журналах см. в статье [Обзор запросов журналов](../logs/log-query-overview.md).

>[!div class="mx-imgBorder"]
>![Результаты доступности](media/availability-azure-functions/availabilityresults.png)

>[!div class="mx-imgBorder"]
>![Снимок экрана показывает новую вкладку запросов с зависимостями, ограниченными 50.](media/availability-azure-functions/dependencies.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Схема сопоставления приложений](./app-map.md)
- [Диагностика транзакции](./transaction-diagnostics.md)

