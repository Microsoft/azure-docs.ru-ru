---
title: Создание политик для свойств массива ресурсов
description: Узнайте, как работать с параметрами массива и выражениями языка массива, оценивать псевдоним [*] и добавлять элементы с помощью правил определения Политики Azure.
ms.date: 10/22/2020
ms.topic: how-to
ms.openlocfilehash: 75f4fcfb88bd4cb1ac0c8bfeac236b452479b8c6
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104721619"
---
# <a name="author-policies-for-array-properties-on-azure-resources"></a>Создание политик для свойств массива ресурсов Azure

Свойства Azure Resource Manager обычно определяются как строки и логические значения. Если существует связь "один ко многим", то сложные свойства определяются как массивы. В Политике Azure массивы используются несколькими различными способами:

- тип [параметра определения](../concepts/definition-structure.md#parameters) для предоставления нескольких параметров;
- часть [правила политики](../concepts/definition-structure.md#policy-rule) с использованием условий **in** или **notIn**;
- Часть правила политики, которая подсчитывает, сколько элементов массива удовлетворяет условию
- В эффектах [добавления](../concepts/effects.md#append) и [изменения](../concepts/effects.md#modify) для обновления существующего массива

В этой статье рассматривается каждый вариант использования в Политике Azure и приводится несколько примеров определений.

## <a name="parameter-arrays"></a>Массивы параметров

### <a name="define-a-parameter-array"></a>Определение массива параметров

Определение параметра в качестве массива обеспечивает гибкость политики, если требуется более одного значения.
Это определение политики допускает любое отдельное расположение в параметре **allowedLocations** и по умолчанию использует значение _eastus2_:

```json
"parameters": {
    "allowedLocations": {
        "type": "string",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": "eastus2"
    }
}
```

Так как параметр **type** имеет значение _string_, при назначении этой политики можно задать только одно значение. При назначении этой политики ресурсы в области допускаются только в пределах одного региона Azure. Большинство определений политик должно допускать список утвержденных параметров, например _eastus2_, _eastus_ и _westus2_.

Чтобы создать определение политики, допускающее несколько параметров, используйте свойство **type** со значением _array_. Эту же политику можно переписать следующим образом:

```json
"parameters": {
    "allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": "eastus2",
        "allowedValues": [
            "eastus2",
            "eastus",
            "westus2"
        ]

    }
}
```

> [!NOTE]
> После сохранения определения политики изменить свойство **type** параметра невозможно.

Это новое определение параметра принимает более одного значения во время назначения политики. Если определено свойство массива **allowedValues**, значения, доступные во время назначения, дополнительно ограничиваются предопределенным списком. Использовать **allowedValues** необязательно.

### <a name="pass-values-to-a-parameter-array-during-assignment"></a>Передача значений в массив параметров во время назначения

При назначении политики с помощью портала Azure параметр с **типом** _массива_ отображается как одно текстовое поле. В подсказке указано "Указывайте значения через ;. (например, Лондон;Нью-Йорк)". Чтобы передать допустимые значения расположения _eastus2_, _eastus_ и _westus2_ в параметр, используйте следующую строку:

`eastus2;eastus;westus2`

Формат значения параметра различается при использовании Azure CLI, Azure PowerShell или REST API. Значения передаются в строке JSON, которая также содержит имя параметра.

```json
{
    "allowedLocations": {
        "value": [
            "eastus2",
            "eastus",
            "westus2"
        ]
    }
}
```

Чтобы использовать эту строку с каждым пакетом SDK, выполните следующие команды:

- Azure CLI: команда [az policy assignment create](/cli/azure/policy/assignment#az_policy_assignment_create) с параметром **params**;
- Azure PowerShell: командлет [New-AzPolicyAssignment](/powershell/module/az.resources/New-Azpolicyassignment) с параметром **PolicyParameter**;
- REST API: операция _PUT_ [create](/rest/api/resources/policyassignments/create) в тексте запроса в качестве значения свойства **properties.parameters**.

## <a name="using-arrays-in-conditions"></a>Использование массивов в условиях

### <a name="in-and-notin"></a>`In` и `notIn`

`in`Условия и `notIn` работают только со значениями массива. Они проверяют существование значения в массиве. Массив может быть литеральным массивом JSON или ссылкой на параметр массива. Пример:

```json
{
      "field": "tags.environment",
      "in": [ "dev", "test" ]
}
```

```json
{
      "field": "location",
      "notIn": "[parameters('allowedLocations')]"
}
```

### <a name="value-count"></a>Число значений

В выражении [числа значений](../concepts/definition-structure.md#value-count) показано количество элементов массива, соответствующих условию. Он позволяет вычислять одно и то же условие несколько раз, используя разные значения для каждой итерации. Например, следующее условие проверяет, соответствует ли имя ресурса любому шаблону из массива шаблонов:

```json
{
    "count": {
        "value": [ "test*", "dev*", "prod*" ],
        "name": "pattern",
        "where": {
            "field": "name",
            "like": "[current('pattern')]"
        }
    },
    "greater": 0
}
```

Чтобы вычислить выражение, политика Azure вычисляет `where` условие 3 раза, по одному разу для каждого члена `[ "test*", "dev*", "prod*" ]` , подсчитывающее количество раз, в которое оно было оценено `true` . При каждой итерации значение текущего элемента массива объединяется с `pattern` именем индекса, определенным `count.name` . На это значение можно ссылаться внутри `where` условия, вызывая специальную функцию шаблона: `current('pattern')` .

| Итерация | `current('pattern')` Возвращаемое значение |
|:---|:---|
| 1 | `"test*"` |
| 2 | `"dev*"` |
| 3 | `"prod*"` |

Условие имеет значение true, только если полученное число больше 0.

Чтобы сделать условие более универсальным, используйте ссылку на параметр вместо массива литералов:

 ```json
{
    "count": {
        "value": "[parameters('patterns')]",
        "name": "pattern",
        "where": {
            "field": "name",
            "like": "[current('pattern')]"
        }
    },
    "greater": 0
}
```

Если выражение **счетчика значений** не находится ни под каким другим выражением **Count** , `count.name` является необязательным и `current()` функция может использоваться без аргументов:

```json
{
    "count": {
        "value": "[parameters('patterns')]",
        "where": {
            "field": "name",
            "like": "[current()]"
        }
    },
    "greater": 0
}
```

**Значение Count** также поддерживает массивы сложных объектов, что позволяет более сложные условия. Например, следующее условие определяет требуемое значение тега для каждого шаблона имени и проверяет, соответствует ли имя ресурса шаблону, но не имеет требуемого значения тега:

```json
{
    "count": {
        "value": [
            { "pattern": "test*", "envTag": "dev" },
            { "pattern": "dev*", "envTag": "dev" },
            { "pattern": "prod*", "envTag": "prod" },
        ],
        "name": "namePatternRequiredTag",
        "where": {
            "allOf": [
                {
                    "field": "name",
                    "like": "[current('namePatternRequiredTag').pattern]"
                },
                {
                    "field": "tags.env",
                    "notEquals": "[current('namePatternRequiredTag').envTag]"
                }
            ]
        }
    },
    "greater": 0
}
```

Полезные примеры см. в разделе [примеры числа значений](../concepts/definition-structure.md#value-count-examples).

## <a name="referencing-array-resource-properties"></a>Ссылки на свойства ресурсов массива

Во многих вариантах использования требуется работа со свойствами массива в вычисляемом ресурсе. Для некоторых сценариев требуется ссылка на весь массив (например, проверку его длины). Для других требуется применение условия к каждому отдельному элементу массива (например, убедитесь, что все правила брандмауэра блокируют доступ из Интернета). Основные сведения о различных способах, с помощью которых политика Azure может ссылаться на свойства ресурсов, а также о том, как эти ссылки ведут себя, когда они ссылаются на свойства массива, являются ключом для написания условий, охватывающих эти сценарии.

### <a name="referencing-resource-properties"></a>Ссылки на свойства ресурсов

На свойства ресурсов можно ссылаться в политике Azure, используя [псевдонимы](../concepts/definition-structure.md#aliases) . Существует два способа ссылки на значения свойства ресурса в политике Azure:

- Используйте условие [поля](../concepts/definition-structure.md#fields) , чтобы проверить, соответствуют ли **все** выбранные свойства ресурса условию. Пример

  ```json
  {
    "field" : "Microsoft.Test/resourceType/property",
    "equals": "value"
  }
  ```

- Используйте `field()` функцию для доступа к значению свойства. Пример

  ```json
  {
    "value": "[take(field('Microsoft.Test/resourceType/property'), 7)]",
    "equals": "prefix_"
  }
  ```

Условие поля имеет неявное поведение "все". Если псевдоним представляет коллекцию значений, он проверяет, соответствуют ли все отдельные значения условию. `field()`Функция возвращает значения, представленные псевдонимом "как есть", который затем может управляться другими функциями шаблонов.

### <a name="referencing-array-fields"></a>Ссылки на поля массивов

Свойства ресурсов массива обычно представлены двумя разными типами псевдонимов. Один псевдоним "Обычная" и [псевдонимы массива](../concepts/definition-structure.md#understanding-the--alias) , `[*]` присоединенные к нему:

- `Microsoft.Test/resourceType/stringArray`
- `Microsoft.Test/resourceType/stringArray[*]`

#### <a name="referencing-the-array"></a>Ссылка на массив

Первый псевдоним представляет одно значение, значение `stringArray` свойства из содержимого запроса. Поскольку значение этого свойства является массивом, оно не очень полезно в условиях политики. Пример:

```json
{
  "field": "Microsoft.Test/resourceType/stringArray",
  "equals": "..."
}
```

Это условие сравнивает весь `stringArray` массив с одним строковым значением. Большинство условий, в том числе `equals` , принимают только строковые значения, поэтому при сравнении массива со строкой не используется. Основной сценарий, в котором используется ссылка на свойство Array, полезен при проверке существования этого свойства:

```json
{
  "field": "Microsoft.Test/resourceType/stringArray",
  "exists": "true"
}
```

При использовании `field()` функции возвращаемое значение представляет собой массив из содержимого запроса, который затем можно использовать с любыми [поддерживаемыми шаблонными функциями](../concepts/definition-structure.md#policy-functions) , принимающими аргументы массива. Например, следующее условие проверяет, имеет ли длина `stringArray` значение больше 0:

```json
{
  "value": "[length(field('Microsoft.Test/resourceType/stringArray'))]",
  "greater": 0
}
```

#### <a name="referencing-the-array-members-collection"></a>Ссылка на коллекцию элементов массива

Псевдонимы, использующие `[*]` синтаксис, представляют **коллекцию значений свойств, выбранных из свойства массива**, что отличается от выбора самого свойства массива. В случае `Microsoft.Test/resourceType/stringArray[*]` он возвращает коллекцию, имеющую все члены `stringArray` . Как упоминалось ранее, `field` Условие проверяет, что все выбранные свойства ресурсов соответствуют условию, поэтому следующее условие выполняется, только если **все** члены `stringArray` равны "" значению "".

```json
{
  "field": "Microsoft.Test/resourceType/stringArray[*]",
  "equals": "value"
}
```

Если массив содержит объекты, `[*]` можно использовать псевдоним для выбора значения определенного свойства из каждого члена массива. Пример

```json
{
  "field": "Microsoft.Test/resourceType/objectArray[*].property",
  "equals": "value"
}
```

Это условие имеет значение true, если значения всех `property` свойств в `objectArray` равны `"value"` . Дополнительные примеры см. в разделе [Дополнительные \[ \* \] примеры псевдонимов](#appendix--additional--alias-examples).

При использовании `field()` функции для ссылки на псевдоним массива возвращаемое значение является массивом всех выбранных значений. Такое поведение означает, что общий вариант использования `field()` функции, который позволяет применять функции шаблонов к значениям свойств ресурсов, очень ограничен. В этом случае можно использовать только те функции шаблона, которые принимают аргументы массива. Например, длину массива можно получить с помощью `[length(field('Microsoft.Test/resourceType/objectArray[*].property'))]` . Однако более сложные сценарии, такие как применение функции шаблона к каждому элементу массива и его сравнение с нужным значением, возможны только при использовании `count` выражения. Дополнительные сведения см. в разделе [выражение количества полей](#field-count-expressions).

Итоговые сведения см. в следующем примере содержимого ресурса и выбранных значениях, возвращенных различными псевдонимами:

```json
{
  "tags": {
    "env": "prod"
  },
  "properties":
  {
    "stringArray": [ "a", "b", "c" ],
    "objectArray": [
      {
        "property": "value1",
        "nestedArray": [ 1, 2 ]
      },
      {
        "property": "value2",
        "nestedArray": [ 3, 4 ]
      }
    ]
  }
}
```

При использовании условия поля в примере содержимого ресурса будут получены следующие результаты.

| Псевдоним | Выбранные значения |
|:--- |:---|
| `Microsoft.Test/resourceType/missingArray` | `null` |
| `Microsoft.Test/resourceType/missingArray[*]` | Пустая коллекция значений. |
| `Microsoft.Test/resourceType/missingArray[*].property` | Пустая коллекция значений. |
| `Microsoft.Test/resourceType/stringArray` | `["a", "b", "c"]` |
| `Microsoft.Test/resourceType/stringArray[*]` | `"a"`, `"b"`, `"c"` |
| `Microsoft.Test/resourceType/objectArray[*]` |  `{ "property": "value1", "nestedArray": [ 1, 2 ] }`,<br/>`{ "property": "value2", "nestedArray": [ 3, 4 ] }`|
| `Microsoft.Test/resourceType/objectArray[*].property` | `"value1"`, `"value2"` |
| `Microsoft.Test/resourceType/objectArray[*].nestedArray` | `[ 1, 2 ]`, `[ 3, 4 ]` |
| `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` | `1`, `2`, `3`, `4` |

При использовании `field()` функции в примере содержимого ресурса будут получены следующие результаты.

| Expression | Возвращаемое значение |
|:--- |:---|
| `[field('Microsoft.Test/resourceType/missingArray')]` | `""` |
| `[field('Microsoft.Test/resourceType/missingArray[*]')]` | `[]` |
| `[field('Microsoft.Test/resourceType/missingArray[*].property')]` | `[]` |
| `[field('Microsoft.Test/resourceType/stringArray')]` | `["a", "b", "c"]` |
| `[field('Microsoft.Test/resourceType/stringArray[*]')]` | `["a", "b", "c"]` |
| `[field('Microsoft.Test/resourceType/objectArray[*]')]` |  `[{ "property": "value1", "nestedArray": [ 1, 2 ] }, { "property": "value2", "nestedArray": [ 3, 4 ] }]`|
| `[field('Microsoft.Test/resourceType/objectArray[*].property')]` | `["value1", "value2"]` |
| `[field('Microsoft.Test/resourceType/objectArray[*].nestedArray')]` | `[[ 1, 2 ], [ 3, 4 ]]` |
| `[field('Microsoft.Test/resourceType/objectArray[*].nestedArray[*]')]` | `[1, 2, 3, 4]` |

### <a name="field-count-expressions"></a>Выражения счетчика полей

Выражение [количества полей](../concepts/definition-structure.md#field-count) подсчитывает количество элементов массива, отвечающих условию, и сравнивает их с целевым значением. `Count` является более интуитивным и универсальным для оценки массивов по сравнению с `field` условиями. Синтаксис выглядит следующим образом.

```json
{
  "count": {
    "field": <[*] alias>,
    "where": <optional policy condition expression>
  },
  "equals|greater|less|any other operator": <target value>
}
```

При использовании без `where` условия `count` просто возвращает длину массива. В примере содержимого ресурса из предыдущего раздела `count` вычисляется следующее выражение, `true` поскольку `stringArray` имеет три члена:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/stringArray[*]"
  },
  "equals": 3
}
```

Это поведение также работает с вложенными массивами. Например, следующее `count` выражение вычисляется `true` как, так как в массивах есть четыре члена массива `nestedArray` :

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/objectArray[*].nestedArray[*]"
  },
  "greaterOrEquals": 4
}
```

Сила `count` в `where` состоянии. Когда оно указано, политика Azure перечисляет элементы массива и оценивает каждое условие, подчисляя число вычисляемых элементов массива `true` . В частности, в каждой итерации `where` оценки условия Политика Azure выбирает один член массива ***i** _ и оценивает содержимое ресурса по `where` условию _*, как если **_i_*_ является единственным членом array_ *. Наличие только одного члена массива в каждой итерации предоставляет способ применения сложных условий к каждому отдельному элементу массива.

Пример

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/stringArray[*]",
    "where": {
      "field": "Microsoft.Test/resourceType/stringArray[*]",
      "equals": "a"
    }
  },
  "equals": 1
}
```
Чтобы вычислить `count` выражение, политика Azure вычисляет `where` условие 3 раза, по одному разу для каждого члена `stringArray` , подсчитывающее количество раз, в которое оно было оценено `true` . Если `where` условие ссылается на `Microsoft.Test/resourceType/stringArray[*]` члены массива, вместо того чтобы выбирать все члены `stringArray` , он будет выбирать только один элемент массива каждый раз:

| Итерация | Выбранные `Microsoft.Test/resourceType/stringArray[*]` значения | `where` Результат оценки |
|:---|:---|:---|
| 1 | `"a"` | `true` |
| 2 | `"b"` | `false` |
| 3 | `"c"` | `false` |

И, таким же, `count` будет возвращен `1` .

Вот более сложное выражение:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/objectArray[*]",
    "where": {
      "allOf": [
        {
          "field": "Microsoft.Test/resourceType/objectArray[*].property",
          "equals": "value2"
        },
        {
          "field": "Microsoft.Test/resourceType/objectArray[*].nestedArray[*]",
          "greater": 2
        }
      ]
    }
  },
  "equals": 1
}
```

| Итерация | Выбранные значения | `where` Результат оценки |
|:---|:---|:---|
| 1 | `Microsoft.Test/resourceType/objectArray[*].property` => `"value1"` </br> `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` => `1`, `2` | `false` |
| 2 | `Microsoft.Test/resourceType/objectArray[*].property` => `"value2"` </br> `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` => `3`, `4`| `true` |

И поэтому `count` возвращает `1` .

Тот факт, что `where` выражение вычисляется по **всему** содержимому запроса (с изменениями только для члена массива, перечисленного в настоящее время) означает, что `where` условие также может ссылаться на поля за пределами массива:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/objectArray[*]",
    "where": {
      "field": "tags.env",
      "equals": "prod"
    }
  },
  "equals": 0
}
```

| Итерация | Выбранные значения | `where` Результат оценки |
|:---|:---|:---|
| 1 | `tags.env` => `"prod"` | `true` |
| 2 | `tags.env` => `"prod"` | `true` |

Выражения вложенных подсчетов можно использовать для применения условий к вложенным полям массива. Например, следующее условие проверяет, что `objectArray[*]` массив содержит ровно 2 члена с `nestedArray[*]` тем, что содержит 1 или более элементов:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/objectArray[*]",
    "where": {
      "count": {
        "field": "Microsoft.Test/resourceType/objectArray[*].nestedArray[*]"
      },
      "greaterOrEquals": 1
    }
  },
  "equals": 2
}
```

| Итерация | Выбранные значения | Результат вычисления вложенного счетчика |
|:---|:---|:---|
| 1 | `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` => `1`, `2` | `nestedArray[*]` содержит 2 члена => `true` |
| 2 | `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` => `3`, `4` | `nestedArray[*]` содержит 2 члена => `true` |

Поскольку оба элемента `objectArray[*]` имеют дочерний массив `nestedArray[*]` с 2 элементами, выражение внешнего числа возвращает `2` .

Более сложный пример: Убедитесь, что `objectArray[*]` массив содержит ровно 2 члена с `nestedArray[*]` любыми элементами, равными `2` или `3` :

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/objectArray[*]",
    "where": {
      "count": {
        "field": "Microsoft.Test/resourceType/objectArray[*].nestedArray[*]",
        "where": {
            "field": "Microsoft.Test/resourceType/objectArray[*].nestedArray[*]",
            "in": [ 2, 3 ]
        }
      },
      "greaterOrEquals": 1
    }
  },
  "equals": 2
}
```

| Итерация | Выбранные значения | Результат вычисления вложенного счетчика
|:---|:---|:---|
| 1 | `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` => `1`, `2` | `nestedArray[*]` содержащих `2` => `true` |
| 2 | `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` => `3`, `4` | `nestedArray[*]` содержащих `3` => `true` |

Поскольку оба элемента `objectArray[*]` имеют дочерний массив `nestedArray[*]` , который содержит `2` или `3` , выражение внешнего счетчика возвращает значение `2` .

> [!NOTE]
> Выражения подсчета вложенных полей могут ссылаться только на вложенные массивы. Например, выражение Count, ссылающееся на, `Microsoft.Test/resourceType/objectArray[*]` может иметь вложенный счетчик, нацеленный на вложенный массив `Microsoft.Test/resourceType/objectArray[*].nestedArray[*]` , но не может иметь целевое выражение вложенного счетчика `Microsoft.Test/resourceType/stringArray[*]` .

#### <a name="accessing-current-array-member-with-template-functions"></a>Доступ к текущему члену массива с помощью функций шаблона

При использовании функций шаблона используйте `current()` функцию для доступа к значению текущего элемента массива или значений любого из его свойств. Чтобы получить доступ к значению текущего элемента массива, передайте псевдоним, определенный в, `count.field` или его дочерние псевдонимы в качестве аргумента `current()` функции. Пример:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/objectArray[*]",
    "where": {
        "value": "[current('Microsoft.Test/resourceType/objectArray[*].property')]",
        "like": "value*"
    }
  },
  "equals": 2
}

```

| Итерация | `current()` Возвращаемое значение | `where` Результат оценки |
|:---|:---|:---|
| 1 | Значение `property` в первом члене `objectArray[*]` : `value1` | `true` |
| 2 | Значение `property` в первом члене `objectArray[*]` : `value2` | `true` |

#### <a name="the-field-function-inside-where-conditions"></a>Функция поля в условиях WHERE

`field()`Функцию также можно использовать для доступа к значению текущего элемента массива, если выражение **Count** не находится внутри **условия существования** ( `field()` функция всегда обращается к ресурсу, вычисленному в условии **If** ).
Поведение `field()` при ссылке на вычисляемый массив основано на следующих понятиях:
1. Псевдонимы массивов разрешаются в коллекцию значений, выбранных из всех элементов массива.
1. `field()` функции, ссылающиеся на псевдонимы массивов, возвращают массив с выбранными значениями.
1. Обращение к подсчитанному псевдониму массива внутри `where` условия возвращает коллекцию с одним значением, выбранным из члена массива, который вычисляется в текущей итерации.

Это означает, что при ссылке на элемент counted Array с `field()` функцией внутри `where` условия **она возвращает массив с одним элементом**. Хотя это и не может быть интуитивно понятным, оно согласуется с идеями того, что псевдонимы массивов всегда возвращают коллекцию выбранных свойств. Приведем пример:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/stringArray[*]",
    "where": {
      "field": "Microsoft.Test/resourceType/stringArray[*]",
      "equals": "[field('Microsoft.Test/resourceType/stringArray[*]')]"
    }
  },
  "equals": 0
}
```

| Итерация | Значения выражений | `where` Результат оценки |
|:---|:---|:---|
| 1 | `Microsoft.Test/resourceType/stringArray[*]` => `"a"` </br>  `[field('Microsoft.Test/resourceType/stringArray[*]')]` => `[ "a" ]` | `false` |
| 2 | `Microsoft.Test/resourceType/stringArray[*]` => `"b"` </br>  `[field('Microsoft.Test/resourceType/stringArray[*]')]` => `[ "b" ]` | `false` |
| 3 | `Microsoft.Test/resourceType/stringArray[*]` => `"c"` </br>  `[field('Microsoft.Test/resourceType/stringArray[*]')]` => `[ "c" ]` | `false` |

Таким образом, при необходимости доступа к значению подсчитанного псевдонима массива с `field()` функцией, способ сделать это заключается в заключении его в оболочку с помощью `first()` функции-шаблона:

```json
{
  "count": {
    "field": "Microsoft.Test/resourceType/stringArray[*]",
    "where": {
      "field": "Microsoft.Test/resourceType/stringArray[*]",
      "equals": "[first(field('Microsoft.Test/resourceType/stringArray[*]'))]"
    }
  }
}
```

| Итерация | Значения выражений | `where` Результат оценки |
|:---|:---|:---|
| 1 | `Microsoft.Test/resourceType/stringArray[*]` => `"a"` </br>  `[first(field('Microsoft.Test/resourceType/stringArray[*]'))]` => `"a"` | `true` |
| 2 | `Microsoft.Test/resourceType/stringArray[*]` => `"b"` </br>  `[first(field('Microsoft.Test/resourceType/stringArray[*]'))]` => `"b"` | `true` |
| 3 | `Microsoft.Test/resourceType/stringArray[*]` => `"c"` </br>  `[first(field('Microsoft.Test/resourceType/stringArray[*]'))]` => `"c"` | `true` |

Полезные примеры см. в разделе [примеры подсчета полей](../concepts/definition-structure.md#field-count-examples).

## <a name="modifying-arrays"></a>Изменение массивов

Свойства [append](../concepts/effects.md#append) и [Modify](../concepts/effects.md#modify) ALTER для ресурса во время создания или обновления. При работе со свойствами массива поведение этих эффектов зависит от того, пытается ли операция изменить  **\[\*\]** псевдоним.

> [!NOTE]
> Использование этого `modify` действия с псевдонимами в настоящее время находится на **этапе предварительной версии**.

|Псевдоним |Действие | Результат |
|-|-|-|
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules` | `append` | Политика Azure добавляет весь массив, указанный в сведениях о результате, если он отсутствует. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules` | `modify` с `add` операцией | Политика Azure добавляет весь массив, указанный в сведениях о результате, если он отсутствует. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules` | `modify` с `addOrReplace` операцией | Политика Azure добавляет весь массив, указанный в сведениях о результате, если он отсутствует или заменяет существующий массив. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]` | `append` | Политика Azure добавляет элемент массива, указанный в сведениях о результате. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]` | `modify` с `add` операцией | Политика Azure добавляет элемент массива, указанный в сведениях о результате. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]` | `modify` с `addOrReplace` операцией | Политика Azure удаляет все существующие члены массива и добавляет элемент массива, указанный в сведениях о результате. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].action` | `append` | Политика Azure добавляет значение к `action` свойству каждого члена массива. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].action` | `modify` с `add` операцией | Политика Azure добавляет значение к `action` свойству каждого члена массива. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].action` | `modify` с `addOrReplace` операцией | Политика Azure добавляет или заменяет существующее `action` свойство каждого члена массива. |

Дополнительные сведения см. в [примерах добавления](../concepts/effects.md#append-examples).

## <a name="appendix--additional--alias-examples"></a>Приложение. Дополнительные примеры псевдонимов [*]

Рекомендуется использовать [выражения количества полей](#field-count-expressions) , чтобы проверить, удовлетворяет ли условие "все" или "любой из" элементов массива в содержимом запроса. Однако для некоторых простых условий можно добиться того же результата с помощью метода доступа к полю с псевдонимом массива (как описано в разделе [ссылки на коллекцию Members](#referencing-the-array-members-collection)). Это может быть полезно в правилах политики, превышающих предел допустимого **количества** выражений. Ниже приведены примеры распространенных вариантов использования.

Вот пример правила политики для приведенной ниже таблицы сценариев:

```json
"policyRule": {
    "if": {
        "allOf": [
            {
                "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules",
                "exists": "true"
            },
            <-- Condition (see table below) -->
        ]
    },
    "then": {
        "effect": "[parameters('effectType')]"
    }
}
```

Массив **ipRules** для приведенной ниже таблицы сценариев выглядит так:

```json
"ipRules": [
    {
        "value": "127.0.0.1",
        "action": "Allow"
    },
    {
        "value": "192.168.1.1",
        "action": "Allow"
    }
]
```

Для каждого примера условия ниже замените `<field>` на `"field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].value"`.

Ниже приведены результаты сочетания условия с примером правила политики и массивом существующих значений, приведенными выше.

|Условие |Результат | Сценарий |Объяснение |
|-|-|-|-|
|`{<field>,"notEquals":"127.0.0.1"}` |Ничего |Ни одного совпадения |Один элемент массива вычисляется как false (127.0.0.1 != 127.0.0.1), а другой — как true (127.0.0.1 != 192.168.1.1), поэтому условие **notEquals** имеет значение _false_ и действие не активируется. |
|`{<field>,"notEquals":"10.0.4.1"}` |Эффект политики |Ни одного совпадения |Оба элемента массива вычисляются как true (10.0.4.1 != 127.0.0.1 и 10.0.4.1 != 192.168.1.1), поэтому условие **notEquals** имеет значение _true_ и действие активируется. |
|`"not":{<field>,"notEquals":"127.0.0.1" }` |Эффект политики |Одно или несколько совпадений |Один элемент массива вычисляется как false (127.0.0.1 != 127.0.0.1), а другой — как true (127.0.0.1 != 192.168.1.1), поэтому условие **notEquals** имеет значение _false_. Логический оператор вычисляется как true (**не** _false_), поэтому действие активируется. |
|`"not":{<field>,"notEquals":"10.0.4.1"}` |Ничего |Одно или несколько совпадений |Оба элемента массива вычисляются как true (10.0.4.1 != 127.0.0.1 и 10.0.4.1 != 192.168.1.1), поэтому условие **notEquals** имеет значение _true_. Логический оператор вычисляется как false (**не** _true_), поэтому действие не активируется. |
|`"not":{<field>,"Equals":"127.0.0.1"}` |Эффект политики |Не все совпадают |Один элемент массива вычисляется как true (127.0.0.1 == 127.0.0.1), а другой — как false (127.0.0.1 == 192.168.1.1), поэтому условие **Equals** имеет значение _false_. Логический оператор вычисляется как true (**не** _false_), поэтому действие активируется. |
|`"not":{<field>,"Equals":"10.0.4.1"}` |Эффект политики |Не все совпадают |Оба элемента массива вычисляются как false (10.0.4.1 == 127.0.0.1 и 10.0.4.1 == 192.168.1.1), поэтому условие **Equals** имеет значение _false_. Логический оператор вычисляется как true (**не** _false_), поэтому действие активируется. |
|`{<field>,"Equals":"127.0.0.1"}` |Ничего |Совпадают все |Один элемент массива вычисляется как true (127.0.0.1 == 127.0.0.1), а другой — как false (127.0.0.1 == 192.168.1.1), поэтому условие **Equals** имеет значение _false_ и действие не активируется. |
|`{<field>,"Equals":"10.0.4.1"}` |Ничего |Совпадают все |Оба элемента массива вычисляются как false (10.0.4.1 == 127.0.0.1 и 10.0.4.1 == 192.168.1.1), поэтому условие **Equals** имеет значение _false_ и действие не активируется. |

## <a name="next-steps"></a>Дальнейшие действия

- Рассмотрите [примеры для службы "Политика Azure"](../samples/index.md).
- Изучите статью о [структуре определения Политики Azure](../concepts/definition-structure.md).
- Изучите [сведения о действии политик](../concepts/effects.md).
- Узнайте о [программном создании политик](programmatically-create.md).
- Узнайте, как [исправлять несоответствующие ресурсы](remediate-resources.md).
- Дополнительные сведения о группе управления см. в статье [Упорядочивание ресурсов с помощью групп управления Azure](../../management-groups/overview.md).
