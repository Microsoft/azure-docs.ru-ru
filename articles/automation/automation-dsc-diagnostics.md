---
title: Интеграция с журналами Azure Monitor
description: В этой статье рассказывается, как отправить данные отчетов Desired State Configuration из State Configuration службы автоматизации Azure в журналы Azure Monitor.
services: automation
ms.service: automation
ms.subservice: dsc
author: mgoedtel
ms.author: magoedte
ms.date: 11/06/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 0afe349473bcddcbf1ac35136f2991ffe82670c6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100576063"
---
# <a name="integrate-with-azure-monitor-logs"></a>Интеграция с журналами Azure Monitor

В конфигурации State Configuration службы автоматизации Azure данные о состоянии узла хранятся в течение 30 дней. Вы можете отправить данные о состоянии узла в рабочую область Log Analytics, если предпочитаете хранить эти данные в течение более длительного периода. Сведения о состоянии соответствий узлов и отдельных ресурсов DSC в конфигурации узла можно просмотреть на портале Azure или с помощью PowerShell. 

Журналы Azure Monitor предоставляют расширенный оперативный контроль над данными Automation State Configuration и позволяют быстрее устранять инциденты. С помощью журналов Azure Monitor вы можете:

- получать сведения о соответствии требованиям для управляемых узлов и отдельных ресурсов;
- активировать сообщение электронной почты или предупреждение (в зависимости от состояния соответствия);
- создавать сложные запросы к управляемым узлам;
- сопоставлять состояние соответствия в учетных записях службы автоматизации;
- с помощью пользовательских представлений и поисковых запросов визуализировать результаты модуля Runbook, состояние задания Runbook и другие связанные ключевые индикаторы или метрики.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Предварительные требования

Чтобы начать отправку отчетов Automation State Configuration в журналы Azure Monitor, необходимо следующее.

- Выпуск за ноябрь 2016 года или более поздний выпуск [Azure PowerShell](/powershell/azure/) (вер. 2.3.0).
- Учетная запись службы автоматизации Azure. Дополнительные сведения см. в статье [Общие сведения о службе автоматизации Azure](automation-intro.md).
- Рабочая область Log Analytics с предложением службы "Автоматизация и управление". Дополнительные сведения см. в статье [Начало работы с Log Analytics в Azure Monitor](../azure-monitor/logs/log-analytics-tutorial.md).
- Как минимум один узел службы "Настройка состояния службы автоматизации Azure". Дополнительные сведения см. в статье [Подключение компьютеров для управления с помощью State Configuration службы автоматизации Azure](automation-dsc-onboarding.md).
- Модуль [xDscDiagnostics](https://www.powershellgallery.com/packages/xDscDiagnostics/2.7.0.0), версия 2.7.0.0 или более поздняя. Инструкции по установке см. в разделе [Устранение неполадок с Desired State Configuration службы автоматизации Azure](./troubleshoot/desired-state-configuration.md).

## <a name="set-up-integration-with-azure-monitor-logs"></a>Настройка интеграции с журналами Azure Monitor

Чтобы начать импорт данных из State Configuration службы автоматизации Azure в журналы Azure Monitor, выполните следующие действия.

1. Войдите в свою учетную запись Azure в PowerShell. См. [Вход с помощью Azure PowerShell](/powershell/azure/authenticate-azureps).
1. Получите идентификатор ресурса учетной записи службы автоматизации, выполнив следующий командлет PowerShell. Если у вас есть несколько учетных записей службы автоматизации, выберите идентификатор ресурса для учетной записи, которую требуется настроить.

   ```powershell
   # Find the ResourceId for the Automation account
   Get-AzResource -ResourceType 'Microsoft.Automation/automationAccounts'
   ```

1. Получите идентификатор ресурса рабочей области Log Analytics, выполнив следующий командлет PowerShell. Если у вас несколько рабочих областей, выберите идентификатор ресурса для рабочей области, которую требуется настроить.

   ```powershell
   # Find the ResourceId for the Log Analytics workspace
   Get-AzResource -ResourceType 'Microsoft.OperationalInsights/workspaces'
   ```

1. Выполните следующий командлет PowerShell, заменив `<AutomationResourceId>` и `<WorkspaceResourceId>` значениями `ResourceId` из предыдущих шагов.

   ```powershell
   Set-AzDiagnosticSetting -ResourceId <AutomationResourceId> -WorkspaceId <WorkspaceResourceId> -Enabled $true -Category 'DscNodeStatus'
   ```

1. Если вы хотите остановить импорт данных из State Configuration службы автоматизации Azure в журналах Azure Monitor, выполните следующий командлет PowerShell.

   ```powershell
   Set-AzDiagnosticSetting -ResourceId <AutomationResourceId> -WorkspaceId <WorkspaceResourceId> -Enabled $false -Category 'DscNodeStatus'
   ```

## <a name="view-the-state-configuration-logs"></a>Просмотр журналов State Configuration

После настройки интеграции с журналами Azure Monitor для данных State Configuration службы автоматизации их можно просмотреть, выбрав **Журналы** в разделе **Мониторинг** в левой области страницы State Configuration (DSC).

![Журналы](media/automation-dsc-diagnostics/automation-dsc-logs-toc-item.png)

Откроется панель поиска по журналам с областью запроса, заданной для ресурса учетной записи службы автоматизации. Можно выполнять поиск в журналах State Configuration для операций DSC, выполняя поиск по журналам Azure Monitor. Записи для операций DSC хранятся в таблице `AzureDiagnostics`. Например, чтобы найти несоответствующие узлы, введите следующий запрос.

```AzureDiagnostics
| where Category == 'DscNodeStatus' 
| where OperationName contains 'DSCNodeStatusData'
| where ResultType != 'Compliant'
```

Сведения о фильтрации:

* Выполните фильтрацию по `DscNodeStatusData`, чтобы вернуть операции для каждого узла State Configuration.
* Выполните фильтрацию по `DscResourceStatusData`, чтобы вернуть операции для каждого ресурса DSC, который вызывается в конфигурации узла, примененной к этому ресурсу. 
* Выполните фильтрацию по `DscResourceStatusData`, чтобы получить сведения об ошибке для всех ресурсов DSC, которые не удалось выполнить.

Дополнительные сведения о создании запросов журналов для поиска данных см. в разделе [Обзор запросов журналов в Azure Monitor](../azure-monitor/logs/log-query-overview.md).

### <a name="send-an-email-when-a-state-configuration-compliance-check-fails"></a>Отправка сообщения электронной почты при сбое проверки соответствия State Configuration

Наши клиенты часто интересуются возможностью отправки электронного сообщения или SMS-сообщения при возникновении ошибки в конфигурации DSC.

Чтобы создать правило генерации оповещений, начните с создания поиска журналов для записей отчетов State Configuration, которые должны вызывать оповещение. Щелкните кнопку **+ Новое правило генерации оповещений**, чтобы создать и настроить правило генерации оповещений.

1. На странице обзора рабочей области Log Analytics щелкните **Журналы**.
1. Поищите по журналам свое оповещение при помощи следующего поискового запроса: `Type=AzureDiagnostics Category='DscNodeStatus' NodeName_s='DSCTEST1' OperationName='DscNodeStatusData' ResultType='Failed'`

   Если вы настроили для рабочей области журналы из более чем одной учетной записи службы автоматизации или подписки, то можете группировать оповещения по подписке или учетной записи службы автоматизации. Создайте имя учетной записи службы автоматизации из поля `Resource` в поиске записей `DscNodeStatusData`.
1. Чтобы открыть экран **Создать правило**, щелкните **+ Новое правило генерации оповещений** в верхней части страницы. 

Дополнительные сведения о параметрах для настройки оповещения см. в статье [Создание правила генерации оповещений](../azure-monitor/alerts/alerts-metric.md).

### <a name="find-failed-dsc-resources-across-all-nodes"></a>Поиск ресурсов DSC со сбоями по всем узлам

Одно из преимуществ журналов Azure Monitor — возможность поиска проверок со сбоями по узлам. Чтобы найти все экземпляры ресурсов DSC со сбоями, выполните следующие действия.

1. На странице обзора рабочей области Log Analytics щелкните **Журналы**.
1. Поищите по журналам свое оповещение при помощи следующего поискового запроса: `Type=AzureDiagnostics Category='DscNodeStatus' OperationName='DscResourceStatusData' ResultType='Failed'`

### <a name="view-historical-dsc-node-status"></a>Просмотр состояния узла DSC за предыдущие периоды

Визуализируйте изменение состояния узла DSC с течением времени с помощью следующего запроса:

`Type=AzureDiagnostics ResourceProvider="MICROSOFT.AUTOMATION" Category=DscNodeStatus NOT(ResultType="started") | measure Count() by ResultType interval 1hour`

Запрос отображает диаграмму состояния узла с течением времени.

## <a name="azure-monitor-logs-records"></a>Записи журналов Azure Monitor

При диагностике службы автоматизации Azure в журналах Azure Monitor создаются записи двух категорий:

* Данные о состоянии узла (`DscNodeStatusData`)
* Данные о состоянии ресурса (`DscResourceStatusData`)

### <a name="dscnodestatusdata"></a>DscNodeStatusData

| Свойство | Описание |
| --- | --- |
| TimeGenerated |Дата и время запуска проверки соответствия. |
| OperationName |`DscNodeStatusData`. |
| ResultType |Значение, указывающее, соответствует ли узел требованиям. |
| NodeName_s |Имя управляемого узла. |
| NodeComplianceStatus_s |Значение состояния, указывающее, соответствует ли узел требованиям. |
| DscReportStatus |Значение состояния, указывающее, успешно ли выполнена проверка соответствия. |
| ConfigurationMode | Режим, используемый для применения конфигурации к узлу. Возможны следующие значения: <ul><li>`ApplyOnly`: DSC применяет конфигурацию и не выполняет дальнейшие действия, пока новая конфигурация не будет отправлена в целевой узел или получена с сервера. После первоначального применения новой конфигурации DSC не проверяет наличие отклонений от ранее настроенного состояния. Перед вступлением в силу значения `ApplyOnly` DSC пытается применить конфигурацию, пока это не будет успешно реализовано. </li><li>`ApplyAndMonitor`: Это значение по умолчанию. LCM применяет любую новую конфигурацию. Если после начального применения новой конфигурации состояние целевого узла отклоняется от требуемого, DSC сообщает о расхождении в журналах. Перед вступлением в силу значения `ApplyAndMonitor` DSC пытается применить конфигурацию, пока это не будет успешно реализовано.</li><li>`ApplyAndAutoCorrect`: DSC применяет все новые конфигурации. Если после начального применения новой конфигурации состояние целевого узла отклоняется от требуемого, DSC сообщает о расхождении в журналах, а затем повторно применяет текущую конфигурацию.</li></ul> |
| HostName_s | Имя управляемого узла. |
| IPAddress | Адрес IPv4 управляемого узла. |
| Категория | `DscNodeStatus`. |
| Ресурс | Имя учетной записи службы автоматизации Azure. |
| Tenant_g | GUID, идентифицирующий клиента для вызывающего объекта. |
| NodeId_g | Идентификатор GUID для определения управляемого узла. |
| DscReportId_g | Идентификатор GUID для определения отчета. |
| LastSeenTime_t | Дата и время последнего просмотра отчета. |
| ReportStartTime_t | Дата и время начала отчета. |
| ReportEndTime_t | Дата и время завершения отчета. |
| NumberOfResources_d | Количество ресурсов DSC, которые вызывались в примененной к узлу конфигурации. |
| SourceSystem | Исходная система, определяющая, как журналы Azure Monitor собирают данные. Всегда имеет значение `Azure` для системы диагностики Azure. |
| ResourceId |Идентификатор ресурса учетной записи службы автоматизации Azure. |
| ResultDescription | Описание ресурса для этой операции. |
| SubscriptionId | Идентификатор подписки Azure (GUID) для учетной записи службы автоматизации. |
| ResourceGroup | Имя группы ресурсов для учетной записи службы автоматизации. |
| ResourceProvider | MICROSOFT.AUTOMATION. |
| ResourceType | AUTOMATIONACCOUNTS. |
| CorrelationId | Идентификатор GUID для корреляции отчета о соответствии. |

### <a name="dscresourcestatusdata"></a>DscResourceStatusData

| Свойство | Описание |
| --- | --- |
| TimeGenerated |Дата и время запуска проверки соответствия. |
| OperationName |`DscResourceStatusData`.|
| ResultType |Подтверждено ли соответствие ресурса. |
| NodeName_s |Имя управляемого узла. |
| Категория | DscNodeStatus. |
| Ресурс | Имя учетной записи службы автоматизации Azure. |
| Tenant_g | GUID, идентифицирующий клиента для вызывающего объекта. |
| NodeId_g |Идентификатор GUID для определения управляемого узла. |
| DscReportId_g |Идентификатор GUID для определения отчета. |
| DscResourceId_s |Имя для экземпляра ресурса DSC. |
| DscResourceName_s |Имя ресурса DSC. |
| DscResourceStatus_s |Подтверждено ли соответствие ресурса DSC. |
| DscModuleName_s |Имя модуля PowerShell, который содержит ресурс DSC. |
| DscModuleVersion_s |Версия модуля PowerShell, который содержит ресурс DSC. |
| DscConfigurationName_s |Имя примененной к узлу конфигурации. |
| ErrorCode_s | Код ошибки при сбое ресурса. |
| ErrorMessage_s |Сообщение об ошибке при сбое ресурса. |
| DscResourceDuration_d |Период выполнения ресурса DSC в секундах. |
| SourceSystem | Как журналы Azure Monitor собрали данные. Всегда имеет значение `Azure` для системы диагностики Azure. |
| ResourceId |Идентификатор учетной записи службы автоматизации Azure. |
| ResultDescription | Описание для этой операции. |
| SubscriptionId | Идентификатор подписки Azure (GUID) для учетной записи службы автоматизации. |
| ResourceGroup | Имя группы ресурсов для учетной записи службы автоматизации. |
| ResourceProvider | MICROSOFT.AUTOMATION. |
| ResourceType | AUTOMATIONACCOUNTS. |
| CorrelationId |Идентификатор GUID для корреляции отчета о соответствии. |

## <a name="next-steps"></a>Дальнейшие действия

- Общие сведения см. в статье [Обзор State Configuration службы автоматизации Azure](automation-dsc-overview.md).
- Чтобы приступить к работе со службой State Configuration службы автоматизации Azure, см. сведения в [этой статье](automation-dsc-getting-started.md).
- Сведения о компилировании конфигураций DSC, которые затем можно назначить целевым узлам, см. в статье [Компилирование конфигураций DSC в службе State Configuration службы автоматизации Azure](automation-dsc-compile.md).
- Справочник по командлетам PowerShell см. в документации по [Az.Automation](/powershell/module/az.automation).
- Сведения о ценах см. на странице [с расценками для службы State Configuration службы автоматизации Azure](https://azure.microsoft.com/pricing/details/automation/).
- Пример использования службы State Configuration в службе автоматизации Azure в конвейере непрерывного развертывания см. в разделе [Настройка непрерывного развертывания с помощью Chocolatey](automation-dsc-cd-chocolatey.md).
- Чтобы узнать больше о том, как создавать различные поисковые запросы и просматривать журналы заданий State Configuration службы автоматизации с помощью журналов Azure Monitor, ознакомьтесь со статьей [Основные сведения о поисках по журналам Azure Monitor](../azure-monitor/logs/log-query-overview.md).
- Чтобы узнать больше о журналах Azure Monitor и источниках сбора данных, ознакомьтесь с разделом [Обзор сбора данных о службе хранилища Azure в журналах Azure Monitor](../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace).