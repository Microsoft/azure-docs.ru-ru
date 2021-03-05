---
title: Использование интерфейса командной строки для создания фильтров с помощью служб мультимедиа Azure
description: В этой статье показано, как использовать интерфейс командной строки для создания фильтров с помощью служб мультимедиа Azure v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 54e99c645b93bfcb3aee4e81cef3b2b5d6589d95
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102211470"
---
# <a name="creating-filters-with-cli"></a>Создание фильтров с помощью CLI

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

При доставке содержимого пользователям (потоковой трансляции мероприятий или видео по запросу) клиенту может потребоваться больше возможностей, чем описано в файле манифеста ресурса по умолчанию. Службы мультимедиа Azure позволяют определять фильтры учетной записи и фильтры ресурсов для содержимого.

Подробное описание этой функции и сценариев, в которых она используется, см. в разделе [динамические манифесты](filters-dynamic-manifest-overview.md) и [фильтры](filters-concept.md).

В этой статье показано, как настроить фильтр для ресурса-контейнера видео по запросу и использовать CLI для Служб мультимедиа версии 3, чтобы создать [фильтры учетных записей](/cli/azure/ams/account-filter) и [фильтры ресурсов](/cli/azure/ams/asset-filter).

> [!NOTE]
> Обязательно ознакомьтесь с [пресентатионтимеранже](filters-concept.md#presentationtimerange).

## <a name="prerequisites"></a>Предварительные требования

- [Создание учетной записи Служб мультимедиа](./create-account-howto.md). Обязательно запомните имя группы ресурсов и имя учетной записи Служб мультимедиа.

## <a name="define-a-filter"></a>Определение фильтра

В следующем примере определяются условия выбора дорожки, которые добавляются в манифест. Этот фильтр включает любые аудиодорожки в формате EC3 и любые видеодорожки со скоростью в диапазоне 0–1 000 000.

> [!TIP]
> Если вы планируете определять **фильтры** в других частях, обратите внимание, что необходимо включить объект оболочки JSON "Properties".  

```json
[
    {
        "trackSelections": [
            {
                "property": "Type",
                "value": "Audio",
                "operation": "Equal"
            },
            {
                "property": "FourCC",
                "value": "EC-3",
                "operation": "NotEqual"
            }
        ]
    },
    {
        "trackSelections": [
            {
                "property": "Type",
                "value": "Video",
                "operation": "Equal"
            },
            {
                "property": "Bitrate",
                "value": "0-1000000",
                "operation": "Equal"
            }
        ]
    }
]
```

## <a name="create-account-filters"></a>Создание фильтров учетной записи

Следующая команда [az ams account-filter](/cli/azure/ams/account-filter) создает фильтр учетной записи с выбранными дорожками для отслеживания, которые были [определены ранее](#define-a-filter).

Команде можно передать необязательный параметр `--tracks`, который содержит код JSON, представляющий выбранные элементы для отслеживания.  Чтобы загрузить JSON из файла, используйте синтаксис @{файл}. Если вы используете Azure CLI локально, укажите путь к файлу полностью.

```azurecli
az ams account-filter create -a amsAccount -g resourceGroup -n filterName --tracks @tracks.json
```

Также см. [примеры JSON для фильтров](/rest/api/media/accountfilters/createorupdate#create-an-account-filter).

## <a name="create-asset-filters"></a>Создание фильтров ресурсов

Следующая команда [az ams asset-filter](/cli/azure/ams/asset-filter) создает фильтр ресурса с выбранными дорожками для отслеживания, которые были [определены ранее](#define-a-filter). 

```azurecli
az ams asset-filter create -a amsAccount -g resourceGroup -n filterName --asset-name assetName --tracks @tracks.json
```

Также см. [примеры JSON для фильтров](/rest/api/media/assetfilters/createorupdate#create-an-asset-filter).

## <a name="associate-filters-with-streaming-locator"></a>Связывание фильтров с указателем потоковой передачи

Можно указать список фильтров активов или учетных записей, которые будут применяться к указателю потоковой передачи. [Динамический упаковщик (конечная точка потоковой передачи)](dynamic-packaging-overview.md) применяет этот список фильтров вместе с тем, что ваш клиент указывает в URL-адресе. Это сочетание создает [динамический манифест](filters-dynamic-manifest-overview.md), основанный на фильтрах в URL-адресах и фильтрах, указанных при указателе потоковой передачи. Рекомендуется использовать эту функцию, если вы хотите применить фильтры, но не хотите предоставлять имена фильтров в URL-адресе.

В следующем коде CLI показано, как создать указатель потоковой передачи и указать `filters` . Это необязательное свойство, которое принимает разделенный пробелами список имен фильтров ресурсов и/или имен фильтров учетных записей.

```azurecli
az ams streaming-locator create -a amsAccount -g resourceGroup -n streamingLocatorName \
                                --asset-name assetName \                               
                                --streaming-policy-name policyName \
                                --filters filterName1 filterName2
                                
```

## <a name="stream-using-filters"></a>Поток с использованием фильтров

После определения фильтров клиенты могут использовать их в URL-адресах потоковой передачи. Фильтры можно применять к протоколам потоковой передачи с переменной скоростью: Apple HTTP Live Streaming (HLS), MPEG-DASH и Smooth Streaming.

В представленной ниже таблице приводятся некоторые примеры URL-адресов с фильтрами.

|Протокол|Пример|
|---|---|
|HLS|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=m3u8-aapl,filter=myAccountFilter)`|
|MPEG DASH|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=mpd-time-csf,filter=myAssetFilter)`|
|Smooth Streaming|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(filter=myAssetFilter)`|

## <a name="next-step"></a>Следующий шаг

[Потоковая передача видео](stream-files-tutorial-with-api.md)

## <a name="see-also"></a>См. также раздел

[Azure CLI](/cli/azure/ams)
