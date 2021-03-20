---
title: API службы контроля потребления (коммерческая платформа Майкрософт)
description: API событий потребления позволяет создавать события потребления для предложений SaaS в Microsoft AppSource и Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 05/26/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: b75964f8cfc41efc35858284dbffded3aa406eb6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96436068"
---
# <a name="marketplace-metered-billing-apis"></a>API для выставления счетов с оплатой в Marketplace

Если издатель создает настраиваемые измерения измерений для предложения, которое должно быть опубликовано в центре партнеров, следует использовать API с лимитным выставлением счетов. Интеграция с отслеживаемыми API выставления счетов требуется для любого приобретенного предложения, имеющего один или несколько планов с пользовательскими измерениями для создания событий использования.

Дополнительные сведения о создании настраиваемых измерений измерения для SaaS см. в статье [лимитное выставление счетов SaaS](saas-metered-billing.md).

Дополнительные сведения о создании настраиваемых измерений измерения для предложения приложения Azure с помощью управляемого плана приложений см. в статье [Настройка параметров предложения для приложения Azure](../create-new-azure-apps-offer.md#configure-your-azure-application-offer-setup-details).

## <a name="enforcing-tls-12-note"></a>Применение Примечания к TLS 1,2

Версия TLS 1,2 применяется в качестве минимальной версии для HTTPS-соединений. Убедитесь, что в коде используется эта версия TLS. TLS версии 1,0 и 1,1 являются устаревшими, и попытки подключения будут отклонены.

## <a name="metered-billing-single-usage-event"></a>Событие с оплатой за одно использование

API события использования должен вызываться издателем для отправки событий использования в активный ресурс (с подпиской) для плана, приобретенного конкретным клиентом. Событие использования создается отдельно для каждого пользовательского измерения плана, определенного издателем при публикации предложения.

Для каждого часа календарного дня может быть выдаваться только одно событие использования. Например, в 8:05:15 уже сегодня можно выдать одно событие использования. Если это событие принято, следующее событие использования будет принято через 9:00 am сегодня. Если вы отправите дополнительное событие между 8:15 и 8:59:59 уже сегодня, оно будет отклонено как дубликат. Следует накапливать все единицы, потребляемые в течение часа, а затем выдавать их в одном событии.

Только одно событие использования может быть выдаваться за каждый час календарного дня на ресурс. Если в течение часа используется более одной единицы, накапливает все единицы, потребленные в течение часа, а затем выдает их в одном событии. События использования могут выдаваться только за последние 24 часа. Если вы выдаете событие использования в любое время между 8:00 и 8:59:59 (и принимаете) и отправляете дополнительное событие для того же дня между 8:00 и 8:59:59, оно будет отклонено как повторяющееся.

**POST**: `https://marketplaceapi.microsoft.com/api/usageEvent?api-version=<ApiVersion>`

*Параметры запроса:*

| Параметра | Рекомендация          |
| ---------- | ---------------------- |
| `ApiVersion` | Используйте 2018-08-31. |
| | |

*Заголовки запроса:*

| Content-type       | Используйте `application/json`.  |
| ------------------ | ---------------------------- |
| `x-ms-requestid`     | Уникальное строковое значение для отслеживания запроса клиента, желательно GUID. Если это значение не указано, оно создается случайным образом и возвращается в заголовке ответа. |
| `x-ms-correlationid` | Уникальное строковое значение для операции на стороне клиента. Этот параметр позволяет сопоставить все события клиентской операции с событиями на стороне сервера. Если это значение не указано, оно создается и возвращается в заголовках ответа. |
| `authorization`   | Уникальный маркер доступа, определяющий независимый поставщик программного обеспечения, который делает этот вызов API. Формат — это то, `"Bearer <access_token>"` что значение маркера извлекается издателем, как описано в <br> <ul> <li> SaaS в [получить токен с HTTP POST](./pc-saas-registration.md#get-the-token-with-an-http-post). </li> <li> Управляемое приложение в [стратегиях проверки подлинности](./marketplace-metering-service-authentication.md). </li> </ul> |
| | |

*Пример текста запроса:*

```json
{
  "resourceId": <guid>, // unique identifier of the resource against which usage is emitted. 
  "quantity": 5.0, // how many units were consumed for the date and hour specified in effectiveStartTime, must be greater than 0, can be integer or float value
  "dimension": "dim1", // custom dimension identifier
  "effectiveStartTime": "2018-12-01T08:30:14", // time in UTC when the usage event occurred, from now and until 24 hours back
  "planId": "plan1", // id of the plan purchased for the offer
}
```

>[!NOTE]
>`resourceId` имеет другое значение для приложения SaaS и управляемого приложения, выдачи настраиваемого счетчика. 

Для планов приложений, управляемых приложением Azure, `resourceId` это управляемое приложение `resource group Id` . Пример скрипта для его получения приведен в разделе [Использование токена удостоверений, управляемых Azure](./marketplace-metering-service-authentication.md#using-the-azure-managed-identities-token). 

Для предложений SaaS `resourceId` содержит идентификатор подписки SaaS. Дополнительные сведения о подписках SaaS см. [в этом списке](./pc-saas-fulfillment-api-v2.md#get-list-of-all-subscriptions).

### <a name="responses"></a>Ответы

Код: 200<br>
Все в порядке. Эмиссия использования была принята и записана на стороне Майкрософт для дальнейшей обработки и выставления счетов.

Пример полезных данных ответа: 

```json
{
  "usageEventId": <guid>, // unique identifier associated with the usage event in Microsoft records
  "status": "Accepted" // this is the only value in case of single usage event
  "messageTime": "2020-01-12T13:19:35.3458658Z", // time in UTC this event was accepted
  "resourceId": <guid>, // unique identifier of the resource against which usage is emitted. For SaaS it's the subscriptionId.
  "quantity": 5.0, // amount of emitted units as recorded by Microsoft
  "dimension": "dim1", // custom dimension identifier
  "effectiveStartTime": "2018-12-01T08:30:14", // time in UTC when the usage event occurred, as sent by the ISV
  "planId": "plan1", // id of the plan purchased for the offer
}
```

Код: 400 <br>
Недопустимый запрос.

* Указаны отсутствующие или недопустимые данные запроса.
* `effectiveStartTime` более 24 часов в прошлом. Срок действия события истек.
* Подписка SaaS не находится в состоянии подписки.

Пример полезных данных ответа: 

```json
{
  "message": "One or more errors have occurred.",
  "target": "usageEventRequest",
  "details": [
    {
      "message": "The resourceId is required.",
      "target": "ResourceId",
      "code": "BadArgument"
    }
  ],
  "code": "BadArgument"
}
```

Код: 403<br>

Запрещено. Маркер авторизации не указан, является недопустимым или срок его действия истек.  Или запрос пытается получить доступ к подписке для предложения, которое было опубликовано с другим ИДЕНТИФИКАТОРом Azure AD App из того, который использовался для создания маркера авторизации.

Код: 409<br>
Конфликт. Событие использования уже успешно отправлено для указанного идентификатора ресурса, даты эффективности использования и часа.

Пример полезных данных ответа: 

```json
{
  "additionalInfo": {
    "acceptedMessage": {
      "usageEventId": "<guid>", //unique identifier associated with the usage event in Microsoft records
      "status": "Duplicate",
      "messageTime": "2020-01-12T13:19:35.3458658Z",
      "resourceId": "<guid>", //unique identifier of the resource against which usage is emitted.
      "quantity": 1.0,
      "dimension": "dim1",
      "effectiveStartTime": "2020-01-12T11:03:28.14Z",
      "planId": "plan1"
    }
  },
  "message": "This usage event already exist.",
  "code": "Conflict"
}
```

## <a name="metered-billing-batch-usage-event"></a>Событие использования пакетного выставления счетов

API событий использования пакетной службы позволяет создавать события использования для нескольких приобретенных ресурсов одновременно. Он также позволяет создавать несколько событий использования для одного ресурса, если они предназначены для разных календарных часов. Максимальное число событий в одном пакете — 25.

**POST:** `https://marketplaceapi.microsoft.com/api/batchUsageEvent?api-version=<ApiVersion>`

*Параметры запроса:*

| Параметр  | Рекомендация     |
| ---------- | -------------------- |
| `ApiVersion` | Используйте 2018-08-31. |

*Заголовки запроса:*

| Content-type       | Используйте `application/json`.       |
| ------------------ | ------ |
| `x-ms-requestid`     | Уникальное строковое значение для отслеживания запроса клиента, желательно GUID. Если это значение не указано, оно создается случайным образом и возвращается в заголовках ответа. |
| `x-ms-correlationid` | Уникальное строковое значение для операции на стороне клиента. Этот параметр позволяет сопоставить все события клиентской операции с событиями на стороне сервера. Если это значение не указано, оно создается и возвращается в заголовках ответа. |
| `authorization`      | Уникальный маркер доступа, определяющий независимый поставщик программного обеспечения, который делает этот вызов API. Формат — это то, `Bearer <access_token>` что значение маркера извлекается издателем, как описано в <br> <ul> <li> SaaS в [получить токен с HTTP POST](./pc-saas-registration.md#get-the-token-with-an-http-post). </li> <li> Управляемое приложение в [стратегиях проверки подлинности](./marketplace-metering-service-authentication.md). </li> </ul> |
| | |


*Пример текста запроса:*

```json
{
  "request": [ // list of usage events for the same or different resources of the publisher
    { // first event
      "resourceId": "<guid1>", // Unique identifier of the resource against which usage is emitted. 
      "quantity": 5.0, // how many units were consumed for the date and hour specified in effectiveStartTime, must be greater than 0, can be integer or float value
      "dimension": "dim1", //Custom dimension identifier
      "effectiveStartTime": "2018-12-01T08:30:14",//Time in UTC when the usage event occurred, from now and until 24 hours back
      "planId": "plan1", // id of the plan purchased for the offer
    },
    { // next event
      "resourceId": "<guid2>", 
      "quantity": 39.0, 
      "dimension": "email", 
      "effectiveStartTime": "2018-11-01T23:33:10
      "planId": "gold", // id of the plan purchased for the offer
    }
  ]
}
```

>[!NOTE]
>`resourceId` имеет другое значение для приложения SaaS и управляемого приложения, выдачи настраиваемого счетчика. 

Для планов приложений, управляемых приложением Azure, `resourceId` это управляемое приложение `resource group Id` . Пример скрипта для его получения приведен в разделе [Использование токена удостоверений, управляемых Azure](./marketplace-metering-service-authentication.md#using-the-azure-managed-identities-token). 

Для предложений SaaS `resourceId` содержит идентификатор подписки SaaS. Дополнительные сведения о подписках SaaS см. [в этом списке](./pc-saas-fulfillment-api-v2.md#get-list-of-all-subscriptions).

### <a name="responses"></a>Ответы

Код: 200<br>
Все в порядке. Эмиссия использования пакетов была принята и записана на стороне Майкрософт для дальнейшей обработки и выставления счетов. Список ответов возвращается с состоянием каждого отдельного события в пакете. Необходимо выполнить итерацию полезных данных ответа, чтобы понять ответы для каждого отдельного события использования, отправляемого как часть события пакетной службы.

Пример полезных данных ответа: 

```json
{
  "count": 2, // number of records in the response
  "result": [
    { // first response
      "usageEventId": "<guid>", // unique identifier associated with the usage event in Microsoft records
      "status": "Accepted" // see list of possible statuses below,
      "messageTime": "2020-01-12T13:19:35.3458658Z", // Time in UTC this event was accepted by Microsoft,
      "resourceId": "<guid1>", // unique identifier of the resource against which usage is emitted.
      "quantity": 5.0, // amount of emitted units as recorded by Microsoft 
      "dimension": "dim1", // custom dimension identifier
      "effectiveStartTime": "2018-12-01T08:30:14",// time in UTC when the usage event occurred, as sent by the ISV
      "planId": "plan1", // id of the plan purchased for the offer
    },
    { // second response
      "status": "Duplicate",
      "messageTime": "0001-01-01T00:00:00",
      "error": {
        "additionalInfo": {
          "acceptedMessage": {
            "usageEventId": "<guid>",
            "status": "Duplicate",
            "messageTime": "2020-01-12T13:19:35.3458658Z",
            "resourceId": "<guid2>",
            "quantity": 1.0,
            "dimension": "email",
            "effectiveStartTime": "2020-01-12T11:03:28.14Z",
            "planId": "gold"
          }
        },
        "message": "This usage event already exist.",
        "code": "Conflict"
      },
      "resourceId": "<guid2>",
      "quantity": 1.0,
      "dimension": "email",
      "effectiveStartTime": "2020-01-12T11:03:28.14Z",
      "planId": "gold"
    }
  ]
}
```

Описание кода состояния, указанного в ответе API `BatchUsageEvent`:

| Код состояния  | Описание |
| ---------- | -------------------- |
| `Accepted` | Принято. |
| `Expired` | Потребление с истекшим сроком действия. |
| `Duplicate` | Предоставлен дубликат сведений о потреблении. |
| `Error` | Код ошибки. |
| `ResourceNotFound` | Указан недопустимый ресурс потребления. |
| `ResourceNotAuthorized` | У вас нет разрешения на предоставление сведений о потреблении для этого ресурса. |
| `InvalidDimension` | Измерение, для которого предоставлены сведения о потреблении, недопустимо для этого предложения или плана. |
| `InvalidQuantity` | Переданное количество меньше или равно 0. |
| `BadArgument` | Входное значение отсутствует или является недопустимым. |

Код: 400<br>
Недопустимый запрос. Пакет содержал более 25 событий использования.

Код: 403<br>
Запрещено. Маркер авторизации не указан, является недопустимым или срок его действия истек.  Или запрос пытается получить доступ к подписке для предложения, которое было опубликовано с другим ИДЕНТИФИКАТОРом Azure AD App из того, который использовался для создания маркера авторизации.

## <a name="development-and-testing-best-practices"></a>Рекомендации по разработке и тестированию

Чтобы протестировать пользовательский счетчик, реализуйте интеграцию с API отслеживания использования, создав план для опубликованного предложения SaaS с пользовательскими измерениями, определенными в нем, с нулевой ценой на единицу. И опубликовать это предложение как предварительный просмотр, поэтому только ограниченные пользователи смогут получить доступ к интеграции и проверить ее.

Вы также можете использовать частный план для существующего предложения Live, чтобы ограничить доступ к этому плану во время тестирования до ограниченной аудитории.

## <a name="get-support"></a>Получение поддержки

Следуйте инструкциям в [технической поддержке программы коммерческого рынка в центре партнеров](../support.md) , чтобы узнать о вариантах поддержки издателя и отправить запрос в службу поддержки корпорации Майкрософт.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об API-интерфейсах службы измерения см. в статье [вопросы и ответы по API службы измерения Marketplace](./marketplace-metering-service-apis-faq.md).
