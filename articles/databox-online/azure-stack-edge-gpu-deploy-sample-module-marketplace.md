---
title: Развертывание модуля GPU на устройстве Microsoft Azure Stack ребра Pro из Azure Marketplace | Документация Майкрософт
description: В этой статье описывается, как развернуть модуль IoT с поддержкой GPU на устройстве Azure Stack ребра Pro GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: e6a58a1f404228564e4e1ac6d77051a747debaa7
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102438168"
---
# <a name="deploy-a-gpu-enabled-iot-module-from-azure-marketplace-on-azure-stack-edge-pro-gpu-device"></a>Развертывание модуля IoT с поддержкой GPU из Azure Marketplace на устройстве Azure Stack с помощью GPU Pro

[!INCLUDE [applies-to-GPU-and-pro-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-sku.md)]

В этой статье описывается, как развернуть модуль обработки графических процессоров (GPU) с поддержкой IoT Edge Module из Azure Marketplace на устройстве Azure Stack. Pro. 

Вы узнаете, как выполнять следующие задачи:
  - Подготовьте Azure Stack ребра Pro, чтобы запустить модуль GPU.
  - Скачайте и разверните модуль IoT с поддержкой GPU из Azure Marketplace.
  - Мониторинг выходных данных модуля.

## <a name="about-sample-module"></a>Пример модуля "о программе"

Пример модуля GPU в этой статье включает в себя пример кода для ЦП PyTorch и TensorFlow.

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы убедитесь, что у вас есть следующие ресурсы:

- У вас есть доступ к GPU, поддерживающему 1-node Azure Stack пограничном устройстве. Это устройство активируется с помощью ресурса в Azure. 
- Вы настроили вычисление на этом устройстве. Выполните действия, описанные в [руководстве по настройке вычислений на Azure Stack пограничном устройстве](azure-stack-edge-gpu-deploy-configure-compute.md).
- Следующие ресурсы по разработке в клиенте Windows:
    - [Visual Studio Code](https://code.visualstudio.com/)  
    - [Расширение Azure IOT Edge для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge).   


## <a name="get-module-from-azure-marketplace"></a>Получение модуля из Azure Marketplace

1. Просмотр всех [приложений в Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps).

    ![Обзор приложений в Azure Marketplace](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/browse-apps-marketplace-1.png)

2. Поиск по **началу работы с графическими процессорами**.

    ![Пример модуля поиска GPU](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/search-gpu-sample-module-1.png)

3. Выберите **Get it now** (Получить).

    ![Получение примера модуля](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/get-sample-module-1.png)

4. Нажмите кнопку **продолжить** , чтобы подтвердить условия использования и политику конфиденциальности поставщика. 

    ![Получение примера модуля 2](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/terms-of-use-1.png)

5. Выберите подписку, которая использовалась для развертывания устройства Azure Stack ребра Pro.

    ![Выбор подписки](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/select-subscription-1.png)

6. Введите имя службы центра Интернета вещей, созданной при настройке устройства Azure Stack ребра Pro. Чтобы найти это имя службы центра Интернета вещей, перейдите к ресурсу Azure Stack ребра, связанному с устройством, в портал Azure. 

    1. В области слева выберите пункт **службы пограничных служб > IOT Edge**. 

        ![Просмотреть конфигурацию вычислений](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/view-config-1.png)

    1. Перейдите в раздел **Свойства**. 

        1. Запишите службу центра Интернета вещей, созданную при настройке вычислений на устройстве Azure Stack ребра Pro.
        2. Обратите внимание на имя устройства IoT Edge, которое было создано при настройке вычислений. Это имя будет использоваться на следующем шаге.

        ![Конфигурация вычислений ребра](media/azure-stack-edge-gpu-deploy-sample-module/view-compute-config-1.png)

10. Выберите **Развертывание на устройстве**.

11. Введите имя устройства IoT Edge или выберите **найти устройство** , чтобы просмотреть устройства, зарегистрированные в центре.

    ![Найти устройство](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/find-device-1.png)

12. Выберите **создать** , чтобы продолжить стандартный процесс настройки манифеста развертывания, включая добавление других модулей при необходимости. Сведения новых модулей, например URI образа, параметры создания или требуемые свойства, предопределены, но их можно изменить.

    ![Выбор кнопки "Создать"](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/target-devices-iot-edge-module-1.png)


13. Убедитесь, что модуль развернут в центре Интернета вещей в портал Azure. Выберите устройство, щелкните **задать модули** , и модуль должен быть указан в разделе **модули IOT Edge** .

    ![Выберите создать 2.](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/running-module-iotres-1.png)

## <a name="monitor-the-module"></a>Мониторинг модуля  

1. В палитре команд VS Code выберите команду **Azure IoT Hub: Select IoT Hub** (Центр Интернета вещей Azure: выбрать Центр Интернета вещей).

2. Выберите подписку и Центр Интернета вещей, содержащий устройство IoT Edge, которое нужно настроить. В этом случае выберите подписку, используемую для развертывания устройства Azure Stack пограничной Pro, и выберите устройство IoT Edge, созданное для устройства Azure Stack ребра Pro. Это происходит при настройке вычислений с помощью портал Azure на предыдущих шагах.

3. В обозревателе VS Code разверните раздел центр Интернета вещей Azure. На вкладке **устройства** вы увидите устройство IOT EDGE, соответствующее устройству Azure Stack. Pro. 

    1. Выберите это устройство, щелкните его правой кнопкой мыши и выберите команду **начать мониторинг встроенной конечной точки события**.
  
        ![Запуск мониторинга](media/azure-stack-edge-gpu-deploy-sample-module/monitor-builtin-event-endpoint-1.png)  

    2. Перейдите в раздел **устройства > модули** , и вы должны увидеть, что **модуль GPU** работает.

    3. В VS Code терминале также должны отображаться события центра Интернета вещей в качестве выходных данных мониторинга для устройства Azure Stack ребра Pro.

        ![Выходные данные мониторинга](media/azure-stack-edge-gpu-deploy-sample-module/monitor-events-output-1.png) 

        Вы видите, что время, затраченное на выполнение одного и того же набора операций (5000 итераций преобразования фигур) по GPU, гораздо меньше, чем для ЦП.

## <a name="next-steps"></a>Next Steps

- Дополнительные сведения о [настройке GPU для использования модуля](azure-stack-edge-j-series-configure-gpu-modules.md).
