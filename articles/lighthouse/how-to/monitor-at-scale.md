---
title: Мониторинг делегированных ресурсов в масштабе
description: Узнайте, как эффективно использовать журналы Azure Monitor в масштабируемом способе между клиентами клиентов, которыми вы управляете.
ms.date: 02/11/2021
ms.topic: how-to
ms.openlocfilehash: 98fd984492276dbdfbc2f8001bca19560764a2a7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101742591"
---
# <a name="monitor-delegated-resources-at-scale"></a>Мониторинг делегированных ресурсов в масштабе

Как поставщик услуг, вы могли подключить несколько клиентов клиента к [Azure лигхсаусе](../overview.md). Azure Lighthouse позволяет поставщикам служб выполнять операции одновременно в нескольких клиентах, что делает задачи управления более эффективными.

В этом разделе показано, как использовать [журналы Azure Monitor](../../azure-monitor/logs/data-platform-logs.md) в масштабируемом способе между клиентами клиентов, которыми вы управляете. Хотя в этом разделе мы будем называть поставщиков услуг и клиентов, это руководство также применяется к [предприятиям, использующим Azure лигхсаусе для управления несколькими клиентами](../concepts/enterprise.md).

> [!NOTE]
> Убедитесь, что пользователям в ваших управляющих клиентах были предоставлены [необходимые роли для управления рабочими областями log Analytics](../../azure-monitor/logs/manage-access.md#manage-access-using-azure-permissions) в делегированных подписках клиента.

## <a name="create-log-analytics-workspaces"></a>Создание рабочих областей Log Analytics

Для получения данных необходимо создать Log Analytics рабочие области. Эти Log Analytics рабочие области являются уникальными окружениями для данных, собираемых Azure Monitor. Каждая рабочая область имеет свои репозиторий данных и конфигурации, при этом источники данных и решения настроены для хранения данных в определенной рабочей области.

Рекомендуется создавать эти рабочие области непосредственно в клиентских клиентах. Таким образом их данные остаются в своих клиентах, а не экспортируются в. Это также позволяет осуществлять централизованный мониторинг любых ресурсов или служб, поддерживаемых Log Analytics, обеспечивая большую гибкость при мониторинге типов данных.

> [!TIP]
> Любая учетная запись службы автоматизации, используемая для доступа к данным из Log Analytics рабочей области, должна быть создана в том же клиенте, что и Рабочая область.

Рабочую область Log Analytics можно создать с помощью [портал Azure](../../azure-monitor/logs/quick-create-workspace.md), с помощью [Azure CLI](../../azure-monitor/logs/quick-create-workspace-cli.md)или с помощью [Azure PowerShell](../../azure-monitor/logs/powershell-workspace-configuration.md).

> [!IMPORTANT]
> Даже если все рабочие области созданы в клиенте клиента, поставщик ресурсов Microsoft. Insights также должен быть зарегистрирован в подписке в управляющем клиенте.

## <a name="deploy-policies-that-log-data"></a>Развертывание политик, которые заменяют данные журнала

После создания рабочих областей Log Analytics можно развернуть [политику Azure](../../governance/policy/index.yml) в иерархиях клиентов, чтобы данные диагностики отправлялись в соответствующую рабочую область каждого клиента. Точные развертываемые политики могут различаться в зависимости от типов ресурсов, которые требуется отслеживать.

Дополнительные сведения о создании политик см. в разделе [учебник. Создание политик и управление ими для обеспечения соответствия](../../governance/policy/tutorials/create-and-manage.md). Это [средство сообщества](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/tools/azure-diagnostics-policy-generator) предоставляет скрипт, помогающий создавать политики для мониторинга конкретных выбранных типов ресурсов.

Если вы определили, какие политики следует развернуть, [их можно развернуть в делегированных подписках в нужном масштабе](policy-at-scale.md).

## <a name="analyze-the-gathered-data"></a>Анализ собранных данных

После развертывания политик данные будут регистрироваться в Log Analytics рабочих областях, созданных в каждом клиенте клиента. Для получения подробных сведений обо всех управляемых клиентах можно использовать такие средства, как [Azure Monitor книги](../../azure-monitor/visualize/workbooks-overview.md) , для сбора и анализа информации из нескольких источников данных.

## <a name="view-alerts-across-customers"></a>Просмотр оповещений по клиентам

Вы можете просматривать [предупреждения](../../azure-monitor/alerts/alerts-overview.md) для делегированных подписок в клиентах клиентов, которыми управляет.

С помощью управляемого клиента можно [создавать, просматривать оповещения журнала действий и управлять ими](../../azure-monitor/alerts/alerts-activity-log.md) в портал Azure или через API-интерфейсы и средства управления.

Чтобы автоматически обновлять оповещения для нескольких клиентов, используйте запрос к [графу ресурсов Azure](../../governance/resource-graph/overview.md) для фильтрации оповещений. Вы можете закрепить запрос на панели мониторинга и выбрать всех соответствующих клиентов и подписок. Например, приведенный ниже запрос будет отображать предупреждения серьезности 0 и 1, обновляя каждые 60 минут.

```kusto
alertsmanagementresources
| where type == "microsoft.alertsmanagement/alerts"
| where properties.essentials.severity =~ "Sev0" or properties.essentials.severity =~ "Sev1"
| where properties.essentials.monitorCondition == "Fired"
| where properties.essentials.startDateTime > ago(60m)
| project StartTime=properties.essentials.startDateTime,name,Description=properties.essentials.description, Severity=properties.essentials.severity, subscriptionId
| sort by tostring(StartTime)
```

## <a name="next-steps"></a>Дальнейшие действия

- Испытайте [журналы действий по](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/workbook-activitylogs-by-domain) рабочей книге в GitHub.
- Изучите [образец книги, созданной](https://github.com/scautomation/Azure-Automation-Update-Management-Workbooks)с помощью MVP, которая отслеживает отчеты о соответствии требованиям, [запросив журналы Управление обновлениями](../../automation/update-management/query-logs.md) в нескольких log Analytics рабочих областях. 
- Узнайте о других [процессах управления между клиентами](../concepts/cross-tenant-management-experience.md).