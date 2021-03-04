---
title: Примеры запросов журнала Azure Monitor | Документация Майкрософт
description: Примеры запросов журнала в Azure Monitor, выполненных с помощью языка запросов Kusto.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/16/2020
ms.openlocfilehash: 56866d8e255f15a93aaa9133f9cda9de4fdd9fd5
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102030806"
---
# <a name="azure-monitor-log-query-examples"></a>Примеры запроса журнала Azure Monitor
Эта статья содержит несколько примеров [запросов](./log-query-overview.md), составленных на основе [языка запросов Kusto](/azure/kusto/query/) для получения разных типов данных журнала из Azure Monitor. Для консолидации и анализа данных используются разные методы, и на основе этих примеров вы сможете определить, какие стратегии лучше всего подойдут под ваши конкретные требования.  

Дополнительные сведения о ключевых словах, используемых в этих примерах, можно получить в [справочнике по языку Kusto](/azure/kusto/query/). Изучите [эту статью](./get-started-queries.md) по созданию запросов, если вы еще не работали с Azure Monitor.

## <a name="events"></a>События

### <a name="search-application-level-events-described-as-cryptographic"></a>Поиск событий шифрования на уровне приложения
Этот пример выполняет поиск записей в таблице **Events** (События), у которых **EventLog** имеет значение _Application_ (Приложение), а **RenderedDescription** содержит значение _cryptographic_ (шифрование). В поиск включаются записи за последние 24 часа.

```Kusto
Event
| where EventLog == "Application" 
| where TimeGenerated > ago(24h) 
| where RenderedDescription contains "cryptographic"
```

### <a name="search-events-related-to-unmarshaling"></a>Поиск событий, связанных с демаршалингом
Поиск в таблицах **Event** и **SecurityEvents** записей, в которых упоминается _unmashaling_ (демаршалинг).

```Kusto
search in (Event, SecurityEvent) "unmarshaling"
```

## <a name="heartbeat"></a>Пульс

### <a name="chart-a-week-over-week-view-of-the-number-of-computers-sending-data"></a>Составление понедельного представления количества компьютеров, отправляющих данные

Следующий пример составляет схему, которая отображает количество отдельных компьютеров, отправлявших пакеты пульса за каждую неделю.

```Kusto
Heartbeat
| where TimeGenerated >= startofweek(ago(21d))
| summarize dcount(Computer) by endofweek(TimeGenerated) | render barchart kind=default
```

### <a name="find-stale-computers"></a>Поиск неактивных компьютеров

Следующий пример находит компьютеры, которые были активны в предыдущий день, но за последний час не отправляли пакеты пульса.

```Kusto
Heartbeat
| where TimeGenerated > ago(1d)
| summarize LastHeartbeat = max(TimeGenerated) by Computer
| where isnotempty(Computer)
| where LastHeartbeat < ago(1h)
```

### <a name="get-the-latest-heartbeat-record-per-computer-ip"></a>Получение последней записи пульса по IP-адресу компьютера

Этот пример возвращает запись о последнем пакете пульса, полученном для каждого IP-адреса.
```Kusto
Heartbeat
| summarize arg_max(TimeGenerated, *) by ComputerIP
```

### <a name="match-protected-status-records-with-heartbeat-records"></a>Сопоставление записей о состоянии защиты с записями пакетов пульса

Этот пример находит записи состояния защиты и записи пакетов пульса, у которых совпадают значения полей времени и поле Computer.
Обратите внимание, что поле времени округляется до ближайшей минуты. Для этого мы используем операцию bin во время выполнения: `round_time=bin(TimeGenerated, 1m)`.

```Kusto
let protection_data = ProtectionStatus
    | project Computer, DetectionId, round_time=bin(TimeGenerated, 1m);
let heartbeat_data = Heartbeat
    | project Computer, Category, round_time=bin(TimeGenerated, 1m);
protection_data | join (heartbeat_data) on Computer, round_time
```

### <a name="server-availability-rate"></a>Коэффициент доступности сервера

Коэффициент доступности сервера вычисляется на основе записей пакетов пульса. Доступность определяется как "не менее 1 пакета пульса в час".
Таким образом, если сервер был доступен в 98 из 100 часов, его коэффициент доступности составляет 98 %.

```Kusto
let start_time=startofday(datetime("2018-03-01"));
let end_time=now();
Heartbeat
| where TimeGenerated > start_time and TimeGenerated < end_time
| summarize heartbeat_per_hour=count() by bin_at(TimeGenerated, 1h, start_time), Computer
| extend available_per_hour=iff(heartbeat_per_hour>0, true, false)
| summarize total_available_hours=countif(available_per_hour==true) by Computer 
| extend total_number_of_buckets=round((end_time-start_time)/1h)+1
| extend availability_rate=total_available_hours*100/total_number_of_buckets
```


## <a name="multiple-data-types"></a>Несколько типов данных

### <a name="chart-the-record-count-per-table"></a>Диаграмма количества записей в таблицах
Следующий пример собирает все записи из всех таблиц за последние пять часов и подсчитывает количество записей в каждой таблице. Результат отображается в виде временной диаграммы.

```Kusto
union withsource=sourceTable *
| where TimeGenerated > ago(5h) 
| summarize count() by bin(TimeGenerated,10m), sourceTable
| render timechart
```

### <a name="count-all-logs-collected-over-the-last-hour-by-type"></a>Подсчет всех журналов, собранных за последний час, с разбивкой по типам
Следующий пример находит все отчеты за последний час и подсчитывает количество записей в каждой таблице с разбивкой по полю **Type**. Результаты отображаются в виде столбчатой диаграммы.

```Kusto
search *
| where TimeGenerated > ago(1h) 
| summarize CountOfRecords = count() by Type
| render barchart
```

## <a name="azurediagnostics"></a>AzureDiagnostics

### <a name="count-azure-diagnostics-records-per-category"></a>Подсчет записей диагностики Azure с разбивкой по категориям
Этот пример подсчитывает все записи системы диагностики Azure для каждой уникальной категории.

```Kusto
AzureDiagnostics 
| where TimeGenerated > ago(1d)
| summarize count() by Category
```

### <a name="get-a-random-record-for-each-unique-category"></a>Получение случайных записей для каждой уникальной категории
Этот пример получает по одной случайной записи системы диагностики Azure для каждой уникальной категории.

```Kusto
AzureDiagnostics
| where TimeGenerated > ago(1d) 
| summarize any(*) by Category
```

### <a name="get-the-latest-record-per-category"></a>Получение последних записей по категориям
Этот пример получает последнюю запись системы диагностики Azure для каждой уникальной категории.

```Kusto
AzureDiagnostics
| where TimeGenerated > ago(1d) 
| summarize arg_max(TimeGenerated, *) by Category
```

## <a name="network-monitoring"></a>Мониторинг сетей.

### <a name="computers-with-unhealthy-latency"></a>Компьютеры с ненормальной задержкой
Этот пример создает список уникальных компьютеров, демонстрирующих ненормальную задержку.

```Kusto
NetworkMonitoring 
| where LatencyHealthState <> "Healthy" 
| where Computer != "" 
| distinct Computer
```

## <a name="performance"></a>Производительность

### <a name="join-computer-perf-records-to-correlate-memory-and-cpu"></a>Объединение записей производительности компьютеров для сопоставления данных о памяти и ЦП
Этот пример сопоставляет записи **perf** для определенного компьютера и создает две временные диаграммы: по средней загрузке ЦП и максимальному объему памяти.

```Kusto
let StartTime = now()-5d;
let EndTime = now()-4d;
Perf
| where CounterName == "% Processor Time"  
| where TimeGenerated > StartTime and TimeGenerated < EndTime
| project TimeGenerated, Computer, cpu=CounterValue 
| join kind= inner (
   Perf
    | where CounterName == "% Used Memory"  
    | where TimeGenerated > StartTime and TimeGenerated < EndTime
    | project TimeGenerated , Computer, mem=CounterValue 
) on TimeGenerated, Computer
| summarize avgCpu=avg(cpu), maxMem=max(mem) by TimeGenerated bin=30m  
| render timechart
```

### <a name="perf-cpu-utilization-graph-per-computer"></a>Диаграмма производительности — использование ЦП с разбивкой по компьютерам
Этот пример вычисляет и отображает в виде диаграммы данные об использовании ЦП для компьютеров, имена которые начинаются со строки _Contoso_.

```Kusto
Perf
| where TimeGenerated > ago(4h)
| where Computer startswith "Contoso" 
| where CounterName == @"% Processor Time"
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 15m) 
| render timechart
```

## <a name="protection-status"></a>Состояние защиты

### <a name="computers-with-non-reporting-protection-status-duration"></a>Длительность отсутствия ответов о состоянии защиты для компьютеров
Этот пример перечисляет компьютеры, которые имеют состояние защиты _Not Reporting_ (Нет отчетов), с указанием длительности нахождения в этом состоянии.

```Kusto
ProtectionStatus
| where ProtectionStatus == "Not Reporting"
| summarize count(), startNotReporting = min(TimeGenerated), endNotReporting = max(TimeGenerated) by Computer, ProtectionStatusDetails
| join ProtectionStatus on Computer
| summarize lastReporting = max(TimeGenerated), startNotReporting = any(startNotReporting), endNotReporting = any(endNotReporting) by Computer
| extend durationNotReporting = endNotReporting - startNotReporting
```

### <a name="match-protected-status-records-with-heartbeat-records"></a>Сопоставление записей о состоянии защиты с записями пакетов пульса
Этот пример находит записи состояния защиты и записи пакетов пульса, у которых совпадают значение полей time и Computer.
Поле времени округляется до ближайшей минуты с помощью функции **bin**.

```Kusto
let protection_data = ProtectionStatus
    | project Computer, DetectionId, round_time=bin(TimeGenerated, 1m);
let heartbeat_data = Heartbeat
    | project Computer, Category, round_time=bin(TimeGenerated, 1m);
protection_data | join (heartbeat_data) on Computer, round_time
```


## <a name="security-records"></a>Записи безопасности

### <a name="count-security-events-by-activity-id"></a>Подсчет событий безопасности по идентификатору действия


В этом примере используется фиксированная структура столбца **Activity** : \<ID\> - \<Name\> .
Он анализирует значение **Activity** и разделяет его на два новых столбца, а затем подсчитывает число вхождений для каждого значения **activityID**.

```Kusto
SecurityEvent
| where TimeGenerated > ago(30m) 
| project Activity 
| parse Activity with activityID " - " activityDesc
| summarize count() by activityID
```

### <a name="count-security-events-related-to-permissions"></a>Подсчет событий безопасности, связанных с разрешениями
Этот пример отображает количество записей **securityEvent**, у которых столбец **Activity** (Активность) содержит слово _Permissions_ (Разрешения) целиком. Запрос применяется к записям, созданным за последние 30 минут.

```Kusto
SecurityEvent
| where TimeGenerated > ago(30m)
| summarize EventCount = countif(Activity has "Permissions")
```

### <a name="find-accounts-that-failed-to-log-in-from-computers-with-a-security-detection"></a>Поиск учетных записей, которые не смогли выполнить вход с компьютеров, для которых сработало обнаружение системой безопасности
Этот пример выполняет поиск и подсчет учетных записей, которым не удалось выполнить вход с компьютеров, на которых сработало обнаружение системой безопасности.

```Kusto
let detections = toscalar(SecurityDetection
| summarize makeset(Computer));
SecurityEvent
| where Computer in (detections) and EventID == 4624
| summarize count() by Account
```

### <a name="is-my-security-data-available"></a>Доступны ли данные о безопасности?
Изучение данных часто начинается с проверки доступности нужных данных. Этот пример отображает количество записей **SecurityEvent** за последние 30 минут.

```Kusto
SecurityEvent 
| where TimeGenerated  > ago(30m) 
| count
```

### <a name="parse-activity-name-and-id"></a>Анализ имени и идентификатора действия
Два приведенных ниже примера зависят от фиксированной структуры столбца **действия** : \<ID\> - \<Name\> . Первый пример с помощью оператора **parse** присваивает значения двум новым столбцам: **activityID** и **activityDesc**.

```Kusto
SecurityEvent
| take 100
| project Activity 
| parse Activity with activityID " - " activityDesc
```

В этом примере используется оператор **split** для создания массива отдельных значений
```Kusto
SecurityEvent
| take 100
| project Activity 
| extend activityArr=split(Activity, " - ") 
| project Activity , activityArr, activityId=activityArr[0]
```

### <a name="explicit-credentials-processes"></a>Процессы с явными учетными данными
Следующий пример отображает круговую диаграмму с данными о процессах за прошлую неделю, которые использовали явные учетные данные

```Kusto
SecurityEvent
| where TimeGenerated > ago(7d)
// filter by id 4648 ("A logon was attempted using explicit credentials")
| where EventID == 4648
| summarize count() by Process
| render piechart 
```

### <a name="top-running-processes"></a>Наиболее активные процессы

Следующий пример отображает временную диаграмму активности для пяти наиболее активных процессов за последние три дня.

```Kusto
// Find all processes that started in the last three days. ID 4688: A new process has been created.
let RunProcesses = 
    SecurityEvent
    | where TimeGenerated > ago(3d)
    | where EventID == "4688";
// Find the 5 processes that were run the most
let Top5Processes =
RunProcesses
| summarize count() by Process
| top 5 by count_;
// Create a time chart of these 5 processes - hour by hour
RunProcesses 
| where Process in (Top5Processes) 
| summarize count() by bin (TimeGenerated, 1h), Process
| render timechart
```


### <a name="find-repeating-failed-login-attempts-by-the-same-account-from-different-ips"></a>Поиск повторяющихся неудачных попыток входа с одной учетной записью с разных IP-адресов

Следующий пример выявляет неудачные попытки входа с одной учетной записью, которые выполнялись с более чем пяти разных IP-адресов в течение последние шести часов.

```Kusto
SecurityEvent 
| where AccountType == "User" and EventID == 4625 and TimeGenerated > ago(6h) 
| summarize IPCount = dcount(IpAddress), makeset(IpAddress)  by Account
| where IPCount > 5
| sort by IPCount desc
```

### <a name="find-user-accounts-that-failed-to-log-in"></a>Поиск учетных записей пользователей, которым не удается выполнить вход 
Следующий пример выявляет учетные записи пользователей, которым не удалось войти в систему более пяти раз за последний день, с указанием времени последней попытки входа.

```Kusto
let timeframe = 1d;
SecurityEvent
| where TimeGenerated > ago(1d)
| where AccountType == 'User' and EventID == 4625 // 4625 - failed log in
| summarize failed_login_attempts=count(), latest_failed_login=arg_max(TimeGenerated, Account) by Account 
| where failed_login_attempts > 5
| project-away Account1
```

С помощью инструкций **join** и **let** мы проверяем, удалось ли этим подозрительным учетным записям позднее успешно войти в систему.

```Kusto
let timeframe = 1d;
let suspicious_users = 
    SecurityEvent
    | where TimeGenerated > ago(timeframe)
    | where AccountType == 'User' and EventID == 4625 // 4625 - failed login
    | summarize failed_login_attempts=count(), latest_failed_login=arg_max(TimeGenerated, Account) by Account 
    | where failed_login_attempts > 5
    | project-away Account1;
let suspicious_users_that_later_logged_in = 
    suspicious_users 
    | join kind=innerunique (
        SecurityEvent
        | where TimeGenerated > ago(timeframe)
        | where AccountType == 'User' and EventID == 4624 // 4624 - successful login,
        | summarize latest_successful_login=arg_max(TimeGenerated, Account) by Account
    ) on Account
    | extend was_login_after_failures = iif(latest_successful_login>latest_failed_login, 1, 0)
    | where was_login_after_failures == 1
;
suspicious_users_that_later_logged_in
```

## <a name="usage"></a>Использование

`Usage`Тип данных можно использовать для отслеживания принимаемого объема данных по решению или типу данных. Существуют и другие способы изучения полученных томов данных по [компьютерам](./manage-cost-storage.md#data-volume-by-computer) или [подпискам Azure, группам ресурсов или ресурсам](./manage-cost-storage.md#data-volume-by-azure-resource-resource-group-or-subscription).

#### <a name="data-volume-by-solution"></a>объем данных для каждого решения;

Запрос, используемый для просмотра объема оплачиваемых данных по решению за последний месяц (за исключением части последнего дня):

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by bin(StartTime, 1d), Solution | render barchart
```

Обратите внимание, что предложение `where IsBillable = true` отсеивает типы данных из определенных решений, для которых не взимается плата за прием данных.  Кроме того, предложение с `TimeGenerated` имеет значение только для того, чтобы убедиться в том, что запрос в портал Azure будет выглядеть после истечения 24 часов по умолчанию. При использовании типа данных об использовании `StartTime` и `EndTime` представляют временные периоды, для которых выводятся результаты. 

#### <a name="data-volume-by-type"></a>Объем данных по типу

Вы можете изучить данные еще подробнее, чтобы найти тенденции для определенных типов данных, выполнив следующий запрос:

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by bin(StartTime, 1d), DataType | render barchart
```

Или чтобы просмотреть таблицу по решению и типу за прошлый месяц, выполните следующий запрос:

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by Solution, DataType
| sort by Solution asc, DataType asc
```

> [!NOTE]
> Некоторые поля с типом данных Usage (Потребление) уже устарели и данные в них не заполняются, хотя они пока сохраняются в схеме. Например, сюда относятся поля **Computer** и ряд данных о приеме данных (**TotalBatches**, **BatchesWithinSla**, **BatchesOutsideSla**, **BatchesCapped** и **AverageProcessingTimeMs**).

## <a name="updates"></a>Обновления

### <a name="computers-still-missing-updates"></a>Компьютеры, на которых нет обновлений
Этот пример отображает список компьютеров, на которых несколько дней назад отсутствовали одно или несколько критических обновлений, и эти обновления по-прежнему отсутствуют.

```Kusto
let ComputersMissingUpdates3DaysAgo = Update
| where TimeGenerated between (ago(30d)..ago(1h))
| where Classification !has "Critical" and UpdateState =~ "Needed"
| summarize makeset(Computer);
Update
| where TimeGenerated > ago(1d)
| where Classification has "Critical" and UpdateState =~ "Needed"
| where Computer in (ComputersMissingUpdates3DaysAgo)
| summarize UniqueUpdatesCount = dcount(Product) by Computer, OSType
```


## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о языке можно получить в [справочнике по языку Kusto](/azure/kusto/query).
- Ознакомьтесь со статьей [Начало работы с запросами журнала Azure Monitor](./get-started-queries.md).