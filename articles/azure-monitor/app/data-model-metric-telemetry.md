---
title: Модель данных для телеметрии метрик в Azure Application Insights
description: Модель данных Application Insights для телеметрии метрик
ms.topic: conceptual
ms.date: 04/25/2017
ms.reviewer: sergkanz
ms.openlocfilehash: f8e787b8a94987e7d68e12a0e4e69f4c63e21e19
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87320584"
---
# <a name="metric-telemetry-application-insights-data-model"></a>Телеметрия метрик: модель данных Application Insights

[Application Insights](./app-insights-overview.md) поддерживает два типа телеметрии метрик — отдельное измерение и предварительно вычисленная метрика. Отдельное измерение содержит только имя и значение. Предварительно вычисленная метрика указывает минимальное и максимальное значение метрики в интервале статистической обработки, а также его стандартное отклонение.

Телеметрия предварительно вычисленных метрик предполагает, что период статистической обработки составлял одну минуту.

Существует несколько известных имен метрик, поддерживаемых Application Insights. Эти метрики помещаются в таблицу performanceCounters.

Метрика, представляющая счетчики системы и процессов:

| **Имя .NET**             | **Платформонезависимое имя** | **Имя REST API** | **Описание**
| ------------------------- | -------------------------- | ----------------- | ---------------- 
| `\Processor(_Total)\% Processor Time` | Ведется работа... | [processorCpuPercentage](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FprocessorCpuPercentage) | Общее использование ЦП на компьютере
| `\Memory\Available Bytes`                 | Ведется работа... | [memoryAvailableBytes](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FmemoryAvailableBytes) | Объем физической памяти в байтах, доступной для процессов, выполняющихся на компьютере. Это общий объем обнуленных страниц памяти, свободной памяти и резервной памяти. Свободная память — это пространство, готовое к использованию; обнуленные страницы памяти — это страницы, которые заполняются нулями, чтобы для последующих процессов были недоступны данные предыдущего; резервная память — это память, которая удалена из рабочего набора процесса (его физическая память) в маршруте к диску, но которую можно вызвать повторно. См. сведения об [объекте памяти](/previous-versions/ms804008(v=msdn.10)).
| `\Process(??APP_WIN32_PROC??)\% Processor Time` | Ведется работа... | [processCpuPercentage](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FprocessCpuPercentage) | Использование ЦП для процесса, где размещается приложение
| `\Process(??APP_WIN32_PROC??)\Private Bytes`      | Ведется работа... | [processPrivateBytes](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FprocessPrivateBytes) | Использование памяти для процесса, где размещается приложение
| `\Process(??APP_WIN32_PROC??)\IO Data Bytes/sec` | Ведется работа... | [processIOBytesPerSecond](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FprocessIOBytesPerSecond) | Частота операций ввода-вывода, выполняемых процессом, где размещается приложение
| `\ASP.NET Applications(??APP_W3SVC_PROC??)\Requests/Sec`             | Ведется работа... | [requestsPerSecond](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FrequestsPerSecond) | Частота запросов, обрабатываемых приложением 
| `\.NET CLR Exceptions(??APP_CLR_PROC??)\# of Exceps Thrown / sec`    | Ведется работа... | [exceptionsPerSecond](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FexceptionsPerSecond) | Частота исключений, выдаваемых приложением
| `\ASP.NET Applications(??APP_W3SVC_PROC??)\Request Execution Time`   | Ведется работа... | [requestExecutionTime](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FrequestExecutionTime) | Среднее время выполнения запросов
| `\ASP.NET Applications(??APP_W3SVC_PROC??)\Requests In Application Queue` | Ведется работа... | [requestsInQueue](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FrequestsInQueue) | Количество запросов в очереди, ожидающих обработки

## <a name="name"></a>Имя

Имя метрики, которое нужно отобразить на портале Application Insights и в пользовательском интерфейсе. 

## <a name="value"></a>Значение

Отдельное значение для измерения. Сумма отдельных измерений для агрегата.

## <a name="count"></a>Count

Вес вычисленной метрики. Не следует задавать для измерения.

## <a name="min"></a>Min

Минимальное значение вычисленной метрики. Не следует задавать для измерения.

## <a name="max"></a>Макс.

Максимальное значение вычисленной метрики. Не следует задавать для измерения.

## <a name="standard-deviation"></a>Standard deviation

Стандартное отклонение вычисленной метрики. Не следует задавать для измерения.

## <a name="custom-properties"></a>Пользовательские свойства

Если пользовательское свойство метрики `CustomPerfCounter` имеет значение `true`, это означает, что метрика представляет счетчик производительности Windows. Эти метрики помещаются в таблицу performanceCounters, а не в customMetrics. Кроме того, анализируется имя этой метрики для извлечения имени категории, счетчика и экземпляра.

[!INCLUDE [application-insights-data-model-properties](../../../includes/application-insights-data-model-properties.md)]

## <a name="next-steps"></a>Дальнейшие действия

- Вы можете узнать, как использовать [API Application Insights для пользовательских событий и метрик](./api-custom-events-metrics.md#trackmetric).
- В [этой статье](data-model.md) представлены типы данных и модель данных для Application Insights.
- Ознакомление с [платформами](./platforms.md), поддерживаемыми Application Insights.

