---
ms.openlocfilehash: c5d83cc709c334e15a1c13e64ba17ef24835fed0
ms.sourcegitcommit: b4647f06c0953435af3cb24baaf6d15a5a761a9c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/02/2021
ms.locfileid: "101657678"
---
## <a name="add-managed-identity-to-your-communication-services-solution-java"></a>Добавление управляемого удостоверения в решение "службы связи" (Java)

### <a name="install-the-client-library-packages"></a>Установка пакетов клиентской библиотеки
В файл pom.xml добавьте следующие элементы зависимости в группу зависимостей.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-identity</artifactId>
    <version>1.0.0</version>
</dependency>
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-sms</artifactId>
    <version>1.0.0</version>
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
import com.azure.identity.*;
import com.azure.communication.sms.*;
import com.azure.communication.identity.*;
import com.azure.communication.common.*;
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
          SendSmsResponse response = smsClient.sendMessage(
               new PhoneNumberIdentifier("<leased-phone-number>"),
               to,
               "your message",
               options /* Optional */
          );
          return response;
    }
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о проверке подлинности](../concepts/authentication.md)

Возможно, вы также захотите выполнить такие задачи:

- [Дополнительные сведения об управлении доступом на основе ролей в Azure](../../../../articles/role-based-access-control/index.yml)
- [Дополнительные сведения о библиотеке удостоверений Azure для Java](/java/api/overview/azure/identity-readme)
- [Создание маркеров доступа пользователей](../../quickstarts/access-tokens.md)
- [Отправка SMS](../../quickstarts/telephony-sms/send.md)
- [Дополнительные сведения об SMS](../../concepts/telephony-sms/concepts.md)
