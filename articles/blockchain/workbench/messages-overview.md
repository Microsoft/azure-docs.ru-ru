---
title: Использование сообщений для интеграции с Azure Блокчейн Workbench
description: Общие сведения об использовании сообщений для интеграции предварительной версии Azure Блокчейн Workbench с другими системами.
ms.date: 09/05/2019
ms.topic: article
ms.reviewer: brendal
ms.openlocfilehash: 14bd0f84bc9490d95d3dbe0b9f122882f0d2059d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "74324510"
---
# <a name="azure-blockchain-workbench-messaging-integration"></a>Интеграция службы сообщений Azure Blockchain Workbench

Помимо REST API, Azure Blockchain Workbench также предоставляет интеграции на основе обмена сообщениями. Workbench публикует связанные с реестром события через службу "Сетка событий Azure", позволяя нисходящим объектам-получателям получать данные или предпринимать действия на основе этих событий. Для клиентов, которым требуется надежный обмен сообщениями, Azure Blockchain Workbench также доставляет сообщения в конечную точку служебной шины Azure.

## <a name="input-apis"></a>API ввода

Если вы хотите инициировать транзакции из внешних систем для создания пользователей, создания и обновления контрактов, можно использовать API ввода службы сообщений, чтобы осуществлять транзакции в реестре. Пример, демонстрирующий API ввода, см. в репозитории с [примерами интеграции службы сообщений](https://aka.ms/blockchain-workbench-integration-sample).

Ниже перечислены API ввода, которые доступны в настоящее время.

### <a name="create-user"></a>Создать пользователя

Создает пользователя.

Для данного запроса требуются следующие поля:

| **имя**;             | **Описание**                                      |
|----------------------|------------------------------------------------------|
| requestId            | GUID, предоставляемый клиентом                                |
| firstName            | Имя пользователя                              |
| lastName             | Фамилия пользователя                               |
| emailAddress         | Адрес электронной почты пользователя                           |
| externalId           | Идентификатор объекта пользователя Azure AD                      |
| connectionId         | Уникальный идентификатор подключения блокчейна |
| messageSchemaVersion | Версия схемы службы сообщений                            |
| messageName          | **CreateUserRequest**                               |

Пример

``` json
{
    "requestId": "e2264523-6147-41fc-bbbb-edba8e44562d",
    "firstName": "Ali",
    "lastName": "Alio",
    "emailAddress": "aa@contoso.com",
    "externalId": "6a9b7f65-ffff-442f-b3b8-58a35abd1bcd",
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateUserRequest"
}
```

Blockchain Workbench возвращает ответ со следующими полями:

| **имя**;              | **Описание**                                                                                                             |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------------|
| requestId             | GUID, предоставляемый клиентом |
| userId                | Идентификатор созданного пользователя |
| userChainIdentifier   | Адрес пользователя, который был создан в сети блокчейна. В Ethereum адрес — это адрес **локальной цепочки** пользователя. |
| connectionId          | Уникальный идентификатор подключения блокчейна|
| messageSchemaVersion  | Версия схемы службы сообщений |
| messageName           | **CreateUserUpdate** |
| status                | Состояние запроса на создание пользователя.  При успешном выполнении значение будет **Success**. При сбое значение будет **Failure**.     |
| additionalInformation | Предоставляемые дополнительные сведения на основе состояния |

Пример ответа при успешном выполнении запроса на **создание пользователя** из Blockchain Workbench:

``` json
{ 
    "requestId": "e2264523-6147-41fc-bb59-edba8e44562d", 
    "userId": 15, 
    "userChainIdentifier": "0x9a8DDaCa9B7488683A4d62d0817E965E8f248398", 
    "connectionId": 1, 
    "messageSchemaVersion": "1.0.0", 
    "messageName": "CreateUserUpdate", 
    "status": "Success", 
    "additionalInformation": { } 
} 
```

Если запрос не удалось выполнить, в дополнительных сведениях будут указаны подробности сбоя.

``` json
{
    "requestId": "e2264523-6147-41fc-bb59-edba8e44562d", 
    "userId": 15, 
    "userChainIdentifier": null, 
    "connectionId": 1, 
    "messageSchemaVersion": "1.0.0", 
    "messageName": "CreateUserUpdate", 
    "status": "Failure", 
    "additionalInformation": { 
        "errorCode": 4000, 
        "errorMessage": "User cannot be provisioned on connection." 
    }
}
```

### <a name="create-contract"></a>Создание контракта

Создание нового контракта.

Для данного запроса требуются следующие поля:

| **имя**;             | **Описание**                                                                                                           |
|----------------------|---------------------------------------------------------------------------------------------------------------------------|
| requestId            | GUID, предоставляемый клиентом |
| userChainIdentifier  | Адрес пользователя, который был создан в сети блокчейна. В Ethereum это адрес пользователя **в цепочке**. |
| applicationName      | Имя приложения |
| version              | Версия приложения. Требуется, если у вас есть несколько версий включенного приложения. В противном случае версия является необязательной. Дополнительные сведения об управлении версиями приложения см. в разделе статьи [Управление версиями приложения в Azure Blockchain Workbench](version-app.md). |
| workflowName         | Имя рабочего процесса |
| parameters           | Параметры для создания контракта |
| connectionId         | Уникальный идентификатор подключения блокчейна |
| messageSchemaVersion | Версия схемы службы сообщений |
| messageName          | **CreateContractRequest** |

Пример

``` json
{ 
    "requestId": "ce3c429b-a091-4baa-b29b-5b576162b211", 
    "userChainIdentifier": "0x9a8DDaCa9B7488683A4d62d0817E965E8f248398", 
    "applicationName": "AssetTransfer",
    "version": "1.0",
    "workflowName": "AssetTransfer", 
    "parameters": [ 
        { 
            "name": "description", 
            "value": "a 1969 dodge charger" 
        }, 
        { 
            "name": "price", 
            "value": "12345" 
        } 
    ], 
    "connectionId": 1, 
    "messageSchemaVersion": "1.0.0", 
    "messageName": "CreateContractRequest" 
}
```

Blockchain Workbench возвращает ответ со следующими полями:

| **имя**;                 | **Описание**                                                                   |
|--------------------------|-----------------------------------------------------------------------------------|
| requestId                | GUID, предоставляемый клиентом                                                             |
| contractId               | Уникальный идентификатор контракта в Azure Blockchain Workbench |
| contractLedgerIdentifier | Адрес контракта в реестре                                            |
| connectionId             | Уникальный идентификатор подключения блокчейна                               |
| messageSchemaVersion     | Версия схемы службы сообщений                                                         |
| messageName              | **CreateContractUpdate**                                                      |
| status                   | Состояние запроса на создание контракта.  Возможные значения: **Submitted**, **Committed**, **Failure**.  |
| additionalInformation    | Предоставляемые дополнительные сведения на основе состояния                              |

Пример ответа при отправленном запросе на **создание контракта** из Blockchain Workbench:

``` json
{
    "requestId": "ce3c429b-a091-4baa-b29b-5b576162b211",
    "contractId": 55,
    "contractLedgerIdentifier": "0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe",
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractUpdate",
    "status": "Submitted",
    "additionalInformation": { }
}
```

Пример ответа при исполненном запросе на **создание контракта** из Blockchain Workbench:

``` json
{
    "requestId": "ce3c429b-a091-4baa-b29b-5b576162b211",
    "contractId": 55,
    "contractLedgerIdentifier": "0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe",
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractUpdate",
    "status": "Committed",
    "additionalInformation": { }
}
```

Если запрос не удалось выполнить, в дополнительных сведениях будут указаны подробности сбоя.

``` json
{
    "requestId": "ce3c429b-a091-4baa-b29b-5b576162b211",
    "contractId": 55,
    "contractLedgerIdentifier": null,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractUpdate",
    "status": "Failure",
    "additionalInformation": {
        "errorCode": 4000,
        "errorMessage": "Contract cannot be provisioned on connection."
    }
}
```

### <a name="create-contract-action"></a>Создание действия контракта

Создает действие контракта.

Для данного запроса требуются следующие поля:

| **имя**;                 | **Описание**                                                                                                           |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------|
| requestId                | GUID, предоставляемый клиентом |
| userChainIdentifier      | Адрес пользователя, который был создан в сети блокчейна. В Ethereum это адрес пользователя **в цепочке**. |
| contractLedgerIdentifier | Адрес контракта в реестре |
| version                  | Версия приложения. Требуется, если у вас есть несколько версий включенного приложения. В противном случае версия является необязательной. Дополнительные сведения об управлении версиями приложения см. в разделе статьи [Управление версиями приложения в Azure Blockchain Workbench](version-app.md). |
| workflowFunctionName     | Имя функции рабочего процесса |
| parameters               | Параметры для создания контракта |
| connectionId             | Уникальный идентификатор подключения блокчейна |
| messageSchemaVersion     | Версия схемы службы сообщений |
| messageName              | **CreateContractActionRequest** |

Пример

``` json
{
    "requestId": "a5530932-9d6b-4eed-8623-441a647741d3",
    "userChainIdentifier": "0x9a8DDaCa9B7488683A4d62d0817E965E8f248398",
    "contractLedgerIdentifier": "0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe",
    "version": "1.0",
    "workflowFunctionName": "modify",
    "parameters": [
        {
            "name": "description",
            "value": "a 1969 dodge charger"
        },
        {
            "name": "price",
            "value": "12345"
        }
    ],
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractActionRequest"
}
```

Blockchain Workbench возвращает ответ со следующими полями:

| **имя**;              | **Описание**                                                                   |
|-----------------------|-----------------------------------------------------------------------------------|
| requestId             | GUID, предоставляемый клиентом|
| contractId            | Уникальный идентификатор контракта в Azure Blockchain Workbench |
| connectionId          | Уникальный идентификатор подключения блокчейна |
| messageSchemaVersion  | Версия схемы службы сообщений |
| messageName           | **CreateContractActionUpdate** |
| status                | Состояние запроса на создание действия контракта. Возможные значения: **Submitted**, **Committed**, **Failure**.                         |
| additionalInformation | Предоставляемые дополнительные сведения на основе состояния |

Пример ответа при отправленном запросе на **создание действия контракта** из Blockchain Workbench:

``` json
{
    "requestId": "a5530932-9d6b-4eed-8623-441a647741d3",
    "contractId": 105,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractActionUpdate",
    "status": "Submitted",
    "additionalInformation": { }
}
```

Пример ответа при исполненном запросе на **создание действия контракта** из Blockchain Workbench:

``` json
{
    "requestId": "a5530932-9d6b-4eed-8623-441a647741d3",
    "contractId": 105,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractActionUpdate",
    "status": "Committed",
    "additionalInformation": { }
}
```

Если запрос не удалось выполнить, в дополнительных сведениях будут указаны подробности сбоя.

``` json
{
    "requestId": "a5530932-9d6b-4eed-8623-441a647741d3",
    "contractId": 105,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractActionUpdate",
    "status": "Failure",
    "additionalInformation": {
        "errorCode": 4000,
        "errorMessage": "Contract action cannot be provisioned on connection."
    }
}
```

### <a name="input-api-error-codes-and-messages"></a>Коды и сообщения ошибок API ввода

**Код ошибки 4000. Недопустимый запрос**
- Недопустимое connectionId
- Не удалось выполнить десериализацию CreateUserRequest
- Не удалось выполнить десериализацию CreateContractRequest
- Не удалось выполнить десериализацию CreateContractActionRequest
- Приложение {определяемое по имени} не существует
- В приложении {определяемому по имени} нет рабочего процесса
- UserChainIdentifier не существует
- Контракт {определяемый по идентификатору реестра} не существует
- Контракт {определяемый по идентификатору реестра} не имеет функции {имя функции рабочего процесса}
- UserChainIdentifier не существует

**Код ошибки 4090. Ошибка из-за конфликта**
- Пользователь уже существует
- Контракт уже существует
- Действие контракта уже существует

**Код ошибки 5000. Внутренняя ошибка сервера**
- Сообщения об исключении

## <a name="event-notifications"></a>Уведомления о событиях

Уведомления о событиях можно использовать для уведомления пользователей и подчиненных систем о событиях, происходящих в Blockchain Workbench и сети блокчейна, к которой оно подключено. Уведомление о событии можно использовать непосредственно в коде или с такими инструментами, как Logic Apps и Flow, чтобы активировать поток данных в подчиненные системы.

Дополнительные сведения о различных сообщениях, которые могут быть получены, см. в [справочниках по сообщениям об уведомлениях](#notification-message-reference).

### <a name="consuming-event-grid-events-with-azure-functions"></a>Использование событий службы "Сетка событий Azure" с решением "Функции Azure"

Если пользователь хочет получать уведомления о событиях, которые происходят в Blockchain Workbench, через службу "Сетка событий",события из этой службы можно получать, используя решение "Функции Azure".

1. Создайте **приложение-функция Azure** в портал Azure.
2. Создайте функцию.
3. Найдите шаблон для службы "Сетка событий". Будет показан базовый код шаблона для чтения сообщения. При необходимости измените код.
4. Сохраните функцию. 
5. Выберите службу "Сетка событий" из группы ресурсов Blockchain Workbench.

### <a name="consuming-event-grid-events-with-logic-apps"></a>Использование событий службы "Сетка событий Azure" со службой Logic Apps

1. Создайте **приложение логики Azure** на портале Azure.
2. При открытии этого приложения на портале Azure будет предложено выбрать триггер. Выберите **Azure Event Grid -- When a resource event occurs** (Сетка событий Azure — при возникновении события ресурса).
3. При появлении конструктора рабочих процессов вам будет предложено войти в систему.
4. Выберите подписку. Вам нужен ресурс в виде **Microsoft.EventGrid.Topics**. Выберите **имя ресурса** среди доступных в группе ресурсов Azure Blockchain Workbench.
5. Выберите службу "Сетка событий" из группы ресурсов Blockchain Workbench.

## <a name="using-service-bus-topics-for-notifications"></a>Использование разделов служебной шины для уведомлений

Разделы служебной шины могут использоваться для уведомления пользователей о событиях, произошедших в Blockchain Workbench. 

1. Перейдите к служебной шине в группе ресурсов Workbench.
2. Выберите **Разделы**.
3. Выберите **egress-topic**.
4. Создайте подписку для этого раздела. Получите ключ для нее.
5. Создайте программу, которая подписывается на события из этой подписки.

### <a name="consuming-service-bus-messages-with-logic-apps"></a>Использование сообщений служебной шины с Logic Apps

1. Создайте **приложение логики Azure** на портале Azure.
2. При открытии этого приложения на портале Azure будет предложено выбрать триггер. Введите **Служебная шина** в поле поиска и выберите соответствующий триггер для типа взаимодействия со служебной шиной. Например, **Service Bus -- When a message is received in a topic subscription (auto-complete)** (Служебная шина — при получении сообщения в подписке раздела (автозавершение)).
3. При отображении конструктора рабочих процессов укажите информацию о подключении для служебной шины.
4. Выберите подписку и укажите раздел **workbench-external**.
5. Разработайте логику приложения, которая использует сообщения из этого триггера.

## <a name="notification-message-reference"></a>Справочник по сообщениям об уведомлениях

В зависимости от **мессаженаме** сообщения уведомления имеют один из следующих типов сообщений.

### <a name="block-message"></a>Сообщение о блоке

Содержит информацию об отдельных блоках. *BlockMessage* включает раздел с информацией об уровне блока и раздел с информацией о транзакциях.

| Имя | Описание |
|------|-------------|
| block | Содержит [информацию о блоке](#block-information) |
| транзакции | Содержит коллекцию [сведений о транзакциях](#transaction-information) для блока |
| connectionId | Уникальный идентификатор подключения |
| messageSchemaVersion | Версия схемы службы сообщений |
| messageName | **BlockMessage** |
| additionalInformation | Предоставленные дополнительные сведения |

#### <a name="block-information"></a>Информация о блоке

| Имя              | Описание |
|-------------------|-------------|
| blockId           | Уникальный идентификатор блока в Azure Blockchain Workbench |
| blockNumber       | Уникальный идентификатор блока в реестре |
| blockHash         | Хэш блока |
| previousBlockHash | Хэш-код предыдущего блока |
| blockTimestamp    | Метка времени блока |

#### <a name="transaction-information"></a>Сведения о транзакциях

| Имя               | Описание |
|--------------------|-------------|
| transactionId      | Уникальный идентификатор транзакции в Azure Blockchain Workbench |
| transactionHash    | Хэш транзакции в реестре |
| из               | Уникальный идентификатор в реестре для исходного объекта транзакции |
| значение                 | Уникальный идентификатор в реестре для целевого объекта транзакции |
| provisioningStatus | Определяет текущее состояние процесса подготовки для транзакции. Возможны следующие значения: </br>0 — действие транзакции было создано в базе данных с помощью API</br>1 — транзакция была отправлена распределенному реестру</br>2 — транзакция успешно зафиксирована в распределенном реестре;</br>3 или 4 — транзакцию не удалось зафиксировать в реестре;</br>5 — транзакция успешно зафиксирована в распределенном реестре. |

Пример сообщения *BlockMessage* из Blockchain Workbench:

``` json
{
    "block": {
        "blockId": 123,
        "blockNumber": 1738312,
        "blockHash": "0x03a39411e25e25b47d0ec6433b73b488554a4a5f6b1a253e0ac8a200d13fffff",
        "previousBlockHash": null,
        "blockTimestamp": "2018-10-09T23:35:58Z",
    },
    "transactions": [
        {
            "transactionId": 234,
            "transactionHash": "0xa4d9c95b581f299e41b8cc193dd742ef5a1d3a4ddf97bd11b80d123fec27ffff",
            "from": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60dffff",
            "to": null,
            "provisioningStatus": 1
        },
        {
            "transactionId": 235,
            "transactionHash": "0x5c1fddea83bf19d719e52a935ec8620437a0a6bdaa00ecb7c3d852cf92e1ffff",
            "from": "0xadd97e1e595916e29ea94fda894941574000ffff",
            "to": "0x9a8DDaCa9B7488683A4d62d0817E965E8f24ffff",
            "provisioningStatus": 2
        }
    ],
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "BlockMessage",
    "additionalInformation": {}
}
```

### <a name="contract-message"></a>Сообщение о контракте

Содержит информацию о контракте. Это сообщение включает раздел свойств контракта и раздел с информацией о транзакциях. В раздел транзакций включаются все транзакции, которые изменяли контракт для конкретного блока.

| Имя | Описание |
|------|-------------|
| blockId | Уникальный идентификатор блока в Azure Blockchain Workbench |
| blockHash | Хэш блока |
| modifyingTransactions | [Транзакции, которые изменяли](#modifying-transaction-information) этот контракт |
| contractId | Уникальный идентификатор контракта в Azure Blockchain Workbench |
| contractLedgerIdentifier | Уникальный идентификатор контракта в реестре |
| contractProperties | [Свойства контракта](#contract-properties) |
| isNewContract | Указывает, является ли этот контракт новым. Возможные значения: true — если это новый контракт; false — если это обновление существующего контракта. |
| connectionId | Уникальный идентификатор подключения |
| messageSchemaVersion | Версия схемы службы сообщений |
| messageName | **ContractMessage** |
| additionalInformation | Предоставленные дополнительные сведения |

#### <a name="modifying-transaction-information"></a>Изменение сведений о транзакциях

| Имя               | Описание |
|--------------------|-------------|
| transactionId | Уникальный идентификатор транзакции в Azure Blockchain Workbench |
| transactionHash | Хэш транзакции в реестре |
| из | Уникальный идентификатор в реестре для исходного объекта транзакции |
| значение | Уникальный идентификатор в реестре для целевого объекта транзакции |

#### <a name="contract-properties"></a>Свойства контракта

| Имя               | Описание |
|--------------------|-------------|
| workflowPropertyId | Уникальный идентификатор свойства рабочего процесса в Azure Blockchain Workbench |
| name | Имя свойства рабочего процесса |
| value | Значение свойства рабочего процесса |

Пример сообщения *ContractMessage* из Blockchain Workbench:

``` json
{
    "blockId": 123,
    "blockhash": "0x03a39411e25e25b47d0ec6433b73b488554a4a5f6b1a253e0ac8a200d13fffff",
    "modifyingTransactions": [
        {
            "transactionId": 234,
            "transactionHash": "0x5c1fddea83bf19d719e52a935ec8620437a0a6bdaa00ecb7c3d852cf92e1ffff",
            "from": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60dffff",
            "to": "0xf8559473b3c7197d59212b401f5a9f07ffff"
        },
        {
            "transactionId": 235,
            "transactionHash": "0xa4d9c95b581f299e41b8cc193dd742ef5a1d3a4ddf97bd11b80d123fec27ffff",
            "from": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60dffff",
            "to": "0xf8559473b3c7197d59212b401f5a9f07b429ffff"
        }
    ],
    "contractId": 111,
    "contractLedgerIdentifier": "0xf8559473b3c7197d59212b401f5a9f07b429ffff",
    "contractProperties": [
        {
            "workflowPropertyId": 1,
            "name": "State",
            "value": "0"
        },
        {
            "workflowPropertyId": 2,
            "name": "Description",
            "value": "1969 Dodge Charger"
        },
        {
            "workflowPropertyId": 3,
            "name": "AskingPrice",
            "value": "30000"
        },
        {
            "workflowPropertyId": 4,
            "name": "OfferPrice",
            "value": "0"
        },
        {
            "workflowPropertyId": 5,
            "name": "InstanceAppraiser",
            "value": "0x0000000000000000000000000000000000000000"
        },
        {
            "workflowPropertyId": 6,
            "name": "InstanceBuyer",
            "value": "0x0000000000000000000000000000000000000000"
        },
        {
            "workflowPropertyId": 7,
            "name": "InstanceInspector",
            "value": "0x0000000000000000000000000000000000000000"
        },
        {
            "workflowPropertyId": 8,
            "name": "InstanceOwner",
            "value": "0x9a8DDaCa9B7488683A4d62d0817E965E8f24ffff"
        },
        {
            "workflowPropertyId": 9,
            "name": "ClosingDayOptions",
            "value": "[21,48,69]"
        }
    ],
    "isNewContract": false,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "ContractMessage",
    "additionalInformation": {}
}
```

### <a name="event-message-contract-function-invocation"></a>Сообщение о событии: вызов функции контракта

Содержит сведения о вызове функции контракта, в том числе имя функции, входные параметры и сведения о вызывающем объекте.

| Имя | Описание |
|------|-------------|
| eventName                   | **ContractFunctionInvocation** |
| caller                      | [Сведения о вызывающем объекте](#caller-information) |
| contractId                  | Уникальный идентификатор контракта в Azure Blockchain Workbench |
| contractLedgerIdentifier    | Уникальный идентификатор контракта в реестре |
| functionName                | Имя функции |
| parameters                  | [Сведения о параметрах](#parameter-information) |
| transaction                 | Сведения о транзакциях |
| inTransactionSequenceNumber | Порядковый номер транзакции в блоке |
| connectionId                | Уникальный идентификатор подключения |
| messageSchemaVersion        | Версия схемы службы сообщений |
| messageName                 | **EventMessage** |
| additionalInformation       | Предоставленные дополнительные сведения |

#### <a name="caller-information"></a>Сведения о вызывающем объекте

| Имя | Описание |
|------|-------------|
| type | Тип вызывающего объекта, например пользователь или контракт |
| идентификатор | Уникальный идентификатор вызывающего объекта в Azure Blockchain Workbench |
| ledgerIdentifier | Уникальный идентификатор вызывающего объекта в реестре |

#### <a name="parameter-information"></a>Сведения о параметрах

| Имя | Описание |
|------|-------------|
| name | Имя параметра |
| value | Значение параметра |

#### <a name="event-message-transaction-information"></a>Сведения о транзакциях в сообщении о событии

| Имя               | Описание |
|--------------------|-------------|
| transactionId      | Уникальный идентификатор транзакции в Azure Blockchain Workbench |
| transactionHash    | Хэш транзакции в реестре |
| из               | Уникальный идентификатор в реестре для исходного объекта транзакции |
| значение                 | Уникальный идентификатор в реестре для целевого объекта транзакции |

Пример *EventMessage ContractFunctionInvocation* из Blockchain Workbench:

``` json
{
    "eventName": "ContractFunctionInvocation",
    "caller": {
        "type": "User",
        "id": 21,
        "ledgerIdentifier": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60ffff"
    },
    "contractId": 34,
    "contractLedgerIdentifier": "0xf8559473b3c7197d59212b401f5a9f07b429ffff",
    "functionName": "Modify",
    "parameters": [
        {
            "name": "description",
            "value": "a new description"
        },
        {
            "name": "price",
            "value": "4567"
        }
    ],
    "transaction": {
        "transactionId": 234,
        "transactionHash": "0x5c1fddea83bf19d719e52a935ec8620437a0a6bdaa00ecb7c3d852cf92e1ffff",
        "from": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60dffff",
        "to": "0xf8559473b3c7197d59212b401f5a9f07b429ffff"
    },
    "inTransactionSequenceNumber": 1,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "EventMessage",
    "additionalInformation": { }
}
```

### <a name="event-message-application-ingestion"></a>Сообщение о событии: прием приложения

Содержит сведения об отправке приложения в Workbench, например имя и версия отправляемого приложения.

| Имя | Описание |
|------|-------------|
| eventName | **ApplicationIngestion** |
| applicationId | Уникальный идентификатор приложения в Azure Blockchain Workbench |
| applicationName | имя приложения; |
| applicationDisplayName | Отображаемое имя приложения |
| applicationVersion | Версия приложения |
| applicationDefinitionLocation | URL-адрес, по которому размещается файл конфигурации приложения |
| contractCodes | Коллекция [кодов контракта](#contract-code-information) для приложения |
| applicationRoles | Коллекция [ролей приложения](#application-role-information) для приложения |
| applicationWorkflows | Коллекция [рабочих процессов приложения](#application-workflow-information) для приложения |
| connectionId | Уникальный идентификатор подключения |
| messageSchemaVersion | Версия схемы службы сообщений |
| messageName | **EventMessage** |
| additionalInformation | Здесь представлены дополнительные сведения, в том числе состояния рабочего процесса приложения и сведения о переходе. |

#### <a name="contract-code-information"></a>Сведения о коде контракта

| Имя | Описание |
|------|-------------|
| идентификатор | Уникальный идентификатор файла кода контракта в Azure Blockchain Workbench |
| ledgerId | Уникальный идентификатор реестра в Azure Blockchain Workbench |
| location | URL-адрес, по которому расположен файл кода контракта |

#### <a name="application-role-information"></a>Сведения о роли приложения

| Имя | Описание |
|------|-------------|
| идентификатор | Уникальный идентификатор роли приложения в Azure Blockchain Workbench |
| name | Имя роли приложения |

#### <a name="application-workflow-information"></a>Сведения о рабочем процессе приложения

| Имя | Описание |
|------|-------------|
| идентификатор | Уникальный идентификатор рабочего процесса приложения в Azure Blockchain Workbench |
| name | Имя рабочего процесса приложения |
| displayName | Отображаемое имя рабочего процесса приложения |
| functions | Коллекция [функций для рабочего процесса приложения](#workflow-function-information)|
| штаты | Коллекция [состояний для рабочего процесса приложения](#workflow-state-information) |
| properties | [Сведения о свойствах рабочего процесса](#workflow-property-information) приложения |

##### <a name="workflow-function-information"></a>Сведения о функции рабочего процесса

| Имя | Описание |
|------|-------------|
| идентификатор | Уникальный идентификатор функции рабочего процесса приложения в Azure Blockchain Workbench |
| name | Имя функции |
| parameters | Параметры для функции |

##### <a name="workflow-state-information"></a>Сведения о состоянии рабочего процесса

| Имя | Описание |
|------|-------------|
| name | Имя состояния |
| displayName | Отображаемое имя состояния |
| Стиль | Стиль состояния (успех или сбой) |

##### <a name="workflow-property-information"></a>Сведения о свойствах рабочего процесса

| Имя | Описание |
|------|-------------|
| идентификатор | Уникальный идентификатор свойства рабочего процесса приложения в Azure Blockchain Workbench |
| name | Имя свойства |
| тип | Тип свойства |

Пример *EventMessage ApplicationIngestion* в Blockchain Workbench:

``` json
{
    "eventName": "ApplicationIngestion",
    "applicationId": 31,
    "applicationName": "AssetTransfer",
    "applicationDisplayName": "Asset Transfer",
    "applicationVersion": "1.0",
    "applicationDefinitionLocation": "http://url",
    "contractCodes": [
        {
            "id": 23,
            "ledgerId": 1,
            "location": "http://url"
        }
    ],
    "applicationRoles": [
            {
                "id": 134,
                "name": "Buyer"
            },
            {
                "id": 135,
                "name": "Seller"
            }
       ],
    "applicationWorkflows": [
        {
            "id": 89,
            "name": "AssetTransfer",
            "displayName": "Asset Transfer",
            "functions": [
                {
                    "id": 912,
                    "name": "",
                    "parameters": [
                        {
                            "name": "description",
                            "type": {
                                "name": "string"
                             }
                        },
                        {
                            "name": "price",
                            "type": {
                                "name": "int"
                            }
                        }
                    ]
                },
                {
                    "id": 913,
                    "name": "modify",
                    "parameters": [
                        {
                            "name": "description",
                            "type": {
                                "name": "string"
                             }
                        },
                        {
                            "name": "price",
                            "type": {
                                "name": "int"
                            }
                        }
                    ]
                }
            ],
            "states": [ 
                 {
                      "name": "Created",
                      "displayName": "Created",
                      "style" : "Success"
                 },
                 {
                      "name": "Terminated",
                      "displayName": "Terminated",
                      "style" : "Failure"
                 }
            ],
            "properties": [
                {
                    "id": 879,
                    "name": "Description",
                    "type": {
                        "name": "string"
                     }
                },
                {
                    "id": 880,
                    "name": "Price",
                    "type": {
                        "name": "int"
                     }
                }
            ]
        }
    ],
    "connectionId": [ ],
    "messageSchemaVersion": "1.0.0",
    "messageName": "EventMessage",
    "additionalInformation":
        {
            "states" :
            [
                {
                    "Name": "BuyerAccepted",
                    "Transitions": [
                        {
                            "DisplayName": "Accept",
                            "AllowedRoles": [ ],
                            "AllowedInstanceRoles": [ "InstanceOwner" ],
                            "Function": "Accept",
                            "NextStates": [ "SellerAccepted" ]
                        }
                    ]
                }
            ]
        }
}
```

### <a name="event-message-role-assignment"></a>Сообщение о событии: назначение роли

Содержит сведения о назначении пользователю роли в Workbench, в том числе идентификатор того, кто выполнил назначение роли, имя роли и соответствующее приложение.

| Имя | Описание |
|------|-------------|
| eventName | **RoleAssignment** |
| applicationId | Уникальный идентификатор приложения в Azure Blockchain Workbench |
| applicationName | имя приложения; |
| applicationDisplayName | Отображаемое имя приложения |
| applicationVersion | Версия приложения |
| applicationRole        | Сведения о [роли приложения](#roleassignment-application-role) |
| assigner               | Информация о [том, кто назначил роль](#roleassignment-assigner) |
| assignee               | Информация о [том, кому назначена роль](#roleassignment-assignee) |
| connectionId           | Уникальный идентификатор подключения |
| messageSchemaVersion   | Версия схемы службы сообщений |
| messageName            | **EventMessage** |
| additionalInformation  | Предоставленные дополнительные сведения |

#### <a name="roleassignment-application-role"></a>Роль приложения RoleAssignment

| Имя | Описание |
|------|-------------|
| идентификатор | Уникальный идентификатор роли приложения в Azure Blockchain Workbench |
| name | Имя роли приложения |

#### <a name="roleassignment-assigner"></a>Кто назначил RoleAssignment

| Имя | Описание |
|------|-------------|
| идентификатор | Уникальный идентификатор пользователя в Azure Blockchain Workbench |
| тип | Тип назначающего |
| chainIdentifier | Уникальный идентификатор пользователя в реестре |

#### <a name="roleassignment-assignee"></a>Кому назначается RoleAssignment

| Имя | Описание |
|------|-------------|
| идентификатор | Уникальный идентификатор пользователя в Azure Blockchain Workbench |
| тип | Тип субъекта назначения |
| chainIdentifier | Уникальный идентификатор пользователя в реестре |

Пример *EventMessage RoleAssignment* в Blockchain Workbench:

``` json
{
    "eventName": "RoleAssignment",
    "applicationId": 31,
    "applicationName": "AssetTransfer",
    "applicationDisplayName": "Asset Transfer",
    "applicationVersion": "1.0",
    "applicationRole": {
        "id": 134,
        "name": "Buyer"
    },
    "assigner": {
        "id": 1,
        "type": null,
        "chainIdentifier": "0xeFFC7766d38aC862d79706c3C5CEEf089564ffff"
    },
    "assignee": {
        "id": 3,
        "type": null,
        "chainIdentifier": "0x9a8DDaCa9B7488683A4d62d0817E965E8f24ffff"
    },
    "connectionId": [ ],
    "messageSchemaVersion": "1.0.0",
    "messageName": "EventMessage",
    "additionalInformation": { }
}
```

## <a name="next-steps"></a>Дальнейшие действия

- [Шаблоны интеграции смарт-контрактов](integration-patterns.md)
