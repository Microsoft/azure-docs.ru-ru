---
title: Управление службами Office 365 с помощью службы автоматизации Azure
description: В этой статье описывается, как управлять службами подписки Office 365 с помощью службы автоматизации Azure.
services: automation
ms.date: 11/05/2020
ms.topic: conceptual
ms.openlocfilehash: 70c8892969a3b13175c60a4e20e0cf9086112abe
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93398051"
---
# <a name="manage-office-365-services"></a>Управление службами Office 365

Службу автоматизации Azure можно использовать для управления службами подписки Office 365 для таких продуктов, как Microsoft Word и Microsoft Outlook. Взаимодействие с Office 365 выполняется через [Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md). Сведения см. в статье [Использование Azure AD в службе автоматизации Azure для проверки подлинности в Azure](automation-use-azure-ad.md).

## <a name="prerequisites"></a>Предварительные требования

Для управления службами подписки Office 365 в службе автоматизации Azure вам потребуется следующее:

* Подписка Azure. См. [Руководство по выбору модели подписки](/azure/cloud-adoption-framework/decision-guides/subscriptions/).
* Объект службы автоматизации в Azure для хранения учетных данных учетной записи пользователя и последовательностей runbook. См. [Общие сведения о службе автоматизации Azure](./automation-intro.md).
* Azure AD. Сведения см. в статье [Использование Azure AD в службе автоматизации Azure для проверки подлинности в Azure](automation-use-azure-ad.md).
* Арендатор Office 365 и учетная запись. См. сведения в статье [Настройка клиента Office 365](/sharepoint/dev/spfx/set-up-your-developer-tenant).

## <a name="install-the-msonline-and-msonlineext-modules"></a>Установка модулей MSOnline и MSOnlineExt

Чтобы использовать Office 365 в службе автоматизации Azure, требуется Microsoft Azure Active Directory для Windows PowerShell (модуль `MSOnline`). Кроме того, вам потребуется модуль [`MSOnlineExt`](https://www.powershellgallery.com/packages/MSOnlineExt/1.0.35), который упрощает управление Azure AD в средах с одним и несколькими арендаторами. Установка этих модулей описана в статье [Использование Azure AD в службе автоматизации Azure для проверки подлинности в Azure](automation-use-azure-ad.md).

>[!NOTE]
>Чтобы использовать MSOnline PowerShell, нужно быть членом Azure AD. Гостевые пользователи не могут использовать этот модуль.

## <a name="create-an-azure-automation-account"></a>Создание учетной записи службы автоматизации Azure

Чтобы выполнить описанные в этой статье действия, вам нужна учетная запись службы автоматизации Azure. Сведения см. в статье [Создание учетной записи службы автоматизации Azure.](automation-quickstart-create-account.md)
 
## <a name="add-msonline-and-msonlineext-as-assets"></a>Добавление модулей MSOnline и MSOnlineExt в качестве ресурсов

Теперь добавьте установленные модули MSOnline и MSOnlineExt, чтобы включить возможности Office 365. Сведения см. в статье [Администрирование модулей в службе автоматизации Azure](shared-resources/modules.md).

1. На портале Azure выберите раздел **Учетные записи службы автоматизации**.
2. Выберите нужную учетную запись службы автоматизации.
3. Щелкните **Коллекция модулей** в разделе **Общие ресурсы**.
4. Найдите MSOnline.
5. Выберите модуль PowerShell `MSOnline` и щелкните **Импорт**, чтобы импортировать этот модуль в качестве ресурса.
6. Повторите шаги 4 и 5, чтобы выбрать и импортировать модуль `MSOnlineExt`.

## <a name="create-a-credential-asset-optional"></a>Создание ресурса учетных данных (необязательно)

При необходимости вы можете создать ресурс учетных данных для пользователя с правами администратора Office 365, который имеет разрешения на выполнение скрипта. Это удобно для того, чтобы не предоставлять имена пользователей и пароли в скриптах PowerShell. Инструкции см. в разделе [Создание ресурса учетных данных](automation-use-azure-ad.md#create-a-credential-asset).

## <a name="create-an-office-365-service-account"></a>Создание учетной записи службы Office 365

Для запуска служб подписки Office 365 требуется учетная запись службы Office 365 с разрешениями на выполнение необходимых действий. Вы можете использовать одну учетную запись глобального администратора, отдельную учетную запись для каждой службы либо создать одну функцию или скрипт для выполнения. В любом случае для учетной записи службы нужно создать сложный и безопасный пароль. Сведения см. в статье [Настройка Microsoft 365 для бизнеса](/microsoft-365/admin/setup/setup).

## <a name="connect-to-the-azure-ad-online-service"></a>Подключение к интернет-службе AAD

>[!NOTE]
>Чтобы использовать командлеты модуля MSOnline, необходимо запустить их из Windows PowerShell. PowerShell Core не поддерживает эти командлеты.

Вы можете подключиться к Azure AD из подписки Office 365 с помощью модуля MSOnline. Такое подключение использует имя пользователя и пароль для Office 365 или многофакторную проверку подлинности (MFA). Подключиться можно с помощью портала Azure или из командной строки Windows PowerShell (не требуется повышать уровень).

Ниже приведен пример для PowerShell. Командлет [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential) запрашивает учетные данные и сохраняет их в переменной `Msolcred`. Затем командлет [Connect-MsolService](/powershell/module/msonline/connect-msolservice) применяет учетные данные для подключения к интернет-службе каталогов Azure. Если вы хотите подключиться к конкретной среде Azure, используйте параметр `AzureEnvironment`.

```powershell
$Msolcred = Get-Credential
Connect-MsolService -Credential $MsolCred -AzureEnvironment "AzureCloud"
```

Если ошибки не появляются, значит подключение успешно установлено. Это можно быстро проверить, запустив любой командлет Office 365, например `Get-MsolUser`, и просмотрев его результат. Если будут возникать ошибки, проверьте самую распространенную причину — неправильный пароль.

>[!NOTE]
>Также для подключения к Azure AD из подписки Office 365 можно использовать модуль AzureRM или модуль Az. Основной командлет для подключения — [Connect-AzureAD](/powershell/module/azuread/connect-azuread). Этот командлет поддерживает параметр `AzureEnvironmentName` для определенных сред Office 365.

## <a name="create-a-powershell-runbook-from-an-existing-script"></a>Создание runbook PowerShell из существующего скрипта

Доступ к возможностям Office 365 осуществляется из скрипта PowerShell. Ниже приведен пример скрипта для учетных данных с именем `Office-Credentials` и именем пользователя `admin@TenantOne.com`. Он применяет `Get-AutomationPSCredential` для импорта учетных данных Office 365.

```powershell
$emailFromAddress = "admin@TenantOne.com"
$emailToAddress = "servicedesk@TenantOne.com"
$emailSMTPServer = "outlook.office365.com"
$emailSubject = "Office 365 License Report"

$credObject = Get-AutomationPSCredential -Name "Office-Credentials"
Connect-MsolService -Credential $credObject

$O365Licenses = Get-MsolAccountSku | Out-String
Send-MailMessage -Credential $credObject -From $emailFromAddress -To $emailToAddress -Subject $emailSubject -Body $O365Licenses -SmtpServer $emailSMTPServer -UseSSL
```

## <a name="run-the-script-in-a-runbook"></a>Запуск скрипта в runbook

Вы можете применять скрипты в runbook службы автоматизации Azure. Мы продемонстрируем это на примере runbook для PowerShell.

1. Создайте runbook PowerShell. Сведения см. в статье [Создание runbook службы автоматизации Azure](./automation-quickstart-create-runbook.md).
2. В учетной записи службы автоматизации выберите **Runbook** в области **Автоматизация процессов**.
3. Выберите runbook и щелкните **Правка**.
4. Скопируйте скрипт и вставьте его в текстовый редактор для runbook.
5. Выберите **Ресурсы**, а затем разверните раздел **Учетные данные** и убедитесь, что там есть учетные данные Office 365.
6. Выберите команду **Сохранить**.
7. Выберите **Область тестирования** и щелкните **Запустить**, чтобы начать тестирование runbook. Сведения см. в статье [Управление последовательностями runbook в службе автоматизации Azure](./manage-runbooks.md).
8. После завершения тестирования выйдите из области тестирования.

## <a name="publish-and-schedule-the-runbook"></a>Публикация runbook и создание расписания

Сведения о публикации runbook и создании расписания см. в статье [Управление последовательностями runbook в службе автоматизации Azure](./manage-runbooks.md).

## <a name="next-steps"></a>Дальнейшие действия

* Подробные сведения об использовании учетных данных см. в статье [Управление учетными данными в службе автоматизации Azure](shared-resources/credentials.md).
* Дополнительные сведения о модулях можно найти в статье [Администрирование модулей в службе автоматизации Azure](shared-resources/modules.md).
* При необходимости см. статью [Запуск runbook в службе автоматизации Azure](start-runbooks.md).
* Дополнительные сведения о PowerShell см. в [документации по PowerShell](/powershell/scripting/overview).
