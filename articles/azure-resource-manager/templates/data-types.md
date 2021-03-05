---
title: Типы данных в шаблонах
description: Описывает типы данных, доступные в шаблонах Azure Resource Manager.
ms.topic: conceptual
ms.author: tomfitz
author: tfitzmac
ms.date: 03/04/2021
ms.openlocfilehash: 7d3f15c8852e6e25c621baad9bc6f20c303ffdb9
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102125139"
---
# <a name="data-types-in-arm-templates"></a>Типы данных в шаблонах ARM

В этой статье описываются типы данных, поддерживаемые в шаблонах Azure Resource Manager (шаблоны ARM). Он охватывает типы данных JSON и Бицеп.

## <a name="supported-types"></a>Поддерживаемые типы

В шаблоне ARM можно использовать следующие типы данных:

* array
* bool
* INT
* объект
* secureObject — указывается модификатором в Бицеп
* secureString — указывается модификатором в Бицеп
* строка

## <a name="arrays"></a>Массивы

Массивы начинаются с левой квадратной скобки ( `[` ) и заканчиваются правой квадратной скобкой ( `]` ).

В JSON массив можно объявить в одной строке или нескольких строках. Каждый элемент отделяется запятой.

В Бицеп массив должен быть объявлен в нескольких строках. Не используйте запятые между значениями.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "exampleArray": {
    "type": "array",
    "defaultValue": [
      1,
      2,
      3
    ]
  }
},
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param exampleArray array = [
  1
  2
  3
]
```

---

Элементы массива могут быть одного типа или разных типов.

# <a name="json"></a>[JSON](#tab/json)

```json
"variables": {
  "mixedArray": [
    "[resourceGroup().name]",
    1,
    true,
    "example string"
  ]
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
var mixedArray = [
  resourceGroup().name
  1
  true
  'example string'
]
```

---

## <a name="booleans"></a>Boolean

При указании логических значений используйте `true` или `false` . Не заключайте значение в кавычки.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "exampleBool": {
    "type": "bool",
    "defaultValue": true
  }
},
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param exampleBool bool = true
```

---

## <a name="integers"></a>Целые числа

При указании целочисленных значений не используйте кавычки.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "exampleInt": {
    "type": "int",
    "defaultValue": 1
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param exampleInt int = 1
```

---

Для целых чисел, переданных в качестве встроенных параметров, диапазон значений может быть ограничен пакетом SDK или программой командной строки, используемой для развертывания. Например, при использовании PowerShell для развертывания шаблона целочисленные типы могут находиться в диапазоне от-2147483648 до 2147483647. Чтобы избежать этого ограничения, укажите большие целочисленные значения в [файле параметров](parameter-files.md). Типы ресурсов применяют собственные ограничения для целочисленных свойств.

## <a name="objects"></a>Объекты

Объекты начинаются с левой фигурной скобки ( `{` ) и заканчиваются закрывающей фигурной скобкой ( `}` ). Каждое свойство в объекте состоит из ключа и значения. Ключ и значение разделяются двоеточием ( `:` ).

В JSON ключ заключен в двойные кавычки. Каждое свойство разделяются запятыми.

В Бицеп ключ не заключен в кавычки. Не используйте запятые между свойствами.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "exampleObject": {
    "type": "object",
    "defaultValue": {
      "name": "test name",
      "id": "123-abc",
      "isCurrent": true,
      "tier": 1
    }
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param exampleObject object = {
  name: 'test name'
  id: '123-abc'
  isCurrent: true
  tier: 1
}
```

---

## <a name="strings"></a>Строки

В JSON строки помечаются двойными кавычками. В Бицеп строки помечаются одинарными кавычками.

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "exampleString": {
    "type": "string",
    "defaultValue": "test value"
  }
},
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param exampleString string = 'test value'
```
---

## <a name="secure-strings-and-objects"></a>Защита строк и объектов

Безопасная строка использует тот же формат, что и строка, а защищенный объект использует тот же формат, что и объект. Если для параметра задана Безопасная строка или защищенный объект, значение параметра не сохраняется в журнале развертывания и не регистрируется. Однако если задать для свойства безопасное значение, которое не ожидает безопасного значения, это значение не будет защищено. Например, если для защищенной строки задан тег, это значение сохраняется как обычный текст. Используйте защищенные строки для паролей и секретов.

С помощью Бицеп вы добавляете `@secure()` Модификатор в строку или объект.

В следующем примере показаны два защищенных параметра:

# <a name="json"></a>[JSON](#tab/json)

```json
"parameters": {
  "password": {
    "type": "secureString"
  },
  "configValues": {
    "type": "secureObject"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
@secure()
param password string

@secure()
param configValues object
```

---

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о синтаксисе шаблона см. в разделе [сведения о структуре и синтаксисе шаблонов ARM](template-syntax.md).
