---
title: Анализ видеотрансляции с помощью собственной модели gRPC — Azure
description: В этом кратком руководстве вы примените компьютерное зрение для анализа потока видеотрансляции из IP-камеры (имитированной).
ms.topic: quickstart
ms.date: 08/14/2020
zone_pivot_groups: ams-lva-edge-programming-languages
ms.openlocfilehash: 907d402a4b0ef5d5e437322d02431a0ccd2d8da3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "89421543"
---
# <a name="quickstart-analyze-live-video-by-using-your-own-grpc-model"></a>Краткое руководство. Анализ видеотрансляции с помощью собственной модели gRPC

В этом кратком руководстве показано, как использовать Аналитику видеотрансляций в службе IoT Edge для анализа потока видеотрансляции из (имитированной) IP-камеры. Вы узнаете, как применить модель компьютерного зрения для обнаружения объектов. Подмножество кадров в потоке видео в реальном времени отправляется в службу вывода. Результаты отправляются в центр IoT Edge.

::: zone pivot="programming-language-csharp"
[!INCLUDE [header](includes/analyze-live-video-your-grpc-model-quickstart/csharp/header.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [header](includes/analyze-live-video-your-grpc-model-quickstart/python/header.md)]
::: zone-end

## <a name="prerequisites"></a>Предварительные требования

::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/analyze-live-video-your-grpc-model-quickstart/csharp/prerequisites.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prerequisites](includes/analyze-live-video-your-grpc-model-quickstart/python/prerequisites.md)]
::: zone-end

## <a name="review-the-sample-video"></a>Просмотр примера видео

::: zone pivot="programming-language-csharp"
[!INCLUDE [review sample video](includes/analyze-live-video-your-grpc-model-quickstart/csharp/review-sample-video.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [review sample video](includes/analyze-live-video-your-grpc-model-quickstart/python/review-sample-video.md)]
::: zone-end

## <a name="overview"></a>Обзор

::: zone pivot="programming-language-csharp"
[!INCLUDE [overview](includes/analyze-live-video-your-grpc-model-quickstart/csharp/overview.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [overview](includes/analyze-live-video-your-grpc-model-quickstart/python/overview.md)]
::: zone-end

## <a name="create-and-deploy-the-media-graph"></a>Создание и развертывание графа мультимедиа

::: zone pivot="programming-language-csharp"
[!INCLUDE [create and deploy the media graph](includes/analyze-live-video-your-grpc-model-quickstart/csharp/create-deploy-media-graph.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [create and deploy the media graph](includes/analyze-live-video-your-grpc-model-quickstart/python/create-deploy-media-graph.md)]
::: zone-end

## <a name="interpret-results"></a>Интерпретация результатов

::: zone pivot="programming-language-csharp"
[!INCLUDE [interpret results](includes/analyze-live-video-your-grpc-model-quickstart/csharp/interpret-results.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [interpret results](includes/analyze-live-video-your-grpc-model-quickstart/python/interpret-results.md)]
::: zone-end

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете работать с другими краткими руководствами, сохраните созданные ресурсы. В противном случае на портале Azure перейдите к группам ресурсов, выберите группу ресурсов, с которой вы работали в этом кратком руководстве, и удалите все ресурсы.

## <a name="next-steps"></a>Дальнейшие действия

* Попробуйте выполнить различные топологии графа мультимедиа с помощью протокола gRPC.
* **Создание и запуск примеров расширений службы "Аналитика видеотрансляций" (LVA)**
<br/>Ознакомьтесь с примерами записных книжек Jupyter, которые позволяют создавать и запускать модели YOLO на базе [ONNX](http://onnx.ai/) в качестве расширения службы "Аналитика видеотрансляций".
    * [Пример модели YOLOv3](https://github.com/Azure/live-video-analytics/tree/master/utilities/video-analysis/notebooks/Yolo/yolov3/yolov3-grpc-icpu-onnx/readme.md)
    * [Пример модели YOLOv4](https://github.com/Azure/live-video-analytics/blob/master/utilities/video-analysis/notebooks/Yolo/yolov4/yolov4-grpc-icpu-onnx/readme.md)

