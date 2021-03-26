---
title: Key Vaultный секрет с помощью шаблона
description: Демонстрирует передачу секретного кода из хранилища ключей в виде параметра при развертывании.
ms.topic: conceptual
ms.date: 12/17/2020
ms.openlocfilehash: 05749fe2e9179051c3183ea2e592cf7190ddb347
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104889864"
---
# <a name="use-azure-key-vault-to-pass-secure-parameter-value-during-deployment"></a>Использование Azure Key Vault для передачи защищенного значения параметра во время развертывания

Вместо того, чтобы размещать безопасное значение (например, пароль) непосредственно в шаблоне или файле параметров, можно извлечь значение из [Azure Key Vault](../../key-vault/general/overview.md) во время развертывания. Для получения значения нужно указать в параметре ссылку на хранилище ключей и секрет. Это значение никогда не будет раскрыто, так как указывается только его идентификатор в хранилище ключей. Хранилище ключей может существовать в той же подписке, что и группа ресурсов, в которую выполняется развертывание.

В этой статье рассматривается ситуация передачи конфиденциального значения в качестве параметра шаблона. Не рассматривается ситуация, когда для свойства виртуальной машины задается URL-адрес сертификата в Key Vault. Шаблон быстрого запуска этого сценария см. в [статье Установка сертификата из Azure Key Vault на виртуальной машине](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-winrm-keyvault-windows).

## <a name="deploy-key-vaults-and-secrets"></a>Развертывание хранилищ Key Vault и секретов

Чтобы получить доступ к хранилищу ключей во время развертывания шаблона, задайте для `enabledForTemplateDeployment` хранилища ключей значение `true` .

Если у вас уже есть Key Vault, убедитесь, что он разрешает развертывание шаблонов.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az keyvault update  --name ExampleVault --enabled-for-template-deployment true
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Set-AzKeyVaultAccessPolicy -VaultName ExampleVault -EnabledForTemplateDeployment
```

---

Чтобы создать новый Key Vault и добавить секрет, используйте:

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az group create --name ExampleGroup --location centralus
az keyvault create \
  --name ExampleVault \
  --resource-group ExampleGroup \
  --location centralus \
  --enabled-for-template-deployment true
az keyvault secret set --vault-name ExampleVault --name "ExamplePassword" --value "hVFkk965BuUv"
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name ExampleGroup -Location centralus
New-AzKeyVault `
  -VaultName ExampleVault `
  -resourceGroupName ExampleGroup `
  -Location centralus `
  -EnabledForTemplateDeployment
$secretvalue = ConvertTo-SecureString 'hVFkk965BuUv' -AsPlainText -Force
$secret = Set-AzKeyVaultSecret -VaultName ExampleVault -Name 'ExamplePassword' -SecretValue $secretvalue
```

---

Как владелец хранилища ключей, вы автоматически имеете доступ к созданию секретов. Если пользователь, работающий с секретами, не является владельцем хранилища ключей, предоставьте доступ с помощью:

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az keyvault set-policy \
  --upn <user-principal-name> \
  --name ExampleVault \
  --secret-permissions set delete get list
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$userPrincipalName = "<Email Address of the deployment operator>"

Set-AzKeyVaultAccessPolicy `
  -VaultName ExampleVault `
  -UserPrincipalName <user-principal-name> `
  -PermissionsToSecrets set,delete,get,list
```

---

Дополнительные сведения о создании хранилищ ключей и добавлении секретов см. в следующих статьях:

- [Краткое руководство. Настройка и получение секрета из Azure Key Vault с помощью Azure CLI](../../key-vault/secrets/quick-create-cli.md)
- [Установка и получение секрета с помощью PowerShell](../../key-vault/secrets/quick-create-powershell.md)
- [Краткое руководство. Настройка и получение секрета из Azure Key Vault с помощью портала Azure](../../key-vault/secrets/quick-create-portal.md)
- [Краткое руководство. Настройка и получение секрета из Azure Key Vault с помощью веб-приложения .NET](../../key-vault/secrets/quick-create-net.md)
- [Краткое руководство. Настройка и получение секрета из Azure Key Vault с помощью веб-приложения Node](../../key-vault/secrets/quick-create-node.md)

## <a name="grant-access-to-the-secrets"></a>Предоставление доступа к секретам

Пользователь, который развертывает шаблон, должен иметь разрешение на `Microsoft.KeyVault/vaults/deploy/action` доступ к области группы ресурсов и хранилища ключей. Оно имеется у ролей [Владелец](../../role-based-access-control/built-in-roles.md#owner) и [Участник](../../role-based-access-control/built-in-roles.md#contributor). Если вы создали хранилище ключей, вы являетесь владельцем и имеете разрешение.

Ниже показано, как создать роль с минимальным разрешением и назначить пользователя

1. Создание JSON-файла определения пользовательской роли

    ```json
    {
      "Name": "Key Vault resource manager template deployment operator",
      "IsCustom": true,
      "Description": "Lets you deploy a resource manager template with the access to the secrets in the Key Vault.",
      "Actions": [
        "Microsoft.KeyVault/vaults/deploy/action"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ]
    }
    ```
    Замените "00000000-0000-0000-0000-000000000000" на идентификатор подписки.

2. Создание новой роли с помощью JSON-файла:

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

    ```azurecli-interactive
    az role definition create --role-definition "<path-to-role-file>"
    az role assignment create \
      --role "Key Vault resource manager template deployment operator" \
      --assignee <user-principal-name> \
      --resource-group ExampleGroup
    ```

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

    ```azurepowershell-interactive
    New-AzRoleDefinition -InputFile "<path-to-role-file>"
    New-AzRoleAssignment `
      -ResourceGroupName ExampleGroup `
      -RoleDefinitionName "Key Vault resource manager template deployment operator" `
      -SignInName <user-principal-name>
    ```

    ---

    В примерах пользовательская роль назначается пользователю на уровне группы ресурсов.

При использовании хранилища Key Vault с шаблоном для [управляемого приложения](../managed-applications/overview.md) необходимо предоставить доступ к субъекту-службе **Поставщик ресурсов устройств**. Дополнительные сведения см. в статье [Доступ к секрету Key Vault при развертывании Управляемых приложений Azure](../managed-applications/key-vault-access.md).

## <a name="reference-secrets-with-static-id"></a>Секреты ссылки со статическим идентификатором

Этот способ дает возможность ссылаться на хранилище ключей в файле параметров, а не в шаблоне. На следующем рисунке показано, как файл параметров ссылается на секрет и передает это значение в шаблон.

![Диаграмма статического идентификатора интеграции хранилища ключей Resource Manager](./media/key-vault-parameter/statickeyvault.png)

[Учебник. интеграция Azure Key Vault в диспетчер ресурсов шаблоны развертывания](./template-tutorial-use-key-vault.md) использует этот метод.

Следующий шаблон развертывает сервер SQL Server, который содержит пароль администратора. В качестве параметра пароля задается защищенная строка. Но шаблон не указывает, откуда берется это значение.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "adminLogin": {
      "type": "string"
    },
    "adminPassword": {
      "type": "securestring"
    },
    "sqlServerName": {
      "type": "string"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Sql/servers",
      "apiVersion": "2015-05-01-preview",
      "name": "[parameters('sqlServerName')]",
      "location": "[resourceGroup().location]",
      "tags": {},
      "properties": {
        "administratorLogin": "[parameters('adminLogin')]",
        "administratorLoginPassword": "[parameters('adminPassword')]",
        "version": "12.0"
      }
    }
  ],
  "outputs": {
  }
}
```

Теперь создайте файл параметров для предыдущего шаблона. В файле параметров укажите параметр, который совпадает с именем параметра в шаблоне. Для значения параметра используйте ссылку на секрет из хранилища ключей. Для ссылки на секретный код необходимо передать идентификатор ресурса хранилища ключей и имя секрета.

В приведенном ниже файле параметров секрет хранилища ключей уже должен существовать. При этом для идентификатора ресурса используется статическое значение.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "adminLogin": {
        "value": "exampleadmin"
      },
      "adminPassword": {
        "reference": {
          "keyVault": {
          "id": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"
          },
          "secretName": "ExamplePassword"
        }
      },
      "sqlServerName": {
        "value": "<your-server-name>"
      }
  }
}
```

Если необходимо использовать версию секрета, отличную от текущей, примените свойство `secretVersion`.

```json
"secretName": "ExamplePassword",
"secretVersion": "cd91b2b7e10e492ebb870a6ee0591b68"
```

Разверните шаблон и передайте файл параметров.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az group create --name SqlGroup --location westus2
az deployment group create \
  --resource-group SqlGroup \
  --template-uri <template-file-URI> \
  --parameters <parameter-file>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment `
  -ResourceGroupName $resourceGroupName `
  -TemplateUri <template-file-URI> `
  -TemplateParameterFile <parameter-file>
```

---

## <a name="reference-secrets-with-dynamic-id"></a>Ссылка на секреты с динамическим идентификатором

Выше мы рассмотрели способ передачи статического идентификатора ресурса для секрета хранилища ключей из параметра. Однако в некоторых сценариях вам нужно использовать ссылку на секретный код хранилища ключей, который изменяется в зависимости от текущего развертывания. Или можете просто передать значения параметра в шаблон вместо создания ссылочного параметра в файле параметров. В любом случае можно динамически создать идентификатор ресурса для секрета хранилища ключей с помощью связанного шаблона.

Динамически создавать идентификатор ресурса в файле параметров нельзя, так как в файле параметров не разрешены выражения шаблонов.

В родительском шаблоне Добавьте вложенный шаблон и передайте параметр, содержащий динамически созданный идентификатор ресурса. На следующем рисунке показано, как параметр в связанном шаблоне ссылается на секрет.

![Динамический идентификатор](./media/key-vault-parameter/dynamickeyvault.png)

Следующий шаблон динамически создает идентификатор хранилища ключей и передает его в качестве параметра.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]",
        "metadata": {
          "description": "The location where the resources will be deployed."
        }
      },
      "vaultName": {
        "type": "string",
        "metadata": {
          "description": "The name of the keyvault that contains the secret."
        }
      },
      "secretName": {
        "type": "string",
        "metadata": {
          "description": "The name of the secret."
        }
      },
      "vaultResourceGroupName": {
        "type": "string",
        "metadata": {
          "description": "The name of the resource group that contains the keyvault."
        }
      },
      "vaultSubscription": {
        "type": "string",
        "defaultValue": "[subscription().subscriptionId]",
        "metadata": {
          "description": "The name of the subscription that contains the keyvault."
        }
      }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-10-01",
      "name": "dynamicSecret",
      "properties": {
        "mode": "Incremental",
        "expressionEvaluationOptions": {
          "scope": "inner"
        },
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "adminLogin": {
              "type": "string"
            },
            "adminPassword": {
              "type": "securestring"
            },
            "location": {
              "type": "string"
            }
          },
          "variables": {
            "sqlServerName": "[concat('sql-', uniqueString(resourceGroup().id, 'sql'))]"
          },
          "resources": [
            {
              "type": "Microsoft.Sql/servers",
              "apiVersion": "2018-06-01-preview",
              "name": "[variables('sqlServerName')]",
              "location": "[parameters('location')]",
              "properties": {
                "administratorLogin": "[parameters('adminLogin')]",
                "administratorLoginPassword": "[parameters('adminPassword')]"
              }
            }
          ],
          "outputs": {
            "sqlFQDN": {
              "type": "string",
              "value": "[reference(variables('sqlServerName')).fullyQualifiedDomainName]"
            }
          }
        },
        "parameters": {
          "location": {
            "value": "[parameters('location')]"
          },
          "adminLogin": {
            "value": "ghuser"
          },
          "adminPassword": {
            "reference": {
              "keyVault": {
                "id": "[resourceId(parameters('vaultSubscription'), parameters('vaultResourceGroupName'), 'Microsoft.KeyVault/vaults', parameters('vaultName'))]"
              },
              "secretName": "[parameters('secretName')]"
            }
          }
        }
      }
    }
  ],
  "outputs": {
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о хранилищах ключей см. в статье [Что такое хранилище ключей Azure?](../../key-vault/general/overview.md)
- Полные примеры использования ссылок на секреты ключей приведены [здесь](https://github.com/rjmax/ArmExamples/tree/master/keyvaultexamples).
- Модуль Microsoft Learn, который охватывает передачу защищенного значения из хранилища ключей, см. в статье [управление комплексными облачными развертываниями с помощью расширенных функций шаблонов ARM](/learn/modules/manage-deployments-advanced-arm-template-features/).
