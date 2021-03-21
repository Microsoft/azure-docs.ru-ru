---
title: Использование проигрывателя Video.js с помощью служб мультимедиа Azure
description: В этой статье объясняется, как использовать объект видео HTML и JavaScript с помощью служб мультимедиа Azure.
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
ms.openlocfilehash: a0b357705de04ed4c2be3223f9dd07f61f75d970
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98954586"
---
# <a name="how-to-use-the-videojs-player-with-azure-media-services"></a>Использование проигрывателя Video.js с помощью служб мультимедиа Azure

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

## <a name="overview"></a>Обзор

Video.js — это веб-Видеопроигрыватель, созданный для HTML5. Он воспроизводит в браузере адаптивные форматы мультимедиа (например, DASH и HLS) без использования подключаемых модулей или Flash. Вместо этого Video.js использует расширения веб-стандартов Медиасаурце и расширения зашифрованного носителя. Более того, он поддерживает воспроизведение видео на настольных и мобильных устройствах.

Его официальную документацию можно найти по адресу [https://docs.videojs.com/](https://docs.videojs.com/) .

## <a name="sample-code"></a>Пример кода
Пример кода для этой статьи доступен на странице [Azure-Samples/Media-Services-3rdParty-Player-Samples](https://github.com/Azure-Samples/media-services-3rdparty-player-samples).

## <a name="implement-the-player"></a>Реализация проигрывателя

1. Создайте `index.html` файл, в котором будет размещен проигрыватель. Добавьте следующие строки кода (можно заменить версии для более новых, если применимо):

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

2. Добавьте `index.js` файл со следующим кодом:

    ```javascript
    var videoJS = videojs("video");
    videoJS.src({
      src: "manifestUrl",
      type: "protocolType",
    });
    ```

3. Замените на `manifestUrl` URL-адрес HLS или тире из указателя потоковой передачи ресурса, который можно найти на странице указатель потоковой передачи в портал Azure.
    ![URL-адреса указателя для потоковой передачи](media/how-to-shaka-player/streaming-urls.png)

4. Замените на `protocolType` следующие параметры:

    - "Application/x-Мпегурл" для протоколов HLS
    - "Application/тире + XML" для протоколов ТИРЕ

### <a name="set-up-captions"></a>Настройка субтитров

Выполните `addRemoteTextTrack` метод и замените:

- `subtitleKind` с помощью `"captions"` , `"subtitles"` , `"descriptions"` или `"metadata"`  
- `caption` При использовании пути к файлу ВТТ (файл ВТТ должен находиться на том же узле, чтобы избежать ошибки CORS)
- `subtitleLang` с кодом BCP 47 для языка, например `"eng"` для английского или `"es"` испанского
- `subtitleLabel` с требуемым отображаемым именем заголовка

```javascript
videojs.players.video.addRemoteTextTrack({
  kind: subtitleKind,
  src: caption,
  srclang: subtitleLang,
  label: subtitleLabel
});
```

### <a name="set-up-token-authentication"></a>Настройка проверки подлинности маркеров

Маркер должен быть задан в поле Authorization заголовка запроса. Чтобы избежать проблем с CORS, этот маркер должен быть установлен только в запросах с `'keydeliver'` URL-адресом. Следующие строки кода должны выполнить работу:

```javascript
setupTokenForDecrypt (options) {
  if (options.uri.includes('keydeliver')) {
    options.headers = options.headers || {}
    options.headers.Authorization = 'Bearer=' + this.getInputToken()
  }

  return options
}
```

Затем приведенная выше функция должна быть присоединена к `videojs.Hls.xhr.beforeRequest` событию.

```javascript
videojs.Hls.xhr.beforeRequest = setupTokenForDecrypt;
```

### <a name="set-up-aes-128-encryption"></a>Настройка шифрования AES-128

Video.js поддерживает шифрование AES-128 без дополнительной настройки. 

> [!NOTE] 
> В настоящее время есть [проблемы](https://github.com/videojs/video.js/issues/6717) с шифрованием и HLS/тире кмаф, которые не могут воспроизводиться.

### <a name="set-up-drm-protection"></a>Настройка защиты DRM

Для поддержки защиты DRM необходимо добавить официальное расширение [видеожс-от участников сообщества-EME](https://github.com/videojs/videojs-contrib-eme) . Также работает версия CDN.

1. В `index.js` файле, описанном выше, необходимо инициализировать расширение EME, добавив `videoJS.eme();` *перед* добавлением источника видео:

   ```javascript
    videoJS.eme();
   ```

2. Теперь можно определить URL-адреса служб DRM и URL-адреса соответствующих лицензий следующим образом:

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

Чтобы получить URL-адрес лицензии, можно выполнить следующие действия:

- Ознакомьтесь с конфигурацией поставщика DRM
- или, если вы используете пример, ознакомьтесь с `output.json` документом, созданным при выполнении *setup-vod.ps1* сценария PowerShell для ВОДС или скрипта *start-live.ps1* для динамических потоков. Кроме того, вы найдете детей в этом файле.

#### <a name="using-tokenized-drm"></a>Использование маркеров управления цифровыми правами

Чтобы обеспечить поддержку маркерной защиты DRM, необходимо добавить следующую строку в `src` свойство проигрывателя:

```javascript
videoJS.src({
src: ...,
emeHeaders: {'Authorization': "Bearer=" + "YOUR TOKEN"},
keySystems: {...
```

## <a name="next-steps"></a>Дальнейшие действия

- [Использование Проигрыватель мультимедиа Azure](../azure-media-player/azure-media-player-overview.md)  
- [Краткое руководство. Шифрование содержимого](encrypt-content-quickstart.md)
