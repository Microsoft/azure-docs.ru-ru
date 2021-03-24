---
title: Использование оповещения для активации runbook службы автоматизации Azure
description: В этой статье рассказывается, как активировать модуль runbook при появлении оповещения Azure.
services: automation
ms.subservice: process-automation
ms.date: 02/14/2021
ms.topic: conceptual
ms.openlocfilehash: ea7979ad4a401d317ec126b7abfe354690475235
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104953108"
---
# <a name="use-an-alert-to-trigger-an-azure-automation-runbook"></a>Использование оповещения для активации runbook службы автоматизации Azure

С помощью [Azure Monitor](../azure-monitor/overview.md) можно отслеживать метрики и журналы базового уровня большинства служб Microsoft Azure. Вы можете вызывать модули Runbook службы автоматизации Azure, используя [группы действий](../azure-monitor/alerts/action-groups.md) для автоматизации задач на основе предупреждений. В этой статье показано, как настроить и запустить runbook с помощью оповещений.

## <a name="alert-types"></a>Типы оповещений

Модули runbook службы автоматизации можно использовать с тремя типами оповещений:

* Распространенные оповещения
* Оповещения журнала действий
* Оповещения метрик почти в реальном времени

> [!NOTE]
> Общая схема оповещений позволяет стандартизировать возможности использования уведомлений об оповещениях в Azure в настоящее время. Исторически сложилось, что на сегодняшний день существует три типа оповещений в Azure (метрика, журнал и журнал действий). Все они имеют собственные шаблоны электронной почты, схемы веб-перехватчиков и т. д. Дополнительные сведения см. в разделе [Общая схема оповещений](../azure-monitor/alerts/alerts-common-schema.md).

Когда оповещение вызывает runbook, фактически осуществляется вызов веб-перехватчика с помощью HTTP-запроса POST. Текст запроса POST содержит объект в формате JSON с полезными свойствами, связанными с оповещением. В следующей таблице приведены ссылки на схему полезных данных для каждого типа оповещения.

|Предупреждение  |Описание|Схема полезных данных  |
|---------|---------|---------|
|[Общее оповещение](../azure-monitor/alerts/alerts-common-schema.md)|Общая схема оповещений, которая стандартизирует возможности использования уведомлений об оповещениях в Azure в настоящее время.|Общая схема полезных данных оповещений|
|[Оповещение журнала действий](../azure-monitor/alerts/activity-log-alerts.md)    |Отправляет уведомление, когда какое-либо новое событие в журнале действий Azure соответствует определенным условиям. Например, если в **myProductionResourceGroup** выполняется операция `Delete VM` или возникает новое событие службы "Работоспособность служб Azure" в состоянии "Активно".| [Схема полезных данных оповещения журнала действий](../azure-monitor/alerts/activity-log-alerts-webhook.md)        |
|[Оповещения на основе метрик практически в реальном времени](../azure-monitor/alerts/alerts-metric-near-real-time.md)    |Отправляет уведомление быстрее, чем оповещения о метриках, когда одна или несколько метрик уровня платформы удовлетворяют указанным условиям. Например, если за последние 5 минут значение метрики **% ЦП** на виртуальной машине превышает 90 и значение метрики **Сеть (входящий трафик)** превышает 500 МБ.| [Схема полезны данных оповещения на основе метрик почти в реальном времени](../azure-monitor/alerts/alerts-webhooks.md#payload-schema)          |

Так как данные, предоставляемые для каждого типа оповещения, различны, оповещения разных типов обрабатываются по-разному. В следующем разделе вы узнаете, как создать runbook для обработки различных типов оповещений.

## <a name="create-a-runbook-to-handle-alerts"></a>Создание модуля Runbook для обработки оповещений

Чтобы применять автоматизацию к оповещениям, нужен runbook с логикой для управления полезными данными JSON оповещения, которые передаются в runbook. Приведенный ниже пример модуля Runbook должен быть вызван из оповещения Azure.

Как описано в предыдущем разделе, каждый тип оповещения имеет свою схему. Сценарий принимает данные веб-перехватчика из оповещения во входном параметре модуля runbook `WebhookData`. Затем сценарий оценивает полезные данные JSON, чтобы определить, какой тип оповещения использован.

В этом примере используется оповещение из виртуальной машины. Он извлекает данные виртуальной машины из полезных данных и использует эту информацию для остановки виртуальной машины. Подключение должно быть настроено в учетной записи службы автоматизации, в которой выполняется runbook. При использовании оповещений для активации модулей runbook очень важно проверять состояние оповещения в активируемом модуле. Модуль runbook будет активирован каждый раз, когда состояние оповещения меняется. Оповещения имеют несколько состояний, из которых два наиболее распространенных — "Активировано" и "Разрешено". Проверьте состояние в логике модуля runbook, чтобы убедиться, что модуль runbook выполняется не более одного раза. Пример в этой статье показывает, как выполнять поиск только оповещений с состоянием "Активировано".

Модуль runbook будет использовать [учетную запись запуска от имени](./automation-security-overview.md) `AzureRunAsConnection` для проверки подлинности в Azure, чтобы выполнить действие управления с виртуальной машиной.

Используйте этот пример для создания runbook **Stop-AzureVmInResponsetoVMAlert**. Можно изменить сценарий PowerShell и использовать его для множества разных ресурсов.

1. Перейдите к своей учетной записи службы автоматизации Azure.
2. В разделе **Автоматизация процессов** щелкните **Модули Runbook**.
3. В верхней части списка модулей runbook выберите **+ Создать Runbook**.
4. На странице **Добавить модуль runbook** введите в поле имени модуля значение **AzureVmInResponsetoVMAlert**. Для типа runbook выберите значение **PowerShell**. Затем выберите **Создать**.  
5. Скопируйте следующий пример PowerShell на страницу **Изменение**.

    ```powershell-interactive
    [OutputType("PSAzureOperationResponse")]
    param
    (
        [Parameter (Mandatory=$false)]
        [object] $WebhookData
    )
    $ErrorActionPreference = "stop"

    if ($WebhookData)
    {
        # Get the data object from WebhookData
        $WebhookBody = (ConvertFrom-Json -InputObject $WebhookData.RequestBody)

        # Get the info needed to identify the VM (depends on the payload schema)
        $schemaId = $WebhookBody.schemaId
        Write-Verbose "schemaId: $schemaId" -Verbose
        if ($schemaId -eq "azureMonitorCommonAlertSchema") {
            # This is the common Metric Alert schema (released March 2019)
            $Essentials = [object] ($WebhookBody.data).essentials
            # Get the first target only as this script doesn't handle multiple
            $alertTargetIdArray = (($Essentials.alertTargetIds)[0]).Split("/")
            $SubId = ($alertTargetIdArray)[2]
            $ResourceGroupName = ($alertTargetIdArray)[4]
            $ResourceType = ($alertTargetIdArray)[6] + "/" + ($alertTargetIdArray)[7]
            $ResourceName = ($alertTargetIdArray)[-1]
            $status = $Essentials.monitorCondition
        }
        elseif ($schemaId -eq "AzureMonitorMetricAlert") {
            # This is the near-real-time Metric Alert schema
            $AlertContext = [object] ($WebhookBody.data).context
            $SubId = $AlertContext.subscriptionId
            $ResourceGroupName = $AlertContext.resourceGroupName
            $ResourceType = $AlertContext.resourceType
            $ResourceName = $AlertContext.resourceName
            $status = ($WebhookBody.data).status
        }
        elseif ($schemaId -eq "Microsoft.Insights/activityLogs") {
            # This is the Activity Log Alert schema
            $AlertContext = [object] (($WebhookBody.data).context).activityLog
            $SubId = $AlertContext.subscriptionId
            $ResourceGroupName = $AlertContext.resourceGroupName
            $ResourceType = $AlertContext.resourceType
            $ResourceName = (($AlertContext.resourceId).Split("/"))[-1]
            $status = ($WebhookBody.data).status
        }
        elseif ($schemaId -eq $null) {
            # This is the original Metric Alert schema
            $AlertContext = [object] $WebhookBody.context
            $SubId = $AlertContext.subscriptionId
            $ResourceGroupName = $AlertContext.resourceGroupName
            $ResourceType = $AlertContext.resourceType
            $ResourceName = $AlertContext.resourceName
            $status = $WebhookBody.status
        }
        else {
            # Schema not supported
            Write-Error "The alert data schema - $schemaId - is not supported."
        }

        Write-Verbose "status: $status" -Verbose
        if (($status -eq "Activated") -or ($status -eq "Fired"))
        {
            Write-Verbose "resourceType: $ResourceType" -Verbose
            Write-Verbose "resourceName: $ResourceName" -Verbose
            Write-Verbose "resourceGroupName: $ResourceGroupName" -Verbose
            Write-Verbose "subscriptionId: $SubId" -Verbose

            # Determine code path depending on the resourceType
            if ($ResourceType -eq "Microsoft.Compute/virtualMachines")
            {
                # This is an Resource Manager VM
                Write-Verbose "This is an Resource Manager VM." -Verbose

                # Authenticate to Azure with service principal and certificate and set subscription
                Write-Verbose "Authenticating to Azure with service principal and certificate" -Verbose
                $ConnectionAssetName = "AzureRunAsConnection"
                Write-Verbose "Get connection asset: $ConnectionAssetName" -Verbose
                $Conn = Get-AutomationConnection -Name $ConnectionAssetName
                if ($Conn -eq $null)
                {
                    throw "Could not retrieve connection asset: $ConnectionAssetName. Check that this asset exists in the Automation account."
                }
                Write-Verbose "Authenticating to Azure with service principal." -Verbose
                Add-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint | Write-Verbose
                Write-Verbose "Setting subscription to work against: $SubId" -Verbose
                Set-AzContext -SubscriptionId $SubId -ErrorAction Stop | Write-Verbose

                # Stop the Resource Manager VM
                Write-Verbose "Stopping the VM - $ResourceName - in resource group - $ResourceGroupName -" -Verbose
                Stop-AzVM -Name $ResourceName -ResourceGroupName $ResourceGroupName -Force
                # [OutputType(PSAzureOperationResponse")]
            }
            else {
                # ResourceType not supported
                Write-Error "$ResourceType is not a supported resource type for this runbook."
            }
        }
        else {
            # The alert status was not 'Activated' or 'Fired' so no action taken
            Write-Verbose ("No action taken. Alert status: " + $status) -Verbose
        }
    }
    else {
        # Error
        Write-Error "This runbook is meant to be started from an Azure alert webhook only."
    }
    ```

6. Нажмите кнопку **Опубликовать**, чтобы сохранить и опубликовать runbook.

## <a name="create-the-alert"></a>Создание оповещения

Оповещения используют группы действий, которые представляют собой коллекции действий, активируемых оповещением. Модули runbook — это лишь одно из многих действий, доступных для групп действий.

1. В учетной записи службы автоматизации в разделе **Мониторинг** выберите **Оповещения**.
1. Выберите **Новое правило генерации оповещений**.
1. Нажмите **Выбрать** в разделе **Ресурс**. На странице **Выбор ресурса** выберите виртуальную машину для оповещения и нажмите **Готово**.
1. В разделе **Условие** нажмите **Добавить условие**. Выберите сигнал, который следует использовать, например **Процент ЦП**, и нажмите **Готово**.
1. На странице **Настроить логику сигналов** введите **Пороговое значение** в разделе **Логика оповещения** и нажмите **Готово**.
1. В разделе **Группы действий** выберите **Создать**.
1. На странице **Добавить группу действий** присвойте группе действий имя и короткое имя.
1. Присвоение имени действию Выберите **Runbook автоматизации** для параметра "Тип действия".
1. Нажмите **Изменить сведения**. На странице **Настройка Runbook** в разделе **Источник модуля Runbook** выберите **Пользователь**.  
1. Выберите свои **подписку** и **учетную запись службы автоматизации**, затем выберите модуль runbook **Stop-AzureVmInResponsetoVMAlert**.  
1. Выберите **Да** для параметра **Включите общую схему оповещений**.
1. Нажмите кнопку **ОК**, чтобы создать группу действий.

    ![Страница добавления группы действий](./media/automation-create-alert-triggered-runbook/add-action-group.png)

    Можно использовать эту группу действий в создаваемых [оповещениях журнала действий](../azure-monitor/alerts/activity-log-alerts.md) и [оповещениях почти в реальном времени](../azure-monitor/alerts/alerts-overview.md).

1. В разделе **Сведения об оповещении** добавьте имя и описание правила генерации оповещений и нажмите **Создать правило оповещения**.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о том, как запустить модуль runbook с помощью веб-перехватчика, см. в разделе [Запуск модуля runbook с помощью веб-перехватчика](automation-webhooks.md).
* Сведения о различных способах запуска модуля runbook см. разделе [Запуск модуля runbook](./start-runbooks.md).
* Сведения о создании оповещения журнала действий см. в разделе [Создание оповещений журнала действий](../azure-monitor/alerts/activity-log-alerts.md).
* Дополнительные сведения о том, как создать оповещение почти в реальном времени, см. в разделе [Создание правила оповещения с помощью портала Azure](../azure-monitor/alerts/alerts-metric.md?toc=/azure/azure-monitor/toc.json).
* Справочник по командлетам PowerShell см. в документации по [Az.Automation](/powershell/module/az.automation).