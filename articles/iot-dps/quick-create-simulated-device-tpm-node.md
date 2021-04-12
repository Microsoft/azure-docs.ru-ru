---
title: Краткое руководство. Подготовка имитированного устройства доверенного платформенного модуля в Центре Интернета вещей Azure с помощью Node.js
description: Краткое руководство по созданию и подготовке имитированного устройства доверенного платформенного модуля с помощью пакета SDK Node.js для устройств для Службы подготовки устройств к добавлению в Центр Интернета вещей Azure (DPS). В этом кратком руководстве используется индивидуальная регистрация.
author: wesmc7777
ms.author: wesmc
ms.date: 11/08/2018
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.custom:
- mvc
- amqp
- mqtt
- devx-track-js
ms.openlocfilehash: 6e7e986f658570553763001afdd58d7bb1880f94
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94968185"
---
# <a name="quickstart-create-and-provision-a-simulated-tpm-device-using-nodejs-device-sdk-for-iot-hub-device-provisioning-service"></a>Краткое руководство. Создание и подготовка имитированного устройства TPM с помощью пакета SDK устройств для Node.js для службы "Подготовка устройств к добавлению в Центр Интернета вещей"

[!INCLUDE [iot-dps-selector-quick-create-simulated-device-tpm](../../includes/iot-dps-selector-quick-create-simulated-device-tpm.md)]

С помощью этого краткого руководства вы создадите имитированное устройство Интернета вещей на компьютере Windows. В состав этого имитированного устройства входит симулятор TPM в качестве аппаратного модуля безопасности (HSM). Для подключения этого имитированного устройства к центру Интернета вещей вы будете использовать пример кода Node.js для устройства, а также индивидуальную регистрацию в Службе подготовки устройств (DPS).

## <a name="prerequisites"></a>Предварительные требования

- Ознакомьтесь с разделом [Роли и учетные записи Azure](about-iot-dps.md#provisioning-process).
- Выполнение инструкций из краткого руководства по [настройке Службы подготовки устройств к добавлению в Центр Интернета вещей на портале Azure](./quick-setup-auto-provision.md).
- Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [Node.js версии 4.0 и более поздней](https://nodejs.org).
- [Git](https://git-scm.com/download/).

[!INCLUDE [IoT Device Provisioning Service basic](../../includes/iot-dps-basic.md)]

## <a name="prepare-the-environment"></a>Подготовка среды 

1. Убедитесь, что на вашем компьютере установлена платформа [Node.js 4.0 или более поздней версии](https://nodejs.org).

1. Установите на компьютер систему `git` и добавьте ее в переменные среды, доступные в командном окне. Последнюю версию средств `git` для установки, которая включает **Git Bash**, приложение командной строки для взаимодействия с локальным репозиторием Git, можно найти на [этой странице](https://git-scm.com/download/). 


## <a name="simulate-a-tpm-device"></a>Имитация устройства TPM

1. Откройте окно командной строки или Git Bash. Клонируйте репозиторий GitHub `azure-utpm-c`:
    
    ```cmd/sh
    git clone https://github.com/Azure/azure-utpm-c.git --recursive
    ```

1. Перейдите к корневой папке GitHub и запустите симулятор [доверенного платформенного модуля](/windows/device-security/tpm/trusted-platform-module-overview), чтобы он выступал для имитированного устройства в качестве [HSM](https://azure.microsoft.com/blog/azure-iot-supports-new-security-hardware-to-strengthen-iot-security/). Он ожидает передачи данных через сокет на портах 2321 и 2322. Не закрывайте командное окно. Симулятор должен работать, пока вы не выполните все инструкции из этого руководства: 

    ```cmd/sh
    .\azure-utpm-c\tools\tpm_simulator\Simulator.exe
    ```

1. Создайте пустую папку с именем **registerdevice**. В папке **registerdevice** создайте файл package.json, используя следующую команду в командной строке. Ответьте на все вопросы, заданные `npm`, или примите значения по умолчанию, если они вам подходят:
   
    ```cmd/sh
    npm init
    ```

1. Установите следующие основные пакеты:

    ```cmd/sh
    npm install node-gyp -g
    npm install ffi -g
    ```

    > [!NOTE]
    > При установке указанных выше пакетов могут возникнуть некоторые известные проблемы. Чтобы устранить их, выполните команду `npm install --global --production windows-build-tools` с помощью командной строки в режиме **Запуск от имени администратора**, а затем выполните `SET VCTargetsPath=C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\V140`, заменив путь на путь к текущей установленной версии, и повторно выполните указанные выше команды установки.
    >

1. Установите следующие пакеты, содержащие компоненты, которые используются во время регистрации:

   - клиент безопасности, который работает с доверенным платформенным модулем: `azure-iot-security-tpm`;
   - транспортный протокол для подключения устройства к службе подготовки устройств: `azure-iot-provisioning-device-http` или `azure-iot-provisioning-device-amqp`;
   - клиент, использующий транспортный протокол и клиент безопасности: `azure-iot-provisioning-device`.

     После регистрации устройства для подключения можно использовать обычные клиенсткие пакеты устройства Центра Интернета вещей, указав учетные данные, предоставленные во время регистрации. Вам потребуется:

   - клиент устройства: `azure-iot-device`;
   - транспортный протокол: любой из `azure-iot-device-amqp`, `azure-iot-device-mqtt` или `azure-iot-device-http`;
   - установленный клиент безопасности: `azure-iot-security-tpm`.

     > [!NOTE]
     > В примерах ниже используются транспортные протоколы `azure-iot-provisioning-device-http` и `azure-iot-device-mqtt`.
     > 

     Все эти пакеты можно установить одновременно, выполнив следующую команду в командной строке в папке **registereddevice**:

       ```cmd/sh
       npm install --save azure-iot-device azure-iot-device-mqtt azure-iot-security-tpm azure-iot-provisioning-device-http azure-iot-provisioning-device
       ```

1. В текстовом редакторе создайте файл **ExtractDevice.js** в папке **registerdevice**.

1. Добавьте следующие инструкции `require` в начало файла **ExtractDevice.js**:
   
    ```
    'use strict';

    var tpmSecurity = require('azure-iot-security-tpm');
    var tssJs = require("tss.js");

    var myTpm = new tpmSecurity.TpmSecurityClient(undefined, new tssJs.Tpm(true));
    ```

1. Добавьте следующую функцию, чтобы реализовать метод:
   
    ```
    myTpm.getEndorsementKey(function(err, endorsementKey) {
      if (err) {
        console.log('The error returned from get key is: ' + err);
      } else {
        console.log('the endorsement key is: ' + endorsementKey.toString('base64'));
        myTpm.getRegistrationId((getRegistrationIdError, registrationId) => {
          if (getRegistrationIdError) {
            console.log('The error returned from get registration id is: ' + getRegistrationIdError);
          } else {
            console.log('The Registration Id is: ' + registrationId);
            process.exit();
          }
        });
      }
    });
    ```

1. Сохраните файл **ExtractDevice.js** и закройте его. Запустите пример:

    ```cmd/sh
    node ExtractDevice.js
    ```

1. Окно выходных данных отображает **_ключ подтверждения_** и **_идентификатор регистрации_** , необходимые для регистрации устройства. Запишите эти значения. 


## <a name="create-a-device-entry"></a>Создание записи устройства

Служба подготовки устройств Интернета вещей Azure поддерживает два типа регистрации:

- [Группы регистрации](concepts-service.md#enrollment-group). Используются для регистрации нескольких связанных устройств.
- [Индивидуальные регистрации.](concepts-service.md#individual-enrollment) Предназначены для регистрации одного устройства.

В этой статье описана индивидуальная регистрация.

1. Войдите на портал Azure, нажмите кнопку **Все ресурсы** в меню слева и откройте службу подготовки устройств.

1. В меню службы подготовки устройств выберите **Управление регистрациями**. Выберите вкладку **Индивидуальные регистрации** и нажмите кнопку **Добавить индивидуальную регистрацию** вверху. 

1. На панели **Добавление регистрации** введите следующие сведения:
   - Выберите **доверенный платформенный модуль (TPM)** как *механизм* аттестации удостоверения.
   - Введите записанные ранее *идентификатор регистрации* и *ключ подтверждения* для вашего устройства доверенного платформенного модуля.
   - Выберите Центр Интернета вещей, связанный с вашей службой подготовки.
   - При необходимости можно указать следующие сведения:
       - Укажите уникальный *идентификатор устройства*. Убедитесь, что при назначении имен устройства не используются конфиденциальные данные. Если вы решите его не предоставлять, вместо него для идентификации устройства будет использоваться идентификатор регистрации.
       - Обновите **начальное состояние двойника устройства**, используя требуемую начальную конфигурацию для устройства.
   - По завершении нажмите кнопку **Сохранить**. 

     ![Ввод сведений о регистрации устройства в колонке портала](./media/quick-create-simulated-device/enter-device-enrollment.png)  

   После успешной регистрации *идентификатор регистрации* устройства отобразится в списке под вкладкой *Индивидуальные регистрации*. 


## <a name="register-the-device"></a>Регистрация устройства

1. На портале Azure выберите колонку **Обзор** службы подготовки устройств и запишите значения для **_глобальной конечной точки устройства_** и **_области идентификатора_** .

    ![Извлеките сведения о конечной точке службы подготовки устройств из колонки на портале](./media/quick-create-simulated-device/extract-dps-endpoints.png) 

1. В текстовом редакторе создайте файл **RegisterDevice.js** в папке **registerdevice**.

1. Добавьте следующие инструкции `require` в начало файла **RegisterDevice.js**:
   
    ```
    'use strict';

    var ProvisioningTransport = require('azure-iot-provisioning-device-http').Http;
    var iotHubTransport = require('azure-iot-device-mqtt').Mqtt;
    var Client = require('azure-iot-device').Client;
    var Message = require('azure-iot-device').Message;
    var tpmSecurity = require('azure-iot-security-tpm');
    var ProvisioningDeviceClient = require('azure-iot-provisioning-device').ProvisioningDeviceClient;
    ```

    > [!NOTE]
    > **Пакет SDK Центра Интернета вещей Azure для Node.js** поддерживает дополнительные протоколы, такие как _AMQP_, _AMQP WS_ и _MQTT WS_.  Дополнительные примеры на Node.js см. в разделе [для пакета SDK службы подготовки устройств](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning/device/samples).
    > 

1. Добавьте переменные **globalDeviceEndpoint** и **idScope** и используйте их для создания экземпляра **ProvisioningDeviceClient**. Замените **{globalDeviceEndpoint}** и **{idScope}** значениями **_глобальной конечной точки устройства_** и **_идентификатором области_** из **шага 1**:
   
    ```
    var provisioningHost = '{globalDeviceEndpoint}';
    var idScope = '{idScope}';

    var tssJs = require("tss.js");
    var securityClient = new tpmSecurity.TpmSecurityClient('', new tssJs.Tpm(true));
    // if using non-simulated device, replace the above line with following:
    //var securityClient = new tpmSecurity.TpmSecurityClient();

    var provisioningClient = ProvisioningDeviceClient.create(provisioningHost, idScope, new ProvisioningTransport(), securityClient);
    ```

1. Добавьте следующую функцию, чтобы реализовать метод на устройстве:
   
    ```
    provisioningClient.register(function(err, result) {
      if (err) {
        console.log("error registering device: " + err);
      } else {
        console.log('registration succeeded');
        console.log('assigned hub=' + result.registrationState.assignedHub);
        console.log('deviceId=' + result.registrationState.deviceId);
        var tpmAuthenticationProvider = tpmSecurity.TpmAuthenticationProvider.fromTpmSecurityClient(result.registrationState.deviceId, result.registrationState.assignedHub, securityClient);
        var hubClient = Client.fromAuthenticationProvider(tpmAuthenticationProvider, iotHubTransport);

        var connectCallback = function (err) {
          if (err) {
            console.error('Could not connect: ' + err.message);
          } else {
            console.log('Client connected');
            var message = new Message('Hello world');
            hubClient.sendEvent(message, printResultFor('send'));
          }
        };

        hubClient.open(connectCallback);

        function printResultFor(op) {
          return function printResult(err, res) {
            if (err) console.log(op + ' error: ' + err.toString());
            if (res) console.log(op + ' status: ' + res.constructor.name);
            process.exit(1);
          };
        }
      }
    });
    ```

1. Сохраните файл **RegisterDevice.js** и закройте его. Запустите пример:

    ```cmd/sh
    node RegisterDevice.js
    ```

1. Обратите внимание на сообщения, которые имитируют загрузку устройства и его подключение к службе подготовки устройств для получения информации Центра Интернета вещей. После подготовки имитированного устройства для центра Интернета вещей, подключенного к службе подготовки, идентификатор устройства отобразится в колонке **Устройства Интернета вещей**. 

    ![Устройство зарегистрировано в Центре Интернета вещей](./media/quick-create-simulated-device/hub-registration.png) 

    Если в записи регистрации для своего устройства вы изменили значение по умолчанию для *начального состояния двойника устройства*, требуемое состояние двойника будет извлечено из концентратора с последующим выполнением соответствующих действий. См. [общие сведения о двойниках устройств и их использовании в Центре Интернета вещей](../iot-hub/iot-hub-devguide-device-twins.md).


## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете продолжить работу с примером клиентского устройства, не удаляйте ресурсы, созданные в ходе работы с этим кратким руководством. Если вы не планируете продолжать работу, следуйте инструкциям ниже, чтобы удалить все созданные ресурсы.

1. Закройте окно выходных данных примера клиентского устройства на компьютере.
1. Закройте окно симулятора доверенного платформенного модуля на компьютере.
1. В меню слева на портале Azure щелкните **Все ресурсы** и откройте службу подготовки устройств. Откройте колонку **Управление регистрациями** для службы, а затем щелкните вкладку **Индивидуальные регистрации**. Установите флажок рядом с *идентификатором регистрации* устройства, которое вы зарегистрировали в рамках этого краткого руководства, и нажмите кнопку **Удалить** в верхней части панели. 
1. В меню слева на портале Azure щелкните **Все ресурсы** и выберите свой центр Интернета вещей. Откройте колонку **Устройства Интернета вещей** для нужного концентратора, установите флажок *Идентификатор устройства*, зарегистрированного в процессе работы с кратким руководством, и нажмите кнопку **Удалить** в верхней части панели.


## <a name="next-steps"></a>Дальнейшие действия

Вы создали имитированное устройство доверенного платформенного модуля на компьютере и подготовили его для центра Интернета вещей с помощью Службы подготовки устройств к добавлению в Центр Интернета вещей. Чтобы узнать, как программными средствами зарегистрировать устройство доверенного платформенного модуля, изучите соответствующее краткое руководство. 

> [!div class="nextstepaction"]
> [Краткое руководство. Регистрация устройств TPM в службе подготовки устройств Центра Интернета вещей с помощью пакета SDK для службы Java](quick-enroll-device-tpm-node.md)