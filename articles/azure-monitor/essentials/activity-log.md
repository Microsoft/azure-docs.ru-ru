---
title: Журнал действий Azure
description: Просмотрите журнал действий Azure и отправьте его в журналы Azure Monitor, концентраторы событий Azure и службу хранилища Azure.
author: bwren
services: azure-monitor
ms.topic: conceptual
ms.date: 06/12/2020
ms.author: bwren
ms.openlocfilehash: 557fc6e358f371b47c1df314508e3565d843a28c
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102049191"
---
# <a name="azure-activity-log"></a>Журнал действий Azure
Журнал действий — это журнал [платформы в Azure](./platform-logs-overview.md), который предоставляет аналитические сведения о событиях уровня подписки, например об изменении ресурса или запуске виртуальной машины. Вы можете просмотреть журнал действий на портале Azure или получить записи с помощью PowerShell и CLI. Для дополнительной функциональности следует создать параметр диагностики для отправки журнала действий в [журналы Azure Monitor](../logs/data-platform-logs.md), в концентраторы событий Azure для пересылки за пределы Azure или в службу хранилища Azure для архивации. Эта статья содержит сведения о просмотре журнала действий и его отправке в различные назначения.

Дополнительные сведения о создании параметров диагностики см. в статье [Создание параметров диагностики для отправки журналов и метрик платформы в различные назначения](./diagnostic-settings.md) .

> [!NOTE]
> Записи в журнале действий формируются системой и их нельзя изменить или удалить.

## <a name="view-the-activity-log"></a>Просмотр журнала действий
Доступ к журналу действий можно получить из большинства меню на портале Azure. Ото того, из какого меню он открыт, зависит его первоначальный фильтр. Если открыть его из меню " **монитор** ", единственным фильтром будет только подписка. Если открыть его из меню ресурса, то для фильтра будет задан этот ресурс. Вы всегда можете изменить фильтр, чтобы просмотреть все остальные записи. Нажмите кнопку **Добавить фильтр** , чтобы добавить дополнительные свойства в фильтр.

![Просмотр журнала действий](./media/activity-log/view-activity-log.png)

Описание категорий журналов действий см. в статье [схема событий журнала действий Azure](activity-log-schema.md#categories).

### <a name="view-change-history"></a>Просмотр журнала изменений

Для некоторых событий можно просмотреть журнал изменений, который показывает, какие изменения произошли в течение этого времени события. Выберите из журнала действий событие, которое вам нужно изучить более подробно. Выберите вкладку **Журнал изменений (предварительная версия)** , чтобы просмотреть все изменения, связанные с этим событием.

![Список журнала изменений для события](media/activity-log/change-history-event.png)

Если с событием связаны какие-либо изменения, вы увидите здесь список изменений, которые можно выбрать. Это действие открывает страницу **Журнал изменений (предварительная версия)** . На этой странице отображаются все изменения ресурса. В следующем примере можно увидеть не только изменение размера виртуальной машины, но и то, что предыдущий размер виртуальной машины был до изменения и что он изменил. Дополнительные сведения о журнале изменений см. в разделе [Получение изменений ресурсов](../../governance/resource-graph/how-to/get-resource-changes.md).

![Страница журнала изменений с отображением различий](media/activity-log/change-history-event-details.png)


### <a name="other-methods-to-retrieve-activity-log-events"></a>Другие методы для получения событий журнала действий
Вы также можете получить доступ к событиям журнала действий с помощью следующих методов.

- Чтобы получить журнал действий с помощью PowerShell, используйте командлет [Get-AzLog](/powershell/module/az.monitor/get-azlog). См. раздел [примеры Azure Monitor PowerShell](../powershell-samples.md#retrieve-activity-log).
- Чтобы получить журнал действий с помощью интерфейса командной строки, используйте команду [az monitor activity-log](/cli/azure/monitor/activity-log).  См. [примеры CLI для Azure Monitor](../cli-samples.md#view-activity-log).
- Чтобы получить журнал действий из клиента REST, используйте [Azure Monitor REST API](/rest/api/monitor/). 


## <a name="send-to-log-analytics-workspace"></a>Отправка в рабочую область Log Analytics
 Отправьте журнал действий в Log Analytics рабочую область, чтобы включить функции [журналов Azure Monitor](../logs/data-platform-logs.md) , которые включают следующее:

- Сопоставьте данные журнала действий с другими данными мониторинга, собранными Azure Monitor.
- Объедините записи журнала из нескольких подписок Azure и клиентов в единое место для анализа вместе.
- Используйте запросы журналов для выполнения сложного анализа и получения подробных сведений о записях журнала действий.
- Используйте оповещения журнала с записями действий, что позволяет использовать более сложные алгоритмы оповещений.
- Храните записи журнала действий дольше 90 дней.
- Не взимается плата за получение данных журнала действий, хранящихся в Log Analytics рабочей области.
- Для данных журнала действий, хранящихся в Log Analytics рабочей области, не взимается плата за хранение данных до 90 дней.

[Создайте параметр диагностики](./diagnostic-settings.md) для отправки журнала действий в рабочую область log Analytics. Журнал действий можно отправить из любой одной подписки в до пяти рабочих областей. Для сбора журналов из различных клиентов требуется [Azure Lighthouse](../../lighthouse/index.yml).

Данные журнала действий в рабочей области Log Analytics хранятся в таблице с именем *AzureActivity* , которую можно получить с помощью [запроса журнала](../logs/log-query-overview.md) в [log Analytics](../logs/log-analytics-tutorial.md). Структура этой таблицы зависит от [категории записи журнала](activity-log-schema.md). Описание свойств таблицы см. в [справочнике по Azure Monitor данных](/azure/azure-monitor/reference/tables/azureactivity).

Например, чтобы просмотреть число записей журнала действий для каждой категории, используйте следующий запрос.

```kusto
AzureActivity
| summarize count() by CategoryValue
```

Чтобы получить все записи в административной категории, используйте следующий запрос.

```kusto
AzureActivity
| where CategoryValue == "Administrative"
```


## <a name="send-to-azure-event-hubs"></a>Отправка в концентраторы событий Azure
Отправьте журнал действий в концентраторы событий Azure, чтобы отправить записи за пределами Azure, например в сторонние SIEM или другие решения log Analytics. События журнала действий из концентраторов событий потребляются в формате JSON с `records` элементом, содержащим записи в каждой полезной нагрузке. Схема зависит от категории и описана в статье [схема из учетной записи хранения и концентраторов событий](activity-log-schema.md).

Ниже приведен пример выходных данных из концентраторов событий для журнала действий.

``` JSON
{
    "records": [
        {
            "time": "2019-01-21T22:14:26.9792776Z",
            "resourceId": "/subscriptions/s1/resourceGroups/MSSupportGroup/providers/microsoft.support/supporttickets/115012112305841",
            "operationName": "microsoft.support/supporttickets/write",
            "category": "Write",
            "resultType": "Success",
            "resultSignature": "Succeeded.Created",
            "durationMs": 2826,
            "callerIpAddress": "111.111.111.11",
            "correlationId": "c776f9f4-36e5-4e0e-809b-c9b3c3fb62a8",
            "identity": {
                "authorization": {
                    "scope": "/subscriptions/s1/resourceGroups/MSSupportGroup/providers/microsoft.support/supporttickets/115012112305841",
                    "action": "microsoft.support/supporttickets/write",
                    "evidence": {
                        "role": "Subscription Admin"
                    }
                },
                "claims": {
                    "aud": "https://management.core.windows.net/",
                    "iss": "https://sts.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db47/",
                    "iat": "1421876371",
                    "nbf": "1421876371",
                    "exp": "1421880271",
                    "ver": "1.0",
                    "http://schemas.microsoft.com/identity/claims/tenantid": "00000000-0000-0000-0000-000000000000",
                    "http://schemas.microsoft.com/claims/authnmethodsreferences": "pwd",
                    "http://schemas.microsoft.com/identity/claims/objectidentifier": "2468adf0-8211-44e3-95xq-85137af64708",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "admin@contoso.com",
                    "puid": "20030000801A118C",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "9vckmEGF7zDKk1YzIY8k0t1_EAPaXoeHyPRn6f413zM",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "John",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "Smith",
                    "name": "John Smith",
                    "groups": "cacfe77c-e058-4712-83qw-f9b08849fd60,7f71d11d-4c41-4b23-99d2-d32ce7aa621c,31522864-0578-4ea0-9gdc-e66cc564d18c",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": " admin@contoso.com",
                    "appid": "c44b4083-3bq0-49c1-b47d-974e53cbdf3c",
                    "appidacr": "2",
                    "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
                    "http://schemas.microsoft.com/claims/authnclassreference": "1"
                }
            },
            "level": "Information",
            "location": "global",
            "properties": {
                "statusCode": "Created",
                "serviceRequestId": "50d5cddb-8ca0-47ad-9b80-6cde2207f97c"
            }
        }
    ]
}
```


## <a name="send-to--azure-storage"></a>Отправка в службу хранилища Azure
Отправьте журнал действий в учетную запись хранения Azure, если вы хотите хранить данные журнала дольше 90 дней для аудита, статического анализа или резервного копирования. Если вам нужно хранить события только в течение 90 дней или меньше, вам не нужно настраивать архивацию для учетной записи хранения, так как события журнала действий сохраняются на платформе Azure в течение 90 дней.

При отправке журнала действий в Azure в учетной записи хранения создается контейнер хранилища, как только происходит событие. Большие двоичные объекты в контейнере используют следующее соглашение об именовании:

```
insights-activity-logs/resourceId=/SUBSCRIPTIONS/{subscription ID}/y={four-digit numeric year}/m={two-digit numeric month}/d={two-digit numeric day}/h={two-digit 24-hour clock hour}/m=00/PT1H.json
```

Например, определенный большой двоичный объект может иметь имя, аналогичное следующему:

```
insights-logs-networksecuritygrouprulecounter/resourceId=/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/y=2020/m=06/d=08/h=18/m=00/PT1H.json
```

Каждый большой двоичный объект PT1H.json содержит большой двоичный объект JSON с событиями, произошедшими в течение часа, указанного в URL-адресе этого объекта (например, h=12). В течение заданного часа события добавляются в файл PT1H.json по мере возникновения. Значение минуты (m = 00) всегда равно 00, так как события журнала ресурсов разбиваются на отдельные большие двоичные объекты в час.

Каждое событие хранится в PT1H.jsв файле со следующим форматом, который использует общую схему верхнего уровня, но для каждой категории в ином случае уникален, как описано в  [схеме журнала действий](activity-log-schema.md).

``` JSON
{ "time": "2020-06-12T13:07:46.766Z", "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/MY-RESOURCE-GROUP/PROVIDERS/MICROSOFT.COMPUTE/VIRTUALMACHINES/MV-VM-01", "correlationId": "0f0cb6b4-804b-4129-b893-70aeeb63997e", "operationName": "Microsoft.Resourcehealth/healthevent/Updated/action", "level": "Information", "resultType": "Updated", "category": "ResourceHealth", "properties": {"eventCategory":"ResourceHealth","eventProperties":{"title":"This virtual machine is starting as requested by an authorized user or process. It will be online shortly.","details":"VirtualMachineStartInitiatedByControlPlane","currentHealthStatus":"Unknown","previousHealthStatus":"Unknown","type":"Downtime","cause":"UserInitiated"}}}
```


## <a name="legacy-collection-methods"></a>Устаревшие методы коллекции
В этом разделе описаны устаревшие методы для сбора журнала действий, которые использовались до настройки диагностики. Если вы используете эти методы, следует рассмотреть возможность перехода к параметрам диагностики, обеспечивающим лучшую функциональность и согласованность с журналами ресурсов.

### <a name="log-profiles"></a>Профили журнала
Профили журналов — это устаревший метод отправки журнала действий в службу хранилища Azure или концентраторы событий. Используйте следующую процедуру, чтобы продолжить работу с профилем журнала или отключить его при подготовке к миграции в параметр диагностики.

1. В меню **Azure Monitor** в портал Azure выберите пункт **Журнал действий**.
3. Щелкните **Параметры диагностики**.

   ![Параметры диагностики](media/activity-log/diagnostic-settings.png)

4. Щелкните сиреневый баннер для устаревшего интерфейса.

    ![Устаревшие возможности](media/activity-log/legacy-experience.png)



### <a name="configure-log-profile-using-powershell"></a>Настройка профиля журнала с помощью PowerShell

Если профиль журнала уже существует, сначала необходимо удалить существующий профиль журнала, а затем создать новый.

1. Используйте `Get-AzLogProfile`, чтобы проверить, существует ли профиль журнала.  Если профиль журнала существует, обратите внимание на свойство *Name* .

1. Удалите журнал профиля с помощью `Remove-AzLogProfile`, используя значение свойства *name*.

    ```powershell
    # For example, if the log profile name is 'default'
    Remove-AzLogProfile -Name "default"
    ```

3. Создайте профиль журнала с помощью `Add-AzLogProfile`:

    ```powershell
    Add-AzLogProfile -Name my_log_profile -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -serviceBusRuleId /subscriptions/s1/resourceGroups/Default-ServiceBus-EastUS/providers/Microsoft.ServiceBus/namespaces/mytestSB/authorizationrules/RootManageSharedAccessKey -Location global,westus,eastus -RetentionInDays 90 -Category Write,Delete,Action
    ```

    | Свойство | Обязательно | Описание |
    | --- | --- | --- |
    | name |Да |Имя профиля журнала. |
    | StorageAccountId |Нет |Идентификатор ресурса учетной записи хранения, в которой должен быть сохранен журнал действий. |
    | serviceBusRuleId |Нет |Идентификатор правила служебной шины для пространства имен служебной шины, в котором вы сможете создать концентраторы событий. Это строка в формате: `{service bus resource ID}/authorizationrules/{key name}` . |
    | Расположение |Да |Разделенный запятыми список регионов, для которых будут собираться события журнала действий. |
    | RetentionInDays |Да |Число дней, в течение которых события должны храниться в учетной записи хранения в диапазоне от 1 до 365. Нулевое значение означает, что журналы хранятся неограниченно долго. |
    | Категория |Нет |Разделенный запятыми список категорий событий, которые будут собираться. Возможные значения: _Write_, _Delete_ и _Action_. |

### <a name="example-script"></a>Пример сценария
Ниже приведен пример скрипта PowerShell для создания профиля журнала, который записывает журнал действий в учетную запись хранения и концентратор событий.

   ```powershell
   # Settings needed for the new log profile
   $logProfileName = "default"
   $locations = (Get-AzLocation).Location
   $locations += "global"
   $subscriptionId = "<your Azure subscription Id>"
   $resourceGroupName = "<resource group name your event hub belongs to>"
   $eventHubNamespace = "<event hub namespace>"

   # Build the service bus rule Id from the settings above
   $serviceBusRuleId = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.EventHub/namespaces/$eventHubNamespace/authorizationrules/RootManageSharedAccessKey"

   # Build the storage account Id from the settings above
   $storageAccountId = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$storageAccountName"

   Add-AzLogProfile -Name $logProfileName -Location $locations -StorageAccountId  $storageAccountId -ServiceBusRuleId $serviceBusRuleId
   ```


### <a name="configure-log-profile-using-azure-cli"></a>Настройка профиля журнала с помощью Azure CLI

Если профиль журнала уже существует, сначала удалите его, а затем создайте новый.

1. Используйте `az monitor log-profiles list`, чтобы проверить, существует ли профиль журнала.
2. Удалите журнал профиля с помощью `az monitor log-profiles delete --name "<log profile name>`, используя значение свойства *name*.
3. Создайте профиль журнала с помощью `az monitor log-profiles create`:

   ```azurecli-interactive
   az monitor log-profiles create --name "default" --location null --locations "global" "eastus" "westus" --categories "Delete" "Write" "Action"  --enabled false --days 0 --service-bus-rule-id "/subscriptions/<YOUR SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventHub/namespaces/<EVENT HUB NAME SPACE>/authorizationrules/RootManageSharedAccessKey"
   ```

    | Свойство | Обязательно | Описание |
    | --- | --- | --- |
    | name |Да |Имя профиля журнала. |
    | storage-account-id |Да |Идентификатор ресурса для учетной записи хранения, в которую будут сохранены журналы действий. |
    | Расположения |Да |Разделенный пробелами список регионов, для которых будут собираться события журнала действий. Вы можете просмотреть список всех регионов для своей подписки с помощью `az account list-locations --query [].name`. |
    | days |Да |Число дней, в течение которых должны храниться события, от 1 до 365. Нулевое значение означает, что журналы будут храниться неограниченно долго, то есть всегда.  Если значение равно нулю, то параметру Enabled должно быть присвоено значение false. |
    |Включено | Да |Верно или неверно.  Позволяет включить или отключить политику хранения.  Если установлено значение True, параметр дней должен иметь значение больше 0.
    | Категории |Да |Разделенный пробелами список категорий событий, которые будут собираться. Возможные значения: Write, Delete или Action. |


### <a name="log-analytics-workspace"></a>Рабочая область Log Analytics
Устаревший метод отправки журнала действий в рабочую область Log Analytics подключается к журналу в конфигурации рабочей области. 

1. В меню **log Analytics рабочие области** в портал Azure выберите рабочую область, чтобы получить журнал действий.
1. В разделе **Источники данных рабочей области** в меню рабочей области выберите **Журнал действий Azure**.
1. Щелкните подписку, которую нужно подключить.

    ![На снимке экрана показана Log Analytics Рабочая область с выбранным журналом действий Azure.](media/activity-log/workspaces.png)

2. Нажмите кнопку **подключить** , чтобы подключить журнал действий в подписке к выбранной рабочей области. Если подписка уже подключена к другой рабочей области, нажмите кнопку **Отключить** сначала, чтобы отключить ее.

    ![Подключение рабочих областей](media/activity-log/connect-workspace.png)


Чтобы отключить этот параметр, выполните ту же процедуру и нажмите кнопку **Отключить** , чтобы удалить подписку из рабочей области.

### <a name="data-structure-changes"></a>Изменения структуры данных
Параметры диагностики отправляют те же данные, что и устаревший метод, используемый для отправки журнала действий с некоторыми изменениями в структуре таблицы *AzureActivity* .

Столбцы в следующей таблице не являются устаревшими в обновленной схеме. Они по-прежнему существуют в *AzureActivity* , но они не будут иметь данных. Замена этих столбцов не является новой, но содержит те же данные, что и устаревший столбец. Они имеют другой формат, поэтому может потребоваться изменить запросы к журналу, которые их используют. 

| Устаревший столбец | Столбец замены |
|:---|:---|
| ActivityStatus    | ActivityStatusValue    |
| активитисубстатус | активитисубстатусвалуе |
| Категория          | категоривалуе          |
| OperationName     | оператионнамевалуе     |
| ResourceProvider  | ресаурцепровидервалуе  |

> [!IMPORTANT]
> В некоторых случаях значения в этих столбцах могут быть заданы в верхнем регистре. Если запрос включает эти столбцы, следует использовать [оператор =~](/azure/kusto/query/datatypes-string-operators) для сравнения без учета регистра.

Следующий столбец был добавлен в *AzureActivity* в обновленной схеме:

- Authorization_d
- Claims_d
- Properties_d

## <a name="activity-log-analytics-monitoring-solution"></a>Решение для мониторинга Аналитика журнала действий
Решение для мониторинга Log Analytics Azure будет считаться устаревшим и заменено книгой, использующей обновленную схему в рабочей области Log Analytics. Вы по-прежнему можете использовать решение, если оно уже включено, но его можно использовать только при сборе журнала действий с использованием устаревших параметров. 



### <a name="use-the-solution"></a>Использование решения
Доступ к решениям мониторинга осуществляется из меню " **монитор** " в портал Azure. Выберите **больше** в разделе **Insights** , чтобы открыть страницу **обзора** с плитками решения. На плитке **журналы действий Azure** отображается количество записей **AzureActivity** в рабочей области.

![Плитка "Журналы действий Azure"](media/activity-log/azure-activity-logs-tile.png)


Щелкните плитку **журналы действий Azure** , чтобы открыть представление **журналы действий Azure** . Представление включает в себя элементы визуализации, приведенные в следующей таблице. Каждая часть содержит до 10 элементов, соответствующих критериям этой части за указанный диапазон времени. Можно выполнить запрос журнала, возвращающий все совпадающие записи, щелкнув **Просмотреть все** в нижней части части.

![Панель мониторинга "Журналы действий Azure"](media/activity-log/activity-log-dash.png)


### <a name="enable-the-solution-for-new-subscriptions"></a>Включение решения для новых подписок
Вскоре вы больше не сможете добавить решение "аналитика журналов действий" в подписку с помощью портал Azure. Его можно добавить, используя следующую процедуру с шаблоном диспетчер ресурсов. 

1. Скопируйте следующий код JSON в файл с именем *активитилогтемплате*. JSON.

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "type": "String",
            "defaultValue": "my-workspace",
            "metadata": {
              "description": "Specifies the name of the workspace."
            }
        },
        "location": {
            "type": "String",
            "allowedValues": [
              "east us",
              "west us",
              "australia central",
              "west europe"
            ],
            "defaultValue": "australia central",
            "metadata": {
              "description": "Specifies the location in which to create the workspace."
            }
        }
      },
        "resources": [
        {
            "type": "Microsoft.OperationalInsights/workspaces",
            "name": "[parameters('workspaceName')]",
            "apiVersion": "2015-11-01-preview",
            "location": "[parameters('location')]",
            "properties": {
                "features": {
                    "searchVersion": 2
                }
            }
        },
        {
            "type": "Microsoft.OperationsManagement/solutions",
            "apiVersion": "2015-11-01-preview",
            "name": "[concat('AzureActivity(', parameters('workspaceName'),')')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('microsoft.operationalinsights/workspaces', parameters('workspaceName'))]"
            ],
            "plan": {
                "name": "[concat('AzureActivity(', parameters('workspaceName'),')')]",
                "promotionCode": "",
                "product": "OMSGallery/AzureActivity",
                "publisher": "Microsoft"
            },
            "properties": {
                "workspaceResourceId": "[resourceId('microsoft.operationalinsights/workspaces', parameters('workspaceName'))]",
                "containedResources": [
                    "[concat(resourceId('microsoft.operationalinsights/workspaces', parameters('workspaceName')), '/views/AzureActivity(',parameters('workspaceName'))]"
                ]
            }
        },
        {
          "type": "Microsoft.OperationalInsights/workspaces/datasources",
          "kind": "AzureActivityLog",
          "name": "[concat(parameters('workspaceName'), '/', subscription().subscriptionId)]",
          "apiVersion": "2015-11-01-preview",
          "location": "[parameters('location')]",
          "dependsOn": [
              "[parameters('WorkspaceName')]"
          ],
          "properties": {
              "linkedResourceId": "[concat(subscription().Id, '/providers/microsoft.insights/eventTypes/management')]"
          }
        }
      ]
    }    
    ```

2. Разверните шаблон с помощью следующих команд PowerShell:

    ```PowerShell
    Connect-AzAccount
    Select-AzSubscription <SubscriptionName>
    New-AzResourceGroupDeployment -Name activitysolution -ResourceGroupName <ResourceGroup> -TemplateFile <Path to template file>
    ```



## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь со статьей [Общие сведения о журналах платформы Azure](./platform-logs-overview.md).
* [Проверка схемы событий журнала действий](activity-log-schema.md)
* См. статью о [создании параметра диагностики для отправки журналов действий в другие целевые расположения](./diagnostic-settings.md).