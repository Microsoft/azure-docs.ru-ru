---
title: Руководством для разработчиков служб — самонастраивающийся IoT | Документация Майкрософт
description: Описание самонастраивающийся IoT для разработчиков служб
author: dominicbetts
ms.author: dobett
ms.date: 10/01/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
zone_pivot_groups: programming-languages-set-ten
ms.openlocfilehash: a5889be88dfd0870a2eee868c97787ff354cff68
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104582737"
---
# <a name="iot-plug-and-play-service-developer-guide"></a>Руководством для разработчиков служб IoT самонастраивающийся

Самонастраивающийся IoT позволяет создавать интеллектуальные устройства, которые объявляют свои возможности для приложений Интернета вещей Azure. Устройства IoT самонастраивающийся не нуждаются в ручной настройке, когда клиент подключает их к приложениям с поддержкой Интернета вещей самонастраивающийся.

IoT самонастраивающийся позволяет использовать устройства, которые объявили идентификатор модели с помощью центра Интернета вещей. Например, можно напрямую обращаться к свойствам и командам устройства.

Чтобы использовать устройство IoT самонастраивающийся, подключенное к центру Интернета вещей, воспользуйтесь одним из пакетов SDK для службы IoT:

## <a name="service-sdks"></a>Пакеты SDK службы

Используйте пакеты SDK службы Azure IoT в решении для взаимодействия с устройствами и модулями. Например, пакеты SDK для службы можно использовать для чтения и обновления свойств двойника и вызова команд. Поддерживаются следующие языки: C#, Java, Node.js и Python.

Пакеты SDK для служб позволяют получать доступ к сведениям об устройстве из решения, например рабочего стола или веб-приложения. Пакеты SDK для служб включают два пространства имен и объектные модели, которые можно использовать для получения идентификатора модели:

- Клиент службы центра Интернета вещей. Эта служба предоставляет идентификатор модели в качестве свойства двойникаа устройства.

- Клиент Digital двойников. Новый API Digital двойников работает с конструкциями модели на [языке двойников Definition Language (дтдл)](concepts-digital-twin.md) , такими как компоненты, свойства и команды. Интерфейсы API Digital двойника упрощают создание решений IoT самонастраивающийся решениями для сборщиков решений.

:::zone pivot="programming-language-csharp"

[!INCLUDE [iot-pnp-service-devguide-csharp](../../includes/iot-pnp-service-devguide-csharp.md)]

:::zone-end

:::zone pivot="programming-language-java"

[!INCLUDE [iot-pnp-service-devguide-java](../../includes/iot-pnp-service-devguide-java.md)]

:::zone-end

:::zone pivot="programming-language-javascript"

[!INCLUDE [iot-pnp-service-devguide-node](../../includes/iot-pnp-service-devguide-node.md)]

:::zone-end

:::zone pivot="programming-language-python"

[!INCLUDE [iot-pnp-service-devguide-python](../../includes/iot-pnp-service-devguide-python.md)]

:::zone-end

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали о моделировании устройств, ниже приведены некоторые дополнительные ресурсы.

- [Язык определения цифровых двойников (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl)
- [Пакет SDK для устройств C](/azure/iot-hub/iot-c-sdk-ref/)
- [REST API IoT](/rest/api/iothub/device)
- [Руководством по моделированию IoT самонастраивающийся](concepts-modeling-guide.md)
- [Установка и использование средств разработки ДТДЛ](howto-use-dtdl-authoring-tools.md)