---
title: Элемент пользовательского интерфейса StorageAccountSelector
description: Сведения об элементе пользовательского интерфейса Microsoft.Storage.StorageAccountSelector для портала Azure.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/28/2018
ms.author: tomfitz
ms.openlocfilehash: fe70c42387e9dedea8943b5dcce04a1f9ded5190
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87033307"
---
# <a name="microsoftstoragestorageaccountselector-ui-element"></a>Элемент пользовательского интерфейса Microsoft.Storage.StorageAccountSelector

Элемент управления для выбора новой или имеющейся учетной записи хранения.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

Элемент управления, который отображает значение по умолчанию.

![Элемент пользовательского интерфейса Microsoft.Storage.StorageAccountSelector](./media/managed-application-elements/microsoft-storage-storageaccountselector.png)

Элемент управления, который позволяет выбрать существующую учетную запись хранения или создать новую.

![Microsoft.Storage.StorageAccountSelector: создание учетной записи хранения](./media/managed-application-elements/microsoft-storage-storageaccountselector-new.png)

## <a name="schema"></a>схема

```json
{
  "name": "element1",
  "type": "Microsoft.Storage.StorageAccountSelector",
  "label": "Storage account",
  "toolTip": "",
  "defaultValue": {
    "name": "storageaccount01",
    "type": "Premium_LRS"
  },
  "constraints": {
    "allowedTypes": [],
    "excludedTypes": []
  },
  "options": {
    "hideExisting": false
  },
  "visible": true
}
```

## <a name="sample-output"></a>Пример выходных данных

```json
{
  "name": "storageaccount01",
  "resourceGroup": "rg01",
  "type": "Premium_LRS",
  "newOrExisting": "new"
}
```

## <a name="remarks"></a>Комментарии

- Если необходимо, параметр `defaultValue.name` автоматически проверяется на уникальность. Если имя учетной записи хранения не является уникальным, пользователь должен указать другое имя или выбрать существующую учетную запись хранения.
- Значением по умолчанию для параметра `defaultValue.type` является **Premium_LRS**.
- Любой тип, не указанный в `constraints.allowedTypes`, скрыт, а любой тип, не указанный в `constraints.excludedTypes`, отображается. Параметры `constraints.allowedTypes` и `constraints.excludedTypes` являются необязательными. При этом их нельзя использовать одновременно.
- Если для параметра `options.hideExisting` задано значение **true**, пользователь не может выбрать имеющуюся учетную запись хранения. Значение по умолчанию — **false**.

## <a name="next-steps"></a>Дальнейшие действия
* Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
* Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
