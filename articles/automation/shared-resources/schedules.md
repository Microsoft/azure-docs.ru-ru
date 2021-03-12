---
title: Управление расписаниями в службе автоматизации Azure
description: В этой статье рассказывается, как создать расписание в службе автоматизации Azure и работать с ним.
services: automation
ms.subservice: shared-capabilities
ms.date: 09/10/2020
ms.topic: conceptual
ms.openlocfilehash: f40e3d555d6e1472b9d2368a114ee27d588f6383
ms.sourcegitcommit: 6776f0a27e2000fb1acb34a8dddc67af01ac14ac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103149483"
---
# <a name="manage-schedules-in-azure-automation"></a>Управление расписаниями в службе автоматизации Azure

Чтобы запускать модуль Runbook в определенное время с помощью расписания в службе автоматизации Azure, его необходимо привязать к одному или нескольким расписаниям. На портале Azure расписание для модулей runbook можно настроить для однократного, ежечасного или ежедневного выполнения. Можно также запланировать еженедельное и ежемесячное выполнение, а также выполнение в определенные дни недели или месяца. Один модуль Runbook может быть связан с несколькими расписаниями, и одно расписание может иметь несколько привязанных модулей Runbook.

> [!NOTE]
> Служба автоматизации Azure поддерживает летнее время и учитывает его в расписании.

> [!NOTE]
> В настоящее время расписания не поддерживают конфигурации Azure Automation DSC.

## <a name="powershell-cmdlets-used-to-access-schedules"></a>Командлеты PowerShell, используемые для доступа к расписаниям

Командлеты, представленные в следующей таблице, используются для создания расписаний автоматизации с помощью PowerShell и управления ими. Они поставляются в составе [модулей Az](modules.md#az-modules).

| Командлеты | Описание |
|:--- |:--- |
| [Get-AzAutomationSchedule](/powershell/module/Az.Automation/Get-AzAutomationSchedule) |Получает расписание. |
| [Get-AzAutomationScheduledRunbook](/powershell/module/az.automation/get-azautomationscheduledrunbook) |Получает расписание модулей Runbook. |
| [New-AzAutomationSchedule](/powershell/module/Az.Automation/New-AzAutomationSchedule) |Создает новое расписание. |
| [Register-AzAutomationScheduledRunbook](/powershell/module/Az.Automation/Register-AzAutomationScheduledRunbook) |Связывает Runbook с расписанием. |
| [Remove-AzAutomationSchedule](/powershell/module/Az.Automation/Remove-AzAutomationSchedule) |Удаляет расписание. |
| [Set-AzAutomationSchedule](/powershell/module/Az.Automation/Set-AzAutomationSchedule) |Задает свойства для существующего расписания. |
| [Unregister-AzAutomationScheduledRunbook](/powershell/module/Az.Automation/Unregister-AzAutomationScheduledRunbook) |Отменяет связь Runbook с расписанием. |

## <a name="create-a-schedule"></a>Создание расписания

Новое расписание для модулей runbook вы можете создать на портале Azure или с помощью PowerShell. Чтобы избежать влияния на модули runbook и автоматизируемые ими процессы, необходимо сначала проверить все модули runbook, которые имеют связанные расписания, используя учетную запись автоматизации, предназначенную для тестирования. Тест проверяет, продолжают ли запланированные модули runbook работать правильно. Если вы видите проблему, вы можете устранить неполадки и применить все необходимые изменения перед переносом обновленной версии runbook в рабочую среду.

> [!NOTE]
> Учетная запись автоматизации не получает новые версии модулей автоматически, если их не обновить вручную, выбрав параметр [Обновить модули Azure](../automation-update-azure-modules.md) в области **Модули**. При запуске нового запланированного задания служба автоматизации Azure использует модули последней версии в вашей учетной записи службы автоматизации. 

### <a name="create-a-new-schedule-in-the-azure-portal"></a>Создание нового расписания на портале Azure

1. В учетной записи службы автоматизации в области слева выберите **расписания** в разделе **Общие ресурсы**.
2. На странице **расписания** выберите **Добавить расписание**.
3. На странице **Создание расписания** введите имя и при необходимости введите описание нового расписания.

    >[!NOTE]
    >В настоящее время расписания автоматизации не поддерживают использование специальных символов в имени расписания.
    >

4. Выберите **однократное** или **повторяющееся** расписание, которое будет выполняться однократно или по расписанию. Если вы выбрали режим **Однократно**, то укажите время начала и нажмите кнопку **Создать**. При выборе значения **Периодически** укажите время начала. Для значения **Повторять каждые** выберите частоту повторения модуля Runbook. Выберите час, день, неделю или месяц.

    * Если вы выбрали **Неделя**, отобразится список дней недели на выбор. Выберите столько дней, сколько требуется. Первый запуск расписания будет происходить на первый день после времени начала. Например, чтобы выбрать расписание для выходных дней, выберите "Суббота" и "Воскресенье".

    ![Задание повторяющегося расписания для выходных дней](../media/schedules/week-end-weekly-recurrence.png)

    * Если вы выбрали **Месяц**, у вас есть различные варианты. В разделе **Ежемесячное повторение** выберите **дни месяца** или **дни недели**. Если вы выбрали **дни месяца**, появится календарь, в котором можно выбрать любое количество дней. Если выбранная дата, например 31, не наступает в текущем месяце, расписание не будет выполняться. Чтобы запустить его в последний день, выберите **Да** в поле **Последний день месяца**. Если вы выберете **дни недели**, появляется параметр **Повторять каждые**. Выберите **первый**, **второй**, **третий**, **четвертый**, или **последний**. Наконец, выберите день для повторения.

    ![Ежемесячное расписание на первый, пятнадцатый и последний день месяца](../media/schedules/monthly-first-fifteenth-last.png)

5. По завершении нажмите кнопку **Создать**.

### <a name="create-a-new-schedule-with-powershell"></a>Создание расписания с помощью PowerShell

Используйте командлет [New-AzAutomationSchedule](/powershell/module/Az.Automation/New-AzAutomationSchedule) для создания расписаний. Укажите для расписания время начала и частоту выполнения. В следующих примерах показано, как создать множество различных сценариев расписания.

>[!NOTE]
>В настоящее время расписания автоматизации не поддерживают использование специальных символов в имени расписания.
>

#### <a name="create-a-one-time-schedule"></a>Создание разового расписания

В следующем примере создается разовое расписание

```azurepowershell-interactive
$TimeZone = ([System.TimeZoneInfo]::Local).Id
New-AzAutomationSchedule -AutomationAccountName "ContosoAutomation" -Name "Schedule01" -StartTime "23:00" -OneTime -ResourceGroupName "ResourceGroup01" -TimeZone $TimeZone
```

#### <a name="create-a-recurring-schedule"></a>Создание повторяющегося расписания

В следующем примере показано, как создать повторяющееся расписание, которое выполняется каждый день в 13:00 в течение года.

```azurepowershell-interactive
$StartTime = Get-Date "13:00:00"
$EndTime = $StartTime.AddYears(1)
New-AzAutomationSchedule -AutomationAccountName "ContosoAutomation" -Name "Schedule02" -StartTime $StartTime -ExpiryTime $EndTime -DayInterval 1 -ResourceGroupName "ResourceGroup01"
```

#### <a name="create-a-weekly-recurring-schedule"></a>Создание еженедельного повторяющегося расписания

В следующем примере показано, как создать еженедельное расписание, выполняемое только по рабочим дням.

```azurepowershell-interactive
$StartTime = (Get-Date "13:00:00").AddDays(1)
[System.DayOfWeek[]]$WeekDays = @([System.DayOfWeek]::Monday..[System.DayOfWeek]::Friday)
New-AzAutomationSchedule -AutomationAccountName "ContosoAutomation" -Name "Schedule03" -StartTime $StartTime -WeekInterval 1 -DaysOfWeek $WeekDays -ResourceGroupName "ResourceGroup01"
```

#### <a name="create-a-weekly-recurring-schedule-for-weekends"></a>Создание еженедельного повторяющегося расписания по выходным дням

В следующем примере показано, как создать еженедельное расписание, выполняемое только по выходным дням.

```azurepowershell-interactive
$StartTime = (Get-Date "18:00:00").AddDays(1)
[System.DayOfWeek[]]$WeekendDays = @([System.DayOfWeek]::Saturday,[System.DayOfWeek]::Sunday)
New-AzAutomationSchedule -AutomationAccountName "ContosoAutomation" -Name "Weekends 6PM" -StartTime $StartTime -WeekInterval 1 -DaysOfWeek $WeekendDays -ResourceGroupName "ResourceGroup01"
```

#### <a name="create-a-recurring-schedule-for-the-first-fifteenth-and-last-days-of-the-month"></a>Создание повторяющегося расписания для первого, пятнадцатого и последнего дней месяца

В следующем примере показано, как создать повторяющееся расписание, выполняемое в первый, пятнадцатый и последний день месяца.

```azurepowershell-interactive
$StartTime = (Get-Date "18:00:00").AddDays(1)
New-AzAutomationSchedule -AutomationAccountName "TestAzureAuto" -Name "1st, 15th and Last" -StartTime $StartTime -DaysOfMonth @("One", "Fifteenth", "Last") -ResourceGroupName "TestAzureAuto" -MonthInterval 1
```

## <a name="create-a-schedule-with-a-resource-manager-template"></a>Создание расписания с помощью шаблона диспетчер ресурсов

В этом примере мы используем шаблон автоматизации диспетчер ресурсов (ARM), который создает новое расписание заданий. Общие сведения об этом шаблоне для управления расписаниями заданий службы автоматизации см. в [справочнике по шаблонам Microsoft. Automation automationAccounts/жобсчедулес](/templates/microsoft.automation/automationaccounts/jobschedules#quickstart-templates).

Скопируйте этот файл шаблона в текстовый редактор:

```json
{
  "name": "5d5f3a05-111d-4892-8dcc-9064fa591b96",
  "type": "Microsoft.Automation/automationAccounts/jobSchedules",
  "apiVersion": "2015-10-31",
  "properties": {
    "schedule": {
      "name": "scheduleName"
    },
    "runbook": {
      "name": "runbookName"
    },
    "runOn": "hybridWorkerGroup",
    "parameters": {}
  }
}
```

Измените следующие значения параметров и сохраните шаблон как JSON-файл:

* Имя объекта расписания задания: идентификатор GUID (глобальный уникальный идентификатор) используется в качестве имени объекта расписания задания.

   >[!IMPORTANT]
   > Для каждого расписания заданий, развернутого с помощью шаблона ARM, идентификатор GUID должен быть уникальным. Даже при перепланировании существующего расписания необходимо изменить идентификатор GUID. Это применимо даже в том случае, если ранее было удалено существующее расписание заданий, созданное с помощью того же шаблона. Повторное использование одного и того же идентификатора GUID приводит к неудачному развертыванию.</br></br>
   > В сети есть службы, которые могут создать новый GUID, например, [Бесплатный генератор GUID в Интернете](https://guidgenerator.com/).

* Имя расписания: представляет имя расписания заданий службы автоматизации, которое будет связано с указанным модулем Runbook.
* Имя Runbook: представляет имя модуля Runbook службы автоматизации, с которым должно быть связано расписание задания.

После сохранения файла можно создать расписание задания Runbook с помощью следующей команды PowerShell. Команда использует параметр, `TemplateFile` чтобы указать путь и имя файла шаблона.

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "ContosoEngineering" -TemplateFile "<path>\RunbookJobSchedule.json"
```

## <a name="link-a-schedule-to-a-runbook"></a>Связывание расписания с модулем runbook

Один модуль Runbook может быть связан с несколькими расписаниями, и одно расписание может иметь несколько привязанных модулей Runbook. Если runbook имеет параметры, для них можно указать значения. Необходимо предоставить значения для всех обязательных параметров, а также значения для необязательных параметров. Значения применяются каждый раз при запуске модуля Runbook с помощью расписания. Один и тот же модуль Runbook можно привязать к другому расписанию и указать другие значения параметров.

### <a name="link-a-schedule-to-a-runbook-with-the-azure-portal"></a>Связывание расписания с модулем runbook с помощью портала Azure

1. На портале Azure в учетной записи автоматизации выберите **Модули runbook** в разделе **Автоматизация процессов**.
1. Щелкните имя одного модуля runbook для привязки к расписанию.
1. Если модуль runbook сейчас не связан с расписанием, вам будет предложено создать новое расписание или связать модуль с существующим расписанием.
1. Если модуль runbook имеет параметры, следует выбрать действие **Изменить параметры запуска (по умолчанию: Azure)** , чтобы открыть область **Параметры**. Здесь можно ввести сведения о параметрах.

### <a name="link-a-schedule-to-a-runbook-with-powershell"></a>Связывание расписания с модулем runbook с помощью PowerShell

Вы можете использовать командлет [Register-AzAutomationScheduledRunbook](/powershell/module/Az.Automation/Register-AzAutomationScheduledRunbook), чтобы связать расписание. С помощью параметра "Параметры" можно указать значения параметров для модуля Runbook. Дополнительные сведения об указании значений параметров см. в статье [Запуск модуля runbook в службе автоматизации Azure](../start-runbooks.md).
Приведенные ниже примеры демонстрируют, как связать расписание с модулем runbook, используя командлет управления службами Azure с параметрами.

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$runbookName = "Test-Runbook"
$scheduleName = "Sample-DailySchedule"
$params = @{"FirstName"="Joe";"LastName"="Smith";"RepeatCount"=2;"Show"=$true}
Register-AzAutomationScheduledRunbook –AutomationAccountName $automationAccountName `
–Name $runbookName –ScheduleName $scheduleName –Parameters $params `
-ResourceGroupName "ResourceGroup01"
```

## <a name="schedule-runbooks-to-run-more-frequently"></a>Планирование более частого выполнения модулей runbook

Самая большая частота расписания в службе автоматизации Azure, которую можно настроить, составляет один час. Если требуется, чтобы расписания выполнялись чаще, есть два варианта.

* Создайте [веб-перехватчик](../automation-webhooks.md) для модуля runbook и используйте [Azure Logic Apps](../../logic-apps/logic-apps-overview.md) для вызова веб-перехватчика. Azure Logic Apps обеспечивает более детализированную настройку для определения расписания.

* Создайте четыре расписания, которые будут запускаться каждые 15 минут в течение часа. Этот сценарий позволяет модулю runbook запускаться каждые 15 минут с разными расписаниями.

## <a name="disable-a-schedule"></a>Отключение расписания

Если отключить расписание, модуль runbook, связанный с ним, больше не будет запускаться по такому расписанию. Можно отключить расписание вручную или указать срок действия при создании периодических расписаний. Когда срок действия истекает, расписание отключается.

### <a name="disable-a-schedule-from-the-azure-portal"></a>Отключение расписания на портале Azure

1. В учетной записи службы автоматизации в области слева выберите **расписания** в разделе **Общие ресурсы**.
1. Щелкните имя расписания, чтобы открыть область подробных сведений.
1. Задайте значение **Нет** для параметра **Включено**.

> [!NOTE]
> Если вы хотите отключить расписание со временем начала в прошлом, необходимо изменить дату начала на время в будущем перед сохранением.

### <a name="disable-a-schedule-with-powershell"></a>Отключение расписания с помощью PowerShell

Вы можете использовать командлет [Set-AzAutomationSchedule](/powershell/module/Az.Automation/Set-AzAutomationSchedule), чтобы изменить свойства имеющегося расписания. Чтобы отключить расписание, укажите значение false для параметра `IsEnabled`.

В следующем примере показано, как отключить расписание для runbook, используя командлет Azure Resource Manager.

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$scheduleName = "Sample-MonthlyDaysOfMonthSchedule"
Set-AzAutomationSchedule –AutomationAccountName $automationAccountName `
–Name $scheduleName –IsEnabled $false -ResourceGroupName "ResourceGroup01"
```

## <a name="remove-a-schedule"></a>Удаление расписания

Когда будете готовы удалить расписания, можно использовать портал Azure или PowerShell. Помните, что можно удалить только расписание, которое было отключено, как описано в предыдущем разделе.

### <a name="remove-a-schedule-using-the-azure-portal"></a>Удаление расписания с помощью портала Azure

1. В учетной записи службы автоматизации в области слева выберите **расписания** в разделе **Общие ресурсы**.
2. Щелкните имя расписания, чтобы открыть область подробных сведений.
3. Щелкните **Удалить**.

### <a name="remove-a-schedule-with-powershell"></a>Удаление расписания с помощью PowerShell

Для удаления существующего расписания можно использовать командлет `Remove-AzAutomationSchedule`, как показано ниже.

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$scheduleName = "Sample-MonthlyDaysOfMonthSchedule"
Remove-AzAutomationSchedule -AutomationAccountName $automationAccountName `
-Name $scheduleName -ResourceGroupName "ResourceGroup01"
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о командлетах, используемых для доступа к расписаниям, см. в разделе [Управление модулями в службе автоматизации Azure](modules.md).
* Общие сведения о модулях runbook см. в разделе [Выполнение runbook в службе автоматизации Azure](../automation-runbook-execution.md).