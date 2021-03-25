---
title: Шаблон. Определения групповой политики с инициативами
description: Этот шаблон Политики Azure содержит пример группирования определений политики в инициативе.
ms.date: 10/14/2020
ms.topic: sample
ms.openlocfilehash: aa09cafe636a4665dba6a2e746c13b95ff304895
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92072923"
---
# <a name="azure-policy-pattern-group-policy-definitions"></a>Шаблон политики Azure: группирование определений политики

Инициатива — это группа определений политики. Группируя связанные определения политики в один объект, можно создать одно назначение, которое было бы несколькими назначениями.

## <a name="sample-initiative-definition"></a>Пример определения инициативы

Эта инициатива развертывает два определения политики, каждое из которых принимает параметры **tagName** и **tagValue**. Сама инициатива имеет два параметра: **costCenterValue** и **productNameValue**.
Эти параметры инициативы предоставляются каждому из сгруппированных определений политик. Такая конструкция максимально увеличивает повторное использование существующих определений политики, ограничивая при этом количество заданий, создаваемых для их реализации по мере необходимости.

:::code language="json" source="~/policy-templates/patterns/pattern-group-with-initiative.json":::

### <a name="explanation"></a>Объяснение

#### <a name="initiative-parameters"></a>Параметры инициативы

Инициатива может определять собственные параметры, которые затем передаются в определения групповых политик.
В этом примере как **costCenterValue**, так и **productNameValue** определены как параметры инициативы. Значения предоставляются при назначении инициативы.

:::code language="json" source="~/policy-templates/patterns/pattern-group-with-initiative.json" range="5-18":::

#### <a name="includes-policy-definitions"></a>Включает определения политик

Каждое включаемое определение политики должно предоставлять массивы **policyDefinitionId** и **Параметры**, если определение политики принимает параметры. В приведенном ниже фрагменте определения включаемой политики принимается два параметра: **tagName** и **tagValue**. **tagName** определяется с помощью литерала, но **tagValue** использует параметр **costCenterValue**, определенный инициативой. Эта сквозная передача значений улучшает повторное использование.

:::code language="json" source="~/policy-templates/patterns/pattern-group-with-initiative.json" range="30-40":::

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с другими [шаблонами и встроенными определениями](./index.md).
- Изучите статью о [структуре определения Политики Azure](../concepts/definition-structure.md).
- Изучите [сведения о действии политик](../concepts/effects.md).