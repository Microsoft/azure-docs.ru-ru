---
title: Общие сведения о проигрывателях мультимедиа для Служб мультимедиа
description: Какие проигрыватели мультимедиа можно использовать со Службами мультимедиа Azure? На сегодняшний день можно использовать Проигрыватель мультимедиа Azure, Shaka и Video.js.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 3/08/2021
ms.openlocfilehash: 47ef56a770ca9965cef3e50630be4b69342a038d
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105559865"
---
# <a name="media-players-for-media-services"></a>Проигрыватели мультимедиа для Служб мультимедиа

Для Служб мультимедиа можно использовать несколько проигрывателей мультимедиа.

## <a name="azure-media-player"></a>Проигрыватель мультимедиа Azure

Проигрыватель мультимедиа Azure — это видеопроигрыватель, который можно использовать для широкого спектра браузеров и устройств. Проигрыватель мультимедиа Azure использует такие отраслевые стандарты, как HTML5, Media Source Extensions (MSE) и Encrypted Media Extensions (EME), для оптимальной адаптивной потоковой передачи. Если эти стандарты недоступны на устройстве или в браузере, Проигрыватель мультимедиа Azure использует Flash и Silverlight. Независимо от используемой технологии воспроизведения разработчики получают единый интерфейс JavaScript для доступа к интерфейсам API. Это позволяет службам мультимедиа Azure обслуживать содержимое для воспроизведения на широком спектре устройств и в различных браузерах без дополнительных усилий.

См. [документацию по Проигрывателю мультимедиа Azure](../azure-media-player/azure-media-player-overview.md).

## <a name="shaka"></a>Shaka

Проигрыватель Shaka — это библиотека JavaScript с открытым исходным кодом для адаптивных мультимедиа. Он воспроизводит в браузере адаптивные форматы мультимедиа (например, DASH и HLS) без использования подключаемых модулей или Flash. Вместо этого проигрыватель Shaka использует открытые веб-стандарты Media Source Extensions и Encrypted Media Extensions.

См. статью [Как использовать проигрыватель Shaka с помощью Служб мультимедиа Azure](how-to-shaka-player.md).

## <a name="videojs"></a>Video.js

Video.js — это видеопроигрыватель, который воспроизводит в браузере адаптивные форматы мультимедиа (например, DASH и HLS). Video.js использует открытые веб-стандарты Media Source Extensions и Encrypted Media Extensions. Он поддерживает воспроизведение видео на настольных компьютерах и мобильных устройствах. Официальную документацию можно найти на странице https://docs.videojs.com/.

См. статью [Как использовать проигрыватель Video.js с помощью Служб мультимедиа Azure](how-to-video-js-player.md).


## <a name="next-steps"></a>Дальнейшие действия ##

- [Краткое руководство. Проигрыватель мультимедиа Azure](../azure-media-player/azure-media-player-quickstart.md)