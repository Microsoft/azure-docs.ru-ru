---
title: Шифрование данных — Azure CLI для базы данных Azure для PostgreSQL — один сервер
description: Узнайте, как настроить шифрование данных для сервера базы данных Azure для PostgreSQL и управлять им с помощью Azure CLI.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 03/30/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 757782e8842fbcaca9c8d95ec8086dd5791a817b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93240619"
---
# <a name="data-encryption-for-azure-database-for-postgresql-single-server-by-using-the-azure-cli"></a>Шифрование данных для одного сервера базы данных Azure для PostgreSQL с помощью Azure CLI

Узнайте, как использовать Azure CLI для настройки и управления шифрованием данных для одного сервера базы данных Azure для PostgreSQL.

## <a name="prerequisites-for-azure-cli"></a>Необходимые условия для Azure CLI

* Подписка Azure и права администратора для нее.
* Создайте хранилище ключей и ключ, который будет использоваться для ключа, управляемого клиентом. Также включите защиту от очистки и обратимое удаление в хранилище ключей.

   ```azurecli-interactive
   az keyvault create -g <resource_group> -n <vault_name> --enable-soft-delete true --enable-purge-protection true
   ```

* В созданном Azure Key Vault создайте ключ, который будет использоваться для шифрования данных на одиночном сервере базы данных Azure для PostgreSQL.

   ```azurecli-interactive
   az keyvault key create --name <key_name> -p software --vault-name <vault_name>
   ```

* Чтобы использовать существующее хранилище ключей, оно должно иметь следующие свойства для использования в качестве ключа, управляемого клиентом:
  * [Обратимое удаление](../key-vault/general/soft-delete-overview.md)

      ```azurecli-interactive
      az resource update --id $(az keyvault show --name \ <key_vault_name> -o tsv | awk '{print $1}') --set \ properties.enableSoftDelete=true
      ```

  * [Очистить защищенные](../key-vault/general/soft-delete-overview.md#purge-protection)

      ```azurecli-interactive
      az keyvault update --name <key_vault_name> --resource-group <resource_group_name>  --enable-purge-protection true
      ```

* Чтобы использовать ключ в качестве управляемого клиентом, он должен иметь следующие атрибуты:
  * без даты окончания срока действия;
  * не отключено;
  * Выполнение операций **получения**, переноса **и распаковки** 

## <a name="set-the-right-permissions-for-key-operations"></a>Задайте правильные разрешения для операций с ключами.

1. Существует два способа получения управляемого удостоверения для базы данных Azure для PostgreSQL одного сервера.

    ### <a name="create-an-new-azure-database-for-postgresql-server-with-a-managed-identity"></a>Создайте новый сервер базы данных Azure для PostgreSQL с управляемым удостоверением.

    ```azurecli-interactive
    az postgres server create --name <server_name> -g <resource_group> --location <location> --storage-size <size>  -u <user> -p <pwd> --backup-retention <7> --sku-name <sku name> --geo-redundant-backup <Enabled/Disabled> --assign-identity
    ```

    ### <a name="update-an-existing-the-azure-database-for-postgresql-server-to-get-a-managed-identity"></a>Обновите существующий сервер базы данных Azure для PostgreSQL, чтобы получить управляемое удостоверение.

    ```azurecli-interactive
    az postgres server update --resource-group <resource_group> --name <server_name> --assign-identity
    ```

2. Задайте **разрешения ключа** (**Get**, **Wrap**, **Unwrap**) для **участника**, который представляет собой имя сервера PostgreSQL с одним сервером.

    ```azurecli-interactive
    az keyvault set-policy --name -g <resource_group> --key-permissions get unwrapKey wrapKey --object-id <principal id of the server>
    ```

## <a name="set-data-encryption-for-azure-database-for-postgresql-single-server"></a>Настройка шифрования данных для одного сервера базы данных Azure для PostgreSQL

1. Включите шифрование данных для одного сервера базы данных Azure для PostgreSQL с помощью ключа, созданного в Azure Key Vault.

    ```azurecli-interactive
    az postgres server key create --name <server_name> -g <resource_group> --kid <key_url>
    ```

    URL-адрес ключа:  `https://YourVaultName.vault.azure.net/keys/YourKeyName/01234567890123456789012345678901>`

## <a name="using-data-encryption-for-restore-or-replica-servers"></a>Использование шифрования данных для серверов восстановления или реплик

После шифрования отдельного сервера базы данных Azure для PostgreSQL с помощью управляемого ключа клиента, хранящегося в Key Vault, все создаваемые копии сервера также шифруются. Эту новую копию можно создать с помощью локальной или географической операции либо с помощью операции реплики (локальной или кросс-региона). Поэтому для зашифрованного сервера PostgreSQL с одним сервером можно выполнить следующие действия, чтобы создать зашифрованный восстановленный сервер.

### <a name="creating-a-restoredreplica-server"></a>Создание восстановленного или сервера-реплики

* [Создание сервера восстановления](howto-restore-server-cli.md)
* [Создание сервера реплики для чтения](howto-read-replicas-cli.md)

### <a name="once-the-server-is-restored-revalidate-data-encryption-the-restored-server"></a>После восстановления сервера повторно проверьте шифрование данных на восстановленном сервере.

*   Назначение удостоверения для сервера реплики
```azurecli-interactive
az postgres server update --name  <server name>  -g <resoure_group> --assign-identity
```

*   Получение существующего ключа, который должен использоваться для восстановленного или сервера-реплики

```azurecli-interactive
az postgres server key list --name  '<server_name>'  -g '<resource_group_name>'
```

*   Задать политику для нового удостоверения для восстановленного или сервера-реплики

```azurecli-interactive
az keyvault set-policy --name <keyvault> -g <resoure_group> --key-permissions get unwrapKey wrapKey --object-id <principl id of the server returned by the step 1>
```

* Повторная проверка восстановленного или сервера-реплики с помощью ключа шифрования

```azurecli-interactive
az postgres server key create –name  <server name> -g <resource_group> --kid <key url>
```

## <a name="additional-capability-for-the-key-being-used-for-the-azure-database-for-postgresql-single-server"></a>Дополнительные возможности для ключа, используемого для одного сервера базы данных Azure для PostgreSQL

### <a name="get-the-key-used"></a>Получение ключа, используемого

```azurecli-interactive
az postgres server key show --name <server name>  -g <resource_group> --kid <key url>
```

URL-адрес ключа: `https://YourVaultName.vault.azure.net/keys/YourKeyName/01234567890123456789012345678901>`

### <a name="list-the-key-used"></a>Список используемых ключей

```azurecli-interactive
az postgres server key list --name  <server name>  -g <resource_group>
```

### <a name="drop-the-key-being-used"></a>Удаление используемого ключа

```azurecli-interactive
az postgres server key delete -g <resource_group> --kid <key url> 
```

## <a name="using-an-azure-resource-manager-template-to-enable-data-encryption"></a>Использование шаблона Azure Resource Manager для включения шифрования данных

Кроме портал Azure можно включить шифрование данных в базе данных Azure для PostgreSQL на одном сервере с помощью шаблонов Azure Resource Manager для нового и существующего сервера.

### <a name="for-a-new-server"></a>Для нового сервера

Используйте один из предварительно созданных шаблонов Azure Resource Manager для подготовки сервера с включенным шифрованием данных: [пример с шифрованием данных](https://github.com/Azure/azure-postgresql/tree/master/arm-templates/ExampleWithDataEncryption) .

Этот шаблон Azure Resource Manager создает базу данных Azure для одного сервера PostgreSQL и использует **KeyVault** и **ключ** , переданные в качестве параметров для включения шифрования данных на сервере.

### <a name="for-an-existing-server"></a>Для существующего сервера
Кроме того, можно использовать шаблоны Azure Resource Manager, чтобы включить шифрование данных в существующей базе данных Azure для PostgreSQL отдельных серверов.

* Передайте идентификатор ресурса Azure Key Vaultого ключа, скопированный ранее в `Uri` свойстве объекта Properties.

* Используйте *2020-01-01-Preview* в качестве версии API.

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string"
    },
    "serverName": {
      "type": "string"
    },
    "keyVaultName": {
      "type": "string",
      "metadata": {
        "description": "Key vault name where the key to use is stored"
      }
    },
    "keyVaultResourceGroupName": {
      "type": "string",
      "metadata": {
        "description": "Key vault resource group name where it is stored"
      }
    },
    "keyName": {
      "type": "string",
      "metadata": {
        "description": "Key name in the key vault to use as encryption protector"
      }
    },
    "keyVersion": {
      "type": "string",
      "metadata": {
        "description": "Version of the key in the key vault to use as encryption protector"
      }
    }
  },
  "variables": {
    "serverKeyName": "[concat(parameters('keyVaultName'), '_', parameters('keyName'), '_', parameters('keyVersion'))]"
  },
  "resources": [
    {
      "type": "Microsoft.DBforPostgreSQL/servers",
      "apiVersion": "2017-12-01",
      "kind": "",
      "location": "[parameters('location')]",
      "identity": {
        "type": "SystemAssigned"
      },
      "name": "[parameters('serverName')]",
      "properties": {
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-05-01",
      "name": "addAccessPolicy",
      "resourceGroup": "[parameters('keyVaultResourceGroupName')]",
      "dependsOn": [
        "[resourceId('Microsoft.DBforPostgreSQL/servers', parameters('serverName'))]"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "resources": [
            {
              "type": "Microsoft.KeyVault/vaults/accessPolicies",
              "name": "[concat(parameters('keyVaultName'), '/add')]",
              "apiVersion": "2018-02-14-preview",
              "properties": {
                "accessPolicies": [
                  {
                    "tenantId": "[subscription().tenantId]",
                    "objectId": "[reference(resourceId('Microsoft.DBforPostgreSQL/servers/', parameters('serverName')), '2017-12-01', 'Full').identity.principalId]",
                    "permissions": {
                      "keys": [
                        "get",
                        "wrapKey",
                        "unwrapKey"
                      ]
                    }
                  }
                ]
              }
            }
          ]
        }
      }
    },
    {
      "name": "[concat(parameters('serverName'), '/', variables('serverKeyName'))]",
      "type": "Microsoft.DBforPostgreSQL/servers/keys",
      "apiVersion": "2020-01-01-preview",
      "dependsOn": [
        "addAccessPolicy",
        "[resourceId('Microsoft.DBforPostgreSQL/servers', parameters('serverName'))]"
      ],
      "properties": {
        "serverKeyType": "AzureKeyVault",
        "uri": "[concat(reference(resourceId(parameters('keyVaultResourceGroupName'), 'Microsoft.KeyVault/vaults/', parameters('keyVaultName')), '2018-02-14-preview', 'Full').properties.vaultUri, 'keys/', parameters('keyName'), '/', parameters('keyVersion'))]"
      }
    }
  ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

 Дополнительные сведения о шифровании данных см. в статье [Шифрование данных единого сервера в базе данных Azure для PostgreSQL с помощью ключа, управляемого клиентом](concepts-data-encryption-postgresql.md).
