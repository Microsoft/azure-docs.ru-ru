---
title: Полезные данные телеметрии, свойств и команд в Azure IoT Central | Документация Майкрософт
description: Шаблоны устройств IoT Central Azure позволяют указать данные телеметрии, свойства и команды, которые должны быть реализованы устройством. Сведения о формате данных, которые устройство может обменивать с IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 12/19/2020
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.custom: device-developer
ms.openlocfilehash: f027b2d41f63b5aa7ea3df87e06224abd629799b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100535320"
---
# <a name="telemetry-property-and-command-payloads"></a>Полезные данные телеметрии, свойств и команд

_Эта статья предназначена для разработчиков устройств_

Шаблон устройства в Azure IoT Central — это схема, которая определяет:

* Данные телеметрии, которые устройство отправляет в IoT Central.
* Свойства, с которыми синхронизируется устройство IoT Central.
* Команды, которые IoT Central вызовы на устройстве.

В этой статье описывается, для разработчиков устройств, полезных данных JSON, которые устройства отправляют и получают для телеметрии, свойств и команд, определенных в шаблоне устройства.

В этой статье не описываются все возможные типы полезных данных телеметрии, свойств и команд, но в примерах показаны все ключи.

В каждом примере показан фрагмент кода из модели устройства, который определяет тип и примеры полезных данных JSON, чтобы продемонстрировать, как устройство должно взаимодействовать с IoT Central приложением.

> [!NOTE]
> IoT Central принимает любой допустимый JSON, но его можно использовать только для визуализаций, если он соответствует определению в модели устройства. Вы можете экспортировать данные, не соответствующие определению, [в разделе Экспорт данных IOT в назначения в Azure](howto-export-data.md).

JSON-файл, определяющий модель устройства, использует [язык определения цифровых двойника (дтдл) версии 2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md).

Пример кода устройства, в котором показаны некоторые из этих полезных данных, см. в руководстве по [созданию и подключению клиентского приложения к IOT Centralному приложению Azure](tutorial-connect-device.md) .

## <a name="view-raw-data"></a>Просмотр необработанных данных

IoT Central позволяет просматривать необработанные данные, которые устройство отправляет в приложение. Это представление полезно для устранения неполадок, связанных с полезной нагрузкой, отправленной с устройства. Просмотр необработанных данных, отправляемых устройством:

1. Перейдите на устройство со страницы **устройства** .

1. Перейдите на вкладку **необработанные данные** :

    :::image type="content" source="media/concepts-telemetry-properties-commands/raw-data.png" alt-text="Представление &quot;Необработанные данные&quot;":::

    В этом представлении можно выбрать отображаемые столбцы и задать диапазон времени для просмотра. В столбце **Unmodeled data** (Немоделированные данные) отображаются данные с устройства, которые не соответствуют ни одному из определений свойств или телеметрии в шаблоне устройства.

## <a name="telemetry"></a>Телеметрия

### <a name="primitive-types"></a>Примитивные типы

В этом разделе приведены примеры типов данных телеметрии, которые передаются в приложение IoT Central в потоке устройства.

В следующем фрагменте кода из модели устройства показано определение `boolean` типа телеметрии.

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "BooleanTelemetry"
  },
  "name": "BooleanTelemetry",
  "schema": "boolean"
}
```

Клиент устройства должен отправить данные телеметрии как JSON, который выглядит, как в следующем примере:

```json
{ "BooleanTelemetry": true }
```

В следующем фрагменте кода из модели устройства показано определение `string` типа телеметрии.

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "StringTelemetry"
  },
  "name": "StringTelemetry",
  "schema": "string"
}
```

Клиент устройства должен отправить данные телеметрии как JSON, который выглядит, как в следующем примере:

```json
{ "StringTelemetry": "A string value - could be a URL" }
```

В следующем фрагменте кода из модели устройства показано определение `integer` типа телеметрии.

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "IntegerTelemetry"
  },
  "name": "IntegerTelemetry",
  "schema": "integer"
}

```

Клиент устройства должен отправить данные телеметрии как JSON, который выглядит, как в следующем примере:

```json
{ "IntegerTelemetry": 23 }
```

В следующем фрагменте кода из модели устройства показано определение `double` типа телеметрии.

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "DoubleTelemetry"
  },
  "name": "DoubleTelemetry",
  "schema": "double"
}
```

Клиент устройства должен отправить данные телеметрии как JSON, который выглядит, как в следующем примере:

```json
{ "DoubleTelemetry": 56.78 }
```

В следующем фрагменте кода из модели устройства показано определение `dateTime` типа телеметрии.

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "DateTimeTelemetry"
  },
  "name": "DateTimeTelemetry",
  "schema": "dateTime"
}
```

Клиент устройства должен отправить данные телеметрии как JSON, который выглядит так, как в следующем примере `DateTime` : типы должны быть в формате ISO 8061.

```json
{ "DateTimeTelemetry": "2020-08-30T19:16:13.853Z" }
```

В следующем фрагменте кода из модели устройства показано определение `duration` типа телеметрии.

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "DurationTelemetry"
  },
  "name": "DurationTelemetry",
  "schema": "duration"
}
```

Клиент устройства должен отправить данные телеметрии как JSON, который выглядит, как в следующем примере: длительность должна быть в формате ISO 8601.

```json
{ "DurationTelemetry": "PT10H24M6.169083011336625S" }
```

### <a name="complex-types"></a>Сложные типы

В этом разделе приведены примеры сложных типов телеметрии, которые передаются в приложение IoT Central в потоке устройства.

В следующем фрагменте кода из модели устройства показано определение `geopoint` типа телеметрии.

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "GeopointTelemetry"
  },
  "name": "GeopointTelemetry",
  "schema": "geopoint"
}
```

> [!NOTE]
> Тип схемы **геоточек** не является частью [спецификации языка определения цифровых двойников](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md). В настоящее время IoT Central поддерживает тип схемы **географической** **точки** и тип семантики расположения для обеспечения обратной совместимости.

Клиент устройства должен отправить данные телеметрии как JSON, который выглядит, как в следующем примере. IoT Central отображает значение в виде ПИН-кода на карте:

```json
{
  "GeopointTelemetry": {
    "lat": 47.64263,
    "lon": -122.13035,
    "alt": 0
  }
}
```

В следующем фрагменте кода из модели устройства показано определение `Enum` типа телеметрии.

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "EnumTelemetry"
  },
  "name": "EnumTelemetry",
  "schema": {
    "@type": "Enum",
    "displayName": {
      "en": "Enum"
    },
    "valueSchema": "integer",
    "enumValues": [
      {
        "displayName": {
          "en": "Item1"
        },
        "enumValue": 0,
        "name": "Item1"
      },
      {
        "displayName": {
          "en": "Item2"
        },
        "enumValue": 1,
        "name": "Item2"
      },
      {
        "displayName": {
          "en": "Item3"
        },
        "enumValue": 2,
        "name": "Item3"
      }
    ]
  }
}
```

Клиент устройства должен отправить данные телеметрии как JSON, который выглядит, как в следующем примере. Возможные значения: `0` , `1` и `2` , которые отображаются в IOT Central как `Item1` , `Item2` и `Item3` :

```json
{ "EnumTelemetry": 1 }
```

В следующем фрагменте кода из модели устройства показано определение `Object` типа телеметрии. Этот объект содержит три поля с типами `dateTime` , `integer` и `Enum` :

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "ObjectTelemetry"
  },
  "name": "ObjectTelemetry",
  "schema": {
    "@type": "Object",
    "displayName": {
      "en": "Object"
    },
    "fields": [
      {
        "displayName": {
          "en": "Property1"
        },
        "name": "Property1",
        "schema": "dateTime"
      },
      {
        "displayName": {
          "en": "Property2"
        },
        "name": "Property2",
        "schema": "integer"
      },
      {
        "displayName": {
          "en": "Property3"
        },
        "name": "Property3",
        "schema": {
          "@type": "Enum",
          "displayName": {
            "en": "Enum"
          },
          "valueSchema": "integer",
          "enumValues": [
            {
              "displayName": {
                "en": "Item1"
              },
              "enumValue": 0,
              "name": "Item1"
            },
            {
              "displayName": {
                "en": "Item2"
              },
              "enumValue": 1,
              "name": "Item2"
            },
            {
              "displayName": {
                "en": "Item3"
              },
              "enumValue": 2,
              "name": "Item3"
            }
          ]
        }
      }
    ]
  }
}
```

Клиент устройства должен отправить данные телеметрии как JSON, который выглядит, как в следующем примере. `DateTime` типы должны соответствовать стандартам ISO 8061. Возможные значения для `Property3` : `0` , `1` и, которые отображаются в IOT Central как `Item1` , `Item2` и `Item3` :

```json
{
  "ObjectTelemetry": {
      "Property1": "2020-09-09T03:36:46.195Z",
      "Property2": 37,
      "Property3": 2
  }
}
```

В следующем фрагменте кода из модели устройства показано определение `vector` типа телеметрии.

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "VectorTelemetry"
  },
  "name": "VectorTelemetry",
  "schema": "vector"
}
```

Клиент устройства должен отправить данные телеметрии как JSON, который выглядит, как в следующем примере:

```json
{
  "VectorTelemetry": {
    "x": 74.72395045538597,
    "y": 74.72395045538597,
    "z": 74.72395045538597
  }
}
```

### <a name="event-and-state-types"></a>Типы событий и состояний

В этом разделе приведены примеры событий телеметрии и состояний, которые устройство отправляет в приложение IoT Central.

В следующем фрагменте кода из модели устройства показано определение `integer` типа события:

```json
{
  "@type": [
    "Telemetry",
    "Event"
  ],
  "displayName": {
    "en": "IntegerEvent"
  },
  "name": "IntegerEvent",
  "schema": "integer"
}
```

Клиент устройства должен отправить данные события как JSON, который выглядит, как в следующем примере:

```json
{ "IntegerEvent": 74 }
```

В следующем фрагменте кода из модели устройства показано определение `integer` типа состояния:

```json
{
  "@type": [
    "Telemetry",
    "State"
  ],
  "displayName": {
    "en": "IntegerState"
  },
  "name": "IntegerState",
  "schema": {
    "@type": "Enum",
    "valueSchema": "integer",
    "enumValues": [
      {
        "displayName": {
          "en": "Level1"
        },
        "enumValue": 1,
        "name": "Level1"
      },
      {
        "displayName": {
          "en": "Level2"
        },
        "enumValue": 2,
        "name": "Level2"
      },
      {
        "displayName": {
          "en": "Level3"
        },
        "enumValue": 3,
        "name": "Level3"
      }
    ]
  }
}
```

Клиент устройства должен отправить состояние как JSON, как показано в следующем примере. Возможные значения целочисленного состояния `1` `2` :, или `3` .

```json
{ "IntegerState": 2 }
```

## <a name="properties"></a>Свойства

> [!NOTE]
> Форматы полезных данных для свойств применяются к приложениям, созданным в или после 07/14/2020.

### <a name="primitive-types"></a>Примитивные типы

В этом разделе приведены примеры типов простых свойств, которые устройство отправляет в приложение IoT Central.

В следующем фрагменте кода из модели устройства показано определение `boolean` типа свойства:

```json
{
  "@type": "Property",
  "displayName": {
    "en": "BooleanProperty"
  },
  "name": "BooleanProperty",
  "schema": "boolean",
  "writable": false
}
```

Клиент устройства должен отправить полезные данные JSON, как в следующем примере, как сообщаемое свойство в двойникае устройства:

```json
{ "BooleanProperty": false }
```

В следующем фрагменте кода из модели устройства показано определение `boolean` типа свойства:

```json
{
  "@type": "Property",
  "displayName": {
    "en": "LongProperty"
  },
  "name": "LongProperty",
  "schema": "long",
  "writable": false
}
```

Клиент устройства должен отправить полезные данные JSON, как в следующем примере, как сообщаемое свойство в двойникае устройства:

```json
{ "LongProperty": 439 }
```

В следующем фрагменте кода из модели устройства показано определение `date` типа свойства:

```json
{
  "@type": "Property",
  "displayName": {
    "en": "DateProperty"
  },
  "name": "DateProperty",
  "schema": "date",
  "writable": false
}
```

Клиент устройства должен отправить полезные данные JSON, как в следующем примере, как сообщаемое свойство в двойникае устройства. `Date` типы должны соответствовать стандартам ISO 8061:

```json
{ "DateProperty": "2020-05-17" }
```

В следующем фрагменте кода из модели устройства показано определение `duration` типа свойства:

```json
{
  "@type": "Property",
  "displayName": {
    "en": "DurationProperty"
  },
  "name": "DurationProperty",
  "schema": "duration",
  "writable": false
}
```

Клиент устройства должен отправить полезные данные JSON, которые выглядят так, как в следующем примере, как сообщаемое свойство в двойникае устройства. Длительность должна соответствовать требованиям ISO 8601:

```json
{ "DurationProperty": "PT10H24M6.169083011336625S" }
```

В следующем фрагменте кода из модели устройства показано определение `float` типа свойства:

```json
{
  "@type": "Property",
  "displayName": {
    "en": "FloatProperty"
  },
  "name": "FloatProperty",
  "schema": "float",
  "writable": false
}
```

Клиент устройства должен отправить полезные данные JSON, как в следующем примере, как сообщаемое свойство в двойникае устройства:

```json
{ "FloatProperty": 1.9 }
```

В следующем фрагменте кода из модели устройства показано определение `string` типа свойства:

```json
{
  "@type": "Property",
  "displayName": {
    "en": "StringProperty"
  },
  "name": "StringProperty",
  "schema": "string",
  "writable": false
}
```

Клиент устройства должен отправить полезные данные JSON, как в следующем примере, как сообщаемое свойство в двойникае устройства:

```json
{ "StringProperty": "A string value - could be a URL" }
```

### <a name="complex-types"></a>Сложные типы

В этом разделе приведены примеры сложных типов свойств, которые устройство отправляет в приложение IoT Central.

В следующем фрагменте кода из модели устройства показано определение `geopoint` типа свойства:

```json
{
  "@type": "Property",
  "displayName": {
    "en": "GeopointProperty"
  },
  "name": "GeopointProperty",
  "schema": "geopoint",
  "writable": false
}
```

> [!NOTE]
> Тип схемы **геоточек** не является частью [спецификации языка определения цифровых двойников](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md). В настоящее время IoT Central поддерживает тип схемы **географической** **точки** и тип семантики расположения для обеспечения обратной совместимости.

Клиент устройства должен отправить полезные данные JSON, как в следующем примере, как сообщаемое свойство в двойникае устройства:

```json
{
  "GeopointProperty": {
    "lat": 47.64263,
    "lon": -122.13035,
    "alt": 0
  }
}
```

В следующем фрагменте кода из модели устройства показано определение `Enum` типа свойства:

```json
{
  "@type": "Property",
  "displayName": {
    "en": "EnumProperty"
  },
  "name": "EnumProperty",
  "writable": false,
  "schema": {
    "@type": "Enum",
    "displayName": {
      "en": "Enum"
    },
    "valueSchema": "integer",
    "enumValues": [
      {
        "displayName": {
          "en": "Item1"
        },
        "enumValue": 0,
        "name": "Item1"
      },
      {
        "displayName": {
          "en": "Item2"
        },
        "enumValue": 1,
        "name": "Item2"
      },
      {
        "displayName": {
          "en": "Item3"
        },
        "enumValue": 2,
        "name": "Item3"
      }
    ]
  }
}
```

Клиент устройства должен отправить полезные данные JSON, как в следующем примере, как сообщаемое свойство в двойникае устройства. Возможные значения: `0` , `1` и, которые отображаются в IOT Central как `Item1` , `Item2` и `Item3` :

```json
{ "EnumProperty": 1 }
```

В следующем фрагменте кода из модели устройства показано определение `Object` типа свойства. Этот объект содержит два поля с типами `string` и `integer` :

```json
{
  "@type": "Property",
  "displayName": {
    "en": "ObjectProperty"
  },
  "name": "ObjectProperty",
  "writable": false,
  "schema": {
    "@type": "Object",
    "displayName": {
      "en": "Object"
    },
    "fields": [
      {
        "displayName": {
          "en": "Field1"
        },
        "name": "Field1",
        "schema": "integer"
      },
      {
        "displayName": {
          "en": "Field2"
        },
        "name": "Field2",
        "schema": "string"
      }
    ]
  }
}
```

Клиент устройства должен отправить полезные данные JSON, как в следующем примере, как сообщаемое свойство в двойникае устройства:

```json
{
  "ObjectProperty": {
    "Field1": 37,
    "Field2": "A string value"
  }
}
```

В следующем фрагменте кода из модели устройства показано определение `vector` типа свойства:

```json
{
  "@type": "Property",
  "displayName": {
    "en": "VectorProperty"
  },
  "name": "VectorProperty",
  "schema": "vector",
  "writable": false
}
```

Клиент устройства должен отправить полезные данные JSON, как в следующем примере, как сообщаемое свойство в двойникае устройства:

```json
{
  "VectorProperty": {
    "x": 74.72395045538597,
    "y": 74.72395045538597,
    "z": 74.72395045538597
  }
}
```

### <a name="writeable-property-types"></a>Типы свойств с возможностью записи

В этом разделе приводятся примеры записываемых типов свойств, получаемых устройством от IoT Central приложения.

IoT Central ждет от устройства отклика на записываемые обновления свойств. Ответное сообщение должно включать `ac` поля и `av` . Поле `ad` является необязательным. Примеры см. в следующих фрагментах кода.

`ac` числовое поле, использующее значения из следующей таблицы:

| Значение | Метка | Описание |
| ----- | ----- | ----------- |
| `'ac': 200` | Завершено | Операция изменения свойства успешно завершена. |
| `'ac': 202`  ни `'ac': 201` | Ожидает | Операция изменения свойства находится в состоянии ожидания или выполняется |
| `'ac': 4xx` | Ошибка | Запрошенное изменение свойства недопустимо или имело ошибку |
| `'ac': 5xx` | Ошибка | Произошла непредвиденная ошибка устройства при обработке запрошенного изменения. |

`av` номер версии, отправляемый на устройство.

`ad` — Это описание строки параметра.

В следующем фрагменте кода из модели устройства показано определение типа записываемого `string` Свойства:

```json
{
  "@type": "Property",
  "displayName": {
    "en": "StringPropertyWritable"
  },
  "name": "StringPropertyWritable",
  "writable": true,
  "schema": "string"
}
```

Устройство получает следующие полезные данные из IoT Central:

```json
{  
  "StringPropertyWritable": "A string from IoT Central", "$version": 7
}
```

Устройство должно отправить следующие полезные данные JSON в IoT Central после обработки обновления. Это сообщение включает номер версии исходного обновления, полученного от IoT Central. Когда IoT Central получает это сообщение, оно помечает свойство как **синхронизированное** в пользовательском интерфейсе:

```json
{
  "StringPropertyWritable": {
    "value": "A string from IoT Central",
    "ac": 200,
    "ad": "completed",
    "av": 7
  }
}
```

В следующем фрагменте кода из модели устройства показано определение типа записываемого `Enum` Свойства:

```json
{
  "@type": "Property",
  "displayName": {
    "en": "EnumPropertyWritable"
  },
  "name": "EnumPropertyWritable",
  "writable": true,
  "schema": {
    "@type": "Enum",
    "displayName": {
      "en": "Enum"
    },
    "valueSchema": "integer",
    "enumValues": [
      {
        "displayName": {
          "en": "Item1"
        },
        "enumValue": 0,
        "name": "Item1"
      },
      {
        "displayName": {
          "en": "Item2"
        },
        "enumValue": 1,
        "name": "Item2"
      },
      {
        "displayName": {
          "en": "Item3"
        },
        "enumValue": 2,
        "name": "Item3"
      }
    ]
  }
}
```

Устройство получает следующие полезные данные из IoT Central:

```json
{  
  "EnumPropertyWritable":  1 , "$version": 10
}
```

Устройство должно отправить следующие полезные данные JSON в IoT Central после обработки обновления. Это сообщение включает номер версии исходного обновления, полученного от IoT Central. Когда IoT Central получает это сообщение, оно помечает свойство как **синхронизированное** в пользовательском интерфейсе:

```json
{
  "EnumPropertyWritable": {
    "value": 1,
    "ac": 200,
    "ad": "completed",
    "av": 10
  }
}
```

## <a name="commands"></a>Команды

В следующем фрагменте кода из модели устройства показано определение команды, которая не имеет параметров и не ждет, что устройство возвратит какое-либо значение:

```json
{
  "@type": "Command",
  "displayName": {
    "en": "CommandBasic"
  },
  "name": "CommandBasic"
}
```

Устройство получает пустые полезные данные в запросе и должно возвращать пустую полезную нагрузку в ответе с `200` кодом HTTP-ответа, чтобы указать на успешное выполнение.

В следующем фрагменте кода из модели устройства показано определение команды с целочисленным параметром и ожидание возврата устройством целочисленного значения:

```json
{
  "@type": "Command",
  "request": {
    "@type": "CommandPayload",
    "displayName": {
      "en": "RequestParam"
    },
    "name": "RequestParam",
    "schema": "integer"
  },
  "response": {
    "@type": "CommandPayload",
    "displayName": {
      "en": "ResponseParam"
    },
    "name": "ResponseParam",
    "schema": "integer"
  },
  "displayName": {
    "en": "CommandSimple"
  },
  "name": "CommandSimple"
}
```

Устройство получает целочисленное значение в качестве полезных данных запроса. Устройство должно вернуть целочисленное значение в качестве полезных данных ответа с `200` кодом HTTP-ответа, чтобы указать на успешное выполнение.

В следующем фрагменте кода из модели устройства показано определение команды, имеющей параметр объекта и ожидающее, что устройство возвращает объект. В этом примере оба объекта имеют целочисленные и строковые поля:

```json
{
  "@type": "Command",
  "request": {
    "@type": "CommandPayload",
    "displayName": {
      "en": "RequestParam"
    },
    "name": "RequestParam",
    "schema": {
      "@type": "Object",
      "displayName": {
        "en": "Object"
      },
      "fields": [
        {
          "displayName": {
            "en": "Field1"
          },
          "name": "Field1",
          "schema": "integer"
        },
        {
          "displayName": {
            "en": "Field2"
          },
          "name": "Field2",
          "schema": "string"
        }
      ]
    }
  },
  "response": {
    "@type": "CommandPayload",
    "displayName": {
      "en": "ResponseParam"
    },
    "name": "ResponseParam",
    "schema": {
      "@type": "Object",
      "displayName": {
        "en": "Object"
      },
      "fields": [
        {
          "displayName": {
            "en": "Field1"
          },
          "name": "Field1",
          "schema": "integer"
        },
        {
          "displayName": {
            "en": "Field2"
          },
          "name": "Field2",
          "schema": "string"
        }
      ]
    }
  },
  "displayName": {
    "en": "CommandComplex"
  },
  "name": "CommandComplex"
}
```

В следующем фрагменте кода показаны примеры полезных данных запроса, отправленные на устройство:

```json
{ "Field1": 56, "Field2": "A string value" }
```

В следующем фрагменте кода показаны примеры полезных данных ответа, отправленных с устройства. Используйте `200` код ответа HTTP, чтобы указать на успешное выполнение:

```json
{ "Field1": 87, "Field2": "Another string value" }
```

### <a name="long-running-commands"></a>Длительно выполняемые команды

В следующем фрагменте кода из модели устройства показано определение команды. Команда имеет целочисленный параметр и ждет, пока устройство возвратит целочисленное значение:

```json
{
  "@type": "Command",
  "request": {
    "@type": "CommandPayload",
    "displayName": {
      "en": "RequestParam"
    },
    "name": "RequestParam",
    "schema": "integer"
  },
  "response": {
    "@type": "CommandPayload",
    "displayName": {
      "en": "ResponseParam"
    },
    "name": "ResponseParam",
    "schema": "integer"
  },
  "displayName": {
    "en": "LongRunningCommandSimple"
  },
  "name": "LongRunningCommandSimple"
}
```

Устройство получает целочисленное значение в качестве полезных данных запроса. Если устройству требуется время для обработки этой команды, она должна вернуть пустые полезные данные ответа с `202` кодом HTTP-ответа, чтобы указать, что устройство приняло запрос на обработку.

Когда устройство закончит обработку запроса, оно должно отправить свойство в IoT Central, как показано в следующем примере. Имя свойства должно совпадать с именем команды:

```json
{
  "LongRunningCommandSimple": {
    "value": 87
  }
}
```

### <a name="offline-commands"></a>Автономные команды

В веб-ИНТЕРФЕЙСе IoT Central можно выбрать параметр **очередь при автономной работе** для команды. Автономные команды — это односторонние уведомления для устройства из вашего решения, которые доставляются сразу же после подключения устройства. Автономные команды могут иметь параметры запроса, но не возвращать ответ.

Если в шаблоне устройства была экспортирована модель или интерфейс, то **очередь** не включается. При просмотре экспортированной модели или интерфейса JSON нельзя определить, что команда является автономной.

Автономные команды используют сообщения, отправляемые из [облака центра Интернета вещей на устройство](../../iot-hub/iot-hub-devguide-messages-c2d.md) , для отправки команды и полезных данных на устройство.

В следующем фрагменте кода из модели устройства показано определение команды. Команда имеет параметр объекта с полем DateTime и перечислением:

```json
{
  "@type": "Command",
  "displayName": {
    "en": "Generate Diagnostics"
  },
  "name": "GenerateDiagnostics",
  "request": {
    "@type": "CommandPayload",
    "displayName": {
      "en": "Payload"
    },
    "name": "Payload",
    "schema": {
      "@type": "Object",
      "displayName": {
        "en": "Object"
      },
      "fields": [
        {
          "displayName": {
            "en": "StartTime"
          },
          "name": "StartTime",
          "schema": "dateTime"
        },
        {
          "displayName": {
            "en": "Bank"
          },
          "name": "Bank",
          "schema": {
            "@type": "Enum",
            "displayName": {
              "en": "Enum"
            },
            "enumValues": [
              {
                "displayName": {
                  "en": "Bank 1"
                },
                "enumValue": 1,
                "name": "Bank1"
              },
              {
                "displayName": {
                  "en": "Bank2"
                },
                "enumValue": 2,
                "name": "Bank2"
              },
              {
                "displayName": {
                  "en": "Bank3"
                },
                "enumValue": 3,
                "name": "Bank3"
              }
            ],
            "valueSchema": "integer"
          }
        }
      ]
    }
  }
}
```

Если включить параметр **Queue, если** в пользовательском интерфейсе шаблона устройства для команды в предыдущем фрагменте, то сообщение, которое получит устройство, будет содержать следующие свойства:

| Имя свойства | Пример значения |
| ---------- | ----- |
| `custom_properties` | `{'method-name': 'GenerateDiagnostics'}` |
| `data` | `{"StartTime":"2021-01-05T08:00:00.000Z","Bank":2}` |

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда разработчик устройства изучили шаблоны устройств, мы рекомендуем ознакомиться с разработкой " [Подключение к Azure IOT Central](./concepts-get-connected.md) ", чтобы узнать больше о том, как регистрировать устройства в IOT Central и как IOT Central защищает подключения устройств.
