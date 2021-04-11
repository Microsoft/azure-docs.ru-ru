---
title: Модуль переопределения URL-адресов Проигрывателя мультимедиа Azure
description: Проигрыватель мультимедиа Azure перезапишет заданный URL-адрес из Служб мультимедиа Azure, чтобы предоставить потоки в форматах SMOOTH, DASH, HLS v3 и HLS v4.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 04/05/2021
ms.custom: devx-track-js
ms.openlocfilehash: efa77ce7416b94331dce2bb4e7f77dd50c20d450
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106448582"
---
# <a name="url-rewriter"></a>Модуль переопределения URL-адресов #

По умолчанию проигрыватель мультимедиа Azure перезапишет заданный URL-адрес из Служб мультимедиа Azure, чтобы предоставить потоки в форматах SMOOTH, DASH, HLS v3 и HLS v4. Например, если источник указан следующим образом, Проигрыватель мультимедиа Azure попытается воспроизвести все указанные выше протоколы:

```html
    <video id="vid1" class="azuremediaplayer amp-default-skin">
        <source src="//example/path/to/myVideo.ism/manifest" type="application/vnd.ms-sstr+xml" />
    </video>
```

Но если вы не хотите использовать модуль переопределения URL-адресов, его можно отключить, добавив к параметру свойство `disableUrlRewriter`. Это означает, что вся информация, передаваемая в источники, напрямую передается проигрывателю без изменения.  Ниже приведен пример добавления двух источников в проигрыватель — в форматах потоковой передачи DASH и Smooth Streaming.

```html
    <video id="vid1" class="azuremediaplayer amp-default-skin">
        <source src="//example/path/to/myVideo.ism/manifest(format=mpd-time-csf)" type="application/dash+xml" data-setup='{"disableUrlRewriter": true}'/>
        <source src="//example/path/to/myVideo.ism/manifest" type="application/vnd.ms-sstr+xml" data-setup='{"disableUrlRewriter": true}'/>
    </video>
```

или диспетчер конфигурации служб

```javascript
    myPlayer.src([
        { src: "//example/path/to/myVideo.ism/manifest(format=mpd-time-csf)", type: "application/dash+xml", disableUrlRewriter: true },
        { src: "//example/path/to/myVideo.ism/manifest", type: "application/vnd.ms-sstr+xml", disableUrlRewriter: true }
    ]);
```

Кроме того, при необходимости можно указать конкретные форматы потоковой передачи, которые Проигрывателю мультимедиа Azure следует переопределить. Для этого используйте параметр `streamingFormats`. Возможные значения: `DASH`, `SMOOTH`, `HLSv3`, `HLSv4`, `HLS`. Разница между HLS и HLS v3 или v4 заключается в том, что формат HLS поддерживает воспроизведение содержимого FairPlay. Версии 3 и 4 не поддерживают FairPlay. Это полезно, если у вас нет политики доставки для конкретного протокола.  Ниже приведен пример, когда протокол DASH не включен для ресурса.

```html
    <video id="vid1" class="azuremediaplayer amp-default-skin">
        <source src="//example/path/to/myVideo.ism/manifest" type="application/vnd.ms-sstr+xml" data-setup='{"streamingFormats": ["SMOOTH", "HLS","HLS-V3", "HLS-V4"] }'/>
    </video>
```

или диспетчер конфигурации служб

```javascript
    myPlayer.src([
        { src: "//example/path/to/myVideo.ism/manifest", type: "application/vnd.ms-sstr+xml", streamingFormats: ["SMOOTH", "HLS","HLS-V3", "HLS-V4"]},
    ]);
```

В некоторых случаях приведенные выше два варианта можно использовать в сочетании друг с другом в зависимости от конкретного ресурса.

> [!NOTE]
> Сведения о защите Widevine сохраняются только для протокола DASH.

## <a name="next-steps"></a>Дальнейшие действия ##

- [Краткое руководство. Проигрыватель мультимедиа Azure](azure-media-player-quickstart.md)