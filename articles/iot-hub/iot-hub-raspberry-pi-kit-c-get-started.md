---
title: Подключение Raspberry Pi к Центру Интернета вещей Azure с помощью С | Документация Майкрософт
description: Узнайте, как подключить компьютер Raspberry Pi к Центру Интернета вещей Azure и передавать данные на платформу облака Azure.
author: wesmc7777
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: conceptual
ms.date: 02/14/2019
ms.author: wesmc
ms.custom:
- 'Role: Cloud Development'
ms.openlocfilehash: 52da3c28faa375f8b308e3fe78329fec4f996af9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97804060"
---
# <a name="connect-raspberry-pi-to-azure-iot-hub-c"></a>Подключение Raspberry Pi к Центру Интернета вещей Azure (C)

[!INCLUDE [iot-hub-get-started-device-selector](../../includes/iot-hub-get-started-device-selector.md)]

В этом учебнике описано, как начать работу с устройством Raspberry Pi под управлением Raspbian. Также вы узнаете, как можно легко подключать устройства к облаку с помощью [Центра Интернета вещей Azure](about-iot-hub.md). Примеры для Windows 10 IoT Базовая представлены в [Центре разработки для Windows](https://www.windowsondevices.com/).

Нет начального набора? Используйте [онлайн-симулятор Raspberry Pi](iot-hub-raspberry-pi-web-simulator-get-started.md). Или купите новый комплект [здесь](https://azure.microsoft.com/develop/iot/starter-kits).

## <a name="what-you-do"></a>Что нужно сделать

* Создайте Центр Интернета вещей.

* Зарегистрируем устройство для Pi в Центре Интернета вещей.

* Настроим Raspberry Pi.

* Мы запустим пример приложения на Pi для отправки данных в Центр Интернета вещей.

Подключим Raspberry Pi к созданному Центру Интернета вещей. Затем запустим пример приложения на Pi, чтобы собрать данные о температуре и влажности, полученные с датчика BME280. После этого отправим данные с датчика в Центр Интернета вещей.

## <a name="what-you-learn"></a>Что вы узнаете

* Как создать Центр Интернета вещей Azure и получить строку подключения нового устройства.

* Как подключать Pi к датчику BME280.

* Как собирать данные датчика, запустив пример приложения на Pi.

* Как отправить данные датчика в Центр Интернета вещей.

## <a name="what-you-need"></a>Что нужно

![Что нужно](./media/iot-hub-raspberry-pi-kit-c-get-started/0-starter-kit.png)

* Плата Raspberry Pi 2 или Raspberry Pi 3.

* Активная подписка Azure. Если у вас нет учетной записи Azure, [Создайте бесплатную пробную учетную запись Azure](https://azure.microsoft.com/free/) всего за несколько минут.

* Монитор, USB-клавиатура и мышь, подключенные к Pi.

* ПК или компьютер Mac под управлением Windows или Linux.

* Подключение к Интернету.

* Карта microSD емкостью 16 ГБ или больше.

* Адаптер USB-SD или карта microSD для записи образа операционной системы на карту microSD.

* Источник питания 5 В 2 A с кабелем Micro USB длиной примерно 1,8 метра.

Ниже приведены необязательные компоненты.

* Датчик температуры, давления и влажности Adafruit BME280 в сборе.

* Монтажная плата.

* 6 оптоволоконных кабелей с разъемом на одном конце и гнездом на другом.

* Светодиодный индикатор 10 мм с рассеянным освещением.

> [!NOTE]
> Эти элементы являются необязательными, так как пример кода поддерживает имитацию данных датчика.
>

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Регистрация нового устройства в центре Интернета вещей

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="set-up-raspberry-pi"></a>Настройка Raspberry Pi

Теперь настройте Raspberry Pi.

### <a name="install-the-raspbian-operating-system-for-pi"></a>Установка операционной системы Raspbian на Pi

Подготовьте карту microSD для установки образа ОС Raspbian.

1. Скачайте ОС Raspbian.

   1. [Скачайте Raspbian Stretch с Desktop](https://www.raspberrypi.org/software/) (ZIP-файл).

   2. Извлеките образ ОС Raspbian в папку на компьютере.

2. Установите ОС Raspbian на карту microSD.

   1. [Скачайте и установите служебную программу для записи SD Card](https://etcher.io/).

   2. Запустите Etcher и выберите образ Raspbian, извлеченный на шаге 1.

   3. Выберите устройство для чтения карт microSD. Обратите внимание, что в программе Etcher уже может быть выбрано правильное устройство для чтения.

   4. Щелкните Flash (Переключиться), чтобы установить ОС Raspbian на карту microSD.

   5. По завершении установки удалите карту microSD из компьютера. Удалять карту microSD напрямую безопасно, так как программа Etcher автоматически извлекает или отключает карту microSD после завершения.

   6. Вставьте карту microSD в устройство Pi.

### <a name="enable-ssh-and-spi"></a>Включение SSH и SPI

1. Подключите PI к монитору, клавиатуре и мыши, запустите PI, а затем войдите в Raspbian, используя в `pi` качестве имени пользователя и `raspberry` пароля.
 
2. Щелкните значок Raspberry > **настройки**  >  **Raspberry Pi**.

   ![Меню параметров Raspbian](./media/iot-hub-raspberry-pi-kit-c-get-started/1-raspbian-preferences-menu.png)

3. На вкладке **Interfaces** (Интерфейсы) установите для параметров **SPI** и **SSH** значение **Enable** (Включить), а затем нажмите кнопку **ОК**. Этот шаг является необязательным, если у вас нет физических датчиков и вы будете использовать симулированные данные датчика.

   ![Включение SPI и SSH на Raspberry Pi](./media/iot-hub-raspberry-pi-kit-c-get-started/2-enable-spi-ssh-on-raspberry-pi.png)

> [!NOTE]
> Сведения о том, как включить SSH и SPI, можно найти в дополнительных справочных документах на [raspberrypi.org](https://www.raspberrypi.org/documentation/remote-access/ssh/) и [RASPI-CONFIG](https://www.raspberrypi.org/documentation/configuration/raspi-config.md).
>

### <a name="connect-the-sensor-to-pi"></a>Подключение датчика к Pi

Подключите светодиодный индикатор и датчик BME280 к Pi с помощью монтажной платы и оптоволоконных кабелей, как показано ниже. Если у вас нет датчика, [пропустите этот раздел](#connect-pi-to-the-network).

![Подключение Raspberry Pi и датчика](./media/iot-hub-raspberry-pi-kit-c-get-started/3-raspberry-pi-sensor-connection.png)

Датчик BME280 может собирать данные о температуре и влажности. Светодиодный индикатор мигает при обмене данными между устройством и облаком.

Чтобы подключить выводы датчика, используйте следующие кабели:

| Начало (датчик и светодиодный индикатор)     | Конец (плата)            | Цвет кабеля   |
| -----------------------  | ---------------------- | ------------: |
| LED VDD (вывод 5G)         | GPIO 4 (вывод 7)         | Белый кабель   |
| LED GND (вывод 6G)         | GND (вывод 6)            | Черный кабель   |
| VDD (вывод 18F)            | 3.3V PWR (вывод 17)      | Белый кабель   |
| GND (вывод 20F)            | GND (вывод 20)           | Черный кабель   |
| SCK (вывод 21F)            | SPI0 SCLK (вывод 23)     | Оранжевый кабель  |
| SDO (вывод 22F)            | SPI0 MISO (вывод 21)     | Желтый кабель  |
| SDI (вывод 23F)            | SPI0 MOSI (вывод 19)     | Зеленый кабель   |
| CS (вывод 24F)             | SPI0 CS (вывод 24)       | Синий кабель    |

Щелкните, чтобы просмотреть [схему соответствия выводов Raspberry Pi 2 и 3](/windows/iot-core/learn-about-hardware/pinmappings/pinmappingsrpi) для справки.

После успешного подключения датчика BME280 к Raspberry Pi схема должна выглядеть так, как на изображении ниже.

![Подключенный компьютер Pi и датчик BME280](./media/iot-hub-raspberry-pi-kit-c-get-started/4-connected-pi.png)

### <a name="connect-pi-to-the-network"></a>Подключение устройства Pi к сети

Включите устройство Pi, используя кабель Micro USB и источник питания. Подключите Pi к проводной сети с помощью кабеля Ethernet или выполните [инструкции](https://www.raspberrypi.org/documentation/configuration/wireless/) от Raspberry Pi Foundation для подключения устройства Pi к беспроводной сети. После успешного подключения Pi к сети необходимо запомнить [IP-адрес устройства Pi](https://www.raspberrypi.org/documentation/remote-access/ip-address.md).

![Подключение к проводной сети](./media/iot-hub-raspberry-pi-kit-c-get-started/5-power-on-pi.png)

## <a name="run-a-sample-application-on-pi"></a>Запуск примера приложения на Pi

### <a name="sign-into-your-raspberry-pi"></a>Вход в Raspberry Pi

1. Используйте один из следующих SSH-клиентов для подключения к Raspberry Pi с главного компьютера.
   
   **Пользователи Windows**
   1. Скачайте и установите [PuTTY](https://www.putty.org/) для Windows. 
   1. Скопируйте IP-адрес устройства Pi и вставьте его в поле для имени узла (или для IP-адреса), а затем выберите тип подключения SSH.
   
   ![PuTTy](./media/iot-hub-raspberry-pi-kit-node-get-started/7-putty-windows.png)

   **Пользователи MAC и Ubuntu**

   Используйте SSH-клиент, встроенный в Ubuntu или macOS. Возможно, для подключения устройства Pi по протоколу SSH потребуется выполнить `ssh pi@<ip address of pi>`.
   > [!NOTE]
   > Имя пользователя по умолчанию — `pi`, а пароль — `raspberry`.


### <a name="configure-the-sample-application"></a>Настройка примера приложения

1. Создайте клон примера приложения, выполнив следующую команду:

   ```bash
   sudo apt-get install git-core
   git clone https://github.com/Azure-Samples/iot-hub-c-raspberrypi-client-app.git
   ```

2. Запустите сценарий установки.

   ```bash
   cd ./iot-hub-c-raspberrypi-client-app
   sudo chmod u+x setup.sh
   sudo ./setup.sh
   ```

   > [!NOTE] 
   > Если у вас **нет физического устройства BME280**, можно использовать параметр командной строки --simulated-data для имитации данных температуры и влажности. `sudo ./setup.sh --simulated-data`
   >

### <a name="build-and-run-the-sample-application"></a>Создание и запуск примера приложения

1. Создайте пример приложения, выполнив следующую команду:

   ```bash
   cmake . && make
   ```
   
   ![Выходные данные при создании](./media/iot-hub-raspberry-pi-kit-c-get-started/7-build-output.png)

1. Запустите пример приложения, выполнив следующую команду:

   ```bash
   sudo ./app '<DEVICE CONNECTION STRING>'
   ```

   > [!NOTE] 
   > Обязательно скопируйте и вставьте строку подключения устройства, заключив ее в одинарные кавычки.
   >

Должны отобразиться следующие результаты, содержащие данные датчика и сообщения, которые отправляются в Центр Интернета вещей.

![Выходные данные — данные датчика, отправленные с Raspberry Pi в Центр Интернета вещей](./media/iot-hub-raspberry-pi-kit-c-get-started/8-run-output.png)

## <a name="read-the-messages-received-by-your-hub"></a>Чтение сообщений, полученных центром

Одним из способов мониторинга сообщений, полученных центром Интернета вещей с устройства, является использование средств Azure IoT для Visual Studio Code. Дополнительные сведения см. в статье [использование средств Azure IOT для Visual Studio Code для отправки и получения сообщений между устройством и центром Интернета вещей](iot-hub-vscode-iot-toolkit-cloud-device-messaging.md).

Чтобы узнать больше о способах обработки данных, отправляемых устройством, перейдите к следующему разделу.

## <a name="next-steps"></a>Дальнейшие действия

Вы запустили пример приложения, чтобы собрать данные датчика и отправить их в Центр Интернета вещей.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]