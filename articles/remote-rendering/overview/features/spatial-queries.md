---
title: Пространственные запросы
description: Узнайте, как выполнять пространственные запросы к сцене
author: jakrams
ms.author: jakras
ms.date: 02/07/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: c664df586c260b3e16f64c071190055dbaeccd24
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99594050"
---
# <a name="spatial-queries"></a>Пространственные запросы

Пространственные запросы — это операции, с помощью которых можно опрашивать службу удаленной визуализации о том, какие объекты расположены в области. Пространственные запросы часто используются для реализации взаимодействий, например, для выяснения того, на какой объект указывает пользователь.

Все пространственные запросы вычисляются на сервере. Следовательно, эти операции выполняются асинхронно, и результаты поступают с некоторой задержкой, которая зависит от задержки сети. Так как каждый пространственный запрос создает сетевой трафик, старайтесь не выполнять слишком много запросов одновременно.

## <a name="collision-meshes"></a>Сетки столкновений

Пространственные запросы работают на основе ядра [Havok Physics](https://www.havok.com/products/havok-physics) и требуют наличия сетки столкновений. По умолчанию сетка столкновений создается при [преобразовании модели](../../how-tos/conversion/model-conversion.md). Если вы не планируете использовать пространственные запросы к сложным моделям, вы можете отключить создание сетки столкновений в [параметрах преобразования](../../how-tos/conversion/configure-model-conversion.md), так как это связано с некоторыми моментами:

* [Преобразование модели](../../how-tos/conversion/model-conversion.md) занимает значительно больше времени.
* Файлы преобразованных моделей становятся заметно больше, что в свою очередь влияет на скорость загрузки.
* Время загрузки в среду выполнения увеличивается.
* Потребление ЦП и памяти в среде выполнения возрастает.
* Для каждого экземпляра модели немного снижается производительность.

## <a name="ray-casts"></a>Пересечение лучами

Пространственный запрос *пересечение лучом* позволяет среде выполнения выяснить, какие объекты пересекаются лучом, который начинается из указанной точки и направлен в определенную сторону. Для оптимизации производительности можно указать максимальную дальность луча, чтобы не искать слишком далекие объекты.

```cs
async void CastRay(RenderingSession session)
{
    // trace a line from the origin into the +z direction, over 10 units of distance.
    RayCast rayCast = new RayCast(new Double3(0, 0, 0), new Double3(0, 0, 1), 10);

    // only return the closest hit
    rayCast.HitCollection = HitCollectionPolicy.ClosestHit;

    RayCastQueryResult result = await session.Connection.RayCastQueryAsync(rayCast);
    RayCastHit[] hits = result.Hits;
    if (hits.Length > 0)
    {
        var hitObject = hits[0].HitObject;
        var hitPosition = hits[0].HitPosition;
        var hitNormal = hits[0].HitNormal;

        // do something with the hit information
    }
}
```

```cpp
void CastRay(ApiHandle<RenderingSession> session)
{
    // trace a line from the origin into the +z direction, over 10 units of distance.
    RayCast rayCast;
    rayCast.StartPos = {0, 0, 0};
    rayCast.EndPos = {0, 0, 1};
    rayCast.MaxHits = 10;

    // only return the closest hit
    rayCast.HitCollection = HitCollectionPolicy::ClosestHit;

    session->Connection()->RayCastQueryAsync(rayCast, [](Status status, ApiHandle<RayCastQueryResult> result)
    {
        if (status == Status::OK)
        {
            std::vector<RayCastHit> hits;
            result->GetHits(hits);

            if (hits.size() > 0)
            {
                auto hitObject = hits[0].HitObject;
                auto hitPosition = hits[0].HitPosition;
                auto hitNormal = hits[0].HitNormal;

                // do something with the hit information
            }
        }
    });
}
```


Существует три режима сбора пересечений.

* **`Closest`:** В этом режиме возвращается только ближайшее пересечение.
* **`Any`:** Выбирайте этот режим, если вам нужно только узнать, *существует ли* пересечение луча с любым объектом, но не важно, какой именно этот объект. Этот запрос зачастую значительно дешевле в вычислении, но его применимость очень ограничена.
* **`All`:** В этом режиме возвращаются все пересечения лучом, отсортированные по дальности. Не используйте этот режим, если для вашей задачи достаточно первого пересечения. Ограничьте количество возвращаемых пересечений с помощью параметра `MaxHits`.

Чтобы избирательно исключить объекты из оценки на возможность применения пересечения лучом, можно использовать компонент [HierarchicalStateOverrideComponent](override-hierarchical-state.md).

<!--
The CollisionMask allows the query to consider or ignore some objects based on their collision layer. If an object has layer L, it will be hit only if the mask has bit L set.
It is useful in case you want to ignore objects, for instance when setting an object transparent, and trying to select another object behind it.
TODO : Add an API to make that possible.
-->

### <a name="hit-result"></a>Результат попадания

Результатом запроса пересечения лучом является массив попаданий. Если этот массив пуст, значит нет ни одного попадания.

Попадание имеет формат объекта Hit со следующими свойствами.

* **`HitEntity`:** [Сущность](../../concepts/entities.md), для которой зафиксировано попадание.
* **`SubPartId`:** *Подсетка*, для которой зафиксировано попадание в [MeshComponent](../../concepts/meshes.md). Можно использовать для индексирования `MeshComponent.UsedMaterials` и поиска [материала](../../concepts/materials.md) в этой точке.
* **`HitPosition`:** Место в мировом пространстве, где заданный луч пересекается с этим объектом.
* **`HitNormal`:** Нормаль в мировом пространстве к поверхности сетки в точке ее пересечения с лучом.
* **`DistanceToHit`:** Расстояние от начала заданного луча до позиции попадания.

## <a name="api-documentation"></a>Документирование API

* [C# Рендерингконнектион. Райкасткуерясинк ()](/dotnet/api/microsoft.azure.remoterendering.renderingconnection.raycastqueryasync)
* [C++ Рендерингконнектион:: Райкасткуерясинк ()](/cpp/api/remote-rendering/renderingconnection#raycastqueryasync)

## <a name="next-steps"></a>Дальнейшие действия

* [Границы объектов](../../concepts/object-bounds.md)
* [Переопределение иерархических состояний](override-hierarchical-state.md)