---
title: Настройка управляемых удостоверений на виртуальной машине Azure с помощью PowerShell — Azure AD
description: Пошаговые инструкции по настройке управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью Azure.
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
ms.date: 08/19/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1eca17106067d964ba2f280d358b2973b41459a6
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783275"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-vm-using-powershell"></a>Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью PowerShell

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Функция управляемых удостоверений для ресурсов Azure предоставляет службам Azure автоматически управляемые удостоверения в Azure Active Directory. Это удостоверение можно использовать для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, не храня какие-либо учетные данные в коде.

В этой статье вы узнаете, как с помощью PowerShell выполнять приведенные ниже операции с управляемыми удостоверениями для ресурсов Azure на виртуальной машине Azure.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

- Если вы не работали с управляемыми удостоверениями для ресурсов Azure, изучите [общие сведения](overview.md). **Обратите внимание на [различие между управляемыми удостоверениями, назначаемыми системой и назначаемыми пользователями](overview.md#managed-identity-types)**.
- Если у вас нет учетной записи Azure, [зарегистрируйтесь для получения бесплатной пробной учетной записи](https://azure.microsoft.com/free/), прежде чем продолжать.
- Выполнить примеры скриптов можно двумя способами:
    - используйте службу [Azure Cloud Shell](../../cloud-shell/overview.md), которую можно открыть с помощью кнопки **Попробовать** в правом верхнем углу блоков кода.
    - Выполните скрипты локально, установив последнюю версию [Azure PowerShell](/powershell/azure/install-az-ps), а затем войдите в Azure с помощью команды `Connect-AzAccount`. 

## <a name="system-assigned-managed-identity"></a>Управляемое удостоверение, назначаемое системой

В этом разделе вы узнаете, как включить и отключить управляемое удостоверение, назначаемое системой, с помощью Azure PowerShell.

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-vm"></a>Включение управляемого удостоверения, назначаемого системой, во время создания виртуальной машины Azure

Чтобы создать виртуальную машину Azure с включенным управляемым удостоверением, назначаемым системой, вашей учетной записи должна быть назначена роль [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Назначать другие роли в каталоге Azure AD не требуется.

1. Используя одно из кратких руководств ниже, выполните действия только в нужных разделах ("Вход в Azure", "Создание группы ресурсов", "Создание группы сети", "Создание виртуальной машины").

    При выполнении действий, описанных в разделе о создании виртуальной машины, не забудьте внести небольшие изменения в синтаксис командлета [New-AzVMConfig](/powershell/module/az.compute/new-azvm). Чтобы подготовить виртуальную машину с включенным удостоверением, назначаемым системой, добавьте параметр `-IdentityType SystemAssigned`, как показано ниже.

    ```azurepowershell-interactive
    $vmConfig = New-AzVMConfig -VMName myVM -IdentityType SystemAssigned ...
    ```

   - [Создание виртуальной машины Windows с помощью PowerShell](../../virtual-machines/windows/quick-create-powershell.md)
   - [Создание виртуальной машины Linux с помощью PowerShell](../../virtual-machines/linux/quick-create-powershell.md)

### <a name="enable-system-assigned-managed-identity-on-an-existing-azure-vm"></a>Включение управляемого удостоверения, назначаемого системой, для имеющейся виртуальной машины Azure

Чтобы включить назначаемое системой управляемое удостоверение на виртуальной машине, которая была подготовлена без него, вашей учетной записи должна быть назначена роль [Участника виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Назначать другие роли в каталоге Azure AD не требуется.

1. Извлеките свойства виртуальной машины с помощью командлета `Get-AzVM`. Затем, чтобы включить управляемое удостоверение, назначаемое системой, используйте параметр `-IdentityType` в командлете [Update-AzVM](/powershell/module/az.compute/update-azvm).

   ```azurepowershell-interactive
   $vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
   Update-AzVM -ResourceGroupName myResourceGroup -VM $vm -IdentityType SystemAssigned
   ```

### <a name="add-vm-system-assigned-identity-to-a-group"></a>Добавление назначаемого системой удостоверения виртуальной машины в группу

После включения назначенного системной удостоверения на виртуальной машине, вы можете добавить его в группу.  Следующая процедура добавляет назначенное системой удостоверение в группу.

1. Получите и запишите `ObjectID` (как указано в поле `Id` возвращаемых значений) субъекта-службы виртуальной машины.

   ```azurepowershell-interactive
   Get-AzADServicePrincipal -displayname "myVM"
   ```

1. Получите и запишите `ObjectID` (как указано в поле `Id` возвращаемых значений) группы.

   ```azurepowershell-interactive
   Get-AzADGroup -searchstring "myGroup"
   ```

1. Добавление субъекта-службы виртуальной машины в группу.

   ```azurepowershell-interactive
   Add-AzureADGroupMember -ObjectId "<objectID of group>" -RefObjectId "<object id of VM service principal>"
   ```

## <a name="disable-system-assigned-managed-identity-from-an-azure-vm"></a>Отключение управляемого удостоверения, назначаемого системой, на виртуальной машине Azure

Чтобы отключить назначаемое системой управляемое удостоверение на виртуальной машине, вашей учетной записи должна быть назначена роль [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Назначать другие роли в каталоге Azure AD не требуется.

Если на виртуальной машине больше не требуется управляемое удостоверение, назначаемое системой, но по-прежнему требуются управляемые удостоверения, назначаемые пользователем, используйте следующий командлет.

1. Получите свойства виртуальной машины с помощью командлета `Get-AzVM` и задайте для параметра `-IdentityType` значение `UserAssigned`.

   ```azurepowershell-interactive
   $vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
   Update-AzVm -ResourceGroupName myResourceGroup -VM $vm -IdentityType "UserAssigned"
   ```

Если имеется виртуальная машина, для которой не требуется управляемое удостоверение, назначаемое системой, и у которой нет управляемых удостоверений, назначаемых пользователем, используйте следующие команды.

```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
Update-AzVm -ResourceGroupName myResourceGroup -VM $vm -IdentityType None
```

## <a name="user-assigned-managed-identity"></a>Управляемое удостоверение, назначаемое пользователем

В этом разделе вы узнаете, как добавить и удалить управляемое удостоверение, назначаемое пользователем, для виртуальной машины с помощью Azure PowerShell.

### <a name="assign-a-user-assigned-managed-identity-to-a-vm-during-creation"></a>Задание управляемого удостоверения, назначаемого пользователем, для виртуальной машины во время ее создания

Чтобы присвоить назначаемое пользователем удостоверение виртуальной машине, вашей учетной записи должны быть назначены роли [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) и [Оператор управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-operator). Назначать другие роли в каталоге Azure AD не требуется.

1. Используя одно из кратких руководств ниже, выполните действия только в нужных разделах ("Вход в Azure", "Создание группы ресурсов", "Создание группы сети", "Создание виртуальной машины").

    При выполнении действий, описанных в разделе о создании виртуальной машины, не забудьте внести небольшие изменения в синтаксис командлета [`New-AzVMConfig`](/powershell/module/az.compute/new-azvm). Добавьте параметры `-IdentityType UserAssigned` и `-IdentityID`, чтобы подготовить виртуальную машину с удостоверением, назначаемым пользователем.  Замените `<VM NAME>`,`<SUBSCRIPTION ID>`, `<RESROURCE GROUP>` и `<USER ASSIGNED IDENTITY NAME>` собственными значениями.  Пример:

    ```azurepowershell-interactive
    $vmConfig = New-AzVMConfig -VMName <VM NAME> -IdentityType UserAssigned -IdentityID "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESROURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>..."
    ```

    - [Создание виртуальной машины Windows с помощью PowerShell](../../virtual-machines/windows/quick-create-powershell.md)
    - [Создание виртуальной машины Linux с помощью PowerShell](../../virtual-machines/linux/quick-create-powershell.md)


### <a name="assign-a-user-assigned-managed-identity-to-an-existing-azure-vm"></a>Задание управляемого удостоверения, назначаемого пользователем, для имеющейся виртуальной машины Azure

Чтобы присвоить назначаемое пользователем удостоверение виртуальной машине, вашей учетной записи должны быть назначены роли [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) и [Оператор управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-operator). Назначать другие роли в каталоге Azure AD не требуется.

1. Создайте управляемое удостоверение, назначаемое пользователем, с помощью командлета [New-AzUserAssignedIdentity](/powershell/module/az.managedserviceidentity/new-azuserassignedidentity).  Обратите внимание на свойство `Id` в выходных данных. Оно потребуется в следующем шаге.

   > [!IMPORTANT]
   > При создании управляемых удостоверений, назначаемых пользователем, можно использовать только буквы, цифры, знак подчеркивания и дефис (0–9, a–z, A–Z, \_ или -). Кроме того, чтобы назначение виртуальной машине или масштабируемому набору виртуальных машин производилось правильно, длина имени должна составлять 3–128 символов. Дополнительные сведения см. в разделе [Часто задаваемые вопросы и известные проблемы](known-issues.md).

   ```azurepowershell-interactive
   New-AzUserAssignedIdentity -ResourceGroupName <RESOURCEGROUP> -Name <USER ASSIGNED IDENTITY NAME>
   ```
1. Извлеките свойства виртуальной машины с помощью командлета `Get-AzVM`. Затем, чтобы задать управляемое удостоверение, назначаемое пользователем, для виртуальной машины Azure, используйте параметры `-IdentityType` и `-IdentityID` в командлете [Update-AzVM](/powershell/module/az.compute/update-azvm).  Для параметра `-IdentityId` следует использовать значение `Id` из предыдущего шага.  Замените `<VM NAME>`, `<SUBSCRIPTION ID>`, `<RESROURCE GROUP>` и `<USER ASSIGNED IDENTITY NAME>` собственными значениями.

   > [!WARNING]
   > Чтобы сохранить ранее назначенные пользователем управляемые удостоверения, предназначенные для виртуальной машины, запросите свойство `Identity` объекта VM (например, `$vm.Identity`).  Если какие-либо назначенные пользователем управляемые удостоверения возвращаются, включите их в следующую команду вместе с новым удостоверением, которое необходимо назначить виртуальной машине.

   ```azurepowershell-interactive
   $vm = Get-AzVM -ResourceGroupName <RESOURCE GROUP> -Name <VM NAME>
   Update-AzVM -ResourceGroupName <RESOURCE GROUP> -VM $vm -IdentityType UserAssigned -IdentityID "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESROURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>"
   ```

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-vm"></a>Удаление управляемого удостоверения, назначаемого пользователем, из виртуальной машины Azure

Чтобы удалить назначаемое пользователем удостоверение на виртуальной машине, вашей учетной записи должна быть назначена роль [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).

Если у виртуальной машины несколько управляемых удостоверений, назначаемых пользователем, с помощью приведенных ниже команд можно удалить все эти удостоверения, кроме последнего. Не забудьте заменить значения параметров `<RESOURCE GROUP>` и `<VM NAME>` собственными. `<USER ASSIGNED IDENTITY NAME>` — это имя управляемого удостоверения, назначаемого пользователем, которое следует оставить в виртуальной машине. Эти сведения можно найти, запросив свойство `Identity` объекта виртуальной машины.  Например, `$vm.Identity`:

```azurepowershell-interactive
$vm = Get-AzVm -ResourceGroupName myResourceGroup -Name myVm
Update-AzVm -ResourceGroupName myResourceGroup -VirtualMachine $vm -IdentityType UserAssigned -IdentityID <USER ASSIGNED IDENTITY NAME>
```
Если в виртуальной машине нет управляемого удостоверения, назначаемого системой, и вы хотите удалить из нее все управляемые удостоверения, назначаемые пользователем, используйте следующую команду.

```azurepowershell-interactive
$vm = Get-AzVm -ResourceGroupName myResourceGroup -Name myVm
Update-AzVm -ResourceGroupName myResourceGroup -VM $vm -IdentityType None
```
Если у виртуальной машины есть управляемые удостоверения, назначаемые системой и назначаемые пользователем, вы можете удалить все управляемые удостоверения, назначаемые пользователем, переключившись на использование только управляемого удостоверения, назначаемого системой.

```azurepowershell-interactive
$vm = Get-AzVm -ResourceGroupName myResourceGroup -Name myVm
Update-AzVm -ResourceGroupName myResourceGroup -VirtualMachine $vm -IdentityType "SystemAssigned"
```

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор управляемых удостоверений для ресурсов Azure](overview.md).
- Ниже приведены комплексные краткие руководства по созданию виртуальных машин Azure:

  - [Создание виртуальной машины Windows с помощью PowerShell](../../virtual-machines/windows/quick-create-powershell.md)
  - [Создание виртуальной машины Linux с помощью PowerShell](../../virtual-machines/linux/quick-create-powershell.md)
