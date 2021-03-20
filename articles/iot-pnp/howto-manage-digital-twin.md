---
title: Управление самонастраивающийся Интернета вещей Digital двойников
description: Управление устройством самонастраивающийся IoT с помощью цифровых API-интерфейсов двойника
author: prashmo
ms.author: prashmo
ms.date: 07/20/2020
ms.topic: how-to
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: ddb8027c145f6a38bfcd953be66dae2943a20c3a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97654614"
---
# <a name="manage-iot-plug-and-play-digital-twins"></a>Управление самонастраивающийся Интернета вещей Digital двойников

IoT самонастраивающийся поддерживает **получение цифровых двойника** и **обновление цифровых двойниканых** операций для управления цифровыми двойников. Можно использовать [API-интерфейсы](/rest/api/iothub/service/digitaltwin) или один из [пакетов SDK для служб](libraries-sdks.md).

На момент написания статьи версия API Digital двойника — `2020-09-30` .

## <a name="update-a-digital-twin"></a>Обновление цифрового двойника

Устройство IoT самонастраивающийся реализует модель, описанную в статье [Digital двойников Definition Language v2 (дтдл)](https://github.com/Azure/opendigitaltwins-dtdl). Разработчики решений могут использовать **API обновления Digital двойника** для обновления состояния компонента и свойств цифрового двойника.

Устройство IoT самонастраивающийся, используемое в качестве примера в этой статье, реализует [модель контроллера температуры](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) с компонентами [термостата](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json) .

В следующем фрагменте кода показан ответ на запрос **Get Digital двойника** , отформатированный как объект JSON. Дополнительные сведения о формате Digital двойника см. в разделе [Общие сведения об IoT Самонастраивающийся Digital двойников](./concepts-digital-twin.md#digital-twin-example).

```json
{
    "$dtId": "sample-device",
    "serialNumber": "alwinexlepaho8329",
    "thermostat1": {
        "maxTempSinceLastReboot": 25.3,
        "targetTemperature": 20.4,
        "$metadata": {
            "targetTemperature": {
                "desiredValue": 20.4,
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

Digital двойников позволяет обновить весь компонент или свойство с помощью [исправления JSON](http://jsonpatch.com/).

Например, можно обновить `targetTemperature` свойство следующим образом:

```json
[
    {
        "op": "add",
        "path": "/thermostat1/targetTemperature",
        "value": 21.4
    }
]
```

Предыдущее обновление задает нужное значение свойства в соответствующем уровне компонента `$metadata` , как показано в следующем фрагменте кода. Центр Интернета вещей обновляет требуемую версию свойства:

```json
"thermostat1": {
    "targetTemperature": 20.4,
    "$metadata": {
        "targetTemperature": {
            "desiredValue": 21.4,
            "desiredVersion": 5,
            "ackVersion": 4,
            "ackCode": 200,
            "ackDescription": "Successfully executed patch",
            "lastUpdateTime": "2020-07-17T06:11:04.9309159Z"
        }
    }
}
```

### <a name="add-replace-or-remove-a-component"></a>Добавление, замена или удаление компонента

Для операций уровня компонента требуется пустой `$metadata` маркер объекта в пределах значения.

Операция добавления или замены компонента задает нужные значения всех предоставленных свойств. Он также очищает нужные значения для всех записываемых свойств, не предоставляемых вместе с обновлением.

При удалении компонента очищаются нужные значения всех имеющихся свойств, доступных для записи. Устройство в конечном итоге синхронизирует это удаление и прекращает отправку отчетов по отдельным свойствам. Затем компонент удаляется из цифрового двойника.

В следующем примере исправления JSON показано, как добавить, заменить или удалить компонент:

```json
[
    {
        "op": "add",
        "path": "/thermostat1",
        "value": {
            "targetTemperature": 21.4,
            "anotherWritableProperty": 42,
            "$metadata": {}
        }
    },
    {
        "op": "replace",
        "path": "/thermostat1",
        "value": {
            "targetTemperature": 21.4,
            "$metadata": {}
        }
    },
    {
        "op": "remove",
        "path": "/thermostat2"
    }
]
```

### <a name="add-replace-or-remove-a-property"></a>Добавление, замена или удаление свойства

Операция добавления или замены задает нужное значение свойства. Устройство может синхронизировать состояние и сообщить об обновлении значения вместе с `ack` кодом, версией и описанием.

Удаление свойства очищает нужное значение свойства, если оно задано. После этого устройство может перестанут сообщить об этом свойстве, и оно будет удалено из компонента. Если это свойство является последним в компоненте, компонент также удаляется.

В следующем примере исправления JSON показано, как добавить, заменить или удалить свойство в компоненте.

```json
[
    {
        "op": "add",
        "path": "/thermostat1/targetTemperature",
        "value": 21.4
    },
    {
        "op": "replace",
        "path": "/thermostat1/anotherWritableProperty",
        "value": 42
    },
    {
        "op": "remove",
        "path": "/thermostat2/targetTemperature",
    }
]
```

### <a name="rules-for-setting-the-desired-value-of-a-digital-twin-property"></a>Правила настройки требуемого значения свойства цифрового двойника

**имя**;

Имя компонента или свойства должно быть допустимым именем ДТДЛ v2.

Допустимые символы: a – z, A – Z, 0-9 (не в качестве первого символа) и символ подчеркивания (не как первый или последний символ).

Длина имени может составлять 1-64 символов.

**Значение свойства**

Значение должно быть допустимым [свойством дтдл v2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#property).

Поддерживаются все типы-примитивы. В сложных типах поддерживаются перечисления, сопоставления и объекты. Дополнительные сведения см. в разделе [схемы дтдл v2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#schemas).

Свойства не поддерживают массив или сложную схему с массивом.

Для сложного объекта поддерживается максимальная глубина пяти уровней.

Все имена полей в сложном объекте должны быть допустимыми именами ДТДЛ v2.

Все ключи карт должны быть допустимыми именами ДТДЛ v2.

## <a name="troubleshoot-update-digital-twin-api-errors"></a>Устранение ошибок при обновлении цифровых двойника API

API Digital двойника создает следующее общее сообщение об ошибке:

`ErrorCode:ArgumentInvalid;'{propertyName}' exists within the device twin and is not digital twin conformant property. Please refer to aka.ms/dtpatch to update this to be conformant.`

Если вы видите эту ошибку, убедитесь, что обновление обновления соответствует [правилам установки требуемого значения свойства Digital двойника](#rules-for-setting-the-desired-value-of-a-digital-twin-property) .

При обновлении компонента убедитесь, что установлен [маркер пустой объект $Metadata](#add-replace-or-remove-a-component) .

Обновление может завершиться неудачей, если сообщаемые значения устройства не соответствуют [соглашениям об отсоединениях и игре IOT](./concepts-convention.md#writable-properties).

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали о цифровом двойников, вот несколько дополнительных ресурсов:

- [Взаимодействие с устройством из вашего решения](quickstart-service.md)
- [REST API IoT Digital двойника](/rest/api/iothub/service/digitaltwin)
- [Обозреватель Интернета вещей Azure](howto-use-iot-explorer.md)
