---
title: Шаблон. Использование тегов в определении политики
description: Этот шаблон Политики Azure содержит примеры добавления параметризованных тегов или наследования тегов из группы ресурсов в определении политики.
ms.date: 03/31/2021
ms.topic: sample
ms.openlocfilehash: c748eb9b8ea795f9725082ec0aa0b8065ada8c65
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106093390"
---
# <a name="azure-policy-pattern-tags"></a>Шаблон Политики Azure: теги

[Теги](../../..//azure-resource-manager/management/tag-resources.md) — это важная часть управления ресурсами Azure и их упорядочивания и контроля. Политика Azure позволяет настроить теги для новых и существующих ресурсов в большом масштабе с помощью действия [modify](../concepts/effects.md#modify) и [задач исправления](../how-to/remediate-resources.md).

## <a name="sample-1-parameterize-tags"></a>Пример 1. Параметризация тегов

Это определение политики использует два параметра: **tagName** и **tagValue**. Эти параметры указывают, что именно будет искать назначение политики в группах ресурсов. Этот формат позволяет использовать определение политики для любого количества сочетаний имен тегов и значений тегов, но поддерживает только одно определение политики.

> [!NOTE]
> Хотя этот шаблон определения политики похож на шаблон из раздела [Пример 1: параметры строки](./pattern-parameters.md#sample-1-string-parameters), в этом примере параметр **mode** имеет значение _All_ и нацелен на группы ресурсов.

:::code language="json" source="~/policy-templates/patterns/pattern-tags-1.json":::

### <a name="sample-1-explanation"></a>Пример 1. Объяснение

:::code language="json" source="~/policy-templates/patterns/pattern-tags-1.json" range="2-8" highlight="3":::

В этом примере параметр **mode** имеет значение _All_, так как он нацелен на группу ресурсов. В большинстве случаев при работе с тегами параметр **mode** должен иметь значение _Indexed_. Дополнительные сведения см. в разделе о [режимах](../concepts/definition-structure.md#resource-manager-modes).

:::code language="json" source="~/policy-templates/patterns/pattern-tags-1.json" range="26-36" highlight="7-8":::

В этой части определения политики `concat` объединяет параметризованный параметр **tagName** и формат `tags['name']`, чтобы сообщить **field** произвести оценку этого тега для параметра **tagValue**.
Поскольку используется **notEquals**, если значение **tags\[tagName\]** не равно **tagValue**, активируется действие **modify**.

:::code language="json" source="~/policy-templates/patterns/pattern-tags-1.json" range="43-47" highlight="3-4":::

Здесь операция **addOrReplace** использует тот же формат значений параметризованного тега для создания тега и обновления его значения до требуемого в оцениваемой группе ресурсов.

## <a name="sample-2-inherit-tag-value-from-resource-group"></a>Пример 2. Наследование значения тега из группы ресурсов

Это определение политики использует параметр **tagName**, чтобы определить значение тега, которое следует унаследовать от родительской группы ресурсов.

:::code language="json" source="~/policy-templates/patterns/pattern-tags-2.json":::

### <a name="sample-2-explanation"></a>Пример 2. Объяснение

:::code language="json" source="~/policy-templates/patterns/pattern-tags-2.json" range="2-8" highlight="3":::

В этом примере параметр **mode** имеет значение _Indexed_, так как он не нацелен на группу ресурсов или подписку, несмотря на то, что получает значение из группы ресурсов. Дополнительные сведения см. в разделе о [режимах](../concepts/definition-structure.md#resource-manager-modes).

:::code language="json" source="~/policy-templates/patterns/pattern-tags-2.json" range="19-29" highlight="3-4,7-8":::

**policyRule.if** использует `concat`, как в [примере 1](#sample-1-parameterize-tags), для вычисления значения **tagName**, а также использует функцию `resourceGroup()` для сравнения этого значения со значением того же тега в родительской группе ресурсов. Второе предложение проверяет, имеет ли тег в группе ресурсов значение, отличное от NULL.

:::code language="json" source="~/policy-templates/patterns/pattern-tags-2.json" range="36-40" highlight="3-4":::

В этой части значение, присваиваемое тегу **tagName** в ресурсе, также использует функцию `resourceGroup()` для получения значения из родительской группы ресурсов. Таким образом можно наследовать теги от родительских групп ресурсов. Если вы уже создали ресурс, но еще не добавили тег, это же определение политики и [задача исправления](../how-to/remediate-resources.md) могут обновлять существующие ресурсы.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с другими [шаблонами и встроенными определениями](./index.md).
- Изучите статью о [структуре определения Политики Azure](../concepts/definition-structure.md).
- Изучите [сведения о действии политик](../concepts/effects.md).