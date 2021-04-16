---
title: включить файл
description: включить файл
services: iot-central
author: dominicbetts
ms.service: iot-central
ms.topic: include
ms.date: 03/12/2020
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 77fdaf297fff0e145b1dd53908887bc14f9d3f14
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106491134"
---
<!-- All needs updating -->
Оператор приложения Azure IoT Central может выполнять следующие действия:

* Просматривать данные телеметрии, переданные двумя компонентами термостата, на странице **Обзор**:

    :::image type="content" source="media/iot-central-monitor-thermostat/view-telemetry.png" alt-text="Просмотр телеметрии устройства":::

* Просматривать свойства устройства на странице **Сведения**. На этой странице показаны свойства компонента со сведениями об устройстве и двух компонентов термостата:

    :::image type="content" source="media/iot-central-monitor-thermostat/about-properties.png" alt-text="Просмотр свойств устройства":::

## <a name="customize-the-device-template"></a>Настройка шаблона устройства

Разработчик решения может настроить шаблон устройства, который автоматически создается IoT Central при подключении устройства контроллера температуры.

Чтобы добавить свойство облака для хранения имени клиента, связанного с устройством, выполните следующие действия.

1. В приложении IoT Central перейдите к шаблону устройства **Контроллер температуры** на странице **Шаблоны устройств**.

1. В шаблоне устройства **Контроллер температуры** выберите **Облачные свойства**.

1. Выберите команду **Добавить облачное свойство**. Введите *имя клиента* в качестве **отображаемого имени** и выберите **строку** в качестве **схемы**. Затем выберите **Сохранить**.

Чтобы настроить отображение команд **Get Max-Min report** в приложении IoT Central, выполните следующие действия.

1. Выберите **Настроить** в шаблоне устройства.

1. Для **getMaxMinReport (thermostat1)** замените *Get Max-Min report.* на *Get thermostat1 status report*.

1. Для **getMaxMinReport (thermostat2)** замените *Get Max-Min report.* на *Get thermostat2 status report*.

1. Щелкните **Сохранить**.

Чтобы настроить отображение свойств **целевой температуры**, доступных для записи, в приложении IoT Central, выполните следующие действия:

1. Выберите **Настроить** в шаблоне устройства.

1. Для **targetTemperature (thermostat1)** замените *Target Temperature* на *Target Temperature (1)* .

1. Для **targetTemperature (thermostat2)** замените *Target Temperature* на *Target Temperature (2)* .

1. Щелкните **Сохранить**.

Компоненты термостата в модели **Контроллер температуры** включают доступное для записи свойство **Target Temperature** (Целевая температура), а шаблон устройства включает облачное свойство **Customer Name** (Имя клиента). Создайте представление, которое оператор может использовать для изменения этих свойств, выполнив следующие действия:

1. Выберите **Views** (Представления) и щелкните плитку **Editing device and cloud data** (Изменение данных об устройстве и облаке).

1. Введите имя формы _Properties_ (Свойства).

1. Выберите свойства **Target Temperature (1)** , **Target Temperature (2)** и **Customer Name**. Затем щелкните **Add section** (Добавить раздел).

1. Сохраните изменения.

:::image type="content" source="media/iot-central-monitor-thermostat/properties-view.png" alt-text="Представление для обновления значений свойств":::

## <a name="publish-the-device-template"></a>Публикация шаблона устройства

Прежде чем оператор сможет просмотреть и использовать выполненные настройки, необходимо опубликовать шаблон устройства.

В меню шаблона устройства выберите устройство **термостата** и щелкните **Опубликовать**. На панели **Опубликовать этот шаблон устройства в приложении** щелкните **Опубликовать**.

Теперь оператор может использовать представление **Свойства** для обновления значений свойств и вызова команд **Get thermostat1 status report** и **Get thermostat2 status report** на странице команд устройства:

* Измените значения доступных для записи свойств на странице **Properties** (Свойства).

    :::image type="content" source="media/iot-central-monitor-thermostat/update-properties.png" alt-text="Обновление свойств устройства":::

* Выполните команды на странице **Команды**.

    :::image type="content" source="media/iot-central-monitor-thermostat/call-command.png" alt-text="Вызов команды":::

    :::image type="content" source="media/iot-central-monitor-thermostat/command-response.png" alt-text="Просмотр ответа команды":::
