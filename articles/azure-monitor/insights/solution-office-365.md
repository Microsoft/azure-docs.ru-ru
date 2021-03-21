---
title: Решение для управления Office 365 в Azure
description: В этой статье представлены сведения о настройке и использовании решения Office 365 в Azure.  Она содержит подробное описание записей Office 365, создаваемых в службе Azure Monitor.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/30/2020
ms.openlocfilehash: be5409f7130f0e79b77871a2657609e8ddb76e36
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101728643"
---
# <a name="office-365-management-solution-in-azure-preview"></a>Решение по управлению Office 365 в Azure (предварительная версия)

![Логотип Office 365](media/solution-office-365/icon.png)

> [!IMPORTANT]
> ## <a name="solution-update"></a>Обновление решения
> Это решение заменено решением "общедоступная доступность [Office 365](../../sentinel/connect-office-365.md) " в [Azure Sentinel](../../sentinel/overview.md) , а также [решением Azure AD Reporting and Monitoring](../../active-directory/reports-monitoring/plan-monitoring-and-reporting.md). Вместе они предоставляют обновленную версию предыдущего решения Azure Monitor Office 365 с улучшенными возможностями настройки. Вы можете продолжать использовать существующее решение до 31 октября 2020 г.
> 
> Azure Sentinel — это облачная собственная информация о безопасности и решение для управления событиями, которое принимает журналы и предоставляет дополнительные функции SIEM, в том числе обнаружения, исследования, поиска и анализа машинного обучения. С помощью Sentinel Azure вы получите возможность приема данных об активности Office 365 SharePoint и журналах управления Exchange.
> 
> Служба отчетов Azure AD предоставляет более исчерпывающее представление журналов из действия Azure AD в вашей среде, включая события входа, события аудита и изменения в каталоге. Чтобы подключить журналы Azure AD, можно использовать [Azure AD Sentinel Connector](../../sentinel/connect-azure-active-directory.md) или настроить [ИНТЕГРАЦИЮ журналов azure AD с Azure Monitor](../../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md). 
>
> Сбор данных журнала Azure AD зависит от Azure Monitor цен.  Дополнительные сведения см. в разделе [Azure Monitor цены](https://azure.microsoft.com/pricing/details/monitor/) .
>
> Чтобы использовать Azure Sentinel Office 365, выполните следующие действия.
> 1. Использование соединителя Office 365 в Azure Sentinel влияет на цены на вашу рабочую область. Дополнительные сведения см. в статье о [ценах на Sentinel Azure](https://azure.microsoft.com/pricing/details/azure-sentinel/).
> 2. Если вы уже используете решение Azure Monitor Office 365, сначала удалите его с помощью сценария в [разделе Удаление ниже](#uninstall).
> 3. [Включите решение Sentinel для Azure](../../sentinel/quickstart-onboard.md) в рабочей области.
> 4. Перейдите на страницу **соединителей данных** в Azure Sentinel и включите соединитель **Office 365** .
>
> ## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы
> 
> ### <a name="q-is-it-possible-to-on-board-the-office-365-azure-monitor-solution-between-now-and-october-31"></a>Вопрос. можно ли подключить решение Office 365 Azure Monitor между сейчас и 31 октября?
> Нет, сценарии адаптации решения Azure Monitor Office 365 больше не доступны. Решение будет удалено 31 октября.
> 
> ### <a name="q-will-the-tables-and-schemas-be-changed"></a>Вопрос. будут ли изменены таблицы и схемы?
> Имя и схема таблицы **оффицеактивити** останутся такими же, как и в текущем решении. Вы можете продолжать использовать те же запросы в новом решении, за исключением запросов, которые ссылаются на данные Azure AD.
> 
> Новые журналы [решений Azure AD Reporting и Monitoring](../../active-directory/reports-monitoring/plan-monitoring-and-reporting.md) будут применяться к таблицам [сигнинлогс](../../active-directory/reports-monitoring/concept-sign-ins.md) и [AuditLogs](../../active-directory/reports-monitoring/concept-audit-logs.md) вместо **оффицеактивити**. Дополнительные сведения см. [в статье анализ журналов Azure AD](../../active-directory/reports-monitoring/howto-analyze-activity-logs-log-analytics.md), который также относится к Azure Sentinel и Azure Monitor пользователям.
> 
> Ниже приведены примеры преобразования запросов из **оффицеактивити** в **сигнинлогс**.
> 
> **Неудачные попытки входа в запрос от пользователя:**
> 
> ```Kusto
> OfficeActivity
> | where TimeGenerated >= ago(1d) 
> | where OfficeWorkload == "AzureActiveDirectory"                      
> | where Operation == 'UserLoginFailed'
> | summarize count() by UserId    
> ```
> 
> ```Kusto
> SigninLogs
> | where ConditionalAccessStatus == "failure" or ConditionalAccessStatus == "notApplied"
> | summarize count() by UserDisplayName
> ```
> 
> **Просмотр операций Azure AD:**
> 
> ```Kusto
> OfficeActivity
> | where OfficeWorkload =~ "AzureActiveDirectory"
> | sort by TimeGenerated desc
> | summarize AggregatedValue = count() by Operation
> ```
> 
> ```Kusto
> AuditLogs
> | summarize count() by OperationName
> ```
> 
> ### <a name="q-how-can-i-on-board-azure-sentinel"></a>Вопрос. как можно настроить метку Azure?
> Azure Sentinel — это решение, которое можно включить в новой или существующей рабочей области Log Analytics. Дополнительные сведения см. [в документации по Azure Sentinel](../../sentinel/quickstart-onboard.md).
>
> ### <a name="q-do-i-need-azure-sentinel-to-connect-the-azure-ad-logs"></a>Вопрос. нужно ли мне использовать метку Azure для подключения журналов Azure AD?
> Вы можете настроить [интеграцию журналов Azure AD с Azure Monitor](../../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md), которая не связана с решением Sentinel Azure. Azure Sentinel предоставляет собственный соединитель и готовые материалы для журналов Azure AD. Дополнительные сведения см. в разделе, посвященном встроенному содержимому, ориентированному на безопасность.
>
> ###    <a name="q-what-are-the-differences-when-connecting-azure-ad-logs-from-azure-sentinel-and-azure-monitor"></a>Вопрос. Каковы различия при подключении журналов Azure AD с помощью Sentinel и Azure Monitor в Azure?
> Azure Sentinel и Azure Monitor подключаются к журналам Azure AD на основе того же [решения Azure AD Reporting и мониторинга](../../active-directory/reports-monitoring/plan-monitoring-and-reporting.md). Azure Sentinel предоставляет один и тот же собственный соединитель, который подключает те же данные и предоставляет данные мониторинга.
>
> ###    <a name="q-what-do-i-need-to-change-when-moving-to-the-new-azure-ad-reporting-and-monitoring-tables"></a>Вопрос. что нужно изменить при переходе к новым таблицам отчетов и мониторинга Azure AD?
> Все запросы, использующие данные Azure AD, включая запросы в оповещениях, панелях мониторинга и любое содержимое, созданное с помощью Office 365 Azure AD, необходимо создать повторно с помощью новых таблиц.
>
> Azure AD Sentinel и Azure Active Directory предоставляют встроенное содержимое, которое можно использовать при переходе к решению для мониторинга и отслеживания Azure AD. Дополнительные сведения см. в следующем вопросе встроенного содержимого, ориентированного на безопасность, и [об использовании Azure Monitor книг для Azure Active Directoryных отчетов](../../active-directory/reports-monitoring/howto-use-azure-monitor-workbooks.md). 
>
> ### <a name="q-how-i-can-use-the-azure-sentinel-out-of-the-box-security-oriented-content"></a>Вопрос. как можно использовать предварительно встроенное содержимое, ориентированное на Azure?
> Azure Sentinel предоставляет готовые панели мониторинга, ориентированные на безопасность, пользовательские запросы оповещений, запросы на поиск, исследование и автоматизированные возможности реагирования на основе журналов Office 365 и Azure AD. Дополнительные сведения см. в статье об Azure Sentinel GitHub и руководствах.
>
> - [Обнаружение встроенных угроз](../../sentinel/tutorial-detect-threats-built-in.md)
> - [Создание настраиваемых правил аналитики для обнаружения потенциальных угроз](../../sentinel/tutorial-detect-threats-custom.md)
> - [Мониторинг данных](../../sentinel/tutorial-monitor-your-data.md)
> - [Исследование инцидентов с помощью Azure Sentinel](../../sentinel/tutorial-investigate-cases.md)
> - [настройке автоматического реагирования на угрозы в Azure Sentinel](../../sentinel/tutorial-respond-threats-playbook.md)
> - [Сообщество Azure Sentinel GitHub](https://github.com/Azure/Azure-Sentinel/tree/master/Playbooks)
> 
> ### <a name="q-does-azure-sentinel-provide-additional-connectors-as-part-of-the-solution"></a>Вопрос. поддерживает ли Azure Sentinel дополнительные соединители в рамках решения?
> Да, см. статью [Azure Sentinel Connect Data Sources](../../sentinel/connect-data-sources.md).
> 
> ###    <a name="q-what-will-happen-on-october-31-do-i-need-to-offboard-beforehand"></a>Вопрос. что произойдет 31 октября? Нужно ли отключение заранее?
> 
> - Вы не сможете получать данные из решения **Office 365** . Решение будет удалено из рабочей области и больше не будет доступно в Marketplace.
> - Для клиентов с метками Azure **Office 365** в решение Azure Sentinel **секуритинсигхтс** будет включена log Analyticsная Рабочая область.
> - Если вы не отключение ваше решение вручную 31 октября, данные будут отключены автоматически, а таблица **оффицеактивити** удалена. Несмотря на это, вы по-прежнему сможете восстановить таблицу при включении соединителя Office 365 в Azure Sentinel, как описано ниже.
> 
> ### <a name="q-will-my-data-transfer-to-the-new-solution"></a>Вопрос. будет ли переносить данные в новое решение?
> Да. При удалении решения **Office 365** из рабочей области его данные становятся временно недоступными из-за удаления схемы. Когда вы включаете новый соединитель **Office 365** в Azure Sentinel, схема восстанавливается в рабочей области, а все собранные данные становятся доступными. 
 

Решение по управлению Office 365 позволяет выполнять мониторинг среды Office 365 в Azure Monitor.

- Отслеживать действия пользователей в учетных записях Office 365 для анализа шаблонов использования, а также определять поведенческие тенденции. Например, можно извлечь конкретные сценарии использования, такие как файлы, к которым предоставлен общий доступ за пределами организации, или самые популярные сайты SharePoint.
- Отслеживать действия администратора, чтобы контролировать изменения конфигурации или операции с высоким уровнем привилегий.
- Выявлять и анализировать нежелательное поведение пользователей, которое можно настраивать, исходя из потребностей организации.
- Выполнять аудит и проверку соответствия. Например, можно отслеживать операции доступа к файлам для конфиденциальных файлов, что поможет в процессе аудита и проверки соответствия.
- Оперативно устраняйте неполадки с помощью [запросов журнала](../logs/log-query-overview.md) на основе данных о действиях Office 365 в вашей организации.


## <a name="uninstall"></a>Удаление

Вы можете удалить решение по управлению Office 365 с помощью процесса, описанного в этом [разделе](solutions.md#remove-a-monitoring-solution). Однако это не остановит сбор данных в Office 365 и последующую их отправку в Azure Monitor. Выполните процедуру ниже, чтобы отменить подписку на Office 365 и прекратить сбор данных.

1. Сохраните следующий сценарий как *office365_unsubscribe.ps1*.

    ```powershell
    param (
        [Parameter(Mandatory=$True)][string]$WorkspaceName,
        [Parameter(Mandatory=$True)][string]$ResourceGroupName,
        [Parameter(Mandatory=$True)][string]$SubscriptionId,
        [Parameter(Mandatory=$True)][string]$OfficeTennantId
    )
    $line='#-------------------------------------------------------------------------------------------------------------------------------------------------------------------------'
    
    $line
    IF ($Subscription -eq $null)
        {Login-AzAccount -ErrorAction Stop}
    $Subscription = (Select-AzSubscription -SubscriptionId $($SubscriptionId) -ErrorAction Stop)
    $Subscription
    $option = [System.StringSplitOptions]::RemoveEmptyEntries 
    $Workspace = (Set-AzOperationalInsightsWorkspace -Name $($WorkspaceName) -ResourceGroupName $($ResourceGroupName) -ErrorAction Stop)
    $Workspace
    $WorkspaceLocation= $Workspace.Location
    
    # Client ID for Azure PowerShell
    $clientId = "1950a258-227b-4e31-a9cf-717495945fc2"
    # Set redirect URI for Azure PowerShell
    $redirectUri = "urn:ietf:wg:oauth:2.0:oob"
    $domain='login.microsoftonline.com'
    $adTenant =  $Subscription[0].Tenant.Id
    $authority = "https://login.windows.net/$adTenant";
    $ARMResource ="https://management.azure.com/";
    $xms_client_tenant_Id ='55b65fb5-b825-43b5-8972-c8b6875867c1'
    
    switch ($WorkspaceLocation) {
           "USGov Virginia" { 
                             $domain='login.microsoftonline.us';
                              $authority = "https://login.microsoftonline.us/$adTenant";
                              $ARMResource ="https://management.usgovcloudapi.net/"; break} # US Gov Virginia
           default {
                    $domain='login.microsoftonline.com'; 
                    $authority = "https://login.windows.net/$adTenant";
                    $ARMResource ="https://management.azure.com/";break} 
                    }
    
    Function RESTAPI-Auth { 
    
    $global:SubscriptionID = $Subscription.SubscriptionId
    # Set Resource URI to Azure Service Management API
    $resourceAppIdURIARM=$ARMResource;
    # Authenticate and Acquire Token 
    # Create Authentication Context tied to Azure AD Tenant
    $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList $authority
    # Acquire token
    $platformParameters = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.PlatformParameters" -ArgumentList "Auto"
    $global:authResultARM = $authContext.AcquireTokenAsync($resourceAppIdURIARM, $clientId, $redirectUri, $platformParameters)
    $global:authResultARM.Wait()
    $authHeader = $global:authResultARM.Result.CreateAuthorizationHeader()
    $authHeader
    }
    
    Function Office-UnSubscribe-Call{
    
    #----------------------------------------------------------------------------------------------------------------------------------------------
    $authHeader = $global:authResultARM.Result.CreateAuthorizationHeader()
    $ResourceName = "https://manage.office.com"
    $SubscriptionId   = $Subscription[0].Subscription.Id
    $OfficeAPIUrl = $ARMResource + 'subscriptions/' + $SubscriptionId + '/resourceGroups/' + $ResourceGroupName + '/providers/Microsoft.OperationalInsights/workspaces/' + $WorkspaceName + '/datasources/office365datasources_'  + $SubscriptionId + $OfficeTennantId + '?api-version=2015-11-01-preview'
    
    $Officeparams = @{
        ContentType = 'application/json'
        Headers = @{
        'Authorization'="$($authHeader)"
        'x-ms-client-tenant-id'=$xms_client_tenant_Id
        'Content-Type' = 'application/json'
        }
        Method = 'Delete'
        URI = $OfficeAPIUrl
      }
    
    $officeresponse = Invoke-WebRequest @Officeparams 
    $officeresponse
    
    }
    
    #GetDetails 
    RESTAPI-Auth -ErrorAction Stop
    Office-UnSubscribe-Call -ErrorAction Stop
    ```

2. Выполните скрипт с помощью следующей команды:

    ```
    .\office365_unsubscribe.ps1 -WorkspaceName <Log Analytics workspace name> -ResourceGroupName <Resource Group name> -SubscriptionId <Subscription ID> -OfficeTennantID <Tenant ID> 
    ```

    Пример

    ```powershell
    .\office365_unsubscribe.ps1 -WorkspaceName MyWorkspace -ResourceGroupName MyResourceGroup -SubscriptionId '60b79d74-f4e4-4867-b631-yyyyyyyyyyyy' -OfficeTennantID 'ce4464f8-a172-4dcf-b675-xxxxxxxxxxxx'
    ```

Вам будет предложено ввести учетные данные. Укажите учетные данные для рабочей области Log Analytics.

## <a name="data-collection"></a>Сбор данных

Для сбора данных может потребоваться несколько часов. После запуска сбора при создании каждой записи Office 365 отправляет в службу Azure Monitor [уведомление веб-перехватчика](/office/office-365-management-api/office-365-management-activity-api-reference#receiving-notifications) с подробными данными. Эта запись становится доступной в Azure Monitor в течение нескольких минут после ее получения.

## <a name="using-the-solution"></a>Использование решения

[!INCLUDE [azure-monitor-solutions-overview-page](../../../includes/azure-monitor-solutions-overview-page.md)]

Когда вы добавите решение Office 365 в рабочую область Log Analytics, плитка **Office 365** будет добавлена на вашу панель инструментов. На этой плитке отображается количество и графическое представление количества компьютеров в вашей среде и соответствие обновлений.<br><br>
![Плитка сводки Office 365](media/solution-office-365/tile.png)  

Щелкните элемент **Office 365**, чтобы открыть панель мониторинга **Office 365**.

![Панель мониторинга Office 365](media/solution-office-365/dashboard.png)  

Панель мониторинга содержит столбцы, перечисленные в приведенной ниже таблице. Каждый столбец содержит десять ведущих оповещений с числом, показывающим соответствие оповещения критериям данного столбца, таким как область действия и диапазон времени. Можно выполнить поиск журналов, выводящий весь список, щелкнув элемент "Просмотреть все" внизу столбца или заголовок этого столбца.

| Столбец | Описание |
|:--|:--|
| Операции | Предоставляет сведения об активных пользователях из всех отслеживаемых подписок Office 365. Также можно просмотреть количество действий, выполненных за определенный период времени.
| Exchange | Отображает разбивку по действиям Exchange Server, таким как Add-MailboxPermission (Добавление разрешения для почтового ящика) или Set-Mailbox (Настройка почтового ящика). |
| SharePoint | Отображает основные действия, выполняемые пользователями в отношении документов SharePoint. Если выполнить детализацию из этого элемента, то на странице поиска отобразятся сведения об этих действиях, такие как целевой документ и расположение действия. Например, для события с доступом к файлу вы сможете увидеть документ, к которому осуществляется доступ, имя связанной учетной записи и IP-адрес. |
| Azure Active Directory | Включает в себя основные действия пользователей, такие как сброс пароля пользователя и попытка входа в систему. Если выполнить детализацию, то можно просмотреть подробные сведения об этих действиях, такие как состояние результата. Это полезно, в основном, для отслеживания подозрительных действий в Azure Active Directory. |




## <a name="azure-monitor-log-records"></a>Записи журнала Azure Monitor

У всех записей, создаваемых решением Office 365 в рабочей области Log Analytics в Azure Monitor, свойство **Type** (Тип) имеет значение **OfficeActivity**.  Свойство **OfficeWorkload** определяет, к какой службе Office 365 относится запись — Exchange, Azure Active Directory, SharePoint или OneDrive.  Свойство **RecordType** указывает тип операции.  Эти свойства будут отличаться для каждого типа операции. Их описание приводится в таблице ниже.

### <a name="common-properties"></a>Общие свойства

Следующие свойства являются общими для всех записей Office 365.

| Свойство. | Описание |
|:--- |:--- |
| Type | *OfficeActivity* |
| ClientIP | IP-адрес устройства, которое использовалось при записи действия в журнал. IP-адрес отображается в формате IPv4- или IPv6-адреса. |
| OfficeWorkload | Служба Office 365, к которой относится запись.<br><br>AzureActiveDirectory<br>Exchange<br>SharePoint|
| Операция | Имя действия пользователя или администратора.  |
| OrganizationId | Идентификатор GUID для клиента Office 365 организации. Это значение для вашей организации всегда будет одинаковым независимо от службы Office 365, в которой оно встречается. |
| RecordType | Тип выполняемой операции. |
| ResultStatus | Указывает, было ли успешным действие (указанное в свойстве Operation). Возможные значения: Succeeded (Успешно), PartiallySucceeded (Выполнено частично) и Failed (Сбой). Для действий администратора Exchange возможные значения — True (Истина) или False (Ложь). |
| UserId | Имя участника-пользователя (UPN) для пользователя, который выполнил действие, приведшее к регистрации в журнале данной записи. Например, my_name@my_domain_name. Обратите внимание, что сюда также включаются записи для действий, выполняемых системными учетными записями (такими как SHAREPOINT\system или NTAUTHORITY\SYSTEM). | 
| UserKey | Альтернативный идентификатор пользователя, определенного в свойстве UserId.  Например, значение этого свойства может заполняться уникальным идентификатором Passport (PUID) для событий, выполняемых пользователями в SharePoint, OneDrive для бизнеса и Exchange. Это свойство также может указывать то же значение, что и свойство UserID событий, происходящих в других службах, и событий, выполняемых системными учетными записями.|
| UserType | Тип пользователя, выполнившего операцию.<br><br>Административный<br>Приложение<br>DcAdmin<br>Обычный<br>Зарезервировано<br>ServicePrincipal<br>Система |


### <a name="azure-active-directory-base"></a>Основа Azure Active Directory

Следующие свойства являются общими для всех записей Azure Active Directory.

| Свойство. | Описание |
|:--- |:--- |
| OfficeWorkload | AzureActiveDirectory |
| RecordType     | AzureActiveDirectory |
| AzureActiveDirectory_EventType | Тип события Azure AD. |
| ExtendedProperties | Расширенные свойства события Azure AD. |


### <a name="azure-active-directory-account-logon"></a>Вход в учетную запись Azure Active Directory

Эти записи создаются при попытке пользователя Active Directory войти в систему.

| Свойство. | Описание |
|:--- |:--- |
| `OfficeWorkload` | AzureActiveDirectory |
| `RecordType`     | AzureActiveDirectoryAccountLogon |
| `Application` | Приложение, которое активирует событие входа в учетную запись, например Office 15. |
| `Client` | Подробные сведения о клиентском устройстве, операционной системе устройства и браузере устройства, который использовался при событии входа в учетную запись. |
| `LoginStatus` | Это свойство получается непосредственно из параметра OrgIdLogon.LoginStatus. Сопоставление различных неудачных попыток входа в систему можно выполнить с помощью алгоритмов оповещения. |
| `UserDomain` | Сведения об идентификации клиента (TII). | 


### <a name="azure-active-directory"></a>Azure Active Directory

Эти записи создаются при внесении изменений или дополнений в объекты Azure Active Directory.

| Свойство. | Описание |
|:--- |:--- |
| OfficeWorkload | AzureActiveDirectory |
| RecordType     | AzureActiveDirectory |
| AADTarget | Пользователь, в отношении которого было выполнено действие (указанное в свойстве Operation). |
| Субъект | Пользователь или субъект-служба, которые выполнили действие. |
| ActorContextId | Идентификатор GUID организации, к которой принадлежит субъект. |
| ActorIpAddress | IP-адрес субъекта в формате IPv4- или IPv6-адреса. |
| InterSystemsId | Идентификатор GUID, который отслеживает действия по компонентам в службе Office 365. |
| IntraSystemId |     Идентификатор GUID, который создается Azure Active Directory для отслеживания действия. |
| SupportTicketId | Идентификатор запроса пользователя в службу поддержки для действия в ситуациях "действовать от имени". |
| TargetContextId | Идентификатор GUID организации, к которой принадлежит целевой пользователь. |


### <a name="data-center-security"></a>Безопасность центра обработки данных

Эти записи создаются на основе данных аудита безопасности центра обработки данных.  

| Свойство. | Описание |
|:--- |:--- |
| EffectiveOrganization | Имя клиента, в отношении которого выполнялось повышение прав или командлет. |
| ElevationApprovedTime | Метка времени, когда был утвержден запрос на повышение прав. |
| ElevationApprover | Имя представителя корпорации Майкрософт. |
| ElevationDuration | Период, в течение которого действовало повышение прав. |
| ElevationRequestId |     Уникальный идентификатор запроса на повышение прав. |
| ElevationRole | Роль, для которой было запрошено повышение прав. |
| ElevationTime | Время начала повышения прав. |
| Start_Time | Время начала выполнения командлета. |


### <a name="exchange-admin"></a>Администратор Exchange

Эти записи создаются при внесении изменений в конфигурацию Exchange.

| Свойство. | Описание |
|:--- |:--- |
| OfficeWorkload | Exchange |
| RecordType     | ExchangeAdmin |
| ExternalAccess |     Указывает, кем выполнялся командлет (пользователем организации, персоналом центра обработки данных корпорации Майкрософт, учетной записью службы центра обработки данных или делегированным администратором). Значение False указывает на то, что командлет выполнялся пользователем вашей организации. Значение True означает, что командлет выполнялся персоналом центра обработки данных, учетной записью службы центра обработки данных или делегированным администратором. |
| ModifiedObjectResolvedName |     Это понятное имя объекта, который был изменен с помощью командлета. Эта запись вносится в журнал только в том случае, если командлет изменяет объект. |
| OrganizationName | Имя клиента. |
| OriginatingServer | Имя сервера, из которого выполнялся командлет. |
| Параметры | Имя и значение для всех параметров, которые использовались с командлетом, указанным в свойстве Operations. |


### <a name="exchange-mailbox"></a>Почтовый ящик Exchange

Эти записи создаются при внесении изменений или дополнений в почтовые ящики Exchange.

| Свойство. | Описание |
|:--- |:--- |
| OfficeWorkload | Exchange |
| RecordType     | ExchangeItem |
| ClientInfoString | Сведения о почтовом клиенте, который использовался для выполнения операции, такие как версия браузера, версия Outlook или сведения о мобильном устройстве. |
| Client_IPAddress | IP-адрес устройства, которое использовалось при записи операции в журнал. IP-адрес отображается в формате IPv4- или IPv6-адреса. |
| ClientMachineName | Имя компьютера, на котором размещен клиент Outlook. |
| ClientProcessName | Клиент электронной почты, который использовался для доступа к почтовому ящику. |
| ClientVersion | Версия клиента электронной почты. |
| InternalLogonType | Зарезервировано для внутреннего использования. |
| Logon_Type | Указывает тип пользователя, который осуществил доступ к почтовому ящику и выполнил операцию, записанную в журнал. |
| LogonUserDisplayName |     Понятное имя пользователя, который выполнил операцию. |
| LogonUserSid | ИД безопасности пользователя, который выполнил операцию. |
| MailboxGuid | Идентификатор GUID Exchange почтового ящика, к которому осуществлялся доступ. |
| MailboxOwnerMasterAccountSid | ИД безопасности главной учетной записи для учетной записи владельца почтового ящика. |
| MailboxOwnerSid | ИД безопасности владельца почтового ящика. |
| MailboxOwnerUPN | Адрес электронной почты пользователя, владеющего почтовым ящиком, к которому осуществлялся доступ. |


### <a name="exchange-mailbox-audit"></a>Аудит почтового ящика Exchange

Эти записи создаются при создании записей аудита почтового ящика.

| Свойство. | Описание |
|:--- |:--- |
| OfficeWorkload | Exchange |
| RecordType     | ExchangeItem |
| Элемент | Представляет элемент, в отношении которого выполнялась операция. | 
| SendAsUserMailboxGuid | Идентификатор GUID Exchange почтового ящика, к которому осуществлялся доступ для отправки почты с помощью команды "Отправить как". |
| SendAsUserSmtp | SMTP-адрес пользователя, олицетворяемого при отправке. |
| SendonBehalfOfUserMailboxGuid | Идентификатор GUID Exchange почтового ящика, к которому осуществлялся доступ для отправки почты с помощью команды "Отправить от имени". |
| SendOnBehalfOfUserSmtp | SMTP-адрес пользователя, от имени которого отправлялась почта. |


### <a name="exchange-mailbox-audit-group"></a>Группа аудита почтового ящика Exchange

Эти записи создаются при внесении изменений или дополнений в группы Exchange.

| Свойство. | Описание |
|:--- |:--- |
| OfficeWorkload | Exchange |
| OfficeWorkload | ExchangeItemGroup |
| AffectedItems | Сведения о каждом элементе в группе. |
| CrossMailboxOperations | Указывает, задействовано ли в операции более одного почтового ящика. |
| DestMailboxId | Используется только в том случае, если параметр CrossMailboxOperations имеет значение True. Указывает GUID целевого почтового ящика. |
| DestMailboxOwnerMasterAccountSid | Используется только в том случае, если параметр CrossMailboxOperations имеет значение True. Указывает ИД безопасности главной учетной записи владельца целевого почтового ящика. |
| DestMailboxOwnerSid | Используется только в том случае, если параметр CrossMailboxOperations имеет значение True. Указывает ИД безопасности целевого почтового ящика. |
| DestMailboxOwnerUPN | Используется только в том случае, если параметр CrossMailboxOperations имеет значение True. Указывает имя участника-пользователя (UPN) владельца целевого почтового ящика. |
| DestFolder | Папка назначения для операций, таких как перемещение. |
| Папка | Папка, где находится группа элементов. |
| Папки |     Сведения об исходных папках, задействованных в операции. Например, если папки выбираются, а затем удаляются. |


### <a name="sharepoint-base"></a>Основа SharePoint

Эти свойства являются общими для всех записей SharePoint.

| Свойство. | Описание |
|:--- |:--- |
| OfficeWorkload | SharePoint |
| OfficeWorkload | SharePoint |
| EventSource | Указывает, что событие произошло в SharePoint. Возможные значения: SharePoint или ObjectModel. |
| ItemType | Тип объекта, к которому осуществлялся доступ или который был изменен. Подробные сведения о типах объектов см. в таблице ItemType. |
| MachineDomainInfo | Сведения об операциях синхронизации устройств. Эти сведения сообщаются только в том случае, если они присутствуют в запросе. |
| MachineId |     Сведения об операциях синхронизации устройств. Эти сведения сообщаются только в том случае, если они присутствуют в запросе. |
| Site_ | Идентификатор GUID сайта, где расположен файл или папка, к которым осуществляется доступ пользователя. |
| Source_Name | Сущность, которая активировала отслеживаемую операцию. Возможные значения: SharePoint или ObjectModel. |
| UserAgent | Сведения о клиенте или браузере пользователя. Эти сведения предоставляются клиентом или браузером. |


### <a name="sharepoint-schema"></a>Схема SharePoint

Эти записи создаются при внесении изменений в конфигурацию SharePoint.

| Свойство. | Описание |
|:--- |:--- |
| OfficeWorkload | SharePoint |
| OfficeWorkload | SharePoint |
| CustomEvent | Необязательная строка для пользовательских событий. |
| Event_Data |     Необязательные полезные данные для пользовательских событий. |
| ModifiedProperties | Свойство включено для событий администрирования, таких как добавление пользователя в качестве участника сайта или группы администраторов семейства веб-сайтов. Это свойство включает в себя имя свойства, которое было изменено (например, группа администраторов сайта), новое значение измененного свойства (такое как пользователь, добавленный в качестве администратора сайта) и предыдущее значение измененного объекта. |


### <a name="sharepoint-file-operations"></a>Операции с файлами SharePoint

Эти записи создаются в ответ на операции с файлами в SharePoint.

| Свойство. | Описание |
|:--- |:--- |
| OfficeWorkload | SharePoint |
| OfficeWorkload | SharePointFileOperation |
| DestinationFileExtension | Расширение файла, который копируется или перемещается. Это свойство отображается только для событий FileCopied и FileMoved. |
| DestinationFileName | Имя файла, который копируется или перемещается. Это свойство отображается только для событий FileCopied и FileMoved. |
| DestinationRelativeUrl | URL-адрес папки назначения, в которую копируется или перемещается файл. Сочетание значений параметров SiteURL, DestinationRelativeURL и DestinationFileName совпадает со значением свойства ObjectID, которое является полным именем пути для скопированного файла. Это свойство отображается только для событий FileCopied и FileMoved. |
| SharingType | Тип разрешений на совместное использование, назначенных пользователю, которому был предоставлен доступ к ресурсу. Пользователь идентифицируется с помощью параметра UserSharedWith. |
| Site_Url | URL-адрес сайта, где расположен файл или папка, к которым осуществляется доступ пользователя. |
| SourceFileExtension | Расширение файла, к которому осуществлялся доступ пользователя. Это свойство остается пустым, если объект, к которому осуществлялся доступ, является папкой. |
| SourceFileName |     Имя файла или папки, к которым осуществляется доступ пользователя. |
| SourceRelativeUrl | URL-адрес папки, содержащей файл, к которому осуществляется доступ пользователя. Сочетание значений параметров SiteURL, SourceRelativeURL и SourceFileName совпадает со значением свойства ObjectID. Это свойство является полным именем пути для файла, к которому осуществляется доступ пользователя. |
| UserSharedWith |     Пользователь, которому был предоставлен доступ к ресурсу. |




## <a name="sample-log-queries"></a>Примеры запросов журналов

В следующей таблице приведены примеры запросов к журналу для записей обновления, собранных этим решением.

| Запрос | Описание |
| --- | --- |
|Количество всех операций для подписки Office 365 |OfficeActivity &#124; summarize count() by Operation |
|Использование сайтов SharePoint|Оффицеактивити &#124;, где Оффицеворклоад = ~ "SharePoint" &#124; суммировать Count () путем SiteUrl \| сортировки по убыванию|
|Операции доступа к файлам по типу пользователя | Оффицеактивити &#124; суммировать Count () по UserType |
|Мониторинг внешних действий в отношении Exchange|OfficeActivity &#124; where OfficeWorkload =~ "exchange" and ExternalAccess == true|



## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения об обновлении данных см. в статье [Анализ данных Log Analytics в Azure Monitor](../logs/log-query-overview.md).
* [Создайте собственные панели мониторинга](../visualize/tutorial-logs-dashboards.md) для отображения избранных поисковых запросов Office 365.
* [Создайте оповещения](../alerts/alerts-overview.md), чтобы заранее получать уведомления о важных действиях в Office 365.  
