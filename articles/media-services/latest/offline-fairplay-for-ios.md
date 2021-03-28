---
title: Служба мультимедиа v3 автономная FairPlay потоковая передача для iOS
description: В этом разделе приводится обзор и демонстрируется использование служб мультимедиа Azure версии 3 для динамического шифрования содержимого HTTP Live Streaming (HLS) с помощью Apple FairPlay в автономном режиме.
services: media-services
author: willzhan
manager: femila
ms.service: media-services
ms.topic: how-to
ms.date: 03/25/2021
ms.author: inhenkel
ms.openlocfilehash: 0412ef303a301873d437e03e91c58c0a2e8e7e15
ms.sourcegitcommit: c8b50a8aa8d9596ee3d4f3905bde94c984fc8aa2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/28/2021
ms.locfileid: "105640733"
---
# <a name="offline-fairplay-streaming-for-ios-with-media-services-v3"></a>Автономная потоковая передача FairPlay для iOS с помощью служб мультимедиа v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

 Службы мультимедиа Azure предоставляют набор хорошо спроектированных [служб системы защиты содержимого](https://azure.microsoft.com/services/media-services/content-protection/), которые охватывают:

- Microsoft PlayReady
- Google Widevine
    
    Widevine — это служба, которая предоставляется компанией Google Inc. и подпадает под условия предоставления услуг и политику конфиденциальности Google Inc.
- Apple FairPlay
- Шифрование AES-128

Шифрование содержимого "Управление цифровыми правами" (DRM) или AES выполняется динамически по запросу для различных протоколов потоковой передачи. Службы доставки ключей расшифровки AES и лицензии DRM также предоставляются службами мультимедиа.

Помимо защиты содержимого для интерактивной потоковой передачи по различным протоколам потоковой передачи также часто запрашивается автономный режим для защищенного содержимого. Поддержка автономного режима необходима в следующих случаях:

* Воспроизведение, когда подключение к Интернету недоступно, например во время путешествий.
* Некоторые поставщики содержимого могут запретить доставку лицензий DRM за границу страны или региона. Если пользователи хотят просматривать содержимое в поездках за пределами страны или региона, Загрузка в автономном режиме не требуется.
* В некоторых странах и регионах доступ к Интернету и (или) пропускная способность по-прежнему ограничены. Пользователи могут сначала скачать содержимое, чтобы иметь возможность смотреть его в достаточно высоком разрешении для удобного просмотра. В этом случае проблемой является не доступность сети, а ее ограниченная пропускная способность. Для поставщиков ОТТ и платформы интернет-видео (OVP) необходима поддержка автономного режима.

В этой статье рассматривается поддержка автономного режима потоковой передачи FairPlay, предназначенная для устройств под управлением iOS 10 или более поздней версии. Эта функция не поддерживается для других платформ Apple, таких как watchOS, tvOS или Safari на macOS.

> [!NOTE]
> В автономной системе DRM оплачивается только один запрос на лицензию при скачивании содержимого. Плата за любые ошибки не взимается.

## <a name="prerequisites"></a>Предварительные условия

Перед реализацией автономного DRM для FairPlay на устройстве iOS 10+ вам нужно:

* Ознакомиться с интерактивной защитой содержимого FairPlay. 

    - [Требования и конфигурация лицензии Apple FairPlay](fairplay-license-overview.md)
    - [Использование динамического шифрования DRM и службы доставки лицензий](protect-with-drm.md)
    - Пример .NET, который включает в себя настройку потоковой передачи кадров в режиме «в сети»: [конфигурефаирплайполициоптионс](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L505)
* Получить пакет SDK FPS из сети разработчиков Apple. Пакет SDK FPS состоит из двух компонентов:

    - Пакет SDK сервера FPS, который содержит модуль безопасности ключа (KSM), образцы клиентов, спецификацию и набор тестовых векторов.
    - Пакет развертывания FPS, который содержит D function, спецификации вместе с инструкциями о том, как создать сертификат FPS, пользовательский закрытый ключ и секретный код приложения. Apple выдает пакет развертывания FPS только лицензированным поставщикам содержимого.
* Клонировать https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git. 

    Вам потребуется изменить код в репозитории сценариев для [шифрования с помощью DRM при использовании .NET](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/EncryptWithDRM) для добавления конфигураций FairPlay.  

## <a name="configure-content-protection-in-azure-media-services"></a>Настройка защиты содержимого в Службах мультимедиа Azure

В методе [GetOrCreateContentKeyPolicyAsync](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L189) выполните следующие действия.

Раскомментируйте код, который настраивает параметр политики FairPlay.

```csharp
ContentKeyPolicyFairPlayConfiguration fairplayConfig = ConfigureFairPlayPolicyOptions();
```

Кроме того, раскомментируйте код, который добавляет CBCS ContentKeyPolicyOption в список ContentKeyPolicyOptions.

```csharp
options.Add(
    new ContentKeyPolicyOption()
    {
        Configuration = fairplayConfig,
        Restriction = restriction,
        Name = "ContentKeyPolicyOption_CBCS"
    });
```

## <a name="enable-offline-mode"></a>Включение автономного режима

Для включения автономного режима создайте пользовательскую политику StreamingPolicy и используйте ее имя при создании StreamingLocator в [CreateStreamingLocatorAsync](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L561).
 
```csharp
CommonEncryptionCbcs objStreamingPolicyInput= new CommonEncryptionCbcs()
{
    Drm = new CbcsDrmConfiguration()
    {
        FairPlay = new StreamingPolicyFairPlayConfiguration()
        {
            AllowPersistentLicense = true // This enables offline mode
        }
    },
    EnabledProtocols = new EnabledProtocols()
    {
        Hls = true,
        Dash = true // Even though DASH under CBCS is not supported for either CSF or CMAF, HLS-CMAF-CBCS uses DASH-CBCS fragments in its HLS playlist
    },

    ContentKeys = new StreamingPolicyContentKeys()
    {
        // Default key must be specified if keyToTrackMappings is present
        DefaultKey = new DefaultKey()
        {
            Label = "CBCS_DefaultKeyLabel"
        }
    }
}

```

Теперь ваша учетная запись мультимедийных служб настроена на предоставление автономных лицензий FairPlay.

## <a name="sample-ios-player"></a>Пример проигрывателя iOS

Поддержка режима автономной работы FPS доступна только в iOS 10 и более поздних версиях. Пакет SDK сервера FPS (версии 3.0 или более поздней) содержит документ и пример автономного режима FPS. В частности, пакет SDK сервера FPS (версии 3.0 или более поздней версии) содержит следующие два элемента, связанные с автономным режимом:

* Документ: автономное воспроизведение с помощью потоковой передачи FairPlay и HTTP Live Streaming. Apple, 14 сентября 2016 г. В версии 4.0 пакета SDK сервера FPS этот документ объединяется с основным документом FPS.
* Пример кода: пример HLSCatalog (часть пакета SDK для сервера от компании Apple с пакетом управления кадров от а) для автономного режима в пакете SDK для \Fairplay Streaming Streaming Server версии 3.1 \ Девелопмент\клиент\ HLSCatalog_With_FPS \Хлскаталог\. В примере приложения HLSCatalog следующие файлы кода, которые используются для реализации функций автономного режима:

    - Файл кода AssetPersistenceManager.swift: AssetPersistenceManager — это основной класс в этом примере, демонстрирующий:

        - Как управлять загрузкой потоков HLS, таких как API, используемые для запуска и отмены загрузки, а также для удаления имеющихся ресурсов с устройств.
        - Как отслеживать ход выполнения скачивания.
    - AssetListTableViewController.swift и AssetListTableViewCell.swift файлов с кодом: AssetListTableViewController — это основной интерфейс этого примера. Он предоставляет список ресурсов, которые пример может использовать для воспроизведения, скачивания, удаления и отмены загрузки. 

С помощью этих шагов можно настроить работающий проигрыватель iOS. Предположим, что вы начинаете с образца HLSCatalog в версии 4.0.1 пакета SDK сервера FPS. Выполните следующие изменения кода:

В HLSCatalog\Shared\Managers\ContentKeyDelegate.swift реализуйте метод `requestContentKeyFromKeySecurityModule(spcData: Data, assetID: String)` с помощью следующего кода. Пусть drmUr будет переменной, назначенной URL-адресу HLS.

```swift
    var ckcData: Data? = nil
    
    let semaphore = DispatchSemaphore(value: 0)
    let postString = "spc=\(spcData.base64EncodedString())&assetId=\(assetIDString)"
    
    if let postData = postString.data(using: .ascii, allowLossyConversion: true), let drmServerUrl = URL(string: self.drmUrl) {
        var request = URLRequest(url: drmServerUrl)
        request.httpMethod = "POST"
        request.setValue(String(postData.count), forHTTPHeaderField: "Content-Length")
        request.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        request.httpBody = postData
        
        URLSession.shared.dataTask(with: request) { (data, _, error) in
            if let data = data, var responseString = String(data: data, encoding: .utf8) {
                responseString = responseString.replacingOccurrences(of: "<ckc>", with: "").replacingOccurrences(of: "</ckc>", with: "")
                ckcData = Data(base64Encoded: responseString)
            } else {
                print("Error encountered while fetching FairPlay license for URL: \(self.drmUrl), \(error?.localizedDescription ?? "Unknown error")")
            }
            
            semaphore.signal()
            }.resume()
    } else {
        fatalError("Invalid post data")
    }
    
    semaphore.wait()
    return ckcData
```

В HLSCatalog\Shared\Managers\ContentKeyDelegate.swift реализуйте метод `requestApplicationCertificate()`. Эта реализация зависит от того, внедряете ли вы сертификат (только открытый ключ) в устройство или размещаете сертификат в Интернете. Следующая реализация использует сертификат размещенного приложения, используемого в наших тестовых примерах. Пусть certUrl будет переменной, которая содержит URL-адрес сертификата приложения.

```swift
func requestApplicationCertificate() throws -> Data {

        var applicationCertificate: Data? = nil
        do {
            applicationCertificate = try Data(contentsOf: URL(string: certUrl)!)
        } catch {
            print("Error loading FairPlay application certificate: \(error)")
        }
        
        return applicationCertificate
    }
```

Для окончательного интегрированного теста URL-адреса видео и сертификата приложения будут указаны в разделе "Интегрированный тест."

В HLSCatalog\Shared\Resources\Streams.plist добавьте URL-адрес тестового видео. В качестве идентификатора ключа содержимого используйте URL-адрес для приобретения лицензии FairPlay, а протокол skd как уникальное значение.

![Автономные потоки FairPlay приложения iOS](media/offline-fairplay-for-ios/offline-fairplay-ios-app-streams.png)

Используйте собственные URL-адреса тестового видео, приобретения лицензии FairPlay, сертификата приложения (если они настроены). Или же вы можете перейти к следующему разделу, содержащему тестовые образцы.

## <a name="integrated-test"></a>Интегрированный тест

Три тестовых образца в службах мультимедиа охватывают следующие три сценария:

* Защита FPS с видео, аудио и альтернативной звуковой дорожкой.
* Защита FPS с видео и аудио, но без альтернативной звуковой дорожки.
* Защита FPS с видео, но без аудио.

Вы можете найти образцы на этом [демо-сайте](https://aka.ms/poc#22) вместе с соответствующим сертификатом приложения, размещенным в веб-приложении Azure.
При использовании 3-й или 4-й версии примера пакета SDK сервера FPS, если главный плейлист содержит альтернативный аудио-поток, в автономном режиме воспроизводится только аудио. Таким образом, вам нужно убрать альтернативный аудиопоток. Другими словами, второй и третий образцы, перечисленные ранее, работают в интерактивном и автономном режимах. В первом примере звук будет воспроизводиться только в автономном режиме, в то время как интерактивная потоковая передача работает правильно.

## <a name="offline-fairplay-questions"></a>Автономные Fairplay вопросы

См. раздел " [автономные Fairplay вопросы](questions-collection.md#why-does-only-audio-play-but-not-video-during-offline-mode)".
