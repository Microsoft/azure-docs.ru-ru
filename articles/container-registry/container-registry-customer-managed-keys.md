---
title: Шифрование реестра с помощью управляемого клиентом ключа
description: Узнайте о шифровании неактивных данных реестра контейнеров Azure и о шифровании реестра уровня "Премиум" с помощью управляемого клиентом ключа, хранящегося в Azure Key Vault
ms.topic: article
ms.date: 03/03/2021
ms.custom: ''
ms.openlocfilehash: aad9419fdb139ff615bfe07075be78a2ca4ee4ac
ms.sourcegitcommit: 8d1b97c3777684bd98f2cfbc9d440b1299a02e8f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102489078"
---
# <a name="encrypt-registry-using-a-customer-managed-key"></a>Шифрование реестра с помощью управляемого клиентом ключа

При хранении образов и других артефактов в реестре контейнеров Azure содержимое реестра автоматически шифруется, используя [ключи, управляемые службой](../security/fundamentals/encryption-models.md). Шифрование по умолчанию можно расширить с помощью дополнительного уровня шифрования, используя ключ, который вы создаете и управляете в Azure Key Vault (управляемый клиентом ключ). В этой статье описано, как использовать Azure CLI, портал Azure или шаблон диспетчер ресурсов.

Шифрование на стороне сервера с помощью управляемых клиентом ключей поддерживается с помощью интеграции с [Azure Key Vault](../key-vault/general/overview.md): 

* Вы можете создавать собственные ключи шифрования и хранить их в хранилище ключей либо использовать API-интерфейсы Azure Key Vault для генерации этих ключей. 
* С помощью Azure Key Vault можно также выполнять аудит использования ключа.
* Реестр контейнеров Azure поддерживает автоматический поворот ключей шифрования реестра при наличии новой версии ключа в Azure Key Vault. Также можно вручную повернуть ключи шифрования реестра.

Эта возможность доступна на уровне **Премиум** службы реестра контейнеров. Ознакомиться с информацией об уровнях служб реестра и ограничениях можно в статье [Уровни служб Реестра контейнеров Azure](container-registry-skus.md).


## <a name="things-to-know"></a>Полезная информация

* В настоящее время ключ, управляемый клиентом, можно включить только при создании реестра. При включении ключа вы настраиваете управляемое удостоверение, *назначаемое пользователем* , для доступа к хранилищу ключей.
* После включения шифрования с помощью ключа, управляемого клиентом, в реестр нельзя отключить шифрование.  
* Реестр контейнеров Azure поддерживает только ключи RSA или RSA-HSM. Ключи эллиптической кривой в настоящее время не поддерживаются.
* [Доверенное содержимое](container-registry-content-trust.md) в настоящее время не поддерживается в реестре, зашифрованном с помощью управляемого клиентом ключа.
* В реестре, зашифрованном с помощью управляемого клиентом ключа, файлы журналов для[задач ACR](container-registry-tasks-overview.md) в настоящее время хранятся только в течение 24 часов. Если вам необходимо хранить файлы журналов дольше, ознакомьтесь с инструкциями по [экспорту и хранению журналов выполнения задач](container-registry-tasks-logs.md#alternative-log-storage).


> [!IMPORTANT]
> Если вы планируете хранить ключ шифрования реестра в существующем хранилище ключей Azure, которое запрещает общий доступ и разрешает только закрытую конечную точку или выбранные виртуальные сети, необходимо выполнить дополнительные действия по настройке. См. [Расширенный сценарий: Key Vault брандмауэр](#advanced-scenario-key-vault-firewall) в этой статье.

## <a name="automatic-or-manual-update-of-key-versions"></a>Автоматическое или ручное обновление основных версий

Важно учитывать, что безопасность реестра, зашифрованного с помощью управляемого клиентом ключа, — это частота обновления (смены) ключа шифрования. У вашей организации могут быть политики соответствия, требующие регулярного обновления основных [версий](../key-vault/general/about-keys-secrets-certificates.md#objects-identifiers-and-versioning) , хранящихся в Azure Key Vault при использовании в качестве ключей, управляемых клиентом. 

При настройке шифрования реестра с помощью ключа, управляемого клиентом, существует два варианта обновления версии ключа, используемой для шифрования:

* **Автоматически обновлять ключ версии** . чтобы автоматически обновлять ключ, управляемый клиентом, когда доступна новая версия в Azure Key Vault, не указывайте версию ключа при включении шифрования реестра с помощью ключа, управляемого клиентом. При шифровании реестра с ключом, не относящимся к версии, реестр контейнеров Azure регулярно проверяет хранилище ключей на наличие новой версии ключа и обновляет ключ, управляемый клиентом, в течение 1 часа. Реестр контейнеров Azure автоматически использует последнюю версию ключа.

* **Обновите версию ключа вручную** . чтобы использовать определенную версию ключа для шифрования реестра, укажите эту версию ключа при включении шифрования реестра с ключом, управляемым клиентом. Если реестр шифруется с помощью определенной версии ключа, реестр контейнеров Azure использует эту версию для шифрования до тех пор, пока вы вручную не повернете ключ, управляемый клиентом.

Дополнительные сведения см. в разделе [Выбор идентификатора ключа с версией ключа или без](#choose-key-id-with-or-without-key-version) нее и [Обновление версии ключа](#update-key-version)далее в этой статье.

## <a name="prerequisites"></a>Предварительные требования

Чтобы использовать Azure CLI действия, описанные в этой статье, необходимо Azure CLI версии 2.2.0 или более поздней или Azure Cloud Shell. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="enable-customer-managed-key---cli"></a>Активация управляемых клиентом ключей через CLI

### <a name="create-a-resource-group"></a>Создание группы ресурсов

При необходимости выполните команду [az Group Create][az-group-create], чтобы создать группу ресурсов для создания хранилища ключей, реестра контейнеров и других необходимых ресурсов.

```azurecli
az group create --name <resource-group-name> --location <location>
```

### <a name="create-a-user-assigned-managed-identity"></a>Создание управляемого удостоверения, назначаемого пользователем

Чтобы создать [управляемое удостоверение, назначаемое пользователем](../active-directory/managed-identities-azure-resources/overview.md), используйте команду [az identity create][az-identity-create]. Это удостоверение будет использоваться реестром для доступа к службе Key Vault.

```azurecli
az identity create \
  --resource-group <resource-group-name> \
  --name <managed-identity-name>
```

В выходных данных команды обратите внимание на значения `id` и `principalId`. Они будут необходимы в последующих шагах для настройки доступа к хранилищу ключей в реестре.

```JSON
{
  "clientId": "xxxx2bac-xxxx-xxxx-xxxx-192cxxxx6273",
  "clientSecretUrl": "https://control-eastus.identity.azure.net/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myresourcegroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myidentityname/credentials?tid=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&oid=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&aid=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myresourcegroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myresourcegroup",
  "location": "eastus",
  "name": "myidentityname",
  "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "resourceGroup": "myresourcegroup",
  "tags": {},
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.ManagedIdentity/userAssignedIdentities"
}
```

Для удобства можно сохранить эти значения в переменных среды:

```azurecli
identityID=$(az identity show --resource-group <resource-group-name> --name <managed-identity-name> --query 'id' --output tsv)

identityPrincipalID=$(az identity show --resource-group <resource-group-name> --name <managed-identity-name> --query 'principalId' --output tsv)
```

### <a name="create-a-key-vault"></a>Создайте хранилище ключей.

Создайте хранилище ключей командой [az keyvault Create][az-keyvault-create] для хранения ключа, управляемого клиентом, используемого для шифрования реестра. 

По умолчанию параметр **обратимого удаления** автоматически включается в новом хранилище ключей. Чтобы предотвратить потери данных, вызванные случайными операциями удаления ключей или хранилища ключей, также включите параметр **Защита от очистки** .

```azurecli
az keyvault create --name <key-vault-name> \
  --resource-group <resource-group-name> \
  --enable-purge-protection
```

Для использования в последующих шагах получите идентификатор ресурса хранилища ключей:

```azurecli
keyvaultID=$(az keyvault show --resource-group <resource-group-name> --name <key-vault-name> --query 'id' --output tsv)
```

### <a name="enable-key-vault-access"></a>Включение доступа к хранилищу ключей

Настройте политику для хранилища ключей, чтобы идентификатор мог получить к нему доступ. При следующем использовании [команды az keyvault Set-Policy][az-keyvault-set-policy] вы передадите идентификатор субъекта созданного управляемого удостоверения, сохраненного ранее в переменной среды. Задайте разрешения для ключа: **get**, **unwrapKey** и **wrapKey**.  

```azurecli
az keyvault set-policy \
  --resource-group <resource-group-name> \
  --name <key-vault-name> \
  --object-id $identityPrincipalID \
  --key-permissions get unwrapKey wrapKey
```

Кроме того, можно использовать [Azure RBAC для Key Vault](../key-vault/general/rbac-guide.md) для назначения разрешений удостоверению для доступа к хранилищу ключей. Например, назначьте удостоверению Key Vault для удостоверения службы шифрования с помощью команды [AZ Role назначение Create](/cli/azure/role/assignment#az-role-assignment-create) .

```azurecli 
az role assignment create --assignee $identityPrincipalID \
  --role "Key Vault Crypto Service Encryption User" \
  --scope $keyvaultID
```

### <a name="create-key-and-get-key-id"></a>Создайте ключ и получите идентификатор ключа

С помощью команды [az keyvault key create][az-keyvault-key-create] создайте ключ в хранилище.

```azurecli
az keyvault key create \
  --name <key-name> \
  --vault-name <key-vault-name>
```

Запомните идентификатор ключа в выходных данных — `kid`. Он потребуется для следующего шага.

```JSON
[...]
  "key": {
    "crv": null,
    "d": null,
    "dp": null,
    "dq": null,
    "e": "AQAB",
    "k": null,
    "keyOps": [
      "encrypt",
      "decrypt",
      "sign",
      "verify",
      "wrapKey",
      "unwrapKey"
    ],
    "kid": "https://mykeyvault.vault.azure.net/keys/mykey/<version>",
    "kty": "RSA",
[...]
```

### <a name="choose-key-id-with-or-without-key-version"></a>Выберите идентификатор ключа с версией ключа или без нее

Для удобства сохраните формат, выбранный для идентификатора ключа, в переменной среды $keyID. Идентификатор ключа можно использовать с версией или ключом без версии.

#### <a name="manual-key-rotation---key-id-with-version"></a>Смена ключа вручную — идентификатор ключа с версией

При использовании для шифрования реестра с ключом, управляемым клиентом, этот ключ позволяет вручную сменить ключ в реестре контейнеров Azure.

В этом примере хранится свойство ключа `kid` :

```azurecli
keyID=$(az keyvault key show \
  --name <keyname> \
  --vault-name <key-vault-name> \
  --query 'key.kid' --output tsv)
```

#### <a name="automatic-key-rotation---key-id-omitting-version"></a>Автоматическая смена ключа — идентификатор ключа пропущена версия 

При использовании для шифрования реестра с ключом, управляемым клиентом, этот ключ включает автоматическое вращение ключей при обнаружении новой версии ключа в Azure Key Vault.

В этом примере удаляется версия из свойства ключа `kid` :

```azurecli
keyID=$(az keyvault key show \
  --name <keyname> \
  --vault-name <key-vault-name> \
  --query 'key.kid' --output tsv)

keyID=$(echo $keyID | sed -e "s/\/[^/]*$//")
```

### <a name="create-a-registry-with-customer-managed-key"></a>Создайте реестр с использованием управляемого клиентом ключа.

Запустите команду [az acr create][az-acr-create], чтобы создать реестр на уровне службы Premium и включить управляемый клиентом ключ. Передайте идентификатор управляемого удостоверения и идентификатор ключа, сохраненные ранее в переменных среды:

```azurecli
az acr create \
  --resource-group <resource-group-name> \
  --name <container-registry-name> \
  --identity $identityID \
  --key-encryption-key $keyID \
  --sku Premium
```

### <a name="show-encryption-status"></a>Показать статус шифрования

Чтобы узнать, включено ли шифрование реестра с ключом, управляемым клиентом, выполните команду [az acr encryption show][az-acr-encryption-show]:

```azurecli
az acr encryption show --name <registry-name>
```

В зависимости от ключа, используемого для шифрования реестра, выходные данные похожи на:

```console
{
  "keyVaultProperties": {
    "identity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "keyIdentifier": "https://myvault.vault.azure.net/keys/myresourcegroup/abcdefg123456789...",
    "keyRotationEnabled": true,
    "lastKeyRotationTimestamp": xxxxxxxx
    "versionedKeyIdentifier": "https://myvault.vault.azure.net/keys/myresourcegroup/abcdefg123456789...",
  },
  "status": "enabled"
}
```

## <a name="enable-customer-managed-key---portal"></a>Активация управляемых клиентом ключей через портал

### <a name="create-a-managed-identity"></a>Создание управляемого удостоверения

Создайте [управляемое удостоверение](../active-directory/managed-identities-azure-resources/overview.md), назначаемое пользователем, через портал Azure. Подробную инструкцию можно получить на [странице создания управляемого пользователем удостоверения](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity).

Имя удостоверения будет использоваться в последующих шагах.

:::image type="content" source="media/container-registry-customer-managed-keys/create-managed-identity.png" alt-text="Создание назначаемого пользователем управляемого удостоверения через портал Azure":::

### <a name="create-a-key-vault"></a>Создание хранилища ключей

Инструкции по созданию хранилища ключей см. в разделе [Краткое руководство. Создание хранилища ключей с помощью портал Azure](../key-vault/general/quick-create-portal.md).

При создании хранилища ключей для управляемого клиентом ключа на вкладке " **базовые** " включите параметр **Защита от очистки** . Этот параметр позволяет предотвратить потери данных, вызванные случайными операциями удаления ключей или хранилища ключей.

:::image type="content" source="media/container-registry-customer-managed-keys/create-key-vault.png" alt-text="Создайте хранилище ключей через портал Azure":::

### <a name="enable-key-vault-access"></a>Включение доступа к хранилищу ключей

Настройте политику для хранилища ключей, чтобы идентификатор мог получить к нему доступ.

1. Перейдите в службу хранилища ключей.
1. Выберите пункт **Параметры** > **Политика доступа > +Добавить политику доступа**.
1. В разделе **Разрешения ключей** выберите **Получение**, **Распаковка ключа** и **Упаковка ключа**.
1. В поле **Выбор субъекта** выберите имя ресурса для управляемого удостоверения, назначенного пользователем.  
1. Нажмите кнопку **Добавить**, а затем щелкните **Сохранить**.

:::image type="content" source="media/container-registry-customer-managed-keys/add-key-vault-access-policy.png" alt-text="Создайте политику доступа к хранилищу ключей":::

Кроме того, можно использовать [Azure RBAC для Key Vault](../key-vault/general/rbac-guide.md) для назначения разрешений удостоверению для доступа к хранилищу ключей. Например, назначьте удостоверению Key Vault роль шифрования службы криптографии.

1. Перейдите в службу хранилища ключей.
1. Выберите **Управление доступом (IAM)**  >  **+ добавить**  >  **назначение роли добавить**.
1. В окне **Добавление назначения ролей** :
    1. Выберите **Key Vault роль пользователя шифрование службы шифрования** . 
    1. Назначение доступа для **управляемого удостоверения, назначенного пользователем**.
    1. Выберите имя ресурса, назначаемое пользователем управляемое удостоверение, и нажмите кнопку **сохранить**.

### <a name="create-key-optional"></a>Создать ключ (необязательно)

При необходимости создайте ключ в хранилище ключей, который будет использоваться для шифрования реестра. Выполните следующие действия, если необходимо выбрать конкретную версию ключа в качестве ключа, управляемого клиентом. 

1. Перейдите в службу хранилища ключей.
1. Последовательно выберите **Параметры** > **Ключи**.
1. Выберите **+ создать или импортировать** и введите уникальное имя для ключа.
1. Оставьте остальные значения по умолчанию и выберите **Создать**.
1. После создания выберите ключ и выберите текущую версию. Скопируйте **идентификатор ключа** для версии ключа.

### <a name="create-azure-container-registry"></a>Создание реестра контейнеров Azure

1. Последовательно выберите **Создать ресурс** > **Контейнеры** > **Реестр контейнеров**.
1. На вкладке **Основы** выберите или создайте группу ресурсов, затем введите имя реестра. В поле **SKU** выберите пункт **Премиум**.
1. На вкладке **шифрование** в разделе **управляемых клиентом ключей** выберите пункт **включено**.
1. В **разделе удостоверений** выберите управляемое удостоверение, которое вы создали.
1. В поле **Шифрование** выберите один из следующих элементов:
    * Выберите **выбрать из Key Vault** и выберите существующее хранилище ключей и ключ или **Создайте новый**. Выбранный ключ не имеет версии и включает автоматическое вращение ключей.
    * Выберите **ввести URI ключа** и укажите идентификатор ключа напрямую. Вы можете указать универсальный код ресурса (URI) ключа (для ключа, который необходимо поворачивать вручную) или универсальный код ресурса (URI) без версии (что позволяет автоматически сменить ключ). 
1. На вкладке **Шифрование** выберите пункт **Проверить + создать**.
1. Нажмите кнопку **Создать**, чтобы активировать реестр.

:::image type="content" source="media/container-registry-customer-managed-keys/create-encrypted-registry.png" alt-text="Создайте зашифрованный реестр в портал Azure":::

Чтобы просмотреть состояние шифрования реестра на портале, перейдите к реестру. В разделе **Параметры** выберите **Шифрование**.

## <a name="enable-customer-managed-key---template"></a>Активация управляемых клиентом ключей — шаблон

Можно также использовать шаблон диспетчера ресурсов, чтобы создать реестр и включить шифрование с помощью ключа, управляемого клиентом.

Следующий шаблон создаст новый реестр контейнеров и назначаемое пользователем удостоверение. Скопируйте следующее содержимое в новый файл и сохраните его, используя имя файла вида `CMKtemplate.json`.

```JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vault_name": {
      "defaultValue": "",
      "type": "String"
    },
    "registry_name": {
      "defaultValue": "",
      "type": "String"
    },
    "identity_name": {
      "defaultValue": "",
      "type": "String"
    },
    "kek_id": {
      "type": "String"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.ContainerRegistry/registries",
      "apiVersion": "2019-12-01-preview",
      "name": "[parameters('registry_name')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Premium",
        "tier": "Premium"
      },
      "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
          "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name'))]": {}
        }
      },
      "dependsOn": [
        "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name'))]"
      ],
      "properties": {
        "adminUserEnabled": false,
        "encryption": {
          "status": "enabled",
          "keyVaultProperties": {
            "identity": "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name')), '2018-11-30').clientId]",
            "KeyIdentifier": "[parameters('kek_id')]"
          }
        },
        "networkRuleSet": {
          "defaultAction": "Allow",
          "virtualNetworkRules": [],
          "ipRules": []
        },
        "policies": {
          "quarantinePolicy": {
            "status": "disabled"
          },
          "trustPolicy": {
            "type": "Notary",
            "status": "disabled"
          },
          "retentionPolicy": {
            "days": 7,
            "status": "disabled"
          }
        }
      }
    },
    {
      "type": "Microsoft.KeyVault/vaults/accessPolicies",
      "apiVersion": "2018-02-14",
      "name": "[concat(parameters('vault_name'), '/add')]",
      "dependsOn": [
        "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name'))]"
      ],
      "properties": {
        "accessPolicies": [
          {
            "tenantId": "[subscription().tenantId]",
            "objectId": "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name')), '2018-11-30').principalId]",
            "permissions": {
              "keys": [
                "get",
                "unwrapKey",
                "wrapKey"
              ]
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
      "apiVersion": "2018-11-30",
      "name": "[parameters('identity_name')]",
      "location": "[resourceGroup().location]"
    }
  ]
}
```

Выполните действия, описанные в предыдущих разделах, чтобы создать следующие ресурсы:

* Хранилище ключей, идентифицируемое по имени
* Ключ хранилища ключей, идентифицируемый по идентификатору ключа

Выполните следующую команду [AZ Deployment Group Create][az-deployment-group-create] , чтобы создать реестр с помощью предыдущего файла шаблона. Укажите в соответствующих полях новое имя реестра и имя управляемого удостоверения, а также созданные вами имя хранилища ключей и идентификатор ключа.

```bash
az deployment group create \
  --resource-group <resource-group-name> \
  --template-file CMKtemplate.json \
  --parameters \
    registry_name=<registry-name> \
    identity_name=<managed-identity> \
    vault_name=<key-vault-name> \
    kek_id=<key-vault-key-id>
```

### <a name="show-encryption-status"></a>Показать статус шифрования

Чтобы отобразить состояние шифрования реестра, выполните команду [az acr encryption show][az-acr-encryption-show]:

```azurecli
az acr encryption show --name <registry-name>
```

## <a name="use-the-registry"></a>Использование реестра

После включения ключа, управляемого клиентом, в реестр, можно выполнять те же операции с реестром, которые выполняются в незашифрованном реестре. Например, можно выполнить проверку подлинности в реестре и отправить образы DOCKER. Примеры команд можно увидеть в [Push and Pull a Image](container-registry-get-started-docker-cli.md).

## <a name="rotate-key"></a>Сменить ключ

Обновите версию ключа в Azure Key Vault или создайте новый ключ, а затем обновите реестр для шифрования данных с помощью ключа. Это можно сделать с помощью Azure CLI или на портале.

При смене ключа обычно указывается то же удостоверение, которое использовалось при создании реестра. Также вы всегда можете настроить новое пользовательское удостоверение для доступа к ключам или же указать назначенное системой удостоверение для реестра.

> [!NOTE]
> Убедитесь, что для удостоверения, настроенного для доступа к ключу, установлен требуемый [доступ к хранилищу ключей](#enable-key-vault-access) .

### <a name="update-key-version"></a>Обновить версию ключа

Распространенным сценарием является обновление версии ключа, используемого в качестве ключа, управляемого клиентом. В зависимости от того, как настроено шифрование реестра, ключ, управляемый клиентом в реестре контейнеров Azure, обновляется автоматически, или его необходимо обновить вручную.

### <a name="azure-cli"></a>Azure CLI

Для создания ключей хранилища или управления ими используйте команду [az keyvault key][az-keyvault-key]. Чтобы создать новую версию ключа, выполните команду [AZ keyvault Key Create][az-keyvault-key-create] :

```azurecli
# Create new version of existing key
az keyvault key create \
  –-name <key-name> \
  --vault-name <key-vault-name>
```

Следующий шаг зависит от настройки шифрования реестра.

* Если реестр настроен для обнаружения обновлений основных версий, то ключ, управляемый клиентом, обновляется автоматически в течение 1 часа.

* Если в реестре настроено требование ручного обновления для новой версии ключа, выполните команду [AZ запись контроля доступа][az-acr-encryption-rotate-key] , ПЕРЕДАВ новый идентификатор ключа и удостоверение, которое нужно настроить:

Чтобы вручную обновить версию ключа, управляемого клиентом, выполните следующие действия.

```azurecli
# Rotate key and use user-assigned identity
az acr encryption rotate-key \
  --name <registry-name> \
  --key-encryption-key <new-key-id> \
  --identity <principal-id-user-assigned-identity>

# Rotate key and use system-assigned identity
az acr encryption rotate-key \
  --name <registry-name> \
  --key-encryption-key <new-key-id> \
  --identity [system]
```

> [!TIP]
> При запуске `az acr encryption rotate-key` можно передать либо идентификатор ключа версии, либо идентификатор ключа без номера версии. Если используется идентификатор ключа без номера версии, реестр настраивается для автоматического обнаружения последующих обновлений версии ключа.

### <a name="portal"></a>Портал

Используйте параметры **шифрования** реестра, чтобы обновить параметры хранилища ключей, ключа или удостоверения, используемые для ключа, управляемого клиентом.

Например, чтобы настроить новый ключ, выполните следующие действия.

1. На портале перейдите к нужному реестру.
1. В разделе **Настройки** выберите пункты **Шифрование** > **Изменить ключ**.

    :::image type="content" source="media/container-registry-customer-managed-keys/rotate-key.png" alt-text="Смените ключ через портал Azure":::
1. В поле **Шифрование** выберите одно из следующих:
    * Выберите **выбрать из Key Vault** и выберите существующее хранилище ключей и ключ или **Создайте новый**. Выбранный ключ не имеет версии и включает автоматическое вращение ключей.
    * Выберите **ввести URI ключа** и укажите идентификатор ключа напрямую. Вы можете указать универсальный код ресурса (URI) ключа (для ключа, который необходимо поворачивать вручную) или универсальный код ресурса (URI) без версии (что позволяет автоматически сменить ключ).
1. Завершите выбор ключа и нажмите на кнопку **Сохранить**.

## <a name="revoke-key"></a>Как отозвать ключ

Отозвать ключ шифрования, управляемый клиентом, можно, изменив политику доступа или разрешения для хранилища ключей или удалив ключ. Например, используйте команду [az keyvault delete-policy][az-keyvault-delete-policy], чтобы изменить политику доступа управляемого удостоверения, используемого реестром.

```azurecli
az keyvault delete-policy \
  --resource-group <resource-group-name> \
  --name <key-vault-name> \
  --object-id $identityPrincipalID
```

Отзыв ключа мгновенно блокирует доступ ко всем данным реестра, так как реестр не сможет получить доступ к ключу шифрования. Если доступ к ключу будет заново активирован или удаленный ключ будет восстановлен, реестр выберет ключ для повторного получения доступа к зашифрованным данным реестра.

## <a name="advanced-scenario-key-vault-firewall"></a>Расширенный сценарий: Key Vault брандмауэр

Вы можете сохранить ключ шифрования с помощью существующего хранилища ключей Azure, настроенного с помощью [Key Vault брандмауэре](../key-vault/general/network-security.md), который запрещает общий доступ и разрешает только частные конечные точки или выбранные виртуальные сети. 

В этом сценарии сначала создайте новое назначенное пользователем удостоверение, хранилище ключей и реестр контейнеров, зашифрованные с помощью ключа, управляемого клиентом, используя [Azure CLI](#enable-customer-managed-key---cli), [портал](#enable-customer-managed-key---portal)или [шаблон](#enable-customer-managed-key---template). Подробные инструкции приведены в предыдущих разделах этой статьи.
   > [!NOTE]
   > Новое хранилище ключей развертывается за пределами брандмауэра. Он используется временно для хранения ключа, управляемого клиентом.

После создания реестра выполните следующие действия. Подробные сведения приведены в следующих разделах.

1. Включите назначенное системой удостоверение реестра.
1. Предоставьте назначенным системе разрешения удостоверения для доступа к ключам в хранилище ключей, которое ограничено брандмауэром Key Vault.
1. Убедитесь, что брандмауэр Key Vault разрешается выполнять обход с помощью доверенных служб. В настоящее время реестр контейнеров Azure может обходить только брандмауэр при использовании управляемого системой удостоверения. 
1. Поворачивайте управляемый клиентом ключ, выбрав его в хранилище ключей, которое ограничено брандмауэром Key Vault.
1. Вы можете удалить хранилище ключей, созданное за пределами брандмауэра, если оно больше не требуется.


### <a name="step-1---enable-registrys-system-assigned-identity"></a>Шаг 1. Включение назначенного системой удостоверения реестра

1. На портале перейдите к нужному реестру.
1. Перейдите в пункт **Настройки** >  **Удостоверение**.
1. В разделе **Назначенные системой** задайте для параметра **Состояние** значение **Включено**. Щелкните **Сохранить**.
1. Скопируйте **идентификатор** удостоверения.

### <a name="step-2---grant-system-assigned-identity-access-to-your-key-vault"></a>Шаг 2. предоставление системе доступа к хранилищу ключей, назначенному системой

1. На портале перейдите к своему хранилищу ключей.
1. Выберите пункт **Параметры** > **Политика доступа > +Добавить политику доступа**.
1. В разделе **Разрешения ключей** выберите **Получение**, **Распаковка ключа** и **Упаковка ключа**.
1. Выберите **выбрать субъект** и выполните поиск идентификатора объекта управляемого удостоверения, назначенного системой, или имя реестра.  
1. Нажмите кнопку **Добавить**, а затем щелкните **Сохранить**.

### <a name="step-3---enable-key-vault-bypass"></a>Шаг 3. Включение обхода хранилища ключей

Для доступа к хранилищу ключей, настроенному с помощью брандмауэра Key Vault, реестр должен обходить брандмауэр. Убедитесь, что хранилище ключей настроено для разрешения доступа любой [доверенной службой](../key-vault/general/overview-vnet-service-endpoints.md#trusted-services). Реестр контейнеров Azure — это одна из доверенных служб.

1. На портале перейдите к своему хранилищу ключей.
1. Выберите **Параметры**  >  **сети**.
1. Подтвердите, обновите или добавьте параметры виртуальной сети. Подробную инструкцию можно узнать на странице [Настройка брандмауэров и виртуальных сетей Azure Key Vault](../key-vault/general/network-security.md).
1. **Чтобы разрешить доверенным службам Майкрософт обход этого брандмауэра**, выберите **Да**. 

### <a name="step-4---rotate-the-customer-managed-key"></a>Шаг 4. поворот управляемого клиентом ключа

После выполнения описанных выше действий необходимо повернуть на ключ, хранящийся в хранилище ключей, защищенном брандмауэром.

1. На портале перейдите к нужному реестру.
1. В разделе **Параметры** выберите   >  **ключ изменения** шифрования.
1. В списке **удостоверение** выберите пункт **назначено системой**.
1. Выберите **выбрать из Key Vault** и укажите имя хранилища ключей, которое находится за брандмауэром.
1. Выберите существующий ключ или **Создайте новый**. Выбранный ключ не имеет версии и включает автоматическое вращение ключей.
1. Завершите выбор ключа и нажмите на кнопку **Сохранить**.

## <a name="troubleshoot"></a>Диагностика

### <a name="removing-managed-identity"></a>Удаление управляемого удостоверения


При попытке удалить назначенное пользователем или управляемое системой удостоверение из реестра, используемого для настройки шифрования, может появиться сообщение об ошибке следующего вида:
 
```
Azure resource '/subscriptions/xxxx/resourcegroups/myGroup/providers/Microsoft.ContainerRegistry/registries/myRegistry' does not have access to identity 'xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx' Try forcibly adding the identity to the registry <registry name>. For more information on bring your own key, please visit 'https://aka.ms/acr/cmk'.
```
 
Кроме того, невозможно изменить (повернуть) ключ шифрования. Действия по устранению зависят от типа удостоверения, используемого для шифрования.

**Назначаемое пользователем удостоверение**

Если эта проблема возникает при назначении пользователем удостоверения, сначала необходимо переназначить удостоверение с помощью идентификатора GUID, отображаемого в сообщении об ошибке. Пример:

```azurecli
az acr identity assign -n myRegistry --identities xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx
```
        
После изменения ключа и назначения другого удостоверения можно удалить исходное удостоверение, назначенное пользователем.

**Назначаемое системой удостоверение**

Если эта проблема возникает при использовании назначенного системой удостоверения, [Создайте запрос в службу поддержки Azure](https://azure.microsoft.com/support/create-ticket/) , чтобы получить помощь по восстановлению удостоверения.


## <a name="next-steps"></a>Дальнейшие действия

* Узнайте больше о [Шифровании неактивных данных в Azure](../security/fundamentals/encryption-atrest.md).
* Узнайте больше о политиках доступа и о том, как [обезопасить доступ к хранилищу ключей](../key-vault/general/secure-your-key-vault.md).


<!-- LINKS - external -->

<!-- LINKS - internal -->

[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-show]: /cli/azure/feature#az-feature-show
[az-group-create]: /cli/azure/group#az-group-create
[az-identity-create]: /cli/azure/identity#az-identity-create
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
[az-keyvault-create]: /cli/azure/keyvault#az-keyvault-create
[az-keyvault-key-create]: /cli/azure/keyvault/key#az-keyvault-key-create
[az-keyvault-key]: /cli/azure/keyvault/key
[az-keyvault-set-policy]: /cli/azure/keyvault#az-keyvault-set-policy
[az-keyvault-delete-policy]: /cli/azure/keyvault#az-keyvault-delete-policy
[az-resource-show]: /cli/azure/resource#az-resource-show
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-acr-encryption-rotate-key]: /cli/azure/acr/encryption#az-acr-encryption-rotate-key
[az-acr-encryption-show]: /cli/azure/acr/encryption#az-acr-encryption-show
