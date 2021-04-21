---
title: Архивация и отчетность с помощью Azure Monitor — управление правами Azure AD
description: Узнайте, как архивировать журналы и создавать отчеты с помощью Azure Monitor в управлении правами Azure Active Directory.
services: active-directory
documentationCenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 12/23/2020
ms.author: barclayn
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 354805b3e2b538c92ba2345df2bcd93968640068
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107764393"
---
# <a name="archive-logs-and-reporting-on-azure-ad-entitlement-management-in-azure-monitor"></a>Архивируйте журналы и создавайте отчеты в управлении правами Azure AD в Azure Monitor

Azure AD хранит события аудита в журнале аудита до 30 дней. При этом, вы можете хранить данные аудита дольше указанных по умолчанию сроков хранения, указанных в статье [Как долго Azure AD хранит данные отчетов?](../reports-monitoring/reference-reports-data-retention.md), отправив их в учетную запись службы хранилища Azure или используя Azure Monitor. Затем вы можете использовать книги и пользовательские запросы и отчеты для этих данных.


## <a name="configure-azure-ad-to-use-azure-monitor"></a>Настройка Azure AD для использования Azure Monitor
Прежде чем использовать книги Azure Monitor, необходимо настроить Azure AD для отправки копий журналов аудита в Azure Monitor.

Для архивации журналов аудита Azure AD требуется, чтобы у вас был Azure Monitor в подписке Azure. Дополнительные сведения о необходимых условиях и ожидаемых затратах на использование Azure Monitor см. в статье [Журналы действий Azure AD в Azure Monitor](../reports-monitoring/concept-activity-logs-azure-monitor.md).

**Требуемая роль**: глобальный администратор.

1. Войдите на портал Azure как глобальный администратор. Убедитесь, что у вас есть доступ к группе ресурсов, в которой содержится рабочая область Azure Monitor.
 
1. Выберите **Azure Active Directory** затем щелкните **Параметры диагностики** в разделе "Мониторинг" в меню навигации слева. Проверьте, существует ли уже параметр для отправки журналов аудита в эту рабочую область.

1. Если этого параметра еще нет, щелкните **Добавить параметр диагностики**. Используйте инструкции из статьи [Интеграция журналов Azure AD с Azure Monitor журналами](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md#send-logs-to-azure-monitor) для отправки журнала аудита Azure AD в рабочую область Azure Monitor.

    ![Панель параметров диагностики](./media/entitlement-management-logs-and-reporting/audit-log-diagnostics-settings.png)


1. После того как журнал отправлен в Azure Monitor, выберите **Рабочие области Log Analytics**, и выберите рабочую область в которой содержатся журналы аудита Azure AD.

1. Выберите **Использование и ожидаемые затраты** и щелкните **Хранение данных**. С помощью ползунка выберите количество дней, в течение которых будут храниться данные, чтобы они соответствовали требованиям аудита.

    ![Панель рабочих областей Log Analytics](./media/entitlement-management-logs-and-reporting/log-analytics-workspaces.png)

1. Позже, чтобы увидеть диапазон дат, хранящихся в рабочей области, можно использовать книгу *Диапазон дат архивного журнала*:  
    
    1. Выберите **Azure Active Directory** и нажмите кнопку **Книги**. 
    
    1. Разверните раздел **Устранение неполадок Azure Active Directory**  и выберите **Диапазон дат архивного журнала**. 


## <a name="view-events-for-an-access-package"></a>Просмотр событий для пакета доступа  

Чтобы просмотреть события для пакета доступа, необходимо иметь доступ к базовой рабочей области Azure Monitor (Дополнительные сведения см. в статье [Управление доступом к данным журнала и рабочим областям в Azure Monitor](../../azure-monitor/logs/manage-access.md#manage-access-using-azure-permissions)) и в одной из следующих ролей: 

- Глобальный администратор.  
- администратор безопасности;  
- Читатель сведений о безопасности  
- Читатель отчетов.  
- администратор приложений;  

Чтобы просматривать события, используйте следующую процедуру: 

1. На портале Azure выберите **Azure Active Directory**, а затем нажмите кнопку **Книги**. Если у вас только одна подписка, переходите к 3 шагу: 

1. Если у вас несколько подписок, выберите ту, в которой есть рабочая область.  

1. Выберите книгу, которая называется *Действие пакета для доступа*. 

1. В этой книге выберите диапазон времени (измените значение на **Все**, если не уверены), и выберите идентификатор пакета доступа из раскрывающегося списка всех пакетов доступа, в которых была активность в течении этого интервала времени. Отобразятся события, связанные с пакетом доступа, который появлялся в течении выбранного интервала времени.  

    ![Просмотр событий пакета доступа](./media/entitlement-management-logs-and-reporting/view-events-access-package.png) 

    Каждая строка включает время, доступ к идентификатору пакета, имя операции, идентификатор объекта, имя субъекта-пользователя и отображаемое имя пользователя, запустившего операцию.  Дополнительные сведения содержатся в нотации объектов JavaScript   


## <a name="create-custom-azure-monitor-queries-using-the-azure-portal"></a>Создавайте пользовательские запросы Azure Monitor с помощью портала Azure
Вы можете создавать собственные запросы к событиям аудита Azure AD, включая события управления правами.  

1. В Azure Active Directory портала Azure щелкните **Журналы** в разделе Мониторинг в левом меню навигации, чтобы создать новую страницу запроса.

1. Ваша рабочая область должна отображаться в левом верхнем углу страницы запроса. Если у вас несколько рабочих областей Azure Monitor, а рабочая область, которую вы используете для хранения событий аудита Azure AD, не отображается, нажмите **Выбрать область**. Затем выберите нужную подписку и рабочую область.

1. Затем в текстовом поле запроса удалите строку "search *" и замените ее следующим запросом:

    ```
    AuditLogs | where Category == "EntitlementManagement"
    ```

1. Затем нажмите **Запуск**. 

    ![Для запуска запроса, нажмите кнопку "Запуск".](./media/entitlement-management-logs-and-reporting/run-query.png)

В таблице будут отображаться события журнала аудита для управления правами за последний час по умолчанию. Вы можете изменить параметр "Диапазон времени", чтобы просмотреть старые события. Однако при изменении этого параметра будут показаны только те события, которые произошли после того как Azure AD настроили для отправки событий в Azure Monitor.

Если вы хотите получить сведения о самых старых и новых событиях аудита, которые хранятся в Azure Monitor, используйте следующий запрос:

```
AuditLogs | where TimeGenerated > ago(3653d) | summarize OldestAuditEvent=min(TimeGenerated), NewestAuditEvent=max(TimeGenerated) by Type
```

Дополнительные сведения о столбцах, которые хранятся для событий аудита в Azure Monitor, см. в статье [Интерпретации схемы журналов аудита Azure AD в Azure Monitor](../reports-monitoring/reference-azure-monitor-audit-log-schema.md).

## <a name="create-custom-azure-monitor-queries-using-azure-powershell"></a>Создание пользовательских запросов Azure Monitor с помощью Azure PowerShell

Вы можете получить доступ к журналам через PowerShell после того, как настроите Azure AD для отправки журналов в Azure Monitor. После этого вы можете отправлять запросы из скриптов или из командной строки PowerShell без необходимости быть глобальным администратором в клиенте. 

### <a name="ensure-the-user-or-service-principal-has-the-correct-role-assignment"></a>Убедитесь, что пользователь или субъект-служба имеют правильное назначение ролей.

Убедитесь, что пользователь или субъект-служба, которые будут проходить аутентификацию в Azure AD, находятся в соответствующей роли Azure в рабочей области Log Analytics. Роли могут быть либо читатель Log Analytics или участник Log Analytics. Если вы уже используете одну из этих ролей, перейдите к [Получить идентификатор Log Analytics с помощью одной подписки Azure](#retrieve-log-analytics-id-with-one-azure-subscription).

Чтобы задать назначение ролей и создать запрос, выполните следующие действия:

1. На портале Azure перейдите в [рабочую область Log Analytics](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.OperationalInsights%2Fworkspaces
).

1. Выберите **Управление доступом (IAM)** .

1. Щелкните **Добавить**, чтобы добавить назначение ролей.

    ![Добавление назначения роли](./media/entitlement-management-logs-and-reporting/workspace-set-role-assignment.png)

### <a name="install-azure-powershell-module"></a>Установите модуль Azure PowerShell.

Получив необходимое назначение ролей, запустите PowerShell и [установите модуль Azure PowerShell](/powershell/azure/install-az-ps) (если вы еще этого не сделали), введя:

```azurepowershell
install-module -Name az -allowClobber -Scope CurrentUser
```
    
Теперь вы можете пройти аутентификацию в Azure AD и получить идентификатор рабочей области Log Analytics на которую выполняется запрос.

### <a name="retrieve-log-analytics-id-with-one-azure-subscription"></a>Получите идентификатора Log Analytics с одной подпиской Azure
Если у вас есть только одна подписка Azure и одна рабочая область Log Analytics, введите следующую команду для аутентификации в Azure AD, подключитесь к этой подписке и получите рабочую область:
 
```azurepowershell
Connect-AzAccount
$wks = Get-AzOperationalInsightsWorkspace
```
 
### <a name="retrieve-log-analytics-id-with-multiple-azure-subscriptions"></a>Получение идентификатора Log Analytics с несколькими подписками Azure

 Командлет [Get-AzOperationalInsightsWorkspace](/powershell/module/Az.OperationalInsights/Get-AzOperationalInsightsWorkspace) работает в одной подписке за раз. Таким образом, если у вас несколько подписок Azure, вам необходимо подключиться к той, у которой есть рабочая область Log Analytics с журналами Azure AD. 
 
 Следующие командлеты выводят список подписок и находят идентификатор подписки у которой есть рабочая область Log Analytics:
 
```azurepowershell
Connect-AzAccount
$subs = Get-AzSubscription
$subs | ft
```
 
Вы можете повторно пройти аутентификацию и связать сеанс PowerShell с этой подпиской с помощью команды, например `Connect-AzAccount –Subscription $subs[0].id`. Дополнительные сведения о аутентификации в Azure с помощью PowerShell, в том числе в неинтерактивном режиме, см. в разделе [Войти используя Azure PowerShell](/powershell/azure/authenticate-azureps).

Если в этой подписке имеется несколько рабочих областей Log Analytics, командлет [Get-AzOperationalInsightsWorkspace](/powershell/module/Az.OperationalInsights/Get-AzOperationalInsightsWorkspace) возвращает список рабочих областей. Затем вы можете найти тот, у которого есть журналы Azure AD. `CustomerId`Поле, возвращаемое этим командлетом, совпадает со значением "идентификатор рабочей области", отображаемым в портал Azure в обзоре рабочей области Log Analytics.
 
```powershell
$wks = Get-AzOperationalInsightsWorkspace
$wks | ft CustomerId, Name
```

### <a name="send-the-query-to-the-log-analytics-workspace"></a>Отправьте запрос в рабочую область Log Analytics
Наконец, после определения рабочей области можно использовать [Invoke-AzOperationalInsightsQuery](/powershell/module/az.operationalinsights/Invoke-AzOperationalInsightsQuery), чтобы отправить запрос Kusto в эту рабочую область. Эти запросы написаны на [языке запросов Kusto](/azure/kusto/query/).
 
Например, можно получить диапазон дат записей событий аудита из рабочей области Log Analytics, используя командлеты PowerShell для отправки запроса:
 
```powershell
$aQuery = "AuditLogs | where TimeGenerated > ago(3653d) | summarize OldestAuditEvent=min(TimeGenerated), NewestAuditEvent=max(TimeGenerated) by Type"
$aResponse = Invoke-AzOperationalInsightsQuery -WorkspaceId $wks[0].CustomerId -Query $aQuery
$aResponse.Results |ft
```

Вы также можете получать события управления правами с помощью запроса, например:

```azurepowershell
$bQuery = 'AuditLogs | where Category == "EntitlementManagement"'
$bResponse = Invoke-AzOperationalInsightsQuery -WorkspaceId $wks[0].CustomerId -Query $Query
$bResponse.Results |ft 
```

## <a name="next-steps"></a>Дальнейшие действия:
- [Создавайте интерактивные отчеты с книгами Azure Monitor](../../azure-monitor/visualize/workbooks-overview.md)
