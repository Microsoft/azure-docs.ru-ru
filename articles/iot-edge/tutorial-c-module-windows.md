---
title: Руководство. Разработка модулей C для Windows с помощью Azure IoT Edge
description: В этом руководстве показано, как создавать модули IoT Edge c использованием кода C и развертывать их на устройстве с Windows под управлением IoT Edge.
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 05/28/2019
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
monikerRange: =iotedge-2018-06
ms.openlocfilehash: a67bf92dbee9bf3d444add3be32544e4c3eba4f3
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/02/2021
ms.locfileid: "106221608"
---
# <a name="tutorial-develop-c-iot-edge-modules-using-windows-containers"></a>Руководство. Разработка модулей IoT Edge на C с использованием контейнеров Windows

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

В этой статье показано, как разрабатывать и развертывать код C на устройствах Windows с Azure IoT Edge с помощью Visual Studio.

>[!NOTE]
>IoT Edge 1.1 LTS — это последний канал выпусков, который поддерживает контейнеры Windows. Начиная с версии 1.2, контейнеры Windows не поддерживаются. Мы рекомендуем вам использовать [IoT Edge для Linux в Windows](iot-edge-for-linux-on-windows.md) для выполнения IoT Edge на устройствах Windows.

Вы можете использовать модули Azure IoT Edge для развертывания кода, который реализует бизнес-логику непосредственно на устройствах IoT Edge. В этом руководстве рассматриваются создание и развертывание модуля IoT Edge, который фильтрует данные датчика.

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> * Создать модуль IoT Edge на основе пакета SDK для C с помощью Visual Studio.
> * Использовать Visual Studio и Docker для создания образа Docker и его публикации в реестре.
> * Развертывать модуль на устройстве IoT Edge.
> * Просматривать сформированные данные.

Модуль IoT Edge, создаваемый в этом руководстве, фильтрует данные температуры, которые создаются устройством. Он отправляет сообщения в потоке, только если температура превышает заданный порог. Такой тип пограничного анализа удобен для сокращения объема данных, передаваемых в облако и сохраняемых в нем.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

В этом руководстве показано, как разработать модуль на языке C с помощью Visual Studio 2019 и развернуть его на устройстве Windows. Если вы разрабатываете модули с использованием контейнеров Linux, см. статью [Разработка модулей IoT Edge на C с использованием контейнеров Linux](tutorial-csharp-module.md).

В следующей таблице приведены возможные варианты для разработки и развертывания модулей C с использованием контейнеров Windows:

| C | Visual&nbsp;Studio&nbsp;Code | Visual Studio 2017&nbsp;и&nbsp;2019 |
| -- | ------------------ | :------------------: |
| Windows AMD64 |  | ![Разработка модулей C для WinAMD64 в Visual Studio](./media/tutorial-c-module/green-check.png) |

Прежде чем начинать работу с этим руководством, настройте среду разработки, как описано в статье [Разработка модулей IoT Edge с использованием контейнеров Windows](tutorial-develop-for-windows.md). После этого среда будет содержать следующие необходимые компоненты:

* [Центр Интернета вещей](../iot-hub/iot-hub-create-through-portal.md) ценовой категории "Бесплатный" или "Стандартный" в Azure.
* [устройство Windows, на котором выполняется Azure IoT Edge](how-to-install-iot-edge-windows-on-windows.md);
* реестр контейнеров, например [Реестр контейнеров Azure](../container-registry/index.yml);
* настроенная среда [Visual Studio 2019](/visualstudio/install/install-visual-studio) с расширением [Azure IoT Edge Tools](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16iotedgetools);
* приложение [Docker Desktop](https://docs.docker.com/docker-for-windows/install/), настроенное для запуска контейнеров Windows.

Установите пакет SDK Azure IoT для C для Windows x64 с помощью vcpkg, выполнив следующие команды:

   ```powershell
   git clone https://github.com/Microsoft/vcpkg
   cd vcpkg
   .\bootstrap-vcpkg.bat
   .\vcpkg install azure-iot-sdk-c:x64-windows
   .\vcpkg --triplet x64-windows integrate install
   ```

> [!TIP]
> Если вы используете Visual Studio 2017 (15.7 или более поздней версии), скачайте и установите Azure IoT Edge Tools для Visual Studio 2017 из [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vsiotedgetools).

## <a name="create-a-module-project"></a>Создание проекта модуля

В этом разделе показано, как создать проект модуля IoT Edge на основе пакета SDK для С, используя Visual Studio и расширение Azure IoT Edge Tools. Создав шаблон проекта, добавьте новый код, чтобы модуль фильтровал сообщения по их сообщаемым свойствам.

### <a name="create-a-new-project"></a>Создание нового проекта

Создайте шаблон решения C, которое можно настроить с помощью собственного кода.

1. Откройте Visual Studio 2019 и выберите **Создать проект**.

1. В области **Создание нового проекта** введите **IoT Edge** для поиска, а затем в списке результатов выберите проект **Azure IoT Edge (Windows AMD64)** .

   ![Снимок экрана: область "Создание нового проекта" для IoT Edge.](./media/tutorial-c-module-windows/new-project.png)

1. Выберите **Далее**.

    Откроется область **Настроить новый проект**.

   ![Снимок экрана: область "Настроить новый проект".](./media/tutorial-c-module-windows/configure-project.png)

1. В области **Настроить новый проект** переименуйте проект и решение, указав описательное имя, например **CTutorialApp**. 

1. Выберите **Создать**, чтобы создать проект.

   Откроется область **Добавление модуля**.

   ![Снимок экрана: область "Добавление модуля" для настройки проекта.](./media/tutorial-c-module-windows/add-application-and-module.png)

1. На странице **Настроить новый проект** сделайте следующее:

   а. В области слева выберите шаблон **Модуль С**.  
   b. В поле **Имя модуля** введите **CModule**.  
   c. В поле **URL-адрес репозитория** замените **localhost:5000** на значение **сервера входа** из реестра контейнеров Azure в следующем формате: `<registry name>.azurecr.io/cmodule`.

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

Код стандартного модуля получает сообщения из очереди входящих сообщений и передает их через очередь исходящих сообщений. Давайте добавим еще немного кода, чтобы модуль обрабатывал сообщения в пограничной среде до передачи в центр Интернета вещей. Обновите модуль так, чтобы он анализировал данные о температуре, получаемые в каждом сообщении, и отправлял в центр Интернета вещей только сообщения со сведениями о том, что температура превышает определенный порог.

1. Данные с датчиков в этом сценарии поступают в формате JSON. Для фильтрации сообщений в формате JSON импортируйте библиотеку JSON для модуля C. Это руководство использует Parson.

   а. Скачайте [репозиторий GitHub для Parson](https://github.com/kgabis/parson).  
   b. Скопируйте файлы *parson.c* и *parson.h* в проект CModule.  
   c. В Visual Studio откройте файл *CMakeLists.txt* из папки проекта CModule.  
   d. В верхней части файла импортируйте файлы Parson в качестве библиотеки, называемой **my_parson**.

      ```txt
      add_library(my_parson
          parson.c
          parson.h
      )
      ```  
   д) Добавьте `my_parson` в список библиотек в разделе target_link_libraries файла *CMakeLists.txt*.  
   е) Сохраните файл *CMakeLists.txt*.  
   ж. Выберите **CModule** > **main.c**. В нижней части списка содержатся операторы. Добавьте новый оператор, чтобы включить `parson.h` для поддержки JSON:

      ```c
      #include "parson.h"
      ```

1. В файле *main.c* добавьте глобальную переменную с именем `temperatureThreshold` рядом с переменной `messagesReceivedByInput1Queue`. Эта переменная устанавливает значение, которое должно быть измеренной температурой, чтобы данные были отправлены в центр Интернета вещей.

    ```c
    static double temperatureThreshold = 25;
    ```

1. Найдите функцию `CreateMessageInstance` в файле *main.c*. Замените внутренний оператор *if-else* следующим кодом, добавив несколько строк, которые определяют новые возможности:

   ```c
   if ((messageInstance->messageHandle = IoTHubMessage_Clone(message)) == NULL)
   {
       free(messageInstance);
       messageInstance = NULL;
   }
   else
   {
       messageInstance->messageTrackingId = messagesReceivedByInput1Queue;
       MAP_HANDLE propMap = IoTHubMessage_Properties(messageInstance->messageHandle);
       if (Map_AddOrUpdate(propMap, "MessageType", "Alert") != MAP_OK)
       {
          printf("ERROR: Map_AddOrUpdate Failed!\r\n");
       }
   }
   ```

   Эти новые строки кода в операторе *else* добавляют в сообщение новое свойство, которое маркирует сообщение как оповещение. Этот код маркирует все сообщения как оповещения, так как наша новая функция отправляет в центр Интернета вещей сообщения со сведениями о высокой температуре.

1. Найдите функцию `InputQueue1Callback` и замените всю функцию следующим кодом. Эта функция реализует фактический фильтр обмена сообщениями. При получении сообщения функция проверяет, превышает ли температура, указанная в сообщении, заданный порог. Если температура превышает порог, функция пересылает сообщение через выходную очередь. Если порог не превышен, функция игнорирует сообщение.

    ```c
    static unsigned char *bytearray_to_str(const unsigned char *buffer, size_t len)
    {
        unsigned char *ret = (unsigned char *)malloc(len + 1);
        memcpy(ret, buffer, len);
        ret[len] = '\0';
        return ret;
    }

    static IOTHUBMESSAGE_DISPOSITION_RESULT InputQueue1Callback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
    {
        IOTHUBMESSAGE_DISPOSITION_RESULT result;
        IOTHUB_CLIENT_RESULT clientResult;
        IOTHUB_MODULE_CLIENT_LL_HANDLE iotHubModuleClientHandle = (IOTHUB_MODULE_CLIENT_LL_HANDLE)userContextCallback;

        unsigned const char* messageBody;
        size_t contentSize;

        if (IoTHubMessage_GetByteArray(message, &messageBody, &contentSize) == IOTHUB_MESSAGE_OK)
        {
            messageBody = bytearray_to_str(messageBody, contentSize);
        } else
        {
            messageBody = "<null>";
        }

        printf("Received Message [%zu]\r\n Data: [%s]\r\n",
                messagesReceivedByInput1Queue, messageBody);

        // Check whether the message reports temperatures that exceed the threshold
        JSON_Value *root_value = json_parse_string(messageBody);
        JSON_Object *root_object = json_value_get_object(root_value);
        double temperature;

        // If temperature exceeds the threshold, send to output1
        if (json_object_dotget_value(root_object, "machine.temperature") != NULL && (temperature = json_object_dotget_number(root_object, "machine.temperature")) > temperatureThreshold)
        {
            printf("Machine temperature %f exceeds threshold %f\r\n", temperature, temperatureThreshold);
            // This message should be sent to next stop in the pipeline, namely "output1".  What happens at "outpu1" is determined
            // by the configuration of the Edge routing table setup.
            MESSAGE_INSTANCE *messageInstance = CreateMessageInstance(message);
            if (NULL == messageInstance)
            {
                result = IOTHUBMESSAGE_ABANDONED;
            }
            else
            {
                printf("Sending message (%zu) to the next stage in pipeline\n", messagesReceivedByInput1Queue);

                clientResult = IoTHubModuleClient_LL_SendEventToOutputAsync(iotHubModuleClientHandle, messageInstance->messageHandle, "output1", SendConfirmationCallback, (void *)messageInstance);
                if (clientResult != IOTHUB_CLIENT_OK)
                {
                    IoTHubMessage_Destroy(messageInstance->messageHandle);
                    free(messageInstance);
                    printf("IoTHubModuleClient_LL_SendEventToOutputAsync failed on sending msg#=%zu, err=%d\n", messagesReceivedByInput1Queue, clientResult);
                    result = IOTHUBMESSAGE_ABANDONED;
                }
                else
                {
                    result = IOTHUBMESSAGE_ACCEPTED;
                }
            }
        }
        // If message does not exceed the threshold, do not forward
        else
        {
            printf("Not sending message (%zu) to the next stage in pipeline.\r\n", messagesReceivedByInput1Queue);
            result = IOTHUBMESSAGE_ACCEPTED;
        }

        messagesReceivedByInput1Queue++;
        return result;
    }
    ```

1. Добавьте функцию `moduleTwinCallback`. Этот метод принимает изменения требуемых свойств из двойника модуля и соответствующим образом изменяет переменную **temperatureThreshold**. У каждого модуля есть собственный двойник модуля, что позволяет настроить код, выполняемый в модуле, непосредственно из облака.

    ```c
    static void moduleTwinCallback(DEVICE_TWIN_UPDATE_STATE update_state, const unsigned char* payLoad, size_t size, void* userContextCallback)
    {
        printf("\r\nTwin callback called with (state=%s, size=%zu):\r\n%s\r\n",
            MU_ENUM_TO_STRING(DEVICE_TWIN_UPDATE_STATE, update_state), size, payLoad);
        JSON_Value *root_value = json_parse_string(payLoad);
        JSON_Object *root_object = json_value_get_object(root_value);
        if (json_object_dotget_value(root_object, "desired.TemperatureThreshold") != NULL) {
            temperatureThreshold = json_object_dotget_number(root_object, "desired.TemperatureThreshold");
        }
        if (json_object_get_value(root_object, "TemperatureThreshold") != NULL) {
            temperatureThreshold = json_object_get_number(root_object, "TemperatureThreshold");
        }
    }
    ```

1. Найдите функцию `SetupCallbacksForModule`. Замените эту функцию приведенным ниже кодом с оператором *else-if* для проверки обновления двойника модуля.

   ```c
   static int SetupCallbacksForModule(IOTHUB_MODULE_CLIENT_LL_HANDLE iotHubModuleClientHandle)
   {
       int ret;

       if (IoTHubModuleClient_LL_SetInputMessageCallback(iotHubModuleClientHandle, "input1", InputQueue1Callback, (void*)iotHubModuleClientHandle) != IOTHUB_CLIENT_OK)
       {
           printf("ERROR: IoTHubModuleClient_LL_SetInputMessageCallback(\"input1\")..........FAILED!\r\n");
           ret = MU_FAILURE;
       }
       else if (IoTHubModuleClient_LL_SetModuleTwinCallback(iotHubModuleClientHandle, moduleTwinCallback, (void*)iotHubModuleClientHandle) != IOTHUB_CLIENT_OK)
       {
           printf("ERROR: IoTHubModuleClient_LL_SetModuleTwinCallback(default)..........FAILED!\r\n");
           ret = MU_FAILURE;
       }
       else
       {
           ret = 0;
       }

       return ret;
   }
   ```

1. Сохраните файл *main.c*.

1. Откройте файл *deployment.template.json*.

1. Добавьте двойник модуля **CModule** в манифест развертывания. Вставьте следующее содержимое JSON в нижней части раздела `moduleContent` после двойника модуля `$edgeHub`:

   ```json
   "CModule": {
       "properties.desired":{
           "TemperatureThreshold":25
       }
   }
   ```

   ![Снимок экрана: двойник модуля, добавляемый в шаблон развертывания.](./media/tutorial-c-module-windows/module-twin.png)

1. Сохраните файл *deployment.template.json*.

## <a name="build-and-push-your-module"></a>Сборка и отправка модуля

В предыдущем разделе показано, как создать решение IoT Edge и добавить в **CModule** код для фильтрации сообщений, в которых указана температура компьютера ниже допустимого порога. Теперь необходимо создать решение в качестве образа контейнера и передать его в реестр контейнеров.

### <a name="sign-in-to-docker"></a>Вход в Docker

Предоставьте свои учетные данные реестра контейнеров в Docker компьютеру для разработки, чтобы он мог передать образ контейнера для хранения в реестре.

1. Откройте окно командной строки или PowerShell.

1. Войдите в Docker с помощью учетных данных реестра контейнеров Azure, которые вы сохранили после создания реестра.

   ```cmd
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Возможно, появится предупреждение системы безопасности, в котором рекомендуется использовать `--password-stdin`. Для рабочих сценариев это лучший вариант, но мы не будем рассматривать его в этом руководстве. Дополнительные сведения см. в описании команды [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin) в справочнике.

### <a name="build-and-push"></a>Создание и отправка

Компьютер для разработки и устройства IoT Edge теперь имеют доступ к реестру контейнеров. Пришло время превратить код проекта в образ контейнера.

1. В Обозревателе решений Visual Studio щелкните правой кнопкой мыши имя проекта, который нужно скомпилировать. Имя по умолчанию — **AzureIotEdgeApp1**. В этом руководстве используется имя **CTutorialApp**. Так как вы создаете модуль Windows, расширение должно быть **Windows.Amd64**.

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

1. Просмотрите сообщения, поступающие в центр Интернета вещей. Для поступления сообщений может потребоваться некоторое время, так как устройство IoT Edge должно сначала получить новое развертывание и запустить все модули. После этого измененный нами код CModule ожидает, пока температура компьютера не достигнет 25 градусов, и лишь затем отправляет сообщения. Код также присваивает тип **Оповещение** всем сообщениям со сведениями о достижении порога температуры.

   ![Снимок экрана: окно вывода, в котором отображаются сообщения, поступающие в центр Интернета вещей.](./media/tutorial-c-module-windows/view-d2c-message.png)

## <a name="edit-the-module-twin"></a>Изменение двойника модуля

Мы использовали двойник модуля CModule, чтобы задать порог температуры 25 градусов. С помощью двойника модуля вы можете изменять функциональные возможности без необходимости обновлять код модуля.

1. В Visual Studio откройте файл *deployment.windows-amd64.json*. 

   *Не* открывайте файл *deployment.template*. Если вы не видите манифест развертывания в файле *config* в Обозревателе решений, щелкните значок **Показать все файлы** на панели инструментов Обозревателя.

1. Найдите двойник модуля CModule и измените значение параметра **temperatureThreshold** на новое, которое будет на 5–10 градусов выше, чем последнее полученное значение температуры.

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
