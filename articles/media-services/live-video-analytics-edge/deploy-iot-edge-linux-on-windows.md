---
title: Развертывание в IoT Edge для Linux в Windows — Azure
description: В этой статье содержатся рекомендации по развертыванию в IoT Edge для Linux на устройстве Windows.
ms.topic: how-to
ms.date: 02/18/2021
ms.openlocfilehash: b635070d6a4834da2c6d82cc768684b6cde9cd71
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102042187"
---
# <a name="deploy-to-an-iot-edge-for-linux-on-windows-eflow-device"></a>Развертывание на устройстве IoT Edge для Linux на Windows (ЕФЛОВ)

В этой статье вы узнаете, как развернуть Live Video Analytics на пограничном устройстве с [IOT Edge для Linux в Windows (ефлов)](https://docs.microsoft.com/azure/iot-edge/iot-edge-for-linux-on-windows?view=iotedge-2018-06). После завершения действий, описанных в этом документе, вы сможете запустить [граф мультимедиа](media-graph-concept.md) , который обнаружит движение в видео и выдаст такие события в центр Интернета вещей в облаке. Затем можно переключить граф мультимедиа для расширенных сценариев и перенести возможности интерактивной аналитики видео на устройство IoT Edge под управлением Windows.

## <a name="prerequisites"></a>Предварительные требования 

* Учетная запись Azure с активной подпиской. Если у вас еще нет учетной записи, [ создайте ее бесплатно](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

    > [!NOTE]
    > Вам потребуется подписка Azure с разрешениями на создание субъектов-служб (такие разрешения предоставляет роль **Владелец**). Если у вас нет нужных разрешений, обратитесь к администратору учетной записи для предоставления вам нужных разрешений.
* [Visual Studio Code](https://code.visualstudio.com/) на компьютере для разработки. Убедитесь, что у вас есть [расширение Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* Узнайте [, что такое ефлов](https://aka.ms/AzEFLOW-docs).

## <a name="deployment-steps"></a>Шаги по развертыванию

Ниже показано общее описание потока документа и пять простых шагов, которые необходимо настроить для запуска функции Live Video Analytics на устройстве Windows с ЕФЛОВ:

:::image type="content" source="./media/deploy-iot-edge-linux-on-windows/eflow.png" alt-text="Схема IoT Edge для Linux в Windows (ЕФЛОВ)":::

1. [Установите ефлов](https://aka.ms/AzEFLOW-install) на устройстве Windows. 

    1. Если вы используете компьютер с Windows, то на начальной странице [центра администрирования Windows](https://docs.microsoft.com/windows-server/manage/windows-admin-center/overview) в списке подключений вы увидите подключение к локальному узлу, представляющее компьютер, на котором запущен центр администрирования Windows. 
    1. Все дополнительные серверы, компьютеры или кластеры, которыми вы управляете, также будут отображаться здесь.
    1. Центр администрирования Windows можно использовать для установки Azure ЕФЛОВ на локальном устройстве или удаленных управляемых устройствах и управления им. В этом пошаговом окне Подключение к локальному узлу выдается в качестве целевого устройства для развертывания Azure IoT Edge для Linux в Windows. Таким образом, localhost также отображается как устройство IoT Edge.

    :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/windows-admin-center.png" alt-text="Шаги развертывания — центр администрирования Windows":::
1. Щелкните устройство IoT Edge, чтобы подключиться к нему и открыть вкладку "Обзор" и "Командная оболочка". На вкладке Командная оболочка можно выдавать команды на пограничные устройства.
 
    :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/azure-iot-edge-manager.png" alt-text="Этапы развертывания — Диспетчер Azure IoT Edge":::
1. Перейдите в командную оболочку и введите следующую команду:
    
    ```bash
    bash -c "$(curl -sL https://aka.ms/lva-edge/prep_device)"
    ```

    Модуль Live Video Analytics выполняется на пограничном устройстве с [учетными записями локальных пользователей](deploy-iot-edge-device.md#create-and-use-local-user-account-for-deployment)без привилегий. Кроме того, для хранения данных конфигурации приложения [необходимы определенные локальные папки](deploy-iot-edge-device.md#granting-permissions-to-device-storage) . Наконец, в этом практическом руководством мы используем [симулятор RTSP](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) , который передает видеопоток в реальном времени в модуль лва для анализа. Этот симулятор принимает в качестве входных файлов предварительно записанные видеофайлы из входного каталога. 
    
    Сценарий подготовки к работе с устройством, который использовался ранее, автоматизирует эти задачи, поэтому вы можете выполнить одну команду и все соответствующие папки ввода и настройки, входные видеофайлы и учетные записи пользователей с легко созданными привилегиями. После успешного завершения команды вы увидите следующие папки, созданные на пограничном устройстве. 
    
    * `/home/lvaedgeuser/samples`
    * `/home/lvaedgeuser/samples/input`
    * `/var/lib/azuremediaservices`
    * `/var/media`
    
    Обратите внимание на видеофайлы (*. MKV) в папке/Хоме/лваеджеусер/самплес/инпут, которая выступает в качестве входных файлов для анализа. 
    
    :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/azure-iot-edge-manager-analysis.png" alt-text="Анализ":::
1. Теперь, когда вы настроили пограничное устройство, зарегистрированное в центре и успешно работающее с правильно созданными структурами папок, следующим шагом является настройка следующих дополнительных ресурсов Azure и развертывание модуля лва. 

    * Учетная запись хранения
    * Учетная запись Служб мультимедиа Azure

    Для этого мы рекомендуем использовать [сценарий настройки ресурсов Live Video Analytics](https://github.com/Azure/live-video-analytics/tree/master/edge/setup) для развертывания необходимых ресурсов в подписке Azure. Для этого выполните следующие действия.

    1. Откройте [Azure Cloud Shell](https://ms.portal.azure.com/#cloudshell/).

        :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/azure-resources-cloud-shell.png" alt-text="Cloud Shell":::
    1. Если вы используете Cloud Shell впервые, вам будет предложено выбрать подписку для создания учетной записи хранения и общей папки для службы "Файлы Microsoft Azure". Выберите **Create storage** (Создать хранилище), чтобы создать учетную запись для хранения сведений о сеансах Cloud Shell. Эта учетная запись хранения отличается от учетной записи, которая будет создана сценарием для использования с учетной записью Служб мультимедиа Azure.
    1. В раскрывающемся меню в левой части окна Cloud Shell выберите в качестве среды bash.

        :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/bash.png" alt-text="Среда bash":::
    1. Выполните следующую команду.

        ```bash
        bash -c "$(curl -sL https://aka.ms/lva-edge/setup-resources-for-samples)"
        ```
        
        При появлении запроса на выбор собственного устройства в качестве устройства IoT Edge обязательно выберите **Y** , так как устройство и центр Интернета вещей были созданы ранее. Вам также будет предложено ввести имя центра Интернета вещей и IoT Edge идентификатор устройства. Вы можете получить их, войдя в портал Azure и щелкнув центр Интернета вещей, а затем перейдя к параметру IoT Edge в колонке портала центра Интернета вещей.

        :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/iot-edge-devices.png" alt-text="См. IoT Edge устройства":::

    После завершения можно снова войти в командную оболочку устройства IoT Edge и выполнить следующую команду.
    
    `sudo iotedge list`
    
    Вы должны увидеть следующие четыре модуля, развернутые и работающие на пограничном устройстве. Обратите внимание, что сценарий создания ресурсов развертывает модуль лва вместе с модулями IoT Edge (edgeAgent и edgeHub) и модулем имитатора RTSP для предоставления смоделированного видеоканала RTSP.
    
    :::image type="content" source="./media/deploy-iot-edge-linux-on-windows/running.png" alt-text="Просмотреть состояние":::
1. После развернутых и настроенных модулей вы можете запустить первый граф лва Media на ЕФЛОВ. Вы можете запустить простой граф обнаружения движения, как показано ниже, и визуализировать результаты, выполнив следующие действия.

    :::image type="content" source="./media/analyze-live-video/motion-detection.svg" alt-text="Диаграмма обнаружения движения":::

    1. [Настройте](get-started-detect-motion-emit-events-quickstart.md#configure-the-azure-iot-tools-extension) расширение "средства Azure IOT".
    
        > [!Note]
        > Используйте следующий путь: `~/clouddrive/lva_byod/appsettings.json. - instead of ~/clouddrive/lva-sample/appsettings.json` .
    1. Задайте топологию, создайте экземпляр графа и активируйте его с помощью этих [прямых вызовов методов](get-started-detect-motion-emit-events-quickstart.md#use-direct-method-calls).
    1. [Просмотрите результаты](get-started-detect-motion-emit-events-quickstart.md#observe-results) в центре.
    1. Вызов [методов очистки](get-started-detect-motion-emit-events-quickstart.md#invoke-graphinstancedeactivate).
    1. Удалите ресурсы, если они не требуются.

        > [!IMPORTANT]
        > Неудаленные ресурсы по-прежнему могут быть активны и повлечь за собой расходы Azure.
    
## <a name="next-steps"></a>Дальнейшие действия

* Попробуйте определить движение, а также записать соответствующие видео в облако. Следуйте инструкциям из руководства по [обнаружению движения, записи видеороликов в службах мультимедиа](detect-motion-record-video-clips-media-services-quickstart.md#review-the-sample-video) .
* Запустите [AI в реальном времени](use-your-model-quickstart.md#overview) (можно пропустить установку необходимых компонентов, так как она уже была выполнена выше).
* Используйте наше [расширение VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.live-video-analytics-edge) для просмотра дополнительных графов мультимедиа.
* Используйте [IP-камеру](https://en.wikipedia.org/wiki/IP_camera)  , поддерживающую RTSP, а не используя симулятор RTSP. IP-камеры, поддерживающие протокол RTSP, можно найти на странице [продуктов, соответствующих ONVIF](https://www.onvif.org/conformant-products/). Ищите устройства, которые соответствуют профилям G, S или T.

