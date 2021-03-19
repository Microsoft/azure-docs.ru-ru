---
title: Восстановление Key Vault ключа & секрета для зашифрованной виртуальной машины
description: Узнайте, как восстановить ключ и секрет хранилища ключей в службе архивации Azure с помощью PowerShell
ms.topic: conceptual
ms.date: 08/28/2017
ms.openlocfilehash: 456ce18f253ffa02cd6b13826a7839f18beecba7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88827092"
---
# <a name="restore-key-vault-key-and-secret-for-encrypted-vms-using-azure-backup"></a>Восстановление ключа и секрета в хранилище ключей для зашифрованных виртуальных машин с помощью службы архивации Azure

В этой статье рассказывается, как использовать резервное копирование ВИРТУАЛЬНЫХ машин Azure для восстановления зашифрованных виртуальных машин Azure, если ключ и секрет не существуют в хранилище ключей. Эти действия также можно использовать, если требуется сохранить отдельную копию ключа (ключа шифрования ключа) и секрет (ключ шифрования BitLocker) для восстановленной виртуальной машины.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

* **Резервная копия зашифрованных виртуальных машин** — выполнять резервное копирование зашифрованных виртуальных машин Azure нужно с помощью службы архивации Azure. Дополнительные сведения о резервном копировании зашифрованных виртуальных машин Azure см. в статье [Управление резервным копированием и восстановлением виртуальных машин Azure с помощью PowerShell](backup-azure-vms-automation.md) .
* **Настроенное хранилище ключей Azure** — убедитесь, что хранилище ключей, в которое нужно восстановить ключи и секреты, существует. Дополнительные сведения об управлении хранилищем ключей см. в статье [Начало работы с Azure Key Vault](../key-vault/general/overview.md) .
* **Восстановление диска** . Убедитесь, что вы активировали задание восстановления для восстановления дисков для ЗАШИФРОВАННЫХ виртуальных машин с помощью [действий PowerShell](backup-azure-vms-automation.md#restore-an-azure-vm). Это связано с тем, что данное задание создает JSON-файл в учетной записи хранения, которая содержит ключи и секреты для восстанавливаемой зашифрованной виртуальной машины.

## <a name="get-key-and-secret-from-azure-backup"></a>Получение ключа и секрета из службы Azure Backup

> [!NOTE]
> После восстановления диска для зашифрованной виртуальной машины убедитесь, что выполняются следующие требования:
>
> * Переменная $details заполняется сведениями о задании восстановления диска, как описано в [разделе о восстановлении дисков с помощью PowerShell](backup-azure-vms-automation.md#restore-an-azure-vm).
> * Создавать виртуальную машину из восстановленных дисков следует только **после восстановления ключа и секрета в хранилище ключей**.

Запросите свойства восстановленного диска для получения сведений о задании.

```powershell
$properties = $details.properties
$storageAccountName = $properties["Target Storage Account Name"]
$containerName = $properties["Config Blob Container Name"]
$encryptedBlobName = $properties["Encryption Info Blob Name"]
```

Задайте контекст хранилища Azure и восстановите файл конфигурации JSON, который содержит ключ и секрет для зашифрованной виртуальной машины.

```powershell
Set-AzCurrentStorageAccount -Name $storageaccountname -ResourceGroupName '<rg-name>'
$destination_path = 'C:\vmencryption_config.json'
Get-AzStorageBlobContent -Blob $encryptedBlobName -Container $containerName -Destination $destination_path
$encryptionObject = Get-Content -Path $destination_path  | ConvertFrom-Json
```

## <a name="restore-key"></a>Ключ восстановления

Когда JSON-файл создан в упомянутом выше конечном пути, создайте из этого JSON-файла ключ в виде файла большого двоичного объекта и укажите его при выполнении командлета восстановления ключа, чтобы поместить ключ (KEK) обратно в хранилище ключей.

```powershell
$keyDestination = 'C:\keyDetails.blob'
[io.file]::WriteAllBytes($keyDestination, [System.Convert]::FromBase64String($encryptionObject.OsDiskKeyAndSecretDetails.KeyBackupData))
Restore-AzureKeyVaultKey -VaultName '<target_key_vault_name>' -InputFile $keyDestination
```

## <a name="restore-secret"></a>Восстановление секрета

Используйте созданный выше JSON-файл для получения имени и значения секрета и укажите его при выполнении командлета настройки секрета, чтобы поместить секрет (BEK) обратно в хранилище ключей. Используйте эти командлеты, если ваша **Виртуальная машина зашифрована с помощью BEK и KEK**.

**Воспользуйтесь этими командлетами, если виртуальная машина Windows зашифрована с помощью BEK и KEK.**

```powershell
$secretdata = $encryptionObject.OsDiskKeyAndSecretDetails.SecretData
$Secret = ConvertTo-SecureString -String $secretdata -AsPlainText -Force
$secretname = 'B3284AAA-DAAA-4AAA-B393-60CAA848AAAA'
$Tags = @{'DiskEncryptionKeyEncryptionAlgorithm' = 'RSA-OAEP';'DiskEncryptionKeyFileName' = 'B3284AAA-DAAA-4AAA-B393-60CAA848AAAA.BEK';'DiskEncryptionKeyEncryptionKeyURL' = $encryptionObject.OsDiskKeyAndSecretDetails.KeyUrl;'MachineName' = 'vm-name'}
Set-AzureKeyVaultSecret -VaultName '<target_key_vault_name>' -Name $secretname -SecretValue $Secret -ContentType  'Wrapped BEK' -Tags $Tags
```

**Воспользуйтесь этими командлетами, если виртуальная машина Linux зашифрована с помощью BEK и KEK.**

```powershell
$secretdata = $encryptionObject.OsDiskKeyAndSecretDetails.SecretData
$Secret = ConvertTo-SecureString -String $secretdata -AsPlainText -Force
$secretname = 'B3284AAA-DAAA-4AAA-B393-60CAA848AAAA'
$Tags = @{'DiskEncryptionKeyEncryptionAlgorithm' = 'RSA-OAEP';'DiskEncryptionKeyFileName' = 'LinuxPassPhraseFileName';'DiskEncryptionKeyEncryptionKeyURL' = <Key_url_of_newly_restored_key>;'MachineName' = 'vm-name'}
Set-AzureKeyVaultSecret -VaultName '<target_key_vault_name>' -Name $secretname -SecretValue $Secret -ContentType  'Wrapped BEK' -Tags $Tags
```

Используйте созданный выше JSON-файл для получения имени и значения секрета и укажите его при выполнении командлета настройки секрета, чтобы поместить секрет (BEK) обратно в хранилище ключей. **Воспользуйтесь этими командлетами, если виртуальная машина зашифрована только с помощью BEK.**

```powershell
$secretDestination = 'C:\secret.blob'
[io.file]::WriteAllBytes($secretDestination, [System.Convert]::FromBase64String($encryptionObject.OsDiskKeyAndSecretDetails.KeyVaultSecretBackupData))
Restore-AzureKeyVaultSecret -VaultName '<target_key_vault_name>' -InputFile $secretDestination -Verbose
  ```

> [!NOTE]
>
> * Значение для $secretname можно получить, обратившись к выходным данным $encryptionObject. Осдисккэйандсекретдетаилс. SecretUrl и используя текст после секретов/, например, URL-адрес выходного секрета — `https://keyvaultname.vault.azure.net/secrets/B3284AAA-DAAA-4AAA-B393-60CAA848AAAA/xx000000xx0849999f3xx30000003163` и имя секрета — b3284aaa-daaa-4AAA-b393-60caa848aaaa.
> * Значение тега Diskencryptionkeyfilename совпадает с совпадает с именем секрета.
>
>

## <a name="create-virtual-machine-from-restored-disk"></a>Создание виртуальной машины на основе восстановленного диска

Если вы заархивированы зашифрованную виртуальную машину с помощью резервного копирования виртуальных машин Azure, командлеты PowerShell, упомянутые выше, помогут вам восстановить ключ и секрет в хранилище ключей. После восстановления см. статью [Управление резервным копированием и восстановлением виртуальных машин Azure с помощью PowerShell](backup-azure-vms-automation.md#create-a-vm-from-restored-disks) для создания зашифрованных виртуальных машин из восстановленного диска, ключа и секрета.

## <a name="legacy-approach"></a>Устаревший подход

Описанный выше подход подойдет для всех точек восстановления. Однако более старый подход получения сведений о ключе и секрете из точки восстановления можно по-прежнему использовать для точек восстановления, созданных до 11 июля 2017 г. (для виртуальных машин, зашифрованных с помощью BEK и KEK). После выполнения задания восстановления диска для зашифрованной виртуальной машины с помощью [PowerShell](backup-azure-vms-automation.md#restore-an-azure-vm) убедитесь, что переменная $rp заполняется допустимым значением.

### <a name="restore-key-legacy-approach"></a>Ключ восстановления (прежний подход)

Используйте следующие командлеты для получения из точки восстановления сведений о ключе (KEK) и укажите его при выполнении командлета восстановления ключа, чтобы поместить его обратно в хранилище ключей.

```powershell
$rp1 = Get-AzRecoveryServicesBackupRecoveryPoint -RecoveryPointId $rp[0].RecoveryPointId -Item $backupItem -KeyFileDownloadLocation 'C:\Users\downloads'
Restore-AzureKeyVaultKey -VaultName '<target_key_vault_name>' -InputFile 'C:\Users\downloads'
```

### <a name="restore-secret-legacy-approach"></a>Восстановить секрет (устаревший подход)

Используйте следующие командлеты для получения из точки восстановления сведений о секрете (BEK) и укажите его при выполнении командлета настройки секрета, чтобы поместить его обратно в хранилище ключей.

```powershell
$secretname = 'B3284AAA-DAAA-4AAA-B393-60CAA848AAAA'
$secretdata = $rp1.KeyAndSecretDetails.SecretData
$Secret = ConvertTo-SecureString -String $secretdata -AsPlainText -Force
$Tags = @{'DiskEncryptionKeyEncryptionAlgorithm' = 'RSA-OAEP';'DiskEncryptionKeyFileName' = 'B3284AAA-DAAA-4AAA-B393-60CAA848AAAA.BEK';'DiskEncryptionKeyEncryptionKeyURL' = 'https://mykeyvault.vault.azure.net:443/keys/KeyName/84daaac999949999030bf99aaa5a9f9';'MachineName' = 'vm-name'}
Set-AzureKeyVaultSecret -VaultName '<target_key_vault_name>' -Name $secretname -SecretValue $secret -Tags $Tags -SecretValue $Secret -ContentType  'Wrapped BEK'
```

> [!NOTE]
>
> * Значение для $secretname можно получить, обратившись к выходным данным $rp 1. Кэйандсекретдетаилс. SecretUrl и использование текста после секретов/, например, URL-адрес выходного секрета —, `https://keyvaultname.vault.azure.net/secrets/B3284AAA-DAAA-4AAA-B393-60CAA848AAAA/xx000000xx0849999f3xx30000003163` а имя секрета — b3284aaa-daaa-4AAA-b393-60caa848aaaa.
> * Значение тега DiskEncryptionKeyFileName совпадает с именем секрета.
> * Значение для DiskEncryptionKeyEncryptionKeyURL можно получить из хранилища ключей после восстановления ключей с помощью командлета [Get-AzureKeyVaultKey](/powershell/module/azurerm.keyvault/get-azurekeyvaultkey).
>
>

## <a name="next-steps"></a>Дальнейшие действия

После восстановления ключа и секрета в хранилище ключей обратитесь к статье [Управление резервным копированием и восстановлением виртуальных машин Azure с помощью PowerShell](backup-azure-vms-automation.md#create-a-vm-from-restored-disks) , чтобы создать зашифрованные виртуальные машины из восстановленного диска, ключа и секрета.
