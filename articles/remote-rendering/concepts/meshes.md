---
title: Сетки
description: Определение сеток в области удаленной подготовки к просмотру Azure
author: florianborn71
ms.author: flborn
ms.date: 02/05/2020
ms.topic: conceptual
ms.openlocfilehash: 33894e0d0b6f3ea50ffeac4f0c90c60f28e09f5e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99594524"
---
# <a name="meshes"></a>Сетки

## <a name="mesh-resource"></a>Ресурс сетки

Сетки — это неизменяемый [общий ресурс](../concepts/lifetime.md), который можно создать только с помощью [преобразования модели](../how-tos/conversion/model-conversion.md). Сетки содержат одну или несколько вложенных *сеток* , а также физическое представление [запросов райкаст](../overview/features/spatial-queries.md). Каждая подсеть ссылается на [материал](materials.md) , по умолчанию который должен быть визуализирован. Чтобы поместить сетку в трехмерное пространство, добавьте [мешкомпонент](#meshcomponent) в [сущность](entities.md).

### <a name="mesh-resource-properties"></a>Свойства ресурса сетки

`Mesh`Свойства класса:

* **Материалы:** Массив материалов. Каждый материал используется другой подсетью. Несколько записей в массиве могут ссылаться на один и тот же [материал](materials.md). Эти данные невозможно изменить во время выполнения.

* **Границы:** Ограничивающий прямоугольник по оси локального пробела (ААББ) вершин сетки.

## <a name="meshcomponent"></a>мешкомпонент

`MeshComponent`Класс используется для размещения экземпляра ресурса сетки. Каждый Мешкомпонент ссылается на одну сеть. Он может переопределить, какие материалы используются для отрисовки каждой подсети.

### <a name="meshcomponent-properties"></a>Свойства Мешкомпонент

* **Сетка:** Ресурс сетки, используемый этим компонентом.

* **Материалы:** Массив материалов, указанный в самом компоненте сетки. Размер массива всегда будет такой же, как у массива *Materials* в ресурсе сетки. Материалы, которые не должны переопределяться из сетки по умолчанию, имеют значение *null* в этом массиве.

* **Уседматериалс:** Массив фактически используемых материалов для каждой подсети. Будет совпадать с данными в массиве *Materials* для значений, отличных от NULL. В противном случае он содержит значение из массива *Materials* в экземпляре сетки.

### <a name="sharing-of-meshes"></a>Совместное использование сеток

`Mesh`Ресурс может совместно использоваться несколькими экземплярами компонентов сетки. Кроме того, `Mesh` ресурс, назначенный компоненту сетки, может быть изменен программным способом в любое время. В приведенном ниже коде показано, как клонировать сетку.

```cs
Entity CloneEntityWithModel(RenderingConnection api, Entity sourceEntity)
{
    MeshComponent meshComp = sourceEntity.FindComponentOfType<MeshComponent>();
    if (meshComp != null)
    {
        Entity newEntity = api.CreateEntity();
        MeshComponent newMeshComp = api.CreateComponent(ObjectType.MeshComponent, newEntity) as MeshComponent;
        newMeshComp.Mesh = meshComp.Mesh; // share the mesh
        return newEntity;
    }
    return null;
}
```

```cpp
ApiHandle<Entity> CloneEntityWithModel(ApiHandle<RenderingConnection> api, ApiHandle<Entity> sourceEntity)
{
    if (ApiHandle<MeshComponent> meshComp = sourceEntity->FindComponentOfType<MeshComponent>())
    {
        ApiHandle<Entity> newEntity = *api->CreateEntity();
        ApiHandle<MeshComponent> newMeshComp = api->CreateComponent(ObjectType::MeshComponent, newEntity)->as<RemoteRendering::MeshComponent>();
        newMeshComp->SetMesh(meshComp->GetMesh()); // share the mesh
        return newEntity;
    }
    return nullptr;
}
```

## <a name="api-documentation"></a>Документирование API

* [Класс сетки C#](/dotnet/api/microsoft.azure.remoterendering.mesh)
* [Класс C# Мешкомпонент](/dotnet/api/microsoft.azure.remoterendering.meshcomponent)
* [Класс сетки C++](/cpp/api/remote-rendering/mesh)
* [Класс C++ Мешкомпонент](/cpp/api/remote-rendering/meshcomponent)


## <a name="next-steps"></a>Дальнейшие действия

* [Материалы](materials.md)