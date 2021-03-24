---
title: Развертывание ресурсов в группах ресурсов
description: Описывает развертывание ресурсов в шаблоне Azure Resource Manager. В нем показано, как ориентироваться на несколько групп ресурсов.
ms.topic: conceptual
ms.date: 01/13/2021
ms.openlocfilehash: 1d636be9ffab5a4398e3e12867e601ce6df382bf
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104889796"
---
# <a name="resource-group-deployments-with-arm-templates"></a>Развертывание группы ресурсов с помощью шаблонов ARM

В этой статье описывается область развертывания для группы ресурсов. Для развертывания используется шаблон Azure Resource Manager (шаблон ARM). В статье также показано, как расширить область за пределами группы ресурсов в операции развертывания.

## <a name="supported-resources"></a>Поддерживаемые ресурсы

Большинство ресурсов можно развернуть в группе ресурсов. Список доступных ресурсов см. в [справочнике по шаблонам ARM](/azure/templates).

## <a name="schema"></a>схема

Для шаблонов используйте следующую схему:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    ...
}
```

Для файлов параметров используйте:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    ...
}
```

## <a name="deployment-commands"></a>Команды развертывания

Для развертывания в группе ресурсов используйте команды развертывания группы ресурсов.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Для Azure CLI используйте команду [AZ Deployment Group Create](/cli/azure/deployment/group#az_deployment_group_create). В следующем примере выполняется развертывание шаблона для создания группы ресурсов.

```azurecli-interactive
az deployment group create \
  --name demoRGDeployment \
  --resource-group ExampleGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json" \
  --parameters storageAccountType=Standard_GRS
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Для команды развертывания PowerShell используйте [New-азресаурцеграупдеплоймент](/powershell/module/az.resources/new-azresourcegroupdeployment). В следующем примере выполняется развертывание шаблона для создания группы ресурсов.

```azurepowershell-interactive
New-AzResourceGroupDeployment `
  -Name demoRGDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json `
  -storageAccountType Standard_GRS `
```

---

Более подробные сведения о командах и параметрах развертывания для развертывания шаблонов ARM см. в следующих статьях:

* [Развертывание ресурсов с помощью шаблонов ARM и портал Azure](deploy-portal.md)
* [Развертывание ресурсов с помощью шаблонов ARM и Azure CLI](deploy-cli.md)
* [Развертывание ресурсов с помощью шаблонов ARM и Azure PowerShell](deploy-powershell.md)
* [Развертывание ресурсов с помощью шаблонов ARM и Azure Resource Manager REST API](deploy-rest.md)
* [Использование кнопки развертывания для развертывания шаблонов из репозитория GitHub](deploy-to-azure-button.md)
* [Развертывание шаблонов ARM из Cloud Shell](deploy-cloud-shell.md)

## <a name="deployment-scopes"></a>Области развертывания

При развертывании в группе ресурсов можно развернуть ресурсы в:

* Целевая группа ресурсов из операции
* другие группы ресурсов в той же подписке или других подписках
* Любая подписка в клиенте
* Клиент для группы ресурсов

[Ресурс расширения](scope-extension-resources.md) можно ограничить целевым объектом, который отличается от целевого объекта развертывания.

Пользователь, развертывающий шаблон, должен иметь доступ к указанной области.

В этом разделе показано, как указать различные области. Эти различные области можно объединить в один шаблон.

### <a name="scope-to-target-resource-group"></a>Область для целевой группы ресурсов

Чтобы развернуть ресурсы в целевом ресурсе, добавьте эти ресурсы в раздел ресурсов шаблона.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/default-rg.json" highlight="5":::

Пример шаблона см. в разделе [развертывание в целевую группу ресурсов](#deploy-to-target-resource-group).

### <a name="scope-to-resource-group-in-same-subscription"></a>Область действия группы ресурсов в одной подписке

Чтобы развернуть ресурсы в другой группе ресурсов в той же подписке, добавьте вложенное развертывание и включите `resourceGroup` свойство. Если не указать идентификатор подписки или группу ресурсов, используются подписка и группа ресурсов из родительского шаблона. Все группы ресурсов необходимо создать до выполнения развертывания.

В следующем примере вложенное развертывание предназначено для группы ресурсов с именем `demoResourceGroup` .

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/same-sub-to-resource-group.json" highlight="9,13":::

Пример шаблона см. в разделе [развертывание в несколько групп ресурсов](#deploy-to-multiple-resource-groups).

### <a name="scope-to-resource-group-in-different-subscription"></a>Область действия группы ресурсов в другой подписке

Чтобы развернуть ресурсы в группе ресурсов в другой подписке, добавьте вложенное развертывание и включите `subscriptionId` `resourceGroup` Свойства и. В следующем примере вложенное развертывание предназначено для группы ресурсов с именем `demoResourceGroup` .

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/different-sub-to-resource-group.json" highlight="9,10,14":::

Пример шаблона см. в разделе [развертывание в несколько групп ресурсов](#deploy-to-multiple-resource-groups).

### <a name="scope-to-subscription"></a>Область действия для подписки

Чтобы развернуть ресурсы в подписке, добавьте вложенное развертывание и включите `subscriptionId` свойство. Подписка может быть подпиской для целевой группы ресурсов или любой другой подписки в клиенте. Кроме того, задайте `location` свойство для вложенного развертывания.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/resource-group-to-subscription.json" highlight="9,10,14":::

Пример шаблона см. в разделе [Создание группы ресурсов](#create-resource-group).

### <a name="scope-to-tenant"></a>Область для клиента

Чтобы создать ресурсы в клиенте, присвойте свойству значение `scope` `/` . Пользователь, развертывающий шаблон, должен иметь [необходимый доступ для развертывания в клиенте](deploy-to-tenant.md#required-access).

Чтобы использовать вложенное развертывание, задайте `scope` и `location` .

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/resource-group-to-tenant.json" highlight="9,10,14":::

Или можно задать область `/` для некоторых типов ресурсов, например для групп управления.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/resource-group-create-mg.json" highlight="12,15":::

Дополнительные сведения см. в разделе [Группа управления](deploy-to-management-group.md#management-group).

## <a name="deploy-to-target-resource-group"></a>Развертывание в целевую группу ресурсов

Чтобы развернуть ресурсы в Целевой группе ресурсов, определите эти ресурсы в `resources` разделе шаблона. Следующий шаблон создает учетную запись хранения в группе ресурсов, указанной в операции развертывания.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-outputs/azuredeploy.json":::

## <a name="deploy-to-multiple-resource-groups"></a>Развертывание в нескольких группах ресурсов

Можно выполнить развертывание в несколько групп ресурсов в одном шаблоне ARM. Чтобы выбрать группу ресурсов, которая отличается от указанной в родительском шаблоне, используйте [вложенный или связанный шаблон](linked-templates.md). В обозначении типа ресурса для развертывания укажите идентификатор подписки и группу ресурсов, в которых вы намерены развернуть этот вложенный шаблон. Эти группы ресурсов могут находиться в разных подписках.

> [!NOTE]
> В рамках одного развертывания можно выполнять развертывание **в 800 группах ресурсов**. Как правило, это ограничение означает, что для развертывания используется одна группа ресурсов из родительского шаблона и не более 799 групп ресурсов из вложенных или связанных развертываний. Но если родительский шаблон содержит только вложенные или связанные шаблоны и сам не развертывает ресурсы, то вложенные или связанные развертывания могут включать до 800 групп ресурсов.

Следующий пример создает две учетные записи хранения. Первая учетная запись хранения развертывается в той группе ресурсов, которая указана в операции развертывания. Вторая учетная запись хранения развертывается в группе ресурсов, которая указана в параметрах `secondResourceGroup` и `secondSubscriptionID`:

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/crosssubscription.json":::

Если в качестве значения параметра `resourceGroup` задать имя несуществующей группы ресурсов, развертывание завершится сбоем.

Чтобы протестировать предыдущий шаблон и просмотреть результаты, используйте PowerShell или Azure CLI.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы развернуть две учетные записи хранения в двух группах ресурсов в **одной подписке**, используйте следующий код:

```azurepowershell-interactive
$firstRG = "primarygroup"
$secondRG = "secondarygroup"

New-AzResourceGroup -Name $firstRG -Location southcentralus
New-AzResourceGroup -Name $secondRG -Location eastus

New-AzResourceGroupDeployment `
  -ResourceGroupName $firstRG `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json `
  -storagePrefix storage `
  -secondResourceGroup $secondRG `
  -secondStorageLocation eastus
```

Чтобы развернуть две учетные записи хранения в **двух подписках**, используйте следующий код:

```azurepowershell-interactive
$firstRG = "primarygroup"
$secondRG = "secondarygroup"

$firstSub = "<first-subscription-id>"
$secondSub = "<second-subscription-id>"

Set-AzContext -Subscription $secondSub
New-AzResourceGroup -Name $secondRG -Location eastus

Set-AzContext -Subscription $firstSub
New-AzResourceGroup -Name $firstRG -Location southcentralus

New-AzResourceGroupDeployment `
  -ResourceGroupName $firstRG `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json `
  -storagePrefix storage `
  -secondResourceGroup $secondRG `
  -secondStorageLocation eastus `
  -secondSubscriptionID $secondSub
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы развернуть две учетные записи хранения в двух группах ресурсов в **одной подписке**, используйте следующий код:

```azurecli-interactive
firstRG="primarygroup"
secondRG="secondarygroup"

az group create --name $firstRG --location southcentralus
az group create --name $secondRG --location eastus
az deployment group create \
  --name ExampleDeployment \
  --resource-group $firstRG \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json \
  --parameters storagePrefix=tfstorage secondResourceGroup=$secondRG secondStorageLocation=eastus
```

Чтобы развернуть две учетные записи хранения в **двух подписках**, используйте следующий код:

```azurecli-interactive
firstRG="primarygroup"
secondRG="secondarygroup"

firstSub="<first-subscription-id>"
secondSub="<second-subscription-id>"

az account set --subscription $secondSub
az group create --name $secondRG --location eastus

az account set --subscription $firstSub
az group create --name $firstRG --location southcentralus

az deployment group create \
  --name ExampleDeployment \
  --resource-group $firstRG \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json \
  --parameters storagePrefix=storage secondResourceGroup=$secondRG secondStorageLocation=eastus secondSubscriptionID=$secondSub
```

---

## <a name="create-resource-group"></a>Создать группу ресурсов

В развертывании группы ресурсов можно переключиться на уровень подписки и создать группу ресурсов. Следующий шаблон развертывает учетную запись хранения в целевую группу ресурсов и создает новую группу ресурсов в указанной подписке.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storagePrefix": {
            "type": "string",
            "maxLength": 11
        },
        "newResourceGroupName": {
            "type": "string"
        },
        "nestedSubscriptionID": {
            "type": "string"
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "variables": {
        "storageName": "[concat(parameters('storagePrefix'), uniqueString(resourceGroup().id))]"
    },
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "[variables('storageName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {
            }
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2020-06-01",
            "name": "demoSubDeployment",
            "location": "westus",
            "subscriptionId": "[parameters('nestedSubscriptionID')]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "type": "Microsoft.Resources/resourceGroups",
                            "apiVersion": "2020-10-01",
                            "name": "[parameters('newResourceGroupName')]",
                            "location": "[parameters('location')]",
                            "properties": {}
                        }
                    ],
                    "outputs": {}
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

* Пример развертывания параметров рабочей области для центра безопасности Azure см. в разделе о [deployASCwithWorkspaceSettings.json](https://github.com/krnese/AzureDeploy/blob/master/ARM/deployments/deployASCwithWorkspaceSettings.json).
