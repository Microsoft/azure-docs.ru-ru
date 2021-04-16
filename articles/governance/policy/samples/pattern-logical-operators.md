---
title: Шаблон. Логические операторы в определении политики
description: Этот шаблон Политики Azure предоставляет пример использования логических операторов в определении политики.
ms.date: 03/31/2021
ms.topic: sample
ms.openlocfilehash: feb9e50b0c73c19027b747cf0f95fa1cb6fbd47c
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106093356"
---
# <a name="azure-policy-pattern-logical-operators"></a>Шаблон политики Azure: логические операторы

Определение политики может содержать несколько условных операторов. Вам может понадобиться, чтобы каждое утверждение имело значение "true", или чтобы некоторые из них были "true". Для поддержки этих потребностей язык имеет [логические операторы](../concepts/definition-structure.md#logical-operators) для **not**, **allOf** и **anyOf**. Они необязательны и могут быть вложены для создания сложных сценариев.

## <a name="sample-1-one-logical-operator"></a>Пример 1 Один логический оператор

Это определение политики оценивает учетные записи [Azure Cosmos DB](../../../cosmos-db/introduction.md), чтобы узнать, настроены ли автоматические переходы на другой ресурс и несколько расположений для записи. Если нет, выполняется [аудит](../concepts/effects.md#audit), который создает запись в журнале при создании или обновлении несоответствующего ресурса.

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-1.json":::

### <a name="sample-1-explanation"></a>Пример 1 Объяснение

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-1.json" range="6-22" highlight="3":::

Блок **policyRule.if** использует один **allOf**, чтобы убедиться, что все три условия имеют значение "true".
Действие **audit** срабатывает, только если все эти условия имеют значение "true".

## <a name="sample-2-multiple-logical-operators"></a>Пример 2 Несколько логических операторов

Это определение политики оценивает ресурсы для шаблона именования. Если ресурс не соответствует, он [отклоняется](../concepts/effects.md#deny).

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-2.json":::

### <a name="sample-2-explanation"></a>Пример 2 Объяснение

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-2.json" range="7-21" highlight="2,3,9":::

Этот блок **policyRule.if** также включает один **allOf**, но каждое условие заключено логическим оператором **not**. Сначала вычисляется условие внутри логического оператора **not**, а затем вычисляется значение **not**, чтобы определить, является целое предложение "true" или "false". Если оба логические операторы **not** имеют значение "true", то действие политики срабатывает.

## <a name="sample-3-combining-logical-operators"></a>Пример 3. Объединение логических операторов

Это определение политики проверяет, включена ли трассировка в учетных записях [Spring on Azure](/azure/developer/java/spring-framework) и находится ли она в рабочем состоянии.

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-3.json":::

### <a name="sample-3-explanation"></a>Пример 3. Объяснение

:::code language="json" source="~/policy-templates/patterns/pattern-logical-operators-3.json" range="6-28" highlight="3,8":::

Этот блок **policyRule.if** содержит оба логических оператора — **allOf** и **anyOf**. Оператор **anyOf** имеет значение "true", если любое из его условий имеет значение "true". Поскольку _type_ лежит в основе **allOf**, он должен всегда иметь значение "true". Если _type_ и одно из условий **anyOf** имеют значение "true", то действие политики срабатывает.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с другими [шаблонами и встроенными определениями](./index.md).
- Изучите статью о [структуре определения Политики Azure](../concepts/definition-structure.md).
- Изучите [сведения о действии политик](../concepts/effects.md).