---
title: Краткое руководство Azure. Создание хранилища ключей Azure и ключа с помощью шаблона Azure Resource Manager | Документация Майкрософт
description: Краткое руководство, в котором показано, как создавать хранилища ключей Azure и добавлять ключи в хранилища с помощью шаблона Azure Resource Manager (шаблона ARM).
services: key-vault
author: sebansal
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.custom: mvc,subject-armqs
ms.date: 10/14/2020
ms.author: sebansal
ms.openlocfilehash: 48686fb78dc4295da3bd390d92ae3837cd09a4d0
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105639430"
---
# <a name="quickstart-create-an-azure-key-vault-and-a-key-by-using-arm-template"></a>Краткое руководство. Создание хранилища ключей Azure и ключа с помощью шаблона ARM 

[Azure Key Vault](../general/overview.md) — это облачная служба, которая обеспечивает безопасное хранение секретов, таких как ключи, пароли, сертификаты, и другой секретной информации. В этом кратком руководстве показано, как развернуть шаблон Azure Resource Manager (шаблон ARM) для создания хранилища ключей и ключа.

> [!NOTE]
> Эта функция недоступна для Azure для государственных организаций.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этой статьей необходимо сделать следующее:

- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
- Пользователю должна быть назначена встроенная роль Azure, например роль участника. [Дополнительные сведения см. здесь](../../role-based-access-control/role-assignments-portal.md)
- Шаблону для настройки разрешений требуется идентификатор объекта пользователя Azure AD. Следующая процедура возвращает идентификатор объекта (GUID).

    1. Выполните следующую команду Azure PowerShell или Azure CLI, выбрав **Попробовать**, а затем вставьте сценарий в панель оболочки. Чтобы вставить сценарий, щелкните правой кнопкой мыши оболочку и выберите **Вставить**.

        # <a name="cli"></a>[CLI](#tab/CLI)
        ```azurecli-interactive
        echo "Enter your email address that is used to sign in to Azure:" &&
        read upn &&
        az ad user show --id $upn --query "objectId" &&
        echo "Press [ENTER] to continue ..."
        ```

        # <a name="powershell"></a>[PowerShell](#tab/PowerShell)
        ```azurepowershell-interactive
        $upn = Read-Host -Prompt "Enter your email address used to sign in to Azure"
        (Get-AzADUser -UserPrincipalName $upn).Id
        Write-Host "Press [ENTER] to continue..."
        ```

        ---

    1. Запишите идентификатор объекта. Он понадобится в следующем разделе этого краткого руководства.

## <a name="review-the-template"></a>Изучение шаблона

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vaultName": {
      "type": "string",
      "metadata": {
        "description": "The name of the key vault to be created."
      }
    },
    "skuName": {
      "type": "string",
      "defaultValue": "Standard",
      "allowedValues": [
        "Standard",
        "Premium"
      ],
      "metadata": {
        "description": "The SKU of the vault to be created."
      }
    },
    "keyName": {
      "type": "string",
      "metadata": {
        "description": "The name of the key to be created."
      }
    },
    "keyType": {
      "type": "string",
      "metadata": {
        "description": "The JsonWebKeyType of the key to be created."
      }
    },
    "keyOps": {
      "type": "array",
      "defaultValue": [],
      "metadata": {
        "description": "The permitted JSON web key operations of the key to be created."
      }
    },
    "keySize": {
      "type": "int",
      "defaultValue": 2048,
      "metadata": {
        "description": "The size in bits of the key to be created."
      }
    },
    "curveName": {
      "type": "string",
      "defaultValue": "",
      "metadata": {
        "description": "The JsonWebKeyCurveName of the key to be created."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.KeyVault/vaults",
      "apiVersion": "2019-09-01",
      "name": "[parameters('vaultName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "enableRbacAuthorization": false,
        "enableSoftDelete": false,
        "enabledForDeployment": false,
        "enabledForDiskEncryption": false,
        "enabledForTemplateDeployment": false,
        "tenantId": "[subscription().tenantId]",
        "accessPolicies": [],
        "sku": {
          "name": "[parameters('skuName')]",
          "family": "A"
        },
        "networkAcls": {
          "defaultAction": "Allow",
          "bypass": "AzureServices"
        }
      }
    },
    {
      "type": "Microsoft.KeyVault/vaults/keys",
      "apiVersion": "2019-09-01",
      "name": "[concat(parameters('vaultName'), '/', parameters('keyName'))]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.KeyVault/vaults', parameters('vaultName'))]"
      ],
      "properties": {
        "kty": "[parameters('keyType')]",
        "keyOps": "[parameters('keyOps')]",
        "keySize": "[parameters('keySize')]",
        "curveName": "[parameters('curveName')]"
      }
    }
  ],
  "outputs": {
    "proxyKey": {
      "type": "object",
      "value": "[reference(resourceId('Microsoft.KeyVault/vaults/keys', parameters('vaultName'), parameters('keyName')))]"
    }
  }
}
```

В шаблоне определены два ресурса:

- [Microsoft.KeyVault/vaults](/azure/templates/microsoft.keyvault/vaults)
- Microsoft.KeyVault/vaults/keys

Дополнительные примеры шаблонов Azure Key Vault можно найти в [шаблонах быстрого запуска Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Keyvault&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Развертывание шаблона
Можно использовать [порта Azure](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-portal), Azure PowerShell, Azure CLI или REST API. Дополнительные сведения о методах развертывания см. в статье о [Развертывание ресурсов с помощью шаблонов ARM и Azure PowerShell](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-powershell).

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Можно проверить хранилище ключей и ключ с помощью портала Azure или использовать следующий скрипт Azure CLI или Azure PowerShell, чтобы просмотреть созданный ключ.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter your key vault name:" &&
read keyVaultName &&
az keyvault key list --vault-name $keyVaultName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$keyVaultName = Read-Host -Prompt "Enter your key vault name"
Get-AzKeyVaultKey -vaultName $keyVaultName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Другие руководства о Key Vault созданы на основе этого документа. Если вы планируете продолжить работу с последующими краткими руководствами и статьями, эти ресурсы можно не удалять.
Удалите ненужную группу ресурсов. Key Vault и связанные ресурсы будут также удалены. Чтобы удалить группу ресурсов с помощью Azure CLI или Azure PowerShell, выполните следующие действия.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как создать хранилище ключей и ключ с помощью шаблона ARM и проверить развертывание. Дополнительные сведения о Key Vault и Azure Resource Manager см. в следующих статьях.

- [Обзор Azure Key Vault](../general/overview.md)
- Сведения об [Azure Resource Manager](../../azure-resource-manager/management/overview.md)
- Статья [Обзор системы безопасности Key Vault](../general/security-overview.md)
