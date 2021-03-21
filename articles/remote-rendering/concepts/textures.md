---
title: Текстуры
description: Рабочий процесс ресурса текстуры
author: florianborn71
ms.author: flborn
ms.date: 02/05/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: e01ddf0690f11d41021e0a5ae5958c7c80646743
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99594423"
---
# <a name="textures"></a>Текстуры

Текстуры являются неизменяемым [общим ресурсом](../concepts/lifetime.md). Текстуры можно загружать из [хранилища BLOB-объектов](../how-tos/conversion/blob-storage.md) и применять непосредственно к моделям, как показано в [Учебник. Изменение среды и материалов](../tutorials/unity/materials-lighting-effects/materials-lighting-effects.md). Однако наиболее часто текстуры будут частью [преобразованной модели](../how-tos/conversion/model-conversion.md), где на них ссылаются ее [материалы](materials.md).

## <a name="texture-types"></a>Типы текстур

Различные типы текстур используются в разных случаях.

* **2D-текстуры** в основном используются в [ материалах](materials.md).
* **Кубические карты** обычно используются для [неба](../overview/features/sky.md).

## <a name="supported-texture-formats"></a>Поддерживаемые форматы текстур

Все текстуры, переданные в ARR, должны быть в [формате DDS](https://en.wikipedia.org/wiki/DirectDraw_Surface). Желательно с MIP-картами и сжатием текстур.

## <a name="loading-textures"></a>Загрузка текстур

При загрузке текстуры необходимо указать ожидаемый тип. Если тип не соответствует, происходит сбой загрузки текстуры.
При загрузке текстуры с одним и тем же URI дважды, возвращается один и тот же объект текстуры, так как это [общий ресурс](../concepts/lifetime.md).

Аналогично загрузке моделей, существует два варианта обращения к ресурсу текстуры в исходном хранилище BLOB-объектов.

* К текстуре также можно обращаться напрямую по параметрам хранилища BLOB-объектов, если [хранилище BLOB-объектов связано с учетной записью](../how-tos/create-an-account.md#link-storage-accounts). В этом случае используется функция загрузки `LoadTextureAsync` с параметром `LoadTextureOptions`.
* К ресурсу текстуры можно обращаться по подписанному URL-адресу. Для этого используется функция `LoadTextureFromSasAsync` с параметром `LoadTextureFromSasOptions`. Также используйте этот вариант для загрузки [встроенных текстур](../overview/features/sky.md#built-in-environment-maps).

В следующем примере кода показано, как загрузить текстуру.

```cs
async void LoadMyTexture(RenderingSession session, string storageContainer, string blobName, string assetPath)
{
    try
    {
        LoadTextureOptions options = new LoadTextureOptions(storageContainer, blobName, assetPath, TextureType.Texture2D);
        Texture texture = await session.Connection.LoadTextureAsync(options);
    
        // use texture...
    }
    catch (RRException ex)
    {
    }
}
```

```cpp
void LoadMyTexture(ApiHandle<RenderingSession> session, std::string storageContainer, std::string blobName, std::string assetPath)
{
    LoadTextureOptions params;
    params.TextureType = TextureType::Texture2D;
    params.Blob.StorageAccountName = std::move(storageContainer);
    params.Blob.BlobContainerName = std::move(blobName);
    params.Blob.AssetPath = std::move(assetPath);
    session->Connection()->LoadTextureAsync(params, [](Status status, ApiHandle<Texture> texture)
    {
        // use texture...
    });
}
```

Обратите внимание, что в случае использования варианта SAS, функция или параметр не отличаются.

В зависимости от того, для чего предполагается использовать текстуру, могут существовать ограничения по типу и содержимому текстуры. Например, карты неровностей [материала PBR](../overview/features/pbr-materials.md) должны быть в оттенках серого.

## <a name="api-documentation"></a>Документирование API

* [Класс текстуры C#](/dotnet/api/microsoft.azure.remoterendering.texture)
* [C# Рендерингконнектион. Лоадтекстуреасинк ()](/dotnet/api/microsoft.azure.remoterendering.renderingconnection.loadtextureasync)
* [C# Рендерингконнектион. Лоадтекстурефромсасасинк ()](/dotnet/api/microsoft.azure.remoterendering.renderingconnection.loadtexturefromsasasync)
* [Класс текстуры C++](/cpp/api/remote-rendering/texture)
* [C++ Рендерингконнектион:: Лоадтекстуреасинк ()](/cpp/api/remote-rendering/renderingconnection#loadtextureasync)
* [C++ Рендерингконнектион:: Лоадтекстурефромсасасинк ()](/cpp/api/remote-rendering/renderingconnection#loadtexturefromsasasync)

## <a name="next-steps"></a>Дальнейшие действия

* [Материалы](materials.md)
* [Небо](../overview/features/sky.md)