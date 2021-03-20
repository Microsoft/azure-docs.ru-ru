---
title: Схемы отслеживания AS2 для сообщений B2B
description: Создание схем отслеживания для отслеживания сообщений AS2 в Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 01/01/2020
ms.openlocfilehash: bccf69362279afd9e8148b20b61ff3ea9b472a03
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "76906973"
---
# <a name="create-schemas-for-tracking-as2-messages-in-azure-logic-apps"></a>Создание схем для отслеживания сообщений AS2 в Azure Logic Apps

Схемы отслеживания B2B позволяют выполнять в учетной записи интеграции Azure мониторинг ошибок, успешных операций и свойств сообщений транзакций типа "бизнес — бизнес" (AS2):

* Схема отслеживания сообщений AS2
* Схема отслеживания уведомления о расположении сообщений AS2 (MDN)

## <a name="as2-message-tracking-schema"></a>Схема отслеживания сообщений AS2

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "as2To": "",
      "as2From": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "messageId": "",
      "dispositionType": "",
      "fileName": "",
      "isMessageFailed": "",
      "isMessageSigned": "",
      "isMessageEncrypted": "",
      "isMessageCompressed": "",
      "correlationMessageId": "",
      "incomingHeaders": {},
      "outgoingHeaders": {},
      "isNrrEnabled": "",
      "isMdnExpected": "",
      "mdnType": ""
    }
}
```

| Свойство | Обязательно | Тип | Описание |
|----------|----------|------|-------------|
| senderPartnerName | нет | Строка | Имя партнера отправителя сообщений AS2 |
| receiverPartnerName | нет | Строка | Имя партнера получателя сообщений AS2 |
| as2To | Да | Строка | Имя получателя сообщения AS2 из заголовков сообщения AS2 |
| as2From | Да | Строка | Имя отправителя сообщения AS2 из заголовков сообщения AS2 |
| agreementName | нет | Строка | Имя соглашения AS2, в соответствии с которым разрешаются сообщения. |
| direction | Да | Строка | Направление потока сообщений `receive` , либо `send` |
| messageId | нет | Строка | Идентификатор сообщения AS2 из заголовков сообщения AS2 |
| dispositionType | нет | Строка | Значение типа расстановки уведомления о размещении сообщений (MDN) |
| fileName | нет | Строка | Имя файла из заголовка сообщения AS2 |
| isMessageFailed | Да | Логическое | Не удалось ли сообщение AS2 |
| isMessageSigned | Да | Логическое | Было ли сообщение AS2 подписано |
| isMessageEncrypted | Да | Логическое | Было ли сообщение AS2 зашифровано |
| isMessageCompressed | Да | Логическое | Указывает, было ли сообщение AS2 сжато |
| correlationMessageId | нет | Строка | Идентификатор сообщения AS2 для корреляции сообщений с помощью MDN |
| incomingHeaders | Нет | Словарь JToken | Сведения о заголовке входящего сообщения AS2 |
| outgoingHeaders | Нет | Словарь JToken | Сведения о заголовке исходящего сообщения AS2 |
| isNrrEnabled | Да | Логическое | Следует ли использовать значение по умолчанию, если значение неизвестно |
| isMdnExpected | Да | Логическое | Следует ли использовать значение по умолчанию, если значение неизвестно |
| mdnType | Да | Перечисление | Допустимые значения: `NotConfigured` , `Sync` и `Async` |
|||||

## <a name="as2-mdn-tracking-schema"></a>Схема отслеживания уведомлений о состоянии сообщений AS2

```json
{
   "agreementProperties": {
      "senderPartnerName": "",
      "receiverPartnerName": "",
      "as2To": "",
      "as2From": "",
      "agreementName": ""
   },
   "messageProperties": {
      "direction": "",
      "messageId": "",
      "originalMessageId": "",
      "dispositionType": "",
      "isMessageFailed": "",
      "isMessageSigned": "",
      "isNrrEnabled": "",
      "statusCode": "",
      "micVerificationStatus": "",
      "correlationMessageId": "",
      "incomingHeaders": {
      },
      "outgoingHeaders": {
      }
   }
}
```

| Свойство | Обязательно | Тип | Описание |
|----------|----------|------|-------------|
| senderPartnerName | нет | Строка | Имя партнера отправителя сообщений AS2 |
| receiverPartnerName | нет | Строка | Имя партнера получателя сообщений AS2 |
| as2To | Да | Строка | Имя партнера, получающего сообщение AS2 |
| as2From | Да | Строка | Имя партнера, отправляющего сообщение AS2 |
| agreementName | нет | Строка | Имя соглашения AS2, в соответствии с которым разрешаются сообщения. |
| direction | Да | Строка | Направление потока сообщений `receive` , либо `send` |
| messageId | нет | Строка | Идентификатор сообщения AS2 |
| originalMessageId | нет | Строка | Идентификатор исходного сообщения AS2 |
| dispositionType | нет | Строка | Значение типа метода обработки MDN |
| isMessageFailed | Да | Логическое | Не удалось ли сообщение AS2 |
| isMessageSigned | Да | Логическое | Было ли сообщение AS2 подписано |
| isNrrEnabled | Да | Логическое | Следует ли использовать значение по умолчанию, если значение неизвестно |
| statusCode | Да | Перечисление | Допустимые значения: `Accepted` , `Rejected` и `AcceptedWithErrors` |
| micVerificationStatus | Да | Перечисление | Допустимые значения: `NotApplicable` , `Succeeded` и `Failed` |
| correlationMessageId | нет | Строка | Идентификатор корреляции, который является ИДЕНТИФИКАТОРом исходного сообщения с настроенным MDN |
| incomingHeaders | Нет | Словарь JToken | Сведения о заголовке входящего сообщения |
| outgoingHeaders | Нет | Словарь JToken | Сведения о заголовке исходящего сообщения |
|||||

## <a name="b2b-protocol-tracking-schemas"></a>Схемы отслеживания для протоколов B2B

Сведения о схемах отслеживания для протоколов B2B см. в следующих статьях:

* [Схемы отслеживания X12](logic-apps-track-integration-account-x12-tracking-schema.md)
* [Настраиваемые схемы отслеживания B2B](logic-apps-track-integration-account-custom-tracking-schema.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Мониторинг сообщений B2B с помощью журналов Azure Monitor](../logic-apps/monitor-b2b-messages-log-analytics.md)