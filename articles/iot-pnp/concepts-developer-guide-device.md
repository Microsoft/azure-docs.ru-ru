---
title: Руководством для разработчиков устройств (C) — самонастраивающийся IoT | Документация Майкрософт
description: Описание самонастраивающийся IoT для разработчиков устройств C
author: rido-min
ms.author: rmpablos
ms.date: 11/19/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
zone_pivot_groups: programming-languages-set-twenty-six
ms.openlocfilehash: 0cca47269e632e1fcba1f8f9eb1c835f27e63059
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104582839"
---
# <a name="iot-plug-and-play-device-developer-guide"></a>Справочное руководством для разработчиков IoT самонастраивающийся

Самонастраивающийся IoT позволяет создавать интеллектуальные устройства, которые объявляют свои возможности для приложений Интернета вещей Azure. Устройства IoT самонастраивающийся не нуждаются в ручной настройке, когда клиент подключает их к приложениям с поддержкой Интернета вещей самонастраивающийся.

Смарт-устройство может быть реализовано напрямую, использовать [модули](../iot-hub/iot-hub-devguide-module-twins.md)или использовать [модули IOT Edge](../iot-edge/about-iot-edge.md).

В этом руководстве описываются основные шаги, необходимые для создания устройства, модуля или IoT Edge модуля, который соответствует [соглашениям Самонастраивающийся IOT](../iot-pnp/concepts-convention.md).

Чтобы создать модуль самонастраивающийся для устройства, модуля или IoT Edge IoT, выполните следующие действия.

1. Убедитесь, что устройство использует протокол MQTT или MQTT через WebSocket для подключения к центру Интернета вещей Azure.
1. Создайте модель [дтдл (Digital двойников Definition Language)](https://github.com/Azure/opendigitaltwins-dtdl) для описания устройства. Дополнительные сведения см. в разделе [Знакомство с компонентами в самонастраивающийсяных моделях Интернета вещей](concepts-modeling-guide.md).
1. Обновите устройство или модуль, чтобы объявить `model-id` об этом как часть подключения устройства.
1. Реализация телеметрии, свойств и команд с помощью [соглашений Самонастраивающийся IOT](concepts-convention.md)

Когда ваша реализация устройства или модуля будет готова, воспользуйтесь [обозревателем Azure IOT](howto-use-iot-explorer.md) , чтобы проверить, соответствует ли устройство правилам Самонастраивающийся IOT.

:::zone pivot="programming-language-ansi-c"

[!INCLUDE [iot-pnp-device-devguide-c](../../includes/iot-pnp-device-devguide-c.md)]

:::zone-end

:::zone pivot="programming-language-csharp"

[!INCLUDE [iot-pnp-device-devguide-csharp](../../includes/iot-pnp-device-devguide-csharp.md)]

:::zone-end

:::zone pivot="programming-language-java"

[!INCLUDE [iot-pnp-device-devguide-java](../../includes/iot-pnp-device-devguide-java.md)]

:::zone-end

:::zone pivot="programming-language-javascript"

[!INCLUDE [iot-pnp-device-devguide-node](../../includes/iot-pnp-device-devguide-node.md)]

:::zone-end

:::zone pivot="programming-language-python"

[!INCLUDE [iot-pnp-device-devguide-python](../../includes/iot-pnp-device-devguide-python.md)]

:::zone-end

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали о разработке самонастраивающийся для устройств IoT, ниже приведены некоторые дополнительные ресурсы.

- [Язык определения цифровых двойников (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl)
- [Пакет SDK для устройств C](/azure/iot-hub/iot-c-sdk-ref/)
- [REST API IoT](/rest/api/iothub/device)
- [Общие сведения о компонентах в моделях самонастраивающийся IoT](concepts-modeling-guide.md)
- [Установка и использование средств разработки ДТДЛ](howto-use-dtdl-authoring-tools.md)
- [Руководством для разработчиков служб IoT самонастраивающийся](concepts-developer-guide-service.md)