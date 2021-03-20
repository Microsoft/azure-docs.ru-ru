---
title: Расположение ресурса шаблона
description: Описывает, как задать расположение ресурса в шаблоне Azure Resource Manager (шаблон ARM).
ms.topic: conceptual
ms.date: 09/04/2019
ms.custom: ''
ms.openlocfilehash: 84a818109e6681b8d0e18de4d2d7969310582818
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96922400"
---
# <a name="set-resource-location-in-arm-template"></a>Задание расположения ресурса в шаблоне ARM

При развертывании шаблона Azure Resource Manager (шаблон ARM) необходимо указать расположение для каждого ресурса. Расположение не должно совпадать с расположением группы ресурсов.

## <a name="get-available-locations"></a>Получить доступные расположения

Различные типы ресурсов поддерживаются в разных расположениях. Чтобы получить поддерживаемые расположения для типа ресурса, используйте Azure PowerShell или Azure CLI.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
((Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes `
  | Where-Object ResourceTypeName -eq batchAccounts).Locations
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az provider show \
  --namespace Microsoft.Batch \
  --query "resourceTypes[?resourceType=='batchAccounts'].locations | [0]" \
  --out table
```

---

## <a name="use-location-parameter"></a>Использовать параметр location

Чтобы обеспечить гибкость при развертывании шаблона, используйте параметр, чтобы указать расположение ресурсов. Задайте для параметра значение по умолчанию `resourceGroup().location` .

В следующем примере показана учетная запись хранения, которая развертывается в расположении, указанном как параметр:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
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
        "description": "Storage Account type"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    }
  },
  "variables": {
    "storageAccountName": "[concat('storage', uniquestring(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2018-07-01",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('storageAccountType')]"
      },
      "kind": "StorageV2",
      "properties": {}
    }
  ],
  "outputs": {
    "storageAccountName": {
      "type": "string",
      "value": "[variables('storageAccountName')]"
    }
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия

* Полный список функций шаблонов см. в разделе [функции шаблонов ARM](template-functions.md).
* Дополнительные сведения о файлах шаблонов см. [в разделе сведения о структуре и синтаксисе шаблонов ARM](template-syntax.md).
