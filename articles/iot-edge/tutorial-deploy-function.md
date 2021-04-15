---
title: Руководство по Развертывание Функций Azure в виде модулей с использованием Azure IoT Edge
description: В этом учебнике описано, как разработать Функции Azure в виде модуля IoT Edge с последующим развертыванием на пограничном устройстве.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/29/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: 693c181f8a4a6db3b8b163f4b4d3350a3730b618
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/02/2021
ms.locfileid: "106221653"
---
# <a name="tutorial-deploy-azure-functions-as-iot-edge-modules"></a>Руководство по Развертывание Функций Azure как модулей IoT Edge

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

Службу "Функции Azure" можно использовать для развертывания кода, который реализует нужную бизнес-логику, непосредственно на устройствах Azure IoT Edge. В этом руководстве рассматривается создание и развертывание службы "Функции Azure", которая фильтрует данные датчика на имитированном устройстве IoT Edge. Вы используете имитированное устройство IoT Edge, созданное при изучении кратких руководств. В этом руководстве описано следующее:

> [!div class="checklist"]
>
> * Использование Visual Studio Code для создания Функции Azure.
> * Использовать VS Code и Docker для создания образа Docker и его публикации в реестре контейнеров.
> * Развертывать модуль из реестра контейнеров на устройстве IoT Edge.
> * Просматривать отфильтрованные данные.

<center>

![Схема архитектуры из руководства: размещение и развертывание Функций как модуля](./media/tutorial-deploy-function/functions-architecture.png)
</center>

Функция Azure, создание которой описано в этом руководстве, фильтрует данные температуры, созданные устройством. Она отправляет сообщения в вышестоящий Центр Интернета вещей Azure, когда температура превышает заданное пороговое значение.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим учебником вы уже должны изучить предыдущий учебник по настройке среды для разработки контейнеров Linux: [Разработка модулей IoT Edge с использованием контейнеров Linux](tutorial-develop-for-linux.md). После работы с ним у вас должны быть готовы все необходимые компоненты:

* [Центр Интернета вещей](../iot-hub/iot-hub-create-through-portal.md) ценовой категории "Бесплатный" или "Стандартный" в Azure.
* Устройство, на котором выполняется Azure IoT Edge с контейнерами Linux. Вы можете воспользоваться краткими руководствами для настройки устройства [Linux](quickstart-linux.md) или [Windows](quickstart.md).
* реестр контейнеров, например [Реестр контейнеров Azure](../container-registry/index.yml);
* средство [Visual Studio Code](https://code.visualstudio.com/), настроенное с помощью [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools);
* выпуск [Docker CE](https://docs.docker.com/install/), настроенный для выполнения контейнеров Linux.

Для разработки модуля IoT Edge с Функциями Azure установите на компьютере разработки следующие дополнительные компоненты:

* [C# для расширения Visual Studio Code (на платформе OmniSharp)](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).
* [пакет SDK для .NET Core 2.1](https://www.microsoft.com/net/download).

## <a name="create-a-function-project"></a>Создание проекта функции

Средства Azure IoT для Visual Studio Code, установленные при выполнении предварительных требований, предоставляют возможности управления, а также некоторые шаблоны кода. В этом разделе Visual Studio Code используется для создания решения IoT Edge, содержащего Функцию Azure.

### <a name="create-a-new-project"></a>Создание нового проекта

Создайте шаблон решения C# с Функциями Azure, который можно настроить с помощью собственного кода.

1. Откройте Visual Studio Code на компьютере для разработки.

2. Выберите **Представление** > **Палитра команд** для открытия палитры команд VS Code.

3. В палитре команд введите и выполните команду **Azure IoT Edge: New IoT Edge Solution** (Azure IoT Edge: создать решение IoT Edge). Следуйте инструкциям на экране в палитре команд для создания решения.

   | Поле | Значение |
   | ----- | ----- |
   | Выбор папки | Выберите расположение на компьютере разработчика для VS Code, чтобы создать файлы решения. |
   | Введите название решения. | Введите описательное имя для решения, например **FunctionSolution**, или примите значение по умолчанию. |
   | Выбор шаблона модуля | Выберите **Функции Azure — C#** . |
   | Указание имени модуля | Назовите модуль **CSharpFunction**. |
   | Указание репозитория изображений Docker для модуля | Репозиторий изображений включает в себя имя реестра контейнеров и имя образа контейнера. Образ контейнера предварительно заполняется на последнем шаге. Замените **localhost:5000** значением **сервера входа** из реестра контейнеров Azure. Вы можете узнать сервер входа на странице "Обзор" реестра контейнеров на портале Azure. Окончательная строка выглядит следующим образом: \<registry name\>.azurecr.io/CSharpFunction. |

   ![Выбор репозитория образа Docker](./media/tutorial-deploy-function/repository.png)

### <a name="add-your-registry-credentials"></a>Добавление учетных данных реестра

Файл среды хранит учетные данные для реестра контейнеров и совместно использует их со средой выполнения IoT Edge. Среде выполнения нужны эти учетные данные, чтобы извлечь частный образ на устройство IoT Edge.

Расширение IoT Edge пытается извлечь учетные данные реестра контейнеров из Azure и заполнить их в файле среды. Проверьте, включены ли ваши учетные данные. В противном случае добавьте их:

1. Откройте в обозревателе VS Code файл с расширением ENV.
2. Обновите поля с **именем пользователя** и **паролем**, скопированные из своего реестра контейнера Azure.
3. Сохраните этот файл.

### <a name="select-your-target-architecture"></a>Выбор целевой архитектуры

Сейчас Visual Studio Code позволяет разрабатывать модули C для устройств Linux AMD64 и Linux ARM32v7. Для каждого решения вам нужно выбрать одну целевую платформу, так как сборка и запуск контейнера для разных архитектур различается. По умолчанию используется Linux AMD64.

1. Откройте палитру команд и выполните поиск **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Azure IoT Edge: установить целевую платформу по умолчанию для решения Edge) или выберите значок ярлыка на боковой панели в нижней части окна.

2. В палитре команд выберите целевую архитектуру из списка параметров. В рамках этого руководства мы используем в качестве устройства IoT Edge виртуальную машину Ubuntu. Поэтому сохраним значение по умолчанию **amd64**.

### <a name="update-the-module-with-custom-code"></a>Обновление модуля с помощью пользовательского кода

Давайте добавим еще немного кода, чтобы модуль обрабатывал сообщения на границе до передачи в Центр Интернета вещей.

1. В Visual Studio Code откройте **Модули** > **CSharpFunction** > **CSharpFunction.cs**.

1. Измените содержимое файла **CSharpFunction.cs** на код, приведенный ниже. Этот код получает данные телеметрии о температуре компьютера и окружающей среды и передает сообщение в Центр Интернета вещей, только если температура компьютера превышает указанный порог.

   ```csharp
   using System;
   using System.Collections.Generic;
   using System.IO;
   using System.Text;
   using System.Threading.Tasks;
   using Microsoft.Azure.Devices.Client;
   using Microsoft.Azure.WebJobs;
   using Microsoft.Azure.WebJobs.Extensions.EdgeHub;
   using Microsoft.Azure.WebJobs.Host;
   using Microsoft.Extensions.Logging;
   using Newtonsoft.Json;

   namespace Functions.Samples
   {
       public static class CSharpFunction
       {
           [FunctionName("CSharpFunction")]
           public static async Task FilterMessageAndSendMessage(
               [EdgeHubTrigger("input1")] Message messageReceived,
               [EdgeHub(OutputName = "output1")] IAsyncCollector<Message> output,
               ILogger logger)
           {
               const int temperatureThreshold = 20;
               byte[] messageBytes = messageReceived.GetBytes();
               var messageString = System.Text.Encoding.UTF8.GetString(messageBytes);

               if (!string.IsNullOrEmpty(messageString))
               {
                   logger.LogInformation("Info: Received one non-empty message");
                   // Get the body of the message and deserialize it.
                   var messageBody = JsonConvert.DeserializeObject<MessageBody>(messageString);

                   if (messageBody != null && messageBody.machine.temperature > temperatureThreshold)
                   {
                       // Send the message to the output as the temperature value is greater than the threshold.
                       using (var filteredMessage = new Message(messageBytes))
                       {
                            // Copy the properties of the original message into the new Message object.
                            foreach (KeyValuePair<string, string> prop in messageReceived.Properties)
                            {filteredMessage.Properties.Add(prop.Key, prop.Value);}
                            // Add a new property to the message to indicate it is an alert.
                            filteredMessage.Properties.Add("MessageType", "Alert");
                            // Send the message.
                            await output.AddAsync(filteredMessage);
                            logger.LogInformation("Info: Received and transferred a message with temperature above the threshold");
                       }
                   }
               }
           }
       }
       //Define the expected schema for the body of incoming messages.
       class MessageBody
       {
           public Machine machine {get; set;}
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
   }
   ```

1. Сохраните файл.

## <a name="build-and-push-your-iot-edge-solution"></a>Создание и отправка решения IoT Edge

В предыдущем разделе создано решение IoT Edge и изменено **CSharpFunction**, который будет фильтровать сообщения, в которых температура компьютера ниже допустимого порогового значения. Теперь необходимо создать решение в качестве образа контейнера и передать его в реестр контейнеров.

1. Откройте интегрированный терминал VS Code, выбрав **Вид** > **Терминал**.

2. Войдите в Docker. Для этого введите представленную ниже команду в окне терминала. Выполните вход в систему, используя имя пользователя, пароль и сервер входа из реестра контейнеров Azure. Вы можете получить эти значения в разделе **Ключи доступа** в разделе реестра на портале Azure.

   ```bash
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Возможно, появится предупреждение системы безопасности, в котором рекомендуется использовать `--password-stdin`. Для рабочих сценариев это лучшая методика, но мы не будем рассматривать ее в этом учебнике. Дополнительные сведения см. в описании команды [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin) в справочнике.

3. В обозревателе VS Code щелкните правой кнопкой мыши файл **deployment.template.json** и выберите **Build and Push IoT Edge Solution** (Создать и отправить решение IoT Edge).

   Эта команда сборки и отправки позволяет запустить три операции. Во-первых, в решении создается папка с именем **config**, которая содержит полный манифест развертывания на основе информации из шаблона развертывания и других файлов решения. Во-вторых, выполняется `docker build` для сборки образа контейнера на основе подходящего файла dockerfile для целевой архитектуры. В-третьих, выполняется `docker push` для отправки образа в реестр контейнеров.

   В первый раз выполнение этого процесса может занять несколько минут, но при следующем запуске команд он будет быстрее.

## <a name="view-your-container-image"></a>Просмотр образа контейнера

Visual Studio Code выводит сообщение об успешном завершении, когда образ контейнера передается в ваш реестр контейнеров. Если требуется лично убедиться в успешности выполнения, можно просмотреть образ в реестре.

1. Перейдите в реестр контейнеров на портале Azure.
2. Выберите **Репозитории**.
3. Репозиторий **csharpfunction** отобразится в списке. Чтобы просмотреть дополнительные сведения о репозитории, выберите его.
4. В разделе **Теги** должен отобразиться тег **0.0.1-amd64**. Он указывает версию и платформу созданного образа. Эти значения устанавливаются в файле module.json в папке CSharpFunction.

## <a name="deploy-and-run-the-solution"></a>Развертывание и запуск решения

Для развертывания модуля функции на устройстве IoT Edge можно использовать портал Azure (как это было показано в кратком руководстве). Также в Visual Studio Code можно развертывать модули и проводить их мониторинг. В следующих разделах используются средства Azure IoT для VS Code, указанные в предварительных требованиях. Если расширение не было установлено, то теперь это можно сделать.

1. В обозревателе Visual Studio Code в разделе **Центр Интернета вещей Azure** разверните меню **Устройства**, чтобы отобразить список устройств Интернета вещей.

2. Щелкните имя устройства IoT Edge правой кнопкой мыши, а затем выберите **Create Deployment for Single Device** (Создать развертывание для одного устройства).

3. Перейдите в папку решения, содержащую **CSharpFunction**. Откройте папку конфигурации, выберите файл **deployment.amd64.json** и затем **Select Edge Deployment Manifest** (Выбрать манифест развертывания Edge).

4. Разверните меню **Модули** для своего устройства, чтобы просмотреть список развернутых и запущенных модулей. Нажмите кнопку "Обновить". Появится новый запущенный модуль **CSharpFunction**, модуль **SimulatedTemperatureSensor**, а также **$edgeAgent** и **$edgeHub**.

    Может потребоваться несколько минут, чтобы новые модули отобразились. Устройство IoT Edge должно получить сведения о новом развертывании из Центра Интернета вещей, запустить новые контейнеры и затем сообщить о своем состоянии в Центр Интернета вещей.

   ![Просмотр развернутых модулей в VS Code](./media/tutorial-deploy-function/view-modules.png)

## <a name="view-the-generated-data"></a>Просмотр созданных данных

Чтобы отобразить все сообщения, поступающие в Центр Интернета вещей, в палитре команд выполните команду **Центр Интернета вещей: Start Monitoring Built-in Event Endpoint** (Начать мониторинг встроенной конечной точки событий).

Также можно отфильтровать представления, чтобы увидеть все сообщения, поступающие в Центр Интернета вещей с определенного устройства. Щелкните имя устройства правой кнопкой мыши в разделе **Azure IoT Hub Devices** (Устройства Центра Интернета вещей Azure) и выберите **Start Monitoring Built-in Event Endpoint** (Начать мониторинг встроенной конечной точки событий).

Чтобы перестать отслеживать сообщения, в палитре команд выполните команду **Центр Интернета вещей: Stop Monitoring Built-in Event Endpoint** (Остановить мониторинг встроенной конечной точки событий).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете перейти к следующей рекомендуемой статье, можно сохранить созданные и повторно используемые ресурсы и конфигурации. Это же устройство IoT Edge также можно использовать в качестве тестового устройства.

В противном случае чтобы избежать расходов, можно удалить локальные конфигурации и ресурсы Azure, созданные в рамках этой статьи.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как создать модуль Функции Azure, содержащий код для фильтрации необработанных данных, которые были созданы устройством IoT Edge.

Чтобы узнать о других способах, с помощью которых Azure IoT Edge может помочь превратить данные пограничного устройства в бизнес-аналитику, перейдите к следующим руководствам.

> [!div class="nextstepaction"]
> [Руководство по развертыванию Azure Stream Analytics в качестве модуля IoT Edge (предварительная версия)](tutorial-deploy-stream-analytics.md)
