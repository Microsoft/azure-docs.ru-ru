---
title: Часто задаваемые вопросы о службе автоматизации Azure | Документация Майкрософт
description: В этой статье представлены ответы на часто задаваемые вопросы о службе автоматизации Azure.
services: automation
ms.subservice: ''
ms.topic: conceptual
ms.date: 12/17/2020
ms.openlocfilehash: 2b40cc3d4cea4476ffde8bee8cec694975eb5083
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97724278"
---
# <a name="azure-automation-frequently-asked-questions"></a>Часто задаваемые вопросы о службе автоматизации Azure

В этом списке корпорация Майкрософт собрала часто задаваемые вопросы о службе автоматизации Azure. Если у вас есть другие вопросы о своих возможностях, перейдите на дискуссионный форум и опубликуйте свои вопросы. Если вопрос часто запрашивается, мы добавим его в эту статью, чтобы он был быстро и легко найден.

## <a name="update-management"></a>Управление обновлениями

### <a name="can-i-prevent-unexpected-os-level-upgrades"></a>Можно ли избежать непредвиденных обновлений на уровне ОС?

В некоторых вариациях Linux, например в Red Hat Enterprise Linux, обновления на уровне ОС могут выполняться через пакеты. Это может привести к тому, что при запуске функции "Управление обновлениями" изменяется номер версии операционной системы. Так как функция "Управление обновлениями" использует те же методы для обновления пакетов, что и администратор на локальном компьютере Linux, такое поведение считается преднамеренным.

Чтобы избежать обновления версий ОС при развертывании функции "Управление обновлениями", примените функцию **исключения**.

В Red Hat Enterprise Linux укажите для исключения пакет с именем `redhat-release-server.x86_64`.

### <a name="why-arent-criticalsecurity-updates-applied"></a>Почему не применяются критические обновления и обновления для системы безопасности?

При развертывании обновлений на компьютер Linux вы можете выбрать категории обновлений. Это позволит отобрать те обновления, которые соответствуют указанным критериям. Этот фильтр применяется локально на компьютере при развертывании обновления.

Так как функция "Управление обновлениями" оптимизирует обновления в облаке, некоторые обновления могут быть помечены как влияющие на безопасность, а на локальном компьютере такие сведения отсутствуют. Когда вы применяете критические обновления для компьютера Linux, некоторые из них не будут на локальном компьютере помечены как влияющие на безопасность, а значит не будут применены. При этом функция "Управление обновлениями" сохранит для компьютера статус не соответствующего требованиям, так как есть дополнительные сведения о необходимых обновлениях.

Развертывание обновлений на основе их классификации не поддерживается в CentOS версий RTM. Чтобы правильно развернуть обновления для CentOS, выберите все классификации для установки всех обновлений. Для SUSE, выбрав только **другие обновления** , так как классификация может установить другие обновления безопасности, если они связаны с zypper (диспетчером пакетов) или сначала требуются его зависимости. Это поведение является ограничением zypper. В некоторых случаях вам придется повторно запускать развертывание обновлений, а затем проверять развертывание по журналу обновлений.

### <a name="can-i-deploy-updates-across-azure-tenants"></a>Можно ли развертывать обновления в другом клиенте Azure?

Если у вас есть в другом клиенте Azure компьютеры для обновления, которые подключены к функции "Управление обновлениями", для включения их в график обновлений вы можете применить следующее обходное решение. Выполните командлет [New-AzAutomationSchedule](/powershell/module/Az.Automation/New-AzAutomationSchedule) с параметром `ForUpdateConfiguration`, чтобы создать расписание. Можете также применить командлет [New-AzAutomationSoftwareUpdateConfiguration](/powershell/module/Az.Automation/New-AzAutomationSoftwareUpdateConfiguration), передав ему параметр `NonAzureComputer` со списком компьютеров в другом клиенте. В приведенном ниже примере показано, как это сделать.

```azurepowershell-interactive
$nonAzurecomputers = @("server-01", "server-02")

$startTime = ([DateTime]::Now).AddMinutes(10)

$sched = New-AzAutomationSchedule -ResourceGroupName mygroup -AutomationAccountName myaccount -Name myupdateconfig -Description test-OneTime -OneTime -StartTime $startTime -ForUpdateConfiguration

New-AzAutomationSoftwareUpdateConfiguration  -ResourceGroupName $rg -AutomationAccountName <automationAccountName> -Schedule $sched -Windows -NonAzureComputer $nonAzurecomputers -Duration (New-TimeSpan -Hours 2) -IncludedUpdateClassification Security,UpdateRollup -ExcludedKbNumber KB01,KB02 -IncludedKbNumber KB100
```

## <a name="process-automation---python-runbooks"></a>Автоматизация процессов — модули Runbook Python

### <a name="which-python-3-version-is-supported-in-azure-automation"></a>Какая версия Python 3 поддерживается в службе автоматизации Azure?

Для облачных заданий поддерживается Python 3,8. Сценарии и пакеты из любой версии 3. x могут работать, если код совместим в разных версиях.

Для гибридных заданий на гибридных рабочих ролях Runbook Windows можно выбрать установку любой версии 3. x, которую вы хотите использовать. Для гибридных заданий в гибридных рабочих ролях Runbook Linux мы зависели от версии Python 3, установленной на компьютере, для запуска DSC OMSConfig и гибридной рабочей роли Linux. Рекомендуется установить версию 3,6; Однако разные версии также должны работать, если в сигнатурах методов или контрактах между версиями Python 3 нет критических изменений.

### <a name="can-python-2-and-python-3-runbooks-run-in-same-automation-account"></a>Могут ли модули Runbook Python 2 и Python 3 работать в одной учетной записи службы автоматизации?

Да, не существует ограничений на использование модулей Runbook Python 2 и Python 3 в той же учетной записи службы автоматизации.  

### <a name="what-is-the-plan-for-migrating-existing-python-2-runbooks-and-packages-to-python-3"></a>Каков план переноса существующих модулей Runbook и пакетов Python 2 на Python 3?

Служба автоматизации Azure не планирует переносить модули Runbook и пакеты Python 2 в Python 3. Вам потребуется выполнить эту миграцию самостоятельно. Существующие и новые модули Runbook и пакеты Python 2 продолжат работать.

### <a name="what-are-the-packages-supported-by-default-in-python-3-environment"></a>Какие пакеты поддерживаются в среде Python 3 по умолчанию?

Пакет Azure 4.0.0 устанавливается по умолчанию в среде автоматизации Python 3. Вы можете вручную импортировать более позднюю версию пакета Azure, чтобы переопределить версию по умолчанию.

### <a name="what-if-i-run-a-python-3-runbook-that-references-a-python-2-package-or-vice-versa"></a>Что делать, если запустить модуль Runbook Python 3, который ссылается на пакет Python 2 или наоборот?

Python 2 и Python 3 имеют разные среды выполнения. Во время выполнения модуля Runbook Python 2 можно импортировать только пакеты Python 2 и аналогичные для Python 3.

### <a name="how-do-i-differentiate-between-python-2-and-python-3-runbooks-and-packages"></a>Разделы справки различить модули Runbook и пакеты Python 2 и Python 3?

Python 3 — это новое определение Runbook, которое различает модули Runbook Python 2 и Python 3. Аналогичным образом для пакетов Python 3 появился другой тип пакета.

## <a name="next-steps"></a>Дальнейшие действия

Если вы не ответили на этот вопрос, вы можете обратиться к следующим источникам для получения дополнительных вопросов и ответов.

- [Служба автоматизации Azure](/answers/topics/azure-automation.html)
- [Форум отзывов и предложений](https://feedback.azure.com/forums/905242-update-management)
