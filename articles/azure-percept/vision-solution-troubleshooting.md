---
title: Устранение неполадок, связанных с концепцией и концепциями Azure Перцепт
description: Советы по устранению неполадок для некоторых из наиболее распространенных проблем, возникающих в процессе создания прототипов искусственного интеллекта
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 02/18/2021
ms.custom: template-how-to
ms.openlocfilehash: e7351079e7325aa7750dc0d10f0923bc847ccc3c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101663519"
---
# <a name="vision-solution-troubleshooting"></a>Устранение неполадок решений для визуального распознавания

Сведения об устранении неполадок решений, не связанных с кодом, в Azure Перцепт Studio см. в следующих руководствах.

## <a name="delete-a-vision-project"></a>Удаление проекта концепции

1. Перейдите к https://www.customvision.ai/projects.

1. Наведите указатель мыши на проект, который вы хотите удалить, и щелкните значок корзины, чтобы удалить проект.

    :::image type="content" source="./media/vision-solution-troubleshooting/vision-delete-project.png" alt-text="Страница &quot;проекты&quot; в Пользовательское визуальное распознавание с выделенным значком удаления.":::

## <a name="check-which-modules-are-on-a-device"></a>Проверьте, какие модули находятся на устройстве

1. Перейдите на [портал Microsoft Azure](https://portal.azure.com/?feature.canmodifystamps=true&Microsoft_Azure_Iothub=aduprod&microsoft_azure_marketplace_ItemHideKey=Microsoft_Azure_ADUHidden#home).

1. Щелкните значок **центра Интернета вещей** .

    :::image type="content" source="./media/vision-solution-troubleshooting/vision-iot-hub-2-inline.png" alt-text="Портал Azure домашнюю страницу с выделенным значком центра Интернета вещей." lightbox= "./media/vision-solution-troubleshooting/vision-iot-hub-2.png":::

1. Выберите центр Интернета вещей, к которому подключено целевое устройство.

    :::image type="content" source="./media/vision-solution-troubleshooting/vision-iot-hub.png" alt-text="Список центров Интернета вещей.":::

1. Выберите **IOT Edge** и щелкните свое устройство на вкладке **идентификатор устройства** .

    :::image type="content" source="./media/vision-solution-troubleshooting/vision-iot-edge.png" alt-text="IoT Edge домашней странице.":::

1. Модули устройства будут перечислены на вкладке **модули** .

    :::image type="content" source="./media/vision-solution-troubleshooting/vision-device-modules-inline.png" alt-text="Страница IOT ребра для выбранного устройства, отображающая содержимое вкладки &quot;модули&quot;." lightbox= "./media/vision-solution-troubleshooting/vision-device-modules.png":::

## <a name="delete-a-device"></a>Удаление устройства

1. Перейдите на [портал Microsoft Azure](https://portal.azure.com/?feature.canmodifystamps=true&Microsoft_Azure_Iothub=aduprod&microsoft_azure_marketplace_ItemHideKey=Microsoft_Azure_ADUHidden#home).

1. Щелкните значок **центра Интернета вещей** .

1. Выберите центр Интернета вещей, к которому подключено целевое устройство.

1. Выберите **IOT Edge** и установите флажок рядом с идентификатором целевого устройства. Щелкните значок корзины, чтобы удалить устройство.

    :::image type="content" source="./media/vision-solution-troubleshooting/vision-delete-device.png" alt-text="Значок удаления, выделенный на домашней странице IOT.":::

## <a name="eye-module-troubleshooting-tips"></a>Советы по устранению неполадок модуля глаза

Если возникла проблема с **вебстреаммодуле**, убедитесь, что **азуриемодуле**, который выполняет выборку концепции модели, работает. Чтобы проверить состояние среды выполнения, перейдите к [портал Azure](https://portal.azure.com/?feature.canmodifystamps=true&Microsoft_Azure_Iothub=aduprod&microsoft_azure_marketplace_ItemHideKey=Microsoft_Azure_ADUHidden#home) и перейдите ко **всем ресурсам**  ->  **\<your IoT hub>**  ->  **IOT Edge**  ->  **\<your device ID>** . Перейдите на вкладку **модули** , чтобы просмотреть состояние среды выполнения всех установленных модулей.

:::image type="content" source="./media/vision-solution-troubleshooting/over-the-air-iot-edge-device-page-inline.png" alt-text="Экран состояния среды выполнения модуля устройства." lightbox= "./media/vision-solution-troubleshooting/over-the-air-iot-edge-device-page.png":::

Если состояние среды выполнения **азуриемодуле** не указано как **выполняемое**, щелкните **задать модули**  ->  **азуриемодуле**. На странице **Параметры модуля** задайте для параметра **требуемое состояние** значение **работает** и нажмите кнопку **Обновить**.

 :::image type="content" source="./media/vision-solution-troubleshooting/firmware-desired-status-stopped.png" alt-text="Экран настройки параметров модуля.":::

## <a name="view-device-rtsp-video-stream"></a>Просмотр видеопотока для устройства RTSP

Просмотр видеопотока RTSP на устройстве в [Azure Перцепт Studio](./how-to-view-video-stream.md) или [Влк Media Player](https://www.videolan.org/vlc/index.html).

Чтобы открыть поток RTSP в проигрывателе Влк Media Player, перейдите на страницу **мультимедиа**  ->  **Открытый сетевой поток**  ->  **RTSP://[IP-адрес устройства]/ресулт**.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об устранении неполадок в Azure Перцепт DK см. в разделе [Общее руководство по устранению неполадок](./troubleshoot-dev-kit.md) .