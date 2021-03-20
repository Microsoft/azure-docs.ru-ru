---
title: Подключение Raspberry Pi к центру Интернета вещей Azure в облаке (Node.js)
description: Узнайте, как настроить и подключить Raspberry Pi к центру Интернета вещей Azure для Raspberry Pi, чтобы отправить данные на облачную платформу Azure в этом руководстве.
author: wesmc7777
manager: eliotgra
keywords: Raspberry Pi и Центр Интернета вещей Azure, Raspberry Pi и Центр Интернета вещей, отправка данных с Raspberry Pi в облако, подключение Raspberry Pi к облаку
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 03/13/2020
ms.author: wesmc
ms.custom:
- 'Role: Cloud Development'
- devx-track-js
ms.openlocfilehash: c96f674b64401250d45542d0f59f13654cf37caa
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97802530"
---
# <a name="connect-raspberry-pi-to-azure-iot-hub-nodejs"></a>Подключение Raspberry Pi к Центру Интернета вещей Azure (Node.js)

[!INCLUDE [iot-hub-get-started-device-selector](../../includes/iot-hub-get-started-device-selector.md)]

В этом учебнике описано, как начать работу с устройством Raspberry Pi под управлением Raspbian. Также вы узнаете, как можно легко подключать устройства к облаку с помощью [Центра Интернета вещей Azure](about-iot-hub.md). Примеры для Windows 10 IoT Базовая представлены в [Центре разработки для Windows](https://www.windowsondevices.com/).

Нет начального набора? Используйте [онлайн-симулятор Raspberry Pi](iot-hub-raspberry-pi-web-simulator-get-started.md). Или купите новый комплект [здесь](https://azure.microsoft.com/develop/iot/starter-kits).

## <a name="what-you-do"></a>Что нужно сделать

* Создайте Центр Интернета вещей.

* Зарегистрируем устройство для Pi в Центре Интернета вещей.

* Настроим Raspberry Pi.

* Мы запустим пример приложения на Pi для отправки данных в Центр Интернета вещей.

## <a name="what-you-learn"></a>Что вы узнаете

* Как создать Центр Интернета вещей Azure и получить строку подключения нового устройства.

* Как подключать Pi к датчику BME280.

* Как собирать данные датчика, запустив пример приложения на Pi.

* Как отправить данные датчика в Центр Интернета вещей.

## <a name="what-you-need"></a>Что нужно

![Что нужно](./media/iot-hub-raspberry-pi-kit-node-get-started/0-starter-kit.png)

* Плата Raspberry Pi 2 или Raspberry Pi 3.

* Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

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
> Если у вас нет дополнительных элементов, можно использовать имитацию датчиков.

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Регистрация нового устройства в центре Интернета вещей

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="set-up-raspberry-pi"></a>Настройка Raspberry Pi

### <a name="install-the-raspbian-operating-system-for-pi"></a>Установка операционной системы Raspbian на Pi

Подготовьте карту microSD для установки образа ОС Raspbian.

1. Скачайте ОС Raspbian.

   а. [Raspbian бустер с рабочим столом](https://www.raspberrypi.org/software/) (ZIP-файл).

   b. Извлеките образ ОС Raspbian в папку на компьютере.

2. Установите ОС Raspbian на карту microSD.

   а. [Скачайте и установите служебную программу для записи SD Card](https://etcher.io/).

   b. Запустите Etcher и выберите образ Raspbian, извлеченный на шаге 1.

   c. Выберите устройство для чтения карт microSD. В программе Etcher уже может быть выбрано правильное устройство для чтения.

   d. Щелкните Flash (Переключиться), чтобы установить ОС Raspbian на карту microSD.

   д) По завершении установки удалите карту microSD из компьютера. Удалять карту microSD напрямую безопасно, так как программа Etcher автоматически извлекает или отключает карту microSD после завершения.

   е) Вставьте карту microSD в устройство Pi.

### <a name="enable-ssh-and-i2c"></a>Включение SSH и I2C

1. Подключите Pi к монитору, клавиатуре и мыши.

2. Запустите PI, а затем войдите в Raspbian, используя в `pi` качестве имени пользователя и `raspberry` пароля.

3. Щелкните значок Raspberry > **настройки**  >  **Raspberry Pi**.

   ![Меню параметров Raspbian](./media/iot-hub-raspberry-pi-kit-node-get-started/1-raspbian-preferences-menu.png)

4. На вкладке **Interfaces** (Интерфейсы) установите для параметров **I2C** и **SSH** значение **Enable** (Включить), а затем нажмите кнопку **ОК**. Этот шаг является необязательным, если у вас нет физических датчиков и вы будете использовать симулированные данные датчика.

   ![Включение I2C и SSH на Raspberry Pi](./media/iot-hub-raspberry-pi-kit-node-get-started/2-enable-i2c-ssh-on-raspberry-pi.png)

> [!NOTE]
> Сведения о том, как включить SSH и I2C, можно найти в дополнительных справочных документах на [raspberrypi.org](https://www.raspberrypi.org/documentation/remote-access/ssh/) и [Adafruit.com](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-4-gpio-setup/configuring-i2c).

### <a name="connect-the-sensor-to-pi"></a>Подключение датчика к Pi

Подключите светодиодный индикатор и датчик BME280 к Pi с помощью монтажной платы и оптоволоконных кабелей, как показано ниже. Если у вас нет датчика, [пропустите этот раздел](#connect-pi-to-the-network).

![Подключение Raspberry Pi и датчика](./media/iot-hub-raspberry-pi-kit-node-get-started/3-raspberry-pi-sensor-connection.png)

Датчик BME280 может собирать данные о температуре и влажности. Светодиодный индикатор мигает, когда устройство отправляет сообщение в облако.

Чтобы подключить выводы датчика, используйте следующие кабели:

| Начало (датчик и светодиодный индикатор)     | Конец (плата)            | Цвет кабеля   |
| -----------------------  | ---------------------- | ------------: |
| VDD (вывод 5G)             | 3.3V PWR (вывод 1)       | Белый кабель   |
| GND (вывод 7G)             | GND (вывод 6)            | Коричневый кабель   |
| SDI (вывод 10G)            | I2C1 SDA (вывод 3)       | Красный кабель     |
| SCK (вывод 8G)             | I2C1 SCL (вывод 5)       | Оранжевый кабель  |
| LED VDD (вывод 18F)        | GPIO 24 (вывод 18)       | Белый кабель   |
| LED GND (вывод 17F)        | GND (вывод 20)           | Черный кабель   |

Щелкните для просмотра [Raspberry Pi 2 & 3 сопоставления ПИН-кода](/windows/iot-core/learn-about-hardware/pinmappings/pinmappingsrpi) для вашей ссылки.

После успешного подключения датчика BME280 к Raspberry Pi схема должна выглядеть так, как на изображении ниже.

![Подключенный компьютер Pi и датчик BME280](./media/iot-hub-raspberry-pi-kit-node-get-started/4-connected-pi.png)

### <a name="connect-pi-to-the-network"></a>Подключение устройства Pi к сети

Включите устройство Pi, используя кабель Micro USB и источник питания. Подключите Pi к проводной сети с помощью кабеля Ethernet или выполните [инструкции](https://www.raspberrypi.org/documentation/configuration/wireless/) от Raspberry Pi Foundation для подключения устройства Pi к беспроводной сети. После успешного подключения Pi к сети необходимо запомнить [IP-адрес устройства Pi](https://www.raspberrypi.org/documentation/remote-access/ip-address.md).

![Подключение к проводной сети](./media/iot-hub-raspberry-pi-kit-node-get-started/5-power-on-pi.png)

> [!NOTE]
> Убедитесь, что плата Pi подключена к той же сети, что и компьютер. Например, если компьютер подключен к беспроводной сети, а плата Pi подключена к проводной сети, то IP-адрес может не отобразиться в выходных данных devdisco.

## <a name="run-a-sample-application-on-pi"></a>Запуск примера приложения на Pi

### <a name="clone-sample-application-and-install-the-prerequisite-packages"></a>Клонирование примера приложения и установка пакетов необходимых компонентов

1. Используйте один из следующих SSH-клиентов для подключения к Raspberry Pi с главного компьютера:

   **Пользователи Windows**

   а. Скачайте и установите [PuTTY](https://www.putty.org/) для Windows.

   b. Скопируйте IP-адрес устройства Pi и вставьте его в поле для имени узла (или для IP-адреса), а затем выберите тип подключения SSH.

   ![PuTTy](./media/iot-hub-raspberry-pi-kit-node-get-started/7-putty-windows.png)

   **Пользователи MAC и Ubuntu**

   Используйте SSH-клиент, встроенный в Ubuntu или macOS. Возможно, для подключения устройства Pi по протоколу SSH потребуется выполнить `ssh pi@<ip address of pi>`.

   > [!NOTE]
   > Имя пользователя по умолчанию — `pi`, а пароль — `raspberry`.

2. Установите Node.js и NPM на устройстве Pi.

   Сначала проверьте версии Node.js.

   ```bash
   node -v
   ```

   Если версия меньше 10. x или если на устройстве PI нет Node.js, установите последнюю версию.

   ```bash
   curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash
   sudo apt-get -y install nodejs
   ```

3. Клонируйте пример приложения.

   ```bash
   git clone https://github.com/Azure-Samples/azure-iot-samples-node.git
   ```

4. Установите все пакеты для примера, в том числе пакет SDK для устройств Azure IoT, библиотеку датчика BME280 и библиотеку Wiring Pi.

   ```bash
   cd azure-iot-samples-node/iot-hub/Tutorials/RaspberryPiApp
   npm install
   ```

   > [!NOTE]
   >В зависимости от сетевого подключения процесс установки может занять несколько минут.

### <a name="configure-the-sample-application"></a>Настройка примера приложения

1. Откройте файл конфигурации, выполнив следующую команду:

   ```bash
   nano config.json
   ```

   ![Файл конфигурации](./media/iot-hub-raspberry-pi-kit-node-get-started/6-config-file.png)

   В этом файле можно настроить два элемента. Первый — `interval`. Он определяет время (в миллисекундах) между отправкой двух сообщений в облако. Второй — `simulatedData`. Он представляет логическое значение, определяющее, будут ли использоваться смоделированные данные датчика.

   Если у вас **нет датчика**, задайте для параметра `simulatedData` значение `true`, чтобы пример приложения создал и использовал смоделированные данные датчика.

   *Примечание. адрес I2C, используемый в этом руководстве, по умолчанию — 0x77. В зависимости от конфигурации это также может быть 0x76: Если возникла ошибка I2C, попробуйте изменить значение на 118 и убедитесь, что это работает лучше. Чтобы узнать, какой адрес используется датчиком, выполните команду `sudo i2cdetect -y 1` в оболочке на устройстве Raspberry Pi.*

2. Сохраните изменения и закройте окно, нажав клавиши CTRL+O > ВВОД > CTRL+X.

### <a name="run-the-sample-application"></a>Запуск примера приложения

Запустите пример приложения, выполнив следующую команду:

   ```bash
   sudo node index.js '<YOUR AZURE IOT HUB DEVICE CONNECTION STRING>'
   ```

   > [!NOTE]
   > Обязательно скопируйте и вставьте строку подключения устройства, заключив ее в одинарные кавычки.

Должны отобразиться следующие результаты, содержащие данные датчика и сообщения, которые отправляются в Центр Интернета вещей.

![Выходные данные — данные датчика, отправленные с Raspberry Pi в Центр Интернета вещей](./media/iot-hub-raspberry-pi-kit-node-get-started/8-run-output.png)

## <a name="read-the-messages-received-by-your-hub"></a>Чтение сообщений, полученных центром

Одним из способов мониторинга сообщений, полученных центром Интернета вещей с устройства, является использование средств Azure IoT для Visual Studio Code. Дополнительные сведения см. в статье [использование средств Azure IOT для Visual Studio Code для отправки и получения сообщений между устройством и центром Интернета вещей](iot-hub-vscode-iot-toolkit-cloud-device-messaging.md).

Чтобы узнать больше о способах обработки данных, отправляемых устройством, перейдите к следующему разделу.

## <a name="next-steps"></a>Дальнейшие действия

Вы запустили пример приложения для регистрации данных датчика и их отправки в центр Интернета вещей.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
