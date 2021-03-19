---
title: Элемент пользовательского интерфейса Тагсбиресаурце
description: Описывает элемент пользовательского интерфейса Microsoft. Common. Тагсбиресаурце для портал Azure. Используйте для применения тегов к ресурсу во время развертывания.
author: tfitzmac
ms.topic: conceptual
ms.date: 11/11/2019
ms.author: tomfitz
ms.openlocfilehash: e730201812005a9b469a964e4acd081ebe86b100
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87063938"
---
# <a name="microsoftcommontagsbyresource-ui-element"></a>Элемент пользовательского интерфейса Microsoft. Common. Тагсбиресаурце

Элемент управления для связывания [тегов](../management/tag-resources.md) с ресурсами в развертывании.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

![Элемент пользовательского интерфейса Microsoft.Common.DropDown](./media/managed-application-elements/microsoft-common-tagsbyresource.png)

## <a name="schema"></a>схема

```json
{
  "name": "element1",
  "type": "Microsoft.Common.TagsByResource",
  "resources": [
    "Microsoft.Storage/storageAccounts",
    "Microsoft.Compute/virtualMachines"
  ]
}
```

## <a name="sample-output"></a>Пример выходных данных

```json
{
  "Microsoft.Storage/storageAccounts": {
    "Dept": "Finance",
    "Environment": "Production"
  },
  "Microsoft.Compute/virtualMachines": {
    "Dept": "Finance"
  }
}
```

## <a name="remarks"></a>Комментарии

- Необходимо указать хотя бы один элемент в `resources` массиве.
- Каждый элемент в `resources` должен быть полным типом ресурса. Эти элементы отображаются в раскрывающемся списке **ресурсов** и таггабле пользователем.
- Выходные данные элемента управления форматируются для простого присваивания значений тегов в шаблоне Azure Resource Manager. Чтобы получить выходные данные элемента управления в шаблоне, включите параметр в шаблон, как показано в следующем примере:

  ```json
  "parameters": {
    "tagsByResource": { "type": "object", "defaultValue": {} }
  }
  ```

  Для каждого ресурса, который может быть помечен, присвойте свойству Tags значение параметра для этого типа ресурса:

  ```json
  {
    "name": "saName1",
    "type": "Microsoft.Storage/storageAccounts",
    "tags": "[ if(contains(parameters('tagsByResource'), 'Microsoft.Storage/storageAccounts'), parameters('tagsByResource')['Microsoft.Storage/storageAccounts'], json('{}')) ]",
    ...
  ```

- Используйте функцию [If](../templates/template-functions-logical.md#if) при доступе к параметру тагсбиресаурце. Он позволяет назначить пустой объект, если ни один из тегов не назначен данному типу ресурса.

## <a name="next-steps"></a>Дальнейшие действия

- Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
- Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
