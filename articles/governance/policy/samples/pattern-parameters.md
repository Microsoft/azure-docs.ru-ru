---
title: Шаблон. Параметры в определении политики
description: Этот шаблон Политики Azure содержит пример использования параметров строки и массива в определении политики, а также способ параметризации результата.
ms.date: 03/31/2021
ms.topic: sample
ms.openlocfilehash: b742aaaf950e2b5670edbaa1f0134da144e675b6
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106092778"
---
# <a name="azure-policy-pattern-parameters"></a>Шаблон Политики Azure: параметры

Определение политики можно сделать динамическим, чтобы уменьшить количество определений политик, необходимых для использования [параметров](../concepts/definition-structure.md#parameters). Параметр определяется во время назначения политики. Параметры имеют набор предварительно определенных свойств, описывающих параметр и его использование.

## <a name="sample-1-string-parameters"></a>Пример 1 Параметры строки

Это определение политики использует два параметра: **tagName** и **tagValue**, чтобы задать, что именно будет искать назначение политики на ресурсах. Этот формат позволяет использовать определение политики для любого количества сочетаний имен тегов и значений тегов, но поддерживает только одно определение политики.

> [!NOTE]
> Пример тега, в котором используется **режим** _Все_ и который работает с группой ресурсов, см. в разделе [Шаблон: Теги — пример 1](./pattern-tags.md#sample-1-parameterize-tags).

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-1.json":::

### <a name="sample-1-explanation"></a>Пример 1 Объяснение

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-1.json" range="8-13":::

В этой части определения политики параметр **tagName** определен как _строковый_ и для его использования предоставляется описание.

Затем параметр используется в блоке **policyRule.if**, чтобы сделать политику динамической. Здесь оно используется для определения вычисляемого поля, которое представляет собой тег со значением **tagName**.

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-1.json" range="22-27" highlight="3":::

## <a name="sample-2-array-parameters"></a>Пример 2 Параметры массива

Это определение политики использует один параметр, **listOfBandwidthinMbps**, чтобы проверить, настроено ли для ресурса контура Express Route параметр пропускной способности на одно из утвержденных значений. Если оно не совпадает, то создание или обновление ресурса [отклоняется](../concepts/effects.md#deny)

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-2.json":::

### <a name="sample-2-explanation"></a>Пример 2 Объяснение

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-2.json" range="6-12":::

В этой части определения политики параметр **listOfBandwidthinMbps** определяется как _массив_ и для его использования предоставляется описание. В качестве _массива_ он имеет несколько значений для сопоставления.

Затем параметр используется в блоке **policyRule.if**. В качестве параметра _массива_ следует использовать 
[условия](../concepts/definition-structure.md#conditions) _массива_ **in** или **notIn**.
Здесь он используется для псевдонима **serviceProvider.bandwidthInMbps** как одно из определенных значений.

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-2.json" range="21-24" highlight="3":::

## <a name="sample-3-parameterized-effect"></a>Пример 3. Параметризованный effect

Распространенным способом обеспечения многократного использования определений политики является параметризация самого effect. В этом примере используется один параметр, **effect**. Параметризация effect позволяет присвоить одно и то же определение разным областям с разными effect.

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-3.json":::

### <a name="sample-3-explanation"></a>Пример 3. Объяснение

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-3.json" range="11-25":::

В этой части определения политики параметр **effect** определяется как _строка_. Определение политики устанавливает для назначения значение по умолчанию _аудит_ и ограничивает остальные параметры значениями _отключено_ и _запретить_.

Затем параметр используется в блоке **policyRule.if** для _effect_.

:::code language="json" source="~/policy-templates/patterns/pattern-parameters-3.json" range="38-40" highlight="2":::

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с другими [шаблонами и встроенными определениями](./index.md).
- Изучите статью о [структуре определения Политики Azure](../concepts/definition-structure.md).
- Изучите [сведения о действии политик](../concepts/effects.md).