---
title: Устранение проблем с настройкой состояния службы автоматизации Azure
description: В этой статье рассказывается, как диагностировать и устранять проблемы с настройкой состояния службы автоматизации Azure.
services: automation
ms.subservice: ''
ms.date: 04/16/2019
ms.topic: troubleshooting
ms.openlocfilehash: e6caf3fed708e89b55a88719ca5358f6174c2ac8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98896534"
---
# <a name="troubleshoot-azure-automation-state-configuration-issues"></a>Устранение проблем с настройкой состояния службы автоматизации Azure

В этой статье содержатся сведения об устранении неполадок и решении проблем, возникающих при компиляции или развертывании конфигураций в настройке состояния службы автоматизации Azure. Общие сведения о функции «Настройка состояния» доступны в разделе [Общие сведения о настройке состояния службы автоматизации Azure](../automation-dsc-overview.md).

## <a name="diagnose-an-issue"></a>Диагностика проблемы

При возникновении ошибки компиляции или развертывания для настройки нужно выполнить следующие диагностические действия.

### <a name="1-ensure-that-your-configuration-compiles-successfully-on-the-local-machine"></a>1. Убедитесь, что конфигурация успешно компилируется на локальном компьютере

Настройка состояния службы автоматизации Azure основана на настройке требуемого состояния (DSC) PowerShell. Документацию по языку DSC и синтаксису можно найти в разделе [Документация DSC PowerShell](/powershell/scripting/overview).

Компилируя конфигурацию DSC на локальном компьютере, можно обнаруживать и устранять распространенные ошибки:

   - отсутствующие модули;
   - синтаксические ошибки;
   - логические ошибки.

### <a name="2-view-dsc-logs-on-your-node"></a>2. Просмотр журналов DSC на узле

Если конфигурация компилируется успешно, но при использовании на узле происходит сбой, найти подробную информацию можно в журналах DSC. Сведения о том, где найти эти журналы, приводятся в статье [Где найти журналы событий DSC](/powershell/scripting/dsc/troubleshooting/troubleshooting#where-are-dsc-event-logs).

Модуль [xDscDiagnostics](https://github.com/PowerShell/xDscDiagnostics) поможет проанализировать подробные сведения из журналов DSC. При обращении в службу поддержки эти журналы необходимы для диагностики проблемы.

Можно установить модуль `xDscDiagnostics` на локальном компьютере, выполнив инструкции из статьи [Установка модуля стабильной версии](https://github.com/PowerShell/xDscDiagnostics#install-the-stable-version-module).

Чтобы установить модуль `xDscDiagnostics` на компьютере Azure, используйте команду [Invoke-AzVMRunCommand](/powershell/module/az.compute/invoke-azvmruncommand). Кроме того, можно воспользоваться функцией **Выполнить команду** на портале Microsoft Azure. Для этого выполните инструкции в статье [Запуск сценариев PowerShell на виртуальной машине с Windows с помощью функции «Выполнить команду»](../../virtual-machines/windows/run-command.md).

Сведения об использовании командлета **xDscDiagnostics** см. в статье [Использование xDscDiagnostics для анализа журналов DSC](/powershell/scripting/dsc/troubleshooting/troubleshooting#using-xdscdiagnostics-to-analyze-dsc-logs). См. также статью [Командлеты xDscDiagnostics](https://github.com/PowerShell/xDscDiagnostics#cmdlets).

### <a name="3-ensure-that-nodes-and-the-automation-workspace-have-required-modules"></a>3. Убедитесь, что узлы и рабочая область службы автоматизации содержат необходимые модули

DSC зависит от модулей, установленных на узле. При настройке состояния службы автоматизации Azure импортируйте необходимые модули в учетную запись службы автоматизации, выполнив инструкции из раздела [Импорт модулей](../shared-resources/modules.md#import-modules). Конфигурации также могут иметь зависимости от определенных версий модулей. Дополнительные сведения см. в статье [Устранение неполадок в модулях](shared-resources.md#modules).

## <a name="scenario-a-configuration-with-special-characters-cant-be-deleted-from-the-portal"></a><a name="unsupported-characters"></a>Сценарий. Невозможно удалить с портала конфигурацию со специальными символами

### <a name="issue"></a>Проблема

При попытке удалить конфигурацию DSC с портала отображается следующая ошибка.

```error
An error occurred while deleting the DSC configuration '<name>'.  Error-details: The argument configurationName with the value <name> is not valid.  Valid configuration names can contain only letters,  numbers, and underscores.  The name must start with a letter.  The length of the name must be between 1 and 64 characters.
```

### <a name="cause"></a>Причина

Эта ошибка является временной проблемой, которую скоро планируется устранить.

### <a name="resolution"></a>Решение

Удалите конфигурацию с помощью командлета [Remove-азаутоматиондскконфигуратион](/powershell/module/Az.Automation/Remove-AzAutomationDscConfiguration) .

## <a name="scenario-failed-to-register-the-dsc-agent"></a><a name="failed-to-register-agent"></a>Сценарий. Не удалось зарегистрировать агент DSC

### <a name="issue"></a>Проблема

При использовании [Set-DscLocalConfigurationManager](/powershell/module/psdesiredstateconfiguration/set-dsclocalconfigurationmanager) или другого командлета DSC отображается ошибка:

```error
Registration of the Dsc Agent with the server
https://<location>-agentservice-prod-1.azure-automation.net/accounts/00000000-0000-0000-0000-000000000000 failed. The
underlying error is: Failed to register Dsc Agent with AgentId 00000000-0000-0000-0000-000000000000 with the server htt
ps://<location>-agentservice-prod-1.azure-automation.net/accounts/00000000-0000-0000-0000-000000000000/Nodes(AgentId='00000000-0000-0000-0000-000000000000'). .
    + CategoryInfo          : InvalidResult: (root/Microsoft/...gurationManager:String) [], CimException
    + FullyQualifiedErrorId : RegisterDscAgentCommandFailed,Microsoft.PowerShell.DesiredStateConfiguration.Commands.Re
   gisterDscAgentCommand
    + PSComputerName        : <computerName>
```

### <a name="cause"></a>Причина

Эта ошибка обычно вызвана брандмауэром, компьютером, который находится за прокси-сервером, или другими сетевыми проблемами.

### <a name="resolution"></a>Решение

Убедитесь, что у компьютера есть доступ к соответствующим конечным точкам DSC, и повторите попытку. Список необходимых портов и адресов можно узнать в разделе [Планирование сети](../automation-dsc-overview.md#network-planning).

## <a name="a-nameunauthorizedscenario-status-reports-return-the-response-code-unauthorized"></a><a name="unauthorized"><a/>Сценарий. Отчеты о состоянии возвращают код ответа «Не санкционировано»

### <a name="issue"></a>Проблема

При регистрации узла во время настройки состояния службы автоматизации Azure отображается одно из следующих сообщений об ошибке.

```error
The attempt to send status report to the server https://{your Automation account URL}/accounts/xxxxxxxxxxxxxxxxxxxxxx/Nodes(AgentId='xxxxxxxxxxxxxxxxxxxxxxxxx')/SendReport returned unexpected response code Unauthorized.
```

```error
VM has reported a failure when processing extension 'Microsoft.Powershell.DSC / Registration of the Dsc Agent with the server failed.
```

### <a name="cause"></a>Причина

Эта проблема вызвана неверным или просроченным сертификатом. См. статью [Повторная регистрация узла](../automation-dsc-onboarding.md#re-register-a-node).

Возможно, проблема также вызвана конфигурацией прокси-сервера, которая блокирует доступ к * **.azure-automation.net**. Дополнительные сведения см. в разделе [Настройка частных сетей](../automation-dsc-overview.md#network-planning). 

### <a name="resolution"></a>Решение

Чтобы повторно зарегистрировать неисправный узел DSC, выполните следующие действия.

#### <a name="step-1-unregister-the-node"></a>Шаг 1. Отмените регистрацию узла

1. На портале Azure откройте **Главная** > **Учетные записи службы автоматизации** > (ваша учетная запись службы автоматизации) > **Настройка состояния (DSC)** .
1. Выберите **Узлы** и узел с проблемами.
1. Выберите **Отменить регистрацию**, чтобы отменить регистрацию узла.

#### <a name="step-2-uninstall-the-dsc-extension-from-the-node"></a>Шаг 2. Удалите из узла расширение DSC

1. На портале Azure откройте **Главная** > **Виртуальная машина** > (неисправный узел) > **Расширения**.
1. Выберите **Microsoft.Powershell.DSC** — расширение PowerShell DSC.
1. Выберите **Удалить**, чтобы удалить расширение.

#### <a name="step-3-remove-all-bad-or-expired-certificates-from-the-node"></a>Шаг 3. Удалите из узла все ошибочные и просроченные сертификаты

На неисправном узле выполните следующие команды в сеансе PowerShell с повышенными правами.

```powershell
$certs = @()
$certs += dir cert:\localmachine\my | ?{$_.FriendlyName -like "DSC"}
$certs += dir cert:\localmachine\my | ?{$_.FriendlyName -like "DSC-OaaS Client Authentication"}
$certs += dir cert:\localmachine\CA | ?{$_.subject -like "CN=AzureDSCExtension*"}
"";"== DSC Certificates found: " + $certs.Count
$certs | FL ThumbPrint,FriendlyName,Subject
If (($certs.Count) -gt 0)
{ 
    ForEach ($Cert in $certs) 
    {
        RD -LiteralPath ($Cert.Pspath) 
    }
}
```

#### <a name="step-4-reregister-the-failing-node"></a>Шаг 4. Снова зарегистрируйте неисправный узел

1. На портале Azure откройте **Главная** > **Учетные записи службы автоматизации** > (ваша учетная запись службы автоматизации) > **Настройка состояния (DSC)** .
1. Выберите **Узлы**.
1. Выберите **Добавить**.
1. Выберите неисправный узел.
1. Нажмите **Подключить** и выберите нужные параметры.

## <a name="scenario-node-is-in-failed-status-with-a-not-found-error"></a><a name="failed-not-found"></a>Сценарий. Узел находится в состоянии сбоя с ошибкой "Не найдено"

### <a name="issue"></a>Проблема

Узел содержит отчет со статусом «Сбой» и ошибку:

```error
The attempt to get the action from server https://<url>//accounts/<account-id>/Nodes(AgentId=<agent-id>)/GetDscAction failed because a valid configuration <guid> cannot be found.
```

### <a name="cause"></a>Причина

Эта ошибка обычно возникает из-за того, что узлу назначается имя конфигурации (например, **ABC**) вместо имени конфигурации узла (файл MOF), например **ABC.WebServer**.

### <a name="resolution"></a>Решение

* Убедитесь, что вы назначаете узел с именем конфигурации узла, а не имя конфигурации.
* Конфигурацию узла можно назначить узлу с помощью портала Azure или с помощью командлета PowerShell.

  * На портале Azure откройте **Главная** > **Учетные записи службы автоматизации** > (ваша учетная запись службы автоматизации) > **Настройка состояния (DSC)** . Затем выберите узел и щелкните **Назначить конфигурацию узла**.
  * Используйте командлет [Set-AzAutomationDscNode](/powershell/module/Az.Automation/Set-AzAutomationDscNode).

## <a name="scenario-no-node-configurations-mof-files-were-produced-when-a-configuration-was-compiled"></a><a name="no-mof-files"></a>Сценарий. При компиляции конфигурации не были созданы файлы конфигурации узла (MOF-файлы)

### <a name="issue"></a>Проблема

Задание компиляции DSC приостановлено с ошибкой:

```error
Compilation completed successfully, but no node configuration **.mof** files were generated.
```

### <a name="cause"></a>Причина

Если выражение, указанное в конфигурации DSC рядом с ключевым словом `Node`, возвращает результат `$null`, конфигурация узла не создается.

### <a name="resolution"></a>Решение

Эту проблему можно устранить одним из следующих способов.

* Убедитесь, что выражение рядом с ключевым словом `Node` в определении конфигурации не равно Null.
* Если при компиляции конфигурации вы передаете [ConfigurationData](../automation-dsc-compile.md), то убедитесь, что передаются значения, которые конфигурация ожидает получить из данных конфигурации.

## <a name="scenario-the-dsc-node-report-becomes-stuck-in-the-in-progress-state"></a><a name="dsc-in-progress"></a>Сценарий. Отчет узла DSC зависает в состоянии «Выполняется»

### <a name="issue"></a>Проблема

Агент DSC выводит такое сообщение:

```error
No instance found with given property values
```

### <a name="cause"></a>Причина

Вы обновили версию Windows Management Framework (WMF) и повредили Инструментарий управления Windows (WMI).

### <a name="resolution"></a>Решение

Выполните инструкции из статьи [Известные проблемы и ограничения настройки требуемого состояния (DSC)](/powershell/scripting/wmf/known-issues/known-issues-dsc).

## <a name="scenario-unable-to-use-a-credential-in-a-dsc-configuration"></a><a name="issue-using-credential"></a>Сценарий. Не удается использовать учетные данные в конфигурации DSC

### <a name="issue"></a>Проблема

Задание компиляции DSC приостановлено со следующей ошибкой:

```error
System.InvalidOperationException error processing property 'Credential' of type <some resource name>: Converting and storing an encrypted password as plaintext is allowed only if PSDscAllowPlainTextPassword is set to true.
```

### <a name="cause"></a>Причина

Вы использовали учетные данные в конфигурации, но не передали соответствующие данные `ConfigurationData`, чтобы присвоить `PSDscAllowPlainTextPassword` значение true для каждой конфигурации узла.

### <a name="resolution"></a>Решение

Передайте правильные данные `ConfigurationData`, чтобы присвоить `PSDscAllowPlainTextPassword` значение true для конфигурации каждого узла, упомянутого в конфигурации. См. раздел [Компилирование конфигураций DSC в службе «Настройка состояния службы автоматизации Azure»](../automation-dsc-compile.md).

## <a name="scenario-failure-processing-extension-error-when-enabling-a-machine-from-a-dsc-extension"></a><a name="failure-processing-extension"></a>Сценарий. Ошибка «Сбой обработки расширения» при включении компьютера из расширения DSC

### <a name="issue"></a>Проблема

При включении компьютера с помощью расширения DSC происходит сбой, который содержит ошибку:

```error
VM has reported a failure when processing extension 'Microsoft.Powershell.DSC'. Error message: \"DSC COnfiguration 'RegistrationMetaConfigV2' completed with error(s). Following are the first few: Registration of the Dsc Agent with the server <url> failed. The underlying error is: The attempt to register Dsc Agent with Agent Id <ID> with the server <url> return unexpected response code BadRequest. .\".
```

### <a name="cause"></a>Причина

Эта ошибка обычно возникает, когда узлу назначено имя конфигурации узла, которое не существует в службе.

### <a name="resolution"></a>Решение

* Убедитесь, что вы назначаете узлу имя, которое точно соответствует имени в службе.
* Можно не включать имя конфигурации узла, что приведет к включению узла, но не к назначению конфигурации узла.

## <a name="scenario-one-or-more-errors-occurred-error-when-registering-a-node-by-using-powershell"></a><a name="cross-subscription"></a>Сценарий. Ошибка «Произошла одна или несколько ошибок» при регистрации узла с помощью PowerShell

### <a name="issue"></a>Проблема

При регистрации узла с использованием командлета [Register-AzAutomationDSCNode](/powershell/module/az.automation/register-azautomationdscnode) или [Register-AzureRMAutomationDSCNode](/powershell/module/azurerm.automation/register-azurermautomationdscnode) отображается следующая ошибка.

```error
One or more errors occurred.
```

### <a name="cause"></a>Причина

Эта ошибка возникает при попытке зарегистрировать узел в отдельной подписке, которая используется учетной записью службы автоматизации.

### <a name="resolution"></a>Решение

Рассматривайте узел для использования с несколькими подписками, как если бы он был определен для отдельного облака или локальной среды. Зарегистрируйте узел, используя один из следующих параметров для включения компьютеров.

* Windows: [физические или виртуальные машины под управлением Windows, расположенные локально или в облачной службе, отличной от Azure или AWS](../automation-dsc-onboarding.md#enable-physicalvirtual-windows-machines).
* Linux: [физические или виртуальные машины под управлением Linux, расположенные локально или в облачной службе, отличной от Azure](../automation-dsc-onboarding.md#enable-physicalvirtual-linux-machines).

## <a name="scenario-provisioning-has-failed-error-message"></a><a name="agent-has-a-problem"></a>Сценарий. Сообщение об ошибке «Сбой подготовки»

### <a name="issue"></a>Проблема

При регистрации узла отображается сообщение об ошибке:

```error
Provisioning has failed
```

### <a name="cause"></a>Причина

Это сообщение появляется при наличии проблем с подключением между узлом и Azure.

### <a name="resolution"></a>Решение

Определите, находится ли узел в виртуальной частной сети (VPN) или имеет другие проблемы с подключением к Azure. См. раздел [Устранение неполадок с развертыванием компонентов](onboarding.md).

## <a name="scenario-failure-with-a-general-error-when-applying-a-configuration-in-linux"></a><a name="failure-linux-temp-noexec"></a>Сценарий. Сбой с общей ошибкой при применении конфигурации в Linux

### <a name="issue"></a>Проблема

При применении конфигурации в Linux происходит сбой, который содержит ошибку:

```error
This event indicates that failure happens when LCM is processing the configuration. ErrorId is 1. ErrorDetail is The SendConfigurationApply function did not succeed.. ResourceId is [resource]name and SourceInfo is ::nnn::n::resource. ErrorMessage is A general error occurred, not covered by a more specific error code..
```

### <a name="cause"></a>Причина

Если для расположения **/tmp** задано значение `noexec`, текущей версии DSC не удается применить конфигурации.

### <a name="resolution"></a>Решение

Удалите параметр `noexec` из расположения **/tmp**.

## <a name="scenario-node-configuration-names-that-overlap-can-result-in-a-bad-release"></a><a name="compilation-node-name-overlap"></a>Сценарий. Перекрывающиеся имена конфигураций узлов могут привести к ошибкам выпуска

### <a name="issue"></a>Проблема

При использовании одного сценария конфигурации для создания нескольких конфигураций узлов, когда некоторые имена конфигураций узлов являются подмножествами других имен, служба компиляции может назначить неправильную конфигурацию. Эта проблема возникает только при использовании одного сценария для генерирования конфигураций с данными конфигурации для каждого узла и только когда происходит перекрытие имен в начале строки. В примере показано использование одного сценария конфигурации для генерирования конфигураций на основе данных узлов, переданных в качестве таблицы хэшей с использованием командлетов, когда данные узла включают серверы с именем **server** и **1server**.

### <a name="cause"></a>Причина

Это известная проблема со службой компиляции.

### <a name="resolution"></a>Решение

Наилучшим решением является локальная компиляция или компиляция в процессе непрерывной интеграции и развертывания с последующей отправкой MOS-файлов конфигурации узла непосредственно в службу. Если компиляция в службе является обязательным требованием, то следующим лучшим решением будет разделение заданий компиляции так, чтобы имена не перекрывались.

## <a name="scenario-gateway-timeout-error-on-dsc-configuration-upload"></a><a name="gateway-timeout"></a>Сценарий. Ошибка времени ожидания шлюза при отправке конфигурации DSC

#### <a name="issue"></a>Проблема

При отправке конфигурации DSC появляется сообщение об ошибке `GatewayTimeout`. 

### <a name="cause"></a>Причина

Эта ошибка может быть вызвана настройками DSC, компиляция которых занимает много времени.

### <a name="resolution"></a>Решение

Можно ускорить синтаксический анализ настроек DSC, явно включив параметры `ModuleName` для любых вызовов [Import-DSCResource](/powershell/scripting/dsc/configurations/import-dscresource).

## <a name="next-steps"></a>Дальнейшие действия

Если вы не нашли здесь свою проблему или рекомендации не позволили ее решить, попробуйте использовать один из следующих каналов для получения дополнительной поддержки.

* Получите ответы специалистов Azure на [форумах Azure](https://azure.microsoft.com/support/forums/).
* Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами. Служба поддержки Azure взаимодействует с сообществом Azure, предоставляя ответы, поддержку и советы экспертов.
* Отправьте запрос в службу поддержки Azure Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и выберите **Получить поддержку**.
