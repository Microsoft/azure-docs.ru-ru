---
title: Технология воспроизведения в Проигрывателе мультимедиа Azure
description: Узнайте больше о технологии, которая используется для воспроизведения видео или аудио.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 04/05/2021
ms.custom: devx-track-js
ms.openlocfilehash: 0fadb679260582a788eb6408ff3761f00c2ceb1a
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106448834"
---
# <a name="playback-technology-tech"></a>Технология воспроизведения #

Технология воспроизведения — это технологические решения, которые используются конкретным браузером или подключаемым модулем для воспроизведения видео или аудио.

- **azureHtml5JS.** Используются стандарты MSE и EME в сочетании с элементом Video для воспроизведения без подключаемых модулей содержимого формата DASH. При этом поддерживается 128-битное шифрование содержимого AES или общее шифрование содержимого DRM (через PlayReady и Widevine, если браузер поддерживает это) из Служб мультимедиа Azure.
- **flashSS.** Используется технология Flash Player для воспроизведения содержимого Smooth с поддержкой 128-битного шифрования AES из Служб мультимедиа Azure. Требуется версия Flash 11.4 или более поздняя.
- **html5FairPlayHLS.** Используется браузерная технология воспроизведения, характерная для Safari, на основе HLS с элементом Video. Для этой технологии требуется воспроизведение защищенного содержимого FairPlay из Служб мультимедиа Azure. Ее поддержка добавлена в techOrder 19 октября 2016 г.
- **SilverlightSS.** Используется технология Silverlight для воспроизведения содержимого Smooth с поддержкой защищенного содержимого PlayReady из Служб мультимедиа Azure.
- **html5.** Используется браузерная технология воспроизведения с элементом Video.  На устройстве Apple iOS или Android эта технология позволяет воспроизводить потоки HLS с некоторой базовой поддержкой 128-битового шифрования AES или содержимого DRM (через FairPlay, если браузер поддерживает это).

## <a name="tech-order"></a>Порядок технологий ##

Чтобы гарантировать возможность воспроизводить ресурс на широком спектре устройств, мы рекомендуем использовать следующий порядок технологий, который установлен по умолчанию: `techOrder: ["azureHtml5JS", "flashSS", "html5FairPlayHLS","silverlightSS", "html5"]`. Его можно задать непосредственно в `<video>` или программными средствами через параметры.

`<video data-setup='{"techOrder": ["azureHtml5JS", "flashSS", "html5FairPlayHLS, "silverlightSS", "html5"]}`

или диспетчер конфигурации служб

```javascript
    amp("vid1", {
          techOrder: ["azureHtml5JS", "flashSS", "html5FairPlayHLS, "silverlightSS", "html5"]
    });
```

## <a name="compatibility-matrix"></a>Матрица совместимости ##

Учитывая рекомендуемый порядок технологий для содержимого потоковой передачи в Службах мультимедиа Azure, ожидается указанная матрица совместимости воспроизведения.

| Браузер        | OS                                                       | Ожидаемая технология (без шифрования)  | Ожидаемая технология (AES)  | Ожидаемая технология (DRM)          |
|----------------|----------------------------------------------------------|------------------------|----------------------|------------------------------|
| EdgeIE 11      | Windows 10, Windows 8.1, Windows Phone 10<sup>1</sup>               | azureHtml5JS           | azureHtml5JS         | azureHtml5JS (PlayReady)     |
| IE 11          | Windows 7, Windows Vista<sup>1</sup>                     | flashSS                | flashSS              | SilverlightSS (PlayReady)    |
| IE 11          | Windows Phone 8.1                                        | azureHtml5JS           | azureHtml5JS         | Не поддерживается                |
| Edge           | Xbox One<sup>1</sup> (обновление от ноября 2015 г.)                   | azureHtml5JS           | azureHtml5JS         | Не поддерживается                |
| Chrome 37+     | Windows 10, Windows 8.1, macOS X Yosemite<sup>1</sup>   | azureHtml5JS           | azureHtml5JS         | azureHtml5JS (Widevine)      |
| Firefox 47+    | Windows 10, Windows 8.1, macOS X Yosemite+<sup>1</sup>  | azureHtml5JS           | azureHtml5JS         | azureHtml5JS (Widevine)      |
| Firefox 42-46  | Windows 10, Windows 8.1, macOS X Yosemite+<sup>1</sup>  | azureHtml5JS           | azureHtml5JS         | SilverlightSS (PlayReady)    |
| Firefox 35–41  | Windows 10, Windows 8.1                                  | flashSS                | flashSS              | SilverlightSS (PlayReady)    |
| Safari         | iOS 6+                                                   | html5                  | html5 (без маркера)3    | Не поддерживается                |
| Safari 8+      | OS X Yosemite+                                           | azureHtml5JS           | azureHtml5JS         | html5FairPlayHLS (FairPlay)  |
| Safari 6       | OS X Mountain Lion<sup>1</sup>                           | flashSS                | flashSS              | SilverlightSS (PlayReady)    |
| Chrome 37+     | Android 4.4.4+<sup>2</sup>                               | azureHtml5JS           | azureHtml5JS         | azureHtml5JS (Widevine)      |
| Chrome 37+     | Android 4.02                                             | html5                  | html5 (без маркера)<sup>3</sup>    | Не поддерживается                |
| Firefox 42+    | Android 5.0+<sup>2</sup>                                 | azureHtml5JS           | azureHtml5JS         | Не поддерживается                |
| IE 8, IE 9, IE 10  | Windows                                                  | Не поддерживается          | Не поддерживается        | Не поддерживается                |

<sup>1</sup> Такая конфигурация не поддерживается или не протестирована. Здесь она указана только для полноты информации.

<sup>2</sup> Для успешного воспроизведения на устройствах Android требуется правильное сочетание возможностей устройства, поддержки графических устройств, визуализации в кодеках, поддержки ОС и т. д. Так как Android — это платформа с открытым кодом, которая позволяет изготовителям телефонов изменять чистую ОС Android, предоставляемую Google, среду Android нельзя считать достаточно однородной и некоторые устройства могут не поддерживаться из-за отсутствия необходимых функций. Также некоторые устройства Android поддерживают не все кодеки.  

<sup>3</sup> Если маркер не поддерживается, эту функцию можно реализовать через прокси-сервер. Дополнительные сведения об этом решении вы найдете в нашем [блоге](https://azure.microsoft.com/blog/2015/03/06/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/).

> [!NOTE]
> Если для приоритетной технологии требуется установить подключаемый модуль (например, Flash), который не установлен на устройстве пользователя, AMP перейдет к проверке возможности для следующей технологии в списке приоритетов с учетом типов источника и сведений о защите. Например, при попытке просмотреть незащищенный поток по требованию в Safari 8 под управлением OS X Yosemite, где не установлены ни Flash, ни Silverlight, AMP выберет для воспроизведения встроенную реализацию Html5.<br/><br/>Ежедневно появляются новые браузерные технологии. Поэтому в эту таблицу могут быть внесены изменения.

## <a name="next-steps"></a>Дальнейшие действия ##

- [Краткое руководство. Проигрыватель мультимедиа Azure](azure-media-player-quickstart.md)