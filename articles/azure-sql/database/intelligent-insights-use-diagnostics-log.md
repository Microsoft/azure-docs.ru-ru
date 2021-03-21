---
title: Журнал диагностики производительности Intelligent Insights
description: Intelligent Insights предоставляет журнал диагностики базы данных SQL Azure и проблем с производительностью Управляемый экземпляр SQL Azure
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: danimir
ms.author: danil
ms.reviewer: wiassaf, sstein
ms.date: 06/12/2020
ms.openlocfilehash: b03c21eea18c966616154b5cfc5df5d8924fd335
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100589312"
---
# <a name="use-the-intelligent-insights-performance-diagnostics-log-of-azure-sql-database-and-azure-sql-managed-instance-performance-issues"></a>Использование журнала диагностики производительности Intelligent Insights базы данных SQL Azure и проблем с производительностью Управляемый экземпляр SQL Azure
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

На этой странице содержатся сведения о том, как использовать журнал диагностики производительности, созданный [Intelligent Insights](intelligent-insights-overview.md) базы данных SQL Azure и проблем с производительностью управляемый экземпляр SQL Azure, его форматом и данными, которые они содержат для пользовательских нужд разработки. Этот журнал диагностики можно отправить в [журналы Azure Monitor](../../azure-monitor/insights/azure-sql.md), [концентраторы событий Azure](../../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs), службу [хранилища Azure](metrics-diagnostic-telemetry-logging-streaming-export-configure.md#stream-into-azure-storage)или стороннее решение для настраиваемых оповещений DevOps и отчетов.

> [!NOTE]
> Intelligent Insights — это предварительная версия функции, недоступная в следующих регионах: Западная Европа, Северная Европа, Западная часть США 1 и Восточная часть США 1.

## <a name="log-header"></a>Заголовок журнала

Журнал диагностики использует стандартный формат JSON для вывода результатов Intelligent Insights. Для доступа к журналу Intelligent Insights используется фиксированное значение свойства категории "SQLInsights".

Заголовок журнала является типичным и состоит из метки времени (TimeGenerated), которая показывает, когда была создана запись. Он также включает идентификатор ресурса (ResourceId), который ссылается на конкретную базу данных, к которой относится запись. Категория (Category), уровень (Level) и имя операции (OperationName) являются фиксированными свойствами, значения которых не изменяются. Они указывают, что запись журнала является информационной и что она сгенерирована в Intelligent Insights (SQLInsights).

```json
"TimeGenerated" : "2017-9-25 11:00:00", // time stamp of the log entry
"ResourceId" : "database identifier", // value points to a database resource
"Category": "SQLInsights", // fixed property
"Level" : "Informational", // fixed property
"OperationName" : "Insight", // fixed property
```

## <a name="issue-id-and-database-affected"></a>ИД проблемы и затронутая база данных

Свойства идентификации проблемы (issueId_d) позволяет однозначно отслеживать проблемы с производительностью вплоть до их устранения. Несколько записей об одном и том же событии в журнале будут иметь одинаковый идентификатор проблемы.

Помимо идентификатора проблемы в журнал диагностики записываются метки времени начала (intervalStartTime_t) и окончания (intervalEndTme_t) определенного события, связанного с проблемой.

Свойство эластичного пула (elasticPoolName_s) указывает, какому эластичному пулу принадлежит база данных, в которой обнаружена проблема. Если база данных не является частью эластичного пула, этому свойству значение не присваивается. База данных, в которой выявлена проблема, указывается в свойстве имени базы данных (databaseName_s).

```json
"intervalStartTime_t": "2017-9-25 11:00", // start of the issue reported time stamp
"intervalEndTme_t":"2017-9-25 12:00", // end of the issue reported time stamp
"elasticPoolName_s" : "", // resource elastic pool (if applicable)
"databaseName_s" : "db_name", // database name
"issueId_d" : 1525, // unique ID of the issue detected
"status_s" : "Active" // status of the issue – possible values: "Active", "Verifying", and "Complete"
```

## <a name="detected-issues"></a>Обнаруженные проблемы

Следующий раздел журнала производительности Intelligent Insights содержит проблемы с производительностью, выявленные с помощью встроенного искусственного интеллекта. Выявленные проблемы указываются в свойствах в журнале диагностики формата JSON вместе с категорией проблемы, ее влиянием, затронутыми запросами и метриками. Свойства обнаружений могут содержать несколько выявленных проблем с производительностью.

Выявленные проблемы с производительностью отображаются посредством следующей структуры свойства обнаружений.

```json
"detections_s" : [{
"impact" : 1 to 3, // impact of the issue detected, possible values 1-3 (1 low, 2 moderate, 3 high impact)
"category" : "Detectable performance pattern", // performance issue detected, see the table
"details": <Details outputted> // details of an issue (see the table)
}]
```

В следующей таблице приведены выявляемые шаблоны снижения производительности и сведения, выводимые в журнал диагностики.

### <a name="detection-category"></a>Категория выявленной проблемы

Свойство категории (category) описывает категорию выявляемых шаблонов снижения производительности. В следующей таблице приведены все возможные категории выявляемых шаблонов снижения производительности. Дополнительные сведения см. в статье [Устранение проблем с производительностью базы данных SQL Azure с помощью Intelligent Insights](intelligent-insights-troubleshoot-performance.md).

Сведения, выводимые в файл журнала диагностики, зависят от выявленной проблемы с производительностью.

| Выявляемые шаблоны снижения производительности | Выводимые сведения |
| :------------------- | ------------------- |
| достижение лимитов ресурсов; | <li>Затронутые ресурсы</li><li>Хэши запросов</li><li>Процент потребления ресурсов</li> |
| Увеличение рабочей нагрузки | <li>Количество запросов, объем выполнения которых увеличился</li><li>Хэши запросов, больше всего повлиявших на увеличение рабочей нагрузки</li> |
| Нехватка памяти | <li>Клерк памяти</li> |
| Блокировка | <li>Хэши затронутых запросов</li><li>Хэши блокирующих запросов</li> |
| Повышенное значение MAXDOP | <li>Хэши запросов</li><li>Время ожидания CXP</li><li>Время ожидания</li> |
| Состязание за кратковременную блокировку страниц | <li>Хэши запросов, которые приводят к состязанию</li> |
| Отсутствующий индекс | <li>Хэши запросов</li> |
| Новый запрос | <li>Хэши новых запросов</li> |
| Необычная статистика ожидания | <li>Типы необычного времени ожидания</li><li>Хэши запросов</li><li>Время ожидания запросов</li> |
| Состязание за tempdb | <li>Хэши запросов, которые приводят к состязанию</li><li>Сопоставление запросов и общего времени ожидания из-за состязания за кратковременную блокировку страниц базы данных [%]</li> |
| Нехватка DTU эластичного пула | <li>Эластичный пул</li><li>Базы данных, использующие больше всего DTU</li><li>Процент DTU, используемых наиболее ресурсоемким участником пула</li> |
| Снижение эффективности плана | <li>Хэши запросов</li><li>Идентификаторы высокоэффективных планов</li><li>Идентификаторы низкоэффективных планов</li> |
| Изменение значения конфигурации уровня базы данных | <li>Изменения конфигурации уровня базы данных по сравнению со значениями по умолчанию</li> |
| Медленная работа клиента | <li>Хэши запросов</li><li>Время ожидания</li> |
| Переход на более низкую ценовую категорию | <li>Текстовое уведомление</li> |

### <a name="impact"></a>Влияние

Свойство влияния (impact) показывает, насколько база данных подвержена выявленной проблеме по шкале от 1 до 3, где 3 — это наибольшая подверженность, 2 — умеренная, а 1 — наименьшая. Значение impact может использоваться в качестве входных данных для автоматизации настраиваемых оповещений, в зависимости от конкретных потребностей. Свойство затронутых запросов (QueryHashes) содержит список хэшей запросов, затронутых выявленной проблемой.

### <a name="impacted-queries"></a>Затронутые запросы

Следующий раздел журнала Intelligent Insights содержит сведения о конкретных запросах, которые были затронуты выявленной проблемой с производительностью. Эти сведения предоставляются в виде массива объектов, внедренного в свойство impact_s. Свойство влияния состоит из сущностей и метрик. Сущности относятся к определенному запросу (Type: Query). Уникальный хэш запроса указывается в свойстве значения (Value). Кроме того, каждый указанный запрос сопровождается метрикой и значением, которые описывают выявленную проблему с производительностью.

В следующем примере журнала было обнаружено, что время выполнения запроса с хэшем 0x9102EXZ4 увеличилось (Metric: DurationIncreaseSeconds). Значение 110 секунд означает, что этот запрос выполнялся на 110 секунд дольше. Так как могут быть выявлены несколько запросов, этот раздел журнала может содержать несколько записей о запросах.

```json
"impact" : [{
"entity" : {
"Type" : "Query", // type of entity - query
"Value" : "query hash value", // for example "0x9102EXZ4" query hash value },
"Metric" : "DurationIncreaseSeconds", // measured metric and the measurement unit (in this case seconds)
"Value" : 110 // value of the measured metric (in this case seconds)
}]
```

### <a name="metrics"></a>Метрики

Единицы измерения каждой полученной метрики предоставляются в свойстве метрики (metric). Его возможные значения: seconds, number и percentage. Значение измеряемой метрики указывается в свойстве значения (value).

Свойство DurationIncreaseSeconds указывается в секундах, а для CriticalErrorCount единицей измерения является число, представляющее собой количество ошибок.

```json
"metric" : "DurationIncreaseSeconds", // issue metric type – possible values: DurationIncreaseSeconds, CriticalErrorCount, WaitingSeconds
"value" : 102 // value of the measured metric (in this case seconds)
```

## <a name="root-cause-analysis-and-improvement-recommendations"></a>Анализ первопричин и рекомендации по повышению производительности

Последняя часть журнала производительности Intelligent Insights содержит результат автоматического анализа первопричин выявленной проблемы снижения производительности. Эта информация отображается в понятном виде в свойстве анализа первопричин (rootCauseAnalysis_s). Везде, где это возможно, в журнале содержатся рекомендации по улучшению.

```json
// example of reported root cause analysis of the detected performance issue, in a human-readable format

"rootCauseAnalysis_s" : "High data IO caused performance to degrade. It seems that this database is missing some indexes that could help."
```

Журнал производительности Intelligent Insights можно использовать с [Azure Monitor журналами]( https://docs.microsoft.com/azure/log-analytics/log-analytics-azure-sql) или решениями сторонних производителей для настраиваемых оповещений и отчетов DevOps.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с понятиями [Intelligent Insights](intelligent-insights-overview.md).
- Узнайте, как [устранять проблемы с производительностью Intelligent Insights](intelligent-insights-troubleshoot-performance.md).
- Узнайте, как [отслеживать проблемы с производительностью с помощью аналитика SQL Azure](../../azure-monitor/insights/azure-sql.md).
- Узнайте, как [собирать и использовать данные журнала из ресурсов Azure](../../azure-monitor/essentials/platform-logs-overview.md).