---
title: Настройка шифрования с помощью управляемых клиентом ключей, хранящихся в Azure Key Vault
titleSuffix: Azure Storage
description: Узнайте, как настроить шифрование службы хранилища Azure с использованием управляемых клиентом ключей, хранящихся в Azure Key Vault с помощью портал Azure, PowerShell или Azure CLI.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 02/16/2021
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 24fbe843986b732a04c9e356c54f3d768d6739be
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100558183"
---
# <a name="configure-encryption-with-customer-managed-keys-stored-in-azure-key-vault"></a>Настройка шифрования с помощью управляемых клиентом ключей, хранящихся в Azure Key Vault

Служба хранилища Azure шифрует все данные в неактивных учетных записях хранения. По умолчанию данные шифруются с помощью ключей, управляемых корпорацией Майкрософт. Для дополнительного контроля над ключами шифрования можно управлять собственными ключами. Ключи, управляемые клиентом, должны храниться в Azure Key Vault или Key Vault управляемой аппаратной модели безопасности (HSM) (в предварительной версии).

В этой статье показано, как настроить шифрование с помощью управляемых клиентом ключей, хранящихся в хранилище ключей, с использованием портал Azure, PowerShell или Azure CLI. Сведения о настройке шифрования с помощью управляемых клиентом ключей, хранящихся в управляемом HSM, см. в статье [Настройка шифрования с помощью управляемых клиентом ключей, хранящихся в Azure Key Vault управляемый HSM (Предварительная версия)](customer-managed-keys-configure-key-vault-hsm.md).

> [!NOTE]
> Управляемый модуль HSM Azure Key Vault и Azure Key Vault поддерживают те же интерфейсы API и интерфейсы управления для конфигурации.

## <a name="configure-a-key-vault"></a>Настройка хранилища ключей

Для хранения ключей, управляемых клиентом, можно использовать новое или существующее хранилище ключей. Учетная запись хранения и Key Vault должны быть расположены в одном регионе, но могут находиться в разных подписках.

Для использования управляемых клиентом ключей с шифрованием службы хранилища Azure требуется включить для хранилища ключей как обратимое удаление, так и очистку. Обратимое удаление включено по умолчанию при создании нового хранилища ключей и не может быть отключено. Защиту от очистки можно включить при создании хранилища ключей или после его создания.

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Сведения о создании хранилища ключей с портал Azure см. в разделе Краткое руководство [. Создание хранилища ключей с помощью портал Azure](../../key-vault/general/quick-create-portal.md). При создании хранилища ключей выберите **включить защиту от удаления**, как показано на следующем рисунке.

:::image type="content" source="media/customer-managed-keys-configure-key-vault/configure-key-vault-portal.png" alt-text="Снимок экрана, показывающий, как включить защиту от очистки при создании хранилища ключей":::

Чтобы включить защиту от очистки в существующем хранилище ключей, выполните следующие действия.

1. Перейдите к хранилищу ключей в портал Azure.
1. В разделе **Параметры** выберите **свойства**.
1. В разделе **Защита от вирусов** выберите **включить защиту от очистки**.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы создать новое хранилище ключей с помощью PowerShell, установите версию 2.0.0 или более позднюю версию модуля PowerShell [AZ. KeyVault](https://www.powershellgallery.com/packages/Az.KeyVault/2.0.0) . Затем вызовите [New-азкэйваулт](/powershell/module/az.keyvault/new-azkeyvault) , чтобы создать новое хранилище ключей. В версии 2.0.0 и более поздних версиях модуля AZ. KeyVault обратимое удаление включено по умолчанию при создании нового хранилища ключей.

В следующем примере создается новое хранилище ключей с включенной защитой обратимого удаления и очистки. Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями.

```powershell
$keyVault = New-AzKeyVault -Name <key-vault> `
    -ResourceGroupName <resource_group> `
    -Location <location> `
    -EnablePurgeProtection
```

Сведения о том, как включить защиту для очистки в существующем хранилище ключей с помощью PowerShell, см. в статье [Использование обратимого удаления с помощью PowerShell](../../key-vault/general/key-vault-recovery.md).

Затем назначьте вашей учетной записи хранения управляемое удостоверение, назначенное системой. Вы будете использовать это управляемое удостоверение, чтобы предоставить учетной записи хранения разрешения на доступ к хранилищу ключей. Дополнительные сведения о назначенных системой удостоверениях см. в статье [что такое управляемые удостоверения для ресурсов Azure?](../../active-directory/managed-identities-azure-resources/overview.md).

Чтобы назначить управляемое удостоверение с помощью PowerShell, вызовите [Set-азсторажеаккаунт](/powershell/module/az.storage/set-azstorageaccount):

```powershell
$storageAccount = Set-AzStorageAccount -ResourceGroupName <resource_group> `
    -Name <storage-account> `
    -AssignIdentity
```

Наконец, настройте политику доступа для хранилища ключей, чтобы у учетной записи хранения были разрешения на доступ к ней. На этом шаге вы будете использовать управляемое удостоверение, назначенное ранее учетной записи хранения.

Чтобы задать политику доступа для хранилища ключей, вызовите [Set-азкэйваултакцессполици](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy):

```powershell
Set-AzKeyVaultAccessPolicy `
    -VaultName $keyVault.VaultName `
    -ObjectId $storageAccount.Identity.PrincipalId `
    -PermissionsToKeys wrapkey,unwrapkey,get
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы создать новое хранилище ключей с помощью Azure CLI, вызовите команду [AZ keyvault Create](/cli/azure/keyvault#az-keyvault-create). Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями:

```azurecli-interactive
az keyvault create \
    --name <key-vault> \
    --resource-group <resource_group> \
    --location <region> \
    --enable-purge-protection
```

Сведения о включении защиты от очистки в существующем хранилище ключей с Azure CLI см. в статье [Использование обратимого удаления с помощью интерфейса командной строки](../../key-vault/general/key-vault-recovery.md).

Затем назначьте учетной записи хранения управляемый идентификатор, назначенный системой. Вы будете использовать это управляемое удостоверение, чтобы предоставить учетной записи хранения разрешения на доступ к хранилищу ключей. Дополнительные сведения о назначенных системой удостоверениях см. в статье [что такое управляемые удостоверения для ресурсов Azure?](../../active-directory/managed-identities-azure-resources/overview.md).

Чтобы назначить управляемое удостоверение с помощью Azure CLI, вызовите команду [AZ Storage учетная запись Update](/cli/azure/storage/account#az-storage-account-update):

```azurecli-interactive
az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --assign-identity
```

Наконец, настройте политику доступа для хранилища ключей, чтобы у учетной записи хранения были разрешения на доступ к ней. На этом шаге вы будете использовать управляемое удостоверение, назначенное ранее учетной записи хранения.

Чтобы задать политику доступа для хранилища ключей, вызовите команду [AZ keyvault Set-Policy](/cli/azure/keyvault#az-keyvault-set-policy):

```azurecli-interactive
storage_account_principal=$(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query identity.principalId \
    --output tsv)
az keyvault set-policy \
    --name <key-vault> \
    --resource-group <resource_group>
    --object-id $storage_account_principal \
    --key-permissions get unwrapKey wrapKey
```

---

## <a name="add-a-key"></a>Добавление ключа

Затем добавьте ключ в хранилище ключей.

Шифрование службы хранилища Azure поддерживает ключи RSA и RSA-HSM размером 2048, 3072 и 4096. Дополнительные сведения о ключах см. в разделе [о ключах](../../key-vault/keys/about-keys.md).

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Дополнительные сведения о добавлении ключа с портал Azure см. в разделе [Краткое руководство. задание и получение ключа из Azure Key Vault с помощью портал Azure](../../key-vault/keys/quick-create-portal.md).

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы добавить ключ с помощью PowerShell, вызовите [Add-азкэйваулткэй](/powershell/module/az.keyvault/add-azkeyvaultkey). Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями и использовать переменные, определенные в предыдущих примерах.

```powershell
$key = Add-AzKeyVaultKey -VaultName $keyVault.VaultName `
    -Name <key> `
    -Destination 'Software'
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы добавить ключ с Azure CLI, вызовите команду [AZ keyvault Key Create](/cli/azure/keyvault/key#az-keyvault-key-create). Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями.

```azurecli-interactive
az keyvault key create \
    --name <key> \
    --vault-name <key-vault>
```

---

## <a name="configure-encryption-with-customer-managed-keys"></a>Настройка шифрования с использованием ключей, управляемых клиентом

Затем настройте учетную запись хранения Azure для использования ключей, управляемых клиентом, с Azure Key Vault, а затем укажите ключ, связываемый с учетной записью хранения.

При настройке шифрования с помощью управляемых клиентом ключей можно автоматически обновлять ключ, используемый для шифрования службы хранилища Azure, когда в связанном хранилище ключей будет доступна новая версия. Кроме того, можно явно указать версию ключа, которая будет использоваться для шифрования до тех пор, пока не будет обновлена версия ключа вручную.

> [!NOTE]
> Чтобы повернуть ключ, создайте новую версию ключа в Azure Key Vault. Служба хранилища Azure не обрабатывает вращение ключа в Azure Key Vault, поэтому вам потребуется вручную сменить ключ или создать функцию, чтобы ее можно было поворачивать по расписанию.

### <a name="configure-encryption-for-automatic-updating-of-key-versions"></a>Настройка шифрования для автоматического обновления основных версий

Служба хранилища Azure может автоматически обновить управляемый клиентом ключ, используемый для шифрования, чтобы использовать последнюю версию ключа. Когда ключ, управляемый клиентом, поворачивается в Azure Key Vault, служба хранилища Azure автоматически начинает использовать последнюю версию ключа для шифрования.

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Чтобы настроить ключи, управляемые клиентом, с помощью автоматического обновления ключа версии в портал Azure выполните следующие действия.

1. Войдите в свою учетную запись хранения.
1. В колоне **параметров** для учетной записи хранения выберите **Шифрование**. По умолчанию для управления ключами устанавливаются **ключи, управляемые корпорацией Майкрософт**, как показано на следующем рисунке.

    ![Снимок экрана портала с параметром шифрования](./media/customer-managed-keys-configure-key-vault/portal-configure-encryption-keys.png)

1. Выберите параметр " **управляемые ключи клиента** ".
1. Выберите параметр **Выбрать в Key Vault**.
1. Выберите **выбрать хранилище ключей и ключ**.
1. Выберите хранилище ключей, содержащее ключ, который вы хотите использовать.
1. Выберите ключ из хранилища ключей.

   ![Снимок экрана, показывающий, как выбрать хранилище ключей и ключ](./media/customer-managed-keys-configure-key-vault/portal-select-key-from-key-vault.png)

1. Сохраните изменения.

После указания ключа портал Azure указывает, что автоматическое обновление версии ключа включено и отображает текущую версию ключа, используемую для шифрования.

:::image type="content" source="media/customer-managed-keys-configure-key-vault/portal-auto-rotation-enabled.png" alt-text="Снимок экрана с включенным автоматическим обновлением версии ключа":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы настроить ключи, управляемые клиентом, с помощью автоматического обновления ключа версии с помощью PowerShell, установите модуль [AZ. Storage](https://www.powershellgallery.com/packages/Az.Storage) , версия 2.0.0 или более поздняя.

Чтобы автоматически обновить ключ для управляемого клиентом ключа, не указывайте версию ключа при настройке шифрования с помощью управляемых клиентом ключей для учетной записи хранения. Вызовите [Set-азсторажеаккаунт](/powershell/module/az.storage/set-azstorageaccount) , чтобы обновить параметры шифрования учетной записи хранения, как показано в следующем примере, и включите параметр **-кэйваултенкриптион** , чтобы включить управляемые клиентом ключи для учетной записи хранения.

Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями и использовать переменные, определенные в предыдущих примерах.

```powershell
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -KeyvaultEncryption `
    -KeyName $key.Name `
    -KeyVaultUri $keyVault.VaultUri
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы настроить ключи, управляемые клиентом, с помощью автоматического обновления ключа версии с Azure CLI, установите [Azure CLI версии 2.4.0](/cli/azure/release-notes-azure-cli#april-21-2020) или более поздней. Дополнительные сведения см. в статье [Установка Azure CLI](/cli/azure/install-azure-cli).

Чтобы автоматически обновить ключ для управляемого клиентом ключа, не указывайте версию ключа при настройке шифрования с помощью управляемых клиентом ключей для учетной записи хранения. Чтобы обновить параметры шифрования учетной записи хранения, вызовите команду [AZ Storage Account Update](/cli/azure/storage/account#az-storage-account-update) , как показано в следующем примере. Включите `--encryption-key-source` параметр и установите его в значение, `Microsoft.Keyvault` чтобы включить управляемые клиентом ключи для учетной записи.

Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями.

```azurecli-interactive
key_vault_uri=$(az keyvault show \
    --name <key-vault> \
    --resource-group <resource_group> \
    --query properties.vaultUri \
    --output tsv)
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $key_vault_uri
```

---

### <a name="configure-encryption-for-manual-updating-of-key-versions"></a>Настройка шифрования для обновления основных версий вручную

Если вы предпочитаете вручную обновить версию ключа, необходимо явно указать ее версию во время настройки шифрования с помощью управляемых клиентом ключей. В этом случае служба хранилища Azure не будет автоматически обновлять версию ключа при создании новой версии в хранилище ключей. Чтобы использовать новую версию ключа, необходимо вручную обновить версию, используемую для шифрования службы хранилища Azure.

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Чтобы настроить ключи, управляемые клиентом, вручную обновив ключ версии в портал Azure, укажите универсальный код ресурса (URI) ключа, включая версию. Чтобы указать ключ в качестве универсального кода ресурса (URI), выполните следующие действия.

1. Чтобы найти URI ключа в портал Azure, перейдите в хранилище ключей и выберите параметр **ключи** . Выберите нужный ключ, а затем щелкните ключ, чтобы просмотреть его версии. Выберите версию ключа, чтобы просмотреть параметры для этой версии.
1. Скопируйте значение поля **идентификатор ключа** , которое предоставляет универсальный код ресурса (URI).

    ![Снимок экрана, показывающий URI ключа хранилища ключей](media/customer-managed-keys-configure-key-vault/portal-copy-key-identifier.png)

1. В параметрах **ключа шифрования** для учетной записи хранения выберите параметр **Введите URI ключа** .
1. Вставьте URI, скопированный в поле **ключа URI** . Не указывайте версию ключа из URI, чтобы включить автоматическое обновление версии ключа.

   ![Снимок экрана, показывающий, как ввести URI ключа](./media/customer-managed-keys-configure-key-vault/portal-specify-key-uri.png)

1. Укажите подписку, которая содержит хранилище ключей.
1. Сохраните изменения.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы настроить ключи, управляемые клиентом, с обновлением версии ключа вручную, необходимо явно указать версию ключа при настройке шифрования для учетной записи хранения. Вызовите [Set-азсторажеаккаунт](/powershell/module/az.storage/set-azstorageaccount) , чтобы обновить параметры шифрования учетной записи хранения, как показано в следующем примере, и включите параметр **-кэйваултенкриптион** , чтобы включить управляемые клиентом ключи для учетной записи хранения.

Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями и использовать переменные, определенные в предыдущих примерах.

```powershell
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -KeyvaultEncryption `
    -KeyName $key.Name `
    -KeyVersion $key.Version `
    -KeyVaultUri $keyVault.VaultUri
```

При ручном обновлении версии ключа необходимо обновить параметры шифрования учетной записи хранения, чтобы использовать новую версию. Сначала вызовите [Get-азкэйваулткэй](/powershell/module/az.keyvault/get-azkeyvaultkey) , чтобы получить последнюю версию ключа. Затем вызовите [Set-азсторажеаккаунт](/powershell/module/az.storage/set-azstorageaccount) , чтобы обновить параметры шифрования учетной записи хранения для использования новой версии ключа, как показано в предыдущем примере.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы настроить ключи, управляемые клиентом, с обновлением версии ключа вручную, необходимо явно указать версию ключа при настройке шифрования для учетной записи хранения. Чтобы обновить параметры шифрования учетной записи хранения, вызовите команду [AZ Storage Account Update](/cli/azure/storage/account#az-storage-account-update) , как показано в следующем примере. Включите `--encryption-key-source` параметр и установите его в значение, `Microsoft.Keyvault` чтобы включить управляемые клиентом ключи для учетной записи.

Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями.

```azurecli-interactive
key_vault_uri=$(az keyvault show \
    --name <key-vault> \
    --resource-group <resource_group> \
    --query properties.vaultUri \
    --output tsv)
key_version=$(az keyvault key list-versions \
    --name <key> \
    --vault-name <key-vault> \
    --query [-1].kid \
    --output tsv | cut -d '/' -f 6)
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-version $key_version \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $key_vault_uri
```

При ручном обновлении версии ключа необходимо обновить параметры шифрования учетной записи хранения, чтобы использовать новую версию. Сначала запросите универсальный код ресурса (URI) хранилища ключей, вызвав команду [AZ keyvault показывать](/cli/azure/keyvault#az-keyvault-show), а для версии key [— вызов AZ keyvault Key List-Versions](/cli/azure/keyvault/key#az-keyvault-key-list-versions). Затем вызовите команду [AZ Storage Account Update](/cli/azure/storage/account#az-storage-account-update) , чтобы обновить параметры шифрования учетной записи хранения для использования новой версии ключа, как показано в предыдущем примере.

---

## <a name="change-the-key"></a>Изменение ключа

Ключ, который вы используете для шифрования службы хранилища Azure, можно изменить в любое время.

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Чтобы изменить ключ на портал Azure, выполните следующие действия.

1. Перейдите к своей учетной записи хранения и отобразите параметры **шифрования** .
1. Выберите хранилище ключей и выберите новый ключ.
1. Сохраните изменения.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы изменить ключ с помощью PowerShell, вызовите [Set-азсторажеаккаунт](/powershell/module/az.storage/set-azstorageaccount) , как показано в разделе [Настройка шифрования с помощью управляемых клиентом ключей](#configure-encryption-with-customer-managed-keys) и предоставление нового имени и версии ключа. Если новый ключ находится в другом хранилище ключей, необходимо также обновить URI хранилища ключей.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы изменить ключ с Azure CLI, вызовите команду [AZ Storage Account Update](/cli/azure/storage/account#az-storage-account-update) , как показано в разделе [Настройка шифрования с помощью управляемых клиентом ключей](#configure-encryption-with-customer-managed-keys) и предоставление нового имени и версии ключа. Если новый ключ находится в другом хранилище ключей, необходимо также обновить URI хранилища ключей.

---

## <a name="revoke-customer-managed-keys"></a>Отозвать ключи, управляемые клиентом

При отзыве ключа, управляемого клиентом, удаляется связь между учетной записью хранения и хранилищем ключей.

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Чтобы отозвать ключи, управляемые клиентом, с помощью портал Azure отключите ключ, как описано в разделе [Отключение управляемых клиентом ключей](#disable-customer-managed-keys).

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Вы можете отозвать ключи, управляемые клиентом, удалив политику доступа к хранилищу ключей. Чтобы отозвать ключ, управляемый клиентом, с помощью PowerShell, вызовите команду [Remove-азкэйваултакцессполици](/powershell/module/az.keyvault/remove-azkeyvaultaccesspolicy) , как показано в следующем примере. Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями и использовать переменные, определенные в предыдущих примерах.

```powershell
Remove-AzKeyVaultAccessPolicy -VaultName $keyVault.VaultName `
    -ObjectId $storageAccount.Identity.PrincipalId `
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Вы можете отозвать ключи, управляемые клиентом, удалив политику доступа к хранилищу ключей. Чтобы отозвать ключ, управляемый клиентом, с Azure CLI, вызовите команду [AZ keyvault Delete-Policy](/cli/azure/keyvault#az-keyvault-delete-policy) , как показано в следующем примере. Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями и использовать переменные, определенные в предыдущих примерах.

```azurecli-interactive
az keyvault delete-policy \
    --name <key-vault> \
    --object-id $storage_account_principal
```

---

## <a name="disable-customer-managed-keys"></a>Отключение ключей, управляемых клиентом

При отключении управляемых пользователем ключей учетная запись хранения снова шифруется с помощью ключей, управляемых корпорацией Майкрософт.

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Чтобы отключить управляемые клиентом ключи в портал Azure, выполните следующие действия.

1. Перейдите к своей учетной записи хранения и отобразите параметры **шифрования** .
1. Снимите флажок рядом с параметром **использовать собственный ключ** .

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы отключить управляемые клиентом ключи с помощью PowerShell, вызовите [Set-азсторажеаккаунт](/powershell/module/az.storage/set-azstorageaccount) с `-StorageEncryption` параметром, как показано в следующем примере. Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями и использовать переменные, определенные в предыдущих примерах.

```powershell
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -StorageEncryption  
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы отключить управляемые клиентом ключи с помощью Azure CLI, вызовите команду [AZ Storage Account Update](/cli/azure/storage/account#az-storage-account-update) и задайте `--encryption-key-source parameter` для значение `Microsoft.Storage` , как показано в следующем примере. Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями и использовать переменные, определенные в предыдущих примерах.

```azurecli-interactive
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-source Microsoft.Storage
```

---

## <a name="next-steps"></a>Дальнейшие действия

- [Шифрование службы хранилища Azure для неактивных данных](storage-service-encryption.md)
- [Управляемые клиентом ключи для шифрования службы хранилища Azure](customer-managed-keys-overview.md)
- [Настройка шифрования с помощью управляемых клиентом ключей, хранящихся в Azure Key Vault управляемом HSM (Предварительная версия)](customer-managed-keys-configure-key-vault-hsm.md)
