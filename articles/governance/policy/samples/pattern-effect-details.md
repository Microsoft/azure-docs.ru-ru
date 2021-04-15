---
title: Шаблон. Действия определения политики
description: Этот шаблон Политики Azure предоставляет пример использования разных действий с помощью определения политики.
ms.date: 03/31/2021
ms.topic: sample
ms.openlocfilehash: 9fa95fdc793e7762c39525a83f38ae952f532a60
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106092846"
---
# <a name="azure-policy-pattern-effects"></a>Шаблон Политики Azure: действия

В Политике Azure есть много [действий](../concepts/effects.md), определяющих реакцию службы на ресурсы, не соответствующие требованиям. Некоторые действия являются простыми и не требуют дополнительных свойств в определении политики, а другие требуют несколько свойств.

## <a name="sample-1-simple-effect"></a>Пример 1 Простое действие

Это определение политики проверяет, существует ли тег, определенный в параметре **tagName**, в вычисляемом ресурсе. Если тег еще не существует, срабатывает действие [modify](../concepts/effects.md#modify) для добавления тега со значением в параметре **tagValue**.

:::code language="json" source="~/policy-templates/patterns/pattern-effect-details-1.json":::

### <a name="sample-1-explanation"></a>Пример 1 Объяснение

:::code language="json" source="~/policy-templates/patterns/pattern-effect-details-1.json" range="40-50":::

Действие **modify** требует блока **policyRule.then.details**, который определяет **roleDefinitionIds** и **операции**. Эти параметры информируют Политику Azure о том, какие роли необходимы для добавления тега и исправления ресурса, а также какую операцию **изменения** следует использовать. В нашем примере **операцией** является _добавление_, а ее параметры задают имя и значение тега.

## <a name="sample-2-complex-effect"></a>Пример 2 Сложное действие

Это определение политики выполняет аудит каждой виртуальной машины, если расширение, определенное в параметрах **издателя** и **типа**, не существует. Он использует [auditIfNotExists](../concepts/effects.md#auditifnotexists) для проверки ресурса, связанного с виртуальной машиной, чтобы узнать, существует ли экземпляр, соответствующий заданным параметрам. В этом примере проверяется тип **расширения**.

:::code language="json" source="~/policy-templates/patterns/pattern-effect-details-2.json":::

### <a name="sample-2-explanation"></a>Пример 2 Объяснение

:::code language="json" source="~/policy-templates/patterns/pattern-effect-details-2.json" range="45-58":::

Действие **auditIfNotExists** требует, чтобы блок **policyRule.then.details** определял для поиска как **тип**, так и **existenceCondition**. В **existenceCondition** используются элементы языка политики, такие как [логические операторы](../concepts/definition-structure.md#logical-operators), чтобы определить, существует ли соответствующий ресурс. В данном примере значения, проверенные с каждым [псевдонимом](../concepts/definition-structure.md#aliases), задаются в параметрах.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с другими [шаблонами и встроенными определениями](./index.md).
- Изучите статью о [структуре определения Политики Azure](../concepts/definition-structure.md).
- Изучите [сведения о действии политик](../concepts/effects.md).