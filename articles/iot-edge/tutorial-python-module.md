---
title: Руководство. Создание пользовательского модуля Python с помощью Azure IoT Edge
description: В этом руководстве показано, как создать модуль IoT Edge с кодом Python и его развертывание на пограничном устройстве.
services: iot-edge
author: kgremban
manager: philmea
ms.reviewer: kgremban
ms.author: kgremban
ms.date: 08/04/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
ms.openlocfilehash: d6a95dae91ef3e6aa7d39cf8af51c355a87ea73a
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103462734"
---
# <a name="tutorial-develop-and-deploy-a-python-iot-edge-module-using-linux-containers"></a>Руководство. Разработка и развертывание модуля IoT Edge на Python с использованием контейнеров Linux

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

Используйте Visual Studio Code для разработки и развертывания кода Python на устройствах с Azure IoT Edge.

Вы можете использовать модули Azure IoT Edge для развертывания кода, который реализует бизнес-логику непосредственно на устройствах IoT Edge. В этом руководстве рассматривается создание и развертывание модуля IoT Edge, который фильтрует данные датчика на устройстве IoT Edge, настроенном с помощью краткого руководства. В этом руководстве описано следующее:

> [!div class="checklist"]
>
> * создание модуля Python для IoT Edge с помощью Visual Studio Code;
> * Использовать Visual Studio Code и Docker для создания образа Docker и его публикации в реестре.
> * Развертывать модуль на устройстве IoT Edge.
> * Просматривать сформированные данные.

Модуль IoT Edge, создаваемый в этом руководстве, фильтрует данные температуры, которые создаются устройством. Оно отправляет сообщения, только если температура превышает заданное пороговое значение. Такой тип пограничного анализа удобен для сокращения объема данных, передаваемых в облако и сохраняемых в нем.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

В этом руководстве показано, как разработать модуль на **Python** с помощью **Visual Studio Code** и развернуть его на устройстве IoT Edge.

IoT Edge не поддерживает модули Python, использующие контейнеры Windows.

В приведенной ниже таблице приведены возможные варианты для разработки и развертывания модулей Python с использованием контейнеров Linux:

| Python | Visual Studio Code | Visual Studio 2017 или 2019 |
| - | ------------------ | ------------------ |
| **Linux AMD64** | ![Использование VS Code для модулей Python в Linux AMD64](./media/tutorial-c-module/green-check.png) |  |
| **Linux ARM32** | ![Использование VS Code для модулей Python в Linux ARM32](./media/tutorial-c-module/green-check.png) |  |

Прежде, чем начинать работу с этим руководством, вам нужно пройти предыдущее руководство, где описано, как настроить среду для разработки контейнеров Linux: [Краткое руководство. Разработка модулей IoT Edge с использованием контейнеров Linux](tutorial-develop-for-linux.md). После работы с ним у вас должны быть готовы все необходимые компоненты:

* [Центр Интернета вещей](../iot-hub/iot-hub-create-through-portal.md) ценовой категории "Бесплатный" или "Стандартный" в Azure.
* Устройство с Azure IoT Edge. Вы можете воспользоваться краткими руководствами для настройки устройства [Linux](quickstart-linux.md) или [Windows](quickstart.md).
* реестр контейнеров, например [Реестр контейнеров Azure](../container-registry/index.yml);
* средство [Visual Studio Code](https://code.visualstudio.com/), настроенное с помощью [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools);
* выпуск [Docker CE](https://docs.docker.com/install/), настроенный для выполнения контейнеров Linux.

Для разработки модуля IoT Edge на языке Python установите на компьютере разработки следующие дополнительные компоненты:

* [Расширение Visual Studio Code для Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python).
* [Python](https://www.python.org/downloads/).
* [PIP](https://pip.pypa.io/en/stable/installing/#installation) для установки пакетов Python (обычно входит в состав установки Python).

>[!Note]
>Убедитесь, что папка `bin` расположена в соответствии с требованиями вашей платформы. Обычно это `~/.local/` для UNIX и macOS или `%APPDATA%\Python` для Windows.

## <a name="create-a-module-project"></a>Создание проекта модуля

Описанный ниже процесс позволяет создать модуль Python для IoT Edge с использованием Visual Studio Code и средств Azure IoT.

### <a name="create-a-new-project"></a>Создание нового проекта

Создайте шаблон решения Python, в который вы сможете поместить собственный код.

1. В Visual Studio Code выберите **Представление** > **Палитра команд** для открытия палитры команд VS Code.

2. В палитре команд введите и выполните команду **Azure: Sign in** (Azure: войти). Следуйте инструкциям, чтобы войти в свою учетную запись Azure. Если вход был выполнен, то этот шаг можно пропустить.

3. В палитре команд введите и выполните команду **Azure IoT Edge: New IoT Edge Solution** (Azure IoT Edge: создать решение IoT Edge). Выполняя инструкции, укажите следующие сведения для создания решения:

   | Поле | Значение |
   | ----- | ----- |
   | Выбор папки | Выберите расположение на компьютере разработчика для VS Code, чтобы создать файлы решения. |
   | Введите название решения. | Введите описательное имя решения или примите имя по умолчанию **EdgeSolution**. |
   | Выбор шаблона модуля | Выберите **Модуль Python**. |
   | Указание имени модуля | Назовите модуль **PythonModule**. |
   | Указание репозитория изображений Docker для модуля | Репозиторий изображений включает в себя имя реестра контейнеров и имя образа контейнера. Образ контейнера предварительно заполняется именем, которое вы указали на последнем шаге. Замените **localhost:5000** значением **сервера входа** из реестра контейнеров Azure. Вы можете узнать сервер входа на странице "Обзор" реестра контейнеров на портале Azure. <br><br>Окончательный вариант репозитория образов выглядит так: \<registry name\>.azurecr.io/pythonmodule. |

   ![Выбор репозитория образа Docker](./media/tutorial-python-module/repository.png)

### <a name="add-your-registry-credentials"></a>Добавление учетных данных реестра

Файл среды хранит учетные данные для репозитория контейнеров и совместно использует их со средой выполнения IoT Edge. Среде выполнения нужны эти учетные данные, чтобы извлечь частный образ на устройство IoT Edge.

Расширение IoT Edge пытается извлечь учетные данные реестра контейнеров из Azure и заполнить их в файле среды. Проверьте, включены ли ваши учетные данные. В противном случае добавьте их:

1. Откройте в обозревателе VS Code файл с расширением **ENV**.
2. Обновите поля с **именем пользователя** и **паролем**, скопированные из своего реестра контейнера Azure.
3. Сохраните ENV-файл.

### <a name="select-your-target-architecture"></a>Выбор целевой архитектуры

Сейчас Visual Studio Code позволяет разрабатывать модули Python для устройств Linux AMD64 и Linux ARM32v7. Для каждого решения вам нужно выбрать одну целевую платформу, так как сборка и запуск контейнера для разных архитектур различается. По умолчанию используется Linux AMD64.

1. Откройте палитру команд и выполните поиск **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Azure IoT Edge: установить целевую платформу по умолчанию для решения Edge) или выберите значок ярлыка на боковой панели в нижней части окна.

2. В палитре команд выберите целевую архитектуру из списка параметров. В рамках этого руководства мы используем в качестве устройства IoT Edge виртуальную машину Ubuntu. Поэтому сохраним значение по умолчанию **amd64**.

### <a name="update-the-module-with-custom-code"></a>Обновление модуля с помощью пользовательского кода

Каждый шаблон содержит пример кода, который принимает имитируемые данные с датчиков модуля **SimulatedTemperatureSensor** и направляет их в Центр Интернета вещей. В этом разделе добавьте код, который расширяет **PythonModule**, для анализа сообщений перед их отправкой.

1. В обозревателе VS Code откройте **modules** > **PythonModule** > **main.py**.

2. В верхней части файла **main.py** импортируйте библиотеку **json**:

    ```python
    import json
    ```

3. Добавьте переменные **TEMPERATURE_THRESHOLD** и **TWIN_CALLBACKS** в разделе глобальных счетчиков. Порог температуры задает значение измеренной температуры компьютера, при превышении которого данные отправляются в Центр Интернета вещей.

    ```python
    # global counters
    TEMPERATURE_THRESHOLD = 25
    TWIN_CALLBACKS = 0
    RECEIVED_MESSAGES = 0
    ```

4. Замените функцию **input1_listener** следующим кодом:

    ```python
        # Define behavior for receiving an input message on input1
        # Because this is a filter module, we forward this message to the "output1" queue.
        async def input1_listener(module_client):
            global RECEIVED_MESSAGES
            global TEMPERATURE_THRESHOLD
            while True:
                try:
                    input_message = await module_client.receive_message_on_input("input1")  # blocking call
                    message = input_message.data
                    size = len(message)
                    message_text = message.decode('utf-8')
                    print ( "    Data: <<<%s>>> & Size=%d" % (message_text, size) )
                    custom_properties = input_message.custom_properties
                    print ( "    Properties: %s" % custom_properties )
                    RECEIVED_MESSAGES += 1
                    print ( "    Total messages received: %d" % RECEIVED_MESSAGES )
                    data = json.loads(message_text)
                    if "machine" in data and "temperature" in data["machine"] and data["machine"]["temperature"] > TEMPERATURE_THRESHOLD:
                        custom_properties["MessageType"] = "Alert"
                        print ( "Machine temperature %s exceeds threshold %s" % (data["machine"]["temperature"], TEMPERATURE_THRESHOLD))
                        await module_client.send_message_to_output(input_message, "output1")
                except Exception as ex:
                    print ( "Unexpected error in input1_listener: %s" % ex )

        # twin_patch_listener is invoked when the module twin's desired properties are updated.
        async def twin_patch_listener(module_client):
            global TWIN_CALLBACKS
            global TEMPERATURE_THRESHOLD
            while True:
                try:
                    data = await module_client.receive_twin_desired_properties_patch()  # blocking call
                    print( "The data in the desired properties patch was: %s" % data)
                    if "TemperatureThreshold" in data:
                        TEMPERATURE_THRESHOLD = data["TemperatureThreshold"]
                    TWIN_CALLBACKS += 1
                    print ( "Total calls confirmed: %d\n" % TWIN_CALLBACKS )
                except Exception as ex:
                    print ( "Unexpected error in twin_patch_listener: %s" % ex )
    ```

5. Измените **listeners**, чтобы также прослушивать операции обновления двойников.

    ```python
        # Schedule task for C2D Listener
        listeners = asyncio.gather(input1_listener(module_client), twin_patch_listener(module_client))

        print ( "The sample is now waiting for messages. ")
    ```

6. Сохраните файл main.py.

7. В обозревателе VS Code откройте файл **deployment.template.json** в рабочей области решения IoT Edge.

8. Добавьте двойник модуля **PythonModule** в манифест развертывания. Вставьте следующее содержимое JSON в нижней части раздела **moduleContent** после двойника модуля **$edgeHub**:

   ```json
       "PythonModule": {
           "properties.desired":{
               "TemperatureThreshold":25
           }
       }
   ```

   ![Добавление двойника модуля в шаблон развертывания](./media/tutorial-python-module/module-twin.png)

9. Сохраните файл deployment.template.json.

## <a name="build-and-push-your-module"></a>Сборка и отправка модуля

В предыдущем разделе вы создали решение IoT Edge и добавили код в PythonModule, который будет отфильтровывать сообщения, где указанная температура компьютера находится в рамках допустимых ограничений. Теперь необходимо создать решение в качестве образа контейнера и передать его в реестр контейнеров.

1. Откройте интегрированный терминал VS Code, выбрав **Вид** > **Терминал**.

2. Войдите в Docker. Для этого введите представленную ниже команду в окне терминала. Выполните вход в систему, используя имя пользователя, пароль и сервер входа из реестра контейнеров Azure. Вы можете получить эти значения в разделе **Ключи доступа** в разделе реестра на портале Azure.

   ```bash
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Возможно, появится предупреждение системы безопасности, в котором рекомендуется использовать `--password-stdin`. Для рабочих сценариев это лучшая методика, но мы не будем рассматривать ее в этом учебнике. Дополнительные сведения см. в описании команды [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin) в справочнике.

3. В обозревателе VS Code щелкните правой кнопкой мыши файл **deployment.template.json** и выберите **Build and Push IoT Edge Solution** (Создать и отправить решение IoT Edge).

   Эта команда сборки и отправки позволяет запустить три операции. Во-первых,в решении создается папка с именем **config**, которая содержит полный манифест развертывания на основе информации из шаблона развертывания и других файлов решения. Во-вторых, выполняется `docker build` для сборки образа контейнера на основе подходящего файла dockerfile для целевой архитектуры. В-третьих, выполняется `docker push` для отправки образа в реестр контейнеров.

   В первый раз выполнение этого процесса может занять несколько минут, но при следующем запуске команд он будет быстрее.

## <a name="deploy-modules-to-device"></a>Развертывание модулей на устройстве

Разверните проект модуля на устройстве IoT Edge с помощью обозревателя Visual Studio Code и расширения Azure IoT Tools. У вас уже есть манифест развертывания, подготовленный для вашего сценария (файл **deployment.amd64.json** в папке config). Теперь вам осталось выбрать устройство для получения развертывания.

Убедитесь, что устройство IoT Edge работает.

1. В обозревателе Visual Studio Code в разделе **Центр Интернета вещей Azure** разверните меню **Устройства**, чтобы отобразить список устройств Интернета вещей.

2. Щелкните имя устройства IoT Edge правой кнопкой мыши, а затем выберите **Create Deployment for Single Device** (Создание развертывания для одного устройства).

3. Выберите файл **deployment.amd64.json** в папке **config** и щелкните **Select Edge Deployment Manifest** (Выбрать манифест развертывания Edge). Не используйте файл deployment.template.json.

4. Разверните меню **Модули** для своего устройства, чтобы просмотреть список развернутых и запущенных модулей. Нажмите кнопку "Обновить". Появится новый запущенный модуль **PythonModule**, модуль **SimulatedTemperatureSensor**, а также **$edgeAgent** и **$edgeHub**.

    На запуск модулей может потребоваться несколько минут. Среда выполнения IoT Edge должна получить новый манифест развертывания, извлечь образы модулей из среды выполнения контейнеров, а затем запустить каждый новый модуль.

## <a name="view-the-generated-data"></a>Просмотр созданных данных

Когда вы примените манифест развертывания к устройству IoT Edge, в среде выполнения IoT Edge на устройстве начнется сбор сведений о новом развертывании и выполнение операций. Работа всех выполняющихся на устройстве модулей, которые не включены в манифест развертывания, будет остановлена. А модули, которых нет на устройстве, будут запущены.

Сведения о состоянии устройства IoT Edge можно просмотреть в разделе **Azure IoT Hub Devices** (Устройства Центра Интернета вещей Azure) в обозревателе Visual Studio Code. Разверните раздел со сведениями об устройстве, чтобы просмотреть список развернутых и запущенных модулей.

1. В обозревателе Visual Studio Code щелкните имя устройства IoT Edge правой кнопкой мыши и выберите **Start Monitoring Built-in Event Endpoint** (Начать мониторинг строенной конечной точки событий).

2. Просмотрите сообщения, поступающие в Центр Интернета вещей. Получение сообщений может занять некоторое время. Устройство IoT Edge должно получить новое развертывание и запустить все модули. После этого измененный нами код PythonModule ожидает, пока температура компьютера не достигнет 25 градусов, и лишь затем отправляет сообщения. С помощью кода также присваивается тип **Оповещение** всем сообщениям со сведениями о достижении порога температуры.

## <a name="edit-the-module-twin"></a>Изменение двойника модуля

Мы использовали двойник модуля PythonModule в манифесте развертывания, чтобы задать порог температуры на уровне 25 градусов. С помощью двойника модуля вы можете изменять функциональные возможности без необходимости обновлять код модуля.

1. В Visual Studio Code разверните окно подробных сведений об устройстве IoT Edge, чтобы просмотреть список выполняющихся модулей.

2. Щелкните **PythonModule** правой кнопкой мыши и выберите **Edit module twin** (Изменение двойника модуля).

3. В списке требуемых свойств найдите **TemperatureThreshold**. Измените его значение, увеличив температуру на 5 или 10 градусов относительно последней сообщаемой температуры.

4. Сохраните файл двойника модуля.

5. Щелкните правой кнопкой мыши в любой точки на панели редактирования двойника модуля и выберите **Update module twin** (Обновление двойника модуля).

6. Выполните мониторинг сообщений, поступающих с устройства в облако. Вы увидите, что поток сообщений прекратится, пока не будет достигнут новый порог температуры.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете перейти к следующей рекомендуемой статье, можно сохранить созданные и повторно используемые ресурсы и конфигурации. Это же устройство IoT Edge также можно использовать в качестве тестового устройства.

В противном случае вы можете удалить локальные конфигурации и ресурсы Azure, созданные в рамках этой статьи, чтобы избежать расходов.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы создали модуль IoT Edge, который содержит код для фильтрации необработанных данных, созданных вашим устройством IoT Edge.

Перейдите к следующим руководствам, чтобы узнать, как Azure IoT Edge поможет развернуть облачные службы Azure для обработки данных на пограничном устройстве.

> [!div class="nextstepaction"]
> [Функции](tutorial-deploy-function.md)
> [Stream Analytics](tutorial-deploy-stream-analytics.md)
> [Машинное обучение](tutorial-deploy-machine-learning.md)
> [Служба пользовательского визуального распознавания](tutorial-deploy-custom-vision.md)