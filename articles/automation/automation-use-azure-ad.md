---
title: Использование Azure AD в службе автоматизации Azure для проверки подлинности в Azure
description: В этой статье рассказывается, как в службе автоматизации Azure использовать Azure AD в качестве поставщика аутентификации в Azure.
services: automation
ms.date: 03/30/2020
ms.topic: conceptual
ms.openlocfilehash: 336c0387ac9febcc517c2ce358d0b04c80d10678
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99576809"
---
# <a name="use-azure-ad-to-authenticate-to-azure"></a>Аутентификация в Azure с помощью Azure AD

Служба [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) позволяет выполнять целый ряд административных задач, таких как управление пользователями, управление доменами или настройка единого входа. В этой статье описывается, как в службе автоматизации Azure использовать Azure AD в качестве поставщика аутентификации в Azure. 

## <a name="install-azure-ad-modules"></a>Установка модулей Azure AD

Возможностями Azure Active Directory можно воспользоваться через следующие модули PowerShell:

* Azure Active Directory PowerShell для Graph (модули AzureRM и Az). В состав службы автоматизации Azure входит модуль AzureRM и его недавнее обновление, модуль Az. Его функциональные возможности включают неинтерактивную аутентификацию в Azure на основе учетных данных пользователя Azure AD (OrgId). См. [описание Azure AD 2.0.2.76](https://www.powershellgallery.com/packages/AzureAD/2.0.2.76).

* Microsoft Azure Active Directory для Windows PowerShell (модуль MSOnline). Этот модуль обеспечивает взаимодействие с Microsoft Online, в том числе Microsoft 365.

>[!NOTE]
>PowerShell Core не поддерживает модуль MSOnline. Чтобы использовать командлеты этого модуля, необходимо запускать их из Windows PowerShell. Вместо модуля MSOnline мы рекомендуем использовать более новую версию модулей Azure Active Directory PowerShell для Graph. 

### <a name="preinstallation"></a>Действия перед установкой

Перед установкой модулей Azure AD на компьютере следует выполнить следующее:

* Удалите все предыдущие версии модулей AzureRM, Az и MSOnline. 

* Удалите помощника по входу в Microsoft Online Services, чтобы он не мешал правильной работе новых модулей PowerShell.  

### <a name="install-the-azurerm-and-az-modules"></a>Установка модулей AzureRM и Az

>[!NOTE]
>Для работы с этими модулями нужно использовать PowerShell 5.1 или более поздней версии на 64-разрядной версии Windows. 

1. Установите Windows Management Framework (WMF) 5.1. Подробные сведения см. в статье [Установка и настройка WMF 5.1](/powershell/scripting/wmf/setup/install-configure).

2. Установите AzureRM и (или) Az, следуя инструкциям из статьи [Установка Azure PowerShell в ОС Windows с помощью PowerShellGet](/powershell/azure/azurerm/install-azurerm-ps).

### <a name="install-the-msonline-module"></a>Установка модуля MSOnline

>[!NOTE]
>Чтобы установить модуль MSOnline, необходимо быть членом роли администратора. Сведения о ролях администраторов см. [здесь](/microsoft-365/admin/add-users/about-admin-roles).

1. Убедитесь, что на вашем компьютере включен компонент Microsoft .NET Framework 3.5.x. Вероятно, что на компьютере уже установлена более новая версия, но обратная совместимость платформы .NET Framework с более старыми версиями может быть включена или отключена. 

2. Установите 64-разрядную версию [помощника по входу в Microsoft Online Services](https://www.microsoft.com/Download/details.aspx?id=28177).

3. Откройте сеанс Windows PowerShell от имени администратора, чтобы запустить командную строку Windows PowerShell с повышенными привилегиями.

4. Разверните Azure Active Directory, используя [MSOnline 1.0](http://www.powershellgallery.com/packages/MSOnline/1.0).

5. Если появится запрос на установку поставщика NuGet, введите с клавиатуры Y и нажмите клавишу ВВОД.

6. Если появится запрос на установку модуля из коллекции [PSGallery](https://www.powershellgallery.com/), введите с клавиатуры Y и нажмите клавишу ВВОД.

### <a name="install-support-for-pscredential"></a>Установка поддержки для PSCredential

Служба автоматизации Azure использует класс [PSCredential](/dotnet/api/system.management.automation.pscredential) для представления ресурса учетных данных. Ваши скрипты извлекают объекты `PSCredential` через командлет `Get-AutomationPSCredential`. См. дополнительные сведения в статье [Управление учетными данными в службе автоматизации Azure](shared-resources/credentials.md).

## <a name="assign-a-subscription-administrator"></a>Назначение администратора подписки

Вам нужно назначить администратора для подписки Azure. Это должен быть пользователь с ролью владельца для области подписки. Подробные сведения см. в статье [Управление доступом на основе ролей в службе автоматизации Azure](automation-role-based-access-control.md). 

## <a name="change-the-azure-ad-users-password"></a>Смена пароля для пользователя Azure Active Directory

Чтобы сменить пароль для пользователя Azure Active Directory, выполните следующее:

1. Выйдите из Azure.

2. Попросите администратора выполнить вход в Azure от имени только что созданного пользователя Azure AD, используя полное имя пользователя (с указанием домена) и временный пароль. 

3. Попросите администратора изменить пароль в ответ на соответствующий запрос.

## <a name="configure-azure-automation-to-manage-the-azure-subscription"></a>Настройка управления подпиской Azure в службе автоматизации Azure

Чтобы служба автоматизации Azure могла взаимодействовать с Azure AD, нужно получить учетные данные, связанные с подключением Azure к Azure AD. В качестве учетных данных могут использоваться идентификатор клиента, идентификатор подписки и т. п. Дополнительные сведения о подключении между Azure и Azure AD см. в статье [Подключение вашей организации к Azure Active Directory](/azure/devops/organizations/accounts/connect-organization-to-azure-ad).

## <a name="create-a-credential-asset"></a>Создание ресурса учетных данных

Получив учетные данные Azure для Azure AD, вам нужно создать ресурс учетных данных службы автоматизации Azure для безопасного хранения учетных данных Azure AD, чтобы runbook и скрипты Desired State Configuration (DSC) могли их использовать. Это можно сделать с помощью портала Azure или командлетов PowerShell.

### <a name="create-the-credential-asset-in-azure-portal"></a>Создание ресурса учетных данных на портале Azure

Для создания ресурса учетных данных можно использовать портал Azure. Для этого выберите **Учетные данные** в области **Общие ресурсы** для учетной записи службы автоматизации. Подробные сведения см. в статье [Управление учетными данными в службе автоматизации Azure](shared-resources/credentials.md).

### <a name="create-the-credential-asset-with-windows-powershell"></a>Создание ресурса учетных данных с помощью Windows PowerShell

Чтобы подготовить новый ресурс учетных данных в Windows PowerShell, скрипт первым делом создает объект `PSCredential` с назначенным именем пользователя и паролем. Затем скрипт использует этот объект для создания ресурса, вызывая командлет [New-AzureAutomationCredential](/powershell/module/servicemanagement/azure.service/new-azureautomationcredential). Вместо этого скрипт может использовать командлет [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential), чтобы предложить пользователю ввести имя и пароль. Подробные сведения см. в статье [Управление учетными данными в службе автоматизации Azure](shared-resources/credentials.md). 

## <a name="manage-azure-resources-from-an-azure-automation-runbook"></a>Управление ресурсами Azure из runbook службы автоматизации Azure

Вы можете управлять ресурсами Azure из runbook службы автоматизации Azure, используя ресурс учетных данных. Ниже приведен пример runbook PowerShell, который собирает ресурс учетных данных и применяет его для остановки и запуска виртуальных машин в подписке Azure. Этот runbook сначала вызывает `Get-AutomationPSCredential` для получения учетных данных для аутентификации в Azure. Затем он вызывает командлет [Connect AzAccount](/powershell/module/az.accounts/connect-azaccount), чтобы подключиться к Azure с использованием полученных учетных данных. Затем скрипт вызывает командлет [Select-AzureSubscription](/powershell/module/servicemanagement/azure.service/select-azuresubscription), который позволяет выбрать подписку для работы. 

```azurepowershell
Workflow Stop-Start-AzureVM 
{ 
    Param 
    (    
        [Parameter(Mandatory=$true)][ValidateNotNullOrEmpty()] 
        [String] 
        $AzureSubscriptionId, 
        [Parameter(Mandatory=$true)][ValidateNotNullOrEmpty()] 
        [String] 
        $AzureVMList="All", 
        [Parameter(Mandatory=$true)][ValidateSet("Start","Stop")] 
        [String] 
        $Action 
    ) 
     
    $credential = Get-AutomationPSCredential -Name 'AzureCredential' 
    Connect-AzAccount -Credential $credential 
    Select-AzureSubscription -SubscriptionId $AzureSubscriptionId 
 
    if($AzureVMList -ne "All") 
    { 
        $AzureVMs = $AzureVMList.Split(",") 
        [System.Collections.ArrayList]$AzureVMsToHandle = $AzureVMs 
    } 
    else 
    { 
        $AzureVMs = (Get-AzVM).Name 
        [System.Collections.ArrayList]$AzureVMsToHandle = $AzureVMs 
 
    } 
 
    foreach($AzureVM in $AzureVMsToHandle) 
    { 
        if(!(Get-AzVM | ? {$_.Name -eq $AzureVM})) 
        { 
            throw " AzureVM : [$AzureVM] - Does not exist! - Check your inputs " 
        } 
    } 
 
    if($Action -eq "Stop") 
    { 
        Write-Output "Stopping VMs"; 
        foreach -parallel ($AzureVM in $AzureVMsToHandle) 
        { 
            Get-AzVM | ? {$_.Name -eq $AzureVM} | Stop-AzVM -Force 
        } 
    } 
    else 
    { 
        Write-Output "Starting VMs"; 
        foreach -parallel ($AzureVM in $AzureVMsToHandle) 
        { 
            Get-AzVM | ? {$_.Name -eq $AzureVM} | Start-AzVM 
        } 
    } 
}
```  

## <a name="next-steps"></a>Дальнейшие действия

* Подробные сведения об использовании учетных данных см. в статье [Управление учетными данными в службе автоматизации Azure](shared-resources/credentials.md).
* Дополнительные сведения о модулях можно найти в статье [Администрирование модулей в службе автоматизации Azure](shared-resources/modules.md).
* При необходимости см. статью [Запуск runbook в службе автоматизации Azure](start-runbooks.md).
* Дополнительные сведения о PowerShell см. в [документации по PowerShell](/powershell/scripting/overview).
