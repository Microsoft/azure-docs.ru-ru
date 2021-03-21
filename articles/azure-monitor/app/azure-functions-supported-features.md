---
title: Azure Application Insights — поддерживаемые функции функций Azure
description: Поддерживаемые функции Application Insights для службы "Функции Azure"
ms.topic: reference
author: TimothyMothra
ms.author: tilee
ms.date: 4/23/2019
ms.reviewer: mbullwin
ms.openlocfilehash: b44279f31aea8fc02130f1c3d7520f42c648bd4c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97607955"
---
# <a name="application-insights-for-azure-functions-supported-features"></a>Application Insights для функций, поддерживаемых в службе "Функции Azure"

Служба "Функции Azure" предлагает [встроенную интеграцию](../../azure-functions/functions-monitoring.md) с Application Insights, которая доступна через интерфейс ILogger. Ниже приведен список функций, поддерживаемых в настоящее время. Ознакомьтесь с руководством по службе "Функции Azure", чтобы [приступить к работе](../../azure-functions/configure-monitoring.md#enable-application-insights-integration).

Дополнительные сведения о версиях среды выполнения функций см. [здесь](../../azure-functions/functions-versions.md).

Дополнительные сведения о совместимых версиях Application Insights см. в разделе [зависимости](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Logging.ApplicationInsights/).

## <a name="supported-features"></a>Поддерживаемые функции

| Функции Azure                   | V1            | V2 & V3 | 
|-----------------------------------|---------------|------------------|
| | | | 
| **Автоматический сбор**        |               |                  |
| &bull; Запросы                     | Да           | Да              |
| &bull; Исключения                   | Да           | Да              |
| &bull; Счетчики производительности         | Да           | Да              |
| &bull; Зависимости                 |               |                  |
| &nbsp;&nbsp;&nbsp;&mdash; HTTP      |               | Да              |
| &nbsp;&nbsp;&nbsp;&mdash; Служебная шина|               | Да              |
| &nbsp;&nbsp;&nbsp;&mdash; EventHub  |               | Да              |
| &nbsp;&nbsp;&nbsp;&mdash; SQL       |               | Да              |
| | | | 
| **Поддерживаемые функции**              |               |                  |
| &bull; QuickPulse/LiveMetrics       | Да           | Да              | 
| &nbsp;&nbsp;&nbsp;&mdash; Безопасный канал управления |               | Да | 
| &bull; Выборка                     | Да           | Да              | 
| &bull; Пакет пульса                   | | Да              | 
| | | |
| **Correlation** (корреляция).                    |               |                  |
| &bull; Служебная шина                  |               | Да              |
| &bull; EventHub                    |               | Да              |
| | | | 
| **Настраиваемый**                  |               |                  |           
| &bull;Полностью настраивается.<br/>Инструкции см. в комментариях [в этой статье](https://github.com/Microsoft/ApplicationInsights-aspnetcore/issues/759#issuecomment-426687852).<br/>См. [ASP.NET Core](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Custom-Configuration) для всех вариантов.           |               | Да                 | 

## <a name="performance-counters"></a>Счетчики производительности

Автоматический сбор данных счетчиков производительности работает только на компьютерах Windows.

## <a name="live-metrics--secure-control-channel"></a>Интерактивные метрики и безопасный канал управления

Указываемые вами пользовательские критерии фильтра передаются в компонент Live Metrics в пакете SDK для Application Insights. Фильтры могут содержать конфиденциальные сведения, например идентификаторы клиентов. Вы можете защитить канал, используя секретный ключ API. Инструкции см. в разделе [Защита канала управления](./live-stream.md#secure-the-control-channel).

## <a name="sampling"></a>Дискретизация

Служба "Функции Azure" включает в своей конфигурации выборку по умолчанию. Дополнительные сведения см. в разделе [о настройке выборки](../../azure-functions/configure-monitoring.md#configure-sampling).

Если проект использует зависимость от пакета SDK Application Insights для ручного отслеживания телеметрии, то может наблюдаться странное поведение, если конфигурация выборки отличается от конфигурации выборки функций. 

Мы рекомендуем использовать ту же конфигурацию, что и функции. В **функциях версии 2** можно получить ту же конфигурацию с помощью внедрения зависимостей в конструкторе:

```csharp
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.Extensibility;

public class Function1 
{

    private readonly TelemetryClient telemetryClient;

    public Function1(TelemetryConfiguration configuration)
    {
        this.telemetryClient = new TelemetryClient(configuration);
    }

    [FunctionName("Function1")]
    public async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req, ILogger logger)
    {
        this.telemetryClient.TrackTrace("C# HTTP trigger function processed a request.");
    }
}
```
