---
title: Подключение приложения IoT Central Azure к службам Dynamics 365 Field Service | Документация Майкрософт
description: Узнайте, как создать комплексное решение с помощью Azure IoT Central и службы Dynamics 365 Field Service.
author: miriambrus
ms.author: miriamb
ms.date: 12/11/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 1d098f56bbadfe115620580c8d93fb6dd021550d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97586680"
---
# <a name="build-end-to-end-solution-with-azure-iot-central-and-dynamics-365-field-service"></a>Создание комплексного решения с помощью Azure IoT Central и службы Dynamics 365 Field Service 
Создатель решения может включить интеграцию приложения IoT Central Azure с другими бизнес-системами. 

Например, в подключенном решении для утилизации отходов можно оптимизировать отправку грузовиков для уборки мусора. Оптимизация может быть выполнена на основе данных датчиков IoT из подключенных ячеек отхода. В [IOT Central подключенном приложении управления отходами](./tutorial-connected-waste-management.md) можно настроить правила и действия и настроить их для активации создания предупреждений в службе полей Dynamics. Этот сценарий выполняется с помощью Power автоматизиру, который будет настроен непосредственно в IoT Central для автоматизации рабочих процессов в приложениях и службах. Кроме того, на основе действий службы в Field Service данные можно отправить обратно в Azure IoT Central. 

## <a name="how-to-connect-your-azure-iot-central-application-with-dynamics-365-field-services"></a>Подключение приложения IoT Central Azure к службам Dynamics 365 Field Service 

Описанные ниже процессы интеграции можно легко реализовать, обладая опытом настройки с самого начала:
* Azure IoT Central может отправлять сведения об аномалиях устройств в подключенную службу Field Service (как оповещение IoT) для диагностики.
* Подключенная служба Field Service может создавать сценарии или заказы на работу, активируемые аномалиями устройств.
* Подключенная служба Field Service может запланировать визит специалистов для осмотра, чтобы предотвратить простои.
* Панель мониторинга устройств IoT Central Azure можно обновить с помощью соответствующей службы и сведений о расписании.


## <a name="next-steps"></a>Дальнейшие действия
* Узнайте больше о [шаблонах IoT Central для государственных организаций](./overview-iot-central-government.md).
* Дополнительные сведения об [IoT Central](../core/overview-iot-central.md)
* Дополнительные сведения о службах [Dynamics 365 Field Service](/dynamics365/field-service/cfs-iot-overview)
