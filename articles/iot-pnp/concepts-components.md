---
title: Общие сведения о компонентах в самонастраивающийсяных моделях Интернета вещей | Документация Майкрософт
description: Разница между моделями ДТДЛ Интернета вещей самонастраивающийся, которые используют компоненты и модели, которые не используют компоненты.
author: ericmitt
ms.author: ericmitt
ms.date: 07/07/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: eef8179567d83e3727c3ab949eef2706ce2a9b16
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102175810"
---
# <a name="iot-plug-and-play-components-in-models"></a>Компоненты IoT Plug and Play в моделях

В соглашениях IoT Plug and Play устройство является устройством IoT Plug and Play, если при подключении к центру Интернета вещей оно представляет идентификатор модели языка определения цифровых двойников.

В следующем фрагменте показан пример ИД модели:

```json
 "@id": "dtmi:com:example:TemperatureController;1"
 "@id": "dtmi:com:example:Thermostat;1",
```

## <a name="no-components"></a>Нет компонентов

В простой модели не используются внедренные или каскадные компоненты. В нем содержатся сведения о заголовке и разделе "содержимое" для определения телеметрии, свойств и команд.

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

Хотя модель явно не определяет компонент, она ведет себя так, как если бы он был единственным _компонентом по умолчанию_ со всеми определениями телеметрии, свойств и команд.

На следующем снимке экрана показано, как модель отображается в средстве Azure IoT Explorer.

:::image type="content" source="media/concepts-components/default-component.png" alt-text="Компонент по умолчанию в Azure IoT Explorer":::

Идентификатор модели хранится в свойстве двойникаа устройства, как показано на следующем снимке экрана:

:::image type="content" source="media/concepts-components/twin-model-id.png" alt-text="Идентификатор модели в свойстве Digital двойника":::

Модель ДТДЛ без компонентов является удобным упрощением для устройства или IoT Edge модуля с единым набором телеметрии, свойств и команд. Модель, которая не использует компоненты, упрощает перенос существующего устройства или модуля в центр Интернета вещей самонастраивающийся устройство или модуль. вы создаете модель ДТДЛ, которая описывает реальное устройство или модуль без необходимости определять какие-либо компоненты.

> [!TIP]
> Модуль может быть [модулем](../iot-hub/iot-hub-devguide-module-twins.md) устройства или [модулем IOT Edge](../iot-edge/about-iot-edge.md).

## <a name="multiple-components"></a>Несколько компонентов

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
...
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
...
```

Эта модель содержит три компонента, определенные в разделе содержимого — два `Thermostat` `DeviceInformation` компонента и компонент. Существует также компонент по умолчанию.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали о компонентах модели, вот некоторые дополнительные ресурсы:

- [Установка и использование средств разработки ДТДЛ](howto-use-dtdl-authoring-tools.md)
- [Digital двойников Definition Language v2 (ДТДЛ)](https://github.com/Azure/opendigitaltwins-dtdl)
- [Репозитории модели](./concepts-model-repository.md)
