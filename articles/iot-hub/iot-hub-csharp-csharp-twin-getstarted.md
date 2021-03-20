---
title: Начало работы с двойниками устройств Центра Интернета вещей Azure (.NET/.NET) | Документы Майкрософт
description: Добавление тегов и последующее использование запроса Центра Интернета вещей с помощью двойников устройств Центра Интернета вещей. Используйте пакет SDK для устройств Azure IoT для .NET, чтобы реализовать приложение имитации устройства, и пакет SDK для служб Azure IoT для .NET, чтобы реализовать приложение-службу, которое добавит теги и выполнит запрос к Центру Интернета вещей.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/26/2019
ms.author: robinsh
ms.custom: mqtt, devx-track-csharp
ms.openlocfilehash: 267a69486dc91ef95c0de704346eeb1d1780ef48
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89013764"
---
# <a name="get-started-with-device-twins-net"></a>Начало работы с двойниками устройств (.NET)

[!INCLUDE [iot-hub-selector-twin-get-started](../../includes/iot-hub-selector-twin-get-started.md)]

В этом руководстве создаются следующие консольные приложения для .NET:

* **AddTagsAndQuery**. Это серверное приложение добавляет теги и запрашивает двойники устройств.

* **ReportConnectivity**. Это приложение имитирует устройство, подключающееся к центру Интернета вещей с использованием удостоверения устройства, созданного ранее, и передает условия его подключения.

> [!NOTE]
> Статья [Общие сведения о пакетах SDK для Azure IoT и их использование](iot-hub-devguide-sdks.md) содержит сведения о разных пакетах SDK для Интернета вещей Azure, с помощью которых можно создать приложения для устройств и внутренние приложения.
>

## <a name="prerequisites"></a>Предварительные требования

* приведенному.

* Активная учетная запись Azure. Если ее нет, можно создать [бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/) всего за несколько минут.

* Убедитесь, что в брандмауэре открыт порт 8883. Пример устройства в этой статье использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Регистрация нового устройства в центре Интернета вещей

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="get-the-iot-hub-connection-string"></a>Получение строки подключения центра Интернета вещей

[!INCLUDE [iot-hub-howto-twin-shared-access-policy-text](../../includes/iot-hub-howto-twin-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-custom-connection-string](../../includes/iot-hub-include-find-custom-connection-string.md)]

## <a name="create-the-service-app"></a>Создание приложения службы

В этом разделе рассказывается о том, как создать консольное приложение .NET с помощью C#, которое добавляет метаданные расположения в двойник устройства, связанный с **myDeviceId**. После этого оно запрашивает данные двойников устройств, хранящихся в Центре Интернета вещей, выбирает устройства, находящиеся в США, а затем те, которые сообщили о подключении по сети мобильной связи.

1. В Visual Studio выберите **Создать проект**. В окне **Создание проекта** выберите **Консольное приложение (.NET Framework)** и нажмите кнопку **Далее**.

1. В окне **Настроить новый проект** присвойте проекту имя **AddTagsAndQuery**.

    ![Настройка проекта AddTagsAndQuery](./media/iot-hub-csharp-csharp-twin-getstarted/config-addtagsandquery-app.png)

1. В обозревателе решений щелкните правой кнопкой мыши проект **AddTagsAndQuery**, а затем **Управление пакетами Nuget**.

1. Выберите **Обзор**, найдите и выберите **Microsoft.Azure.Devices**. Выберите пункт **Установить**.

    ![Окно "Диспетчер пакетов NuGet"](./media/iot-hub-csharp-csharp-twin-getstarted/nuget-package-addtagsandquery-app.png)

   В результате выполняется скачивание и установка пакета NuGet [SDK для служб Azure IoT](https://www.nuget.org/packages/Microsoft.Azure.Devices/) и его зависимостей, а также добавляется соответствующая ссылка.

1. Добавьте следующие инструкции `using` в начало файла **Program.cs** :

    ```csharp  
    using Microsoft.Azure.Devices;
    ```

1. Добавьте следующие поля в класс **Program** . Замените `{iot hub connection string}` строкой подключения Центра Интернета вещей, скопированной в разделе [Получение строки подключения центра Интернета вещей](#get-the-iot-hub-connection-string).

    ```csharp  
    static RegistryManager registryManager;
    static string connectionString = "{iot hub connection string}";
    ```

1. Добавьте следующий метод в класс **Program** .

    ```csharp  
    public static async Task AddTagsAndQuery()
    {
        var twin = await registryManager.GetTwinAsync("myDeviceId");
        var patch =
            @"{
                tags: {
                    location: {
                        region: 'US',
                        plant: 'Redmond43'
                    }
                }
            }";
        await registryManager.UpdateTwinAsync(twin.DeviceId, patch, twin.ETag);

        var query = registryManager.CreateQuery(
          "SELECT * FROM devices WHERE tags.location.plant = 'Redmond43'", 100);
        var twinsInRedmond43 = await query.GetNextAsTwinAsync();
        Console.WriteLine("Devices in Redmond43: {0}", 
          string.Join(", ", twinsInRedmond43.Select(t => t.DeviceId)));

        query = registryManager.CreateQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43' AND properties.reported.connectivity.type = 'cellular'", 100);
        var twinsInRedmond43UsingCellular = await query.GetNextAsTwinAsync();
        Console.WriteLine("Devices in Redmond43 using cellular network: {0}", 
          string.Join(", ", twinsInRedmond43UsingCellular.Select(t => t.DeviceId)));
    }
    ```

    Класс **RegistryManager** предоставляет все методы, необходимые для взаимодействия с двойниками устройства из службы. Предыдущий код сначала инициализирует объект **RegistryManager**, затем извлекает двойник устройства для **MyDeviceId** и обновляет его теги, используя сведения о требуемом расположении.

    После обновления он выполняет два запроса: первый выбирает только двойники устройств, расположенные на фабрике **Redmond43**, а второй уточняет условия первого запроса и выбирает устройства, подключенные по сети мобильной связи.

    При создании объекта **query** предыдущий код указывает максимальное количество возвращаемых документов. Объект **Query** содержит логическое свойство **HasMoreResults**, которое можно использовать для вызова методов **GetNextAsTwinAsync** несколько раз, чтобы получить все результаты. Метод **GetNextAsJson** доступен для результатов, которые не являются двойниками устройств, например результатов статистических запросов.

1. Наконец, добавьте следующие строки в метод **Main** :

    ```csharp  
    registryManager = RegistryManager.CreateFromConnectionString(connectionString);
    AddTagsAndQuery().Wait();
    Console.WriteLine("Press Enter to exit.");
    Console.ReadLine();
    ```

1. Запустите это приложение, щелкнув правой кнопкой мыши проект **AddTagsAndQuery** и выбрав **Отладка**, а затем — **Запустить новый экземпляр**. В результатах запроса на все устройства, расположенные на фабрике **Redmond43**, отобразится одно устройство, а для запроса на ограничение результатов устройствами, использующими сеть мобильной связи, не отобразится ни одного устройства.

    ![Результаты запроса в окне](./media/iot-hub-csharp-csharp-twin-getstarted/addtagapp.png)

В следующем разделе рассказывается о том, как создать приложение устройства, которое сообщает сведения о подключении и изменяет результат запроса, описанного в предыдущем разделе.

## <a name="create-the-device-app"></a>Создание приложения устройства

В этом разделе вы создадите консольное приложение .NET, которое подключается к центру как **myDeviceId** и обновляет передаваемые свойства, добавив в них сведения о подключении по сети мобильной связи.

1. В Visual Studio выберите **Файл** > **Создать** > **Проект**. В окне **Создание проекта** выберите **Консольное приложение (.NET Framework)** и нажмите кнопку **Далее**.

1. В окне **Настроить новый проект** присвойте проекту имя **ReportConnectivity**. Для параметра **Решение** выберите **Добавить к решению** и нажмите **Создать**.

1. В обозревателе решений щелкните правой кнопкой мыши проект **ReportConnectivity**, а затем **Управление пакетами NuGet**

1. Выберите **Обзор**, найдите и выберите **Microsoft.Azure.Devices.Client**. Выберите пункт **Установить**.

   В результате выполняется скачивание и установка пакета NuGet [SDK для устройств Azure IoT](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/) и его зависимостей, а также добавляется соответствующая ссылка.

1. Добавьте следующие инструкции `using` в начало файла **Program.cs** :

    ```csharp  
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using Newtonsoft.Json;
    ```

1. Добавьте следующие поля в класс **Program** . Замените `{device connection string}` строкой подключения устройства, записанной в разделе [Регистрация нового устройства в центре Интернета вещей](#register-a-new-device-in-the-iot-hub).

    ```csharp  
    static string DeviceConnectionString = "HostName=<yourIotHubName>.azure-devices.net;DeviceId=<yourIotDeviceName>;SharedAccessKey=<yourIotDeviceAccessKey>";
    static DeviceClient Client = null;
    ```

1. Добавьте следующий метод в класс **Program** .

    ```csharp
    public static async void InitClient()
    {
        try
        {
            Console.WriteLine("Connecting to hub");
            Client = DeviceClient.CreateFromConnectionString(DeviceConnectionString, 
              TransportType.Mqtt);
            Console.WriteLine("Retrieving twin");
            await Client.GetTwinAsync();
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error in sample: {0}", ex.Message);
        }
    }
    ```

    Объект **Client** позволяет получить все методы, необходимые для взаимодействия с двойниками устройства с устройства. Приведенный выше код инициализирует объект **Client**, а затем получает двойник устройства для **myDeviceId**.

1. Добавьте следующий метод в класс **Program** .

    ```csharp  
    public static async void ReportConnectivity()
    {
        try
        {
            Console.WriteLine("Sending connectivity data as reported property");

            TwinCollection reportedProperties, connectivity;
            reportedProperties = new TwinCollection();
            connectivity = new TwinCollection();
            connectivity["type"] = "cellular";
            reportedProperties["connectivity"] = connectivity;
            await Client.UpdateReportedPropertiesAsync(reportedProperties);
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error in sample: {0}", ex.Message);
        }
    }
    ```

   Вышеприведенный код изменяет передаваемое свойство **myDeviceId** с помощью сведений о подключении.

1. Наконец, добавьте следующие строки в метод **Main** :

    ```csharp
    try
    {
        InitClient();
        ReportConnectivity();
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    Console.WriteLine("Press Enter to exit.");
    Console.ReadLine();
    ```

1. В обозревателе решений щелкните решение правой кнопкой мыши и выберите **Назначить запускаемые проекты**.

1. Выберите **Общие свойства** > **Запускаемый проект**, затем пункт **Несколько запускаемых проектов**. Для проекта **ReportConnectivity** выберите **Запустить** в качестве значения параметра **Действие**. Нажмите кнопку **ОК** , чтобы сохранить изменения.  

1. Запустите это приложение, щелкнув правой кнопкой мыши проект **ReportConnectivity** и выбрав **Отладка**, а затем — **Запустить новый экземпляр**. Вы увидите, что приложение получает сведения о двойника, а затем отправляет подключение как **_сообщаемое свойство_**.

    ![Запуск приложения устройства для передачи данных подключения](./media/iot-hub-csharp-csharp-twin-getstarted/rundeviceapp.png)

   После того как устройство сообщило сведения о подключении, оно должно появиться в обоих запросах.

1. Щелкните правой кнопкой мыши проект **AddTagsAndQuery** и выберите **Отладка** > **Запустить новый экземпляр**, чтобы снова запустить запросы. На этот раз **myDeviceId** должен появиться в результатах обоих запросов.

    ![Сведения о подключении устройства успешно переданы](./media/iot-hub-csharp-csharp-twin-getstarted/tagappsuccess.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве мы настроили новый Центр Интернета вещей на портале Azure и создали удостоверение устройства в реестре удостоверений Центра Интернета вещей. Вы добавили метаданные устройства в качестве тегов из внутреннего приложения и написали код приложения имитации устройства, чтобы сообщить сведения о подключении в двойнике устройства. Вы также узнали, как запрашивать эти сведения, используя похожий на SQL язык запросов Центра Интернета вещей.

Для получения дополнительных сведений ознакомьтесь со следующими ресурсами:

* Чтобы узнать, как отправлять данные телеметрии с устройств, см. руководство [Отправка данных телеметрии с устройства в центр Интернета вещей](quickstart-send-telemetry-dotnet.md).

* Чтобы узнать, как настроить устройства с помощью требуемых свойств двойника устройства, см. руководство [Настройка устройств с помощью желаемых свойств](tutorial-device-twins.md).

* Чтобы узнать, как управлять устройствами в интерактивном режиме, например включить вентилятор из управляемого пользователем приложения, см. руководство [Использование прямых методов](quickstart-control-device-dotnet.md).
