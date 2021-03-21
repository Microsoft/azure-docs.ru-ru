---
title: Как использовать проигрыватель Шака со службами мультимедиа Azure
description: В этой статье объясняется, как использовать проигрыватель Шака со службами мультимедиа Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-js
ms.openlocfilehash: 7b4bda5a439b1e0b35dc6d3af99e0273e4987568
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101091346"
---
# <a name="how-to-use-the-shaka-player-with-azure-media-services"></a>Как использовать проигрыватель Шака со службами мультимедиа Azure

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

## <a name="overview"></a>Обзор

Проигрыватель Shaka — это библиотека JavaScript с открытым исходным кодом для адаптивных мультимедиа. Он воспроизводит в браузере адаптивные форматы мультимедиа (например, DASH и HLS) без использования подключаемых модулей или Flash. Вместо этого проигрыватель Shaka использует открытые веб-стандарты Media Source Extensions и Encrypted Media Extensions.

Рекомендуется использовать [Mux.js](https://github.com/videojs/mux.js/) как, без него, проигрыватель Шака будет поддерживать формат HLS кмаф, но не HLS TS.

Его официальную документацию можно найти в [документации по Шака Player](https://shaka-player-demo.appspot.com/docs/api/tutorial-welcome.html).

## <a name="sample-code"></a>Пример кода

Пример кода для этой статьи доступен на странице [Azure-Samples/Media-Services-3rdParty-Player-Samples](https://github.com/Azure-Samples/media-services-3rdparty-player-samples).

## <a name="implementing-the-player"></a>Реализация проигрывателя

Если необходимо реализовать собственный экземпляр проигрывателя, следуйте этим инструкциям.

1. Создайте `index.html` файл, в котором будет размещен проигрыватель. Добавьте следующие строки кода (можно заменить версии для более новых, если применимо):

    ```html
    <html>
      <head>
        <script src="//cdn.jsdelivr.net/npm/shaka-player@3.0.1/dist/shaka-player.compiled.js"></script>
        <script src="//cdn.jsdelivr.net/npm/mux.js@5.6.3/dist/mux.js"></script>
        <script type="module" src="index.js"></script>
      </head>
      <body>
        <video id="video" controls></video>
      </body>
    </html>
    ```

1. Добавьте файл JavaScript со следующим кодом:

    ```javascript
    // myScript.js
    shaka.polyfill.installAll();

    var video = document.getElementById('video');
    var player = new shaka.Player(video);
    window.player = player;

    var manifestUrl = 'https://amsplayeraccount-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/sample-vod.ism/manifest(format=m3u8-aapl)';
    player.load(manifestUrl);
    ```

1. Замените на `manifestUrl` URL-адрес HLS или тире из указателя потоковой передачи ресурса, который можно найти на странице указатель потоковой передачи в портал Azure.
    ![URL-адреса указателя для потоковой передачи](media/how-to-shaka-player/streaming-urls.png)

1. Запустите сервер (например, с `npm http-server` ), и проигрыватель должен работать...

## <a name="set-up-captions"></a>Настройка субтитров

### <a name="set-up-vod-captions"></a>Настройка субтитров VOD

Выполните приведенные ниже строки кода и замените `captionUrl` каталогом. ВТТ (файл ВТТ должен находиться на том же узле, чтобы избежать ошибки CORS), `lang` с использованием двух буквенных кодов для языка, а `type` также с помощью `caption` либо `subtitle` :

```javascript
player.configure('streaming.alwaysStreamText', true)
player.load(manifestUrl).then(function(){
        player.addTextTrack(captionUrl, lang, type, 'text/vtt');
        var tracks = player.getTextTracks();
        player.selectTextTrack(tracks[0]);
});
```

### <a name="set-up-live-stream-captions"></a>Настройка субтитров в Live Stream

Включить субтитры в динамическом потоке настраивается добавлением следующей строки кода:

```javascript
player.setTextTrackVisibility(true)
```

## <a name="set-up-token-authentication"></a>Настройка проверки подлинности маркеров

Выполните следующие строки кода и замените `token` строкой токена:

```javascript
player.getNetworkingEngine().registerRequestFilter(function (type, request) {
  if (type === shaka.net.NetworkingEngine.RequestType.LICENSE) {
    request.headers['Authorization'] = 'Bearer ' + token;
  }
});
```

## <a name="set-up-aes-128-encryption"></a>Настройка шифрования AES-128

Проигрыватель Шака в настоящее время не поддерживает шифрование AES-128.

Ссылка на [вопрос](https://github.com/google/shaka-player/issues/850) GitHub для отслеживания состояния этой функции.

## <a name="set-up-drm-protection"></a>Настройка защиты DRM

Проигрыватель Шака использует зашифрованные расширения мультимедиа (EME), для использования которых требуется безопасный URL-адрес. Таким образом, для тестирования любого содержимого, защищенного DRM, необходимо использовать HTTPS. Если сайт использует протокол HTTPS, то в манифесте и каждом сегменте также потребуется использовать протокол HTTPS. Это обусловлено требованиями к смешанному содержимому.

Порядок предпочтения Шака для управления URL-адресами своих серверов лицензирования:

1. Клеаркэй config, используемый для отладки, должен переопределять все остальное. (Приложение по-прежнему может указывать сервер лицензий Клеаркэй.)
2. Серверы, настроенные на уровне приложения (если таковые имеются), должны переопределять что-либо из манифеста.
3. Серверы лицензий, предоставляемые манифестом, используются только в том случае, если иное не указано.

Чтобы указать URL-адрес сервера лицензирования для Widevine или PlayReady, можно использовать следующий код:

```javascript
player.configure({
  drm: {
    servers: {
      "com.widevine.alpha": "YOUR WIDEVINE LICENSE URL",
      "com.microsoft.playready": "YOUR PLAYREADY LICENSE URL"
    }
  }
});

```

Для всего содержимого FairPlay требуется задать сертификат сервера. Он задается в конфигурации проигрывателя:

```javascript
const req = await fetch("YOUR FAIRPLAY CERTIFICATE URL");
const cert = await req.arrayBuffer();
player.configure('drm.advanced.com\\.apple\\.fps\\.1_0.serverCertificate', new Uint8Array(cert));
```

Дополнительные сведения см. в [документации по защите DRM Шака Player](https://shaka-player-demo.appspot.com/docs/api/tutorial-drm-config.html).

## <a name="next-steps"></a>Дальнейшие действия

* [Использование Проигрыватель мультимедиа Azure](../azure-media-player/azure-media-player-overview.md)
* [Краткое руководство. Шифрование содержимого](encrypt-content-quickstart.md)
