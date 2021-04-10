---
title: Учебник по использованию пользовательских политик выделения со Службой подготовки устройств к добавлению в Центр Интернета вещей Azure (DPS)
description: Учебник по использованию пользовательских политик выделения со Службой подготовки устройств к добавлению в Центр Интернета вещей Azure (DPS)
author: wesmc7777
ms.author: wesmc
ms.date: 09/23/2020
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.custom: mvc
ms.openlocfilehash: 42098786bb92e98b89b1dbfba01a2e071398f460
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99258546"
---
# <a name="tutorial-use-custom-allocation-policies-with-device-provisioning-service-dps"></a>Руководство по использованию пользовательских политик выделения со Службой подготовки устройств к добавлению в Центр Интернета вещей Azure (DPS)

Пользовательская политика выделения обеспечивает больший контроль над назначением устройств в Центре Интернета вещей. Это достигается с помощью пользовательского кода в [Функциях Azure](../azure-functions/functions-overview.md), который выполняется при подготовке для назначения устройств в центр Интернета вещей. Служба подготовки устройств вызывает код функции Azure, предоставляя всю необходимую информацию об устройстве и регистрации. Код функции выполняется и возвращает данные Центра Интернета вещей, используемые при подготовке устройства.

С помощью функции пользовательских политик выделения можно определить собственные политики выделения, если политики, предоставленные службой подготовки устройств, не соответствуют требованиям вашего сценария.

Например, вам может понадобиться проверить сертификат, который устройство использует во время подготовки, и назначить устройство в центр Интернета вещей на основе свойства сертификата. Кроме того, сведения об устройствах могут храниться в базе данных, и чтобы определить, какому центру Интернета вещей следует назначить устройство, требуется отправить запрос к этой базе данных.

В этой статье рассказывается о применении группы регистрации с пользовательской политикой выделения, использующей функцию Azure, написанную на C#, для подготовки тостеров с использованием симметричных ключей. Устройства, не распознанные как тостеры, не будут подготовлены для центра Интернета вещей.

Устройства будут запрашивать подготовку с помощью примера для подготовки из [пакета SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c).


В этом учебнике рассматриваются следующие задачи:

> [!div class="checklist"]
> * Создание нового приложения-функции Azure для поддержки пользовательской функции выделения
> * Создание группы регистрации с помощью функции Azure для пользовательской политики выделения.
> * Создание ключей устройства для двух устройств
> * Подготовка среды разработки для примера кода устройства из [пакета SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c)
> * Запуск устройств и проверка их подготовки в соответствии с пользовательской политикой выделения


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

* В этой статье предполагается, что вы выполнили шаги, описанные в статье [Настройка Службы подготовки устройств для Центра Интернета вещей на портале Azure](./quick-setup-auto-provision.md) для создания центра Интернета вещей и экземпляра DPS.

* Установите последнюю версию [Git](https://git-scm.com/download/).

* Для среды разработки Windows требуется [Visual Studio](https://visualstudio.microsoft.com/vs/) 2019 с включенной рабочей нагрузкой [Разработка классических приложений на C++](/cpp/ide/using-the-visual-studio-ide-for-cpp-desktop-development). Visual Studio 2015 или Visual Studio 2017 также поддерживаются.

* При использовании Linux или macOS ознакомьтесь с соответствующим разделом статье [Подготовка среды разработки](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md) из документации к [пакету SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c).


[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-the-custom-allocation-function"></a>Создание пользовательской функции выделения

В этом разделе вы создадите функцию Azure, которая реализует пользовательскую политику выделения. Эта функция определяет, следует ли зарегистрировать устройство в Центре Интернета вещей, в зависимости от того, содержит ли его идентификатор регистрации строковый префикс **contoso-toaster**.

1. Войдите на [портал Azure](https://portal.azure.com). На домашней странице портала Azure выберите **+ Создать ресурс**.

2. В поле *Поиск в Marketplace* введите "Приложение-функция". В раскрывающемся списке выберите **Приложение-функция**, а затем — **Создать**.

3. На открывшейся странице создания **приложения-функции** на вкладке **Основы** введите приведенные ниже параметры новой функции и нажмите **Просмотр и создание**.

    **Подписка**: Если у вас несколько подписок и нужная подписка не выбрана, выберите ту, которую хотите использовать.

    **Группа ресурсов**. Это поле позволяет создать новую группу ресурсов или выбрать существующую. В этой группе ресурсов будет содержаться новое приложение-функция. Выберите ту же группу ресурсов, содержащую центр Интернета вещей, созданный ранее для тестирования, например **TestResources**. Помещение всех связанных ресурсов в одну группу позволит управлять ими совместно.

    **Имя приложения-функции**. Введите уникальное имя приложения-функции. В данном примере это **contoso-function-app**.

    **Publish**: Убедитесь, что выбран параметр **Код**.

    **Стек среды выполнения**. В раскрывающемся меню выберите **.NET Core**.

    **Регион**. Выберите тот же регион, что и для группы ресурсов. В этом примере используется регион **западная часть США**.

    > [!NOTE]
    > По умолчанию служба Application Insights активирована. Она необязательна для этой статьи, но может помочь понять и исследовать проблемы, возникающие при работе с пользовательским выделением. При желании Application Insights можно отключить. Для этого на вкладке **Мониторинг** выберите **Нет** для параметра **Включить Application Insights**.

    ![Создание приложения-функции Azure для размещения пользовательской функции выделения](./media/tutorial-custom-allocation-policies/create-function-app.png)

4. На странице **Сводка** щелкните **Создать**, чтобы создать приложение-функцию. Развертывание может занять несколько минут. По завершении выберите **Перейдите к ресурсу**.

5. В области слева в разделе **Функции** нажмите **Функции**, а затем — **+ Добавить**, чтобы добавить новую функцию.

6. На странице шаблонов выберите плитку **Триггер HTTP**, а затем нажмите **Создать функцию**. Будет создана функция с именем **HttpTrigger1**, а на портале отобразится ее страница обзора.

7. Щелкните **Код и тестирование** для новой функции. На портале отобразится содержимое файла с кодом **run.csx**. 

8. Замените код этой функции **HttpTrigger1** приведенным ниже кодом и нажмите **Сохранить**. Пользовательский код выделения готов к использованию.

    ```csharp
    #r "Newtonsoft.Json"

    using System.Net;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;

    public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        // Get request body
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);

        log.LogInformation("Request.Body:...");
        log.LogInformation(requestBody);

        // Get registration ID of the device
        string regId = data?.deviceRuntimeContext?.registrationId;

        string message = "Uncaught error";
        bool fail = false;
        ResponseObj obj = new ResponseObj();

        if (regId == null)
        {
            message = "Registration ID not provided for the device.";
            log.LogInformation("Registration ID : NULL");
            fail = true;
        }
        else
        {
            string[] hubs = data?.linkedHubs.ToObject<string[]>();

            // Must have hubs selected on the enrollment
            if (hubs == null)
            {
                message = "No hub group defined for the enrollment.";
                log.LogInformation("linkedHubs : NULL");
                fail = true;
            }
            else
            {
                // This is a Contoso Toaster 
                if (regId.Contains("contoso-toaster"))
                {
                    //Log IoT hubs configured for the enrollment
                    foreach(string hubString in hubs)
                    {
                        log.LogInformation("linkedHub : " + hubString);
                    }

                    obj.iotHubHostName = hubs[0];
                    log.LogInformation("Selected hub : " + obj.iotHubHostName);
                }
                else
                {
                    fail = true;
                    message = "Unrecognized device registration.";
                    log.LogInformation("Unknown device registration");
                }
            }
        }

        log.LogInformation("\nResponse");
        log.LogInformation((obj.iotHubHostName != null) ? JsonConvert.SerializeObject(obj) : message);

        return (fail)
            ? new BadRequestObjectResult(message) 
            : (ActionResult)new OkObjectResult(obj);
    }

    public class ResponseObj
    {
        public string iotHubHostName {get; set;}
    }
    ```

9. В самом низу файла с кодом **run.csx** щелкните **Журналы**, чтобы отслеживать ведение журнала пользовательской функцией выделения. 


## <a name="create-the-enrollment"></a>Создание регистрации

В этом разделе вы узнаете, как создать группу регистрации, использующую пользовательскую политику выделения. Для удобства в этой статье используется [аттестация симметричного ключа](concepts-symmetric-key-attestation.md) с регистрацией. В качестве более безопасного решения рекомендуется использовать [аттестацию сертификатов X.509](concepts-x509-attestation.md) с цепочкой доверия.

1. На [портале Azure](https://portal.azure.com) откройте службу подготовки.

2. В области слева выберите вкладку **Управление регистрациями**, а затем нажмите кнопку **Добавить группу регистрации** в верхней части страницы.

3. В разделе **Добавление группы регистрации** введите сведения из таблицы ниже, а затем нажмите кнопку **Сохранить**.

    | Поле | Описание и (или) предложенное значение |
    | :---- | :----------------------------- |
    | **Имя группы** | Введите **contoso-custom-allocated-devices** |
    | **Тип аттестации** | Выберите **Симметричный ключ** |
    | **Автоматическое формирование ключей** | этот флажок должен быть уже установлен. |
    | **Выберите способ назначения устройств для Центров** | Выберите **Пользовательская (используйте функцию Azure)** |
    | **Выберите центры Интернета вещей, которым может быть назначена эта группа** | Выберите центр Интернета вещей, созданный ранее при прохождении краткого руководства. |
    | **Выберите функцию Azure** | Выберите подписку, которая содержит созданное приложение-функцию. Затем выберите для функции **contoso-function-app** и **HttpTrigger1**. |

    ![Добавление группы регистрации пользовательского выделения для аттестации симметричного ключа](./media/tutorial-custom-allocation-policies/create-custom-allocation-enrollment.png)

4. После сохранения регистрации снова откройте ее и запомните или запишите **первичный ключ**. Для формирования ключей необходимо сначала сохранить регистрацию. Этот первичный симметричный ключ будет использоваться для создания уникальных ключей устройств, которые попытаются выполнить подготовку позже. 

## <a name="derive-unique-device-keys"></a>Получение производных уникальных ключей

Устройства не используют первичный симметричный ключ напрямую. Вместо этого для получения ключа каждого устройства используется первичный ключ. В этом разделе описано создание двух уникальных ключей устройств. Один ключ будет использоваться для имитированного устройства toaster. Другой ключ будет использоваться для имитированного устройства heat pump. Созданные ключи позволяют обоим устройствам предпринять попытку регистрации. Только один идентификатор регистрации устройства будет иметь действительный суффикс, который должен быть принят примером кода пользовательской политики выделения. В результате один из них будет принят, а другой отклонен.

Чтобы получить ключ устройства, используйте симметричный ключ, записанный ранее, для вычисления [HMAC-SHA256](https://wikipedia.org/wiki/HMAC) на основе идентификатора регистрации каждого устройства и преобразования результата в формат Base64. Дополнительные сведения о создании производных ключей устройств с группами регистраций см. в разделе о регистрации групп в статье [Аттестация симметричного ключа](concepts-symmetric-key-attestation.md).

Для примера в этой статье используйте следующие два идентификатора регистрации устройств с приведенными ниже кодом для вычисления ключей обоих устройств:

* **contoso-toaster-007**
* **contoso-heatpump-088**

Замените значение переменной **KEY** **первичным ключом**, который вы записали ранее после создания группы регистрации. Значение ключа и выходные данные для кода ниже приведены только для примера.

#### <a name="powershell"></a>[PowerShell](#tab/powershell)

Если вы используете рабочую станцию для Windows, можно использовать PowerShell для формирования производного ключа устройства, как показано в следующем примере.

```powershell
$KEY='oiK77Oy7rBw8YB6IS6ukRChAw+Yq6GC61RMrPLSTiOOtdI+XDu0LmLuNm11p+qv2I+adqGUdZHm46zXAQdZoOA=='

$REG_ID1='contoso-toaster-007'
$REG_ID2='contoso-heatpump-088'

$hmacsha256 = New-Object System.Security.Cryptography.HMACSHA256
$hmacsha256.key = [Convert]::FromBase64String($key)
$sig1 = $hmacsha256.ComputeHash([Text.Encoding]::ASCII.GetBytes($REG_ID1))
$sig2 = $hmacsha256.ComputeHash([Text.Encoding]::ASCII.GetBytes($REG_ID2))
$derivedkey1 = [Convert]::ToBase64String($sig1)
$derivedkey2 = [Convert]::ToBase64String($sig2)

echo "`n`n$REG_ID1 : $derivedkey1`n$REG_ID2 : $derivedkey2`n`n"
```

```powershell
contoso-toaster-007 : JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=
contoso-heatpump-088 : 6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=
```

#### <a name="bash"></a>[Bash](#tab/bash)

Если вы используете рабочую станцию Linux, можно использовать openssl для формирования производных ключей устройств, как показано в следующем примере для Bash.

```bash
KEY=oiK77Oy7rBw8YB6IS6ukRChAw+Yq6GC61RMrPLSTiOOtdI+XDu0LmLuNm11p+qv2I+adqGUdZHm46zXAQdZoOA==

REG_ID1=contoso-toaster-007
REG_ID2=contoso-heatpump-088

keybytes=$(echo $KEY | base64 --decode | xxd -p -u -c 1000)
devkey1=$(echo -n $REG_ID1 | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64)
devkey2=$(echo -n $REG_ID2 | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64)

echo -e $"\n\n$REG_ID1 : $devkey1\n$REG_ID2 : $devkey2\n\n"
```

```bash
contoso-toaster-007 : JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=
contoso-heatpump-088 : 6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=
```

---


## <a name="prepare-an-azure-iot-c-sdk-development-environment"></a>Подготовка среды разработки для пакета SDK Azure IoT для C

Устройства будут запрашивать подготовку с помощью примера для подготовки из [пакета SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c).

В этом разделе описано, как подготовить среду разработки, которая используется для сборки [пакета SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c). В пакет SDK входит пример кода для имитированного устройства. Для этого имитированного устройства будет выполнена попытка подготовки во время последовательности загрузки.

В этом разделе описывается использование рабочей станции Windows. Пример для Linux приведен в инструкциях по настройке виртуальных машин в статье [Подготовка к мультитенантности](how-to-provision-multitenant.md).

1. Скачайте [систему сборки CMake](https://cmake.org/download/).

    **Перед** установкой `CMake` очень важно установить на компьютер необходимые компоненты Visual Studio (Visual Studio с рабочей нагрузкой "Разработка классических приложений на C++"). После установки компонентов и проверки загрузки установите систему сборки CMake.

2. Найдите имя тега для [последнего выпуска](https://github.com/Azure/azure-iot-sdk-c/releases/latest) пакета SDK.

3. Откройте командную строку или оболочку Git Bash. Выполните приведенные ниже команды, чтобы клонировать репозиторий GitHub с последним выпуском [пакета SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c). Используйте найденный тег в качестве значения для параметра `-b`:

    ```cmd/sh
    git clone -b <release-tag> https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Выполнение этой операции может занять несколько минут.

4. Создайте подкаталог `cmake` в корневом каталоге репозитория Git и перейдите в эту папку. В каталоге `azure-iot-sdk-c` выполните следующие команды:

    ```cmd/sh
    mkdir cmake
    cd cmake
    ```

5. Выполните приведенную ниже команду, чтобы создать версию пакета SDK для используемой клиентской платформы разработки. Эта команда также создает решение Visual Studio для имитированного устройства в каталоге `cmake`. 

    ```cmd
    cmake -Dhsm_type_symm_key:BOOL=ON -Duse_prov_client:BOOL=ON  ..
    ```

    Если `cmake` не удается найти компилятор C++, во время выполнения команды могут возникнуть ошибки сборки. В этом случае попробуйте, выполнить эту команду в [командной строке Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs).

    После успешного создания последние несколько строк выходных данных будут выглядеть следующим образом:

    ```cmd/sh
    $ cmake -Dhsm_type_symm_key:BOOL=ON -Duse_prov_client:BOOL=ON  ..
    -- Building for: Visual Studio 15 2017
    -- Selecting Windows SDK version 10.0.16299.0 to target Windows 10.0.17134.
    -- The C compiler identification is MSVC 19.12.25835.0
    -- The CXX compiler identification is MSVC 19.12.25835.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: E:/IoT Testing/azure-iot-sdk-c/cmake
    ```

## <a name="simulate-the-devices"></a>Имитация устройств

В этом разделе описано, как обновить пример для подготовки **prov\_dev\_client\_sample**, который размещен в пакете SDK Azure IoT для C, настроенном ранее.

Пример кода имитирует последовательность загрузки устройства, которое отправляет запрос на подготовку в экземпляр службы подготовки устройств. При выполнении последовательности загрузки устройство toaster будет распознано и назначено Центру Интернета вещей с помощью пользовательской политики выделения.

1. На портале Azure выберите вкладку **Обзор** службы подготовки устройств и запишите значение **_области идентификатора_**.

    ![Извлеките сведения о конечной точке службы подготовки устройств из колонки на портале](./media/quick-create-simulated-device-x509/extract-dps-endpoints.png) 

2. В Visual Studio откройте файл решения **azure_iot_sdks.sln**, который был создан ранее в результате запуска CMake. Файл решения должен находиться в следующем расположении:

    ```
    azure-iot-sdk-c\cmake\azure_iot_sdks.sln
    ```

3. В окне *Обозреватель решений* Visual Studio перейдите в папку **Provision\_Samples**. Разверните пример проекта с именем **prov\_dev\_client\_sample**. Разверните **исходные файлы** и откройте **prov\_dev\_client\_sample.c**.

4. Найдите константу `id_scope` и замените ее значение ранее скопированным значением **области идентификатора**. 

    ```c
    static const char* id_scope = "0ne00002193";
    ```

5. Найдите определение функции `main()` в том же файле. Проверьте, чтобы переменной `hsm_type` было задано значение `SECURE_DEVICE_TYPE_SYMMETRIC_KEY`, как показано ниже.

    ```c
    SECURE_DEVICE_TYPE hsm_type;
    //hsm_type = SECURE_DEVICE_TYPE_TPM;
    //hsm_type = SECURE_DEVICE_TYPE_X509;
    hsm_type = SECURE_DEVICE_TYPE_SYMMETRIC_KEY;
    ```

6. В функции `main()` найдите вызов `Prov_Device_Register_Device()`. Непосредственно перед этим вызовом добавьте следующие строки кода, которые используются в [`Prov_Device_Set_Provisioning_Payload()`](/azure/iot-hub/iot-c-sdk-ref/prov-device-client-h/prov-device-set-provisioning-payload) для передачи пользовательских полезных данных JSON во время подготовки. Их можно использовать для предоставления дополнительной информации вашим пользовательским функциям распределения. Их также можно использовать для передачи типа устройства вместо проверки идентификатора регистрации. Дополнительные сведения об отправке и получении полезных данных с помощью DPS см. в статье [Передача полезных данных между устройствами и службой DPS](how-to-send-additional-data.md).

    ```c
    // An example custom payload
    const char* custom_json_payload = "{\"MyDeviceFirmwareVersion\":\"12.0.2.5\",\"MyDeviceProvisioningVersion\":\"1.0.0.0\"}";

    prov_device_result = Prov_Device_Set_Provisioning_Payload(prov_device_handle, custom_json_payload);
    if (prov_device_result != PROV_DEVICE_RESULT_OK)
    {
        (void)printf("\r\nFailure setting provisioning payload: %s\r\n", MU_ENUM_TO_STRING(PROV_DEVICE_RESULT, prov_device_result));
    }
    ```

7. Щелкните проект **prov\_dev\_client\_sample** правой кнопкой мыши и выберите пункт **Назначить запускаемым проектом**.

### <a name="simulate-the-contoso-toaster-device"></a>Имитация устройства toaster Contoso

1. Для имитации устройства тостера найдите закомментированный вызов `prov_dev_set_symmetric_key_info()` в **prov\_dev\_client\_sample.c**.

    ```c
    // Set the symmetric key if using they auth type
    //prov_dev_set_symmetric_key_info("<symm_registration_id>", "<symmetric_Key>");
    ```

    Раскомментируйте вызов функции и замените значения заполнителей (включая угловые скобки) идентификатором регистрации тостера и производным ключом устройства, который был создан ранее. Значение ключа **JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=**, показанное ниже, приведено только в качестве примера.

    ```c
    // Set the symmetric key if using they auth type
    prov_dev_set_symmetric_key_info("contoso-toaster-007", "JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=");
    ```

    Сохраните файл.

2. В меню Visual Studio выберите **Отладка** > **Запуск без отладки**, чтобы запустить решение. При появлении запроса перестроить проект щелкните **Да**, чтобы перестроить его перед запуском.

    Ниже приведен пример вывода данных журнала из кода пользовательской функции выделения, выполняемого для тостера. Обратите внимание, что для тостера успешно выбран центр. Кроме того, обратите внимание, что элемент `payload` содержит настраиваемое содержимое JSON, добавленное вами в код. Благодаря этому код можно использовать в `deviceRuntimeContext`.

    Эти данные журнала можно просмотреть, щелкнув **Журналы** в коде функции на портале.

    ```cmd
    2020-09-23T11:44:37.505 [Information] Executing 'Functions.HttpTrigger1' (Reason='This function was programmatically called via the host APIs.', Id=4596d45e-086f-4e86-929b-4a02814eee40)
    2020-09-23T11:44:41.380 [Information] C# HTTP trigger function processed a request.
    2020-09-23T11:44:41.381 [Information] Request.Body:...
    2020-09-23T11:44:41.381 [Information] {"enrollmentGroup":{"enrollmentGroupId":"contoso-custom-allocated-devices","attestation":{"type":"symmetricKey"},"capabilities":{"iotEdge":false},"etag":"\"e8002126-0000-0100-0000-5f6b2a570000\"","provisioningStatus":"enabled","reprovisionPolicy":{"updateHubAssignment":true,"migrateDeviceData":true},"createdDateTimeUtc":"2020-09-23T10:58:31.62286Z","lastUpdatedDateTimeUtc":"2020-09-23T10:58:31.62286Z","allocationPolicy":"custom","iotHubs":["contoso-toasters-hub-1098.azure-devices.net"],"customAllocationDefinition":{"webhookUrl":"https://contoso-function-app.azurewebsites.net/api/HttpTrigger1?****","apiVersion":"2019-03-31"}},"deviceRuntimeContext":{"registrationId":"contoso-toaster-007","symmetricKey":{},"payload":{"MyDeviceFirmwareVersion":"12.0.2.5","MyDeviceProvisioningVersion":"1.0.0.0"}},"linkedHubs":["contoso-toasters-hub-1098.azure-devices.net"]}
    2020-09-23T11:44:41.687 [Information] linkedHub : contoso-toasters-hub-1098.azure-devices.net
    2020-09-23T11:44:41.688 [Information] Selected hub : contoso-toasters-hub-1098.azure-devices.net
    2020-09-23T11:44:41.688 [Information] Response
    2020-09-23T11:44:41.688 [Information] {"iotHubHostName":"contoso-toasters-hub-1098.azure-devices.net"}
    2020-09-23T11:44:41.689 [Information] Executed 'Functions.HttpTrigger1' (Succeeded, Id=4596d45e-086f-4e86-929b-4a02814eee40, Duration=4347ms)    
    ```

    Следующий пример результата для устройства демонстрирует успешную загрузку имитированного тостера и его подключения к экземпляру службы подготовки для назначения центру Интернета вещей для тостеров с помощью пользовательской политики выделения.

    ```cmd
    Provisioning API Version: 1.3.6

    Registering Device

    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

    Registration Information received from service: contoso-toasters-hub-1098.azure-devices.net, deviceId: contoso-toaster-007

    Press enter key to exit:
    ```

### <a name="simulate-the-contoso-heat-pump-device"></a>Имитация устройства heat pump Contoso

1. Для имитации устройства теплового насоса обновите вызов `prov_dev_set_symmetric_key_info()` в **prov\_dev\_client\_sample.c**, указав идентификатор регистрации теплового насоса и производный ключ устройства, созданный ранее. Значение ключа **6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=**, показанное ниже, приведено только в качестве примера.

    ```c
    // Set the symmetric key if using they auth type
    prov_dev_set_symmetric_key_info("contoso-heatpump-088", "6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=");
    ```

    Сохраните файл.

2. В меню Visual Studio выберите **Отладка** > **Запуск без отладки**, чтобы запустить решение. При появлении запроса перестроить проект щелкните **Да**, чтобы перестроить его перед запуском.

    Ниже приведен пример вывода данных журнала из кода пользовательской функции выделения, выполняемого для теплового насоса. Пользовательская политика выделения отклоняет эту регистрацию с ошибкой HTTP 400 (неверный запрос). Обратите внимание, что элемент `payload` содержит настраиваемое содержимое JSON, добавленное вами в код. Благодаря этому код можно использовать в `deviceRuntimeContext`.

    Эти данные журнала можно просмотреть, щелкнув **Журналы** в коде функции на портале.

    ```cmd
    2020-09-23T11:50:23.652 [Information] Executing 'Functions.HttpTrigger1' (Reason='This function was programmatically called via the host APIs.', Id=2fa77f10-42f8-43fe-88d9-a8c01d4d3f68)
    2020-09-23T11:50:23.653 [Information] C# HTTP trigger function processed a request.
    2020-09-23T11:50:23.654 [Information] Request.Body:...
    2020-09-23T11:50:23.654 [Information] {"enrollmentGroup":{"enrollmentGroupId":"contoso-custom-allocated-devices","attestation":{"type":"symmetricKey"},"capabilities":{"iotEdge":false},"etag":"\"e8002126-0000-0100-0000-5f6b2a570000\"","provisioningStatus":"enabled","reprovisionPolicy":{"updateHubAssignment":true,"migrateDeviceData":true},"createdDateTimeUtc":"2020-09-23T10:58:31.62286Z","lastUpdatedDateTimeUtc":"2020-09-23T10:58:31.62286Z","allocationPolicy":"custom","iotHubs":["contoso-toasters-hub-1098.azure-devices.net"],"customAllocationDefinition":{"webhookUrl":"https://contoso-function-app.azurewebsites.net/api/HttpTrigger1?****","apiVersion":"2019-03-31"}},"deviceRuntimeContext":{"registrationId":"contoso-heatpump-088","symmetricKey":{},"payload":{"MyDeviceFirmwareVersion":"12.0.2.5","MyDeviceProvisioningVersion":"1.0.0.0"}},"linkedHubs":["contoso-toasters-hub-1098.azure-devices.net"]}
    2020-09-23T11:50:23.654 [Information] Unknown device registration
    2020-09-23T11:50:23.654 [Information] Response
    2020-09-23T11:50:23.654 [Information] Unrecognized device registration.
    2020-09-23T11:50:23.655 [Information] Executed 'Functions.HttpTrigger1' (Succeeded, Id=2fa77f10-42f8-43fe-88d9-a8c01d4d3f68, Duration=11ms)
    ```

    Следующий пример результата для устройства демонстрирует успешную загрузку имитированного теплового насоса и его подключения к экземпляру службы подготовки для попытки регистрации в центре Интернета вещей с помощью пользовательской политики выделения. Это приведет к ошибке (`Custom allocation failed with status code: 400`), так как пользовательская политика выделения предназначена только для тостеров.


    ```cmd
    Provisioning API Version: 1.3.7
    
    Registering Device
    
    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Error: Time:Wed Sep 23 13:06:01 2020 File:d:\testing\azure-iot-sdk-c\provisioning_client\src\prov_device_ll_client.c Func:_prov_transport_process_json_reply Line:658 Provisioning Failure: OperationId: 4.eb89f3e8407a3711.2525bd34-02e9-4e91-a9c0-4dbc4ad5de66 - Date: 2020-09-23T17:05:58.2363145Z - Msg: Custom allocation failed with status code: 400
    Error: Time:Wed Sep 23 13:06:01 2020 File:d:\testing\azure-iot-sdk-c\provisioning_client\src\prov_transport_mqtt_common.c Func:_prov_transport_common_mqtt_dowork Line:1014 Unable to process registration reply.
    Error: Time:Wed Sep 23 13:06:01 2020 File:d:\testing\azure-iot-sdk-c\provisioning_client\src\prov_device_ll_client.c Func:_on_transport_registration_data Line:770 Failure retrieving data from the provisioning service
    
    Failure registering device: PROV_DEVICE_RESULT_DEV_AUTH_ERROR
    Press enter key to exit:    
    ```
    
## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете продолжить работу с ресурсами, созданными при работе с этой статьей, не удаляйте их. В противном случае выполните следующие действия, чтобы удалить все созданные ресурсы, за которые может взиматься плата.

Здесь предполагается, что вы создали все используемые в этой статье ресурсы, как было указано, в одной группе ресурсов, **contoso-us-resource-group**.

> [!IMPORTANT]
> Удаление группы ресурсов — процесс необратимый. Группа ресурсов и все содержащиеся в ней ресурсы удаляются без возможности восстановления. Будьте внимательны, чтобы случайно не удалить не ту группу ресурсов или не те ресурсы. Если вы создали Центр Интернета вещей в группе ресурсов, содержащей ресурсы, которые нужно сохранить, удалите только ресурс Центра Интернета вещей, не удаляя всю группу ресурсов.
>

Удаление группы ресурсов по имени.

1. Войдите в портал [Azure](https://portal.azure.com) и выберите **Группы ресурсов**.

2. Введите в текстовое поле **Фильтровать по имени...** имя вашей группы ресурсов: **contoso-us-resource-group**. 

3. Справа от своей группы ресурсов в списке результатов щелкните **...**, а затем выберите **Удалить группу ресурсов**.

4. Вам будет предложено подтвердить операцию удаления группы ресурсов. Снова введите имя группы ресурсов, которую необходимо удалить, и щелкните **Удалить**. Через некоторое время группа ресурсов и все ее ресурсы будут удалены.

## <a name="next-steps"></a>Дальнейшие действия

* Более подробный пример пользовательской политики выделения см. в статье [Как использовать пользовательские политики выделения](how-to-use-custom-allocation-policies.md).
* Дополнительные сведения о повторной подготовке см. в статье, посвященной [основным понятиям повторной подготовки устройств в Центре Интернета вещей](concepts-device-reprovision.md).
* Дополнительные сведения об отмене подготовки см. в статье [Как отменить подготовку устройств, которые были автоматически подготовлены](how-to-unprovision-devices.md).