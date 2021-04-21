---
title: Начало работы с функцией управления Центром Интернета вещей Azure (Java) | Документы Майкрософт
description: Использование функции управления устройствами в Центре Интернета вещей Azure для начала удаленной перезагрузки устройства. Используйте пакет SDK для устройств Azure IoT для Java, чтобы реализовать приложение имитации устройства, включающее прямой метод, и пакет SDK для служб Azure IoT для Java, чтобы реализовать приложение-службу, вызывающее прямой метод.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: conceptual
ms.date: 08/20/2019
ms.custom: mqtt, devx-track-java, devx-track-azurecli
ms.openlocfilehash: 797c795b6c936e2f117a96fed4b4a483876f1c47
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107478086"
---
# <a name="get-started-with-device-management-java"></a>Начало работы с управлением устройствами (Java)

[!INCLUDE [iot-hub-selector-dm-getstarted](../../includes/iot-hub-selector-dm-getstarted.md)]

В этом учебнике описаны следующие процедуры.

* Создание экземпляра Центра Интернета вещей на портале Azure и удостоверения устройства в экземпляре Центра Интернета вещей.

* Создание приложения имитации устройства, которое реализует прямой метод для перезагрузки устройства. Прямые методы вызываются из облака.

* Создание приложения, вызывающего прямой метод перезагрузки в приложении имитации устройства с помощью Центра Интернета вещей. Это приложение затем отслеживает передаваемые свойства с устройства, ожидая завершения операции перезагрузки.

Завершив работу с этим руководством, вы создадите два консольных приложения Java:

**simulated-device**. Это приложение:

* подключается к вашему Центру Интернета вещей с помощью идентификатора устройства, созданного ранее;

* получает вызов прямого метода перезагрузки;

* имитирует физическую перезагрузку;

* возвращает время последней перезагрузки через переданное свойство.

**trigger-reboot**. Это приложение:

* вызывает прямой метод в приложении имитации устройства;

* отображает ответ на вызов прямого метода, отправленный имитируемым устройством;

* отображает обновленные переданные свойства.

> [!NOTE]
> Статья о [пакетах SDK для Центра Интернета вещей Azure](iot-hub-devguide-sdks.md) содержит сведения о различных пакетах SDK, которые можно использовать для создания приложений, работающих на устройствах и в серверной части решения.

## <a name="prerequisites"></a>Предварительные требования

* [Пакет SDK для Java SE 8](/java/azure/jdk/). Щелкните ссылку **Java 8** в разделе **Долгосрочная поддержка**, чтобы скачать все необходимое для работы с JDK 8.

* [Maven 3](https://maven.apache.org/download.cgi)

* Активная учетная запись Azure. Если ее нет, можно создать [бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/) всего за несколько минут.

* Убедитесь, что в брандмауэре открыт порт 8883. Пример устройства в этой статье использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Регистрация нового устройства в центре Интернета вещей

[!INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## <a name="get-the-iot-hub-connection-string"></a>Получение строки подключения центра Интернета вещей

[!INCLUDE [iot-hub-howto-device-management-shared-access-policy-text](../../includes/iot-hub-howto-device-management-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="trigger-a-remote-reboot-on-the-device-using-a-direct-method"></a>Активация удаленной перезагрузки на устройстве с помощью прямого метода

В этом разделе приведена процедура создания консольного приложения Java, которое выполняет следующие действия:

1. вызывает прямой метод перезагрузки в приложении имитации устройства;

2. отображает ответ;

3. опрашивает переданные свойства, отправляемые с устройства, для определения завершения перезагрузки.

Это консольное приложение подключается к Центру Интернета вещей для вызова прямого метода и чтения переданных свойств.

1. Создайте пустую папку с именем **dm-get-started**.

2. В папке **dm-get-started** создайте проект Maven с именем **trigger-reboot**, выполнив в командной строке следующую команду:

    ```cmd/sh
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=trigger-reboot -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

3. В командной строке перейдите к папке **trigger-reboot**.

4. Откройте в текстовом редакторе файл **pom.xml** из папки **trigger-reboot** и добавьте зависимости, приведенные ниже, в узел **dependencies**. Эта зависимость позволит вам использовать в приложении пакет iot-service-client для обмена данными с Центром Интернета вещей:

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-service-client</artifactId>
      <version>1.17.1</version>
      <type>jar</type>
    </dependency>
    ```

    > [!NOTE]
    > Наличие последней версии пакета **iot-service-client** можно проверить с помощью [поиска Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-service-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

5. Добавьте следующий узел, **build**, после узла **dependencies**. Эта конфигурация дает указание Maven использовать Java версии 1.8 для создания приложения:

    ```xml
    <build>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.3</version>
          <configuration>
            <source>1.8</source>
            <target>1.8</target>
          </configuration>
        </plugin>
      </plugins>
    </build>
    ```

6. Сохраните и закройте файл **pom.xml**.

7. Откройте в текстовом редакторе файл **trigger-reboot\src\main\java\com\mycompany\app\App.java**.

8. Добавьте в файл следующие инструкции **import** .

    ```java
    import com.microsoft.azure.sdk.iot.service.devicetwin.DeviceMethod;
    import com.microsoft.azure.sdk.iot.service.devicetwin.MethodResult;
    import com.microsoft.azure.sdk.iot.service.exceptions.IotHubException;
    import com.microsoft.azure.sdk.iot.service.devicetwin.DeviceTwin;
    import com.microsoft.azure.sdk.iot.service.devicetwin.DeviceTwinDevice;

    import java.io.IOException;
    import java.util.concurrent.TimeUnit;
    import java.util.concurrent.Executors;
    import java.util.concurrent.ExecutorService;
    ```

9. Добавьте в класс **App** . Замените `{youriothubconnectionstring}` строкой подключения Центра Интернета вещей, скопированной ранее в разделе [Получение строки подключения центра Интернета вещей](#get-the-iot-hub-connection-string):

    ```java
    public static final String iotHubConnectionString = "{youriothubconnectionstring}";
    public static final String deviceId = "myDeviceId";

    private static final String methodName = "reboot";
    private static final Long responseTimeout = TimeUnit.SECONDS.toSeconds(30);
    private static final Long connectTimeout = TimeUnit.SECONDS.toSeconds(5);
    ```

10. Для реализации потока, который считывает переданные свойства с двойника устройства каждые 10 секунд, добавьте следующий вложенный класс в класс **App**:

    ```java
    private static class ShowReportedProperties implements Runnable {
      public void run() {
        try {
          DeviceTwin deviceTwins = DeviceTwin.createFromConnectionString(iotHubConnectionString);
          DeviceTwinDevice twinDevice = new DeviceTwinDevice(deviceId);
          while (true) {
            System.out.println("Get reported properties from device twin");
            deviceTwins.getTwin(twinDevice);
            System.out.println(twinDevice.reportedPropertiesToString());
            Thread.sleep(10000);
          }
        } catch (Exception ex) {
          System.out.println("Exception reading reported properties: " + ex.getMessage());
        }
      }
    }
    ```

11. Измените сигнатуру **основного** метода, чтобы создавать следующие исключения:

    ```java
    public static void main(String[] args) throws IOException
    ```

12. Для вызова прямого метода перезагрузки на имитируемом устройстве замените код в методе **main** на следующий код:

    ```java
    System.out.println("Starting sample...");
    DeviceMethod methodClient = DeviceMethod.createFromConnectionString(iotHubConnectionString);

    try
    {
      System.out.println("Invoke reboot direct method");
      MethodResult result = methodClient.invoke(deviceId, methodName, responseTimeout, connectTimeout, null);

      if(result == null)
      {
        throw new IOException("Invoke direct method reboot returns null");
      }
      System.out.println("Invoked reboot on device");
      System.out.println("Status for device:   " + result.getStatus());
      System.out.println("Message from device: " + result.getPayload());
    }
    catch (IotHubException e)
    {
        System.out.println(e.getMessage());
    }
    ```

13. Добавьте следующий код в метод **main** для запуска потока, опрашивающего переданные свойства на имитируемом устройстве:

    ```java
    ShowReportedProperties showReportedProperties = new ShowReportedProperties();
    ExecutorService executor = Executors.newFixedThreadPool(1);
    executor.execute(showReportedProperties);
    ```

14. Чтобы иметь возможность остановить приложение, добавьте следующий код в метод **main**:

    ```java
    System.out.println("Press ENTER to exit.");
    System.in.read();
    executor.shutdownNow();
    System.out.println("Shutting down sample...");
    ```

15. Сохраните и закройте файл **trigger-reboot\src\main\java\com\mycompany\app\App.java**.

16. Выполните сборку внутреннего приложения **trigger-reboot** и исправьте ошибки (при наличии). В командной строке перейдите к папке **trigger-reboot** и выполните следующую команду:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="create-a-simulated-device-app"></a>Создание приложения виртуального устройства

В этом разделе приведена процедура создания консольного приложения Java, которое имитирует устройство. Приложение прослушивает вызовы прямого метода перезагрузки из вашего Центра Интернета вещей и немедленно отвечает на такие вызовы. Затем приложение на некоторое время переходит в спящий режим, чтобы имитировать процесс перезагрузки, после чего использует переданное свойство для уведомления внутреннего приложения **trigger-reboot** о завершении перезагрузки.

1. В папке **dm-get-started** создайте проект Maven с именем **simulated-device**, выполнив в командной строке следующую команду:

    ```cmd/sh
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=simulated-device -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. В командной строке перейдите к папке **simulated-device**.

3. Откройте в текстовом редакторе файл **pom.xml** из папки **simulated-device** и добавьте приведенную ниже зависимость в узел **dependencies**. Эта зависимость позволит вам использовать в приложении пакет iot-service-client для обмена данными с Центром Интернета вещей:

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-device-client</artifactId>
      <version>1.17.5</version>
    </dependency>
    ```

    > [!NOTE]
    > Наличие последней версии пакета **iot-device-client** можно проверить с помощью [поиска Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-device-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

4. Добавьте указанную ниже зависимость в узел **dependencies**. Эта зависимость настраивает NOP для интерфейса ведения журнала Apache [SLF4J](https://www.slf4j.org/), который используется пакетом SDK клиента устройства для реализации ведения журнала. Эта конфигурация является необязательной, но если ее опустить, при запуске приложения может появиться предупреждение в консоли. Дополнительные сведения о ведении журнала в пакете SDK клиента устройства см. в разделе [Ведение журнала](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-samples/readme.md#logging) в файле сведений *Примеры для пакета SDK для устройств Azure IoT для Java*.

    ```xml
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-nop</artifactId>
      <version>1.7.28</version>
    </dependency>
    ```

5. Добавьте следующий узел, **build**, после узла **dependencies**. Эта конфигурация дает указание Maven использовать Java версии 1.8 для создания приложения:

    ```xml
    <build>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.3</version>
          <configuration>
            <source>1.8</source>
            <target>1.8</target>
          </configuration>
        </plugin>
      </plugins>
    </build>
    ```

6. Сохраните и закройте файл **pom.xml**.

7. Откройте в текстовом редакторе исходный файл **simulated-device\src\main\java\com\mycompany\app\App.java**.

8. Добавьте в файл следующие инструкции **import** .

    ```java
    import com.microsoft.azure.sdk.iot.device.*;
    import com.microsoft.azure.sdk.iot.device.DeviceTwin.*;

    import java.io.IOException;
    import java.net.URISyntaxException;
    import java.time.LocalDateTime;
    import java.util.Scanner;
    import java.util.Set;
    import java.util.HashSet;
    ```

9. Добавьте в класс **App** . Замените `{yourdeviceconnectionstring}` строкой подключения устройства, записанной в разделе [Регистрация нового устройства в центре Интернета вещей](#register-a-new-device-in-the-iot-hub):

    ```java
    private static final int METHOD_SUCCESS = 200;
    private static final int METHOD_NOT_DEFINED = 404;

    private static IotHubClientProtocol protocol = IotHubClientProtocol.MQTT;
    private static String connString = "{yourdeviceconnectionstring}";
    private static DeviceClient client;
    ```

10. Чтобы реализовать обработчик обратного вызова для событий состояния прямого метода, добавьте следующий вложенный класс в класс **App**:

    ```java
    protected static class DirectMethodStatusCallback implements IotHubEventCallback
    {
      public void execute(IotHubStatusCode status, Object context)
      {
        System.out.println("IoT Hub responded to device method operation with status " + status.name());
      }
    }
    ```

11. Чтобы реализовать обработчик обратного вызова для событий состояния двойника устройства, добавьте следующий вложенный класс в класс **App**:

    ```java
    protected static class DeviceTwinStatusCallback implements IotHubEventCallback
    {
        public void execute(IotHubStatusCode status, Object context)
        {
            System.out.println("IoT Hub responded to device twin operation with status " + status.name());
        }
    }
    ```

12. Чтобы реализовать обработчик обратного вызова для событий свойства, добавьте следующий вложенный класс в класс **App**:

    ```java
    protected static class PropertyCallback implements PropertyCallBack<String, String>
    {
      public void PropertyCall(String propertyKey, String propertyValue, Object context)
      {
        System.out.println("PropertyKey:     " + propertyKey);
        System.out.println("PropertyKvalue:  " + propertyKey);
      }
    }
    ```

13. Для реализации потока для имитации перезагрузка устройства добавьте указанный ниже вложенный класс в класс **App**. Поток бездействует в течение пяти секунд, а затем устанавливает переданное свойство **lastReboot**:

    ```java
    protected static class RebootDeviceThread implements Runnable {
      public void run() {
        try {
          System.out.println("Rebooting...");
          Thread.sleep(5000);
          Property property = new Property("lastReboot", LocalDateTime.now());
          Set<Property> properties = new HashSet<Property>();
          properties.add(property);
          client.sendReportedProperties(properties);
          System.out.println("Rebooted");
        }
        catch (Exception ex) {
          System.out.println("Exception in reboot thread: " + ex.getMessage());
        }
      }
    }
    ```

14. Чтобы реализовать прямой метод на устройстве, добавьте указанный ниже вложенный класс в класс **App**. Когда приложение имитации получает вызов прямого метода **reboot**, оно возвращает вызывающему объекту подтверждение, а затем запускает поток для обработки перезагрузки:

    ```java
    protected static class DirectMethodCallback implements com.microsoft.azure.sdk.iot.device.DeviceTwin.DeviceMethodCallback
    {
      @Override
      public DeviceMethodData call(String methodName, Object methodData, Object context)
      {
        DeviceMethodData deviceMethodData;
        switch (methodName)
        {
          case "reboot" :
          {
            int status = METHOD_SUCCESS;
            System.out.println("Received reboot request");
            deviceMethodData = new DeviceMethodData(status, "Started reboot");
            RebootDeviceThread rebootThread = new RebootDeviceThread();
            Thread t = new Thread(rebootThread);
            t.start();
            break;
          }
          default:
          {
            int status = METHOD_NOT_DEFINED;
            deviceMethodData = new DeviceMethodData(status, "Not defined direct method " + methodName);
          }
        }
        return deviceMethodData;
      }
    }
    ```

15. Измените сигнатуру метода **main**, чтобы создавать следующие исключения:

    ```java
    public static void main(String[] args) throws IOException, URISyntaxException
    ```

16. Для создания экземпляра **DeviceClient** замените код в методе **main** следующим кодом:

    ```java
    System.out.println("Starting device client sample...");
    client = new DeviceClient(connString, protocol);
    ```

17. Чтобы запустить прослушивание вызовов прямого метода, добавьте следующий код в метод **main**:

    ```java
    try
    {
      client.open();
      client.subscribeToDeviceMethod(new DirectMethodCallback(), null, new DirectMethodStatusCallback(), null);
      client.startDeviceTwin(new DeviceTwinStatusCallback(), null, new PropertyCallback(), null);
      System.out.println("Subscribed to direct methods and polling for reported properties. Waiting...");
    }
    catch (Exception e)
    {
      System.out.println("On exception, shutting down \n" + " Cause: " + e.getCause() + " \n" +  e.getMessage());
      client.close();
      System.out.println("Shutting down...");
    }
    ```

18. Для завершения работы симулятора устройства добавьте следующий код в метод **main**:

    ```java
    System.out.println("Press any key to exit...");
    Scanner scanner = new Scanner(System.in);
    scanner.nextLine();
    scanner.close();
    client.close();
    System.out.println("Shutting down...");
    ```

19. Сохраните и закройте файл simulated-device\src\main\java\com\mycompany\app\App.java.

20. Создайте приложение **simulated-device** и исправьте все ошибки. В командной строке перейдите к папке **simulated-device** и выполните следующую команду:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="run-the-apps"></a>Запуск приложений

Теперь все готово к запуску приложений.

1. В командной строке в папке **simulated-device** выполните следующую команду, чтобы начать прослушивать вызовы метода перезагрузки из центра Интернета вещей:

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![Приложение имитации устройства Центра Интернета вещей на языке Java для прослушивания вызовов прямого метода перезагрузки](./media/iot-hub-java-java-device-management-getstarted/launchsimulator.png)

2. В командной строке в папке **trigger-reboot** выполните следующую команду, чтобы вызвать метод перезагрузки на имитируемом устройстве из центра Интернета вещей:

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![Приложение службы Центра Интернета вещей на языке Java для вызова прямого метода перезагрузки](./media/iot-hub-java-java-device-management-getstarted/triggerreboot.png)

3. Ответ имитируемого устройства на вызов прямого метода перезагрузки:

    ![Ответ приложения имитации устройства Центра Интернета вещей на языке Java на вызов прямого метода](./media/iot-hub-java-java-device-management-getstarted/respondtoreboot.png)

[!INCLUDE [iot-hub-dm-followup](../../includes/iot-hub-dm-followup.md)]
