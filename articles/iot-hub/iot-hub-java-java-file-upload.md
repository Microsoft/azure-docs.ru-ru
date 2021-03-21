---
title: Отправка файлов с устройств в Центр Интернета вещей Azure с помощью Java | Документация Майкрософт
description: Сведения об отправке файлов с устройства в облако с помощью пакета SDK для устройств Центра Интернета вещей Azure для Java. Отправленные файлы хранятся в контейнере больших двоичных объектов службы хранилища Azure.
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
ms.openlocfilehash: 3529361cacf0890b7c4752bbd745a9240020b4f3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102217828"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub-java"></a>Передача файлов с устройства в облако с помощью Центра Интернета вещей (Java)

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

В этом руководстве используется код статьи [Отправка сообщений из облака на устройство с помощью Центра Интернета вещей](iot-hub-java-java-c2d.md), чтобы показать, как использовать [возможности передачи файлов Центра Интернета вещей](iot-hub-devguide-file-upload.md) для передачи файлов в [хранилище BLOB-объектов Azure](../storage/index.yml). В этом руководстве описаны следующие процедуры.

* как безопасно предоставить устройству универсальный код ресурса (URI) большого двоичного объекта Azure для передачи файла;

* как использовать уведомления о передаче файлов Центра Интернета вещей для активации обработки файла в серверной части приложения.

В кратком руководстве [Отправка данных телеметрии с устройства в центр Интернета вещей](quickstart-send-telemetry-java.md) и руководстве [Отправка сообщений из облака на устройство с помощью Центра Интернета вещей](iot-hub-java-java-c2d.md) описаны базовые функции Центра Интернета вещей для обмена сообщениями между устройством и облаком. В статье [Руководство. Настройка маршрутизации сообщений с Центром Интернета вещей](tutorial-routing.md) описан способ надежного хранения сообщений, отправляемых с устройства в облако, в хранилище BLOB-объектов Azure. Тем не менее в некоторых случаях не просто сопоставить данные, отправляемые устройством, с относительно небольшими сообщениями, отправляемыми из устройства в облако, которые принимает Центр Интернета вещей. Пример:

* Большие файлы, содержащие изображения.
* Видео
* Данные вибрации с высокой частотой выборки.
* Некоторые виды предварительно обработанных данных.

Эти файлы обычно обрабатываются в виде пакета в облаке с помощью таких инструментов, как [фабрика данных Azure](../data-factory/introduction.md) или стек [Hadoop](../hdinsight/index.yml). При передаче файлов с устройства вы можете рассчитывать на безопасность и надежность Центра Интернета вещей.

В конце этого руководства запускаются два консольных приложения для Java:

* **simulated-device** — измененная версия приложения, созданного в руководстве [Отправка сообщений из облака на устройство с помощью Центра Интернета вещей]. Это приложение передает файл в хранилище с помощью универсального кода ресурса (URI) SAS, предоставленного вашим Центром Интернета вещей.

* **read-file-upload-notification** — приложение, которое получает уведомления о передаче файлов от Центра Интернета вещей.

> [!NOTE]
> Существуют пакеты SDK для устройств Центра Интернета вещей Azure, обеспечивающие поддержку многих платформ устройств и языков (включая C, .NET и JavaScript) в Центре Интернета вещей. Пошаговые инструкции по подключению устройства к Центру Интернета вещей Azure см. в [Центре разработчика для Центра Интернета вещей Azure](https://azure.microsoft.com/develop/iot).

[!INCLUDE [iot-hub-include-x509-ca-signed-file-upload-support-note](../../includes/iot-hub-include-x509-ca-signed-file-upload-support-note.md)]

## <a name="prerequisites"></a>Предварительные требования

* [Пакет SDK для Java SE 8](/java/azure/jdk/). Щелкните ссылку **Java 8** в разделе **Долгосрочная поддержка**, чтобы скачать все необходимое для работы с JDK 8.

* [Maven 3](https://maven.apache.org/download.cgi)

* Активная учетная запись Azure. Если ее нет, можно создать [бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/) всего за несколько минут.

* Убедитесь, что в брандмауэре открыт порт 8883. Пример устройства в этой статье использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [iot-hub-associate-storage](../../includes/iot-hub-associate-storage.md)]

## <a name="upload-a-file-from-a-device-app"></a>Передача файла из приложения устройства

В этом разделе вы измените приложение устройства, созданное в руководстве [Отправка сообщений из облака на устройство с помощью Центра Интернета вещей](iot-hub-java-java-c2d.md), чтобы передать файл в Центр Интернета вещей.

1. Скопируйте файл образа в папку `simulated-device` и переименуйте его на `myimage.png`.

2. Откройте в текстовом редакторе файл `simulated-device\src\main\java\com\mycompany\app\App.java`.

3. Добавьте объявление переменной в класс **App**:

    ```java
    private static String fileName = "myimage.png";
    ```

4. Для обработки сообщений обратного вызова состояния передачи файлов добавьте приведенный ниже вложенный класс в класс **App**:

    ```java
    // Define a callback method to print status codes from IoT Hub.
    protected static class FileUploadStatusCallBack implements IotHubEventCallback {
      public void execute(IotHubStatusCode status, Object context) {
        System.out.println("IoT Hub responded to file upload for " + fileName
            + " operation with status " + status.name());
      }
    }
    ```

5. Для отправки образов в Центр Интернета вещей добавьте следующий метод в класс **App**:

    ```java
    // Use IoT Hub to upload a file asynchronously to Azure blob storage.
    private static void uploadFile(String fullFileName) throws FileNotFoundException, IOException
    {
      File file = new File(fullFileName);
      InputStream inputStream = new FileInputStream(file);
      long streamLength = file.length();

      client.uploadToBlobAsync(fileName, inputStream, streamLength, new FileUploadStatusCallBack(), null);
    }
    ```

6. Измените метод **main** для вызова метода **uploadFile**, как показано в следующем фрагменте кода:

    ```java
    client.open();

    try
    {
      // Get the filename and start the upload.
      String fullFileName = System.getProperty("user.dir") + File.separator + fileName;
      uploadFile(fullFileName);
      System.out.println("File upload started with success");
    }
    catch (Exception e)
    {
      System.out.println("Exception uploading file: " + e.getCause() + " \nERROR: " + e.getMessage());
    }

    MessageSender sender = new MessageSender();
    ```

7. Используйте следующую команду, чтобы создать приложение **simulated-device**, и проверьте наличие ошибок:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="get-the-iot-hub-connection-string"></a>Получение строки подключения центра Интернета вещей

В этой статье вы создадите серверную службу для получения сообщений с уведомлением о передаче файлов из центра Интернета вещей, созданного в кратком руководстве [Отправка данных телеметрии с устройства в центр Интернета вещей](quickstart-send-telemetry-java.md). Для получения сообщений об отправке файлов службе требуется разрешение **service connect**. По умолчанию каждый Центр Интернета вещей создается с помощью политики общего доступа, называемой **службой**, которая предоставляет это разрешение.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="receive-a-file-upload-notification"></a>Получение уведомления о передачи файла

В этом разделе вам предстоит создать консольное приложение для Java, которое получает уведомления об отправке файлов из Центра Интернета вещей.

1. Создайте проект Maven **read-file-upload-notification**, выполнив следующую команду в командной строке. Обратите внимание, что это одна длинная команда.

    ```cmd/sh
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=read-file-upload-notification -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. В командной строке перейдите к новой папке `read-file-upload-notification`.

3. Откройте в текстовом редакторе файл `pom.xml` из папки `read-file-upload-notification` и добавьте зависимости, приведенные ниже, в узел **dependencies**. Добавление зависимости позволяет использовать в приложении пакет **iothub-java-service-client** для обмена данными со службой Центра Интернета вещей.

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-service-client</artifactId>
      <version>1.7.23</version>
    </dependency>
    ```

    > [!NOTE]
    > Наличие последней версии пакета **iot-service-client** можно проверить с помощью [поиска Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-service-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

4. Сохраните и закройте файл `pom.xml`.

5. Откройте в текстовом редакторе файл `read-file-upload-notification\src\main\java\com\mycompany\app\App.java`.

6. Добавьте в файл следующие инструкции **import** .

    ```java
    import com.microsoft.azure.sdk.iot.service.*;
    import java.io.IOException;
    import java.net.URISyntaxException;
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;
    ```

7. Добавьте в класс **App** . Замените заполнитель `{Your IoT Hub connection string}` строкой подключения Центра Интернета вещей, скопированной в разделе [Получение строки подключения центра Интернета вещей](#get-the-iot-hub-connection-string):

    ```java
    private static final String connectionString = "{Your IoT Hub connection string}";
    private static final IotHubServiceClientProtocol protocol = IotHubServiceClientProtocol.AMQPS;
    private static FileUploadNotificationReceiver fileUploadNotificationReceiver = null;
    ```

8. Для вывода сведений о передаче файла в консоль добавьте следующий вложенный класс в класс **App**:

    ```java
    // Create a thread to receive file upload notifications.
    private static class ShowFileUploadNotifications implements Runnable {
      public void run() {
        try {
          while (true) {
            System.out.println("Receive file upload notifications...");
            FileUploadNotification fileUploadNotification = fileUploadNotificationReceiver.receive();
            if (fileUploadNotification != null) {
              System.out.println("File Upload notification received");
              System.out.println("Device Id : " + fileUploadNotification.getDeviceId());
              System.out.println("Blob Uri: " + fileUploadNotification.getBlobUri());
              System.out.println("Blob Name: " + fileUploadNotification.getBlobName());
              System.out.println("Last Updated : " + fileUploadNotification.getLastUpdatedTimeDate());
              System.out.println("Blob Size (Bytes): " + fileUploadNotification.getBlobSizeInBytes());
              System.out.println("Enqueued Time: " + fileUploadNotification.getEnqueuedTimeUtcDate());
            }
          }
        } catch (Exception ex) {
          System.out.println("Exception reading reported properties: " + ex.getMessage());
        }
      }
    }
    ```

9. Чтобы запустить поток, который ожидает уведомления о передаче файлов, добавьте следующий код в метод **main**:

    ```java
    public static void main(String[] args) throws IOException, URISyntaxException, Exception {
      ServiceClient serviceClient = ServiceClient.createFromConnectionString(connectionString, protocol);

      if (serviceClient != null) {
        serviceClient.open();

        // Get a file upload notification receiver from the ServiceClient.
        fileUploadNotificationReceiver = serviceClient.getFileUploadNotificationReceiver();
        fileUploadNotificationReceiver.open();

        // Start the thread to receive file upload notifications.
        ShowFileUploadNotifications showFileUploadNotifications = new ShowFileUploadNotifications();
        ExecutorService executor = Executors.newFixedThreadPool(1);
        executor.execute(showFileUploadNotifications);

        System.out.println("Press ENTER to exit.");
        System.in.read();
        executor.shutdownNow();
        System.out.println("Shutting down sample...");
        fileUploadNotificationReceiver.close();
        serviceClient.close();
      }
    }
    ```

10. Сохраните и закройте файл `read-file-upload-notification\src\main\java\com\mycompany\app\App.java`.

11. Используйте следующую команду, чтобы создать приложение **read-file-upload-notification**, и проверьте наличие ошибок:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="run-the-applications"></a>Запуск приложений

Теперь все готово к запуску приложений.

В командной строке в папке `read-file-upload-notification` выполните следующую команду:

```cmd/sh
mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
```

В командной строке в папке `simulated-device` выполните следующую команду:

```cmd/sh
mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
```

На следующем снимке экрана показаны выходные данные приложения **simulated-device**:

![Выходные данные приложения simulated-device](media/iot-hub-java-java-upload/simulated-device.png)

На следующем снимке экрана показаны выходные данные приложения **read-file-upload-notification**:

![Выходные данные приложения read-file-upload-notification](media/iot-hub-java-java-upload/read-file-upload-notification.png)

На портале можно просмотреть отправленный файл в контейнере хранилища, который вы настроили.

![Отправленный файл](media/iot-hub-java-java-upload/uploaded-file.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство показано, как использовать возможности передачи файлов Центра Интернета вещей, чтобы упростить передачу файлов из устройств. Изучение функций и сценариев Центра Интернета вещей можно продолжить в следующих руководствах:

* [Создание Центра Интернета вещей с помощью шаблона Azure Resource Manager (PowerShell)](iot-hub-rm-template-powershell.md)

* [Пакет SDK для устройств Azure IoT для C](iot-hub-device-sdk-c-intro.md)

* [Пакеты SDK для Центра Интернета вещей Azure](iot-hub-devguide-sdks.md)

Для дальнейшего изучения возможностей Центра Интернета вещей см. следующие статьи:

* [Моделирование устройства с помощью Edge Интернета вещей](../iot-edge/quickstart-linux.md).