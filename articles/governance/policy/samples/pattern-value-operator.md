---
title: Шаблон. Оператор value в определении политики
description: Этот шаблон Политики Azure предоставляет пример использования оператора value в определении политики.
ms.date: 03/31/2021
ms.topic: sample
ms.openlocfilehash: 560f128dc5f78ca2335f2712e7fd81bd94eda761
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106092805"
---
# <a name="azure-policy-pattern-the-value-operator"></a>Шаблон Политики Azure: оператор value

Оператор [value](../concepts/definition-structure.md#value) выражает [параметры](../concepts/definition-structure.md#parameters), [поддерживаемые функции шаблонов](../concepts/definition-structure.md#policy-functions) или литералы в указанном значении для заданного [условия](../concepts/definition-structure.md#conditions).

> [!WARNING]
> Если результатом _функции шаблона_ является ошибка, оценка политики завершится сбоем. Завершившаяся сбоем оценка — это неявный **запрет**. Дополнительные сведения см. в разделе [Аvoiding template failures](../concepts/definition-structure.md#avoiding-template-failures) (Предотвращение сбоев шаблонов).

## <a name="sample-policy-definition"></a>Пример определения политики

Это определение политики добавляет или заменяет тег, указанный в параметре **tagName** (_строка_) для ресурсов, и наследует значение **tagName** из группы ресурсов, в которой находится ресурс. Эта оценка происходит при создании или обновлении ресурса. Как и в действии [modify](../concepts/effects.md#modify), в существующих ресурсах исправление можно запустить с помощью [задачи исправления](../how-to/remediate-resources.md).

:::code language="json" source="~/policy-templates/patterns/pattern-value-operator.json":::

### <a name="explanation"></a>Объяснение

:::code language="json" source="~/policy-templates/patterns/pattern-value-operator.json" range="20-30" highlight="7,8":::

Оператор **value** используется в блоке **policyRule.if** в **свойствах**. В данном примере [логический оператор](../concepts/definition-structure.md#logical-operators) **allOf** используется для указания, что для выполнения действия **modify** оба условных оператора должны иметь значение true.

**value** преобразовывает результат функции шаблона [resourceGroup()](../../../azure-resource-manager/templates/template-functions-resource.md#resourcegroup) в условие **notEquals** пустого значения. Если имя тега, указанное в **tagName** в родительской группе ресурсов, существует, то условие вычисляется как true.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с другими [шаблонами и встроенными определениями](./index.md).
- Изучите статью о [структуре определения Политики Azure](../concepts/definition-structure.md).
- Изучите [сведения о действии политик](../concepts/effects.md).