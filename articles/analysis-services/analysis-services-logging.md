---
title: Ведение журнала диагностики для Azure Analysis Services | Документация Майкрософт
description: Узнайте, как настроить ведение журнала для мониторинга сервера Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 2bee856adef1208aabbe65ecd5fd11235579bb82
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100582700"
---
# <a name="setup-diagnostic-logging"></a>Настройка журнала ведения диагностики

Важной частью любого решения Analysis Services является мониторинг работы серверов. Службы Azure Analysis Services интегрированы с Azure Monitor. С помощью [журналов ресурсов Azure Monitor](../azure-monitor/essentials/platform-logs-overview.md) вы можете отслеживать и отправлять журналы в [службу хранилища Azure](https://azure.microsoft.com/services/storage/), выполнять их потоковую передачу в [Центры событий Azure](https://azure.microsoft.com/services/event-hubs/) и экспортировать их в [журналы Azure Monitor](../azure-monitor/overview.md).

![Ведение журнала ресурсов в хранилище, Центрах событий и журналах Azure Monitor](./media/analysis-services-logging/aas-logging-overview.png)

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="whats-logged"></a>Какие данные регистрируются?

Можно выбрать категории **Подсистема**, **Служба** и **Метрики**.

### <a name="engine"></a>Подсистема

При выборе категории **Подсистема** в журнале регистрируется все события [xEvent](/analysis-services/instances/monitor-analysis-services-with-sql-server-extended-events). Невозможно будет выбрать отдельные события. 

|Категории событий xEvent |Имя события  |
|---------|---------|
|Аудит безопасности    |   Аудит входа в систему      |
|Аудит безопасности    |   Audit Logout      |
|Аудит безопасности    |   Audit Server Starts And Stops      |
|Отчеты о ходе выполнения     |   Progress Report Begin      |
|Отчеты о ходе выполнения     |   Progress Report End      |
|Отчеты о ходе выполнения     |   Progress Report Current      |
|Запросы     |  Query Begin       |
|Запросы     |   Query End      |
|Команды     |  Command Begin       |
|Команды     |  Command End       |
|Ошибки и предупреждения     |   Error      |
|Поиск     |   Discover End      |
|Уведомление     |    Уведомление     |
|Сеанс     |  Session Initialize       |
|Блокировки    |  Deadlock       |
|Обработка запросов     |   VertiPaq SE Query Begin      |
|Обработка запросов     |   VertiPaq SE Query End      |
|Обработка запросов     |   VertiPaq SE Query Cache Match      |
|Обработка запросов     |   Direct Query Begin      |
|Обработка запросов     |  Direct Query End       |

### <a name="service"></a>Служба

|Имя операции  |Когда выполняется  |
|---------|---------|
|ResumeServer     |    Возобновление работы сервера.     |
|SuspendServer    |   Приостановка работы сервера.      |
|DeleteServer     |    Удаление сервера     |
|RestartServer    |     Пользователь перезапускает сервер с помощью SSMS или PowerShell.    |
|GetServerLogFiles    |    Пользователь экспортирует журнал сервера с помощью PowerShell.     |
|ExportModel     |   Пользователь экспортирует модель на портал с помощью функции открытия в Visual Studio.     |

### <a name="all-metrics"></a>Все метрики

Категория "Метрики" позволяет регистрировать [метрики сервера](analysis-services-monitor.md#server-metrics) в таблице AzureMetrics. Если вы используете [горизонтальное увеличение масштаба](analysis-services-scale-out.md) запросов и вам нужно разделить метрики для каждой реплики чтения, используйте вместо этого таблицу AzureDiagnostics, где значение **OperationName** равно **LogMetric**.

## <a name="setup-diagnostics-logging"></a>Настройка ведения журнала диагностики

### <a name="azure-portal"></a>Портал Azure

1. На [портале Azure](https://portal.azure.com) выберите сервер, в области навигации слева выберите **Журналы диагностики**, а затем — **Включить диагностику**.

    ![Включение журнала ресурсов для Azure Cosmos DB на портале Azure](./media/analysis-services-logging/aas-logging-turn-on-diagnostics.png)

2. На странице **Параметры диагностики** настройте следующие параметры: 

    * **Имя**. Введите имя для создаваемых журналов.

    * **Archive to a storage account** (Архивировать в учетной записи хранения). Чтобы использовать этот параметр, необходима учетная запись хранения для подключения. Ознакомьтесь со статьей [Создание учетной записи хранения](../storage/common/storage-account-create.md). Следуйте указаниям для создания диспетчера ресурсов и учетной записи общего назначения, а затем выберите учетную запись хранения, вернувшись к этой странице портала. Возможно, потребуется подождать несколько минут, пока созданная учетная запись хранения отобразится в раскрывающемся меню.
    * **Stream to an event hub** (Потоковая передача в концентратор событий). Чтобы использовать этот параметр, вам понадобится пространство имен концентратора событий и концентратор событий для подключения. Дополнительные сведения см. в статье [Создание пространства имен Центров событий и концентратора событий с помощью портала Azure](../event-hubs/event-hubs-create.md). Затем на портале вернитесь на эту страницу, чтобы выбрать пространство имен концентратора событий и имя политики.
    * **Отправка в Azure Monitor (рабочую область Log Analytics)** . Чтобы использовать этот параметр, воспользуйтесь одной из имеющихся рабочих областей или [создайте новый ресурс рабочей области](../azure-monitor/logs/quick-create-workspace.md) на портале. См. дополнительные сведения о [просмотре журналов в рабочей области Log Analytics](#view-logs-in-log-analytics-workspace).

    * **Подсистема**. Выберите этот параметр для ведения журнала xEvents. Если выполняется архивация в учетную запись хранения, можно выбрать период хранения журналов ресурсов. По окончании периода хранения журналы удаляются автоматически.
    * **Служба**. Выберите этот параметр, чтоб вести журнал событий уровня службы. Если выполняется архивация в учетную запись хранения, можно выбрать период хранения журналов ресурсов. По окончании периода хранения журналы удаляются автоматически.
    * **Метрики**. Выберите этот параметр, чтобы хранить подробные данные в разделе [Метрики](analysis-services-monitor.md#server-metrics). Если выполняется архивация в учетную запись хранения, можно выбрать период хранения журналов ресурсов. По окончании периода хранения журналы удаляются автоматически.

3. Выберите команду **Сохранить**.

    Если появится сообщение об ошибке "не удалось обновить диагностику для \<workspace name> . Подписка \<subscription id> не зарегистрирована для использования Microsoft. Insights. " следуйте инструкциям статьи [Устранение неполадок Диагностики Azure](../azure-monitor/essentials/resource-logs.md), чтобы зарегистрировать учетную запись, а затем повторите процедуру.

    Если вы хотите изменить способ сохранения журналов ресурсов в дальнейшем, можно в любое время вернуться на эту страницу, чтобы изменить параметры.

### <a name="powershell"></a>PowerShell

Ниже приведены основные команды для ознакомления. Если вам требуются пошаговые указания по настройке ведения журнала в учетной записи хранения с помощью PowerShell, ознакомьтесь с руководством далее в этой статье.

Чтобы включить ведение журналов метрик и ресурсов с помощью PowerShell, используйте следующие команды.

- Выполните следующую команду, чтобы включить хранение журналов ресурсов в учетной записи хранения.

   ```powershell
   Set-AzDiagnosticSetting -ResourceId [your resource id] -StorageAccountId [your storage account id] -Enabled $true
   ```

   StorageAccountId — это идентификатор ресурса учетной записи хранения, в которую будут отправляться журналы.

- Чтобы включить потоковую передачу журналов ресурсов в концентратор событий, используйте следующую команду.

   ```powershell
   Set-AzDiagnosticSetting -ResourceId [your resource id] -ServiceBusRuleId [your service bus rule id] -Enabled $true
   ```

   Идентификатор правила служебной шины Azure — это строка в формате:

   ```powershell
   {service bus resource ID}/authorizationrules/{key name}
   ``` 

- Чтобы включить отправку журналов ресурсов в рабочую область Log Analytics, используйте следующую команду.

   ```powershell
   Set-AzDiagnosticSetting -ResourceId [your resource id] -WorkspaceId [resource id of the log analytics workspace] -Enabled $true
   ```

- Идентификатор ресурса рабочей области Log Analytics можно получить с помощью следующей команды:

   ```powershell
   (Get-AzOperationalInsightsWorkspace).ResourceId
   ```

Можно объединять эти параметры, чтобы получить несколько вариантов вывода.

### <a name="rest-api"></a>REST API

Узнайте, как [изменить параметры диагностики с помощью REST API Azure Monitor](/rest/api/monitor/). 

### <a name="resource-manager-template"></a>Шаблон Resource Manager

Узнайте, как [включить параметры диагностики при создании ресурса из шаблона Resource Manager](../azure-monitor/essentials/resource-manager-diagnostic-settings.md). 

## <a name="manage-your-logs"></a>Управление журналами

Журналы обычно доступны через пару часов после настройки ведения журнала. Способ управления журналами в своей учетной записи хранения вы выбираете сами.

* Используйте стандартные методы управления доступом, предоставляемые Azure, для защиты журналов путем ограничения доступа к ним.
* Удаляйте журналы, которые больше не нужно хранить в учетной записи хранения.
* Не забудьте установить срок хранения, чтобы устаревшие журналы удалялись из вашей учетной записи хранения.

## <a name="view-logs-in-log-analytics-workspace"></a>Просмотр журналов в рабочей области Log Analytics

Метрики и события сервера объединяются с событиями xEvent в ресурсе рабочей области Log Analytics для параллельного анализа. Рабочую область Log Analytics можно также настроить для получения событий из других служб Azure, обеспечив целостное представление о данных журнала ведения диагностики в своей архитектуре.

Чтобы просмотреть данные диагностики, в рабочей области Log Analytics откройте **Журналы** в меню слева.

![Параметры поиска по журналам на портале Azure](./media/analysis-services-logging/aas-logging-open-log-search.png)

В конструкторе запросов разверните **LogManagement** > **AzureDiagnostics**. AzureDiagnostics включает в себя события "Подсистема" и "Служба". Обратите внимание на то, что запрос создается в режиме реального времени. Поле "EventClass\_s" содержит имена событий xEvent, которые могут быть вам знакомы, если вы использовали события xEvent для локального входа. Щелкните поле **EventClass\_s** или одно из имен событий, и рабочая область Log Analytics продолжит создание запроса. Не забудьте сохранить запросы для последующего повторного использования.

### <a name="example-queries"></a>Примеры запросов

#### <a name="example-1"></a>Пример 1

Этот запрос возвращает значение длительности каждого события окончания или обновления запроса для шаблона базы данных и сервера. При горизонтальном масштабировании результаты разбиваются по реплике, так как номер реплики включается в ServerName_s. Группирование по RootActivityId_g сокращает количество строк, извлекаемых из REST API Диагностики Azure и обеспечивает соблюдение ограничений, как описано в статье [Пределы скорости Log Analytics](https://dev.loganalytics.io/documentation/Using-the-API/Limits).

```Kusto
let window = AzureDiagnostics
   | where ResourceProvider == "MICROSOFT.ANALYSISSERVICES" and Resource =~ "MyServerName" and DatabaseName_s =~ "MyDatabaseName" ;
window
| where OperationName has "QueryEnd" or (OperationName has "CommandEnd" and EventSubclass_s == 38)
| where extract(@"([^,]*)", 1,Duration_s, typeof(long)) > 0
| extend DurationMs=extract(@"([^,]*)", 1,Duration_s, typeof(long))
| project  StartTime_t,EndTime_t,ServerName_s,OperationName,RootActivityId_g,TextData_s,DatabaseName_s,ApplicationName_s,Duration_s,EffectiveUsername_s,User_s,EventSubclass_s,DurationMs
| order by StartTime_t asc
```

#### <a name="example-2"></a>Пример 2

Следующий запрос возвращает сведения о потреблении памяти и QPU для сервера. При горизонтальном масштабировании результаты разбиваются по реплике, так как номер реплики включается в ServerName_s.

```Kusto
let window = AzureDiagnostics
   | where ResourceProvider == "MICROSOFT.ANALYSISSERVICES" and Resource =~ "MyServerName";
window
| where OperationName == "LogMetric" 
| where name_s == "memory_metric" or name_s == "qpu_metric"
| project ServerName_s, TimeGenerated, name_s, value_s
| summarize avg(todecimal(value_s)) by ServerName_s, name_s, bin(TimeGenerated, 1m)
| order by TimeGenerated asc 
```

#### <a name="example-3"></a>Пример 3

Следующий запрос возвращает значения счетчиков производительности "Считано строк/с" обработчика Analysis Services для сервера.

```Kusto
let window =  AzureDiagnostics
   | where ResourceProvider == "MICROSOFT.ANALYSISSERVICES" and Resource =~ "MyServerName";
window
| where OperationName == "LogMetric" 
| where parse_json(tostring(parse_json(perfobject_s).counters))[0].name == "Rows read/sec" 
| extend Value = tostring(parse_json(tostring(parse_json(perfobject_s).counters))[0].value) 
| project ServerName_s, TimeGenerated, Value
| summarize avg(todecimal(Value)) by ServerName_s, bin(TimeGenerated, 1m)
| order by TimeGenerated asc 
```

Можно использовать сотни запросов. Дополнительные сведения о запросах см. в статье [Начало работы с запросами журналов Azure Monitor](../azure-monitor/logs/get-started-queries.md).


## <a name="turn-on-logging-by-using-powershell"></a>Включение ведения журнала с помощью PowerShell

В этом кратком руководстве вы создаете учетную запись хранения в тех же подписке и группе ресурсов, что и сервер Analysis Services. Затем с помощью командлета Set-AzDiagnosticSetting вы включаете журнал ведения диагностики, отправляя выходные данные в новую учетную запись хранения.

### <a name="prerequisites"></a>Предварительные требования
Для работы с этим руководством вам потребуются следующие ресурсы:

* Существующий сервер Azure Analysis Services. Инструкции по созданию ресурса сервера см. в разделе [Создание сервера Azure Analysis Services на портале Azure](analysis-services-create-server.md) или [Создание сервера Azure Analysis Services с помощью PowerShell](analysis-services-create-powershell.md).

### <a name="aconnect-to-your-subscriptions"></a></a>Подключение к своим подпискам

Запустите сеанс Azure PowerShell и войдите в учетную запись Azure, используя следующую команду:  

```powershell
Connect-AzAccount
```

Во всплывающем окне браузера введите имя пользователя и пароль учетной записи Azure. Azure PowerShell получает все подписки, связанные с этой учетной записью, и по умолчанию использует первую из них.

Если у вас есть несколько подписок, возможно, вам нужно будет указать ту, которая использовалась для создания хранилища ключей Azure. Чтобы увидеть подписки для своей учетной записи, введите следующую команду:

```powershell
Get-AzSubscription
```

Затем укажите подписку, связанную с учетной записью Azure Analysis Services, данные которой будут регистрироваться. Для этого введите следующую команду.

```powershell
Set-AzContext -SubscriptionId <subscription ID>
```

> [!NOTE]
> Если с учетной записью связано несколько подписок, очень важно указать подписку, которую нужно использовать.
>
>

### <a name="create-a-new-storage-account-for-your-logs"></a>Создание учетной записи хранения для журналов

Можно использовать существующую учетную запись хранения для журналов, если она находится в той же подписке, что и ваш сервер. В этом руководстве создается новая учетная запись хранения для журналов Analysis Services. Для удобства вы сохраните сведения об учетной записи хранения в переменной **sa**.

Вы также используете группу ресурсов, которая содержит сервер Analysis Services. Замените значения `awsales_resgroup`, `awsaleslogs` и `West Central US` собственными значениями.

```powershell
$sa = New-AzStorageAccount -ResourceGroupName awsales_resgroup `
-Name awsaleslogs -Type Standard_LRS -Location 'West Central US'
```

### <a name="identify-the-server-account-for-your-logs"></a>Определение учетной записи сервера для журналов

Задайте в качестве имени учетной записи переменную **account**, где ResourceName — это имя учетной записи.

```powershell
$account = Get-AzResource -ResourceGroupName awsales_resgroup `
-ResourceName awsales -ResourceType "Microsoft.AnalysisServices/servers"
```

### <a name="enable-logging"></a>Включение ведения журналов

Чтобы включить ведение журнала, выполните командлет Set-AzDiagnosticSetting с переменными для новой учетной записи хранения, учетной записи сервера и категории. Выполните следующую команду, задав для флага **-Enabled** значение **$true**:

```powershell
Set-AzDiagnosticSetting  -ResourceId $account.ResourceId -StorageAccountId $sa.Id -Enabled $true -Categories Engine
```

Результат должен выглядеть примерно так:

```powershell
StorageAccountId            : 
/subscriptions/a23279b5-xxxx-xxxx-xxxx-47b7c6d423ea/resourceGroups/awsales_resgroup/providers/Microsoft.Storage/storageAccounts/awsaleslogs
ServiceBusRuleId            :
EventHubAuthorizationRuleId :
Metrics                    
    TimeGrain       : PT1M
    Enabled         : False
    RetentionPolicy
    Enabled : False
    Days    : 0


Logs                       
    Category        : Engine
    Enabled         : True
    RetentionPolicy
    Enabled : False
    Days    : 0


    Category        : Service
    Enabled         : False
    RetentionPolicy
    Enabled : False
    Days    : 0


WorkspaceId                 :
Id                          : /subscriptions/a23279b5-xxxx-xxxx-xxxx-47b7c6d423ea/resourcegroups/awsales_resgroup/providers/microsoft.analysisservic
es/servers/awsales/providers/microsoft.insights/diagnosticSettings/service
Name                        : service
Type                        :
Location                    :
Tags                        :
```

Этот результат значит, что ведение журнала для сервера включено, и данные сохраняются в учетной записи хранения.

Можно также задать политику хранения для журналов, чтобы устаревшие журналы автоматически удалялись. Например, задайте политику хранения, установив для флага **-RetentionEnabled** значение **$true**, а для параметра **-RetentionInDays** — значение **90**. Журналы, которым больше 90 дней, будут автоматически удаляться.

```powershell
Set-AzDiagnosticSetting -ResourceId $account.ResourceId`
 -StorageAccountId $sa.Id -Enabled $true -Categories Engine`
  -RetentionEnabled $true -RetentionInDays 90
```

## <a name="next-steps"></a>Дальнейшие действия

См. сведения [о ведении журнала ресурсов Azure Monitor](../azure-monitor/essentials/platform-logs-overview.md).

Ознакомьтесь с описанием [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) в справке PowerShell.