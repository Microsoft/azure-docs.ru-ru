---
title: Создание расширенной модели имитированного устройства в Azure | Документация Майкрософт
description: В этом практическом руководстве вы узнаете, как создать расширенную модель устройства для использования с акселератором решений "Имитация устройств".
author: troyhopwood
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 03/18/2019
ms.author: troyhop
ms.custom:
- mvc
- amqp
- mqtt
- devx-track-js
ms.openlocfilehash: 9b8c1bb5669f058aba00f44ce62e48c7fab233f9
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106057751"
---
# <a name="create-an-advanced-device-model"></a>Создание расширенной модели устройства

В этом практическом руководстве описаны файлы JSON и JavaScript, которые определяют пользовательскую модель устройства. Статья содержит несколько примеров файлов с определениями устройств и инструкции, позволяющие передать их в экземпляр симулятора устройств. С помощью расширенной модели устройства вы можете реализовать более реалистичное поведение устройства для целей тестирования.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить действия, описанные в этом практическом руководстве, требуется развернутый в подписке Azure экземпляр симулятора устройств.

Если вы еще не развернули имитацию устройств, см. сведения о [развертывании имитации устройств](https://github.com/Azure/device-simulation-dotnet/blob/master/README.md) на сайте GitHub.

### <a name="open-device-simulation"></a>Открытие симулятора устройств

Если вы еще не развернули имитацию устройств, см. сведения о [развертывании имитации устройств](https://github.com/Azure/device-simulation-dotnet/blob/master/README.md) на сайте GitHub.

## <a name="device-models"></a>Модель устройства

Каждое имитированное устройство сопоставлено с определенной моделью устройства, которая определяет поведение этой имитации. Поведение описывает такие параметры, как частота отправки данных телеметрии, типы отправляемых сообщений, поддерживаемые методы.

Чтобы определить модель устройства, вам потребуется JSON-файл определения устройства и набор файлов JavaScript. В этих файлах JavaScript описано поведение имитации, например случайные данные телеметрии и логика методов.

Типичная модель включает в себя следующие компоненты.

* Один файл JSON для каждой модели устройства (например, elevator.json).
* Один файл JavaScript со скриптами поведения для каждой модели устройства (например, elevator-state.js)
* Один файл JavaScript со скриптом метода для каждого метода устройства (например, elevator-go-down.js)

> [!NOTE]
> Не все модели устройства определяет методы. Это означает, что модель устройства может иметь или не иметь скрипты методов. Но все модели устройства обязаны иметь скрипт поведения.

## <a name="device-definition-file"></a>Файл определения устройства

Каждый файл определения устройства содержит подробные сведения о модели имитированного устройства, включая следующие сведения.

* Имя модели устройства в строковом формате.
* Протокол: AMQP | MQTT | HTTP.
* Первоначальное состояние устройства.
* Частота обновления состояния устройства.
* Какой файл JavaScript следует использовать для обновления состояния устройства.
* Список сообщений телеметрии для отправки с указанием частоты для каждого.
* Схема сообщений телеметрии, используемая приложениями серверной части для синтаксического анализа полученной телеметрии.
* Список поддерживаемых методов и файлы JavaScript, используемые для имитации каждого метода.

### <a name="file-schema"></a>Схема файла

Версия схемы всегда имеет значение 1.0.0 и соответствует формату этого файла.

```json
"SchemaVersion": "1.0.0"
```

### <a name="device-model-description"></a>Описание модели устройства

Следующие свойства описывают модель устройства. Каждый тип имеет уникальный идентификатор, семантическую версию, имя и описание:

```json
"Id": "chiller-01",
"Version": "0.0.1",
"Name": "Chiller",
"Description": "Chiller with external temperature and humidity sensors."
```

### <a name="iot-protocol"></a>Протокол Интернета вещей

Устройства Интернета вещей могут использовать для связи разные протоколы. Для имитации можно выбрать протокол **AMQP**, **MQTT** или **HTTP**:

```json
"Protocol": "AMQP"
```

### <a name="simulated-device-state"></a>Состояние имитированного устройства

Каждое имитированное устройство имеет внутреннее состояние, который должно быть определено. Также это состояние определяет свойства, передаваемые в телеметрии. Например холодильник может иметь такое начальное состояние:

```json
"InitialState": {
    "temperature": 50,
    "humidity": 40
},
```

Устройство с возможностью перемещения и несколькими датчиками будет иметь дополнительные свойства, например:

```json
"InitialState": {
    "latitude": 47.445301,
    "longitude": -122.296307,
    "speed": 30.0,
    "speed_unit": "mph",
    "cargotemperature": 38.0,
    "cargotemperature_unit": "F"
}
```

Состояние устройства хранится в памяти службы симулятора и передается в качестве входных данных для функции JavaScript. Функция JavaScript может выбрать следующие стратегии:

* игнорировать состояние и создать случайные данные;
* обновить состояние устройства некоторым образом, который имеет смысл для выбранного сценария.

Функция, которая генерирует состояние, также получает в качестве входных данных:

* идентификатор устройства;
* модель устройства;
* Текущее время. Это значение позволяет создавать разные данные в зависимости от конкретного устройства и текущего времени.

### <a name="generating-telemetry-messages"></a>Формирование сообщений телеметрии

Служба симулятора может отправлять данные телеметрии нескольких разных типов для каждого устройства. Обычно данные телеметрии содержат сведения о состоянии устройства. Например, имитация комнаты может каждые 10 секунд отправлять сведения о температуре и влажности. Обратите внимание на заполнители в приведенном ниже фрагменте кода, которые автоматически заменяются значениями из состояния устройства:

```json
"Telemetry": [
    {
        "Interval": "00:00:10",
        "MessageTemplate":
            "{\"temperature\":${temperature},\"temperature_unit\":\"${temperature_unit}\",\"humidity\":\"${humidity}\"}",
        "MessageSchema": {
            "Name": "RoomComfort;v1",
            "Format": "JSON",
            "Fields": {
                "temperature": "double",
                "temperature_unit": "text",
                "humidity": "integer"
            }
        }
    }
],
```

Заполнители имеют специальный синтаксис **${NAME}** где **NAME** обозначает ключ из объекта состояния устройства, который возвращается функцией JavaScript **main**. Строки следует заключить в кавычки, а числа — нет.

#### <a name="message-schema"></a>Схема сообщений

Каждый тип сообщения должен иметь четко определенную схему. Схема сообщения также передается в Центр Интернета вещей, что позволяет серверным приложениям повторно использовать эти сведения для анализа входящей телеметрии.

Схема поддерживает формат JSON, который удобен для синтаксического анализа, преобразования и аналитики в самых разных системах и службах.

Поля, перечисленные в схеме, могут быть следующих типов:

* Object — можно сериализовать с помощью JSON.
* Binary — можно сериализовать с помощью Base64.
* текст
* Логическое
* Целое число
* Double
* Дата и время

### <a name="supported-methods"></a>Поддерживаемые методы

Имитированные устройства могут реагировать на вызовы методов, выполняя для них определенную логику и предоставляя ответы. Как и для имитации, логика метода хранится в файле JavaScript и может взаимодействовать с состоянием устройства. Пример:

```json
"CloudToDeviceMethods": {
    "Start": {
        "Type": "JavaScript",
        "Path": "release-pressure.js"
    }
}
```

## <a name="create-a-device-definition-file"></a>Создание файла определения устройства

В этом практическом руководстве вы узнаете, как создать модель устройства дрона. Этот дрон будет случайным образом летать вокруг начальной координатной позиции, изменяя расположение и высоту.

Скопируйте приведенный ниже код JSON в текстовый редактор и сохраните его в файл **drone.json**.

### <a name="device-definition-json-example"></a>Пример кода JSON для определения устройства

```json
{
  "SchemaVersion": "1.0.0",
  "Id": "drone",
  "Version": "0.0.1",
  "Name": "Drone",
  "Description": "Simple drone.",
  "Protocol": "AMQP",
  "Simulation": {
    "InitialState": {
      "velocity": 0.0,
      "velocity_unit": "mm/sec",
      "acceleration": 0.0,
      "acceleration_unit": "mm/sec^2",
      "latitude": 47.476075,
      "longitude": -122.192026,
      "altitude": 0.0
    },
    "Interval": "00:00:05",
    "Scripts": [{
      "Type": "JavaScript",
      "Path": "drone-state.js"
    }]
  },
  "Properties": {
    "Type": "Drone",
    "Firmware": "1.0",
    "Model": "P-96"
  },
  "Tags": {
    "Owner": "Contoso"
  },
  "Telemetry": [{
      "Interval": "00:00:05",
      "MessageTemplate": "{\"velocity\":\"${velocity}\",\"acceleration\":\"${acceleration}\",\"position\":\"${latitude}|${longitude}|${altitude}\"}",
      "MessageSchema": {
        "Name": "drone-event-sensor;v1",
        "Format": "JSON",
        "Fields": {
          "velocity": "double",
          "velocity_unit": "text",
          "acceleration": "double",
          "acceleration_unit": "text",
          "latitude": "double",
          "longitude": "double",
          "altitude": "double"
        }
      }
    }
  ],
    "CloudToDeviceMethods": {
        "RecallDrone": {
            "Type": "JavaScript",
            "Path": "droneRecall-method.js"
        }
    }
}
```

## <a name="behavior-script-files"></a>Файлы скриптов поведения

Код в файле скрипта поведения перемещает дрон. Этот скрипт изменяет высоту и расположение дрона, сохраняя в память новые состояния устройства.

Каждый файл JavaScript должен иметь функцию **main**, которая принимает два параметра.

* Объект **context** с тремя свойствами:
    * **currentTime** содержит строку в формате **yyyy-MM-dd'T'HH:mm:sszzz**.
    * **deviceId** Идентификатор устройства, например **Simulated.Elevator.123**.
    * **deviceModel**. Модель устройства, например **Elevator** (Лифт).
* Объект **state** (состояние), который хранит значение, полученное при предыдущем вызове функции. Это состояние устройства поддерживается службой симулятора и используется для создания сообщений телеметрии.

Функция **main** возвращает новое состояние устройства. Пример:

```JavaScript
function main(context, state) {

    // Use context if the simulation depends on
    // time or device details.
    // Execute some logic, updating 'state'

    return state;
}
```

## <a name="create-a-behavior-script-file"></a>Создайте файл скрипта поведения

Скопируйте приведенный ниже код JavaScript в текстовый редактор и сохраните его в файл **drone-state.js**.

### <a name="device-model-javascript-simulation-example"></a>Пример модели имитации устройства на JavaScript

```JavaScript
"use strict";

// Position control
const DefaultLatitude = 47.476075;
const DefaultLongitude = -122.192026;
const DistanceVariation = 10;

// Altitude control
const DefaultAltitude = 0.0;
const AverageAltitude = 499.99;
const AltitudeVariation = 5;

// Velocity control
const AverageVelocity = 60.00;
const MinVelocity = 20.00;
const MaxVelocity = 120.00;
const VelocityVariation = 5;

// Acceleration control
const AverageAcceleration = 2.50;
const MinAcceleration = 0.01;
const MaxAcceleration = 9.99;
const AccelerationVariation = 1;

// Display control for position and other attributes
const GeoSpatialPrecision = 6;
const DecimalPrecision = 2;

// Default state
var state = {
    velocity: 0.0,
    velocity_unit: "mm/sec",
    acceleration: 0.0,
    acceleration_unit: "mm/sec^2",
    latitude: DefaultLatitude,
    longitude: DefaultLongitude,
    altitude: DefaultAltitude
};

// Default device properties
var properties = {};

/**
 * Restore the global state using data from the previous iteration.
 *
 * @param previousState device state from the previous iteration
 * @param previousProperties device properties from the previous iteration
 */
function restoreSimulation(previousState, previousProperties) {
    // If the previous state is null, force a default state
    if (previousState) {
        state = previousState;
    } else {
        log("Using default state");
    }

    if (previousProperties) {
        properties = previousProperties;
    } else {
        log("Using default properties");
    }
}

/**
 * Simple formula generating a random value around the average
 * in between min and max
 */
function vary(avg, percentage, min, max) {
    var value = avg * (1 + ((percentage / 100) * (2 * Math.random() - 1)));
    value = Math.max(value, min);
    value = Math.min(value, max);
    return value;
}

/**
 * Entry point function called by the simulation engine.
 * Returns updated simulation state.
 * Device property updates must call updateProperties() to persist.
 *
 * @param context             The context contains current time, device model and id
 * @param previousState       The device state since the last iteration
 * @param previousProperties  The device properties since the last iteration
 */
/*jslint unparam: true*/
function main(context, previousState, previousProperties) {

    // Restore the global state before generating the new telemetry, so that
    // the telemetry can apply changes using the previous function state.
    restoreSimulation(previousState, previousProperties);

    state.acceleration = vary(AverageAcceleration, AccelerationVariation, MinAcceleration, MaxAcceleration).toFixed(DecimalPrecision);
    state.velocity = vary(AverageVelocity, VelocityVariation, MinVelocity, MaxVelocity).toFixed(DecimalPrecision);

    // Use the last coordinates to calculate the next set with a given variation
    var coords = varylocation(Number(state.latitude), Number(state.longitude), DistanceVariation);
    state.latitude = Number(coords.latitude).toFixed(GeoSpatialPrecision);
    state.longitude = Number(coords.longitude).toFixed(GeoSpatialPrecision);

    // Fluctuate altitude between given variation constant by more or less
    state.altitude = vary(AverageAltitude, AltitudeVariation, AverageAltitude - AltitudeVariation, AverageAltitude + AltitudeVariation).toFixed(DecimalPrecision);

    return state;
}

/**
 * Generate a random geolocation at some distance (in miles)
 * from a given location
 */
function varylocation(latitude, longitude, distance) {
    // Convert to meters, use Earth radius, convert to radians
    var radians = (distance * 1609.344 / 6378137) * (180 / Math.PI);
    return {
        latitude: latitude + radians,
        longitude: longitude + radians / Math.cos(latitude * Math.PI / 180)
    };
}
```

## <a name="create-a-method-script-file"></a>Создание файла для скрипта метода

Скрипты методов похожи на скрипты поведения. Они определяют поведение устройства при вызове из облака конкретного метода устройства.

Скрипт recall для нашего дрона устанавливает сохраненные значения в качестве координат дрона, имитируя возвращение дрона к стартовому положению.

Скопируйте приведенный ниже код JavaScript в текстовый редактор и сохраните его в файл **droneRecall-method.js**.

### <a name="device-model-javascript-simulation-example"></a>Пример модели имитации устройства на JavaScript

```JavaScript
"use strict";

// Default state
var state = {
    velocity: 0.0,
    velocity_unit: "mm/sec",
    acceleration: 0.0,
    acceleration_unit: "mm/sec^2",
    latitude: 0.0,
    longitude: 0.0,
    altitude: 0.0
};

// Default device properties
var properties = {};

/**
 * Restore the global state using data from the previous iteration.
 *
 * @param previousState device state from the previous iteration
 * @param previousProperties device properties from the previous iteration
 */
function restoreSimulation(previousState, previousProperties) {
    // If the previous state is null, force a default state
    if (previousState) {
        state = previousState;
    } else {
        log("Using default state");
    }

    if (previousProperties) {
        properties = previousProperties;
    } else {
        log("Using default properties");
    }
}

/**
 * Entry point function called by the simulation engine.
 *
 * @param context        The context contains current time, device model and id, not used
 * @param previousState  The device state since the last iteration, not used
 * @param previousProperties  The device properties since the last iteration
 */
/*jslint unparam: true*/
function main(context, previousState, previousProperties) {

    // Restore the global device properties and the global state before
    // generating the new telemetry, so that the telemetry can apply changes
    // using the previous function state.
    restoreSimulation(previousState, previousProperties);

    //simulate the behavior of a drone when recalled
  state.latitude = 47.476075;
  state.longitude = -122.192026;
  return state;
}
```

## <a name="debugging-script-files"></a>Отладка файлов сценария

Вы не можете подключить отладчик к выполняемому файлу поведения, но отладочную информацию можно сохранять в журнал службы с помощью функции **log**. Встретив синтаксические ошибки, интерпретатор завершает работу и сохраняет в журнал сведения о возникшем исключении.

Пример записи в журнал.

```JavaScript
function main(context, state) {

    log("This message will appear in the service logs.");

    log(context.deviceId);

    if (typeof(state) !== "undefined" && state !== null) {
        log("Previous value: " + state.temperature);
    }

    // ...

    return updateState;
}
```

## <a name="deploy-an-advanced-device-model"></a>Развертывание расширенной модели устройства

Чтобы развернуть расширенную модель устройства, передайте файлы в экземпляр симулятора устройств:

В строке меню выберите **Device models** (Модели устройств). На странице **Device models** (Модели устройств) перечислены доступные модели в этом экземпляре симулятора устройств:

![Модель устройства](media/iot-accelerators-device-simulation-advanced-device/devicemodelnav.png)

Щелкните **+ Add Device Models** (+ Добавить модель устройства) в правом верхнем углу страницы:

![Добавление модели устройства](media/iot-accelerators-device-simulation-advanced-device/devicemodels.png)

Щелкните **Дополнительно**, чтобы открыть вкладку моделей устройств:

![Вкладка «Дополнительно»](media/iot-accelerators-device-simulation-advanced-device/advancedtab.png)

Щелкните **Обзор** и выберите файлы JSON и JavaScript, которые вы создали ранее. Обязательно выберите все три файла. Если любой из них будет отсутствовать, проверка не пройдет:

![Обзор файлов](media/iot-accelerators-device-simulation-advanced-device/browse.png)

Если файлы успешно пройдут проверку, щелкните **Сохранить**. Теперь модель устройства готова для использования в симуляторе. Если что-то не так, исправьте ошибки и повторно отправьте файлы:

![Сохранять](media/iot-accelerators-device-simulation-advanced-device/validated.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом практическом руководстве вы познакомились с файлами модели устройства, которые используются симулятором устройств, и научились создавать расширенную модель устройства. Теперь вы можете перейти к руководству [Использование службы "Аналитика временных рядов Azure" для визуализации данных телеметрии, отправленных из акселератора решений для имитации устройств](./iot-accelerators-device-simulation-time-series-insights.md)