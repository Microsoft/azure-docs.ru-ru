---
title: Вставка рекламы на стороне клиента | Документация Майкрософт
description: В этой статье показано, как вставлять рекламу в мультимедиа на стороне клиента.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 65c9c747-128e-497e-afe0-3f92d2bf7972
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 2d942099d0abbdfc4ddfa0276184500166250728
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103014967"
---
# <a name="inserting-ads-on-the-client-side"></a>Вставка рекламы на стороне клиента

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

В этой статье описано, как использовать разные типы рекламы на стороне клиента.

Сведения о поддержке субтитров и рекламы в динамических потоковых видео см. в разделе [Вставка субтитров и рекламы](media-services-live-streaming-with-onprem-encoders.md#cc_and_ads).

> [!NOTE]
> Проигрыватель мультимедиа Azure на данный момент не поддерживает рекламу.
> 
> 

## <a name="inserting-ads-into-your-media"></a><a id="insert_ads_into_media"></a>Вставка рекламы в мультимедиа
Службы мультимедиа Azure поддерживают вставку рекламы с помощью платформ проигрывателя Windows Media. Платформы проигрывателя с поддержкой рекламы доступны для устройств Windows 8, Silverlight, Windows Phone 8 и iOS. Каждая платформа проигрывателя содержит образец кода, в котором демонстрируется реализация приложения проигрывателя. В содержимое мультимедиа можно вставить три вида рекламы:

* **Линейная** — реклама на весь кадр, которая приостанавливает основное видео.
* **Нелинейная** — наложенная реклама, отображаемая во время воспроизведения основного видео, обычно это логотип или другое статическое изображение, размещенное в проигрывателе.
* **Сопутствующая** — рекламные объявления, отображаемые за пределами проигрывателя.

Рекламу можно размещать в любой временной точке основного видео. Необходимо сообщить проигрывателю, когда будет показана реклама и какие рекламные объявления следует показать. Для этого используется набор стандартных XML-файлов: Video Ad Service Template (VAST), Digital Video Multiple Ad Playlist (VMAP), Media Abstract Sequencing Template (MAST) и Digital Video Player Ad Interface Definition (VPAID). VAST-файлы указывают, какая реклама будет отображаться. VMAP-файлы указывают, когда следует воспроизводить различные рекламные ролики, и содержат XML-код VAST. MAST-файлы также упорядочивают рекламные объявления, которые могут содержать XML VAST. VPAID-файлы определяют интерфейс между видеопроигрывателем и рекламой или сервером рекламы.

Каждая платформа проигрывателя работает по-разному, и каждой из них посвящена отдельная статья. В этой статье описывается базовый механизм, используемый для вставки рекламы. Приложения видеопроигрывателя запрашивают рекламу с сервера рекламы. Сервер рекламы может ответить несколькими способами:

* Вернуть VAST-файл
* Вернуть VMAP-файл (с внедренным кодом VAST)
* Вернуть MAST-файл (с внедренным кодом VAST)
* Вернуть VAST-файл с рекламными объявлениями VPAID

### <a name="using-a-video-ad-service-template-vast-file"></a>Использование файла Video Ad Service Template (VAST)
VAST-файл определяет, какие рекламные ролики будут отображены. Следующий XML-код является примером VAST-файла для линейной рекламы:

```xml
    <VAST version="2.0" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
      <Ad id="115571748">
        <InLine>
          <AdSystem version="2.0 alpha">Atlas</AdSystem>
          <AdTitle>Unknown</AdTitle>
          <Description>Unknown</Description>
          <Survey></Survey>
          <Error></Error>
          <Impression id="Atlas"><![CDATA[http://www.myserver.com/tracking-resource]]></Impression>
          <Creatives>
            <Creative id="video" sequence="0" AdID="">
              <Linear>
                <Duration>00:00:32</Duration>
                <TrackingEvents>
                  <Tracking event="start"><![CDATA[http://www.myserver.com/start-tracking-resource]]></Tracking>
                  <Tracking event="midpoint"><![CDATA[http://www.myserver.com/midpoint-tracking-resource]]></Tracking>
                  <Tracking event="complete"><![CDATA http://www.myserver.com/complete-tracking-resource]]></Tracking>
                  <Tracking event="expand"><![CDATA[http://www.myserver.com/expand-tracking-resource]]></Tracking>
                </TrackingEvents>
                <VideoClicks>
                  <ClickThrough id="Atlas Redirect"><![CDATA[http://www.myserver.com/click-resource]]></ClickThrough>
                  <ClickTracking id="Spare"></ClickTracking>
                </VideoClicks>
                <MediaFiles>
                  <MediaFile apiFramework="Windows Media" id="windows_progressive_200" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="200" width="400" height="300" type="video/x-ms-wmv">
                    <![CDATA[http://www.myserver.com/media/myad_200_4x3.wmv]]>
                  </MediaFile>
                  <MediaFile apiFramework="Windows Media" id="windows_progressive_300" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="300" width="400" height="300" type="video/x-ms-wmv">
                    <![CDATA[http://www.myserver.com/media/myad_300_4x3.wmv]]>
                  </MediaFile>
                </MediaFiles>
              </Linear>
            </Creative>
          </Creatives>
          <Extensions>
            <Extension type="Atlas">
            </Extension>
          </Extensions>
        </InLine>
      </Ad>
    </VAST>
```

Линейная реклама описывается элементом <**Linear**>. Он определяет продолжительность рекламы, события отслеживания, переход по щелчкам, отслеживание щелчков и ряд элементов **MediaFile**. События отслеживания задаются в элементе <**TrackingEvents**> и позволяют серверу рекламы отслеживать различные события, происходящие во время просмотра рекламы. В данном случае отслеживаются события начала, середины, завершения и разворачивания. Событие начала происходит при отображении рекламного объявления. Событие середины происходит, когда просмотрено не менее 50 % времени объявления. Событие завершения происходит, когда реклама была воспроизведена до конца. Событие разворачивания происходит, когда пользователь разворачивает видеопроигрыватель на весь экран. Переходы по щелчкам указываются с помощью элемента <**ClickThrough**> внутри элемента <**VideoClicks**> и указывают URI ресурса, отображаемого при щелчке пользователя по рекламному объявлению. Отслеживание щелчков указывается в элементе <**ClickTracking**>, а также в элементе <**VideoClicks**>, позволяя определить ресурс отслеживания для запроса проигрывателей при щелчке пользователя по рекламному объявлению. Элементы <**MediaFile**> определяют сведения о конкретной кодировке рекламы. При наличии более одного элемента <**MediaFile**> видеопроигрыватель может выбрать лучшую кодировку для платформы.

Линейную рекламу можно отображать в указанном порядке. Для этого добавьте в VAST-файл дополнительные элементы `<Ad>` и укажите порядок с помощью атрибута последовательности (sequence). Проиллюстрируем это на примере.

```xml
    <VAST version="2.0" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
      <Ad id="1" sequence="0">
        <InLine>
          <AdSystem version="2.0 alpha">Atlas</AdSystem>
          <AdTitle>Unknown</AdTitle>
          <Description>Unknown</Description>
          <Survey></Survey>
          <Error></Error>
          <Impression id="Atlas"><![CDATA[http://myserver.com/Impression/Ad1trackingResource]]></Impression>
          <Creatives>
            <Creative id="video" sequence="0" AdID="">
              <Linear>
                <Duration>00:00:32</Duration>
                <MediaFiles>
                  <!-- ... -->
                </MediaFiles>
              </Linear>
            </Creative>
          </Creatives>
        </InLine>
      </Ad>
      <Ad id="2" sequence="1">
        <InLine>
          <AdSystem version="2.0 alpha">Atlas</AdSystem>
          <AdTitle>Unknown</AdTitle>
          <Description>Unknown</Description>
          <Survey></Survey>
          <Error></Error>
          <Impression id="Atlas"><![CDATA[http://myserver.com/Impression/Ad2trackingResource]]></Impression>
          <Creatives>
            <Creative id="video" sequence="0" AdID="">
              <Linear>
                <Duration>00:00:30</Duration>
                <MediaFiles>
                  <!-- ... -->
                </MediaFiles>
              </Linear>
            </Creative>
          </Creatives>
        </InLine>
      </Ad>
    </VAST>
```

Нелинейная реклама также задается в элементе `<Creative>`. В следующем примере показан элемент `<Creative>`, описывающий нелинейную рекламу.

```xml
    <Creative id="video" sequence="1" AdID="">
      <NonLinearAds>
        <NonLinear width="216" height="121" minSuggestedDuration="00:00:15">
          <StaticResource creativeType="image/png"><![CDATA[http://myserver/images/image.png]]></StaticResource>
          <StaticResource creativeType="image/jpg"><![CDATA[http://myserver/images/image.jpg]]></StaticResource>
        </NonLinear>
        <TrackingEvents>
             <Tracking event="acceptInvitation"><![CDATA[http://myserver/tracking/trackingID]></Tracking>
             <Tracking event="collapse"><![CDATA[http://myserver/tracking/trackingID2]]></Tracking>
         </TrackingEvents>
       </NonLinearAds>
    </Creative>
```

В элементе <**NonLinearAds**> могут содержаться один или несколько элементов <**NonLinear**>, каждый из которых описывает нелинейное рекламное объявление. Элемент <**NonLinear**> определяет ресурс для нелинейной рекламы. Ресурсом может быть <**StaticResource**>, <**IFrameResource**> или <**HTMLResource**>. \<**StaticResource**> описывает ресурс, не относящийся к HTML, и определяет атрибут creativeType, который определяет способ отображения ресурса.

Image/gif, image/jpeg, image/png — ресурс отображается в HTML-теге <**img**>.

Application/x-javascript — ресурс отображается в HTML-теге <**script**>.

Application/x-shockwave-flash — ресурс отображается в проигрывателе Flash.

**IFrameResource** описывает ресурс HTML, который может быть отображен в IFrame. **HTMLResource** описывает фрагмент HTML-кода, который может быть вставлен в веб-страницу. **TrackingEvents** указывают события отслеживания и универсальный код ресурса (URI), который должен запрашиваться при возникновении события. В этом примере отслеживаются события acceptInvitation и collapse. Дополнительные сведения об элементе **NonLinearAds** и его дочерних элементах см. в разделе IAB.NET/VAST. Обратите внимание, что элемент **TrackingEvents** располагается внутри элемента **NonLinearAds**, а не элемента **NonLinear**.

Сопутствующая реклама задается в элементе `<CompanionAds>`. В элементе `<CompanionAds>` могут содержаться один или несколько элементов `<Companion>`. Каждый элемент `<Companion>` описывает сопутствующее рекламное объявление и может содержать `<StaticResource>`, `<IFrameResource>` или `<HTMLResource>`, которые указываются таким же способом, как и в нелинейной рекламе. VAST-файл может содержать несколько элементов сопутствующей рекламы, и приложение проигрывателя может выбрать наиболее подходящее для отображения рекламное объявление. Дополнительные сведения о VAST см. в разделе [VAST 3.0](https://www.iab.net/media/file/VASTv3.0.pdf).

### <a name="using-a-digital-video-multiple-ad-playlist-vmap-file"></a>Использование файла Digital Video Multiple Ad Playlist (VMAP)
VMAP-файл позволяет указать время рекламных пауз, продолжительность каждой паузы, количество отображаемых в течение одной паузы рекламных объявлений, а также типы рекламы, которые могут отображаться во время паузы. Ниже приведен пример VMAP-файла, который определяет одну рекламную паузу:

```xml
    <vmap:VMAP xmlns:vmap="http://www.iab.net/vmap-1.0" version="1.0">
      <vmap:AdBreak breakType="linear" breakId="mypre" timeOffset="start">
        <vmap:AdSource allowMultipleAds="true" followRedirects="true" id="1">
          <vmap:VASTData>
            <VAST version="2.0" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
              <Ad id="115571748">
                <InLine>
                  <AdSystem version="2.0 alpha">Atlas</AdSystem>
                  <AdTitle>Unknown</AdTitle>
                  <Description>Unknown</Description>
                  <Survey></Survey>
                  <Error></Error>
                  <Impression id="Atlas"><![CDATA[https://view.atdmt.com/000/sview/115571748/direct;ai.201582527;vt.2/01/634364885739970673]]></Impression>
                  <Creatives>
                    <Creative id="video" sequence="0" AdID="">
                      <Linear>
                        <Duration>00:00:32</Duration>
                        <MediaFiles>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_200" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="200" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_1_000_200_4x3.wmv]]>
                          </MediaFile>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_300" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="300" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_2_000_300_4x3.wmv]]>
                          </MediaFile>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_500" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="500" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_1_000_500_4x3.wmv]]>
                          </MediaFile>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_700" maintainAspectRatio="true" scalable="true" delivery="progressive" bitrate="700" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_2_000_700_4x3.wmv]]>
                          </MediaFile>
                        </MediaFiles>
                      </Linear>
                    </Creative>
                  </Creatives>
                </InLine>
              </Ad>
            </VAST>
          </vmap:VASTData>
        </vmap:AdSource>
        <vmap:TrackingEvents>
          <vmap:Tracking event="breakStart">
            http://MyServer.com/breakstart.gif
          </vmap:Tracking>
        </vmap:TrackingEvents>
      </vmap:AdBreak>
    </vmap:VMAP>
```

VMAP-файл начинается с элемента `<VMAP>`, в котором содержатся один или несколько элементов `<AdBreak>`, каждый из которых определяет рекламную паузу. Каждая рекламная пауза определяется типом паузы, идентификатором и смещение времени. Атрибут breakType указывает тип рекламы, который может воспроизводиться во время паузы: линейная, нелинейная или экранная. Экранная реклама сравнима с сопутствующей рекламой VAST. Можно указать несколько типов рекламы в списке, разделенном запятыми (без пробелов). Атрибут breakID является необязательным идентификатором для рекламы. Атрибут timeOffset указывает, когда должна отображаться реклама. Его можно задать одним из следующих способов:

1. Время в формате hh:mm:ss или hh:mm:ss.mmm, где .mmm — количество миллисекунд. Значение этого атрибута указывает время от начала временной шкалы видео до начала рекламной паузы.
2. Процент в формате n%, где n — процент временной шкалы видео, который будет воспроизведен до начала рекламы.
3. Начало/конец — указывает, что реклама должна воспроизводиться до или после отображения видео.
4. Позиция — указывает порядок рекламных пауз, когда время их показа неизвестно, например при прямой потоковой трансляции. Порядок каждой рекламной паузы задается в формате #n, где n — целое число 1 или больше. 1 означает, что рекламу следует воспроизводить при первой же возможности, 2 — при второй возможности и т. д.

В элементе `<AdBreak>` может быть один элемент <**AdSource**>. Элемент <**AdSource**> содержит следующие атрибуты.

1. Id — указывает идентификатор источника рекламы
2. allowMultipleAds — логическое значение, указывающее, могут ли отображаться несколько рекламных объявлений во время паузы
3. followRedirects — необязательное логическое значение, указывающее, следует ли видеопроигрывателю учитывать перенаправления в ходе отклика на рекламу

Элемент <**AdSource**> предоставляет проигрывателю встроенный отклик на рекламу или ссылку на отклик на рекламу. Он может содержать один из следующих элементов:

* `<VASTAdData>` указывает, что отклик на рекламу VAST внедрен в VMAP-файл;
* `<AdTagURI>` — URI, который ссылается на отклик на рекламу из другой системы;
* `<CustomAdData>` — произвольная строка, представляющая отклик, не связанный с VAST.

В этом примере указывается встроенный отклик на рекламу с элементом `<VASTAdData>`, содержащим отклик на рекламу VAST. Дополнительные сведения о других элементах см. в разделе [VMAP](https://www.iab.net/guidelines/508676/digitalvideo/vsuite/vmap).

В элементе <**AdBreak**> также может содержаться один элемент <**TrackingEvents**>. Элемент <**TrackingEvents**> позволяет отслеживать начало или конец рекламной паузы или возникновение ошибок во время рекламной паузы. Элемент <**TrackingEvents**> содержит один или несколько элементов <**Tracking**>, каждый из которых задает событие отслеживания и URI отслеживания. Ниже перечислены возможные события отслеживания:

1. breakStart — отслеживает начало рекламной паузы;
2. breakEnd — отслеживает завершение рекламной паузы;
3. error — отслеживает ошибку, возникшую во время рекламной паузы.

В следующем примере показан VMAP-файл, который задает события отслеживания

```xml
    <vmap:VMAP xmlns:vmap="http://www.iab.net/vmap-1.0" version="1.0">
      <vmap:AdBreak breakType="linear" breakId="mypre" timeOffset="start">
        <vmap:AdSource allowMultipleAds="true" followRedirects="true" id="1">
          <vmap:VASTData>
            <!--Inline VAST -->
          </vmap:VASTData>
        </vmap:AdSource>
        <vmap:TrackingEvents>
          <vmap:Tracking event="breakStart">
            http://MyServer.com/breakstart.gif
          </vmap:Tracking>
          <vmap:Tracking event="breakend">
            http://MyServer.com/breakend.gif
          </vmap:Tracking>
          <vmap:Tracking event="error">
            http://MyServer.com/error.gif
          </vmap:Tracking>
        </vmap:TrackingEvents>
      </vmap:AdBreak>
    </vmap:VMAP>
```

Дополнительные сведения об элементе <**TrackingEvents**> и его дочерних элементах приведены в документе http://iab.net/VMAP.pdf.

### <a name="using-a-media-abstract-sequencing-template-mast-file"></a>Использование файла Media Abstract Sequencing Template (MAST)
MAST-файл позволяет указать триггеры, определяющие, когда будет отображаться реклама. Ниже приведен пример MAST-файла, содержащего триггеры для показа отдельных рекламных роликов до просмотра видео, в середине просмотра и после просмотра.

```xml
    <MAST xsi:schemaLocation="http://openvideoplayer.sf.net/mast http://openvideoplayer.sf.net/mast/mast.xsd" xmlns="http://openvideoplayer.sf.net/mast" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
      <triggers>
        <trigger id="preroll" description="preroll every item"  >
          <startConditions>
            <condition type="event" name="OnItemStart" />
          </startConditions>
          <sources>
            <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
              <sources />
            </source>
          </sources>
        </trigger>

        <trigger id="midroll" description="midroll at 15 sec."  >
          <startConditions>
            <condition type="property" name="Position" value="00:00:15.0" operator="GEQ" />
          </startConditions>
          <endConditions>
            <condition type="event" name="OnItemEnd"/>
            <!--This 'resets' the trigger for the next clip-->
          </endConditions>
          <sources>
            <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
              <sources />
            </source>
          </sources>
        </trigger>

        <trigger id="postroll" description="postroll"  >
          <startConditions>
            <condition type="event" name="OnItemEnd"/>
          </startConditions>
          <sources>
            <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
              <sources />
            </source>
          </sources>
        </trigger>
      </triggers>
    </MAST>
```


MAST-файл начинается с элемента **MAST**, в котором содержится один элемент **triggers**. Элемент `<triggers>` содержит один или несколько элементов **trigger**, которые определяют, когда должна воспроизводиться реклама.

В элементе **trigger** содержится элемент **startConditions**, который указывает, когда должно начаться воспроизведение рекламы. Элемент **startConditions** содержит один или несколько элементов `<condition>`. Если каждое значение `<condition>` вычисляется как true, триггер инициируется или отзывается в зависимости от того, содержится ли `<condition>` в элементе **startConditions** или **endConditions** соответственно. При наличии нескольких элементов `<condition>` они обрабатываются как неявное логическое ИЛИ: любое условие, которое истинно, вызовет запуск триггера. Элементы `<condition>` могут быть вложенными. Если предварительно заданы дочерние элементы `<condition>`, они обрабатываются как неявное логическое И: для запуска триггера все условия должны быть истинными. Элемент `<condition>` содержит следующие атрибуты, определяющие условие:

1. **type** — указывает тип условия, событие или свойство;
2. **name** — имя свойства или события, которое будет использоваться при вычислении;
3. **value** — значение, по которому будет вычисляться свойство;
4. **operator** — операция, которая будет использоваться при вычислении: EQ (равно), NEQ (не равно), GTR (больше), GEQ (больше или равно), LT (меньше), LEQ (меньше или равно), MOD (остаток от деления)

В **endConditions** также содержатся элементы `<condition>`. Когда условие получает значение true, триггер сбрасывается. Элемент `<trigger>` также содержит элемент `<sources>`, который содержит один или несколько элементов `<source>`. Элементы `<source>` определяют URI отклика на рекламу и тип отклика. В этом примере для URI задан отклик VAST.

```xml
    <trigger id="postroll" description="postroll"  >
      <startConditions>
        <condition/>
      </startConditions>
      <sources>
        <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
          <sources />
        </source>
      </sources>
    </trigger>
```

### <a name="using-video-player-ad-interface-definition-vpaid"></a>Использование Video Player-Ad Interface Definition (VPAID)
VPAID — это интерфейс API для включения исполняемых блоков рекламы, способных взаимодействовать с видеопроигрывателем. Это позволяет создавать рекламу с высокой степенью интерактивности. Пользователь может взаимодействовать с рекламой, а реклама может отвечать на действия зрителя. Например, в рекламе могут отображаться кнопки, которые позволяют пользователю просматривать дополнительные сведения или более длинную версию рекламы. Видеопроигрыватель должен поддерживать интерфейс VPAID API, а исполняемая реклама должна реализовывать этот API. Когда проигрыватель запрашивает рекламу с сервера рекламы, сервер может отвечать откликом VAST, который содержит рекламу VPAID.

Исполняемая реклама создается в коде, который должен выполняться в среде выполнения, например Adobe Flash ™ или JavaScript, способной функционировать в веб-браузере. Когда сервер рекламы возвращает отклик VAST, содержащий рекламу VPAID, значением атрибута apiFramework в элементе `<MediaFile>` должно быть VPAID. Этот атрибут указывает, что содержащаяся внутри реклама является исполняемой рекламой VPAID. Атрибуту type должен быть присвоен тип MIME исполняемого файла, например "application/x-shockwave-flash" или "application/x-javascript". В следующем фрагменте XML-кода показан элемент `<MediaFile>` из отклика VAST, содержащий исполняемое объявление VPAID.

```xml
    <MediaFiles>
       <MediaFile id="1" delivery="progressive" type="application/x-shockwaveflash"
                  width="640" height="480" apiFramework="VPAID">
           <!-- CDATA wrapped URI to executable ad -->
       </MediaFile>
    </MediaFiles>
```

Исполняемую рекламу можно инициализировать с помощью элемента `<AdParameters>` внутри элементов `<Linear>` или `<NonLinear>` в отклике VAST. Дополнительные сведения об элементе `<AdParameters>` см. в разделе [VAST 3.0](https://www.iab.net/media/file/VASTv3.0.pdf). Дополнительные сведения об интерфейсе VPAID API см. в разделе [VPAID 2.0](https://www.iab.net/media/file/VPAID_2.0_Final_04-10-2012.pdf).

## <a name="implementing-a-windows-or-windows-phone-8-player-with-ad-support"></a>Реализация проигрывателя Windows или Windows Phone 8 с поддержкой рекламы
Платформа проигрывателя Microsoft Media Platform для Windows 8 и Windows Phone 8 содержит набор образцов приложений, в которых демонстрируется реализация приложения видеопроигрывателя с использованием данной платформы. Платформу проигрывателя и образцы можно скачать на странице [Платформа проигрывателя для Windows 8 и Windows Phone 8](https://playerframework.codeplex.com).

При открытии решения Microsoft.PlayerFramework.Xaml.Samples на экране появится ряд папок проекта. В папке Advertising содержится образец кода, относящийся к созданию видеопроигрывателя с поддержкой рекламы. Внутри папки Advertising находятся ряд файлов XAML/cs, каждый из которых показывает отдельный способ вставки рекламы. В следующем списке описаны все эти файлы:

* AdPodPage.xaml — показывает, как отобразить рекламную вставку.
* AdSchedulingPage.xaml — показывает, как планировать расписание рекламы.
* FreeWheelPage.xaml — показывает использование подключаемого модуля FreeWheel для планирования расписания рекламы.
* MastPage.xaml — демонстрирует планирование рекламы с помощью MAST-файла.
* ProgrammaticAdPage.xaml — показывает, как программно задать расписание отображения рекламы в видеоролике.
* ScheduleClipPage.xaml — показывает, как запланировать показ рекламного объявления без VAST-файла.
* VastLinearCompanionPage.xaml — показывает, как вставить линейную и сопутствующую рекламу.
* VastNonLinearPage.xaml — демонстрирует вставку нелинейной рекламы.
* VmapPage.xaml — демонстрирует задание рекламных объявлений с помощью VMAP-файла.

Каждый из этих образцов использует класс MediaPlayer, определенный платформой проигрывателя. В большинстве образцов используются подключаемые модули, которые обеспечивают поддержку различных форматов откликов. Образец ProgrammaticAdPage программно взаимодействует с экземпляром MediaPlayer.

### <a name="adpodpage-sample"></a>Образец AdPodPage
Этот образец использует AdSchedulerPlugin для определения времени отображения рекламы. В этом примере запланировано воспроизведение рекламной вставки через пять секунд. Рекламная вставка (группа рекламных объявлений, отображаемых по порядку) задается в VAST-файле, возвращаемом с сервера рекламы. Код URI VAST-файла указывается в элементе `<RemoteAdSource>`.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">

        <mmppf:MediaPlayer.Plugins>
            <ads:AdSchedulerPlugin>
                <ads:AdSchedulerPlugin.Advertisements>

                    <ads:MidrollAdvertisement Time="00:00:05">
                        <ads:MidrollAdvertisement.Source>
                            <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_adpod.xml" Type="vast"/>
                        </ads:MidrollAdvertisement.Source>
                    </ads:MidrollAdvertisement>

                </ads:AdSchedulerPlugin.Advertisements>
            </ads:AdSchedulerPlugin>
            <ads:AdHandlerPlugin/>
        </mmppf:MediaPlayer.Plugins>
    </mmppf:MediaPlayer>
```

Дополнительные сведения об AdSchedulerPlugin см. в разделе [Реклама на платформе проигрывателя в Windows 8 и Windows Phone 8](https://playerframework.codeplex.com/wikipage?title=Advertising&referringTitle=Windows%208%20Player%20Documentation).

### <a name="adschedulingpage"></a>AdSchedulingPage
В этом примере также используется подключаемый модуль AdSchedulerPlugin. Он планирует воспроизведение трех рекламных объявлений: до, во время и после показа видео. URI VAST-файла каждого из объявлений указывается в элементе `<RemoteAdSource>`.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:PrerollAdvertisement>
                                <ads:PrerollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" Type="vast"/>
                                </ads:PrerollAdvertisement.Source>
                            </ads:PrerollAdvertisement>

                            <ads:MidrollAdvertisement Time="00:00:15">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" Type="vast"/>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                            <ads:PostrollAdvertisement>
                                <ads:PostrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" Type="vast"/>
                                </ads:PostrollAdvertisement.Source>
                            </ads:PostrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="freewheelpage"></a>FreeWheelPage
В этом примере используется подключаемый модуль FreeWheelPlugin. Он задает исходный атрибут, указывающий URI файла SmartXML, который определяет содержимое рекламы, а также сведения о ее расписании.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:FreeWheelPlugin Source="http://smf.blob.core.windows.net/samples/win8/ads/freewheel.xml"/>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="mastpage"></a>MastPage
В этом примере используется MastSchedulerPlugin, который позволяет использовать MAST-файл. Атрибут Source указывает расположение MAST-файла.
```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:MastSchedulerPlugin Source="http://smf.blob.core.windows.net/samples/win8/ads/mast.xml" />
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="programmaticadpage"></a>ProgrammaticAdPage
Этот образец программно взаимодействует с MediaPlayer. Файл ProgrammaticAdPage.xaml создает экземпляр MediaPlayer:

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4"/>
```

Файл ProgrammaticAdPage.xaml.cs создает AdHandlerPlugin, добавляет TimelineMarker для указания времени, когда должна отображаться реклама. Затем он добавляет обработчик для события MarkerReached, который загружает RemoteAdSource, указывающий URI VAST-файла, и воспроизводит рекламу.

```csharp
    public sealed partial class ProgrammaticAdPage : Microsoft.PlayerFramework.Samples.Common.LayoutAwarePage
        {
            AdHandlerPlugin adHandler;

            public ProgrammaticAdPage()
            {
                this.InitializeComponent();
                adHandler = new AdHandlerPlugin();
                player.Plugins.Add(new AdHandlerPlugin());
                player.Markers.Add(new TimelineMarker() { Time = TimeSpan.FromSeconds(5), Type = "myAd" });
                player.MarkerReached += pf_MarkerReached;
            }

            async void pf_MarkerReached(object sender, TimelineMarkerRoutedEventArgs e)
            {
                if (e.Marker.Type == "myAd")
                {
                    var adSource = new RemoteAdSource() { Type = VastAdPayloadHandler.AdType, Uri = new Uri("http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml") };
                    //var adSource = new AdSource() { Type = DocumentAdPayloadHandler.AdType, Payload = SampleAdDocument };
                    var progress = new Progress<AdStatus>();
                    try
                    {
                        await player.PlayAd(adSource, progress, CancellationToken.None);
                    }
                    catch { /* ignore */ }
                }
            }
```

### <a name="scheduleclippage"></a>ScheduleClipPage
Этот образец использует AdSchedulerPlugin для планирования воспроизведения рекламы во время показа видео, указывая WMV-файл, который содержит рекламу.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.cloudapp.net/html5/media/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:MidrollAdvertisement Time="00:00:05">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:AdSource Type="clip">
                                        <ads:AdSource.Payload>
                                            <ads:ClipAdPayload MediaSource="http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_2_000_700_4x3.wmv" MimeType="video/x-ms-wmv" />
                                        </ads:AdSource.Payload>
                                    </ads:AdSource>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="vastlinearcompanionpage"></a>VastLinearCompanionPage
В этом примере показано, как использовать AdSchedulerPlugin для планирования линейной рекламы во время показа видео с сопутствующей рекламой. Элемент `<RemoteAdSource>` указывает расположение VAST-файла.

```xml
    <mmppf:MediaPlayer Grid.Row="1"  x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:MidrollAdvertisement Time="00:00:05">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear_companions.xml" Type="vast"/>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="vastlinearnonlinearpage"></a>VastLinearNonLinearPage
В этом примере используется AdSchedulerPlugin для планирования линейной и нелинейной рекламы. Расположение VAST-файла указывается элементом `<RemoteAdSource>`.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:MidrollAdvertisement Time="00:00:05">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear_nonlinear.xml" Type="vast"/>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="vmappage"></a>VMAPPage
В этом примере используется VmapSchedulerPlugin для планирования рекламных объявлений с помощью VMAP-файла. URI VMAP-файла указывается в атрибуте Source элемента `<VmapSchedulerPlugin>`.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:VmapSchedulerPlugin Source="http://smf.blob.core.windows.net/samples/win8/ads/vmap.xml"/>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

## <a name="implementing-an-ios-video-player-with-ad-support"></a>Реализация видеопроигрывателя с поддержкой рекламы для iOS
Платформа проигрывателя Microsoft Media Platform для iOS содержит набор образцов приложений, в которых демонстрируется реализация приложения видеопроигрывателя с использованием данной платформы. Платформу проигрывателя и образцы можно скачать на странице [Платформа Проигрывателя мультимедиа Azure](https://github.com/CloudMetal/azure-media-player-framework). На странице GitHub есть ссылка на вики-сайт, содержащий дополнительные сведения о платформе проигрывателя, а также введение в пример для проигрывателя: [проигрыватель мультимедиа Azure вики-сайта](https://github.com/CloudMetal/azure-media-player-framework/wiki/How-to-use-Azure-media-player-framework).

### <a name="scheduling-ads-with-vmap"></a>Планирование рекламы с помощью VMAP
Следующий пример демонстрирует планирование рекламы с помощью VMAP-файла.

```csharp
    // How to schedule an Ad using VMAP.
    //First download the VMAP manifest

    if (![framework.adResolver downloadManifest:&manifest withURL:[NSURL URLWithString:@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/vast/PlayerTestVMAP.xml"]])
            {
                [self logFrameworkError];
            }
            else
            {
                // Schedule a list of ads using the downloaded VMAP manifest
                if (![framework scheduleVMAPWithManifest:manifest])
                {
                    [self logFrameworkError];
                }
            }
```

### <a name="scheduling-ads-with-vast"></a>Планирование рекламы с помощью VAST
Следующий пример демонстрирует планирование рекламы VAST с поздней привязкой.


```csharp
    //Example:3 How to schedule a late binding VAST ad.
    // set the start time for the ad
    adLinearTime.startTime = 13;
    adLinearTime.duration = 0;
    // Specify the URI of the VAST file
    NSString *vastAd1=@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/vast/PlayerTestVAST.xml";
    // Create an AdInfo object
     AdInfo *vastAdInfo1 = [[[AdInfo alloc] init] autorelease];
    // set URL to VAST file
    vastAdInfo1.clipURL = [NSURL URLWithString:vastAd1];
    // set running time of ad
    vastAdInfo1.mediaTime = [[[MediaTime alloc] init] autorelease];
    vastAdInfo1.mediaTime.clipBeginMediaTime = 0;
    vastAdInfo1.mediaTime.clipEndMediaTime = 10;
    vastAdInfo1.policy = @"policy for late binding VAST";
    // specify ad type
    vastAdInfo1.type = AdType_Midroll;
    vastAdInfo1.appendTo=-1;
    adIndex = 0;
    // schedule ad
    if (![framework scheduleClip:vastAdInfo1 atTime:adLinearTime forType:PlaylistEntryType_VAST andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

   Следующий пример демонстрирует планирование рекламы VAST с ранней привязкой.

```csharp
    //Example:4 Schedule an early binding VAST ad
    //Download the VAST file
    if (![framework.adResolver downloadManifest:&manifest withURL:[NSURL URLWithString:@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/vast/PlayerTestVAST.xml"]])
    {
        [self logFrameworkError];
    }
    else
    {
        adLinearTime.startTime = 7;
        adLinearTime.duration = 0;

        // Create AdInfo instance
        AdInfo *vastAdInfo2 = [[[AdInfo alloc] init] autorelease];
        vastAdInfo2.mediaTime = [[[MediaTime alloc] init] autorelease];
        vastAdInfo2.policy = @"policy for early binding VAST";
        // specify ad type
        vastAdInfo2.type = AdType_Midroll;
        vastAdInfo2.appendTo=-1;
        // schedule ad
        if (![framework scheduleVASTClip:vastAdInfo2 withManifest:manifest atTime:adLinearTime andGetClipId:&adIndex])
        {
            [self logFrameworkError];
        }
    }
```

Следующий пример демонстрирует вставку рекламы посредством редактирования чернового монтажа (RCE)

```csharp
    //Example:1 How to use RCE.
    // specify manifest for ad content
    NSString *secondContent=@"http://wamsblureg001orig-hs.cloudapp.net/6651424c-a9d1-419b-895c-6993f0f48a26/The%20making%20of%20Microsoft%20Surface-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";

    // specify ad length
    mediaTime.currentPlaybackPosition = 0;
    mediaTime.clipBeginMediaTime = 0;
    mediaTime.clipEndMediaTime = 80;
    // append ad content
    if (![framework appendContentClip:[NSURL URLWithString:secondContent] withMediaTime:mediaTime andGetClipId:&clipId])
    {
        [self logFrameworkError];
    }
```

В следующем примере показано, как запланировать рекламную вставку.

```csharp
    //Example:5 Schedule an ad Pod.
    // Set start time for ad
    adLinearTime.startTime = 23;
    adLinearTime.duration = 0;

    // Specify URL to content
    NSString *adpodSt1=@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/asset-e47b43fd-05dc-4587-ac87-5916439ad07f/Windows%208_%20Cliffjumpers.mp4?st=2012-11-28T16%3A31%3A57Z&se=2014-11-28T16%3A31%3A57Z&sr=c&si=2a6dbb1e-f906-4187-a3d3-7e517192cbd0&sig=qrXYZBekqlbbYKqwovxzaVZNLv9cgyINgMazSCbdrfU%3D";
    // Create an AdInfo instance
    AdInfo *adpodInfo1 = [[[AdInfo alloc] init] autorelease];
    // set URI to ad content
    adpodInfo1.clipURL = [NSURL URLWithString:adpodSt1];
    // Set ad running time
    adpodInfo1.mediaTime = [[[MediaTime alloc] init] autorelease];
    adpodInfo1.mediaTime.clipBeginMediaTime = 0;
    adpodInfo1.mediaTime.clipEndMediaTime = 17;
    adpodInfo1.policy = @"policy for ad pod 1";
    // Set ad type
    adpodInfo1.type = AdType_Midroll;
    adpodInfo1.appendTo=-1;
    adIndex = 0;
    // Schedule ad
    if (![framework scheduleClip:adpodInfo1 atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

В следующем примере показано, как запланировать одноразовую рекламу во время показа видео. Одноразовая реклама воспроизводится только один раз независимо от любых действий пользователя в окне просмотра.

```csharp
    //Example:6 Schedule a single non sticky mid roll Ad
    // specify URL to content
    NSString *oneTimeAd=@"http://wamsblureg001orig-hs.cloudapp.net/5389c0c5-340f-48d7-90bc-0aab664e5f02/Windows%208_%20You%20and%20Me%20Together-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";

    // create an AdInfo instance
    AdInfo *oneTimeInfo = [[[AdInfo alloc] init] autorelease];
    // set URL of ad
    oneTimeInfo.clipURL = [NSURL URLWithString:oneTimeAd];
    oneTimeInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    oneTimeInfo.mediaTime.clipBeginMediaTime = 0;
    oneTimeInfo.mediaTime.clipEndMediaTime = -1;
    oneTimeInfo.policy = @"policy for one-time ad";
    // set ad start time
    adLinearTime.startTime = 43;
    adLinearTime.duration = 0;
    // set ad type
    oneTimeInfo.type = AdType_Midroll;
    // non sticky ad
    oneTimeInfo.deleteAfterPlayed = YES;
    // schedule ad
    if (![framework scheduleClip:oneTimeInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

В следующем примере показано, как запланировать многоразовую рекламу во время показа видео. Многоразовая реклама отображается каждый раз при достижении заданной точки на временной шкале видео.

```csharp
    //Example:7 Schedule a single sticky mid roll Ad
    NSString *stickyAd=@"http://wamsblureg001orig-hs.cloudapp.net/2e4e7d1f-b72a-4994-a406-810c796fc4fc/The%20Surface%20Movement-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    // create AdInfo instance
    AdInfo *stickyAdInfo = [[[AdInfo alloc] init] autorelease];
    // set URI to ad
    stickyAdInfo.clipURL = [NSURL URLWithString:stickyAd];
    stickyAdInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    stickyAdInfo.mediaTime.clipBeginMediaTime = 0;
    stickyAdInfo.mediaTime.clipEndMediaTime = 15;
    stickyAdInfo.policy = @"policy for sticky mid-roll ad";
    // set ad start time
    adLinearTime.startTime = 64;
    adLinearTime.duration = 0;
    // set ad type
    stickyAdInfo.type = AdType_Midroll;
    stickyAdInfo.deleteAfterPlayed = NO;
    // schedule ad
    if (![framework scheduleClip:stickyAdInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

В следующем примере показано, как запланировать рекламу после показа видео.

```csharp
    //Example:8 Schedule Post Roll Ad
    NSString *postAdURLString=@"http://wamsblureg001orig-hs.cloudapp.net/aa152d7f-3c54-487b-ba07-a58e0e33280b/wp-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    // create AdInfo instance
    AdInfo *postAdInfo = [[[AdInfo alloc] init] autorelease];
    postAdInfo.clipURL = [NSURL URLWithString:postAdURLString];
    postAdInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    postAdInfo.mediaTime.clipBeginMediaTime = 0;
    // set ad length
    postAdInfo.mediaTime.clipEndMediaTime = 45;
    postAdInfo.policy = @"policy for post-roll ad";
    // set ad type
    postAdInfo.type = AdType_Postroll;
    adLinearTime.duration = 0;
    if (![framework scheduleClip:postAdInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

В следующем примере показано, как запланировать рекламу до показа видео.

```csharp
    //Example:9 Schedule Pre Roll Ad
    NSString *adURLString = @"http://wamsblureg001orig-hs.cloudapp.net/2e4e7d1f-b72a-4994-a406-810c796fc4fc/The%20Surface%20Movement-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    AdInfo *adInfo = [[[AdInfo alloc] init] autorelease];
    adInfo.clipURL = [NSURL URLWithString:adURLString];
    adInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    adInfo.mediaTime.currentPlaybackPosition = 0;
    adInfo.mediaTime.clipBeginMediaTime = 40; //You could play a portion of an Ad. Yeh!
    adInfo.mediaTime.clipEndMediaTime = 59;
    adInfo.policy = @"policy for pre-roll ad";
    adInfo.appendTo = -1;
    adInfo.type = AdType_Preroll;
    adLinearTime.duration = 0;
    // schedule ad
    if (![framework scheduleClip:adInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

В следующем примере показано, как запланировать рекламное наложение во время показа видео.

```csharp
    // Example10: Schedule a Mid Roll overlay Ad
    NSString *adURLString = @"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/asset-e47b43fd-05dc-4587-ac87-5916439ad07f/Windows%208_%20Cliffjumpers.mp4?st=2012-11-28T16%3A31%3A57Z&se=2014-11-28T16%3A31%3A57Z&sr=c&si=2a6dbb1e-f906-4187-a3d3-7e517192cbd0&sig=qrXYZBekqlbbYKqwovxzaVZNLv9cgyINgMazSCbdrfU%3D";
    //Create AdInfo instance
    AdInfo *adInfo = [[[AdInfo alloc] init] autorelease];
    adInfo.clipURL = [NSURL URLWithString:adURLString];
    adInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    adInfo.mediaTime.currentPlaybackPosition = 0;
    adInfo.mediaTime.clipBeginMediaTime = 0;
    // specify ad length
    adInfo.mediaTime.clipEndMediaTime = 20;
    adInfo.policy = @"policy for mid-roll overlay ad";
    adInfo.appendTo = -1;
    // specify ad type
    adInfo.type = AdType_Midroll;
    // specify ad start time & duration
    adLinearTime.startTime = 300;
    adLinearTime.duration = 20;
    // schedule ad            if (![framework scheduleClip:adInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```


## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
