---
title: Руководство. Разработка модулей C# для Windows с помощью Azure IoT Edge
description: В этом учебнике показано, как создавать модули IoT Edge c использованием кода C# и развертывать их на устройстве с Windows под управлением IoT Edge.
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 08/03/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc, amqp, devx-track-csharp
ms.openlocfilehash: 4e01b1ca9a3858ff31ad9b5da1d1159209c44330
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103464073"
---
# <a name="tutorial-develop-c-iot-edge-modules-using-windows-containers"></a>Учебник. Разработка модулей IoT Edge на C# с использованием контейнеров Windows

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

В этой статье показано, как разрабатывать и развертывать код C# на устройствах Windows с Azure IoT Edge с помощью Visual Studio.

>[!NOTE]
>IoT Edge 1.1 LTS — это последний канал выпусков, который будет поддерживать контейнеры Windows. Начиная с версии 1.2, контейнеры Windows не поддерживаются. Мы рекомендуем вам использовать [IoT Edge для Linux в Windows](iot-edge-for-linux-on-windows.md) для выполнения IoT Edge на устройствах Windows.

Вы можете использовать модули Azure IoT Edge для развертывания кода, который реализует бизнес-логику непосредственно на устройствах IoT Edge. В этом руководстве рассматриваются создание и развертывание модуля IoT Edge, который фильтрует данные датчика.

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> * Создать модуль IoT Edge на основе пакета SDK C# с помощью Visual Studio.
> * Использовать Visual Studio и Docker для создания образа Docker и его публикации в реестре.
> * Развертывать модуль на устройстве IoT Edge.
> * Просматривать сформированные данные.

Модуль IoT Edge, создаваемый в этом руководстве, фильтрует данные температуры, которые создаются устройством. Он отправляет сообщения в потоке, только если температура превышает заданный порог. Такой тип пограничного анализа удобен для сокращения объема данных, передаваемых в облако и сохраняемых в нем.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

В этом учебнике показано, как разработать модуль на языке C# с помощью Visual Studio 2019 и развернуть его на устройстве Windows. Если вы разрабатываете модули с использованием контейнеров Linux, перейдите к статье [Разработка модулей IoT Edge на C# с использованием контейнеров Linux](tutorial-csharp-module.md).

В следующей таблице приведены возможные варианты для разработки и развертывания модулей C# с использованием контейнеров Windows:

| C# | Visual&nbsp;Studio&nbsp;Code | Visual Studio 2017&nbsp;и&nbsp;2019 |
| -- | :------------------: | :------------------: |
| Разработка Windows AMD64 | ![Разработка модулей C# для WinAMD64 в Visual Studio Code](./media/tutorial-c-module/green-check.png) | ![Разработка модулей C# для WinAMD64 в Visual Studio](./media/tutorial-c-module/green-check.png) |
| Отладка Windows AMD64 |   | ![Отладка модулей C# для WinAMD64 в Visual Studio](./media/tutorial-c-module/green-check.png) |

Прежде чем приступить к работе с этим учебником, настройте среду разработки, следуя инструкциям в статье [Разработка модулей IoT Edge с использованием контейнеров Windows](tutorial-develop-for-windows.md). После этого среда будет содержать следующие необходимые компоненты:

* [Центр Интернета вещей](../iot-hub/iot-hub-create-through-portal.md) ценовой категории "Бесплатный" или "Стандартный" в Azure.
* [устройство Windows, на котором выполняется Azure IoT Edge](how-to-install-iot-edge-windows-on-windows.md);
* реестр контейнеров, например [Реестр контейнеров Azure](../container-registry/index.yml);
* настроенная среда [Visual Studio 2019](/visualstudio/install/install-visual-studio) с расширением [Azure IoT Edge Tools](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16iotedgetools);
* приложение [Docker Desktop](https://docs.docker.com/docker-for-windows/install/), настроенное для запуска контейнеров Windows.

> [!TIP]
> Если вы используете Visual Studio 2017 (15.7 или более поздней версии), скачайте и установите Azure IoT Edge Tools для Visual Studio 2017 из [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vsiotedgetools).

## <a name="create-a-module-project"></a>Создание проекта модуля

В этом разделе описывается создание проекта модуля IoT Edge с использованием Visual Studio и расширения Azure IoT Edge Tools. Создав шаблон проекта, добавьте новый код, чтобы модуль фильтровал сообщения по их сообщаемым свойствам.

### <a name="create-a-new-project"></a>Создание проекта

Расширение Azure IoT Edge Tools предоставляет шаблоны проектов для всех поддерживаемых языков модулей IoT Edge в Visual Studio. Эти шаблоны содержат все файлы и код, необходимые для развертывания рабочего модуля для тестирования IoT Edge. Они также могут послужить отправной точкой путем их настройки с помощью собственной бизнес-логики.

1. Откройте Visual Studio 2019 и выберите **Создать проект**.

1. В области **Создание нового проекта** введите **IoT Edge** для поиска, а затем в списке результатов выберите проект **Azure IoT Edge (Windows AMD64)** .

   ![Снимок экрана: область "Создание нового проекта" для IoT Edge.](./media/tutorial-csharp-module-windows/new-project.png)

1. Выберите **Далее**.

    Откроется область **Настроить новый проект**.

   ![Снимок экрана: область "Настроить новый проект".](./media/tutorial-csharp-module-windows/configure-project.png)

1. В области **Настроить новый проект** переименуйте проект и решение, указав описательное имя, например **CSharpTutorialApp**. 

1. Выберите **Создать**, чтобы создать проект.

   Откроется область **Добавление модуля**.

   ![Снимок экрана: область "Добавление модуля" для настройки проекта.](./media/tutorial-csharp-module-windows/add-application-and-module.png)

1. На странице **Настроить новый проект** сделайте следующее:

   а. В области слева выберите шаблон **Модуль С#** .  
   b. В поле **Имя модуля** введите **CSharpModule**.  
   c. В поле **URL-адрес репозитория** замените **localhost:5000** на значение **сервера входа** из реестра контейнеров Azure в следующем формате: `<registry name>.azurecr.io/csharpmodule`.

    > [!NOTE]
    > Репозиторий изображений включает в себя имя реестра контейнеров и имя образа контейнера. Образ контейнера предварительно заполняется значением имени проекта модуля.  Вы можете узнать значение сервера входа на странице "Обзор" реестра контейнеров на портале Azure.

1. Нажмите кнопку **Добавить**, чтобы создать проект.

### <a name="add-your-registry-credentials"></a>Добавление учетных данных реестра

Манифест развертывания совместно использует учетные данные для реестра контейнеров со средой выполнения IoT Edge. Среде выполнения нужны эти учетные данные, чтобы извлечь частный образ на устройство IoT Edge. Используйте учетные данные из раздела **Ключи доступа** в реестре контейнеров Azure.

1. В Обозревателе решений Visual Studio откройте файл *deployment.template.json*.

1. Найдите свойство **registryCredentials** в требуемых свойствах $edgeAgent. Адрес реестра свойства должен быть автоматически заполнен данными, которые вы указывали при создании проекта. Поля "имя пользователя" и "пароль" должны содержать имена переменных. Пример:

   ```json
   "registryCredentials": {
     "<registry name>": {
       "username": "$CONTAINER_REGISTRY_USERNAME_<registry name>",
       "password": "$CONTAINER_REGISTRY_PASSWORD_<registry name>",
       "address": "<registry name>.azurecr.io"
     }
   }
   ```

1. Откройте файл среды (ENV) в решении модуля. По умолчанию он скрыт в Обозревателе решений. Чтобы отобразить его, нажмите кнопку **Показать все файлы**. ENV-файл должен содержать те же переменные имени пользователя и пароля, которые вы видели в файле *deployment.template.json*.

1. Добавьте значения **имени пользователя** и **пароля**, скопированные из реестра контейнеров Azure.

1. Сохраните изменения в ENV-файле.

### <a name="update-the-module-with-custom-code"></a>Обновление модуля с помощью пользовательского кода

Код стандартного модуля получает сообщения из очереди входящих сообщений и передает их через очередь исходящих сообщений. Давайте добавим еще немного кода, чтобы модуль обрабатывал сообщения на границе до передачи в Центр Интернета вещей. Обновите модуль так, чтобы он анализировал данные о температуре, получаемые в каждом сообщении, и отправлял в центр Интернета вещей только сообщения со сведениями о том, что температура превышает определенный порог.

1. В Visual Studio выберите **CSharpModule** > **Program.cs**.

1. В верхней части пространства имен **CSharpModule** добавьте три **инструкции** для типов, которые будут использоваться позже.

    ```csharp
    using System.Collections.Generic;     // For KeyValuePair<>
    using Microsoft.Azure.Devices.Shared; // For TwinCollection
    using Newtonsoft.Json;                // For JsonConvert
    ```

1. Добавьте переменную **temperatureThreshold** к классу **Program** после переменной счетчика. Для переменной temperatureThreshold указывается значение измеренной температуры, при превышении которого данные отправляются в Центр Интернета вещей.

    ```csharp
    static int temperatureThreshold { get; set; } = 25;
    ```

1. Добавьте классы **MessageBody**, **Machine** и **Ambient** к классу **Program** после объявления переменных. Эти классы определяют ожидаемую схему текста входящего сообщения.

    ```csharp
    class MessageBody
    {
        public Machine machine {get;set;}
        public Ambient ambient {get; set;}
        public string timeCreated {get; set;}
    }
    class Machine
    {
        public double temperature {get; set;}
        public double pressure {get; set;}
    }
    class Ambient
    {
        public double temperature {get; set;}
        public int humidity {get; set;}
    }
    ```

1. Найдите метод **Init**. Этот метод создает и настраивает объект **ModuleClient**, позволяющий модулю подключаться к локальной среде выполнения Azure IoT Edge для отправки и получения сообщений. Этот код также регистрирует обратный вызов для получения сообщений из центра IoT Edge через конечную точку **input1**.

   Замените весь метод Init приведенным ниже кодом.

   ```csharp
   static async Task Init()
   {
       AmqpTransportSettings amqpSetting = new AmqpTransportSettings(TransportType.Amqp_Tcp_Only);
       ITransportSettings[] settings = { amqpSetting };

       // Open a connection to the Edge runtime.
       ModuleClient ioTHubModuleClient = await ModuleClient.CreateFromEnvironmentAsync(settings);
       await ioTHubModuleClient.OpenAsync();
       Console.WriteLine("IoT Hub module client initialized.");

       // Read the TemperatureThreshold value from the module twin's desired properties.
       var moduleTwin = await ioTHubModuleClient.GetTwinAsync();
       await OnDesiredPropertiesUpdate(moduleTwin.Properties.Desired, ioTHubModuleClient);

       // Attach a callback for updates to the module twin's desired properties.
       await ioTHubModuleClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertiesUpdate, null);

       // Register a callback for messages that are received by the module.
       await ioTHubModuleClient.SetInputMessageHandlerAsync("input1", FilterMessages, ioTHubModuleClient);
   }
   ```

   Этот обновленный метод Init по-прежнему устанавливает подключение к среде выполнения IoT Edge с помощью ModuleClient, а также добавляет новые функции. Он считывает требуемые свойства двойника модуля, чтобы получить значение **temperatureThreshold**. Затем он создает обратный вызов, который прослушивает все будущие обновления требуемых свойств двойника модуля. С помощью этого обратного вызова можно удаленно обновить пороговое значение температуры в двойнике модуля, и эти изменения будут включены в модуль.

   Обновленный метод Init также изменяет существующий метод **SetInputMessageHandlerAsync**. В примере кода входящие сообщения на *input1* обрабатываются с помощью функции *PipeMessage*. Но нам нужно изменить ее на функцию *FilterMessages*, которую создадим на следующих этапах.

1. Добавьте новый метод **onDesiredPropertiesUpdate** к классу **Program**. Этот метод принимает изменения требуемых свойств из двойника модуля и соответствующим образом изменяет переменную **temperatureThreshold**. У каждого модуля есть собственный модуль-двойник, что позволяет настроить код, выполняемый в модуле, непосредственно из облака.

    ```csharp
    static Task OnDesiredPropertiesUpdate(TwinCollection desiredProperties, object userContext)
    {
        try
        {
            Console.WriteLine("Desired property change:");
            Console.WriteLine(JsonConvert.SerializeObject(desiredProperties));

            if (desiredProperties["TemperatureThreshold"]!=null)
                temperatureThreshold = desiredProperties["TemperatureThreshold"];

        }
        catch (AggregateException ex)
        {
            foreach (Exception exception in ex.InnerExceptions)
            {
                Console.WriteLine();
                Console.WriteLine("Error when receiving desired property: {0}", exception);
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error when receiving desired property: {0}", ex.Message);
        }
        return Task.CompletedTask;
    }
    ```

1. Удалите пример метода **PipeMessage** и замените его новым методом **FilterMessages**. Этот метод вызывается каждый раз, когда модуль получает сообщение из центра IoT Edge. Он отфильтровывает сообщения о температуре ниже порогового значения, настроенного с помощью двойника модуля. Этот метод также добавляет свойство **MessageType** в сообщение со значением **Alert**.

    ```csharp
    static async Task<MessageResponse> FilterMessages(Message message, object userContext)
    {
        var counterValue = Interlocked.Increment(ref counter);
        try
        {
            ModuleClient moduleClient = (ModuleClient)userContext;
            var messageBytes = message.GetBytes();
            var messageString = Encoding.UTF8.GetString(messageBytes);
            Console.WriteLine($"Received message {counterValue}: [{messageString}]");

            // Get the message body.
            var messageBody = JsonConvert.DeserializeObject<MessageBody>(messageString);

            if (messageBody != null && messageBody.machine.temperature > temperatureThreshold)
            {
                Console.WriteLine($"Machine temperature {messageBody.machine.temperature} " +
                    $"exceeds threshold {temperatureThreshold}");
                using(var filteredMessage = new Message(messageBytes))
                {
                    foreach (KeyValuePair<string, string> prop in message.Properties)
                    {
                        filteredMessage.Properties.Add(prop.Key, prop.Value);
                    }

                    filteredMessage.Properties.Add("MessageType", "Alert");
                    await moduleClient.SendEventAsync("output1", filteredMessage);
                }
            }

            // Indicate that the message treatment is completed.
            return MessageResponse.Completed;
        }
        catch (AggregateException ex)
        {
            foreach (Exception exception in ex.InnerExceptions)
            {
                Console.WriteLine();
                Console.WriteLine("Error in sample: {0}", exception);
            }
            // Indicate that the message treatment is not completed.
            var moduleClient = (ModuleClient)userContext;
            return MessageResponse.Abandoned;
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error in sample: {0}", ex.Message);
            // Indicate that the message treatment is not completed.
            ModuleClient moduleClient = (ModuleClient)userContext;
            return MessageResponse.Abandoned;
        }
    }
    ```

1. Сохраните файл *Program.cs*.

1. Откройте файл *deployment.template.json* в папке решения IoT Edge. Этот файл указывает агенту IoT Edge на модуль для развертывания, а центру IoT Edge на то, как маршрутизировать сообщения между ними. В этом случае развертываемыми модулями являются **SimulatedTemperatureSensor** и **CSharpModule**.

1. Добавьте двойник модуля **CSharpModule** в манифест развертывания. Вставьте следующее содержимое JSON в нижней части раздела `modulesContent` после двойника модуля **$edgeHub**:

    ```json
       "CSharpModule": {
           "properties.desired":{
               "TemperatureThreshold":25
           }
       }
    ```

    ![Снимок экрана: двойник модуля, добавляемый в шаблон развертывания.](./media/tutorial-csharp-module-windows/module-twin.png)

1. Сохраните файл *deployment.template.json*.

## <a name="build-and-push-your-module"></a>Сборка и отправка модуля

В предыдущем разделе показано, как создать решение IoT Edge и добавить в **CSharpModule** код для фильтрации сообщений, в которых указана температура компьютера ниже допустимого порога. Теперь необходимо создать решение в качестве образа контейнера и передать его в реестр контейнеров.

### <a name="sign-in-to-docker"></a>Вход в Docker

1. Используйте следующую команду, чтобы войти в Docker на компьютере разработки. Выполните вход в систему, используя имя пользователя, пароль и сервер входа из реестра контейнеров Azure. Вы можете получить эти значения в разделе **Ключи доступа** в разделе реестра на портале Azure.

   ```cmd
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Возможно, появится предупреждение системы безопасности, в котором рекомендуется использовать `--password-stdin`. Для рабочих сценариев это лучший вариант, но мы не будем рассматривать его в этом руководстве. Дополнительные сведения см. в описании команды [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin) в справочнике.

### <a name="build-and-push"></a>Создание и отправка

1. В Обозревателе решений Visual Studio щелкните правой кнопкой мыши имя проекта, который нужно скомпилировать. Имя по умолчанию — **AzureIotEdgeApp1**, и так как вы создаете модуль Windows, должно быть расширение **Windows.Amd64**.

1. Выберите **Build and Push IoT Edge Modules** (Создание и отправка модулей IoT Edge).

   Эта команда сборки и отправки запускает три операции.
   * Во-первых, она создает в решении новую папку с именем *config*, содержащую полный манифест развертывания на основе информации в шаблоне развертывания и других файлах решения. 
   * Во-вторых, выполняется `docker build` для сборки образа контейнера на основе подходящего файла Dockerfile для целевой архитектуры. 
   * В-третьих, выполняется `docker push` для отправки репозитория образа в реестр контейнеров.

   В первый раз выполнение этого процесса может занять несколько минут, но при следующем запуске команд он выполняется быстрее.

## <a name="deploy-modules-to-the-device"></a>Развертывание модулей на устройстве

Разверните проект модуля на устройство IoT Edge с помощью Visual Studio Cloud Explorer и расширения Azure IoT Edge Tools. У вас уже есть манифест развертывания, подготовленный для вашего сценария (файл *deployment.windows-amd64.json* в папке *config*). Теперь вам осталось выбрать устройство для получения развертывания.

Убедитесь, что устройство IoT Edge работает.

1. В Visual Studio Cloud Explorer разверните ресурсы, чтобы увидеть список устройств Интернета вещей.

1. Щелкните правой кнопкой мыши имя устройства IoT Edge, на котором необходимо выполнить развертывание.

1. Выберите **Создать развертывание**.

1. В проводнике Visual Studio выберите файл *deployment.windows-amd64.json* в папке *config* решения.

1. Обновите Cloud Explorer, чтобы отобразился список развернутых модулей для устройства.

## <a name="view-generated-data"></a>Просмотр сформированных данных

Когда вы примените манифест развертывания к устройству IoT Edge, в среде выполнения IoT Edge на устройстве начнется сбор сведений о новом развертывании и выполнение операций. Работа всех выполняющихся на устройстве модулей, которые не включены в манифест развертывания, будет остановлена. А модули, которых нет на устройстве, будут запущены.

Расширение IoT Edge Tools можно использовать для просмотра сообщений по мере их поступления в центр Интернета вещей.

1. В Visual Studio Cloud Explorer выберите имя устройства IoT Edge.

1. Из списка **Actions** (Действия) выберите **Start Monitoring Built-in Event Endpoint** (Начать мониторинг встроенной конечной точки событий).

1. Просмотрите сообщения, поступающие в центр Интернета вещей. Для поступления сообщений может потребоваться некоторое время, так как устройство IoT Edge должно сначала получить новое развертывание и запустить все модули. После этого измененный нами код CSharpModule ожидает, пока температура компьютера не достигнет 25 градусов, и лишь затем отправляет сообщения. Код также присваивает тип **Оповещение** всем сообщениям со сведениями о достижении порога температуры.

   ![Снимок экрана: окно вывода, в котором отображаются сообщения, поступающие в центр Интернета вещей.](./media/tutorial-csharp-module-windows/view-d2c-message.png)

## <a name="edit-the-module-twin"></a>Изменение двойника модуля

Мы использовали двойник модуля CSharpModule, чтобы задать порог температуры 25 градусов. С помощью двойника модуля вы можете изменять функциональные возможности без необходимости обновлять код модуля.

1. В Visual Studio откройте файл *deployment.windows-amd64.json*. 

   *Не* открывайте файл *deployment.template*. Если вы не видите манифест развертывания в файле *config* в Обозревателе решений, щелкните значок **Показать все файлы** на панели инструментов Обозревателя.

1. Найдите двойник модуля CSharpModule и измените значение параметра **temperatureThreshold** на новое, которое будет на 5–10 градусов выше, чем последнее полученное значение температуры.

1. Сохраните файл *deployment.windows-amd64.json*.

1. Выполните шаги по развертыванию еще раз, чтобы применить к устройству обновленный манифест развертывания.

1. Выполните мониторинг сообщений, поступающих с устройства в облако. Поток сообщений должен быть приостановлен, пока не будет достигнут новый порог температуры.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете перейти к следующей рекомендуемой статье, можно сохранить созданные и повторно используемые ресурсы и конфигурации, с которыми вы работали в этом руководстве. Это же устройство IoT Edge также можно использовать в качестве тестового устройства.

В противном случае вы можете удалить локальные конфигурации и ресурсы Azure, с которыми вы работали в этом руководстве, чтобы избежать расходов.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы создали модуль IoT Edge с кодом для фильтрации необработанных данных, созданных устройством IoT Edge.

Перейдите к следующим руководствам, чтобы узнать, как Azure IoT Edge поможет развернуть облачные службы Azure для обработки и анализа данных на пограничном устройстве.

> [!div class="nextstepaction"]
> [Функции Azure](tutorial-deploy-function.md)
> [Azure Stream Analytics](tutorial-deploy-stream-analytics.md)
> [Машинное обучение Azure](tutorial-deploy-machine-learning.md)
> [Пользовательское визуальное распознавание](tutorial-deploy-custom-vision.md)
