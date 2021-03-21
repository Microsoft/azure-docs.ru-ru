---
title: Узнайте, как подготовка интеграции с Azure Monitor журналами в Azure Active Directory.
description: Узнайте, как подготовка интеграции с Azure Monitor журналами в Azure Active Directory.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: conceptual
ms.workload: identity
ms.date: 10/12/2020
ms.author: kenwith
ms.reviewer: arvinh,luleon
ms.openlocfilehash: f656f55b0c74103053f110b8c66645d353db9fbf
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102561574"
---
# <a name="understand-how-provisioning-integrates-with-azure-monitor-logs"></a>Сведения о том, как подготовка интегрируется с журналами Azure Monitor

Подготовка интегрируется с журналами Azure Monitor и Log Analytics. С помощью мониторинга Azure можно выполнять такие действия, как создание книг, также известных как панели мониторинга, хранение журналов подготовки в течение 30 дней и создание настраиваемых запросов и оповещений. В этой статье описывается интеграция журналов подготовки с журналами Azure Monitor. Дополнительные сведения о работе журналов подготовки см. в статье [Подготовка журналов](../reports-monitoring/concept-provisioning-logs.md)в целом.

## <a name="enabling-provisioning-logs"></a>Включение журналов подготовки

Вы уже должны быть знакомы с мониторингом и Log Analytics Azure. Если нет, перейдите к разделу, чтобы узнать о них, а затем вернитесь к сведениям о журналах подготовки приложений. Дополнительные сведения о мониторинге Azure см. в разделе [Azure Monitor обзор](../../azure-monitor/overview.md). Дополнительные сведения о Azure Monitor журналах и Log Analytics см. [в статье Обзор запросов журналов в Azure Monitor](../../azure-monitor/logs/log-query-overview.md).

После настройки мониторинга Azure можно включить журналы для подготовки приложений. Параметр находится на странице **параметры диагностики** .

:::image type="content" source="media/application-provisioning-log-analytics/diagnostic-settings.png" alt-text="Доступ к параметрам диагностики" lightbox="media/application-provisioning-log-analytics/diagnostic-settings.png":::

:::image type="content" source="media/application-provisioning-log-analytics/enable-log-analytics.png" alt-text="Включить журналы подготовки приложений" lightbox="media/application-provisioning-log-analytics/enable-log-analytics.png":::

> [!NOTE]
> Если Рабочая область недавно была подготовлена, может потребоваться некоторое время, прежде чем можно будет отправить в нее журналы. Если вы получаете сообщение о том, что подписка не зарегистрирована для использования *Microsoft. Insights* , проверяйте ее через несколько минут.
 
## <a name="understanding-the-data"></a>Основные сведения о данных
Базовый поток данных, который подготовка отправляет средства просмотра журналов, практически идентичны. Azure Monitor журналы получают практически тот же поток, что и пользовательский интерфейс портал Azure и API Azure. Существует лишь несколько **отличий** в полях журнала, как описано в следующей таблице. Дополнительные сведения об этих полях см. в разделе [List провисионингобжектсуммари](/graph/api/provisioningobjectsummary-list?preserve-view=true&tabs=http&view=graph-rest-beta).

|Журналы Azure Monitor   |Пользовательский интерфейс портала Azure   |API Azure |
|----------|-----------|------------|
|errorDescription |reason |resultDescription |
|status |resultType |resultType |
|активитидатетиме |TimeGenerated |TimeGenerated |


## <a name="azure-monitor-workbooks"></a>Azure Monitor книги

Azure Monitor книги предоставляют гибкий холст для анализа данных. Они также обеспечивают создание визуальных отчетов с широкими возможностями в портал Azure. Дополнительные сведения см. в разделе [Общие сведения о Azure Monitor книгах](../../azure-monitor/visualize/workbooks-overview.md).

Подготовка приложений поставляется с набором готовых книг. Их можно найти на странице книги. Для просмотра данных необходимо убедиться, что все фильтры (timeRange, jobID, appName) заполнены. Кроме того, необходимо убедиться, что приложение подготовлено, в противном случае данные в журналах отсутствуют.

:::image type="content" source="media/application-provisioning-log-analytics/workbooks.png" alt-text="Книги подготовки приложений" lightbox="media/application-provisioning-log-analytics/workbooks.png":::

:::image type="content" source="media/application-provisioning-log-analytics/report.png" alt-text="Панель мониторинга подготовки приложений" lightbox="media/application-provisioning-log-analytics/report.png":::

## <a name="custom-queries"></a>Пользовательские запросы

Вы можете создавать пользовательские запросы и отображать данные на панелях мониторинга Azure. Дополнительные сведения см. в статье [Создание и совместное использование панелей мониторинга log Analytics данных](../../azure-monitor/logs/get-started-queries.md). Кроме того, ознакомьтесь с [обзором запросов журналов в Azure Monitor](../../azure-monitor/logs/log-query-overview.md).

Ниже приведены некоторые примеры, которые помогут приступить к подготовке приложений.

Запросите журналы для пользователя а на основе их идентификатора в исходной системе:
```kusto
AADProvisioningLogs
| extend SourceIdentity = parse_json(SourceIdentity)
| where tostring(SourceIdentity.Id) == "49a4974bb-5011-415d-b9b8-78caa7024f9a"
```

Число суммарных значений на код ошибки:
```kusto
AADProvisioningLogs
| summarize count() by ErrorCode = ResultSignature
```

Суммировать количество событий в день по действию:
```kusto
AADProvisioningLogs
| where TimeGenerated > ago(7d)
| summarize count() by Action, bin(TimeGenerated, 1d)
```

Возьмем 100 событий и ключевых свойств проекта:
```kusto
AADProvisioningLogs
| extend SourceIdentity = parse_json(SourceIdentity)
| extend TargetIdentity = parse_json(TargetIdentity)
| extend ServicePrincipal = parse_json(ServicePrincipal)
| where tostring(SourceIdentity.identityType) == "Group"
| project tostring(ServicePrincipal.Id), tostring(ServicePrincipal.Name), ModifiedProperties, JobId, Id, CycleId, ChangeId, Action, SourceIdentity.identityType, SourceIdentity.details, TargetIdentity.identityType, TargetIdentity.details, ProvisioningSteps
|take 100
```

## <a name="custom-alerts"></a>Настраиваемые оповещения

Azure Monitor позволяет настроить пользовательские оповещения, чтобы получать уведомления о ключевых событиях, связанных с подготовкой. Например, может потребоваться получить предупреждение о пиковых нагрузках в сбоях. Или, возможно, пики в отключении или удалении. Еще один пример того, где может потребоваться оповещение, — отсутствие какой-либо подготовки, что указывает на неправильное назначение.

Дополнительные сведения об оповещениях см. в статье [реагирование на события с помощью оповещений Azure Monitor](../../azure-monitor/alerts/tutorial-response.md).

Оповещать при возникновении сбоев. Замените идентификатор jobID для приложения.

:::image type="content" source="media/application-provisioning-log-analytics/alert1.png" alt-text="Оповещать при возникновении сбоев." lightbox="media/application-provisioning-log-analytics/alert1.png":::

Возможно, возникла ошибка, из – за которой служба подготовки может перестать работать. Используйте следующее предупреждение, чтобы определить, когда в течение заданного интервала времени нет событий подготовки.

:::image type="content" source="media/application-provisioning-log-analytics/alert2.png" alt-text="Возможно, возникла ошибка, из – за которой служба подготовки может перестать работать." lightbox="media/application-provisioning-log-analytics/alert2.png":::

Оповещать при появлении пика в отключении или удалении.

:::image type="content" source="media/application-provisioning-log-analytics/alert3.png" alt-text="Оповещать при появлении пика в отключении или удалении." lightbox="media/application-provisioning-log-analytics/alert3.png":::


## <a name="community-contributions"></a>Материалы сообщества

Мы используем подход на основе открытого кода и сообщества для подготовки к работе запросов и панелей мониторинга приложений. Если вы создали запрос, оповещение или книгу, которые могут оказаться полезными для других пользователей, обязательно опубликуйте ее в [репозитории GitHub азуремониторкоммунити](https://github.com/microsoft/AzureMonitorCommunity). Затем прокрутить сообщение электронной почты со ссылкой. Мы будем просматривать и публиковать его в службе, чтобы другие могли воспользоваться преимуществами. Свяжитесь с нами по адресу provisioningfeedback@microsoft.com.

## <a name="next-steps"></a>Следующие шаги

- [Log Analytics](../reports-monitoring/howto-analyze-activity-logs-log-analytics.md)
- [Начало работы с запросами журналов Azure Monitor](../../azure-monitor/logs/get-started-queries.md)
- [Создание групп действий и управление ими на портале Azure](../../azure-monitor/alerts/action-groups.md)
- [Установка и использование представлений Log Analytics для Azure Active Directory](../reports-monitoring/howto-install-use-log-analytics-views.md)
- [API подготовки журналов](/graph/api/resources/provisioningobjectsummary?preserve-view=true&view=graph-rest-beta)