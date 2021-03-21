---
title: Назначение политики доступа Azure Key Vault (CLI)
description: Использование Azure CLI для назначения политики доступа Key Vault для субъекта-службы или удостоверения приложения.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 08/27/2020
ms.author: mbaldwin
ms.openlocfilehash: 0c7910ac149c8de43eeac92913a0d314fcc1854e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97934583"
---
# <a name="assign-a-key-vault-access-policy"></a>Назначение политики доступа Key Vault

Политика доступа Key Vault определяет, может ли заданный субъект-служба, а именно приложение или группа пользователей, выполнять различные операции с Key Vault [секретами](../secrets/index.yml), [ключами](../keys/index.yml)и [сертификатами](../certificates/index.yml). Политики доступа можно назначать с помощью [портал Azure](assign-access-policy-portal.md), Azure CLI (в этой статье) или [Azure PowerShell](assign-access-policy-powershell.md).

[!INCLUDE [key-vault-access-policy-limits.md](../../../includes/key-vault-access-policy-limits.md)]

Дополнительные сведения о создании групп в Azure Active Directory с помощью Azure CLI см. в разделе [AZ AD Group Create](/cli/azure/ad/group#az-ad-group-create) и [AZ AD Member Add](/cli/azure/ad/group/member#az-ad-group-member-add).

## <a name="configure-the-azure-cli-and-sign-in"></a>Настройка Azure CLI и вход

1. Чтобы выполнить Azure CLIные команды локально, установите [Azure CLI](/cli/azure/install-azure-cli).
 
    Для выполнения команд непосредственно в облаке используйте [Azure Cloud Shell](../../cloud-shell/overview.md).

1. Только локальная CLI. Войдите в Azure с помощью `az login` :

    ```bash
    az login
    ```

    `az login`При необходимости команда открывает окно браузера для сбора учетных данных.

## <a name="acquire-the-object-id"></a>Получение идентификатора объекта

Определите идентификатор объекта приложения, группы или пользователя, которому необходимо назначить политику доступа:

- Приложения и другие субъекты-службы. для получения субъектов-служб используйте команду [AZ AD SP List](/cli/azure/ad/sp#az-ad-sp-list) . Проверьте выходные данные команды, чтобы определить идентификатор объекта субъекта безопасности, которому необходимо назначить политику доступа.

    ```azurecli-interactive
    az ad sp list --show-mine
    ```

- Группы: используйте команду [AZ AD Group List](/cli/azure/ad/group#az-ad-group-list) , чтобы отфильтровать результаты с помощью `--display-name` параметра:

     ```azurecli-interactive
    az ad group list --display-name <search-string>
    ```

- Пользователи: используйте команду [AZ AD user показ](/cli/azure/ad/user#az-ad-user-show) , передав адрес электронной почты пользователя в `--id` параметре:

    ```azurecli-interactive
    az ad user show --id <email-address-of-user>
    ```

## <a name="assign-the-access-policy"></a>Назначение политики доступа
    
Чтобы назначить нужные разрешения, используйте команду [AZ keyvault Set-Policy](/cli/azure/keyvault#az-keyvault-set-policy) :

```azurecli-interactive
az keyvault set-policy --name myKeyVault --object-id <object-id> --secret-permissions <secret-permissions> --key-permissions <key-permissions> --certificate-permissions <certificate-permissions>
```

Замените `<object-id>` идентификатором объекта субъекта-службы.

Необходимо включать только `--secret-permissions` , `--key-permissions` и `--certificate-permissions` при назначении разрешений для этих конкретных типов. Допустимые значения для параметров `<secret-permissions>` , `<key-permissions>` и `<certificate-permissions>` приведены в документации [AZ keyvault Set-Policy](/cli/azure/keyvault#az-keyvault-set-policy) .

## <a name="next-steps"></a>Дальнейшие действия

- [Azure Key Vault security: Identity and access management](security-overview.md#identity-management) (Azure Key Vault security: управление удостоверениями и доступом)
- [Обеспечьте безопасность хранилища ключей](secure-your-key-vault.md).
- [Руководство разработчика Azure Key Vault](developers-guide.md)