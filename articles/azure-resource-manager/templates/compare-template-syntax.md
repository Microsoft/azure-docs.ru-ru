---
title: Синтаксис Compare для шаблонов Azure Resource Manager в JSON и Бицеп
description: Сравнение шаблонов Azure Resource Manager, разработанных с помощью JSON и Бицеп, и показано, как выполнять преобразование между языками.
ms.topic: conceptual
ms.date: 03/12/2021
ms.openlocfilehash: 225e52e9534a77a01502b762f043a4f34df19caa
ms.sourcegitcommit: afb9e9d0b0c7e37166b9d1de6b71cd0e2fb9abf5
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/14/2021
ms.locfileid: "103461800"
---
# <a name="comparing-json-and-bicep-for-templates"></a>Сравнение JSON и Бицеп для шаблонов

В этой статье сравнивается синтаксис Бицеп с синтаксисом JSON для шаблонов Azure Resource Manager (шаблоны ARM). В большинстве случаев Бицеп предоставляет синтаксис, который менее подробный, чем эквивалент в JSON.

Если вы знакомы с использованием JSON для разработки шаблонов ARM, используйте следующие примеры, чтобы узнать о эквивалентном синтаксисе для Бицеп.

## <a name="expressions"></a>Выражения

Чтобы создать выражение, сделайте следующее:

```bicep
func()
```

```json
"[func()]"
```

## <a name="parameters"></a>Параметры

Чтобы объявить параметр со значением по умолчанию:

```bicep
param demoParam string = 'Contoso'
```

```json
"parameters": {
  "demoParam": {
    "type": "string",
    "defaultValue": "Contoso"
  }
}
```

Чтобы получить значение параметра, сделайте следующее:

```bicep
demoParam
```

```json
[parameters('demoParam'))]
```

## <a name="variables"></a>Переменные

Для объявления переменной:

```bicep
var demoVar = 'example value'
```

```json
"variables": {
  "demoVar": "example value"
},
```

Чтобы получить значение переменной, сделайте следующее:

```bicep
demoVar
```

```json
[variables('demoVar'))]
```

## <a name="strings"></a>Строки

Объединение строк:

```bicep
'${namePrefix}-vm'
```

```json
[concat(parameters('namePrefix'), '-vm')]
```

## <a name="logical-operators"></a>Логические операторы

Для возврата логических **и**:

```bicep
isMonday && isNovember
```

```json
[and(parameter('isMonday'), parameter('isNovember'))]
```

Для условного задания значения:

```bicep
isMonday ? 'valueIfTrue' : 'valueIfFalse'
```

```json
[if(parameters('isMonday'), 'valueIfTrue', 'valueIfFalse')]
```

## <a name="deployment-scope"></a>Область развертывания

Чтобы задать целевую область развертывания, выполните следующие действия.

```bicep
targetScope = 'subscription'
```

```json
"$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#"
```

## <a name="resources"></a>Ресурсы

Чтобы объявить ресурс, выполните следующие действия.

```bicep
resource vm 'Microsoft.Compute/virtualMachines@2020-06-01' = {
  ...
}
```

```json
"resources": [ 
  { 
    "type": "Microsoft.Compute/virtualMachines", 
    "apiVersion": "2020-06-01", 
    ... 
  } 
]
```

Для условного развертывания ресурса:

```bicep
resource vm 'Microsoft.Compute/virtualMachines@2020-06-01' = if(deployVM) {
  ...
}
```

```json
"resources": [ 
  {
    "condition": "[parameters('deployVM')]",
    "type": "Microsoft.Compute/virtualMachines", 
    "apiVersion": "2020-06-01", 
    ... 
  } 
]
```

Чтобы задать свойство ресурса, сделайте следующее:

```bicep
sku: '2016-Datacenter'
```

```json
"sku": "2016-Datacenter",
```

Чтобы получить идентификатор ресурса в шаблоне, выполните следующие действия.

```bicep
nic1.id
```

```json
[resourceId('Microsoft.Network/networkInterfaces', variables('nic1Name'))]
```

## <a name="loops"></a>Циклы

Для прохода по элементам массива или счетчика:

```bicep
[for storageName in storageAccounts: {
  ...
}]
```

```json
"copy": {
  "name": "storagecopy",
  "count": "[length(parameters('storageAccounts'))]"
},
...
```

## <a name="resource-dependencies"></a>Зависимости ресурсов

Чтобы задать зависимость между ресурсами, сделайте следующее:

Для Бицеп следует либо использовать автоматическое обнаружение зависимостей, либо задать зависимость вручную.

```bicep
dependsOn: [ stg ]
```

```json
"dependsOn": ["[resourceId('Microsoft.Storage/storageAccounts', 'parameters('storageAccountName'))]"]
```

## <a name="reference-resources"></a>Справочные ресурсы

Чтобы получить свойство из ресурса в шаблоне, выполните следующие действия.

```bicep
diagsAccount.properties.primaryEndpoints.blob
```

```json
[reference(resourceId('Microsoft.Storage/storageAccounts', variables('diagStorageAccountName'))).primaryEndpoints.blob]
```

Чтобы получить свойство из существующего ресурса, который не развернут в шаблоне, выполните следующие действия.

```bicep
resource stg 'Microsoft.Storage/storageAccounts@2019-06-01' existing = {
  name: storageAccountName
}

// use later in template as often as needed
stg.properties.primaryEndpoints.blob
```

```json
// required every time the property is needed
"[reference(resourceId('Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), '2019-06-01').primaryEndpoints.blob]"
```

## <a name="outputs"></a>Выходные данные

Вывод свойства из ресурса в шаблоне:

```bicep
output hostname string = publicIP.properties.dnsSettings.fqdn
```

```json
"outputs": {
  "hostname": {
    "type": "string",
    "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPAddressName'))).dnsSettings.fqdn]"
  },
}
```

## <a name="code-reuse"></a>Повторное использование кода

Разделение решения на несколько файлов:

* Для Бицеп используйте [модули](bicep-tutorial-add-modules.md).
* Для JSON используйте [связанные шаблоны](linked-templates.md).

## <a name="recommendations"></a>Рекомендации

* По возможности избегайте использования функций [Reference](template-functions-resource.md#reference) и [ResourceId](template-functions-resource.md#resourceid) в файле бицеп. При ссылке на ресурс в том же развертывании Бицеп используйте вместо него идентификатор ресурса. Например, если вы развернули ресурс в файле Бицеп с `stg` идентификатором ресурса, используйте синтаксис, например или, `stg.id` `stg.properties.primaryEndpoints.blob` для получения значений свойств. С помощью идентификатора ресурса можно создать неявную зависимость между ресурсами. Не нужно явно задавать зависимость с помощью свойства dependsOn.
* Если ресурс не развернут в файле Бицеп, вы по-прежнему можете получить символическую ссылку на ресурс с помощью ключевого слова **existing** .
* Используйте постоянный регистр для идентификаторов. Если вы не знаете, какой тип регистра используется, попробуйте использовать прописные буквы в стиле Camel. Например, `param myCamelCasedParameter string`.
* Добавьте описание к параметру только в том случае, если описание предоставляет пользователям нужные сведения. `//`Для получения некоторых сведений можно использовать комментарии.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о Бицеп см. в разделе [учебник по бицеп](./bicep-tutorial-create-first-bicep.md).
* Дополнительные сведения о преобразовании шаблонов между языками см. в разделе [Преобразование шаблонов ARM между JSON и бицеп](bicep-decompile.md).
