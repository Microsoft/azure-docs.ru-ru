---
title: Модели
description: Узнайте, что такое модель в Удаленной отрисовке Azure
author: jakrams
ms.author: jakras
ms.date: 02/05/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: 0d1e66d09db3e3934871ed15493feb685d1cbe6a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99593880"
---
# <a name="models"></a>Модели

В Удаленной отрисовке Azure *моделью* называется полное представление объекта, составленное из [сущностей](entities.md) и [компонентов](components.md). Модели — это основной способ передачи пользовательских данных в службу удаленной отрисовки.

## <a name="model-structure"></a>Структура модели

У каждой модели только одна [сущность](entities.md) является корневым узлом. На следующих уровнях иерархии может существовать произвольная структура из дочерних сущностей. При загрузке модели возвращается ссылка на эту корневую сущность.

К каждой сущности могут прикрепляться [компоненты](components.md). В большинстве случаев сущности имеют компоненты *MeshComponents* со ссылками на [ресурсы сетки](meshes.md).

## <a name="creating-models"></a>Создание моделей

Создание моделей для среды выполнения достигается за счет [преобразования входных моделей](../how-tos/conversion/model-conversion.md) из файлов таких форматов, как FBX и GLTF. Процесс преобразования извлекает все ресурсы (текстуры, материалы, сетки и т. д.) и преобразует их в форматы, оптимизированные для среды выполнения. Он также извлекает структурную информацию и преобразует ее в структуру графа из сущностей и компонентов Удаленной отрисовки Azure.

> [!IMPORTANT]
> [Преобразование модели](../how-tos/conversion/model-conversion.md) — это единственный способ создания [сеток](meshes.md). В среде выполнения сетки могут совместно использоваться несколькими сущностями, но передать сетку в среду выполнения можно только путем загрузки модели.

## <a name="loading-models"></a>Загрузка моделей

После преобразования модель можно загрузить из хранилища BLOB-объектов Azure в среду выполнения.

Существуют две разные функции загрузки, которые отличаются способом адресации ресурса в хранилище BLOB-объектов.

* К модели можно обращаться напрямую по параметрам хранилища BLOB-объектов, если [хранилище BLOB-объектов связано с учетной записью](../how-tos/create-an-account.md#link-storage-accounts). В этом случае используется функция загрузки `LoadModelAsync` с параметром `LoadModelOptions`.
* К модели можно обращаться по подписанному URL-адресу. Для этого используется функция `LoadModelFromSasAsync` с параметром `LoadModelFromSasOptions`. Также используйте этот вариант для загрузки [встроенных моделей](../samples/sample-model.md).

В приведенных ниже фрагментах кода показано, как загрузить модели с помощью обеих этих функций. Чтобы загрузить модель с помощью параметров хранилища BLOB-объектов, используйте следующий код:


```cs
async void LoadModel(RenderingSession session, Entity modelParent, string storageAccount, string containerName, string assetFilePath)
{
    // load a model that will be parented to modelParent
    var modelOptions = new LoadModelOptions(
        storageAccount, // storage account name + '.blob.core.windows.net', e.g., 'mystorageaccount.blob.core.windows.net'
        containerName,  // name of the container in your storage account, e.g., 'mytestcontainer'
        assetFilePath,  // the file path to the asset within the container, e.g., 'path/to/file/myAsset.arrAsset'
        modelParent
    );

    var loadOp = session.Connection.LoadModelAsync(modelOptions, (float progress) =>
    {
        Debug.WriteLine($"Loading: {progress * 100.0f}%");
    });

    await loadOp;
}
```

```cpp
void LoadModel(ApiHandle<RenderingSession> session, ApiHandle<Entity> modelParent, std::string storageAccount, std::string containerName, std::string assetFilePath)
{
    LoadModelOptions modelOptions;
    modelOptions.Parent = modelParent;
    modelOptions.Blob.StorageAccountName = std::move(storageAccount);
    modelOptions.Blob.BlobContainerName = std::move(containerName);
    modelOptions.Blob.AssetPath = std::move(assetFilePath);

    ApiHandle<LoadModelResult> result;
    session->Connection()->LoadModelAsync(modelOptions,
        // completion callback
        [](Status status, ApiHandle<LoadModelResult> result)
        {
            printf("Loading: finished.");
        },
        // progress callback
        [](float progress)
        {
            printf("Loading: %.1f%%", progress * 100.f);
        }
    );
}
```

Если вы хотите загрузить модель с помощью маркера SAS, используйте код, аналогичный приведенному ниже.

```cs
async void LoadModel(RenderingSession session, Entity modelParent, string modelUri)
{
    // load a model that will be parented to modelParent
    var modelOptions = new LoadModelFromSasOptions(modelUri, modelParent);

    var loadOp = session.Connection.LoadModelFromSasAsync(modelOptions, (float progress) =>
    {
        Debug.WriteLine($"Loading: {progress * 100.0f}%");
    });

    await loadOp;
}
```

```cpp
void LoadModel(ApiHandle<RenderingSession> session, ApiHandle<Entity> modelParent, std::string modelUri)
{
    LoadModelFromSasOptions modelOptions;
    modelOptions.ModelUri = modelUri;
    modelOptions.Parent = modelParent;

    ApiHandle<LoadModelResult> result;
    session->Connection()->LoadModelFromSasAsync(modelOptions,
        // completion callback
        [](Status status, ApiHandle<LoadModelResult> result)
        {
            printf("Loading: finished.");
        },
        // progress callback
        [](float progress)
        {
            printf("Loading: %.1f%%", progress * 100.f);
        }
    );
}
```

После этого вы сможете перемещаться по иерархии сущностей и изменять сущности и компоненты. Многократная загрузка одной и той же модели создает несколько экземпляров, каждый из которых содержит собственную копию структуры из сущностей и компонентов. Но сетки, материалы и текстуры являются [общими ресурсами](../concepts/lifetime.md), поэтому их данные не будут загружаться повторно. Это означает, что создание нескольких экземпляров модели потребляет ресурсы памяти незначительно.

## <a name="api-documentation"></a>Документирование API

* [C# Рендерингконнектион. Лоадмоделасинк ()](/dotnet/api/microsoft.azure.remoterendering.renderingconnection.loadmodelasync)
* [C# Рендерингконнектион. Лоадмоделфромсасасинк ()](/dotnet/api/microsoft.azure.remoterendering.renderingconnection.loadmodelfromsasasync)
* [C++ Рендерингконнектион:: Лоадмоделасинк ()](/cpp/api/remote-rendering/renderingconnection#loadmodelasync)
* [C++ Рендерингконнектион:: Лоадмоделфромсасасинк ()](/cpp/api/remote-rendering/renderingconnection#loadmodelfromsasasync)

## <a name="next-steps"></a>Дальнейшие действия

* [Сущности](entities.md)
* [Сетки](meshes.md)