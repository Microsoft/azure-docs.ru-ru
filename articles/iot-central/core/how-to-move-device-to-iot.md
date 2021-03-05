---
title: Как переместить устройство в Azure IoT Central из центра Интернета вещей
description: Как переместить устройство в IoT Central Azure из центра Интернета вещей
author: TheRealJasonAndrew
ms.author: v-anjaso
ms.date: 02/20/2021
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 543c3f1c72857098540cc2a77e8a0093b907b799
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102210841"
---
# <a name="how-to-transfer-a-device-to-azure-iot-central-from-iot-hub"></a>Как передавать устройство в IoT Central Azure из центра Интернета вещей

*Эта статья относится к операторам и разработчикам устройств.*  

В этой статье описывается, как передавать устройство в приложение IoT Central Azure из центра Интернета вещей. 

Сначала устройство подключается к конечной точке DPS для получения сведений, необходимых для подключения к приложению. На внутреннем уровне приложение IoT Central использует центр Интернета вещей для управления подключением устройства.  

Устройство может быть подключено к центру Интернета вещей напрямую с помощью строки подключения или с помощью DPS. [Служба подготовки устройств для центра Интернета вещей Azure (DPS)](../../iot-dps/about-iot-dps.md) — это маршрут для IOT Central.

## <a name="to-move-the-device-to-azure-iot-central"></a>Перенос устройства в Azure IoT Central

Чтобы подключить устройство к IoT Central из центра Интернета вещей, необходимо обновить устройство с помощью:

* [Идентификатор области](../../iot-dps/concepts-service.md) IOT Central приложения.
* Ключ, полученный либо из ключа [SAS группы](concepts-get-connected.md) , либо с [помощью сертификата X. 509](../../iot-hub/iot-hub-x509ca-overview.md) .

Для взаимодействия с IoT Central должен существовать шаблон устройства, моделирующий свойства, данные телеметрии и команды, реализуемые устройством. Дополнительные сведения см. [в статьях подключение к IOT Central](concepts-get-connected.md) и [что такое шаблоны устройств?](concepts-device-templates.md)

## <a name="next-steps"></a>Дальнейшие действия

Если вы являетесь разработчиком устройства, вот некоторые из предлагаемых дальнейших действий:

- Ознакомьтесь с примером кода, в котором показано, как использовать маркеры SAS в [руководстве по созданию и подключению клиентского приложения к приложению IOT Central Azure.](tutorial-connect-device.md)
- Узнайте, как [подключать устройства с помощью сертификатов X. 509, используя пакет SDK для устройств Node.js для IOT Central приложения](how-to-connect-devices-x509.md) .
- Узнайте, как [отслеживать подключение устройств с помощью Azure CLI](./howto-monitor-devices-azure-cli.md)
- Узнайте, как [определить новый тип устройств IOT в приложении IOT Central Azure](./howto-set-up-template.md) .
- Узнайте об [устройствах Azure IOT EDGE и Azure IOT Central](./concepts-iot-edge.md)