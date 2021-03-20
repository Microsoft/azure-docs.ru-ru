---
title: Просмотр видеопотока RTSP в Azure Перцепт DK
description: Узнайте, как просмотреть поток видео RTSP из Azure Перцепт DK.
author: elqu20
ms.author: v-elqu
ms.service: azure-percept
ms.topic: how-to
ms.date: 02/12/2021
ms.custom: template-how-to
ms.openlocfilehash: 5f77e99dc5c34867fef2b0ac47c709824fa4477d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102096009"
---
# <a name="view-your-azure-percept-dks-rtsp-video-stream"></a>Просмотр видеопотока RTSP в Azure Перцепт DK

Следуйте этому руководству, чтобы просмотреть поток видео RTSP из Azure Перцепт DK в Azure Перцепт Studio. При этом в веб-потоке будет отображаться представление о моделях AI, развернутых на устройстве.

## <a name="prerequisites"></a>Предварительные требования

- Azure Percept DK (DevKit).
- [Подписка Azure.](https://azure.microsoft.com/free/)
- [Выполненная настройка Azure Percept DK](./quickstart-percept-dk-set-up.md). Предполагается, что вы уже подключили DevKit к сети Wi-Fi, создали центр Интернета вещей и подключили к нему DevKit.

## <a name="view-the-rtsp-video-stream"></a>Просмотр видеопотока RTSP

1. Включите свой DevKit.

1. Перейдите в [Azure Percept Studio](https://go.microsoft.com/fwlink/?linkid=2135819).

1. В левой части страницы Обзор щелкните **устройства**.

    :::image type="content" source="./media/how-to-view-video-stream/overview-devices-inline.png" alt-text="Экран обзора Azure Percept Studio." lightbox="./media/how-to-view-video-stream/overview-devices.png":::

1. Выберите DevKit из списка.

    :::image type="content" source="./media/how-to-view-video-stream/select-device.png" alt-text="Снимок экрана доступных устройств в Azure Перцепт Studio.":::

1. Щелкните **Просмотр потока устройства**.

    :::image type="content" source="./media/how-to-view-video-stream/view-device-stream.png" alt-text="Снимок экрана со страницей &quot;устройство&quot;, на которой показаны доступные действия по проекту &quot;концепция&quot;.":::

    Откроется отдельная вкладка, показывающая динамический веб-поток из Azure Перцепт DK.

    :::image type="content" source="./media/how-to-view-video-stream/webstream.png" alt-text="Снимок экрана веб-потока устройства.":::

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как просмотреть данные [телеметрии Azure ПЕРЦЕПТ DK](./how-to-view-telemetry.md).