---
title: Текущий выпуск IoT Plug and Play | Документация Майкрософт
description: Узнайте, что входит в текущий выпуск IoT Plug and Play.
author: dominicbetts
ms.author: dobett
ms.date: 10/01/2020
ms.topic: overview
ms.custom: mvc
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 7b6669ca55dc9b94dc5d702e54b42011120b8812
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99831881"
---
# <a name="what-is-in-the-current-iot-plug-and-play-release"></a>Каков текущий выпуск IoT Plug and Play?

В этой статье перечислены средства, пакеты SDK и API, поддерживающие текущий выпуск IoT Plug and Play. Отображаемые номера версий соответствуют номеру версии на момент выпуска общедоступной версии IoT Plug and Play. Номера версий могут увеличиваться по мере новых выпусков.

## <a name="modeling-language"></a>Язык моделирования

[Язык определения цифровых двойников (DTDL) версии 2](https://github.com/Azure/opendigitaltwins-dtdl)

Дополнительные сведения о том, как устройства IoT Plug and Play работают с DTDL, см. в статье [Соглашения IoT Plug and Play](concepts-convention.md).

## <a name="tools-and-utilities"></a>Средства и служебные программы

- Обозреватель Интернета вещей Azure версии 0.12.0.

    Дополнительные сведения см. в статье [Установка и использование обозревателя Интернета вещей Azure](howto-use-iot-explorer.md).

- Расширение VS Code версии 1.0.0.

    Дополнительные сведения см. в статье [Установка и использование средств разработки DTDL](howto-use-dtdl-authoring-tools.md).

- Расширение Visual Studio 2019 версии 1.0.0.

    Дополнительные сведения см. в статье [Установка и использование средств разработки DTDL](howto-use-dtdl-authoring-tools.md).

- Расширение Интернета вещей для Azure CLI версии 0.10.0.

    Расширение Интернета вещей Azure содержит команды, помогающие сертифицировать устройства. См. раздел `az iot product -h`.

## <a name="libraries-and-sdks"></a>Библиотеки и пакеты SDK

Дополнительные сведения о библиотеках и пакетах SDK см. в статье [Пакеты SDK Microsoft для IoT Plug and Play](libraries-sdks.md).

- Пакет SDK для устройства C [vcpkg 1.3.9](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/setting_up_vcpkg.md)
- Встроенный пакет SDK для устройства C [GitHub](https://github.com/Azure/azure-sdk-for-c/)
- Пакет SDK для устройства .NET [NuGet 1.31.0](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client)
- Пакет SDK для устройства Java [Maven 1.26.0](https://mvnrepository.com/artifact/com.microsoft.azure.sdk.iot/iot-device-client)
- Пакет SDK для устройства Python [pip 2.3.0](https://pypi.org/project/azure-iot-device/)
- Пакет SDK для устройства Node.js [npm 1.17.2](https://www.npmjs.com/package/azure-iot-device)
- Служба Центра Интернета вещей для .NET [NuGet 1.27.1](https://www.nuget.org/packages/Microsoft.Azure.Devices )
- Служба Центра Интернета вещей для Java [Maven 1.26.0](https://mvnrepository.com/artifact/com.microsoft.azure.sdk.iot/iot-service-client/1.26.0)
- Служба Центра Интернета вещей для Node.js [npm 1.13.0](https://www.npmjs.com/package/azure-iothub)
- Служба Центра Интернета вещей или Digital Twins для Python [pip 2.2.3](https://pypi.org/project/azure-iot-hub)
- Средство синтаксического анализа DTDL [NuGet](https://www.nuget.org/packages/Microsoft.Azure.DigitalTwins.Parser).

## <a name="rest-apis"></a>Интерфейсы REST API

REST API [30.09.2020](/rest/api/iothub).

Дополнительные сведения см. в статье [Руководство для разработчиков IoT Plug and Play](concepts-developer-guide-service.md).

## <a name="iot-hub"></a>Центр Интернета вещей

Центр Интернета вещей поддерживает IoT Plug and Play во всех регионах. IoT Plug and Play поддерживают только центры Интернета вещей уровня "Стандартный" или "Бесплатный".

## <a name="announcements"></a>Объявления

Сведения о текущих и предыдущих объявлениях IoT Plug and Play см. в следующих записях блога:

- [Обновление общедоступной предварительной версии (опубликовано 29 августа 2020 г.)](https://techcommunity.microsoft.com/t5/internet-of-things/add-quot-plug-and-play-quot-to-your-iot-solutions/ba-p/1548531)
- [Подготовка и сертификация устройств для IoT Plug and Play (опубликовано 26 августа 2020 г.)](https://azure.microsoft.com/blog/prepare-and-certify-your-devices-for-iot-plug-and-play/)
- [Теперь IoT Plug and Play доступен в предварительной версии (опубликовано 22 августа 2019 г.)](https://azure.microsoft.com/blog/iot-plug-and-play-is-now-available-in-preview/)
- [Сборка с помощью Azure IoT Central и IoT Plug and Play (опубликовано 7 мая 2019 г.)](https://azure.microsoft.com/blog/build-with-azure-iot-central-and-iot-plug-and-play/)

## <a name="next-steps"></a>Дальнейшие действия

На следующем шаге мы рекомендуем ознакомиться со статьей [Что такое IoT Plug and Play?](overview-iot-plug-and-play.md).