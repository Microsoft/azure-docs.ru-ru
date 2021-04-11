---
title: Создание последовательностей runbook PowerShell в службе автоматизации Azure
description: Эта статья содержит инструкции по созданию, тестированию и публикации простой последовательности runbook PowerShell.
keywords: Azure PowerShell, руководство по скриптам PowerShell, инструменты автоматизации PowerShell
services: automation
ms.subservice: process-automation
ms.date: 04/19/2020
ms.topic: tutorial
ms.openlocfilehash: 0e5573634edf29698e7b118487fb2ec4e22e8eec
ms.sourcegitcommit: d23602c57d797fb89a470288fcf94c63546b1314
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106169442"
---
# <a name="tutorial-create-a-powershell-runbook"></a>Руководство по Создание модуля runbook в PowerShell

В этом руководстве описана процедура создания [модуля Runbook PowerShell](../automation-runbook-types.md#powershell-runbooks) в службе автоматизации Azure. Модули Runbook PowerShell используют Windows PowerShell. Код модуля Runbook можно редактировать в текстовом редакторе на портале Azure.

> [!div class="checklist"]
> * Создание простой последовательности runbook PowerShell
> * Тестирование и публикация runbook
> * Выполнение задания модуля runbook и отслеживание его состояния
> * Обновление runbook для запуска виртуальной машины Azure с указанными в runbook параметрами

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником требуется:

* Подписка Azure. Если у вас ее нет, [активируйте преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) или [зарегистрируйте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Учетная запись службы автоматизации](../automation-quickstart-create-account.md) , чтобы хранить модуль Runbook и выполнять проверку подлинности ресурсов Azure. Эта учетная запись должна иметь разрешение на запуск и остановку виртуальной машины.
* Виртуальная машина Azure. Это не должна быть рабочая виртуальная машина, так как для работы с этим руководством ее нужно остановить и запустить заново.
* Если потребуется, [импортируйте](../shared-resources/modules.md) или [обновите](../automation-update-azure-modules.md) модули Azure для используемых командлетов.

## <a name="differences-from-powershell-workflow-runbooks"></a>Отличия от последовательности runbook рабочего процесса PowerShell

Последовательности runbook PowerShell похожи на последовательности runbook рабочего процесса PowerShell в контексте жизненного цикла, возможностей и средств управления. Но есть и некоторые отличия и ограничения.

| Характеристика  | Последовательности runbook PowerShell | Последовательности runbook рабочего процесса PowerShell |
| ------ | ----- | ----- |
| Speed | Выполняются быстрее благодаря отсутствию этапа компиляции. | Выполняются медленнее. |
| Контрольные точки | Контрольные точки не поддерживаются. Последовательность runbook PowerShell может возобновить работу только с самого начала. | Используют контрольные точки, которые позволяют возобновить работу рабочей книги с любой точки. |
| Выполнение команд | Поддерживают только последовательное выполнение. | Поддерживают последовательное и параллельное выполнение.|
| Пространство выполнения | Единое пространство выполнения объединяет все операции скрипта. | Можно использовать отдельное пространство выполнения для действия, команды или блока скриптов. |

Кроме этих отличий, последовательности runbook PowerShell и рабочих процессов PowerShell имеют ряд [синтаксических отличий](/previous-versions/technet-magazine/dn151046(v=msdn.10)).

## <a name="step-1---create-runbook"></a>Шаг 1. Создание нового модуля Runbook

Для начала вы создадите простой модуль runbook, который выводит текст `Hello World`.

1. На портале Azure выберите свою учетную запись службы автоматизации.

2. Щелкните **Модули Runbook** в разделе **Автоматизация процессов**, чтобы открыть список модулей runbook.

3. Создайте новый модуль runbook с помощью команды **Создать runbook**.

4. Присвойте модулю Runbook имя **MyFirstRunbook-PowerShell**.

5. В этом примере вы создадите [последовательность runbook PowerShell](../automation-runbook-types.md#powershell-runbooks). Для параметра **Тип runbook** выберите значение **PowerShell**.

6. Щелкните **Создать** , чтобы создать модуль Runbook и открыть текстовый редактор.

## <a name="step-2---add-code-to-the-runbook"></a>Шаг 2. Добавление кода в Runbook

Можно либо напрямую ввести код в модуль Runbook, либо выбрать командлеты, модули Runbook и ресурсы из элемента управления "Библиотека" и добавить их к модулю Runbook с любыми связанными параметрами. В этом руководстве показано, как вводить код прямо в runbook.

1. Сейчас эта последовательность runbook пустая. Введите `Write-Output "Hello World"` в текст скрипта.

   ![Hello World](../media/automation-tutorial-runbook-textual-powershell/automation-helloworld.png)

2. Сохраните модуль Runbook, щелкнув **Сохранить**.

## <a name="step-3---test-the-runbook"></a><a name="step-3---test-the-runbook"> </a> Шаг 3. Тестирование последовательностей runbook

Прежде чем опубликовать модуль runbook и сделать его доступным в рабочей среде, необходимо проверить, правильно ли он работает. Тестирование runbook выполняется по черновой версии и позволяет проверить выходные данные в интерактивном режиме.

1. Щелкните **Область тестирования**.

2. Щелкните **Пуск** , чтобы начать тестирование. Активным должен быть только этот параметр.

3. Вы увидите, что будет создан объект [задания runbook](../automation-runbook-execution.md) и на панели отобразится его состояние.

   Сначала задание получает состояние "В очереди". Это означает, что задание ожидает доступности рабочей роли runbook в облаке. Как только рабочая роль затребует задание, оно получит состояние "Запуск". Когда модуль runbook начнет выполнение, состояние изменится на "Выполняется".

4. Когда задание runbook будет выполнено, на панели тестирования отобразится результат выполнения. В нашем примере отображается результат `Hello World`.

   ![Вывод области тестирования](../media/automation-tutorial-runbook-textual-powershell/automation-testpane-output.png)

5. Закройте область тестирования, чтобы вернуться на холст.

## <a name="step-4---publish-and-start-the-runbook"></a>Шаг 4. Публикация и запуск модуля Runbook

Модуль runbook, который вы создали, все еще используется в черновом режиме. Прежде чем запустить модуль в рабочей среде, его нужно опубликовать. При публикации модуля Runbook существующая опубликованная версия перезаписывается черновой версией. В нашем случае опубликованной версии не существует, поскольку модуль runbook создан только что.

1. Щелкните **Опубликовать**, чтобы опубликовать модуль Runbook, а затем нажмите кнопку **Да** в появившемся запросе.

2. Прокрутите содержимое влево, чтобы просмотреть модуль runbook на странице модулей runbook, и убедитесь, что параметр **Состояние создания** имеет значение **Опубликовано**.

3. Прокрутите обратно вправо до страницы объекта **MyFirstRunbook-PowerShell**.
   
   Параметры в верхней части экрана позволяют немедленно запустить модуль runbook, запланировать его запуск на определенное время в будущем или создать [веб-перехватчик](../automation-webhooks.md) для запуска модуля runbook с помощью HTTP-вызова.

4. Чтобы запустить runbook, выберите **Запуск** и нажмите кнопку **Да** в появившемся запросе. 

5. Откроется область заданий с заданием runbook, которое вы только что создали. Эту область можно закрыть, но лучше пока оставить ее открытой, чтобы следить за ходом выполнения задания. В области **Сводка по заданию** отображается состояние задания, где возможные варианты совпадают с теми, которые описаны для тестирования последовательности runbook.

   ![Сводные данные задания](../media/automation-tutorial-runbook-textual-powershell/job-pane-status-blade-jobsummary.png)

6. Когда состояние задания изменится на "Завершено", щелкните элемент **Выходные данные**, чтобы открыть панель "Выходные данные", где отображается `Hello World`.

   ![Выходные данные задания](../media/automation-tutorial-runbook-textual-powershell/job-pane-status-blade-outputtile.png)

7. Закройте страницу выходных данных.

8. Щелкните **Все журналы** , чтобы открыть область "Потоки" для задания Runbook. В потоке выходных данных должно отобразиться только `Hello World`.

    Но не забывайте, что на панели "Потоки" могут присутствовать и другие потоки задания runbook, например потоки "Подробные сведения" и "Ошибка", если этот модуль runbook записывает в них какие-то данные.

   ![Все журналы](../media/automation-tutorial-runbook-textual-powershell/job-pane-status-blade-alllogstile.png)

9. Закройте панели "Потоки" и "Задания", чтобы вернуться к странице MyFirstRunbook-PowerShell.

10. В разделе **Сведения** щелкните **Задания**, чтобы открыть область "Задания" для этой последовательности runbook. Откроется страница со списком всех заданий, созданных этим модулем runbook. Список должен содержать одно задание, так как вы запустили задание только один раз.

   ![Список заданий](../media/automation-tutorial-runbook-textual-powershell/runbook-control-job-tile.png)

11. Щелкните имя задания, чтобы открыть ту же область заданий, что и при запуске runbook. Используйте эту панель для просмотра сведений о любом задании, созданном для runbook.

## <a name="step-5---add-authentication-to-manage-azure-resources"></a>Шаг 5. Добавление проверки подлинности для управления ресурсами Azure

Вы протестировали и опубликовали свой модуль runbook, но пока он не выполняет никаких полезных действий. Нужно, чтобы он управлял ресурсами Azure. Для этого последовательность runbook должна поддерживать проверку подлинности с учетной записью запуска от имени, которая создается автоматически при создании учетной записи службы автоматизации.

Как показано в примере ниже, подключение запуска от имени устанавливается с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Если вы управляете ресурсами в нескольких подписках, вам нужно использовать параметр `AzContext` в команде [Get-AzContext](/powershell/module/Az.Accounts/Get-AzContext).

> [!NOTE]
> Для модулей runbook PowerShell `Add-AzAccount` и `Add-AzureRMAccount` являются псевдонимами для `Connect-AzAccount`. Вы можете использовать эти командлеты или [обновить модули](../automation-update-azure-modules.md) в учетной записи службы автоматизации до последних версий. Обновление модулей может потребоваться, даже если учетная запись службы автоматизации только что создана.

   ```powershell
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave -Scope Process

   $connection = Get-AutomationConnection -Name AzureRunAsConnection

   # Wrap authentication in retry logic for transient network failures
   $logonAttempt = 0
   while(!($connectionResult) -and ($logonAttempt -le 10))
   {
       $LogonAttempt++
       # Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                               -ServicePrincipal `
                               -Tenant $connection.TenantID `
                               -ApplicationId $connection.ApplicationID `
                               -CertificateThumbprint $connection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }

   $AzureContext = Get-AzSubscription -SubscriptionId $connection.SubscriptionID

   Get-AzVM -ResourceGroupName myResourceGroup -AzContext $AzureContext
   ```

1. Откройте текстовый редактор, щелкнув **Изменить** на странице MyFirstRunbook-PowerShell.

2. Теперь строка `Write-Output` уже не нужна. Просто удалите ее.

3. Введите или скопируйте и вставьте следующий код, который выполнит проверку подлинности с учетной записью запуска от имени в службе автоматизации.

   ```powershell
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave -Scope Process

   $connection = Get-AutomationConnection -Name AzureRunAsConnection

   while(!($connectionResult) -and ($logonAttempt -le 10))
   {
       $LogonAttempt++
       # Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                               -ServicePrincipal `
                               -Tenant $connection.TenantID `
                               -ApplicationId $connection.ApplicationID `
                               -CertificateThumbprint $connection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }
   ```

4. Щелкните **Тестовая область**, чтобы проверить модуль runbook.

5. Щелкните **Пуск** , чтобы начать тестирование. После его завершения вы увидите примерно такие основные сведения из вашей учетной записи. Этот вывод подтверждает допустимость учетной записи запуска от имени.

   ![Authenticate](../media/automation-tutorial-runbook-textual-powershell/runbook-auth-output.png)

## <a name="step-6---add-code-to-start-a-virtual-machine"></a>Шаг 6. Добавление кода запуска виртуальной машины

Теперь, когда модуль runbook прошел аутентификацию в подписке Azure, вы можете управлять ресурсами. Добавьте команду для запуска виртуальной машины. Вы можете выбрать любую виртуальную машину в своей подписке Azure. Пока просто пропишите имя этой виртуальной машины в последовательности runbook.

1. Чтобы запустить виртуальную машину, добавьте в скрипт runbook командлет [Start-AzVM](/powershell/module/Az.Compute/Start-AzVM). Как показано ниже, этот командлет запускает виртуальную машину с именем `VMName` и группой ресурсов `ResourceGroupName`.

   ```powershell
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave -Scope Process

   $connection = Get-AutomationConnection -Name AzureRunAsConnection
   while(!($connectionResult) -and ($logonAttempt -le 10))
   {
       $LogonAttempt++
       # Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                               -ServicePrincipal `
                               -Tenant $connection.TenantID `
                               -ApplicationId $connection.ApplicationID `
                               -CertificateThumbprint $connection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }

   Start-AzVM -Name 'VMName' -ResourceGroupName 'ResourceGroupName'
   ```

2. Сохраните модуль runbook, а затем щелкните **Тестовая область**, чтобы проверить модуль.

3. Щелкните **Пуск**, чтобы начать тестирование. После завершения теста убедитесь, что виртуальная машина запущена.

## <a name="step-7---add-an-input-parameter"></a>Шаг 7. Добавление входного параметра

Сейчас runbook запускает виртуальную машину, имя которой прописано в его коде. Последовательность runbook принесет больше пользы, если вы сможете указывать нужную виртуальную машину при ее запуске. Для поддержки такой возможности добавьте в runbook входные параметры.

1. В текстовом редакторе измените командлет `Start-AzVM`, добавив переменные для параметров `VMName` и `ResourceGroupName`. 

   ```powershell
   Param(
    [string]$VMName,
    [string]$ResourceGroupName
   )
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave -Scope Process

   $connection = Get-AutomationConnection -Name AzureRunAsConnection
   while(!($connectionResult) -and ($logonAttempt -le 10))
   {
       $LogonAttempt++
       # Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                               -ServicePrincipal `
                               -Tenant $connection.TenantID `
                               -ApplicationId $connection.ApplicationID `
                               -CertificateThumbprint $connection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }

   Start-AzVM -Name $VMName -ResourceGroupName $ResourceGroupName
   ```

2. Сохраните модуль Runbook и откройте область тестирования. Теперь вы можете указать значения двух входных переменных, которые будут использоваться в тесте.

3. Закройте область тестирования.

4. Щелкните **Опубликовать** , чтобы опубликовать новую версию модуля Runbook.

5. Остановите виртуальную машину, которую вы запустили ранее.

6. Щелкните **Пуск** , чтобы запустить модуль Runbook. 

7. Введите значения **VMNAME** и **RESOURCEGROUPNAME** для виртуальной машины, которую вы хотите запустить, и щелкните **OK**.

    ![Передача параметра](../media/automation-tutorial-runbook-textual-powershell/automation-pass-params.png)

8. Когда runbook завершит работу, убедитесь, что виртуальная машина запущена.

## <a name="next-steps"></a>Дальнейшие действия

* См. сведения о [PowerShell, включая справочник по языку и обучающие модули](/powershell/scripting/overview).
* Справочник по командлетам PowerShell см. в документации по [Az.Automation](/powershell/module/az.automation).
* Чтобы начать работу с графическими модулями runbook, см. инструкции в [этой статье](automation-tutorial-runbook-graphical.md).
* Чтобы приступить к работе с runbook рабочего процесса PowerShell, обратитесь к [этом руководству](automation-tutorial-runbook-textual.md).
* См. сведения о типах runbook, их преимуществах и ограничениях в описании [типов последовательностей runbook в службе автоматизации Azure](../automation-runbook-types.md)
* См. сведения о [поддержке скриптов PowerShell в службе автоматизации Azure](https://azure.microsoft.com/blog/announcing-powershell-script-support-azure-automation-2/).
