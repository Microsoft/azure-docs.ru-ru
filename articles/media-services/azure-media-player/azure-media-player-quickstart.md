---
title: Краткое руководство. Проигрыватель мультимедиа Azure
description: Узнайте о базовой настройке Проигрывателя мультимедиа Azure.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: quickstart
ms.date: 04/05/2021
ms.openlocfilehash: a6fd603318a25e15d1d4dcc1e3eaf75f96fc5ade
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106448633"
---
# <a name="azure-media-player-quickstart"></a>Краткое руководство по Проигрывателю мультимедиа Azure
Проигрыватель мультимедиа Azure можно легко настроить. Для базового воспроизведения мультимедийного содержимого из учетной записи Служб мультимедиа Azure требуется всего несколько минут. В этом разделе приведено общее описание основных этапов. В следующих разделах содержатся подробные сведения о настройке Проигрывателя мультимедиа Azure.  Просто добавьте указанные ниже включаемые файлы в `<head>` документа:

```html
    <link href="//amp.azure.net/libs/amp/latest/skins/amp-default/azuremediaplayer.min.css" rel="stylesheet">
    <script src= "//amp.azure.net/libs/amp/latest/azuremediaplayer.min.js"></script>
```

> [!IMPORTANT]
> **НЕ** используйте версию `latest` в рабочей среде, так как она может быть изменена по запросу. Измените `latest` на версию Проигрывателя мультимедиа Azure (например, измените `latest` на `1.0.0`). Вы можете [запросить требуемую версию Проигрывателя мультимедиа Azure](https://amp.azure.net/libs/amp/latest/docs/changelog.html).

## <a name="use-the-video-element"></a>Использование элемента video

Затем используйте элемент `<video>` как обычно, но с дополнительным атрибутом `data-setup`, содержащим любые параметры. Эти параметры могут содержать любой параметр Служб мультимедиа Azure в допустимом объекте JSON.

```html
    <video id="vid1" class="azuremediaplayer amp-default-skin" autoplay controls width="640" height="400" poster="poster.jpg" data-setup='{"nativeControlsForTouch": false}'>
        <source src="http://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest" type="application/vnd.ms-sstr+xml" />
        <p class="amp-no-js">
            To view this video please enable JavaScript, and consider upgrading to a web browser that supports HTML5 video
        </p>
    </video>
```

Если вы не хотите использовать автоматическую настройку, опустите атрибут `data-setup` и инициализируйте элемент video вручную.

```javascript
    var myPlayer = amp('vid1', { /* Options */
            "nativeControlsForTouch": false,
            autoplay: false,
            controls: true,
            width: "640",
            height: "400",
            poster: ""
        }, function() {
              console.log('Good to go!');
               // add an event listener
              this.addEventListener('ended', function() {
                console.log('Finished!');
            });
          }
    );
    myPlayer.src([{
        src: "http://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest",
        type: "application/vnd.ms-sstr+xml"
    }]);
```

## <a name="next-steps"></a>Дальнейшие действия ##

- [Полная настройка Проигрывателя мультимедиа Azure](./azure-media-player-full-setup.md)
