---
title: Назначение ролей Azure с помощью Azure CLI — Azure RBAC
description: Узнайте, как предоставить доступ к ресурсам Azure для пользователей, групп, субъектов-служб или управляемых удостоверений с помощью Azure CLI и управления доступом на основе ролей Azure (Azure RBAC).
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 09/28/2020
ms.author: rolyon
ms.custom: contperf-fy21q1, devx-track-azurecli
ms.openlocfilehash: ee356f32b6799c6182ec1c9e061a35271a4bbc23
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100556979"
---
# <a name="assign-azure-roles-using-azure-cli"></a>Назначение ролей Azure с помощью Azure CLI

[!INCLUDE [Azure RBAC definition grant access](../../includes/role-based-access-control/definition-grant.md)] В этой статье описывается назначение ролей с помощью Azure CLI.

## <a name="prerequisites"></a>Предварительные условия

Для назначения ролей необходимо иметь следующее:

- `Microsoft.Authorization/roleAssignments/write` разрешения, такие как [администратор доступа пользователей](built-in-roles.md#user-access-administrator) или [владелец](built-in-roles.md#owner)
- [Bash в Azure Cloud Shell](../cloud-shell/overview.md) или [Azure CLI](/cli/azure)

## <a name="steps-to-assign-an-azure-role"></a>Действия по назначению роли Azure

Назначение роли состоит из трех элементов: субъекта безопасности, определения роли и области.

### <a name="step-1-determine-who-needs-access"></a>Шаг 1. Определение пользователей, которым требуется доступ

Роль можно назначить пользователю, группе, субъекту-службе или управляемому удостоверению. Чтобы назначить роль, может потребоваться указать уникальный идентификатор объекта. Идентификатор имеет следующий формат: `11111111-1111-1111-1111-111111111111`. Идентификатор можно получить с помощью портал Azure или Azure CLI.

**Пользователь**

Для пользователя Azure AD получите имя участника-пользователя, например *патлонг \@ contoso.com* или идентификатор объекта пользователя. Чтобы получить идентификатор объекта, можно использовать команду [AZ AD user демонстрация](/cli/azure/ad/user#az_ad_user_show).

```azurecli
az ad user show --id "{principalName}" --query "objectId" --output tsv
```

**Группа**

Для группы Azure AD требуется идентификатор объекта группы. Чтобы получить идентификатор объекта, можно использовать список групп [AZ AD Group](/cli/azure/ad/group#az_ad_group_show) или [AZ AD](/cli/azure/ad/group#az_ad_group_list).

```azurecli
az ad group show --group "{groupName}" --query "objectId" --output tsv
```

**Субъект-служба**

Для субъекта-службы Azure AD (удостоверения, используемого приложением) необходим идентификатор объекта субъекта-службы. Чтобы получить идентификатор объекта, можно использовать команду [AZ AD SP List](/cli/azure/ad/sp#az_ad_sp_list). Для субъекта-службы используйте идентификатор объекта, а **не** идентификатор приложения.

```azurecli
az ad sp list --all --query "[].{displayName:displayName, objectId:objectId}" --output tsv
az ad sp list --display-name "{displayName}"
```

**Управляемое удостоверение**

Для назначенного системой или управляемого пользователем удостоверения требуется идентификатор объекта. Чтобы получить идентификатор объекта, можно использовать команду [AZ AD SP List](/cli/azure/ad/sp#az_ad_sp_list).

```azurecli
az ad sp list --all --filter "servicePrincipalType eq 'ManagedIdentity'"
```

Чтобы просто перечислить назначенные пользователю управляемые удостоверения, можно использовать команду [AZ Identity List](/cli/azure/identity#az_identity_list).

```azurecli
az identity list
```
    
### <a name="step-2-select-the-appropriate-role"></a>Шаг 2. Выбор соответствующей роли

Разрешения объединяются в роли. Вы можете выбрать из списка несколько [встроенных ролей Azure](built-in-roles.md) или использовать собственные пользовательские роли. Рекомендуется предоставлять доступ с минимальными необходимыми привилегиями, поэтому не следует назначать более широкие роли.

Чтобы вывести список ролей и получить уникальный идентификатор роли, можно использовать команду [AZ Role Definition List](/cli/azure/role/definition#az_role_definition_list).

```azurecli
az role definition list --query "[].{name:name, roleType:roleType, roleName:roleName}" --output tsv
```

Вот как можно вывести сведения о конкретной роли.

```azurecli
az role definition list --name "{roleName}"
```

Дополнительные сведения см. в разделе [список определений ролей Azure](role-definitions-list.md#azure-cli).
 
### <a name="step-3-identify-the-needed-scope"></a>Шаг 3. Определение требуемой области

Azure предоставляет четыре уровня области: ресурс, [Группа ресурсов](../azure-resource-manager/management/overview.md#resource-groups), подписка и [Группа управления](../governance/management-groups/overview.md). Рекомендуется предоставлять доступ с минимальными необходимыми привилегиями, поэтому не следует назначать роль в более широкой области. Дополнительные сведения об областях см. в статье [Общие сведения об областях для Azure RBAC](scope-overview.md).

**Область ресурса**

Для области ресурсов требуется идентификатор ресурса. Идентификатор ресурса можно найти, просмотрев свойства ресурса в портал Azure. Идентификатор ресурса имеет следующий формат.

```
/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/{providerName}/{resourceType}/{resourceSubType}/{resourceName}
```

**Область группы ресурсов**

Для области группы ресурсов требуется имя группы ресурсов. Имя можно найти на странице " **группы ресурсов** " в портал Azure или можно использовать команду [AZ Group List](/cli/azure/group#az_group_list).

```azurecli
az group list --query "[].{name:name}" --output tsv
```

**Область действия подписки** 

Для области подписки необходим идентификатор подписки. Идентификатор можно найти на странице " **подписки** " в портал Azure или можно использовать команду [AZ Account List](/cli/azure/account#az_account_list).

```azurecli
az account list --query "[].{name:name, id:id}" --output tsv
```

**Область группы управления** 

Для области группы управления требуется имя группы управления. Имя можно найти на странице " **группы управления** " в портал Azure или можно использовать команду [AZ Account Management-Group List](/cli/azure/account/management-group#az_account_management_group_list).

```azurecli
az account management-group list --query "[].{name:name, id:id}" --output tsv
```
    
### <a name="step-4-assign-role"></a>Шаг 4. Назначение роли

Чтобы назначить роль, используйте команду [AZ Role назначение Create](/cli/azure/role/assignment#az_role_assignment_create) . В зависимости от области команда обычно имеет один из следующих форматов.

**Область ресурса**

```azurecli
az role assignment create --assignee "{assignee}" \
--role "{roleNameOrId}" \
--scope "/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/{providerName}/{resourceType}/{resourceSubType}/{resourceName}"
```

**Область группы ресурсов**

```azurecli
az role assignment create --assignee "{assignee}" \
--role "{roleNameOrId}" \
--resource-group "{resourceGroupName}"
```

**Область действия подписки** 

```azurecli
az role assignment create --assignee "{assignee}" \
--role "{roleNameOrId}" \
--subscription "{subscriptionNameOrId}"
```

**Область группы управления** 

```azurecli
az role assignment create --assignee "{assignee}" \
--role "{roleNameOrId}" \
--scope "/providers/Microsoft.Management/managementGroups/{managementGroupName}"
``` 

Ниже приведен пример выходных данных при назначении роли [участника виртуальной машины](built-in-roles.md#virtual-machine-contributor) пользователю в области группы ресурсов.

```azurecli
{
  "canDelegate": null,
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
  "name": "{roleAssignmentId}",
  "principalId": "{principalId}",
  "principalType": "User",
  "resourceGroup": "{resourceGroupName}",
  "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/9980e02c-c2be-4d73-94e8-173b1dc7cf3c",
  "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}",
  "type": "Microsoft.Authorization/roleAssignments"
}
```
    
## <a name="assign-role-examples"></a>Примеры назначения ролей

#### <a name="assign-a-role-for-all-blob-containers-in-a-storage-account-resource-scope"></a>Назначение роли для всех контейнеров больших двоичных объектов в области ресурсов учетной записи хранения

Назначает роль [участника данных большого двоичного объекта хранилища](built-in-roles.md#storage-blob-data-contributor) субъекту-службе с идентификатором объекта *55555555-5555-5555-5555-555555555555* в области действия ресурса для учетной записи хранения с именем *storage12345*.

```azurecli
az role assignment create --assignee "55555555-5555-5555-5555-555555555555" \
--role "Storage Blob Data Contributor" \
--scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345"
```

#### <a name="assign-a-role-for-a-specific-blob-container-resource-scope"></a>Назначение роли для определенной области ресурсов контейнера больших двоичных объектов

Назначает роль [участника данных большого двоичного объекта хранилища](built-in-roles.md#storage-blob-data-contributor) субъекту-службе с идентификатором объекта *55555555-5555-5555-5555-555555555555* в области действия ресурса для контейнера больших двоичных объектов с именем *BLOB-Container-01*.

```azurecli
az role assignment create --assignee "55555555-5555-5555-5555-555555555555" \
--role "Storage Blob Data Contributor" \
--scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345/blobServices/default/containers/blob-container-01"
```

#### <a name="assign-a-role-for-a-group-in-a-specific-virtual-network-resource-scope"></a>Назначение роли для группы в определенной области ресурсов виртуальной сети

Назначает роль [участника виртуальной машины](built-in-roles.md#virtual-machine-contributor) группе группы *Анна Mack* с идентификатором 22222222-2222-2222-2222-222222222222 в области действия ресурса для виртуальной сети с именем *Pharma-Sales-Project-Network*.

```azurecli
az role assignment create --assignee "22222222-2222-2222-2222-222222222222" \
--role "Virtual Machine Contributor" \
--scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/pharma-sales/providers/Microsoft.Network/virtualNetworks/pharma-sales-project-network"
```

#### <a name="assign-a-role-for-a-user-at-a-resource-group-scope"></a>Назначение роли для пользователя в области группы ресурсов

Назначает роль [участника виртуальной машины](built-in-roles.md#virtual-machine-contributor) пользователю *патлонг \@ contoso.com* в области группы ресурсов *Pharma-Sales* .

```azurecli
az role assignment create --assignee "patlong@contoso.com" \
--role "Virtual Machine Contributor" \
--resource-group "pharma-sales"
```

#### <a name="assign-a-role-for-a-user-using-the-unique-role-id-at-a-resource-group-scope"></a>Назначение роли для пользователя с помощью уникального идентификатора роли в области группы ресурсов

Существует несколько случаев, когда имя роли может измениться, например:

- Вы используете собственную пользовательскую роль и решили изменить ее имя.
- Вы используете роль предварительного просмотра с именем **(Предварительная версия)** . При освобождении роли роль переименовывается.

Даже если роль переименована, идентификатор роли не изменяется. Если вы используете скрипты или автоматизацию для создания назначений ролей, рекомендуется использовать уникальный идентификатор роли вместо имени роли. Таким образом, если роль переименована, скорее всего, сценарии будут работать.

В следующем примере роль [участника виртуальной машины](built-in-roles.md#virtual-machine-contributor) назначается пользователю *патлонг \@ contoso.com* в области группы ресурсов *Pharma-Sales* .

```azurecli
az role assignment create --assignee "patlong@contoso.com" \
--role "9980e02c-c2be-4d73-94e8-173b1dc7cf3c" \
--resource-group "pharma-sales"
```

#### <a name="assign-a-role-for-all-blob-containers-at-a-resource-group-scope"></a>Назначение роли для всех контейнеров больших двоичных объектов в области группы ресурсов

Назначает роль [участника данных BLOB-объекта хранилища](built-in-roles.md#storage-blob-data-contributor) субъекту-службе с идентификатором объекта *55555555-5555-5555-5555-555555555555* в области группы ресурсов *example. RG* .

```azurecli
az role assignment create --assignee "55555555-5555-5555-5555-555555555555" \
--role "Storage Blob Data Contributor" \
--resource-group "Example-Storage-rg"
```

Кроме того, можно указать полное имя группы ресурсов с помощью `--scope` параметра:

```azurecli
az role assignment create --assignee "55555555-5555-5555-5555-555555555555" \
--role "Storage Blob Data Contributor" \
--scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg"
```

#### <a name="assign-a-role-for-an-application-at-a-resource-group-scope"></a>Назначение роли для приложения в области группы ресурсов

Назначает роль [участника виртуальной машины](built-in-roles.md#virtual-machine-contributor) приложению с идентификатором объекта субъекта-службы 44444444-4444-4444-4444-444444444444 в области группы ресурсов *Pharma — Sales* .

```azurecli
az role assignment create --assignee "44444444-4444-4444-4444-444444444444" \
--role "Virtual Machine Contributor" \
--resource-group "pharma-sales"
```

#### <a name="assign-a-role-for-a-new-service-principal-at-a-resource-group-scope"></a>Назначение роли для нового субъекта-службы в области группы ресурсов

Если вы создадите новый субъект-службу и сразу же попытаетесь назначить роль этому субъекту-службе, в некоторых случаях такое назначение роли может завершиться ошибкой. Например, при использовании скрипта для создания нового управляемого удостоверения и последующей попытке назначить роль этому субъекту-службе может произойти сбой назначения роли. Наиболее вероятной причиной этой ошибки является задержка репликации. Субъект-служба создается в одном регионе; однако назначение ролей может происходить в другом регионе, который еще не реплицировал субъект-службу. Для решения этой ситуации необходимо указать тип участника при создании назначения роли.

Чтобы назначить роль, используйте команду [AZ Role назначение Create](/cli/azure/role/assignment#az_role_assignment_create), укажите значение для параметра `--assignee-object-id` , а затем установите `--assignee-principal-type` для значения `ServicePrincipal` .

```azurecli
az role assignment create --assignee-object-id "{assigneeObjectId}" \
--assignee-principal-type "{assigneePrincipalType}" \
--role "{roleNameOrId}" \
--resource-group "{resourceGroupName}" \
--scope "/subscriptions/{subscriptionId}"
```

В следующем примере роль [участника виртуальной машины](built-in-roles.md#virtual-machine-contributor) назначается управляемому удостоверению *MSI-Test* в области группы ресурсов *Pharma-Sales* :

```azurecli
az role assignment create --assignee-object-id "33333333-3333-3333-3333-333333333333" \
--assignee-principal-type "ServicePrincipal" \
--role "Virtual Machine Contributor" \
--resource-group "pharma-sales"
```

#### <a name="assign-a-role-for-a-user-at-a-subscription-scope"></a>Назначение роли для пользователя в области действия подписки

Назначает роль [читателя](built-in-roles.md#reader) пользователю *аннм \@ example.com* в области действия подписки.

```azurecli
az role assignment create --assignee "annm@example.com" \
--role "Reader" \
--subscription "00000000-0000-0000-0000-000000000000"
```

#### <a name="assign-a-role-for-a-group-at-a-subscription-scope"></a>Назначение роли для группы в области действия подписки

Назначает роль [читателя](built-in-roles.md#reader) группе группы *Анна Mack* с идентификатором 22222222-2222-2222-2222-222222222222 в области действия подписки.

```azurecli
az role assignment create --assignee "22222222-2222-2222-2222-222222222222" \
--role "Reader" \
--subscription "00000000-0000-0000-0000-000000000000"
```

#### <a name="assign-a-role-for-all-blob-containers-at-a-subscription-scope"></a>Назначение роли для всех контейнеров больших двоичных объектов в области действия подписки

Назначает роль [читателя данных BLOB-объекта хранилища](built-in-roles.md#storage-blob-data-reader) пользователю *(Alain \@ example.com* в области действия подписки.

```azurecli
az role assignment create --assignee "alain@example.com" \
--role "Storage Blob Data Reader" \
--scope "/subscriptions/00000000-0000-0000-0000-000000000000"
```

#### <a name="assign-a-role-for-a-user-at-a-management-group-scope"></a>Назначение роли для пользователя в области группы управления

Назначает роль [читателя выставления счетов](built-in-roles.md#billing-reader) пользователю *(Alain \@ example.com* в области группы управления.

```azurecli
az role assignment create --assignee "alain@example.com" \
--role "Billing Reader" \
--scope "/providers/Microsoft.Management/managementGroups/marketing-group"
```

## <a name="next-steps"></a>Дальнейшие действия

- [Вывод списка назначений ролей Azure с помощью Azure CLI](role-assignments-list-cli.md)
- [Управление ресурсами и группами ресурсов Azure с помощью интерфейса командной строки Azure](../azure-resource-manager/management/manage-resources-cli.md)
