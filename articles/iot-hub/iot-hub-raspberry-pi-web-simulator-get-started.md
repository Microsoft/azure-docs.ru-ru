---
title: Подключение веб-симулятора Raspberry Pi к центру Интернета вещей Azure (Node.js)
description: Подключение веб-симулятора Raspberry Pi к Центру Интернета вещей для передачи данных с него в облако Azure.
author: wesmc7777
manager: philmea
keywords: симулятор raspberry pi, raspberry pi и центр интернета вещей azure, raspberry pi и центр интернета вещей, отправка данных с raspberry pi в облако, подключение raspberry pi к облаку
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 04/11/2018
ms.author: wesmc
ms.custom:
- 'Role: Cloud Development'
- devx-track-js
ms.openlocfilehash: 702dee108577665eded6dd1a92203236d74e866e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91308355"
---
# <a name="connect-raspberry-pi-online-simulator-to-azure-iot-hub-nodejs"></a>Подключение онлайн-симулятора Raspberry Pi к Центру Интернета вещей Azure (Node.js)

[!INCLUDE [iot-hub-get-started-device-selector](../../includes/iot-hub-get-started-device-selector.md)]

В этом руководстве описано, как начать работу с онлайн-симулятором Raspberry Pi. Кроме того, вы узнаете, как можно легко подключить симулятор Pi к облаку с помощью [Центра Интернета вещей Azure](about-iot-hub.md).

<p>
<div id="diag" style="width:100%; text-align:center">
<img src="media/iot-hub-raspberry-pi-web-simulator/3-banner.png" alt="Connect Raspberry Pi web simulator to Azure IoT Hub" width="400">
</div>
</p>
<p>
<div id="button" style="width:100%; text-align:center">
<a href="https://azure-samples.github.io/raspberry-pi-web-simulator/#getstarted" target="_blank">
<img src="media/iot-hub-raspberry-pi-web-simulator/6-button-default.png" alt="Start Raspberry Pi simulator" width="400" onmouseover="this.src='media/iot-hub-raspberry-pi-web-simulator/5-button-click.png';" onmouseout="this.src='media/iot-hub-raspberry-pi-web-simulator/6-button-default.png';">
</a>
</div>
</p>

Если вы используете физические устройства, ознакомьтесь со статьей [Подключение Raspberry Pi к Центру Интернета вещей Azure (Node.js)](iot-hub-raspberry-pi-kit-node-get-started.md), чтобы приступить к работе.

## <a name="what-you-do"></a>Что нужно сделать

* Изучим основы использования онлайн-симулятора Raspberry Pi.

* Создайте Центр Интернета вещей.

* Зарегистрируем устройство для Pi в Центре Интернета вещей.

* Запустим пример приложения на Pi для отправки симулированных данных датчика в Центр Интернета вещей.

Подключите виртуальное устройство Raspberry Pi к созданному Центру Интернета вещей. Затем запустите пример приложения с помощью симулятора, чтобы создать данные датчика. После этого отправим данные с датчика в Центр Интернета вещей.

## <a name="what-you-learn"></a>Что вы узнаете

* Как создать Центр Интернета вещей Azure и получить строку подключения нового устройства. Если у вас нет учетной записи Azure, [Создайте бесплатную пробную учетную запись Azure](https://azure.microsoft.com/free/) всего за несколько минут.

* Как использовать онлайн-симулятор Raspberry Pi.

* Как отправить данные датчика в Центр Интернета вещей.

## <a name="overview-of-raspberry-pi-web-simulator"></a>Общие сведения о веб-симуляторе Raspberry Pi

Нажмите кнопку, чтобы запустить онлайн-симулятор Raspberry Pi.

> [!div class="button"]
> <a href="https://azure-samples.github.io/raspberry-pi-web-simulator/#GetStarted" target="_blank">Запуск симулятора Raspberry Pi</a>

В веб-симуляторе есть 3 области.

1. Область сборки. В стандартной схеме Pi соединяется с датчиком BME280 и светодиодным индикатором. В предварительной версии эта область заблокирована, поэтому вы не можете выполнить настройку.

2. Область кодирования. Интерактивный редактор кода, позволяющий писать код, используя Raspberry Pi. Стандартный пример приложения позволяет выполнить сбор данных датчика BME280 и отправить их в Центр Интернета вещей. Приложение полностью совместимо с реальными устройствами Pi. 

3. Встроенное окно консоли. Показывает выходные данные вашего кода. В верхней части этого окна есть 3 кнопки.

   * **Запуск.** Запуск приложения в области кодирования.

   * **Сброс.** Сброс параметров области кодирования до стандартных параметров примера приложения.

   * **Fold/Expand** (Свернуть или развернуть). В правой части расположена кнопка, с помощью которой можно свернуть или развернуть окно консоли.

> [!NOTE]
> Веб-симулятор Raspberry Pi доступен в предварительной версии. Оставляйте свои комментарии в [чате Gitter](https://gitter.im/Microsoft/raspberry-pi-web-simulator). Исходный код доступен на сайте [GitHub](https://github.com/Azure-Samples/raspberry-pi-web-simulator).

![Общие сведения об онлайн-симуляторе Pi](media/iot-hub-raspberry-pi-web-simulator/0-overview.png)

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Регистрация нового устройства в центре Интернета вещей

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="run-a-sample-application-on-pi-web-simulator"></a>Запуск примера приложения на веб-симуляторе Pi

1. В области кодирования убедитесь, что вы работаете над стандартным примером приложения. Замените значения заполнителя в 15 строке на строку подключения устройства Центра Интернета вещей.
1. 
   ![Замена строки подключения устройства](media/iot-hub-raspberry-pi-web-simulator/1-connectionstring.png)

2. Выберите **выполнить** или введите, `npm start` чтобы запустить приложение.

Должны отобразиться следующие результаты, содержащие данные датчика и сообщения, которые отправляются в Центр Интернета вещей: ![Выходные данные — данные датчика, отправленные с Raspberry Pi в Центр Интернета вещей](media/iot-hub-raspberry-pi-web-simulator/2-run-application.png)

## <a name="read-the-messages-received-by-your-hub"></a>Чтение сообщений, полученных центром

Одним из способов мониторинга сообщений, полученных центром Интернета вещей от имитации устройства, является использование средств Azure IoT для Visual Studio Code. Дополнительные сведения см. в статье [использование средств Azure IOT для Visual Studio Code для отправки и получения сообщений между устройством и центром Интернета вещей](iot-hub-vscode-iot-toolkit-cloud-device-messaging.md).

Чтобы узнать больше о способах обработки данных, отправляемых устройством, перейдите к следующему разделу.

## <a name="next-steps"></a>Дальнейшие действия

Вы запустили пример приложения, чтобы собрать данные датчика и отправить их в Центр Интернета вещей.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
