---
title: Управление учетными данными в службе автоматизации Azure
description: В этой статье рассказывается, как создать ресурсы-контейнеры учетных данных и использовать их в модуле Runbook или конфигурации DSC.
services: automation
ms.subservice: shared-capabilities
ms.date: 12/22/2020
ms.topic: conceptual
ms.openlocfilehash: 9b9e42d55a982aeb55d7c9e26f7b1a6cbca32e0a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99052810"
---
# <a name="manage-credentials-in-azure-automation"></a>Управление учетными данными в службе автоматизации Azure

В ресурсе-контейнере учетных данных службы автоматизации хранится объект, содержащий учетные данные безопасности, например имя пользователя и пароль. Модули Runbook и конфигурации DSC используют командлеты, которые принимают объект [PSCredential](/dotnet/api/system.management.automation.pscredential) для проверки подлинности. Кроме того, они могут извлекать имя пользователя и пароль объекта `PSCredential`, чтобы предоставить их приложению или службе, для доступа к которым требуется проверка подлинности.

>[!NOTE]
>Безопасные средства в службе автоматизации Azure включают учетные данные, сертификаты, подключения и зашифрованные переменные. Эти ресурсы шифруются и хранятся в службе автоматизации Azure с помощью уникального ключа, который создается для каждой учетной записи службы автоматизации. Служба автоматизации Azure хранит ключ в управляемом системой хранилище ключей Key Vault. Перед сохранением защищенного ресурса служба автоматизации Azure загружает ключ из Key Vault, а затем использует его для шифрования ресурса. 

[!INCLUDE [gdpr-dsr-and-stp-note.md](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="powershell-cmdlets-used-to-access-credentials"></a>Командлеты PowerShell, используемые для доступа к учетным данным

Командлеты, представленные в следующей таблице, используются для создания учетных данных службы автоматизации с помощью PowerShell и управления ими. Они поставляются в составе [модулей Az](modules.md#az-modules).

| Командлет | Описание |
|:--- |:--- |
| [Get-AzAutomationCredential](/powershell/module/az.automation/get-azautomationcredential) |Извлекает объект [CredentialInfo](/dotnet/api/microsoft.azure.commands.automation.model.credentialinfo), содержащий метаданные учетных данных. Командлет не извлекает сам объект `PSCredential`.  |
| [New-AzAutomationCredential](/powershell/module/az.automation/new-azautomationcredential) |Создает новые учетные данные службы автоматизации. |
| [Remove-AzAutomationCredential](/powershell/module/az.automation/remove-azautomationcredential) |Удаляет учетные данные службы автоматизации. |
| [Set-AzAutomationCredential](/powershell/module/az.automation/set-azautomationcredential) |Задает свойства для существующих учетных данных службы автоматизации. |

## <a name="other-cmdlets-used-to-access-credentials"></a>Другие командлеты, используемые для доступа к учетным данным

Командлеты в следующей таблице используются для доступа к учетным данным в модулях Runbook и конфигурациях DSC. 

| Командлет | Описание |
|:--- |:--- |
| `Get-AutomationPSCredential` |Получает объект `PSCredential` для использования в модуле Runbook или в конфигурации DSC. Чаще всего используется именно этот [внутренний командлет](modules.md#internal-cmdlets), а не командлет `Get-AzAutomationCredential`, поскольку последний извлекает только сведения об учетных данных. Эта информация обычно не представляет ценности для передачи другому командлету. |
| [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential) |Получает учетные данные с запросом имени пользователя и пароля. Этот командлет является частью модуля Microsoft.PowerShell.Security по умолчанию. См. раздел [Модули по умолчанию](modules.md#default-modules).|
| [New-AzureAutomationCredential](/powershell/module/servicemanagement/azure.service/new-azureautomationcredential) | Создает ресурс-контейнер учетных данных. Этот командлет является частью модуля Azure по умолчанию. См. раздел [Модули по умолчанию](modules.md#default-modules).|

Чтобы получить объекты `PSCredential` в коде, необходимо импортировать модуль `Orchestrator.AssetManagement.Cmdlets`. Дополнительные сведения см. в статье [Администрирование модулей в службе автоматизации Azure](modules.md).

```powershell
Import-Module Orchestrator.AssetManagement.Cmdlets -ErrorAction SilentlyContinue
```

> [!NOTE]
> Не следует использовать переменные в параметре `Name` `Get-AutomationPSCredential`. Их использование может усложнить обнаружение зависимостей между модулями Runbook или конфигурациями DSC и ресурсами-контейнерами учетных данных во время разработки.

## <a name="python-functions-that-access-credentials"></a>Функции Python, обращающиеся к учетным данным

Функция, приведенная в следующей таблице, используется для доступа к учетным данным в модуле Runbook Python 2 и 3. Модули Runbook Python 3 сейчас доступны в предварительной версии.

| Компонент | Описание |
|:---|:---|
| `automationassets.get_automation_credential` | Извлекает сведения о ресурсе учетных данных. |

> [!NOTE]
> Импортируйте модуль `automationassets` в верхнюю часть модуля Runbook Python, чтобы получить доступ к функциям ресурса.

## <a name="create-a-new-credential-asset"></a>Создание ресурса-контейнера учетных данных

Ресурс-контейнер учетных данных можно создать с помощью портала Azure или Windows PowerShell.

### <a name="create-a-new-credential-asset-with-the-azure-portal"></a>Создание нового ресурса-контейнера учетных данных на портале Azure

1. В учетной записи службы автоматизации на панели слева выберите **учетные данные** в разделе **Общие ресурсы**.
2. На странице **учетные данные** выберите **Добавить учетные данные**.
3. В области "Новые учетные данные" введите соответствующее имя учетных данных, следуя стандартам именования.
4. Введите идентификатор доступа в поле **Имя пользователя**.
5. В полях пароля введите секретный ключ доступа.

    ![Создание новых учетных данных](../media/credentials/credential-create.png)

6. Если установлен флажок многофакторной проверки подлинности, снимите его.
7. Нажмите **Создать**, чтобы создать новый ресурс-контейнер учетных данных.

> [!NOTE]
> Служба автоматизации Azure не поддерживает учетные записи пользователей, которые используют многофакторную проверку подлинности.

### <a name="create-a-new-credential-asset-with-windows-powershell"></a>Создание нового ресурса-контейнера учетных данных с помощью Windows PowerShell

В примере ниже демонстрируется порядок создания новых учетных данных службы автоматизации Azure. Сначала создается объект `PSCredential` с именем и паролем, после чего он используется для создания ресурса-контейнера учетных данных. Вместо этого можно использовать командлет `Get-Credential`, чтобы предложить пользователю ввести имя и пароль.

```powershell
$user = "MyDomain\MyUser"
$pw = ConvertTo-SecureString "PassWord!" -AsPlainText -Force
$cred = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $user, $pw
New-AzureAutomationCredential -AutomationAccountName "MyAutomationAccount" -Name "MyCredential" -Value $cred
```

## <a name="get-a-credential-asset"></a>Получение ресурса-контейнера учетных данных

Модуль Runbook или Конфигурация DSC получают ресурс-контейнер учетных данных с помощью внутреннего командлета `Get-AutomationPSCredential`. Этот командлет получает объект `PSCredential`, который можно использовать с командлетом, для которого требуются учетные данные. Можно также получить свойства объекта учетных данных, чтобы использовать их по отдельности. Объект содержит свойства имени пользователя и защищенного пароля. 

> [!NOTE]
> Командлет `Get-AzAutomationCredential` не извлекает объект `PSCredential`, который можно использовать для проверки подлинности. Он только предоставляет сведения об учетных данных. Если необходимо применить учетные данные в модуле Runbook, необходимо получить их как объект `PSCredential`, используя `Get-AutomationPSCredential`.

Кроме того, можно использовать метод [GetNetworkCredential](/dotnet/api/system.management.automation.pscredential.getnetworkcredential) для получения объекта [NetworkCredential](/dotnet/api/system.net.networkcredential), который представляет собой незащищенную версию пароля.

### <a name="textual-runbook-example"></a>Пример текстового модуля Runbook

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Команды в приведенном ниже примере демонстрируют использование учетных данных PowerShell в модуле Runbook. Он получает учетные данные и присваивает переменным имя пользователя и пароль.

```powershell
$myCredential = Get-AutomationPSCredential -Name 'MyCredential'
$userName = $myCredential.UserName
$securePassword = $myCredential.Password
$password = $myCredential.GetNetworkCredential().Password
```

Вы также можете использовать учетные данные для проверки подлинности в Azure с помощью [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). В большинстве случаев следует использовать параметр [Учетная запись запуска от имени](../automation-security-overview.md#run-as-accounts) и получить подключение с помощью [Get-AzAutomationConnection](../automation-connections.md).

```powershell
$myCred = Get-AutomationPSCredential -Name 'MyCredential'
$userName = $myCred.UserName
$securePassword = $myCred.Password
$password = $myCred.GetNetworkCredential().Password

$myPsCred = New-Object System.Management.Automation.PSCredential ($userName,$securePassword)

Connect-AzAccount -Credential $myPsCred
```

# <a name="python-2"></a>[Python 2;](#tab/python2)

Ниже приведен пример доступа к учетным данным в модулях Runbook Python 2.

```python
import automationassets
from automationassets import AutomationAssetNotFound

# get a credential
cred = automationassets.get_automation_credential("credtest")
print cred["username"]
print cred["password"]
```

# <a name="python-3"></a>[Python 3](#tab/python3);

В следующем примере показан пример доступа к учетным данным в модулях Runbook для Python 3 (Предварительная версия).

```python
import automationassets
from automationassets import AutomationAssetNotFound

# get a credential
cred = automationassets.get_automation_credential("credtest")
print (cred["username"])
print (cred["password"])
```

---

### <a name="graphical-runbook-example"></a>Пример графического модуля Runbook

Можно добавить действие для внутреннего командлета `Get-AutomationPSCredential` в графический модуль Runbook, щелкнув правой кнопкой мыши учетные данные в области "Библиотека" графического редактора и выбрав пункт **Добавить на холст**.

![Добавление командлета Credential на холст](../media/credentials/credential-add-canvas.png)

На следующем рисунке показан пример использования учетных данных в графическом Runbook. В этом случае учетные данные обеспечивают проверку подлинности для модуля Runbook для доступа к ресурсам Azure, как описано в статье [Использование Azure AD в службе автоматизации Azure для проверки подлинности в Azure](../automation-use-azure-ad.md). Первое действие получает учетные данные с доступом к подписке Azure. Затем действие подключения учетной записи использует эти учетные данные для проверки подлинности любых последующих действий. Здесь используется [конвейерная связь](../automation-graphical-authoring-intro.md#use-links-for-workflow), поскольку параметр `Get-AutomationPSCredential` ожидает один объект.  

![Рабочий процесс учетных данных с примером связи конвейера](../media/credentials/get-credential.png)

## <a name="use-credentials-in-a-dsc-configuration"></a>Использование учетных данных в конфигурации DSC

Несмотря на то, что конфигурации DSC в службе автоматизации Azure могут работать с ресурсами-контейнерами учетных данных с помощью `Get-AutomationPSCredential`, они также могут передавать ресурсы-контейнеры учетных данных посредством параметров. Дополнительные сведения см. в статье [Компилирование конфигураций в Azure Automation DSC](../automation-dsc-compile.md#credential-assets).

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о командлетах, используемых для доступа к сертификатам, см. в статье [Управление модулями в службе автоматизации Azure](modules.md).
* Общие сведения о runbook см. в статье [Выполнение модуля Runbook в службе автоматизации Azure](../automation-runbook-execution.md).
* Дополнительные сведения о конфигурациях DSC см. в разделе [Общие сведения о настройке состояния службы автоматизации Azure](../automation-dsc-overview.md).