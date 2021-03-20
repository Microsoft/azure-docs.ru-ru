---
title: Components
description: Определение компонентов в области Удаленной отрисовки Azure
author: florianborn71
ms.author: flborn
ms.date: 02/04/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: 9679be03c69090a0c11d007cfc542bae70bd3cbc
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99592200"
---
# <a name="components"></a>Components

Удаленная отрисовка Azure использует шаблон проектирования [Entity-Component-System](https://en.wikipedia.org/wiki/Entity_component_system) (Система компонентов cущностей). [Сущности](entities.md) представляют собой данные о расположении и иерархической композиции объектов, а компоненты отвечают за реализацию поведения.

Чаще всего используются типы компонентов [:::no-loc text="mesh components":::](meshes.md) , которые добавляют сетки в конвейер отрисовки. Аналогичным образом [компоненты освещения](../overview/features/lights.md) используются для добавления освещения, а [компоненты сечения](../overview/features/cut-planes.md) — для сечения открытых сеток.

Все эти компоненты используют преобразование (изменение расположения, поворота, масштабирование) сущности, к которой они присоединены, в качестве опорной точки.

## <a name="working-with-components"></a>Работа с компонентами

Вы можете легко программно добавлять и удалять компоненты и оперировать ими.

```cs
// create a point light component
RenderingSession session = GetCurrentlyConnectedSession();
PointLightComponent lightComponent = session.Connection.CreateComponent(ObjectType.PointLightComponent, ownerEntity) as PointLightComponent;

lightComponent.Color = new Color4Ub(255, 150, 20, 255);
lightComponent.Intensity = 11;

// ...

// destroy the component
lightComponent.Destroy();
lightComponent = null;
```

```cpp
// create a point light component
ApiHandle<RenderingSession> session = GetCurrentlyConnectedSession();

ApiHandle<PointLightComponent> lightComponent = session->Connection()->CreateComponent(ObjectType::PointLightComponent, ownerEntity)->as<PointLightComponent>();

// ...

// destroy the component
lightComponent->Destroy();
lightComponent = nullptr;
```

Компонент прикрепляется к сущности во время создания. После этого его нельзя передать другой сущности. Компоненты явным образом удаляются с помощью функции `Component.Destroy()` или это происходит автоматически при уничтожении сущности-владельца компонента.

За раз к сущности можно добавить только один экземпляр каждого типа компонента.

## <a name="unity-specific"></a>Особенности при работе с Unity

Интеграция Unity предоставляет дополнительные функции расширения для взаимодействия с компонентами. Дополнительную информацию см. в статье [Игровые объекты и компоненты Unity](../how-tos/unity/objects-components.md).

## <a name="api-documentation"></a>Документирование API

* [Компонентбасе C#](/dotnet/api/microsoft.azure.remoterendering.componentbase)
* [C# Рендерингконнектион. Креатекомпонент ()](/dotnet/api/microsoft.azure.remoterendering.renderingconnection.createcomponent)
* [C# Entity. Финдкомпонентофтипе ()](/dotnet/api/microsoft.azure.remoterendering.entity.findcomponentoftype)
* [Компонентбасе C++](/cpp/api/remote-rendering/componentbase)
* [C++ Рендерингконнектион:: Креатекомпонент ()](/cpp/api/remote-rendering/renderingconnection#createcomponent)
* [C++ Entity:: Финдкомпонентофтипе ()](/cpp/api/remote-rendering/entity#findcomponentoftype)

## <a name="next-steps"></a>Дальнейшие действия

* [Границы объектов](object-bounds.md)
* [Сетки](meshes.md)