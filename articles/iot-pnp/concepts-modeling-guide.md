---
title: Общие сведения о моделях устройств самонастраивающийся IoT | Документация Майкрософт
description: Знакомство с языком моделирования двойников Definition Language (ДТДЛ) для устройств IoT самонастраивающийся. В этой статье описываются примитивные и сложные типы, а также шаблоны, которые используют компоненты и наследование, а также семантические типы. В этой статье приводятся рекомендации по выбору идентификатора модели двойникаа устройств и поддержке средств для создания моделей.
author: dominicbetts
ms.author: dobett
ms.date: 03/09/2021
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: a3389408b0942875aa7d760f1c514b995e381f9c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104609475"
---
# <a name="iot-plug-and-play-modeling-guide"></a>Руководством по моделированию IoT самонастраивающийся

В основе центра Интернета вещей самонастраивающийся — это _модель_ устройства, которая описывает возможности устройства для приложения, поддерживающего Самонастраивающийся IOT. Эта модель структурирована как набор интерфейсов, определяющих:

- _Свойства_, которые отражают характеристики состояния устройства или другой сущности, доступные только для чтения или только для записи. Например, серийный номер устройства может быть свойством только для чтения, а требуемая температура для термостата — свойством только для записи.
- Поля _телеметрии_ , определяющие данные, порожденные устройством, данные являются обычным потоком считывания датчиков, случайной ошибкой или информационным сообщением.
- _Команды_ описывают функции или операции, которые можно выполнить на устройстве. Например, можно определить команду для перезапуска шлюза или создания снимка дистанционно управляемой камерой.

Дополнительные сведения о том, как IoT самонастраивающийся использует модели устройств, см. в статье [Руководство разработчика устройств iot Самонастраивающийся](concepts-developer-guide-device.md) и [руководство для разработчиков IOT Самонастраивающийся Service](concepts-developer-guide-service.md).

Для определения модели используется язык определения Digital двойников Definition Language (ДТДЛ). ДТДЛ использует вариант JSON, именуемый [JSON-LD](https://json-ld.org/). В следующем фрагменте кода показана модель для устройства термостата, которое:

- Имеет уникальный идентификатор модели: `dtmi:com:example:Thermostat;1` .
- Отправляет данные телеметрии температуры.
- Имеет свойство, доступное для записи, для задания целевой температуры.
- Свойство доступно только для чтения, чтобы сообщить о максимальной температуре с момента последней перезагрузки.
- Реагирует на команду, которая запрашивает максимальную, минимальную и среднюю температуры за период времени.

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:com:example:Thermostat;1",
  "@type": "Interface",
  "displayName": "Thermostat",
  "description": "Reports current temperature and provides desired temperature control.",
  "contents": [
    {
      "@type": [
        "Telemetry",
        "Temperature"
      ],
      "name": "temperature",
      "displayName": "Temperature",
      "description": "Temperature in degrees Celsius.",
      "schema": "double",
      "unit": "degreeCelsius"
    },
    {
      "@type": [
        "Property",
        "Temperature"
      ],
      "name": "targetTemperature",
      "schema": "double",
      "displayName": "Target Temperature",
      "description": "Allows to remotely specify the desired target temperature.",
      "unit": "degreeCelsius",
      "writable": true
    },
    {
      "@type": [
        "Property",
        "Temperature"
      ],
      "name": "maxTempSinceLastReboot",
      "schema": "double",
      "unit": "degreeCelsius",
      "displayName": "Max temperature since last reboot.",
      "description": "Returns the max temperature since last device reboot."
    },
    {
      "@type": "Command",
      "name": "getMaxMinReport",
      "displayName": "Get Max-Min report.",
      "description": "This command returns the max, min and average temperature from the specified time to the current time.",
      "request": {
        "name": "since",
        "displayName": "Since",
        "description": "Period to return the max-min report.",
        "schema": "dateTime"
      },
      "response": {
        "name": "tempReport",
        "displayName": "Temperature Report",
        "schema": {
          "@type": "Object",
          "fields": [
            {
              "name": "maxTemp",
              "displayName": "Max temperature",
              "schema": "double"
            },
            {
              "name": "minTemp",
              "displayName": "Min temperature",
              "schema": "double"
            },
            {
              "name": "avgTemp",
              "displayName": "Average Temperature",
              "schema": "double"
            },
            {
              "name": "startTime",
              "displayName": "Start Time",
              "schema": "dateTime"
            },
            {
              "name": "endTime",
              "displayName": "End Time",
              "schema": "dateTime"
            }
          ]
        }
      }
    }
  ]
}
```

Модель термостата имеет один интерфейс. В последующих примерах в этой статье демонстрируются более сложные модели, использующие компоненты и наследование.

В этой статье описывается разработка и создание собственных моделей, а также рассматриваются такие темы, как типы данных, структура модели и средства.

Дополнительные сведения см. в спецификации [Digital двойников Definition Language v2](https://github.com/Azure/opendigitaltwins-dtdl) .

## <a name="model-structure"></a>Структура модели

Свойства, данные телеметрии и команды группируются в интерфейсы. В этом разделе описывается использование интерфейсов для описания простых и сложных моделей с помощью компонентов и наследования.

### <a name="model-ids"></a>Идентификаторы моделей

Каждый интерфейс имеет уникальный идентификатор модели Digital двойника (ДТМИ). В сложных моделях для обнаружения компонентов используется Дтмис. Приложения могут использовать Дтмис, которые устройства отправляют для определения местоположения определений модели в репозитории.

Дтмис должен соответствовать соглашению об именовании, требуемому для [репозитория Самонастраивающийся для модели Интернета вещей](https://github.com/Azure/iot-plugandplay-models):

- Префикс ДТМИ — `dtmi:` .
- Суффикс ДТМИ — это номер версии для модели, например `;2` .
- Текст ДТМИ сопоставляется с папкой и файлом в репозитории модели, в котором хранится модель. Номер версии является частью имени файла.

Например, модель, определяемая параметром ДТМИ, `dtmi:com:Example:Thermostat;2` хранится в файле *дтми/com/example/thermostat-2.js* .

В следующем фрагменте кода показана структура определения интерфейса с уникальным ДТМИ:

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:com:example:Thermostat;2",
  "@type": "Interface",
  "displayName": "Thermostat",
  "description": "Reports current temperature and provides desired temperature control.",
  "contents": [
    ...
  ]
}
```

### <a name="no-components"></a>Нет компонентов

Простая модель, например термостата, показанная ранее, не использует внедренные или каскадные компоненты. Данные телеметрии, свойств и команд определяются в `contents` узле интерфейса.

В следующем примере показана часть простой модели, которая не использует компоненты.

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:com:example:Thermostat;1",
  "@type": "Interface",
  "displayName": "Thermostat",
  "description": "Reports current temperature and provides desired temperature control.",
  "contents": [
    {
      "@type": [
        "Telemetry",
        "Temperature"
      ],
      "name": "temperature",
      "displayName": "Temperature",
      "description": "Temperature in degrees Celsius.",
      "schema": "double",
      "unit": "degreeCelsius"
    },
    {
      "@type": [
        "Property",
...
```

Такие средства, как Azure IoT Explorer и конструктор шаблонов устройств IoT Central, помечают изолированный интерфейс, такой как термостата, как _компонент по умолчанию_.

На следующем снимке экрана показано, как модель отображается в средстве Azure IoT Explorer.

:::image type="content" source="media/concepts-modeling-guide/default-component.png" alt-text="Компонент по умолчанию в Azure IoT Explorer":::

На следующем снимке экрана показано, как модель отображается как компонент по умолчанию в конструкторе шаблонов устройств IoT Central. Выберите **Просмотр удостоверения** , чтобы просмотреть дтми модели:

:::image type="content" source="media/concepts-modeling-guide/iot-central-model.png" alt-text="Снимок экрана, показывающий модель термостата в конструкторе IoT Central":::

Идентификатор модели хранится в свойстве двойникаа устройства, как показано на следующем снимке экрана:

:::image type="content" source="media/concepts-modeling-guide/twin-model-id.png" alt-text="Идентификатор модели в свойстве Digital двойника":::

Модель ДТДЛ без компонентов является полезным упрощением для устройства или модуля IoT Edge с единым набором телеметрии, свойств и команд. Модель, которая не использует компоненты, упрощает перенос существующего устройства или модуля в центр Интернета вещей самонастраивающийся устройство или модуль. вы создаете модель ДТДЛ, которая описывает реальное устройство или модуль без необходимости определять какие-либо компоненты.

> [!TIP]
> Модуль может быть [модулем](../iot-hub/iot-hub-devguide-module-twins.md) устройства или [модулем IOT Edge](../iot-edge/about-iot-edge.md).

### <a name="reuse"></a>Повторное использование

Существует два способа повторного использования определений интерфейсов. Использование нескольких компонентов в модели для ссылки на другие определения интерфейса. Используйте наследование для расширения существующих определений интерфейса.

### <a name="multiple-components"></a>Несколько компонентов

Компоненты позволяют создать интерфейс модели в виде сборки других интерфейсов.

Например, интерфейс [термостата](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json) определяется как модель. Этот интерфейс можно включить в качестве одного или нескольких компонентов при определении [модели контроллера температуры](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json). В следующем примере эти компоненты называются `thermostat1` и `thermostat2` .

Для модели ДТДЛ с несколькими компонентами существует два или более раздела компонентов. Каждый раздел имеет `@type` значение `Component` и явно ссылается на схему, как показано в следующем фрагменте кода:

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:com:example:TemperatureController;1",
  "@type": "Interface",
  "displayName": "Temperature Controller",
  "description": "Device with two thermostats and remote reboot.",
  "contents": [
    {
      "@type": [
        "Telemetry",
        "DataSize"
      ],
      "name": "workingSet",
      "displayName": "Working Set",
      "description": "Current working set of the device memory in KiB.",
      "schema": "double",
      "unit": "kibibyte"
    },
    {
      "@type": "Property",
      "name": "serialNumber",
      "displayName": "Serial Number",
      "description": "Serial number of the device.",
      "schema": "string"
    },
    {
      "@type": "Command",
      "name": "reboot",
      "displayName": "Reboot",
      "description": "Reboots the device after waiting the number of seconds specified.",
      "request": {
        "name": "delay",
        "displayName": "Delay",
        "description": "Number of seconds to wait before rebooting the device.",
        "schema": "integer"
      }
    },
    {
      "@type" : "Component",
      "schema": "dtmi:com:example:Thermostat;1",
      "name": "thermostat1",
      "displayName": "Thermostat One",
      "description": "Thermostat One of Two."
    },
    {
      "@type" : "Component",
      "schema": "dtmi:com:example:Thermostat;1",
      "name": "thermostat2",
      "displayName": "Thermostat Two",
      "description": "Thermostat Two of Two."
    },
    {
      "@type": "Component",
      "schema": "dtmi:azure:DeviceManagement:DeviceInformation;1",
      "name": "deviceInformation",
      "displayName": "Device Information interface",
      "description": "Optional interface with basic device hardware information."
    }
  ]
}
```

Эта модель содержит три компонента, определенные в разделе содержимого — два `Thermostat` `DeviceInformation` компонента и компонент. В раздел содержимого также входят свойства, данные телеметрии и определения команд.

На следующих снимках экрана показано, как эта модель отображается в IoT Central. Свойства, данные телеметрии и определения команд в контроллере температуры отображаются в **компоненте по умолчанию** верхнего уровня. Свойства, данные телеметрии и определения команд для каждого термостата отображаются в определениях компонентов:

:::image type="content" source="media/concepts-modeling-guide/temperature-controller.png" alt-text="Снимок экрана, показывающий шаблон устройства контроллера температуры в IoT Central.":::

:::image type="content" source="media/concepts-modeling-guide/temperature-controller-components.png" alt-text="Снимок экрана, показывающий компоненты термостата в шаблоне устройства контроллера температуры в IoT Central.":::

Сведения о написании кода устройства, взаимодействующего с компонентами, см. в статье [Руководство разработчика для центра Интернета вещей Самонастраивающийся](concepts-developer-guide-device.md).

Сведения о написании кода службы, который взаимоследует с компонентами на устройстве, см. в разделе [Руководство разработчика для центра Интернета вещей Самонастраивающийся](concepts-developer-guide-service.md).

### <a name="inheritance"></a>Наследование

Наследование позволяет повторно использовать возможности в базовых интерфейсах для расширения возможностей интерфейса. Например, несколько моделей устройств могут совместно использовать такие общие возможности, как серийный номер:

:::image type="content" source="media/concepts-modeling-guide/inheritance.png" alt-text="Пример наследования в модели устройства, показывающей термостата и контроллер потока, который совместно используют возможности базового интерфейса." border="false":::

В следующем фрагменте кода показана модель ДТМЛ, которая использует `extends` ключевое слово для определения отношения наследования, показанного на предыдущей схеме.

```json
[
  {
    "@context": "dtmi:dtdl:context;2",
    "@id": "dtmi:com:example:Thermostat;1",
    "@type": "Interface",
    "contents": [
      {
        "@type": "Telemetry",
        "name": "temperature",
        "schema": "double",
        "unit": "degreeCelsius"
      },
      {
        "@type": "Property",
        "name": "targetTemperature",
        "schema": "double",
        "unit": "degreeCelsius",
        "writable": true
      }
    ],
    "extends": [
      "dtmi:com:example:baseDevice;1"
    ]
  },
  {
    "@context": "dtmi:dtdl:context;2",
    "@id": "dtmi:com:example:baseDevice;1",
    "@type": "Interface",
    "contents": [
      {
        "@type": "Property",
        "name": "SerialNumber",
        "schema": "double",
        "writable": false
      }
    ]
  }
]
```

На следующем снимке экрана показана эта модель в среде шаблона устройства IoT Central:

:::image type="content" source="media/concepts-modeling-guide/iot-central-inheritance.png" alt-text="Снимок экрана, демонстрирующий наследование интерфейса в IoT Central":::

При написании кода устройства или на стороне службы коду не нужно предпринимать никаких специальных действий для работы с унаследованными интерфейсами. В примере, приведенном в этом разделе, код устройства сообщает серийный номер, как если бы он был частью интерфейса термостата.

### <a name="tips"></a>"Советы"

При создании модели можно сочетать компоненты и наследование. На следующей диаграмме показана `thermostat` модель, наследуемая от `baseDevice` интерфейса. `baseDevice`Интерфейс содержит компонент, который сам наследуется от другого интерфейса:

:::image type="content" source="media/concepts-modeling-guide/inheritance-components.png" alt-text="Схема, показывающая модель, которая использует как компоненты, так и наследование." border="false":::

В следующем фрагменте кода показана модель ДТМЛ, которая использует `extends` `component` Ключевые слова и для определения отношения наследования и использования компонентов, показанных на предыдущей схеме:

```json
[
  {
    "@context": "dtmi:dtdl:context;2",
    "@id": "dtmi:com:example:Thermostat;1",
    "@type": "Interface",
    "contents": [
      {
        "@type": "Telemetry",
        "name": "temperature",
        "schema": "double",
        "unit": "degreeCelsius"
      },
      {
        "@type": "Property",
        "name": "targetTemperature",
        "schema": "double",
        "unit": "degreeCelsius",
        "writable": true
      }
    ],
    "extends": [
      "dtmi:com:example:baseDevice;1"
    ]
  },
  {
    "@context": "dtmi:dtdl:context;2",
    "@id": "dtmi:com:example:baseDevice;1",
    "@type": "Interface",
    "contents": [
      {
        "@type": "Property",
        "name": "SerialNumber",
        "schema": "double",
        "writable": false
      },
      {
        "@type" : "Component",
        "schema": "dtmi:com:example:baseComponent;1",
        "name": "baseComponent"
      }
    ]
  }
]
```

## <a name="data-types"></a>Типы данных

Используйте типы данных для определения телеметрии, свойств и параметров команды. Типы данных могут быть простыми или сложными. Сложные типы типа «тип» используют примитивы или другие сложные виды. Максимальная глубина для сложных типов составляет пять уровней.

### <a name="primitive-types"></a>Примитивные типы

В следующей таблице показан набор типов-примитивов, которые можно использовать.

| Тип-примитив | Описание |
| --- | --- |
| `boolean` | Логическое значение. |
| `date` | Полная дата, как определено в [разделе 5,6 RFC 3339](https://tools.ietf.org/html/rfc3339#section-5.6) |
| `dateTime` | Дата и время, как определено в [RFC 3339](https://tools.ietf.org/html/rfc3339) |
| `double` | 8-байтная плавающая точка в формате IEEE |
| `duration` | Длительность в формате ISO 8601 |
| `float` | 4-байтовая плавающая точка IEEE |
| `integer` | 4-байтовое целое число со знаком |
| `long` | 8-байтовое целое число со знаком |
| `string` | Строка UTF8 |
| `time` | Полное время, как определено в [разделе 5,6 RFC 3339](https://tools.ietf.org/html/rfc3339#section-5.6) |

В следующем фрагменте кода показан пример определения телеметрии, в котором используется `double` тип в `schema` поле:

```json
{
  "@type": "Telemetry",
  "name": "temperature",
  "displayName": "Temperature",
  "schema": "double"
}
```

### <a name="complex-datatypes"></a>Сложные типы

Сложные типы данных — это один из *массивов*, *перечислений*, *карт*, *объектов* или одного из геопространственных типов.

#### <a name="arrays"></a>Массивы

Массив — это индексируемый тип данных, в котором все элементы имеют один и тот же тип. Тип элемента может быть примитивным или сложным типом.

В следующем фрагменте кода показан пример определения телеметрии, в котором используется `Array` тип в `schema` поле. Элементы массива являются логическими значениями:

```json
{
  "@type": "Telemetry",
  "name": "ledState",
  "schema": {
    "@type": "Array",
    "elementSchema": "boolean"
  }
}
```

#### <a name="enumerations"></a>Перечисления

Перечисление описывает тип с набором именованных меток, которые сопоставляются со значениями. Значения могут быть либо целыми числами, либо строками, но метки всегда являются строками.

В следующем фрагменте кода показан пример определения телеметрии, в котором используется `Enum` тип в `schema` поле. Значения в перечислении являются целыми числами:

```json
{
  "@type": "Telemetry",
  "name": "state",
  "schema": {
    "@type": "Enum",
    "valueSchema": "integer",
    "enumValues": [
      {
        "name": "offline",
        "displayName": "Offline",
        "enumValue": 1
      },
      {
        "name": "online",
        "displayName": "Online",
        "enumValue": 2
      }
    ]
  }
}
```

#### <a name="maps"></a>Maps

Map — это тип с парами "ключ-значение", в котором все значения имеют одинаковый тип. Ключ в сопоставлении должен быть строкой. Значения в сопоставлении могут быть любого типа, включая другой сложный тип.

В следующем фрагменте кода показан пример определения свойства, использующего `Map` тип в `schema` поле. Значения на карте являются строками:

```json
{
  "@type": "Property",
  "name": "modules",
  "writable": true,
  "schema": {
    "@type": "Map",
    "mapKey": {
      "name": "moduleName",
      "schema": "string"
    },
    "mapValue": {
      "name": "moduleState",
      "schema": "string"
    }
  }
}
```

### <a name="objects"></a>Объекты

Тип объекта состоит из именованных полей. Типы полей в сопоставлении объектов могут быть простыми или сложными типами.

В следующем фрагменте кода показан пример определения телеметрии, в котором используется `Object` тип в `schema` поле. Поля в объекте имеют `dateTime` `duration` типы, и `string` :

```json
{
  "@type": "Telemetry",
  "name": "monitor",
  "schema": {
    "@type": "Object",
    "fields": [
      {
        "name": "start",
        "schema": "dateTime"
      },
      {
        "name": "interval",
        "schema": "duration"
      },
      {
        "name": "status",
        "schema": "string"
      }
    ]
  }
}
```

#### <a name="geospatial-types"></a>Геопространственные типы

Дтдл предоставляет набор геопространственных типов, основанных на географической [JSON](https://geojson.org/), для моделирования структур географических данных: `point` , `multiPoint` , `lineString` ,, `multiLineString` `polygon` и `multiPolygon` . Эти типы являются предопределенными вложенными структурами массивов, объектов и перечислений.

В следующем фрагменте кода показан пример определения телеметрии, в котором используется `point` тип в `schema` поле:

```json
{
  "@type": "Telemetry",
  "name": "location",
  "schema": "point"
}
```

Поскольку Геопространственные типы основаны на массивах, они не могут использоваться в определениях свойств.

## <a name="semantic-types"></a>Семантические типы

Тип данных определения свойства или телеметрии определяет формат, в котором устройство обменивается данными со службой. Семантический тип предоставляет сведения о телеметрии и свойствах, которые приложение может использовать для определения способа обработки или отображения значения. Каждый семантический тип имеет одно или несколько связанных единиц. Например, градусы Цельсия и Фаренгейта являются единицами для семантического типа температуры. IoT Central панели мониторинга и аналитика могут использовать сведения о семантическом типе для определения способа построения данных телеметрии или значений свойств и единиц отображения. Сведения об использовании средства синтаксического анализа модели для чтения семантических типов см. в разделе [Знакомство с анализатором цифровых двойников Model Parser](concepts-model-parser.md).

В следующем фрагменте кода показан пример определения телеметрии, включающего сведения о семантическом типе. Семантический тип `Temperature` добавляется в `@type` массив, а `unit` значение `degreeCelsius` — один из допустимых единиц для семантического типа:

```json
{
  "@type": [
    "Telemetry",
    "Temperature"
  ],
  "name": "temperature",
  "schema": "double",
  "unit": "degreeCelsius"
}
```

## <a name="localization"></a>Локализация

Приложения, такие как IoT Central, используют сведения в модели для динамического создания пользовательского интерфейса на основе данных, которыми обмениваются устройства IoT самонастраивающийся. Например, плитки на панели мониторинга могут отображать имена и описания для телеметрии, свойств и команд.

Необязательные `description` `displayName` поля и в модели содержат строки, предназначенные для использования в пользовательском интерфейсе. Эти поля могут содержать локализованные строки, которые приложение может использовать для отображения локализованного пользовательского интерфейса.

В следующем фрагменте кода показан пример определения телеметрии температуры, включающего локализованные строки:

```json
{
  "@type": [
    "Telemetry",
    "Temperature"
  ],
  "description": {
    "en": "Temperature in degrees Celsius.",
    "it": "Temperatura in gradi Celsius."
  },
  "displayName": {
    "en": "Temperature",
    "it": "Temperatura"
  },
  "name": "temperature",
  "schema": "double",
  "unit": "degreeCelsius"
}
```

Добавление локализованных строк является необязательным. В следующем примере используется только один язык по умолчанию:

```json
{
  "@type": [
    "Telemetry",
    "Temperature"
  ],
  "description": "Temperature in degrees Celsius.",
  "displayName": "Temperature",
  "name": "temperature",
  "schema": "double",
  "unit": "degreeCelsius"
}
```

## <a name="lifecycle-and-tools"></a>Жизненный цикл и средства

Четыре этапа жизненного цикла модели устройства — это создание, публикация, использование и управление версиями.

### <a name="author"></a>Автор

Модели устройств ДТМЛ — это документы JSON, которые можно создать в текстовом редакторе. Однако в IoT Central можно использовать среду с графическим интерфейсом шаблона устройства, чтобы создать модель ДТМЛ. В IoT Central можно:

- Создание интерфейсов, определяющих свойства, данные телеметрии и команды.
- Используйте компоненты для объединения нескольких интерфейсов.
- Определение отношений наследования между интерфейсами.
- Импорт и экспорт файлов модели ДТМЛ.

Дополнительные сведения см. [в разделе Определение нового типа устройства IOT в приложении IOT Central Azure](../iot-central/core/howto-set-up-template.md).

[Редактор дтдл для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.vscode-dtdl) предоставляет текстовую среду редактирования с проверкой синтаксиса и автозаполнением для более точного управления процессом создания модели.

### <a name="publish"></a>Публикация

Чтобы модели ДТМЛ были общими и обнаруживаемыми, их необходимо опубликовать в репозитории моделей устройств.

Перед публикацией модели в общедоступном репозитории можно использовать `dmr-client` средства для проверки модели.

Дополнительные сведения см. в разделе [хранилище моделей устройств](concepts-model-repository.md).

### <a name="use"></a>Использование

Приложения, такие как IoT Central, используют модели устройств. В IoT Central модель является частью шаблона устройства, который описывает возможности устройства. IoT Central использует шаблон устройства для динамического создания пользовательского интерфейса для устройства, включая панели мониторинга и аналитику.

Пользовательское решение может использовать [средство синтаксического анализа модели Digital двойников](concepts-model-parser.md) для понимания возможностей устройства, реализующего модель. Дополнительные сведения см. в статье [использование Интернета вещей Самонастраивающийся моделях в решении IOT](concepts-model-discovery.md).

### <a name="version"></a>Версия

Чтобы гарантировать, что устройства и серверные решения, использующие модели, продолжают работать, опубликованные модели являются неизменяемыми.

ДТМИ включает номер версии, который можно использовать для создания нескольких версий модели. Устройства и серверные решения могут использовать определенную версию, для использования которой они предназначены.

IoT Central реализует дополнительные правила управления версиями для моделей устройств. Если версия шаблона устройства и его модели в IoT Central, можно перенести устройства из предыдущих версий в более поздние версии. Однако перенесенные устройства не могут использовать новые возможности без обновления встроенного по. Дополнительные сведения см. в разделе [Создание новой версии шаблона устройства](../iot-central/core/howto-version-device-template.md).

## <a name="limits-and-constraints"></a>Ограничения

В следующем списке приведены некоторые ключевые ограничения и ограничения для моделей.

- В настоящее время максимальная глубина массивов, карт и объектов составляет пять уровней глубины.
- Нельзя использовать массивы в определениях свойств.
- Можно расширить интерфейсы до 10 уровней.
- Интерфейс может расширять не более двух других интерфейсов.
- Компонент не может содержать другой компонент.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали о моделировании устройств, ниже приведены некоторые дополнительные ресурсы.

- [Установка и использование средств разработки ДТДЛ](howto-use-dtdl-authoring-tools.md)
- [Digital двойников Definition Language v2 (ДТДЛ)](https://github.com/Azure/opendigitaltwins-dtdl)
- [Репозитории модели](./concepts-model-repository.md)
