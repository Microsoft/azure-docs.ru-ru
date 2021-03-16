---
title: Что такое Azure IoT Edge для Linux в Windows | Документация Майкрософт
description: Общие сведения о запуске модулей IoT Edge Linux на устройствах Windows 10
author: kgremban
manager: philmea
ms.reviewer: twarwick
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 01/20/2021
ms.author: kgremban
monikerRange: =iotedge-2018-06
ms.openlocfilehash: 330eaf5c12372347917e9f3a4aeafb6a2088c592
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103492580"
---
# <a name="what-is-azure-iot-edge-for-linux-on-windows-preview"></a>Что такое Azure IoT Edge для Linux в Windows (предварительная версия)

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

Azure IoT Edge для Linux в Windows позволяет запускать контейнерные рабочие нагрузки Linux вместе с приложениями Windows в развертываниях Windows IoT. Компании, использующие Windows IoT для работы с пограничными устройствами, теперь могут воспользоваться преимуществами облачных решений аналитики, созданных в Linux.

IoT Edge для Linux в Windows работает путем запуска виртуальной машины Linux на устройстве Windows. Виртуальная машина Linux предварительно устанавливается со средой выполнения IoT Edge. Все IoT Edge модули, развернутые на устройстве, выполняются внутри виртуальной машины. В то же время приложения Windows, работающие на устройстве узла Windows, могут взаимодействовать с модулями, работающими на виртуальной машине Linux.

Начните [работу](how-to-install-iot-edge-on-windows.md) с предварительной версией уже сегодня.

>[!NOTE]
>Рекомендуется принять наши [опросы по продуктам](https://aka.ms/AzEFLOW-Registration) , чтобы помочь нам улучшить Azure IOT Edge для Linux в Windows на основе IOT Edge фоновых данных и целей. Этот опрос также можно использовать для регистрации будущих Azure IoT Edge для объявлений Linux в Windows.

## <a name="components"></a>Components

IoT Edge для Linux в Windows использует следующие компоненты, чтобы разрешить рабочим нагрузкам Linux и Windows параллельно работать друг с другом и легко взаимодействовать:

* **Виртуальная машина Linux под управлением Azure IOT Edge**: виртуальная машина Linux, основанная на первой [Маринер](https://github.com/microsoft/CBL-Mariner) операционной системе корпорации Майкрософт, построена на основе IOT Edge среды выполнения и проверена как среда, поддерживаемая уровнями 1 для IOT Edge рабочих нагрузок.

* **Центр администрирования Windows**. расширение IOT Edge для центра администрирования Windows упрощает установку, настройку и диагностику IOT EDGE на виртуальной машине Linux. Центр администрирования Windows может развертывать IoT Edge для Linux в Windows на локальном устройстве, а также подключаться к целевым устройствам и управлять ими удаленно.

* **Центр обновления Майкрософт**. интеграция с центр обновления Майкрософт поддерживает компоненты среды выполнения Windows, виртуальную машину Маринер Linux и IOT Edge обновления.

![Windows и виртуальная машина Linux работают параллельно, а центр администрирования Windows контролирует оба компонента.](./media/iot-edge-for-linux-on-windows/architecture-and-communication.png)

Двусторонний обмен данными между процессом Windows и виртуальной машиной Linux означает, что процессы Windows могут предоставлять пользовательские интерфейсы или аппаратные прокси-серверы для рабочих нагрузок в контейнерах Linux.

## <a name="samples"></a>Примеры

IoT Edge для Linux в Windows подчеркивает взаимодействие между компонентами Linux и Windows.

Примеры, демонстрирующие взаимодействие между приложениями Windows и IoT Edgeными модулями, см. в разделе [примеры Windows 10 IOT](https://github.com/microsoft/Windows-IoT-Samples).

## <a name="public-preview"></a>Общедоступная предварительная версия

IoT Edge для Linux в Windows в настоящее время находится в [общедоступной предварительной версии](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Процессы установки и управления могут отличаться от общих для общедоступных функций.

## <a name="support"></a>Поддержка

Используйте IoT Edge каналы поддержки и обратной связи, чтобы получить помощь в IoT Edge для Linux в Windows.

**Отчеты об ошибках** — ошибки могут быть получены на [странице "проблемы](https://github.com/azure/iotedge/issues) " IOT Edge проекте с открытым кодом. Ошибки, связанные с Azure IoT Edge для Linux в Windows, можно получить на [странице проблем iotedge-ефлов](https://github.com/azure/iotedge-eflow/issues).

**Служба поддержки пользователей корпорации Майкрософт**. Пользователи, приобретающие [план поддержки,](https://azure.microsoft.com/support/plans/) могут обращаться в службу поддержки пользователей корпорации Майкрософт, создав запрос прямо на [портале Azure](https://ms.portal.azure.com/signin/index/?feature.settingsportalinstance=mpac).

**Запросы функций.** Для продукта Azure IoT Edge запросы функций отслеживаются на специальной странице [User Voice](https://feedback.azure.com/forums/907045-azure-iot-edge).

## <a name="next-steps"></a>Дальнейшие действия

Просмотрите [IOT Edge для Linux в Windows 10 IOT Корпоративная](https://aka.ms/EFLOWPPC9) , чтобы получить дополнительные сведения и пример в действии.

Выполните действия, описанные в статье [Установка и инициализация Azure IOT Edge для Linux на устройстве Windows](how-to-install-iot-edge-on-windows.md) , чтобы настроить устройство с IOT Edge для Linux в Windows.
