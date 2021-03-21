---
title: Преобразование протоколов Modbus с помощью шлюзов в Azure IoT Edge | Документация Майкрософт
description: Разрешите устройствам, использующим Modbus TCP, взаимодействовать с Центром Интернета, создав устройство шлюза IoT Edge.
author: kgremban
manager: philmea
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 11/19/2019
ms.author: kgremban
ms.openlocfilehash: 0388520903e208b3225375d5cee81e8321740a1b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103487740"
---
# <a name="connect-modbus-tcp-devices-through-an-iot-edge-device-gateway"></a>Подключение устройств Modbus TCP через шлюз устройств IoT Edge

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

Если вы хотите подключить устройства IoT, использующие протоколы Modbus TCP или рту, к центру Интернета вещей Azure, вы можете использовать устройство IoT Edge в качестве шлюза. Устройство шлюза считывает данные с устройства Modbus, а затем передает эти данные в облако с помощью поддерживаемого протокола.

![Устройства MODBUS подключаются к центру Интернета вещей через шлюз IoT Edge](./media/deploy-modbus-gateway/diagram.png)

В этой статье объясняется, как создать собственный образ контейнера для модуля Modbus (или можно использовать предварительно подготовленный пример), а затем развернуть его на устройство IoT Edge, которое будет выполнять роль шлюза.

В этой статье предполагается, что вы используете протокол Modbus TCP. Дополнительные сведения о том, как настроить модуль для поддержки Modbus RTU, см. в проекте [модуля Modbus для Azure IoT Edge](https://github.com/Azure/iot-edge-modbus) в GitHub.

## <a name="prerequisites"></a>Предварительные требования

* Устройство Azure IoT Edge. Пошаговое руководство по настройке одного из них см. в статье [развертывание Azure IOT EDGE в Windows](quickstart.md) или [Linux](quickstart-linux.md).
* Строка подключения первичного ключа для устройства IoT Edge.
* Физическое устройство или имитация устройства, которые поддерживают Modbus TCP. Необходимо знать его IPv4-адрес.

## <a name="prepare-a-modbus-container"></a>Подготовка контейнера Modbus

Если вы хотите протестировать функции шлюза Modbus, корпорация Майкрософт предлагает использовать пример модуля. Вы можете получить доступ к модулю из Azure Marketplace, [Modbus](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft_iot.edge-modbus?tab=Overview)или с помощью URI образа `mcr.microsoft.com/azureiotedge/modbus:1.0` .

Если вы хотите создать собственный модуль и настроить его для своей среды, в проекте GitHub можно найти [модуль Modbus для Azure IoT Edge](https://github.com/Azure/iot-edge-modbus) с открытым кодом. Следуйте инструкциям в этом проекте, чтобы создать образ контейнера. Чтобы создать образ контейнера, см. статью [Разработка модулей C# в Visual Studio](./how-to-visual-studio-develop-module.md) или [разработка модулей в Visual Studio Code](how-to-vs-code-develop-module.md). В этих статьях содержатся инструкции по созданию новых модулей и публикации образов контейнеров в реестре.

## <a name="try-the-solution"></a>Попробуйте решение

В этом разделе описывается развертывание примера модуля Modbus Майкрософт на устройстве IoT Edge.

1. Найдите нужный Центр Интернета вещей на [портале Azure](https://portal.azure.com/).

2. Щелкните **IoT Edge** и выберите устройство IoT Edge.

3. Щелкните **Set modules** (Настроить модули).

4. В разделе **модули IOT Edge** добавьте модуль Modbus:

   1. Щелкните раскрывающийся список **Добавить** и выберите **модуль Marketplace**.
   2. Найдите `Modbus` и выберите **модуль TCP Modbus** в Майкрософт.
   3. Модуль автоматически настраивается для центра Интернета вещей и отображается в списке модулей IoT Edge. Маршруты также настраиваются автоматически. Выберите **Review + create** (Просмотреть и создать).
   4. Проверьте манифест развертывания и нажмите кнопку **создать**.

5. Выберите Модуль Modbus, `ModbusTCPModule` в списке и перейдите на вкладку **двойника Settings (параметры модуля** ). Обязательный JSON для модуля, двойника требуемые свойства, заполняется автоматически.

6. Найдите свойство **славеконнектион** в JSON и присвойте ему значение IPv4-адреса устройства MODBUS.

7. Нажмите кнопку **Обновить**.

8. Выберите **Проверка + создать**, просмотрите развертывание, а затем выберите **создать**.

9. Вернитесь на страницу сведений об устройстве и выберите **Обновить**. Вы увидите, что новый `ModbusTCPModule` модуль работает вместе со средой выполнения IOT Edge.

## <a name="view-data"></a>Просмотр данных

Просмотр данных, поступающих через модуль Modbus:

```cmd/sh
iotedge logs modbus
```

Вы также можете просмотреть данные телеметрии, которые устройство отправляет, используя [расширение центра Интернета вещей Azure для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit) (ранее — расширение Azure IOT Toolkit).

## <a name="next-steps"></a>Следующие шаги

* Дополнительные сведения о том, как IoT Edge устройства могут выступать в качестве шлюзов, см. в разделе [Создание устройства IOT EDGE, работающего в качестве прозрачного шлюза](./how-to-create-transparent-gateway.md).
* Дополнительные сведения о работе модулей IoT Edge см. в разделе [Знакомство с Azure IOT Edgeными модулями](iot-edge-modules.md).