---
title: Защита содержимого HLS в Azure с помощью Apple FairPlay в автономном режиме | Документация Майкрософт
description: В этом разделе приводятся общие сведения и показывается, как использовать службы мультимедиа Azure для динамического шифрования содержимого HTTP Live Streaming (HLS) с использованием Apple FairPlay в автономном режиме.
services: media-services
keywords: HLS, DRM, потоковая передача FairPlay (FPS), Offline, iOS 10
documentationcenter: ''
author: IngridAtMicrosoft
manager: steveng
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: willzhan
ms.reviewer: dwgeo
ms.custom: devx-track-csharp
ms.openlocfilehash: 2d05810da6abfd33ff6605fa7a6f3bfb6a0c202b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103009578"
---
# <a name="offline-fairplay-streaming-for-ios"></a>Потоковая передача FairPlay в автономном режиме для iOS

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector" title1="Выберите версию служб мультимедиа, которую вы используете:"]
> * [Версия 3](../latest/offline-fairplay-for-ios.md)
> * [Версия 2](media-services-protect-hls-with-offline-fairplay.md)

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

Службы мультимедиа Azure предоставляют набор хорошо спроектированных [служб системы защиты содержимого](https://azure.microsoft.com/services/media-services/content-protection/), которые охватывают:

- Microsoft PlayReady
- Google Widevine
- Apple FairPlay
- Шифрование AES-128

Шифрование содержимого "Управление цифровыми правами" (DRM) или AES выполняется динамически по запросу для различных протоколов потоковой передачи. Службы доставки ключей расшифровки AES и лицензии DRM также предоставляются службами мультимедиа.

Помимо защиты содержимого для интерактивной потоковой передачи по различным протоколам потоковой передачи также часто запрашивается автономный режим для защищенного содержимого. Поддержка автономного режима необходима в следующих случаях:

* Воспроизведение, когда подключение к Интернету недоступно, например во время путешествий.
* Некоторые поставщики содержимого могут запретить доставку лицензий DRM за границу страны или региона. Если пользователи хотят просматривать содержимое в поездках за пределами страны или региона, Загрузка в автономном режиме не требуется.
* В некоторых странах и регионах доступ к Интернету и (или) пропускная способность по-прежнему ограничены. Пользователи могут сначала скачать содержимое, чтобы иметь возможность смотреть его в достаточно высоком разрешении для удобного просмотра. В этом случае проблемой является не доступность сети, а ее ограниченная пропускная способность. Для поставщиков ОТТ и платформы интернет-видео (OVP) необходима поддержка автономного режима.

В этой статье рассматривается поддержка автономного режима потоковой передачи FairPlay, предназначенная для устройств под управлением iOS 10 или более поздней версии. Эта функция не поддерживается для других платформ Apple, таких как watchOS, tvOS или Safari на macOS.

## <a name="preliminary-steps"></a>Предварительные действия
Перед реализацией автономного DRM для FairPlay на устройстве iOS 10+ вам нужно:

* Ознакомиться с интерактивной защитой содержимого FairPlay. Дополнительные сведения см. в следующих статьях и образцах:

    - [Apple FairPlay Streaming for Azure Media Services generally available](https://azure.microsoft.com/blog/apple-FairPlay-streaming-for-azure-media-services-generally-available/) (Общедоступная версия Apple FairPlay Streaming для служб мультимедиа Azure).
    - [Защита содержимого HLS с помощью Apple FairPlay или Microsoft PlayReady](./media-services-protect-hls-with-fairplay.md).
    - [Образец для интерактивной потоковой передачи FPS](https://azure.microsoft.com/resources/samples/media-services-dotnet-dynamic-encryption-with-FairPlay/).

* Получить пакет SDK FPS из сети разработчиков Apple. Пакет SDK FPS состоит из двух компонентов:

    - Пакет SDK сервера FPS, который содержит модуль безопасности ключа (KSM), образцы клиентов, спецификацию и набор тестовых векторов.
    - Пакет развертывания FPS, который содержит D function, спецификации вместе с инструкциями о том, как создать сертификат FPS, пользовательский закрытый ключ и секретный код приложения. Apple выдает пакет развертывания FPS только лицензированным поставщикам содержимого.

## <a name="configuration-in-media-services"></a>Конфигурация в службах мультимедиа
Для конфигурации автономного режима FPS с помощью [пакета SDK служб мультимедиа для .NET](https://www.nuget.org/packages/windowsazure.mediaservices) используйте версии 4.0.0.4 (или новее) пакета SDK для .NET, которая предоставляет необходимый API для настройки автономного режима FPS.
Вам также нужен рабочий код для настройки защиты содержимого в интерактивном режиме для FPS. После получения кода для настройки защиты содержимого в оперативном режиме для FPS вам понадобятся только следующие два изменения.

## <a name="code-change-in-the-fairplay-configuration"></a>Изменение кода в конфигурации FairPlay
дмедикиПервое изменение — определить логическое значение для включения автономного режима, objDRMSettings.EnableOfflineMode, которое является истинным при включении автономного сценария DRM. В зависимости от этого индикатора сделайте следующее изменение конфигурации FairPlay:

```csharp
if (objDRMSettings.EnableOfflineMode)
    {
        FairPlayConfiguration = Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(
            objX509Certificate2,
            pfxPassword,
            pfxPasswordId,
            askId,
            iv, 
            RentalAndLeaseKeyType.PersistentUnlimited,
            0x9999);
    }
    else
    {
        FairPlayConfiguration = Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(
            objX509Certificate2,
            pfxPassword,
            pfxPasswordId,
            askId,
            iv);
    }
```

## <a name="code-change-in-the-asset-delivery-policy-configuration"></a>Изменение кода в конфигурации политики доставки ресурсов
Второе изменение — добавить третий ключ в Dictionary<AssetDeliveryPolicyConfigurationKey, string>.
Добавьте AssetDeliveryPolicyConfigurationKey, как показано ниже:
 
```csharp
// FPS offline mode
    if (drmSettings.EnableOfflineMode)
    {
        objDictionary_AssetDeliveryPolicyConfigurationKey.Add(AssetDeliveryPolicyConfigurationKey.AllowPersistentLicense, "true");
        Console.WriteLine("FPS OFFLINE MODE: AssetDeliveryPolicyConfigurationKey.AllowPersistentLicense added into asset delivery policy configuration.");
    }

    // for IAssetDelivery for FPS
    IAssetDeliveryPolicy objIAssetDeliveryPolicy = objCloudMediaContext.AssetDeliveryPolicies.Create(
            drmSettings.AssetDeliveryPolicyName,
            AssetDeliveryPolicyType.DynamicCommonEncryptionCbcs,
            AssetDeliveryProtocol.HLS,
            objDictionary_AssetDeliveryPolicyConfigurationKey);
```

После выполнения этого шага <Dictionary_AssetDeliveryPolicyConfigurationKey,>string в политике доставки ресурсов FPS содержит следующие три записи:

* AssetDeliveryPolicyConfigurationKey.FairPlayBaseLicenseAcquisitionUrl или AssetDeliveryPolicyConfigurationKey.FairPlayLicenseAcquisitionUrl в зависимости от таких факторов, как используемый FPS KSM или сервер ключей, и используете ли вы повторно одну и ту же политику доставки ресурсов в нескольких ресурсах.
* AssetDeliveryPolicyConfigurationKey.CommonEncryptionIVForCbcs.
* AssetDeliveryPolicyConfigurationKey.AllowPersistentLicense.

Теперь ваша учетная запись мультимедийных служб настроена на предоставление автономных лицензий FairPlay.

## <a name="sample-ios-player"></a>Пример проигрывателя iOS
Поддержка режима автономной работы FPS доступна только в iOS 10 и более поздних версиях. Пакет SDK сервера FPS (версии 3.0 или более поздней) содержит документ и пример автономного режима FPS. В частности, пакет SDK сервера FPS (версии 3.0 или более поздней версии) содержит следующие два элемента, связанные с автономным режимом:

* Документ: автономное воспроизведение с помощью потоковой передачи FairPlay и HTTP Live Streaming. Apple, 14 сентября 2016 г. В версии 4.0 пакета SDK сервера FPS этот документ объединяется с основным документом FPS.
* Пример кода: образец HLSCatalog для автономного режима FPS в \FairPlay Streaming Server SDK version 3.1\Development\Client\HLSCatalog_With_FPS\HLSCatalog\. В примере приложения HLSCatalog следующие файлы кода, которые используются для реализации функций автономного режима:

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

![Автономные потоки FairPlay приложения iOS](media/media-services-protect-hls-with-offline-FairPlay/media-services-offline-FairPlay-ios-app-streams.png)

Используйте собственные URL-адреса тестового видео, приобретения лицензии FairPlay, сертификата приложения (если они настроены). Или же вы можете перейти к следующему разделу, содержащему тестовые образцы.

## <a name="integrated-test"></a>Интегрированный тест
Три тестовых образца в службах мультимедиа охватывают следующие три сценария:

* Защита FPS с видео, аудио и альтернативной звуковой дорожкой.
* Защита FPS с видео и аудио, но без альтернативной звуковой дорожки.
* Защита FPS с видео, но без аудио.

Вы можете найти образцы на этом [демо-сайте](https://aka.ms/poc#22) вместе с соответствующим сертификатом приложения, размещенным в веб-приложении Azure.
При использовании 3-й или 4-й версии примера пакета SDK сервера FPS, если главный плейлист содержит альтернативный аудио-поток, в автономном режиме воспроизводится только аудио. Таким образом, вам нужно убрать альтернативный аудиопоток. Другими словами, второй и третий образцы, перечисленные ранее, работают в интерактивном и автономном режимах. В первом примере звук будет воспроизводиться только в автономном режиме, в то время как интерактивная потоковая передача работает правильно.

## <a name="faq"></a>Вопросы и ответы
Следующие ответы на часто задаваемые вопросы помогут устранить неполадки:

- **Почему в автономном режиме воспроизводится только звук, но нет видео?** Такое поведение, по-видимому, связано с дизайном примера приложения. При наличии альтернативной звуковой дорожки (например, для HLS) во время работы в автономном режиме как для iOS 10, так и для iOS 11 по умолчанию используется альтернативная звуковая дорожка. Чтобы компенсировать это поведение в автономном режиме, удалите из потока альтернативную звуковую дорожку. Чтобы сделать это для служб мультимедиа, добавьте динамический фильтр манифеста audio-only=false. Другими словами URL-адрес HLS оканчивается на .ism/manifest(format=m3u8-aapl,audio-only=false). 
- **Почему после добавления audio-only=false в автономном режиме по-прежнему воспроизводится аудио без видео?** Содержимое может быть кэшировано в зависимости от структуры ключа кэша сети доставки содержимого (CDN). Очистите кэш.
- **Режим автономной работы FPS также поддерживается на iOS 11 в дополнение к iOS 10?** Да. Автономный режим FPS поддерживается в iOS 10 и в iOS 11.
- **Почему я не могу найти документ автономного воспроизведения с помощью потоковой передачи FairPlay и HTTP Live Streaming в пакете SDK сервера FPS?** Начиная с 4-й версии пакета SDK сервера FPS, этот документ был объединен с руководством по потоковому программированию FairPlay.
- **Что делает последний параметр для в следующем интерфейсе API для автономного режима?**
`Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(objX509Certificate2, pfxPassword, pfxPasswordId, askId, iv, RentalAndLeaseKeyType.PersistentUnlimited, 0x9999);`

    Документацию по этому API см. в статье [FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration Method](/dotnet/api/microsoft.windowsazure.mediaservices.client.fairplay.fairplayconfiguration.createserializedfairplayoptionconfiguration) (Метод FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration). Параметр представляет продолжительность автономной аренды, а вторая — как единицу.
- **Какова структура скачанного или автономного файла на устройствах iOS?** Структура скачанного файла на устройстве iOS выглядит как на следующем снимке экрана. В папке `_keys` хранятся скачанные лицензии FPS, один файл хранилища для каждого узла службы лицензий. В папке `.movpkg` хранится аудио- и видеосодержимое. Первая папка, имя которой заканчивается дефисом с числами, содержит видео. Числовое значение равно значению PeakBandwidth для представлений видео. Вторая папка с именем, заканчивающимся тире, за которым следует 0, содержит аудио. Третья папка с именем "Данные" содержит главный список воспроизведения содержимого FPS. Наконец, boot.xml предоставляет полное описание содержимого папки `.movpkg`. 

![Структура файла примера приложения автономного FairPlay iOS](media/media-services-protect-hls-with-offline-FairPlay/media-services-offline-FairPlay-file-structure.png)

Пример файла boot.xml:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<HLSMoviePackage xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://apple.com/IMG/Schemas/HLSMoviePackage" xsi:schemaLocation="http://apple.com/IMG/Schemas/HLSMoviePackage /System/Library/Schemas/HLSMoviePackage.xsd">
  <Version>1.0</Version>
  <HLSMoviePackageType>PersistedStore</HLSMoviePackageType>
  <Streams>
    <Stream ID="1-4DTFY3A3VDRCNZ53YZ3RJ2NPG2AJHNBD-0" Path="1-4DTFY3A3VDRCNZ53YZ3RJ2NPG2AJHNBD-0" NetworkURL="https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/QualityLevels(127000)/Manifest(aac_eng_2_127,format=m3u8-aapl)">
      <Complete>YES</Complete>
    </Stream>
    <Stream ID="0-HC6H5GWC5IU62P4VHE7NWNGO2SZGPKUJ-310656" Path="0-HC6H5GWC5IU62P4VHE7NWNGO2SZGPKUJ-310656" NetworkURL="https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/QualityLevels(161000)/Manifest(video,format=m3u8-aapl)">
      <Complete>YES</Complete>
    </Stream>
  </Streams>
  <MasterPlaylist>
    <NetworkURL>https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/manifest(format=m3u8-aapl,audio-only=false)</NetworkURL>
  </MasterPlaylist>
  <DataItems Directory="Data">
    <DataItem>
      <ID>CB50F631-8227-477A-BCEC-365BBF12BCC0</ID>
      <Category>Playlist</Category>
      <Name>master.m3u8</Name>
      <DataPath>Playlist-master.m3u8-CB50F631-8227-477A-BCEC-365BBF12BCC0.data</DataPath>
      <Role>Master</Role>
    </DataItem>
  </DataItems>
</HLSMoviePackage>
```

## <a name="additional-notes"></a>Дополнительные замечания

* Widevine — это служба, которая предоставляется компанией Google Inc. и подпадает под условия предоставления услуг и политику конфиденциальности Google Inc.

## <a name="summary"></a>Итоги
Этот документ содержит следующие шаги и информацию, с помощью которых можно реализовать автономный режим FPS:

* Конфигурация защиты содержимого служб мультимедиа с помощью API служб мультимедиа для .NET настраивает динамическое шифрование FairPlay и доставку лицензии FairPlay в службы мультимедиа.
* Проигрыватель iOS на основе примера из пакета SDK сервера FPS настраивает проигрыватель iOS, который может воспроизводить содержимое FPS либо в режиме интерактивной потоковой передачи, либо в автономном режиме.
* Примеры видео FPS используются для тестирования автономного и интерактивного режима потоковой передачи.
* В разделе "Часто задаваемые вопросы" содержатся ответы на вопросы об автономном режиме FPS.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]
