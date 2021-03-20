---
title: Элемент пользовательского интерфейса Slider
description: Описывает элемент пользовательского интерфейса Microsoft. Common. Slider для портал Azure. Позволяет пользователям задавать значение из диапазона параметров.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/10/2020
ms.author: tomfitz
ms.openlocfilehash: bb168a4bff6d2570f539307530232b5063bb535c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87098532"
---
# <a name="microsoftcommonslider-ui-element"></a>Элемент пользовательского интерфейса Microsoft. Common. Slider

Элемент управления "ползунок" позволяет пользователям выбирать из диапазона допустимых значений.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

:::image type="content" source="./media/managed-application-elements/microsoft-common-slider.png" alt-text="Microsoft. Common. Slider":::

## <a name="schema"></a>схема

```json
{
    "name": "memorySize",
    "type": "Microsoft.Common.Slider",
    "min": 1,
    "max": 64,
    "label": "Memory",
    "subLabel": "MB",
    "defaultValue": 24,
    "showStepMarkers": false,
    "toolTip": "Pick the size in MB",
    "constraints": {
        "required": false
    },
    "visible": true
}
```

## <a name="sample-output"></a>Пример выходных данных

```json
26
```

## <a name="remarks"></a>Комментарии

- `min`Значения и `max` являются обязательными. Они устанавливают начальную и конечную точки для ползунка.
- `showStepMarkers`Свойство по умолчанию имеет значение true. Маркеры шага отображаются, только если диапазон от min до max равен 100 или меньше.


## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
* Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
