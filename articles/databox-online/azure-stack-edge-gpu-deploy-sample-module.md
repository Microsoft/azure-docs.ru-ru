---
title: Развертывание модуля GPU на устройстве Azure Stack ребра Pro GPU | Документация Майкрософт
description: В этой статье описывается, как включить функцию "вычисление" и сделать устройство Azure Stack пограничной Pro готовым для вычислений через локальный пользовательский интерфейс.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: 4142542c9f17ae464a996df310c50d73d8711d8e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102438205"
---
# <a name="deploy-a-gpu-enabled-iot-module-on-azure-stack-edge-pro-gpu-device"></a>Развертывание модуля IoT с включенным GPU на устройстве Azure Stack ребра Pro GPU

[!INCLUDE [applies-to-GPU-and-pro-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-sku.md)]

В этой статье описывается, как развернуть модуль IoT Edge с поддержкой GPU на устройстве Azure Stack ребра Pro GPU. 

Вы узнаете, как выполнять следующие задачи:
  - Подготовьте Azure Stack ребра Pro, чтобы запустить модуль GPU.
  - Скачайте и установите пример кода из репозитория Git.
  - Создайте решение и создайте манифест развертывания.
  - Разверните решение на устройстве с Azure Stack пограничной Pro.
  - Мониторинг выходных данных модуля.


## <a name="about-sample-module"></a>Пример модуля "о программе"

Пример модуля GPU в этой статье включает в себя пример кода для ЦП PyTorch и TensorFlow.

## <a name="prerequisites"></a>Предварительные условия

Перед началом работы убедитесь, что у вас есть следующие ресурсы:

- У вас есть доступ к GPU, поддерживающему 1-node Azure Stack пограничным устройством Pro. Это устройство активируется с помощью ресурса в Azure. См. раздел [Активация устройства](azure-stack-edge-gpu-deploy-activate.md).
- Вы настроили вычисление на этом устройстве. Выполните действия, описанные в [руководстве по настройке вычислений на устройстве Azure Stack пограничной Pro](azure-stack-edge-gpu-deploy-configure-compute.md).
- Реестр контейнеров Azure (запись контроля доступа). Перейдите в колонку **ключи доступа** и запишите имя, имя пользователя и пароль для входа в систему контроля учетных записей. Дополнительные сведения см. в [разделе Краткое руководство. создание закрытого реестра контейнеров с помощью портал Azure](../container-registry/container-registry-get-started-portal.md#create-a-container-registry).
- Следующие ресурсы по разработке в клиенте Windows:
    - [Azure CLI 2,0 или более поздней версии](https://aka.ms/installazurecliwindows)
    - [DOCKER CE](https://store.docker.com/editions/community/docker-ce-desktop-windows). Вам может потребоваться учетная запись для скачивания и установки программного обеспечения.
    - [Visual Studio Code](https://code.visualstudio.com/)  
    - [Расширение Azure IOT Edge для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge).    
    - [Расширение Python для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)    
    - [Python 3](https://www.python.org/);    
    - PIP для установки пакетов Python (обычно входит в состав установки Python)

## <a name="get-the-sample-code"></a>Получение примера кода

1. Перейдите к [шаблонам Azure Intelligent ребра в примерах Azure](https://github.com/azure-samples/azure-intelligent-edge-patterns). Клонировать или скачать ZIP-файл для кода. 

    ![Скачать ZIP-файл](media/azure-stack-edge-gpu-deploy-sample-module/download-zip-file-1.png)

    Извлеките файлы из ZIP-файла. Можно также клонировать образцы.

    ```json
    git clone https://github.com/Azure-Samples/azure-intelligent-edge-patterns.git
    ```

## <a name="build-and-deploy-module"></a>Модуль сборки и развертывания

1. Откройте папку **гпуреференцемодулес** в Visual Studio Code.

    ![Открытие Гпуреференцемодулес в VS Code](media/azure-stack-edge-gpu-deploy-sample-module/open-folder-gpu-sample-1.png)

2. Откройте *deployment.template.jsв* и найдите параметры, на которые он ссылается, для реестра контейнеров. В следующем файле используются CONTAINER_REGISTRY_USERNAME, CONTAINER_REGISTRY_PASSWORD и CONTAINER_REGISTRY_NAME.

    ```json
        {
      "$schema-template": "2.0.0",
      "modulesContent": {
        "$edgeAgent": {
          "properties.desired": {
            "schemaVersion": "1.0",
            "runtime": {
              "type": "docker",
              "settings": {
                "minDockerVersion": "v1.25",
                "loggingOptions": "",
                "registryCredentials": {
                  "${CONTAINER_REGISTRY_NAME}":{
                  "username": "$CONTAINER_REGISTRY_USERNAME",
                  "password": "$CONTAINER_REGISTRY_PASSWORD",
                  "address": "${CONTAINER_REGISTRY_NAME}.azurecr.io"
                  }
                }
              }
            },
    ```
3. Создайте новый файл. Заполните значения параметров реестра контейнеров (используйте те, которые определены на предыдущем шаге) следующим образом. 

    ```json
    CONTAINER_REGISTRY_NAME=<YourContainerRegistryName>
    CONTAINER_REGISTRY_USERNAME=<YourContainerRegistryUserName>
    CONTAINER_REGISTRY_PASSWORD=<YourContainerRegistryPassword>
    ```
    Ниже показан пример файла *. env* .
    
    ![Создание и сохранение файла. env](media/azure-stack-edge-gpu-deploy-sample-module/create-save-env-file-1.png)

4. Сохраните файл как *. env* в папке **самплесолутион**

5. Чтобы войти в DOCKER, введите следующую команду в интегрированном терминале Visual Studio Code. 

    ```json
    docker login -u <CONTAINER_REGISTRY_USERNAME> -p <CONTAINER_REGISTRY_PASSWORD> <CONTAINER_REGISTRY_NAME>
    ```
    Перейдите к разделу **ключи доступа** в реестре контейнеров в портал Azure. Скопируйте и используйте имя реестра, пароль и сервер входа.

    ![Ключи доступа в реестре контейнеров](media/azure-stack-edge-gpu-deploy-sample-module/container-registry-access-keys-1.png)

    После того как учетные данные указаны, вход будет выполнен.

    ![Успешный вход](media/azure-stack-edge-gpu-deploy-sample-module/successful-sign-in-1.png)

6. Отправьте образ в реестр контейнеров Azure. В обозревателе VS Code выберите и щелкните правой кнопкой мыши **deployment.template.jsв** файле, а затем выберите пункт **сборка и отправка IOT EDGE решение**. 

    ![Сборка и отправка решения IoT Edge](media/azure-stack-edge-gpu-deploy-sample-module/build-push-iot-edge-solution-1.png)   

    Если расширение Python и Python не установлены, они будут установлены при сборке и отправке решения. Однако это приведет к более длительному времени сборки. 

    После завершения этого шага вы увидите модуль в реестре контейнеров.

    ![Модуль в реестре контейнеров](media/azure-stack-edge-gpu-deploy-sample-module/module-container-registry-1.png)    


7. Чтобы создать манифест развертывания, щелкните правой кнопкой мыши **deployment.template.js** , а затем выберите **создать IOT Edge манифест развертывания**. 

    ![Создание файла манифеста развертывания IoT Edge](media/azure-stack-edge-gpu-deploy-sample-module/generate-iot-edge-deployment-manifest-1.png)  

    Уведомление уведомляет о пути, по которому был создан манифест развертывания. Манифест — это `deployment.amd64.json` файл, созданный в папке **config** . 

8. Выберите **deployment.amd64.jsдля** файла в папке **config** , а затем выберите **Создать развертывание для одного устройства**. Не используйте **deployment.template.jsдля** файла. 

    ![Создание развертывания для одного устройства](media/azure-stack-edge-gpu-deploy-sample-module/create-deployment-single-device-1.png)  

    В окне **вывода** должно отобразиться сообщение о том, что развертывание прошло удачно.

    ![Развертывание в выходных данных прошло](media/azure-stack-edge-gpu-deploy-sample-module/deployment-succeeded-output-1.png) 

## <a name="monitor-the-module"></a>Мониторинг модуля  

1. В палитре команд VS Code выберите команду **Azure IoT Hub: Select IoT Hub** (Центр Интернета вещей Azure: выбрать Центр Интернета вещей).

2. Выберите подписку и Центр Интернета вещей, содержащий устройство IoT Edge, которое нужно настроить. В этом случае выберите подписку, используемую для развертывания устройства Azure Stack пограничной Pro, и выберите устройство IoT Edge, созданное для устройства Azure Stack ребра Pro. Это происходит при настройке вычислений с помощью портал Azure на предыдущих шагах.

3. В обозревателе VS Code разверните раздел центр Интернета вещей Azure. На вкладке **устройства** вы увидите устройство IOT EDGE, соответствующее устройству Azure Stack. Pro. 

    1. Выберите это устройство, щелкните его правой кнопкой мыши и выберите команду **начать мониторинг встроенной конечной точки события**.
  
        ![Запуск мониторинга](media/azure-stack-edge-gpu-deploy-sample-module/monitor-builtin-event-endpoint-1.png)  

    2. Перейдите в раздел **устройства > модули** , и вы должны увидеть, что **модуль GPU** работает.

        ![Модуль в центре Интернета вещей](media/azure-stack-edge-gpu-deploy-sample-module/module-iot-hub-1.png)  

    3. В VS Code терминале также должны отображаться события центра Интернета вещей в качестве выходных данных мониторинга для устройства Azure Stack ребра Pro.

        ![Выходные данные мониторинга](media/azure-stack-edge-gpu-deploy-sample-module/monitor-events-output-1.png) 

        Вы видите, что время, затраченное на выполнение одного и того же набора операций (5000 итераций преобразования фигур) по GPU, гораздо меньше, чем для ЦП.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [настройке GPU для использования модуля](azure-stack-edge-j-series-configure-gpu-modules.md).
