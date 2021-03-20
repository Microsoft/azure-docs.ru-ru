---
title: Получение сообщений из Twitter с помощью решения "Функции Azure" | Документация Майкрософт
description: Используйте датчик движения для отслеживания встряхиваний и извлекайте случайный твит с выбранным хэштегом с помощью службы "Функции Azure".
author: liydu
manager: jeffya
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 03/07/2018
ms.author: liydu
ms.custom: devx-track-csharp
ms.openlocfilehash: af1685f6455c0642800cba7dd604fcc836bcd7a4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92147892"
---
# <a name="shake-shake-for-a-tweet----retrieve-a-twitter-message-with-azure-functions"></a>Shake, Shake: получение сообщений из Twitter с помощью решения "Функции Azure"

В этом проекте вы научитесь использовать датчик движения для активации событий с помощью решения "Функции Azure" Это приложение извлекает случайный твит с определенным хэштегом, который вы настроите в эскизе Arduino. Затем оно выводит твит на экран DevKit.

## <a name="what-you-need"></a>Необходимые элементы

Выполните указания в [руководстве по началу работы](./iot-hub-arduino-iot-devkit-az3166-get-started.md), чтобы перейти к таким этапам:

* Подключение DevKit к сети Wi-Fi.
* Подготовка среды разработки.

Активная подписка Azure. Если у вас ее нет, зарегистрируйтесь одним из следующих способов:

* Вы можете активировать [бесплатную 30-дневную пробную учетную запись Microsoft Azure](https://azure.microsoft.com/free/).
* Заявка на [кредит Azure](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) , если вы являетесь подписчиком MSDN или Visual Studio

## <a name="open-the-project-folder"></a>Открытие папки проекта

Начните с открытия папки проекта. 

### <a name="start-vs-code"></a>Запуск VS Code

* Убедитесь, что плата DevKit подключена к компьютеру.

* Запустите VSCode.

* Подключите DevKit на компьютере.

   > [!NOTE]
   > При запуске VS Code может появиться сообщение об ошибке о том, что не обнаружена среда IDE Arduino или связанный пакет платы. В этом случае закройте VS Code и снова запустите среду IDE Arduino. Теперь VS Code правильно определит путь к ней.

### <a name="open-the-arduino-examples-folder"></a>Открытие папки с примерами Arduino

Разверните раздел **Arduino Examples** (Примеры Arduino) слева, перейдите в папку **Examples for MXCHIP AZ3166 (Примеры для MXCHIP AZ3166) > AzureIoT** и выберите **ShakeShake**. Откроется новое окно VS Code с папкой проекта. Если не видите раздел с MXCHIP AZ3166, проверьте правильность подключения устройства и перезапустите Visual Studio Code.  
![Мини-решение: примеры](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/vscode_examples.png)

Пример проекта также можно открыть из палитры команд. Нажмите `Ctrl+Shift+P` (macOS: `Cmd+Shift+P`) для вызова палитры команд. Введите **Arduino**, а затем найдите и выберите **Arduino: Examples** (Arduino: примеры).

## <a name="provision-azure-services"></a>Подготовка служб Azure

В окне решения запустите свою задачу с помощью клавиш `Ctrl+P` (macOS: `Cmd+P`), введя `task cloud-provision`.

В терминале VS Code с помощью указаний в интерактивной командной строке подготовьте необходимые службы Azure.

![На снимке экрана показана интерактивная Командная строка терминала Visual Studio Code.](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/cloud-provision.png)

> [!NOTE]
> Если страница зависает в состоянии загрузки на этапе входа в Azure, ознакомьтесь с часто задаваемыми вопросами по IoT DevKit, [раздел о зависании страницы входа](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#page-hangs-when-log-in-azure).
 
## <a name="modify-the-hashtag"></a>Изменение хэштега

Откройте `ShakeShake.ino` и найдите следующую строку кода:

```cpp
static const char* iot_event = "{\"topic\":\"iot\"}";
```

Замените строку `iot` в фигурных скобках любым интересующим вас хэштегом. DevKit будет извлекать случайные твиты, содержащие хэштег, указанный вами на этом шаге.

## <a name="deploy-azure-functions"></a>Развертывание решения "Функции Azure"

Чтобы запустить `task cloud-deploy` и начать развертывание кода решения "Функции Azure", используйте команду `Ctrl+P` (`Cmd+P` в macOS).

![На снимке экрана показаны Visual Studio Code, где можно запустить задачу Cloud-Deploy для развертывания кода функций Azure.](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/cloud-deploy.png)

> [!NOTE]
> Иногда функция Azure может выполняться неправильно. Чтобы устранить эту проблему при ее возникновении, ознакомьтесь с разделом об [ошибке компиляции в часто задаваемых вопросах по IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#compilation-error-for-azure-function).

## <a name="build-and-upload-the-device-code"></a>Создание и передача кода устройства

Долее создайте и передайте код устройства.

### <a name="windows"></a>Windows

1. Нажмите `Ctrl+P`, чтобы запустить задачу `task device-upload`.

2. Терминал предложит перейти в режим настройки. Для этого сделайте следующее:

   * Нажмите и удерживайте кнопку A.

   * Нажмите и отпустите кнопку сброса.

3. На экране появится идентификатор DevKit и надпись "Configuration" (Настройка).

### <a name="macos"></a>MacOS

1. Переведите DevKit в режим настройки:

   Удерживая нажатой кнопку "A", нажмите и отпустите кнопку "Reset" (Сброс). На экране отобразится надпись "Configuration" (Настройка).

2. С помощью `Cmd+P` запустите `task device-upload`, чтобы задать строку подключения, полученную на этапе `task cloud-provision`.

### <a name="verify-upload-and-run"></a>Проверка, отправка и запуск

Теперь строка подключения установлена. Она проверяет и загружает приложение, а затем запускает его. 

1. VS Code начнет проверку и отправку эскиза Arduino в DevKit.

   ![Снимок экрана показывает, Visual Studio Code проверит и отправку эскиза Arduino.](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/device-upload.png)

2. DevKit перезагрузится и начнет выполнение кода.

Может появиться сообщение об ошибке "Error: AZ3166: Unknown package" (Ошибка AZ3166: неизвестный пакет). Такая ошибка возникает, если индекс пакета платы неправильно обновлен. Действия по устранению проблемы см. в разделе об ошибке, связанной с [неизвестным пакетом, в часто задаваемых вопросах по IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#development).

## <a name="test-the-project"></a>Тестирование проекта

Когда инициализация приложения завершится, отпустите кнопку A и аккуратно встряхните плату DevKit. Произойдет получение случайного твита с хэштегом, который вы указали ранее. Через несколько секунд этот твит отобразится на экране DevKit:

### <a name="arduino-application-initializing"></a>Инициализация приложения Arduino...

![Инициализация приложения Arduino](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-1.png)

### <a name="press-a-to-shake"></a>Нажмите кнопку A, чтобы встряхнуть...

![Нажмите кнопку A, чтобы встряхнуть](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-2.png)

### <a name="ready-to-shake"></a>Готов к встряхиванию...

![Готов к встряхиванию](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-3.png)

### <a name="processing"></a>Обработка...

![Обработка](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-4.png)

### <a name="press-b-to-read"></a>Нажмите кнопку B, чтобы прочитать...

![Нажмите кнопку B, чтобы прочитать](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-5.png)

### <a name="display-a-random-tweet"></a>Отображение случайного твита...

![Отображение случайного твита](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-6.png)

- Снова нажмите кнопку A и повторно встряхните устройство, чтобы получить новый твит.
- С помощью кнопки B вы можете прокрутить твит до конца.

## <a name="how-it-works"></a>Принцип работы

![На схеме показано мобильное устройство, отправляющее событие в центр Azure I T, которое запускает приложение-функцию Azure для запроса твита, который отправляется обратно в приложение и пересылается в центр и на мобильное устройство.](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/diagram.png)

Эскиз Arduino отправляет событие в Центр Интернета вещей Azure. Это событие запускает приложение решения "Функции Azure". Приложение решения "Функции Azure" содержит логику для подключения к API Twitter и получения твитов. Полученный твит упаковывается в сообщение C2D (из облака на устройство) и отправляется на вызывающее устройство.

## <a name="optional-use-your-own-twitter-bearer-token"></a>Необязательный шаг: настройка собственного маркера носителя Twitter

Для тестирования в этом примере используется предварительно настроенный тестовый маркер носителя Twitter. Но для каждой учетной записи Twitter существует [ограничение скорости](https://dev.twitter.com/rest/reference/get/search/tweets). Если вы хотите применить собственный маркер, выполните следующие действия:

1. Откройте [портал Twitter для разработчиков](https://dev.twitter.com/), чтобы зарегистрировать новое приложение Twitter.

2. [Получите ключ и секреты объекта-получателя](https://support.yapsody.com/hc/en-us/articles/360003291573-How-do-I-get-a-Twitter-Consumer-Key-and-Consumer-Secret-key-) для этого приложения.

3. С помощью подходящей [служебной программы](https://gearside.com/nebula/utilities/twitter-bearer-token-generator/) создайте из этих двух ключей маркер носителя Twitter.

4. На [портале Azure](https://portal.azure.com/){:target="_blank"} откройте **группу ресурсов** и найдите функцию Azure (тип: служба приложений) для проекта "Shake, Shake". Это имя обязательно содержит строку "shake...".

   ![Снимок экрана портал Azure показывает службу приложений для проекта.](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/azure-function.png)

5. Измените код `run.csx` в разделе **Функции > shakeshake-cs**, включив в него собственный маркер:

   ```csharp
   string authHeader = "Bearer " + "[your own token]";
   ```
  
   ![На снимке экрана показан код C# для функции, в которой можно ввести маркер.](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/twitter-token.png)

6. Сохраните файл и щелкните **Запустить**.

## <a name="problems-and-feedback"></a>Проблемы и обратная связь

Как устранить проблемы или предоставить обратную связь. 

### <a name="problems"></a>Проблемы

Например, все этапы выполнены успешно, но на экране отображается "No Tweets" (Нет твитов). Это обычно происходит при первом запуске примера после развертывания, так как для "холодного" запуска приложения-функции нужно от нескольких секунд до целой минуты. 

Иногда при выполнении кода могут возникать сбои, приводящие к перезапуску приложения. В таких случаях приложению устройства потребуется некоторое время для получения очередного твита. Чтобы устранить такую проблему, примените любой из следующих методов или оба сразу:

1. Нажмите на DevKit кнопку сброса, чтобы перезапустить приложение устройства.

2. Найдите созданное приложение решения "Функции Azure" на [портале Azure](https://portal.azure.com/) и перезапустите его:

   ![На снимке экрана показан портал Azure с помощью приложения "функции Azure" и кнопки "перезапустить".](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/azure-function-restart.png)

Если вы столкнулись с другими проблемами, изучите [часто задаваемые вопросы по IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) или свяжитесь с нами по любому из доступных каналов:

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы знаете, как подключить устройство DevKit к акселератору решения Azure IoT для удаленного мониторинга и получить твит. Выполните следующие шаги в рамках дальнейшего обучения:

* [Общие сведения об акселераторе решения Azure IoT для удаленного мониторинга](/azure/iot-suite/)