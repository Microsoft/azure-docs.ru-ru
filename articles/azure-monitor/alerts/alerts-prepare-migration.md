---
title: Обновление приложений логики & модули Runbook для миграции предупреждений
description: Узнайте, как изменить веб-перехватчики, приложения логики и модули Runbook для подготовки к добровольной миграции.
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 02/14/2021
ms.openlocfilehash: ce61c3539a4ea29cbeb48c379ed143363500701e
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102038022"
---
# <a name="prepare-your-logic-apps-and-runbooks-for-migration-of-classic-alert-rules"></a>Prepare your logic apps and runbooks for migration of classic alert rules (Подготовка приложений логики и модулей runbook для миграции классических правил генерации оповещений)

> [!NOTE]
> Как [было объявлено ранее](monitoring-classic-retirement.md), классические оповещения в Azure Monitor исключаются для пользователей общедоступного облака, хотя и в ограниченном объеме использования до **31 мая 2021**. Классические оповещения для облачных служб Azure для государственных организаций и Azure для Китая, истечение **29 февраля 2024**.
>

Если вы решили самостоятельно перенести классические правила генерации оповещений в новые правила генерации оповещений, существуют некоторые различия между этими двумя системами. В этой статье объясняются эти различия и способы подготовки к изменению.

## <a name="api-changes"></a>Изменения API

Интерфейсы API, которые создают классические правила генерации оповещений и управляют ими ( `microsoft.insights/alertrules` ), отличаются от API-интерфейсов, которые создают новые оповещения метрик () и управляют ими `microsoft.insights/metricalerts` . Если вы программно создаете классические правила генерации оповещений и управляете ими уже сегодня, обновите сценарии развертывания для работы с новыми API.

В следующей таблице приведены ссылки на программные интерфейсы для классических и новых оповещений.

| Тип скрипта развертывания | Классические оповещения | Новые оповещения метрик |
| ---------------------- | -------------- | ----------------- |
|REST API     | [microsoft.insights/alertrules](/rest/api/monitor/alertrules)         | [microsoft.insights/metricalerts](/rest/api/monitor/metricalerts)       |
|Azure CLI     | [az monitor alert](/cli/azure/monitor/alert)        | [предупреждение о метриках монитора AZ](/cli/azure/monitor/metrics/alert)        |
|PowerShell      | [Ссылки](/powershell/module/az.monitor/add-azmetricalertrule)       |  [Ссылки](/powershell/module/az.monitor/add-azmetricalertrulev2)    |
| Шаблон Azure Resource Manager | [Для классических оповещений](./alerts-enable-template.md)|[Для новых оповещений метрик](./alerts-metric-create-templates.md)|

## <a name="notification-payload-changes"></a>Изменения полезных данных уведомлений

Формат полезных данных уведомления немного отличается между [классическими правилами генерации оповещений](alerts-webhooks.md) и [новыми оповещениями метрик](alerts-metric-near-real-time.md#payload-schema). Если у вас есть классические правила генерации оповещений с веб-перехватчиком, приложением логики или действиями Runbook, необходимо обновить целевые объекты, чтобы они принимали новый формат полезных данных.

Используйте следующую таблицу для привязки полей полезных данных веб-перехватчика из классического формата к новому формату:

| Тип конечной точки уведомления | Классические оповещения | Новые оповещения метрик |
| -------------------------- | -------------- | ----------------- |
|Было ли оповещение активировано или разрешено?    | **status**       | **данные. состояние** |
|Контекстные сведения о предупреждении     | **context**        | **Data. Context**        |
|Метка времени, когда оповещение было активировано или разрешено     | **контекст. timestamp**       | **Data. Context. timestamp**        |
| Идентификатор правила оповещения | **context.id** | **data.context.id** |
| Имя правила генерации оповещений | **context.name** | **data.context.name** |
| Описание правила генерации оповещений | **context. Description** | **Data. Context. Description** |
| Условие для правила генерации оповещений | **context. Condition** | **Data. Context. Condition** |
| Имя метрики | **context. Condition. metricName** | **Data. Context. Condition. allOf [0]. metricName** |
| Статистическая обработка времени (вычисление метрики в окне оценки)| **context. Condition. timeAggregation** | **context. Condition. timeAggregation** |
| Период оценки | **context. Condition. windowSize** | **Data. Context. Condition. windowSize** |
| Оператор (как агрегированное значение метрики сравнивается с пороговым значением) | **context. Condition. оператор** | **Data. Context. Condition. оператор** |
| Порог | **context. Condition. threshold** | **Data. Context. Condition. allOf [0]. пороговое значение** |
| Значение метрики | **context. Condition. Метриквалуе** | **Data. Context. Condition. allOf [0]. Метриквалуе** |
| Идентификатор подписки | **context. subscriptionId** | **Data. Context. subscriptionId** |
| Группа ресурсов затронутого ресурса | **context. resourceGroup** | **Data. Context. resourceGroup** |
| Имя затронутого ресурса | **context. resourceName** | **Data. Context. resourceName** |
| Тип затрагиваемого ресурса | **context. resourceType** | **Data. Context. resourceType** |
| Идентификатор ресурса затронутого ресурса | **context. resourceId** | **Data. Context. resourceId** |
| Прямая ссылка на страницу сводки по ресурсам портала | **context. Порталлинк** | **Data. Context. Порталлинк** |
| Настраиваемые поля полезных данных, передаваемые веб-перехватчику или приложению логики | **properties** | **Data. Properties** |

Полезные данные похожи, как видите. В следующем разделе предлагаются следующие возможности.

- Сведения об изменении формата приложений логики для работы с новым форматом.
- Пример модуля Runbook, который анализирует полезные данные уведомления для новых оповещений.

## <a name="modify-a-logic-app-to-receive-a-metric-alert-notification"></a>Изменение приложения логики для получения уведомлений о предупреждениях метрик

Если вы используете приложения логики с классическими оповещениями, необходимо изменить код логики приложения, чтобы проанализировать новые полезные данные оповещений метрик. Выполните следующие действия:

1. Создайте новое приложение логики.

1. Используйте шаблон "Azure Monitor — обработчик оповещений об метриках". Этот шаблон содержит триггер **http-запроса** с соответствующей определенной схемой.

    ![На снимке экрана показаны две кнопки: Пустое приложение логики и Azure Monitor — обработчик оповещений метрик.](media/alerts-prepare-migration/logic-app-template.png "Шаблон оповещения метрики")

1. Добавьте действие для размещения логики обработки.

## <a name="use-an-automation-runbook-that-receives-a-metric-alert-notification"></a>Использование модуля Runbook службы автоматизации, который получает уведомление о предупреждении метрики

В следующем примере показан код PowerShell для использования в модуле Runbook. Этот код может анализировать полезные данные для классических правил генерации оповещений метрик и новых правил генерации оповещений метрик.

```PowerShell
## Example PowerShell code to use in a runbook to handle parsing of both classic and new metric alerts.

[OutputType("PSAzureOperationResponse")]

param
(
    [Parameter (Mandatory=$false)]
    [object] $WebhookData
)

$ErrorActionPreference = "stop"

if ($WebhookData)
{
    # Get the data object from WebhookData.
    $WebhookBody = (ConvertFrom-Json -InputObject $WebhookData.RequestBody)

    # Determine whether the alert triggering the runbook is a classic metric alert or a new metric alert (depends on the payload schema).
    $schemaId = $WebhookBody.schemaId
    Write-Verbose "schemaId: $schemaId" -Verbose
    if ($schemaId -eq "AzureMonitorMetricAlert") {

        # This is the new metric alert schema.
        $AlertContext = [object] ($WebhookBody.data).context
        $status = ($WebhookBody.data).status

        # Parse fields related to alert rule condition.
        $metricName = $AlertContext.condition.allOf[0].metricName
        $metricValue = $AlertContext.condition.allOf[0].metricValue
        $threshold = $AlertContext.condition.allOf[0].threshold
        $timeAggregation = $AlertContext.condition.allOf[0].timeAggregation
    }
    elseif ($schemaId -eq $null) {
        # This is the classic metric alert schema.
        $AlertContext = [object] $WebhookBody.context
        $status = $WebhookBody.status

        # Parse fields related to alert rule condition.
        $metricName = $AlertContext.condition.metricName
        $metricValue = $AlertContext.condition.metricValue
        $threshold = $AlertContext.condition.threshold
        $timeAggregation = $AlertContext.condition.timeAggregation
    }
    else {
        # The schema is neither a classic metric alert nor a new metric alert.
        Write-Error "The alert data schema - $schemaId - is not supported."
    }

    # Parse fields related to resource affected.
    $ResourceName = $AlertContext.resourceName
    $ResourceType = $AlertContext.resourceType
    $ResourceGroupName = $AlertContext.resourceGroupName
    $ResourceId = $AlertContext.resourceId
    $SubId = $AlertContext.subscriptionId

    ## Your logic to handle the alert here.
}
else {
    # Error
    Write-Error "This runbook is meant to be started from an Azure alert webhook only."
}

```

Полный пример модуля Runbook, который останавливает виртуальную машину при активации оповещения, см. в [документации по службе автоматизации Azure](../../automation/automation-create-alert-triggered-runbook.md).

## <a name="partner-integration-via-webhooks"></a>Интеграция партнеров через веб-перехватчики

Большинство [наших партнеров, которые интегрируются с классическими оповещениями,](../partners.md) уже поддерживают новые оповещения метрик с помощью их интеграции. Известные интеграции, уже работающие с новыми оповещениями метрик:

- [PagerDuty](https://www.pagerduty.com/docs/guides/azure-integration-guide/)
- [OpsGenie](https://docs.opsgenie.com/docs/microsoft-azure-integration)
- [Signl4](https://www.signl4.com/blog/mobile-alert-notifications-azure-monitor/)

Если вы используете интеграцию с партнером, не указанную здесь, выясните у поставщика, что они работают с новыми оповещениями метрик.

## <a name="next-steps"></a>Дальнейшие действия

- [Как использовать средство миграции](alerts-using-migration-tool.md)
- [Принцип работы средства миграции](alerts-understand-migration.md)