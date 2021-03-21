---
title: Внедрение графических мини-приложений индексатора видео в приложения
titleSuffix: Azure Media Services
description: Сведения о внедрении мини-приложений индексатора видео в приложения.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 01/25/2021
ms.author: juliako
ms.custom: devx-track-js
ms.openlocfilehash: b13086e11e1181bba91a3255e68e9f8a32e78450
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98797784"
---
# <a name="embed-video-indexer-widgets-in-your-apps"></a>Внедрение графических мини-приложений индексатора видео в приложения

В этой статье показано, как внедрить графические элементы индексатора видео в приложения. Индексатор видео поддерживает внедрение в приложение трех типов мини-приложений: *Аналитические сведения*, *Проигрыватель* и *Редактор*.

Начиная с версии 2, базовый URL-адрес Widget включает в себя регион указанной учетной записи. Например, создается учетная запись в западной части США: `https://www.videoindexer.ai/embed/insights/.../?location=westus2`.

## <a name="widget-types"></a>Типы мини-приложений

### <a name="cognitive-insights-widget"></a>Мини-приложение Cognitive Insights

Мини-приложение Cognitive Insights содержит все визуальные аналитические сведения, извлеченные при индексировании видео. Мини-приложение для анализа сведений поддерживает следующие дополнительные параметры URL:

|Имя|Определение|Описание|
|---|---|---|
|`widgets` | Строки, разделенные запятыми. | Позволяет управлять аналитическими сведениями, которые необходимо визуализировать.<br/>Пример: `https://www.videoindexer.ai/embed/insights/<accountId>/<videoId>/?widgets=people,keywords` визуализирует только пользователей и ключевые слова UI Insights.<br/>Доступные варианты: люди, Аниматедчарактерс, ключевые слова, метки, тональности, эмоции, темы, опорные кадры, транскрипция, OCR, динамики, сцены и Намедентитиес.|
|`controls`|Строки, разделенные запятыми.|Позволяет управлять элементами управления, которые необходимо визуализировать.<br/>Пример: `https://www.videoindexer.ai/embed/insights/<accountId>/<videoId>/?controls=search,download` отображает только параметры поиска и кнопку Скачать.<br/>Доступные параметры: Поиск, скачивание, предустановка, язык.|
|`language`|Короткий код языка (имя языка)|Язык управления аналитикой.<br/>Например, `https://www.videoindexer.ai/embed/insights/<accountId>/<videoId>/?language=es-es`. <br/>или `https://www.videoindexer.ai/embed/insights/<accountId>/<videoId>/?language=spanish`|
|`locale` | Короткий код языка | Управляет языком пользовательского интерфейса. Значение по умолчанию — `en`. <br/>Например, `locale=de`.|
|`tab` | Выбранная по умолчанию вкладка | Управляет вкладкой **Insights** , которая отображается по умолчанию. <br/>Пример: `tab=timeline` подготавливает аналитику к выбранной вкладке **временной шкалы** .|
|`location` ||`location`Параметр должен быть включен во внедренные ссылки. см. раздел [как получить имя вашего региона](regions.md). Если ваша учетная запись находится на этапе предварительной версии, `trial` следует использовать для значения Location. `trial` значение по умолчанию для `location` параметра.| 

### <a name="player-widget"></a>Мини-приложение Player

Вы можете использовать мини-приложение проигрывателя для потоковой передачи видео с использованием адаптивной скорости. Мини-приложение проигрывателя поддерживает следующие необязательные параметры URL.

|Имя|Определение|Описание|
|---|---|---|
|`t` | Секунд с начала | Запускает воспроизведение проигрывателя с указанной временной точки.<br/> Например, `t=60`. |
|`captions` | Код языка | Извлекает заголовок на указанном языке во время загрузки мини-приложения, чтобы быть доступным в меню **субтитров** .<br/> Например, `captions=en-US`. |
|`showCaptions` | Логическое значение | Позволяет проигрывателю загружаться с уже включенными субтитрами.<br/> Например, `showCaptions=true`. |
|`type`| | Активирует обложку аудио Player (часть видео удаляется).<br/> Например, `type=audio`. |
|`autoplay` | Логическое значение | Указывает, должен ли проигрыватель начать воспроизведение видео при загрузке. Значение по умолчанию — `true`.<br/> Например, `autoplay=false`. |
|`language`/`locale` | Код языка | Управляет языком проигрывателя. Значение по умолчанию — `en-US`.<br/>Например, `language=de-DE`.|
|`location` ||`location`Параметр должен быть включен во внедренные ссылки. см. раздел [как получить имя вашего региона](regions.md). Если ваша учетная запись находится на этапе предварительной версии, `trial` следует использовать для значения Location. `trial` значение по умолчанию для `location` параметра.| 

### <a name="editor-widget"></a>Мини-приложение редактора

Вы можете использовать мини-приложение редактора для создания новых проектов и управления аналитическими данными видео. Мини-приложение редактора поддерживает следующие необязательные параметры URL.

|Имя|Определение|Описание|
|---|---|---|
|`accessToken`<sup>*</sup> | Строка | Предоставляет доступ к видео, которые используются только в учетной записи, используемой для внедрения мини-приложения.<br> Для мини-приложения редактора требуется `accessToken` параметр. |
|`language` | Код языка | Управляет языком проигрывателя. Значение по умолчанию — `en-US`.<br/>Например, `language=de-DE`. |
|`locale` | Короткий код языка | Управляет языком Insights. Значение по умолчанию — `en`.<br/>Например, `language=de`. |
|`location` ||`location`Параметр должен быть включен во внедренные ссылки. см. раздел [как получить имя вашего региона](regions.md). Если ваша учетная запись находится на этапе предварительной версии, `trial` следует использовать для значения Location. `trial` значение по умолчанию для `location` параметра.| 

<sup>*</sup>Владелец должен предоставить `accessToken` осторожность.

## <a name="embedding-videos"></a>Внедрение видео

В этом разделе обсуждается внедрение открытого и закрытого содержимого в приложения.

`location`Параметр должен быть включен во внедренные ссылки. см. раздел [как получить имя вашего региона](regions.md). Если ваша учетная запись находится на этапе предварительной версии, `trial` следует использовать для значения Location. `trial` значение по умолчанию для `location` параметра. Например, `https://www.videoindexer.ai/accounts/00000000-0000-0000-0000-000000000000/videos/b2b2c74b8e/?location=trial`.

> [!IMPORTANT]
> Совместное использование ссылки на мини-приложение **Player** или **Insights** будет включать маркер доступа и предоставлять учетной записи разрешения только для чтения.

### <a name="public-content"></a>Общедоступное содержимое

1. Войдите на веб-сайт [индексатора видео](https://www.videoindexer.ai/) .
1. Выберите видео, с которым хотите работать, и нажмите кнопку **воспроизвести**.
1. Выберите нужный тип мини-приложения («**автоаналитические** сведения», « **проигрыватель**» или « **Редактор**»).
1. Нажмите кнопку **&lt; / &gt; внедрить**.
5. Скопируйте код внедрения (отображается в поле **Копировать внедренный код** в диалоговом окне **общий доступ & embed** ).
6. Добавьте код в приложение.

### <a name="private-content"></a>Конфиденциальное содержимое

Чтобы внедрить частное видео, необходимо передать маркер доступа в `src` атрибуте IFRAME:

`https://www.videoindexer.ai/embed/[insights | player]/<accountId>/<videoId>/?accessToken=<accessToken>`
    
Чтобы получить содержимое мини-приложения для анализа данных, используйте один из следующих методов.

- API [мини-приложения Get Insights](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Video-Insights-Widget?&pattern=widget) .<br/>
- [Маркер доступа для получения видео](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Get-Video-Access-Token?). Добавьте его в качестве параметра запроса к URL-адресу. Укажите этот URL-адрес в качестве `src` значения для IFRAME, как показано выше.

Чтобы предоставить возможности изменения аналитических сведений во внедренном мини-приложении, необходимо передать маркер доступа, включающий разрешения на редактирование. Используйте [мини-приложение Get Insights](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Video-Insights-Widget?&pattern=widget) или [Получите маркер доступа к видео](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Video-Access-Token?) с помощью `&allowEdit=true` .

## <a name="widgets-interaction"></a>Взаимодействие с мини-приложениями

Мини-приложение «информированная Аналитика» может взаимодействовать с видео в приложении. В этом разделе показано, как реализовать это взаимодействие.

![Индексатор видео мини-приложения "автоаналитические сведения"](./media/video-indexer-embed-widgets/video-indexer-widget03.png)

### <a name="flow-overview"></a>Обзор потока

При редактировании записей происходит следующая последовательность:

1. Вы изменяете запись на временной шкале.
1. Индексатор видео получает эти обновления и сохраняет их в записи [из правки](customize-language-model-with-website.md#customize-language-models-by-correcting-transcripts) в языковой модели.
1. Заголовки обновляются:

    * Если вы используете мини-приложение проигрывателя индексатора видео, оно автоматически обновляется.
    * Если вы используете внешний проигрыватель, вы получаете новый пользователь файл субтитров вызов **получения субтитров видео** .

### <a name="cross-origin-communications"></a>Обмен данными независимо от источника

Чтобы получить графические элементы индексатора видео для взаимодействия с другими компонентами, служба индексатора видео:

- Использует метод подключения HTML5 между источниками `postMessage` .
- проверяет сообщения, поступающие из источника VideoIndexer.ai и в него.

Если вы реализуете собственный код проигрывателя и интегрируетесь с мини-приложениями для «автоанализа», вы обязаны проверить происхождение сообщения, поступающих от VideoIndexer.ai.

### <a name="embed-widgets-in-your-app-or-blog-recommended"></a>Внедрение мини-приложений в приложение или блог (рекомендуется)

В этом разделе показано, как обеспечить взаимодействие между двумя мини-приложениями индексатора видео, чтобы при выборе пользователем элемента управления "аналитика" в приложении игрок перейдет к соответствующему моменту.

1. Скопируйте код внедрения для мини-приложения Player.
2. Скопируйте код внедрения для Cognitive Insights.
3. Добавьте [файл медиатора](https://breakdown.blob.core.windows.net/public/vb.widgets.mediator.js), чтобы реализовать взаимодействие между двумя мини-приложениями:<br/> 
`<script src="https://breakdown.blob.core.windows.net/public/vb.widgets.mediator.js"></script>`

Теперь, когда пользователь выбирает элемент управления Insights в приложении, игрок переходит к соответствующему моменту.

Дополнительные сведения см. в статье [индексатор видео — внедрение как мини-приложений демонстрации](https://codepen.io/videoindexer/pen/NzJeOb).

### <a name="embed-the-cognitive-insights-widget-and-use-azure-media-player-to-play-the-content"></a>Внедрение мини-приложения Cognitive Insights и использование Проигрывателя мультимедиа Azure для воспроизведения содержимого

В этом разделе показано, как обеспечить взаимодействие мини-приложения "автоаналитическая аналитика" и экземпляра Проигрыватель мультимедиа Azure с помощью [подключаемого модуля amp](https://breakdown.blob.core.windows.net/public/amp-vb.plugin.js).

1. Добавьте подключаемый модуль индексатора видео для проигрывателя AMP:<br/> `<script src="https://breakdown.blob.core.windows.net/public/amp-vb.plugin.js"></script>`
2. Создайте экземпляр Проигрыватель мультимедиа Azure с помощью подключаемого модуля индексатора видео.

    ```javascript
    // Init the source.
    function initSource() {
        var tracks = [{
        kind: 'captions',
        // To load vtt from VI, replace it with your vtt URL.
        src: this.getSubtitlesUrl("c4c1ad4c9a", "English"),
        srclang: 'en',
        label: 'English'
        }];
        myPlayer.src([
        {
            "src": "//amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest",
            "type": "application/vnd.ms-sstr+xml"
        }
        ], tracks);
    }

    // Init your AMP instance.
    var myPlayer = amp('vid1', { /* Options */
        "nativeControlsForTouch": false,
        autoplay: true,
        controls: true,
        width: "640",
        height: "400",
        poster: "",
        plugins: {
        videobreakedown: {}
        }
    }, function () {
        // Activate the plug-in.
        this.videobreakdown({
        videoId: "c4c1ad4c9a",
        syncTranscript: true,
        syncLanguage: true,
        location: "trial" /* location option for paid accounts (default is trial) */
        });

        // Set the source dynamically.
        initSource.call(this);
    });
    ```

3. Скопируйте код внедрения для Cognitive Insights.

Теперь вы можете взаимодействовать с Проигрыватель мультимедиа Azure.

Дополнительные сведения см. на странице [демонстрации проигрыватель мультимедиа Azure + VI Insights](https://codepen.io/videoindexer/pen/rYONrO).

### <a name="embed-the-video-indexer-cognitive-insights-widget-and-use-a-different-video-player"></a>Внедрение мини-приложения "автоаналитическая аналитика" для индексатора видео и использование другого видеопроигрывателя

При использовании видеопроигрывателя, отличного от Проигрыватель мультимедиа Azure, необходимо вручную управлять видеопроигрывателем, чтобы обеспечить взаимодействие.

1. Вставьте свой видеопроигрыватель.

    Например, стандартный проигрыватель HTML5:

    ```html
    <video id="vid1" width="640" height="360" controls autoplay preload>
       <source src="//breakdown.blob.core.windows.net/public/Microsoft%20HoloLens-%20RoboRaid.mp4" type="video/mp4" /> 
       Your browser does not support the video tag.
    </video>
    ```

2. Внедрите мини-приложение Cognitive Insights.
3. Реализуйте обмен данными для проигрывателя, настроив ожидание передачи данных о событии message. Пример:

    ```javascript
    <script>
    
        (function(){
        // Reference your player instance.
        var playerInstance = document.getElementById('vid1');
        
        function jumpTo(evt) {
          var origin = evt.origin || evt.originalEvent.origin;
        
          // Validate that the event comes from the videobreakdown domain.
          if ((origin === "https://www.videobreakdown.com") && evt.data.time !== undefined){
                
            // Call your player's "jumpTo" implementation.
            playerInstance.currentTime = evt.data.time;
               
            // Confirm the arrival to us.
            if ('postMessage' in window) {
              evt.source.postMessage({confirm: true, time: evt.data.time}, origin);
            }
          }
        }
        
        // Listen to the message event.
        window.addEventListener("message", jumpTo, false);
          
        }())    
        
    </script>
    ```

Дополнительные сведения см. на странице [демонстрации проигрыватель мультимедиа Azure + VI Insights](https://codepen.io/videoindexer/pen/YEyPLd).

## <a name="adding-subtitles"></a>Добавление субтитров

При внедрении индексатора видео с помощью собственного [проигрыватель мультимедиа Azure](https://aka.ms/azuremediaplayer)можно использовать `GetVttUrl` метод для получения скрытых субтитров (субтитров). Также можно вызвать метод JavaScript из подключаемого модуля индексатора AMP `getSubtitlesUrl` (как показано выше).

## <a name="customizing-embeddable-widgets"></a>Настройка внедряемых мини-приложений

### <a name="cognitive-insights-widget"></a>Мини-приложение Cognitive Insights

Вы можете выбрать нужные типы аналитики. Для этого укажите их в качестве значения для следующего параметра URL-адреса, который добавляется в полученный код внедрения (из API или из веб-приложения): `&widgets=<list of wanted widgets>` .

Возможные значения: `people` , `animatedCharacters` , `keywords` , `labels` , `sentiments` , `emotions` , `topics` , `keyframes` , `transcript` ,,, `ocr` `speakers` `scenes` и `namedEntities` .

Например, если вы хотите внедрить мини-приложение, содержащее только людей и ключевые слова, URL внедрения IFRAME будет выглядеть следующим образом:

`https://www.videoindexer.ai/embed/insights/<accountId>/<videoId>/?widgets=people,keywords`

Заголовок окна iframe можно настроить, указав `&title=<YourTitle>` в URL-адресе для iframe. (Он настраивает значение HTML `<title>` ).
   
Например, если для окна iframe нужно указать заголовок MyInsights, URL-адрес будет выглядеть так:

`https://www.videoindexer.ai/embed/insights/<accountId>/<videoId>/?title=MyInsights`

Обратите внимание на то, что этот параметр применим, только когда аналитические сведения нужно открыть в новом окне.

### <a name="player-widget"></a>Мини-приложение Player

При внедрении проигрывателя Индексатора видео можно выбрать размер проигрывателя, указав размер для элемента iframe.

Пример:

`<iframe width="640" height="360" src="https://www.videoindexer.ai/embed/player/<accountId>/<videoId>/" frameborder="0" allowfullscreen />`

По умолчанию проигрыватель индексаторов видео содержит автоматически созданные субтитры, основанные на транскрипции видео. Запись извлекается из видео с использованием исходного языка, выбранного при загрузке видео.

Если вы хотите внедрить с другим языком, можно добавить `&captions=<Language Code>` URL-адрес для внедрения проигрывателя. Если требуется, чтобы заголовки отображались по умолчанию, можно передать &Шовкаптионс = true.

URL-адрес внедрения будет выглядеть так:

`https://www.videoindexer.ai/embed/player/<accountId>/<videoId>/?captions=en-us`

#### <a name="autoplay"></a>Автозапуск

По умолчанию проигрыватель начнет воспроизведение видео. Вы можете не использовать, передав `&autoplay=false` предыдущий URL-адрес внедрения.

## <a name="code-samples"></a>Примеры кода

См. репозиторий с [примерами кода](https://github.com/Azure-Samples/media-services-video-indexer/tree/master/Embedding%20widgets) , который содержит примеры для API индексатора видео и мини-приложений:

| Файл или папка                       | Описание                                |
|-----------------------------------|--------------------------------------------|
| `azure-media-player`              | Загрузка видео индексатора видео в пользовательском Проигрыватель мультимедиа Azure.                        |
| `azure-media-player-vi-insights`  | Внедрите аналитику VI Insights с помощью настраиваемого Проигрыватель мультимедиа Azure.                             |
| `control-vi-embedded-player`      | Внедрять проигрыватель VI и управлять им извне.                                    |
| `custom-index-location`           | Внедрите аналитику VI из пользовательского внешнего расположения (это может быть клиентский большой двоичный объект).     |
| `embed-both-insights`             | Основное использование редактора VI Insights и игрока и ценных сведений.                            |
| `embed-insights-with-AMP`         | Внедрить мини-приложение VI Insights с настраиваемым Проигрыватель мультимедиа Azure.                      |
| `customize-the-widgets`           | Внедрите мини-приложения VI с настраиваемыми параметрами.                                     |
| `embed-both-widgets`              | Внедрите проигрыватель VI и аналитические сведения и обмен данными между ними.                      |
| `url-generator`                   | Создает URL-адрес настраиваемого внедрения для мини-приложений на основе заданных пользователем параметров.             |
| `html5-player`                    | Внедрение "VI Insights" с видеопроигрывателем HTML5 по умолчанию.                           |

## <a name="supported-browsers"></a>Поддерживаемые браузеры

Дополнительные сведения см. в разделе [Поддерживаемые браузеры](video-indexer-overview.md#supported-browsers).

## <a name="next-steps"></a>Дальнейшие действия

Сведения о просмотре и редактировании индексатора видео см. в статье [Просмотр и изменение индексаторов видео](video-indexer-view-edit.md).

Кроме того, ознакомьтесь с [CodePenом индексатора видео](https://codepen.io/videoindexer/pen/eGxebZ).
