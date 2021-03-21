---
title: Планирование заданий с помощью Центра Интернета вещей Azure (Java) | Документация Майкрософт
description: Планирование заданий в Центре Интернета вещей Azure для вызова прямого метода или изменения требуемых свойств на нескольких устройствах. Вы создадите приложения имитации устройства с помощью пакета SDK для устройств Azure IoT для Java, а также приложение-службу для выполнения заданий с помощью пакета SDK для служб Azure IoT для Java.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: conceptual
ms.date: 08/16/2019
ms.custom: mqtt, devx-track-java
ms.openlocfilehash: 3e98cfc2d8c7fb8d40c8565a1c620f123ce171ff
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102217845"
---
# <a name="schedule-and-broadcast-jobs-java"></a>Планирование и трансляция заданий (Java)

[!INCLUDE [iot-hub-selector-schedule-jobs](../../includes/iot-hub-selector-schedule-jobs.md)]

Центр Интернета вещей Azure позволяет планировать и отслеживать задания по обновлению для миллионов устройств. Что можно сделать с помощью заданий?

* Обновление требуемых свойств
* Обновление тегов
* Вызов прямых методов

Задание выступает в роли оболочки для одного из этих действий и отслеживает его выполнение для определенного набора устройств. Запрос на двойнике устройства определяет набор устройств, для которых будет выполнено задание. Например, внутреннее приложение может использовать задание для вызова прямого метода перезапуска на 10 000 устройств. Вы можете определить набор устройств с помощью запроса на двойнике устройства и запланировать момент времени в будущем для выполнения задания. Задание отслеживает ход выполнения по мере того, как каждое из устройств получает вызов и выполняет прямой метод перезагрузки.

Дополнительные сведения об этих возможностях см. в указанных ниже статьях.

* Двойник устройства и свойства: [Начало работы с двойниками устройств](iot-hub-java-java-twin-getstarted.md)

* Прямые методы: [Руководство для разработчика Центра Интернета вещей — прямые методы](iot-hub-devguide-direct-methods.md) и [Руководство. Использование прямых методов](quickstart-control-device-java.md)

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

В этом учебнике описаны следующие процедуры.

* Создание приложения для устройств, которое реализует прямой метод c именем **lockDoor**. Приложение для устройств также получает запросы на изменение требуемых свойств от внутреннего приложения.

* Создание внутреннего приложения, которое создает задание для вызова прямого метода **lockDoor** на нескольких устройствах. Еще одно задание отправляет обновления требуемых свойств на несколько устройств.

По завершении работы с этим руководством у вас будут готовы консольное приложение устройства и консольное приложение серверной части.

**simulated-device** подключается к Центру Интернета вещей, выполняет прямой метод **lockDoor** и обрабатывает изменения требуемых свойств.

**schedule-jobs** использует задания для вызова прямого метода **lockDoor** и обновления на нескольких устройствах требуемых свойств, установленных на двойнике устройства.

> [!NOTE]
> Статья [Общие сведения о пакетах SDK для Azure IoT и их использование](iot-hub-devguide-sdks.md) содержит сведения о разных пакетах SDK для Интернета вещей Azure, с помощью которых можно создать приложения для устройств и внутренние приложения.

## <a name="prerequisites"></a>Предварительные требования

* [Пакет SDK для Java SE 8](/java/azure/jdk/). Щелкните ссылку **Java 8** в разделе **Долгосрочная поддержка**, чтобы скачать все необходимое для работы с JDK 8.

* [Maven 3](https://maven.apache.org/download.cgi)

* Активная учетная запись Azure. Если ее нет, можно создать [бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/) всего за несколько минут.

* Убедитесь, что в брандмауэре открыт порт 8883. Пример устройства в этой статье использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Регистрация нового устройства в центре Интернета вещей

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

Для добавления устройства в центр Интернета вещей можно также использовать [расширение Интернета вещей для Azure CLI](https://github.com/Azure/azure-iot-cli-extension).

## <a name="get-the-iot-hub-connection-string"></a>Получение строки подключения центра Интернета вещей

[!INCLUDE [iot-hub-howto-schedule-jobs-shared-access-policy-text](../../includes/iot-hub-howto-schedule-jobs-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-registryrw-connection-string](../../includes/iot-hub-include-find-registryrw-connection-string.md)]

## <a name="create-the-service-app"></a>Создание приложения службы

В этом разделе приведена процедура создания консольного приложения Java, которое с помощью заданий выполняет следующие действия.

* Вызов прямого метода **lockDoor** на нескольких устройствах.

* Отправка изменений требуемых свойств на несколько устройств.

Создание приложения.

1. На компьютере разработки создайте пустую папку с именем **iot-java-schedule-jobs**.

2. В папке **iot-java-schedule-jobs** создайте проект Maven с именем **schedule-jobs**, выполнив приведенную ниже команду в командной строке. Обратите внимание, что это одна длинная команда.

   ```cmd/sh
   mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=schedule-jobs -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   ```

3. В командной строке перейдите к папке **schedule-jobs**.

4. Откройте в текстовом редакторе файл **pom.xml** из папки **schedule-jobs** и добавьте зависимости, приведенные ниже, в узел **dependencies**. Эта зависимость позволит вам использовать в приложении пакет **iot-service-client** для обмена данными с Центром Интернета вещей:

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

7. Откройте в текстовом редакторе файл **schedule-jobs\src\main\java\com\mycompany\app\App.java**.

8. Добавьте в файл следующие инструкции **import** .

    ```java
    import com.microsoft.azure.sdk.iot.service.devicetwin.DeviceTwinDevice;
    import com.microsoft.azure.sdk.iot.service.devicetwin.Pair;
    import com.microsoft.azure.sdk.iot.service.devicetwin.Query;
    import com.microsoft.azure.sdk.iot.service.devicetwin.SqlQuery;
    import com.microsoft.azure.sdk.iot.service.jobs.JobClient;
    import com.microsoft.azure.sdk.iot.service.jobs.JobResult;
    import com.microsoft.azure.sdk.iot.service.jobs.JobStatus;

    import java.util.Date;
    import java.time.Instant;
    import java.util.HashSet;
    import java.util.Set;
    import java.util.UUID;
    ```

9. Добавьте в класс **App** . Замените `{youriothubconnectionstring}` строкой подключения центра Интернета вещей, скопированной ранее в разделе [Получение строки подключения центра Интернета вещей](#get-the-iot-hub-connection-string):

    ```java
    public static final String iotHubConnectionString = "{youriothubconnectionstring}";
    public static final String deviceId = "myDeviceId";

    // How long the job is permitted to run without
    // completing its work on the set of devices
    private static final long maxExecutionTimeInSeconds = 30;
    ```

10. Добавьте следующий метод в класс **App**, чтобы запланировать задание для изменения требуемых свойств **Building** и **Floor** в двойнике устройства:

    ```java
    private static JobResult scheduleJobSetDesiredProperties(JobClient jobClient, String jobId) {
      DeviceTwinDevice twin = new DeviceTwinDevice(deviceId);
      Set<Pair> desiredProperties = new HashSet<Pair>();
      desiredProperties.add(new Pair("Building", 43));
      desiredProperties.add(new Pair("Floor", 3));
      twin.setDesiredProperties(desiredProperties);
      // Optimistic concurrency control
      twin.setETag("*");

      // Schedule the update twin job to run now
      // against a single device
      System.out.println("Schedule job " + jobId + " for device " + deviceId);
      try {
        JobResult jobResult = jobClient.scheduleUpdateTwin(jobId, 
          "deviceId='" + deviceId + "'",
          twin,
          new Date(),
          maxExecutionTimeInSeconds);
        return jobResult;
      } catch (Exception e) {
        System.out.println("Exception scheduling desired properties job: " + jobId);
        System.out.println(e.getMessage());
        return null;
      }
    }
    ```

11. Чтобы запланировать задание для вызова метода **lockDoor**, добавьте следующий метод в класс **App**:

    ```java
    private static JobResult scheduleJobCallDirectMethod(JobClient jobClient, String jobId) {
      // Schedule a job now to call the lockDoor direct method
      // against a single device. Response and connection
      // timeouts are set to 5 seconds.
      System.out.println("Schedule job " + jobId + " for device " + deviceId);
      try {
        JobResult jobResult = jobClient.scheduleDeviceMethod(jobId,
          "deviceId='" + deviceId + "'",
          "lockDoor",
          5L, 5L, null,
          new Date(),
          maxExecutionTimeInSeconds);
        return jobResult;
      } catch (Exception e) {
        System.out.println("Exception scheduling direct method job: " + jobId);
        System.out.println(e.getMessage());
        return null;
      }
    };
    ```

12. Чтобы отслеживать задания, добавьте приведенный ниже метод в класс **App**.

    ```java
    private static void monitorJob(JobClient jobClient, String jobId) {
      try {
        JobResult jobResult = jobClient.getJob(jobId);
        if(jobResult == null)
        {
          System.out.println("No JobResult for: " + jobId);
          return;
        }
        // Check the job result until it's completed
        while(jobResult.getJobStatus() != JobStatus.completed)
        {
          Thread.sleep(100);
          jobResult = jobClient.getJob(jobId);
          System.out.println("Status " + jobResult.getJobStatus() + " for job " + jobId);
        }
        System.out.println("Final status " + jobResult.getJobStatus() + " for job " + jobId);
      } catch (Exception e) {
        System.out.println("Exception monitoring job: " + jobId);
        System.out.println(e.getMessage());
        return;
      }
    }
    ```

13. Чтобы получить сведения о выполнении запущенных заданий, добавьте следующий метод:

    ```java
    private static void queryDeviceJobs(JobClient jobClient, String start) throws Exception {
      System.out.println("\nQuery device jobs since " + start);

      // Create a jobs query using the time the jobs started
      Query deviceJobQuery = jobClient
          .queryDeviceJob(SqlQuery.createSqlQuery("*", SqlQuery.FromType.JOBS, "devices.jobs.startTimeUtc > '" + start + "'", null).getQuery());

      // Iterate over the list of jobs and print the details
      while (jobClient.hasNextJob(deviceJobQuery)) {
        System.out.println(jobClient.getNextJob(deviceJobQuery));
      }
    }
    ```

14. Обновите подпись метода **main**, добавив следующее предложение `throws`:

    ```java
    public static void main( String[] args ) throws Exception
    ```

15. Чтобы два задания запускались и отслеживались последовательно, замените код в методе **main** на следующий:

    ```java
    // Record the start time
    String start = Instant.now().toString();

    // Create JobClient
    JobClient jobClient = JobClient.createFromConnectionString(iotHubConnectionString);
    System.out.println("JobClient created with success");

    // Schedule twin job desired properties
    // Maximum concurrent jobs is 1 for Free and S1 tiers
    String desiredPropertiesJobId = "DPCMD" + UUID.randomUUID();
    scheduleJobSetDesiredProperties(jobClient, desiredPropertiesJobId);
    monitorJob(jobClient, desiredPropertiesJobId);

    // Schedule twin job direct method
    String directMethodJobId = "DMCMD" + UUID.randomUUID();
    scheduleJobCallDirectMethod(jobClient, directMethodJobId);
    monitorJob(jobClient, directMethodJobId);

    // Run a query to show the job detail
    queryDeviceJobs(jobClient, start);

    System.out.println("Shutting down schedule-jobs app");
    ```

16. Сохраните и закройте файл **schedule-jobs\src\main\java\com\mycompany\app\App.java**.

17. Создайте приложение **schedule-jobs** и исправьте обнаруженные ошибки. В командной строке перейдите к папке **schedule-jobs** и выполните следующую команду:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="create-a-device-app"></a>Создание приложения устройства

В этом разделе вы создадите консольное приложение Java, которое обрабатывает требуемые свойства, отправленные из Центр Интернета вещей реализует непосредственный вызов методов.

1. В папке **iot-java-schedule-jobs** создайте проект Maven с именем **simulated-device**, выполнив в командной строке приведенную ниже команду. Обратите внимание, что это одна длинная команда.

   ```cmd/sh
   mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=simulated-device -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   ```

2. В командной строке перейдите к папке **simulated-device**.

3. Откройте в текстовом редакторе файл **pom.xml** из папки **simulated-device** и добавьте приведенные ниже зависимости в узел **dependencies**. Эта зависимость позволит вам использовать в приложении пакет **iot-device-client** для обмена данными с Центром Интернета вещей:

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

7. Откройте в текстовом редакторе файл **simulated-device\src\main\java\com\mycompany\app\App.java**.

8. Добавьте в файл следующие инструкции **import** .

    ```java
    import com.microsoft.azure.sdk.iot.device.*;
    import com.microsoft.azure.sdk.iot.device.DeviceTwin.*;

    import java.io.IOException;
    import java.net.URISyntaxException;
    import java.util.Scanner;
    ```

9. Добавьте в класс **App** . Замените `{yourdeviceconnectionstring}` строкой подключения устройства, скопированной ранее в разделе [Регистрация нового устройства в центре Интернета вещей](#register-a-new-device-in-the-iot-hub):

    ```java
    private static String connString = "{yourdeviceconnectionstring}";
    private static IotHubClientProtocol protocol = IotHubClientProtocol.MQTT;
    private static final int METHOD_SUCCESS = 200;
    private static final int METHOD_NOT_DEFINED = 404;
    ```

    При создании экземпляра объекта **DeviceClient** в этом примере приложения используется переменная **protocol**.

10. Чтобы выводить оповещения двойника устройства в консоль, добавьте следующий вложенный класс в класс **App**:

    ```java
    // Handler for device twin operation notifications from IoT Hub
    protected static class DeviceTwinStatusCallBack implements IotHubEventCallback {
      public void execute(IotHubStatusCode status, Object context) {
        System.out.println("IoT Hub responded to device twin operation with status " + status.name());
      }
    }
    ```

11. Чтобы выводить оповещения прямого метода в консоль, добавьте следующий вложенный класс в класс **App**:

    ```java
    // Handler for direct method notifications from IoT Hub
    protected static class DirectMethodStatusCallback implements IotHubEventCallback {
      public void execute(IotHubStatusCode status, Object context) {
        System.out.println("IoT Hub responded to direct method operation with status " + status.name());
      }
    }
    ```

12. Чтобы обрабатывать вызовы прямого метода от Центра Интернета вещей, добавьте следующий вложенный класс в класс **App**:

    ```java
    // Handler for direct method calls from IoT Hub
    protected static class DirectMethodCallback
        implements DeviceMethodCallback {
      @Override
      public DeviceMethodData call(String methodName, Object methodData, Object context) {
        DeviceMethodData deviceMethodData;
        switch (methodName) {
          case "lockDoor": {
            System.out.println("Executing direct method: " + methodName);
            deviceMethodData = new DeviceMethodData(METHOD_SUCCESS, "Executed direct method " + methodName);
            break;
          }
          default: {
            deviceMethodData = new DeviceMethodData(METHOD_NOT_DEFINED, "Not defined direct method " + methodName);
          }
        }
        // Notify IoT Hub of result
        return deviceMethodData;
      }
    }
    ```

13. Обновите подпись метода **main**, добавив следующее предложение `throws`:

    ```java
    public static void main( String[] args ) throws IOException, URISyntaxException
    ```

14. Замените код в методе **main** следующим кодом:
    * Создать клиент устройства для взаимодействия с Центром Интернета вещей.
    * Создать объект **Device** для хранения свойств двойника устройства.

    ```java
    // Create a device client
    DeviceClient client = new DeviceClient(connString, protocol);

    // An object to manage device twin desired and reported properties
    Device dataCollector = new Device() {
      @Override
      public void PropertyCall(String propertyKey, Object propertyValue, Object context)
      {
        System.out.println("Received desired property change: " + propertyKey + " " + propertyValue);
      }
    };
    ```

15. Добавьте следующий код в метод **main** для запуска клиентских служб на устройстве:

    ```java
    try {
      // Open the DeviceClient
      // Start the device twin services
      // Subscribe to direct method calls
      client.open();
      client.startDeviceTwin(new DeviceTwinStatusCallBack(), null, dataCollector, null);
      client.subscribeToDeviceMethod(new DirectMethodCallback(), null, new DirectMethodStatusCallback(), null);
    } catch (Exception e) {
      System.out.println("Exception, shutting down \n" + " Cause: " + e.getCause() + " \n" + e.getMessage());
      dataCollector.clean();
      client.closeNow();
      System.out.println("Shutting down...");
    }
    ```

16. Чтобы ожидать нажатия пользователем клавиши **Ввод** перед завершением работы, добавьте следующий код в конец метода **main**:

    ```java
    // Close the app
    System.out.println("Press any key to exit...");
    Scanner scanner = new Scanner(System.in);
    scanner.nextLine();
    dataCollector.clean();
    client.closeNow();
    scanner.close();
    ```

17. Сохраните и закройте файл **simulated-device\src\main\java\com\mycompany\app\App.java**.

18. Создайте приложение **simulated-device** и исправьте все ошибки. В командной строке перейдите к папке **simulated-device** и выполните следующую команду:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="run-the-apps"></a>Запуск приложений

Теперь все готово к запуску консоли приложений.

1. В командной строке, находясь в папке **simulated-device**, выполните следующую команду, которая запускает приложение устройства для прослушивания изменений требуемых свойств и ожидания вызовов прямого метода:

   ```cmd/sh
   mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
   ```

   ![Запуск клиента на устройстве](./media/iot-hub-java-java-schedule-jobs/device-app-1.png)

2. В командной строке в папке `schedule-jobs` выполните следующую команду, чтобы запустить приложение службы **schedule-jobs** для выполнения двух заданий. Первое задание устанавливает значения требуемых свойств, а второе вызывает прямой метод:

   ```cmd\sh
   mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
   ```

   ![Java-приложение службы Центра Интернета вещей создает два задания](./media/iot-hub-java-java-schedule-jobs/service-app-1.png)

3. Приложения для устройств обрабатывает изменения требуемого свойства и вызов прямого метода:

   ![Клиент устройства реагирует на изменения](./media/iot-hub-java-java-schedule-jobs/device-app-2.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике описано использование задания для планирования прямого метода на устройстве и обновления свойств двойника устройства.

Ознакомьтесь со следующими материалами, чтобы узнать как:

* отправить данные телеметрии с устройств (руководство [Подключение устройства к Центру Интернета вещей с помощью Java](quickstart-send-telemetry-java.md));

* управлять устройствами в интерактивном режиме, например включить вентилятор из управляемого пользователем приложения ([Краткое руководство по управлению подключенным к Центру Интернета вещей устройством (Java)](quickstart-control-device-java.md)).