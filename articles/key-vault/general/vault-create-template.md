---
title: Создание хранилища ключей Azure и политики доступа к хранилищу с помощью шаблона ARM
description: В этой статье показано, как создать хранилища ключей Azure и политики доступа к хранилищу с помощью шаблона Azure Resource Manager.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 3/14/2021
ms.author: mbaldwin
ms.openlocfilehash: e489efd335d2a46a0e55d4ceb9bdee809ae6c39d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104595249"
---
# <a name="how-to-create-an-azure-key-vault-and-vault-access-policy-by-using-a-resource-manager-template"></a>Как создать хранилище ключей Azure и политику доступа к хранилищу с помощью шаблона диспетчер ресурсов

[Azure Key Vault](../general/overview.md) — это облачная служба, которая обеспечивает безопасное хранение секретов, таких как ключи, пароли и сертификаты. В этой статье описывается процесс развертывания шаблона Azure Resource Manager (шаблон ARM) для создания хранилища ключей.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить действия, описанные в этой статье, сделайте следующее:

* Если у вас нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) перед началом работы.


## <a name="create-a-key-vault-resource-manager-template"></a>Создание шаблона диспетчер ресурсов Key Vault

В следующем шаблоне показан базовый способ создания хранилища ключей. Некоторые значения указываются в шаблоне.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "keyVaultName": {
      "type": "string",
      "metadata": {
        "description": "Specifies the name of the key vault."
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
        "description": "Specifies whether the key vault is a standard vault or a premium vault."
      }
    }
   },
  "resources": [
    {
      "type": "Microsoft.KeyVault/vaults",
      "apiVersion": "2019-09-01",
      "name": "[parameters('keyVaultName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "enabledForDeployment": "false",
        "enabledForDiskEncryption": "false",
        "enabledForTemplateDeployment": "false",
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
    }
  ]
}

```

Дополнительные сведения о параметрах шаблонов Key Vault см. в разделе [Справочник по шаблонам ARM Key Vault](/azure/templates/microsoft.keyvault/vaults).

> [!IMPORTANT]
> При повторном развертывании шаблона все существующие политики доступа в хранилище ключей будут переопределены. Рекомендуется заполнять `accessPolicies` свойство существующими политиками доступа, чтобы избежать потери доступа к хранилищу ключей. 

## <a name="add-an-access-policy-to-a-key-vault-resource-manager-template"></a>Добавление политики доступа в шаблон диспетчер ресурсов Key Vault

Политики доступа можно развернуть в существующем хранилище ключей без повторного развертывания всего шаблона хранилища ключей. В следующем шаблоне показан базовый способ создания политик доступа.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "keyVaultName": {
      "type": "string",
      "metadata": {
        "description": "Specifies the name of the key vault."
      }
    },
    "objectId": {
      "type": "string",
      "metadata": {
        "description": "Specifies the object ID of a user, service principal or security group in the Azure Active Directory tenant for the vault. The object ID must be unique for the list of access policies. Get it by using Get-AzADUser or Get-AzADServicePrincipal cmdlets."
      }
    },
    "keysPermissions": {
      "type": "array",
      "defaultValue": [
        "list"
      ],
      "metadata": {
        "description": "Specifies the permissions to keys in the vault. Valid values are: all, encrypt, decrypt, wrapKey, unwrapKey, sign, verify, get, list, create, update, import, delete, backup, restore, recover, and purge."
      }
    },
    "secretsPermissions": {
      "type": "array",
      "defaultValue": [
        "list"
      ],
      "metadata": {
        "description": "Specifies the permissions to secrets in the vault. Valid values are: all, get, list, set, delete, backup, restore, recover, and purge."
      }
    },
    "certificatePermissions": {
      "type": "array",
      "defaultValue": [
        "list"
      ],
      "metadata": {
        "description": "Specifies the permissions to certificates in the vault. Valid values are: all,  create, delete, update, deleteissuers, get, getissuers, import, list, listissuers, managecontacts, manageissuers,  recover, backup, restore, setissuers, and purge."
      }
    },
  "resources": [
     {
      "type": "Microsoft.KeyVault/vaults/accessPolicies",
      "name": "[concat(parameters('keyVaultName'), '/add')]",
      "apiVersion": "2019-09-01",
      "properties": {
        "accessPolicies": [
          {
            "tenantId": "[subscription().tenantId]",
            "objectId": "[parameters('objectId')]",
            "permissions": {
              "keys": "[parameters('keysPermissions')]",
              "secrets": "[parameters('secretsPermissions')]",
              "certificates": "[parameters('certificatePermissions')]"
            }
          }
        ]
      }
    }
  ]
}

```

Дополнительные сведения о параметрах шаблонов Key Vault см. в разделе [Справочник по шаблонам ARM Key Vault](/azure/templates/microsoft.keyvault/vaults/accesspolicies).

## <a name="more-key-vault-resource-manager-templates"></a>Дополнительные Key Vault диспетчер ресурсов шаблонов

Существуют другие шаблоны диспетчер ресурсов, доступные для Key Vault объектов.

| Секреты | Ключи | Сертификаты |
|--|--|--|
|<ul><li>[Краткое руководство](../secrets/quick-create-template.md)<li>[Ссылки](/azure/templates/microsoft.keyvault/vaults/secrets)|Н/Д|Н/Д|

Дополнительные Key Vault шаблоны можно найти здесь: [Key Vault диспетчер ресурсов Reference](/azure/templates/microsoft.keyvault/allversions).

## <a name="deploy-the-templates"></a>Развертывание шаблонов.

Вы можете использовать портал Azure для развертывания описанных выше шаблонов с помощью параметра **создать собственный шаблон в редакторе** , как описано здесь: [развертывание ресурсов из пользовательского шаблона](../../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template).

Вы также можете сохранить приведенные выше шаблоны в файлы и использовать следующие команды:  [New-азресаурцеграупдеплоймент](/powershell/module/az.resources/new-azresourcegroupdeployment) и [AZ Deployment Group Create](/cli/azure/deployment/group#az_deployment_group_create):

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateFile key-vault-template.json
```

```azurecli
az deployment group create --resource-group ExampleGroup --template-file key-vault-template.json
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете продолжить работу с последующими краткими руководствами и учебниками, эти ресурсы можно оставить на месте. Если ресурсы больше не нужны, удалите группу ресурсов. При удалении группы хранилище ключей и связанные с ним ресурсы также будут удалены. Чтобы удалить группу ресурсов с помощью Azure CLI или Azure PowerShell, выполните следующие действия.

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

## <a name="resources"></a>Ресурсы

- Ознакомьтесь с [обзором Azure Key Vault](../general/overview.md).
- Дополнительные сведения о [Azure Resource Manager](../../azure-resource-manager/management/overview.md).
- Статья [Обзор системы безопасности Azure Key Vault](security-overview.md)

## <a name="next-steps"></a>Дальнейшие действия

- [Безопасный доступ к хранилищу ключей](secure-your-key-vault.md)
- [Аутентификация в хранилище ключей](authentication.md)
- [Руководство разработчика хранилища ключей Azure](developers-guide.md)
