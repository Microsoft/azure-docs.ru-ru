---
title: Дочерние ресурсы в шаблонах
description: Описывает, как задать имя и тип для дочерних ресурсов в шаблоне Azure Resource Manager (шаблон ARM).
ms.topic: conceptual
ms.date: 12/21/2020
ms.openlocfilehash: a950d72751b829c0a2aa3ba5ca27316a0544d9cc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97963918"
---
# <a name="set-name-and-type-for-child-resources"></a>Задать имя и тип для дочерних ресурсов

Дочерние ресурсы — это ресурсы, которые существуют только в контексте другого ресурса. Например, [расширение виртуальной машины](/azure/templates/microsoft.compute/virtualmachines/extensions) не может существовать без [виртуальной машины](/azure/templates/microsoft.compute/virtualmachines). Ресурс расширения является дочерним для виртуальной машины.

Каждый родительский ресурс принимает в качестве дочерних ресурсов только определенные типы ресурсов. Тип ресурса для дочернего ресурса включает тип ресурса для родительского ресурса. Например, `Microsoft.Web/sites/config` и `Microsoft.Web/sites/extensions` являются дочерними ресурсами `Microsoft.Web/sites` . Принимаемые типы ресурсов указаны в [схеме шаблона](https://github.com/Azure/azure-resource-manager-schemas) родительского ресурса.

В шаблоне Azure Resource Manager (шаблон ARM) можно указать дочерний ресурс либо в родительском ресурсе, либо вне родительского ресурса. В следующем примере показан дочерний ресурс, входящий в свойство Resources родительского ресурса.

```json
"resources": [
  {
    <parent-resource>
    "resources": [
      <child-resource>
    ]
  }
]
```

Дочерние ресурсы можно определять максимум на пяти нижестоящих уровнях.

В следующем примере показан дочерний ресурс за пределами родительского ресурса. Этот подход можно использовать, если родительский ресурс не развертывается в том же шаблоне или если требуется использовать [Copy](copy-resources.md) для создания более чем одного дочернего ресурса.

```json
"resources": [
  {
    <parent-resource>
  },
  {
    <child-resource>
  }
]
```

Значения, указанные для имени ресурса и типа, зависят от того, определен ли дочерний ресурс внутри или за пределами родительского ресурса.

## <a name="within-parent-resource"></a>В родительском ресурсе

При определении в родительском типе ресурса значения типа и имени отформатированы как одно слово без косой черты.

```json
"type": "{child-resource-type}",
"name": "{child-resource-name}",
```

В следующем примере показана виртуальная сеть и подсеть. Обратите внимание, что подсеть включена в массив ресурсов для виртуальной сети. Для имени задано значение **Subnet1** , а для типа — **подсети**. Дочерний ресурс помечается как зависимый от родительского ресурса, так как родительский ресурс должен существовать до развертывания дочернего ресурса.

```json
"resources": [
  {
    "type": "Microsoft.Network/virtualNetworks",
    "apiVersion": "2018-10-01",
    "name": "VNet1",
    "location": "[parameters('location')]",
    "properties": {
      "addressSpace": {
        "addressPrefixes": [
          "10.0.0.0/16"
        ]
      }
    },
    "resources": [
      {
        "type": "subnets",
        "apiVersion": "2018-10-01",
        "name": "Subnet1",
        "location": "[parameters('location')]",
        "dependsOn": [
          "VNet1"
        ],
        "properties": {
          "addressPrefix": "10.0.0.0/24"
        }
      }
    ]
  }
]
```

Полный тип ресурса по-прежнему `Microsoft.Network/virtualNetworks/subnets` . `Microsoft.Network/virtualNetworks/` не указывается, так как предполагается из типа родительского ресурса.

Имя дочернего ресурса имеет значение **Subnet1** , а полное имя включает имя родительского объекта. Вы не предоставляете **VNet1** , так как он предполагается из родительского ресурса.

## <a name="outside-parent-resource"></a>Внешний родительский ресурс

При определении за пределами родительского ресурса необходимо отформатировать тип и с помощью косой черты включить родительский тип и имя.

```json
"type": "{resource-provider-namespace}/{parent-resource-type}/{child-resource-type}",
"name": "{parent-resource-name}/{child-resource-name}",
```

В следующем примере показана виртуальная сеть и подсеть, которые определены на корневом уровне. Обратите внимание, что подсеть не входит в массив ресурсов для виртуальной сети. Для имени задано значение **VNet1/Subnet1** , а для типа — значение `Microsoft.Network/virtualNetworks/subnets` . Дочерний ресурс помечается как зависимый от родительского ресурса, так как родительский ресурс должен существовать до развертывания дочернего ресурса.

```json
"resources": [
  {
    "type": "Microsoft.Network/virtualNetworks",
    "apiVersion": "2018-10-01",
    "name": "VNet1",
    "location": "[parameters('location')]",
    "properties": {
      "addressSpace": {
        "addressPrefixes": [
          "10.0.0.0/16"
        ]
      }
    }
  },
  {
    "type": "Microsoft.Network/virtualNetworks/subnets",
    "apiVersion": "2018-10-01",
    "location": "[parameters('location')]",
    "name": "VNet1/Subnet1",
    "dependsOn": [
      "VNet1"
    ],
    "properties": {
      "addressPrefix": "10.0.0.0/24"
    }
  }
]
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о создании шаблонов ARM см. [в разделе Общие сведения о структуре и синтаксисе шаблонов ARM](template-syntax.md).
* Дополнительные сведения о формате имени ресурса при ссылке на ресурс см. в описании [функции Reference](template-functions-resource.md#reference).
