---
title: Запись видео на пограничные устройства при обнаружении движения — Azure
description: В этом кратком руководстве описано, как использовать Аналитику видеотрансляции в IoT Edge для анализа видеопотока, поступающего с (имитированной ) IP-камеры, определять наличие движений и записывать видеоролики MP4 в локальную файловую систему на пограничном устройстве.
ms.topic: quickstart
ms.date: 04/27/2020
zone_pivot_groups: ams-lva-edge-programming-languages
ms.openlocfilehash: e4334cdd14242337a2a870f31886606020654685
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96498359"
---
# <a name="quickstart-detect-motion-and-record-video-on-edge-devices"></a>Краткое руководство. Запись видео на пограничные устройства при обнаружении движения
 
В этом кратком руководстве показано, как использовать Аналитику видеотрансляций в службе IoT Edge для анализа потока видеотрансляции из (имитированной) IP-камеры. Здесь показано, как определять наличие движения, и если так, записывать MP4-видеофайл в локальную файловую систему пограничного устройства. В этом кратком руководстве используется имитация видеотрансляции, а в качестве устройства IoT Edge используется виртуальная машина Azure. 

::: zone pivot="programming-language-csharp"
[!INCLUDE [header](includes/detect-motion-record-video-clips-edge-devices-quickstart/csharp/header.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [header](includes/detect-motion-record-video-clips-edge-devices-quickstart/python/header.md)]
::: zone-end

## <a name="prerequisites"></a>Предварительные требования

::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/detect-motion-record-video-clips-edge-devices-quickstart/csharp/prerequisites.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prerequisites](includes/detect-motion-record-video-clips-edge-devices-quickstart/python/prerequisites.md)]
::: zone-end

## <a name="review-the-sample-video"></a>Просмотр примера видео

::: zone pivot="programming-language-csharp"
[!INCLUDE [review-sample-video](includes/detect-motion-record-video-clips-edge-devices-quickstart/csharp/review-sample-video.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [review-sample-video](includes/detect-motion-record-video-clips-edge-devices-quickstart/python/review-sample-video.md)]
::: zone-end

## <a name="overview"></a>Обзор

::: zone pivot="programming-language-csharp"
[!INCLUDE [overview](includes/detect-motion-record-video-clips-edge-devices-quickstart/csharp/overview.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [overview](includes/detect-motion-record-video-clips-edge-devices-quickstart/python/overview.md)]
::: zone-end

## <a name="examine-and-edit-the-sample-files"></a>Изучение и изменение образцов файлов

::: zone pivot="programming-language-csharp"
[!INCLUDE [examine-edit-sample-files](includes/detect-motion-record-video-clips-edge-devices-quickstart/csharp/examine-edit-sample-files.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [examine-edit-sample-files](includes/detect-motion-record-video-clips-edge-devices-quickstart/python/examine-edit-sample-files.md)]
::: zone-end

## <a name="review---check-the-modules-status"></a>Проверка. Оценка состояния модулей

::: zone pivot="programming-language-csharp"
[!INCLUDE [check-modules-status](includes/detect-motion-record-video-clips-edge-devices-quickstart/csharp/check-modules-status.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [check-modules-status](includes/detect-motion-record-video-clips-edge-devices-quickstart/python/check-modules-status.md)]
::: zone-end

## <a name="review---prepare-for-monitoring-events"></a>Проверка. Подготовка к мониторингу событий

::: zone pivot="programming-language-csharp"
[!INCLUDE [prepare-monitoring-events](includes/detect-motion-record-video-clips-edge-devices-quickstart/csharp/prepare-monitoring-events.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prepare-monitoring-events](includes/detect-motion-record-video-clips-edge-devices-quickstart/python/prepare-monitoring-events.md)]
::: zone-end

## <a name="run-the-sample-program"></a>Запуск примера программы

::: zone pivot="programming-language-csharp"
[!INCLUDE [run-sample-program](includes/detect-motion-record-video-clips-edge-devices-quickstart/csharp/run-sample-program.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [run-sample-program](includes/detect-motion-record-video-clips-edge-devices-quickstart/python/run-sample-program.md)]
::: zone-end

## <a name="interpret-results"></a>Интерпретация результатов 

::: zone pivot="programming-language-csharp"
[!INCLUDE [interpret-results](includes/detect-motion-record-video-clips-edge-devices-quickstart/csharp/interpret-results.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [interpret-results](includes/detect-motion-record-video-clips-edge-devices-quickstart/python/interpret-results.md)]
::: zone-end

## <a name="play-the-mp4-clip"></a>Воспроизведение MP4-файла

::: zone pivot="programming-language-csharp"
[!INCLUDE [play-mp4-clip](includes/detect-motion-record-video-clips-edge-devices-quickstart/csharp/play-mp4-clip.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [play-mp4-clip](includes/detect-motion-record-video-clips-edge-devices-quickstart/python/play-mp4-clip.md)]
::: zone-end

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете работать с другими краткими руководствами, сохраните созданные ресурсы. В противном случае на портале Azure перейдите к группам ресурсов, выберите группу ресурсов, с которой вы работали в этом кратком руководстве, а затем удалите все ресурсы.

## <a name="next-steps"></a>Дальнейшие действия

* Следуйте краткому руководству [Запуск Аналитики видеотрансляции с помощью собственной модели](use-your-model-quickstart.md), чтобы узнать, как применить ИИ к каналам видеотрансляций.
* Ознакомьтесь с дополнительными задачами для опытных пользователей.

    * Используйте [IP-камеру](https://en.wikipedia.org/wiki/IP_camera), которая поддерживает протокол RTSP вместо симулятора RTSP. IP-камеры, поддерживающие протокол RTSP, можно найти на странице [продуктов, соответствующих ONVIF](https://www.onvif.org/conformant-products). Ищите устройства, которые соответствуют профилям G, S или T.
    * Используйте устройство с архитектурой AMD64 или x64 вместо использования виртуальной машины Linux в Azure. Устройство должно находиться в той же сети, что и IP-камера. Выполните инструкции из статьи [Install Azure IoT Edge runtime on Linux](../../iot-edge/how-to-install-iot-edge.md) (Установка среды выполнения Azure IoT Edge в системах Linux на основе Debian). Потом следуйте инструкциям краткого руководства [Развертывание первого модуля IoT Edge на виртуальном устройстве Linux](../../iot-edge/quickstart-linux.md), чтобы зарегистрировать устройство в Центре Интернета вещей Azure.