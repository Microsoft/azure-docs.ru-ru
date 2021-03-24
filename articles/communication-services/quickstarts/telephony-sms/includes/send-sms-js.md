---
title: включить файл
description: включить файл
services: azure-communication-services
author: bertong
manager: ankita
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/11/2021
ms.topic: include
ms.custom: include file
ms.author: bertong
ms.openlocfilehash: 0d142c477e1de2a2a34a8abfd948800cc0b607ee
ms.sourcegitcommit: 27cd3e515fee7821807c03e64ce8ac2dd2dd82d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103622026"
---
Начало работы со Службами коммуникации Azure с помощью клиентской библиотеки SMS Служб коммуникации Azure для JavaScript для отправки SMS-сообщений.

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

<!--**TODO: update all these reference links as the resources go live**

[API reference documentation](../../../references/overview.md) | [Library source code](https://github.com/Azure/azure-sdk-for-js-pr/tree/feature/communication/sdk/communication/communication-sms) | [Package (NPM)](https://www.npmjs.com/package/@azure/communication-sms) | [Samples](#todo-samples)-->

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [Node.js](https://nodejs.org/), активная версия LTS и версия Maintenance LTS (рекомендуются версии 8.11.1 и 10.14.1).
- Активный ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)
- Номер телефона с поддержкой SMS-сообщений. [Получите номер телефона.](../get-phone-number.md)

### <a name="prerequisite-check"></a>Проверка предварительных условий

- В терминале или командном окне воспользуйтесь `node --version`, чтобы проверить, установлен ли пакет Node.js.
- Чтобы просмотреть номера телефонов, связанные с ресурсом Служб коммуникации, войдите на [портал Azure](https://portal.azure.com/), перейдите к ресурсу Служб коммуникации и откройте вкладку с **номерами телефонов** в области навигации слева.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-nodejs-application"></a>Создание нового приложения Node.js

Откройте терминал или командное окно, создайте каталог для своего приложения и перейдите к нему.

```console
mkdir sms-quickstart && cd sms-quickstart
```

Воспользуйтесь командой `npm init -y`, чтобы создать файл **package.json** с параметрами по умолчанию.

```console
npm init -y
```

Используйте текстовый редактор, чтобы создать файл с именем **send-sms.js** в корневом каталоге проекта. В следующих разделах показано, как добавить в этот файл весь исходный код для примера из этого краткого руководства.

### <a name="install-the-package"></a>Установка пакета

Используйте команду `npm install`, чтобы установить клиентскую библиотеку SMS Служб коммуникации для реализации вызовов на JavaScript.

```console
npm install @azure/communication-sms --save
```

Параметр `--save` указывает библиотеку как зависимость в файле пакета **package.json**.

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции клиентской библиотеки Служб коммуникации SMS для Node.js.

| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| SmsClient | Этот класс требуется для реализации всех функций обмена текстовыми сообщениями. Его экземпляр можно создать на основе сведений о подписке и использовать для отправки SMS. |
| SmsSendResult               | Этот класс содержит результат, полученный от службы SMS.                                          |
| SmsSendOptions | Этот интерфейс предоставляет параметры для настройки отчетов о доставке. Если `enableDeliveryReport` имеет значение `true`, при успешной доставке будет создано событие. |
| SmsSendRequest | Этот интерфейс является моделью для создания запроса SMS (например, настройки номеров телефонов для отправки и получения содержимого текстовых сообщений). |

## <a name="authenticate-the-client"></a>Аутентификация клиента

Импортируйте **SmsClient** из клиентской библиотеки и создайте его экземпляр с помощью строки подключения. Приведенный ниже код извлекает строку подключения для ресурса из переменной среды с именем `COMMUNICATION_SERVICES_CONNECTION_STRING`. См. сведения о том, как [управлять строкой подключения ресурса](../../create-communication-resource.md#store-your-connection-string).

Добавьте в **send-sms.js** следующий код:

```javascript
const { SmsClient } = require('@azure/communication-sms');

// This code demonstrates how to fetch your connection string
// from an environment variable.
const connectionString = process.env['COMMUNICATION_SERVICES_CONNECTION_STRING'];

// Instantiate the SMS client
const smsClient = new SmsClient(connectionString);
```

## <a name="send-a-1n-sms-message"></a>Отправка группового текстового сообщения

Чтобы отправить текстовое сообщение списку получателей, вызовите функцию `send` из SmsClient со списком телефонных номеров получателей (если вы хотите отправить сообщение одному получателю, укажите только один номер из списка). Добавьте следующий код в конец функции **send-sms.js**:

```javascript
async function main() {
  const sendResults = await smsClient.send({
    from: "<from-phone-number>",
    to: ["<to-phone-number-1>", "<to-phone-number-2>"],
    message: "Hello World 👋🏻 via SMS"
  });

  // individual messages can encounter errors during sending
  // use the "successful" property to verify
  for (const sendResult of sendResults) {
    if (sendResult.successful) {
      console.log("Success: ", sendResult);
    } else {
      console.error("Something went wrong when trying to send this message: ", sendResult);
    }
  }
}

main();
```
`<from-phone-number>` необходимо заменить номером телефона с поддержкой SMS, связанным с ресурсом Служб коммуникации, а `<to-phone-number>` — номером телефона, на который вы хотите отправить сообщение.

## <a name="send-a-1n-sms-message-with-options"></a>Отправка группового текстового сообщения с параметрами

Кроме того, вы можете передать объект параметров для указания того, должен ли быть включен отчет о доставке, а также задать пользовательские теги.

```javascript

async function main() {
  await smsClient.send({
    from: "<from-phone-number>",
    to: ["<to-phone-number-1>", "<to-phone-number-2>"],
    message: "Weekly Promotion!"
  }, {
    //Optional parameter
    enableDeliveryReport: true,
    tag: "marketing"
  });

  // individual messages can encounter errors during sending
  // use the "successful" property to verify
  for (const sendResult of sendResults) {
    if (sendResult.successful) {
      console.log("Success: ", sendResult);
    } else {
      console.error("Something went wrong when trying to send this message: ", sendResult);
    }
  }
}

main();
```

Параметр `enableDeliveryReport` является необязательным. Его можно использовать для настройки отчетов о доставке. Это полезно, если вы хотите, чтобы при доставке SMS-сообщений создавались события. Сведения о настройке отчетов о доставке SMS-сообщений см. в кратком руководстве [Обработка событий SMS-сообщений](../handle-sms-events.md).
`tag` — это необязательный параметр, который можно использовать для применения тегов к отчету о доставке.

## <a name="run-the-code"></a>Выполнение кода

Используйте команду `node`, чтобы выполнить код, добавленный в файл **send-sms.js**.

```console

node ./send-sms.js

```
