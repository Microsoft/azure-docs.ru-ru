---
title: Элемент пользовательского интерфейса PublicIpAddressCombo
description: Сведения об элементе пользовательского интерфейса Microsoft.Network.PublicIpAddressCombo для портала Azure.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/28/2018
ms.author: tomfitz
ms.openlocfilehash: 5def6db9d551b3882204c9f997f164a0df7ac223
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87063299"
---
# <a name="microsoftnetworkpublicipaddresscombo-ui-element"></a>Элемент пользовательского интерфейса Microsoft.Network.PublicIpAddressCombo

Группа элементов управления для выбора нового или имеющегося общедоступного IP-адреса.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

![Элемент пользовательского интерфейса Microsoft.Network.PublicIpAddressCombo](./media/managed-application-elements/microsoft-network-publicipaddresscombo.png)

- Если для общедоступного IP-адреса выбрано значение "Нет", текстовое поле метки доменного имени будет скрыто.
- Если выбран имеющийся общедоступный IP-адрес, текстовое поле метки доменного имени будет отключено. Его значение является меткой доменного имени выбранного IP-адреса.
- Суффикс доменного имени (например, westus.cloudapp.azure.com) обновляется автоматически на основе выбранного расположения.

## <a name="schema"></a>схема

```json
{
  "name": "element1",
  "type": "Microsoft.Network.PublicIpAddressCombo",
  "label": {
    "publicIpAddress": "Public IP address",
    "domainNameLabel": "Domain name label"
  },
  "toolTip": {
    "publicIpAddress": "",
    "domainNameLabel": ""
  },
  "defaultValue": {
    "publicIpAddressName": "ip01",
    "domainNameLabel": "mydomain"
  },
  "constraints": {
    "required": {
      "domainNameLabel": true
    }
  },
  "options": {
    "hideNone": false,
    "hideDomainNameLabel": false,
    "hideExisting": false,
    "zone": 3
  },
  "visible": true
}
```

## <a name="sample-output"></a>Пример выходных данных

Если общедоступный IP-адрес не выбран, элемент управления возвращает следующие выходные данные:

```json
{
  "newOrExistingOrNone": "none"
}
```

Если пользователь выбрал или создал общедоступный IP-адрес, элемент управления возвращает следующие выходные данные:

```json
{
  "name": "ip01",
  "resourceGroup": "rg01",
  "domainNameLabel": "mydomain",
  "publicIPAllocationMethod": "Dynamic",
  "sku": "Basic",
  "newOrExistingOrNone": "new"
}
```

- Если параметр `options.hideNone` имеет значение **true**, `newOrExistingOrNone` то параметр будет иметь только **новое** или **существующее**.
- Если `options.hideDomainNameLabel` задано **значение true**, `domainNameLabel` то значение не объявлено.

## <a name="remarks"></a>Комментарии

- Если для параметра `constraints.required.domainNameLabel` задано значение **true**, пользователю необходимо указать метку доменного имени при создании общедоступного IP-адреса. Имеющиеся общедоступные IP-адреса без метки недоступны для выбора.
- Если для параметра `options.hideNone` задано значение **true**, значение **Нет** для общедоступного IP-адреса будет скрыто. Значение по умолчанию — **false**.
- Если для параметра `options.hideDomainNameLabel` задано значение **true**, текстовое поле для метки доменного имени будет скрыто. Значение по умолчанию — **false**.
- Если для параметра `options.hideExisting` задано значение true, пользователь не сможет выбрать имеющийся общедоступный IP-адрес. Значение по умолчанию — **false**.
- В качестве значения параметра `zone` можно указать общедоступные IP-адреса указанной зоны или отказоустойчивые в зоне общедоступные IP-адреса.

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
* Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
