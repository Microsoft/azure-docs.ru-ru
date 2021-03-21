---
title: Сведения о цифровых двойниках IoT Plug and Play
description: Сведения о том, как IoT самонастраивающийся использует цифровые двойников
author: prashmo
ms.author: prashmo
ms.date: 12/14/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 99c957e5bf6ffe69c94e109796590f5ab975c3cf
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97656892"
---
# <a name="understand-iot-plug-and-play-digital-twins"></a>Сведения о цифровых двойниках IoT Plug and Play

Устройство IoT самонастраивающийся реализует модель, описываемую схемой [Digital двойников Definition Language (дтдл)](https://github.com/Azure/opendigitaltwins-dtdl) . Модель описывает набор компонентов, свойств, команд и сообщений телеметрии, которые может иметь определенное устройство.

В самонастраивающийся IoT используется версия 2 ДТДЛ. Дополнительные сведения об этой версии см. в спецификации [Digital двойников Definition Language (дтдл) — версия 2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) на сайте GitHub.

> [!NOTE]
> ДТДЛ не является эксклюзивным самонастраивающийся Интернета вещей. Другие службы IoT, такие как [Azure Digital двойников](../digital-twins/overview.md), используют ее для представления целых сред, таких как здания и сети Energy.

Пакеты SDK для службы Azure IoT включают интерфейсы API, которые позволяют службе взаимодействовать с цифровым двойникаом устройства. Например, служба может считывать свойства устройства из цифрового двойника или использовать цифровое двойника для вызова команды на устройстве. Дополнительные сведения см. в статье [примеры цифровых двойника для центра Интернета вещей](concepts-developer-guide-service.md#iot-hub-digital-twin-examples).

В примере устройства IoT самонастраивающийся в этой статье реализована [модель контроллера температуры](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) с компонентами [термостата](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json) .

## <a name="device-twins-and-digital-twins"></a>Двойникови устройств и Цифровые двойников

Кроме цифрового двойника, центр Интернета вещей Azure также поддерживает *двойника устройств* для каждого подключенного устройства. Двойника устройства похоже на цифровое двойника в том, что это представление свойств устройства. Пакеты SDK для службы Azure IoT включают API для взаимодействия с двойниковами устройств.

Центр Интернета вещей инициализирует цифровое двойника и устройство двойника при первом подключении устройства IoT самонастраивающийся.

Двойникови устройств — это документы JSON, в которых хранятся сведения о состоянии устройства, включая метаданные, конфигурации и условия. Дополнительные сведения см. в статье [примеры клиентов службы центра Интернета вещей](concepts-developer-guide-service.md#iot-hub-service-client-examples). Построители устройств и решений могут по-прежнему использовать один и тот же набор API-интерфейсов Двойникаа устройств и пакетов SDK для реализации устройств и решений с помощью соглашений самонастраивающийся IoT.

API-интерфейсы Digital двойника работают с ДТДЛми конструкциями высокого уровня, такими как компоненты, свойства и команды. Интерфейсы API Digital двойника упрощают создание решений IoT самонастраивающийся решениями для сборщиков решений.

В двойникае устройства состояние доступного для записи свойства разбивается по *нужным свойствам* и разделам *сообщаемых свойств* . Все свойства, доступные только для чтения, доступны в разделе сообщаемые свойства.

В цифровом двойника есть унифицированное представление текущего и требуемого состояния свойства. Состояние синхронизации для данного свойства хранится в соответствующем разделе компонента по умолчанию `$metadata` .

### <a name="device-twin-json-example"></a>Пример JSON двойникаа устройства

В следующем фрагменте кода показано, как двойника устройство IoT самонастраивающийся отформатировано как объект JSON:

```json
{
  "deviceId": "sample-device",
  "modelId": "dtmi:com:example:TemperatureController;1",
  "version": 15,
  "properties": {
    "desired": {
      "thermostat1": {
        "__t": "c",
        "targetTemperature": 21.8
      },
      "$metadata": {...},
      "$version": 4
    },
    "reported": {
      "serialNumber": "alwinexlepaho8329",
      "thermostat1": {
        "maxTempSinceLastReboot": 25.3,
        "__t": "c",
        "targetTemperature": {
          "value": 21.8,
          "ac": 200,
          "ad": "Successfully executed patch",
        }
      },
      "$metadata": {...},
      "$version": 11
    }
  }
}
```

### <a name="digital-twin-example"></a>Пример цифрового двойника

В следующем фрагменте кода показан цифровой двойника, отформатированный как объект JSON:

```json
{
  "$dtId": "sample-device",
  "serialNumber": "alwinexlepaho8329",
  "thermostat1": {
    "maxTempSinceLastReboot": 25.3,
    "targetTemperature": 21.8,
    "$metadata": {
      "targetTemperature": {
        "desiredValue": 21.8,
        "desiredVersion": 4,
        "ackVersion": 4,
        "ackCode": 200,
        "ackDescription": "Successfully executed patch",
        "lastUpdateTime": "2020-07-17T06:11:04.9309159Z"
      },
      "maxTempSinceLastReboot": {
         "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
      }
    }
  },
  "$metadata": {
    "$model": "dtmi:com:example:TemperatureController;1",
    "serialNumber": {
      "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
    }
  }
}
```

В следующей таблице описаны поля в объекте Digital двойника JSON:

| Имя поля | Описание |
| --- | --- |
| `$dtId` | Предоставляемая пользователем строка, представляющая идентификатор цифрового двойникаа устройства |
| `{propertyName}` | Значение свойства в JSON |
| `$metadata.$model` | Используемых Идентификатор интерфейса модели, который характеризует этот цифровой двойника |
| `$metadata.{propertyName}.desiredValue` | [Только для свойств, доступных для записи] Требуемое значение указанного свойства |
| `$metadata.{propertyName}.desiredVersion` | [Только для свойств, доступных для записи] Версия требуемого значения, поддерживаемого центром Интернета вещей|
| `$metadata.{propertyName}.ackVersion` | [Обязательно, только для свойств, доступных для записи] Версия, подтвержденная устройством, реализующим цифровой двойника, должна быть больше или равна желаемой версии |
| `$metadata.{propertyName}.ackCode` | [Обязательно, только для свойств, доступных для записи] `ack` Код, возвращенный приложением устройства, реализующим цифровой двойника |
| `$metadata.{propertyName}.ackDescription` | [Необязательно, только для свойств, доступных для записи] `ack` Описание, возвращаемое приложением устройства, реализующим цифровой двойника |
| `$metadata.{propertyName}.lastUpdateTime` | Центр Интернета вещей сохраняет метку времени последнего обновления свойства устройством. Метки времени находятся в формате UTC и кодируются в формате ISO8601 гггг-мм-DDTHH: MM: SS. Мммп |
| `{componentName}` | Объект JSON, содержащий значения и метаданные свойств компонента. |
| `{componentName}.{propertyName}` | Значение свойства компонента в JSON |
| `{componentName}.$metadata` | Сведения о метаданных для компонента. |

### <a name="properties"></a>Свойства

Свойства — это поля данных, представляющие состояние сущности (например, свойства во многих языках объектно-ориентированного программирования).

#### <a name="read-only-property"></a>Свойство только для чтения

Схема ДТДЛ:

```json
{
    "@type": "Property",
    "name": "serialNumber",
    "displayName": "Serial Number",
    "description": "Serial number of the device.",
    "schema": "string"
}
```

В этом примере `alwinexlepaho8329` — Текущее значение свойства, доступного `serialNumber` только для чтения, сообщаемое устройством.
В следующих фрагментах кода показано параллельное представление JSON `serialNumber` Свойства:

:::row:::
   :::column span="":::
      **"Двойник" устройства**

```json
"properties": {
  "reported": {
    "serialNumber": "alwinexlepaho8329"
  }
}
```

   :::column-end:::
   :::column span="":::
      **Цифровой двойник**

```json
"serialNumber": "alwinexlepaho8329"
```

   :::column-end:::
:::row-end:::

#### <a name="writable-property"></a>Доступное для записи свойство

В следующих примерах показано свойство, доступное для записи, в компоненте по умолчанию.

ДТДЛ:

```json
{
  "@type": "Property",
  "name": "fanSpeed",
  "displayName": "Fan Speed",
  "writable": true,
  "schema": "double"
}
```

:::row:::
   :::column span="":::
      **"Двойник" устройства**

```json
{
  "properties": {
    "desired": {
      "fanSpeed": 2.0,
    },
    "reported": {
      "fanSpeed": {
        "value": 3.0,
        "ac": 200,
        "av": 1,
        "ad": "Successfully executed patch version 1"
      }
    }
  },
}
```

   :::column-end:::
   :::column span="":::
      **Цифровой двойник**

```json
{
  "fanSpeed": 3.0,
  "$metadata": {
    "fanSpeed": {
      "desiredValue": 2.0,
      "desiredVersion": 2,
      "ackVersion": 1,
      "ackCode": 200,
      "ackDescription": "Successfully executed patch version 1",
      "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
    }
  }
}
```

   :::column-end:::
:::row-end:::

В этом примере `3.0` — это текущее значение `fanSpeed` свойства, сообщаемое устройством. `2.0` требуемое значение задается решением. Требуемое значение и состояние синхронизации свойства корневого уровня устанавливаются в корневом уровне `$metadata` для цифрового двойника. Когда устройство переходит в режим «в сети», оно может применить это обновление и сообщить о обновленном значении.

### <a name="components"></a>Компоненты

Компоненты позволяют строить интерфейс модели в виде сборки других интерфейсов.
Например, интерфейс [термостата](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json) может быть включен как компоненты `thermostat1` и  `thermostat2` в модель [модели контроллера температуры](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) .

В двойникае устройства компонент идентифицируется `{ "__t": "c"}` маркером. В цифровом двойника присутствие `$metadata` помечает компонент.

В этом примере `thermostat1` является компонентом с двумя свойствами:

- `maxTempSinceLastReboot` свойство доступно только для чтения.
- `targetTemperature` — Это свойство, доступное для записи, которое было успешно синхронизировано устройством. Требуемое значение и состояние синхронизации этих свойств находятся в компоненте `$metadata` .

В следующих фрагментах кода показано параллельное представление JSON `thermostat1` компонента:

:::row:::
   :::column span="":::
      **"Двойник" устройства**

```json
"properties": {
  "desired": {
    "thermostat1": {
      "__t": "c",
      "targetTemperature": 21.8
    },
    "$metadata": {
    },
    "$version": 4
  },
  "reported": {
    "thermostat1": {
      "maxTempSinceLastReboot": 25.3,
      "__t": "c",
      "targetTemperature": {
        "value": 21.8,
        "ac": 200,
        "ad": "Successfully executed patch",
        "av": 4
      }
    },
    "$metadata": {
    },
    "$version": 11
  }
}
```

   :::column-end:::
   :::column span="":::
      **Цифровой двойник**

```json
"thermostat1": {
  "maxTempSinceLastReboot": 25.3,
  "targetTemperature": 21.8,
  "$metadata": {
    "targetTemperature": {
      "desiredValue": 21.8,
      "desiredVersion": 4,
      "ackVersion": 4,
      "ackCode": 200,
      "ackDescription": "Successfully executed patch",
      "lastUpdateTime": "2020-07-17T06:11:04.9309159Z"
    },
    "maxTempSinceLastReboot": {
       "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
    }
  }
}
```

   :::column-end:::
:::row-end:::

## <a name="digital-twin-apis"></a>Интерфейсы API цифровых двойника

В число API-интерфейсов Digital двойника входит **получение цифровых двойника**, **обновление цифровых двойника**, **вызов компонента Component** и **вызов** командных операций для управления цифровыми двойника. Вы можете использовать API- [интерфейсы RESTful](/rest/api/iothub/service/digitaltwin) напрямую или через [пакет SDK службы](../iot-pnp/libraries-sdks.md).

## <a name="digital-twin-change-events"></a>События изменения цифровых двойников

Если включить события изменения цифрового двойника, событие будет активироваться при каждом изменении текущего или требуемого значения компонента или свойства. События изменения цифрового двойника создаются в формате [обновления JSON](http://jsonpatch.com/) . Соответствующие события создаются в формате двойникаа устройства, если включены события изменения двойника.

Сведения о том, как включить маршрутизацию для событий устройств и цифровых двойника, см. в статье [Использование маршрутизации сообщений центра Интернета вещей для отправки сообщений с устройства в облако в разные конечные точки](../iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events). Сведения о формате сообщений см. в [статье Создание и чтение сообщений центра Интернета вещей](../iot-hub/iot-hub-devguide-messages-construct.md).

Например, следующее событие двойника Changed (Цифровое изменение) активируется, когда `targetTemperature` задается решением:

```json
iothub-connection-device-id:sample-device
iothub-enqueuedtime:7/17/2020 6:11:04 AM
iothub-message-source:digitalTwinChangeEvents
correlation-id:275d463fa034
content-type:application/json-patch+json
content-encoding:utf-8
[
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature",
    "value": {
      "desiredValue": 21.8,
      "desiredVersion": 4
    }
  }
]
```

Следующее событие двойника Changed срабатывает, когда устройство сообщает о том, что было применено указанное выше изменение:

```json
iothub-connection-device-id:sample-device
iothub-enqueuedtime:7/17/2020 6:11:05 AM
iothub-message-source:digitalTwinChangeEvents
correlation-id:275d464a2c80
content-type:application/json-patch+json
content-encoding:utf-8
[
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/ackCode",
    "value": 200
  },
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/ackDescription",
    "value": "Successfully executed patch"
  },
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/ackVersion",
    "value": 4
  },
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/lastUpdateTime",
    "value": "2020-07-17T06:11:04.9309159Z"
  },
  {
    "op": "add",
    "path": "/thermostat1/targetTemperature",
    "value": 21.8
  }
]
```

> [!NOTE]
> Двойника сообщения об изменении уведомлений при включении в уведомлениях об изменениях устройства и цифрового двойника удваивается.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали о цифровом двойников, вот несколько дополнительных ресурсов:

- [Как использовать центры Интернета вещей самонастраивающийся Digital двойника API](howto-manage-digital-twin.md)
- [Взаимодействие с устройством из вашего решения](quickstart-service.md)
- [REST API IoT Digital двойника](/rest/api/iothub/service/digitaltwin)
- [Обозреватель Интернета вещей Azure](howto-use-iot-explorer.md)