---
title: Отправка сообщений на сервер MQTT с помощью клиентской библиотеки Azure MQTT
description: Узнайте, как использовать клиентскую библиотеку MQTT для отправки сообщений в брокер MQTT. Также Узнайте, как настроить mXChip IoT DevKit как клиент MQTT.
author: liydu
manager: jeffya
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 04/02/2018
ms.author: liydu
ms.custom: mqtt
ms.openlocfilehash: fb8bf593568825793a1a205a2955599b16fa78cf
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92151759"
---
# <a name="send-messages-to-an-mqtt-server"></a>Отправка сообщений на сервер MQTT

Подключение систем Интернета вещей к Интернету зачастую бывает нестабильным, медленным или низкого качества. Протокол MQTT для связи между компьютерами (M2M) специально разработан с учетом этих трудностей. 

Используемая в нашем примере клиентская библиотека MQTT входит в проект [Eclipse Paho](https://www.eclipse.org/paho/), который предоставляет API для применения MQTT в разных транспортных средах.

## <a name="what-you-learn"></a>Что вы узнаете

В этом проекте вы узнаете:
- как использовать клиентскую библиотеку MQTT для отправки сообщений на брокер MQTT;
- как настроить MXChip Iot DevKit в качестве клиента MQTT.

## <a name="what-you-need"></a>Необходимые элементы

Выполните указания в [руководстве по началу работы](./iot-hub-arduino-iot-devkit-az3166-get-started.md), чтобы перейти к таким этапам:

* Подключение DevKit к сети Wi-Fi.
* Подготовка среды разработки

## <a name="open-the-project-folder"></a>Открытие папки проекта

1. Если устройство DevKit подключено к компьютеру, отключите его.

2. Запустите VSCode.

3. Подключите DevKit на компьютере.

## <a name="open-the-mqttclient-sample"></a>Открытие примера MQTTClient

Разверните раздел **Arduino Examples** (Примеры Arduino) слева, перейдите в папку **Examples for MXCHIP AZ3166 > MQTT** (Примеры для MXCHIP AZ3166 > MQTT) и выберите **MQTTClient**. Откроется новое окно VS Code с папкой проекта.

> [!NOTE]
> Пример также можно открыть из палитры команд. Нажмите `Ctrl+Shift+P` (macOS: `Cmd+Shift+P`) для вызова палитры команд. Введите **Arduino**, а затем найдите и выберите **Arduino: примеры**.

## <a name="build-and-upload-the-arduino-sketch-to-the-devkit"></a>Создание эскиза Arduino и его передача на DevKit

Нажмите клавиши `Ctrl+P` (macOS: `Cmd+P`), чтобы выполнить команду `task device-upload`. По завершении передачи устройство DevKit перезапускается и запускает эскиз.

![На снимке экрана показано окно командной строки, которое отправляет и выполняет эскиз Arduino.](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/device-upload.jpg)

> [!NOTE]
> Вы можете увидеть сообщение об ошибке "Error: AZ3166: Unknown package" (Ошибка AZ3166: неизвестный пакет). Такая ошибка возникает, если индекс пакета платы неправильно обновлен. Чтобы устранить эту ошибку, ознакомьтесь с [разделом по разработке](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#development) в статье с часто задаваемыми вопросами об IoT DevKit.

## <a name="test-the-project"></a>Тестирование проекта

В VS Code выполните следующую процедуру, чтобы открыть и настроить средство Serial Monitor:

1. Щелкните `COM[X]` слово в строке состояния, чтобы задать правильный COM-порт с `STMicroelectronics` : на ![ снимке экрана показано Visual Studio Code с выбранной COM8ой микроэлектроники.](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/set-com-port.jpg)

2. Щелкните значок подключения к электросети в строке состояния, чтобы открыть последовательный монитор: ![ на снимке экрана отображается сводка по выпуску и значок подключения к электросети в строке состояния.](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/serial-monitor.jpg)
  
3. В строке состояния щелкните число, представляющее скорость, и задайте для него значение: на `115200` ![ снимке экрана показана установка скорости передачи в Visual Studio Code.](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/set-baud-rate.jpg)

Serial Monitor отобразит все сообщения, отправленные из примененного эскиза. Этот эскиз подключается к DevKit по Wi-Fi. После успешного подключения по Wi-Fi эскиз отправляет сообщение на брокер MQTT. Затем в примере поочередно отправляются два сообщения iot.eclipse.org с параметрами QoS 0 и QoS 1, соответственно.

![На снимке экрана показан последовательный монитор, отображающий сообщения, отправленные наброской.](media/iot-hub-arduino-iot-devkit-az3166-mqtt-helloworld/serial-output.jpg)

## <a name="problems-and-feedback"></a>Проблемы и обратная связь

Если вы столкнулись с проблемами, ознакомьтесь с [вопросами и ответами](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) об IoT DevKit или используйте один из следующих каналов связи:

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="see-also"></a>См. также раздел

* [Подключение платы IoT DevKit AZ3166 к Центру Интернета вещей Azure в облаке](iot-hub-arduino-iot-devkit-az3166-get-started.md)
* [Shake, Shake: получение сообщений из Twitter](iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message.md)

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как настроить MXChip IOT DevKit в качестве клиента MQTT и использовать клиентскую библиотеку MQTT для отправки сообщений в MQTT Broker, Вот рекомендуемый следующий шаг: [Обзор ускорителя решений удаленного мониторинга Azure IOT](/azure/iot-suite/)