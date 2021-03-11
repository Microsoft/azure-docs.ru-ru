---
title: Настройка доступа к WinRM для виртуальной машины Azure
description: Настройте доступ WinRM для использования с виртуальной машиной Azure, созданной в модели развертывания Resource Manager.
author: mimckitt
manager: vashan
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 06/16/2016
ms.author: mimckitt
ms.openlocfilehash: ab676e7595a8ccd902eea27612e4c2fd035fae0c
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102555726"
---
# <a name="setting-up-winrm-access-for-virtual-machines-in-azure-resource-manager"></a>Настройка доступа WinRM для виртуальных машин в Azure Resource Manager

Ниже приведены шаги, которые необходимо выполнить для настройки виртуальной машины с возможностью подключения WinRM:

1. Создание хранилища ключей
2. Создание самозаверяющего сертификата
3. передать самозаверяющий сертификат в хранилище ключей;
4. получить URL-адрес для самозаверяющего сертификата в хранилище ключей;
5. сослаться на URL-адрес самозаверяющего сертификата при создании виртуальной машины.

 

## <a name="step-1-create-a-key-vault"></a>Шаг 1. Создание хранилища ключей
Для создания хранилища ключей можно воспользоваться следующей командой:

```azurepowershell
New-AzKeyVault -VaultName "<vault-name>" -ResourceGroupName "<rg-name>" -Location "<vault-location>" -EnabledForDeployment -EnabledForTemplateDeployment
```

## <a name="step-2-create-a-self-signed-certificate"></a>Шаг 2. Создание самозаверяющего сертификата
Можно создать самозаверяющий сертификат с помощью этого сценария PowerShell.

```azurepowershell
$certificateName = "somename"

$thumbprint = (New-SelfSignedCertificate -DnsName $certificateName -CertStoreLocation Cert:\CurrentUser\My -KeySpec KeyExchange).Thumbprint

$cert = (Get-ChildItem -Path cert:\CurrentUser\My\$thumbprint)

$password = Read-Host -Prompt "Please enter the certificate password." -AsSecureString

Export-PfxCertificate -Cert $cert -FilePath ".\$certificateName.pfx" -Password $password
```

## <a name="step-3-upload-your-self-signed-certificate-to-the-key-vault"></a>Шаг 3. Передача самозаверяющего сертификата в хранилище ключей
Перед передачей сертификата в хранилище ключей, созданное на шаге 1, необходимо преобразовать его в формат, который будет понятен для поставщика ресурсов Microsoft.Compute. Следующий сценарий PowerShell позволит это сделать:

```azurepowershell
$fileName = "<Path to the .pfx file>"
$fileContentBytes = Get-Content $fileName -Encoding Byte
$fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)

[System.Collections.HashTable]$TableForJSON = @{
    "data"     = $filecontentencoded;
    "dataType" = "pfx";
    "password" = "<password>";
}
[System.String]$JSONObject = $TableForJSON | ConvertTo-Json

$secret = ConvertTo-SecureString -String $jsonEncoded -AsPlainText –Force
Set-AzKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>" -SecretValue $secret
```

## <a name="step-4-get-the-url-for-your-self-signed-certificate-in-the-key-vault"></a>Шаг 4. Получение URL-адреса для самозаверяющего сертификата в хранилище ключей
При подготовке виртуальной машины поставщику ресурсов Microsoft.Compute требуется URL-адрес для секрета в хранилище ключей. Это позволяет поставщику ресурсов Microsoft.Compute скачать секрет и создать эквивалент сертификата на виртуальной машине.

> [!NOTE]
> URL-адрес секрета также должен включать в себя версию. Пример URL-адреса выглядит примерно ниже HTTPS: \/ /contosovault.Vault.Azure.NET:443/Secrets/contososecret/01h9db0df2cd4300a20ence585a6s7ve

#### <a name="templates"></a>Шаблоны
Получить ссылку на URL-адрес в шаблоне можно с помощью следующего кода:

```json
"certificateUrl": "[reference(resourceId(resourceGroup().name, 'Microsoft.KeyVault/vaults/secrets', '<vault-name>', '<secret-name>'), '2015-06-01').secretUriWithVersion]"
```

#### <a name="powershell"></a>PowerShell
Также для получения этого URL-адреса можно воспользоваться следующей командой PowerShell:

```azurepowershell
$secretURL = (Get-AzKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>").Id
```

## <a name="step-5-reference-your-self-signed-certificates-url-while-creating-a-vm"></a>Шаг 5. Создание ссылки на URL-адрес самозаверяющего сертификата при создании виртуальной машины
#### <a name="azure-resource-manager-templates"></a>Шаблоны Azure Resource Manager
При создании виртуальной машины с помощью шаблонов ссылки на сертификат задаются в разделе секретов и разделе winRM, как показано ниже.

```json
"osProfile": {
      ...
      "secrets": [
        {
          "sourceVault": {
            "id": "<resource id of the Key Vault containing the secret>"
          },
          "vaultCertificates": [
            {
              "certificateUrl": "<URL for the certificate you got in Step 4>",
              "certificateStore": "<Name of the certificate store on the VM>"
            }
          ]
        }
      ],
      "windowsConfiguration": {
        ...
        "winRM": {
          "listeners": [
            {
              "protocol": "http"
            },
            {
              "protocol": "https",
              "certificateUrl": "<URL for the certificate you got in Step 4>"
            }
          ]
        },
        ...
      }
    },
```

Пример шаблона для описанного выше сценария можно найти здесь: [201-vm-winrm-keyvault-windows](https://azure.microsoft.com/documentation/templates/201-vm-winrm-keyvault-windows)

Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-winrm-keyvault-windows)

#### <a name="powershell"></a>PowerShell
```azurepowershell
$vm = New-AzVMConfig -VMName "<VM name>" -VMSize "<VM Size>"
$credential = Get-Credential
$secretURL = (Get-AzKeyVaultSecret -VaultName "<vault name>" -Name "<secret name>").Id
$vm = Set-AzVMOperatingSystem -VM $vm -Windows -ComputerName "<Computer Name>" -Credential $credential -WinRMHttp -WinRMHttps -ProvisionVMAgent -WinRMCertificateUrl $secretURL
$sourceVaultId = (Get-AzKeyVault -ResourceGroupName "<Resource Group name>" -VaultName "<Vault Name>").ResourceId
$CertificateStore = "My"
$vm = Add-AzVMSecret -VM $vm -SourceVaultId $sourceVaultId -CertificateStore $CertificateStore -CertificateUrl $secretURL
```

## <a name="step-6-connecting-to-the-vm"></a>Шаг 6. Подключение к виртуальной машине
Перед подключением к виртуальной машине убедитесь, что машина настроена для удаленного управления WinRM. Запустите PowerShell от имени администратора и выполните следующую команду, чтобы убедиться в правильности настроек:

```azurepowershell
Enable-PSRemoting -Force
```

> [!NOTE]
> Если вышеописанные действия не дали результата, убедитесь, что служба WinRM запущена. Это можно сделать с помощью команды `Get-Service WinRM`
> 
> 

Когда настройки выполнены, можно подключиться к виртуальной машине с помощью следующей команды:

```azurepowershell
Enter-PSSession -ConnectionUri https://<public-ip-dns-of-the-vm>:5986 -Credential $cred -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck) -Authentication Negotiate
```
