---
title: Развертывание ресурсов в подписке
description: В этой статье описывается создание группы ресурсов в шаблоне Azure Resource Manager. Здесь также показано, как развернуть ресурсы в области подписки Azure.
ms.topic: conceptual
ms.date: 01/13/2021
ms.openlocfilehash: f557a3a15da33b7394d22784bcd2c1c914ad6201
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104889439"
---
# <a name="subscription-deployments-with-arm-templates"></a>Развертывание подписок с помощью шаблонов ARM

Чтобы упростить управление ресурсами, можно использовать шаблон Azure Resource Manager (шаблон ARM) для развертывания ресурсов на уровне подписки Azure. Например, можно развернуть [политики](../../governance/policy/overview.md) и [Управление доступом на основе ролей Azure (Azure RBAC)](../../role-based-access-control/overview.md) в подписке, которая применяет их в рамках подписки. Вы также можете создавать группы ресурсов в подписке и развертывать ресурсы в группах ресурсов в подписке.

> [!NOTE]
> На уровне подписки можно развернуть до 800 различных групп ресурсов.

Чтобы развернуть шаблоны на уровне подписки, используйте Azure CLI, PowerShell, REST API или портал.

## <a name="supported-resources"></a>Поддерживаемые ресурсы

Не все типы ресурсов можно развернуть на уровне подписки. В этом разделе перечислены поддерживаемые типы ресурсов.

Для чертежей Azure используйте:

* [artifacts](/azure/templates/microsoft.blueprint/blueprints/artifacts)
* [blueprints](/azure/templates/microsoft.blueprint/blueprints);
* [блуепринтассигнментс](/azure/templates/microsoft.blueprint/blueprintassignments)
* [версии (проекты)](/azure/templates/microsoft.blueprint/blueprints/versions)

Для политик Azure используйте:

* [policyAssignments](/azure/templates/microsoft.authorization/policyassignments);
* [policyDefinitions](/azure/templates/microsoft.authorization/policydefinitions);
* [policySetDefinitions](/azure/templates/microsoft.authorization/policysetdefinitions);
* [remediations](/azure/templates/microsoft.policyinsights/remediations);

Для управления доступом на основе ролей Azure (Azure RBAC) используйте:

* [roleAssignments](/azure/templates/microsoft.authorization/roleassignments);
* [roleDefinitions](/azure/templates/microsoft.authorization/roledefinitions).

Для вложенных шаблонов, которые развертываются в группах ресурсов, используйте:

* [размещения](/azure/templates/microsoft.resources/deployments)

Для создания новых групп ресурсов используйте:

* [resourceGroups](/azure/templates/microsoft.resources/resourcegroups)

Для управления подпиской используйте:

* [Конфигурации Advisor](/azure/templates/microsoft.advisor/configurations)
* [budgets](/azure/templates/microsoft.consumption/budgets);
* [Изменить профиль анализа](/azure/templates/microsoft.changeanalysis/profile)
* [supportPlanTypes](/azure/templates/microsoft.addons/supportproviders/supportplantypes);
* [теги](/azure/templates/microsoft.resources/tags)

К другим поддерживаемым типам относятся:

* [scopeAssignments](/azure/templates/microsoft.managednetwork/scopeassignments);
* [eventSubscriptions](/azure/templates/microsoft.eventgrid/eventsubscriptions);
* [peerAsns](/azure/templates/microsoft.peering/2019-09-01-preview/peerasns);

## <a name="schema"></a>схема

Схема, используемая для развертываний на уровне подписки, отличается от схемы развертываний группы ресурсов.

Для шаблонов используйте:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
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

Для развертывания в подписке используйте команды развертывания на уровне подписки.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Для Azure CLI используйте [az deployment sub create](/cli/azure/deployment/sub#az-deployment-sub-create). В следующем примере выполняется развертывание шаблона для создания группы ресурсов.

```azurecli-interactive
az deployment sub create \
  --name demoSubDeployment \
  --location centralus \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/emptyRG.json" \
  --parameters rgName=demoResourceGroup rgLocation=centralus
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Для команды развертывания PowerShell используйте [New-аздеплоймент](/powershell/module/az.resources/new-azdeployment) или его псевдоним `New-AzSubscriptionDeployment` . В следующем примере выполняется развертывание шаблона для создания группы ресурсов.

```azurepowershell-interactive
New-AzSubscriptionDeployment `
  -Name demoSubDeployment `
  -Location centralus `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/emptyRG.json" `
  -rgName demoResourceGroup `
  -rgLocation centralus
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

Для развертываний на уровне подписки необходимо указать расположение для развертывания. Расположение развертывания отделено от расположения развертываемых ресурсов. В расположении развертывания указывается место хранения данных развертывания. Для развертывания [группы управления](deploy-to-management-group.md) и [клиента](deploy-to-tenant.md) также требуется расположение. Для развертываний [группы](deploy-to-resource-group.md) ресурсов расположение группы ресурсов используется для хранения данных развертывания.

Можно указать имя развертывания или использовать имя развертывания по умолчанию. Имя по умолчанию — это имя файла шаблона. Например, развернув шаблон с именем _azuredeploy.json_ создается имя развертывания по умолчанию **azuredeploy**.

Для каждого имени развертывания расположение остается неизменным. Нельзя создать развертывание в одном расположении, если в другом уже есть развертывание с таким же именем. Например, если вы создаете развертывание подписки с именем **deployment1** в **centralus**, вы не сможете позднее создать другое развертывание с именем **deployment1** , а расположение **westus**. Если появится код ошибки `InvalidDeploymentLocation`, используйте другое имя или то же расположение, что и для предыдущего развертывания с этим именем.

## <a name="deployment-scopes"></a>Области развертывания

При развертывании в подписку можно развернуть ресурсы в:

* Целевая подписка из операции
* Любая подписка в клиенте
* группы ресурсов в подписке или других подписках
* Клиент для подписки

[Ресурс расширения](scope-extension-resources.md) можно ограничить целевым объектом, который отличается от целевого объекта развертывания.

Пользователь, развертывающий шаблон, должен иметь доступ к указанной области.

В этом разделе показано, как указать различные области. Эти различные области можно объединить в один шаблон.

### <a name="scope-to-target-subscription"></a>Область для назначения подписки

Чтобы развернуть ресурсы в целевой подписке, добавьте эти ресурсы в раздел ресурсов шаблона.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/default-sub.json" highlight="5":::

Примеры развертывания в подписке см. в разделе [Создание групп ресурсов](#create-resource-groups) и [назначение определения политики](#assign-policy-definition).

### <a name="scope-to-other-subscription"></a>Область в другую подписку

Чтобы развернуть ресурсы в подписке, которая отличается от операции подписки, добавьте вложенное развертывание. Задайте `subscriptionId` для свойства Идентификатор подписки, в которую требуется выполнить развертывание. Задайте `location` свойство для вложенного развертывания.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/sub-to-sub.json" highlight="9,10,14":::

### <a name="scope-to-resource-group"></a>Область действия для группы ресурсов

Чтобы развернуть ресурсы в группе ресурсов в подписке, добавьте вложенное развертывание и включите `resourceGroup` свойство. В следующем примере вложенное развертывание предназначено для группы ресурсов с именем `demoResourceGroup` .

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/sub-to-resource-group.json" highlight="9,13":::

Пример развертывания в группе ресурсов см. в разделе [Создание группы ресурсов и ресурсов](#create-resource-group-and-resources).

### <a name="scope-to-tenant"></a>Область для клиента

Чтобы создать ресурсы в клиенте, присвойте свойству значение `scope` `/` . Пользователь, развертывающий шаблон, должен иметь [необходимый доступ для развертывания в клиенте](deploy-to-tenant.md#required-access).

Чтобы использовать вложенное развертывание, задайте `scope` и `location` .

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/subscription-to-tenant.json" highlight="9,10,14":::

Или можно задать область `/` для некоторых типов ресурсов, например для групп управления.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/subscription-create-mg.json" highlight="12,15":::

Дополнительные сведения см. в разделе [Группа управления](deploy-to-management-group.md#management-group).

## <a name="resource-groups"></a>Группы ресурсов

### <a name="create-resource-groups"></a>Создание группы ресурсов

Чтобы создать группу ресурсов в шаблоне ARM, определите ресурс [Microsoft. Resources/resourceGroups](/azure/templates/microsoft.resources/allversions) с именем и расположением группы ресурсов.

В следующем примере создается пустая группа ресурсов.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgName": {
      "type": "string"
    },
    "rgLocation": {
      "type": "string"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2020-10-01",
      "name": "[parameters('rgName')]",
      "location": "[parameters('rgLocation')]",
      "properties": {}
    }
  ],
  "outputs": {}
}
```

Используйте [элемент copy](copy-resources.md) с группами ресурсов, чтобы создать несколько групп ресурсов.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgNamePrefix": {
      "type": "string"
    },
    "rgLocation": {
      "type": "string"
    },
    "instanceCount": {
      "type": "int"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2020-10-01",
      "location": "[parameters('rgLocation')]",
      "name": "[concat(parameters('rgNamePrefix'), copyIndex())]",
      "copy": {
        "name": "rgCopy",
        "count": "[parameters('instanceCount')]"
      },
      "properties": {}
    }
  ],
  "outputs": {}
}
```

Сведения о итерации ресурсов см. [в статье итерация ресурсов в шаблонах ARM](./copy-resources.md)и [учебник. Создание нескольких экземпляров ресурсов с помощью шаблонов ARM](./template-tutorial-create-multiple-instances.md).

### <a name="create-resource-group-and-resources"></a>Создание группы ресурсов и ресурсов

Чтобы создать группу ресурсов и развернуть в нее ресурсы, используйте вложенный шаблон. Вложенный шаблон определяет ресурсы для развертывания в группе ресурсов. Установите вложенный шаблон как зависимый от группы ресурсов, чтобы группа существовала до развертывания ресурсов. Можно выполнить развертывание до 800 групп ресурсов.

В следующем примере создается группа ресурсов и в нее развертывается учетная запись хранения.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgName": {
      "type": "string"
    },
    "rgLocation": {
      "type": "string"
    },
    "storagePrefix": {
      "type": "string",
      "maxLength": 11
    }
  },
  "variables": {
    "storageName": "[concat(parameters('storagePrefix'), uniqueString(subscription().id, parameters('rgName')))]"
  },
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2020-10-01",
      "name": "[parameters('rgName')]",
      "location": "[parameters('rgLocation')]",
      "properties": {}
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-10-01",
      "name": "storageDeployment",
      "resourceGroup": "[parameters('rgName')]",
      "dependsOn": [
        "[resourceId('Microsoft.Resources/resourceGroups/', parameters('rgName'))]"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
            {
              "type": "Microsoft.Storage/storageAccounts",
              "apiVersion": "2019-06-01",
              "name": "[variables('storageName')]",
              "location": "[parameters('rgLocation')]",
              "sku": {
                "name": "Standard_LRS"
              },
              "kind": "StorageV2"
            }
          ],
          "outputs": {}
        }
      }
    }
  ],
  "outputs": {}
}
```

## <a name="azure-policy"></a>Политика Azure

### <a name="assign-policy-definition"></a>Назначение определения политики

В следующем примере подписке присваивается имеющееся определение политики. Предоставьте параметры в качестве объекта, если определение политики принимает их. Используйте пустой объект по умолчанию, если определение политики не принимает параметры.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "policyDefinitionID": {
      "type": "string"
    },
    "policyName": {
      "type": "string"
    },
    "policyParameters": {
      "type": "object",
      "defaultValue": {}
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Authorization/policyAssignments",
      "apiVersion": "2018-03-01",
      "name": "[parameters('policyName')]",
      "properties": {
        "scope": "[subscription().id]",
        "policyDefinitionId": "[parameters('policyDefinitionID')]",
        "parameters": "[parameters('policyParameters')]"
      }
    }
  ]
}
```

Развернуть этот шаблон с помощью Azure CLI можно так:

```azurecli-interactive
# Built-in policy definition that accepts parameters
definition=$(az policy definition list --query "[?displayName=='Allowed locations'].id" --output tsv)

az deployment sub create \
  --name demoDeployment \
  --location centralus \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/policyassign.json" \
  --parameters policyDefinitionID=$definition policyName=setLocation policyParameters="{'listOfAllowedLocations': {'value': ['westus']} }"
```

Развернуть этот шаблон с помощью PowerShell можно так:

```azurepowershell-interactive
$definition = Get-AzPolicyDefinition | Where-Object { $_.Properties.DisplayName -eq 'Allowed locations' }

$locations = @("westus", "westus2")
$policyParams =@{listOfAllowedLocations = @{ value = $locations}}

New-AzSubscriptionDeployment `
  -Name policyassign `
  -Location centralus `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/policyassign.json" `
  -policyDefinitionID $definition.PolicyDefinitionId `
  -policyName setLocation `
  -policyParameters $policyParams
```

### <a name="create-and-assign-policy-definitions"></a>Создание и назначение определения политики

Вы можете [определить](../../governance/policy/concepts/definition-structure.md) и назначить определение политики в том же шаблоне.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Authorization/policyDefinitions",
      "apiVersion": "2018-05-01",
      "name": "locationpolicy",
      "properties": {
        "policyType": "Custom",
        "parameters": {},
        "policyRule": {
          "if": {
            "field": "location",
            "equals": "northeurope"
          },
          "then": {
            "effect": "deny"
          }
        }
      }
    },
    {
      "type": "Microsoft.Authorization/policyAssignments",
      "apiVersion": "2018-05-01",
      "name": "location-lock",
      "dependsOn": [
        "locationpolicy"
      ],
      "properties": {
        "scope": "[subscription().id]",
        "policyDefinitionId": "[subscriptionResourceId('Microsoft.Authorization/policyDefinitions', 'locationpolicy')]"
      }
    }
  ]
}
```

Чтобы создать определение политики и назначить ее в подписке, используйте следующую команду CLI.

```azurecli
az deployment sub create \
  --name demoDeployment \
  --location centralus \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/policydefineandassign.json"
```

Развернуть этот шаблон с помощью PowerShell можно так:

```azurepowershell
New-AzSubscriptionDeployment `
  -Name definePolicy `
  -Location centralus `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/policydefineandassign.json"
```

## <a name="azure-blueprints"></a>Azure Blueprints

### <a name="create-blueprint-definition"></a>Создание определения схемы

Можно [создать](../../governance/blueprints/tutorials/create-from-sample.md) определение схемы из шаблона.

:::code language="json" source="~/quickstart-templates/subscription-deployments/blueprints-new-blueprint/azuredeploy.json":::

Чтобы создать определение схемы в подписке, используйте следующую команду CLI.

```azurecli
az deployment sub create \
  --name demoDeployment \
  --location centralus \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/subscription-deployments/blueprints-new-blueprint/azuredeploy.json"
```

Развернуть этот шаблон с помощью PowerShell можно так:

```azurepowershell
New-AzSubscriptionDeployment `
  -Name demoDeployment `
  -Location centralus `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/subscription-deployments/blueprints-new-blueprint/azuredeploy.json"
```

## <a name="access-control"></a>Управление доступом

Дополнительные сведения о назначении ролей см. в статье [Добавление назначений ролей Azure с помощью шаблонов Azure Resource Manager](../../role-based-access-control/role-assignments-template.md).

Следующий пример создает группу ресурсов, применяет к ней блокировку и назначает роль участнику.

:::code language="json" source="~/quickstart-templates/subscription-deployments/create-rg-lock-role-assignment/azuredeploy.json":::

## <a name="next-steps"></a>Дальнейшие действия

* Пример развертывания параметров рабочей области для центра безопасности Azure см. в разделе о [deployASCwithWorkspaceSettings.json](https://github.com/krnese/AzureDeploy/blob/master/ARM/deployments/deployASCwithWorkspaceSettings.json).
* Примеры шаблонов можно найти на [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/subscription-deployments).
* Кроме того, можно развернуть шаблоны на [уровне группы управления](deploy-to-management-group.md) и на [уровне клиента](deploy-to-tenant.md).
