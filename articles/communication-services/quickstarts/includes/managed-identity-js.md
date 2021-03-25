---
ms.openlocfilehash: 3ec650f82b95cdfabc923e92325959c63f2846de
ms.sourcegitcommit: bed20f85722deec33050e0d8881e465f94c79ac2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105110770"
---
## <a name="add-managed-identity-to-your-communication-services-solution-js"></a>Добавление управляемого удостоверения в решение служб связи (JS)

### <a name="install-the-sdk-packages"></a>Установка пакетов SDK

```console
npm install @azure/communication-identity
npm install @azure/communication-common
npm install @azure/communication-sms
npm install @azure/identity
```

### <a name="use-the-sdk-packages"></a>Использование пакетов SDK

Добавьте в `import` код следующие директивы для использования удостоверения Azure и пакетов SDK службы хранилища Azure.

```typescript
import { DefaultAzureCredential } from "@azure/identity";
import { CommunicationIdentityClient, CommunicationUserToken } from "@azure/communication-identity";
import { SmsClient, SmsSendRequest } from "@azure/communication-sms";
```

В приведенных ниже примерах используется [дефаултазурекредентиал](/javascript/api/azure.identity.defaultazurecredential). Эти учетные данные подходят для сред рабочей среды и разработки.

Сведения о регистрации приложения в среде разработки и настройке переменных среды см. в разделе [авторизация доступа с помощью управляемого удостоверения](../managed-identity-from-cli.md) .  

### <a name="create-an-identity-and-issue-a-token-with-managed-identity"></a>Создание удостоверения и выдача маркера с управляемым удостоверением

В следующем примере кода показано, как создать объект клиента службы с управляемым удостоверением, а затем использовать клиент для выдаче маркера для нового пользователя:

```JavaScript
export async function createIdentityAndIssueToken(resourceEndpoint: string): Promise<CommunicationUserToken> {
     let credential = new DefaultAzureCredential();
     const client = new CommunicationIdentityClient(resourceEndpoint, credential);
     return await client.createUserAndToken(["chat"]);
}
```

### <a name="send-an-sms-with-managed-identity"></a>Отправка SMS с управляемым удостоверением

В следующем примере кода показано, как создать объект клиента службы с управляемым удостоверением, а затем использовать клиент для отправки SMS-сообщения:

```JavaScript
export async function sendSms(resourceEndpoint: string, fromNumber: any, toNumber: any, message: string) {
     let credential = new DefaultAzureCredential();
     const smsClient = new SmsClient(resourceEndpoint, credential);
     const sendRequest: SmsSendRequest = { 
          from: fromNumber, 
          to: [toNumber], 
          message: message 
     };

     const response = await smsClient.send(
          sendRequest, 
          {} //Optional SendOptions
          );
}
```

