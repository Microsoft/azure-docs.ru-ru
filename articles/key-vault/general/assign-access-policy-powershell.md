---
title: Назначение политики доступа Azure Key Vault
description: Использование портал Azure, Azure CLI или Azure PowerShell для назначения политики доступа Key Vault к удостоверению субъекта-службы или приложения.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 08/27/2020
ms.author: mbaldwin
ms.openlocfilehash: 4cf0393cdb0aae1b352b81df4893f874f34970cc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97934543"
---
# <a name="assign-a-key-vault-access-policy-using-azure-powershell"></a>Назначение политики доступа Key Vault с помощью Azure PowerShell

Политика доступа Key Vault определяет, может ли заданный субъект-служба, а именно приложение или группа пользователей, выполнять различные операции с Key Vault [секретами](../secrets/index.yml), [ключами](../keys/index.yml)и [сертификатами](../certificates/index.yml). Политики доступа можно назначать с помощью [портал Azure](assign-access-policy-portal.md), [Azure CLI](assign-access-policy-cli.md)или Azure PowerShell (в этой статье).

[!INCLUDE [key-vault-access-policy-limits.md](../../../includes/key-vault-access-policy-limits.md)]

Дополнительные сведения о создании групп в Azure Active Directory с помощью Azure PowerShell см. в разделе [New-AzureADGroup](/powershell/module/azuread/new-azureadgroup) и [Add-азадграупмембер](/powershell/module/az.resources/add-azadgroupmember).

## <a name="configure-powershell-and-sign-in"></a>Настройка PowerShell и вход в систему

1. Чтобы выполнить команды локально, установите [Azure PowerShell](/powershell/azure/) , если вы еще этого не сделали.

    Для выполнения команд непосредственно в облаке используйте [Azure Cloud Shell](../../cloud-shell/overview.md).

1. Только локальная оболочка PowerShell:

    1. Установите модуль [PowerShell для Azure Active Directory](https://www.powershellgallery.com/packages/AzureAD).

    1. Войдите в Azure.

        ```azurepowershell-interactive
        Login-AzAccount
        ```
    
## <a name="acquire-the-object-id"></a>Получение идентификатора объекта

Определите идентификатор объекта приложения, группы или пользователя, которому необходимо назначить политику доступа:

- Приложения и другие субъекты-службы. Используйте командлет [Get-азадсервицепринЦипал](/powershell/module/az.resources/get-azadserviceprincipal) с `-SearchString` параметром, чтобы отфильтровать результаты по имени нужного субъекта-службы:

    ```azurepowershell-interactive
    Get-AzADServicePrincipal -SearchString <search-string>
    ```

- Группы: используйте командлет [Get-азадграуп](/powershell/module/az.resources/get-azadgroup) с `-SearchString` параметром, чтобы отфильтровать результаты по имени нужной группы:

    ```azurepowershell-interactive
    Get-AzADGroup -SearchString <search-string>
    ```
    
    В выходных данных идентификатор объекта указан как `Id` .

- Пользователи: используйте командлет [Get-азадусер](/powershell/module/az.resources/get-azaduser) , передав адрес электронной почты пользователя в `-UserPrincipalName` параметр.

    ```azurepowershell-interactive
     Get-AzAdUser -UserPrincipalName <email-address-of-user>
    ```

    В выходных данных идентификатор объекта указан как `Id` .

## <a name="assign-the-access-policy"></a>Назначение политики доступа

Чтобы назначить политику доступа, используйте командлет [Set-азкэйваултакцессполици](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) :

```azurepowershell-interactive
Set-AzKeyVaultAccessPolicy -VaultName <key-vault-name> -ObjectId <Id> -PermissionsToSecrets <secrets-permissions> -PermissionsToKeys <keys-permissions> -PermissionsToCertificates <certificate-permissions    
```

Необходимо включать только `-PermissionsToSecrets` , `-PermissionsToKeys` и `-PermissionsToCertificates` при назначении разрешений для этих конкретных типов. Допустимые значения для параметров `<secret-permissions>` , `<key-permissions>` и `<certificate-permissions>` приведены в документации по [Set-азкэйваултакцессполици-Parameters](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy#parameters) .

## <a name="next-steps"></a>Дальнейшие действия

- [Azure Key Vault security: Identity and access management](security-overview.md#identity-management) (Azure Key Vault security: управление удостоверениями и доступом)
- [Обеспечьте безопасность хранилища ключей](secure-your-key-vault.md).
- [Руководство разработчика Azure Key Vault](developers-guide.md)