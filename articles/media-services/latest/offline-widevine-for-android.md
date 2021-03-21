---
title: Потоковая передача Widevine Android в автономном режиме
description: В этом разделе показано, как настроить учетную запись служб мультимедиа Azure версии 3 для автономной потоковой передачи защищенного содержимого Widevine.
services: media-services
keywords: DASH, DRM, режим автономной работы Widevine, ExoPlayer, Android
documentationcenter: ''
author: willzhan
manager: steveng
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: willzhan
ms.custom: devx-track-csharp
ms.openlocfilehash: 4c7adab9c14075e12cf5656f9e798e78ff26ff0b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98955213"
---
# <a name="offline-widevine-streaming-for-android-with-media-services-v3"></a>Автономная потоковая передача Widevine для Android с помощью служб мультимедиа v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Помимо защиты содержимого для автономной потоковой передачи подписка содержимого мультимедиа и службы аренды предоставляют содержимое, которое можно загрузить. Это подходит при отсутствии подключения к Интернету. Возможно, потребуется загрузить содержимое на ваш телефон или планшет для воспроизведения в режиме "в самолете", когда во время полета нет подключения к сети. Дополнительные сценарии, в которых может потребоваться загрузить содержимое:

- Некоторые поставщики содержимого могут запретить доставку лицензий DRM за рамку страны или региона. Если пользователь хочет смотреть содержимое во время поездок за границу, необходимо автономное скачивание.
- В некоторых странах и регионах доступ к Интернету и (или) пропускная способность ограничены. Пользователи могут загрузить содержимое, чтобы иметь возможность смотреть его в достаточно высоком разрешении для удобного просмотра.

[!INCLUDE [Widevine is not available in the GovCloud region.](./includes/widevine-not-available-govcloud.md)]

В этой статье обсуждается реализация воспроизведения в автономном режиме содержимого DASH, защищенного Widevine на устройствах Android. Автономный DRM позволяет создать подписку, модели аренды и приобретения для содержимого, чтобы позволить клиентам ваших служб легко загружать содержимое без подключения к Интернету.

Вы можете создать приложения для проигрывателя Android такими тремя способами:

> [!div class="checklist"]
> * Создание проигрывателя с использованием API Java пакета SDK ExoPlayer.
> * Создание проигрывателя с использованием привязки Xamarin пакета SDK ExoPlayer.
> * Создание проигрывателя с использованием расширения зашифрованных носителей (EME) и расширения источника мультимедиа (MSE) в браузере Chrome для мобильных устройств версии 62 или более поздней.

В статье также содержатся ответы на некоторые распространенные вопросы, связанные с автономной потоковой передачей содержимого, защищенного Widevine.

> [!NOTE]
> В автономной системе DRM оплачивается только один запрос на лицензию при скачивании содержимого. Плата за любые ошибки не взимается.

## <a name="prerequisites"></a>Предварительные условия 

Перед реализацией автономного DRM для Widevine на устройствах Android вам потребуется:

- Ознакомиться с концепциями, реализованными для защиты интернет-содержимого с помощью Widevine DRM. Это подробно рассматривается в следующих документах и примерах:
    - [Проектирование системы для защиты содержимого с несколькими подсистемами DRM и управлением доступом](design-multi-drm-system-with-access-control.md)
    - [Использование динамического шифрования DRM и службы доставки лицензий](protect-with-drm.md)
- Клонировать https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git.

    Вам потребуется изменить код в [Шифровать с помощью DRM, используя .NET](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/EncryptWithDRM), чтобы добавить конфигурации Widevine.  
- Ознакомиться с пакетом SDK ExoPlayer Google для Android, пакетом SDK видеопроигрывателя с открытым исходным кодом, поддерживающего автономное воспроизведение Widevine DRM. 
    - [Пакет SDK ExoPlayer](https://github.com/google/ExoPlayer)
    - [Руководство для разработчиков ExoPlayer](https://google.github.io/ExoPlayer/guide.html)
    - [Блог для разработчиков ExoPlayer](https://medium.com/google-exoplayer)

## <a name="configure-content-protection-in-azure-media-services"></a>Настройка защиты содержимого в Службах мультимедиа Azure

В методе [GetOrCreateContentKeyPolicyAsync](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L189) выполните следующие действия:

1. Укажите, как выполняется авторизация в службе доставки лицензий доставки ключей содержимого. 

    ```csharp
    ContentKeyPolicySymmetricTokenKey primaryKey = new ContentKeyPolicySymmetricTokenKey(tokenSigningKey);
    List<ContentKeyPolicyTokenClaim> requiredClaims = new List<ContentKeyPolicyTokenClaim>()
    {
        ContentKeyPolicyTokenClaim.ContentKeyIdentifierClaim
    };
    List<ContentKeyPolicyRestrictionTokenKey> alternateKeys = null;
    ContentKeyPolicyTokenRestriction restriction 
        = new ContentKeyPolicyTokenRestriction(Issuer, Audience, primaryKey, ContentKeyPolicyRestrictionTokenType.Jwt, alternateKeys, requiredClaims);
    ```
2. Настройте шаблон лицензии Widevine.  

    ```csharp
    ContentKeyPolicyWidevineConfiguration widevineConfig = ConfigureWidevineLicenseTempate();
    ```

3. Создайте ContentKeyPolicyOptions.

    ```csharp
    options.Add(
        new ContentKeyPolicyOption()
        {
            Configuration = widevineConfig,
            Restriction = restriction
        });
    ```

## <a name="enable-offline-mode"></a>Включение автономного режима

Чтобы включить **автономный** режим лицензий Widevine, необходимо настроить [шаблон лицензии Widevine](widevine-license-template-overview.md). В объекте **policy_overrides** для свойства **can_persist** задайте значение **true** (по умолчанию false), как показано в [ConfigureWidevineLicenseTempate](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L563). 

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#ConfigureWidevineLicenseTempate)]

## <a name="configuring-the-android-player-for-offline-playback"></a>Настройка проигрывателя Android для воспроизведения в автономном режиме

Для разработки приложений со встроенным проигрывателем для устройств Android проще всего использовать [пакет SDK Google ExoPlayer](https://github.com/google/ExoPlayer), пакет SDK видеопроигрывателя с открытым кодом. ExoPlayer поддерживает функции, которые сейчас не поддерживаются собственным API MediaPlayer для Android, в том числе протоколы MPEG-ТИРЕ и Microsoft Smooth Streaming.

ExoPlayer версии 2.6 и более поздних версий включает в себя множество классов, поддерживающих воспроизведение Widevine DRM в автономном режиме. В частности класс OfflineLicenseHelper предоставляет служебные функции, упрощающие загрузку, обновление и выдачу автономных лицензий с помощью DefaultDrmSessionManager. Классы, предоставленные в папке library/core/src/main/java/com/google/android/exoplayer2/offline/ пакета SDK, поддерживают загрузку видеосодержимого в автономном режиме.

Указанный ниже перечень классов упрощает автономный режим в пакете SDK ExoPlayer для Android.

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

Во время разработки приложения разработчикам следует использовать [руководство разработчика ExoPlayer](https://google.github.io/ExoPlayer/guide.html) и соответствующий [блог разработчиков](https://medium.com/google-exoplayer). Сейчас компания Google еще не выпустила полностью документированный пример реализации или пример кода для приложения ExoPlayer, поддерживающего автономный режим Widevine, поэтому информация ограничена блогом и руководством для разработчиков. 

### <a name="working-with-older-android-devices"></a>Работа со старыми устройствами Android

Для некоторых старых устройств Android необходимо задать значения для следующих свойств **policy_overrides** (определенных в [шаблоне лицензии Widevine](widevine-license-template-overview.md)): **rental_duration_seconds**, **playback_duration_seconds** и **license_duration_seconds**. Кроме того, для них можно задать значение "ноль", что будет означать неограниченную длительность.  

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

## <a name="faqs"></a>Часто задаваемые вопросы

Дополнительные сведения см. в статье [Widevine FAQ](frequently-asked-questions.md#widevine-streaming-for-android).

## <a name="additional-notes"></a>Дополнительные замечания

Widevine — это служба, которая предоставляется компанией Google Inc. и подпадает под условия предоставления услуг и политику конфиденциальности Google Inc.

## <a name="summary"></a>Итоги

В этой статье рассматриваются способы реализации воспроизведения в автономном режиме содержимого DASH, защищенного Widevine на устройствах Android.  В ней также содержатся ответы на распространенные вопросы, связанные с автономной потоковой передачей содержимого, защищенного Widevine.
