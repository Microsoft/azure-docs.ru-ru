---
title: Элемент пользовательского интерфейса SizeSelector
description: Сведения об элементе пользовательского интерфейса Microsoft.Compute.SizeSelector для портала Azure. Используется для выбора размера виртуальной машины.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: d6408f8c08694ae681d302ae35f5778894091733
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87063662"
---
# <a name="microsoftcomputesizeselector-ui-element"></a>Элемент пользовательского интерфейса Microsoft.Compute.SizeSelector

Элемент управления для выбора размера одного или нескольких экземпляров виртуальной машины.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

Пользователь видит селектор со значениями по умолчанию из определения элемента.

![Элемент пользовательского интерфейса Microsoft.Compute.SizeSelector](./media/managed-application-elements/microsoft-compute-sizeselector.png)

Выбрав элемент управления, пользователь видит развернутое представление доступных размеров.

![Развернутый элемент Microsoft.Compute.SizeSelector](./media/managed-application-elements/microsoft-compute-sizeselector-expanded.png)

## <a name="schema"></a>схема

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.SizeSelector",
  "label": "Size",
  "toolTip": "",
  "recommendedSizes": [
    "Standard_D1",
    "Standard_D2",
    "Standard_D3"
  ],
  "constraints": {
    "allowedSizes": [],
    "excludedSizes": [],
    "numAvailabilityZonesRequired": 3,
    "zone": "3"
  },
  "options": {
    "hideDiskTypeFilter": false
  },
  "osPlatform": "Windows",
  "imageReference": {
    "publisher": "MicrosoftWindowsServer",
    "offer": "WindowsServer",
    "sku": "2012-R2-Datacenter"
  },
  "count": 2,
  "visible": true
}
```

## <a name="sample-output"></a>Пример выходных данных

```json
"Standard_D1"
```

## <a name="remarks"></a>Комментарии

- В `recommendedSizes` должен быть указан по меньшей мере один размер. По умолчанию используется первый рекомендуемый размер. Список доступных размеров не отсортирован по столбцу "Рекомендуемое состояние". Для сортировки по этому столбцу его нужно щелкнуть.
- Если в выбранном расположении рекомендуемый размер недоступен, он автоматически пропускается. Вместо него используется следующий рекомендуемый размер.
- Параметры `constraints.allowedSizes` и `constraints.excludedSizes` являются необязательными. При этом их нельзя использовать одновременно. Сведения о получении списка доступных размеров виртуальных машин для подписки см. в [этой статье](/rest/api/compute/virtualmachines/virtualmachines-list-sizes-region). Любой размер, не указанный в параметре `constraints.allowedSizes`, скрыт. При этом любой размер, не указанный в параметре `constraints.excludedSizes`, отображается.
- Необходимо задать значение для параметра `osPlatform` (**Windows** или **Linux**). Он используется для определения затрат на оборудование виртуальных машин.
- Параметр `imageReference` не указывается для основных образов, но указывается для сторонних. Он используется для определения затрат на программное обеспечение виртуальных машин.
- Параметр `count` используется для задания соответствующего числа для элемента. Он поддерживает статическое значение, например **2**, или динамическое значение из другого элемента, например `[steps('step1').vmCount]`. Значение по умолчанию — **1**.
- Параметру `numAvailabilityZonesRequired` можно установить значение 1, 2 или 3.
- Значение `hideDiskTypeFilter` по умолчанию — **false**. Фильтр по типу диска позволяет пользователю просматривать все типы дисков или только SSD.

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
* Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
