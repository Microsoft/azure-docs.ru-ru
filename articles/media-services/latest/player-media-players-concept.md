---
title: Общие сведения о проигрывателях мультимедиа для Служб мультимедиа
description: Какие проигрыватели мультимедиа можно использовать со Службами мультимедиа Azure? На сегодняшний день можно использовать Проигрыватель мультимедиа Azure, Shaka и Video.js.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 3/08/2021
ms.openlocfilehash: 43d0a8ee82a84a57be7c8ded9e7f5afc172bd9d6
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106281402"
---
# <a name="media-players-for-media-services"></a>Проигрыватели мультимедиа для Служб мультимедиа

Для Служб мультимедиа можно использовать несколько проигрывателей мультимедиа.

## <a name="azure-media-player"></a>Проигрыватель мультимедиа Azure

Проигрыватель мультимедиа Azure — это видеопроигрыватель, который можно использовать для широкого спектра браузеров и устройств. Проигрыватель мультимедиа Azure использует такие отраслевые стандарты, как HTML5, Media Source Extensions (MSE) и Encrypted Media Extensions (EME), для оптимальной адаптивной потоковой передачи. Если эти стандарты недоступны на устройстве или в браузере, Проигрыватель мультимедиа Azure использует Flash и Silverlight. Независимо от используемой технологии воспроизведения разработчики получают единый интерфейс JavaScript для доступа к интерфейсам API. Это позволяет службам мультимедиа Azure обслуживать содержимое для воспроизведения на широком спектре устройств и в различных браузерах без дополнительных усилий.

См. [документацию по Проигрывателю мультимедиа Azure](../azure-media-player/azure-media-player-overview.md).

## <a name="shaka"></a>Shaka

Проигрыватель Shaka — это библиотека JavaScript с открытым исходным кодом для адаптивных мультимедиа. Он воспроизводит в браузере адаптивные форматы мультимедиа (например, DASH и HLS) без использования подключаемых модулей или Flash. Вместо этого проигрыватель Shaka использует открытые веб-стандарты Media Source Extensions и Encrypted Media Extensions.

См. статью [Как использовать проигрыватель Shaka с помощью Служб мультимедиа Azure](player-shaka-player-how-to.md).

## <a name="videojs"></a>Video.js

Video.js — это видеопроигрыватель, который воспроизводит в браузере адаптивные форматы мультимедиа (например, DASH и HLS). Video.js использует открытые веб-стандарты Media Source Extensions и Encrypted Media Extensions. Он поддерживает воспроизведение видео на настольных компьютерах и мобильных устройствах. Официальную документацию можно найти на странице https://docs.videojs.com/.

См. статью [Как использовать проигрыватель Video.js с помощью Служб мультимедиа Azure](player-how-to-video-js-player.md).


## <a name="next-steps"></a>Дальнейшие действия ##

- [Краткое руководство. Проигрыватель мультимедиа Azure](../azure-media-player/azure-media-player-quickstart.md)