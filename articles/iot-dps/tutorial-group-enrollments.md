---
title: Учебник. Подготовка имитированного устройства X.509 в Центре Интернета вещей Azure с помощью Java и групп регистрации
description: Работая с этим учебником, вы создадите и подготовите имитированное устройство X.509 с помощью пакета SDK Java для службы и устройства, а также групп регистрации для Службы подготовки устройств к добавлению в Центр Интернета вещей (DPS).
author: wesmc7777
ms.author: wesmc
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.devlang: java
ms.custom: mvc, devx-track-java
ms.openlocfilehash: 4cfbfe3e3e3ba620d8292767012c9bb866d8a878
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94968100"
---
# <a name="tutorial-create-and-provision-a-simulated-x509-device-using-java-device-and-service-sdk-and-group-enrollments-for-iot-hub-device-provisioning-service"></a>Руководство. Создание и подготовка имитированного устройства X.509 с помощью пакета SDK Java для службы и устройства, а также с помощью групп регистрации для Службы подготовки устройств к добавлению в Центр Интернета вещей

В этом руководстве показано, как имитировать устройство X.509 на компьютере разработки под управлением ОС Windows, а также как с помощью примера кода подключить имитированное устройство к службе подготовки устройств и Центру Интернета вещей, используя группы регистрации. 

Прежде чем продолжить, выполните инструкции по [настройке службы подготовки устройств Центра Интернета вещей на портале Azure](./quick-setup-auto-provision.md).


## <a name="prerequisites"></a>Предварительные условия

1. Убедитесь, что на вашем компьютере установлен [пакет SDK 8 для Java SE](/azure/developer/java/fundamentals/java-jdk-long-term-support).

1. Скачайте и установите [Maven](https://maven.apache.org/install.html).

1. Установите на компьютер систему `git` и добавьте ее в переменные среды, доступные в командном окне. Последнюю версию средств `git` для установки, которая включает **Git Bash**, приложение командной строки для взаимодействия с локальным репозиторием Git, можно найти на [этой странице](https://git-scm.com/download/). 

1. Для создания тестовых сертификатов воспользуйтесь данными из [обзора сертификата](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md).

    > [!NOTE]
    > На этом шаге необходим [OpenSSL](https://www.openssl.org/), который можно либо создать и установить из источника, либо скачать и установить из [сторонних ресурсов](https://wiki.openssl.org/index.php/Binaries), например [из этого](https://sourceforge.net/projects/openssl/). Если _корневой_, _промежуточный_ сертификаты и сертификат _устройства_ уже созданы, то этот шаг можно пропустить.
    >

    1. Выполните первые два шага, чтобы создать _корневой_ и _промежуточный_ сертификаты.

    1. Войдите на портал Azure, нажмите кнопку **Все ресурсы** в меню слева и откройте службу подготовки.

        1. В колонке сводной информации о службе подготовки устройств выберите **Сертификаты** и нажмите кнопку **Добавить** в верхней части экрана.

        1. В разделе **Добавление сертификата** введите следующие данные.
            - Введите уникальное имя сертификата.
            - Выберите созданный файл **_RootCA.pem_**.
            - Затем нажмите кнопку **Сохранить**.

           ![Добавление сертификата](./media/tutorial-group-enrollments/add-certificate.png)

        1. Выберите только что созданный сертификат.
            - Щелкните **Создать код проверки**. Скопируйте созданный код.
            - Выполните этап проверки. Введите _код проверки_ или щелкните правой кнопкой мыши в открытом окне PowerShell, чтобы вставить код.  Нажмите клавишу **ВВОД**.
            - На портале Azure выберите созданный файл **_verifyCert4.pem_**. Нажмите кнопку **Проверка**.

              ![Проверка сертификата](./media/tutorial-group-enrollments/validate-certificate.png)

    1. Выполните шаги по созданию сертификатов для устройства и удалению ресурсов.

       > [!NOTE]
       > Создавая сертификаты устройств убедитесь, что в имени устройства используются только строчные буквы, цифры и дефисы.
       >


## <a name="create-a-device-enrollment-entry"></a>Создание записи о регистрации устройства

1. Откройте командную строку. Клонируйте репозиторий GitHub для образцов кода пакета SDK для Java:

    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-java.git --recursive
    ```

1. В загруженном исходном коде найдите папку **_azure-iot-sdk-java/provisioning/provisioning-samples/service-enrollment-group-sample_** . Откройте файл **_/src/main/java/samples/com/microsoft/azure/sdk/iot/ServiceEnrollmentGroupSample.java_** в любом удобном текстовом редакторе и добавьте следующие данные:

    1. В службу подготовки добавьте `[Provisioning Connection String]` с портала следующим образом:

        1. Откройте службу подготовки на [портале Azure](https://portal.azure.com).

        1. Откройте раздел **Политики общего доступа** и выберите политику с разрешением *EnrollmentWrite*.

        1. Скопируйте **строку подключения первичного ключа**.

            ![Получение строки подключения к службе подготовки на портале](./media/tutorial-group-enrollments/provisioning-string.png)  

        1. В файле с примером кода **_ServiceEnrollmentGroupSample.java_** замените `[Provisioning Connection String]` значением **строки подключения первичного ключа**.

            ```java
            private static final String PROVISIONING_CONNECTION_STRING = "[Provisioning Connection String]";
            ```

    1. Откройте в текстовом редакторе файл промежуточного сертификата для подписи. Обновите значение `PUBLIC_KEY_CERTIFICATE_STRING` с помощью значения из вашего промежуточного сертификата для подписи.

        Если вы создали сертификаты ваших устройств с помощью оболочки Bash, ключ промежуточного сертификата содержится в *./certs/azure-iot-test-only.intermediate.cert.pem*. Если ваши сертификаты были созданы с помощью PowerShell, то ваш файл промежуточного сертификата — *./Intermediate1.pem*.

        ```java
        private static final String PUBLIC_KEY_CERTIFICATE_STRING =
                "-----BEGIN CERTIFICATE-----\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "-----END CERTIFICATE-----\n";
        ```

    1. На [портале Azure](https://portal.azure.com) откройте Центр Интернета вещей, связанный с используемой службой подготовки. Откройте для этого центра вкладку **Обзор** и скопируйте значение **Hostname**. Присвойте значение **Hostname** параметру *IOTHUB_HOST_NAME*.

        ```java
        private static final String IOTHUB_HOST_NAME = "[Host name].azure-devices.net";
        ```

    1. Изучите пример кода. Этот код создает, обновляет и удаляет группу регистрации для устройств X.509. Чтобы проверить успешность регистрации на портале, временно закомментируйте в конце файла _ServiceEnrollmentGroupSample.java_ следующие строки кода:

        ```java
        // ************************************** Delete info of enrollmentGroup ***************************************
        System.out.println("\nDelete the enrollmentGroup...");
        provisioningServiceClient.deleteEnrollmentGroup(enrollmentGroupId);
        ```

    1. Сохраните файл _ServiceEnrollmentGroupSample.java_.

1. Откройте окно командной строки и перейдите к папке **_azure-iot-sdk-java/provisioning/provisioning-samples/service-enrollment-group-sample_** .

1. Соберите пример кода с помощью следующей команды:

    ```cmd\sh
    mvn install -DskipTests
    ```

1. Запустите пример кода, выполнив в окне командной строки следующие команды:

    ```cmd\sh
    cd target
    java -jar ./service-enrollment-group-sample-{version}-with-deps.jar
    ```

1. Удостоверьтесь, что регистрация прошла успешно, просмотрев данные в окне вывода.

    ![Регистрация прошла успешно](./media/tutorial-group-enrollments/enrollment.png) 

1. Откройте службу подготовки на портале Azure. Щелкните **Управление регистрациями**. Убедитесь, что на вкладке **Группы регистрации** появилась группа устройств X.509 с автоматически созданным *именем группы*.

## <a name="simulate-the-device"></a>Имитация устройства

1. В колонке сводной информации о службе подготовки устройств выберите **Обзор** и запишите указанные там значения _Область идентификатора_ и _Provisioning Service Global Endpoint_ (Глобальная конечная точка службы подготовки).

    ![Сведения о службе](./media/tutorial-group-enrollments/extract-dps-endpoints.png)

1. Откройте командную строку. Перейдите к папке с примером проекта.

    ```cmd/sh
    cd azure-iot-sdk-java/provisioning/provisioning-samples/provisioning-X509-sample
    ```

1. Измените `/src/main/java/samples/com/microsoft/azure/sdk/iot/ProvisioningX509Sample.java`, чтобы добавить ваши _область идентификатора_ и _глобальную конечную точку службы подготовки_, которые вы упоминали выше.

    ```java
    private static final String idScope = "[Your ID scope here]";
    private static final String globalEndpoint = "[Your Provisioning Service Global Endpoint here]";
    private static final ProvisioningDeviceClientTransportProtocol PROVISIONING_DEVICE_CLIENT_TRANSPORT_PROTOCOL = ProvisioningDeviceClientTransportProtocol.HTTPS;
    private static final int MAX_TIME_TO_WAIT_FOR_REGISTRATION = 10000; // in milli seconds
    private static final String leafPublicPem = "<Your Public PEM Certificate here>";
    private static final String leafPrivateKey = "<Your Private PEM Key here>";
    ```

1. Обновите переменные `leafPublicPem` и `leafPrivateKey`, используя открытый и закрытый сертификаты вашего устройства.

    Если вы создали сертификаты вашего устройства с помощью PowerShell, файлы mydevice* содержат открытый ключ, закрытый ключ и PFX-файл для устройства.

    Если вы создали сертификаты вашего устройства с помощью оболочки Bash, то открытый ключ находится в ./certs/new-device.cert.pem. Закрытый ключ будет находиться в файле ./private/new-device.key.pem.

    Откройте ваш файл с открытым ключом и обновите переменную `leafPublicPem`, используя это значение. Скопируйте текст от строки _-----BEGIN PRIVATE KEY-----_ по строку _-----END PRIVATE KEY-----_.

    ```java
    private static final String leafPublicPem = "-----BEGIN CERTIFICATE-----\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "-----END CERTIFICATE-----\n";
    ```

    Откройте ваш файл с закрытым ключом и обновите переменную `leafPrivatePem`, используя это значение. Скопируйте текст от строки _-----BEGIN RSA PRIVATE KEY-----_ по строку _-----END RSA PRIVATE KEY-----_.

    ```java
    private static final String leafPrivateKey = "-----BEGIN RSA PRIVATE KEY-----\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "-----END RSA PRIVATE KEY-----\n";
    ```

1. Для получения промежуточного сертификата добавьте новую переменную под `leafPrivateKey`. Назовите ее `intermediateKey`. Присвойте ей значение из вашего промежуточного сертификата для подписи.

    Если вы создали сертификаты ваших устройств с помощью оболочки Bash, ключ промежуточного сертификата содержится в *./certs/azure-iot-test-only.intermediate.cert.pem*. Если ваши сертификаты были созданы с помощью PowerShell, то ваш файл промежуточного сертификата — *./Intermediate1.pem*.

    ```java
    private static final String intermediateKey = "-----BEGIN CERTIFICATE-----\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "-----END CERTIFICATE-----\n";
    ```

1. В функции `main` до инициализации `securityProviderX509` добавьте `intermediateKey` к коллекции `signerCertificates`.

    ```java
    public static void main(String[] args) throws Exception
    {
        ...

        try
        {
            ProvisioningStatus provisioningStatus = new ProvisioningStatus();

            // Add intermediate certificate as part of the certificate key chain.
            signerCertificates.add(intermediateKey);

            SecurityProvider securityProviderX509 = new SecurityProviderX509Cert(leafPublicPem, leafPrivateKey, signerCertificates);
    ```

1. Сохраните изменения и выполните сборку примера. Перейдите к целевой папке и запустите созданный JAR-файл.

    ```cmd/sh
    mvn clean install
    cd target
    java -jar ./provisioning-x509-sample-{version}-with-deps.jar
    ```

    ![Успешная регистрация](./media/tutorial-group-enrollments/registration.png)

1. На портале перейдите в Центр Интернета вещей, связанный со службой подготовки, и откройте колонку **Обозреватель устройств**. Когда имитированное устройство X.509 будет успешно подготовлено для центра, в колонке **Обозреватель устройств** появится идентификатор этого устройства со значением **Включено** в столбце *Состояние*. Обратите внимание, что может потребоваться нажать кнопку **Обновить** в верхней части окна, если вы уже открывали колонку, прежде чем запустить пример приложения для устройства. 

    ![Устройство зарегистрировано в Центре Интернета вещей](./media/tutorial-group-enrollments/hub-registration.png) 


## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете продолжить работу с примером клиентского устройства, не удаляйте ресурсы, созданные в ходе работы с этим руководством. Если вы не планируете продолжать работу, следуйте инструкциям ниже, чтобы удалить все созданные ресурсы.

1. Закройте окно выходных данных примера клиентского устройства на компьютере.
1. В меню слева на портале Azure щелкните **Все ресурсы** и откройте службу подготовки устройств. Откройте колонку **Управление регистрациями** для службы, затем откройте вкладку **Индивидуальные регистрации**. Выберите *идентификатор регистрации* устройства, которое вы зарегистрировали в процессе работы с этим руководством, и нажмите кнопку **Удалить** вверху. 
1. В меню слева на портале Azure нажмите кнопку **Все ресурсы** и выберите свой Центр Интернета вещей. Откройте колонку **Устройства Интернета вещей** для нужного концентратора, выберите *идентификатор устройства*, зарегистрированного в процессе работы с руководством, и нажмите кнопку **Удалить** вверху.


## <a name="next-steps"></a>Дальнейшие действия

В ходе работы с этим руководством вы создали имитированное устройство X.509 на компьютере с ОС Windows и подготовили его для Центра Интернета вещей с помощью службы подготовки устройств Центра Интернета вещей и групп регистрации. Дополнительные сведения об устройстве X.509 см. в статье с основными понятиями. 

> [!div class="nextstepaction"]
> [Понятия службы подготовки устройств для Центра Интернета вещей](concepts-service.md)