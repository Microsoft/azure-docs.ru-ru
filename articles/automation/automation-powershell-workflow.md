---
title: Рабочий процесс PowerShell для службы автоматизации Azure
description: В этой статье рассматриваются различия между PowerShell и рабочим процессом PowerShell, а также понятия, относящиеся к модулям runbook службы автоматизации.
services: automation
ms.subservice: process-automation
ms.date: 12/14/2018
ms.topic: conceptual
ms.openlocfilehash: 483d4c16f1b77bf7328857eb25e1571a741d144e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98896924"
---
# <a name="learn-powershell-workflow-for-azure-automation"></a>Рабочий процесс PowerShell для службы автоматизации Azure

Модули runbook в службе автоматизации Azure реализованы в виде рабочих процессов Windows PowerShell, то есть скриптов Windows PowerShell, использующих Windows Workflow Foundation. Рабочий процесс — это последовательность связанных программируемых операций, в ходе которых выполняются длительные задачи или скоординированные действия на нескольких устройствах или управляемых узлах. 

Хотя рабочий процесс определяется в синтаксисе Windows PowerShell и запускается средой Windows PowerShell, его обработка выполняется в Windows Workflow Foundation. Преимущества рабочего процесса в сравнении с использованием обычного скрипта заключаются в возможности одновременного выполнения действия применительно ко многим устройствам и автоматического восстановления после сбоев. 

> [!NOTE]
> Скрипт рабочего процесса PowerShell очень похож на скрипт Windows PowerShell, но имеет ряд существенных отличий, которые могут запутать нового пользователя. Поэтому рекомендуется создавать модули runbook с помощью рабочего процесса PowerShell только в том случае, если необходимо использовать [контрольные точки](#use-checkpoints-in-a-workflow). 

Дополнительные сведения по темам, рассмотренным в этой статье, см. на странице [Общие сведения о рабочем процессе Windows PowerShell](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj134242(v=ws.11)).

## <a name="use-workflow-keyword"></a>Использование ключевого слова Workflow

Первый шаг к преобразованию скрипта PowerShell в рабочий процесс PowerShell — это добавление ключевого слова `Workflow`. Рабочий процесс начинается с ключевого слова `Workflow`, за которым следует текст скрипта, заключенный в фигурные скобки. Имя рабочего процесса следует за ключевым словом `Workflow`, как это показано в следующем синтаксисе:

```powershell
Workflow Test-Workflow
{
    <Commands>
}
```

Имя рабочего процесса должно соответствовать имени модуля Runbook в службе автоматизации. При импорте модуля runbook имя файла должно соответствовать имени рабочего процесса и заканчиваться на **.ps1**.

Чтобы добавить параметры в рабочий процесс, используйте ключевое слово `Param` (как и в случае скрипта).

## <a name="learn-differences-between-powershell-workflow-code-and-powershell-script-code"></a>Сведения о различиях между кодом рабочего процесса PowerShell и кодом скрипта PowerShell

Код рабочего процесса PowerShell выглядит почти так же, как код скрипта PowerShell, но с некоторыми существенными изменениями. В следующих разделах описываются изменения, которые нужно внести в сценарий PowerShell, чтобы он выполнялся как рабочий процесс.

### <a name="activities"></a>Действия

Действие — это конкретная задача в рабочем процессе, которая выполняется в свою очередь. Рабочий процесс Windows PowerShell автоматически преобразует множество командлетов Windows PowerShell в действия при выполнении рабочего процесса. Если указать один из этих командлетов в модуле runbook, соответствующее действие запускается в Windows Workflow Foundation. 

Если у командлета нет соответствующего действия, рабочий процесс Windows PowerShell автоматически запускает командлет в действии [InlineScript](#use-inlinescript). Некоторые командлеты исключаются и не могут использоваться в рабочем процессе, если они явно не включены в блок InlineScript. Дополнительные сведения см. в статье [Использование действий в рабочих процессах сценариев](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj574194(v=ws.11)).

Действия рабочих процессов совместно используют набор общих параметров для настройки их работы. См. раздел [about_WorkflowCommonParameters](/powershell/module/psworkflow/about/about_workflowcommonparameters).

### <a name="positional-parameters"></a>Позиционные параметры

Позиционные параметры нельзя использовать с действиями и командлетами в рабочем процессе. Поэтому необходимо использовать имена параметров. Рассмотрим следующий код, получающий список всех запущенных служб:

```azurepowershell-interactive
Get-Service | Where-Object {$_.Status -eq "Running"}
```

Попытавшись выполнить этот код в рабочем процессе, вы получите сообщение следующего вида: `Parameter set cannot be resolved using the specified named parameters.`. Чтобы устранить эту проблему, укажите имя параметра, как в следующем примере:

```powershell
Workflow Get-RunningServices
{
    Get-Service | Where-Object -FilterScript {$_.Status -eq "Running"}
}
```

### <a name="deserialized-objects"></a>Десериализованные объекты

Объекты в рабочих процессах десериализируются. Это означает, что их свойства по-прежнему доступны, а методы — нет.  Рассмотрим, например, приведенный ниже код PowerShell, который останавливает службу с помощью метода `Stop` объекта `Service`.

```azurepowershell-interactive
$Service = Get-Service -Name MyService
$Service.Stop()
```

При попытке выполнить этот код в рабочем процессе вы получите сообщение об ошибке `Method invocation is not supported in a Windows PowerShell Workflow.`

Один вариант — перенести эти две строки кода в блок [InlineScript](#use-inlinescript). В этом случае `Service` представляет объект службы в блоке.

```powershell
Workflow Stop-Service
{
    InlineScript {
        $Service = Get-Service -Name MyService
        $Service.Stop()
    }
}
```

Второй путь — использовать другой командлет, который выполняет те же функции, что и метод, если такой командлет существует. В нашем примере командлет `Stop-Service` выполняет те же функции, что и метод `Stop`, поэтому для рабочего процесса можно использовать приведенный ниже код.

```powershell
Workflow Stop-MyService
{
    $Service = Get-Service -Name MyService
    Stop-Service -Name $Service.Name
}
```

## <a name="use-inlinescript"></a>Использование InlineScript

Действие `InlineScript` пригодится, если вам нужно выполнить одну или несколько команд, используя не рабочий процесс PowerShell, а традиционный скрипт PowerShell.  Пока команды в рабочем процессе отправляются в Windows Workflow Foundation для обработки, команды в блоке InlineScript обрабатываются при помощи Windows PowerShell.

InlineScript использует описанный ниже синтаксис.

```powershell
InlineScript
{
    <Script Block>
} <Common Parameters>
```

Для получения выходных данных из InlineScript можно присвоить им переменную. Код в приведенном ниже примере останавливает службу и выдает ее имя.

```powershell
Workflow Stop-MyService
{
    $Output = InlineScript {
        $Service = Get-Service -Name MyService
        $Service.Stop()
        $Service
    }

    $Output.Name
}
```

Значения можно передавать в блок InlineScript, но при этом необходимо использовать модификатор области **$Using** .  Приведенный ниже пример идентичен предыдущему за исключением того, что имя службы предоставляется переменной.

```powershell
Workflow Stop-MyService
{
    $ServiceName = "MyService"

    $Output = InlineScript {
        $Service = Get-Service -Name $Using:ServiceName
        $Service.Stop()
        $Service
    }

    $Output.Name
}
```

В то время как выполнение действий InlineScript может быть критически важно в некоторых рабочих процессах, они не поддерживают конструкции рабочих процессов. Их следует использовать только тогда, когда это необходимо по следующим причинам:

* [Контрольные точки](#use-checkpoints-in-a-workflow) в блоке InlineScript не используются. Если в блоке происходит сбой, его выполнение должно быть возобновлено с начала блока.
* [Параллельное выполнение](#use-parallel-processing) в блоке InlineScript не используется.
* InlineScript влияет на масштабируемость рабочего процесса, поскольку содержит сеанс Windows PowerShell для всей продолжительности блока InlineScript.

Дополнительные сведения об использовании InlineScript см. в статьях [Выполнение команд Windows PowerShell в рабочем процессе](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj574197(v=ws.11)) и [about_InlineScript](/powershell/module/psworkflow/about/about_inlinescript).

## <a name="use-parallel-processing"></a>Использование параллельной обработки

Одним из преимуществ рабочих процессов Windows PowerShell является возможность выполнять набор команд параллельно, а не последовательно, как это делается в стандартном скрипте.

Можно использовать ключевое слово `Parallel`, чтобы создать блок скрипта с несколькими командами, которые выполняются параллельно. Для этого используется описанный ниже синтаксис. В этом случае действия Activity1 и Activity2 запускаются одновременно. Действие Activity3 запускается только после завершения действий Activity1 и Activity2.

```powershell
Parallel
{
    <Activity1>
    <Activity2>
}
<Activity3>
```

Рассмотрим, например, приведенные ниже команды PowerShell, которые копируют несколько файлов в определенный узел сети. Команды выполняются последовательно, поэтому следующий файл не копируется, пока не закончится копирование предыдущего.

```azurepowershell-interactive
Copy-Item -Path C:\LocalPath\File1.txt -Destination \\NetworkPath\File1.txt
Copy-Item -Path C:\LocalPath\File2.txt -Destination \\NetworkPath\File2.txt
Copy-Item -Path C:\LocalPath\File3.txt -Destination \\NetworkPath\File3.txt
```

Приведенный ниже рабочий процесс выполняет те же команды параллельно, так что все файлы копируются одновременно.  При этом сообщение о завершении отображается только после того, как все файлы будут скопированы.

```powershell
Workflow Copy-Files
{
    Parallel
    {
        Copy-Item -Path "C:\LocalPath\File1.txt" -Destination "\\NetworkPath"
        Copy-Item -Path "C:\LocalPath\File2.txt" -Destination "\\NetworkPath"
        Copy-Item -Path "C:\LocalPath\File3.txt" -Destination "\\NetworkPath"
    }

    Write-Output "Files copied."
}
```

Можно использовать конструкцию `ForEach -Parallel` для обработки команд для каждого элемента в коллекции одновременно. Элементы в коллекции обрабатываются параллельно, а команды в блоке скрипта выполняются последовательно. Для этого процесса используется описанный ниже синтаксис. В этом случае действие Activity1 запускается одновременно для всех элементов в коллекции. Для каждого элемента действие Activity2 запускается после завершения действия Activity1. Действие Activity3 запускается только после завершения действий Activity1 и Activity2 для всех элементов. Мы используем параметр `ThrottleLimit`, чтобы ограничить параллелизм. Слишком высокое значение параметра `ThrottleLimit` может вызвать проблемы. Оптимальное значение для параметра `ThrottleLimit` зависит от многих факторов вашей среды. Начните с низкого значения и попытайтесь дискретно наращивать его, пока не найдете подходящее для вашей среды.

```powershell
ForEach -Parallel -ThrottleLimit 10 ($<item> in $<collection>)
{
    <Activity1>
    <Activity2>
}
<Activity3>
```

Приведенный ниже пример аналогичен предыдущему: файлы копируются параллельно.  В данном случае после копирования каждого файла отображается отдельное сообщение.  При этом сообщение об окончательном завершении отображается только после того, как все файлы будут скопированы.

```powershell
Workflow Copy-Files
{
    $files = @("C:\LocalPath\File1.txt","C:\LocalPath\File2.txt","C:\LocalPath\File3.txt")

    ForEach -Parallel -ThrottleLimit 10 ($File in $Files)
    {
        Copy-Item -Path $File -Destination \\NetworkPath
        Write-Output "$File copied."
    }

    Write-Output "All files copied."
}
```

> [!NOTE]
> Не рекомендуется запускать дочерние модули Runbook параллельно, так как обычно этот приводит к недостоверным результатам. Выходные данные дочернего модуля Runbook могут не отображаться, а параметры одного дочернего модуля Runbook могут влиять на параметры других параллельно выполняемых дочерних модулей Runbook. Такие переменные, как `VerbosePreference`, `WarningPreference` и другие, могут не распространяться на дочерние модули runbook. Кроме того, при изменении этих значений в дочернем модуле runbook они могут не восстанавливаться должным образом после вызова.

## <a name="use-checkpoints-in-a-workflow"></a>Использование контрольных точек в рабочем процессе

Контрольная точка представляет собой моментальный снимок текущего состояния рабочего процесса, который включает текущие значения переменных и любые выходные данные, созданные для этой точки. Если рабочий процесс завершается ошибкой или приостанавливается, при следующем запуске он будет выполняться не с начала, а с последней контрольной точки. 

Контрольную точку для рабочего процесса можно установить с помощью действия `Checkpoint-Workflow`. В службе автоматизации Azure есть функция [справедливого распределения](automation-runbook-execution.md#fair-share), благодаря которой все модули runbook, выполнявшиеся три часа, выгружаются, чтобы могли быть выполнены другие модули runbook. Со временем выгруженный модуль runbook загружается повторно. В этом случае его выполнение возобновляется с последней созданной контрольной точки.

Чтобы модуль runbook в конечном итоге был выполнен, необходимо добавлять контрольные точки с интервалами не более трех часов. Если новая контрольная точка добавляется при каждом запуске, а модуль runbook исключается по истечении трех часов из-за ошибки, его выполнение будет возобновляться бесконечно.

В приведенном ниже примере действие Activity2 вызывает остановку рабочего процесса, в связи с чем выдается исключение. После возобновления рабочий процесс начинается с действия Activity2, поскольку оно идет сразу за последней установленной контрольной точкой.

```powershell
<Activity1>
Checkpoint-Workflow
<Activity2>
<Exception>
<Activity3>
```

Устанавливайте контрольные точки в рабочем процессе после действий, которые могут выдавать исключения и которые не должны повторяться при возобновлении рабочего процесса. Допустим, рабочий процесс создает виртуальную машину. Контрольную точку можно создать как до, так и после команд создания виртуальной машины. Если создать виртуальную машину не удастся, то при повторном запуске рабочего процесса команды повторятся. Если рабочий процесс завершится неудачей после создания виртуальной машины, то при возобновлении она не будет создаваться заново.

Код в приведенном ниже примере копирует несколько файлов в определенный узел сети и устанавливает контрольную точку после каждого файла.  Если подключение к этому узлу утеряно, рабочий процесс завершатся ошибкой.  При повторном запуске он возобновляется с последней контрольной точки. Пропускаются только те файлы, которые уже были скопированы.

```powershell
Workflow Copy-Files
{
    $files = @("C:\LocalPath\File1.txt","C:\LocalPath\File2.txt","C:\LocalPath\File3.txt")

    ForEach ($File in $Files)
    {
        Copy-Item -Path $File -Destination \\NetworkPath
        Write-Output "$File copied."
        Checkpoint-Workflow
    }

    Write-Output "All files copied."
}
```

Учетные данные пользователя не сохраняются после вызова действия [Suspend-Workflow](/powershell/module/psworkflow/about/about_suspend-workflow) или последней контрольной точки. Поэтому для учетных данных необходимо указать значение NULL, а затем извлечь их снова из хранилища ресурсов после вызова `Suspend-Workflow` или контрольной точки.  Иначе можно получить сообщение об ошибке `The workflow job cannot be resumed, either because persistence data could not be saved completely, or saved persistence data has been corrupted. You must restart the workflow.`.

Приведенный ниже код показывает, как это сделать в модулях runbook рабочих процессов PowerShell.

```powershell
workflow CreateTestVms
{
    $Cred = Get-AzAutomationCredential -Name "MyCredential"
    $null = Connect-AzAccount -Credential $Cred

    $VmsToCreate = Get-AzAutomationVariable -Name "VmsToCreate"

    foreach ($VmName in $VmsToCreate)
        {
        # Do work first to create the VM (code not shown)

        # Now add the VM
        New-AzVM -VM $Vm -Location "WestUs" -ResourceGroupName "ResourceGroup01"

        # Checkpoint so that VM creation is not repeated if workflow suspends
        $Cred = $null
        Checkpoint-Workflow
        $Cred = Get-AzAutomationCredential -Name "MyCredential"
        $null = Connect-AzAccount -Credential $Cred
        }
}
```

> [!NOTE]
> Для неграфических модулей runbook PowerShell `Add-AzAccount` и `Add-AzureRMAccount` являются псевдонимами для [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Вы можете использовать эти командлеты или [обновить модули](automation-update-azure-modules.md) в учетной записи службы автоматизации до последних версий. Обновление модулей может потребоваться, даже если учетная запись службы автоматизации только что создана. Использовать эти командлеты необязательно, если вы проходите проверку подлинности с помощью учетной записи запуска от имени, настроенной с помощью субъекта-службы.

Дополнительные сведения о контрольных точках см. в статье [Добавление контрольных точек в рабочий процесс сценария](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj574114(v=ws.11)).

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о модулях runbook для рабочих процессов PowerShell см. в статье [Руководство. Создание модуля runbook рабочего процесса PowerShell](learn/automation-tutorial-runbook-textual.md).
