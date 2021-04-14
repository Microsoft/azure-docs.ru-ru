---
title: включить файл
description: включить файл
services: azure-communication-services
author: tomaschladek
manager: nmurav
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/10/2021
ms.topic: include
ms.custom: include file
ms.author: tchladek
ms.openlocfilehash: a0f3e3547c38df63bdab77cf378525072d1e9ad4
ms.sourcegitcommit: 9f4510cb67e566d8dad9a7908fd8b58ade9da3b7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106125816"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [комплект SDK для Java (JDK)](https://docs.microsoft.com/azure/developer/java/fundamentals/java-jdk-install) версии 8 или более поздней версии.
- [Apache Maven](https://maven.apache.org/download.cgi).
- Развернутый ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](../create-communication-resource.md)

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-java-application"></a>Создание нового приложения Java

Откройте терминал или командное окно. Перейдите в каталог, в котором нужно создать приложение Java. Выполните приведенную ниже команду, чтобы создать проект Java из шаблона maven-archetype-quickstart.

```console
mvn archetype:generate -DgroupId=com.communication.quickstart -DartifactId=communication-quickstart -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

Вы заметите, что задача generate создала каталог с тем же именем, что и у `artifactId`. В этом каталоге каталог src/main/java содержит исходный код проекта, `pom.xml` содержит источник теста, а файл `src/test/java directory` является объектной моделью проекта (POM).

### <a name="install-the-package"></a>Установка пакета

Откройте файл **pom.xml** в текстовом редакторе. Добавьте приведенный ниже элемент зависимости в группу зависимостей.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-identity</artifactId>
    <version>1.0.0</version>
</dependency>
```

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

Из каталога проекта:

1. Перейдите в каталог */src/main/java/com/communication/quickstart*.
1. Откройте файл *App.java* в редакторе.
1. Замените оператор `System.out.println("Hello world!");`.
1. Добавьте директивы `import`.

Используйте следующий код:

```java
package com.communication.quickstart;

import com.azure.communication.common.*;
import com.azure.communication.identity.*;
import com.azure.communication.identity.models.*;
import com.azure.core.credential.*;

import java.io.IOException;
import java.time.*;
import java.util.*;

public class App
{
    public static void main( String[] args ) throws IOException
    {
        System.out.println("Azure Communication Services - Access Tokens Quickstart");
        // Quickstart code goes here
    }
}
```

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте экземпляр `CommunicationIdentityClient` с помощью ключа доступа и конечной точки ресурса. См. сведения о том, как [управлять строкой подключения ресурса](../create-communication-resource.md#store-your-connection-string). Кроме того, вы можете инициализировать клиент с помощью любого пользовательского HTTP-клиента, реализующего интерфейс `com.azure.core.http.HttpClient`.

Добавьте следующий код в метод `main`:

```java
// Your can find your endpoint and access key from your resource in the Azure portal
String endpoint = "https://<RESOURCE_NAME>.communication.azure.com";
String accessKey = "SECRET";

CommunicationIdentityClient communicationIdentityClient = new CommunicationIdentityClientBuilder()
        .endpoint(endpoint)
        .credential(new AzureKeyCredential(accessKey))
        .buildClient();
```

Чтобы не указывать конечную точку и ключ доступа, вы можете указать всю строку подключения с помощью функции `connectionString()`.
```java
// Your can find your connection string from your resource in the Azure portal
String connectionString = "<connection_string>";

CommunicationIdentityClient communicationIdentityClient = new CommunicationIdentityClientBuilder()
    .connectionString(connectionString)
    .buildClient();
```

Если вы настроили управляемое удостоверение, то из статьи [Использование управляемых удостоверений](../managed-identity.md) можете узнать, как использовать его для аутентификации.
```java
String endpoint = "https://<RESOURCE_NAME>.communication.azure.com";
TokenCredential credential = new DefaultAzureCredentialBuilder().build();

CommunicationIdentityClient communicationIdentityClient = new CommunicationIdentityClientBuilder()
        .endpoint(endpoint)
        .credential(credential)
        .buildClient();
```

## <a name="create-an-identity"></a>Создание удостоверения

Службы коммуникации Azure позволяют использовать упрощенный каталог удостоверений. Чтобы создать новую запись в каталоге с уникальным идентификатором (`Id`), используйте метод `createUser`. Сохраните полученное удостоверение с сопоставлением с пользователями вашего приложения. Например, сохраните их в базе данных сервера приложений. Удостоверение потребуется позже для выдачи маркеров доступа.

```java
CommunicationUserIdentifier user = communicationIdentityClient.createUser();
System.out.println("\nCreated an identity with ID: " + user.getId());
```

## <a name="issue-access-tokens"></a>Выпуск маркеров доступа

Чтобы выдать маркер доступа для имеющегося удостоверения Служб коммуникации, используйте метод `getToken`. Параметр `scopes` определяет набор базовых функций, которые будут авторизовать этот маркер доступа. Ознакомьтесь со [списком поддерживаемых действий](../../concepts/authentication.md). Новый экземпляр параметра `user` можно создать на основе строкового представления удостоверения Служб коммуникации Azure.

```java
// Issue an access token with the "voip" scope for a user identity
List<CommunicationTokenScope> scopes = new ArrayList<>(Arrays.asList(CommunicationTokenScope.VOIP));
AccessToken accessToken = communicationIdentityClient.getToken(user, scopes);
OffsetDateTime expiresAt = accessToken.getExpiresAt();
String token = accessToken.getToken();
System.out.println("\nIssued an access token with 'voip' scope that expires at: " + expiresAt + ": " + token);
```

## <a name="create-an-identity-and-issue-token-in-one-call"></a>Создание удостоверения и маркера в одном вызове

Кроме того, с помощью метода createUserAndToken вы можете создать новую запись в каталоге с уникальным значением `Id`, а также маркер доступа.

```java
List<CommunicationTokenScope> scopes = Arrays.asList(CommunicationTokenScope.CHAT);
CommunicationUserIdentifierAndToken result = communicationIdentityClient.createUserAndToken(scopes);
CommunicationUserIdentifier user = result.getUser();
System.out.println("\nCreated a user identity with ID: " + user.getId());
AccessToken accessToken = result.getUserToken();
OffsetDateTime expiresAt = accessToken.getExpiresAt();
String token = accessToken.getToken();
System.out.println("\nIssued an access token with 'chat' scope that expires at: " + expiresAt + ": " + token);
```

Маркеры доступа — это недолговечные учетные данные, которые необходимо выдавать повторно. Если этого не сделать, это может привести к нарушению работы пользователей приложения. Свойство `expiresAt` обозначает время существования маркера доступа.

## <a name="refresh-access-tokens"></a>Обновление токенов доступа

Чтобы обновить маркер доступа, используйте объект `CommunicationUserIdentifier` для повторной выдачи:

```java
// Value existingIdentity represents identity of Azure Communication Services stored during identity creation
CommunicationUserIdentifier identity = new CommunicationUserIdentifier(existingIdentity);
AccessToken response = communicationIdentityClient.getToken(identity, scopes);
```

## <a name="revoke-access-tokens"></a>Отмена маркеров доступа

В некоторых случаях вы можете явно отменить маркеры доступа. Например, когда пользователь приложения меняет пароль, который он использует для проверки подлинности в службе. Метод `revokeTokens` аннулирует все активные маркеры доступа, выданные удостоверению.

```java
communicationIdentityClient.revokeTokens(user);
System.out.println("\nSuccessfully revoked all access tokens for user identity with ID: " + user.getId());
```

## <a name="delete-an-identity"></a>Удаление удостоверения

При удалении удостоверения отзываются все активные маркеры доступа и запрещается выдача последующих маркеров для удостоверения. Вместе с этим удаляется и все хранимое содержимое, связанное с удостоверением.

```java
communicationIdentityClient.deleteUser(user);
System.out.println("\nDeleted the user identity with ID: " + user.getId());
```

## <a name="run-the-code"></a>Выполнение кода

Перейдите в каталог, содержащий файл *pom.xml*, и скомпилируйте проект с помощью следующей команды `mvn`.

```console
mvn compile
```

Затем выполните сборку пакета.

```console
mvn package
```

Выполните следующую команду `mvn` для запуска приложения.

```console
mvn exec:java -Dexec.mainClass="com.communication.quickstart.App" -Dexec.cleanupDaemonThreads=false
```
