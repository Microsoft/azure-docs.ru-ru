---
title: Включение аудита безопасности для доменных служб Azure AD | Документация Майкрософт
description: Узнайте, как включить аудит безопасности, чтобы централизовать ведение журнала событий для анализа и оповещений в доменных службах Azure AD.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 662362c3-1a5e-4e94-ae09-8e4254443697
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: caf46850b3d8d6946225575b8a9a732a90847482
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100574147"
---
# <a name="enable-security-audits-for-azure-active-directory-domain-services"></a>Включение аудита безопасности для доменных служб Azure Active Directory

Аудит безопасности Azure Active Directory доменных служб (Azure AD DS) позволяет выполнять события безопасности потоков Azure для целевых ресурсов. К этим ресурсам относятся служба хранилища Azure, рабочие области Azure Log Analytics или концентратор событий Azure. После включения событий аудита безопасности Azure AD DS отправляет все события аудита для выбранной категории в целевой ресурс.

Вы можете архивировать события в службе хранилища Azure и потоковых событиях в программном обеспечении и службе управления событиями (SIEM), используя концентраторы событий Azure, или выполнить собственный анализ и использовать Log Analytics рабочих областей Azure из портал Azure.

> [!IMPORTANT]
> Аудит безопасности AD DS Azure доступен только для управляемых доменов на основе Azure Resource Manager. Сведения о том, как выполнить миграцию, см. в статье [миграция AD DS Azure из классической модели виртуальной сети в Диспетчер ресурсов][migrate-azure-adds].

## <a name="security-audit-destinations"></a>Назначения аудита безопасности

Вы можете использовать службу хранилища Azure, концентраторы событий Azure или рабочие области Azure Log Analytics в качестве целевого ресурса для аудита AD DS безопасности Azure. Эти назначения можно объединять. Например, вы можете использовать службу хранилища Azure для архивации событий аудита безопасности, но рабочую область Azure Log Analytics для анализа и получения сведений в течение короткого срока.

В следующей таблице описаны сценарии для каждого типа ресурса назначения.

> [!IMPORTANT]
> Прежде чем включать аудит безопасности Azure AD DS, необходимо создать целевой ресурс. Эти ресурсы можно создать с помощью портал Azure, Azure PowerShell или Azure CLI.

| Целевой ресурс | Сценарий |
|:---|:---|
|Хранилище Azure| Этот целевой объект следует использовать, если основным необходимо хранить события аудита безопасности в целях архивирования. Другие целевые объекты можно использовать в целях архивирования, однако эти целевые объекты предоставляют возможности, которые выходят за пределы основного нужды в архивации. <br /><br />Прежде чем включать события аудита безопасности Azure AD DS, сначала [Создайте учетную запись хранения Azure](../storage/common/storage-account-create.md).|
|Центры событий Azure| Этот целевой объект следует использовать, когда основная потребность заключается в совместном использовании событий аудита безопасности с дополнительным программным обеспечением, например программным обеспечением анализа данных или программным обеспечением для управления & событий (SIEM).<br /><br />Прежде чем включать события аудита безопасности Azure AD DS, [Создайте концентратор событий с помощью портал Azure](../event-hubs/event-hubs-create.md)|
|Рабочая область Azure Log Analytics| Этот целевой объект следует использовать, если необходимо проанализировать и проверить безопасные аудиты портал Azure напрямую.<br /><br />Прежде чем включать события аудита безопасности Azure AD DS, [Создайте рабочую область log Analytics в портал Azure.](../azure-monitor/logs/quick-create-workspace.md)|

## <a name="enable-security-audit-events-using-the-azure-portal"></a>Включение событий аудита безопасности с помощью портал Azure

Чтобы включить AD DS событий аудита безопасности Azure с помощью портал Azure, выполните следующие действия.

> [!IMPORTANT]
> Аудиты безопасности AD DS Azure не ретроактивного. Вы не можете получить или воспроизвести события из прошлого. AD DS Azure может отсылать события, происходящие только после включения аудита безопасности.

1. Войдите на портал Azure по адресу https://portal.azure.com.
1. В верхней части портал Azure найдите и выберите **доменные службы Azure AD**. Выберите нужный управляемый домен, например *aaddscontoso.com*
1. В окне AD DS Azure выберите **параметры диагностики** в левой части.
1. Диагностика не настраивается по умолчанию. Чтобы начать работу, выберите **Добавить параметр диагностики**.

    ![Добавление параметра диагностики для доменных служб Azure AD](./media/security-audit-events/add-diagnostic-settings.png)

1. Введите имя конфигурации диагностики, например *aadds-Audit*.

    Установите флажок для нужного назначения аудита безопасности. Вы можете выбрать учетную запись хранения Azure, концентратор событий Azure или рабочую область Log Analytics. Эти целевые ресурсы уже должны существовать в подписке Azure. Вы не можете создать ресурсы назначения в этом мастере.

    ![Включение отслеживания необходимого назначения и типа событий аудита](./media/security-audit-events/diagnostic-settings-page.png)

    * **Служба хранилища Azure**
        * Выберите **архивировать в учетной записи хранения**, а затем щелкните **настроить**.
        * Выберите **подписку** и **учетную запись хранения** , которые вы хотите использовать для архивации событий аудита безопасности.
        * Когда все будет готово, нажмите кнопку **ОК**.
    * **Концентраторы событий Azure**
        * Выберите **поток в концентраторе событий**, а затем щелкните **настроить**.
        * Выберите **подписку** и **пространство имен концентратора событий**. При необходимости также выберите **имя концентратора событий** , а затем **имя политики концентратора событий**.
        * Когда все будет готово, нажмите кнопку **ОК**.
    * **Рабочие области аналитики журналов Azure**
        * Выберите **отправить log Analytics**, а затем выберите **подписку** и **log Analytics рабочую область** , которую вы хотите использовать для хранения событий аудита безопасности.

1. Выберите категории журналов, которые требуется добавить для конкретного целевого ресурса. При отправке событий аудита в учетную запись хранения Azure можно также настроить политику хранения, определяющую количество дней хранения данных. Значение по умолчанию *0* оставляет все данные и не поворачивает события по истечении определенного периода времени.

    Можно выбрать разные категории журналов для каждого целевого ресурса в пределах одной конфигурации. Эта возможность позволяет выбрать, какие категории журналов необходимо сохранить для Log Analytics и какие категории следует архивировать, например.

1. По завершении нажмите кнопку **сохранить** , чтобы зафиксировать изменения. Целевые ресурсы начинают принимать события аудита AD DS безопасности Azure вскоре после сохранения конфигурации.

## <a name="enable-security-audit-events-using-azure-powershell"></a>Включение событий аудита безопасности с помощью Azure PowerShell

Чтобы включить AD DS событий аудита безопасности Azure с помощью Azure PowerShell, выполните следующие действия. При необходимости сначала [установите модуль Azure PowerShell и подключитесь к подписке Azure](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Аудиты безопасности AD DS Azure не ретроактивного. Вы не можете получить или воспроизвести события из прошлого. AD DS Azure может отсылать события, происходящие только после включения аудита безопасности.

1. Аутентификация в подписке Azure с помощью командлета [Connect-азаккаунт](/powershell/module/Az.Accounts/Connect-AzAccount) . При появлении запроса введите учетные данные.

    ```azurepowershell
    Connect-AzAccount
    ```

1. Создайте целевой ресурс для событий аудита безопасности.

    * Служба **хранилища Azure**  -  [Создание учетной записи хранения с помощью Azure PowerShell](../storage/common/storage-account-create.md?tabs=azure-powershell)
    * **Концентраторы**  -  событий Azure [Создайте концентратор событий с помощью Azure PowerShell](../event-hubs/event-hubs-quickstart-powershell.md). Также может потребоваться использовать командлет [New-азевенсубаусоризатионруле](/powershell/module/az.eventhub/new-azeventhubauthorizationrule) , чтобы создать правило авторизации, которое предоставляет Azure AD DS разрешения для *пространства имен* концентратора событий. Правило авторизации должно включать права на **Управление**, **прослушивание** и **отправку** .

        > [!IMPORTANT]
        > Убедитесь, что правило авторизации задано для пространства имен концентратора событий, а не самого концентратора событий.

    * **Рабочие области**  -  аналитики журналов Azure [Создайте рабочую область log Analytics с Azure PowerShell](../azure-monitor/logs/powershell-workspace-configuration.md).

1. Получите идентификатор ресурса для управляемого домена Azure AD DS с помощью командлета [Get-азресаурце](/powershell/module/Az.Resources/Get-AzResource) . Создайте переменную с именем *$aadds. ResourceId* для хранения значения:

    ```azurepowershell
    $aadds = Get-AzResource -name aaddsDomainName
    ```

1. Настройте параметры диагностики Azure с помощью командлета [Set-аздиагностиксеттинг](/powershell/module/Az.Monitor/Set-AzDiagnosticSetting) , чтобы использовать целевой ресурс для событий аудита безопасности доменных служб Azure AD. В следующих примерах переменная *$aadds. ResourceId* используется на предыдущем шаге.

    * Служба **хранилища Azure** . Замените *сторажеаккаунтид* именем своей учетной записи хранения:

        ```powershell
        Set-AzDiagnosticSetting `
            -ResourceId $aadds.ResourceId `
            -StorageAccountId storageAccountId `
            -Enabled $true
        ```

    * **Концентраторы событий Azure** — замените *eventHubName* именем концентратора событий, а *евенсубрулеид* — идентификатором правила авторизации:

        ```powershell
        Set-AzDiagnosticSetting -ResourceId $aadds.ResourceId `
            -EventHubName eventHubName `
            -EventHubAuthorizationRuleId eventHubRuleId `
            -Enabled $true
        ```

    * **Рабочие области аналитики журналов Azure** — замените *workspaceId* на идентификатор рабочей области log Analytics:

        ```powershell
        Set-AzureRmDiagnosticSetting -ResourceId $aadds.ResourceId `
            -WorkspaceID workspaceId `
            -Enabled $true
        ```

## <a name="query-and-view-security-audit-events-using-azure-monitor"></a>Запрос и Просмотр событий аудита безопасности с помощью Azure Monitor

Аналитические рабочие области журналов позволяют просматривать и анализировать события аудита безопасности с помощью Azure Monitor и языка запросов Kusto. Этот язык запросов предназначен для использования только для чтения, может похвастаться возможности Power Analytics с простым и удобочитаемым синтаксисом. Дополнительные сведения о том, как приступить к работе с языками запросов Kusto, см. в следующих статьях:

* [Документация по Azure Monitor](../azure-monitor/index.yml)
* [Log Analytics в Azure Monitor](../azure-monitor/logs/log-analytics-tutorial.md);
* [Начало работы с запросами к журналам Azure Monitor](../azure-monitor/logs/get-started-queries.md)
* [Создание панелей мониторинга данных Log Analytics и предоставление общего доступа к ним](../azure-monitor/visualize/tutorial-logs-dashboards.md)

Чтобы начать анализ событий аудита безопасности из AD DS Azure, можно использовать следующие примеры запросов.

### <a name="sample-query-1"></a>Образец запроса 1

Просмотреть все события блокировки учетной записи за последние семь дней:

```Kusto
AADDomainServicesAccountManagement
| where TimeGenerated >= ago(7d)
| where OperationName has "4740"
```

### <a name="sample-query-2"></a>Образец запроса 2

Просмотр всех событий блокировки учетной записи (*4740*) с 3 июня 2020 в 9:00 и 10 июня 2020 полночь, отсортированные по возрастанию на дату и время:

```Kusto
AADDomainServicesAccountManagement
| where TimeGenerated >= datetime(2020-06-03 09:00) and TimeGenerated <= datetime(2020-06-10)
| where OperationName has "4740"
| sort by TimeGenerated asc
```

### <a name="sample-query-3"></a>Образец запроса 3

Просмотр событий входа в учетную запись семь дней назад (с) для учетной записи с именем "пользователь":

```Kusto
AADDomainServicesAccountLogon
| where TimeGenerated >= ago(7d)
| where "user" == tolower(extract("Logon Account:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
```

### <a name="sample-query-4"></a>Образец запроса 4

Просмотр событий входа в учетную запись за семь дней назад с учетной записью с именем пользователя, пытающейся выполнить вход с использованием неправильного пароля (*0xC0000006a*):

```Kusto
AADDomainServicesAccountLogon
| where TimeGenerated >= ago(7d)
| where "user" == tolower(extract("Logon Account:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
| where "0xc000006a" == tolower(extract("Error Code:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
```

### <a name="sample-query-5"></a>Образец запроса 5

Просмотр событий входа в учетную запись за семь дней назад с учетной записью с именем пользователя, пытающейся выполнить вход, пока учетная запись была заблокирована (*0xC0000234*):

```Kusto
AADDomainServicesAccountLogon
| where TimeGenerated >= ago(7d)
| where "user" == tolower(extract("Logon Account:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
| where "0xc0000234" == tolower(extract("Error Code:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
```

### <a name="sample-query-6"></a>Образец запроса 6

Просмотр числа событий входа в учетную запись за семь дней назад после всех попыток входа, выполненных для всех заблокированных пользователей:

```Kusto
AADDomainServicesAccountLogon
| where TimeGenerated >= ago(7d)
| where "0xc0000234" == tolower(extract("Error Code:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
| summarize count()
```

## <a name="audit-event-categories"></a>Категории событий аудита

Аудит безопасности Azure AD DS соответствует традиционному аудиту для традиционных AD DS контроллеров домена. В гибридных средах можно повторно использовать существующие шаблоны аудита, чтобы при анализе событий можно было использовать одну и ту же логику. В зависимости от сценария, необходимого для устранения неполадок или анализа, необходимо ориентироваться на различные категории событий аудита.

Доступны следующие категории событий аудита:

| Имя категории аудита | Описание |
|:---|:---|
| Вход учетной записи|Аудит пытается проверить подлинность данных учетной записи на контроллере домена или в локальном диспетчере учетных записей безопасности (SAM).</p>Параметры и события политики входа и выхода из системы следят за попытками доступа к определенному компьютеру. Параметры и события в этой категории сосредоточены на используемой базе данных учетных записей. Эта категория включает следующие подкатегории:<ul><li>[Аудит проверки учетных данных](/windows/security/threat-protection/auditing/audit-credential-validation)</li><li>[Аудит службы проверки подлинности Kerberos](/windows/security/threat-protection/auditing/audit-kerberos-authentication-service)</li><li>[Аудит операций с билетами службы Kerberos](/windows/security/threat-protection/auditing/audit-kerberos-service-ticket-operations)</li><li>[Аудит других событий входа и выхода](/windows/security/threat-protection/auditing/audit-other-logonlogoff-events)</li></ul>|
| Управление учетными записями|Аудит изменений учетных записей и групп пользователей и компьютеров. Эта категория включает следующие подкатегории:<ul><li>[Аудит управления группами приложений](/windows/security/threat-protection/auditing/audit-application-group-management)</li><li>[Аудит управления учетными записями компьютеров](/windows/security/threat-protection/auditing/audit-computer-account-management)</li><li>[Аудит управления группами распространения](/windows/security/threat-protection/auditing/audit-distribution-group-management)</li><li>[Аудит управления другими учетными записями](/windows/security/threat-protection/auditing/audit-other-account-management-events)</li><li>[Аудит управления группами безопасности](/windows/security/threat-protection/auditing/audit-security-group-management)</li><li>[Аудит управления учетными записями пользователей](/windows/security/threat-protection/auditing/audit-user-account-management)</li></ul>|
| Отслеживание сведений|Выполняет аудит действий отдельных приложений и пользователей на этом компьютере и позволяет понять, как используется компьютер. Эта категория включает следующие подкатегории:<ul><li>[Аудит активности DPAPI](/windows/security/threat-protection/auditing/audit-dpapi-activity)</li><li>[Действие "аудит PNP"](/windows/security/threat-protection/auditing/audit-pnp-activity)</li><li>[Аудит создания процессов](/windows/security/threat-protection/auditing/audit-process-creation)</li><li>[Аудит завершения процессов](/windows/security/threat-protection/auditing/audit-process-termination)</li><li>[Аудит событий RPC](/windows/security/threat-protection/auditing/audit-rpc-events)</li></ul>|
| Доступ к службам каталогов|Аудит пытается получить доступ к объектам в службах домен Active Directory Services (AD DS) и изменять их. Эти события аудита регистрируются только на контроллерах домена. Эта категория включает следующие подкатегории:<ul><li>[Аудит подробной репликации службы каталогов](/windows/security/threat-protection/auditing/audit-detailed-directory-service-replication)</li><li>[Аудит доступа к службе каталогов](/windows/security/threat-protection/auditing/audit-directory-service-access)</li><li>[Аудит изменения службы каталогов](/windows/security/threat-protection/auditing/audit-directory-service-changes)</li><li>[Аудит репликации службы каталогов](/windows/security/threat-protection/auditing/audit-directory-service-replication)</li></ul>|
| Logon-Logoff|Аудит пытается войти на компьютер в интерактивном режиме или по сети. Эти события полезны для отслеживания активности пользователей и выявления потенциальных атак на сетевые ресурсы. Эта категория включает следующие подкатегории:<ul><li>[Аудит блокировки учетных записей](/windows/security/threat-protection/auditing/audit-account-lockout)</li><li>[Аудит заявок пользователей или устройств на доступ](/windows/security/threat-protection/auditing/audit-user-device-claims)</li><li>[Аудит расширенного режима IPsec](/windows/security/threat-protection/auditing/audit-ipsec-extended-mode)</li><li>[Аудит участия в группе](/windows/security/threat-protection/auditing/audit-group-membership)</li><li>[Аудит основного режима IPsec](/windows/security/threat-protection/auditing/audit-ipsec-main-mode)</li><li>[Аудит быстрого режима IPsec](/windows/security/threat-protection/auditing/audit-ipsec-quick-mode)</li><li>[Аудит выхода из системы](/windows/security/threat-protection/auditing/audit-logoff)</li><li>[Аудит входа в систему](/windows/security/threat-protection/auditing/audit-logon)</li><li>[Аудит сервера политики сети](/windows/security/threat-protection/auditing/audit-network-policy-server)</li><li>[Аудит других событий входа и выхода](/windows/security/threat-protection/auditing/audit-other-logonlogoff-events)</li><li>[Аудит специального входа](/windows/security/threat-protection/auditing/audit-special-logon)</li></ul>|
|Доступ к объектам| Аудит пытается получить доступ к конкретным объектам или типам объектов в сети или на компьютере. Эта категория включает следующие подкатегории:<ul><li>[Аудит событий, создаваемых приложениями](/windows/security/threat-protection/auditing/audit-application-generated)</li><li>[Аудит служб сертификации](/windows/security/threat-protection/auditing/audit-certification-services)</li><li>[Аудит сведений об общем файловом ресурсе](/windows/security/threat-protection/auditing/audit-detailed-file-share)</li><li>[Аудит общего файлового ресурса](/windows/security/threat-protection/auditing/audit-file-share)</li><li>[Файловая система аудита](/windows/security/threat-protection/auditing/audit-file-system)</li><li>[Аудит подключения платформы фильтрации](/windows/security/threat-protection/auditing/audit-filtering-platform-connection)</li><li>[Аудит отбрасывания пакетов платформой фильтрации](/windows/security/threat-protection/auditing/audit-filtering-platform-packet-drop)</li><li>[Управление маркерами аудита](/windows/security/threat-protection/auditing/audit-handle-manipulation)</li><li>[Аудит объекта ядра](/windows/security/threat-protection/auditing/audit-kernel-object)</li><li>[Аудит других событий доступа к объектам](/windows/security/threat-protection/auditing/audit-other-object-access-events)</li><li>[Реестр аудита](/windows/security/threat-protection/auditing/audit-registry)</li><li>[Аудит съемных носителей](/windows/security/threat-protection/auditing/audit-removable-storage)</li><li>[Аудит диспетчера учетных записей безопасности](/windows/security/threat-protection/auditing/audit-sam)</li><li>[Аудит сверки с централизованной политикой доступа](/windows/security/threat-protection/auditing/audit-central-access-policy-staging)</li></ul>|
|Изменение политики|Аудит изменений важных политик безопасности в локальной системе или сети. Политики обычно устанавливаются администраторами для защиты сетевых ресурсов. Наблюдение за изменениями или попытками изменения этих политик может быть важным аспектом управления безопасностью для сети. Эта категория включает следующие подкатегории:<ul><li>[Аудит изменения политики аудита](/windows/security/threat-protection/auditing/audit-audit-policy-change)</li><li>[Аудит изменения политики проверки подлинности](/windows/security/threat-protection/auditing/audit-authentication-policy-change)</li><li>[Аудит изменения политики авторизации](/windows/security/threat-protection/auditing/audit-authorization-policy-change)</li><li>[Аудит изменения политики платформы фильтрации](/windows/security/threat-protection/auditing/audit-filtering-platform-policy-change)</li><li>[Аудит изменения политики на уровне правил MPSSVC](/windows/security/threat-protection/auditing/audit-mpssvc-rule-level-policy-change)</li><li>[Аудит других изменений политики](/windows/security/threat-protection/auditing/audit-other-policy-change-events)</li></ul>|
|Использование привилегий| Аудит использования определенных разрешений в одной или нескольких системах. Эта категория включает следующие подкатегории:<ul><li>[Аудит использования привилегий, не затрагивающих конфиденциальные данные](/windows/security/threat-protection/auditing/audit-non-sensitive-privilege-use)</li><li>[Аудит использования привилегий, затрагивающих конфиденциальные данные](/windows/security/threat-protection/auditing/audit-sensitive-privilege-use)</li><li>[Аудит других событий использования привилегий](/windows/security/threat-protection/auditing/audit-other-privilege-use-events)</li></ul>|
|Система| Выполняет аудит изменений на уровне системы на компьютере, не входящем в другие категории, и имеет потенциальные последствия для безопасности. Эта категория включает следующие подкатегории:<ul><li>[Аудит драйвера IPsec](/windows/security/threat-protection/auditing/audit-ipsec-driver)</li><li>[Аудит других системных событий](/windows/security/threat-protection/auditing/audit-other-system-events)</li><li>[Аудит изменения состояния безопасности](/windows/security/threat-protection/auditing/audit-security-state-change)</li><li>[Аудит расширения системы безопасности](/windows/security/threat-protection/auditing/audit-security-system-extension)</li><li>[Аудит целостности системы](/windows/security/threat-protection/auditing/audit-system-integrity)</li></ul>|

## <a name="event-ids-per-category"></a>Идентификаторы событий для каждой категории

 Аудит безопасности Azure AD DS записывает следующие идентификаторы событий, когда конкретное действие активирует событие, подкоторое аудиту:

| Имя категории событий | Идентификаторы событий |
|:---|:---|
|Безопасность входа в учетную запись|4767, 4774, 4775, 4776, 4777|
|Безопасность управления учетными записями|4720, 4722, 4723, 4724, 4725, 4726, 4727, 4728, 4729, 4730, 4731, 4732, 4733, 4734, 4735, 4737, 4738, 4740, 4741, 4742, 4743, 4754, 4755, 4756, 4757, 4758, 4764, 4765, 4766, 4780, 4781|
|Безопасность отслеживания сведений|Нет|
|Безопасность доступа к DS|5136, 5137, 5138, 5139, 5141|
|Безопасность Logon-Logoff|4624, 4625, 4634, 4647, 4648, 4672, 4675, 4964|
|Безопасность доступа к объектам|Нет|
|Безопасность изменения политики|4670, 4703, 4704, 4705, 4706, 4707, 4713, 4715, 4716, 4717, 4718, 4719, 4739, 4864, 4865, 4866, 4867, 4904, 4906, 4911, 4912|
|Безопасность использование привилегий|4985|
|Безопасность системы|4612, 4621|

## <a name="next-steps"></a>Дальнейшие действия

Конкретные сведения о Kusto см. в следующих статьях:

* [Общие сведения о](/azure/kusto/query/) языке запросов Kusto.
* [Руководство по Kusto](/azure/kusto/query/tutorial) для ознакомления с основами запросов.
* [Примеры запросов](/azure/kusto/query/samples) , которые помогут вам изучить новые способы просмотра данных.
* Kusto [рекомендации](/azure/kusto/query/best-practices) по оптимизации запросов в случае успеха.

<!-- LINKS - Internal -->
[migrate-azure-adds]: migrate-from-classic-vnet.md