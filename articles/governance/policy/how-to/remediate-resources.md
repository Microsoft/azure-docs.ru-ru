---
title: Исправление несоответствующих ресурсов
description: В этом руководстве описывается исправление ресурсов, которые не соответствуют политикам в службе "Политика Azure".
ms.date: 02/17/2021
ms.topic: how-to
ms.openlocfilehash: e567bedf48393a36215c1ac3f3d11f467ae7badd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101742234"
---
# <a name="remediate-non-compliant-resources-with-azure-policy"></a>Исправление несоответствующих ресурсов с помощью службы "Политика Azure"

Ресурсы, которые не соответствуют политике **deployIfNotExists** или **modify**, могут быть переведены в состояние соответствия с помощью **исправления**. Исправление достигается путем указания политики Azure на выполнение **deployIfNotExists** эффектов или **операций изменения** назначенной политики для существующих ресурсов и подписок, будь то назначение к группе управления, подписке, группе ресурсов или отдельному ресурсу. В этой статье приведены шаги, необходимые для распознавания не соответствующих политике ресурсов и выполнения исправлений с помощью службы "Политика Azure".

## <a name="how-remediation-security-works"></a>Как работает безопасность исправлений

Если служба "Политика Azure" применяет шаблон в определении политики **deployIfNotExists**, она использует [управляемое удостоверение](../../../active-directory/managed-identities-azure-resources/overview.md).
Служба "Политика Azure" создает управляемое удостоверение для каждого назначения, но вам нужно иметь сведения о том, каким ролям нужно предоставить управляемое удостоверение. Если в управляемом удостоверении отсутствуют роли, во время назначения политики или инициативы выводится сообщение об ошибке. При использовании портала служба "Политика Azure" автоматически предоставляет управляемое удостоверение всем перечисленным ролям после начала процесса назначения. При использовании пакета SDK роли должны быть вручную предоставлены управляемому удостоверению. _Расположение_ управляемого удостоверения не влияет на его работу с политикой Azure.

:::image type="content" source="../media/remediate-resources/missing-role.png" alt-text="Снимок экрана политики deployIfNotExists, в которой отсутствует определенное разрешение на управляемое удостоверение." border="false":::

> [!IMPORTANT]
> В следующих сценариях управляемому удостоверению назначения должен быть [предоставлен доступ вручную](#manually-configure-the-managed-identity) , или развертывание исправления завершится ошибкой:
>
> - Если назначение создается с помощью пакета SDK
> - Если ресурс, измененный с помощью **deployIfNotExists** или **Modify** , находится за пределами области назначения политики
> - Если шаблон обращается к свойствам ресурсов за пределами области назначения политики,

## <a name="configure-policy-definition"></a>Настройка определения политики

Первый шаг — определение ролей, которые требуются **deployIfNotExists** или **modify** в определении политики для успешного развертывания содержимого включенного шаблона. В свойстве **details** добавьте свойство **roleDefinitionIds**. Это свойство является массивом строк, соответствующих ролям в среде. См. полные примеры для [deployIfNotExists](../concepts/effects.md#deployifnotexists-example) и [modify](../concepts/effects.md#modify-examples).

```json
"details": {
    ...
    "roleDefinitionIds": [
        "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/{roleGUID}",
        "/providers/Microsoft.Authorization/roleDefinitions/{builtinroleGUID}"
    ]
}
```

Свойство **roleDefinitionIds** использует полный идентификатор ресурса и не принимает короткое имя роли **roleName**. Чтобы получить идентификатор для роли "Участник" в среде, используйте следующий код:

```azurecli-interactive
az role definition list --name 'Contributor'
```

## <a name="manually-configure-the-managed-identity"></a>Настройка управляемого удостоверения вручную

Когда вы создаете назначение с помощью портала, служба "Политика Azure" одновременно создает управляемое удостоверение и предоставляет ему роли, определенные в **roleDefinitionIds**. В следующих ситуациях шаги по созданию управляемого удостоверения и присвоению ему разрешений должны выполняться вручную:

- при использовании пакета SDK (например, Azure PowerShell);
- при изменении ресурсов, находящихся за пределами области назначения, с помощью шаблона;
- при чтении ресурсов, находящихся за пределами области назначения, с помощью шаблона.

### <a name="create-managed-identity-with-powershell"></a>Создание управляемого удостоверения с помощью PowerShell

Для создания управляемого удостоверения во время назначения политики необходимо определить свойство **Location** и использовать свойство **AssignIdentity**. Следующий пример получает определение встроенной политики **Развернуть прозрачное шифрование базы данных SQL**, задает целевую группу ресурсов, а затем создает назначение.

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get the built-in "Deploy SQL DB transparent data encryption" policy definition
$policyDef = Get-AzPolicyDefinition -Id '/providers/Microsoft.Authorization/policyDefinitions/86a912f6-9a06-4e26-b447-11b16ba8659f'

# Get the reference to the resource group
$resourceGroup = Get-AzResourceGroup -Name 'MyResourceGroup'

# Create the assignment using the -Location and -AssignIdentity properties
$assignment = New-AzPolicyAssignment -Name 'sqlDbTDE' -DisplayName 'Deploy SQL DB transparent data encryption' -Scope $resourceGroup.ResourceId -PolicyDefinition $policyDef -Location 'westus' -AssignIdentity
```

Переменная `$assignment` теперь содержит идентификатор субъекта для управляемого удостоверения, а также стандартные значения, возвращаемые при создании назначения политики. Доступ может осуществляться через `$assignment.Identity.PrincipalId`.

### <a name="grant-defined-roles-with-powershell"></a>Предоставление определенных ролей с помощью PowerShell

Новое управляемое удостоверение должно выполнить репликацию через Azure Active Directory, прежде чем ему можно будет предоставить необходимые роли. После завершения репликации приведенный ниже пример выполняет итерацию определения политики в `$policyDef` для **roleDefinitionIds** и использует командлет [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment), чтобы предоставить роли новому управляемому удостоверению.

```azurepowershell-interactive
# Use the $policyDef to get to the roleDefinitionIds array
$roleDefinitionIds = $policyDef.Properties.policyRule.then.details.roleDefinitionIds

if ($roleDefinitionIds.Count -gt 0)
{
    $roleDefinitionIds | ForEach-Object {
        $roleDefId = $_.Split("/") | Select-Object -Last 1
        New-AzRoleAssignment -Scope $resourceGroup.ResourceId -ObjectId $assignment.Identity.PrincipalId -RoleDefinitionId $roleDefId
    }
}
```

### <a name="grant-defined-roles-through-portal"></a>Предоставление определенных ролей через портал

Существует два способа предоставления управляемых удостоверений назначения определенным ролям с помощью портала, с помощью **управления доступом (IAM)** или путем изменения политики или назначения инициативы и выбора команды **сохранить**.

Чтобы добавить роль для управляемого удостоверения назначения, выполните следующие действия:

1. Запустите службу "Политика Azure" на портале Azure. Для этого щелкните **Все службы**, выполните поиск по запросу **Политика** и выберите этот элемент.

1. Выберите **Назначения** на странице службы Политики Azure слева.

1. Перейдите к назначению с управляемым удостоверением и выберите его имя.

1. Найдите свойство **Идентификатор назначения** на странице изменения. Идентификатор назначения будет выглядеть примерно так:

   ```output
   /subscriptions/{subscriptionId}/resourceGroups/PolicyTarget/providers/Microsoft.Authorization/policyAssignments/2802056bfc094dfb95d4d7a5
   ```

   Имя управляемого удостоверения — это последняя часть идентификатора ресурса назначения (в этом примере `2802056bfc094dfb95d4d7a5`). Скопируйте эту часть идентификатора ресурса назначения.

1. Перейдите к ресурсу или родительскому контейнеру ресурсов (группа ресурсов, подписка, группа управления), для которого нужно вручную добавить определение роли.

1. На странице ресурсы выберите ссылку **Управление доступом (IAM)** , а затем в верхней части страницы контроля доступа выберите **+ добавить назначение ролей** .

1. Выберите нужную роль, соответствующую **roleDefinitionIds** определения политики.
   В поле **Назначение доступа к** оставьте значение по умолчанию "Пользователь, группа или приложение Azure AD". В поле **Выберите** вставьте или введите часть идентификатора ресурса назначения, обнаруженного ранее. После завершения поиска выберите объект с тем же именем, чтобы выбрать идентификатор, и нажмите кнопку **сохранить**.

## <a name="create-a-remediation-task"></a>Создание задачи исправления

### <a name="create-a-remediation-task-through-portal"></a>Создание задачи исправления с помощью портала

Во время оценки назначение политики с **deployIfNotExists** или **Modify** Effects определяет, есть ли несоответствующие ресурсы или подписки. При обнаружении несоответствующих ресурсов или подписок сведения о них предоставляются на странице **исправления** . Вместе со списком политик, которые имеют несоответствующие ресурсы или подписки, можно активировать **задачу исправления**.
Этот параметр позволяет создать развертывание из шаблона **deployIfNotExists** или операции **modify**.

Чтобы создать **задачу исправления**, выполните следующие действия:

1. Запустите службу "Политика Azure" на портале Azure. Для этого щелкните **Все службы**, выполните поиск по запросу **Политика** и выберите этот элемент.

   :::image type="content" source="../media/remediate-resources/search-policy.png" alt-text="Снимок экрана: поиск пункта &quot;Политика&quot; в области &quot;Все службы&quot;." border="false":::

1. Выберите **Исправление** на странице службы "Политика Azure" слева.

   :::image type="content" source="../media/remediate-resources/select-remediation.png" alt-text="Снимок экрана узла &quot;исправление&quot; на странице политики." border="false":::

1. Все назначения политики с **deployIfNotExists** или **modify**, для которых есть несоответствующие ресурсы, включаются в список на вкладке **Политики, подлежащие исправлению** и в таблицу данных. Выберите политику с ресурсами, которые не соответствуют требованиям. Откроется страница **Новая задача исправления**.

   > [!NOTE]
   > Другой способ открыть страницу **задачи исправления** — найти и выбрать политику на странице **соответствие** , а затем нажать кнопку **создать задачу исправления** .

1. На странице **Новая задача исправления** отфильтруйте ресурсы, которые нужно исправить, используя многоточие возле поля **Область**, чтобы выбрать дочерние ресурсы, которым была присвоена политика (в том числе отдельные объекты ресурса). Кроме того, используйте раскрывающийся список **Расположения**, чтобы дополнительно отфильтровать ресурсы. Будут исправлены только те ресурсы, которые перечислены в таблице.

   :::image type="content" source="../media/remediate-resources/select-resources.png" alt-text="Снимок экрана: исправьте узел и таблицу ресурсов для исправления." border="false":::

1. Запустите задачу исправления после фильтрации ресурсов, выбрав **исправить**. Откроется страница соответствия политики со вкладкой **Задачи исправления**, где отображается состояние хода выполнения задач. Развертывания, созданные задачей исправления, немедленно запускаются.

   :::image type="content" source="../media/remediate-resources/task-progress.png" alt-text="Снимок экрана: вкладка &quot;задачи исправления&quot; и ход выполнения существующих задач по исправлению." border="false":::

1. Чтобы получить сведения о ходе выполнения, выберите **задачу "исправление** " на странице "соответствие политике". Фильтрация, используемая для задачи, отображается вместе со списком ресурсов, которые будут исправлены.

1. На странице **задача по исправлению** щелкните ресурс правой кнопкой мыши, чтобы просмотреть развертывание или ресурс задачи по исправлению. В конце строки выберите **связанные события** , чтобы просмотреть такие сведения, как сообщение об ошибке.

   :::image type="content" source="../media/remediate-resources/resource-task-context-menu.png" alt-text="Снимок экрана: контекстное меню для ресурса на вкладке &quot;исправить задачу&quot;." border="false":::

Ресурсы, развернутые посредством **задачи исправления**, добавляются на вкладку **Развернутые ресурсы** на странице соответствия политике.

### <a name="create-a-remediation-task-through-azure-cli"></a>Создание задачи исправления с помощью Azure CLI

Чтобы создать **задачу исправления** с помощью Azure CLI, используйте команды `az policy remediation`. Замените `{subscriptionId}` значением идентификатора подписки, а `{myAssignmentId}` — идентификатором назначения политики с действием **deployIfNotExists** или **modify**.

```azurecli-interactive
# Login first with az login if not using Cloud Shell

# Create a remediation for a specific assignment
az policy remediation create --name myRemediation --policy-assignment '/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyAssignments/{myAssignmentId}'
```

Другие команды и примеры исправления см. в документации по командам [az policy remediation](/cli/azure/policy/remediation).

### <a name="create-a-remediation-task-through-azure-powershell"></a>Создание задачи исправления с помощью Azure PowerShell

Чтобы создать **задачу исправления** с помощью Azure PowerShell, используйте команды `Start-AzPolicyRemediation`. Замените `{subscriptionId}` значением идентификатора подписки, а `{myAssignmentId}` — идентификатором назначения политики с действием **deployIfNotExists** или **modify**.

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Create a remediation for a specific assignment
Start-AzPolicyRemediation -Name 'myRemedation' -PolicyAssignmentId '/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyAssignments/{myAssignmentId}'
```

Другие командлеты и примеры исправления см. в документации по модулю [Az.PolicyInsights](/powershell/module/az.policyinsights/#policy_insights).

### <a name="create-a-remediation-task-during-policy-assignment-in-the-azure-portal"></a>Создание задачи исправления в процессе назначения политики на портале Azure

Задачу исправления проще всего создать прямо при назначении политики на портале Azure. Если определение политики для назначения является **deployIfNotExists** или действием **изменения** , мастер на вкладке " **исправление** " предлагает параметр " _создать задачу исправления_ ". Если выбран этот параметр, задача исправления создается одновременно с назначением политики.

## <a name="next-steps"></a>Дальнейшие действия

- Рассмотрите [примеры для службы "Политика Azure"](../samples/index.md).
- Изучите статью о [структуре определения Политики Azure](../concepts/definition-structure.md).
- Изучите [сведения о действии политик](../concepts/effects.md).
- Узнайте о [программном создании политик](programmatically-create.md).
- Узнайте, как [получать данные о соответствии](get-compliance-data.md).
- Дополнительные сведения о группе управления см. в статье [Упорядочивание ресурсов с помощью групп управления Azure](../../management-groups/overview.md).
