---
title: Начало работы с удостоверением модуля центра Интернета вещей Azure & модуля двойника (.NET)
description: Сведения о создании удостоверения модуля и обновлении двойника модуля с помощью пакетов SDK для Центра Интернета для .NET.
author: chrissie926
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/07/2019
ms.author: menchi
ms.custom: amqp, devx-track-csharp
ms.openlocfilehash: daba400b9daadf464c4c125ad266745237e71367
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92142518"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-net"></a>Приступая к работе с удостоверениями модулей центра Интернета вещей и модулем двойника (.NET)

[!INCLUDE [iot-hub-selector-module-twin-getstarted](../../includes/iot-hub-selector-module-twin-getstarted.md)]

> [!NOTE]
> [Удостоверение и двойник модуля](iot-hub-devguide-module-twins.md) аналогичны удостоверению и двойнику устройства Центра Интернета вещей Azure, однако они предоставляют более высокую степень детализации. Хотя удостоверение устройства центра Интернета вещей Azure и двойника устройства позволяют внутреннему приложению настраивать устройство и обеспечивать видимость условий устройства, удостоверение модуля и модуль двойника предоставляют эти возможности отдельным компонентам устройства. В поддерживающих устройствах с несколькими компонентами, такими как устройства на основе операционной системы или устройства встроенного по, удостоверения модулей и модуль двойников позволяют выполнять изолированную настройку и условия для каждого компонента.

По завершении работы с этим руководством у вас будет два консольных приложения .NET:

* **Креатеидентитиес**. Это приложение создает удостоверение устройства, идентификатор модуля и связанный ключ безопасности для подключения клиентов устройства и модуля.

* **Упдатемодулетвинрепортедпропертиес**. Это приложение отправляет обновленные свойства модуля двойника в центр Интернета вещей.

> [!NOTE]
> Сведения о пакетах SDK Azure IoT, которые можно использовать для создания приложений, которые будут работать на устройствах и в серверной части решения, см. в [этой статье](iot-hub-devguide-sdks.md).

## <a name="prerequisites"></a>Предварительные требования

* приведенному.

* Активная учетная запись Azure. Если ее нет, можно создать [бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/) всего за несколько минут.

## <a name="create-a-hub"></a>Создание концентратора.

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="get-the-iot-hub-connection-string"></a>Получение строки подключения центра Интернета вещей

[!INCLUDE [iot-hub-howto-module-twin-shared-access-policy-text](../../includes/iot-hub-howto-module-twin-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-registryrw-connection-string](../../includes/iot-hub-include-find-registryrw-connection-string.md)]

[!INCLUDE [iot-hub-get-started-create-module-identity-csharp](../../includes/iot-hub-get-started-create-module-identity-csharp.md)]

## <a name="update-the-module-twin-using-net-device-sdk"></a>Обновление двойника модуля с помощью пакета SDK для устройства .NET

В этом разделе вы создадите консольное приложение .NET на имитированном устройстве, которое обновляет сообщаемые свойства двойника модуля.

Перед началом работы получите строку подключения модуля. Войдите на [портал Azure](https://portal.azure.com/). Перейдите в центр и выберите **устройства IOT**. Найти **myFirstDevice**. Выберите **myFirstDevice** , чтобы открыть его, а затем выберите **мифирстмодуле** , чтобы открыть его. В разделе **сведения об удостоверении модуля** скопируйте **строку подключения (первичный ключ)** , если это необходимо в следующей процедуре.

   ![Сведения о модуле на портале Azure](./media/iot-hub-csharp-csharp-module-twin-getstarted/module-identity-detail.png)

1. В Visual Studio добавьте новый проект в решение, выбрав **файл**  >  **создать**  >  **проект**. В окне Создание нового проекта выберите **консольное приложение (платформа .NET Framework)** и нажмите кнопку **Далее**.

1. В качестве имени проекта укажите *UpdateModuleTwinReportedProperties*. Для **решения** выберите **Добавить в решение**. Убедитесь, что указана версия платформы .NET 4.6.1 или более поздняя.

    ![Создание проекта Visual Studio](./media/iot-hub-csharp-csharp-module-twin-getstarted/configure-update-twins-csharp1.png)

1. Чтобы создать проект, выберите **Создать**.

1. В Visual Studio откройте **инструменты**  >  **Диспетчер пакетов NuGet**  >  **Управление пакетами NuGet для решения**. Выберите вкладку **Обзор**.

1. Найдите и выберите **Microsoft. Azure. Devices. Client**, а затем нажмите кнопку **установить**.

    ![Снимок экрана, на котором показаны выбранные "Microsoft. Azure. Devices. Client" и кнопка "установить".](./media/iot-hub-csharp-csharp-module-twin-getstarted/install-client-sdk.png)

1. Добавьте следующие инструкции `using` в начало файла **Program.cs** :

    ```csharp
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using System.Threading.Tasks;
    using Newtonsoft.Json;
    ```

1. Добавьте следующие поля в класс **Program** . Замените значение заполнителя строкой подключения модуля.

    ```csharp
    private const string ModuleConnectionString = "<Your module connection string>";
    private static ModuleClient Client = null;
    static void ConnectionStatusChangeHandler(ConnectionStatus status, 
      ConnectionStatusChangeReason reason)
    {
        Console.WriteLine("Connection Status Changed to {0}; the reason is {1}", 
          status, reason);
    }
    ```

1. Добавьте метод **OnDesiredPropertyChanged** в класс **Program**:

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, 
      object userContext)
        {
            Console.WriteLine("desired property change:");
            Console.WriteLine(JsonConvert.SerializeObject(desiredProperties));
            Console.WriteLine("Sending current time as reported property");
            TwinCollection reportedProperties = new TwinCollection
            {
                ["DateTimeLastDesiredPropertyChangeReceived"] = DateTime.Now
            };

            await Client.UpdateReportedPropertiesAsync(reportedProperties).ConfigureAwait(false);
        }
    ```

1. Добавьте следующие строки в метод **Main** :

    ```csharp
    static void Main(string[] args)
    {
        Microsoft.Azure.Devices.Client.TransportType transport = 
          Microsoft.Azure.Devices.Client.TransportType.Amqp;

        try
        {
            Client = 
              ModuleClient.CreateFromConnectionString(ModuleConnectionString, transport);
            Client.SetConnectionStatusChangesHandler(ConnectionStatusChangeHandler);
            Client.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).Wait();

            Console.WriteLine("Retrieving twin");
            var twinTask = Client.GetTwinAsync();
            twinTask.Wait();
            var twin = twinTask.Result;
            Console.WriteLine(JsonConvert.SerializeObject(twin.Properties)); 

            Console.WriteLine("Sending app start time as reported property");
            TwinCollection reportedProperties = new TwinCollection();
            reportedProperties["DateTimeLastAppLaunch"] = DateTime.Now;

            Client.UpdateReportedPropertiesAsync(reportedProperties);
        }
        catch (AggregateException ex)
        {
            Console.WriteLine("Error in sample: {0}", ex);
        }

        Console.WriteLine("Waiting for Events.  Press enter to exit...");
        Console.ReadLine();
        Client.CloseAsync().Wait();
    }
    ```

    В этом примере кода показано, как извлечь двойник модуля и обновить сообщаемые свойства с помощью протокола AMQP. В общедоступной предварительной версии для операций двойника модуля поддерживается только протокол AMQP.

1. При необходимости можно добавить эти инструкции в метод **Main** для отправки события в центр Интернета вещей из модуля. Поместите эти строки под `try catch` блок.

    ```csharp
    Byte[] bytes = new Byte[2];
    bytes[0] = 0;
    bytes[1] = 1;
    var sendEventsTask = Client.SendEventAsync(new Message(bytes));
    sendEventsTask.Wait();
    Console.WriteLine("Event sent to IoT Hub.");
    ```

## <a name="run-the-apps"></a>Запуск приложений

Теперь вы можете запускать приложения.

1. В **Обозреватель решений** щелкните правой кнопкой мыши решение в Visual Studio и выберите команду **задать запускаемые проекты**.

1. В разделе **Общие свойства** выберите **запускаемый проект.**

1. Выберите **Несколько запускаемых проектов**, а затем выберите **Запуск** в качестве действия для приложений и нажмите **кнопку ОК** , чтобы принять изменения.

1. Нажмите клавишу **F5** , чтобы запустить приложения.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы продолжить знакомство с Центром Интернета вещей и изучить другие сценарии Интернета вещей, см. следующие ресурсы:

* [How to get started with device management (Node)](iot-hub-node-node-device-management-get-started.md) (Начало работы с управлением устройствами (Node))

* [Краткое руководство. Развертывание первого модуля IoT Edge на устройстве под управлением 64-разрядной ОС Linux](../iot-edge/quickstart-linux.md)