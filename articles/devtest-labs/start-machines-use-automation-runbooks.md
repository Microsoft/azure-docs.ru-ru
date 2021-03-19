---
title: Запуск компьютеров с помощью модулей Runbook службы автоматизации в Azure DevTest Labs
description: Узнайте, как запускать виртуальные машины в лаборатории в Azure DevTest Labs с помощью модулей Runbook службы автоматизации Azure.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 231e79d594aab7c59fa21f9ee512abaa9ac67043
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87282268"
---
# <a name="start-virtual-machines-in-a-lab-in-order-by-using-azure-automation-runbooks"></a>Запуск виртуальных машин в лаборатории по порядку с помощью модулей Runbook службы автоматизации Azure
Функция [автозапуска](devtest-lab-set-lab-policy.md#set-autostart) в DevTest Labs позволяет настроить автоматический запуск виртуальных машин в указанное время. Однако эта функция не поддерживает запуск компьютеров в определенном порядке. Существует несколько сценариев, в которых такой тип автоматизации будет полезен.  Один из сценариев заключается в том, где необходимо сначала запустить виртуальную машину Jumpbox в лаборатории, прежде чем другие виртуальные машины, так как Jumpbox используется в качестве точки доступа к другим виртуальным машинам.  В этой статье показано, как настроить учетную запись службы автоматизации Azure с помощью модуля Runbook PowerShell, который выполняет скрипт. Сценарий использует теги на виртуальных машинах в лаборатории, чтобы управлять порядком загрузки без необходимости изменения сценария.

## <a name="setup"></a>Настройка
В этом примере для виртуальных машин в лаборатории необходимо добавить тег **стартупордер** с соответствующим значением (0, 1, 2 и т. д.). Назначьте любой компьютер, который не нужно запускать как-1.

## <a name="create-an-azure-automation-account"></a>Создание учетной записи службы автоматизации Azure
Создайте учетную запись службы автоматизации Azure, следуя инструкциям в [этой статье](../automation/automation-create-standalone-account.md). При создании учетной записи выберите параметр **учетные записи запуска от имени** . После создания учетной записи службы автоматизации откройте страницу **модули** и в строке меню выберите **обновить модули Azure** . Модули по умолчанию имеют несколько версий Old и без обновления скрипт может не работать.

## <a name="add-a-runbook"></a>Добавление модуля Runbook
Теперь, чтобы добавить модуль Runbook в учетную запись службы автоматизации, выберите **модули Runbook** в меню слева. Выберите **Добавить Runbook** в меню и следуйте инструкциям по [созданию модуля Runbook PowerShell](../automation/learn/automation-tutorial-runbook-textual-powershell.md).

## <a name="powershell-script"></a>Сценарий PowerShell
Следующий скрипт принимает имя подписки, имя лаборатории в качестве параметров. Поток сценария заключается в том, чтобы получить все виртуальные машины в лаборатории, а затем проанализировать сведения о теге, чтобы создать список имен виртуальных машин и их порядок запуска. Сценарий проходит через виртуальные машины по порядку и запускает виртуальные машины. Если в определенном порядковом номере есть несколько виртуальных машин, они запускаются асинхронно с помощью заданий PowerShell. Для тех виртуальных машин, у которых нет тега, задайте для параметра Startup значение Last (10). по умолчанию они будут запускаться последними.  Если лаборатория не хочет, чтобы виртуальная машина была запущена автозапуском, установите значение тега 11, и оно будет пропущено.

```powershell
#Requires -Version 3.0
#Requires -Module AzureRM.Resources

param
(
    [Parameter(Mandatory=$false, HelpMessage="Name of the subscription that has the lab")]
    [string] $SubscriptionName,

    [Parameter(Mandatory=$false, HelpMessage="Lab name")]
    [string] $LabName
)

# Connect and add the appropriate subscription
$Conn = Get-AutomationConnection -Name AzureRunAsConnection

Add-AzureRMAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationID $Conn.ApplicationId -Subscription $SubscriptionName -CertificateThumbprint $Conn.CertificateThumbprint

# Find the lab
$dtLab = Find-AzResource -ResourceType 'Microsoft.DevTestLab/labs' -ResourceNameEquals $LabName

# Get the VMs
$dtlAllVms = New-Object System.Collections.ArrayList
$AllVMs = Get-AzResource -ResourceId "$($dtLab.ResourceId)/virtualmachines" -ApiVersion 2016-05-15

# Get the StartupOrder tag, if missing set to be run last (10)
ForEach ($vm in $AllVMs) {
    if ($vm.Tags) {
        if ($vm.Tags['StartupOrder']) {
            $startupValue = $vm.Tags['StartupOrder']
        } else {
            $startupValue = 10
        }
        } else {
            $startupValue = 10
        }
        $dtlAllVms.Add(@{$vm.Name = $startupValue}) > $null
}

# Setup for the async multiple vm start

# Save profile
$profilePath = Join-Path $env:Temp "profile.json"
If (Test-Path $profilePath){
    Remove-Item $profilePath
}
Save-AzContext -Path $profilePath

# Job to start VMs asynch
$startVMBlock = {
    Param($devTestLab,$vmToStart,$profilePath)
    Import-AzContext -Path ($profilePath)
    Invoke-AzResourceAction `
        -ResourceId "$($devTestLab.ResourceId)/virtualmachines/$vmToStart" `
        -Action Start `
        -Force
    Write-Output "Started: $vmToStart"
}

$current = 0
# Start in order from 0 to 10

While ($current -le 10) {
# Get the VMs in the current stage
    $tobeStarted = $null
    $tobeStarted = $dtlAllVms | Where-Object { $_.Values -eq $current}
    if ($tobeStarted.Count -eq 1) {
        # Run sync – jobs not necessary for a single VM
        $returnStatus = Invoke-AzResourceAction `
                -ResourceId "$($dtLab.ResourceId)/virtualmachines/$($tobeStarted.Keys)" `
                -Action Start `
                -Force
        Write-Output "$($tobeStarted.Keys) status: $($returnStatus.status)"
    } elseif ($tobeStarted.Count -gt 1) {
        # Start multiple VMs async
        $jobs = @()
        Write-Output "Start Jobs start: $(Get-Date)"
        
        # Jobs
        $jobs += Start-Job -ScriptBlock $startVMBlock -ArgumentList $dtLab, $($singlevm.Keys), $profilePath
        Write-Output "Start Jobs end: $(Get-Date)"
    }

    # Get results from all jobs
    if($jobs.Count -ne 0) {
        Write-Output "Receive Jobs start: $(Get-Date)"
        foreach ($job in $jobs){
            $jobResult = Receive-Job -Job $job -Wait | Write-Output
        }
        Remove-Job -Job $jobs -Force
    }
    else
    {
        Write-Output "Information: No jobs available"
    }
}
```

## <a name="create-a-schedule"></a>Создание расписания
Чтобы этот сценарий выполнялся ежедневно, [Создайте расписание](../automation/shared-resources/schedules.md#create-a-schedule) в учетной записи службы автоматизации. После создания расписания [свяжите его с модулем Runbook](../automation/shared-resources/schedules.md#link-a-schedule-to-a-runbook). 

В крупномасштабной ситуации, когда существует несколько подписок с несколькими лабораториями, храните сведения о параметрах в файле для разных лабораторий и передайте файл в сценарий вместо отдельных параметров. Сценарий потребуется изменить, но основное выполнение будет таким же. Хотя в этом примере для выполнения сценария PowerShell используется служба автоматизации Azure, существуют и другие параметры, такие как использование задачи в конвейере сборки или выпуска.

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о службе автоматизации Azure см. в следующей статье: [Введение в службу автоматизации Azure](../automation/automation-intro.md).
