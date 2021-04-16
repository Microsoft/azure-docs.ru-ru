---
ms.openlocfilehash: 956c92c5c020f892b8148e9d43d403b1099fbdba
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106113400"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [Node.js](https://nodejs.org/), активная версия LTS и версия Maintenance LTS (рекомендуются версии 8.11.1 и 10.14.1).
- Активный ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)

### <a name="prerequisite-check"></a>Проверка предварительных условий

- В терминале или командном окне воспользуйтесь `node --version`, чтобы проверить, установлен ли пакет Node.js.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-nodejs-application"></a>Создание нового приложения Node.js

Откройте терминал или командное окно, создайте каталог для своего приложения и перейдите к нему.

```console
mkdir phone-numbers-quickstart && cd phone-numbers-quickstart
```

Воспользуйтесь командой `npm init -y`, чтобы создать файл **package.json** с параметрами по умолчанию.

```console
npm init -y
```

Создайте файл с именем **phone-numbers-quickstart.js** в корне только что созданного каталога. Добавьте в него следующий фрагмент кода:

```javascript
async function main() {
    // quickstart code will here
}

main();
```

### <a name="install-the-package"></a>Установка пакета

Используйте команду `npm install`, чтобы установить клиентскую библиотеку Служб коммуникации для номеров телефона для JavaScript.

```console
npm install @azure/communication-phone-numbers --save
```

Параметр `--save` добавляет библиотеку как зависимость в файле пакета **package.json**.

## <a name="authenticate-the-client"></a>Аутентификация клиента

Импортируйте **PhoneNumbersClient** из клиентской библиотеки и создайте его экземпляр с помощью строки подключения. Приведенный ниже код извлекает строку подключения для ресурса из переменной среды с именем `COMMUNICATION_SERVICES_CONNECTION_STRING`. См. сведения о том, как [управлять строкой подключения ресурса](../../create-communication-resource.md#store-your-connection-string).

Добавьте следующий код в начало файла **phone-numbers-quickstart.js**:

```javascript
const { PhoneNumbersClient } = require('@azure/communication-phone-numbers');

// This code demonstrates how to fetch your connection string
// from an environment variable.
const connectionString = process.env['COMMUNICATION_SERVICES_CONNECTION_STRING'];

// Instantiate the phone numbers client
const phoneNumbersClient = new PhoneNumbersClient(connectionString);
```

## <a name="manage-phone-numbers"></a>Управление номерами телефонов

### <a name="search-for-available-phone-numbers"></a>Поиск доступных номеров телефонов

Чтобы приобрести номера телефонов, сначала необходимо найти доступные номера. Для поиска номеров телефонов укажите код города, тип назначения, [возможности номеров телефона](../../../concepts/telephony-sms/plan-solution.md#phone-number-capabilities-in-azure-communication-services), [тип номера телефона](../../../concepts/telephony-sms/plan-solution.md#phone-number-types-in-azure-communication-services) и количество. Обратите внимание, что для типа бесплатного телефонного номера указывать код города необязательно.

Добавьте в функцию `main` следующий фрагмент кода:

```javascript
/**
 * Search for Available Phone Number
 */

// Create search request
const searchRequest = {
    countryCode: "US",
    phoneNumberType: "tollFree",
    assignmentType: "application",
    capabilities: {
      sms: "outbound",
      calling: "none"
    },
    areaCode: "833",
    quantity: 1
  };

const searchPoller = await phoneNumbersClient.beginSearchAvailablePhoneNumbers(searchRequest);

// The search is underway. Wait to receive searchId.
const { searchId, phoneNumbers } = searchPoller.pollUntilDone();
const phoneNumber = phoneNumbers[0];

console.log(`Found phone number: ${phoneNumber}`);
console.log(`searchId: ${searchId}`);
```

### <a name="purchase-phone-number"></a>Покупка номера телефона

Результатом поиска номеров телефона является `PhoneNumberSearchResult`. Он содержит `searchId`, который можно передать в API для покупки номеров, чтобы купить найденные номера. Обратите внимание, что при вызове API для покупки номеров телефона на учетную запись Azure будет начисляться платеж.

Добавьте в функцию `main` следующий фрагмент кода:

```javascript
/**
 * Purchase Phone Number
 */

const purchasePoller = await phoneNumbersClient.beginPurchasePhoneNumbers(searchId);

// Purchase is underway.
await purchasePoller.pollUntilDone();
console.log(`Successfully purchased ${phoneNumber}`);
```

### <a name="update-phone-number-capabilities"></a>Обновление возможностей номера телефона

После покупки номера телефона добавьте следующий код, чтобы обновить его возможности:

```javascript
/**
 * Update Phone Number Capabilities
 */

// Create update request.
// This will update phone number to send and receive sms, but only send calls.
const updateRequest = {
  sms: "inbound+outbound",
  calling: "outbound"
};

const updatePoller = await phoneNumbersClient.beginUpdatePhoneNumberCapabilities(
  phoneNumber,
  updateRequest
);

// Update is underway.
await updatePoller.pollUntilDone();
console.log("Phone number updated successfully.");
```

### <a name="get-purchased-phone-numbers"></a>Получение приобретенных номеров телефонов

После покупки номера его можно получить из клиента. Добавьте следующий код в функцию `main`, чтобы получить только что приобретенный номер телефона:

```javascript
/**
 * Get Purchased Phone Number
 */

const { capabilities } = await phoneNumbersClient.getPurchasedPhoneNumber(phoneNumber);
console.log(`These capabilities: ${capabilities}, should be the same as these: ${updateRequest}.`);
```

Вы также можете получить все приобретенные номера телефонов.

```javascript
const phoneNumbers = await phoneNumbersClient.listPurchasedPhoneNumbers();

for await (const purchasedPhoneNumber of phoneNumbers) {
  console.log(`Phone number: ${purchasedPhoneNumber.phoneNumber}, country code: ${purchasedPhoneNumber.countryCode}.`);
}
```

### <a name="release-phone-number"></a>Освобождение номера телефона

Теперь приобретенный номер телефона можно освободить. Добавьте приведенный ниже фрагмент кода в функцию `main`:

```javascript
/**
 * Release Purchased Phone Number
 */

const releasePoller = await client.beginReleasePhoneNumber(phoneNumber);

// Release is underway.
await releasePoller.pollUntilDone();
console.log("Successfully release phone number.");
```

## <a name="run-the-code"></a>Выполнение кода

Для выполнения кода, добавленного в файла **phone-numbers-quickstart.js**, используйте команду `node`.

```console
node phone-numbers-quickstart.js
```