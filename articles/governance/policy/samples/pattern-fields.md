---
title: Шаблон. Свойства поля в определении политики
description: Этот шаблон Политики Azure предоставляет пример использования свойств поля в определении политики.
ms.date: 10/14/2020
ms.topic: sample
ms.openlocfilehash: 267c687f78f0bbb100843faee40ab6f3d3cbb64c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92072974"
---
# <a name="azure-policy-pattern-field-properties"></a>Шаблон Политики Azure: свойства поля

Оператор [field](../concepts/definition-structure.md#fields) вычисляет указанное свойство или [псевдоним](../concepts/definition-structure.md#aliases) предоставленному значению для заданного [условия](../concepts/definition-structure.md#conditions).

## <a name="sample-policy-definition"></a>Пример определения политики

Это определение политики позволяет определить разрешенные регионы, соответствующие требованиям географического расположения вашей организации. Разрешенные ресурсы определяются в параметре **listOfAllowedLocations** (_массив_). Ресурсы, соответствующие определению, [отклоняются](../concepts/effects.md#deny).

:::code language="json" source="~/policy-templates/patterns/pattern-fields.json":::

### <a name="explanation"></a>Объяснение

:::code language="json" source="~/policy-templates/patterns/pattern-fields.json" range="18-36" highlight="3,7,11":::

Оператор **field** используется три раза в [логическом операторе](../concepts/definition-structure.md#logical-operators) **allOf**.

- При первом использовании вычисляется свойство `location` с условием **notIn** в параметре **listOfAllowedLocations**. **notIn** работает, как ожидает _массив_, и параметр является _массивом_. Если `location` созданного или обновленного ресурса не находится в списке утвержденных, этот элемент возвращает значение "true".
- Второе использование также оценивает свойство `location`, но использует условие **notEquals**, чтобы определить, является ли ресурс _глобальным_. Если `location` созданного или обновленного ресурса не _глобальное_, этот элемент возвращает значение "true".
- При последнем использовании вычисляется свойство `type` и используется условие **notEquals** для проверки, не является ли тип ресурса _Microsoft.AzureActiveDirectory/b2cDirectories_. Если это не так, этот элемент возвращает значение "true".

Если все три оператора условий в логическом операторе **allOf** оцениваются как "true", то создание или обновление ресурса блокируется Политикой Azure.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с другими [шаблонами и встроенными определениями](./index.md).
- Изучите статью о [структуре определения Политики Azure](../concepts/definition-structure.md).
- Изучите [сведения о действии политик](../concepts/effects.md).