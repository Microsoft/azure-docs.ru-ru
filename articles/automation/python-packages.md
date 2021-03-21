---
title: Управление пакетами Python 2 в службе автоматизации Azure
description: В этой статье описывается управление пакетами Python 2 в службе автоматизации Azure.
services: automation
ms.subservice: process-automation
ms.date: 12/17/2020
ms.topic: conceptual
ms.custom: devx-track-python
ms.openlocfilehash: fd830afd5628591019902ca583f9cbc8e2a7ecad
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97683400"
---
# <a name="manage-python-2-packages-in-azure-automation"></a>Управление пакетами Python 2 в службе автоматизации Azure

Служба автоматизации Azure позволяет выполнять модули runbook Python 2 в Azure и в гибридных рабочих ролях Runbook Linux. Чтобы упростить модули runbook, можно использовать пакеты Python для импорта необходимых модулей. В этой статье описывается управление пакетами Python и их использование в службе автоматизации Azure.

## <a name="import-packages"></a>Импорт пакетов

В учетной записи службы автоматизации в области **Общие ресурсы** выберите **Пакеты Python 2**. Щелкните **+ Добавить пакет Python 2**.

:::image type="content" source="media/python-packages/add-python-package.png" alt-text="Снимок экрана: страница &quot;пакеты Python 2&quot; отображает пакеты Python 2 в меню слева и добавляет выделенный пакет Python 2.":::

На странице "Добавление пакета Python 2" выберите локальный пакет для передачи. Пакет может иметь расширение файла **.whl** или **.tar.gz**. Выбрав пакет, щелкните **ОК**, чтобы передать его.

:::image type="content" source="media/python-packages/upload-package.png" alt-text="На снимке экрана показана страница Добавление пакета Python 2 с загруженным файлом tar. gz.":::

После импорта пакета он появится в списке на странице пакетов Python 2 в учетной записи службы автоматизации. Если вам нужно удалить пакет, выберите его и нажмите кнопку **Удалить**.

:::image type="content" source="media/python-packages/package-list.png" alt-text="На снимке экрана показана страница пакетов Python 2 после импорта пакета.":::

## <a name="import-packages-with-dependencies"></a>Импорт пакетов с зависимостями

Служба автоматизации Azure не разрешает зависимости для пакетов Python в процессе импорта. У вас есть два варианта, позволяющих импортировать пакет со всеми зависимостями. Для импорта пакетов в учетную запись службы автоматизации выполните только один из следующих шагов.

### <a name="manually-download"></a>Скачивание вручную

На компьютере под управлением 64-разрядной версии Windows и с установленными [Python 2.7](https://www.python.org/downloads/release/latest/python2) и [PIP](https://pip.pypa.io/en/stable/) выполните следующую команду, чтобы скачать пакет и все его зависимости:

```cmd
C:\Python27\Scripts\pip2.7.exe download -d <output dir> <package name>
```

Когда скачивание пакетов завершится, импортируйте их в учетную запись службы автоматизации.

### <a name="runbook"></a>Модуль Runbook

 Чтобы получить модуль Runbook, [импортируйте пакеты Python 2 из PyPI в учетную запись службы автоматизации Azure](https://github.com/azureautomation/import-python-2-packages-from-pypi-into-azure-automation-account) из Организации Azure Automation GitHub в учетную запись службы автоматизации. Убедитесь, что в параметрах запуска задано значение **Azure** и запустите runbook с нужными параметрами. Для работы runbook требуется учетная запись запуска от имени, чтобы использовать учетную запись службы автоматизации. Следите за тем, чтобы в начале каждого параметра был правильный символ, как показано в следующем списке и на рисунке ниже.

* -s \<subscriptionId\>
* -g \<resourceGroup\>
* -a \<automationAccount\>
* -m \<modulePackage\>

:::image type="content" source="media/python-packages/import-python-runbook.png" alt-text="На снимке экрана показана страница обзора для import_py2package_from_pypi с панелью &quot;запустить Runbook&quot; в правой части.":::

Runbook позволяет указать нужный пакет для скачивания. Например, если задать параметр `Azure`, будут скачаны все модули Azure и все зависимости (около 105).

Когда последовательность runbook будет выполнена, проверьте список **Пакеты Python 2** в разделе **Общие ресурсы** для учетной записи службы автоматизации и убедитесь, что пакет был импортирован правильно.

## <a name="use-a-package-in-a-runbook"></a>Использование пакета в модуле runbook

Завершив импорт пакета, вы сможете использовать его в runbook. В приведенном ниже примере используется [служебный пакет службы автоматизации Azure](https://github.com/azureautomation/azure_automation_utility). Он упрощает использование Python со службой автоматизации Azure. Чтобы использовать этот пакет, следуйте инструкциям в репозитории GitHub и добавьте его в последовательность runbook. Например, с помощью `from azure_automation_utility import get_automation_runas_credential` можно импортировать функцию для извлечения учетной записи запуска от имени.

```python
import azure.mgmt.resource
import automationassets
from azure_automation_utility import get_automation_runas_credential

# Authenticate to Azure using the Azure Automation RunAs service principal
runas_connection = automationassets.get_automation_connection("AzureRunAsConnection")
azure_credential = get_automation_runas_credential()

# Intialize the resource management client with the RunAs credential and subscription
resource_client = azure.mgmt.resource.ResourceManagementClient(
    azure_credential,
    str(runas_connection["SubscriptionId"]))

# Get list of resource groups and print them out
groups = resource_client.resource_groups.list()
for group in groups:
    print group.name
```

## <a name="develop-and-test-runbooks-offline"></a>Разработка и тестирование модулей runbook в автономном режиме

Для автономной разработки и тестирования runbook на основе Python 2 можно использовать модуль [эмулируемых ресурсов Python для службы автоматизации Azure](https://github.com/azureautomation/python_emulated_assets), размещенный в репозитории GitHub. Он позволяет ссылаться на общие ресурсы, такие как учетные данные, переменные, подключения и сертификаты.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье [Создание runbook для Python](learn/automation-tutorial-runbook-textual-python2.md).
