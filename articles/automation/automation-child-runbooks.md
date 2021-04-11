---
title: Создание модульных runbook в службе автоматизации Azure
description: В этой статье рассказывается, как создать модуль runbook, который вызывается другим модулем runbook.
services: automation
ms.subservice: process-automation
ms.date: 01/17/2019
ms.topic: conceptual
ms.openlocfilehash: 338de996b06769b9d2891c7208b9050cc3acc7ed
ms.sourcegitcommit: d23602c57d797fb89a470288fcf94c63546b1314
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106167300"
---
# <a name="create-modular-runbooks"></a>Создание модульного runbook

Мы советуем создавать в службе автоматизации Azure модульные runbook с четкой функцией, которую могут использовать другие модули runbook, чтобы их было удобно использовать многократно. Родительский Runbook часто вызывает один или несколько дочерних Runbook для выполнения требуемых функций. 

Вызвать дочерний модуль runbook можно двумя способами, и у каждого есть определенные особенности, которые следует хорошо понимать для правильного выбора в том или ином сценарии. В следующей таблице перечислены различия между двумя методами вызова одного модуля runbook из другого.

|  | Встроенный | Командлет |
|:--- |:--- |:--- |
| **Задание** |Дочерние Runbook выполняются в том же задании, что и родительский. |Для дочернего Runbook создается отдельное задание. |
| **Выполнение** |Родительский Runbook ожидает завершения дочернего Runbook, прежде чем продолжить выполнение. |Родительский модуль Runbook продолжает работу сразу после запуска дочернего модуля Runbook *или* ожидает завершения дочернего задания. |
| **Выходные данные** |Родительский Runbook может получить выходные данные непосредственно из дочернего Runbook. |Родительский модуль Runbook должен получить выходные данные из задания дочернего модуля Runbook *или* может получить выходные данные непосредственно из дочернего модуля Runbook. |
| **Параметры** |Значения параметров дочернего Runbook указываются отдельно и могут иметь любой тип данных. |Значения для параметров дочернего Runbook должны быть объединены в одну хэш-таблицу. Эта хэш-таблица может содержать только простые, регулярные и объектные типы данных, использующие сериализацию JSON. |
| **Учетная запись службы автоматизации** |Родительский модуль runbook может использовать только дочерний модуль runbook, находящийся в той же учетной записи службы автоматизации. |Родительский модуль runbook может использовать дочерний модуль runbook из любой учетной записи службы автоматизации в той же подписке Azure и даже в другой подписке, если к ней есть подключение. |
| **Публикация** |Перед публикацией родительского Runbook должен быть опубликован дочерний Runbook. |Дочерний модуль runbook публикуется в любое время до запуска родительского модуля runbook. |

## <a name="invoke-a-child-runbook-using-inline-execution"></a>Вызов дочернего модуля runbook с использованием встроенного выполнения

Чтобы вызвать один модуль runbook изнутри другого, используйте имя модуля runbook и укажите значения его параметров точно так же, как при использовании действия или командлета. Так все Runbook в одной учетной записи службы автоматизации доступны для использования всем остальным. Родительский модуль runbook ожидает завершения работы дочернего модуля runbook, прежде чем перейти к следующей строке, и любые выходные данные возвращаются непосредственно в родительский модуль runbook.

При встроенном вызове дочерний Runbook выполняется в той же задаче, что и родительский Runbook. В журнале заданий дочернего модуля runbook нет записей о его выполнении. Любые исключения и любой потоковый вывод дочернего модуля runbook связаны с родительским модулем runbook. Такое поведение сокращает число заданий, упрощает мониторинг и устранение неполадок.

При публикации Runbook все дочерние Runbook, которые он вызывает, уже должны быть опубликованы. Это связано с тем, что служба автоматизации Azure создает связь с любыми дочерними модулями runbook при компиляции модуля runbook. Если дочерние модули runbook еще не опубликованы, родительский модуль runbook публикуется без ошибок, но при запуске создает исключение. В этом случае можно повторно опубликовать родительский Runbook, чтобы исправить ссылки на дочерние Runbook. Если изменится любой из дочерних модулей runbook, повторно публиковать родительский модуль runbook не нужно, так как связь уже создана.

Параметры встроенного дочернего модуля runbook могут иметь любой тип данных, включая сложные объекты. [Сериализация JSON](start-runbooks.md#work-with-runbook-parameters), как при запуске модуля runbook с портала Azure или с помощью командлета [Start-AzAutomationRunbook](/powershell/module/Az.Automation/Start-AzAutomationRunbook), отсутствует.

### <a name="runbook-types"></a>Типы Runbook

Какие типы модулей runbook могут вызывать друг друга?

* Модуль [runbook PowerShell](automation-runbook-types.md#powershell-runbooks) и [графические модули runbook](automation-runbook-types.md#graphical-runbooks) могут вызывать друг друга с помощью встроенного вызова, так как оба этих типа основаны на PowerShell.
* Модуль [runbook рабочего процесса PowerShell](automation-runbook-types.md#powershell-workflow-runbooks) и графический модуль runbook рабочего процесса PowerShell могут вызывать друг друга с помощью встроенного вызова, так как оба этих типа основаны на рабочих процессах PowerShell.
* Типы PowerShell и типы рабочего процесса PowerShell не могут вызывать друг друга с помощью встроенного вызова, но могут использовать `Start-AzAutomationRunbook`.

Когда имеет значение порядок публикации?

Порядок публикации модулей runbook имеет значение только для модулей runbook рабочего процесса PowerShell и графических модулей runbook рабочего процесса PowerShell.

Когда родительский модуль runbook использует встроенное выполнение для вызова дочернего графического модуля runbook или модуля runbook рабочего процесса PowerShell, он указывает имя дочернего модуля runbook. Это имя должно начинаться `.\\`, чтобы указать локальное размещение сценария.

### <a name="example"></a>Пример

В следующем примере запускается тестовый дочерний модуль runbook, который принимает сложный объект, целое число и логическое значение. Выходные данные дочернего Runbook присваиваются переменной. В этом случае дочерний модуль runbook является модулем runbook рабочего процесса PowerShell.

```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName "LabRG" -Name "MyVM"
$output = PSWF-ChildRunbook -VM $vm -RepeatCount 2 -Restart $true
```

Этот тот же пример, но с использованием модуля runbook PowerShell в качестве дочернего элемента.

```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName "LabRG" -Name "MyVM"
$output = .\PS-ChildRunbook.ps1 -VM $vm -RepeatCount 2 -Restart $true
```

## <a name="start-a-child-runbook-using-a-cmdlet"></a>Запуск дочернего модуля runbook с помощью командлета

> [!IMPORTANT]
> Если модуль runbook вызывает дочерний модуль runbook с помощью командлета `Start-AzAutomationRunbook` с параметром `Wait`, а дочерний модуль runbook создает результат с типом объекта, операция может завершиться ошибкой. Чтобы обойти такую ошибку, воспользуйтесь статьей [о дочерних модулях runbook с выходными данными объекта](troubleshoot/runbooks.md#child-runbook-object), где описана реализация логики для опроса результатов с применением командлета [Get-AzAutomationJobOutputRecord](/powershell/module/az.automation/get-azautomationjoboutputrecord).

Для запуска модуля runbook можно использовать `Start-AzAutomationRunbook`, как описано в статье [Запуск модуля runbook с помощью Windows PowerShell](start-runbooks.md#start-a-runbook-with-powershell). Этот командлет можно использовать в двух режимах. В одном из режимов командлет возвращает идентификатор задания, которое создается для дочернего модуля runbook. В другом режиме, который можно реализовать в сценарии с помощью параметра *-wait*, командлет ожидает завершения дочернего задания и возвращает выходные данные из дочернего модуля runbook.

Задание дочернего модуля runbook, запущенного с помощью командлета, выполняется отдельно от задания родительского модуля runbook. Это поведение порождает больше заданий, чем при встроенном вызове модуля runbook, и усложняет их отслеживание. Родительский Runbook может запустить несколько дочерних Runbook асинхронно, не ожидая завершения каждого из них. Чтобы реализовать аналогичный механизм параллельного выполнения дочерних модулей runbook при использовании встроенного вызова, в родительском модуле runbook нужно указать [ключевое слово parallel](automation-powershell-workflow.md#use-parallel-processing).

Получение выходных данных дочерних модулей runbook в родительском модуле runbook не гарантируется из-за расписания выполнения. Кроме того, переменные `$VerbosePreference`, `$WarningPreference` и другие могут не передаваться в дочерние модули runbook. Чтобы избежать таких проблем, запускайте дочерние модули runbook как отдельные задания службы автоматизации, используя командлет `Start-AzAutomationRunbook` с параметром `Wait`. Такая методика блокирует выполнение родительского модуля runbook до завершения дочернего модуля runbook.

Если вы не хотите блокировать родительский модуль runbook на весь период ожидания, запускайте дочерний модуль runbook с помощью командлета `Start-AzAutomationRunbook` без параметра `Wait`. В этом случае модуль runbook должен использовать [Get-AzAutomationJob](/powershell/module/az.automation/get-azautomationjob), чтобы дождаться завершения задания. Затем он с помощью командлетов [Get-AzAutomationJobOutput](/powershell/module/az.automation/get-azautomationjoboutput) и [Get-AzAutomationJobOutputRecord](/powershell/module/az.automation/get-azautomationjoboutputrecord) получит нужные результаты.

Параметры дочернего модуля runbook, запускаемого с помощью командлета, предоставляются в формате хэш-таблицы, как описано в разделе, посвященном [параметрам модулей runbook](start-runbooks.md#work-with-runbook-parameters). Можно использовать только простые типы данных. Если у Runbook есть параметр со сложным типом данных, то для него необходимо использовать встроенный вызов.

Контекст подписки может быть потерян при запуске дочерних Runbook в качестве отдельных заданий. Чтобы дочерний модуль runbook вызывал командлеты Az из определенной подписки Azure, он должен выполнить проверку подлинности в этой подписке независимо от родительского модуля runbook.

Если задания из одной учетной записи службы автоматизации работают с несколькими подписками, выбор подписки в одном задании может изменить текущий контекст подписки для других заданий. Чтобы избежать этой проблемы, используйте `Disable-AzContextAutosave -Scope Process` в начале каждого модуля runbook. Это действие сохраняет только контекст для выполнения этого модуля Runbook.

### <a name="example"></a>Пример

Следующий пример запускает дочерний модуль runbook с параметрами и ожидает его завершения, используя командлет `Start-AzAutomationRunbook` с параметром `Wait`. После завершения он получает выходные данные из дочернего модуля runbook с помощью командлета. Для использования `Start-AzAutomationRunbook` сценарий должен выполнить проверку подлинности в подписке Azure.

```azurepowershell-interactive
# Ensure that the runbook does not inherit an AzContext
Disable-AzContextAutosave -Scope Process

# Connect to Azure with Run As account
$ServicePrincipalConnection = Get-AutomationConnection -Name 'AzureRunAsConnection'

Connect-AzAccount `
    -ServicePrincipal `
    -Tenant $ServicePrincipalConnection.TenantId `
    -ApplicationId $ServicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint

$AzureContext = Set-AzContext -SubscriptionId $ServicePrincipalConnection.SubscriptionID

$params = @{"VMName"="MyVM";"RepeatCount"=2;"Restart"=$true}

Start-AzAutomationRunbook `
    -AutomationAccountName 'MyAutomationAccount' `
    -Name 'Test-ChildRunbook' `
    -ResourceGroupName 'LabRG' `
    -AzContext $AzureContext `
    -Parameters $params -Wait
```

## <a name="next-steps"></a>Дальнейшие действия

* Изучите также, [как запустить модуль runbook в службе автоматизации Azure](start-runbooks.md).
* Сведения о мониторинге работы модулей runbook см. в статье [Выходные данные и сообщения runbook в службе автоматизации Azure](automation-runbook-output-and-messages.md).
