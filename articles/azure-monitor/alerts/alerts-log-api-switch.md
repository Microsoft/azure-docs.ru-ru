---
title: Обновление до текущего Azure Monitor API оповещений журнала
description: Узнайте, как переключиться на API оповещений журнала Счедуледкуерирулес
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: b91560f09b30cc5d3089e5beca4d670f94f33402
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102031316"
---
# <a name="upgrade-to-the-current-log-alerts-api-from-legacy-log-analytics-alert-api"></a>Обновление до текущего API предупреждений журнала из устаревшей версии API предупреждений Log Analytics

> [!NOTE]
> Эта статья относится только к Azure для общего пользования (**не** в Azure для государственных организаций или в облаке Azure для Китая).

> [!NOTE]
> После того как пользователь выбрал параметр для перехода к текущему [API счедуледкуерирулес](/rest/api/monitor/scheduledqueryrules) , невозможно вернуться к старому [API предупреждений устаревших log Analytics](./api-alerts.md).

В прошлом пользователи использовали [Старый API предупреждений log Analytics](./api-alerts.md) для управления правилами генерации оповещений журнала. В текущих рабочих областях используется [API счедуледкуерирулес](/rest/api/monitor/scheduledqueryrules). В этой статье описываются преимущества и процесс перехода от устаревшего API к текущему API.

## <a name="benefits"></a>Преимущества

- Один шаблон для создания правил генерации оповещений (ранее требовалось три отдельных шаблона).
- Один API для Log Analytics рабочих областей или Application Insights ресурсов.
- [Поддержка командлетов PowerShell](./alerts-log.md#managing-log-alerts-using-powershell).
- Выравнивание серьезности со всеми другими типами оповещений.
- Возможность создания [оповещений журнала перекрестных рабочих областей](../logs/cross-workspace-query.md) , охватывающего несколько внешних ресурсов, таких как log Analytics рабочие области или ресурсы Application Insights.
- Пользователи могут указать измерения для разбиения предупреждений с помощью параметра "Aggregate on".
- Срок действия оповещений журнала составляет не более двух дней (раньше он был ограничен одним днем).

## <a name="impact"></a>Влияние

- Все новые правила должны быть созданы или изменены с помощью текущего API. См. [Пример использования с помощью шаблона ресурсов Azure](alerts-log-create-templates.md) и [примеры использования с помощью PowerShell](./alerts-log.md#managing-log-alerts-using-powershell).
- Так как правила становятся Azure Resource Manager отслеживанию ресурсов в текущем API и должны быть уникальными, идентификатор ресурса правил изменится на эту структуру: `<WorkspaceName>|<savedSearchId>|<scheduleId>|<ActionId>` . Отображаемые имена правила генерации оповещений останутся без изменений.

## <a name="process"></a>Процесс

Процесс переключения не является интерактивным и не требует действий, выполняемых вручную. в большинстве случаев это не требуется. Правила оповещений не останавливаются или не останавливаются во время или после переключения.
Выполните этот вызов, чтобы переключить все правила генерации оповещений, связанные с конкретной Log Analytics рабочей областью:

```
PUT /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview
```

С текстом запроса, содержащим приведенный ниже код JSON:

```json
{
    "scheduledQueryRulesEnabled" : true
}
```

Ниже приведен пример использования [ARMClient](https://github.com/projectkudu/ARMClient), средства командной строки с открытым кодом, которое упрощает вызов приведенного выше вызова API:

```powershell
$switchJSON = '{"scheduledQueryRulesEnabled": "true"}'
armclient PUT /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview $switchJSON
```

Если параметр успешно выполнен, ответ будет следующим:

```json
{
    "version": 2,
    "scheduledQueryRulesEnabled" : true
}
```

## <a name="check-switching-status-of-workspace"></a>Проверка состояния переключения рабочей области

Этот вызов API также можно использовать для проверки состояния переключателя:

```
GET /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview
```

Также можно использовать средство [ARMClient](https://github.com/projectkudu/ARMClient) :

```powershell
armclient GET /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview
```

Если Рабочая область Log Analytics была переключена на [API счедуледкуерирулес](/rest/api/monitor/scheduledqueryrules), ответ будет следующим:

```json
{
    "version": 2,
    "scheduledQueryRulesEnabled" : true
}
```
Если Рабочая область Log Analytics не была переключена, ответ будет следующим:

```json
{
    "version": 2,
    "scheduledQueryRulesEnabled" : false
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения см. в статье [Оповещения журнала в Azure Monitor](./alerts-unified-log.md).
- Узнайте, как [управлять оповещениями журналов с помощью API](alerts-log-create-templates.md).
- Узнайте, как [управлять оповещениями журнала с помощью PowerShell](./alerts-log.md#managing-log-alerts-using-powershell).
- Дополнительные сведения об [оповещениях Azure](./alerts-overview.md).