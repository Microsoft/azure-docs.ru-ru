---
title: включить файл
description: включить файл
services: azure-communication-services
author: pvicencio
manager: ankita
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/12/2021
ms.topic: include
ms.custom: include file
ms.author: pvicencio
ms.openlocfilehash: 2739079b67d80f3e4a9f367aaa58f6dcbbb650ca
ms.sourcegitcommit: 18a91f7fe1432ee09efafd5bd29a181e038cee05
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103622221"
---
Начало работы со Службами коммуникации Azure с помощью клиентской библиотеки SMS Служб коммуникации Azure для Java для отправки SMS-сообщений.

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

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
    <version>1.0.0-beta.4</version>
</dependency>
```

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

Добавьте зависимость `azure-core-http-netty` в файл **pom.xml**.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-core-http-netty</artifactId>
    <version>1.8.0</version>
</dependency>
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-core</artifactId>
    <version>1.13.0</version> <!-- {x-version-update;com.azure:azure-core;dependency} -->
</dependency>
```

Откройте **/src/main/java/com/communication/quickstart/App.java** в текстовом редакторе, добавьте директивы импорта и удалите оператор `System.out.println("Hello world!");`:

```java
package com.communication.quickstart;

import com.azure.communication.sms.models.*;
import com.azure.core.credential.AzureKeyCredential;
import com.azure.communication.sms.*;
import com.azure.core.http.HttpClient;
import com.azure.core.http.netty.NettyAsyncHttpClientBuilder;
import com.azure.core.util.Context;
import java.util.Arrays;

public class App
{
    public static void main( String[] args )
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
| SmsSendResult                | Этот класс содержит результат, полученный от службы SMS.                                          |
| SmsSendOptions               | Этот класс предоставляет варианты для добавления пользовательских тегов и настройки отчетов о доставке. Если deliveryReportEnabled имеет значение True, при успешной доставке будет создано событие.|                           |

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте экземпляр `SmsClient` с помощью строки подключения. (В качестве учетных данных используйте `Key` с портала Azure.) См. сведения о том, как [управлять строкой подключения ресурса](../../create-communication-resource.md#store-your-connection-string).

Добавьте следующий код в метод `main`:

```java
// You can find your endpoint and access key from your resource in the Azure Portal
String endpoint = "https://<resource-name>.communication.azure.com/";
AzureKeyCredential azureKeyCredential = new AzureKeyCredential("<access-key-credential>");

// Create an HttpClient builder of your choice and customize it
HttpClient httpClient = new NettyAsyncHttpClientBuilder().build();

SmsClient smsClient = new SmsClientBuilder()
                .endpoint(endpoint)
                .credential(azureKeyCredential)
                .httpClient(httpClient)
                .buildClient();
```

Вы можете инициализировать клиент с помощью любого пользовательского HTTP-клиента, реализующего интерфейс `com.azure.core.http.HttpClient`. В приведенном выше коде показано, как использовать [HTTP-клиент Netty Azure Core](/java/api/overview/azure/core-http-netty-readme), предоставляемый `azure-core`.

Чтобы не указывать конечную точку и ключ доступа, вы можете указать всю строку подключения с помощью функции connectionString(). 
```java
// You can find your connection string from your resource in the Azure Portal
String connectionString = "https://<resource-name>.communication.azure.com/;<access-key>";

// Create an HttpClient builder of your choice and customize it
HttpClient httpClient = new NettyAsyncHttpClientBuilder().build();

SmsClient smsClient = new SmsClientBuilder()
            .connectionString(connectionString)
            .httpClient(httpClient)
            .buildClient();
```

## <a name="send-a-11-sms-message"></a>Отправка личного текстового сообщения

Чтобы отправить текстовое сообщение одному получателю, вызовите метод `send` из SmsClient с номером телефона этого получателя. Кроме того, вы можете передать необязательные параметры для указания того, должен ли быть включен отчет о доставке, а также задать пользовательские теги.

```java
SmsSendResult sendResult = smsClient.send(
                "<from-phone-number>",
                "<to-phone-number>",
                "Weekly Promotion");

System.out.println("Message Id: " + sendResult.getMessageId());
System.out.println("Recipient Number: " + sendResult.getTo());
System.out.println("Send Result Successful:" + sendResult.isSuccessful());
```
## <a name="send-a-1n-sms-message-with-options"></a>Отправка группового текстового сообщения с параметрами
Чтобы отправить текстовое сообщение списку получателей, вызовите метод `send` со списком номеров телефонов нужных получателей. Кроме того, вы можете передать необязательные параметры для указания того, должен ли быть включен отчет о доставке, а также задать пользовательские теги.
```java
SmsSendOptions options = new SmsSendOptions();
options.setDeliveryReportEnabled(true);
options.setTag("Marketing");

Iterable<SmsSendResult> sendResults = smsClient.send(
    "<from-phone-number>",
    Arrays.asList("<to-phone-number1>", "<to-phone-number2>"),
    "Weekly Promotion",
    options /* Optional */,
    Context.NONE);

for (SmsSendResult result : sendResults) {
    System.out.println("Message Id: " + result.getMessageId());
    System.out.println("Recipient Number: " + result.getTo());
    System.out.println("Send Result Successful:" + result.isSuccessful());
}
```

`<from-phone-number>` необходимо заменить номером телефона с поддержкой SMS, связанным с ресурсом Служб коммуникации, а `<to-phone-number>` — номером телефона или списком номеров, на которые вы хотите отправить сообщение.

## <a name="optional-parameters"></a>Необязательные параметры

Параметр `deliveryReportEnabled` является необязательным. Его можно использовать для настройки отчетов о доставке. Это полезно, если вы хотите, чтобы при доставке SMS-сообщений создавались события. Сведения о настройке отчетов о доставке SMS-сообщений см. в кратком руководстве [Обработка событий SMS-сообщений](../handle-sms-events.md).

`tag` — это необязательный параметр, который можно использовать для применения тегов к отчету о доставке.

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