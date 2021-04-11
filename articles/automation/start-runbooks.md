---
title: Запуск модуля Runbook в службе автоматизации Azure
description: В этой статье рассказывается, как запустить последовательность runbook в службе автоматизации Azure.
services: automation
ms.subservice: process-automation
ms.date: 03/16/2018
ms.topic: conceptual
ms.openlocfilehash: 3a08d727f5257f48ba45fc132f493d33bbb0848f
ms.sourcegitcommit: d23602c57d797fb89a470288fcf94c63546b1314
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106169374"
---
# <a name="start-a-runbook-in-azure-automation"></a>Запуск модуля Runbook в службе автоматизации Azure

Следующая таблица поможет определить метод запуска модуля runbook в службе автоматизации Azure, наиболее подходящий для конкретной ситуации. Данная статья содержит сведения о запуске модуля Runbook с помощью портала Azure и Windows PowerShell. Сведения об остальных методах приведены в другой документации, доступной по ссылкам, которые указаны ниже.

| **Метод** | **Характеристики** |
| --- | --- |
| [Портал Azure](#start-a-runbook-with-the-azure-portal) |<li>Простейший способ с интерактивным пользовательским интерфейсом.<br> <li>Форма для предоставления значений простых параметров.<br> <li>Легко отслеживать состояние задания.<br> <li>Доступ с проверкой подлинности с помощью входа в Azure. |
| [Windows PowerShell](/powershell/module/azurerm.automation/start-azurermautomationrunbook) |<li>Вызов из командной строки с помощью командлетов Windows PowerShell.<br> <li>Может включаться в автоматизированную функцию из нескольких шагов.<br> <li>Запрос проходит проверку подлинности с помощью сертификата или по протоколу OAuth.<br> <li>Необходимо предоставить значения простых и сложных параметров.<br> <li>Отслеживание состояния задания.<br> <li>Для поддержки командлетов PowerShell необходим клиент. |
| [API-интерфейс в службе автоматизации Azure](/rest/api/automation/) |<li>Наиболее гибкий и наиболее сложный метод.<br> <li>Вызов из любого пользовательского кода, который может выполнять HTTP-запросы.<br> <li>Запрос проходит проверку подлинности с помощью сертификата или по протоколу OAuth.<br> <li>Необходимо предоставить значения простых и сложных параметров. *При вызове модуля Runbook Python с помощью API полезные данные JSON необходимо сериализовать.*<br> <li>Отслеживание состояния задания. |
| [Объекты Webhook](automation-webhooks.md) |<li>Запуск Runbook из одного HTTP-запроса.<br> <li>Проверка подлинности с помощью токена безопасности в URL-адресе.<br> <li>Клиент не может переопределять значения параметров, указанные при создании веб-перехватчика. Runbook может определить один параметр, который содержит сведения об HTTP-запросе.<br> <li>Отсутствует возможность отслеживать состояния задания через URL-адрес webhook. |
| [Ответ на оповещение Azure](../azure-monitor/alerts/alerts-overview.md) |<li>Запуск Runbook в ответ на оповещение Azure.<br> <li>Настройка веб-перехватчика для Runbook и ссылки на оповещение.<br> <li>Проверка подлинности с помощью токена безопасности в URL-адресе. |
| [Расписание](./shared-resources/schedules.md) |<li>Автоматический запуск Runbook по ежечасному, ежедневному, еженедельному или ежемесячному расписанию.<br> <li>Управление расписанием с помощью портала Azure, командлетов PowerShell или API-интерфейса Azure.<br> <li>Предоставление значений параметров, которые будут использованы в расписании. |
| [Из другого модуля Runbook](automation-child-runbooks.md) |<li>Использование одного модуля Runbook в качестве процесса в другом модуле Runbook.<br> <li>Подходит для функций, используемых в нескольких модулях Runbook.<br> <li>Предоставление значений параметров для дочернего модуля Runbook и использование выходных данных в родительском модуле Runbook. |

На следующем рисунке показан подробный пошаговый процесс жизненного цикла модуля Runbook. Он включает различные способы запуска модуля Runbook в службе автоматизации Azure, компоненты, необходимые для выполнения модулей Runbook для службы автоматизации Azure в гибридной рабочей роли Runbook, и взаимодействие между различными компонентами. Дополнительные сведения о выполнении модулей Runbook службы автоматизации Azure в центре обработки данных см. в статье о [гибридных рабочих ролях Runbook](automation-hybrid-runbook-worker.md).

![Архитектура Runbook](media/automation-starting-runbook/runbooks-architecture.png)

## <a name="work-with-runbook-parameters"></a>Работа с параметрами последовательности runbook

При запуске модуля runbook с помощью портала управления Azure или Windows PowerShell инструкция отправляется через веб-службу автоматизации Azure. Эта служба не поддерживает параметры со сложными типами данных. Если необходимо указать значение для сложного параметра, его необходимо вызвать из другого модуля Runbook, как описано в разделе [Дочерние модули Runbook в службе автоматизации Azure](automation-child-runbooks.md).

Веб-служба автоматизации Azure предоставляет специальные функции для параметров, в которых используются определенные типы данных, как описано в следующих разделах.

### <a name="named-values"></a>Именованные значения

Если параметр имеет тип данных [object], используйте следующий формат JSON, чтобы передать в параметр список именованных значений: *{имя1:'значение1', имя2:'значение2', имя3:'значение3'}* . Значения должны быть простого типа. Модуль runbook получает параметр в виде объекта [PSCustomObject](/dotnet/api/system.management.automation.pscustomobject), свойства которого соответствуют каждому именованному значению.

Рассмотрим следующий тестовый модуль Runbook, который принимает параметр user.

```powershell
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][object]$user
   )
    $userObject = $user | ConvertFrom-JSON
    if ($userObject.Show) {
        foreach ($i in 1..$userObject.RepeatCount) {
            $userObject.FirstName
            $userObject.LastName
        }
    }
}
```

Для параметра user может быть использован следующий текст.

```json
{FirstName:'Joe',LastName:'Smith',RepeatCount:'2',Show:'True'}
```

Результат должен быть таким:

```output
Joe
Smith
Joe
Smith
```

### <a name="arrays"></a>Массивы

Если параметр представляет собой массив, например [array] или [string[]], используйте следующий формат JSON, чтобы отправить его в виде списка значений: *[значение1, значение2, значение3]* . Значения должны быть простого типа.

Рассмотрим следующий тестовый модуль Runbook, который принимает параметр *user*.

```powershell
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][array]$user
   )
    if ($user[3]) {
        foreach ($i in 1..$user[2]) {
            $ user[0]
            $ user[1]
        }
    }
}
```

Для параметра user может быть использован следующий текст.

```input
["Joe","Smith",2,true]
```

Результат должен быть таким:

```output
Joe
Smith
Joe
Smith
```

### <a name="credentials"></a>Учетные данные

Если параметр относится к типу данных `PSCredential`, можно предоставить имя [ресурса учетных данных](./shared-resources/credentials.md) в службе автоматизации Azure. Модуль runbook получит учетные данные с указанным именем. Рассмотрим следующую тестовую последовательность runbook, которая принимает параметр `credential`.

```powershell
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][PSCredential]$credential
   )
   $credential.UserName
}
```

Следующий текст можно использовать для параметра user, если используется ресурс учетной записи `My Credential`.

```input
My Credential
```

Предполагая, что в учетных данных указано имя пользователя `jsmith`, будут отображаться следующие выходные данные:

```output
jsmith
```

## <a name="start-a-runbook-with-the-azure-portal"></a>Запуск модуля Runbook с помощью портала Azure

1. На портале Azure выберите **Служба автоматизации**, а затем выберите имя учетной записи службы автоматизации.
2. В меню концентратора выберите **Модули Runbook**.
3. На странице "Модули Runbook" выберите runbook и щелкните **Запустить**.
4. Если модуль Runbook содержит параметры, вам будет предложено указать значения в текстовом поле для каждого параметра. Дополнительные сведения о параметрах см. в разделе [Параметры модуля Runbook](#work-with-runbook-parameters).
5. В области "Задание" можно просмотреть состояние задания runbook.

## <a name="start-a-runbook-with-powershell"></a>Запуск модуля Runbook с помощью PowerShell

Чтобы запустить последовательность runbook с помощью Windows PowerShell, воспользуйтесь командлетом [Start-AzureRmAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook). В следующем примере кода будет запущена последовательность runbook с именем **Test-Runbook**.

```azurepowershell-interactive
Start-AzAutomationRunbook -AutomationAccountName "MyAutomationAccount" -Name "Test-Runbook" -ResourceGroupName "ResourceGroup01"
```

Командлет `Start-AzAutomationRunbook` возвращает объект задания, с помощью которого можно отслеживать состояние runbook после запуска. Затем этот объект задания можно указать в командлете [Get-AzureAutomationJob](/powershell/module/Az.Automation/Get-AzAutomationJob), чтобы определить состояние задания, и в командлете Get-[AzureAutomationJobOutput](/powershell/module/az.automation/get-azautomationjoboutput), чтобы получить выходные данные. В следующем примере будет запущена последовательность runbook с именем **Test-Runbook**, а после ее завершения выводится результат.

```azurepowershell-interactive
$runbookName = "Test-Runbook"
$ResourceGroup = "ResourceGroup01"
$AutomationAcct = "MyAutomationAccount"

$job = Start-AzAutomationRunbook -AutomationAccountName $AutomationAcct -Name $runbookName -ResourceGroupName $ResourceGroup

$doLoop = $true
While ($doLoop) {
   $job = Get-AzAutomationJob -AutomationAccountName $AutomationAcct -Id $job.JobId -ResourceGroupName $ResourceGroup
   $status = $job.Status
   $doLoop = (($status -ne "Completed") -and ($status -ne "Failed") -and ($status -ne "Suspended") -and ($status -ne "Stopped"))
}

Get-AzAutomationJobOutput -AutomationAccountName $AutomationAcct -Id $job.JobId -ResourceGroupName $ResourceGroup -Stream Output
```

Если для модуля Runbook требуются параметры, необходимо предоставить их в виде [хэш-таблицы](/powershell/module/microsoft.powershell.core/about/about_hash_tables). Ключ хэш-таблицы должен совпадать с именем параметра, а значение — со значением параметра. В следующем примере показано, как запустить модуль Runbook с двумя строковыми параметрами FirstName и LastName, целочисленным параметром RepeatCount и логическим параметром Show. Дополнительные сведения о параметрах см. в разделе [Параметры модуля Runbook](#work-with-runbook-parameters).

```azurepowershell-interactive
$params = @{"FirstName"="Joe";"LastName"="Smith";"RepeatCount"=2;"Show"=$true}
Start-AzAutomationRunbook -AutomationAccountName "MyAutomationAccount" -Name "Test-Runbook" -ResourceGroupName "ResourceGroup01" -Parameters $params
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения об управлении последовательностями runbook см. в статье [Управление модулями runbook в службе автоматизации Azure](manage-runbooks.md).
* Дополнительные сведения о PowerShell см. в [документации по PowerShell](/powershell/scripting/overview).
* Сведения об устранении неполадок, связанных с выполнением последовательности runbook, см. в статье [Устранение ошибок runbook](troubleshoot/runbooks.md).
