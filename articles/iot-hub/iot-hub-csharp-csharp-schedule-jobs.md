---
title: Планирование заданий c помощью Центра Интернета вещей Azure (.NET и .NET) | Документация Майкрософт
description: Планирование заданий с помощью Центра Интернета вещей Azure для вызова прямого метода на нескольких устройствах. Используйте пакеты SDK для устройств Azure IoT для .NET, чтобы реализовать приложения имитации устройства и приложение службы, на которых будет выполнено задание.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/20/2019
ms.author: robinsh
ms.custom: mqtt, devx-track-csharp
ms.openlocfilehash: 0391afe5562238f488fde7e50d914c46a5c32944
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92142453"
---
# <a name="schedule-and-broadcast-jobs-net"></a>Планирование и трансляция заданий (.NET)

[!INCLUDE [iot-hub-selector-schedule-jobs](../../includes/iot-hub-selector-schedule-jobs.md)]

Центр Интернета вещей Azure позволяет планировать и отслеживать задания по обновлению для миллионов устройств. Что можно сделать с помощью заданий?

* Обновление требуемых свойств

* Обновление тегов

* Вызов прямых методов

Задание выступает в роли оболочки для одного из этих действий и отслеживает его выполнение для определенного набора устройств в соответствии с запросом на двойнике устройства. Например, внутреннее приложение может использовать задание для вызова прямого метода перезапуска на 10 000 устройств. Вы можете определить набор устройств с помощью запроса на двойнике устройства и запланировать момент времени в будущем для выполнения задания. Задание отслеживает ход выполнения по мере того, как каждое из устройств получает вызов и выполняет прямой метод перезагрузки.

Дополнительные сведения об этих возможностях см. в указанных ниже статьях.

* Двойник устройства и свойства: [Начало работы с двойниками устройств](iot-hub-csharp-csharp-twin-getstarted.md) и [Руководство. Использование свойств двойников устройств](tutorial-device-twins.md)

* Прямые методы: [Руководство для разработчика Центра Интернета вещей — прямые методы](iot-hub-devguide-direct-methods.md) и [Руководство. Использование прямых методов](quickstart-control-device-dotnet.md)

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

В этом учебнике описаны следующие процедуры.

* Создание приложения для устройств, которое реализует прямой метод **LockDoor**, вызываемый внутренним приложением.

* Создание внутреннего приложения, которое создает задание для вызова прямого метода **LockDoor** на нескольких устройствах. Еще одно задание отправляет обновления требуемых свойств на несколько устройств.

По завершении работы с этим руководством у вас будет два консольных приложения .NET (C#):

* **SimulateDeviceMethods**. Это приложение подключается к центру Интернета вещей и реализует прямой метод **LockDoor**.

* **ScheduleJob**. Это приложение использует задания для вызова прямого метода **LockDoor** и для обновления на нескольких устройствах свойств, установленных на двойнике устройства.

## <a name="prerequisites"></a>Предварительные требования

* приведенному.

* Активная учетная запись Azure. Если ее нет, можно создать [бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/) всего за несколько минут.

* Убедитесь, что в брандмауэре открыт порт 8883. Пример устройства в этой статье использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Регистрация нового устройства в центре Интернета вещей

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="create-a-simulated-device-app"></a>Создание приложения виртуального устройства

В этом разделе вы создадите консольное приложение .NET, которое отвечает на вызов прямого метода, выполняемый серверной частью решения.

1. В Visual Studio выберите **Создать новый проект**, а затем выберите шаблон проекта **Консольное приложение (.NET Framework)** . Нажмите кнопку **Далее**, чтобы продолжить.

1. В разделе **Настроить новый проект** введите имя проекта *SimulateDeviceMethods* и нажмите **Создать**.

    ![Настройка проекта SimulateDeviceMethods](./media/iot-hub-csharp-csharp-schedule-jobs/configure-device-app.png)

1. В обозревателе решений щелкните правой кнопкой мыши проект **SimulateDeviceMethods** и выберите пункт **Управление пакетами NuGet**.

1. В окне **Диспетчер пакетов NuGet** выберите **Обзор** и найдите **Microsoft.Azure.Devices.Client**. Выберите пункт **Установить**.

    ![Клиентское приложение в окне "Диспетчер пакетов NuGet"](./media/iot-hub-csharp-csharp-schedule-jobs/device-app-nuget.png)

    В результате выполняется скачивание и установка пакета NuGet [SDK для устройств Azure IoT](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/) и его зависимостей, а также добавляется соответствующая ссылка.

1. Добавьте следующие инструкции `using` в начало файла **Program.cs** :

    ```csharp
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using Newtonsoft.Json;
    ```

1. Добавьте следующие поля в класс **Program** . Замените значение заполнителя строкой подключения устройства, записанной в предыдущем разделе.

    ```csharp
    static string DeviceConnectionString = "<yourDeviceConnectionString>";
    static DeviceClient Client = null;
    ```

1. Добавьте следующий код, чтобы реализовать прямой метод на устройстве:

    ```csharp
    static Task<MethodResponse> LockDoor(MethodRequest methodRequest, object userContext)
    {
        Console.WriteLine();
        Console.WriteLine("Locking Door!");
        Console.WriteLine("\nReturning response for method {0}", methodRequest.Name);

        string result = "'Door was locked.'";
        return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
    }
    ```

1. Добавьте следующий метод, чтобы реализовать прослушиватель двойников устройства на устройстве:

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, 
      object userContext)
    {
        Console.WriteLine("Desired property change:");
        Console.WriteLine(JsonConvert.SerializeObject(desiredProperties));
    }
    ```

1. И наконец, добавьте этот код в метод **Main**, чтобы открыть подключение к Центру Интернета вещей и инициализировать прослушиватель метода:

    ```csharp
    try
    {
        Console.WriteLine("Connecting to hub");
        Client = DeviceClient.CreateFromConnectionString(DeviceConnectionString, 
          TransportType.Mqtt);

        Client.SetMethodHandlerAsync("LockDoor", LockDoor, null);
        Client.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null);

        Console.WriteLine("Waiting for direct method call and device twin update\n Press enter to exit.");
        Console.ReadLine();

        Console.WriteLine("Exiting...");

        Client.SetMethodHandlerAsync("LockDoor", null, null);
        Client.CloseAsync().Wait();
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    ```

1. Сохраните результаты работы и создайте решение.

> [!NOTE]
> Для простоты в этом руководстве не реализуются политики повтора. В рабочем коде следует реализовать политики повтора (например, повторную попытку подключения), как указано в статье [Обработка временных сбоев](/azure/architecture/best-practices/transient-faults).
>

## <a name="get-the-iot-hub-connection-string"></a>Получение строки подключения центра Интернета вещей

[!INCLUDE [iot-hub-howto-schedule-jobs-shared-access-policy-text](../../includes/iot-hub-howto-schedule-jobs-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-registryrw-connection-string](../../includes/iot-hub-include-find-registryrw-connection-string.md)]

## <a name="schedule-jobs-for-calling-a-direct-method-and-sending-device-twin-updates"></a>Планирование заданий для вызова прямого метода и обновления свойств двойника устройства

В этом разделе вы создадите консольное приложение .NET на языке C#, которое использует задания для вызова прямого метода **LockDoor** и для обновления свойств на нескольких устройствах.

1. В Visual Studio выберите **Файл** > **Создать** > **Проект**. В окне **Создание проекта** выберите **Консольное приложение (.NET Framework)** и нажмите кнопку **Далее**.

1. В окне **Настроить новый проект** присвойте проекту имя *ScheduleJobtodo*. Для параметра **Решение** выберите **Добавить к решению** и нажмите **Создать**.

    ![Настройка проекта ScheduleJob и присвоение ему имени](./media/iot-hub-csharp-csharp-schedule-jobs/config-schedule-job-app.png)

1. В обозревателе решений щелкните правой кнопкой мыши проект **ScheduleJob** и выберите **Управление пакетами NuGet**.

1. В окне **Диспетчер пакетов NuGet** щелкните **Обзор**, найдите и выберите **Microsoft.Azure.Devices**, а затем щелкните **Установить**.

   В результате выполняется скачивание и установка пакета NuGet [SDK для служб Azure IoT](https://www.nuget.org/packages/Microsoft.Azure.Devices/) и его зависимостей, а также добавляется соответствующая ссылка.

1. Добавьте следующие инструкции `using` в начало файла **Program.cs** :

    ```csharp
    using Microsoft.Azure.Devices;
    using Microsoft.Azure.Devices.Shared;
    ```

1. Добавьте следующую инструкцию `using`, если она отсутствует в инструкциях по умолчанию.

    ```csharp
    using System.Threading;
    using System.Threading.Tasks;
    ```

1. Добавьте следующие поля в класс **Program** . Замените заполнители строкой подключения Центра Интернета вещей, скопированной в разделе [Получение строки подключения центра Интернета вещей](#get-the-iot-hub-connection-string), и именем устройства.

    ```csharp
    static JobClient jobClient;
    static string connString = "<yourIotHubConnectionString>";
    static string deviceId = "<yourDeviceId>";
    ```

1. Добавьте следующий метод в класс **Program** .

    ```csharp
    public static async Task MonitorJob(string jobId)
    {
        JobResponse result;
        do
        {
            result = await jobClient.GetJobAsync(jobId);
            Console.WriteLine("Job Status : " + result.Status.ToString());
            Thread.Sleep(2000);
        } while ((result.Status != JobStatus.Completed) && 
          (result.Status != JobStatus.Failed));
    }
    ```

1. Добавьте следующий метод в класс **Program** .

    ```csharp
    public static async Task StartMethodJob(string jobId)
    {
        CloudToDeviceMethod directMethod = 
          new CloudToDeviceMethod("LockDoor", TimeSpan.FromSeconds(5), 
          TimeSpan.FromSeconds(5));

        JobResponse result = await jobClient.ScheduleDeviceMethodAsync(jobId,
            $"DeviceId IN ['{deviceId}']",
            directMethod,
            DateTime.UtcNow,
            (long)TimeSpan.FromMinutes(2).TotalSeconds);

        Console.WriteLine("Started Method Job");
    }
    ```

1. Добавьте другой метод в класс **Program**.

    ```csharp
    public static async Task StartTwinUpdateJob(string jobId)
    {
        Twin twin = new Twin(deviceId);
        twin.Tags = new TwinCollection();
        twin.Tags["Building"] = "43";
        twin.Tags["Floor"] = "3";
        twin.ETag = "*";

        twin.Properties.Desired["LocationUpdate"] = DateTime.UtcNow;

        JobResponse createJobResponse = jobClient.ScheduleTwinUpdateAsync(
            jobId,
            $"DeviceId IN ['{deviceId}']", 
            twin, 
            DateTime.UtcNow, 
            (long)TimeSpan.FromMinutes(2).TotalSeconds).Result;

        Console.WriteLine("Started Twin Update Job");
    }
    ```

    > [!NOTE]
    > Дополнительные сведения о синтаксисе запросов см. в статье [Язык запросов Центра Интернета вещей для двойников устройств и двойников модулей, заданий и маршрутизации сообщений](./iot-hub-devguide-query-language.md).
    >

1. Наконец, добавьте следующие строки в метод **Main** :

    ```csharp
    Console.WriteLine("Press ENTER to start running jobs.");
    Console.ReadLine();

    jobClient = JobClient.CreateFromConnectionString(connString);

    string methodJobId = Guid.NewGuid().ToString();

    StartMethodJob(methodJobId);
    MonitorJob(methodJobId).Wait();
    Console.WriteLine("Press ENTER to run the next job.");
    Console.ReadLine();

    string twinUpdateJobId = Guid.NewGuid().ToString();

    StartTwinUpdateJob(twinUpdateJobId);
    MonitorJob(twinUpdateJobId).Wait();
    Console.WriteLine("Press ENTER to exit.");
    Console.ReadLine();
    ```

1. Сохраните результаты работы и создайте решение.

## <a name="run-the-apps"></a>Запуск приложений

Теперь все готово к запуску приложений.

1. В обозревателе решений Visual Studio щелкните правой кнопкой мыши решение и выберите пункт **Назначить запускаемые проекты**.

1. Выберите **Общие свойства** > **Запускаемый проект**, затем пункт **Несколько запускаемых проектов**.

1. Убедитесь, что `SimulateDeviceMethods` находится в верхней части списка, за которым следует `ScheduleJob`. Установите значение **Запуск** для обоих действий и щелкните **ОК**.

1. Запустите проекты, щелкнув **Запуск**, или перейдите к меню **Отладка** и щелкните **Начать отладку**.

   Отобразятся выходные данные с устройства и серверных приложений.

    ![Выполнение приложений для планирования заданий](./media/iot-hub-csharp-csharp-schedule-jobs/schedule-jobs-console-results.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике описано использование задания для планирования прямого метода на устройстве и обновления свойств двойника устройства.

* Чтобы продолжить знакомство с Центром Интернета вещей и шаблонами управления устройствами, такими как удаленное обновление встроенного ПО, см. [Руководство по реализации процесса обновления встроенного ПО](tutorial-firmware-update.md).

* Сведения о развертывании решений на базе искусственного интеллекта на пограничных устройствах с помощью Azure IoT Edge см. в руководстве [Развертывание первого модуля IoT Edge на устройстве под управлением 64-разрядной ОС Linux](../iot-edge/quickstart-linux.md).