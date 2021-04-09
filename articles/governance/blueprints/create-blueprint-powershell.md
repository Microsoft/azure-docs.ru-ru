---
title: Краткое руководство. Создание схемы с помощью PowerShell
description: В этом кратком руководстве вы создадите, определите и развернете артефакты с помощью PowerShell и Azure Blueprints.
ms.date: 01/27/2021
ms.topic: quickstart
ms.openlocfilehash: 002ba6fcf20b10bcc44fb486a85bc24bfdd12ae3
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105559049"
---
# <a name="quickstart-define-and-assign-an-azure-blueprint-with-powershell"></a>Краткое руководство. Определение и назначение схемы Azure с помощью PowerShell

Зная, как создавать и назначать схемы, вы сможете определять типичные шаблоны для разработки многоразовых и быстро развертываемых конфигураций на основе шаблонов Azure Resource Manager (шаблонов ARM), политик, правил безопасности и т. п. В этом руководстве вы научитесь использовать службу Azure Blueprint для выполнения некоторых общих задач, связанных с созданием, публикацией и назначением схемы в вашей организации, таких как:

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free), прежде чем начинать работу.
- Выполните инструкции из [этой статьи](./how-to/manage-assignments-ps.md#add-the-azblueprint-module), чтобы установить модуль **Az.Blueprint** из коллекции PowerShell и проверить его работу.
- Если вы ранее не использовали Azure Blueprints, зарегистрируйте поставщик ресурсов, выполнив командлет `Register-AzResourceProvider -ProviderNamespace Microsoft.Blueprint` в Azure PowerShell.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-blueprint"></a>Создание схемы

Первым этапом при определении стандартной модели соответствия требованиям является формирование схемы на основе доступных ресурсов. Мы создадим схему с именем MyBlueprint для настройки роли и назначения политик для подписки. Затем мы добавим группу ресурсов, шаблон ARM и назначение ролей группе ресурсов.

> [!NOTE]
> При использовании PowerShell сначала создается объект _схемы_. Для каждого _артефакта_ с параметрами, который необходимо добавить, необходимо заранее определить параметры в исходной _схеме_.

1. Создайте исходный объект _схемы_. Параметр **BlueprintFile** принимает JSON-файл со свойствами схемы, групп ресурсов, которые необходимо создать, и всеми параметрами уровня схемы. Эти параметры задаются при назначении и используются артефактами, которые будут добавлены в дальнейшем.

   - JSON-файл blueprint.json

     ```json
     {
         "properties": {
             "description": "This blueprint sets tag policy and role assignment on the subscription, creates a ResourceGroup, and deploys a resource template and role assignment to that ResourceGroup.",
             "targetScope": "subscription",
             "parameters": {
                 "storageAccountType": {
                     "type": "string",
                     "defaultValue": "Standard_LRS",
                     "allowedValues": [
                         "Standard_LRS",
                         "Standard_GRS",
                         "Standard_ZRS",
                         "Premium_LRS"
                     ],
                     "metadata": {
                         "displayName": "storage account type.",
                         "description": null
                     }
                 },
                 "tagName": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The name of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "tagValue": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The value of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "contributors": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Contributor role at the subscription",
                         "strongType": "PrincipalId"
                     }
                 },
                 "owners": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Owner role at the resource group",
                         "strongType": "PrincipalId"
                     }
                 }
             },
             "resourceGroups": {
                 "storageRG": {
                     "description": "Contains the resource template deployment and a role assignment."
                 }
             }
         }
     }
     ```

   - Команда PowerShell

     ```azurepowershell-interactive
     # Login first with Connect-AzAccount if not using Cloud Shell

     # Get a reference to the new blueprint object, we'll use it in subsequent steps
     $blueprint = New-AzBlueprint -Name 'MyBlueprint' -BlueprintFile .\blueprint.json
     ```

     > [!NOTE]
     > При создании определений схем с помощью программных средств используйте имя файла _blueprint.json_.
     > Это имя файла используется при вызове [Import-AzBlueprintWithArtifact](/powershell/module/az.blueprint/import-azblueprintwithartifact).

     По умолчанию объект схемы создается в подписке, заданной по умолчанию. Чтобы указать группу управления, используйте параметр **ManagementGroupId**. Чтобы указать подписку, используйте параметр **SubscriptionId**.

1. Добавьте назначение ролей в подписку. В **ArtifactFile** определяется _тип_ артефакта и свойства, соответствующие идентификатору определения роли, а удостоверения субъектов передаются в виде массива значений. В приведенном ниже примере удостоверения субъектов, которым предоставляется указанная роль, передаются в параметре, задаваемом во время назначения схемы. В этом примере используется встроенная роль _Участник_ с глобальным уникальным идентификатором `b24988ac-6180-42a0-ab88-20f7382dd24c`.

   - JSON-файл \artifacts\roleContributor.json

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
             "principalIds": "[parameters('contributors')]"
         }
     }
     ```

   - Команда PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Name 'roleContributor' -ArtifactFile .\artifacts\roleContributor.json
     ```

1. Добавьте назначение политики в подписку. В **ArtifactFile** определяется _тип_ артефакта и свойства, соответствующие определению политики или инициативы, и настраивается назначение политики для использования указанных параметров схемы, настраиваемых во время назначения схемы. В этом примере используется встроенная политика "_Задать тег и его значение по умолчанию для групп ресурсов_" с глобальным уникальным идентификатором `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - JSON-файл \artifacts\policyTags.json

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "displayName": "Apply tag and its default value to resource groups",
             "description": "Apply tag and its default value to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "[parameters('tagName')]"
                 },
                 "tagValue": {
                     "value": "[parameters('tagValue')]"
                 }
             }
         }
     }
     ```

   - Команда PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Name 'policyTags' -ArtifactFile .\artifacts\policyTags.json
     ```

1. Добавьте в подписку еще одно назначение политики для тега хранилища (повторно используя параметр _storageAccountType_). Этот дополнительный артефакт назначения политики демонстрирует, что определенный в схеме параметр может использоваться несколькими артефактами. В этом примере **storageAccountType** используется для настройки тега группы ресурсов. Это значение предоставляет сведения об учетной записи хранения, которая будет создана на следующем шаге. В этом примере используется встроенная политика "_Задать тег и его значение по умолчанию для групп ресурсов_" с глобальным уникальным идентификатором `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - JSON-файл \artifacts\policyStorageTags.json

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "displayName": "Apply storage tag to resource group",
             "description": "Apply storage tag and the parameter also used by the template to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "StorageType"
                 },
                 "tagValue": {
                     "value": "[parameters('storageAccountType')]"
                 }
             }
         }
     }
     ```

   - Команда PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Name 'policyStorageTags' -ArtifactFile .\artifacts\policyStorageTags.json
     ```

1. Добавьте шаблон в группу ресурсов. **TemplateFile** для шаблона ARM содержит стандартный JSON-компонент шаблона. Этот шаблон также повторно использует параметры схемы **storageAccountType**, **tagName** и **tagValue**, передавая каждый из них в шаблон. Параметры схемы доступны шаблону за счет использования параметра **TemplateParameterFile**. В коде JSON-шаблона эта пара "ключ — значение" используется для внедрения значения. Имена параметров схемы и шаблона могут быть одинаковыми.

   - Файл JSON шаблона ARM — \artifacts\templateStorage.json

     ```json
     {
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
             "storageAccountTypeFromBP": {
                 "type": "string",
                 "metadata": {
                     "description": "Storage Account type"
                 }
             },
             "tagNameFromBP": {
                 "type": "string",
                 "defaultValue": "NotSet",
                 "metadata": {
                     "description": "Tag name from blueprint"
                 }
             },
             "tagValueFromBP": {
                 "type": "string",
                 "defaultValue": "NotSet",
                 "metadata": {
                     "description": "Tag value from blueprint"
                 }
             }
         },
         "variables": {
             "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
         },
         "resources": [{
             "type": "Microsoft.Storage/storageAccounts",
             "name": "[variables('storageAccountName')]",
             "apiVersion": "2016-01-01",
             "tags": {
                 "[parameters('tagNameFromBP')]": "[parameters('tagValueFromBP')]"
             },
             "location": "[resourceGroup().location]",
             "sku": {
                 "name": "[parameters('storageAccountTypeFromBP')]"
             },
             "kind": "Storage",
             "properties": {}
         }],
         "outputs": {
             "storageAccountSku": {
                 "type": "string",
                 "value": "[variables('storageAccountName')]"
             }
         }
     }
     ```

   - Файл JSON параметров шаблона ARM — \artifacts\templateStorageParams.json

     ```json
     {
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
             "storageAccountTypeFromBP": {
                 "value": "[parameters('storageAccountType')]"
             },
             "tagNameFromBP": {
                 "value": "[parameters('tagName')]"
             },
             "tagValueFromBP": {
                 "value": "[parameters('tagValue')]"
             }
         }
     }
     ```

   - Команда PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Type TemplateArtifact -Name 'templateStorage' -TemplateFile .\artifacts\templateStorage.json -TemplateParameterFile .\artifacts\templateStorageParams.json -ResourceGroupName storageRG
     ```

1. Добавьте назначение ролей в группу ресурсов. Так же как и в предыдущем фрагменте назначения ролей, в приведенном ниже примере используется идентификатор для роли **Владелец**, но из схемы для него передается другой параметр. В этом примере используется встроенная роль _Владелец_ с глобальным уникальным идентификатором `8e3af657-a8ff-443c-a75c-2fe8c4bcb635`.

   - JSON-файл \artifacts\roleOwner.json

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "resourceGroup": "storageRG",
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
             "principalIds": "[parameters('owners')]"
         }
     }
     ```

   - Команда PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Name 'roleOwner' -ArtifactFile .\artifacts\roleOwner.json
     ```

## <a name="publish-a-blueprint"></a>Публикация схемы

Теперь, когда артефакты добавлены в схему, пора ее опубликовать. Публикация схемы позволяет назначать ее подписке.

```azurepowershell-interactive
# Use the reference to the new blueprint object from the previous steps
Publish-AzBlueprint -Blueprint $blueprint -Version '{BlueprintVersion}'
```

Значение `{BlueprintVersion}` представляет собой строку, состоящую из букв, цифр и дефисов (без пробелов или других специальных символов), содержащую не более 20 символов. Оно должно быть уникальным и информативным, например **v20180622-135541**.

## <a name="assign-a-blueprint"></a>Назначение схемы

После публикации схемы с помощью PowerShell ее можно назначить подписке. Назначьте созданную схему одной из подписок в иерархии группы управления. Если схема сохранена в подписке, схему можно будет назначить только этой подписке. Параметр **Blueprint** определяет схему для назначения. Чтобы указать параметры имени, расположения, идентификатора, блокировки и схемы, используйте соответствующие параметры PowerShell для командлета `New-AzBlueprintAssignment` или укажите их в параметре JSON-файла **AssignmentFile**.

1. Запустите развертывание схемы, назначив ее подписке. Так как в параметрах **contributors** и **owners** должны быть переданы массивы идентификаторов objectId субъектов, которым следует предоставить назначение ролей, используйте [API Graph Azure Active Directory](/graph/migrate-azure-ad-graph-planning-checklist) для сбора идентификаторов objectId, которые вы примените в **AssignmentFile** для собственных пользователей, групп или субъект-служб.

   - JSON-файл blueprintAssignment.json

     ```json
     {
         "properties": {
             "blueprintId": "/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint",
             "resourceGroups": {
                 "storageRG": {
                     "name": "StorageAccount",
                     "location": "eastus2"
                 }
             },
             "parameters": {
                 "storageAccountType": {
                     "value": "Standard_GRS"
                 },
                 "tagName": {
                     "value": "CostCenter"
                 },
                 "tagValue": {
                     "value": "ContosoIT"
                 },
                 "contributors": {
                     "value": [
                         "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
                         "38833b56-194d-420b-90ce-cff578296714"
                     ]
                 },
                 "owners": {
                     "value": [
                         "44254d2b-a0c7-405f-959c-f829ee31c2e7",
                         "316deb5f-7187-4512-9dd4-21e7798b0ef9"
                     ]
                 }
             }
         },
         "identity": {
             "type": "systemAssigned"
         },
         "location": "westus"
     }
     ```

   - Команда PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintAssignment -Blueprint $blueprint -Name 'assignMyBlueprint' -AssignmentFile .\blueprintAssignment.json
     ```

   - Управляемое удостоверение, назначаемое пользователем

     Для назначения схемы можно также использовать [управляемое удостоверение, назначаемое пользователем](../../active-directory/managed-identities-azure-resources/overview.md).
     В этом случае раздел **identity** JSON-файла назначения нужно изменить следующим образом. Замените `{tenantId}`, `{subscriptionId}`, `{yourRG}` и `{userIdentity}` идентификатором клиента, идентификатором подписки, именем группы ресурсов и именем управляемого удостоверения, назначаемого пользователем, соответственно.

     ```json
     "identity": {
         "type": "userAssigned",
         "tenantId": "{tenantId}",
         "userAssignedIdentities": {
             "/subscriptions/{subscriptionId}/resourceGroups/{yourRG}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{userIdentity}": {}
         }
     },
     ```

     **Управляемое удостоверение, назначаемое пользователем**, может быть в любой подписке и группе ресурсов, к которым у пользователя, назначающего схему, есть разрешения на доступ.

     > [!IMPORTANT]
     > Azure Blueprints не администрирует управляемое удостоверение, назначаемое пользователем. Назначение соответствующих ролей и разрешений выполняют пользователи. В противном случае назначение схемы завершится ошибкой.

## <a name="clean-up-resources"></a>Очистка ресурсов

### <a name="unassign-a-blueprint"></a>Отмена назначения схемы

Вы можете удалить схему из подписки. Удаление часто выполняется, когда ресурсы артефакта больше не требуются. При удалении схемы назначенные артефакты оставляются. Чтобы удалить назначение схемы, используйте командлет `Remove-AzBlueprintAssignment`:

assignMyBlueprint

```azurepowershell-interactive
Remove-AzBlueprintAssignment -Name 'assignMyBlueprint'
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого краткого руководства вы создали, назначили и удалили схему с помощью PowerShell. Чтобы узнать больше о схемах Azure, перейдите к статье о жизненном цикле схемы.

> [!div class="nextstepaction"]
> [Ознакомьтесь со сведениями о жизненном цикле схем](./concepts/lifecycle.md)