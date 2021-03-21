---
title: Развертывание нескольких экземпляров ресурсов
description: Используйте операцию копирования и массивы в шаблоне Azure Resource Manager (шаблон ARM), чтобы развернуть тип ресурса много раз.
ms.topic: conceptual
ms.date: 12/21/2020
ms.openlocfilehash: c9bcb22ec53129520fd9574d0eb58b1e5777531e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97724499"
---
# <a name="resource-iteration-in-arm-templates"></a>Итерация ресурсов в шаблонах ARM

В этой статье показано, как создать более одного экземпляра ресурса в шаблоне Azure Resource Manager (шаблон ARM). Добавив `copy` элемент в раздел ресурсов шаблона, можно динамически задать число развертываемых ресурсов. Кроме того, не нужно повторять синтаксис шаблона.

Также можно использовать `copy` со [свойствами](copy-properties.md), [переменными](copy-variables.md)и [выходными данными](copy-outputs.md).

Если вам нужно указать, развернут ли ресурс, см. описание [элемента condition](conditional-resource-deployment.md).

## <a name="syntax"></a>Синтаксис

`copy`Элемент имеет следующий общий формат:

```json
"copy": {
  "name": "<name-of-loop>",
  "count": <number-of-iterations>,
  "mode": "serial" <or> "parallel",
  "batchSize": <number-to-deploy-serially>
}
```

`name`Свойство — это любое значение, идентифицирующее цикл. `count`Свойство определяет количество итераций, которое требуется для типа ресурса.

Используйте `mode` Свойства и, `batchSize` чтобы указать, должны ли ресурсы развертываться параллельно или последовательно. Эти свойства описаны в разделе [последовательные или параллельные](#serial-or-parallel).

## <a name="copy-limits"></a>Ограничения копирования

Число не может превышать 800.

Число не может быть отрицательным числом. Если вы развертываете шаблон с помощью последней версии Azure CLI, PowerShell или REST API, он может быть равен нулю. В частности, необходимо использовать:

* Azure PowerShell **2,6** или более поздней версии
* Azure CLI **2.0.74** или более поздней версии
* REST API версии **2019-05-10** или более поздней
* [Связанные развертывания](linked-templates.md) должны использовать API версии **2019-05-10** или более поздней для типа ресурса развертывания.

Более ранние версии PowerShell, CLI и REST API не поддерживают нулевое значение для счетчика.

Будьте внимательны при [развертывании полного режима](deployment-modes.md) с помощью команды Copy. При повторном развертывании с полным режимом для группы ресурсов все ресурсы, не указанные в шаблоне после разрешения цикла копирования, удаляются.

## <a name="resource-iteration"></a>Итерация ресурса

В следующем примере создается число учетных записей хранения, указанное в `storageCount` параметре.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageCount": {
            "type": "int",
            "defaultValue": 2
        }
    },
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-04-01",
            "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
            "location": "[resourceGroup().location]",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {},
            "copy": {
                "name": "storagecopy",
                "count": "[parameters('storageCount')]"
            }
        }
    ],
    "outputs": {}
}
```

Обратите внимание, что имя каждого ресурса содержит функцию `copyIndex()`, которая возвращает текущую итерацию в цикле. Индекс `copyIndex()` отсчитывается от нуля. См. приведенный ниже пример.

```json
"name": "[concat('storage', copyIndex())]",
```

Он создает следующие имена:

* storage0;
* storage1;
* storage2.

Чтобы сместить значение индекса, можно передать нужное значение в функцию `copyIndex()`. Число итераций по-прежнему указывается в элементе Copy, но значение параметра `copyIndex` смещается на указанное значение. См. приведенный ниже пример.

```json
"name": "[concat('storage', copyIndex(1))]",
```

Он создает следующие имена:

* storage1;
* storage2;
* storage3.

Операция копирования удобна при работе с массивами, так как позволяет выполнить итерацию по каждому элементу в массиве. Используйте функцию `length` в массиве, чтобы указать число итераций, а также функцию `copyIndex` для получения текущего индекса в массиве.

В следующем примере создается одна учетная запись хранения для каждого имени, указанного в параметре.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "storageNames": {
          "type": "array",
          "defaultValue": [
            "contoso",
            "fabrikam",
            "coho"
          ]
      }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(parameters('storageNames')[copyIndex()], uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {},
      "copy": {
        "name": "storagecopy",
        "count": "[length(parameters('storageNames'))]"
      }
    }
  ],
  "outputs": {}
}
```

Если требуется возвращать значения из развернутых ресурсов, можно использовать [Copy в разделе Outputs](copy-outputs.md).

## <a name="serial-or-parallel"></a>Последовательный или параллельный

По умолчанию Resource Manager создает ресурсы параллельно. Оно не ограничивает количество ресурсов, развернутых параллельно, за исключением общего ограничения в 800 ресурсов в шаблоне. Порядок, в котором они создаются, не гарантируется.

Тем не менее можно настроить последовательное развертывание ресурсов. Например, при обновлении рабочей среды может потребоваться дифференцировать обновления, чтобы только определенное число элементов могло быть обновлено в любой момент времени. Чтобы последовательно развернуть несколько экземпляров ресурса, задайте параметру `mode` значение **serial**, а в качестве значения `batchSize` введите число экземпляров, которое необходимо развернуть за раз. В последовательном режиме Resource Manager создает зависимость от предыдущих экземпляров в цикле, поэтому он не начинает выполнение следующего пакета до завершения предыдущего.

Значение для параметра `batchSize` не может превышать значение для `count` в элементе Copy.

Например, чтобы последовательно развернуть две учетные записи хранения за раз, используйте следующую команду:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "copy": {
        "name": "storagecopy",
        "count": 4,
        "mode": "serial",
        "batchSize": 2
      },
      "properties": {}
    }
  ],
  "outputs": {}
}
```

`mode`Свойство также принимает значение **Parallel**, которое является значением по умолчанию.

## <a name="iteration-for-a-child-resource"></a>Итерация дочерних ресурсов

Нельзя включить дочерний ресурс в цикл копирования. Чтобы создать несколько экземпляров ресурса, которые определяются как вложенные в другой ресурс, необходимо создать его как ресурс верхнего уровня. Вы можете определить связь с родительским ресурсом с помощью свойств type и name.

Предположим, вы определяете набор данных как дочерний ресурс фабрики данных.

```json
"resources": [
{
  "type": "Microsoft.DataFactory/datafactories",
  "name": "exampleDataFactory",
  ...
  "resources": [
    {
      "type": "datasets",
      "name": "exampleDataSet",
      "dependsOn": [
        "exampleDataFactory"
      ],
      ...
    }
  ]
```

Чтобы создать несколько наборов данных, переместите его за пределы фабрики данных. Набор данных должен находиться на том же уровне, что и фабрика данных, но при этом он должен быть дочерним ресурсом фабрики данных. Вы можете сохранить связь между набором данных и фабрикой данных с помощью свойств type и name. Так как тип нельзя определить по положению в шаблоне, укажите полное имя типа в следующем формате: `{resource-provider-namespace}/{parent-resource-type}/{child-resource-type}`.

Чтобы установить связь между родительским и дочерним ресурсами, укажите имя набора данных, которое включает имя родительского ресурса. Используйте формат `{parent-resource-name}/{child-resource-name}`.

В следующем примере показано, как это сделать:

```json
"resources": [
{
  "type": "Microsoft.DataFactory/datafactories",
  "name": "exampleDataFactory",
  ...
},
{
  "type": "Microsoft.DataFactory/datafactories/datasets",
  "name": "[concat('exampleDataFactory', '/', 'exampleDataSet', copyIndex())]",
  "dependsOn": [
    "exampleDataFactory"
  ],
  "copy": {
    "name": "datasetcopy",
    "count": "3"
  },
  ...
}]
```

## <a name="example-templates"></a>Образцы шаблонов

Ниже приведены примеры распространенных сценариев для создания нескольких экземпляров ресурса или свойства.

|Шаблон  |Описание  |
|---------|---------|
|[Копирования хранилища](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copystorage.json) |Развертывает несколько учетных записей хранения с номером индекса в имени. |
|[Последовательное копирование хранилища](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/serialcopystorage.json) |Последовательно развертывает несколько учетных записей хранения. Имя содержит номер индекса. |
|[Копирования хранилища с массивом](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copystoragewitharray.json) |Развертывает несколько учетных записей хранения. Имя содержит значение из массива. |

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о настройке зависимостей для ресурсов, создаваемых в цикле копирования, см. [в разделе Определение порядка развертывания ресурсов в шаблонах ARM](define-resource-dependency.md).
* Чтобы пройти обучение, см. раздел [учебник. Создание нескольких экземпляров ресурсов с помощью шаблонов ARM](template-tutorial-create-multiple-instances.md).
* Сведения о модулях Microsoft Learn с описанием копирования ресурсов см. в статье [Управление сложными облачными развертываниями с помощью расширенных функций шаблонов Resource Manager](/learn/modules/manage-deployments-advanced-arm-template-features/).
* Другие способы использования элемента copy см. в следующих статьях:
  * [Итерация свойства в шаблонах ARM](copy-properties.md)
  * [Итерация переменных в шаблонах ARM](copy-variables.md)
  * [Выходная итерация в шаблонах ARM](copy-outputs.md)
* Сведения об использовании инструкции Copy с вложенными шаблонами см. [в разделе Использование копирования](linked-templates.md#using-copy).
