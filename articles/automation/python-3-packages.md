---
title: Управление пакетами Python 3 в службе автоматизации Azure
description: В этой статье рассказывается, как управлять пакетами Python 3 (Предварительная версия) в службе автоматизации Azure.
services: automation
ms.subservice: process-automation
ms.date: 02/19/2021
ms.topic: conceptual
ms.openlocfilehash: fd4d8ee92b670bc2544619a0dce16a26d9342c13
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102122040"
---
# <a name="manage-python-3-packages-preview-in-azure-automation"></a>Управление пакетами Python 3 (Предварительная версия) в службе автоматизации Azure

Служба автоматизации Azure позволяет запускать модули Runbook для Python 3 (Предварительная версия) в Azure и гибридные рабочие роли Runbook для Linux. Чтобы упростить модули runbook, можно использовать пакеты Python для импорта необходимых модулей. Чтобы импортировать один пакет, см. раздел [Импорт пакета](#import-a-package). Сведения о импорте пакета с несколькими пакетами см. в разделе [Импорт пакета с зависимостями](#import-a-package-with-dependencies). В этой статье описывается, как управлять пакетами Python 3 (Предварительная версия) и использовать их в службе автоматизации Azure.

## <a name="import-a-package"></a>Импорт пакета

В учетной записи службы автоматизации выберите **пакеты Python** в разделе **Общие ресурсы**. Выберите **+ Добавить пакет Python**.

:::image type="content" source="media/python-3-packages/add-python-3-package.png" alt-text="Снимок экрана: страница пакетов Python 3 содержит пакеты Python 3 в меню слева и добавляет выделенный пакет Python 2.":::

На странице **Добавление пакета Python** выберите **Python 3** для **версии** и выберите локальный пакет для отправки. Пакет может иметь расширение файла **.whl** или **.tar.gz**. Когда пакет выбран, нажмите кнопку **ОК** , чтобы передать его.

:::image type="content" source="media/python-3-packages/upload-package.png" alt-text="На снимке экрана показана страница Добавление пакета Python 3 с выбранным файлом tar. gz.":::

После импорта пакета он отображается на странице пакеты Python в учетной записи службы автоматизации на вкладке **пакеты Python 3 (Предварительная версия)** . Если необходимо удалить пакет, выберите пакет и нажмите кнопку **Удалить**.

:::image type="content" source="media/python-3-packages/python-3-packages-list.png" alt-text="На снимке экрана показана страница пакетов Python 3 после импорта пакета.":::

### <a name="import-a-package-with-dependencies"></a>Импорт пакета с зависимостями

Вы можете импортировать пакет Python 3 и его зависимости, импортировав следующий скрипт Python в модуль Runbook Python 3, а затем запустив его.

```cmd
https://github.com/azureautomation/runbooks/blob/master/Utility/Python/import_py3package_from_pypi.py
```

#### <a name="importing-the-script-into-a-runbook"></a>Импорт скрипта в модуль Runbook
Сведения об импорте модуля Runbook см. в разделе [Импорт модуля Runbook из портал Azure](manage-runbooks.md#import-a-runbook-from-the-azure-portal). Скопируйте файл из репозитория GitHub в хранилище, к которому может получить доступ портал, прежде чем выполнять импорт.

На странице **Импорт модуля Runbook** по умолчанию используется имя модуля Runbook в соответствии с именем скрипта. Если у вас есть доступ к полю, можно изменить его имя. **Тип Runbook** может по умолчанию иметь **Python 2**. Если это так, обязательно измените его на **Python 3**.

:::image type="content" source="media/python-3-packages/import-python-3-package.png" alt-text="На снимке экрана показана страница импорта Runbook для Python 3.":::

#### <a name="executing-the-runbook-to-import-the-package-and-dependencies"></a>Запуск модуля Runbook для импорта пакета и зависимостей

После создания и публикации модуля Runbook запустите его, чтобы импортировать пакет. Дополнительные сведения о запуске модуля Runbook см. в статье [Запуск модуля Runbook в службе автоматизации Azure](start-runbooks.md) .

Для скрипта ( `import_py3package_from_pypi.py` ) требуются следующие параметры.

| Параметр | Описание |
|---------------|-----------------|
|subscription_id | Идентификатор подписки учетной записи службы автоматизации |
| resource_group | Имя группы ресурсов, в которой определена учетная запись службы автоматизации |
| automation_account | Имя учетной записи службы автоматизации |
| module_name | Имя модуля для импорта `pypi.org` |

Дополнительные сведения об использовании параметров с модулями Runbook см. в разделе [Работа с параметрами Runbook](start-runbooks.md#work-with-runbook-parameters).

## <a name="use-a-package-in-a-runbook"></a>Использование пакета в модуле runbook

Импортируемый пакет можно использовать в модуле Runbook. Добавьте следующий код, чтобы вывести список всех групп ресурсов в подписке Azure.

```python
import os  
import azure.mgmt.resource  
import automationassets  

def get_automation_runas_credential(runas_connection):  
    from OpenSSL import crypto  
    import binascii  
    from msrestazure import azure_active_directory  
    import adal 

    # Get the Azure Automation RunAs service principal certificate  
    cert = automationassets.get_automation_certificate("AzureRunAsCertificate")  
    pks12_cert = crypto.load_pkcs12(cert)  
    pem_pkey = crypto.dump_privatekey(crypto.FILETYPE_PEM,pks12_cert.get_privatekey())  

    # Get run as connection information for the Azure Automation service principal 
    application_id = runas_connection["ApplicationId"]  
    thumbprint = runas_connection["CertificateThumbprint"]  
    tenant_id = runas_connection["TenantId"]  

    # Authenticate with service principal certificate  
    resource ="https://management.core.windows.net/"  
    authority_url = ("https://login.microsoftonline.com/"+tenant_id)  
    context = adal.AuthenticationContext(authority_url)  
    return azure_active_directory.AdalAuthentication(  
    lambda: context.acquire_token_with_client_certificate(  
            resource,  
            application_id,  
            pem_pkey,  
            thumbprint) 
    ) 

# Authenticate to Azure using the Azure Automation RunAs service principal  
runas_connection = automationassets.get_automation_connection("AzureRunAsConnection")  
azure_credential = get_automation_runas_credential(runas_connection)  

# Intialize the resource management client with the RunAs credential and subscription  
resource_client = azure.mgmt.resource.ResourceManagementClient(  
    azure_credential,  
    str(runas_connection["SubscriptionId"]))  

# Get list of resource groups and print them out  
groups = resource_client.resource_groups.list()  
for group in groups:  
    print(group.name) 
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье [Создание runbook для Python](learn/automation-tutorial-runbook-textual-python-3.md).
