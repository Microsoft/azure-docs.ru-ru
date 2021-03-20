---
title: Подключение устройств с помощью сертификатов X. 509 в приложении IoT Central Azure
description: Как подключить устройства с сертификатами X. 509 с помощью пакета SDK для Node.js устройств для IoT Central приложения
author: dominicbetts
ms.author: dobett
ms.date: 08/12/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom: device-developer
ms.openlocfilehash: cf0db71600c9350b4d70e6375f509a6e88709f70
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100378338"
---
# <a name="how-to-connect-devices-with-x509-certificates-using-nodejs-device-sdk-for-iot-central-application"></a>Как подключить устройства с сертификатами X. 509 с помощью пакета SDK для Node.js устройств для IoT Central приложения

IoT Central поддерживает подписанные URL-адрес (SAS) и сертификаты X. 509 для защиты обмена данными между устройством и приложением. В руководстве по [созданию и подключению клиентского приложения к IOT Centralному приложению Azure](./tutorial-connect-device.md) используется SAS. Из этой статьи вы узнаете, как изменить пример кода для использования X. 509.  Сертификаты X. 509 рекомендуется использовать в рабочих средах. Дополнительные сведения см. [в статье подключение к Azure IOT Central](./concepts-get-connected.md).

В этой статье описаны два способа использования [регистраций](how-to-connect-devices-x509.md#use-a-group-enrollment) X. 509-Group, обычно используемых в рабочей среде, и [индивидуальных регистраций](how-to-connect-devices-x509.md#use-an-individual-enrollment) , которые можно использовать для тестирования.

В фрагментах кода в этой статье используется JavaScript. Примеры кода на других языках см. в следующих статьях:

- [В](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples/iothub_ll_client_x509_sample)
- [C#](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/iot-hub/Samples/device/X509DeviceCertWithChainSample)
- [Java](https://github.com/Azure/azure-iot-sdk-java/tree/master/device/iot-device-samples/send-event-x509)
- [Python](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/sync-samples)

## <a name="prerequisites"></a>Предварительные условия

- Завершение [создания и подключения клиентского приложения к руководству по использованию приложения IOT Central Azure (JavaScript)](./tutorial-connect-device.md) .
- [Git](https://git-scm.com/download/).
- Скачайте и установите [OpenSSL](https://www.openssl.org/). Если вы используете Windows, вы можете использовать двоичные файлы на [странице OpenSSL в SourceForge](https://sourceforge.net/projects/openssl/).

## <a name="use-a-group-enrollment"></a>Использование групповой регистрации

Используйте сертификаты X. 509 с групповой регистрацией в рабочей среде. При регистрации группы вы добавляете корневой или промежуточный сертификат X. 509 в приложение IoT Central. Устройства с конечными сертификатами, полученными из корневого или промежуточного сертификата, могут подключаться к приложению.

## <a name="generate-root-and-device-cert"></a>Создание корня и сертификата устройства

В этом разделе вы используете сертификат X. 509 для подключения устройства с сертификатом, полученным от сертификата группы регистрации, который может подключиться к приложению IoT Central.

> [!WARNING]
> Такой способ создания сертификатов X. 509 предназначен только для тестирования. Для рабочей среды следует использовать официальный, безопасный механизм создания сертификатов.

1. Откройте командную строку. Клонировать репозиторий GitHub для скриптов создания сертификатов:

    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-node.git
    ```

1. Перейдите к скрипту генератора сертификатов и установите необходимые пакеты:

    ```cmd/sh
    cd azure-iot-sdk-node/provisioning/tools
    npm install
    ```

1. Создайте корневой сертификат, а затем выберете сертификат устройства, выполнив следующий сценарий:

    ```cmd/sh
    node create_test_cert.js root mytestrootcert
    node create_test_cert.js device sample-device-01 mytestrootcert
    ```

    > [!TIP]
    > Код устройства может содержать только буквы, цифры и символ `-`.

Эти команды создают по три файла для корня и сертификата устройства.

имя_файла | содержимое
-------- | --------
\<name\>_cert PEM | Открытая часть сертификата X509.
\<name\>_key PEM | Закрытый ключ для сертификата X509
\<name\>_fullchain PEM | Вся цепочка ключей для сертификата X509.

## <a name="create-a-group-enrollment"></a>Создание группы регистрации

1. Откройте приложение IoT Central и перейдите к элементу **Администрирование**  в левой области и выберите **Подключение устройства**.

1. Выберите **+ создать группу регистрации** и создайте новую группу регистрации с именем _MyX509Group_ с типом аттестации **Certificates (X. 509)**.

1. Откройте созданную группу регистрации и выберите **Управление первичными**.

1. Выберите параметр файл и отправьте файл корневого сертификата с именем _mytestrootcert_cert. pem_ , созданный ранее.

    ![Отправка сертификата](./media/how-to-connect-devices-x509/certificate-upload.png)

1. Чтобы завершить проверку, создайте код проверки, скопируйте его, а затем используйте его для создания сертификата проверки X. 509 в командной строке:

    ```cmd/sh
    node create_test_cert.js verification --ca mytestrootcert_cert.pem --key mytestrootcert_key.pem --nonce  {verification-code}
    ```

1. Передайте подписанный сертификат проверки _verification_cert. pem_ , чтобы завершить проверку:

    ![Проверенный сертификат](./media/how-to-connect-devices-x509/verified.png)

Теперь вы можете подключать устройства, имеющие сертификат X. 509, производный от этого основного корневого сертификата.

После сохранения группы регистрации запишите область ИДЕНТИФИКАТОРов.

## <a name="run-sample-device-code"></a>Запуск примера кода устройства

1. Скопируйте файлы **sampleDevice01_key. pem** и **sampleDevice01_cert. pem** в папку _Azure-IOT-SDK-node/Device/Samples/pnp_ , которая содержит приложение **simple_thermostat.js** . Это приложение использовалось при завершении [учебника подключение устройства (JavaScript)](./tutorial-connect-device.md).

1. Перейдите в папку _Azure-IOT-SDK-node/Device/Samples/PnP_ , содержащую приложение **simple_thermostat.js** , и выполните следующую команду, чтобы установить пакет X. 509:

    ```cmd/sh
    npm install azure-iot-security-x509 --save
    ```

1. Откройте файл **simple_thermostat.js** в текстовом редакторе.

1. Измените `require` инструкции, включив в них следующее:

    ```javascript
    const fs = require('fs');
    const X509Security = require('azure-iot-security-x509').X509Security;
    ```

1. Добавьте следующие четыре строки в раздел "сведения о подключении DPS", чтобы инициализировать `deviceCert` переменную:

    ```javascript
    const deviceCert = {
      cert: fs.readFileSync(process.env.IOTHUB_DEVICE_X509_CERT).toString(),
      key: fs.readFileSync(process.env.IOTHUB_DEVICE_X509_KEY).toString()
    };
    ```

1. Измените `provisionDevice` функцию, которая создает клиент, заменив первую строку следующим:

    ```javascript
    var provSecurityClient = new X509Security(registrationId, deviceCert);
    ```

1. В той же функции измените строку, которая задает `deviceConnectionString` переменную, следующим образом:

    ```javascript
    deviceConnectionString = 'HostName=' + result.assignedHub + ';DeviceId=' + result.deviceId + ';x509=true';
    ```

1. В `main` функции добавьте следующую строку после строки, которая вызывает `Client.fromConnectionString` :

    ```javascript
    client.setOptions(deviceCert);
    ```

1. В среде оболочки задайте следующие две переменные среды:

    ```cmd/sh
    set IOTHUB_DEVICE_X509_CERT=sampleDevice01_cert.pem
    set IOTHUB_DEVICE_X509_KEY=sampleDevice01_key.pem
    ```

    > [!TIP]
    > Другие необходимые переменные среды задаются при выполнении учебника [Создание и подключение клиентского приложения к IOT Centralному приложению Azure](./tutorial-connect-device.md) .

1. Выполните сценарий и убедитесь, что устройство успешно подготовлено:

    ```cmd/sh
    node simple_thermostat.js
    ```

    Вы также можете проверить, отображается ли телеметрия на панели мониторинга.

    ![Проверка телеметрии устройства](./media/how-to-connect-devices-x509/telemetry.png)

## <a name="use-an-individual-enrollment"></a>Использование индивидуальной регистрации

Используйте сертификаты X. 509 с индивидуальной регистрацией для тестирования устройства и решения. В индивидуальной регистрации отсутствует корневой или промежуточный сертификат X. 509 в приложении IoT Central. Устройства используют самозаверяющий сертификат X. 509 для подключения к приложению.

## <a name="generate-self-signed-device-cert"></a>Создание самозаверяющего сертификата устройства

В этом разделе вы используете самозаверяющий сертификат X. 509 для подключения устройств к отдельной регистрации, которая используется для регистрации одного устройства. Самозаверяющие сертификаты предназначены только для тестирования.

Создайте самозаверяющий сертификат устройства X. 509, выполнив сценарий. Используйте для имени сертификата только буквы в нижнем регистре и дефисы:

  ```cmd/sh
    cd azure-iot-sdk-node/provisioning/tools
    node create_test_cert.js device mytestselfcertprimary
    node create_test_cert.js device mytestselfcertsecondary 
  ```

## <a name="create-individual-enrollment"></a>Создание отдельной регистрации

1. В приложении IoT Central Azure выберите **устройства** и создайте новое устройство с **идентификатором устройства** как _митестселфцертпримари_ из шаблона устройства термостата. Запишите **область идентификаторов**, которая будет использоваться позже.

1. Откройте созданное устройство и нажмите кнопку **подключить**.

1. Выберите **отдельные регистрации** в качестве **метода подключения** и **сертификатов (X. 509)** в качестве механизма:

    ![Отдельная регистрация](./media/how-to-connect-devices-x509/individual-device-connect.png)

1. Выберите параметр файл в разделе основная и отправьте файл сертификата с именем _mytestselfcertprimary_cert. pem_ , который был создан ранее.

1. Выберите параметр файл для дополнительного сертификата и отправьте файл сертификата с именем _mytestselfcertsecondary_cert. PEM._ Затем выберите **сохранить**:

    ![Отправка сертификата отдельной регистрации](./media/how-to-connect-devices-x509/individual-enrollment.png)

Теперь устройство подготовлено с помощью сертификата X. 509.

## <a name="run-a-sample-individual-enrollment-device"></a>Запуск примера отдельного устройства регистрации

1. Скопируйте файлы _mytestselfcertprimary_key. pem_ и _mytestselfcertprimary_cert. pem_ в папку _Azure-IOT-SDK-node/Device/Samples/pnp_ , которая содержит приложение **simple_thermostat.js** . Это приложение использовалось при завершении [учебника подключение устройства (JavaScript)](./tutorial-connect-device.md).

1. Измените переменные среды, использованные в примере выше, следующим образом:

    ```cmd/sh
    set IOTHUB_DEVICE_DPS_DEVICE_ID=mytestselfcertprimary
    set IOTHUB_DEVICE_X509_CERT=mytestselfcertprimary_cert.pem
    set IOTHUB_DEVICE_X509_KEY=mytestselfcertprimary_key.pem
    ```

1. Выполните сценарий и убедитесь, что устройство успешно подготовлено:

    ```cmd/sh
    node environmentalSensor.js
    ```

    Вы также можете проверить, отображается ли телеметрия на панели мониторинга.

    ![Индивидуальная регистрация телеметрии](./media/how-to-connect-devices-x509/telemetry-primary.png)

Вы также можете повторить описанные выше действия для сертификата _митестселфцертсекондари_ .

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как подключать устройства с помощью сертификатов X. 509, предлагаем следующий шаг: Узнайте, как [отслеживать подключение устройств с помощью Azure CLI](howto-monitor-devices-azure-cli.md)
