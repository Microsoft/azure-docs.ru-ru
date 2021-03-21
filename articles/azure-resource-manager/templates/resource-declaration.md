---
title: Объявление ресурсов в шаблонах
description: Описание объявления ресурсов для развертывания в шаблоне Azure Resource Manager (шаблон ARM).
ms.topic: conceptual
ms.date: 03/02/2021
ms.openlocfilehash: 13f4a08162c40cbb36173627d88a4a8202a4ed26
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101746661"
---
# <a name="resource-declaration-in-arm-templates"></a>Объявление ресурса в шаблонах ARM

Чтобы развернуть ресурс с помощью шаблона Azure Resource Manager (шаблон ARM), добавьте объявление ресурса. Используйте `resources` массив для шаблона JSON или `resource` ключевое слово для бицеп.

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="set-resource-type-and-version"></a>Задание типа и версии ресурса

При добавлении ресурса в шаблон Начните с настройки типа ресурса и версии API. Эти значения определяют другие свойства, доступные для ресурса.

В следующем примере показано, как задать тип ресурса и версию API для учетной записи хранения. В примере не показано полное объявление ресурса.

# <a name="json"></a>[JSON](#tab/json)

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2019-06-01",
    ...
  }
]
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
resource sa 'Microsoft.Storage/storageAccounts@2019-06-01' = {
  ...
}
```

---

Для Бицеп задается символьное имя ресурса. В предыдущем примере символическое имя — `sa` . Можно использовать любое значение для символьного имени, но оно не может совпадать с другим ресурсом, параметром или переменной в шаблоне. Символьное имя не совпадает с именем ресурса. Символическое имя используется для простой ссылки на ресурс в других частях шаблона.

## <a name="set-resource-name"></a>Задать имя ресурса

У каждого ресурса есть имя. При задании имени ресурса Обратите внимание на [правила и ограничения для имен ресурсов](../management/resource-name-rules.md).

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "storageAccountName": {
    "type": "string",
    "minLength": 3,
    "maxLength": 24
  }
},
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2019-06-01",
    "name": "[parameters('storageAccountName')]",
    ...
  }
]
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
@minLength(3)
@maxLength(24)
param storageAccountName string

resource sa 'Microsoft.Storage/storageAccounts@2019-06-01' = {
  name: storageAccountName
  ...
}
```

---

## <a name="set-location"></a>Указание расположения

Для многих ресурсов требуется расположение. Определить, требуется ли ресурс в качестве расположения, можно с помощью IntelliSense или [справочника по шаблону](/azure/templates/). В следующем примере добавляется параметр location, используемый для учетной записи хранения.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "storageAccountName": {
    "type": "string",
    "minLength": 3,
    "maxLength": 24
  },
  "location": {
    "type": "string",
    "defaultValue": "[resourceGroup().location]"
  }
},
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2019-06-01",
    "name": "[parameters('storageAccountName')]",
    "location": "[parameters('location')]",
    ...
  }
]
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
@minLength(3)
@maxLength(24)
param storageAccountName string
param location string = resourceGroup().location

resource sa 'Microsoft.Storage/storageAccounts@2019-06-01' = {
  name: storageAccountName
  location: location
  ...
}
```

---

Дополнительные сведения см. [в разделе Задание расположения ресурса в шаблоне ARM](resource-location.md).

## <a name="set-tags"></a>Задать Теги

Теги можно применить к ресурсу во время развертывания. Теги помогают логически упорядочивать развернутые ресурсы. Примеры различных способов указания тегов см. в разделе [Теги шаблонов ARM](../management/tag-resources.md#arm-templates).

## <a name="set-resource-specific-properties"></a>Задание свойств для конкретного ресурса

Приведенные выше свойства являются общими для большинства типов ресурсов. После задания этих значений необходимо задать свойства, относящиеся к развертываемому типу ресурсов.

Используйте IntelliSense или [ссылку на шаблон](/azure/templates/) , чтобы определить, какие свойства доступны и какие из них являются обязательными. В следующем примере задаются оставшиеся свойства учетной записи хранения.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string",
      "minLength": 3,
      "maxLength": 24
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "functions": [],
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
      },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
@minLength(3)
@maxLength(24)
param storageAccountName string
param location string = resourceGroup().location

resource sa 'Microsoft.Storage/storageAccounts@2019-06-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
    tier: 'Standard'
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
  }
}
```

---

## <a name="next-steps"></a>Дальнейшие действия

* См. Дополнительные сведения о [условном развертывании в шаблонах ARM](conditional-resource-deployment.md).
* Сведения о настройке зависимостей ресурсов см. [в разделе Определение порядка развертывания ресурсов в шаблонах ARM](define-resource-dependency.md).
