---
title: Устранение неполадок гибридной рабочей роли Runbook в службе автоматизации Azure
description: В этой статье описано, как устранять неполадки, которые могут возникать для гибридных рабочих ролей Runbook в службе автоматизации Azure.
services: automation
ms.subservice: ''
author: mgoedtel
ms.author: magoedte
ms.date: 02/11/2021
ms.topic: troubleshooting
ms.openlocfilehash: 15a18cbfc3a80bbfea0b92e5b616104dc0f593af
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100581000"
---
# <a name="troubleshoot-hybrid-runbook-worker-issues"></a>Устранение неполадок с гибридной рабочей ролью runbook

В этой статье описано, как устранять неполадки гибридных рабочих ролей Runbook в службе автоматизации Azure. Общие сведения см. в разделе [Обзор гибридной рабочей роли Runbook](../automation-hybrid-runbook-worker.md).

## <a name="general"></a>Общие сведения

Гибридная рабочая роль Runbook зависит от агента, который используется для взаимодействия с учетной записью службы автоматизации Azure для регистрации рабочей роли, получения заданий runbook и сообщения о состоянии. В Windows это агент Log Analytics для Windows. В Linux это агент Log Analytics для Linux.

### <a name="scenario-runbook-execution-fails"></a><a name="runbook-execution-fails"></a>Сценарий. Происходит сбой выполнения модуля Runbook

#### <a name="issue"></a>Проблема

Происходит сбой выполнения модуля runbook со следующим сообщением об ошибке.

`The job action 'Activate' cannot be run, because the process stopped unexpectedly. The job action was attempted three times.`

Модуль runbook приостанавливается сразу после третьей попытки выполнения. Некоторые условия могут помешать модулю runbook завершить работу. Соответствующее сообщение об ошибке может не включать дополнительные сведения.

#### <a name="cause"></a>Причина

Возможны следующие причины.

* Модули runbook не могут выполнить проверку подлинности с помощью локальных ресурсов.
* Гибридная рабочая роль находится за прокси-сервером или брандмауэром.
* Компьютер, предназначенный для выполнения гибридной рабочей роли Runbook, не соответствует минимальным требованиям к оборудованию.

#### <a name="resolution"></a>Решение

Убедитесь, что компьютер имеет исходящий доступ к **\* . Azure-Automation.NET** через порт 443.

Прежде чем настраивать гибридную рабочую роль Runbook для данной функции, необходимо убедиться, что компьютеры, на которых будет выполняться эта рабочая роль, соответствуют минимальным требованиям к оборудованию. Модули runbook и используемый ими фоновый процесс могут привести к слишком интенсивному использованию системы, что вызовет задержки или увеличение времени ожидания при выполнении заданий runbook.

Убедитесь, что компьютер, предназначенный для выполнения гибридной рабочей роли Runbook, соответствует минимальным требованиям к оборудованию. Если требования выполнены, отследите использование ЦП и памяти, чтобы определить корреляцию между производительностью процессов гибридной рабочей роли Runbook и Windows. Если память или ЦП используются слишком интенсивно, может потребоваться обновить ресурсы. Вы также можете выбрать другой вычислительный ресурс, который соответствует минимальным требованиям и масштабируется в соответствии с требованиями рабочей нагрузки.

Проверьте журнал событий **Microsoft-SMA** на наличие соответствующего события с описанием `Win32 Process Exited with code [4294967295]`. Эта ошибка возникла из-за того, что проверка подлинности в модулях runbook еще не настроена или для группы гибридных рабочих ролей Runbook указаны учетные данные запуска от имени. Проверьте разрешения runbook в разделе [Запуск модулей runbook в гибридной рабочей роли Runbook](../automation-hrw-run-runbooks.md), чтобы убедиться в правильности настройки проверки подлинности для модулей runbook.

### <a name="scenario-event-15011-in-the-hybrid-runbook-worker"></a><a name="cannot-connect-signalr"></a>Сценарий. Событие 15011 в гибридной рабочей роли Runbook

#### <a name="issue"></a>Проблема

Гибридная рабочая роль Runbook получает событие 15011, указывающее, что результат запроса недействителен. Приведенная ниже ошибка возникает, когда рабочая роль пытается открыть подключение к [серверу SignalR](/aspnet/core/signalr/introduction).

`[AccountId={c7d22bd3-47b2-4144-bf88-97940102f6ca}]
[Uri=https://cc-jobruntimedata-prod-su1.azure-automation.net/notifications/hub][Exception=System.TimeoutException: Transport timed out trying to connect
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at JobRuntimeData.NotificationsClient.JobRuntimeDataServiceSignalRClient.<Start>d__45.MoveNext()
`

#### <a name="cause"></a>Причина

Гибридная рабочая роль Runbook настроена неправильно для автоматического развертывания компонентов, например для Управления обновлениями. Развертывание содержит часть, которая подключает виртуальную машину к рабочей области Log Analytics. Сценарий PowerShell ищет рабочую область в подписке с заданным именем. В этом случае рабочая область Log Analytics находится в другой подписке. Сценарий не может найти рабочую область и пытается создать ее, но соответствующее имя уже занято. В результате происходит сбой развертывания.

#### <a name="resolution"></a>Решение

Существует два варианта решения этой проблемы.

* Измените сценарий PowerShell, чтобы искать рабочую область Log Analytics в другой подписке. Это хорошее решение, если в будущем планируется развернуть множество компьютеров с гибридной рабочей ролью Runbook.

* Вручную настройте компьютер с рабочей ролью для запуска в песочнице Orchestrator. Затем запустите модуль runbook, созданный в учетной записи службы автоматизации Azure, для рабочей роли, чтобы протестировать функциональные возможности.

### <a name="scenario-windows-azure-vms-automatically-dropped-from-a-hybrid-worker-group"></a><a name="vm-automatically-dropped"></a>Сценарий. Виртуальные машины Windows Azure автоматически удаляются из группы гибридных рабочих ролей

#### <a name="issue"></a>Проблема

Гибридная рабочая роль Runbook или виртуальные машины не отображаются, если компьютер с рабочей ролью был отключен в течение длительного времени.

#### <a name="cause"></a>Причина

Компьютер с гибридной рабочей ролью Runbook не проверял связь со службой автоматизации Azure более 30 дней. В результате служба автоматизации удалила группу гибридных рабочих ролей Runbook или системную группу рабочих ролей. 

#### <a name="resolution"></a>Решение

Запустите компьютер с рабочей ролью, а затем перерегистрируйте его с помощью службы автоматизации Azure. Инструкции по установке среды runbook и подключению к службе автоматизации Azure см. в разделе [Развертывание гибридной рабочей роли Runbook для Windows](../automation-windows-hrw-install.md).

### <a name="scenario-no-certificate-was-found-in-the-certificate-store-on-the-hybrid-runbook-worker"></a><a name="no-cert-found"></a>Сценарий. Не найден сертификат в хранилище сертификатов в гибридной рабочей роли Runbook

#### <a name="issue"></a>Проблема

Модуль Runbook, запущенный в гибридной рабочей роли Runbook, завершается сбоем со следующей ошибкой.

`Connect-AzAccount : No certificate was found in the certificate store with thumbprint 0000000000000000000000000000000000000000`  
`At line:3 char:1`  
`+ Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -Appl ...`  
`+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~`  
`    + CategoryInfo          : CloseError: (:) [Connect-AzAccount],ArgumentException`  
`    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Profile.ConnectAzAccountCommand`

#### <a name="cause"></a>Причина

Эта ошибка возникает при попытке использовать [учетную запись запуска от имени](../automation-security-overview.md#run-as-accounts) в модуле runbook, который выполняется в гибридной рабочей роли Runbook, где отсутствует сертификат учетной записи запуска от имени. У гибридных рабочих ролей Runbook по умолчанию отсутствует локальный ресурс сертификата. Этот ресурс требуется для правильной работы учетной записи запуска от имени.

#### <a name="resolution"></a>Решение

Если гибридная рабочая роль Runbook представляет собой виртуальную машину Azure, вы можете использовать вместо этого [проверку подлинности runbook с помощью управляемых удостоверений](../automation-hrw-run-runbooks.md#runbook-auth-managed-identities). Этот сценарий упрощает проверку подлинности, позволяя проходить ее для ресурсов Azure с помощью управляемого удостоверения виртуальной машины Azure вместо учетной записи запуска от имени. Если гибридная рабочая роль Runbook установлена на локальном компьютере, необходимо установить сертификат учетной записи запуска от имени на этом компьютере. Чтобы узнать, как установить сертификат, ознакомьтесь с действиями по запуску модуля runbook PowerShell **Export-RunAsCertificateToHybridWorker** в разделе [Запуск модулей runbook в гибридной рабочей роли Runbook](../automation-hrw-run-runbooks.md).

### <a name="scenario-error-403-during-registration-of-a-hybrid-runbook-worker"></a><a name="error-403-on-registration"></a>Сценарий. Ошибка 403 при регистрации гибридной рабочей роли Runbook

#### <a name="issue"></a>Проблема

На этапе первоначальной регистрации рабочей роли происходит сбой, и появляется следующее сообщение об ошибке (403).

`Forbidden: You don't have permission to access / on this server.`

#### <a name="cause"></a>Причина

Возможны следующие причины.

* В параметрах агента есть неправильно введенный идентификатор или ключ (первичный) рабочей области. 
* Гибридная рабочая роль Runbook не может скачать конфигурацию, что приводит к ошибке связывания учетной записи. Когда служба Azure включает компоненты на компьютерах, она поддерживает связывание рабочей области Log Analytics и учетной записи службы автоматизации не для всех регионов. Также возможно, что на компьютере установлено неправильное значение даты или времени. Если время отличается от текущего на 15 минут, развертывание компонента завершается сбоем.

#### <a name="resolution"></a>Решение

##### <a name="mistyped-workspace-id-or-key"></a>Неправильно введенный ключ или идентификатор рабочей области
Чтобы проверить, не был ли неправильно введен ключ или идентификатор рабочей области, см. раздел [Добавление или удаление рабочей области — агент Windows](../../azure-monitor/agents/agent-manage.md#windows-agent) для агента Windows или [Добавление или удаление рабочей области — агент Linux ](../../azure-monitor/agents/agent-manage.md#linux-agent) для агента Linux. Обязательно выберите полную строку на портале Azure, а затем скопируйте и вставьте ее с особой тщательностью.

##### <a name="configuration-not-downloaded"></a>Конфигурация не скачана

Рабочая область Log Analytics и учетная запись службы автоматизации должны находиться в связанном регионе. Список поддерживаемых регионов см. в разделе [Сопоставления службы автоматизации Azure и рабочей области Log Analytics](../how-to/region-mappings.md).

Также может потребоваться изменить дату или часовой пояс компьютера. При выборе настраиваемого диапазона времени убедитесь, что он указан в формате UTC, который может отличаться от местного часового пояса.

### <a name="scenario-set-azstorageblobcontent-fails-on-a-hybrid-runbook-worker"></a><a name="set-azstorageblobcontent-execution-fails"></a>Сценарий: сбой Set-AzStorageBlobContent в гибридной рабочей роли Runbook 

#### <a name="issue"></a>Проблема

При попытке выполнить Runbook происходит сбой `Set-AzStorageBlobContent` , и появляется следующее сообщение об ошибке:

`Set-AzStorageBlobContent : Failed to open file xxxxxxxxxxxxxxxx: Illegal characters in path`

#### <a name="cause"></a>Причина

 Эта ошибка вызвана длительным поведением имени файла для вызовов `[System.IO.Path]::GetFullPath()` , которые добавляют пути UNC.

#### <a name="resolution"></a>Решение

В качестве обходного решения можно создать файл конфигурации с именем `OrchestratorSandbox.exe.config` со следующим содержимым:

```azurecli
<configuration>
  <runtime>
    <AppContextSwitchOverrides value="Switch.System.IO.UseLegacyPathHandling=false" />
  </runtime>
</configuration>
```

Поместите этот файл в ту же папку, где находится исполняемый файл `OrchestratorSandbox.exe` . Например,

`%ProgramFiles%\Microsoft Monitoring Agent\Agent\AzureAutomation\7.3.702.0\HybridAgent`

>[!Note]
> При обновлении агента этот файл конфигурации будет удален, и его потребуется создать заново.

## <a name="linux"></a>Linux

Гибридная рабочая роль Runbook для Linux зависит от [агента Log Analytics для Linux](../../azure-monitor/agents/log-analytics-agent.md), который используется для взаимодействия с учетной записью службы автоматизации с целью регистрации рабочей роли, получения заданий runbook и передачи сведений о состоянии. Если при регистрации рабочей роли произошла ошибка, это могло произойти по следующим причинам:

### <a name="scenario-linux-hybrid-runbook-worker-receives-prompt-for-a-password-when-signing-a-runbook"></a><a name="prompt-for-password"></a>Сценарий. Гибридная рабочая роль Runbook для Linux получает запрос на ввод пароля при подписании модуля runbook

#### <a name="issue"></a>Проблема

При выполнении команды `sudo` для гибридной рабочей роли Runbook для Linux получается непредвиденный запрос на ввод пароля.

#### <a name="cause"></a>Причина

Учетная запись **nxautomationuser** для агента Log Analytics для Linux неправильно настроена в файле **sudoers**. Гибридной рабочей роли Runbook требуется соответствующая настройка разрешений учетной записи и других данных, чтобы она могла подписывать модули runbook в рабочей роли Runbook для Linux.

#### <a name="resolution"></a>Решение

* Убедитесь, что гибридная рабочая роль Runbook имеет исполняемый файл GnuPG (GPG) на компьютере.

* Проверьте конфигурацию учетной записи **nxautomationuser** в файле **sudoers**. См. раздел [Запуск модулей runbook в гибридной рабочей роли Runbook](../automation-hrw-run-runbooks.md).

### <a name="scenario-log-analytics-agent-for-linux-isnt-running"></a><a name="oms-agent-not-running"></a>Сценарий. Агент Log Analytics для Linux не работает

#### <a name="issue"></a>Проблема

Агент Log Analytics для Linux не работает.

#### <a name="cause"></a>Причина

Если агент не запущен, гибридная рабочая роль Runbook для Linux не сможет взаимодействовать со службой автоматизации Azure. Агент может не работать по разным причинам.

#### <a name="resolution"></a>Решение

 Проверьте, запущен ли агент, выполнив команду `ps -ef | grep python`. Вы должны увидеть результат, аналогичный приведенному ниже. Это процессы Python, запущенные от имени учетной записи **nxautomation**. Если компонент службы автоматизации Azure не включен, ни один из следующих процессов не будет запущен.

```bash
nxautom+   8567      1  0 14:45 ?        00:00:00 python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/worker/main.py /var/opt/microsoft/omsagent/state/automationworker/oms.conf rworkspace:<workspaceId> <Linux hybrid worker version>
nxautom+   8593      1  0 14:45 ?        00:00:02 python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/worker/hybridworker.py /var/opt/microsoft/omsagent/state/automationworker/worker.conf managed rworkspace:<workspaceId> rversion:<Linux hybrid worker version>
nxautom+   8595      1  0 14:45 ?        00:00:02 python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/worker/hybridworker.py /var/opt/microsoft/omsagent/<workspaceId>/state/automationworker/diy/worker.conf managed rworkspace:<workspaceId> rversion:<Linux hybrid worker version>
```

Ниже перечислены процессы, запущенные для гибридной рабочей роли Runbook Linux. Все они находятся в каталоге /var/opt/microsoft/omsagent/state/automationworker/.

* **oms.conf**: процесс диспетчера рабочих ролей. Он запускается непосредственно из DSC.
* **worker.conf**: процесс автоматически зарегистрированной гибридной рабочей роли. Он запускается диспетчером рабочих ролей. Этот процесс используется в решении "Управление обновлениями" и незаметен для пользователя. Если Управление обновлениями не включено на компьютере, этот процесс будет отсутствовать.
* **diy/worker.conf**: процесс гибридной рабочей роли DIY. Процесс гибридной рабочей роли DIY используется для выполнения модулей Runbook пользователя в гибридной рабочей роли Runbook. Он отличается от процесса автоматически зарегистрированной гибридной рабочей роли только тем, что использует другую конфигурацию. Если служба автоматизации Azure отключена и гибридная рабочая роль DIY для Linux отсутствует, этот процесс будет отсутствовать.

Если агент не запущен, выполните следующую команду, чтобы запустить службу: `sudo /opt/microsoft/omsagent/bin/service_control restart`.

### <a name="scenario-the-specified-class-doesnt-exist"></a><a name="class-does-not-exist"></a>Сценарий. Указанный класс не существует.

Если в **/var/opt/microsoft/omsconfig/omsconfig.log** присутствует сообщение об ошибке `The specified class does not exist..`, необходимо обновить агент Log Analytics для Linux. Выполните приведенную ниже команду, чтобы повторно установить агент.

```Bash
wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <WorkspaceID> -s <WorkspaceKey>
```

## <a name="windows"></a>Windows

Гибридная рабочая роль Runbook для Windows зависит от [агента Log Analytics для Windows](../../azure-monitor/agents/log-analytics-agent.md), который используется для взаимодействия с учетной записью службы автоматизации с целью регистрации рабочей роли, получения заданий runbook и передачи сведений о состоянии. В случае сбоя регистрации рабочей роли этот раздел содержит некоторые возможные причины.

### <a name="scenario-the-log-analytics-agent-for-windows-isnt-running"></a><a name="mma-not-running"></a>Сценарий. Агент Log Analytics для Windows не работает

#### <a name="issue"></a>Проблема

Служба `healthservice` не работает на компьютере с гибридной рабочей ролью Runbook.

#### <a name="cause"></a>Причина

Если служба Log Analytics для Windows не запущена, гибридная рабочая роль Runbook не сможет взаимодействовать со службой автоматизации Azure.

#### <a name="resolution"></a>Решение

Проверьте, запущен ли агент, введя в PowerShell команду `Get-Service healthservice`. Если служба остановлена, введите в PowerShell команду `Start-Service healthservice`, чтобы запустить эту службу.

### <a name="scenario-event-4502-in-the-operations-manager-log"></a><a name="event-4502"></a>Сценарий. Событие 4502 в журнале Operations Manager

#### <a name="issue"></a>Проблема

В журнале событий в папке **Application and Services Logs\Operations Manager** отображается событие 4502 и сообщение о событии с `Microsoft.EnterpriseManagement.HealthService.AzureAutomation.HybridAgent` со следующим описанием.<br>`The certificate presented by the service \<wsid\>.oms.opinsights.azure.com was not issued by a certificate authority used for Microsoft services. Please contact your network administrator to see if they are running a proxy that intercepts TLS/SSL communication.`

#### <a name="cause"></a>Причина

Эта проблема может быть вызвана тем, что прокси-сервер или сетевой брандмауэр блокируют подключение к Microsoft Azure. Убедитесь, что компьютер имеет исходящий доступ к **\* . Azure-Automation.NET** через порт 443.

#### <a name="resolution"></a>Решение

Журналы сохраняются локально в каждом гибридном компоненте Worker по адресу C:\ProgramData\Microsoft\System Center\Orchestrator\7.2\SMA\Sandboxes. Можно проверить, зарегистрированы ли в журналах событий в папках **Application and Services Logs\Microsoft-SMA\Operations** и **Application and Services Logs\Operations Manager** какие-либо предупреждения или ошибки. Эти журналы указывают на проблемы подключения или другие проблемы, влияющие на включение роли в службе автоматизации Azure, или на проблему, возникающую при нормальных условиях работы. Дополнительные сведения об устранении неполадок агента Log Analytics см. в разделе [Устранение неполадок агента Log Analytics для Windows](../../azure-monitor/agents/agent-windows-troubleshoot.md).

Гибридные рабочие роли отправляют [выходные данные и сообщения Runbook](../automation-runbook-output-and-messages.md) в службу автоматизации Azure так же, как задания runbook, выполняемые в облаке, отправляют выходные данные и сообщения. Вы можете включить потоки Verbose и Progress точно так же, как и для модулей runbook.

### <a name="scenario-orchestratorsandboxexe-cant-connect-to-microsoft-365-through-proxy"></a>Сценарий: Orchestrator.Sandbox.exe не удается подключиться к Microsoft 365 через прокси-сервер

#### <a name="issue"></a>Проблема

Сценарий, выполняющийся в гибридной рабочей роли Runbook Windows, не может подключиться, как ожидалось, Microsoft 365 в песочнице Orchestrator. Для подключения сценарий использует [Connect-MsolService](/powershell/module/msonline/connect-msolservice). 

При настройке **Orchestrator.Sandbox.exe.config** для задания прокси-сервера и списка обхода песочница по-прежнему не подключается должным образом. Файл **Powershell_ise.exe.config** с теми же параметрами прокси-сервера и списка обхода работает ожидаемым образом. Журналы Service Management Automation (SMA) и журналы PowerShell не предоставляют никаких сведений о прокси-сервере.

#### <a name="cause"></a>Причина

Соединение со службами федерации Active Directory (AD FS) на сервере не может обойти прокси-сервер. Помните, что песочница PowerShell запускается от имени зарегистрированного пользователя. Однако песочница Orchestrator допускает глубокую настройку и может игнорировать параметры файла **Orchestrator.Sandbox.exe.config**. Он содержит специальный код для обработки параметров прокси-сервера агента Log Analytics или компьютера, но не для обработки других настраиваемых параметров прокси-сервера. 

#### <a name="resolution"></a>Решение

Вы можете устранить проблему в песочнице Orchestrator, перенеся сценарий, чтобы использовать модули Azure Active Directory вместо модуля MSOnline для командлетов PowerShell. Дополнительные сведения см. в разделе [Миграция из Orchestrator в службу автоматизации Azure (бета-версия)](../automation-orchestrator-migration.md).

Если вы хотите продолжить использовать командлеты модуля MSOnline, измените сценарий так, чтобы он использовал [Invoke-Command](/powershell/module/microsoft.powershell.core/invoke-command). Укажите значения параметров `ComputerName` и `Credential`. 

```powershell
$Credential = Get-AutomationPSCredential -Name MyProxyAccessibleCredential
Invoke-Command -ComputerName $env:COMPUTERNAME -Credential $Credential 
{ Connect-MsolService … }
```

Это изменение кода запускает совершенно новый сеанс PowerShell в контексте указанных учетных данных. Он должен обеспечить прохождение трафика через прокси-сервер, выполняющий проверку подлинности активного пользователя.

>[!NOTE]
>Это решение устраняет необходимость работы с файлом конфигурации песочницы. Даже если вы добились работы файла конфигурации со сценарием, этот файл удаляется при каждом обновлении агента гибридной рабочей роли Runbook.

### <a name="scenario-hybrid-runbook-worker-not-reporting"></a><a name="corrupt-cache"></a>Сценарий. Гибридная рабочая роль Runbook не создает отчеты

#### <a name="issue"></a>Проблема

Компьютер гибридной рабочей роли Runbook работает, но в рабочей области не отображаются данные о пульсе для компьютера.

В следующем примере запроса показаны компьютеры в рабочей области и их последние сведения о пульсе:

```kusto
Heartbeat
| summarize arg_max(TimeGenerated, *) by Computer
```

#### <a name="cause"></a>Причина

Эта проблема может быть вызвана повреждением кэша в гибридной рабочей роли Runbook.

#### <a name="resolution"></a>Решение

Чтобы решить данную проблему, войдите в систему гибридной рабочей роли Runbook и запустите следующий сценарий. Он останавливает агент Log Analytics для Windows, удаляет его кэш и перезапускает службу. Это действие заставляет гибридную рабочую роль Runbook повторно скачать свою конфигурацию из службы автоматизации Azure.

```powershell
Stop-Service -Name HealthService

Remove-Item -Path 'C:\Program Files\Microsoft Monitoring Agent\Agent\Health Service State' -Recurse

Start-Service -Name HealthService
```

### <a name="scenario-you-cant-add-a-windows-hybrid-runbook-worker"></a><a name="already-registered"></a>Сценарий: невозможно добавить гибридную рабочую роль Runbook Windows

#### <a name="issue"></a>Проблема

Вы получаете следующее сообщение при попытке добавить гибридную рабочую роль Runbook с помощью командлета `Add-HybridRunbookWorker`.

`Machine is already registered`

#### <a name="cause"></a>Причина

Эта проблема может возникать, если компьютер уже зарегистрирован в другой учетной записи службы автоматизации или если это попытка повторного добавления гибридной рабочей роли Runbook после ее удаления с компьютера.

#### <a name="resolution"></a>Решение

Чтобы устранить эту проблему, удалите следующий раздел реестра, перезапустите `HealthService` и выполните командлет `Add-HybridRunbookWorker` еще раз.

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\HybridRunbookWorker`

### <a name="scenario-you-cant-add-a-linux-hybrid-runbook-worker"></a><a name="already-registered"></a>Сценарий. невозможно добавить гибридную рабочую роль Runbook Linux

#### <a name="issue"></a>Проблема

При попытке добавить гибридную рабочую роль Runbook с помощью скрипта Python появляется следующее сообщение `sudo python /opt/microsoft/omsconfig/.../onboarding.py --register` :

`Unable to register, an existing worker was found. Please deregister any existing worker and try again.`

Кроме того, попытка отменить регистрацию гибридной рабочей роли Runbook с помощью `sudo python /opt/microsoft/omsconfig/.../onboarding.py --deregister` скрипта Python:

`Failed to deregister worker. [response_status=404]`

#### <a name="cause"></a>Причина

Эта проблема может возникать, если компьютер уже зарегистрирован в другой учетной записи службы автоматизации, если группа гибридных рабочих ролей Azure была удалена, или если вы попытаетесь повторно добавить гибридную рабочую роль Runbook после удаления с компьютера.

#### <a name="resolution"></a>Решение

Для разрешения этой проблемы:

1. Удалите агент `sudo sh onboard_agent.sh --purge` .

1. Выполните следующие команды:

   ```
   sudo mv -f /home/nxautomation/state/worker.conf /home/nxautomation/state/worker.conf_old
   sudo mv -f /home/nxautomation/state/worker_diy.crt /home/nxautomation/state/worker_diy.crt_old
   sudo mv -f /home/nxautomation/state/worker_diy.key /home/nxautomation/state/worker_diy.key_old
   ```

1. Повторно подключите агент `sudo sh onboard_agent.sh -w <workspace id> -s <workspace key> -d opinsights.azure.com` .

1. Дождитесь `/opt/microsoft/omsconfig/modules/nxOMSAutomationWorker` заполнения папки.

1. Повторите `sudo python /opt/microsoft/omsconfig/.../onboarding.py --register` сценарий Python.

## <a name="next-steps"></a>Дальнейшие действия

Если вы не нашли здесь свою проблему или рекомендации не позволили ее решить, попробуйте использовать один из следующих каналов для получения дополнительной поддержки.

* Получите ответы специалистов Azure на [форумах Azure](https://azure.microsoft.com/support/forums/).
* Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами. Служба поддержки Azure взаимодействует с сообществом Azure, предоставляя ответы, поддержку и советы экспертов.
* Отправьте запрос в службу поддержки Azure Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и выберите **Получить поддержку**.
