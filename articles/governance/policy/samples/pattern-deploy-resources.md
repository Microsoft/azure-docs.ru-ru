---
title: Шаблон. Развертывание ресурсов с помощью определения политики
description: В этом шаблоне Политики Azure предоставляется пример развертывания ресурсов с помощью определения политики deployIfNotExists.
ms.date: 03/31/2021
ms.topic: sample
ms.openlocfilehash: 1dda3d5f2d4bef57919931ec3c1635310723e0c8
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106093406"
---
# <a name="azure-policy-pattern-deploy-resources"></a>Шаблон Политики Azure для развертывания ресурсов

Действие [deployIfNotExists](../concepts/effects.md#deployifnotexists) позволяет развертывать [шаблон Azure Resource Manager](../../../azure-resource-manager/templates/overview.md) (шаблон ARM) при создании или обновлении ресурса, не соответствующего требованиям. Такой подход можно применять вместо действия [deny](../concepts/effects.md#deny) (запрет), так как он позволяет создавать ресурсы, гарантированно приводя их в соответствие требованиям.

## <a name="sample-policy-definition"></a>Пример определения политики

В этом определении политики используется оператор **field** для вычисления `type` созданного или обновленного ресурса. Если это ресурс с типом _Microsoft.Network/virtualNetworks_, политика ищет наблюдатель за сетями в том же расположении, где находится новый или обновленный ресурс. Если соответствующий наблюдатель за сетями найти не удается, развертывается шаблон ARM для создания недостающего ресурса.

:::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json":::

### <a name="explanation"></a>Объяснение

#### <a name="existencecondition"></a>existenceCondition

:::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json" range="18-23":::

В блоке **properties.policyRule.then.details** сообщается Политике Azure, что нужно искать для ресурса, созданного или обновленного в блоке **properties.policyRule.if**. В нашем примере в группе ресурсов **networkWatcherRG** должен существовать наблюдатель,у которого значение **field** `location` совпадает с расположением нового или обновленного ресурса. Использование функции `field()` позволяет **existenceCondition** обращаться к свойствам нового или обновленного ресурса, в частности к свойству `location`.

#### <a name="roledefinitionids"></a>roleDefinitionIds

:::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json" range="24-26":::

Свойство _array_ **roleDefinitionIds** в блоке **properties.policyRule.then.details** сообщает определению политики, какие права должно иметь управляемое удостоверение для развертывания включенного шаблона Resource Manager. Это свойство должно включать роли с разрешениями, необходимыми для развертывания шаблона, но в соответствии с принципом "наименьших привилегий" не должно включать ничего, кроме обязательных операций.

#### <a name="deployment-template"></a>Шаблон развертывания

Раздел **deployment** в определении политики содержит блок **properties**, в котором определены три ключевых компонента.

- Свойство **mode** позволяет задать [режим развертывания](../../../azure-resource-manager/templates/deployment-modes.md) шаблона.

- Свойство **template** содержит сам шаблон. В нашем примере параметр шаблона **location** позволяет задать расположение нового ресурса наблюдателя за сетями.

  :::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json" range="30-44":::
  
- Свойство **parameters** определяет параметры, предоставленные в **template**. Имена параметров должны совпадать с теми, которые определены в **template**. В нашем примере подразумевается, что параметр имеет такое же имя **location**. Значение **location** повторно применяет функцию `field()` для получения значения оцениваемого ресурса, который здесь является виртуальной сетью из блока **policyRule.if**.

  :::code language="json" source="~/policy-templates/patterns/pattern-deploy-resources.json" range="45-49":::

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с другими [шаблонами и встроенными определениями](./index.md).
- Изучите статью о [структуре определения Политики Azure](../concepts/definition-structure.md).
- Изучите [сведения о действии политик](../concepts/effects.md).