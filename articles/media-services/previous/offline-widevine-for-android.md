---
title: Настройка учетной записи для автономной потоковой передачи содержимого, защищенного Widevine, в Azure
description: В этой статье показано, как настроить учетную запись служб мультимедиа Azure для автономной потоковой передачи содержимого, защищенного Widevine.
services: media-services
keywords: DASH, DRM, режим автономной работы Widevine, ExoPlayer, Android
documentationcenter: ''
author: IngridAtMicrosoft
manager: steveng
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 3/10/2021
ms.author: willzhan
ms.reviewer: dwgeo
ms.openlocfilehash: b13feeda162f9babe1958402a694b728312ea4e5
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103015409"
---
# <a name="offline-widevine-streaming-for-android"></a>Потоковая передача Widevine для Android (автономный режим)

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector" title1="Выберите версию служб мультимедиа, которую вы используете:"]
> * [Версия 3](../latest/offline-widevine-for-android.md)
> * [Версия 2](offline-widevine-for-android.md)

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

Помимо защиты содержимого для автономной потоковой передачи подписка содержимого мультимедиа и службы аренды предоставляют содержимое, которое можно загрузить. Это подходит при отсутствии подключения к Интернету. Возможно, потребуется загрузить содержимое на ваш телефон или планшет для воспроизведения в режиме "в самолете", когда во время полета нет подключения к сети. Дополнительные сценарии, в которых может потребоваться загрузить содержимое:

- Некоторые поставщики содержимого могут запретить доставку лицензий DRM за рамку страны или региона. Если пользователь хочет смотреть содержимое во время поездок за границу, необходимо автономное скачивание.
- В некоторых странах и регионах доступ к Интернету и (или) пропускная способность ограничены. Пользователи могут загрузить содержимое, чтобы иметь возможность смотреть его в достаточно высоком разрешении для удобного просмотра.

В этой статье обсуждается реализация воспроизведения в автономном режиме содержимого DASH, защищенного Widevine на устройствах Android. Автономный DRM позволяет создать подписку, модели аренды и приобретения для содержимого, чтобы позволить клиентам ваших служб легко загружать содержимое без подключения к Интернету.

Вы можете создать приложения для проигрывателя Android такими тремя способами:

> [!div class="checklist"]
> * Создание проигрывателя с использованием API Java пакета SDK ExoPlayer.
> * Создание проигрывателя с использованием привязки Xamarin пакета SDK ExoPlayer.
> * Создание проигрывателя с использованием расширения зашифрованных носителей (EME) и расширения источника мультимедиа (MSE) в браузере Chrome для мобильных устройств версии 62 или более поздней.

В статье также содержатся ответы на некоторые распространенные вопросы, связанные с автономной потоковой передачей содержимого, защищенного Widevine.

## <a name="requirements"></a>Требования 

Перед реализацией автономного DRM для Widevine на устройствах Android вам потребуется:

- Ознакомиться с концепциями, реализованными для защиты интернет-содержимого с помощью Widevine DRM. Это подробно рассматривается в следующих документах и примерах:
    - [Use Azure Media Services to deliver DRM licenses or AES keys](media-services-deliver-keys-and-licenses.md) (Использование служб мультимедиа Azure для доставки лицензий DRM или ключей AES).
    - [CENC с несколькими компонентами управления цифровыми правами и контролем доступа: пример разработки и реализации в Azure и службах мультимедиа Azure](media-services-cenc-with-multidrm-access-control.md)
    - [Using PlayReady and/or Widevine Dynamic Common Encryption with .NET](https://azure.microsoft.com/resources/samples/media-services-dotnet-dynamic-encryption-with-drm/) (Использование общего динамического шифрования PlayReady и/или Widevine DRM с помощью .NET).
    - [Use Azure Media Services to deliver PlayReady and/or Widevine licenses with .NET](https://azure.microsoft.com/resources/samples/media-services-dotnet-deliver-playready-widevine-licenses/) (Использование служб мультимедиа Azure для предоставления лицензий PlayReady и/или Widevine с помощью .NET).
- Ознакомиться с пакетом SDK ExoPlayer Google для Android, пакетом SDK видеопроигрывателя с открытым исходным кодом, поддерживающего автономное воспроизведение Widevine DRM. 
    - [Пакет SDK ExoPlayer](https://github.com/google/ExoPlayer)
    - [Руководство для разработчиков ExoPlayer](https://google.github.io/ExoPlayer/guide.html)
    - [Блог для разработчиков ExoPlayer](https://medium.com/google-exoplayer)

## <a name="content-protection-configuration-in-azure-media-services"></a>Конфигурация защиты содержимого в службах мультимедиа Azure

Во время настройки защиты Widevine ресурса в службах мультимедиа необходимо создать параметр ContentKeyAuthorizationPolicyOption, который определяет следующие три параметра:

1. Система DRM (Widevine).
2. Параметр ContentKeyAuthorizationPolicyRestriction, указывающий, как при доставке ключей содержимого выполняется авторизация в службе доставки лицензий (открытая авторизация или авторизация маркеров).
3. Шаблон лицензии DRM (Widevine).

Чтобы включить **автономный** режим лицензий Widevine, необходимо настроить [шаблон лицензии Widevine](media-services-widevine-license-template-overview.md). В объекте **policy_overrides** для свойства **can_persist** задайте значение **true** (по умолчанию false). 

В следующем примере кода используется платформа .NET для включения **автономного** режима для лицензий Widevine. Код основан на примере кода .NET, приведенном в примере [Using PlayReady and/or Widevine Dynamic Common Encryption with .NET](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-drm) (Использование общего динамического шифрования PlayReady и (или) Widevine DRM с помощью .NET). 

```
private static string ConfigureWidevineLicenseTemplateOffline(Uri keyDeliveryUrl)
{
    var template = new WidevineMessage
    {
        allowed_track_types = AllowedTrackTypes.SD_HD,
        content_key_specs = new[]
        {
            new ContentKeySpecs
            {
                required_output_protection = new RequiredOutputProtection { hdcp = Hdcp.HDCP_NONE},
                security_level = 1,
                track_type = "SD"
            }
        },
        policy_overrides = new
        {
            can_play = true,
            can_persist = true,
            //can_renew = true,                                //if you set can_renew = false, you do not need renewal_server_url
            //renewal_server_url = keyDeliveryUrl.ToString(),    //not mandatory, renewal_server_url is needed only if license_duration_seconds is set
            can_renew = false,
            //rental_duration_seconds = 1209600,
            //playback_duration_seconds = 1209600,
            //license_duration_seconds = 1209600
        }
        };

    string configuration = Newtonsoft.Json.JsonConvert.SerializeObject(template);
    return configuration;
}
```

## <a name="configuring-the-android-player-for-offline-playback"></a>Настройка проигрывателя Android для воспроизведения в автономном режиме

Для разработки приложений со встроенным проигрывателем для устройств Android проще всего использовать [пакет SDK Google ExoPlayer](https://github.com/google/ExoPlayer), пакет SDK видеопроигрывателя с открытым кодом. ExoPlayer поддерживает функции, которые сейчас не поддерживаются собственным API MediaPlayer для Android, в том числе протоколы MPEG-ТИРЕ и Microsoft Smooth Streaming.

ExoPlayer версии 2.6 и более поздних версий включает в себя множество классов, поддерживающих воспроизведение Widevine DRM в автономном режиме. В частности класс OfflineLicenseHelper предоставляет служебные функции, упрощающие загрузку, обновление и выдачу автономных лицензий с помощью DefaultDrmSessionManager. Классы, предоставленные в папке library/core/src/main/java/com/google/android/exoplayer2/offline/ пакета SDK, поддерживают загрузку видеосодержимого в автономном режиме.

Приведенный ниже перечень классов упрощает автономный режим в пакете SDK ExoPlayer для Android:

- library/core/src/main/java/com/google/android/exoplayer2/drm/OfflineLicenseHelper.java;  
- library/core/src/main/java/com/google/android/exoplayer2/drm/DefaultDrmSession.java;
- library/core/src/main/java/com/google/android/exoplayer2/drm/DefaultDrmSessionManager.java;
- library/core/src/main/java/com/google/android/exoplayer2/drm/DrmSession.java;
- library/core/src/main/java/com/google/android/exoplayer2/drm/ErrorStateDrmSession.java;
- library/core/src/main/java/com/google/android/exoplayer2/drm/ExoMediaDrm.java;
- library/core/src/main/java/com/google/android/exoplayer2/offline/SegmentDownloader.java;
- library/core/src/main/java/com/google/android/exoplayer2/offline/DownloaderConstructorHelper.java; 
- library/core/src/main/java/com/google/android/exoplayer2/offline/Downloader.java;
- library/dash/src/main/java/com/google/android/exoplayer2/source/dash/offline/DashDownloader.java. 

Во время разработки приложения разработчикам следует использовать [руководство разработчика ExoPlayer](https://google.github.io/ExoPlayer/guide.html) и соответствующий [блог разработчиков](https://medium.com/google-exoplayer). На данный момент компания Google еще не выпустила полностью документированный пример реализации или пример кода для приложения ExoPlayer, поддерживающего автономный режим Widevine, поэтому информация ограничена блогом и руководством для разработчиков. 

### <a name="working-with-older-android-devices"></a>Работа со старыми устройствами Android

Для некоторых старых устройств Android необходимо задать значения для следующих свойств **policy_overrides** (определенных в [шаблоне лицензии Widevine](media-services-widevine-license-template-overview.md)): **rental_duration_seconds**, **playback_duration_seconds** и **license_duration_seconds**. Кроме того, для них можно задать значение "ноль", что будет означать неограниченную длительность.  

Значения необходимо задать, чтобы избежать ошибки переполнения целочисленного значения. Дополнительные сведения об этой проблеме приведены на страницах https://github.com/google/ExoPlayer/issues/3150 и https://github.com/google/ExoPlayer/issues/3112. <br/>Если не задать значения явно, для свойств **PlaybackDurationRemaining** и **LicenseDurationRemaining** будут назначены очень большие значения (например, 9223372036854775807, которое является максимально положительным значением для 64-разрядного целого числа). В результате этого срок действия лицензии Widevine истечет и расшифровка не произойдет. 

Эта проблема не возникает в Android 5.0 Lollipop или более поздней версии, так как Android 5.0 является первой версией Android, которая полностью поддерживает ARMv8 ([Advanced RISC Machine](https://en.wikipedia.org/wiki/ARM_architecture)) и 64-разрядные платформы, тогда как Android 4.4 KitKat изначально поддерживает ARMv7 и 32-разрядные платформы, как и другие ранние версии Android.

## <a name="using-xamarin-to-build-an-android-playback-app"></a>Создание приложения для воспроизведения Android с помощью Xamarin

Привязки Xamarin для ExoPlayer доступны по следующим ссылкам:

- [Библиотека привязок Xamarin для библиотеки Google ExoPlayer](https://github.com/martijn00/ExoPlayerXamarin).
- [Привязки Xamarin для ExoPlayer NuGet](https://www.nuget.org/packages/Xam.Plugins.Android.ExoPlayer/).

Ознакомьтесь со следующим обсуждением: [Привязка Xamarin](https://github.com/martijn00/ExoPlayerXamarin/pull/57). 

## <a name="chrome-player-apps-for-android"></a>Приложения проигрывателя Chrome для Android

Начиная с выпуска [Chrome для Android v. 62](https://developers.google.com/web/updates/2017/09/chrome-62-media-updates), поддерживается постоянная лицензия в EME. [Widevine L1](https://developers.google.com/web/updates/2017/09/chrome-62-media-updates#widevine_l1) теперь поддерживается в браузере Chrome для Android. Это позволяет создавать приложения для автономного воспроизведения в браузере Chrome, если у пользователей эта версия Chrome (или более поздняя). 

Кроме того, компания Google выпустила Progressive Web App (PWA) и создала открытый код для него: 

- [Исходный код](https://github.com/GoogleChromeLabs/sample-media-pwa)
- [Версия, размещенная в Google](https://biograf-155113.appspot.com/ttt/episode-2/) (работает только в Chrome версии 62 и более поздних версий на устройствах Android).

Если обновить браузер Chrome версии 62 (или более поздней) для мобильных устройств на телефоне Android и протестировать размещенный выше пример приложения, будет работать как потоковая передача в оперативном режиме, так и воспроизведение в автономном.

Приведенное выше приложение PWA с открытым исходным кодом создается на языке Node.js. Если вы хотите разместить собственную версию на сервере Ubuntu, необходимо учитывать следующие распространенные обнаруженные проблемы, которые могут помешать воспроизведению.

1. Проблема CORS: пример видео в примере приложения размещен в https://storage.googleapis.com/biograf-video-files/videos/. Компания Google настроила CORS для всех своих тестовых примеров, размещенных в контейнере Google Cloud Storage. Они обслуживаются с заголовками CORS, где явно указана запись CORS `https://biograf-155113.appspot.com` (домен, в котором компания Google разместила свой пример), препятствующая доступу с других сайтов. В случае попытки вы увидите следующую ошибку HTTP: `Failed to load https://storage.googleapis.com/biograf-video-files/videos/poly-sizzle-2015/mp4/dash.mpd: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'https:\//13.85.80.81:8080' is therefore not allowed access. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.`
2. Проблема с сертификатом: начиная с версии 58 Chrome для EME для Widevine требуется протокол HTTPS. Таким образом, необходимо разместить пример приложения по протоколу HTTPS с сертификатом X509. Обычный тестовый сертификат не подходит из-за следующих требований. Вам необходимо получить сертификат, который удовлетворяет следующим минимальным требованиям:
    - Для Chrome и Firefox требуется, чтобы в сертификате был параметр "Альтернативное имя субъекта" (SAN).
    - Сертификат должен быть подписан доверенным центром сертификации, самозаверяющий сертификат разработки не подходит.
    - Сертификат должен иметь имя CN, соответствующее DNS-имени веб-сервера или шлюза.

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="question"></a>Вопрос

Как можно предоставить постоянные лицензии (с поддержкой автономного режима) для некоторых клиентов или пользователей и непостоянные лицензии (без поддержки автономного режима) для остальных пользователей? Нужно ли дублировать содержимое и использовать отдельный ключ содержимого?

### <a name="answer"></a>Ответ
Не нужно копировать содержимое. Можно просто использовать одну копию содержимого и одну ContentKeyAuthorizationPolicy, но два отдельных Контенткэйаусоризатионполициоптион:

1. IContentKeyAuthorizationPolicyOption 1: использует постоянную лицензию и ContentKeyAuthorizationPolicyRestriction 1, которая содержит утверждение, например license_type = "persistent".
2. IContentKeyAuthorizationPolicyOption 2: использует непостоянную лицензию и ContentKeyAuthorizationPolicyRestriction 2, которая содержит утверждение, например license_type = "неустойчивая"

Таким образом при поступлении запроса на лицензию из клиентского приложения или от запроса на лицензию нет никаких различий. Однако для разных конечных пользователей и устройств STS должна иметь бизнес-логику, которая будет выдавать разные токены JWT, содержащие различные утверждения (один из указанных выше двух license_type). Значение утверждений в токене JWT будет использоваться для определения на основе службы лицензирования того, какой тип лицензии выдать: постоянный или непостоянный.

Это означает, что в службе токенов безопасности (STS) должна быть бизнес-логика и информация о клиенте или устройстве для добавления соответствующего значения утверждения в токен.

### <a name="question"></a>Вопрос

Для уровней безопасности Widevine в документации, посвященной [архитектуре DRM Google Widevine](https://storage.googleapis.com/wvdocs/Widevine_DRM_Architecture_Overview.pdf) , он определяет три разных уровня безопасности. Однако в [документации по службам мультимедиа Azure в шаблоне лицензии Widevine](./media-services-widevine-license-template-overview.md) описаны пять различных уровней безопасности. Что такое связь или сопоставление между двумя различными наборами уровней безопасности?

### <a name="answer"></a>Ответ

В [обзоре архитектуры DRM Widevine](https://storage.googleapis.com/wvdocs/Widevine_DRM_Architecture_Overview.pdf)для Google определяются следующие три уровня безопасности:

1.  Уровень безопасности 1. Вся обработка, шифрование содержимого и управление им выполняется в доверенной среде выполнения (TEE). В некоторых моделях реализации обработка безопасности может выполняться в разных микросхемах.
2.  Уровень безопасности 2. Шифрование выполняется (но не обработка видео) в TEE: расшифрованные буферы возвращаются в домен приложения и обрабатываются с помощью отдельного аппаратного или программного обеспечения видео. На уровне 2 данные шифрования по-прежнему обрабатываются только в пределах TEE.
3.  Уровень безопасности 3. На устройстве нет TEE. Для защиты сведений шифрования и расшифрованного содержимого в операционной системе узла можно предпринять соответствующие меры. Реализация уровня 3 может также включать механизм шифрования оборудования, но при этом повышается только уровень производительности, а не безопасность.

В то же время в [документации по службам мультимедиа Azure в шаблоне лицензии Widevine](./media-services-widevine-license-template-overview.md) свойство security_level объекта content_key_specs может иметь пять различных значений (требования к надежности клиента для воспроизведения):

1.  Требуется программное шифрование методом белого ящика.
2.  Требуется шифрование ПО и скрытый декодер.
3.  Материал ключа и операции шифрования должны быть выполнены в резервной TEE оборудования.
4.  Операции шифрования и расшифровки содержимого должны быть выполнены в резервной TEE оборудования.
5.  Шифрование, расшифровка и обработка всех носителей (сжатых и несжатых) должны быть выполнены в резервной TEE оборудования.

Оба уровня безопасности определяются с помощью Google Widevine. Разница заключается в уровнях использования: уровень архитектуры или уровень API. В API Widevine используется пять уровней безопасности. Объект content_key_specs, который содержит свойство security_level, десериализуется и передается службе глобальной доставки Widevine с помощью службы лицензий Widevine служб мультимедиа Azure. В следующей таблице показано сопоставление между двумя наборами уровней безопасности.

| **Уровни безопасности, определенные в архитектуре Widevine** |**Уровни безопасности, используемые в API Widevine**|
|---|---| 
| **Уровень безопасности 1**. Вся обработка, шифрование и управление содержимым выполняются в доверенной среде выполнения (tee). В некоторых моделях реализации обработка безопасности может выполняться в разных микросхемах.|**security_level=5**. Шифрование, расшифровка и обработка всех носителей (сжатых и несжатых) должны быть выполнены в резервной TEE оборудования.<br/><br/>**security_level=4**. Операции шифрования и расшифровки содержимого должны быть выполнены в резервной TEE оборудования.|
**Уровень безопасности 2**. Шифрование выполняется (но не обработка видео) в TEE: расшифрованные буферы возвращаются в домен приложения и обрабатываются с помощью отдельного аппаратного или программного обеспечения видео. На уровне 2 данные шифрования по-прежнему обрабатываются только в пределах TEE.| **security_level=3**. Материал ключа и операции шифрования должны быть выполнены в резервной TEE оборудования. |
| **Уровень безопасности 3**. На устройстве нет TEE. Для защиты сведений шифрования и расшифрованного содержимого в операционной системе узла можно предпринять соответствующие меры. Реализация уровня 3 может также включать механизм шифрования оборудования, но при этом повышается только уровень производительности, а не безопасность. | **security_level=2**. Требуется шифрование ПО и скрытый декодер.<br/><br/>**security_level=1**. Требуется программное шифрование методом белого ящика.|

### <a name="question"></a>Вопрос

Почему для загрузки содержимого требуется так много времени?

### <a name="answer"></a>Ответ

Увеличить скорости загрузки можно двумя способами.

1.  Включите CDN, чтобы пользователи с большей долей вероятности нажимали CDN вместо источника или конечной точки потоковой передачи для загрузки содержимого. Если пользователь нажимает конечную точку потоковой передачи, каждый сегмент HLS или фрагмент DASH динамически упаковывается и зашифровывается. Хотя эта задержка измеряется в миллисекундах для каждого сегмента или фрагмента, если длительность видео составляет один час, суммарная задержка может быть большой, из-за чего увеличивается время загрузки.
2.  Предоставьте пользователям возможность загружать отдельные уровни качества видео и звуковые дорожки, а не все содержимое. Для автономного режима нет смысла загружать все уровни качества. Это достигается двумя способами.
    1.  Управляется клиентом: пользователь выбирает уровень качества видео и звуковые дорожки для загрузки либо в проигрывателе приложения это происходит автоматически.
    2.  Управляется службой: в службах мультимедиа Azure можно использовать функцию динамического манифеста, чтобы создать (глобальный) фильтр, который ограничивает список воспроизведения HLS или DASH MPD до одного уровня качества видео и выбранных звуковых дорожек. URL-адрес загрузки, предоставленный пользователям, будет содержать этот фильтр.

## <a name="additional-notes"></a>Дополнительные замечания

* Widevine — это служба, которая предоставляется компанией Google Inc. и подпадает под условия предоставления услуг и политику конфиденциальности Google Inc.

## <a name="summary"></a>Итоги

В этой статье рассматриваются способы реализации воспроизведения в автономном режиме содержимого DASH, защищенного Widevine на устройствах Android.  В ней также содержатся ответы на распространенные вопросы, связанные с автономной потоковой передачей содержимого, защищенного Widevine.
