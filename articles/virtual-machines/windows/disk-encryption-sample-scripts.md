---
title: Примеры сценариев шифрования дисков Azure для виртуальных машин Windows
description: В этой статье приводятся сведения о Microsoft Azure шифровании дисков для виртуальных машин Windows.
author: msmbaldwin
ms.service: virtual-machines
ms.subservice: disks
ms.collection: windows
ms.topic: how-to
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: 413553165fcf74fa4590cb4661212b885a277579
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102550643"
---
# <a name="azure-disk-encryption-sample-scripts"></a>Примеры скриптов шифрования дисков Azure 

В этой статье приводятся примеры сценариев для подготовки предварительно зашифрованных виртуальных жестких дисков и других задач.

> [!NOTE]
> Все скрипты относятся к последней версии файла ADE, отличной от AAD, за исключением указанных выше.

## <a name="sample-powershell-scripts-for-azure-disk-encryption"></a>Примеры сценариев PowerShell для шифрования дисков Azure 


- **Вывод списка всех зашифрованных виртуальных машин в подписке**

  Все виртуальные машины, зашифрованные с помощью ADE, и версия расширения во всех группах ресурсов, имеющихся в подписке, можно найти с помощью [этого сценария PowerShell](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/Find_1passAdeVersion_VM.ps1).

  Кроме того, эти командлеты будут показывать все виртуальные машины, зашифрованные с помощью ADE (но не версия расширения):

    ```azurepowershell-interactive
    $osVolEncrypted = {(Get-AzVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).OsVolumeEncrypted}
    $dataVolEncrypted= {(Get-AzVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).DataVolumesEncrypted}
    Get-AzVm | Format-Table @{Label="MachineName"; Expression={$_.Name}}, @{Label="OsVolumeEncrypted"; Expression=$osVolEncrypted}, @{Label="DataVolumesEncrypted"; Expression=$dataVolEncrypted}
    ```

- **Вывод списка всех зашифрованных экземпляров VMSS в вашей подписке**
    
    Все экземпляры VMSS, зашифрованные с помощью ADE, и версия расширения во всех группах ресурсов, имеющихся в подписке, можно найти с помощью [этого сценария PowerShell](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/Find_1passAdeVersion_VMSS.ps1).
 
- **Вывод списка всех секретов шифрования дисков, используемых для шифрования виртуальных машин в хранилище ключей**

```azurepowershell-interactive
Get-AzKeyVaultSecret -VaultName $KeyVaultName | where {$_.Tags.ContainsKey('DiskEncryptionKeyFileName')} | format-table @{Label="MachineName"; Expression={$_.Tags['MachineName']}}, @{Label="VolumeLetter"; Expression={$_.Tags['VolumeLetter']}}, @{Label="EncryptionKeyURL"; Expression={$_.Id}}
```

### <a name="using-the-azure-disk-encryption-prerequisites-powershell-script"></a> Выполнение сценария PowerShell для установки компонентов, необходимых при шифровании дисков Azure

Если вы уже знакомы с предварительными требованиями для шифрования дисков Azure, можно использовать [соответствующий сценарий PowerShell предварительных требований](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/AzureDiskEncryptionPreRequisiteSetup.ps1 ). Пример использования этого сценария PowerShell см. в статье [Краткое руководство. Шифрование виртуальной машины IaaS под управлением Windows с помощью Azure PowerShell](disk-encryption-powershell-quickstart.md). Вы можете удалить комментарии из раздела сценария, начиная со строки 211, чтобы шифровать все диски имеющихся виртуальных машин в имеющейся группе ресурсов. 

В следующей таблице показано, какие параметры могут использоваться в сценарии PowerShell: 

|Параметр|Описание|Обязательное?|
|------|------|------|
|$resourceGroupName| Имя группы ресурсов, к которой принадлежит хранилище ключей.  При отсутствии группы ресурсов с таким именем — она будет создана.| True|
|$keyVaultName|Имя хранилища ключей, в котором будут размещаться ключи шифрования. При отсутствии хранилища ключей с таким именем — оно будет создано.| True|
|$location|Расположение хранилища ключей. Убедитесь, что хранилище ключей и виртуальные машины, которые предстоит зашифровать, находятся в одном расположении. Получите список расположений с помощью команды `Get-AzLocation`.|True|
|$subscriptionId|Идентификатор подписки Azure для использования.  Вы можете получить идентификатор подписки с помощью команды `Get-AzSubscription`.|True|
|$aadAppName|Имя приложения Azure AD, которое будет использоваться для записи секретов в хранилище ключей. Будет создано приложение с таким именем (если оно еще не создано). Если это приложение уже есть, передайте параметр aadClientSecret в сценарий.|False|
|$aadClientSecret|Секрет клиента приложения Azure AD, который был создан ранее.|False|
|$keyEncryptionKeyName|Имя дополнительного ключа шифрования ключа в хранилище ключей. При отсутствии ключа с таким именем — он будет создан.|False|

## <a name="resource-manager-templates"></a>Шаблоны Resource Manager

### <a name="encrypt-or-decrypt-vms-without-an-azure-ad-app"></a>Шифрование или расшифровка виртуальных машин без приложения Azure AD

- [Включение шифрования дисков на существующей или работающей виртуальной машине Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm-without-aad)  
- [Отключение шифрования на работающей виртуальной машине Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm-without-aad) 

### <a name="encrypt-or-decrypt-vms-with-an-azure-ad-app-previous-release"></a>Шифрование или расшифровка виртуальных машин с приложением Azure AD (предыдущий выпуск) 
 
- [Включение шифрования дисков на существующей или работающей виртуальной машине Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm)    
- [Отключение шифрования на работающей виртуальной машине Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm) 
- [Создание нового зашифрованного управляемого диска из предварительно зашифрованного виртуального жесткого диска или большого двоичного объекта хранилища](https://github.com/Azure/azure-quickstart-templates/tree/master/201-create-encrypted-managed-disk)
    - Создает зашифрованный управляемый диск на основе предварительно зашифрованного виртуального жесткого диска и его соответствующих параметров шифрования.

## <a name="prepare-a-pre-encrypted-windows-vhd"></a>Подготовка предварительно зашифрованного виртуального жесткого диска Windows
В следующем разделе приведены инструкции по подготовке предварительно зашифрованного виртуального жесткого диска Windows к развертыванию в качестве зашифрованного виртуального жесткого диска в IaaS Azure. Используйте эти сведения, чтобы подготовить и загрузить новую виртуальную машину Windows (VHD) в Azure Site Recovery Azure или Azure. Дополнительные сведения о подготовке и отправке виртуального жесткого диска см. в статье [Отправка универсального диска VHD и создание виртуальных машин с его помощью в Azure](upload-generalized-managed.md).

### <a name="update-group-policy-to-allow-non-tpm-for-os-protection"></a>Обновление групповой политики с целью разрешить защиту ОС без TPM
Настройте параметр групповой политики BitLocker, который называется **Шифрование дисков BitLocker**. Для этого выберите **Политика локального компьютера** > **Конфигурация компьютера** > **Административные шаблоны** > **Компоненты Windows**. Измените этот параметр на **диски операционной системы**,  >  **требующие дополнительной проверки подлинности при запуске**  >  ,**Разрешить BitLocker без совместимого доверенного платформенного модуля**, как показано на следующем рисунке:

![Антивредоносное ПО Майкрософт в Azure](../media/disk-encryption/disk-encryption-fig8.png)

### <a name="install-bitlocker-feature-components"></a>Установка компонентов BitLocker
Для Windows Server 2012 или более поздней версии используйте следующую команду.

```console
dism /online /Enable-Feature /all /FeatureName:BitLocker /quiet /norestart
```

Для Windows Server 2008 R2 используйте следующую команду.

```console
ServerManagerCmd -install BitLockers
```

### <a name="prepare-the-os-volume-for-bitlocker-by-using-bdehdcfg"></a>Подготовка тома операционной системы для BitLocker с помощью `bdehdcfg`
Чтобы сжать раздел операционной системы и подготовить компьютер к использованию BitLocker, запустите компонент [bdehdcfg](/windows/security/information-protection/bitlocker/bitlocker-basic-deployment).

```console
bdehdcfg -target c: shrink -quiet 
```

### <a name="protect-the-os-volume-by-using-bitlocker"></a>Защита тома операционной системы с помощью BitLocker
Используйте [`manage-bde`](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/ff829849(v=ws.11)) команду, чтобы включить шифрование загрузочного тома с помощью внешнего предохранителя ключа. Можно также поместить внешний ключ (BEK-файл) на внешний диск или том. Шифрование будет включено для системного или загрузочного тома после перезагрузки.

```console
manage-bde -on %systemdrive% -sk [ExternalDriveOrVolume]
reboot
```

> [!NOTE]
> Чтобы получить внешний ключ с использованием BitLocker, подготовьте виртуальную машину с отдельным виртуальным жестким диском для данных и ресурсов.

## <a name="upload-encrypted-vhd-to-an-azure-storage-account"></a>Передача зашифрованного виртуального жесткого диска в учетную запись хранения Azure
После включения шифрования DM-Crypt необходимо отправить локальный зашифрованный виртуальный жесткий диск в учетную запись хранения.
```powershell
    Add-AzVhd [-Destination] <Uri> [-LocalFilePath] <FileInfo> [[-NumberOfUploaderThreads] <Int32> ] [[-BaseImageUriToPatch] <Uri> ] [[-OverWrite]] [ <CommonParameters>]
```

## <a name="upload-the-secret-for-the-pre-encrypted-vm-to-your-key-vault"></a> Передача секрета для предварительно зашифрованной виртуальной машины в хранилище ключей
Полученный ранее секрет шифрования диска должен быть отправлен в качестве секрета в хранилище ключей.  Для этого необходимо предоставить разрешение Set Secret и разрешение wrapkey учетной записи, которая будет отправлять секреты.

```powershell 
# Typically, account Id is the user principal name (in user@domain.com format)
$upn = (Get-AzureRmContext).Account.Id
Set-AzKeyVaultAccessPolicy -VaultName $kvname -UserPrincipalName $acctid -PermissionsToKeys wrapKey -PermissionsToSecrets set

# In cloud shell, the account ID is a managed service identity, so specify the username directly 
# $upn = "user@domain.com" 
# Set-AzKeyVaultAccessPolicy -VaultName $kvname -UserPrincipalName $acctid -PermissionsToKeys wrapKey -PermissionsToSecrets set

# When running as a service principal, retrieve the service principal ID from the account ID, and set access policy to that 
# $acctid = (Get-AzureRmContext).Account.Id
# $spoid = (Get-AzureRmADServicePrincipal -ServicePrincipalName $acctid).Id
# Set-AzKeyVaultAccessPolicy -VaultName $kvname -ObjectId $spoid -BypassObjectIdValidation -PermissionsToKeys wrapKey -PermissionsToSecrets set

```

### <a name="disk-encryption-secret-not-encrypted-with-a-kek"></a>Секрет дискового шифрования не шифруется с помощью KEK
Чтобы настроить секрет в хранилище ключей, используйте [Set-азкэйваултсекрет](/powershell/module/az.keyvault/set-azkeyvaultsecret). Парольная фраза кодируется как строка Base64, а затем передается в хранилище ключей. Убедитесь также, что при создании секрета в хранилище ключей были установлены следующие теги.

```powershell

 # This is the passphrase that was provided for encryption during the distribution installation
 $passphrase = "contoso-password"

 $tags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
 $secretName = [guid]::NewGuid().ToString()
 $secretValue = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($passphrase))
 $secureSecretValue = ConvertTo-SecureString $secretValue -AsPlainText -Force

 $secret = Set-AzKeyVaultSecret -VaultName $KeyVaultName -Name $secretName -SecretValue $secureSecretValue -tags $tags
 $secretUrl = $secret.Id
```


На следующем шаге используйте `$secretUrl`, чтобы [подключить диск ОС, не применяя ключ шифрования ключей](#without-using-a-kek).

### <a name="disk-encryption-secret-encrypted-with-a-kek"></a>Секрет дискового шифрования, зашифрованный с помощью KEK
Перед передачей секрета в хранилище ключей его можно дополнительно зашифровать с помощью ключа шифрования ключей. Используйте [API](/rest/api/keyvault/wrapkey) для создания оболочки, чтобы сначала зашифровать секрет с использованием ключа шифрования ключей. Выходными данными этой операции по переносу является строка в кодировке Base64, которую можно затем передать в качестве секрета с помощью [`Set-AzKeyVaultSecret`](/powershell/module/az.keyvault/set-azkeyvaultsecret) командлета.

```powershell
    # This is the passphrase that was provided for encryption during the distribution installation
    $passphrase = "contoso-password"

    Add-AzKeyVaultKey -VaultName $KeyVaultName -Name "keyencryptionkey" -Destination Software
    $KeyEncryptionKey = Get-AzKeyVaultKey -VaultName $KeyVault.OriginalVault.Name -Name "keyencryptionkey"

    $apiversion = "2015-06-01"

    ##############################
    # Get Auth URI
    ##############################

    $uri = $KeyVault.VaultUri + "/keys"
    $headers = @{}

    $response = try { Invoke-RestMethod -Method GET -Uri $uri -Headers $headers } catch { $_.Exception.Response }

    $authHeader = $response.Headers["www-authenticate"]
    $authUri = [regex]::match($authHeader, 'authorization="(.*?)"').Groups[1].Value

    Write-Host "Got Auth URI successfully"

    ##############################
    # Get Auth Token
    ##############################

    $uri = $authUri + "/oauth2/token"
    $body = "grant_type=client_credentials"
    $body += "&client_id=" + $AadClientId
    $body += "&client_secret=" + [Uri]::EscapeDataString($AadClientSecret)
    $body += "&resource=" + [Uri]::EscapeDataString("https://vault.azure.net")
    $headers = @{}

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $access_token = $response.access_token

    Write-Host "Got Auth Token successfully"

    ##############################
    # Get KEK info
    ##############################

    $uri = $KeyEncryptionKey.Id + "?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token}

    $response = Invoke-RestMethod -Method GET -Uri $uri -Headers $headers

    $keyid = $response.key.kid

    Write-Host "Got KEK info successfully"

    ##############################
    # Encrypt passphrase using KEK
    ##############################

    $passphraseB64 = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($Passphrase))
    $uri = $keyid + "/encrypt?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"alg" = "RSA-OAEP"; "value" = $passphraseB64}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $wrappedSecret = $response.value

    Write-Host "Encrypted passphrase successfully"

    ##############################
    # Store secret
    ##############################

    $secretName = [guid]::NewGuid().ToString()
    $uri = $KeyVault.VaultUri + "/secrets/" + $secretName + "?api-version=" + $apiversion
    $secretAttributes = @{"enabled" = $true}
    $secretTags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"value" = $wrappedSecret; "attributes" = $secretAttributes; "tags" = $secretTags}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method PUT -Uri $uri -Headers $headers -Body $body

    Write-Host "Stored secret successfully"

    $secretUrl = $response.id
```

На следующем шаге мы используем `$KeyEncryptionKey` и `$secretUrl`, чтобы [подключить диск ОС, применив ключ шифрования ключей](#using-a-kek).

##  <a name="specify-a-secret-url-when-you-attach-an-os-disk"></a>Указание URL-адреса секрета при подключении диска ОС

###  <a name="without-using-a-kek"></a>Без использования ключа шифрования ключа
При подключении диска ОС необходимо передать `$secretUrl`. Этот URL-адрес был создан в разделе "Секрет шифрования дисков, не зашифрованный с помощью ключа шифрования ключей".
```powershell
    Set-AzVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $VhdUri `
            -VhdUri $OSDiskUri `
            -Windows `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl
```
### <a name="using-a-kek"></a>С использованием ключа шифрования ключа
При подключении диска ОС передайте `$KeyEncryptionKey` и `$secretUrl`. Этот URL-адрес был создан в разделе "Секрет шифрования дисков, зашифрованный с помощью ключа шифрования ключей".
```powershell
    Set-AzVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $CopiedTemplateBlobUri `
            -VhdUri $OSDiskUri `
            -Windows `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl `
            -KeyEncryptionKeyVaultId $KeyVault.ResourceId `
            -KeyEncryptionKeyURL $KeyEncryptionKey.Id
```
