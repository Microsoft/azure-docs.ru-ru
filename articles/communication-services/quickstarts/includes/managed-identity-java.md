---
ms.openlocfilehash: f2e4bf603fa4cfb93c7ca51f64029ccaedcff727
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103020947"
---
## <a name="add-managed-identity-to-your-communication-services-solution-java"></a>Добавление управляемого удостоверения в решение "службы связи" (Java)

### <a name="install-the-client-library-packages"></a>Установка пакетов клиентской библиотеки
В файл pom.xml добавьте следующие элементы зависимости в группу зависимостей.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-identity</artifactId>
    <version>1.0.0-beta.6</version>
</dependency>
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-sms</artifactId>
    <version>1.0.0-beta.4</version>
</dependency>
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-identity</artifactId>
    <version>1.2.3</version>
</dependency>
```

### <a name="use-the-client-library-packages"></a>Использование пакетов клиентских библиотек

Добавьте следующие `import` директивы в код для использования Azure Identity и клиентских библиотек Azure Communication.

```java
import com.azure.communication.common.*;
import com.azure.communication.identity.*;
import com.azure.communication.identity.models.*;
import com.azure.communication.sms.*;
import com.azure.core.credential.*;
import com.azure.core.http.*;
import com.azure.core.http.netty.*;
import com.azure.identity.*;

import java.io.IOException;
import java.util.*;
```

В приведенных ниже примерах используется [дефаултазурекредентиал](/java/api/azure.identity.defaultazurecredential). Эти учетные данные подходят для сред рабочей среды и разработки.

`AZURE_CLIENT_SECRET`, `AZURE_CLIENT_ID` а `AZURE_TENANT_ID` переменные среды необходимы для создания `DefaultAzureCredential` объекта. Сведения о создании зарегистрированного приложения в среде разработки и настройке переменных среды см. в разделе [авторизация доступа с помощью управляемого удостоверения](../managed-identity-from-cli.md).

### <a name="create-an-identity-and-issue-a-token-with-managed-identity"></a>Создание удостоверения и выдача маркера с управляемым удостоверением

В следующем примере кода показано, как создать объект клиента службы с управляемым удостоверением.
Затем используйте клиент, чтобы выдать маркер для нового пользователя:

```java
     public AccessToken createIdentityAndGetTokenAsync() {
          // You can find your endpoint and access key from your resource in the Azure portal
          String endpoint = "https://<RESOURCE_NAME>.communication.azure.com";

          HttpClient httpClient = new NettyAsyncHttpClientBuilder().build();
          TokenCredential credential = new DefaultAzureCredentialBuilder().build();

          CommunicationIdentityClient communicationIdentityClient = new CommunicationIdentityClientBuilder()
               .endpoint(endpoint)
               .credential(credential)
               .httpClient(httpClient)
               .buildClient();

          CommunicationUserIdentifier user = communicationIdentityClient.createUser();
          AccessToken userToken = communicationIdentityClient.getToken(user, new ArrayList<>(Arrays.asList(CommunicationTokenScope.CHAT)));
          return userToken;
     }
```

### <a name="send-an-sms-with-managed-identity"></a>Отправка SMS с управляемым удостоверением

В следующем примере кода показано, как создать объект клиента службы с управляемым удостоверением, а затем использовать клиент для отправки SMS-сообщения:

```java
     public SendSmsResponse sendSms() {
          // You can find your endpoint and access key from your resource in the Azure portal
          String endpoint = "https://<RESOURCE_NAME>.communication.azure.com";

          HttpClient httpClient = new NettyAsyncHttpClientBuilder().build();
          TokenCredential credential = new DefaultAzureCredentialBuilder().build();

          SmsClient smsClient = new SmsClientBuilder()
               .endpoint(endpoint)
               .credential(credential)
               .httpClient(httpClient)
               .buildClient();

          // Send the message and check the response for a message id
          SmsSendResult response = smsClient.send(
               "<from-phone-number>",
               "<to-phone-number>",
               "your message"
          );

          return response;
    }
```

