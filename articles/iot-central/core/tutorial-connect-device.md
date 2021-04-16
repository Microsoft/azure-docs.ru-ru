---
title: Учебник. Подключение универсального клиентского приложения к Azure IoT Central | Документация Майкрософт
description: Из этого учебника вы узнаете, как разработчик устройства может подключить устройство, на котором выполняются клиентские приложения C, C#, Java, JavaScript или Python, к приложению Azure IoT Central. Вы также измените автоматически созданный шаблон устройства, добавив представления, позволяющие оператору взаимодействовать с подключенным устройством.
author: dominicbetts
ms.author: dobett
ms.date: 11/24/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.custom:
- mqtt
- device-developer
zone_pivot_groups: programming-languages-set-twenty-six
ms.openlocfilehash: bbf94b6e000d5c082debd6a0d41a8d62b8b3f26e
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106491106"
---
# <a name="tutorial-create-and-connect-a-client-application-to-your-azure-iot-central-application"></a>Руководство по созданию клиентского приложения и его подключению к приложению Azure IoT Central

*Эта статья предназначена для создателей решений и разработчиков устройств.*

Из этого учебника вы узнаете, как разработчик устройства может подключить клиентское приложение к приложению Azure IoT Central. Приложение моделирует поведение устройства контроллера температуры. При подключении приложения к IoT Central оно отправляет идентификатор модели устройства контроллера температуры. IoT Central использует идентификатор модели, чтобы получить модель устройства и автоматически создать шаблон устройства. Затем вы добавите настройки и представления в шаблон устройства, чтобы разрешить оператору взаимодействовать с устройством.

В этом руководстве описано следующее.

> [!div class="checklist"]
> * создание кода устройства и его выполнение для подключения к приложению IoT Central;
> * просмотр смоделированной телеметрии, которую отправляет устройство;
> * добавление пользовательских представлений в шаблон устройства;
> * публикация шаблона устройства;
> * управление свойствами устройства через представления;
> * вызов команд для управления устройством.

:::zone pivot="programming-language-ansi-c"

[!INCLUDE [iot-central-connect-device-c](../../../includes/iot-central-connect-device-c.md)]

:::zone-end

:::zone pivot="programming-language-csharp"

[!INCLUDE [iot-central-connect-device-csharp](../../../includes/iot-central-connect-device-csharp.md)]

:::zone-end

:::zone pivot="programming-language-java"

[!INCLUDE [iot-central-connect-device-java](../../../includes/iot-central-connect-device-java.md)]

:::zone-end

:::zone pivot="programming-language-javascript"

[!INCLUDE [iot-central-connect-device-nodejs](../../../includes/iot-central-connect-device-nodejs.md)]

:::zone-end

:::zone pivot="programming-language-python"

[!INCLUDE [iot-central-connect-device-python](../../../includes/iot-central-connect-device-python.md)]

:::zone-end

## <a name="view-raw-data"></a>Просмотр необработанных данных

Разработчик для устройств может использовать представление **Необработанные данные** для просмотра необработанных данных, отправляемых устройством в IoT Central.

:::image type="content" source="media/tutorial-connect-device/raw-data.png" alt-text="Представление необработанных данных":::

В этом представлении можно выбрать отображаемые столбцы и задать диапазон времени для просмотра. В столбце **Немоделированные данные** отображаются данные устройства, которые не соответствуют ни одному из определений свойств или телеметрии в шаблоне устройства.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [iot-central-clean-up-resources](../../../includes/iot-central-clean-up-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

Если вы хотите продолжать работу с набором руководств по IoT Central, чтобы узнать больше о создании решении IoT Central, ознакомьтесь со следующим руководством:

> [!div class="nextstepaction"]
> [Создайте шаблон устройства шлюза](./tutorial-define-gateway-device-type.md)

Теперь, когда вы как разработчик устройства узнали о принципах создания устройств с помощью Java, ознакомьтесь со следующими руководствами:

* Прочитайте статью [Что такое шаблоны устройств?](./concepts-device-templates.md), чтобы узнать больше о роли шаблонов устройств при реализации кода устройства.
* Узнайте о [регистрации устройств с помощью IoT Central и безопасном подключении устройств к Azure IoT Central](./concepts-get-connected.md).
* Дополнительные сведения о данных, которыми устройство обменивается с IoT Central см. в статье [Полезные данные телеметрии, свойств и команд](concepts-telemetry-properties-commands.md).
* Прочитайте [руководство для разработчиков по работе с устройствами IoT Plug and Play](../../iot-pnp/concepts-developer-guide-device.md).
