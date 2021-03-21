---
title: Развертывание приложения без отслеживания состояния Kubernetes на Azure Stack пограничных графических процессоров Pro с помощью модуля IoT Edge | Документация Майкрософт
description: В этой статье описывается, как развернуть приложение без отслеживания состояния Kubernetes на устройстве с Azure Stack ребра Pro GPU с помощью модуля IoT Edge, доступ к которому осуществляется через внешний IP-адрес.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: 4eda6eaf69fad6f8600651660ae4ac6223fe8f8c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102438069"
---
# <a name="use-iot-edge-module-to-run-a-kubernetes-stateless-application-on-your-azure-stack-edge-pro-gpu-device"></a>Использование модуля IoT Edge для запуска приложения без отслеживания состояния Kubernetes на устройстве Azure Stack ребра Pro GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этой статье описывается, как можно использовать модуль IoT Edge для развертывания приложения без отслеживания состояния на устройстве Azure Stack. Pro.

Чтобы развернуть приложение без отслеживания состояния, выполните следующие действия.

- Перед развертыванием модуля IoT Edge убедитесь, что предварительные условия выполнены.
- Добавьте модуль IoT Edge для доступа к сети вычислений на Azure Stack крае Pro.
- Убедитесь, что модуль может получить доступ к включенному сетевому интерфейсу.

В этом пошаговом руководстве для демонстрации сценария используется модуль приложения на основе сервера.

## <a name="prerequisites"></a>Предварительные условия

Прежде чем начать, вам потребуется:

- Устройство Azure Stack пограничной Pro. Убедитесь, что выполнены следующие условия:

    - На устройстве настроены параметры вычислений сети.
    - Устройство активируется в соответствии с инструкциями в разделе [учебник. Активация устройства](azure-stack-edge-gpu-deploy-activate.md).
- Вы завершили **настройку шага вычислений** в соответствии с [руководством: Настройка вычислений на устройстве Azure Stack пограничной Pro](azure-stack-edge-gpu-deploy-configure-compute.md) на устройстве. Устройство должно иметь связанный ресурс центра Интернета вещей, устройство IoT и устройство IoT Edge.


## <a name="add-webserver-app-module"></a>Добавление модуля приложения для сервера приложений

Чтобы добавить модуль приложения веб-сервера на устройство Azure Stack погранично Pro, выполните следующие действия.

1. В ресурсе центра Интернета вещей, связанном с устройством, перейдите в раздел **Автоматическое управление устройствами > IOT Edge**.
1. Выберите и щелкните устройство IoT Edge, связанное с устройством Azure Stack ребра Pro. 

    ![Выбор устройства IoT Edge](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/select-iot-edge-device-1.png)  

1. Щелкните **Set modules** (Настроить модули). На **устройстве Set modules On Device (Установка модулей на устройство**) выберите **+ Добавить** , а затем выберите **модуль IOT Edge**.

    ![Выбор модуля IoT Edge](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/select-iot-edge-module-1.png)

1. В **модуле добавить IOT Edge**:

    1. Укажите **имя** модуля приложения веб-сервера, который требуется развернуть.
    2. На вкладке **Параметры модуля** укажите **URI образа** для вашего образа модуля. Извлекается модуль, соответствующий предоставленному имени и тегам. В этом случае `nginx:stable` будет извлекать стабильный образ nginx (помечается как стабильный) из общедоступного [репозитория DOCKER](https://hub.docker.com/_/nginx/).

        ![Добавить модуль IoT Edge](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/set-module-settings-1.png)    

    3. На вкладке **Параметры создания контейнера** вставьте следующий пример кода:  

        ```
        {
            "HostConfig": {
                "PortBindings": {
                    "80/tcp": [
                        {
                            "HostPort": "8080"
                        }
                    ]
                }
            }
        }
        ```

        Эта конфигурация позволяет получить доступ к модулю с помощью сетевого IP-адреса вычислений по протоколу *http* через TCP-порт 8080 (с портом сервера по умолчанию 80). Выберите **Добавить**.

        ![Указание сведений о порте в IoT Edge пользовательской колонке модуля](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/verify-module-status-1.png)

    4. Выберите **Review + create** (Просмотреть и создать). Проверьте сведения о модуле и нажмите кнопку **создать**.

## <a name="verify-module-access"></a>Проверка доступа к модулю

1. Убедитесь, что модуль успешно развернут и запущен. На вкладке **модули** состояние среды выполнения модуля должно быть **работает**.  

    ![Проверка состояния модуля выполняется](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/verify-module-status-1.png)

1. Чтобы получить внешнюю конечную точку приложения "сервер", скачайте [панель мониторинга Kubernetes](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md#access-dashboard). 
1. В левой области панели мониторинга отфильтруйте по пространству имен **iotedge** . Перейдите в раздел **Обнаружение и балансировка нагрузки > служб**. В списке служб выберите внешнюю конечную точку для модуля приложения для служб. 

    ![Подключение к приложению внешнего сервера во внешней конечной точке](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/connect-external-endpoint-1.png)

1. Выберите внешнюю конечную точку, чтобы открыть новое окно браузера.

    Вы должны увидеть, что приложение-сервер работает.

    ![Проверка подключения к модулю через указанный порт](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/verify-webserver-app-1.png)

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как предоставить приложение с отслеживанием состояния через модуль IoT Edge<!--insert link-->.
