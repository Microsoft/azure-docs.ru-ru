---
title: Определение нескольких экземпляров свойства
description: Используйте операцию копирования в шаблоне Azure Resource Manager (шаблон ARM) для многократного прохода при создании свойства ресурса.
ms.topic: conceptual
ms.date: 09/15/2020
ms.openlocfilehash: 958deba6152ffa3bcb1d2d79cd026c0cb2eebcbe
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102521667"
---
# <a name="property-iteration-in-arm-templates"></a>Итерация свойства в шаблонах ARM

В этой статье показано, как создать более одного экземпляра свойства в шаблоне Azure Resource Manager (шаблон ARM). Добавив `copy` элемент в раздел свойств ресурса в шаблоне, можно динамически задать число элементов для свойства во время развертывания. Кроме того, не нужно повторять синтаксис шаблона.

Можно использовать только `copy` для ресурсов верхнего уровня, даже при применении `copy` к свойству. Сведения о том, как изменить дочерний ресурс на ресурс верхнего уровня, см. в разделе [Итерация для дочернего ресурса](copy-resources.md#iteration-for-a-child-resource).

Можно также использовать Copy с [ресурсами](copy-resources.md), [переменными](copy-variables.md)и [выходными данными](copy-outputs.md).

## <a name="syntax"></a>Синтаксис

Элемент Copy имеет следующий общий формат:

```json
"copy": [
  {
    "name": "<name-of-loop>",
    "count": <number-of-iterations>,
    "input": <values-for-the-property>
  }
]
```

Для `name` Укажите имя свойства ресурса, которое необходимо создать.

`count`Свойство определяет количество итераций, которое требуется для свойства.

`input`Свойство указывает свойства, которые необходимо повторить. Вы создадите массив элементов на основе значения в свойстве `input`.

## <a name="copy-limits"></a>Ограничения копирования

Число не может превышать 800.

Число не может быть отрицательным числом. Если вы развертываете шаблон с помощью последней версии Azure CLI, PowerShell или REST API, он может быть равен нулю. В частности, необходимо использовать:

* Azure PowerShell **2,6** или более поздней версии
* Azure CLI **2.0.74** или более поздней версии
* REST API версии **2019-05-10** или более поздней
* [Связанные развертывания](linked-templates.md) должны использовать API версии **2019-05-10** или более поздней для типа ресурса развертывания.

Более ранние версии PowerShell, CLI и REST API не поддерживают нулевое значение для счетчика.

## <a name="property-iteration"></a>Итерация свойства

В следующем примере показано, как применить `copy` к `dataDisks` свойству на виртуальной машине:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "numberOfDataDisks": {
      "type": "int",
      "minValue": 0,
      "maxValue": 16,
      "defaultValue": 3,
      "metadata": {
        "description": "The number of dataDisks to create."
      }
    },
    ...
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2017-03-30",
      ...
      "properties": {
        "storageProfile": {
          ...
          "copy": [
            {
              "name": "dataDisks",
              "count": "[parameters('numberOfDataDisks')]",
              "input": {
                "diskSizeGB": 1023,
                "lun": "[copyIndex('dataDisks')]",
                "createOption": "Empty"
              }
            }
          ]
        }
      }
    }
  ]
}
```

Обратите внимание, что при использовании `copyIndex` в итерации свойства, необходимо указать имя итерации. Итерация свойства также поддерживает аргумент offset. Смещение должно быть после имени итерации, например `copyIndex('dataDisks', 1)` .

Диспетчер ресурсов разворачивает массив `copy` во время развертывания. Имя массива становится именем свойства. Входные значения становятся свойствами объекта. Развернутый шаблон выглядит так:

```json
{
  "name": "examplevm",
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2017-03-30",
  "properties": {
    "storageProfile": {
      "dataDisks": [
        {
          "lun": 0,
          "createOption": "Empty",
          "diskSizeGB": 1023
        },
        {
          "lun": 1,
          "createOption": "Empty",
          "diskSizeGB": 1023
        },
        {
          "lun": 2,
          "createOption": "Empty",
          "diskSizeGB": 1023
        }
      ],
      ...
```

Операция копирования удобна при работе с массивами, так как позволяет выполнить итерацию по каждому элементу в массиве. Используйте функцию `length` в массиве, чтобы указать число итераций, а также функцию `copyIndex` для получения текущего индекса в массиве.

В следующем примере шаблона создается группа отработки отказа для баз данных, передаваемых в качестве массива.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "primaryServerName": {
            "type": "string"
        },
        "secondaryServerName": {
            "type": "string"
        },
        "databaseNames": {
            "type": "array",
            "defaultValue": [
                "mydb1",
                "mydb2",
                "mydb3"
            ]
        }
    },
    "variables": {
        "failoverName": "[concat(parameters('primaryServerName'),'/', parameters('primaryServerName'),'failovergroups')]"
    },
    "resources": [
        {
            "type": "Microsoft.Sql/servers/failoverGroups",
            "apiVersion": "2015-05-01-preview",
            "name": "[variables('failoverName')]",
            "properties": {
                "readWriteEndpoint": {
                    "failoverPolicy": "Automatic",
                    "failoverWithDataLossGracePeriodMinutes": 60
                },
                "readOnlyEndpoint": {
                    "failoverPolicy": "Disabled"
                },
                "partnerServers": [
                    {
                        "id": "[resourceId('Microsoft.Sql/servers', parameters('secondaryServerName'))]"
                    }
                ],
                "copy": [
                    {
                        "name": "databases",
                        "count": "[length(parameters('databaseNames'))]",
                        "input": "[resourceId('Microsoft.Sql/servers/databases', parameters('primaryServerName'), parameters('databaseNames')[copyIndex('databases')])]"
                    }
                ]
            }
        }
    ],
    "outputs": {
    }
}
```

`copy`Элемент является массивом, поэтому можно указать более одного свойства для ресурса.

```json
{
  "type": "Microsoft.Network/loadBalancers",
  "apiVersion": "2017-10-01",
  "name": "exampleLB",
  "properties": {
    "copy": [
      {
        "name": "loadBalancingRules",
        "count": "[length(parameters('loadBalancingRules'))]",
        "input": {
          ...
        }
      },
      {
        "name": "probes",
        "count": "[length(parameters('loadBalancingRules'))]",
        "input": {
          ...
        }
      }
    ]
  }
}
```

Вы можете совместно использовать итерацию свойства и ресурса. Обращайтесь к итерации свойства по имени.

```json
{
  "type": "Microsoft.Network/virtualNetworks",
  "apiVersion": "2018-04-01",
  "name": "[concat(parameters('vnetname'), copyIndex())]",
  "copy":{
    "count": 2,
    "name": "vnetloop"
  },
  "location": "[resourceGroup().location]",
  "properties": {
    "addressSpace": {
      "addressPrefixes": [
        "[parameters('addressPrefix')]"
      ]
    },
    "copy": [
      {
        "name": "subnets",
        "count": 2,
        "input": {
          "name": "[concat('subnet-', copyIndex('subnets'))]",
          "properties": {
            "addressPrefix": "[variables('subnetAddressPrefix')[copyIndex('subnets')]]"
          }
        }
      }
    ]
  }
}
```

## <a name="example-templates"></a>Образцы шаблонов

В следующем примере показан типичный сценарий для создания более одного значения для свойства.

|Шаблон  |Описание  |
|---------|---------|
|[Развертывание виртуальной машины с переменным количеством дисков данных](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-windows-copy-datadisks) |Развертывает несколько дисков данных с виртуальной машиной. |

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы пройти обучение, см. раздел [учебник. Создание нескольких экземпляров ресурсов с помощью шаблонов ARM](template-tutorial-create-multiple-instances.md).
* Другие способы использования элемента copy см. в следующих статьях:
  * [Итерация ресурсов в шаблонах ARM](copy-resources.md)
  * [Итерация переменных в шаблонах ARM](copy-variables.md)
  * [Выходная итерация в шаблонах ARM](copy-outputs.md)
* Сведения о разделах шаблона см. в разделе Общие сведения о [структуре и синтаксисе шаблонов ARM](template-syntax.md).
* Сведения о развертывании шаблона см. в статье [развертывание ресурсов с помощью шаблонов ARM и Azure PowerShell](deploy-powershell.md).
