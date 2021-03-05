---
title: Создание учетной записи запуска от имени службы автоматизации Azure
description: В этой статье рассказывается, как создать учетную запись запуска от имени с помощью PowerShell или портал Azure.
services: automation
ms.subservice: process-automation
ms.date: 01/06/2021
ms.topic: conceptual
ms.openlocfilehash: ef6afff30da48b79b42e5fb4b3c72c3500f22dd1
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102172309"
---
# <a name="how-to-create-an-azure-automation-run-as-account"></a>Как создать учетную запись запуска от имени службы автоматизации Azure

Учетные записи запуска от имени в службе автоматизации Azure обеспечивают проверку подлинности для управления ресурсами в Azure Resource Manager или классической модели развертывания Azure с помощью модулей Runbook автоматизации и других функций автоматизации. В этой статье описывается, как создать учетную запись запуска от имени или классическая из портал Azure или Azure PowerShell.

## <a name="create-account-in-azure-portal"></a>Создание учетной записи в портал Azure

Выполните действия, с помощью которых на портале Azure можно обновить учетную запись службы автоматизации Azure. Учетные записи запуска от имени и классические запуска от имени создаются отдельно. Если вам не нужно управлять классическими ресурсами, достаточно создать учетную запись запуска от имени Azure.

1. Войдите на портал Azure с помощью учетной записи, которая является участником роли "Администраторы подписки" и соадминистратором подписки.

2. Найдите и выберите раздел **Учетные записи службы автоматизации**.

3. На странице **учетные записи службы автоматизации** выберите учетную запись службы автоматизации из списка.

4. В левой области выберите **учетные записи запуска от имени** в разделе **Параметры учетной записи** .

    :::image type="content" source="media/create-run-as-account/automation-account-properties-pane.png" alt-text="Выберите параметр Учетная запись запуска от имени.":::

5. В зависимости от требуемой учетной записи используйте панель " **+ Azure: учетная запись запуска от имени** " или " **Классическая учетная запись запуска от имени Azure** ". Просмотрев общие сведения, щелкните **Создать**.

    :::image type="content" source="media/create-run-as-account/automation-account-create-run-as.png" alt-text="Выберите параметр для создания учетной записи запуска от имени":::

6. Ход создания учетной записи запуска от имени в Azure можно отслеживать в разделе **Уведомления** в меню. Кроме того, отображается баннер с уведомлением о создании учетной записи. Процесс создания может занять несколько минут.

## <a name="create-account-using-powershell"></a>Создание учетной записи с помощью PowerShell

В следующем списке приведены требования для создания учетной записи запуска от имени в PowerShell с помощью предоставленного скрипта. Эти требования применяются к обоим типам учетных записей запуска от имени.

* Windows 10 или Windows Server 2016 с модулями Azure Resource Manager версии 3.4.1 и выше. Сценарий PowerShell не поддерживает более ранние версии Windows.
* Azure PowerShell PowerShell 6.2.4 или более поздней версии. Дополнительные сведения см. [в разделе Установка и настройка Azure PowerShell](/powershell/azure/install-az-ps).
* Учетная запись службы автоматизации, указанная в качестве значений параметров `AutomationAccountName` и `ApplicationDisplayName`.
* Разрешения, аналогичные списку в разделе [разрешений для настройки учетной записи запуска от имени](automation-security-overview.md#permissions).

Чтобы получить значения для `AutomationAccountName` , `SubscriptionId` и `ResourceGroupName` , которые являются обязательными параметрами для скрипта PowerShell, выполните следующие действия.

1. Войдите на портал Azure.

1. Найдите и выберите раздел **Учетные записи службы автоматизации**.

1. На странице учетных записей службы автоматизации выберите в списке нужную учетную запись.

1. На левой панели выберите **Свойства**.

1. Обратите внимание на значения параметров **имя**, **идентификатор подписки** и **Группа ресурсов** на странице **Свойства** .

   ![Страница свойств учетной записи службы автоматизации](media/create-run-as-account/automation-account-properties.png)

### <a name="powershell-script-to-create-a-run-as-account"></a>Сценарий PowerShell для создания учетной записи запуска от имени

Сценарий PowerShell включает поддержку нескольких конфигураций.

* создание учетной записи запуска от имени с использованием самозаверяющего сертификата;
* Создайте учетную запись запуска от имени и (или) классическую учетную запись запуска от имени с помощью самозаверяющего сертификата.
* Создайте учетную запись запуска от имени и (или) классическую учетную запись запуска от имени, используя сертификат, выданный центром сертификации (ЦС) предприятия.
* Создайте учетную запись запуска от имени и (или) классическую учетную запись запуска от имени, используя самозаверяющий сертификат в облаке Azure для государственных организаций.

1. Скачайте и сохраните скрипт в локальной папке с помощью следующей команды.

    ```powershell
    wget https://raw.githubusercontent.com/azureautomation/runbooks/master/Utility/AzRunAs/Create-RunAsAccount.ps1 -outfile Create-RunAsAccount.ps1
    ```

2. Запустите PowerShell с повышенными правами пользователя и перейдите в папку, содержащую скрипт.

3. Выполните одну из приведенных ниже команд, чтобы создать учетную запись запуска от имени и (или) классический Запуск от имени в соответствии с вашими требованиями.

    * Создайте учетную запись запуска от имени с помощью самозаверяющего сертификата.

        ```powershell
        .\Create-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $false
        ```

    * создание учетной записи запуска от имени и классической учетной записи запуска от имени с использованием самозаверяющего сертификата;

        ```powershell
        .\Create-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true
        ```

    * создание учетной записи запуска от имени и классической учетной записи запуска от имени Azure с использованием корпоративного сертификата;

        ```powershell
        .\Create-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication>  -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true -EnterpriseCertPathForRunAsAccount <EnterpriseCertPfxPathForRunAsAccount> -EnterpriseCertPlainPasswordForRunAsAccount <StrongPassword> -EnterpriseCertPathForClassicRunAsAccount <EnterpriseCertPfxPathForClassicRunAsAccount> -EnterpriseCertPlainPasswordForClassicRunAsAccount <StrongPassword>
        ```

        Если вы создали классическую учетную запись запуска от имени с использованием открытого корпоративного сертификата (CER-файл), используйте этот сертификат. Сценарий создает и сохраняет его в папке временных файлов на компьютере в профиле пользователя, `%USERPROFILE%\AppData\Local\Temp` который использовался для выполнения сеанса PowerShell. Ознакомьтесь с разделом [Передача сертификата управления службами Azure](../cloud-services/cloud-services-configure-ssl-certificate-portal.md).

    * Создание учетной записи запуска от имени и классической учетной записи запуска от имени с использованием самозаверяющего сертификата в облаке Azure для государственных организаций.

        ```powershell
        .\Create-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true -EnvironmentName AzureUSGovernment
        ```

4. После выполнения сценария появится запрос на аутентификацию в Azure. Войдите с помощью учетной записи, которая является членом роли администраторов подписки. При создании классической учетной записи запуска от имени ваша учетная запись должна быть соадминистратором подписки.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о графической разработке см. в статье [о создании графических runbook в службе автоматизации Azure](automation-graphical-authoring-intro.md).
* Чтобы начать работу с модулями runbook PowerShell, изучите документ [Руководство. Создание модуля runbook PowerShell](learn/automation-tutorial-runbook-textual-powershell.md).
* Чтобы приступить к работе с модулем Runbook для Python 3, см. раздел [учебник. Создание модуля Runbook Python 3](learn/automation-tutorial-runbook-textual-python-3.md).
