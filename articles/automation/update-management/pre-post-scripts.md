---
title: Управление сценариями предварительного и последующего выполнения в развертывании службы "Управление обновлениями" в Azure
description: В этой статье описывается настройка и администрирование сценариев предварительного и последующего выполнения для развертывания обновлений.
services: automation
ms.subservice: update-management
ms.date: 03/08/2021
ms.topic: conceptual
ms.openlocfilehash: ce60c773626d951062de3cc830b898e3b875f3cb
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102485543"
---
# <a name="manage-pre-scripts-and-post-scripts"></a>Управление сценариями предварительного и последующего выполнения

Сценарии предварительного и последующего выполнения представляют собой модули runbook, которые выполняются в учетной записи службы автоматизации до и после задач развертывания обновлений. Сценарии предварительного и последующего выполнения выполняются в контексте Azure, а не в локальной среде. Сценарии предварительного выполнения запускаются в начале развертывания обновлений. Сценарии последующего выполнения запускаются в конце развертывания и после каждой настроенной перезагрузки.

## <a name="pre-script-and-post-script-requirements"></a>Требования к сценариям предварительного и последующего выполнения

Чтобы модуль runbook можно было использовать в качестве сценария предварительного или последующего выполнения, его необходимо импортировать в учетную запись службы автоматизации и [опубликовать как runbook](../manage-runbooks.md#publish-a-runbook).

В настоящее время в качестве скриптов pre и POST поддерживаются только модули Runbook PowerShell и Python 2. Другие типы модулей Runbook, такие как Python 3, графический поток, Рабочий процесс PowerShell, в настоящее время не поддерживаются как скрипты предварительной или публикации.

## <a name="pre-script-and-post-script-parameters"></a>Параметры сценариев предварительного и последующего выполнения

Настроив сценарии предварительного и последующего выполнения, вы сможете передавать в них параметры, как при создании расписания для модуля runbook. Параметры определяются при создании развертывания обновлений. Сценарии предварительного и последующего выполнения поддерживают следующие типы:

* [char]
* [byte]
* [int]
* [long]
* [decimal]
* [single]
* [double]
* [DateTime]
* [string]

В параметрах runbook для сценариев предварительного и последующего выполнения не поддерживаются логические значения, объекты и массивы. Наличие таких значений приводит к сбою выполнения модуля runbook. 

Если вам нужен другой тип объекта, вы можете привести его к другому типу с помощью собственной логики runbook.

Помимо стандартных параметров модулей runbook, поддерживается дополнительный параметр `SoftwareUpdateConfigurationRunContext` (с типом строки JSON). Если этот параметр определен для сценария предварительного или последующего выполнения, в него автоматически передается значение при развертывании обновлений. Этот параметр содержит сведения о развертывании обновлений, то есть подмножество информации, возвращаемой из [API SoftwareUpdateconfigurations](/rest/api/automation/softwareupdateconfigurations/getbyname#updateconfiguration). В следующих разделах описываются соответствующие свойства.

### <a name="softwareupdateconfigurationruncontext-properties"></a>Свойства SoftwareUpdateConfigurationRunContext

|Свойство  |Описание  |
|---------|---------|
|SoftwareUpdateConfigurationName     | Имя конфигурации обновления программного обеспечения.        |
|SoftwareUpdateConfigurationRunId     | Уникальный идентификатор для запуска.        |
|SoftwareUpdateConfigurationSettings     | Коллекция свойств, связанных с конфигурацией обновления программного обеспечения.         |
|SoftwareUpdateConfigurationSettings.operatingSystem     | Операционные системы, на которых будут развертываться обновления.         |
|SoftwareUpdateConfigurationSettings.duration     | Максимальная продолжительность развертывания обновлений (период обслуживания) в формате `PT[n]H[n]M[n]S` по стандарту ISO8601.          |
|SoftwareUpdateConfigurationSettings.Windows     | Коллекция свойств, связанных с компьютерами Windows.         |
|SoftwareUpdateConfigurationSettings.Windows.excludedKbNumbers     | Список номеров обновлений, которые не включаются в развертывание.        |
|SoftwareUpdateConfigurationSettings.Windows.includedUpdateClassifications     | Классификации обновлений, выбранных для развертывания обновлений.        |
|SoftwareUpdateConfigurationSettings.Windows.rebootSetting     | Параметры перезагрузки для развертывания обновлений.        |
|azureVirtualMachines     | Список идентификаторов ресурсов для виртуальных машин Azure, включенных в развертывание обновлений.        |
|nonAzureComputerNames|Список полных доменных имен компьютеров, не связанных с Azure, включенных в развертывание обновлений.|

В следующем примере приведена строка JSON, передаваемая в параметре **SoftwareUpdateConfigurationRunContext**.

```json
"SoftwareUpdateConfigurationRunContext": {
    "SoftwareUpdateConfigurationName": "sampleConfiguration",
    "SoftwareUpdateConfigurationRunId": "00000000-0000-0000-0000-000000000000",
    "SoftwareUpdateConfigurationSettings": {
      "operatingSystem": "Windows",
      "duration": "PT2H0M",
      "windows": {
        "excludedKbNumbers": [
          "168934",
          "168973"
        ],
        "includedUpdateClassifications": "Critical",
        "rebootSetting": "IfRequired"
      },
      "azureVirtualMachines": [
        "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresources/providers/Microsoft.Compute/virtualMachines/vm-01",
        "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresources/providers/Microsoft.Compute/virtualMachines/vm-02",
        "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresources/providers/Microsoft.Compute/virtualMachines/vm-03"
      ],
      "nonAzureComputerNames": [
        "box1.contoso.com",
        "box2.contoso.com"
      ]
    }
  }
```

Полный пример со всеми доступными свойствами можно просмотреть [здесь](/rest/api/automation/softwareupdateconfigurations/getbyname#examples).

> [!NOTE]
> Объект `SoftwareUpdateConfigurationRunContext` может содержать повторяющиеся записи для компьютеров. В этом случае сценарии предварительного и последующего выполнения будут выполнены на одном компьютере несколько раз. Если такое поведение нежелательно, используйте `Sort-Object -Unique` для выбора только уникальных имен виртуальных машин.

## <a name="use-a-pre-script-or-post-script-in-a-deployment"></a>Использование сценариев предварительного и последующего выполнения в развертывании

Чтобы применить сценарий предварительного или последующего выполнения при развертывании обновлений, прежде всего создайте развертывание обновлений. Выберите **Сценарии предварительного и последующего выполнения**. Откроется страница **Выбор сценариев предварительного и последующего выполнения**.

![Выбор сценариев](./media/pre-post-scripts/select-scripts.png)

Выберите сценарий, который вы хотите использовать. В нашем примере используется модуль runbook с именем **UpdateManagement-TurnOnVms**. При выборе модуля runbook открывается страница **Настройка сценария**. Выберите **Сценарий предварительного выполнения** и щелкните **ОК**.

Повторите эту процедуру для сценария **UpdateManagement-TurnOffVms**. Но для параметра **Тип сценария** выберите **Сценарий последующего выполнения**.

Теперь раздел **Выбранные элементы** отображает оба выбранных сценария. Один из них — сценарий предварительного выполнения, а второй — сценарий последующего выполнения.

![Выбранные элементы](./media/pre-post-scripts/selected-items.png)

Завершите настройку развертывания обновлений.

Когда развертывание обновлений будет готово, откройте страницу **Развертывания обновлений** и проверьте результаты. Здесь, среди прочего, предоставляется состояние для обоих сценариев.

![Результаты обновления](./media/pre-post-scripts/update-results.png)

Выбрав запуск развертывания обновлений, вы увидите дополнительные сведения о сценариях предварительного и последующего выполнения. В открывшемся окне будет ссылка на код сценария, который выполнялся.

![Результаты развертывания](./media/pre-post-scripts/deployment-run.png)

## <a name="stop-a-deployment"></a>Остановка развертывания

Если вам придется прерывать развертывание по результатам работы сценария предварительного выполнения, он должен [создавать исключение](../automation-runbook-execution.md#throw). Без этого развертывание продолжится и сценарий последующего выполнения также будет выполнен. В следующем фрагменте кода показано, как создать исключение с помощью PowerShell.

```powershell
#In this case, we want to terminate the patch job if any run fails.
#This logic might not hold for all cases - you might want to allow success as long as at least 1 run succeeds
foreach($summary in $finalStatus)
{
    if ($summary.Type -eq "Error")
    {
        #We must throw in order to fail the patch deployment.
        throw $summary.Summary
    }
}
```

В Python 2 Управление обработкой исключений осуществляется в блоке [try](https://www.python-course.eu/exception_handling.php) .

## <a name="interact-with-machines"></a>Взаимодействие с компьютерами

Предварительные и посторонние скрипты выполняются в учетной записи службы автоматизации в качестве модулей Runbook, а не непосредственно на компьютерах в развертывании. Задачи предварительного и последующего выполнения также работают в контексте Azure и не имеют доступа к компьютерам в других средах. В следующих разделах показано, как организовать взаимодействие с компьютерами напрямую, будь то виртуальные машины Azure или компьютеры, не связанные с Azure.

### <a name="interact-with-azure-machines"></a>Взаимодействие с компьютерами Azure

Задачи предварительного и последующего выполнения работают как модули runbook, а не в среде ОС виртуальных машин Azure в развертывании. Чтобы взаимодействовать с виртуальными машинами Azure, вам потребуется следующее:

* учетная запись запуска от имени;
* модуль runbook, который вам нужно выполнить.

Для взаимодействия с виртуальными машинами Azure следует применять командлет [Invoke-AzVMRunCommand](/powershell/module/az.compute/invoke-azvmruncommand). Такое взаимодействие описано в примере модуля runbook [Управление обновлениями — запуск сценария командой Run](https://github.com/azureautomation/update-management-run-script-with-run-command).

### <a name="interact-with-non-azure-machines"></a>Взаимодействие с компьютерами, не связанными с Azure

Задачи предварительного и последующего выполнения работают в контексте Azure и не имеют доступ к компьютерам в других средах. Чтобы взаимодействовать с компьютерами, не связанными с Azure, вам потребуется следующее:

* учетная запись запуска от имени;
* установленная на компьютере гибридная рабочая роль Runbook;
* модуль Runbook, который будет запускаться локально;
* родительский модуль runbook.

Для взаимодействия с компьютерами, не связанными с Azure, в контексте Azure выполняется родительский модуль runbook. Он вызывает дочерний модуль runbook с помощью командлета [Start-AzAutomationRunbook](/powershell/module/Az.Automation/Start-AzAutomationRunbook). Ему нужно передать параметр `RunOn` и имя гибридной рабочей роли Runbook, которая будет выполнять сценарий. Пример такого модуля runbook см. в статье [Управление обновлениями — локальный запуск сценария](https://github.com/azureautomation/update-management-run-script-locally)

## <a name="abort-patch-deployment"></a>Отмена развертывания обновлений

Если сценарий предварительного выполнения возвращает ошибку, иногда нужно прервать развертывание. Для этого нужно [создавать исключения](/powershell/module/microsoft.powershell.core/about/about_throw) в сценарии по любому поводу, который для вашего развертывания считается сбоем.

```powershell
if (<My custom error logic>)
{
    #Throw an error to fail the patch deployment.
    throw "There was an error, abort deployment"
}
```

В Python 2, если возникает ошибка при возникновении определенного условия, используйте оператор [raise](https://docs.python.org/2.7/reference/simple_stmts.html#the-raise-statement) .

```python
If (<My custom error logic>)
   raise Exception('Something happened.')
```

## <a name="samples"></a>Примеры

Примеры для предварительных сценариев и сценариев публикации можно найти в [Организации Azure Automation GitHub](https://github.com/azureautomation) и [коллекция PowerShell](https://www.powershellgallery.com/packages?q=Tags%3A%22UpdateManagement%22+Tags%3A%22Automation%22), либо можно импортировать их с помощью портал Azure. Для этого в учетной записи службы автоматизации выберите в разделе **Автоматизация процессов** элемент **Коллекция модулей runbook**. В качестве фильтра укажите **Управление обновлениями**.

![Список в коллекции](./media/pre-post-scripts/runbook-gallery.png)

Также вы можете искать сценарии по именам, которые указаны в списке ниже.

* Update Management - Turn On VMs (Управление обновлениями — включить виртуальные машины).
* Update Management - Turn Off VMs (Управление обновлениями — выключить виртуальные машины).
* Update Management - Run Script Locally (Управление обновлениями — локальный запуск сценария).
* Update Management - Template for Pre/Post Scripts (Управление обновлениями — шаблон для скриптов предварительного и последующего выполнения).
* Update Management - Run Script with Run Command (Управление обновлениями — запуск сценария командой Run).

> [!IMPORTANT]
> Чтобы воспользоваться импортированным модулем runbook, его нужно опубликовать. Для этого найдите нужный модуль runbook в учетной записи службы автоматизации, а затем щелкните **Изменить** и **Опубликовать**.

Все примеры основаны на базовом шаблоне, который представлен в примере ниже. Этот шаблон можно использовать для создания собственных модулей runbook, которые можно применять в сценариях предварительного и последующего выполнения. В шаблон уже включена вся необходимая логика для проверки подлинности в Azure и обработки параметра `SoftwareUpdateConfigurationRunContext`.

```powershell
<#
.SYNOPSIS
 Barebones script for Update Management Pre/Post

.DESCRIPTION
  This script is intended to be run as a part of Update Management pre/post-scripts.
  It requires a RunAs account.

.PARAMETER SoftwareUpdateConfigurationRunContext
  This is a system variable which is automatically passed in by Update Management during a deployment.
#>

param(
    [string]$SoftwareUpdateConfigurationRunContext
)
#region BoilerplateAuthentication
#This requires a RunAs account
$ServicePrincipalConnection = Get-AutomationConnection -Name 'AzureRunAsConnection'

Add-AzAccount `
    -ServicePrincipal `
    -TenantId $ServicePrincipalConnection.TenantId `
    -ApplicationId $ServicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint

$AzureContext = Select-AzSubscription -SubscriptionId $ServicePrincipalConnection.SubscriptionID
#endregion BoilerplateAuthentication

#If you wish to use the run context, it must be converted from JSON
$context = ConvertFrom-Json  $SoftwareUpdateConfigurationRunContext
#Access the properties of the SoftwareUpdateConfigurationRunContext
$vmIds = $context.SoftwareUpdateConfigurationSettings.AzureVirtualMachines | Sort-Object -Unique
$runId = $context.SoftwareUpdateConfigurationRunId

Write-Output $context

#Example: How to create and write to a variable using the pre-script:
<#
#Create variable named after this run so it can be retrieved
New-AzAutomationVariable -ResourceGroupName $ResourceGroup –AutomationAccountName $AutomationAccount –Name $runId -Value "" –Encrypted $false
#Set value of variable
Set-AutomationVariable –Name $runId -Value $vmIds
#>

#Example: How to retrieve information from a variable set during the pre-script
<#
$variable = Get-AutomationVariable -Name $runId
#>
```

> [!NOTE]
> Для неграфических runbook PowerShell `Add-AzAccount` и `Add-AzureRMAccount` являются псевдонимами для [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Вы можете использовать эти командлеты или [обновить модули](../automation-update-azure-modules.md) в учетной записи службы автоматизации до последних версий. Обновление модулей может потребоваться, даже если учетная запись службы автоматизации только что создана.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об управлении обновлениями см. [в статье Управление обновлениями и исправлениями для виртуальных машин](manage-updates-for-vm.md).
