---
title: Отправка сообщений из облака на устройство с помощью Центра Интернета вещей Azure (Java) | Документация Майкрософт
description: Сведения об отправке сообщений из облака на устройство из Центра Интернета вещей Azure с помощью пакетов SDK для Azure IoT для Java. Для получения сообщений, отправляемых из облака на устройство, следует изменить приложение имитации устройства, а для отправки таких сообщений следует изменить внутреннее приложение.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: conceptual
ms.date: 06/28/2017
ms.custom:
- amqp
- mqtt
- devx-track-java
ms.openlocfilehash: 5ae1850add94d83278b0fe1905dfa6e53c71fc8e
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102217896"
---
# <a name="send-cloud-to-device-messages-with-iot-hub-java"></a>Отправка сообщений из облака на устройство с помощью Центра Интернета вещей (Java)

[!INCLUDE [iot-hub-selector-c2d](../../includes/iot-hub-selector-c2d.md)]

Центр Интернета вещей Azure — это полностью управляемая служба, которая обеспечивает надежный и защищенный двунаправленный обмен данными между миллионами устройств и серверной частью решения. В руководстве по [отправке данных телеметрии с устройства в центр Интернета вещей](quickstart-send-telemetry-java.md) показано, как создать центр Интернета вещей, подготовить в нем удостоверение устройства и написать код приложения для имитации устройства, которое отправляет сообщения из устройства в облако.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Этот учебник является логическим продолжением статьи по [отправке данных телеметрии с устройства в центр Интернета вещей](quickstart-send-telemetry-java.md). В ней описывается, как сделать следующее:

* Отправка сообщений, передаваемых из облака на устройство, из серверной части решения на одно устройство через Центр Интернета вещей.

* Получение на устройстве сообщений, передаваемых из облака на устройство.

* Запрос из серверной части решения подтверждения доставки (*отзыва*) для сообщений, отправленных на устройство из Центра Интернета вещей.

Дополнительные сведения о сообщениях, отправляемых из облака на устройство, см. в [руководстве разработчика для Центра Интернета вещей](iot-hub-devguide-messaging.md).

В конце этого руководства запускаются два консольных приложения для Java:

* **simulated-device**, модифицированная версия приложения, созданного при работе с руководством по [отправке данных телеметрии с устройства в центр Интернета вещей](quickstart-send-telemetry-java.md), которая подключается к центру Интернета вещей и получает сообщения из облака на устройство.

* **send-c2d-messages**, которое отправляет сообщение из облака на приложение для имитации устройства с помощью Центра Интернета вещей и затем получает подтверждение его доставки.

> [!NOTE]
> В Центре Интернета вещей реализована поддержка для пакетов SDK для многих платформ устройств и языков (включая C, Java, Python и Javascript). Эти пакеты работают на основе пакетов SDK для устройств Azure IoT. Пошаговые указания по связыванию устройства с кодом из этого руководства, а также по подключению к Центру Интернета вещей Azure см. в [центре разработчиков для Интернета вещей Azure](https://azure.microsoft.com/develop/iot).

## <a name="prerequisites"></a>Предварительные требования

* Полная рабочая версия краткого руководства [Отправка данных телеметрии с устройства в центр Интернета вещей](quickstart-send-telemetry-java.md) или руководства [Настройка маршрутизации сообщений с Центром Интернета вещей](tutorial-routing.md).

* [Пакет SDK для Java SE 8](/java/azure/jdk/). Щелкните ссылку **Java 8** в разделе **Долгосрочная поддержка**, чтобы скачать все необходимое для работы с JDK 8.

* [Maven 3](https://maven.apache.org/download.cgi)

* Активная учетная запись Azure. Если ее нет, можно создать [бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/) всего за несколько минут.

* Убедитесь, что в брандмауэре открыт порт 8883. Пример устройства в этой статье использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="receive-messages-in-the-simulated-device-app"></a>Получение сообщений в приложении для имитации устройства

В этом разделе вы измените приложение имитации устройства, созданное в рамках краткого руководства [Отправка данных телеметрии с устройства в центр Интернета вещей](quickstart-send-telemetry-java.md), чтобы с помощью центра Интернета вещей получать сообщения из облака на устройство.

1. Откройте в текстовом редакторе файл simulated-device\src\main\java\com\mycompany\app\App.java.

2. Добавьте следующий класс **MessageCallback** как вложенный класс внутри класса **App**. Метод **execute** вызывается, когда устройство получает сообщение из Центра Интернета вещей. В этом примере устройство всегда уведомляет Центр Интернета вещей о завершении отправки сообщения:

    ```java
    private static class AppMessageCallback implements MessageCallback {
      public IotHubMessageResult execute(Message msg, Object context) {
        System.out.println("Received message from hub: "
          + new String(msg.getBytes(), Message.DEFAULT_IOTHUB_MESSAGE_CHARSET));

        return IotHubMessageResult.COMPLETE;
      }
    }
    ```

3. Измените метод **main**, чтобы создать экземпляр **AppMessageCallback** и вызвать метод **setMessageCallback** перед открытием клиента.

    ```java
    client = new DeviceClient(connString, protocol);

    MessageCallback callback = new AppMessageCallback();
    client.setMessageCallback(callback, null);
    client.open();
    ```

4. Чтобы создать приложение **simulated-device** с помощью Maven, выполните в командной строке в папке simulated-device следующую команду:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

`execute`Метод в `AppMessageCallback` классе возвращает `IotHubMessageResult.COMPLETE` . Это уведомление центра Интернета вещей о том, что сообщение успешно обработано, и что сообщение можно безопасно удалить из очереди устройства. Устройство должно вернуть это значение в случае успешного завершения обработки независимо от используемого протокола.

С AMQP и HTTPS, но не с MQTT, устройство также может:

* Отказаться от сообщения, в результате чего центр Интернета вещей будет хранить сообщение в очереди устройства для будущего использования.
* Отклонить сообщение, которое окончательно удаляет сообщение из очереди устройства.

Если что-то происходит, что не позволяет устройству завершить работу, отменять или отклонять сообщение, центр Интернета вещей через фиксированный период времени ожидания помещает сообщение в очередь для доставки. По этой причине логика обработки сообщений в приложении устройства должна быть *идемпотентными*, поэтому при многократном получении одного и того же сообщения выдается один и тот же результат.

Дополнительные сведения о том, как центр Интернета вещей обрабатывает сообщения, пересылаемые из облака на устройство, включая сведения о жизненном цикле сообщений, отправляемых из облака на устройство, см. в статье [Отправка сообщений из облака на устройство из центра Интернета вещей](iot-hub-devguide-messages-c2d.md).

> [!NOTE]
> При использовании HTTPS вместо MQTT или AMQP в качестве транспорта экземпляр **DeviceClient** проверяет наличие сообщений в центре Интернета вещей редко (по крайней мере каждые 25 минут). Дополнительные сведения о различиях между MQTT, AMQP и поддержкой HTTPS см. в [статье Руководство по обмену данными между облаком и устройством](iot-hub-devguide-c2d-guidance.md) и [Выбор протокола связи](iot-hub-devguide-protocols.md).

## <a name="get-the-iot-hub-connection-string"></a>Получение строки подключения центра Интернета вещей

В этой статье вы создадите серверную службу для отправки сообщений из облака на устройство через центр Интернета вещей, созданный в кратком руководстве [Отправка данных телеметрии с устройства в центр Интернета вещей](quickstart-send-telemetry-java.md). Для отправки сообщений из облака на устройство службе требуется разрешение **service connect**. По умолчанию каждый Центр Интернета вещей создается с помощью политики общего доступа, называемой **службой**, которая предоставляет это разрешение.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="send-a-cloud-to-device-message"></a>Отправка сообщения из облака на устройство

В этом разделе вам предстоит создать консольное приложение Java, которое отправляет сообщения, передаваемые из облака на устройство, в приложение имитации устройства. Вам необходимо ввести идентификатор устройства, добавленного в рамках краткого руководства [Отправка данных телеметрии с устройства в центр Интернета вещей](quickstart-send-telemetry-java.md). Вам также потребуется строка подключения Центра Интернета вещей, скопированной ранее в разделе [Получение строки подключения центра Интернета вещей](#get-the-iot-hub-connection-string).

1. Создайте проект Maven **send-c2d-messages**, выполнив следующую команду в командной строке. Обратите внимание, что это одна длинная команда.

    ```cmd/sh
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=send-c2d-messages -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. В командной строке перейдите к новой папке send-c2d-messages.

3. Откройте в текстовом редакторе файл pom.xml из папки send-c2d-messages и добавьте зависимость, приведенную ниже, в узел **dependencies** . Добавление зависимости позволяет использовать в приложении пакет **iothub-java-service-client** для обмена данными со службой Центра Интернета вещей.

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-service-client</artifactId>
      <version>1.7.23</version>
    </dependency>
    ```

    > [!NOTE]
    > Наличие последней версии пакета **iot-service-client** можно проверить с помощью [поиска Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-service-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

4. Сохраните и закройте файл pom.xml.

5. Откройте в текстовом редакторе файл send-c2d-messages\src\main\java\com\mycompany\app\App.java.

6. Добавьте в файл следующие инструкции **import** .

    ```java
    import com.microsoft.azure.sdk.iot.service.*;
    import java.io.IOException;
    import java.net.URISyntaxException;
    ```

7. Добавьте следующие переменные уровня класса в класс **App**, заменив **{yourhubconnectionstring}** и **{yourdeviceid}** значениями, записанными ранее:

    ```java
    private static final String connectionString = "{yourhubconnectionstring}";
    private static final String deviceId = "{yourdeviceid}";
    private static final IotHubServiceClientProtocol protocol =    
        IotHubServiceClientProtocol.AMQPS;
    ```

8. Замените метод **main** следующим кодом, который подключается к Центру Интернета вещей, отправляет сообщение на устройство, а затем ждет подтверждения о том, что устройство получило и обработало это сообщение.

    ```java
    public static void main(String[] args) throws IOException,
        URISyntaxException, Exception {
      ServiceClient serviceClient = ServiceClient.createFromConnectionString(
        connectionString, protocol);

      if (serviceClient != null) {
        serviceClient.open();
        FeedbackReceiver feedbackReceiver = serviceClient
          .getFeedbackReceiver();
        if (feedbackReceiver != null) feedbackReceiver.open();

        Message messageToSend = new Message("Cloud to device message.");
        messageToSend.setDeliveryAcknowledgement(DeliveryAcknowledgement.Full);

        serviceClient.send(deviceId, messageToSend);
        System.out.println("Message sent to device");

        FeedbackBatch feedbackBatch = feedbackReceiver.receive(10000);
        if (feedbackBatch != null) {
          System.out.println("Message feedback received, feedback time: "
            + feedbackBatch.getEnqueuedTimeUtc().toString());
        }

        if (feedbackReceiver != null) feedbackReceiver.close();
        serviceClient.close();
      }
    }
    ```

    > [!NOTE]
    > Для упрощения в этом руководстве не реализуются политики повтора. В рабочем коде следует реализовать политики повторных попыток (например, с экспоненциальной задержкой), как указано в статье [Обработка временных сбоев](/azure/architecture/best-practices/transient-faults).

9. Чтобы создать приложение **simulated-device** с помощью Maven, выполните в командной строке в папке simulated-device следующую команду:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="run-the-applications"></a>Запуск приложений

Теперь все готово к запуску приложений.

1. В командной строке в папке simulated-device выполните приведенную ниже команду, чтобы начать отправку данных телеметрии в Центр Интернета вещей и прослушивание сообщений из облака на устройство, отправляемых из центра:

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![Запуск приложения виртуального устройства](./media/iot-hub-java-java-c2d/receivec2d.png)

2. В командной строке в папке send-c2d-messages выполните следующую команду, чтобы отправить сообщение из облака на устройство и ожидать подтверждения доставки:

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![Выполнение команды для отправки сообщения из облака на устройство](media/iot-hub-java-java-c2d/sendc2d.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике вы научились отправлять и получать сообщения с облака на устройство.

Примеры комплексных решений, в которых используется Центр Интернета вещей, см. в [документации по акселераторам решений Интернета вещей Azure](https://azure.microsoft.com/documentation/suites/iot-suite/).

Дополнительные сведения о разработке решений с помощью Центра Интернета вещей см. в [руководстве разработчика для Центра Интернета вещей](iot-hub-devguide.md).