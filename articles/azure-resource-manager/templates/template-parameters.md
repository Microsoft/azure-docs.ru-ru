---
title: Параметры в шаблонах
description: Описывает, как определить параметры в шаблоне Azure Resource Manager (шаблон ARM) и в файле Бицеп.
ms.topic: conceptual
ms.date: 03/03/2021
ms.openlocfilehash: a8608e4733960d7f5ba7e5f548c47f16a2b244bb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102123400"
---
# <a name="parameters-in-arm-templates"></a>Параметры в шаблонах ARM

В этой статье описывается определение и использование параметров в шаблоне Azure Resource Manager (шаблон ARM) и в файле Бицеп. Предоставляя различные значения параметров, можно повторно использовать шаблон для разных сред.

Диспетчер ресурсов разрешает значения параметров перед началом операций развертывания. Везде, где в шаблоне используется параметр, диспетчер ресурсов заменяет его разрешенным значением.

Для каждого параметра необходимо задать один из [типов данных](data-types.md).

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="minimal-declaration"></a>Минимальное объявление

Как минимум, каждому параметру требуется имя и тип. В Бицеп параметр не может иметь то же имя, что и переменная, ресурс, выход или другой параметр в той же области.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "demoString": {
    "type": "string"
  },
  "demoInt": {
    "type": "int"
  },
  "demoBool": {
    "type": "bool"
  },
  "demoObject": {
    "type": "object"
  },
  "demoArray": {
    "type": "array"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param demoString string
param demoInt int
param demoBool bool
param demoObject object
param demoArray array
```

---

## <a name="secure-parameters"></a>Параметры безопасности

Можно пометить параметры строки или объекта как безопасные. Значение безопасного параметра не сохраняется в журнале развертывания и не записывается в журнал.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "demoPassword": {
    "type": "secureString"
  },
  "demoSecretObject": {
    "type": "secureObject"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
@secure()
param demoPassword string

@secure()
param demoSecretObject object
```

---

## <a name="allowed-values"></a>Допустимые значения

Можно определить допустимые значения для параметра. Допустимые значения задаются в массиве. Развертывание завершается ошибкой во время проверки, если значение передается в качестве параметра, который не является одним из допустимых значений.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "demoEnum": {
    "type": "string",
    "allowedValues": [
      "one",
      "two"
    ]
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
@allowed([
  'one'
  'two'
])
param demoEnum string
```

---

## <a name="default-value"></a>Значение по умолчанию

Для параметра можно указать значение по умолчанию. Значение по умолчанию используется, если значение не указано во время развертывания.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "demoParam": {
    "type": "string",
    "defaultValue": "Contoso"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param demoParam string = 'Contoso'
```

---

Чтобы указать значение по умолчанию вместе с другими свойствами для параметра, используйте следующий синтаксис.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "demoParam": {
    "type": "string",
    "defaultValue": "Contoso",
    "allowedValues": [
      "Contoso",
      "Fabrikam"
    ]
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
@allowed([
  'Contoso'
  'Fabrikam'
])
param demoParam string = 'Contoso'
```

---

Выражения можно использовать со значением по умолчанию. Нельзя использовать функцию [Reference](template-functions-resource.md#reference) или любую из функций [List](template-functions-resource.md#list) в разделе Parameters. Эти функции получают состояние среды выполнения ресурса и не могут быть выполнены перед развертыванием при разрешении параметров.

Выражения не допускаются с другими свойствами параметров.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "location": {
    "type": "string",
    "defaultValue": "[resourceGroup().location]"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param location string = resourceGroup().location
```

---

Также для создания значения по умолчанию можно использовать значение другого параметра. Следующий шаблон конструирует имя плана узла из имени сайта.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "siteName": {
    "type": "string",
    "defaultValue": "[concat('site', uniqueString(resourceGroup().id))]"
  },
  "hostingPlanName": {
    "type": "string",
    "defaultValue": "[concat(parameters('siteName'),'-plan')]"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param siteName string = 'site${uniqueString(resourceGroup().id)}'
param hostingPlanName string = '${siteName}-plan'
```

---

## <a name="length-constraints"></a>Ограничения длины

Для параметров String и Array можно указать минимальную и максимальную длину. Можно задать одно или оба ограничения. Для строк длина указывает количество символов. Для массивов длина указывает количество элементов в массиве.

В следующем примере объявляются два параметра. Один из параметров — имя учетной записи хранения, которая должна содержать 3-24 символов. Другой параметр — это массив, который должен содержать от 1-5 элементов.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "storageAccountName": {
    "type": "string",
    "minLength": 3,
    "maxLength": 24
  },
  "appNames": {
    "type": "array",
    "minLength": 1,
    "maxLength": 5
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
@minLength(3)
@maxLength(24)
param storageAccountName string

@minLength(1)
@maxLength(5)
param appNames array
```

---

## <a name="integer-constraints"></a>Целочисленные ограничения

Для целочисленных параметров можно задать минимальное и максимальное значения. Можно задать одно или оба ограничения.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "month": {
    "type": "int",
    "minValue": 1,
    "maxValue": 12
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
@minValue(1)
@maxValue(12)
param month int
```

---

## <a name="description"></a>Описание

Можно добавить описание параметра, чтобы помочь пользователям шаблона понять значение, которое необходимо предоставить. При развертывании шаблона через портал текст, который вы указываете в описании, автоматически используется в качестве подсказки для этого параметра. Добавлять описание можно только в том случае, если текст содержит больше информации, чем может быть выведено из имени параметра.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "virtualMachineSize": {
    "type": "string",
    "metadata": {
      "description": "Must be at least Standard_A3 to support 2 NICs."
    },
    "defaultValue": "Standard_DS1_v2"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
@description('Must be at least Standard_A3 to support 2 NICs.')
param virtualMachineSize string = 'Standard_DS1_v2'
```

---

## <a name="use-parameter"></a>Использовать параметр

В шаблоне JSON вы ссылаетесь на значение параметра с помощью функции [Parameters](template-functions-deployment.md#parameters) . В Бицеп используется имя параметра. В следующем примере используется значение параметра для Key Vault имени.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vaultName": {
      "type": "string",
      "defaultValue": "[format('keyVault{0}', uniqueString(resourceGroup().id))]"
    }
  },
  "resources": [
    {
      "type": "Microsoft.KeyVault/vaults",
      "apiVersion": "2019-09-01",
      "name": "[parameters('vaultName')]",
      ...
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param vaultName string = 'keyVault${uniqueString(resourceGroup().id)}'

resource keyvault 'Microsoft.KeyVault/vaults@2019-09-01' = {
  name: vaultName
  ...
}
```

---

## <a name="objects-as-parameters"></a>Объекты в качестве параметров

Чтобы структурировать связанные друг с другом значения, их можно передать в виде объекта. Что не менее важно, этот подход сокращает число параметров в шаблоне.

В следующем примере показан параметр, который является объектом. Значение по умолчанию показывает ожидаемые свойства объекта. Эти свойства используются при определении развертываемого ресурса.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vNetSettings": {
      "type": "object",
      "defaultValue": {
        "name": "VNet1",
        "location": "eastus",
        "addressPrefixes": [
          {
            "name": "firstPrefix",
            "addressPrefix": "10.0.0.0/22"
          }
        ],
        "subnets": [
          {
            "name": "firstSubnet",
            "addressPrefix": "10.0.0.0/24"
          },
          {
            "name": "secondSubnet",
            "addressPrefix": "10.0.1.0/24"
          }
        ]
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2020-06-01",
      "name": "[parameters('vNetSettings').name]",
      "location": "[parameters('vNetSettings').location]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": [
            "[parameters('vNetSettings').addressPrefixes[0].addressPrefix]"
          ]
        },
        "subnets": [
          {
            "name": "[parameters('vNetSettings').subnets[0].name]",
            "properties": {
              "addressPrefix": "[parameters('vNetSettings').subnets[0].addressPrefix]"
            }
          },
          {
            "name": "[parameters('vNetSettings').subnets[1].name]",
            "properties": {
              "addressPrefix": "[parameters('vNetSettings').subnets[1].addressPrefix]"
            }
          }
        ]
      }
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param vNetSettings object = {
  name: 'VNet1'
  location: 'eastus'
  addressPrefixes: [
    {
      name: 'firstPrefix'
      addressPrefix: '10.0.0.0/22'
    }
  ]
  subnets: [
    {
      name: 'firstSubnet'
      addressPrefix: '10.0.0.0/24'
    }
    {
      name: 'secondSubnet'
      addressPrefix: '10.0.1.0/24'
    }
  ]
}
resource vnet 'Microsoft.Network/virtualNetworks@2020-06-01' = {
  name: vNetSettings.name
  location: vNetSettings.location
  properties: {
    addressSpace: {
      addressPrefixes: [
        vNetSettings.addressPrefixes[0].addressPrefix
      ]
    }
    subnets: [
      {
        name: vNetSettings.subnets[0].name
        properties: {
          addressPrefix: vNetSettings.subnets[0].addressPrefix
        }
      }
      {
        name: vNetSettings.subnets[1].name
        properties: {
          addressPrefix: vNetSettings.subnets[1].addressPrefix
        }
      }
    ]
  }
}
```

---

## <a name="example-templates"></a>Образцы шаблонов

В следующих примерах демонстрируются сценарии использования параметров.

|Шаблон  |Описание  |
|---------|---------|
|[Использование параметров с функциями для определения значений по умолчанию](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/parameterswithfunctions.json) | Демонстрирует, как можно применить функции шаблонов при определении значений по умолчанию для параметров. Шаблон не развертывает ресурсы. Он только создает значения параметров и возвращает их. |
|[Объект Parameter](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/parameterobject.json) | Демонстрирует использование объекта в качестве параметра. Шаблон не развертывает ресурсы. Он только создает значения параметров и возвращает их. |

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о доступных свойствах параметров см. в разделе Общие сведения о [структуре и синтаксисе шаблонов ARM](template-syntax.md).
* Дополнительные сведения о передаче значений параметров в виде файла см. в разделе [Create диспетчер ресурсов Parameter File](parameter-files.md).
* Рекомендации по созданию параметров см. в разделе рекомендации [— Параметры](template-best-practices.md#parameters).
