---
title: Настройка управляемых удостоверений в масштабируемом наборе виртуальных машин Azure с помощью PowerShell — Azure AD
description: Пошаговые инструкции по настройке управляемых удостоверений, назначаемых системой и назначаемых пользователем, в масштабируемом наборе виртуальных машин с помощью PowerShell.
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
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: eb651df2fe3b9771154f9e188d41c660f7ffd028
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107774151"
---
# <a name="configure-managed-identities-for-azure-resources-on-virtual-machine-scale-sets-using-powershell"></a>Настройка управляемых удостоверений для ресурсов Azure в масштабируемых наборах виртуальных машин с помощью PowerShell

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Функция управляемых удостоверений для ресурсов Azure предоставляет службам Azure автоматически управляемые удостоверения в Azure Active Directory. Это удостоверение можно использовать для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, не храня какие-либо учетные данные в коде. 

В этой статье вы узнаете, как с помощью PowerShell выполнять операции с управляемыми удостоверениями для ресурсов Azure в масштабируемом наборе виртуальных машин.

- Включение и отключение управляемого удостоверения, назначаемого системой, в масштабируемом наборе виртуальных машин
- Добавление и удаление управляемого удостоверения, назначаемого пользователем, в масштабируемом наборе виртуальных машин

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

- Если вы не работали с управляемыми удостоверениями для ресурсов Azure, изучите [общие сведения](overview.md). **Обратите внимание на [различие между управляемыми удостоверениями, назначаемыми системой и назначаемыми пользователями](overview.md#managed-identity-types)**.

- Если у вас нет учетной записи Azure, [зарегистрируйтесь для получения бесплатной пробной учетной записи](https://azure.microsoft.com/free/), прежде чем продолжать.

- Для выполнения операций управления, описанных в этой статье, учетной записи требуются следующие назначения управления доступом на основе ролей Azure:

    > [!NOTE]
    > Дополнительные назначения ролей в каталоге Azure AD не требуются.

    - [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) для создания масштабируемого набора виртуальных машин, а также для включения и удаления управляемого удостоверения, назначаемого системой, и (или) управляемого удостоверения, назначаемого пользователем, из масштабируемого набора виртуальных машин.
    - Роль [Участник управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-contributor): для создания управляемого удостоверения, назначаемого пользователем.
    - Роль [Оператор управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-operator): для назначения и удаления управляемого удостоверения, назначаемого пользователем, в масштабируемом наборе виртуальных машин.

- Выполнить примеры скриптов можно двумя способами:
    - используйте службу [Azure Cloud Shell](../../cloud-shell/overview.md), которую можно открыть с помощью кнопки **Попробовать** в правом верхнем углу блоков кода.
    - Выполните скрипты локально, установив последнюю версию [Azure PowerShell](/powershell/azure/install-az-ps), а затем войдите в Azure с помощью команды `Connect-AzAccount`. 

## <a name="system-assigned-managed-identity"></a>Управляемое удостоверение, назначаемое системой

В этом разделе вы узнаете, как включить и удалить управляемое удостоверение, назначаемое системой, с помощью Azure PowerShell.

### <a name="enable-system-assigned-managed-identity-during-the-creation-of-an-azure-virtual-machine-scale-set"></a>Включение управляемого удостоверения, назначаемого системой, во время создания масштабируемого набора виртуальных машин Azure

Чтобы создать масштабируемый набор виртуальных машин с включенным управляемым удостоверением, назначаемым системой, сделайте следующее.

1. Инструкции по созданию масштабируемого набора виртуальных машин с управляемым удостоверением, назначаемым системой, приведены в *примере 1* в справочной статье по командлету [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig).  Добавьте параметр `-IdentityType SystemAssigned` в командлет `New-AzVmssConfig`:

    ```azurepowershell-interactive
    $VMSS = New-AzVmssConfig -Location $Loc -SkuCapacity 2 -SkuName "Standard_A0" -UpgradePolicyMode "Automatic" -NetworkInterfaceConfiguration $NetCfg -IdentityType SystemAssigned`
    ```

### <a name="enable-system-assigned-managed-identity-on-an-existing-azure-virtual-machine-scale-set"></a>Включение управляемого удостоверения, назначаемого системой, в существующем масштабируемом наборе виртуальных машин Azure

Если нужно включить управляемое удостоверение, назначаемое системой, в существующем масштабируемом наборе виртуальных машин Azure, сделайте следующее.

1. Убедитесь, что используемая учетная запись принадлежит роли, которая предоставляет разрешения на запись в масштабируемом наборе виртуальных машин, например "Участник виртуальных машин".
   
1. Получите свойства масштабируемого набора виртуальных машин с помощью командлета [`Get-AzVmss`](/powershell/module/az.compute/get-azvmss). Затем, чтобы включить управляемое удостоверение, назначаемое системой, используйте параметр `-IdentityType` в командлете [Update-AzVmss](/powershell/module/az.compute/update-azvmss).

   ```azurepowershell-interactive
   Update-AzVmss -ResourceGroupName myResourceGroup -Name -myVmss -IdentityType "SystemAssigned"
   ```

### <a name="disable-the-system-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>Отключение управляемого удостоверения, назначаемого системой, в масштабируемом наборе виртуальных машин Azure

Если в масштабируемом наборе виртуальных машин не требуется управляемое удостоверение, назначаемое системой, но по-прежнему требуются управляемые удостоверения, назначаемые пользователем, используйте следующий командлет.

1. Убедитесь, что учетная запись принадлежит роли, которая предоставляет разрешения на запись в масштабируемом наборе виртуальных машин, например "Участник виртуальных машин".

1. Выполните следующий командлет:

   ```azurepowershell-interactive
   Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType "UserAssigned"
   ```

1. При наличии масштабируемого набора виртуальных машин, для которого не требуется управляемое удостоверение, назначаемое системой, и у которого нет управляемых удостоверений, назначаемых пользователем, используйте следующую команду:

    ```azurepowershell-interactive
    Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType None
    ```
    
## <a name="user-assigned-managed-identity"></a>Управляемое удостоверение, назначаемое пользователем

В этом разделе вы узнаете, как добавить и удалить управляемое удостоверение, назначаемое пользователем, в масштабируемом наборе виртуальных машин с помощью Azure PowerShell.

### <a name="assign-a-user-assigned-managed-identity-during-creation-of-an-azure-virtual-machine-scale-set"></a>Задание управляемого удостоверения, назначаемого пользователем, во время создания масштабируемого набора виртуальных машин Azure

Сейчас не поддерживается создание масштабируемого набора виртуальных машин с управляемым удостоверением, назначаемым пользователем, с помощью PowerShell. Сведения о добавлении управляемого удостоверения, назначаемого пользователем, в существующий масштабируемый набор виртуальных машин приведены в следующем разделе. Загляните сюда позже, чтобы проверить наличие новой информации.

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-azure-virtual-machine-scale-set"></a>Задание управляемого удостоверения, назначаемого пользователем, для существующего масштабируемого набора виртуальных машин Azure

Чтобы задать управляемое удостоверение, назначаемое пользователем, для существующего масштабируемого набора виртуальных машин Azure, выполните следующее.

1. Убедитесь, что учетная запись принадлежит роли, которая предоставляет разрешения на запись в масштабируемом наборе виртуальных машин, например "Участник виртуальных машин".

1. Получите свойства масштабируемого набора виртуальных машин с помощью командлета `Get-AzVM`. Затем, чтобы задать управляемое удостоверение, назначаемое пользователем, для масштабируемого набора виртуальных машин, используйте параметры `-IdentityType` и `-IdentityID` в командлете [Update-AzVmss](/powershell/module/az.compute/update-azvmss). Замените `<VM NAME>`, `<SUBSCRIPTION ID>`, `<RESROURCE GROUP>`, `<USER ASSIGNED ID1>`, `USER ASSIGNED ID2` собственными значениями.

   [!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

   ```azurepowershell-interactive
   Update-AzVmss -ResourceGroupName <RESOURCE GROUP> -Name <VMSS NAME> -IdentityType UserAssigned -IdentityID "<USER ASSIGNED ID1>","<USER ASSIGNED ID2>"
   ```

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>Удаление управляемого удостоверения, назначаемого пользователем, из масштабируемого набора виртуальных машин Azure

Если у масштабируемого набора виртуальных машин несколько управляемых удостоверений, назначаемых пользователем, с помощью приведенных ниже команд можно удалить все удостоверения, кроме последнего. Не забудьте заменить значения параметров `<RESOURCE GROUP>` и `<VIRTUAL MACHINE SCALE SET NAME>` собственными. `<USER ASSIGNED IDENTITY NAME>` — это имя управляемого удостоверения, назначаемого пользователем, которое следует оставить в масштабируемом наборе виртуальных машин. Эти сведения можно получить в разделе удостоверений масштабируемого набора виртуальных машин с помощью команды `az vmss show`.

```azurepowershell-interactive
Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType UserAssigned -IdentityID "<USER ASSIGNED IDENTITY NAME>"
```
Если в масштабируемом наборе виртуальных машин нет управляемого удостоверения, назначаемого системой, и вы хотите удалить из него все управляемые удостоверения, назначаемые пользователем, используйте следующую команду.

```azurepowershell-interactive
Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType None
```
Если в масштабируемом наборе виртуальных машин есть управляемые удостоверения, как назначаемое системой, так и назначаемые пользователем, вы можете удалить все управляемые удостоверения, назначаемые пользователем, переключившись на использование только управляемого удостоверения, назначаемого системой.

```azurepowershell-interactive
Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType "SystemAssigned"
```

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор управляемых удостоверений для ресурсов Azure](overview.md).
- Ниже приведены комплексные краткие руководства по созданию виртуальных машин Azure:
  
  - [Создание виртуальной машины Windows с помощью PowerShell](../../virtual-machines/windows/quick-create-powershell.md) 
  - [Создание виртуальной машины Linux с помощью PowerShell](../../virtual-machines/linux/quick-create-powershell.md)
