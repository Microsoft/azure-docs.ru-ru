---
title: Устранение неполадок с runbook службы автоматизации Azure
description: В этой статье рассказывается, как устранять проблемы с последовательностями runbook службы автоматизации Azure.
services: automation
ms.date: 02/11/2021
ms.topic: troubleshooting
ms.custom: has-adal-ref
ms.openlocfilehash: 1ff5adf3ec974cc922d73cf5993a78722ca1b591
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101723815"
---
# <a name="troubleshoot-runbook-issues"></a>Устранение неполадок с последовательностями runbook

 В этой статье описываются возможные проблемы с последовательностями runbook и способы их устранения. Общие сведения см. в статье [Выполнение последовательности runbook в службе автоматизации Azure](../automation-runbook-execution.md).

## <a name="diagnose-runbook-issues"></a>Диагностика проблем с runbook

Для диагностирования проблем в случае ошибок во время выполнения runbook в службе автоматизации Azure можно выполнить следующие действия.

1. Убедитесь, что скрипт runbook успешно выполнен на локальном компьютере.

    Справочные сведения по языку и обучающие модули см. в документации по [PowerShell](/powershell/scripting/overview) или [Python](https://docs.python.org/3/). Локальное выполнение скрипта позволяет обнаружить и устранить распространенные ошибки, например:

      * отсутствующие модули;
      * синтаксические ошибки;
      * логические ошибки.

1. Изучите [потоки ошибок](../automation-runbook-output-and-messages.md#runbook-output) в runbook.

    Просмотрите конкретные сообщения об ошибках в этих потоках и сопоставьте их с ошибками, описанными в этой статье.

1. Убедитесь, что узлы и рабочая область службы автоматизации содержат необходимые модули.

    Если runbook импортирует все модули, убедитесь, что они доступны для учетной записи службы автоматизации, выполнив действия, описанные в разделе [Импорт модулей](../shared-resources/modules.md#import-modules). Обновите модули PowerShell до последней версии, выполнив инструкции в статье [Обновление модулей Azure PowerShell в службе автоматизации Azure](../automation-update-azure-modules.md). Дополнительные сведения об устранении неполадок см. в разделе об [устранении неполадок в модулях](shared-resources.md#modules).

1. Если выполнение runbook приостановлено или завершилось неожиданным сбоем, выполните следующие действия.

    * [Продлите сертификат](../manage-runas-account.md#cert-renewal), если истек срок действия учетной записи запуска от имени.
    * [Продлите веб-перехватчик](../automation-webhooks.md#renew-a-webhook), если вы пытаетесь использовать для запуска runbook веб-перехватчик с истекшим сроком действия.
    * [Проверьте состояния заданий](../automation-runbook-execution.md#job-statuses), чтобы определить текущие состояния runbook и возможные причины проблемы.
    * [Добавьте в последовательность runbook дополнительные выходные данные](../automation-runbook-output-and-messages.md#working-with-message-streams), чтобы определить, что происходит перед приостановкой ее выполнения.
    * [Обработайте исключения](../automation-runbook-execution.md#exceptions), вызываемые заданием.

1. Выполните этот шаг, если задание runbook или среда с гибридной рабочей ролью Runbook не отвечает.

    Если runbook выполняется с помощью гибридной рабочей роли, а не в службе автоматизации Azure, возможно, вам потребуется [устранить неполадки самой гибридной рабочей роли](hybrid-runbook-worker.md).

## <a name="scenario-runbook-fails-with-a-no-permission-or-forbidden-403-error"></a><a name="runbook-fails-no-permission"></a>Сценарий. Выполнение последовательности runbook прекратилось с ошибкой "Нет разрешения" или "Запрещено 403"

### <a name="issue"></a>Проблема

Выполнение последовательности runbook прекратилось с ошибкой "Нет разрешения", "Запрещено 403" или эквивалентной.

### <a name="cause"></a>Причина

Учетные записи запуска от имени могут не иметь таких же разрешений для ресурсов Azure, как ваша текущая учетная запись службы автоматизации. 

### <a name="resolution"></a>Решение

Убедитесь, что у вашей учетной записи запуска от имени [есть разрешения на доступ к любым ресурсам](../../role-based-access-control/role-assignments-portal.md), используемым в скрипте.

## <a name="scenario-sign-in-to-azure-account-failed"></a><a name="sign-in-failed"></a>Сценарий. Не удается войти в учетную запись Azure

### <a name="issue"></a>Проблема

При работе с командлетом `Connect-AzAccount` происходит одна из следующих ошибок.

```error
Unknown_user_type: Unknown User Type
```

```error
No certificate was found in the certificate store with thumbprint
```

### <a name="cause"></a>Причина

Эта ошибка происходит, если указано недопустимое имя ресурса учетных данных. Ошибка также может произойти, если имя пользователя и пароль, которые вы ввели для настройки ресурса учетных данных службы автоматизации, указаны неправильно.

### <a name="resolution"></a>Решение

Чтобы определить причину ошибки, выполните следующие действия:

1. Убедитесь, что вы не использовали специальные символы. В имени ресурса учетных данных службы автоматизации Azure, используемом для подключения к Azure, запрещается вводить такие специальные символы, как `\@`.
1. Убедитесь, что вы можете использовать имя пользователя и пароль, которые хранятся в учетных данных службы автоматизации Azure в редакторе локальной интегрированной среды сценариев PowerShell. В интегрированной среде сценариев PowerShell запустите следующий командлет.

   ```powershell
   $Cred = Get-Credential
   #Using Azure Service Management
   Add-AzureAccount –Credential $Cred
   #Using Azure Resource Manager
   Connect-AzAccount –Credential $Cred
   ```

1. Если произошел локальный сбой проверки подлинности, значит учетные данные Azure Active Directory (Azure AD) указаны неправильно. Чтобы правильно настроить учетную запись Azure AD, см. статью [Проверка подлинности в Azure с помощью Azure Active Directory](../automation-use-azure-ad.md).

1. Если ошибка, скорее всего, временная, попробуйте добавить логику повторных попыток в процедуру проверки подлинности, чтобы сделать ее более надежной.

   ```powershell
   # Get the connection "AzureRunAsConnection"
   $connectionName = "AzureRunAsConnection"
   $servicePrincipalConnection = Get-AutomationConnection -Name $connectionName

   $logonAttempt = 0
   $logonResult = $False

   while(!($connectionResult) -And ($logonAttempt -le 10))
   {
       $LogonAttempt++
       #Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                              -ServicePrincipal `
                              -Tenant $servicePrincipalConnection.TenantId `
                              -ApplicationId $servicePrincipalConnection.ApplicationId `
                              -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }
   ```

## <a name="scenario-run-login-azurermaccount-to-log-in"></a><a name="login-azurerm"></a>Сценарий. Выполнение входа с помощью Login-AzureRMAccount

### <a name="issue"></a>Проблема

При запуске модуля runbook отображается следующее сообщение об ошибке:

```error
Run Login-AzureRMAccount to login.
```

### <a name="cause"></a>Причина

Эта ошибка может произойти, если вы не используете учетную запись запуска от имени или срок ее действия истек. Дополнительные сведения см. в статье [Общие сведения об учетных записях запуска от имени службы автоматизации Azure](../automation-security-overview.md#run-as-accounts).

Эта ошибка происходит по двум основным причинам.

* Используются разные версии модуля AzureRM или Az.
* Вы пытаетесь получить доступ к ресурсам в отдельной подписке.

### <a name="resolution"></a>Решение

Если эта ошибка происходит после обновления одного модуля AzureRM или Az, обновите все модули до одной версии.

Если вы пытаетесь получить доступ к ресурсам в другой подписке, выполните следующие действия, чтобы настроить разрешения.

1. Перейдите к учетной записи запуска от имени службы автоматизации и скопируйте **идентификатор приложения** и **отпечаток**.

    ![Копирование идентификатора приложения и отпечатка](../media/troubleshoot-runbooks/collect-app-id.png)

1. Перейдите в средство **управления доступом** подписки, *отличной* от той, где размещена учетная запись службы автоматизации, и добавьте новое назначение ролей.

    ![Управление доступом](../media/troubleshoot-runbooks/access-control.png)

1. Добавьте **идентификатор приложения**, полученный ранее. Выберите разрешения на уровне **Участник**.

    ![Добавление назначения роли](../media/troubleshoot-runbooks/add-role-assignment.png)

1. Скопируйте имя подписки.

1. Теперь можно использовать следующий код runbook для проверки разрешений учетной записи службы автоматизации в другой подписке. Замените `<CertificateThumbprint>` значением, скопированным на шаге 1. Замените `"<SubscriptionName>"` значением, скопированным на шаге 4.

    ```powershell
    $Conn = Get-AutomationConnection -Name AzureRunAsConnection
    Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint "<CertificateThumbprint>"
    #Select the subscription you want to work with
    Select-AzSubscription -SubscriptionName '<YourSubscriptionNameGoesHere>'

    #Test and get outputs of the subscriptions you granted access.
    $subscriptions = Get-AzSubscription
    foreach($subscription in $subscriptions)
    {
        Set-AzContext $subscription
        Write-Output $subscription.Name
    }
    ```

## <a name="scenario-unable-to-find-the-azure-subscription"></a><a name="unable-to-find-subscription"></a>Сценарий. не удается найти подписку Azure

### <a name="issue"></a>Проблема

При работе с командлетом `Select-AzureSubscription`, `Select-AzureRMSubscription` или `Select-AzSubscription` произошла следующая ошибка:

```error
The subscription named <subscription name> cannot be found.
```

### <a name="error"></a>Error

Эта ошибка может произойти, если:

* имя подписки указано неправильно;
* пользователь Azure AD, который пытается получить информацию о подписке, не имеет прав администратора для этой подписки.
* Командлет недоступен.

### <a name="resolution"></a>Решение

Выполните следующие действия, чтобы определить, пройдена ли проверка подлинности в Azure и есть ли доступ к подписке, которую вы пытаетесь выбрать.

1. Проверьте скрипт вне службы автоматизации Azure и убедитесь, что он может работать автономно.
1. Убедитесь, что скрипт запускает командлет [Connect-AzAccount](/powershell/module/Az.Accounts/Connect-AzAccount) перед запуском командлета `Select-*`.
1. К началу вашего модуля runbook добавьте `Disable-AzContextAutosave –Scope Process`. Этот командлет обеспечит применение любых учетных данных только к выполнению текущего модуля runbook.
1. Если это сообщение об ошибке не исчезло, измените код, добавив параметр `AzContext` для `Connect-AzAccount`, а затем выполните код.

   ```powershell
   Disable-AzContextAutosave –Scope Process

   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

   $context = Get-AzContext

   Get-AzVM -ResourceGroupName myResourceGroup -AzContext $context
    ```

## <a name="scenario-runbooks-fail-when-dealing-with-multiple-subscriptions"></a><a name="runbook-auth-failure"></a>Сценарий. Сбой модуля runbook при работе с несколькими подписками

### <a name="issue"></a>Проблема

При выполнении последовательностей runbook им не удается управлять ресурсами Azure.

### <a name="cause"></a>Причина

При запуске модуля runbook используется неправильный контекст. Это может быть вызвано тем, что модуль Runbook случайно пытается получить доступ к неверной подписке.

Вы можете столкнуться с такими ошибками:

```error
Get-AzVM : The client '<automation-runas-account-guid>' with object id '<automation-runas-account-guid>' does not have authorization to perform action 'Microsoft.Compute/virtualMachines/read' over scope '/subscriptions/<subcriptionIdOfSubscriptionWichDoesntContainTheVM>/resourceGroups/REsourceGroupName/providers/Microsoft.Compute/virtualMachines/VMName '.
   ErrorCode: AuthorizationFailed
   StatusCode: 403
   ReasonPhrase: Forbidden Operation
   ID : <AGuidRepresentingTheOperation> At line:51 char:7 + $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $UNBV... +
```

### <a name="resolution"></a>Решение

Контекст подписки может быть утрачен, если runbook вызывает несколько последовательностей runbook. Чтобы избежать случайной попытки доступа к неверной подписке, следуйте указаниям ниже.

* Чтобы избежать создания ссылок на неправильную подписку, отключите сохранение контекста в модулях Runbook службы автоматизации, используя следующий код в начале каждого модуля Runbook.

   ```azurepowershell-interactive
   Disable-AzContextAutosave –Scope Process
   ```

* Командлеты Azure PowerShell поддерживают `-DefaultProfile` параметр. Он был добавлен ко всем командлетам AZ и AzureRm для поддержки выполнения нескольких сценариев PowerShell в одном процессе, что позволяет указать контекст и подписку, которую следует использовать для каждого командлета. При работе с модулями Runbook объект контекста следует сохранять в модуле Runbook при создании модуля Runbook (то есть при входе в учетную запись) и при каждом изменении, а также ссылаться на контекст при указании командлета AZ.

   > [!NOTE]
   > Необходимо передать объект контекста, даже если манипулировать контекстом напрямую с помощью командлетов, таких как [Set-азконтекст](/powershell/module/az.accounts/Set-AzContext) или [SELECT-азсубскриптион](/powershell/module/servicemanagement/azure.service/set-azuresubscription).

   ```azurepowershell-interactive
   $servicePrincipalConnection=Get-AutomationConnection -Name $connectionName 
   $context = Add-AzAccount `
             -ServicePrincipal `
             -TenantId $servicePrincipalConnection.TenantId `
             -ApplicationId $servicePrincipalConnection.ApplicationId `
             -Subscription 'cd4dxxxx-xxxx-xxxx-xxxx-xxxxxxxx9749' `
             -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint 
   $context = Set-AzContext -SubscriptionName $subscription `
       -DefaultProfile $context
   Get-AzVm -DefaultProfile $context
   ```
  
## <a name="scenario-authentication-to-azure-fails-because-multifactor-authentication-is-enabled"></a><a name="auth-failed-mfa"></a>Сценарий. Сбой проверки подлинности в Azure из-за включенной многофакторной проверки подлинности

### <a name="issue"></a>Проблема

При проверке подлинности в Azure с помощью имени и пароля происходит следующая ошибка:

```error
Add-AzureAccount: AADSTS50079: Strong authentication enrollment (proof-up) is required
```

### <a name="cause"></a>Причина

Если в учетной записи Azure активирована многофакторная проверка подлинности, применять учетную запись пользователя Azure Active Directory для аутентификации в Azure нельзя. Вместо этого используйте для проверки подлинности сертификат или субъект-службу.

### <a name="resolution"></a>Решение

Сведения об использовании классической учетной записи запуска от имени с командлетами классической модели развертывания Azure см. [в статье Создание классической учетной записи запуска от имени для управления службами Azure](../automation-create-standalone-account.md#create-a-classic-run-as-account). Сведения об использовании субъекта-службы с командлетами диспетчера ресурсов Azure см. в статьях о [создании субъекта-службы с помощью портала Azure](../../active-directory/develop/howto-create-service-principal-portal.md) и о [проверке подлинности субъекта-службы в Azure Resource Manager](../../active-directory/develop/howto-authenticate-service-principal-powershell.md).

## <a name="scenario-runbook-fails-with-a-task-was-canceled-error-message"></a><a name="task-was-cancelled"></a>Сценарий. Сбой runbook с сообщением об ошибке "Задача была отменена"

### <a name="issue"></a>Проблема

В модуле Runbook возникает ошибка, аналогичная приведенной ниже:

```error
Exception: A task was canceled.
```

### <a name="cause"></a>Причина

Это ошибка может быть вызвана использованием устаревших модулей Azure.

### <a name="resolution"></a>Решение

Эту ошибку можно устранить, обновив модули Azure до последней версии.

1. В учетной записи службы автоматизации последовательно выберите **Модули** и **Обновить модули Azure**.
1. Обновление занимает примерно 15 минут. После его завершения повторно запустите последовательность runbook, которая завершилась сбоем.

Дополнительные сведения об обновлении модулей см. в разделе [Обновление модулей Azure в службе автоматизации Azure](../automation-update-azure-modules.md).

## <a name="scenario-term-not-recognized-as-the-name-of-a-cmdlet-function-or-script"></a><a name="not-recognized-as-cmdlet"></a>Сценарий. Термин не распознан как имя командлета, функции или скрипта

### <a name="issue"></a>Проблема

В модуле Runbook возникает ошибка, аналогичная приведенной ниже:

```error
The term 'Connect-AzAccount' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if the path was included verify that the path is correct and try again.
```

### <a name="cause"></a>Причина

Эта ошибка может происходить по следующим причинам:

* Модуль, содержащий командлет, не импортирован в учетную запись службы автоматизации.
* Модуль, содержащий командлет, импортирован, но устарел.

### <a name="resolution"></a>Решение

Чтобы устранить эту ошибку, выполните одно из следующих действий.

* Если это модуль Azure, см. статью об [обновлении модулей Azure PowerShell в учетной записи службы автоматизации Azure](../automation-update-azure-modules.md).
* Если это другой модуль, убедитесь, что он импортирован в учетную запись службы автоматизации.

## <a name="scenario-cmdlet-fails-in-pnp-powershell-runbook-on-azure-automation"></a>Сценарий: Сбой командлета в runbook PowerShell PnP в службе автоматизации Azure

### <a name="issue"></a>Проблема

Когда runbook записывает автоматически созданный PowerShell объект PnP в выходные данные службы автоматизации Azure напрямую, выходные данные командлета не могут передаваться потоком обратно в службу автоматизации.

### <a name="cause"></a>Причина

Чаще всего эта проблема возникает, когда служба автоматизации Azure обрабатывает последовательности runbook, которые вызывают командлеты PowerShell PnP, например `add-pnplistitem`, без перехвата возвращаемых объектов.

### <a name="resolution"></a>Решение

Измените скрипты, чтобы все возвращаемые значения присваивались переменным, а командлеты не предпринимали попытки записи целых объектов в стандартный поток вывода. Скрипт может перенаправить поток вывода в командлет, как показано ниже.

```azurecli
  $null = add-pnplistitem
```

Если скрипт анализирует выходные данные командлета, он должен сохранять их в переменной и оперировать ею, а не просто передавать эти выходные данные потоком.

```azurecli
$SomeVariable = add-pnplistitem ....
if ($SomeVariable.someproperty -eq ....
```

## <a name="scenario-cmdlet-not-recognized-when-executing-a-runbook"></a><a name="cmdlet-not-recognized"></a>Сценарий. не удается распознать командлет при выполнении модуля Runbook

### <a name="issue"></a>Проблема

Выполнение задания Runbook завершилось ошибкой.

```error
<cmdlet name>: The term <cmdlet name> is not recognized as the name of a cmdlet, function, script file, or operable program.
```

### <a name="cause"></a>Причина

Эта ошибка возникает, когда модулю PowerShell не удается найти командлет, который используется в runbook. Такое возможно, когда в учетной записи отсутствует модуль, содержащий командлет, когда возникает конфликт имен с именем runbook, а также когда командлет существует и в другом модуле, а службе автоматизации не удается разрешить имя.

### <a name="resolution"></a>Решение

Эту проблему можно устранить любым из следующих способов.

* Проверьте правильность ввода имени командлета.
* Убедитесь, что командлет имеется в учетной записи службы автоматизации, а конфликты отсутствуют. Чтобы проверить наличие командлета, откройте runbook в режиме редактирования и найдите требуемый командлет в библиотеке или выполните команду `Get-Command <CommandName>`. Убедившись в наличии командлета в учетной записи и отсутствии конфликта имен с другими командлетами или последовательностями runbook, добавьте его на холст. Убедитесь, что в runbook используется допустимый набор параметров.
* Если обнаружен конфликт имен, а сам командлет доступен в двух разных модулях, эту проблему можно решить использованием полного имени командлета. Например, вы можете использовать `ModuleName\CmdletName`.
* Если runbook выполняется локально в группе гибридных рабочих ролей, убедитесь, что на компьютере, где размещена гибридная рабочая роль, установлен соответствующий модуль или командлет.

## <a name="scenario-incorrect-object-reference-on-call-to-add-azaccount"></a><a name="object-reference-not-set"></a>Сценарий. Неверная ссылка на объект при вызове Add-AzAccount

### <a name="issue"></a>Проблема

Эта ошибка происходит при работе с псевдонимом `Add-AzAccount` для командлета `Connect-AzAccount`.

```error
Add-AzAccount : Object reference not set to an instance of an object
```

### <a name="cause"></a>Причина

Эта ошибка может происходить, если runbook не выполняет нужных действий перед вызовом `Add-AzAccount` для добавления учетной записи службы автоматизации. Примером одного из необходимых действий может служить вход с использованием учетной записи запуска от имени. Сведения о том, какие операции следует использовать в runbook, см. в статье [Выполнение runbook в службе автоматизации Azure](../automation-runbook-execution.md).

## <a name="scenario-object-reference-not-set-to-an-instance-of-an-object"></a><a name="child-runbook-object"></a>Сценарий. Ссылка на объект не указывает на экземпляр объекта

### <a name="issue"></a>Проблема

Если поток вывода содержит объект, при вызове дочерней последовательности runbook с параметром `Wait` происходит следующая ошибка:

```error
Object reference not set to an instance of an object
```

### <a name="cause"></a>Причина

Если поток содержит объекты, `Start-AzAutomationRunbook` неправильно обрабатывает поток вывода.

### <a name="resolution"></a>Решение

Реализуйте логику опроса и используйте для получения выходных данных командлет [Get-AzAutomationJobOutput](/powershell/module/Az.Automation/Get-AzAutomationJobOutput). Пример этой логики определяется здесь:

```powershell
$automationAccountName = "ContosoAutomationAccount"
$runbookName = "ChildRunbookExample"
$resourceGroupName = "ContosoRG"

function IsJobTerminalState([string] $status) {
    return $status -eq "Completed" -or $status -eq "Failed" -or $status -eq "Stopped" -or $status -eq "Suspended"
}

$job = Start-AzAutomationRunbook -AutomationAccountName $automationAccountName -Name $runbookName -ResourceGroupName $resourceGroupName
$pollingSeconds = 5
$maxTimeout = 10800
$waitTime = 0
while($false -eq (IsJobTerminalState $job.Status) -and $waitTime -lt $maxTimeout) {
   Start-Sleep -Seconds $pollingSeconds
   $waitTime += $pollingSeconds
   $job = $job | Get-AzAutomationJob
}

$job | Get-AzAutomationJobOutput | Get-AzAutomationJobOutputRecord | Select-Object -ExpandProperty Value
```

## <a name="scenario-runbook-fails-because-of-deserialized-object"></a><a name="fails-deserialized-object"></a>Сценарий. сбой модуля Runbook из-за десериализованного объекта

### <a name="issue"></a>Проблема

Выполнение модуля Runbook завершилось ошибкой.

```error
Cannot bind parameter <ParameterName>.

Cannot convert the <ParameterType> value of type Deserialized <ParameterType> to type <ParameterType>.
```

### <a name="cause"></a>Причина

Когда runbook является рабочим процессом PowerShell, он хранит сложные объекты в десериализированном формате, чтобы сохранить состояние модуля при приостановке рабочего процесса.

### <a name="resolution"></a>Решение

Эту проблему можно устранить любым из следующих способов.

* Если сложные объекты передаются от одного командлета к другому через конвейер, используйте в качестве оболочки для этих командлетов действие `InlineScript`.
* Передайте только требуемое имя или значение сложного объекта, а не весь объект.
* Используйте модуль Runbook PowerShell, а не модуль Runbook рабочего процесса PowerShell.

## <a name="scenario-400-bad-request-status-when-calling-a-webhook"></a><a name="expired webhook"></a>Сценарий. Состояние "400 — недопустимый запрос" при вызове веб-перехватчика

### <a name="issue"></a>Проблема

При попытке вызова веб-перехватчика для runbook службы автоматизации Azure происходит следующая ошибка:

```error
400 Bad Request : This webhook has expired or is disabled
```

### <a name="cause"></a>Причина

Вызываемый веб-перехватчик отключен, или срок его действия истек. 

### <a name="resolution"></a>Решение

Если веб-перехватчик отключен, его можно повторно включить на портале Azure. Если срок действия веб-перехватчика истек, его необходимо удалить, а затем создать заново. Если срок действия еще не истек, можно только [обновить веб-перехватчик](../automation-webhooks.md#renew-a-webhook). 

## <a name="scenario-429-the-request-rate-is-currently-too-large"></a><a name="429"></a>Сценарий. 429: слишком большая текущая частота запросов

### <a name="issue"></a>Проблема

При выполнении командлета `Get-AzAutomationJobOutput` вы получаете следующее сообщение об ошибке:

```error
429: The request rate is currently too large. Please try again
```

### <a name="cause"></a>Причина

Эта ошибка может происходить при извлечении выходных данных задания из последовательности runbook с множеством [подробных потоков](../automation-runbook-output-and-messages.md#write-output-to-verbose-stream).

### <a name="resolution"></a>Решение

Чтобы устранить эту ошибку, выполните одно из следующих действий.

* изменить модуль Runbook и сократить число создаваемых потоков заданий;
* уменьшите количество потоков, полученных при выполнении командлета. Для этого можно задать значение параметра `Stream` для командлета [Get-AzAutomationJobOutput](/powershell/module/Az.Automation/Get-AzAutomationJobOutput), чтобы получить только потоки вывода. 

## <a name="scenario-runbook-job-fails-because-allocated-quota-was-exceeded"></a><a name="quota-exceeded"></a>Сценарий. Не удается выполнить задание runbook из-за превышения выделенной квоты

### <a name="issue"></a>Проблема

Выполнение задания Runbook завершилось ошибкой.

```error
The quota for the monthly total job run time has been reached for this subscription
```

### <a name="cause"></a>Причина

Эта ошибка возникает, когда время выполнения задания превышает квоту для учетной записи в 500 бесплатных минут. Эта квота применяется ко всем типам задач на выполнение заданий. К этим типам задач может относиться тестирование задания, запуск задания на портале Azure, выполнение задания с помощью веб-перехватчиков, а также планирование задания с помощью портала Azure или центра обработки данных. Дополнительные сведения см. на странице [цен на службу автоматизации](https://azure.microsoft.com/pricing/details/automation/).

### <a name="resolution"></a>Решение

Если на обработку вам требуется более 500 минут в месяц, выберите для подписки уровень "Базовый" вместо уровня "Бесплатный".

1. Войдите в свою подписку Azure.
1. Выберите учетную запись службы автоматизации, которую вы хотите обновить.
1. Выберите **Параметры**, а затем — **Цены**.
1. В нижней части страницы нажмите кнопку **Включить**, чтобы обновить свою учетную запись до уровня "Базовый".

## <a name="scenario-runbook-output-stream-greater-than-1-mb"></a><a name="output-stream-greater-1mb"></a>Сценарий: поток вывода Runbook больше 1 МБ

### <a name="issue"></a>Проблема

Модуль Runbook, работающий в песочнице Azure, завершается со следующей ошибкой:

```error
The runbook job failed due to a job stream being larger than 1MB, this is the limit supported by an Azure Automation sandbox.
```

### <a name="cause"></a>Причина

Эта ошибка возникает из-за того, что модуль Runbook попытался записать слишком много данных об исключении в поток вывода.

### <a name="resolution"></a>Решение

В потоке выходных данных задания превышено ограничение в 1 МБ. Убедитесь, что runbook включает вызовы в исполняемый файл или вложенный процесс с помощью блоков `try` и `catch`. Если операции приводят к исключению, код будет записывать сообщение из исключения в переменную службы автоматизации. Этот подход предотвращает запись сообщения в поток вывода задания. Для выполнения заданий гибридной рабочей роли Runbook поток вывода, усеченный до 1 МБ, отображается без сообщения об ошибке.

## <a name="scenario-runbook-job-start-attempted-three-times-but-fails-to-start-each-time"></a><a name="job-attempted-3-times"></a>Сценарий. Предприняты три неудачные попытки запуска задания runbook

### <a name="issue"></a>Проблема

Последовательность runbook завершается сбоем со следующей ошибкой:

```error
The job was tried three times but it failed
```

### <a name="cause"></a>Причина

Эта ошибка происходит из-за одной из следующих проблем.

* **Предельный объем памяти.** Задание может завершиться ошибкой, если оно использует более 400 МБ памяти. Ограничения на выделяемый песочнице объем памяти описаны в разделе [Ограничения службы автоматизации](../../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits). 

* **Сетевые сокеты.** В песочницах Azure одновременно может выполняться не более 1000 сетевых сокетов. Дополнительные сведения см. в статье [Ограничения службы автоматизации](../../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits).

* **Несовместим модуль.** Зависимости модуля могут быть неправильными. В этом случае runbook обычно возвращает сообщение `Command not found` или `Cannot bind parameter`.

* **Для песочницы отсутствует проверка подлинности в Active Directory.** Последовательность runbook попыталась вызвать исполняемый файл или подпроцесс, который запущен в песочнице Azure. Настройка runbook для проверки подлинности в Azure AD с помощью Библиотеки проверки подлинности Azure Active Directory (ADAL) не поддерживается.

### <a name="resolution"></a>Решение

* **Ограничение памяти, сетевые сокеты.** Чтобы избежать превышения предельного объема памяти, рекомендуется разделить рабочую нагрузку между несколькими последовательностями runbook, не обрабатывать слишком большой объем данных в памяти, не записывать ненужные выходные данные из последовательностей runbook и учитывать, сколько контрольных точек записывается в последовательности runbook рабочих процессов PowerShell. Чтобы очистить переменную и немедленно запустить сборку мусора, вы можете использовать методы `$myVar.clear` и `[GC]::Collect` соответственно. Эти действия уменьшают объем памяти, занимаемой модулем runbook во время выполнения.

* **Несовместим модуль.** Обновите модули Azure, выполнив инструкции, приведенные в статье об [обновлении модулей Azure PowerShell в учетной записи службы автоматизации Azure](../automation-update-azure-modules.md).

* **Для песочницы отсутствует проверка подлинности в Active Directory.** При проверке подлинности в Azure AD с помощью runbook убедитесь, что модуль Azure AD доступен в учетной записи службы автоматизации. Не забудьте предоставить учетной записи запуска от имени необходимые разрешения для выполнения задач, которые автоматизируются последовательностью runbook.

  Если runbook не может вызвать исполняемый файл или подпроцесс, выполняющийся в песочнице Azure, используйте runbook с [гибридной рабочей ролью Runbook](../automation-hrw-run-runbooks.md). В гибридных рабочих ролях нет таких ограничений памяти и сетевых ресурсов, как в песочницах Azure.

## <a name="scenario-powershell-job-fails-with-cannot-invoke-method-error-message"></a><a name="cannot-invoke-method"></a>Сценарий. Сбой задания PowerShell с сообщением об ошибке "Не удается вызвать метод"

### <a name="issue"></a>Проблема

При запуске задания PowerShell в последовательности runbook, запущенной в Azure, отображается следующее сообщение об ошибке:

```error
Exception was thrown - Cannot invoke method. Method invocation is supported only on core types in this language mode.
```

### <a name="cause"></a>Причина

Эта ошибка может означать, что последовательности runbook, выполняемые в песочнице Azure, не могут выполняться в [полноязыковом режиме](/powershell/module/microsoft.powershell.core/about/about_language_modes).

### <a name="resolution"></a>Решение

Эту проблему можно решить двумя способами:

* Вместо [Start-Job](/powershell/module/microsoft.powershell.core/start-job) используйте [Start-AzAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook) для запуска runbook.
* Попытайтесь запустить runbook с использованием гибридной рабочей роли Runbook.

Дополнительные сведения об этом и других видах поведения последовательностей runbook службы автоматизации Azure см. в статье [о выполнении runbook в службе автоматизации Azure](../automation-runbook-execution.md).

## <a name="scenario-a-long-running-runbook-fails-to-complete"></a><a name="long-running-runbook"></a>Сценарий. Не завершается работа долго выполняющейся последовательности runbook

### <a name="issue"></a>Проблема

После выполнения в течение трех часов последовательность runbook обозначается как остановленная. Возможно, отобразится следующее сообщение об ошибке:

```error
The job was evicted and subsequently reached a Stopped state. The job cannot continue running.
```

Это предусмотренная реакция на событие в песочницах Azure, которая связана с мониторингом [справедливого распределения](../automation-runbook-execution.md#fair-share) процессов в службе автоматизации Azure. Если последовательность runbook выполняется дольше трех часов, служба справедливого распределения останавливает ее. Состояние последовательности runbook, для который превышено предельное время для справедливого распределения, зависит от типа runbook. Последовательности runbook PowerShell и Python переходят в состояние "Остановлено". Последовательности runbook рабочих процессов PowerShell переходят в состояние "Сбой".

### <a name="cause"></a>Причина

Последовательность runbook выполнялся дольше трех часов (предельной продолжительности, заданной для песочницы Azure в рамках справедливого распределения).

### <a name="resolution"></a>Решение

Одно из рекомендуемых решений — запустить runbook в [гибридной рабочей роли Runbook](../automation-hrw-run-runbooks.md). Гибридные рабочие роли не ограничены тремя часами для справедливого распределения, как песочницы Azure. Последовательности runbook, выполняемые в гибридных рабочих ролях Runbook, следует разрабатывать с поддержкой перезапуска при возникновении непредвиденных проблем локальной инфраструктуры.

Еще одно решение — оптимизировать runbook с помощью [дочерних последовательностей runbook](../automation-child-runbooks.md). Если последовательность runbook циклически выполняет одну и ту же функцию для ряда ресурсов, например определенную операцию в нескольких базах данных, такую функцию можно переместить в дочернюю последовательность runbook. Каждая дочерняя последовательность runbook выполняется параллельно в отдельных процессах. Это уменьшает количество времени на завершение родительского модуля runbook.

Командлеты PowerShell для реализации дочерних runbook:

* [Start-AzAutomationRunbook](/powershell/module/Az.Automation/Start-AzAutomationRunbook). Этот командлет позволяет запустить runbook и передать в него параметры.
* [Get-AzAutomationJob](/powershell/module/Az.Automation/Get-AzAutomationJob). Если есть операции, которые нужно выполнить после выполнения дочерней последовательности runbook, этот командлет позволяет проверить состояние задания для каждой дочерней последовательности.

## <a name="scenario-error-in-job-streams-about-the-get_serializationsettings-method"></a><a name="get-serializationsettings"></a>Сценарий. Ошибка в потоках заданий из-за метода get_SerializationSettings

### <a name="issue"></a>Проблема

Для последовательности runbook в потоке задания происходит следующая ошибка:

```error
Connect-AzAccount : Method 'get_SerializationSettings' in type
'Microsoft.Azure.Management.Internal.Resources.ResourceManagementClient' from assembly
'Microsoft.Azure.Commands.ResourceManager.Common, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35'
does not have an implementation.
At line:16 char:1
+ Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -Appl ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Connect-AzAccount], TypeLoadException
    + FullyQualifiedErrorId : System.TypeLoadException,Microsoft.Azure.Commands.Profile.ConnectAzAccountCommand
```

### <a name="cause"></a>Причина

Возможно, эта ошибка вызвана неполной миграцией из AzureRM в модули Az в последовательности runbook. Эта ситуация может привести к тому, что служба автоматизации Azure запустит задание runbook с использованием только модулей AzureRM, а затем запустит другое задание, используя только модули Az, что приводит к аварийному завершению работы песочницы.

### <a name="resolution"></a>Решение

Мы не рекомендуем использовать командлеты Az и AzureRM в одной последовательности runbook. Дополнительные сведения о правильном использовании модулей см. в разделе [Миграция в модули Az](../shared-resources/modules.md#migrate-to-az-modules).

## <a name="scenario-access-denied-when-using-azure-sandbox-for-runbook-or-application"></a><a name="access-denied-azure-sandbox"></a>Сценарий. Отказано в доступе при использовании песочницы Azure для runbook или приложения

### <a name="issue"></a>Проблема

При попытке запуска runbook или приложения в песочнице Azure среда запрещает доступ.

### <a name="cause"></a>Причина

Эта проблема может возникать, поскольку песочницы Azure запрещают доступ ко всем COM-серверам вне процессов. Например, изолированное приложение или последовательность runbook не может вызывать инструментарий управления Windows (WMI) или службу установщика Windows (msiserver.exe).

### <a name="resolution"></a>Решение

Дополнительные сведения об использовании изолированных программ Azure см. в разделе [Среда выполнения runbook](../automation-runbook-execution.md#runbook-execution-environment).

## <a name="scenario-invalid-forbidden-status-code-when-using-key-vault-inside-a-runbook"></a>Сценарий: Недопустимый код состояния "Запрещено" при использовании Key Vault в runbook

### <a name="issue"></a>Проблема

При попытке получить доступ к Azure Key Vault с помощью последовательности runbook службы автоматизации Azure происходит следующая ошибка:

```error
Operation returned an invalid status code 'Forbidden'
```

### <a name="cause"></a>Причина

Возможны следующие причины этой проблемы.

* Не используется учетная запись запуска от имени.
* Недостаточно разрешений.

### <a name="resolution"></a>Решение

#### <a name="not-using-a-run-as-account"></a>Не используется учетная запись запуска от имени

Выполните шаг 5 [Добавление проверки подлинности для управления ресурсами Azure](../learn/automation-tutorial-runbook-textual-powershell.md#step-5---add-authentication-to-manage-azure-resources) и убедитесь, что для доступа к Key Vault используется учетная запись запуска от имени.

#### <a name="insufficient-permissions"></a>Недостаточные разрешения

[Добавьте разрешения для Key Vault](../manage-runas-account.md#add-permissions-to-key-vault), чтобы обеспечить учетной записи запуска от имени достаточные разрешения для доступа к Key Vault.

## <a name="recommended-documents"></a>Рекомендуемые документы

* [Выполнение модуля Runbook в службе автоматизации Azure](../automation-runbook-execution.md)
* [Запуск модуля Runbook в службе автоматизации Azure](../start-runbooks.md)

## <a name="next-steps"></a>Дальнейшие действия

Если вы не нашли здесь свою проблему или вам не удалось ее решить, попробуйте использовать один из следующих каналов для получения дополнительной поддержки.

* Получите ответы специалистов Azure на [форумах Azure](https://azure.microsoft.com/support/forums/).
* Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами. Служба поддержки Azure взаимодействует с сообществом Azure, предоставляя ответы, поддержку и советы экспертов.
* Если вам нужна дополнительная помощь, отправьте запрос в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и выберите **Получить поддержку**.
