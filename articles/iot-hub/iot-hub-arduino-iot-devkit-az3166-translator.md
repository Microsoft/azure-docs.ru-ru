---
title: Создание транслятора с функциями Azure, Cognitive Services, IoT DevKit
description: Использование микрофона в IoT DevKit для получения голосового сообщения и его преобразование в текст на английском языке при помощи Azure Cognitive Services
author: liydu
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 12/19/2018
ms.author: liydu
ms.custom: devx-track-csharp
ms.openlocfilehash: be26c6fe03dac9b9ff9dbff4a2bdce391ec0837e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96024169"
---
# <a name="use-iot-devkit-az3166-with-azure-functions-and-cognitive-services-to-make-a-language-translator"></a>Использование IoT DevKit AZ3166 с Функциями Azure и Cognitive Services для создания переводчика

Из этой статьи вы узнаете, как создать переводчик на основе IoT DevKit с использованием [Azure Cognitive Services](https://azure.microsoft.com/services/cognitive-services/). Это средство записывает голосовое сообщение, которое преобразуется в текст на английском языке, отображающийся на экране DevKit.

[MXChip IoT DevKit](https://aka.ms/iot-devkit) — это универсальная совместимая с Arduino плата со множеством периферийных устройств и датчиков. Решения для нее можно разрабатывать с помощью [Azure IoT Device Workbench](https://aka.ms/iot-workbench) или пакета расширений [Инструменты Azure IoT](https://aka.ms/azure-iot-tools) в Visual Studio Code. [Каталог проектов](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/) содержит образцы приложений, которые помогут создать прототип решения для Интернета вещей.

## <a name="before-you-begin"></a>Перед началом

Для работы с этим руководством сначала следует выполнить следующие задачи:

* Подготовьте DevKit по инструкциям из статьи [Подключение платы IoT DevKit AZ3166 к Центру Интернета вещей Azure в облаке](./iot-hub-arduino-iot-devkit-az3166-get-started.md).

## <a name="create-azure-cognitive-service"></a>Создание службы Azure Cognitive Services

1. В портал Azure щелкните **создать ресурс** и выполните поиск по слову **речь**. Заполните форму для создания службы распознавания речи.
  ![Служба распознавания речи](media/iot-hub-arduino-iot-devkit-az3166-translator/speech-service.png)

1. Перейдите к созданной службе речи, щелкните раздел **Ключи**, чтобы скопировать и записать значение параметра **Key1** для устройства DevKit, получающего к ней доступ.
  ![Копирование ключей](media/iot-hub-arduino-iot-devkit-az3166-translator/copy-keys.png)

## <a name="open-sample-project"></a>Открытие примера проекта

1. Убедитесь, что плата IoT DevKit **не подключена** к компьютеру. Сначала запустите VS Code, а затем подключите плату DevKit к компьютеру.

1. Щелкните `F1`, чтобы открыть палитру команд, затем введите и выберите **Azure IoT Device Workbench. Открыть примеры...** Затем выберите **IoT DevKit** в качестве платы.

1. На странице примеров IoT Workbench найдите **DevKit Translator** и щелкните **Open Sample (открыть образец**). Затем выберите путь по умолчанию, чтобы загрузить пример кода.
  ![Открытие примера](media/iot-hub-arduino-iot-devkit-az3166-translator/open-sample.png)

## <a name="use-speech-service-with-azure-functions"></a>Использование службы распознавания речи с функциями Azure

1. В VS Code щелкните `F1` , введите и выберите **Azure IOT Device Workbench: подготавливать службы Azure.** ![ .. Предоставление служб Azure](media/iot-hub-arduino-iot-devkit-az3166-translator/provision.png)

1. Выполните действия, чтобы завершить подготовку к работе Центра Интернета вещей Azure и функций Azure.
   ![Процедура подготовки](media/iot-hub-arduino-iot-devkit-az3166-translator/provision-steps.png)

   Запишите имя созданного устройства Центра Интернета вещей Azure.

1. Откройте `Functions\DevKitTranslatorFunction.cs` и обновите следующие строки кода, указав имя устройства и ключ службы речи, которые вы заговорили.
   ```csharp
   // Subscription Key of Speech Service
   const string speechSubscriptionKey = "";

   // Region of the speech service, see https://docs.microsoft.com/azure/cognitive-services/speech-service/regions for more details.
   const string speechServiceRegion = "";

   // Device ID
   const string deviceName = "";
   ```

1. Щелкните `F1` , введите и выберите **Azure IOT Device Workbench: развертывание в Azure.**.. Если VS Code запрашивает подтверждение повторного развертывания, нажмите кнопку **Да**.
   ![Предупреждение о развертывании](media/iot-hub-arduino-iot-devkit-az3166-translator/deploy-warning.png)

1. Убедитесь, что развертывание выполнено успешно.
   ![Успешное развертывание](media/iot-hub-arduino-iot-devkit-az3166-translator/deploy-success.png)

1. На портале Azure перейдите в раздел **Приложения-функции** и найдите только что созданное приложение-функцию Azure. Щелкните `devkit_translator`, затем **</> Get Function URL** (Получить URL-адрес функции), чтобы скопировать URL-адрес.
   ![Копирование URL-адреса функции](media/iot-hub-arduino-iot-devkit-az3166-translator/get-function-url.png)

1. Вставьте URL-адрес в файл `azure_config.h`.
   ![Azure config](media/iot-hub-arduino-iot-devkit-az3166-translator/azure-config.png)

   > [!NOTE]
   > Если приложение-функция работает неправильно, просмотрите раздел [часто задаваемых вопросов](https://microsoft.github.io/azure-iot-developer-kit/docs/faq#compilation-error-for-azure-function), чтобы устранить проблему.

## <a name="build-and-upload-device-code"></a>Сборка и отправка кода устройства

1. Переведите DevKit в **режим настройки** следующим образом.
   * Нажмите и удерживайте кнопку **A**.
   * Нажмите и отпустите кнопку **сброса**.

   На экране появится идентификатор DevKit и надпись **Configuration** (Настройка).

   ![Режим настройки DevKit](media/iot-hub-arduino-iot-devkit-az3166-translator/devkit-configuration-mode.png)

1. Щелкните `F1` , введите и выберите **Azure IOT Device Workbench: Настройка параметров устройства... > конфигурация строка подключения устройства**. Выберите **строку подключения к устройству Центра Интернета вещей**, чтобы настроить ее для DevKit.
   ![Настройка строки подключения](media/iot-hub-arduino-iot-devkit-az3166-translator/configure-connection-string.png)

1. После успешного завершения вы увидите уведомление.
   ![Настройка строки подключения выполнена](media/iot-hub-arduino-iot-devkit-az3166-translator/configure-connection-string-success.png)

1. Снова щелкните `F1`, введите и выберите **Azure IoT Device Workbench. Отправка кода устройства**. Будет запущена компиляция, затем код будет отправлен на устройство DevKit.
   ![Отправка устройства](media/iot-hub-arduino-iot-devkit-az3166-translator/device-upload.png)

## <a name="test-the-project"></a>Тестирование проекта

После инициализации приложения следуйте инструкциям на экране DevKit. Исходный язык по умолчанию — китайский.

Чтобы выбрать другой язык для перевода, выполните следующие действия.

1. Нажмите кнопку A, чтобы войти в режим настройки.

2. Нажмите кнопку B, чтобы прокрутить поддерживаемые исходные языки.

3. Нажмите кнопку A, чтобы подтвердить выбор исходного языка.

4. Нажмите и удерживайте кнопку B во время диктовки, а затем отпустите кнопку B, чтобы начать перевод.

5. Переведенный текст на английском языке отобразится на экране.

![Прокрутка для выбора языка](media/iot-hub-arduino-iot-devkit-az3166-translator/select-language.jpg)

![Результат перевода](media/iot-hub-arduino-iot-devkit-az3166-translator/translation-result.jpg)

На экране результатов перевода можно выполнить следующие действия.

- Нажимайте кнопки A и B, чтобы прокрутить и выбрать исходный язык.

- Нажмите кнопку B для разговора. Чтобы отправить голосовое сообщение и получить текст перевода, отпустите кнопку B.

## <a name="how-it-works"></a>Принцип работы

![mini-solution-voice-to-tweet-diagram](media/iot-hub-arduino-iot-devkit-az3166-translator/diagram.png)

В IoT DevKit записывается ваш голос. Затем отправляется HTTP-запрос для активации решения "Функции Azure". В решении "Функции Azure" для перевода вызывается API переводчика речи Cognitive Services. Когда в решение "Функции Azure" поступает текст перевода, на устройство отправляется сообщение C2D. После этого перевод отображается на экране.

## <a name="problems-and-feedback"></a>Проблемы и обратная связь

Если вы столкнулись с проблемами, ознакомьтесь с [вопросами и ответами](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) об IoT DevKit или используйте один из следующих каналов связи:

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы узнали, как использовать IoT DevKit для перевода при помощи Функций Azure и Cognitive Services. В этой статье вы узнали, как:

> [!div class="checklist"]
> * использовать Visual Studio Code, чтобы автоматизировать подготовку облака;
> * настроить строку подключения для устройства Интернета вещей Azure;
> * Развертывание экземпляра Функции Azure
> * протестировать перевод голосовых сообщений.

Перейдите к следующему руководству:

> [!div class="nextstepaction"]
> [Подключение IoT DevKit AZ3166 к акселератору решения Azure IoT для удаленного мониторинга](./iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring.md)