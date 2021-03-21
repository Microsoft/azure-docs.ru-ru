---
title: Функции шаблонов — числовые
description: Описывает функции, используемые в шаблоне Azure Resource Manager (шаблон ARM) для работы с числами.
ms.topic: conceptual
ms.date: 11/18/2020
ms.openlocfilehash: f3687581d94f80cc923614a0655da1813bd5c97b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97359716"
---
# <a name="numeric-functions-for-arm-templates"></a>Числовые функции для шаблонов ARM

Диспетчер ресурсов предоставляет следующие функции для работы с целыми числами в шаблоне Azure Resource Manager (шаблон ARM):

* [add](#add)
* [copyIndex](#copyindex)
* [div](#div)
* [float](#float)
* [int](#int)
* [max](#max)
* [min](#min)
* [mod (модуль)](#mod)
* [mul](#mul)
* [sub](#sub)

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="add"></a>add

`add(operand1, operand2)`

Возвращает сумму двух указанных целочисленных значений. `add`Функция не поддерживается в бицеп. `+`Вместо этого используйте оператор.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
|операнд1 |Да |INT |Первое слагаемое. |
|операнд2 |Да |INT |Второе слагаемое. |

### <a name="return-value"></a>Возвращаемое значение

Целое число, содержащее сумму параметров.

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/add.json) суммируются два параметра.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "first": {
      "type": "int",
      "defaultValue": 5,
      "metadata": {
        "description": "First integer to add"
      }
    },
    "second": {
      "type": "int",
      "defaultValue": 3,
      "metadata": {
        "description": "Second integer to add"
      }
    }
  },
  "resources": [
  ],
  "outputs": {
    "addResult": {
      "type": "int",
      "value": "[add(parameters('first'), parameters('second'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param first int = 5
param second int = 3

output addResult int = first + second
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| addResult | Int | 8 |

## <a name="copyindex"></a>copyIndex

`copyIndex(loopName, offset)`

Возвращает индекс цикла итерации.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| loopName | Нет | строка | Имя цикла для получения итерации. |
| offset |Нет |INT |Число, добавляемое к отсчитываемому от нуля значению итерации. |

### <a name="remarks"></a>Remarks

Эта функция всегда используется с объектом **copy**. Если значение **offset** не указано, возвращается текущее значение итерации. Значение итерации начинается с нуля.

Свойство **loopName** позволяет указать, на какую итерацию ссылается copyIndex: ресурса или свойства. Если значение **loopName** не указано, используется текущая итерация типа ресурса. Укажите значение **loopName** при выполнении итерации по свойству.

Дополнительные сведения об использовании копирования см. в следующих статьях:

* [Итерация ресурсов в шаблонах ARM](copy-resources.md)
* [Итерация свойства в шаблонах ARM](copy-properties.md)
* [Итерация переменных в шаблонах ARM](copy-variables.md)
* [Выходная итерация в шаблонах ARM](copy-outputs.md)

### <a name="example"></a>Пример

В следующем примере представлен цикл копирования, в котором значение индекса включается в имя.

# <a name="json"></a>[JSON](#tab/json)

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

# <a name="bicep"></a>[Bicep](#tab/bicep)

> [!NOTE]
> Циклы и `copyIndex` еще не реализованы в бицеп.  См. раздел [Loops](https://github.com/Azure/bicep/blob/main/docs/spec/loops.md).

---

### <a name="return-value"></a>Возвращаемое значение

Целое число, представляющее текущей индекс итерации.

## <a name="div"></a>div

`div(operand1, operand2)`

Возвращает целую часть частного от деления двух указанных целочисленных значений. `div`Функция не поддерживается в бицеп. `/`Вместо этого используйте оператор.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| операнд1 |Да |INT |Делимое. |
| операнд2 |Да |INT |Делитель. Не может быть равно 0. |

### <a name="return-value"></a>Возвращаемое значение

Целое число, представляющее деление.

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/div.json) один параметр делится на другой.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "first": {
      "type": "int",
      "defaultValue": 8,
      "metadata": {
        "description": "Integer being divided"
      }
    },
    "second": {
      "type": "int",
      "defaultValue": 3,
      "metadata": {
        "description": "Integer used to divide"
      }
    }
  },
  "resources": [
  ],
  "outputs": {
    "divResult": {
      "type": "int",
      "value": "[div(parameters('first'), parameters('second'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param first int = 8
param second int = 3

output addResult int = first / second
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| divResult | Int | 2 |

## <a name="float"></a>FLOAT

`float(arg1)`

Преобразует значение в число с плавающей запятой. Эта функция используется только при передаче пользовательских параметров в приложение, такое как приложение логики. `float`Функция не поддерживается в бицеп.  См. раздел [поддержка числовых типов, отличных от 32-разрядных целых чисел](https://github.com/Azure/bicep/issues/486).

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| arg1 |Да |строка или целое число |Значение, которое необходимо преобразовать в число с плавающей запятой. |

### <a name="return-value"></a>Возвращаемое значение

Число с плавающей запятой.

### <a name="example"></a>Пример

В следующем примере показано, как с помощью функции float передать параметры в приложение логики:

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "type": "Microsoft.Logic/workflows",
  "properties": {
    ...
    "parameters": {
      "custom1": {
        "value": "[float('3.0')]"
      },
      "custom2": {
        "value": "[float(3)]"
      },
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

> [!NOTE]
> `float`Функция не поддерживается в бицеп.  См. раздел [поддержка числовых типов, отличных от 32-разрядных целых чисел](https://github.com/Azure/bicep/issues/486).

---

## <a name="int"></a>INT

`int(valueToConvert)`

Преобразует указанное значение в целое число.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| valueToConvert |Да |строка или целое число |Значение, которое необходимо преобразовать в целое число. |

### <a name="return-value"></a>Возвращаемое значение

Целое число преобразованного значения.

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/int.json) указанное пользователем значение параметра преобразуется в целое число.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "stringToConvert": {
      "type": "string",
      "defaultValue": "4"
    }
  },
  "resources": [
  ],
  "outputs": {
    "intResult": {
      "type": "int",
      "value": "[int(parameters('stringToConvert'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param stringToConvert string = '4'

output inResult int = int(stringToConvert)
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| intResult | Int | 4 |

## <a name="max"></a>max

`max (arg1)`

Возвращает максимальное значение из массива целых чисел или разделенный запятыми список целых чисел.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| arg1 |Да |массив целых чисел или разделенный запятыми список целых чисел |Коллекция, для которой необходимо получить максимальное значение. |

### <a name="return-value"></a>Возвращаемое значение

Целое число, представляющее максимальное значение из коллекции.

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/max.json) показано, как использовать функцию max с массивом и списком целых чисел.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "arrayToTest": {
      "type": "array",
      "defaultValue": [ 0, 3, 2, 5, 4 ]
    }
  },
  "resources": [],
  "outputs": {
    "arrayOutput": {
      "type": "int",
      "value": "[max(parameters('arrayToTest'))]"
    },
    "intOutput": {
      "type": "int",
      "value": "[max(0,3,2,5,4)]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param arrayToTest array = [
  0
  3
  2
  5
  4
]

output arrayOutPut int = max(arrayToTest)
output intOutput int = max(0,3,2,5,4)
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| arrayOutput | Int | 5 |
| intOutput | Int | 5 |

## <a name="min"></a>мин

`min (arg1)`

Возвращает минимальное значение из массива целых чисел или разделенный запятыми список целых чисел.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| arg1 |Да |массив целых чисел или разделенный запятыми список целых чисел |Коллекция, для которой необходимо получить минимальное значение. |

### <a name="return-value"></a>Возвращаемое значение

Целое число, представляющее минимальное значение из коллекции.

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/min.json) показано, как использовать функцию min с массивом и списком целых чисел.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "arrayToTest": {
      "type": "array",
      "defaultValue": [ 0, 3, 2, 5, 4 ]
    }
  },
  "resources": [],
  "outputs": {
    "arrayOutput": {
      "type": "int",
      "value": "[min(parameters('arrayToTest'))]"
    },
    "intOutput": {
      "type": "int",
      "value": "[min(0,3,2,5,4)]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param arrayToTest array = [
  0
  3
  2
  5
  4
]

output arrayOutPut int = min(arrayToTest)
output intOutput int = min(0,3,2,5,4)
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| arrayOutput | Int | 0 |
| intOutput | Int | 0 |

## <a name="mod"></a>mod

`mod(operand1, operand2)`

Возвращает остаток от деления двух указанных целочисленных значений. `mod`Функция не поддерживается в бицеп. `%`Вместо этого используйте оператор.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| операнд1 |Да |INT |Делимое. |
| операнд2 |Да |INT |Число, используемое для деления, не может быть равно 0. |

### <a name="return-value"></a>Возвращаемое значение

Целое число, представляющее остаток.

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/mod.json) возвращается остаток от деления одного параметра на другой.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "first": {
      "type": "int",
      "defaultValue": 7,
      "metadata": {
        "description": "Integer being divided"
      }
    },
    "second": {
      "type": "int",
      "defaultValue": 3,
      "metadata": {
        "description": "Integer used to divide"
      }
    }
  },
  "resources": [
  ],
  "outputs": {
    "modResult": {
      "type": "int",
      "value": "[mod(parameters('first'), parameters('second'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param first int = 7
param second int = 3

output modResult int = first % second
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| modResult | Int | 1 |

## <a name="mul"></a>mul

`mul(operand1, operand2)`

Возвращает произведение двух указанных целочисленных значений. `mul`Функция не поддерживается в бицеп. `*`Вместо этого используйте оператор.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| операнд1 |Да |INT |Первый множитель. |
| операнд2 |Да |INT |Второй множитель. |

### <a name="return-value"></a>Возвращаемое значение

Целое число, представляющее произведение.

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/mul.json) один параметр умножается на другой.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "first": {
      "type": "int",
      "defaultValue": 5,
      "metadata": {
        "description": "First integer to multiply"
      }
    },
    "second": {
      "type": "int",
      "defaultValue": 3,
      "metadata": {
        "description": "Second integer to multiply"
      }
    }
  },
  "resources": [
  ],
  "outputs": {
    "mulResult": {
      "type": "int",
      "value": "[mul(parameters('first'), parameters('second'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param first int = 5
param second int = 3

output mulResult int = first * second
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| mulResult | Int | 15 |

## <a name="sub"></a>sub

`sub(operand1, operand2)`

Возвращает разность двух указанных целочисленных значений. `sub`Функция не поддерживается в бицеп. `-`Вместо этого используйте оператор.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| операнд1 |Да |INT |Уменьшаемое. |
| операнд2 |Да |INT |Вычитаемое. |

### <a name="return-value"></a>Возвращаемое значение

Целое число, представляющее разность.

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/sub.json) один параметр вычитается из другого.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "first": {
      "type": "int",
      "defaultValue": 7,
      "metadata": {
        "description": "Integer subtracted from"
      }
    },
    "second": {
      "type": "int",
      "defaultValue": 3,
      "metadata": {
        "description": "Integer to subtract"
      }
    }
  },
  "resources": [
  ],
  "outputs": {
    "subResult": {
      "type": "int",
      "value": "[sub(parameters('first'), parameters('second'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param first int = 7
param second int = 3

output subResult int = first - second
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| subResult | Int | 4 |

## <a name="next-steps"></a>Дальнейшие действия

* Описание разделов в шаблоне ARM см. [в разделе Общие сведения о структуре и синтаксисе шаблонов ARM](template-syntax.md).
* Сведения о итерации указанного числа раз при создании типа ресурса см. в разделе [Итерация ресурсов в шаблонах ARM](copy-resources.md).
