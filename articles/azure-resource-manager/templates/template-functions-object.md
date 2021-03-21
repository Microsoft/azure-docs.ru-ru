---
title: Функции шаблонов — объекты
description: Описывает функции, используемые в шаблоне Azure Resource Manager (шаблон ARM) для работы с объектами.
ms.topic: conceptual
ms.date: 11/18/2020
ms.openlocfilehash: 5e13177db1a7cf2f19a822363cb3884474566add
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96920467"
---
# <a name="object-functions-for-arm-templates"></a>Функции объектов для шаблонов ARM

Диспетчер ресурсов предоставляет несколько функций для работы с объектами в шаблоне Azure Resource Manager (шаблон ARM):

* [contains](#contains)
* [createObject](#createobject)
* [empty](#empty)
* [intersection](#intersection)
* [json](#json)
* [length](#length)
* [null](#null)
* [union](#union)

[!INCLUDE [Bicep preview](../../../includes/resource-manager-bicep-preview.md)]

## <a name="contains"></a>содержит

`contains(container, itemToFind)`

Проверяет, содержит ли массив значение, содержит ли объект ключ или содержит ли строка подстроку. При сравнении строк учитывается регистр. Тем не менее при проверке объекта на наличие ключа сравнение выполняется без учета регистра.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| контейнер |Да |массив, объект или строка |Значение, содержащее значение, которое необходимо найти. |
| itemToFind |Да |строка или целое число |Значение, которое необходимо найти. |

### <a name="return-value"></a>Возвращаемое значение

**True**, если элемент найден. В противном случае — **False**.

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/contains.json) показано, как использовать функцию contains с различными типами.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "stringToTest": {
      "type": "string",
      "defaultValue": "OneTwoThree"
    },
    "objectToTest": {
      "type": "object",
      "defaultValue": {
        "one": "a",
        "two": "b",
        "three": "c"
      }
    },
    "arrayToTest": {
      "type": "array",
      "defaultValue": [ "one", "two", "three" ]
    }
  },
  "resources": [
  ],
  "outputs": {
    "stringTrue": {
      "type": "bool",
      "value": "[contains(parameters('stringToTest'), 'e')]"
    },
    "stringFalse": {
      "type": "bool",
      "value": "[contains(parameters('stringToTest'), 'z')]"
    },
    "objectTrue": {
      "type": "bool",
      "value": "[contains(parameters('objectToTest'), 'one')]"
    },
    "objectFalse": {
      "type": "bool",
      "value": "[contains(parameters('objectToTest'), 'a')]"
    },
    "arrayTrue": {
      "type": "bool",
      "value": "[contains(parameters('arrayToTest'), 'three')]"
    },
    "arrayFalse": {
      "type": "bool",
      "value": "[contains(parameters('arrayToTest'), 'four')]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param stringToTest string = 'OneTwoThree'
param objectToTest object = {
  'one': 'a'
  'two': 'b'
  'three': 'c'
}
param arrayToTest array = [
  'one'
  'two'
  'three'
]

output stringTrue bool = contains(stringToTest, 'e')
output stringFalse bool = contains(stringToTest, 'z')
output objectTrue bool = contains(objectToTest, 'one')
output objectFalse bool = contains(objectToTest, 'a')
output arrayTrue bool = contains(arrayToTest, 'three')
output arrayFalse bool = contains(arrayToTest, 'four')
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| stringTrue | Bool | True |
| stringFalse | Bool | Неверно |
| objectTrue | Bool | True |
| objectFalse | Bool | Неверно |
| arrayTrue | Bool | True |
| arrayFalse | Bool | Неверно |

## <a name="createobject"></a>createObject

`createObject(key1, value1, key2, value2, ...)`

Создает объект из ключей и значений. `createObject`Функция не поддерживается бицеп.  Создайте объект с помощью `{}` .

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| key1 |Нет |строка |Имя ключа. |
| value1 |Нет |int, Boolean, строка, объект или массив |Значение для ключа. |
| Дополнительные ключи |Нет |строка |Дополнительные имена ключей. |
| дополнительные значения |Нет |int, Boolean, строка, объект или массив |Дополнительные значения для ключей. |

Функция принимает только четное число параметров. Каждый ключ должен иметь совпадающее значение.

### <a name="return-value"></a>Возвращаемое значение

Объект с каждой парой "ключ — значение".

### <a name="example"></a>Пример

В следующем примере создается объект из различных типов значений.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
  ],
  "outputs": {
    "newObject": {
      "type": "object",
      "value": "[createObject('intProp', 1, 'stringProp', 'abc', 'boolProp', true(), 'arrayProp', createArray('a', 'b', 'c'), 'objectProp', createObject('key1', 'value1'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
output newObject object = {
  'intProp': 1
  'stringProp': 'abc'
  'boolProp': true
  'arrayProp': [
    'a'
    'b'
    'c'
  ]
  'objectProp': {
    'key1': 'value1'
  }
}
```

---

Выходные данные из предыдущего примера со значениями по умолчанию — объект с именем `newObject` со следующим значением:

```json
{
  "intProp": 1,
  "stringProp": "abc",
  "boolProp": true,
  "arrayProp": ["a", "b", "c"],
  "objectProp": {"key1": "value1"}
}
```

## <a name="empty"></a>пустых

`empty(itemToTest)`

Определяет, являются ли пустыми массив, объект или строка.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| itemToTest |Да |массив, объект или строка |Значение, которое необходимо проверить, если оно пустое. |

### <a name="return-value"></a>Возвращаемое значение

Возвращает результат **True**, если значение пустое. В противном случае — **False**.

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/empty.json) проверяется, являются ли пустыми массив, объект и строка.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "testArray": {
      "type": "array",
      "defaultValue": []
    },
    "testObject": {
      "type": "object",
      "defaultValue": {}
    },
    "testString": {
      "type": "string",
      "defaultValue": ""
    }
  },
  "resources": [
  ],
  "outputs": {
    "arrayEmpty": {
      "type": "bool",
      "value": "[empty(parameters('testArray'))]"
    },
    "objectEmpty": {
      "type": "bool",
      "value": "[empty(parameters('testObject'))]"
    },
    "stringEmpty": {
      "type": "bool",
      "value": "[empty(parameters('testString'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param testArray array = []
param testObject object = {}
param testString string = ''

output arrayEmpty bool = empty(testArray)
output objectEmpty bool = empty(testObject)
output stringEmpty bool = empty(testString)
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| arrayEmpty | Bool | True |
| objectEmpty | Bool | True |
| stringEmpty | Bool | True |

## <a name="intersection"></a>пересечению

`intersection(arg1, arg2, arg3, ...)`

Возвращает из параметров один массив или объект с общими элементами.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| arg1 |Да |массив или объект |Первое значение для поиска общих элементов. |
| arg2 |Да |массив или объект |Второе значение для поиска общих элементов. |
| дополнительные аргументы |нет |массив или объект |Дополнительные значения для поиска общих элементов. |

### <a name="return-value"></a>Возвращаемое значение

Массив или объект с общими элементами.

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/intersection.json) показано, как использовать пересечение с массивами и объектами.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "firstObject": {
      "type": "object",
      "defaultValue": {
        "one": "a",
        "two": "b",
        "three": "c"
      }
    },
    "secondObject": {
      "type": "object",
      "defaultValue": {
        "one": "a",
        "two": "z",
        "three": "c"
      }
    },
    "firstArray": {
      "type": "array",
      "defaultValue": [ "one", "two", "three" ]
    },
    "secondArray": {
      "type": "array",
      "defaultValue": [ "two", "three" ]
    }
  },
  "resources": [
  ],
  "outputs": {
    "objectOutput": {
      "type": "object",
      "value": "[intersection(parameters('firstObject'), parameters('secondObject'))]"
    },
    "arrayOutput": {
      "type": "array",
      "value": "[intersection(parameters('firstArray'), parameters('secondArray'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param firstObject object = {
  'one': 'a'
  'two': 'b'
  'three': 'c'
}
param secondObject object = {
  'one': 'a'
  'two': 'z'
  'three': 'c'
}
param firstArray array = [
  'one'
  'two'
  'three'
]
param secondArray array = [
  'two'
  'three'
]

output objectOutput object = intersection(firstObject, secondObject)
output arrayOutput array = intersection(firstArray, secondArray)
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| objectOutput | Объект | {"one": "a", "three": "c"} |
| arrayOutput | Array | ["two", "three"] |

<a id="json"></a>

## <a name="json"></a>json

`json(arg1)`

Преобразует допустимую строку JSON в тип данных JSON.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| arg1 |Да |строка |Значение для преобразования в JSON. Строка должна быть правильно отформатированной строкой JSON. |

### <a name="return-value"></a>Возвращаемое значение

Тип данных JSON из указанной строки или пустое значение, если указано **значение NULL** .

### <a name="remarks"></a>Remarks

Если необходимо включить значение параметра или переменную в объект JSON, используйте функцию [concat](template-functions-string.md#concat) для создания строки, передаваемой в функцию.

Для получения значения NULL можно также использовать [значение null ()](#null) .

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/json.json) показано, как использовать функцию JSON. Обратите внимание, что для пустого объекта можно передать **значение NULL** .

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "jsonEmptyObject": {
      "type": "string",
      "defaultValue": "null"
    },
    "jsonObject": {
      "type": "string",
      "defaultValue": "{\"a\": \"b\"}"
    },
    "jsonString": {
      "type": "string",
      "defaultValue": "\"test\""
    },
    "jsonBoolean": {
      "type": "string",
      "defaultValue": "true"
    },
    "jsonInt": {
      "type": "string",
      "defaultValue": "3"
    },
    "jsonArray": {
      "type": "string",
      "defaultValue": "[[1,2,3 ]"
    },
    "concatValue": {
      "type": "string",
      "defaultValue": "demo value"
    }
  },
  "resources": [
  ],
  "outputs": {
    "emptyObjectOutput": {
      "type": "bool",
      "value": "[empty(json(parameters('jsonEmptyObject')))]"
    },
    "objectOutput": {
      "type": "object",
      "value": "[json(parameters('jsonObject'))]"
    },
    "stringOutput": {
      "type": "string",
      "value": "[json(parameters('jsonString'))]"
    },
    "booleanOutput": {
      "type": "bool",
      "value": "[json(parameters('jsonBoolean'))]"
    },
    "intOutput": {
      "type": "int",
      "value": "[json(parameters('jsonInt'))]"
    },
    "arrayOutput": {
      "type": "array",
      "value": "[json(parameters('jsonArray'))]"
    },
    "concatObjectOutput": {
      "type": "object",
      "value": "[json(concat('{\"a\": \"', parameters('concatValue'), '\"}'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param jsonEmptyObject string = 'null'
param jsonObject string = '{\'a\': \'b\'}'
param jsonString string = '\'test\''
param jsonBoolean string = 'true'
param jsonInt string = '3'
param jsonArray string = '[[1,2,3]]'
param concatValue string = 'demo value'

output emptyObjectOutput bool = empty(json(jsonEmptyObject))
output objectOutput object = json(jsonObject)
output stringOutput string =json(jsonString)
output booleanOutput bool = json(jsonBoolean)
output intOutput int = json(jsonInt)
output arrayOutput array = json(jsonArray)
output concatObjectOutput object = json(concat('{"a": "', concatValue, '"}'))
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| емптйобжектаутпут | Логическое значение | True |
| objectOutput | Объект | {"a": "b"} |
| stringOutput | Строка | номер |
| булеанаутпут | Логическое значение | True |
| intOutput | Целое число | 3 |
| arrayOutput | Array | [1, 2, 3] |
| конкатобжектаутпут | Объект | {"a": "демонстрационное значение"} |

## <a name="length"></a>length

`length(arg1)`

Возвращает количество элементов в массиве, символах в строке или свойствах корневого уровня в объекте.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| arg1 |Да |массив, строка или объект |Массив, используемый для получения числа элементов, строки, используемой для получения числа символов, или объекта, используемого для получения числа свойств корневого уровня. |

### <a name="return-value"></a>Возвращаемое значение

Целое число.

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/length.json) показано, как использовать функцию length с массивом и строкой.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "arrayToTest": {
      "type": "array",
      "defaultValue": [
        "one",
        "two",
        "three"
      ]
    },
    "stringToTest": {
      "type": "string",
      "defaultValue": "One Two Three"
    },
    "objectToTest": {
      "type": "object",
      "defaultValue": {
        "propA": "one",
        "propB": "two",
        "propC": "three",
        "propD": {
          "propD-1": "sub",
          "propD-2": "sub"
        }
      }
    }
  },
  "resources": [],
  "outputs": {
    "arrayLength": {
      "type": "int",
      "value": "[length(parameters('arrayToTest'))]"
    },
    "stringLength": {
      "type": "int",
      "value": "[length(parameters('stringToTest'))]"
    },
    "objectLength": {
      "type": "int",
      "value": "[length(parameters('objectToTest'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param arrayToTest array = [
  'one'
  'two'
  'three'
]
param stringToTest string = 'One Two Three'
param objectToTest object = {
  'propA': 'one'
  'propB': 'two'
  'propC': 'three'
  'propD': {
      'propD-1': 'sub'
      'propD-2': 'sub'
  }
}

output arrayLength int = length(arrayToTest)
output stringLength int = length(stringToTest)
output objectLength int = length(objectToTest)
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| arrayLength | Int | 3 |
| stringLength | Int | 13 |
| обжектленгс | Int | 4 |

## <a name="null"></a>null

`null()`

Возвращает значение NULL. `null`Функция недоступна в бицеп. `null`Вместо этого используйте ключевое слово.

### <a name="parameters"></a>Параметры

Функция null не принимает никаких параметров.

### <a name="return-value"></a>Возвращаемое значение

Значение, всегда равное NULL.

### <a name="example"></a>Пример

В следующем примере используется функция null.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [],
  "outputs": {
    "emptyOutput": {
      "type": "bool",
      "value": "[empty(null())]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
output emptyOutput bool = empty(null)
```

---

Выходные данные из предыдущего примера:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| emptyOutput | Bool | True |

## <a name="union"></a>union

`union(arg1, arg2, arg3, ...)`

Возвращает из параметров один массив или объект со всеми элементами. Повторяющиеся значения или ключи включаются только один раз.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| arg1 |Да |массив или объект |Первое значение для объединения элементов. |
| arg2 |Да |массив или объект |Второе значение для объединения элементов. |
| дополнительные аргументы |нет |массив или объект |Дополнительные значения для объединения элементов. |

### <a name="return-value"></a>Возвращаемое значение

Массив или объект.

### <a name="example"></a>Пример

В следующем [примере шаблона](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/union.json) показано, как использовать функцию union с массивами и объектами.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "firstObject": {
      "type": "object",
      "defaultValue": {
        "one": "a",
        "two": "b",
        "three": "c1"
      }
    },
    "secondObject": {
      "type": "object",
      "defaultValue": {
        "three": "c2",
        "four": "d",
        "five": "e"
      }
    },
    "firstArray": {
      "type": "array",
      "defaultValue": [ "one", "two", "three" ]
    },
    "secondArray": {
      "type": "array",
      "defaultValue": [ "three", "four" ]
    }
  },
  "resources": [
  ],
  "outputs": {
    "objectOutput": {
      "type": "object",
      "value": "[union(parameters('firstObject'), parameters('secondObject'))]"
    },
    "arrayOutput": {
      "type": "array",
      "value": "[union(parameters('firstArray'), parameters('secondArray'))]"
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param firstObject object = {
  'one': 'a'
  'two': 'b'
  'three': 'c1'
}

param secondObject object = {
  'three': 'c2'
  'four': 'd'
  'five': 'e'
}

param firstArray array = [
  'one'
  'two'
  'three'
]

param secondArray array = [
  'three'
  'four'
]

output objectOutput object = union(firstObject, secondObject)
output arrayOutput array = union(firstArray, secondArray)
```

---

Выходные данные из предыдущего примера со значениями по умолчанию:

| Имя | Type | Значение |
| ---- | ---- | ----- |
| objectOutput | Объект | {"one": "a", "two": "b", "three": "c2", "four": "d", "five": "e"} |
| arrayOutput | Array | ["one", "two", "three", "four"] |

## <a name="next-steps"></a>Дальнейшие действия

* Описание разделов в шаблоне ARM см. [в разделе Общие сведения о структуре и синтаксисе шаблонов ARM](template-syntax.md).
