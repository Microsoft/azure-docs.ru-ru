---
title: Подключение MXChip IoT DevKit к удаленному мониторингу Azure IoT
description: Из этого руководства вы узнаете, как с помощью IoT DevKit AZ3166 отправлять данные о состоянии датчиков в акселератор решения Azure IoT для удаленного мониторинга.
author: liydu
manager: jeffya
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 02/02/2018
ms.author: liydu
ms.openlocfilehash: 6912124ce8d1741731d625dccfea445417b2488d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98785022"
---
# <a name="connect-mxchip-iot-devkit-to-azure-iot-remote-monitoring-solution-accelerator"></a>Подключение MXChip IoT DevKit к акселератору решения Azure IoT для удаленного мониторинга

Из этого руководства вы узнаете, как запустить на DevKit пример приложения, которое отправляет данные с датчиков в акселератор решения Azure IoT для удаленного мониторинга.

[MXChip IoT DevKit](https://aka.ms/iot-devkit) — это универсальная совместимая с Arduino плата со множеством периферийных устройств и датчиков. Вы можете выполнить для нее разработку с помощью [расширения Visual Studio Code для Arduino](https://aka.ms/arduino). Она предоставляется с расширяющимся [каталогом проектом](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/), чтобы помочь вам выполнить прототипирование решений Интернета вещей, использующих службы Microsoft Azure.

## <a name="what-you-need"></a>Необходимые элементы

Выполните указания в [руководстве по началу работы](./iot-hub-arduino-iot-devkit-az3166-get-started.md), чтобы перейти к таким этапам:

* Подключение DevKit к сети Wi-Fi.
* Подготовка среды разработки

Активная подписка Azure. Если у вас еще нет ее, вы можете зарегистрироваться одним из двух способов:

* Вы можете активировать [бесплатную 30-дневную пробную учетную запись Microsoft Azure](https://azure.microsoft.com/free/).

* Запросите [деньги на счете в Azure](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/), если у вас есть подписка MSDN или Visual Studio.

## <a name="create-an-azure-iot-remote-monitoring-solution-accelerator"></a>Настройка акселератора решения Azure IoT для удаленного мониторинга

1. Перейдите на [сайт ускорителей решений для Интернета вещей Azure](https://www.azureiotsolutions.com/) и щелкните **Создать решение**.

   ![Выбор типа акселератора решений для Интернета вещей Azure](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-solution-types.png)

   > [!WARNING]
   > По умолчанию этот пример сначала создает акселератор решения Интернета вещей для удаленного мониторинга, а затем Центр Интернета вещей S2. Если этот Центр Интернета вещей предназначен для небольшого количества устройств, мы рекомендуем перейти с уровня S2 на S1 и удалить акселератор решения Интернета вещей для удаленного мониторинга, чтобы связанный с ним Центр Интернета вещей тоже можно было удалить, когда потребность в нем отпадет. 

2. Щелкните **Удаленный мониторинг**.

3. Введите имя решения, выберите подписку и регион, а затем щелкните **Создать решение**. Подготовка решения к работе может занять некоторое время.
  
   ![Создание решения](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-new-solution.png)

4. После завершения подготовки щелкните **Запустить**. Во время подготовки для решения создаются имитированные устройства. Щелкните вкладку **Devices** (Устройства), чтобы просмотреть их список.

   ![Панель мониторинга](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-new-solution-created.png)
  
   ![Консоль](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-console.png)

5. Щелкните **Add a device** (Добавить устройство).

6. Нажмите кнопку **Add New** (Добавить новое) в разделе **Пользовательское устройство**.
  
   ![Добавление нового устройства](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-add-new-device.png)

7. Установите переключатель **Позвольте мне определить собственный идентификатор устройства**, введите `AZ3166`, а затем нажмите кнопку **Создать**.
  
   ![Создание устройства с использованием идентификатора](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-new-device-configuration.png)

8. Запомните или запишите параметр **Имя узла Центра Интернета вещей** и нажмите кнопку **Готово**.

## <a name="open-the-remotemonitoring-sample"></a>Открытие примера RemoteMonitoring

1. Отключите DevKit на компьютере, если набор подключен.

2. Запустите VSCode.

3. Подключите DevKit на компьютере. VSCode автоматически обнаружит ваш DevKit и откроет следующие страницы:

   * Страница введения в набор разработки.
   * Примеры Arduino. Практические примеры по началу работы с набором разработки.

4. Разверните раздел **Arduino Examples** (Примеры Arduino) слева, перейдите в папку **Examples for MXCHIP AZ3166 (Примеры для MXCHIP AZ3166) > AzureIoT** и щелкните **RemoteMonitoring**. Откроется новое окно VSCode с папкой проекта.

   > [!NOTE]
   > Если вы закрыли эту область, вы можете открыть ее снова. Нажмите `Ctrl+Shift+P` (macOS: `Cmd+Shift+P`) для вызова палитры команд. Введите **Arduino**, а затем найдите и выберите **Arduino: примеры**.

## <a name="provision-required-azure-services"></a>Подготовка необходимых служб Azure

В окне решения запустите свою задачу: нажмите клавиши `Ctrl+P` (в macOS — `Cmd+P`) и введите `task cloud-provision` в текстовое поле.

В терминале VS Code с помощью указаний в интерактивной командной строке подготовьте необходимые службы Azure.

![Подготовка ресурсов Azure](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/provision.png)

## <a name="build-and-upload-the-device-code"></a>Создание и передача кода устройства

1. Нажмите клавиши `Ctrl+P` (в macOS — `Cmd + P`) и введите **task config-device-connection**.

2. В терминале отобразится запрос на использование строки подключения, полученной на шаге выполнения `task cloud-provision`. Вы можете также ввести в строке подключения устройства другую необходимую команду, щелкнув для этого "Create New..." ("Создать").

3. Терминал предложит перейти в режим настройки. Чтобы сделать это, удерживая нажатой кнопку "A", нажмите и отпустите кнопку "Reset" (Сброс). На экране появится идентификатор DevKit и надпись "Configuration" (Настройка).

   ![Ввод в строке подключения](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/config-device-connection.png)

4. После выполнения `task config-device-connection` щелкните `F1` для загрузки команд VS Code, а затем выберите `Arduino: Upload`. VS Code начнет проверку и отправку эскиза Arduino.
  
   ![Проверка и передача эскиза Arduino](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/arduino-upload.png)

Плата DevKit перезагрузится и начнет выполнение кода.

## <a name="test-the-project"></a>Тестирование проекта

При запуске примера приложения DevKit отправляет данные от датчиков по сети Wi-Fi в акселератор решения Azure IoT для удаленного мониторинга. Чтобы просмотреть результат, сделайте следующее:

1. Перейдите в акселератор решения Azure IoT для удаленного мониторинга и щелкните **Панель мониторинга**.

2. В консоли решения для удаленного мониторинга отображается состояние датчиков DevKit.

   ![Данные датчиков в акселераторе решения Azure IoT для удаленного мониторинга](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/sensor-status.png)

## <a name="change-device-id"></a>Изменение идентификатора устройства

Если необходимо изменить указанный в коде идентификатор **AZ3166** на пользовательский идентификатор устройства, измените строку кода, как показано в [примере удаленного мониторинга](/previous-versions/azure/iot-accelerators/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoring-v2).

## <a name="problems-and-feedback"></a>Проблемы и обратная связь

Если вы столкнулись с проблемами, ознакомьтесь с [вопросами и ответами](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) об IoT DevKit или используйте один из следующих каналов связи:

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы знаете, как подключить устройство DevKit к акселератору решения Azure IoT для удаленного мониторинга и визуализировать данные датчиков. Мы предлагаем вам следующие шаги для дальнейшего обучения:

* [Обзор акселераторов решений для Интернета вещей Azure](/azure/iot-suite/)

* [Пакет средств разработки Интернета вещей](https://microsoft.github.io/azure-iot-developer-kit/)