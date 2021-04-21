---
title: Использование проигрывателя Video.js с помощью Служб мультимедиа Azure
description: В этой статье объясняется, как использовать видеообъект HTML и JavaScript с помощью служб мультимедиа Azure
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
ms.openlocfilehash: 0ac00ecbbe5e0a72bccf6effe5e82fc7d973c91e
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106281407"
---
# <a name="how-to-use-the-videojs-player-with-azure-media-services"></a>Как использовать проигрыватель Video.js с помощью Служб мультимедиа Azure?

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

## <a name="overview"></a>Обзор

Video.js — это сетевой видеопроигрыватель, созданный для HTML5. Он воспроизводит в браузере адаптивные форматы мультимедиа (например, DASH и HLS) без использования подключаемых модулей или Flash. Video.js применяет открытые веб-стандарты Media Source Extensions и Encrypted Media Extensions. Кроме этого, он поддерживает воспроизведение видео на настольных компьютерах и мобильных устройствах.

Официальную документацию на него можно найти на странице [https://docs.videojs.com/](https://docs.videojs.com/).

## <a name="sample-code"></a>Пример кода
Пример кода по этой статье находится на странице [Azure-Samples/Media-Services-3rdParty-Player-Samples](https://github.com/Azure-Samples/media-services-3rdparty-player-samples).

## <a name="implement-the-player"></a>Ввод проигрывателя в работу

1. Создайте файл `index.html`, в котором будет размещен проигрыватель. Добавьте следующие строки кода (данные версии можно заменить на более новые, если применимо):

    ```html
    <html>
      <head>
        <link href="https://vjs.zencdn.net/7.8.2/video-js.css" rel="stylesheet" />
      </head>
      <body>
        <video id="video" class="video-js vjs-default-skin vjs-16-9" controls data-setup="{}">
        </video>

        <script src="https://vjs.zencdn.net/7.8.2/video.js"></script>
        <script src="https://cdn.jsdelivr.net/npm/videojs-contrib-eme@3.7.0/dist/videojs-contrib-eme.min.js"></script>
        <script type="module" src="index.js"></script>
      </body>
    <html>
    ```

2. Добавьте файл `index.js` со следующим кодом:

    ```javascript
    var videoJS = videojs("video");
    videoJS.src({
      src: "manifestUrl",
      type: "protocolType",
    });
    ```

3. Замените `manifestUrl` на URL-адрес HLS- или DASH-формата из указателя потоковой передачи вашего устройства; его можно найти на странице указателя потоковой передачи на портале Azure.
    ![URL-адреса указателя потоковой передачи](media/player-shaka-player-how-to/streaming-urls.png)

4. Замените `protocolType` следующими опциями:

    - "application/x-mpegURL" для протоколов HLS
    - "application/dash+xml" для протоколов DASH

### <a name="set-up-captions"></a>Настройка заголовков

Запустите `addRemoteTextTrack` метод и замените:

- `subtitleKind` на `"captions"`, `"subtitles"`, `"descriptions"` или `"metadata"`  
- `caption` на путь .vtt-файла (vtt-файл должен быть на том же самом хосте, во-избежание ошибки CORS)
- `subtitleLang` на код BCP 47 — для выбора языка, например `"eng"` для английского или `"es"` испанского
- `subtitleLabel` на желаемое имя заголовка, которое будет отображаться

```javascript
videojs.players.video.addRemoteTextTrack({
  kind: subtitleKind,
  src: caption,
  srclang: subtitleLang,
  label: subtitleLabel
});
```

### <a name="set-up-token-authentication"></a>Настройка проверки подлинности на основе маркеров

Маркер следует задавать в поле Authorization заголовка запроса. Чтобы избежать проблем с CORS, этот маркер следует устанавливать только в запросах с `'keydeliver'` URL-адресом. Следующие строки кода должны выполнить свою работу:

```javascript
setupTokenForDecrypt (options) {
  if (options.uri.includes('keydeliver')) {
    options.headers = options.headers || {}
    options.headers.Authorization = 'Bearer=' + this.getInputToken()
  }

  return options
}
```

После этого, приведенную выше функцию необходимо присоединить к событию `videojs.Hls.xhr.beforeRequest`.

```javascript
videojs.Hls.xhr.beforeRequest = setupTokenForDecrypt;
```

### <a name="set-up-aes-128-encryption"></a>Настройка шифрования AES-128

Video.js поддерживает шифрование AES-128 без дополнительных настроек. 

> [!NOTE] 
> В настоящее время имеется [проблема](https://github.com/videojs/video.js/issues/6717) с шифрованием и контентом HLS/DASH CMAF, которые не могут воспроизводиться.

### <a name="set-up-drm-protection"></a>Настройка защиты DRM

Для усиления защиты DRM необходимо добавить официальное расширение [videojs-contrib-eme](https://github.com/videojs/videojs-contrib-eme). Также работает и версия CDN.

1. В файле `index.js`, описанном выше, необходимо инициализировать расширение EME, добавив `videoJS.eme();` *перед* добавлением источника видео:

   ```javascript
    videoJS.eme();
   ```

2. Теперь можно определить URL-адреса служб DRM и URL-адреса соответствующих лицензий, это делается следующим образом:

   ```javascript
   videoJS.src({
       keySystems: {
           "com.microsoft.playready": "YOUR PLAYREADY LICENSE URL",
           "com.widevine.alpha": "YOUR WIDEVINE LICENSE URL",
           "com.apple.fps.1_0": {
            certificateUri: "YOUR FAIRPLAY CERTIFICATE URL",
            licenseUri: "YOUR FAIRPLAY LICENSE URL"
           }
         }
       })

   ```

#### <a name="acquiring-the-license-url"></a>Получение URL-адреса лицензии

Чтобы получить URL-адрес лицензии, выполняются следующие действия:

- Ознакомьтесь с конфигурацией поставщика DRM
- или, если вы используете пример, ознакомьтесь с `output.json` документом, созданным при выполнении *setup-vod.ps1* сценария PowerShell для VODs или скрипта *start-live.ps1* для динамических потоков. Кроме того, в этом файле вы найдете KID.

#### <a name="using-tokenized-drm"></a>Использование маркеров управления цифровыми правами

Чтобы обеспечить поддержку маркерной защиты DRM, в свойство проигрывателя `src` необходимо добавить следующую строку:

```javascript
videoJS.src({
src: ...,
emeHeaders: {'Authorization': "Bearer=" + "YOUR TOKEN"},
keySystems: {...
```

## <a name="next-steps"></a>Дальнейшие действия

- [Использование Проигрывателя мультимедиа Azure](../azure-media-player/azure-media-player-overview.md)  
- [Краткое руководство. Шифрование содержимого](drm-encrypt-content-how-to.md)
