---
title: Работа с группами управления — Управление Azure
description: Узнайте, как просматривать, сохранять, обновлять и удалять иерархию групп управления.
ms.date: 01/15/2021
ms.topic: conceptual
ms.openlocfilehash: 05e78d66c29e500842d14a6eeb563c4569ecf0bd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100596481"
---
# <a name="manage-your-resources-with-management-groups"></a>Управление ресурсами с помощью групп управления

Если в вашей организации оформлено много подписок, возможно, потребуется повысить эффективность управления доступом, политиками и соответствием требованиям для этих подписок. Группы управления Azure обеспечивают высокий уровень охвата подписок. Подписки упорядочиваются в контейнеры, которые называются группами управления. К ним применяются условия системы управления. Все подписки в группе управления автоматически наследуют условия, применяемые к группе управления.

Группы управления обеспечивают корпоративное управление в больших масштабах независимо от типа подписки. Дополнительные сведения о группах управления см. в статье [Упорядочение ресурсов с помощью групп управления Azure](./overview.md).

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

> [!IMPORTANT]
> Токены пользователей и кэш группы управления в Azure Resource Manager сохраняются в течение 30 минут, а затем принудительно обновляются. После выполнения любых действий, таких как перемещение группы управления или подписки, изменения могут отобразиться только через 30 минут. Чтобы увидеть их раньше, вам нужно принудительно обновить токен. Для этого обновите страницу в браузере, выйдите из системы и снова войдите, или запросите новый токен.  

## <a name="change-the-name-of-a-management-group"></a>Изменение имени группы управления

Имя группы управления можно изменить с помощью портала, PowerShell или Azure CLI.

### <a name="change-the-name-in-the-portal"></a>Изменение имени на портале

1. Войдите на [портал Azure](https://portal.azure.com).

1. Выберите **Все службы** > **Группы управления**.

1. Выберите группу управления, которую требуется переименовать.

1. Щелкните **Сведения**.

1. Выберите параметр **Переименовать группу** в верхней части страницы.

   :::image type="content" source="./media/detail_action_small.png" alt-text="Снимок экрана: панель действий и кнопка &quot;переименовать группу&quot; на странице группы управления." border="false":::

1. Когда откроется меню, введите новое имя.

   :::image type="content" source="./media/rename_context.png" alt-text="Снимок экрана: окно &quot;Переименование группы&quot; и параметры для переименования группы управления." border="false":::

1. Щелкните **Сохранить**.

### <a name="change-the-name-in-powershell"></a>Изменение имени в PowerShell

Чтобы обновить отображаемое имя, выполните команду **Update-AzManagementGroup**. Например, чтобы изменить отображаемое имя группы управления с Contoso IT на Contoso Group, выполните следующую команду.

```azurepowershell-interactive
Update-AzManagementGroup -GroupName 'ContosoIt' -DisplayName 'Contoso Group'
```

### <a name="change-the-name-in-azure-cli"></a>Изменение имени в Azure CLI

Для Azure CLI используется команда обновления.

```azurecli-interactive
az account management-group update --name 'Contoso' --display-name 'Contoso Group'
```

## <a name="delete-a-management-group"></a>Удаление группы управления

Чтобы удалить группу управления, необходимо выполнить следующие требования:

1. В группе управления нет дочерних групп управления или подписок. Сведения о перемещении подписки или группы управления в другую группу управления см. [в разделе Перемещение групп управления и подписок в иерархии](#moving-management-groups-and-subscriptions).

1. Вам потребуются разрешения на запись для группы управления (роль "Владелец", "Участник" или "Участник группы управления"). Чтобы получить сведения об имеющихся разрешениях, выберите группу управления, а затем выберите **IAM**. Дополнительные сведения о ролях Azure см. в разделе  
   [Управление доступом на основе ролей в Azure (Azure RBAC)](../../role-based-access-control/overview.md).

### <a name="delete-in-the-portal"></a>Удаление на портале

1. Войдите на [портал Azure](https://portal.azure.com).

1. Выберите **Все службы** > **Группы управления**.

1. Выберите группу управления, которую требуется удалить.

1. Щелкните **Сведения**.

1. Выберите **Удалить**.

   :::image type="content" source="./media/delete.png" alt-text="Снимок экрана: страница группы управления с выделенной кнопкой &quot;Удалить&quot;." border="false":::

   > [!TIP]
   > Если значок неактивен, наведите указатель мыши на значок, чтобы узнать причину.

1. Откроется окно, в котором нужно подтвердить удаление группы управления.

   :::image type="content" source="./media/delete_confirm.png" alt-text="Снимок экрана с диалоговым окном подтверждения &quot;Удаление группы&quot; для удаления группы управления." border="false":::

1. Выберите **Да**.

### <a name="delete-in-powershell"></a>Удаление в PowerShell

В PowerShell для удаления группы управления выполните команду **Remove-AzManagementGroup**.

```azurepowershell-interactive
Remove-AzManagementGroup -GroupName 'Contoso'
```

### <a name="delete-in-azure-cli"></a>Удаление в Azure CLI

В Azure CLI используется команда az account management-group delete.

```azurecli-interactive
az account management-group delete --name 'Contoso'
```

## <a name="view-management-groups"></a>Просмотр групп управления

Вы можете просмотреть любую группу управления, на которой имеется прямая или унаследованная роль Azure.  

### <a name="view-in-the-portal"></a>Просмотр на портале

1. Войдите на [портал Azure](https://portal.azure.com).

1. Выберите **Все службы** > **Группы управления**.

1. Загрузится страница иерархического дерева группы управления. На этой странице вы можете изучить все группы управления и подписки, к которым у вас есть доступ. Выбор имени группы приводит к переходу на более низкий уровень иерархии. Навигация работает так же, как и в обозревателе файлов.

1. Чтобы просмотреть сведения о группе управления, перейдите по ссылке **(подробности)** рядом с заголовком группы управления. Если ссылка недоступна, у вас нет разрешения на просмотр этой группы управления.

   :::image type="content" source="./media/main.png" alt-text="Снимок экрана: страница &quot;Группы управления&quot;, на которой отображаются дочерние группы управления и подписки." border="false":::

### <a name="view-in-powershell"></a>Просмотр в PowerShell

Для получения всех групп выполните команду Get-AzManagementGroup. Полный список команд GET для группы управления в PowerShell см. в документации по модулям [Az.Resources](/powershell/module/az.resources/Get-AzManagementGroup).  

```azurepowershell-interactive
Get-AzManagementGroup
```

Для получения сведений об одной группе управления используйте параметр -GroupName.

```azurepowershell-interactive
Get-AzManagementGroup -GroupName 'Contoso'
```

Чтобы получить определенную группу управления и все ее вложенные элементы, используйте параметры **-Expand** и **-Recurse**.  

```azurepowershell-interactive
PS C:\> $response = Get-AzManagementGroup -GroupName TestGroupParent -Expand -Recurse
PS C:\> $response

Id                : /providers/Microsoft.Management/managementGroups/TestGroupParent
Type              : /providers/Microsoft.Management/managementGroups
Name              : TestGroupParent
TenantId          : 00000000-0000-0000-0000-000000000000
DisplayName       : TestGroupParent
UpdatedTime       : 2/1/2018 11:15:46 AM
UpdatedBy         : 00000000-0000-0000-0000-000000000000
ParentId          : /providers/Microsoft.Management/managementGroups/00000000-0000-0000-0000-000000000000
ParentName        : 00000000-0000-0000-0000-000000000000
ParentDisplayName : 00000000-0000-0000-0000-000000000000
Children          : {TestGroup1DisplayName, TestGroup2DisplayName}

PS C:\> $response.Children[0]

Type        : /managementGroup
Id          : /providers/Microsoft.Management/managementGroups/TestGroup1
Name        : TestGroup1
DisplayName : TestGroup1DisplayName
Children    : {TestRecurseChild}

PS C:\> $response.Children[0].Children[0]

Type        : /managementGroup
Id          : /providers/Microsoft.Management/managementGroups/TestRecurseChild
Name        : TestRecurseChild
DisplayName : TestRecurseChild
Children    :
```

### <a name="view-in-azure-cli"></a>Просмотр в Azure CLI

Для извлечения всех групп используется команда вывода списка.  

```azurecli-interactive
az account management-group list
```

Для получения сведений об одной группе управления используется команда отображения.

```azurecli-interactive
az account management-group show --name 'Contoso'
```

Чтобы получить определенную группу управления и все ее вложенные элементы, используйте параметры **-Expand** и **-Recurse**.

```azurecli-interactive
az account management-group show --name 'Contoso' -e -r
```

## <a name="moving-management-groups-and-subscriptions"></a>Перемещение групп управления и подписок   

Одна из причин для создания группы управления — объединение подписок. Дочерним элементом группы управления могут быть только подписки и группы управления. При перемещении в группу управления подписка наследует все права доступа и политики пользователя из родительской группы управления.

При перемещении группы управления или подписки в качестве дочерней для другой группы управления необходимо оценить три правила, как true.

Чтобы выполнить перемещение, вам потребуется следующее: 

- Разрешения на запись сведений о группе управления и назначении роли в дочерней подписке или группе управления.
  - Пример встроенной роли: **Владелец**
- Разрешения на запись сведений о группе управления в целевой родительской группе управления.
  - Пример встроенной роли: **Владелец**, **Участник**, **Участник группы управления**.
- Разрешения на запись сведений о группе управления в нынешней родительской группе управления.
  - Пример встроенной роли: **Владелец**, **Участник**, **Участник группы управления**.

**Исключение**. Если целевой или нынешней родительской группой управления является корневая группа управления, требования к разрешениям не применяются. Так как корневая группа управления по умолчанию считается родительской для всех новых групп управления и подписок, перемещение элементов в нее не требует дополнительных разрешений.

Если роль владельца подписки наследуется из нынешней родительской группы управления, выбор целевых объектов для перемещения будет ограничен. Вы можете переместить подписку только в другую группу управления, для которой у вас есть роль владельца. Вы не можете переместить подписку в группу управления, в которой вы только являетесь участником, так как вы потеряли права владельца подписки. Если роль владельца подписки напрямую назначена, ее можно переместить в любую группу управления, в которой вы являетесь участником.

Чтобы получить на портале Azure сведения о существующих разрешениях, выберите группу управления и щелкните **IAM**. Дополнительные сведения о ролях Azure см. в статье [Управление доступом на основе ролей в Azure (Azure RBAC)](../../role-based-access-control/overview.md).

## <a name="move-subscriptions"></a>Перемещение подписок 

### <a name="add-an-existing-subscription-to-a-management-group-in-the-portal"></a>Добавление существующей подписки в группу управления с помощью портала

1. Войдите на [портал Azure](https://portal.azure.com).

1. Выберите **Все службы** > **Группы управления**.

1. Выберите группу управления, которую вы планируете сделать родительской.

1. В верхней части страницы выберите пункт **Добавить подписку**.

1. Выберите подписку в списке с правильным идентификатором.

   :::image type="content" source="./media/add_context_sub.png" alt-text="Снимок экрана с параметрами &quot;добавить подписку&quot; для выбора существующей подписки, добавляемой в группу управления." border="false":::

1. Щелкните "Сохранить".

### <a name="remove-a-subscription-from-a-management-group-in-the-portal"></a>Удаление подписки из группы управления с помощью портала

1. Войдите на [портал Azure](https://portal.azure.com).

1. Выберите **Все службы** > **Группы управления**.

1. Выберите группу управления, которая является родительской.  

1. Выберите многоточие в конце строки подписки в списке, которую требуется переместить.

   :::image type="content" source="./media/move_small.png" alt-text="Снимок экрана альтернативного меню подписки для выбора параметра &quot;Переместить&quot;." border="false":::

1. Выберите **Переместить**.

1. В открывшемся меню выберите **родительскую группу управления**.

   :::image type="content" source="./media/move_small_context.png" alt-text="Снимок экрана: окно &quot;Переместить&quot; и параметры для перемещения подписки в другую группу управления." border="false":::

1. Щелкните **Сохранить**.

### <a name="move-subscriptions-in-powershell"></a>Перемещение подписок в PowerShell

Чтобы переместить подписку с помощью PowerShell, выполните команду New-AzManagementGroupSubscription.  

```azurepowershell-interactive
New-AzManagementGroupSubscription -GroupName 'Contoso' -SubscriptionId '12345678-1234-1234-1234-123456789012'
```

Чтобы удалить связь между подпиской и группой управления, выполните команду Remove-AzManagementGroupSubscription.

```azurepowershell-interactive
Remove-AzManagementGroupSubscription -GroupName 'Contoso' -SubscriptionId '12345678-1234-1234-1234-123456789012'
```

### <a name="move-subscriptions-in-azure-cli"></a>Перемещение подписок в Azure CLI

Чтобы переместить подписку в CLI, используйте команду добавления.

```azurecli-interactive
az account management-group subscription add --name 'Contoso' --subscription '12345678-1234-1234-1234-123456789012'
```

Чтобы удалить подписку из группы управления, используйте команду удаления.  

```azurecli-interactive
az account management-group subscription remove --name 'Contoso' --subscription '12345678-1234-1234-1234-123456789012'
```

### <a name="move-subscriptions-in-arm-template"></a>Перемещение подписок в шаблоне ARM

Чтобы переместить подписку в шаблоне Azure Resource Manager (шаблон ARM), используйте следующий шаблон.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "targetMgId": {
            "type": "string",
            "metadata": {
                "description": "Provide the ID of the management group that you want to move the subscription to."
            }
        },
        "subscriptionId": {
            "type": "string",
            "metadata": {
                "description": "Provide the ID of the existing subscription to move."
            }
        }
    },
    "resources": [
        {
            "scope": "/", 
            "type": "Microsoft.Management/managementGroups/subscriptions",
            "apiVersion": "2020-05-01",
            "name": "[concat(parameters('targetMgId'), '/', parameters('subscriptionId'))]",
            "properties": {
            }
        }
    ],
    "outputs": {}
}
```

## <a name="move-management-groups"></a>Перемещение групп управления 

### <a name="move-management-groups-in-the-portal"></a>Перемещение групп управления на портале

1. Войдите на [портал Azure](https://portal.azure.com).

1. Выберите **Все службы** > **Группы управления**.

1. Выберите группу управления, которую вы планируете сделать родительской.

1. В верхней части страницы выберите пункт **Добавить группу управления**.

1. В открывшемся меню выберите создать новую или использовать существующую группу управления.

   - Если вы выберете новую, то создадите новую группу управления.
   - Если вы выберете существующую, вы сможете выбрать доступную группу управления из раскрывающегося списка.  

   :::image type="content" source="./media/add_context_MG.png" alt-text="Снимок экрана: окно &quot;Добавить группу управления&quot;, в котором показаны параметры для создания группы управления." border="false":::

1. Щелкните **Сохранить**.

### <a name="move-management-groups-in-powershell"></a>Перемещение групп управления в PowerShell

Чтобы переместить группу управления в другую группу с помощью PowerShell, выполните команду Update-AzManagementGroup.

```azurepowershell-interactive
$parentGroup = Get-AzManagementGroup -GroupName ContosoIT
Update-AzManagementGroup -GroupName 'Contoso' -ParentId $parentGroup.id
```  

### <a name="move-management-groups-in-azure-cli"></a>Перемещение групп управления в Azure CLI

Чтобы переместить группу управления с помощью Azure CLI, используйте команду обновления.

```azurecli-interactive
az account management-group update --name 'Contoso' --parent ContosoIT
```

## <a name="audit-management-groups-using-activity-logs"></a>Аудит групп управления с помощью журналов действий

[Журнал действий Azure](../../azure-monitor/essentials/platform-logs-overview.md) поддерживает группы управления. Сведения обо всех событиях, происходящих в группе управления, можно получить в том же централизованном расположении, что и для других ресурсов Azure. Например, вы можете просмотреть сведения обо всех изменениях в назначениях ролей или назначении политик, внесенные в пределах определенной группы управления.

:::image type="content" source="./media/al-mg.png" alt-text="Снимок экрана журналов действий и операций, связанных с выбранной группой управления." border="false":::

Если нужно создать запрос к группам управления вне портала Azure, целевая область для групп управления будет такой: **"/providers/Microsoft.Management/managementGroups/{идентификатор_вашей_группы_управления}"** .

## <a name="referencing-management-groups-from-other-resource-providers"></a>Ссылки на группы управления из других поставщиков ресурсов

При использовании ссылок на группы управления из действий другого поставщика ресурсов используйте в качестве области следующий путь. Этот путь используется при работе с PowerShell, Azure CLI и REST API.  

`/providers/Microsoft.Management/managementGroups/{yourMgID}`

Вот пример использования этого пути при назначении новой роли для группы управления с помощью PowerShell.

```azurepowershell-interactive
New-AzRoleAssignment -Scope "/providers/Microsoft.Management/managementGroups/Contoso"
```

При извлечении определения политики в группе управления используется тот же путь к области.

```http
GET https://management.azure.com/providers/Microsoft.Management/managementgroups/MyManagementGroup/providers/Microsoft.Authorization/policyDefinitions/ResourceNaming?api-version=2019-09-01
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о группах управления:

- [Создание групп управления для организации ресурсов Azure](./create-management-group-portal.md)
- [Изменение, удаление групп управления и управление ими](./manage.md)
- [Просмотр групп управления в модуле ресурсов Azure PowerShell](/powershell/module/az.resources#resources)
- [Просмотр групп управления в REST API](/rest/api/resources/managementgroups)
- [Просмотр групп управления в Azure CLI](/cli/azure/account/management-group)