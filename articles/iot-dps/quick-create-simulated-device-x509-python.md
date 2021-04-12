---
title: Краткое руководство. Подготовка имитированного устройства X.509 в Центре Интернета вещей Azure с помощью Python
description: Краткое руководство по созданию и подготовке имитированного устройства X.509 с помощью пакета SDK Python для устройств для Службы подготовки устройств к добавлению в Центр Интернета вещей (DPS). В этом кратком руководстве используется индивидуальная регистрация.
author: wesmc7777
ms.author: wesmc
ms.date: 01/29/2021
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.devlang: python
ms.custom: mvc, devx-track-python
ms.openlocfilehash: 5db02f8ca1f0c311617a787525ee2fa5856eb5dc
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105045269"
---
# <a name="quickstart-create-and-provision-a-simulated-x509-device-using-python-device-sdk-for-iot-hub-device-provisioning-service"></a>Краткое руководство. Создание и подготовка имитированного устройства X.509 с помощью пакета SDK службы устройства Python для службы подготовки устройств Центра Интернета вещей

[!INCLUDE [iot-dps-selector-quick-create-simulated-device-x509](../../includes/iot-dps-selector-quick-create-simulated-device-x509.md)]

При работе с этим кратким руководством вы подготовите компьютер разработки в качестве устройства Python X.509. Для подключения устройства к центру Интернета вещей применяется пример кода устройства из [пакета SDK Интернета вещей Azure для Python](https://github.com/Azure/azure-iot-sdk-python). В этом примере используется отдельная регистрация в Службе подготовки устройств (DPS).

## <a name="prerequisites"></a>Предварительные требования

- Ознакомьтесь с разделом [Роли и учетные записи Azure](about-iot-dps.md#provisioning-process).
- Выполнение инструкций из краткого руководства по [настройке Службы подготовки устройств к добавлению в Центр Интернета вещей на портале Azure](./quick-setup-auto-provision.md).
- Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [Python 3.6 или более поздней версии](https://www.python.org/downloads/).
- [Git](https://git-scm.com/download/).


[!INCLUDE [IoT Device Provisioning Service basic](../../includes/iot-dps-basic.md)]

## <a name="prepare-the-environment"></a>Подготовка среды 

1. Установите на компьютер систему `git` и добавьте ее в переменные среды, доступные в командном окне. Последнюю версию средств `git` для установки, которая включает **Git Bash**, приложение командной строки для взаимодействия с локальным репозиторием Git, можно найти на [этой странице](https://git-scm.com/download/). 

2. Откройте строку Git Bash. Клонируйте репозиторий GitHub с [пакетом SDK Интернета вещей Azure для Python](https://github.com/Azure/azure-iot-sdk-python).
    
    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-python.git --recursive
    ```


## <a name="create-a-self-signed-x509-device-certificate"></a>Создание самозаверяющего сертификата для устройства X.509 

При работе с этим разделом вы создадите самозаверяющий сертификат X.509. При этом стоит учесть следующее:

* Самозаверяющие сертификаты предназначены только для тестирования и не должны использоваться в рабочей среде.
* Срок действия самозаверяющего сертификата по умолчанию составляет один год.

Если у вас еще нет сертификатов для аутентификации устройства, можно создать самозаверяющий сертификат с помощью OpenSSL для тестирования в рамках работы с этой статьей.  OpenSSL включается в установку Git. 

1. Выполните приведенную ниже команду в командной строке Git Bash.

    # <a name="windows"></a>[Windows](#tab/windows)
    
    ```bash
    winpty openssl req -outform PEM -x509 -sha256 -newkey rsa:4096 -keyout ./python-device.key.pem -out ./python-device.pem -days 365 -extensions usr_cert -subj "//CN=Python-device-01"
    ```

    > [!IMPORTANT]
    > Добавлять к имени субъекта (`//CN=Python-device-01`) дополнительную косую черту нужно только для того, чтобы строка экранировалась с помощью Git на платформах Windows. 

    # <a name="linux"></a>[Linux](#tab/linux)
    
    ```bash
    openssl req -outform PEM -x509 -sha256 -newkey rsa:4096 -keyout ./python-device.key.pem -out ./python-device.pem -days 365 -extensions usr_cert -subj "/CN=Python-device-01"
    ```
    
    ---
    
2. При появлении запроса на **ввод парольной фразы PEM** используйте `1234` для тестирования в рамках работы с этой статьей.    

3. При повторном запросе на **ввод парольной фразы PEM для проверки** повторно используйте парольную фразу `1234`.    

Файл тестового сертификата (*python-device.pem*) и файл закрытого ключа (*python-device.key.pem*) создаются в каталоге, где была выполнена команда `openssl`.


## <a name="create-an-individual-enrollment-entry-in-dps"></a>Создание отдельной записи регистрации в DPS


Служба подготовки устройств Интернета вещей Azure поддерживает два типа регистрации:

- [Группы регистрации](concepts-service.md#enrollment-group). Используются для регистрации нескольких связанных устройств.
- [Индивидуальные регистрации.](concepts-service.md#individual-enrollment) Предназначены для регистрации одного устройства.

В этой статье описана отдельная регистрация для одного устройства, которое нужно подготовить с помощью центра Интернета вещей.

1. Войдите на портал Azure, нажмите кнопку **Все ресурсы** в меню слева и откройте службу подготовки.

2. В меню службы подготовки устройств выберите **Управление регистрациями**. Выберите вкладку **Индивидуальные регистрации** и нажмите кнопку **Добавить индивидуальную регистрацию** вверху. 

3. На панели **Добавление регистрации** введите следующие сведения:
   - Выберите **X.509** как *механизм* аттестации удостоверения.
   - В поле *PEM-файл или CER-файл первичного сертификата* щелкните элемент *Выбрать файл*, чтобы выбрать файл сертификата **python-device.pem**, если вы используете созданный ранее тестовый сертификат.
   - При необходимости можно указать следующие сведения:
     - Выберите Центр Интернета вещей, связанный с вашей службой подготовки.
     - Обновите **начальное состояние двойника устройства**, используя требуемую начальную конфигурацию для устройства.
   - По завершении нажмите кнопку **Сохранить**. 

     [![Добавление индивидуальной регистрации для аттестации X.509 на портале](./media/python-quick-create-simulated-device-x509/device-enrollment.png)](./media/python-quick-create-simulated-device-x509/device-enrollment.png#lightbox)

   После успешной регистрации устройство X.509 отобразится как **Python-device-01** в столбце *Идентификатор регистрации* на вкладке *Отдельные регистрации*. Это значение регистрации основано на имени субъекта в сертификате устройства. 

## <a name="simulate-the-device"></a>Имитация устройства

Пример подготовки Python [provision_x509.py](https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-device/samples/async-hub-scenarios/provision_x509.py) находится в каталоге `azure-iot-sdk-python/azure-iot-device/samples/async-hub-scenarios`. В этом примере для аутентификации и подготовки устройства Интернета вещей с помощью DPS используются шесть переменных среды. Ниже перечислены эти переменные среды:

| Имя переменной              | Описание                                     |
| :------------------------- | :---------------------------------------------- |
| `PROVISIONING_HOST`        |  Глобальная конечная точка, используемая для подключения к ресурсу DPS. |    
| `PROVISIONING_IDSCOPE`     |  Область идентификатора для ресурса DPS. |    
| `DPS_X509_REGISTRATION_ID` |  Идентификатор устройства. Значение должно совпадать с именем субъекта в сертификате устройства. |    
| `X509_CERT_FILE`           |  Имя файла сертификата вашего устройства. |    
| `X509_KEY_FILE`            |  Имя файла закрытого ключа для сертификата вашего устройства. |
| `PASS_PHRASE`              |  Парольная фраза, которая использовалась для шифрования файлов сертификата и закрытого ключа (`1234`). |    

1. В меню службы подготовки устройств выберите **Обзор**. Запишите значения параметров _Область идентификатора_ и _Глобальная конечная точка устройства_.

    ![Сведения о службе](./media/python-quick-create-simulated-device-x509/extract-dps-endpoints.png)

2. В командной строке Git Bash используйте приведенные ниже команды, чтобы добавить переменные среды для глобальной конечной точки устройства и области идентификатора.

    ```bash
    $export PROVISIONING_HOST=global.azure-devices-provisioning.net
    $export PROVISIONING_IDSCOPE=<ID scope for your DPS resource>
    ```

3. Идентификатор регистрации для устройства Интернета вещей должен совпадать с именем субъекта в сертификате устройства. Если вы создали самозаверяющий тестовый сертификат, `Python-device-01` будет именем субъекта и идентификатором регистрации для устройства. 

    Если у вас уже есть сертификат устройства, можно использовать `certutil`, чтобы проверить общее имя субъекта, используемое для устройства, как показано ниже для самозаверяющего тестового сертификата:

    ```bash
    $ certutil python-device.pem
    X509 Certificate:
    Version: 3
    Serial Number: fa33152fe1140dc8
    Signature Algorithm:
        Algorithm ObjectId: 1.2.840.113549.1.1.11 sha256RSA
        Algorithm Parameters:
        05 00
    Issuer:
        CN=Python-device-01
      Name Hash(sha1): 1dd88de40e9501fb64892b698afe12d027011000
      Name Hash(md5): a62c784820daa931b9d3977739b30d12
    
     NotBefore: 1/29/2021 7:05 PM
     NotAfter: 1/29/2022 7:05 PM
    
    Subject:
        ===> CN=Python-device-01 <===
      Name Hash(sha1): 1dd88de40e9501fb64892b698afe12d027011000
      Name Hash(md5): a62c784820daa931b9d3977739b30d12
    ```

    В командной строке Git Bash задайте переменную среды для идентификатора регистрации, как показано ниже:

    ```bash
    $export DPS_X509_REGISTRATION_ID=Python-device-01
    ```

4. В командной строке Git Bash задайте переменные среды для файла сертификата и файла закрытого ключа, а также парольную фразу.

    ```bash
    $export X509_CERT_FILE=./python-device.pem
    $export X509_KEY_FILE=./python-device.key.pem
    $export PASS_PHRASE=1234
    ```

5. Просмотрите код для [provision_x509.py](https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-device/samples/async-hub-scenarios/provision_x509.py). Если вы не используете **Python 3.7** или более поздней версии, внесите [описанные здесь изменения в код](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/async-hub-scenarios#advanced-iot-hub-scenario-samples-for-the-azure-iot-hub-device-sdk), чтобы заменить `asyncio.run(main())` и сохранить изменения. 

6. Запустите образец. В рамках этого примера будет создано подключение для устройства, а также выполнена подготовка устройства для центра и отправка в него нескольких тестовых сообщений.

    ```bash
    $ winpty python azure-iot-sdk-python/azure-iot-device/samples/async-hub-scenarios/provision_x509.py
    RegistrationStage(RequestAndResponseOperation): Op will transition into polling after interval 2.  Setting timer.
    The complete registration result is
    Python-device-01
    TestHub12345.azure-devices.net
    initialAssignment
    null
    Will send telemetry from the provisioned device
    sending message #4
    sending message #7
    sending message #2
    sending message #8
    sending message #5
    sending message #9
    sending message #1
    sending message #6
    sending message #10
    sending message #3
    done sending message #4
    done sending message #7
    done sending message #2
    done sending message #8
    done sending message #5
    done sending message #9
    done sending message #1
    done sending message #6
    done sending message #10
    done sending message #3
    ```

7. На портале перейдите в центр Интернета вещей, связанный со службой подготовки, и откройте колонку **Устройства Интернета вещей** в разделе **Обозреватели** в меню слева. Когда имитированное устройство X.509 будет успешно подготовлено для центра, в колонке **Обозреватель устройств** появится идентификатор этого устройства со значением **Включено** в столбце *Состояние*. Если вы уже открывали колонку, прежде чем запустить пример приложения для устройства, возможно, потребуется нажать кнопку **Обновить** вверху. 

    ![Устройство зарегистрировано в Центре Интернета вещей](./media/python-quick-create-simulated-device-x509/registration.png) 

> [!NOTE]
> Если в записи регистрации для своего устройства вы изменили значение по умолчанию для *начального состояния двойника устройства*, требуемое состояние двойника будет извлечено из концентратора с последующим выполнением соответствующих действий. См. [общие сведения о двойниках устройств и их использовании в Центре Интернета вещей](../iot-hub/iot-hub-devguide-device-twins.md).
>

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете продолжить работу с примером клиентского устройства, не удаляйте ресурсы, созданные в ходе работы с этим кратким руководством. Если вы не планируете продолжать работу, следуйте инструкциям ниже, чтобы удалить все созданные ресурсы.

1. Закройте окно выходных данных примера клиентского устройства на компьютере.
2. В меню слева на портале Azure щелкните **Все ресурсы** и откройте службу подготовки устройств. Откройте колонку **Управление регистрациями** для службы, а затем щелкните вкладку **Индивидуальные регистрации**. Установите флажок рядом с *идентификатором регистрации* устройства, которое вы зарегистрировали в рамках этого краткого руководства, и нажмите кнопку **Удалить** в верхней части панели. 
3. В меню слева на портале Azure щелкните **Все ресурсы** и выберите свой центр Интернета вещей. Откройте колонку **Устройства Интернета вещей** для нужного концентратора, установите флажок *Идентификатор устройства*, зарегистрированного в процессе работы с кратким руководством, и нажмите кнопку **Удалить** в верхней части панели.

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы создали имитированное устройство X.509 на компьютере разработки и подготовили его для центра Интернета вещей с помощью Службы подготовки устройств к добавлению в Центр Интернета вещей Azure на портале. Чтобы узнать, как выполнить программную регистрацию устройства X.509, изучите соответствующее краткое руководство. 

> [!div class="nextstepaction"]
> [Краткое руководство. Регистрация устройств X.509 в Службе подготовки устройств с помощью C#](quick-enroll-device-x509-python.md)
