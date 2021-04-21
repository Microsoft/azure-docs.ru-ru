---
title: Настройка управляемых удостоверений на виртуальной машине Azure с помощью Azure CLI — Azure AD
description: Пошаговые инструкции по настройке управляемых удостоверений, назначаемых системой и пользователем, на виртуальной машине Azure с помощью Azure CLI.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/15/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: a8dbe9665612e888b7e7afe95a472ba6b0de8d48
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107762521"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-vm-using-azure-cli"></a>Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью Azure CLI

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Функция управляемых удостоверений для ресурсов Azure предоставляет службам Azure автоматически управляемые удостоверения в Azure Active Directory. Это удостоверение можно использовать для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, не храня какие-либо учетные данные в коде. 

В этой статье вы узнаете, как с помощью Azure CLI выполнять приведенные ниже операции с управляемыми удостоверениями для ресурсов Azure на виртуальной машине Azure.

- Включение и отключение управляемого удостоверения, назначаемого системой, на виртуальной машине Azure
- Добавление и удаление управляемого удостоверения, назначаемого пользователем, в виртуальной машине Azure

Если у вас нет учетной записи Azure, [зарегистрируйтесь для получения бесплатной пробной учетной записи](https://azure.microsoft.com/free/), прежде чем продолжать.

## <a name="prerequisites"></a>Обязательные условия

- Если вы не работали с управляемыми удостоверениями для ресурсов Azure, см. статью [Что такое управляемые удостоверения для ресурсов Azure](overview.md). Сведения об удостоверениях, назначаемых системой и назначаемых пользователем, см. в разделе [Типы управляемых удостоверений](overview.md#managed-identity-types).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="system-assigned-managed-identity"></a>Управляемое удостоверение, назначаемое системой

В этом разделе вы узнаете, как включить и отключить управляемое удостоверение, назначаемое системой, на виртуальной машине Azure с помощью Azure CLI.

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-vm"></a>Включение управляемого удостоверения, назначаемого системой, во время создания виртуальной машины Azure

Чтобы создать виртуальную машину Azure с включенным управляемым удостоверением, назначаемым системой, вашей учетной записи должна быть назначена роль [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Назначать другие роли в каталоге Azure AD не требуется.

1. Создайте [группу ресурсов](../../azure-resource-manager/management/overview.md#terminology) с помощью параметра [az group create](/cli/azure/group/#az_group_create), чтобы сохранить и развернуть виртуальную машину и связанные с ней ресурсы. Если вы уже создали группу ресурсов, которую можно использовать, этот шаг можно пропустить:

   ```azurecli-interactive 
   az group create --name myResourceGroup --location westus
   ```

1. Создайте виртуальную машину, выполнив команду [az vm create](/cli/azure/vm/#az_vm_create). В приведенном ниже примере создается виртуальная машина *myVM* с управляемым удостоверением, назначаемым системой, как указывает параметр `--assign-identity`. В параметрах `--admin-username` и `--admin-password` определяются имя и пароль учетной записи администратора для входа в виртуальную машину. Подставьте соответствующие значения для своей среды: 

   ```azurecli-interactive 
   az vm create --resource-group myResourceGroup --name myVM --image win2016datacenter --generate-ssh-keys --assign-identity --admin-username azureuser --admin-password myPassword12
   ```

### <a name="enable-system-assigned-managed-identity-on-an-existing-azure-vm"></a>Включение управляемого удостоверения, назначаемого системой, для имеющейся виртуальной машины Azure

Чтобы включить назначаемое системой управляемое удостоверение на виртуальной машине, вашей учетной записи должна быть назначена роль [Участника виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Назначать другие роли в каталоге Azure AD не требуется.

1. Если вы используете Azure CLI в локальной консоли, сначала выполните вход в Azure с помощью команды [az login](/cli/azure/reference-index#az_login). Используйте учетную запись, связанную с подпиской Azure, которая содержит виртуальную машину.

   ```azurecli-interactive
   az login
   ```

2. Чтобы включить удостоверение, назначаемое системой на имеющейся виртуальной машине, используйте команду [az vm identity assign](/cli/azure/vm/identity/) с параметром `identity assign`.

   ```azurecli-interactive
   az vm identity assign -g myResourceGroup -n myVm
   ```

### <a name="disable-system-assigned-identity-from-an-azure-vm"></a>Отключение удостоверения, назначаемого системой, на виртуальной машине Azure

Чтобы отключить назначаемое системой управляемое удостоверение на виртуальной машине, вашей учетной записи должна быть назначена роль [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Назначать другие роли в каталоге Azure AD не требуется.

Если на виртуальной машине больше не требуется удостоверение, назначаемое системой, но по-прежнему требуются удостоверения, назначаемые пользователем, используйте следующую команду.

```azurecli-interactive
az vm update -n myVM -g myResourceGroup --set identity.type='UserAssigned' 
```

Если имеется виртуальная машина, для которой не требуется удостоверение, назначаемое системой, и на которой нет удостоверений, назначаемых пользователем, используйте следующие команды.

> [!NOTE]
> В значении `none` учитывается регистр. Оно должно содержать строчные буквы. 

```azurecli-interactive
az vm update -n myVM -g myResourceGroup --set identity.type="none"
```


## <a name="user-assigned-managed-identity"></a>Управляемое удостоверение, назначаемое пользователем

В этом разделе вы узнаете, как добавить и удалить управляемое удостоверение, назначаемое пользователем, для виртуальной машины Azure с помощью Azure CLI. Вы можете создать управляемое удостоверение, назначаемое пользователем, в группе ресурсов, отличающейся от той, в которую включена виртуальная машина. Чтобы назначить его виртуальной машине, используйте URL-адрес управляемого удостоверения.
Например, --identities "/subscriptions/<SUBID>/resourcegroups/<RESROURCEGROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER_ASSIGNED_ID_NAME>"

### <a name="assign-a-user-assigned-managed-identity-during-the-creation-of-an-azure-vm"></a>Задание управляемого удостоверения, назначаемого пользователем, во время создания виртуальной машины Azure

Чтобы назначить назначаемое пользователем удостоверение виртуальной машине во время ее создания, вашей учетной записи должны быть назначены роли [Участника виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) и [Оператора управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-operator). Назначать другие роли в каталоге Azure AD не требуется.

1. Если вы уже создали группу ресурсов, которую можно использовать, этот шаг можно пропустить. Создайте [группу ресурсов](~/articles/azure-resource-manager/management/overview.md#terminology) для хранения и развертывания управляемого удостоверения, назначаемого пользователем, используя команду [az group create](/cli/azure/group/#az_group_create). Не забудьте заменить значения параметров `<RESOURCE GROUP>` и `<LOCATION>` собственными. :

   ```azurecli-interactive 
   az group create --name <RESOURCE GROUP> --location <LOCATION>
   ```

2. Создайте управляемое удостоверение, назначаемое пользователем, с помощью команды [az identity create](/cli/azure/identity#az_identity_create).  Параметр `-g` указывает группу ресурсов, в которой создается управляемое удостоверение, назначаемое пользователем, а параметр `-n` — его имя.    
    
   [!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

   ```azurecli-interactive
   az identity create -g myResourceGroup -n myUserAssignedIdentity
   ```
   Ответ содержит подробные сведения о созданном управляемом удостоверении, назначаемом пользователем, как показано ниже. Значение идентификатора ресурса, присвоенное управляемому удостоверению, назначаемому пользователем, используется на следующем шаге.

   ```json
   {
       "clientId": "73444643-8088-4d70-9532-c3a0fdc190fz",
       "clientSecretUrl": "https://control-westcentralus.identity.azure.net/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<myUserAssignedIdentity>/credentials?tid=5678&oid=9012&aid=73444643-8088-4d70-9532-c3a0fdc190fz",
       "id": "/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>",
       "location": "westcentralus",
       "name": "<USER ASSIGNED IDENTITY NAME>",
       "principalId": "e5fdfdc1-ed84-4d48-8551-fe9fb9dedfll",
       "resourceGroup": "<RESOURCE GROUP>",
       "tags": {},
       "tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533bl",
       "type": "Microsoft.ManagedIdentity/userAssignedIdentities"    
   }
   ```

3. Создайте виртуальную машину, выполнив команду [az vm create](/cli/azure/vm/#az_vm_create). В приведенном ниже примере создается виртуальная машина, связанная с новым удостоверением, назначаемым пользователем, в соответствии с параметром `--assign-identity`. Не забудьте заменить значения параметров `<RESOURCE GROUP>`, `<VM NAME>`, `<USER NAME>`, `<PASSWORD>` и `<USER ASSIGNED IDENTITY NAME>` своими значениями. 

   ```azurecli-interactive 
   az vm create --resource-group <RESOURCE GROUP> --name <VM NAME> --image UbuntuLTS --admin-username <USER NAME> --admin-password <PASSWORD> --assign-identity <USER ASSIGNED IDENTITY NAME>
   ```

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-azure-vm"></a>Задание управляемого удостоверения, назначаемого пользователем, для имеющейся виртуальной машины Azure

Чтобы присвоить назначаемое пользователем удостоверение виртуальной машине, вашей учетной записи должны быть назначены роли [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) и [Оператор управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-operator). Назначать другие роли в каталоге Azure AD не требуется.

1. Создайте пользовательское удостоверение с помощью команды [az identity create](/cli/azure/identity#az_identity_create).  Параметр `-g` указывает группу ресурсов, в которой создается удостоверение, назначаемое пользователем, а параметр `-n` — его имя. Не забудьте заменить значения параметров `<RESOURCE GROUP>` и `<USER ASSIGNED IDENTITY NAME>` собственными:

    > [!IMPORTANT]
    > При создании управляемых удостоверений, назначаемых пользователем, не поддерживается использование специальных знаков (например, символа подчеркивания) в имени. Используйте буквенно-цифровые символы. Загляните сюда позже, чтобы проверить наличие новой информации.  Дополнительные сведения см. в разделе [Часто задаваемые вопросы и известные проблемы](known-issues.md).

    ```azurecli-interactive
    az identity create -g <RESOURCE GROUP> -n <USER ASSIGNED IDENTITY NAME>
    ```
   Ответ содержит подробные сведения о созданном управляемом удостоверении, назначаемом пользователем, как показано ниже. 

   ```json
   {
        "clientId": "73444643-8088-4d70-9532-c3a0fdc190fz",
        "clientSecretUrl": "https://control-westcentralus.identity.azure.net/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>/credentials?tid=5678&oid=9012&aid=73444643-8088-4d70-9532-c3a0fdc190fz",
        "id": "/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>",
        "location": "westcentralus",
        "name": "<USER ASSIGNED IDENTITY NAME>",
        "principalId": "e5fdfdc1-ed84-4d48-8551-fe9fb9dedfll",
        "resourceGroup": "<RESOURCE GROUP>",
        "tags": {},
        "tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533bl",
        "type": "Microsoft.ManagedIdentity/userAssignedIdentities"    
   }
   ```

2. Задайте удостоверение, назначаемое пользователем, для виртуальной машины с помощью команды [az vm identity assign](/cli/azure/vm). Не забудьте заменить значения параметров `<RESOURCE GROUP>` и `<VM NAME>` собственными. `<USER ASSIGNED IDENTITY NAME>` — это свойство ресурса `name` управляемого удостоверения, назначаемого пользователем, созданного на предыдущем шаге. Вы можете создать управляемое удостоверение, назначаемое пользователем, в группе ресурсов, отличающейся от той, в которую включена виртуальная машина. Используйте URL-адрес управляемого удостоверения.

    ```azurecli-interactive
    az vm identity assign -g <RESOURCE GROUP> -n <VM NAME> --identities <USER ASSIGNED IDENTITY>
    ```

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-vm"></a>Удаление управляемого удостоверения, назначаемого пользователем, из виртуальной машины Azure

Чтобы удалить назначаемое пользователем удостоверение на виртуальной машине, вашей учетной записи должна быть назначена роль [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor). 

Если это единственное управляемое удостоверение, назначаемое пользователем, заданное для виртуальной машины, то `UserAssigned` будет удалено из значения типа удостоверения.  Не забудьте заменить значения параметров `<RESOURCE GROUP>` и `<VM NAME>` собственными. `<USER ASSIGNED IDENTITY>` будет свойством `name` удостоверения, назначаемого пользователем, которое можно найти в разделе удостоверений виртуальной машины, используя `az vm identity show`.

```azurecli-interactive
az vm identity remove -g <RESOURCE GROUP> -n <VM NAME> --identities <USER ASSIGNED IDENTITY>
```

Если в виртуальной машине нет управляемого удостоверения, назначаемого системой, и вы хотите удалить из нее все управляемые удостоверения, назначаемые пользователем, используйте следующую команду.

> [!NOTE]
> В значении `none` учитывается регистр. Оно должно содержать строчные буквы.

```azurecli-interactive
az vm update -n myVM -g myResourceGroup --set identity.type="none" identity.userAssignedIdentities=null
```

Если у виртуальной машины есть как удостоверение, назначаемое системой, так и удостоверения, назначаемые пользователем, вы можете удалить все удостоверения, назначаемые пользователем, переключившись на использование только удостоверения, назначаемого системой. Используйте следующую команду:

```azurecli-interactive
az vm update -n myVM -g myResourceGroup --set identity.type='SystemAssigned' identity.userAssignedIdentities=null 
```

## <a name="next-steps"></a>Дальнейшие действия
- [Обзор управляемых удостоверений для ресурсов Azure](overview.md).
- Ниже приведены комплексные краткие руководства по созданию виртуальных машин Azure: 
  - [Создание виртуальной машины Windows с помощью Azure CLI](../../virtual-machines/windows/quick-create-cli.md)  
  - [Создание виртуальной машины Linux с помощью Azure CLI](../../virtual-machines/linux/quick-create-cli.md)
