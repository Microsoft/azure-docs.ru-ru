---
title: Управление модулями runbook в службе автоматизации Azure
description: В этой статье описывается управление последовательностями runbook в службе автоматизации Azure.
services: automation
ms.subservice: process-automation
ms.date: 02/24/2021
ms.topic: conceptual
ms.openlocfilehash: af767ab37e8e77195b7d13b24ea78f4fb88485fb
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102122142"
---
# <a name="manage-runbooks-in-azure-automation"></a>Управление модулями runbook в службе автоматизации Azure

Чтобы добавить последовательность runbook в службу автоматизации Azure, можно создать новую последовательность или импортировать уже имеющуюся из файла или из [коллекции runbook](automation-runbook-gallery.md). В этой статье содержатся сведения об управлении последовательностью runbook, импортированной из файла. Информацию о получении доступа к последовательностям runbook и модулям сообществ см. в статье о [коллекциях runbook и модулей для службы автоматизации Azure](automation-runbook-gallery.md).

## <a name="create-a-runbook"></a>Создание модуля runbook

Создайте новую последовательность runbook в службе автоматизации Azure с помощью портала Azure или Windows PowerShell. После создания последовательности runbook ее можно изменить, используя информацию в следующих документах.

* [Изменение текстовой последовательности runbook в службе автоматизации Azure](automation-edit-textual-runbook.md)
* [Изучение основных понятий рабочего процесса Windows PowerShell для последовательностей runbook службы автоматизации](automation-powershell-workflow.md)
* [Управление пакетами Python 2 в службе автоматизации Azure](python-packages.md)
* [Управление пакетами Python 3 (Предварительная версия) в службе автоматизации Azure](python-3-packages.md)

### <a name="create-a-runbook-in-the-azure-portal"></a>Создание модуля Runbook на портале Azure

1. На портале Azure выберите свою учетную запись службы автоматизации.
2. В концентраторе щелкните **Модули Runbook** в разделе **Автоматизация процессов**, чтобы открыть список последовательностей runbook.
3. Щелкните **Создать Runbook**.
4. Введите имя последовательности runbook и выберите ее [тип](automation-runbook-types.md). Имя последовательности runbook должно начинаться с буквы и может содержать буквы, цифры, символы подчеркивания и дефисы.
5. Щелкните **Создать** , чтобы создать модуль Runbook и открыть текстовый редактор.

### <a name="create-a-runbook-with-powershell"></a>Создание модуля Runbook с помощью PowerShell

Используйте командлет [New-AzAutomationRunbook](/powershell/module/az.automation/new-azautomationrunbook), чтобы создать пустую последовательность runbook. Используйте параметр `Type`, чтобы задать один из четырех типов Runbook, определенных для `New-AzAutomationRunbook`.

В приведенном ниже примере показано, как создать пустую последовательность runbook.

```azurepowershell-interactive
New-AzAutomationRunbook -AutomationAccountName MyAccount `
-Name NewRunbook -ResourceGroupName MyResourceGroup -Type PowerShell
```

## <a name="import-a-runbook"></a>Импорт модуля Runbook

Вы можете импортировать скрипт PowerShell или рабочего процесса PowerShell (**PS1**), графический Runbook (**. graphrunbook**) или сценарий Python 2 или Python 3 (**. копировать**), чтобы создать собственный модуль Runbook. При этом необходимо указать [тип модуля Runbook](automation-runbook-types.md), который создается во время импорта, с учетом следующих рекомендаций.

* Вы можете импортировать файл **PS1** без рабочего процесса либо в [последовательность runbook PowerShell](automation-runbook-types.md#powershell-runbooks), либо в [последовательность runbook рабочего процесса PowerShell](automation-runbook-types.md#powershell-workflow-runbooks). Если импортировать его в последовательность runbook рабочего процесса PowerShell, он преобразуется в рабочий процесс. В этом случае в runbook добавляются комментарии для описания внесенных изменений.

* В **последовательность runbook рабочего процесса PowerShell** можно импортировать только файл [PS1](automation-runbook-types.md#powershell-workflow-runbooks), содержащий рабочий процесс PowerShell. Если файл содержит несколько рабочих процессов PowerShell, импорт завершится ошибкой. Каждый рабочий процесс необходимо сохранить в отдельный файл и импортировать отдельно.

* Не импортируйте файл **PS1**, содержащий рабочий процесс PowerShell, в [последовательность runbook PowerShell](automation-runbook-types.md#powershell-runbooks), так как обработчик скриптов PowerShell не сможет его распознать.

* В новую [графическую последовательность runbook](automation-runbook-types.md#graphical-runbooks) следует импортировать только файл **GRAPHRUNBOOK**.

### <a name="import-a-runbook-from-the-azure-portal"></a>Импорт последовательности runbook с портала Azure

Для импорта файла сценария в службу автоматизации Azure можно использовать описанную ниже процедуру.

> [!NOTE]
> С помощью портала **PS1**-файл можно импортировать только в последовательность runbook рабочего процесса PowerShell.

1. На портале Azure выберите свою учетную запись службы автоматизации.
2. Щелкните **Модули Runbook** в разделе **Автоматизация процессов**, чтобы открыть список модулей runbook.
3. Щелкните **Импортировать модуль Runbook**.
4. Щелкните **Файл Runbook** и выберите файл для импорта.
5. Если поле **Имя** активно, можно изменить имя runbook. Имя должно начинаться с буквы и может содержать буквы, цифры, символы подчеркивания и дефисы.
6. [Тип Runbook](automation-runbook-types.md) выбирается автоматически, но его можно изменить, учитывая применимые ограничения.
7. Нажмите кнопку **Создать**. Новый модуль Runbook появится в списке модулей Runbook для учетной записи службы автоматизации.
8. Перед запуском модуля его необходимо [опубликовать](#publish-a-runbook) .

> [!NOTE]
> После импорта графической последовательности runbook ее можно преобразовать в другой тип. Однако графическую последовательность runbook нельзя преобразовать в текстовую последовательность runbook.

### <a name="import-a-runbook-with-windows-powershell"></a>Импорт последовательности runbook с помощью Windows PowerShell

Чтобы импортировать файл скрипта как черновик runbook, воспользуйтесь командлетом [Import-AzAutomationRunbook](/powershell/module/az.automation/import-azautomationrunbook). Если runbook уже существует, импорт завершится ошибкой. Чтобы этого не произошло, используйте командлет с параметром `Force`.

В приведенных ниже примерах показано, как импортировать файл сценария в последовательность runbook.

```azurepowershell-interactive
$automationAccountName =  "AutomationAccount"
$runbookName = "Sample_TestRunbook"
$scriptPath = "C:\Runbooks\Sample_TestRunbook.ps1"
$RGName = "ResourceGroup"

Import-AzAutomationRunbook -Name $runbookName -Path $scriptPath `
-ResourceGroupName $RGName -AutomationAccountName $automationAccountName `
-Type PowerShellWorkflow
```

## <a name="handle-resources"></a>Работа с ресурсами

Если runbook создает [ресурсы](automation-runbook-execution.md#resources), перед их созданием скрипт должен проверить их наличие. Ниже приведен простой пример.

```powershell
$vmName = "WindowsVM1"
$resourceGroupName = "myResourceGroup"
$myCred = Get-AutomationPSCredential "MyCredential"
$vmExists = Get-AzResource -Name $vmName -ResourceGroupName $resourceGroupName

if(!$vmExists)
    {
    Write-Output "VM $vmName does not exist, creating"
    New-AzVM -Name $vmName -ResourceGroupName $resourceGroupName -Credential $myCred
    }
else
    {
    Write-Output "VM $vmName already exists, skipping"
    }
```

## <a name="retrieve-details-from-activity-log"></a>Получение сведений из журнала действий

Вы можете получить сведения о runbook, например данные о пользователе, запустившем runbook, или учетной записи, которая для этого использовалась, из [журнала действий](automation-runbook-execution.md#activity-logging) учетной записи службы автоматизации. В следующем примере PowerShell определяется последний пользователь, выполнивший указанную последовательность runbook.

```powershell-interactive
$SubID = "00000000-0000-0000-0000-000000000000"
$AutomationResourceGroupName = "MyResourceGroup"
$AutomationAccountName = "MyAutomationAccount"
$RunbookName = "MyRunbook"
$StartTime = (Get-Date).AddDays(-1)
$JobActivityLogs = Get-AzLog -ResourceGroupName $AutomationResourceGroupName -StartTime $StartTime `
                                | Where-Object {$_.Authorization.Action -eq "Microsoft.Automation/automationAccounts/jobs/write"}

$JobInfo = @{}
foreach ($log in $JobActivityLogs)
{
    # Get job resource
    $JobResource = Get-AzResource -ResourceId $log.ResourceId

    if ($JobInfo[$log.SubmissionTimestamp] -eq $null -and $JobResource.Properties.runbook.name -eq $RunbookName)
    {
        # Get runbook
        $Runbook = Get-AzAutomationJob -ResourceGroupName $AutomationResourceGroupName -AutomationAccountName $AutomationAccountName `
                                            -Id $JobResource.Properties.jobId | ? {$_.RunbookName -eq $RunbookName}

        # Add job information to hashtable
        $JobInfo.Add($log.SubmissionTimestamp, @($Runbook.RunbookName,$Log.Caller, $JobResource.Properties.jobId))
    }
}
$JobInfo.GetEnumerator() | sort key -Descending | Select-Object -First 1
```

## <a name="track-progress"></a>Отслеживание хода выполнения

Рекомендуется создавать последовательности runbook с модульной структурой, используя удобную для повторного использования и перезапуска логику. Отслеживание хода выполнения в последовательности runbook гарантирует правильность выполнения ее логики при возникновении проблем.

Отслеживать ход выполнения последовательности runbook можно с использованием внешнего источника, например учетной записи хранения, базы данных или общих файлов. Создайте в последовательности runbook логику, которая позволит сначала проверять состояние последнего выполненного действия. Затем, основываясь на результатах проверки, логика может либо пропустить, либо продолжить выполнение конкретных задач в последовательности runbook.

## <a name="prevent-concurrent-jobs"></a>Предотвращение выполнения одновременных заданий

При выполнении нескольких заданий одновременно некоторые последовательности runbook могут работать некорректно. В этом случае важно реализовать логику для проверки последовательности runbook на наличие уже выполняющегося задания. Ниже приведен простой пример.

```powershell
# Authenticate to Azure
$connection = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -Tenant $connection.TenantID `
-ApplicationId $connection.ApplicationID -CertificateThumbprint $connection.CertificateThumbprint

$AzureContext = Get-AzSubscription -SubscriptionId $connection.SubscriptionID

# Check for already running or new runbooks
$runbookName = "<RunbookName>"
$rgName = "<ResourceGroupName>"
$aaName = "<AutomationAccountName>"
$jobs = Get-AzAutomationJob -ResourceGroupName $rgName -AutomationAccountName $aaName -RunbookName $runbookName -AzContext $AzureContext

# Check to see if it is already running
$runningCount = ($jobs | ? {$_.Status -eq "Running"}).count

If (($jobs.status -contains "Running" -And $runningCount -gt 1 ) -Or ($jobs.Status -eq "New")) {
    # Exit code
    Write-Output "Runbook is already running"
    Exit 1
} else {
    # Insert Your code here
}
```

## <a name="handle-transient-errors-in-a-time-dependent-script"></a>Обработка временных ошибок в скрипте, зависящем от времени

Последовательности runbook должны быть надежными и поддерживать обработку [ошибок](automation-runbook-execution.md#errors), в том числе временных ошибок, которые могут привести к их перезапуску или сбою. В случае сбоя последовательности runbook служба автоматизации Azure повторяет попытку ее выполнения.

Если runbook обычно работает в условиях ограничения по времени, то для проверки времени выполнения скрипт должен реализовывать логику. Эта проверка обеспечивает выполнение таких операций, как запуск, завершение работы или горизонтальное увеличение масштаба, только в определенное время.

> [!NOTE]
> Местное время для процесса песочницы Azure задается в формате UTC. Этот факт следует учитывать при вычислении даты и времени в runbook.

## <a name="work-with-multiple-subscriptions"></a>Использование нескольких подписок

Последовательность runbook должна поддерживать работу с [подписками](automation-runbook-execution.md#subscriptions). Например, для работы с несколькими подписками последовательность runbook использует командлет [Disable-AzContextAutosave](/powershell/module/Az.Accounts/Disable-AzContextAutosave). Этот командлет гарантирует, что контекст проверки подлинности не будет получен из другой последовательности runbook, выполняющейся в той же песочнице. Модуль Runbook также использует `Get-AzContext` командлет, чтобы получить контекст текущего сеанса и присвоить его переменной `$AzureContext` .

```powershell
Disable-AzContextAutosave -Scope Process

$Conn = Get-AutomationConnection -Name AzureRunAsConnection
$AzureContext = Connect-AzAccount -ServicePrincipal `
-Tenant $Conn.TenantID `
-ApplicationId $Conn.ApplicationID `
-CertificateThumbprint $Conn.CertificateThumbprint `
-Subscription $Conn.SubscriptionId

$ChildRunbookName = 'ChildRunbookDemo'
$AutomationAccountName = 'myAutomationAccount'
$ResourceGroupName = 'myResourceGroup'

Start-AzAutomationRunbook `
-ResourceGroupName $ResourceGroupName `
-AutomationAccountName $AutomationAccountName `
-Name $ChildRunbookName `
-DefaultProfile $AzureContext
```

## <a name="work-with-a-custom-script"></a>Работа с пользовательскими скриптами

> [!NOTE]
> Обычно пользовательские скрипты и последовательности runbook нельзя запускать на узле с установленным агентом Log Analytics.

Чтобы использовать пользовательский скрипт, выполните следующие действия.

1. Создайте учетную запись службы автоматизации и получите [роль участника](automation-role-based-access-control.md).
2. [Свяжите учетную запись с рабочей областью Azure](../security-center/security-center-enable-data-collection.md).
3. Включите функцию [Гибридная рабочая роль Runbook](automation-hybrid-runbook-worker.md), [Управление обновлениями](./update-management/overview.md) или другую функцию службы автоматизации. 
4. На компьютере с Linux требуется высокий уровень разрешений. Войдите, чтобы [отключить проверку подписи](automation-linux-hrw-install.md#turn-off-signature-validation).

## <a name="test-a-runbook"></a>Тестирование runbook

При тестировании модуля Runbook запускается его [черновая версия](#publish-a-runbook) и завершаются все действия, которые он выполняет. Журнал заданий не создается, однако в области вывода теста отображаются [поток вывода](automation-runbook-output-and-messages.md#use-the-output-stream) и поток [предупреждений и ошибок](automation-runbook-output-and-messages.md#working-with-message-streams). Сообщения, предназначенные для [подробного потока](automation-runbook-output-and-messages.md#write-output-to-verbose-stream), отображаются в области выходных данных, только если переменная [VerbosePreference](automation-runbook-output-and-messages.md#work-with-preference-variables) имеет значение `Continue`.

Несмотря на то что выполняется черновая версия, модуль Runbook работает в обычном режиме и выполняет все действия с использованием ресурсов среды. В связи с этим тестировать модули Runbook можно только в непроизводственных ресурсах.

Процедура тестирования одинакова для всех [типов runbook](automation-runbook-types.md). Тестирование выполняется на портале Azure одинаково и в текстовом, и в графическом редакторе.

1. Откройте черновую версию runbook в [текстовом](automation-edit-textual-runbook.md) или [графическом](automation-graphical-authoring-intro.md) редакторе.
1. Щелкните **Тест**, чтобы открыть страницу тестирования.
1. Если у runbook имеются параметры, они отображаются в области слева, где можно указать значения для теста.
1. Если вы хотите запустить тест с [гибридной рабочей ролью Runbook](automation-hybrid-runbook-worker.md), измените **Параметры запуска**, выбрав значение **Гибридная рабочая роль**, и выберите имя целевой группы.  В противном случае оставьте значение **Azure** по умолчанию, чтобы тест выполнялся в облаке.
1. Щелкните **Пуск**, чтобы начать тестирование.
1. С помощью кнопок в области вывода можно остановить или приостановить последовательность [runbook рабочего процесса PowerShell](automation-runbook-types.md#powershell-workflow-runbooks) или [графическую последовательность runbook](automation-runbook-types.md#graphical-runbooks) во время ее тестирования. В случае приостановки модуль Runbook завершает действие, начатое до приостановки. Приостановленный модуль Runbook можно остановить или перезапустить.
1. Проверьте выходные данные runbook в области выходных данных.

## <a name="publish-a-runbook"></a>Публикация модуля Runbook

Перед запуском вновь созданного или импортированного модуля Runbook его необходимо опубликовать. У каждой последовательности runbook в службе автоматизации Azure есть черновая и опубликованная версия. Запустить можно только опубликованную версию, а изменить — только черновую. Изменения, внесенные в черновик, не влияют на опубликованную версию. Если требуется черновая версия, ее можно опубликовать, перезаписав текущую опубликованную версию черновой.

### <a name="publish-a-runbook-in-the-azure-portal"></a>Публикация runbook на портале Azure

1. Откройте модуль Runbook на портале Azure.
2. Нажмите кнопку **Изменить**.
3. Нажмите кнопку **Опубликовать**, а в ответ на вопрос в проверочном сообщении — кнопку **Да**.

### <a name="publish-a-runbook-using-powershell"></a>Публикация runbook с помощью PowerShell

Для публикации runbook используйте командлет [Publish-AzAutomationRunbook](/powershell/module/Az.Automation/Publish-AzAutomationRunbook). 

```azurepowershell-interactive
$automationAccountName =  "AutomationAccount"
$runbookName = "Sample_TestRunbook"
$RGName = "ResourceGroup"

Publish-AzAutomationRunbook -AutomationAccountName $automationAccountName `
-Name $runbookName -ResourceGroupName $RGName
```

## <a name="schedule-a-runbook-in-the-azure-portal"></a>Создание расписания для runbook на портале Azure

После публикации последовательности runbook можно запланировать ее выполнение.

1. Откройте модуль Runbook на портале Azure.
2. В разделе **Ресурсы** выберите **Расписания**.
3. Выберите **Добавить расписание**.
4. В области создания расписания runbook выберите **Связать расписание с модулем Runbook**.
5. В области "Расписание" выберите **Создать новое расписание**.
6. Введите имя, описание и другие параметры в области создания расписания.
7. После создания расписания выделите его и нажмите кнопку **ОК**. Теперь оно должно быть связано с последовательностью runbook.
8. Найдите в почтовом ящике сообщение электронной почты с уведомлением о состоянии runbook.

## <a name="obtain-job-statuses"></a>Получение данных о состоянии заданий

### <a name="view-statuses-in-the-azure-portal"></a>Просмотр состояний на портале Azure

Подробные сведения об обработке заданий в службе автоматизации Azure приведены в разделе [Задания](automation-runbook-execution.md#jobs). Когда вы будете готовы просмотреть задания runbook, воспользуйтесь порталом Azure и войдите в учетную запись службы автоматизации. В области **Статистика по заданиям** справа можно просмотреть сводку по всем заданиям runbook.

![Элемент "Статистика по заданиям"](./media/manage-runbooks/automation-account-job-status-summary.png)

В сводке отображается количество и графическое представление состояний всех выполненных заданий.

Щелкните плитку, чтобы открылась страница "Задания", на которой представлен сводный список всех выполненных заданий. На этой странице отображаются сведения о состоянии, имени runbook, времени запуска и времени выполнения каждого задания.

:::image type="content" source="./media/manage-runbooks/automation-account-jobs-status-blade.png" alt-text="Снимок экрана со страницей &quot;задания&quot;.":::

Список заданий можно отфильтровать, выбрав **Фильтровать задания**. Отфильтруйте список по определенной последовательности runbook или состоянию задания либо выберите вариант из раскрывающегося списка и укажите диапазон времени для поиска.

![Фильтрация по состоянию задания](./media/manage-runbooks/automation-account-jobs-filter.png)

Кроме того, для просмотра сводных данных о состоянии заданий определенной последовательности runbook можно выбрать ее модуль на странице "Модули Runbook" учетной записи службы автоматизации, а затем выбрать элемент **Задания**. При выполнении этого действия отображается страница "Задания". Здесь можно щелкнуть запись задания, чтобы просмотреть сведения о ней и ее выходные данные.

:::image type="content" source="./media/manage-runbooks/automation-runbook-job-summary-blade.png" alt-text="Снимок экрана со страницей &quot;задания&quot; с выделенной кнопкой &quot;ошибки&quot;.":::

### <a name="retrieve-job-statuses-using-powershell"></a>Получение данных о состоянии заданий с помощью PowerShell

Чтобы получить список созданных заданий для последовательности runbook и сведения о конкретном задании, воспользуйтесь командлетом [Get-AzAutomationJob](/powershell/module/Az.Automation/Get-AzAutomationJob). Если вы запускаете последовательность runbook с помощью `Start-AzAutomationRunbook`, возвращается результирующее задание. Для получения выходных данных задания используйте командлет [Get-AzAutomationJobOutput](/powershell/module/Az.Automation/Get-AzAutomationJobOutput).

В приведенном ниже примере извлекаются сведения о последнем задании для примера runbook и отображаются данные о его состоянии, значения, заданные для параметров runbook, а также выходные данные этого задания.

```azurepowershell-interactive
$job = (Get-AzAutomationJob –AutomationAccountName "MyAutomationAccount" `
–RunbookName "Test-Runbook" -ResourceGroupName "ResourceGroup01" | sort LastModifiedDate –desc)[0]
$job.Status
$job.JobParameters
Get-AzAutomationJobOutput -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAcct" -Id $job.JobId –Stream Output
```

В следующем примере извлекаются выходные данные для конкретного задания и возвращается каждая запись. Если для одной из записей возникает [исключение](automation-runbook-execution.md#exceptions), скрипт записывает его вместо значения. Это полезно, так как исключения могут предоставлять дополнительные сведения, которые обычно не удается записать в журнал во время вывода.

```azurepowershell-interactive
$output = Get-AzAutomationJobOutput -AutomationAccountName <AutomationAccountName> -Id <jobID> -ResourceGroupName <ResourceGroupName> -Stream "Any"
foreach($item in $output)
{
    $fullRecord = Get-AzAutomationJobOutputRecord -AutomationAccountName <AutomationAccountName> -ResourceGroupName <ResourceGroupName> -JobId <jobID> -Id $item.StreamRecordId
    if ($fullRecord.Type -eq "Error")
    {
        $fullRecord.Value.Exception
    }
    else
    {
    $fullRecord.Value
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения об управлении последовательностями runbook см. в статье [Выполнение runbook в службе автоматизации Azure](automation-runbook-execution.md).
* Сведения о подготовке последовательностей runbook PowerShell см. в статье [Изменение текстовых последовательностей runbook в службе автоматизации Azure](automation-edit-textual-runbook.md).
* Сведения об устранении неполадок, связанных с выполнением последовательности runbook, см. в статье [Устранение ошибок runbook](troubleshoot/runbooks.md).