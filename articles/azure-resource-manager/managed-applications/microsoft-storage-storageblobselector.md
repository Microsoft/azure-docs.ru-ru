---
title: Элемент пользовательского интерфейса Сторажеблобселектор
description: Описывает элемент пользовательского интерфейса Microsoft. Storage. Сторажеблобселектор для портал Azure.
author: tfitzmac
ms.topic: conceptual
ms.date: 10/27/2020
ms.author: tomfitz
ms.openlocfilehash: 1085b70df67a3f16a7f7f8c5ae85c9ab271b62ac
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92754622"
---
# <a name="microsoftstoragestorageblobselector-ui-element"></a>Элемент пользовательского интерфейса Microsoft. Storage. Сторажеблобселектор

Элемент управления для выбора большого двоичного объекта из учетной записи хранения Azure.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

Пользователю предоставляется возможность поиска доступных больших двоичных объектов хранилища.

:::image type="content" source="./media/managed-application-elements/microsoft-storage-storageblobselector-browse.png" alt-text="Microsoft. Storage. Сторажеблобселектор — обзор":::

После нажатия кнопки **Обзор** пользователь может выбрать учетную запись хранения.

:::image type="content" source="./media/managed-application-elements/microsoft-storage-storageblobselector-select.png" alt-text="Microsoft. Storage. Сторажеблобселектор — Выбор хранилища":::

Пользователь видит контейнеры в учетной записи хранения и может выбрать одну из них.

:::image type="content" source="./media/managed-application-elements/microsoft-storage-storageblobselector-containers.png" alt-text="Microsoft. Storage. Сторажеблобселектор — выбор контейнера":::

Из контейнера пользователь может выбрать файл.

:::image type="content" source="./media/managed-application-elements/microsoft-storage-storageblobselector-file.png" alt-text="Файл Microsoft. Storage. Сторажеблобселектор-File":::

Элемент управления обновляется для вывода имени выбранного файла.

:::image type="content" source="./media/managed-application-elements/microsoft-storage-storageblobselector-result.png" alt-text="Microsoft. Storage. Сторажеблобселектор — показывать Select File":::

## <a name="schema"></a>схема

```json
{
  "name": "storageBlobSelection",
  "type": "Microsoft.Storage.StorageBlobSelector",
  "visible": true,
  "toolTip": "Select storage blob",
  "label": "Package (.zip, .cspkg)",
  "options": {
    "text": "Select Package"
  },
  "constraints": {
    "allowedFileExtensions": [ "zip", "cspkg" ]
  }
}
```

## <a name="sample-output"></a>Пример выходных данных

```json
{
  "blobName": "test.zip",
  "sasUri": "https://azstorageaccnt1.blob.core.windows.net/container1/test.zip?sp=r&se=2020-10-10T07:46:22Z&sv=2019-12-12&sr=b&sig=X4EL8ZsRmiP1TVxkVfTcGyMj2sHg1zCbFBXsDmnNOyg%3D"
}

```

## <a name="remarks"></a>Remarks

Свойство **rechecks. алловедфиликстенсионс** указывает допустимые типы файлов.

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
* Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
