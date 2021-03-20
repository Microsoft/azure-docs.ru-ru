---
title: Создание адаптера для моста самонастраивающийся IoT | Документация Майкрософт
description: Найдите компоненты адаптера моста IoT самонастраивающийся. Узнайте, как расширить мост, создав собственный адаптер.
author: usivagna
ms.author: ugans
ms.date: 1/20/2021
ms.topic: how-to
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 9a7028dfaeb94e87366de7acfa8cebc4c2f4c767
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98746834"
---
# <a name="extend-the-iot-plug-and-play-bridge"></a>Расширение моста самонастраивающийся IoT
[Мост Интернета вещей Самонастраивающийся](concepts-iot-pnp-bridge.md#iot-plug-and-play-bridge-architecture) позволяет подключать существующие устройства, подключенные к шлюзу, к центру Интернета вещей. С помощью моста можно сопоставлять интерфейсы Интернета вещей самонастраивающийся с подключенными устройствами. Интерфейс Интернета вещей самонастраивающийся определяет данные телеметрии, которые отправляет устройство, свойства, синхронизированные между устройством и облаком, а также команды, на которые отвечает устройство. Вы можете установить и настроить приложение моста с открытым исходным кодом в шлюзах Windows или Linux. Кроме того, мост можно запустить как модуль среды выполнения Azure IoT Edge.

В этой статье подробно описывается, как:

- Расширение моста самонастраивающийся IoT с помощью адаптера.
- Реализуйте общие обратные вызовы для адаптера моста.

Простой пример, демонстрирующий использование моста, см. в статье [Подключение примера моста Самонастраивающийся для Интернета вещей, работающего в Linux или Windows, в центр Интернета вещей](howto-use-iot-pnp-bridge.md).

Руководства и примеры в этой статье предполагают знание [Azure Digital двойников](../digital-twins/overview.md) и [IOT Самонастраивающийся](overview-iot-plug-and-play.md). Кроме того, в этой статье предполагается, что вы знаете [, как создать и развернуть мост IoT Самонастраивающийся](howto-build-deploy-extend-pnp-bridge.md).

## <a name="design-guide-to-extend-the-iot-plug-and-play-bridge-with-an-adapter"></a>Руководство по проектированию для расширения моста самонастраивающийся IoT с помощью адаптера

Чтобы расширить возможности моста, можно создать собственные адаптеры моста.

Мост использует адаптеры для:

- Установите подключение между устройством и облаком.
- Включите поток данных между устройством и облаком.
- Включение управления устройствами из облака.

Каждый адаптер моста должен:

- Создайте интерфейс Digital двойников.
- Используйте интерфейс для привязки функций на стороне устройства к облачным возможностям, таким как данные телеметрии, свойства и команды.
- Установите связь между системой управления и данными с оборудованием устройства или встроенным по.

Каждый адаптер моста взаимодействует с устройством определенного типа в зависимости от способа подключения адаптера к устройству и взаимодействия с ним. Даже если связь с устройством использует протокол подтверждения, адаптер моста может иметь несколько способов интерпретации данных с устройства. В этом сценарии адаптер моста использует сведения для адаптера в файле конфигурации, чтобы определить *конфигурацию интерфейса* , которую адаптер должен использовать для анализа данных.

Для взаимодействия с устройством адаптер моста использует протокол связи, поддерживаемый устройством и API, предоставляемые базовой операционной системой или поставщиком устройства.

Для взаимодействия с облаком адаптер моста использует интерфейсы API, предоставляемые пакетом SDK для устройства Azure IoT для отправки телеметрии, создания цифровых двойника интерфейсов, отправки обновлений свойств и создания функций обратного вызова для обновлений свойств и команд.

### <a name="create-a-bridge-adapter"></a>Создание адаптера моста

Мосту требуется адаптер моста для реализации интерфейсов API, определенных в интерфейсе [_PNP_ADAPTER](https://github.com/Azure/iot-plug-and-play-bridge/blob/9964f7f9f77ecbf4db3b60960b69af57fd83a871/pnpbridge/src/pnpbridge/inc/pnpadapter_api.h#L296) :

```c
typedef struct _PNP_ADAPTER {
  // Identity of the IoT Plug and Play adapter that is retrieved from the config
  const char* identity;

  PNPBRIDGE_ADAPTER_CREATE createAdapter;
  PNPBRIDGE_COMPONENT_CREATE createPnpComponent;
  PNPBRIDGE_COMPONENT_START startPnpComponent;
  PNPBRIDGE_COMPONENT_STOP stopPnpComponent;
  PNPBRIDGE_COMPONENT_DESTROY destroyPnpComponent;
  PNPBRIDGE_ADAPTER_DESTOY destroyAdapter;
} PNP_ADAPTER, * PPNP_ADAPTER;
```

В этом интерфейсе:

- `PNPBRIDGE_ADAPTER_CREATE` создает адаптер и настраивает ресурсы управления интерфейсом. Адаптер может также использовать глобальные параметры адаптера для создания адаптера. Эта функция вызывается один раз для одного адаптера.
- `PNPBRIDGE_COMPONENT_CREATE` создает клиентские интерфейсы Digital двойника и привязывает функции обратного вызова. Адаптер инициирует канал связи с устройством. Адаптер может настроить ресурсы для включения потока данных телеметрии, но не будет запускать передачу данных телеметрии до тех пор `PNPBRIDGE_COMPONENT_START` , пока не будет вызван. Эта функция вызывается один раз для каждого компонента интерфейса в файле конфигурации.
- `PNPBRIDGE_COMPONENT_START` вызывается, чтобы позволить адаптеру моста начать пересылку данных телеметрии с устройства на клиент Digital двойника. Эта функция вызывается один раз для каждого компонента интерфейса в файле конфигурации.
- `PNPBRIDGE_COMPONENT_STOP` останавливает поток данных телеметрии.
- `PNPBRIDGE_COMPONENT_DESTROY` уничтожает клиент Digital двойника и связанные ресурсы интерфейса. Эта функция вызывается один раз для каждого компонента интерфейса в файле конфигурации при разрыве моста или при возникновении неустранимой ошибки.
- `PNPBRIDGE_ADAPTER_DESTROY` Очищает ресурсы адаптера моста.

### <a name="bridge-core-interaction-with-bridge-adapters"></a>Взаимодействие основного моста с адаптерами моста

В следующем списке показано, что происходит при запуске моста:

1. При запуске моста диспетчер адаптеров моста просматривает каждый компонент интерфейса, определенный в файле конфигурации, и вызывает `PNPBRIDGE_ADAPTER_CREATE` на соответствующем адаптере. Адаптер может использовать глобальные параметры конфигурации адаптера, чтобы настроить ресурсы для поддержки различных *конфигураций интерфейса*.
1. Для каждого устройства в файле конфигурации диспетчер моста инициирует создание интерфейса путем вызова `PNPBRIDGE_COMPONENT_CREATE` в соответствующем адаптере моста.
1. Адаптер получает все необязательные параметры конфигурации адаптера для компонента интерфейса и использует эти сведения для настройки подключений к устройству.
1. Адаптер создает клиентские интерфейсы Digital двойника и привязывает функции обратного вызова для обновлений свойств и команд. Установка подключения устройства не должна блокировать возврат обратных вызовов после того, как создание интерфейса Digital двойника завершилось удачно. Подключение активного устройства не зависит от активного клиента интерфейса, создаваемого мостом. В случае сбоя подключения адаптер предполагает, что устройство неактивно. Адаптер моста может повторить попытку подключения.
1. После того как диспетчер адаптеров моста создаст все компоненты интерфейса, указанные в файле конфигурации, он будет регистрировать все интерфейсы с помощью центра Интернета вещей Azure. Регистрация является блокирующим асинхронным вызовом. По завершении вызова он активирует обратный вызов в адаптере моста, который затем может начать обработку обратных вызовов свойств и команд из облака.
1. Затем Диспетчер адаптера моста вызывает `PNPBRIDGE_INTERFACE_START` на каждом компоненте, а адаптер моста запускает данные телеметрии для клиента Digital двойника.

### <a name="design-guidelines"></a>Рекомендации по проектированию

При разработке нового адаптера моста следуйте приведенным ниже рекомендациям.

- Определите, какие возможности устройств поддерживаются, а также укажите, как выглядит определение интерфейса компонентов, использующих этот адаптер.
- Определите интерфейс и глобальные параметры, которые требуются адаптеру для определения в файле конфигурации.
- Выявление низкого уровня связи устройств, необходимого для поддержки свойств и команд компонента.
- Определите, как адаптер должен анализировать необработанные данные с устройства и преобразовать их в типы телеметрии, которые указывает определение интерфейса IoT самонастраивающийся.
- Реализуйте интерфейс адаптера моста, описанный выше.
- Добавьте новый адаптер в манифест адаптера и создайте мост.

### <a name="enable-a-new-bridge-adapter"></a>Включение нового адаптера моста

Вы включаете адаптеры в Мосте, добавляя ссылку в [adapter_manifest. c](https://github.com/Azure/iot-plug-and-play-bridge/blob/master/pnpbridge/src/adapters/src/shared/adapter_manifest.c):

```c
  extern PNP_ADAPTER MyPnpAdapter;
  PPNP_ADAPTER PNP_ADAPTER_MANIFEST[] = {
    .
    .
    &MyPnpAdapter
  }
```

> [!IMPORTANT]
> Обратные вызовы адаптера моста вызываются последовательно. Адаптер не должен блокировать обратный вызов, так как это не позволяет ядру моста делать ход выполнения.

### <a name="sample-camera-adapter"></a>Адаптер примера камеры

В [файле readme адаптера камеры](https://github.com/Azure/iot-plug-and-play-bridge/blob/master/pnpbridge/src/adapters/src/Camera/readme.md) описывается пример адаптера камеры, который можно включить.

## <a name="code-examples-for-common-adapter-scenarioscallbacks"></a>Примеры кода для распространенных сценариев и обратных вызовов адаптера

В следующем разделе приводятся сведения о том, как адаптер для моста будет реализовывать обратные вызовы для ряда распространенных сценариев и использования в этом разделе рассматриваются следующие обратные вызовы:
- [Получить обновление свойства (от облака к устройству)](#receive-property-update-cloud-to-device)
- [Сообщить об обновлении свойства (устройство в облако)](#report-a-property-update-device-to-cloud)
- [Отправить телеметрию (устройство в облако)](#send-telemetry-device-to-cloud)
- [Получение обратного вызова обновления команды из облака и его обработка на стороне устройства (с облака на устройство)](#receive-command-update-callback-from-the-cloud-and-process-it-on-the-device-side-cloud-to-device)
- [Ответ на обновление команды на стороне устройства (устройство в облако)](#respond-to-command-update-on-the-device-side-device-to-cloud)

Приведенные ниже примеры основаны на [примере адаптера "датчик окружающей среды](https://github.com/Azure/iot-plug-and-play-bridge/tree/master/pnpbridge/src/adapters/samples/environmental_sensor)".

### <a name="receive-property-update-cloud-to-device"></a>Получить обновление свойства (от облака к устройству)
Первым шагом является регистрация функции обратного вызова:

```c
PnpComponentHandleSetPropertyUpdateCallback(BridgeComponentHandle, EnvironmentSensor_ProcessPropertyUpdate);
```
Следующим шагом является реализация функции обратного вызова для чтения обновления свойства на устройстве:

```c
void EnvironmentSensor_ProcessPropertyUpdate(
    PNPBRIDGE_COMPONENT_HANDLE PnpComponentHandle,
    const char* PropertyName,
    JSON_Value* PropertyValue,
    int version,
    void* userContextCallback
)
{
  // User context for the callback is set to the IoT Hub client handle, and therefore can be type-cast to the client handle type
    SampleEnvironmentalSensor_ProcessPropertyUpdate(userContextCallback, PropertyName, PropertyValue, version, PnpComponentHandle);
}

// SampleEnvironmentalSensor_ProcessPropertyUpdate receives updated properties from the server.  This implementation
// acts as a simple dispatcher to the functions to perform the actual processing.
void SampleEnvironmentalSensor_ProcessPropertyUpdate(
    void * ClientHandle,
    const char* PropertyName,
    JSON_Value* PropertyValue,
    int version,
    PNPBRIDGE_COMPONENT_HANDLE PnpComponentHandle)
{
  if (strcmp(PropertyName, sampleEnvironmentalSensorPropertyBrightness) == 0)
    {
        SampleEnvironmentalSensor_BrightnessCallback(ClientHandle, PropertyName, PropertyValue, version, PnpComponentHandle);
    }
    else
    {
        // If the property is not implemented by this interface, presently we only record a log message but do not have a mechanism to report back to the service
        LogError("Environmental Sensor Adapter:: Property name <%s> is not associated with this interface", PropertyName);
    }
}

// Process a property update for bright level.
static void SampleEnvironmentalSensor_BrightnessCallback(
    void * ClientHandle,
    const char* PropertyName,
    JSON_Value* PropertyValue,
    int version,
    PNPBRIDGE_COMPONENT_HANDLE PnpComponentHandle)
{
    IOTHUB_CLIENT_RESULT iothubClientResult;
    STRING_HANDLE jsonToSend = NULL;
    char targetBrightnessString[32];

    LogInfo("Environmental Sensor Adapter:: Brightness property invoked...");

    PENVIRONMENT_SENSOR EnvironmentalSensor = PnpComponentHandleGetContext(PnpComponentHandle);

    if (json_value_get_type(PropertyValue) != JSONNumber)
    {
        LogError("JSON field %s is not a number", PropertyName);
    }
    else if(EnvironmentalSensor == NULL || EnvironmentalSensor->SensorState == NULL)
    {
        LogError("Environmental sensor device context not initialized correctly.");
    }
    else if (SampleEnvironmentalSensor_ValidateBrightness(json_value_get_number(PropertyValue)))
    {
        EnvironmentalSensor->SensorState->brightness = (int) json_value_get_number(PropertyValue);
        if (snprintf(targetBrightnessString, sizeof(targetBrightnessString), 
            g_environmentalSensorBrightnessResponseFormat, EnvironmentalSensor->SensorState->brightness) < 0)
        {
            LogError("Unable to create target brightness string for reporting result");
        }
        else if ((jsonToSend = PnP_CreateReportedPropertyWithStatus(EnvironmentalSensor->SensorState->componentName,
                    PropertyName, targetBrightnessString, PNP_STATUS_SUCCESS, g_environmentalSensorPropertyResponseDescription,
                    version)) == NULL)
        {
            LogError("Unable to build reported property response");
        }
        else
        {
            const char* jsonToSendStr = STRING_c_str(jsonToSend);
            size_t jsonToSendStrLen = strlen(jsonToSendStr);

            if ((iothubClientResult = SampleEnvironmentalSensor_RouteReportedState(ClientHandle, PnpComponentHandle, (const unsigned char*)jsonToSendStr, jsonToSendStrLen,
                                        SampleEnvironmentalSensor_PropertyCallback,
                                        (void*) &EnvironmentalSensor->SensorState->brightness)) != IOTHUB_CLIENT_OK)
            {
                LogError("Environmental Sensor Adapter:: SampleEnvironmentalSensor_RouteReportedState for brightness failed, error=%d", iothubClientResult);
            }
            else
            {
                LogInfo("Environmental Sensor Adapter:: Successfully queued Property update for Brightness for component=%s", EnvironmentalSensor->SensorState->componentName);
            }

            STRING_delete(jsonToSend);
        }
    }
}

```

### <a name="report-a-property-update-device-to-cloud"></a>Сообщить об обновлении свойства (устройство в облако)
В любой момент после создания компонента устройство может передавать свойства в облако с состоянием: 
```c
// Environmental sensor's read-only property, device state indiciating whether its online or not
//
static const char sampleDeviceStateProperty[] = "state";
static const unsigned char sampleDeviceStateData[] = "true";
static const int sampleDeviceStateDataLen = sizeof(sampleDeviceStateData) - 1;

// Sends a reported property for device state of this simulated device.
IOTHUB_CLIENT_RESULT SampleEnvironmentalSensor_ReportDeviceStateAsync(
    PNPBRIDGE_COMPONENT_HANDLE PnpComponentHandle,
    const char * ComponentName)
{

    IOTHUB_CLIENT_RESULT iothubClientResult = IOTHUB_CLIENT_OK;
    STRING_HANDLE jsonToSend = NULL;

    if ((jsonToSend = PnP_CreateReportedProperty(ComponentName, sampleDeviceStateProperty, (const char*) sampleDeviceStateData)) == NULL)
    {
        LogError("Unable to build reported property response for propertyName=%s, propertyValue=%s", sampleDeviceStateProperty, sampleDeviceStateData);
    }
    else
    {
        const char* jsonToSendStr = STRING_c_str(jsonToSend);
        size_t jsonToSendStrLen = strlen(jsonToSendStr);

        if ((iothubClientResult = SampleEnvironmentalSensor_RouteReportedState(NULL, PnpComponentHandle, (const unsigned char*)jsonToSendStr, jsonToSendStrLen,
            SampleEnvironmentalSensor_PropertyCallback, (void*)sampleDeviceStateProperty)) != IOTHUB_CLIENT_OK)
        {
            LogError("Environmental Sensor Adapter:: Unable to send reported state for property=%s, error=%d",
                                sampleDeviceStateProperty, iothubClientResult);
        }
        else
        {
            LogInfo("Environmental Sensor Adapter:: Sending device information property to IoTHub. propertyName=%s, propertyValue=%s",
                        sampleDeviceStateProperty, sampleDeviceStateData);
        }

        STRING_delete(jsonToSend);
    }

    return iothubClientResult;
}


// Routes the reported property for device or module client. This function can be called either by passing a valid client handle or by passing
// a NULL client handle after components have been started such that the client handle can be extracted from the PnpComponentHandle
IOTHUB_CLIENT_RESULT SampleEnvironmentalSensor_RouteReportedState(
    void * ClientHandle,
    PNPBRIDGE_COMPONENT_HANDLE PnpComponentHandle,
    const unsigned char * ReportedState,
    size_t Size,
    IOTHUB_CLIENT_REPORTED_STATE_CALLBACK ReportedStateCallback,
    void * UserContextCallback)
{
    IOTHUB_CLIENT_RESULT iothubClientResult = IOTHUB_CLIENT_OK;

    PNP_BRIDGE_CLIENT_HANDLE clientHandle = (ClientHandle != NULL) ?
            (PNP_BRIDGE_CLIENT_HANDLE) ClientHandle : PnpComponentHandleGetClientHandle(PnpComponentHandle);

    if ((iothubClientResult = PnpBridgeClient_SendReportedState(clientHandle, ReportedState, Size,
            ReportedStateCallback, UserContextCallback)) != IOTHUB_CLIENT_OK)
    {
        LogError("IoTHub client call to _SendReportedState failed with error code %d", iothubClientResult);
        goto exit;
    }
    else
    {
        LogInfo("IoTHub client call to _SendReportedState succeeded");
    }

exit:
    return iothubClientResult;
}

```

### <a name="send-telemetry-device-to-cloud"></a>Отправить телеметрию (устройство в облако)
```c
//
// SampleEnvironmentalSensor_SendTelemetryMessagesAsync is periodically invoked by the caller to
// send telemetry containing the current temperature and humidity (in both cases random numbers
// so this sample will work on platforms without these sensors).
//
IOTHUB_CLIENT_RESULT SampleEnvironmentalSensor_SendTelemetryMessagesAsync(
    PNPBRIDGE_COMPONENT_HANDLE PnpComponentHandle)
{
    IOTHUB_CLIENT_RESULT result = IOTHUB_CLIENT_OK;
    IOTHUB_MESSAGE_HANDLE messageHandle = NULL;
    PENVIRONMENT_SENSOR device = PnpComponentHandleGetContext(PnpComponentHandle);

    float currentTemperature = 20.0f + ((float)rand() / RAND_MAX) * 15.0f;
    float currentHumidity = 60.0f + ((float)rand() / RAND_MAX) * 20.0f;

    char currentMessage[128];
    sprintf(currentMessage, "{\"%s\":%.3f, \"%s\":%.3f}", SampleEnvironmentalSensor_TemperatureTelemetry, 
            currentTemperature, SampleEnvironmentalSensor_HumidityTelemetry, currentHumidity);


    if ((messageHandle = PnP_CreateTelemetryMessageHandle(device->SensorState->componentName, currentMessage)) == NULL)
    {
        LogError("Environmental Sensor Adapter:: PnP_CreateTelemetryMessageHandle failed.");
    }
    else if ((result = SampleEnvironmentalSensor_RouteSendEventAsync(PnpComponentHandle, messageHandle,
            SampleEnvironmentalSensor_TelemetryCallback, device)) != IOTHUB_CLIENT_OK)
    {
        LogError("Environmental Sensor Adapter:: SampleEnvironmentalSensor_RouteSendEventAsync failed, error=%d", result);
    }

    IoTHubMessage_Destroy(messageHandle);

    return result;
}

// Routes the sending asynchronous events for device or module client
IOTHUB_CLIENT_RESULT SampleEnvironmentalSensor_RouteSendEventAsync(
        PNPBRIDGE_COMPONENT_HANDLE PnpComponentHandle,
        IOTHUB_MESSAGE_HANDLE EventMessageHandle,
        IOTHUB_CLIENT_EVENT_CONFIRMATION_CALLBACK EventConfirmationCallback,
        void * UserContextCallback)
{
    IOTHUB_CLIENT_RESULT iothubClientResult = IOTHUB_CLIENT_OK;
    PNP_BRIDGE_CLIENT_HANDLE clientHandle = PnpComponentHandleGetClientHandle(PnpComponentHandle);
    if ((iothubClientResult = PnpBridgeClient_SendEventAsync(clientHandle, EventMessageHandle,
            EventConfirmationCallback, UserContextCallback)) != IOTHUB_CLIENT_OK)
    {
        LogError("IoTHub client call to _SendEventAsync failed with error code %d", iothubClientResult);
        goto exit;
    }
    else
    {
        LogInfo("IoTHub client call to _SendEventAsync succeeded");
    }

exit:
    return iothubClientResult;
}

```
### <a name="receive-command-update-callback-from-the-cloud-and-process-it-on-the-device-side-cloud-to-device"></a>Получение обратного вызова обновления команды из облака и его обработка на стороне устройства (с облака на устройство)
```c
// SampleEnvironmentalSensor_ProcessCommandUpdate receives commands from the server.  This implementation acts as a simple dispatcher
// to the functions to perform the actual processing.
int SampleEnvironmentalSensor_ProcessCommandUpdate(
    PENVIRONMENT_SENSOR EnvironmentalSensor,
    const char* CommandName,
    JSON_Value* CommandValue,
    unsigned char** CommandResponse,
    size_t* CommandResponseSize)
{
    if (strcmp(CommandName, sampleEnvironmentalSensorCommandBlink) == 0)
    {
        return SampleEnvironmentalSensor_BlinkCallback(EnvironmentalSensor, CommandValue, CommandResponse, CommandResponseSize);
    }
    else if (strcmp(CommandName, sampleEnvironmentalSensorCommandTurnOn) == 0)
    {
        return SampleEnvironmentalSensor_TurnOnLightCallback(EnvironmentalSensor, CommandValue, CommandResponse, CommandResponseSize);
    }
    else if (strcmp(CommandName, sampleEnvironmentalSensorCommandTurnOff) == 0)
    {
        return SampleEnvironmentalSensor_TurnOffLightCallback(EnvironmentalSensor, CommandValue, CommandResponse, CommandResponseSize);
    }
    else
    {
        // If the command is not implemented by this interface, by convention we return a 404 error to server.
        LogError("Environmental Sensor Adapter:: Command name <%s> is not associated with this interface", CommandName);
        return SampleEnvironmentalSensor_SetCommandResponse(CommandResponse, CommandResponseSize, sampleEnviromentalSensor_NotImplemented);
    }
}

// Implement the callback to process the command "blink". Information pertaining to the request is
// specified in the CommandValue parameter, and the callback fills out data it wishes to
// return to the caller on the service in CommandResponse.

static int SampleEnvironmentalSensor_BlinkCallback(
    PENVIRONMENT_SENSOR EnvironmentalSensor,
    JSON_Value* CommandValue,
    unsigned char** CommandResponse,
    size_t* CommandResponseSize)
{
    int result = PNP_STATUS_SUCCESS;
    int BlinkInterval = 0;

    LogInfo("Environmental Sensor Adapter:: Blink command invoked. It has been invoked %d times previously", EnvironmentalSensor->SensorState->numTimesBlinkCommandCalled);

    if (json_value_get_type(CommandValue) != JSONNumber)
    {
        LogError("Cannot retrieve blink interval for blink command");
        result = PNP_STATUS_BAD_FORMAT;
    }
    else
    {
        BlinkInterval = (int)json_value_get_number(CommandValue);
        LogInfo("Environmental Sensor Adapter:: Blinking with interval=%d second(s)", BlinkInterval);
        EnvironmentalSensor->SensorState->numTimesBlinkCommandCalled++;
        EnvironmentalSensor->SensorState->blinkInterval = BlinkInterval;

        result = SampleEnvironmentalSensor_SetCommandResponse(CommandResponse, CommandResponseSize, sampleEnviromentalSensor_BlinkResponse);
    }

    return result;
}

```
### <a name="respond-to-command-update-on-the-device-side-device-to-cloud"></a>Ответ на обновление команды на стороне устройства (устройство в облако)

```c
    static int SampleEnvironmentalSensor_BlinkCallback(
        PENVIRONMENT_SENSOR EnvironmentalSensor,
        JSON_Value* CommandValue,
        unsigned char** CommandResponse,
        size_t* CommandResponseSize)
    {
        int result = PNP_STATUS_SUCCESS;
        int BlinkInterval = 0;
    
        LogInfo("Environmental Sensor Adapter:: Blink command invoked. It has been invoked %d times previously", EnvironmentalSensor->SensorState->numTimesBlinkCommandCalled);
    
        if (json_value_get_type(CommandValue) != JSONNumber)
        {
            LogError("Cannot retrieve blink interval for blink command");
            result = PNP_STATUS_BAD_FORMAT;
        }
        else
        {
            BlinkInterval = (int)json_value_get_number(CommandValue);
            LogInfo("Environmental Sensor Adapter:: Blinking with interval=%d second(s)", BlinkInterval);
            EnvironmentalSensor->SensorState->numTimesBlinkCommandCalled++;
            EnvironmentalSensor->SensorState->blinkInterval = BlinkInterval;
    
            result = SampleEnvironmentalSensor_SetCommandResponse(CommandResponse, CommandResponseSize, sampleEnviromentalSensor_BlinkResponse);
        }
    
        return result;
    }
    
    // SampleEnvironmentalSensor_SetCommandResponse is a helper that fills out a command response
    static int SampleEnvironmentalSensor_SetCommandResponse(
        unsigned char** CommandResponse,
        size_t* CommandResponseSize,
        const unsigned char* ResponseData)
    {
        int result = PNP_STATUS_SUCCESS;
        if (ResponseData == NULL)
        {
            LogError("Environmental Sensor Adapter:: Response Data is empty");
            *CommandResponseSize = 0;
            return PNP_STATUS_INTERNAL_ERROR;
        }
    
        *CommandResponseSize = strlen((char*)ResponseData);
        memset(CommandResponse, 0, sizeof(*CommandResponse));
    
        // Allocate a copy of the response data to return to the invoker. Caller will free this.
        if (mallocAndStrcpy_s((char**)CommandResponse, (char*)ResponseData) != 0)
        {
            LogError("Environmental Sensor Adapter:: Unable to allocate response data");
            result = PNP_STATUS_INTERNAL_ERROR;
        }
    
        return result;
}
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Bridge самонастраивающийся центра Интернета вещей см. в репозитории GitHub [Самонастраивающийся Bridge](https://github.com/Azure/iot-plug-and-play-bridge) .
