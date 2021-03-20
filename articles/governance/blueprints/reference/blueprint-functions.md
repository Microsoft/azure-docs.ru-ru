---
title: Функции Azure Blueprints
description: Описание функций, доступных для использования с артефактами схемы в определениях и назначениях Azure Blueprints.
ms.date: 01/27/2021
ms.topic: reference
ms.openlocfilehash: 92cb906e87179073b7a69aa0bd4eab22c77087f9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98919280"
---
# <a name="functions-for-use-with-azure-blueprints"></a>Функции, доступные для использования с Azure Blueprints

Azure Blueprints предоставляют функции, которые делают определение схемы более динамичным. Эти функции предназначены для использования с определениями и артефактами схем. Артефакт шаблона Azure Resource Manager (шаблона ARM) поддерживает полное использование функций диспетчер ресурсов, помимо получения динамического значения с помощью параметра схемы.

Поддерживаются следующие функции:

- [artifacts](#artifacts)
- [concat](#concat)
- [parameters](#parameters)
- [resourceGroup](#resourcegroup)
- [resourceGroups](#resourcegroups)
- [subscription](#subscription)

## <a name="artifacts"></a>artifacts

`artifacts(artifactName)`

Возвращает объект свойств, заполненных выходными данными выбранных артефактов схемы.

> [!NOTE]
> `artifacts()`Функция не может использоваться внутри шаблона ARM. Эту функцию можно использовать только в JSON определения схемы или артефакта для управления схемой с помощью Azure PowerShell или REST API в режиме [схема как код](https://github.com/Azure/azure-blueprints/blob/master/README.md).

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| artifactName |Да |строка |Имя артефакта схемы. |

### <a name="return-value"></a>Возвращаемое значение

Объект выходных свойств. Свойства **outputs** зависят от типа артефакта схемы, на который указывает ссылка. Все типы соблюдают следующий формат:

```json
{
  "outputs": {collectionOfOutputProperties}
}
```

#### <a name="policy-assignment-artifact"></a>Артефакт назначения политики

```json
{
    "outputs": {
        "policyAssignmentId": "{resourceId-of-policy-assignment}",
        "policyAssignmentName": "{name-of-policy-assignment}",
        "policyDefinitionId": "{resourceId-of-policy-definition}",
    }
}
```

#### <a name="arm-template-artifact"></a>Артефакт шаблона ARM

Свойства **выходных** данных возвращаемого объекта определяются в шаблоне ARM и возвращаются развертыванием.

#### <a name="role-assignment-artifact"></a>Артефакт назначения ролей

```json
{
    "outputs": {
        "roleAssignmentId": "{resourceId-of-role-assignment}",
        "roleDefinitionId": "{resourceId-of-role-definition}",
        "principalId": "{principalId-role-is-being-assigned-to}",
    }
}
```

### <a name="example"></a>Пример

Артефакт шаблона ARM с ИДЕНТИФИКАТОРом _митемплатеартифакт_ , содержащий следующий пример выходного свойства:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    ...
    "outputs": {
        "myArray": {
            "type": "array",
            "value": ["first", "second"]
        },
        "myString": {
            "type": "string",
            "value": "my string value"
        },
        "myObject": {
            "type": "object",
            "value": {
                "myProperty": "my value",
                "anotherProperty": true
            }
        }
    }
}
```

Ниже приводятся несколько примеров получения данных из примера _myTemplateArtifact_.

| Выражение | Тип | Значение |
|:---|:---|:---|
|`[artifacts("myTemplateArtifact").outputs.myArray]` | Array | \["first", "second"\] |
|`[artifacts("myTemplateArtifact").outputs.myArray[0]]` | Строка | "first" |
|`[artifacts("myTemplateArtifact").outputs.myString]` | Строка | "my string value" |
|`[artifacts("myTemplateArtifact").outputs.myObject]` | Объект | { "myproperty": "my value", "anotherProperty": true } |
|`[artifacts("myTemplateArtifact").outputs.myObject.myProperty]` | Строка | "my value" |
|`[artifacts("myTemplateArtifact").outputs.myObject.anotherProperty]` | Bool | True |

## <a name="concat"></a>concat

`concat(string1, string2, string3, ...)`

Объединяет несколько строковых значений и возвращает результирующую строку.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| строка1 |Да |строка |Первое значение для сцепки. |
| дополнительные аргументы |нет |строка |Упорядоченные в последовательность дополнительные значения для объединения |

### <a name="return-value"></a>Возвращаемое значение

Строка объединенных значений.

### <a name="remarks"></a>Remarks

Функция Azure Blueprint отличается от функции шаблона ARM тем, что она работает только со строками.

### <a name="example"></a>Пример

`concat(parameters('organizationName'), '-vm')`

## <a name="parameters"></a>параметры

`parameters(parameterName)`

Возвращает значение параметра схемы. Указанное имя параметра должно быть определено в разделе определении схемы или в артефактах схемы.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| parameterName |Да |строка |Имя параметра, который требуется вернуть. |

### <a name="return-value"></a>Возвращаемое значение

Значение указанного параметра схемы или артефакта схемы.

### <a name="remarks"></a>Remarks

Функция Azure Blueprint отличается от функции шаблона ARM тем, что она работает только с параметрами схемы.

### <a name="example"></a>Пример

Определите параметр _principalIds_ в определении схемы:

```json
{
    "type": "Microsoft.Blueprint/blueprints",
    "properties": {
        ...
        "parameters": {
            "principalIds": {
                "type": "array",
                "metadata": {
                    "displayName": "Principal IDs",
                    "description": "This is a blueprint parameter that any artifact can reference. We'll display these descriptions for you in the info bubble. Supply principal IDs for the users,groups, or service principals for the Azure role assignment.",
                    "strongType": "PrincipalId"
                }
            }
        },
        ...
    }
}
```

Затем укажите _principalIds_ в качестве аргумента для `parameters()` в артефакте схемы:

```json
{
    "type": "Microsoft.Blueprint/blueprints/artifacts",
    "kind": "roleAssignment",
    ...
    "properties": {
        "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
        "principalIds": "[parameters('principalIds')]",
        ...
    }
}
```

## <a name="resourcegroup"></a>resourceGroup

`resourceGroup()`

Возвращает объект, который представляет текущую группу ресурсов.

### <a name="return-value"></a>Возвращаемое значение

Возвращаемый объект имеет следующий формат:

```json
{
  "name": "{resourceGroupName}",
  "location": "{resourceGroupLocation}",
}
```

### <a name="remarks"></a>Remarks

Функция Azure Blueprint отличается от функции шаблона ARM. Функцию `resourceGroup()` нельзя использовать в артефакте уровня подписки или определении схемы. Ее можно использовать только в артефактах схемы, которые входят в артефакт группы ресурсов.

Как правило, функция `resourceGroup()` используется для создания ресурсов в одном расположении как артефакт группы ресурсов.

### <a name="example"></a>Пример

Чтобы указать расположение группы ресурсов, заданное в определении схемы или во время назначения, как расположение для другого артефакта, объявите в определении схемы объект-заполнитель для группы ресурсов. В этом примере используется заполнитель группы ресурсов _NetworkingPlaceholder_.

```json
{
    "type": "Microsoft.Blueprint/blueprints",
    "properties": {
        ...
        "resourceGroups": {
            "NetworkingPlaceholder": {
                "location": "eastus"
            }
        }
    }
}
```

Теперь примените функцию `resourceGroup()` в контексте артефакта схемы, который предназначен для объекта-заполнителя группы ресурсов. Наш пример развертывает артефакт шаблона в группе ресурсов _NetworkingPlaceholder_ и предоставляет параметр _resourceLocation_, который динамически заполняется расположением группы ресурсов _NetworkingPlaceholder_ для шаблона. Расположение группы ресурсов _NetworkingPlaceholder_ в такой ситуации можно определять статически в определении схемы или динамически во время назначения. В любом случае артефакт шаблона получает эту информацию в виде параметра и применяет ее для развертывания ресурсов в правильном расположении.

```json
{
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "kind": "template",
  "properties": {
      "template": {
        ...
      },
      "resourceGroup": "NetworkingPlaceholder",
      ...
      "parameters": {
        "resourceLocation": {
          "value": "[resourceGroup().location]"
        }
      }
  }
}
```

## <a name="resourcegroups"></a>resourceGroups

`resourceGroups(placeholderName)`

Возвращает объект, который представляет артефакт для указанной группы ресурсов. В отличие от функции `resourceGroup()`, для которой требуется контекст артефакта, эта функция позволяет получить свойства определенного объекта-заполнителя группы ресурсов не из контекста этой группы ресурсов.

### <a name="parameters"></a>Параметры

| Параметр | Обязательно | Тип | Описание |
|:--- |:--- |:--- |:--- |
| placeholderName |Да |строка |Имя заполнителя для возвращаемого артефакта группы ресурсов. |

### <a name="return-value"></a>Возвращаемое значение

Возвращаемый объект имеет следующий формат:

```json
{
  "name": "{resourceGroupName}",
  "location": "{resourceGroupLocation}",
}
```

### <a name="example"></a>Пример

Чтобы указать расположение группы ресурсов, заданное в определении схемы или во время назначения, как расположение для другого артефакта, объявите в определении схемы объект-заполнитель для группы ресурсов. В этом примере используется заполнитель группы ресурсов _NetworkingPlaceholder_.

```json
{
    "type": "Microsoft.Blueprint/blueprints",
    "properties": {
        ...
        "resourceGroups": {
            "NetworkingPlaceholder": {
                "location": "eastus"
            }
        }
    }
}
```

Теперь примените функцию `resourceGroups()` в контексте любого артефакта схемы, чтобы получить ссылку на объект-заполнитель указанной группы ресурсов. Наш пример развертывает артефакт шаблона вне группы ресурсов _NetworkingPlaceholder_ и предоставляет параметр _artifactLocation_, который динамически заполняется расположением группы ресурсов _NetworkingPlaceholder_ для шаблона. Расположение группы ресурсов _NetworkingPlaceholder_ в такой ситуации можно определять статически в определении схемы или динамически во время назначения. В любом случае артефакт шаблона получает эту информацию в виде параметра и применяет ее для развертывания ресурсов в правильном расположении.

```json
{
  "kind": "template",
  "properties": {
      "template": {
          ...
      },
      ...
      "parameters": {
        "artifactLocation": {
          "value": "[resourceGroups('NetworkingPlaceholder').location]"
        }
      }
  },
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "name": "myTemplate"
}
```

## <a name="subscription"></a>Подписка

`subscription()`

Возвращает сведения о подписке для текущего назначения схемы.

### <a name="return-value"></a>Возвращаемое значение

Возвращаемый объект имеет следующий формат:

```json
{
    "id": "/subscriptions/{subscriptionId}",
    "subscriptionId": "{subscriptionId}",
    "tenantId": "{tenantId}",
    "displayName": "{name-of-subscription}"
}
```

### <a name="example"></a>Пример

Используйте отображаемое имя подписки и функцию `concat()`, чтобы создать соглашение об именовании, которое передается в качестве параметра _resourceName_ в артефакт шаблона.

```json
{
  "kind": "template",
  "properties": {
      "template": {
          ...
      },
      ...
      "parameters": {
        "resourceName": {
          "value": "[concat(subscription().displayName, '-vm')]"
        }
      }
  },
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "name": "myTemplate"
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь со сведениями о [жизненном цикле схем](../concepts/lifecycle.md).
- Узнайте, как использовать [статические и динамические параметры](../concepts/parameters.md).
- Научитесь настраивать [последовательность схемы](../concepts/sequencing-order.md).
- Узнайте, как применять [блокировку ресурсов схемы](../concepts/resource-locking.md).
- Узнайте, как [обновлять существующие назначения](../how-to/update-existing-assignments.md).
- Устраняйте проблемы, возникающие во время назначения схемы, с помощью [общих инструкций по устранению неполадок](../troubleshoot/general.md).