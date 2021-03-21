---
title: Условное развертывание с помощью шаблонов
description: Описание условного развертывания ресурса в шаблоне Azure Resource Manager (шаблон ARM).
ms.topic: conceptual
ms.date: 03/02/2021
ms.openlocfilehash: 409d258d7dfe3ed186e5cf97cc0dbe6dc149b849
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101741180"
---
# <a name="conditional-deployment-in-arm-templates"></a>Условное развертывание в шаблонах ARM

Иногда при необходимости можно развернуть ресурс в шаблоне Azure Resource Manager (шаблон ARM) или в файле Бицеп. Для шаблонов JSON используйте элемент, `condition` чтобы указать, развернут ли ресурс. Для Бицеп используйте `if` ключевое слово, чтобы указать, развернут ли ресурс. Значение условия разрешается в true или false. Если значение true, ресурс создан. Если значение false, ресурс не создан. Значение может применяться только ко всему ресурсу.

> [!NOTE]
> Условное развертывание не распространяется каскадом на [дочерние ресурсы](child-resource-name-type.md). Если требуется условно развернуть ресурс и его дочерние ресурсы, необходимо применить одно и то же условие к каждому типу ресурсов.

## <a name="deploy-condition"></a>Условие развертывания

Можно передать значение параметра, указывающее, развернут ли ресурс. В следующем примере выполняется условное развертывание зоны DNS.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "deployZone": {
      "type": "bool"
    }
  },
  "functions": [],
  "resources": [
    {
      "condition": "[parameters('deployZone')]",
      "type": "Microsoft.Network/dnsZones",
      "apiVersion": "2018-05-01",
      "name": "myZone",
      "location": "global"
    }
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param deployZone bool

resource dnsZone 'Microsoft.Network/dnszones@2018-05-01' = if (deployZone) {
  name: 'myZone'
  location: 'global'
}
```

---

Более сложный пример см. в статье [логический сервер SQL Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-logical-server).

## <a name="new-or-existing-resource"></a>Новый или существующий ресурс

Условное развертывание можно использовать для создания нового ресурса или использования существующего. В следующем примере показано, как либо развернуть новую учетную запись хранения, либо использовать существующую учетную запись хранения.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string"
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "newOrExisting": {
      "type": "string",
      "defaultValue": "new",
      "allowedValues": [
        "new",
        "existing"
      ]
    }
  },
  "functions": [],
  "resources": [
    {
      "condition": "[equals(parameters('newOrExisting'), 'new')]",
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
param storageAccountName string
param location string = resourceGroup().location

@allowed([
  'new'
  'existing'
])
param newOrExisting string = 'new'

resource sa 'Microsoft.Storage/storageAccounts@2019-06-01' = if (newOrExisting == 'new') {
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

Если для параметра `newOrExisting` задано значение **New**, условие принимает значение true. Учетная запись хранения будет развернута. Однако если параметр `newOrExisting` имеет значение **existing**, условие принимает значение false, а учетная запись хранения не развернута.

Полный пример шаблона с использованием элемента `condition` см. по адресу [VM with a new or existing Virtual Network, Storage, and Public IP](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-new-or-existing-conditions) (Виртуальная машина с новой или существующей виртуальной сетью, хранилищем и общедоступным IP-адресом).

## <a name="runtime-functions"></a>Функции среды выполнения

Если вы используете функцию [ссылки](template-functions-resource.md#reference) или [списка](template-functions-resource.md#list) с ресурсом, который условно развернут, функция вычисляется, даже если ресурс не развернут. Если функция ссылается на несуществующий ресурс, возникает ошибка.

Используйте функцию [If](template-functions-logical.md#if) , чтобы убедиться, что функция вычисляется только для условий при развертывании ресурса. См. [функцию if](template-functions-logical.md#if) для примера шаблона, который использует `if` и `reference` с условно развернутым ресурсом.

Ресурс задается [как зависимый](define-resource-dependency.md) от условного ресурса точно так же, как любой другой ресурс. Если условный ресурс не развернут, Azure Resource Manager автоматически удаляет его из необходимых зависимостей.

## <a name="complete-mode"></a>Полный режим

Если вы развертываете шаблон с [полным режимом](deployment-modes.md) , а ресурс не развернут, так как `condition` принимает значение false, результат зависит от того, какая REST API версия используется для развертывания шаблона. Если вы используете более раннюю версию, чем 2019-05-10, ресурс **не удаляется**. В 2019-05-10 или более поздней версии ресурс **удаляется**. Последние версии Azure PowerShell и Azure CLI удаляют ресурс, если условие имеет значение false.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о модуле Microsoft Learn, охватывающем условное развертывание, см. в статье [управление комплексными облачными развертываниями с помощью расширенных функций шаблонов ARM](/learn/modules/manage-deployments-advanced-arm-template-features/).
* Рекомендации по созданию шаблонов см. в статье рекомендации по использованию [шаблонов ARM](template-best-practices.md).
* Сведения о создании нескольких экземпляров ресурса см. [в статье итерация ресурсов в шаблонах ARM](copy-resources.md).
