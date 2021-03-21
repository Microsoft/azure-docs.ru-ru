---
title: Определение нескольких экземпляров переменной
description: Используйте операцию копирования в шаблоне Azure Resource Manager (шаблон ARM) для многократного прохода при создании переменной.
ms.topic: conceptual
ms.date: 02/13/2020
ms.openlocfilehash: b8acd85659b843cb482e1ccc61e28da03431db1b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96905899"
---
# <a name="variable-iteration-in-arm-templates"></a>Итерация переменных в шаблонах ARM

В этой статье показано, как создать более одного значения для переменной в шаблоне Azure Resource Manager (шаблон ARM). Добавив `copy` элемент в раздел Variables шаблона, можно динамически задать число элементов для переменной во время развертывания. Кроме того, не нужно повторять синтаксис шаблона.

Можно также использовать Copy с [ресурсами](copy-resources.md), [свойствами в ресурсе](copy-properties.md)и [выходами](copy-outputs.md).

## <a name="syntax"></a>Синтаксис

Элемент Copy имеет следующий общий формат:

```json
"copy": [
  {
    "name": "<name-of-loop>",
    "count": <number-of-iterations>,
    "input": <values-for-the-variable>
  }
]
```

`name`Свойство — это любое значение, идентифицирующее цикл. `count`Свойство определяет количество итераций, которое требуется для переменной.

`input`Свойство указывает свойства, которые необходимо повторить. Вы создадите массив элементов на основе значения в свойстве `input`. Это может быть одно свойство (например, строка) или объект с несколькими свойствами.

## <a name="copy-limits"></a>Ограничения копирования

Число не может превышать 800.

Число не может быть отрицательным числом. Если вы развертываете шаблон с помощью последней версии Azure CLI, PowerShell или REST API, он может быть равен нулю. В частности, необходимо использовать:

* Azure PowerShell **2,6** или более поздней версии
* Azure CLI **2.0.74** или более поздней версии
* REST API версии **2019-05-10** или более поздней
* [Связанные развертывания](linked-templates.md) должны использовать API версии **2019-05-10** или более поздней для типа ресурса развертывания.

Более ранние версии PowerShell, CLI и REST API не поддерживают нулевое значение для счетчика.

## <a name="variable-iteration"></a>Итерация переменной

В следующем примере показано, как создать массив строковых значений.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "itemCount": {
            "type": "int",
            "defaultValue": 5
        }
     },
    "variables": {
        "copy": [
            {
                "name": "stringArray",
                "count": "[parameters('itemCount')]",
                "input": "[concat('item', copyIndex('stringArray', 1))]"
            }
        ]
    },
    "resources": [],
    "outputs": {
        "arrayResult": {
            "type": "array",
            "value": "[variables('stringArray')]"
        }
    }
}
```

Предыдущий шаблон возвращает массив со следующими значениями:

```json
[
    "item1",
    "item2",
    "item3",
    "item4",
    "item5"
]
```

В следующем примере показано, как создать массив объектов с тремя свойствами: `name` , `diskSizeGB` и `diskIndex` .

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "itemCount": {
            "type": "int",
            "defaultValue": 5
        }
    },
    "variables": {
        "copy": [
            {
                "name": "objectArray",
                "count": "[parameters('itemCount')]",
                "input": {
                    "name": "[concat('myDataDisk', copyIndex('objectArray', 1))]",
                    "diskSizeGB": "1",
                    "diskIndex": "[copyIndex('objectArray')]"
                }
            }
        ]
    },
    "resources": [],
    "outputs": {
        "arrayResult": {
            "type": "array",
            "value": "[variables('objectArray')]"
        }
    }
}
```

В предыдущем примере возвращается массив со следующими значениями:

```json
[
    {
        "name": "myDataDisk1",
        "diskSizeGB": "1",
        "diskIndex": 0
    },
    {
        "name": "myDataDisk2",
        "diskSizeGB": "1",
        "diskIndex": 1
    },
    {
        "name": "myDataDisk3",
        "diskSizeGB": "1",
        "diskIndex": 2
    },
    {
        "name": "myDataDisk4",
        "diskSizeGB": "1",
        "diskIndex": 3
    },
    {
        "name": "myDataDisk5",
        "diskSizeGB": "1",
        "diskIndex": 4
    }
]
```

> [!NOTE]
> Переменная iteration поддерживает аргумент offset. Смещение должно быть после имени итерации, например `copyIndex('diskNames', 1)` . Если значение смещения не введено, то по умолчанию используется 0 для первого экземпляра.
>

Можно также использовать `copy` элемент внутри переменной. В следующем примере создается объект, имеющий массив в качестве одного из его значений.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "itemCount": {
            "type": "int",
            "defaultValue": 5
        }
    },
    "variables": {
        "topLevelObject": {
            "sampleProperty": "sampleValue",
            "copy": [
                {
                    "name": "disks",
                    "count": "[parameters('itemCount')]",
                    "input": {
                        "name": "[concat('myDataDisk', copyIndex('disks', 1))]",
                        "diskSizeGB": "1",
                        "diskIndex": "[copyIndex('disks')]"
                    }
                }
            ]
        }
    },
    "resources": [],
    "outputs": {
        "objectResult": {
            "type": "object",
            "value": "[variables('topLevelObject')]"
        }
    }
}
```

В предыдущем примере возвращается объект со следующими значениями:

```json
{
    "sampleProperty": "sampleValue",
    "disks": [
        {
            "name": "myDataDisk1",
            "diskSizeGB": "1",
            "diskIndex": 0
        },
        {
            "name": "myDataDisk2",
            "diskSizeGB": "1",
            "diskIndex": 1
        },
        {
            "name": "myDataDisk3",
            "diskSizeGB": "1",
            "diskIndex": 2
        },
        {
            "name": "myDataDisk4",
            "diskSizeGB": "1",
            "diskIndex": 3
        },
        {
            "name": "myDataDisk5",
            "diskSizeGB": "1",
            "diskIndex": 4
        }
    ]
}
```

В следующем примере показаны различные способы использования `copy` с переменными.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {
    "disk-array-on-object": {
      "copy": [
        {
          "name": "disks",
          "count": 5,
          "input": {
            "name": "[concat('myDataDisk', copyIndex('disks', 1))]",
            "diskSizeGB": "1",
            "diskIndex": "[copyIndex('disks')]"
          }
        },
        {
          "name": "diskNames",
          "count": 5,
          "input": "[concat('myDataDisk', copyIndex('diskNames', 1))]"
        }
      ]
    },
    "copy": [
      {
        "name": "top-level-object-array",
        "count": 5,
        "input": {
          "name": "[concat('myDataDisk', copyIndex('top-level-object-array', 1))]",
          "diskSizeGB": "1",
          "diskIndex": "[copyIndex('top-level-object-array')]"
        }
      },
      {
        "name": "top-level-string-array",
        "count": 5,
        "input": "[concat('myDataDisk', copyIndex('top-level-string-array', 1))]"
      },
      {
        "name": "top-level-integer-array",
        "count": 5,
        "input": "[copyIndex('top-level-integer-array')]"
      }
    ]
  },
  "resources": [],
  "outputs": {
    "exampleObject": {
      "value": "[variables('disk-array-on-object')]",
      "type": "object"
    },
    "exampleArrayOnObject": {
      "value": "[variables('disk-array-on-object').disks]",
      "type" : "array"
    },
    "exampleObjectArray": {
      "value": "[variables('top-level-object-array')]",
      "type" : "array"
    },
    "exampleStringArray": {
      "value": "[variables('top-level-string-array')]",
      "type" : "array"
    },
    "exampleIntegerArray": {
      "value": "[variables('top-level-integer-array')]",
      "type" : "array"
    }
  }
}
```

## <a name="example-templates"></a>Образцы шаблонов

В следующих примерах показаны распространенные сценарии для создания более одного значения для переменной.

|Шаблон  |Описание  |
|---------|---------|
|[Копирование переменных](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copyvariables.json) |Демонстрирует разные способы итерации переменных. |
|[Несколько правил безопасности](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.json) |Развертывает несколько правил безопасности в группу безопасности сети. Кроме того, этот шаблон создает правила безопасности на основе параметра. Чтобы узнать параметр, см. [файл параметров нескольких групп безопасности сети](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/multiplesecurityrules.parameters.json). |

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы пройти обучение, см. раздел [учебник. Создание нескольких экземпляров ресурсов с помощью шаблонов ARM](template-tutorial-create-multiple-instances.md).
* Другие способы использования элемента copy см. в следующих статьях:
  * [Итерация ресурсов в шаблонах ARM](copy-resources.md)
  * [Итерация свойства в шаблонах ARM](copy-properties.md)
  * [Выходная итерация в шаблонах ARM](copy-outputs.md)
* Сведения о разделах шаблона см. в разделе Общие сведения о [структуре и синтаксисе шаблонов ARM](template-syntax.md).
* Сведения о развертывании шаблона см. в статье [развертывание ресурсов с помощью шаблонов ARM и Azure PowerShell](deploy-powershell.md).
