---
title: включить файл
description: включить файл
services: azure-communication-services
author: chrwhit
manager: nimag
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 08/20/2020
ms.topic: include
ms.custom: include file
ms.author: chrwhit
ms.openlocfilehash: 1c4f3c47e3ac6e1e701b673574bb664237c1a9af
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102244886"
---
Начало работы со Службами коммуникации Azure с помощью клиентской библиотеки SMS Служб коммуникации Azure для Java для отправки SMS-сообщений.

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

<!--**TODO: update all these reference links as the resources go live**

[API reference documentation](../../../references/overview.md) | [Library source code](https://github.com/Azure/azure-sdk-for-net-pr/tree/feature/communication/sdk/communication/Azure.Communication.Sms#todo-update-to-public) | [Artifact (Maven)](#todo-nuget) | [Samples](#todo-samples)-->

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [комплект SDK для Java (JDK)](/java/azure/jdk/) версии 8 или более поздней версии.
- [Apache Maven](https://maven.apache.org/download.cgi).
- Активный ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)
- Номер телефона с поддержкой SMS-сообщений. [Получите номер телефона.](../get-phone-number.md)

### <a name="prerequisite-check"></a>Проверка предварительных условий

- В терминале или командном окне воспользуйтесь `mvn -v`, чтобы проверить, установлен ли пакет maven.
- Чтобы просмотреть номера телефонов, связанные с ресурсом Служб коммуникации, войдите на [портал Azure](https://portal.azure.com/), перейдите к ресурсу Служб коммуникации и откройте вкладку с **номерами телефонов** в области навигации слева.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-java-application"></a>Создание нового приложения Java

Откройте терминал или командное окно и перейдите в каталог, в котором необходимо создать приложение Java. Выполните приведенную ниже команду, чтобы создать проект Java из шаблона maven-archetype-quickstart.

```console
mvn archetype:generate -DgroupId=com.communication.quickstart -DartifactId=communication-quickstart -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

Цель generate создаст каталог с тем же именем, что и у artifactId. В этом каталоге каталог **src/main/java** содержит исходный код проекта, каталог **src/test/java** содержит источник теста, а файл **pom.xml** является объектной моделью проекта (POM).

### <a name="install-the-package"></a>Установка пакета

Откройте файл **pom.xml** в текстовом редакторе. Добавьте приведенный ниже элемент зависимости в группу зависимостей.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-sms</artifactId>
    <version>1.0.0-beta.3</version>
</dependency>
```

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

Добавьте зависимость `azure-core-http-netty` в файл **pom.xml**.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-core-http-netty</artifactId>
    <version>1.3.0</version>
</dependency>
```

Откройте **/src/main/java/com/communication/quickstart/App.java** в текстовом редакторе, добавьте директивы импорта и удалите оператор `System.out.println("Hello world!");`:

```java
package com.communication.quickstart;

import java.io.IOException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.util.ArrayList;
import java.util.List;

import com.azure.communication.common.PhoneNumber;
import com.azure.communication.sms.SmsClient;
import com.azure.communication.sms.SmsClientBuilder;
import com.azure.communication.sms.models.SendSmsOptions;
import com.azure.communication.sms.models.SendSmsResponse;
import com.azure.core.http.HttpClient;
import com.azure.core.http.netty.NettyAsyncHttpClientBuilder;

public class App
{
    public static void main( String[] args ) throws IOException, NoSuchAlgorithmException, InvalidKeyException
    {
        // Quickstart code goes here
    }
}

```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции клиентской библиотеки Служб коммуникации SMS для Java.

| Имя                                                             | Описание                                                                                     |
| ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| SmsClientBuilder              | Этот класс создает SmsClient. Вы предоставляете ему конечную точку, учетные данные и HTTP-клиент. |
| SmsClient                    | Этот класс требуется для реализации всех функций обмена текстовыми сообщениями. Он используется для отправки SMS-сообщений.                |
| SendSmsResponse               | Этот класс содержит ответ от службы SMS.                                          |
| PhoneNumber                   | Этот класс содержит сведения о номере телефона.

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте экземпляр `SmsClient` с помощью строки подключения. Приведенный ниже код извлекает строки конечной точки и учетных данных ресурса из переменных среды с именами `COMMUNICATION_SERVICES_ENDPOINT_STRING` и `COMMUNICATION_SERVICES_CREDENTIAL_STRING` (учетные данные — это `Key` с портала Azure). См. сведения о том, как [управлять строкой подключения ресурса](../../create-communication-resource.md#store-your-connection-string).

Добавьте следующий код в метод `main`:

```java
// Your can find your endpoint and access key from your resource in the Azure Portal
String endpoint = "https://<RESOURCE_NAME>.communication.azure.com";
String accessKey = "SECRET";

// Create an HttpClient builder of your choice and customize it
HttpClient httpClient = new NettyAsyncHttpClientBuilder().build();

// Configure and build a new SmsClient
SmsClient client = new SmsClientBuilder()
    .endpoint(endpoint)
    .accessKey(accessKey)
    .httpClient(httpClient)
    .buildClient();
```

Вы можете инициализировать клиент с помощью любого пользовательского HTTP-клиента, реализующего интерфейс `com.azure.core.http.HttpClient`. В приведенном выше коде показано, как использовать [HTTP-клиент Netty Azure Core](/java/api/overview/azure/core-http-netty-readme), предоставляемый `azure-core`.

Чтобы не указывать конечную точку и ключ доступа, вы можете указать всю строку подключения с помощью функции connectionString(). 
```java
// Your can find your connection string from your resource in the Azure Portal
String connectionString = "<connection_string>";
SmsClient client = new SmsClientBuilder()
    .connectionString(connectionString)
    .httpClient(httpClient)
    .buildClient();
```

## <a name="send-an-sms-message"></a>Отправка SMS-сообщения

Отправьте SMS-сообщение, вызвав метод sendMessage. Добавьте этот код в конец метода `main`:

```java
List<PhoneNumber> to = new ArrayList<PhoneNumber>();
to.add(new PhoneNumber("<to-phone-number>"));

// SendSmsOptions is an optional field. It can be used
// to enable a delivery report to the Azure Event Grid
SendSmsOptions options = new SendSmsOptions();
options.setEnableDeliveryReport(true);

// Send the message and check the response for a message id
SendSmsResponse response = client.sendMessage(
    new PhoneNumber("<leased-phone-number>"),
    to,
    "<message-text>",
    options);

System.out.println("MessageId: " + response.getMessageId());
```

`<leased-phone-number>` необходимо заменить номером телефона с поддержкой SMS, связанным с ресурсом Служб коммуникации, а `<to-phone-number>` — номером телефона, на который вы хотите отправить сообщение.

Параметр `enableDeliveryReport` является необязательным. Его можно использовать для настройки отчетов о доставке. Это полезно, если вы хотите, чтобы при доставке SMS-сообщений создавались события. Сведения о настройке отчетов о доставке SMS-сообщений см. в кратком руководстве [Обработка событий SMS-сообщений](../handle-sms-events.md).

<!--todo: the signature of the `sendMessage` method changes when configuring delivery reporting. Need to confirm that this is how our client library is to be used.-->

## <a name="run-the-code"></a>Выполнение кода

Перейдите в каталог, содержащий файл **pom.xml**, и скомпилируйте проект с помощью команды `mvn`.

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