---
title: Создание фильтров с помощью пакета SDK для .NET в службах мультимедиа Azure v3
description: В этом разделе описывается создание фильтров, с помощью которых клиент может передавать определенные секции потока. Пакет SDK служб мультимедиа v3 для .NET создает динамические манифесты для реализации этой выборочной потоковой передачи.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 11c65498d5a31c2e2ee997bdaf18037b1f0f9060
ms.sourcegitcommit: 6386854467e74d0745c281cc53621af3bb201920
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102455218"
---
# <a name="create-filters-with-media-services-net-sdk"></a>Создание фильтров с помощью пакета SDK Служб мультимедиа для .NET

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

При распространении содержимого пользователям (потоковой трансляции событий или видео по запросу) клиенту может потребоваться больше возможностей, чем определено в файле манифеста ресурса-контейнера по умолчанию. Службы мультимедиа Azure позволяют определять фильтры учетной записи и фильтры ресурсов для содержимого.

Подробное описание этой функции и сценариев, в которых она используется, см. в разделе [динамические манифесты](filters-dynamic-manifest-overview.md) и [фильтры](filters-concept.md).

В этой статье показано, как использовать пакет SDK Служб мультимедиа для .NET, чтобы определить фильтр для ресурса-контейнера видео по запросу и создать [фильтры учетных записей](/dotnet/api/microsoft.azure.management.media.models.accountfilter) и [фильтры ресурсов-контейнеров](/dotnet/api/microsoft.azure.management.media.models.assetfilter). 

> [!NOTE]
> Обязательно ознакомьтесь с [пресентатионтимеранже](filters-concept.md#presentationtimerange).

## <a name="prerequisites"></a>Предварительные требования 

- См. дополнительные сведения о [фильтрах и динамических манифестах](filters-dynamic-manifest-overview.md).
- [Создание учетной записи Служб мультимедиа](./create-account-howto.md). Обязательно запомните имя группы ресурсов и имя учетной записи Служб мультимедиа. 
- Получите информацию, необходимую для [доступа к API-интерфейсам](./access-api-howto.md).
- Ознакомьтесь со статьей об [отправке, кодировании и потоковой передаче видео с помощью Служб мультимедиа Azure](stream-files-tutorial-with-api.md), чтобы [начать использовать пакет SDK для .NET](stream-files-tutorial-with-api.md#start-using-media-services-apis-with-net-sdk).

## <a name="define-a-filter"></a>Определение фильтра  

В .NET выбранные дорожки настраиваются с помощью классов [FilterTrackSelection](/dotnet/api/microsoft.azure.management.media.models.filtertrackselection) и [FilterTrackPropertyCondition](/dotnet/api/microsoft.azure.management.media.models.filtertrackpropertycondition). 

Приведенный ниже код определяет фильтр, который включает в себя любые звуковые дорожки с EC-3 и любые видеодорожки со скоростью в диапазоне 0–1 000 000.

```csharp
var audioConditions = new List<FilterTrackPropertyCondition>()
{
    new FilterTrackPropertyCondition(FilterTrackPropertyType.Type, "Audio", FilterTrackPropertyCompareOperation.Equal),
    new FilterTrackPropertyCondition(FilterTrackPropertyType.FourCC, "EC-3", FilterTrackPropertyCompareOperation.Equal)
};

var videoConditions = new List<FilterTrackPropertyCondition>()
{
    new FilterTrackPropertyCondition(FilterTrackPropertyType.Type, "Video", FilterTrackPropertyCompareOperation.Equal),
    new FilterTrackPropertyCondition(FilterTrackPropertyType.Bitrate, "0-1000000", FilterTrackPropertyCompareOperation.Equal)
};

List<FilterTrackSelection> includedTracks = new List<FilterTrackSelection>()
{
    new FilterTrackSelection(audioConditions),
    new FilterTrackSelection(videoConditions)
};
```

## <a name="create-account-filters"></a>Создание фильтров учетной записи

В приведенном ниже коде показано, как с помощью .NET создать фильтр учетной записи, который включает все выбранные дорожки, [определенные выше](#define-a-filter). 

```csharp
AccountFilter accountFilterParams = new AccountFilter(tracks: includedTracks);
client.AccountFilters.CreateOrUpdate(config.ResourceGroup, config.AccountName, "accountFilterName1", accountFilter);
```

## <a name="create-asset-filters"></a>Создание фильтров ресурсов

В приведенном ниже коде показано, как с помощью .NET создать фильтр ресурса-контейнера, который включает все выбранные дорожки, [определенные выше](#define-a-filter). 

```csharp
AssetFilter assetFilterParams = new AssetFilter(tracks: includedTracks);
client.AssetFilters.CreateOrUpdate(config.ResourceGroup, config.AccountName, encodedOutputAsset.Name, "assetFilterName1", assetFilterParams);
```

## <a name="associate-filters-with-streaming-locator"></a>Связывание фильтров с указателем потоковой передачи

Можно указать список фильтров активов или учетных записей, которые будут применяться к указателю потоковой передачи. [Динамический упаковщик (конечная точка потоковой передачи)](dynamic-packaging-overview.md) применяет этот список фильтров вместе с тем, что ваш клиент указывает в URL-адресе. Это сочетание создает [динамический манифест](filters-dynamic-manifest-overview.md), основанный на фильтрах в URL-адресах и фильтрах, указанных при указателе потоковой передачи. Рекомендуется использовать эту функцию, если вы хотите применить фильтры, но не хотите предоставлять имена фильтров в URL-адресе.

В следующем коде C# показано, как создать указатель потоковой передачи и указать `StreamingLocator.Filters` . Это необязательное свойство, принимающее `IList<string>` имена фильтров.

```csharp
IList<string> filters = new List<string>();
filters.Add("filterName");

StreamingLocator locator = await client.StreamingLocators.CreateAsync(
    resourceGroup,
    accountName,
    locatorName,
    new StreamingLocator
    {
        AssetName = assetName,
        StreamingPolicyName = PredefinedStreamingPolicy.ClearStreamingOnly,
        Filters = filters
    });
```
      
## <a name="stream-using-filters"></a>Поток с использованием фильтров

После определения фильтров клиенты могут использовать их в URL-адресах потоковой передачи. Фильтры можно применять к протоколам потоковой передачи с переменной скоростью: Apple HTTP Live Streaming (HLS), MPEG-DASH и Smooth Streaming.

В представленной ниже таблице приводятся некоторые примеры URL-адресов с фильтрами.

|Протокол|Пример|
|---|---|
|HLS|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=m3u8-aapl,filter=myAccountFilter)`|
|MPEG DASH|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=mpd-time-csf,filter=myAssetFilter)`|
|Smooth Streaming|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(filter=myAssetFilter)`|

## <a name="next-steps"></a>Дальнейшие действия

[Потоковая передача видео](stream-files-tutorial-with-api.md) 
