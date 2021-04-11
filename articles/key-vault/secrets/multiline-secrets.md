---
title: Хранение многострочных секретов в Azure Key Vault
description: В это руководстве показано, как использовать многострочные секреты из Azure Key Vault с помощью Azure CLI и PowerShell.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: mvc, seo-javascript-september2019, seo-javascript-october2019, devx-track-azurecli
ms.date: 03/17/2021
ms.author: mbaldwin
ms.openlocfilehash: e320795d5e8623dea9b97ac6371a0ffc6e6117f1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104610193"
---
# <a name="store-a-multi-line-secret-in-azure-key-vault"></a>Хранение многострочных секретов в Azure Key Vault

В кратких руководствах для [Azure CLI](quick-create-cli.md) и [Azure PowerShell](quick-create-powershell.md) показано, как сохранить однострочный секрет.   Но вы можете использовать Key Vault и для хранения многострочного секрета, например JSON-файла или закрытого ключа RSA.

Многострочные секреты не могут быть переданы в команду Azure CLI [az keyvault secret set](/cli/azure/keyvault/secret#az_keyvault_secret_set) или в командлет Azure PowerShell [Set-AzKeyVaultSecret](/powershell/module/az.keyvault/set-azkeyvaultsecret) через командную строку. Вместо этого их нужно сохранять в виде текстового файла. 

Например, вы можете создать текстовый файл с именем secretfile.txt, который содержит следующие строки:

```bash
This is my
multi-line
secret
```

Затем этот файл можно передать в команду Azure CLI [az keyvault secret set](/cli/azure/keyvault/secret#az_keyvault_secret_set) с помощью параметра `--file`.

```azurecli-interactive
az keyvault secret set --vault-name "<your-unique-keyvault-name>" --name "MultilineSecret" --file "secretfile.txt"
```

При работе в Azure PowerShell следует сначала считать файл с помощью командлета [Get-Content](/powershell/module/microsoft.powershell.management/get-content), а затем преобразовать его в защищенную строку с помощью [ConvertTo-SecureString](/powershell/module/microsoft.powershell.security/convertto-securestring). 

```azurepowershell-interactive
$RawSecret =  Get-Content "secretfile.txt" -Raw
$SecureSecret = ConvertTo-SecureString -String $RawSecret -AsPlainText -Force
```

И наконец, для сохранения секрета используйте командлет [Set-AzKeyVaultSecret](/powershell/module/az.keyvault/set-azkeyvaultsecret).

```azurepowershell-interactive
$secret = Set-AzKeyVaultSecret -VaultName "<your-unique-keyvault-name>" -Name "MultilineSecret" -SecretValue $SecureSecret
```

В любом случае вы сможете получить сохраненный секрет с помощью команды Azure CLI [az keyvault secret show](/cli/azure/keyvault/secret#az_keyvault_secret_show) или командлета Azure PowerShell [Get-AzKeyVaultSecret](/powershell/module/az.keyvault/get-azkeyvaultsecret).

```azurecli-interactive
az keyvault secret show --name "MultilineSecret" --vault-name "<your-unique-keyvault-name>" --query "value"
```

Они возвращают сохраненный секрет с внедренными символами новой строки:

```bash
"This is\nmy multi-line\nsecret"
```

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор Azure Key Vault](../general/overview.md)
- [Краткое руководство по Azure CLI](quick-create-cli.md)
- [Команды az keyvault в Azure CLI](/cli/azure/keyvault)
- [Краткое руководство по Azure PowerShell](quick-create-powershell.md)
- [Командлеты Az.KeyVault в Azure PowerShell](/powershell/module/az.keyvault#key-vault)
