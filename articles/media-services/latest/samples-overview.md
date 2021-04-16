---
title: Примеры для Служб мультимедиа версии 3
description: В этой статье приведен список всех доступных примеров для Служб мультимедиа версии 3, упорядоченных по методу и пакету SDK.  Примеры включают в себя .NET, Node.JS, Python и Java, а также REST с Postman.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: overview
ms.date: 03/24/2021
ms.author: inhenkel
ms.openlocfilehash: 1d827d734c434204ff6b7ec60d27e507ae626abd
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/09/2021
ms.locfileid: "107227693"
---
# <a name="media-services-v3-samples"></a>Примеры для Служб мультимедиа версии 3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В этой статье приведен список всех доступных примеров для Служб мультимедиа, упорядоченных по методу и пакету SDK.  Примеры включают в себя .NET, Node.JS, Python и Java, а также REST с Postman.

## <a name="rest-postman-collection"></a>Коллекция REST Postman

Примеры [REST Postman](https://github.com/Azure-Samples/media-services-v3-rest-postman) включают в себя среду и коллекцию Postman, чтобы вы могли выполнять импорт в клиент Postman.

## <a name="samples-by-sdk"></a>Примеры для пакета SDK

Ниже приведены описание и ссылки на примеры, которые будет можно найти на каждой из вкладок.

## <a name="net"></a>[.NET](#tab/net/)

| Папка | Описание |
|-------------|-------------|
| [VideoEncoding/EncodingWithMESPredefinedPreset](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/EncodingWithMESPredefinedPreset)|Отправка задания с помощью встроенной предустановки и входного URL-адреса HTTP, публикация выходного ресурса для потоковой передачи и загрузка результатов для проверки.|
| [VideoEncoding/EncodingWithMESCustomPreset_H264](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/EncodingWithMESCustomPreset_H264)|Отправка задания с помощью настраиваемой предустановки кодирования H.264 и входного URL-адреса HTTP, публикация выходного ресурса для потоковой передачи и загрузка результатов для проверки.|
| [VideoEncoding/EncodingWithMESCustomPreset_HEVC](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/EncodingWithMESCustomPreset_HEVC)|Отправка задания с помощью настраиваемой предустановки кодирования HEVC и входного URL-адреса HTTP, публикация выходного ресурса для потоковой передачи и загрузка результатов для проверки.|
| [VideoEncoding/EncodingWithMESCustomStitchTwoAssets](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/EncodingWithMESCustomStitchTwoAssets)|Отправка задания с помощью JobInputSequence для объединения 2 или более ресурсов, которые можно обрезать по времени начала или окончания. Полученный в результате кодированный файл представляет собой единое видео со всеми ресурсами, объединенными вместе.  В примере также публикуется выходной ресурс для потоковой передачи и будут загружены результаты для проверки.|
| [VideoEncoding/EncodingWithMESCustomPresetAndSprite](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoEncoding/EncodingWithMESCustomPresetAndSprite)|Отправка задания с помощью настраиваемой предустановки со спрайтом эскиза и входным URL-адресом HTTP, публикация выходного ресурса для потоковой передачи и загрузка результатов для проверки.|
| [Live/LiveEventWithDVR](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/Live/LiveEventWithDVR)|Создание LiveEvent с полным архивом длительностью до 25 часов и фильтра по ресурсу с 5-минутным окном DVR. Использование фильтра для создания указателя для потоковой передачи.|
| [VideoAnalytics/VideoAnalyzer](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/VideoAnalytics/VideoAnalyzer)|Создание преобразования анализатора видео, отправка видеофайла во входной ресурс, отправка задания с преобразованием и загрузка результатов для проверки.|
| [AudioAnalytics/AudioAnalyzer](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/AudioAnalytics/AudioAnalyzer)|Создание преобразования в анализаторе звука, отправка файла мультимедиа во входной ресурс, отправка задания с преобразованием и загрузка результатов для проверки.|
| [ContentProtection/BasicAESClearKey](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/BasicAESClearKey)|Создание преобразования с помощью встроенной предустановки AdaptiveStreaming, отправка задания, создание ContentKeyPolicy с помощью секретного ключа, связывание ContentKeyPolicy с StreamingLocator, получение маркера и печать URL-адреса для воспроизведения в проигрывателе мультимедиа Azure. Когда поток запрашивается проигрывателем, Службы мультимедиа используют указанный ключ для динамического шифрования содержимого с помощью AES-128, а Проигрыватель мультимедиа Azure использует маркер для расшифровки.|
| [ContentProtection/BasicWidevine](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/BasicWidevine)|Создание преобразования с помощью встроенной предустановки AdaptiveStreaming, отправка задания, создание ContentKeyPolicy с конфигурацией Widevine с помощью секретного ключа, связывание ContentKeyPolicy с StreamingLocator, получение маркера и печать URL-адреса для воспроизведения в проигрывателе Widevine. Когда пользователь запрашивает содержимое, защищенное с помощью Widevine, приложение проигрывателя, в свою очередь, запрашивает лицензию из службы лицензий Служб мультимедиа. Если приложение проигрывателя авторизовано, служба лицензий служб мультимедиа выдает лицензию для проигрывателя. В лицензии Widevine содержится ключ расшифровки, который может использоваться клиентским проигрывателем для расшифровки и потоковой передачи содержимого.|
| [ContentProtection/BasicPlayReady](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/BasicPlayReady)|Создание преобразования с помощью встроенной предустановки AdaptiveStreaming, отправка задания, создание ContentKeyPolicy с конфигурацией PlayReady с помощью секретного ключа, связывание ContentKeyPolicy с StreamingLocator, получение маркера и печать URL-адреса для воспроизведения в Проигрывателе мультимедиа Azure. Когда пользователь запрашивает содержимое, защищенное с помощью PlayReady, приложение проигрывателя, в свою очередь, запрашивает лицензию из службы лицензий Служб мультимедиа. Если приложение проигрывателя авторизовано, служба лицензий служб мультимедиа выдает лицензию для проигрывателя. Лицензия PlayReady содержит ключ расшифровки, который может использоваться проигрывателем клиента для расшифровки и потоковой передачи контента.|
| [ContentProtection/OfflinePlayReadyAndWidevine](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/ContentProtection/OfflinePlayReadyAndWidevine)|Динамическое шифрование содержимого с помощью PlayReady и Widevine DRM, воспроизведение содержимого без запроса лицензии из службы лицензий. В этом примере представлены процедуры создания преобразования с помощью встроенной предустановки AdaptiveStreaming, отправки задания, создания ContentKeyPolicy с открытым ограничением и устойчивой конфигурацией PlayReady/Widevine, связывание ContentKeyPolicy с StreamingLocator и печать URL-адреса для воспроизведения.|
| [Streaming/AssetFilters](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/Streaming/AssetFilters)|Создание преобразования с помощью встроенной предустановки AdaptiveStreaming, отправка задания, создание фильтра ресурсов и фильтра учетных записей, связывание фильтров с указателями потоковой передачи и печать URL-адресов для воспроизведения.|
| [Streaming/StreamHLSAndDASH](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/Streaming/StreamHLSAndDASH)|Создание преобразования с помощью встроенной предустановки AdaptiveStreaming, отправка задания, публикация выходного ресурса для потоковой передачи HLS и DASH.|
| [HighAvailabilityEncodingStreaming](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/HighAvailabilityEncodingStreaming/) | Рекомендации и методики для производственной системы, где используется кодирование или аналитика по запросу. Читателям нужно начать с сопутствующей статьи [Высокая доступность со службами мультимедиа и VOD](https://docs.microsoft.com/azure/media-services/latest/architecture-high-availability-encoding-concept). Для примера [HighAvailabilityEncodingStreaming](https://github.com/Azure-Samples/media-services-v3-dotnet/blob/main/HighAvailabilityEncodingStreaming/README.md) существует отдельный файл с решением. |

## <a name="nodejs"></a>[Node.JS](#tab/node/)

|Папка|Описание|
|---|---|
|[HelloWorld-ListAssets](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/HelloWorld-ListAssets) |Подключение и перечисление ресурсов |
|[Live](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/Live)| Выполнение базовой потоковой трансляции. **Предупреждение**. Убедитесь, что все ресурсы очищены и что на портале во время трансляции функция выставления счетов отключена.|
|[StreamFilesSample](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/StreamFilesSample)| Отправка локального файла или кодировки с исходного URL-адреса, использование пакета SDK для службы хранилища для загрузки содержимого и выполнение потоковой передачи в проигрыватель. |
|[StreamFilesWithDRMSample](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/StreamFilesWithDRMSample)| Кодирование и потоковая передача с помощью Widevine и PlayReady DRM |
|[VideoIndexerSample](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/VideoIndexerSample)| Использование предустановок анализатора видео и звука для создания метаданных и аналитических сведений из видео или звукового файла. |

## <a name="python"></a>[Python](#tab/python)

В настоящее время существует один пример Python: [Базовое кодирование с помощью Python](https://github.com/Azure-Samples/media-services-v3-python).

## <a name="java"></a>[Java](#tab/java)

|Папка|Описание|
|---|---|
|[AudioAnalytics/AudioAnalyzer/](https://github.com/Azure-Samples/media-services-v3-java/tree/master/AudioAnalytics/AudioAnalyzer)|Анализ звука в файле мультимедиа. |
|Защита содержимого|
|ContentProtection||
|[BasicAESClearKey](https://github.com/Azure-Samples/media-services-v3-java/tree/master/ContentProtection/BasicAESClearKey)|Динамическое шифрование содержимого с помощью AES-128.|
|[BasicPlayReady](https://github.com/Azure-Samples/media-services-v3-java/tree/master/ContentProtection/BasicPlayReady)|Динамическое шифрование содержимого с помощью PlayReady DRM.|
|[BasicWidevine](https://github.com/Azure-Samples/media-services-v3-java/tree/master/ContentProtection/BasicWidevine)|Динамическое шифрование содержимого с помощью Widevine DRM.|
|[OfflineFairPlay](https://github.com/Azure-Samples/media-services-v3-java/tree/master/ContentProtection/OfflineFairPlay)|Динамическое шифрование содержимого с помощью FairPlay DRM и воспроизведение содержимого без запроса лицензии из службы лицензий.|
|[OfflinePlayReadyAndWidevine](https://github.com/Azure-Samples/media-services-v3-java/tree/master/ContentProtection/OfflinePlayReadyAndWidevine)|Динамическое шифрование содержимого с помощью PlayReady и Widevine DRM, воспроизведение содержимого без запроса лицензии из службы лицензий.|
|DynamicPackagingVODContent||
|[AssetFilters](https://github.com/Azure-Samples/media-services-v3-java/tree/master/DynamicPackagingVODContent/AssetFilters)|Фильтрация содержимого с помощью фильтров ресурсов и учетных записей.|
|[StreamHLSAndDASH](https://github.com/Azure-Samples/media-services-v3-java/tree/master/DynamicPackagingVODContent/StreamHLSAndDASH)|Динамическая упаковка содержимого VOD в HLS/DASH для потоковой передачи.|
|[LiveIngest/LiveEventWithDVR](https://github.com/Azure-Samples/media-services-v3-java/tree/master/LiveIngest/LiveEventWithDVR)|Создание Live Event с полным архивом длительностью до 25 часов и фильтра по ресурсу с 5-минутным окном DVR, создание указателя для потоковой передачи и использование фильтра.|
|[VideoAnalytics/VideoAnalyzer](https://github.com/Azure-Samples/media-services-v3-java/tree/master/VideoAnalytics/VideoAnalyzer)|Создание Live Event с полным архивом длительностью до 25 часов и фильтра по ресурсу с 5-минутным окном DVR, создание указателя для потоковой передачи и использование фильтра.|
|VideoEncoding||
|[EncodingWithMESCustomPreset](https://github.com/Azure-Samples/media-services-v3-java/tree/master/VideoEncoding/EncodingWithMESCustomPreset)|Создание настраиваемого преобразования кодирования с помощью параметров StandardEncoderPreset.|
|[EncodingWithMESPredefinedPreset](https://github.com/Azure-Samples/media-services-v3-java/tree/master/VideoEncoding/EncodingWithMESPredefinedPreset)|Отправка задания с помощью встроенной предустановки и входного URL-адреса HTTP, публикация выходного ресурса для потоковой передачи и загрузка результатов для проверки.|

---
