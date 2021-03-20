---
title: Проигрыватель мультимедиа Azure методы API
description: Проигрыватель мультимедиа Azure API позволяет взаимодействовать с видео с помощью JavaScript, будь то браузер воспроизводит видео через HTML5 video, Flash, Silverlight или любые другие поддерживаемые технологии воспроизведения.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: reference
ms.date: 04/20/2020
ms.openlocfilehash: d1ad0cbe30ce20e61a31f0534a47fe031f6f257b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "81727272"
---
# <a name="api"></a>API #

Проигрыватель мультимедиа Azure API позволяет взаимодействовать с видео с помощью JavaScript, будь то браузер воспроизводит видео через HTML5 video, Flash, Silverlight или любые другие поддерживаемые технологии воспроизведения.

## <a name="referencing-the-player"></a>Ссылка на проигрыватель ##

Чтобы использовать функции API, необходим доступ к объекту Player. К счастью, ее легко получить. Необходимо только убедиться, что у тега Video есть идентификатор. Пример кода внедрения имеет идентификатор `vid1` . Если на одной странице есть несколько видеороликов, убедитесь, что каждый тег видео имеет уникальный идентификатор.

`var myPlayer = amp('vid1');`

> [!NOTE]
> Если проигрыватель еще не инициализирован с помощью атрибута настройки данных или другого метода, это также приведет к инициализации проигрывателя.

## <a name="wait-until-the-player-is-ready"></a>Дождитесь готовности проигрывателя ##

Время, необходимое Проигрыватель мультимедиа Azure для настройки видео и API, зависит от используемой технологии воспроизведения. Для загрузки HTML5 часто будет выполняться гораздо быстрее, чем Flash или Silverlight. По этой причине функция "Ready" игрока должна использоваться для активации кода, требующего API проигрывателя.

```javacript
    amp("vid_1").ready(function(){
      var myPlayer = this;

      // EXAMPLE: Start playing the video.
      myPlayer.play();
    });
```

OR

```javacript
    var myPlayer = amp("vid_1", myOptions, function(){
        //this is the ready function and will only execute after the player is loaded
    });
```

## <a name="api-methods"></a>Методы API ##

Теперь, когда у вас есть доступ к готовому игроку, вы можете управлять видео, получать значения или реагировать на события видео. Имена функций API Проигрыватель мультимедиа Azure пытаются следовать [API мультимедиа HTML5](http://www.whatwg.org/specs/web-apps/current-work/multipage/the-video-element.html). Основное различие заключается в том, что функции getter/setter используются для свойств видео.

```javacript
    // setting a property on a bare HTML5 video element
    myVideoElement.currentTime = 120;

    // setting a property with Azure Media Player
    myPlayer.currentTime(120);
```

## <a name="registering-for-events"></a>Регистрация для событий ##
События должны быть зарегистрированы непосредственно после инициализации проигрывателя в первый раз, чтобы убедиться, что все события должным образом переданы в приложение, и следует выполнять за пределами события ready.

```javacript
    var myPlayer = amp("vid_1", myOptions, function(){
        //this is the ready function and will only execute after the player is loaded
    });
    myPlayer.addEventListener(amp.eventName.error, _ampEventHandler);
    //add other event listeners
```

## <a name="next-steps"></a>Дальнейшие действия ##

<!---Some context for the following links goes here--->
- [Краткое руководство. Проигрыватель мультимедиа Azure](azure-media-player-quickstart.md)