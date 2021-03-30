---
title: Краткое руководство. Настройка и получение секрета из Key Vault с помощью PowerShell
description: Из этого руководства вы узнаете, как создавать, извлекать и удалять секреты из Azure Key Vault с помощью Azure PowerShell.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: mvc, devx-track-azurepowershell
ms.date: 01/27/2021
ms.author: mbaldwin
ms.openlocfilehash: 5e3b78dfb8f08f1a596ad12884ec088cd36c4a98
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104582460"
---
# <a name="quickstart-set-and-retrieve-a-secret-from-azure-key-vault-using-powershell"></a>Краткое руководство. Настройка и получение секрета из Azure Key Vault с помощью PowerShell

Azure Key Vault — это облачная служба, которая работает как защищенное хранилище секретов. Вы можете безопасно хранить ключи, пароли, сертификаты и другие секреты. Дополнительные сведения о хранилище ключей см. в статье [обзора](../general/overview.md). В этом кратком руководстве для создания хранилища ключей вы используете PowerShell. Затем вы сохраните секрет в только что созданном хранилище.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально для работы с этим руководством, понадобится модуль Azure PowerShell 5.0.0 или более поздней версии. Чтобы узнать версию, введите `$PSVersionTable.PSVersion`. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

```azurepowershell-interactive
Connect-AzAccount
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

[!INCLUDE [Create a resource group](../../../includes/key-vault-powershell-rg-creation.md)]

## <a name="create-a-key-vault"></a>Создание хранилища ключей

[!INCLUDE [Create a key vault](../../../includes/key-vault-powershell-kv-creation.md)]

## <a name="give-your-user-account-permissions-to-manage-secrets-in-key-vault"></a>Предоставление учетной записи пользователя разрешений на управление секретами в Key Vault

Используйте командлет Azure PowerShell Set-AzKeyVaultAccessPolicy, чтобы обновить политику доступа Key Vault и предоставить разрешения секретов учетной записи пользователя.

```azurepowershell-interactive
Set-AzKeyVaultAccessPolicy -VaultName "<your-unique-keyvault-name>" -UserPrincipalName "user@domain.com" -PermissionsToSecrets get,set,delete
```

## <a name="adding-a-secret-to-key-vault"></a>Добавление секрета в Key Vault

Чтобы добавить секрет в хранилище, вам просто нужно выполнить несколько шагов. В этом случае вы добавите пароль, который может быть использован приложением. Пароль называется **ExamplePassword**, и в нем хранится значение **hVFkk965BuUv**.

Сначала преобразуйте значение **hVFkk965BuUv** в безопасную строку, выполнив следующую команду:

```azurepowershell-interactive
$secretvalue = ConvertTo-SecureString "hVFkk965BuUv" -AsPlainText -Force
```

Затем используйте командлет Azure PowerShell [Set-AzKeyVaultSecret](/powershell/module/az.keyvault/set-azkeyvaultsecret), чтобы создать в Key Vault секрет с именем **ExamplePassword** и значением **hVFkk965BuUv**:


```azurepowershell-interactive
$secret = Set-AzKeyVaultSecret -VaultName "<your-unique-keyvault-name>" -Name "ExamplePassword" -SecretValue $secretvalue
```

## <a name="retrieve-a-secret-from-key-vault"></a>получение секрета из Key Vault.

Чтобы просмотреть содержащееся в секрете значение в виде обычного текста, введите следующее:

```azurepowershell-interactive
$secret = Get-AzKeyVaultSecret -VaultName "<your-unique-keyvault-name>" -Name "ExamplePassword"
$ssPtr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($secret.SecretValue)
try {
   $secretValueText = [System.Runtime.InteropServices.Marshal]::PtrToStringBSTR($ssPtr)
} finally {
   [System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($ssPtr)
}
Write-Output $secretValueText
```

Вы создали Key Vault, сохранили в нем секрет и извлекли его.

## <a name="clean-up-resources"></a>Очистка ресурсов

 Другие руководства в этой серии созданы на основе этого документа. Если вы планируете продолжить работу с другими краткими руководствами и статьями, эти ресурсы можно не удалять.

Вы можете удалить ставшие ненужными группу ресурсов, хранилище ключей и все связанные с ним ресурсы, выполнив команду [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name ContosoResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого краткого руководства вы создали Key Vault и сохранили в нем секрет. Дополнительные сведения о Key Vault и его интеграции в приложения см. в следующих статьях.

- [Обзор Azure Key Vault](../general/overview.md)
- Узнайте, как [хранить многострочные секреты в Key Vault](multiline-secrets.md).
- Справочник по [командлетам Azure PowerShell для Azure Key Vault](/powershell/module/az.keyvault/#key_vault)
- Статья [Обзор системы безопасности Key Vault](../general/security-overview.md)
