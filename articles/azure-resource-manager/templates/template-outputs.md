---
title: Выходные данные в шаблонах
description: Описывает, как определить выходные значения в шаблоне Azure Resource Manager (шаблон ARM) и файле Бицеп.
ms.topic: conceptual
ms.date: 02/19/2021
ms.openlocfilehash: 2b6a6afa127bf43102103baadae576233843f00d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102123417"
---
# <a name="outputs-in-arm-templates"></a>Выходные данные в шаблонах ARM

В этой статье описывается, как определить выходные значения в шаблоне Azure Resource Manager (шаблон ARM) и файле Бицеп. Выходные данные используются при необходимости возврата значений из развернутых ресурсов.

Формат каждого выходного значения должен разрешаться в один из [типов данных](data-types.md).

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="define-output-values"></a>Определение выходных значений

В следующем примере показано, как вернуть свойство из развернутого ресурса.

# <a name="json"></a>[JSON](#tab/json)

Для JSON добавьте раздел в `outputs` шаблон. Выходное значение получает полное доменное имя для общедоступного IP-адреса.

```json
"outputs": {
  "hostname": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPAddressName'))).dnsSettings.fqdn]"
    },
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

Для Бицеп используйте `output` ключевое слово.

В следующем примере `publicIP` — это идентификатор общедоступного IP-адреса, развернутого в файле бицеп. Выходное значение получает полное доменное имя для общедоступного IP-адреса.

```bicep
output hostname string = publicIP.properties.dnsSettings.fqdn
```

---

Если необходимо выводить свойство, имеющее дефис в имени, используйте квадратные скобки вокруг имени вместо точечной нотации. Например, используйте  `['property-name']` вместо `.property-name` .

# <a name="json"></a>[JSON](#tab/json)

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "variables": {
        "user": {
            "user-name": "Test Person"
        }
    },
    "resources": [
    ],
    "outputs": {
        "nameResult": {
            "type": "string",
            "value": "[variables('user')['user-name']]"
        }
    }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
var user = {
  'user-name': 'Test Person'
}

output stringOutput string = user['user-name']
```

---

## <a name="conditional-output"></a>Условный вывод

Можно условно возвращать значение. Обычно условный вывод используется при [условии развертывания](conditional-resource-deployment.md) ресурса. В следующем примере показано, как условно вернуть идентификатор ресурса для общедоступного IP-адреса в зависимости от того, был ли развернут новый.

# <a name="json"></a>[JSON](#tab/json)

В JSON добавьте элемент, `condition` определяющий, возвращаются ли выходные данные.

```json
"outputs": {
  "resourceID": {
    "condition": "[equals(parameters('publicIpNewOrExisting'), 'new')]",
    "type": "string",
    "value": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIPAddresses_name'))]"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

Чтобы указать условный вывод в Бицеп, используйте `?` оператор. В следующем примере возвращается либо URL-адрес конечной точки, либо пустая строка в зависимости от условия.

```bicep
param deployStorage bool = true
param storageName string
param location string = resourceGroup().location

resource sa 'Microsoft.Storage/storageAccounts@2019-06-01' = if (deployStorage) {
  name: storageName
  location: location
  kind: 'StorageV2'
  sku:{
    name:'Standard_LRS'
    tier: 'Standard'
  }
  properties: {
    accessTier: 'Hot'
  }
}

output endpoint string = deployStorage ? sa.properties.primaryEndpoints.blob : ''
```

---

Простой пример условного вывода см. в разделе [шаблон условного вывода](https://github.com/bmoore-msft/AzureRM-Samples/blob/master/conditional-output/azuredeploy.json).

## <a name="dynamic-number-of-outputs"></a>Динамическое число выходов

В некоторых случаях неизвестно число экземпляров значения, которое необходимо вернуть при создании шаблона. Можно вернуть переменное число значений с помощью итерационного вывода.

# <a name="json"></a>[JSON](#tab/json)

В JSON добавьте элемент, `copy` чтобы выполнить итерацию выходных данных.

```json
"outputs": {
  "storageEndpoints": {
    "type": "array",
    "copy": {
      "count": "[parameters('storageCount')]",
      "input": "[reference(concat(copyIndex(), variables('baseName'))).primaryEndpoints.blob]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

Итеративные выходные данные в настоящее время недоступны для Бицеп.

---

Дополнительные сведения см. [в разделе итерация выходных данных в шаблонах ARM](copy-outputs.md).

## <a name="linked-templates"></a>Связанные шаблоны

В шаблонах JSON можно развертывать связанные шаблоны с помощью [связанных шаблонов](linked-templates.md). Чтобы получить выходное значение из связанного шаблона, используйте функцию [Reference](template-functions-resource.md#reference) в родительском шаблоне. В родительском шаблоне используется следующий синтаксис:

```json
"[reference('<deploymentName>').outputs.<propertyName>.value]"
```

В следующем примере показано, как задать IP-адрес в подсистеме балансировки нагрузки, извлекая значение из связанного шаблона.

```json
"publicIPAddress": {
  "id": "[reference('linkedTemplate').outputs.resourceID.value]"
}
```

Если имя свойства имеет дефис, вместо точечной нотации следует использовать квадратные скобки.

```json
"publicIPAddress": {
  "id": "[reference('linkedTemplate').outputs['resource-ID'].value]"
}
```

Вы не можете использовать функцию `reference` в разделе выходных данных [вложенного шаблона](linked-templates.md#nested-template). Чтобы извлечь значения для развернутого ресурса во вложенном шаблоне, преобразуйте этот шаблон в связанный.

[Шаблон общедоступного IP-адреса](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip.json) создает общедоступный IP-адрес и выводит идентификатор ресурса. [Шаблон подсистемы балансировки нагрузки](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json) ссылается на предыдущий шаблон. Он использует идентификатор ресурса в выходных данных при создании балансировщика нагрузки.

## <a name="modules"></a>Модули

В файлах Бицеп можно развертывать связанные шаблоны с помощью модулей. Чтобы получить выходное значение из модуля, используйте следующий синтаксис:

```bicep
<module-name>.outputs.<property-name>
```

В следующем примере показано, как задать IP-адрес в подсистеме балансировки нагрузки, извлекая значение из модуля. Имя модуля — `publicIP` .

```bicep
publicIPAddress: {
  id: publicIP.outputs.resourceID
}
```

## <a name="example-template"></a>Пример шаблона

Следующий шаблон не развертывает ресурсы. В нем показаны некоторые способы возврата выходов различных типов.

# <a name="json"></a>[JSON](#tab/json)

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/outputs.json":::

# <a name="bicep"></a>[Bicep](#tab/bicep)

Бицеп в настоящее время не поддерживает циклы.

:::code language="bicep" source="~/resourcemanager-templates/azure-resource-manager/outputs.bicep":::

---

## <a name="get-output-values"></a>Получение выходных значений

После завершения развертывания выходные значения автоматически возвращаются в результатах развертывания.

Чтобы получить выходные значения из журнала развертывания, можно использовать скрипт.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
(Get-AzResourceGroupDeployment `
  -ResourceGroupName <resource-group-name> `
  -Name <deployment-name>).Outputs.resourceID.value
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az deployment group show \
  -g <resource-group-name> \
  -n <deployment-name> \
  --query properties.outputs.resourceID.value
```

---

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о доступных свойствах выходных данных см. в разделе Общие сведения о [структуре и синтаксисе шаблонов ARM](template-syntax.md).
