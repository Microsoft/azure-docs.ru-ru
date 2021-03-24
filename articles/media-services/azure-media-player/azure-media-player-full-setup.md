---
title: Полная настройка Проигрывателя мультимедиа Azure
description: Узнайте, как настроить Проигрыватель мультимедиа Azure.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: how-to
ms.date: 04/20/2020
ms.custom: devx-track-js
ms.openlocfilehash: e6ec59e3cb8d285652935b42493c820a7379eeb8
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104956083"
---
# <a name="azure-media-player-full-setup"></a>Полная настройка Проигрывателя мультимедиа Azure #

Проигрыватель мультимедиа Azure можно легко настроить. Для получения базового воспроизведения мультимедийного содержимого прямо из учетной записи служб мультимедиа Azure требуется несколько секунд. [Примеры](https://github.com/Azure-Samples/azure-media-player-samples) также предоставляются в каталоге Samples выпуска.

## <a name="step-1-include-the-javascript-and-css-files-in-the-head-of-your-page"></a>Шаг 1. Включение файлов JavaScript и CSS в заголовок страницы ##

С Проигрыватель мультимедиа Azure можно получить доступ к скриптам из размещенной версии CDN. Теперь рекомендуется размещать JavaScript перед тегом Body `<body>` , а не с `<head>` , но проигрыватель Azure меиа включает "HTML5 Шив", который должен находиться в головном офисе для более старых версий IE, чтобы учитывать тег video в качестве допустимого элемента.

> [!NOTE]
> Если вы уже используете HTML5 Шив, например [Modernizr](https://modernizr.com/) , можно включить проигрыватель мультимедиа Azure JavaScript в любом месте. Однако убедитесь, что ваша версия Modernizr включает в себя Шив для видео.

### <a name="cdn-version"></a>Версия CDN ###

```html
    <link href="//amp.azure.net/libs/amp/latest/skins/amp-default/azuremediaplayer.min.css" rel="stylesheet">
    <script src= "//amp.azure.net/libs/amp/latest/azuremediaplayer.min.js"></script>
```

> [!IMPORTANT]
> **НЕ** используйте версию `latest` в рабочей среде, так как она может быть изменена по запросу. Замените на `latest` версию проигрыватель мультимедиа Azure. Например, замените `latest` на `2.1.1`. Вы можете [запросить требуемую версию Проигрывателя мультимедиа Azure](azure-media-player-changelog.md).

> [!NOTE]
> С момента `1.2.0` выпуска больше не требуется включать расположение в резервные конференции (оно автоматически выбирает расположение по относительному пути к файлу azuremediaplayer.min.js). Вы можете изменить расположение резервных технических специалистов, добавив следующий скрипт в сценарий после приведенных `<head>` выше сценариев.

> [!NOTE]
> Из-за природы подключаемых модулей Flash и Silverlight файлы SWF и XAP должны размещаться в домене без конфиденциальной информации или данных. это автоматически позаботится о Azure CDN размещенной версии.

```javascript
    <script>
      amp.options.flashSS.swf = "//amp.azure.net/libs/amp/latest/techs/StrobeMediaPlayback.2.0.swf"
      amp.options.silverlightSS.xap = "//amp.azure.net/libs/amp/latest/techs/SmoothStreamingPlayer.xap"
    </script>
```

## <a name="step-2-add-an-html5-video-tag-to-your-page"></a>Шаг 2. Добавление на страницу тега видео HTML5 ##

С помощью Проигрыватель мультимедиа Azure можно внедрить видео с помощью тега видео HTML5. Проигрыватель мультимедиа Azure прочитает тег и сделает его работоспособным во всех браузерах, а не только на тех, которые поддерживают видео HTML5. Помимо базовой разметки Проигрыватель мультимедиа Azure требуется несколько дополнительных элементов.

1. `<data-setup>`Атрибут на вкладке `<video>` сообщает проигрыватель мультимедиа Azure, чтобы автоматически настроить видео, когда страница готова, и прочитать все (в формате JSON) из атрибута.
1. `id`Атрибут: должен использоваться и быть уникальным для каждого видео на одной странице.
1. `class`Атрибут содержит два класса:
    - `azuremediaplayer` применяет стили, необходимые для функций пользовательского интерфейса Проигрыватель мультимедиа Azure
    - `amp-default-skin` применяет обложку по умолчанию к элементам управления HTML5
1. `<source>`Включает два обязательных атрибута.
    - `src` атрибут может включать файл **. ISM/manifest* из служб мультимедиа Azure, проигрыватель мультимедиа Azure автоматически добавляет URL-адреса для ТИРЕ, СГЛАЖИВАНИЯ и HLS к проигрывателю.
    - `type` атрибут является требуемым типом MIME потока. Тип MIME, связанный с *". ISM/manifest"* , — *"application/vnd. МС-сстр + XML"*
1. *Необязательный* `<data-setup>` атрибут в `<source>` сообщает проигрыватель мультимедиа Azure, если для потока из служб мультимедиа Azure существуют уникальные политики доставки, включая, но не ограничено, тип шифрования (AES или PlayReady, Widevine или Fairplay) и токен.

Включить или исключить атрибуты, параметры, источники и дорожки точно так же, как в видеоролике HTML5.

```html
    <video id="vid1" class="azuremediaplayer amp-default-skin" autoplay controls width="640" height="400" poster="poster.jpg" data-setup='{"techOrder": ["azureHtml5JS", "flashSS", "html5FairPlayHLS","silverlightSS", "html5"], "nativeControlsForTouch": false}'>
        <source src="http://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest" type="application/vnd.ms-sstr+xml" />
        <p class="amp-no-js">
            To view this video please enable JavaScript, and consider upgrading to a web browser that supports HTML5 video
        </p>
    </video>
```

По умолчанию большая кнопка воспроизведения находится в верхнем левом углу, поэтому она не охватывает интересные части афиши. Если вы предпочитаете выровнять крупные кнопки воспроизведения, можно добавить дополнительный `amp-big-play-centered` `class` `<video>` элемент.

### <a name="alternative-setup-for-dynamically-loaded-html"></a>Альтернативная настройка динамически загружаемого HTML ###

Если веб-страница или приложение загружает тег video динамически (AJAX, appendChild и т. д.), так что он может не существовать при загрузке страницы, необходимо вручную настроить проигрыватель, вместо того чтобы полагаться на атрибут установки данных. Для этого сначала удалите атрибут настройки данных из тега, чтобы не было путаницы при инициализации проигрывателя. Затем запустите следующий сценарий JavaScript через некоторое время после загрузки Проигрыватель мультимедиа Azure JavaScript и после загрузки тега Video в модель DOM.

```javascript
    var myPlayer = amp('vid1', { /* Options */
            techOrder: ["azureHtml5JS", "flashSS", "html5FairPlayHLS","silverlightSS", "html5"],
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
        src: "http://samplescdn.origin.mediaservices.windows.net/e0e820ec-f6a2-4ea2-afe3-1eed4e06ab2c/AzureMediaServices_Overview.ism/manifest",
        type: "application/vnd.ms-sstr+xml"
    }]);
```

Первым аргументом в `amp` функции является идентификатор вашего видео-тега. Замените его собственным.

Вторым аргументом является объект Options. Он позволяет задавать дополнительные параметры, например, с помощью атрибута настройки данных.

Третьим аргументом является обратный вызов "Ready". После инициализации Проигрыватель мультимедиа Azure будет вызывать эту функцию. В обратном вызове Ready объект "this" ссылается на экземпляр проигрывателя.

Вместо использования идентификатора элемента можно также передать ссылку на сам элемент.

```javascript

    amp(document.getElementById('example_video_1'), {/*Options*/}, function() {
        //This is functionally the same as the previous example.
    });
    myPlayer.src([{ src: "//example/path/to/myVideo.ism/manifest", type: "application/vnd.ms-sstr+xml"]);
```

## <a name="next-steps"></a>Дальнейшие действия ##

- [Краткое руководство. Проигрыватель мультимедиа Azure](azure-media-player-quickstart.md)
