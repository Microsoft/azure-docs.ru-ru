---
title: Обнаружение движения объектов и движений в Azure IoT Central Video Analytics | Документация Майкрософт
description: Узнайте, как создать приложение IoT Central с помощью шаблона приложения "аналитика видео — объект и приложение обнаружения движения" в IoT Central. Этот шаблон использует Live Video Analytics и подключенные камеры.
author: KishorIoT
ms.author: nandab
ms.date: 07/27/2020
ms.topic: conceptual
ms.service: iot-central
ms.subservice: iot-central-retail
services: iot-central
ms.openlocfilehash: 48808f762536390287bae40e8af3849da20b81c2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91874294"
---
# <a name="video-analytics---object-and-motion-detection-application-architecture"></a>Аналитика видео — архитектура приложения для обнаружения объектов и движения

Шаблон приложения " **аналитика видео — объект и приложение обнаружения движения** " позволяет создавать решения IOT, включая возможности Live Video Analytics.

:::image type="content" source="media/architecture-video-analytics/architecture.png" alt-text="Схема обзора объектов и компонентов обнаружения движения видео.":::

Ключевые компоненты решения "аналитика видео" включают:

## <a name="live-video-analytics-lva"></a>Интерактивная аналитика видео (лва)

ЛВА предоставляет платформу для создания интеллектуальных видеороликов, охватывающих пограничные и облачные приложения. Платформа позволяет создавать интеллектуальные видеоролики, охватывающие пограничные и облачные приложения. Платформа предоставляет возможность сканирования, записи, анализа видеотрансляций, а также публикации результатов (видео или видеоаналитики) в службах Azure. Службы Azure могут работать в облаке или на пограничных устройствах. Эта платформа позволяет повысить эффективность ваших решений для Интернета вещей за счет анализа видео.

Дополнительные сведения см. в статье [Live Video Analytics](https://github.com/Azure/live-video-analytics) на сайте GitHub.

## <a name="iot-edge-lva-gateway-module"></a>Модуль шлюза IoT Edge лва

Модуль IoT Edge лва Gateway создает экземпляры камер как новые устройства и подключает их непосредственно к IoT Central с помощью клиентского пакета SDK для устройства IoT.

В этой эталонной реализации устройства подключаются к решению с помощью симметричных ключей из пограничных. Дополнительные сведения о подключении устройств см. [в статье подключение к Azure IOT Central](../core/concepts-get-connected.md) .

## <a name="media-graph"></a>Граф мультимедиа

Media Graph позволяет определить, откуда следует записывать носитель, как обработать его и куда доставлять результаты. Вы настраиваете Media Graph, подключаясь к компонентам или узлам нужным образом. Дополнительные сведения см. в статье [Media Graph](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph) на GitHub.

## <a name="next-steps"></a>Дальнейшие действия

На следующем шаге мы научитесь [развертывать приложение IOT Central с помощью шаблона приложения Video Analytics — объект и приложение обнаружения движения](tutorial-video-analytics-deploy.md).
