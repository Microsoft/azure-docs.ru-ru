---
title: Управление плоскостью данных для Управляемого устройства HSM (Azure Key Vault) | Документация Майкрософт
description: Эта статья поможет вам в управлении назначениями ролей для управляемого устройства HSM.
services: key-vault
author: amitbapat
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: a4cc898744109475bc119f37350d1b689c550f58
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102209566"
---
# <a name="managed-hsm-role-management"></a>Управление ролями в службе "Управляемое устройство HSM"

> [!NOTE]
> Key Vault поддерживает два типа ресурсов: хранилища и управляемые устройства HSM. Эта статья посвящена **управляемому устройству HSM**. Если вы хотите узнать, как управлять хранилищем, см. статью [Управление Key Vault с помощью Azure CLI](../general/manage-with-cli2.md).

Общие сведения о службе "Управляемое устройство HSM" см. в статье [Что собой представляет служба "Управляемое устройство HSM"?](overview.md) Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

В этой статье описывается, как управлять ролями для плоскости данных Управляемого устройства HSM. Дополнительные сведения о модели контроля доступа к Управляемому устройству HSM см. [здесь](access-control.md).

Чтобы субъект безопасности (будь то пользователь, субъект-служба, группа или управляемое удостоверение) мог выполнять операции плоскости данных на управляемом устройстве HSM, ему должна быть назначена роль с разрешениями на выполнение этих операций. Например, если ваше приложение должно выполнять операцию подписывания с помощью ключа, ему должна быть присвоена роль, которая содержит действие с данными Microsoft.KeyVault/managedHSM/keys/sign/action. Роль назначается в определенной области действия. Локальная система RBAC для управляемых устройств HSM поддерживает две области: для всего HSM (`/` или `/keys`) и для конкретного ключа (`/keys/<keyname>`).

Полный список встроенных ролей Управляемого устройства HSM и операций, которые они позволяют выполнять, см. [здесь](built-in-roles.md).

## <a name="prerequisites"></a>Предварительные требования

Чтобы использовать команды Azure CLI из этой строки, вам необходимо следующее:

* подписка на Microsoft Azure. Если у вас ее нет, зарегистрируйтесь, чтобы воспользоваться [бесплатной пробной версией](https://azure.microsoft.com/pricing/free-trial).
* Azure CLI 2.12.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI]( /cli/azure/install-azure-cli).
* Управляемое устройство HSM в подписке. См. [Краткое руководство. Подготовка и активация управляемого устройства HSM с помощью Azure CLI](quick-create-cli.md) для выполнения соответствующих действий.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="sign-in-to-azure"></a>Вход в Azure

Чтобы войти в Azure с помощью CLI, введите следующее:

```azurecli
az login
```

Дополнительные сведения о вариантах входа с помощью Azure CLI см. в [этой статье](/cli/azure/authenticate-azure-cli).

## <a name="create-a-new-role-assignment"></a>Создание нового назначения роли

### <a name="assign-roles-for-all-keys"></a>Назначение ролей для всех ключей

Команда `az keyvault role assignment create` позволяет назначить роль **Managed HSM Crypto Officer** (Специалист по криптографии для управляемого устройства HSM) некоторому пользователю, определяемого именем участника-пользователя **user2\@contoso.com**, для всех **ключей** (область действия `/keys`) в ContosoHSM.

```azurecli-interactive
az keyvault role assignment create --hsm-name ContosoMHSM --role "Managed HSM Crypto Officer" --assignee user2@contoso.com  --scope /keys
```

### <a name="assign-role-for-a-specific-key"></a>Назначение роли для конкретного ключа

Команда `az keyvault role assignment create` позволяет назначить роль **Managed HSM Crypto Officer** (Специалист по криптографии для Управляемого устройства HSM) любому пользователю, определяемому именем участника-пользователя **user2\@contoso.com**, для конкретного ключа с именем **myrsakey**.

```azurecli-interactive
az keyvault role assignment create --hsm-name ContosoMHSM --role "Managed HSM Crypto Officer" --assignee user2@contoso.com  --scope /keys/myrsakey
```

## <a name="list-existing-role-assignments"></a>Перечисление существующих назначений ролей

Команда `az keyvault role assignment list` возвращает список назначений ролей.

Все назначения ролей в области (это вариант по умолчанию, если не указан аргумент --scope) для всех пользователей (вариант по умолчанию, если не указан аргумент --assignee)

```azurecli-interactive
az keyvault role assignment list --hsm-name ContosoMHSM
```

Все назначения ролей на уровне HSM для конкретного пользователя **user1@contoso.com** .

```azurecli-interactive
az keyvault role assignment list --hsm-name ContosoMHSM --assignee user@contoso.com
```

> [!NOTE]
> Если для области указан символ / (или /keys), команда list перечисляет все назначения ролей на верхнем уровне и не выводит назначения ролей на уровне отдельных ключей.

Все назначения ролей для конкретного пользователя **user2@contoso.com** для конкретного ключа **myrsakey**.

```azurecli-interactive
az keyvault role assignment list --hsm-name ContosoMHSM --assignee user2@contoso.com --scope /keys/myrsakey
```

Конкретное назначение роли **Managed HSM Crypto Officer** (Специалист по криптографии для Управляемого устройства HSM) для конкретного пользователя **user2@contoso.com** для конкретного ключа **myrsakey**.


```azurecli-interactive
az keyvault role assignment list --hsm-name ContosoMHSM --assignee user2@contoso.com --scope /keys/myrsakey --role "Managed HSM Crypto Officer"
```

## <a name="delete-a-role-assignment"></a>Удаление назначения ролей

Команда `az keyvault role assignment delete` позволяет удалить роль **Managed HSM Crypto Officer** (Специалист по криптографии для Управляемого устройства HSM), назначенную пользователю **user2\@contoso.com** для ключа **user2contoso**.

```azurecli-interactive
az keyvault role assignment delete --hsm-name ContosoMHSM --role "Managed HSM Crypto Officer" --assignee user2@contoso.com  --scope /keys/myrsakey2
```

## <a name="list-all-available-role-definitions"></a>Список всех доступных определений ролей

Команда `az keyvault role definition list` возвращает список всех определений ролей.

```azurecli-interactive
az keyvault role definition list --hsm-name ContosoMHSM
```

## <a name="next-steps"></a>Дальнейшие действия

- См. общие сведения об [управлении доступом на основе ролей в Azure (Azure RBAC)](../../role-based-access-control/overview.md).
- См. руководство [Управление ролями в службе "Управляемое устройство HSM"](role-management.md).
- Узнайте больше о [модели контроля доступом в службе "Управляемое устройство HSM"](access-control.md).
- Ознакомьтесь со [списком встроенных ролей в локальной системе RBAC службы "Управляемое устройство HSM"](built-in-roles.md).
