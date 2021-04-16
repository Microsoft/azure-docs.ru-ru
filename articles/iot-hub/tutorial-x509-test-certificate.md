---
title: Учебник. Тестирование аутентификации на основе сертификата X.509 для устройств в Центре Интернета вещей Azure | Документация Майкрософт
description: Учебник. Тестирование аутентификации на основе сертификата X.509 в Центре Интернета вещей Azure
author: v-gpettibone
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 02/26/2021
ms.author: robinsh
ms.custom:
- mvc
- 'Role: Cloud Development'
- 'Role: Data Analytics'
ms.openlocfilehash: 7d1900782fce6b84ed79014e985393f3626d171b
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107379440"
---
# <a name="tutorial-testing-certificate-authentication"></a>Учебник. Тестирование аутентификации на основе сертификата

Используйте приведенный ниже пример кода на C#, чтобы проверить, что ваш сертификат позволяет устройству пройти аутентификацию в Центре Интернета вещей. Прежде чем выполнить пример кода, обязательно сделайте следующее:

* Создайте сертификат корневого ЦС или подчиненного ЦС.
* Отправьте сертификат ЦС в Центр Интернета вещей.
* Подтвердите, что вы владеете сертификатом корневого ЦС.
* Добавьте устройство в Центр Интернета вещей.
* Создайте сертификат устройства, указав код своего устройства.

## <a name="code-example"></a>Пример кода

В приведенном ниже примере кода показано, как создать приложение C# для имитации устройства X.509, зарегистрированного в вашем центре Интернета вещей. Пример позволяет отправлять значения температуры и влажности с имитированного устройства в ваш центр. В этом учебнике показано только, как создать приложение устройства. Создание приложения службы Центра Интернета вещей, которое будет отправлять ответы на события, отправленные этим виртуальным устройством, останется в качестве упражнения для читателей.

1. Откройте Visual Studio и выберите **Создать проект**, а затем выберите шаблон проекта **Консольное приложение (.NET Framework)** . Выберите **Далее**.

1. В окне **Настроить новый проект** введите имя проекта *SimulateX509Device* и нажмите кнопку **Создать**.

   ![Создание проекта устройства X.509 в Visual Studio](./media/iot-hub-security-x509-get-started/create-device-project-vs2019.png)

1. В Обозревателе решений щелкните правой кнопкой мыши проект **SimulateX509Device** и выберите пункт **Управление пакетами NuGet**.

1. В окне **Диспетчер пакетов NuGet** выберите **Обзор**, а затем найдите и выберите **Microsoft.Azure.Devices.Client**. Выберите пункт **Установить**.

   ![Добавление пакета NuGet SDK устройства в Visual Studio](./media/iot-hub-security-x509-get-started/device-sdk-nuget.png)

    В результате выполняется скачивание и установка пакета NuGet SDK для устройств Azure IoT и его зависимостей, а также добавляется соответствующая ссылка.

    Введите и выполните следующий код:

```csharp
using System;
using Microsoft.Azure.Devices.Client;
using System.Security.Cryptography.X509Certificates;
using System.Threading.Tasks;
using System.Text;

namespace SimulateX509Device
{
    class Program
    {
        private static int MESSAGE_COUNT = 5;

        // Temperature and humidity variables.
        private const int TEMPERATURE_THRESHOLD = 30;
        private static float temperature;
        private static float humidity;
        private static Random rnd = new Random();

        // Set the device ID to the name (device identifier) of your device.
        private static String deviceId = "{your-device-id}";

        static async Task SendEvent(DeviceClient deviceClient)
        {
            string dataBuffer;
            Console.WriteLine("Device sending {0} messages to IoTHub...\n", MESSAGE_COUNT);

            // Iterate MESSAGE_COUNT times to set randomm termperature and humidity values.
            for (int count = 0; count < MESSAGE_COUNT; count++)
            {
                // Set random values for temperature and humidity.
                temperature = rnd.Next(20, 35);
                humidity = rnd.Next(60, 80);
                dataBuffer = string.Format("{{\"deviceId\":\"{0}\",\"messageId\":{1},\"temperature\":{2},\"humidity\":{3}}}", deviceId, count, temperature, humidity);
                Message eventMessage = new Message(Encoding.UTF8.GetBytes(dataBuffer));
                eventMessage.Properties.Add("temperatureAlert", (temperature > TEMPERATURE_THRESHOLD) ? "true" : "false");
                Console.WriteLine("\t{0}> Sending message: {1}, Data: [{2}]", DateTime.Now.ToLocalTime(), count, dataBuffer);

                // Send to IoT Hub.
                await deviceClient.SendEventAsync(eventMessage);
            }
        }
        static void Main(string[] args)
        {
            try
            {
                // Create an X.509 certificate object.
                var cert = new X509Certificate2(@"{full path to pfx certificate.pfx}", "{your certificate password}");

                // Create an authentication object using your X.509 certificate. 
                var auth = new DeviceAuthenticationWithX509Certificate("{your-device-id}", cert);

                // Create the device client.
                var deviceClient = DeviceClient.Create("{your-IoT-Hub-name}.azure-devices.net", auth, TransportType.Mqtt);

                if (deviceClient == null)
                {
                    Console.WriteLine("Failed to create DeviceClient!");
                }
                else
                {
                    Console.WriteLine("Successfully created DeviceClient!");
                    SendEvent(deviceClient).Wait();
                }

                Console.WriteLine("Exiting...\n");
            }
            catch (Exception ex)
            {
                Console.WriteLine("Error in sample: {0}", ex.Message);
            }
         }
    }
}
```