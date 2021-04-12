---
title: Создание runbook рабочего процесса PowerShell в службе автоматизации Azure
description: Эта статья содержит инструкции по созданию, тестированию и публикации простого runbook рабочего процесса PowerShell.
services: automation
ms.subservice: process-automation
ms.date: 04/19/2020
ms.topic: tutorial
ms.openlocfilehash: e0247d1d113f7fb0bf23116b3473f73b78a58c80
ms.sourcegitcommit: d23602c57d797fb89a470288fcf94c63546b1314
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106167606"
---
# <a name="tutorial-create-a-powershell-workflow-runbook"></a>Руководство по Создание runbook рабочего процесса PowerShell

В данном учебнике описана процедура создания [Runbook рабочего процесса PowerShell](../automation-runbook-types.md#powershell-workflow-runbooks) в службе автоматизации Azure. Runbook рабочего процесса PowerShell представляют собой текстовые модули unbook, основанные на рабочем процессе Windows PowerShell. Код для таких модулей runbook можно создавать и редактировать в текстовом редакторе прямо на портале Azure. 

> [!div class="checklist"]
> * Создание простого модуля runbook рабочего процесса PowerShell
> * Тестирование и публикация runbook
> * Выполнение задания модуля runbook и отслеживание его состояния
> * Обновление runbook для запуска виртуальной машины Azure с указанными в runbook параметрами

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником необходимы указанные ниже компоненты.

* Подписка Azure. Если у вас ее нет, [активируйте преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) или [зарегистрируйте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Учетная запись службы автоматизации](../index.yml) , чтобы хранить модуль Runbook и выполнять проверку подлинности ресурсов Azure. Эта учетная запись должна иметь разрешение на запуск и остановку виртуальной машины.
* Виртуальная машина Azure. Это не должна быть рабочая виртуальная машина, так как для работы с этим руководством ее нужно остановить и запустить заново.

## <a name="step-1---create-new-runbook"></a>Шаг 1. Создание нового модуля Runbook

Для начала вы создадите простой модуль runbook, который выводит текст `Hello World`.

1. На портале Azure выберите свою учетную запись службы автоматизации.

   Страница учетной записи в службе автоматизации позволяет быстро получить представление о ресурсах, доступных в этой учетной записи. Некоторые ресурсы уже должны быть доступны. Большинство из них — это модули, автоматически добавленные в новую учетную запись службы автоматизации. Кроме того, вам потребуется ресурс учетных данных, связанный с используемой подпиской.
 
2. Щелкните **Модули Runbook** в разделе **Автоматизация процессов**, чтобы открыть список модулей runbook.

3. Создайте новый модуль runbook с помощью команды **Создать runbook**.

4. Присвойте Runbook имя **MyFirstRunbook-Workflow**.

5. В этом примере вы создадите [runbook рабочего процесса PowerShell](../automation-runbook-types.md#powershell-workflow-runbooks). Для параметра **Тип Runbook** выберите значение **Рабочий процесс PowerShell**.

6. Щелкните **Создать** , чтобы создать модуль Runbook и открыть текстовый редактор.

## <a name="step-2---add-code-to-the-runbook"></a>Шаг 2. Добавление кода в Runbook

Можно либо напрямую ввести код в runbook, либо выбрать командлеты, runbook и ресурсы из элемента управления "Библиотека" и добавить их в runbook с любыми нужными параметрами. Для этого руководства код будет вводиться прямо в runbook.

1. Созданный модуль runbook пока пустой и содержит лишь обязательное ключевое слово `Workflow`, имя и фигурные скобки, в которые будет добавлен весь рабочий процесс.

   ```powershell-interactive
   Workflow MyFirstRunbook-Workflow
   {
   }
   ```

2. Введите `Write-Output "Hello World"` между фигурными скобками.

   ```powershell-interactive
   Workflow MyFirstRunbook-Workflow
   {
   Write-Output "Hello World"
   }
   ```

3. Сохраните модуль Runbook, щелкнув **Сохранить**.

## <a name="step-3---test-the-runbook"></a>Шаг 3. Тестирование модуля Runbook

Прежде чем опубликовать модуль runbook и сделать его доступным в рабочей среде, необходимо проверить, правильно ли он работает. Тестирование runbook выполняется по черновой версии и позволяет проверить выходные данные в интерактивном режиме.

1. Выберите элемент **Тестовая область**, чтобы открыть панель тестирования.

2. Щелкните **Пуск**, чтобы начать тест для проверки выбранных вариантов.

3. Вы увидите, что будет создан объект [задания runbook](../automation-runbook-execution.md) и на панели отобразится его состояние.

   Сначала задание получает состояние "В очереди". Это означает, что задание ожидает доступности рабочей роли runbook в облаке. Как только рабочая роль затребует задание, оно получит состояние "Запуск". Когда модуль runbook начнет выполнение, состояние изменится на "Выполняется".

4. Когда задание runbook будет выполнено, на панели тестирования отобразится результат выполнения. В нашем примере отображается результат `Hello World`.

   ![Hello World](../media/automation-tutorial-runbook-textual/test-output-hello-world.png)

5. Закройте область тестирования, чтобы вернуться на холст.

## <a name="step-4---publish-and-start-the-runbook"></a>Шаг 4. Публикация и запуск модуля Runbook

Модуль runbook, который вы только что создали, все еще находится в режиме черновика. Прежде чем запустить модуль в рабочей среде, его нужно опубликовать. При публикации модуля Runbook существующая опубликованная версия перезаписывается черновой версией. В нашем случае опубликованной версии не существует, поскольку модуль runbook создан только что.

1. Щелкните **Опубликовать**, чтобы опубликовать модуль Runbook, а затем нажмите кнопку **Да** в появившемся запросе.

2. Прокрутите содержимое влево, чтобы просмотреть runbook на странице **Модули Runbook**, и убедитесь, что параметр **Состояние создания** имеет значение **Опубликовано**.

3. Прокрутите экран вправо до страницы **MyFirstRunbook-Workflow**.

   Параметры в верхней части экрана позволяют немедленно запустить модуль runbook, запланировать его запуск на определенное время в будущем или создать [веб-перехватчик](../automation-webhooks.md) для запуска модуля runbook с помощью HTTP-вызова.

4. Чтобы запустить runbook, выберите **Запуск** и нажмите кнопку **Да** в появившемся запросе.

   ![Запуск модуля Runbook](../media/automation-tutorial-runbook-textual/automation-runbook-controls-start.png)

5. Откроется область заданий с заданием runbook, которое вы только что создали. В нашем примере пока оставьте панель открытой, чтобы следить за ходом выполнения задания.

6. Вы увидите, что состояние задания отображается в разделе **Сводка по заданию**. Здесь поддерживаются те же состояния, что и ранее при тестировании runbook.

   :::image type="content" source="../media/automation-tutorial-runbook-textual/job-pane-status-blade-jobsummary.png" alt-text="Снимок экрана: область задания runbook, отображающая выделенный раздел сводки по заданию.":::

7. Как только состояние runbook изменится на "Выполнено", щелкните **Выходные данные**. Откроется область "Выходные данные", где отображается сообщение `Hello World`.

   :::image type="content" source="../media/automation-tutorial-runbook-textual/job-pane-status-blade-outputtile.png" alt-text="Снимок экрана: область задания runbook, отображающая выделенную кнопку выходных данных.":::

8. Закройте страницу выходных данных.

9. Щелкните **Все журналы** , чтобы открыть область "Потоки" для задания Runbook. В потоке выходных данных должно отобразиться только `Hello World`. Но не забывайте, что на панели "Потоки" могут присутствовать и другие потоки задания runbook, например потоки "Подробные сведения" и "Ошибка", если этот модуль runbook записывает в них какие-то данные.

   :::image type="content" source="../media/automation-tutorial-runbook-textual/job-pane-status-blade-alllogstile.png" alt-text="Снимок экрана: область задания runbook, отображающая выделенную кнопку всех журналов.":::

10. Закройте панель потоков и область заданий, чтобы вернуться на страницу MyFirstRunbook.

11. Щелкните **Задания** в области **Ресурсы**, чтобы открыть страницу "Задания" для этого модуля runbook. Откроется страница со списком всех заданий, созданных этим модулем runbook. Список должен содержать одно задание, так как вы запустили задание только один раз.

   ![Задания](../media/automation-tutorial-runbook-textual/runbook-control-job-tile.png)

12. Щелкните имя задания, чтобы открыть ту же область заданий, что и при запуске runbook. Используйте эту панель для просмотра сведений о любом задании, созданном для runbook.

## <a name="step-5---add-authentication-to-manage-azure-resources"></a>Шаг 5. Добавление проверки подлинности для управления ресурсами Azure

Вы протестировали и опубликовали свой модуль runbook, но пока он не выполняет никаких полезных действий. Нужно, чтобы он управлял ресурсами Azure. Модуль не сможет выполнять эту задачу, пока не проведена проверка подлинности с использованием учетных данных для подписки. Для проверки подлинности используется командлет [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

>[!NOTE]
>Для модулей runbook PowerShell `Add-AzAccount` и `Add-AzureRMAccount` являются псевдонимами для `Connect-AzAccount`. Вы можете использовать эти командлеты или [обновить модули](../automation-update-azure-modules.md) в учетной записи службы автоматизации до последних версий. Обновление модулей может потребоваться, даже если учетная запись службы автоматизации только что создана.

1. Перейдите на страницу области MyFirstRunbook-Workflow и откройте текстовый редактор, щелкнув **Изменить**.

2. Удалите строку `Write-Output`.

3. Поместите курсор в пустую строку между фигурными скобками.

4. Введите или скопируйте и вставьте следующий код, который выполнит проверку подлинности с учетной записью запуска от имени в службе автоматизации.

   ```powershell-interactive
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave -Scope Process

   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID `
   -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

   $AzureContext = Select-AzSubscription -SubscriptionId $Conn.SubscriptionID
   ```

5. Щелкните **Тестовая область**, чтобы проверить модуль runbook.

6. Щелкните **Пуск** , чтобы начать тестирование. После его завершения вы увидите примерно такие основные сведения из вашей учетной записи. Это действие подтверждает допустимость учетных данных.

   ![Authenticate](../media/automation-tutorial-runbook-textual/runbook-auth-output.png)

## <a name="step-6---add-code-to-start-a-virtual-machine"></a>Шаг 6. Добавление кода запуска виртуальной машины

Теперь, когда модуль runbook прошел аутентификацию в подписке Azure, вы можете управлять ресурсами. Добавьте команду для запуска виртуальной машины. Вы можете выбрать любую виртуальную машину в своей подписке Azure. Имя этой машины вы жестко пропишете в runbook. Если вы управляете ресурсами в нескольких подписках, вам нужно использовать параметр `AzContext` в командлете [Get-AzContext](/powershell/module/az.accounts/get-azcontext).

1. Укажите имя запускаемой виртуальной машины и имя группы ресурсов для нее, вызвав командлет [Start-AzVM](/powershell/module/Az.Compute/Start-AzVM) такой строкой, как показано ниже. 

   ```powershell-interactive
   workflow MyFirstRunbook-Workflow
   {
   # Ensures that you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave -Scope Process

   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

   $AzureContext = Get-AzSubscription -SubscriptionId $Conn.SubscriptionID

   Start-AzVM -Name 'VMName' -ResourceGroupName 'ResourceGroupName' -AzContext $AzureContext
   }
   ```

2. Сохраните модуль runbook, а затем щелкните **Тестовая область**, чтобы проверить модуль.

3. Щелкните **Пуск** , чтобы начать тестирование. После завершения теста убедитесь, что виртуальная машина запущена.

## <a name="step-7---add-an-input-parameter-to-the-runbook"></a>Шаг 7. Добавление входного параметра в модуль Runbook

Сейчас runbook запускает виртуальную машину, которая жестко прописана в его коде. Он принесет больше пользы, если вы сможете указывать виртуальную машину при запуске runbook. Для поддержки такой возможности добавьте в runbook входные параметры.

1. Добавьте в runbook переменные для параметров `VMName` и `ResourceGroupName`, а затем примените эти переменные для командлета `Start-AzVM`, как показано ниже.

   ```powershell-interactive
   workflow MyFirstRunbook-Workflow
   {
    Param(
     [string]$VMName,
     [string]$ResourceGroupName
    )
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave -Scope Process

   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
   Start-AzVM -Name $VMName -ResourceGroupName $ResourceGroupName
   }
   ```

2. Сохраните модуль Runbook и откройте область тестирования. Теперь можно указать значения двух входных переменных, которые будут использоваться при тестировании.

3. Закройте область тестирования.

4. Щелкните **Опубликовать** , чтобы опубликовать новую версию модуля Runbook.

5. Остановите виртуальную машину, которую запустили.

6. Щелкните **Пуск** , чтобы запустить модуль Runbook. 

7. Введите значения **VMNAME** и **RESOURCEGROUPNAME** для виртуальной машины, которую решили запустить.

   ![Запуск модуля Runbook](../media/automation-tutorial-runbook-textual/automation-pass-params.png)

8. Когда runbook завершит работу, убедитесь, что виртуальная машина запущена.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о PowerShell, включая справочник по языку и обучающие модули, вы найдете в [этой документации](/powershell/scripting/overview).
* Справочник по командлетам PowerShell см. в документации по [Az.Automation](/powershell/module/az.automation).
* Чтобы начать работу с графическими модулями runbook, см. инструкции в [этой статье](automation-tutorial-runbook-graphical.md).
* Сведения о том, как начать работу с runbook, см. в статье о [создании runbook PowerShell](automation-tutorial-runbook-textual-powershell.md).
* Дополнительные сведения о типах модулей runbook в службе автоматизации Azure, а также об их преимуществах и ограничениях см. в [этой статье](../automation-runbook-types.md).
* Дополнительные сведения о встроенной функции поддержки скриптов PowerShell в службе автоматизации Azure см. в [этой статье](https://azure.microsoft.com/blog/announcing-powershell-script-support-azure-automation-2/).
