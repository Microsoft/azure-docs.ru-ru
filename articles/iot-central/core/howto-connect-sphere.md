---
title: Подключение устройства Azure Sphere в Azure IoT Central | Документация Майкрософт
description: Узнайте, как подключить устройство Azure Sphere (DevKit) к приложению IoT Central Azure.
services: iot-central
ms.service: iot-central
ms.topic: how-to
ms.author: sandeepu
author: sandeeppujar
ms.date: 04/30/2020
ms.custom: device-developer
ms.openlocfilehash: 770f6e56a669ab2d9b425a7a2879eeef5d37377b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92123429"
---
# <a name="connect-an-azure-sphere-device-to-your-azure-iot-central-application"></a>Подключение устройства Azure Sphere к приложению IoT Central Azure

*Эта статья предназначена для разработчиков устройств*

Из этой статьи вы узнаете, как подключить устройство Azure Sphere (DevKit) к приложению IoT Central Azure.

Azure Sphere — это защищенная платформа приложений высокого уровня со встроенными функциями взаимодействия и обеспечения безопасности, предназначенная для устройств, подключенных к Интернету. Она состоит из защищенного подключенного перекрестного микроконтроллера (MCU), специализированной высокоуровневой операционной системы на базе Linux и облачной службы безопасности, которая обеспечивает непрерывную возобновляемую безопасность. Дополнительные сведения см. в статье [Что такое Azure Sphere?](/azure-sphere/product-overview/what-is-azure-sphere).

[Комплекты SDK Azure Sphere](https://azure.microsoft.com/services/azure-sphere/get-started/) предоставляют все необходимое для начала создания прототипов и разработки приложений Azure Sphere. Использование Azure IoT Central с Azure Sphere предоставляет сквозной стек для решения IoT. Azure Sphere обеспечивает поддержку устройств и IoT Central в качестве платформы управляемых приложений IoT без кода.

В этом пошаговом руководстве вы узнаете как:

- создать устройство Azure Sphere в IoT Central с помощью шаблона устройства Azure Sphere DevKit из библиотеки;
- подготовить устройство DevKit Azure Sphere для Azure IoT;
- подключить набор DevKit Azure Sphere к Azure IoT Central;
- и просмотрите данные телеметрии с устройства в IoT Central.

## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить действия, описанные в этой статье, необходимы следующие ресурсы:

- Приложение Azure IoT Central.
- Visual Studio 2019 версии 16.4 (или более новая версия).
- [Пакет средств разработки Azure Sphere MT3620 из Seeed Studios](/azure-sphere/hardware/mt3620-reference-board-design).

> [!NOTE]
> Если у вас нет физического устройства, перейдите после первого шага к последнему разделу, чтобы использовать имитированное устройство.

## <a name="create-the-device-in-iot-central"></a>Создание устройства в IoT Central

Чтобы создать устройство Azure Sphere в IoT Central:

1. В приложении IoT Central Azure перейдите на вкладку **Шаблоны устройств** и выберите **+ Создать**. В разделе **Use a featured device template** (Использование избранного шаблона устройства) выберите **Пример устройства Azure Sphere**.

    :::image type="content" source="media/howto-connect-sphere/sphere-create-template.png" alt-text="Шаблон устройства для Azure Sphere DevKit":::

1. В шаблоне устройства измените представление с именем **Обзор** чтобы показать **температуры** и **нажатие кнопки**.

1. Выберите тип представления **Изменение устройства и облачных данных**, чтобы добавить другое представление, в котором отображается свойство чтения и записи **Индикатор состояния**. Перетащите свойство **Индикатор состояния** в пустой пунктирный прямоугольник на правой стороне формы. Щелкните **Сохранить**.

## <a name="prepare-the-device"></a>Подготовка устройства

Перед подключением устройства Azure Sphere DevKit к IoT Central необходимо [настроить среду устройства и разработки](https://github.com/Azure/azure-sphere-samples/tree/master/Samples/AzureIoT).

## <a name="connect-the-device"></a>Подключение устройства

Чтобы разрешить подключение примера к IoT Central, необходимо [настроить приложение IoT Central Azure, а затем изменить манифест приложения в примере](https://aka.ms/iotcentral-sphere-git-readme).

## <a name="view-the-telemetry-from-the-device"></a>Просмотр данных телеметрии с устройства

Когда устройство подключено к IoT Central, на панели мониторинга отображаются данные телеметрии.

:::image type="content" source="media/howto-connect-sphere/sphere-view.png" alt-text="Панель мониторинга для Azure Sphere DevKit":::

## <a name="create-a-simulated-device"></a>Создайте виртуальное устройство.

Если у вас нет физического устройства Azure Sphere DevKit, можно создать имитированное устройство, чтобы опробовать приложение Azure IoT Central.

Чтобы создать имитированное устройство, выполните приведенные ниже действия.

- Выберите **Устройства > Azure IoT Sphere**
- Выберите **+ Создать**.
- Введите уникальный **идентификатор устройства** и понятное **имя устройства**.
- Включите параметр **Имитация**.
- Нажмите кнопку **создания**.

## <a name="next-steps"></a>Дальнейшие действия

Если вы являетесь разработчиком устройства, вот некоторые из предлагаемых дальнейших действий:

- См. сведения о [Возможности подключения устройств в Azure IoT Central.](./concepts-get-connected.md)
- Узнайте, как [отслеживать подключение устройств с помощью Azure CLI](./howto-monitor-devices-azure-cli.md)