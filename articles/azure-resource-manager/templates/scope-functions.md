---
title: Функции шаблонов в развертываниях с заданной областью
description: Описание разрешения функций шаблона в развертываниях с заданной областью. Областью может быть клиент, группы управления, подписки и группы ресурсов.
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: f128448380612bc9b8d9114226e8a3036feeead8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99492099"
---
# <a name="arm-template-functions-in-deployment-scopes"></a>Функции шаблонов ARM в областях развертывания

С помощью шаблонов Azure Resource Manager (шаблоны ARM) можно выполнять развертывание в группах ресурсов, подписках, группах управления или клиентах. Как правило, [функции шаблонов ARM](template-functions.md) работают одинаково для всех областей. В этой статье описаны различия, которые существуют для некоторых функций в зависимости от области.

## <a name="supported-functions"></a>Поддерживаемые функции

При развертывании в разных областях необходимо учитывать следующие моменты.

* Функция [resourceGroup ()](template-functions-resource.md#resourcegroup) **поддерживается** для развертываний групп ресурсов.
* Функция [Subscription ()](template-functions-resource.md#subscription) **поддерживается** для развертываний групп ресурсов и подписок.
* Функции [Reference ()](template-functions-resource.md#reference) и [List ()](template-functions-resource.md#list) **поддерживаются** для всех областей.
* Используйте [resourceId ()](template-functions-resource.md#resourceid) , чтобы получить идентификатор для ресурса, развернутого в группе ресурсов.

  ```json
  "subnet": {
    "id": "[resourceId(parameters('virtualNetworkResourceGroup'), 'Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworkName'), parameters('subnet1Name'))]"
  }
  ```

* Используйте функцию [субскриптионресаурцеид ()](template-functions-resource.md#subscriptionresourceid) , чтобы получить идентификатор для ресурса, развернутого в подписке.

  Например, чтобы получить идентификатор ресурса для определения политики, развернутой в подписке, используйте:

  ```json
  "roleDefinitionId": "[subscriptionResourceId('Microsoft.Authorization/roleDefinitions', 'acdd72a7-3385-48ef-bd42-f606fba81ae7')]"
  ```

* Используйте функцию [екстенсионресаурцеид ()](template-functions-resource.md#extensionresourceid) для ресурсов, которые реализуются как расширения группы управления. Определения настраиваемой политики, развернутые в группе управления, являются расширениями группы управления.

  Чтобы получить идентификатор ресурса для определения пользовательской политики на уровне группы управления, используйте:

  ```json
  "policyDefinitionId": "[extensionResourceId(variables('mgScope'), 'Microsoft.Authorization/policyDefinitions', parameters('policyDefinitionID'))]"
  ```

* Используйте функцию [тенантресаурцеид ()](template-functions-resource.md#tenantresourceid) , чтобы получить идентификатор для ресурса, развернутого в клиенте. Встроенные определения политик — это ресурсы уровня клиента. При назначении встроенной политики на уровне группы управления используйте функцию Тенантресаурцеид.

  Чтобы получить идентификатор ресурса для встроенного определения политики, используйте:

  ```json
  "policyDefinitionId": "[tenantResourceId('Microsoft.Authorization/policyDefinitions', parameters('policyDefinitionID'))]"
  ```

## <a name="function-resolution-in-scopes"></a>Разрешение функций в областях

При развертывании в нескольких областях функции [resourceGroup ()](template-functions-resource.md#resourcegroup) и [Subscription ()](template-functions-resource.md#subscription) разрешаются по-разному в зависимости от способа указания шаблона. При указании ссылки на внешний шаблон функции всегда разрешаются в области этого шаблона. Если шаблон вложен в родительский шаблон, свойство `expressionEvaluationOptions` позволяет указать, где будут разрешаться функции: в группе ресурсов и подписке родительского шаблона или вложенного шаблона. Задайте для свойства значение `inner`, чтобы использовать область вложенного шаблона. Задайте для свойства значение `outer`, чтобы использовать область родительского шаблона.

В следующей таблице указано, где будут разрешаться функции: в группе ресурсов и подписке родительского шаблона или вложенного шаблона.

| Тип шаблона | Область | Решение |
| ------------- | ----- | ---------- |
| вложенный        | внешний (по умолчанию) | родительская группа ресурсов |
| вложенный        | Внутреннее | вложенная группа ресурсов |
| связанный        | Недоступно   | вложенная группа ресурсов |

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/crossresourcegroupproperties.json) представлено следующее:

* вложенный шаблон с областью по умолчанию (внешней);
* вложенный шаблон с внутренней областью;
* связанный шаблон.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/crossresourcegroupproperties.json":::

Чтобы протестировать предыдущий шаблон и просмотреть результаты, используйте PowerShell или Azure CLI.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name parentGroup -Location southcentralus
New-AzResourceGroup -Name inlineGroup -Location southcentralus
New-AzResourceGroup -Name linkedGroup -Location southcentralus

New-AzResourceGroupDeployment `
  -ResourceGroupName parentGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crossresourcegroupproperties.json
```

Выходные данные из предыдущего примера:

```output
 Name             Type                       Value
 ===============  =========================  ==========
 parentRG         String                     Parent resource group is parentGroup
 defaultScopeRG   String                     Default scope resource group is parentGroup
 innerScopeRG     String                     Inner scope resource group is inlineGroup
 linkedRG         String                     Linked resource group is linkedgroup
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az group create --name parentGroup --location southcentralus
az group create --name inlineGroup --location southcentralus
az group create --name linkedGroup --location southcentralus

az deployment group create \
  --name ExampleDeployment \
  --resource-group parentGroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crossresourcegroupproperties.json
```

Выходные данные из предыдущего примера:

```output
"outputs": {
  "defaultScopeRG": {
    "type": "String",
    "value": "Default scope resource group is parentGroup"
  },
  "innerScopeRG": {
    "type": "String",
    "value": "Inner scope resource group is inlineGroup"
  },
  "linkedRG": {
    "type": "String",
    "value": "Linked resource group is linkedGroup"
  },
  "parentRG": {
    "type": "String",
    "value": "Parent resource group is parentGroup"
  }
},
```

---

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о том, как определить параметры в шаблоне, см. [в разделе Общие сведения о структуре и синтаксисе шаблонов ARM](template-syntax.md).
* Советы по устранению распространенных ошибок развертывания см. в разделе [Устранение распространенных ошибок развертывания в Azure с помощью Azure Resource Manager](common-deployment-errors.md).
* Сведения о развертывании шаблона, требующего маркер SAS, см. в статье [развертывание закрытого шаблона ARM с помощью маркера SAS](secure-template-with-sas-token.md).
