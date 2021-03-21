---
title: Оптимизация запросов предупреждений журнала | Документация Майкрософт
description: Рекомендации по написанию эффективных запросов оповещений
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: a7d65d7c65dabde3834458a36b50216878f7cf8d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102031299"
---
# <a name="optimizing-log-alert-queries"></a>Оптимизация запросов предупреждений журнала
В этой статье описывается написание и преобразование запросов [предупреждений журнала](./alerts-unified-log.md) для достижения оптимальной производительности. Оптимизированные запросы уменьшают задержку и нагрузку на предупреждения, которые выполняются часто.

## <a name="how-to-start-writing-an-alert-log-query"></a>Начало записи запроса в журнал предупреждений

Запросы предупреждений начинаются с [запроса данных журнала в log Analytics](alerts-log.md#create-a-log-alert-rule-with-the-azure-portal) , указывающих на ошибку. Вы можете ознакомиться с [разделом примеры запросов предупреждений](../logs/example-queries.md) , чтобы понять, что можно обнаружить. Вы также можете приступить [к написанию собственного запроса](../logs/log-analytics-tutorial.md). 

### <a name="queries-that-indicate-the-issue-and-not-the-alert"></a>Запросы, указывающие на ошибку, а не на предупреждение

Поток предупреждений был создан для преобразования результатов, указывающих на ошибку в предупреждении. Например, в случае запроса:

``` Kusto
SecurityEvent
| where EventID == 4624
```

Если предполагается, что пользователь получит предупреждение, то при возникновении этого события логика предупреждений добавляется `count` к запросу. Будет выполнен запрос:

``` Kusto
SecurityEvent
| where EventID == 4624
| count
```

Нет необходимости добавлять в запрос логику предупреждений, и это может вызвать проблемы. В приведенном выше примере, если включить `count` в запрос, всегда будет получено значение 1, так как служба оповещений будет делать `count` `count` .

### <a name="avoid-limit-and-take-operators"></a>Избегайте `limit` `take` операторов and

Использование `limit` и `take` в запросах может увеличить задержку и нагрузку на предупреждения, так как результаты не согласуются со временем. Рекомендуется использовать его только при необходимости.

## <a name="log-query-constraints"></a>Регистрация ограничений запросов
[Запросы к журналу в Azure Monitor](../logs/log-query-overview.md) начинаются с оператора таблицы, [`search`](/azure/kusto/query/searchoperator) или [`union`](/azure/kusto/query/unionoperator) .

Запросы для правил генерации оповещений журнала должны всегда начинаться с таблицы для определения области видимости, что повышает производительность запросов и релевантность результатов. Запросы в правилах генерации оповещений выполняются часто, поэтому использование `search` и `union` может привести к чрезмерному увеличению нагрузки на предупреждение, так как оно требует сканирования в нескольких таблицах. Эти операторы также снижают возможность службы предупреждений оптимизировать запрос.

Мы не поддерживаем создание или изменение правил генерации оповещений журнала, использующих `search` `union` операторы или, в ожидании запросов между ресурсами.

Например, следующий запрос оповещения ограничивается таблицей _SecurityEvent_ и выполняет поиск конкретного идентификатора события. Это единственная таблица, которую должен обработать запрос.

``` Kusto
SecurityEvent
| where EventID == 4624
```

Это изменение не затрагивает правила генерации оповещений журнала с помощью [межресурсных запросов](../logs/cross-workspace-query.md) , так как запросы между ресурсами используют тип `union` , который ограничивает область запроса конкретными ресурсами. В следующем примере будет использоваться допустимый запрос оповещения журнала:

```Kusto
union
app('Contoso-app1').requests,
app('Contoso-app2').requests,
workspace('Contoso-workspace1').Perf 
```

>[!NOTE]
> [Запросы между ресурсами](../logs/cross-workspace-query.md) поддерживаются в новом [API счедуледкуерирулес](/rest/api/monitor/scheduledqueryrules). Если вы по-прежнему используете [устаревший API предупреждений log Analytics](./api-alerts.md) для создания оповещений журнала, см. сведения о переключении [здесь](../alerts/alerts-log-api-switch.md).

## <a name="examples"></a>Примеры
Следующие примеры включают в себя запросы журналов, в которых используются `search` и `union` , а также шаги, с помощью которых можно изменить эти запросы для использования в правилах генерации оповещений.

### <a name="example-1"></a>Пример 1
Вы хотите создать правило генерации оповещений журнала, используя следующий запрос, который получает сведения о производительности с помощью `search` : 

``` Kusto
search *
| where Type == 'Perf' and CounterName == '% Free Space'
| where CounterValue < 30
```

Чтобы изменить этот запрос, нужно определить таблицу, к которой относятся свойства. Это делает следующий запрос:

``` Kusto
search *
| where CounterName == '% Free Space'
| summarize by $table
```

В результате запроса будет показано, что свойство _CounterName_ получено из таблицы _Perf_.

Этот результат можно использовать для создания следующего запроса, который будет использоваться для правила оповещения:

``` Kusto
Perf
| where CounterName == '% Free Space'
| where CounterValue < 30
```

### <a name="example-2"></a>Пример 2
Вы хотите создать правило генерации оповещений журнала, используя следующий запрос, который получает сведения о производительности с помощью `search` : 

``` Kusto
search ObjectName =="Memory" and CounterName=="% Committed Bytes In Use"
| summarize Avg_Memory_Usage =avg(CounterValue) by Computer
| where Avg_Memory_Usage between(90 .. 95)  
```

Чтобы изменить этот запрос, нужно определить таблицу, к которой относятся свойства. Это делает следующий запрос:

``` Kusto
search ObjectName=="Memory" and CounterName=="% Committed Bytes In Use"
| summarize by $table
```

В результате запроса будет показано, что свойства _ObjectName_ и _CounterName_ получены из таблицы _Perf_.

Этот результат можно использовать для создания следующего запроса, который будет использоваться для правила оповещения:

``` Kusto
Perf
| where ObjectName =="Memory" and CounterName=="% Committed Bytes In Use"
| summarize Avg_Memory_Usage=avg(CounterValue) by Computer
| where Avg_Memory_Usage between(90 .. 95)
```

### <a name="example-3"></a>Пример 3

Вы хотите создать правило генерации оповещений журнала, используя следующий запрос, который использует `search` и `union` для получения сведений о производительности: 

``` Kusto
search (ObjectName == "Processor" and CounterName == "% Idle Time" and InstanceName == "_Total")
| where Computer !in (
    union *
    | where CounterName == "% Processor Utility"
    | summarize by Computer)
| summarize Avg_Idle_Time = avg(CounterValue) by Computer
```

Чтобы изменить этот запрос, нужно определить таблицу, к которой относятся свойства в первой части запроса: 

``` Kusto
search (ObjectName == "Processor" and CounterName == "% Idle Time" and InstanceName == "_Total")
| summarize by $table
```

В результате запроса будет показано, что все свойства получены из таблицы _Perf_.

Теперь используйте `union` с командой `withsource`, чтобы определить исходную таблицу, которая повлияла на каждую запись.

``` Kusto
union withsource=table *
| where CounterName == "% Processor Utility"
| summarize by table
```

В результате запроса будет показано, что эти свойства также получены из таблицы _Perf_.

Эти результаты можно использовать для создания следующего запроса, который будет использоваться для правила оповещения: 

``` Kusto
Perf
| where ObjectName == "Processor" and CounterName == "% Idle Time" and InstanceName == "_Total"
| where Computer !in (
    (Perf
    | where CounterName == "% Processor Utility"
    | summarize by Computer))
| summarize Avg_Idle_Time = avg(CounterValue) by Computer
``` 

### <a name="example-4"></a>Пример 4
Вы хотите создать правило генерации оповещений журнала, используя следующий запрос, который соединяет результаты двух `search` запросов:

```Kusto
search Type == 'SecurityEvent' and EventID == '4625'
| summarize by Computer, Hour = bin(TimeGenerated, 1h)
| join kind = leftouter (
    search in (Heartbeat) OSType == 'Windows'
    | summarize arg_max(TimeGenerated, Computer) by Computer , Hour = bin(TimeGenerated, 1h)
    | project Hour , Computer
) on Hour
```

Чтобы изменить этот запрос, нужно определить таблицу, которая содержит свойства в левой части объединения: 

``` Kusto
search Type == 'SecurityEvent' and EventID == '4625'
| summarize by $table
```

В результате показано, что свойства в левой части объединения принадлежат таблице _SecurityEvent_. 

Теперь используйте следующий запрос, чтобы определить таблицу, которая содержит свойства в правой части объединения: 
 
``` Kusto
search in (Heartbeat) OSType == 'Windows'
| summarize by $table
```
 
Результат указывает, что свойства в правой части соединения принадлежат таблице _пульса_ .

Эти результаты можно использовать для создания следующего запроса, который будет использоваться для правила оповещения: 

``` Kusto
SecurityEvent
| where EventID == '4625'
| summarize by Computer, Hour = bin(TimeGenerated, 1h)
| join kind = leftouter (
    Heartbeat
    | where OSType == 'Windows'
    | summarize arg_max(TimeGenerated, Computer) by Computer , Hour = bin(TimeGenerated, 1h)
    | project Hour , Computer
) on Hour
```

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения об [оповещениях журналов](alerts-log.md) в Azure Monitor.
- Дополнительные сведения о [запросах журналов](../logs/log-query-overview.md).