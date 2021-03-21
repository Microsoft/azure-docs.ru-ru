---
title: Развертывание ресурсов в клиенте
description: В этой статье объясняется, как развертывать ресурсы в клиенте в шаблоне Azure Resource Manager.
ms.topic: conceptual
ms.date: 01/13/2021
ms.openlocfilehash: fd5a9ae60c578a3be7f70d82baae0a15e406b9db
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99491492"
---
# <a name="tenant-deployments-with-arm-templates"></a>Развертывание клиентов с помощью шаблонов ARM

По мере развития вашей организации может потребоваться определить и назначить [политики](../../governance/policy/overview.md) или [Управление доступом на основе ролей Azure (Azure RBAC)](../../role-based-access-control/overview.md) в клиенте Azure AD. С помощью шаблонов уровня клиента можно декларативно применять политики и назначать роли на глобальном уровне.

## <a name="supported-resources"></a>Поддерживаемые ресурсы

Не все типы ресурсов можно развернуть на уровне клиента. В этом разделе перечислены поддерживаемые типы ресурсов.

Для политик Azure используйте:

* [policyAssignments](/azure/templates/microsoft.authorization/policyassignments);
* [policyDefinitions](/azure/templates/microsoft.authorization/policydefinitions);
* [policySetDefinitions](/azure/templates/microsoft.authorization/policysetdefinitions);

Для управления доступом на основе ролей Azure (Azure RBAC) используйте:

* [roleAssignments](/azure/templates/microsoft.authorization/roleassignments);

Для вложенных шаблонов, которые развертываются в группах управления, подписках или группах ресурсов, используйте:

* [размещения](/azure/templates/microsoft.resources/deployments)

Для создания групп управления используйте:

* [managementGroups](/azure/templates/microsoft.management/managementgroups)

Для создания подписок используйте:

* [псевдоним](/azure/templates/microsoft.subscription/aliases)

Для управления затратами используйте:

* [биллингпрофилес](/azure/templates/microsoft.billing/billingaccounts/billingprofiles)
* [водим](/azure/templates/microsoft.billing/billingaccounts/billingprofiles/instructions)
* [инвоицесектионс](/azure/templates/microsoft.billing/billingaccounts/billingprofiles/invoicesections)

Для настройки портала используйте:

* [тенантконфигуратионс](/azure/templates/microsoft.portal/tenantconfigurations)

## <a name="schema"></a>схема

Схема, используемая для развертываний клиентов, отличается от схемы развертываний группы ресурсов.

Для шаблонов используйте:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#",
    ...
}
```

Схема для файла параметров одинакова для всех областей развертывания. Для файлов параметров используйте:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    ...
}
```

## <a name="required-access"></a>Требуемый доступ

У субъекта, развертывающего шаблон, должны быть разрешения на создание ресурсов в области клиента. У субъекта должно быть разрешение на действия развертывания (`Microsoft.Resources/deployments/*`) и создание ресурсов, определенных в шаблоне. Например, чтобы создать группу управления, у субъекта должно быть разрешение "Участник" в области клиента. Чтобы создавать назначения ролей, у участника должно быть разрешение "Владелец".

Глобальный администратор Azure Active Directory не получает разрешения на назначение ролей автоматически. Чтобы включить развертывания шаблонов в области клиента, глобальному администратору следует выполнить такие действия:

1. Повысить уровень доступа к учетной записи, чтобы глобальный администратор мог назначать роли. Дополнительные сведения см. в статье [Повышение прав доступа для управления всеми подписками Azure и группами управления](../../role-based-access-control/elevate-access-global-admin.md).

1. Назначьте роль "Владелец" или "Участник" субъекту, который будет развертывать шаблоны.

   ```azurepowershell-interactive
   New-AzRoleAssignment -SignInName "[userId]" -Scope "/" -RoleDefinitionName "Owner"
   ```

   ```azurecli-interactive
   az role assignment create --assignee "[userId]" --scope "/" --role "Owner"
   ```

У субъекта теперь есть необходимые разрешения для развертывания шаблона.

## <a name="deployment-commands"></a>Команды развертывания

Команды, используемые для развертываний клиентов, отличаются от команд для развертываний группы ресурсов.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Для Azure CLI используйте [az deployment tenant create](/cli/azure/deployment/tenant#az-deployment-tenant-create):

```azurecli-interactive
az deployment tenant create \
  --name demoTenantDeployment \
  --location WestUS \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/tenant-deployments/new-mg/azuredeploy.json"
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Для Azure PowerShell используйте [New-AzTenantDeployment](/powershell/module/az.resources/new-aztenantdeployment).

```azurepowershell-interactive
New-AzTenantDeployment `
  -Name demoTenantDeployment `
  -Location "West US" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/tenant-deployments/new-mg/azuredeploy.json"
```

---

Более подробные сведения о командах и параметрах развертывания для развертывания шаблонов ARM см. в следующих статьях:

* [Развертывание ресурсов с помощью шаблонов ARM и портал Azure](deploy-portal.md)
* [Развертывание ресурсов с помощью шаблонов ARM и Azure CLI](deploy-cli.md)
* [Развертывание ресурсов с помощью шаблонов ARM и Azure PowerShell](deploy-powershell.md)
* [Развертывание ресурсов с помощью шаблонов ARM и Azure Resource Manager REST API](deploy-rest.md)
* [Использование кнопки развертывания для развертывания шаблонов из репозитория GitHub](deploy-to-azure-button.md)
* [Развертывание шаблонов ARM из Cloud Shell](deploy-cloud-shell.md)

## <a name="deployment-location-and-name"></a>Расположение и имя развертывания

Для развертываний на уровне клиента необходимо указать расположение для развертывания. Расположение развертывания отделено от расположения развертываемых ресурсов. В расположении развертывания указывается место хранения данных развертывания. Для развертываний [подписок](deploy-to-subscription.md) и [групп управления](deploy-to-management-group.md) также требуется расположение. Для развертываний [группы](deploy-to-resource-group.md) ресурсов расположение группы ресурсов используется для хранения данных развертывания.

Можно указать имя развертывания или использовать имя развертывания по умолчанию. Имя по умолчанию — это имя файла шаблона. Например, развернув шаблон с именем _azuredeploy.json_ создается имя развертывания по умолчанию **azuredeploy**.

Для каждого имени развертывания расположение остается неизменным. Нельзя создать развертывание в одном расположении, если в другом уже есть развертывание с таким же именем. Например, если вы создадите развертывание клиента с именем **deployment1** в **centralus**, вы не сможете позднее создать другое развертывание с именем **deployment1** , а расположение **westus**. Если появится код ошибки `InvalidDeploymentLocation`, используйте другое имя или то же расположение, что и для предыдущего развертывания с этим именем.

## <a name="deployment-scopes"></a>Области развертывания

При развертывании в клиенте можно развернуть ресурсы в:

* Клиент
* группы управления в клиенте
* subscriptions
* группы ресурсов

[Ресурс расширения](scope-extension-resources.md) можно ограничить целевым объектом, который отличается от целевого объекта развертывания.

Пользователь, развертывающий шаблон, должен иметь доступ к указанной области.

В этом разделе показано, как указать различные области. Эти различные области можно объединить в один шаблон.

### <a name="scope-to-tenant"></a>Область для клиента

Ресурсы, определенные в разделе ресурсов шаблона, применяются к клиенту.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/default-tenant.json" highlight="5":::

### <a name="scope-to-management-group"></a>Область в группе управления

Чтобы выбрать целевую группу управления в клиенте, добавьте вложенное развертывание и укажите `scope` свойство.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/tenant-to-mg.json" highlight="10,17,18,22":::

### <a name="scope-to-subscription"></a>Область действия для подписки

Вы также можете ориентироваться на подписки в клиенте. Пользователь, развертывающий шаблон, должен иметь доступ к указанной области.

Чтобы назначить подписку в клиенте, используйте вложенное развертывание и `subscriptionId` свойство.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/tenant-to-subscription.json" highlight="9,10,18":::

### <a name="scope-to-resource-group"></a>Область действия для группы ресурсов

Вы также можете ориентироваться на группы ресурсов в клиенте. Пользователь, развертывающий шаблон, должен иметь доступ к указанной области.

Чтобы назначить группу ресурсов в клиенте, используйте вложенное развертывание. Укажите свойства `subscriptionId` и `resourceGroup`. Не задавайте расположение для вложенного развертывания, так как оно развертывается в расположении группы ресурсов.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/tenant-to-rg.json" highlight="9,10,18":::

## <a name="create-management-group"></a>Создание группы управления

В следующем примере создается группа управления:

:::code language="json" source="~/quickstart-templates/tenant-deployments/new-mg/azuredeploy.json":::

Если у вашей учетной записи нет разрешения на развертывание в клиенте, вы все равно можете создавать группы управления, развернув их в другую область. Дополнительные сведения см. в разделе [Группа управления](deploy-to-management-group.md#management-group).

## <a name="assign-role"></a>Назначение роли

В следующем шаблоне назначается роль в области клиента:

:::code language="json" source="~/quickstart-templates/tenant-deployments/tenant-role-assignment/azuredeploy.json":::

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о назначении ролей см. в статье [Добавление назначений ролей Azure с помощью шаблонов Azure Resource Manager](../../role-based-access-control/role-assignments-template.md).
* Кроме того, можно развертывать шаблоны на уровне [подписки](deploy-to-subscription.md) или [группы управления](deploy-to-management-group.md).
