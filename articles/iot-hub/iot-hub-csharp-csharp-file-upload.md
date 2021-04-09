---
title: Отправка файлов с устройств в Центр Интернета вещей Azure | Документация Майкрософт
description: Сведения об отправке файлов с устройства в облако с помощью пакета SDK для устройств Azure IoT для .NET. Отправленные файлы хранятся в контейнере больших двоичных объектов службы хранилища Azure.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 07/04/2017
ms.author: robinsh
ms.custom: mqtt, devx-track-csharp
ms.openlocfilehash: 43cafb8c5efe0581fe7c4136aa41980b3d817be2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99981414"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub-net"></a>Передача файлов с устройства в облако с помощью Центра Интернета вещей (.NET)

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

В этом учебнике используется код учебника [Отправка сообщений из облака на устройства с помощью Центра Интернета вещей](iot-hub-csharp-csharp-c2d.md), чтобы показать, как использовать возможности передачи файлов Центра Интернета вещей. В нем показано следующее:

* как безопасно предоставить устройству универсальный код ресурса (URI) большого двоичного объекта Azure для передачи файла;

* как использовать уведомления о передаче файлов Центра Интернета вещей для активации обработки файла в серверной части приложения.

В кратком руководстве [Отправка данных телеметрии с устройства в центр Интернета вещей](quickstart-send-telemetry-dotnet.md) и руководстве [Отправка сообщений из облака на устройство с помощью Центра Интернета вещей](iot-hub-csharp-csharp-c2d.md) описаны базовые функции Центра Интернета вещей для обмена сообщениями между устройством и облаком. В руководстве [Настройка маршрутизации сообщений с Центром Интернета вещей](tutorial-routing.md) описан способ надежного хранения сообщений, отправляемых с устройства в облако, в хранилище BLOB-объектов Microsoft Azure. Тем не менее в некоторых случаях непросто сопоставить данные, отправляемые устройством, с относительно небольшими сообщениями, отправляемыми из устройства в облако, которые принимает Центр Интернета вещей. Пример:

* Большие файлы, содержащие изображения.

* Видео

* Данные вибрации с высокой частотой выборки.

* Некоторые виды предварительно обработанных данных.

Эти файлы обычно обрабатываются в виде пакета в облаке с помощью таких инструментов, как [фабрика данных Azure](../data-factory/introduction.md) или стек [Hadoop](../hdinsight/index.yml). При передаче файлов с устройства вы можете рассчитывать на безопасность и надежность Центра Интернета вещей.

В конце этого руководства запускаются два консольных приложения для .NET:

* **SimulatedDevice**. Это приложение передает файл в хранилище с помощью универсального кода ресурса (URI) SAS, предоставленного вашим Центром Интернета вещей. Это измененная версия приложения, созданного в руководстве [Отправка сообщений из облака на устройство с помощью Центра Интернета вещей](iot-hub-csharp-csharp-c2d.md).

* **ReadFileUploadNotification**. Это приложение, которое получает уведомления о передаче файлов от Центра Интернета вещей.

> [!NOTE]
> Существуют пакеты SDK для устройств Azure IoT, обеспечивающие поддержку многих платформ устройств и языков (включая C, Java, Python и JavaScript) в Центре Интернета вещей. Пошаговые инструкции по подключению устройства к Центру Интернета вещей Azure см. в [Центре разработчика для Центра Интернета вещей Azure](https://azure.microsoft.com/develop/iot).

[!INCLUDE [iot-hub-include-x509-ca-signed-file-upload-support-note](../../includes/iot-hub-include-x509-ca-signed-file-upload-support-note.md)]

## <a name="prerequisites"></a>Предварительные требования

* Visual Studio

* Активная учетная запись Azure. Если ее нет, можно создать [бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/) всего за несколько минут.

* Убедитесь, что в брандмауэре открыт порт 8883. Пример устройства в этой статье использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [iot-hub-associate-storage](../../includes/iot-hub-associate-storage.md)]

## <a name="upload-a-file-from-a-device-app"></a>Передача файла из приложения устройства

В этом разделе вы измените приложение устройства, созданное в руководстве [Отправка сообщений из облака на устройство с помощью Центра Интернета вещей](iot-hub-csharp-csharp-c2d.md), чтобы с помощью центра Интернета вещей передавать сообщения из облака на устройство.

1. В обозревателе решений Visual Studio щелкните правой кнопкой мыши проект **SimulatedDevice** и выберите **Добавить** > **Существующий элемент**. Найдите файл образа и добавьте его в проект. В этом учебнике используется образ `image.jpg`.

1. Щелкните правой кнопкой мыши образ и выберите **Свойства**. Убедитесь, что параметр **Копировать в выходной каталог** имеет значение **Всегда копировать**.

    ![Покажите, где можно обновить свойство изображения для параметра Copy to Output Directory (Копирование в выходной каталог)](./media/iot-hub-csharp-csharp-file-upload/image-properties.png)

1. Добавьте в начало файла **Program.cs** следующие инструкции:

    ```csharp
    using System.IO;
    ```

1. Добавьте следующий метод в класс **Program** .

    ```csharp
    private static async Task SendToBlobAsync(string fileName)
    {
        Console.WriteLine("Uploading file: {0}", fileName);
        var watch = System.Diagnostics.Stopwatch.StartNew();

        await deviceClient.GetFileUploadSasUriAsync(new FileUploadSasUriRequest { BlobName = fileName });
        var blob = new CloudBlockBlob(sas.GetBlobUri());
        await blob.UploadFromFileAsync(fileName);
        await deviceClient.CompleteFileUploadAsync(new FileUploadCompletionNotification { CorrelationId = sas.CorrelationId, IsSuccess = true });

        watch.Stop();
        Console.WriteLine("Time to upload file: {0}ms\n", watch.ElapsedMilliseconds);
    }
    ```

    Метод `UploadToBlobAsync` принимает имя передаваемого файла и источник его передачи и обрабатывает передачу в хранилище. Консольное приложение отображает время, необходимое для отправки файла.

1. Добавьте в метод **Main** непосредственно перед строкой `Console.ReadLine()` следующую строку:

    ```csharp
    await SendToBlobAsync("image.jpg");
    ```

> [!NOTE]
> Для упрощения в этом руководстве не реализуются какие-либо политики повтора. В рабочем коде следует реализовать политики повторных попыток (например, с экспоненциальной задержкой), как указано в статье [Обработка временных сбоев](/azure/architecture/best-practices/transient-faults).

## <a name="get-the-iot-hub-connection-string"></a>Получение строки подключения центра Интернета вещей

В этой статье вы создадите серверную службу для получения сообщений с уведомлением о передаче файлов из центра Интернета вещей, созданного в кратком руководстве [Отправка данных телеметрии с устройства в центр Интернета вещей](quickstart-send-telemetry-dotnet.md). Для получения сообщений об отправке файлов службе требуется разрешение **service connect**. По умолчанию каждый Центр Интернета вещей создается с помощью политики общего доступа, называемой **службой**, которая предоставляет это разрешение.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="receive-a-file-upload-notification"></a>Получение уведомления о передачи файла

В этом разделе вам предстоит создать консольное приложение для .NET, которое получает уведомления об отправке файлов из Центра Интернета вещей.

1. В текущем решении Visual Studio выберите **Файл** > **Создать** > **Проект**. В окне **Создание проекта** выберите **Консольное приложение (.NET Framework)** и нажмите кнопку **Далее**.

1. Назовите проект *ReadFileUploadNotification*. В разделе **Решение** выберите **Добавить в решение**. Выберите **Создать**, чтобы создать проект.

    ![Настройка проекта ReadFileUploadNotification в Visual Studio](./media/iot-hub-csharp-csharp-file-upload/read-file-upload-project-configure.png)

1. В обозревателе решений щелкните правой кнопкой мыши проект **ReadFileUploadNotification** и выберите пункт **Управление пакетами NuGet**.

1. В разделе **Диспетчер пакетов NuGet** выберите **Обзор**. Найдите и выберите **Microsoft.Azure.Devices** и нажмите **Установить**.

    После этого будут выполнены скачивание, установка [пакета NuGet пакета SDK для служб Azure IoT](https://www.nuget.org/packages/Microsoft.Azure.Devices/) и добавление ссылки на него в проект **ReadFileUploadNotification**.

1. Добавьте в начало файла **Program.cs** этого проекта следующие инструкции:

    ```csharp
    using Microsoft.Azure.Devices;
    ```

1. Добавьте следующие поля в класс **Program** . Замените заполнитель `{iot hub connection string}` строкой подключения Центра Интернета вещей, скопированной в разделе [Получение строки подключения центра Интернета вещей](#get-the-iot-hub-connection-string):

    ```csharp
    static ServiceClient serviceClient;
    static string connectionString = "{iot hub connection string}";
    ```

1. Добавьте следующий метод в класс **Program** .

    ```csharp
    private async static void ReceiveFileUploadNotificationAsync()
    {
        var notificationReceiver = serviceClient.GetFileNotificationReceiver();

        Console.WriteLine("\nReceiving file upload notification from service");
        while (true)
        {
            var fileUploadNotification = await notificationReceiver.ReceiveAsync();
            if (fileUploadNotification == null) continue;

            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("Received file upload notification: {0}", 
              string.Join(", ", fileUploadNotification.BlobName));
            Console.ResetColor();

            await notificationReceiver.CompleteAsync(fileUploadNotification);
        }
    }
    ```

    Обратите внимание, что шаблон получения здесь аналогичен шаблону, использовавшемуся для получения сообщений из облака на устройство из приложения устройства.

1. Наконец, добавьте следующие строки в метод **Main** :

    ```csharp
    Console.WriteLine("Receive file upload notifications\n");
    serviceClient = ServiceClient.CreateFromConnectionString(connectionString);
    ReceiveFileUploadNotificationAsync();
    Console.WriteLine("Press Enter to exit\n");
    Console.ReadLine();
    ```

## <a name="run-the-applications"></a>Запуск приложений

Теперь все готово к запуску приложений.

1. В обозревателе решений щелкните решение правой кнопкой мыши и выберите **Назначить запускаемые проекты**.

1. В разделе **Общие свойства** > **Запускаемый проект** выберите **Несколько запускаемых проектов**, а затем выберите действие **Запуск** для обоих приложений — **ReadFileUploadNotification** и **SimulatedDevice**. Нажмите кнопку **ОК** , чтобы сохранить изменения.

1. Нажмите клавишу **F5**. Должны запуститься оба приложения. Вы должны увидеть, как завершится передача в одном консольном приложении и как другое консольное приложение получит уведомление о передаче. Наличие отправленного файла в учетной записи хранения Azure можно проверить с помощью [портала Azure](https://portal.azure.com/) или обозревателя серверов Visual Studio.

    ![Снимок экрана, показывающий экран выходных данных](./media/iot-hub-csharp-csharp-file-upload/run-apps1.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство показано, как использовать возможности передачи файлов Центра Интернета вещей, чтобы упростить передачу файлов из устройств. Изучение функций и сценариев Центра Интернета вещей можно продолжить в следующих руководствах:

* [Создание Центра Интернета вещей с помощью шаблона Azure Resource Manager (PowerShell)](iot-hub-rm-template-powershell.md)

* [Пакет SDK для устройств Azure IoT для C](iot-hub-device-sdk-c-intro.md)

* [Пакеты SDK для Центра Интернета вещей Azure](iot-hub-devguide-sdks.md)

Для дальнейшего изучения возможностей Центра Интернета вещей см. следующие статьи:

* [Краткое руководство. Развертывание первого модуля IoT Edge на устройстве под управлением 64-разрядной ОС Linux](../iot-edge/quickstart-linux.md)