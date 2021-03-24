---
title: Развертывание ресурсов в группе управления
description: Описывает развертывание ресурсов в области группы управления в шаблоне Azure Resource Manager.
ms.topic: conceptual
ms.date: 03/18/2021
ms.openlocfilehash: 603b7e32e6f4e1181a8ef2df67382b5e21ed6715
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104889813"
---
# <a name="management-group-deployments-with-arm-templates"></a>Развертывание группы управления с помощью шаблонов ARM

По мере развития вашей организации можно развернуть шаблон Azure Resource Manager (шаблон ARM) для создания ресурсов на уровне группы управления. Например, может потребоваться определить и назначить [политики](../../governance/policy/overview.md) или [Управление доступом на основе ролей Azure (Azure RBAC)](../../role-based-access-control/overview.md) для группы управления. С помощью шаблонов уровня группы управления можно декларативно применять политики и назначать роли на уровне группы управления.

## <a name="supported-resources"></a>Поддерживаемые ресурсы

Не все типы ресурсов можно развернуть на уровне группы управления. В этом разделе перечислены поддерживаемые типы ресурсов.

Для чертежей Azure используйте:

* [artifacts](/azure/templates/microsoft.blueprint/blueprints/artifacts)
* [blueprints](/azure/templates/microsoft.blueprint/blueprints);
* [блуепринтассигнментс](/azure/templates/microsoft.blueprint/blueprintassignments)
* [операционных](/azure/templates/microsoft.blueprint/blueprints/versions)

Для политик Azure используйте:

* [policyAssignments](/azure/templates/microsoft.authorization/policyassignments);
* [policyDefinitions](/azure/templates/microsoft.authorization/policydefinitions);
* [policySetDefinitions](/azure/templates/microsoft.authorization/policysetdefinitions);
* [remediations](/azure/templates/microsoft.policyinsights/remediations);

Для управления доступом на основе ролей Azure (Azure RBAC) используйте:

* [roleAssignments](/azure/templates/microsoft.authorization/roleassignments);
* [roleDefinitions](/azure/templates/microsoft.authorization/roledefinitions).

Для вложенных шаблонов, которые развертываются в подписках или группах ресурсов, используйте:

* [размещения](/azure/templates/microsoft.resources/deployments)

Для управления ресурсами используйте:

* [теги](/azure/templates/microsoft.resources/tags)

Группы управления — это ресурсы уровня клиента. Однако вы можете создать группы управления в развертывании группы управления, задав область действия новой группы управления для клиента. См. раздел [Группа управления](#management-group).

## <a name="schema"></a>схема

Схема, используемая для развертываний группы управления, отличается от схемы развертываний группы ресурсов.

Для шаблонов используйте:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
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

## <a name="deployment-commands"></a>Команды развертывания

Для развертывания в группе управления используйте команды развертывания группы управления.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Для Azure CLI используйте команду [AZ Deployment mg Create](/cli/azure/deployment/mg#az-deployment-mg-create):

```azurecli-interactive
az deployment mg create \
  --name demoMGDeployment \
  --location WestUS \
  --management-group-id myMG \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/management-level-deployment/azuredeploy.json"
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Для Azure PowerShell используйте [New-азманажементграупдеплоймент](/powershell/module/az.resources/new-azmanagementgroupdeployment).

```azurepowershell-interactive
New-AzManagementGroupDeployment `
  -Name demoMGDeployment `
  -Location "West US" `
  -ManagementGroupId "myMG" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/management-level-deployment/azuredeploy.json"
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

Для развертываний на уровне группы управления необходимо указать расположение для развертывания. Расположение развертывания отделено от расположения развертываемых ресурсов. В расположении развертывания указывается место хранения данных развертывания. Для развертываний [подписок](deploy-to-subscription.md) и [клиентов](deploy-to-tenant.md) также требуется расположение. Для развертываний [группы](deploy-to-resource-group.md) ресурсов расположение группы ресурсов используется для хранения данных развертывания.

Можно указать имя развертывания или использовать имя развертывания по умолчанию. Имя по умолчанию — это имя файла шаблона. Например, развернув шаблон с именем _azuredeploy.json_ создается имя развертывания по умолчанию **azuredeploy**.

Для каждого имени развертывания расположение остается неизменным. Нельзя создать развертывание в одном расположении, если в другом уже есть развертывание с таким же именем. Например, если вы создадите развертывание группы управления с именем **deployment1** в **centralus**, вы не сможете позднее создать другое развертывание с именем **deployment1** , а расположение **westus**. Если появится код ошибки `InvalidDeploymentLocation`, используйте другое имя или то же расположение, что и для предыдущего развертывания с этим именем.

## <a name="deployment-scopes"></a>Области развертывания

При развертывании в группе управления можно развернуть ресурсы в:

* Целевая группа управления из операции
* Другая группа управления в клиенте
* подписки в группе управления
* группы ресурсов в группе управления
* Клиент для группы ресурсов

[Ресурс расширения](scope-extension-resources.md) можно ограничить целевым объектом, который отличается от целевого объекта развертывания.

Пользователь, развертывающий шаблон, должен иметь доступ к указанной области.

В этом разделе показано, как указать различные области. Эти различные области можно объединить в один шаблон.

### <a name="scope-to-target-management-group"></a>Область действия для целевой группы управления

Ресурсы, определенные в разделе ресурсов шаблона, применяются к группе управления из команды развертывания.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/default-mg.json" highlight="5":::

### <a name="scope-to-another-management-group"></a>Область в другую группу управления

Чтобы выбрать другую группу управления, добавьте вложенное развертывание и укажите `scope` свойство. Задайте `scope` для свойства значение в формате `Microsoft.Management/managementGroups/<mg-name>` .

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/scope-mg.json" highlight="10,17,18,22":::

### <a name="scope-to-subscription"></a>Область действия для подписки

Кроме того, можно ориентироваться на подписки в группе управления. Пользователь, развертывающий шаблон, должен иметь доступ к указанной области.

Чтобы назначить подписку в группе управления, используйте вложенное развертывание и `subscriptionId` свойство.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/mg-to-subscription.json" highlight="9,10,18":::

### <a name="scope-to-resource-group"></a>Область действия для группы ресурсов

Вы также можете ориентироваться на группы ресурсов в группе управления. Пользователь, развертывающий шаблон, должен иметь доступ к указанной области.

Чтобы выбрать группу ресурсов в группе управления, используйте вложенное развертывание. Укажите свойства `subscriptionId` и `resourceGroup`. Не задавайте расположение для вложенного развертывания, так как оно развертывается в расположении группы ресурсов.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/mg-to-resource-group.json" highlight="9,10,18":::

Сведения об использовании развертывания группы управления для создания группы ресурсов в подписке и развертывании учетной записи хранения в этой группе ресурсов см. в статье [развертывание в подписке и группе ресурсов](#deploy-to-subscription-and-resource-group).

### <a name="scope-to-tenant"></a>Область для клиента

Чтобы создать ресурсы в клиенте, присвойте свойству значение `scope` `/` . Пользователь, развертывающий шаблон, должен иметь [необходимый доступ для развертывания в клиенте](deploy-to-tenant.md#required-access).

Чтобы использовать вложенное развертывание, задайте `scope` и `location` .

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/management-group-to-tenant.json" highlight="9,10,14":::

Или можно задать область `/` для некоторых типов ресурсов, например для групп управления. Создание новой группы управления описано в следующем разделе.

## <a name="management-group"></a>группа управления;

Чтобы создать группу управления в развертывании группы управления, необходимо задать для области значение `/` для группы управления.

В следующем примере создается новая группа управления в корневой группе управления.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/management-group-create-mg.json" highlight="12,15":::

В следующем примере создается новая группа управления в группе управления, заданной как родительская. Обратите внимание, что для области задано значение `/` .

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "mgName": {
            "type": "string",
            "defaultValue": "[concat('mg-', uniqueString(newGuid()))]"
        },
        "parentMG": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('mgName')]",
            "type": "Microsoft.Management/managementGroups",
            "apiVersion": "2020-05-01",
            "scope": "/",
            "location": "eastus",
            "properties": {
                "details": {
                    "parent": {
                        "id": "[tenantResourceId('Microsoft.Management/managementGroups', parameters('parentMG'))]"
                    }
                }
            }
        }
    ],
    "outputs": {
        "output": {
            "type": "string",
            "value": "[parameters('mgName')]"
        }
    }
}
```

## <a name="subscriptions"></a>Подписки

Чтобы создать новую подписку Azure в группе управления с помощью шаблона ARM, см. следующие статьи:

* [Программное создание подписок Azure Соглашение Enterprise](../../cost-management-billing/manage/programmatically-create-subscription-enterprise-agreement.md)
* [Программное создание подписок Azure для соглашения клиента Майкрософт](../../cost-management-billing/manage/programmatically-create-subscription-microsoft-customer-agreement.md)
* [Программное создание подписок Azure для партнерского соглашения Майкрософт](../../cost-management-billing/manage/programmatically-create-subscription-microsoft-partner-agreement.md)

Сведения о развертывании шаблона, который перемещает существующую подписку Azure в новую группу управления, см. в разделе [перемещение подписок в шаблоне ARM](../../governance/management-groups/manage.md#move-subscriptions-in-arm-template) .

## <a name="azure-policy"></a>Политика Azure

Определения настраиваемой политики, развернутые в группе управления, являются расширениями группы управления. Чтобы получить идентификатор определения пользовательской политики, используйте функцию [екстенсионресаурцеид ()](template-functions-resource.md#extensionresourceid) . Встроенные определения политик — это ресурсы уровня клиента. Чтобы получить идентификатор определения встроенной политики, используйте функцию [тенантресаурцеид ()](template-functions-resource.md#tenantresourceid) .

В следующем примере показано, как [определить](../../governance/policy/concepts/definition-structure.md) политику на уровне группы управления и назначить ее.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "targetMG": {
            "type": "string",
            "metadata": {
                "description": "Target Management Group"
            }
        },
        "allowedLocations": {
            "type": "array",
            "defaultValue": [
                "australiaeast",
                "australiasoutheast",
                "australiacentral"
            ],
            "metadata": {
                "description": "An array of the allowed locations, all other locations will be denied by the created policy."
            }
        }
    },
    "variables": {
        "mgScope": "[tenantResourceId('Microsoft.Management/managementGroups', parameters('targetMG'))]",
        "policyDefinition": "LocationRestriction"
    },
    "resources": [
        {
            "type": "Microsoft.Authorization/policyDefinitions",
            "name": "[variables('policyDefinition')]",
            "apiVersion": "2019-09-01",
            "properties": {
                "policyType": "Custom",
                "mode": "All",
                "parameters": {
                },
                "policyRule": {
                    "if": {
                        "not": {
                            "field": "location",
                            "in": "[parameters('allowedLocations')]"
                        }
                    },
                    "then": {
                        "effect": "deny"
                    }
                }
            }
        },
        {
            "type": "Microsoft.Authorization/policyAssignments",
            "name": "location-lock",
            "apiVersion": "2019-09-01",
            "dependsOn": [
                "[variables('policyDefinition')]"
            ],
            "properties": {
                "scope": "[variables('mgScope')]",
                "policyDefinitionId": "[extensionResourceId(variables('mgScope'), 'Microsoft.Authorization/policyDefinitions', variables('policyDefinition'))]"
            }
        }
    ]
}
```

## <a name="deploy-to-subscription-and-resource-group"></a>Развертывание в подписке и группе ресурсов

Из развертывания на уровне группы управления можно ориентироваться на подписку в группе управления. В следующем примере создается группа ресурсов в рамках подписки и выполняется развертывание учетной записи хранения в этой группе ресурсов.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "nestedsubId": {
            "type": "string"
        },
        "nestedRG": {
            "type": "string"
        },
        "storageAccountName": {
            "type": "string"
        },
        "nestedLocation": {
            "type": "string"
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2020-10-01",
            "name": "nestedSub",
            "location": "[parameters('nestedLocation')]",
            "subscriptionId": "[parameters('nestedSubId')]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {
                    },
                    "variables": {
                    },
                    "resources": [
                        {
                            "type": "Microsoft.Resources/resourceGroups",
                            "apiVersion": "2020-10-01",
                            "name": "[parameters('nestedRG')]",
                            "location": "[parameters('nestedLocation')]"
                        }
                    ]
                }
            }
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2020-10-01",
            "name": "nestedRG",
            "subscriptionId": "[parameters('nestedSubId')]",
            "resourceGroup": "[parameters('nestedRG')]",
            "dependsOn": [
                "nestedSub"
            ],
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "resources": [
                        {
                            "type": "Microsoft.Storage/storageAccounts",
                            "apiVersion": "2019-04-01",
                            "name": "[parameters('storageAccountName')]",
                            "location": "[parameters('nestedLocation')]",
                            "kind": "StorageV2",
                            "sku": {
                                "name": "Standard_LRS"
                            }
                        }
                    ]
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о назначении ролей см. в статье [Добавление назначений ролей Azure с помощью шаблонов Azure Resource Manager](../../role-based-access-control/role-assignments-template.md).
* Пример развертывания параметров рабочей области для центра безопасности Azure см. в разделе о [deployASCwithWorkspaceSettings.json](https://github.com/krnese/AzureDeploy/blob/master/ARM/deployments/deployASCwithWorkspaceSettings.json).
* Можно также развернуть шаблоны на уровне [подписки](deploy-to-subscription.md) и на [уровне клиента](deploy-to-tenant.md).
