---
title: Границы объектов
description: Объяснение того, как можно запрашивать границы пространственных объектов
author: florianborn71
ms.author: flborn
ms.date: 02/03/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: df04b767035dffb62fde89d1e74b808d62fcc943
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99594490"
---
# <a name="object-bounds"></a>Границы объектов

Границы объектов представляют том, занятый [сущностью](entities.md) и ее дочерними элементами. В удаленной отрисовке Azure границы объектов всегда задаются в виде *выровненных по оси ограничивающих прямоугольников* (AABB). Границы объектов могут находиться либо в *локальном пространстве*, либо в *мировом пространстве*. В любом случае они всегда выравниваются по оси. Это означает, что экстенты и объемы могут отличаться между локальным и мировым представлением пространства.

## <a name="querying-object-bounds"></a>Запросы границ объектов

Ограничивающий прямоугольник с согласованием локальной оси [сетки](meshes.md) можно запрашивать непосредственно из ресурса сетки. Эти границы можно преобразовать в локальное или мировое пространство сущности с помощью преобразования сущности.

Таким образом можно вычислить границы всей иерархии объектов, но для этого необходимо выполнить обход иерархии, запросить границы для каждой сетки и объединить их вручную. Эта операция утомительна и неэффективна.

Лучше будет вызвать `QueryLocalBoundsAsync` или `QueryWorldBoundsAsync` на сущности. Тогда вычисления передаются на сервер и возвращаются с минимальной задержкой.

```cs
public async void GetBounds(Entity entity)
{
    try
    {
        Task<Bounds> boundsQuery = entity.QueryWorldBoundsAsync();
        Bounds result = await boundsQuery;
    
        Double3 aabbMin = result.Min;
        Double3 aabbMax = result.Max;
        // ...
    }
    catch (RRException ex)
    {
    }
}
```

```cpp
void GetBounds(ApiHandle<Entity> entity)
{
    entity->QueryWorldBoundsAsync(
        // completion callback:
        [](Status status, Bounds bounds)
        {
           if (status == Status::OK)
            {
                Double3 aabbMin = bounds.Min;
                Double3 aabbMax = bounds.Max;
                // ...
            }
        }
    );
}
```

## <a name="api-documentation"></a>Документирование API

* [C# Entity. Куерилокалбаундсасинк](/dotnet/api/microsoft.azure.remoterendering.entity.querylocalboundsasync)
* [Сущность C++:: Куерилокалбаундсасинк](/cpp/api/remote-rendering/entity#querylocalboundsasync)

## <a name="next-steps"></a>Дальнейшие действия

* [Пространственные запросы](../overview/features/spatial-queries.md)