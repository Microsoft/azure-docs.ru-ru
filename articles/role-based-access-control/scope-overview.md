---
title: Общие сведения об области для Azure RBAC
description: Сведения об управлении доступом на основе ролей в Azure (Azure RBAC) и о том, как определить область действия ресурса.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 10/08/2020
ms.author: rolyon
ms.openlocfilehash: deee42c46c9b08bb265c972695b9319413d4fcb1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100555914"
---
# <a name="understand-scope-for-azure-rbac"></a>Общие сведения об области для Azure RBAC

*Область* — это набор ресурсов, к которым применяется доступ. При назначении роли важно понимать область, чтобы вы могли предоставить субъекту безопасности только тот доступ, который ему действительно нужен. Ограничивая область, вы ограничиваете ресурсы, подверженные риску, в случае компрометации субъекта безопасности.

## <a name="scope-levels"></a>Уровни области

В Azure можно указать область на четырех уровнях: [Группа управления](../governance/management-groups/overview.md), подписка, [Группа ресурсов](../azure-resource-manager/management/overview.md#resource-groups)и ресурс. Структура областей строится на отношениях "родитель-потомок". Каждый уровень иерархии делает область более конкретной. Вы можете назначать роли на любом из этих уровней области действия. Выбранный уровень определяет, насколько широко применяется роль. Более низкие уровни наследуют разрешения роли от более высоких уровней. 

![Область для назначения ролей](./media/scope-overview/rbac-scope-no-label.png)

Группы управления — это уровень области действия подписок, но группы управления поддерживают более сложные иерархии. На следующей диаграмме показан пример иерархии групп управления и подписок, которые можно определить. Дополнительные сведения о группах управления см. в статье [что такое группы управления Azure?](../governance/management-groups/overview.md).

![Иерархия группы управления и подписки](./media/scope-overview/rbac-scope-management-groups.png)

## <a name="scope-format"></a>Формат области

При назначении ролей с помощью командной строки необходимо указать область. Для программ командной строки область является потенциально длинной строкой, определяющей точную область назначения роли. В портал Azure эта область обычно указывается как *идентификатор ресурса*.

Область состоит из последовательности идентификаторов, разделенных символом косой черты (/). Эту строку можно представить как выражение следующей иерархии, где текстом без заполнителей (`{}`) являются фиксированные идентификаторы:

```
/subscriptions
    /{subscriptionId}
        /resourcegroups
            /{resourceGroupName}
                /providers
                    /{providerName}
                        /{resourceType}
                            /{resourceSubType1}
                                /{resourceSubType2}
                                    /{resourceName}
```

- `{subscriptionId}` — идентификатор используемой подписки (GUID).
- `{resourcesGroupName}` — имя содержащейся группы ресурсов.
- `{providerName}` имя [поставщика ресурсов](../azure-resource-manager/management/azure-services-resource-providers.md) , который обрабатывает ресурс, `{resourceType}` а затем `{resourceSubType*}` определяет дополнительные уровни в этом поставщике ресурсов.
- `{resourceName}` — последняя часть строки, идентифицирующая конкретный ресурс.

Группы управления — это уровень выше подписок и имеют широкую (наименее конкретную) область. Назначения ролей на этом уровне применяются к подпискам в группе управления. Область группы управления имеет следующий формат:

```
/providers
    /Microsoft.Management
        /managementGroups
            /{managmentGroupName}
```

## <a name="scope-examples"></a>Примеры областей

> [!div class="mx-tableFixed"]
> | Область | Пример |
> | --- | --- |
> | группа управления; | `/providers/Microsoft.Management/managementGroups/marketing-group` |
> | Подписка | `/subscriptions/00000000-0000-0000-0000-000000000000` |
> | Группа ресурсов | `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg` |
> |  | `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales` |
> | Ресурс | `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/azurestorage12345/blobServices/default/containers/blob-container-01` |
> |  | `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/MyVirtualNetworkResourceGroup/providers/Microsoft.Network/virtualNetworks/MyVirtualNetwork12345` |

## <a name="how-to-determine-the-scope-for-a-resource"></a>Определение области действия для ресурса

Определить область группы управления, подписки или группы ресурсов довольно просто. Вам нужно просто знать имя и идентификатор подписки. Однако определение области действия ресурса занимает немного больше усилий. Вот несколько способов определения области действия ресурса.

- В портал Azure откройте ресурс и просмотрите свойства. Ресурс должен перечислить **идентификатор ресурса** , в котором можно определить область. Например, ниже приведены идентификаторы ресурсов для учетной записи хранения.

    ![Идентификаторы ресурсов для учетной записи хранения в портал Azure](./media/scope-overview/scope-resource-id.png)

- Другой способ — использовать портал Azure, чтобы временно назначить роль в области действия ресурса, а затем использовать [Azure PowerShell](role-assignments-list-powershell.md) или [Azure CLI](role-assignments-list-cli.md) для перечисления назначения ролей. В выходных данных область будет отображаться как свойство.

    ```azurepowershell
    RoleAssignmentId   : /subscriptions/<subscriptionId>/resourceGroups/test-rg/providers/Microsoft.Storage/storageAccounts/azurestorage12345/blobServices/default/containers/blob-container-01/pro
                         viders/Microsoft.Authorization/roleAssignments/<roleAssignmentId>
    Scope              : /subscriptions/<subscriptionId>/resourceGroups/test-rg/providers/Microsoft.Storage/storageAccounts/azurestorage12345/blobServices/default/containers/blob-container-01
    DisplayName        : User
    SignInName         : user@contoso.com
    RoleDefinitionName : Storage Blob Data Reader
    RoleDefinitionId   : 2a2b9908-6ea1-4ae2-8e65-a410df84e7d1
    ObjectId           : <principalId>
    ObjectType         : User
    CanDelegate        : False
    Description        :
    ConditionVersion   :
    Condition          :
    ```

    ```azurecli
    {
        "canDelegate": null,
        "condition": null,
        "conditionVersion": null,
        "description": null,
        "id": "/subscriptions/{subscriptionId}/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/azurestorage12345/blobServices/default/containers/blob-container-01/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
        "name": "{roleAssignmentId}",
        "principalId": "{principalId}",
        "principalName": "user@contoso.com",
        "principalType": "User",
        "resourceGroup": "test-rg",
        "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
        "roleDefinitionName": "Storage Blob Data Reader",
        "scope": "/subscriptions/{subscriptionId}/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/azurestorage12345/blobServices/default/containers/blob-container-01",
        "type": "Microsoft.Authorization/roleAssignments"
      }
    ```

## <a name="next-steps"></a>Дальнейшие действия

- [Действия по назначению роли Azure](role-assignments-steps.md)
- [Поставщики ресурсов для служб Azure](../azure-resource-manager/management/azure-services-resource-providers.md)
- [Что такое группы управления Azure?](../governance/management-groups/overview.md)
