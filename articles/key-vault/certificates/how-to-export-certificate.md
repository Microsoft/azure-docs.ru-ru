---
title: Экспорт сертификатов из Azure Key Vault
description: Узнайте, как экспортировать сертификаты из Azure Key Vault.
services: key-vault
author: sebansal
tags: azure-key-vault
ms.service: key-vault
ms.subservice: certificates
ms.topic: how-to
ms.custom: mvc
ms.date: 08/11/2020
ms.author: sebansal
ms.openlocfilehash: 116bafe2e26ca3af5b4ed68373d20e1e787502b7
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105729207"
---
# <a name="export-certificates-from-azure-key-vault"></a>Экспорт сертификатов из Azure Key Vault

Узнайте, как экспортировать сертификаты из Azure Key Vault. Вы можете экспортировать сертификаты с помощью портала Azure, Azure PowerShell или Azure CLI. 

## <a name="about-azure-key-vault-certificates"></a>Сведения о сертификатах Azure Key Vault

Azure Key Vault позволяет легко подготавливать и развертывать цифровые сертификаты для сети, а также управлять ими. Это решение также обеспечивает защищенный обмен данными для приложений. Дополнительные сведения см. в статье [Сертификаты Azure Key Vault](./about-certificates.md).

### <a name="composition-of-a-certificate"></a>Создание сертификатов

Когда создается сертификат Key Vault, создаются адресуемые *ключ* и *секрет* с тем же именем. Ключ Key Vault разрешает операции с ключами, а секрет Key Vault позволяет извлекать значение сертификата в виде секрета. Сертификат Key Vault также содержит открытые метаданные сертификата x509. Дополнительные сведения см. в разделе [Создание сертификатов](./about-certificates.md#composition-of-a-certificate).

### <a name="exportable-and-non-exportable-keys"></a>Доступные и недоступные для экспорта ключи

Созданный сертификат Key Vault можно извлечь из адресуемого секрета с помощью закрытого ключа в формате PFX или PEM.

- **Доступный для экспорта:** политика, используемая для создания сертификата, указывает, что ключ доступен для экспорта.
- **Недоступный для экспорта:** политика, используемая для создания сертификата, указывает, что ключ недоступен для экспорта. В таком случае закрытый ключ не является частью значения при его получении в виде секрета.

Поддерживаемые типы ключей: RSA, RSA-HSM, EC, EC-HSM и т. д. (см. [здесь](/rest/api/keyvault/createcertificate/createcertificate#jsonwebkeytype)) Для экспорта доступны только RSA и EC. Ключи HSM нельзя экспортировать.

Дополнительные сведения см. в разделе [О сертификатах Azure Key Vault](./about-certificates.md#exportable-or-non-exportable-key).

## <a name="export-stored-certificates"></a>Экспорт сохраненных сертификатов

Вы можете экспортировать сохраненные сертификаты из Azure Key Vault с помощью портала Azure, Azure PowerShell или Azure CLI.

> [!NOTE]
> Пароль сертификата требуется только при импорте сертификата в хранилище ключей. Key Vault не хранит связанный пароль. При экспорте сертификата пароль будет пустым.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

С помощью следующей команды в Azure CLI скачайте **открытую часть** сертификата Key Vault.

```azurecli
az keyvault certificate download --file
                                 [--encoding {DER, PEM}]
                                 [--id]
                                 [--name]
                                 [--subscription]
                                 [--vault-name]
                                 [--version]
```

Дополнительные сведения см. в разделе с [примерами и определениями параметров](/cli/azure/keyvault/certificate#az-keyvault-certificate-download).

Скачивание в качестве сертификата означает получение общедоступной части. Если вы хотите использовать как закрытый ключ, так и общедоступные метаданные, вы можете скачать их как секрет.

```azurecli
az keyvault secret download -–file {nameofcert.pfx}
                            [--encoding {ascii, base64, hex, utf-16be, utf-16le, utf-8}]
                            [--id]
                            [--name]
                            [--subscription]
                            [--vault-name]
                            [--version]
```

Дополнительные сведения см. в разделе с [определениями параметров](/cli/azure/keyvault/secret#az-keyvault-secret-download).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

С помощью этой команды в Azure PowerShell получите сертификат с именем **TestCert01** из хранилища ключей с именем **ContosoKV01**. Чтобы скачать сертификат в виде PFX-файла, выполните следующую команду. Эти команды обращаются к параметру **SecretId**, а затем сохраняют его содержимое в виде PFX-файла.

```azurepowershell
$cert = Get-AzKeyVaultCertificate -VaultName "ContosoKV01" -Name "TestCert01"
$secret = Get-AzKeyVaultSecret -VaultName "ContosoKV01" -Name $cert.Name
$secretValueText = '';
$ssPtr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($secret.SecretValue)
try {
    $secretValueText = [System.Runtime.InteropServices.Marshal]::PtrToStringBSTR($ssPtr)
} finally {
    [System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($ssPtr)
}
$secretByte = [Convert]::FromBase64String($secretValueText)
$x509Cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2
$x509Cert.Import($secretByte, "", "Exportable,PersistKeySet")
$type = [System.Security.Cryptography.X509Certificates.X509ContentType]::Pfx
$pfxFileByte = $x509Cert.Export($type, $password)

# Write to a file
[System.IO.File]::WriteAllBytes("KeyVault.pfx", $pfxFileByte)
```

Эта команда позволяет экспортировать всю цепочку сертификатов с использованием закрытого ключа (так же, как и при импорте). Сертификат защищен паролем.
Дополнительные сведения о команде **Get-AzKeyVaultCertificate** и ее параметрах см. в статье [Get-AzKeyVaultCertificate — пример 2](/powershell/module/az.keyvault/Get-AzKeyVaultCertificate).

# <a name="portal"></a>[Портал](#tab/azure-portal)

После создания или импорта сертификата в колонке **Сертификат** на портале Azure вы получите уведомление об успешном создании сертификата. Выберите сертификат и текущую версию, чтобы увидеть вариант скачивания.

Чтобы скачать сертификат, нажмите кнопку **Скачать в формате CER** или **Скачать в формате PFX или PEM**.

![Скачивание сертификата](../media/certificates/quick-create-portal/current-version-shown.png)

**Экспорт сертификатов Службы приложений Azure**

Сертификаты Службы приложений Azure предоставляют удобный способ покупки SSL-сертификатов. Вы можете назначить их приложениям Azure на портале. Импортированные сертификаты Службы приложений находятся в разделе с **секретами**.

Дополнительные сведения см. в статье с инструкциями по [экспорту сертификатов Службы приложений](https://social.technet.microsoft.com/wiki/contents/articles/37431.exporting-azure-app-service-certificates.aspx).

---

## <a name="read-more"></a>Дополнительные сведения
* [Типы и расширения файлов SSL/TLS-сертификатов](/archive/blogs/kaushal/various-ssltls-certificate-file-typesextensions)
