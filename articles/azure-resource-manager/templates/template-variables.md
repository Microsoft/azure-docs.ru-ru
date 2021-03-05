---
title: Переменные в шаблонах
description: Описывает, как определять переменные в шаблоне Azure Resource Manager (шаблон ARM) и файле Бицеп.
ms.topic: conceptual
ms.date: 02/19/2021
ms.openlocfilehash: 3ab14c9acfcc2d6c9edd23fb3bc4d876cd5ac756
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102123383"
---
# <a name="variables-in-arm-templates"></a>Переменные в шаблонах ARM

В этой статье описывается определение и использование переменных в шаблоне Azure Resource Manager (шаблон ARM) или в файле Бицеп. Для упрощения шаблона используются переменные. Вместо повторения сложных выражений во всем шаблоне определяется переменная, которая содержит сложное выражение. Затем вы используете эту переменную по мере необходимости во всем шаблоне.

Диспетчер ресурсов разрешает переменные перед началом операций развертывания. Когда в шаблоне используется переменная, Resource Manager заменяет ее разрешенным значением.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="define-variable"></a>Определение переменной

При определении переменной вы не указываете [тип данных](data-types.md) для переменной. Вместо этого укажите значение или выражение шаблона. Тип переменной выводится из разрешенного значения. В следующем примере переменная задается в виде строки.

# <a name="json"></a>[JSON](#tab/json)

```json
"variables": {
  "stringVar": "example value"
},
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
var stringVar = 'example value'
```

---

При построении переменной можно использовать значение из параметра или другой переменной.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "inputValue": {
    "defaultValue": "deployment parameter",
    "type": "string"
  }
},
"variables": {
  "stringVar": "myVariable",
  "concatToVar": "[concat(variables('stringVar'), '-addtovar') ]",
  "concatToParam": "[concat(parameters('inputValue'), '-addtoparam')]"
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param inputValue string = 'deployment parameter'

var stringVar = 'myVariable'
var concatToVar =  '${stringVar}-addtovar'
var concatToParam = '${inputValue}-addtoparam'
```

---

Для создания значения переменной можно использовать [функции шаблонов](template-functions.md) .

В следующем примере создается строковое значение для имени учетной записи хранения. Он использует несколько функций шаблона для получения значения параметра и объединяет его в уникальную строку.

# <a name="json"></a>[JSON](#tab/json)

```json
"variables": {
  "storageName": "[concat(toLower(parameters('storageNamePrefix')), uniqueString(resourceGroup().id))]"
},
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
var storageName = '${toLower(storageNamePrefix)}${uniqueString(resourceGroup().id)}'
```

---

В шаблонах JSON нельзя использовать функцию [Reference](template-functions-resource.md#reference) или любые функции [List](template-functions-resource.md#list) в объявлении переменной. Эти функции получают состояние среды выполнения ресурса и не могут быть выполнены до развертывания при разрешении переменных.

В файлах Бицеп функции Reference и List являются допустимыми при объявлении переменной.

## <a name="use-variable"></a>Использование переменной

В следующем примере показано, как использовать переменную для свойства ресурса.

# <a name="json"></a>[JSON](#tab/json)

В шаблоне JSON вы ссылаетесь на значение переменной с помощью функции [Variables](template-functions-deployment.md#variables) .

```json
"variables": {
  "storageName": "[concat(toLower(parameters('storageNamePrefix')), uniqueString(resourceGroup().id))]"
},
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageName')]",
    ...
  }
]
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

В файле Бицеп вы ссылаетесь на значение переменной, указав имя переменной.

```bicep
var storageName = '${toLower(storageNamePrefix)}${uniqueString(resourceGroup().id)}'

resource demoAccount 'Microsoft.Storage/storageAccounts@2019-06-01' = {
  name: storageName
```

---

## <a name="example-template"></a>Пример шаблона

Следующий шаблон не развертывает ресурсы. В нем показаны некоторые способы объявления переменных различных типов.

# <a name="json"></a>[JSON](#tab/json)

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/variables.json":::

# <a name="bicep"></a>[Bicep](#tab/bicep)

Бицеп в настоящее время не поддерживает циклы.

:::code language="bicep" source="~/resourcemanager-templates/azure-resource-manager/variables.bicep":::

---

## <a name="configuration-variables"></a>Переменные конфигурации

Можно определить переменные, которые содержат связанные значения для настройки среды. Переменная определяется как объект со значениями. В следующем примере показан объект, содержащий значения для двух сред — **Test** и **произ**. Передайте одно из этих значений во время развертывания.

# <a name="json"></a>[JSON](#tab/json)

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/variablesconfigurations.json":::

# <a name="bicep"></a>[Bicep](#tab/bicep)

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/variablesconfigurations.bicep":::

---

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о доступных свойствах переменных см. в разделе Общие сведения о [структуре и синтаксисе шаблонов ARM](template-syntax.md).
* Рекомендации по созданию переменных см. в разделе рекомендации [— переменные](template-best-practices.md#variables).
* Пример шаблона, который назначает правила безопасности для группы безопасности сети, см. в разделе [правила сетевой безопасности](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.json) и [файл параметров](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.parameters.json).
