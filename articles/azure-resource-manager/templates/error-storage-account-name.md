---
title: Ошибки имени учетной записи хранения
description: Описание ошибок, которые могут возникнуть при указании имени учетной записи хранения.
ms.topic: troubleshooting
ms.date: 03/09/2018
ms.openlocfilehash: 5b2706d8540ea38ef08bf7ca0f804e6811a93085
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "76153978"
---
# <a name="resolve-errors-for-storage-account-names"></a>Устранение ошибок имен учетной записи хранения

В этой статье описываются ошибки именования, которые могут возникнуть при развертывании учетной записи хранения.

## <a name="symptom"></a>Симптом

Если имя учетной записи хранения содержит запрещенные знаки, возникает ошибка. Пример такой ошибки показан ниже.

```
Code=AccountNameInvalid
Message=S!torageckrexph7isnoc is not a valid storage account name. Storage account name must be
between 3 and 24 characters in length and use numbers and lower-case letters only.
```

Для учетных записей хранения необходимо указывать имя ресурса, уникальное в среде Azure. Если не указать уникальное имя, возникнет такая ошибка:

```
Code=StorageAccountAlreadyTaken
Message=The storage account named mystorage is already taken.
```

Если при развертывании учетной записи хранения указать для нее то же имя, что и для существующей учетной записи хранения в подписке, но указать другое расположение, появится сообщение о том, что учетная запись хранения уже существует в другом расположении. Удалите существующую учетную запись хранения или укажите то же расположение, что и для существующей учетной записи хранения.

## <a name="cause"></a>Причина

Имя учетной записи хранения должно содержать от 3 до 24 символов и состоять только из цифр и букв нижнего регистра. Имя должно быть уникальным.

## <a name="solution"></a>Решение

Убедитесь, что имя учетной записи хранения является уникальным. Уникальное имя можно создать, используя соглашение об именовании и результат функции [uniqueString](template-functions-string.md#uniquestring) .

```json
"name": "[concat('storage', uniqueString(resourceGroup().id))]",
"type": "Microsoft.Storage/storageAccounts",
```

Убедитесь, что имя учетной записи хранения содержит не больше 24 знаков. Функция [UniqueString](template-functions-string.md#uniquestring) возвращает 13 знаков. Если используется сцепка префикса или постфикса с результатом **uniqueString**, то следует использовать значение, содержащие не более 11 знаков.

```json
"parameters": {
  "storageNamePrefix": {
    "type": "string",
    "maxLength": 11,
    "defaultValue": "storage",
    "metadata": {
    "description": "The value to use for starting the storage account name."
    }
  }
}
```

Убедитесь, что имя учетной записи хранения не содержит буквы в верхнем регистре или специальные знаки.