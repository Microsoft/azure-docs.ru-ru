---
title: Настройка управляемых удостоверений на виртуальной машине Azure с помощью шаблона — Azure AD
description: Пошаговые инструкции по настройке управляемых удостоверений на виртуальной машине Azure с помощью шаблона Azure Resource Manager.
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
ms.openlocfilehash: 4948e17d1e0e782a8fa18c3eb5a2185e816a459a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102631410"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-vm-using-templates"></a>Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью шаблонов

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Функция управляемых удостоверений для ресурсов Azure предоставляет службам Azure автоматически управляемые удостоверения в Azure Active Directory. Это удостоверение можно использовать для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, не храня какие-либо учетные данные в коде.

В этой статье вы узнаете, как с помощью шаблона развертывания Azure Resource Manager выполнять приведенные ниже операции с управляемыми удостоверениями для ресурсов Azure на виртуальной машине Azure.

## <a name="prerequisites"></a>Предварительные требования

- Если вы еще не использовали шаблоны развертывания Azure Resource Manager, прочитайте этот [обзор](overview.md). **Обратите внимание на [различие между управляемыми удостоверениями, назначаемыми системой и назначаемыми пользователями](overview.md#managed-identity-types)**.
- Если у вас нет учетной записи Azure, [зарегистрируйтесь для получения бесплатной пробной учетной записи](https://azure.microsoft.com/free/), прежде чем продолжать.

## <a name="azure-resource-manager-templates"></a>Шаблоны Azure Resource Manager

Так же как портал Azure и сценарии, шаблоны [Azure Resource Manager](../../azure-resource-manager/management/overview.md) предоставляют возможность развертывать новые или измененные ресурсы, определенные в группе ресурсов Azure. Доступно несколько способов редактирования и развертывания шаблона, локально и на портале, в том числе:

   - Применение [пользовательского шаблона из Azure Marketplace](../../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template), что позволяет создать шаблон с нуля или взять за основу существующий общий шаблон или [шаблон быстрого запуска](https://azure.microsoft.com/documentation/templates/).
   - Наследование от имеющейся группы ресурсов путем экспорта шаблона из [исходного развертывания](../../azure-resource-manager/templates/export-template-portal.md) или от [текущего состояния развертывания](../../azure-resource-manager/templates/export-template-portal.md).
   - Использование локального [редактора JSON (например, VS Code)](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md), а затем передача и развертывание с помощью PowerShell или интерфейса командной строки.
   - Использование [проекта группы ресурсов Azure](../../azure-resource-manager/templates/create-visual-studio-deployment-project.md) Visual Studio для создания и развертывания шаблона.  

Независимо оттого, какой вариант выбран, во время первоначального развертывания и повторного развертывания в шаблоне используется одинаковый синтаксис. Включение управляемого удостоверения, назначаемого системой или пользователем, для новой или существующей виртуальной машины выполняется таким же образом. Коме того, по умолчанию Azure Resource Manager выполняет [добавочное обновление](../../azure-resource-manager/templates/deployment-modes.md) для развертываний.

## <a name="system-assigned-managed-identity"></a>Управляемое удостоверение, назначаемое системой

В этом разделе вы узнаете, как включить и отключить управляемое удостоверение, назначаемое системой, с помощью шаблона Azure Resource Manager.

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-vm-or-on-an-existing-vm"></a>Включение управляемого удостоверения, назначаемого системой, для создаваемой или существующей виртуальной машины Azure

Чтобы включить назначаемое системой управляемое удостоверение на виртуальной машине, вашей учетной записи должна быть назначена роль [Участника виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Назначать другие роли в каталоге Azure AD не требуется.

1. После входа в Azure локально или через портал Azure используйте учетную запись, связанную с подпиской Azure, содержащей виртуальную машину.

2. Чтобы включить управляемое удостоверение, назначаемое системой, загрузите шаблон в редактор, найдите интересующий ресурс `Microsoft.Compute/virtualMachines` в разделе `resources` и добавьте свойство `"identity"` на том же уровне, что и свойство `"type": "Microsoft.Compute/virtualMachines"`. Используйте следующий синтаксис:

   ```JSON
   "identity": {
       "type": "SystemAssigned"
   },
   ```



3. Когда все будет готово, следующие разделы должны быть добавлены в раздел `resource` шаблона, который в результате должен выглядеть следующим образом.

   ```JSON
    "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2018-06-01",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "[variables('vmName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "SystemAssigned",
                }                        
        }
    ]
   ```

### <a name="assign-a-role-the-vms-system-assigned-managed-identity"></a>Назначение роли управляемого удостоверения, назначаемого системой, для виртуальной машины

После включения управляемого удостоверения, назначаемого системой, на виртуальной машине, вы можете предоставить ему роль, например **Читатель**.

Чтобы назначить роль для присвоенного системой удостоверения виртуальной машины, вашей учетной записи должна быть назначена роль [Администратора доступа пользователей](../../role-based-access-control/built-in-roles.md#user-access-administrator).

1. После входа в Azure локально или через портал Azure используйте учетную запись, связанную с подпиской Azure, содержащей виртуальную машину.

2. Загрузите шаблон в [редактор](#azure-resource-manager-templates) и добавьте приведенную ниже информацию, чтобы предоставить виртуальной машине доступ для **чтения** к группе ресурсов, в которой она создана.  Структура шаблона зависит от редактора и выбранной модели развертывания.

   Добавьте следующий элемент в раздел `parameters`:

    ```JSON
    "builtInRoleType": {
        "type": "string",
        "defaultValue": "Reader"
    },
    "rbacGuid": {
        "type": "string"
    }
    ```

    Добавьте следующий элемент в раздел `variables`:

    ```JSON
    "Reader": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'acdd72a7-3385-48ef-bd42-f606fba81ae7')]"
    ```

    Добавьте следующий элемент в раздел `resources`:

    ```JSON
    {
        "apiVersion": "2017-09-01",
        "type": "Microsoft.Authorization/roleAssignments",
        "name": "[parameters('rbacGuid')]",
        "properties": {
            "roleDefinitionId": "[variables(parameters('builtInRoleType'))]",
            "principalId": "[reference(variables('vmResourceId'), '2017-12-01', 'Full').identity.principalId]",
            "scope": "[resourceGroup().id]"
        },
         "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', parameters('vmName'))]"
        ]
    }
    ```

### <a name="disable-a-system-assigned-managed-identity-from-an-azure-vm"></a>Отключение управляемого удостоверения, назначаемого системой, на виртуальной машине Azure

Чтобы удалить назначаемое системой управляемое удостоверение на виртуальной машине, вашей учетной записи должна быть назначена роль [Участника виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Назначать другие роли в каталоге Azure AD не требуется.

1. После входа в Azure локально или через портал Azure используйте учетную запись, связанную с подпиской Azure, содержащей виртуальную машину.

2. Загрузив шаблон в [редактор](#azure-resource-manager-templates), найдите нужный ресурс `Microsoft.Compute/virtualMachines` в разделе `resources`. Если у вашей виртуальной машины есть только управляемое удостоверение, назначаемое системой, его можно отключить, изменив тип удостоверения на `None`.  

   **Microsoft.Compute/virtualMachines API версии 2018-06-01**

   Если у виртуальной машины есть управляемые удостоверения, назначаемые как системой, так и пользователями, удалите `SystemAssigned` из типа удостоверения и сохраните `UserAssigned` вместе со значениями словаря `userAssignedIdentities`.

   **Microsoft.Compute/virtualMachines API версии 2018-06-01**

   Если значение `apiVersion` соответствует `2017-12-01` и у виртуальной машины есть управляемые удостоверения, назначаемые как системой, так и пользователями, удалите `SystemAssigned` из типа удостоверения и сохраните`UserAssigned` с массивом пользовательских удостоверений `identityIds`.  

В следующем примере показано, как удалить управляемое удостоверение, назначаемое системой, из виртуальной машины без управляемых удостоверений, назначаемых пользователем.

 ```JSON
 {
     "apiVersion": "2018-06-01",
     "type": "Microsoft.Compute/virtualMachines",
     "name": "[parameters('vmName')]",
     "location": "[resourceGroup().location]",
     "identity": {
         "type": "None"
     }
 }
 ```

## <a name="user-assigned-managed-identity"></a>Управляемое удостоверение, назначаемое пользователем

В этом разделе вы назначите управляемое удостоверение, назначаемое пользователем, виртуальной машине Azure с помощью шаблона Azure Resource Manager.

> [!Note]
> Чтобы создать управляемое удостоверение, назначаемое пользователем, с помощью шаблона Azure Resource Manager, обратитесь к разделу [Создание управляемого удостоверения, назначаемого пользователем](how-to-manage-ua-identity-arm.md#create-a-user-assigned-managed-identity).

### <a name="assign-a-user-assigned-managed-identity-to-an-azure-vm"></a>Назначение управляемого удостоверения, назначаемого пользователем, виртуальной машине Linux

Чтобы присвоить назначаемое пользователем удостоверение виртуальной машине, вашей учетной записи должны быть назначены роли [Участник виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) и [Оператор управляемого удостоверения](../../role-based-access-control/built-in-roles.md#managed-identity-operator). Назначать другие роли в каталоге Azure AD не требуется.

1. Чтобы назначить управляемое удостоверение, назначаемое пользователем, виртуальной машине, в элемент `resources` добавьте приведенную ниже запись.  Не забудьте заменить `<USERASSIGNEDIDENTITY>` именем созданного управляемого удостоверения, назначаемого пользователем.

   **Microsoft.Compute/virtualMachines API версии 2018-06-01**

   Если значение `apiVersion` соответствует `2018-06-01`, управляемые удостоверения, назначаемые пользователем, хранятся в формате словаря `userAssignedIdentities`, а значение `<USERASSIGNEDIDENTITYNAME>` должно храниться в переменной, определенной в разделе `variables` шаблона.

   ```JSON
    {
        "apiVersion": "2018-06-01",
        "type": "Microsoft.Compute/virtualMachines",
        "name": "[variables('vmName')]",
        "location": "[resourceGroup().location]",
        "identity": {
            "type": "userAssigned",
            "userAssignedIdentities": {
                "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]": {}
            }
        }
    }
   ```

   **Microsoft.Compute/virtualMachines API версии 2017-12-01**

   Если значение `apiVersion` соответствует `2017-12-01`, управляемые удостоверения, назначаемые пользователем, хранятся в массиве `identityIds`, а значение `<USERASSIGNEDIDENTITYNAME>` должно храниться в переменной, определенной в разделе `variables` шаблона.

   ```JSON
   {
       "apiVersion": "2017-12-01",
       "type": "Microsoft.Compute/virtualMachines",
       "name": "[variables('vmName')]",
       "location": "[resourceGroup().location]",
       "identity": {
           "type": "userAssigned",
           "identityIds": [
               "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]"
           ]
       }
   }
   ```

3. Когда все будет готово, следующие разделы должны быть добавлены в раздел `resource` шаблона, который в результате должен выглядеть следующим образом.

   **Microsoft.Compute/virtualMachines API версии 2018-06-01**    

   ```JSON
     "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2018-06-01",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "[variables('vmName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "userAssigned",
                "userAssignedIdentities": {
                   "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]": {}
                }
            }
        }
    ] 
   ```
   **Microsoft.Compute/virtualMachines API версии 2017-12-01**

   ```JSON
   "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2017-12-01",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "[variables('vmName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "userAssigned",
                "identityIds": [
                   "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]"
                ]
            }
        }
   ]
   ```

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-vm"></a>Удаление управляемого удостоверения, назначаемого пользователем, из виртуальной машины Azure

Чтобы удалить назначаемое пользователем удостоверение на виртуальной машине, вашей учетной записи должна быть назначена роль [Участника виртуальных машин](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor). Назначать другие роли в каталоге Azure AD не требуется.

1. После входа в Azure локально или через портал Azure используйте учетную запись, связанную с подпиской Azure, содержащей виртуальную машину.

2. Загрузив шаблон в [редактор](#azure-resource-manager-templates), найдите нужный ресурс `Microsoft.Compute/virtualMachines` в разделе `resources`. Если у вашей виртуальной машины есть только управляемое удостоверение, назначаемое пользователем, его можно отключить, изменив тип удостоверения на `None`.

   В следующем примере показано, как удалить все управляемые удостоверения, назначаемые пользователем, из виртуальной машины без управляемых удостоверений, назначаемых системой.

   ```json
    {
      "apiVersion": "2018-06-01",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "identity": {
          "type": "None"
          },
    }
   ```

   **Microsoft.Compute/virtualMachines API версии 2018-06-01**

   Чтобы удалить отдельное управляемое удостоверение, назначаемое пользователем, из виртуальной машины, удалите его из словаря `useraAssignedIdentities`.

   Если у вас есть управляемое удостоверение, назначаемое системой, сохраните его в значении `type` в рамках значения `identity`.

   **Microsoft.Compute/virtualMachines API версии 2017-12-01**

   Чтобы удалить отдельное управляемое удостоверение, назначаемое пользователем, из виртуальной машины, удалите его из массива `identityIds`.

   Если у вас есть управляемое удостоверение, назначаемое системой, сохраните его в значении `type` в рамках значения `identity`.

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор управляемых удостоверений для ресурсов Azure](overview.md).
