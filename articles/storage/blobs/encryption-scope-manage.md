---
title: Создание областей шифрования и управление ими
description: Узнайте, как создать область шифрования для изоляции данных большого двоичного объекта на уровне контейнера или большого двоичного объекта.
services: storage
author: tamram
ms.service: storage
ms.date: 03/26/2021
ms.topic: conceptual
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: c29282637f6854248c98dff59f8fae46ad1a9d39
ms.sourcegitcommit: c8b50a8aa8d9596ee3d4f3905bde94c984fc8aa2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/28/2021
ms.locfileid: "105640543"
---
# <a name="create-and-manage-encryption-scopes"></a>Создание областей шифрования и управление ими

Области шифрования позволяют управлять шифрованием на уровне отдельного большого двоичного объекта или контейнера. Области шифрования можно использовать для создания безопасных границ между данными, которые находятся в одной учетной записи хранения, но принадлежат разным клиентам. Дополнительные сведения об областях шифрования см. в статье [области шифрования для хранилища BLOB-объектов](encryption-scope-overview.md).

В этой статье показано, как создать область шифрования. В нем также показано, как указать область шифрования при создании большого двоичного объекта или контейнера.

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="create-an-encryption-scope"></a>Создание области шифрования

Можно создать область шифрования, защищенную с помощью ключа, управляемого корпорацией Майкрософт, или с помощью управляемого клиентом ключа, который хранится в Azure Key Vault или в Azure Key Vault управляемой аппаратной модели безопасности (HSM) (Предварительная версия). Чтобы создать область шифрования с помощью управляемого клиентом ключа, необходимо сначала создать хранилище ключей или управляемый модуль HSM, а затем добавить ключ, который предполагается использовать для этой области. Для хранилища ключей или управляемого модуля HSM должна быть включена защита от вирусов и она должна находиться в том же регионе, что и учетная запись хранения.

При создании область шифрования автоматически включается. После создания области шифрования ее можно указать при создании большого двоичного объекта. Можно также указать область шифрования по умолчанию при создании контейнера, который автоматически применяется ко всем BLOB-объектам в контейнере.

# <a name="portal"></a>[Портал](#tab/portal)

Чтобы создать область шифрования в портал Azure, выполните следующие действия.

1. Войдите в свою учетную запись хранения на портале Azure.
1. Выберите параметр **шифрования** .
1. Перейдите на вкладку **области шифрования** .
1. Нажмите кнопку **Добавить** , чтобы добавить новую область шифрования.
1. В области **Создание области шифрования** введите имя новой области.
1. Выберите необходимый тип поддержки ключа шифрования — **ключи, управляемые корпорацией Майкрософт** , или **ключи, управляемые клиентом**.
    - Если вы выбрали **ключи, управляемые корпорацией Майкрософт**, нажмите кнопку **создать** , чтобы создать область шифрования.
    - Если вы выбрали **ключи, управляемые клиентом**, выберите подписку и укажите хранилище ключей или управляемый модуль HSM, а также ключ, который будет использоваться для этой области шифрования, как показано на следующем рисунке.

    :::image type="content" source="media/encryption-scope-manage/create-encryption-scope-customer-managed-key-portal.png" alt-text="Снимок экрана, показывающий, как создать область шифрования в портал Azure":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы создать область шифрования с помощью PowerShell, установите модуль PowerShell [AZ. Storage](https://www.powershellgallery.com/packages/Az.Storage) , версия 3.4.0 или более поздняя.

### <a name="create-an-encryption-scope-protected-by-microsoft-managed-keys"></a>Создание области шифрования, защищенной с помощью ключей, управляемых корпорацией Майкрософт

Чтобы создать новую область шифрования, защищенную с помощью ключей, управляемых корпорацией Майкрософт, вызовите команду **New-азсторажеенкриптионскопе** с `-StorageEncryption` параметром.

Не забудьте заменить значения заполнителей в примере собственными значениями:

```powershell
$rgName = "<resource-group>"
$accountName = "<storage-account>"
$scopeName1 = "customer1scope"

New-AzStorageEncryptionScope -ResourceGroupName $rgName `
    -AccountName $accountName `
    -EncryptionScopeName $scopeName1 `
    -StorageEncryption
```

### <a name="create-an-encryption-scope-protected-by-customer-managed-keys"></a>Создание области шифрования, защищенной ключами, управляемыми клиентом

Чтобы создать новую область шифрования, защищенную управляемыми клиентом ключами, хранящимися в хранилище ключей или управляемым модулем HSM, сначала настройте ключи, управляемые клиентом, для учетной записи хранения. Необходимо назначить управляемое удостоверение учетной записи хранения, а затем использовать управляемое удостоверение, чтобы настроить политику доступа для хранилища ключей или управляемого модуля HSM, чтобы у учетной записи хранения были разрешения на доступ к ней.

Чтобы настроить ключи, управляемые клиентом, для использования с областью шифрования, необходимо включить защиту в хранилище ключей или управляемом HSM. Хранилище ключей или управляемый модуль HSM должны находиться в том же регионе, что и учетная запись хранения.

Не забудьте заменить значения заполнителей в примере собственными значениями:

```powershell
$rgName = "<resource-group>"
$accountName = "<storage-account>"
$keyVaultName = "<key-vault>"
$keyUri = "<key-uri>"
$scopeName2 = "customer2scope"

# Assign a system managed identity to the storage account.
$storageAccount = Set-AzStorageAccount -ResourceGroupName $rgName `
    -Name $accountName `
    -AssignIdentity

# Configure the access policy for the key vault.
Set-AzKeyVaultAccessPolicy `
    -VaultName $keyVaultName `
    -ObjectId $storageAccount.Identity.PrincipalId `
    -PermissionsToKeys wrapkey,unwrapkey,get
```

Затем вызовите команду **New-азсторажеенкриптионскопе** с `-KeyvaultEncryption` параметром и укажите URI ключа. Включение версии ключа в URI ключа является необязательным. Если опустить версию ключа, то в области шифрования будет автоматически использоваться последняя версия ключа. При включении версии ключа необходимо вручную обновить версию ключа, чтобы использовать другую версию.

Не забудьте заменить значения заполнителей в примере собственными значениями:

```powershell
New-AzStorageEncryptionScope -ResourceGroupName $rgName `
    -AccountName $accountName `
    -EncryptionScopeName $scopeName2 `
    -KeyUri $keyUri `
    -KeyvaultEncryption
```

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Чтобы создать область шифрования с Azure CLI, сначала установите Azure CLI версии 2.20.0 или более поздней.

### <a name="create-an-encryption-scope-protected-by-microsoft-managed-keys"></a>Создание области шифрования, защищенной с помощью ключей, управляемых корпорацией Майкрософт

Чтобы создать новую область шифрования, защищенную с помощью ключей, управляемых корпорацией Майкрософт, вызовите команду [AZ Storage Account Encryption-Scope Create](/cli/azure/storage/account/encryption-scope#az-storage-account-encryption-scope-create) , указав `--key-source` параметр в качестве `Microsoft.Storage` . Не забудьте заменить значения заполнителей собственными значениями.

```azurecli-interactive
az storage account encryption-scope create \
    --resource-group <resource-group> \
    --account-name <storage-account> \
    --name <scope> \
    --key-source Microsoft.Storage
```

### <a name="create-an-encryption-scope-protected-by-customer-managed-keys"></a>Создание области шифрования, защищенной ключами, управляемыми клиентом

Чтобы создать новую область шифрования, защищенную с помощью ключей, управляемых корпорацией Майкрософт, вызовите команду [AZ Storage Account Encryption-Scope Create](/cli/azure/storage/account/encryption-scope#az-storage-account-encryption-scope-create) , указав `--key-source` параметр в качестве `Microsoft.Storage` . Не забудьте заменить значения заполнителей собственными значениями.

Чтобы создать новую область шифрования, защищенную управляемыми клиентом ключами в хранилище ключей или управляемом модулем HSM, сначала настройте управляемые клиентом ключи для учетной записи хранения. Необходимо назначить управляемое удостоверение учетной записи хранения, а затем использовать управляемое удостоверение, чтобы настроить политику доступа для хранилища ключей, чтобы у учетной записи хранения были разрешения на доступ к ней. Дополнительные сведения см. в статье [Ключи, управляемые клиентом, для шифрования службы хранилища Azure](../common/customer-managed-keys-overview.md).

Чтобы настроить ключи, управляемые клиентом, для использования с областью шифрования, необходимо включить защиту в хранилище ключей или управляемом HSM. Хранилище ключей или управляемый модуль HSM должны находиться в том же регионе, что и учетная запись хранения.

Не забудьте заменить значения заполнителей в примере собственными значениями:

```azurecli-interactive
az login
az account set --subscription <subscription-id>

az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --assign-identity

storage_account_principal=$(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query identity.principalId \
    --output tsv)

az keyvault set-policy \
    --name <key-vault> \
    --resource-group <resource_group> \
    --object-id $storage_account_principal \
    --key-permissions get unwrapKey wrapKey
```

Затем вызовите команду **AZ Storage Account Encryption-Scope Create** с `--key-uri` параметром и укажите URI ключа. Включение версии ключа в URI ключа является необязательным. Если опустить версию ключа, то в области шифрования будет автоматически использоваться последняя версия ключа. При включении версии ключа необходимо вручную обновить версию ключа, чтобы использовать другую версию.

Не забудьте заменить значения заполнителей в примере собственными значениями:

```azurecli-interactive
az storage account encryption-scope create \
    --resource-group <resource-group> \
    --account-name <storage-account> \
    --name <scope> \
    --key-source Microsoft.KeyVault \
    --key-uri <key-uri>
```

---

Сведения о настройке шифрования службы хранилища Azure с помощью управляемых клиентом ключей в хранилище ключей или управляемом модуле HSM см. в следующих статьях:

- [Настройка шифрования с помощью управляемых клиентом ключей, хранящихся в Azure Key Vault](../common/customer-managed-keys-configure-key-vault.md)
- [Настройка шифрования с помощью управляемых клиентом ключей, хранящихся в Azure Key Vault управляемый HSM (Предварительная версия)](../common/customer-managed-keys-configure-key-vault-hsm.md).

## <a name="list-encryption-scopes-for-storage-account"></a>Список областей шифрования для учетной записи хранения

# <a name="portal"></a>[Портал](#tab/portal)

Чтобы просмотреть области шифрования для учетной записи хранения в портал Azure, перейдите к параметру **области шифрования** для учетной записи хранения. В этой области можно включить или отключить область шифрования или изменить ключ для области шифрования.

:::image type="content" source="media/encryption-scope-manage/list-encryption-scopes-portal.png" alt-text="Снимок экрана, показывающий список областей шифрования в портал Azure":::

Чтобы просмотреть сведения о ключе, управляемом клиентом, включая URI ключа и версию, а также автоматическое обновление ключа, перейдите по ссылке в **ключевом** столбце.

:::image type="content" source="media/encryption-scope-manage/customer-managed-key-details-portal.png" alt-text="Снимок экрана, показывающий сведения о ключе, используемом в области шифрования":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы получить список областей шифрования, доступных для учетной записи хранения с помощью PowerShell, вызовите команду **Get-азсторажеенкриптионскопе** . Не забудьте заменить значения заполнителей в примере собственными значениями:

```powershell
Get-AzStorageEncryptionScope -ResourceGroupName $rgName `
    -StorageAccountName $accountName
```

Чтобы получить список всех областей шифрования в группе ресурсов по учетной записи хранения, используйте синтаксис конвейера:

```powershell
Get-AzStorageAccount -ResourceGroupName $rgName | Get-AzStorageEncryptionScope
```

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Чтобы получить список областей шифрования, доступных для учетной записи хранения с Azure CLI, вызовите команду [AZ Storage Account Encryption-Scope List](/cli/azure/storage/account/encryption-scope#az-storage-account-encryption-scope-list) . Не забудьте заменить значения заполнителей в примере собственными значениями:

```azurecli-interactive
az storage account encryption-scope list \
    --account-name <storage-account> \
    --resource-group <resource-group>
```

---

## <a name="create-a-container-with-a-default-encryption-scope"></a>Создание контейнера с областью шифрования по умолчанию

При создании контейнера можно указать область шифрования по умолчанию. Большие двоичные объекты в этом контейнере будут использовать эту область по умолчанию.

Можно создать отдельный большой двоичный объект с собственной областью шифрования, если только контейнер не настроен так, чтобы все большие двоичные объекты использовали область по умолчанию. Дополнительные сведения см. в разделе [области шифрования для контейнеров и больших двоичных объектов](encryption-scope-overview.md#encryption-scopes-for-containers-and-blobs).

# <a name="portal"></a>[Портал](#tab/portal)

Чтобы создать контейнер с областью шифрования по умолчанию в портал Azure, сначала создайте область шифрования, как описано в разделе [Создание области шифрования](#create-an-encryption-scope). Затем выполните следующие действия, чтобы создать контейнер:

1. Перейдите к списку контейнеров в учетной записи хранения и нажмите кнопку **Добавить** , чтобы создать новый контейнер.
1. Разверните **Дополнительные** параметры на панели **новый контейнер** .
1. В раскрывающемся списке **область шифрования** выберите область шифрования по умолчанию для контейнера.
1. Чтобы указать, что все большие двоичные объекты в контейнере используют область шифрования по умолчанию, установите флажок, чтобы **использовать эту область шифрования для всех больших двоичных объектов в контейнере**. Если этот флажок установлен, то отдельный большой двоичный объект в контейнере не может переопределить область шифрования по умолчанию.

    :::image type="content" source="media/encryption-scope-manage/create-container-default-encryption-scope.png" alt-text="Снимок экрана, показывающий контейнер с областью шифрования по умолчанию":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы создать контейнер с областью шифрования по умолчанию с помощью PowerShell, вызовите команду [New-азсторажеконтаинер](/powershell/module/az.storage/new-azstoragecontainer) , указав область для `-DefaultEncryptionScope` параметра. Чтобы принудительно использовать область по умолчанию контейнера для всех больших двоичных объектов в контейнере, задайте `-PreventEncryptionScopeOverride` для параметра значение `true` .

```powershell
$containerName1 = "container1"
$ctx = New-AzStorageContext -StorageAccountName $accountName -UseConnectedAccount

# Create a container with a default encryption scope that cannot be overridden.
New-AzStorageContainer -Name $containerName1 `
    -Context $ctx `
    -DefaultEncryptionScope $scopeName1 `
    -PreventEncryptionScopeOverride $true
```

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Чтобы создать контейнер с областью шифрования по умолчанию с Azure CLI, вызовите команду [AZ Storage Container Create](/cli/azure/storage/container#az-storage-container-create) , указав область для `--default-encryption-scope` параметра. Чтобы принудительно использовать область по умолчанию контейнера для всех больших двоичных объектов в контейнере, задайте `--prevent-encryption-scope-override` для параметра значение `true` .

В следующем примере учетная запись Azure AD используется для авторизации операции создания контейнера. Вы также можете использовать ключ доступа к учетной записи. Дополнительные сведения см. в статье [Авторизация доступа к данным BLOB-объектов или очередей с помощью Azure CLI](./authorize-data-operations-cli.md).

```azurecli-interactive
az storage container create \
    --account-name <storage-account> \
    --resource-group <resource-group> \
    --name <container> \
    --default-encryption-scope <scope> \
    --prevent-encryption-scope-override true \
    --auth-mode login
```

---

Если клиент пытается указать область при отправке большого двоичного объекта в контейнер с областью шифрования по умолчанию, а контейнер настроен так, чтобы предотвратить переопределение области по умолчанию для больших двоичных объектов, то операция завершается с сообщением о том, что запрос запрещен политикой шифрования контейнеров.

## <a name="upload-a-blob-with-an-encryption-scope"></a>Отправка большого двоичного объекта с областью шифрования

При передаче большого двоичного объекта можно указать область шифрования для этого большого двоичного объекта или использовать область шифрования по умолчанию для контейнера, если он был указан.

# <a name="portal"></a>[Портал](#tab/portal)

Чтобы передать большой двоичный объект с областью шифрования с помощью портал Azure, сначала создайте область шифрования, как описано в разделе [Создание области шифрования](#create-an-encryption-scope). Затем выполните следующие действия, чтобы создать большой двоичный объект:

1. Перейдите к контейнеру, в который необходимо передать большой двоичный объект.
1. Нажмите кнопку **Отправить** и выберите большой двоичный объект для отправки.
1. Разверните **Дополнительные** параметры в области **Отправить BLOB-объект** .
1. Перейдите к раскрывающийся список **область шифрования** . По умолчанию большой двоичный объект создается с областью шифрования по умолчанию для контейнера, если он указан. Если контейнер требует, чтобы большие двоичные объекты использовали область шифрования по умолчанию, этот раздел будет отключен.
1. Чтобы указать другую область для загружаемого большого двоичного объекта, выберите **выбрать существующую область**, а затем выберите нужную область из раскрывающегося списка.

    :::image type="content" source="media/encryption-scope-manage/upload-blob-encryption-scope.png" alt-text="Снимок экрана, показывающий, как передать большой двоичный объект с областью шифрования":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы передать большой двоичный объект с областью шифрования с помощью PowerShell, вызовите команду [Set-азсторажеблобконтент](/powershell/module/az.storage/set-azstorageblobcontent) и укажите область шифрования для большого двоичного объекта.

```powershell
$containerName2 = "container2"
$localSrcFile = "C:\temp\helloworld.txt"
$ctx = New-AzStorageContext -StorageAccountName $accountName -UseConnectedAccount

# Create a new container with no default scope defined.
New-AzStorageContainer -Name $containerName2 -Context $ctx

# Upload a block upload with an encryption scope specified.
Set-AzStorageBlobContent -Context $ctx `
    -Container $containerName2 `
    -File $localSrcFile `
    -Blob "helloworld.txt" `
    -BlobType Block `
    -EncryptionScope $scopeName2
```

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Чтобы передать большой двоичный объект с областью шифрования с помощью Azure CLI, вызовите команду [AZ Storage BLOB upload](/cli/azure/storage/blob#az-storage-blob-upload) и укажите область шифрования для большого двоичного объекта.

Если вы используете Azure Cloud Shell, выполните действия, описанные в разделе [Отправка BLOB-объекта](storage-quickstart-blobs-cli.md#upload-a-blob) , чтобы создать файл в корневом каталоге. Затем этот файл можно передать в большой двоичный объект, используя следующий пример.

```azurecli-interactive
az storage blob upload \
    --account-name <storage-account> \
    --container-name <container> \
    --file <file> \
    --name <file> \
    --encryption-scope <scope>
```

---

## <a name="change-the-encryption-key-for-a-scope"></a>Изменение ключа шифрования для области

Чтобы изменить ключ, защищающий область шифрования от ключа, управляемого Майкрософт, к ключу, управляемому клиентом, сначала убедитесь, что вы включили ключи, управляемые клиентом, с Azure Key Vault или Key Vault HSM для учетной записи хранения. Дополнительные сведения см. в статьях [Настройка шифрования с помощью управляемых клиентом ключей, хранящихся в Azure Key Vault](../common/customer-managed-keys-configure-key-vault.md) или [Настройка шифрования с помощью управляемых клиентом ключей, хранящихся в Azure Key Vault](../common/customer-managed-keys-configure-key-vault.md).

# <a name="portal"></a>[Портал](#tab/portal)

Чтобы изменить ключ, защищающий область в портал Azure, выполните следующие действия.

1. Перейдите на вкладку **области шифрования** , чтобы просмотреть список областей шифрования для учетной записи хранения.
1. Нажмите кнопку **Дополнительно** рядом с областью, которую вы хотите изменить.
1. На панели **Изменение области шифрования** можно изменить тип шифрования ключа, управляемого корпорацией Майкрософт, на ключ, управляемый клиентом, или наоборот.
1. Чтобы выбрать новый ключ, управляемый клиентом, выберите **использовать новый ключ** и укажите хранилище ключей, ключ и версию ключа.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы изменить ключ, защищающий область шифрования от ключа, управляемого клиентом, к ключу, управляемому корпорацией Майкрософт, с помощью PowerShell, вызовите команду **Update-азсторажеенкриптионскопе** и передайте `-StorageEncryption` параметр:

```powershell
Update-AzStorageEncryptionScope -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -EncryptionScopeName $scopeName2 `
    -StorageEncryption
```

Затем вызовите команду **Update-азсторажеенкриптионскопе** и передайте `-KeyUri` `-KeyvaultEncryption` Параметры и:

```powershell
Update-AzStorageEncryptionScope -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -EncryptionScopeName $scopeName1 `
    -KeyUri $keyUri `
    -KeyvaultEncryption
```

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Чтобы изменить ключ, защищающий область шифрования от ключа, управляемого клиентом, до ключа, управляемого корпорацией Майкрософт, с Azure CLI, вызовите команду [AZ Storage Account Encryption-Scope Update](/cli/azure/storage/account/encryption-scope#az-storage-account-encryption-scope-update) и передайте `--key-source` параметр со значением `Microsoft.Storage` :

```azurecli-interactive
az storage account encryption-scope update \
    --account-name <storage-account> \
    --resource-group <resource-group>
    --name <scope> \
    --key-source Microsoft.Storage
```

Затем вызовите команду **AZ Storage Account Encryption-Scope Update** , передайте `--key-uri` параметр и передайте `--key-source` параметр со значением `Microsoft.KeyVault` :

```powershell
az storage account encryption-scope update \
    --resource-group <resource-group> \
    --account-name <storage-account> \
    --name <scope> \
    --key-source Microsoft.KeyVault \
    --key-uri <key-uri>
```

---

## <a name="disable-an-encryption-scope"></a>Отключение области шифрования

Если область шифрования отключена, плата за нее больше не взимается. Отключите все области шифрования, которые не требуются, чтобы избежать ненужных расходов. Дополнительные сведения см. в статье [Шифрование службы хранилища Azure для неактивных данных](../common/storage-service-encryption.md).

# <a name="portal"></a>[Портал](#tab/portal)

Чтобы отключить область шифрования в портал Azure, перейдите к параметру **области шифрования** для учетной записи хранения, выберите нужную область шифрования и нажмите кнопку **Отключить**.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы отключить область шифрования с помощью PowerShell, вызовите команду Update-AzStorageEncryptionScope и включите `-State` параметр со значением `disabled` , как показано в следующем примере. Чтобы повторно включить область шифрования, вызовите ту же команду с `-State` параметром, для которого задано значение `enabled` . Не забудьте заменить значения заполнителей в примере собственными значениями:

```powershell
Update-AzStorageEncryptionScope -ResourceGroupName $rgName `
    -StorageAccountName $accountName `
    -EncryptionScopeName $scopeName1 `
    -State disabled
```

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Чтобы отключить область шифрования с помощью Azure CLI, вызовите команду [AZ Storage Account Encryption-Scope Update](/cli/azure/storage/account/encryption-scope#az-storage-account-encryption-scope-update) , включив `--state` параметр со значением `Disabled` , как показано в следующем примере. Чтобы повторно включить область шифрования, вызовите ту же команду с `--state` параметром, для которого задано значение `Enabled` . Не забудьте заменить значения заполнителей в примере собственными значениями:

```azurecli-interactive
az storage account encryption-scope update \
    --account-name <storage-account> \
    --resource-group <resource-group> \
    --name <scope> \
    --state Disabled
```

> [!IMPORTANT]
> Невозможно удалить область шифрования. Чтобы избежать непредвиденных затрат, не забудьте отключить все области шифрования, которые в настоящее время не нужны.

---

## <a name="next-steps"></a>Дальнейшие шаги

- [Шифрование службы хранилища Azure для неактивных данных](../common/storage-service-encryption.md)
- [Области шифрования для хранилища BLOB-объектов](encryption-scope-overview.md)
- [Управляемые клиентом ключи для шифрования службы хранилища Azure](../common/customer-managed-keys-overview.md)