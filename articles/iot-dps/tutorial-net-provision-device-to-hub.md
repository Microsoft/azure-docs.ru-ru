---
title: Учебник. Подготовка устройства с помощью службы подготовки устройств к добавлению в Центр Интернета вещей (.NET)
description: В этом учебнике показано, как подготовить устройство в одном Центре Интернета вещей с помощью Службы подготовки устройств к добавлению в Центр Интернета вещей с использованием .NET.
author: wesmc7777
ms.author: wesmc
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.devlang: csharp
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: f9a14ee6ee3e10b36d64ec11fc23807efe2bfaf2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94966570"
---
# <a name="tutorial-enroll-the-device-to-an-iot-hub-using-the-azure-iot-hub-provisioning-service-client-net"></a>Руководство. Регистрация устройства в центре Интернета вещей с помощью клиента Службы подготовки устройств к добавлению в Центр Интернета вещей Azure (.NET)

В предыдущем руководстве вы узнали, как настраивать устройство для подключения к службе подготовки устройств. В этом руководстве описывается, как использовать эту службу для подготовки устройства в одном Центре Интернета вещей с помощью как **_индивидуальной регистрации_**, так и **_групп регистраций_**. В этом учебнике описаны следующие процедуры.

> [!div class="checklist"]
> * Регистрация устройства
> * запуск устройства;
> * проверка регистрации устройства.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем продолжить, настройте устройство и его *аппаратный модуль безопасности*, как описано в руководстве [Set up a device to provision using the Azure IoT Hub Device Provisioning Service](./tutorial-set-up-device.md) (Настройка устройства для подготовки с помощью службы подготовки устройств для Центра Интернета вещей Azure).

* Visual Studio

> [!NOTE]
> Visual Studio не требуется. Достаточно установить [.NET](https://www.microsoft.com/net), чтобы разработчики могли использовать предпочтительный редактор в ОС Windows или Linux.  

В этом руководстве моделируется добавление информации об устройстве в службу подготовки во время производства оборудования или сразу после этого. Обычно этот код выполняется на компьютере или на заводском устройстве, на котором можно запустить код .NET, который не добавляется на само устройство.


## <a name="enroll-the-device"></a>Регистрация устройства

Этот шаг подразумевает добавление уникальных артефактов безопасности устройства в службу подготовки устройств. Эти артефакты безопасности выглядят так:

- Для устройств на основе доверенного платформенного модуля (TPM):
    - *Ключ подтверждения*, который уникален для каждой микросхемы или имитации доверенного платформенного модуля. Дополнительные сведения см. в статье [Общие сведения о ключе подтверждения доверенного платформенного модуля](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc770443(v=ws.11)).
    - *Registration ID* (ИД регистрации), который позволяет уникально идентифицировать устройство в пространстве имен и (или) области. Он может как совпадать, так и не совпадать с идентификатором устройства. Идентификатор — обязателен для каждого устройства. Для устройств на основе доверенного платформенного модуля идентификатор регистрации может быть производным от самого TPM, например хэш SHA-256 ключа подтверждения доверенного платформенного модуля.

- Для устройств на основе X.509:
    - [Сертификат X.509 выдается устройству](/windows/win32/seccertenroll/about-x-509-public-key-certificates) в файле формата *PEM* или *CER*. Для отдельной регистрации необходимо использовать *конечный сертификат* для системы X.509, а для групп регистраций — *корневой сертификат* либо его эквивалент, *сертификат подписчика*.
    - *Registration ID* (ИД регистрации), который позволяет уникально идентифицировать устройство в пространстве имен и (или) области. Он может как совпадать, так и не совпадать с идентификатором устройства. Идентификатор — обязателен для каждого устройства. Для устройств на базе X.509 идентификатор регистрации является производным от общего имени сертификата (CN). Дополнительные сведения об этих требованиях см. в статье [Понятия устройства в контексте подготовки устройств в Центре Интернета вещей](./concepts-service.md).

Имеется два способа регистрации устройства в службе подготовки устройств:

- **Individual Enrollments** (Отдельные регистрации). Представляет собой запись для одного устройства, которое можно зарегистрировать с помощью службы подготовки устройств. В качестве механизмов аттестации отдельные регистрации могут использовать сертификаты X.509 или токены SAS (в реальном или виртуальном доверенном платформенном модуле). Советуем использовать отдельные регистрации для устройств, которым требуются уникальные начальные конфигурации, или для устройств, которые могут использовать только маркеры SAS через TPM в качестве механизма аттестации. Для отдельных регистраций можно указать нужный идентификатор устройства Центра Интернета вещей.

- **Enrollment Groups** (Группы регистраций). Представляет группу устройств, использующих определенный механизм аттестации. Мы советуем использовать группу регистраций для большого количества устройств, имеющих необходимую начальную конфигурацию, или для устройств, предназначенных для одного и того же клиента. Группы регистраций создаются только на базе сертификатов X.509 и используют сертификат для подписи в цепочке сертификатов X.509.

### <a name="enroll-the-device-using-individual-enrollments"></a>Индивидуальная регистрация устройства

1. С помощью шаблона проекта **Консольное приложение** создайте проект "Консольное приложение Visual C#" в Visual Studio. Назовите проект **DeviceProvisioning**.
    
1. В обозревателе решений щелкните правой кнопкой мыши проект **DeviceProvisioning** и выберите пункт **Manage NuGet Packages…** (Управление пакетами NuGet...).

1. В окне **Диспетчер пакетов NuGet** выберите **Обзор** и найдите **microsoft.azure.devices.client**. Примите условия использования, затем выберите запись и щелкните **Установить**, чтобы установить пакет **Microsoft.Azure.Devices.Provisioning.Service**. В результате выполняется скачивание и установка пакета [SDK NuGet для службы подготовки устройств Интернета вещей Azure](https://www.nuget.org/packages/Microsoft.Azure.Devices.Provisioning.Service/) и его зависимостей, а также добавляется соответствующая ссылка.

1. Добавьте следующие инструкции `using` в начало файла **Program.cs** :
   
    ```csharp
    using Microsoft.Azure.Devices.Provisioning.Service;
    ```

1. Добавьте следующие поля в класс **Program** . Замените значение заполнителя строкой подключения службы подготовки устройств, записанной в предыдущем разделе.
   
    ```csharp
    static readonly string ServiceConnectionString = "{Device Provisioning Service connection string}";

    private const string SampleRegistrationId = "sample-individual-csharp";
    private const string SampleTpmEndorsementKey =
            "AToAAQALAAMAsgAgg3GXZ0SEs/gakMyNRqXXJP1S124GUgtk8qHaGzMUaaoABgCAAEMAEAgAAAAAAAEAxsj2gUS" +
            "cTk1UjuioeTlfGYZrrimExB+bScH75adUMRIi2UOMxG1kw4y+9RW/IVoMl4e620VxZad0ARX2gUqVjYO7KPVt3d" +
            "yKhZS3dkcvfBisBhP1XH9B33VqHG9SHnbnQXdBUaCgKAfxome8UmBKfe+naTsE5fkvjb/do3/dD6l4sGBwFCnKR" +
            "dln4XpM03zLpoHFao8zOwt8l/uP3qUIxmCYv9A7m69Ms+5/pCkTu/rK4mRDsfhZ0QLfbzVI6zQFOKF/rwsfBtFe" +
            "WlWtcuJMKlXdD8TXWElTzgh7JS4qhFzreL0c1mI0GCj+Aws0usZh7dLIVPnlgZcBhgy1SSDQMQ==";
    private const string OptionalDeviceId = "myCSharpDevice";
    private const ProvisioningStatus OptionalProvisioningStatus = ProvisioningStatus.Enabled;
    ```

1. Добавьте следующий код, чтобы реализовать регистрацию для устройства:

    ```csharp
    static async Task SetRegistrationDataAsync()
    {
        Console.WriteLine("Starting SetRegistrationData");

        Attestation attestation = new TpmAttestation(SampleTpmEndorsementKey);

        IndividualEnrollment individualEnrollment = new IndividualEnrollment(SampleRegistrationId, attestation);

        individualEnrollment.DeviceId = OptionalDeviceId;
        individualEnrollment.ProvisioningStatus = OptionalProvisioningStatus;

        Console.WriteLine("\nAdding new individualEnrollment...");
        var serviceClient = ProvisioningServiceClient.CreateFromConnectionString(ServiceConnectionString);

        IndividualEnrollment individualEnrollmentResult =
            await serviceClient.CreateOrUpdateIndividualEnrollmentAsync(individualEnrollment).ConfigureAwait(false);

        Console.WriteLine("\nIndividualEnrollment created with success.");
        Console.WriteLine(individualEnrollmentResult);
    }
    ```

1. И наконец, добавьте этот код в метод **Main**, чтобы открыть подключение к Центру Интернета вещей и начать регистрацию.
   
    ```csharp
    try
    {
        Console.WriteLine("IoT Device Provisioning example");

        SetRegistrationDataAsync().GetAwaiter().GetResult();
            
        Console.WriteLine("Done, hit enter to exit.");
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    Console.ReadLine();
    ```
        
1. В обозревателе решений Visual Studio щелкните правой кнопкой мыши решение и выберите пункт **Назначить запускаемые проекты**. Выберите пункт **Один запускаемый проект**, а затем в раскрывающемся меню выберите проект **DeviceProvisioning**.  

1. Запустите приложение для устройства .NET **DeviceProvisiong**. Оно должно настроить подготовку устройства. 

    ![Выполнение отдельной регистрации](./media/tutorial-net-provision-device-to-hub/individual.png)

При успешной регистрации устройства вы увидите на портале следующее:

   ![Успешная регистрация на портале](./media/tutorial-net-provision-device-to-hub/individual-portal.png)

### <a name="enroll-the-device-using-enrollment-groups"></a>Регистрация устройства с помощью группы регистраций

> [!NOTE]
> Для примера группы регистрации требуется сертификат X.509.

1. В обозревателе решений Visual Studio откройте проект **DeviceProvisioning**, созданный выше. 

1. Добавьте следующие инструкции `using` в начало файла **Program.cs** :
    
    ```csharp
    using System.Security.Cryptography.X509Certificates;
    ```

1. Добавьте следующие поля в класс **Program** . Замените значение заполнителя на расположение сертификата X509.
   
    ```csharp
    private const string X509RootCertPathVar = "{X509 Certificate Location}";
    private const string SampleEnrollmentGroupId = "sample-group-csharp";
    ```

1. Добавьте следующий код в файл **Program.cs**, чтобы реализовать групповую регистрацию:

    ```csharp
    public static async Task SetGroupRegistrationDataAsync()
    {
        Console.WriteLine("Starting SetGroupRegistrationData");

        using (ProvisioningServiceClient provisioningServiceClient =
                ProvisioningServiceClient.CreateFromConnectionString(ServiceConnectionString))
        {
            Console.WriteLine("\nCreating a new enrollmentGroup...");

            var certificate = new X509Certificate2(X509RootCertPathVar);

            Attestation attestation = X509Attestation.CreateFromRootCertificates(certificate);

            EnrollmentGroup enrollmentGroup = new EnrollmentGroup(SampleEnrollmentGroupId, attestation);

            Console.WriteLine(enrollmentGroup);
            Console.WriteLine("\nAdding new enrollmentGroup...");

            EnrollmentGroup enrollmentGroupResult =
                await provisioningServiceClient.CreateOrUpdateEnrollmentGroupAsync(enrollmentGroup).ConfigureAwait(false);

            Console.WriteLine("\nEnrollmentGroup created with success.");
            Console.WriteLine(enrollmentGroupResult);
        }
    }
    ```

1. И наконец, добавьте этот код в метод **Main**, чтобы открыть подключение к Центру Интернета вещей и начать групповую регистрацию.
   
    ```csharp
    try
    {
        Console.WriteLine("IoT Device Group Provisioning example");

        SetGroupRegistrationDataAsync().GetAwaiter().GetResult();
            
        Console.WriteLine("Done, hit enter to exit.");
        Console.ReadLine();
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    ```

1. Запустите приложение для устройства .NET **DeviceProvisiong**. Оно должно настроить группы подготовки устройства. 

    ![Выполнение регистрации группы](./media/tutorial-net-provision-device-to-hub/group.png)

    При успешной регистрации группы устройств на портале можно увидеть следующее:

   ![Успешная регистрация группы на портале](./media/tutorial-net-provision-device-to-hub/group-portal.png)


## <a name="start-the-device"></a>запуск устройства;

На этом этапе для регистрации устройства готова следующая настройка:

1. Устройство или группа устройств регистрируется в службе подготовки устройств. 
2. Устройство готово и доступно через приложение, которое использует клиентский пакет SDK службы подготовки устройств.

Запустите устройство, чтобы разрешить клиентскому приложению начать регистрацию в службе подготовки устройств.  


## <a name="verify-the-device-is-registered"></a>проверка регистрации устройства.

После загрузки устройства необходимо выполнить следующие действия. Дополнительные сведения см. в репозитории с [примером клиента для подготовки устройств](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/provisioning/Samples/device). 

1. Устройство отправляет запрос на регистрацию в службу подготовки устройств.
2. Устройствам с доверенным платформенным модулем служба подготовки устройств отправляет обратный запрос регистрации, на который отвечает устройство. 
3. При успешной регистрации служба подготовки устройств отправляет обратно к устройству универсальный код ресурса Центра Интернета вещей, идентификатор устройства и зашифрованный ключ. 
4. Затем клиентское приложение Центра Интернета вещей на устройстве подключается к центру. 
5. При успешном подключении к центру устройство отобразится в средстве **Device Explorer** Центра Интернета вещей. 

    ![Успешное подключение к центру на портале](./media/tutorial-net-provision-device-to-hub/hub-connect-success.png)

## <a name="next-steps"></a>Дальнейшие действия
В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * Регистрация устройства
> * запуск устройства;
> * проверка регистрации устройства.

Перейдите к следующему руководству, чтобы узнать, как подготовить несколько устройств через концентраторы с балансировкой нагрузки. 

> [!div class="nextstepaction"]
> [Подготовка устройств в Центрах Интернета вещей с балансировкой нагрузки](./tutorial-provision-multiple-hubs.md)