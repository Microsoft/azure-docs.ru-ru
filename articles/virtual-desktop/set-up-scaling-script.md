---
title: Масштабирование узлов сеансов с помощью службы автоматизации Azure — Azure
description: Автоматическое масштабирование узлов сеансов Виртуального рабочего стола Windows с помощью службы автоматизации Azure.
author: Heidilohr
ms.topic: how-to
ms.date: 03/09/2021
ms.author: helohr
manager: lizross
ms.openlocfilehash: f60341ea51f1cf4e856b1b4598887da3dc37ebb2
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102613125"
---
# <a name="scale-session-hosts-using-azure-automation"></a>Масштабирование узлов сеансов с помощью службы автоматизации Azure

Вы можете сократить общие затраты на развертывание Виртуального рабочего стола Windows путем масштабирования виртуальных машин. Это означает завершение работы и освобождение виртуальных машин узла сеансов в часы непиковой нагрузки и их включение и повторное выделение в часы пиковой нагрузки.

В этой статье вы узнаете о средстве масштабирования, созданном с помощью учетной записи службы автоматизации Azure, и приложения логики Azure, которое автоматически масштабирует виртуальные машины узла сеансов в среде виртуальных рабочих столов Windows. Чтобы узнать, как использовать средство масштабирования, перейдите к разделу [Предварительные требования](#prerequisites).

## <a name="how-the-scaling-tool-works"></a>Принцип работы средства масштабирования

Средство масштабирования предоставляет недорогой вариант автоматизации для клиентов, желающих оптимизировать затраты на виртуальные машины узла сеансов.

Средство масштабирования можно использовать для выполнения следующих задач.

- Планирование запуска и остановки виртуальных машин на основе пиковых и непиковых рабочих часов.
- Увеличение масштаба виртуальных машин на основе числа сеансов на ядро ЦП.
- Уменьшение масштаба виртуальных машин в часы наименьшей нагрузки с минимальным количеством запущенных виртуальных машин.

Средство масштабирования использует сочетание учетной записи службы автоматизации Azure, модуля Runbook PowerShell, веб-перехватчика и приложения логики Azure для работы. При запуске средства Azure Logic App вызывает веб-перехватчик для запуска модуля Runbook службы автоматизации Azure. Затем модуль runbook создает задание.

Во время пикового использования это задание проверяет текущее количество сеансов и емкость виртуальной машины текущего работающего узла сеансов для каждого пула узлов. Эта информация используется для вычисления того, могут ли виртуальные машины, выполняющиеся на узле сеансов, поддерживать существующие сеансы на основе параметра *сессионсрешолдперкпу* , определенного для файла **CreateOrUpdateAzLogicApp.ps1** . Если виртуальные машины узла сеансов не справляются с существующими сеансами, задание запускает в пуле узлов дополнительные виртуальные машины узла сеансов.

>[!NOTE]
>Параметр *SessionThresholdPerCPU* не ограничивает количество сеансов на виртуальной машине. Он определяет только, нужно ли запускать новые виртуальные машины для балансировки нагрузки подключений. Чтобы ограничить количество сеансов, необходимо выполнить инструкции [Update-азввдхостпул](configure-host-pool-load-balancing.md#configure-breadth-first-load-balancing) , чтобы соответствующим образом настроить параметр *макссессионлимит* .

Во время непикового использования задание определяет, сколько виртуальных машин узла сеансов должно быть завершено, в зависимости от параметра *минимумнумберофрдш* . Если для параметра *лимитсекондстофорцелогоффусер* задано ненулевое положительное значение, задание установит для виртуальных машин узла сеанса режим стока, чтобы предотвратить подключение новых сеансов к узлам. После этого задание будет уведомлять всех пользователей, выполнивших вход в систему, сохранить свою работу, подождать заданное время, а затем заставить пользователей выйти из нее. После выхода всех пользовательских сеансов на виртуальной машине узла сеансов задание завершит работу виртуальной машины. После завершения работы виртуальной машины задание сбрасывает режим стока узла сеанса.

>[!NOTE]
>Если вы вручную настроили виртуальную машину узла сеанса в режим стока, задание не будет управлять виртуальной машиной узла сеансов. Если виртуальная машина узла сеансов работает и настроена на режим стока, она будет считаться недоступной, что сделает задание запуском дополнительных виртуальных машин для обработки нагрузки. Мы рекомендуем пометить все виртуальные машины Azure, прежде чем вручную установить их в режим стока. При создании планировщика приложений логики Azure в дальнейшем можно присвоить тегу параметру *маинтенанцетагнаме* . Теги помогут отличать эти виртуальные машины от тех, которыми управляет инструмент масштабирования. Установка тега обслуживания также предотвращает внесение изменений средством масштабирования в виртуальную машину до тех пор, пока не будет удален тег.

Если для параметра *лимитсекондстофорцелогоффусер* задано значение 0, задание разрешает параметр конфигурации сеанса в указанных групповых политиках выполнять выход из сеансов пользователя. Чтобы просмотреть эти групповые политики, перейдите в раздел политики **конфигурации компьютера**  >    >  **Административные шаблоны**  >  **компоненты Windows**  >  **службы удаленных рабочих столов**  >  **Удаленный рабочий стол**  >  **ограничения времени сеанса** узла сеансов. Если на виртуальной машине узла сеансов есть активные сеансы, то задание оставит ее работающей. Если нет активных сеансов, задание завершит работу виртуальной машины узла сеансов.

В любое время задание также принимает учетные записи *макссессионлимит* пула узлов, чтобы определить, превышает ли текущее число сеансов более 90% от максимальной емкости. Если это так, задание запустит дополнительные виртуальные машины узла сеансов.

Задание выполняется периодически в зависимости от заданного интервала повторения. Этот интервал можно изменить в зависимости от размера среды виртуальных рабочих столов Windows, но помните, что запуск и завершение работы виртуальных машин может занять некоторое время, поэтому не забывайте учитывать задержку. Рекомендуется устанавливать интервал повторения раз в 15 минут.

Однако это средство также имеет следующие ограничения.

- Это решение применимо только к виртуальным машинам узла сеансов с несколькими сеансами в составе пула.
- Это решение управляет виртуальными машинами в любом регионе, но может использоваться только в той же подписке, что и учетная запись службы автоматизации Azure и приложение логики Azure.
- Максимальное время выполнения задания в модуле Runbook — 3 часа. Если запуск или остановка виртуальных машин в пуле узлов занимает больше времени, задание завершится ошибкой. Дополнительные сведения см. в разделе [Общие ресурсы](../automation/automation-runbook-execution.md#fair-share).
- Чтобы алгоритм масштабирования работал правильно, необходимо включить хотя бы одну виртуальную машину или узел сеансов.
- Средство масштабирования не поддерживает масштабирование на основе ЦП или памяти.
- Масштабирование работает только с существующими узлами в пуле узлов. Средство масштабирования не поддерживает масштабирование новых узлов сеансов.

>[!NOTE]
>Средство масштабирования управляет режимом балансировки нагрузки для пула узлов, который в настоящее время масштабируется. Для пиковых и непиковых часов средство использует режим балансировки нагрузки уровня "в первый уровень".

## <a name="prerequisites"></a>Предварительные требования

Прежде чем приступить к настройке средства масштабирования, убедитесь, что у вас настроены следующие компоненты.

- [Пул узлов виртуальных рабочих столов Windows](create-host-pools-azure-marketplace.md)
- Виртуальные машины пула узлов сеансов, настроенные и зарегистрированные в службе Виртуального рабочего стола Windows.
- Пользователь с правами [участника](../role-based-access-control/role-assignments-portal.md) в подписке Azure.

На компьютере для развертывания средства должны быть установлены следующие компоненты.

- Windows PowerShell 5.1 или более поздней версии
- Модуль PowerShell Microsoft AZ

Если у вас все готово, давайте приступим.

## <a name="create-or-update-an-azure-automation-account"></a>Создание или обновление учетной записи службы автоматизации Azure

>[!NOTE]
>Если у вас уже есть учетная запись службы автоматизации Azure с модулем Runbook, на котором запущена более старая версия скрипта масштабирования, необходимо выполнить приведенные ниже инструкции, чтобы убедиться в том, что он обновлен.

Сначала вам потребуется учетная запись службы автоматизации Azure для запуска модуля PowerShell runbook. Процесс, описанный в этом разделе, допустим, даже если у вас есть учетная запись службы автоматизации Azure, которую вы хотите использовать для настройки модуля Runbook PowerShell. Настройка приложения

1. Откройте Windows PowerShell.

2. Выполните следующий командлет, чтобы войти в учетную запись Azure.

    ```powershell
    Login-AzAccount
    ```

    >[!NOTE]
    >Ваша учетная запись должна иметь права участника в подписке Azure, в которой вы хотите развернуть Средство масштабирования.

3. Выполните следующий командлет, чтобы скачать сценарий для создания учетной записи службы автоматизации Azure.

    ```powershell
    New-Item -ItemType Directory -Path "C:\Temp" -Force
    Set-Location -Path "C:\Temp"
    $Uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzAutoAccount.ps1"
    # Download the script
    Invoke-WebRequest -Uri $Uri -OutFile ".\CreateOrUpdateAzAutoAccount.ps1"
    ```

4. Выполните следующий командлет, чтобы выполнить скрипт и создать учетную запись службы автоматизации Azure. Можно либо ввести значения для параметров, либо закомментировать их для использования значений по умолчанию.

    ```powershell
    $Params = @{
         "AADTenantId"           = "<Azure_Active_Directory_tenant_ID>"   # Optional. If not specified, it will use the current Azure context
         "SubscriptionId"        = "<Azure_subscription_ID>"              # Optional. If not specified, it will use the current Azure context
         "UseARMAPI"             = $true
         "ResourceGroupName"     = "<Resource_group_name>"                # Optional. Default: "WVDAutoScaleResourceGroup"
         "AutomationAccountName" = "<Automation_account_name>"            # Optional. Default: "WVDAutoScaleAutomationAccount"
         "Location"              = "<Azure_region_for_deployment>"
         "WorkspaceName"         = "<Log_analytics_workspace_name>"       # Optional. If specified, Log Analytics will be used to configure the custom log table that the runbook PowerShell script can send logs to
    }

    .\CreateOrUpdateAzAutoAccount.ps1 @Params
    ```

5. Выходные данные командлета будут содержать URI веб-перехватчика. Обязательно Составьте запись универсального кода ресурса (URI), так как она будет использоваться в качестве параметра при настройке расписания выполнения для приложения логики Azure.

6. Если вы указали параметр **workspacename** для log Analytics, выходные данные командлета также будут содержать идентификатор рабочей области log Analytics и ее первичный ключ. Запомните URI, так как его необходимо будет использовать позже в качестве параметра при настройке расписания выполнения для приложения логики Azure.

7. После настройки учетной записи службы автоматизации Azure войдите в свою подписку Azure и убедитесь, что ваша учетная запись службы автоматизации Azure и соответствующий модуль runbook появлялись в указанной группе ресурсов, как это показано на следующем рисунке.

    >[!div class="mx-imgBorder"]
    >![Изображение страницы "Обзор Azure" с созданной учетной записью службы автоматизации Azure и модулем Runbook.](media/automation-account.png)

    Чтобы проверить, что веб-перехватчик находится на своем месте, выберите имя модуля runbook. Затем перейдите к разделу ресурсов модуля runbook и выберите **Веб-перехватчики**.

## <a name="create-an-azure-automation-run-as-account"></a>Создание учетной записи запуска от имени службы автоматизации Azure

Теперь, когда у вас есть учетная запись службы автоматизации Azure, вам также потребуется создать учетную запись запуска от имени службы автоматизации Azure, если она еще не создана. Эта учетная запись позволит средству получить доступ к ресурсам Azure.

[Учетная запись запуска от имени службы автоматизации Azure](../automation/manage-runas-account.md) обеспечивает проверку подлинности для управления ресурсами в Azure с помощью командлетов Azure. При создании учетной записи запуска от имени создается новый пользователь субъекта-службы в Azure Active Directory и назначается роль участника для пользователя субъекта-службы на уровне подписки. Учетная запись запуска от имени Azure — это отличный способ безопасной проверки подлинности с помощью сертификатов и имени субъекта-службы без необходимости хранить имя пользователя и пароль в объекте учетных данных. Дополнительные сведения о проверке подлинности учетной записи запуска от имени см. в разделе [ограничение разрешений учетной записи запуска от имени](../automation/manage-runas-account.md#limit-run-as-account-permissions).

Любой пользователь, который является членом роли администраторов Подписки и соадминистратором подписки, может создать учетную запись запуска от имени.

Чтобы создать учетную запись запуска от имени в учетной записи службы автоматизации Azure, сделайте следующее:

1. На портале Azure щелкните **Все службы**. В списке ресурсов введите и выберите **учетные записи службы автоматизации**.

2. На странице **учетные записи службы автоматизации** выберите имя учетной записи службы автоматизации Azure.

3. На панели в левой части окна выберите **учетные записи запуска от имени** в разделе **Параметры учетной записи** .

4. Выберите **учетную запись запуска от имени Azure**. Когда появится панель **Добавление учетной записи запуска от имени Azure** , просмотрите Общие сведения и нажмите кнопку **создать** , чтобы начать процесс создания учетной записи.

5. Подождите несколько минут, пока Azure создаст учетную запись запуска от имени. Ход создания можно отслеживать в меню в разделе "Уведомления".

6. По завершении процесса он создаст ресурс с именем **AzureRunAsConnection** в указанной учетной записи службы автоматизации Azure. Выберите **учетную запись запуска от имени Azure**. Этот ресурс содержит идентификаторы приложения, клиента и подписки, а также отпечаток сертификата. Эти же сведения можно найти на странице **подключения** . Чтобы открыть эту страницу, на панели в левой части окна выберите **подключения** в разделе **Общие ресурсы** и щелкните ресурс подключения с именем **AzureRunAsConnection**.

## <a name="create-the-azure-logic-app-and-execution-schedule"></a>Создание приложения логики Azure и расписания выполнения

Наконец, вам потребуется создать приложение логики Azure и настроить расписание выполнения для нового инструмента масштабирования. Сначала скачайте и импортируйте [модуль PowerShell для виртуализации рабочей среды](powershell-module.md) для использования в сеансе PowerShell, если вы этого еще не сделали.

1. Откройте Windows PowerShell.

2. Выполните следующий командлет, чтобы войти в учетную запись Azure.

    ```powershell
    Login-AzAccount
    ```

3. Выполните следующий командлет, чтобы скачать скрипт для создания приложения логики Azure.

    ```powershell
    New-Item -ItemType Directory -Path "C:\Temp" -Force
    Set-Location -Path "C:\Temp"
    $Uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzLogicApp.ps1"
    # Download the script
    Invoke-WebRequest -Uri $Uri -OutFile ".\CreateOrUpdateAzLogicApp.ps1"
    ```

4. Выполните следующий сценарий PowerShell, чтобы создать приложение логики Azure и расписание выполнения для пула узлов.

    >[!NOTE]
    >Вам потребуется выполнить этот скрипт для каждого пула узлов, для которого требуется Автомасштабирование, но требуется только одна учетная запись службы автоматизации Azure.

    ```powershell
    $AADTenantId = (Get-AzContext).Tenant.Id

    $AzSubscription = Get-AzSubscription | Out-GridView -OutputMode:Single -Title "Select your Azure Subscription"
    Select-AzSubscription -Subscription $AzSubscription.Id

    $ResourceGroup = Get-AzResourceGroup | Out-GridView -OutputMode:Single -Title "Select the resource group for the new Azure Logic App"

    $WVDHostPool = Get-AzResource -ResourceType "Microsoft.DesktopVirtualization/hostpools" | Out-GridView -OutputMode:Single -Title "Select the host pool you'd like to scale"

    $LogAnalyticsWorkspaceId = Read-Host -Prompt "If you want to use Log Analytics, enter the Log Analytics Workspace ID returned by when you created the Azure Automation account, otherwise leave it blank"
    $LogAnalyticsPrimaryKey = Read-Host -Prompt "If you want to use Log Analytics, enter the Log Analytics Primary Key returned by when you created the Azure Automation account, otherwise leave it blank"
    $RecurrenceInterval = Read-Host -Prompt "Enter how often you'd like the job to run in minutes, e.g. '15'"
    $BeginPeakTime = Read-Host -Prompt "Enter the start time for peak hours in local time, e.g. 9:00"
    $EndPeakTime = Read-Host -Prompt "Enter the end time for peak hours in local time, e.g. 18:00"
    $TimeDifference = Read-Host -Prompt "Enter the time difference between local time and UTC in hours, e.g. +5:30"
    $SessionThresholdPerCPU = Read-Host -Prompt "Enter the maximum number of sessions per CPU that will be used as a threshold to determine when new session host VMs need to be started during peak hours"
    $MinimumNumberOfRDSH = Read-Host -Prompt "Enter the minimum number of session host VMs to keep running during off-peak hours"
    $MaintenanceTagName = Read-Host -Prompt "Enter the name of the Tag associated with VMs you don't want to be managed by this scaling tool"
    $LimitSecondsToForceLogOffUser = Read-Host -Prompt "Enter the number of seconds to wait before automatically signing out users. If set to 0, any session host VM that has user sessions, will be left untouched"
    $LogOffMessageTitle = Read-Host -Prompt "Enter the title of the message sent to the user before they are forced to sign out"
    $LogOffMessageBody = Read-Host -Prompt "Enter the body of the message sent to the user before they are forced to sign out"

    $AutoAccount = Get-AzAutomationAccount | Out-GridView -OutputMode:Single -Title "Select the Azure Automation account"
    $AutoAccountConnection = Get-AzAutomationConnection -ResourceGroupName $AutoAccount.ResourceGroupName -AutomationAccountName $AutoAccount.AutomationAccountName | Out-GridView -OutputMode:Single -Title "Select the Azure RunAs connection asset"

    $WebhookURIAutoVar = Get-AzAutomationVariable -Name 'WebhookURIARMBased' -ResourceGroupName $AutoAccount.ResourceGroupName -AutomationAccountName $AutoAccount.AutomationAccountName

    $Params = @{
         "AADTenantId"                   = $AADTenantId                             # Optional. If not specified, it will use the current Azure context
         "SubscriptionID"                = $AzSubscription.Id                       # Optional. If not specified, it will use the current Azure context
         "ResourceGroupName"             = $ResourceGroup.ResourceGroupName         # Optional. Default: "WVDAutoScaleResourceGroup"
         "Location"                      = $ResourceGroup.Location                  # Optional. Default: "West US2"
         "UseARMAPI"                     = $true
         "HostPoolName"                  = $WVDHostPool.Name
         "HostPoolResourceGroupName"     = $WVDHostPool.ResourceGroupName           # Optional. Default: same as ResourceGroupName param value
         "LogAnalyticsWorkspaceId"       = $LogAnalyticsWorkspaceId                 # Optional. If not specified, script will not log to the Log Analytics
         "LogAnalyticsPrimaryKey"        = $LogAnalyticsPrimaryKey                  # Optional. If not specified, script will not log to the Log Analytics
         "ConnectionAssetName"           = $AutoAccountConnection.Name              # Optional. Default: "AzureRunAsConnection"
         "RecurrenceInterval"            = $RecurrenceInterval                      # Optional. Default: 15
         "BeginPeakTime"                 = $BeginPeakTime                           # Optional. Default: "09:00"
         "EndPeakTime"                   = $EndPeakTime                             # Optional. Default: "17:00"
         "TimeDifference"                = $TimeDifference                          # Optional. Default: "-7:00"
         "SessionThresholdPerCPU"        = $SessionThresholdPerCPU                  # Optional. Default: 1
         "MinimumNumberOfRDSH"           = $MinimumNumberOfRDSH                     # Optional. Default: 1
         "MaintenanceTagName"            = $MaintenanceTagName                      # Optional.
         "LimitSecondsToForceLogOffUser" = $LimitSecondsToForceLogOffUser           # Optional. Default: 1
         "LogOffMessageTitle"            = $LogOffMessageTitle                      # Optional. Default: "Machine is about to shutdown."
         "LogOffMessageBody"             = $LogOffMessageBody                       # Optional. Default: "Your session will be logged off. Please save and close everything."
         "WebhookURI"                    = $WebhookURIAutoVar.Value
    }

    .\CreateOrUpdateAzLogicApp.ps1 @Params
    ```

    После выполнения скрипта приложение логики Azure должно отобразиться в группе ресурсов, как показано на следующем рисунке.

    >[!div class="mx-imgBorder"]
    >![Изображение страницы обзора для примера приложения логики Azure.](media/logic-app.png)

    Чтобы внести изменения в расписание выполнения, например изменить интервал повторения или часовой пояс, перейдите в планировщик автомасштабирования Azure Logic Apps и выберите **изменить** , чтобы перейти в конструктор приложений логики Azure.

    >[!div class="mx-imgBorder"]
    >![Изображение конструктора приложений логики Azure. Меню повторение и веб-перехватчик позволяют пользователю изменять время повторения и файл веб-перехватчика.](media/logic-apps-designer.png)

## <a name="manage-your-scaling-tool"></a>Управление средством масштабирования

Теперь, когда вы создали средство масштабирования, вы можете получить доступ к его выходным данным. В этом разделе описаны некоторые функции, которые могут оказаться полезными.

### <a name="view-job-status"></a>Просмотр состояния задания

Вы можете просмотреть сводку по состоянию всех заданий runbook или просмотреть подробное состояние определенного задания на портале Azure.

В правой части выбранной учетной записи службы автоматизации Azure в разделе "Статистика заданий" можно просмотреть список сводок всех заданий Runbook. При открытии страницы **Задания** в левой части окна отображаются текущие состояния заданий, время запуска и время завершения.

>[!div class="mx-imgBorder"]
>![Снимок экрана со страницей состояния заданий.](media/jobs-status.png)

### <a name="view-logs-and-scaling-tool-output"></a>Просмотр журналов и выходных данных средства масштабирования

Вы можете просмотреть журналы операций масштабирования и масштабирования, открыв модуль Runbook и выбрав задание.

Перейдите к модулю Runbook в группе ресурсов, в которой размещена учетная запись службы автоматизации Azure, и выберите **Обзор**. На странице Обзор выберите задание в разделе **недавние задания** , чтобы просмотреть его выходные данные средства масштабирования, как показано на следующем рисунке.

>[!div class="mx-imgBorder"]
>![Изображение окна вывода для средства масштабирования.](media/tool-output.png)

### <a name="check-the-runbook-script-version-number"></a>Проверка номера версии скрипта Runbook

Чтобы узнать, какую версию скрипта Runbook вы используете, откройте файл Runbook в учетной записи службы автоматизации Azure и выберите **вид**. Скрипт для модуля Runbook появится в правой части экрана. В сценарии вы увидите номер версии в формате, который находится в `v#.#.#` `SYNOPSIS` разделе. Номер последней версии можно найти [здесь](https://github.com/Azure/RDS-Templates/blob/master/wvd-templates/wvd-scaling-script/ARM_based/basicScale.ps1#L1). Если номер версии в скрипте Runbook не отображается, это означает, что вы используете более раннюю версию скрипта, и вы должны обновить его сразу же. Если вам нужно обновить скрипт Runbook, следуйте инструкциям в статье [Создание или обновление учетной записи службы автоматизации Azure](#create-or-update-an-azure-automation-account).

### <a name="reporting-issues"></a>Создание отчетов о проблемах

При возникновении проблемы необходимо предоставить следующие сведения, которые помогут нам в устранении неполадок:

- Полный журнал на вкладке **все журналы** в задании, вызвавшем ошибку. Чтобы узнать, как получить журнал, следуйте инструкциям в разделе [Просмотр журналов и масштабирование в выходных данных средства масштабирования](#view-logs-and-scaling-tool-output). Если в журнале есть конфиденциальная или частная информация, вы можете удалить ее, прежде чем отправить нам вопрос.

- Версия скрипта Runbook, который вы используете. Чтобы узнать, как получить номер версии, см. раздел [Проверка номера версии скрипта Runbook](#check-the-runbook-script-version-number) .

- Номер версии каждого из следующих модулей PowerShell, установленных в учетной записи службы автоматизации Azure. Чтобы найти эти модули, откройте учетную запись службы автоматизации Azure, выберите **модули** в разделе **Общие ресурсы** на панели в левой части окна, а затем найдите имя модуля.
    - Az.Accounts
    - Az.Compute
    - Az.Resources
    - Az.Automation
    - OMSIngestionAPI
    - Az.DesktopVirtualization

- Дата истечения срока действия [учетной записи запуска от имени](#create-an-azure-automation-run-as-account). Чтобы найти это, откройте учетную запись службы автоматизации Azure, а затем выберите **учетные записи запуска от имени** в разделе **Параметры учетной записи** на панели в левой части окна. Дата окончания срока действия должна быть в **учетной записи запуска от имени Azure**.

### <a name="log-analytics"></a>Log Analytics

Если вы решили использовать Log Analytics, можно просмотреть все данные журнала в пользовательском журнале с именем **WVDTenantScale_CL** в разделе **пользовательские журналы** в представлении **журналы** рабочей области log Analytics. Некоторые примеры запросов, которые могут оказаться полезными, перечислены выше.

- Чтобы просмотреть все журналы для пула узлов, введите следующий запрос.

    ```Kusto
    WVDTenantScale_CL
    | where hostpoolName_s == "<host_pool_name>"
    | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
    ```

- Чтобы просмотреть общее число выполняющихся в настоящее время виртуальных машин узла сеансов и активных пользовательских сеансов в пуле узлов, введите следующий запрос.

    ```Kusto
    WVDTenantScale_CL
    | where logmessage_s contains "Number of running session hosts:"
         or logmessage_s contains "Number of user sessions:"
         or logmessage_s contains "Number of user sessions per Core:"
    | where hostpoolName_s == "<host_pool_name>"
    | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
    ```

- Чтобы просмотреть состояние всех виртуальных машин узла сеансов в пуле узлов, введите следующий запрос.

    ```Kusto
    WVDTenantScale_CL
    | where logmessage_s contains "Session host:"
    | where hostpoolName_s == "<host_pool_name>"
    | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
    ```

- Чтобы просмотреть ошибки и предупреждения, введите следующий запрос.

    ```Kusto
    WVDTenantScale_CL
    | where logmessage_s contains "ERROR:" or logmessage_s contains "WARN:"
    | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
    ```

## <a name="report-issues"></a>Сообщить о проблеме

Отчеты о проблемах для инструмента масштабирования в настоящее время обрабатываются служба поддержки Майкрософт. При создании отчета о проблемах обязательно следуйте инструкциям, приведенным в разделе [отчеты о проблемах](#reporting-issues). Если у вас есть отзывы о средстве или требуется запросить новые функции, откройте ошибку GitHub с меткой "4-ВВД-Scaling-Tool" на [странице GitHub RDS](https://github.com/Azure/RDS-Templates/issues?q=is%3Aissue+is%3Aopen+label%3A4-WVD-scaling-tool).
