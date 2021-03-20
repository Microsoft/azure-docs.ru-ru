---
title: Удостоверение модуля центра Интернета вещей Azure & модуля двойника (портал и .NET)
description: Сведения о создании удостоверения модуля и обновлении двойника модуля с помощью портала и .NET.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/20/2019
ms.custom: amqp, devx-track-csharp
ms.openlocfilehash: 0d16d7a6dbf903ae790015c12e3d34e4a2a553d3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92139344"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-using-the-portal-and-net-device"></a>Начало работы с удостоверением модуля Центра Интернета вещей и двойником модуля с использованием портала и устройства .NET

[!INCLUDE [iot-hub-selector-module-twin-getstarted](../../includes/iot-hub-selector-module-twin-getstarted.md)]

> [!NOTE]
> [Удостоверение и двойник модуля](iot-hub-devguide-module-twins.md) аналогичны удостоверению и двойнику устройства Центра Интернета вещей Azure, однако они предоставляют более высокую степень детализации. Хотя удостоверение устройства центра Интернета вещей Azure и двойника устройства позволяют внутреннему приложению настраивать устройство и обеспечивать видимость условий устройства, удостоверение модуля и модуль двойника предоставляют эти возможности отдельным компонентам устройства. В поддерживающих устройствах с несколькими компонентами, такими как устройства на основе операционной системы или устройства встроенного по, удостоверения модулей и модуль двойников позволяют выполнять изолированную настройку и условия для каждого компонента.
>

Из этого руководства вы узнаете:

* как создать удостоверение модуля на портале;

* Как использовать пакет SDK для устройств .NET для обновления модуля двойника с вашего устройства.

> [!NOTE]
> Сведения о пакетах SDK для центра Интернета вещей Azure, которые можно использовать для создания приложений для запуска на устройствах и серверной части решения, см. в статье [пакеты SDK для IOT Azure](iot-hub-devguide-sdks.md).
>

## <a name="prerequisites"></a>Предварительные требования

* приведенному.

* Активная учетная запись Azure. Если ее нет, можно создать [бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/) всего за несколько минут.

## <a name="create-a-hub"></a>Создание концентратора.

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-hub"></a>Регистрация нового устройства в концентраторе

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="create-a-module-identity-in-the-portal"></a>Создание удостоверения модуля на портале

В рамках одного удостоверения устройства можно создать до 20 удостоверений модуля. Чтобы добавить удостоверение, выполните следующие действия.

1. Для устройства, созданного в предыдущем разделе, щелкните **Добавить удостоверение модуля** , чтобы создать первое удостоверение модуля.

1. Введите имя *мифирстмодуле*. Сохраните удостоверение модуля.

    ![Добавить удостоверение модуля](./media/iot-hub-portal-csharp-module-twin-getstarted/add-module-identity.png)

    Новое удостоверение модуля появится в нижней части экрана. Выберите его, чтобы просмотреть сведения об удостоверении модуля.

    ![См. сведения об удостоверении модуля](./media/iot-hub-portal-csharp-module-twin-getstarted/module-identity-details.png)

Сохраните **строку подключения — первичный ключ**. Его можно использовать в следующем разделе, чтобы настроить модуль на устройстве.

## <a name="update-the-module-twin-using-net-device-sdk"></a>Обновление двойника модуля с помощью пакета SDK для устройства .NET

Вы успешно создали удостоверение модуля в своем Центре Интернета вещей. Давайте попробуем обменяться данными с облаком со своего имитированного устройства. После создания удостоверения модуля двойник модуля будет неявно создан в Центре Интернета вещей. В этом разделе вы создадите консольное приложение .NET на имитированном устройстве, которое обновляет сообщаемые свойства двойника модуля.

### <a name="create-a-visual-studio-project"></a>Создание проекта Visual Studio

Чтобы создать приложение, которое обновляет сообщаемые свойства модуля двойника, выполните следующие действия.

1. В Visual Studio выберите **создать новый проект**, затем выберите **консольное приложение (платформа .NET Framework)** и нажмите кнопку **Далее**.

1. В окне **Настройка нового проекта** введите *упдатемодулетвинрепортедпропертиес* в качестве **имени проекта**. Щелкните **Создать** , чтобы продолжить.

    ![Настройка проекта Visual Studio](./media/iot-hub-portal-csharp-module-twin-getstarted/configure-twins-project.png)

### <a name="install-the-latest-azure-iot-hub-net-device-sdk"></a>Установка последнего пакета SDK для устройств Центра Интернета вещей Azure для .NET

Удостоверения модулей и двойники модулей предоставляются в общедоступной предварительной версии. Он доступен только в пакетах SDK для устройств предварительной версии центра Интернета вещей. Чтобы установить расширение, выполните следующие действия:

1. В Visual Studio откройте **инструменты**  >  **Диспетчер пакетов NuGet**  >  **Управление пакетами NuGet для решения**.

1. Нажмите кнопку **Обзор**, а затем выберите **включить предварительные выпуски**. Выполните поиск *Microsoft. Azure. Devices. Client*. Выберите последнюю версию и установите ее.

    ![Установка пакета SDK для службы .NET центра Интернета вещей Azure (Предварительная версия)](./media/iot-hub-csharp-csharp-module-twin-getstarted/install-sdk.png)

    Теперь у вас есть доступ ко всем возможностям этого модуля.

### <a name="get-your-module-connection-string"></a>Получение строки подключения модуля

Вам потребуется строка подключения модуля для консольного приложения. Выполните указанные ниже действия:

1. Войдите на [портал Microsoft Azure](https://portal.azure.com/).

1. Перейдите в центр Интернета вещей и выберите **устройства IOT**. Откройте **myFirstDevice** , и вы увидите, что **мифирстмодуле** был успешно создан.

1. В разделе **удостоверения модулей** выберите **мифирстмодуле** . В разделе **сведения об удостоверении модуля** скопируйте **строку подключения (первичный ключ)**.

    ![Сведения о модуле на портале Azure](./media/iot-hub-portal-csharp-module-twin-getstarted/module-identity-details.png)

### <a name="create-updatemoduletwinreportedproperties-console-app"></a>Создание консольного приложения UpdateModuleTwinReportedProperties

Чтобы создать приложение, выполните следующие действия.

1. Добавьте следующие инструкции `using` в начало файла **Program.cs** :

  ```csharp
  using Microsoft.Azure.Devices.Client;
  using Microsoft.Azure.Devices.Shared;
  using Newtonsoft.Json;
  ```

2. Добавьте следующие поля в класс **Program** . Замените значение заполнителя строкой подключения модуля.

  ```csharp
  private const string ModuleConnectionString = "<Your module connection string>";
  private static ModuleClient Client = null;
  ```

3. Добавьте метод **OnDesiredPropertyChanged** в класс **Program**:

  ```csharp
  private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
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

4. Наконец, замените метод **Main** следующим кодом:

  ```csharp
  static void Main(string[] args)
  {
      Microsoft.Azure.Devices.Client.TransportType transport = Microsoft.Azure.Devices.Client.TransportType.Amqp;
  
      try
      {
          Client = ModuleClient.CreateFromConnectionString(ModuleConnectionString, transport);
          Client.SetConnectionStatusChangesHandler(ConnectionStatusChangeHandler);
          Client.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).Wait();
  
          Console.WriteLine("Retrieving twin");
          var twinTask = Client.GetTwinAsync();
          twinTask.Wait();
          var twin = twinTask.Result;
          Console.WriteLine(JsonConvert.SerializeObject(twin));
  
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
      Console.ReadKey();
      Client.CloseAsync().Wait();
  }
  
  private static void ConnectionStatusChangeHandler(ConnectionStatus status, ConnectionStatusChangeReason reason)
  {
      Console.WriteLine($"Status {status} changed: {reason}");
  }
  ```
  
  Вы можете создать и запустить это приложение с помощью **клавиши F5**.

В этом примере кода показано, как извлечь двойник модуля и обновить сообщаемые свойства с помощью протокола AMQP. В общедоступной предварительной версии для операций двойника модуля поддерживается только протокол AMQP.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы продолжить знакомство с Центром Интернета вещей и изучить другие сценарии Интернета вещей, см. следующие ресурсы:

* [Приступая к работе с удостоверением Центра Интернета вещей и двойником модуля с использованием резервной копии и устройства .NET](iot-hub-csharp-csharp-module-twin-getstarted.md)

* [Краткое руководство. Развертывание первого модуля IoT Edge на устройстве под управлением 64-разрядной ОС Linux](../iot-edge/quickstart-linux.md)