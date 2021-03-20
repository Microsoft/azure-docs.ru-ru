---
title: Подключение платы IoT DevKit AZ3166 к Центру Интернета вещей Azure
description: В этом руководстве содержатся сведения о том, как настроить и подключить плату IoT DevKit AZ3166 к Центру Интернета вещей Azure и передавать с нее данные в облачную платформу Azure.
author: wesmc7777
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 06/25/2019
ms.author: wesmc
ms.custom:
- mqtt
- 'Role: Cloud Development'
- devx-track-azurecli
ms.openlocfilehash: 3e5ab1667ee0cda459785efa624bd7f4fc6818b8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97562962"
---
# <a name="connect-iot-devkit-az3166-to-azure-iot-hub"></a>Подключение платы IoT DevKit AZ3166 к Центру Интернета вещей Azure

[!INCLUDE [iot-hub-get-started-device-selector](../../includes/iot-hub-get-started-device-selector.md)]

Плату [MXChip IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/) можно использовать для разработки и прототипирования решений Интернета вещей, использующих службы Microsoft Azure. DevKit содержит совместимую с Arduino плату с множеством периферийных устройств и датчиков, пакет ПО для платы с открытым кодом и обширную [коллекцию примеров](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/).

## <a name="what-you-learn"></a>Что вы узнаете

* Как создать Центр Интернета вещей и зарегистрировать устройство для MXChip IoT DevKit.
* Как подключить DevKit Интернета вещей к Wi-Fi и настроить строку подключения для Центра Интернета вещей.
* Как отправлять данные телеметрии датчика DevKit в Центр Интернета вещей.
* Как подготовить среду разработки и разработать приложение для DevKit Интернета вещей.

У вас еще нет платы DevKit? Воспользуйтесь [симулятором платы DevKit](https://azure-samples.github.io/iot-devkit-web-simulator/) или [приобретите ее](https://aka.ms/iot-devkit-purchase).

Исходный код для всех руководств DevKit можно найти в [коллекции примеров кода](/samples/browse/?term=mxchip).

## <a name="what-you-need"></a>Необходимые элементы

- Плата MXChip DevKit Интернета вещей с кабелем micro USB. Вы можете [получить ее прямо сейчас](https://aka.ms/iot-devkit-purchase).
- Компьютер под управлением Windows 10, macOS 10.10 или Ubuntu 18.04 (или более поздней версии).
- Активная подписка Azure. Вы можете активировать [бесплатную 30-дневную пробную учетную запись Microsoft Azure](https://azureinfo.microsoft.com/us-freetrial.html).
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]
  
## <a name="prepare-your-hardware"></a>Подготовка оборудования

Подключите к компьютеру следующее оборудование.

* Плата DevKit.
* Кабель micro USB.

![Необходимое оборудование](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/hardware.jpg)

Чтобы подключить плату DevKit к компьютеру, сделайте следующее:

1. Подключите разъем USB кабеля к компьютеру.

2. Подключите разъем micro USB кабеля к плате DevKit.

3. Зеленый светодиодный индикатор питания будет означать, что подключение установлено.

   ![Подключение оборудования](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/connect.jpg)

## <a name="quickstart-send-telemetry-from-devkit-to-an-iot-hub"></a>Краткое руководство. Отправка данных телеметрии с устройства в Центр Интернета вещей

В кратком руководстве по отправке данных телеметрии в Центр Интернета вещей используется предварительно скомпилированное встроенное ПО DevKit. Прежде чем запускать его, создайте Центр Интернета вещей и зарегистрируйте устройство в этом Центре.

### <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

### <a name="register-a-device"></a>Регистрация устройства

Устройство должно быть зарегистрировано в Центре Интернета вещей, прежде чем оно сможет подключиться. В этом кратком руководстве для регистрации имитируемого устройства используется Azure Cloud Shell.

1. Выполните приведенные ниже команды в Azure Cloud Shell, чтобы создать удостоверение устройства.

   **YourIoTHubName**. Замените этот заполнитель именем вашего Центра Интернета вещей.

   **MyNodeDevice**. Имя регистрируемого устройства. Используйте **MyNodeDevice**, как показано ниже. Если вы выбрали другое имя для устройства, используйте его при работе с этим руководством и обновите имя устройства в примерах приложений перед их запуском.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name YourIoTHubName --device-id MyNodeDevice
    ```

   > [!NOTE]
   > Если возникнет ошибка при исполнении `device-identity`, установите [расширение Azure IoT для Azure CLI](https://github.com/Azure/azure-iot-cli-extension/blob/dev/README.md).
   > Выполните следующую команду, чтобы добавить расширение Интернета вещей Microsoft Azure для Azure CLI в экземпляр Cloud Shell. Расширение Интернета вещей добавляет в Azure CLI специальные команды Центра Интернета вещей, IoT Edge и службы подготовки устройств Интернета вещей (DPS).
   > 
   > ```azurecli-interactive
   > az extension add --name azure-iot
   >  ```
   >
  
1. Выполните следующую команду в Azure Cloud Shell, чтобы получить _строку подключения_ зарегистрированного устройства:

   **YourIoTHubName**. Замените этот заполнитель именем вашего Центра Интернета вещей.

    ```azurecli-interactive
    az iot hub device-identity connection-string show --hub-name YourIoTHubName --device-id MyNodeDevice --output table
    ```

    Запишите строку подключения устройства, которая выглядит так:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyNodeDevice;SharedAccessKey={YourSharedAccessKey}`

    Это значение понадобится позже в рамках этого краткого руководства.

### <a name="send-devkit-telemetry"></a>Отправка данных телеметрии DevKit

DevKit подключается к конечной точке конкретного устройства Центра Интернета вещей и отправляет телеметрию температуры и влажности.

1. Скачайте последнюю версию [микропрограммного обеспечения GetStarted](https://aka.ms/devkit/prod/getstarted/latest) для DevKit Интернета вещей.

1. Убедитесь, что DevKit Интернета вещей подключается к компьютеру через USB. Откройте проводник. Там имеется запоминающее устройство USB с именем **AZ3166**.

    ![Открытие проводника Windows](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/az3166-usb.png)

1. Перетащите встроенное ПО, только что загруженное на запоминающее устройство массового хранения, и оно запишется автоматически.

    ![Копирование встроенного ПО](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/copy-firmware.png)

1. На DevKit, удерживая нажатой **кнопку В**, нажмите и отпустите кнопку **Reset** (Сброс), а затем отпустите **кнопку В**. DevKit перейдет в режим AP. В качестве подтверждения на экране отобразятся идентификатор SSID платы DevKit и IP-адрес портала настройки.

    ![Кнопка Reset (Сброс), кнопка B и идентификатор SSID](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/wifi-ap.jpg)

    ![Задание режима точки беспроводного доступа](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/set-ap-mode.gif)

1. Используйте веб-браузер на другом устройстве с модулем Wi-Fi (компьютер или мобильный телефон), чтобы подключиться к плате DevKit Интернета вещей по идентификатору SSID, показанному выше. Если будет запрошен пароль, оставьте его пустым.

    ![Подключение SSID](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/connect-ssid.png)

1. Откройте **192.168.0.1** в браузере. Выберите Wi-Fi, к которому нужно подключить DevKit Интернета вещей, введите пароль Wi-Fi, а затем вставьте строку подключения устройства, отмеченную ранее. Нажмите кнопку "Сохранить".

    ![Пользовательский интерфейс конфигурации](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/configuration-ui.png)

    > [!NOTE]
    > DevKit Интернета вещей поддерживает только сеть с частотой 2,4 ГГц. Дополнительные сведения см. в разделе [Ошибка разрешения Homebrew в macOS](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#wi-fi-configuration).

1. Информация о Wi-Fi и строка подключения устройства будут сохранены в DevKit Интернета вещей при отображении страницы результатов.

    ![Результат конфигурации](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/configuration-ui-result.png)

    > [!NOTE]
    > После настройки Wi-Fi учетные данные для этого подключения сохранятся на устройстве, даже если оно будет отсоединено.

1. DevKit Интернета вещей перезагрузится через несколько секунд. На экране DevKit вы увидите IP-адрес для DevKit, за которым следуют данные телеметрии, включая значение температуры и влажности, вместе с числом сообщений, отправленных в Центр Интернета вещей Azure.

    ![IP-адрес WiFi](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/wifi-ip.jpg)

    ![Отправка данных](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/sending-data.jpg)

1. Чтобы проверить данные телеметрии, отправляемые в Azure, выполните следующую команду в Azure Cloud Shell:

    ```azurecli
    az iot hub monitor-events --hub-name YourIoTHubName --output table
    ```

## <a name="prepare-the-development-environment"></a>Подготовка среды разработки

Выполните инструкции ниже, чтобы подготовить среду разработки для DevKit.

### <a name="install-visual-studio-code-with-azure-iot-tools-extension-package"></a>Установите Visual Studio Code с расширением Azure IoT Tools.

1. Установите [интегрированную среду разработки Arduino](https://www.arduino.cc/en/Main/Software). Она предоставляет необходимую цепочку инструментов для компиляции и передачи кода Arduino.
    * **Windows**: используйте версию установщика Windows. Не выполняйте установку из магазина приложений.
    * **macOS**: перетащите извлеченное приложение **Arduino.app** в папку `/Applications`.
    * **Ubuntu**: распакуйте его в папку, например `$HOME/Downloads/arduino-1.8.8`.

2. Установите кроссплатформенный редактор исходного кода [Visual Studio Code](https://code.visualstudio.com/) с эффективными средствами Intellisense, поддержкой отладки и завершением кода. С коммерческой платформы также можно установить широкий набор расширений.

3. Запустите VS Code, найдите **Arduino** в расширениях Marketplace и установите его. Это расширение предоставляет улучшенные возможности для разработки на платформе Arduino.

    ![Установите Arduino](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/install-arduino.png)

4. Найдите [Средства Azure IoT](https://aka.ms/azure-iot-tools) в расширениях Marketplace и установите его.

    ![Снимок экрана, показывающий средства Azure IoT в расширении Marketplace.](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/install-azure-iot-tools.png)

    Или скопируйте и вставьте этот URL-адрес в окно браузера: `vscode:extension/vsciot-vscode.azure-iot-tools`

    > [!NOTE]
    > Пакет расширений средств Azure IoT Tools содержит [Azure IoT Device Workbench](https://aka.ms/iot-workbench), который используется для разработки и отладки на различных устройствах DevKit Интернета вещей. [Расширение Центра Интернета вещей Azure](https://aka.ms/iot-toolkit), также включенное в пакет расширений Azure IoT Tools, используется для управления центрами Интернета вещей Azure и взаимодействия с ними.

5. Настройте VS Code с параметрами Arduino.

    В Visual Studio Code щелкните **Файл > Настройки > Параметры** (в macOS **Код > Настройки > Параметры**). Затем щелкните значок **Открыть параметры (JSON)** в правом верхнем углу страницы *Параметры*.

    ![Установка средств Azure IoT](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/user-settings-arduino.png)

    Добавьте следующие строки для настройки Arduino в зависимости от используемой платформы: 

    * **Windows**:

        ```json
        "arduino.path": "C:\\Program Files (x86)\\Arduino",
        "arduino.additionalUrls": "https://raw.githubusercontent.com/VSChina/azureiotdevkit_tools/master/package_azureboard_index.json"
        ```

    * **macOS**:

        ```json
        "arduino.path": "/Applications",
        "arduino.additionalUrls": "https://raw.githubusercontent.com/VSChina/azureiotdevkit_tools/master/package_azureboard_index.json"
        ```

    * **Ubuntu**:

        замените заполнитель **{username}** , приведенный ниже, своим именем пользователя.

        ```json
        "arduino.path": "/home/{username}/Downloads/arduino-1.8.8",
        "arduino.additionalUrls": "https://raw.githubusercontent.com/VSChina/azureiotdevkit_tools/master/package_azureboard_index.json"
        ```

6. Щелкните `F1`, чтобы открыть палитру команд, а затем введите и выберите **Arduino: Board Manager** (Диспетчер плат Arduino). Выполните поиск **AZ3166** и установите последнюю версию.

    ![Установка пакета SDK для DevKit](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/install-az3166-sdk.png)

### <a name="install-st-link-drivers"></a>Установка драйверов ST-Link

[ST-Link/V2](https://www.st.com/en/development-tools/st-link-v2.html) — это интерфейс USB, который IoT DevKit использует для взаимодействия с вашим компьютером разработки. Его необходимо установить в Windows, чтобы записать скомпилированный код устройства в DevKit. Выполните действия для определенной ОС, чтобы разрешить компьютеру доступ к устройству.

* **Windows**: скачайте и установите USB-драйвер с [веб-сайта STMicroelectronics](https://www.st.com/en/development-tools/stsw-link009.html).
* **macOS**: для macOS не требуется драйвер.
* **Ubuntu**: Запустите команды в терминале, затем выйдите и войдите, чтобы изменение группы вступило в силу.

    ```bash
    # Copy the default rules. This grants permission to the group 'plugdev'
    sudo cp ~/.arduino15/packages/AZ3166/tools/openocd/0.10.0/linux/contrib/60-openocd.rules /etc/udev/rules.d/
    sudo udevadm control --reload-rules

    # Add yourself to the group 'plugdev'
    # Logout and log back in for the group to take effect
    sudo usermod -a -G plugdev $(whoami)
    ```

Теперь подготовка и настройка среды разработки завершены. Давайте создадим только что запущенный пример GetStarted.

## <a name="build-your-first-project"></a>Создание первого проекта

### <a name="open-sample-code-from-sample-gallery"></a>Открытие примера кода из коллекции примеров

DevKit Интернета вещей содержит обширную коллекцию примеров, которые можно использовать для изучения подключения DevKit к различным службам Azure.

1. Убедитесь, что плата IoT DevKit **не подключена** к компьютеру. Сначала запустите VS Code, а затем подключите плату DevKit к компьютеру.

1. Щелкните `F1`, чтобы открыть палитру команд, затем введите и выберите **Azure IoT Device Workbench. Открыть примеры...** Затем выберите **IoT DevKit** в качестве платы.

1. На странице с примерами IoT Workbench найдите **Get Started** (Приступить к работе) и щелкните **Open Sample** (Открыть пример). Затем выберите путь по умолчанию, чтобы загрузить пример кода.

    ![Открытие примера](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/open-sample.png)

### <a name="provision-azure-iot-hub-and-device"></a>Инициализация Центра Интернета вещей Azure и устройства

Вместо подготовки Центра Интернета вещей Azure и устройства с портала Azure, это можно сделать в VS Code, не покидая среду разработки.

1. В открывшемся окне проекта щелкните `F1`, чтобы открыть палитру команд, введите и выберите **Azure IoT Device Workbench. Подготовка служб Azure...** . Выполните пошаговые инструкции для завершения подготовки Центра Интернета вещей Azure и создания устройства Центра Интернета вещей.

    ![Команда подготовки](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/provision.png)

    > [!NOTE]
    > Если вы не вошли в Azure. Используйте всплывающее уведомление для входа.

1. Выберите подписку, которую нужно использовать.

    ![Выбор подписки](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/select-subscription.png)

1. Выберите или создайте [группу ресурсов](../azure-resource-manager/management/overview.md#terminology).

    ![Выбор группы ресурсов](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/select-resource-group.png)

1. В указанной группе ресурсов следуйте указаниям, чтобы выбрать или создать Центр Интернета вещей Azure.

    ![Шаги по выбору Центра Интернета вещей](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/iot-hub-provision.png)

    ![Выбор Центра Интернета вещей](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/select-iot-hub.png)

    ![Выбранный Центр Интернета вещей](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/iot-hub-selected.png)

1. В окне вывода вы увидите подготовленный Центр Интернета вещей Azure.

    ![Центр Интернета вещей подготовлен](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/iot-hub-provisioned.png)

1. Выберите или создайте новое устройство в подготовленном Центре Интернета вещей Azure.

    ![Шаги по выбору устройства Интернета вещей](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/iot-device-provision.png)

    ![Выберите подготовленное устройство Интернета вещей](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/select-iot-device.png)

1. Теперь у вас есть подготовленный Центр Интернета вещей Azure и созданное в нем устройство. Кроме того, строка подключения устройства будет сохранена в VS Code для настройки DevKit Интернета вещей позже.

    ![Подготовка завершена](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/provision-done.png)

### <a name="configure-and-compile-device-code"></a>Настройка и компиляция кода устройства

1. В нижней правой строке состояния проверьте, выбрана ли плата **MXCHIP AZ3166** и используется ли последовательный порт с **STMicroelectronics**.

    ![Выбор платы и модели COM](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/select-com.png)

1. Щелкните `F1`, чтобы открыть палитру команд, затем введите и выберите **Azure IoT Device Workbench. Настройка параметров устройства...** , а затем последовательно выберите **Config Device Connection String (Настроить строку подключения к устройству) > Select IoT Hub Device Connection String (Выбрать строку подключения к устройству Центра Интернета вещей)** .

1. В DevKit удерживайте нажатой **кнопку A**, нажмите и отпустите кнопку **Reset** (Сброс), а затем отпустите **кнопку А**. DevKit перейдет в режим настройки и сохранит строку подключения.

    ![Строка подключения](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/connection-string.png)

1. Снова щелкните `F1`, введите и выберите **Azure IoT Device Workbench. Отправка кода устройства**. Будет запущена компиляция, затем код будет отправлен на устройство DevKit.

    ![Отправка Arduino](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/arduino-upload.png)

Плата DevKit перезагрузится и начнет выполнение кода.

> [!NOTE]
> Если есть ошибки или прерывания, всегда можно восстановить данные, выполнив команду еще раз.

## <a name="test-the-project"></a>Тестирование проекта

### <a name="view-the-telemetry-sent-to-azure-iot-hub"></a>Просмотр данных телеметрии, отправленных в Центр Интернета вещей Azure

Щелкните значок вилки в строке состояния, чтобы открыть Serial Monitor:

![Serial Monitor](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/serial-monitor.png)

Пример приложения успешно выполняется, если отображаются следующие результаты.

* Serial Monitor отображает сообщение, отправленное в Центр Интернета вещей.
* Светодиодный индикатор на плате MXChip IoT DevKit мигает.

![Выходные данные Serial Monitor](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/result-serial-output.png)

> [!NOTE]
> Во время тестирования может возникнуть ошибка, при которой светоиндикатор не мигает, портал Azure не отображает входящие данные с устройства, но на ОСИД-экране устройства отображается **Выполняется...** . Чтобы устранить эту проблему, в портале Azure перейдите к устройству в Центре Интернета вещей и отправьте сообщение на устройство. Если в VS Code Serial Monitor отображается следующий ответ, возможно, что прямое взаимодействие с устройством блокируется на уровне маршрутизатора. Проверьте правила брандмауэра и маршрутизатора, настроенные для подключения устройств. Кроме того, убедитесь, что исходящий порт 1833 открыт.
> 
> ОШИБКА: mqtt_client.c (ln 454): Ошибка: ошибка при открытии подключения к конечной точке  
> СВЕДЕНИЯ: >>>Состояние подключения: отключено  
> ОШИБКА: tlsio_mbedtls.c (ln 604): Не удалось открыть базовый ввод-вывод  
> ОШИБКА: mqtt_client.c (ln 1042): Ошибка: сбой io_open  
> ОШИБКА: iothubtransport_mqtt_common.c (ln 2283): сбой подключения к адресу atcsliothub.azure-devices.net.  
> СВЕДЕНИЯ: > > >Повторное подключение.  
> СВЕДЕНИЯ: Версия IoThub: 1.3.6  

### <a name="view-the-telemetry-received-by-azure-iot-hub"></a>Просмотр данных телеметрии, полученных Центром Интернета вещей Azure

Вы можете использовать [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) для мониторинга сообщений, отправленных из устройства в облако (D2C) в Центре Интернета вещей.

1. Войдите на [портал Azure](https://portal.azure.com/) и найдите созданный Центр Интернета вещей.

    ![Портал Azure](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/azure-iot-hub-portal.png)

1. В области **Политики общего доступа** выберите политику **iothubowner** и запишите строку подключения Центра Интернета вещей.

    ![Строка подключения Центра Интернета вещей](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/azure-portal-conn-string.png)

1. В VS Code щелкните `F1`, введите и выберите **Центр Интернета вещей Azure. Установка строки подключения Центра Интернета вещей**. Скопируйте строку подключения в это поле.

    ![Установка строки подключения Центра Интернета вещей](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/set-iothub-connection-string.png)

1. Разверните панель **AZURE IOT HUB DEVICES** (Устройства Центра Интернета вещей Azure) слева, щелкните имя созданного устройства правой кнопкой мыши и выберите **Start Monitoring Built-in Event Endpoint** (Запуск мониторинга встроенной конечной точки события).

    ![Мониторинг сообщений D2C](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/monitor-d2c.png)

1. На панели **выходных данных** вы можете просмотреть сообщения D2C, поступающие в Центр Интернета вещей.

    ![Снимок экрана, показывающий входящие сообщения D2C в центр Интернета вещей.](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/d2c-output.png)

## <a name="review-the-code"></a>Просмотр кода

`GetStarted.ino` является основным файлом эскиза Arduino.

![Сообщения D2C](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/code.png)

Чтобы увидеть, как телеметрические данные устройства отправляются в Центр Интернета вещей Azure, откройте файл `utility.cpp` в той же папке. Просмотрите [Справочник по API](https://microsoft.github.io/azure-iot-developer-kit/docs/apis/arduino-language-reference/), чтобы узнать, как использовать датчики и периферийные устройства в DevKit Интернета вещей.

`DevKitMQTTClient` используется в качестве оболочки **iothub_client** из [Пакетов SDK и библиотек Интернета вещей для C Microsoft Azure](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client) для взаимодействия с Центром Интернета вещей Azure.

## <a name="problems-and-feedback"></a>Проблемы и обратная связь

Если вы столкнулись с проблемами, вы можете найти решение в [часто задаваемых вопросах по IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) или обратиться к нам с помощью [Gitter](https://gitter.im/Microsoft/azure-iot-developer-kit). Вы также можете оставить нам свой отзыв на этой странице.

## <a name="next-steps"></a>Дальнейшие действия

Вы успешно подключили плату MXChip IoT DevKit к Центру Интернета вещей и отправили в него собранные данные датчика.

[!INCLUDE [iot-hub-get-started-az3166-next-steps](../../includes/iot-hub-get-started-az3166-next-steps.md)]
