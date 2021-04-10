---
title: Устранение неполадок в Службах коммуникации Azure
description: Узнайте, как получить сведения, необходимые для устранения неполадок в решении "Службы коммуникации".
author: manoskow
manager: jken
services: azure-communication-services
ms.author: manoskow
ms.date: 03/10/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 80db53a5ed8d2edc90bc847578d5df4d603cc437
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105107233"
---
# <a name="troubleshooting-in-azure-communication-services"></a>Устранение неполадок в Службах коммуникации Azure

Эта документация поможет вам устранить неполадки, которые могут возникнуть в решении "Службы коммуникации". Если вы устраняете неполадки с SMS, вы можете [включить отчеты о доставке с помощью Сетки событий](../quickstarts/telephony-sms/handle-sms-events.md) для сбора сведений о доставке SMS.

## <a name="getting-help"></a>Получение справки

Мы советуем разработчикам отправлять вопросы, предлагать функции и сообщать о проблемах. Чтобы помочь вам в этом, у нас есть [специальная страница поддержки и параметров справки](../support.md) со списком вариантов поддержки.

Для устранения определенных типов проблем могут потребоваться указанные ниже сведения.

* **ИД MS-CV**. Этот идентификатор используется для устранения неполадок при вызовах и сообщениях.
* **ИД вызова**. Этот идентификатор используется для идентификации вызовов Служб коммуникации.
* **ИД SMS-сообщений**. Этот идентификатор используется для идентификации SMS-сообщений.
* **Журналы вызовов**. В этих журналах содержится подробная информация, которую можно использовать для устранения проблем с вызовами и сетью.


## <a name="access-your-ms-cv-id"></a>Доступ к ИД MS-CV

Доступ к ИД MS-CV можно получить, настроив диагностику в экземпляре объекта `clientOptions` при инициализации пакетов SDK. Диагностику можно настроить для любого из пакетов Azure SDK, в том числе для чата, удостоверений и вызовов VoIP.

### <a name="client-options-example"></a>Пример параметров клиента

В указанных ниже фрагментах кода показана конфигурация диагностики. При использовании пакетов SDK с включенной диагностикой сведения о диагностике передаются настроенному прослушивателю событий:

# <a name="c"></a>[C#](#tab/csharp)
```
// 1. Import Azure.Core.Diagnostics
using Azure.Core.Diagnostics;

// 2. Initialize an event source listener instance
using var listener = AzureEventSourceListener.CreateConsoleLogger();
Uri endpoint = new Uri("https://<RESOURCE-NAME>.communication.azure.net");
var (token, communicationUser) = await GetCommunicationUserAndToken();
CommunicationUserCredential communicationUserCredential = new CommunicationUserCredential(token);

// 3. Setup diagnostic settings
var clientOptions = new ChatClientOptions()
{
    Diagnostics =
    {
        LoggedHeaderNames = { "*" },
        LoggedQueryParameters = { "*" },
        IsLoggingContentEnabled = true,
    }
};

// 4. Initialize the ChatClient instance with the clientOptions
ChatClient chatClient = new ChatClient(endpoint, communicationUserCredential, clientOptions);
ChatThreadClient chatThreadClient = await chatClient.CreateChatThreadAsync("Thread Topic", new[] { new ChatThreadMember(communicationUser) });
```

# <a name="python"></a>[Python](#tab/python)
```
from azure.communication.chat import ChatClient, CommunicationUserCredential
endpoint = "https://communication-services-sdk-live-tests-for-python.communication.azure.com"
chat_client = ChatClient(
    endpoint,
    CommunicationUserCredential(token),
    http_logging_policy=your_logging_policy)
```
---

## <a name="access-your-call-id"></a>Доступ к ИД вызова

При создании связанного с проблемами при вызовах запроса в службу поддержки с помощью портала Azure может понадобиться указать идентификатор вызова, на который вы ссылаетесь. Доступ к нему можно получить с помощью пакета SDK для вызовов:

# <a name="javascript"></a>[JavaScript](#tab/javascript)
```javascript
// `call` is an instance of a call created by `callAgent.call` or `callAgent.join` methods
console.log(call.id)
```

# <a name="ios"></a>[iOS](#tab/ios)
```objc
// The `call id` property can be retrieved by calling the `call.getCallId()` method on a call object after a call ends
// todo: the code snippet suggests it's a property while the comment suggests it's a method call
print(call.callId)
```

# <a name="android"></a>[Android](#tab/android)
```java
// The `call id` property can be retrieved by calling the `call.getCallId()` method on a call object after a call ends
// `call` is an instance of a call created by `callAgent.call(…)` or `callAgent.join(…)` methods
Log.d(call.getCallId())
```
---

## <a name="access-your-sms-message-id"></a>Доступ к ИД SMS-сообщений

При проблемах с SMS идентификатор сообщения можно получить из объекта ответа.

# <a name="net"></a>[.NET](#tab/dotnet)
```
// Instantiate the SMS client
const smsClient = new SmsClient(connectionString);
async function main() {
  const result = await smsClient.send({
    from: "+18445792722",
    to: ["+1972xxxxxxx"],
    message: "Hello World 👋🏻 via Sms"
  }, {
    enableDeliveryReport: true // Optional parameter
  });
console.log(result); // your message ID will be in the result
}
```
---

## <a name="enable-and-access-call-logs"></a>Включение журналов вызовов и доступ к ним

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Следующий код можно использовать для настройки `AzureLogger` для вывода журналов на консоль с помощью пакета SDK для JavaScript:

```javascript
import { AzureLogger } from '@azure/logger';

AzureLogger.verbose = (...args) => { console.info(...args); }
AzureLogger.info = (...args) => { console.info(...args); }
AzureLogger.warning = (...args) => { console.info(...args); }
AzureLogger.error = (...args) => { console.info(...args); }

callClient = new CallClient({logger: AzureLogger});
```

# <a name="ios"></a>[iOS](#tab/ios)

При разработке для iOS ваши журналы хранятся в файлах `.blog`. Обратите внимание, что вы не можете просматривать журналы напрямую, потому что они зашифрованы.

Доступ к ним можно получить, открыв Xcode. Последовательно выберите Windows > Devices and Simulators (Устройства и симуляторы) > Devices (Устройства). Выберите свое устройство. В разделе "Установленные приложения" выберите свое приложение и нажмите Download container (Загрузить контейнер).

Таким образом, вы получите файл `xcappdata`. Щелкните этот файл правой кнопкой мыши и выберите Show package contents (Показать содержимое пакета). Затем вы увидите файлы `.blog`, которые затем можно будет прикрепить к запросу в службу поддержки Azure.

# <a name="android"></a>[Android](#tab/android)

При разработке для Android ваши журналы хранятся в файлах `.blog`. Обратите внимание, что вы не можете просматривать журналы напрямую, потому что они зашифрованы.

В Android Studio перейдите в проводник файлов устройства, последовательно выбрав View > Tool Windows > Device File Explorer (Вид >Окна инструментов > Проводник файлов устройства) как в симуляторе, так и на устройстве. Файл `.blog` будет находиться в каталоге приложения, которое должно выглядеть примерно так: `/data/data/[app_name_space:com.contoso.com.acsquickstartapp]/files/acs_sdk.blog`. Вы можете прикрепить этот файл к запросу в службу поддержки.


---

## <a name="calling-sdk-error-codes"></a>Коды ошибок пакета SDK для вызовов

В пакете SDK для вызовов Служб коммуникации Azure используются следующие коды ошибок, которые помогут вам в устранении неполадок с вызовами. Эти коды ошибок предоставляются с помощью свойства `call.callEndReason` после завершения вызова.

| Код ошибки | Описание | Предстоящее действие |
| -------- | ---------------| ---------------|
| 403 | Запрещено / сбой проверки подлинности. | Убедитесь, что маркер Служб коммуникации действителен и не просрочен. |
| 404 | Вызов не найден. | Убедитесь, что номер, по которому вы звоните, или вызов, к которому вы присоединяетесь, существует. |
| 408 | Истекло время ожидания для контроллера вызовов. | В контроллере вызовов истекло время ожидания сообщений протокола от конечных точек пользователей. Убедитесь, что клиенты подключены и доступны. |
| 410 | Ошибка стека локальных носителей или инфраструктуры носителей. | Убедитесь, что вы используете последнюю версию пакета SDK в поддерживаемой среде. |
| 430 | Не удалось доставить сообщение в клиентское приложение. | Убедитесь, что клиентское приложение работает и доступно. |
| 480 | Удаленная конечная точка клиента не зарегистрирована. | Убедитесь, что удаленная конечная точка доступна. |
| 481 | Не удалось обработать входящий вызов. | Создайте запрос на поддержку на портале Azure. |
| 487 | Вызов отменен, локально отклонен, завершен из-за несоответствия конечной точки или не удалось создать предложение в отношении носителя. | Ожидаемое поведение. |
| 490, 491, 496, 487, 498 | Проблемы с сетью в локальной конечной точке. | Проверьте сеть. |
| 500, 503, 504 | Ошибка инфраструктуры Служб коммуникации. | Создайте запрос на поддержку на портале Azure. |
| 603 | Вызов глобально отклонен удаленным участником Служб коммуникации. | Ожидаемое поведение. |

## <a name="related-information"></a>Дополнительные сведения
- [Журналы и диагностика](logging-and-diagnostics.md)
- [Метрики](metrics.md)
