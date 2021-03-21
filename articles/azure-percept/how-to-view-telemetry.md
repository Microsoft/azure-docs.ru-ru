---
title: Просмотр данных телеметрии для определения модели в Azure Перцепт DK
description: Узнайте, как просмотреть данные телеметрии определения концепции в Azure Перцепт DK в Azure IoT Explorer
author: elqu20
ms.author: v-elqu
ms.service: azure-percept
ms.topic: how-to
ms.date: 02/17/2021
ms.custom: template-how-to
ms.openlocfilehash: 07abbc5f5e85c75b73774d11b6b81dd2085735b7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102095329"
---
# <a name="view-your-azure-percept-dks-model-inference-telemetry"></a>Просмотр данных телеметрии для определения модели в Azure Перцепт DK

Следуйте указаниям в этом разделе, чтобы просмотреть данные телеметрии определения концепции в Azure Перцепт DK в Azure [IOT Explorer](https://github.com/Azure/azure-iot-explorer/releases).

## <a name="prerequisites"></a>Предварительные требования

- Azure Percept DK (DevKit).
- [Подписка Azure.](https://azure.microsoft.com/free/)
- [Выполненная настройка Azure Percept DK](./quickstart-percept-dk-set-up.md). Предполагается, что вы уже подключили DevKit к сети Wi-Fi, создали центр Интернета вещей и подключили к нему DevKit.
- [Модель искусственного интеллекта развернута в Azure Перцепт DK](./how-to-deploy-model.md)

## <a name="view-telemetry"></a>Просмотр телеметрии

1. Включите свой DevKit.

1. Скачайте и установите [Azure IOT Explorer](https://github.com/Azure/azure-iot-explorer/releases). Если вы являетесь пользователем Windows, выберите MSI-файл.

    :::image type="content" source="./media/how-to-view-telemetry/azure-iot-explorer-download.png" alt-text="Экран загрузки для Azure IoT Explorer.":::

1. Подключение центра Интернета вещей к Azure IoT Explorer:

    1. Перейдите на [портал Microsoft Azure](https://portal.azure.com).

    1. Щелкните **Все ресурсы**.

        :::image type="content" source="./media/how-to-view-telemetry/azure-portal.png" alt-text="Домашняя страница портала Azure.":::

    1. Выберите центр Интернета вещей, к которому подключена служба Azure Перцепт DK.

        :::image type="content" source="./media/how-to-view-telemetry/iot-hub.png" alt-text="Список центра Интернета вещей в портал Azure.":::

    1. В левой части страницы центра Интернета вещей выберите **политики общего доступа**.

        :::image type="content" source="./media/how-to-view-telemetry/shared-access-policies.png" alt-text="Страница центра Интернета вещей с политиками общего доступа.":::

    1. Щелкните **iothubowner**.

        :::image type="content" source="./media/how-to-view-telemetry/iothubowner.png" alt-text="Экран политик общего доступа с выделенным iothubowner.":::

    1. Щелкните синий значок копирования рядом с полем **строка подключения — первичный ключ**.

        :::image type="content" source="./media/how-to-view-telemetry/connection-string.png" alt-text="окно iothubowner с выделенной кнопкой копирования строки подключения.":::

    1. Откройте Azure IoT Explorer и щелкните **+ Добавить подключение**.

    1. Вставьте строку подключения в поле **строка подключения** в окне **Добавление строки подключения** и нажмите кнопку **сохранить**.

        :::image type="content" source="./media/how-to-view-telemetry/add-connection-string.png" alt-text="Окно Azure IOT Explorer с полем для вставления строки подключения в.":::

    1. Наведите на объект концепцию концепции в объекте, чтобы присвоить значение модели.

    1. Выберите данные **телеметрии**.

    1. Нажмите кнопку **Пуск** , чтобы просмотреть события телеметрии с устройства.

## <a name="next-steps"></a>Дальнейшие действия
Узнайте, как просмотреть [поток видео Azure ПЕРЦЕПТ DK](./how-to-view-video-stream.md).