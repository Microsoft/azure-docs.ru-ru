---
title: Пакет SDK для устройств Azure IoT для C | Документация Майкрософт
description: Начните работу с пакетом SDK для устройств Azure IoT для C и узнайте, как создавать приложения для устройств, взаимодействующие с Центром Интернета вещей.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: conceptual
ms.date: 05/17/2019
ms.author: robinsh
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 17b31e365e311b97e322828927827f40733313a6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97588834"
---
# <a name="azure-iot-device-sdk-for-c"></a>Пакет SDK для устройств Azure IoT для C

**Пакет SDK для устройств Azure IoT** — это набор библиотек, предназначенных для упрощения процесса отправки сообщений и их получения из службы **Центра Интернета вещей Azure**. Существуют различные варианты пакетов SDK, предназначенные для разных платформ. В этой статье описывается **пакет SDK для устройств Azure IoT для C**.

> [!NOTE]
> Встроенный пакет C SDK — это альтернатива ограниченным устройствам, поддерживающим подход "использовать собственную сеть" (БЙОН). Разработчики Интернета вещей имеют свободу выбора клиента MQTT, TLS и сокетов для создания решения устройства. Дополнительные [сведения о встроенном пакете C SDK](https://github.com/Azure/azure-sdk-for-c/tree/master/sdk/docs/iot).

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Пакет SDK для устройств Azure IoT для C написан на языке ANSI C (C99), что обеспечивает максимальную переносимость. Эта функция делает библиотеки хорошо подходящими для использования на нескольких платформах и устройствах, особенно в тех случаях, когда приоритетом является минимизация объема дискового пространства и памяти.

Этот пакет SDK протестирован на множестве платформ (подробные сведения см. в [каталоге устройств, сертифицированных по программе Microsoft Azure Certified for IoT](https://catalog.azureiotsolutions.com/)). В этой статье приведены пошаговые инструкции с примерами кода, выполняемого на платформе Windows. Для других поддерживаемых платформ код будет аналогичным.

В следующем видео представлен обзор пакета SDK Azure IoT для C:

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Azure-IoT-C-SDK-insights/Player]

Из этой статьи вы узнаете об архитектуре пакета SDK для устройств Azure IoT для C. Вы узнаете, как инициализировать библиотеку устройств, отправлять данные в Центр Интернета вещей и получать сообщения из него. Приведенной здесь информации достаточно, чтобы вы могли начать работу с пакетом SDK. Также вы найдете здесь ссылки для получения дополнительных сведений о библиотеках.

## <a name="sdk-architecture"></a>Архитектура пакета SDK

[**Пакет SDK для устройств Интернета вещей Azure для C**](https://github.com/Azure/azure-iot-sdk-c) доступен в репозитории на сайте GitHub. Дополнительные сведения об API см. в [справочной документации по API для C](/azure/iot-hub/iot-c-sdk-ref/).

Последнюю версию библиотек можно найти в **основной** ветви репозитория:

  ![Снимок экрана: Главная ветвь репозитория](./media/iot-hub-device-sdk-c-intro/RepoMasterBranch.png)

* Реализация ядра пакета SDK находится в папке **iothub\_client**, где содержится реализация самого нижнего уровня API в пакете SDK — библиотека **IoTHubClient**. Библиотека **IoTHubClient** содержит API-интерфейсы, реализующие обмен необработанными сообщениями для отправки сообщений в Центр Интернета вещей, а также получения сообщений из него. Если вы используете эту библиотеку, вам нужно самостоятельно реализовать сериализацию сообщений. Другие составляющие процесса взаимодействия с Центром Интернета вещей реализуются автоматически.

* Папка **serializer** содержит вспомогательные функции и примеры, демонстрирующие сериализацию данных перед их отправкой в Центр Интернета вещей Azure с помощью клиентской библиотеки. Сериализатор не обязателен для использования и предоставляется только для удобства. Чтобы использовать библиотеку **serializer**, необходимо определить модель, которая указывает данные, отправляемые в Центр Интернета вещей, а также сообщения, получаемые из него. После определения модели пакет SDK предоставит поверхность API, которая упрощает работу с сообщениями, передаваемыми из устройства в облако и наоборот, а также избавляет вас от необходимости выполнять задачи, связанные с сериализацией. Библиотека зависит от других библиотек с открытым исходным кодом, которые реализуют транспорт с помощью таких протоколов, как MQTT и AMQP.

* Библиотека **IoTHubClient** зависит от других библиотек с открытым исходным кодом:

  * [общей вспомогательной библиотеки Azure C](https://github.com/Azure/azure-c-shared-utility), которая предоставляет основные функции для решения базовых задач (включая манипуляции со строками и списками, ввод и вывод данных и т. д.), требуемых в различных связанных с Azure пакетах SDK для C;

  * библиотеки [Azure uAMQP](https://github.com/Azure/azure-uamqp-c), которая представляет собой реализацию AMQP на стороне клиента, оптимизированную для устройств с ограниченными ресурсами;

  * [Azure uMQTT](https://github.com/Azure/azure-umqtt-c) — библиотеки общего назначения, реализующей протокол MQTT и оптимизированной для устройств с ограниченными ресурсами.

Все это проще понять, взглянув на примеры кода. Разделы, представленные ниже, содержат описание нескольких примеров приложений, включенных в пакет SDK. Ознакомившись с данным руководством, вы получите представление о разных возможностях архитектурных уровней пакета и о работе с API.

## <a name="before-you-run-the-samples"></a>Что необходимо сделать, прежде чем выполнять примеры

Прежде чем выполнять примеры в пакете SDK устройств Azure IoT для C, необходимо [создать экземпляр службы Центра Интернета вещей](iot-hub-create-through-portal.md) в подписке Azure. Затем выполните следующие задачи:

* Подготовка среды разработки
* получение учетных данных устройства.

### <a name="prepare-your-development-environment"></a>Подготовка среды разработки

Пакеты предоставляются для общих платформ (например, NuGet для Windows или apt_get для Debian и Ubuntu) и по возможности используются в примерах. В некоторых случаях может потребоваться выполнить компиляцию пакета SDK для устройства. Дополнительные сведения о компиляции пакета SDK см. на [странице о подготовке среды разработки](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md) в репозитории GitHub.

Чтобы получить код примера приложения, скачайте копию пакета SDK из GitHub. Получите копию источника из **основной** ветви [репозитория GitHub](https://github.com/Azure/azure-iot-sdk-c).


### <a name="obtain-the-device-credentials"></a>Получение учетных данных устройства

После получения примера исходного кода вам необходимо получить набор учетных данных устройства. Чтобы обеспечить устройству доступ к Центру Интернета вещей, добавьте его в реестр удостоверений Центра Интернета вещей. Когда устройство будет добавлено, вы получите набор учетных данных, которые потребуются для подключения этого устройства к Центру Интернета вещей. Примеры приложений, которые мы рассмотрим в разделе ниже, предполагают использование этих учетных данных в виде **строки подключения устройства**.

Существует несколько средств с открытым кодом, помогающих управлять центром Интернета вещей.

* Приложение Windows, именуемое [Azure IOT Explorer](https://github.com/Azure/azure-iot-explorer).

* Кроссплатформенное расширение Visual Studio Code, которое называется [Средства Интернета вещей Azure](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).

* Кроссплатформенный интерфейс командной строки Python — [расширение Интернета вещей для Azure CLI](https://github.com/Azure/azure-iot-cli-extension).

В этом руководстве описывается использование графического *обозревателя устройств*. Если вы разрабатываете решения в VS Code, можно воспользоваться *Средствами Интернета вещей Azure для VS Code*. Можно также использовать *расширение IOT для средства Azure CLI 2,0* , если вы предпочитаете использовать средство CLI.

Обозреватель устройств использует библиотеки службы Интернета вещей Azure для выполнения различных операций в Центре Интернета вещей, включая добавление устройств. Если для добавления устройств вы используете обозреватель устройств, вы получите строку подключения для устройства. Эта строка подключения понадобится вам для запуска примеров приложений.

Если вы еще не знакомы с обозревателем устройств, изучите процедуру, представленную ниже, в которой описывается добавление устройства и получение строки подключения устройства.

1. Дополнительные сведения об установке обозревателя устройств см. на странице об [использовании обозревателя устройств для устройств Центра Интернета вещей](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/).

1. При выполнении программы вы увидите следующий интерфейс:

   ![Снимок экрана с двойником устройства в Device Explorer](./media/iot-hub-device-sdk-c-intro/DeviceExplorerTwinConfigTab.png)

1. Введите в первое поле **строку подключения Центра Интернета вещей** и нажмите кнопку **Обновить**. Теперь средство настроено для обмена данными с Центром Интернета вещей. 

**Строку подключения** можно найти в разделе **IoT Hub Service (Служба Центра Интернета вещей)** > **Параметры** > **Политика общего доступа** > **iothubowner**.

1. Настроив строку подключения Центра Интернета вещей, перейдите на вкладку **Управление**.

   ![Снимок экрана с двойником устройства и элементами управления в Device Explorer](./media/iot-hub-device-sdk-c-intro/DeviceExplorerTwinManagementTab.png)

Здесь вы можете управлять устройствами, зарегистрированными в Центре Интернета вещей.

1. Чтобы создать устройство, нажмите кнопку **Создать**. Откроется диалоговое окно с заполненным набором ключей (первичных и вторичных). Введите **идентификатор устройства**, а затем нажмите кнопку **Создать**.

   ![Снимок экрана с созданием устройства](./media/iot-hub-device-sdk-c-intro/CreateDevice.png)

1. Когда устройство будет создано, список зарегистрированных устройств обновится. Он будет включать только что созданное устройство. Если щелкнуть новое устройство правой кнопкой мыши, появится следующее меню:

   ![Результат щелчка правой кнопкой мыши по двойнику устройства Device Explorer](./media/iot-hub-device-sdk-c-intro/DeviceExplorerTwinManagementTab_RightClick.png)

1. Если выбрать пункт **Copy connection string for selected device** (Копировать строку подключения для выбранного устройства), то строка подключения устройства будет скопирована в буфер обмена. Сохраните копию строки подключения устройства. Она потребуется вам при запуске примеров приложений, описанных в следующих разделах.

Выполнив описанные выше действия, вы будете готовы к запуску кода. В большинстве примеров в верхней части основного исходного файла есть константа для ввода строки подключения. Например, соответствующая строка из приложения **iothub_client \_ samples \_ iothub_convenience_sample** Application выглядит следующим образом.

```c
static const char* connectionString = "[device connection string]";
```

## <a name="use-the-iothubclient-library"></a>Использование библиотеки IoTHubClient

В папке **iothub\_client** репозитория [azure-iot-sdk-c](https://github.com/azure/azure-iot-sdk-c) есть папка **samples**. Она содержит приложение с именем **iothub\_client\_sample\_mqtt**.

Версия Windows **iothub_client \_ samples \_ iothub_convenience_sample** Application содержит следующее решение Visual Studio:

  ![Обозреватель решений Visual Studio](./media/iot-hub-device-sdk-c-intro/iothub-client-sample-mqtt.png)

> [!NOTE]
> Если Visual Studio предложит перенацелить проект на последнюю версию, примите приглашение.

Это решение содержит один проект: В этом решении установлено четыре пакета NuGet:

* Microsoft.Azure.C.SharedUtility
* Microsoft.Azure.IoTHub.MqttTransport.
* Microsoft.Azure.IoTHub.IoTHubClient
* Microsoft.Azure.umqtt.

Для работы с пакетом SDK всегда необходим пакет **Microsoft.Azure.C.SharedUtility** . Так как этот пример использует протокол MQTT, вам также потребуются пакеты **Microsoft.Azure.umqtt** и **Microsoft.Azure.IoTHub.MqttTransport** (для AMQP и HTTP есть эквивалентные пакеты). Так как в рассматриваемом примере используется библиотека **IoTHubClient**, вам также нужно включить в решение пакет **Microsoft.Azure.IoTHub.IoTHubClient**.

Реализацию для примера приложения можно найти в **iothub_client \_ samples \_ iothub_convenience_sample** Source File.

На примере этого приложения мы покажем, что необходимо сделать для использования библиотеки **IoTHubClient**.

### <a name="initialize-the-library"></a>Инициализация библиотеки

> [!NOTE]
> Перед началом работы с библиотеками вам может потребоваться выполнить определенные действия по инициализации для конкретной платформы. Например, если вы планируете использовать AMQP в Linux, то вам необходимо инициализировать библиотеку OpenSSL. Примеры в [репозитории GitHub](https://github.com/Azure/azure-iot-sdk-c) вызывают служебную функцию **platform\_init**, когда клиент запускает и вызывает функцию **platform\_deinit** перед выходом. Эти функции объявлены в файле заголовка platform.h. Изучите определения этих функций для целевой платформы в [репозитории](https://github.com/Azure/azure-iot-sdk-c), чтобы определить, необходимо ли включить код для инициализации платформы в клиент.

Чтобы начать работу с библиотеками, сначала выделите дескриптор клиента Центра Интернета вещей.

```c
if ((iotHubClientHandle = 
  IoTHubClient_LL_CreateFromConnectionString(connectionString, MQTT_Protocol)) == NULL)
{
    (void)printf("ERROR: iotHubClientHandle is NULL!\r\n");
}
else
{
    ...
```

Вы передаете этой функции копию строки подключения устройства, полученную из обозревателя устройств. Также вы указываете протокол связи, который будет использоваться. В этом примере используется протокол MQTT, но вы также можете использовать протоколы AMQP и HTTPS.

Получив обработчик **IOTHUB\_CLIENT\_HANDLE**, вы можете начинать вызывать API-интерфейсы для отправки сообщений и их получения из Центра Интернета вещей.

### <a name="send-messages"></a>Отправка сообщений

Пример приложения настраивает цикл для отправки сообщений в Центр Интернета вещей. Пример кода ниже выполняет следующие действия:

- создает сообщение;
- добавляет свойство в сообщение;
- отправляет сообщение.

Сначала создайте сообщение:

```c
size_t iterator = 0;
do
{
    if (iterator < MESSAGE_COUNT)
    {
        sprintf_s(msgText, sizeof(msgText), "{\"deviceId\":\"myFirstDevice\",\"windSpeed\":%.2f}", avgWindSpeed + (rand() % 4 + 2));
        if ((messages[iterator].messageHandle = IoTHubMessage_CreateFromByteArray((const unsigned char*)msgText, strlen(msgText))) == NULL)
        {
            (void)printf("ERROR: iotHubMessageHandle is NULL!\r\n");
        }
        else
        {
            messages[iterator].messageTrackingId = iterator;
            MAP_HANDLE propMap = IoTHubMessage_Properties(messages[iterator].messageHandle);
            (void)sprintf_s(propText, sizeof(propText), "PropMsg_%zu", iterator);
            if (Map_AddOrUpdate(propMap, "PropName", propText) != MAP_OK)
            {
                (void)printf("ERROR: Map_AddOrUpdate Failed!\r\n");
            }

            if (IoTHubClient_LL_SendEventAsync(iotHubClientHandle, messages[iterator].messageHandle, SendConfirmationCallback, &messages[iterator]) != IOTHUB_CLIENT_OK)
            {
                (void)printf("ERROR: IoTHubClient_LL_SendEventAsync..........FAILED!\r\n");
            }
            else
            {
                (void)printf("IoTHubClient_LL_SendEventAsync accepted message [%d] for transmission to IoT Hub.\r\n", (int)iterator);
            }
        }
    }
    IoTHubClient_LL_DoWork(iotHubClientHandle);
    ThreadAPI_Sleep(1);

    iterator++;
} while (g_continueRunning);
```

Во время каждой отправки сообщения вам нужно указывать ссылку на функцию обратного вызова, которая будет вызываться при отправке данных. В этом примере функция обратного вызова называется **SendConfirmationCallback**. Эта функция обратного вызова показана в примере кода ниже.

```c
static void SendConfirmationCallback(IOTHUB_CLIENT_CONFIRMATION_RESULT result, void* userContextCallback)
{
    EVENT_INSTANCE* eventInstance = (EVENT_INSTANCE*)userContextCallback;
    (void)printf("Confirmation[%d] received for message tracking id = %zu with result = %s\r\n", callbackCounter, eventInstance->messageTrackingId, MU_ENUM_TO_STRING(IOTHUB_CLIENT_CONFIRMATION_RESULT, result));
    /* Some device specific action code goes here... */
    callbackCounter++;
    IoTHubMessage_Destroy(eventInstance->messageHandle);
}
```

Обратите внимание на вызов функции **IoTHubMessage\_Destroy** после завершения операции с сообщением. Эта функция необходима для освобождения ресурсов, выделенных во время создания сообщения.

### <a name="receive-messages"></a>Получение сообщений

Получение сообщения является асинхронной операцией. Сначала вам необходимо зарегистрировать функцию обратного вызова, выполняемую при получении устройством сообщения.

```c
if (IoTHubClient_LL_SetMessageCallback(iotHubClientHandle, ReceiveMessageCallback, &receiveContext) != IOTHUB_CLIENT_OK)
{
    (void)printf("ERROR: IoTHubClient_LL_SetMessageCallback..........FAILED!\r\n");
}
else
{
    (void)printf("IoTHubClient_LL_SetMessageCallback...successful.\r\n");
    ...
```

Последний параметр — это указатель void, который может указывать на что угодно. Хотя в нашем примере он указывает на целое число, в других случаях он может указывать и на более сложную структуру данных. В результате функция обратного вызова может работать в общем состоянии с объектом, вызывающим эту функцию.

Когда устройство получает сообщение, вызывается зарегистрированная функция обратного вызова. Эта функция обратного вызова получает следующее:

* Идентификатор сообщения и идентификатор корреляции из сообщения.
* содержимое сообщения;
* любые пользовательские свойства из сообщения.

```c
static IOTHUBMESSAGE_DISPOSITION_RESULT ReceiveMessageCallback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
{
    int* counter = (int*)userContextCallback;
    const char* buffer;
    size_t size;
    MAP_HANDLE mapProperties;
    const char* messageId;
    const char* correlationId;

    // Message properties
    if ((messageId = IoTHubMessage_GetMessageId(message)) == NULL)
    {
        messageId = "<null>";
    }

    if ((correlationId = IoTHubMessage_GetCorrelationId(message)) == NULL)
    {
        correlationId = "<null>";
    }

    // Message content
    if (IoTHubMessage_GetByteArray(message, (const unsigned char**)&buffer, &size) != IOTHUB_MESSAGE_OK)
    {
        (void)printf("unable to retrieve the message data\r\n");
    }
    else
    {
        (void)printf("Received Message [%d]\r\n Message ID: %s\r\n Correlation ID: %s\r\n Data: <<<%.*s>>> & Size=%d\r\n", *counter, messageId, correlationId, (int)size, buffer, (int)size);
        // If we receive the work 'quit' then we stop running
        if (size == (strlen("quit") * sizeof(char)) && memcmp(buffer, "quit", size) == 0)
        {
            g_continueRunning = false;
        }
    }

    // Retrieve properties from the message
    mapProperties = IoTHubMessage_Properties(message);
    if (mapProperties != NULL)
    {
        const char*const* keys;
        const char*const* values;
        size_t propertyCount = 0;
        if (Map_GetInternals(mapProperties, &keys, &values, &propertyCount) == MAP_OK)
        {
            if (propertyCount > 0)
            {
                size_t index;

                printf(" Message Properties:\r\n");
                for (index = 0; index < propertyCount; index++)
                {
                    (void)printf("\tKey: %s Value: %s\r\n", keys[index], values[index]);
                }
                (void)printf("\r\n");
            }
        }
    }

    /* Some device specific action code goes here... */
    (*counter)++;
    return IOTHUBMESSAGE_ACCEPTED;
}
```

Используйте функцию **IoTHubMessage\_GetByteArray**, чтобы получить сообщение, которое в этом примере представлено строкой.

### <a name="uninitialize-the-library"></a>Отмена инициализации библиотеки

После завершения отправки событий и получения сообщений вы можете отменить инициализацию библиотеки Центра Интернета вещей. Это можно сделать с помощью вызова следующей функции:

```c
IoTHubClient_LL_Destroy(iotHubClientHandle);
```

Этот вызов освобождает ресурсы, ранее выделенные функцией **IoTHubClient\_CreateFromConnectionString**.

Очевидно, что с помощью библиотеки **IoTHubClient** вы можете легко отправлять и получать сообщения. Библиотека обеспечивает обмен данными с Центром Интернета вещей, включая выбор используемого протокола (что с точки зрения разработчика является простым параметром конфигурации).

Библиотека **IoTHubClient** также позволяет полностью управлять способом сериализации данных, которые ваше устройство отправляет в Центр Интернета вещей. В некоторых случаях этот уровень контроля является преимуществом, но иногда в ходе реализации он может и не требоваться. В таком случае целесообразно использовать библиотеку **serializer**, о которой будет рассказано в следующем разделе.

## <a name="use-the-serializer-library"></a>Использование библиотеки serializer

На концептуальном уровне библиотека **serializer** управляет библиотекой **IoTHubClient**, входящей в пакет SDK. Она использует библиотеку **IoTHubClient** в качестве базового канала связи с Центром Интернета вещей, а также предоставляет возможности моделирования, которые помогают решать задачи, связанные с сериализацией сообщений. Работу этой библиотеки лучше всего продемонстрировать на примере.

Папка **serializer**[репозитория azure-iot-sdk-c](https://github.com/Azure/azure-iot-sdk-c) включает папку **samples**. В ней находится приложение **simplesample\_mqtt**. Версия этого примера для Windows включает следующее решение Visual Studio:

  ![Решение Visual Studio для примера mqtt](./media/iot-hub-device-sdk-c-intro/simplesample_mqtt.png)

> [!NOTE]
> Если Visual Studio предложит перенацелить проект на последнюю версию, примите приглашение.

Этот пример, как и предыдущий, включает несколько пакетов NuGet:

* Microsoft.Azure.C.SharedUtility
* Microsoft.Azure.IoTHub.MqttTransport.
* Microsoft.Azure.IoTHub.IoTHubClient
* Microsoft.Azure.IoTHub.Serializer
* Microsoft.Azure.umqtt.

Большинство из них мы видели в предыдущем примере, но **Microsoft.Azure.IoTHub.Serializer** — это новый пакет. Наличие этого пакета обязательно, если используется библиотека **serializer**.

Реализацию примера приложения можно найти в **iothub_client \_ samples \_ iothub_convenience_sample** File.

Следующие разделы содержат описание основных частей этого примера.

### <a name="initialize-the-library"></a>Инициализация библиотеки

Чтобы приступить к работе с библиотекой **serializer**, вам нужно вызвать API-интерфейсы инициализации.

```c
if (serializer_init(NULL) != SERIALIZER_OK)
{
    (void)printf("Failed on serializer_init\r\n");
}
else
{
    IOTHUB_CLIENT_LL_HANDLE iotHubClientHandle = IoTHubClient_LL_CreateFromConnectionString(connectionString, MQTT_Protocol);
    srand((unsigned int)time(NULL));
    int avgWindSpeed = 10;

    if (iotHubClientHandle == NULL)
    {
        (void)printf("Failed on IoTHubClient_LL_Create\r\n");
    }
    else
    {
        ContosoAnemometer* myWeather = CREATE_MODEL_INSTANCE(WeatherStation, ContosoAnemometer);
        if (myWeather == NULL)
        {
            (void)printf("Failed on CREATE_MODEL_INSTANCE\r\n");
        }
        else
        {
...
```

Функция **serializer\_init** вызывается один раз. Она инициализирует базовую библиотеку. Затем вызывается функция **IoTHubClient\_LL\_CreateFromConnectionString**. Это тот же API, что и в примере **IoTHubClient**. Вызов задает строку подключения устройства (на этом этапе вы также можете выбрать используемый протокол). В этом примере для транспорта используется протокол MQTT, но также могут использоваться протоколы AMQP или HTTPS.

Наконец, вызывается функция **CREATE\_MODEL\_INSTANCE**. **WeatherStation** — это пространство имен модели, а **ContosoAnemometer** — имя модели. Создав экземпляр модели, вы сможете использовать его для отправки и получения сообщений. Однако сначала нужно понять, что такое модель.

### <a name="define-the-model"></a>Определение модели

Модель в библиотеке **serializer** определяет сообщения, которые ваше устройство может отправлять в Центр Интернета вещей, а также сообщения, которые оно может получать (на языке моделирования называются *действия*). Модель определяется с помощью набора макросов на языке C, как в **iothub_client \_ примеров \_ iothub_convenience_sample** примере приложения:

```c
BEGIN_NAMESPACE(WeatherStation);

DECLARE_MODEL(ContosoAnemometer,
WITH_DATA(ascii_char_ptr, DeviceId),
WITH_DATA(int, WindSpeed),
WITH_ACTION(TurnFanOn),
WITH_ACTION(TurnFanOff),
WITH_ACTION(SetAirResistance, int, Position)
);

END_NAMESPACE(WeatherStation);
```

Макросы **BEGIN\_NAMESPACE** и **END\_NAMESPACE** принимают имя пространства имен модели в качестве аргумента. Предполагается, что весь код, находящийся между этими макросами, и является определением модели (или моделей) и структур данных, используемых в моделях.

В этом примере приведена одна модель с именем **ContosoAnemometer**. Эта модель определяет два фрагмента данных, которые ваше устройство может отправить в Центр Интернета вещей: **DeviceId** и **WindSpeed**. Она также определяет три действия (сообщения), которые ваше устройство может получить: **TurnFanOn**, **TurnFanOff** и **SetAirResistance**. Каждый элемент данных имеет тип, а каждое действие имеет имя (и при необходимости набор параметров).

Данные и действия, определенные в модели, определяют поверхность API, которую можно использовать для отправки сообщений в Центр Интернета вещей и ответа на сообщения, отправленные устройству. Лучше всего это показать на примере.

### <a name="send-messages"></a>Отправка сообщений

Модель определяет данные, которые вы можете отправить в Центр Интернета вещей. В нашем примере это один из двух элементов данных, определенных с помощью макроса **WITH_DATA**. Чтобы отправить в Центр Интернета вещей значения **DeviceId** и **WindSpeed**, необходимо выполнить несколько действий. Первое действие — определение данных для отправки.

```c
myWeather->DeviceId = "myFirstDevice";
myWeather->WindSpeed = avgWindSpeed + (rand() % 4 + 2);
```

Модель, которую вы определили раньше, позволяет задать значения путем определения участников **структуры**. Затем необходимо выполнить сериализацию сообщения, которое вы хотите отправить:

```c
unsigned char* destination;
size_t destinationSize;
if (SERIALIZE(&destination, &destinationSize, myWeather->DeviceId, myWeather->WindSpeed) != CODEFIRST_OK)
{
    (void)printf("Failed to serialize\r\n");
}
else
{
    sendMessage(iotHubClientHandle, destination, destinationSize);
    free(destination);
}
```

Этот код выполняет сериализацию сообщения, переданного из устройства в облако, в буфер (на который указывает **destination**). Затем код вызывает функцию **sendMessage**, чтобы отправить сообщение в Центр Интернета вещей.

```c
static void sendMessage(IOTHUB_CLIENT_LL_HANDLE iotHubClientHandle, const unsigned char* buffer, size_t size)
{
    static unsigned int messageTrackingId;
    IOTHUB_MESSAGE_HANDLE messageHandle = IoTHubMessage_CreateFromByteArray(buffer, size);
    if (messageHandle == NULL)
    {
        printf("unable to create a new IoTHubMessage\r\n");
    }
    else
    {
        if (IoTHubClient_LL_SendEventAsync(iotHubClientHandle, messageHandle, sendCallback, (void*)(uintptr_t)messageTrackingId) != IOTHUB_CLIENT_OK)
        {
            printf("failed to hand over the message to IoTHubClient");
        }
        else
        {
            printf("IoTHubClient accepted the message for delivery\r\n");
        }
        IoTHubMessage_Destroy(messageHandle);
    }
    messageTrackingId++;
}
```

Предпоследний параметр **IoTHubClient\_LL\_SendEventAsync** — это ссылка на функцию обратного вызова, вызываемую при успешной отправке данных. Функция обратного вызова в примере:

```c
void sendCallback(IOTHUB_CLIENT_CONFIRMATION_RESULT result, void* userContextCallback)
{
    unsigned int messageTrackingId = (unsigned int)(uintptr_t)userContextCallback;

    (void)printf("Message Id: %u Received.\r\n", messageTrackingId);

    (void)printf("Result Call Back Called! Result is: %s \r\n", MU_ENUM_TO_STRING(IOTHUB_CLIENT_CONFIRMATION_RESULT, result));
}
```

Второй параметр — это указатель на контекст пользователя. Такой же указатель мы передавали в **IoTHubClient\_LL\_SendEventAsync**. В данном случае контекстом является простой счетчик, но это может быть любой другой объект.

Это все, что требуется для отправки сообщений из устройства в облако. Единственное, о чем осталось рассказать, — это получение сообщений.

### <a name="receive-messages"></a>Получение сообщений

Получение сообщений выполняется аналогично обработке сообщений в библиотеке **IoTHubClient** . Сначала вам нужно зарегистрировать функцию обратного вызова сообщения:

```c
if (IoTHubClient_LL_SetMessageCallback(iotHubClientHandle, 
  IoTHubMessage, myWeather) != IOTHUB_CLIENT_OK)
{
    printf("unable to IoTHubClient_SetMessageCallback\r\n");
}
else
{
...
```

Затем вы создаете функцию обратного вызова, которая вызывается при получении сообщения:

```c
static IOTHUBMESSAGE_DISPOSITION_RESULT IoTHubMessage(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
{
    IOTHUBMESSAGE_DISPOSITION_RESULT result;
    const unsigned char* buffer;
    size_t size;
    if (IoTHubMessage_GetByteArray(message, &buffer, &size) != IOTHUB_MESSAGE_OK)
    {
        printf("unable to IoTHubMessage_GetByteArray\r\n");
        result = IOTHUBMESSAGE_ABANDONED;
    }
    else
    {
        /*buffer is not zero terminated*/
        char* temp = malloc(size + 1);
        if (temp == NULL)
        {
            printf("failed to malloc\r\n");
            result = IOTHUBMESSAGE_ABANDONED;
        }
        else
        {
            (void)memcpy(temp, buffer, size);
            temp[size] = '\0';
            EXECUTE_COMMAND_RESULT executeCommandResult = EXECUTE_COMMAND(userContextCallback, temp);
            result =
                (executeCommandResult == EXECUTE_COMMAND_ERROR) ? IOTHUBMESSAGE_ABANDONED :
                (executeCommandResult == EXECUTE_COMMAND_SUCCESS) ? IOTHUBMESSAGE_ACCEPTED :
                IOTHUBMESSAGE_REJECTED;
            free(temp);
        }
    }
    return result;
}
```

Данный код является стандартным и может использоваться с любым решением. Эта функция получает сообщение и перенаправляет его соответствующей функции через вызов **EXECUTE\_COMMAND**. Функция, которая будет вызвана в этот момент, зависит от определения действий в модели.

Когда вы определяете действие в модели, вам нужно реализовать функцию, которая вызывается, когда устройство получает соответствующее сообщение. Например, если модель определяет следующее действие:

```c
WITH_ACTION(SetAirResistance, int, Position)
```

Определите функцию со следующей сигнатурой:

```c
EXECUTE_COMMAND_RESULT SetAirResistance(ContosoAnemometer* device, int Position)
{
    (void)device;
    (void)printf("Setting Air Resistance Position to %d.\r\n", Position);
    return EXECUTE_COMMAND_SUCCESS;
}
```

Обратите внимание, что имя функции соответствует имени действия в модели, а параметры функции — параметрам, указанным для действия. Первый параметр всегда является обязательным и содержит указатель на экземпляр модели.

Когда устройство получает сообщение, которое соответствует сигнатуре, вызывается соответствующая функция. Таким образом, кроме включения стереотипного кода из **IoTHubMessage**, для получения сообщений нужно просто определить простую функцию для каждого действия, заданного в модели.

### <a name="uninitialize-the-library"></a>Отмена инициализации библиотеки

После завершения отправки данных и получения сообщений вы можете отменить инициализацию библиотеки Центра Интернета вещей:

```c
...
        DESTROY_MODEL_INSTANCE(myWeather);
    }
    IoTHubClient_LL_Destroy(iotHubClientHandle);
}
serializer_deinit();
```

Каждая из этих трех функций соответствуют трем функциям инициализации, описанным ранее. Вызов этих API позволит освободить выделенные ранее ресурсы.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье были рассмотрены основы использования библиотек в **пакете SDK для устройств Azure IOT для C**. Он предоставил достаточно информации, чтобы понять, что входит в пакет SDK, его архитектуру и как приступить к работе с примерами Windows. В следующей статье также рассказывается о пакете SDK и приводятся [дополнительные сведения о библиотеке IoTHubClient](iot-hub-device-sdk-c-iothubclient.md).

Дополнительные сведения о разработке для Центра Интернета вещей см. в статье [Понимание и использование пакетов SDK для Центра Интернета вещей Azure](iot-hub-devguide-sdks.md).

Для дальнейшего изучения возможностей Центра Интернета вещей см. следующие статьи:

* [Краткое руководство. Развертывание первого модуля IoT Edge на устройстве под управлением 64-разрядной ОС Linux](../iot-edge/quickstart-linux.md)
