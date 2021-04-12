---
title: Краткое руководство. Настройка и просмотр сертификатов Azure Key Vault с помощью Azure PowerShell
description: Краткое руководство по настройке и получению сертификата из Azure Key Vault с помощью Azure PowerShell
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: quickstart
ms.custom: mvc, seo-javascript-september2019, seo-javascript-october2019
ms.date: 01/27/2021
ms.author: mbaldwin
ms.openlocfilehash: 587815cf9628df35f1e1efdbc6a7a3c89a27ed55
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99071923"
---
# <a name="quickstart-set-and-retrieve-a-certificate-from-azure-key-vault-using-azure-powershell"></a>Краткое руководство. Настройка и получение сертификата из Azure Key Vault с помощью Azure PowerShell

Из этого краткого руководства вы узнаете, как создать хранилище ключей в Azure Key Vault с помощью Azure PowerShell. Azure Key Vault — это облачная служба, которая работает как защищенное хранилище секретов. Вы можете безопасно хранить ключи, пароли, сертификаты и другие секреты. Дополнительные сведения о хранилище ключей см. в статье [обзора](../general/overview.md). Azure PowerShell используется для создания ресурсов Azure и управления ими с помощью скриптов и команд. Создав ресурс, вы сможете сохранить в нем сертификат.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.


[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально для работы с этим руководством, понадобится модуль Azure PowerShell 1.0.0 или более поздней версии. Чтобы узнать версию, введите `$PSVersionTable.PSVersion`. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Login-AzAccount`, чтобы создать подключение к Azure.

```azurepowershell-interactive
Login-AzAccount
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

[!INCLUDE [Create a resource group](../../../includes/key-vault-powershell-rg-creation.md)]

## <a name="create-a-key-vault"></a>Создание хранилища ключей

[!INCLUDE [Create a key vault](../../../includes/key-vault-powershell-kv-creation.md)]

## <a name="add-a-certificate-to-key-vault"></a>Добавление сертификата в Key Vault

Чтобы добавить сертификат в хранилище, вам нужно выполнить несколько дополнительных действий. Этот сертификат может использоваться приложением. 

Выполните приведенные ниже команды, чтобы создать самозаверяющий сертификат с политикой — **ExampleCertificate**.

```azurepowershell-interactive
$Policy = New-AzKeyVaultCertificatePolicy -SecretContentType "application/x-pkcs12" -SubjectName "CN=contoso.com" -IssuerName "Self" -ValidityInMonths 6 -ReuseKeyOnRenewal

Add-AzKeyVaultCertificate -VaultName "<your-unique-keyvault-name>" -Name "ExampleCertificate" -CertificatePolicy $Policy
```

Теперь сертификат, добавленный в Azure Key Vault, можно вызвать, используя его URI. Чтобы получить текущую версию, используйте **https://<your-unique-keyvault-name>.vault.azure.net/certificates/ExampleCertificate**. 

Чтобы просмотреть ранее сохраненный сертификат, выполните следующие действия:

```azurepowershell-interactive
Get-AzKeyVaultCertificate -VaultName "<your-unique-keyvault-name>" -Name "ExampleCertificate"
```

Вы создали Key Vault, сохранили в нем сертификат и извлекли его.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [Create a key vault](../../../includes/key-vault-powershell-delete-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать Key Vault и сохранить в нем сертификат. Дополнительные сведения о Key Vault и его интеграции в приложения см. в следующих статьях.

- [Обзор Azure Key Vault](../general/overview.md)
- Справочник по [командлетам Azure PowerShell для Azure Key Vault](/powershell/module/az.keyvault/)
- Статья [Обзор системы безопасности Key Vault](../general/security-overview.md)
