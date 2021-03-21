---
title: Управление подключениями в службе автоматизации Azure
description: В этой статье рассказывается, как управлять подключениями службы автоматизации Azure к внешним службам и приложениям, а также как работать с ними в модулях runbook.
services: automation
ms.subservice: shared-capabilities
ms.date: 12/22/2020
ms.topic: conceptual
ms.custom: has-adal-ref
ms.openlocfilehash: 5f6494eb72084c683ddbb8b27a49acdb1fbc2bfc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99054269"
---
# <a name="manage-connections-in-azure-automation"></a>Управление подключениями в службе автоматизации Azure

Ресурс подключения службы автоматизации Azure содержит приведенные ниже сведения. Эти сведения необходимы для подключения к внешней службе или приложению из модуля runbook либо конфигурации DSC. 

* Сведения, необходимые для проверки подлинности, такие как имя пользователя и пароль.
* Сведения о подключении, например URL-адрес или порт.

Ресурс подключения хранит все свойства для подключения к определенному приложению, что избавляет от необходимости создавать несколько переменных. Вы можете изменять значения для подключения в одном месте, а также передать имя подключения в модуль runbook или конфигурацию DSC в одном параметре. Модуль runbook или конфигурация обращается к свойствам подключения с помощью внутреннего командлета `Get-AutomationConnection`.

При создании подключения необходимо указать его тип. Тип подключения — это шаблон, определяющий набор его свойств. Вы можете добавить тип подключения в службу автоматизации Azure с помощью модуля интеграции с файлом метаданных. Тип подключения можно также создать с помощью [API службы автоматизации](/previous-versions/azure/reference/mt163818(v=azure.100)), если модуль интеграции содержит тип подключения и импортируется в учетную запись службы автоматизации. 

>[!NOTE]
>Безопасные средства в службе автоматизации Azure включают учетные данные, сертификаты, подключения и зашифрованные переменные. Эти ресурсы шифруются и хранятся в службе автоматизации Azure с помощью уникального ключа, который создается для каждой учетной записи службы автоматизации. Служба автоматизации Azure хранит ключ в управляемом системой хранилище ключей Key Vault. Перед сохранением защищенного ресурса служба автоматизации загружает ключ из Key Vault, а затем использует его для шифрования ресурса. 

## <a name="connection-types"></a>Типы подключений

При использовании службы автоматизации Azure доступны следующие встроенные типы подключений:

* `Azure` — подключение для управления классическими ресурсами.
* `AzureServicePrincipal` — подключение, используемое учетной записью запуска от имени Azure.
* `AzureClassicCertificate` — подключение, используемое классической версией учетной записи запуска от имени Azure.

В большинстве случаев создавать ресурс подключения не нужно, так как он создается вместе с [учетной записью запуска от имени](automation-security-overview.md).

## <a name="powershell-cmdlets-to-access-connections"></a>Командлеты PowerShell для доступа к подключениям

Командлеты, представленные в следующей таблице, используются для создания подключений службы автоматизации и управления ими с помощью PowerShell. Они предоставляются в составе [модулей Az](shared-resources/modules.md#az-modules).

|Командлет|Описание|
|---|---|
|[Get-AzAutomationConnection](/powershell/module/az.automation/get-azautomationconnection)|Извлекает сведения о подключении.|
|[New-AzAutomationConnection](/powershell/module/az.automation/new-azautomationconnection)|Создает новое подключение.|
|[Remove-AzAutomationConnection](/powershell/module/Az.Automation/Remove-AzAutomationConnection)|Удаляет существующее подключение.|
|[Set-AzAutomationConnectionFieldValue](/powershell/module/Az.Automation/Set-AzAutomationConnectionFieldValue)|Задает значение определенного поля для существующего подключения.|

## <a name="internal-cmdlets-to-access-connections"></a>Внутренние командлеты для доступа к подключениям

Внутренние командлеты в следующей таблице используются для доступа к подключениям в модулях runbook и конфигурациях DSC. Этот командлет предоставляется с глобальным модулем `Orchestrator.AssetManagement.Cmdlets`. Дополнительные сведения см. в статье [Внутренние командлеты](shared-resources/modules.md#internal-cmdlets).

|Внутренний командлет|Описание|
|---|---|
|`Get-AutomationConnection` | Извлекает значения различных полей в подключении и возвращает их в виде [хэш-таблицы](/powershell/module/microsoft.powershell.core/about/about_hash_tables). Затем эту хэш-таблицу можно использовать с соответствующими командами в модуле runbook или конфигурации DSC.|

>[!NOTE]
>Не следует использовать переменные с параметром `Name` командлета `Get-AutomationConnection`. Использование переменных в таком случае может усложнить обнаружение зависимостей между модулями runbook или конфигурациями DSC и ресурсами подключения во время разработки.

## <a name="python-functions-to-access-connections"></a>Функции Python для доступа к подключениям

Функция, приведенная в следующей таблице, используется для доступа к подключениям в модуле Runbook Python 2 и 3. Модули Runbook Python 3 сейчас доступны в предварительной версии.

| Компонент | Описание |
|:---|:---|
| `automationassets.get_automation_connection` | Извлекает подключение. Возвращает словарь со свойствами подключения. |

> [!NOTE]
> Импортируйте модуль `automationassets` в верхнюю часть модуля runbook на Python, чтобы получить доступ к функциям ресурса.

## <a name="create-a-new-connection"></a>Создание подключения

### <a name="create-a-new-connection-with-the-azure-portal"></a>Создание подключения на портале Azure

Чтобы создать новое подключение на портале Azure, сделайте следующее:

1. В учетной записи службы автоматизации в области **Общие ресурсы** выберите **Подключения**.
2. На странице "Подключения" щелкните **+ Добавить подключение**.
4. В поле **Тип** на панели "Новое подключение" выберите тип создаваемого подключения. Возможные варианты: `Azure`, `AzureServicePrincipal` или `AzureClassicCertificate`. 
5. В форме представлены свойства выбранного типа подключения. Заполните форму и нажмите кнопку **Создать** для сохранения нового подключения.

### <a name="create-a-new-connection-with-windows-powershell"></a>Создание подключения с помощью Windows PowerShell

Создайте новое подключение с помощью командлета `New-AzAutomationConnection` в Windows PowerShell. У этого командлета есть параметр `ConnectionFieldValues`, который ожидает хэш-таблицу, задающую значения для каждого свойства, определенного типом подключения.

Приведенные ниже примеры команд можно использовать в качестве альтернативы созданию учетной записи запуска от имени на портале для создания ресурса подключения.

```powershell
$ConnectionAssetName = "AzureRunAsConnection"
$ConnectionFieldValues = @{"ApplicationId" = $Application.ApplicationId; "TenantId" = $TenantID.TenantId; "CertificateThumbprint" = $Cert.Thumbprint; "SubscriptionId" = $SubscriptionId}
New-AzAutomationConnection -ResourceGroupName $ResourceGroup -AutomationAccountName $AutomationAccountName -Name $ConnectionAssetName -ConnectionTypeName AzureServicePrincipal -ConnectionFieldValues $ConnectionFieldValues
```

При создании учетной записи службы автоматизации в нее по умолчанию включается несколько глобальных модулей, а также тип подключения `AzureServicePrincipal` для создания ресурса подключения `AzureRunAsConnection`. При попытке использовать другой метод проверки подлинности вы не сможете создать ресурс подключения для подключения к службе или приложению, так как другие типы подключения не определены в учетной записи службы автоматизации. Дополнительные сведения о создании собственного типа подключения для пользовательского модуля см. в разделе [Добавление типа подключения](#add-a-connection-type).

## <a name="add-a-connection-type"></a>Добавление типа подключения

Если модуль runbook или конфигурация DSC подключается к внешней службе, необходимо определить тип подключения в [пользовательском модуле](shared-resources/modules.md#custom-modules), который называется модулем интеграции. Этот модуль включает файл метаданных, который задает свойства типа подключения, называется **&lt;ModuleName&gt;-Automation.json** и расположен в папке модуля **ZIP-файла**. Этот файл содержит поля подключения, необходимые для подключения к системе или службе, которую представляет модуль. С помощью этого файла можно задать имена полей, типы данных, состояние шифрования и необязательное состояние для типа подключения. 

Следующий пример — это шаблон в формате **JSON**, который определяет свойства имени пользователя и пароля для пользовательского типа подключения с именем `MyModuleConnection`.

```json
{
   "ConnectionFields": [
   {
      "IsEncrypted":  false,
      "IsOptional":  true,
      "Name":  "Username",
      "TypeName":  "System.String"
   },
   {
      "IsEncrypted":  true,
      "IsOptional":  false,
      "Name":  "Password",
      "TypeName":  "System.String"
   }
   ],
   "ConnectionTypeName":  "MyModuleConnection",
   "IntegrationModuleName":  "MyModule"
}
```

## <a name="get-a-connection-in-a-runbook-or-dsc-configuration"></a>Получение подключения в модуле runbook или конфигурации DSC

Получите подключение в модуле runbook или конфигурации DSC, используя внутренний командлет `Get-AutomationConnection`. Этот командлет является предпочтительным по отношению к командлету `Get-AzAutomationConnection`, так как он получает значения подключения вместо сведений о подключении.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

В следующем примере показано, как использовать учетную запись запуска от имени для проверки подлинности при обращении из модуля runbook к ресурсам Azure Resource Manager. Здесь используется ресурс подключения, представляющий учетную запись запуска от имени, которая ссылается на субъект-службу на основе сертификатов.

```powershell
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
```

# <a name="python"></a>[Python](#tab/python2)

В следующем примере показано, как выполнить проверку подлинности с помощью подключения запуска от имени в модуле Runbook Python 2 и 3.

```python
""" Tutorial to show how to authenticate against Azure resource manager resources """
import azure.mgmt.resource
import automationassets

def get_automation_runas_credential(runas_connection):
    """ Returns credentials to authenticate against Azure resource manager """
    from OpenSSL import crypto
    from msrestazure import azure_active_directory
    import adal

    # Get the Azure Automation Run As service principal certificate
    cert = automationassets.get_automation_certificate("AzureRunAsCertificate")
    pks12_cert = crypto.load_pkcs12(cert)
    pem_pkey = crypto.dump_privatekey(
        crypto.FILETYPE_PEM, pks12_cert.get_privatekey())

    # Get Run As connection information for the Azure Automation service principal
    application_id = runas_connection["ApplicationId"]
    thumbprint = runas_connection["CertificateThumbprint"]
    tenant_id = runas_connection["TenantId"]

    # Authenticate with service principal certificate
    resource = "https://management.core.windows.net/"
    authority_url = ("https://login.microsoftonline.com/" + tenant_id)
    context = adal.AuthenticationContext(authority_url)
    return azure_active_directory.AdalAuthentication(
        lambda: context.acquire_token_with_client_certificate(
            resource,
            application_id,
            pem_pkey,
            thumbprint)
    )


# Authenticate to Azure using the Azure Automation Run As service principal
runas_connection = automationassets.get_automation_connection(
    "AzureRunAsConnection")
azure_credential = get_automation_runas_credential(runas_connection)
```

---

### <a name="graphical-runbook-examples"></a>Примеры графического модуля runbook

Вы можете добавить действие для внутреннего командлета `Get-AutomationConnection` в графический модуль runbook. Щелкните правой кнопкой мыши подключение в области "Библиотека" графического редактора и выберите **Добавить на холст**.

![Добавление на холст](media/automation-connections/connection-add-canvas.png)

На следующем рисунке показан пример использования объекта подключения в графическом модуле runbook. В этом примере используется набор данных `Constant value` для действия `Get RunAs Connection`, в котором объект подключения используется для проверки подлинности. Здесь используется [конвейерная связь](automation-graphical-authoring-intro.md#use-links-for-workflow), так как набор параметров `ServicePrincipalCertificate` ожидает один объект.

![Установление подключений](media/automation-connections/automation-get-connection-object.png)

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о командлетах, используемых для доступа к подключениям, см. в статье [Администрирование модулей в службе автоматизации Azure](shared-resources/modules.md).
* Общие сведения о модулях runbook см. в статье [Выполнение модуля Runbook в службе автоматизации Azure](automation-runbook-execution.md).
* Дополнительные сведения о конфигурациях DSC см. в разделе [Общие сведения о настройке состояния](automation-dsc-overview.md).