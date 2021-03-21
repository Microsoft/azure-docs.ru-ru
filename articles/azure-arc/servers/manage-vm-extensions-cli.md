---
title: Включение расширения виртуальной машины с помощью Azure CLI
description: В этой статье описывается, как развернуть расширения виртуальной машины на серверах с поддержкой дуги Azure, работающих в гибридных облачных средах, с помощью Azure CLI.
ms.date: 01/05/2021
ms.topic: conceptual
ms.custom: devx-track-azurecli
ms.openlocfilehash: 6edb7d55e542f963c75693d535fa3b50dc5b827b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97916207"
---
# <a name="enable-azure-vm-extensions-using-the-azure-cli"></a>Включение расширений виртуальной машины Azure с помощью Azure CLI

В этой статье показано, как развертывать и удалять расширения виртуальных машин, поддерживаемые серверами с поддержкой ARC в Azure, на гибридный компьютер под управлением Linux или Windows с помощью Azure CLI.

[!INCLUDE [Azure CLI Prepare your environment](../../../includes/azure-cli-prepare-your-environment.md)]

## <a name="install-the-azure-cli-extension"></a>Установка расширения Azure CLI

Команды Коннектедмачине не поставляются в составе Azure CLI. Прежде чем использовать Azure CLI для управления расширениями виртуальной машины на гибридном сервере, управляемом серверами с поддержкой Arc, необходимо загрузить расширение Коннектедмачине. Выполните следующую команду, чтобы получить его:

```azurecli
az extension add --name connectedmachine
```

## <a name="enable-extension"></a>Включить расширение

Чтобы включить расширение виртуальной машины на сервере с поддержкой Arc, используйте команду [AZ коннектедмачине Extension Create](/cli/azure/ext/connectedmachine/connectedmachine/extension#ext_connectedmachine_az_connectedmachine_extension_create) с `--machine-name` параметрами,, `--extension-name` ,, `--location` `--type` `settings` и `--publisher` .

В следующем примере показано включение расширения виртуальной машины Log Analytics на сервере с поддержкой ARC:

```azurecli
az connectedmachine extension create --machine-name "myMachineName" --name "OmsAgentForLinux or MicrosoftMonitoringAgent" --location "eastus" --settings '{\"workspaceId\":\"myWorkspaceId\"}' --protected-settings '{\"workspaceKey\":\"myWorkspaceKey\"}' --resource-group "myResourceGroup" --type-handler-version "1.13" --type "OmsAgentForLinux or MicrosoftMonitoringAgent" --publisher "Microsoft.EnterpriseCloud.Monitoring" 
```

В следующем примере активируется расширение пользовательского скрипта на сервере с поддержкой ARC:

```azurecli
az connectedmachine extension create --machine-name "myMachineName" --name "CustomScriptExtension" --location "eastus" --type "CustomScriptExtension" --publisher "Microsoft.Compute" --settings "{\"commandToExecute\":\"powershell.exe -c \\\"Get-Process | Where-Object { $_.CPU -gt 10000 }\\\"\"}" --type-handler-version "1.10" --resource-group "myResourceGroup"
```

В следующем примере показано включение расширения виртуальной машины Key Vault (Предварительная версия) на сервере с поддержкой ARC:

```azurecli
az connectedmachine extension create --resource-group "resourceGroupName" --machine-name "myMachineName" --location "regionName" --publisher "Microsoft.Azure.KeyVault" --type "KeyVaultForLinux or KeyVaultForWindows" --name "KeyVaultForLinux or KeyVaultForWindows" --settings '{"secretsManagementSettings": { "pollingIntervalInS": "60", "observedCertificates": ["observedCert1"] }, "authenticationSettings": { "msiEndpoint": "http://localhost:40342/metadata/identity" }}'
```

## <a name="list-extensions-installed"></a>Список установленных расширений

Чтобы получить список расширений виртуальной машины на сервере с поддержкой Arc, используйте команду [AZ коннектедмачине Extension List](/cli/azure/ext/connectedmachine/connectedmachine/extension#ext_connectedmachine_az_connectedmachine_extension_list) с `--machine-name` `--resource-group` параметрами и.

Пример

```azurecli
az connectedmachine extension list --machine-name "myMachineName" --resource-group "myResourceGroup"
```

По умолчанию выходные данные команд Azure CLI в формате JSON (JavaScript Object Notation). Например, чтобы изменить назначение выходных данных на таблицу или список, используйте команду [az configure --output](/cli/azure/reference-index). Параметр `--output` можно добавлять к любой команде — он один раз изменяет формат выходных данных.

В следующем примере показаны частичные выходные данные JSON из `az connectedmachine extension -list` команды:

```json
[
  {
    "autoUpgradingMinorVersion": "false",
    "forceUpdateTag": null,
    "id": "/subscriptions/subscriptionId/resourceGroups/resourceGroupName/providers/Microsoft.HybridCompute/machines/SVR01/extensions/DependencyAgentWindows",
    "location": "eastus",
    "name": "DependencyAgentWindows",
    "namePropertiesInstanceViewName": "DependencyAgentWindows",
```

## <a name="remove-an-installed-extension"></a>Удаление установленного расширения

Чтобы удалить установленное расширение виртуальной машины на сервере с поддержкой Arc, используйте команду [AZ коннектедмачине Extension Delete](/cli/azure/ext/connectedmachine/connectedmachine/extension#ext_connectedmachine_az_connectedmachine_extension_delete) с `--extension-name` `--machine-name` параметрами, и `--resource-group` .

Например, чтобы удалить расширение виртуальной машины Log Analytics для Linux, выполните следующую команду:

```azurecli
az connectedmachine extension delete --machine-name "myMachineName" --name "OmsAgentForLinux" --resource-group "myResourceGroup"
```

## <a name="next-steps"></a>Дальнейшие действия

- Расширения виртуальных машин можно развертывать, администрировать и удалять с помощью [Azure PowerShell](manage-vm-extensions-powershell.md), из [портал Azure](manage-vm-extensions-portal.md)или [шаблонов Azure Resource Manager](manage-vm-extensions-template.md).

- Сведения об устранении неполадок можно найти в разделе [руководство по устранению неполадок РАСШИРЕНИЙ ВМ](troubleshoot-vm-extensions.md).

- Дополнительные сведения о командах см. в статье [Обзор](/cli/azure/ext/connectedmachine/connectedmachine/extension) расширения виртуальной машины Azure CLI.
