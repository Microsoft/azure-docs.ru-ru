---
title: создание хранилищ Служб восстановления с помощью REST API
description: Из этой статьи вы узнаете, как управлять операциями резервного копирования и восстановления резервной копии виртуальной машины Azure с помощью REST API.
ms.topic: conceptual
ms.date: 08/21/2018
ms.assetid: e54750b4-4518-4262-8f23-ca2f0c7c0439
ms.openlocfilehash: a37808548ec58977b7d6af16c75b94b7b5efe446
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96002942"
---
# <a name="create-azure-recovery-services-vault-using-rest-api"></a>Создание хранилища служб восстановления Azure с помощью REST API

Инструкции по созданию хранилища служб восстановления Azure с помощью REST API приведены в документации по [созданию REST API в хранилище](/rest/api/recoveryservices/vaults/createorupdate) . Давайте будем использовать этот документ в качестве справочной документации по созданию хранилища "Testvault задано" в "западной части США".

Для создания или обновления хранилища Служб восстановления Azure используйте следующую операцию *PUT*.

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}?api-version=2016-06-01
```

## <a name="create-a-request"></a>Создание запроса

Для создания запроса *PUT* требуется параметр `{subscription-id}`. Если у вас несколько подписок, см. раздел [Работа с несколькими подписками](/cli/azure/manage-azure-subscriptions-azure-cli). Для ваших ресурсов параметры `{resourceGroupName}` и `{vaultName}` определяются вместе с параметром `api-version`. Для этой статьи используется `api-version=2016-06-01`.

Ниже приведены обязательные заголовки.

| Заголовок запроса   | Описание |
|------------------|-----------------|
| *Content-Type:*  | Обязательный элемент. Задайте значение `application/json`. |
| *Authorization:* | Обязательный элемент. Задайте допустимый [маркер доступа](/rest/api/azure/#authorization-code-grant-interactive-clients) `Bearer`. |

Дополнительные сведения по созданию запросов см. в разделе [Components of a REST API request/response](/rest/api/azure/#components-of-a-rest-api-requestresponse) (Компоненты запроса или ответа REST API).

## <a name="create-the-request-body"></a>Создание текста запроса

Для создания текста запроса используются следующие общие определения.

|Имя  |Обязательно  |Тип  |Описание  |
|---------|---------|---------|---------|
|eTag     |         |   Строка      |  Необязательный eTag       |
|location     |  true       |Строка         |   Расположение ресурса      |
|properties     |         | [VaultProperties](/rest/api/recoveryservices/vaults/createorupdate#vaultproperties)        |  Свойства хранилища       |
|sku     |         |  [SKU](/rest/api/recoveryservices/vaults/createorupdate#sku)       |    В этом столбце указан уникальный системный идентификатор для каждого ресурса Azure.     |
|tags     |         | Объект        |     Теги ресурсов    |

Обратите внимание, что имена хранилища и группы ресурсов указаны в универсальном коде ресурса (URI) PUT. Текст запроса определяет расположение.

## <a name="example-request-body"></a>Примеры текста запроса

Следующий пример текста используется для создания хранилища в регионе "Западная часть США". Укажите расположение. Номер SKU всегда будет ценовой категории "Стандартный".

```json
{
  "properties": {},
  "sku": {
    "name": "Standard"
  },
  "location": "West US"
}
```

## <a name="responses"></a>Ответы

Есть два успешных ответа для операции по созданию или обновлению хранилища Служб восстановления:

|Имя  |Type  |Описание  |
|---------|---------|---------|
|200 ОК     |   [Хранилище](/rest/api/recoveryservices/vaults/createorupdate#vault)      | ОК        |
|201 Создано     | [Хранилище](/rest/api/recoveryservices/vaults/createorupdate#vault)        |   Создание      |

Дополнительные сведения об ответах REST API можно узнать в разделе [Process the response message](/rest/api/azure/#process-the-response-message) (Обработка ответного сообщения).

### <a name="example-response"></a>Пример ответа

Сокращенный ответ *201 Created* (201 Создано), полученный из предыдущего примера текста запроса, показывает, что был назначен *идентификатор* и что для состояния *provisioningState* указано значение *Succeeded*:

```json
{
  "location": "westus",
  "name": "testVault",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "id": "/subscriptions/77777777-b0c6-47a2-b37c-d8e65a629c18/resourceGroups/Default-RecoveryServices-ResourceGroup/providers/Microsoft.RecoveryServices/vaults/testVault",
  "type": "Microsoft.RecoveryServices/vaults",
  "sku": {
    "name": "Standard"
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия

[Create Azure Recovery Services backup policies using REST API](backup-azure-arm-userestapi-createorupdatepolicy.md) (Создание политик резервного копирования Служб восстановления Azure с помощью REST API)

Дополнительные сведения о REST API Azure см. в следующих документах:

- [Recovery Services](/rest/api/recoveryservices/) (Службы восстановления)
- [Начало работы с Azure REST API](/rest/api/azure/)
