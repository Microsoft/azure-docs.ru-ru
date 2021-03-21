---
title: Использование существующих проигрывателей для воспроизведения содержимого в Azure | Документация Майкрософт
description: В этой статье перечислены существующие проигрыватели, которые можно использовать для воспроизведения содержимого.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 7e9fcf89-0fb6-4fa4-96cb-666320684d69
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.openlocfilehash: ce773adace1baf6db8f1b747643ef0ffdc56a97c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103008299"
---
# <a name="playing-your-content-with-existing-players"></a>Воспроизведение содержимого с помощью существующих проигрывателей

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Службы мультимедиа Azure поддерживают многие популярные форматы потоковой передачи, например Smooth Streaming, HTTP Live Streaming и MPEG-Dash. В этой статье описываются имеющиеся проигрыватели, которые можно использовать для тестирования потоков.

## <a name="the-azure-portal-media-services-content-player"></a>Проигрыватель содержимого служб мультимедиа портала Azure

Портал **Azure** предоставляет проигрыватель содержимого, который можно использовать для тестирования видео.

Выберите нужное видео (убедитесь, что оно [опубликовано](media-services-portal-publish.md)) и нажмите кнопку **Воспроизвести** в нижней части портала.

Важные особенности

* **ПРОИГРЫВАТЕЛЬ КОНТЕНТА СЛУЖБ МУЛЬТИМЕДИА** выполняет воспроизведение из конечной точки потоковой передачи по умолчанию. Если требуется воспроизвести из конечной точке потоковой передачи не по умолчанию, используйте другой проигрыватель. Например, [Проигрыватель мультимедиа Azure](https://aka.ms/azuremediaplayer).

### <a name="azure-media-player"></a>Проигрыватель мультимедиа Azure

Используйте [Проигрыватель мультимедиа Azure](https://aka.ms/azuremediaplayer) для воспроизведения содержимого (незашифрованного или защищенного) в любом из следующих форматов:

* Smooth Streaming
* MPEG DASH
* HLS
* Последовательный MP4

### <a name="flash-player"></a>Flash Player

#### <a name="playready-with-token"></a>PlayReady с маркером

[http://reference.dashif.org/dash.js/nightly/samples/dash-if-reference-player/index.html](http://reference.dashif.org/dash.js/nightly/samples/dash-if-reference-player/index.html)

### <a name="dash-players"></a>Проигрыватели DASH

[пунктирный проигрыватель](http://players.akamai.com/players/dashjs)

[https://dashif.org](https://dashif.org)

### <a name="other"></a>Другое

Для проверки URL-адресов HLS также можно использовать:

* **Safari** на устройстве iOS или
* **3ivx HLS Player** в Windows.

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
