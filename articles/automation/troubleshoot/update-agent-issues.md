---
title: Устранение неполадок с агентом обновления Windows в службе автоматизации Azure
description: В этой статье содержатся сведения об устранении неполадок с агентом обновлений Windows в процессе работы с Управлением обновлениями.
services: automation
ms.date: 01/25/2020
ms.topic: troubleshooting
ms.subservice: update-management
ms.openlocfilehash: 9516210021ce48f069ae3b3b4e02503527e0db24
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "100580885"
---
# <a name="troubleshoot-windows-update-agent-issues"></a>Устранение неполадок с агентом обновления Windows

Существует несколько причин, по которым компьютер во время развертывания Управления обновлениями не будет отображатся как готовый к работе (работоспособный). Вы можете проверить работоспособность агента гибридной рабочей роли Runbook Windows, чтобы определить первоначальную проблему. Далее приведены три состояния готовности компьютера:

* Готово: гибридная рабочая роль Runbook развернута и последний раз отображалась менее часа назад.
* Без подключения. Гибридная рабочая роль Runbook развернута и последний раз встречалась больше часа назад.
* Не настроено. Гибридная рабочая роль Runbook не найдена или не завершила развертывание.

> [!NOTE]
> Между отображением на портале Azure и текущим состоянием компьютера может быть небольшая задержка.

В этой статье описывается, как запустить средство устранения неполадок для компьютеров Azure на портале Azure и сторонних компьютеров в [автономных сценариях](#troubleshoot-offline).

> [!NOTE]
> Сценарий устранения неполадок теперь включает проверки для Служб обновлений Windows Server (WSUS), а также для ключей автоматической загрузки и установки.

## <a name="start-the-troubleshooter"></a>Запуск средства устранения неполадок

Для компьютеров Azure можно запустить страницу "Устранение неполадок агента обновления", щелкнув ссылку **Устранение неполадок** в столбце **Готовность к обновлению агента** на портале. Для устранения неполадок со сторонними компьютерами перейдите к разделу в этой статье. См. раздел [Автономное устранение неполадок](#troubleshoot-offline), чтобы получить сведения об устранении неполадок на компьютере, отличном от Azure.

![Снимок экрана списка виртуальных машин Управления обновлениями](../media/update-agent-issues/vm-list.png)

> [!NOTE]
> Чтобы проверить работоспособность гибридной рабочей роли Runbook, необходимо, чтобы виртуальная машина была запущена. Если виртуальная машина не запущена, появится кнопка **запуска виртуальной машины**.

На странице "Устранения неполадок с агентом обновления" выберите **Выполнить проверки**, чтобы запустить средство устранения неполадок. Средство устранения неполадок использует функцию [Выполнение команд](../../virtual-machines/windows/run-command.md) для запуска на компьютере скрипта, проверяющего зависимости. После того как средство устранения неполадок завершило свою работу, оно возвращает результаты проверок.

![Снимок экрана страницы устранения неполадок с агентом обновления](../media/update-agent-issues/troubleshoot-page.png)

Результаты отобразятся на странице, когда будут готовы. В разделах о проверках отображается то, что входит в состав каждой проверки.

![Снимок экрана проверки для устранения неполадок с агентом обновления](../media/update-agent-issues/update-agent-checks.png)

## <a name="prerequisite-checks"></a>Проверка предварительных требований

### <a name="operating-system"></a>Операционная система

Проверка операционной системы обнаруживает, работает ли в гибридной рабочей роли Runbook одна из операционных систем, показанных в следующей таблице:

|Операционная система  |Примечания  |
|---------|---------|
|Windows Server 2012 и более поздних версий |Требуется .NET Framework 4.6 или более поздней версии. ([Скачать .NET Framework](/dotnet/framework/install/guide-for-developers).)<br/> Требуется Windows PowerShell 5.1.  ([Скачать Windows Management Framework 5.1](https://www.microsoft.com/download/details.aspx?id=54616).)        |

### <a name="net-462"></a>.NET 4.6.2

Проверка .NET Framework определяет, установлена ли в системе версия [.NET Framework 4.6.2](https://www.microsoft.com/en-us/download/details.aspx?id=53345), или более поздняя.

### <a name="wmf-51"></a>WMF 5.1

Проверка WMF определяет, есть ли в системе требуемая версия платформы Windows Management Framework, которой является [Windows Management Framework версии 5.1](https://www.microsoft.com/download/details.aspx?id=54616).

### <a name="tls-12"></a>TLS 1.2

Эта проверка определяет, используете ли вы протокол TLS 1.2 для шифрования связи. Протокол TLS 1.0 больше не поддерживается платформой. Используйте протокол TLS 1.2 для обмена данными со службой "Управление обновлениями".

## <a name="connectivity-checks"></a>Проверка подключения

### <a name="registration-endpoint"></a>Конечная точка регистрации

Эта проверка определяет, может ли агент правильно взаимодействовать со службой агента.

В конфигурации прокси-сервера и брандмауэра необходимо разрешить агенту гибридной рабочей роли Runbook взаимодействовать с конечной точкой регистрации. Список адресов и портов, которые необходимо открыть, приведен в разделе [Настройка сети](../automation-hybrid-runbook-worker.md#network-planning).

### <a name="operations-endpoint"></a>Конечная точка операций

Эта проверка определяет, может ли агент правильно взаимодействовать со службой Job Runtime Data.

В конфигурации прокси-сервера и брандмауэра необходимо разрешить агенту гибридной рабочей роли Runbook взаимодействовать со службой Job Runtime Data. Список адресов и портов, которые необходимо открыть, см. в разделе [Настройка сети](../automation-hybrid-runbook-worker.md#network-planning).

## <a name="vm-service-health-checks"></a>Проверки работоспособности службы виртуальных машин

### <a name="monitoring-agent-service-status"></a>Отслеживание состояния службы агента

Эта проверка определяет, запущен ли на компьютере агент Log Analytics для Windows (`healthservice`). Дополнительные сведения об устранении неполадок со службой см. в разделе [Агент Log Analytics для Windows не работает](hybrid-runbook-worker.md#mma-not-running).

Чтобы получить сведения о переустановке агент Log Analytics для Windows, ознакомьтесь с разделом [Установка агента для Windows](../../azure-monitor/vm/quick-collect-windows-computer.md#install-the-agent-for-windows).

### <a name="monitoring-agent-service-events"></a>Мониторинг событий службы агента

Эта проверка определяет, записывались ли какие-либо события 4502 в журнал Azure Operations Manager на компьютере за последние 24 часа.

Дополнительные сведения об этом событии см. в разделе [Сценарий: событие 4502 в журнале Operations Manager](hybrid-runbook-worker.md#event-4502).

## <a name="access-permissions-checks"></a>Проверка разрешений доступа

> [!NOTE]
> В настоящее время средство устранения неполадок не направляет трафик через прокси-сервер, если он настроен.

### <a name="crypto-folder-access"></a>Доступ к криптографической папке

Проверка доступа к криптографической папке определяет, имеет ли учетная запись локальной системы доступ к папке C:\ProgramData\Microsoft\Crypto\RSA.

## <a name="troubleshoot-offline"></a><a name="troubleshoot-offline"></a>Автономное устранение неполадок

Средство устранения неполадок можно использовать в гибридной рабочей роли в автономном режиме, запустив скрипт локально. Получите следующий скрипт из GitHub: [UM_Windows_Troubleshooter_Offline.ps1](https://github.com/Azure/updatemanagement/blob/main/UM_Windows_Troubleshooter_Offline.ps1). Чтобы запустить сценарий, необходимо установить WMF версии 4.0 или более поздней версии. Сведения о загрузке последней версии PowerShell см. в статье [Установка разных версий PowerShell](/powershell/scripting/install/installing-powershell).

Выходные данные этого скрипта выглядят так:

```output
RuleId                      : OperatingSystemCheck
RuleGroupId                 : prerequisites
RuleName                    : Operating System
RuleGroupName               : Prerequisite Checks
RuleDescription             : The Windows Operating system must be version 6.2.9200 (Windows Server 2012) or higher
CheckResult                 : Passed
CheckResultMessage          : Operating System version is supported
CheckResultMessageId        : OperatingSystemCheck.Passed
CheckResultMessageArguments : {}

RuleId                      : DotNetFrameworkInstalledCheck
RuleGroupId                 : prerequisites
RuleName                    : .NET Framework 4.5+
RuleGroupName               : Prerequisite Checks
RuleDescription             : .NET Framework version 4.5 or higher is required
CheckResult                 : Passed
CheckResultMessage          : .NET Framework version 4.5+ is found
CheckResultMessageId        : DotNetFrameworkInstalledCheck.Passed
CheckResultMessageArguments : {}

RuleId                      : WindowsManagementFrameworkInstalledCheck
RuleGroupId                 : prerequisites
RuleName                    : WMF 5.1
RuleGroupName               : Prerequisite Checks
RuleDescription             : Windows Management Framework version 4.0 or higher is required (version 5.1 or higher is preferable)
CheckResult                 : Passed
CheckResultMessage          : Detected Windows Management Framework version: 5.1.17763.1
CheckResultMessageId        : WindowsManagementFrameworkInstalledCheck.Passed
CheckResultMessageArguments : {5.1.17763.1}

RuleId                      : AutomationAgentServiceConnectivityCheck1
RuleGroupId                 : connectivity
RuleName                    : Registration endpoint
RuleGroupName               : connectivity
RuleDescription             :
CheckResult                 : Failed
CheckResultMessage          : Unable to find Workspace registration information in registry
CheckResultMessageId        : AutomationAgentServiceConnectivityCheck1.Failed.NoRegistrationFound
CheckResultMessageArguments : {}

RuleId                      : AutomationJobRuntimeDataServiceConnectivityCheck
RuleGroupId                 : connectivity
RuleName                    : Operations endpoint
RuleGroupName               : connectivity
RuleDescription             : Proxy and firewall configuration must allow Automation Hybrid Worker agent to communicate with eus2-jobruntimedata-prod-su1.azure-automation.net
CheckResult                 : Passed
CheckResultMessage          : TCP Test for eus2-jobruntimedata-prod-su1.azure-automation.net (port 443) succeeded
CheckResultMessageId        : AutomationJobRuntimeDataServiceConnectivityCheck.Passed
CheckResultMessageArguments : {eus2-jobruntimedata-prod-su1.azure-automation.net}

RuleId                      : MonitoringAgentServiceRunningCheck
RuleGroupId                 : servicehealth
RuleName                    : Monitoring Agent service status
RuleGroupName               : VM Service Health Checks
RuleDescription             : HealthService must be running on the machine
CheckResult                 : Failed
CheckResultMessage          : Log Analytics for Windows service (HealthService) is not running
CheckResultMessageId        : MonitoringAgentServiceRunningCheck.Failed
CheckResultMessageArguments : {Log Analytics agent for Windows, HealthService}

RuleId                      : MonitoringAgentServiceEventsCheck
RuleGroupId                 : servicehealth
RuleName                    : Monitoring Agent service events
RuleGroupName               : VM Service Health Checks
RuleDescription             : Event Log must not have event 4502 logged in the past 24 hours
CheckResult                 : Failed
CheckResultMessage          : Log Analytics agent for Windows service Event Log (Operations Manager) does not exist on the machine
CheckResultMessageId        : MonitoringAgentServiceEventsCheck.Failed.NoLog
CheckResultMessageArguments : {Log Analytics agent for Windows, Operations Manager, 4502}

RuleId                      : CryptoRsaMachineKeysFolderAccessCheck
RuleGroupId                 : permissions
RuleName                    : Crypto RSA MachineKeys Folder Access
RuleGroupName               : Access Permission Checks
RuleDescription             : SYSTEM account must have WRITE and MODIFY access to 'C:\\ProgramData\\Microsoft\\Crypto\\RSA\\MachineKeys'
CheckResult                 : Passed
CheckResultMessage          : Have permissions to access C:\\ProgramData\\Microsoft\\Crypto\\RSA\\MachineKeys
CheckResultMessageId        : CryptoRsaMachineKeysFolderAccessCheck.Passed
CheckResultMessageArguments : {C:\\ProgramData\\Microsoft\\Crypto\\RSA\\MachineKeys}

RuleId                      : TlsVersionCheck
RuleGroupId                 : prerequisites
RuleName                    : TLS 1.2
RuleGroupName               : Prerequisite Checks
RuleDescription             : Client and Server connections must support TLS 1.2
CheckResult                 : Passed
CheckResultMessage          : TLS 1.2 is enabled by default on the Operating System.
CheckResultMessageId        : TlsVersionCheck.Passed.EnabledByDefault
CheckResultMessageArguments : {}
```

## <a name="next-steps"></a>Дальнейшие действия

[Устранение неполадок с гибридной рабочей ролью Runbook](hybrid-runbook-worker.md).
