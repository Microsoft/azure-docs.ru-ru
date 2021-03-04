---
title: Использование REST API оповещений Log Analytics
description: REST API оповещение Log Analytics позволяет создавать оповещения и управлять ими в Log Analytics, который является частью Log Analytics.  В этой статье приводятся сведения об интерфейсе API и примеры выполнения различных операций.
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 33f5ce86a21a9aad110db162849ff3e3c7bbb2b4
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102037495"
---
# <a name="create-and-manage-alert-rules-in-log-analytics-with-rest-api"></a>Создание правил генерации оповещений и управление ими в Log Analytics с помощью REST API 

> [!IMPORTANT]
> Как [объявлено](https://azure.microsoft.com/updates/switch-api-preference-log-alerts/), рабочие области log Analytics, созданные после *1 июня, 2019,* управляют правилами генерации оповещений с помощью текущего [API счедуледкуерирулес](/rest/api/monitor/scheduledqueryrules/). Клиентам рекомендуется [переключиться на текущий API](./alerts-log-api-switch.md) в старых рабочих областях, чтобы использовать [преимущества](./alerts-log-api-switch.md#benefits)Azure Monitor счедуледкуерирулес. В этой статье описывается управление правилами генерации оповещений с помощью API прежних версий.

REST API оповещений Log Analytics позволяет создавать оповещения и управлять ими в Log Analytics.  В этой статье приводятся сведения об интерфейсе API и примеры выполнения различных операций.

Для поиска в службе Log Analytics используется REST API категории RESTful — к нему можно получить доступ с помощью REST API Azure Resource Manager. В этом документе вы найдете примеры, когда доступ к API из командной строки PowerShell осуществляется через [ARMClient](https://github.com/projectkudu/ARMClient) — программу командной строки с открытым кодом, которая упрощает вызов API Azure Resource Manager. Использование ARMClient и PowerShell — это один из множества вариантов получения доступа к API поиска по службе Log Analytics. С помощью этих инструментов можно использовать API Azure Resource Manager категории RESTful для осуществления вызовов рабочих областей Log Analytics и выполнения команд поиска в них. API будет выдавать результаты поиска в формате JSON, позволяя программно использовать результаты поиска различными способами.

## <a name="prerequisites"></a>Предварительные требования
В настоящее время оповещения могут создаваться только с помощью сохраненного поиска в службе Log Analytics.  Дополнительные сведения см. в статье [REST API поиска в журналах](../logs/log-query-overview.md).

## <a name="schedules"></a>Расписания
Сохраненный поиск может включать одно расписание или несколько. Расписание определяет, как часто выполняется поиск, а также интервал времени для определения условий.
У расписаний есть свойства, приведенные в таблице ниже.

| Свойство | Описание |
|:--- |:--- |
| Интервал |Насколько часто выполняется поиск. Измеряется в минутах. |
| QueryTimeSpan |Интервал времени для вычисления условий. Значение должно быть больше или равно значению свойства Interval. Измеряется в минутах. |
| Версия |Используемая версия API.  Сейчас это свойство всегда должно иметь значение 1. |

Например, рассмотрим запрос событий с интервалом (свойство Interval) 15 минут и временным диапазоном (свойство TimeSpan) 30 минут. В этом случае запрос будет выполняться каждые 15 минут, а оповещение будет создаваться, если условие по-прежнему соблюдается по прошествии 30-минутного периода.

### <a name="retrieving-schedules"></a>Получение расписаний
Используйте метод Get для получения всех расписаний для сохраненного поиска.

```powershell
armclient get /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search  ID}/schedules?api-version=2015-03-20
```

Чтобы получить конкретное расписание для сохраненного поискового запроса, используйте метод Get с идентификатором расписания.

```powershell
armclient get /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}?api-version=2015-03-20
```

Ниже приведен пример ответа для расписания.

```json
{
   "value": [{
      "id": "subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/sampleRG/providers/Microsoft.OperationalInsights/workspaces/MyWorkspace/savedSearches/0f0f4853-17f8-4ed1-9a03-8e888b0d16ec/schedules/a17b53ef-bd70-4ca4-9ead-83b00f2024a8",
      "etag": "W/\"datetime'2016-02-25T20%3A54%3A49.8074679Z'\"",
      "properties": {
         "Interval": 15,
         "QueryTimeSpan": 15,
         "Enabled": true,
      }
   }]
}
```

### <a name="creating-a-schedule"></a>Создание расписания
Чтобы создать расписание, используйте метод Put с уникальным идентификатором расписания.  Два расписания не могут иметь одинаковый идентификатор, даже если они связаны с разными сохраненными поисковыми запросами.  При создании расписания в консоли Log Analytics для идентификатора расписания создается GUID.

> [!NOTE]
> Имена всех сохраненных поисковых запросов, отчетов и действий, создаваемых с помощью API Log Analytics, должны быть в нижнем регистре.

```powershell
$scheduleJson = "{'properties': { 'Interval': 15, 'QueryTimeSpan':15, 'Enabled':'true' } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/mynewschedule?api-version=2015-03-20 $scheduleJson
```

### <a name="editing-a-schedule"></a>Изменение расписания
Чтобы изменить расписание, используйте метод Put с идентификатором имеющегося расписания для того же сохраненного поискового запроса (в примере ниже расписание отключено). Текст запроса должен включать *ETag* расписания.

```powershell
$scheduleJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A49.8074679Z'\""','properties': { 'Interval': 15, 'QueryTimeSpan':15, 'Enabled':'false' } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/mynewschedule?api-version=2015-03-20 $scheduleJson
```

### <a name="deleting-schedules"></a>Удаление расписаний
Чтобы удалить расписание, используйте метод Delete с идентификатором расписания.

```powershell
armclient delete /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}?api-version=2015-03-20
```

## <a name="actions"></a>Actions
Расписание может включать несколько действий. Действие может определять один выполняемый процесс или несколько (например, отправку почты или запуск модуля Runbook) или задавать пороговое значение, определяющее, когда результаты поиска отвечают некоторым условиям.  Некоторые действия будут определять оба элемента так, чтобы процессы выполнялись при достижении порогового значения.

У всех действий есть свойства, приведенные в таблице ниже.  Разные типы оповещений имеют различные дополнительные свойства, которые описаны ниже.

| Свойство | Описание |
|:--- |:--- |
| `Type` |Тип действия.  Сейчас возможными значениями являются Alert и Webhook. |
| `Name` |Отображаемое имя оповещения. |
| `Version` |Используемая версия API.  Сейчас это свойство всегда должно иметь значение 1. |

### <a name="retrieving-actions"></a>Получение действий

Используйте метод Get для получения всех действий для расписания.

```powershell
armclient get /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search  ID}/schedules/{Schedule ID}/actions?api-version=2015-03-20
```

Используйте метод Get с идентификатором действия для получения конкретного действия для расписания.

```powershell
armclient get /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}/actions/{Action ID}?api-version=2015-03-20
```

### <a name="creating-or-editing-actions"></a>Создание или изменение действий
Чтобы создать действие, используйте метод Put с идентификатором действия, уникальным для расписания.  При создании действия в консоли Log Analytics для идентификатора действия отображается GUID.

> [!NOTE]
> Имена всех сохраненных поисковых запросов, отчетов и действий, создаваемых с помощью API Log Analytics, должны быть в нижнем регистре.

Чтобы изменить расписание, используйте метод Put с идентификатором существующего действия для того же сохраненного поискового запроса.  Текст запроса должен содержать Etag расписания.

Формат запроса для создания действия зависит от типа действия, поэтому эти примеры приведены в разделах ниже.

### <a name="deleting-actions"></a>Удаление действий

Для удаления действия используйте метод Delete с идентификатором действия.

```powershell
armclient delete /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}/Actions/{Action ID}?api-version=2015-03-20
```

### <a name="alert-actions"></a>Действия оповещений
Расписание должно включать одно-единственное действие оповещения.  Действия оповещений включают один или несколько разделов в таблице ниже.  Они более подробно описаны далее.

| Section | Описание | Использование |
|:--- |:--- |:--- |
| Порог |Условие для запуска действия.| Требуется для каждого оповещения до или после его отображения в Azure. |
| Severity |Метка, используемая для классификации оповещения при его активации.| Требуется для каждого оповещения до или после его отображения в Azure. |
| Подавление |Параметр, который прекращает отправку уведомлений из оповещения. | Требуется для каждого оповещения до или после его отображения в Azure. |
| Группы действий |Идентификаторы группы ActionGroup Azure, в которой указаны необходимые действия, например сообщения электронной почты, текстовые сообщения, голосовые вызовы, веб-перехватчики, модули runbook службы автоматизации, соединители ITSM и т. д.| Требуется после переноса оповещений в Azure.|
| Настройка действий|Изменение стандартного потока вывода для выбора действий из ActionGroup.| Необязательно для каждого оповещения, может использоваться после переноса оповещений в Azure. |

### <a name="thresholds"></a>Пороговые значения
Действие оповещения должно включать одно-единственное пороговое значение.  Когда результат сохраненного поиска соответствует пороговому значению в действии, связанном с этим поиском, запускаются другие процессы в этом действии.  Действие также может содержать только пороговое значение, чтобы его можно было использовать с действиями других типов, которые не содержат пороговых значений.

У пороговых значений есть свойства, приведенные в таблице ниже.

| Свойство | Описание |
|:--- |:--- |
| `Operator` |Оператор для сравнения порогового значения. <br> gt — больше <br>  lt = меньше чем |
| `Value` |Пороговое значение. |

Например, рассмотрим запрос событий с интервалом (свойство Interval) 15 минут, временным диапазоном (свойство TimeSpan) 30 минут и пороговым значением больше 10. В этом случае запрос будет выполняться каждые 15 минут, а оповещение будет создаваться при возврате 10 событий, созданных за 30-минутный период.

Ниже приведен пример ответа для действия, содержащего только пороговое значение.  

```json
"etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "My threshold action",
   "Threshold": {
      "Operator": "gt",
      "Value": 10
   },
   "Version": 1
}
```

Чтобы создать действие с пороговым значением для расписания, используйте метод Put с уникальным идентификатором действия.  

```powershell
$thresholdJson = "{'properties': { 'Name': 'My Threshold', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdJson
```

Чтобы изменить действие с пороговым значением для расписания, используйте метод Put с идентификатором существующего действия.  Текст запроса должен содержать Etag действия.

```powershell
$thresholdJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'My Threshold', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdJson
```

#### <a name="severity"></a>Severity
Log Analytics позволяет классифицировать оповещения по категориям, чтобы упростить управление и рассмотрение. По уровню серьезности оповещение может быть информационным, предупреждающим и критическим. Эти уровни соответствуют нормализованной шкале серьезности оповещений Azure следующим образом.

|Уровень серьезности Log Analytics  |Уровень серьезности оповещений Azure  |
|---------|---------|
|`critical` |Серьезность 0|
|`warning` |Серьезность 1|
|`informational` | Серьезность 2|

Ниже приведен пример ответа для действия, содержащего только пороговое значение и серьезность. 

```json
"etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "My threshold action",
   "Threshold": {
      "Operator": "gt",
      "Value": 10
   },
   "Severity": "critical",
   "Version": 1
}
```

Чтобы создать действие для расписания с указанием серьезности, используйте метод Put с уникальным идентификатором действия.  

```powershell
$thresholdWithSevJson = "{'properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdWithSevJson
```

Чтобы изменить действие серьезности для расписания, используйте метод Put с идентификатором существующего действия.  Текст запроса должен содержать Etag действия.

```powershell
$thresholdWithSevJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdWithSevJson
```

#### <a name="suppress"></a>Подавление
Предупреждения о запросах на основе Log Analytics будут срабатывать каждый раз, когда пороговое значение будет достигнуто или превышено. Основываясь на логике, подразумеваемой в запросе, это может привести к тому, что оповещение будет запущено в течение нескольких интервалов, и, следовательно, будут постоянно отправляться уведомления. Чтобы предотвратить такой сценарий, пользователь может установить опцию "Подавление" в Log Analytics, чтобы некоторое время ожидать получение уведомления во второй раз с целью предупреждения. Поэтому, если подавление установлено на 30 минут, то оповещение будет срабатывать в первый раз и отправлять настроенные уведомления. Прежде чем снова использовать уведомления для правила генерации оповещений, подождите 30 минут. В промежуточный период правило предупреждения будет продолжать выполняться. Уведомление подавляется Log Analytics за указанное время независимо от того, сколько раз срабатывало правило предупреждения в этот период.

Свойства подавления правила генерации оповещений в Log Analytics определены с помощью значения *Throttling* и подавления периода, используя значение *DurationInMinutes*.

Ниже приведен пример ответа для действия, содержащего только пороговое значение, серьезность и свойство подавления.

```json
"etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "My threshold action",
   "Threshold": {
      "Operator": "gt",
      "Value": 10
   },
   "Throttling": {
   "DurationInMinutes": 30
   },
   "Severity": "critical",
   "Version": 1
}
```

Чтобы создать действие для расписания с указанием серьезности, используйте метод Put с уникальным идентификатором действия.  

```powershell
$AlertSuppressJson = "{'properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Throttling': { 'DurationInMinutes': 30 },'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myalert?api-version=2015-03-20 $AlertSuppressJson
```

Чтобы изменить действие серьезности для расписания, используйте метод Put с идентификатором существующего действия.  Текст запроса должен содержать Etag действия.

```powershell
$AlertSuppressJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Throttling': { 'DurationInMinutes': 30 },'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myalert?api-version=2015-03-20 $AlertSuppressJson
```

#### <a name="action-groups"></a>Группы действий
Все оповещения в Azure используют группу действий в качестве механизма обработки действий по умолчанию. С помощью группы действий можно указать действия один раз, а затем связать группу действий для несколькими оповещениями в Azure. Это избавляет от необходимости повторно объявить одни и те же действия снова и снова. Группы действий поддерживает несколько действий, включая сообщения электронной почты, текстовые сообщения, голосовые вызовы, подключения ITSM, runbook службы автоматизации, универсальный код ресурса (URI) веб-перехватчика и многое другое. 

Теперь для пользователей, перенесших оповещения в Azure, расписание должно передавать сведения о группе действий вместе с пороговым значением, чтобы обеспечить создание оповещения. Сведения о сообщениях электронной почты, URL-адреса веб-перехватчиков, данные runbook службы автоматизации и другие действия должны быть определены в группе действий перед созданием оповещения. Можно создать [группу действий в Azure Monitor](./action-groups.md) на портале или использовать [API группы действий](/rest/api/monitor/actiongroups).

Чтобы добавить связь группы действий с оповещением, укажите уникальный идентификатор Azure Resource Manager группы действий в определении оповещения. Ниже приведен наглядный пример.

```json
"etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "test-alert",
   "Description": "I need to put a description here",
   "Threshold": {
      "Operator": "gt",
      "Value": 12
   },
   "AzNsNotification": {
      "GroupIds": [
         "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
      ]
   },
   "Severity": "critical",
   "Version": 1
}
```

Используйте метод Put с уникальным идентификатором действия, чтобы привязать уже существующую группу действий к расписанию.  Ниже приведен пример использования.

```powershell
$AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']} } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

Чтобы изменить группу действий, связанную с расписанием, используйте метод Put с идентификатором существующего действия.  Текст запроса должен содержать Etag действия.

```powershell
$AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', 'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': { 'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup'] } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

#### <a name="customize-actions"></a>Настройка действий
По умолчанию действия используют стандартные шаблон и формат для уведомлений. Однако пользователь может настроить некоторые действия, даже если ими управляют группы действий. В настоящее время можно настроить тему сообщения электронной почты и полезные данные веб-перехватчика.

##### <a name="customize-e-mail-subject-for-action-group"></a>Настройка темы сообщения электронной почты для группы действий
По умолчанию в качестве темы для оповещений используется сообщение: уведомление о предупреждении `<AlertName>` `<WorkspaceName>` . Однако ее можно настроить, указав определенные слова или теги, чтобы вы могли легко применять правила фильтрации в папке "Входящие". Сведения о пользовательском заголовке электронного сообщения необходимо отправлять вместе с данными ActionGroup, как показано в следующем примере.

```json
"etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "test-alert",
   "Description": "I need to put a description here",
   "Threshold": {
      "Operator": "gt",
      "Value": 12
   },
   "AzNsNotification": {
      "GroupIds": [
         "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
      ],
      "CustomEmailSubject": "Azure Alert fired"
   },
   "Severity": "critical",
   "Version": 1
}
```

Используйте метод Put с уникальным идентификатором действия, чтобы привязать уже существующую группу действий с измененными параметрами к расписанию.  Ниже приведен пример использования.

```powershell
$AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup'], 'CustomEmailSubject': 'Azure Alert fired'} } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

Чтобы изменить группу действий, связанную с расписанием, используйте метод Put с идентификатором существующего действия.  Текст запроса должен содержать Etag действия.

```powershell
$AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', 'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']}, 'CustomEmailSubject': 'Azure Alert fired' } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

##### <a name="customize-webhook-payload-for-action-group"></a>Настройка полезных данных веб-перехватчика для группы действий
По умолчанию веб-перехватчик, отправляемый группой действий в Log Analytics, имеет фиксированную структуру. Но с помощью специальных поддерживаемых переменных можно настроить полезные данные JSON, чтобы соблюсти требования конечной точки веб-перехватчика. Дополнительные сведения см. в статье [Действия веб-перехватчика для правил оповещений журнала](./alerts-log-webhook.md). 

Сведения о пользовательском веб-перехватчике необходимо отправлять вместе с данными ActionGroup. Они будут применены для всех универсальных кодов ресурса (URI) веб-перехватчиков, указанных в группе действий, как показано в следующем примере.

```json
"etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "test-alert",
   "Description": "I need to put a description here",
   "Threshold": {
      "Operator": "gt",
      "Value": 12
   },
   "AzNsNotification": {
      "GroupIds": [
         "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
      ],
   "CustomWebhookPayload": "{\"field1\":\"value1\",\"field2\":\"value2\"}",
   "CustomEmailSubject": "Azure Alert fired"
   },
   "Severity": "critical",
   "Version": 1
},
```

Используйте метод Put с уникальным идентификатором действия, чтобы привязать уже существующую группу действий с измененными параметрами к расписанию.  Ниже приведен пример использования.

```powershell
$AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup'], 'CustomEmailSubject': 'Azure Alert fired','CustomWebhookPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}'} } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

Чтобы изменить группу действий, связанную с расписанием, используйте метод Put с идентификатором существующего действия.  Текст запроса должен содержать Etag действия.

```powershell
$AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', 'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']}, 'CustomEmailSubject': 'Azure Alert fired','CustomWebhookPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}' } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

## <a name="next-steps"></a>Дальнейшие действия

* Используйте [REST API для выполнения поиска в журналах](../logs/log-query-overview.md) в службе Log Analytics.
* Сведения об [оповещениях журнала в Azure Monitor](./alerts-unified-log.md)
* [Создание, изменение и управление правилами генерации оповещений в журнале в Azure Monitor](./alerts-log.md)