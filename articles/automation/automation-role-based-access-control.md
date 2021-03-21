---
title: Управление разрешениями для ролей и безопасностью в службе автоматизации Azure
description: В этой статье описывается, как использовать управление доступом на основе ролей Azure (Azure RBAC), которое обеспечивает управление доступом для ресурсов Azure.
keywords: автоматизация RBAC, контроль доступа на основе ролей, RBAC Azure
services: automation
ms.subservice: shared-capabilities
ms.date: 07/21/2020
ms.topic: conceptual
ms.openlocfilehash: 320668f9596376cf7aa12ed97872671404a07658
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98895923"
---
# <a name="manage-role-permissions-and-security"></a>Управление разрешениями ролей и безопасностью

Управление доступом на основе ролей Azure (Azure RBAC) обеспечивает управление доступом для ресурсов Azure. С помощью [Azure RBAC](../role-based-access-control/overview.md)можно разделить обязанности внутри группы и предоставить доступ к пользователям, группам и приложениям, которые им необходимы для выполнения своих задач. Доступ на основе ролей можно предоставлять пользователям с помощью портала Azure, средств командной строки Azure и API управления Azure.

## <a name="roles-in-automation-accounts"></a>Роли в учетных записях службы автоматизации

В службе автоматизации Azure доступ предоставляется путем назначения соответствующей роли Azure пользователям, группам и приложениям в области учетной записи службы автоматизации. Ниже перечислены встроенные роли, поддерживаемые учетной записью автоматизации.

| **Роль** | **Описание** |
|:--- |:--- |
| Владелец |Роль владельца обеспечивает доступ ко всем ресурсам и действиям в учетной записи службы автоматизации, включая предоставление доступа на управление учетной записью службы автоматизации другим пользователям, группам и приложениям. |
| Участник |Роль участника позволяет управлять всем, кроме изменения разрешений других пользователей на доступ к учетной записи службы автоматизации. |
| Читатель |Роль читателя позволяет просматривать все ресурсы в учетной записи службы автоматизации, но не дает возможность вносить какие-либо изменения. |
| Оператор службы автоматизации |Роль оператора службы автоматизации позволяет просматривать имя и свойства модуля runbook, а также создавать задания и управлять ими для всех модулей runbook в учетной записи службы автоматизации. Эта роль пригодится, если вам нужно защитить ресурсы учетной записи службы автоматизации, такие как учетные данные, активы и runbook, от просмотра и изменения, и в то же время разрешить членам организации выполнять эти runbook. |
|Оператор заданий службы автоматизации|Роль оператора заданий службы автоматизации позволяет создавать задания и управлять ими для всех модулей runbook в учетной записи службы автоматизации.|
|Оператор Runbook службы автоматизации|Роль оператора runbook службы автоматизации позволяет просматривать имя и свойства модуля runbook.|
| участник Log Analytics. | Участник Log Analytics может считывать все данные мониторинга и изменять его параметры. Изменение параметров мониторинга подразумевает добавление расширений в виртуальные машины, чтение ключей учетной записи хранения для настройки коллекции журналов в службе хранилища Azure, создание и настройку учетных записей службы автоматизации, добавление возможностей службы автоматизации Azure и настройку диагностики Azure во всех ресурсах Azure.|
| читатель Log Analytics; | Читатель Log Analytics может просматривать параметры мониторинга, выполнять поиск всех данных мониторинга и просматривать эти данные. Читатель также может просматривать конфигурации диагностики Azure на всех ресурсах Azure. |
| Monitoring Contributor | Участник мониторинга может считывать все данные мониторинга и изменять его параметры.|
| Роль Monitoring Reader | Читатель мониторинга может считывать все данные мониторинга. |
| Администратор доступа пользователей |Роль администратора доступа пользователей позволяет управлять доступом пользователей к учетным записям службы автоматизации Azure. |

## <a name="role-permissions"></a>Разрешения ролей

В следующих таблицах описываются разрешения, предоставленные каждой роли. Это могут быть свойства Actions, которые предоставляют разрешения, и свойства NotActions, которые их ограничивают.

### <a name="owner"></a>Владелец

Владелец может управлять всем, включая доступ. В следующей таблице показаны разрешения, предоставленные для этой роли.

|Действия|Описание|
|---|---|
|Microsoft.Automation/automationAccounts/|Создание ресурсов всех типов и управление ими.|

### <a name="contributor"></a>Участник

Участник может управлять всем, кроме доступа. В следующей таблице показаны разрешения, которые предоставлены и запрещены для этой роли.

|**Действия**  |**Описание**  |
|---------|---------|
|Microsoft.Automation/automationAccounts/|Создание ресурсов всех типов и управление ими|
|**Запрещенные действия**||
|Microsoft.Authorization/*/Delete| Удаление и назначение ролей.       |
|Microsoft.Authorization/*/Write     |  Создание и назначение ролей.       |
|Microsoft.Authorization/elevateAccess/Action    | Запрет создания администратора доступа пользователей.       |

### <a name="reader"></a>Читатель

Читатель может просматривать все ресурсы в учетной записи службы автоматизации, но не может вносить какие-либо изменения.

|**Действия**  |**Описание**  |
|---------|---------|
|Microsoft.Automation/automationAccounts/read|Просмотр всех ресурсов в учетной записи службы автоматизации. |

### <a name="automation-operator"></a>Оператор службы автоматизации

Оператор службы автоматизации может создавать задания и управлять ими, а также просматривать имена и свойства всех модулей runbook в учетной записи службы автоматизации.

>[!NOTE]
>Если вы хотите управлять доступом оператора к отдельным модулям Runbook, не устанавливайте эту роль. Вместо этого используйте роли оператора **задания службы автоматизации** и **оператора автоматизации Runbook** в сочетании.

В следующей таблице показаны разрешения, предоставленные для этой роли.

|**Действия**  |**Описание**  |
|---------|---------|
|Microsoft.Authorization/*/read|Авторизация на чтение.|
|Microsoft.Automation/automationAccounts/hybridRunbookWorkerGroups/read|Считывает ресурсы гибридной рабочей роли Runbook.|
|Microsoft.Automation/automationAccounts/jobs/read|Вывод списка заданий Runbook.|
|Microsoft.Automation/automationAccounts/jobs/resume/action|Возобновление приостановленного задания.|
|Microsoft.Automation/automationAccounts/jobs/stop/action|Отмена выполняющегося задания.|
|Microsoft.Automation/automationAccounts/jobs/streams/read|Чтение потоков и выходных данных задания.|
|Microsoft.Automation/automationAccounts/jobs/output/read|Возвращает выходные данные задания.|
|Microsoft.Automation/automationAccounts/jobs/suspend/action|Остановка выполняющегося задания.|
|Microsoft.Automation/automationAccounts/jobs/write|Создание заданий.|
|Microsoft.Automation/automationAccounts/jobSchedules/read|Возвращает расписание заданий службы автоматизации Azure.|
|Microsoft.Automation/automationAccounts/jobSchedules/write|Создает расписание заданий службы автоматизации Azure.|
|Microsoft.Automation/automationAccounts/linkedWorkspace/read|Возвращает рабочую область, связанную с учетной записью службы автоматизации.|
|Microsoft.Automation/automationAccounts/read|Возвращает учетную запись службы автоматизации Azure.|
|Microsoft.Automation/automationAccounts/runbooks/read|Возвращает runbook службы автоматизации Azure.|
|Microsoft.Automation/automationAccounts/schedules/read|Возвращает ресурс расписания службы автоматизации Azure.|
|Microsoft.Automation/automationAccounts/schedules/write|Создает или обновляет ресурс расписания службы автоматизации Azure.|
|Microsoft.Resources/subscriptions/resourceGroups/read      |Чтение ролей и их назначений.         |
|Microsoft.Resources/deployments/*      |Создание развертываний группы ресурсов и управление ими.         |
|Microsoft.Insights/alertRules/*      | Создание правил оповещения и управление ими.        |
|Microsoft.Support/* |Создание запросов в службу поддержки и управление ими.|

### <a name="automation-job-operator"></a>Оператор заданий службы автоматизации

Роль оператора заданий службы автоматизации предоставляется на уровне учетной записи службы автоматизации. Она предоставляет разрешения на создание заданий и управление ими для всех модулей runbook в учетной записи. Если роли оператора задания предоставлены разрешения на чтение для группы ресурсов, содержащей учетную запись службы автоматизации, члены роли смогут запускать модули Runbook. Однако они не имеют возможности создавать, изменять и удалять их.

В следующей таблице показаны разрешения, предоставленные для этой роли.

|**Действия**  |**Описание**  |
|---------|---------|
|Microsoft.Authorization/*/read|Авторизация на чтение.|
|Microsoft.Automation/automationAccounts/jobs/read|Вывод списка заданий Runbook.|
|Microsoft.Automation/automationAccounts/jobs/resume/action|Возобновление приостановленного задания.|
|Microsoft.Automation/automationAccounts/jobs/stop/action|Отмена выполняющегося задания.|
|Microsoft.Automation/automationAccounts/jobs/streams/read|Чтение потоков и выходных данных задания.|
|Microsoft.Automation/automationAccounts/jobs/suspend/action|Остановка выполняющегося задания.|
|Microsoft.Automation/automationAccounts/jobs/write|Создание заданий.|
|Microsoft.Resources/subscriptions/resourceGroups/read      |  Чтение ролей и их назначений.       |
|Microsoft.Resources/deployments/*      |Создание развертываний группы ресурсов и управление ими.         |
|Microsoft.Insights/alertRules/*      | Создание правил оповещения и управление ими.        |
|Microsoft.Support/* |Создание запросов в службу поддержки и управление ими.|

### <a name="automation-runbook-operator"></a>Оператор Runbook службы автоматизации

Роль оператора Runbook службы автоматизации предоставляется в области Runbook. Оператор runbook службы автоматизации может просматривать имя и свойства модуля runbook.Эта роль в сочетании с ролью **оператора заданий службы автоматизации** позволяет оператору создавать задания и управлять ими для модуля Runbook. В следующей таблице показаны разрешения, предоставленные для этой роли.

|**Действия**  |**Описание**  |
|---------|---------|
|Microsoft.Automation/automationAccounts/runbooks/read     | Вывод списка Runbook.        |
|Microsoft.Authorization/*/read      | Авторизация на чтение.        |
|Microsoft.Resources/subscriptions/resourceGroups/read      |Чтение ролей и их назначений.         |
|Microsoft.Resources/deployments/*      | Создание развертываний группы ресурсов и управление ими.         |
|Microsoft.Insights/alertRules/*      | Создание правил оповещения и управление ими.        |
|Microsoft.Support/*      | Создание запросов в службу поддержки и управление ими.        |

### <a name="log-analytics-contributor"></a>участник Log Analytics.

Участник Log Analytics может читать все данные мониторинга и изменять его параметры. Изменение параметров мониторинга подразумевает добавление расширений в виртуальные машины, чтение ключей учетной записи хранения для настройки коллекции журналов в службе хранилища Azure, создание и настройку учетных записей службы автоматизации, добавление возможностей и настройку диагностики Azure для всех ресурсов Azure. В следующей таблице показаны разрешения, предоставленные для этой роли.

|**Действия**  |**Описание**  |
|---------|---------|
|*/чтение|Чтение ресурсов всех типов, кроме секретов.|
|Microsoft.Automation/automationAccounts/*|Управление учетными записями службы автоматизации.|
|Microsoft.ClassicCompute/virtualMachines/extensions/*|Создание расширений виртуальных машин и управление ими.|
|Microsoft.ClassicStorage/storageAccounts/listKeys/action|Вывод списка ключей классической учетной записи хранения.|
|Microsoft.Compute/virtualMachines/extensions/*|Создание расширений классических виртуальных машин и управление ими.|
|Microsoft.Insights/alertRules/*|Чтение, запись и удаление правила генерации оповещений.|
|Microsoft.Insights/diagnosticSettings/*|Чтение, запись и удаление параметров диагностики.|
|Microsoft.OperationalInsights/*|Управление журналами Azure Monitor.|
|Microsoft.OperationsManagement/*|Управление возможностями службы автоматизации Azure в рабочих областях.|
|Microsoft.Resources/deployments/*|Создание развертываний группы ресурсов и управление ими.|
|Microsoft.Resources/subscriptions/resourcegroups/deployments/*|Создание развертываний группы ресурсов и управление ими.|
|Microsoft.Storage/storageAccounts/listKeys/action|Составление списка ключей учетной записи хранения.|
|Microsoft.Support/*|Создание запросов в службу поддержки и управление ими.|

### <a name="log-analytics-reader"></a>читатель Log Analytics;

Читатель Log Analytics может просматривать и искать все данные мониторинга, а также просматривать его параметры, в том числе конфигурацию Диагностики Azure во всех ресурсах Azure. В следующей таблице показаны разрешения, предоставленные или запрещенные для этой роли.

|**Действия**  |**Описание**  |
|---------|---------|
|*/чтение|Чтение ресурсов всех типов, кроме секретов.|
|Microsoft.OperationalInsights/workspaces/analytics/query/action|Управление запросами в журналах Azure Monitor.|
|Microsoft.OperationalInsights/workspaces/search/action|Поиск по данным журналов Azure Monitor.|
|Microsoft.Support/*|Создание запросов в службу поддержки и управление ими.|
|**Запрещенные действия**| |
|Microsoft.OperationalInsights/workspaces/sharedKeys/read|Чтение ключей общего доступа запрещено.|

### <a name="monitoring-contributor"></a>Monitoring Contributor

Участник мониторинга может читать все данные мониторинга и изменять его параметры. В следующей таблице показаны разрешения, предоставленные для этой роли.

|**Действия**  |**Описание**  |
|---------|---------|
|*/чтение|Чтение ресурсов всех типов, кроме секретов.|
|Microsoft.AlertsManagement/alerts/*|Управление предупреждениями.|
|Microsoft.AlertsManagement/alertsSummary/*|Управление панелью мониторинга оповещений.|
|Microsoft.Insights/AlertRules/*|Управление правилами генерации оповещений.|
|Microsoft.Insights/components/*|Управление компонентами Application Insights.|
|Microsoft.Insights/DiagnosticSettings/*|Управление параметрами диагностики.|
|Microsoft.Insights/eventtypes/*|Вывод списка событий журнала действий (событий управления) в подписке. Это разрешение применяется для доступа к журналу действий посредством кода или портала.|
|Microsoft.Insights/LogDefinitions/*|Это разрешение необходимо пользователям, которым требуется доступ к журналам действия на портале. Получение списка категорий журнала в журнале действий.|
|Microsoft.Insights/MetricDefinitions/*|Чтение определений метрик (вывод списка доступных типов метрик для ресурса).|
|Microsoft.Insights/Metrics/*|Чтение метрик для ресурса.|
|Microsoft.Insights/Register/Action|Регистрация поставщика Microsoft.Insights.|
|Microsoft.Insights/webtests/*|Управление веб-тестами Application Insights.|
|Microsoft.OperationalInsights/workspaces/intelligencepacks/*|Управление пакетами решений для журналов Azure Monitor.|
|Microsoft.OperationalInsights/workspaces/savedSearches/*|Управление сохраненными поисками для журналов Azure Monitor.|
|Microsoft.OperationalInsights/workspaces/search/action|Поиск в рабочих областях Log Analytics.|
|Microsoft.OperationalInsights/workspaces/sharedKeys/action|Получение списка ключей для рабочей области Log Analytics.|
|Microsoft.OperationalInsights/workspaces/storageinsightconfigs/*|Управление конфигурациями аналитических сведений о хранилищах для журналов Azure Monitor.|
|Microsoft.Support/*|Создание запросов в службу поддержки и управление ими.|
|Microsoft.WorkloadMonitor/workloads/*|Управление рабочими нагрузками.|

### <a name="monitoring-reader"></a>Роль Monitoring Reader

Читатель мониторинга может читать все данные мониторинга. В следующей таблице показаны разрешения, предоставленные для этой роли.

|**Действия**  |**Описание**  |
|---------|---------|
|*/чтение|Чтение ресурсов всех типов, кроме секретов.|
|Microsoft.OperationalInsights/workspaces/search/action|Поиск в рабочих областях Log Analytics.|
|Microsoft.Support/*|Создание запросов в службу поддержки и управление ими|

### <a name="user-access-administrator"></a>Администратор доступа пользователей

Администратор доступа пользователей может управлять доступом пользователей к ресурсам Azure. В следующей таблице показаны разрешения, предоставленные для этой роли.

|**Действия**  |**Описание**  |
|---------|---------|
|*/чтение|Чтение всех ресурсов|
|Microsoft.Authorization/*|Управление авторизацией|
|Microsoft.Support/*|Создание запросов в службу поддержки и управление ими|

## <a name="feature-setup-permissions"></a>Разрешения на настройку возможностей

В следующих разделах показаны минимальные требуемые разрешения для включения возможностей "Управление обновлениями" и "Отслеживание изменений и инвентаризация".

### <a name="permissions-for-enabling-update-management-and-change-tracking-and-inventory-from-a-vm"></a>Разрешения на включение возможностей "Управление обновлениями" и "Отслеживание изменений и инвентаризация" для виртуальной машины Azure

|**Действие**  |**Разрешение**  |**Минимальная область**  |
|---------|---------|---------|
|Запись нового развертывания      | Microsoft.Resources/deployments/*          |Подписка          |
|Запись новой группы ресурсов      | Microsoft.Resources/subscriptions/resourceGroups/write        | Подписка          |
|Создание рабочего пространства по умолчанию      | Microsoft.OperationalInsights/workspaces/write         | Группа ресурсов         |
|Создание учетной записи      |  Microsoft.Automation/automationAccounts/write        |Группа ресурсов         |
|Связывание рабочей области и учетной записи      |Microsoft.OperationalInsights/workspaces/write</br>Microsoft.Automation/automationAccounts/read|Рабочая область</br>Учетная запись службы автоматизации
|Создание расширения MMA      | Microsoft.Compute/virtualMachines/write         | Виртуальная машина         |
|Создание сохраненного поискового запроса      | Microsoft.OperationalInsights/workspaces/write          | Рабочая область         |
|Создание конфигурации области      | Microsoft.OperationalInsights/workspaces/write          | Рабочая область         |
|Проверка состояния подключения — чтение рабочей области      | Microsoft.OperationalInsights/workspaces/read         | Рабочая область         |
|Проверка состояния подключения — чтение свойств связанной рабочей области учетной записи     | Microsoft.Automation/automationAccounts/read      | Учетная запись службы автоматизации        |
|Проверка состояния подключения — чтение решения      | Microsoft.OperationalInsights/workspaces/intelligencepacks/read          | Решение         |
|Проверка состояния подключения — чтение виртуальной машины      | Microsoft.Compute/virtualMachines/read         | Виртуальная машина         |
|Проверка состояния подключения — чтение учетной записи      | Microsoft.Automation/automationAccounts/read  |  Учетная запись службы автоматизации   |
| Проверка подключения рабочей области для виртуальной машины<sup>1</sup>       | Microsoft.OperationalInsights/workspaces/read         | Подписка         |
| Регистрация поставщика Log Analytics |Microsoft.Insights/register/action | Подписка|

<sup>1</sup> Это разрешение требуется для включения возможностей через интерфейс портала виртуальных машин.

### <a name="permissions-for-enabling-update-management-and-change-tracking-and-inventory-from-an-automation-account"></a>Разрешения на включение возможностей "Управление обновлениями" и "Отслеживание изменений и инвентаризация" для учетной записи службы автоматизации

|**Действие**  |**Разрешение** |**Минимальная область**  |
|---------|---------|---------|
|Создание развертывания     | Microsoft.Resources/deployments/*        | Подписка         |
|Создание группы ресурсов     | Microsoft.Resources/subscriptions/resourceGroups/write         | Подписка        |
|Колонка AutomationOnboarding — создание рабочей области     |Microsoft.OperationalInsights/workspaces/write           | Группа ресурсов        |
|Колонка AutomationOnboarding — чтение связанной рабочей области     | Microsoft.Automation/automationAccounts/read        | Учетная запись службы автоматизации       |
|Колонка AutomationOnboarding — чтение решения     | Microsoft.OperationalInsights/workspaces/intelligencepacks/read         | Решение        |
|Колонка AutomationOnboarding — чтение рабочей области     | Microsoft.OperationalInsights/workspaces/intelligencepacks/read        | Рабочая область        |
|Создание связи для рабочей области и учетной записи     | Microsoft.OperationalInsights/workspaces/write        | Рабочая область        |
|Запись учетной записи для Shoebox      | Microsoft.Automation/automationAccounts/write        | Учетная запись        |
|Создание или изменение сохраненного поискового запроса     | Microsoft.OperationalInsights/workspaces/write        | Рабочая область        |
|Создание или изменение конфигурации области     | Microsoft.OperationalInsights/workspaces/write        | Рабочая область        |
| Регистрация поставщика Log Analytics |Microsoft.Insights/register/action | Подписка|
|**Шаг 2. Включение нескольких виртуальных машин**     |         |         |
|Колонка VMOnboarding — создание расширения MMA     | Microsoft.Compute/virtualMachines/write           | Виртуальная машина        |
|Создание или изменение сохраненного поискового запроса     | Microsoft.OperationalInsights/workspaces/write           | Рабочая область        |
|Создание или изменение конфигурации области  | Microsoft.OperationalInsights/workspaces/write   | Рабочая область|

## <a name="update-management-permissions"></a>Разрешения на управление изменениями

Управление обновлениями действует между несколькими службами для обеспечения их обслуживания. В следующей таблице показаны разрешения, необходимые для развертываний управления обновлениями.

|**Ресурс**  |**Роль**  |**Область действия**  |
|---------|---------|---------|
|Учетная запись службы автоматизации     | участник Log Analytics.       | Учетная запись службы автоматизации        |
|Учетная запись службы автоматизации    | Участник виртуальной машины        | Группа ресурсов для учетной записи        |
|Рабочая область Log Analytics     | участник Log Analytics.| Рабочая область Log Analytics        |
|Рабочая область Log Analytics |читатель Log Analytics;| Подписка|
|Решение     |участник Log Analytics.         | Решение|
|Виртуальная машина     | Участник виртуальной машины        | Виртуальная машина        |

## <a name="configure-azure-rbac-for-your-automation-account"></a>Настройка Azure RBAC для учетной записи службы автоматизации

В следующем разделе показано, как настроить Azure RBAC в учетной записи службы автоматизации с помощью [портал Azure](#configure-azure-rbac-using-the-azure-portal) и [PowerShell](#configure-azure-rbac-using-powershell).

### <a name="configure-azure-rbac-using-the-azure-portal"></a>Настройка Azure RBAC с помощью портал Azure

1. Войдите на [портал Azure](https://portal.azure.com/) и откройте учетную запись службы автоматизации на странице "Учетные записи автоматизации".
2. Щелкните **Управление доступом (IAM)** , чтобы открыть страницу "Управление доступом". На этой странице вы можете добавлять новых пользователей, группы и приложения для управления учетной записью службы автоматизации, а также просматривать имеющиеся роли, доступные для настройки в учетной записи службы автоматизации.
3. Перейдите на вкладку **Назначения ролей**.

   ![Кнопка доступа](media/automation-role-based-access-control/automation-01-access-button.png)

#### <a name="add-a-new-user-and-assign-a-role"></a>Добавление нового пользователя и назначение роли

1. На странице "Управление доступом (IAM)" щелкните **Добавить назначение ролей**. Это действие открывает страницу "Добавление назначения роли", на которой можно добавить пользователя, группу или приложение, а также назначить им роль.

2. Выберите роль из списка доступных ролей. Вы можете выбрать любую из доступных встроенных ролей, поддерживаемых учетной записью службы автоматизации, или пользовательскую роль, которую вы создали сами.

3. Введите имя пользователя, которому вы намерены предоставить разрешения, в поле **Выбрать**. Выберите пользователя из списка и щелкните **Сохранить**.

   ![Добавление пользователей](media/automation-role-based-access-control/automation-04-add-users.png)

   Пользователь будет добавлен на страницу "Пользователи", и ему будет назначена выбранная роль.

   ![Список пользователей](media/automation-role-based-access-control/automation-05-list-users.png)

   Кроме того, роль можно назначить пользователю на странице "Роли".

4. На странице "Управление доступом (IAM)" щелкните **Роли**, чтобы открыть страницу "Роли". Здесь можно просмотреть имена ролей и число пользователей и групп, которым она назначена.

    ![Назначение роли из странице пользователей](media/automation-role-based-access-control/automation-06-assign-role-from-users-blade.png)

   > [!NOTE]
   > Управление доступом на основе ролей можно настроить только на уровне учетной записи службы автоматизации, но не в ресурсах более низкого уровня.

#### <a name="remove-a-user"></a>Удаление пользователя

Разрешение на доступ для пользователя, который не управляет учетной записью службы автоматизации или прекращает работу в организации, можно удалить. Чтобы удалить пользователя, выполните описанные ниже действия.

1. На странице "Управление доступом (IAM)" выберите пользователя, которого нужно удалить, и щелкните **Удалить**.
2. Нажмите кнопку **Удалить** в области сведений о назначении.
3. Нажмите кнопку **Да** для подтверждения удаления.

   ![Удаление пользователей](media/automation-role-based-access-control/automation-08-remove-users.png)

### <a name="configure-azure-rbac-using-powershell"></a>Настройка Azure RBAC с помощью PowerShell

Доступ к учетной записи службы автоматизации на основе ролей можно также настроить с помощью указанных ниже [командлетов Azure PowerShell](../role-based-access-control/role-assignments-powershell.md).

[Get-азроледефинитион](/powershell/module/Az.Resources/Get-AzRoleDefinition) перечисляет все роли Azure, доступные в Azure Active Directory. При использовании с параметром `Name` этот командлет возвращает список всех действий, которые может выполнить определенная роль.

```azurepowershell-interactive
Get-AzRoleDefinition -Name 'Automation Operator'
```

Ниже приведен пример выходных данных.

```azurepowershell
Name             : Automation Operator
Id               : d3881f73-407a-4167-8283-e981cbba0404
IsCustom         : False
Description      : Automation Operators are able to start, stop, suspend, and resume jobs
Actions          : {Microsoft.Authorization/*/read, Microsoft.Automation/automationAccounts/jobs/read, Microsoft.Automation/automationAccounts/jobs/resume/action,
                   Microsoft.Automation/automationAccounts/jobs/stop/action...}
NotActions       : {}
AssignableScopes : {/}
```

[Get-азролеассигнмент](/powershell/module/az.resources/get-azroleassignment) выводит список назначений ролей Azure в указанной области. Без параметров этот командлет возвращает все назначения ролей, выполненные в подписке. Указав параметр `ExpandPrincipalGroups`, вы получите список назначений доступа для указанного пользователя и всех групп, в состав которых он входит.

**Пример**. Следующий командлет выводит список всех пользователей в учетной записи службы автоматизации с указанием их ролей.

```azurepowershell-interactive
Get-AzRoleAssignment -Scope '/subscriptions/<SubscriptionID>/resourcegroups/<Resource Group Name>/Providers/Microsoft.Automation/automationAccounts/<Automation account name>'
```

Ниже приведен пример выходных данных.

```powershell
RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Automation/automationAccounts/myAutomationAccount/provid
                     ers/Microsoft.Authorization/roleAssignments/cc594d39-ac10-46c4-9505-f182a355c41f
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Automation/automationAccounts/myAutomationAccount
DisplayName        : admin@contoso.com
SignInName         : admin@contoso.com
RoleDefinitionName : Automation Operator
RoleDefinitionId   : d3881f73-407a-4167-8283-e981cbba0404
ObjectId           : 15f26a47-812d-489a-8197-3d4853558347
ObjectType         : User
```

Используйте командлет [New-AzRoleAssignment](/powershell/module/Az.Resources/New-AzRoleAssignment), чтобы присвоить права доступа к определенной области пользователям, группам и приложениям.

**Пример**. Следующая команда назначает роль оператора службы автоматизации пользователю в области учетной записи службы автоматизации.

```azurepowershell-interactive
New-AzRoleAssignment -SignInName <sign-in Id of a user you wish to grant access> -RoleDefinitionName 'Automation operator' -Scope '/subscriptions/<SubscriptionID>/resourcegroups/<Resource Group Name>/Providers/Microsoft.Automation/automationAccounts/<Automation account name>'
```

Ниже приведен пример выходных данных.

```azurepowershell
RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/myResourceGroup/Providers/Microsoft.Automation/automationAccounts/myAutomationAccount/provid
                     ers/Microsoft.Authorization/roleAssignments/25377770-561e-4496-8b4f-7cba1d6fa346
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/myResourceGroup/Providers/Microsoft.Automation/automationAccounts/myAutomationAccount
DisplayName        : admin@contoso.com
SignInName         : admin@contoso.com
RoleDefinitionName : Automation Operator
RoleDefinitionId   : d3881f73-407a-4167-8283-e981cbba0404
ObjectId           : f5ecbe87-1181-43d2-88d5-a8f5e9d8014e
ObjectType         : User
```

Командлет [Remove-AzRoleAssignment](/powershell/module/Az.Resources/Remove-AzRoleAssignment) удаляет права доступа к определенной области для указанного пользователя, группы или приложения.

**Пример**. Следующая команда удаляет пользователя из роли оператора службы автоматизации в области учетной записи службы автоматизации.

```azurepowershell-interactive
Remove-AzRoleAssignment -SignInName <sign-in Id of a user you wish to remove> -RoleDefinitionName 'Automation Operator' -Scope '/subscriptions/<SubscriptionID>/resourcegroups/<Resource Group Name>/Providers/Microsoft.Automation/automationAccounts/<Automation account name>'
```

В предыдущем примере замените `sign-in ID of a user you wish to remove`, `SubscriptionID`, `Resource Group Name` и `Automation account name` реальными данными для учетной записи. Прежде чем продолжить удаление назначений ролей пользователей, выберите **Да** в ответ на запрос подтверждения.

### <a name="user-experience-for-automation-operator-role---automation-account"></a>Взаимодействие с пользователем, который имеет роль оператора службы автоматизации, в учетной записи службы автоматизации

Когда пользователь с ролью оператора службы автоматизации в области учетной записи службы автоматизации просматривает эту учетную запись службы автоматизации, он увидит только список runbook, заданий runbook и расписаний, которые созданы в этой учетной записи службы автоматизации. Такой пользователь не может просматривать определения этих элементов. Такой пользователь может запускать, останавливать, приостанавливать, возобновлять и планировать задания runbook. Но у него нет доступа к другим ресурсам службы автоматизации, включая конфигурации, группы гибридных рабочих ролей и узлы DSC.

![Нет доступа к ресурсам](media/automation-role-based-access-control/automation-10-no-access-to-resources.png)

## <a name="configure-azure-rbac-for-runbooks"></a>Настройка Azure RBAC для модулей Runbook

Служба автоматизации Azure позволяет назначать роли Azure конкретным модулям Runbook. Для этого выполните следующий скрипт, который добавляет пользователя к определенному runbook. Этот скрипт может выполнять администратор учетной записи службы автоматизации или администратор клиента.

```azurepowershell-interactive
$rgName = "<Resource Group Name>" # Resource Group name for the Automation account
$automationAccountName ="<Automation account name>" # Name of the Automation account
$rbName = "<Name of Runbook>" # Name of the runbook
$userId = "<User ObjectId>" # Azure Active Directory (AAD) user's ObjectId from the directory

# Gets the Automation account resource
$aa = Get-AzResource -ResourceGroupName $rgName -ResourceType "Microsoft.Automation/automationAccounts" -ResourceName $automationAccountName

# Get the Runbook resource
$rb = Get-AzResource -ResourceGroupName $rgName -ResourceType "Microsoft.Automation/automationAccounts/runbooks" -ResourceName "$rbName"

# The Automation Job Operator role only needs to be run once per user.
New-AzRoleAssignment -ObjectId $userId -RoleDefinitionName "Automation Job Operator" -Scope $aa.ResourceId

# Adds the user to the Automation Runbook Operator role to the Runbook scope
New-AzRoleAssignment -ObjectId $userId -RoleDefinitionName "Automation Runbook Operator" -Scope $rb.ResourceId
```

После выполнения скрипта предложите пользователю войти на портал Azure и просмотреть раздел **Все ресурсы**. В этом списке он увидит runbook, для которых у него есть роль оператора runbook службы автоматизации.

![Runbook Azure RBAC на портале](./media/automation-role-based-access-control/runbook-rbac.png)

### <a name="user-experience-for-automation-operator-role---runbook"></a>Возможности для пользователя с ролью оператора службы автоматизации. Модуль runbook

Когда пользователь, которому назначена роль оператора службы автоматизации в области runbook, просматривает назначенный ему runbook, он может только запускать его и просматривать его задания.

![Может только запускать](media/automation-role-based-access-control/automation-only-start.png)

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения об Azure RBAC с помощью PowerShell см. в статье [Добавление и удаление назначений ролей Azure с помощью Azure PowerShell](../role-based-access-control/role-assignments-powershell.md).
* Дополнительные сведения о типах runbook см. в статье [Типы runbook в службе автоматизации Azure](automation-runbook-types.md).
* Изучите также, [как запустить runbook в службе автоматизации Azure](start-runbooks.md).
