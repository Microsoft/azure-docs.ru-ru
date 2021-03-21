---
title: Просмотр журналов действий для изменений Azure RBAC
description: Просмотрите журналы действий для управления доступом на основе ролей Azure (Azure RBAC) за последние 90 дней.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 03/01/2021
ms.author: rolyon
ms.custom: H1Hack27Feb2017, devx-track-azurecli
ms.openlocfilehash: d9b39bc9a2f00fe83cae0ff78c6346042967e8bf
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102042145"
---
# <a name="view-activity-logs-for-azure-rbac-changes"></a>Просмотр журналов действий для изменений Azure RBAC

Иногда требуются сведения об изменениях управления доступом на основе ролей Azure (Azure RBAC), например, в целях аудита или устранения неполадок. Когда кто-то вносит изменения в назначения ролей или определения ролей в ваших подписках, изменения регистрируются в [журнале действий Azure](../azure-monitor/essentials/platform-logs-overview.md). Вы можете просмотреть журналы действий, чтобы увидеть все изменения Azure RBAC за последние 90 дней.

## <a name="operations-that-are-logged"></a>Операции, которые регистрируются в журнале

Ниже приведены операции, связанные с Azure RBAC, которые регистрируются в журнале действий.

- Создание назначения роли
- Удаление назначения роли
- Создание или изменение определения пользовательской роли
- Удаление определения пользовательской роли

## <a name="azure-portal"></a>Портал Azure

Самый простой способ заключается в просмотре журналов действий на портале Azure. На следующем снимке экрана показан пример операций назначения ролей в журнале действий. Кроме того, здесь можно загрузить журналы в виде CSV-файла.

![Снимок экрана: просмотр журналов действий на портале](./media/change-history-report/activity-log-portal.png)

Чтобы получить дополнительные сведения, щелкните запись, чтобы открыть панель сводки. Щелкните вкладку **JSON** , чтобы получить подробный журнал.

![Журналы действий на портале с панелью сводки открыть-снимок экрана](./media/change-history-report/activity-log-summary-portal.png)

Журнал действий на портале имеет несколько фильтров. Ниже приведены фильтры, связанные с Azure RBAC.

| Filter | Значение |
| --------- | --------- |
| Категория событий | <ul><li>Административный</li></ul> |
| Операция | <ul><li>Создание назначения роли</li><li>Удаление назначения роли</li><li>Создание или изменение определения пользовательской роли</li><li>Удаление определения пользовательской роли</li></ul> |

Дополнительные сведения о журналах действий см. [в статье Просмотр журналов действий для отслеживания действий с ресурсами](../azure-resource-manager/management/view-activity-logs.md?toc=%2fazure%2fmonitoring-and-diagnostics%2ftoc.json).


## <a name="interpret-a-log-entry"></a>Интерпретировать запись журнала

Выходные данные журнала на вкладке JSON, Azure PowerShell или Azure CLI могут содержать много информации. Ниже приведены некоторые ключевые свойства, которые следует искать при попытке интерпретации записи журнала. Способы фильтрации выходных данных журнала с помощью Azure PowerShell или Azure CLI см. в следующих разделах.

> [!div class="mx-tableFixed"]
> | Свойство. | Примеры значений | Описание |
> | --- | --- | --- |
> | Авторизация: действие | Microsoft.Authorization/roleAssignments/write. | Создание назначения роли |
> |  | Microsoft.Authorization/roleAssignments/delete | Удаление назначения роли |
> |  | Microsoft.Authorization/roleDefinitions/write | Создание или обновление определения роли |
> |  | Microsoft.Authorization/roleDefinitions/delete | Удалить определение роли |
> | Авторизация: область | /субскриптионс/{субскриптионид}<br/>/субскриптионс/{субскриптионид}/ресаурцеграупс/{ресаурцеграупнаме}/провидерс/микрософт.аусоризатион/ролеассигнментс/{ролеассигнментид} | Область действия |
> | caller | admin@example.com<br/>ИД | Кто инициировал действие |
> | eventTimestamp | 2021-03-01T22:07:41.126243 Z | Время, когда произошло действие |
> | состояние: значение | Запуск<br/>Выполнено<br/>Сбой | Состояние действия |

## <a name="azure-powershell"></a>Azure PowerShell

Чтобы просмотреть журналы действий с помощью Azure PowerShell, используйте команду [Get-AzLog](/powershell/module/Az.Monitor/Get-AzLog).

Эта команда выводит список всех изменений назначений ролей в подписке за последние семь дней.

```azurepowershell
Get-AzLog -StartTime (Get-Date).AddDays(-7) | Where-Object {$_.Authorization.Action -like 'Microsoft.Authorization/roleAssignments/*'}
```

Эта команда выводит список всех изменений определений ролей в подписке за последние семь дней.

```azurepowershell
Get-AzLog -ResourceGroupName pharma-sales -StartTime (Get-Date).AddDays(-7) | Where-Object {$_.Authorization.Action -like 'Microsoft.Authorization/roleDefinitions/*'}
```

### <a name="filter-log-output"></a>Фильтровать выходные данные журнала

Выходные данные журнала могут содержать много информации. Эта команда выводит список всех изменений назначения ролей и определений ролей в подписке за последние семь дней и фильтрует выходные данные:

```azurepowershell
Get-AzLog -StartTime (Get-Date).AddDays(-7) | Where-Object {$_.Authorization.Action -like 'Microsoft.Authorization/role*'} | Format-List Caller,EventTimestamp,{$_.Authorization.Action},Properties
```

Ниже приведен пример выходных данных журнала, отфильтрованного при создании назначения роли.

```azurepowershell
Caller                  : admin@example.com
EventTimestamp          : 3/1/2021 10:07:42 PM
$_.Authorization.Action : Microsoft.Authorization/roleAssignments/write
Properties              :
                          statusCode     : Created
                          serviceRequestId: {serviceRequestId}
                          eventCategory  : Administrative
                          entity         : /subscriptions/{subscriptionId}/resourceGroups/example-group/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}
                          message        : Microsoft.Authorization/roleAssignments/write
                          hierarchy      : {tenantId}/{subscriptionId}

Caller                  : admin@example.com
EventTimestamp          : 3/1/2021 10:07:41 PM
$_.Authorization.Action : Microsoft.Authorization/roleAssignments/write
Properties              :
                          requestbody    : {"Id":"{roleAssignmentId}","Properties":{"PrincipalId":"{principalId}","PrincipalType":"User","RoleDefinitionId":"/providers/Microsoft.Authorization/roleDefinitions/fa23ad8b-c56e-40d8-ac0c-ce449e1d2c64","Scope":"/subscriptions/
                          {subscriptionId}/resourceGroups/example-group"}}
                          eventCategory  : Administrative
                          entity         : /subscriptions/{subscriptionId}/resourceGroups/example-group/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}
                          message        : Microsoft.Authorization/roleAssignments/write
                          hierarchy      : {tenantId}/{subscriptionId}

```

Если для создания назначений ролей используется субъект-служба, свойство caller будет являться ИДЕНТИФИКАТОРом объекта субъекта-службы. Вы можете использовать [Get-азадсервицепринЦипал](/powershell/module/az.resources/get-azadserviceprincipal) для получения сведений о субъекте-службе.

```Example
Caller                  : {objectId}
EventTimestamp          : 3/1/2021 9:43:08 PM
$_.Authorization.Action : Microsoft.Authorization/roleAssignments/write
Properties              : 
                          statusCode     : Created
                          serviceRequestId: {serviceRequestId}
                          eventCategory  : Administrative
```

## <a name="azure-cli"></a>Azure CLI

Чтобы просмотреть журналы действий с помощью Azure CLI, используйте команду [az monitor activity-log list](/cli/azure/monitor/activity-log#az_monitor_activity_log_list).

Эта команда выводит журналы действий в группе ресурсов с 1 марта, просматривая семь дней:

```azurecli
az monitor activity-log list --resource-group example-group --start-time 2021-03-01 --offset 7d
```

Эта команда выводит журналы действий для поставщика ресурсов авторизации с 1 марта, просматривая семь дней:

```azurecli
az monitor activity-log list --namespace "Microsoft.Authorization" --start-time 2021-03-01 --offset 7d
```

### <a name="filter-log-output"></a>Фильтровать выходные данные журнала

Выходные данные журнала могут содержать много информации. Эта команда выводит список всех изменений назначения ролей и определений ролей в подписке, которые просматривают семь дней и отфильтровывают выходные данные:

```azurecli
az monitor activity-log list --namespace "Microsoft.Authorization" --start-time 2021-03-01 --offset 7d --query '[].{authorization:authorization, caller:caller, eventTimestamp:eventTimestamp, properties:properties}'
```

Ниже приведен пример выходных данных журнала, отфильтрованного при создании назначения роли.

```azurecli
[
 {
    "authorization": {
      "action": "Microsoft.Authorization/roleAssignments/write",
      "role": null,
      "scope": "/subscriptions/{subscriptionId}/resourceGroups/example-group/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}"
    },
    "caller": "admin@example.com",
    "eventTimestamp": "2021-03-01T22:07:42.456241+00:00",
    "properties": {
      "entity": "/subscriptions/{subscriptionId}/resourceGroups/example-group/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
      "eventCategory": "Administrative",
      "hierarchy": "{tenantId}/{subscriptionId}",
      "message": "Microsoft.Authorization/roleAssignments/write",
      "serviceRequestId": "{serviceRequestId}",
      "statusCode": "Created"
    }
  },
  {
    "authorization": {
      "action": "Microsoft.Authorization/roleAssignments/write",
      "role": null,
      "scope": "/subscriptions/{subscriptionId}/resourceGroups/example-group/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}"
    },
    "caller": "admin@example.com",
    "eventTimestamp": "2021-03-01T22:07:41.126243+00:00",
    "properties": {
      "entity": "/subscriptions/{subscriptionId}/resourceGroups/example-group/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
      "eventCategory": "Administrative",
      "hierarchy": "{tenantId}/{subscriptionId}",
      "message": "Microsoft.Authorization/roleAssignments/write",
      "requestbody": "{\"Id\":\"{roleAssignmentId}\",\"Properties\":{\"PrincipalId\":\"{principalId}\",\"PrincipalType\":\"User\",\"RoleDefinitionId\":\"/providers/Microsoft.Authorization/roleDefinitions/fa23ad8b-c56e-40d8-ac0c-ce449e1d2c64\",\"Scope\":\"/subscriptions/{subscriptionId}/resourceGroups/example-group\"}}"
    }
  }
]
```

## <a name="azure-monitor-logs"></a>Журналы Azure Monitor

[Журналы Azure Monitor](../azure-monitor/logs/log-query-overview.md) — это еще один инструмент, с помощью которого можно получать и анализировать изменения Azure RBAC для всех ресурсов Azure. Журналы Azure Monitor имеют следующие преимущества:

- создание сложных запросов и логики;
- интеграция с оповещениями, Power BI и другими средствами;
- хранение данных в течение более длительного срока;
- поддержка перекрестных ссылок с другими журналами, такими как журналы безопасности, журналы виртуальных машин и настраиваемые журналы.

Ниже приведены основные шаги, позволяющие приступить к работе.

1. [Создайте рабочую область log Analytics](../azure-monitor/logs/quick-create-workspace.md).

1. [Настройка решения аналитики журнала действий](../azure-monitor/essentials/activity-log.md#activity-log-analytics-monitoring-solution) для рабочей области.

1. [Просмотр журналов действий](../azure-monitor/essentials/activity-log.md#activity-log-analytics-monitoring-solution). Чтобы быстро переходить на страницу обзора решения Аналитика журнала действий, щелкните пункт **журналы** .

   ![Параметр журналов Azure Monitor на портале](./media/change-history-report/azure-log-analytics-option.png)

1. При необходимости используйте [Azure Monitor log Analytics](../azure-monitor/logs/log-analytics-tutorial.md) для запроса и просмотра журналов. Дополнительные сведения см. [в разделе Приступая к работе с запросами журналов в Azure Monitor](../azure-monitor/logs/get-started-queries.md).

Ниже приведен запрос, возвращающий новые назначения ролей, упорядоченные по целевому поставщику ресурсов.

```Kusto
AzureActivity
| where TimeGenerated > ago(60d) and Authorization contains "Microsoft.Authorization/roleAssignments/write" and ActivityStatus == "Succeeded"
| parse ResourceId with * "/providers/" TargetResourceAuthProvider "/" *
| summarize count(), makeset(Caller) by TargetResourceAuthProvider
```

Ниже приведен запрос, возвращающий изменения назначения ролей, отображаемые на схеме.

```Kusto
AzureActivity
| where TimeGenerated > ago(60d) and Authorization contains "Microsoft.Authorization/roleAssignments"
| summarize count() by bin(TimeGenerated, 1d), OperationName
| render timechart
```

![Снимок экрана: журналы действий на портале расширенной аналитики](./media/change-history-report/azure-log-analytics.png)

## <a name="next-steps"></a>Дальнейшие действия
* [Просмотр журналов действий для отслеживания действий с ресурсами](../azure-resource-manager/management/view-activity-logs.md?toc=%2fazure%2fmonitoring-and-diagnostics%2ftoc.json)
* [Мониторинг действий подписки с помощью журнала действий Azure](../azure-monitor/essentials/platform-logs-overview.md)