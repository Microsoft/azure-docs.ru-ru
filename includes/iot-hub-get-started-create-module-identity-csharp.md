---
title: включить файл
description: включить файл
services: iot-hub
author: chrissie926
ms.service: iot-hub
ms.topic: include
ms.date: 08/07/2019
ms.author: menchi
ms.custom: include file
ms.openlocfilehash: a5c1ddd085ae65b9920d73f50f993f4646785a69
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "68883733"
---
## <a name="create-a-module-identity"></a>Создание удостоверения модуля

В этом разделе вы создадите консольное приложение .NET, которое создает удостоверение устройства и удостоверение модуля в реестре удостоверений в центре. Устройство или модуль не может подключиться к концентратору, если он не содержит запись в реестре удостоверений. Дополнительные сведения см. в [разделе реестр удостоверений раздела Azure для разработчиков центра Интернета вещей](../articles/iot-hub/iot-hub-devguide-identity-registry.md).

Запущенное консольное приложение создает уникальные идентификаторы и ключи одновременно для устройства и для модуля. Устройство и модуль используют эти значения для идентификации при отправке сообщений с устройства в облако в центр Интернета вещей. В идентификаторах учитывается регистр символов.

1. Откройте Visual Studio и выберите элемент **Создать проект**.

1. В окне **Создание нового проекта** выберите **консольное приложение (платформа .NET Framework)**.

1. Нажмите кнопку **Далее** , чтобы открыть **настроить новый проект**. Присвойте проекту имя *CreateIdentities*, а решению — *IoTHubGetStarted*. Убедитесь, что указана версия платформы .NET 4.6.1 или более поздняя.

    ![Введите имя и платформу для решения Visual Studio](./media/iot-hub-get-started-create-module-identity-csharp/configure-createidentities-project.png)

1. В Visual Studio откройте **инструменты**  >  **Диспетчер пакетов NuGet**  >  **Управление пакетами NuGet для решения**. Выберите вкладку **Обзор**.

1. Выполните поиск **Microsoft. Azure. Devices**. Выберите его и нажмите кнопку **установить**.

    ![Установка пакета SDK службы .NET для центра Интернета вещей Azure Текущая версия](./media/iot-hub-get-started-create-module-identity-csharp/install-service-sdk.png)

1. Добавьте следующие инструкции `using` в начало файла **Program.cs** :

   ```csharp
   using Microsoft.Azure.Devices;
   using Microsoft.Azure.Devices.Common.Exceptions;
   ```

1. Добавьте следующие поля в класс **Program** . Замените значение заполнителя строкой подключения к Центру Интернета вещей, созданному в предыдущем разделе.

   ```csharp
   const string connectionString = "<replace_with_iothub_connection_string>";
   const string deviceID = "myFirstDevice";
   const string moduleID = "myFirstModule";
   ```

1. Добавьте следующий код в класс **Main**.

   ```csharp
   static void Main(string[] args)
   {
       AddDeviceAsync().Wait();
       AddModuleAsync().Wait();
   }
   ```

1. Добавьте следующие методы в класс **Program**.

    ```csharp
    private static async Task AddDeviceAsync()
    {
       RegistryManager registryManager = 
         RegistryManager.CreateFromConnectionString(connectionString);
       Device device;

       try
       {
           device = await registryManager.AddDeviceAsync(new Device(deviceID));
       }
       catch (DeviceAlreadyExistsException)
        {
            device = await registryManager.GetDeviceAsync(deviceID);
        }

        Console.WriteLine("Generated device key: {0}", 
          device.Authentication.SymmetricKey.PrimaryKey);
    }

    private static async Task AddModuleAsync()
    {
        RegistryManager registryManager = 
          RegistryManager.CreateFromConnectionString(connectionString);
        Module module;

        try
        {
            module = 
              await registryManager.AddModuleAsync(new Module(deviceID, moduleID));
        }
        catch (ModuleAlreadyExistsException)
        {
            module = await registryManager.GetModuleAsync(deviceID, moduleID);
        }

        Console.WriteLine("Generated module key: {0}", module.Authentication.SymmetricKey.PrimaryKey);
    }
    ```

    `AddDeviceAsync`Метод создает удостоверение устройства с идентификатором **myFirstDevice**. Если этот идентификатор устройства уже существует в реестре удостоверений, код просто извлекает сведения о существующем устройстве. Затем приложение отображает первичный ключ для данного удостоверения. Этот ключ используется в приложении имитации устройства для подключения к концентратору.

    `AddModuleAsync`Метод создает удостоверение модуля с идентификатором **мифирстмодуле** в разделе **myFirstDevice** устройства. Если этот идентификатор модуля уже существует в реестре удостоверений, код просто извлекает сведения о существующем модуле. Затем приложение отображает первичный ключ для данного удостоверения. Этот ключ используется в приложении имитации модуля для подключения к концентратору.

   [!INCLUDE [iot-hub-pii-note-naming-device](iot-hub-pii-note-naming-device.md)]

1. Запустите это приложение и запишите ключ устройства и ключ модуля.

> [!NOTE]
> В реестре удостоверений центра Интернета вещей хранятся идентификаторы устройств и модулей, чтобы обеспечить безопасный доступ к концентратору. Реестр удостоверений хранит идентификаторы устройств и ключи, используемые в качестве учетных данных безопасности. Реестр удостоверений также хранит флаги включения и отключения для каждого устройства. Эти флаги можно использовать для отключения доступа для этого устройства. Если приложение должно хранить другие метаданные для конкретных устройств, оно должно использовать хранилище конкретного приложения. У вас нет возможности включать и отключать удостоверения модулей. Дополнительные сведения см. в разделе [Руководство разработчика для Центра Интернета вещей](../articles/iot-hub/iot-hub-devguide-identity-registry.md).
