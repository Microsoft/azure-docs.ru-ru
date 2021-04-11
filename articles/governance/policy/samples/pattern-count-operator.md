---
title: Шаблон. Оператор count в определении политики
description: Этот шаблон Политики Azure предоставляет пример использования оператора count в определении политики.
ms.date: 03/31/2021
ms.topic: sample
ms.openlocfilehash: dc2914028887ae5a91e3379e2a94ddbc57a7cef3
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106093458"
---
# <a name="azure-policy-pattern-the-count-operator"></a>Шаблон Политики Azure: оператор count

Оператор [count](../concepts/definition-structure.md#count) определяет участников псевдонима \[\*\].

## <a name="sample-policy-definition"></a>Пример определения политики

Это определение политики [проверяет](../concepts/effects.md#audit) группы безопасности сети, настроенные на разрешение входящего трафика протокола удаленного рабочего стола (RDP).

:::code language="json" source="~/policy-templates/patterns/pattern-count-operator.json":::

### <a name="explanation"></a>Объяснение

Основными компонентами оператора **count** являются _field_, _where_, и условие. Каждый из них выделяется в приведенном ниже фрагменте кода.

- _field_ указывает количество [псевдонимов](../concepts/definition-structure.md#aliases) для определения участников. Здесь мы рассмотрим _массив_ псевдонимов **securityRules\[\*\]** группы безопасности сети.
- _where_ использует язык политики, чтобы определить, какие элементы _массива_ соответствуют критериям. В этом примере логический оператор **allOf** группирует три разных определения псевдонима для свойств _массива_: _направление_, _доступ_ и _destinationPortRange_.
- Условие подсчета в этом примере **больше**. Функция Count определяет значение "true", если один или несколько членов псевдонима _массива_ совпадают с предложением _where_.

:::code language="json" source="~/policy-templates/patterns/pattern-count-operator.json" range="12-32" highlight="3,4,20":::

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с другими [шаблонами и встроенными определениями](./index.md).
- Изучите статью о [структуре определения Политики Azure](../concepts/definition-structure.md).
- Изучите [сведения о действии политик](../concepts/effects.md).